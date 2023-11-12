Hacash 全节点 RPC API 文档
===

此文档包含区块扫描、交易转账查询、账户余额查询、区块钻石信息查询、创建新账户、创建转账交易等接口的调用规范和示例，是开发Hacash区块链浏览器及对接交易所等功能所必须的接口支持。

本文档内将提供示例测试接口（临时可用，但随时可能关闭），可以帮助你即时查看接口返回的内容，或者临时做测试、调试使用。在生产环境中，请**务必**启用自己的服务器并搭建全节点，才能确保接口的稳定可用和安全性。全节点搭建教程见 [hacash.org](https://hacash.org/)。

下载最新版本的全节点程序，并启动程序同步完所有区块之后，要启用本 RPC API 服务，需要在配置文件（hacash.config.ini）内加上如下配置：

```ini

[service]
enable = true
rpc_listen_port = 8083

```

以上配置 `enable = true` 表示启用 RPC 接口服务，`rpc_listen_port = 8083` 表示监听的 http 服务端口为 8083 。

此时访问 `http://127.0.0.1:8083/` ，正常情况下你将看到如下返回：

```json
{
    "ret": 0,
    "service": "hacash node rpc"
}
```

此时表示 RPC 服务已经正常运行。

### 简单示例、快速入门

在文档正式开始之前，我们来测试一个简单的示例。在全节点保持运行的情况下，访问 [http://127.0.0.1:8083/query?action=balances&address_list=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS](http://127.0.0.1:8083/query?action=balances&address_list=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS)， 这是一个查询账户地址余额的接口，正常情况下将会返回例如：

```json
{
    "list": [
        {
            "diamond": 1016,
            "hacash": "ㄜ1,474,845:244",
            "satoshi": 0
        }
    ],
    "ret": 0
}
```

可以看到，Hacash 的 RPC API 服务采用标准的 http 接口方式，你可以简单的在浏览器中或你开发的程序中很方便、简单的使用它。

下面的文档将使用 hacash.org 提供的测试接口作为示例，访问 [http://rpcapi.hacash.org/query?action=balances&address_list=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS](http://rpcapi.hacash.org/query?action=balances&address_list=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS) 应该返回和上面相同的接口数据内容。

示例二：[http://rpcapi.hacash.org/query?action=balances&address_list=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS,1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9&unitmei=true](http://rpcapi.hacash.org/query?action=balances&address_list=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS,1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9&unitmei=true) 表示同时查询两个地址的余额，且返回的余额单位为“枚”。

### 接口格式、通用参数

采用标准的 HTTP 接口请求和应答方式，包含 4 个路径：

   1. /create  生成数据 ， GET 方式 ， 如创建账户、生成转账交易等
   2. /submit  提交数据 ， POST 方式 ， 如提交交易进待确认池等
   3. /query   查询数据 ， GET 方式 ， 如查询账户余额等
   4. /operate 修改数据 ， GET/POST 方式 ， 如修改系统运行参数等
   
多个接口通用的参数介绍如下：

| 参数名 | 类型 | 默认值 | 示例值 | 功能简介 |
| ----  | ----  | ----  | ----  | ----  |
| action | string | -    | balances, diamond, block_intro, accounts ... | 要查询、生成的数据类型，定义接口的功能 |
| unitmei | bool  | false | true, false, 1, 0 | 是否采用浮点数的“枚”为单位传递或返回数额，否则采用Hacash标准化单位方式。 例如使用“枚”为单位："12.5086"，而标准化单位："ㄜ125086:244" |
| unit | string  | - | mei,zhu,shuo,ai,miao | 返回的 HAC 使用单位 枚、铢、烁、埃、渺 |
| kind | menu  | - | h, s, d, hs, hd, hsd | 筛选返回的账户、交易信息类型。h: hacash, s: satoshi, d: diamond。用途：比如在扫描区块时只需要返回 HAC 转账内容而忽略其他两者，则传递 `kind=h` 即可。 |
| hexbody | bool  | false | true, false, 1, 0 | `/submit` 在提交数据时，是否使用 hex 字符串的形式为 Http Body。默认为原生 bytes 形式。 |


### 返回值、公共字段

采用标准 JSON 格式应答所有请求。公共字段如下：

```js

{
  "ret": 0,  // 表示返回类型， 0 为正确，  >= 1 则表示发生错误或者查询不存在
  "list": [...] // 某些返回列表数据的接口将使用
}

```

下面将介绍每一个具体接口的参数传递和返回值细节。

---

## 1. /create 创建

#### 1.1 创建账户 `GET: /create ? action=accounts`

随机批量批量创建账户，返回包含私钥、公钥和地址的账户信息列表。可传递参数：

1. number [int] 表示批量创建账户的数量，默认为 1， 最大值 200

示例接口： [http://rpcapi.hacash.org/create?action=accounts&number=5](http://rpcapi.hacash.org/create?action=accounts&number=5)

返回值：

```js
{
    list: [
        {
            address: "1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS",  // 账户地址
            prikey: "2e50243243abc2e41f3b2ae90029640e235d884a88cfb5ea3e4d0e9efbae6710",  // 私钥
            pubkey: "03e22fc27a0d7ae325fa024875febd58266b8b6adbfb966116c9ba958ff5bad7e6"  // 公钥
        },
        ...
    ],
    ret: 0
}
```

特别注意：系统采用随机生成算法创建账户，系统并不会保留或记录本接口创建的账户私钥，请务必备份储存好创建的私钥，并做好安全防护。

#### 1.2 创建转账交易 `GET: /create ? action=value_transfer_tx`

本接口用于创建HAC、单向转移的比特币和区块钻石的转账交易，基本参数如下：

1. unitmei [bool] 是否采用单位“枚”浮点数形式，去解析传递的数额参数
2. main_prikey [hex string] 主地址/手续费地址的私钥hex字符串
3. fee [string] 交易将要给出的手续费值，例如 "0.0001" 或 "ㄜ1:244"
4. timestamp [int] 交易的时间戳；可选传递；不传时则自动设为当前时间戳
5. transfer_kind [menu] (hacash, satoshi, diamond) 要创建的交易类型，HAC转账、BTC转账还是区块钻石转账

【注意一】：当只有传递相同的 `timestamp` 时间戳参数，而且保持其它参数始终相同时，则每次创建的交易则具备相同的 hash 值，被视为同一笔交易。

【注意二】：由于 Hacash 系统支持对同一笔交易进行重复签名手续费竞价，所以仅改变 `fee` 字段并不会更改交易的 `hash` 值， 而只会改变其 `hash_with_fee` 值。

【注意三】：手续费 `fee` 字段不能设置得过小，否则将无法被整个系统接受，目前费用最小值为 0.0001 枚（即 ㄜ1:244 ）。请不要设置手续费低于这个值。

##### 1.2.1 创建HAC普通转账交易

传递参数 `transfer_kind=hacash`，且增加参数如下：

 - amount [string] 转账数额；单位格式视 `unitmei` 参数而定； 例如 "0.1" 或 "ㄜ1:247"。
 - to_address [string] 对方（收款）账户地址
 
调用接口示例： [http://rpcapi.hacash.org/create?action=value_transfer_tx&main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0001&unitmei=true&timestamp=1603284999&transfer_kind=hacash&amount=12.45&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS](http://rpcapi.hacash.org/create?action=value_transfer_tx&main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0001&unitmei=true&timestamp=1603284999&transfer_kind=hacash&amount=12.45&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS)

返回值如下：

```js
{
    // 公共参数
    ret: 0,
    // 交易的 hash 值
    hash: "6066cef4fe51669aec5b5596375dba11dafaf2560c4bd8c0432ac4ea98ff3ad1",
    // 交易包含 fee 的 hash 值
    hash_with_fee: "9cbc4821d0d921b429dbe4d6b67d6412aa5cae4b724f1e7dab9f870646cb1bb6",
    // 交易体、内容 的 hex 值
    body: "02005f90300700e63c33a796b3032ce6b856f68fccf06608d9ed18f401010001000100e9fdd992667de1734f0ef758bafcd517179e6f1bf60204dd00010231745adae24044ff09c3541537160abb8d5d720275bbaeed0b3d035b1e8b263cb73b724218f13c09c16e7065212128cf0c037ebb9e588754eb073886486d950607d59bef462d2731e15b667c6ff1f0badd6259c6f58d5ca7a5f75856b8cae8e80000",
    // 交易使用的时间戳
    timestamp: 1603284999
}
```

在生产环境中，请在数据库中保存以上的返回值，以便进行对账，或者在区块链网络延迟时重新提交未被确认的交易。上面的内容并不会泄露你的私钥，而仅仅是签名后的交易数据，请放心储存。

创建BTC转账和区块钻石的转账，调用接口的的返回值与上者相同。

##### 1.2.2 创建单向转移的 Bitcoin 普通转账交易

传递参数 `transfer_kind=satoshi`，且增加参数如下：

 - amount [int] 要支付的比特币数额，单位为“聪”、“satoshi” (0.00000001枚比特币)；例如转账10枚比特币则传递 "1000000000"，转账0.01枚则传递"1000000"；系统不支持低于 1 聪的比特币单位。
 - to_address [string] 对方（收款）账户地址

例如给某个地址转账一枚比特币，示例接口如：[http://rpcapi.hacash.org/create?action=value_transfer_tx&main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0001&unitmei=true&timestamp=1603284999&transfer_kind=satoshi&amount=100000000&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS](http://rpcapi.hacash.org/create?action=value_transfer_tx&main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0001&unitmei=true&timestamp=1603284999&transfer_kind=satoshi&amount=100000000&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS)

##### 1.2.3 创建区块钻石转账交易

传递参数 `transfer_kind=diamond`，且增加参数如下：

 - diamonds [string] 逗号分割的钻石字面值，例如 "EVUNXZ,BVVTSI"，可传一个或多个，最多一次批量转移200枚钻石
 - diamond_owner_prikey [hex string] 选填，付款（支付钻石、钻石所有者）的账户私钥；如果不传则默认为 `main_prikey`
 - to_address [string] 对方（收取钻石）账户地址

示例接口调用：[http://rpcapi.hacash.org/create?action=value_transfer_tx&main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0003&unitmei=true&timestamp=1603284999&transfer_kind=diamond&diamonds=EVUNXZ,BVVTSI&diamond_owner_prikey=EF797C8118F02DFB649607DD5D3F8C7623048C9C063D532CC95C5ED7A898A64F&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS](http://rpcapi.hacash.org/create?action=value_transfer_tx&main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0003&unitmei=true&timestamp=1603284999&transfer_kind=diamond&diamonds=EVUNXZ,BVVTSI&diamond_owner_prikey=EF797C8118F02DFB649607DD5D3F8C7623048C9C063D532CC95C5ED7A898A64F&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS)


---

## 2. /submit 提交

#### 2.1 提交交易至交易池 `POST: /submit ? action=transaction`

提交一笔交易至全网的内存池内。

url 参数如下：

 - hexbody [bool] 以 hex 字符串的形式传递 post body 值；否则以原生 bytes 形式传递
 
调用后返回值如下

```js
{
    ret: 0 // ret = 0 表示返回成功
}
```

或者返回错误

```js
{
    ret: 1 // ret = 1 表示提交交易错误
    // 例如余额不足，错误消息如下：
    errmsg: "address 1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9 balance ㄜ0:0 not enough， need ㄜ1,245:246."
}
```
curl 命令行测试调用示例：

```shell script
curl "http://rpcapi.hacash.org/submit?action=transaction&hexbody=1" -X POST -d "02005f90300700e63c33a796b3032ce6b856f68fccf06608d9ed18f401010001000100e9fdd992667de1734f0ef758bafcd517179e6f1bf60204dd00010231745adae24044ff09c3541537160abb8d5d720275bbaeed0b3d035b1e8b263cb73b724218f13c09c16e7065212128cf0c037ebb9e588754eb073886486d950607d59bef462d2731e15b667c6ff1f0badd6259c6f58d5ca7a5f75856b8cae8e80000"
```

手续费竞价说明一：交易提交后，当需要付出更高的手续费以作为钻石竞价，或者提高手续费以加快交易确认时，可调用 `/create` 接口再次创建相同的转账交易，仅仅修改数额更高的 `fee` 字段（其它全部字段包括时间戳都必须保持一致，否则就是不同的两笔交易），然后重新提交这笔交易，全网就会丢弃手续费出价低的，而打包这一笔手续费出价更高的交易。所有不同的手续费出价都视为一笔交易的不同状态，无论出价多少次，最终只会有一次打包。

手续费竞价说明二：当查询到交易仍然在交易池内时，可以调用 `/operate` 接口更方便的修改手续费竞价，而不用重新提交整个交易内容。接口说明见文档下方内容。

---

## 3. /query 查询

#### 3.1 查询账户余额 `GET: /query ? action=balances`

一次查询一个或多个账户的HAC、BTC、钻石余额。

传递参数：

 - address_list [string] 逗号隔开的账户地址列表；最多200个批量查询
 - unitmei [bool] 是否以“枚”为单位返回浮点数字符串；否则返回ㄜ标准单位
 - kind [menu] 查询返回的种类；`kind=h`只返回HAC余额， `kind=hs`返回HAC、BTC余额，不传或传递`hsd`则返回全部类型余额
 
 示例接口：[http://rpcapi.hacash.org/query?action=balances&unitmei=1&address_list=18Yt6UbnDKaXaBaMPnBdEHomRYVKwcGgyH,1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF](http://rpcapi.hacash.org/query?action=balances&unitmei=1&address_list=18Yt6UbnDKaXaBaMPnBdEHomRYVKwcGgyH,1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF)
 
 示例返回值：
 
 ```js
{
    ret: 0, // 通用返回值
    list: [
        {
            diamond: 350, // 拥有钻石的数量
            hacash: "3101.0826", // HAC余额
            satoshi: 0 // BTC余额，单位“聪”：satoshi
        },
        {
            diamond: 0,
            hacash: "3842.24693214",
            satoshi: 0
        }
    ]
}
```
注意：返回的余额列表与传递的账户列表位置相对应。

#### 3.2 查询区块钻石信息 `GET: /query ? action=diamond`

查询一枚钻石的哈希、产生的区块高度，当前所属地址等详细信息；可以通过钻石字面值（name）或者标号（number）来查询。

传递参数：

 - name [string] 钻石字面值，例如： "ZAKXMI"
 - number [int] 钻石标号，例如： "20001"
 - unitmei [bool] 可选，是否以单位“枚”返回手续费竞价数额字段
 
示例查询接口一： [http://rpcapi.hacash.org/query?action=diamond&number=20001](http://rpcapi.hacash.org/query?action=diamond&number=20001)

示例查询接口一： [http://rpcapi.hacash.org/query?action=diamond&name=ZAKXMI](http://rpcapi.hacash.org/query?action=diamond&name=ZAKXMI)

以上两个接口返回内容相同：

```js
{
    ret: 0, // 公共返回值
    number: 20001, // 钻石编号
    name: "ZAKXMI", // 钻石名称/字面值
    approx_fee_offer: "ㄜ382:246", // 本颗钻石花费的竞价手续费
    nonce: "00000000c9babb01", // 挖掘的随机数
    contain_block_height: 179610, // 被挖出的区块高度
    contain_block_hash: "0000000010914f286cc035ee35ae0afc6294417adb775dddf25b3bd4aa22524b", // 被挖出的区块hash
    custom_message: "7d8367f6e46e9ffee311b9f3a38519d674b52407fd0aa287442715fe2f0c4db0", // 附加消息
    prev_block_hash: "000000000ae89f8a1c93fbffe9baad198fda287f13e39c37294eb7a7b617bd70", // 上一个包含钻石的区块hash
    miner_address: "1KcXiRhMgGcvgxZGLBkLvogKLNKNXfKjEr", // 首次挖出的矿工地址
    current_belong_address: "1KcXiRhMgGcvgxZGLBkLvogKLNKNXfKjEr" // 钻石当前的归属地址
}

```

可以都看到，以上两个接口查询的实质上是同一枚钻石。

#### 3.3 查询最新一个有效区块信息 `GET: /query ? action=last_block`

查询当前区块链的最新有效区块信息。

参数：

 - info [bool] 是否返回详细信息；否则仅仅返回 height 高度

示例请求：[http://rpcapi.hacash.org/query?action=last_block](http://rpcapi.hacash.org/query?action=last_block)

返回值： 

```js
{
    ret: 0,
    height: 181719, // 当前已经写入区块链生效的区块高度
    
    // 传递 info 字段后更多返回如下
    hash: "0000000003ba7ed589b5881ea1852276cc7feb9b6a18dc3d9dbdc3d110055bd6",
    mrkl_root: "5a28df311c7fb707914ff50394484e2079036284e38cbd1eaf811ab8c2ff34b1",
    prev_hash: "000000000c57ae62d74d7f05de67b949899f6115cac52441b109241d5da2edca",
    timestamp: 1603341936, // 区块创建时间戳
    transaction_count: 0 // 区块包含交易数量
}
```

#### 3.4 查询指定区块信息 `GET: /query ? action=block_intro`

参数：

 - height [int] 可选，通过指定高度查询区块信息
 - hash [string] 可选，通过区块 hash 值查询区块信息
 - unitmei [bool] 可选，是否以单位“枚”返回区块奖励等数额字段
 
示例接口一: [http://rpcapi.hacash.org/query?action=block_intro&height=177255&unitmei=1](http://rpcapi.hacash.org/query?action=block_intro&height=177255&unitmei=1)

示例接口二: [http://rpcapi.hacash.org/query?action=block_intro&hash=000000001e76b27f70f9a5e5b0f9b9824b416c210548ffad298577ed5378cdcd](http://rpcapi.hacash.org/query?action=block_intro&hash=000000001e76b27f70f9a5e5b0f9b9824b416c210548ffad298577ed5378cdcd)
 
示例返回值：

```js
{
    ret: 0,
    coinbase: {
        address: "1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF", // 矿工地址
        reward: "ㄜ1:248" // 本区块奖励
    },
    height: 177255, // 区块高度
    hash: "000000001e76b27f70f9a5e5b0f9b9824b416c210548ffad298577ed5378cdcd", // 区块哈希
    mrkl_root: "d18df37a52c377115f053aca7822c65e5d154dac4ab6e062326dc770b5d1e5ad", // 默克尔根
    nonce: 3327878209, // 挖矿随机数
    difficulty: 3717887710, // 目标难度值
    prev_hash: "000000001868742351b8ab196444d22c0db2951c522ebc5b9ef057ad70a6556c", // 上一个区块hash
    timestamp: 1601931702, // 区块创建时间戳
    transaction_count: 5, // 包含交易数量
    version: 1 // 区块版本
}
```

#### 3.5 扫描每一笔交易获取转账信息 `GET: /query ? action=scan_value_transfers`

通过区块高度来遍历扫描每一笔交易，获取里面的转账内容，以确定交易所充币、提币等功能。

传递参数：

 - height [int] 要扫描的区块高度
 - txposi [int] 要扫描的交易位于区块中的数组索引位置，从0开始索引
 - txhash [string] 选传，交易哈希，如果传递了交易哈希则不用传 height、txposi 值即可直接查询交易，可用于交易是否确认的判断
 - unitmei [bool] 选传，是否以“枚”为单位返回浮点字符串
 - kind [menu] 查询返回的种类；`kind=h`只返回HAC转账， `kind=hs`返回HAC、BTC转账，不传或传递`hsd`则返回全部类型转账
 
 示例调用接口一：[http://rpcapi.hacash.org/query?action=scan_value_transfers&unitmei=1&height=180115&txposi=1&kind=hsd](http://rpcapi.hacash.org/query?action=scan_value_transfers&unitmei=1&height=180115&txposi=1&kind=hsd)
 
 示例调用接口二：[http://rpcapi.hacash.org/query?action=scan_value_transfers&txhash=6d678040e5d3d8104de1ea627fde1973ab9d0e036a5fe20913dfdd6192dec266](http://rpcapi.hacash.org/query?action=scan_value_transfers&txhash=6d678040e5d3d8104de1ea627fde1973ab9d0e036a5fe20913dfdd6192dec266)
 
 返回值示例：
 
```js
{
    ret: 0,
    type: 2, // 交易类型
    hash: "6d678040e5d3d8104de1ea627fde1973ab9d0e036a5fe20913dfdd6192dec266", // 交易哈希
    // main_adderss
    address: "1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF", // 交易手续费地址、主地址
    feepay: "ㄜ4:244", // 交易支付手续费
    feegot: "ㄜ4:243", // 被挖掘区块的矿工获得交易手续费（剩下部分被销毁了）
    // 说明：用户付出的手续费 和 矿工获得的手续费可能相同，也可能不同。但获得的一定不会大于付出的。
    timestamp: 1602764644, // 交易创建时间戳
    height: 103764, // 包含这笔交易的区块高度
    // 扫描转账类型示例如下：
    effective_actions: [
        {
            // 表示 main_adderss: 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 给 to_Address: 16DL2tFV2PxLx4RsjKLFeFXm7sued4UbPv 转账 15.38 枚HAC, 付款账户 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 支付手续费 0.0001 枚 HAC
            ai: 0, // ai字段表示 action_index ，表示此 action 在此笔交易中的数组索引，从0开始计算
            hacash: "ㄜ1538:246", // hacash 字段表示 HAC 转账 
            to: "16DL2tFV2PxLx4RsjKLFeFXm7sued4UbPv"
        },
        {
            // 表示 from_address: 18o6aqFiUDurPqie6zrNwV2pmafWyziZZP 给 main_adderss: 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 转账 50 枚 HAC， 收款账户 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 支付手续费
            ai: 2,
            hacash: "ㄜ50:248", // hacash 字段表示 HAC 转账 
            from: "18o6aqFiUDurPqie6zrNwV2pmafWyziZZP"
        },
        {
            // 表示 from_address: 1EM4j8xmxJosAi1N2RS2QL4VMe6x3agb7f 给 to_adderss: 1CYRSqhB7LgYXShHitxjBPaYcs4VxVWHyn 支付 128 枚 HAC ， 主账户 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 既不收款也不付款，仅仅为此笔转账提供手续费
            ai: 5,
            hacash: "ㄜ128:248", // hacash 字段表示 HAC 转账 
            from: "1EM4j8xmxJosAi1N2RS2QL4VMe6x3agb7f",
            to: "1CYRSqhB7LgYXShHitxjBPaYcs4VxVWHyn"
        },
        {
            // 表示钻石的挖出/创造，矿工地址为 18Yt6UbnDKaXaBaMPnBdEHomRYVKwcGgyH ，钻石字面值为 SWBAVY ，编号为 20094
            ai: 6,
            diamond: "SWBAVY",
            miner: "18Yt6UbnDKaXaBaMPnBdEHomRYVKwcGgyH",
            number: 20094
        },
        {
        	// 表示将 main_adderss: 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 拥有的钻石 XMYVEM 转移给 to_address: 1CYRSqhB7LgYXShHitxjBPaYcs4VxVWHyn, 由 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 来负担此笔交易的手续费
            ai: 8,
            diamonds: "XMYVEM",
            to: "1CYRSqhB7LgYXShHitxjBPaYcs4VxVWHyn"
        },
        {
            // 批量转移钻石：表示将 main_adderss: 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 拥有的三枚钻石 WUZXYM 和 IZHTEW 和 IIUMWH 都一次性转移给 to_address: 1Bjj8rijMJSD6dRC6eSHFs6C3E1JzcTfyQ, 由 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 来负担此笔交易的手续费； 一次批量转移最多 200 枚钻石
            ai: 9,
            diamonds: "WUZXYM,IZHTEW,IIUMWH",
            to: "1Bjj8rijMJSD6dRC6eSHFs6C3E1JzcTfyQ"
        },
        {
            // 批量转移钻石：表示将 from_adderss: 1GRyscUQ2poZPKrR4wCxrkJqMrM8Wqnvm1 拥有的2枚钻石 WUZXYM 和 IZHTEW 都一次性转移给 to_address: 19Co8BRxmh5uHBemvd8e9BbpyXJbRfeSiS, 主账户 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 既不收钻石也不付出钻石， 仅仅负担此笔交易的手续费
            ai: 13,
            diamonds: "WUZXYM,IZHTEW",
            from: "1GRyscUQ2poZPKrR4wCxrkJqMrM8Wqnvm1",
            to: "19Co8BRxmh5uHBemvd8e9BbpyXJbRfeSiS"
        },
        {
        	// 表示将 from_adderss: 1JFo6YPy4niZPiBUBuKJQ3UQAe4LN8tCfw 拥有的钻石 WSNZAX 转移给 main_address: 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF, 由收款方 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 来负担此笔交易的手续费
            ai: 14,
            diamonds: "WSNZAX",
            form: "1JFo6YPy4niZPiBUBuKJQ3UQAe4LN8tCfw"
        },
        {
        	// 单向转移比特币起始交易，表示有多少枚比特币被转移进了Hacash主网
            ai: 17,
            btctrsno: 3, // 往Hacash内转移比特币的转账序号
            owner: "17XTDDHWjbUoKZmgFmdTg1m9UHYLuX4JVe", // 比特币所有者地址
            satoshi: 200000000 // 转移的比特币数额，单位：“聪” satoshi， 此数值表示 2 枚比特币
        },
        {
        	// 比特币转账：从 main_address: 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 转给 to_address: 1KfVAAgSkfDVfr3WLkcsqBcptKwm2ffYhB 比特币 0.1 枚，由付款方主地址 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 负担此笔交易的手续费
            ai: 18,
            satoshi: 10000000, // 转移的比特币数额，单位：“聪” satoshi， 此数值表示 0.1 枚比特币
            to: "1KfVAAgSkfDVfr3WLkcsqBcptKwm2ffYhB" // 比特币收款地址
        },
        {
        	// 比特币转账：从 form_address: 1PXosXwBwcYTXS7wHq7agMJgYpHqdMYoG5 转给 main_address: 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 比特币 0.05 枚，由收款方地址 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF 负担此笔交易的手续费
            ai: 19,
            satoshi: 5000000, // 转移的比特币数额，单位：“聪” satoshi， 此数值表示 0.05 枚比特币
            form: "1PXosXwBwcYTXS7wHq7agMJgYpHqdMYoG5" // 比特币付款地址
        },
        {
        	// 比特币转账：从 form_address: 1PXosXwBwcYTXS7wHq7agMJgYpHqdMYoG5 转给 to_address: 1EM4j8xmxJosAi1N2RS2QL4VMe6x3agb7f 比特币 1.2 枚，主地址 1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF ，不付款也不收款， 仅仅负担此笔交易的手续费
            ai: 20,
            satoshi: 120000000, // 转移的比特币数额，单位：“聪” satoshi， 此数值表示 1.2 枚比特币
            form: "1PXosXwBwcYTXS7wHq7agMJgYpHqdMYoG5", // 比特币付款地址
            to: "1EM4j8xmxJosAi1N2RS2QL4VMe6x3agb7f" // 比特币收款地址
        }
    ]
}
```

使用 `scan_value_transfers` 接口注意：

 - 无论转账 HAC、BTC 还是区块钻石，都只能使用 HAC 支付手续费
 - 转移 BTC 的单位为“聪 satoshi”，  1 BTC = 100000000 satoshi
 - 通过 `hacash`、`satoshi` 和 `diamonds` 字段，以及 `miner`、`owner` 来判断转账类型
 - `ai` 字段表示 `action_index` ，是转账在交易内的数组索引位置，而不是类型type
 - 通过 `from` 和 `to` 两个地址字段的检查，来判断转账的资金流向
 - Hacash 系统在转账时，支持付款方、收款方或任意不相关第三方来支付交易确认手续费
 - 一笔交易可包含多种类型、多种数额、多比款项的转账，一笔交易就是一份综合性的支付合约
 - 在将来有可能增加其他种类的的转账类型


#### 3.6 扫描每一笔交易获取简要转账操作 `GET: /query ? action=scan_coin_transfers`

`scan_value_transfers` 接口返回的是更加丰富且结构化的内容， 如果仅仅需要获取HAC、HACD和BTC的转账动作，用于交易所充值等简单需求，可以使用 `scan_coin_transfers` 接口，内容更加简洁。
传递参数：

- height [int] 要扫描的区块高度
- txposi [int] 要扫描的交易位于区块中的数组索引位置，从0开始索引
- txhash [string] 选传，交易哈希，如果传递了交易哈希则不用传 height、txposi 值即可直接查询交易，可用于交易是否确认的判断
- unitmei [bool] 选传，是否以“枚”为单位返回浮点字符串
- kind [menu] 查询返回的种类；`kind=h`只返回HAC转账， `kind=hs`返回HAC、BTC转账，不传或传递`hsd`则返回全部类型转账
- from [string] 要筛选的 from 地址，传递后仅仅会展示传入的地址，而忽略其他 from 地址
- to [string] 要筛选的 to 地址，传递后仅仅会展示传入的地址，而忽略其他 to 地址

示例调用接口一：[http://rpcapi.hacash.org/query?action=scan_coin_transfers&unitmei=1&height=180115&txposi=1&kind=hsd](http://rpcapi.hacash.org/query?action=scan_coin_transfers&unitmei=1&height=180115&txposi=1&kind=hsd)

示例调用接口二：[http://rpcapi.hacash.org/query?action=scan_coin_transfers&txhash=6d678040e5d3d8104de1ea627fde1973ab9d0e036a5fe20913dfdd6192dec266](http://rpcapi.hacash.org/query?action=scan_coin_transfers&txhash=6d678040e5d3d8104de1ea627fde1973ab9d0e036a5fe20913dfdd6192dec266)

返回值示例：

```js
{
    hash: "43e98177bc426a2f15c15fe3e8968ece1b2e0829eab7cacb8715c89082f48aef",
    height: 490194,
    ret: 0,
    timestamp: 1698278005,
    transfers: [
        {
            // 默认都会显示 from 和 to 地址，更加方便查询
            from: "13RnDii79ypWayV8XkrBFFci29cHtzmq3Z",
            to: "1EcrtFAUmVeLnGeaEcMDoPZH7ZPysks1H2",
            hacash: "0.9370996"
        },
        {
            from: "13RnDii79ypWayV8XkrBFFci29cHtzmq3Z",
            to: "1EcrtFAUmVeLnGeaEcMDoPZH7ZPysks1H2",
            sotoshi: "5000000"
        },
        {
            from: "13RnDii79ypWayV8XkrBFFci29cHtzmq3Z",
            to: "1EcrtFAUmVeLnGeaEcMDoPZH7ZPysks1H2",
            diamond: "WUZXYM"
        },
        {
            from: "13RnDii79ypWayV8XkrBFFci29cHtzmq3Z",
            to: "1EcrtFAUmVeLnGeaEcMDoPZH7ZPysks1H2",
            diamond: "WUZXYM,IZHTEW,IIUMWH" // 多枚钻石
        }
    ],
    type: 2
}

```

#### 3.7 查询总供应量 `GET: /query ? action=total_supply`

 示例接口：[http://rpcapi.hacash.org/query?action=total_supply](http://rpcapi.hacash.org/query?action=total_supply)
 
 示例返回值：
 
 ```js
{
    ret: 0, // 通用返回值
    minted_diamond: 27538, // 当前已经成功挖掘出的钻石数量
    transferred_bitcoin: 0, // 成功转移到hacash主网的比特币数量，单位：枚
    block_reward: 240213, // 区块奖励HAC累计
    channel_interest: 151.6902713463587, // 通道锁定利息HAC累计
    btcmove_subsidy: 0, // BTC转移增发HAC已解锁部分累计
    burning_fee: 0, // 被燃烧销毁的手续费
    current_circulation: 240364.69027134636, // 当前流通供应量
    located_in_channel: 0, // 被锁定在通道链内的HAC实时统计

}
```

---

## 4. /operate 修改、操作

#### 4.1 提升待确认交易手续费 `GET: /operate ? action=raise_tx_fee`

当一笔交易存在于交易池中还没有被确认时，可以重新签名提升（但不能降低）此笔交易的手续费，以达到加快交易确认或者钻石竞价报价的作用。

传递参数：

 - txhash [string] 要提升手续费的交易的 hash 值
 - fee [string] 要修改的目标手续费
 - unitmei [bool] 可选，是否以单位“枚”为手续费 `fee` 字段传递的单位
 - fee_prikey [string] 手续费地址的私钥
 
示例接口访问： [http://rpcapi.hacash.org/operate?action=raise_tx_fee&fee_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=ㄜ32:247&txhash=ad26a35116664176426f3c08adad147577b9a85999cb89d465becf6a27002c04](http://rpcapi.hacash.org/operate?action=raise_tx_fee&fee_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=ㄜ32:247&txhash=ad26a35116664176426f3c08adad147577b9a85999cb89d465becf6a27002c04)

接口返回示例：

```js
{
    ret: 0,
    status: "success" // 表示提升手续费成功
}
```

失败的返回示例：

```js
{
    ret: 1,
    errmsg: "Tx fee address password error: need 18Yt6UbnDKaXaBaMPnBdEHomRYVKwcGgyH but got 1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9" // 失败原因：手续费地址不一致
}

```
 
