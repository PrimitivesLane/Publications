---
title: 'Bitcoin Optech Newsletter #199'
permalink: /zh/newsletters/2022/05/11/
name: 2022-05-11-newsletter-zh
slug: 2022-05-11-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---

本周的简报总结了一场 Bitcoin Core PR 审核俱乐部会议的成果，并介绍了 Rust Bitcoin 的一项升级。

## 新闻

本周没有重大的新闻。我们之前介绍过的主题，包括 [OP_CHECKTEMPLATEVERIFY][OP_CHECKTEMPLATEVERIFY] 和 [SIGHASH_ANYPREVOUT][SIGHASH_ANYPREVOUT]，都得到了新的评论 —— 但大部分讨论要么是非技术性的，要么是跟我们认为并无广泛影响的小细节有关。在本周报编辑之时，我们在开发者邮件组中收到了许多有趣的帖子；我们会在下一周作深入介绍。

## Bitcoin Core PR 审核俱乐部

*在这个每月一次的栏目中，我们会总结最近的一期 [Bitcoin Core PR 审核俱乐部][Bitcoin Core PR Review Club]会议，提炼出一些重要的问题和答案。点击下方的问题描述，就可以看到来自会议的答案。*

[“通过剪枝分块器来优化剪枝节点的索引”][review club pr] 是 Fabian Jahr 提出的 PR，引入了一种新方法来判断什么时候才区块存储中修建一个区块是安全的。这种新的方法让剪枝节点可以维护一个 Coinstats 索引，并移除验证模块对索引相关代码的依赖。

<details>
	<summary>
	现在 Bitcoin Core 里面使用什么索引，它们有什么用？
	</summary>
	节点可以选择维护最多三种索引来帮助高效从硬盘中检索数据。交易索引（`-txindex`）将交易的哈希值映射到该交易所在的区块。区块过滤器索引（`-blockfilterindex`）将 BIP157 过滤器与每个区块关联起来。资金状态索引（`-coinstatsindex`）存储 UTXO 集的统计数据。<a href="https://bitcoincore.reviews/21726#l-28">➚</a>
</details>

<details>
	<summary>
	什么是 “循环依赖”？为什么我们要尽可能避免它？
	</summary>
	“循环依赖” 是指两个代码模块互相依赖、缺了对方就不能工作。虽然循环依赖不是安全隐患， 但因为它会让代码更难开发、使用和在隔离环境中测试特定的模块和功能，它会大幅劣化代码的协调以并阻碍开发。<a href="https://bitcoincore.reviews/21726#l-44">➚</a>
</details>

<details>
	<summary>
	那么<a href="https://github.com/bitcoin-core-review-club/bitcoin/commit/527ef4463b23ab8c80b8502cd833d64245c5cfc4">这个 PR</a> 中的剪枝分块器是怎么工作的呢？
	</summary>
	这次的  PR 引入了一系列的 “剪枝锁”，是为使每一个索引正常工作而需保存的最早区块的区块高度。在 `CChainState::FlushStateToDisk` 中，当节点要决定是否要修剪一个区块时，它会避免修剪高于剪枝锁高度的区块。每次索引更新其对最优区块索引的视角时，剪枝锁也随之更新。<a href="https://bitcoincore.reviews/21726#l-68">➚</a>
</details>

<details>
	<summary>
	这种剪枝的新方法和旧的相比，有什么好处？开销有何变化？
	</summary>
	以前， `CChainState::FlushStateToDisk` 中的逻辑会查询各索引的最优高度，以决定剪枝到哪个区块就停止；索引和验证逻辑时相互依赖的。现在，剪枝锁是主动更新的，因此可能会计算得更加频繁，但不再需要验证模块来查询索引。
</details>

## 代码和文档的重大变更

*本周出现重大变更的有：[Bitcoin Core][Bitcoin Core]、[Core Lightning][Core Lightning]、[Eclair][Eclair]、[LDK][LDK]、[LND][LND]、[libsecp256k1][libsecp256k1]、[硬件钱包接口（HWI）][Hardware Wallet Interface (HWI)]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK]、[比特币升级提议（BIP）][Bitcoin Improvement Proposals (BIPs)] 以及  [Lightning BOLTs][Lightning BOLTs]。*

- [Rust Bitcoin #716][Rust Bitcoin #716] 加入了 ` amount::Display ` ，一个可配置的 Dsiplay（显示）类型，用于显示面额和其它向用户展示的数个。这个补丁默认将所有数字的展示都减少到最小的快如，因此减少了对多余的 0 的使用（它们会导致所生成的 [BIP21][BIP21] URI 长得没有必要，并进一步使所生成的二维码更大、能难以扫描。

[review club commit]: https://github.com/bitcoin-core-review-club/bitcoin/commit/527ef4463b23ab8c80b8502cd833d64245c5cfc4
[review club pr]: https://bitcoincore.reviews/21726

[OP_CHECKTEMPLATEVERIFY]: https://bitcoinops.org/en/topics/op_checktemplateverify/
[SIGHASH_ANYPREVOUT]: https://bitcoinops.org/en/topics/sighash_anyprevout/
[Bitcoin Core PR Review Club]: https://bitcoincore.reviews/
[Bitcoin Core]: https://github.com/bitcoin/bitcoin
[Core Lightning]: https://github.com/ElementsProject/lightning
[Eclair]: https://github.com/ACINQ/eclair
[LDK]: https://github.com/lightningdevkit/rust-lightning
[LND]: https://github.com/lightningnetwork/lnd/
[libsecp256k1]: https://github.com/bitcoin-core/secp256k1
[Hardware Wallet Interface (HWI)]: https://github.com/bitcoin-core/HWI
[Rust Bitcoin]: https://github.com/rust-bitcoin/rust-bitcoin
[BTCPay Server]: https://github.com/btcpayserver/btcpayserver/
[BDK]: https://github.com/bitcoindevkit/bdk
[Bitcoin Improvement Proposals (BIPs)]: https://github.com/bitcoin/bips/
[Lightning BOLTs]: https://github.com/lightning/bolts
[Rust Bitcoin #716]: https://github.com/rust-bitcoin/rust-bitcoin/issues/716
[BIP21]: https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki
