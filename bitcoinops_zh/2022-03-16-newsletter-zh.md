---
title: 'Bitcoin Optech Newsletter #191'
permalink: /zh/newsletters/2022/03/16/
name: 2022-03-16-newsletter-zh
slug: 2022-03-16-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---

本周的周报介绍了使用新的操作码来拓展或者替代 Bitcoin Script（比特币脚本语言）的提议，并总结了最近围绕提升 RBF 方法的讨论，还链接了  ` OP_CHECKTEMPLATEVERIFY ` 操作码上的持续工作。此外，还包括我们的常规部分：流行的比特币基础设施项目的重要变更。

## 新闻

- **Bitcoin Script 的拓展和替代方案**：多个开发者在 Bitcoin-Dev 邮件组中讨论了所提升比特币的 Script 和 [tapscript][tapscript] 语言的想法。这些编程语言是用来编程脚本，让作为收款的比特币指定后续花费行为的授权要求的。

  - *Looping（folding）*：开发者 ZnmSCPxj [提出了][described]一个 ` OP_FOLD ` 操作码，作为一种在 Bitcoin Script 中允许类似循环（loop）的行为的方法。他介绍了可以在循环上施加的一系列限制，可保证这些循环使用到的 CPU 和内存不会超过比特币 Script 和 tapscript 当前可用的数量 —— 但因为不再需要在脚本中加入重复的代码，可以节省带宽。
  - 使用 *Chia Lisp*：Anthomy Towns [发帖][posted]提出为比特币加入 [Chia Lisp][Chia Lisp] 的一个变种。Chia Lisp 是一个专为 Chia 区块链设计的 [Lisp][Lisp] 语言变种。那么，它将是跟传统的 Script 和 tapscript 完全不同的替代方案；因为它也是一个全新的基础，它能提供的部分好处与之前人们提议的 [Simplicity][Simplicity] 语言相同。Towns 声称他的替代方案 —— “二进制树编码脚本（Binary Tree Coded Script）” 或者说 “btc-script”比 Simplicity 更容易理解和使用，虽然它可能更难形式化验证。

- **提升 RBF 方法的想法**：Golria Zhao [发帖][posted]总结了最近在伦敦发生的 

  [CoreDev.Tech][CoreDev.Tech] 会议上关于 “手续费替换（Replace-by-Fee，[RBF][RBF]）” 方法的讨论，以及一些相关的升级。她报告称，大家讨论的核心概念是尝试限制交易和它们的替代交易的广播所用的资源上限，比如限制在一段时间内可以得到转发的相关交易的数量。

  Zhao 还总结了一个发生在一个 gist 页面的专门[讨论][discussion]，该讨论测试了允许交易指明后代可用限制的情形。举个例子，一笔交易可以指明它以及它的后代可以消费的交易池空间上限为 1000 vbytes（而不是默认的 10 0000 vbytes）。这将使诚实的一方在 “[交易钉死攻击][pinning attack]” 最糟糕的情况下可以用小一些的代价来解决。

  此外，Zhao 也在为一种计算一笔交易在给定的交易池中对矿工的价值的算法寻求反馈。这种算法可以帮助节点在选择接受或拒绝一笔替换交易时做出更灵活的决定。

- **持续进行的 CTV 讨论**：如我们在[周报 #183][Newsletter #183]中提到的，讨论 [OP_CHECKTEMPLATEVERIFY][OP_CHECKTEMPLATEVERIFY]（CTV）操作码提案的会议仍在继续，Jeremy Run 提供了一系列总结：[1][1]、[2][2]、[3][3]、[4][4] 和 [5][5]、此外，在上周，James O'Beirne [提出][posted]了一种基于 CTV 的[金库][vault]的代码和设计文档。

## 重大的代码和文档更新

本周出现重大变更的有：[Bitcoin Core][Bitcoin Core]、[C-Lightning][C-Lightning]、[Eclair][Eclair]、[LDK][LDK]、[LND][LND]、[libsecp256k1][libsecp256k1]、[Hardware Wallet Interface (HWI)][Hardware Wallet Interface (HWI)]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK]、[Bitcoin Improvement Proposals (BIPs)][Bitcoin Improvement Proposals (BIPs)]，以及 [Lightning BOLTs][Lightning BOLTs]。

- [Bitcoin Core #24198][Bitcoin Core #24198] 使用一个新的字段 ` wtxid ` 拓展了  ` listsinceblock`、`listtransactions` 和 `gettransaction ` RPC 方法，该新字段包含了每笔交易的 Witness Transaction 标识符（由 [BIP141][BIP141] 定义）。
- [Bitcoin Core #24043][Bitcoin Core #24043] 加入了一个新的  ` multi_a `  和 ` sortedmulti_a ` [描述符][descriptors]，使得创建花费授权的方法可以跟 [tapscript 的][tapscript’s]  ` OP_CHECKSIGADD ` 操作码一起工作，而不是使用更老的 Script 的  ` OP_CHECKMULTISIG ` 和  ` OP_CHECKMULTISIGVERIFY ` 操作码。阅读[周报 #46][Newsletter #46] 了解关于 tapscript 这方面的更多信息。
- [Bitcoin Core #24304][Bitcoin Core #24304] 加入了一个新的 demo 实现：可抽取的  ` bitcoin-chainstate ` ，可以传递一个 Bitcoin Core 的数据目录和一个区块，它也会验证这个区块并添加到数据目录中。人们没期望这东西有什么直接的用场，但 [libbitcoinkernel][libbitcoinkernel] 项目将利用这个工具来产生一个其它项目也可以用 Bitcoin Core 所用的代码来验证区块和交易的库。
- [C-Lightning #5068][C-Lightning #5068] 提高了 [BOLT7][BOLT7]  `node_announcement ` 消息的最小数量；C-Lightning 的每个节点每天都会转发这种消息一到两次。这可能会缓解跟节点切换 IP 地址或短暂下线维护相关的一些问题。
- [BIPs #1269][BIPs #1269] 将 [BIP326][BIP326] 定性为一种建议：在需要使用 [BIP68][BIP68] “共识保证实施的 nSequence 值” 时，[taproot][taproot] 交易可以设定一个 nSequence 值来提高隐私性，即使不需要用于合约协议时也是如此。BIP326 也描述了如何使用 nSequence 来提供替代性的[抗 “手续费狙击（fee sniping）”][anti fee sniping]保护措施（当前，这种措施是通过交易的 locktime 字段来提供的）。请看[周报 #153][Newsletter #153] 对该提议发表在邮件组的初版的总结。

[tapscript]:https://bitcoinops.org/en/topics/tapscript/

[described]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/020021.html

[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020036.html

[Chia Lisp]:https://chialisp.com/

[Lisp]:https://en.wikipedia.org/wiki/Lisp_(programming_language

[Simplicity]:https://bitcoinops.org/en/topics/simplicity/

[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020095.html

[RBF]:https://bitcoinops.org/en/topics/replace-by-fee/

[CoreDev.Tech]:https://coredev.tech/

[discussion]:https://gist.github.com/glozow/25d9662c52453bd08b4b4b1d3783b9ff?permalink_comment_id=4058140#gistcomment-4058140

[pinning attack]:https://bitcoinops.org/en/topics/transaction-pinning/

[Newsletter #183]:https://bitcoinops.org/en/newsletters/2022/01/19/#irc-meeting

[OP_CHECKTEMPLATEVERIFY]:https://bitcoinops.org/en/topics/op_checktemplateverify/

[1]:https://bitcoinops.org/en/newsletters/2022/01/19/#irc-meeting

[2]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019855.html

[3]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019874.html

[4]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019974.html

[5]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020086.html

[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020067.html

[vault]:https://bitcoinops.org/en/topics/vaults/

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

[Bitcoin Core #24198]:https://github.com/bitcoin/bitcoin/issues/24198

[BIP141]:https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki

[Bitcoin Core #24043]:https://github.com/bitcoin/bitcoin/issues/24043

[descriptors]:https://bitcoinops.org/en/topics/output-script-descriptors/

[tapscript’s]:https://bitcoinops.org/en/topics/tapscript/

[Newsletter #46]:https://bitcoinops.org/en/newsletters/2019/05/14/#new-script-based-multisig-semantics

[Bitcoin Core #24304]:https://github.com/bitcoin/bitcoin/issues/24304

[libbitcoinkernel]:https://github.com/bitcoin/bitcoin/issues/24303

[C-Lightning #5068]:https://github.com/ElementsProject/lightning/issues/5068

[BOLT7]:https://github.com/lightningnetwork/lightning-rfc/blob/master/07-routing-gossip.md

[BIPs #1269]:https://github.com/bitcoin/bips/issues/1269

[BIP326]:https://github.com/bitcoin/bips/blob/master/bip-0326.mediawiki

[taproot]:https://bitcoinops.org/en/topics/taproot/

[BIP68]:https://github.com/bitcoin/bips/blob/master/bip-0068.mediawiki

[anti fee sniping]:https://bitcoinops.org/en/topics/fee-sniping/

[Newsletter #153]:https://bitcoinops.org/en/newsletters/2021/06/16/#bip-proposed-for-wallets-to-set-nsequence-by-default-on-taproot-transactions

