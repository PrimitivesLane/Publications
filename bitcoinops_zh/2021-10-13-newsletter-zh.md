---
title: 'Bitcoin Optech Newsletter #170'
permalink: /zh/newsletters/2021/10/13/
name: 2021-10-13-newsletter-zh 
slug: 2021-10-13-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 描述了最近在几个LN实现中修复的一个漏洞，并总结了一个让闪电网络可以利用 taproot 特性的升级提议。还包括我们的常规部分，包括最近的 Bitcoin Core 新代码审查俱乐部（Bitcoin Core PR Review Club）会议的总结，为 taproot 作准备的信息，新软件发布和候选发布的列表，以及对流行的比特币基础设施软件的显著变化的描述。

## 新闻
- **闪电网络（LN）手续费支付漏洞**：上周，Antoine Riard 在 Lightning-Dev（闪电网络开发）邮件列表中[宣布](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-October/003257.html)发现了多个程序的常见漏洞和缺陷（CVE）。比特币用户一直被劝阻不要创造[不经济输出](https://bitcoinops.org/en/topics/uneconomical-outputs/)，也即若要使用，手续费就会花掉一大部分面额的输出。然而，LN 允许用户发送在链上不经济的小数额。在这些情况下，支付节点或路由节点为小额的承诺事务支付的矿工费是超过承诺事务的价值的，如果这些承诺事务上链，等于是在给矿工捐钱（尽管大部分时候都不会出现这种情形）。

    正如 Riard 所报告的那样，LN 的实现允许用户设置可容忍的不经济上限为通道价值的 20% 乃至更多，因此五次或更少的支付就可以将一个通道的价值全部 “捐” 给矿工。在手续费中利益受损，是 LN 小额支付机制的一个基本风险，但仅仅五次支付就有可能损失一个渠道的所有价值，这显然过分了。

  Riard 的邮件中描述了几种缓解措施，包括让LN节点直接拒绝路由手续费可能超过一定数量的支付。实施这一点可能会降低节点同时路由多个链上不经济的小额支付的能力，尽管目前还不清楚它是否会在实践中造成任何问题。Optech 跟踪的所有受影响的 LN 实现都已经发布或即将发布一个版本，实现至少一个提议的缓解措施。

- **LN 的多个改进提议**：Anthony Towns 在 Lightning-Dev 邮件列表中[发布了](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-October/003278.html)一个详细的建议（附有一些示例代码），描述了如何减少支付延迟，提高备份弹性，并允许在签名密钥离线时接收 LN 付款。该提案提供了一些与 [eltoo](https://bitcoinops.org/en/topics/eltoo/) 相同的好处，但不需要 [SIGHASH_ANYPREVOUT](https://bitcoinops.org/en/topics/sighash_anyprevout/) 软分叉，也不需要（即将在区块高度 709,632 处激活的）taproot 软分叉之外的任何其他共识变化。因此，它可以在 LN 开发者实施和测试之后立即部署。看一下主要的功能。

  - **减少支付延迟**：通道参与者可以提前交换一些处理支付必需、但与支付交易的具体内容无关的信息，这样一来，节点只需向通道对手方发送支付交易或支付交易的签名，即可启动或路由支付。取消在关键路径上往返通信的需要，使得支付可以接近 LN 节点之间基础链接的速度在网络上传播。在发生故障的情况下，退还付款的速度会比较慢，但至少与这个变更实施之前持平。这个功能是开发者 ZmnSCPxj 之前提出的想法的延伸（见[本报第 152期](https://bitcoinops.org/en/newsletters/2021/06/09/#receiving-ln-payments-with-a-mostly-offline-private-key)），他也基于自己与 Towns 的私人讨论，在本周[发出了](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-October/003265.html)一个相关的帖子。

  - **改进备份弹性**：目前 LN 要求通道双方和他们所用的[瞭望塔服务](https://bitcoinops.org/en/topics/watchtowers/)存储关于通道的每一个先前状态的信息，以防被盗。Towns 建议为大多数关于通道状态的信息使用确定性的推导方法，并在每个交易中编码一个状态号码，以允许恢复必要的信息（在某些情况下需要一些暴力推断）。这使得节点可以在创建通道时备份所有与密钥有关的信息。任何其他所需的信息应该可以从区块链（在企图盗窃的情况下）或通道对手方处（在节点丢失自己的数据的情况下）获得。

  - **用离线密钥接收付款**：在 LN 中发送或路由付款基本需要一个在线（热）密钥，但目前的协议也需要一个在线密钥来接收付款。基于 Lloyd Fournier 对 ZmnSCPxj 之前提到的想法的改编（亦见于[本报第 152期](https://bitcoinops.org/en/newsletters/2021/06/09/#receiving-ln-payments-with-a-mostly-offline-private-key)），接收节点可能只需在开设、关闭和重平衡通道时让密钥在线。这可以提高商家节点的安全性。

该提案还提议将 LN 升级到使用 taproot 和 [PTLC](https://bitcoinops.org/en/topics/ptlc/)，以提供[众所周知的](https://bitcoinops.org/en/preparing-for-taproot/#ln-with-taproot)隐私和效率优势。这个想法在邮件列表中得到了很好的讨论，在撰写本文之时，讨论还在进行。

## Bitcoin Core 新代码审查俱乐部
*在这个每月一次的栏目中，我们会总结最近一次 [Bitcoin Core 新代码审查俱乐部](https://bitcoincore.reviews/)会议的内容，摘录一些重要问题和回答。点击下面的问题，可以看到会议上的回答的摘要。*

“[提取 RBF 逻辑放入 policy/rbf](https://bitcoincore.reviews/22675)”是 Gloria Zhao 提交的一个代码更新，将 Bitcoin Core 软件的 [Replace By Fee](https://bitcoinops.org/en/topics/replace-by-fee/) 逻辑提取到独立的应用函数中。

<details><summary> 交易池（mempool）的高级设计目标是什么？
</summary>
交易池的目标是为挖矿节点（甚至是不挖矿的节点）保留最符合激励机制的交易候选者。然而，DoS保护（例如不允许人们利用 P2P 网络广播永远不会得到区块确认的交易）和矿工激励（最大化内存池中价值最高的 1 个区块的联合费用）之间存在着根本的冲突。此处的设计目标是确定冲突发生的位置，并试图将其最小化 <a href="https://bitcoincore.reviews/22675#l-86">➚</a>
</details>

<details><summary>将 RBF 逻辑提取为独立的辅助函数有什么好处？
</summary>
将逻辑分离到较小的函数中，可以更好地进行单元测试，并且在实现包的交易池受理和<a href="https://bitcoinops.org/en/topics/package-relay/">转发</a>时重用这些实现。<a href="https://bitcoincore.reviews/22675#l-24">➚</a>
</details>
<details><summary>
在 <a href="https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki">BIP125</a> 规则 #2 中，为什么要确保替代事务不会引入任何未经区块确认的输入？
</summary>
如果允许替代事务增加新的未确认输入，那么即使手续费率增加，它也不会改变需要先上链的祖先事务的手续费优先级，祖先事务的手续费率评级仍可能会下降。矿工们是根据手续费率来选择纳入区块的交易，所以如果我们允许增加新的输入，那么替代交易的吸引力可能会比它所替代的交易还要小。<a  href="https://bitcoincore.reviews/22675#l-52">➚</a>
</details>

<details><summary>
在 BIP125 规则 #4 中，交易 “为自己的带宽付费” 是什么意思？为什么不是只要手续费率更高就可以作为替代事务？
</summary>
“为自己的带宽付费” 是指替代事务所支付的手续费必须比原事务高出一定的额度。如果没有这条规则，恶意人士就可以通过每次只提高手续费 1 聪（来攻击网络），这样大家的收获与交易池和网络带宽的资源占用是不成比例的。<a id="" href="https://bitcoincore.reviews/22675#l-117">➚</a>
</details>

<details><summary>
Replace-by-fee 逻辑关注的是交易池对策。为什么<a href="https://github.com/bitcoin/bitcoin/blob/0ed5ad102/src/validation.cpp#L774">这个逻辑</a>会返回替换交易失败的原因是共识规则，而不是交易池对策？
</summary>
这个逻辑捕捉了这样的情况：替代事务正在花费它所替换的事务。由于原始交易和替换交易都不可能被确认，这个替换交易永远不可能出现在区块链上，所以是共识错误。<a id="" href="https://bitcoincore.reviews/22675#l-40">➚</a>
</details>

## 为 taproot 作准备 #17：总有与人合作的可能吗？
*关于开发者和服务提供者如何为即将在区块高度 709,632 处激活的 taproot 作准备的每周[系列](https://bitcoinops.org/en/preparing-for-taproot/)文章。*

Gregory Maxwell [最初提出的 taproot 提议](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015614.html)为合约设计提出了一个有趣的原则：

> "几乎总是这样，有趣的脚本都有一个逻辑上的顶层分支，不需要其他东西、只需所有各方生成的一个签名就可以满足脚本要求。其他分支只有在某些参与者不合作的情况下才会被使用。更强烈的说法是，我相信任何预先设定了有限数量固定参与者的合同都可以、也应该被表示为 N-of-N 多签名以及任意更复杂合约的或（OR）运算。" (大写为原文所加)

从那时起，[专家们](https://gnusha.org/taproot-bip-review/2020-02-09.log) [辩论](https://gnusha.org/taproot-bip-review/2020-02-10.log)了这一原理的普遍性，我们所知道的两个可能的例外情况都集中在时间锁上。

- **使用共识来增强的自我控制**：有些人使用时间锁来防止自己在一段时间内花费自己的比特币。时间锁的要求似乎表明，需要的不仅仅是一个签名；但也有人提出了一些反对意见：
  - 一个真正急于花掉他们被锁住的比特币的人可以贷款，也许用其他资产做担保。这就削弱了这种自我契约的效用。
  
  - 除了共识强制的脚本路径时间锁，用户可以允许自己的私钥和第三方私钥所组成的密钥路径（keypath）来花费资金，其中第三方私钥仅在时间锁过期后才会签名。这样做不仅效率更高，还能允许实现一个更灵活的支付策略，比如让用户可以卖出自己收到的所有分叉币（forkcoin）、在遭遇人生重大变故或者价格暴涨时与第三方合作从而提前动用资金。
  
- **金库**：正如 Antoine Poinsot 几周前在这里的[专栏](https://bitcoinops.org/en/preparing-for-taproot/#vaults-with-taproot)中提到的那样，金库也广泛使用时间锁来帮助保护资金，这似乎 “与 taproot 的洞见相反，即大多数合约都有一个乐观的情形，所有参与者都通过签名参与合作”。 还有人认为，任何时候人们都会想要一个密钥路径来逃避金库的条件，而且因为给一个具备脚本路径（scriptpath）的合约增加一个密钥路径选项并不增加任何成本，启用一个密钥路径必定是严格更优的选择。

反对总是提供密钥路径的论点似乎是，有些情况下，即使所有的签名者可以一致行动，他们也不相信自己。他们反而相信其他人  —— 执行比特币共识规则的自利全节点的操作者 —— 给签名者的消费能力实施他们自己不总是愿意实施的限制。

反过来说，至少从纯理论上讲，你可以在普通签名者和所有这些经济全节点操作者之间建立一个密钥路径，以获得同样的安全性。更实际的是，可能有一些子集或替代性的节点操作者集合，可以被添加到你的密钥路径集合中，如果他们可以的话，他们会执行你想要的策略（如果他们做不到，你也没有损失，因为你仍然可以使用脚本路径花费）。

撇开理论不谈，我们建议多花点时间考虑是否有机会在基于 taproot 的合约中使用密钥路径，即便是看起来不可行的时候。

## 发布和候选发布
*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- [Eclair 0.6.2](https://github.com/ACINQ/eclair/releases/tag/v0.6.2) 是一个新的版本，包括对上述 *新闻* 部分所描述的漏洞的修复，以及其[发布说明](https://github.com/ACINQ/eclair/blob/master/docs/release-notes/eclair-v0.6.2.md)中描述的新功能和其他错误修复。

## 重大代码和文档更新
*本周内，[Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[Hardware Wallet Interface(HWI)](https://github.com/bitcoin-core/HWI)、[Rust Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay Server](https://bitcoinops.org/en/newsletters/2021/08/11/)、[Bitcoin Improvement Proposals(BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 发布了值得注意的变更。*

- [Bitcoin Core #20487](https://github.com/bitcoin/bitcoin/issues/20487) 增加了一个 `-sandbox` 配置选项，可以用来启用一个实验性的系统调用（syscall）沙盒。在沙盒激活时，内核会终止 Bitcoin Core 对系统的调用，除非调用内容在按进程指定的白名单中。该模式目前只在x86_64 系统上可用，主要用于测试特定线程使用了哪些系统调用。

- [Bitcoin Core #17211](https://github.com/bitcoin/bitcoin/issues/17211) 更新了钱包的 `fundrawtransaction`、`walletcreatefundedpsbt` 和 `send` RPC 方法，以允许交易花费钱包当前还不拥有的输出。

  以前，钱包无法估计使用它不拥有的输出所需的费用，因为它不知道花费该输出的的交易输入的体积。这个 PR 更新了这些 RPC 方法，以接受一个 `solving_data` 参数。通过提供公钥、序列化的 scriptPubKey 或交易中花费的输出的[描述符](https://bitcoinops.org/en/topics/output-script-descriptors/)，钱包可以估计花费这些输出所需的输入体积（从而估计出所需的费用）。

- [Bitcoin Core #22340](https://github.com/bitcoin/bitcoin/issues/22340)。在一个区块被挖出后，它会被广播到 p2p 网络，最终将被转发到网络上的所有节点。传统上，有两种方法来中继区块：传统的中继和 [BIP152](https://github.com/bitcoin/bips/blob/master/bip-0152.mediawiki) 式的[致密区块中继](https://bitcoinops.org/en/topics/compact-block-relay/)。

  仅下载区块的节点不参与交易中继，以减少其带宽的使用，因此，这样的节点没有交易池。因此，致密区块对这样的节点没有好处，因为它总是要下载完整的区块。然而，在高带宽和低带宽模式下，`cmpctblock` 消息都要中继，这对只下载区块的节点来说是额外的带宽开销，因为 `cmpctblock` 消息平均要比同样内容的区块头或者 `inv` 公告大几倍。

  如[本报第 165 期](https://bitcoinops.org/en/newsletters/2021/09/08/#bitcoin-core-pr-review-club)所述，该 PR 通过防止纯区块节点启动高带宽区块中继连接并禁用`sendcmpct(1)` 的发送，使得纯区块节点使用传统的中继方式下载新区块。此外，纯区块节点不再使用 `getdata(CMPCT)` 请求压缩区块。

- [Bitcoin Core #23123](https://github.com/bitcoin/bitcoin/issues/23123) 移除 `-rescan` 启动选项。用户可以使用 `rescan` RPC来代替。

- [Eclair #1980](https://github.com/ACINQ/eclair/issues/1980) 在使用[锚点输出](https://bitcoinops.org/en/topics/anchor-outputs/)（anchor output）时，将接受用任何高于本地全节点动态最低中继费率的承诺交易。

- [LND #5363](https://github.com/lightningnetwork/lnd/issues/5363) 允许跳过 LND 内的部分签名比特币事务（[PSBT](https://bitcoinops.org/en/topics/psbt/)）终局化步骤，允许使用其他软件敲定 PSBT 并广播。如果交易的 txid 被意外改变，这可能导致资金损失，但它确实允许替代工作流程。

- [LND #5642](https://github.com/lightningnetwork/lnd/issues/5642) 现在将在内存内保留一个通道图的缓存，以加快寻路操作的速度。以前，寻路需要昂贵的数据库查询，根据 PR 作者的测量，其速度要慢 10 倍以上。

  在低内存系统上运行 LND 的用户可以通过使用 `routing.strictgraphpruning=true` 标签来减少这个新缓存的内存占用，它会更积极地删除僵尸通道。

- [LND #5770](https://github.com/lightningnetwork/lnd/issues/5770) 为 LND 的子系统提供了更多关于不经济输出的信息，以便对上述新闻部分中描述的 LN CVE 提供缓解措施。
