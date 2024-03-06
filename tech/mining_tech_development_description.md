Hacash 挖矿技术开发说明
===

> 本文档为第三方挖矿工具、平台或服务的开发者准备，是专业矿机（系统集成或软硬件一体化、专业硬件等）、矿池、算力数据平台服务等欲接入Hacash挖矿相关事项的技术开发人员的开发指引和挖矿算法说明。
> 如果需要了解Hacash挖矿产出等基础说明，请查阅 [HAC 和 HACD 挖矿公平性说明↗](https://github.com/hacash/doc-chinese/blob/main/tech/HAC_HACD_mining_fairness_description.md)

Hacash 的区块挖矿、钻石（HACD）挖矿算法都采用独创的 [X16RS↗](https://github.com/hacash/x16rs/) 作为基础，此算法在Hacash主网启动时随全节点代码一并 [开源↗](https://github.com/hacash/x16rs/)。

Hacash的挖矿分为区块和钻石两个独立部分，其中又分别分为多个环节，本文将详细描述挖矿的算法细节和相关注意事项。

## X16RS

X16RS 的基础算法部分与 渡鸦币 (Ravencoin) 所采用的 X16R 算法相同，也是 16 种 SHA3 算法的组合。 X16RS (Random Scalable) 是 X16R (Random) 算法的升级版，X16RS 的改进在于，将原有的每区块确定一次哈希组合，变成每一步哈希都重新随机选择算法，从而大幅增加 FPGA 或 ASIC 设计的难度。采用的 16 种哈希算法是：

```C
enum AlgoX16 {
    BLAKE = 0,
    BMW,
    GROESTL,
    JH,
    KECCAK,
    SKEIN,
    LUFFA,
    CUBEHASH,
    SHAVITE,
    SIMD,
    ECHO,
    HAMSI,
    FUGUE,
    SHABAL,
    WHIRLPOOL,
    SHA512
};
```

- [相关算法链接↗](https://github.com/hacash/x16rs/blob/7b2afef3bafb0a6d7f7301b04230dfa00ed52508/x16rs.c#L56)

X16RS 随机性的关键部分，是取上一轮哈希结果（32位）的后4位，作为一个 uint32 值，取 16 的余数，得到一个范围为 0~15 之间的整数，作为 16 种 SHA3 算法列表的选择下标，取出对应的算法执行下一次哈希计算。这意味着每次下一步选择的哈希算法必须以上一次哈希的计算结果为参数，无法提前预知计算链。如果算法重复执行 16 次，则存在 16^16 大约  1.84467440737096e+19 种计算路径，这是导致 X16RS 算法巨大的随机性的核心：

```C
void hash_x16rs_choice_step(hash_t* stephash){

    uint8_t algo = stephash->h4[7] % 16;

    switch (algo) {
        case BLAKE:     hash_x16rs_func_0 ( stephash ); break;
        case BMW:       hash_x16rs_func_1 ( stephash ); break;
        case GROESTL:   hash_x16rs_func_2 ( stephash ); break;
        case JH:        hash_x16rs_func_3 ( stephash ); break;
        case KECCAK:    hash_x16rs_func_4 ( stephash ); break;
        case SKEIN:     hash_x16rs_func_5 ( stephash ); break;
        case LUFFA:     hash_x16rs_func_6 ( stephash ); break;
        case CUBEHASH:  hash_x16rs_func_7 ( stephash ); break;
        case SHAVITE:   hash_x16rs_func_8 ( stephash ); break;
        case SIMD:      hash_x16rs_func_9 ( stephash ); break;
        case ECHO:      hash_x16rs_func_10( stephash ); break;
        case HAMSI:     hash_x16rs_func_11( stephash ); break;
        case FUGUE:     hash_x16rs_func_12( stephash ); break;
        case SHABAL:    hash_x16rs_func_13( stephash ); break;
        case WHIRLPOOL: hash_x16rs_func_14( stephash ); break;
        default SHA512: hash_x16rs_func_15( stephash ); break; // case 15
    }
}
```

- [相关算法链接↗](https://github.com/hacash/x16rs/blob/7b2afef3bafb0a6d7f7301b04230dfa00ed52508/x16rs.c#L444)

这种随机性是抵抗或延缓 FPGA 及 ASIC 专业硬件挖矿设备出现的基础，也是 GPU 显卡采矿相比 CPU 设备来说暂时并没有明显效率优势的原因。由于每次哈希执行的算法组合都不相同，而这些算法也都具备不同的时间或空间复杂度，这种巨大的随机性也是 HAC 挖矿单台设备的算力统计结果波动性巨大、无法得到平稳数值的缘由。

HAC 和 HACD 的挖矿都以 X16RS 算法为基础。

## HAC 挖矿

Hacash 区块挖矿的难度规则与比特币相同，都是通过区块哈希值的前导零的数量来体现。

Hacash 的区块哈希值由区块头数据序列化后，经过一次 SHA3_256 的计算，在经过多次 X16RS 的计算得出：[计算区块Hash相关代码↗](https://github.com/hacash/x16rs/blob/7b2afef3bafb0a6d7f7301b04230dfa00ed52508/x16rs.go#L76)

也就是说，HAC 的挖矿分为两步：

1. 计算区块头的 SHA3_256 哈希值； 
2. 将上一步哈希结果再进行若干次 X16RS 哈希计算；

并且，X16RS 的计算次数是不固定的，随着区块高度的增长而增加，从 1 次开始，每 50000 个区块约半年时间增加一次，一直到 16 次哈希后保持不变：[区块哈希次数计算↗](https://github.com/hacash/x16rs/blob/7b2afef3bafb0a6d7f7301b04230dfa00ed52508/x16rs.go#L68), 前 8 年时间挖矿的次数都处于变化之中，此种设计也是为了在早期抵抗 ASIC 等专业采矿设备的出现。

Hacash 的区块头包含以下字段：

```golang
type HacashBlock struct {
	// head
	Version          VarUint1
	Height           BlockHeight
	Timestamp        BlockTxTimestamp
	PrevHash         Hash
	MrklRoot         Hash
	TransactionCount VarUint4
	// meta
	Nonce            VarUint4 // Mining random value
	Difficulty       VarUint4 // Target difficulty value
	WitnessStage     VarUint2 // Witness quantity level
}
```

- [相关代码链接↗](https://github.com/hacash/core/blob/425d0f2cff5a747c448ffd1d0b091f37de6af082/blocks/version1.go#L16)

与挖矿相关的字段为 `Nonce` 和 `Difficulty`，其中， Difficulty 为本区块的目标哈希难度的压缩值，Nonce 则为哈希碰撞挖矿的随机因子（挖矿随机数），两者长度都为 4 个字节。

显然，与比特币的情况一样，当 Hacash 挖矿难度上升到一定规模之后，4字节的 Nonce 值作为随机因子将无法满足区块挖矿的难度目标。此时，交易集合的默克尔根（MrklRoot）将发挥作用：通过改变交易的排序、插入包含随机数据的交易或者修改 Coinbase 交易（固定为区块交易列表的第一币交易）的 MinerNonce 值（32位），可以改变 MrklRoot 值，从而重新获得 Nonce 随机数空间。每改变一次 MrklRoot 可以得到 256^4 次（一个uint32值范围）的碰撞尝试机会。

这将导致算力较强的采矿设备，在尝试完一个 Nonce 空间之后，需要与全节点或者矿池进行通信，以获得新的区块头数据（指新的 MrklRoot），才能继续挖矿。当然，也可以通过将区块头、区块内 Coinbase 交易的 tx_body 和其他交易的 tx_hash 都加载至采设备，由矿机自己替换掉 Coinbase 交易内的 MinerNonce 数值后，计算出新的 MrklRoot ，从而可以获得无限次的 Nonce 采矿空间，避免重复繁琐的通信。不过，矿机仍然需要与矿池或全节点保持连接，因为需要监听全网新区块挖掘成功的消息，以便及时切换至下一个区块的采矿，这意味着矿池与矿机、矿池与全网之间的网络延迟越低越好，否则将影响挖矿效率，如果出现掉线问题，则设备的挖矿可能变得无意义。

MinerNonce 可以是任意的随机数，用于扩展挖矿区块头的 Nonce 空间，或者附带其它挖矿信息。Coinbase 交易的结构及 MinerNonce 位置：

```golang
type TransactionCoinbase struct {
	// type
	Address Address
	Reward  Amount
	Message TrimString16
	// Version number
	ExtendDataVersion fields.VarUint1 // Equal to 0 before 220000 height
	// When extenddataversion >= 1, it has the following fields:
	MinerNonce   fields.Bytes32
    /* other fields */
}
```

- [相关代码链接↗](https://github.com/hacash/core/blob/425d0f2cff5a747c448ffd1d0b091f37de6af082/transactions/coinbase.go#L16)

#### 区块难度调整

Hacash 的区块挖矿难度调整为每 288 个区块（也就是一天）调整一次，这有助于更快速响应算力规模的变化。每次难度调节变化的幅度不高于当前难度的 4 倍，或不低于当前难度的 1/4 。

[区块难度调节代码链接↗](https://github.com/hacash/mint/blob/909621f84f4873445064d89be24503082c026266/difficulty/algo_v2.go#L97)


## HACD 钻石挖矿

HACD 的铸造分为两个部分： 1. 挖矿； 2. 竞价；

本文档仅涉及挖矿部分，有关 HACD 竞价的内容和说明，请查阅另一个文档：[HAC 和 HACD 挖矿公平性说明↗](https://github.com/hacash/doc-chinese/blob/main/tech/HAC_HACD_mining_fairness_description.md)

HACD 的挖矿机制已经在 Hacash 白皮书内有简要描述，其分为 采掘 和 难度检查 两个步骤。

#### HACD 初步采掘


HACD 的采掘算法，是从一个包含当前钻石编号、上一个包含了钻石的区块地址、矿工地址、随机数等组合成的二进制数据通过 SHA3_256 计算得出的 32 位哈希值开始：

```golang
// get 32byte hash stuff
func Diamond(diamondNumber uint32, prevblockhash []byte, nonce []byte, address []byte, extendMessage []byte) ([]byte, []byte, string) {
	repeat := HashRepeatForDiamondNumber(diamondNumber)
	stuff := new(bytes.Buffer)
	stuff.Write(prevblockhash)
	stuff.Write(nonce)
	stuff.Write(address)
	stuff.Write(extendMessage)

	/* get ssshash by sha3 algrotithm */
	ssshash := sha3.Sum256(stuff.Bytes())
	/* get diamond hash value by HashX16RS algorithm */
	reshash := HashX16RS(repeat, ssshash[:])
	/* get diamond name by DiamondHash function */
	diamond_str := DiamondHash(reshash)

	return ssshash[:], reshash, diamond_str
}
```

- [相关算法链接↗](https://github.com/hacash/x16rs/blob/7b2afef3bafb0a6d7f7301b04230dfa00ed52508/x16rs.go#L149)

HACD 的字面值为 6 位字母，从 `WTYUIAHXVMEKBSZN` 这 16 个字母中产生，且不重复，所以其固定总量为 16 ^ 6 = 16,777,216 枚。Hacash 创造了 `diamond_hash` 哈希算法，将 32 位的哈希值经过计算映射为 16 位的哈希字符串结果，例如 `VMEKUBIAHXSZNWTY`、`IAX0K0VM00EUBZWY` 或 `0000VMEKUBIAXZWY` 等。

当且仅当这 16 位结果字符串的前 10 位为字符 `0`，后 6 位字符不为 `0` 时（例如 0000000000YKUBIA ）时，即视为成功采掘出一枚 HACD。取出后 6 位字符 `YKUBIA` 作为这枚 HACD 的字面值。

```C
// input length must be 32
static const uint8_t diamond_hash_base_stuff[17] = "0WTYUIAHXVMEKBSZN";
void diamond_hash(const char* hash32, char* output16)
{
    uint8_t *stuff32 = (uint8_t*)hash32;
    int i, p = 13;
    for(i = 0; i < 16; i++){
        int num = p * (int)stuff32[i*2] * (int)stuff32[i*2+1];
        p = num % 17;
        output16[i] = diamond_hash_base_stuff[p];
        if(p == 0){
           p = 13; 
        }
    }
}
```

- [相关算法链接↗](https://github.com/hacash/x16rs/blob/7b2afef3bafb0a6d7f7301b04230dfa00ed52508/x16rs.c#L578)

通过上面的两步算法，采掘出一枚 HACD 后，并不意味着就能直接上链确认，还需要完成另外两个步骤：

1. 排重
2. 难度检查

排重的意思是经过初步算法挖出的 HACD 的字面值有可能和已经挖出的 HACD 重复，这时第二次挖出的就只能丢弃掉，永远无法上链。在后期随着挖出的 HACD 越来越多，重复挖出的概率将越来越大，这将慢慢影响有效哈希值的空间，从而作为抬升挖矿难度的一个方面。

难度检查是一个检测哈希值有效性的函数，与 HACD 的编号有关，编号越大，则目标难度越高：

```golang
func CheckDiamondDifficulty(dNumber uint32, sha3hash, dBytes []byte) bool {
	var DiaMooreDiffBits = []byte{ // difficulty requirements
		128, 132, 136, 140, 144, 148, 152, 156, // step +4
		160, 164, 168, 172, 176, 180, 184, 188,
		192, 196, 200, 204, 208, 212, 216, 220,
		224, 228, 232, 236, 240, 244, 248, 252,
	}

	// Referring to Moore's law, the excavation difficulty of every 42000 diamonds will double in about 2 years,
	// and the difficulty increment will tend to decrease to zero in 64 years
	// and increase the difficulty by 32-bit hash every 65536 diamonds
	shnumlp := dNumber / 42000 // 32step max to 64 years
	shmaxit := 255 - uint8(dNumber/65536)
	for i := 0; i < 32; i++ {
		if i < int(shnumlp) && sha3hash[i] >= DiaMooreDiffBits[i] {
			return false // Check failed, difficulty value does not meet requirements
		}
		if sha3hash[i] > shmaxit {
			return false // Check failed, difficulty value does not meet requirements
		}
	}

	// Every 3277 diamonds is about 56 days. Adjust the difficulty 3277 = 16 ^ 6 / 256 / 20
	// When the difficulty is the highest, the first 20 bytes of the hash are 0, not all 32 bytes are 0.
	diffnum := dNumber / 3277
	for _, bt := range dBytes {
		if diffnum < 255 {
			if uint32(bt)+diffnum > 255 {
				return false // Difficulty check failed
			} else {
				return true
			}
		} else if diffnum >= 255 {
			if uint8(bt) != 0 {
				return false // Difficulty check failed
			}
			// to do next round check
			diffnum -= 255
		}
	}

	return false
}
```

- [相关算法链接↗](https://github.com/hacash/x16rs/blob/7b2afef3bafb0a6d7f7301b04230dfa00ed52508/x16rs.go#L254)

HACD 难度检查算法分为以下几个方面：

1. X16RS 算法的哈希次数每挖出 8192 枚大约半年增加一次，8年时间从 1 次哈希增加到 16 次哈希，并持续增加直到2048次哈希。相当于算力难度将在早期 8 年时间增加 16 倍；
2. 每挖出 42000 枚约 2 年，挖矿难度将增加为当前挖矿难度的 2 倍，也就是难度系数 × 2 。但随后每次难度调整的乘数将略微减少低于 2 倍，直到 64 年后最终减少到 1 为止。具体变化大概为：2.00, 1.94, 1.88, 1.83, 1.78, 1.73, 1.68, 1.64, 1.60, 1.56, 1.52, 1.49, 1.45, 1.42, 1.39, 1.36, 1.33, 1.31, 1.28, 1.25, 1.23, 1.21, 1.19, 1.16, 1.14, 1.12, 1.10, 1.08, 1.07, 1.05, 1.03, 1.02，最后系数为 1 ，也就是此维度控制的难度因素不增加了；
3. 每挖出 3277 枚大约 56 天，难度系数将调整一个 byte 的数字（256减一），也就是将最终难度哈希的左侧byte值减1。此种难度调整按指数曲线方式变化，目前还无法计算出具体的倍数，并且前期难度增加的幅度并不大，甚至不太容易察觉的。但随着 HACD 挖出数量的增多，难度增加将越来越大，在挖矿的中间时期难度增长才会慢慢变得显著，将会超越比特币的挖矿难度，在挖矿的后期，增幅越来越大，直到超越地球上所有计算设备的计算能力。最后阶段，挖矿难度将增长到数学上的极限值，达到耗尽太阳的能量也难以挖出一颗钻石的极端程度。
4. 由于钻石的字面值为6位字母组成，总量上限为 16,777,216 颗，哈希计算的随机性特征导致可以重复挖出已经被确认的钻石字面值，则此次计算就算已经挖出合法的钻石字面值，也只能视为无效，消耗的算力作废。这就将压缩有效挖矿的空间，虽然在早期对难度的影响不大，但当挖出一半钻石时，难度将增加到原先的 2 倍，再挖出剩下的一半的一半钻石，难度将增加4倍，再挖出一半，难度增加到 8 倍。每挖出剩余钻石的一半，难度将上升一倍，以此类推，直到挖掘最后一批钻石时，这时几乎全部挖矿都大概率挖到重复的字面值，挖出新的有效的钻石字面值将非常困难，难度将增长到不可能达到的极致。
   
以上 4 个维度的联合作用导致钻石挖矿的难度调整很难用简单的参数曲线展现出来，很长时间内只能依赖与矿工的挖矿测试和统计。需要有数学能力更强者，能将此种复合难度调整机制转换成数学公式，最后制作成图表供大家查看。

每一枚 HACD 挖出后都必须通过此难度检查函数才能提交到主网进行竞价，挖矿难度只与已经挖出的 HACD 数量有关，与当前周期内采矿设备投入规模无关。

