---
title: 'Bitcoin Optech Newsletter #189'
permalink: /en/newsletters/2022/03/02/
name: 2022-03-02-newsletter
slug: 2022-03-02-newsletter
type: newsletter
layout: newsletter
lang: zh
---

本周的周报介绍了一个新提议的 `OP_EVICT ` 操作码，还包含了我们的常规部分：软件新版本和候选版本的最般配，以及热门比特币基础设施软件的重大变更。

## 新闻

- 为简化 UTXO 所有权共享而提出的操作码：开发者 ZmnSCPxj 在 Bitcoin-Dev 邮件组中[发帖][posted]提出了一种 `OP_EVICT ` 操作码，作为[此前提议的][previously proposed]  ` OP_TAPLEAF_UPDATE_VERIFY `（TLUV）操作码的竞争方案。与 TLUV 一样，`OP_EVICT ` 操作码也致力于超过两名用户共享单个 UTXO 所有权的应用场景，比如 “[joinpools][joinpools]”、“[通道工厂][channel factories]”，以及特定的[限制条款合约][covenants]。想要理解 `OP_EVICT ` 是怎么工作的，想象一个 joinpool：一个 UTXO 由 4 名用户（Alice、Bob、Carol 和 Dan）一起控制。

  当前，这 4 名用户可以创建一个 P2TR（taproot）输出，这个输出的密钥路径允许他们使用一套类似于 [MuSig2][MuSig2] 这样的协议来高效地花费这个输出，只要他们所有人都愿意参与生成一条签名。但是，如果 Dan 这时候不在线或怀有恶意，那么 Alice、Bob 和 Carol 想要保留 joinpool 所具有的隐私和效率优势的唯一方法，就是提前跟 Dan 准备好一棵预签名的交易所构成的树 —— 树上的预签名交易并不都需要用到，但为了保证完全容错，每一笔交易都不可或缺。

  ![](./Image/2022-03-combinatorial-txes.dot.png)

  随着共享一个 UTXO 的用户数量的增加，需要创建的预签名交易的数量会组合式上升（increase combinatorially），使得这种安排完全不可扩展（10 个用户就需要预先准备好 100 万笔交易）。其它被提议的操作码如 TLUL 和  [OP_CHECKTEMPLATEVERIFY][OP_CHECKTEMPLATEVERIFY] 可以消除这种爆炸式增长（ combinatorial explosion）。` OP_EVICT ` 也有同样的作用，但 ZmnSCPxj 声称它是一个更优的选择（针对这个应用场景），因为在移除共享UTXO 所有权的群组成员时，它只需更少的链上数据。

  如果 ` OP_EVICT ` 被纳入一个软分叉中，群组的每个成员都可以跟其它成员分享一把公钥以及该公钥的一个签名，让这个输出给该成员支付预期的数额（例如，向  Alice 支付 1 BTC、向 Bob 支付 2 BTC，等等）。一旦每个成员都拥有了所有其他成员的公钥和签名，他们就能免信任地构造出一个地址，并使用下述两种方式之一来花费其中的资金：

  1. 使用 taproot 密钥路径来花费，如上文所述。
  2. 使用脚本路径，调用带有 ` OP_EVICT `  的 [tapscript][tapscript] 来花费。

  在驱逐 Dan 这个案例中， ` OP_EVICT ` 可以接受下列参数：

  - **共享的公钥**：整个群组共享的公钥，可以使用对一个模板的单字节引用来高效提供
  - **被驱逐者的数量**：要创建的 joinpool 退出输出的数量（这个案例中就是一个）
  - **退群输出**：在这个案例中就是给 Dan 的一个输出，这个数据会提供其索引为之以及 Dan 对此的签名。Dan 的公钥与他用来签名输出的公钥相同
  - **未退群者的签名**：对应于整个群组的公钥在减去退群输出所用公钥之后的公钥的签名。换句话来说，是剩余成员的签名（在这个案例中就是 Alice、Bob 和 Carol）

  这使得 Alice、Bob 和 Carol 可以随时花费这个团体 UTXO 而无需 Dan 的合作，因为可以使用 Dan 提前签名的输出、Dan 的相应签名、Alice 等三人实时创建的对整个花费交易的签名来创建一笔交易（这笔交易也将支付相应的交易费并按他们的意愿分配剩余的资金）。

  截至本通讯发稿，` OP_EVICT ` 在邮件组中收到了一定数量的讨论，没有值得一提的重大担忧，但也跟 TLUV 提议在去年的情形一样，没有特别积极的表示。

## 新版本和候选版本

热门比特币基础设施项目软件的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。

- [BTCPay Server 1.4.6][BTCPay Server 1.4.6] 是这个支付处理软件的最新版本。自 Optech 介绍的上一个版本一来，该软件已经加入了 [CPFP][CPFP]（子为父偿）手续费加速方法，可以使用 LN URL 的额外特性，并在用户界面上有多个升级。

## 重大代码和文献变更

本周出现重大变更的有：[Bitcoin Core][Bitcoin Core]、[C-Lightning][C-Lightning]、[Eclair][Eclair]、[LDK][LDK]、[LND][LND][libsecp256k1][libsecp256k1]、[Hardware Wallet Interface (HWI)][Hardware Wallet Interface (HWI)]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK]、[Bitcoin Improvement Proposals (BIPs)][Bitcoin Improvement Proposals (BIPs)]、以及 [Lightning BOLTs][Lightning BOLTs]。

- [HWI #550][HWI #550] 为 Ledger 硬件签名设备可加载的最新比特币固件增加了支持，该固件支持版本 2 的 [PSBT][PSBT] 以及一些[输出脚本描述符][output script descriptors]。



[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019926.html

[previously proposed]:https://bitcoinops.org/en/newsletters/2021/09/15/#covenant-opcode-proposal

[joinpools]:https://bitcoinops.org/en/topics/joinpools/

[channel factories]:https://bitcoinops.org/en/topics/channel-factories/

[covenants]:https://bitcoinops.org/en/topics/covenants/

[MuSig2]:https://bitcoinops.org/en/topics/musig/

[OP_CHECKTEMPLATEVERIFY]:https://bitcoinops.org/en/topics/op_checktemplateverify/

[tapscript]:https://bitcoinops.org/en/topics/tapscript/

[BTCPay Server 1.4.6]:https://github.com/btcpayserver/btcpayserver/releases/tag/v1.4.6

[CPFP]:https://bitcoinops.org/en/topics/cpfp/

[Bitcoin Core]:https://github.com/bitcoin/bitcoin

[C-Lightning]:https://github.com/ElementsProject/lightning

[Eclair]:https://github.com/ACINQ/eclair

[LDK]:https://github.com/lightningdevkit/rust-lightning

[LND]:https://github.com/lightningnetwork/lnd/

[libsecp256k1]:https://github.com/bitcoin-core/secp256k1

[Hardware Wallet Interface (HWI)]:https://github.com/bitcoin-core/HWI

[Rust Bitcoin]:https://github.com/rust-bitcoin/rust-bitcoin

[BTCPay Server]:https://github.com/btcpayserver/btcpayserver/

[BDK]:https://github.com/bitcoindevkit/bdk

[Bitcoin Improvement Proposals (BIPs)]:https://github.com/bitcoin/bips/

[Lightning BOLTs]:https://github.com/lightning/bolts

[HWI #550]:https://github.com/bitcoin-core/HWI/issues/550

[PSBT]:https://bitcoinops.org/en/topics/psbt/

[output script descriptors]:https://bitcoinops.org/en/topics/output-script-descriptors/

