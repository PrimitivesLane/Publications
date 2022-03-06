---
title: 'Bitcoin Optech Newsletter #186'
permalink: /zh/newsletters/2022/02/09/
name: 2022-02-09-newsletter-zh
slug: 2022-02-09-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 介绍了关于费用替换交易的中继政策的讨论，并包括我们的常规部分，包括比特币核心 PR 审核俱乐部会议的总结，新版本和候选版本的公告，以及主流的比特币基础设施软件中值得注意的变更。

## News
- **关于 RBF 策略的讨论：** Gloria Zhao 在 Bitcoin-Dev 邮件列表中发起了关于费用替换（[RBF][]）策略的[讨论][zhao rbf]。她的邮件提供了当前策略的背景，列举了数年来发现的一些问题（如[钉死攻击][topic transaction pinning]），研究了该政策如何影响钱包的用户界面，然后描述了几个可能的改进。基于下一个区块模板（即矿工在试图产生工作量证明时创建并计划提交的区块）来考虑交易的改进思路被给予了极大的关注。通过评估替换对下一个区块模板的影响，就有可能在不使用启发式方法的情况下，确定它是否会给下一个区块的矿工带来更多的费用收入。一些开发者在回复中对 Gloria Zhao 的总结和她的建议做出了回复，包括可以采用的额外或替代性的修改建议。

  在写这个总结的时候，讨论还在进行。

## Bitcoin Core PR 审核俱乐部会议

*在这个每月一次的栏目中，我们总结最近一次的 [Bitcoin Core PR 审核俱乐部会议][Bitcoin Core PR Review Club]，集中展示一些重要的问题和答案。点击具体的问题可看到会议答案的总结。*

[Add usage examples][reviews 748] 是 Elichai Turkel 的一个 PR，用于添加 ECDSA 签名、[schnorr 签名][topic schnorr signatures]和 ECDH 密钥交换的使用实例。这是针对 libsecp256k1 PR 的第一次审核俱乐部会议。与会者讨论了好的随机源的重要性，通过实例进行了讨论，并提出了关于 libsecp256k1 的常规问题。

<details><summary>为什么例子中会展示如何获得随机性？
</summary>
本库中许多密码方案的安全性都依赖于秘密的密钥、一次性随机数和盐的机密性/随机性。如果攻击者能够猜测或影响我们的随机源所返回的值，他们可能会伪造签名，从我们试图保密的内容中获取信息，猜测密钥等。因此，实现一个加密方案的挑战往往在于获得随机性。例子也强调了这一事实。<a href="https://bitcoincore.reviews/libsecp256k1-748#l-99">➚</a>
</details>

<details><summary>对如何获得随机性提出建议是个好主意吗？
</summary>
libsecp256k1 的主要用户，比特币核心，有自己的随机性算法，其中组合了操作系统、p2p 网络上收到的消息以及其他熵的来源。对于其他必须 "自带熵 "的用户来说，建议可能对其有帮助，因为一个好的随机性来源是非常关键的，而操作系统的文档并不总是清晰的。这些建议存在着维护的负担，因为它们可能会由于操作系统的支持和漏洞而变得过时，但由于这些 API 变化的频率很低，所以预计维护的负担会很小。<a href="https://bitcoincore.reviews/libsecp256k1-748#l-120">➚</a>
</details>

<details><summary>你能实现 PR 中添加的例子吗？其中有什么遗漏吗？
</summary>
与会者讨论了他们关于以下事物的经验，包括编译、运行示例，使用调试器，将示例代码与比特币核心的使用进行比较，以及考虑非 Bitcoin 用户的用户体验。一位与会者指出，在产生 schnorr 签名后不对其进行验证是对比特币核心代码和 <a href="https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki">BIP340</a> 建议的一种偏离。另一位与会者建议在 <code>secp256k1_ecdsa_sign</code> 之前演示 <code>secp256k1_sha256</code> 的用法，因为忘记对信息进行哈希处理对用户来说可能是一个潜在的坑。<a href="https://bitcoincore.reviews/libsecp256k1-748#l-193">➚</a>
</details>

<details><summary>如果用户忘记做一些事情，比如在签名后验证签名，调用 <code>seckey_verify</code>，或者随机化上下文，会发生什么？
</summary>
在最坏的情况下，如果实现中存在缺陷，忘记在签名后验证签名可能意味着意外地给出了一个无效的签名。在随机生成密钥后忘记调用 <code>seckey_verify</code> 意味着存在（可以忽略不计的）无效密钥的概率。随机化上下文的目的是为了防止侧信道攻击——它掩盖了对最终结果没有影响的中间值，但可能被利用来获取执行的操作的信息。<a href="https://bitcoincore.reviews/libsecp256k1-748#l-226">➚</a>
</details>


## 新版本和候选版本

*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

 - [LND 0.14.2-beta][] 是一个维护版本的发布，包括几个错误的修复和一些小的改进。

## 代码和文档的重大变更
*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Bitcoin Core #23508][] 将软分叉部署的状态从 `getblockchaininfo` 转移到一个新的 `getdeploymentinfo` RPC。新的 RPC 还可以查询特定区块高度的部署状态，而不是仅仅是查询最新区块。

- [Bitcoin Core #21851][] 增加了对 arm64-apple-darwin (Apple M1) 的构建支持。随着这些变更的合并，社区可以期待在下一个版本中使用 M1 二进制文件。

- [Bitcoin Core #16795][] 更新了 `getrawtransaction`、`gettxout`、`decoderawtransaction` 和 `decodescript` RPC，以便为任何被解码的 scriptPubKey 返回推断的[输出脚本描述符][topic descriptors]。

- [LND #6226][] 将 5% 作为使用传统的 `SendPayment`、`SendPaymentSync` 和 `QueryRoutes` RPC 创建的通过 LN 路由的付款的默认费用。使用较新的 `SendPaymentV2` RPC 发送的付款默认为零费用，会要求用户指定一个值。另外一个合并的 PR，[LND #6234][]，对于使用传统的 RPC 进行的少于 1000 聪的支付，默认支付 100% 的费用。

- [LND #6177][] 允许 [HTLC][topic HTLC] 拦截器的用户定义 HTLC 失败的原因，使拦截器在测试失败如何影响使用 LND 的软件时更加有用。

- [LDK #1227][] 改进了寻路逻辑，以考虑已知的历史支付失败/成功的情况。这些失败/成功的案例被用来确定通道余额的上下限，这使得寻路逻辑在评估路径时有更准确的成功概率。这是以前 René Pickhardt 和其他人描述的一些想法的实现，这些想法在以前的 Newsletter 中提到过（包括 [#142][news142 pps]、[#163][news163 pickhardt richter paper] 和 [#172][news172 cl4771]）。

- [HWI #549][] 增加了对 [BIP370][] 中定义的 [PSBT][topic psbt] 第二版的支持。当使用原生支持零版本 PSBT 的设备时，如现有的 Coldcard 硬件签名设备，v2 版本的 PSBT 将被翻译成 v0 版本的 PSBT。

- [HWI #544][] 增加了对使用 Trezor 硬件签名设备接收和花费 [taproot][topic taproot] 付款的支持。


[topic transaction pinning]: https://bitcoinops.org/en/topics/transaction-pinning/
[topic schnorr signatures]: https://bitcoinops.org/en/topics/schnorr-signatures/
[topic descriptors]: https://bitcoinops.org/en/topics/output-script-descriptors/
[topic HTLC]: https://github.com/lightningnetwork/lnd/issues/6177
[topic psbt]: https://bitcoinops.org/en/topics/psbt/
[topic taproot]: https://bitcoinops.org/en/topics/taproot/

[lnd 0.14.2-beta]: https://github.com/lightningnetwork/lnd/releases/tag/v0.14.2-beta
[zhao rbf]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019817.html
[news163 pickhardt richter paper]: /en/newsletters/2021/08/25/#zero-base-fee-ln-discussion
[news142 pps]: /en/newsletters/2021/03/31/#paper-on-probabilistic-path-selection
[news172 cl4771]: /en/newsletters/2021/10/27/#c-lightning-4771
[reviews 748]: https://bitcoincore.reviews/libsecp256k1-748
[Bitcoin Core PR Review Club]:https://bitcoincore.reviews/
[Bitcoin Core #23508]: https://github.com/bitcoin/bitcoin/issues/23508
[Bitcoin Core #21851]: https://github.com/bitcoin/bitcoin/issues/21851
[Bitcoin Core #16795]: https://github.com/bitcoin/bitcoin/issues/16795
[LND #6226]: https://github.com/lightningnetwork/lnd/pull/6226
[LND #6234]: https://github.com/lightningnetwork/lnd/pull/6234
[LND #6177]: https://github.com/lightningnetwork/lnd/pull/6177
[LDK #1227]: https://github.com/lightningdevkit/rust-lightning/issues/1227
[HWI #549]: https://github.com/bitcoin-core/HWI/pull/549
[BIP370]: https://github.com/bitcoin/bips/blob/master/bip-0370.mediawiki
[HWI #544]: https://github.com/bitcoin-core/HWI/issues/544
[RBF]: https://bitcoinops.org/en/topics/replace-by-fee/

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
