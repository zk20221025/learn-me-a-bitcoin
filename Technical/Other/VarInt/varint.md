# <center>VarInt</center>
<center>指示即将到来数据大小的格式。</center>

![varint-1.png](img/varint-1-svg.png)

VarInt（可变整数）是一种数据表示格式，在[交易数据](../../Transaction/Transaction%20Data/Transaction%20Data.md)中用于指示即将到来的**字段的数量**或**字段的长度**。

示例：  
以下是一个[交易](https://learnmeabitcoin.com/explorer/transaction/2dc4031a55c38ba93d74fb6b7d881f930b78f389a3bc548acc2fd18c532b3907)，其中<span style="color: yellow">VarInt</span>字段已突出显示（如果它是指字段长度，则也突出显示**该字段的长度**）：
```
01000000017b9c26b765a24997e0f855e5d25e86e6816b213e2bbc67bc918df239bfc20158040000006a47304402200aa5891780e216bf1941b502de29890834a2584eb576657e340d1fa95f2c0268022010712e05b30bfa9a9aaa146927fce1819f2ec6d118d25946256770541a8117b6012103d2305c392cbd5ac36b54d3f23f7305ee024e25000f5277a8c065e12df5035926ffffffff028555a700000000001976a914aca504fd373f5f3ba2774a3643d714d6419463bc88ac9bc0ba01000000001976a9143bbebbd7a3414f9e5afebe79b3b408bada63cde288ac00000000
```

## 结构
VarInt通常是一个1字节的[十六进制](../Hexadecimal/hexadecimal.md)值：
```
                 0x6a = 106 bytes
--|------------------------------------- ... --|
6a47304402200aa5891780e216bf1941b502de29 ... 926
```

然而，如果你需要表示的数字大于**0xfc**（即无法容纳在两个十六进制字符内），那么可以按照以下方式扩展字段：

|大小	|示例	|描述|
|---|---|---|
|<= **0xfc**|	**12**|---|	
|<= **0xffff**	|**fd1234**|	以**fd**为前缀，接下来的2个字节是VarInt (in little-endian).|
|<= **0xffffffff**|	**fe12345678**|	以**fe**为前缀，接下来的4个字节是VarInt(in little-endian).|
|<= **0xffffffffffffffff**|	**ff1234567890abcdef**|	以**ff**为前缀，接下来的8个字节是VarInt (in little-endian).|

>1字节= 2个字符

## 例子：

### 1. VarInt = 6a

没有前缀**fd**、**fe**或**ff**，所以这1个字节是VarInt值，并且可以直接将十六进制转换为十进制：
```
Varint = 6a
       = 106 bytes
```
>[十六进制](https://learnmeabitcoin.com/tools/hexdec/)

### 2. VarInt = fd2602
前缀是**fd**，因此接下来的2个字节（按小端字节序）将提供即将到来的字段的大小：
```
VarInt = fd2602
       =   2602    (next 2 bytes)
       =   0226    ([swap endian](/tools/swapendian))
       =   550 bytes
```
>[交换节字序](https://learnmeabitcoin.com/tools/swapendian)

### 3. Varint = fe703a0f00
与上一个例子类似，只是**fe**表示接下来的4个字节：
```
VarInt = fe703a0f00
VarInt =   703a0f00
VarInt =   000f3a70
       =   998000 bytes
```

## 为什么要使用VarInts？

在[交易数据](../../Transaction/Transaction%20Data/Transaction%20Data.md)中，像[txid](../../Transaction/TXID/TXID.md)和[vout](../../Other/VOUT/VOUT.md)这样的字段具有固定的大小，因此始终知道它们的起始位置和结束位置。但是，有些字段像scriptSig的长度可以变化，因此在它之前放置一个VarInt字段，以便知道这个字段的长度是多少。

如果你编写了读取交易数据的脚本或程序，这些VarInt是必不可少的，因为如果没有它们，你将不知道可变长度字段在哪里结束。

## 资源
* https://bitcoin.org/en/developer-reference#compactsize-unsigned-integers