---
title: 'Bitcoin Optech Newsletter #194'
permalink: /zh/newsletters/2022/04/06/
name: 2022-04-06-newsletter-zh
slug: 2022-04-06-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---
本周的 Newsletter 介绍了一个关于脱钩的可重用地址的提案，总结了 WabiSabi 协议如何被用作 payjoin 的增强替代方案，查看了关于为 DLC 规范添加通信标准的讨论，并关注了关于更新 LN 承诺格式的新讨论。此外，还包括我们的常规部分，总结了新版本和候选版本，以及主流的比特币基础设施软件中值得注意的变更。


## 新闻

- **脱钩的可重用地址：** Ruben Somsen 在 Bitcoin-Dev 邮件列表中发布了一个[提案][somsen silpay]，允许某人向一个公共标识符（"地址"）付款而不在链上使用该标识符。例如，Alice 以公钥的形式创建了一个标识符，她在她的网站上发布了这个标识符。Bob 能够使用他自己的一个私钥将 Alice 的公钥转化为一个新的比特币地址，并使用这个地址来支付给 Alice。只有 Alice 和 Bob 有必要的信息来确定该地址是支付给 Alice 的，也只有 Alice 有必要的信息（她的私钥）来花费该地址收到的资金。如果 Carol 后来去了 Alice 的网站并重新使用 Alice 的公共标识符，她会得出一个不同的地址来支付给 Alice，这个地址无论是 Bob 还是任何其他第三方都无法直接确定属于 Alice。

  以前的脱钩的可重用地址方案，如 [BIP47][] 可重用支付代码和未发布的 BIP63 隐形地址，要么依赖于区块链外的花费者和接收者之间的通信，要么通过包含额外数据的链上交易导致费用比普通付款要高。相比之下，Somsen 的*静默支付*提案与普通交易相比没有额外的开销。

  静默支付的最大缺点是，检查新收到的交易需要扫描每个新区块中的每笔交易。这是全节点已经在做的事情，但该方案还需要维护许多交易的父交易的信息，这是今天许多全节点不做的事情，这可能需要相当多的额外 I/O 操作，以及额外的 CPU 计算。从备份中恢复一个钱包也可能是很麻烦的。尽管 Somsen 描述了一些可能的方法来减少负担，但对于大多数想要接收付款的轻钱包来说，该方案似乎不太可能有用。然而，几乎所有的钱包都愿意在仅增加几行代码且没有明显增加资源需求的情况下支持发送静默支付。这可能使得该计划的成本主要由最希望增强隐私的用户承担，而这些用户不能或不愿意使用其他机制来防止[地址重用][topic output linking]。

  Somsen 正在寻求对该提案的反馈，包括对其使用密码学的安全分析，以及使其对接收者的资源密集度要求降低的任何建议（不明显降低其隐私优势）。

- **WabiSabi 替代 payjoin：** 几周前，Wasabi 钱包和 [coinjoin][topic coinjoin] 的开发者在 Bitcoin-Dev 邮件列表中[宣布][wasabi2]了一个新版本，支持 [Newsletter #102][news102 wabisabi] 中介绍的 WabiSabi 协议。之前使用的 chaumian coinjoin 协议允许任意的输入金额，但只支持固定大小的输出金额；增强的 WabiSabi 协议允许输入和输出金额都是任意值（受其他约束，如限制最大交易大小和[尘额限制][topic uneconomical outputs]）。

  本周，Max Hillebrand [发布][hillebrand wormhole]了一个关于 WabiSabi 协议如何被用作 [payjoin][topic payjoin] 协议的替代品的介绍。在标准的 payjoin 中，发送方和接收方都为交易提供输入并接收输出。尽管每一方都了解对方的输入和输出，但对于参与 payjoin 的用户和其他多输入的交易来说，公共区块上的所有权信息是不明确的。Hillebrand 提出的虫洞 2.0 协议将使用 WabiSabi 来确保支付方或接收方都不会了解对方的输入或输出（前提是 WabiSabi coinjoin 包含其他多个具有保密性的参与方）。这将提供更多的隐私，尽管它需要使用实现 WabiSabi 的软件，并等待一个已被协调的 WabiSabi coinjoin 交易的发生。

- **DLC 的通信和网络组织：** Thibaut Le Guilly 在 DLC-Dev 邮件列表中[发布][leguilly
  dlcmsg]了关于改善不同 DLC 实现之间的通信的内容。当前，不同的实现方案使用各种不同的方法来发现并与其他 DLC 节点通信。这些不同方案使使用一种实现的节点无法与使用其他实现的节点进行互操作。Le Guilly 指出，制定 DLC 标准的意义在于允许互操作，因此他建议在标准中增加相关细节。

  邮件列表中讨论了几个细节。特别是，Le Guilly 希望在可能的情况下重新使用 LN 规范的元素（BOLTs）。这使得有人提到最近关于升级 LN 通道广播的建议（见 [Newsletter #193][news193
 major update]），该建议允许使用任何 UTXO 进行防洪泛 DoS 保护（不仅仅是用看起来像 LN 设置交易的 UTXO），这将允许 DLC 和其他二层协议使用相同的抗 DoS 保护。

- **更新 LN 承诺：** Olaoluwa Osuntokun 本周在 Lightning-Dev 邮件列表中[发布][osuntokun dyncom]了他之前关于[升级通道承诺格式][topic channel commitment upgrades]或升级任何其他会影响承诺交易的设置的后续建议。关于他之前的建议，请看 [Newsletter #108][news108 upcom] 的总结。目标是即使通道增加了新的功能（例如[使用 taproot][zmn post] 功能），也能让通道保持不关闭。

  Osuntokun 的提议与 [BOLTs #868][] 中描述的、在C-Lightning中实验性实现的、[Newsletter #152][news152 cl4532] 中提到的另一个提案形成对比。这两个提案之间的一个值得注意的区别是，Osuntokun 的提案是在可能有新的支付（[HTLC][topic htlc]）的情况下升级信道，而 BOLTS #868 提案定义了一个安静期，在此期间可以进行升级。

## 新版本和候选版本

*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- [Bitcoin Core 23.0 RC2][] 是这个主流的全节点软件的下一个主要版本的候选版本。[发布草案说明][bcc23 rn]列出了多项改进，并鼓励高级用户和系统管理员在最终发布前进行[测试][test guide]。

- [LND 0.14.3-beta.rc1][] 是这个流行的 LN 节点软件的候选发布版本，包含了几个错误修复。

- [C-Lightning 0.11.0rc1][] 是这个流行的 LN 节点软件的下一个主要版本的候选发布版本。


## 代码和文档的重大变更
*本周内，[Bitcoin Core][bitcoin core repo]、[C-Lightning][c-lightning repo]、[Eclair][eclair repo]、[LND][lnd repo]、[Rust-Lightning][rust-lightning repo]、[libsecp256k1][libsecp256k1 repo]、[Hardware Wallet Interface (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[Bitcoin Improvement Proposals (BIPs)][bips repo] 和 [Lightning BOLTs][bolts repo] 出现的重大变更。*

- [Bitcoin Core #24118][] 增加了一个新的 RPC，`sendall`，它清空一个钱包将资金发送给一个收款地址。该调用的选项可以用来指定额外的收款人，指定钱包的 UTXO 池的一个子集作为输入，或者通过跳过尘额而不是使用所有 UTXO 来最大化收款人的收款金额。`sendall` 因此提供了一个方便的选择，以实现其他 `send* RPC` 的 `fSubtractFeeAmount` 参数的一些应用，但 `fSubtractFeeAmount` 仍然是支付给负责交易费用的收款人的最佳选择。

- [Bitcoin Core #23536][] 设置了验证规则，只要 segwit 被强制执行，就一定会执行 [taproot][topic taproot]，包括验证历史区块（除了区块 692261，其中包括一个在 taproot 激活前花费 v1 见证输出的交易）。这种*埋藏式部署*也在 P2SH 和 segwit 软分叉上做过（见 [Newsletter #60][news60 buried]）；它简化了测试和代码评审，减轻了部署代码中潜在 bug 的一些风险，并为极端情况提供缓冲保护，让节点下载一个替代的、taproot 没有激活的情况下的包含最大可工作集的区块链。

- [Bitcoin Core #24555][] 和 [Bitcoin Core #24710][]增加了在 CJDNS 网络上运行比特币核心的[文档][cjdns.md]（见 [Newsletter #175][news175 cjdns]）。

- [C-Lightning #5013][] 增加了使用 mTLS 认证的 gRPC 管理节点的能力。

- [C-Lightning #5121][] 用一个新的 `deschash` 参数更新了 `invoice` RPC，该参数将包括 [BOLT11][] 收据中任意数据的哈希值，而不是一个简短的描述字符串。这是由 [LNURL][] 等方案的支持的，这些方案可以通过单独的通信渠道发送大型描述（包括像图片这样的数据），而不受 BOLT11 的限制。

- [Eclair #2196][] 添加了一个 `channelbalances` API，列出了所有节点的通道的余额，包括已经被禁用的通道。

- [LND #6263][] 为 LND 的钱包增加了对 [taproot][topic taproot] 密钥路径支付的支持，现在可以使用默认的 signet 进行测试。

- [Libsecp256k1 #1089][] 将 `secp256k1_schnorrsig_sign()` 函数更名为 `..._sign32()`，使其更清楚地表明该函数签署 32 字节的信息（例如 SHA256 哈希值）。这与 `secp256k1_schnorrsig_sign_custom()` 函数形成对比，后者签署任意长度的消息（见 [Newsletter #157][news157 schnorrsig] 的补充讨论）。

- [Rust Bitcoin #909][] 通过传入特定脚本路径被使用的概率,增加了对使用 [huffman 编码][huffman coding]创建最佳 taproot 脚本路径树的支持。

- [LDK #1388][] 通过找到一个可以更紧凑地编码的值增加了对创建比平均情况更小的 ECDSA 签名的默认支持，这个过程以前在 Bitcoin Core 和 C-Lightning 的钱包中实现（见[Newsletters #8][news8 lowr] 和 [#71][news71 lowr]）。这种[低-r grinding][topic low-r grinding] 模式将在链上交易中为每个 LDK 对等节点节省大约 0.125 vbytes。


[topic output linking]: https://bitcoinops.org/en/topics/output-linking/
[topic coinjoin]: https://bitcoinops.org/en/topics/coinjoin/
[topic uneconomical outputs]: https://bitcoinops.org/en/topics/uneconomical-outputs/
[topic payjoin]: https://bitcoinops.org/en/topics/payjoin/
[topic channel commitment upgrades]:https://bitcoinops.org/en/topics/channel-commitment-upgrades/
[topic htlc]: https://bitcoinops.org/en/topics/htlc/
[topic taproot]: https://bitcoinops.org/en/topics/taproot/
[topic low-r grinding]: https://bitcoinops.org/en/topics/low-r-grinding/

[bitcoin core 23.0 rc2]: https://bitcoincore.org/bin/bitcoin-core-23.0/
[bcc23 rn]: https://github.com/bitcoin-core/bitcoin-devwiki/wiki/23.0-Release-Notes-draft
[test guide]: https://github.com/bitcoin-core/bitcoin-devwiki/wiki/23.0-Release-Candidate-Testing-Guide
[lnd 0.14.3-beta.rc1]: https://github.com/lightningnetwork/lnd/releases/tag/v0.14.3-beta.rc1
[C-Lightning 0.11.0rc1]: https://github.com/ElementsProject/lightning/releases/tag/v0.11.0rc1
[news157 schnorrsig]: https://bitcoinops.org/en/newsletters/2021/07/14/#libsecp256k1-844
[news8 lowr]: https://bitcoinops.org/en/newsletters/2018/08/14/#bitcoin-core-wallet-to-begin-only-creating-low-r-signatures
[news71 lowr]: https://bitcoinops.org/en/newsletters/2019/11/06/#c-lightning-3220
[news108 upcom]: https://bitcoinops.org/en/newsletters/2020/07/29/#upgrading-channel-commitment-formats
[news152 cl4532]: https://bitcoinops.org/en/newsletters/2021/06/09/#c-lightning-4532
[zmn post]: https://bitcoinops.org/en/newsletters/2021/09/01/#preparing-for-taproot-11-ln-with-taproot
[osuntokun dyncom]: https://lists.linuxfoundation.org/pipermail/lightning-dev/2022-March/003531.html
[somsen silpay]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020180.html
[wasabi2]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020032.html
[news102 wabisabi]: https://bitcoinops.org/en/newsletters/2020/06/17/#wabisabi-coordinated-coinjoins-with-arbitrary-output-values
[hillebrand wormhole]: https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020186.html
[leguilly dlcmsg]: https://mailmanlists.org/pipermail/dlc-dev/2022-March/000135.html
[news193 major update]: https://bitcoinops.org/en/newsletters/2022/03/30/#major-update
[lnurl]: https://github.com/fiatjaf/lnurl-rfc
[huffman coding]: https://en.wikipedia.org/wiki/Huffman_coding
[news60 buried]: https://bitcoinops.org/en/newsletters/2019/08/21/#hardcoded-previous-soft-fork-activation-blocks
[news175 cjdns]: https://bitcoinops.org/en/newsletters/2021/11/17/#bitcoin-core-23077
[cjdns.md]: https://github.com/bitcoin/bitcoin/blob/6a02355ae9/doc/cjdns.md
[BIP47]: https://github.com/bitcoin/bips/blob/master/bip-0047.mediawiki
[BOLTs #868]: https://github.com/lightning/bolts/issues/868
[Bitcoin Core #24118]: https://github.com/bitcoin/bitcoin/issues/24118
[Bitcoin Core #23536]: https://github.com/bitcoin/bitcoin/issues/23536
[Bitcoin Core #24555]:https://github.com/bitcoin/bitcoin/pull/24555
[Bitcoin Core #24710]: https://github.com/bitcoin/bitcoin/issues/24710
[C-Lightning #5013]: https://github.com/ElementsProject/lightning/pull/5013
[C-Lightning #5121]: https://github.com/ElementsProject/lightning/issues/5121
[Eclair #2196]: https://github.com/ACINQ/eclair/issues/2196
[LND #6263]: https://github.com/lightningnetwork/lnd/issues/6263
[Libsecp256k1 #1089]: https://github.com/bitcoin-core/secp256k1/issues/1089
[Rust Bitcoin #909]: https://github.com/rust-bitcoin/rust-bitcoin/issues/909
[LDK #1388]: https://github.com/lightningdevkit/rust-lightning/pull/1388
[BOLT11]: https://github.com/lightning/bolts/blob/master/11-payment-encoding.md

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
