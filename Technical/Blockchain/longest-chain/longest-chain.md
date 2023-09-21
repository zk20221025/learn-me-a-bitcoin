# <center>最长链</center>

<center>节点采用的区块链。</center>

![longest-chain-1.png](img/longest-chain-1%20(1).png)

**最长的链**是各个[节点](../../../Beginners/How%20Bitcoin%20Works/1.Network/Nodes/Nodes.md)接受作为[区块链](../blockchain.md)有效版本的链。

节点采用最长的区块链规则，使**网络上的每个节点都能**够就区块链的正本达成一致，从而对交易历史达成共识。

换句话说，这意味着在网络上独立操作的计算机可以维护一个全球共享的文件视图。

>工作量证明链是同步问题的解决方案，无需信任任何人就能知道全球共享视图是什么。-[中本聪](https://satoshi.nakamotoinstitute.org/emails/cryptography/7/)

## 最长的链是哪个?
最长的链是**花费最多精力**来构建的区块链。

要向区块链添加新区块需要大量的[计算能力](../../Mining/Mining.md)，这意味着区块链上的每个块都需要消耗大量的能源才能得到。

![longest-chain-2.png](img/longest-chain-2%20(1).png)

<center>添加新的块需要能量。</center>

区块数更多的区块链比区块数较少的链需要更多的能量来建立，按照规则，而节点总是会选择这个“最长”的链，而不是“更短”的链。

![longest-chain-3.png](img/longest-chain-3%20(1).png)

<center>更长的链需要更多的工作来建造。</center>

节点将始终采用消耗最多能量建造的链，这就是我们所说的“最长链”。

>最长链代表的是多数决定，即其中投入最大工作量证明的链。- [中本聪](https://nakamotoinstitute.org/bitcoin/)

## 最长的链是拥有最多区块的链吗？
听起来大概是对的，但实际上，消耗最多能量构建的链并不一定是区块最多的链。你知道，[难度](../../../Beginners/How%20Bitcoin%20Works/2.Mining/3.Difficulty/Difficulty.md)的变化意味着有些区块需要比其他区块更多的能量来挖掘。

例如，在同一难度期间，每个新区块都需要相同的挖掘工作量，因此对区块链添加的“工作”量也相同：

![longest-chain-4.png](img/longest-chain-4%20(1).png)

目标是[区块](../../Mining/Target/Target.md)必须达到的条件，才能添加到区块链中。

然而，如果难度增加（因为区块比平均每10分钟被挖掘得更快），新难度期的区块将需要更多的努力才能挖掘到区块链上。

![longest-chain-5.png](img/longest-chain-5%20(1).png)

现在，由于节点采用工作量最大的链，如果构建链不需要太多的工作量，那么节点实际上不会采用它，即使它的块数更多。例如，如果两个版本的区块链跨越了多个难度周期，节点将采用具有最多累积“链工作量”的版本，而不仅仅是区块最多的版本：

![longest-chain-6.png](img/longest-chain-6%20(1).png)

综上所述，“最长的链”一词指的是**构建所需最多能量的区块链**。在大多数情况下，这是包含最多区块的链，但更准确地说，它是具有最多**工作量**的链。

>**注意**：在[比特币的第一个版本](https://github.com/Dan-McG/bitcoin-0.1.0)中，Satoshi实际上使用区块数作为确定最长链的度量标准，认为这是构建所需最多工作的链。然而，这种方法容易受到操纵，因此后来改为使用链工作量作为最长链的度量标准[^1]。

## 你如何计算最长的链？
最长的区块链是通过叫做“区块链上工作量总值”的度量来衡量的。

>[区块链上工作量总值]是指产生当前链所需的总哈希数。-[Pieter Wuille](https://bitcoin.stackexchange.com/a/26894/24926)
  
要计算区块链上工作量总值，只需要计算出挖掘链中**每个区块所需的哈希数**，然后将**它们相加**即可。

![longest-chain-7.png](img/longest-chain-7%20(1).png)

每个区块的平均预期哈希数量取决于当时的[目标](../../Mining/Target/Target.md)。

### 区块链上工作量总值计算解释
挖矿的过程涉及对[块头](../../Block/block-header/block-header.md)进行哈希计算。

每次执行哈希计算时，[哈希函数](../../Block/block-hash/block-hash.md)会输出一个**256位的数字**，该数字可能在**0和以下任何数字之间**：
```
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
```
为了成功将此区块挖掘到区块链上，此哈希结果需要低于该特定高度在链中的[目标](../../Mining/Target/Target.md)值。[第一个区块](https://learnmeabitcoin.com/explorer/block/000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f)的目标值设置为：
```
0x00000000ffff0000000000000000000000000000000000000000000000000000
```
为了计算平均需要进行多少次哈希才能达到该值以下，需要将数字的最大范围除以想要达到的数值。
```
0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff /
0x00000000ffff0000000000000000000000000000000000000000000000000000
= 0x0100010001
```
这意味着你需要平均执行0x0100010001（4295032833）次哈希才能获得低于此目标值的结果。这实际上是第一个块的链工作量。

要计算链中的**总区块链上工作量总值**，只需计算每个块的预期哈希值并将它们**相加**即可。

>你可以通过查看[块头](../../Block/block-header/block-header.md)中的[比特](../../Block/block-header/bits/bits.md)字段来查找每个块的目标。

### 更简单的例子
假设你正在随机生成1到100之间的数字，并且希望随机生成一个**小于或等于5**的数字。平均而言，在你获得一个低于目标的数字之前，你需要生成多少个数字？
```
100 / 5 = 20
```
平均而言，**你需要生成20个数字**才能得到一个小于**5**的数字。

这与比特币中的计算完全相同，只不过比特币使用的数字更大(通常以[十六进制](../../Other/Hexadecimal/hexadecimal.md)表示)。

## 为什么节点会采用最长的链？
让节点采用最长可用的链，使得网络中的计算机能够共享区块链的相同视图。

以下是两个例子，说明这个规则的实用性：

### 1. 解决在同时挖掘两个区块时的分歧。

由于比特币是在网络上运行的，因此有可能两台独立的计算机同时挖掘一个区块。在这种情况下，网络中的节点将对哪个区块应该在区块链的顶部产生分歧。

![longest-chain-8.png](img/longest-chain-8%20(1).png)

<center>每个节点将它们收到的第一个区块放在它们的区块链的顶部。</center>

然而，这种情况可以通过节点采用最长的块链来解决。这是因为**下一个要挖掘的块**将建立在这两个块之一上，创建一个新的最长链，所有网络上的节点都将乐意采用。

![longest-chain-9.png](img/longest-chain-9%20(1).png)

节点愿意放弃较短的链，选择新的更长链。这被称为[链重组](../chain-reorganisation/chain-reorganisation.md)。

因为挖矿的不可预测性和数据在网络中传播的速度，节点在任何时候都可能存在分歧。然而采用最长可用链意味着节点最终将始终达成对区块链的相同共识。

### 2. 保护已经挖掘到区块链上的区块。
节点总是采用最长的链作为区块链的有效版本，这意味着很难替换链中的区块（因此也难以替换交易）。

如果有人想要替换区块链中的交易，他们需要努力构建一个新的最长链来替换当前的链。然而，如果大多数矿工不断努力扩展相同的当前最长已知链，一个独立的矿工将无法竞争以超过所有其他矿工的工作量。

![longest-chain-10.png](img/longest-chain-10%20(1).png)

你需要掌握大多数的挖矿算力，才能超过其他矿工，建立一个新的最长链（称为[51%攻击](../51-attack/51-attack.md)）。

实际上，矿工合作扩展同一条链的努力保护了现有的块和交易，防止被单个矿工替换。

>可以将其视为合作努力来建立一条链。-[中本聪](https://satoshi.nakamotoinstitute.org/posts/bitcointalk/8/)

## 为什么矿工要在最长的区块链上建造？

因为矿工可以通过挖掘区块来获得**区块奖励**。

此外，只有当区块**在最长的链中深入100个块时**，该区块奖励中的比特币才能被使用。因此，该区块奖励激励矿工始终尝试挖掘新的区块，以便这些区块成为最长链的一部分（始终尝试在当前最长链上进行构建）。

![longest-chain-11.png](img/longest-chain-11%20(1).png)

<center>仅当该块是最长链的一部分时才能使用块奖励。</center>

>矿工通过[coinbase交易](../../Transaction/Coinbase%20Transaction/Coinbase%20Transaction.md)获得区块奖励。

## 什么情况下的交易不属于最长链？
不在最长链中的区块内的交易是**无效**的。

如果你试图花费不在最长链中的交易中的比特币，节点将不会接受它，也不会尝试将其挖掘到区块中。这是因为节点只认为**最长链是交易的有效历史记录**，而任何在最长链之外的交易都是无效的。

![longest-chain-12.png](img/longest-chain-12%20(1).png)

<center>不属于最长链的交易中的比特币是无法使用的。</center>

因此，只有最长链内的交易被认为是有效交易历史的一部分，而任何在最长链之外的交易实际上从未发生过。

>**提示**：这就是为什么建议你在交易成功确认2个或更多区块后才认为比特币是“你的”。总有一种可能性，即由于[链重组](../chain-reorganisation/chain-reorganisation.md)，区块链中的最顶层区块可能会发生变化，使以前有效的区块和交易无效。

## 摘要
采用最长的区块链可以使计算机网络上的节点能够共享全球公认的区块链视图。此外，添加新区块需要消耗能量，这使得任何个体难以替换已经被挖掘到链中的区块。

“最长的链”通常指具有连续块数量最多的链，但实际上根据挖掘每个块的难度，它指的是具有最多工作量的链。

## 命令
你可以使用以下bitcoin-cli命令自行查看链工作值：

>bitcoin-cli getblockchaininfo
查看当前最长链的总工作量。
```
$ bitcoin-cli getblockchaininfo                                                         
{
  "chain": "main",
  "blocks": 599501,
  "headers": 599767,
  "bestblockhash": "0000000000000000000cb6141c8076e24f3a1799eef37201634ef392197668f3",
  "difficulty": 13008091666971.9,
  ...
  "chainwork": "0000000000000000000000000000000000000000094b1874d991d4e1fc51005a",
  ...
}
```

>bitcoin-cli getblock [blockhash]
查看区块链中任何块的链工作量证明。
```
$ bitcoin-cli getblock 00000000b8980ec1fe96bc1b4425788ddc88dd36699521a448ebca2020b38699
{
  "hash": "00000000b8980ec1fe96bc1b4425788ddc88dd36699521a448ebca2020b38699",
  ...
  "height": 12345,
  ...
  "bits": "1d00ffff",
  "difficulty": 1,
  "chainwork": "0000000000000000000000000000000000000000000000000000303a303a303a",
  ...
}
```

## 链接
* https://bitcoin.stackexchange.com/questions/5540/what-does-the-term-longest-chain-mean
* https://github.com/bitcoin/bitcoin/blob/56376f336548b53cf31e98a58dfb4db22cede6e5/src/chain.cpp#L122


[^1]:https://bitcoin.stackexchange.com/a/29744/24926