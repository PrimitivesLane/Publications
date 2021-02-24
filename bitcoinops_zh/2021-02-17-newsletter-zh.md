---
title: 'Bitcoin Optech Newsletter #136'
permalink: /zh/newsletters/2021/02/17/
name: 2021-02-17-newsletter-zh 
slug: 2021-02-17-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---
本周的 Bitcoin Optech 介绍了一种新的恒定时间的生成和验证签名的算法，提及了一个停止处理未经请求事务的提案，总结了设置多签钱包的 BIP，分享了从管理闪电网络上托管物的讨论中的提炼的一些思考，提及了降低恶意硬件钱包风险的新协议。此外还有我们常规的新闻章节部分，包括客户端和服务的更新，新版本和候选版本发布的公告，以及主流比特币基础设施软件的重要更新。

## News

* **快速签名操作：** Russell O’Connor 和 Andrew Poelstra 发表了一篇[博客](https://medium.com/blockstream/a-formal-proof-of-safegcd-bounds-695e1735a348)，宣布实现了一种算法，可以将 Bitcoin Core 的签名验证效率提升 15% 。同时，它还可以将生成签名的时间降低 25% ，同时保持使用常数时间算法，以降低侧信道数据泄漏的风险。硬件钱包的开发者应对此特别感兴趣，因为硬件钱包的计算资源极其有限。
这篇博文描述了由 Daniel J. Bernstein 和 Bo-Yin Yang 提出的新算法的进展；由Peter Dettman（[第 111 期](https://bitcoinops.org/en/newsletters/2020/08/19/#proposed-uniform-tiebreaker-in-schnorr-signatures) Bitcoin Optech 曾介绍）完成的针对 libsecp256k1 的实现；由 Pieter Wuille 提出的一种新颖的、节省 CPU 的方法，用于计算算法在恒定时间内实现其目标所需的最大步数；由 Gregory Maxwell 提出的更高效的 Bernstein 和 Yang 的算法变体；以及由 O'Connor 和 Poelstra 在 Coq 证明助手中实现 Wuille 的约束验证程序，以帮助确保其正确性。Wuille 在 [PR #831](https://github.com/bitcoin-core/secp256k1/issues/831) 的进展中更新了 Dettman 针对 libsecp256k1 的原始实现。

* **停止处理未经请求事务的提案：** 节点通常希望在 P2P 协议的 inv 消息中收到新事务的公告。如果节点对此事务有兴趣（例如此前从未从其它节点收到该事务）,它需要事务使用「getdata」消息，广播者使用 tx 消息来应答。然而，在这十年当中，一些轻客户端和其它软件都跳过了 inv 和 getdata 的步骤，仅发送未经请求的 tx 消息（[例子](https://github.com/bitcoinj/bitcoinj/blob/7d2d8d7792ec5f4ce07ff82980b4e723993221e8/core/src/main/java/org/bitcoinj/core/TransactionBroadcast.java#L145)）。
本周，Antoine Riard 在比特币开发者邮件列表中[发表](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018391.html)了一个提案，建议停止处理这些未经请求的 tx 消息。正如在 [Bitcoin Core PR](https://github.com/bitcoin/bitcoin/issues/20277) 中所讨论，这将给节点更多的控制权，让它在接收和处理事务的时候，有新方法来限制那些发送昂贵的验证事务的节点的影响。Riard 提议这一变更最早可在比特币核心的下一个主要版本，即 22.0 版本中进行。
此方案的缺点是，目前所有发送未经请求的 tx 消息的客户端将无法发送交易，除非它们在最后几个 0.21.x 或更早版本的 Bitcoin Core 离开网络之前升级（或 Bitcoin 的实现）。旧版本通常需要几年时间才能完全离开网络，所以应该有足够的时间来完成客户端的升级。我们鼓励受影响客户端的开发者阅读该提案并对其发表评论。

* **安全设置多签钱包：** Hugo Nguyen 在比特币开发者邮件列表[提出](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)一个 BIP 草稿，该草稿描述了钱包尤其是硬件钱包如何在多签钱包的签名者间安全地交换信息。这些需要交换的信息包括使用的脚本模板（例如 2-3 门限的 P2WSH）以及每个签名者的 [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 扩展公钥的密钥路径 (xpub)
简单来说，Nguyen 的提案（与一些硬件钱包制造商共同提出）有一个协调人通过选择脚本模板以及私有加密和认证凭证来初始化多签组的流程。这些参数通过成员的钱包共享，成员选择合适的密钥路径并提供其 xpub 与对应私钥的签名。每个成员的钱包将集合 {identifying_parameters, key_path, xpub, signature} 加密后返回给协调人。协调人将这些参数组合起来，并将加密后的[输出脚本描述符](https://bitcoinops.org/en/topics/output-script-descriptors/)返回给每个成员的钱包。成员分别验证他们的 xpub 包含在其中，并将描述符存储成他们愿意签名的脚本模板。
该提案在邮件列表中引发了大量的讨论，并计划进行一些修改。我们将观察讨论中的重要更新，以便随着提案的成熟逐渐迈向实施。
* **闪电网络上的托管：** 一年多前，在 Lightning-Dev 邮件列表上[发起](https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-June/002028.html)了关于为闪电网络创建非托管链上托管的[讨论](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-February/002948.html)，在上周讨论继续。讨论中引人关注的是 ZmnSCPxj 的一个[帖子](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-February/002955.html)，他使用[德摩根定律](https://en.wikipedia.org/wiki/De_Morgan's_laws)中的一个布尔语句法则，大大简化了托管——[可能](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-February/002970.html)还有许多其他基于闪电网络的合约——的构建，但代价是要求卖方在收到付款前提交保证金。ZmnSCPxj 的想法需要升级闪电网络以使用 [PTLCs](https://bitcoinops.org/en/topics/ptlc/)，目前预计要等到 [taproot](https://bitcoinops.org/en/topics/taproot/) 激活后才能实现。
* **重新讨论双向预付闪电网络费用问题：** Joost Jager [重启](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-February/002958.html)了闪电网络开发者邮件列表中关于增加闪电网络服务费的讨论，向花费者和接受者收取的这一服务费将由他们使用通道中的有限资金（"流动性"）和并行支付能力能力（"HTLC插槽"）所消耗的时间决定。Jager 以之前的提案为基础（见[通讯#122](https://bitcoinops.org/en/newsletters/2020/11/04/#bi-directional-upfront-fees-for-ln)），将其固定费用扩展到与支付处理时间成比例的费用（"占有费用"）。该建议得到了适度讨论，其中一个[关注点](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-February/002967.html)是将减少付款人/收款人的隐私。[这一问题](https://lists.linuxfoundation.org/pipermail/lightning-dev/2015-August/000135.html)已经存在了五年，并经过了多次详细的讨论，因此我们希望讨论能够继续下去。
* **反渗透：** Andrew Poelstra 和 Jonas Nick [发表](https://medium.com/blockstream/anti-exfil-stopping-key-exfiltration-589f02facc2e)了一篇博客文章，介绍了正在为 Shift Crypto [BitBox02](https://shiftcrypto.ch/bitbox02/) 和 Blockstream [Jade](https://blockstream.com/jade/) 硬件钱包实施的一项安全技术。其目标是让硬件钱包和与之相连的计算机签名时生成的 nonce 均是一个不可猜测的值。这可以防止恶意的硬件钱包固件生成固件作者已知的 nonce，作者可以将这些 nonce 与链上相应的此设备的事务签名结合起来，从而计算出私钥，便可以花费这些私钥控制的所有比特币。该帖子描述了所使用的技术，该技术曾在近一年前的一个邮件列表线程中提到过这个问题（见[通讯#87](https://bitcoinops.org/en/newsletters/2020/03/04/#proposal-to-standardize-an-exfiltration-resistant-nonce-protocol)和[#88](https://bitcoinops.org/en/newsletters/2020/03/11/#exfiltration-resistant-nonce-protocols)）。

## Changes to services and client software

*本月关于比特币的功能变化中，我们将重点介绍比特币钱包和服务的有意思的更新。*

* **Blockstream 宣布推出 [LNsync](https://blockstream.com/2021/01/22/en-lnsync-getting-your-lightning-node-up-to-speed-quickly/)：** 对于离线一段时间的 Lightning 钱包，LNsync 能够使其快速下载闪电网络的最新更新，以便其优化支付路由。一个开源插件 [historian](https://github.com/lightningd/plugins/tree/master/historian) 为 C-Lightning 用户提供了这个功能。

* **Rust 轻客户端 Nokamoto 发布：** Alexis Sellier 发布了 "[Nakamoto](https://cloudhead.io/nakamoto/)"，这是一个基于[简洁区块过滤器](https://bitcoinops.org/en/topics/compact-block-filters/)，“使用 Rust 实现的高保障比特币轻客户端，专注于更低的资源占用，模块化和安全”。

* **Blockstream Satellite 广播了闪电网络数据和比特币 Core 团队维护的源码：** Blockstream 的 Satellite 除了广播比特币区块链数据外，现在还包括[比特币 Core 团队维护的源码](https://blockstream.com/2021/02/02/en-blockstream-provides-backup-satellite-broadcast-for-bitcoin-core-source-code/)、为卫星广播优化了的比特币分叉（版本为 [Bitcoin Satellite](https://github.com/Blockstream/bitcoinsatellite)）的代码和[闪电网络 gossip 数据](https://blockstream.com/2021/01/29/en-new-blockstream-satellite-updates/)。
* **Blockstream Green实现CSV：** Green Wallet 此前使用 nLockTime 作为钱包恢复机制，要求用户在每笔交易后接收 Blockstream 发来的，含有预签名交易的备份邮件，以便在需要的时候恢复资金。 通过实现 [OP_CHECKSEQUENCEVERIFY(CSV) 恢复机制](https://blockstream.com/2021/01/25/en-blockstream-green-bitcoin-wallets-now-using-checksequenceverify-timelocks/)，现在可以恢复钱包，而不需要特定的交易备份文件或将电子邮件地址与钱包关联。
* **Muun 2.0发布：** 面向比特币和闪电网络的移动端钱包，Muun，的 [2.0 版本](https://medium.com/muunwallet/announcing-muun-2-0-d61b0844dc0a)包含了多签，钱包恢复功能，还开源了 Android 和 iOS 移动端应用源码。
* **Joinmarket 0.8.1发布：** [0.8.1 版本](https://github.com/JoinMarket-Org/joinmarket-clientserver/blob/master/docs/release-notes/release-notes-0.8.1.md)包含了对外部创建的 [PSBT](https://bitcoinops.org/en/topics/psbt/) 的额外支持，[书签](https://bitcoinops.org/en/topics/signet/)支持，以及对 [BIP21](https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki) URI 的大写地址支持（见[通讯#127](https://bitcoinops.org/en/newsletters/2020/12/09/#thwarted-upgrade-to-uppercase-bech32-qr-codes)）。 JoinMarket 的一个基于终端的 UI，[JoininBox](https://github.com/openoms/joininbox)，也更新到支持 0.8.1。
* **VBTC 允许通过 LN 和 segwit 批量提款：** 越南的一家交易所 VBTC 在最近[支持了闪电网络提币](https://blog.vbtc.exchange/2020/how-to-withdraw-bitcoin-lightning-network-tutorial-3)后，[增加了批量 segwit 提币选项](https://blog.vbtc.exchange/2021/batched-segwit-withdrawals-tutorial-5)。鼓励的 segwit 批量提现每周发生一次，目标在 mempools 上不太可能有大量交易积压的时间去做。
* **比特币开发工具包 [v0.3.0](https://bitcoindevkit.org/blog/2021/01/release-v0.3.0/) 发布：** Rust 钱包库 Bitcoin Dev Kit 宣布发布 v0.3.0 版本，包括将 CLI 拆分出自己的仓库。 最近的 [BDK v0.2.0 版本](https://bitcoindevkit.org/blog/2020/12/release-v0.2.0/)增加了分支和绑定 (BnB) 币的选择，[描述符](https://bitcoinops.org/en/topics/output-script-descriptors/)模板（包括最近增加的多排序）等。

## Releases and release candidates

*流行的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

* [LND 0.12.1-beta.rc1](https://github.com/lightningnetwork/lnd/releases/tag/v0.12.1-beta.rc1) 是 LND 维护的候选版本。 它修复了可能导致通道意外关闭的极端情形和可能导致不必要的支付失败的bug，以及一些其他小的改进和 bug 修复。

## Notable code and documentation changes


*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[硬件钱包接口 (HWI)](https://github.com/bitcoin-core/HWI)、[Rust 比特币](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay 服务器](https://github.com/btcpayserver/btcpayserver/)、[比特币改进提案 (BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 的显著更新。*

* [Bitcoin Core #20944](https://github.com/bitcoin/bitcoin/issues/20944) 在 getmempoolinfo RPC 和 mempool/info REST 端点返回的对象中增加了一个新的 total_fee 字段。total_fee 表示当前 mempool 中所有交易的交易费用总和。

* [LND #4909](https://github.com/lightningnetwork/lnd/issues/4909) 增加了新的 getmccfg 和 setmccfg RPC，分别可以在不重启守护进程的情况下，检索和临时更改 LND 任务控制子系统中的设置。任务控制模块利用过去尝试付款的信息，为以后的付款尝试选择更好的路由。

* [Rust-Lightning #787](https://github.com/rust-bitcoin/rust-lightning/issues/787) 确保只有当发送消息的节点是通道的交易方时，才会发生因错误消息导致的信道关闭。 以前，如果恶意节点知道通道 id，就可以强制关闭任何通道。

* [BTCPay Server #2164](https://github.com/btcpayserver/btcpayserver/issues/2164) 重新设计了钱包设置向导，引导用户设置 BTCPay 的内部钱包，并可选择将其与用户自己的远程软件或硬件钱包整合。这是 BTCPay 界面重新设计的开始。

* [HWI #443](https://github.com/bitcoin-core/HWI/issues/443) 增加了对用 BitBox02 硬件钱包多签签名输入的支持。

* 比特币 Core 团队在 [#19145](https://github.com/bitcoin/bitcoin/issues/19145) 扩展了 gettxoutsetinfo RPC 的 hash_type 选项，来接受一个新的 muhash 参数，该参数将产生当前区块高度的 UTXO 集的 [MuHash3072](https://bitcoinops.org/en/newsletters/2021/01/13/#bitcoin-core-19055) 摘要。相对于默认产生 UTXO 集的 SHA256 摘要，这是另一种选择。更新完成后，MuHash 要处理的数据量和 SHA256 函数一样，所以预计 gettxoutsetinfo RPC 的性能不会有明显的变化，在速度慢的单板电脑上，MuHash 的返回时间可能需要几分钟。然而，一个先前计算过的MuHash 对象可以相对便宜增加或者删除元素。因此，将来的 PR（代码提交）有望快速计算每个块上设置的 UTXO 的 MuHash 摘要，将这些摘要保存在简单的数据库中，并允许根据需求，以摘要形式快速返回它们。这也将使 [AssumeUTXO](https://bitcoinops.org/en/topics/assumeutxo/) 项目受益，该项目依赖于用户对比过去不同区块的 UTXO 集的哈希的能力。

* [C-Lightning＃4364](https://github.com/ElementsProject/lightning/issues/4364) 更改了通知通道合作方的方式，将现有的错误消息分为真正的错误和仅是警告的错误。 当前的 [BOLT1](https://github.com/lightningnetwork/lightning-rfc/blob/master/01-messaging.md) 规范要求任何错误都会导致通道关闭（尽管这似乎并未普遍实现），但是一个[被提案的规范更新](https://github.com/lightningnetwork/lightning-rfc/issues/834)引入了警告消息类型，可以允许更灵活的响应。任何有兴趣的人也可以阅读 Carla Kirk-Cohen 本周在 Lightning-Dev 邮件列表中[发布](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-February/002964.html)的有关扩展错误消息说明的帖子，她将其描述为“与软警告错误相关但不直接相关的”主题。