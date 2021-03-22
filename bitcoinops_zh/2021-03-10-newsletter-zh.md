---
title: 'Bitcoin Optech Newsletter #139'
permalink: /zh/newsletters/2021/03/10/
name: 2021-03-10-newsletter-zh 
slug: 2021-03-10-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
Markdown.styles: ./newsletter.css
---



本周的 Newsletter总结了有关激活taproot的建议的方法和到对建立在taproot的现有软件的记录工作的链接。同时也包括我们的常规部分，其中包括比特币核心PR审查俱乐部（Bitcoin Core PR Review Club）会议的总结，新版本和可用的候选版本以及对常见比特币基础设施软件的重要更新。

# 新闻

- Taproot 激活讨论：看到了不同的人群反对[BIP8](https://github.com/bitcoin/bips/blob/master/bip-0008.mediawiki) `LockinOnTimeout=true`（`LOT=true`）或`LOT=false`，因此本周邮件列表上的大多数讨论都集中在可供替代的激活机制上。一些建议包括：
  - 用户激活的软分叉（UASF）：正在讨论的用来在比特币内核的软分叉中实现BIP8 `LOT=true`的一个计划，这个计划授权矿工在2022年7月之前为taproot的激活发送信号（广泛建议），但也允许矿工更早地激活它。
  - 标记日（Flag day）： 若干提案（[1](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018495.html)，[2](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018538.html)）在特定块的高度或时间将程序写入节点，是从taproot激活的时候开始（建议）大致18个月。矿工发送信号不需要引起激活，也不能引起较早的激活。Anthony Towns编写了[实施草案](https://github.com/bitcoin/bitcoin/issues/21378)。

- Taproot 激活讨论：看到了不同的人群反对[BIP8](https://github.com/bitcoin/bips/blob/master/bip-0008.mediawiki) `LockinOnTimeout=true`（`LOT=true`）或`LOT=false`，因此本周邮件列表上的大多数讨论都集中在可供替代的激活机制上。一些建议包括：
  - 用户激活的软分叉（UASF）：正在讨论的用来在比特币内核的软分叉中实现BIP8 `LOT=true`的一个计划，这个计划授权矿工在2022年7月之前为taproot的激活发送信号（广泛建议），但也允许矿工更早地激活它。
  - 标记日（Flag day）： 若干提案（[1](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018495.html)，[2](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018538.html)）将程序写入节点，在特定块的高度或时间，从taproot激活的时候开始（建议）大致18个月。矿工发送信号不需要引起激活，也不能引起较早的激活。Anthony Towns编写了[实施草案](https://github.com/bitcoin/bitcoin/issues/21378)。

  - 降低门槛：若干提案（[1](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018476.html)， [2](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018587.html)）逐步减少区块的数量，这些区块必须表明矿工准备好在新的共识规则锁定之前执行taproot。另见安东尼·唐斯去年描述的提案 [Newsletter #107](https://bitcoinops.org/en/newsletters/2020/07/22/#mailing-list-thread)。
  - 可配置的`LOT`：除了前面讨论的使BIP8的`LOT`值成为配置选项的提案（请参阅[Newsletter #137](https://bitcoinops.org/en/newsletters/2021/02/24/#taproot-activation-discussion)）之外，还[发布](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018514.html)了简单的代码， 显示了`LOT=true`如何通过调用RPC命令的外部脚本来实施。另外还[创建](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018512.html)了额外的代码显示`LOT=true`如何也会被节点操作员反对，他们担心这会导致区块链不稳定。
  - 矿工激活的短期尝试：给矿工大约三个月的时间来锁定taproot的一个更新的[提案](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018583.html)，将从矿工实现激活逻辑的完整节点发布后不久后开始。如果尝试失败，将鼓励社区继续使用其他激活方法。如果尝试成功，在taproot激活之前将会有几个月的时间来允许大多数经济体更新他们的节点。该提案[基于Bitcoin Core现有BIP9代码](https://github.com/bitcoin/bitcoin/issues/21377)并[先前提出的BIP8实现](https://github.com/bitcoin/bitcoin/issues/21392)的草案实施分别由Anthony Towns和Andrew Chow编写。



似乎所有提案都不大可能成为每个人的首选，但似乎很多人都[愿意接受](https://gist.github.com/michaelfolkson/92899f27f1ab30aa2ebee82314f8fe7f)以Speedy Trial为名的短时尝试 (the short-duration attempt )。这里仍然存在一些问题，包括：

- 可以选择强制性激活：尽管该提议明确鼓励如果矿工没有迅速表示对taroot的充分支持，就进行其他激活尝试，但有人[担心](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018596.html)可能会被寻求快速激活的一组用户选择强制激活，尽管有人[指出](http://gnusha.org/taproot-activation/2021-03-06.log)，以前没有人表达过在如此危险的较短时间线上尝试强制激活的愿望。
- 使用基于高度或基于时间参数：该提案描述了使用时间戳(基于前11个块的中值)或块高度设置其`start`，`timeout`和`minimum_activation`参数之间的权衡。使用时间戳将导致比特币核心的最小和最容易审查的补丁。使用heights可以提供更多的可预测性，特别是对于矿工来说，并且可以与其他使用BIP8的尝试兼容。

- 短视：有人[担心](https://twitter.com/rusty_twit/status/1368325392591822848)该提案过于关注短期。正如[在IRC上总结的](http://gnusha.org/taproot-activation/2021-03-08.log)：“ Speedy Trial为矿工激活taproot的（可能的）情况做了充分的准备，但它没有整理记录从Segwit未能及时激活中吸取的教训。通过激活taproot，我们有机会为未来的激活创建一个模板，该模板将明确定义开发者、矿工、商人、投资者和终端用户的角色和责任，激活可以以各种方式进行，而不仅仅是最好的结果;特别是赋予比特币经济用户最终仲裁者的地位。在未来定义这一点只会变得更加困难，因为我们只有在已经陷入危机的时候才会这么做，而且比特币的增长意味着未来的协议需要在更大的规模上完成，因此也会有更大的困难。”
- 速度：该提案是基于## taproot-activation IRC频道的初步讨论，建议给矿工大约三个月的时间来锁定taproot，并从信号测量开始到激活前等待六个月的固定时间（如果锁定已达成）。有些人寻求稍短或稍长的时间线。

我们将继续跟踪有关各种提案的讨论，并将总结任何重大进展在未来的新闻通讯中。

- 记录使用和建立在taproot上的意图：在讨论激活方法时，克里斯·贝尔彻（Chris Belcher）[表示](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018538.html)，已经编译了大量软件，其开发人员在围绕激活该软分叉的辩论中表示了实现隔离见证的意图。他建议可以创建一个类似的列表，以记录taproot提供的支持数量以供以后使用。这样一来，很明显，无论经济最终如何被激活，大部分经济体都希望拥有主根。

  杰里米·鲁宾[贴](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018604.html)到Bitcoin-dev邮件列表一个链接到一个有点相关的[维基页面](https://en.bitcoin.it/wiki/Taproot_Uses)，开发者可以张贴链接到他们在taproot的新提议的功能上建设的项目。这可以确保taproot提供人们真正想要的解决方案，并且其设计方式将使其功能得以使用。



## 比特币核心PR审查俱乐部

在本月度部分中，我们总结了最近的[比特币核心PR审查俱乐部（Bitcoin Core PR Review Club）](https://bitcoincore.reviews/) 会议，重点介绍了一些重要的问题和答案。单击下面的问题以查看会议答案的摘要。

[Erlay：高效带宽的事务中继协议](https://bitcoincore.reviews/18261)是Gleb Naumenko的PR（[＃18261](https://github.com/bitcoin/bitcoin/issues/18261)），提议在Bitcoin Core中实现[BIP330](https://github.com/bitcoin/bips/blob/master/bip-0330.mediawiki)。

评论俱乐部的讨论重点在于权衡，实施和和[Erlay](https://bitcoinops.org/en/topics/erlay/)相关的潜在的新攻击媒介。在随后的会议中，Review俱乐部讨论了[Minisketch](https://bitcoinops.org/en/topics/minisketch/)，这是一个实现PinSketch集协调算法的 [库](https://github.com/sipa/minisketch)，该算法是Erlay中高效中继协议的基础。

<details class="details-1">
<summary>什么是Erlay？</summary>
一种新的交易中继方法，该方法基于洪泛和集合对帐的组合（当前事务中继仅是洪泛），以提高带宽效率，可伸缩性和网络安全性。该想法在2019年的论文<a href="(https://arxiv.org/abs/1905.10518)">《比特币的带宽高效交易中继》</a>中提出，并在<a href="https://github.com/bitcoin/bips/blob/master/bip-0330.mediawiki">BIP330</a>中进行了指定 。  
</details>

<details>
<summary>Erlay有什么优点？</summary>
<a href="https://bitcoincore.reviews/18261#l-94">交易中继使用的带宽较低</a>，大约包括操作节点所需带宽的一半，以及<a href="https://bitcoincore.reviews/18261#l-97">peer节点连接的可伸缩性</a>，从而使网络对分区攻击更健壮，而<a href="https://bitcoincore.reviews/18261#l-99">单个节点对Eclipse攻击的抵抗力更强</a>。  
</details>

<details>
<summary>Erlay的一些权衡是什么？</summary>
交易传播延迟稍有增加。据估计，Erlay将在所有节点之间中继未确认事务的时间从3.15s增加到5.75s，仅占整个事务处理时间约10分钟的一小部分。另一个权衡是额外的代码和计算复杂性。 
</details>

<details>
<summary>为什么设置由Erlay引入的对帐的规模要比泛洪更好？</summary>
通过泛洪传播交易（每个节点向每个peer节点宣布它收到的每个事务）具有较差的带宽效率和较高的冗余性。随着网络连接的增加，这种情况会变得越来越明显，这对于网络的增长和安全性来说是不可以接受的。Erlay通过减少效率低下的泛洪发送的事务数据并将其替换为更有效的集合对帐来提高可拓展性。  
</details>


<details>
<summary>现有P2P消息类型的变换频率将发生什么变化？</summary>
使用Erlay，inv消息发送的频率将降低。getdata并且tx 消息频率将保持不变。 
</details>

<details>
<summary>2个peer节点将如何使用Erlay的对帐达成协议？</summary>
通过sendrecon在版本Verack握手期间交换的新对等消息。 
</details>


## 新版本和候选版本发布

新版本和流行的比特币基础设施项目的候选版本。请考虑升级到新版本或帮助测试候选版本。

- [Eclair 0.5.1](https://github.com/ACINQ/eclair/releases/tag/v0.5.1)是此LN节点的最新版本，包括对启动速度的改进，在同步网络图时减少了带宽消耗，以及在准备支持[锚定输出](https://bitcoinops.org/en/topics/anchor-outputs/)方面的一系列小改进。
- [HWI 2.0.0RC2](https://github.com/bitcoin-core/HWI/releases/tag/2.0.0-rc.2)是[HWI](https://github.com/bitcoin-core/HWI/releases/tag/2.0.0-rc.2)的下一个主要版本的发行候选版本。

## 重要代码和文档更新

*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[硬件钱包接口 (HWI)](https://github.com/bitcoin-core/HWI)、[Rust 比特币](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay 服务器](https://github.com/btcpayserver/btcpayserver/)、[比特币改进提案 (BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 的重要更新。*

- [Bitcoin Core＃20685](https://github.com/bitcoin/bitcoin/issues/20685)通过使用[I2P SAM协议](https://geti2p.net/en/docs/api/samv3)增加了对I2P隐私网络的支持 。[长期以来一直要求使用](https://github.com/bitcoin/bitcoin/issues/2091)此功能，直到最近才通过添加[addr v2](https://bitcoinops.org/en/topics/addr-v2/)使该功能成为可能。尽管仍在为希望运行I2P的节点运营商创建文档，但[比特币StackExchange Q＆A](https://bitcoin.stackexchange.com/questions/103402/how-can-i-use-bitcoin-core-with-the-anonymous-network-protocol-i2p)提供了入门指南。
- [C-Lightning＃4407](https://github.com/ElementsProject/lightning/issues/4407)使用新字段更新RPC`listpeers`，该字段提供有关每个渠道当前单边平仓交易的信息，包括其费用（以总费用条款和费用率计）。
- [Rust-Lightning＃646](https://github.com/rust-bitcoin/rust-lightning/issues/646)添加了查找付款的多个路径的功能，从而可以添加[多路径付款](https://bitcoinops.org/en/topics/multipath-payments/)的功能支持。
- [BOLTs＃839](https://github.com/lightningnetwork/lightning-rfc/issues/839)增加了资金交易超时建议，以在无法确认资金交易时节省资金费用，从而为渠道出资者和受款人提供更有力的保证。新建议建议出资者承诺确保在2016年区块中确认资金交易，如果在2016年区块中未确认资金交易，则受赠人忘记了待处理的渠道。
- [BTCPay Server＃2181](https://github.com/btcpayserver/btcpayserver/issues/2181)大写bech32地址在将[BIP21](https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki) URI表示为QR码的时候。 由于大写子串可以更有效地编码，因此这会导致[密度较低的QR码](https://bitcoinops.org/en/bech32-sending-support/#creating-more-efficient-qr-codes-with-bech32-addresses)。更改之前，对具有BIP21 URI方案的钱包进行了广泛的[兼容性调查](https://github.com/btcpayserver/btcpayserver/issues/2110)。







