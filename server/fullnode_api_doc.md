Hacash 全节点 API 文档
===


此文档包含区块扫描、交易转账查询、账户余额查询、区块钻石信息查询、创建新账户、创建转账交易等接口的调用规范和示例，是开发Hacash区块链浏览器及对接交易所等功能所必须的接口支持。

本文档内将提供示例测试接口（临时可用，但随时可能关闭），可以帮助你即时查看接口返回的内容，或者临时做测试、调试使用。在生产环境中，请**务必**启用自己的服务器并搭建全节点，才能确保接口的稳定可用和安全性。全节点搭建教程见 [hacash.org](https://hacash.org/)。

如果你需要编译和构建一个全节点，或者了解配置文件的每一个功能，请访问以下两个文档：

- [Build_compilation.md](https://github.com/hacash/doc/blob/main/build/build_compilation.md)
- [Configuration description](https://github.com/hacash/doc-chinese/blob/main/build/config_description.md)

下载最新版本的全节点程序，并启动程序同步完所有区块之后，要启用本 RPC API 服务，需要在配置文件（hacash.config.ini）内加上如下配置：

```ini

[server]
enable = true
listen = 8081

```

以上配置 `enable = true` 表示启用 API 接口服务，`listen = 8081` 表示监听的 http 服务端口为 8083 。

此时访问 `http://127.0.0.1:8081/` ，正常情况下你将看到如下返回：


```
Hacash console
Latest height 575258 time 2024-08-13 13:31:22
Block span times: day: 288s, week: 320s, month: 304s, quarter: 303s, year: 297s, all: 302s
[TxPool] tx count: normal(0), diamond mint(1)
Miner worker notice connected: 0
```

此时表示 API 服务已经正常运行。

### 简单示例、快速入门

在文档正式开始之前，我们来测试一个简单的示例。在全节点保持运行的情况下，访问  [http://127.0.0.1:8081/query/balance?address=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS](http://127.0.0.1:8081/query/balance?address=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS), 这是一个查询账户地址余额的接口，正常情况下将会返回例如：

```json
{
    "list": [
        {
            "diamond": 1016,
            "hacash": "1474845:244",
            "satoshi": 0
        }
    ],
    "ret": 0
}
```

可以看到，Hacash 的 API 服务采用标准的 http 接口方式，你可以简单的在浏览器中或你开发的程序中很方便、简单的使用它。


下面的文档将使用 hacash.org 提供的测试接口作为示例，访问 [http://nodeapi.hacash.org/query/balance?address=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS](http://nodeapi.hacash.org/query/balance?address=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS) 应该返回和上面相同的接口数据内容。

示例二：[http://nodeapi.hacash.org/query/balance?address=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS,1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9&unit=mei](http://nodeapi.hacash.org/query/balance?address=1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS,1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9&unit=mei) 表示同时查询两个地址的余额，且返回的余额单位为“枚”。

### 接口格式、通用参数

采用标准的 HTTP 接口请求和应答方式，包含 4 个路径：

   1. /create 生成数据 ， GET 方式 ， 如创建账户、生成转账交易等
   2. /submit 提交数据 ， POST 方式 ， 如提交交易进待确认池等
   3. /query  查询数据 ， GET 方式 ， 如查询账户余额等
   4. /operate 修改数据 ， GET/POST 方式 ， 如修改系统运行参数等
   
多个接口通用的参数介绍如下：

| 参数名 | 类型 | 默认值 | 示例值 | 功能简介 |
| ---- | ---- | ---- | ---- | ---- |
| unit | string  | - | mei,zhu,shuo,ai,miao,248,244,240,... | 返回的 HAC 使用单位 枚、铢、烁、埃、渺， 例如使用“枚”为单位："12.5086"，而标准化单位："125086:244" |
| coinkind | menu |-| h, s, d, hs, hd, hsd, all | 筛选返回的账户、交易信息类型。h: hacash, s: satoshi, d: diamond。用途：比如在扫描区块时只需要返回 HAC 转账内容而忽略其他两者，则传递 `coinkind=h` 即可。 |
| hexbody | bool | false | true, false |  `/submit` 在提交数据时，是否使用 hex 字符串的形式为 Http Body。默认为原生 bytes 形式。 |
| base64body | bool | false | true, false | 在提交数据时，是否使用 base64 编码. |

所有以上字段都是可选的。

### 返回值、公共字段

采用标准 JSON 格式应答所有请求。公共字段如下：

```js

{
  "ret": 0, // 表示返回类型， 0 为正确，  >= 1 则表示发生错误或者查询不存在
  "list": [...], // 某些返回列表数据的接口将使用
  "err": "...." // 如果存在错误，则返回此字段
}

```

下面将介绍每一个具体接口的参数传递和返回值细节。

---

## 1. /create create

#### 1.1 创建账户 `GET: /create/accounts`

随机批量批量创建账户，返回包含私钥、公钥和地址的账户信息列表。可传递参数：

- quantity [int] 表示批量创建账户的数量，默认为 1， 最大值 200

示例接口： [http://nodeapi.hacash.org/create/account?quantity=5](http://nodeapi.hacash.org/create/account?quantity=5)

return value:

```js
{
    list: [
        {
            address: "1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS", // 账户地址
            prikey: "2e50243243abc2e41f3b2ae90029640e235d884a88cfb5ea3e4d0e9efbae6710", // 私钥
            pubkey: "03e22fc27a0d7ae325fa024875febd58266b8b6adbfb966116c9ba958ff5bad7e6" // 公钥
        },
        ...
    ],
    ret: 0
}
```

特别注意：系统采用随机生成算法创建账户，系统并不会保留或记录本接口创建的账户私钥，请务必备份储存好创建的私钥，并做好安全防护。

#### 1.2 创建转账交易 `GET: /create/coin/transfer`

本接口用于创建HAC、单向转移的比特币和区块钻石的转账交易，基本参数如下：

- main_prikey [hex string] 主地址/手续费地址的私钥hex字符串
- from_prikey [hex string] from 地址的私钥hex字符串（不传递时，与 main_prikey 相同）
- fee [string] 交易将要给出的手续费值，例如 "0.0001" 或 "1:244"
- timestamp [int] 交易的时间戳；可选传递；不传时则自动设为当前时间戳
- to_address [string] 收取代币的地址

【注意一】：当只有传递相同的 `timestamp` 时间戳参数，而且保持其它参数始终相同时，则每次创建的交易则具备相同的 hash 值，被视为同一笔交易。

【注意二】：由于 Hacash 系统支持对同一笔交易进行重复签名手续费竞价，所以仅改变 `fee` 字段并不会更改交易的 `hash` 值， 而只会改变其 `hash_with_fee` 值。

【注意三】：手续费 `fee` 字段不能设置得过小，否则将无法被整个系统接受，目前费用最小值为 0.0001 枚（即 ㄜ1:244 ）。请不要设置手续费低于这个值。当区块链交易过多时，请设置更高的手续费，以免交易长时间不被打包，甚至被交易池丢弃。当交易池丢弃手续费过低的交易后，请不要生成第二笔交易，而是修改第一笔交易的手续费字段后重新提交。不然可能出现两笔转账都成功确认的情况，造成资金重复支出两次。

##### 1.2.1 创建HAC普通转账交易

传递参数 `hacash=1:248`，参数如下：

 - hacash [string] HAC 的数额 "0.1" 枚或者 "1:247".

调用接口示例：  [http://nodeapi.hacash.org/create/coin/transfer?main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0001&timestamp=1603284999&hacash=12.45&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS](http://nodeapi.hacash.org/create/coin/transfer?main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0001&timestamp=1603284999&hacash=12.45&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS)

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
    body: "02005f90300700e63c33a796b3032ce6b856f68fccf06608d9ed18f401010001000100e9fdd992667de1734f0ef758bafcd517179e6f1bf60204dd00010231745adae24044ff09c3541537160abb8d5d720275bbaeed0b3d035b1e8b263cb73b724218f13c09c16e7065212128cf0c037ebb9e588754eb073886486d950607d59bef462d2731e15b667c6ff1f0badd6259c6f58d5ca7a5f75856b8cae8e80000 ",
    // 交易使用的时间戳
    timestamp: 1603284999
}
```

在生产环境中，请在数据库中保存以上的返回值，以便进行对账，或者在区块链网络延迟时重新提交未被确认的交易。上面的内容并不会泄露你的私钥，而仅仅是签名后的交易数据，请放心储存。

创建BTC转账和区块钻石的转账，调用接口的的返回值与上者相同。

##### 1.2.2 创建单向转移的 Bitcoin 普通转账交易

传递参数 `satoshi=50000000`, 说明如下

 - satoshi [int] 要支付的比特币数额，单位为“聪”、“satoshi” (0.00000001枚比特币)；例如转账10枚比特币则传递 "1000000000"，转账0.01枚则传递"1000000"；系统不支持低于 1 聪的比特币单位。

例如给某个地址转账 0.5 枚比特币，示例接口如：

[http://nodeapi.hacash.org/create/coin/transfer?main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0001&timestamp=1603284999&satoshi=50000000&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS](http://nodeapi.hacash.org/create/coin/transfer?main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0001&timestamp=1603284999&satoshi=50000000&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS)

##### 1.2.3 创建区块钻石转账交易

传递参数 `diamonds=?`, 详细如下:

- diamonds [string] 逗号分割的钻石字面值，例如 "EVUNXZ,BVVTSI"，可传一个或多个，最多一次批量转移200枚钻石

示例接口调用： [http://nodeapi.hacash.org/create/coin/transfer?main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0003&timestamp=1603284999&diamonds=EVUNXZ,BVVTSI&from_prikey=EF797C8118F02DFB649607DD5D3F8C7623048C9C063D532CC95C5ED7A898A64F&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS](http://nodeapi.hacash.org/create/coin/transfer?main_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=0.0003&timestamp=1603284999&diamonds=EVUNXZ,BVVTSI&from_prikey=EF797C8118F02DFB649607DD5D3F8C7623048C9C063D532CC95C5ED7A898A64F&to_address=1NLEYVmmUkhAH18WfCUDc5CHnbr7Bv5TaS)


---

## 2. /submit 提交

#### 2.1 提交交易至交易池 `POST: /submit/transaction`

提交一笔交易至全网的内存池内。

url 参数如下：

- hexbody [bool] 以 hex 字符串的形式传递 post body 值；否则以原生 bytes 形式传递
 
调用后返回值如下

```js
{
    ret: 0 // ret = 0 表示交易提交成功
}
```

或者返回错误

```js
{
    ret: 1 // ret = 1 表示提交交易错误
    // 例如余额不足，错误消息如下：
    err: "address 1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9 balance ㄜ0:0 not enough, need ㄜ1,245:246."
}
```
curl 命令行测试调用示例：

```shell
curl "http://nodeapi.hacash.org/submit/transaction?hexbody=true" -X POST -d "02005f90300700e63c33a796b3032ce6b856f68fccf06608d9ed18f401010001000100e9fdd992667de1734f0ef758bafcd517179e6f1bf60204dd00010231745adae24044ff09c3541537160abb8d5d720275bbaeed0b3d035b1e8b263cb73b724218f13c09c16e7065212128cf0c037ebb9e588754eb073886486d950607d59bef462d2731e15b667c6ff1f0badd6259c6f58d5ca7a5f75856b8cae8e80000"
```

手续费竞价说明一：交易提交后，当需要付出更高的手续费以作为钻石竞价，或者提高手续费以加快交易确认时，可调用 `/create` 接口再次创建相同的转账交易，仅仅修改数额更高的 `fee` 字段（其它全部字段包括时间戳都必须保持一致，否则就是不同的两笔交易），然后重新提交这笔交易，全网就会丢弃手续费出价低的，而打包这一笔手续费出价更高的交易。所有不同的手续费出价都视为一笔交易的不同状态，无论出价多少次，最终只会有一次打包。

手续费竞价说明二：当查询到交易仍然在交易池内时，可以调用 `/operate` 接口更方便的修改手续费竞价，而不用重新提交整个交易内容。接口说明见文档下方内容。

#### 2.2 提交新挖出的区块到主网 `POST: /submit/block`

向全网区块链提交区块。

url 参数如下：

- hexbody [bool] 以十六进制字符串的形式传递区块数据，否则，以原始字节的形式传递它

该接口主要供第三方矿池开发者使用。

调用后的返回值如下

```js
{
    ret: 0 // ret = 0 表示区块提交成功
}
```

---

## 3. /query 查询

#### 3.0 参数简介

| 查询类型 | 参数 | 必传参 | 说明 |
| ---- | ---- | ---- | ---- |
| balance | address, unit, kind,  |address| 逗号隔开，最多200个 , kind : hsd, unit : boolean |
| diamond | name, number | name or number | 查询 HACD 详情 |
| latest |  | | 最新区块高度和最新 HACD 钻石编号 |
| supply |  | | 代币的统计信息 |
| hashrate |  | | PoW hashrates 信息 |
| hashrate/logs |  | | PoW mining hashrates 历史统计 |
| block/intro | height, hash | height or hash | 区块信息简介 |
| coin/transfer | height, txhash, txposi, kind, unit| height or txhash | 扫描代币转账信息，通过区块高度或交易哈希 (default=0) |
| transaction | hash, unit| hash| 返回一笔交易的详情 |


#### 3.1 查询账户余额 | `GET: /query/balance`

一次查询一个或多个账户的HAC、BTC、钻石余额。传递参数：

- address [string] : 逗号隔开的地址列表，最多200个
- diamonds [bool] : 是否返回此地址拥有的 HACD 名称列表
- coinkind [menu] :查询返回的种类；`coinkind=h`只返回HAC余额， `coinkind=hs`返回HAC、BTC余额，不传或传递`hsd`则返回全部类型余额

 示例接口： [http://nodeapi.hacash.org/query/balance?unit=mei&address=18Yt6UbnDKaXaBaMPnBdEHomRYVKwcGgyH,1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF](http://nodeapi.hacash.org/query/balance?unit=mei&address=18Yt6UbnDKaXaBaMPnBdEHomRYVKwcGgyH,1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF)
 
或者返回全部钻石列表 `diamonds=true`:

 [http://nodeapi.hacash.org/query/balance?unit=mei&address=18Yt6UbnDKaXaBaMPnBdEHomRYVKwcGgyH,1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF&diamonds=true](http://nodeapi.hacash.org/query/balance?unit=mei&address=18Yt6UbnDKaXaBaMPnBdEHomRYVKwcGgyH,1DYY4ZRsWnhjcwwnE3dWgtiqe2mctDS2HF&diamonds=true)
 

 示例返回值：

 ```js
{
    ret: 0, // general return value
    list: [
        {
            diamond: 350, // the number of diamonds
            hacash: "3101.0826", // HAC balance
            satoshi: 0, // BTC balance, unit "satoshi": satoshi
            diamonds: "ITHWMSVBYYYTWKIKNZKNTUYWAAABBBWWWTTT"
        },
        {
            diamond: 0,
            hacash: "3842.24693214",
            satoshi: 0
        }
    ]
}
```

注意一：返回的余额列表与传递的账户列表位置相对应。

注意二：如果需要返回 `diamonds` 字段，请开启全节点配置文件下 `[server]` 组中的 `diamond_form = true`， 全节点才会开启账户 HACD 列表的记录。

#### 3.2 查询区块钻石 HACD 信息 `GET: /query/diamond`

查询一枚钻石的哈希、产生的区块高度，当前所属地址等详细信息；可以通过钻石字面值（name）或者标号（number）来查询。

传递参数：

- name [string] 钻石字面值，例如： "ZAKXMI"
- number [int] 钻石编号，例如： "20001"

示例查询接口一： [http://nodeapi.hacash.org/query/diamond?number=20001](http://nodeapi.hacash.org/query/diamond?number=20001)

示例查询接口二：[http://nodeapi.hacash.org/query/diamond?name=ZAKXMI](http://nodeapi.hacash.org/query/diamond?name=ZAKXMI)

以上两个接口返回内容相同：

```js
{
    ret: 0, // 公共返回值
    number: 20001, // 钻石编号
    name: "ZAKXMI", // 钻石名称/字面值
    bid_fee: "382:246", // 本颗钻石花费的竞价手续费
    mint_height: 179610, /被挖出的区块高度
    belong: "1KcXiRhMgGcvgxZGLBkLvogKLNKNXfKjEr", // 当前拥有地址
    visual_gene: "e5e5b79409af7423bca1", // 可视化基因
    life_gene:"aa41cc92bbff792266fca82217ddd0aabb2477f519badf071472037b7cea21e5", // 生命基因
    inscripts:["...."], // 铭文列表
}

```

可以看到，以上两个接口查询的实质上是同一枚钻石。

#### 3.3 查询主网最新区块和最新钻石编号 `GET: /query/latest`

查询当前区块链的最新有效区块信息。

请求示例: [http://nodeapi.hacash.org/query/latest](http://nodeapi.hacash.org/query/latest)

返回值： 

```js
{
    ret: 0,
    height: 575611, // 当前已经写入区块链的区块高度
    diamond: 96254, // 最新被成功铸造的钻石编号
}
```

#### 3.4 查询指定区块信息 `GET: /query/block/intro`

参数:

- height [int] 可选，通过指定高度查询区块信息
- hash [string] 可选，通过区块 hash 值查询区块信息
 
示例接口一:  [http://nodeapi.hacash.org/query/block/intro?height=177255&unit=mei](http://nodeapi.hacash.org/query/block/intro?height=177255&unit=mei)

示例接口二:  [http://nodeapi.hacash.org/query/block/intro?hash=000000001e76b27f70f9a5e5b0f9b9824b416c210548ffad298577ed5378cdcd](http://nodeapi.hacash.org/query/block/intro?hash=000000001e76b27f70f9a5e5b0f9b9824b416c210548ffad298577ed5378cdcd)
 
示例返回值：

```js
{
    ret: 0,
    height: 177255, // 区块高度
    hash: "000000001e76b27f70f9a5e5b0f9b9824b416c210548ffad298577ed5378cdcd", // 区块哈希
    mrklroot: "d18df37a52c377115f053aca7822c65e5d154dac4ab6e062326dc770b5d1e5ad", // 默克尔根
    prevhash: "000000001868742351b8ab196444d22c0db2951c522ebc5b9ef057ad70a6556c", // 上一个区块hash
    timestamp: 1601931702, // 区块时间戳
    transaction: 5, // 包含交易数量
}
```

#### 3.5 扫描每一笔交易获取简要转账记录 `GET: /query/coin/transfer`

传递参数:

- height [int] 要扫描的区块高度
- txposi [int] 要扫描的交易位于区块中的数组索引位置，从0开始索引
- unit [string] 可选, HAC 返回的单位形式
- coinkind [menu] 查询返回的种类；`coinkind=h`只返回HAC转账， `coinkind=hs`返回HAC、BTC转账，不传或传递`hsd`则返回全部类型转账
- from [string] 要筛选的 from 地址，传递后仅仅会展示传入的地址，而忽略其他 from 地址
- to [string] 要筛选的 to 地址，传递后仅仅会展示传入的地址，而忽略其他 to 地址

示例调用接口一：[http://nodeapi.hacash.org/query/coin/transfer?unit=mei&height=180115&txposi=1&coinkind=hsd](http://nodeapi.hacash.org/query/coin/transfer?unit=mei&height=180115&txposi=1&coinkind=hsd)

返回值示例：

```js
{
    ret: 0,
    tx_hash: "43e98177bc426a2f15c15fe3e8968ece1b2e0829eab7cacb8715c89082f48aef",
    tx_timestamp: 1698278005,
    block_hash: "0000000006010e2588ccd1a2b3c391348e204e119a9be6c05f05e57fba8c7ddd",
    block_timestamp: 1602764680,
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
            diamond: 1,
            diamonds: "WUZXYM"
        },
        {
            from: "13RnDii79ypWayV8XkrBFFci29cHtzmq3Z",
            to: "1EcrtFAUmVeLnGeaEcMDoPZH7ZPysks1H2",
            diamond: 3,
            diamonds: "WUZXYMIZHTEWIIUMWH" // 多枚钻石
        }
    ],
    type: 2
}

```


#### 3.6 查询 HACD 铭文或者清除铭文的记录 `GET: /query/diamond/engrave`

传递参数:

- height [int] 要扫描的区块高度
- txposi [int] 要扫描的交易位于区块中的数组索引位置，从0开始索引
- tx_hash [bool] 可选，通过铭刻交易的hash查询

如果不传递 `txposi` 字段, 则扫描整个区块的铭文，并一次性返回。


示例 API 1：[http://nodeapi.hacash.org/query/diamond/engrave?height=519242&txposi=0](http://nodeapi.hacash.org/query/diamond/engrave?height=519242&txposi=0)

示例 API 2：[http://nodeapi.hacash.org/query/diamond/engrave?height=531045](http://nodeapi.hacash.org/query/diamond/engrave?height=531045)

示例 API 3：[http://nodeapi.hacash.org/query/diamond/engrave?height=540771&tx_hash=true](http://nodeapi.hacash.org/query/diamond/engrave?height=540771&tx_hash=true)

返回值示例：

```js
{
    ret: 0,
    list: [
    {   /* 对单个钻石进行铭刻 */
        inscription: "HIP-15 1st HACD",
        diamonds:"BMABAT"
    },
    {   /* 批量铭刻 */
        inscription: "Hacds",
        diamonds:"SHHHKSUKIUEIZKBYEVHKYWBYAUTZHY",
        tx_hash: '9beeb7c528d5e9f9edfbd08f16a5d117164d466ae4c8edc038609d9117119cc2' /* 如果传递 `txposi` 参数则返回交易hash值  */
    },
    {   /* 清除全部铭文 */
        clear: true, // 表示是否清除
        diamonds:"WETUAX" // 或则一次性清除多个 "SHHHKSUKIUEIZKBYEVHKYWBYAUTZHY"
    }
    ]
}
```



#### 3.7 查询交易详情 `GET: /query/transaction`

传递参数:

- hash [int] 交易 哈希
- action [bool] 可选，是否返回 actions 详情
- unit [string] 可选，HAC 返回单位 "mei" or "248"
- body [bool] 可选，是返回交易的 body 值
- signature [bool] 可选，返回交易签名详情
- description [bool] 可选，返回交易简介

示例 API 1：[http://nodeapi.hacash.org/query/transaction?hash=b563b525d5a840f14c2f1d7da52a1eb3f7ee7357e784af8b68d0b0f74b95cbb4&action=true&unit=mei](http://nodeapi.hacash.org/query/transaction?hash=b563b525d5a840f14c2f1d7da52a1eb3f7ee7357e784af8b68d0b0f74b95cbb4&action=true&unit=mei)

示例返回值：

```js
{
    ret: 0,
    block: {
        height: 583881,
        timestamp: 1726169459
    },
    confirm: 4,
    hash_with_fee: "ee5c254b1f1628eacff17ea39836a17569d5d5bbd8d9e97b8a0383dac95740ac",
    hash: "b563b525d5a840f14c2f1d7da52a1eb3f7ee7357e784af8b68d0b0f74b95cbb4"
    fee: "0.0001",
    fee_got: "0.0001",
    type: 2,
    timestamp: 1726169260,
    main_address: "12pbJdDmvZcRs3BhJnJ1RKuiHE48mVJS71",
    action: 1,
    actions:[{
        kind: 1,
        hacash: "15.9975414",
        from: "12pbJdDmvZcRs3BhJnJ1RKuiHE48mVJS71",
        to: "1NzdWYfp42gpRDf9Hsv5CA6D5gQvMt3599"
    }],
}

```

注意: 字段  `actions` 内的 ' hacash / satoshi / diamond / diamonds / from / to ' 与 `coin/transfer` 接口返回的字段所表示的意义相同，接口 `GET: /query/coin/transfer`


#### 3.8 查询代币总供应量 `GET: /query/supply`

示例：[http://nodeapi.hacash.org/query/supply](http://nodeapi.hacash.org/query/supply)
 
示例返回：
 
```js
{
    ret: 0,
    minted_diamond: 27538, // 最新的钻石 HACD 编号
    channel_interest: 151.6902713463587, // 通道利息
    burning_fee: 890984.44654182, // 燃烧的交易手续费（包括钻石竞价）
    current_circulation: 240364.69027134636, // 当前 HAC 流通量（剔除已燃烧的）
    block_reward: 1804911.0, // 总的区块奖励释放
}
```

---

## 4. /operate 修改、操作

#### 4.1 提升待确认交易手续费 `GET: /operate/fee/raise`

当一笔交易存在于交易池中还没有被确认时，可以重新签名提升（但不能降低）此笔交易的手续费，以达到加快交易确认或者钻石竞价报价的作用。

传递参数：

- tx_hash [string]  要提升手续费的交易的 hash 值
- fee [string] 要修改的目标手续费
- fee_prikey [string] 手续费地址的私钥
- unit [string] HAC 单位
- tx_body [hex] 可选，当交易池内不存在这笔交易时，通过使用提交的tx_body来修改手续费，并再次将交易广播给全网

示例接口: 

[http://nodeapi.hacash.org/operate/fee/raise?fee_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=32:247&tx_hash=ad26a35116664176426f3c08adad147577b9a85999cb89d465becf6a27002c04](http://nodeapi.hacash.org/operate?action=raise_tx_fee&fee_prikey=8D969EEF6ECAD3C29A3A629280E686CF0C3F5D5A86AFF3CA12020C923ADC6C92&fee=ㄜ32:247&txhash=ad26a35116664176426f3c08adad147577b9a85999cb89d465becf6a27002c04)

示例返回值:

```js
{
    ret: 0,
    hash: "",
    hash_with_fee: "",
    fee: "2:244",
    tx_body: "...",
}
```

或者发生错误时:

```js
{
    ret: 1,
    err: "Tx fee address password error: need 18Yt6UbnDKaXaBaMPnBdEHomRYVKwcGgyH but got 1MzNY1oA3kfgYi75zquj3SRUPYztzXHzK9" // Reason for failure: Inconsistent fee address
}

```   


---


## 与挖矿或矿池相关的接口

新版 Rust 版本的 Full Node 并没有在内部集成挖矿功能，而是通过 HTTP 接口暴露给外部程序。随全节点一起发布的 PoWorker 挖矿程序，直接连接全节点，通过以下接口自动挖矿。

#### 5.1 读取有关正在开采的区块信息 `GET: /query/miner/pending`

接口示例: 

[http://nodeapi.hacash.org/query/miner/pending](http://nodeapi.hacash.org/query/miner/pending)

传递参数:

- detail [bool] 可选, 是否返回详细的区块字段
- transaction [bool] 可选, 是否返回全部交易 body 信息
- base64 [bool] 可须那, 是否按 base64 编码格式返回数据（默认hex格式）

返回示例:

```js
{
    ret: 0,
    height: 575618,
    target_hash: "00000000001172658fffffffffffffffffffffffffff",
    coinbase_nonce: "0000000000000000000000000000000000000000000000000000000000000001",
    block_intro: "01000008c8820066bc9bce000000000008e3c7eda5a7abd046e09ac2b5902aca50d4cd88ae8963e13925522566fb0e6c3e95ed652ee5e23a7d0b10db6e88ac5c27c1ba2b1cbe429a68cb540000000700000000d48b932c0000"
}
```

或者但传递 `detail=true` 参数时的返回:

```js
{
    version: 1,
    prevhash: "...",
    timestamp: 23485762,
    transaction_count: 6,
    reward_address: "1AVRuFXNFi3rdMrPH4hdqSgFrEBnWisWaS",
}
```

当传递 `transaction=true` 参数时的返回:

```js
{
    transaction_body_list: [
        "....",
        "...."
        // ...
    ]
}
```

上述返回值，当同一个区块时，将为每个请求返回不同的 `coinbase_nonce`，从而自动获得一个称为 `block_nonce` ，大小为 `uint32` 的哈希尝试空间。

`target_hash` 是当前计算难度下的目标难度哈希，即计算出的区块哈希的前导零数不能小于此数。`coinbase_nonce` 需要挖矿方记录下来，以便在区块挖出成功时通过接口返回。`block_intro` 是区块头数据，是挖矿的原材料。

挖矿是通过不断改变区块的 `nonce` 值来完成的，以尝试计算满足 `target_hash` 难度目标的区块哈希值。示例代码：

```rust
    let mut block_intro = hex::decode("01000008c8820066bc9bce000000000008e3c7eda5a7abd046e09ac2b5902aca50d4cd88ae8963e13925522566fb0e6c3e95ed652ee5e23a7d0b10db6e88ac5c27c1ba2b1cbe429a68cb540000000700000000d48b932c0000").unwrap();
    for nonce in 0..u32::MAX {
        block_intro[79..83].copy_from_slice(&nonce.to_be_bytes());
        let reshx = x16rs::block_hash(hei, &intro).to_vec();
        // check reshx
    }
```

可以在此处找到挖掘算法的实现：

[https://github.com/hacash/rust/blob/95eb6670a3f26b69e4b4a7f006942553fec003af/src/run/poworker.rs#L488](https://github.com/hacash/rust/blob/95eb6670a3f26b69e4b4a7f006942553fec003af/src/run/poworker.rs#L488)



#### 5.2 提交一个新挖掘出的有效区块 `GET: /submit/miner/success`

参数传递:

- height [int] 
- block_nonce [uint32] 成功找到的 nonce 值
- coinbase_nonce [string] 通过 `/query/miner/pending` 获取到的值

接口示例: 

[http://nodeapi.hacash.org/submit/miner/success?height=575618&block_nonce=2236475323&coinbase_nonce=0000000000000000000000000000000000000000000000000000000000000001](http://nodeapi.hacash.org/submit/miner/success?height=575618&block_nonce=2236475323&coinbase_nonce=0000000000000000000000000000000000000000000000000000000000000001)

返回示例:

```js
{
    ret: 0,
    height: 575618,
    mining: "success",
}
```

注意，你必须先访问 `/query/miner/pending` 接口来创建并获取待挖出的区块，稍后再通过该接口提交区块才能生效。否则，你将受到“区块未创建或未找到”的错误。


#### 5.3 订阅最新的区块高度变化 `GET: /query/miner/notice`

次接口通过 HTTP 长链接来实现：

参数传递:

- height [int] 要监控的区块高度
- rqid [string] 为每个请求随机生成一个唯一的任意字符串，以防止 HTTP 缓存等问题
- wait [int] 长 HTTP 链接等待的秒数

示例: 

[http://127.0.0.1:8081/query/miner/notice?height=367454&rqid=50d4cd88ae8963e13925522566fb0e6c3e95ed652ee5e23a](http://127.0.0.1:8081/query/miner/notice?height=367454&rqid=50d4cd88ae8963e13925522566fb0e6c3e95ed652ee5e23a)

返回值:

```js
{
    ret: 0,
    height: 575622,
}
```

该 API 是一个通过 HTTP 长链接轮询全网区块高度变化的接口。当传入的 `height` 被挖到时，这个接口会立即返回，当要求的区块高度没有被挖到时，这个接口会保持连接并等待，每次等待的时间由 `wait` 参数指定，最长可达 300 秒。

使用 HTTP 轮询代替 TCP 或 WebSocket 主要是为了挖矿或矿池开发的简单性，以及多种设备或网络环境的丰富性，可以支持更多技术系统接入挖矿。

这是一个 500 行多线程挖矿客户端的 Rust 实现，它实时显示哈希率统计数据，如果你是 Hacash 挖矿或矿池的开发者，这是一个很好的例子：

[https://github.com/hacash/rust/blob/main/src/run/poworker.rs](https://github.com/hacash/rust/blob/main/src/run/poworker.rs)



