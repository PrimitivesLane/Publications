---
title: 'Bitcoin Optech Newsletter #182'
permalink: /en/newsletters/2022/01/12/
name: 2022-01-12-newsletter
slug: 2022-01-12-newsletter
type: newsletter
layout: newsletter
lang: zh
---
本周的 Newsletter 介绍了一个在比特币中添加账户以支付交易费用的想法，还包括我们的常规部分，其中有比特币核心 PR 审核俱乐部会议的总结和主流的比特币基础设施软件中值得注意的变更。

## 新闻

- **费用账户：** Jeremy Rubin 在 Bitcoin-Dev 邮件列表中[发布][rubin feea]了一个关于软分叉的粗略想法，这个想法可以使预设交易更容易提升费用，比如那些在 LN 和其他合约协议中使用的场景下。这个想法是他在 [Newsletter #116][news116 sponsorship] 中描述的交易赞助想法的一个衍生。

  费用账户的基本想法是，用户可以创建交易，将比特币存入一个由了解新共识规则的升级后的全节点跟踪的账户。当用户随后想在交易中提升费用时，他们将签署一个简短的信息，包含他们想支付的金额和该交易的 txid。已升级的全节点将允许同时包含交易和签名信息的区块向该区块的矿工支付已签名的费用金额。

  Rubin 表示，这将消除 [CPFP][topic cpfp] 和 [RBF][topic rbf] 费用更新的许多问题，这些问题与两个或更多用户共享 UTXO 所有权的合约协议有关，或者在其他情况下，使用预签名交易意味着在过去签署交易时不可能知道当前的网络费率。



## Bitcoin Core PR 审核俱乐部会议

*在这个每月一次的栏目中，我们总结最近一次的 [Bitcoin Core PR 审核俱乐部会议][Bitcoin Core PR Review Club]，集中展示一些重要的问题和答案。点击具体的问题可看到会议答案的总结。*

[Erlay 支持信号][reviews 23443]是 Gleb Naumenko 的一个 PR，为 p2p 代码添加交易调解协商。它是在比特币核心中增加对 [Erlay][topic erlay] 支持的一系列 PR 中的一个。评审会会议讨论了调解握手协议，并权衡了将大项目拆分成小块的利弊。

<details><summary>将 PR 拆成小块有什么好处？这种方法有什么缺点吗？
</summary>
将一个大的 PR 拆成小块，有助于鼓励在合并前对 PR 进行更集中、更彻底的评审，而不会迫使评审人员一次评审巨大的修改集，而且可以减少由于 Github 可扩展性问题而遇到评审障碍的可能性。无争议和简单的代码修改可以更快地被合并，而有争议的部分可以更多地得到讨论。然而，除非评审员同意全部的修改内容，否则他们是相信作者把他们带向正确的方向。另外，由于合并不是原子性的，作者需要确保中间状态不是不安全的或做一些无意义的事情，比如在节点真正能够进行调解之前宣布支持 Erlay。<a href="https://bitcoincore.reviews/23443#l-29">➚</a>
</details>

<details><summary>节点应该在什么时候宣布支持调解？
</summary>
节点只有在这个连接上的交易中继开启的情况下，才应该向对等节点发送 <code>sendrecon</code>：节点不是在 blocksonly 的模式下，这不是一个 blocksonly 中继的连接，并且对等节点没有发送 <code>fRelay=false</code>。对等节点还必须支持见证交易标识符（wtxid）中继，因为交易调解的框架是基于交易 wtxid 的。<a href="https://bitcoincore.reviews/23443#l-51">➚</a>
</details>

<details><summary>整个握手和“注册调解”协议的流程是什么？
</summary>
在 <code>version</code> 消息发送之后，<code>verack</code> 消息发送之前，对等节点各自发送一个 <code>sendrecon</code> 消息，其中包含他们本地生成的盐等信息。没有强制的顺序；任何一个对等节点都可以先发送。如果一个节点发送并收到有效的 <code>sendrecon</code> 消息，它应该为该对等节点初始化调解状态。<a href="https://bitcoincore.reviews/23443#l-63">➚</a>
</details>

<details><summary>为什么 Erlay 不包括 p2p 协议版本升级？
</summary>
新的协议版本并不是工作的必要条件；使用 Erlay 的节点不会与现有协议不兼容。不理解 Erlay 消息（如 <code>sendrecon</code>）的旧节点，会忽略它们，但仍然能够正常工作。<a href="https://bitcoincore.reviews/23443#l-78">➚</a>
</details>

<details><summary>为什么要为每个连接生成本地的盐？它是如何生成的？
</summary>
一个连接的调解盐是两个对等节点本地生成的盐的组合，用于为每个交易创建短标志符。用于短标志符的盐散列函数被设计为能有效地创建紧凑的标志符，但如果攻击者能够控制盐的内容，就不能保证抗碰撞安全。当双方都提供盐的一部分内容时，没有第三方可以控制盐是什么。在本地，每个连接都会产生一个新的盐，所以节点不能以这种方式被指纹化。<a href="https://bitcoincore.reviews/23443#l-93">➚</a>
</details>


## 代码和文档的重大变更

*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Bitcoin Core #23882][Bitcoin Core #23882] 包含对 `validation.cpp` 中关于 testnet3 操作的文档的更新。

  在比特币的原始版本中，交易有可能具有相同的内容，因此有可能碰撞 txid。重复的问题尤其可能发生在币基交易中，因为每笔币基交易的输入和输出的组成部分是相同的，而其他部分则完全由区块模板的创建者决定。主网区块链包含两个重复的币基交易，高度为 91,842 和 91,880。它们与之前的币基交易相同，并在花费之前覆盖了已有的币基输出，使可用的总供应量减少了 100 BTC。这些事件促使了 [BIP30][] 的引入，它禁止重复的交易。BIP30 的[实现][bip30-impl]是通过检查每笔交易是否已经有 UTXO 在相应的 txid 中来实现的。随后引入的 [BIP34][] 有效地防止了重复的问题，它要求区块的高度作为币基交易 scriptSig 的第一项。由于高度是唯一的，币基的内容在不同的高度上不可能再相同，这也从根本上防止了后代交易中的这个问题。因此，它消除了对重复内容进行额外检查的需要。

  后来证明，[BIP34][] 的缺陷在于，在 [BIP34][] 推出之前，已经存在一些符合未来区块高度模式的币基交易。矿工能够产生违反 BIP30 的碰撞的第一个区块高度是在 1,983,702，我们预计在主网在 2040 年之后。然而，testnet3 已经超过了 1,983,702 的高度。因此，比特币核心恢复了对每笔 testnet 交易进行重复未花费交易的检查。

- [Eclair #2117][Eclair #2117] 增加了对处理[洋葱消息][topic onion messages]回复的支持，为支持[要约协议][topic offers]做准备。

- [LND #5964][LND #5964] 增加了一个 `leaseoutput` RPC，告诉钱包在指定的时间内不要花费指定的 UTXO。这与其他钱包软件提供的 RPC 类似，比如比特币核心的 `lockunspent`。

- [BOLTs #912][BOLTs #912] 为 BOLT11 收据增加了一个新的可选字段，用于存放接收方提供的元数据。如果在收据中使用该字段，花费节点可能需要将元数据包括在它通过网络传送给接收者的支付消息中。然后，接收方可以使用元数据作为处理付款的一部分，例如[一开始提出的][news168 stateless]使用该信息来实现[无状态收据][topic stateless
  invoices]。

- [BOLTs #950][BOLTs #950] 为 [BOLT1][] 引入了向后兼容的警告信息，减少了发送致命错误的要求，避免了不必要的通道关闭。这是迈向更标准化和更丰富的错误的类型第一步。更多的讨论可以在 [BOLTs #834][] 和 [Carla Kirk-Cohen][Error Codes for LN] 在 Lightning-dev 邮件列表中的帖子中找到（见 [Newsletter #136][news136 warning post]）。


[topic cpfp]: https://bitcoinops.org/en/topics/cpfp/
[topic rbf]: https://bitcoinops.org/en/topics/replace-by-fee/
[topic erlay]: https://bitcoinops.org/en/topics/erlay/
[topic onion messages]: https://bitcoinops.org/en/topics/onion-messages/
[topic offers]: https://bitcoinops.org/en/topics/offers/
[topic stateless invoices]: https://bitcoinops.org/en/topics/stateless-invoices/
[Bitcoin Core #23882]: https://github.com/bitcoin/bitcoin/pull/23882
[Eclair #2117]: https://github.com/ACINQ/eclair/pull/2117
[LND #5964]: https://github.com/lightningnetwork/lnd/pull/5964
[BOLTs #912]: https://github.com/lightning/bolts/issues/912
[BOLTs #950]: https://github.com/lightning/bolts/pull/950
[BIP30]: https://github.com/bitcoin/bips/blob/master/bip-0030.mediawiki
[BIP34]: https://github.com/bitcoin/bips/blob/master/bip-0034.mediawiki
[BOLT1]: https://github.com/lightning/bolts/blob/master/01-messaging.md
[BOLTs #834]: https://github.com/lightning/bolts/issues/834

[Bitcoin Core PR Review Club]:https://bitcoincore.reviews/
[rubin feea]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019724.html
[news116 sponsorship]: /en/newsletters/2020/09/23/#transaction-fee-sponsorship
[news168 stateless]: /en/newsletters/2021/09/29/#stateless-ln-invoice-generation
[reviews 23443]: https://bitcoincore.reviews/23443
[Error Codes for LN]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-February/002964.html
[news136 warning post]: /en/newsletters/2021/02/17/#c-lightning-4364
[bip30-impl]: https://github.com/bitcoin/bitcoin/pull/915/files

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

