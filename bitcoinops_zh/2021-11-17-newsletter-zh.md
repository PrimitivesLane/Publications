---
title: 'Bitcoin Optech Newsletter #175'
permalink: /zh/newsletters/2021/11/17/
name: 2021-11-17-newsletter-zh 
slug: 2021-11-17-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 提供了关于激活 taproot 的信息，并包括我们的常规部分，总结了服务和客户端软件的变更，新版本和候选版本，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻

- **Taproot激活：** 正如预期，[taproot][topic taproot] 软分叉在区块高度 709,632 处激活。截至本文撰写时，几个大型矿池没有挖出含有 taproot 花费的区块。这可能表明，他们错误地发出了准备执行 taproot 规则的信号，我们[之前警告过][p4tr what happens]这个风险。另外，他们可能使用一个执行 taproot 的节点来选择使用哪条区块链，同时也使用一个较旧的节点或自定义软件来选择在他们的区块中包含哪些交易来避免风险。

  对于用户和企业来说，最安全的做法是运行他们自己的执行 taproot 的节点（如 Bitcoin Core 22.0），并只接受由它确认的交易。

## 服务和客户端软件的变更

*在这个月的专题中，我们会重点介绍比特币钱包和服务中有意思的更新。*
- **bitcoinj 增加了 bech32m，P2TR 支持：** Andreas Schildbach 在 bitcoinj 仓库中增加了 [bech32m][topic bech32] 的[提交][bitcoinj bech32m]和 支持 [P2TR][bitcoinj p2tr] 的提交。

- **libwally-core 增加了 bech32m 支持：** 这个钱包原语库的 [0.8.4 版本][libwally 0.8.4] 包含了 [bech32m 支持][libwally 297]。

- **Spark 闪电钱包增加了 BOLT12 要约：** Spark [v0.3.0][spark v0.3.0] 增加了 [要约][topic offers] 功能，包括要约创建、发送要约付款和拉取支付。循环要约功能计划在未来的版本中推出。

- **BitGo 钱包支持 taproot：** BitGo [宣布][bitgo taproot blog]支持使用其 API 发送至或者接受到 [taproot][topic taproot] 输出。Taproot 的 UI 支持计划在未来的更新中进行。

- **NthKey 增加 bech32m 发送功能：** iOS 签名服务 [NthKey][nthkey website] 在 [v1.0.4][nthkey v1.0.4] 版本中增加了对 taproot 发送的支持。

- **Ledger Live 支持 taproot：** Ledger 的客户端软件，Ledger Live，宣布在其 [v2.35.0][ledger v2.35.0] 版本中支持 taproot 作为一项实验性功能。

- **Muun 钱包支持 taproot：** Muun 钱包在激活发生后启用了 taproot 地址支持，包括让用户默认设置 taproot 接收地址。

- **Kollider 推出了阿尔法版本的基于 LN 的交易平台：** Kollider 的最新[公告][kollider blog]详细介绍了衍生品平台的功能，包括 LN 存款和提款，外加 LNAUTH 和 LNURL 支持。

## 新版本和候选版本

*流行的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- [LND 0.14.0-beta.rc4][] 是一个候选发布版本，包括新增的[日蚀攻击][topic eclipse attacks]保护（见 [Newsletter #164][news164 ping]），远程数据库支持（[Newsletter #157][news157 db]），更快的寻路（[Newsletter #170][news170
  path]），为 Lightning Pool 用户做的改进（[Newsletter #172][news172 pool]），以及可重复使用的 [AMP][topic amp] 账单（[Newsletter #173][news173 amp]），以及许多其他功能和错误修复。

## 重大代码和文档更新

*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Bitcoin Core #22934][] 在 ECDSA 签名和 [schnorr 签名][topic schnorr signatures]创建后都增加了一个验证步骤。这可以防止软件暴露不正确生成的签名，导致泄露用于生成签名的私钥或 nonce 的信息。这遵循了之前在 [Newsletter #83][news83 safety] 中讨论的 [BIP340][]（见 [Newsletter #87][news87 bips886]）的更新中给出的建议。

- [Bitcoin Core #23077][] 通过 [CJDNS][] 实现了地址中继，使 CJDNS 成为像 IPv4、IPv6、Tor 和 I2P 一样完全支持的网络。一旦 CJDNS 在比特币核心之外被设置，节点操作员可以切换新的配置选项 `-cjdnsreachable`，让比特币核心将 `fc00::/8` 地址解释为属于 CJDNS，而不是被解释为私人 IPv6 地址。

- [Eclair #1957][] 增加了对 [BOLTs #759][] 的[洋葱消息][topic onion messages]的基本支持，允许洋葱消息的中继，但不支持发起或接收洋葱消息。

- [Rust Bitcoin #691][] 增加了一个API，用于从 pubkey 和可选的 [tapscript][topic tapscript] merkle 根为 [P2TR][topic taproot] scriptPubKey 创建 [bech32m][topic bech32] 地址。

- [BDK #460][] 增加了一个新函数，用于在交易中添加 `OP_RETURN` 输出。

- [BIPs #1225][] 用 [Newsletter #173][news173 taproot tests] 中描述的 taproot 测试向量更新了 BIP341。



[topic taproot]: https://bitcoinops.org/en/topics/taproot/
[topic bech32]: https://bitcoinops.org/en/topics/bech32/
[topic offers]: https://bitcoinops.org/en/topics/offers/
[topic eclipse attacks]: https://bitcoinops.org/en/topics/eclipse-attacks/
[topic amp]: https://bitcoinops.org/en/topics/atomic-multipath/
[topic schnorr signatures]: https://bitcoinops.org/en/topics/schnorr-signatures/
[topic onion messages]: https://bitcoinops.org/en/topics/onion-messages/
[topic tapscript]: https://bitcoinops.org/en/topics/tapscript/
[lnd 0.14.0-beta.rc4]: https://github.com/lightningnetwork/lnd/releases/tag/v0.14.0-beta.rc4
[news164 ping]: /en/newsletters/2021/09/01/#lnd-5621
[news157 db]: /en/newsletters/2021/07/14/#lnd-5447
[news170 path]: /en/newsletters/2021/10/13/#lnd-5642
[news172 pool]: /en/newsletters/2021/10/27/#lnd-5709
[news173 amp]: /en/newsletters/2021/11/03/#lnd-5803
[news87 bips886]: /en/newsletters/2020/03/04/#bips-886
[news83 safety]: /en/newsletters/2020/02/05/#safety-concerns-related-to-precomputed-public-keys-used-with-schnorr-signatures
[news173 taproot tests]: /en/newsletters/2021/11/03/#taproot-test-vectors
[p4tr what happens]: /en/preparing-for-taproot/#what-happens-at-activation
[cjdns]: https://github.com/cjdelisle/cjdns
[bitcoinj bech32m]: https://github.com/bitcoinj/bitcoinj/pull/2099
[bitcoinj p2tr]: https://github.com/bitcoinj/bitcoinj/pull/2225
[libwally 0.8.4]: https://github.com/ElementsProject/libwally-core/releases/tag/release_0.8.4
[libwally 297]: https://github.com/ElementsProject/libwally-core/pull/297
[spark v0.3.0]: https://github.com/shesek/spark-wallet/releases/tag/v0.3.0
[bitgo taproot blog]: https://blog.bitgo.com/taproot-support-for-bitgo-wallets-9ed97f412460
[nthkey website]: https://nthkey.com/
[nthkey v1.0.4]: https://github.com/Sjors/nthkey-ios/releases/tag/v1.0.4
[ledger v2.35.0]: https://github.com/LedgerHQ/ledger-live-desktop/releases/tag/v2.35.0
[kollider blog]: https://kollider.medium.com/kollider-alpha-version-h1-3bec739df1d4
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
[Bitcoin Core #22934]: https://github.com/bitcoin/bitcoin/pull/22934
[BIP340]: https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki
[Bitcoin Core #23077]: https://github.com/bitcoin/bitcoin/issues/23077
[CJDNS]: https://github.com/cjdelisle/cjdns
[Eclair #1957]: https://github.com/ACINQ/eclair/pull/1957
[BOLTs #759]: https://github.com/lightning/bolts/issues/759
[Rust Bitcoin #691]:https://github.com/rust-bitcoin/rust-bitcoin/issues/691
[BDK #460]: https://github.com/bitcoindevkit/bdk/pull/460
[BIPs #1225]: https://github.com/bitcoin/bips/pull/1225
