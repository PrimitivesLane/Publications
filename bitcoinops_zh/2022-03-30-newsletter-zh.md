---
title: 'Bitcoin Optech Newsletter #193'
permalink: /zh/newsletters/2022/03/30/
name: 2022-03-30-newsletter-zh
slug: 2022-03-30-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---

本周的周报介绍了一个让 Bitcoin Core 允许替换交易池中交易的见证数据（而不改变其 txid）的提议，以及关于升级闪电网络的 gossip 协议的持续讨论。还包括了我们的常规部分：来自 Bitcoin Stack Exchange 的精选问题和答案、软件的新版本和候选版本、热门比特币基础设施项目的重大变更。

## 新闻

- **交易见证数据的替换**：Larry Ruane  在 Bitcoin-Dev 邮件组中[询问][asked]对一项提议的意见：允许一笔交易以更小的见证数据（因此将使用不同的 wtxid）来替换原有的见证数据、且不改变交易的标识符（txid）。Ruane 想知道有没有什么应用现在会创建见证数据的大小可能改变（例如，从一个 [taproot][taproot] 脚本路径花费转变成密钥路径花费）而其它交易细节（例如输出地址和数额）都不变的交易。

  如果现有或者被构想的应用可以从能够替换见证数据的能力中获益，Ruane 也希望探讨：应该容许节约了多少见证数据的交易使用替换？对节约程度的要求越高，能够真正替换的交易也就越少 —— 在最差的情况（被攻击）下节点被浪费的带宽也就越少。但若提出了较为严苛的要求，则又阻止了那些只能获得少量增益的交易使用见证数据替换。

- **关于闪电网络 gossip 协议升级的持续讨论**：如[周报 #188][Newsletter #188]所述，闪电网络的协议开发者们正在讨论如何修正闪电网络用来宣传可用支付通道信息的 gossip 协议。具体来说，本周我们看到了两个活跃的讨论：
  - *重大升级*：在[回应][response] Rusty Russell 在上个月提出的的[重大升级][major update]提议时，Olaoluwa Osuntokun 多次表达了对该提议的一个侧面的担忧：该提议将使链上资金和具体的闪电网络通道的链接产生合理推诿属性（plausible deniability）。这种特性将使得非闪电网络的用户更容易宣传一个实际上并不存在的通道，从而降低支付方在网络中发现可用路径、完成支付的能力。

  - *小规模升级*：Osuntokun [发帖][posted]提出了一个独立的提案，给 gossip 协议提出了一个小得多的升级，致力于允许基于 taproot 的通道。这个提案使用[MuSig2][MuSig2] 来允许一个签名为相关的所有 4 个公钥（两个节点的标识符公钥、两个通道花费公钥）提供授权，有可能要求使用 MuSig 2 来花费通道创建交易。

    他也声称，为通道宣告消息加入一个 SPV 部分默克尔分支证明可能是有用的。这将证明，该通道的创建交易已经被某个区块确认；从而轻量级客户端无需下载整个区块来验证该交易存在。

## 来自 Bitcoin Stack Exchange 网站的精选 Q&A

*[Bitcoin Stack Exchange][Bitcoin Stack Exchange] 是 Optech 贡献者寻找答案以及有闲暇时帮助好奇和困惑用户的首选站点。在这个月度栏目中，我们会记录自本栏目上次发布以来出现的高赞问题和回答*。

- [地址重用有哪些好处和坏处？][What are the advantages or disadvantages to address reuse?] RedGrittyBrick 和 Pieter Wuille 列举了地址重用的坏处，包括隐私上的以及关于公钥曝光的有争议的担忧。Wuille 接着指出，虽然生成新的地址没有额外的金融成本，但这会给钱包软件、备份和可用性增加复杂性。
- [什么是 “block-relay-only” 型连接，它有什么用？][What is a block-relay-only connection and what is it used for?]用户 vnprc 介绍了 “block-realy-only（仅转发区块）” 型连接：对等节点仅转发区块信息，而不会转发交易和潜在对等节点的网络地址信息。这些连接可以帮助对抗网络分区（所谓的 “[日蚀攻击][eclipse]” ，因为它使攻击者更难确定比特币网络的拓扑图。vnrpc 继续介绍了 “锚定连接（anchor connections）”，这是一种在节点重启后依旧维持的仅转发区块连接，可以进一步抵抗日蚀攻击。
- [除了区块难度调整以外，还有别的地方需要用到时间戳吗？][Is timestamping needed for anything except difficulty adjustment?] Pieter Wuille 解释了关于区块头的 `nTime ` 时间戳字段的限制条件（必须大于[过去中值时间（MTP）][Median Time Past (MTP)]，并且与节点本地时间的超前量不得超过两小时）并指出：区块的时间戳用在了[难度][difficulty]的计算和交易的[时间锁][timelocks]中。
- [尝试花费被时间锁锁定的 UTXO 是如何被拒绝的？][How are attempts to spend from a timelocked UTXO rejected?] Pieter Wuille 区分了使用交易的 ` nLockTime ` 字段的时间锁，和使用脚本的 [`OP_CHECKLOCKTIMEVERIFY`][`OP_CHECKLOCKTIMEVERIFY`] 实施的时间锁。
## 新版本和候选版本

*流行的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试旧版本*。

- [BDK 0.17.0][BDK 0.17.0] 是这个用于开发比特币钱包的库的新版本。这个版本将使钱包即使离线也能更容易地派生地址。
- [Bitcoin Core 23.0 RC2][Bitcoin Core 23.0 RC2] 是为这个占主导地位的全节点软件的下一个大版本而发布的候选版本。[更新说明草稿][draft release notes]列举了鼓励高级用户和系统管理员在最终版本发布前[测试][test]的多项提升。
- [LND 0.14.3-beta.rc1][LND 0.14.3-beta.rc1] 是这个流行的闪电网络节点软件的候选版本，修复了多个 bug。

## 代码和文档的重大变更

本周出现重大变更的有：[Bitcoin Core][Bitcoin Core]、[C-Lightning][C-Lightning]、[Eclair][Eclair]、[LDK][LDK]、[LND][LND]、[libsecp256k1][libsecp256k1]、[Hardware Wallet Interface (HWI)][Hardware Wallet Interface (HWI)]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK]、[Bitcoin Improvement Proposals (BIPs)][Bitcoin Improvement Proposals (BIPs)] 和 [Lightning BOLTs][Lightning BOLTs]。

- [C-Lightning #5078][C-Lightning #5078] 允许节点高效使用与同一个对等节点的多个通道，包括在路由支付时使用不同于路由消息所指定的、但（与同一个对等节点的）更优的通道。
- [C-Lightning #5103][C-Lightning #5103] 加入了一个新的 ` setchannel ` 命令，可以配置具体一条通道的路由费、最小支付数额，以及最大支付数额。它取代了现在已经弃用的 `setchannelfee` 命令。
- [C-Lightning #5058][C-Lightning #5058] 移除了对最初的定长洋葱数据格式的支持。这个格式也被提议从闪电网络规范中移除（见[BOLTs #962][BOLTs #962]）。升级后的变长格式已经在差不多三年以前[加入到了规范中][added to the specification]，而且，BOLT #962 提供的网络扫描的结果显示，在超过 17000 个公开节点中，只有 5 个不支持该方法。
- [LND #5476][LND #5476] 升级了 ` GetTransactions ` 和 ` SubscribeTransactions ` RPC 方法，提供关于被创建的输出的额外的信息，包括数额和锁定脚本以及该地址（脚本）是否属于内部钱包。
- [LND #6232][LND #6232] 加入了一个配置设定，可以要求所有 [HTLC][HTLCs] 都被在 HTLC 拦截钩子中注册的某个插件处理。这保证了在一个 HTLC 拦截器有时间注册自身之前，不会有 HTLC 被接受或被拒绝。这个 HTLC 连接允许调用一个外部程序来检验一个 HTLC（支付）并确定它应该被接受或拒绝。


[asked]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-March/020167.html

[taproot]:https://bitcoinops.org/en/topics/taproot/

[Newsletter #188]:https://bitcoinops.org/en/newsletters/2022/02/23/#updated-ln-gossip-proposal

[response]:https://lists.linuxfoundation.org/pipermail/lightning-dev/2022-March/003527.html

[major update]:https://lists.linuxfoundation.org/pipermail/lightning-dev/2022-February/003470.html

[posted]:https://lists.linuxfoundation.org/pipermail/lightning-dev/2022-March/003526.html

[MuSig2]:https://bitcoinops.org/en/topics/musig/

[Bitcoin Stack Exchange]:https://bitcoin.stackexchange.com/

[What are the advantages or disadvantages to address reuse?]:https://bitcoin.stackexchange.com/a/112955

[What is a block-relay-only connection and what is it used for?]:https://bitcoin.stackexchange.com/a/112828

[eclipse]:https://bitcoinops.org/en/topics/eclipse-attacks/

[Is timestamping needed for anything except difficulty adjustment?]:https://bitcoin.stackexchange.com/a/112929

[Median Time Past (MTP)]:https://bitcoinops.org/en/newsletters/2021/04/28/#what-are-the-different-contexts-where-mtp-is-used-in-bitcoin

[difficulty]:https://en.bitcoin.it/wiki/Difficulty

[timelocks]:https://bitcoinops.org/en/topics/timelocks/

[How are attempts to spend from a timelocked UTXO rejected?]:https://bitcoin.stackexchange.com/a/112989

[`OP_CHECKLOCKTIMEVERIFY`]:https://github.com/bitcoin/bips/blob/master/bip-0065.mediawiki

[BDK 0.17.0]:https://github.com/bitcoindevkit/bdk/releases/tag/v0.17.0

[Bitcoin Core 23.0 RC2]:https://bitcoincore.org/bin/bitcoin-core-23.0/

[draft release notes]:https://github.com/bitcoin-core/bitcoin-devwiki/wiki/23.0-Release-Notes-draft

[test]:https://github.com/bitcoin-core/bitcoin-devwiki/wiki/23.0-Release-Candidate-Testing-Guide

[LND 0.14.3-beta.rc1]:https://github.com/lightningnetwork/lnd/releases/tag/v0.14.3-beta.rc1

[Bitcoin Core]:https://github.com/bitcoin/bitcoin

[C-Lightning]:https://github.com/ElementsProject/lightning

[Eclair]:https://github.com/ACINQ/eclair

[LDK]:https://github.com/lightningdevkit/rust-lightning

[LND]:https://github.com/lightningnetwork/lnd/

[libsecp256k1]:https://github.com/bitcoin-core/secp256k1

[Hardware Wallet Interface (HWI)]:https://github.com/bitcoin-core/HWI

[Rust Bitcoin]:https://github.com/rust-bitcoin/rust-bitcoin

[BTCPay Server]:https://github.com/btcpayserver/btcpayserver/

[BDK]:https://github.com/bitcoindevkit/bdk

[Bitcoin Improvement Proposals (BIPs)]:https://github.com/bitcoin/bips/

[Lightning BOLTs]:https://github.com/lightning/bolts

[C-Lightning #5078]:https://github.com/ElementsProject/lightning/issues/5078

[C-Lightning #5103]:https://github.com/ElementsProject/lightning/issues/5103

[C-Lightning #5058]:https://github.com/ElementsProject/lightning/issues/5058

[BOLTs #962]:https://github.com/lightning/bolts/issues/962

[added to the specification]:https://github.com/lightning/bolts/issues/619

[LND #5476]:https://github.com/lightningnetwork/lnd/issues/5476

[LND #6232]:https://github.com/lightningnetwork/lnd/issues/6232

[HTLCs]:https://bitcoinops.org/en/topics/htlc/

