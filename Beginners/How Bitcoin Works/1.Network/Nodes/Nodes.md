# <center>节点</center>
<center>运行比特币程序的计算机。</center>

## 什么是节点？
节点是**运行比特币程序的计算机**。更重要的是，它与其他运行相同程序的计算机连接，以创建一个[网络](../Network.md)。

![Nodes-1.png](img/Nodes-1%20(1).png)


## 节点是做什么的？
节点有三项工作：

1. 遵循规则  
2. 分享信息  
3. 保留已确认交易的副本

### 1. 遵循规则
每个节点（比特币客户端）都经过编程来遵循一组规则。通过遵循这些规则，节点能够检查它收到的交易，并仅在一切正常时转发它们。如果有任何问题，交易就不会被传递。

![Nodes-2.png](img/Nodes-2%20(1).png)

你的节点不会转发任何可疑的交易。

例如，其中一个规则是一个人必须拥有等于或大于试图发送的比特币数量，才能发送这些比特币。因此，如果你的节点收到一个交易，其中某人试图发送超过他们所拥有的比特币数量，这个交易将不会传递给其他节点。

### 2. 分享信息

节点的主要工作是与其他节点共享信息，而节点共享的最本质的信息就是交易。

现在，节点分享的交易有两种类型：

1. 新交易——最近进入网络的交易。  
2. 已确认交易——已经被“确认”并写入文件的交易。这些交易以交易区块的形式共享，而不是单独共享。

![Nodes-3.png](img/Nodes-3%20(1).png)

节点可以共享新交易和已确认交易的区块。

现在不要担心这两者之间的差异。在[挖矿](../../2.Mining/mining.md)和[区块](../../2.Mining/2.Blocks/Blocks.md)中都介绍的很清晰明了。

### 3. 保留已确认的交易副本

如上所述，每个节点还保留已确认交易的区块。这些区块被保存在一个名为[区块链](../../2.Mining/1.Blockchain/Blockchain.md)的文件中。

![Nodes-4.png](img/Nodes-4%20(1).png)

每个节点也会保留区块链的副本。

新的交易在网络中被传播，直到它们被记录在区块链上，区块链是已确认交易的账本。

每个节点都有一份区块链副本用于安全保存，如果其他节点的副本不是最新的，则与其他节点共享。

将新交易添加到区块链的过程称为[挖矿](../../2.Mining/mining.md)。

>每个节点都是独立自主的。  
当你运行比特币客户端时，你的比特币客户端已经知道该做什么，并且做出自己的决定。  
因此，整个比特币网络由自己做出决定的节点组成，而且每个节点都做出相同的决定，这使得它成为一个完全去中心化且功能强大的网络。
>>如果其他所有节点都离线，你的节点将维护整个比特币网络。

## 必须成为一个节点才能使用比特币吗？
不是的。

你无需成为节点可以发送和接收比特币。你只需要将交易发送到比特币网络中即可。

![Nodes-5.png](img/Nodes-5%20(1).png)

如果你向一个节点发送有关交易的消息，它最终会传播到整个网络中。

如果你使用的是网页钱包，它们会代替你将你的交易输入到网络中。