---
title: 'Bitcoin Optech Newsletter #195'
permalink: /zh/newsletters/2022/04/13/
name: 2022-04-13-newsletter-zh
slug: 2022-04-13-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---

本周的周报介绍了一套在比特币交易和闪电支付中传输非比特币的 token 的协议，并链接了一个关于 MuSig 2 多签名协议的 BIP。此外还有我们的常规栏目：Bitcoin Core PR 审核俱乐部会议的总结、软件的新版本和候选版本，以及热门的比特币基础设施软件的重大变更。

## 新闻

- **可以转移 token 的方案**：Olaoluwa Osuntokun 在 Bitcoin-Dev 邮件组和 Lightning-Dev 中[发帖][posted]列举了关于 *Taro* 协议的一组 BIP。Taro协议将允许在比特币区块链上记录非比特币的 token的创造和转移。举个例子，Alice 可以发行 100 个 token，转移 50 个给 Bob，然后 Bob 可以拿其中 25 个来跟 Carol 交换 10 BTC。想法上跟以前已经在比特币上实现的类似，但细节有所不同，比如，它复用了来自 [taproot][taproot] 的几种设计元素来减少需要重写的新代码的数量、使用默克尔树来减少证明某些操作已经发生的时候需要传输的数据量。

  Taro [最初设想][intended]的使用场景是闪电网络的可路由链下转账。跟以往提出的在闪电网络上做跨资产转账的提议相似，只是路由支付的中间节点不需要理解 Taro 协议和被转移的资产的详情 —— 只需跟其他的闪电网络支付使用同样的协议即可。

- **MuSig2 提议 BIP**：Jonas Nick、Tim Ruffing 和 Elliott Jin 在 Bitcoin-Dev 邮件组中[发帖][posted]介绍了为 [MuSig2][MuSig2]（一种创建公钥和签名的[多签名][multisignature]协议）[提议的 BIP][proposed BIP]。由相互独立的多方或多个软件控制的多个私钥可以使用 MuSig2 来推导出一个聚合公钥，并且无需任何一方向其他人暴露任何私有信息。而后，聚合签名也可以创建出来，并且也不需要任何人暴露私有信息。这个聚合公钥和聚合签名在第三方看来，就跟普通的公钥和 [schnorr 签名][schnorr signature]没有区别，所以它不会暴露有多少方、多少私钥参与到了创建聚合公钥和签名的过程中，提高了链上多签名的隐私性（原本的链上多签名会暴露独立公钥和签名的数量）。

  MuSig2 在几乎所有能够设想出来的应用上，都优于原始的 MuSig 协议（现在叫做 “MuSig1”）。MuSig2 比替代方案 MuSig-DN（确定性 nonce）更容易实现，虽然两者有些取舍，需要一些应用的开发者因地制宜。

## Bitcoin Core PR 审核俱乐部

*在这个每月一次的栏目中，我们会总结最近的一期 [Bitcoin Core PR 审核俱乐部][Bitcoin Core PR Review Club]会议，提炼出一些重要的问题和答案。点击下方的问题描述，就可以看到来自会议的答案。*

“[通过发送额外的 getheaders 消息来防止基于区块索引的设备指纹识别][Prevent block index fingerprinting by sending additional getheaders messages]” 是由 Niklas Gögge 提出的 PR，用意是防止节点遭受基于其区块索引的指纹识别攻击。

<details>
	<summary>
	什么是区块索引？有什么用？
	</summary>
	区块索引是放在内存种的索引，用于查找区块头和区块在硬盘中的位置。在区块 “树” 上可能会有多个分支（即它可能保留多个分支，包括陈旧的区块同）来适用区块链重组。<a href="https://bitcoincore.reviews/24571#l-17">➚</a>
</details>
<details>
	<summary>
	我们应该在区块索引中保留陈旧区块吗？支持理由和反对理由都有哪些？
	</summary>
	当区块链存在多个分支时，把多个分支都保存在区块索引中，可以帮助节点在分支间快速切换（假设最多工作量的分支发生了变化）。会议的一些参与者指出，保留非常陈旧的区块可能没什么用，因为发生这么深的重组的概率是非常小的。但是，这些区块头仅占用非常小的存储空间，而且，因为节点会在存储它们之前检查工作量证明，想通过发送具备有效工作量证明的陈旧区块头来耗尽节点的资源是完全划不来的。<a href="https://bitcoincore.reviews/24571#l-68">➚</a>
</details>

<details>
	<summary>
	介绍一下使用一个节点的区块索引的指纹识别攻击。
	</summary>
	在初次区块同步期间，节点只请求和下载（自己所知）最多工作量的链上的区块。因此，其区块索引中的陈旧区块通常是在初次区块同步之后挖出的，但是它很可能会自然发生变化，或者被一个拥有大量过时区块头的攻击者操纵。拥有陈旧区块头 H 和 H+1 的攻击者可以发送 H+1 给目标节点，如果该节点的区块索引中没有 H+1 的祖先区块，它就会使用 <code>getdata</code> 请求 H。如果它已经有 H 了，就不会发起这种请求。<a href="https://bitcoincore.reviews/24571#l-75">➚</a>
</details>

<details>
	<summary>
	为什么有必要防范节点的指纹识别攻击？
	</summary>
	节点的运营者可能使用了多种技术来混淆自己的节点 IP 地址，例如使用 Tor。但是，如果这种攻击可以将（同时运行在两个网络中的）节点的 IPv4 和 Tor 地址关联起来，那么这些隐私技术的好处就将非常有限。如果节点只运行在 Tor 网络中，指纹识别可能会让属于同一个节点的多个 Tor 地址关联起来，或者在节点迁移到 IPv4 网络的时候识别出它。<a href="https://bitcoincore.reviews/24571#l-84">➚</a>
</details>
## 新版本和候选版本

*热门的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- [LDK 0.0.106][LDK 0.0.106] 是这个闪电网络节点库的最新版本。加入了对 [BOLTs #910][BOLTs #910] 所提议的通道识别符 ` alias ` 字段的支持，LDK 用它在一些情况下提升隐私性。还纳入了多项其它特性和 bug 修复。
- [BTCPay Server 1.4.9][BTCPay Server 1.4.9] 是这个流行的支付处理软件的最新版本。
- [Bitcoin Core 23.0 RC4][Bitcoin Core 23.0 RC4] 是这个主导性全节点软件的下一个大版本的候选。[更新说明草稿][draft release notes]例举了多项提升，鼓励高级用户和系统管理员们在最终发布前[帮助测试][test]。
- [LND 0.14.3-beta.rc1][LND 0.14.3-beta.rc1] 是这个流行的闪电网络节点软件的候选版本，修复了多个 bug。
- [Core Lightning 0.11.0rc1][Core Lightning 0.11.0rc1] 是这个流行的闪电网络节点软件的下一个大版本的候选。

## 重大的代码和文献变更。

本周出现重大变更的有：[Bitcoin Core][Bitcoin Core]、[Core Lightning][Core Lightning]、[Eclair][Eclair]、[LDK][LDK]、[LND][LND]、[libsecp256k1][libsecp256k1]、[Hardware Wallet Interface (HWI)][Hardware Wallet Interface (HWI)]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK]、[Bitcoin Improvement Proposals (BIPs)][Bitcoin Improvement Proposals (BIPs)] 和 [Lightning BOLTs][Lightning BOLTs]。

- [Bitcoin Core #24152][Bitcoin Core #24152] 通过引入[交易包费率][package feerate]并用它替代单体交易费率，在包含未确认父交易和子交易的交易包种启用了 [CPFP][CPFP]（子为父偿手续费追加方法）。如同 [186 期周报][newsletter #186]所说，这是为加强 CPFP 和 [RBF][RBF]（手续费替换）两种手续费追加方法的灵活性和可靠性的一系列强化措施的一部分。这个补丁[也会先验证单体交易][validates individual transactions first]，以避免像 “父为子偿” 和 “同辈互偿” 这样的激励不兼容的花费策略。
- [Bitcoin Core #24098][Bitcoin Core #24098] 更新了 ` /rest/headers/ ` 和 ` /rest/blockfilterheaders/ ` PRC 方法，以为额外的操作（例如 ` ?count=<count> ` ）使用查询参数，作为对端点（例如 ` /<count/ ` ）的替代。文档已经更新，鼓励在端点参数种使用查询参数。
- [Bitcoin Core #24147][Bitcoin Core #24147] 加入了对 [miniscript][miniscript] 的后端支持。后续的 [#24148][#24148] 和 [#24149][#24149] PR，如果得以合并，将在[输出脚本描述符][output script descriptors]和钱包的签名逻辑中增加对 miniscript 的支持。
- [Core Lightning #5165][Core Lightning #5165] 将 C-Lightning 项目更名为 “[Core Lightning][Core Lightning]”，缩写为 CLN。
- [Core Lightning #5068][Core Lightning #5068] 支持为支付附带 ` option_payment_metadata ` 发票数据，为[无状态发票][stateless invoices]增加支付端的支持。该 PR 尚未加入对接收端的支持。

[posted]:https://lists.linuxfoundation.org/pipermail/lightning-dev/2022-April/003539.html

[taproot]:https://bitcoinops.org/en/topics/taproot/

[intended]:https://lightning.engineering/posts/2022-4-5-taro-launch/

[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-April/020198.html

[proposed BIP]:https://github.com/jonasnick/bips/blob/musig2/bip-musig2.mediawiki

[MuSig2]:https://bitcoinops.org/en/topics/musig/

[multisignature]:https://bitcoinops.org/en/topics/multisignature/

[schnorr signature]:https://bitcoinops.org/en/topics/schnorr-signatures/

[Bitcoin Core PR Review Club]:https://bitcoincore.reviews/

[Prevent block index fingerprinting by sending additional getheaders messages]:https://bitcoincore.reviews/24571

[LDK 0.0.106]:https://github.com/lightningdevkit/rust-lightning/releases/tag/v0.0.106

[BOLTs #910]:https://github.com/lightning/bolts/issues/910

[BTCPay Server 1.4.9]:https://github.com/btcpayserver/btcpayserver/releases/tag/v1.4.9

[Bitcoin Core 23.0 RC4]:https://bitcoincore.org/bin/bitcoin-core-23.0/

[draft release notes]:https://github.com/bitcoin-core/bitcoin-devwiki/wiki/23.0-Release-Notes-draft

[test]:https://github.com/bitcoin-core/bitcoin-devwiki/wiki/23.0-Release-Candidate-Testing-Guide

[LND 0.14.3-beta.rc1]:https://github.com/lightningnetwork/lnd/releases/tag/v0.14.3-beta.rc1

[Core Lightning 0.11.0rc1]:https://github.com/ElementsProject/lightning/releases/tag/v0.11.0rc1

[Bitcoin Core]:https://github.com/bitcoin/bitcoin

[Core Lightning]:https://github.com/ElementsProject/lightning

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

[Bitcoin Core #24152]:https://github.com/bitcoin/bitcoin/issues/24152

[CPFP]:https://bitcoinops.org/en/topics/cpfp/

[package feerate]:https://gist.github.com/glozow/dc4e9d5c5b14ade7cdfac40f43adb18a#fee-related-checks-use-package-feerate

[newsletter #186]:https://bitcoinops.org/en/newsletters/2021/09/22/#package-mempool-acceptance-and-package-rbf

[RBF]:https://bitcoinops.org/en/topics/replace-by-fee/

[validates individual transactions first]:https://gist.github.com/glozow/dc4e9d5c5b14ade7cdfac40f43adb18a#always-try-individual-submission-first

[Bitcoin Core #24098]:https://github.com/bitcoin/bitcoin/issues/24098

[Bitcoin Core #24147]:https://github.com/bitcoin/bitcoin/issues/24147

[miniscript]:https://bitcoinops.org/en/topics/miniscript/

[#24148]:https://github.com/bitcoin/bitcoin/issues/24148

[#24149]:https://github.com/bitcoin/bitcoin/issues/24149

[output script descriptors]:https://bitcoinops.org/en/topics/output-script-descriptors/

[Core Lightning #5165]:https://github.com/ElementsProject/lightning/issues/5165

[Core Lightning]:https://github.com/ElementsProject/lightning

[Core Lightning #5068]:https://github.com/ElementsProject/lightning/issues/5068

[stateless invoices]:https://bitcoinops.org/en/topics/stateless-invoices/

