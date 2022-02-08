---
title: 'Bitcoin Optech Newsletter #184'
permalink: /en/newsletters/2022/01/26/
name: 2022-01-26-newsletter
slug: 2022-01-26-newsletter
type: newsletter
layout: newsletter
lang: zh
---
本周的 Newsletter 介绍了一个扩展 PSBT 的提案，其中包括使用支付给合约协议构建的花费输出字段，同时还包括我们的常规部分，其中有来自 Bitcoin Stack Exchange 热门帖子的摘要和主流的比特币基础设施软件中值得注意的变更。

## 新闻
- **P2C 字段的 PSBT 扩展：** Maxim Orlovsky [提出][orlovsky p2c]了一个新的 BIP，为 [PSBT][topic psbt] 增加可选的字段，用于从使用 [Pay-to-Contract][topic p2c]（P2C）协议创建的输出中花费，正如之前在 [Newsletter #37][news37 psbt p2c] 中提到的。P2C 允许支出方和接收方就合约文本（或其他任何东西）达成一致，然后创建一个关于该文本的公钥的承诺。付款人随后可以证明，付款人承诺了该文本，并且如果没有收款者的合作，该承诺在计算上是不可行的。简而言之，付款人可以向法院或公众证明他们所支付的内容。

  然而，为了让收款者随后构建一个花费他们收到的资金的签名，除了他们使用的密钥（该密钥通常是硬化衍生密钥链的一部分）之外，他们还需要合约的哈希值。Orlovsky 的提案允许将该哈希值添加到 PSBT 中，以便签名钱包或硬件设备能够产生有效的签名。

## Bitcoin Stack Exchange 问答选摘
*[Bitcoin Stack Exchange](https://bitcoin.stackexchange.com/) 是 Optech 贡献者寻找问题答案，或者当我们有一些空闲时间时帮助好奇或困惑的用户的首选地方之一。在这个月度专题中，我们将重点介绍一些自上次更新以来被投票最多的问题和答案。*

- [是否可以将 taproot 根地址转换成 v0 的 native segwit 地址？][Q1]在一个交易所由于缺乏 taproot 支持而将用户的 P2TR（native segwit v1）taproot 提款地址改为P2WSH（原生segwit v0）地址后，用户问是否有办法在产生的 v0 输出中提取比特币。Pieter Wuille 指出，这些比特币是无法找回的，因为用户需要找到一个脚本，其哈希值对应于 P2TR 地址的公钥，这是一个计算上不可行的操作。

- [比特币 0.3.7 是一个硬分叉吗？][Q2]用户 BA20D731B5806B1D 想知道是什么导致比特币的 0.3.7 版本被归类为硬分叉。Antoine Poinsot 给出了 `scriptPubKey` 和 `scriptSig` 值的例子，说明在 [0.3.7][bitcoin 0.3.7 github] 对 `scriptSig` + `scriptPubKey` 分开计算的 bug 修复后，以前无效的签名可以有效。

- [什么是签名研磨？][Q3] Murch 解释说，ECDSA 的签名研磨是一个反复签名的过程，直到你得到一个在下半区 r 值的签名，根据比特币用于 ECDSA 的序列化格式，会产生一个小 1 字节的签名（32 字节对 33 字节）。这种较小的签名会有较低的费用，而且签名是已知的 32 字节大小的事实有助于更准确的费用估计。

- [如何避免链间的网络冲突？][Q4] Murch 解释了节点如何使用 P2P [消息结构][wiki message structure]中规定的奇异数来识别它们的对等节点是否连接到同一网络（mainnet、testnet、signet）。

- [2021 年标准客户端中采用了多少个 BIP？][Q5] Pieter Wuille 贴出了比特币核心的 [BIP 文档][bitcoin bips doc]链接，其中记录了在比特币核心中实现的 BIP。

## 代码和文档的重大变更

*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Eclair #2134][] 默认启用了[锚定输出][topic anchor outputs]，允许承诺交易在广播时费率过低的情况下提升费率。由于锚定输出的费用提升是通过 [CPFP][topic cpfp] 实现的，用户需要在他们的 `bitcoind` 钱包中保持 UTXO 可访问。

- [Eclair #2113][] 增加了自动管理费用提升的功能。这包括按照确认时间的重要性对交易进行分类，在每个区块产生后重新评估交易以确定是否适合提升其费用，也重新评估当前的网络费率以应对交易的费率需要增加的情形，并在必要时为交易增加输入以增加交易的费率。PR 还呼吁[改进][Bitcoin Core #23201]比特币核心的钱包 API，以减少 Eclair 等外部程序所需的附加钱包管理的需求。

- [Eclair #2133][] 开始默认转发[洋葱消息][topic onion messages]。在[Newsletter #181][news181 onion] 中提到的速率限制是用来防止滥用 LN 协议中的这一实验性部分。

- [BTCPay Server #3083][] 允许管理员使用 [LNURL 认证][LNURL authentication]登录 BTCPay 实例（这也可以在非 LN 软件中实现）。

- [BIPs #1270][] 明确了 [PSBT][topic psbt] 规范中关于签名字段的可接受值。在 Rust Bitcoin 最近的一次更新[引入了更严格的签名字段解析][news183 rust-btc psbt]后，人们讨论了 PSBT 中的签名字段是否可以放置占位符，或者只允许有效签名。最后决定， PSBT 应该只包含有效的签名。

- [BOLTs #917][] 扩展了 [BOLT1][] 定义的 `init` 消息，使节点能够识别连接的对等节点使用的 IPv4 或 IPv6 地址。由于 [NAT][] 下的对等节点不能看到他们自己的 IP 地址，这允许对等节点在地址改变时更新它向网络公布的 IP 地址。


[topic psbt]: https://bitcoinops.org/en/topics/psbt/
[topic p2c]: https://bitcoinops.org/en/topics/pay-to-contract-outputs/
[topic anchor outputs]: https://bitcoinops.org/en/topics/anchor-outputs/
[topic cpfp]: https://bitcoinops.org/en/topics/cpfp/
[topic onion messages]: https://bitcoinops.org/en/topics/onion-messages/

[orlovsky p2c]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019761.html
[news181 onion]: /en/newsletters/2022/01/05/#eclair-2099
[lnurl authentication]: https://github.com/fiatjaf/lnurl-rfc/blob/legacy/lnurl-auth.md
[nat]: https://en.wikipedia.org/wiki/Network_address_translation
[news37 psbt p2c]: /en/newsletters/2019/03/12/#extension-fields-to-partially-signed-bitcoin-transactions-psbts
[bitcoin 0.3.7 github]: https://github.com/bitcoin/bitcoin/commit/6ff5f718b6a67797b2b3bab8905d607ad216ee21#diff-8458adcedc17d046942185cb709ff5c3L1135
[wiki message structure]: https://en.bitcoin.it/wiki/Protocol_documentation#Message_structure
[bitcoin bips doc]: https://github.com/bitcoin/bitcoin/blob/master/doc/bips.md
[news183 rust-btc psbt]:/en/newsletters/2022/01/19/#rust-bitcoin-669
[BOLT1]: https://github.com/lightning/bolts/blob/master/01-messaging.md

[Q1]:https://bitcoin.stackexchange.com/questions/111440/is-it-possible-to-convert-a-taproot-address-into-a-native-segwit-address
[Q2]:https://bitcoin.stackexchange.com/questions/111673/was-bitcoin-0-3-7-actually-hard-forking
[Q3]:https://bitcoin.stackexchange.com/questions/111660/what-is-signature-grinding
[Q4]:https://bitcoin.stackexchange.com/questions/111967/how-is-network-conflict-avoided-between-chains
[Q5]:https://bitcoin.stackexchange.com/questions/111901/how-many-bips-were-adopted-in-the-standard-client-in-2021
[Eclair #2134]: https://github.com/ACINQ/eclair/pull/2134
[Eclair #2113]: https://github.com/ACINQ/eclair/issues/2113
[Bitcoin Core #23201]: https://github.com/bitcoin/bitcoin/issues/23201
[Eclair #2133]: https://github.com/ACINQ/eclair/pull/2133
[BTCPay Server #3083]: https://github.com/btcpayserver/btcpayserver/issues/3083
[BIPs #1270]: https://github.com/bitcoin/bips/pull/1270
[BOLTs #917]: https://github.com/lightning/bolts/issues/917

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
