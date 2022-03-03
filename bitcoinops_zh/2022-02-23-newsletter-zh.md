---
title: 'Bitcoin Optech Newsletter #188'
permalink: /en/newsletters/2022/02/23/
name: 2022-02-23-newsletter
slug: 2022-02-23-newsletter
type: newsletter
layout: newsletter
lang: zh
---
本周的 Newsletter 总结了关于费用更新和交易费用赞助的讨论，介绍了一个更新的 LN gossip 协议的提案，以及一个测试 `OP_CHECKTEMPLATEVERIFY` 的 signet。此外，还包括我们的常规部分，包括来自 Bitcoin Stack Exchange 的问答摘选，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻

- **费用更新和交易费用赞助：** 与几周前开始的费用替换讨论不同（见 [Newsletter #186][news186 rbf]），本周 James O'Beirne [开始][obeirne bump]了关于费用更新的讨论。特别是，O'Beirne 担心正在提出的一些交易中继策略的变化会使用户和钱包开发者使用费用更新变复杂。作为一个替代方案，他希望重新考虑[交易费用赞助][topic fee sponsorship]（之前在 [Newsletter #116][news116 sponsorship] 中介绍过）。

  这些想法在邮件列表中得到了大量的讨论，许多回复中提到了实施费用赞助的挑战。

- **更新的 LN gossip 的提案：** Rusty Russell 在 Lightning-Dev 邮件列表中[发布][russell gossip]了一套新的 LN gossip 消息的详细提案，和在 [Newsletter #55][news55 gossip] 中介绍的他 2019 年的提案类似。新提案使用 [BIP340][] 风格的 [Schnorr 签名][topic schnorr signatures]和 [x-only][news72 xonly] 公钥。此外，还包括对现有的 LN gossip 协议的一些简化，该协议用于宣传用于路由的公共信道的存在。更新后的协议旨在最大限度地提高效率，特别是在与类 [erlay][topic erlay] 的基于 [minisketch][topic minisketch] 的高效 gossip 协议相结合时。

- **CTV signet：** Jeremy Rubin [发布][rubin ctv signet]了激活 [OP_CHECKTEMPLATEVERIFY][topic op_checktemplateverify] 的 [signet][topic signet] 的参数和代码。这简化了公众对被提议的操作码的试用，并使使用该代码的不同软件之间的兼容性测试变得更加容易。

## Bitcoin Stack Exchange 问答选摘
*[Bitcoin Stack Exchange](https://bitcoin.stackexchange.com/) 是 Optech 贡献者寻找问题答案，或者当我们有一些空闲时间时帮助好奇或困惑的用户的首选地方之一。在这个月度专题中，我们将重点介绍一些自上次更新以来被投票最多的问题和答案。*

- [没有交易的后补贴区块会不会包括 coinbase 交易？][Q1]Pieter Wuille 解释说，每个区块都必须有一个 coinbase 交易，由于每个交易必须包括至少一个输入和一个输出，一个没有区块奖励的后补贴区块（没有交易费和补贴）仍然需要至少一个零价值的输出。

- [创世区块为什么能包含任意的数据，如果脚本不合法？][Q2] Pieter Wuille 列出了创世区块的 coinbase "Chancellor... "文本合法的原因。首先，根据定义，[创世区块][bitcoin se 13122]是有效的。第二是 coinbase 输入脚本从未被执行。第三是对于非 Taproot 输入，执行后堆栈上有一个元素的要求只是一个策略规则，而不是一个共识规则。最后，该策略规则仅适用于输入脚本与相应的输出脚本一起执行后的最终堆栈。由于 coinbase 交易的输入没有相应的输出脚本，所以该策略并不适用。Wuille 还指出，创世区块无法花费的原因与本次讨论无关，而是涉及到原始比特币软件[没有将创世区块加入][bitcoin github genesis]其内部数据库。

- [什么是 Feeler Connection？它在什么时候被使用？][Q3]用户 vnprc 解释了比特币核心的 [feeler 连接][chaincode p2p]的目的，这是一个临时的出站连接，与默认的 8 个出站连接和 2 个 blocks-only 的出站连接是分开的。feeler connection 接用于测试 gossip 网络建议的潜在新对等节点，以及测试以前无法连接的对等节点，这些对等节点是更换节点的候选。

- [OP_RETURN 交易是否不存储在链状态数据库中？][Q4] Antoine Poinsot 指出，由于 `OP_RETURN` 输出是[不可花费的][bitcoin github unspendable]，所以它们不存储在[链状态目录][bitcoin docs data]中。

## 代码和文档的重大变更
*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Bitcoin Core #24307][] 用 `external_signer` 字段扩展了 `getwalletinfo` RPC 的结果对象。这个新字段表明钱包是否被配置为使用外部签名器，如硬件签名设备。

- [C-Lightning #5010][] 增加了一个语言绑定生成工具 `MsgGen` 和一个 Rust RPC 客户端 `cln-rpc`。`MsgGen` 解析 C-Lightning 的 JSON-RPC 模式，并生成 `cln-rpc` 使用的 Rust 绑定，以正确调用 C-Lightning 的 JSON-RPC 接口。

- [LDK #1199][] 增加了对 "幽灵节点支付 "的支持，支付可以由几个节点中的任何一个接收，这可以用于负载均衡。这需要创建带有 [BOLT11][] 路由提示的 LN 收据，提示多条路径都通向同一个不存在的（"幽灵"）节点。对于每条路径，在到幽灵节点之前的最后一跳是一个真实的节点，它知道幽灵节点用于解密和重建[无状态收据][topic stateless invoices]的密钥（见 [Newsletter #181][news181 rl1177]），允许它接受付款的 [HTLC][topic htlc]。

  ![幽灵节点路径提示示例](./image/2022-02-23-newsletter-zh-p1.png)



[topic fee sponsorship]: https://bitcoinops.org/en/topics/fee-sponsorship/
[topic schnorr signatures]: https://bitcoinops.org/en/topics/schnorr-signatures/
[topic erlay]: https://bitcoinops.org/en/topics/erlay/
[topic minisketch]: https://bitcoinops.org/en/topics/minisketch/
[topic op_checktemplateverify]: https://bitcoinops.org/en/topics/op_checktemplateverify/
[topic signet]: https://bitcoinops.org/en/topics/signet/
[topic stateless invoices]: https://bitcoinops.org/en/topics/stateless-invoices/
[topic htlc]: https://bitcoinops.org/en/topics/htlc/

[news186 rbf]: https://bitcoinops.org/en/newsletters/2022/02/09/#discussion-about-rbf-policy
[obeirne bump]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019879.html
[news116 sponsorship]: https://bitcoinops.org/en/newsletters/2020/09/23/#transaction-fee-sponsorship
[russell gossip]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2022-February/003470.html
[news72 xonly]: https://bitcoinops.org/en/newsletters/2019/11/13/#x-only-pubkeys
[news55 gossip]: https://bitcoinops.org/en/newsletters/2019/07/17/#gossip-update-proposal
[rubin ctv signet]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019925.html
[news181 rl1177]: https://bitcoinops.org/en/newsletters/2022/01/05/#rust-lightning-1177
[bitcoin se 13122]: https://bitcoin.stackexchange.com/a/13123/87121
[bitcoin github genesis]: https://github.com/bitcoin/bitcoin/blob/9546a977d354b2ec6cd8455538e68fe4ba343a44/src/main.cpp#L1668
[chaincode p2p]: https://residency.chaincode.com/presentations/bitcoin/ethan_heilman_p2p.pdf#page=18
[bitcoin github unspendable]: https://github.com/bitcoin/bitcoin/blob/280a7777d3a368101d667a80ebc536e95abb2f8c/src/script/script.h#L539-L547
[bitcoin docs data]: https://github.com/bitcoin/bitcoin/blob/master/doc/files.md#data-directory-layout
[BIP340]: https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki
[Q1]: https://bitcoin.stackexchange.com/questions/112193/will-a-post-subsidy-block-with-no-transactions-include-a-coinbase-transaction
[Q2]: https://bitcoin.stackexchange.com/questions/112439/how-can-the-genesis-block-contain-arbitrary-data-on-it-if-the-script-is-invalid
[Q3]: https://bitcoin.stackexchange.com/questions/112247/what-is-a-feeler-connection-when-is-it-used
[Q4]: https://bitcoin.stackexchange.com/questions/112312/are-op-return-transactions-not-stored-in-chainstate-database
[Bitcoin Core #24307]: https://github.com/bitcoin/bitcoin/pull/24307
[C-Lightning #5010]: https://github.com/ElementsProject/lightning/issues/5010
[LDK #1199]: https://github.com/lightningdevkit/rust-lightning/issues/1199
[BOLT11]: https://github.com/lightning/bolts/blob/master/11-payment-encoding.md


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
