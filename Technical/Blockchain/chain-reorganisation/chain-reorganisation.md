# <center>链重组</center>
<center>禁用和启用区块以采用新的最长链。</center>

![chain-reorganisation-1.png](img/chain-reorganisation-1%20(1).png)

链重组（或“重组”）发生在你的节点接收到新的[最长链](../longest-chain/longest-chain.md)的区块时。你的节点将停用旧的最长链中的区块，而选择构建**新的最长链**的区块。

这个过程使得网络中的各个节点就**相同版本的区块链**达成一致，因为全球公认的区块链视图始终是具有最长区块链的那个*。

*从技术上讲，它是工作量最大的链，但是通常情况下，区块数量最多的链也是这个。

## 什么时候会发生链重组？

链重组最常发生在两个区块同时被开采之后.

![chain-reorganisation-2.png](img/chain-reorganisation-2%20(1).png)

由于区块在网络中的传播速度不同，有些节点会先接收到一个区块，另一些节点会先接收到另一个区块。因此，关于其中哪个新区块实际上是“最早的”并应位于区块链顶部，将会存在分歧。

![chain-reorganisation-3.png](img/chain-reorganisation-3%20(1).png)

<center>节点对于区块链的版本有不同的看法。</center>

那么，如何解决这个争议，确保每个人都同意相同的区块链版本呢？

我们可以通过挖出下一个区块来解决这个问题。下一个被挖出的区块会在现有的区块上构建，从而形成一个新的最长链。当各个节点接收到这个新挖出的区块，它们会发现这个区块构成了一个新的最长链，然后每个节点都会进行链的重组，以接受这个新的最长链，从而使得所有节点对区块链的版本达成一致。

![chain-reorganisation-4.png](img/chain-reorganisation-4%20(1).png)

<center>旧的短链中的区块被停用，新的更长链中的区块被激活。</center>

因此，由于区块链的链重组，每个节点都能独立地达成对区块链版本的一致。

## 旧的最长链中的交易会发生什么？

由于链重组，被停用的区块（也称为“孤立区块”）中的交易不再是区块链的交易历史的一部分。

![chain-reorganisation-5.png](img/chain-reorganisation-5%20(1).png)

<center>只有最长的区块链中的交易才是有效交易历史的一部分。</center>

因此，如果你试图花费孤立区块中的比特币，节点会拒绝你的交易，因为你试图花费不存在于有效链中的比特币。

![chain-reorganisation-6.png](img/chain-reorganisation-6%20(1).png)

<center>孤立区块中的交易就像从未发生过一样。</center>

如果同时挖掘了两个区块，它们很可能会包含相同的交易，因此重组通常不会给任何人带来问题。

![chain-reorganisation-7.png](img/chain-reorganisation-7%20(1).png)

<center>你的交易很可能被包含在链重组后的“竞争”区块中。</center>

如果孤立区块中存在不在竞争区块中的交易，那么它们将被发送回节点的内存池，并在网络中再次传播，以便有机会在未来的区块中被挖掘。

![chain-reorganisation-8.png](img/chain-reorganisation-8%20(1).png)

但这并不能保证。如果交易不存在于活跃的链中，那么它可能就等于从未发生过。

**提示**：在将交易视为最终交易之前，等待交易在区块链中深入超过一个区块是值得的。因为区块链会出现被重新组织的情况，然后你需要等待/希望交易重新被挖掘到最长的链中。

### 孤立区块的视觉示例
在区块高度[578141](https://learnmeabitcoin.com/explorer/blockchain/578141)处，我们可以看到区块链进行了链重组，一些孤立区块中的交易被重新挖掘回到了链中。

![chain-reorganisation-9.png](img/chain-reorganisation-9.png)

在[比特币Neo4j](../../../Neo4j/Neo4j.md)图数据库中看到的区块578141。

## 常见问题

### 链重组能有多大？

链重组可以涉及任意数量的区块。

如果你的节点收到了一个比你当前活动链更长的新链，你的节点将进行链重组以采用新链，无论它有多长。

>这就是为什么拥有大部分哈希能力的矿工可以通过 [51% Attack](../51-attack/51-attack.md)替换你当前最长链中的区块和交易。因为在区块链中，最长的链总是会胜出。

然而，“自然”的链重组（由于同时挖掘两个区块而发生的）**很少涉及链中顶部以外的区块**。

以下是我[节点](https://learnmeabitcoin.com/explorer)**最近经历**的链重组：
|高度|长度|日期|
|---|---|---|
|[803389](https://learnmeabitcoin.com/explorer/blockchain/803389)|	1	|2023 年 8 月 16 日|
|[800786](https://learnmeabitcoin.com/explorer/blockchain/800786)|	1	|2023 年 7 月 29 日|
|[792379](https://learnmeabitcoin.com/explorer/blockchain/792379)|	1	|2023 年 6 月 1 日|
|[789147](https://learnmeabitcoin.com/explorer/blockchain/789147)|	1	|2023 年 5 月 10 日|
|[788837](https://learnmeabitcoin.com/explorer/blockchain/788837)|	1	|2023 年 5 月 8 日|
|[788805](https://learnmeabitcoin.com/explorer/blockchain/788805)|  1	|2023 年 5 月 8 日|
|[781487](https://learnmeabitcoin.com/explorer/blockchain/781487)|	1	|2023 年 3 月 19 日|
|[781277](https://learnmeabitcoin.com/explorer/blockchain/781277)|	1	|2023 年 3 月 18 日|
|[730848](https://learnmeabitcoin.com/explorer/blockchain/730848)|	1	|2022 年 4 月 7 日|
|[685135](https://learnmeabitcoin.com/explorer/blockchain/685135)|	1	|2021 年 5 月 27 日|

|长度	|高度	|日期|
|---|---|---|
|2| -|-|

## 链重组事件有多频繁？

这种情况并不常见。要想你的节点经历可靠的链重组，需要满足以下条件：

1. 同时挖掘了两个区块。
2. 你的节点首先收到其中一个区块，但另一个区块最终被挖掘在其上并成为新的最长链。

我不知道这个的数学概率是多少，所以以下是基于我的比特币[节点](https://learnmeabitcoin.com/explorer)的数据显示的链重组的频率（该[节点](https://learnmeabitcoin.com/explorer)自**2016年12月17日**以来一直在连续运行）：

* **实际重新组织：12次[^1]**每30,112个块/203.8天一次）
* **避免重新组织：188次[^2]**（每1,922个块/13天一次）
  
## 在哪里可以找到链重组？
你可以使用**bitcoin-cli getchaintips**命令查看 你的节点观察到的链重组事件：
```
[
  {
    "height": 589919,
    "hash": "000000000000000000149b18e74316248d106e42ca410f509305ae58ccda6b13",
    "branchlen": 0,
    "status": "active"
  },
  {
    "height": 578141,
    "hash": "0000000000000000001253a5f37d3763dbe928d21f7d72a708f05268c044179c",
    "branchlen": 1,
    "status": "valid-fork"
  },
  {
    "height": 575695,
    "hash": "0000000000000000002409ed07fdbb1d0359a0c516014115c5451aea724baec8",
    "branchlen": 1,
    "status": "valid-headers"
  },
  ...
```

* **brachlen**字段告诉你竞争区块链中有多少个区块。

* 该**status**字段表示以下内容：
    * **active** - 这是当前的活动链（最长链）。
    * **valid-fork** - **节点执行了链重组**。我们下载并验证了这些区块，并将它们作为活动链的一部分，但后来在接收到新的更长的区块链后将其停用。
    * **valid-headers** - **节点观察到可能发生链重组**。我们下载了这些区块，但由于活动链是等效的且变得更长，所以没有验证它们。
    * **headers-only** - **节点观察到可能发生链重组**。我们接收了一个竞争链的区块头，但没有下载完整的区块。
    * **invalid** - 包含无效区块的分支。
    
![chain-reorganisation-10.png](img/chain-reorganisation-10%20(1).png)

状态为**valid-fork**的分支是最初认为是活动区块链的一部分的区块，但在接收到新的更长的区块链后会将其停用。

此外，具有有效**valid-headers** 的分支是在已经有等效活动链之后收到的竞争链。这些可能导致重组，但我们的链仍然是最长的链，因此没有进行重组。

>如果你没有连续几周或几个月运行节点，则不太可能看到任何链重组。当你的节点下载区块链时，它只会下载最长链中的块（而不是任何分支和旧的链重组中的块）。 你的节点需要在它们发生时经历链重组才能在**bitcoin-cli getchaintips**中显示出来。

## 总结
链重组是比特币节点功能的正常组成部分。这是实现网络中所有节点对区块链共识的一种方式。

如果由于链重组，某些区块被停用，那么这些区块内的交易就会变得无效。但这些无效的交易并不会消失，而是会被放回到内存池中，等待有机会被加入到新的最长链的区块中。

## 链接
*  https://bitcoin.stackexchange.com/questions/44437/how-to-detect-a-fork-with-bitcoin-cli?rq=1
*  https://bitcoin.stackexchange.com/questions/91111/understanding-getchaintips-in-terms-of-chain-reorganisations
*  https://github.com/bitcoin/bitcoin/blob/46d6930f8c7ba7cbcd7d86dd5d0117642fcbc819/src/rpc/blockchain.cpp


[^1]:我们收到了一个新的最长区块链并更新了它，停用了旧的最长区块链中的区块。
[^2]:我们听说有一条链可能成为新的最长链，但我们当时的活跃链仍然是最长的。