---
title: 'Bitcoin Optech Newsletter #183'
permalink: /en/newsletters/2022/01/19/
name: 2022-01-19-newsletter
slug: 2022-01-19-newsletter
type: newsletter
layout: newsletter
lang: zh
---



本周的周报分享了一个为比特币开发者设立一个新的法律辩护基金的公告，并总结了近期对 `OP_CHECKTEMPLATEVERIFY` 软分叉提议的讨论。其它内容包括我们的常规部分：服务和客户端软件近期的变更，以及比特币流行基础设施软件重大变更的总结。

## 新闻

- <a id="bitcoin-and-ln-legal-defense-fund" href="#bitcoin-and-ln-legal-defense-fund)">●</a> **比特币和闪电网络法律辩护基金**：Jack Dorsey、Alex Morcos 和 Martin White 在 Bitcoin-Dev 邮件组中[发文][posted]宣布要为开发比特币、闪电网络和相关可急的开发者设立一个法律辩护基金。这个基金 “将是一个非营利的实体，致力于尽可能减少阻碍开发者主动开发比特币和相关项目的法律障碍”。

- <a id="op-checktemplateverify-discussion" href="#op-checktemplateverify-discussion)">●</a> **OP_CHECKTEMPLATEVERIFY 讨论**：本周内，为比特币加入 [OP_CHECKTEMPLATEVERIFY][OP_CHECKTEMPLATEVERIFY] （CTV）操作码的软分叉提议在 Bitcoin-Dev 邮件组和 IRC 频道中都得到了讨论。

  - <a id="mailing-list-discussion" href="#mailing-list-discussion)">●</a>  *Bitcoin-Dev 邮件组讨论*：Peter Todd [撰文][posted]便是了对该提议的几项担忧，包括：它的好处并非几乎所有比特币用户都能享受（而他声称之前所有的增加功能的软分叉都做到了这一点）、它可能会创造新的拒绝服务式攻击界面、提议可使用 CTV 的应用场景是模糊的，而且（可能）因为过于复杂而不可能在显示中大规模部署。

    CTV 作者 Jeremy Rubin [引用][referenced]更新后的代码和强化后的稳定，表示许多对 DoS 攻击的担忧可以得到解决。他也指出，至少两个钱包（其中一个是广泛使用的）计划使用 CTV 提供的至少一个功能。截至本期通讯截稿，尚不清楚 Rubin 的回复是否完全解决了 Peter Todd 的担忧。

  - <a id="irc-meeting" href="#irc-meeting)">●</a> *IRC 频道讨论*：如 [181 期周报][Newsletter #181]所宣布的，Rubin 也主持了 CTV 系列讨论的第一期。Rubin 提供了一份[总结][summary]，作为[该期的会议日志][meeting log]。会议的多位参议者显然支持这个提议，但也有一些人表达了技术上的怀疑，至少在一定程度上与 Peter Todd 更早的时候发布的邮件一致。

    下一次会议计划更详细地探讨 CTV 的一些应用，可能会帮助研究它是否能在实质上提供使大部分比特币用户受益的、有说服力的案例。

## 服务和客户端软件的变更

*在这个月的栏目中，我们突出比特币钱包和服务的有趣升级*。

- <a id="cash-app-adds-lightning-support" href="#cash-app-adds-lightning-support)">●</a> **Cash App 开始支持闪电网络**：Cash App 加入了使用闪电网络来发送支付的功能。
- <a id="lnp-node-opens-first-mainnet-channel" href="#lnp-node-opens-first-mainnet-channel)">●</a> **LNP 节点开启第一个主网通道**：新的闪电王烈节点软件 [LNP Node][LNP Node] 开启了[第一个闪电网络通道][first LN channel]。LNP 节点是使用 Rust 语言编写的，支持闪电网络以及集体名为 “Bifrost” 的插件。这个插件可以支持未来闪电网络的升级，以及闪电网络上的增量协议。
- <a id="samourai-adds-taproot-support" href="#samourai-adds-taproot-support)">●</a> **Samourai 加入了 taproot 支持**：Samourai [v0.99.98][v0.99.98] 和 Samourai [Dojo v1.13.0][Dojo v1.13.0] 版本（使用了 [bitcoinjs-lib][bitcoinjs-lib] 库）支持 P2TR 的 [bech32m][bech32m] 地址。
- <a id="block-explorer-mempool-v2-3-0-released" href="#block-explorer-mempool-v2-3-0-released)">●</a> **区块浏览器 Mempool v2.3.0 放出**：Mempool [v2.3.0][v2.3.0] 以及 [mempool.space][mempool.space] 网页加入了版本和 locktime 数据，一个十六进制形式交易的广播功能、一个 “为花费 Taproot 输出的交易而设的标签”，以及其它升级。

## 重大代码和文档变更

本周出现重大变更的有 [Bitcoin Core][Bitcoin Core]、[C-Lightning][C-Lightning]、[Eclair][Eclair]、[LDK][LDK]、[LND][LND]、[libsecp256k1][libsecp256k1]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK] 以及 [Lightning BOLTs][Lightning BOLTs]。

- <a id="eclair-2063" href="#eclair-2063)">●</a> [Eclair #2063][Eclair #2063] 加入了对 [BOLTs #912][BOLTs #912] 添加到闪电网络协议中的新的 `option_payment_metadata` 发票字段的支持（亦见 [182 期周报][Newsletter #182]），现在允许 Eclair 创建的发票包含加密的支付元数据。理解这个新字段的支付者将在他们的交易加入这部分负载并在网络中路由，Eclair 可以解密这些数据并使用它来重构接受这笔支付的所有必要信息。未来如果所有的支付者都支持这个功能，它将使 “[无状态的发票][stateless invoices]” 成为可能，而且运行 Eclair 可以无需在数据库中存储任何实质发票数据，直到收到支付；这就消除了存储上的浪费、防止了创建发票的拒绝式服务攻击。
- <a id="ldk-1013" href="#ldk-1013)">●</a> [LDK #1013][LDK #1013] 加入了对创建和处理警告消息的支持（警告消息由 [BOLTs #950][BOLTs #950] 引入，见 [182 期周报][Newsletter #182]）。
- <a id="lnd-6006" href="#lnd-6006)">●</a> [LND #6006][LND #6006] 抛弃了 LND 需要连入一个全节点或者一个 Neutrino 轻节点的要求，如果用户只是想拿它来签名交易的话。这使得 LND 的签名部分可以高效运行在一个并不直接连入互联网的计算机上。
- <a id="rust-bitcoin-590" href="#rust-bitcoin-590)">●</a> [Rust Bitcoin #590][Rust Bitcoin #590] 创建了一个对 API 影响重大的变更，它简化了在 ECDSA 签名和 [schnorr 签名][schnorr signatures]中使用同一个[层级确定式密钥材料][HD key material]的用法（注意：应用应该为不同的签名算法使用不同的 HD 密钥路径，见 [157 期周报][Newsletter #157]）。[Rust Bitcoin #591][Rust Bitcoin #591] 推进了破坏式变更未影响部分的工作。
- <a id="rust-bitcoin-669" href="#rust-bitcoin-669)">●</a> [Rust Bitcoin #669][Rust Bitcoin #669] 延展了其 [PSBT][PSBT] 代码来加入一种数据类型，以保存关于一个碎片签名（使交易有效的必要但不充分的签名）的信息。以前，签名是作为原始数据而存储的，但新的数据可以协助对碎片签名执行额外的操作。这个 PR 的讨论包含了一些[有趣的评论][interesting comments]，关于不想签名的签名者是否应该在 PSBT 中放置一个空的字节向量（[“nulldummy”][“nulldummy”]）。

[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019741.html

[OP_CHECKTEMPLATEVERIFY]:https://bitcoinops.org/en/topics/op_checktemplateverify/

[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019738.html

[referenced]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019739.html

[Newsletter #181]:https://bitcoinops.org/en/newsletters/2022/01/05/#bip119-ctv-review-workshops

[meeting log]:https://gnusha.org/ctv-bip-review/2022-01-11.log

[summary]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019744.html

[LNP Node]:https://github.com/LNP-BP/lnp-node

[first LN channel]:https://twitter.com/dr_orlovsky/status/1473768786750750733

[v0.99.98]:https://docs.samourai.io/en/wallet/releases#v09998

[Dojo v1.13.0]:https://code.samourai.io/dojo/samourai-dojo/-/blob/develop/RELEASES.md#samourai-dojo-v1130

[bitcoinjs-lib]:https://github.com/bitcoinjs/bitcoinjs-lib

[bech32m]:https://bitcoinops.org/en/topics/bech32/

[v2.3.0]:https://github.com/mempool/mempool/releases/tag/v2.3.0

[mempool.space]:https://mempool.space/

[Bitcoin Core]:https://github.com/bitcoin/bitcoin

[C-Lightning]:https://github.com/ElementsProject/lightning

[Eclair]:https://github.com/ACINQ/eclair

[LDK]:https://github.com/lightningdevkit/rust-lightning

[LND]:https://github.com/lightningnetwork/lnd/

[libsecp256k1]:https://github.com/bitcoin-core/secp256k1

[Rust Bitcoin]:https://github.com/rust-bitcoin/rust-bitcoin

[BTCPay Server]:https://github.com/btcpayserver/btcpayserver/

[BDK]:https://github.com/bitcoindevkit/bdk

[Lightning BOLTs]:https://github.com/lightning/bolts

[Eclair #2063]:https://github.com/ACINQ/eclair/issues/2063

[BOLTs #912]:https://github.com/lightning/bolts/issues/912

[Newsletter #182]:https://bitcoinops.org/en/newsletters/2022/01/12/#bolts-912

[stateless invoices]:https://bitcoinops.org/en/topics/stateless-invoices/

[LDK #1013]:https://github.com/lightningdevkit/rust-lightning/issues/1013

[BOLTs #950]:https://github.com/lightning/bolts/issues/950

[Newsletter #182]:https://bitcoinops.org/en/newsletters/2022/01/12/#bolts-950

[LND #6006]:https://github.com/lightningnetwork/lnd/issues/6006

[Rust Bitcoin #590]:https://github.com/rust-bitcoin/rust-bitcoin/issues/590

[HD key material]:https://bitcoinops.org/en/topics/hd-key-generation/

[schnorr signatures]:https://bitcoinops.org/en/topics/schnorr-signatures/

[Newsletter #157]:https://bitcoinops.org/en/newsletters/2021/07/14/#use-a-new-bip32-key-derivation-path

[Rust Bitcoin #591]:https://github.com/rust-bitcoin/rust-bitcoin/issues/591

[Rust Bitcoin #669]:https://github.com/rust-bitcoin/rust-bitcoin/issues/669

[PSBT]:https://bitcoinops.org/en/topics/psbt/

[interesting comments]:https://github.com/rust-bitcoin/rust-bitcoin/pull/669#issuecomment-1008021007

[“nulldummy”]:https://github.com/bitcoin/bips/blob/master/bip-0147.mediawiki

