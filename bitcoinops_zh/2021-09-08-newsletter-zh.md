```
title: 'Bitcoin Optech Newsletter #165'
permalink: /zh/newsletters/2021/09/08/
name: 2021-09-08-newsletter-zh 
slug: 2021-09-08-newsletter-zh 
type: newsletterk
layout: newsletter
lang: zh
```

本周的 Newsletter 介绍了一个与比特币有关的 MIME 类型的建议，并总结了一篇关于设计一种新的去中心化矿池的论文。另外还包括我们的常规部分，包括比特币核心 PR 审核俱乐部会议的总结，如何为 taproot 做准备，新版本和候选版本，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻
- **Bitcoin 相关的 MIME 类型**：Peter Gray 在 Bitcoin-Dev 邮件列表中[发布](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-August/019385.html)了关于向 [IANA](https://en.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority) 注册 [PSBT](https://bitcoinops.org/en/topics/psbt/)、二进制格式的原始交易和 [BIP21](https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki) URI 的 MIME 类型。Andrew Chow [解释](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-August/019386.html)说，他以前曾经试图为 PSBT 注册一个 MIME 类型，但他的申请被拒绝。他认为注册一个 MIME 类型需要编写一个 [IETF](https://en.wikipedia.org/wiki/Internet_Engineering_Task_Force) 规范（一个 RFC），要形成成熟的文档可能需要大量的工作。[Gray](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-September/019390.html) 建议创建一个 BIP 来定义非官方的 MIME 类型，供比特币应用程序使用。

- **Braidpool，一个 P2Pool 的替代品**：[P2Pool](https://bitcointalk.org/index.php?topic=18313.0) 是一个自 2011 年以来就用于去中心化矿池的比特币采矿系统。[发布](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-August/019371.html)在 Bitcoin-Dev 邮件列表中的一篇新[论文](https://github.com/pool2win/braidpool/raw/main/proposal/proposal.pdf)介绍了几个当前系统的缺陷和一个替代的分布是矿池的设计，这个设计有两个明显的改进：通过使用对第三方信任最小化的支付通道，能更有效地利用区块空间进行支付；以及对池子成员之间更高延迟的连接有更大的容忍度。

## 比特币核心 PR 审核俱乐部
*在这个每月一次的栏目中，我们会总结最近一次[比特币核心 PR 审核俱乐部](https://bitcoincore.reviews/)会议的内容，摘录一些重要问题和回答。点击下面的问题，可以看到会议上的回答的摘要。*

[Use legacy relaying to download blocks in blocks-only mode](https://bitcoincore.reviews/22340) 是 Niklas Gögge 的一个 PR。这个 PR 可以使设置 `-blocksonly` 配置选项的节点总是使用传统区块中继下载区块。评审会比较了传统区块下载和 [BIP152](https://github.com/bitcoin/bips/blob/master/bip-0152.mediawiki) 风格的[紧凑块](https://bitcoinops.org/en/topics/compact-block-relay/)下载，并讨论了为什么 blocksonly 的节点不能从后者中获益。

<details><summary>传统的区块中继使用的是怎样的消息顺序？
</summary>

运行 v0.10 和更新版本的节点使用[头优先同步](https://github.com/bitcoin/bitcoin/pull/4468)：节点首先从对等节点收到一个包含区块头的 `headers` 信息。在验证区块头之后，再通过向它传输区块头的对等节点发送 `getdata(MSG_BLOCK, blockhash)` 消息来请求完整的区块。然后，对等节点会回复包含完整区块的 `block` 消息作为回应。[➚](https://bitcoincore.reviews/22340#l-49)
</details>

<details><summary>BIP152 低带宽紧凑块中继使用的是怎样的消息序列？
</summary>

对等节点通过在连接开始时发送 `sendcmpct` 来表明他们想使用紧凑块中继。低带宽紧凑块中继与传统块中继非常相似：在处理区块头后，节点使用 `getdata(MSG_CMPCT_BLOCK, blockhash)` 向其对等节点请求一个紧凑块，并收到一个 `cmpctblock` 作为响应。节点可以使用紧凑块的短码在它的内存池和缓存中寻找该区块的交易。如果还有未知交易，它可以使用 `getblocktxn` 向对等节点请求它们，并收到 `blocktxn` 消息作为响应。[➚](https://bitcoincore.reviews/22340#l-56)
</details>

<details><summary>BIP152 高带宽紧凑块中继使用的是怎样的消息序列？
</summary>

节点可以在第一次建立连接时，通过在连接开始时发送 `sendcmpct` 并将 `hb_mode`设置为 1，向对等节点请求高带宽紧凑块。这意味着对等体可以立即发送 `cmpct` 块，而不需要先发送区块头信息或等待区块的 `getdata` 请求。如果需要，与低带宽紧凑块中继相同，节点可以使用 `getblocktxn` 和 `blocktxn` 请求和下载任何未知的区块交易。[➚](https://bitcoincore.reviews/22340#l-59)
</details>

<details><summary>在区块下载过程中，为什么紧凑区块中继会浪费 blocksonly 的节点的带宽？浪费了多少带宽？
</summary>

紧凑区块中继减少了拥有内存池的节点的带宽使用，因为他们不需要重新下载大部分的区块交易。然而，blocksonly 模式的节点不参与交易中继，通常他们的内存池是空的，这意味着它们无论如何都需要下载所有的交易。shortid、`getblocktxn` 和 `blocktxn` 的开销加起来，每个区块浪费了[大约 38kB 的带宽](https://github.com/bitcoin/bitcoin/pull/22340#issuecomment-872723147)，而 `getblocktxn` 和 `blocktxn` 消息的发送和响应也额外增加了下载区块的时间。[➚](https://bitcoincore.reviews/22340#l-82)
</details>

<details><summary>blocksonly 模式下的节点会保留一个内存池吗？
</summary>

虽然 blocksonly 的节点不参与交易中继，但他们仍然有一个内存池，并且由于某些原因其中可能会包含交易。例如，如果节点处于正常模式，然后在 blocksonly 模式下重新启动，那么内存池会在重新启动时持续存在。此外，任何通过钱包和客户端接口提交的交易都会被验证，并使用内存池进行转发。[➚](https://bitcoincore.reviews/22340#l-97)
</details>

<details><summary>blocksonly 和 block-relay-only 之间的区别是什么？这些变化是否也应该适用于 block-relay-only 的连接？
</summary>

Blocksonly 模式是一个节点设置，而 block-relay-only 是对等节点连接的一个属性。当一个节点以 blocksonly 模式启动时，该节点在与所有对等节点的版本握手中发送 `fRelay=false`，并断开发送任何交易相关消息的对等节点的连接。无论是否在 blocksonly 模式下，节点都可能有 block-relay-only 的连接，在这种连接下它们会忽略传入的交易和地址消息。因此，block-relay-only 连接的存在与节点的内存池内容和从紧凑块消息中重建区块的能力没有关系，所以这些变化不应该应用于 block-relay-only 连接。[➚](https://bitcoincore.reviews/22340#l-111)
</details>


## 为 taproot 做准备 #12: 有 taproot 的金库
*关于开发者和服务提供者如何为即将在区块高度 709,632 处激活的 taproot 做准备的每周[系列](https://bitcoinops.org/en/preparing-for-taproot/)文章。*

作者：[Antoine Poinsot](https://github.com/darosior)，[Revault](https://github.com/revault) 开发者

比特币 [金库](https://bitcoinops.org/en/topics/vaults/) 是一种合约，其需要两个连续的交易，用户才能从他们的钱包里花钱。迄今已经提出了许多这样的[协议](https://bitcoinops.org/en/topics/covenants/)（单方或多方，有或没有契约），所以我们将集中讨论它们的共同点。

与用单一链上交易执行多笔支付的[批量支付](https://bitcoinops.org/en/topics/payment-batching/)相反，金库使用多笔交易来执行一笔支付。第一笔交易，即 *unvault*，支付给以下二者之一:

1. 一组有时间锁的公钥，或
2. 单个没有任何时间锁的公钥。

第一个支付路径是主线情况，和 “热”密钥一起使用。第二种支付路径允许取消交易（有时称为回拨、*恢复或 re-vaulting* 交易）。

因此，比特币金库与 taproot 思路相反，taproot 的思路是大多数合约都有一个路径，这个路径上所有参与者都通过签名进行合作（而争议路径通常包含时间锁）。比特币金库则恰恰相反，*花费*交易必须使用 taproot 脚本发送路径，因为它受到相对时间锁<sup>[1](#footnote1)</sup>的约束，而*取消*交易在理论上可以使用密钥花费路径。

由于多方金库在实践中已经需要大量的交互，它们在理论上可以从 [BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki) 实现的交互式[多签](https://bitcoinops.org/en/topics/multisignature/)和[门限签名](https://bitcoinops.org/en/topics/threshold-signature/)方案中受益，例如 [Musig2](https://bitcoinops.org/en/topics/musig/)。然而，这些方案带来了新的安全挑战。由于金库协议的目的是用于冷存储，因此设计上的选择比较保守，金库可能会是最后一个使用这些新技术的地方。

通过改用 taproot，由于使用 merkle 分支和较短的 BIP340 签名（特别是对于多方的），金库也会从隐私和效率的轻微改善中受益。例如，在一个有 6 个 "冷" 密钥和 3 个"活跃"密钥（阈值为 2）的多方参与的情况下，*unvault* 输出脚本可以表示为一个深度为 2 的带叶子的 Taproot：

- `<X> CSV DROP <active key 1> CHECKSIG <active key 2> CHECKSIGADD 2 EQUAL`
- `<X> CSV DROP <active key 2> CHECKSIG <active key 3> CHECKSIGADD 2 EQUAL`
- `<X> CSV DROP <active key 3> CHECKSIG <active key 1> CHECKSIGADD 2 EQUAL`
- `<cold key 1> CHECKSIG <cold key 2> CHECKSIGADD <cold key 3> CHECKSIGADD <cold key 4> CHECKSIGADD <cold key 5> CHECKSIGADD <cold key 6> CHECKSIGADD 6 EQUAL`

在taproot中，只有用于花费输出的叶子会被揭示，所以交易权重比同等的 P2WSH 脚本要小得多。

```
IF
  6 <cold key 1> <cold key 2> <cold key 3> <cold key 4> <cold key 5> <cold key 6> 6 CHECKMULTISIG
ELSE
  <X> CSV DROP
  2 <active key 1> <active key 2> <active key 3> 3 CHECKMULTISIG
ENDIF
```

虽然撤销分支在成功支出的情况下可以被隐藏（如果使用多签门限，它的存在和参与者的数量也会被混淆），但隐私收益是很小的，因为金库的使用模式在链上是很容易被识别的。

最后，金库协议，像大多数预签署的交易协议一样，将在很大程度上受益于基于 taproot 进一步提出的比特币升级，如 [BIP118](https://github.com/bitcoin/bips/blob/master/bip-0118.mediawiki) 的 [SIGHASH_ANYPREVOUT](https://bitcoinops.org/en/topics/sighash_anyprevout/)。尽管需要进一步的考量和协议调整，`ANYPREVOUT` 和 `ANYPREVOUTANYSCRIPT` 将启用可重新绑定的*取消*签名，这在很大程度上可以减少互动性，并允许 0(1) 签名存储。这对 [Revault 协议](https://github.com/revault/practical-revault)中的*紧急*签名特别有用，因为它将在很大程度上减少 DoS 攻击面。通过在输出中设置 `ANYPREVOUTANYSCRIPT` 签名，你通过限制花费这个币的交易如何创建输出，来有效地创建一个合约。此外，更多的可定制的未来签名哈希值将允许更灵活的限制。

## 发布和候选发布
*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- [Bitcoin Core 22.0rc2](https://bitcoincore.org/bin/bitcoin-core-22.0/) 是下一个主要版本的全节点实现及其相关钱包和其他软件的候选发布版本。这个新版本的主要变化包括支持 [I2P](https://bitcoinops.org/en/topics/anonymity-networks/) 连接，取消了对[第二版 Tor](https://bitcoinops.org/en/topics/anonymity-networks/) 连接的支持，并加强了对硬件钱包的支持。

- [Bitcoin Core 0.21.2rc1](https://bitcoincore.org/bin/bitcoin-core-0.21.2/) 是 Bitcoin Core的维护版本的一个候选发布版本，包含了若干代码中的缺陷修复以及小规模的实现优化。

## 重大代码和文档更新
*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[Hardware Wallet Interface(HWI)](https://github.com/bitcoin-core/HWI)、[Rust Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay Server](https://bitcoinops.org/en/newsletters/2021/08/11/)、[Bitcoin Improvement Proposals(BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 中值得注意的变更。*

- [Bitcoin Core #22009](https://github.com/bitcoin/bitcoin/issues/22009) 更新了钱包，实现保持同时运行 Branch and Bound (BnB) 和 knapsack 算法进行 [UTXO 选择](https://bitcoinops.org/en/topics/coin-selection/)，然后使用新的*浪费得分*启发式方法选择最佳结果，以比较结果的成本收益。之前的方案中，总是会优先选择无变化的 BnB 结果(如果可以找到的话)。

  浪费评分启发式法假设每项输入最终都必须以 10 sat/vB 的费率进行花费，并且可以避免变化的输出。输入的得分是其在当前费率下的成本与基线费率之间的差额，结果是较低费率下的花费输入的浪费得分会降低，较高费率下浪费得分增加。变化输出会因为变化输出的创建成本以及未来花费变化输出的成本两种方式增加浪费分数。

  在一个有多种花费费率的钱包中，这种方法把一些输入消费转到较低的费率上，从而降低整个钱包的运营成本。基线费率的 consolidatory 以及 thrifty 的 UTXO 选择可以用新的选项 `-consolidatefeerate` 来配置。随后的 PR [Bitcoin Core #17526](https://github.com/bitcoin/bitcoin/issues/17526) 提议增加第三种基于单一随机抽取的 UTXO 选择算法。

- [Eclair #1907](https://github.com/ACINQ/eclair/issues/1907) 更新了它如何使用区块链监察人来防止[日蚀攻击](https://bitcoinops.org/en/topics/eclipse-attacks/)（见 [Newsletter #123](https://bitcoinops.org/en/newsletters/2020/11/11/#eclair-1545)）。当 Tor 可用时，Eclair 使用它来联系监察人（尽可能使用本地洋葱端点）。这使上游攻击者更难以有选择地审查监察人服务提供者。

- [Eclair #1910](https://github.com/ACINQ/eclair/issues/1910) 更新了它如何使用比特币核心的 ZMQ 消息传递接口，以提高了解新区块的可靠性。任何其他使用 ZMQ 进行区块发现的人可能希望了解这些变化。

- [BIPs #1143](https://github.com/bitcoin/bips/issues/1143) 介绍了 BIPs 380-386，定义了[输出脚本描述符](https://bitcoinops.org/en/topics/output-script-descriptors/)。输出脚本描述符是一种简单的语言，它包含所有必要的信息，允许钱包或其他程序跟踪向特定脚本或一组相关脚本的付款或支出。BIP380 描述了其理念、一般结构、共享表达式和校验和。其余的 BIP 规定了每个描述符功能，分为：非 segwit（[BIP381](https://github.com/bitcoin/bips/blob/master/bip-0381.mediawiki)）、segwit（[BIP382](https://github.com/bitcoin/bips/blob/master/bip-0382.mediawiki)）、multisig（[BIP383](https://github.com/bitcoin/bips/blob/master/bip-0383.mediawiki)）、组合输出（[BIP384](https://github.com/bitcoin/bips/blob/master/bip-0384.mediawiki)）、原始脚本和地址（[BIP385](https://github.com/bitcoin/bips/blob/master/bip-0385.mediawiki)）以及树（[BIP386](https://github.com/bitcoin/bips/blob/master/bip-0386.mediawiki)）描述符。

- [BOLTs #847](https://github.com/lightningnetwork/lightning-rfc/issues/847) 允许通道的两个对等节点协商在共同关闭交易中应如何支付费用。以前，只有一方发送费用，另一方只能接受或拒绝这个费用。 [编辑：[更准确的描述](https://bitcoinops.org/en/newsletters/2021/09/15/#c-lightning-4599)见下周的 Newsletter。]

- [BOLTs #880](https://github.com/lightningnetwork/lightning-rfc/issues/880) 在 `openchannel` 和 `acceptchannel` 消息中增加了一个 `channel_type` 字段，允许发送节点明确请求与节点公布的特征位所表示的不同的通道特征。这个向后兼容的变化已经在 [Eclair #1867](https://github.com/ACINQ/eclair/issues/1867)、[LND #5669](https://github.com/lightningnetwork/lnd/pull/5669) 中实现，并且正在等待合并为 [C-Lightning #4616](https://github.com/ElementsProject/lightning/pull/4616) 的一部分。

- [BOLTs #824](https://github.com/lightningnetwork/lightning-rfc/issues/824) 在[锚点输出](https://bitcoinops.org/en/topics/anchor-outputs/)通道状态承诺协议上增加了一个微小的变化。在之前的协议中，预签的 [HTLC](https://bitcoinops.org/en/topics/htlc/) 的花费可以包括费用，但这会导致 [Newsletter #115](https://bitcoinops.org/en/newsletters/2020/09/16/#stealing-onchain-fees-from-ln-htlcs) 中提到的费用窃取攻击矢量。在这个替代协议中，所有预签的 HTLC 花费使用零费用，所以没有费用可以被窃取。

## 脚注
<a name="footnote1">1. </a>如果事先知道，你可以用特定的 nSequence 预先签署*花费*交易，但那样的话，你根本不需要带有 "活跃 "密钥的替代花费路径。另外，你在收到币的时候通常还不知道要怎么花。↩