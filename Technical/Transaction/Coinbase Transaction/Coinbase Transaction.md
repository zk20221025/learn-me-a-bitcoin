# Coinbase交易
用于领取区块奖励的交易。
![coinbase-transaction-1.png](img/Coinbase%20Transaction-1.svg)
一个Coinbase交易是一个区块中的第一个交易。矿工使用它来收集**区块奖**励和任何额外的*交易费用*。

这就像在一个自我地址的信封上写下你的详细信息，以便你可以领取奖金。

## 用法
当矿工创建一个*候选区块*时，第一个交易的空间被保留给Coinbase交易。
![coinbase-transaction-2.png](img/Coinbase%20Transaction-2.svg)
每个区块必须有一个coinbase交易。

## 结构
一个Coinbase交易与普通*交易数据*略有不同。主要的区别是它只有一个“空白”的*输入*，我们称之为Coinbase。
![coinbase-transaction-3.png](img/Coinbase%20Transaction-3.svg)

原始数据（*来源*）
```
01000000010000000000000000000000000000000000000000000000000000000000000000ffffffff4503ec59062f48616f4254432f53756e204368756e2059753a205a6875616e67205975616e2c2077696c6c20796f75206d61727279206d653f2f06fcc9cacc19c5f278560300ffffffff01529c6d98000000001976a914bfd3ebb5485b49a6cf1657824623ead693b5a45888ac00000000
```
* **TXID**为全零。（我们不引用任何现有交易）
* **VOUT**为全f（这个字段的*十六进制*最大值）。 （同样，因为我们不试图引用任何现有输出）。
* **scriptSig**实际上可以包含任何你喜欢的数据。[^1][^2]（因为我们不必解锁任何东西）。
  
除此之外，你只需要确保你的输出值的总和不超过你所收取的区块奖励和交易手续费。

## 备注
从BIP 34开始，scriptSig必须以块高度的推送开始。这个改变是为了*防止coinbase交易具有相同的TXID。*

**例如**。

在上面的交易数据中，03给出了被推送的*字节数*，因此接下来的3个字节ec5906表示块的高度。

当你*交换字节顺序*并转换为*十进制*时，ec5906是416236。

>**Coinbase交易中的信息。**
矿工们经常使用scriptSig来存储文本字符串。您只需要将它们解码（从十六进制到ASCII）即可阅读。
以下是一些有趣的信息：

|Coinbase交易|scriptSig（解码后）|注释；笔记|
|---|---|---|
|8b50f51b49f27e7bb0efb0b3bf38d12ce4f7e6258b90a75802a394cb585c879d|BitFury/BIP100/|矿工通常会包括他们所在的矿池的名称。|
|d0ec21e1d73d06be76c2b5b1e5ec486085bda8264229046c11b95f66f2eded83|/HaoBTC/Sun Chun Yu: Zhuang Yuan, will you marry me?/|你可以把任何文本字符串放进去。|
|4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b|The Times 03/Jan/2009 Chancellor on brink of second bailout for banks|这是由中本聪挖掘的第一个Coinbase交易。|

>**一笔 Coinbase 交易必须经过 100 个区块的确认，才能花费它的输出。**
这是一种安全措施，以防止来自 Coinbase 交易的输出因为区块链分叉而变得无法花费。
* [COINBASE_MATURITY](https://github.com/bitcoin/bitcoin/search?q=COINBASE_MATURITY)
* http://bitcoin.stackexchange.com/questions/1991/what-is-the-block-maturation-time
* http://bitcoin.stackexchange.com/questions/40655/coinbase-transactions-100-block-cooldown-period

## 链接
* http://bitcoin.stackexchange.com/questions/20721/what-is-the-format-of-the-coinbase-transaction
* https://bitcoinstrings.com/