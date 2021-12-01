---
title: 'Bitcoin Optech Newsletter #175'
permalink: /zh/newsletters/2021/11/24/
name: 2021-11-24-newsletter-zh 
slug: 2021-11-24-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---



本周的周报索引了一场关于如何允许闪电网络的用户在更高手续费和更高支付可靠性之间自行选择的讨论。也包含了我们的常规栏目：来自 Bitcoin Stack Exchange 问答网站的热门问题和回答、软件的新版本和候选版本公告、流行的比特币基础设施软件重大变更总结。

## 新闻

- <a id="ln-reliability-versus-fee-parameterization" href="#ln-reliability-versus-fee-parameterization)">●</a> 闪电网络 “可靠性 vs. 手续费” 参数化：Joost Jager 在 Lightning-Dev 邮件列表中开了一个[帖子][thread]讨论如何能最方便用户在支付更多手续费（从而更快完成支付）和等待更长时间（以节约手续费）之间作选择。帖子讨论到的困难之一是，如何将存在于单一连续体中的用户偏好，与离散的、多因子的（由寻路算法返回的）路径关联起来？

## 来自 Bitcoin Stack Exchange 的精选问题和回答

*[Bitcoin Stack Exchange][Bitcoin Stack Exchange] 是 Optech 贡献者们有所疑惑时寻找答案的首选之一，也是我们在有闲暇时会去帮助好奇和迷惑的用户的地方。在这个每月一次的栏目中，我们会挑出上一次栏目发布以来新出现的一些高票的问题和回答。*

- <a id="why-is-it-important-that-nonces-when-signing-not-be-related" href="#why-is-it-important-that-nonces-when-signing-not-be-related)">●</a> [为什么每次签名都要使用互不关联的 nonce 值？][Why is it important that nonces when signing not be related?] Pieter Wuille 从数学上讲解了在以下三种情形中，使用同一把公钥签名两次会导致私钥泄露：（两次签名）使用相同的 nonce；使用已知偏移量的 nonce；使用已知因子的 nonce。Wuille 也列举了三种无法攻破的 nonce 生成方案、两种已经被证明不安全的方案，还指出：技术上 “已知是安全的” 和 “尚未被攻破的”，两者之间有巨大的差别。
- <a id="how-could-a-2-byte-witness-program-make-sense" href="#how-could-a-2-byte-witness-program-make-sense)">●</a> [2 字节大小的 witness 程序有什么用？][How could a 2 byte witness program make sense?] 讨论了 [BIP141][BIP141] 对 witness 程序体积的要求：在 2~40 字节之间。Kalle Rosenbaum 畅想了一些 2 字节大小的 witness 程序的可能用途。
- <a id="what-is-the-xpriv-xpub-type-for-p2tr" href="#what-is-the-xpriv-xpub-type-for-p2tr)">●</a> [P2TR 的 xpriv/xpub 类型是什么样的？][What is the xpriv/xpub type for P2TR?] Andrew Chow 提醒，因为更多样化和复杂的脚本会变得更广泛，为了业务分离，没有为 taproot 设计 xpub/ypub/zpub 等价的格式。他建议 “与 xpiv/xpub 搭配使用一些能指明所创建的脚本是 Taproot 脚本的信息（例如 `tr()` 描述符）”。

## 新版本和候选版本

*流行的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本*。

- <a id="lnd-0-14-0-beta" href="#lnd-0-14-0-beta)">●</a> [LND 0.14.0-beta][LND 0.14.0-beta] 是一个新版本，包含了额外的[日蚀攻击][eclipse attack]保护措施（见[周报 #164][Newsletter #164]）、远程数据库支持（见[周报 #157][Newsletter #157]）、更快的寻路算法（见[周报 #170][Newsletter #170]）、Lightning Pool 使用体验提升（见[周报 #172][Newsletter #172]）以及可重用的 [AMP][AMP] 付款请求（见[周报 #173][Newsletter #173]），还有许多新功能和 bug 修复。

## 代码和文档的重大变更

*本周有重大变更的包括：[Bitcoin Core][Bitcoin Core]、[C-Lightning][C-Lightning]、[Eclair][Eclair]、[LND][LND]、[Rust-Lightning][Rust-Lightning]、[libsecp256k1][libsecp256k1]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK] 和 [Lightning BOLTs][Lightning BOLTs]*。

- <a id="c-lightning-4890" href="#c-lightning-4890)">●</a> [C-Lightning #4890][C-Lightning #4890] 让用户可以为钱包配置一个 sqlite 数据库备份文件。在操作期间，所有数据都会在主 sqlite 文件和备份文件间复制。新功能已有[详情文档][Extensive documentation]。
- <a id="rust-lightning-1173" href="#rust-lightning-1173)">●</a>[Rust-Lightning #1173][Rust-Lightning #1173] 加入了一个新的 `accept_inbound_channels` 配置，可以用来防止节点接受新的连入通道。默认值为真。
- <a id="rust-lightning-1166" href="#rust-lightning-1166)">●</a> [Rust-Lightning #1166][Rust-Lightning #1166] 改进了默认的路由评分逻辑，降级支付的 HTLC 数值超过通道容量 1/8 的通道。而且这个差评会随着支付额趋于通道容量而线性增加。



[thread]:https://lists.linuxfoundation.org/pipermail/lightning-dev/2021-November/003342.html

[Bitcoin Stack Exchange]:https://bitcoin.stackexchange.com/

[Why is it important that nonces when signing not be related?]:https://bitcoin.stackexchange.com/a/110811

[How could a 2 byte witness program make sense?]:https://bitcoin.stackexchange.com/a/110660

[BIP141]:https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki

[What is the xpriv/xpub type for P2TR?]:https://bitcoin.stackexchange.com/a/110733

[LND 0.14.0-beta]:https://github.com/lightningnetwork/lnd/releases/tag/v0.14.0-beta

[eclipse attack]:https://bitcoinops.org/en/topics/eclipse-attacks/

[Newsletter #164]:https://bitcoinops.org/en/newsletters/2021/09/01/#lnd-5621

[Newsletter #157]:https://bitcoinops.org/en/newsletters/2021/07/14/#lnd-5447

[Newsletter #170]:https://bitcoinops.org/en/newsletters/2021/10/13/#lnd-5642

[Newsletter #172]:https://bitcoinops.org/en/newsletters/2021/10/27/#lnd-5709

[AMP]:https://bitcoinops.org/en/topics/atomic-multipath/

[Newsletter #173]:https://bitcoinops.org/en/newsletters/2021/11/03/#lnd-5803

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

[C-Lightning #4890]:https://github.com/ElementsProject/lightning/issues/4890

[Extensive documentation]:https://github.com/ElementsProject/lightning/blob/163d3a9203922a0493cf6038493bd4b5e078d987/doc/BACKUP.md#sqlite3---walletmainbackup-and-remote-nfs-mount

[Rust-Lightning #1173]:https://github.com/rust-bitcoin/rust-lightning/issues/1173

[Rust-Lightning #1166]:https://github.com/rust-bitcoin/rust-lightning/issues/1166

