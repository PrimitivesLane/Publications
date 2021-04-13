---
title: 'Bitcoin Optech Newsletter #142'
permalink: /zh/newsletters/2021/04/07/
name: 2021-04-07-newsletter-zh 
slug: 2021-04-07-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh

---

本周的新闻简报包括我们的常规部分，其中包含服务和客户端软件的重要更新，新版本和候选版本以及对常见比特币基础设施软件的重要更新。

## 新闻

*本周没有值得关注的新闻报道。*



## 新版本和候选版本发布

新版本和通用的比特币基础设施项目的候选版本。请考虑升级到新版本或帮助测试候选版本。

- [C-Lightning 0.10.0](https://github.com/ElementsProject/lightning/releases/tag/v0.10.0)是此LN节点软件的最新的主要版本，包含其API的许多增强的功能，并包括对 [dual funding](https://bitcoinops.org/en/topics/dual-funding/)的实验性支持。
- [BTCPay 1.0.7.2](https://github.com/btcpayserver/btcpayserver/releases/tag/v1.0.7.2)修复了上周安全更新发布后发现的小问题。



## 重要代码和文档更新

*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[硬件钱包接口 (HWI)](https://github.com/bitcoin-core/HWI)、[Rust 比特币](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay 服务器](https://github.com/btcpayserver/btcpayserver/)、[比特币改进提案 (BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 的重要更新。*

- [Bitcoin Core＃20286](https://github.com/bitcoin/bitcoin/issues/20286)从`gettxout`，`getrawtransaction`， `decoderawtransaction`，`decodescript`，`gettransaction`，REST接口`/rest/tx`，`/rest/getutxos`，`/rest/block`这些RPCs的响应中删除字段`addresses`和`reqSigs`。当存在定义明确的地址时，响应目前会包括可选字段`address` 。这些废弃的字段过去在非P2SH多签（bare multisig）的上下文中使用，而在当今的网络上并没有实质性的用途。在 Bitcoin Core 23.0中删除配置选项`-deprecatedrpc=addresses`之前可以通过配置这一字段来输出这些废弃字段。

- [Bitcoin Core＃20197](https://github.com/bitcoin/bitcoin/issues/20197)通过更新入站peer的驱逐逻辑（eviction logic ）来提高对等体连接的多样性，以保护运行时间最长的洋葱peer节点（onion peers）。它还为当前驱逐保护逻辑增加了单元测试覆盖率。洋葱节点由于其相对于IPv4和IPv6对等体更高的延迟而在驱逐标准上处于较差的水平，导致用户提交了多个issues，包括[1](https://github.com/bitcoin/bitcoin/issues/11537)、[2](https://github.com/bitcoin/bitcoin/issues/19500)。对这些issue的[最初的回复](https://bitcoinops.org/en/newsletters/2020/09/09/#bitcoin-core-19670)是开始为本地服务器节点保留插槽作为洋葱节点的代理。后来，添加了[对入站洋葱节点连接的显式检测](https://bitcoinops.org/en/newsletters/2020/10/07/#bitcoin-core-19991)。

  使用更新后的逻辑，一半受保护的插槽将分配给任何洋葱节点和本地主机节点，且洋葱节点的优先级高于本地服务器节点。现在对I2P隐私网络的支持已添加到Bitcoin Core中（请参阅 [Newsletter #139](https://bitcoinops.org/en/newsletters/2021/03/10/#bitcoin-core-20685)），下一步将是将驱逐保护扩展到I2P节点，因为它们通常比洋葱节点具有更高的延迟。

- [Eclair＃1750](https://github.com/ACINQ/eclair/issues/1750)删除了对Electrum的支持和同时也删除了相应的10,000行代码。Eclair以前曾将Electrum用于移动钱包。但是，现在建议将新实现[Eclair-kmp](https://github.com/ACINQ/eclair-kmp)用于移动钱包，从而使Electrum不再需要支持Eclair。

- [Eclair＃1751](https://github.com/ACINQ/eclair/issues/1751)在`payinvoice`命令中添加了一个选项`blocking`，该选项会在付款完成前阻塞对`payinvoice`的调用。在此之前，用户需要低效率地轮询`getsentinfo `API才能知道何时付款完成。

