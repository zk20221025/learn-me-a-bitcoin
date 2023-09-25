# <center>**比特币是如何运作的**</center>

![Home-1.png](img/Home-1.png)

比特币是一种于**2009年创建的电子支付系统**。它允许你向世界上任何人发送资金，而且无需任何人的许可来创建账户。

它是作为现代金融体系的解决方案而创建的，在现代金融体系中，少数几家大银行控制着谁可以开户，哪些交易可以处理。这意味着对货币的控制是集中的，我们必须相信银行会负责任地行事。

>我们必须信任银行来保管我们的钱，并以电子方式转账，但是他们在几乎没有储备的情况下将资金借出，形成了信贷泡沫。-[中本聪](https://satoshi.nakamotoinstitute.org/posts/p2pfoundation/1/)

银行业的集中化以及由此引发的2007年金融危机激发了比特币的发展。这是一个支付系统，它的运行没有一个中央控制点。它是由中本聪匿名设计的，于[2009年1月](https://www.metzdowd.com/pipermail/cryptography/2009-January/014994.html)发布。

任何人都可以运行该程序或使用该系统。

以下是对其工作原理的简单解释。

## 什么是比特币?
比特币只是一个**计算机程序**。你可以下载并在你的计算机上运行它。

![Home-2.png](img/Home-2.png)

来吧，[试试](https://bitcoin.org/en/download)

当你运行该程序时，它将连接到其他也在运行该程序的计算机，并且它们将开始与你共享文件。这个文件被称为[区块链](../Technical/Blockchain/blockchain.md)，它是一份[交易清单](../Technical/Transaction/Transaction%20Data/Transaction%20Data.md)。

![Home-3.png](img/Home-3%20(1).png)

当一笔新的交易进入网络时，它会从一台计算机转发到另一台计算机，直到每个人都有一份交易的副本。每隔大约10分钟，网络上的一个随机计算机([节点](../Beginners/How%20Bitcoin%20Works/1.Network/Nodes/Nodes.md))将最新接收到的交易添加到区块链上，并与网络上的其他人分享更新。

![Home-4.png](img/Home-4%20(1).png)

因此，比特币程序创建了一个庞大的计算机[网络](../Beginners/How%20Bitcoin%20Works/1.Network/Network.md)，这些计算机相互通信、**共享文件并更新其中的新交易。**。

## 比特币解决了什么问题？
在比特币出现之前，已经可以在计算机网络中转发交易了。但问题是**你可以将冲突的交易插入到计算机网络中**。例如，你可以创建两笔分别花费同一数字货币的交易，并将这两笔交易同时发送到网络中。
这就是所谓的“**双重支付**”。
![Home-5.png](img/Home-5%20(1).png)

因此，如果你创建一个没有中央控制点的电子支付系统，你就会遇到计算出这些交易中哪一个是“先发生”的问题，当你有一个由所有独立行动的计算机组成的网络时，这是一件很难做到的事情。有些电脑会先收到<span style="color: green">绿色</span>的交易，有些电脑会先收到<span style="color: red">红色</span>的交易。
谁来决定哪一个是“第一”的并且应该是唯一写入文件的？
比特币通过强制节点在将其写入文件之前将其接收的所有交易保存在[内存](../Technical/Node/Memory%20Pool/Memory%20Pool.md)中来解决这个问题。然后，每隔10分钟，网络上的一个随机节点会将其内存中的交易添加到文件中。
![Home-6.png](img/Home-6%20(1).png)

更新后的文件将与网络共享，节点将接受更新文件中“正确”的交易，从其内存中删除任何冲突的交易。因此，不会将任何双重支付交易写入文件中，所有节点都可以在彼此达成一致的情况下更新其文件。
![Home-7.png](img/Home-7%20(1).png)

将交易添加到文件的过程称为[挖矿](../Technical/Mining/Mining.md)，它是一个网络范围内的竞争，不能由网络上的单个节点控制。

## 挖矿是如何工作的？
首先，每个节点会将他们收到的最新[交易](../Technical/Transaction/Transaction%20Data/Transaction%20Data.md)存储在[内存池](../Technical/Node/Memory%20Pool/Memory%20Pool.md)，这只是计算机上的临时存储空间。任何节点都可以尝试将内存池中的交易挖掘到[区块链](../Technical/Blockchain/blockchain.md)文件中。
为了做到这一点，节点将从其内存池中收集交易，并将其放入一个称为[区块](../Beginners/How%20Bitcoin%20Works/2.Mining/2.Blocks/Blocks.md)的容器中，然后使用处理能力尝试将这个交易区块添加到区块链上。

![Home-8.png](img/Home-8%20(1).png)

那么这个处理能力从哪里得来呢？要将这个区块添加到区块链中，你必须将你的交易区块输入到一个叫做[哈希函数](../Technical/Other/Hash%20Function/Hash%20Function.md)的东西中。哈希函数是一个小型计算机程序，可以接受任何数量的数据，将其混淆，然后输出一个完全随机（但唯一）的数字。

![Home-9.png](img/Home-9%20(1).png)

为了使你的区块成功添加到区块链上，该数字（[区块哈希](../Technical/Block/block-hash/block-hash.md)）必须低于[目标值](../Technical/Mining/Target/Target.md)，而目标值是网络上所有人都同意的阈值。

![Home-10.png](img/Home-10%20(1).png)

如果你的结果**区块哈希值**不低于目标值，你可以对区块内部数据进行微小调整，再次进行哈希运算。这将产生一个完全不同的数字，并希望它低于目标值。如果不是，你需要再次调整区块并重试。

![Home-11.png](img/Home-11%20(1).png)

总之，挖矿的过程利用处理能力尽可能快地执行哈希计算，以尝试成为网络上第一个获得低于目标的区块哈希的计算机。如果成功，你可以将交易块添加到区块链上并与网络的其余部分共享。
>**注意**：尽管任何人仍然可以尝试挖掘区块，但在家用计算机上这样做已不再具有竞争力。现在已经有专门设计的硬件可以尽可能快（并且尽可能高效）地进行哈希计算，这意味着挖矿现在主要是由那些有专门硬件和便宜电力的人来进行的。

## 比特币来自哪里？
作为使用处理能力尝试将新的交易块添加到区块链的激励，每个新区块都会提供一定数量的以前不存在的比特币。因此，如果你能够成功挖掘一个块，你就可以将这些新的比特币作为奖励“发送”给自己。

![Home-12.png](img/Home-12%20(1).png)

这个新比特币的奖励被称为**区块奖励**，也是为什么这个过程被称为“挖矿”的原因。

## 为什么文件被称为“区块链”？
正如我们所见，交易不是单独添加到文件中的 - 它们被收集在一起并以块的形式添加。每个新区块都建立在现有块之上，因此文件由一系列**块**构成；因此，称为[区块链](../Technical/Blockchain/blockchain.md)。

![Home-13.png](img/Home-13%20(1).png)

此外，网络上的每个节点**始终会采用它们接收到的**[最长块链](../Technical/Blockchain/longest-chain/longest-chain.md)作为“官方”版本。这意味着矿工始终会尝试在已知的最长块链的“顶端”上构建，因为任何不属于最长链的块都不会被其他节点视为有效区块。

![Home-14.png](img/Home-14%20(1).png)

因此，如果有人想要重写交易历史，他们需要重建一个更长的区块链，以创建一个新的最长链，让其他节点采用。然而，要实现这一点，这个矿工需要拥有比网络中其他矿工加起来更多的计算机处理能力。

![Home-15.png](img/Home-15%20(1).png)

因此，网络的集体努力使得任何个人都很难“超越”网络并重新编写区块链。


## 交易是如何工作的？
你可以将区块链想象成一个保险箱存储设施，我们称之为[outputs](../Technical/Transaction/Transaction%20Data/output/output.md)。这些输出只是容器，用于存放不同数量的比特币。

![Home-16.png](img/Home-16%20(1).png)

当你进行比特币[交易](../Technical/Transaction/Transaction%20Data/Transaction%20Data.md)时，你选择一些输出并将它们解锁，然后创建新的输出并对它们进行新的[锁定](../Beginners/How%20Bitcoin%20Works/3.Transactions/Outputs/Outputt%20Locks/Outputt%20Locks.md)。

![Home-17.png](img/Home-17%20(1).png)

因当你“发送”比特币给某人时，实际上是将一定数量的比特币放入一个新的保险箱中，并在上面加上一个锁，只有将要收取你“发送”的比特币的那个人能够解锁。

例如，如果我想要向你发送一些比特币，我将从区块链中选择一些我可以解锁的输出，并从它们创建成一个新的输出，只有你可以解锁。此外，如果我不想将我解锁的所有比特币都发送给你，我将创建一个额外的输出作为我的“找零”，并将其锁定给自己。

![Home-18.png](img/Home-18%20(1).png)

往后，如果你想将比特币发送给其他人，你将重复选择现有输出（你可以解锁）并从中创建新输出的过程。因此，比特币交易形成了一个类似图形的结构，比特币的移动通过一系列交易相互连接。

![Home-19.png](img/Home-19%20(1).png)

最后，当一笔交易被挖掘到区块链上时，被使用的输出（已花费）将不能在另一笔交易中使用，而新创建的输出将可在未来的交易中使用。

![Home-20.png](img/Home-20%20(1).png)


## 你如何拥有比特币？

要能够“接收”比特币，你需要拥有自己的一组[密钥](../Beginners/How%20Bitcoin%20Works/4.Keys%26Addresses/keys_addresses.md)。这组密钥就像你的账号号码和密码，但在比特币中，它们被称为你的[公钥](../Technical/Keys/Public%20Key/Public%20Key.md)和[私钥](../Technical/Keys/Private%20Key/Private%20Key.md)。

例如，如果我想给你发送一些比特币，你首先需要给我你的**公钥**。当我创建交易时，我会把你的公钥放在输出（保险箱）上的锁里。当你想把比特币发送给其他人时，你需要使用你的**私钥**来解锁这个输出。

![Home-21.png](img/Home-21%20(1).png)


那么你在哪里可以获取公钥和私钥呢？好吧，借助**密码学**，你实际上可以自己生成它们。

简而言之，你的私钥只是一个大的随机数，而你的公钥是从这个私钥计算出来的数字。但聪明的部分是，你可以把你的公钥给别人，但他们无法从中计算出私钥。

![Home-22.png](img/Home-22%20(1).png)

当你想解锁分配给你公钥的比特币时，你使用你的私钥创建所谓的[数字签名](../Beginners/How%20Bitcoin%20Works/4.Keys%26Addresses/Public%20keys/Digital%20Signatures/Digital%20Signatures.md)。这个数字签名证明你是公钥的所有者（因此可以解锁比特币），而不必透露你的私钥。这个数字签名也只对它创建的交易有效，因此不能用来解锁锁定在同一公钥下的其他比特币。

![Home-23.png](img/Home-23%20(1).png)

这个系统被称为“公钥密码学”，自1978[^1]年以来就已经存在。比特币利用这个系统，允许任何人创建用于安全发送和接收比特币的密钥，无需中央机构发放账户和密码。

## 所有内容放在一起
[开始使用比特币](../Beginners/Getting%20Started/getting-started/getting%20started.md)，你需要生成自己的[私钥](../Technical/Keys/Private%20Key/Private%20Key.md)和[公钥](../Technical/Keys/Private%20Key/Private%20Key.md)。你的私钥只是一个非常大的随机数，你的公钥是从它计算出来的。这些密钥可以在计算机上轻松生成，甚至可以在简单的计算器上生成。大多数人使用[比特币钱包](https://electrum.org/)来帮助生成和管理他们的密钥。

要接收比特币，你需要将你的公钥提供给想要向你发送比特币的人。这个人将创建一笔[交易](../Technical/Transaction/Transaction%20Data/Transaction%20Data.md)，在其中解锁他拥有的比特币，并创建一个新的“保险箱”，将你的公钥放在锁内。

这个交易被发送到比特币网络上的任何一个[节点](../Beginners/How%20Bitcoin%20Works/1.Network/Network.md)，然后在计算机之间传递，直到网络上[节点网络](../Beginners/How%20Bitcoin%20Works/1.Network/Network.md) 上的每个节点都有交易的副本。从这里开始，每个节点都有机会尝试将他们收到的最新交易挖掘到区块链上。

这个[挖掘](../Technical/Mining/Mining.md)过程涉及一个节点从其[内存池](../Technical/Node/Memory%20Pool/Memory%20Pool.md)中收集交易到一个[区块](../Beginners/How%20Bitcoin%20Works/2.Mining/2.Blocks/Blocks.md)中，然后反复将该块数据通过[哈希函数](../Technical/Other/Hash%20Function/Hash%20Function.md)（每次进行微小的调整）以尝试获得低于[目标值](../Technical/Mining/Target/Target.md)的[区块哈希](../Technical/Block/block-hash/block-hash.md)。

第一个找到低于目标哈希值的矿工将把该块添加到他们的[区块链](../Beginners/How%20Bitcoin%20Works/2.Mining/1.Blockchain/Blockchain.md)中，并将此块广播给网络上的其他节点。每个节点也会将该块添加到他们的区块链中（从内存池中删除任何冲突的交易），并重新启动挖矿过程，以尝试在该链中基于这个新区块构建。

最后，挖掘出这个区块的矿工会在区块内放置自己的[特殊交易](../Technical/Transaction/Coinbase%20Transaction/Coinbase%20Transaction.md)，允许他们收集一定数量的比特币，这些比特币之前不存在。这个**区块奖励**作为节点继续构建区块链的激励，同时将新币分配到比特币网络中。

![Home-24.png](img/Home-24%20(1).png)

## 结论
比特币是一个计算机程序，它与世界各地的其他计算机共享一个安全文件。这个安全文件由交易组成，这些交易使用密码学人们发送和接收数字保险箱。因此，这创造了一个电子支付系统，任何人都可以使用，而且去中心化。

比特币网络自2009年1月发布以来一直运行顺畅。2019年，比特币网络处理了超过**1.12亿笔交易**，总共移动了**15.58万亿美元**[^2]。

比特币程序本身也在积极开发中，自发布以来已有超过**600**人为代码做出了贡献[^3]。这是因为该软件是“开源”的，这意味着任何人都可以查看代码并为其改进做出贡献。

https://bitcoin.org/bitcoin.pdf (白皮书)

https://github.com/bitcoin/bitcoin/ (源代码)

## 想要了解更多吗？
好的，你来对地方了。

这个网站充满了有关比特币如何运作的**简单解释**。

1. [初学者指南](../Beginners/Beginners%20Guide.md) - 有时候你只需要一个完整的基础教程。这是我在2015年第一次学习比特币运作时写的最简短、最简单的指南。
2. [技术指南](../Technical/Technical.md) - 更全面、更深入的比特币运作指南。适合程序员。
3. *区块链浏览器* - 通过浏览数据并查看它们如何相互连接，你可以感受比特币的运作方式。就像打开汽车引擎盖看里面的一样。
4. [视频(YouTube)](https://www.youtube.com/channel/UCj9MFr-7a02d_qe4xVnZ1sA/videos)- 这些是程序员从比特币的角度深入解释其机制的视频。如果你想用比特币编写东西，这些视频课程会让你入门。
5. [代码(GitHub)](https://github.com/in3rsha/learnmeabitcoin-code)- 常见比特币操作的示例代码片段。

## 为什么我应该相信你？
虽然我没有比特币的官方资格，但我阅读了很多代码，[编写了很多代码](https://github.com/in3rsha)，[并提出了很多问题](https://bitcoin.stackexchange.com/users/24926/inersha)。我所知道的关于比特币的一切都来自实践。

另外，[我很酷](../About/about.md)

## 为什么所有这些信息都是免费的？

因为：

* 比特币是一个[开源程序](https://github.com/bitcoin/bitcoin)，你可以免费运行它。
* 我所学到的关于[比特币](https://bitcoin.stackexchange.com/)、编程和[写作](https://www.pokerseo.org/about/)的一切都是免费的。
* 这个网站是完全由免费的[开源工具](https://learnmeabitcoin.com/thanks#development)构建的。
那么为什么不免费教育呢？

尽管如此，还是非常感谢您的捐赠：[3Beer3irc1vgs76ENA4coqsEQpGZeM5CTd](bitcoin:3Beer3irc1vgs76ENA4coqsEQpGZeM5CTd)

## 为什么你做这个网站？

因为我想让其他人也能理解比特币的工作原理。

比特币可以让你向世界上任何人转移价值，我认为这很重要。如果你理解了比特币的工作原理，你可以创建自己的酷炫软件，产生积极的影响。

[^1]:https://en.wikipedia.org/wiki/RSA_(cryptosystem)
[^2]:2020 年转移了 536,916,338 比特币。2020 年 12 月 31 日的汇率为 29,013.39 美元/BTC。
[^3]:https://github.com/bitcoin/bitcoin/graphs/contributors