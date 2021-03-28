---
title: 'Bitcoin Optech Newsletter #140'
permalink: /zh/newsletters/2021/03/17/
name: 2021-03-17-newsletter-zh 
slug: 2021-03-17-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---



本周的新闻简报描述了有关追回丢失的LN Funding Transaction（闪电网络存款交易）的讨论,还包括我们的常规部分，其中包含新版本和候选版本以及对常见比特币基础设施软件的重要更新。

# 新闻

- 追回丢失的闪电网络存款交易：在存在交易延展性问题的情况下，闪电网络*Funding Transaction*（存款交易）并不安全。Segwit（隔离见证）的采用消除了大多数交易可能碰到的第三方延展性问题， 但是它并未解决交易创建者自己更改其txid的情况，例如通过使用[费用替代（RBF）](https://bitcoinops.org/en/topics/replace-by-fee/)来[碰撞（fee bumping）](https://bitcoinops.org/en/scaling/fee-bumping/) 存款交易的手续费。如果发生txid突变，则预签名的退款交易(refund transaction)无效，因此用户无法取回其资金。此外，远程节点可能不会自动看到存款交易，因此可能无法帮助存款者取回他们的钱。

- 本周，Rusty Russell在Lightning-Dev邮件列表中[发布](](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-March/002981.html))了他在C-Lightning中实现的一个快速且实验性的功能，该功能可以帮助用户解决这个问题，追回他们的资金。他还描述了有关问题的替代方案，以及[拟议的](https://github.com/lightningnetwork/lightning-rfc/issues/851)*双出资通道协议（dual-funding)*对该问题的影响。Christian Decker还 [发布](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-March/002982.html)了对LN规范的[拟议更改](https://github.com/lightningnetwork/lightning-rfc/issues/854)，以帮助促进资金追回的工作。随着LN软件增加了对来自外部钱包的资金渠道的支持（例如，[Newsletter #51](https://bitcoinops.org/en/newsletters/2019/06/19/#c-lightning-2672)所述的C-Lightning和 [Newsletter #92](https://bitcoinops.org/en/newsletters/2020/04/08/#lnd-4079)中的LND），开发人员可能希望给予这种类型的失败方案更多的关注。

  


## 新版本和候选版本发布

新版本和通用的比特币基础设施项目的候选版本。请考虑升级到新版本或帮助测试候选版本。

- [HWI 2.0.0](https://github.com/bitcoin-core/HWI/releases/tag/2.0.0)是HWI的下一个主要版本的发行版。除其他改进外，它还包含对BitBox02上的multisig的支持，改进的文档以及对使用Trezor支付`OP_RETURN`输出的支持。

- [Rust-Lightning 0.0.13](https://github.com/rust-bitcoin/rust-lightning/releases/tag/v0.0.13)是这个LN库的最新版本，包含了对[多路径支付](](https://bitcoinops.org/en/topics/multipath-payments/))和未来脚本升级(如[taproot](](https://bitcoinops.org/en/topics/taproot/)))的前向兼容性的改进。

- [BTCPay Server 1.0.7.0](https://github.com/btcpayserver/btcpayserver/releases/tag/v1.0.7.0)是这个自托管支付处理软件的最新版本。值得注意的改进包括功能更丰富、外观上更吸引人的钱包设置向导，导入使用Specter创建的钱包的能力，以及[bech32地址](https://bitcoinops.org/en/topics/bech32/)更有效的二维码。

  

## 重要代码和文档更新

*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[硬件钱包接口 (HWI)](https://github.com/bitcoin-core/HWI)、[Rust 比特币](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay 服务器](https://github.com/btcpayserver/btcpayserver/)、[比特币改进提案 (BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 的重要更新。*

- [Bitcoin Core＃21007](https://github.com/bitcoin/bitcoin/issues/21007)添加了新的`-daemonwait`配置选项。从早期版本开始，可以通过使用`-daemon`配置选项启动程序，将Bitcoin Core作为后台守护进程运行。该`-daemon`选项使程序立即在后台启动守护进程。新`-daemonwait`选项与此类似，但是仅在初始化完成后才将守护进程放在后台。这使用户或父进程可以通过观察程序的输出或退出的代码来更轻松地了解守护程序是否成功启动。
- [C-Lightning＃4404](https://github.com/ElementsProject/lightning/issues/4404)允许`keysend`RPC（请参阅[Newsletter #107](https://bitcoinops.org/en/newsletters/2020/07/22/#c-lightning-3792)）将消息发送到未明确表示其支持该功能的节点。如所 [讨论的](https://github.com/ElementsProject/lightning/issues/4299#issuecomment-781606865)，信号从未标准化，而LND实现的过程也不依赖于信号，因此这个更改应该允许C-Lightning发送到LND可以定位的大致相同的节点集。
- [C-Lightning＃4410](https://github.com/ElementsProject/lightning/issues/4410)使双出资通道的实验性实施与最新的[规范草案](https://github.com/lightningnetwork/lightning-rfc/issues/851)更改保持一致。最值得注意的是，离散对数等价证明（PODLE）的使用被放弃了，至少是暂时性的（有关PODLE的原始讨论，请参阅 [Newsletter #83](https://bitcoinops.org/en/newsletters/2020/02/05/#interactive-construction-of-ln-funding-transactions) ，有关替代方案的讨论，请参阅[Newsletter #131](https://bitcoinops.org/en/newsletters/2021/01/13/#ln-dual-funding-anti-utxo-probing)）。在这次合并之后，一个[新的PR](https://github.com/ElementsProject/lightning/issues/4427)被打开了，通过消除编译带有特殊构建标志的C-Lightning的需要(尽管仍然需要一个特殊的配置选项)，双出资通道的实验将变得更加容易。
- [LND＃5083](https://github.com/lightningnetwork/lnd/issues/5083)允许从文件中读取[PSBT](https://bitcoinops.org/en/topics/psbt/)，而不是通过读取标准输入（stdin）文件描述符来进行。某些终端对可以同时添加（即粘贴）到stdin的字符数有[限制](https://github.com/lightningnetwork/lnd/issues/5080)，这使得超过4096个base64字符（相当于3.072字节的二进制数）的PSBT无法使用。尤其是现在，几个硬件钱包要求PSBT包括以前的隔离见证交易（请参阅[Newsletter #101](https://bitcoinops.org/en/newsletters/2020/06/10/#fee-overpayment-attack-on-multi-input-segwit-transactions)），创建大小超过3 KiB的PSBT是很常见的。
- [LND＃5033](https://github.com/lightningnetwork/lnd/issues/5033)添加了一个`updatechanstatus`RPC，该RPC可以通告一个信道已被禁用（类似于您的节点脱机）或已重新启用（类似于您的节点重新联机）。
- [Rust-Lightning＃826](https://github.com/rust-bitcoin/rust-lightning/issues/826)将输出的最大允许`OP_CHECKSEQUENCEVERIFY` 延迟增加到2,016个块，以支付单方面关闭通道的节点的费用。这解决了使用LND打开通道时的互操作性问题，该问题可能会要求延迟到2016年的数据块，这比以前的Rust-Lightning最大1008数据块更大。
- [HWI＃488](https://github.com/bitcoin-core/HWI/issues/488)在`displayaddress`命令处理在与[输出脚本描述符](https://bitcoinops.org/en/topics/output-script-descriptors/)`--desc`选项一起使用时的多签地址的方式上实现了重大更改。之前HWI根据涉及的硬件设备使用了[BIP67](https://github.com/bitcoin/bips/blob/master/bip-0067.mediawiki)词典密钥自动排序（例如，将BIP67用于Coldcard硬件钱包，而不将其应用于Trezor钱包）。当用户显式指定实现BIP67键排序的`sortedmulti`描述符选项时，这种实现方式会产生问题。更改之后，描述符的用户需要为需要字典式键排序的设备指定`sortedmulti`，或为不需要字典式键排序的设备指定`multi`。





