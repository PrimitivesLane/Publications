---
title: 'Bitcoin Optech Newsletter #196'
permalink: /zh/newsletters/2022/04/20/
name: 2022-04-20-newsletter-zh
slug: 2022-04-20-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---
本周的 Newsletter 总结了关于允许在比特币上进行量子安全的密钥交换的讨论，并包括我们的常规部分，介绍了服务端和客户端软件值得注意的变更，新版本和候选版本，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻

- **量子安全的密钥交换：** Erik Aronesty 在 Bitcoin-Dev 邮件列表中发起了一个关于抗量子的[主题][aronesty qc]——在快速量子计算机（QC）被开发出来的情况下保持比特币的安全。据预测，快速量子计算机能够在不知道原始私钥的情况下生成与比特币公钥相对应的签名，这将允许拥有快速量子计算机的人花费属于其他人的钱。很少有安全研究人员认为快速量子计算机是一个近期的威胁，但任何能够消除它们带来的担忧而又不严重干扰比特币现有使用的方法都值得考虑。

  Aronesty [建议][fournier qc]允许用户接收由量子安全算法保证的公钥的付款，同时也可以用现有的比特币公钥来保证比特币的安全，这样，在任何比特币因加密密钥失效而被盗之前，需要破解两种密钥算法。实现这个目的需要一个软分叉共识变化，并且在最坏的情况下，鉴于量子安全的见证数据比比特币目前使用的 ECDSA 和 [Schnorr][topic schnorr signatures] 见证数据更大，可能会减少每个区块的最大有用交易数量。

  Lloyd Fournier [建议][fournier qc]开发一个标准化的方案，以允许 taproot 输出承诺到常用的 schnorr 公钥之外的量子安全的公钥。量子安全公钥目前可能无法使用，但如果比特币用户对即将到来的快速量子计算机更加担心，他们可以选择软分叉进行共识变更，要求使用量子安全的花费路径。Fournier 还建议，可以在 [BitcoinProblems.org][] 上为当前和未来的研究人员以及开发人员[介绍][qc issue]有关这个问题的细节和可能的解决方案。

## 服务和客户端软件的变更

*在这个月的栏目中，我们突出比特币钱包和服务的有趣升级*。

- **Bitcoin.com 增加向 segwit 发送交易：** 在最近对 Bitcoin.com 钱包的[更新][bitcoin.com segwit]中，支持了向原生 segwit（[bech32][topic bech32]）地址发送交易。

- **Kraken 增加了对闪电网络的支持：** Kraken 增加了对[闪电网络][kraken lightning]存款和提款的支持（使用 LND），最高可达 0.1 BTC。

- **Cash App 增加闪电网络收款支持：** 在之前增加[闪电网络发送功能][news183 cash app ln send]后，Cash App 推出了通过闪电网络收款的能力。Cash App 使用[闪电开发套件（LDK）][ldk website]来实现闪电网络功能。

- **BitPay 增加闪电网络接收支持：** 支付处理商 BitPay [宣布支持][bitpay lightning]他们的商户接受闪电网络支付。

## 新版本和候选版本

*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- [LND 0.14.3-beta][] 是这个流行的闪电网络节点软件的一个带有几个错误修复的版本。

- [Bitcoin Core 23.0 RC5][] 是这个主流全节点软件的下一个主要版本的候选发布版本。[发布草案说明][bcc23 rn]列出了多项改进，我们鼓励高级用户和系统管理员在最终发布前进行[测试][test guide]。

- [Core Lightning 0.11.0rc3][] 是这个流行的闪电网络节点软件的下一个主要版本的候选发布版本。

## 代码和文档的重大变更
*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [LND #5810][] 实现了对付款元数据的发送支持。如果收据包括付款元数据，发送方将把这些数据编码为接收方的 TLV 记录。这是朝着解锁[无状态收据][topic stateless invoices]迈出的另一步，它允许接收方放弃存储出示给潜在发件人的收据，而在付款尝试到达接收方时重新生成这些发票。

- [LND #6212][] 实现了如果接受 HTLC 可能需要立即或在短时间内关闭该通道，可以防止 HTLC 通过 HTLC 拦截器发送到外部进程。这可能会发生在 HTLC 的到期时间接近最近产出的区块时。

- [LND #6024][] 增加了一个 `time_pref` 寻路参数，可以用来改变在选择不同通道作为路径（选择能更快完成支付或费用更低）时的偏好权重。

- [LND #6385][] 删除了在构建新支付时使用原始闪电网络协议洋葱支付格式的选项，要求用户创建一个 TLV 式的洋葱格式。TLV 洋葱是在 2019 年加入协议的（见 [Newsletter #55][news55 tlv]），两年多来一直是所有闪电网络软件的默认格式。其他闪电网络软件也一直在做类似的变更，以放弃对旧的洋葱格式支持，如 [Newsletter
  #158][news158 cl4646] 中提到的对核心闪电网络的更新。


[topic schnorr signatures]: https://bitcoinops.org/en/topics/schnorr-signatures/
[topic bech32]: https://bitcoinops.org/en/topics/bech32/
[topic stateless invoices]: https://bitcoinops.org/en/topics/stateless-invoices/

[bitcoin core 23.0 rc5]: https://bitcoincore.org/bin/bitcoin-core-23.0/
[bcc23 rn]: https://github.com/bitcoin-core/bitcoin-devwiki/wiki/23.0-Release-Notes-draft
[test guide]: https://github.com/bitcoin-core/bitcoin-devwiki/wiki/23.0-Release-Candidate-Testing-Guide
[lnd 0.14.3-beta]: https://github.com/lightningnetwork/lnd/releases/tag/v0.14.3-beta
[core lightning 0.11.0rc3]: https://github.com/ElementsProject/lightning/releases/tag/v0.11.0rc3
[aronesty qc]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-April/020209.html
[fournier qc]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-April/020214.html
[qc issue]: https://github.com/bitcoin-problems/bitcoin-problems.github.io/issues/4
[bitcoinproblems.org]: https://bitcoinproblems.org/
[news55 tlv]: https://bitcoinops.org/en/newsletters/2019/07/17/#bolts-607
[news158 cl4646]: https://bitcoinops.org/en/newsletters/2021/07/21/#c-lightning-4646
[bitcoin.com segwit]: https://support.bitcoin.com/en/articles/3919131-can-i-send-to-a-bc1-address
[kraken lightning]: https://blog.kraken.com/post/13502/kraken-now-supports-instant-lightning-network-btc-transactions/
[news183 cash app ln send]: https://bitcoinops.org/en/newsletters/2022/01/19/#cash-app-adds-lightning-support
[ldk website]: https://lightningdevkit.org/
[bitpay lightning]: https://bitpay.com/blog/bitpay-supports-lightning-network-payments/
[LND #5810]: https://github.com/lightningnetwork/lnd/pull/5810
[LND #6212]: https://github.com/lightningnetwork/lnd/pull/6212
[LND #6024]: https://github.com/lightningnetwork/lnd/pull/6024
[LND #6385]: https://github.com/lightningnetwork/lnd/pull/6385

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