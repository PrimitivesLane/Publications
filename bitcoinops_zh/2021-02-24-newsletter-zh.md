---
title: 'Bitcoin Optech Newsletter #137'
permalink: /zh/newsletters/2021/02/24/
name: 2021-02-24-newsletter-zh 
slug: 2021-02-24-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 描述了为 taproot 软分叉选择激活参数的讨论结果，并包括我们的常规部分，包括 Bitcoin StackExchange 精选问答，新版本和候选版本发布的公告，主流比特币基础设施软件的重要更新。

## News

* **Taproot 激活讨论：**  Michael Folkson [总结了](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018425.html)第二次关于 [taproot](https://bitcoinops.org/en/topics/taproot/) 激活参数的会议，结论是 "无论是 LOT=true 还是 LOT=false，都没有达成压倒性的共识"，[BIP8](https://github.com/bitcoin/bips/blob/master/bip-0008.mediawiki) 的 LockinOnTimeout（LOT）参数决定了节点是否需要强制信令来激活分叉。 然而，在其他激活参数上几乎达成了普遍共识，最明显的是将分叉激活所需信号的哈希率从 95%降低到 90%。
关于邮件列表上的 LOT 参数的讨论还在继续，主要是关于鼓励用户自己选择选项的影响，可以通过命令行选项或选择使用什么软件版本。 截至本报告撰写之时，还没有达成明确的协议，而且似乎也没有一个得到广泛支持的激活 taproot 的途径--尽管 taproot 本身似乎几乎是完全被人们所期望的。

## Selected Q&A from Bitcoin StackExchange

*[Bitcoin StackExchange](https://bitcoin.stackexchange.com/) 是 Optech 的贡献者寻找答案的首选地之一，也是一个我们在空闲时间帮助那些好奇或者困惑的用户的地方。在这个月度专题中，我们将重点介绍一些自上次更新以来投票最多的问题及答案。*

* [（密钥）分片是否是替代多签的好办法？](https://bitcoin.stackexchange.com/a/102007) 用户 S.O.S 问及类似 Shamir 秘密共享（SSS）这样的分片方案是否可行，以实现类似于多签的能力。 Pieter Wuille 指出了 `OP_CHECKMULTISIG` 相对于 SSS 的优势（问责制，不需要在一台机器上重构秘钥），SSS 相对于 `OP_CHECKMULTISIG` 的优势（更少的费用，更多的隐私），以及 [schnorr](https://bitcoinops.org/en/topics/schnorr-signatures/) 的考虑（缺乏问责制，不需要在一台机器上重构秘钥，更少的费用，更多的隐私，额外的复杂性）。

* [当 funding tx 还停留在 mempool 中时，能否关闭通道？](https://bitcoin.stackexchange.com/a/102180) 在尝试打开一个闪电网络通道但将 funding tx 的交易费率设置得太低后，PyrolitePancake 询问关于关闭通道，而资金交易仍在 mempool 中的情形。 虽然一种选择是通过等待确认和可能重播交易来探索打开通道，但 Rene Pickhardt 指出，双花 funding tx 的 inputs 将使其从 mempools 中被丢弃。 cdecker 为 C-lightning 提供了一些示例命令，可以转播资金交易或创建双花。

* [在 peerblockfilters=1 的情况下，数百个“btcwire 0.5.0/neutrino”连接正在从我的比特币节点下载 TB 级别的数据](https://bitcoin.stackexchange.com/a/102263)，qertip 注意到当运行 Bitcoin Core 0.21.0 并启用[简洁区块过滤器](https://bitcoinops.org/en/topics/compact-block-filters/)时，来自 `btcwire 0.5.0/neutrino` 用户代理的大量连接（75%）和带宽占用用（90%）。 Murch 指出，这些节点都是闪电网络守护节点，由于简洁区块过滤器既是 Bitcoin Core 的[新功能](https://bitcoinops.org/en/newsletters/2021/01/20/#bitcoin-core-0-21-0)，也是[默认禁用的](https://bitcoinops.org/en/newsletters/2020/05/20/#bitcoin-core-18877)，所以目前网络上可能缺乏简洁区块过滤器服务的节点，导致支持过滤器的节点流量较大。

* [是否有 `dumpwallet`（注：BTC core 程序的一个命令）的输出文档和解释？](https://bitcoin.stackexchange.com/a/101767) Andrew Chow 以这个答案为契机，对 `dumpwallet` RPC 的输出进行了补充说明和解释。

* [比特币是否有什么地方阻碍了实施 Monero 和 Zcash 相同的隐私协议？](https://bitcoin.stackexchange.com/a/101868) Pieter Wuille 列举了几个挑战，为什么开发者可能不会选择在 Monero 或 Zcash 所采取的隐私方法上工作，生态系统也可能不会选择支持这些方法。考虑因素包括“选择进入”方法的缺点、引入新的加密安全假设、可扩展性问题和供应量可审计性问题。

## Releases and release candidates

*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

* [LND 0.12.1-beta](https://github.com/lightningnetwork/lnd/releases/tag/v0.12.1-beta) 是 LND 的最新维护版本。 它修复了可能导致通道意外关闭的极端情形和可能导致不必要的支付失败的 bug，以及一些其他小的改进和 bug 修复。
    
## Notable code and documentation changes

*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[硬件钱包接口 (HWI)](https://github.com/bitcoin-core/HWI)、[Rust 比特币](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay 服务器](https://github.com/btcpayserver/btcpayserver/)、[比特币改进提案 (BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 的显著更新。*

* [Bitcoin Core #19136](https://github.com/bitcoin/bitcoin/issues/19136) 扩展了 getaddressinfo RPC，增加了一个新的 parent_desc 字段，该字段包含一个包含地址的钱包的[输出脚本描述符](https://bitcoinops.org/en/topics/output-script-descriptors/)。钱包的 [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 路径将剥离所有硬编码前缀，这样就只保留了公共派生步骤，允许将描述符导入到其他钱包软件中，这些软件可以监控收到的比特币到该地址及其关联地址。

* [Bitcoin Core #15946](https://github.com/bitcoin/bitcoin/issues/15946) 使得同时使用配置选项 `prune` 和 `blockfilterindex` 可以在修剪的节点上维护[简洁区块过滤器](https://bitcoinops.org/en/topics/compact-block-filters/)（如果使用 `peerblockfilters` 配置选项，也可以为它们服务）成为可能。 一位 LND 开发者[表示](https://github.com/bitcoin/bitcoin/pull/15946#issuecomment-571854091)，这对他们的软件是有利的，它也可以允许未来的更新，允许钱包逻辑使用区块过滤器来确定修剪的节点需要重新下载哪些历史区块，以便导入钱包。

* [Eclair #1693](https://github.com/ACINQ/eclair/issues/1693) 和 [Rust-Lightning #797](https://github.com/rust-bitcoin/rust-lightning/issues/797) 改变了节点地址公告的处理方式。目前的 [BOLT7](https://github.com/lightningnetwork/lightning-rfc/blob/master/07-routing-gossip.md) 规范要求对公告内的地址进行排序，然而一些实现并没有使用或执行这一规则。 Eclair 更新了他们的实现，开始排序，Rust-Lightning 更新了他们的实现，停止要求排序。 更新规范的 [PR](https://github.com/lightningnetwork/lightning-rfc/issues/842) 是开放的，但具体该怎么改还在讨论中。

* [HWI #454](https://github.com/bitcoin-core/HWI/issues/454) 更新了 `displayaddress` 命令，增加了对在 BitBox02 设备上注册 multisig 地址的支持。


* [BIPs #1052](https://github.com/bitcoin/bips/issues/1052) 将 [BIP338](https://github.com/bitcoin/bips/blob/master/bip-0338.mediawiki) 分配给一个在比特币 P2P 协议中增加 `disabletx` 消息的提案。在连接建立过程中发送此消息的节点向连接到它的节点发出信号，表明它永远不会在该连接上请求或宣布交易。如[通讯#131](https://bitcoinops.org/en/newsletters/2021/01/13/#proposed-disabletx-message) 所述，这允许节点对禁用的中继连接使用不同的限制，例如接受超出当前 125 连接最大值的额外连接。另见 1 月 12 日 P2P 开发者会议的[讨论总结](https://github.com/bitcoin-core/bitcoin-devwiki/wiki/P2P-IRC-meetings#topic-disabletx-p2p-message-sdaftuar)。