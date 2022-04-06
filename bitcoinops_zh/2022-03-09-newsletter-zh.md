---
title: 'Bitcoin Optech Newsletter #190'
permalink: /zh/newsletters/2022/03/09/
name: 2022-03-09-newsletter-zh
slug: 2022-03-09-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 介绍了关于未来软分叉应该在多大程度上提高比特币脚本和 Tapscript 语言的表达能力的讨论的多个方面，并总结了一个对转发洋葱消息的带宽收费的建议。此外，还包括我们的常规部分，包括 Bitcoin Core PR 审核俱乐部会议的总结，新版本和候选版本的公告，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻

- **限制脚本语言的表现力：** 在 Bitcoin-Dev 邮件列表中，有几个子讨论是关于在脚本中添加 `OP_TXHASH` 或 `OP_TX` 操作码的提案（见 Newsletter [#185][news185 optxhash] 和 [#187][news187 optx]）。Jeremy Rubin [指出][rubin recurse]，这些提案（可能与其他操作码提案相结合，如 [OP_CAT][]）可能允许创建递归[契约][topic covenants]——契约中的条件需要在每笔重新花费这些比特币或与之合并的任何比特币的交易中被永久满足。有人[问][harding recurse]是否会对允许比特币的递归契约有顾虑，一些最明显的顾虑被总结如下：

  - *逐渐失去抗审查能力：* 贡献者 Shinobi [发布][shinobi recurse]了他之前在 [Newsletter #157][news157 csfs] 中提到的担忧，即递归契约让一个强大的第三方控制了该实体目前控制的任何比特币的后续支出。例如，一个政府可以（通过法律）要求其民众只接受政府可以扣押的比特币（由比特币共识规则强制执行）。

    对 Shinobi 帖子的[回复][aj reply]与[一年前][harding altcoin]的论点相[呼应][darosior reply]，即用户转而使用另一种加密货币（altcoin）或类似侧链的结构，也有可能逐渐失去审查阻力，他们对第三方控制有同样的要求。

  - *鼓励不必要的计算：* 开发者 James O'Beirne [表示][obeirne reply]担心，在比特币的脚本或 [Tapscript][topic tapscript] 语言中增加过多的表达能力，会鼓励创建超过证明某个被授权花费的人选择花费该比特币所需的最低数量的计算量的操作的脚本。理想情况下，任何 UTXO（硬币）都可以在当前使用一个紧凑的证明，如一个 64 字节的 [schnorr 签名][topic schnorr signatures]，来证明支出是被授权的。比特币已经允许更复杂的脚本来实现合约的创建，如多签和 LN 等协议，但这种能力可能会被滥用于在脚本中加入一些对执行合同条款没有必要的操作。例如，比特币在过去一直面临着被特别设计的交易带来的拒绝服务攻击的[风险][cve-2013-2292]，这些交易反复执行需要大量 CPU 或内存的操作。O'Beirne 担心增强表达能力既会产生新的 DoS 向量，也会导致程序员创建未经优化的脚本，使用超过必要的节点资源。

  - *引入图灵完备性：* 开发者 ZmnSCPxj [批评][zmn turing]说，增加允许创建*故意*递归契约的操作码也就允许了*意外*创建递归契约。不管是有意还是无意，支付给递归契约的币都不会再与普通比特币完全同质。ZmnSCPxj 在[图灵完备性][turing completeness]和[停机问题][halting problem]的背景下表达了这种担忧。

  - *驱动链的实现：* ZmnSCPxj 扩展了他之前关于图灵完备性的论点，他进一步[认为][zmn drivechains]，脚本语言表达能力的提高也会使[驱动链][topic sidechains]的实现与 [BIP300][] 中规定的原则相似，一些比特币开发者[认为][towns drivechains]这可能导致用户资金的损失或降低抗审查性。如果没有足够的比特币经济体选择运行完整的节点来执行驱动链的规则，驱动链的用户可能会失去资金，但如果很大一部分经济体选择执行驱动链的规则，所有其他想要保持共识的用户将需要验证该驱动链的所有数据，这会使驱动链成为比特币的一部分，但不会有明确的软分叉决议来修改比特币的规则。

    这个特殊的子主题得到了广泛的讨论，并产生了一个[附带的主题][drivechains vs ln]，比较了在大多数的挖矿 hashrate 试图窃取比特币的情况下，驱动链的安全性和 LN 的安全性。

- **为洋葱信息付费：** Olaoluwa Osuntokun 本周在 Lightning-Dev 邮件列表中[发布][osuntokun
  bandwidth]了关于增加节点为其发送[洋葱消息][topic onion messages]的带宽付费的功能。之前提出的洋葱消息协议允许一个节点通过 LN 路径向另一个节点发送消息，无需使用 [HTLC][topic htlc]。与 HTLC 相比，洋葱消息的主要优点是 不需要临时锁定任何比特币，使其更有成本效益和更灵活（例如，洋葱消息可以在对等节点之间发送，即使他们不共享一个通道）。然而，发送洋葱消息没有直接的经济成本，也让一些开发者担心它们会被用来在 LN 上免费中继流量，使 LN 节点的运营成本更高，并为大量的节点禁用洋葱消息中继创造了动力。这可能会带来很大问题，如果洋葱消息被用于节点之间的重要通信，例如[要约][topic offers]的报价。

  Osuntokun 建议，节点可以为他们想要使用的洋葱消息带宽预先付费。例如，如果 Alice 想通过 Bob 和 Carol 将 10 kB 的数据路由给 Zed，她将首先使用 [AMP][topic amp] 向 Bob 和 Carol 支付至少 10 kB 的带宽，费用依据他们各自节点所宣传的消息中继费率。在向 Bob 和 Carol 付款时，Alice 向他们每个人都注册了一个唯一的会话 ID，并在她要求他们为她转发的加密消息中包括该 ID。如果 Alice 支付的金额足以覆盖她的信息所使用的带宽，那么 Bob 和 Carol 就会将信息转发到 Zed。

  Rusty Russell 在[回复][russell reply]中提出了一些批评意见，最主要的是：

  - *HTLC 目前已经免费：* 对于洋葱消息免费中继的担忧，主要的反驳意见一直是已经可以使用 HTLC 在 LN 上免费中继流量<sup>[1](#htlcs-essentially-free)</sup>。不过，目前还不清楚这种情况是否会永久保持——许多修复[信道拥塞攻击][topic channel jamming attacks]的提案建议对失败的 HTLC 收费，而这些 HTLC 目前可以用于免费路由数据。

  - *会话标识符减少了隐私：* 在前面的例子中，Alice 向 Bob 和 Carol 注册的会话标识符使他们能够知道哪些消息来自同一个用户。如果没有会话标识，他们就不会知道不同的消息是否都来自同一个用户，或者不同的用户都使用了同一路径的一部分。Russell 指出，他在研究洋葱信息时曾考虑过盲令牌，但他担心这 "会变得很复杂"。

  Russell 反而建议简单地限制一个节点转发的洋葱消息的数量（不同类别的对等节点有不同的限制）。

## Bitcoin Core PR 审核俱乐部会议

*在这个每月一次的栏目中，我们总结最近一次的 [Bitcoin Core PR 审核俱乐部会议][Bitcoin Core PR Review Club]，集中展示一些重要的问题和答案。点击具体的问题可看到会议答案的总结。*

[开放 p2p 连接到在非默认端口上监听的节点][reviews 23542]是 Vasil Dimov 的一个 PR，目的是在选择出站对等节点时取消对 8333 端口的优先处理。与会者讨论了比特币核心的自动连接行为，没有默认端口的网络的好处，以及避免使用某些端口的理由。

<details><summary>对默认端口 8333 的优先处理的历史原因是什么？
</summary>
这种行为一直存在，但中本聪的动机并不明确。常见的说法认为，它可以防止利用比特币网络通过传播其地址来 DoS 一个服务，但这并不是实际的历史原因。另一个传言的解释是，一个默认的端口可能有助于防止攻击者支配一个节点的 IP 地址表，从而使用一个 IP 地址和许多端口的 P2P 连接（我们现在称之为日蚀攻击）。<a href="https://bitcoincore.reviews/23542#l-43">➚</a>
</details>

<details><summary>用这个PR取消对 8333 端口的优先处理有什么好处？
</summary>
最初过滤和存储潜在对等节点的 IP 地址的方法并不像现在这样复杂；我们现在通过地址的网络组、AS、源对等节点等来限制我们存储的 IP 地址的数量。我们还对我们处理和转发的地址数量进行了速度限制。鉴于地址管理器（'addrman'）和地址中继的这些变化，优先处理对预防日蚀和 DoS 攻击的影响很小。此外，对默认端口的偏好意味着很少有连接是在非默认端口上监听的节点。这也是一个隐私泄露，让本地网络管理员可以轻而易举地检测到比特币网络流量——只需寻找 8333 端口。如果政府想禁止比特币，指示互联网服务提供商记录和/或阻止一个端口的流量，比通过监测所有连接的数据发送和接收来识别比特币流量要容易得多。<a href="https://bitcoincore.reviews/23542#l-72">➚</a>
</details>

<details><summary>在这个变更之前，不鼓励自动连接到监听非默认端口的对等节点，但不是不可以。在什么情况下，节点仍然会连接到这样的对等节点？
</summary>
在自动连接逻辑中，节点试图连接到从其地址管理器中随机选择的地址。如果 50 次尝试后没有连接成功，它将开始考虑非默认的地址。一位与会者指出，功能测试中的节点也不使用默认端口，但有人指出，这些节点是使用手动连接，而不是自动出站连接。<a href="https://bitcoincore.reviews/23542#l-123">➚</a>
</details>

<details><summary>在这个 PR 之后，默认端口仍然在比特币核心中发挥作用。哪里还在使用它呢？
</summary>
当没有提供一个端口时，就会使用默认的。这与 DNS 种子特别相关，新的节点使用它来启动他们的地址管理器。需要找到一个替代方案才能完全删除默认端口的概念，因为 DNS 的设计是为了将域名解析为 IP 地址，而不是为了提供服务的地址和端口。<a href="https://bitcoincore.reviews/23542#l-137">➚</a>
</details>

<details><summary>在提交 <a herf="https://github.com/bitcoin-core-review-club/bitcoin/commit/d0abce9a50dd4f507e3a30348eabffb7552471d5">d0abce9</a> 中，允许调用者向 CServiceHash 传递盐，然后用 CServiceHash(0, 0) 初始化它，这是什么原因？
</summary>
节点大约每 24 小时公布一次自己的地址，每个节点都会转发收到的地址，以帮助网络中的节点发现新的对等节点。这段代码使用 IP 地址的哈希值和当前时间来随机挑选一个或两个对等节点来转发最近收到的地址。然而，我们不希望仅仅通过多次发送地址就能提高地址的传播率。因此，我们使用相同的盐（0，0）和时间戳颗粒度。<a href="https://bitcoincore.reviews/23542#l-197">➚</a>
</details>

## 新版本和候选版本

*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- [LDK 0.0.105][] 增加了对幽灵节点支付的支持（见 [Newsletter #188][news188 phantom]）和更好的概率支付寻路（见 [Newsletter #186][news186 pp]），此外还提供了一些其他功能和错误修复（包括[两个潜在的 DoS 漏洞][rl dos]）。

## 代码和文档的重大变更
*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Bitcoin Core #23542][] 删除了比特币核心只在默认的比特币端口（主网为 8333）连接对等节点的偏好。取而代之的是，比特币核心将在任何端口打开与对等节点的连接，除了已知被其他服务使用的几十个端口。端口 8333 仍然是比特币核心本地绑定的默认端口，所以只有覆盖默认端口的节点才会声明他们接受其他端口的连接。关于这个变更的一些额外信息可以在本通讯前面的 *Bitcoin Core PR 审核俱乐部会议*摘要中找到。

- [BDK #537][] 将钱包地址缓存重构为一个公共方法。以前，确保钱包内部数据库加载和缓存地址的唯一方法是通过一个内部函数，这意味着离线钱包没有机制来确保数据库加载地址。这个补丁使离线钱包被用作多签名者和验证更改地址等使用情况成为可能。与此相关，[BDK #522][] 增加了一个内部地址的 API，这对于应用程序创建一个将输出分成几个小的交易是非常有用的。

## 脚注

<a name="htlcs-essentially-free">1.</a>
    当用户 Alice 通过路由节点 Bob 和 Carol 向用户 Zed 转发一个基于 HTLC 的密钥发送消息时，Alice 可以用一个没有已知原像的哈希值来构造 HTLC，保证它会失败，因此 Bob 和 Carol 都不会收到任何钱。Alice 发送这样的信息所承担的成本仅有创建信道的成本（如果她创建了信道）和后来关闭信道的成本（如果她负责支付这些成本）——加上攻击者窃取她的 LN 热钱包的私钥或任何其他可能危及她的 LN 通道的数据的风险。对于一个安全的、没有错误的、具有长寿命通道的节点来说，这些成本应该基本为零，因此基于 HTLC 的密钥发送信息目前可以被认为是免费的。


[topic covenants]: https://bitcoinops.org/en/topics/covenants/
[topic tapscript]: https://bitcoinops.org/en/topics/tapscript/
[topic schnorr signatures]: https://bitcoinops.org/en/topics/schnorr-signatures/
[topic sidechains]: https://bitcoinops.org/en/topics/sidechains/
[topic onion messages]: https://bitcoinops.org/en/topics/onion-messages/
[topic htlc]: https://bitcoinops.org/en/topics/htlc/
[topic offers]: https://bitcoinops.org/en/topics/offers/
[topic amp]: https://bitcoinops.org/en/topics/atomic-multipath/
[topic channel jamming attacks]: https://bitcoinops.org/en/topics/channel-jamming-attacks/

[ldk 0.0.105]: https://github.com/lightningdevkit/rust-lightning/releases/tag/v0.0.105#security
[news185 optxhash]: https://bitcoinops.org/en/newsletters/2022/02/02/#composable-alternatives-to-ctv-and-apo
[news187 optx]: https://bitcoinops.org/en/newsletters/2022/02/16/#simplified-alternative-to-op-txhash
[rubin recurse]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019872.html
[op_cat]: https://bitcoinops.org/en/topics/op_checksigfromstack/#relationship-to-op_cat
[shinobi recurse]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019891.html
[news157 csfs]: https://bitcoinops.org/en/newsletters/2021/07/14/#request-for-op-checksigfromstack-design-suggestions
[darosior reply]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019892.html
[aj reply]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019923.html
[harding altcoin]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-July/019203.html
[obeirne reply]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019890.html
[cve-2013-2292]: https://bitcoinops.org/en/topics/cve/#CVE-2013-2292
[zmn turing]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019928.html
[zmn drivechains]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019976.html
[turing completeness]: https://en.wikipedia.org/wiki/Turing_completeness
[halting problem]: https://en.wikipedia.org/wiki/Halting_problem
[towns drivechains]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019984.html
[drivechains vs ln]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019991.html
[osuntokun bandwidth]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2022-February/003498.html
[russell reply]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2022-February/003499.html
[harding recurse]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-February/019885.html
[rl dos]: https://github.com/lightningdevkit/rust-lightning/blob/main/CHANGELOG.md#security
[news188 phantom]: https://bitcoinops.org/en/newsletters/2022/02/23/#ldk-1199
[news186 pp]: https://bitcoinops.org/en/newsletters/2022/02/09/#ldk-1227
[reviews 23542]: https://bitcoincore.reviews/23542
[BIP300]: https://github.com/bitcoin/bips/blob/master/bip-0300.mediawiki
[Bitcoin Core #23542]: https://github.com/bitcoin/bitcoin/issues/23542
[BDK #537]: https://github.com/bitcoindevkit/bdk/pull/537
[BDK #522]: https://github.com/bitcoindevkit/bdk/issues/522
[Bitcoin Core PR Review Club]: https://bitcoincore.reviews/

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

