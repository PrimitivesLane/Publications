---
title: 'Bitcoin Optech Newsletter #171'
permalink: /zh/newsletters/2021/10/20/
name: 2021-10-20-newsletter-zh 
slug: 2021-10-20-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 总结了一个关于向经常离线的 LN 节点支付的主题，描述了一套降低 LN 支付路径探测成本的提案，以使某些攻击更加昂贵，并引介了在 signet 和 testnet 上创建 taproot 交易的有用的介绍。还包括我们的常规部分，包括了客户端和服务的最新变化，新版本和候选版本，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻
- **向离线的 LN 节点付款：** Matt Corallo 在 Lightning-Dev 邮件列表中发起了一个[主题](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-October/003307.html)，讨论经常离线的 LN 节点（如移动设备上的节点）如何在不需要托管解决方案或长期锁定通道流动性的情况下接收付款。Corallo 认为，一旦 LN [升级](https://bitcoinops.org/en/preparing-for-taproot/#ln-with-taproot)到支持 [PTLC](https://bitcoinops.org/en/topics/ptlc/)，涉及不受信任的第三方的场景会有一个合理的好的的解决方案。但他仍然在寻求可以在添加 PTLC 支持之前部署的替代解决方案的建议。

- **降低探测成本，使攻击更加昂贵：** 相隔几周，开发者 ZmnSCPxj 和 Joost Jager 分别提出了[基本](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-September/003256.html)相似的[提案](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-October/003314.html)，即取消为探测支付路径而锁定资金的要求。这两个提案都建议将此作为在网络中增加预付路由费的第一步，即使支付尝试失败也会让花费者付钱。预付费用是对[信道干扰攻击](https://bitcoinops.org/en/topics/channel-jamming-attacks/)的建议缓解措施之一。

  目前，LN 节点可以发送失败保障支付，以探测支付路径。例如，Alice 生成了一个 [HTLC](https://bitcoinops.org/en/topics/htlc/)，支付一个秘密的原像。她通过 Bob 和 Charlie 将付款路由给 Zed。由于 Zed 不知道付款的预象，他被迫拒绝付款，尽管他是最终的接收者。如果 Alice 收到 Zed 节点的拒绝信息，她知道 Bob 和 Charlie 在他们的通道中有足够的资金可以支付给 Zed，因此她可以立即对 Zed 的拒绝做出反应，发送一个实际的付款，这个付款就有很高的成功概率（唯一与流动性有关的失败原因是如果 Bob 或 Charlie 的余额在这期间发生了变化）。对 Alice 来说，先使用失败保障的探测的好处是，她可以并行地探测多条路径，并使用任何一条先成功的路径，减少整个支付时间。主要的缺点是，每个支付探测都会暂时锁定属于 Alice 以及 Bob 和 Charlie 等中间节点的资金；如果 Alice 并行探测几条长路径，她可能很容易锁定 100 倍甚至更多的支付金额。一个次要的缺点是，这种类型的支付路径探测有时会导致不必要的单边通道关闭及其产生的链上费用。

  ZmnSCPxj 和 Jager 提议允许发送一个特殊的探测信息，不需要使用 HTLC 暂时锁定比特币，或冒通道失败的风险。该消息基本上是免费发送的，尽管 ZmnSCPxj 的提案确实也提出了一些缓解措施以避免拒绝泛洪服务攻击。

  如果免费探测确实使得支出节点在很大比例的时间找到可靠的路径，ZmnSCPxj 和 Jager 认为，那么开发者和用户应该不那么抗拒支付预付费用，用户只会在极少数的支付失败的情况下付出代价。对诚实的用户来说，这种小的偶然成本将成为执行广泛干扰攻击的不诚实节点的高成本，并减少这种攻击发生的风险（并补偿路由节点运营商在持续攻击发生时部署额外资金以改善网络）。

  截至本文写作时，这个想法得到了一定程度的讨论。

- **测试taproot：** 为了回应 Bitcoin-Dev 邮件列表中的[请求](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-October/019532.html)，Anthony Towns [提供](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-October/019543.html)了分步骤的说明，用于在 [signet](https://bitcoinops.org/en/topics/signet/) 或 testnet 上创建一个 [bech32m](https://bitcoinops.org/en/topics/bech32/) 地址来测试比特币核心。对于测试 taproot 的开发者来说，这些说明可能比[之前 Optech 提供](https://bitcoinops.org/en/preparing-for-taproot/#testing-on-signet)的说明更容易使用。

## 服务和客户端软件的变更
*在这个月的专题中，我们会重点介绍比特币钱包和服务中有意思的更新。*

- **Zeus 钱包增加了 LN 功能：** Zeus，一个移动比特币和闪电钱包应用程序，在其 [v0.6.0-alpha3](https://github.com/ZeusLN/zeus/releases/tag/v0.6.0-alpha3) 版本中，提供了对[原子多路径支付（AMP）](https://bitcoinops.org/en/topics/atomic-multipath/)、[闪电地址](https://bitcoinops.org/en/newsletters/2021/09/22/#lightning-address-identifiers-announced)和[硬币控制](https://bitcoinops.org/en/topics/coin-selection/)功能的额外支持。

- **Sparrow 增加了 coinjoin 支持：** [Sparrow 1.5.0](https://github.com/sparrowwallet/sparrow/releases/tag/1.5.0) 通过与 Samourai 的 [Whirlpool](https://bitcoiner.guide/whirlpool/) 整合，增加了 [coinjoin](https://bitcoinops.org/en/topics/coinjoin/) 功能。

- **JoinMarket 0.9.2 增加了 RBF 支持：** [JoinMarket 0.9.3](https://github.com/JoinMarket-Org/joinmarket-clientserver/releases/tag/v0.9.3) 修复了一个针对挂单者的关键错误。我们鼓励所有挂单者进行升级。除了在用户界面中默认使用[忠诚保证保险](https://bitcoinops.org/en/newsletters/2021/08/11/#implementation-of-fidelity-bonds)外，JoinMarket 0.9.2 还支持非 coinjoin 交易的费用替换（RBF）。

- **Coldcard 支持基于描述符的钱包：** [Coldcard 4.1.3](https://blog.coinkite.com/version-4.1.3-released/) 现在支持比特币核心中的 `importdescriptors`，使描述符钱包和 [PSBT](https://bitcoinops.org/en/topics/psbt/) 工作流与比特币核心结合。

- **Simple Bitcoin 钱包增加了 CPFP、RBF、持有账单：** Simple Bitcoin 钱包，以前被称为比特币闪电钱包，在 2.2.14 版本中增加了[子代父支付（CPFP）](https://bitcoinops.org/en/topics/cpfp/)和 RBF（费用变更和取消）功能，在 [2.2.15](https://github.com/btcontract/wallet/releases/tag/2.2.15) 版本中增加了[持有账单](https://bitcoinops.org/en/topics/hold-invoices/)。

- **Electrs 0.9.0发布：** [Electrs 0.9.0](https://github.com/romanz/electrs/releases/tag/v0.9.0) 现在使用比特币的 P2P 协议，而不是从磁盘或 JSON RPC 读取块。用户应参考[升级指南](https://github.com/romanz/electrs/blob/master/doc/usage.md#important-changes-from-versions-older-than-090)，了解升级的细节。

## 为 taproot 作准备 #18：趣闻
*关于开发者和服务提供者如何为即将在区块高度 709,632 处激活的 taproot 作准备的每周[系列](https://bitcoinops.org/en/preparing-for-taproot/)文章。*

- **什么是 taproot？** 维基百科[说](https://en.wikipedia.org/wiki/Taproot)："taproot 是一个大的、中心的和主要的根，其他的根从这里横向发芽。通常情况下，直根直且粗粗，呈锥形，并直接向下生长。在一些植物中，如胡萝卜，直根是一个储存器官，非常发达，以至于它被当作蔬菜来栽培。"

  这怎么适用到比特币的场景？

  - "我一直以为这个名字的来源是'利用默克尔根'（taps into the Merkle root），但实际上我不知道 Gregory Maxwell 的想法是什么。" —— Pieter Wuille（[来源](https://github.com/bitcoinops/bitcoinops.github.io/pull/667#discussion_r731371163)）

  - "我一开始不得不去查这个词；但我认为它的关键路径是'taproot'，因为胡萝卜汤的美味来自于此，而默克尔化的脚本则是你希望忽略的其他较小的根。" —— Anthony Towns（[来源](https://github.com/bitcoinops/bitcoinops.github.io/pull/667#discussion_r731523855)）

  - "这个名字起源于一个可视化的树，它有粗壮的树干，就像蒲公英的直根——这个技术有用主要是因为假设存在一个高概率的路径，其余都是细小的分支。我认为这很好因为它通过利用（tap into）根部（root）的隐藏承诺来验证脚本路径的花费。

  - [......]可惜的是，把有内部节点排序的散列树称为 "桃金娘树 "并不合适。（提到桃金娘树是因为具有相同哈希根的策略集是那些排序不同的策略，其排序可由 t 函数定义，而桃金娘（Myrtle）包括千层树(一种茶树)，听起来像 'merkle'。 :p）" —— Gregory Maxwell（[来源](https://github.com/bitcoinops/bitcoinops.github.io/pull/667#discussion_r732189216)）

- **Schnorr 签名早于 ECDSA：** 我们说 [Schnorr 签名](https://bitcoinops.org/en/topics/schnorr-signatures/)是对比特币原始 ECDSA 签名的升级，因为它们更容易实现各种加密技巧，但 Schnorr 签名算法早于 ECDSA 所基于的 DSA 算法。事实上，创建 DSA 一部分就是为了规避 Clause Schnorr 的 [Schnorr 签名专利](https://patents.google.com/patent/US4995082)，但 Schnorr 仍然声称"[我的]专利适用于离散对数签名的各种实现，因此包括了Nyberg-Rueppel 和 DSA 签名的使用。" 据了解，没有法院支持 Schnorr 的主张，他的专利在 2011 年过期了。

- **不确定使用什么名字：** 虽然没有最终被应用，但在为比特币适配 Schnorr 签名的发展初期，有一个[建议](https://diyhpl.us/wiki/transcripts/discreet-log-contracts/)是，Claus Schnorr 的名字不应该被关联进来，因为他的专利阻止了一项有价值的加密技术的广泛使用超过20年。Pieter Wuille 写道："我们确实考虑过将 BIP340 称为 'DLS'，即'离散对数签名'，但我们最终没有这么做，因为 Schnorr 这个名字已经被广泛提及。"

- **扭曲的 Edwards 曲线的 Schnorr 签名：** 2011 年发表了一个使用椭圆曲线的 Schnorr 签名的应用。这个方案，[EdDSA](https://en.wikipedia.org/wiki/EdDSA)，现在是几个标准的基础。虽然没有在比特币共识中使用，但在 Optech 跟踪的许多比特币库中可以发现在其他系统上下文中对它的引用。
- **支付到合约：** Ilja Gerhardt 和 Timo Hanke 创建了一个[协议](https://arxiv.org/abs/1212.3257)，由 Hanke 在 2013 年圣何塞比特币会议上提出，允许支付承诺到合约的哈希。任何拥有合约副本和 nonce（用于避免某些攻击）的人都可以验证该承诺，但对其他人来说，该付款看起来就像任何其他比特币付款。

  2014 年关于[侧链](https://bitcoinops.org/en/topics/sidechains/)的[论文](https://www.blockstream.com/sidechains.pdf)中包含了对这种支付到合约（P2C）协议的微小改进，其中的承诺还包括原始公钥的支付。Taproot 使用了相同的构造，但是，输出的创建者不是承诺到链下的合约的条款，而是承诺到由接收者选择的共识强制条款，这些条款规定了在链上如何花费收到的比特币。

- **早上好：** 使用 P2C 使得支付到脚本可以在链上看起来与支付到公钥相同的想法，是 2018 年 1 月 22 日在加州洛斯阿尔托斯的小餐馆 "早上好"被想到的。Pieter Wuille 写道，这个想法是由 Andrew Poelstra 和 Gregory Maxwell 提出的，"而我短暂地离开了桌子......！$%@" [原文如此]。

- **2.5 年与 1.5 天：** 为 [bech32m](https://bitcoinops.org/en/topics/bech32/) 选择最佳常数需要大约 2.5 年的 CPU 时间，但是用 Gregory Maxwell 的 CPU 集群在短短1.5天内就完成了。

*我们感谢 Anthony Towns、Gregory Maxwell、Jonas Nick、Pieter Wuille 和 Tim Ruffing 与本专栏的愉快谈话。任何错误都是作者的问题。*

## 发布和候选发布
*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- [BDK 0.12.0](https://github.com/bitcoindevkit/bdk/releases/tag/v0.12.0) 是一个增加了使用 Sqlite 存储数据能力的版本，同时也还有其他改进。

## 重大代码和文档更新
*本周内，[Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[Hardware Wallet Interface(HWI)](https://github.com/bitcoin-core/HWI)、[Rust Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay Server](https://bitcoinops.org/en/newsletters/2021/08/11/)、[Bitcoin Improvement Proposals(BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 发布了值得注意的变更。*

- [Bitcoin Core #22863](https://github.com/bitcoin/bitcoin/pull/22863) 记录了一项决定，即对 P2TR 输出使用与 P2WPKH 输出相同的最低输出金额（"尘额"），294 聪。即使花费一个 P2TR 输出的成本比一个 P2WPKH 输出的成本低，也没有使用更低的尘额限制，因为一些开发者反对在这个时候降低尘额限制。

- [Bitcoin Core #23093](https://github.com/bitcoin/bitcoin/issues/23093) 增加了一个 `newkeypool` RPC方法，它将把钱包中所有预先生成的地址标记为已用，这样就会生成一组新的地址。大多数用户应该永远不需要这个，但如果用户从旧的非 [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 钱包升级到使用[分层确定性密钥生成](https://bitcoinops.org/en/topics/hd-key-generation/)，该功能会在后台运行。

- [Bitcoin Core #22539](https://github.com/bitcoin/bitcoin/issues/22539) 在生成费用估算时考虑本地节点看到的替换交易。替换交易[以前](https://github.com/bitcoin/bitcoin/pull/9519)很少发生，所以没有考虑，但目前约有 [20% 的交易](https://dashboard.bitcoinops.org/d/ZsCio4Dmz/rbf-signalling?orgId=1)发送了选择替换的 [BIP125](https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki) 信号，在一天里会有[几千个](https://github.com/bitcoin/bitcoin/pull/22539#issuecomment-885763670)替换交易发生。

- [Eclair #1985](https://github.com/ACINQ/eclair/pull/1985) 增加了一个新的 `max-exposure-satoshis` 配置设置，允许用户设置在通道因未完成的[不经济的支付](https://bitcoinops.org/en/topics/uneconomical-outputs/)而被迫关闭时，他们捐赠给矿工的最大金额。更多信息见[上周 Newsletter](https://bitcoinops.org/en/newsletters/2021/10/13/#ln-spend-to-fees-cve)中对 CVE-2021-41591 的描述。

- [Rust-Lightning #1124](https://github.com/rust-bitcoin/rust-lightning/issues/1124) 扩展了 `get_route` API，使其能够传递一个额外的参数，可用于避免重复使用以前失败的路由。这个额外的变更将使其更容易使用以前的路由成功和失败的结果来提高后续路由的质量。

- [BOLTs #894](https://github.com/lightningnetwork/lightning-rfc/issues/894) 在规范中增加了建议的检查。实现方可以利用他防止在链上路由支付不经济时，将过多的比特币捐给矿工。关于没有这些检查时可能出现的问题，请看[上周的 Newsletter](https://bitcoinops.org/en/newsletters/2021/10/13/#ln-spend-to-fees-cve)。
