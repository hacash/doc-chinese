HIP-15 股权账户模型和可读合约语法树抽象
===

Hacash 白皮书描述了一种名为 [层级股权控制账户](https://github.com/hacash/paper/blob/master/whitepaper.cn.md#3-%E5%B1%82%E7%BA%A7%E8%82%A1%E6%9D%83%E6%8E%A7%E5%88%B6%E8%B4%A6%E6%88%B7) 的新账户模型，是一种比多签管理更加灵活和强大的账户类型，能够适应现代金融系统组织化、机构化的资产管理方式。

“账户抽象”是以太坊近期受到较多关注的概念，甚至作为了以太坊下一步重大升级。这个升级的存在是因为以太坊在创造时主要以“世界计算机”为目标，着重设计了有关“计算”的部分，实现了图灵完备的智能合约，但在资产的账户模型和控制权限等方面，并未多加考虑，甚至有所忽略。

所谓账户抽象，核心就是资产管理权限的抽象。其原理是在更高的逻辑层面实现“账户”与“合约”的统一，让两者在系统内都成为相同的一等公民，或者说就是变成同一个东西，只是验证转账权限的方式不同，有些是直接通过传统的椭圆曲线验签方式，而另一些则变更为合约代码编程的自定义验证逻辑。而不是像现在这样，账户系统被区分为 EOA (Externally Owned Accounts) 外部账户和 CA (Contract Accounts) 合约账户两种。

从技术上来说，以太坊想要达成的账户抽象升级，其目的就是为了实现类似于Hacash的层级股权控制账户的功能。结合 Hacash 的 [可读金融合约](https://github.com/hacash/paper/blob/master/tech/readability_contract_introduction_cn.md) ，作为以太坊的重大升级，“账户抽象”可以在Hacash内天然得到支持，并且是以最成熟的技术架构，而非需要顾忌到兼容性的打补丁技术。

由此，HIP-15 被提出，目标是将 Hacash 已经具备且可用的可读金融合约进行一个概念上的升级，称之为“合约抽象语法树”，合约的解释器从对一维组合式金融合约的解释，升级成对可为树状结构的程序逻辑语法树的解释。合约的可读性和简洁性继续被保留，但增加了一系列用于逻辑控制和基础运算的 `action`或者 `OP_CODE`, 能够将可读金融合约直接升级为可执行通用计算的“语法树”。这将带来合约可编程性的巨大提升，同时又能够规避基于常见的栈式虚拟机（比如EVM）的各方面弱点、复杂性和安全性问题。

这某种意义上成为了一个在区块链可编程性方面具备重大意义的概念：人类可读的金融合约和执行通用计算的可编程性得到了一致抽象和完美统一。

更为重要的是，这项技术也同时解决了Hacash体系内的多个需求：


1. 让原本的股权控制账户功能避免了被硬编码进Hacash核心，可以通过用户端编程来实现，在控制逻辑和参数设置上进行充分的自定义。


2. 立即达成了“账户抽象”目标，统一了普通账户和合约账户，让更多在以太坊分裂的账户模型下无法满足的业务场景得到支持。


3. Hacash 可读金融合约的可编程性获得了巨大提升，能够支持更多的金融业务。一些更频繁的需求可以得到先期测试和满足，然后作为Hacash核心层“标准化封装”的决策、执行依据。


4. 能够更灵活地支持更多扩容目标，例如各类 Rollup 或 L3 扩容，或者在一层进行资产发行，比如 HRC-20 协议。


同时，HIP-15只需要在已有的可读金融合约的基础上进行较少的改进，且保持了向前兼容性，主网核心的简洁性和去中心化程度也将不受影响。

这里可以有一个简单的用于演示对股权控制地址进行“账户抽象”的合约代码：

```js

/* 定义合约解释器兼容版本 */
pragma vm 1.2.31

/* 定义一个股权控制合约/账户，在“账户抽象”概念下，合约就是账户，账户就是合约，两者为同一个东西 */
contract MyMultiControlAccount {

	/* 合约常量 */
	const minimum_permission_ratio = 51 // 最小的控制权百分比
	
	/* 合约变量 */
	var address restore_account = "1qkdhtn......" // 恢复账户，用于紧急情况的避险


	/* 仅部署时调用一次 */
	constructor(address restore_acc) {
		restore_account = rerestore_acc // 设置恢复账户
	}

	/* 鉴权许可，每次需要本地址验证权限时调用 */

	// 股权管理账户权限验证控制逻辑，每次需要动用资产时执行验证
	// 由四个地址通过不同的投票权控制本账户的资产
	permit() {
		var votes = 0
		// reqsign 函数表示验证地址的签名，签名即表示投了赞成票
		if(reqsign(addr1)) {
			votes += 30 // 投票累加
		}
		if(reqsign(addr2)) {
			votes += 30
		}
		if(reqsign(addr3)) {
			votes += 25
		}
		if(reqsign(addr4)) {
			votes += 15
		}
		if(votes < minimum_permission_ratio) {
			return false // 票数不够，鉴权失败
		}
		// 签名票数过半，鉴权通过
		return true 
	}

}

```

合约的调用和执行势必需要在区块链上预先储存合约代码，如果不加以控制，对状态空间的大量占用将成为影响主网去中心化的严重缺陷。以太坊全节点的“状态爆炸”问题已经无可挽回，预计多年以后以太坊也将变成只在谷歌或亚马逊的云数据库里运行和存放账本的“机房链”。

解决此问题的方案有两种：


1. 严格控制合约对状态空间的占用，比如每个区块只允许10kb的合约空间，将不重要的数据和计算挤出到2层或3层扩容设施中去

2. Nervos区块链对状态空间“租用”和“释放”的机制设计可以参考





