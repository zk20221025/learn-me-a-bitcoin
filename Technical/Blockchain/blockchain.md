# <center>区块链</center>
<center>比特币交易记录文件。</center>

![blockchain-S-1.png](img/blockchain-S-1%20(1).png)

区块链是一个[交易](../../Beginners/How%20Bitcoin%20Works/3.Transactions/Transactions.md)记录文件。它是比特币[节点](../../Beginners/How%20Bitcoin%20Works/1.Network/Nodes/Nodes.md)维护的最重要的文件。

新的交易以[区块](../../Beginners/How%20Bitcoin%20Works/2.Mining/2.Blocks/Blocks.md)的形式添加到文件中，这些区块相互连接堆叠形成一个链。因此被叫做区块链。

区块链是**比特币交易文件的永久存储方式**。

![blockchain-S-2.png](img/blockchain-S-2%20(1).png)

## 如何获取区块链的副本？

![blockchain-S-3.png](img/blockchain-S-3%20(1).png)

当你运行比特币软件时，你的节点会从网络上的其他节点下载区块链数据，直到拥有最新的区块链副本。

当节点相互连接时，它们在初次数据交换时告诉彼此它们的区块链高度（拥有多少个区块）。如果其他节点的区块比你多，你的节点将从其他节点请求这些区块，直到你拥有完整的区块链副本。

节点不断地相互通信，确保网络上每台电脑的区块链数据都是最新的。

>**提示**：没有所谓的唯一或者权威的区块链版本。每个节点都保留自己的本地区块链副本，而这些副本可能在不同的时间点，因为计算机间数据更新的差异而有所不同。

>**注意**：我的节点的[区块链](./blockchain.md)目前为**0.00 GB**，因此第一次运行比特币软件时可能需要一段时间下载。这被称为[初始区块下载](https://btcinformation.org/en/developer-guide#initial-block-download)（或IBD）。

## 如何将新区块添加到区块链？

![blockchain-S-4.png](img/blockchain-S-4%20(1).png)

新的交易块必须先被[挖掘](../Mining/Mining.md),然后才能添加到区块链中。

简而言之，挖掘的过程包括从[内存池](../Node/Memory%20Pool/Memory%20Pool.md)中收集交易到[候选区块](../Node/Candidate%20Block/Candidate%20Block.md)，然后利用处理能力生成一个低于特定的[目标值](../Mining/Target/Target.md)的[区块哈希值](../Block/block-hash/block-hash.md)。这意味着网络上的任何节点都可以挖掘新区块，但需要消耗能源才能做到。

当节点（或“矿工”）成功挖掘出新区块时，它们将与网络上的其他节点共享。其他节点会将把它添加到它们的区块链中，然后开始尝试在该区块上面挖掘新的区块。

![blockchain-S-5.png](img/blockchain-S-5%20(1).png)

矿工不断努力用新的交易区块来扩展区块链。

>**注意**：由于挖掘区块所需的[难度](../../Beginners/How%20Bitcoin%20Works/2.Mining/3.Difficulty/Difficulty.md)和处理能力，平均每10分钟才会添加一个新的区块到区块链中。

>**注意**：节点并不一定要挖掘新的区块，它也可以只保存区块链的副本，并在收到新区块时将其传播到其他节点。

## 两个区块能同时被挖掘吗？

![blockchain-S-7.png](img/blockchain-S-7%20(1).png)

是的，这是完全正常的。

在这种情况下，节点会将他们首先接收到的区块视为他们的区块链的一部分，但也会保留他们接收到的第二个区块以备不时之需。然而，第二个到达的区块（及其内部的交易）不会被视为他们**活动**区块链的一部分。

网络上的节点会就哪个区块应该处于链顶部而存在分歧。

这种分歧在下一个区块被挖掘时得以解决。下一个区块将在其中一个区块之上构建，创建一个新的[最长区块链](./longest-chain/longest-chain.md)，根据规则，**节点将始终采用已知的最长区块链作为其活动区块链**。

一些节点将执行[链重组](./chain-reorganisation/chain-reorganisation.md)，将旧活动链中的区块移出，然后使用新的更长的链中的区块。

尽管在某一时刻可能存在分歧，但通过持续的区块挖掘和最长链的采纳，所有的节点最终都会达成一致。

![blockchain-S-8.png](img/blockchain-S-8%20(1).png)

<center>节点始终采用最长的区块链作为区块链的可接受版本。</center>

## 这是否意味着区块链中的块可以被替换？
是的。

由于节点始终采用最长的链，因此你始终可以尝试构建一个新的更长的区块链以替换现有的区块链，并且网络上的每个节点都将采用它。实际上，这将允许你从区块链中“撤消”或逆转比特币交易。

![blockchain-S-9.png](img/blockchain-S-9%20(1).png)

<center>如果你建立了一个新的最长的区块链，其他节点将会采用它。</center>

然而，问题在于所有矿工都被激励始终在最长链上进行构建。这意味着网络上的矿工的综合处理能力将集中在构建一个链上，这条链将比任何你自己构建的链要建得更快。

![blockchain-S-10.png](img/blockchain-S-10%20(1).png)

<center>其他所有矿工都在努力扩展当前最长的区块链。</center>

网络中的处理能力被用于构建区块链，以保护已经被挖掘到区块链上的区块和交易。

为此，如果你想要进行有意的链重组（以“撤消”现有区块的交易），你需要拥有比其他所有矿工加起来更多的处理能力（算力），这样你就可以超过网络并建立一个更长的链让所有人接受。

这被称为[“51% Attack”](./51-attack/51-attack.md)。

## 区块链在哪里？
区块链文件通常可以在以下位置找到：

* **Linux**：**~/.bitcoin/blocks/**
* **Mac**：**~/Library/Application Support/Bitcoin/**
* **Windows**：C:\Users\[用户名]\AppData\Roaming\Bitcoin\
>**注意**：区块链被分成多个文件，命名为**blk00000.dat**，**blk00001.dat**，**blk00002.dat**等等。这是因为用较小的文件比用一个巨大的文件更容易处理[^1]。

### 摘要
区块链是比特币交易的永久存储文件。新的交易以区块的形式被添加到文件中，这些区块会相互叠加形成链。

新的区块通过挖矿添加到区块链中，这需要使用计算机处理能力。这意味着挖矿需要消耗能源，网络中的任何节点都可以尝试添加下一个区块到链上。

当新的区块被挖出来后，它会在网络中传播，节点会验证并将其添加到他们的链上。通过这种方式，区块链是一份不断增长的交易记录账本，分布在网络上的多台计算机上。

节点总是采用最长的区块链作为活动版本，这解决了关于哪些区块应该在链的顶部的争议。这也保护了已经存在于区块链中的区块，因为建立一个替换链中现有区块的链需要大量的能源。

区块链是一个定期更新的交易文件。通过挖掘新的区块并总是选择最长的区块链，可以**确保网络中的所有计算机都对当前有效的区块和交易达成共识**，同时也使得任何人难以对文件中的区块和交易进行历史性的更改。

![blockchain-S-11.png](img/blockchain-S-11%20(1).png)(https://learnmeabitcoin.com/technical/images/blockchain/blockchain.gif)

点击图像查看区块链建设的gif动画。

[^1]:https://bitcoin.stackexchange.com/questions/50693/why-are-blk-dat-files-134200000-bytes