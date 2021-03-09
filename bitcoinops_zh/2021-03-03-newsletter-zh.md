---
title: 'Bitcoin Optech Newsletter #138'
permalink: /zh/newsletters/2021/03/03/
name: 2021-03-03-newsletter-zh 
slug: 2021-03-03-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 介绍了一个关于 BIP70 支付协议的一些功能的想要的替换的讨论，并总结了一些提案，这些提案是为慎重日志合约（DLC）交换欺诈证明提供标准化方法。此外，还包括我们的常规部分：新版本和可用的候选版本的公告，常见比特币基础设施软件的重要更新。

## News

* **讨论 BIP70 的替换问题：** Thomas Voegtlin 在 Bitcoin-Dev 邮件列表上发起了一个关于替代 [BIP70](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki) 支付协议部分功能的[帖子](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018443.html)，特别是接收签名支付请求的能力。Voegtlin 希望能够证明他支付的地址实际上是收件人提供给他的地址（例如：交易所）。Charles Hill 和 Andrew Kozlik 分别回复了他们正在研究的协议的信息。Hill 的[方案](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018446.html)旨在与 [LNURL](https://github.com/fiatjaf/lnurl-rfc) 一起使用，但可以重新用于 Voegtlin 的预期用途。Kozlik 的[方案](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018448.html)在精神上更接近于 BIP70，但放弃了对 [X.509 证书](https://en.wikipedia.org/wiki/X.509)的使用，并增加了基于交易所的币种交换的功能（例如：用 BTC 换取山寨币或反之）。)

* **保密日志合约（DLC）规范中的欺诈证明 v0 版本：** Thibaut Le Guilly 在 DLC-dev 邮件列表上发起了关于在 DLC 版本 0 的协调规范中加入欺诈证明的[目的](https://github.com/discreetlogcontracts/dlcspecs/blob/master/v0Milestone.md#simple-fraud-proofs-in-progress)的[讨论](https://mailmanlists.org/pipermail/dlc-dev/2021-February/000020.html)。讨论了两种类型的欺诈方式：
  * 模棱两可：一个预言机在同一事件中不止一次地发出信号，产生相互矛盾的结果。模棱两可证明可以由软件自动验证，无需第三方信任。
  * 撒谎：其中，预言机签收了用户知道是错误的结果。几乎无法从用户的合约软件中获取关键证据，所以这种撒谎证明必须由用户手动验证，用户可以将原始合约与预言机签署的结果进行对比。

  讨论的参与者似乎都赞成提供一个等价证明，尽管有人担心这对 v0 规范来说可能会有太多的工作。作为中间解决方法，建议专注于撒谎证明。当这些证明的格式确定后，就可以更新软件，对同一预言机和事件分别采取两个独立的证明，来创建一个模棱两可证明。

  对撒谎证明的一个担忧是，用户可能会被假证明淹没，迫使用户要么浪费时间验证假证明，要么完全放弃检查欺诈证明。相反的论点有 1. 用户能够从 onchain 交易中获得部分证明（这需要有人支付 onchain 费用），2. 用户可以选择从不同地方下载欺诈证明，这样用户会更倾向于从一个只传播准确信息的来源获得证明。

## Notable code and documentation changes
*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[硬件钱包接口 (HWI)](https://github.com/bitcoin-core/HWI)、[Rust 比特币](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay 服务器](https://github.com/btcpayserver/btcpayserver/)、[比特币改进提案 (BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 的重要更新。*

* [Bitcoin Core #16546](https://github.com/bitcoin/bitcoin/issues/16546) 引入了一个新的签名接口，允许 Bitcoin Coin 客户端通过 [HWI](https://bitcoinops.org/en/topics/hwi/) 或任何其他实现了相同接口的应用程序与外部硬件签名设备进行交互。

  [从 Bitcoin Core 0.18 版本开始](https://bitcoinops.org/en/newsletters/2019/05/07/#basic-hardware-signer-support-through-independent-tool)，Bitcoin Core 就可以使用 HWI 与硬件签名设备进行对接。但在本次 PR 之前，[这个过程](https://github.com/bitcoin-core/HWI/blob/7b34fc72c5b2c5af216d8b8d5cd2d2c92b6d2457/docs/examples/bitcoin-core-usage.rst)需要使用命令行在 Bitcoin Core 和 HWI 之间传输数据。这个 PR 简化了用户体验，让比特币核心可以直接与 HWI 通信。该 PR 包括怎样使用新的签名设备和 HWI 进行对接的[完整文档](https://github.com/bitcoin/bitcoin/blob/master/doc/external-signer.md)。

  新的签名设备接口目前只能通过 RPC 方法访问。一个 [PR 草案](https://github.com/bitcoin-core/gui/pull/4)为 GUI 增加了对签名器接口的支持，允许在不使用任何命令行的情况下使用硬件签名设备签名。

* [Rust-Lightning #791](https://github.com/rust-bitcoin/rust-lightning/issues/791) 增加了对启动时轮询 `BlockSource` 接口的支持，以同步块和头，并在同步期间进行分叉检测。正如[通讯#135](https://bitcoinops.org/en/newsletters/2021/02/10/#rust-lightning-774) 中所述，`BlockSource` 允许软件从标准比特币核心兼容节点以外的来源获取数据，允许冗余，有助于防止日食攻击或其他安全问题。

* [Rust-Lightning #794](https://github.com/rust-bitcoin/rust-lightning/issues/794) 启用了对 [BOLT2](https://github.com/lightningnetwork/lightning-rfc/blob/master/02-peer-protocol.md) `option_shutdown_anysegwit` 功能的支持，该功能允许状态通道在开始关闭时使用未来的 segwit 版本。如果协商了 `option_shutdown_anysegwit`，则发送关闭消息来启动关闭的通道方可以发送 scriptpukey 进行支付，但脚本必须符合标准的 [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki) 见证程序格式，即一个版本字节（OP_1 到 OP_16 的 1 字节 push 操作码），然后是一个见证程序（2 到 40 字节的字节向量 push）。这些关闭脚本只限于标准格式，以避免昂贵的有高费用脚本交易或因为不符标准的超大脚本而无法传播的交易。由于在 Bitcoin Core 0.19.0.1（2019 年 11 月发布）中，向任何 segwit 脚本中转支付成为[可能](https://bitcoincore.org/en/releases/0.19.0.1/#mempool-and-transaction-relay)，现在可以安全地[将它们包含](https://github.com/lightningnetwork/lightning-rfc/issues/672)在 LN 的标准格式中。

* [HWI #413](https://github.com/bitcoin-core/HWI/issues/413)、[#469](https://github.com/bitcoin-core/HWI/issues/469)、[#463](https://github.com/bitcoin-core/HWI/issues/463)、[#464](https://github.com/bitcoin-core/HWI/issues/464)、[#471](https://github.com/bitcoin-core/HWI/issues/471)、[#468](https://github.com/bitcoin-core/HWI/issues/468) 和 [#466](https://github.com/bitcoin-core/HWI/issues/466) 大大更新和扩展了 HWI 的文档。特别值得注意的变化包括 [ReadTheDocs.io](https://hwi.readthedocs.io/en/latest/?badge=latest) 上的文档链接，新的和更新的[示例](https://hwi.readthedocs.io/en/latest/examples/index.html)，以及描述新设备必须满足的标准的新[政策](https://hwi.readthedocs.io/en/latest/devices/index.html#support-policy)，以便 HWI 考虑支持它们。

* [Rust Bitcoin #573](https://github.com/rust-bitcoin/rust-bitcoin/issues/573) 增加了一个新的方法 `SigHashType::from_u32_standard`，确保提供的 sighash 字节是[标准值](https://btcinformation.org/en/developer-guide#signature-hash-types)之一，Bitcoin Core 默认会转发和挖矿。每个签名的 sighash 字节表示交易的哪些部分需要签名。比特币的共识规则决定了非标准的 sighash 值被视为等同于 `SIGHASH_ALL`，但事实上，它们没有被转发或默认挖矿，理论上可以用来欺骗使用链外承诺的软件，使其接受无法执行的支付。使用 Rust Bitcoin 的这类软件的开发者可以从 `SigHashType::from_u32` 方法切换到这个新方法，该方法接受任何共识有效的 sighash 字节。

* [BIPs #1069](https://github.com/bitcoin/bips/issues/1069) 更新了 [BIP8](https://github.com/bitcoin/bips/blob/master/bip-0008.mediawiki)，允许配置激活阈值，并根据最近关于 [taproot 激活的讨论](https://bitcoinops.org/en/newsletters/2021/02/24/#taproot-activation-discussion)，将 90% 作为建议的激活阈值，从以前的 95% 降了一点。