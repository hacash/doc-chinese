Hacash 相关货币理论和购买力稳定设计探讨
===

```
说明：本文整理自 Hacash 白皮书、多篇相关文章和中英文社区探讨，默认不一一注明出处。如有版权争议，一切权利归属原作者。
```

Hacash 的终极目标是加密健全货币，意图完成人类货币回归商品属性的历史使命。要实现这个愿景，其中非常关键的部分就是达成一种无需管理、反脆弱的购买力稳定系统，以承担货币作为支付手段、结算单位和长期盈余的核心职能。而这种稳定性的达成需要且仅能通过供应量的市场化调节来实现。

在开始分析 Hacash 的购买力稳定（以 HAC 为实现标的）的细节之前，我们将首先讨论一系列支撑 Hacash 货币原则的明确和潜在理论，并简述 Hacash 在遵循这些理论方面的特征和设计。这将有助于厘清一些有关货币的常见误区和理解 Hacash 各种抉择的内在原因。

---

#### ➫ 理论一：稳定币不可能三角

> 衍生自蒙代尔货币不可能三角理论，指任何挂钩法币或指数的锚定币，在去中心化、币值稳定和大规模流通这三个目标中，只能同时实现两个。

考虑到历史上黄金作为世界货币的时期，尽管金币的购买力一直在持续波动，但这种货币价格水平的长期缓慢变化，只要不是以突然、剧烈和人为的、完全不可预料的形式出现，则对货币职能的实际影响有限。黄金完全做到了去中心化和大规模流通，代价即为购买力存在一定波动。市场对货币的自由选择会综合考虑多方面的因素，购买力稳定只是其中之一，且只需要相对稳定而无需做到绝对。

HAC 在此三角中也选择了完全去中心化和大规模流通应用，实现类似黄金的供应量调节机制，不挂钩任何法币和指数，容许一定程度的购买力变动。几乎所有宣称同时实现了上述三个目标的稳定币项目，大多数通过各种时髦、模糊的词汇掩盖了其中心化运营的本质，而剩下少部分则是因为暂时还没有大范围进入市场来直面各种风险考验。

#### ➫ 理论二：购买力稳定的不可欲、不可达性

> 最初相关探讨见于米塞斯在“社会主义经济计算大辩论”中的论述，意指任何超越市场自由交换之外的对货币购买力稳定的公共诉求，最终都会不可避免地要么滑向滥发货币，要么进入全面价格管控。

这并不是说货币的购买力稳定不重要，而是表明货币也只是一种在市场上交易的商品，它本身天生就具有供给和需求的变化，所以它的“价格”或购买力永远无法保持不变。换言之，货币购买力只是由经济中所有支付结算及储藏行为相互作用形成的一系列交换比率的加总呈现，将不可避免地随着经济中不断波动的相对价格结构发生变化。例如，当发生战争或者严重自然灾害，或者某项科技大发现带来生产力的飞跃之时，货币购买力的快速变动不但不有害，反而是一种非常有利、有效、必须且重要的信号：通过价格机制将影响及时传导到每一个人，告诉大家发生了什么，并遵循价格变化做好准备，不论事情好坏。典型如摩尔定律的支配下的消费计算机行业，几乎每两年在性能翻倍的情况下，价格却大幅降低甚至减半，这种购买力的快速变动得益于技术进步带来的成本降低，这并不是坏事，更不应该期望由货币的调控来稳定价格。如果追求严格意义上的稳定购买力就意味着永久冻结相对价格，排除掉任何市场需求及供给变动的可能，这显然只能借助行政力量的强制性才能做到，最终将导致完全废除货币计算和市场经济。因此，除了信赖自由市场这只看不见的手之外，任何意图绝对稳定货币购买力的强制性动作都是毫不可取且无法达到的。

Hacash 中 HAC 的价格没有任何形式的人为倡议、引导和管理，而是通过设计一系列挖矿、借贷、套利等链上博弈机制，将购买力稳定交给自由市场的实时运作。

#### ➫ 理论三：商品货币理论

> 主要由奥地利经济学派发展，围绕以100%金本位为代表的完全私有、市场自由选择的纯商品货币本位，并严格排除任何行政控制或公共管理的指令和计划。

黄金不可能由任何政府当局、委员会、机构、办公室、跨国公司、国际联盟等的需要而被廉价地创造出来，以黄金为代表的商品作为货币，供应量的变化只能依赖于真实成本和生产效率，无法像纸币那样可以随意印刷，也不能借由银行家通过部分准备金制度开空头支票。简言之，追求100%商品货币本位，避免任何形式的人为裁量管理，是基于这样一个事实，即它是抵御通货膨胀或系统性崩溃的唯一有效保障。这又可被称作 `代理人风险理论` ，任何形式的权力委托机制，都会造成权力代理人决策利益偏向问题，这种风险只有纯粹的市场过程、每个人都严格根据自身利益相关地用脚投票才能规避。

事实上，区块链世界中的所谓“链上治理”等民主过程、投票决策形式，即是一种典型的公权力委托机制。这种方式看起来具有相当大的迷惑性，常被认为是去中心化的，给人的直观感受并不像政府发号施令那么显而易见，但却从效果上无甚区别。Hacash 中并不存在通过任何决策委托、治理机制或国库系统来管理 HAC 价格水平的中心机构，仅有竞争挖矿的成本机制和链上套利的风险敞口来达成其供应量的市场自发调节。


#### ➫ 理论四：判质费用理论

> 脱胎于罗纳德·科斯的交易费用理论和阿曼·阿尔钦的信息成本理论，也可称为“货币清晰成本理论”，意指能全球化、大规模日常使用的健全货币，必定需要一个最能客观清晰、可简单直接观测到的生产成本作为价值依托和价格共识。

在交易行为中，交易双方都免不了要判断对手所供商品或服务的费用，而在判断商品质量过程中付出的交易费用，我们称为 “判质费用”。人类历史上出现过无数种货币：黄金、白银、石头、甚至香烟、鸡蛋；但所有这些货币在相应的社会中都有明确的特征：在一定科技条件下，它们的判质费用是最低的，黄金和白银只有纯度这一个维度，并且检验费用很低，熔化即可；美国产的香烟都是标准化产品，因此在战后德国某段时间被民间用作货币。因此，判定货币质量的强弱，其实无异于要去确定该物判质费用的高低。判质费用越低，越适合用作货币。一个成功的货币，需要在全世界获得最为广泛的流通。不同的地域、环境、科技水平、文化习俗、社会制度等等都需要对这种货币的价值达成有效共识，如此才能不阻碍货币的自由支付结算，最大化货币流通的好处。

任何非公开市场化的内部决策过程和人为自由裁量权力都将严重影响货币质量判别的有效性和信任度。Hacash 货币生产的每个环节都基于完全实时地全球化开放竞争，每个人能及时有效地了解到 HAC 的生产成本和总供应量变动。

#### ➫ 理论五：能源货币理论

> 最初在1921由美国实业家亨利·福特提议建立一种"能源货币"，即一个以"电力单位"为基础的新货币体系。

在福特的能源货币理论下，“可供使用的能源本身”和“能源已经被消耗的证明”对于货币来说没有区别，前者是石油、煤炭，而后者就是类似比特币这种 PoW 机制生产出的电子货币。人类货币的终极形态，或许就是将文明发展最不可或缺的能源直接转化为货币本体的形式，货币所消耗的能源占人类日常所需能源的一定比例。

基于上文“判质费用理论”可知，货币的自由竞争将会使得大家选择最能够简单直接评估其价值的货币作为交易媒介，因为这样交易的双方都才愿意接受。而这种价值的最直接的体现就是生产成本，而且最好是能源价格下的直接成本。因为只有能源，才是全球所有不同背景的人口都能客观观测和直接理解的成本支出方式。

Hacash 以 PoW 为绝对基础构建整个货币体系，拥有三个原生的 PoW 货币 HAC 、HACD 和 BTC，是 PoW 货币与资产范式的极致追求与体现。

#### ➫ 理论六：货币道德理论

>  代表性讨论在吉多·许尔斯曼著《货币生产的伦理》一书中，一种从社会体制、特权、道德和伦理层面论述货币生产的公平性和正义性理论。

货币生产在双重意义上关乎正义，一方面，现代的货币生产机构依赖于既存的法律秩序，另一方面，这些法定事项本身本身便是造成持久不断通货膨胀的问题所在。法定垄断、法定货币法以及合法终止偿付不知不觉成了制造社会不公的工具，它们引发通货膨胀、不负责任和不合理的收入分配——通常是劫贫济富。这意味着我们当今主要的货币机制未来可能会被废除：中央银行、不兑现纸币和部分准备金银行制度。这样的措施并不是单纯的破坏，相反这可以被视为重建合理的货币体系以及建设更为人道的经济的必要条件。

Hacash 的白皮书开篇名义写到：“我们走到了货币体系变革的时代关口”，着重讨论了强制性法币和部分准备金银行制度这些造成严重社会分配不公的内在问题，为 Hacash 的各方面货币机制设计的核心理念指明了道路。

#### ➫ 理论七：代际分配理论

> 货币分配的本质就是财富的分配，财产分配的固化是阶层固化的核心，阶层固化带来经济活力的萎缩，最终将锁死文明的发展。

《比特币本位》一书的作者 Saifedean Ammous 认为认为：“理论上，理想的货币供给量应该是被锁死的，这样没有人能生产出更多的货币。这样的社会中，唯一合法的赚钱途径就是为他人创造有价值的东西，然后与他们交换。”。这是严重错误的，是为比特币总量上限的既有缺陷生硬编造出的理由。货币数量问题就是分配问题，如果分配给谁不是重点，那么法币也就没有了缺陷，比特币则根本不需要存在。

所以，货币的供应量绝对不能锁死，而一定要适度的“可持续生产”，要让后来加入货币分配的其他人和子孙后代们能通过参与货币生产来对冲消除这种“坐享其成的剥削式生活”，通过某一部分专业的人参与新货币的生产，而其他大部分人从事其他行业，通过利润率的竞争来平衡抹除这种剥削效应。如果做不到这一点，这些后来者就会简单地“另起炉灶”，如果他们被权力阻止，就会干脆“掀桌子重来”。任何固定上限的货币体系都将面临致命的“首次分配悖论”。

HAC 没有为总量设定任何固定上限，也没有任何预挖或抽税。流通量可以增加、减少或保持不变，并且采用先增后减最后保持不变的方式设置区块奖励规则，第一年每个区块奖励一枚，6年后每个区块奖励才增加到8枚，然后在用60年时间缓慢下降到1枚，此后永远保持每区块1枚 HAC 奖励。这种极致公平的货币产出设计，将有效避免货币份额向极早期的少数参与者高度集中，是目前市场分配最为公平的货币项目。

#### ➫ 理论八：货币的非协议性

> 货币仅是一种自由市场选择出来的、可能随时被替换、淘汰的商品，而不是大家可一致约定、永远不变的协议。

将货币描述为公共产品的经济学派和把货币比作Tcp/Http协议的比特币信徒， 都未能认识到货币本身几乎100%属于商品，且可与协议解耦。货币政策属于商品特质范畴，任何涉及到不同群体的不同价值影响的东西，本质上都只能是商品。与价值无关的约定才是协议。商品的本质是：人们永远会改变习惯而重新选择更好的那个，抛弃掉旧的不好的。谁能更好满足需求，客户就会换谁。比如衣服、食物、房子、汽车、音乐、洗发水、股票、货币。我们才不管什么习惯不习惯，我们就是要新的更好的汽车、房子，买更赚钱的股票，使用更优质稳固的货币。而协议的本质是：规格或方式一旦定下来，一般不存在更好的选择。比如键盘布局、行人靠右还是靠左、红绿灯颜色、http协议、度量衡单位。 比如已经选择了行车靠右，那再突然改成行车靠左是完全没有意义的。这种“靠左靠右都一样”的价值无关规则才能是协议。

只有协议才存在先发优势不可撼动的“路径依赖锁定”，而商品不存在。货币作为商品，所以也不会出现“大家已经做出了选择，不可能再更改他们的习惯”这种情况。更好的货币始终会取代差的，尽管这个过程需要时间。

#### ➫ 理论九：渐进更替理论

> 更好货币的市场推进，不需要所有人同时一致同意，也不需要任何权力的“强制共识”，由市场过程一步步扩张迭代。 

如普通商品的市场占领过程一样，货币被人们使用起来，并不是一夜之间一蹴而就，不必要也不应该依赖于任何法令，而是由每个参与者自由选择、自主切换。货币能否承担其交换媒介、记账单位和价值储存的职能，只与选择这种货币的群体有关，而不需要整个人类的全部认可。甚至在同一个区域、同一个群体中可以同时存在两个或更多个货币。货币与货币之间并不完全是你死我活的关系，而是存在某种相互补充的需求满足过程，无论这种并行情况是由于习惯的原因还是暂时认识不足导致。

对于 Hacash 以及任何一种货币来说，要取得市场的认可都需要时间。就像历史上的贝壳、粮食、牛羊、贵金属等实物货币的转换过程一样，有时候这个更替所需的时候会很长，并且都会经历一个从极少数人认可到多数人使用起来的缓慢共识过程。市场规模小并不代表这种货币没有前景，而当下占有率高的货币并不意味着能永远保持其市场份额，也不表明其肯定更优质。

#### ➫ 理论十：货币收敛理论

> 此理论阐述和证明这样一个现象，既在同一个经济体中，始终存在一种驱动力或者系统指向，会综合筛选出唯一一个最优货币来承担支付、流通和储值智能，而其它相比不那么良好的类似替代品则将逐渐退出货币流通领域，并最终消除其货币溢价。

不同种类的货币可以长时间共存，也存在不同程度的相对优劣之分，但最终将有动力收敛至最优的那个。由于交换和会计的便捷性，所有人都使用同样一种货币作为支付手段和结算单位的经济体，其货币带来的发展效率是最高的。但这并不意味着使用行政力量强制规定只允许某个指定商品或纸币能够作为货币流通能达到同样的好处，这种收敛应该也只能是在自由市场中有每一个分散个体自由选择的结果。

#### ➫ 理论十一：财富押注效应

> 货币的更替将带来也许是经济活动中最高程度的财富转移，最后一批放弃劣质货币的群体将成为时代的牺牲品。

经济体内承担货币职能的物品会有极高的“货币溢价”，这种溢价会远远超越绝大多数普通商业投资机会的利润。比如普通投资10年十倍，投资未来货币一旦成功就可能是10年一亿倍回报。越早投资，回报率就越高。现在拥有更多的货币份额，就等于是拥有更多的未来财富份额。如果你押注得越晚、成了最后一批“跳船”的人，你的财富就会被剥削缩水，甚至跟随信用崩溃的纸币或其它垃圾资产一起归零。不会或没动力跟随最新货币资产新老更替的人，无论之前财富规模如何，都将会从财务上被慢慢淘汰，甚至一生积蓄化为乌有。

这就是为什么有人倡导所有人都应该将至少1%的金钱买成比特币，也是为什么我们应该始终充分关注和识别更优质货币是否出现的的原因，并根据自己的判断和资产状况提前进行押注。

#### ➫ 理论十二：实际用途理论

> 对于现代成熟经济体来说，货币在除了承担货币职能之外，不需要其它“实际”用途。

尽管历史上实物货币几乎都能找到一些用途，但某些用途其实是来自于它成为货币之后，比如黄金的收藏、收拾和制作工艺品、艺术品。现代经济高度发达，货币流通传播非常之快速，没有任何什么实际用途反而能更好承担货币职能。一旦货币具备某种工业或生活用途，其不可避免的就会被磨损、消耗，并很可能与货币所需要的其它特质相冲突。它唯一的用途就是“有价值”，能换取到市场上几乎任何其它商品。

#### ➫ 理论十三：货币商誉理论

> 也可以被称之为货币的“信任成本理论”。指由于货币居于经济体的绝对核心地位，任何对其价值信任度损害的后果都将被极度放大，所以相比其它普通商品来说我们更加不能也不必要容忍其作恶。

货币的影响是如此之大，以至于应该严格审视其历史并时刻注意目前动作。那些在过去有过作恶行为的货币不值得原谅，浪子回头没有用，信任塌陷容易，重建极难，历史错误将导致持续的不信任感。这将最终形成一种货币“原罪论”，那些一开始分配不公或特质欠缺的货币标的，是无法通过后来改良从而变得优质的。更不可能超越那些从出生起，并从始至终都保持良好特质的货币。

#### ➫ 理论十四：预期稳定理论

> 金融领域内某些细小的扰动会经由传导效应起到放大器的效果，这是因为市场是由受周遭影响决策的人所组成的，而不是纯机械结构。

当市场存在某些客观购买力调节机制时，类似黄金的产出与流通，大家就会通过预期自发形成负反馈循环，从而几倍、十几倍的放大作用于稳定性。这意味着相比于没有任何供应量调节机制的货币，存在市场供需调节系统的货币将引发部分参与者的提前行动，从而在市场操作层面更加有助于其稳定性。

事实上，预期管理是当今美联储几乎全部运作规则的指导性理论，包括任职主席的鹰派或鸽派发言，以及公开市场操作。但这并不意味这这种近乎“舆论操纵”的行为是可取的或始终有效的，最终市场将会识破语言的伎俩，并按照实际发生的事情来做出回应。

#### ➫ 理论十五：PoS成本问题

> PoS 货币发行的成本并不是其质押代币的价值。PoW货币发行的成本明确可计算，PoS的发行成本其实无限趋近于0。锁仓的成本几乎没有，类似于把黄金放在家里就自动生出小块黄金。

一种密码学货币越是把安全预算直接花在电力上，其安全性就越高；越是把钱花在商业运营上，其安全性就越低。权益证明没有（也不可能）消除矿工的开支，只是把 PoW 系统支出在电力上的部分转成了资本开支。锁定资本的外部性，相比电力消耗的外部性如何，是复杂而微妙的问题 —— 但糟糕的是，PoS 系统的支持者往往假装电力是（系统运行）唯一需要付出的代价。

显而易见，PoW 才是能源利用最高效和最直接的方式，PoS 机制充满了隐形的、难以评估的社会活动成本。最终，所有形式的 PoS 都会变成对 PoW 生产方式的一种间接的、低效的、蹩脚的模仿。

---


在明确了上述若干理论和观点之后，我们可以认识到 HAC 的货币购买力稳定性设计只能以 PoW 为基础，并排除任何直接或间接的人为管理环节。

HAC 供应量的产出和缩减完全经由公开市场自发运行，无需任何机构的设定或指令，也无需挂钩任何指数，并分别分为长期供应调整和短期波动平抑两部分：

### 流通增加

长期调节：

1. 区块奖励：先增后减最后保持不变
2. 通道链奖励：约年化1%

短期平抑：

3. HACD 和 BTC 抵押贷出
4. 单向转移奖励：每枚BTC每周1024枚HAC，并持续减少

### 流通缩减

长期调整：

1. HACD 铸造竞价销毁
2. HACD 和 BTC 借贷合约利息销毁，约 1% 

短期平抑：

3. HACD 和 BTC 赎回销毁
4. 扩容设施租金（例如L3账本更新）销毁

以上供应量调节机制已经是部分被证明有效的系统，例如当前区块奖励从1枚增加到了5枚，矿工人数并没有减少，约11%的流通量被存入通道链网络，且 HACD 的竞价已经销毁了流通量的一半以上。
