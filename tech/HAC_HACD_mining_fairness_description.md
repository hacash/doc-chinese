HAC 和 HACD 挖矿的公平性说明
===

本文档并不是对 Hacash 挖矿机制全面深入的详解文档，也不是教授挖矿的操作指导手册，而是对 HAC 和 HACD 的挖矿奖励和产出机制中有关公平性设计的部分进行说明。

## **HAC 挖矿**

HAC 区块奖励遵循先6年增加再60年减少最后永远保持不变的先增后减最后不变机制。让任何愿意参与挖矿的人都有十分充足的时间和机会投入公平的成本获取到应该有的货币份额。

Hacash 平均每五分钟挖出一个区块，初始时每区块奖励给矿工 1 枚 HAC，每天 288 个区块被挖出也就是每天 奖励 288 枚 。

区块奖励的增长以斐波拉切黄金分割数列为标准，分为三个阶段：快速增加、缓慢减少、保持不变。

增加阶段为6年：第一年每区块挖出 1 枚HAC，第二年每区块还是 1 枚，第三年每区块增加到 2 枚，第四年增加到 3 枚，第五年增加到 5 枚，第六年增加到 8 枚每区块奖励。前6年的时间，挖矿奖励一直在增加，从1枚增加到8枚。

减少阶段为60年：从第7年开始，挖矿奖励每区块仍然为8枚且保持10年。第17年，奖励下降到5枚，保持十年。第27年，奖励变为3枚，保持10年。第37年下降到2枚，第47年下降到1枚。第57年和67年仍然为1枚每区块奖励。

保持不变阶段：67年开始，区块奖励进入永恒不变纪元，为每区块1枚，直到永远。

前66年区块奖励可挖出2200万枚。每年挖矿释放货币占前66年产出量的百分比如下：

|**年**|**区块奖励**|**当年释放**|**累计释放**|  
|:----|:----|:----|:----|
|1|1|0.4555%|0.4555%|           
|2|1|0.4555%|0.9111%|           
|3|2|0.9111%|1.8222%|           
|4|3|1.3667%|3.1889%|           
|5|5|2.2778%|5.4667%|           
|6|8|3.6445%|9.1112%|           
|7|8|3.6445%|12.7557%|              
|8|8|3.6445%|16.4004%|           
|9|8|3.6445%|20.0448%|           
|10|8|3.6445%|23.6893%|           
|11|8|3.6445%|27.3338%|           
|12|8|3.6445%|30.9783%|           
|13|8|3.6445%|34.6228%|           
|14|8|3.6445%|38.2673%|           
|15|8|3.6445%|41.9118%|           
|16|8|3.6445%|45.5563%|     
|17|5|2.2778%|47.8341%|     
|18|5|2.2778%|50.1120%|     
|19|5|2.2778%|52.3898%|     
|20|5|2.2778%|54.6676%|     
|21|5|2.2778%|56.9454%|     
|22|5|2.2778%|59.2232%|     
|23|5|2.2778%|61.5011%|   
|24|5|2.2778%|63.7789%|    
|25|5|2.2778%|66.0567%|   
|26|5|2.2778%|68.3345%|   
|27|3|1.3667%|69.7012%|  
|37|2|0.9111%|82.9125%|   
|47|1|0.4555%|91.5682%|   
|57|1|0.4555%|96.1239%|   


相比，比特币已经挖出2100万总量的约 92% 。

## **HACD 挖矿和竞价**

要获得 HACD 钻石需要两个步骤，先挖出，再竞价：第一，投入挖矿算力以既定难度挖出合法的 HACD 字面值，第二，使用 HAC 竞价获得25分钟周期唯一的钻石确认机会。

**挖矿**

HACD 的挖矿难度采用独特的只升不降机制，难度调整只与已挖出钻石的数量有关，与同期多少人参与挖矿无关。

据早期矿工统计，初始挖矿难度在普通家庭消费级电脑平均6个小时能挖出一枚的难度。从算法来看，这个初始难度并不是被有意选择和设定，而是钻石字面值哈希生成算法的客观基础难度决定的。

从算法细节上看，HACD 的挖矿难度调整机制分为五个部分，一并复合作用于难度标准的计算：

1. HACD 发明的 DimondHash 算法具备基础难度，普通家用电脑大概需要6小时能挖出一枚，后续难度的调整将以此为难度基数。
2. 哈希次数每挖出 8192 枚大约半年增加一次，8年时间从 1 次哈希增加到 16 次哈希，并持续增加直到2048次哈希。相当于算力难度将在早期 8 年时间增加 16 倍；
3. 每挖出 42000 枚约 2 年，挖矿难度将增加为当前挖矿难度的 2 倍，也就是难度系数 × 2 。但随后每次难度调整的乘数将略微减少低于 2 倍，直到 64 年后最终减少到 1 为止。具体变化大概为：2.00, 1.94, 1.88, 1.83, 1.78, 1.73, 1.68, 1.64, 1.60, 1.56, 1.52, 1.49, 1.45, 1.42, 1.39, 1.36, 1.33, 1.31, 1.28, 1.25, 1.23, 1.21, 1.19, 1.16, 1.14, 1.12, 1.10, 1.08, 1.07, 1.05, 1.03, 1.02，最后系数为 1 ，也就是此维度控制的难度因素不增加了；
4. 每挖出 3277 枚大约 56 天，难度系数将调整一个 byte 的数字（256减一），也就是将最终难度哈希的左侧byte减少一位。此种难度调整按指数曲线方式变化，目前还无法计算出具体的倍数，并且前期难度增加的幅度并不大，甚至不太容易察觉的。但随着 HACD 挖出数量的增多，难度增加将越来越大，在挖矿的中间时期难度增长才会慢慢变得显著，将会超越比特币的挖矿难度，在挖矿的后期，增幅越来越大，直到超越地球上所有计算设备的计算能力。最后阶段，挖矿难度将增长到数学上的极限值，达到耗尽太阳的能量也难以挖出一颗钻石的极端程度。
5. 由于钻石的字面值为6位字母组成，总量上限为 16,777,216 颗，哈希计算的随机性特征导致可以重复挖出已经被确认的钻石字面值，则此次计算就算已经挖出合法的钻石字面值，也只能视为无效，消耗的算力作废。这就将压缩有效挖矿的空间，虽然在早期对难度的影响不大，但当挖出一半钻石时，难度将增加到原先的 2 倍，再挖出剩下的一半的一半钻石，难度将增加4倍，再挖出一半，难度增加到 8 倍。每挖出剩余钻石的一半，难度将上升一倍，以此类推，直到挖掘最后一批钻石时，这时几乎全部挖矿都大概率挖到重复的字面值，挖出新的有效的钻石字面值将非常困难，难度将增长到不可能达到的极致。
   
以上 5 个维度的联合作用导致钻石挖矿的难度调整很难用简单的参数曲线展现出来，很长时间内只能依赖与矿工的挖矿测试和统计。除非有极高数学能力且同时又是程序员的人，能将此种复合难度调整机制转换成数学公式，最后制作成图表供大家查看。

由于挖矿算力投入的不确定性和新技术的持续发展，判断未来钻石由于难度过大而停止挖矿的时间或是能够挖出的钻石数量不能确定的，也不排除研发出新的更高效挖矿硬件后，再次启动原本已经长期停止的挖矿活动的可能。唯一能确定的，数学机制上保证，钻石永远也无法全部挖出。挖掘后面部分钻石的难度将增加到比特币全部算力每天也挖不出一颗的程度，并且难度会继续增长，直到无穷大的极限。

**竞价**

通过算力挖出 HACD 并不代表已经获得这枚钻石，还需要竞价确认。每枚钻石都有一个自动增加的序号，从1开始，每挖出一枚，下一枚序号自动加一。

挖掘出钻石后，需要将挖矿参数用一笔交易包裹，提交到Hacash主网的交易池内等待确认。HACD 只可包含在能被5整除（也就是区块高度数字以0或者5结尾）的区块内，但也可以不包含钻石，比如算力太低挖不出来或无人挖钻石。也就是说，以25分钟为一个竞价周期，此周期内无论挖出多少枚钻石，最终只能有唯一一枚竞价最高的钻石能得到确认，其它挖出的同序号钻石自动作废。只有竞价成功得到最终确认的钻石需要支付其竞价费用，其它作废钻石出具的竞价费用将**不会**扣除（被视为无效交易永远无法上链）。

上一枚钻石确认后，大家再次开始挖矿，从头公平进行下一轮的钻石挖掘和竞价。由于每天平均挖出 288 个区块，则每天的钻石产量最多为 58 枚，每月1740枚，每年大约产出 21000 枚 HACD 。直到挖矿难度增加到产出开始减少或停止为止。

越早参与挖矿，难度门槛越低，算力投入的成本越低。但竞价成本不可确定，完全由市场其他参与者的出价竞争来决定，受市场整体状况的影响。

竞价机制决定了每天、每年能挖出确认的钻石的最大产量，控制了钻石的产出速度，避免早期被大矿工或者“科学家”在短时间内以资金或信息优势抢夺大部分筹码，让后来者始终有机会根据市场价值投入算力和竞价费用获取到想要的 HACD ，做到分配上的长期公平。

**HACD铸造注意事项**

HACD 的挖矿受制于多个方面，包括算力、网络和竞价策略等多个方面，要完全理解整体挖矿的方方面面需要更多的时间和实践。如果你需要尽可能多地成功铸造钻石，请注意一下几个方面：

1. **尽可能高的算力** HACD 的挖矿难度会随着时间的推移而越来越高，并且不会降低，这意味着你部署的同样设备，在单位时间内内挖到的 HACD 将越来越少，直到完全挖不出来。这意味你必须定时增加投入到 HACD 挖矿中的算力设备。以便最大化地在每一个周期都尽早挖掘出来，参与竞价，获得铸造机会。

2. **尽早参与竞价** 某些第三方的竞价程序可能会在第四个区块挖掘时才会开始竞价，并宣称这可以节省竞价费用，但这可能是一个陷阱，并不能降低竞价费，反而可能导致你丢掉铸造机会：在挖矿周期的后期参与竞价，则将导致竞价并未完成，大家还处于持续报价中，钻石就已经被打包进区块了。看起来全网的竞价费低了，但是对方可能具备更好的网络条件和高频检测程序，在你报价的间隙实时提高报价，造成绝大部分铸造机会都被别人抢走，而你只是白白在浪费算力。

3. **尽可能高的首次报价** 以前一条相同，过低的首次报价，将导致竞价的轮次太多，在达到竞价终局之前已经超越了周期时限，导致铸造机会被高频竞价检测者抢走。

4. **尽可能大的加价幅度** 加价幅度过低，比如每次只上升 0.0001 个HAC，也会导致竞价轮次太长，无法达到终局，铸造机会每次都被高频竞价者抢走。建议设置不低于 1 HAC 的加价幅度。

5. **尽可能高的最高出价** 这很好理解，你的最高报价如果超越了全网，在保证上面几条的时候，那么绝大多数 HACD 都将落入你的口袋。

6. **优化网络质量** HACD 的竞价需要实时与全球的矿工进行竞争，网络的丢包或延迟都将导致你无法及时获得别人的最新报价，或者无法把自己的报价广播出去，导致竞价不成功，丢失了你本来可以获得的钻石；

> 特别注意：每 5 个区块平均 25 分钟为一轮 HACD 的挖矿和竞价周期。但由于 PoW区块的“提前打包”特性，导致在第四个区块挖出来之后至第五个区块挖出来之前的这段时间，HACD 的挖矿和竞价都是无效的，当第四个区块被挖出来后的一两秒时间内，在那一刻处于最高竞价的 HACD 就已经被打包进第五个区块了，后续新挖出的钻石或者新出具的竞价都已经无意义了。这意味着实际上的有效挖矿和竞价时间只有20分钟。所以，尽可能地提升算力和各个维度的报价，将有助于你获得 HACD 铸造机会，否则，由于网络和技术方面的弱势，你将持续处于挖矿的不利地位。

