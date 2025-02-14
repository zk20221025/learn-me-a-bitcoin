# <center>区块链</center>
<center>比特币交易的共享文件。</center>

## 什么是区块链？
>**区块链是一个包含有关每个比特币交易的列表的文件。**

![blockchain-1.png](img/blockchain-1.png)
  
区块链

[比特币网络](../../1.Network/Network.md)上的每个人都共享这个文件的副本，并且它会定期更新最新的交易。

![blockchain-2.png](img/blockchain-2.png)  

比特币网络：每个人共享交易文件（称为区块链）。

## 为什么区块链很重要？

>**区块链告诉你每个人拥有多少比特币。**

这是因为拥有完整的交易清单可以让你计算出每个地址上有多少比特币。

区块链就像一本日志或账本。

>**账本** - 记录企业货币交易的账簿，以借方和贷方的形式进行记录。

## 为什么叫区块链？
因为交易不是单独添加到文件中的，而是被捆绑在一起并以区块的形式添加。因此被称为**区块链**。

此外，这些区块是相互链接的，因此对链下面的区块进行的任何更改都会改变上面的区块。因此这些被链接的区块就构成了所谓的**区块链**。

![blockchain-3.png](img/blockchain-3%20(1).png)

交易被添加到区块中，并且这些区块相互链接。

交易按区块形式添加使得每个人都更容易分享区块链的副本。凭借我们现有的互联网连接速度，传输每10分钟更新一次的文件比传输每秒钟都在更新的文件更容易。

交易链是一项安全功能。它使得在没有人注意的情况下篡改区块链变得困难。

## 区块链是如何共享的？
区块链由[比特币网络](../../1.Network/Network.md)上的节点共享，就像完全合法且未受版权保护的视频可能在[BitTorrent网络](https://en.wikipedia.org/wiki/BitTorrent)上共享一样。

![blockchain-4.png](img/blockchain-4%20(1).png)  

比特币点对点区块链文件共享。如果我的文件没有最新的交易区块，有人会与我分享它们。

P2P（点对点）文件共享是一个复杂的话题，但现在只需知道区块链像BitTorrent文件一样在比特币网络上共享。

## 我可以在哪里获取区块链的副本？

你可以通过下载原始的[比特币客户端](https://bitcoin.org/en/download)，获得一份真正的、合规的、一体化的~~单轨~~区块链副本。

安装并运行该客户端后，它将连接到网络并开始下载区块链。它有180GB以上，所以需要一些时间。

>它包含了你知道的每一笔比特币交易（自2009年1月3日以来的每一笔交易），所以180GB以上是合理的。此外，完整区块链的初始下载只需要一次。之后，更新最新区块只需要很少的空间，大约每个区块1MB。

当下载完成后，你将拥有完整的区块链副本，并掌握了每一笔比特币交易的清单。此外，每次运行比特币客户端时，你都会将该文件分享给加入网络的其他人。你的一些朋友甚至可能会开始称你为“完整节点”。

通过保留区块链副本并与网络上的其他人分享，可以增强比特币网络的稳定性和安全性。

>如果你是BitTorrent的粉丝，你可以把自己想象成在**播种**区块链。每个人都喜欢播种者，因为他们是使得整个系统能够运行的关键角色。

## 我的电脑上的区块链文件存储在哪里？**
区块链存储在名字类似**blk00000.dat**的文件中。还有**blk00001.dat**、**blk00002.dat**等等。它被分成多个文件，因为这比处理一个巨大的文件更容易。

它们的位置取决于你使用的操作系统：

**Linux**
>**/home/[username]/.bitcoin/blocks/**

**windows**
>**C:\Users\[username]\AppData\Roaming\Bitcoin\***

**Mac**
>**~/Library/Application Support/Bitcoin/**

然而，这些“.dat”文件包含的数据是为计算机设计的，如果打开其中一个文件，你会看到很多无意义的信息。但请相信我，所有的交易信息都在里面。

>如果你想浏览一个可读版本的区块链，可以使用我的[区块链浏览器](https://learnmeabitcoin.com/explorer)。我从[“blk*.dat”](../../../../Technical/Blockchain/Blkdat/blkdat.md)文件中获取数据（就是你拥有的那些副本），解析它们，并在网页上显示其内容。