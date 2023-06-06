# 交易数据
![Transaction Data-1.png](img/Transaction%20Data-1-svg.png)

比特币交易只是描述比特币转移的一堆数据。

它吸收[输入](../Transaction%20Data/Input/input.md)，创建新的[输出](../Transaction%20Data/output/output.md)。

结构
```
01000000017967a5185e907a25225574544c31f7b059c1a191d65b53dcc1554d339c4f9efc010000006a47304402206a2eb16b7b92051d0fa38c133e67684ed064effada1d7f925c842da401d4f22702201f196b10e6e4b4a9fff948e5c5d71ec5da53e90529c8dbd122bff2b1d21dc8a90121039b7bcd0824b9a9164f7ba098408e63e5b7e3cf90835cceb19868f54f8961a825ffffffff014baf2100000000001976a914db4d1141d0048b1ed15839d0b7a4c488cd368b0e88ac00000000
```
Transaction: [c1b4e695098210a31fe02abffe9005cffc051bbe86ff33e173155bcbdc5821e3]

## 领域
![Transaction Data-3.png](img/Transaction%20Data-3.png)

* 交易中的所有数据都是[十六进制](../../Other/Hexadecimal/hexadecimal.md)的。
* 以下图标表示数据是[反向字节顺序](../../Other/Little-endian/Little-Endian.md)：⟲

## 图表
交易基本上是**一系列**[输入](../Transaction%20Data/Input/input.md)和**一系列**[输出](../Transaction%20Data/output/output.md)。
![Transaction Data-2.png](img/Transaction%20Data-2%20(1).gif)

更详细地说，交易数据告诉您如何**解锁现有的比特币包**（来自先前的交易），以及如何将它们重新锁定到新的包中。

### 注释

>原始交易有时被称为“串行化交易”，因为它只是一堆单独的数据片段压缩成一个数据字符串。

>**字段大小**
比特币网络上的节点期望每个字段具有特定的长度。这种结构化格式使它们能够运行交易数据并确定每个字段的开头和结尾。
这就是为什么即使版本号为**1**，它也存储为**01000000**，因为比特币节点期望一个4字节大小的字段。
>>如果您将字节数翻倍，它将为您提供该字段中的字符数。
>>>输入计数（和签名）和输出计数（和锁定脚本）的长度可能不同，这就是为什么使用特殊的[VarInt](../../Other/VarInt/varint.md)字段来指定它们即将到来的大小的原因。

## 链接
* http://royalforkblog.github.io/2014/11/20/txn-demo/