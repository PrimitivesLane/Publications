---
title: 'Bitcoin Optech Newsletter #192'
permalink: /zh/newsletters/2022/03/23/
name: 2022-03-23-newsletter-zh
slug: 2022-03-23-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---
本周的 Newsletter 总结了关于快速试验软分叉激活机制的讨论，以及优化 LN 寻路算法的更新的链接。还包括我们的常规部分，近期的服务和客户端软件的变更，新版本和候选版本的公告，以及主流的比特币基础设施软件中值得注意的变更。


## 新闻

- **快速试验的讨论：** 在最近一次关于提议的 [OP_CHECKTEMPLATEVERIFY][topic op_checktemplateverify] 操作码的会议摘要中，提到了[快速试验的软分叉激活方法][topic soft fork activation]，在 Jorge Timón 对使用快速试验的软分叉表示担忧（他认为这是一个有争议的问题）后，在 Bitcoin-Dev 邮件列表中单独开了一个话题进行讨论。

  Russell O'Connor [解释][oconnor st]了以前是如何解决这些问题的。Anthony Towns 进一步[描述][towns st]了反对的用户如何抵制使用快速试验的软分叉激活。

- **支付算法的更新：** René Pickhardt 在 Lightning-Dev 邮件列表中[发文称][pickhardt payment delivery]，他找到了对他和 Stefan Richter 去年发表的寻路算法多的计算近似值的方法。有关该算法的早期讨论，请参见 [Newsletter #163][news163 pp]。

  Pickhardt 的邮件还提出了可以改进快速支付成功率的方法，例如通过实施[无滞碍支付][news53 stuckless]和[允许超额支付退款][news86 boomerang]（如[几篇][boomerang]学术[论文][spear]所建议的）。

## 服务和客户端软件的变更

*在这个月的栏目中，我们突出比特币钱包和服务的有趣升级*。

- **Teleport Transactions 实现了 Coinswap：** 在最近的 Bitcoin-Dev 邮件列表中，Chris Belcher [宣布][belcher teleport]了 Teleport Transactions 的 alpha [版本 0.1][teleport transactions 0.1]，实现了 [coinswap][topic coinswap] 协议。

- **JoinMarket 增加了 taproot 发送：** [JoinMarket v0.9.5][joinmarket v0.9.5] 增加了向 [bech32m][topic bech32] 地址发送交易的功能。

- **Mercury 钱包增加了 RBF 支持：** Mercury 钱包是 Mercury [状态链][topic statechains]的钱包，发布了 [v0.6.5][mercury v0.6.5] 版本，包括对取款的[使用费用替换（RBF）][topic rbf]的交易替换的支持。

- **Hexa 钱包增加对闪电网络的支持：** Hexa 钱包是一个比特币移动钱包，在 [v2.0.71 版本][hexa v2.0.71]中为运行自己节点的 LND 用户增加了闪电网络功能。

- **Sparrow 增加了 BIP47 支持：** [Sparrow 1.6.0][sparrow 1.6.0]（以及 1.6.1 和 1.6.2 版本）支持了 [BIP47][] 可重复使用的支付代码并增加了相关的功能，同时也[介绍了这些功能][sparrow wallet tweet]。

## 新版本和候选版本

*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- [HWI 2.1.0-rc.1][] 是 HWI 的候选版本，增加了对几个硬件签名设备的 taproot 支持，还包含一些其他改进和错误修复。

## 代码和文档的重大变更
*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Eclair #2203][] 增加了额外的配置参数，允许用户为[未公开的通道][topic unannounced channels]指定不同的最小金额，而不是使用公开通道的默认值。

- [LDK #1311][] 增加了对 [BOLTs #910][] 提出的短通道标识符（SCID）`alias` 字段的支持，它允许节点要求其对等节点通过一个任意的值来识别一个通道，而不是从锚定通道的链上交易中获得。这对隐私很有帮助，因为可以防止 SCID 向第三方泄露节点创建的交易，它也被提议用于选择加入零配置通道（有时称为*涡轮通道*）的规范，如 [Newsletter #156][news156 zcc] 所述。

- [LDK #1286][] 增加了 CLTV（`OP_CHECKLOCKTIMEVERIFY`）值的偏移量，用于 [BOLT7 建议的][bolt7 route rec]路由支付。这使得观察部分付款尝试的人（例如，路由付款的节点之一）更难正确猜测哪个节点可能是预定的接收者。

- [HWI #584][] 增加了特性，在使用 BitBox02 硬件签名设备的最新固件版本时，可以支付到 [bech32m][topic bech32] 地址。

- [HWI #581][] 在使用带有未来固件版本的 Trezor 时，禁用具有外部输入的交易的签名（例如，在 [coinjoin][topic coinjoin] 中）。这个 PR 是在固件改变之后，破坏了 HWI 用来实现支持的一个变通方法后被提出的。一个后续的 PR（[HWI #590][]）似乎表明，Trezor 正在考虑在未来给用户提供一种方法来签署此类交易。

- [BDK #515][] 开始在内部数据库中保留已花费的交易输出。这对创建[替代交易][topic rbf]很有帮助，并简化了 [BIP47][] 可重复使用的支付代码的[正在进行的实现][bdk #549]。


[topic op_checktemplateverify]: https://bitcoinops.org/en/topics/op_checktemplateverify/
[topic soft fork activation]: https://bitcoinops.org/en/topics/soft-fork-activation/
[topic coinswap]: https://bitcoinops.org/en/topics/coinswap/
[topic bech32]: https://bitcoinops.org/en/topics/bech32/
[topic statechains]: https://bitcoinops.org/en/topics/statechains/
[topic rbf]: https://bitcoinops.org/en/topics/replace-by-fee/
[topic unannounced channels]: https://bitcoinops.org/en/topics/unannounced-channels/
[topic coinjoin]: https://bitcoinops.org/en/topics/coinjoin/

[BOLTs #910]: https://github.com/lightning/bolts/issues/910
[hwi 2.1.0-rc.1]: https://github.com/bitcoin-core/HWI/releases/tag/2.1.0-rc.1
[bolt7 route rec]: https://github.com/lightning/bolts/blob/master/07-routing-gossip.md#recommendations-for-routing
[oconnor st]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020106.html
[timon st]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020102.html
[towns st]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020127.html
[pickhardt payment delivery]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2022-March/003510.html
[news163 pp]: https://bitcoinops.org/en/newsletters/2021/08/25/#zero-base-fee-ln-discussion
[news53 stuckless]: https://bitcoinops.org/en/newsletters/2019/07/03/#stuckless-payments
[spear]: https://dl.acm.org/doi/10.1145/3479722.3480997
[news156 zcc]: https://bitcoinops.org/en/newsletters/2021/07/07/#zero-conf-channel-opens
[boomerang]: https://arxiv.org/pdf/1910.01834.pdf
[news86 boomerang]: https://bitcoinops.org/en/newsletters/2020/02/26/#boomerang-redundancy-improves-latency-and-throughput-in-payment-channel-networks
[belcher teleport]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/020026.html
[teleport transactions 0.1]: https://github.com/bitcoin-teleport/teleport-transactions/releases/tag/0.1
[joinmarket v0.9.5]: https://github.com/JoinMarket-Org/joinmarket-clientserver/releases/tag/v0.9.5
[mercury v0.6.5]: https://github.com/layer2tech/mercury-wallet/releases/tag/v0.6.5
[hexa v2.0.71]: https://github.com/bithyve/hexa/releases/tag/v2.0.71
[sparrow 1.6.0]: https://github.com/sparrowwallet/sparrow/releases/tag/1.6.0
[sparrow wallet tweet]: https://twitter.com/SparrowWallet/status/1504458210366922759
[BIP47]: https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki
[Eclair #2203]: https://github.com/ACINQ/eclair/pull/2203
[LDK #1311]: https://github.com/lightningdevkit/rust-lightning/issues/1311
[LDK #1286]: https://github.com/lightningdevkit/rust-lightning/issues/1286
[HWI #584]: https://github.com/bitcoin-core/HWI/pull/584
[HWI #581]: https://github.com/bitcoin-core/HWI/pull/581
[BDK #515]: https://github.com/bitcoindevkit/bdk/pull/515
[HWI #590]: https://github.com/bitcoin-core/HWI/pull/590
[bdk #549]: https://github.com/bitcoindevkit/bdk/issues/549

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
