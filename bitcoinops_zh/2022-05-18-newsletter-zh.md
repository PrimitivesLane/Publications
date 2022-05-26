---
title: 'Bitcoin Optech Newsletter #200'
permalink: /zh/newsletters/2022/05/18/
name: 2022-05-18-newsletter-zh
slug: 2022-05-18-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---
本周的 Newsletter 总结了关于为实现递归契约而对比特币脚本语言所做的最小改动的讨论，审查了一个添加 `OP_TX` 操作码的修订提案，并回顾了为硬件签名设备调整输出脚本描述符的研究。还包括我们的常规部分，总结了比特币服务和客户端的最新变更，新版本和候选版本，以及流行的比特币基础设施项目中值得注意的变更。此外，我们庆祝了 Optech 第 200 期定期 Newsletter 的发布。

## 新闻

- **什么时候启用 `OP_CAT` 允许递归契约？** 在 Bitcoin-Dev 邮件列表中关于重新添加 2010 年从比特币中删除的 [OP_CAT][op_cat] 操作码的讨论中，开发者 ZmnSCPxj [表示][zmnscpxj cat1] `OP_CAT` 需要与被提议的操作码 `OP_TX`（见 [Newsletter #187][news187 op_tx]）、[OP_CHECKSIGFROMSTACK][topic op_checksigfromstack]（CSFS）相结合，或与类似的功能，以便它能够实现不可避免的递归[契约][topic covenants]——契约使用比特币的共识规则来强制执行，所有收到的比特币只能花在同一个契约上。

  递归契约依赖于一种被称为*交易自省*的技术，通过这种技术，一个操作码可以分析执行该操作码的部分交易。现有的操作码 `OP_CHECKSIG`、[BIP65][] `OP_CHECKLOCKTIMEVERIFY` 和 [BIP112][] `OP_CHECKSEQUENCEVERIFY` 提供有限的自省。被提议的操作码 TX 和 CSFS 可以允许灵活地自省交易的所有部分，包括它的下一个输出（比特币的数量和接收支付的地址）和它的 *prevout*（之前的输出，以前收到的比特币的数量和目前正在使用的授权其花费的脚本）。

  如果一个操作码或其他脚本特征被用来确保 prevout 和下一个输出是相同的，那么最简单的递归契约类型就产生了。然而，prevout 并不是交易的直接组成部分——它们必须从区块链上获得——所以必须将 prevout 的副本包含在交易中，以便将 prevout 与下一个输出进行比较。更加复杂的是，在比特币交易的多个部分使用哈希函数，似乎阻止了 prevout 脚本能够直接与下一个输出脚本进行比较；相反，无论是 prevout、下一个输出，还是两者都必须从它们的组成元素中动态构建——这就是被提议的连接操作码（CAT）或类似结构对于递归契约的必要性所在。

  在很多情况下，在 [taproot][topic taproot] 之前，在交易中包含 prevout 副本的最有效方法是将其作为授权数据提供，类似于数字签名。如果 prevout 副本与签名一起提供，就可以通过对交易见证的反省来检查，这就需要一个灵活的反省设施，如 TX 或 CSFS。拟议的 CTV 和 APO 脚本修改也将允许对下一个输出进行反省，但它们对见证的反省能力要差得多，这使得它们似乎无法创建递归契约，即使与 CAT 相结合。

  Nadav Ivgi [回答][ivgi cat]说，也有可能在 prevout 本身中包含一份用于构建 prevout 的信息（通过在它是下一个输出时添加该信息）。在创建递归契约时，这仍然需要 CAT 来解决哈希的问题，但这意味着诸如 CTV 和 APO 这样专注于输出自省的功能也能与 CAT 结合起来创建递归契约。当与 taproot 的功能一起使用时，Ivgi 还认为通过下一个输出来验证 prevout 使得契约脚本的编写更加简单，他提供了两个有趣的递归契约的例子的链接。

  ZmnSCPxj [同意][zmnscpxj cat2] Ivgi 的分析，并赞同他之前对在比特币上启用递归契约的风险的担忧（见 [Newsletter #190][news190 recurse]），尽管他也在[随后的帖子][zmnscpxj cat3]中指出，"值得一提的是，递归契约可能是安全的，因为它们事实上不是[图灵完备][Turing-complete]的"。

  Russell O'Connor 还[引用][oconnor cat]了 Andrew Poelstra 的一篇[文章][poelstra cat]（在 [Newsletter #134][news134 cat] 中提及），描述了 CAT 本身是如何与已有的比特币功能相结合，强大到可以创建非递归契约，而且，从理论上讲，如果它被重新添加到比特币中，也可以自己创建递归的契约。

  所有的讨论都是关于对比特币增加功能提案，所以我们对比特币系统当前部署状态的理解没有改变。

- **更新的 OP_TX 建议：** 如 [Newsletter #187][news187 op_tx] 所述，Rusty Russell 提出了一个 `OP_TX` 操作码（基于之前的提案），允许 [tapscript][topic tapscript] 将执行交易的选定部分推送到堆栈。例如，Alice 可以接收比特币到一个包含 `OP_TX(SELECT_LOCKTIME)` 的脚本，将支出交易的锁定时间推到堆栈。使用 TX 的这种通用交易反省功能，Alice 就可以复制 `OP_CHECKLOCKTIMEVERIFY`（CLTV）的具体反省能力，例如 `OP_TX(SELECT_LOCKTIME) <height> OP_GREATERTHAN OP_VERIFY`。

  在上面的例子中，使用 CLTV 会比 TX 使用更少的 vbyte，但是 TX 的灵活性允许检查一个交易的许多其他部分，这些部分目前在比特币中无法被自省。TX 也可以检查交易之外的数据，但需要全节点才能验证。按照最初的提案，TX 被[标记][rubin op_tx recursive]为支持递归契约，目前看来是有争议的（见 [Newsletter #190][news190 recov]）。

  本周，Russell [提出][russell op_tx]了一个限制性的 TX 版本，只提供对 [OP_CHECKTEMPLATEVERIFY][topic op_checktemplateverify]（CTV）所使用的相同字段的访问，并且是 CTV 所使用的相同哈希摘要格式。[BIP119][] CTV 被特别设计为不与任何现有的脚本功能相结合来创建递归契约，因此受限制的 TX 操作码应该提供所有相同的功能，同时也不启用递归契约。

  此外，受限制的 TX 被设计成未来的软分叉可以很容易地用额外的自省功能来扩展它，包括如果有支持添加该功能的话，可以选择启用递归契约的功能。截至目前，有两位开发者建议通过扩展 TX 的更多功能来使用它：

    - *改进的金库原语：* Russell 的提案提出了一个扩展，它将提供把交易中每个输出所花费的聪的数量推到堆栈上的能力。Brandon Black [建议][black op_tx]，也要包括每个 prevout（输入）被花费的数量，将使创建[金库][topic vaults]更加容易。Black 特别建议复制几个月前 `OP_TAPLEAF_UPDATE_VERIFY`（TLUV）提案中 `OP_IN_OUT_AMOUNT` 操作码的能力（见 [Newsletter #166][news166 tluv]）。

    - *使用事务自省来防止 RBF 钉住：* Gregory Sanders [指出][sanders op_tx]，被提案的 LN 的 [Eltoo][topic eltoo] 层利用 [SIGHASH_ANYPREVOUT][topic sighash_anyprevout] 操作码，作为与 [BIP125][] 规则 #3的互动，可能容易受到交易钉住攻击（见 [Newsletter #27][news27 eltoo]）。Sanders 建议，给 TX 提供将交易重量（大小）推到堆栈的能力，将允许基于 ELTOO 的支付渠道的参与者对交易的最大规模进行限制，消除一种依赖于夸大交易规模的钉住的风险。这在概念上似乎与 [Newsletter #191][news191 pinning] 中提到的防止 [CPFP][topic cpfp] 销账的想法相似。

- **为硬件签名设备调整 miniscript 和输出脚本描述符：** Salvatore Ingala 在 Bitcoin-Dev 邮件列表中[发布][ingala desc]了他为硬件签名设备实现[输出脚本描述符][topic descriptors]和 [miniscript][topic miniscript] 的工作。他指出，签名设备特别注重策略验证——用户需要了解如果他们批准一项交易会发生什么，但他们不应该被提示超过必要的信息以确保他们的资金安全。由于许多签名设备的屏幕尺寸较小，而且在那里验证信息有困难，所以尽可能紧凑地显示信息也很重要。Ingala 建议对描述符进行几项改进以解决这些问题：

    - *策略注册：* 在签名设备的设置过程中，用户应该使用设备来验证他们的偏好策略。对于有持久性存储的设备，注册的策略应该被保存到设备中。对于没有存储的设备，设备应该返回一个加密安全的注册证明，该证明可以在每次启动设备时与策略一起快速重新加载。该提案没有描述如何在设备上注册策略的细节，但它确实参考了 [BIP129][] 安全多签设置（见 [Newsletter #136][news136 sms]）。

    - *密钥占位符：* Ingala 建议允许策略定义一个简短的占位符，而不是在描述符中包括重复的 [BIP32][] 扩展密钥，当策略被解释时，将用 BIP32 信息替换。这既可以大大减少策略的字节大小，也可以使它们更容易被人阅读。Ingala 还为描述符中常用的字符串建议了一些缩写。

    - *降低表达能力：* 只支持描述符的一个子集，不过如果有需求的话，以后还可以增加其他的功能。这样可以简化实现。

  截至本文撰写时，该提案在邮件列表中得到了一些讨论。

## 服务和客户端软件的变更

*在这个月的栏目中，我们突出比特币钱包和服务的有趣升级*。

- **MyCitadel Wallet发布：** [MyCitadel Wallet][] 是一个桌面比特币、闪电网络和 [RGB][] 钱包应用程序，支持多签、[PSBT][topic psbt]、segwit、taproot、[时间锁][topic timelocks]、硬件签名设备、[描述符][topic descriptors]和其他功能。

- **Tauros 交易所支持闪电网络：** 墨西哥交易所 [Tauros][] 宣布支持闪电网络存款和提款。

- **闪电网络复用器公布：** 支持 LND 的闪电网络复用器（[lnmux][]）软件，通过允许付款的故障场景，提高了闪电网络支付的可靠性。更多细节可在 [Bottlepay 的博客][lnmux blog]文章中找到。

- **Coldcard 增加了 taproot 的发送：** [最新的 Coldcard 固件][coldcard upgrade]（Mk4 5.0.3，Mk3 4.1.5）支持发送至 [bech32m][topic bech32] 地址。

## 新版本和候选版本

*流行比特币基础设施项目的新版本和候选版本。请考虑升级到最新版本或帮助测试候选版本。*

- [LND 0.15.0-beta.rc1][] 是这个流行的闪电网络节点的下一个主要版本的候选版本。

## 代码和文档的重大变更
*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Bitcoin Core #22235][] 增加了一个脚本，它将直接从定义配置选项的源代码中生成一个 `bitcoin.conf` 文件的例子。这一变更使得未来的 Bitcoin Core 版本可以有一个预装的 bitcoin.conf 文件，而不需要创造第二个关于配置选项状态的可信来源。

- [LND #6450][] 增加了对签署花费 [taproot][topic taproot] 输出的 [PSBT][topic psbt] 的支持。

- [LND #6345][] 允许 LND 通过反复轮询 Bitcoin Core 而不是通过 ZMQ 接收推送更新来获得关于新交易和块的更新。

- [BIPs #1309][] 更新了 [BIP119][] 的 [OP_CHECKTEMPLATEVERIFY][topic op_checktemplateverify] 的实现示例。之前的例子包含了用 C++ 编写的为 Bitcoin Core 实现操作码的代码。新的例子是用 pseudo-python 写的。还增加了关于如何避免在不成熟的实现中可能出现的拒绝服务攻击的额外信息，解决了之前在 Bitcoin-Dev 邮件列表中讨论的一个问题（见 [Newsletter #183][news183 dos]）。

## 庆祝 Optech Newsletter #200

每五十份 Newsletter，我们都会花点时间来强调 Optech 的一些主要成就。我们这样做，一方面是为了帮助读者了解我们所提供的一切，另一方面也是为了吸引捐款，使我们能够继续产出有用的内容。但是，我们这样做有一点也是为了吹嘘。

我们不是出于个人的虚荣心而吹嘘，而是因为 Optech 的每个贡献者本身就是其他贡献者的忠粉。与了不起的人在一个团队一起工作是一种美妙的体验，而这些特别的通讯部分通常是我们可以把我们对彼此毫无保留的钦佩之情付诸于文字的地方。

这不是我们今年要做的事。相反，我们想给你，读者，你自己的机会来分享你对 Optech 的贡献者的一些钦佩。在没有征求意见的情况下，我们发现我们的读者在 Twitter 上说了几十条甚至几百条可爱的话。太多令人难以置信的评论让我们无法很好地展示出来，所以我们只展示了在过去四年的Top 50。

你可以在下面看到这些评论，但首先我们要感谢我们过去一年的主要贡献者——Adam Jonas、Carl Dong、David A.
Harding、Gloria Zhao、John Newbery、Mark Erhardt、Mike Schmidt 和
 Shigeyuki Azuchi，同时也感谢我们的许多[支持者][supporters]，包括我们的[创始赞助商][founding sponsors] Wences Casares、John Pfeffer 和 Alex Morcos。


> 第 100 期的 @bitcoinoptech 刚刚出版。[...] 它是比特币领域最好的技术通讯之一。
<p align="right"><a herf="https://twitter.com/Bitcoin/status/1268171489011998720">@Bitcoin</a></p>

> 关注 @bitcoinoptech 并订阅免费 Newsletter，获取真实信号。 我并不完全理解内容，但无论市场如何，它都让我看涨。
<p align="right"><a herf="https://twitter.com/ChartsBtc/status/1466071392449818629">@ChartsBTC</a></p>

> Bitcoin Optech @bitcoinoptech 对目前比特币领域正在进行的工作有一个很好的概述。它真正显示了在比特币领域实际发生了多少事情，并打破了比特币是 "陈旧的老技术 "的概念。https://bitcoinops.org/en/topic-dates/
<p align="right"><a herf="https://twitter.com/softsimon_/status/1296739713286459398">@softsimon</a></p>
<p align="right">Mempool.Space 创造者</p>

> 我也强烈推荐 @bitcoinoptech :)
<p align="right"><a herf="https://twitter.com/Leishman/status/1082288630297751552">Alexander Leishman</a></p>
<p align="right">创始人, River Financial</p>

> 随着消费市场的发展和基础设施的进步，我们正在通过 @bitcoinoptech 推动企业采用(a)协议升级和(b)技术，使服务提供商更有效地运作，更好地服务于不断增长的市场。
<p align="right"><a herf="https://twitter.com/AlyseKilleen/status/1138921268185407489">Alyse Killeen</a></p>
<p align="right">创始合伙人, Stillmark</p>

> Bitcoin Optech（@bitcoinoptech）通讯持续提供有趣和详细的内容。我刚刚读了[最新的]一篇，很不错。
<p align="right"><a herf="https://twitter.com/aantonop/status/1133785460650721280">Andreas M. Antonopoulos</a></p>
<p align="right">《精通比特币》的作者和《精通闪电网络》的合著者</p>

> 像往常一样，感谢这份 Newsletter!
<p align="right"><a herf="https://twitter.com/realtbast/status/1468553191243591683">Bastien Teinturier</a></p>
<p align="right">ACINQ的工程副总裁</p>

> @bitcoinoptech 向技术集成商的听众很好地解释了技术，我想看他们的研讨会
<p align="right"><a herf="https://twitter.com/Empact/status/1403702689737871362">Ben Woosley</a></p>
<p align="right">比特币核心开发者和 Unchained Capital 高级开发者</p>

> Bitcoin Optech（@bitcoinoptech）成立于2018年，旨在为开源开发和公司的世界搭建桥梁。自成立以来，他们已经走过了很长一段路。
<p align="right"><a herf="https://twitter.com/BitcoinMagazine/status/1229834232861724672">比特币杂志</a></p>

> 优秀的 @bitcoinoptech Newsletter 对 2018 年期间比特币和闪电网络的所有主要发展进行了令人难以置信的综述。一个月又一个月的工作，成就了一个不错的周末阅读清单。
<p align="right"><a herf="https://twitter.com/BuckPerley/status/1078791035864707079">Buck Perley</a></p>
<p align="right">Unchained Capital 的工程师</p>

> 另一个伟大的通讯。谢谢 @bitcoinoptech!
<p align="right"><a herf="https://twitter.com/CardCoinsCo/status/1468783314404261894">CardCoins</a></p>

> 我认为像 @bitcoinoptech 这样的努力正在改善沟通，但我们需要更多的努力!
<p align="right"><a herf="https://twitter.com/carl_dong/status/1057721562449633280">Carl Dong</a></p>
<p align="right">Chaincode 实验室比特币核心开发者</p>

> 通过阅读 @bitcoinoptech 的 2021 年回顾，让你的新年决心先一步跟上比特币发展的精彩世界 🌟
<p align="right"><a herf="https://twitter.com/actuallyCarlaKC/status/1473660532959985669">Carla Kirk-Cohen</a></p>
<p align="right">独立闪电网络开发者，Brink、₿trust 和 Qala 董事会成员</p>

> 在 Taproot 上有点落后？还有时间，幸运的是 @bitcoinoptech 有一个很棒的研讨会，可以让你赶上进度：https://github.com/bitcoinops/taproot-workshop/  <br> 我强烈推荐。它涵盖了你所需要的所有模块。去做吧💪#比特币#taproot
<p align="right"><a herf="https://twitter.com/ElleMouton/status/1418108253096095745">Elle Mouton</a></p>
<p align="right">工程师，闪电实验室</p>

> 我们[在比特币杂志]尽力提炼硬核的东西，但如果你在寻找原始信息，@bitcoinoptech 是很难被打败的。 <br> 到目前为止，这是我最喜欢的非比特币杂志的内容，也是为数不多的能在我的收件箱里待到我能花适当时间看的通讯之一。
<p align="right"><a herf="https://twitter.com/flip_abagnale/status/1298094514159181824">Flip Abagnale</a></p>

> 说实话，@bitcoinoptech 很了不起
<p align="right"><a herf="https://twitter.com/gckaloudis/status/1476196134745948169">George Kaloudis</a></p>
<p align="right">CoinDesk 研究分析师</p>

> @bitcoinoptech 是一个伟大的资源
<p align="right"><a herf="https://twitter.com/theinstagibbs/status/1511413286167851014">Gregory Sanders</a></p>
<p align="right">比特币开发者</p>

> 我想对 @bitcoinoptech 团队的通讯表示感谢。<br> 它为我节省大量浏览聊天和电子邮件的时间，只需跟踪主要的发展，并轻松地深入挖掘任何特别相关或有趣的东西。✊
<p align="right"><a herf="https://twitter.com/TheGuySwann/status/1302029639112691719">Guy Swann</a></p>
<p align="right">Bitcoin Audible的主持人</p>

> P.S. 我想感谢那些每天都在运行和改进网络的开发者、教育者和矿工。<br> 在 @lightning、@ChaincodeLabs、@CryptoGarageInc、@Blockstream、@bitcoinoptech，以及许多许多其他贡献者都在做着惊人的事情。<br>我真的感谢你们所有人。
<p align="right"><a herf="https://twitter.com/haegwankim/status/1314503620910436353">Haegwan Kim</a></p>

> 这一年的伟大总结👍 @bitcoinoptech
<p align="right"><a herf="https://twitter.com/hfangca/status/1211002573282394112">Hong Fang</a></p>
<p align="right">OKCoin CEO</p>

> 无知的人:"比特币停止了创新；发展停滞了！"<br> 我："@bitcoinoptech 的年度发展回顾甚至不能在我的电子邮件客户端中完全显示，因为它太长了。"
<p align="right"><a herf="https://twitter.com/lopp/status/1342123233210998784">Jameson Lopp</a></p>
<p align="right">Casa 联合创始人和 CTO</p>

> 为数不多的我持续阅读每一期的通讯之一。 谢谢 @bitcoinoptech
<p align="right"><a herf="https://twitter.com/lightcoin/status/1218260416171847680">John Light</a></p>
<p align="right">人权基金会 ZK-Rollup Research Fellow</p>

> 对致力于建设#比特币的才华横溢、充满激情的开发者表示无比的钦佩。我很荣幸能够支持 @meshcollider、Antoine Riard、@bitcoinoptech、@mitDCI， 开发培训...还有更多即将到来的活动! 请和我一起支持 #btc 的发展!
<p align="right"><a herf="https://twitter.com/jlppfeffer/status/1324761477694267393">John Pfeffer</a></p>
<p align="right">企业家和投资者</p>

> 向 Bitcoin Optech 喊话，他们记录了比特币的技术发展。没有炒作，没有戏剧性，只有关于使用和部署比特币的进展的了不起的信息。@bitcoinoptech
<p align="right"><a herf="https://twitter.com/jmcorgan/status/1217480526006644736">Johnathan Corgan</a></p>

> @bitcoinoptech 的话题是一个很好的开始。
<p align="right"><a herf="https://twitter.com/jonatack/status/1379839115676561408">Jon Atack</a></p>
<p align="right">比特币开发者</p>

> @bitcoinoptech 写了一份非常好的年终回顾通讯
<p align="right"><a herf="https://twitter.com/_JustinMoon_/status/1219347584789176325">Justin Moon</a></p>

> [...] @bitcoinoptech 写的这篇文章解释了为什么 nonce 重用在多签设置中是不好的，也解释了避免它的挑战。它真的帮助我更好地理解这个问题。谢谢!
<p align="right"><a herf="https://twitter.com/kallerosenbaum/status/1429793328623755265">Kalle Rosenbaum</a></p>
<p align="right">Grokking Bitcoin 的作者</p>

> 有很多关于加密货币的新闻简报可以让你了解最新情况。<br>我一直在关注的是 @bitcoinoptech。<br>如果你持有一些比特币，并希望看到一些真正的工作在使协议更好，强烈推荐。
<p align="right"><a herf="https://twitter.com/Kristian_Kho/status/1452201573799632896">Kristian Kho</a></p>

> @bitcoinoptech 的重要文章，总结了值得注意的编码发展并进行了编目。
<p align="right"><a herf="https://twitter.com/LeahWald/status/1215464602214883329">Leah Wald</a></p>
<p align="right">Valkyrie 投资公司 CEO</p>

> 你难道不喜欢突然发现惊人的教育资源吗？<br>我本周末的首要项目是我最近才发现的 @bitcoinoptech 关于 taproot 的研讨会。
<p align="right"><a herf="https://twitter.com/LucasNuzzi/status/1228417230989340673">Lucas Nuzzi</a></p>
<p align="right">CoinMetrics.io 研发主管</p>

> 哇 - @比特币optech有25位以上的贡献者!<br>最高质量的#比特币新闻——由高水准的自由软件贡献者进行同行评审!<br>感谢所有的支持者。🔥🚀
<p align="right"><a herf="https://twitter.com/HillebrandMax/status/1299315892543725568">Max Hillebrand</a></p>
<p align="right">自由软件企业家</p>

> 我想现在每个人都知道 @bitcoinoptech 的优秀通讯了。但你是否知道它有一个主题页面，按主题名称、事件和它所影响的比特币系统的一部分进行索引？https://bitcoinops.org/en/topic-categories/
<p align="right"><a herf="https://twitter.com/michaelfolkson/status/1254733781233209344">Michael Folkson</a></p>

> 感谢所有帮助我们走到这一步的人：共同作者 @ajtowns、 @n1ckler 和 @real_or_random， 所有为文档做出贡献的人，@bitcoinoptech 研讨会和结构化审查的参与者，Greg Maxwell 的原始想法，以及许多其他人。
<p align="right"><a herf="https://twitter.com/pwuille/status/1220507061026344960">Pieter Wuille</a></p>
<p align="right">Chaincode 实验室比特币开发者</p>

> [......]试图总结各处正在发生的事情几乎是一项全职工作。这就是 @bitcoinoptech 的人所做的，所以只要关注他们的通讯就会有足够的了解。
<p align="right"><a herf="https://twitter.com/RajarshiMaitra/status/1398931136714182656">Rajarshi Maitra</a></p>

> 我又一次喜欢上了 @bitcoinoptech 的新闻通讯[...]这是为开发者和教育者提供的最丰富和最好的资源和概述。
<p align="right"><a herf="https://twitter.com/renepickhardt/status/1120702153985875973">Rene Pickhardt</a></p>
<p align="right">比特币和闪电网络开发者
</p>

> 我推荐 @bitcoinoptech 的系列视频来了解 Taproot，特别是 @digi_james 的互动式 colab 笔记本对了解如何使用 tapscript 很有帮助。
<p align="right"><a herf="https://twitter.com/remyers_/status/1455955655651729415">Richard Myers</a></p>

> @bitcoinoptech 的通讯每周都会介绍#比特币的前沿发展和更新。
<p align="right"><a herf="https://twitter.com/river/status/1268624838161305601">River Financial</a></p>

> @bitcoinoptech 是一个非常有价值的社区项目，参与和学习最佳实践的广大生态系统的比例越大越好。
<p align="right"><a herf="https://twitter.com/RobertSpigler/status/1058780917685239810"> Robert Spigler</a></p>
<p align="right">OpSec 顾问</p>

> 关注 @bitcoinoptech 并注册订阅他们的通讯，5-9 个信号。<br>很少有比特币技术信息的来源有这么好的总结和解释。<br>现在就做吧，这样你就不会忘记了😉
<p align="right"><a herf="https://twitter.com/nvk/status/1455681554047545352">Rodolfo Novak</a></p>
<p align="right">Coinkite CEO 和联合创始人</p>

> 像往常一样，@bitcoinoptech 提供了一个专业的总结[...] 。
<p align="right"><a herf="https://twitter.com/SomsenRuben/status/1511679367688200193">Ruben Somsen</a></p>
<p align="right">Bitcoin Sorcerer 和 The Unhashed Podcast 联合主持人</p>

> 尝试少读微博，多读 @bitcoinoptech
<p align="right"><a herf="https://twitter.com/SahilC0/status/1501703542393946115">Sahil Chaturvedi</a></p>
<p align="right">Unchained Capital 产品设计师</p>

> 在最新一期的 @bitcoinoptech 中，有一个很好的例子说明为什么博弈论如此重要。
<p align="right"><a herf="https://twitter.com/SDWouters/status/1082930342053462016">Sam Wouters</a></p>
<p align="right">比特币教育者</p>

> 感谢 @bitcoinoptech 团队在兼容性矩阵方面所做的全面工作。
<p align="right"><a herf="https://twitter.com/SamouraiWallet/status/1163877643932065793">Samourai Wallet</a></p>

> 写得不错，@bitcoinoptech。👍
<p align="right"><a herf="https://twitter.com/Excellion/status/1211075163732631553">Samson Mow</a></p>
<p align="right">Jan3 CEO</p>

> Taproot 系列是一个非常宝贵的资源，不仅可以了解大局，还可以了解技术细节。感谢所有为此做出贡献的人!
<p align="right"><a herf="https://twitter.com/satsie/status/1458436050091745286">Stacie Waleyko</a></p>
<p align="right">Casa 的工程师</p>

> 对于学习比特币技术的人来说，这是一个伟大的、也许不太为人所知的资源：@bitcoinoptech 主题页面: https://bitcoinops.org/en/topics/
<p align="right"><a herf="https://twitter.com/stephanlivera/status/1342674949392093186">Stephan Livera</a></p>
<p align="right">Swan Bitcoin 董事总经理和播客</p>

> @bitcoinoptech 在 2 月份对 LDK 的方法进行了很好的报道 https://bitcoinops.org/en/newsletters/2022/02/23/#ldk-1199
<p align="right"><a herf="https://twitter.com/moneyball/status/1522260482538627077">Steve Lee</a></p>
<p align="right">Spiral 负责人</p>

> 感谢 @bitcoinoptech，这是比特币领域最好的组织之一，专注于帮助我们其他人使用比特币，成为更好的比特币人!
<p align="right"><a herf="https://twitter.com/TYonClubhouse/status/1473793982408716289">Terrence Yang</a></p>

> 我们整个团队都很荣幸能被列入最受尊敬和最先进的技术#比特币通讯，这就是 @bitcoinoptech
<p align="right"><a herf="https://twitter.com/veriphibtc/status/1293602911910600707">Veriphi</a></p>

> 真的，@bitcoinoptech 是每个比特币人的必读材料。
<p align="right"><a herf="https://twitter.com/zackvoell/status/1064937176922841091">Zack Voell</a></p>

<p align="right"><a herf=""></a></p>

<p align="right"><a herf=""></a></p>
<p align="right"></p>

[topic op_checksigfromstack]: https://bitcoinops.org/en/topics/op_checksigfromstack/
[topic covenants]: https://bitcoinops.org/en/topics/covenants/
[topic taproot]: https://bitcoinops.org/en/topics/taproot/
[topic tapscript]: https://bitcoinops.org/en/topics/tapscript/
[topic op_checktemplateverify]: https://bitcoinops.org/en/topics/op_checktemplateverify/
[topic vaults]: https://bitcoinops.org/en/topics/vaults/
[topic eltoo]: https://bitcoinops.org/en/topics/eltoo/
[topic sighash_anyprevout]: https://bitcoinops.org/en/topics/sighash_anyprevout/
[topic cpfp]: https://bitcoinops.org/en/topics/cpfp/
[topic descriptors]: https://bitcoinops.org/en/topics/output-script-descriptors/
[topic miniscript]: https://bitcoinops.org/en/topics/miniscript/
[topic psbt]: https://bitcoinops.org/en/topics/psbt/
[topic timelocks]: https://bitcoinops.org/en/topics/timelocks/
[topic bech32]: https://bitcoinops.org/en/topics/bech32/


[lnd 0.15.0-beta.rc1]: https://github.com/lightningnetwork/lnd/releases/tag/v0.15.0-beta.rc1
[news27 eltoo]: https://bitcoinops.org/en/newsletters/2018/12/28/#april
[news166 tluv]: https://bitcoinops.org/en/newsletters/2021/09/15/#amount-introspection
[news190 recov]: https://bitcoinops.org/en/newsletters/2022/03/09/#limiting-script-language-expressiveness
[rubin op_tx recursive]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019872.html
[minsc 1/3]: https://min.sc/#c=%24alice%20%3D%20A%3B%0A%24bob%20%3D%20B%3B%0A%24carol%20%3D%20C%3B%0A%0Apk%28%24alice%29%20%7C%7C%20pk%28%24bob%29%20%7C%7C%20pk%28%24carol%29
[news136 sms]: https://bitcoinops.org/en/newsletters/2021/02/17/#securely-setting-up-multisig-wallets
[news183 dos]: https://bitcoinops.org/en/newsletters/2022/01/19/#mailing-list-discussion
[zmnscpxj cat1]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020434.html
[op_cat]: https://bitcoinops.org/en/topics/op_checksigfromstack/#relationship-to-op_cat
[news187 op_tx]: https://bitcoinops.org/en/newsletters/2022/02/16/#simplified-alternative-to-op-txhash
[ivgi cat]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020442.html
[zmnscpxj cat2]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020441.html
[oconnor cat]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020467.html
[poelstra cat]: https://www.wpsoftware.net/andrew/blog/cat-and-schnorr-tricks-i.html
[news190 recurse]: https://bitcoinops.org/en/newsletters/2022/03/09/#introduction-of-turing-completeness
[zmnscpxj cat3]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020462.html
[news134 cat]: https://bitcoinops.org/en/newsletters/2021/02/03/#replicating-op-checksigfromstack-with-bip340-and-op-cat
[turing-complete]: https://en.wikipedia.org/wiki/Turing_completeness
[russell op_tx]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020450.html
[black op_tx]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020454.html
[sanders op_tx]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020458.html
[ingala desc]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020423.html
[supporters]: /#supporters
[founding sponsors]: https://bitcoinops.org/about/#founding-sponsors
[news191 pinning]: https://bitcoinops.org/en/newsletters/2022/03/16/#ideas-for-improving-rbf-policy
[MyCitadel Wallet]: https://github.com/mycitadel/mycitadel-desktop
[RGB]: https://www.rgbfaq.com/what-is-rgb
[Tauros]: https://tauros.io/
[lnmux]: https://github.com/bottlepay/lnmux
[lnmux blog]: https://bottlepay.com/blog/multiplexer/
[coldcard upgrade]: https://coldcard.com/docs/upgrade
[BIP65]: https://github.com/bitcoin/bips/blob/master/bip-0065.mediawiki
[BIP112]: https://github.com/bitcoin/bips/blob/master/bip-0112.mediawiki
[BIP119]: https://github.com/bitcoin/bips/blob/master/bip-0119.mediawiki
[BIP125]: https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki
[BIP129]: https://github.com/bitcoin/bips/blob/master/bip-0129.mediawiki
[BIP32]: https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki
[Bitcoin Core #22235]: https://github.com/bitcoin/bitcoin/issues/22235
[LND #6450]: https://github.com/lightningnetwork/lnd/issues/6450
[LND #6345]: https://github.com/lightningnetwork/lnd/issues/6345
[BIPs #1309]: https://github.com/bitcoin/bips/pull/1309

[bitcoin core repo]: https://github.com/bitcoin/bitcoin
[c-lightning repo]: https://github.com/ElementsProject/lightning
[eclair repo]: https://github.com/ACINQ/eclair
[lnd repo]: https://github.com/lightningnetwork/lnd/
[rust-lightning repo]: https://github.com/rust-bitcoin/rust-lightning
[libsecp256k1 repo]: https://github.com/bitcoin-core/secp256k1
[hwi repo]: https://github.com/bitcoin-core/HWI
[rust bitcoin repo]: https://github.com/rust-bitcoin/rust-bitcoin
[btcpay server repo]: https://github.com/btcpayserver/btcpayserver/
[bdk repo]: https://github.com/bitcoindevkit/bdk
[bips repo]: https://github.com/bitcoin/bips/
[bolts repo]: https://github.com/lightning/bolts