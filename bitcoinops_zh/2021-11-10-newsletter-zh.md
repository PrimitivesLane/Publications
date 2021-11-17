---
title: 'Bitcoin Optech Newsletter #174'
permalink: /zh/newsletters/2021/11/10/
name: 2021-11-10-newsletter-zh 
slug: 2021-11-10-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---

本周的周报总结了一篇讨论 “谨慎日志合约（Discreet Log Contract，DLC）” 如何整合到闪电网络的文章、链接了一份最新的闪电网络开发者大会的详细总结，还介绍了一些为致密区块过滤器（compact block filter）执行额外验证的想法。也包含了我们的常规环节：Bitcoin Core PR 审核俱乐部会议的总结、“为 Taproot 升级作准备” 的最后一章、软件新版本和后续版本的公告，以及常见基础设施软件重大变更的清单。

## 新闻

- <a id="dlcs-over-ln" href="#dlcs-over-ln)">●</a> **闪电网络上的谨慎日志合约**：Thibaut Le Guilly 在 DLC-Dev 邮件组中发表了一篇[帖子][thread]，讨论如何在闪电网络中集成[谨慎日志合约][Discreet Log Contracts]。最初的帖子简述了多种在两个直接相连的闪电节点的交易中包含 DLC 的可能构造。这篇帖子也描述了创建在闪电网络中路由的 DLC 时会遇到的一些挑战。
- <a id="ln-summit-2021-notes" href="#ln-summit-2021-notes)">●</a> **LN 2021 峰会记录**：Olaoluwa Osuntokun 为最近在苏黎世（Zurich）举办的虚拟和真人闪电网络开发者会议[发表][posted]了一份详尽的总结。总结的内容包括：在闪电网络中使用 [taproot][taproot]（包括 [“点时间锁（PTLC）”][PTLCs] 、为[多签名场景][multisignatures]适配 [MuSig2][MuSig2] 以及 [eltoo][eltoo]）；把规范讨论的环境从在线聊天室（IRC）切换到视频讨论；改变当前的 BOLT 规范模式；洋葱消息和 [“收款码”][offers] 功能；无滞碍的支付（见 [本周报第 53 期][Newsletter #53]）；[通道阻塞攻击][channel jamming attacks]以及多种缓解措施，以及 [“蹦床支付”（trampoline routing）][trampoline routing]。
- <a id="additional-compact-block-filter-verification" href="#additional-compact-block-filter-verification)">●</a> **额外的致密区块过滤器验证**：[Neutrino][Neutrino] 软件轻量版用一种启发式算法来检测[致密区块过滤器][compact block filter]是否包含了错误的数据，但这种算法对测试网上生成的、包含了一笔 taproot 交易的区块的正确生成的过滤器，居然报错了。这个问题在 Neutrino 的源代码中已经打上了[补丁][patched]，而且致密区块过滤器的其它实现不受影响。但 Olaoluwa Osuntokun 在 [Bitcoin-Dev][Bitcoin-Dev] 和 [LND-Dev][LND-Dev] 邮件组中开了一个帖子来讨论这个问题 —— 以及致密区块过滤器未来可能的提升，比如：
  - <a id="new-filters" href="#new-filters)">●</a>  *新的过滤器*：创建额外的、可选的过滤器类型，让轻量客户端可以搜索其它类型的数据。
  - <a id="new-p2p-protocol-message" href="#new-p2p-protocol-message)">●</a> *新的 P2P 协议消息*：加入一种新的 P2P 协议消息，用于检索区块撤销数据（block undo data）。区块撤销数据包含一个区块所花费的每个输入的前序输出（以及相关消息），可以结合区块来完全验证一个过滤器是否从撤销数据中生成。撤销数据自身可以在对等节点的差异中得到[验证][verified]。
  - <a id="multi-block-filters" href="#multi-block-filters)">●</a> *多区块过滤器*：可以进一步减少轻量级客户端需要下载的数据。
  - <a id="committed-block-filters" href="#committed-block-filters)">●</a> *得到承诺的区块过滤器*：要求矿工在区块中承诺过滤器，减少轻量客户端为了观察不同的对等节点所提供的过滤器的差异而需要下载的数据量。

## Bitcoin Core 软件 PR 审核俱乐部

*在这个每月一次的栏目中，我们总结最近一次的 [Bitcoin Core PR 审核俱乐部会议][Bitcoin Core PR Review Club]，集中展示一些重要的问题和答案。点击具体的问题可看到会议答案的总结。*

“[Add `ChainstateManager::ProcessTransaction`][Add `ChainstateManager::ProcessTransaction`]” 是 John Newbery 提的一个 PR，加入一种新的 `ChainstateManager::ProcessTransaction()` 接口函数来负责处理进入交易池的候选交易、执行交易池一致性检查。审核俱乐部讨论了当前为交易池加入交易的接口。

<details><summary>什么是 <code>cs_main</code>？为什么要叫 <code>cs_main</code>？
</summary>
<code>cs_main</code> 是一个 mutex（保证多线程不会发生竞入问题的部件），初衷是同步化对验证状态的多线程访问。实际上，它也保护非验证的数据，包括用在 P2P 逻辑中的数据。许多贡献者希望尽可能减少对 <code>cs_main</code> 的使用。这个变量的命名是在把验证功能模块放到一个 main.cpp 文件中的时候决定的。而前缀 <code>cs</code> 是 “关键环节（critical section）” 的缩写。<a href="https://bitcoincore.reviews/23173#l-45">➚</a>
</details>

<details><summary>现在哪些组件会调用 <code>AcceptToMemoryPool</code>？哪些 ATMP 调用是从外部客户端代码中来的，哪些是在内部验证中发起的呢？
</summary>
除了测试中的调用，现在有四种调用情形：
<ol>
    <li>在节点启动时，它会从 mempool.dat 文件中<a href="https://github.com/bitcoin/bitcoin/blob/23ae7931be50376fa6bda692c641a3d2538556ee/src/validation.cpp#L4489-L4490">加载</a>交易，并调用 ATMP 来重验证这些交易并重新存储交易池内容。这是一种内部验证调用。</li>
    <li>从点对点网络中的对等节点处收到的交易，会通过 ATMP 来<a href="https://github.com/bitcoin/bitcoin/blob/23ae7931be50376fa6bda692c641a3d2538556ee/src/net_processing.cpp#L3262">验证并发送到交易池中。这种调用最初来自一个验证过程 “以外” 的组件。</a></li>
    <li>在区块链重组期间，任何当前处在无后续的区块中、但未被新的链头部包含的交易，都会被使用 ATMP <a href="https://github.com/bitcoin/bitcoin/blob/23ae7931be50376fa6bda692c641a3d2538556ee/src/validation.cpp#L352-L354">重新发送</a>到交易池中。这是一种内部验证调用。</li>
    <li>客户端比如 RPC（例如 <code>sendrawtransaction</code>）和钱包（例如 <code>sendtoaddress</code>）使用 <code><a href="https://github.com/bitcoin/bitcoin/blob/23ae7931be50376fa6bda692c641a3d2538556ee/src/node/transaction.cpp#L73-L83">`BroadcastTransaction()`</a></code>）把它们的交易发送给节点时，也会调用 ATMP。<code>testmempoolaccept</code> RPC 方法也会调用 ATMP（且 <code>test_accept</code> 设为 <code>true</code> ）。这些是 “验证外” 组件调用的例子。<a href="https://bitcoincore.reviews/23173#l-80">➚</a></li>
    </ol>
</details>

<details><summary> <code>CTxMemPool::check()</code> 是做什么的？哪个模块负责调用这个函数？
</summary>
<code>CTxMemPool::check()</code> 会检查所有交易的输入是否对应于可用的 UTXO，并对整个交易池执行内部的一致性检查。举个例子，它会统计每个交易池条目的祖先和后代，来保证缓存的 <code>ancestorsize</code>、<code>ancestorcount</code>、<code>descendantsize</code> 和 <code>descendantcount</code> 值都是准确的。当前，ATMP 的调用方负责在事后调用 <code>check()</code>。不过，参与者们讨论说，应该由 <code>ChainstateManager</code>负责执行自己的内部一致性检查。<a href="https://bitcoincore.reviews/23173#l-122">➚</a>
</details>

<details><summary> <code>bypass_limits</code>参数有什么用？在什么情形下调用 ATMP 时需要把这个参数设为 <code>true</code>？
</summary>
在 <code>bypass_limits</code> 为 true 时，节点不会适用交易池的体积上限和手续费率的下限。举个例子，如果你的节点的交易池已经满了，而你的动态的交易池手续费下限是 3 sat/vB，如果把这个参数设为 true，则某一笔单体费率为 1 sat/vB 的交易也会被接受。在<a id="eorg" href="https://github.com/bitcoin/bitcoin/blob/f87e07c6fe321f0fb97703c82c0e4122f800589f/src/validation.cpp#L353">链重组</a>期间，ATMP 会与 <code>bypass_limits</code> 一起调用；这些交易可能会有较低的单体费率，但其后代的费率会较高。这样重新添加到交易池的交易联合体的体积不能超过 <code>MAX_DISCONNECTED_TX_POOL_SIZE</code> 参数的值，默认为 20 MB。<a href="https://bitcoincore.reviews/23173#l-132">➚</a>
</details>
## 为 taproot 升级作准备 #21：感谢你们！

*我们这个每周一次的[连载][series]，记录了开发者和服务提供商如何为将在比特币区块高度 709632 处激活的 taproot 升级作准备；现在已经走到了最后一篇*。

Taproot 升级将在区块高度 709632 处激活，预计是这一卷周报出版的几天后。作为本连载的最后一篇，我们列出了一些名字，表达我们对所有帮助开发和激活 taproot —— 以及接下来要开始强制执行 taproot 规则 —— 的人的谢忱。许多人的名字没有列在其中，但同样值得我们感谢 —— 为我们所有的疏忽表示歉意。

**Bitcoin-dev 邮件组讨论**

taproot 背后的关键想法[源于][originated] 2019 年 1 月 22 日早晨几个密码学家之间的对话。同一天的晚些时候，这个想法被[发布][posted]到 Bitcoin-dev 邮件组中。下面提到的每一位，都参与到了这个以 “taproot” 为名的帖子中。

*Adam Back、Andrea Barontini、Andreas Schildbach、Andrew Chow、Andrew Poelstra、Anthony Towns、Antoine Riard、Ariel Lorenzo-Luaces、Aymeric Vitte、Ben Carman、Ben Woosley、Billy Tetrud、BitcoinMechanic、Bryan Bishop、Carlo Spiller、Chris Belcher、Christopher Allen、Clark Moody、Claus Ehrenberg、Craig Raw、Damian Mee、Daniel Edgecumbe、David A. Harding、DA Williamson、Elichai Turkel、Emil Pfeffer、Eoin McQuinn、Eric Voskuil、Erik Aronesty、Felipe Micaroni Lalli、Giacomo Caironi、Gregory Maxwell、Greg Sanders、Jay Berg、Jeremy Rubin、John Newbery、Johnson Lau、Jonas Nick、Karl-Johan Alm、Keagan McClelland、Lloyd Fournier、Luke Dashjr、Luke Kenneth Casson Leighton、Mark Friedenbach、Martin Schwarz、Matt Corallo、Matt Hill、Michael Folkson、Natanael、Oleg Andreev、Pavol Rusnak、Pieter Wuille、Prayank、R E Broadley、Riccardo Casatta、Robert Spigler、Ruben Somsen、Russell O’Connor、Rusty Russell、Ryan Grant、Salvatore Ingala、Samson Mow、Sjors Provoost、Steve Lee、Tamas Blummer、Thomas Hartman、Tim Ruffing、Vincent Truong、vjudeu、yancy、yanmaani— 以及 ZmnSCPxj*。

不过，taproot 升级中包含的许多点子，例如 [schnorr 签名][schnorr signatures] 和 [MAST][MAST]，比 taproot 早几年甚至几十年提出。所以列出对这些想法的贡献者就超出了我们的能力，但无论如何，我们应该感谢他们。

**Taproot BIP 审核**

从 2019 年 11 月开始，许多的用户和开发者参与到了对 taproot 上级的[有组织的审核][organized review]和开发过程中。

*achow101、afk11、aj、alec、amiti、_andrewtoth、andytoshi、ariard、arik、b10c、belcher、bjarnem、BlueMatt、bsm1175321、cdecker、chm-diederichs、Chris_Stewart_5、cle1408、CubicEarth、Day、ddustin、devrandom、digi_james、dr-orlovsky、dustinwinski、elichai2、evoskuil、fanquake、felixweis、fjahr、ghost43、ghosthell、gmaxwell、harding、hebasto、instagibbs、jeremyrubin、jnewbery、jonatack、justinmoon、kabaum、kanzure、luke-jr、maaku、mattleon、michaelfolkson、midnight、mol、Moller40、moneyball、murch、nickler、nothingmuch、orfeas、pinheadmz、pizzafrank13、potatoe_face、pyskell、pyskl、queip、r251d、raj_149、real_or_random、robert_spigler、roconnor、sanket1729、schmidty、sipa、soju、sosthene、stortz、taky、t-bast、theStack、Tibo、waxwing、xoyi- 以及 ZmnSCPxj*。

**GitHub PR**

taproot 在 Bitcoin Core 中的主要实现从 2020 年 1 月开始，分成[两次][two] [PR][requests] 来提交。下列人员在这些 PR 中留下了自己的审核意见。

*Andrew Chow (achow101)、Anthony Towns (ajtowns)、Antoine Riard (ariard)、Ben Carman (benthecarman)、Ben Woosley (Empact)、Bram (brmdbr)、Cory Fields (theuni)、Dmitry Petukhov (dgpv)、Elichai Turkel (elichai)、Fabian Jahr (fjahr)、Andreas Flack (flack)、Gregory Maxwell (gmaxwell)、Gregory Sanders (instagibbs)、James O’Beirne (jamesob)、Janus Troelsen (ysangkok)、Jeremy Rubin (JeremyRubin)、João Barbosa (promag)、John Newbery (jnewbery)、Jon Atack (jonatack)、Jonathan Underwood (junderw)、Kalle Alm (kallewoof)、Kanon (decryp2kanon)、kiminuo、Luke Dashjr (luke-jr)、Marco Falke (MarcoFalke)、Martin Habovštiak (Kixunil)、Matthew Zipkin (pinheadmz)、Max Hillebrand (MaxHillebrand)、Michael Folkson (michaelfolkson)、Michael Ford (fanquake)、Adam Ficsor (nopara73)、Pieter Wuille (sipa) Sjors Provoost (Sjors)、Steve Huguenin-Elie (StEvUgnIn)、Tim Ruffing (real-or-random) 以及 Yan Pritzker (skwp)*。

这里没有记录 Bitcoin Core 其他相关 PR 和其它软件中的 taproot 实现工作（的贡献者），包括在 libsecp256k1 代码库（Bitcoin Core 就使用这个代码库）和其它节点软件中的 schnorr 签名支持。

**Taproot 激活方法讨论**

自 taproot 实现合并到 Bitcoin Core 代码库中开始，如何激活它就是社区的事了。讨论持续了几个月，在讨论 taproot 激活的 IRC 频道中，下列的用户、开发者和矿工最为活跃：

*6102bitcoin、AaronvanW、achow101、aj、alec、Alexandre_Chery、Alistair_Mann、amiti、andrewtoth、andytoshi、AnthonyRonning、ariel25、arturogoosnargh、AsILayHodling、averagepleb、bcman、belcher、benthecarman、Billy、bitcoinaire、bitentrepreneur、bitsharp、bjarnem、blk014、BlueMatt、bobazY、brg444、btcactivator、btcbb、cato、catwith1hat、cguida、CodeShark__、conman、copumpkin、Crash78、criley、CriptoLuis、CubicEarth、darbsllim、darosior、Day、DeanGuss、DeanWeen、debit、Decentralizedb、devrandom、DigDug、dome、dr_orlovsky、duringo、dustinwinski、eeb77f71f26eee、eidnrf、elector、elichai2、Emcy、emzy、entropy5000、eoin、epson121、erijon、eris、evankaloudis、faketoshi、fanquake、fedorafan、felixweis、fiach_dubh、fjahr、friendly_arthrop、GeraldineG、gevs、gg34、ghost43、ghosthell、giaki3003、gloved、gmaxwell、graeme1、GreenmanPGI、gr-g、GVac、gwillen、gwj、gz12、gz77、h4shcash、harding、hebasto、hiro8、Hotmetal、hsjoberg、huesal、instagibbs、Ironhelix、IT4Crypto、ja、jaenu、JanB、jeremyrubin、jimmy53、jnewbery、jonatack、jonny100051、jtimon、kallewoof、kanon、kanzure、Kappa、keblek、ksedgwic、landeau、lucasmoten、luke-jr、maaku、Majes、maybehuman、mblackmblack、mcm-mike、Memesan、michaelfolkson、midnight、MikeMarzig、mips、mol、molz、moneyball、mrb07r0、MrHodl、murch、naribia、newNickName、nickler、nikitis、NoDeal、norisgOG、nothingmuch、occupier、OP_NOP、OtahMachi、p0x、pinheadmz、PinkElephant、pox、prayank、prepaid、proofofkeags、provoostenator、prusnak、qubenix、queip、r251d、rabidus、Raincloud、raj、RamiDz94、real_or_random、rgrant、riclas、roasbeef、robert_spigler、rocket_fuel、roconnor、rovdi、rubikputer、RusAlex、rusty、sanket1729、satosaurian、schmidty、sdaftuar、setpill、shesek、shinobiusmonk、snash779、solairis、somethinsomethin、stortz、sturles、sugarpuff、taPrOOteD、TechMiX、TheDiktator、thomasb06、tiagocs、tomados、tonysanak、TristanLamonica、UltrA1、V1Technology、vanity、viaj3ro、Victorsueca、virtu、walletscrutiny、wangchun、warren、waxwing、Whatisthis、whuha、willcl_ark、WilliamSantiago、windsok、wumpus、xxxxbtcking、yanmaani、yevaud、ygrtiugf、Yoghurt11411、zmnscpxj 以及 zndtoshi*。

**矿工表态**

我们同样感谢所有在 681408 区块高度后表态愿意执行 taproot 共识规则的矿工。

**辅助项目**

Taproot 的激活只是个开始。现在，需要开发者和用户开始利用 taproot 支持的新功能。一些人已经为此做了许多年的准备工作，在开发包括 [MuSig][MuSig] 在内的辅助项目。获得这些开发者的清单不是很便利，但我们也要感谢他们所有人。

**节点运营者**

更重要地是，我们所有人都应该感谢已经升级到 Bitcoin Core 0.21.1 版本（或其它兼容 taproot 的软件）的几千位比特币全验证节点的运营者，以及使用他们的节点来接收支付、确保他们从区块 709632 开始只会接受适用了 taproot 规则的区块的人。这为其他所有比特币用户只接受 taproot 兼容区块提供了经济激励，让所有人都能更安全地适用 taproot 的功能。

## 新版本和候选版本

*流行的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- <a id="btcpay-server-1-3-3" href="#btcpay-server-1-3-3)">●</a> [BTCPay Server 1.3.3][BTCPay Server 1.3.3] 是一个新版本，为也共享闪电网络节点的共享服务器实例提供了一个关键的安全补丁。也包含了少量的新功能以一些 bug 修复。
- <a id="rust-lightning-0-0-103" href="#rust-lightning-0-0-103)">●</a> [Rust-Lightning 0.0.103][Rust-Lightning 0.0.103] 是一个新版本，加入了 `InvoicePayer` API 来尝试重发交易（在某些路径失败时）。
- <a id="c-lightning-0-10-2" href="#c-lightning-0-10-2)">●</a> [C-Lightning 0.10.2][C-Lightning 0.10.2] 是一个新版本，[包含了][includes]对[不经济输出安全问题][uneconomical outputs security issue]的一个修复。数据库体积变得更小，并提高了 `pay` 命令的效率。
- <a id="lnd-0-13-4-beta" href="#lnd-0-13-4-beta)">●</a> [LND 0.13.4-beta][LND 0.13.4-beta] 是一个维护更新，修复了上面的新闻部分提到的 Neutrino 软件的 bug。更新日志说，“如果你在生产环境中适用 Neutrino，我们强烈建议你在 taproot 激活之前升级到这个版本，以保证你的节点不会脱链。”
- <a id="lnd-0-14-0-beta-rc3" href="#lnd-0-14-0-beta-rc3)">●</a> [LND 0.14.0-beta.rc3][LND 0.14.0-beta.rc3] 是一个候选版本，包含了额外的[日蚀攻击保护][eclipse attack]（见[周报 #164][Newsletter #164]）、远程数据库支持（见[周报 #157][Newsletter #157]）、快速查找路径（[Newsletter #170][Newsletter #170]）、闪电 Pool 使用体验提升（见[周报 #172][Newsletter #172]）以及重用[原子化多路径支付（AMP）][AMP]收款请求（见[周报 #173][Newsletter #173]），以及其它特性和 bug 修复。

## 重要的代码和文献变更

本周有重大变更的包括 [Bitcoin Core][Bitcoin Core]、[C-Lightning][C-Lightning]、[Eclair][Eclair]、[LND][LND]、[Rust-Lightning][Rust-Lightning]、[libsecp256k1][libsecp256k1]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK] 和 [Lightning BOLTs][Lightning BOLTs]。

- <a id="rust-lightning-1078" href="#rust-lightning-1078)">●</a> [Rust-Lightning #1078][Rust-Lightning #1078] 加入了 [BOLTs #880][BOLTs #880] 定义的 `channel_type` 协商，我们在[周报 #165][Newsletter #165]里面介绍过。这个实现当前并不发送 [BOLTs #906][BOLTs #906] 提议的特性位。“[锚点通道（anchor channel）][anchor channels]” 需要 BOLTs #880，可能 “[零确认通道][zero-conf channels]” 也需要。
- <a id="rust-lightning-1144" href="#rust-lightning-1144)">●</a> [Rust-Lightning #1144][Rust-Lightning #1144] 为路由评分逻辑加入了一个惩罚机制。惩罚会适用在支付重试之间失败的通道中，以提醒寻路算法可能的错误通道。
- <a id="bips-1215" href="#bips-1215)">●</a> [BIPs #1215][BIPs #1215] 更新了 [BIP119][BIP119] 提出的 [OP_CHECKTEMPLATEVERIFY][OP_CHECKTEMPLATEVERIFY] 提议：
  - 指明了软分叉需要适用一个[快速适用（speedy trial）][speedy trial] 激活流程来部署，类似于 taproot 激活。
  - 文档记录了使用不带标签的 SHA256 哈希值的设计理由。
  - 加入了更多 OP_CHECKTEMPLATEVERIFY 提议和 [SIGHASH_ANYPREVOUT][SIGHASH_ANYPREVOUT] 提议的比较。
  - 解释了OP_CHECKTEMPLATEVERIFY 和其它可能的共识变更的相互作用。



[thread]:https://mailmanlists.org/pipermail/dlc-dev/2021-November/000091.html

[Discreet Log Contracts]:https://bitcoinops.org/en/topics/discreet-log-contracts/

[posted]:https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-November/003336.html

[taproot]:https://bitcoinops.org/en/topics/taproot/

[PTLCs]:https://bitcoinops.org/en/topics/ptlc/

[MuSig2]:https://bitcoinops.org/en/topics/musig/

[multisignatures]:https://bitcoinops.org/en/topics/multisignature/

[eltoo]:https://bitcoinops.org/en/topics/eltoo/

[offers]:https://bitcoinops.org/en/topics/offers/

[Newsletter #53]:https://bitcoinops.org/en/newsletters/2019/07/03/#stuckless-payments

[channel jamming attacks]:https://bitcoinops.org/en/topics/channel-jamming-attacks/

[trampoline routing]:https://bitcoinops.org/en/topics/trampoline-payments/

[Neutrino]:https://github.com/lightninglabs/neutrino

[compact block filter]:https://bitcoinops.org/en/topics/compact-block-filters/

[patched]:https://github.com/lightninglabs/neutrino/pull/234

[Bitcoin-Dev]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-November/019589.html

[LND-Dev]:https://groups.google.com/a/lightning.engineering/g/lnd/c/CE2EslTiqW4/m/CSV3mL5JBQAJ

[verified]:https://groups.google.com/a/lightning.engineering/g/lnd/c/CE2EslTiqW4/m/O0_kQF7mBQAJ

[Bitcoin Core PR Review Club]:https://bitcoincore.reviews/

[Add `ChainstateManager::ProcessTransaction`]:https://bitcoincore.reviews/23173

[series]:https://bitcoinops.org/en/preparing-for-taproot/

[originated]:https://bitcoinops.org/en/preparing-for-taproot/#a-good-morning

[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015614.html

[schnorr signatures]:https://bitcoinops.org/en/topics/schnorr-signatures/

[MAST]:https://bitcoinops.org/en/topics/mast/

[organized review]:https://bitcoinops.org/en/newsletters/2019/10/23/#taproot-review

[two]:https://github.com/bitcoin/bitcoin/issues/17977

[requests]:https://github.com/bitcoin/bitcoin/issues/19953

[MuSig]:https://bitcoinops.org/en/topics/musig/

[BTCPay Server 1.3.3]:https://github.com/btcpayserver/btcpayserver/releases/tag/v1.3.3

[Rust-Lightning 0.0.103]:https://github.com/rust-bitcoin/rust-lightning/releases/tag/v0.0.103

[C-Lightning 0.10.2]:https://github.com/ElementsProject/lightning/releases/tag/v0.10.2

[includes]:https://twitter.com/Snyke/status/1452260691939938312

[uneconomical outputs security issue]:https://bitcoinops.org/en/newsletters/2021/10/13/#ln-spend-to-fees-cve

[LND 0.13.4-beta]:https://github.com/lightningnetwork/lnd/releases/tag/v0.13.4-beta

[LND 0.14.0-beta.rc3]:https://github.com/lightningnetwork/lnd/releases/tag/v0.14.0-beta.rc3

[eclipse attack]:https://bitcoinops.org/en/topics/eclipse-attacks/

[Newsletter #164]:https://bitcoinops.org/en/newsletters/2021/09/01/#lnd-5621

[Newsletter #157]:https://bitcoinops.org/en/newsletters/2021/07/14/#lnd-5447

[Newsletter #170]:https://bitcoinops.org/en/newsletters/2021/10/13/#lnd-5642

[Newsletter #172]:https://bitcoinops.org/en/newsletters/2021/10/27/#lnd-5709

[AMP]:https://bitcoinops.org/en/topics/atomic-multipath/

[Newsletter #173]:https://bitcoinops.org/en/newsletters/2021/11/03/#lnd-5803

[Bitcoin Core]:https://github.com/bitcoin/bitcoin

[C-Lightning]:https://github.com/ElementsProject/lightning

[Eclair]:https://github.com/ACINQ/eclair

[LND]:https://github.com/lightningnetwork/lnd/

[Rust-Lightning]:https://github.com/rust-bitcoin/rust-lightning

[libsecp256k1]:https://github.com/bitcoin-core/secp256k1

[Rust Bitcoin]:https://github.com/rust-bitcoin/rust-bitcoin

[BTCPay Server]:https://github.com/btcpayserver/btcpayserver/

[BDK]:https://github.com/bitcoindevkit/bdk

[Lightning BOLTs]:https://github.com/lightningnetwork/lightning-rfc/

[Rust-Lightning #1078]:https://github.com/rust-bitcoin/rust-lightning/issues/1078

[BOLTs #880]:https://github.com/lightningnetwork/lightning-rfc/issues/880

[Newsletter #165]:https://bitcoinops.org/en/newsletters/2021/09/08/#bolts-880

[BOLTs #906]:https://github.com/lightningnetwork/lightning-rfc/issues/906

[anchor channels]:https://bitcoinops.org/en/topics/anchor-outputs/

[zero-conf channels]:https://bitcoinops.org/en/newsletters/2021/07/07/#zero-conf-channel-opens

[Rust-Lightning #1144]:https://github.com/rust-bitcoin/rust-lightning/issues/1144

[BIPs #1215]:https://github.com/bitcoin/bips/issues/1215

[OP_CHECKTEMPLATEVERIFY]:https://bitcoinops.org/en/topics/op_checktemplateverify/

[BIP119]:https://github.com/bitcoin/bips/blob/master/bip-0119.mediawiki

[speedy trial]:https://bitcoinops.org/en/newsletters/2021/03/10/#a-short-duration-attempt-at-miner-activation

[SIGHASH_ANYPREVOUT]:https://bitcoinops.org/en/topics/sighash_anyprevout/



