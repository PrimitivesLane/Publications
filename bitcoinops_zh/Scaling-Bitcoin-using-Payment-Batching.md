---
title: 'Scaling Bitcoin using Payment Batching'
permalink: /zh/Scaling Bitcoin using Payment Batching/2021/03/26/
name: Scaling Bitcoin using Payment Batching--zh
slug: Scaling Bitcoin using Payment Batching--zh
type: blog post
layout: blog post
lang: zh
---

这篇文章介绍了高频用户如何在实际情况下使用*批量支付（payment batching）*的扩展技术在实际情况下减少约75%的交易费用和块空间使用量。截至2021年1月，多个通用的比特币服务（主要是交易所）已使用批量支付技术，已被许多钱包（包括比特币核心）集成为内置功能，并具备在自定义钱包和支付发送解决方案轻松实现的条件。缺点是，使用该技术可能会导致收款者出现暂时性的突发状况——无法提高手续费，并可能导致隐私泄露。



## 每个收款者的事务大小

使用P2WPKH输入和输出的典型比特币事务包含一个来自付款者的约67 vbytes的输入和两个各约31 vbytes的输出，一个发送给收款者，另一个返回给付款者。另外11个vbytes用于事务开销(版本、锁定时间和其他字段)。

![p2wpkh-batching-best-case](https://bitcoinops.org/img/posts/payment-batching/p2wpkh-batching-best-case.png)

如果我们仅增加4个收款者，每个接收者都包括一个额外的31 vbyte输出，但是如果保持事务不变，则事务的总大小将变为264 vbytes。先前的事务中，140个vbyte全部用来支付单个收款者的费用，而批量交易每个接收者仅花费约53 vbytes的数据-每次支付节省了60％以上。

简单推断最优情况，可以看到每个收款方花费的vbytes数量趋近于单个输出的大小。这样在极限情况可以节省75％以上的费用。

![p2wpkh-batching-cases-combined](https://bitcoinops.org/img/posts/payment-batching/p2wpkh-batching-cases-combined.png)

实际上，一笔事务花费越多，就越有可能需要额外的输入。这并不妨碍批量支付的用处，尽管其有效性降低了。例如，某些服务可能会收到与其付款大致相同价值的付款，因此，每当它们添加一个输出，平均也需要添加一个输入。在这种典型情况下，手续费约可以节省30％。

对于在事务中经常使用不止一个输入的服务，可以使用两步过程来增加节约的费用。第一步，多个小的输入被合并成一个大的输入，使用缓慢(但低费率)的交易，将钱付给自己。第二步，服务使用批量支付从其合并输入中的一个进行支出，并实现前述情况的最佳效率。

如果我们假设合并交易将仅支付正常交易费用的20％，且一次性合并100个输入，那么可以计算一个输出对应一个输入的场景下使用两步过程能节省的费用（同时可以看出，相比之下，简单的最佳场景是已经有大量可用的输入）。

![p2wpkh-batching-after-consolidation](https://bitcoinops.org/img/posts/payment-batching/p2wpkh-batching-after-consolidation.png)

对于典型情况，合并（consolidation）在只进行一次支付时效率会降低，但在批处理多个支付时，它的表现几乎与最佳情况场景一样好。

除了直接节省费用外，批量支付还通过减少每笔付款的字节数来更有效地利用有限的块空间。这增加了用户可以进行的事务数量，因此，在持续不断的需求下，可以更加经济实惠地使用比特币支付。这样，更多地使用批量支付可以降低所有比特币用户的手续费率。

总而言之，批量支付为那些输入数量是输出数量的5到20倍的服务节省了大量的成本。对于没有达到这种输入输出比值情况的服务，单个批处理所节省的成本较小，但可能仍然值得这样做;如果服务也愿意预先整合它们的输入，那么[可以节省相当大的费用](https://bitcoinops.org/en/veriphi-segwit-batching/)。

注意：以上所有图表均以使用P2WPKH输入和输出为前提。我们希望它将来会成为网络上的主要脚本类型（直到出现[更好的情况](https://bitcoinops.org/en/topics/taproot/)）。但是，如果您使用其他脚本类型（P2PKH或使用P2SH或P2WSH的多重签名），则用于花费它们的vbyte数会更大，因此节省率会更高。

## 顾虑

批量支付的降费（fee-reduction）好处确实会带来权衡和使用该技术时需要解决的顾虑。

### 延迟

这是批量支付的主要问题。尽管某些情况自然会自动进行批量支付（例如，矿池在其挖出的块中支付给哈希率（hashrate）提供者），但您可能需要让用户接受，他们的支付不会立即被广播-某些情况下将被挂起一段时间后再与其他提款请求合并。

用户会注意到此延迟，因为在您发送包含其收款的批量支付之前，他们不会在收款钱包中收到有关未确认事务正在进行中的通知。同时，通过延迟发送他们的付款，你也延迟了它被确认的时间（所有其他事情亦然，比如费率）。

为了减轻延迟的问题，可以允许用户在即时广播和延迟广播之间进行选择，并为每个选项提供不同的费用。例如：

```
[X] 免费提款 (6小时内支付)
[ ] 立即提款 (提款手续费 0.123 mBTC)
```

### 隐私泄露

批量支付的第二个问题是它会让用户感到隐私受损。在同一事务中你支付的每个收款人都可以合理地认为，从该事务中获得输出的其他所有用户都是由你支付的。如果您发送了单独的事务，则各付款之间不会产生明显的链上关系，甚至不存在。

![batch-screenshot](https://bitcoinops.org/img/posts/payment-batching/batch-screenshot.png)

请注意，即使不使用批量支付，属于特定比特币服务的交易也通常可以由专家识别，因此批处理并不一定会降低此类情况的隐私性。

通过与其他用户创建的[coinjoin](https://bitcoinops.org/en/topics/coinjoin/)交易中发送批量支付，可以部分缓解此问题。根据所使用的技术，这不会显著降低批处理的效率，却可以显著增强隐私性。但是，以前由比特币服务提供的coinjoin的简单的实现存在一些[缺陷](http://www.coinjoinsudoku.com/)，使它们无法提供显著的隐私优势。截至2021年1月，没有可用的coinjoin实现完全符合批量支付的需求。

### 可能无法收费

最后要考虑的是，可能无法支付批量付款的费用。比特币核心(Bitcoin Core)等交易中继节点对它们中继的交易施加限制，以防止攻击者浪费带宽、CPU和其他节点资源。你自己可以很容易地避免遇到这些限制，但是你发送的支付的接收者可以在子事务中重新响应它们的输出，这些子事务成为包含您的事务组的一部分。

比特币核心0.20(2020年6月)的限制是<sup>[1](https://bitcoinops.org/en/payment-batching/#fn:package-limits)</sup>，一组相关的未经确认的事务的大小不得超过101,000 vbytes，具有超过25个未经确认的祖先事务（ancestors），或具有超过25个后代事务（descendant）。特别是，如果那些从大批中收到付款的人重新使用其未确认的输出，则可以轻松触及后代事务（descendant）限制。

事务组越接近限制，就越不可能使用 [“Child-pays-for-parent”（CPFP）](https://bitcoinops.org/en/topics/cpfp/)或 [“按需付费（RBF）” ](https://bitcoinops.org/en/topics/replace-by-fee/)的费用来提高交易费用。此外，交易中未确认的子项越多，RBF的费用增加的成本就越高，因为您必须为交易的费率（feerate）的增加以及矿工为了适应您的替换而在删除任何子项事务时损失给矿工的所有潜在费用买单。

这些问题并不是批量支付所特有的，独立支付也会有同样的问题。然而，如果一笔独立支付因为独立的收款者花费了他们的输出而无法提升手续费，那么只有那个用户会受到影响。但是，如果批量支付中的某个收款者消费了输出，以至于无法提升手续费，那么该事务中的所有其他收款者也会受到影响。任何收款者都可以轻松地故意创建达到某一限制的事务，如果他们知道您依赖于这种能力，就可以防止提升手续费，这种攻击被称为 [事务固定](https://bitcoinops.org/en/topics/transaction-pinning/)。

## 执行

使用某些现有的钱包实现，例如使用Bitcoin Core的[sendmany](https://bitcoincore.org/en/doc/0.17.0/rpc/wallet/sendmany/) RPC ，批量支付非常容易。查看软件说明文件，了解允许发送多次付款的功能。

```
bitcoin-cli sendmany "" '{
  "bc1q5c2d2ue7x38hcw2ugk5q7y4ae7nw4r6vxcptu8": 0.1,
  "bc1qztjzd7hpf2xmngr7zkgkxsvdqcv2jpyfgwgtsv": 0.2,
  "bc1qsul9emtnz0kks939egx2ssa6xnjpsvgwq9chrw": 0.3
}'
```

如果使用自己的实现，在大多数情况下，您可能已经创建了具有两个输出(支付输出和更改输出)的事务，因此添加对额外输出的支持应该很容易。唯一值得注意的是，比特币核心节点(和大多数其他节点)将拒绝接受或中继超过100,000 vbytes的交易，所以您不应该尝试发送超过这个值的批量支付。

## 建议总结

1. 尝试创建这样的系统:你的用户和客户不希望他们的支付立即被广播，而是愿意等待一段时间(越长越好)。
2. 使用低费率合并大量输入，使得它们可用于花费。
3. 在每个时间窗口内，在同一笔事务中一起发送所有付款。例如，创建一个每小时的[cronjob](https://en.wikipedia.org/wiki/Cronjob)来发送所有待处理的付款。理想情况下，您之前的合并应该允许事务仅包含单个输入。
4. 不要寄托于可以提高批量付款的手续费。这意味着发起事务时要使用足够高的手续费，以确保在您希望的时间范围内有很高的确认概率。例如，使用 Bitcoin Core 的`estimatesmartfee`RPC 中的 `CONSERVATIVE` 模式。

## 脚注

1. Optech认为，几乎所有节点都在使用默认的Bitcoin Core策略来进行事务组限制。但是，这些默认值可能会随时间变化，因此下面的示例提供了一个命令，可用于查找当前限制值以及当前值。

   ```
   $ bitcoind -help-debug | grep -A3 -- -limit
     -limitancestorcount=<n>
          Do not accept transactions if number of in-mempool ancestors is <n> or
          more (default: 25)
   
     -limitancestorsize=<n>
          Do not accept transactions whose size with all in-mempool ancestors
          exceeds <n> kilobytes (default: 101)
   
     -limitdescendantcount=<n>
          Do not accept transactions if any ancestor would have <n> or more
          in-mempool descendants (default: 25)
   
     -limitdescendantsize=<n>
          Do not accept transactions if any ancestor would have more than <n>
          kilobytes of in-mempool descendants (default: 101).
   ```
