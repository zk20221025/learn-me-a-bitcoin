# <center>哈希函数</center>
<center>一个将数据打乱的小程序。</center>

![hash-function-1.png](img/Hash%20Function-1-svg.png)

>**哈希函数是一种能将数据混淆并输出固定长度结果的小型计算机程序。**

哈希函数的有趣之处在于：

* 你可以将任意数量的数据输入哈希函数，它总是返回相同长度的结果。
* 结果是唯一的，因此你可以将其用作标识该数据的方法。

因此，哈希函数可以为你输入的任何数据创建**数字指纹**。

## 哈希函数的特性

好的哈希函数有**三个**重要的特性使其变得实用。

注意：[SHA256](https://learnmeabitcoin.com/tools/sha256)哈希函数是比特币中使用的主要函数，因此在下面的例子中我将使用它。

### 1.你无法从结果推断出原始数据。

加密哈希函数产生的结果是随机的（没有规律），所以无法通过哈希值（结果）反推出原始数据。

![hash-function-2.png](img/Hash%20Function-2-svg.png)

>这是加密哈希函数的特性。你也许能通过“基本”哈希函数的哈希值反推出原始数据，但加密哈希函数的任务是尽可能地使这一过程难以实现。

### 2.相同的数据总是会返回相同的结果。
哈希函数会系统地对数据进行打乱，使得相同的输入总是会产生相同的结果。

![hash-function-3.png](img/Hash%20Function-3-svg.png)

对于同一组输入数据，无论执行多少次哈希函数，得到的结果总是一样的。

例如：
```
data                sha256(data)
---------------     ----------------------------------------------------------------
learnmeabitcoin     ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2
learnmeabitcoin     ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2
learnmeabitcoin     ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2
```

### 3.不同的数据会产生不同的结果。
如果将唯一的数据输入哈希函数，哈希函数将给出唯一的结果。

![hash-function-4.png](img/Hash%20Function-4-svg.png)

即使数据发生最小的变化也会返回完全不同的结果。

例如：
```
data                sha256(data)
----------------    ----------------------------------------------------------------
learnmeabitcoin     ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2
learnmeabitcoin1    f94a840f1e1a901843a75dd07ffcc5c84478dc4f987797474c9393ac53ab55e6
learnmeabitcoin2    b9638ef00b064055b5d0b408414be02f3ab66cce752c7ac3b7595b0fffaa6567
learnmeabitcoin3    c6fd80741e150fb7ee71453fb0a2a391261f6a0d4d60759b843639e6cbae7b91
learnmeabitcoin4    255da46dc8699fffd841b7c66a31eeb4f8eda8e1ca6850c7356376518f52d3c1
```
>如果不同的数据返回相同的结果，那么这就被称为“碰撞”，在理想的哈希函数中，这种情况几乎是不可能发生的。

## 比特币中哈希函数的使用在哪里？
### 1. 交易哈希
>**将[交易数据](../../Transaction/Transaction%20Data/Transaction%20Data.md)哈希处理为[TXID](../../Transaction/TXID/TXID.md)（交易ID，交易哈希）。**

* 将长字符串的交易数据哈希处理为短且唯一的字符串，使你能够为每一笔交易创建唯一的标识符。

### 2. 区块哈希（和挖矿）
>**对[区块头](../../Block/block-header/block-header.md)进行哈希以获取[区块哈希](../../Block/block-hash/block-hash.md)。**

* 可以为每个区块创建一个唯一的ID。
* 每个哈希结果都是随机的，使得[挖矿](../../Mining/Mining.md)机制得以实现。

### 3. 地址
>**在创建比特币[地址](../../Keys/Address/Address.md)的过程中，[公钥]被(../../Keys/Public%20Key/Public%20Key.md)哈希处理（同时使用SHA256和RIPEMD160）。**

* 你不能对哈希结果进行反向推导，这有助于提高将公钥放入[锁定脚本](../../Transaction/Transaction%20Data/output/scriptPubKey/scriptPubKey.md)时的安全性。
* RIPEMD-160生成一个比公钥长度短的摘要，从而缩短了生成的地址的长度。

## 如何在比特币中哈希数据？
在比特币中，有两种主要的哈希数据方法，它们有以下名称：

1. [Hash256](#1-hash256)-双SHA-256
2. [Hash160](#2-hash160)-SHA-256之后再进行RIPEMD-160

### 1. Hash256
在比特币的系统中，一种常见的数据哈希处理方法是使用[SHA-256](https://github.com/in3rsha/sha256-animation/)算法进行两次处理，也就是“双重SHA-256”，简称为Hash256。

这是比特币中最常用的哈希数据方法。它用于在创建[TXIDs](../../Transaction/TXID/TXID.md)时哈希处理[交易数据](../../Transaction/Transaction%20Data/Transaction%20Data.md)，以及[挖掘](../../Mining/Mining.md)期间的[区块头哈希](../../Block/block-header/block-header.md)。
```ruby
require 'digest'

def hash256(hex)
  # convert hexadecimal string to byte sequence first
  binary = [hex].pack("H*") # H = hex string (highest byte first), * = multiple bytes
  
  # SHA-256 (first round)
  hash1 = Digest::SHA256.digest(binary)
  
  # SHA-256 (second round)
  hash2 = Digest::SHA256.digest(hash1)
  
  # convert from byte sequence back to hexadecimal
  hash256 = hash2.unpack("H*")[0]
  
  return hash256
end

puts hash256("aa") #=> e51600d48d2f72eb428e78733e01fbd6081b349528335bf21269362edfae185d
```

### 2. Hash160
将数据通过SHA-256哈希函数进行处理，然后将结果通过[RIPEMD-160](https://en.bitcoin.it/wiki/RIPEMD-160)哈希函数进行下一步处理。这个过程被简称为Hash160。

RIPEMD-160生成的哈希摘要（160位/20字节）比SHA-256更短，通常用在想要生成比使用Hash256更短的哈希值时使用。

它仅在创建传统[地址](../../Keys/Address/Address.md)（例如以1或3开头的地址）的过程中缩短[公钥](../../Keys/Public%20Key/Public%20Key.md)和[脚本](../../Script/Script.md)时使用。但在比特币近期的开发中，这种方法并未被使用到，因为它仅适用于需要对数据进行哈希处理的场景。

```ruby
require 'digest'

def hash160(data)
  # convert hexadecimal string to byte sequence first
  binary = [data].pack("H*") # H = hex string (highest byte first), * = multiple bytes
  
  # SHA-256
  sha256 = Digest::SHA256.digest(binary)
  
  # RIPEMD-160
  ripemd160 = Digest::RMD160.digest(sha256)
  
  # convert from byte sequence back to hexadecimal
  hash160 = ripemd160.unpack("H*").join
  
  return hash160
end

puts hash160("aa") #=> 58d179ca6112752d00dc9b89ea4f55a585195e26
```

## 常见的哈希错误
对比特币中的数据进行哈希处理时的常见错误是直接将字符串插入到哈希函数中，而不是将字符串实际代表的字节序列插入。

例如，假设我们有十六进制字符串**ab**。

如果我们直接将该字符串插入哈希函数中，编程语言实际上将字符"ab"的[ASCII编码](https://www.ascii-code.com/)输入哈希函数中，其二进制形式如下：
```
"ab" = 01100001 01100010
sha256(0110000101100010) = fb8e20fc2e4c3f248c60c39bd652f3c1347298bb977b8b4d5903b85055620603
```
我们应该将这个十六进制字符串代表的二进制字节送入哈希函数。
```
0xab = 10101011
sha256(10101011) = 087d80f7f182dd44f184aa86ca34488853ebcc04f0c60d5294919a466b463831
```
因此，在进行哈希之前，我们通常需要先将十六进制字符串转换（或“打包”）成字节。

大多数编程语言都提供了这样的功能。
```ruby
hex = "ab"
binary = [hex].pack("H*") # H = hex string (highest byte first), * = multiple bytes
```

```php
$hex = "ab"
$binary = pack("H*", $hex);
```

>如果将这些转换后的二进制值打印出来，你可能会看到一堆没有意义的文字。因为编程语言在会将此二进制数据转换回ASCII编码，但由于这些数据可能并不是文字数据，所以转换回来的结果可能会对应到ASCII表中一些非常规的、不常用的字符，因此看起来就像是一堆乱码。

请记住，哈希函数以二进制数据作为输入，因此我们需要明确我们要插入的是二进制数据。

>**最终，所有比特币数据都只是二进制的。我们只是以十六进制字符串的表示形式来方便使用。**

如果你忘记在进行哈希运算之前将十六进制字符串转换为它们对应的二进制数据，则编程语言将默认你要发送字符串中字符的二进制表示形式，这将产生与预期完全不同的哈希结果。

这是人们第一次在比特币中哈希数据时最常见的问题。因此，如果你没有得到正确的哈希结果，这可能是你的问题。请自我检查。