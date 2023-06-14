# 交易
选择一批比特币，从中创建新的比特币交易。

## 什么是比特币交易？

>**比特币交易是一组数据**。

这些数据包含有关发送**金额**、**发送**账户和**接收**账户的信息。
![transactions-1.png](img/Transactions-1%20(1).png)

这是基本信息，因此可以很容易地用一行数据表示:
![transactions-2.png](img/Transactions-2%20(1).png)

当你进行交易时，你只需要将交易数据发送到[比特币网络](../1.Network/Network.md)。
![transactions-3.png](img/Transactions-3%20(1).png)

最终，网络上的一个节点会将你的交易[挖掘](../2.Mining/mining.md)到一个[区块](../2.Mining/2.Blocks/Blocks.md)中，而这个区块（包含你的交易）将被添加到已确认交易的文件（区块链）中。
![transactions-4.png](img/Transactions-4%20(1).png)

这就是比特币交易的全部内容——将一行简单的数据输入比特币网络，并等待它被挖掘到[区块链](../2.Mining/1.Blockchain/Blockchain.md)中。

## 比特币交易是如何工作的？
一个比特币地址就像是一个账户号码，用来存放比特币。

但是当你进行交易时，它并不像是从一个罐子里取出确切数量的硬币并将它们移动到另一个罐子中那样简单。
![transactions-5.png](img/Transactions-5%20(1).png)

相反，一个地址会跟踪它所接收的每笔交易。
![transactions-6.png](img/Transactions-6%20(1).png)

因此，当你想向其他人发送比特币时，你会获取已经收到的整个金额，并使用它们向新地址发送新金额：
![transactions-7.png](img/Transactions-7%20(1).png)

当其他人想要向另一个人发送比特币时，他们将以同样的方式使用他们收到的全部金额：
![transactions-8.png](img/Transactions-8%20(1).png)

因此，实际上你会批量收到比特币，并使用这些批次创建新批次以发送给其他人。

这就是交易的工作方式。

**稍等一下...**

“如果批次的总数超过了我想要发送的数量怎么办？”

很好的问题先生。在这种情况下（通常是这样），你只需向交易添加另一个输出，并将差额发送回自己：
![transactions-9.png](img/Transactions-9%20(1).png)

这可能一开始看起来有些尴尬，我知道，但从编程的角度来看，这是一种精确的方法，所以要适应它。

总之…

## 总结
1. 你有一个比特币地址。比特币以批次形式到达该地址，称为输出。
2. 比特币交易是使用这些输出（作为交易中的输入）创建属于其他人地址的新输出的过程。
3. 所有这些都可以用一行数据表示。
![transactions-10.png](img/Transactions-10%20(1).png)

>要了解这种输出系统是如何工作的，请查看[输出](../3.Transactions/Outputs/Outputs.md)。

## 怎么防止其他人花费我的比特币？
换句话说...

“如果进行交易只是将一行数据输入比特币网络，为什么不能有人构建一个包含我的地址并使用它将比特币发送到他们的地址的交易呢？”

**答案**：因为每个交易输出都有一个锁...
![transactions-11.png](img/Transactions-11%20(1).png)

如果你在未解锁这些输出的情况下创建交易，比特币网络上的节点将拒绝该交易。
![transactions-12.png](img/Transactions-12%20(1).png)

但幸运的是，你的比特币地址附带有私钥。
![transactions-13.png](img/Transactions-13%20(1).png)

如果你想发送比特币，你需要使用此私钥解锁位于你地址上的输出。
![transactions-14.png](img/Transactions-14%20(1).png)

在解锁你想要使用的所有输出后，该交易将被比特币网络上的节点接受并传播。
![transactions-15.png](img/Transactions-15%20(1).png)
这就是比特币交易的工作原理。