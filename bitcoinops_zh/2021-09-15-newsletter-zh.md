```
title: 'Bitcoin Optech Newsletter #166'
permalink: /zh/newsletters/2021/09/15/
name: 2021-09-15-newsletter-zh 
slug: 2021-09-15-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
```

本周的 Newsletter 描述了一个关于契约操作码的新提案，并总结了关于在 signet 上实施定期重组的反馈请求。另外还包括我们的常规部分，包括如何为 taproot 做准备，新版本和候选版本，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻
- **契约操作码提案**：Anthony Towns 在 Bitcoin-Dev 邮件列表中发布了一个关于契约操作码想法的[概述](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-September/019419.html)，以及一个介绍了这个操作码（和一些其他 tapscript 的变更）如何工作的[更进一步的技术信息](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-September/019420.html)。

  简而言之，一个新的 `OP_TAPLEAF_UPDATE_VERIFY` 操作码（TLUV）会需要 taproot 花费输入的信息，进行 tapscript 中描述的修改，并要求其结果等同于输出中与输入相同位置的 scriptPubKey。从而使得 TLUV 能约束比特币可以花在哪里（[契约](https://bitcoinops.org/en/topics/covenants/)的定义），这和其他提案的操作码类似，如 [OP_CHECKSIGFROMSTACK](https://bitcoinops.org/en/topics/op_checksigfromstack/)（CSFS）和 [OP_CHECKTEMPLATEVERIFY](https://bitcoinops.org/en/topics/op_checktemplateverify/)（CTV）。它与以前的提案不同的地方在于它允许 tapscript 的创建者做如下操作：

  - 内部密钥调整：每个 taproot 地址都是对某个内部密钥的承诺，可以只用签名来进行花费。为了使用一个 tapscript（包括 TLUV），需要揭示当前的内部密钥。TLUV 允许指定要添加到该密钥的调整。例如，如果内部密钥是密钥 `A+B+C` 的集合，可以通过指定 `-C` 的调整来删除密钥 `C`，或者通过指定一个 `X` 的调整来添加密钥 `X`。 TLUV 计算出修改后的内部密钥，并确保这就是同位置输出所要支付的。

    Towns 的邮件中提到了这种方式的一个非常强大的用途是能够更有效地创建一个[混合池](https://gist.github.com/harding/a30864d0315a0cebd7de3732f5bd88f0)。这是一个由多个用户共享的单个 UTXO，这些用户各自控制 UTXO 的一部分资金，但不必在链上公开透露这种所有权（也不透露 UTXO 有多少个所有者）。如果所有资金池成员一起签名，他们可以在高度同质的交易中花费他们的资金。如果有任何分歧，池子里的每个成员都可以发交易来退出他在池子里的所有资金（减去链上的交易费）。

    当前，用预签名的交易就可以创建一个混合池，但如果我们希望池子里的每个成员都能在没有其他成员的合作下单独退出，就会需要创建指数级增长的签名。CTV 也有类似的问题，退出一个池子可能需要创建多个交易，影响多个其他用户。TLUV 允许任何一个成员在任何时候退出，而不需要预签任何东西或影响任何其他用户，除了会显示一个成员已经离开了混合池。

  - Merkle 树调整：taproot 地址也可以是对 tapscripts 树的 merkle 根的承诺，它会作为交易的的输入的 tapscript 以 TLUV 操作码运行。TLUV 允许该脚本指定应该如何修改 merkle 树的相关部分。例如，当前执行的节点（tapleaf）可以从树中删除（例如有人退出 joinpool）或用支付不同 tapscript 的 tapleaf 替换。TLUV 计算修改后的 merkle 根，并确保这就是同位置的输出所要支付的。

    Towns 的邮件描述了如何使用这一点来实现 Bryan Bishop 的 2019 [金库](https://bitcoinops.org/en/topics/vaults/)设计（见 [Newsletter#59](https://bitcoinops.org/en/newsletters/2019/08/14/#bitcoin-vaults-without-covenants)）。Alice 创建了两个密钥对，一个用于她较不安全的热钱包，一个用于她较安全的冷钱包。冷钱包的密钥做 taproot 的内部密钥，可以允许她在任何时候花费资金。热钱包密钥与 TLUV 一起使用，只允许花费到 merkle 树的修改，并且在第二次从热钱包花费之前需要先等待一段延迟时间。

    这意味着 Alice 可以用她的热秘钥花费她所有的资金，但她必须为此创建一个链上交易，然后等待延迟时间（例如 1 天）结束，然后她才能真正将她的资金花费到一个任意的地址。如果别人使用 Alice 的热秘钥消费，Alice 可以使用她的冷秘钥将资金转移到一个安全的地址。

  - 金额自查：除了 TLUV，还增加了第二个操作码，将输入的比特币金额和相应的输出推送到脚本执行栈。可以使用算术和比较操作码来限制可花费的金额。

    在混合池的情况下，这将被用来确保一个离开的成员只能提取他们自己的资金。在金库中，这可以用来设置定期提款限制，例如每天 1 个 BTC。

  截至本文撰写之时，该提案仍在邮件列表中接收初步反馈。我们将在未来的通讯中总结任何值得注意的评论。

- Signet 重组讨论：开发者 0xB10C 在 Bitcoin-Dev 邮件列表中[发布](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-September/019413.html)了一个关于在 [signet](https://bitcoinops.org/en/topics/signet/) 上创建定期区块链重组（reorgs）的提案。最终被重组的区块将在版本字段的一个位上做标识，使得任何不想跟踪重组的人忽略这些区块。重组将定期发生（可能一天三次），并且将遵循两种不同的模式，来展示主网可能出现的重组的情况。

  0xB10C 希望得到反馈，截止到写这篇文章时已经收到了一些评论。我们鼓励任何有兴趣在 signet 上测试重组（或想避免重组）的人阅读讨论内容并考虑参与。

## 为 taproot 做准备 #13: 备份和安全方案
*关于开发者和服务提供者如何为即将在区块高度 709,632 处激活的 taproot 做准备的每周[系列](https://bitcoinops.org/en/preparing-for-taproot/)文章。*

在[上周的专栏中](https://bitcoinops.org/en/preparing-for-taproot/#vaults-with-taproot)，Antoine Poinsot 介绍了 taproot 如何使[金库](https://bitcoinops.org/en/topics/vaults/)式的硬币备份和安全方案更加私密和低成本。在本周的专栏中，我们将看看其他几种通过转换为 taproot 而得到优化的备份和安全方案。

- **简单 2-3**：正如前一周[提到](https://bitcoinops.org/en/preparing-for-taproot/#threshold-signing)的，很容易通过将[多重签名](https://bitcoinops.org/en/topics/multisignature/)和脚本路径花费结合来创建 2-3 的花费策略，通常在它链上的效率和单签名花费一样高，而且比目前的 P2SH 和 P2WSH 多重签名更隐私。即便在非正常情况下，它仍然是相当高效和私密的。这使得 taproot 非常适合做从单签名钱包升级到多重签名合约的安全升级。

  我们希望未来的[门限签名](https://bitcoinops.org/en/topics/threshold-signature/)技术能够进一步改善 2-3 和其他 k-n 情况。

- **降级的多重签名**：[Optech Taproot 研讨会](https://github.com/bitcoinops/taproot-workshop)中的一个练习可以让你尝试创建一个可以在任何时候用三把密钥，或者三天后用两把原始密钥，或者十天后只用其中一把原始密钥花费的 taproot 脚本。(这个练习也使用了备份密钥，但我们将在下一点单独介绍)。调整时间参数和密钥的设置为你提供了一个灵活而强大的备份结构。

  例如，想象一下，你通常可以使用你的笔记本电脑、移动电话和硬件签名设备的组合进行消费。如果其中一个变得不可用，你可以等待一个月，以便能够使用其余两个设备进行消费。如果两个设备变得不可用，你可以在六个月后只使用一个设备进行消费。

  在三个设备都可以使用的正常情况下，你的链上脚本的效率和隐私是最大化的。在其他情况下，它的效率要低一些，但可能仍然有还不错的隐私性（你的脚本和它的树的深度看起来和许多其他合约中使用的脚本和深度相似）。

- **用于备份和安全的社交恢复**：如果你的一台设备被攻击者偷走，上面的例子中你可以被很好地保护。但如果你的两台设备被偷了，会发生什么？另外，如果你经常使用你的钱包，你真的愿意在丢失一个设备后再等待一个月再重新开始消费吗？

  Taproot 使得为你的备份添加社交元素变得简单、便宜和隐私。除了前面例子中的脚本外，你还可以让你的两个设备加上你的两个朋友或家人的签名，就可以立即消费你的比特币。或者只用你的一个密钥加上你信任的五个人的签名就可以立即消费。(一个类似的非社交版本会仅使用你存储在安全位置的额外设备或助记词)。

- **结合时间和社交门限的继承**：结合上述技术，你可以允许某人或一群人在你突然死亡或失去能力的情况下收回你的密码学资产。例如，如果你六个月没有移动你的比特币，你可以允许你的律师和你五个最信任的亲戚中的任何三个人花费你的币。如果你通常每六个月移动一次你的比特币，这种继承准备在你活着的时候就没有任何额外的链上成本，而且完全对外部观察者隐私。你甚至可以对你的律师和家人保持你的交易隐私，只要你有一个可靠的方法让他们在你死后知道你钱包的扩展公钥（xpub）。

  请注意，让你的继承人可以花费你的比特币并不意味着他们可以合法地花费这些币。我们建议任何计划传承比特币的人阅读 Pamela Morgan 的《*加密资产继承规划*》（[实体书和被 DRM 的电子书](https://www.amazon.com/Cryptoasset-Inheritance-Planning-Simple-Owners/dp/1947910116)或无 [DRM 电子书](https://aantonop.com/product/cryptoasset-inheritance-planning-a-simple-guide-for-owners/)），并利用其信息与当地遗产规划专家讨论细节。

- **泄露检测**：一个在 taproot 发明之前被[提出](https://blockstream.com/2015/08/24/en-treesignatures/#h.2lysjsnoo7jd)的想法是，在所有你关心的设备上放一个控制一定数量比特币的密钥，来作为检测设备何时被攻破的方法。如果比特币的数额足够大，攻击者很可能会为了眼前的利益而把它转移给自己，而不是等着在之后攻击中再利用他们获得的非法权限，以致于给你带来更多的损失。

  这种方法的一个问题是，你想让提供的比特币数额足够大来引诱攻击者，但你不想在你的每一个设备上都放大量的比特币——你更愿意只提供一份大额的比特币。然而，如果你在每台设备上都放上相同的密钥，那么攻击者花费的比特币的交易就不会显示出哪台设备被入侵了。Taproot 让你可以很容易地在每台设备上设置一个不同的密钥和不同的脚本路径。任何一个密钥都可以花费该地址所控制的所有资金，但它也可以帮你唯一地识别哪个设备被攻击。

## 发布和候选发布
*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- [Bitcoin Core 22.0](https://bitcoincore.org/bin/bitcoin-core-22.0/) 是下一个主要版本的全节点实现及其相关钱包和其他软件的候选发布版本。这个新版本的主要变化包括支持 [I2P](https://bitcoinops.org/en/topics/anonymity-networks/) 连接，取消了对[第二版 Tor](https://bitcoinops.org/en/topics/anonymity-networks/) 连接的支持，并加强了对硬件钱包的支持。请注意，像在 [Newsletter #162](https://bitcoinops.org/en/newsletters/2021/08/18/#bitcoin-core-22642) 中提到的，这个版本的发布验证说明已经改变。

- [BTCPay 服务 1.2.3](https://github.com/btcpayserver/btcpayserver/releases/tag/v1.2.3) 版本修复了三个被报告的跨站脚本（XSS）漏洞。

- [Bitcoin Core 0.21.2rc2](https://bitcoincore.org/bin/bitcoin-core-0.21.2/) 是 Bitcoin Core的维护版本的一个候选发布版本，包含了若干代码中的缺陷修复以及小规模的实现优化。

## 重大代码和文档更新
*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[Hardware Wallet Interface(HWI)](https://github.com/bitcoin-core/HWI)、[Rust Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay Server](https://bitcoinops.org/en/newsletters/2021/08/11/)、[Bitcoin Improvement Proposals(BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 中值得注意的变更。*

- [Bitcoin Core #22079](https://github.com/bitcoin/bitcoin/pull/22079) 为 [ZMQ 接口](https://github.com/bitcoin/bitcoin/blob/40a9037a1b5d990637d7f5009fc0c39628ed2c05/doc/zmq.md) 增加了 IPv6 支持。

- [C-Lightning #4599](https://github.com/ElementsProject/lightning/pull/4599) 实现了 [BOLTs #843](https://github.com/lightningnetwork/lightning-rfc/pull/843) 中描述的快速关闭费用协商协议。我们在上周的 Newsletter 中[介绍](https://bitcoinops.org/en/newsletters/2021/09/08/#bolts-847)了该协议，但我们对它所取代的现有协议的描述是[不准确的](https://twitter.com/rusty_twit/status/1435758634995105792)。旧的协议需要在试错的基础上协商在共同关闭交易中使用什么费率，并且不允许设置一个高于当前承诺交易费率的费率。这对[锚定输出](https://bitcoinops.org/en/topics/anchor-outputs/)来说是不合理的，因为低费率的承诺交易被设计为如果被使用就会需要增加交易费用。新协议允许支付更高的费用，并在可能时使用更有效的基于范围的协商。本周合并的 [Eclair #1768](https://github.com/ACINQ/eclair/pull/1768) 也实现了该协议。

- [Eclair #1930](https://github.com/ACINQ/eclair/pull/1930) 允许其路径搜索算法以非默认的实验性参数集运行。这可以在达到一定比例的流量时自动运行，也可以通过API手动进行。每个实验参数集的指标都被单独记录，并可用于优化最佳的路径搜索参数。

- [Eclair #1936](https://github.com/ACINQ/eclair/issues/1936) 允许禁用节点的 Tor 洋葱服务地址的发布，以满足用户希望保持该地址隐私的情况。

- [LND #5356](https://github.com/lightningnetwork/lnd/issues/5356) 增加了一个 `BatchChannelOpen` RPC，可以在同一[批次](https://bitcoinops.org/en/topics/payment-batching/)的链上交易中向不同节点打开多个通道。

- [BTCPay 服务 #2830](https://github.com/btcpayserver/btcpayserver/pull/2830) 增加了对创建一个带 [taproot](https://bitcoinops.org/en/topics/taproot/) 的钱包的支持，该钱包可以同时接收和发送单签名的 P2TR 付款，这在 [signet](https://bitcoinops.org/en/topics/signet/) 上进行过测试。另外一个合并的 PR，[#2837](https://github.com/btcpayserver/btcpayserver/pull/2837) ，在钱包设置中列出了 P2TR 地址支持，但在 709,632 区块之前无法使用。