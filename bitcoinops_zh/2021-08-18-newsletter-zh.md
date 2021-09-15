```
title: 'Bitcoin Optech Newsletter #162'
permalink: /zh/newsletters/2021/08/18/
name: 2021-08-18-newsletter-zh 
slug: 2021-08-18-newsletter-zh 
type: newsletterk
layout: newsletter
lang: zh
```

本周的 Newsletter 总结了关于粉尘限制的讨论，还包括我们的常规部分，介绍了服务和客户端软件的变更，如何为 taproot 做准备，新版本和候选版本，以及主流的比特币基础设施软件中值得注意的变更。

## 新闻
- **粉尘限制讨论**：Bitcoin Core和其他节点软件默认拒绝中继或挖掘任何输出值低于一定金额的交易，即[粉尘限制](https://bitcoinops.org/en/topics/uneconomical-outputs/)（具体限制金额因输出类型而异）。这使得用户更难创造出*不经济*的输出——UTXO，这类UTXO的花费的费用会比其持有的价值更高。

  本周，Jeremy Rubin在Bitcoin-Dev邮件列表中[发布](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-August/019307.html)了关于取消粉尘限制的五点论据，并表示认为限制的原因是为了防止 "垃圾邮件 "和 "[粉尘指纹攻击](https://bitcoinops.org/en/topics/output-linking/)"。其他人[回复](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-August/019308.html)了[反驳](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-August/019310.html)，并指出限制的存在不是为了防止垃圾邮件，而是为了防止用户创建的UTXO永久浪费全节点运营商的资源，而用户又没有动力去花费这些UTXO。部分讨论还[描述](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-August/019327.html)了粉尘限制和不经济的产出对LN的[影响](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-August/019333.html)。

  截至本文写作时，似乎没有可能达成一致。至少在短期内，我们预计dust limit将继续存在。

## 服务和客户端软件更新
*在这个月的专题中，我们重点介绍比特币钱包和服务的有趣更新。*

- **Spark Lightning Wallet增加了对BOLT12<sup>[[1]](#myfootnote1)</sup>
的支持**：[Spark](https://github.com/shesek/spark-wallet)的[v0.3.0rc版本](https://github.com/shesek/spark-wallet/releases/tag/v0.3.0rc)增加了对BOLT12 [offers](https://bitcoinops.org/en/topics/offers/)的部分支持。

- **Blockstream宣发非托管LN云服务，Greenlight**：在最近的一篇[博客文章](https://blockstream.com/2021/07/21/en-greenlight-by-blockstream-lightning-made-easy/)中，Blockstream详细介绍了他们的C-Lightning-nodes-in-the-cloud服务，将节点运营方（Blockstream）与节点的资金控制（用户）分开。[Sphinx](https://sphinx.chat/)和[Lastbit](https://gl.striga.com/)目前都使用Greenlight服务。

- **BitGo变更native segwit为默认找零输出方案**：鉴于segwit的采用率超过了75%这个里程碑，[BitGo的博客文章](https://blog.bitgo.com/native-segwit-change-outputs-for-bitcoin-c021406aaae2)宣布更新他们的默认找零输出，从P2SH-wrapped转向[native segwit](https://bitcoinops.org/en/topics/bech32/)输出。

- **Blockstream Green桌面0.1.10版发布**：[0.1.10版本](https://github.com/Blockstream/green_qt/releases/tag/release_0.1.10)增加了segwit-by-default单签名钱包和手动[选币](https://bitcoinops.org/en/topics/coin-selection/)功能。

## 为 taproot 做准备 #9: 签名适配器
*关于开发者和服务提供者如何为即将在区块高度 709,632 处激活的 taproot 做准备的每周[系列](https://bitcoinops.org/en/preparing-for-taproot/)文章。*

想象一下，有一个人提出向某个特定的慈善机构捐赠1000个BTC，条件是有人能猜出他最喜欢的一个非常大的数字。对于这个捐赠者来说，一个简单方法是创建一个没有签名的交易，支付1000个BTC，然后发布一个加密的交易签名副本，将他最喜欢的数字作为解密密钥。

从理论上讲，任何猜到这个数字的人都可以解密签名，然后广播交易，将钱支付给慈善机构。但是，如果捐赠者使用了像AES这样的标准加密方案，在解密之前第三方就没有简单的办法知道签名对该交易确实有效。任何想尝试猜测数字的人，都只能选择相信捐赠者是真诚的，不是一个骗子。

让我们把这个问题再延伸一下。第三方Alice和Bob想在签名是否被揭示上打赌。他们也许可以向签名者要签名的哈希值，并将其作为[HTLC](https://bitcoinops.org/en/topics/htlc/)函数中的哈希值，但这又需要相信捐赠者是诚实的。即使签名最终被揭露，捐赠者也可以通过给他们一个不正确的哈希值来破坏Alice和Bob的赌约。

### 神奇的适配器
[签名适配器](https://bitcoinops.org/en/topics/adaptor-signatures/)，通常也被称为*适配器签名*和*一次性可验证的加密签名*，是前述问题以及目前在比特币生产系统中实际面临的许多其他问题的解决方案。虽然可以使用比特币现有的ECDSA签名方案，但是将适配器和应用了基于[BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki)实现的[schnorr签名](https://bitcoinops.org/en/topics/schnorr-signatures/)的taproot结合，会更加简单和经济。让我们看看如果我们使用适配器，上面的例子会有什么变化。

和以前一样，捐赠者准备了一笔1000BTC的交易。他还是以正常的方式签名，唯一不同的是，他分两部分生成nonce：一个真正随机的nonce，并且会永远被保持私密；另一部分是他们最喜欢的数字——他们在一开始会保持私密，但之后可以被其他人揭示。捐赠者使用这两个值生成分别完全有效的签名，然后把它们组合在一起，结果就像是用单个的nonce生成的一样。

BIP340签名承诺使用两种形式的nonce：一种是数字表示（称为*标量*），通常只有签名者知道；另一种是椭圆曲线（EC）上的一个*点*，它被公布出来以进行验证。

捐赠者从他们的有效签名中提取承诺部分并减去隐藏的标量。这使得签名不完整（因此无效），但这使得捐赠者可以分享（无效的）签名承诺，（有效的）完整nonce的点，和（有效的）隐藏数字的点。这三条信息加在一起就是一个*签名适配器*。

使用BIP340签名验证算法的一个小变体，一旦隐藏的标量被简单地加到（目前无效的）签名承诺中，任何人都可以验证签名适配器是否提供了一个有效的签名。即使不知道那个隐藏的数字是什么，这也是可以验证的。简而言之，现在用户可以在没有信任假设的基础上猜测隐藏标量的值，并且能确保一个正确的猜测将使他们获得签名并发送交易。

像其他收到捐赠者签名适配器的人一样，Alice和Bob现在有一份隐藏数字的EC点。也和其他所有人一样，他们不知道实际的标量。但是，像前面介绍的那样，捐赠者把他有效签名变成无效签名的做法，是把隐藏的标量从他们的签名承诺中减去，同时继续让签名承诺基于隐藏数字的点。Alice同样可以通过不基于她不知道的标量而是基于她已知的EC点创建一个无效的签名。她通过创建她自己的nonce对来实现，在创建她的（无效的）签名时使用私有形式，但聚合承诺她公共形式的nonce和捐赠者的签名适配器的EC点。这就产生了一个支付给Bob的交易的签名适配器。如果Bob知道了标量，他就可以把这个适配器转换成有效的签名并发送交易，赢得赌注。

但是，鲍勃是如何得知被猜中的数字呢？难道他要等着猜中的人发布新闻吗？不是的。再回忆一下，捐赠者公布的签名适配器是他们的实际签名减去标量。当隐藏的数字被发现，有人发送1000BTC的交易时，他们必须公布原始（有效）的签名承诺。Bob可以从该（有效）签名承诺中减去原始签名适配器中的（无效）签名承诺，得到标量。然后，他使用该标量将Alice的适配器转换为有效签名。

### 多签适配器
上一节展示了单个用户如何修改了他们创建签名的方式来产生签名适配器。[多重签名](https://bitcoinops.org/en/topics/multisignature/)的各方也有可能使用同样的技巧。这一点格外有用，因为许多使用签名适配器的情况需要两个用户的合作。

例如，当Alice和Bob进行上述打赌时，他们可能首先将资金存入一个只能由他们控制的多重签名花费的脚本。然后Alice可以以签名适配器的形式产生她的部分签名；如果Bob知道了隐藏的数字，他可以将Alice的适配器转化为她有效的部分签名，然后提供他的部分签名以产生一个完整的签名来花费这笔钱。

这就使签名适配器具备了一般多签名的所有优点：它们看起来和单个签名一样，使用的空间也一样大，最大限度地减少了费用，最大限度地提高了隐私和灵活性。

在下周的*为taproot做准备*的专栏中，我们将探讨我们期望看到的签名适配器的主要使用方式之一。时间点锁定合约（[PTLC](https://bitcoinops.org/en/topics/ptlc/)），他是对被广泛使用在LN、coinswaps和其他一些协议中的哈希时间锁定合约（[HTLC](https://bitcoinops.org/en/topics/htlc/)）的升级。

## 发布和候选发布
*主流的比特币基础设施项目的新版本和候选版本。请考虑升级到新版本或帮助测试候选版本。*

- [Bitcoin Core 22.0rc2](https://bitcoincore.org/bin/bitcoin-core-22.0/)是下一个主要版本的全节点实现及其相关钱包和其他软件的候选发布版本。这个新版本的主要变化包括支持[I2P](https://bitcoinops.org/en/topics/anonymity-networks/)连接，取消了对[第二版Tor](https://bitcoinops.org/en/topics/anonymity-networks/)连接的支持，并加强了对硬件钱包的支持。

- [Bitcoin Core 0.21.2rc1](https://bitcoincore.org/bin/bitcoin-core-0.21.2/)是Bitcoin Core当前维护版本的候选发布版本。它包含几个错误的修复和小的改进。

## 重大代码和文档更新
*本周 [Bitcoin Core](https://github.com/bitcoin/bitcoin)、[C-Lightning](https://github.com/ElementsProject/lightning)、[Eclair](https://github.com/ACINQ/eclair)、[LND](https://github.com/lightningnetwork/lnd/)、[Rust-Lightning](https://github.com/rust-bitcoin/rust-lightning)、[libsecp256k1](https://github.com/bitcoin-core/secp256k1)、[Hardware Wallet Interface(HWI)](https://github.com/bitcoin-core/HWI)、[Rust Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin)、[BTCPay Server](https://bitcoinops.org/en/newsletters/2021/08/11/)、[Bitcoin Improvement Proposals(BIPs)](https://github.com/bitcoin/bips/) 和 [Lightning BOLTs](https://github.com/lightningnetwork/lightning-rfc/) 中值得注意的变更。*

- [Bitcoin Core #22642](https://github.com/bitcoin/bitcoin/pull/22642)更新了Bitcoin Core即将发布的22.0版本的发布流程，将所有[重复构建](https://bitcoinops.org/en/topics/reproducible-builds/)二进制文件的人的GPG签名串联成一个文件，可以批量验证（[示例](https://gist.github.com/harding/78631dbcd65ff4a499e164c4e9dc85d4)）。从确定的开发者获取签名的方式已经存在多年，这次更新会使这些签名更容易获得，同时也减少了现在对项目主要维护者签署发布二进制文件的依赖。

- [Bitcoin Core #21800](https://github.com/bitcoin/bitcoin/issues/21800)实现了mempool包接受前序和后序交易的限制。作为对DoS攻击的保护，Bitcoin Core限制其mempool中关联交易的数量，使区块构建对矿工来说是可行的。默认情况下，这些[限制](https://bitcoinops.org/en/newsletters/2018/12/04/#fn:fn-cpfp-limits)确保mempool中的任何交易以及其mempool中的去前序交易不能超过25个交易或101KvB的权重。同样的规则适用于交易与它的mempool后序交易。

  当一个交易被考虑添加到mempool中时，这些前序和后序交易的限制会被执行。如果添加该交易会导致其中一个限制被打破，那么该交易会被拒绝添加。虽然包的语义还没有最终确定，[#21800](https://github.com/bitcoin/bitcoin/issues/21800)实现了在验证任意的包（即当多个交易被考虑同时添加到mempool）时，前序和后序交易的限制检查。Mempool包的接受当前只在[#20833](https://bitcoinops.org/en/newsletters/2021/06/02/#bitcoin-core-20833)中实现了测试版本，最终将作为[包中继](https://bitcoinops.org/en/topics/package-relay/)的一部分实现在p2p网络中。

- [Bitcoin Core #21500](https://github.com/bitcoin/bitcoin/pull/21500)更新了带有`private`参数的`listdescriptors` RPC，如果这个参数被设置，会在调用返回结果时返回每个描述符的私有形式。私有形式包含任何已知的私钥或衍生私钥（xprvs），让这个新命令可以被用来备份钱包。

- [Rust-Lightning #1009](https://github.com/rust-bitcoin/rust-lightning/issues/1009)增加了一个`max_dust_htlc_exposure_msat`通道配置选项，限制余额低于[粉尘限制](https://bitcoinops.org/en/topics/uneconomical-outputs/)的通道提交 "粉尘HTLC"。

  这一变化是为[拟议](https://github.com/lightningnetwork/lightning-rfc/issues/873)的`option_dusty_htlcs_uncounted`功能位做准备，该功能位表示节点不希望将 "粉尘HTLC"计入`max_accepted_htlcs`。节点运营商可能希望采用这个特征位，因为`max_accepted_htlcs`主要用于限制链上交易的潜在规模，如果发生强制关闭，"粉尘HTLC"在链上不会被认可，也绝不会影响最终的交易规模。

  新增加的`max_dust_htlc_exposure_msat`通道配置选项确保即使`option_dusty_htlcs_uncounted`被打开，用户仍然可以限制 "粉尘 HTLC"的总余额，因为这个余额会在强制关闭时丢失，也就是支付给矿工。


<a name="myfootnote1">[1]</a>译者注：更多关于bolt12的资料可参考[bitcoinmagazine](https://bitcoinmagazine.com/technical/explaining-bolt-12)和[官方资料](https://bolt12.org/)
