---
layout: post
title: Mastering GMX
date: '2023-05-04 20:14:35 +0800'
tags: [defi]
math: true
---

![Desktop View](/assets/blogImg/20230504/image-20230505161658776.png){: width="972" height="589" }

# Intro

### 协议简介

![Desktop View](/assets/blogImg/20230504/gmx.drawio.png){: width="972" height="589" }

GMX是Layer2（Arbitrum, Avalanche)上的**去中心化永续合约交易所**，提供：

1. 主流非稳定币与稳定币组成的指数基金产品

2. 50x杠杆永续合约

3. 零滑点、低手续费现货兑换

**GMX主协议的用户分类**：

1. Trader，永续合约和现货交易者

2. Liquidity provider, 协议（Arbitrum）支持四种主流非稳定币标的以及四种稳定币资产，各种资产按照一定比例组成指数基金，定义为GLP。LP可以提供单一资产来mint GLP，持有GLP可以获得70%的协议手续费分成。GMX通过动态手续费率来维持GLP的各组成资产实际比例的动态平衡。

   LP的风险分析：

   - 指数基金自身的价值变动：1. GLP持有者有50%非稳定币现货风险敞口，当非稳定币价格下跌，GLP价值也会按比例下降；2. 平台调仓带来的价值变动
   - LP作为合约交易者的对手方，要承担对赌亏损的风险

**GMX的双币模型**：GLP&GMX，治理代币GMX持有者获得30%协议手续费分成。

### 运行现状

Arbitrum项目TVL ranking，GMX占整个Arb TVL的40%，也是衍生品赛道的头部产品

![Desktop View](/assets/blogImg/20230504/Arbitrum-TVL-ranking.png){: width="972" height="589" }

从历史上看，GMX 的交易者累计亏损，由此带给 GLP 的平均年化回报在 20% 以上。从统计上，交易者失败是大概率事件，而高杠杆将增加其亏损概率。但近两个月交易者整体盈利。

![Desktop View](/assets/blogImg/20230504/image-20230505180718547.png){: width="972" height="589" }

GLP持有者手续费收益长期维持在20%左右

![Desktop View](/assets/blogImg/20230504/feeAPR.png){: width="972" height="589" }

### 迭代历史

**XVIX**

![Desktop View](/assets/blogImg/20230504/XVIX.png){: width="972" height="589" }

XVIX发行了总量20万的XVIX代币，初始供应量10万用于uniswap作LP，另10万弹性供应。

玩法：用户可以存入ETH来mint XVIX，也可以用XVIX来赎回ETH。存入和赎回都由一个公式来决定XVIX的地板价：

Mint:

$$XVIXOut = ethIn \times \frac{maxSupply-totalSupply}{ethReserve+ethIn}$$


Redeem:

$$ethOut = 0.9 \times XVIXIn\times \frac{ETHReserve}{totalSupplyOfXVIX}$$


1. mint的人越多，XVIX地板价越高，到$maxSupply = totalSupply$时价格无限大
2. 越晚redeem，单位XVIX能赎回的ETH越多。比如：协议内有100ETH和100XVIX，用户A用10个XVIX能赎回$0.9\times10\times\frac{100}{100}=9 eth$，之后用户B用10个XVIX可以赎回$0.9\times10\times\frac{100-9}{100-10}=9.1 eth$。
3. 每笔transfer会burn 0.43%的XVIX；不在项目方指定的质押池或者是uniswap的LP池的XVIX每小时会被burn 0.02%

对于项目方来说：1. 鼓励资金流入，让越来越多的人来mint XVIX；2. 抵触资金流出，通过各种手段让流入的资金尽可能锁在协议内。合理认为项目方是最大的XVIX持有者

对于投机者的盈利方式：1. 有越来越多的人在自己之后去mint XVIX并且持续持有，等别人先redeem以及burn机制导致的代币地板价上升；2. 在协议和uniswap流动池间利用价差套利

总结：没有解决实际需求，没有内生价值，浓厚的庞氏色彩

**Gambit**

Gambit和XVIX是完全独立的两套系统，Gambit提供了真正意义上有价值的Defi产品，打破了后者的庞氏框架。Gambit是GMX的雏形，区别在于Gambit选择了以稳定币（USDG）为突破口，提供杠杆交易业务，并利用杠杆交易转移稳定币的风险。

机制示例：

1. 假设BNB当前价格为$200，用户A质押1BNB铸造200个USDG
2. 用户B用1BNB开2x Long，此时池子里总资金为2BNB
3. 假设BNB价格上涨10%至220，用户从池子里拿走价值240美金的BNB，此时池子里仍有价值2 * 220 - 240 = 200美金的BNB来支持用户A的200USDG
4. 假设BNB价格下跌10%至180，用户只能从池子里赎回价值160美金的BNB，此时池子里仍有价值2*180 - 160 = 200美金的BNB来支持用户A的200USDG

**小结：USDG的持有者是天然的空方，由杠杆交易的多方来支付或者吃掉做空的收益或损失**，这样将稳定币的清算风险从平台转移给杠杆交易者

其他风险保护机制：

合约开平仓、USDG铸造、销毁都会产生手续费，手续费的一部分会用来补贴USDG的持有者；50%会用来作为抵押品存入资金池中，使得USDG随着时间推移会变成超额抵押的稳定币

USDG的产品优点：

1. 1:1等额铸造，不需要用户超额抵押，并将平台资不抵债的风险转移给杠杆交易的多头用户，资金效率高于MakerDao这种超额抵押的协议
2. 白名单资产可以零滑点兑换

**GMX**

![Desktop View](/assets/blogImg/20230504/GMX.png){: width="972" height="589" }

Gambit提供了一个资金效率高、用户体验好的稳定币和杠杆交易产品，但结果论地说有一个比较明显的缺陷，即USDG持有者只能作为杠杆交易多头的对手方，随着USDG发行量的逐步增加，在标的资产价格下跌时，多头要同时赔付空头的盈利和吃掉USDG持有者原本的损失，协议的多空配比会长期处于失衡状态，使协议穿仓的风险增加。

USDG的铸造者实际上在协议中扮演的是LP的角色，所以不如直接用代表流动性份额的、价值随资金池资产价格变动的LP代币替换USDG，将LP从原来的空头变成中性的单纯提供资金的角色，这样就解决了风险失衡的问题。

Gambit成功的核心在于提供了较好的合约交易用户体验，产生了高额的手续费收益吸引外部资金不断进入协议铸造USDG来获取收益分成，从而进入了交易量增加=>流动性增加=>交易量增加的正向循环。所以当然要完善合约交易的业务逻辑，这样稳定币板块被砍掉也就很好理解了。

**core dev：**

![Desktop View](/assets/blogImg/20230504/coreDev-gmx.png){: width="972" height="589" }

### GMX生态

![Desktop View](/assets/blogImg/20230504/GMX-lego.png){: width="972" height="589" }

## 主协议机制设计

1. GMX玩法本质上是用户和LP对赌，用户杠杆交易相当于向GLP池借出资产。

2. 合约价格与手续费：

   合约成交价格完全由标记价格决定，用户的交易不会影响价格，导致了资金费的收取规则和常规的合约交易所有所差异，多方和空方在开仓和关仓时需要支付0.1%的手续费，除此之外每1h还需支付给LP borrow fee，borrow费率与该种资产的资金利用率相关，资金利用率越高费率越高，最高为每小时千一

3. 资金池设置：

   GMX所有资产都在一个pool里。好处是：

   防止流动性分散，提高pool深度

   提供方便的swap功能，基于此可以支持任意的白名单资产进行开仓或进行结算

4. 记账方式：

   将所有的资产根据混合喂价转换成相应的USD价值，设定稳定币1 USDG = 1 USD，使用USDG作为中介代币记账和进行价值转换；池子中持有各种资产的价值，也转换成USDG的数量进行记录

   比如：用户要将ETH兑换为USDT，在使用喂价是不是直接使用ETH ⇒ USDT的价格，而是分别使用ETH ⇒ USD, USDT ⇒ USD的价格，实际交易path为：ETH ⇒ USDG ⇒ USDT，相当于使用ETH购买USDG，再使用USDG购买USDT

    ![Desktop View](/assets/blogImg/20230504/GMX-assets.png){: width="972" height="589" }

5. 交易结算规则：

   不管是开多还是开空，合约都是U本位结算

6. 交易资产限制：

   用户进行杠杆交易时，对于多空仓位，其抵押品资产有不同限制：

   - 开多，只能使用标的代币（非稳定币）作为抵押品

   - 开空，只能使用稳定币稳定币作为抵押品

   - 结算资产与抵押品资产保持一致，例如使用ETH开多，抵押品为ETH，盈亏会以ETH结算；使用USDT开ETH的空单，盈亏会以USDT结算

   - 由于GMX提供了代币间swap功能，所以上述限制只体现在协议核心逻辑中，实际用户可以使用任意白名单资产开仓，但是内部规则仍然遵循上述限制。eg：用户要开ETH的多仓，用户可以使用任何资产作为保证金，比如USDT或BTC，不管提供何种资产，都会按照当时的价格转换成USD价值。由于协议要提供100%的准备金，所以合约会提供ETH来补齐用户仓位，用户平仓时盈亏以ETH进行结算
   
    ![Desktop View](/assets/blogImg/20230504/Leverage-trading.png){: width="972" height="589" }

    == 稳定币开多为什么不行，非稳定币开空为什么不行 ==
   
   考虑LP的资金利用率，开哪个币种的多仓，就向LP借哪个币种；用哪个币种开空，就向LP借哪个币种。这样池子里的所有资产都可以被利用到
  
7. 流动性及swap部分机制设计：
   - 添加流动性：
     LP持有WBTC或其他白名单资产，在添加流动性时，GLP manager将资产从用户账户转移到vault，vault向GLP manager发放USDG债务，GLP manager向LP用户发放GLP。相当于GLP manager是USDG的一个中间承兑方

    ![Desktop View](/assets/blogImg/20230504/addLiquidity.png){: width="972" height="589" }

   - 移除流动性时，GLP manager会根据用户需要赎回的GLP数量和价格计算出其USDG价值并burn掉用户的   GLP代币，随后GLP manager将相应的USDG债务卖出给vault，赎回的pool资产会由vault转给LP
  
    ![Desktop View](/assets/blogImg/20230504/removeLiquidity.png){: width="972" height="589" }

   - swap
    ![Desktop View](/assets/blogImg/20230504/swap.png){: width="972" height="589" }

8. 合约交易机制：
   
     1. 需求：
         - 用户随时开/平仓，计算多空用户即时的P&L
         - 计算每个用户的fundingFee（borrowFee）
         - LP随时进出，需要计算**GLP定价**，计算**LP的即时P&L**，计算**即时的LP总资产价值**（AUM）

     2. 实现：
         - fundingFee计算 
     
           1. 用户：entryFundingRate

              当用户仓位发生变化（开仓、清仓）时更新

           2. 全局：cumulativeFundingRate
              每当有用户和合约进行交互时检查更新

              $$borrowFeeRate = \frac {assets  Borrowed}{LPAssetsInPool} \times 0.01\%$$

              $$cumulativeFundingRate = cumulativeFundingRate_{before} + \frac {now - lastUpdatedTime}{1h} \times borrowFeeRate$$

              $$fundingFee=size\times(cumulativeFundingRate-entryFundingRate)$$

           3. 收费规则：每当用户增/减仓位时收取该段时间内的fundingFee

         - 用户的即时P&L
  
           - 仓位机制：一个方向（多or空），一种标的只能持有一个仓位，合约会对同方向、同标的的仓位在开仓时进行自动合并
  
           - averagePrice：持仓均价，标的物⇒ USDG价格；增加仓位会影响持仓均价，减仓/关仓不会影响持仓均价
            $$ Price_{average}= \frac{PositionCost_{total}}{PositionSize_{total}} =\frac {Price_{averageBefore}\times PositionSize_{before} + Price_{now}\times\Delta PositionSize}{PositionSize_{before}+\Delta PositionSize} $$

           - P&L计算
  
              $$ P\&L_{long}=(Price_{now}-AveragePrice)\times PositionSize$$

              $$P\&L_{short} = -(Price_{now}-AveragePrice)\times PositionSize$$

           - LP的即时P&L

              $$ P\&L_{LP}=-(P\&L_{globalLong}+P\&L_{globalShort})$$
              - 全体空头的P&L:
                - globalShortPositionSize[indexToken]: 每当有用户仓位变化时更新
                - globalShortPrice[indexToken]：每当有用户增加仓位时更新

                  $$P\&L_{globalShort}=\sum-(Price_{now_i}-Price_{globalShortAverage_i})\times globalPositionShortSize_i$$

                  注：在vault合约记录globalShortPosition信息时没有对DAI，USDT等不同抵押品作区分，即认为不同的稳定币抵押品的价格都一致，即为$1。GMX使用额外的shortTracker额外记录了一份全局空头的状态，和vault的取加权平均以解决稳定币价格脱锚的问题。当前的合约shortTracker状态的权重是1

           - 全局多头用户的$ P\&L$:
  
            $ P\&L_{globalLong}$的计算方式与$ P\&L_{globalShort}$不同

            $$ P\&L_{globalLong}=\sum (BorrowedAssetValue_{now_i}+CollateralValue_{now_i}-BorrowedAssetValue_{before_i}-CollateralValue_{before_i})=\sum(BorrowedAssetValue_{now_i} - BorrowedAssetValue_{before_i})$$

           - GLP定价
            $$ Price_{GLP}=\frac {LiquidityValue}{GLPSupply}$$

          例：用户在价格$1500用1 ETH开10x多仓，则用户仓位10ETH，价值15000USDG；仓位手续费0.01%，即0.01ETH；仓位中：用户抵押品1485USDG，LP借给用户9.01ETH
          假设ETH价格上涨到$3000，则此时用户P&L为：（3000- 1500）/ 1500 x 15000 = $15000。
          用户全部平仓，假设平仓手续费+资金费为35USDG，则用户最终收到（1450 + 15000） / 3000 = **5.48ETH**







1.  风控机制：

   - 100%储备金机制

     协议为每个用户实际预留了其仓位大小的准备金，当vault的全部资金都被借出时，则无法开仓。可以保证多头的盈利全额兑付

   - buffer机制

     GMX给每种资产设置了buffer amount，确保每种资产都至少有一定的数量可以用于合约交易，而不会因为swap被掏空

   - 限制仓位，限制杠杆，清算

     - 杠杆不能超过50x开仓，在持仓期间如果杠杆超过50则会被自动平仓
     - 仓位抵押品价值要必须足以支付开仓手续费，降低坏账率

   - 限制快进快出

     开仓后有三个小时cool down，在此期间如果平仓，有最小收益率门槛，如果收益小于设定的门槛，则将平仓部分的P&L设为0；如果大于则按原收益结算

   - 限制价格抢跑：

     - 限制gasPrice，防止被抢跑（但目前没有加上）：

       一种场景，给更新喂价的交易设定一定gas，限制所有开/关/swap交易gasPrice小于喂价交易的gas，可防止喂价交易被抢跑

     - 将用户的交易拆分成挂单-成交两笔交易，实际成交交易由keeper延迟触发，使得套利者无法利用价格抢跑套利

2.  预言机与mark price合成

   - maxPrice 和minPrice，对不同的场景设定获取不同的price，降低攻击者利用价格套利的风险，也可以归属于风控的一部分

     比如：

     - swap asset A for B时，获取A ⇒ USD的minPrice和B ⇒ USD的maxPrice，使得能够兑换出的asset B数量比实际稍少一点
     - 在开ETH的多仓时，获取ETH ⇒ USD的maxPrice，使得用户的持仓价格比实际稍高
     - 在开ETH的空仓时，获取ETH ⇒ USD的minPrice，使得用户的持仓价格比实际稍低
     - 在用户使用asset X开仓时，获取asset X ⇒ USD的minPrice，使用户的抵押品价值比实际稍低
     - 在用户使用asset Y结算收益时，获取asset Y ⇒ USD的maxPrice，使用户能获得的asset Y数量比实际稍低
     - 在计算AUM时，获取各toke的maxPrice，使得AUM比实际稍高，GLP定价比实际稍高

   - mark price合成

     1. chainlink喂价：取最近三次的chainlink价格，如果取maxPrice，则取三次中的最高价，反之取三次中的最低价

     2. 非稳定币：结合中心化喂价合成mark price

        - 中心化喂价和chainlink喂价在一定周期内的价格累计变化量是否超过10%，

          ```
          // 初始100
          // fast: 110(10%), ref: 105(5%) diff: 5%
          // fast: 121(10%), ref: 110.25(5%) diff: 20% - 10% = 10%
          ```

        - 中心化喂价和chainlink喂价的最新价格$\Delta$是否超过10%

        - 如果上述约束都满足，则使用中心化喂价

        - 如果不满足任意一个，则：

          - maxPrice：max(中心化喂价，chainlink喂价)
          - minPrice：min(中心化喂价，chainlink喂价)

   - 稳定币mark price，稳定币只使用chainlink喂价，没有中心化喂价：

     - 如果稳定币价格在 [$0.995, $1.005]之间，视为其价格为$1

     - maximise ? max(chainlinkPrice, $1）：min(chainlinkPrice, $1)

     - 场景：如果稳定币脱锚至0.95，则开仓时取minPrice以$0.95开仓，平仓时取maxPrice以$1结算。例如：用户用100 USDC开空，USDC价格为$0.95，则用户抵押品价值$95。如果不考虑手续费且标的价格无变动，用户全部平仓，能收回95 / 1 = 95个USDC

  ​		

## 代币经济学设计

在 GMX 里面设计了很多种 token，GMX，GLP，exGMX 等。其质押的核心逻辑还是 masterchef 方式。在多种 token 的交互中，主要的设计思路是质押，托管，兑现（vesting）

抛去治理功能外，GMX的主要作用还是stake to earn。Stake GMX可以获得：

- esGMX
- Multiplier Points
- ETH / AVAX Rewards

对于赚取ETH/AVAX其实最容易理解，因为这就是平台各种手续费。手续费分成是30%，根据你质押在不同链上，获得对应的ETH或AVAX的手续费奖励。这里面不一样的点在于esGMX和Multiplier Points。

**esGMX**

esGMX有两种使用方式：

1. 像GMX一样被质押以获取相同的奖励
2. 在一年的时间内被线性解锁为GMX

假设你本身质押了1000个GMX并且获得了100个esGMX, 这100个esGMX可以被stake到esGMX的池子，也能获得ETH/AVAX手续费奖励和奖励乘数。但是这些esGMX是不可转让的。想要变成可以转让/交易的GMX，则需要把esGMX质押到vesting池子里，这样esGMX将在365天内线性变成GMX，变成的GMX随时可取。上述两种玩法是互斥的。

**Multiplier Points**

**奖励乘数。**在其他项目去获得收益的时候往往会伴随着通膨的问题：即随着staker的增多，分到每个人头上的奖励逐渐减少。抛开上述的esGMX可以减少通货膨胀带来的抛压问题以及保持先入场优势外，GMX提供了Multiplier Points这种奖励乘数的形式减少通胀对先入场者权益的稀释。

所以当你stake GMX时，每秒都会以固定速率（和代币1：1）获得奖励乘数。质押1000个GMX一年将会获得1000点奖励乘数。每个奖励乘数将获得与普通GMX代币相同的ETH/AVAX奖励。但是一旦你unstake GMX，对应部分的奖励乘数将会被销毁。

而奖励乘数提升的手续费奖励百分比为：100*（staked奖励乘数）/ (staked GMX+staked esGMX)

举个例子：假设目前你的ETH手续费收益率是10%，你总的质押的GMX和esGMX是10000刀，质押的奖励乘数是它们的20%，那么你的预计收益就是1200刀。

**小结：**

GMX的成功来自于其两驾马车：1. 优秀的合约和现货交易用户体验吸引外部资金流入源源不断给GLP和GMX输血；2. 优秀的代币经济学设计使流动的资金尽可能地锁在协议中，二者缺一不可，但对GMX来说更重要的仍然是代币经济学的设计，不管是稳定币还是合约交易，产品起到的作用只是保持较高的协议收入来维持GMX的经济模型的持续运转，只要能达到这一目的的就是好产品，可以说GMX是为了经济模型这一盆醋包的合约交易这一盘饺子。而从结果上来看，GMX的经济模型设计得非常成功，目前有超过75%的GMX处于锁仓状态

![Desktop View](/assets/blogImg/20230504/image-20230508113442116.png){: width="972" height="589" }
