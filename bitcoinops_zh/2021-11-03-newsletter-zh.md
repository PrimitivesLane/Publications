---
title: 'Bitcoin Optech Newsletter #173'
permalink: /zh/newsletters/2021/11/03/
name: 2021-11-03-newsletter-zh 
slug: 2021-11-03-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 总结了关于直接向矿工提交交易的讨论，为钱包实现介绍了一组建议的 taproot 测试向量，并包括我们关于准备 taproot 的常规部分，新版本和候选版本，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻

- **直接向矿工提交交易：** Lisa Neigut 在Bitcoin-Dev邮件列表中发起了一个[主题][neigut relay]，讨论取消 P2P 交易中继网络，让用户直接提交交易给矿工的可能性。这种操作模式的好处包括减少带宽需求，改善隐私，消除 [RBF][topic rbf] 和 [CPFP][topic cpfp] 包规则的复杂性，以及更好地沟通下一个区块的费率。但也有几个人持反对意见：

  - **带宽要求：** 几个答复指出，自 2016 年以来使用的[致密区块中继][topic compact block relay]，允许收到未确认交易的节点再次收到被包含在区块中的未确认交易时不接收它，实现带宽开销最小。后续的改进，如 [erlay][topic erlay]，将进一步减少未确认交易中继的带宽开销。

  - **改善隐私：** 尽管只向矿工提交交易可以实现在确认之前将这些交易隐藏在公众视野之外，但 Pieter Wuille [认为][wuille relay]，这也让矿工对网络有了特权视角。公开透明似乎是更可取的方向。

  - **RBF、CPFP 和包的复杂性：** 节点运营者确实比矿工对浪费资源的攻击更敏感，但 Gloria Zhao [指出][zhao relay]，这主要是一个程度问题。相同的攻击如果在更大范围内执行就会影响到矿工，因此矿工仍然需要与节点已经采用的相同的类型的防御措施。

  - **费率沟通：** 目前比特币核心用来估计费率的方法是基于接收未确认的交易，然后观察它们需要多长时间出现在确认的区块中。这确实落后于实时的费率信息，但 Pieter Wuille [建议][wuille relay]，可以通过使用其他以前提出的方法来改善，比如让矿工广播弱区块——那些没有足够的工作量证明以被添加到链上的区块。弱区块比 PoW 有效区块出现得更频繁，因此可以提供更多关于矿工目前正在进行的交易的最新信息。

  除了直接反驳之外，当前系统的几个优点也被提及：

    - **矿工匿名：** 目前有至少 5 万个节点在转发交易，矿工很容易通过悄悄地运营其中一个节点来接收他们需要的所有信息。假名开发者 ZmnSCPxj [表示][zmnscpxj relay]，强迫矿工保持一个固定的身份，即使是像 Tor 这样的匿名网络上的身份，也会使人们更容易识别矿工，并胁迫他们审查一些交易。

    - **抗审查：** 今天任何人都可以使用任何计算机启动一个节点，接收中继交易，连接挖矿设备，并开始挖矿。Pieter Wuille [指出][wuille relay]，直接将交易提交给矿工的情况就不是这样了。新的矿工需要宣传他们的节点，以便接收交易，但没有高度可行的能抗审查和抗[女巫攻击][sybil attacks]的方法来做到这一点。特别地，Wuille 指出，"矿工在链上发布他们的[提交 URL]的机制的想法没有用；这[需要]依赖于其他矿工不审查发布。"

    - **抗中心化：** Wuille 还指出，"额外提交的成本和复杂性与[每个矿工的]哈希率无关，但收益与[哈希率]成正比。这将使 "大多数钱包更容易只提交[交易]给最大的几个矿池"，会鼓励中心化。

- **Taproot 测试向量：** Pieter Wuille 在 Bitcoin-Dev 邮件列表中[发布][wuille test]了一组[测试向量][bips #1225]，他建议在 [BIP341][] 规范中加入 [taproot][topic taproot]。他们专注于 "钱包的实现，包括从密钥/脚本中计算默克尔根、tweak 、scriptPubKey；sigmsg、sighash、signature 的计算，用于密钥路径花费；以及控制区块的计算，用于脚本路径花费。"

    这些测试向量对于那些希望在[激活后不久][p4tr waiting]就能使用 taproot 的实现的开发者来说应该特别有用。

## 为 taproot 升级作准备 #20: 激活时会发生什么？

*关于开发者和服务提供者如何为即将在区块高度 709,632 处激活的 taproot 作准备的每周[系列][series]文章。*

在这篇文章发表后不到两周，taproot 预计将在 709,632 区块激活。我们知道我们期望发生什么，我们也知道一些可能的失败模式。

最好和最可能的结果是一切顺利。普通用户不应该注意到任何事情。只有那些仔细监控他们的节点或试图创建 taproot 交易的人才会注意到一些事情。在 709,631 区块，几乎所有我们知道的全节点都会执行相同的共识规则。一个区块之后，运行 Bitcoin Core 0.21.1、22.0 或相关版本的节点将执行早期软件版本不会执行的额外的 [taproot][topic taproot] 的规则。

这存在一个风险，即早期和后期版本的节点软件会接受不同的区块。这种情况在 2015 年激活 [BIP66][] 软分叉时发生了，导致了 6 个区块的分叉和几个更短的分叉。为了防止该问题的重演，已经做了大量的工程上的工作。类似的问题应该只会发生在 taproot 上，如果矿工故意开采一个 taproot 无效的区块，或者禁用硬编码到比特币核心和相关节点软件的安全措施。

具体来说，为了创造一个分叉，矿工需要在不遵守 taproot 规则的情况下，创建或接收一个从 taproot 输出（segwit v1 输出）中花费的交易。如果矿工这样做，且比特币节点运营商的经济共识拒绝了 taproot 无效区块，这些矿工将损失至少 6.25 BTC（在撰写本文时约为 40 万美元）。

在没有创建无效区块的情况下，我们无法确定这些节点运营商会怎么做——节点可以完全私下运行，但[代理测算][bitnodes]显示，可能有超过 50% 的节点运营商正在运行比特币核心的 taproot-enforcing 版本。这可能足以确保任何创建 taproot 无效区块的矿工将会被网络拒绝。

虽然不太可能，但理论上还是有可能出现暂时的分叉。使用 [ForkMonitor.info][] 等服务或使用比特币核心中的 `getchaintips` RPC，应该可以监测到它。如果它真的发生了，轻量级客户可能会收到错误的确认信息。虽然理论上有可能得到 6 个确认，就像 2015 年的分叉一样，这将意味着矿工损失将近 250 万美元（2015 年的损失约为 5 万美元）。我们希望，在价值如此巨大的情况下，矿工会贯彻执行他们六个月前发出信号表示已经准备就绪的 taproot 规则。

在我们可以想象的几乎所有失败的情况下，一个简单而有效的临时应对是提高你的确认限制。如果你通常在接受付款前等待 6 个确认，你可以在几个小时内迅速提高到 30 个确认，直到问题得到解决，或者确认需要一个更高的确认限制。

对于确信全节点运营商的经济共识会执行 taproot 规则的用户和服务，一个更简单的解决方案是只从 Bitcoin Core 0.21.1 或更高版本（或兼容的替代节点实现）获得关于哪些交易被确认的信息。

Optech 预计 taproot 的激活会顺利进行，但我们鼓励交易所和任何接受 709,632 区块附近大额交易的人升级他们的节点，或者准备在有任何问题迹象时暂时提高他们的确认限额。

## 新版本和候选版本

*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- [Bitcoin Core 0.20.2][] 是比特币核心的前一个分支的维护版本，[包含][bcc0.20.2 rn]小的功能和错误修复。

- [C-Lightning 0.10.2rc2][] 是一个候选发布版本，[包括][decker
  tweet]对[不经济的输出安全问题][news170
  unec bug]的修复，更小的数据库大小，以及 `pay` 命令有效性的改进（见下文*重大代码和文档更新*部分）。

## 重大代码和文档更新

*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Bitcoin Core #23306][] 使地址管理器能够支持每个 IP 地址多个端口。历史上，比特币核心一直使用一个固定的端口，8333，并且在进行自动出站连接时，强烈建议使用这个端口的地址。虽然有人认为这种行为可能有助于防止利用比特币节点对非比特币服务进行 DoS（通过在比特币网络上传播他们的地址），但它也使得通过观察网络流量来检测比特币节点变得容易。将每个（IP,端口）组合作为一个独立的地址，可以使未来的工作朝着更统一的地址处理方向发展。

- [C-Lightning #4837][] 增加了一个 `--max-dust-htlc-exposure-msat` 配置选项，限制了余额低于尘额限制的地址发起 HTLC。更多细节请参见我们[之前][news162 mdhemsat]对 Rust-Lightning 中类似选项的报道。

- [Eclair #1982][] 引入了一个新的日志文件，收集需要节点运营者采取行动的重要通知。同时发布的说明指出，节点运营者应当监控 `notifications.log`。

- [LND #5803][] 允许向同一张[原子多路径付款（AMP）][topic multipath
  payments]账单进行多次 [自发付款][topic spontaneous
  payments]，而不要求支出用户为重复付款执行额外步骤。对同一张账单接收多笔付款的能力是 AMP 的一个特点，这在现有的简单多路径支付实现中是不可能的。

- [BTCPay Server #2897][] 增加了支持 [LNURL-Pay][] 作为一种支付方式，也实现了对 [闪电地址][news167 lightning addresses] 的支持。



[c-lightning 0.10.2rc2]: https://github.com/ElementsProject/lightning/releases/tag/v0.10.2rc2
[neigut relay]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-October/019572.html
[topic rbf]: https://bitcoinops.org/en/topics/replace-by-fee/
[topic cpfp]: https://bitcoinops.org/en/topics/cpfp/
[topic compact block relay]: https://bitcoinops.org/en/topics/compact-block-relay/
[topic erlay]: https://bitcoinops.org/en/topics/erlay/
[wuille relay]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-October/019578.html
[zhao relay]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-October/019579.html
[zmnscpxj relay]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-October/019573.html
[sybil attacks]: https://en.wikipedia.org/wiki/Sybil_attack
[decker tweet]: https://twitter.com/Snyke/status/1452260691939938312
[news170 unec bug]: /en/newsletters/2021/10/13/#ln-spend-to-fees-cve
[bitcoin core 0.20.2]: https://bitcoincore.org/bin/bitcoin-core-0.20.2/
[bcc0.20.2 rn]: https://bitcoincore.org/en/releases/0.20.2/
[wuille test]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-November/019587.html
[bips #1225]: https://github.com/bitcoin/bips/issues/1225
[BIP341]: https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki
[topic taproot]: https://bitcoinops.org/en/topics/taproot/
[p4tr waiting]: /en/preparing-for-taproot/#why-are-we-waiting
[series]: https://bitcoinops.org/en/preparing-for-taproot/
[LNURL-Pay]: https://github.com/fiatjaf/lnurl-rfc/blob/luds/06.md
[news167 lightning addresses]: /en/newsletters/2021/09/22/#lightning-address-identifiers-announced
[news162 mdhemsat]: https://bitcoinops.org/en/newsletters/2021/08/18/#rust-lightning-1009
[forkmonitor.info]: https://forkmonitor.info/nodes/btc
[bitnodes]: https://bitnodes.io/nodes/
[BIP66]: https://github.com/bitcoin/bips/blob/master/bip-0066.mediawiki
[bcc0.20.2 rn]: https://bitcoincore.org/en/releases/0.20.2/
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
[Bitcoin Core #23306]: https://github.com/bitcoin/bitcoin/issues/23306
[C-Lightning #4837]: https://github.com/ElementsProject/lightning/issues/4837
[Eclair #1982]: https://github.com/ACINQ/eclair/issues/1982
[LND #5803]: https://github.com/lightningnetwork/lnd/issues/5803
[BTCPay Server #2897]: https://github.com/btcpayserver/btcpayserver/issues/2897
[news167 lightning addresses]: https://bitcoinops.org/en/newsletters/2021/09/22/#lightning-address-identifiers-announced
[topic spontaneous
  payments]: https://bitcoinops.org/en/topics/spontaneous-payments/
[topic multipath payments]: https://bitcoinops.org/en/topics/multipath-payments/