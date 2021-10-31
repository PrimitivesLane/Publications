---
title: 'Bitcoin Optech Newsletter #167'
permalink: /zh/newsletters/2021/09/22/
name: 2021-09-22-newsletter-zh 
slug: 2021-09-22-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---

本周的 Newsletter 介绍了对 BIP 流程的修改建议，总结了一个在比特币核心中增加对包中继支持的计划，另外也涉及到关于在 DNS 中增加 LN 节点信息的讨论。还包括我们的常规部分，服务和客户端软件的变更，如何为 taproot 做准备，新版本和候选版本，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻
- **BIP 扩展**：Karl-Johan Alm 在 Bitcoin-Dev 邮件列表中[发布](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-September/019457.html)了一个提案，即已经达到一定稳定程度的 BIP 不再可以修改，除非是小修改。对稳定的 BIP 条款的修改需要通过新的 BIP 实现，作为对之前文档的扩展。

  Anthony Towns [反对](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-September/019462.html)这个想法，并建议对目前的程序进行一些替代性调整，包括在 BIP 仓库中设立一个草稿文件夹，并取消 BIP 维护者选择提案编号的权力。

- **包的内存池接受和包的 RBF**：Gloria Zhao 在 Bitcoin-Dev 邮件列表中[发布](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-September/019464.html)了一个关于多个关联交易的[包中继](https://bitcoinops.org/en/topics/package-relay/)的设计，这将提高 [CPFP](https://bitcoinops.org/en/topics/cpfp/) 和 [RBF](https://bitcoinops.org/en/topics/replace-by-fee/) 费用更新的灵活性和可靠性。一个[初步的实现](https://github.com/bitcoin/bitcoin/pull/22290)只允许包通过比特币核心的 RPC 接口提交，但最终目标是使该功能在 P2P 网络上可用。Zhao 简明扼要地总结了她对比特币核心交易接受规则的修改建议。

  >- 包可以包含已经在内存池中的交易。
  >- 包内有两代交易，多个父交易和一个子交易。
  >- 用包的费率做费用相关的检查。这意味着，钱包可以创建一个使用 CPFP 的包。
  >- 父代可以使用 RBF 内存池交易，用一套类似于 [BIP125](https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki) 的规则。这实现了 CPFP 和 RBF 的结合，交易的子代费用用于支付替换内存池冲突的开销。

  该邮件希望收到那些期望使用这些功能或认为他们可能受到这些变化影响的开发者对该提案的反馈。

- **LN 节点的 DNS 记录**：Andy Schroder 在 Lightning-Dev 邮件列表中[发布](https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-September/003224.html)了一个提案，即规范使用一套 DNS 记录，以将域名解析为 LN 节点的 IP 地址和公钥。在写这篇文章的时候，这个想法还在讨论中。

## 服务和客户端软件的变化
*在这个月的专题中，我们会重点介绍比特币钱包和服务中有意思的更新。*

- **闪电网络地址标识符公布**：André Neves [宣布](https://twitter.com/andreneves/status/1425651740502892550)了[闪电网络地址](https://lightningaddress.com/)协议，该协议将 [LNURL-pay](https://github.com/fiatjaf/lnurl-rfc/blob/master/lnurl-pay.md) 流包装成用户熟悉的[类电子邮件地址](https://github.com/andrerfneves/lightning-address/blob/master/README.md#tldr)。

- **ZEBEDEE 发布 LN 钱包浏览器插件**：ZEBEDEE [宣布](https://blog.zebedee.io/browser-extension/)了 Chrome 和 Firefox 插件，这些插件跟它[专注游戏的钱包](https://zebedee.io/wallet)做了集成。

- **Specter v1.6.0 支持 single-key taproot**：Specter 的 [v1.6.0](https://github.com/cryptoadvance/specter-desktop/releases/tag/v1.6.0) 版本包括对 regtest 和 [signet](https://bitcoinops.org/en/topics/signet/) taproot 地址的支持。

- **Impervious 发布 LN P2P 数据 API**：建立在 LND 上的 [Impervious](https://www.impervious.ai/) 框架允许开发者通过闪电网络[建立](https://docs.impervious.ai/) P2P 数据流应用。

- **Full Noded v0.2.26 发布**：[Full Noded](https://fullynoded.app/) 是一个用于 macOS/iOS 的比特币和闪电网络钱包，它增加了对 [taproot](https://bitcoinops.org/en/topics/taproot/)、[BIP86](https://github.com/bitcoin/bips/blob/master/bip-0086.mediawiki) 和 signet 的支持。

## 为 taproot 做准备 #14：在 signet 上测试
*关于开发者和服务提供者如何为即将在区块高度 709,632 处激活的 taproot 做准备的每周[系列](https://bitcoinops.org/en/preparing-for-taproot/)文章。*

虽然在主网 [709,632 区块之前你不能安全地使用 taproot](https://bitcoinops.org/en/preparing-for-taproot/#why-are-we-waiting)，但现在你可以在 testnet 或 signet 上使用 taproot。与使用比特币核心的 regtest 模式创建一个本地测试网络相比，如 [Optech taproot 工作手册](https://bitcoinops.org/en/preparing-for-taproot/#learn-taproot-by-using-it)介绍的，使用 testnet 或 signet 可以更容易地测试你的钱包与其他人的钱包的交互。

在这篇文章中，我们将使用比特币核心的内置钱包在 [signet](https://bitcoinops.org/en/topics/signet/) 上接收和花费一笔 taproot 交易。你应该能够基于这些说明，在你自己的钱包和比特币核心之间测试接收和花费。

虽然在技术上可以使用 Bitcoin Core 22.0 的内置钱包接收和花费 taproot 交易，但我们建议你使用 Bitcoin Core pull request [#22364](https://github.com/bitcoin/bitcoin/issues/22364)，它把 taproot 做为[描述符](https://bitcoinops.org/en/topics/output-script-descriptors/)钱包的默认选项。构建完成后，启动 signet：

```
$ bitcoind -signet -daemon
```

如果这是你第一次使用 signet，你将需要同步区块链。目前同步的内容包括不到 200 MB 的数据，可以在短短一分钟内完成同步。你可以使用 `getblockchaininfo` RPC 来监控同步的进度。同步后，创建一个描述符钱包：

```
$ bitcoin-cli -signet -named createwallet wallet_name=p4tr descriptors=true load_on_startup=true
{
  "name": "p4tr",
  "warning": "Wallet is an experimental descriptor wallet"
}
```

现在你可以创建一个 bech32m 地址：

```
$ bitcoin-cli -named -signet getnewaddress address_type=bech32m
tb1p6h5fuzmnvpdthf5shf0qqjzwy7wsqc5rhmgq2ks9xrak4ry6mtrscsqvzp
```

有了这个地址，你就可以在 [signet 水龙头](https://signetfaucet.com/) 上申领测试币。然后你需要等待确认，会需要与在主网上差不多的时间（通常最多 30 分钟，但有时更长）。如果你查看这个交易，你会注意到你创建的 P2TR 脚本。

```
$ bitcoin-cli -signet getrawtransaction 688f8c792a7b3d9cb46b95bfa5b10fe458617b758fe4100c5a1b9536bedae4cd true | jq .vout[0]
{
  "value": 0.001,
  "n": 0,
  "scriptPubKey": {
    "asm": "1 d5e89e0b73605abba690ba5e00484e279d006283bed0055a0530fb6a8c9adac7",
    "hex": "5120d5e89e0b73605abba690ba5e00484e279d006283bed0055a0530fb6a8c9adac7",
    "address": "tb1p6h5fuzmnvpdthf5shf0qqjzwy7wsqc5rhmgq2ks9xrak4ry6mtrscsqvzp",
    "type": "witness_v1_taproot"
  }
}
```

然后，你可以创建第二个 bech32m 地址，并将币发送过去，来测试花费。

```
$ bitcoin-cli -named -signet getnewaddress address_type=bech32m
tb1p53qvqxja52ge4a7dlcng6qsqggdd85fydxs4f5s3s4ndd2yrn6ns0r6uhx
$ bitcoin-cli -named -signet sendtoaddress address=tb1p53qvqxja52ge4a7dlcng6qsqggdd85fydxs4f5s3s4ndd2yrn6ns0r6uhx amount=0.00099
24083fdac05edc9dbe0bb836272601c8893e705a2b046f97193550a30d880a0c
```

对于这个花费，我们可以看一下其中的一个输入，它的见证只包含一个 64 字节的签名。这比一个 P2WPKH 花费或任何其他之前类型的比特币花费所需要的见证字节数要[小](https://bitcoinops.org/en/preparing-for-taproot/#is-taproot-even-worth-it-for-single-sig)。

```
$ bitcoin-cli -signet getrawtransaction 24083fdac05edc9dbe0bb836272601c8893e705a2b046f97193550a30d880a0c true | jq .vin[0]
{
  "txid": "bd6dbd2271a95bce8a806288a751a33fc4cf2c336e52a5b98a5ded432229b6f8",
  "vout": 0,
  "scriptSig": {
    "asm": "",
    "hex": ""
  },
  "txinwitness": [
    "2a926abbc29fba46e0ba9bca45e1e747486dec748df1e07ee8d887e2532eb48e0b0bff511005eeccfe770c0c1bf880d0d06cb42861212832c5f01f7e6c40c3ce"
  ],
  "sequence": 4294967294
}

```

通过试用上述命令，你会发现基于任何支持 signet 的钱包使用 taproot 来接收和花费资金都很容易。

## 发布和候选发布
*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- [Bitcoin Core 0.21.2rc2](https://bitcoincore.org/bin/bitcoin-core-0.21.2/) 是比特币核心维护版本的一个候选发布版本，包含了若干代码中的缺陷修复以及小规模的实现优化。

## 重大代码和文档更新
*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[Hardware Wallet Interface(HWI)](https://github.com/bitcoin-core/HWI)、[Rust Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay Server](https://bitcoinops.org/en/newsletters/2021/08/11/)、[Bitcoin Improvement Proposals(BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 中值得注意的变更。*

- [Eclair #1932](https://github.com/ACINQ/eclair/issues/1932) 实现了 [BOLTs #824](https://github.com/lightningnetwork/lightning-rfc/issues/824) 中定义的修改后的[锚定输出](https://bitcoinops.org/en/topics/anchor-outputs/)协议，所有预签的 HTLC 支出都使用零费用，因此没有费用会被盗。详见 [Newsletter #165](https://bitcoinops.org/en/newsletters/2021/09/08/#bolts-824)。

- [LND #5405](https://github.com/lightningnetwork/lnd/issues/5405) 扩展了 `updatechanpolicy` RPC，它可以返回任何由于当前策略（或由于其他问题，如通道供资交易仍未确认）而无法使用的通道。

- [LND #5304](https://github.com/lightningnetwork/lnd/issues/5304) 使 LND 能够创建和验证 LND 未知的具有外部权限的 macaroon<sup>[[1]](#myfootnote1)</sup>。这一变化使得 [Lightning Terminal](https://bitcoinops.org/en/newsletters/2020/08/19/#lightning-labs-releases-lightning-terminal) 等工具能够使用单一的 macaroon 在多个守护进程中进行验证，这些守护进程都与同一个 LND 连接。

- [Rust Bitcoin #628](https://github.com/rust-bitcoin/rust-bitcoin/issues/628) 增加了对 Pay to Taproot's sighash 结构的支持，并整理了对传统、segwit 和 taproot 输入的 sighash 缓存的存储。

- [Rust Bitcoin #580](https://github.com/rust-bitcoin/rust-bitcoin/pull/580) 增加了对 [BIP37](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki) 规范中定义的 P2P 网络消息的支持，用于交易的布隆过滤器。

- [Rust Bitcoin #626](https://github.com/rust-bitcoin/rust-bitcoin/pull/626) 增加了获取区块的剥离大小（去除所有 segwit 数据的区块）和交易的 vbyte 大小的函数。

- [Rust-Lightning #1034](https://github.com/rust-bitcoin/rust-lightning/issues/1034) 增加了一个函数，可用于检索通道余额的完整列表，包括目前正在关闭的通道。这使得终端用户软件可以显示一致的用户余额，即使有些资金仍需等待确认才能再次使用。

<a name="myfootnote1">[1]</a>译者注：关于 macaroon，可以参考 [ION Lightning Network Wiki 及其中的参考文献](https://wiki.ion.radar.tech/tech/research/macaroons)。
