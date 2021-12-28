---
title: 'Bitcoin Optech Newsletter #179'
permalink: /en/newsletters/2021/12/15/
name: 2021-12-15-newsletter
slug: 2021-12-15-newsletter
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 描述了一项关于在某些情况下允许中继零值输出交易的提案，并总结了关于准备在 LN 上采用 PTLC 的讨论。此外，还包括我们的常规部分，包括服务端和客户端软件的最新变更，Bitcoin Stack Exchange 的热门问题，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻

- **为某些不经济的输出破例：** 自我们在 [Newsletter #162][news162 unec] 中介绍后，Jeremy Rubin 在 Bitcoin-Dev 邮件列表中[重新][rubin unec]讨论了允许交易创建价值低于[尘额限制][topic uneconomical outputs]的输出。尘额限制是一个交易中继策略，节点用来阻止用户创建不经济的 UTXO 花费。UTXO 至少需要被一些全节点储存起来，直到被花费，而且在某些情况下能够被快速检索，所以允许*不经济的输出*可能会造成不必要的问题。

  然而，在 [CPFP][topic cpfp] 费用更新中，零值输出可能会有用武之地，在这种情况下，用于费用更新的交易的资金不能被花费 —— 所有用于费用更新的资金都需要来自独立的 UTXO，例如在 [eltoo][topic eltoo] 中。Ruben Somsen 还提供了一个例子，说明零值输出对 spacechains（一类单向锚定的侧链）是多么有用。

  截至本文写作时，讨论没有得出明确的结论。

- **准备在 LN 上采用 PTLC：** Bastien Teinturier 在 Lightning-Dev 邮件列表中发起了一个[主题][teinturier post]，关于对 LN 通信协议进行必要的最小[变更集合][ln docs 16]，以允许节点开始升级到使用 [PTLC][topic ptlc]。PTLC 比目前使用的 HTLC 更私密，并且使用更少的区块空间。

  Teinturier 正试图产出一套可以与被提议的 `option_simplified_update` [协议变更][bolts #867]同时进行的变更（见 [Newsletter #120][news120
    opt_simp_update]）。一个次要目标是试图使通信协议与 [Newsletter #152][news152 ff] 中描述的基于 fast-forwards 的 PTLC 协议兼容。这将使得节点可以分阶段升级，首先升级到带有 HTLC 的 `option_simplified_update`，然后升级到 PTLC，再升级到 fast-forwards。

## 服务端和客户端软件的变更
*在这个月的专题中，我们会重点介绍比特币钱包和服务中有意思的更新。*

- **Simple Bitcoin Wallet 增加 taproot 发送功能：** SBW [2.4.22][sbw 2.4.22] 版本允许用户向 taproot 地址发送交易。

- **Trezor Suite 支持 taproot：** [Trezor 宣布][trezor taproot blog] 21.12.2 版本的 Trezor Suite 支持 [taproot][topic taproot]。在下载了最新的客户端和固件后，用户可以创建一个新的 taproot 账户。

- **BlueWallet 增加了 taproot 的发送支持：** BlueWallet [v6.2.14][bluewallet 6.2.14] 增加了对 taproot 地址的发送支持。

- **Cash App 增加了 taproot 的发送支持：** 从 [2021 年 12 月 1 日][cash app bech32m]起，Cash App 用户可以向 [bech32m][topic bech32] 地址发送交易。

- **Swan 增加了 taproot 的发送支持：** Swan [宣布][swan taproot tweet]支持 taproot 提款（发送）。

- **Wallet of Satoshi 增加了 taproot 发送支持：** Wallet of Satoshi ，一个移动端比特币和闪电网络钱包，宣布支持 taproot 交易发送。

## Bitcoin Stack Exchange 问答选摘
*[Bitcoin Stack Exchange](https://bitcoin.stackexchange.com/) 是 Optech 贡献者寻找问题答案，或者当我们有一些空闲时间时帮助好奇或困惑的用户的首选地方之一。在这个月度专题中，我们将重点介绍一些自上次更新以来被投票最多的问题和答案。*

- [P2TR 花费（从 taproot 花费）中的组装和执行的脚本是什么样的？][111098] Pieter Wuille 详细介绍了一个简化的 [BIP341][] 例子来构建 taproot 输出、使用密钥路径花费、使用脚本路径花费并验证花费。

- [我怎样才能找到主网的 P2TR 交易的样本？][110995] Murch 提供了以下交易的[区块浏览器][topic block explorers]链接：第一笔 P2TR 交易，第一笔有脚本路径和密钥路径输入的交易，第一笔有多个密钥路径输入的交易，第一笔 2-of-2 多签名脚本路径花费，以及第一次使用新的 [tapscript][topic tapscript] 操作码 `OP_CHECKSIGADD` 的交易。

- [矿工在挖矿时向区块添加交易是否会重置区块的 PoW？][110903] Pieter Wuille 解释说，挖矿是[无进度][oconnor blog]的。破解区块的挖矿难题的每一次哈希碰撞的尝试都与之前做的哈希碰撞的尝试无关，包括新的交易被添加到目前正在挖的区块中的情形。

- [Schnorr 聚合签名可以嵌套在其他 schnorr 聚合签名中吗？][110862] Pieter Wuille 介绍了使用 [schnorr 签名][topic schnorr signatures]的密钥聚合方案的可行性，其中 "密钥可以分层聚合，而签名者不知道他们的'侄子/侄女'密钥"。他指出，[MuSig2][topic musig] 被设计成兼容嵌套，它可以为这种使用情况进行修改，尽管尚无安全证明。

## 代码和文档的重大变更

*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Bitcoin Core #23716][] 为 Bitcoin Core 的测试代码添加了 RIPEMD-160 的本地 Python 实现。这使得 Bitcoin Core 不再使用 Python `hashlib` 库中的 RIPEMD-160 函数，该库封装了 RIPEMD-160 的 OpenSSL 实现。新版本的 OpenSSL 不再默认提供 RIPEMD-160 支持，需要单独启用。

- [Bitcoin Core #20295][] 增加了一个新的 RPC `getblockfrompeer`，允许从一个特定的对等节点手动请求一个特定的块。其目的是以分叉监测和研究为目的来获取过时的最新区块头。

- [Bitcoin Core #14707][] 更新了几个RPC，使其也包括通过矿工铸币交易输出收到的比特币。RPC 的一个新的 `include_immature_coinbase` 选项允许在铸币交易完全被确认前（根据共识规则，至少经过 100 个确认才可以被花费）被加入。

- [Bitcoin Core #23486][] 更新了 `decodescript` RPC，只返回脚本的 P2SH 或 P2WSH 地址，如果该脚本可以分别用于 P2SH 或 P2WSH。

- [BOLTs #940][] 废弃了 `node_announcements` 中对 Tor v2 洋葱服务的公告和解析。[Rust-Lightning #1204][]，更新了遵循该更新的规范的实现，在本周也被合并。

- [BOLTs #918][] 删除了 ping 消息的速率限制。`ping` 消息主要用于检查对等节点连接是否仍然有效。在这次合并之前，`ping` 消息应该是每 30 秒最多发送一次。对于许多节点来说，通过 `ping` 更频繁地发送心跳信息以保证高质量的服务是很有用的。由于其他闪电网络的信息没有速率限制，所以 `ping` 信息的 30 秒速率限制也被废弃了。

- [BOLTs #906][] 为 [Newsletter #165][news165 channel_type] 中描述的 `channel_type` 功能增加了一个新的功能位。通过添加这个功能位，未来的节点将更容易只选择能解析新功能的对等节点。

## 假日发布时间表
节日快乐! 这一期是我们今年的最后一期常规 Newsletter。下周我们将发布我们的年度特别回顾刊。我们将于 1 月 5 日（星期三）恢复正常发布。


[Bitcoin Core #23716]: https://github.com/bitcoin/bitcoin/pull/23716
[Bitcoin Core #20295]: https://github.com/bitcoin/bitcoin/issues/20295
[Bitcoin Core #14707]: https://github.com/bitcoin/bitcoin/issues/14707
[Bitcoin Core #23486]: https://github.com/bitcoin/bitcoin/issues/23486
[BOLTs #940]: https://github.com/lightning/bolts/issues/940
[Rust-Lightning #1204]: https://github.com/lightningdevkit/rust-lightning/pull/1204
[BOLTs #918]: https://github.com/lightning/bolts/pull/918
[BOLTs #906]: https://github.com/lightning/bolts/pull/906

[news162 unec]: /en/newsletters/2021/08/18/#dust-limit-discussion
[rubin unec]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-December/019635.html
[somsen unec]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-December/019637.html
[teinturier post]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-December/003377.html
[ln docs 16]: https://github.com/t-bast/lightning-docs/pull/16
[news120 opt_simp_update]: /en/newsletters/2020/10/21/#simplified-htlc-negotiation
[news152 ff]: /en/newsletters/2021/06/09/#receiving-ln-payments-with-a-mostly-offline-private-key
[news165 channel_type]: /en/newsletters/2021/09/08/#bolts-880
[sbw 2.4.22]: https://github.com/btcontract/wallet/releases/tag/2.4.22
[bluewallet 6.2.14]: https://github.com/BlueWallet/BlueWallet/releases/tag/v6.2.14
[cash app bech32m]: https://cash.app/help/us/en-us/20211114-bitcoin-taproot-upgrade
[trezor taproot blog]: https://blog.trezor.io/trezor-suite-and-firmware-updates-december-2021-d1e74c3ea283
[swan taproot tweet]: https://twitter.com/SwanBitcoin/status/1468318386916663298
[wallet of satoshi website]: https://www.walletofsatoshi.com/
[wallet of satoshi tweet]: https://twitter.com/walletofsatoshi/status/1459782761472872451
[oconnor blog]: http://r6.ca/blog/20180225T160548Z.html
[topic uneconomical outputs]: https://bitcoinops.org/en/topics/uneconomical-outputs/
[topic cpfp]: https://bitcoinops.org/en/topics/cpfp/
[topic eltoo]: https://bitcoinops.org/en/topics/eltoo/
[topic ptlc]: https://bitcoinops.org/en/topics/ptlc/
[bolts #867]: https://github.com/lightning/bolts/pull/867
[news120 opt_simp_update]: https://bitcoinops.org/en/newsletters/2020/10/21/#simplified-htlc-negotiation
[news152 ff]: https://bitcoinops.org/en/newsletters/2021/06/09/#receiving-ln-payments-with-a-mostly-offline-private-key
[topic taproot]: https://bitcoinops.org/en/topics/taproot/
[topic bech32]: https://bitcoinops.org/en/topics/bech32/
[BIP341]: https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki
[111098]: https://bitcoin.stackexchange.com/questions/111098/what-is-the-script-assembly-and-execution-in-p2tr-spend-spend-from-taproot
[110995]: https://bitcoin.stackexchange.com/questions/111098/what-is-the-script-assembly-and-execution-in-p2tr-spend-spend-from-taproot
[topic block explorers]: https://bitcoinops.org/en/topics/block-explorers/
[topic tapscript]: https://bitcoinops.org/en/topics/tapscript/
[110903]: https://bitcoin.stackexchange.com/questions/110903/understanding-pow-and-transactions
[110862]: https://bitcoin.stackexchange.com/questions/110862/can-schnorr-aggregate-signatures-be-nested-inside-other-schnorr-aggregate-signat
[topic schnorr signatures]: https://bitcoinops.org/en/topics/schnorr-signatures/
[topic musig]: https://bitcoinops.org/en/topics/musig/

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
