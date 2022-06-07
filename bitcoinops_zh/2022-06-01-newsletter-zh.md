---
title: 'Bitcoin Optech Newsletter #202'
permalink: /zh/newsletters/2022/06/01/
name: 2022-06-01-newsletter-zh
slug: 2022-06-01-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 描述了开发者们关于静默支付的试验性工作。此外，还有我们的常规栏目总结软件的新版本和候选版本信息，以及热门比特币基础设施软件的重大变更介绍。

## 新闻

- **静默支付的试验：** 如 [Newsletter
  #194][news194 silent] 中所述，*静默支付*使得可以给一个公共标识符（“地址”）付款而无需创建对该地址被付款的公开记录。本周，开发者 w0xlt 在 Bitcoin-Dev 邮件列表中[发布][w0xlt post]了一个[教程][sp tutorial]，可以通过一个 Bitcoin Core 的概念验证[实现][bitcoin core #24897]给默认 [signet][topic signet] 创建静默支付。包括一些热门钱包作者在内的其他几位开发者，在讨论该提案的其他细节，包括为其[创建地址格式][sp address]。

## 新版本和候选版本

*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- [HWI 2.1.1][] 修复了两个影响 Ledger 和 Trezor 设备的小 bug 并增加了对 Ledger Nano S Plus 的支持。

- [LND 0.15.0-beta.rc3][] 是该热门 LN 节点的下一个主要版本的候选版本。

## 代码和文档的重大变更

*本周内，[Bitcoin Core][bitcoin core repo]、[Core
Lightning][core lightning repo]、[Eclair][eclair repo]、[LDK][ldk repo],
[LND][lnd repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet
Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay
Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement
Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [BTCPay Server #3772][] 允许用户启用试验性功能以在正式发布前进行实际测试。

- [BTCPay Server #3744][] 添加了导出钱包交易为 CSV 或 JSON 格式的功能。

- [BOLTs #968][] 为使用比特币测试网和 signet 的节点添加默认 TCP 端口。

{% include references.md %}
{% include linkers/issues.md v=2 issues="3772,3744,968,24897" %}
[lnd 0.15.0-beta.rc3]: https://github.com/lightningnetwork/lnd/releases/tag/v0.15.0-beta.rc3
[hwi 2.1.1]: https://github.com/bitcoin-core/HWI/releases/tag/2.1.1
[news194 silent]: /en/newsletters/2022/04/06/#delinked-reusable-addresses
[w0xlt post]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-May/020513.html
[sp tutorial]: https://gist.github.com/w0xlt/72390ded95dd797594f80baba5d2e6ee
[sp address]: https://gist.github.com/RubenSomsen/c43b79517e7cb701ebf77eec6dbb46b8?permalink_comment_id=4177027#gistcomment-4177027
