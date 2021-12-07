---
title: 'Bitcoin Optech Newsletter #177'
permalink: /en/newsletters/2021/12/01/
name: 2021-12-01-newsletter
slug: 2021-12-01-newsletter
type: newsletter
layout: newsletter
lang: zh
---
本周的 Newsletter 介绍了最近修复的不同 LN 软件之间的互操作性问题。并包括我们的常规部分，包括新版本和候选版本，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻

- **LN 的互操作性：** [Newsletter #165][news165 bolts880] 中介绍的最近对 LN 规范的变更被不同的 LN 节点以不同的方式实现，导致运行最新版本 LND 的节点无法与运行最新版本的 C-Lightning 和 Eclair 的节点新建通道。我们鼓励 LND 用户升级到错误修复版本，0.14.1（在下面的*新版本和候选版本*部分有介绍）。

  Lightning-Dev 邮件列表中出现了一个关于改进互操作性测试的相关[讨论][xraid interop]。之前创建和维护 LN [集成测试框架的][ln integration] Christian Decker 表示，他[认为][decker interop]基本的互操作性测试 "除了最严重的问题，不太可能发现任何问题"。参与讨论的 LN 开发者[指出][zmn interop]，发现这类错误是每个主要的实现方案提供候选版本的原因，也是鼓励生产系统的专家用户和管理员参与这些候选版本的测试的原因。

  对于任何有兴趣参与这种测试的人来说，Optech Newsletter 列出了四个主要的 LN 实现的候选版本，以及其他各种比特币软件。讨论中的 LND 版本在 Newsletters [#174][news174 lnd] 和 Newsletters [#175][news175 lnd] 中被列为候选版本。

## 新版本和候选版本

*流行的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- [LND 0.14.1-beta][] 是一个维护版本，修复了上面*新闻*部分和下面*重大变更*部分的 [LND #6026][] 中详细描述的问题。

## 代码和文档的重大变更
*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Bitcoin Core #16807][] 更新了地址验证，使用 [Newsletter #41][news41 bech32 error detection] 中描述的机制返回 [bech32和bech32m][topic bech32] 地址中可能的错误索引。如果替换错误少于两个，错误将被正确识别。该更新还提升了测试覆盖率，为地址验证码添加了更多的文档，并改进了解码失败时的错误信息，特别是区分 bech32 和 bech32m 的使用。

- [Bitcoin Core #22364][] 增加了对创建 [taproot][bip386] 的[描述符][topic
  descriptors]的支持。这允许钱包用户通过用他们的钱包创建一个默认的 bech32m 描述符，而不是导入一个描述符来生成和使用 P2TR 地址。

- [LND #6026][] 修复了 [BOLTs #880][] 显式通道类型协商的[实现][lnd #5669]问题（见 [Newsletter #165][news165 bolts880]）。对 LN 规范的[变更提案][bolts #906]将允许 LND 最终实现严格的显式协商。

- [Rust-Lightning #1176][] 增加了对[锚点输出][topic anchor outputs]费用更新的初步支持。截至该次代码合并，我们跟踪的四个 LN 实现都实现了对锚点输出的支持。

- [HWI #475][] 增加了对 [Blockstream Jade][news132 jade] 硬件签名的[支持][hwi support matrix]，并使用 [QEMU 模拟器][qemu website]进行测试。


[topic anchor outputs]: https://bitcoinops.org/en/topics/anchor-outputs/
[bip386]: https://github.com/bitcoin/bips/blob/master/bip-0386.mediawiki
[topic descriptors]: https://bitcoinops.org/en/topics/output-script-descriptors/
[topic bech32]: https://bitcoinops.org/en/topics/bech32/
[lnd 0.14.1-beta]: https://github.com/lightningnetwork/lnd/releases/tag/v0.14.1-beta
[news165 bolts880]: /en/newsletters/2021/09/08/#bolts-880
[xraid interop]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-November/003354.html
[decker interop]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-November/003358.html
[zmn interop]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-November/003365.html
[ln integration]: https://cdecker.github.io/lightning-integration/
[news174 lnd]: /en/newsletters/2021/11/10/#lnd-0-14-0-beta-rc3
[news175 lnd]: /en/newsletters/2021/11/17/#lnd-0-14-0-beta-rc4
[hwi support matrix]: https://hwi.readthedocs.io/en/latest/devices/index.html#support-matrix
[news132 jade]: /en/newsletters/2021/01/20/#blockstream-announces-jade-hardware-wallet
[qemu website]: https://www.qemu.org/
[news41 bech32 error detection]: /en/bech32-sending-support/#locating-typos-in-bech32-addresses
[LND #6026]: https://github.com/lightningnetwork/lnd/issues/6026
[Bitcoin Core #16807]: https://github.com/bitcoin/bitcoin/issues/16807
[Bitcoin Core #22364]: https://github.com/bitcoin/bitcoin/pull/22364
[lnd #5669]: https://github.com/lightningnetwork/lnd/issues/5669
[BOLTs #880]: https://github.com/lightning/bolts/issues/880
[bolts #906]: https://github.com/lightning/bolts/issues/906
[Rust-Lightning #1176]: https://github.com/rust-bitcoin/rust-lightning/issues/1176
[HWI #475]: https://github.com/bitcoin-core/HWI/issues/475

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