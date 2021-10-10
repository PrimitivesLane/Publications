```
title: 'Bitcoin Optech Newsletter #168'
permalink: /zh/newsletters/2021/09/29/
name: 2021-09-29-newsletter-zh 
slug: 2021-09-29-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
```

本周的 Newsletter 总结了在 DLC 规范中实施不兼容变更的提案，介绍了允许只使用 BIP32 种子恢复关闭的 LN 通道的选项，并描述了一个生成无状态 LN 账单的想法。同时还包括我们的常规部分，来自 Bitcoin Stack Exchange 的热门问题和答案，为 taproot 激活做准备的想法，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻
- **关于 DLC 规范不兼容变更的讨论**：Nadav Kohen 在 DLC-dev 邮件列表中[发布](https://mailmanlists.org/pipermail/dlc-dev/2021-September/000075.html)了关于更新 [DLC](https://bitcoinops.org/en/topics/discreet-log-contracts/) 规范的几个变更，这些变更可能会破坏与现有应用程序的兼容性。他提出了两个选择：根据需要更新规范，让应用程序公布他们的实现版本，或者将几个不兼容的变更分批进行，以尽量减少影响的次数。要求从事 DLC 软件的开发人员提供反馈。

- **只使用种子恢复 LN 关闭交易的挑战**：Electrum 钱包的开发者 ghost43 在 Lightning-Dev 邮件列表中[发布](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-September/003229.html)了关于扫描区块链上的钱包通道关闭交易的一些挑战。一个具体的问题是在恢复一个只使用 BIP32 风格的分层确定性密钥生成的钱包时，如何处理新的[锚定输出](https://bitcoinops.org/en/topics/anchor-outputs/)协议。ghost43 分析了几种可能的方案，并建议将 Eclair 目前使用的方案作为当前可行的最佳方案。此外，如果实施者愿意稍微修改通道打开协议，也建议进一步改进。

- **无状态的 LN 账单生成**：Joost Jager 在 Lightning-Dev 邮件列表中[发布](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-September/003236.html)了一个可能的拒绝服务攻击，针对为未认证用户生成 LN 账单的应用程序。具体来说，攻击者可以请求数量不受限制的账单，生成服务需要存储这些账单，直到它们过期。Jager 建议，对于不需要太多数据的小型账单，服务可以生成账单并立刻忘记它，并给请求的用户提供账单参数。这些参数将与付款一起提交，使服务能够重建账单，接受付款，并完成订单。

  尽管一些受访者[表示担心](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-September/003252.html)这个想法是不必要的——因为可能以其他方式用请求来淹没应用程序，并且任何解决这些问题的方法也需要解决账单请求淹没的问题——但其他人认为这个想法是有用的。这个想法似乎不需要改变任何协议，只需要改变软件（或插件）来创建和管理账单的生成和重建。

## Bitcoin Stack Exchange 问答选摘
*[Bitcoin Stack Exchange](https://bitcoin.stackexchange.com/) 是 Optech 贡献者寻找问题答案，或者当我们有一些空闲时间时帮助好奇或困惑的用户的首选地方之一。在这个月度专题中，我们将重点介绍一些自上次更新以来被投票最多的问题和答案。*

- [为什么比特币的地址安全是 160 位，TXID 却有 256 位？](https://bitcoin.stackexchange.com/questions/109652/why-does-the-txid-have-256-bits-when-bitcoins-address-security-is-160-bit)Pieter Wuille 概述了在考虑比特币的比特大小时的三个安全考虑因素：抗原像、抗碰撞和存在性伪造。他进一步解释了比特币的 128 位安全级别目标是如何在这些考虑中实现的（或不实现）。

- [为什么不鼓励 OP_RETURN 交易？使用版本和锁定期有什么区别吗？](https://bitcoin.stackexchange.com/questions/108389/why-are-op-return-transactions-discouraged-does-using-version-or-locktime-make)除了在另一个问答中从技术方面概述 `OP_RETURN` 如何[燃烧硬币](https://bitcoin.stackexchange.com/questions/109747/how-does-op-return-burn-coins/109748#109748)，Pieter Wuille 还提供了他对使用 `OP_RETURN` 进行数据存储的看法。

- [在交易中使用非标准的版本号](https://bitcoin.stackexchange.com/questions/108248/version-in-transaction)。 Andrew Chow 和 G. Maxwell 解释说，因为只有 1 和 2 是交易的标准版本号，即使你可以让矿工接受另一个版本号，但如果未来的共识规则使用了该版本号，可能就会导致这些输出无法花费，或者交易本身无效。

- [是否有按地址类型划分的 UTXO 的历史数据？](https://bitcoin.stackexchange.com/questions/109776/tracking-invoice-address-type-migration)Murch 提供了一些来自 [transactionfee.info](https://transactionfee.info/) 的示例图。

- [OP_CSV 和 OP_CLTV 如何向后兼容？](https://bitcoin.stackexchange.com/questions/109834/how-are-op-csv-and-op-cltv-backwards-compatible)Andrew Chow 介绍了此前 BIP65 和 BIP112 [时间锁](https://bitcoinops.org/en/topics/timelocks/)的[软分叉激活](https://bitcoinops.org/en/topics/soft-fork-activation/)机制。

## 为 taproot 做准备 #15: signmessage protocol still needed
*关于开发者和服务提供者如何为即将在区块高度 709,632 处激活的 taproot 做准备的每周[系列](https://bitcoinops.org/en/preparing-for-taproot/)文章。*

自从四年多前激活 segwit 以来，一直没有被广泛接受的方法来为 [bech32或bech32m](https://bitcoinops.org/en/topics/bech32/) 地址创建签名文本信息。可以说，这意味着我们现在可以认为，广泛的消息签名支持对用户或开发者来说并不十分重要，否则会有更多的工作致力于此。但似乎比特币钱包软件退步了，因为之前每个使用传统地址的人可以很容易地交易签名信息。

我们两年前在 [bech32 花费支持](https://bitcoinops.org/en/bech32-sending-support/#message-signing-support)系列中写到的[通用 signmessage](https://bitcoinops.org/en/topics/generic-signmessage/) 解决方案已经陷入困境，甚至没有被比特币核心采用，尽管它的协议文档 [BIP322](https://github.com/bitcoin/bips/blob/master/bip-0322.mediawiki) 偶尔会更新（见Newsletter [#118](https://bitcoinops.org/en/newsletters/2020/10/07/#alternative-to-bip322-generic-signmessage) 和 [#130](https://bitcoinops.org/en/newsletters/2021/01/06/#proposed-updates-to-generic-signmessage)）。尽管如此，我们还不知道有什么更好的替代方案，所以 BIP322 仍然应该是任何想在 P2TR 钱包中添加 signmessage 支持的开发者的首选。

如果实施，通用的 signmessage 将可以为 P2TR 输出真正使用单签名的消息签名，基于[多重签名](https://bitcoinops.org/en/topics/multisignature/)，或 
[tapscript](https://bitcoinops.org/en/topics/tapscript/)。它还将提供与所有传统地址和 bech32 地址的向后兼容性，以及与目前设想的不久的将来的变化类型的向前兼容性（其中一些我们将在未来的*为 taproot 做准备*专栏中预览）。能够访问完整 UTXO 集的应用程序（例如通过全节点）也可以使用 BIP322 来生成和验证[储备证明](https://github.com/bitcoin/bips/blob/master/bip-0322.mediawiki#full-proof-of-funds)，提供证据证明签名者在某个时间控制了一定数量的比特币。

实现对创建通用签名信息的支持应该是非常容易的。BIP322 使用一种叫做*虚拟交易*的技术。第一笔虚拟交易是通过试图从不存在的前一笔交易（txid 全为零的交易）中支出而故意制造出来的无效交易。第一笔交易支付给用户想要签名的地址（脚本），并包含对所需消息的哈希承诺。第二笔交易花费了第一笔交易的输出——如果该花费的签名和其他数据是有效的交易，那么该消息就被认为是有签名的（尽管第二笔虚拟交易不能被纳入链上，因为它是从一个无效的前序交易中花费的）。

对于许多钱包来说，验证通用的签名信息是比较困难的。为了能够完全验证*任何* BIP322 消息，需要实现比特币的所有共识规则。大多数钱包没有做到这一点，所以 BIP322 允许它们在不能完全验证一个脚本时返回一个 "不确定 "的状态。在实践中，尤其是在 taproot 支持密钥路径花费的情况下，这种情况可能很少。任何一个钱包，只要实现了几个最流行的脚本类型，就能够验证 99% 以上的 UTXO 的签名信息。

通用的 signmessage 支持将是对比特币的一个有益补充。虽然我们不能忽视过去几年对它的关注不足，但我们鼓励阅读此文的钱包开发者考虑在你的程序中加入对它的实验性支持。这是给用户提供一个他们已经错过了几年的功能的简单办法。如果你是从事 BIP322 或相关储备证明实现的开发者，或者是发现该功能有用的服务提供商，请随时联系 info@bitcoinops.org 来获取支持。

## 发布和候选发布
*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- [Bitcoin Core 0.21.2](https://bitcoincore.org/bin/bitcoin-core-0.21.2/) 是 Bitcoin Core的维护版本的一个候选发布版本，包含了若干代码中的缺陷修复以及小规模的实现优化。

## 重大代码和文档更新
*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[Hardware Wallet Interface(HWI)](https://github.com/bitcoin-core/HWI)、[Rust Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay Server](https://bitcoinops.org/en/newsletters/2021/08/11/)、[Bitcoin Improvement Proposals(BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 中值得注意的变更。*

- [Bitcoin Core #12677](https://github.com/bitcoin/bitcoin/issues/12677) 在钱包的 `listunspent RPC` 方法返回的交易输出中增加了 `ancestorcount`、`ancestorsize` 和 `ancestorfees` 字段。如果创建交易输出的交易是未确认的，这些字段将显示该交易及其在内存池中所有未确认的祖先的总数、大小和费用。矿工根据其祖先的费率来选择交易纳入区块，所以知道祖先的大小和费用对于用户估计交易的确认时间或试图使用 [CPFP](https://bitcoinops.org/en/topics/cpfp/) 或 [RBF](https://bitcoinops.org/en/topics/replace-by-fee/) 来更新交易费用是非常有用的。

- [Eclair #1942](https://github.com/ACINQ/eclair/issues/1942) 允许配置路径搜寻算法，从而使路由的评估可以一部分基于其容量。这种配置可以作为一个[实验性的参数集](https://bitcoinops.org/en/newsletters/2021/09/15/#eclair-1930)来应用，以潜在提高路由的成功率。*[编辑：这项内容在发表后得到了纠正；感谢 Thomas Huet 报告的不准确之处]*。

- [LND #5101](https://github.com/lightningnetwork/lnd/issues/5101) 增加了一个*中间件拦截器*，该拦截器接收每个发往服务器的 RPC 请求，并可以进行修改。这允许在 LND 之外实现跟踪或影响大量用户和自动化操作的逻辑。为了安全起见，只有明确使用选择加入拦截的认证令牌（macaroon）的 RPC 可以被拦截。