Hacash SDK 开发者文档
===

中心化交易所、第三方钱包等以普通方式接入 Hacash 区块链时，全节点的 [ API 接口](https://github.com/hacash/doc-chinese/blob/main/server/fullnode_api_doc.md) 已经包含了账户、转账的创建及交易的查询、提交等功能，可以满足基础开发需要。

全节点在进行最新交易和区块的查询及提交新交易时，需要保持联网运行以同步到最新区块，这对于金额较大、安全要求较高的业务来说，将始终存在暴露在风险下的可能性。可以单独运行一个不联网、不同步区块的全节点软件，并将其隔离在内网环境下，仅用来创建创建账户、构建转账交易和签署交易，然后通过另一个保持在线的全节点来广播交易。将全节点配置文件内的 boots 行删除并运行在无网络环境下即可，这种“前、后双节点”的方法，能够避免私钥触网的潜在风险。

不过，运行两个全节点的方式虽然非常简单，其 HTTP API 接口能够用于任何语言开发的服务器环境，但对于嵌入式系统、小型专用设备和前端浏览器等特殊环境来说，运行全节点十分困难甚至完全不可行。这时，以 SDK 开发方式集成 Hacash 则是实现高安全性和丰富定制化的最佳技术方案，特别是，提供可以运行在浏览器中的 SDK ，将极大的提升 Hacash 生态方面的技术基础。

### Go & Rust SDK

截止到目前，Hacash 提供三种语言的 SDK ，分别为 Go 、Rust 和 WASM 。Hacash 的全节点软件早期采用 Golang 语言开发，后来整体切换到 Rust ，这两种语言都可以编译为 WASM ，并且也都实现了这两种语言的 Hacash WASM SDK 的编译。

Golang 版本的全节点目前已经停止迭代和开发，但其已经实现的基础功能，比如账户创建、转账构造、交易签名等代码仍然安全可用。采用 Go 语言开发的系统可以直接通过引入代码包的方式集成，有两处源码，可用于开发参考：

1. [Golang 全节点 API 源码](https://github.com/hacash/service/blob/master/rpc)，示例：[构建转账交易](https://github.com/hacash/service/blob/master/rpc/createTransferTx.go)
2. [Golang 编译为 WASM SDK 源码](https://github.com/hacash/jssdk/blob/main/wasmsdk/hac_transfer.go)，示例：[构建 HAC 转账交易](https://github.com/hacash/jssdk/blob/main/wasmsdk/hac_transfer.go)

Rust 也实现了这两者，也可以通过直接参考源码来进行集成开发：

1. [Rust 全节点 API 源码](https://github.com/hacash/fullnode/tree/main/server/src/api)，示例：[构建转账交易](https://github.com/hacash/fullnode/blob/main/server/src/api/create_transfer.rs)
2. [Rust 编译为 WASM SDK 源码](https://github.com/hacash/fullnode/tree/main/sdk/src)，示例：[构建转账交易](https://github.com/hacash/fullnode/blob/main/sdk/src/coin.rs)

Go 和 Rust 的 SDK 不提供单独下载的代码包，而是通过全节点源码一并开源发布。

### WASM SDK 

目前采用 Rust 来编译构建 WASM SDK ，用于在 Node.js 服务器、网页浏览器内或任何其他 JavaScript 语言环境内进行 Hacash 相关业务开发，实现创建账户、构造转账或签署交易等功能。

通过以下 crate 、文档和测试用例，在安装了 Rust 开发环境的情况下，可自己进行 WASM SDK 的编译，以及基于 SDK 的开发：

1. [Rust WASM SDK Crate](https://github.com/hacash/fullnode/tree/main/sdk)
2. [WASM SDK Build Doc](https://github.com/hacash/fullnode/tree/main/sdk/README.md)
3. [WASM SDK Example](https://github.com/hacash/fullnode/tree/main/sdk/tests/test.html)

另外，Hacash 也会在每次全节点发布的同时，附带发布编译好的 WASM SDK 包，可直接下载使用。其区分为 Nodejs 、Web 和 单 JS 文件版面，供不同的开发环境使用。

- [Rust Fullnode Releases](https://github.com/hacash/fullnode/releases)

### 加载 WASM SDK

以单页面为例，在 HTML 内引入编译和打包好，附带 WASM 代码的 JS 文件：

```html
<script src="./hacashsdk_bg.js"></script>
```

成功加载文件后，即可使用全局函数 `hacash_sdk()` 通过 async/await 方式加载和初始化 Hacash SDK:

```js
async function run() {
    let sdk = await hacash_sdk()
    console.log(sdk)
    do_test(sdk)
};
run()
```

或者：

```js
function do_test(sdk) {
    console.log(sdk)
}
hacash_sdk().then(do_test)
```

Node.js 环境在加载 js 模块时将自动初始化 WASM SDK，Web 方式则采用标准的 wasm-bindgen 方式加载。请阅读 [wasm-bindgen 文档](https://wasm.rust-lang.net.cn/docs/wasm-bindgen/) 了更多。


### SDK 文档

#### 1. 创建账户 create_account

```js
// password
let pass_or_prikey_hex = "123456"
// or private key
// let pass_or_prikey_hex = "8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92"
let account = sdk.create_account(
    pass_or_prikey_hex
)
console.log(account)
```

`account` 返回值示例：
```js
{

    prikey: "8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92",
    pubkey: "0231745adae24044ff09c3541537160abb8d5d720275bbaeed0b3d035b1e8b263c",
    address_hex: "00e63c33a796b3032ce6b856f68fccf06608d9ed18",
    address: "1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9",
}
```

#### 2. 验证地址有效性 verify_address

```js
let addr = "1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9"
let result = sdk.verify_address(addr)
```

`result` 返回值示例：

```js
{
    ok: true, // 地址是否有效
    error: "...some error tip...", // 如果无效的错误信息
}
```

#### 3. 生成转账交易 create_coin_transfer

```js

let trsobj = new sdk.CoinTransferParam()
// print(trsobj.timestamp.toString())
trsobj.timestamp = BigInt(1755223764)
trsobj.main_prikey = "123456"
trsobj.to_address = "1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9"
trsobj.fee = "1:244"
trsobj.hacash = "13.5" // HAC
// trsobj.satoshi = BigInt(12000000) // BTC
// trsobj.diamonds = "AAABBB,WWTTSS" // HACD name list
// prints(trsobj, trsobj.timestamp.toString())
let txres = sdk.create_coin_transfer(trsobj)
console.log(txres.body)

```

`txres` 的返回值示例：

```js
{
    hash: "5a86aff3ca12020c929280e686cf0c23a3f58d969eef6ecad3c29a3a6ddc6c92", // tx hash
    hash_with_fee: "...",
    body: "541537160abb8d5d720275bbaeed0b3d......", // tx body
    timestamp: 1755223764n, // tx timestamp
}
```

`body` 值为已经包含了签名的交易体，可以直接向全节点提交。


#### 4. 签署一笔交易 sign_transaction

```js

let stps = new sdk.SignTxParam()
stps.prikey = "abc123"
stps.body = "0200689e96d400e63c33a796b3032ce6b856f68fccf06608d9ed18f401010002000100e63c33a796b3032ce6b856f68fccf06608d9ed18f8010c000a00e63c33a796b3032ce6b856f68fccf06608d9ed180000000000b71b0000010231745adae24044ff09c3541537160abb8d5d720275bbaeed0b3d035b1e8b263c9b607f2bd9e1031536c13741facb78585755c116aa7d10628ebc2adbb4be96493bc1bb8ac6c3e78dee6717b9c4a27280b698efc91097d5900418a59c9d8e7ac30000" // tx body
try {
    let resg = sdk.sign_transaction(stps)
    console.log("sign res:", resg)
    console.log("tx body:", resg.body)
}catch(e) {
    // if something error
    console.log(e)
}
```

`resg` 返回值示例：

```js
{

    hash:          "...", // tx hash
    hash_with_fee: "...",
    body:          "...", 
    signature:     signature.signature.hex(),
    timestamp:     trs.timestamp().uint(),
}
```

---

如果需要更多的 API 接口，或者参与开发，请 [提交 issue](https://github.com/hacash/fullnode/issues) 或者推送新的代码。


