---
title: 'Bitcoin Optech Newsletter #187'
permalink: /zh/newsletters/2022/02/16/
name: 2022-02-16-newsletter-zh
slug: 2022-02-16-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---

本周的周报介绍了关于比特币的限制条款（covenant）实现的继续讨论，也包含了我们的常规部分：客户端软件以及服务的变更总结；流行的比特币基础设施软件的重大变更。

## 新闻

- <a id="simplified-alternative-to-op-txhash" href="#simplified-alternative-to-op-txhash)">●</a>  ` OP_TXHASH ` 的简化替代品：在关于启用[限制条款][covenant]的操作码实现的持续讨论（见[周报 #185][Newsletter #185]）中，Rusty Russel [提出][proposed]  ` OP_TXHASH ` 提供的功能可以由现有的  ` OP_SHA256 ` 操作码加上一个新的 `OP_TX ` 操作码（接受与  ` OP_TXHASH ` 同样才输入）来实现。这个新的操作码将可以从一笔使用 [tapscript][tapscript] 的花费交易中获得序列化的字段。然后，脚本就可以直接测试这些字段（例如，交易的版本号是否大于 1？），或者哈希这些数据、并通过此前提议的 [OP_CHECKSIGFROMSTACK][OP_CHECKSIGFROMSTACK] 操作码来比较这个哈希值和一个签名。

## 客户端软件和服务的变更

在这个月度栏目中，我们会着重列出比特币钱包和服务的有趣更新。

- <a id="blockchain-com-wallet-adds-taproot-sends" href="#blockchain-com-wallet-adds-taproot-sends)">●</a> **Blockchain.com 钱包加入了给 taproot 地址支付的功能**：Blockchain.com 钱包安卓版本的 [v202201.2.0(18481)][v202201.2.0(18481)] 支持向 [bech32m][bech32m] 地址发送资金。在撰文之时，iOS 版本尚未支持向 bech32m 地址支付。
- <a id="sensei-lightning-node-implementation-launches" href="#sensei-lightning-node-implementation-launches)">●</a> **Sensei 闪电节点实现启动**：[Sensei][Sensei] 是使用 [Bitcoin Dev Kit (BDK)][BDK] 和 [Lightning Dev Kit (LDK)][LDK] 开发的软件，当前还是 beta 版。这个节点实现当前需要 Bitcoin Core 和 Electrum server，已计划增加其它后端支持。
- <a id="bitmex-adds-taproot-sends" href="#bitmex-adds-taproot-sends)">●</a> **BitMAX 增加 taproot 地址取款**：在最近的[博客][blog post]中，BitMEX 宣布了支持 bech32m 地址取款。该文还提到了，现在 73% 的 [BitMEX 用户取款][BitMEX user deposits]是用 P2WSH 输出来接收的，节约了约 65% 的手续费。
- <a id="bitbox02-adds-taproot-sends" href="#bitbox02-adds-taproot-sends)">●</a> **BitBox02 支持给 taproot 地址支付**：[v9.9.0 - Multi][v9.9.0 - Multi] 和 [v9.9.0 - Bitcoin-only][v9.9.0 - Bitcoin-only] 两个版本都加入了对发送到 bech32m 地址的支持。
- <a id="fulcrum-1-6-0-adds-performance-improvements" href="#fulcrum-1-6-0-adds-performance-improvements)">●</a> **Fulcrum 1.6.0 性能提升**：地址索引软件 Fulcrum 在 [1.6.0 版本][1.6.0 release]中加入了[性能升级][performance improvements]。
- <a id="kraken-announces-proof-of-reserves-scheme" href="#kraken-announces-proof-of-reserves-scheme)">●</a> **Karken 宣布准备金证明方案**：Karken [详细讲解][details]了他们的准备金证明方案（含有一个被信任的审计员），也提到了短板以及未来的提升方向。方案大体如下：Kraken 创建数字签名来证明链上地址的所有权、生成 Kraken 用户账户余额的默克尔树，请求一个审计员来见证链上的余额是大于账户余额总和的，并提供了工具来让用户验证自己的余额记录在了这棵树上。

## 重大的代码和文献变更

本周发生重大变更的有：[Bitcoin Core][Bitcoin Core]、[C-Lightning][C-Lightning]、[Eclair][Eclair]、[LDK][LDK]、[LND][LND]、[libsecp256k1][libsecp256k1]、[Hardware Wallet Interface (HWI)][HWI]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK]、[Bitcoin Improvement Proposals (BIPs)][BIPs] 以及 [Lightning BOLTs][Lightning BOLTs]。

- <a id="eclair-2164" href="#eclair-2164)">●</a> [Eclair #2164][Eclair #2164] 提升了它在多种情境中处理 feature bits 的功能。重点是，需要强制性但非发票特性的发票将不再被拒绝，因为缺乏对一个非发票特性的支持并不影响一个发票得到履行的能力。
- <a id="btcpay-server-3395" href="#btcpay-server-3395)">●</a> [BTCPay Server #3395][BTCPay Server #3395] 加入了对 [CPFP][CPFP] 手续费追加策略的支持，可加速收到的发票支付以及从钱包发出的交易。
- <a id="bips-1279" href="#bips-1279)">●</a> [BIPs #1279][BIPs #1279] 更新了 [BIP322][BIP322] [通用签名消息协议][generic signmessage protocol]的详述，加入了一些释义和测试变量。

[covenant]:https://bitcoinops.org/en/topics/covenants/

[Newsletter #185]:https://bitcoinops.org/en/newsletters/2022/02/02/#composable-alternatives-to-ctv-and-apo

[proposed]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019871.html

[tapscript]:https://bitcoinops.org/en/topics/tapscript/

[OP_CHECKSIGFROMSTACK]:https://bitcoinops.org/en/topics/op_checksigfromstack/
[v202201.2.0(18481)]: https://github.com/blockchain/My-Wallet-V3-Android/releases/tag/v202201.2.0(18481
[bech32m]:https://bitcoinops.org/en/topics/bech32/

[Sensei]:https://l2.technology/sensei
[BDK]: https://bitcoindevkit.org/
[LDK]: https://lightningdevkit.org/
[blog post]:https://blog.bitmex.com/bitmex-supports-sending-to-taproot-addresses/

[BitMEX user deposits]:https://bitcoinops.org/en/newsletters/2021/03/24/#bitmex-announces-bech32-support

[v9.9.0 - Multi]:https://github.com/digitalbitbox/bitbox02-firmware/releases/tag/firmware%2Fv9.9.0

[v9.9.0 - Bitcoin-only]:https://github.com/digitalbitbox/bitbox02-firmware/releases/tag/firmware-btc-only%2Fv9.9.0

[performance improvements]:https://www.sparrowwallet.com/docs/server-performance.html

[1.6.0 release]:https://github.com/cculianu/Fulcrum/releases/tag/v1.6.0

[details]:https://www.kraken.com/proof-of-reserves

[Bitcoin Core]:https://github.com/bitcoin/bitcoin

[C-Lightning]:https://github.com/ElementsProject/lightning

[Eclair]:https://github.com/ACINQ/eclair

[LDK]:https://github.com/lightningdevkit/rust-lightning

[LND]:https://github.com/lightningnetwork/lnd/

[libsecp256k1]:https://github.com/bitcoin-core/secp256k1
[HWI]:  https://github.com/bitcoin-core/HWI
[Rust Bitcoin]:https://github.com/rust-bitcoin/rust-bitcoin

[BTCPay Server]:https://github.com/btcpayserver/btcpayserver/

[BDK]:https://github.com/bitcoindevkit/bdk
[BIPs]:https://github.com/bitcoin/bips/
[Lightning BOLTs]:https://github.com/lightning/bolts

[Eclair #2164]:https://github.com/ACINQ/eclair/issues/2164

[BTCPay Server #3395]:https://github.com/btcpayserver/btcpayserver/issues/3395

[CPFP]:https://bitcoinops.org/en/topics/cpfp/

[BIPs #1279]:https://github.com/bitcoin/bips/issues/1279

[BIP322]:https://github.com/bitcoin/bips/blob/master/bip-0322.mediawiki

[generic signmessage protocol]:https://bitcoinops.org/en/topics/generic-signmessage/

