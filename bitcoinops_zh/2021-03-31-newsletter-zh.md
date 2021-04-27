```
title: 'Bitcoin Optech Newsletter #142'
permalink: /zh/newsletters/2021/03/31/
name: 2021-03-31-newsletter-zh 
slug: 2021-03-31-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
```

本周简报将介绍一篇有关**闪电网络（LN）概率路径选择**的论文和对其简单的讨论（probabilistic path selection for LN ），还包括我们的常规板块，即在Bitcoin StackExchange 上热门问答的总结，新版本和候选版本，以及比特币基础软件的重要更新。



## 行动项目

- **BTCPay Server 升级至1.0.7.1:** 根据项目的发行说明， [该版本](https://github.com/btcpayserver/btcpayserver/releases/tag/v1.0.7.1)修复了“影响BTCPay Server 1.0.7.0版本甚至更早期的版本中存在的一个关键漏洞和几个影响较小的漏洞”。



## 新闻

**关于概率路径选择（probabilistic path selection）的论文：**René Pickhardt在Lightning-Dev 的邮件列表里 [上传](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-March/002984.html) 了一篇其与Sergei Tikhomirov, Alex Biryukov, and Mariusz Nowostawski一起合著的 [论文](https://arxiv.org/abs/2103.08576) 。这篇文章建立了在信道容量范围内余额均衡分布的信道网络模型。例如，在Alice 和Bob之间有一个容量是1亿聪的信道，该论文假设以下所有状态均相同，在网络上的所有其他信道也是如此：

| Alice           | Bob             |
| --------------- | --------------- |
| 0 sat           | 100,000,000 sat |
| 1 sat           | 99,999,999 sat  |
| …               | …               |
| 100,000,000 sat | 0 sat           |

做该假设可以使得作者们基于交易额和其所需经历的跳数（通道数）得出支付成功的概率。这使作者能够证明几种已知启发式算法的好处----例如保持路径短，并使用[多路径支付](https://bitcoinops.org/en/topics/multipath-payments/)将较大的交易分成较小的交易（在某些其他假设下）。他们还使用该模型评估新提案，如通过 [bolts #780](https://github.com/lightningnetwork/lightning-rfc/issues/780)来保证[Just-In-Time(JIT)再平衡](https://bitcoinops.org/en/topics/jit-routing/)。

这篇论文利用其结论提供了一种路由算法，此算法声称与简化现有的路由算法相比，它可以将支付重试次数减少20%。新算法更倾向于计算成功率更高的路由，而现有算法则使用启发式方法。结合JIT再平衡，他们估计可以提高48％。 鉴于每次重试通常需要几秒钟，并且在某些情况下可能需要更长的时间，因此该算法可以提供更好的用户体验。 **该算法针对多个示例网络进行了测试**，其中包括从近1,000个活跃信道的快照中提取的网络。

该论文有意不考虑路由费，邮件列表上的大多数回复都集中在如何使用该成果上，同时仍得确保用户不会支付过多的费用。关于 [支付批处理文章](https://bitcoinops.org/en/payment-batching/)的更新：Optech发布过一篇关于支付批处理的文章，该文章是对我们在 [Newsletter #37](https://bitcoinops.org/en/newsletters/2019/03/12/#optech-publishes-book-chapter-about-payment-batching) 公告原文的更新。 支付批处理是一项可以帮助付款方节省多达80％交易手续费的技术。

## 选自Bitcoin StackExchange 上的热门问答

*[Bitcoin StackExchange](https://bitcoin.stackexchange.com/)* 是Optech贡献者寻找问题答案的首选地之一，也是一个我们在空闲时间帮助那些好奇或者困惑的用户的地方。在这个月度专题中，我们将重点介绍一些自上次更新以来投票最多的问题及答案。

-  [对于交易所而言采用原生隔离见证有多难？](https://bitcoin.stackexchange.com/a/103674) 

  比特币开发者instagibbs 列出了交易所采用原生隔离见证时需要考虑的几点因素，其中包括地址生成，确保可支付性（spendability),支持和业务方面的考虑，以及像硬件安全模块（ HSMs）这样的数字签名基础设施的兼容性。

  

- [如何计算98%的比特币在何时被挖完？](https://bitcoin.stackexchange.com/a/103159)

  Murch预估98%的比特币将于2030年至2031年被挖完，同时他也链接了一个附有额外量度的[奖励计划表](https://docs.google.com/spreadsheets/d/12tR_9WrY0Hj4AQLoJYj9EDBzfA38XIVLQSOOOVePNm0/edit#gid=0)。

  

- [如何将Bitcoin Core和匿名网络协议I2P配合使用？](https://bitcoin.stackexchange.com/a/103402)

  随着 [Bitcoin Core #20685](https://bitcoinops.org/en/newsletters/2021/03/10/#bitcoin-core-20685)的合并，比特币开始支持I2P网络。Michael Folkson总结了Jon Atack关于如何配置比特币核心以使用I2P的[原始思路](https://twitter.com/jonatack/status/1366764964896075776?s=20)。

  

- [比默认内存池更大的节点是否会重新传输那些从较小的内存池中被删除了的交易？](https://bitcoin.stackexchange.com/a/103104)

  Pieter Wuille指出，交易的重新广播目前是[钱包的必要功能](https://bitcoin.stackexchange.com/questions/103261/does-my-node-rebroadcast-its-mempool-transactions-on-startup/103262#103262)，或许节点也应该重新广播未确认交易，并指出 [Bitcoin Core #21061](https://github.com/bitcoin/bitcoin/issues/21061) 正在朝着这个目标努力。

  

- [在软分叉激励机制中应该使用区块高度、中位时间过去（MTP ）或者这两者的混合产物吗？](https://bitcoin.stackexchange.com/a/103854)

  David A. Harding概述了中位时间过去（MTP）和区块高度两者作为比特币内的计时机制的优缺点。 MTP大致对应于时钟时间，但可以由矿工操纵以跳过信号周期。区块高度不是时间连贯的（是离散的），但也无法像MTP一样具有那么强的矿工可操作性。

  

- [为什么建议“付款时不要发送整数金额”以增加隐私？](https://bitcoin.stackexchange.com/a/103260) 

  使用者 chytrik 提供了不同的示例来说明[整数试探](https://en.bitcoin.it/wiki/Privacy#Round_numbers) 以及避免支付整数金额会更好地保护隐私的原因。



## 新版本和候选版本发布

新版本和通用的比特币基础设施项目的候选版本。请考虑升级到新版本或帮助测试候选版本。

[●](https://bitcoinops.org/en/newsletters/2021/03/31/#btcpay-server-1-0-7-1) [BTCPay Server 1.0.7.1](https://github.com/btcpayserver/btcpayserver/releases/tag/v1.0.7.1) 修复了几个安全漏洞。 它还包括一系列改进和非安全性漏洞的修复。

[●](https://bitcoinops.org/en/newsletters/2021/03/31/#hwi-2-0-1) [HWI 2.0.1](https://github.com/bitcoin-core/HWI/releases/tag/2.0.1) 是补丁修正的版本，该版本解决hwi-qt用户界面中Trezor T密码输入和键盘快捷键的小问题。

[●](https://bitcoinops.org/en/newsletters/2021/03/31/#c-lightning-0-10-0-rc2) [C-Lightning 0.10.0-rc2](https://github.com/ElementsProject/lightning/releases/tag/v0.10.0rc2) 是此闪电网络（LN）节点软件的下一个主要版本的候选版本。



## 重要代码和文档更新

本周重要更新：

*[Bitcoin Core](https://github.com/bitcoin/bitcoin), [C-Lightning](https://github.com/ElementsProject/lightning), [Eclair](https://github.com/ACINQ/eclair), [LND](https://github.com/lightningnetwork/lnd/), [Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning), [libsecp256k1](https://github.com/bitcoin-core/secp256k1), [Hardware Wallet Interface (HWI)](https://github.com/bitcoin-core/HWI), [Rust Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin), [BTCPay Server](https://github.com/btcpayserver/btcpayserver/), [Bitcoin Improvement Proposals (BIPs)](https://github.com/bitcoin/bips/), and [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/).*

- [Bitcoin Core #17227](https://github.com/bitcoin/bitcoin/issues/17227) 添加了一个新的`make apk`指令来构建系统，用于Android操作系统的bitcoin-qt打包。 这将继续之前的工作，即为打包Android NDK增加支持。 还包括用于构建Android版本的Bitcoin Core的文档以及用于测试Android构建系统的持续集成工作。

- [Rust-Lightning #849](https://github.com/rust-bitcoin/rust-lightning/issues/849) 使信道的`cltv_expiry_delta`可配置，并将默认值从72个块减少到36个块。 该参数设置了一个节点在从其下游节点获悉是否支付成功后必须与上游节点进行支付尝试的截止时间。它必须足够长，使其在必要时确认链上事务，但也应足够短，以使其与试图减小延误可能性的其他节点相比有竞争性。 另请参阅[Newsletter #40](https://bitcoinops.org/en/newsletters/2019/04/02/#lnd-2759)，其中LND将其值减小到40个区块。

- [C-Lightning #4427](https://github.com/ElementsProject/lightning/issues/4427) 通过使用配置选项`--experimental-dual-fund`，可以尝试使用[双重注资(dual funded)](https://bitcoinops.org/en/topics/dual-funding/) 的支付渠道。双重注资允许初始通道余额的资金由初始化通道的节点和接受通道的节点共同出资，这对于希望在通道完成开放后立即开始接收付款的商家和其他用户很有用。

- [Eclair #1738](https://github.com/ACINQ/eclair/issues/1738) 更新在使用[锚定输出](https://bitcoinops.org/en/topics/anchor-outputs/)时对被撤销的[HTLCs](https://bitcoinops.org/en/topics/htlc/)的惩罚执行机制。这是一个和锚定输出无关的更改，但是在将它们添加到协议中的同时生效（introduced),就可以将多个`SIGHASH_SINGLE | SIGHASH_ANYONECANPAY HTLC`输出到单个交易中（请参阅[Newsletter #128](https://bitcoinops.org/en/newsletters/2020/12/16/#bolts-803)。该PR确保在每笔交易中，所有有撤销密钥可以使用的输出都可以索赔而不是每笔交易只能索赔一个）。

- [BIPs #1080](https://github.com/bitcoin/bips/issues/1080) 更新 [BIP8](https://github.com/bitcoin/bips/blob/master/bip-0008.mediawiki) 中的一个`nimum_activation_height`参数，该参数将延迟节点开始执行锁定的软分叉的时间，直到超过指定高度。这使得BIP8与快速试验提案(见 [Newsletter #139](https://bitcoinops.org/en/newsletters/2021/03/10/#a-short-duration-attempt-at-miner-activation))兼容，该提案允许矿工激活 [taproot](https://bitcoinops.org/en/topics/taproot/)，但直到实施快速试验的软件发布大约6个月后才开始执行taproot的规则。

