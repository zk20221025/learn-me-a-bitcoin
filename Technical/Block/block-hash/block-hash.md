# <center>区块哈希</center>
<center>区块头的哈希值。</center>

![block-hash-1.png](img/block-hash-1%20(1).png)

**区块哈希值**是[区块链](../../../Beginners/How%20Bitcoin%20Works/2.Mining/1.Blockchain/Blockchain.md)中[区块](../../../Beginners/How%20Bitcoin%20Works/2.Mining/2.Blocks/Blocks.md)的参考编号。

## 如何获得区块哈希值？
可以通过对[区块头](../block-header/block-header.md)进行两次SHA256[哈希](../../Other/Hash%20Function/Hash%20Function.md)运算来获得区块哈希值。

## 例子

这是[区块123,456](https://learnmeabitcoin.com/explorer/block/0000000000002917ED80650C6174AAC8DFC46F5FE36480AAEF682FF6CD83C3CA)的序列化区块头：
```
010000009500c43a25c624520b5100adf82cb9f9da72fd2447a496bc600b0000000000006cd862370395dedf1da2841ccda0fc489e3039de5f1ccddef0e834991a65600ea6c8cb4db3936a1ae3143991
```
![block-hash-2.png](img/block-hash-2.svg)

![block-hash-2.png](img/block-hash-2.svg)

([交换字节序](https://learnmeabitcoin.com/tools/swapendian))
```
0000000000002917ED80650C6174AAC8DFC46F5FE36480AAEF682FF6CD83C3CA
```

### 工具:
[哈希块头](https://learnmeabitcoin.com/tools/hashblockheader)

### 相关页面:
[挖矿](../../Mining/Mining.md)