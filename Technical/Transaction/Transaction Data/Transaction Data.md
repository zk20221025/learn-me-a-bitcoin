# <center>交易数据</center>
![Transaction Data-1.png](img/Transaction%20Data-1-svg.png)

比特币交易只是描述比特币转移的一堆数据。

它接收[输入](../Transaction%20Data/Input/input.md)并创建新的[输出](../Transaction%20Data/output/output.md)。

## 结构
```
01000000017967a5185e907a25225574544c31f7b059c1a191d65b53dcc1554d339c4f9efc010000006a47304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a90121039b7bcd0824b9a9164f7ba098408e63e5b7e3cf90835cceb19868f54f8961a825ffffffff014baf2100000000001976a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac00000000
```
交易： [c1b4e695098210a31fe02abffe9005cffc051bbe86ff33e173155bcbdc5821e3](https://learnmeabitcoin.com/explorer/transaction/c1b4e695098210a31fe02abffe9005cffc051bbe86ff33e173155bcbdc5821e3)

## 字段

![Transaction Data-3.png](img/Transaction%20Data-3.png)

* 交易中的所有数据都是[十六进制](../../Other/Hexadecimal/hexadecimal.md)的。
* 以下图标表示数据是[**反向字节顺序**](../../Other/Little-endian/Little-Endian.md)：⟲

## 图表
比特币交易是**一系列**的[输入](../Transaction%20Data/Input/input.md)和[输出](../Transaction%20Data/output/output.md)。

![Transaction Data-2.png](img/Transaction%20Data-2%20(1).gif)

更详细地说，交易数据告诉你如何**解锁现有的比特币批次**（来自先前的交易），以及如何将它们重新锁定到新的包中。

### 注释

>原始交易有时被称为“序列化交易”，因为它只是一堆单独的数据片段组合成一个数据字符串。

>**字段大小**
比特币网络上的节点期望每个字段具有特定的长度。这种结构化格式使它们能够遍历交易数据并确定每个字段的开头和结尾。  
这就是为什么即使版本号为**1**，它也存储为**01000000**，因为比特币节点需要一个4字节大小的字段。  
>>如果你将字节数翻倍，你就能得到这个字段中字符的数量。
>>>**输入数量（和签名）和输出数量（和锁定脚本）**的长度可以变化，这就是为什么使用特殊的[VarInt](../../Other/VarInt/varint.md)字段来指定它们即将出现的大小的原因。

## 链接
* http://royalforkblog.github.io/2014/11/20/txn-demo/