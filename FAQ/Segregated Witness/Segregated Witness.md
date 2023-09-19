>BIP 141：[隔离见证](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki)（2015年12月21日）

# <center>什么是隔离见证？</center>

隔离见证是一项改变比特币[交易数据](../../Technical/Transaction/Transaction%20Data/Transaction%20Data.md)结构的提议。

主要原因是为了解决**交易可延展性**问题（别担心，我马上会解释这个）。然而，这种改变也带来了其他好处，比如**增加了一个区块可以容纳的交易数量**。

## 有哪些变化？

在当前的交易数据结构中，签名（“解锁”现有比特币的代码）位于每个[输入](../../Technical/Transaction/Transaction%20Data/Input/input.md)旁边，所以这个解锁代码分散在整个交易数据中。然后根据**整个交易数据**创建[TXID](../../Technical/Transaction/TXID/TXID.md)。

![Segregated Witness-1.jpg](img/Segregated%20Witness-1.png)

在隔离见证中，提议将所有的解锁代码移到交易数据的末尾。然后根据**除了解锁代码之外**的所有交易数据创建TXID，。

![Segregated Witness-2.jpg](img/Segregated%20Witness-2.png)

### 结果：
TXID只受到交易效果（比特币的移动）的影响，而不受任何用于验证交易的代码的影响（即用于解锁现有比特币以便可以使用它们的签名）。

![Segregated Witness-3.jpg](img/Segregated%20Witness-3.jpg)

所以从本质上说，你已经将交易的“验证”部分（解锁代码）与“有效”部分分离开来。

如果你将这个验证代码称为见证数据（正如密码学家可能会这样称呼），你可以说你已经“隔离了见证”。*眨眼*

## 有什么好处？

1. [修复交易可延展性](#1-修复交易可延展性)

2. [增加区块容量](#2-增加区块容量)

### 1. 修复交易可延展性
在比特币中，**交易可延展性**指的是通过改变交易中的**解锁代码来改变交易的TXID**。

![Segregated Witness-4.jpg](img/Segregated%20Witness-4.jpg)

这意味着当你将你的交易发送到网络中时，任何节点都有能力在传递它之前改变TXID。

![Segregated Witness-5.jpg](img/Segregated%20Witness-5.jpg)

最终你的交易将以与你预期不同的TXID进入区块链，这会让人有些烦恼。

然而，如果解锁代码不再是TXID的一部分，那么没有节点将能够改变你的TXID。

![Segregated Witness-6.jpg](img/Segregated%20Witness-6.jpg)

> **换句话说，隔离见证使你的TXID变得可靠**。

### 2. 增加区块容量
由于解锁代码已经被移动到交易数据中新的“见证”字段，这意味着区块大小的计算方式也可以被改变。

目前，区块大小限制是1,000,000字节（1MB）。交易以字节为单位。

![Segregated Witness-7.jpg](img/Segregated%20Witness-7.jpg)

在隔离见证中，区块大小限制不再以字节为单位衡量。相反，区块和交易被赋予一个新的度量标准，称为“权重”。

* 区块的最大权重是4,000,000。
    * 交易中的一个典型字节的权重是4。
    * 交易中的一个见证字节的权重是1。

![Segregated Witness-8.jpg](img/Segregated%20Witness-8.jpg)

所以基本上，区块大小限制乘以4，得到新的区块权重限制。然后，交易中的每个字节也乘以4，得出[交易权重](../../Technical/Transaction/Weight/Weight.md)。然而，你只需要将见证数据的字节乘以1，这基本上给了你在区块中占用空间的75%的折扣。

所以你可以说，解锁数据占用的空间只有原来的1/4，这为区块中的交易释放了更多的空间。

>这是对区块大小的增加吗？  
是的，现在每个区块的大小可以超过1,000,000字节（1MB）。  
虽然区块大小限制并没有增加到特定的字节数，但解锁数据的折扣重量意味着区块通常会超过当前的1MB限制。

>隔离见证的“区块大小增加”有多少？  
大约是**1.8MB**。  
**我是从哪里得到这个数字的？**  
目前，一个典型的交易区块大约**包含60%的解锁数据**。[^1]  
所以，如果计算一个由“典型”数量的解锁数据组成的1MB区块的权重，会得到：
```
400,000 bytes * 4 = 1,600,000 weight units
600,000 bytes * 1 =   600,000 weight units

Total Weight      = 2,200,000 weight units  
```
>现在，如果一个区块的最大重量是4,000,000权重单位，可以计算出这增加了多少：
```
4,000,000 / 2,200,000 = 1.81
```
>所以你可以说，这实际上是将区块大小限制增加到**1.8MB**。

>仅仅因为区块大小限制从1,000,000字节改变到4,000,000权重单位，并不代表隔离见证实际上将区块大小增加到4MB，因为典型的区块不会全部填充见证数据（每字节1个权重单位）。  
相反，交易将包含正常数据（4个权重单位）和见证数据（1个权重单位）。所以“区块大小增加”将根据区块中的交易组成而变化。

## 为什么要以这种方式实施？
换句话说…

>如果你想修复交易可延展性并增加区块容量，肯定有更直接的方法吧？为什么你需要重构交易数据，并创建一个新的度量标准叫“区块权重”？

这是个好问题。你是对的 - 这些改变可以更简单地实现。例如，你可以这样做：

![Segregated Witness-9.jpg](img/Segregated%20Witness-9.jpg)

然而，如果你这样做，交易和区块将在当前规则下变为“无效”。

基本上，这意味着网络上的节点会拒绝这些新的交易和区块，因为它们不符合交易和区块当前的规则。

![Segregated Witness-10.jpg](img/Segregated%20Witness-10.jpg)

例如，其中一个规则是每个区块必须1MB或更少。

因此，如果你想做出这些改变，**你需要让网络上的所有人升级他们的软件**（并且显然同意这些改变）。

因为如果你不这样做，你将最终得到一个构建两种不同区块链的网络——新节点构建新区块的区块链，旧节点继续构建旧区块的区块链。

![Segregated Witness-11.jpg](img/Segregated%20Witness-11.jpg)

这被称为硬分叉。这是可行的，但是风险很大，对于那些不升级的人可能会造成问题。

## 隔离见证如何避免这个问题？

采用隔离见证的方法，**交易和区块仍然遵循比特币网络的当前规则**，所以所有节点仍然会把隔离见证区块视为有效。因此，“旧”节点将接受这些“新”区块，并将它们添加到他们的区块链中。

![Segregated Witness-12.jpg](img/Segregated%20Witness-12.jpg)

即使旧节点不升级，也会跟上新节点的步伐。

每个人都会与区块链的单一版本保持同步。

![Segregated Witness-13.jpg](img/Segregated%20Witness-13.jpg)

>对于“旧”节点的缺点是，除非他们升级，否则他们无法利用隔离见证的新功能。只能继续正常进行“旧式”交易。

总的来说，隔离见证看起来像是一个修复交易可延展性和增加区块容量的全面方法，这种方法避免了试图让每个人升级到新软件（否则就会落后）的问题。

## 这将何时生效？
>**只有当95％的矿工表示他们已做好准备时，隔离见证才会生效**。

矿工可以通过在他们正在挖掘的区块中使用指定的版本号来表示准备就绪。

![Segregated Witness-14.jpg](img/Segregated%20Witness-14.jpg)

版本字段是[区块头](../../Technical/Block/block-header/block-header.md)的一部分。

当95%的区块有这个版本号时，隔离见证将被安排激活：

![Segregated Witness-15.jpg](img/Segregated%20Witness-15.jpg)

95%的门槛是在两个重新定向区块之间计算的。如果超过95%，软分叉将在下一个重新定向区块（大约2周后的2016个区块）被激活。

## 为什么矿工被赋予激活的决定权？
因为如果你想让软分叉成功，你会希望大多数矿工在区块链上挖掘“新”类型的区块。

这样，“新”区块的区块链将超过“旧”区块（可能仍在挖掘的未升级矿工）的区块链。因此，“新”区块链将比任何使用“旧”区块建立的区块链建立得更快，所以所有节点都将自然地采用单一的[最长链](../../Technical/Blockchain/longest-chain/longest-chain.md)。

![Segregated Witness-16.jpg](img/Segregated%20Witness-16.jpg)

拥有大多数算力能保证网络上的每个人都在同一链上。

总的来说，拥有大多数矿工的支持可以实现平稳升级，因为它确保了整个网络将汇聚在一个区块链上。

>并不是说矿工是决定隔离见证优点的最合适的群体，更多的是确保了整个网络的平稳升级。

## 是否有部署时间限制？
是的。

* 开始时间：2016年11月15日午夜
* 结束时间：2017年11月15日午夜

如果隔离见证在2017年11月15日午夜之前未激活，该提案将过期。

## 如果我不升级我的节点会发生什么？
如果你正在运行旧节点，你连接的任何隔离见证节点在将交易发送给你之前都会从交易中剥离出所有的隔离见证数据。

![Segregated Witness-17.jpg](img/Segregated%20Witness-17.jpg)

这意味着：

* 你仍然会收到与其他人相同的交易。
* 如果你收到隔离见证交易，你会看到比特币的移动，但你不会看到任何验证数据。

所以基本上，你的节点会得到“轻量级”的隔离见证交易版本。

**我该如何升级？**  
正是这种精神。

* **比特币核心** - 只需确保你正在使用[0.13.1](https://bitcoin.org/en/release/v0.13.1)或更高版本。
* **其他钱包** - 如果你正在使用不同的[钱包](../../Beginners/Getting%20Started/getting-started/getting%20started.md)，你可以通过查看这个[隔离见证采用列表](https://bitcoincore.org/en/segwit_adoption/)来看它是否准备好了。（我喜欢[Electrum](https://electrum.org/)）

## 资源
* [BIP 141（共识层）](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki)
* [BIP 144（对等服务）](https://github.com/bitcoin/bips/blob/master/bip-0144.mediawiki)

## 致谢

* Pieter Wuille；解释了[隔离见证交易数据结构](https://bitcoin.stackexchange.com/questions/49097/what-does-a-segregated-witness-transaction-look-like)（以及其他事情）。
* [Gregory Maxwell](https://github.com/gmaxwell) + [Luke-jr](https://github.com/luke-jr)；解释了[区块权重](https://www.reddit.com/r/Bitcoin/comments/5e7a8n/what_will_be_the_block_size_limit_if_segwit/)。

## 进一步阅读

* [pBitcoin.org - 隔离见证的好处](https://bitcoincore.org/en/2016/01/26/segwit-benefits/)
* [隔离见证解释[视频]](https://www.youtube.com/watch?v=DzBAG2Jp4bg) - 一个很好的关于隔离见证的视频解释，有助于理解交易数据结构的变化。

[^1]:我通过遍历blk.dat文件和加总一个区块中所有交易的scriptSig数据，然后与区块的总大小进行比较，得出这个60%的数字。我没有做过详尽的测试，但60%看起来像一个公平的平均值。例如，这里是[blk00700.dat](https://learnmeabitcoin.com/faq/files/segregated-witness/blk00700_scriptsig.txt)的结果。↩︎