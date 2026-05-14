# Hacash 多层可编程性对以太坊问题的改进

## 摘要

以太坊自 2015 年上线以来，社区通过 EIP 提案持续修补其架构缺陷，但许多核心问题至今仍未彻底解决。Hacash 的五层可编程性架构在设计之初就从结构上规避了其中多个问题。本报告逐项对照分析，区分"结构性解决"（架构层面根除）和"部分改进"（缓解但未完全消除），力求客观。

---

## 1. 重大问题

### 1.1 ERC-20 approve/transferFrom 授权模型缺陷

**以太坊现状**：ERC-20 的 `approve` + `transferFrom` 两步模式是 DeFi 最大的攻击面之一。用户必须先授权（通常是无限额度），再由合约代为转账。这导致了大量的授权钓鱼攻击和无限授权风险。EIP-2612（permit）引入了链下签名授权，EIP-7702 进一步改进，但根本问题——"合约代持授权"模式——未变。

**Hacash 的解决**：Layer 1 的原子化 action 组合从根本上消除了这个问题。资产转移是协议层的原生 action（`HacToTrs`、`HacFromTrs`、`DiaFromToTrs` 等），不需要"授权-转账"两步模式。多方资产交换在一笔 tx 中通过多个 action 原子完成，每个 action 直接要求对应地址的签名（`req_sign()`），不存在"代持授权"的概念。

```
// 以太坊：两步操作，授权可被滥用
token.approve(spender, MAX_UINT256);  // 步骤1：无限授权
spender.transferFrom(user, to, amt);  // 步骤2：代为转账

// Hacash：单笔 tx 内原子完成，无授权概念
tx.actions = [
    HacFromTrs { from: alice, amount: 100 },  // alice 签名
    HacToTrs   { to: bob,   amount: 100 },    // 无需额外授权
]
```

**评级：结构性解决。** 协议层原生资产操作 + 多 action 原子组合，从架构上消除了授权模型。

---

### 1.2 账户抽象的复杂性

**以太坊现状**：原生 EOA（外部拥有账户）只支持 ECDSA 签名，无法自定义验证逻辑。EIP-4337 引入了账户抽象，但需要 EntryPoint 合约、Bundler 网络、Paymaster 等复杂基础设施栈。EIP-7702 允许 EOA 临时委托代码，但仍是补丁式方案。截至 2025 年，完整的账户抽象仍未成为协议原生能力。

**Hacash 的解决**：Layer 4 的 AbstCall 机制是协议原生的账户抽象。每个合约地址天然具备 `Permit*`（转出授权）和 `Payable*`（转入处理）钩子。当 Layer 1 的资产转移 action 涉及合约地址时，`action_hook` 系统自动触发对应的 VM 调用，无需额外基础设施。

代码证据（`vm/src/machine/setup.rs`）：
```rust
// action_hook 在资产操作涉及合约地址时自动触发 AbstCall
// 合约可以在 PermitHAC 中实现自定义的转出验证逻辑
// 合约可以在 PayableHAC 中实现自定义的接收逻辑
```

P2SH（脚本哈希账户）进一步提供了无需部署合约的自定义签名验证——多签、时间锁、条件支付等场景只需提供脚本和 witness。

**评级：结构性解决。** 账户抽象是协议原生能力，不需要 EntryPoint/Bundler/Paymaster 基础设施栈。

---

### 1.3 状态膨胀（State Bloat）

**以太坊现状**：以太坊的存储是永久的——一旦写入，除非合约主动 `SSTORE(key, 0)` 清除，数据永远占用全节点存储。这导致状态持续膨胀（截至 2025 年已超过 300GB）。EIP-4444（历史数据过期）、EIP-4762（Verkle tree gas 调整）、Vitalik 多次提出的"状态过期"方案至今未落地。EIP-7702 和 EOF 系列也未触及这个根本问题。

**Hacash 的解决**：Layer 5 的存储租金模型从经济机制上解决了状态膨胀。

代码证据（`vm/src/field/storage.rs:363-424`）：
- `ssave` 写入存储时按 `value_len + base_size` 计算租金，新建 key 额外收取 `key_create_fee`
- 存储有过期时间（`expire` 字段），过期后 `sload` 返回 Nil
- `srent` 允许续租，`sdel` 主动删除
- 过期超过宽限期的数据被彻底回收（`is_delete` 分支）
- `ssave` 在剩余租期不足一个周期时自动续租一个周期

```rust
// 存储不是永久免费的：
// 1. 创建 key 收取 key_create_fee gas
// 2. 每个周期按 value_len + base_size 收取租金
// 3. 过期数据对 sload 不可见
// 4. 超过宽限期的数据被物理删除
```

**评级：结构性解决。** 存储租金 + 过期回收从经济和技术两个层面解决了状态膨胀，而以太坊至今没有可行的状态过期方案。

---

### 1.4 交易可审计性与 MEV

**以太坊现状**：以太坊交易只有一个入口点（`to` 地址 + `calldata`），实际的资产流向必须通过模拟执行才能确定。这导致：(1) 用户无法在签名前确切知道交易会做什么；(2) MEV 搜索者可以通过模拟发现套利机会并进行三明治攻击。EIP-3074（AUTH/AUTHCALL）和 EIP-7702 试图改善用户体验，但未解决可审计性问题。

**Hacash 的解决**：Layer 1-2 的声明式设计使交易体可以被静态分析。

- Layer 1：所有资产操作（转入、转出、交换）在 action 列表中显式声明，无需执行即可确定资产流向
- Layer 2：`AstSelect` 和 `AstIf` 的所有分支都在交易体中可见，`collect_req_sign()` 递归收集所有可能路径的签名需求
- 钱包和审计工具可以在签名前展示所有可能的执行路径和资产变动

对于 MEV：由于 Layer 1-2 的交易不涉及合约状态读取，其执行结果不依赖于交易排序，天然免疫三明治攻击。只有 Layer 3-5 涉及 VM 执行的交易才可能受 MEV 影响。

**评级：重大改进（Layer 1-2 结构性解决，Layer 3-5 与以太坊类似）。**

---

### 1.5 重入攻击（Reentrancy）

**以太坊现状**：重入攻击是以太坊最经典的安全问题（The DAO 事件）。虽然 Solidity 社区发展出了 ReentrancyGuard 模式和 checks-effects-interactions 最佳实践，但这依赖开发者自律。EIP-1153（transient storage）提供了更高效的重入锁，但仍是可选的。

**Hacash 的解决**：多层防御。

1. **Layer 1-2 天然免疫**：原子化 action 和 AST 条件逻辑不涉及外部调用，不存在重入路径。
2. **Layer 3 EXTACTION 限制**：`EXTACTION` 仅允许在 `Main` 模式、`depth == 0`、非 `callcode` 上下文执行（`execute.rs:245-248`）。合约代码无法通过 EXTACTION 发起资产转移，切断了经典重入路径。
3. **Layer 4-5 重入深度硬限制**：`GasCounter.reentry_depth` 通过 `GasCallGuard` RAII 守卫管理，硬上限 `max_reentry_depth = 4`（`SpaceCap`）。超过限制直接报错，不依赖开发者实现锁。

```rust
// machine.rs - 协议层强制重入深度限制
fn enter(&mut self) -> Rerr {
    let next_depth = self.reentry_depth.checked_add(1)...;
    if next_depth > self.max_reentry + 1 {
        return errf!("re-entry depth {} exceeded limit {}", ...)
    }
    ...
}
```

**评级：结构性解决（Layer 1-2）+ 重大改进（Layer 3-5 协议层强制限制，不依赖开发者）。**

---

### 1.6 多方原子交换的复杂性

**以太坊现状**：多方原子交换需要通过合约实现（如 HTLC 或专用 swap 合约），涉及多笔交易、时间锁、超时退款等复杂逻辑。EIP-7702 改善了单用户的批量操作，但多方场景仍需合约编排。

**Hacash 的解决**：Layer 1 天然支持多方原子交换。一笔 tx 可以包含多个 `FromTrs`/`ToTrs` action，涉及不同地址和不同资产类型（HAC、SAT、HACD、自定义 Asset），所有参与方签名后原子执行。

结合 Layer 2 的 `AstIf`，可以构造条件原子交换：
```
AstIf {
    cond: [检查某个链上条件],
    br_if: [
        HacFromTrs { from: alice, amount: 100 },
        DiaFromToTrs { from: bob, to: alice, diamond: "WTYUIA" },
    ],
    br_else: [无操作或替代方案],
}
```

**评级：结构性解决。** 协议层原生支持，无需合约、无需 HTLC、无需多笔交易。

---

### 1.7 签名内容的用户友好的可审计性

**以太坊现状**：一个常见攻击路径并非协议本身漏洞，而是前端偷换签名内容。用户在钱包中经常看到的是 `hex` 原始数据或复杂结构体（尤其在 `eth_sign` / `personal_sign` / 各类 permit 流程里），很难直观判断到底在授权谁、授权什么、会触发哪些执行路径。一旦前端被篡改，用户可能在不知情下签出危险授权，后续由攻击者提交链上执行完成盗取。

**Hacash 的解决**：可读合约与脚本合约（P2SH）把“签什么”直接绑定到可读语义。Layer 1-2 的 action/AST 交易体天然是声明式结构，钱包可在签名前展示清晰的人类可读信息：资产类型、数量、来源/目标地址、条件分支以及所需签名集合。用户签署的是可审计的业务语义，而不是难以辨认的 `hex` 原始数据，从而显著缩小前端偷换签名内容的攻击面。

**评级：结构性解决（Layer 1-2 + 可读合约/P2SH）。**

---

## 2. 中等问题

### 2.1 Gas 计量的可预测性

**以太坊现状**：EVM 的 gas 计量依赖于运行时状态（冷/热存储访问、SSTORE 的退款规则等）。EIP-2929（冷/热访问列表）、EIP-3529（减少 SSTORE 退款）、EIP-7623（calldata gas 调整）持续修补，但 gas 估算仍然不精确，经常导致交易失败或多付。

**Hacash 的解决**：分层 gas 模型。

- Layer 1-2：gas 消耗等于 action 的序列化大小（`self.size() as u32`），完全确定性，提交前可精确计算。
- Layer 3-5：VM 执行的 gas 消耗基于指令表（`GasTable`），存储操作按 value 大小线性计费，无冷/热访问区分。每种调用类型有最低 gas（`main_call_min`、`abst_call_min`、`p2sh_call_min`），防止极低成本的垃圾调用。
- Gas 单调消费：失败的 AST 分支消耗的 gas 不退还（`GasCounter.remaining` 注释：`never restored by AST recover`），消除了 gas 退款的复杂性。

**评级：重大改进（Layer 1-2 完全确定性；Layer 3-5 模型更简单但仍需估算）。**

---

### 2.2 合约升级的安全性

**以太坊现状**：合约升级依赖代理模式（Proxy Pattern），存在存储布局冲突、实现合约被意外自毁（已被 EIP-6780 部分解决）、升级权限管理等风险。OpenZeppelin 的 UUPS/Transparent Proxy 是事实标准，但增加了复杂度和攻击面。

**Hacash 的解决**：Layer 5 的合约系统内置了升级机制。

- `ContractUpdate` action（kind=98）是协议层的升级操作
- `ContractEdit` 支持添加新函数（`Append`）和修改现有函数（`Change`）
- 升级需要合约 owner 签名
- `revision` 字段跟踪版本号
- 合约可以定义 `Change` AbstCall 钩子来实现自定义的升级验证逻辑

不需要代理模式，不存在存储布局冲突问题（存储 key 是显式的，不依赖 slot 编号）。

**评级：重大改进。** 协议原生升级机制消除了代理模式的复杂性和风险。

---

### 2.3 交易批量操作（Batching）

**以太坊现状**：一笔以太坊交易只能调用一个合约的一个函数。批量操作需要通过 Multicall 合约或 EIP-7702 的临时代码委托。EIP-3074（AUTH/AUTHCALL）被 EIP-7702 取代，但批量操作仍不是协议原生能力。

**Hacash 的解决**：Layer 1 天然支持批量操作——一笔 tx 可以包含任意数量的 action，按序原子执行。这是协议的基础设计，不需要任何额外机制。

**评级：结构性解决。**

---

### 2.4 Token 标准碎片化

**以太坊现状**：ERC-20、ERC-721、ERC-1155、ERC-4626 等 token 标准各自独立，互操作性依赖开发者遵守接口规范。不合规的 token 实现（如缺少返回值的 `transfer`）导致了大量兼容性问题。EIP-7575 等试图统一 vault 接口，但碎片化问题持续存在。

**Hacash 的解决**：Layer 1 的原生多资产系统。HAC、SAT、HACD 在协议层有专用的 action 类型，行为完全一致，不存在"不合规实现"的可能。自定义 Asset 通过 `AssetSmelt` 在协议层注册，转移通过 `AssetToTrs`/`AssetFromTrs` 等标准 action 完成。

**局限**：自定义 Asset 的灵活性低于 ERC-20（无法自定义 transfer 逻辑），但这正是安全性的来源。

**评级：结构性解决（原生资产）/ 部分改进（自定义 Asset 灵活性受限）。**

---

## 3. 次要问题

### 3.1 合约间调用的透明性

**以太坊现状**：合约间的 `CALL`/`DELEGATECALL`/`STATICCALL` 在交易层面不可见，只能通过 trace 分析。EIP-3155（trace 标准化）改善了调试体验，但链上透明性未变。

**Hacash 的解决**：Layer 3 的 EXTACTION 机制使 VM 到 protocol 层的回调可追踪——每次 EXTACTION 都经过 `ctx_action_call()` 统一入口，可以记录和审计。但合约间的 Outer/Inner call 仍然是 VM 内部行为，透明性与以太坊类似。

**评级：部分改进。**

---

### 3.2 存储 Slot 冲突

**以太坊现状**：EVM 的存储基于 256-bit slot 编号，代理模式中实现合约和代理合约可能使用相同的 slot，导致存储冲突。EIP-1967（标准代理存储 slot）和 EIP-7201（命名空间存储布局）是补丁方案。

**Hacash 的解决**：Layer 5 的存储使用显式的 key-value 模型（`ssave(key, value)`），key 是 Value 类型而非固定 slot 编号。合约地址作为 key 前缀（`Self::skey(cadr, &k)`），天然隔离不同合约的存储空间。加上协议原生的升级机制不需要代理模式，存储冲突问题从根本上不存在。

**评级：结构性解决。**

---

### 3.3 签名验证的灵活性

**以太坊现状**：EOA 只支持 secp256k1 ECDSA 签名。EIP-7212（secp256r1 预编译）为 Passkey 等场景提供支持，但仍是逐个曲线添加。通用的签名抽象需要 EIP-4337 的 UserOperation 验证。

**Hacash 的解决**：Layer 4 的 P2SH 允许任意签名验证逻辑——脚本在 VM 中执行，可以实现任何验证算法。AbstCall 的 `Permit*` 钩子也允许合约自定义验证。不需要为每种签名算法添加预编译。

**评级：结构性解决。**

---

### 3.4 交易失败仍扣 Gas 的用户体验

**以太坊现状**：交易执行失败（revert）仍然消耗 gas 并扣费。用户为失败交易付费是长期的体验痛点。

**Hacash 的解决**：Layer 1-2 的确定性 gas 模型使得交易失败的概率大幅降低——action 的执行结果在大多数场景下可以预判。Layer 2 的 `AstSelect(min=0)` 允许"尽力执行"语义，部分 action 失败不导致整笔 tx 失败。但 Layer 3-5 的 VM 执行失败仍然消耗 gas，与以太坊一致。

**评级：部分改进（Layer 1-2 降低失败概率；Layer 3-5 未改变）。**

---

## 4. Hacash 未解决或引入的新问题

客观地说，Hacash 的设计也有其代价：

| 问题 | 说明 |
|------|------|
| 协议演进速度 | action 类型、AbstCall 类型硬编码，新功能需硬分叉 |
| 合约间交互灵活性 | 无法像以太坊那样任意合约间调用，必须通过 AbstCall/Outer call |
| 语言生态 | fitsh 是自定义语言，缺乏成熟的审计工具和开发者社区 |
| 存储租金运营负担 | DeFi 协议需要持续支付租金或设计分摊机制 |
| EXTACTION 限制 | 合约无法直接发起资产转移，增加了某些 DeFi 模式的实现复杂度 |

---

## 5. 总结对照表

| 以太坊问题 | 相关 EIP | Hacash 解决层 | 评级 |
|-----------|----------|-------------|------|
| approve/transferFrom 授权缺陷 | EIP-2612, EIP-7702 | Layer 1 | 结构性解决 |
| 账户抽象复杂性 | EIP-4337, EIP-7702 | Layer 4 | 结构性解决 |
| 状态膨胀 | EIP-4444, EIP-4762 | Layer 5 | 结构性解决 |
| 交易可审计性 / MEV | EIP-3074 | Layer 1-2 | 重大改进 |
| 签名内容用户友好可审计性（防前端偷换） | EIP-712, EIP-2612, EIP-7702 | Layer 1-2 + 可读合约/P2SH | 结构性解决 |
| 重入攻击 | EIP-1153 | Layer 1-5 | 结构性解决 + 重大改进 |
| 多方原子交换 | 无直接 EIP | Layer 1-2 | 结构性解决 |
| Gas 可预测性 | EIP-2929, EIP-3529 | Layer 1-2 | 重大改进 |
| 合约升级安全性 | EIP-1967, EIP-6780 | Layer 5 | 重大改进 |
| 交易批量操作 | EIP-3074, EIP-7702 | Layer 1 | 结构性解决 |
| Token 标准碎片化 | ERC-20/721/1155 | Layer 1 | 结构性解决 |
| 存储 Slot 冲突 | EIP-1967, EIP-7201 | Layer 5 | 结构性解决 |
| 签名验证灵活性 | EIP-7212, EIP-4337 | Layer 4 | 结构性解决 |
| 交易失败扣费体验 | 无直接 EIP | Layer 1-2 | 部分改进 |
| 合约间调用透明性 | EIP-3155 | Layer 3 | 部分改进 |

