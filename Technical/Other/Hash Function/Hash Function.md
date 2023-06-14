# 哈希函数
一个将数据混淆的小程序。
![hash-function-1.png](img/Hash%20Function-1-svg.png)
>**哈希函数是一个小型计算机程序，它接收数据，对其进行混淆处理，并给出一个唯一的固定长度结果。**

哈希函数的有趣之处在于：

* 你可以将任意数量的数据输入哈希函数，但它总是返回相同长度的结果。
* 结果是唯一的，因此你可以将其用作标识该数据的方法。
因此，哈希函数可以把你输入的任何数据创建**数字指纹**。

## 哈希函数的特性

一个好的哈希函数具有三个重要的特性，使其有用。

注意：SHA256哈希函数是比特币中主要使用的函数，因此在我的下面的例子中我将使用它。

### 1.你无法从结果推断出原始数据。

加密哈希函数产生的结果是随机的（没有模式），因此无法通过反向计算哈希函数来确定原始数据是什么。
![hash-function-2.png](img/Hash%20Function-2-svg.png)

>这是加密哈希函数的特性。你可能能够通过“基本”哈希函数的结果重构出原始数据，但加密哈希函数的任务是尽可能地使这一过程难以实现。

### 2.相同的数据总是会返回相同的结果。
哈希函数会系统地对数据进行混淆，使得相同的输入总是会产生相同的结果。
![hash-function-3.png](img/Hash%20Function-3-svg.png)

如果你将一些数据输入到哈希函数中，你可以确信这些数据每次都会产生相同的结果。
例如：
```
data                sha256(data)
---------------     ----------------------------------------------------------------
learnmeabitcoin     ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2
learnmeabitcoin     ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2
learnmeabitcoin     ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2
```

### 3.不同的数据会产生不同的结果。
如果将唯一的数据输入哈希函数，哈希函数将给出一个唯一的结果。
![hash-function-4.png](img/Hash%20Function-4-svg.png)

即使是最小的数据变化也会返回完全不同的结果。
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
>如果不同的数据返回相同的结果，那么这就被称为“碰撞”，并且这意味着哈希函数已经失效。

## 比特币中哈希函数的使用在哪里？
### 1. 交易哈希
>**将[交易数据](../../Transaction/Transaction%20Data/Transaction%20Data.md)哈希为[TXID](../../Transaction/TXID/TXID.md)（交易ID，交易哈希）。**

* 哈希长字符串的能力可以为每个交易创建一个唯一的标识符。

### 2. 区块哈希（和挖矿）
>**将[区块头哈希](../../Block/block-header/block-header.md)为[区块哈希](../../Block/block-hash/block-hash.md)。**

* 可以为每个区块创建一个唯一的ID。
* 每个哈希结果都是随机的事实，使得[挖矿](../../Mining/Mining.md)机制成为可能。

### 3. 地址
>在创建比特币[地址](../../Keys/Address/Address.md)的过程中，使用[公钥](../../Keys/Public%20Key/Public%20Key.md)进行哈希（同时使用SHA256和RIPEMD160）。

* 哈希结果不能被逆推回去的事实，有助于当公钥放置在[锁定脚本](../../Transaction/Transaction%20Data/output/scriptPubKey/scriptPubKey.md)中时提高安全性。
* RIPEMD-160生成一个比公钥长度短的摘要，从而缩短了生成的地址的长度。

## 如何在比特币中哈希数据？
在比特币中，有两种主要的哈希数据方法，它们有以下名称：

1. Hash256-双SHA-256
2. Hash160-SHA-256后跟RIPEMD-160

### 1. Hash256
这涉及将数据通过[SHA-256](https://github.com/in3rsha/sha256-animation/)哈希函数进行处理，然后再次通过SHA-256进行处理。换句话说，这就是“双SHA-256”。我们简称为Hash256。

这是比特币中最常用的哈希数据方法。它用于在创建[TXIDs](../../Transaction/TXID/TXID.md)时哈希[交易数据](../../Transaction/Transaction%20Data/Transaction%20Data.md)，以及[挖掘](../../Mining/Mining.md)期间的[哈希块头](../../Block/block-header/block-header.md)。
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
这涉及将数据通过SHA-256哈希函数进行处理，然后将结果通过[RIPEMD-160](https://en.bitcoin.it/wiki/RIPEMD-160)哈希函数进行下一步处理。我们简称为Hash160。

RIPEMD-160生成比SHA-256更短的哈希摘要（160位/20字节），通常用在想要生成比使用Hash256更短的哈希时使用。

它仅在创建传统[地址](../../Keys/Address/Address.md)（例如以1或3开头的地址）的过程中缩短[公钥](../../Keys/Public%20Key/Public%20Key.md)和[脚本](../../Script/Script.md)时使用。它在比特币中未用于任何需要对数据进行哈希处理的最新开发中。
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

## 一个常见的哈希错误
在比特币中哈希数据的常见错误是将字符串插入到哈希函数中，而不是这些字符串实际表示的底层字节序列。

例如，假设我们有十六进制字符串ab。

如果我们直接将该字符串插入哈希函数中，你的编程语言实际上将这些字符的ASCII编码发送到哈希函数中，其二进制形式如下：
```
"ab" = 01100001 01100010
sha256(0110000101100010) = fb8e20fc2e4c3f248c60c39bd652f3c1347298bb977b8b4d5903b85055620603
```
但实际上我们想要发送到哈希函数的是这个十六进制字符串所代表的字节，在二进制中看起来像这样：
```
0xab = 10101011
sha256(10101011) = 087d80f7f182dd44f184aa86ca34488853ebcc04f0c60d5294919a466b463831
```
这就是为什么我们通常需要先将十六进制字符串“打包”成字节，然后再进行哈希处理。

大多数编程语言都会提供相应的函数来实现这一过程：
```ruby
hex = "ab"
binary = [hex].pack("H*") # H = hex string (highest byte first), * = multiple bytes
```

```php
$hex = "ab"
$binary = pack("H*", $hex);
```

>如果将这些转换后的二进制值打印出来，你可能会看到一堆术语文本。这是有道理的，因为编程语言在打印时将此二进制数据转换回ASCII，并且可能现在引用ASCII表中的奇怪代码点。

请记住，哈希函数以二进制数据作为输入，因此我们需要明确我们要插入的是二进制数据。

>**最终，所有比特币数据都只是一堆字节。我们只是偶尔使用它们的十六进制字符串表示形式来方便使用。**

如果您忘记在此之前将十六进制字符串转换为它们对应的字节，则编程语言将假定你要发送字符串中字符的二进制表示形式，这将产生与预期完全不同的哈希结果。

这是人们第一次在比特币中哈希数据时最常见的问题。因此，如果你没有得到正确的哈希结果，这可能是你的问题。请自我检查。