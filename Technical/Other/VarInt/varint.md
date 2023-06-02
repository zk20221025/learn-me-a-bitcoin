VarInt
一种指示即将到来的数据大小的格式。
![varint-1.png](img/varint-1-svg.png)

可变整数（VarInt）是用于表示即将到来的字段数量或即将到来的字段长度的*交易数据*中的字段。

示例：
以下是一个*交易*，其中VarInt字段已突出显示（如果它是指字段长度，则也突出显示该字段的长度）：
```
01000000017b9c26b765a24997e0f855e5d25e86e6816b213e2bbc67bc918df239bfc20158040000006a47304402200aa5891780e216bf1941b502de29890834a2584eb576657e340d1fa95f2c0268022010712e05b30bfa9a9aaa146927fce1819f2ec6d118d25946256770541a8117b6012103d2305c392cbd5ac36b54d3f23f7305ee024e25000f5277a8c065e12df5035926ffffffff028555a700000000001976a914aca504fd373f5f3ba2774a3643d714d6419463bc88ac9bc0ba01000000001976a9143bbebbd7a3414f9e5afebe79b3b408bada63cde288ac00000000
```

## 结构
VarInt通常是一个1字节的*十六进制*值：
```
                 0x6a = 106 bytes
--|------------------------------------- ... --|
6a47304402200aa5891780e216bf1941b502de29 ... 926
```

然而，如果VarInt大于0xfc（所以你要表达的数字不能适应两个十六进制字符），那么你可以按照以下方式扩展字段：

|大小	|示例	|描述|
|---|---|---|
|<= 0xfc|	12|---|	
|<= 0xffff	|fd1234|	Prefix with fd, and the next 2 bytes is the VarInt (in little-endian).|
|<= 0xffffffff|	fe12345678|	Prefix with fe, and the next 4 bytes is the VarInt (in little-endian).|
|<= 0xffffffffffffffff|	ff1234567890abcdef|	Prefix with ff, and the next 8 bytes is the VarInt (in little-endian).|

>1字节= 2个字符

## 例子：

### 1. VarInt = 6a

没有前缀fd、fe或ff，所以我们知道这1个字节是VarInt值，并且我们可以直接将十六进制转换为十进制：
```
Varint = 6a
       = 106 bytes
```
*hexdec*

### 2. VarInt = fd2602
前缀是fd，因此我们知道接下来的2个字节（按小端字节序）将提供即将到来的字段的大小：
```
VarInt = fd2602
       =   2602    (next 2 bytes)
       =   0226    ([swap endian](/tools/swapendian))
       =   550 bytes
```
*Swap Endian*

### 3. Varint = fe703a0f00
与上一个例子类似，除了fe表示接下来的4个字节：
```
VarInt = fe703a0f00
VarInt =   703a0f00
VarInt =   000f3a70
       =   998000 bytes

```

## 为什么要使用VarInts？

在*交易数据*中，像txid和vout这样的字段具有固定的大小，因此您始终知道它们的起始和结束位置。然而，像scriptSig这样的字段长度可能会有所不同，因此在它之前放置一个VarInt字段，以便您知道它有多少个字节的长度。

如果您编写了一个读取交易数据的脚本或程序，这些VarInts是必不可少的，因为没有它们，您就不会知道可变长度字段的结束位置。

## 资源
* https://bitcoin.org/en/developer-reference#compactsize-unsigned-integers