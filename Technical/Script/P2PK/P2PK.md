# <center>P2PK</center>
<center>支付到公钥

![P2PK-1.png](img/P2PK-1%20(1).png)</center>

**P2PK**（支付到公钥）是一种[脚本](../Script.md)模式，将[输出](../../Transaction/Transaction%20Data/output/output.md)锁定到[公钥](../../Keys/Public%20Key/Public%20Key.md)。

它是更常用的[P2PKH](../P2PKH/P2PKH.md)锁定脚本的简化版本。

## P2PK如何工作？
一个P2PK锁定只包含一个[公钥](../../Keys/Public%20Key/Public%20Key.md)和一个**CHECKSIG**操作码：
|scriptPubKey（脚本公钥）|04ae1a62fe09c5f51b13905f07f06b99a2f7159b2225f374cd378d71302fa28414e7aab37397f554a7df5f142c21c1b7303b8a0626f1baded5c72a704f7e6cd84c OP_CHECKSIG|
|---|---|

要解锁它，你只需要提供有效的签名：
|scriptSig（脚本签名）|30440220576497b7e6f9b553c0aba0d8929432550e092db9c130aae37b84b545e7f4a36c022066cb982ed80608372c139d7bb9af335423d5280350fe3e06bd510e695480914f01|
|---|---|

当脚本运行时，**CHECKSIG**操作码将**签名**与**公钥**进行比较，如果有效，则将**1**推送到堆栈上。

![P2PK-2.png](img/P2PK-2%20(1).gif)

## 你在哪里可以找到P2PK脚本？
尽管P2PK是将比特币锁定到某人公钥的最简单脚本，**但它并没有像类似但更复杂的[P2PKH](../P2PKH/P2PKH.md)脚本一样被广泛使用**。

你最常见的**P2PK在区块链中较早的块中的[coinbase交易](../../Transaction/Coinbase%20Transaction/Coinbase%20Transaction.md)**中找到。这是因为原始的[比特币核心](https://bitcoin.org/en/download)矿工在构建[候选块](../../Node/Candidate%20Block/Candidate%20Block.md)时会使用P2PK进行块奖励:[^1]
![P2PK-3.png](img/P2PK-3%20(1).png)
<center>新的挖矿软件通常使用P2PKH，因此现在很少在coinbase交易中看到P2PK脚本。</center>

以下是一些使用P2PK的交易示例：

* 4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b-创世块中的coinbase交易。（2009年1月3日）
* f4184fc596403b9d638783cf57adfe4c75c605f6356fbc91338530e9831e9e16-从Satoshi到 Hal Finney 的第一笔比特币交易实际上使用了 P2PK（区块 170，2009 年 1 月 12 日）

## 为什么Satoshi在比特币核心矿工中使用P2PK？

不确定。你得问问她/他。

我认为**P2PKH**被用作方便人们进行交易的方式，因为它允许你使用[地址](../../Keys/Address/Address.md)而不是传递你的公钥（长度更长）。然而，矿工不需要地址的便利，因此P2PK成为了更简单的选择。

## 为什么我们不经常使用P2PK？

因为Satoshi设计了[P2PKH](../P2PKH/P2PKH.md)，以便我们可以互相发送较短的**地址**而不是完整的公钥。

参见：[为什么我们既有P2PKH又有P2PK？](../P2PKH/P2PKH.md)

[^1]:https://bitcoin.stackexchange.com/questions/73563/how-did-pay-to-pubkey-hash-come-about-what-is-its-history