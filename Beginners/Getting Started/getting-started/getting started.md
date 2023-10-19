# <center>开始入门</center>
<center>下载你的第一个钱包。</center>

> 简单的开始并不断改进比不开始要好。

所以你已经听说过比特币，而且想参与进来。那么最好的开始方式是什么呢？

让我来告诉你…

## Bitcoin Core
真正开始使用比特币的最佳方式是下载[Bitcoin Core](https://bitcoin.org/en/download)。

![getting-started-1.png](img/getting-started-1%20(1).png)

<center>Bitcoin Core是由Satoshi Nakamoto于2009年创建的原始程序。</center>

当你运行该程序时，它将连接到运行该程序的其他人，创建一个相互通信的计算机[网络](../../How%20Bitcoin%20Works/1.Network/Network.md)。

![getting-started-2.png](img/getting-started-2%20(1).png)

<center>网络上的计算机被称为节点。</center>

第一次运行Bitcoin Core时，你将开始从网络上的其他节点下载一个文件。该文件被称为[区块链](../../../Technical/Blockchain/blockchain.md)，它是一个包含**所有交易记录**的大的文件。

![getting-started-3.gif](img/getting-started-3%20(1).gif)

<center>该文件由各个区块组成，每个区块包含交易。</center>

一旦你下载并验证了完整的区块链后（目前为0.00 GB），你就可以开始进行自己的[交易](../../How%20Bitcoin%20Works/3.Transactions/Transactions.md)，这些交易会在网络中传播，并写入每个人计算机上的区块链。  

![getting-started-4.gif](img/getting-started-4%20(1).gif)

<center>区块链是交易的永久存储。</center>
  
这就是比特币的基础知识。

## 比特币钱包

运行Bitcoin Core的要求是需要**下载并存储完整的区块链**。这可以创建了一个额外的区块链副本，而且你也可以帮助将交易传递给其他计算机。

然而，并非每个人都有足够的硬盘空间来存储自己的区块链副本。

因此，你可以使用称为比特币“钱包”的东西来代替运行Bitcoin Core。钱包可以让你在不需要拥有自己的区块链副本的情况下发送和接收比特币。

![getting-started-5.png](img/getting-started-5%20(1).png)

<center>钱包与节点进行交互。</center>

如果你不想运行完整节点，比特币钱包是开始使用比特币最简单的方法。

### **使用钱包而不是Bitcoin Core的缺点是什么？**
![getting-started-0.png](img/getting-started-0%20(1).png)

* **Bitcoin Core** - 你拥有所有交易的副本。因此你可以验证收到的每笔交易并在自己的计算机上查看它们，无需信任任何其他人。
* **钱包** - 钱包需要连接到节点才能获取交易信息。因此你需要信任钱包会连接到可靠的节点以获取正确的交易详情。

使用钱包是使用比特币最方便的方法，但运行完整节点是使用比特币而无需信任任何人的方法。

选择由你。


## 应该使用哪个比特币钱包？
比特币是一个开源程序，因此任何人都可以创建自己的钱包。以下是我推荐使用的钱包：

* **桌面**：[Electrum](https://electrum.org/)
* **安卓**：[Samourai](https://samouraiwallet.com/)
* **iOS**：[Green](https://blockstream.com/green/)或[Mycelium](https://wallet.mycelium.com/)

任何人都可以创建比特币钱包，因此请确保你使用的钱包可靠（而不仅仅是好看）。

>提示：如果你不了解或不信任钱包的制作者，你就无法相信你的比特币会得到安全保护。

## 钱包是做什么的?

钱包是用于存储和管理比特币的工具。当你首次开始使用比特币钱包时，它会为你生成一个**种子**。这个种子是一个随机生成的独一无二的12-24个单词的列表。

![getting-started-6.png](img/getting-started-6%20(1).png)

<center>没有人见过你的种子。</center>

你的种子是独一无二的，它用于在你的钱包中创建**地址**。

* [地址](../../../Technical/Keys/Address/Address.md)是你提供给别人的，以便你可以接收比特币。
* 每个地址都有它自己的[私钥](../../../Technical/Keys/Private%20Key/Private%20Key.md)，当你向别人发送比特币时会使用它。

![getting-started-7.gif](img/getting-started-7%20(1).gif)

<center>你可以将地址视为你的账号，将私钥视为其密码。</center>

简而言之，**比特币钱包管理你的密钥和地址**，以便你可以发送和接收比特币。

>**提示**：如果你丢失了钱包，你可以通过种子恢复你所有的密钥（和所有的比特币）。

>**警告**：保护好你的种子，并且不要向任何人展示。如果有人得到了你的种子，他们就能够获取你的比特币。

## 比特币是如何工作的？

比特币是一个计算机网络，所有计算机一起工作来共同维护一个名为区块链的文件。

你可以将区块链想象为一个巨大的保险箱房间，其中每个保险箱都包含一定数量的比特币和一个锁。

![getting-started-8.png](img/getting-started-8%20(1).png)

<center>这些盒子被称为输出。它们可以包含任意数量的比特币，并可以在其上放置各种锁定。</center>

当你进行[交易](../../How%20Bitcoin%20Works/3.Transactions/Transactions.md)时，你的钱包会从区块链中选择属于你的一箱比特币，并为你要发送比特币的人创建一箱新的比特币。

你的钱包将另一个人的地址放入新盒子上的锁中，并使用必要的私钥解锁当前锁定在你地址上的比特币盒子。

![getting-started-9.gif](img/getting-started-9%20(1).gif)

<center>私钥和地址负责解锁和锁定。</center>

换句话说，你正在打开你的保险箱，并为其他人创建一个新的保险箱。

无论如何，这个交易（只是一些数据）会被发送到网络上的一台计算机，从一台计算机转发到另一台计算机，直到网络上的每个人都有你的交易副本。

![getting-started-10.png](img/getting-started-10%20(1).png)

<center>节点在网络中传播信息。</center>

最终，这笔交易将出现在每个人的区块链中。

这是因为网络中的一个节点会将它们收到的最新交易都收集到一个区块中，并将该区块[挖掘](../../../Technical/Mining/Mining.md)到区块链上（这需要消耗能源）。然后他们与网络上的其他节点共享这个被挖掘出的区块，并将其添加到他们的区块链中。

![getting-started-11.gif](img/getting-started-11%20(1).gif)

<center>大约每10分钟，一个新的交易块被挖掘添加到区块链上。</center>

网络上的每个节点都会使定期用最新的交易信息更新他们的区块链。这个过程会不断重复，所以区块链会不断地因为新的交易而增长。

无论如何，一旦交易进入区块链，它就无法被删除[^1]，这意味着交易完成，比特币已经转移。

这就是比特币的运作方式。

## 结论

如果你对比特币程序感到好奇并想要支持网络，那么就下载并运行Bitcoin Core
* [Bitcoin Core](https://bitcoin.org/en/download)

另一方面，如果你不关心完整节点的运行，只想发送和接收比特币，那么就直接获取一个钱包。

* **桌面**：[Electrum](https://electrum.org/)
* **安卓**：[Samourai](https://samouraiwallet.com/)
* **iOS**：[Green](https://blockstream.com/green/)或[Mycelium](https://wallet.mycelium.com/)

就我个人而言，我在家里的电脑上运行一个节点（因为我喜欢学习比特币并想要支持网络），但我在日常发送和接收比特币时使用钱包。我建议先下载一个钱包开始使用，如果你决定想要了解更多，再考虑运行节点。

>**注意**：本网站致力于与Bitcoin Core节点进行交互并了解其工作原理，但为了简单起见，本“入门指南”的其余部分将只关注如何使用钱包。

无论如何，开始使用比特币的最佳方式就是实际使用它，而要做到这一点，你需要获得你的第一个比特币……

（你会在使用过程中逐渐了解其余部分。）

## 链接
* https://bitcoin-intro.com/ -另一个使用比特币的良好介绍。此页面顶部引用的来源。 

[^1]:交易可以从区块链中移除，但这并不容易。请参见[*51%攻击*](../../../Technical/Blockchain/51-attack/51-attack.md)。
