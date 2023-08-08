# <center>Nonce</center>
<center>用于辅助挖矿的数字。</center>

**Nonce**是[区块头](../block-header.md)中的一个字段。

我称它为“挖矿字段”。

>Nonce是“仅使用一次的数字”的缩写。
[cryptography.wikia.com](http://cryptography.wikia.com/wiki/Cryptographic_nonce)

## 它有什么用处？
在尝试将其块头的[哈希值](../../../Other/Hash%20Function/Hash%20Function.md)降低到[目标](../../../Mining/Target/Target.md)以下时，矿工会调整Nonce。（即为其候选块获得有效的[块哈希](../../block-hash/block-hash.md)。）
![nonce-1.png](img/Nonce-1.gif)
区块头中的所有其他字段都是预先确定的，因此nonce是允许你创建不同哈希结果的唯一变量。*区块404,104*。

>如果矿工可以获得低于目标的区块的哈希值，他们可以将其发送到网络中，每个人都会将其添加到他们的区块链中，矿工将因此获得区块奖励。
因此，挖矿的过程就是围绕着寻找一个幸运数字或“nonce”展开的。

## Gif
![nonce-2.png](img/Nonce-2%20(1).gif)

## 为什么它被称为“nonce”？
这只是密码学中使用的那些术语之一，就像Alice、Bob等等。

>在密码学中，nonce是一个仅能使用一次的任意数字。

我知道，我也更喜欢“挖矿字段”。

## 工具

* *哈希块头*

## 相关

* [挖矿](../../../Mining/Mining.md)