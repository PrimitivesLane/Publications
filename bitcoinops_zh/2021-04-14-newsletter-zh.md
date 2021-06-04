```
title: 'Bitcoin Optech Newsletter #144'
permalink: /zh/newsletters/2021/04/14/
name: 2021-04-14-newsletter-zh 
slug: 2021-04-14-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
```

本周简报总结了关于激活Taproot的代码方面的最新进展 ，还包括我们的常规板块，即最近Bitcoin Core PR Review Club（Bitcoin Core 审核俱乐部）会议的介绍，以及比特币基础软件的重要更新。



## 新闻

**关于激活Taproot 的探讨** 自我们上次在 [Newsletter #139](https://bitcoinops.org/en/newsletters/2021/03/10/#taproot-activation-discussion)中更新关于探讨Taproot软分叉的激活方式之后，快速试验（Speedy Trial ）方案已成为那些对激活Taproot感兴趣的人所关注的焦点。从这个方案展开出来两个方向的PRs：[PR#21377](https://github.com/bitcoin/bitcoin/issues/21377) （使用在[BIP9](https://github.com/bitcoin/bips/blob/master/bip-0009.mediawiki) 中的一个变体）和[PR#21392](https://github.com/bitcoin/bitcoin/issues/21392) （使用已成为[BIP8](https://github.com/bitcoin/bips/blob/master/bip-0008.mediawiki) 一部分的一个变体）。这些PR之间的主要技术差异是在于规定其起点和终点的方式。 PR＃21377使用的是时间戳中位数([MTP](https://bitcoin.stackexchange.com/a/67622/21052))； PR＃21392使用的是当前区块的高度。

比特币的主网（mainnet）及其各种测试网（例如testnet，默认的 [signet](https://bitcoinops.org/en/topics/signet/)网络和各种独立的signet网络）的MTP（时间戳中位数）基本一致。这使得多个网络即使有截然不同的区块高度，它们也可以共享一组激活参数，从而最大程度地减少了保持网络用户与主网的共识更改同步时产生的工作量。

不幸的是，MTP（时间戳中位数）既可以轻易地由一小部分矿工以小规模的方式进行操作，也可以轻易地以大规模的方式被主要算力篡改。MTP甚至可能在区块链重组期间的突发情况下回退到更早的时间。 相比之下，区块高度只会在非正常重组时降低<sup>[1](https://bitcoinops.org/en/newsletters/2021/04/14/#fn:height-decreasing)</sup>。这让审计人员基本可以得出区块高度只会增加的简单假设，这使得分析基于区块高度的激活机制比MTP机制更容易。

在两项提案之间以及其他顾虑间的这些权衡取舍造成了一种僵局，一些开发人员认为这种僵局阻碍了两项PR获得额外的审计，并且最终其中一方将合并到Bitcoin Core中。这一僵局最终得到了解决，给了激活讨论的一些参与者满意的结果，因为两个PRs的作者同意达成妥协了:

1. 用MTP(时间戳中位数）表示在节点开始对软分叉的块信令进行计数的时间，并且计数会从开始时间之后的下一个2,016块重定向目标周期的开始处进行。这与[BIP9](https://github.com/bitcoin/bips/blob/master/bip-0009.mediawiki) （versionbits模块）和 [BIP148](https://github.com/bitcoin/bips/blob/master/bip-0148.mediawiki) （UASF 用户激活软分叉）开始为其帮助激活过的软分叉做区块计数的方式相同。

2. 也用MTP(时间戳中位数）表示在节点终止对尚未被锁定的软分叉的块信令进行计数的时间。但与BIP9不同的是，只有在执行计数的重定向目标周期结束时才会检查 MTP终止时间。这取消了从启动直接变成失败的激活尝试功能，简化了分析过程，并确保至少有一个完整的2,016个区块周期，使得矿工可以在这期间发出激活信号。

3. 用块高表示最小激活参数。这进一步简化了分析过程，同时还与允许多个测试网络共享激活参数的目标保持一致。 尽管这些网络上的块高可能不同，但是它们都可以使用`0`的最小激活高度在MTP定义的窗口中激活。

虽然一些参与讨论的人对折中方案表示不满，但是到目前为止，已经收到了来自十多位Bitcoin Core积极贡献者和其他两个全节点应用 (btcd and libbitcoin）维护人员对其 [实现](https://github.com/bitcoin/bitcoin/issues/21377)的评论或支持。我们希望激活Taproot的势头继续保持，也会在后续的新闻简报中同步更多进展。



## Bitcoin Core审查俱乐部

本月的这一部分，我们会总结最近一次的[Bitcoin Core审查俱乐部](https://bitcoincore.reviews/)的会议内容，着重介绍一些重要的问题和答案。点击以下问题查看其答案摘要。

[Introduce deploymentstatus](https://bitcoincore.reviews/19438)（介绍部署状态）是Anthony Towns的PR([#19438](https://github.com/bitcoin/bitcoin/issues/19438))，他提出的3个辅助函数，以便 [更容易地隐藏(bury)未来的部署，而无需更改检查软分叉激活状态的所有代码路径](https://github.com/bitcoin/bitcoin/pull/11398#issuecomment-335599326) ：`DeploymentEnabled`测试部署是否可以活跃，`DeploymentActiveAt`检查是否应在给定的区块中强制执行部署，`DeploymentActiveAfter`了解是否应在后面的区块中强制执行部署，这3个函数对嵌入式部署和version bits部署都有效。

审核俱乐部的讨论集中在理解这些变更和其潜在的好处。

<details >
<summary>与<a href='https://github.com/bitcoin/bips/blob/master/bip-0090.mediawiki'> BIP9</a>版本位部署相比，<a href='https://github.com/bitcoin/bips/blob/master/bip-0090.mediawiki'>BIP90</a>埋藏式部署(Buried Deployments)有哪些优势？</summary>
埋藏式部署(Buried Deployments)通过用简单的块高检查代替控制执行的测试来简化部署的逻辑，从而减小与部署这些共识更改相关的技术债。 
</details>

<details >
<summary><a href='https://github.com/bitcoin/bitcoin/issues/19438'>PR</a>中列举了多少埋藏式部署(Buried Deployments)？</summary>
  5个，分别是coinbase里的块高，CLTV (<code>CHECKLOCKTIMEVERIFY</code>)，严格的DER签名，CSV (<code>OP_CHECKSEQUENCEVERIFY</code>)，以及segwit.PR在<a href='https://github.com/bitcoin/bitcoin/blob/e72e062e/src/consensus/params.h#L14-L22'>src/consensus/params.h#L14-22</a>中提出的这些都被列在<code>BuriedDeployment</code>枚举器中。可以说中本聪时代的软分叉被嵌入了。
</details>

<details>
  <summary>目前所定义的版本位(version bits)部署有多少？
  </summary>
  2个:testdummy 和schnorr/taproot (BIPs 340-342), 列举在<a href='https://github.com/bitcoin/bitcoin/blob/e72e062e/src/consensus/params.h#L25-L31'>src/consensus/params.h#L25-31</a>中的代码库。
</details>

<details>
  <summary>如果Taproot软分叉被激活，而且我们之后想要埋藏(bury)该激活方式，若此PR合并，我们需要对Bitcoin Core做哪些修改？
 </summary>
  与当前代码相比，主要更改将大大简化：将<code>DEPLOYMENT_TAPROOT</code>行从<code>DeploymentPos</code>枚举器移动到<code>BuriedDeployment</code>。 最重要的是，<a href='https://bitcoincore.reviews/19438#l-230'>无需更改验证逻辑</a>。
</details>



## 重要代码和文档更新

 本周在[Bitcoin Core](https://github.com/bitcoin/bitcoin), [C-Lightning](https://github.com/ElementsProject/lightning), [Eclair](https://github.com/ACINQ/eclair), [LND](https://github.com/lightningnetwork/lnd/), [Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning), [libsecp256k1](https://github.com/bitcoin-core/secp256k1), [Hardware Wallet Interface (HWI)](https://github.com/bitcoin-core/HWI), [Rust Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin), [BTCPay Server](https://github.com/btcpayserver/btcpayserver/), [Bitcoin Improvement Proposals (BIPs)和](https://github.com/bitcoin/bips/) [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/)中的重要更新。

-  [Bitcoin Core #21594](https://github.com/bitcoin/bitcoin/issues/21594)在`getnodeaddresses`PRC中增加一个`network`字段，以帮助识别各种网络上的节点（即IPv4, IPv6, I2P, onion）。作者还提出这为将来`getnodeaddresses`的一个补丁奠定了基础，该补丁接受特定网络的参数，并仅返回该网络中的地址。

- [Bitcoin Core #21166](https://github.com/bitcoin/bitcoin/issues/21166) 改进了`signrawtransactionwithwallet`PRC，使其能够在具有其他不属于钱包的已签名输入的交易中对输入进行签名。[此前](https://github.com/bitcoin/bitcoin/issues/21151)，如果一个交易通过了RPC，且该交易的已签名输入不属于钱包，那么这些输入的见证(witnesses)会在返回的交易中被破坏。在各种情况下，用其他已签名的输入在交易中进行输入签名可能很有用，[包括增加输入/输出以提高交易费用。](https://github.com/bitcoin/bitcoin/issues/21151)
- [LND #5108](https://github.com/lightningnetwork/lnd/issues/5108) 增加了对使用低级`sendtoroute`RPC进行自发[原子化多路径支付](https://bitcoinops.org/en/topics/multipath-payments/)(AMP，也称*Original AMPs*）的支持。因为付款方会选择所有原像，所以原子化多路径支付本质上是非交互的（或自发的）。付款方原像选择也是keysend样式[自发支付](https://bitcoinops.org/en/topics/spontaneous-payments/)的一部分，该方式已被用于单路径自发支付。预计后续的PR将使自发性多路径支付可以用于更高级别的`sendpayment` PRC。
- [LND #5047](https://github.com/lightningnetwork/lnd/issues/5047) 确保钱包导入[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)扩展公共密钥（xpubs）并将其用于接收对LND链上钱包的付款。 结合LND最近更新的对[PSBT](https://bitcoinops.org/en/topics/psbt/)（部分签名的比特币交易）的支持（请参阅 [Newsletter #118](https://bitcoinops.org/en/newsletters/2020/10/07/#lnd-4389))，这使LND可以为它的非通道资金运作一个监视钱包。 例如，Alice可以从冷钱包中导入xpub，使用LND给她的地址将资金存入该钱包，请求LND打开一个通道，用她的零钱包签署PSBT打开该通道，然后在通道关闭时让LND自动退还资金到其冷钱包里。最后一部分是将已关闭通道的资金存回冷钱包中，这可能需要采取额外的步骤，尤其是在非合作通道关闭的情况下，但是这种变化使LND大部分都可以与兼容PSBT的冷钱包和硬钱包完全互操作。



## 附注：

1. 如果区块链上的每个区块都具有相同的单独工作量证明（PoW），那么累积的PoW最大的有效链也将是最长的链，即最新区块的高度最大的链。但是，比特币协议每产生2016个区块就会调整新区块需要包含的PoW的难度值，从而增加或减少需要证明的工作，以使区块间隔的平均时间保持在10分钟左右。 这意味着，具有较少区块的的链可能比具有较多区块的链具有更大的工作量证明。
- 比特币用户使用PoW最多（拥有最大工作量证明）的链（而不是区块最多）来确定他们是否收到了钱。 当用户看到一条末端的某些区块被不同的区块替换的变种的链时，如果该重组链包含的工作量证明比其当前链更多，则他们将使用该重组链。 因为重组链可能包含更少的块，但是有更多的累积工作量证明，所以链的高度可能降低。

- 虽然这是一个理论上的顾虑，但通常不是实际应用中的问题。 只有在重组部分越过一组2016个块和另一组2016个块之间的重定目标边界中的至少一个时，才有可能降低高度。 涉及大量区块或最近所需的PoW量发生重大变化时链也需要被重组（即预示着算力最近大幅上升或下降，或矿工进行了可观察到的操纵）。 在[BIP8](https://github.com/bitcoin/bips/blob/master/bip-0008.mediawiki)的语境下，我们不认为在激活时降低高度的重组对用户的影响会比更常规的重组行为更大。

