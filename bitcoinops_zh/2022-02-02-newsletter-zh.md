---
title: 'Bitcoin Optech Newsletter #185'
permalink: /en/newsletters/2022/02/02/
name: 2022-02-02-newsletter
slug: 2022-02-02-newsletter
type: newsletter
layout: newsletter
lang: zh
---





本周的周报介绍了一份关于 ` OP_CHECKTEMPLATEVERIFY ` 操作码（CTV）对谨慎日志合约（discreet log contracts，DLC）的影响的分析，以及关于为 tapscript 启用 CTV 和 ` SIGHASH_ANYPREVOUT ` 的替代性变更方案的讨论。还包含了我们的常规部分：软件新版本的宣布以及流行比特币基础设施软件的重大变更。

## 新闻

- <a id="improving-dlc-efficiency-by-changing-script" href="#improving-dlc-efficiency-by-changing-script)">●</a> **通过改变脚本来提高 DLC 的效率**：Lloyd Fournier 在 DLC-Dev 和 Bitcoin-Dev 邮件组中[发帖][posted]论述 [OP_CHECKTEMPLATEVERIFY][OP_CHECKTEMPLATEVERIFY]（CTV）操作码可以大幅减少创建特定类型的谨慎日志合约（[DLCs][DLCs]）所需的签名数量，以及其它某些操作的数量。

  简而言之，对一个合约的每一种可能的终止状态 —— 比如 Alice 获得 1 BTC，Bob 获得 2 BTC —— 来说，当前的 DLC 需要为该状态创建一个专门的[适配器签名][signature adaptor]。许多合约都会定义大量可能的终止状态的，比如一个关于比特币期货价格的合约，精确到美元价格的个位数，即使是相对短期的合约，也需要覆盖几千美元的价格区间。为此参与者需要创建、交换和存储几千个签名碎片。

  与之相对的是，Fournier 主张几千个可能的状态可以使用 CTV 来承诺可能要上链的输出、并容纳在一个 [tapleaf][tapleaf] 中。CTV 使用哈希值来承诺输出，所以参与者可以很快地按需计算出所有可能状态的哈希值，从而最小化计算量、数据交换和数据存储。虽然依然需要一些签名，但数量已经极大地减少。

  Jonas Nick [指出][noted]，使用 [SIGHASH_ANYPREVOUT][SIGHASH_ANYPREVOUT] 签名哈希模式也可以实现类似的优化（而我们要指出，同样的优化用下面一条新闻所述的另类方案也可以实现）。

- <a id="composable-alternatives-to-ctv-and-apo" href="#composable-alternatives-to-ctv-and-apo)">●</a> CTV 和 APO 的可组合替代品：Russell O'Connor 在 Bitcoin-Dev 邮件组中[发帖][posted]列举了使用软分叉给比特币的 [Tapscript][Tapscript] 语言增加一或两个新的操作码的思路。Tapscript 可以使用新的  ` OP_TAHASH ` 操作码来指定一笔花费交易的哪个部分应该被序列化和哈希运算，而且哈希值还可以放到栈上供后续的操作码使用。一个新的 [OP_CHECKSIGFROMSTACK][OP_CHECKSIGFROMSTACK]（CSFS）操作码（如之前所提议的）将允许 tapscript 指定一个公钥并要求对栈上具体数据 —— 比如由  ` OP_TXHASH `  创建的交易摘要 —— 的一个相应的签名。

  O'Connor 解释了这两个操作码如何能模拟两个更早的软分叉提议，[SIGHASH_ANYPREVOUT][SIGHASH_ANYPREVOUT]（APO，详述为 [BIP118][BIP118]）和 [OP_CHECKTEMPLATEVERIFY][OP_CHECKTEMPLATEVERIFY]（CTV，详述为 [BIP119][BIP119]）。出于某些原因，这种模拟可能不如直接使用 CTV 或者 APO 那么有效率，但  ` OP_TXHASH `  和  ` OP_CSFS `  可以让 Tapscript 语言保持简单，并为未来的脚本编程提供更多的灵活性，尤其是在跟其它向 tapscript 增加内容的提议（比如 [OP_CAT][OP_CAT]）相结合的时候。

  在一个[回复][reply]中，Anthony Towns 也使用其它操作码提出了类似的方法。

  截至本稿撰写之时，这个想法仍在积极讨论中。我们预计会在未来的周报中再次讨论这个问题。

## 新版本和候选版本

*流行的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- <a id="btcpay-server-1-4-2" href="#btcpay-server-1-4-2)">●</a> [BTCPay Server 1.4.2][BTCPay Server 1.4.2] 是新的 1.4.x 系列的最新版本，包含了登录授权流程的提升以及一系列[使用界面升级][user interface improvements]。
- <a id="bdk-0-16-0" href="#bdk-0-16-0)">●</a> [BDK 0.16.0][BDK 0.16.0] 是最新版本，包含了一系列 bug 修复和小型升级。
- <a id="eclair-0-7-0" href="#eclair-0-7-0)">●</a>[Eclair 0.7.0][Eclair 0.7.0] 是一个新的大版本，增加了对[锚点输出][anchor outputs]（anchor outputs）、[洋葱消息][onion messages]转发的支持，并在生产环境中使用 PostgreSQL 数据库。
- <a id="lnd-0-14-2-beta-rc1" href="#lnd-0-14-2-beta-rc1)">●</a>[LND 0.14.2-beta.rc1][LND 0.14.2-beta.rc1] 是一个维护版本的候选版本，包含了多个 bug 修复和一些小型升级。

## 重大的代码和文档变更

本周出现重大变更的包括：[Bitcoin Core][Bitcoin Core]、[C-Lightning][C-Lightning]、[Eclair][Eclair]、[LDK][LDK]、[LND][LND]、[libsecp256k1][libsecp256k1]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK]、[Bitcoin Improvement Proposals (BIPs)][ Bitcoin Improvement Proposals (BIPs)] 和 [Lightning BOLTs][Lightning BOLTs]。

- <a id="bitcoin-core-23201" href="#bitcoin-core-23201)">●</a> [Bitcoin Core #23201][Bitcoin Core #23201] 增强了钱包用户使用外部输入来构造交易的能力（此前在[170 期周报][Newsletter #170]提及），办法是允许用户指定最大重量而非解析数据的上限。这使得使 ` fundrawtransaction`、`send `  和 `walletfundpsbt ` 的应用可以为非标准的输出（比如 [HTLCs][HTLCs]，是闪电网络客户端的必要组件）使用手续费追加交易（亦见 [184 期周报][Newsletter #184]）。
- <a id="eclair-2141" href="#eclair-2141)">●</a> [Eclair #2141][Eclair #2141] 增强了自动化的手续费追加机制（[184 期周报][Newsletter #184]亦提及），办法是在钱包的 UTXO 数量较少时选择一个更激进的确认目标。如此一来，让手续费加速交易能尽快确认、保持钱包的 UTXO 计数以防止进一步的强制关闭，就很重要。更多关于 Eclair 所用的锚点输出类型的手续费追加机制的细节，可见[此处][here]。
- <a id="btcpay-server-3341" href="#btcpay-server-3341)">●</a> [BTCPay Server #3341][BTCPay Server #3341] 允许用户在通过闪电网络请求撤回资金时自主配置 [BOLT11][BOLT11] 的超时时间（以往默认是 30 天）。

[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019808.html

[OP_CHECKTEMPLATEVERIFY]:https://bitcoinops.org/en/topics/op_checktemplateverify/

[DLCs]:https://bitcoinops.org/en/topics/discreet-log-contracts/

[signature adaptor]:https://bitcoinops.org/en/topics/adaptor-signatures/

[tapleaf]:https://bitcoinops.org/en/topics/tapscript/

[noted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019812.html

[SIGHASH_ANYPREVOUT]:https://bitcoinops.org/en/topics/sighash_anyprevout/

[posted]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019813.html

[Tapscript]:https://bitcoinops.org/en/topics/tapscript/

[OP_CHECKSIGFROMSTACK]:https://bitcoinops.org/en/topics/op_checksigfromstack/

[SIGHASH_ANYPREVOUT]:https://bitcoinops.org/en/topics/sighash_anyprevout/

[BIP118]:https://github.com/bitcoin/bips/blob/master/bip-0118.mediawiki

[OP_CHECKTEMPLATEVERIFY]:https://bitcoinops.org/en/topics/op_checktemplateverify/

[BIP119]:https://github.com/bitcoin/bips/blob/master/bip-0119.mediawiki

[OP_CAT]:https://bitcoinops.org/en/topics/op_checksigfromstack/#relationship-to-op_cat

[reply]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-January/019819.html

[BTCPay Server 1.4.2]:https://github.com/btcpayserver/btcpayserver/releases/tag/v1.4.2

[user interface improvements]:https://blog.btcpayserver.org/btcpay-server-1-4-0/

[BDK 0.16.0]:https://github.com/bitcoindevkit/bdk/releases/tag/v0.16.0

[Eclair 0.7.0]:https://github.com/ACINQ/eclair/releases/tag/v0.7.0

[anchor outputs]:https://bitcoinops.org/en/topics/anchor-outputs/

[onion messages]:https://bitcoinops.org/en/topics/onion-messages/

[LND 0.14.2-beta.rc1]:https://github.com/lightningnetwork/lnd/releases/tag/v0.14.2-beta.rc1

[Bitcoin Core]:https://github.com/bitcoin/bitcoin

[C-Lightning]:https://github.com/ElementsProject/lightning

[Eclair]:https://github.com/ACINQ/eclair

[LDK]:https://github.com/lightningdevkit/rust-lightning

[LND]:https://github.com/lightningnetwork/lnd/

[libsecp256k1]:https://github.com/bitcoin-core/secp256k1

[Rust Bitcoin]:https://github.com/rust-bitcoin/rust-bitcoin

[BTCPay Server]:https://github.com/btcpayserver/btcpayserver/

[BDK]:https://github.com/bitcoindevkit/bdk

[Lightning BOLTs]:https://github.com/lightning/bolts

[Bitcoin Core #23201]:https://github.com/bitcoin/bitcoin/issues/23201

[Newsletter #170]:https://bitcoinops.org/en/newsletters/2021/10/13/#bitcoin-core-17211

[HTLCs]:https://bitcoinops.org/en/topics/htlc/

[Newsletter #184]:https://bitcoinops.org/en/newsletters/2022/01/26/#eclair-2113

[Eclair #2141]:https://github.com/ACINQ/eclair/issues/2141

[Newsletter #184]:https://bitcoinops.org/en/newsletters/2022/01/26/#eclair-2113

[here]:https://bitcoinops.org/en/topics/anchor-outputs/

[BTCPay Server #3341]:https://github.com/btcpayserver/btcpayserver/issues/3341
[ Bitcoin Improvement Proposals (BIPs)]: https://github.com/bitcoin/bips/
[BOLT11]:https://github.com/lightningnetwork/lightning-rfc/blob/master/11-payment-encoding.md

