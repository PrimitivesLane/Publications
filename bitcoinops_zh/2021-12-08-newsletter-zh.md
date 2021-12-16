---
title: 'Bitcoin Optech Newsletter #178'
permalink: /en/newsletters/2021/12/08/
name: 2021-12-08-newsletter
slug: 2021-12-08-newsletter
type: newsletter
layout: newsletter
lang: zh
---

本周的周报介绍了一篇关于手续费加速的研究，也包含了我们的常规部分：Bitcoin Core PR 审核俱乐部会议的总结、比特币软件的最新版本和候选版本、热门基础设施项目的重大变更。

## News

- <a id="fee-bumping-research" href="#fee-bumping-research)">●</a> **手续费加速研究**：Antoine Poinsot 在 Bitcoin-Dev 邮件组里发布了一篇[文章][posted]，详细讲解了开发者在选择用于 [vaults][vaults] 或合约协议（比如闪电网络）的已签名交易的手续费加速办法时应当关心的几个问题。尤其，Poinsot 查看了一些超过两方参与的多方协议所选的方案；对这些方案来说，当前的 “[子为父付（CPFP carve out）][CPFP carve out]” 交易转发策略不管用，所以它们必须使用可能会遭遇 “[交易钉死（transaction pinning）][transaction pinning]” 攻击的[交易替代（transaction replacement）][transaction replacement]机制。他的文章还包含了早前一点想法的[研究][research]结果。

  保证手续费加速方法能可靠运行，是绝大部分合约协议的安全性前提，而且迄今为止没有出现十全十美的解决方案。看到这个问题的持续研究，令人感到鼓舞。

##Bitcoin Core PR 审核俱乐部

*在这个每月一次的栏目中，我们总结最近一次的 [Bitcoin Core PR 审核俱乐部会议][Bitcoin Core PR Review Club]，集中展示一些重要的问题和答案。点击具体的问题可看到会议答案的总结。*

“[视 taproot 为永远激活][Treat taproot as always active]”是 Marco Falke 提出的一项代码更新，旨在让[taproot][taproot] 输出的花费行为标准化，不论 taproot 部署的情形如何。

<details><summary>代码库的哪个部分用到了 taproot 的激活状态？其中哪些与交易转发策略相关？
</summary>
在这个 PR 之前，有四个部分：<a href="https://github.com/bitcoin/bitcoin/blob/dca9ab48b80ff3a7dbe0ae26964a58e70d570618/src/validation.cpp#L1616">GetBlockScriptFlags()</a>（共识层）、<a href="https://github.com/bitcoin-core-review-club/bitcoin/blob/15d109802ab93b0af9647858c9d8adcd8a2db84a/src/validation.cpp#L726-L729">AreInputsStandard()</a>（交易转发）、<a href="https://github.com/bitcoin/bitcoin/blob/dca9ab48b80ff3a7dbe0ae26964a58e70d570618/src/rpc/blockchain.cpp#L1537">getblockchaininfo()</a>（rpc）以及 <a href="https://github.com/bitcoin-core-review-club/bitcoin/blob/15d109802ab93b0af9647858c9d8adcd8a2db84a/src/interfaces/chain.h#L292">isTaprootActive()</a>（钱包）。<a href="https://bitcoincore.reviews/23512#l-21">➚</a>
</details>

<details><summary>哪个交易池验证函数检查一笔交易是否花费了一个 taproot 输出？这个 PR 会怎样改变这个函数的行为？
</summary>
<a href="https://github.com/bitcoin/bitcoin/blob/dca9ab48b80ff3a7dbe0ae26964a58e70d570618/src/policy/policy.h#L110">`AreInputsStandard()`</a> 作这样的检查。这个 PR 移除了签名的最后一个参数 `bool taproot_active` 并为 v1 segwit（taproot）花费返回 ` true ` ，而不管 taproot 的激活状态。在此之前，如果该函数发现交易在花费一个 taproot 输出，但 ` taproot_active ` 还是  ` false ` ，那这个函数就会返回错误；例如，如果该节点还在初次下载的过程中，还在同步 taproot 激活之前的区块。<a href="https://bitcoincore.reviews/23512#l-40">➚</a>
</details>

<details><summary>这次变更在理论上有问题吗？钱包用户有没有可能丢币？
</summary>
有了这个更新，钱包就允许在任何使用引入 taproot <a href="https://bitcoinops.org/en/topics/output-script-descriptors/">描述符</a>，即，即使 taproot 还未激活而 v1 segwit 输出可以被任何人使用。这意味着你可以使用一个 taproot 输出来接收比特币，即使 taproot 还未激活；如果比特币区块链发生了重组，回到了 709632 以前的高度，矿工（或其他能让一笔非标准的交易打包进入区块的人）可以偷走这些 UTXO。<a href="https://bitcoincore.reviews/23512#l-82">➚</a>
</details>

<details><summary>仅仅从理论上来说，有没有可能存在一条主链，其中 taproot 永远不会被激活，或者在另外一个高度激活？
</summary>
两种情况都是有可能的，如果能发生一场非常大的重组的话（在 taproot 升级锁定区块之前分叉），那么激活过程就要重来一遍。如果在这条新链上，taproot 升级表态的区块数量达不到阈值，这条（仍然有效的）链就不会激活 taproot。如果表态数量在最低激活高度之后、超时之前达成，则 taproot 会在另一个高度激活。<a href="https://bitcoincore.reviews/23512#l-130">➚</a>
</details>
##新版本和候选版本

*流行的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- <a id="bdk-0-14-0" href="#bdk-0-14-0)">●</a> [BDK 0.14.0][BDK 0.14.0] 是这个钱包开发者库的最新版本。它简化了给一笔交易添加一个 `OP_RETURN ` 输出的流程，并为支付给 [bech32m][bech32m] 地址的 [taproot][taproot] 交易增加了一些升级。

## 重要的代码和文献变更

*本周出现重大变更的有 [Bitcoin Core][Bitcoin Core]、[C-Lightning][C-Lightning]、[Eclair][Eclair]、[LND][LND]、[Rust-Lightning][Rust-Lightning]、[libsecp256k1][libsecp256k1]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK] 以及 [Lightning BOLTs][Lightning BOLTs]。

- <a id="bitcoin-core-23155" href="#bitcoin-core-23155)">●</a> [Bitcoin Core #23155][Bitcoin Core #23155] 延伸了 ` dumprxoutset` RPC，可获得链状态（UTXO 集）快照的哈希值，以及整条链到此时为止的事务数量。这个信息可以跟链状态一起发布，这样其他人也能用 `gettxoutseinfo` RPC 来验证它，使之能被用于 [assumeUTXO][assumeUTXO] 节点的引导启动。
- <a id="bitcoin-core-22513" href="#bitcoin-core-22513)">●</a> [Bitcoin Core #22513][Bitcoin Core #22513] 允许  ` walletprocesspsbt ` 不走完 [PSBT][PSBT] 的流程就签名。这对于复杂的脚本是很有用的，比如，在一个有两个路径的 [tapscript][tapscript] 中：一个备用路径只需要 Alice 的签名；一个常用路径需要包括 Alice 在内的多人签名。在 Alice 签名时，最好是使用备用脚本路径推迟完成 PSBT、反过来构造一个包含 Alice 两个签名的 PSBT、并把这个 PSBT 发给其他签名者，等待签名完成。在这种情况下，最终的路径是在所有签名都完成之后决定的。
- <a id="c-lightning-4921" href="#c-lightning-4921)">●</a> [C-Lightning #4921][C-Lightning #4921] 升级了[洋葱消息][onion messages]的实现，来匹配
“[路径盲化（route blinding）][route blinding]” 和[洋葱消息][onion messages]规范草案的最新升级。
- <a id="c-lightning-4829" href="#c-lightning-4829)">●</a> [C-Lightning #4829][C-Lightning #4829] 为 [BOLTs #911][BOLTs #911] 提出的闪电网络协议变更加入了实验性的支持。该升级将允许节点为自己的 DNS 地址（而不是 IP 地址或洋葱服务地址）打广告。
- <a id="eclair-2061" href="#eclair-2061)">●</a> [Eclair #2061][Eclair #2061] 为[洋葱消息][onion messages]加入了初步支持。用户可以启用 ` option_onion_message ` 功能来转发洋葱消息，还可以使用 ` sendonionmessage ` RPC 来发送洋葱消息。处理进入的洋葱消息和[路线盲化][route blinding]的功能还未实现。
- <a id="eclair-2073" href="#eclair-2073)">●</a> [Eclair #2073][Eclair #2073] 根据 [BOLTs #906][BOLTs #906] 草案的详述，为可选的通道类型谈判特征位添加了支持。这与 LND [上周][last week]对同一草案特性的实现是一致的。
- <a id="rust-lightning-1163" href="#rust-lightning-1163)">●</a> [Rust-Lightning #1163][Rust-Lightning #1163] 允许远程参与者将他们的通道预留值设得低于粉尘限制，甚至可以设到 0。在最糟糕的情况下，这个特性允许本地节点无成本地尝试从一个花完的通道中偷取资金 —— 虽然这样的尝试在监控着通道的远程参与者那里还是会失败的。默认情况下，大部分远程节点都会通过设置一个合理的通道预留值来阻止这样的尝试，但一些闪电服务提供者（Lightning Service Providers，LSP）使用很低甚至 0 的通道预留值来为用户提供更好的体验 —— 让他们可以 100% 花完通道内的资金。因为只有远程节点承担风险，本地节点接受这样的通道也是没问题的。

[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-November/019614.html

[vaults]:https://bitcoinops.org/en/topics/vaults/

[CPFP carve out]:https://bitcoinops.org/en/topics/cpfp-carve-out/

[transaction replacement]:https://bitcoinops.org/en/topics/replace-by-fee/

[transaction pinning]:https://bitcoinops.org/en/topics/transaction-pinning/

[research]:https://github.com/revault/research

[Bitcoin Core PR Review Club]:https://bitcoincore.reviews/

[Treat taproot as always active]:https://bitcoincore.reviews/23512

[taproot]:https://bitcoinops.org/en/topics/taproot/

[BDK 0.14.0]:https://github.com/bitcoindevkit/bdk/releases/tag/v0.14.0

[bech32m]:https://bitcoinops.org/en/topics/bech32/

[taproot]:https://bitcoinops.org/en/topics/taproot/

[Bitcoin Core]:https://github.com/bitcoin/bitcoin

[C-Lightning]:https://github.com/ElementsProject/lightning

[Eclair]:https://github.com/ACINQ/eclair

[LND]:https://github.com/lightningnetwork/lnd/

[Rust-Lightning]:https://github.com/rust-bitcoin/rust-lightning

[libsecp256k1]:https://github.com/bitcoin-core/secp256k1

[Rust Bitcoin]:https://github.com/rust-bitcoin/rust-bitcoin

[BTCPay Server]:https://github.com/btcpayserver/btcpayserver/

[BDK]:https://github.com/bitcoindevkit/bdk

[Lightning BOLTs]:https://github.com/lightning/bolts

[Bitcoin Core #23155]:https://github.com/bitcoin/bitcoin/issues/23155

[assumeUTXO]:https://bitcoinops.org/en/topics/assumeutxo/

[Bitcoin Core #22513]:https://github.com/bitcoin/bitcoin/issues/22513

[PSBT]:https://bitcoinops.org/en/topics/psbt/

[tapscript]:https://bitcoinops.org/en/topics/tapscript/

[C-Lightning #4921]:https://github.com/ElementsProject/lightning/issues/4921

[onion messages]:https://bitcoinops.org/en/topics/onion-messages/

[route blinding]:https://github.com/lightning/bolts/issues/765

[onion messages]:https://github.com/lightning/bolts/issues/759

[C-Lightning #4829]:https://github.com/ElementsProject/lightning/issues/4829

[BOLTs #911]:https://github.com/lightning/bolts/issues/911

[Eclair #2061]:https://github.com/ACINQ/eclair/issues/2061

[onion messages]:https://github.com/lightning/bolts/issues/759

[route blinding]:https://github.com/lightning/bolts/issues/765

[Eclair #2073]:https://github.com/ACINQ/eclair/issues/2073

[BOLTs #906]:https://github.com/lightning/bolts/issues/906

[last week]:https://bitcoinops.org/en/newsletters/2021/12/01/#lnd-6026

[Rust-Lightning #1163]:https://github.com/rust-bitcoin/rust-lightning/issues/1163

