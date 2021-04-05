---
title: 'Bitcoin Optech Newsletter #141'
permalink: /zh/newsletters/2021/03/24/
name: 2021-03-24-newsletter-zh 
slug: 2021-03-24-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---



本周的新闻简报描述了根据比特币现有共识规则进行委托签名的技术，总结了关于taproot对比特币抵抗量子密码学的影响的讨论，并宣布了一系列每周会议来帮助激活taproot，还包括我们的常规章节，其中包含服务和客户端软件的重要更新，新版本和候选版本以及对常见比特币基础设施软件的重要更新。

# 新闻

- **根据现有的共识规则签署委托：** 想象 Alice想让Bob有能力使用她的一个UTXO，而无需立即创建链上交易或给Bob她的私钥。这就是已经被讨论了很多年的所谓的*委托*，近期最值得注意的是[graftroot提案的](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-February/015700.html)一部分。上周，Jeremy Rubin在Bitcoin-Dev邮件列表中[发表](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018615.html)了一篇文章，描述了一种使用当前的比特币协议实现委托的技术。

  假设Alice`UTXO_A`和Bob有`UTXO_B`。Alice创建了一个部分签署的事务，同时花费`UTXO_A`和`UTXO_B`。Alice使用Sighash标志[SIGHASH_NONE](https://btcinformation.org/en/developer-guide#term-sighash-none)为她的UTXO签名，这可以防止签名提交至事务的任一输出。这使事务的另一输入的所有者Bob可以对输出选择进行单方面控制，使用他的签名和普通`SIGHASH_ALL`标志来提交那些输出，并防止其他任何人修改事务。通过使用这种双重输入`SIGHASH_NONE`技巧，Alice将Bob的签名权委托给了Bob。

  该技术似乎主要具有理论意义。还有其他拟议的委托技术，包括graftroot， [ OP_CHECKTEMPLATEVERIFY](https://bitcoinops.org/en/topics/op_checktemplateverify/)和 [OP_CHECKSIGFROMSTACK](https://bitcoinops.org/en/topics/op_checksigfromstack/) -其余技术可能在多个方面都更优越，但目前只有该技术在主网上可用，任何人都可以尝试使用它。

- **关于对Taproot进行量子计算攻击的讨论：** 原始的比特币软件提供了两种接收比特币的方式：

  - Pay-to-Public-Key  （P2PK）实施了[原始比特币论文](https://bitcoin.org/bitcoin.pdf)中描述的简单明了的方法，即使用公钥接收[比特币](https://bitcoin.org/bitcoin.pdf)，并允许通过签名来使用这些比特币。当公钥材料可以完全由软件处理时，比特币软件默认使用这种方式。
  - Pay-to-Public-Key-Hash（P2PKH）添加了一个间接层，使用哈希摘要接收比特币，该哈希摘要是待使用的公钥的承诺。要花费这些比特币时，公钥需要与签名一起发布，这使得用于哈希摘要的20个字节成为额外的开销。当付款信息可能需要由人来处理时（例如可以复制和粘贴的地址），默认情况下使用此方式。

  中本聪从来没有描述过为什么要实现两种方法，但是人们普遍认为，他添加了哈希间接方法是为了使比特币地址更小，从而可以更轻松地进行通信。原始比特币实现中的公钥为65字节，但地址哈希仅为20字节。

  ​	此后的十年中，出现了许多发展。为了使某些多重签名协议[在默认情况下简单并且安全](https://bitcoinops.org/en/bech32-sending-support/#address-security)，已确定多方协议的脚本必须使用32字节的承诺。比特币开发人员还了解到此前已知的技术，可以将公钥压缩为33个字节---稍后介绍如何将其[优化](https://bitcoinops.org/en/newsletters/2019/05/29/#move-the-oddness-byte)为32个字节。最后，[taproot的主要创新](https://bitcoinops.org/en/newsletters/2018/12/28/#taproot)表明，将一个32字节的公钥提交给脚本可以同时保证[安全性](https://bitcoinops.org/en/newsletters/2020/03/04/#taproot-in-the-generic-group-model)等同于32字节的哈希。这意味着无论人们使用公钥哈希还是公钥本身都不会改变与地址通信的数据量--如果他们想要一个普遍适用的地址格式，则无论哪种方式都是32字节。但是，直接使用公钥仍然避免了哈希间接导致的额外带宽和存储。如果每次付款都使用公钥而不是32字节的公钥哈希，那么每年将节省约13 GB的区块链空间。 该[BIP341](https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki) taproot的规范将空间节省的原因归结为它使用P2PK风格的公钥支付，而不是P2PKH风格的哈希。

  ​	但是P2PKH哈希的间接性确实具有一个优点：它可以在公开领域中隐藏公钥，直到人们需要授权花费为止。这意味着有能力破坏公钥安全性的敌手需要要等到事务被广播才能开始尝试攻击,而且一旦事务达到一定数量的确认数，他们就无法窃取由这个密钥控制的资金。这限制了它们攻击的时间，也就意味着花费太长时间的攻击将无法奏效。尽管置于taproot选择直接使用P2PK的公钥格式的语境下，此前已详细讨论过这个问题（见[1](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015620.html)，[2](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-December/016556.html)，和newsletters [＃70](https://bitcoinops.org/en/newsletters/2019/10/30/#why-does-hashing-public-keys-not-actually-provide-any-quantum-resistance)和[＃86](https://bitcoinops.org/en/newsletters/2020/02/26/#could-taproot-create-larger-security-risks-or-hinder-future-protocol-adjustments-re-quantum-threats)），但在一封反对taproot的邮件发布后，由于担心我们可能会在 "本世纪末 "看到一个强大到足以攻击比特币风格公钥的量子计算机，本周比特币开发者邮件列表[重新讨论](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018641.html)了这个话题。

  ​	在邮件列表讨论中，没有一个参与者说他们反对taproot，但他们确实检查了论证的前提，讨论了备选方案，并评估了这些备选方案所背后的权衡。这些对话的摘录如下:

  - 哈希法目前尚不能很好地抵抗QC(量子计算）： 在[2019年的一项调查中](https://twitter.com/pwuille/status/1108097835365339136)，拥有强大的QC(量子计算）能力的攻击者在仅拥有一份比特币区块链副本的情况下可以窃取1/3总量的比特币。多数是由于用户[重用地址](https://bitcoinops.org/en/topics/output-linking/)而造成的，这是一种不受鼓励的做法，但似乎不太可能很快消失。

    此外，与会人员指出，与第三方共享其各自的公钥或[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)扩展公钥（xpubs）的任何人，如果其公共密钥泄漏，也将面临巨大的量子算法攻击的风险，这可能包括存储在硬件钱包或LN付款通道中的大多数比特币。简而言之，即使今天我们几乎普遍使用P2PKH样式的哈希公钥，拥有强大量子计算能力的攻击者也有可能通过访问公共或第三方数据来盗取几乎所有比特币。这意味着选择将P2PK样式的非哈希公钥与taproot一起使用不会显著改变比特币当前的安全模型。

  - 无链上成本的针对后量子恢复的Taproot提升： 如果比特币使用者了解到即将出现强大的量子计算机，他们可以拒绝任何Taproot密钥形式的花费，即仅使用单个签名的支出类型。但是，如果用户在创建自己的 taproot 地址时提前做好准备，也可以使用脚本路径花费支付比特币。在这种情况下，taproot地址的承诺将是用户想要使用的[tapscript](https://bitcoinops.org/en/topics/tapscript/)的哈希值 。该哈希承诺（ hash commitment ）可以作为[方案的](https://gist.github.com/harding/bfd094ab488fd3932df59452e5ec753f)一部分，过渡到可以抵御量子计算攻击的新密码算法，或者——如果这种算法在量子计算成为威胁之前已针对比特币进行了标准化，则可以允许比特币的持有者立即过渡到新方案。这仅在以下情况下有效：单个用户创建备份的tapscript花费路径，且没有将这些备份路径所涉及的任何公钥（包括BIP32 xpubs）共享给他人，且人们在量子计算对比特币造成巨大损害之前就察觉到了。

  - **攻击是否现实？** 一位受访者认为，到本世纪末，快速的量子计算[是](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018658.html)“对预计进度太过乐观”。另一位受访者 [指出](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018649.html)，要在极短的时间内将单个慢速量子计算的转变为可以并行工作的量子计算系统，并攻破使用P2PKH样式的哈希的间接保护，是“相当简单的工程挑战”。第三位受访者 [建议](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018648.html)构建特殊的比特币地址，只有在快速量子计算方面取得进展的人才能使用该地址。用户可以自愿将比特币捐赠给这些地址，以创建有激励的预警系统。

  - **在激活taproot之后可以添加新的哈希样式的地址：** 如果有相当多的用户真的认为突然出现强大的量子计算机会构成威胁，则可以使用后续的软分叉增加一种使用哈希的p2pkh风格的taproot地址类型。但是，这引起了一些受访者的反对：

    - 会造成混乱
    - 将使用更多的块空间
    - 在使用诸如[环签名成员证明](https://twitter.com/n1ckler/status/1334240709814136833)或[Provisions](https://eprint.iacr.org/2015/1008)之类的协议时，[减小](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018667.html)了taproot匿名集的大小

  - **带宽/存储成本与CPU成本：** 可以通过从签名及其签名的事务数据中推导公钥，[可能](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018643.html)消除哈希方式带来的额外32字节存储开销，这种技术称为密钥恢复。同样，这遭到了反对。密钥恢复需要[占有大量的CPU资源](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018644.html)，这会减慢节点速度，同时也阻止了schnorr批处理验证的使用，而schnorr批处理验证可以使历史块验证速度提高三倍。这也[使](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018667.html)匿名成员身份证明和相关技术既难以开发，也占用大量CPU资源。还[可能](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-March/018646.html)存在[专利](https://patents.google.com/patent/US7215773B1/en)问题。

    截至本文撰写之时，邮件列表的讨论似乎已经结束，社区并未显著减少对taroot的支持。随着研究人员和企业不断提高量子计算的技术水平，我们期望看到未来关于如何最好地维护比特币安全的讨论。

- **每周taproot激活会议：** [＃taprootroot-activation](https://webchat.freenode.net/##taproot-activation) IRC频道计划举办十次周会，时间在每个星期二UTC 19:00 ，讨论与激活[taproot](https://bitcoinops.org/en/topics/taproot/)有关的细节，第一次[会议](http://gnusha.org/taproot-activation/2021-03-23.log)于昨天（3月23日）举行。

  

## 服务和客户端软件的变更

**在本月度功能中，我们重点介绍了比特币钱包和服务的有趣更新。**

- *OKCoin发布闪电存款和取款：* 一个[博客文章](https://blog.okcoin.com/2021/03/04/how-to-use-bitcoin-lightning-network/)概要OKCoin的闪电存款和取款的支持。结果，他们还将最低存款/取款限制从0.001 BTC降低到0.000001 BTC。而当前使用LN进行交易时，OKCoin的限制为0.05 BTC。
- *BitMEX宣布支持bech32：* 在[博客中](https://blog.bitmex.com/introducing-bech32-deposits-on-bitmex-to-deepen-bitcoin-integration-lower-fees/)，BitMEX详细介绍了[bech32](https://bitcoinops.org/en/topics/bech32/)存款支持的启动计划 。BitMEX之前[已经](https://bitcoinops.org/en/newsletters/2019/12/18/#bitmex-bech32-send-support)推出了bech32取款（发送）的功能支持。
- *Spectre v1.2.0发布：* [Spectre v1.2.0](https://github.com/cryptoadvance/specter-desktop/releases/tag/v1.2.0)包括对比特币核心[描述符钱包](https://bitcoinops.org/en/topics/output-script-descriptors/)和货币控制功能的支持。
- **Breez流式传输音频以进行Lightning付款：** Breez钱包[集成了音频播放器](https://medium.com/breez-technology/podcasts-on-breez-streaming-sats-for-streaming-ideas-d9361ae8a627)，可以和[keyend](https://bitcoinops.org/en/topics/spontaneous-payments/)结合使用 ，使用户可以在收听播客的同时将支付流式传输到发布者并一次性发送小费。
- **密钥管理器Dux Reserve宣布：** ThibaudMaréchal[宣布了](https://twitter.com/thibm_/status/1369331407441510405)Dux Reserve，这是一个Beta开源桌面密钥管理器，同时支持MacOS，Windows和Linux系统，并支持Ledger，Coldcard和Trezor硬件钱包。
- **Coldcard现在使用libsecp256k1：** Coldcard的[4.0.0版本](https://blog.coinkite.com/version-4.0.0-released/)以及其他功能，切换到使用Bitcoin Core的[libsecp256k1](https://github.com/bitcoin-core/secp256k1)库进行加密操作。

## 新版本和候选版本发布

新版本和通用的比特币基础设施项目的候选版本。请考虑升级到新版本或帮助测试候选版本。

- [C-Lightning 0.10.0-rc1](https://github.com/ElementsProject/lightning/releases/tag/v0.10.0rc1)是此LN节点软件的下一个主要版本的候选版本。

## 重要代码和文档更新

*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[硬件钱包接口 (HWI)](https://github.com/bitcoin-core/HWI)、[Rust 比特币](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay 服务器](https://github.com/btcpayserver/btcpayserver/)、[比特币改进提案 (BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 的重要更新。*

- [Bitcoin Core＃20861](https://github.com/bitcoin/bitcoin/issues/20861)实现了对[BIP350](https://github.com/bitcoin/bips/blob/master/bip-0350.mediawiki)（v1+ 见证地址（witness address）的Bech32m格式）的支持。Bech32m取代bech32（[BIP173](https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki)）作为版本1-16的本地隔离见证（segwit）输出的地址格式。本地隔离见证（segwit）version 0的输出（P2WPKH和P2WSH）将继续使用bech32。一旦在网络上定义了[taproot](https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki)输出（[BIP341](https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki)），此PR将确保Bitcoin Core用户能够将付款发送到Pay to Taproot（P2TR）地址。这个改变应该不会影响到任何主网系统，但是可能会在例如signet这样的测试环境中引起不兼容的问题，在这些测试环境中taproot已经使用bech32编码的地址激活了。bech32m 的支持也将被反向移植到比特币核心0.19、0.20和0.21。
- [Bitcoin Core＃21141](https://github.com/bitcoin/bitcoin/issues/21141)更新`-walletnotify`配置设置，每次看到影响到已加载钱包的事务时，该配置设置都会调用用户指定的命令。将两个新的占位符添加到可传递给命令的参数中，`%b`用于表示包含事务的块的哈希值和`%h`用于表示块的高度。两者均设置为未确认事务的定义值。
- [C-Lightning＃4428](https://github.com/ElementsProject/lightning/issues/4428)弃用`fundchannel_complete`RPC对txid的支持，改为要求传递[PSBT](https://bitcoinops.org/en/topics/psbt/)。该PSBT可以被检查，以确保它包含了资金的输出，消除了用户因传递错误数据而可能失去收回资金的能力的[问题](https://github.com/ElementsProject/lightning/issues/4416)。
- [C-Lightning＃4421](https://github.com/ElementsProject/lightning/issues/4421)实现了[上周新闻通讯中](https://bitcoinops.org/en/newsletters/2021/03/17/#rescuing-lost-ln-funding-transactions)讨论过的入资事务恢复程序。通过第一方延展事务（例如使用RBF）错误地为通道提供资金但是没有使用通道的用户现在可以将其交事务输出提供给 `lightning-close`命令，以与支持`shutdown_wrong_funding`功能的对等方协商恢复 。
- [LND＃5068](https://github.com/lightningnetwork/lnd/issues/5068)提供了许多新的配置选项，用于限制LND处理网络gossip信息的数量。这在资源有限的系统上可以提供帮助。
- [Libsecp256k1＃831](https://github.com/bitcoin-core/secp256k1/issues/831)实现了可以将签名验证速度提高15％的算法。它还可以将生成签名所需的时间减少25％，同时仍使用恒定时间算法，以最大程度地提高对侧信道攻击的抵御。此外，它还消除了Libsecp256k1对其他库的某些依赖关系。有关此优化的更多信息，请参见[Newsletter #136](https://bitcoinops.org/en/newsletters/2021/02/17/#faster-signature-operations)。
- [BIP＃1059](https://github.com/bitcoin/bips/issues/1059)添加了[BIP370](https://github.com/bitcoin/bips/blob/master/bip-0370.mediawiki)，该版本指定了先前在邮件列表中所讨论的version 2 PSBT（请参阅 [Newsletter #128](https://bitcoinops.org/en/newsletters/2020/12/16/#new-psbt-version-proposed)）。





