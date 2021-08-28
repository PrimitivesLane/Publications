```
title: 'Bitcoin Optech Newsletter #161'
permalink: /zh/newsletters/2021/08/11/
name: 2021-08-11-newsletter-zh 
slug: 2021-08-11-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
```

本周的Newsletter继续了之前对JoinMarket中忠诚保证保险的介绍，还包含了我们常规的比特币核心PR审核俱乐部会议的总结，学习taproot的建议，发布和候选发布功能的公布，以及一些对主流基础设施项目更新的介绍。

## 新闻
- **实现忠诚保证保险**：[JoinMarket 0.9.0](https://github.com/JoinMarket-Org/joinmarket-clientserver/releases/tag/v0.9.0)对[coinjoin](https://bitcoinops.org/en/topics/coinjoin/)的实现包含了对[忠诚保证保险](https://gist.github.com/chris-belcher/18ea0e6acdb885a2bfbdee43dcd6b5af/)的[支持](https://github.com/JoinMarket-Org/joinmarket-clientserver/blob/master/docs/release-notes/release-notes-0.9.0.md#fidelity-bond-for-improving-sybil-attack-resistance)。如之前在[Newsletter#57](https://bitcoinops.org/en/newsletters/2019/07/31/#fidelity-bonds-for-improved-sybil-resistance)中的介绍，忠诚保证保险提升了JoinMarket系统抵抗女巫攻击的能力，提升了coinjoin的发起者（takers）选择独特的流动性提供者（makers）的能力。在发布后的几天内，超过50个BTC（目前价值超过200万美元）已经被放在带时间锁定的忠诚保证保险里。

  虽然具体的实现是JoinMarket独有的，但整体设计方案可能对其他基于比特币之的去中心化协议有用。

## 比特币核心PR审核俱乐部
*在这个每月一次的栏目中，我们总结了最近一次[比特币核心PR审核俱乐部](https://bitcoincore.reviews/)会议的内容，摘录了一些重要问题和回答。点击下面的问题，可以看到会议上的回答的摘要。*

[Prefer to use txindex if available for GetTransaction](https://bitcoinops.org/en/newsletters/2021/08/11/)是Jameson Lopp的PR。这个PR通过在可以获取到交易索引（txindex）的时候使用它来提升`GetTransaction`的性能(并且可以扩展到用户的`getrawtransaction` RPC)。这一变化修复了一个意外的性能损失，即在一个启用了txindex的节点上,用包含交易的区块的哈希值调用`getrawtransaction`时，会明显变慢。评审会通过比较使用和不使用txindex检索一个交易时各自的步骤来评估这个性能问题的原因。

<details><summary>GetTransaction从磁盘上获取交易有哪些不同方式？
</summary>

交易可以从mempool中获取（如果未确认），或者通过从磁盘中获取整个块并搜索交易，或者使用txindex从磁盘中直接获取交易。[➚](https://bitcoincore.reviews/22383#l-33)
</details>

<details><summary>为什么你认为使用区块哈希值（启用txindex时）的方式性能会变差？
</summary>

与会者猜测，瓶颈在于区块的反序列化。另一个问题也是使用这个方式独有的--尽管不那么耗时--是对整个交易列表的线性搜索。[➚](https://bitcoincore.reviews/22383#l-42)
</details>

<details><summary>如果我们按区块哈希查找交易会有哪些步骤？有多少数据是被反序列化的？
</summary>

我们首先使用区块索引来找到访问区块所需的文件和字节偏移。然后我们获取并反序列化整个区块，在交易列表中扫描，直到找到匹配的交易。这涉及到对大约1-2MB的数据进行反序列化。[➚](https://bitcoincore.reviews/22383#l-56)
</details>

<details><summary>如果我们使用txindex来查询交易会有哪些步骤？有多少数据是被反序列化的？
</summary>

txindex将交易id映射到文件、块的位置（类似于块索引）以及blk*.dat文件中的交易开始位置的偏移量。我们获取并反序列化块头和交易。区块头是80字节，可以用于向用户返回区块哈希（这是不存储在txindex中的信息）。交易可以是任何大小，但通常是区块大小几千分之一。[➚](https://bitcoincore.reviews/22383#l-88)
</details>

<details><summary>这个PR的第一个版本包括一个行为改变：当提供给GetTransaction的block_index不正确时，用txindex找到并返回tx。你认为这个变化是一个改进么？是否应该包括在这个PR中？
</summary>

与会者同意，这可能是有帮助的，但会产生误导。通知用户错误的区块哈希被输入会更好。他们还指出，性能改进和行为改变最好分成单独的PR。[➚](https://bitcoincore.reviews/22383#l-128)
</details>

## 为 taproot 做准备#8: 多重签名 nonce
*关于开发者和服务提供者如何为即将在区块高度709,632处激活的taproot做准备的每周[系列](https://bitcoinops.org/en/preparing-for-taproot/)文章。*

在[上周的专栏](https://bitcoinops.org/en/preparing-for-taproot/#multisignature-overview)中，我们谈到了[多重签名](https://bitcoinops.org/en/topics/multisignature/)，并举了一个使用[MuSig2](https://bitcoinops.org/en/topics/musig/)的例子。我们的描述似乎在技术上是正确的，但几个为MuSig2作出贡献的密码学家[担心](https://gnusha.org/secp256k1/2021-08-04.log)我们建议的使用方式是危险的。我们[更新](https://github.com/bitcoinops/bitcoinops.github.io/pull/622)了我们的描述以解决他们的担忧，然后开始更深入地研究这个问题。在这篇文章中，我们将讨论我们了解到的可能是安全实现多签名的最大挑战：避免nonce重复使用。

为了验证比特币中的一个签名的有效性，你要用签名、被签名的消息（如交易）、你的公钥和一个公开nonce等信息去填充一个等式。只有当你知道私钥和私密形式的nonce，你才有可能平衡这个等式。因此，任何人看到这样一个平衡的等式都会认为该消息的签名和公钥是有效的。

将签名和消息纳入等式的动机是显而易见的。公钥是你私钥的替身。公开nonce的作用是什么？如果没有他，方程中除了你的私钥以外的其他值都是已知的，这意味着我们可以用基本的代数来求解这个唯一的未知值。但代数不能求解两个未知值，所以私密Nonce是为了保持你的私钥的安全性。而且，正如你的公钥在签名方程中是你的私钥的替身一样，nonce的公开形式也是他私密形式的替身。

在这种情况下，nonce不仅是*使用一次的数字*，而且是**必须**只使用一次的数字。如果你在两个不同的签名中重复使用同一个nonce，两个签名方程可以联立，nonce可以被抵消，就可以解出唯一剩下的未知值——你的私钥。如果你使用[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)常规衍生（非硬化衍生），可能几乎所有的多签名钱包都是这样做的，那么一个私钥的泄露就意味着同一BIP32路径（也可能是其他路径）中的所有其他私钥的泄露。这意味着一个上百个地址控制的多签名的钱包，如果有一个签名者重复户使用了一次nonce，那么所有的地址都会被泄露。

单一签名钱包，或那些使用基于脚本的多重签名钱包，可以使用一个简单的技巧来避免重复使用nonce：使nonce取决于他们正在签署的消息。如果信息有任何变化，nonce就会改变，所以他们永远不会重复使用nonce。

多重签名不能使用这个技巧。因为它要求每个联合签名者不仅提供部分签名，还需要提供部分公开nonce。这些nonce被组合在一起，产生一个聚合的公共nonce，被包含在待签名消息内。

这意味着，即使交易保持不变，多次使用相同的部分nonce也是不安全的。如果在你第二次签名时，你的联合签名者之一改变了他的部分nonce（会改变聚合nonce），你的第二次部分签名就被用于一个不同消息。这就暴露了你的私钥。由于每一方都不可能让他们的私密nonce依赖于所有其他方的部分公共nonce，所以没有简单的方法来避免多重签名中的私密nonce重复使用。

乍一看，这似乎并不是一个大问题。只要让签名者在每次需要签名的时候生成一个新的随机nonce。但是这比听起来要难得多——至少从[2012](https://web.archive.org/web/20160308014317/http://www.nilsschneider.net/2013/01/28/recovering-bitcoin-private-keys.html)年开始，就一直会发现依赖于生成随机nonce的钱包中存在会导致比特币丢失的漏洞。

但是，即使一个钱包确实生成了高质量的随机nonce，它也必须确保每个nonce最多只能使用一次。这是一个巨大的挑战。在上周我们专栏的原始版本中，我们介绍了一个与MuSig2兼容的冷钱包或硬件签名设备，它们会在第一次运行时创建大量的nonce。然后，钱包或设备需要确保每个nonce都不会被用于一个以上的部分签名。虽然这听起来很简单——只要在每次使用nonce时递增一个计数器——但考虑到软件和硬件可能以各种方式出现意想不到的问题，这就变成了一个巨大的挑战，更何况它们还可能受到外部甚至是恶意干预的影响。

对于钱包来说，减少nonce重复使用风险的最简单方法是尽可能缩短nonce的存储时间。我们上周的例子建议将nonce存储数月或数年，这不仅为出错创造了很多机会，而且还需要将nonce记录在一个持久的存储介质上，该介质可能被备份和恢复或以其他方式进入一个意想不到的状态。使用MuSig2的另一种方式是只在需要时创建nonce，例如在收到[PSBT](https://bitcoinops.org/en/topics/psbt/)时。这种情况下，nonce可以在使用时被短暂地存储在易失性存储中，因此在一些意外发生的情况下，如软件崩溃或断电，会自动销毁（使其无法被重复使用）。

尽管如此，研究这个问题的密码学家们还是非常关注在原始MuSig协议(MuSig1)和MuSig2中缺乏防止nonce重复使用的万无一失的方法。MuSig-DN(确定性nonce)确实提供了一个解决方案，但它很复杂，也很慢（一个alpha实现在2.9GHz的英特尔i7上创建一个nonce证明需要将近一秒钟；我们不知道这在一个16MHz的硬件签名设备上需要多长时间，这种设备的处理器通常没有那么好的性能）。

我们对任何实施多重签名的人的建议是，在你投入大量的时间和资源之前，考虑在[#secp256k1](https://web.libera.chat/?channels=#secp256k1) IRC房间或其他比特币密码学家聚集的地方讨论你的方案。

## 发布和候选发布
*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- [C-Lightning 0.10.1](https://github.com/ElementsProject/lightning/releases/tag/v0.10.1)包含一些新功能，一些错误的修复，以及一些开发协议的更新（包括[dual funding](https://bitcoinops.org/en/topics/dual-funding/)和[offers](https://bitcoinops.org/en/topics/offers/)）。

- [Bitcoin Core 22.0rc2](https://bitcoincore.org/bin/bitcoin-core-22.0/)是下一个主要版本的全节点实现及其相关钱包和其他软件候选发布版本。这个新版本的主要变化包括支持[I2P](https://bitcoinops.org/en/topics/anonymity-networks/)连接，取消了对[第二版Tor](https://bitcoinops.org/en/topics/anonymity-networks/)连接的支持，并加强了对硬件钱包的支持。

## 重大代码和文档更新
*本周[Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[Hardware Wallet Interface(HWI)](https://github.com/bitcoin-core/HWI)、[Rust Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay Server](https://bitcoinops.org/en/newsletters/2021/08/11/)、[Bitcoin Improvement Proposals(BIPs)](https://github.com/bitcoin/bips/)和[Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/)中值得注意的变更。*

- [Bitcoin Core #21528](https://github.com/bitcoin/bitcoin/issues/21528)旨在改善全节点监听地址的p2p传播。为了避免诸如[日蚀攻击](https://bitcoinops.org/en/topics/eclipse-attacks/)的网络恶意分区类攻击，保护比特币节点同非固定地址集合的通信尤为重要。具体来说，当比特币核心节点接收到一个包含了不多于10个地址的消息时，节点会将该消息转发至1到2个节点。针对这一用于自我广播地址的技术，如果相应的消息广播发送给了那些不再继续转发消息的节点，则会在网络中阻断消息的进一步传播，形成“黑洞”。尽管在恶意场景下，消息传播的失效仍然无法避免，但是该补丁能够在诚实节点参与的场景下（如，仅用于区块中继转发的网络连接或轻节点）有效提高地址的传播能力。

  本更新根据一个接入连接是否发送过地址相关的消息，如`addr`、`addrv2`或`getaddr`，来识别该连接是否能够有效转发地址。如果网络上有依赖接收地址消息的软件，但从来没有发起过地址相关的消息，那么这种行为变化就可能有问题。因此，作者在合并之前进行传播时要留意这种变化，包括将其发布到[邮件列表](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-April/018784.html)，并研究[其他开源客户端](https://github.com/bitcoin/bitcoin/pull/21528#issuecomment-809906430)以确认兼容性。

- [LND #5484](https://github.com/lightningnetwork/lnd/issues/5484)允许将所有数据存储在一个外部Etcd数据库中。通过使集群领导的改变瞬间完成，改善了高可用性的部署。LND相应的集群文档以前在[Newsletter #157](https://bitcoinops.org/en/newsletters/2021/07/14/#lnd-5447)中介绍过。

- [Rust-Lightning #1004](https://github.com/rust-bitcoin/rust-lightning/issues/1004)为`PaymentForwarded`增加了一个新的事件，允许追踪付款被成功转发的时间。由于成功的转发可以为节点赚取费用，这个功能可以用来记录对用户的收入。

- [BTCPay Server #2730](https://github.com/btcpayserver/btcpayserver/pull/2730)使生成invoice时的金额成为可选项。这简化了运营商委托用户选择金额的情况下的支付流程，例如在给账户充值的时候。
