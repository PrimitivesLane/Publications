---
title: 'Bitcoin Optech Newsletter #180: 2021 Year-in-Review Special'
permalink: /en/newsletters/2021/12/22/
name: 2021-12-22-newsletter
slug: 2021-12-22-newsletter
type: newsletter
layout: newsletter
lang: zh
---
这份特别版的 Optech Newsletter 总结了 2021 年全年比特币值得注意的发展。它是我们 [2018][2018 summary] 年、[2019][2019 summary] 年和 [2020][2020 summary] 年总结的续篇。

## 目录

* 一月
  * [比特币核心中的 Signet](#signet)
  * [Bech32m 地址](#bech32m)
  * [洋葱消息和要约协议](#offers)
* 二月
  * [更快的签名创建和验证](#safegcd)
  * [通道拥塞攻击](#jamming)
* 三月
  * [量子计算风险](#quantum)
* 四月
  * [LN 原子多路径支付](#amp)
* 五月
  * [BIP125 的费用替换实现差异](#bip125)
  * [双边供资通道](#dual-funding)
* 六月
  * [基于候选集的区块构建](#csb)
  * [默认的交易手续费替代](#default-rbf)
  * [交易池包接收与包中继](#mpa)
  * [LN 快进及离线接收](#ff)
* 七月
  * [LN 流动性广告](#liq-ads)
  * [输出脚本描述符](#descriptors)
  * [零确认通道创建](#zeroconfchan)
  * [SIGHASH_ANYPREVOUT](#anyprevout)
* 八月
  * [忠诚保证保险](#fibonds)
  * [LN 寻路](#pathfinding)
* 九月
  * [OP_TAPLEAF_UPDATE_VERIFY](#tluv)
* 十月
  * [事务继承标识别符](#txhids)
  * [PTLC 与 LN 快进](#ptlcsx)
* 十一月
  * [LN 开发者峰会](#lnsummit)
* 十二月
  * [高级费用提升](#bumping)
* 精选摘要
  * [Taproot](#taproot)
  * [热门基础设施项目的重大更新](#releases)
  * [Bitcoin Optech](#optech)

## 一月
<p id="signet"></p>

经过数年的讨论，1 月份首次[发布][bcc21]了支持 [signet][topic signet] 的比特币核心版本，此前 C-Lightning [支持][cl#2816]，随后 [LND][lnd#5025] 也支持了。Signet 是任何人都可以用来模拟比特币主网络（mainnet）的测试网络，可以是以现在的状态，也可以是它发生某些变更后的状态（比如软分叉共识变化的激活）。大多数实现 signet 的软件也会支持一个默认的 signet，这为不同团队开发的软件提供了一个特别方便的手段，可以在一个安全的环境中进行测试，同时可以尽可能地接近他们在链上的环境。今年还[讨论][signet reorgs]将有意的区块链重组添加到比特币核心的默认 signet 中，以帮助开发人员针对这类问题测试他们的软件。

<p id="bech32m"></p>

1 月份还[宣布][bech32 bip]了一个针对 [bech32m][topic bech32] 的 BIP 草案。Bech32m 地址对 bech32 地址做了轻微修改，使其可以安全地用于 [taproot][topic taproot] 和未来的协议扩展。今年晚些时候，一个[Bitcoin Wiki 页面][bech32m page]被更新用以跟踪钱包和服务对 bech32m 地址的采用。

<p id="offers"></p>

另一个[首次发布][cl 0.9.3]的新协议是[洋葱消息][topic onion messages]和 [要约协议][topic offers]。洋葱消息允许一个 LN 节点向另一个节点发送消息，与基于 [HTLC][topic htlc] 的消息发送相比，它的开销更小。要约使用洋葱消息，允许一个节点*提议*支付给另一个节点，允许接收节点返回详细的收据和其他必要的信息。洋葱消息和要约在今年剩下的时间里继续作为草案规范，但仍然有了进一步的进展，包括一个使用它们来[减少滞碍支付的影响][offers stuck]的提案。

## 二月
<p id="safegcd"></p>

比特币的贡献者[推进][safegcd blog]了对改进的签名创建和验证算法的研究，然后利用他们的研究产出了一个有额外改进的版本。当在 libsecp256k1（[1][secp831]，[2][secp906]）和[后来][bcc21573]的比特币核心中应用时，这将验证签名所需的时间减少了约 10% ——这是一个重大改进，考虑到需要验证比特币区块链中的大约 10 亿个签名。一些密码学家致力于确保这一变更在数学上是合理的，并且可以安全使用。这一变更也为在低功率的硬件签名设备上安全地创建签名的速度提升提供了巨大的推动。

<p id="jamming"></p>

[通道拥塞攻击][topic channel jamming attacks]是 LN 自 2015 年以来的一个已知问题，在这一年里得到了持续的讨论，[讨论][jam3]了[各种][jam1][可能][jam2]的解决方案。不幸的是，没有找到广泛认可的解决方案，这个问题到年底仍然没有得到缓解。

![拥塞攻击](./image/2021-12-22-newsletter-zh-p1.png)

## 三月
<p id="quantum"></p>

3 月份的重要[讨论][quant]专门分析了量子计算机攻击比特币的风险，特别是针对 taproot 激活并被广泛使用的情况。比特币的原始功能之一，公钥散列——可能是为了使比特币地址更短而添加的——也可能使它在量子计算突然出现重大进展的情况下更难从有限的用户那里窃取资金。[Taproot][topic taproot] 没有提供这个功能，至少一个开发者担心它造成不合理的风险。大量的反驳意见被提出，社区对 taproot 的支持似乎没有改变。

----
<p id="taproot"></p>

### 2021 年总结
### Taproot 的激活
到 2020 年底，一个包含支持 [schnorr][topic schnorr signatures] 签名和 [tapscrip][topic tapscript] 的 [taproot][topic taproot] 软分叉的实现已经[并入][bcc#19953]比特币核心。协议开发者的工作很大程度上完成了；现在要看社区是否愿意激活 taproot，以及钱包开发者是否开始添加对它和相关技术的支持，如 [bech32m][topic bech32] 地址。

- **1 月**以 Bitcoin Core 0.21.0 的[发布][bcc21]开始，这是第一个包含对 [signet][topic signet] 支持的版本，包括已经激活 taproot 的默认 signet——给用户和开发者一个简单的地方来开始测试。

- **2 月**，在 `##taproot-activation` IRC 频道中举行了[系列][tapa2][会议][tapa3]中的[第一次][tapa1]，该频道将成为开发者、用户和矿工之间讨论如何激活 taproot 的主要场所。

- **3 月**开始，许多讨论参与者初步[同意][speedy trial]尝试一种特殊的激活方式，即*快速尝试*，其目的是为了快速收集矿工的反馈，但也给用户足够的时间来升级他们的软件来应用 taproot。快速尝试将成为激活 taproot 的实际[机制][topic soft fork activation]。

  在激活讨论进行的同时，对其设计的其中一个决策进行了最后的[讨论][quant]，即使用裸公钥，有人认为这可能使用户资金被未来的量子计算机窃取的风险增加。许多开发者认为，这些担忧是没有必要的，或者至少是被夸大了。

  同样在 3 月，比特币核心合并了对 [BIP350][] 的支持，允许它支付给 [bech32m][topic bech32] 地址。这种对 bech32 地址轻微变更的版本，用于支付原始 segwit 版本的地址，修复了一个可能导致 taproot 用户在一些非常罕见的情况下损失金钱的错误。(由 bech32 地址创建的原始 segwit 输出是安全的，不受这个错误的影响）。

- **4 月**，由于协议开发者和一些用户对两种略有不同的快速尝试激活机制的优点产生了[争论][tapa4]，激活的进展几乎脱轨。然而，这两个不同版本的不同实现的作者达成了一个[妥协][bcc#21377]才使得一个比特币核心[版本][bcctap]的激活机制和参数被发布。

- **5 月**，矿工们[能够][signal able]开始示意他们准备好应用 taproot，一个跟踪示意进度的网站开始[流行][taproot.watch]。

- **6 月**，矿工们[锁定了 taproot][lockin]，承诺在 6 个月后的 709,632 区块实施激活。这意味着[钱包开发者][bcc#22051]和[其他基础设施开发者][bolts#672]是[时候][rb#589]把更多的[注意][bcc#21365]力[放在][cl#4591]为 taproot [适配][rb#601]他们的[软件][p2trhd]上了，[他们][bcc#22154]中的许多人都这么[做][bcc#22166]了。此外，还有一个[提案][nsequence default]，允许链上的 taproot 钱包轻松地对使用各种合约协议（如 LN 和[硬币交换][topic coinswap]）的钱包的隐私提供帮助。Optech 也[开始][p4tr begins]了[为 Taproot 做准备][p4tr]系列。

- **7 月**，Bitcoin Wiki 的一个[页面][bech32m page]专门用来追踪钱包所需的 bech32m 地址格式的支持，以便在激活后能够使用 taproot。许多钱包和服务的开发者奋起直追，确保他们已经准备好让任何人在激活后使用 taproot。其他[软分叉提案][bip118 update]也被更新为使用 taproot 或从其激活中[学习][bip119 update]。

- **8 月**对 taproot 的发展来说是平静的，尽管写了一些与taproot有关的[文档][reuse risks]。

- **9 月**，比特币最受欢迎的开源商户软件在其计划激活前增加了对 taproot 的[支持][btcpay taproot]。它还[提出][op_tluv]了一个新的操作码，该操作码将特别利用 taproot 的特性来实现基于脚本的[契约][topic covenants]。

- **10 月**开始，随着 taproot 激活的[临近][testing taproot]，[开发][rb#644][重新][rb#563]开始。Taproot 的 BIP 已经更新，[增加了测试向量][expanded test vectors]，以帮助钱包和基础设施开发者验证他们的实现。

- **11 月**庆祝了 [taproot 的激活][taproot activation]。最初有一些混乱，因为 709,632 和随后的几个区块的矿工打包的区块并不包括任何 taproot 的花费交易。然而，由于一些开发者和矿池管理员的工作，在随后几天里开采的大多数区块都准备好了包含 taproot 花费的交易。[开发][nov cs]和[测试][cbf verification]适用于 [taproot][dec cs] 的软件仍在[继续进行][rb#691]。

- **12 月**，比特币核心合并了一个 PR，允许[描述符钱包][topic descriptors]创建 [bech32m][topic bech32] 地址来接收 taproot 付款。LN 开发者也进一步讨论了利用 taproot 的功能。

尽管为 taproot 选择激活机制很复杂，而且在激活后立即出现了一些小混乱，但在比特币中增加对 taproot 软分叉的支持的最后步骤总体看来很顺利。这并不是 taproot 故事的结束。随着钱包和基础设施开发商开始使用它的许多功能，Optech 预计在未来几年将继续花费大量的时间来撰写与之相关的内容。

----

## 四月
<p id="amp"></p>

LND 在四月加入了对原子化多路径支付（ Atomic Multipath Payments，[AMP][AMP]）的[支持][support]，这也叫初版 AMP，因为它早于现在各大 LN 实现都支持的 [“简化多路径支付（Simplified Multipath Payments）”][Simplified Multipath Payments] 提出。AMP 对比 SMP 有隐私上的优势，而且能保证接收方在开始索要支付之前收到了所有部分。缺点在于，AMP 无法产生密码学上的支付证明。LND 实现 AMP 用于 [“自发支付（spontaneous payments）”][spontaneous payments] ，而自发支付根本上是无法提供支付证明的，因此不需要考虑 AMP 的这个重大缺点。

## 五月
<p id="bip125"></p>

[BIP125][BIP125]（交易[替换][replacement]规范）与比特币核心实现的差异被[曝出][disclosed]。就我们所知，这没有导致比特币面临损失的风险，但它确实引发了一系列的讨论，关于意料之外的交易转发行为对合约用户（比如闪电网络）的风险。

<p id="dual-funding"></p>

也是在五月，C-Lightning 项目[合并了][merged]一个插件，用于管理 [“双边供资通道（dual-funded channels）”][dual-funded channels] —— 这种通道是双方都提供了一定数量的初始资金的。加上今年早些时候[合并][merged]的、关于双方充值的工作，这个插件让初始化通道的一方不仅能通过通道来花费资金，还能从通道的初始状态中接收资金。这种接收资金的初始化能力对商家来说尤其有用（他们主要使用闪电网络来接收支付，而不是发送资金）。

----
<p id="releases"></p>

### 2021 年总结
### 热门基础设施项目的重大更新
-  <a id="eclair-0-5-0" href="#eclair-0-5-0)">●</a> [Eclair 0.5.0][Eclair 0.5.0] 为可扩展的集群模式增加了支持（见[第 128 期周报][Newsletter #128]），区块链看门狗（见[第 123 期周报][Newsletter #123]），以及额外的插件钩子。
-  <a id="bitcoin-core-0-21-0" href="#bitcoin-core-0-21-0)">●</a> [Bitcoin Core 0.21.0][Bitcoin Core 0.21.0] 加入了对使用[版本 2 地址宣布消息][version 2 address announcement messages]的新型洋葱服务（Tor onion services）的支持，作为服务[致密区块过滤器][compact block filters]的可选功能；还支持 [signet][signets]（默认的 signet 已经激活 [taproot][taproot]）（译者注：signet 可以认为是一种私有的测试网，其中只有创建者签名的区块才是有效区块）。它也为原生使用 “[输出脚本描述符][output script descriptors]” 的钱包提供了实验性支持。
- <a id="rust-bitcoin-0-26-0" href="#rust-bitcoin-0-26-0)">●</a> [Rust Bitcoin 0.26.0][Rust Bitcoin 0.26.0] 加入了对 signet、版本 2 地址宣布消息的支持，并优化了对 [“部分签名的比特币交易（PSBT）”][PSBT] 的处理。
- <a id="lnd-0-12-0-beta" href="#lnd-0-12-0-beta)">●</a> [LND 0.12.0-beta][LND 0.12.0-beta] 为 [“瞭望塔（watchtowers）”][watchtowers] 增加了 [“锚点输出（anchor outputs）”][anchor outputs] 的支持，还加入了一种新的钱包子命令 ` psbt ` 来处理 [PSBTs][PSBTs] 。
- <a id="hwi-2-0-0" href="#hwi-2-0-0)">●</a> [HWI 2.0.0][HWI 2.0.0] 在 BitBox02 中加入了对多签名的支持，优化了文档，并支持使用 Trezor 硬件钱包来支付 `OP_RETURN` 输出。
- <a id="c-lightning-0-10-0" href="#c-lightning-0-10-0)">●</a> [C-Lightning 0.10.0][C-Lightning 0.10.0] 包含了对其 API 的一系列强化，以及对[双方充值][dual funding]的实验性支持。
- <a id="btcpay-server-1-1-0" href="#btcpay-server-1-1-0)">●</a> [BTCPay Server 1.1.0][BTCPay Server 1.1.0] 增加了对 [“闪电环路（Lightning Loop）”][Lightning Loop] 的支持、加入了 [WebAuthN/FIDO2][WebAuthN/FIDO2] 作为双因子验证的选项，并从此使用[语义化的版本控制系统][semantic versioning]来确定未来的版本号。
- <a id="eclair-0-6-0" href="#eclair-0-6-0)">●</a> [Eclair 0.6.0][Eclair 0.6.0] 包含了多方面的提升，加强了用户的安全性和隐私性。也为未来可能采用 [bech32m][bech32m] 地址形式的软件提供了可组合性。
- <a id="lnd-0-13-0-beta" href="#lnd-0-13-0-beta)">●</a> [LND 0.13.0-beta][LND 0.13.0-beta] 通过让[锚点输出][anchor outputs]成为默认的承诺交易格式，优化了手续费率的关联；加入了对剪枝比特币全节点的支持、允许使用原子化多路径（[AMP][AMP]）收发支付，提高了 LND 的 [PSBT][PSBT] 容量。
- <a id="bitcoin-core-22-0" href="#bitcoin-core-22-0)">●</a> [Bitcoin Core 22.0][Bitcoin Core 22.0] 加入了对 [I2P][I2P] 连接方式的支持，移除了对[版本 2 洋葱连接][version 2 Tor]的支持，加强了对[硬件钱包][hardware wallets]的支持。
- <a id="bdk-0-12-0" href="#bdk-0-12-0)">●</a> [BDK 0.12.0][BDK 0.12.0] 加入了使用 Sqlite 来存储数据的功能。
- <a id="lnd-0-14-0" href="#lnd-0-14-0)">●</a> [LND 0.14.0][LND 0.14.0] 加入了额外的 “日蚀攻击（[eclipse attack][eclipse attack]）” 保护措施（见[第 164 期周报][Newsletter #164]）、远程数据库支持（见[第 157 期周报][Newsletter #157]）、更快的寻路算法（见[周报 #170][Newsletter #170]），Lightning Pool 使用体验优化（见[周报 #172][Newsletter #172]）以及可重用的 [AMP][AMP] 发票（见[周报 #173][Newsletter #173]）。
- <a id="bdk-0-14-0" href="#bdk-0-14-0)">●</a> [BDK 0.14.0][BDK 0.14.0] 简化了给一笔交易增加一个 `OP_RETURN` 输出的操作，并增加了对给 [bech32m][bech32m] 地址（taproot 地址）发送支付的优化。

----

## 六月
<p id="csb"></p>

六月份出现的一份新的[分析][analysis]描述了一种矿工选择在区块中包含哪些交易的另类方法，预计这种新的方法可以在短期内略微提高矿工的利润。在长期中，如果矿工普遍接收了这种方法，意识到这一点的钱包可以在发送 [“子为父追加手续费（CPFP fee bumping）”][CPFP fee bumping] 交易时与矿工合作，从而提高 CPFP 的有效性。

<p id="default-rbf"></p>

另一种尝试提高手续费追加效率的[提案][proposal]允许比特币核心将任何未确认的交易都标记为 [“手续费替代（replaced by fee）”][replaced by fee] 交易 —— 不仅仅是那些自己表态使用了 [BIP125][BIP125]  手续费替代的交易。这有助于解决多方协议中的手续费追加的一些问题，也因为允许更多交易可以使用同样的格式所以能提高隐私性。有关隐私性，另一个[提案][proposal]建议钱包在创建 taproot 支付时，应该设定一个默认的 nSequence 值，即使他们并不需要 [BIP68][BIP68]（共识执行的 sequence 值）提供的功能；这将使得软件创建出的不需要 BIP68 的交易也能跟更普通的交易混同。虽然很少重大的反对意见，但这些提案似乎都没有特别大的意义。

<p id="mpa"></p>

同样在六月份，一系列在比特币核心中实现 [“交易池接受交易包（mempool package acceptance）”][mempool package acceptance] 的 PR 中的[第一个][first PR]，合并到了代码库中。这是迈向交易包转发的第一步。[交易包转发][Package relay]将允许转发节点和矿工把相关交易组成的交易包当成一笔交易，整体来考虑手续费率。一个交易包可能包含一笔费率较低的父交易，以及一笔费率较高的子交易；而包含子交易的好处将会激励矿工也打包父交易。虽然交易包挖矿在 2016 年就已经在比特币核心中[实现][implemented]了，但节点一直没有办法转发交易包，这就意味着低费率的父交易在费率暴涨期间可能根本无法到达矿工，即使它的子交易有更高费率，即使整体上打包它们有利可图。这使得 [CPFP 手续费追加][CPFP fee bumping]对于使用预签名交易的合约协议（比如闪电网络）来说是不可靠的。交易包转发有望解决这个关键的安全问题。

<p id="ff"></p>

一个最早于 2019 年为闪电网络提出的想法也在今年 6 月焕发新生。最初的 [“快进（fast fowrds）”][fast forwards] 提议描述了闪电钱包如何可以用更少的网络往返来接收和转发支付，从而减少带宽消费和支付的延迟。这个想法在今年被[扩展][expanded]成了描述闪电钱包如何可以接收多笔支付、同时无需为每一笔支付保证其签名密钥在线，使得用户更容易保证签名密钥的安全。

## 七月
<p id="liq-ads"></p>

经过数年的讨论和开发, 一个分布式的流动性广告系统被[合并][cl#4639]进一个 LN 的实现中。仍在草稿阶段的[流动性广告][bolts#878]提案允许节点使用 LN gossip 协议宣传其在一段时间内出租其资金的意愿，这让其它节点可购买用于即时付款的入账容量。看到广告的节点可使用开放的[双边供资][topic dual funding] 通道同步地支付并接收入账容量。 即使无法确保这些应征广告的节点真正为支付进行路由，此提案纳入了一个早期的提案（ [随后被用于][lnd#5709]闪电池），避免广告商在商定的租赁期结束前将钱用于其它用途。这意味着拒绝进行支付路由不会带来任何好处，反倒剥夺了应征广告节点挣路由手续费的机会。

<p id="descriptors"></p>

[首次为比特币核心被提出][descriptor gist]三年后，[输出脚本描述符][topic descriptors][草案 BIP][descriptor bips1]被 [创建][descriptor bips2]。 描述符是包含所有必要信息的字符串，这些信息用于允许钱包或其他程序追踪向特定脚本或一组相关脚本的付款或花费。（举个例子，一个地址或一组例如在 [HD wallet][topic bip32] 中的相关地址)。描述符通过 [miniscript][topic miniscript] 较好地组合起来， 允许钱包处理对更大规模脚本的追踪和签名。它也可以和 [PSBT][topic psbt] 较好地组合起来，允许钱包决定使用哪把密钥控制多签脚本。到今年年底，比特币核心的新创建钱包已经[默认][descriptor default]是基于描述符的钱包了。

<p id="zeroconfchan"></p>

一种此前从未纳入 LN 协议的常用创建 LN 通道的方式在 7 月被[指定][0conf channels]为 LN 协议。零确认通道创建，也被称为*涡轮通道*，是一种新的单一资金通道，出资人将其部分或全部初始资金给接受者。这些资金在创建通道的事务收到足够数量的确认之前是不安全的，所以接受者运用标准的 LN 协议将这些资金中的一部分向出资人支付是没有任何风险的。例如，Alice 在 Bob 的托管交易所的一个账户里有几个 BTC。Alice 要求 Bob 创建一个新的通道，支付给她 1.0 BTC。因为 Bob 相信自己不会对他刚刚创建的通道进行双花，所以他可以允许 Alice 通过他的节点向第三方 Carol 发送 0.1 BTC，这甚至可以在创建通道的事务收到一次确认之前发生。指定行为应有助于提高 LN 节点和提供这种服务的商家之间的互操作性。

![零确认通道图例](./image/2021-12-22-newsletter-zh-p2.png)

<p id="anyprevout"></p>

两条针对新的签名散列（sighash）类型的相关提案被[组合][sighash combo]成 [BIP118][]。`SIGHASH_NOINPUT`， 2017 年提出，其中一部分基于可追溯到数十年前的提案，被[`SIGHASH_ANYPREVOUT` 和 `SIGHASH_ANYPREVOUTANYSCRIPT`][topic sighash_anyprevout] 取代，它们是在 2019 年首次[提出的][anyprevout proposed]。 新的 sighash 类型将允许例如 LN 和[金库][topic vaults]等链外协议减少需要保存的中间状态的数量，大大降低了存储需求和复杂性。对于多方协议来说，由于削减了需要一开始生成的不同状态的数量，其好处可能更加显著。

## 八月
<p id="fibonds"></p>

忠诚保证保险是一个至少在 2010 就被[描述][wiki contract]的一个想法，意在锁定比特币一段时间，以便为第三方系统的不当行为创造成本。因为在时间锁定期间，比特币无法再被使用，其它系统中的用户如果在锁定期内被禁止或惩罚，便不能使用相同的比特币创建虚拟身份。在 8 月，JoinMarket [投产][fi bonds]了首个大规模分布式的忠诚保证保险的应用。截至发布日，超过 50 BTC 已被时间锁定（当时超过 200 万美元）。

<p id="pathfinding"></p>

8 月，一个 LN 的新的寻路变体被[讨论][0base]。该技术的支持者认为，路由节点根据金额收取一定比例的手续费相比为每笔支付收取最小的*基础费用*有效得多。 其他人有不同看法。到今年年底，该技术的[一个变体][cl#4771] 将在 C-Lightning 中实现。

----
<p id="optech"></p>

### 2021 总结
### Bitcoin Optech

在 Optech 的第 4 年，我们发布了 51 篇周 [通讯][newsletters]， 为[主题索引][topics index] 增加了 30 页新内容，发布了 [contributed blog post][additive batching], (在[两位][zmn guest]客座[作者][darosior guest]的帮助下) 撰写了 21 期关于 [准备 taproot][p4tr] 的系列.  总的来说，Optech 今年发表了超过 80,000 字的比特币软件研究和开发，大致相当于一本 250 页的书。 

----

## 九月
<p id="tluv"></p>

比特币开发者长期以来一直在讨论的一个功能是向一个脚本发送比特币的能力，而这个脚本可以限制哪些其他脚本后来可以收到这些比特币，一种称为[限制条款][topic covenants]的机制。例如，Alice 收到了一些她的热钱包可以花费的发送至脚本的比特币——但她想使用热钱包花费这些比特币时，仅能将其转移到第二个脚本，且有延时。在延时期间，她的冷钱包可以申领资金。如果没有申领，延时期满，她的热钱包就可以自由花费了。9 月，一个新的操作码 `OP_TAPLEAF_UPDATE_VERIFY` 被[提出][op_tluv] ，用于创建这类限制条款，尤其是利用 taproot 的既可以使用签名（密钥路径花费）也可以使用[类似 MAST][topic mast] 脚本树（脚本路径花费）来花费资金的能力，带来特别的好处。新的操作码尤其对创建 [joinpool][topic joinpools] 特别有用，可大幅提升隐私，通过允许多用户简便、无须信任地共享一个 UTXO 所有权的方式。

## 十月
<p id="txhids"></p>

在十月，比特币开发者讨论了一种新的[标记][heritage identifiers]事务的方法，用于用户选择花费哪个集合的比特币。目前，比特币是通过上次花费时在事务中的位置标记的；例如 “事务甲，输出 0 ”。一个新的提案将允许这样标记比特币，使用此前花费它们的事务再加上它们在继承关系中的位置以及它们在事务中的位置；例如，“事务乙到第 2 个孩子，输出 0 ”。有人建议这将为例如 [eltoo][topic eltoo]， [通道工厂][topic channel factories]以及[瞭望塔服务][topic watchtowers]提供设计优势，这些都造福例如 LN 等合约协议。

<p id="ptlcsx"></p>

此外在 10 月，一组提升 LN 的现存想法被打包成一个[简单的提案][ptlcs extreme]将带来和 eltoo 类似的好处，但无需 [SIGHASH_ANYPREVOUT][topic sighash_anyprevout] 软分叉甚至是任何共识改变。该建议将减少支付延迟，几乎与数据在特定路径上的所有路由节点之间的单向传输速度一样快。它还将提高弹性，允许节点在创建通道时备份它所需要的所有信息，并在数据恢复期间的大多数情况下可获得任何其他信息。它还将允许用离线密钥接收付款，特别是允许商家节点限制其密钥需要被联网计算机使用的时间。

## 十一月
<p id="lnsummit"></p>

LN 开发者举办了 [2018 以来][2018 ln summit]的首次峰会， [讨论了][2021 ln summit]的主题，包括在 LN 使用 [taproot][topic taproot] ， 包括 [PTLC][topic ptlc]，用于[多签名][topic multisignature]的 [MuSig2][topic musig] 以及 [eltoo][topic eltoo]；将规范讨论从 IRC 转移到视频聊天；当前 BOLT 规范模型的改变；[洋葱消息][topic onion messages]以及[要约][topic offers]; [无滞碍支付][stuckless payments]; [通道拥塞攻击][topic channel jamming attacks] 以及多种缓解措施；[蹦床支付][topic trampoline payments].

## 十二月
<p id="bumping"></p>

对于链上的单签事务，提升费率以鼓励矿工更快确认是相对直接的。但是对于合约协议，例如 LN 和 [金库][topic vaults]， 并非所有授权此花费的签名者在需要提升费用的时候都空闲。更糟糕的是，合约协议常要求确定的事务在一个截止时间前确认——否则诚实的用户可能会丢失资金。12 月，我们看到[发布了][publication]费用提升的研究 ，有关如何为合约协议选择有效的费用提升机制，有助于促进对这一重要的长期问题的解决方案的讨论。

## 结论

我们在本年度总结中尝试创新——描述了 2021 年二十多个值得关注的发展，却未提及一个贡献者的名字。我们感谢所有这些贡献者，并非常希望他们因其令人难以置信的工作而受到表彰，但我们也想表彰所有通常不会被提及的贡献者。

他们是花费数小时代码审核的人，是为既定行为编写测试用例以确保不会意外改变的人，是努力调试奇怪的问题以在出现资金风险前修复问题的人，是在做其它数以百计如未妥善处理便会出现大新闻的工作的人。

这份 2021 年的最后一份通讯是献给他们的。没有一个简单的方法来列举这些不为人知的贡献者的名字。 相反，我们在本通讯中省略了所有的名字，以表明发展比特币是一个团队的努力，其中一些最重要的工作是由那些名字从未出现在本通讯任何一期的人完成的。

我们感谢他们，感谢比特币 2021 年的所有贡献者。我们迫不及待地想看看他们将在 2022 年实现哪些令人激动的新发展。

*Optech 通讯将于 1 月 5 日恢复常规的周三发布。*



[topic signet]: https://bitcoinops.org/en/topics/signet/
[topic bech32]: https://bitcoinops.org/en/topics/bech32/
[topic taproot]: https://bitcoinops.org/en/topics/taproot/
[topic onion messages]: https://bitcoinops.org/en/topics/onion-messages/
[topic offers]: https://bitcoinops.org/en/topics/offers/
[topic htlc]: https://bitcoinops.org/en/topics/htlc/
[topic channel jamming attacks]: https://bitcoinops.org/en/topics/channel-jamming-attacks/
[topic schnorr signatures]: https://bitcoinops.org/en/topics/schnorr-signatures/
[topic tapscript]: https://bitcoinops.org/en/topics/tapscript/
[topic soft fork activation]: https://bitcoinops.org/en/topics/soft-fork-activation/
[BIP350]: https://github.com/bitcoin/bips/blob/master/bip-0350.mediawiki
[topic coinswap]: https://bitcoinops.org/en/topics/coinswap/
[topic descriptors]: https://bitcoinops.org/en/topics/output-script-descriptors/
[topic covenants]: https://bitcoinops.org/en/topics/covenants/
[topic dual funding]: https://bitcoinops.org/en/topics/dual-funding/
[topic bip32]: https://bitcoinops.org/en/topics/hd-key-generation/
[topic miniscript]: https://bitcoinops.org/en/topics/miniscript/
[topic psbt]: https://bitcoinops.org/en/topics/psbt/
[topic sighash_anyprevout]: https://bitcoinops.org/en/topics/sighash_anyprevout/
[topic vaults]: https://bitcoinops.org/en/topics/vaults/
[topic mast]: https://bitcoinops.org/en/topics/mast/
[topic joinpools]: https://bitcoinops.org/en/topics/joinpools/
[topic eltoo]: https://bitcoinops.org/en/topics/eltoo/
[topic channel factories]: https://bitcoinops.org/en/topics/channel-factories/
[topic watchtowers]: https://bitcoinops.org/en/topics/watchtowers/
[topic ptlc]: https://bitcoinops.org/en/topics/ptlc/
[topic multisignature]: https://bitcoinops.org/en/topics/multisignature/
[topic musig]: https://bitcoinops.org/en/topics/musig/
[topic trampoline payments]: https://bitcoinops.org/en/topics/trampoline-payments/

[2018 summary]: https://bitcoinops.org/en/newsletters/2018/12/28/
[2019 summary]: https://bitcoinops.org/en/newsletters/2019/12/28/
[2020 summary]: https://bitcoinops.org/en/newsletters/2020/12/23/
[cl 0.9.3]: https://bitcoinops.org/en/newsletters/2021/01/27/#c-lightning-0-9-3
[safegcd blog]: https://bitcoinops.org/en/newsletters/2021/02/17/#faster-signature-operations
[secp831]: https://bitcoinops.org/en/newsletters/2021/03/24/#libsecp256k1-831
[secp906]: https://bitcoinops.org/en/newsletters/2021/04/28/#libsecp256k1-906
[bcc21573]: https://bitcoinops.org/en/newsletters/2021/06/16/#bitcoin-core-21573
[bcc21]: https://bitcoinops.org/en/newsletters/2021/01/20/#bitcoin-core-0-21-0
[cl#2816]: https://bitcoinops.org/en/newsletters/2019/07/24/#c-lightning-2816
[jam1]: https://bitcoinops.org/en/newsletters/2021/02/17/#renewed-discussion-about-bidirectional-upfront-ln-fees
[jam2]: https://bitcoinops.org/en/newsletters/2021/10/20/#lowering-the-cost-of-probing-to-make-attacks-more-expensive
[jam3]: https://bitcoinops.org/en/newsletters/2021/11/10/#ln-summit-2021-notes
[quant]: https://bitcoinops.org/en/newsletters/2021/03/24/#discussion-of-quantum-computer-attacks-on-taproot
[tapa1]: https://bitcoinops.org/en/newsletters/2021/01/27/#scheduled-meeting-to-discuss-taproot-activation
[tapa2]: https://bitcoinops.org/en/newsletters/2021/02/10/#taproot-activation-meeting-summary-and-follow-up
[tapa3]: https://bitcoinops.org/en/newsletters/2021/02/24/#taproot-activation-discussion
[tapa4]: https://bitcoinops.org/en/newsletters/2021/04/14/#taproot-activation-discussion
[bcctap]: https://bitcoinops.org/en/newsletters/2021/04/21/#taproot-activation-release-candidate
[speedy trial]: https://bitcoinops.org/en/newsletters/2021/03/10/#taproot-activation-discussion
[bcc#21377]: https://bitcoinops.org/en/newsletters/2021/04/21/#bitcoin-core-21377
[signal began]: https://bitcoinops.org/en/newsletters/2021/05/05/#miners-encouraged-to-start-signaling-for-taproot
[signal able]: https://bitcoinops.org/en/newsletters/2021/05/05/#bips-1104
[taproot.watch]: https://bitcoinops.org/en/newsletters/2021/05/26/#how-can-i-follow-the-progress-of-miner-signaling-for-taproot-activation
[rb#589]: https://bitcoinops.org/en/newsletters/2021/05/12/#rust-bitcoin-589
[bolts#672]: https://bitcoinops.org/en/newsletters/2021/06/02/#bolts-672
[bcc#22051]: https://bitcoinops.org/en/newsletters/2021/06/09/#bitcoin-core-22051
[lockin]: https://bitcoinops.orghttps://bitcoinops.org/en/newsletters/2021/06/16/#taproot-locked-in
[nsequence default]: https://bitcoinops.org/en/newsletters/2021/06/16/#bip-proposed-for-wallets-to-set-nsequence-by-default-on-taproot-transactions
[cl#4591]: https://bitcoinops.org/en/newsletters/2021/06/16/#c-lightning-4591
[p4tr]: https://bitcoinops.org/en/preparing-for-taproot/
[bcc#21365]: https://bitcoinops.org/en/newsletters/2021/06/23/#bitcoin-core-21365
[rb#601]: https://bitcoinops.org/en/newsletters/2021/06/23/#rust-bitcoin-601
[p2trhd]: https://bitcoinops.org/en/newsletters/2021/06/30/#key-derivation-path-for-single-sig-p2tr
[bcc#22154]: https://bitcoinops.org/en/newsletters/2021/06/30/#bitcoin-core-22154
[bcc#22166]: https://bitcoinops.org/en/newsletters/2021/06/30/#bitcoin-core-22166
[p4tr begins]: https://bitcoinops.org/en/newsletters/2021/06/23/#preparing-for-taproot-1-bech32m-sending-support
[bech32m page]: https://bitcoinops.org/en/newsletters/2021/07/14/#tracking-bech32m-support
[bip118 update]: https://bitcoinops.org/en/newsletters/2021/07/14/#bips-943
[bip119 update]: https://bitcoinops.org/en/newsletters/2021/11/10/#bips-1215
[reuse risks]: https://bitcoinops.org/en/newsletters/2021/08/25/#are-there-risks-to-using-the-same-private-key-for-both-ecdsa-and-schnorr-signatures
[btcpay taproot]: https://bitcoinops.org/en/newsletters/2021/09/15/#btcpay-server-2830
[op_tluv]: https://bitcoinops.org/en/newsletters/2021/09/15/#covenant-opcode-proposal
[rb#563]: https://bitcoinops.org/en/newsletters/2021/10/06/#rust-bitcoin-563
[rb#644]: https://bitcoinops.org/en/newsletters/2021/10/06/#rust-bitcoin-644
[testing taproot]: https://bitcoinops.org/en/newsletters/2021/10/20/#testing-taproot
[expanded test vectors]: https://bitcoinops.org/en/newsletters/2021/11/03/#taproot-test-vectors
[taproot activation]: https://bitcoinops.org/en/newsletters/2021/11/17/#taproot-activated
[rb#691]: https://bitcoinops.org/en/newsletters/2021/11/17/#rust-bitcoin-691
[cbf verification]: https://bitcoinops.org/en/newsletters/2021/11/10/#additional-compact-block-filter-verification
[lnd#5709]: https://bitcoinops.org/en/newsletters/2021/10/27/#lnd-5709

[cl#4639]: https://bitcoinops.org/en/newsletters/2021/07/28/#c-lightning-4639
[bolts#878]: https://github.com/lightning/bolts/pull/878
[descriptor bips1]: https://bitcoinops.org/en/newsletters/2021/07/07/#bips-for-output-script-descriptors
[descriptor bips2]: https://bitcoinops.org/en/newsletters/2021/09/08/#bips-1143
[descriptor default]: https://bitcoinops.org/en/newsletters/2021/10/27/#bitcoin-core-23002
[descriptor gist]: https://gist.github.com/sipa/e3d23d498c430bb601c5bca83523fa82/
[0conf channels]: https://bitcoinops.org/en/newsletters/2021/07/07/#zero-conf-channel-opens
[sighash combo]: https://bitcoinops.org/en/newsletters/2021/07/14/#bips-943
[BIP118]: https://github.com/bitcoin/bips/blob/master/bip-0118.mediawiki
[fi bonds]: https://bitcoinops.org/en/newsletters/2021/08/11/#implementation-of-fidelity-bonds
[0base]: https://bitcoinops.org/en/newsletters/2021/08/25/#zero-base-fee-ln-discussion
[newsletters]: https://bitcoinops.org/en/newsletters/
[topics index]: https://bitcoinops.org/en/topics/
[additive batching]: https://bitcoinops.org/en/cardcoins-rbf-batching/
[zmn guest]: https://bitcoinops.org/en/newsletters/2021/09/01/#preparing-for-taproot-11-ln-with-taproot
[darosior guest]: https://bitcoinops.org/en/newsletters/2021/09/08/#preparing-for-taproot-12-vaults-with-taproot
[heritage identifiers]: https://bitcoinops.org/en/newsletters/2021/10/06/#proposal-for-transaction-heritage-identifiers
[ptlcs extreme]: https://bitcoinops.org/en/newsletters/2021/10/13/#multiple-proposed-ln-improvements
[2018 ln summit]: https://bitcoinops.org/en/newsletters/2018/11/20/#feature-news-lightning-network-protocol-11-goals
[2021 ln summit]: https://bitcoinops.org/en/newsletters/2021/11/10/#ln-summit-2021-notes
[stuckless payments]: https://bitcoinops.org/en/newsletters/2019/07/03/#stuckless-payments
[bcc#19953]: https://bitcoinops.org/en/newsletters/2020/10/21/#bitcoin-core-19953
[lnd#5025]: https://bitcoinops.org/en/newsletters/2021/06/02/#lnd-5025
[signet reorgs]: https://bitcoinops.org/en/newsletters/2021/09/15/#signet-reorg-discussion
[bech32 bip]: https://bitcoinops.org/en/newsletters/2021/01/13/#bech32m
[offers stuck]: https://bitcoinops.org/en/newsletters/2021/04/21/#using-ln-offers-to-partly-address-stuck-payments
[publication]: https://bitcoinops.org/en/newsletters/2021/12/08/#fee-bumping-research
[semantic versioning website]: https://semver.org/
[fido2 website]: https://fidoalliance.org/fido2/fido2-web-authentication-webauthn/
[anyprevout proposed]: https://bitcoinops.org/en/newsletters/2019/05/21/#proposed-anyprevout-sighash-modes
[wiki contract]: https://en.bitcoin.it/wiki/Contract#Example_1:_Providing_a_deposit
[cl#4771]: https://bitcoinops.org/en/newsletters/2021/10/27/#c-lightning-4771
[nov cs]: https://bitcoinops.org/en/newsletters/2021/11/17/#changes-to-services-and-client-software
[dec cs]: https://bitcoinops.org/en/newsletters/2021/12/15/#changes-to-services-and-client-software

[support]:https://bitcoinops.org/en/newsletters/2021/10/27/#lnd-5709
[AMP]:https://bitcoinops.org/en/topics/atomic-multipath/
[Simplified Multipath Payments]:https://bitcoinops.org/en/topics/multipath-payments/
[spontaneous payments]:https://bitcoinops.org/en/topics/spontaneous-payments/
[BIP125]:https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki
[replacement]:https://bitcoinops.org/en/topics/replace-by-fee/
[disclosed]:https://bitcoinops.org/en/newsletters/2021/05/12/#cve-2021-31876-discrepancy-between-bip125-and-bitcoin-core-implementation
[merged]:https://bitcoinops.org/en/newsletters/2021/05/12/#c-lightning-4489
[dual-funded channels]:https://bitcoinops.org/en/topics/dual-funding/
[merged]:https://bitcoinops.org/en/newsletters/2021/03/17/#c-lightning-4404
[Eclair 0.5.0]:https://bitcoinops.org/en/newsletters/2021/01/06/#eclair-0-5-0
[Newsletter #128]:https://bitcoinops.org/en/newsletters/2020/12/16/#eclair-1566
[Newsletter #123]:https://bitcoinops.org/en/newsletters/2020/11/11/#eclair-1545
[Bitcoin Core 0.21.0]:https://bitcoinops.org/en/newsletters/2021/01/20/#bitcoin-core-0-21-0
[version 2 address announcement messages]:https://bitcoinops.org/en/topics/addr-v2/
[compact block filters]:https://bitcoinops.org/en/topics/compact-block-filters/
[signets]:https://bitcoinops.org/en/topics/signet/
[taproot]:https://bitcoinops.org/en/topics/taproot/
[output script descriptors]:https://bitcoinops.org/en/topics/output-script-descriptors/
[Rust Bitcoin 0.26.0]:https://bitcoinops.org/en/newsletters/2021/01/20/#rust-bitcoin-0-26-0
[PSBT]:https://bitcoinops.org/en/topics/psbt/
[LND 0.12.0-beta]:https://bitcoinops.org/en/newsletters/2021/01/27/#lnd-0-12-0-beta
[watchtowers]:https://bitcoinops.org/en/topics/watchtowers/
[anchor outputs]:https://bitcoinops.org/en/topics/anchor-outputs/
[PSBTs]:https://bitcoinops.org/en/topics/psbt/
[HWI 2.0.0]:https://bitcoinops.org/en/newsletters/2021/03/17/#hwi-2-0-0
[C-Lightning 0.10.0]:https://bitcoinops.org/en/newsletters/2021/04/07/#c-lightning-0-10-0
[dual funding]:https://bitcoinops.org/en/topics/dual-funding/
[BTCPay Server 1.1.0]:https://bitcoinops.org/en/newsletters/2021/05/05/#btcpay-1-1-0
[Lightning Loop]:https://bitcoinops.org/en/newsletters/2019/07/03/#lightning-loop-supports-user-loop-ins
[WebAuthN/FIDO2]:https://fidoalliance.org/fido2/fido2-web-authentication-webauthn/
[semantic versioning]:https://semver.org/
[Eclair 0.6.0]:https://bitcoinops.org/en/newsletters/2021/05/26/#eclair-0-6-0
[bech32m]:https://bitcoinops.org/en/topics/bech32/
[LND 0.13.0-beta]:https://bitcoinops.org/en/newsletters/2021/06/23/#lnd-0-13-0-beta
[anchor outputs]:https://bitcoinops.org/en/topics/anchor-outputs/
[AMP]:https://bitcoinops.org/en/topics/atomic-multipath/
[PSBT]:https://bitcoinops.org/en/topics/psbt/
[Bitcoin Core 22.0]:https://bitcoinops.org/en/newsletters/2021/09/15/#bitcoin-core-22-0
[I2P]:https://bitcoinops.org/en/topics/anonymity-networks/
[version 2 Tor]:https://bitcoinops.org/en/topics/anonymity-networks/
[hardware wallets]:https://bitcoinops.org/en/topics/hwi/
[BDK 0.12.0]:https://bitcoinops.org/en/newsletters/2021/10/20/#bdk-0-12-0
[LND 0.14.0]:https://bitcoinops.org/en/newsletters/2021/11/24/#lnd-0-14-0-beta
[eclipse attack]:https://bitcoinops.org/en/topics/eclipse-attacks/
[Newsletter #164]:https://bitcoinops.org/en/newsletters/2021/09/01/#lnd-5621
[Newsletter #157]:https://bitcoinops.org/en/newsletters/2021/07/14/#lnd-5447
[Newsletter #170]:https://bitcoinops.org/en/newsletters/2021/10/13/#lnd-5642
[Newsletter #172]:https://bitcoinops.org/en/newsletters/2021/10/27/#lnd-5709
[Newsletter #173]:https://bitcoinops.org/en/newsletters/2021/11/03/#lnd-5803
[BDK 0.14.0]:https://bitcoinops.org/en/newsletters/2021/12/08/#bdk-0-14-0
[bech32m]:https://bitcoinops.org/en/topics/bech32/
[analysis]:https://bitcoinops.org/en/newsletters/2021/06/02/#candidate-set-based-csb-block-template-construction
[CPFP fee bumping]:https://bitcoinops.org/en/topics/cpfp/
[proposal]:https://bitcoinops.org/en/newsletters/2021/06/23/#allowing-transaction-replacement-by-default
[replaced by fee]:https://bitcoinops.org/en/topics/replace-by-fee/
[BIP125]:https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki
[proposal]:https://bitcoinops.org/en/newsletters/2021/06/16/#bip-proposed-for-wallets-to-set-nsequence-by-default-on-taproot-transactions
[BIP68]:https://github.com/bitcoin/bips/blob/master/bip-0068.mediawiki
[first PR]:https://bitcoinops.org/en/newsletters/2021/06/02/#bitcoin-core-20833
[mempool package acceptance]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-September/019464.html
[Package relay]:https://bitcoinops.org/en/topics/package-relay/
[implemented]:https://github.com/bitcoin/bitcoin/issues/7600
[CPFP fee bumping]:https://bitcoinops.org/en/topics/cpfp/
[fast forwards]:https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-April/001986.html
[expanded]:https://bitcoinops.org/en/newsletters/2021/06/09/#receiving-ln-payments-with-a-mostly-offline-private-key

