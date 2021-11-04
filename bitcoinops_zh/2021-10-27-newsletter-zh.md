---
title: 'Bitcoin Optech Newsletter #172'
permalink: /zh/newsletters/2021/10/27/
name: 2021-10-27-newsletter-zh 
slug: 2021-10-27-newsletter-zh 
type: newsletter
layout: newsletter
lang: zh
---

本周的周报包含了我们的常规部分：来自 Bitcoin Stark Exchange 问答网站常见问题和回复的总结、关于为 taproot 激活作准备的信息、一系列软件的新版本和候选版本，以及流行的比特币基础设施项目中的重大变更。

## 新闻 

本周无重大新闻。

## Bitcoin Stark Exchange 问答精选

*[Bitcoin Stack Exchange][Bitcoin Stack Exchange] 网站是我们 Optech 贡献者们有疑惑时 —— 以及有闲暇去帮助好奇和困惑的用户时 —— 最先想到的地方之一。在这个每月一次的栏目中，我们会挑出自上一次栏目发布以来新出现的一些高赞问题和回答。*

- <a id="where-to-find-the-exact-number-of-hashes-required-to-mine-most-recent-block" href="#where-to-find-the-exact-number-of-hashes-required-to-mine-most-recent-block)">●</a> [如何知道挖出最近的区块具体需要多少次哈希计算][Where to find the exact number of hashes required to mine most recent block?]？Pieter Wuille 回答：虽然一个区块具体尝试了多少次哈希计算不会在区块中发不出来，不过，使用区块的[难度][difficulty]乘以 42 9503 2833，可以简单估计挖出一个区块所需的次数。
- <a id="using-a-2-of-3-taproot-keypath-with-schnorr-signatures" href="#using-a-2-of-3-taproot-keypath-with-schnorr-signatures)">●</a> [能不能用 Schnorr 签名来实现 2-3 的 taproot 密钥路径][Using a 2-of-3 taproot keypath with schnorr signatures?]？Pieter Wuille 回答：虽然 [BIP340][BIP340] 的要求是 1 把公钥和 1 个签名，但你也可以使用[门限签名][threshold signature]方案，比如 [FROST][FROST]；[多签名][multisignature]方案，比如 [MuSig][MuSig]，等等。
- <a id="why-coinbase-maturity-is-defined-to-be-100-and-not-50" href="#why-coinbase-maturity-is-defined-to-be-100-and-not-50)">●</a> [为什么 coinbase 输出的成熟期要设为 100，而不是 50][Why coinbase maturity is defined to be 100 and not 50?]？用户 liorko 问什么 [coinbase 输出从生产到可用][coinbase maturity]要等 100 个区块而不是 50 个。回答指出这个选择是没法解释的，而且很可能只能拍脑袋选一个。
- <a id="why-does-bitcoin-use-double-hashing-so-often" href="#why-does-bitcoin-use-double-hashing-so-often)">●</a> [为什么比特币里面的哈希计算经常连续做两次][Why does Bitcoin use double hashing so often?]？Pieter Wuille 列举了最开始比特币使用连续两次 SHA256 计算和 SHA256+RIPEMD160 连续哈希方案的地方，也指出了哪些新特性使用了同样的方案，最后猜想中本聪是想使用连续哈希方案来缓解某些攻击（虽然实际上是无效的）。

## 为 taproot 升级作准备 #19：未来的共识变更

*关于开发者和服务提供者如何为即将在区块高度 709,632 处激活的 taproot 作准备的每周[系列][series]文章。*

- <a id="cross-input-signature-aggregation" href="#cross-input-signature-aggregation)">●</a> **跨输入的（cross-input）签名聚合**：[schnorr 签名方案][schnorr signatures]使得多个公私钥对的所有者可以容易地用一个签名来证明所有人都参与创建了签名。搭配未来可能施行的共识变更，它也许可以让一笔事务包含单个签名，但是证明该事务所花费的所有 UTXO 的所有者都授权该花费行为。这样做可以为每个密钥使用路径（keypath）节省 16 vbyte，为合并 UTXO 的行为（consolidation）和 [coinjoin 交易][coinjoins]提供巨大的节约效果。它甚至可以让基于 coinjoin 的（集体）花费行为比你自己花自己的钱还要便宜（指链上手续费），为使用更加隐私的技术提供了不小的激励。
- <a id="sighash-anyprevout" href="#sighash-anyprevout)">●</a>  **SIGHASH_ANYPREVOU**：每个普通的比特币事务都有一个乃至多个输入，每个输入都指向此前某笔事务（以 txid 指定）的某个输出。这个指向告诉全验证节点（比如 Bitcoin Core 客户端）这笔事务可以花多少钱，以及，证明这笔事务有权花费需要满足哪些条件。为比特币事务生成签名的所有方法，无论是否实施了 taproot，要么在 prevout 里面承诺 txid，要么完全不承诺。
  这对于不想提前准确安排事务的多用户协议来说是个问题。只要任何一个用户跳过了某笔事务，或者改变了任何一笔事务中 witness 数据以外的细节，后续事务的 txid 就会改变。而改变 txid 会使之前为后续事务创建的签名全都作废。这就迫使链下协议都要实现一些机制（比如 LN-penalty）来惩罚提交老旧事务的用户。
  [SIGHASH_ANYPREVOUT][SIGHASH_ANYPREVOUT] 可以消除这个问题，它让签名不必承诺前序 txid。根据它使用的不同标签，签名仍会承诺 prevout 和当前事务的其它细节（比如数量和脚本），但无论使用什么 txid 来指定前序交易都不会使签名无效。这使得闪电网络的 [eltoo][eltoo] 层、 [vault][vaults] 中的[提升][improvements]和其它合约协议能够实现。
- <a id="delegation-and-generalization" href="#delegation-and-generalization)">●</a>  **委托和通用化**：在你创建一个脚本之后（无论是不是 taproot 类型的），就[几乎][almost]没有办法委托他人来花费这个脚本中的钱，除非你把自己的私钥给 TA（这是[极度危险][extremely dangerous]的，如果你使用 [BIP32][BIP32] 钱包的话）。但是，taproot 之后，主要使用密钥支付、兼具少量基于脚本的条件就变得更加经济实惠。已经出现了多种通过通用化和提供[签名人委托][signer delegation]功能来强化 taproot 的设想：
  - <a id="graftroot" href="#graftroot)">●</a> **Graftroot**：在 taproot 的想法出现之后很快就有人[提出][proposed]来了。Graftroot 可以给予任何有能力使用 taproot 密钥支付路径的人额外的功能。密钥路径的签名者不是直接花费资金，而是签名一个脚本，表述在哪些新条件下这些资金可以花费，如此把花费的权限委托给任何有能力满足这个脚本的人。签名、脚本以及满足脚本所需的全部数据，都要在事务中提供。密钥路径的签名者可以这样把花费的权限委托给无限数量的脚本、无需创建任何链上数据，直到某笔事务实际上链。
  -  <a id="generalized-taproot-g-root" href="#generalized-taproot-g-root)">●</a> **通用化的 taproot (g’root)**：几个月后，Anthony Towns [提出][suggested] 了一种使用公钥点来承诺多个花费条件而不必使用 [MAST][MAST] 或类似构造的方案。这种 *通用化的 taproot*  “[在 taproot 的假设不满足时][where the taproot assumption doesn’t hold]可能更有效率”。它也 “[提供][offers]了一种简单的办法来构造一种免于使用软分叉的跨输出聚合系统（softfork-safe cross-input aggregation system）”。
  -  <a id="entroot" href="#entroot)">●</a> **Entroot**：一种[最近][more recent]才提处的 graftroot 和 g'root 的结合体，简化了许多情况，并使它们更加节约带宽。它支持从任意能够满足一个 entroot 分支的人发起签名人委托，而不仅限于能够创建顶层密钥路径支付事务的人。
- <a id="new-and-old-opcodes" href="#new-and-old-opcodes)">●</a> **新的和旧的操作码**：taproot 包含了对 [tapscript][tapscript] 的支持，后者提供了一种渐进的办法，`OP_SUCCESSx` 操作码，来为比特币增加新的操作码。类似于比特币早期加入的 `OP_NOPx`（无动作）操作码，`OP_SUCCESSx` 操作码被设计成未来可被并不总是返回成功的操作码替代。人们提出的一些新的操作码包括：
  - <a id="restore-old-opcodes" href="#restore-old-opcodes)">●</a> **回复旧的操作码**：一些与数学和字符串操作有关的操作码因为安全方面的担忧而在 2010 年禁用。许多开发者希望看到这些操作码在经过安全审核之后重新启用，以及（在某些时候）可以延伸到处理更大的数字。
  - <a id="op-cat" href="#op-cat)">●</a> **OP_CAT**：一个值得特别提及的已禁用操作码是 [OP_CAT][OP_CAT]。研究者已经发现，它可以在比特币中[启用][enable]所有[类型][sorts]的[有趣][interesting]行为，也可以与其它新操作码以有趣的方式[相结合][combined]。
  - <a id="op-tapleaf-update-verify" href="#op-tapleaf-update-verify)">●</a> **OP_TAPLEAF_UPDATE_VERIFY**：就像在 [第 166 期周报][Newsletter #166]里面讲的一样，一个 `OP_TLUV` 操作码可以在用户使用 taproot 的密钥路径和脚本路径功能时以非常高效和强大的方式启用[限制条款（covenant）][covenants]。它可以用来实现 [joinpool][joinpools]、[vault][vaults] 以及其它安全和隐私性提升。它也许能跟[OP_CHECKTEMPLATEVERIFY][OP_CHECKTEMPLATEVERIFY] 结合得很好。

上面的所有点子都还在提议阶段。没有一个能保证会成功。都仍待研究员和开发者将它们变得更加成熟，并有待用户来决定哪一个特性值得为之改变比特币的共识规则。

## 新版本和候选版本

*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- <a id="rust-lightning-0-0-102" href="#rust-lightning-0-0-102)">●</a> [Rust-Lightning 0.0.102][Rust-Lightning 0.0.102]：一个新版本，做了许多 API 提升并修复了一个导致无法与 LND 节点开设通道的 bug。
- <a id="c-lightning-0-10-2rc1" href="#c-lightning-0-10-2rc1)">●</a> [C-Lightning 0.10.2rc1][C-Lightning 0.10.2rc1]：一个候选版本，[包含了][includes]对 [“不经济输出” 安全问题][uneconomical outputs security issue]的一个修复。实现了一个体积更小的数据库，并提高了 `pay` 命令的效率（见下文 “ *重大代码和文档更新* ”部分）。

## 重大代码和文档更新

本周内，[Bitcoin Core][Bitcoin Core]、[C-Lightning][C-Lightning]、[Eclair][Eclair]、[LND][LND]、[Rust-Lightning][Rust-Lightning]、[libsecp256k1][libsecp256k1]、[Rust Bitcoin][Rust Bitcoin]、[BTCPay Server][BTCPay Server]、[BDK][BDK] 和 [Lightning BOLTs][Lightning BOLTs] 出现了重大变更。

- <a id="bitcoin-core-23002" href="#bitcoin-core-23002)">●</a> [Bitcoin Core #23002][Bitcoin Core #23002] 在创建新钱包时会默认生成基于 [descriptor][描述符] 的钱包。基于描述符的钱包第一次引入是在[Bitcoin Core PR #16528][Bitcoin Core PR #16528]。有一个[长期计划][long-term plan]是把所有的钱包都迁移成基于描述符的，最终淘汰掉对传统钱包的支持。
- <a id="bitcoin-core-22918" href="#bitcoin-core-22918)">●</a> [Bitcoin Core #22918][Bitcoin Core #22918] 延伸了 `getblock` RPC 方法和 `/rest/block/` 端点，使用新的冗余度（`3`）来包含在当前区块中花费的每一个之前创建的输出（即 “prevout”）的信息。
  在一个区块创建一个新的未花用事务输出（UTXO）时，Bitcoin Core 软件会把它存储在一个数据库里。在一个后来的事务准备花费某个 UTXO 时，Bitcoin Core 软件会在数据库中检索它，并验证事务满足了该 UTXO 的所有花费条件（例如，包含对应于某个公钥的数字签名）。如果一个区块中的所有事务都是有效的，Bitcoin Core 会把这些 prevout 条目从 UTXO 数据库中移出、移到一个 *undo* 文件中；如果后来发生链重组，这个区块从主链上消失，这个文件可以用来把 UTXO 数据库恢复到前一个状态中。
  这个 PR 会在 undo 文件中检索 prevout，并将它们加入实际包含在区块中的信息，或用这些内容开展计算。对于需要 prevout 数据的用户和应用来说，这比直接搜索每一笔前序事务及其输出要更快更方便。开启了修剪数据库功能的全节点会删掉旧的 undo 文件，所以无法为这些已删除的区块使用这一新的冗余度。
- <a id="c-lightning-4771" href="#c-lightning-4771)">●</a> [C-Lightning #4771][C-Lightning #4771] 更新了 `pay` 指令，以优先执行包含了较大容量的通道的路由，其它事情保持不变。一个通道的资金总量是公开可知的；不可知的是两个方向上到底有多少资金可用。不过，如果两个通道处在任意状态的[可能性是相同的][equal probability]，那容量更大的通道能够路由成功的可能性自然更大。
- <a id="c-lightning-4685" href="#c-lightning-4685)">●</a> [C-Lightning #4685][C-Lightning #4685] 基于 [BOLTs #891][BOLTs #891] 中的规范草案，加入了一个实验性的 [websocket][websocket] 通信方式。这让 C-Lightning（和其它支持同一个协议的 LN 实现）能发布一个用来与其他对等节点通信的另类端口。底层的 LN 通信协议没有变化，只是用二进制的 websocket 框架（而非纯粹的 TCP/IP）来执行。
- <a id="eclair-1969" href="#eclair-1969)">●</a> [Eclair #1969][Eclair #1969] 延伸了所有 API 调用的 `findroute*` 集合，使用了多个额外的参数：`ignoreNodeIDs`、 `ignoreChannelIDs` 以及 ` maxFeeMsat`。它也加入了一个 `full` 格式，会返回关于已发现的路由的所有信息。
- <a id="lnd-5709" href="#lnd-5709)">●</a> [LND #5709][LND #5709]（一开始是 [#5549][#5549]）加入了一种新的承诺事务格式，用在支持 Lightning Pool （见 [第 123 期周报][Newsletter #123]）的节点间（当前只有 LND 节点支持 Lightning Pool）。新的格式会防止提供通道租约（channel lease）的节点在租约期满之前在链上花费资金。这样做能激励他们在租约期内保持通道开启，好好利用资金（流动性）来赚取路由费。一个通道的承诺事务格式只有两个直接相连的对等节点（在通道开启期间）才能看到，所以使用什么格式都不会影响网络上的其它节点。
- <a id="lnd-5346" href="#lnd-5346)">●</a> [LND #5346][LND #5346] 为 LND 节点加入了与其对等节点交互定制化消息的能力 —— 要求类型识别符大于 32767。该 pull request 中也提出了定制化消息的一些建议用法。`lncli` 命令行也升级到可以简便第发送和收听定制化消息。
- <a id="lnd-5689" href="#lnd-5689)">●</a> [LND #5689][LND #5689] 为 LND 节点加入了可以把所有的私钥操作委托给一个远程的、大部分时间离线的 “签名器” 节点的功能。详细的文档可见[此处][here]。
- <a id="btcpay-server-2517" href="#btcpay-server-2517)">●</a> [BTCPay Server #2517][BTCPay Server #2517] 增加了通过 LN 来和退款的功能。管理员可以输入一个支付的数量，然后接收方可以输入 TA 的节点的细节，让支付成功发送。
- <a id="hwi-497" href="#hwi-497)">●</a> [HWI #497][HWI #497] 向使用 Trezor 固件的设备发送额外的信息，允许他们验证一个找零地址属于一个多签名的联盟。如无此功能，Trezor 会将找零显示为一笔单独的支付，要求用户手动验证他们的找零地址是正确的。





[Bitcoin Stack Exchange]:https://bitcoin.stackexchange.com/

[Where to find the exact number of hashes required to mine most recent block?]:https://bitcoin.stackexchange.com/a/110330

[difficulty]:https://en.bitcoin.it/wiki/Difficulty

[Using a 2-of-3 taproot keypath with schnorr signatures?]:https://bitcoin.stackexchange.com/a/110249

[BIP340]:https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki

[threshold signature]:https://bitcoinops.org/en/topics/threshold-signature/

[FROST]:https://eprint.iacr.org/2020/852.pdf

[multisignature]:https://bitcoinops.org/en/topics/multisignature/

[MuSig]:https://bitcoinops.org/en/topics/musig/

[Why coinbase maturity is defined to be 100 and not 50?]:https://bitcoin.stackexchange.com/a/110085

[coinbase maturity]:https://bitcoin.stackexchange.com/a/1992/87121

[Why does Bitcoin use double hashing so often?]:https://bitcoin.stackexchange.com/a/110065

[series]:https://bitcoinops.org/en/preparing-for-taproot/

[schnorr signatures]:https://bitcoinops.org/en/topics/schnorr-signatures/

[coinjoins]:https://bitcoinops.org/en/topics/coinjoin/

[SIGHASH_ANYPREVOUT]:https://bitcoinops.org/en/topics/sighash_anyprevout/

[eltoo]:https://bitcoinops.org/en/topics/eltoo/

[improvements]:https://bitcoinops.org/en/preparing-for-taproot/#vaults-with-taproot

[vaults]:https://bitcoinops.org/en/topics/vaults/

[almost]:https://bitcoinops.org/en/newsletters/2021/03/24/#signing-delegation-under-existing-consensus-rules

[extremely dangerous]:https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#implications

[BIP32]:https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki

[signer delegation]:https://bitcoinops.org/en/topics/signer-delegation/

[proposed]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-February/015700.html

[suggested]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-July/016249.html

[MAST]:https://bitcoinops.org/en/topics/mast/

[where the taproot assumption doesn’t hold]:https://bitcoinops.org/en/preparing-for-taproot/#is-cooperation-always-an-option

[offers]:https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-October/016461.html

[more recent]:https://gist.github.com/sipa/ca1502f8465d0d5032d9dd2465f32603

[tapscript]:https://bitcoinops.org/en/topics/tapscript/

[OP_CAT]:https://bitcoinops.org/en/topics/op_checksigfromstack/#relationship-to-op_cat

[enable]:https://blockstream.com/2015/08/24/en-treesignatures/#h.2lysjsnoo7jd

[sorts]:https://rubin.io/blog/2021/07/06/quantum-bitcoin/

[interesting]:https://www.wpsoftware.net/andrew/blog/cat-and-schnorr-tricks-i.html

[combined]:https://bitcoinops.org/en/topics/op_checksigfromstack/

[Newsletter #166]:https://bitcoinops.org/en/newsletters/2021/09/15/#covenant-opcode-proposal

[covenants]:https://bitcoinops.org/en/topics/covenants/

[joinpools]:https://bitcoinops.org/en/topics/joinpools/

[vaults]:https://bitcoinops.org/en/topics/vaults/

[OP_CHECKTEMPLATEVERIFY]:https://bitcoinops.org/en/topics/op_checktemplateverify/

[Rust-Lightning 0.0.102]:https://github.com/rust-bitcoin/rust-lightning/releases/tag/v0.0.102

[C-Lightning 0.10.2rc1]:https://github.com/ElementsProject/lightning/releases/tag/v0.10.2rc1

[includes]:https://twitter.com/Snyke/status/1452260691939938312

[uneconomical outputs security issue]:https://bitcoinops.org/en/newsletters/2021/10/13/#ln-spend-to-fees-cve

[Bitcoin Core]:https://github.com/bitcoin/bitcoin

[C-Lightning]:https://github.com/ElementsProject/lightning

[Eclair]:https://github.com/ACINQ/eclair

[LND]:https://github.com/lightningnetwork/lnd/

[Rust-Lightning]:https://github.com/rust-bitcoin/rust-lightning

[libsecp256k1]:https://github.com/bitcoin-core/secp256k1

[Rust Bitcoin]:https://github.com/rust-bitcoin/rust-bitcoin

[BTCPay Server]:https://github.com/btcpayserver/btcpayserver/

[BDK]:https://github.com/bitcoindevkit/bdk

[Lightning BOLTs]:https://github.com/lightningnetwork/lightning-rfc/

[Bitcoin Core #23002]:https://github.com/bitcoin/bitcoin/issues/23002

[descriptor]:https://bitcoinops.org/en/topics/output-script-descriptors/

[Bitcoin Core PR #16528]:https://bitcoinops.org/en/newsletters/2020/05/06/#bitcoin-core-16528

[long-term plan]:https://github.com/bitcoin/bitcoin/issues/20160

[Bitcoin Core #22918]:https://github.com/bitcoin/bitcoin/issues/22918

[C-Lightning #4771]:https://github.com/ElementsProject/lightning/issues/4771

[equal probability]:https://bitcoinops.org/en/newsletters/2021/03/31/#paper-on-probabilistic-path-selection

[C-Lightning #4685]:https://github.com/ElementsProject/lightning/issues/4685

[websocket]:https://en.wikipedia.org/wiki/WebSocket

[BOLTs #891]:https://github.com/lightningnetwork/lightning-rfc/issues/891

[Eclair #1969]:https://github.com/ACINQ/eclair/issues/1969

[LND #5709]:https://github.com/lightningnetwork/lnd/issues/5709

[#5549]:https://github.com/lightningnetwork/lnd/issues/5549

[Newsletter #123]:https://bitcoinops.org/en/newsletters/2020/11/11/#incoming-channel-marketplace

[LND #5346]:https://github.com/lightningnetwork/lnd/issues/5346

[LND #5689]:https://github.com/lightningnetwork/lnd/issues/5689

[here]:https://github.com/guggero/lnd/blob/d43854aa34ca0c2d0dfa12b06f299def39b512fb/docs/remote-signing.md

[BTCPay Server #2517]:https://github.com/btcpayserver/btcpayserver/issues/2517

[HWI #497]:https://github.com/bitcoin-core/HWI/issues/497

