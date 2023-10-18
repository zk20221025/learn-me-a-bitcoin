# <center>地址</center>
<center>一个易于分享的锁定脚本格式。</center>

![address-1.png](img/address-1%20(1).png)

**地址**是你提供给人们的信息，以便他们可以“发送”比特币给你。

当有人收到它时，他们可以根据你提供的地址类型创建特定的[锁定脚本](../../Transaction/Transaction%20Data/output/scriptPubKey/scriptPubKey.md)。

## 如何创建地址？

这取决于你如何设置你的比特币保护方式。

一般来说，一个地址包含：

1. 一些你想包含在锁定中的**特定数据**。例如，你的[公钥哈希](../Public%20Key/Public%20Key%20Hash/public-key-hash.md)。
2. [**前缀**](#前缀)，表示要创建什么类型的锁定。
3. 还包含[**校验和**](../Checksum/Checksum.md)，用于帮助检查任何输入错误。

最后，所有这些都被转换为[**Base58**](../Base58/Base58.md)，这使它更易于使用。

## Pay To Pubkey Hash（P2PKH）
[**P2PKH**](../../Script/P2PKH/P2PKH.md)这是一个传统的将比特币锁定到[**公钥**](../Public%20Key/Public%20Key.md)（或更准确的说法：锁定到[**公钥哈希**](../Public%20Key/Public%20Key%20Hash/public-key-hash.md)）的地址。

如上所述，在哈希公钥前后分别添加**前缀**和**校验和**，然后将其全部编码为base58。

![address-2.png](img/address-2%20(1).png)

现在我们有一个比特币地址可以提供给别人。 

### 解码：
当有人从这个地址创建锁定脚本时，他们只需将地址的base58编码解码以**检索其中的hash160**，然后**创建一个P2PKH锁定脚本**，如下所示：

![address-3.png](img/address-3%20(1).png)

**前缀**指示了要创建什么类型的锁定，而**hash160**则告诉他们需要在锁定脚本中放入什么内容。

## Pay To Script Hash（P2SH）
[**P2SH**](../../Script/P2SH/P2SH.md)：这个锁定包括[脚本](../../Script/Script.md)的哈希值。我们在解锁比特币时，会提供这个脚本的实际内容。这样，我们就可以创建复杂的锁定脚本，而无需让其他人了解这些脚本的详细内容。

与之前相同，只是这次包含了脚本的哈希，并使用**前缀05**来表示这时一个P2SH地址。

![address-4.png](img/address-4%20(1).png)

### 解码后：
这就是P2SH：

![address-5.png](img/address-5%20(1).png)

## 前缀
如前所述，**你使用的前缀将指示要创建的锁定脚本类型**。

以下是常见地址前缀的列表：

在比特币中，在将数据转换为base58之前，会添加不同的前缀以影响结果的首字符。然后，这个首字符帮助我们识别每个base58字符串代表的内容。

以下是比特币中使用的最常见前缀：

主网:
|前缀（十六进制）|	Base58前导字符|	表示|	示例|
|---|---|---|---|
|**00**	|1	|[P2PKH](../../Script/P2PKH/P2PKH.md) Address	|1AKDDsfTh8uY4X3ppy1m7jw1fVMBSMkzjP|
|**05**	|3	|[P2SH](../../Script/P2SH/P2SH.md) Address	|34nSkinWC9rDDJiUY438qQN1JHmGqBHGW7|
|**80**	|K / L|	[WIF Private Key](../Private%20Key/WIF%20Private%20Key/WIF%20Private%20Key.md) 	|L4mee2GrpBSckB9SgC9WhHxvtEgKUvgvTiyYcGu38mr9CGKBGp93|
|**80**|	5|	WIF Private Key 	|5KXWNXeaVMwjzMsrKPv8dmdEZuVPmPay4nm5SfVZCjLHoy1B56w|
|**0488ADE4**|	xprv|	[Extended Private Key](../../HD%20Wallets/Extended%20Keys/Extended%20Keys.md)|xprv9tuogRdb5YTgcL3P8Waj7REqDuQx4sXcodQaWTtEVFEp6yRKh1CjrWfXChnhgHeLDuXxo2auDZegMiVMGGxwxcrb2PmiGyCngLxvLeGsZRq|
|**0488B21E**|	xpub|	Extended Public Key|xpub67uA5wAUuv1ypp7rEY7jUZBZmwFSULFUArLBJrHr3amnymkUEYWzQJz13zLacZv33sSuxKVmerpZeFExapBNt8HpAqtTtWqDQRAgyqSKUHu|

测试网络:
|前缀（十六进制）|	Base58前导字符|	表示|	示例|
|---|---|---|---|
|**6F**	|m / n	|P2PKH Address|	ms2qxPw1Q2nTkm4eMHqe6mM7JAFqAwDhpB|
|**C4**	|2	|P2SH Address|	2MwSNRexxm3uhAKF696xq3ztdiqgMj36rJo|
|**EF**	|c	|WIF Private Key 	|cV8e6wGiFF8succi4bxe4cTzWTyj9NncXm81ihMYdtW9T1QXV5gS|
|**EF**	|9	|WIF Private Key 	|93J8xGU85b1sxRP8wjp3WNBCDZr6vZ8AQjd2XHr4YU5Lb21jS1L|
|**04358394**|	tprv|	Extended Private Key|tprv9tuogRdb5YTgcL3P8Waj7REqDuQx4sXcodQaWTtEVFEp6yRKh1CjrWfXChnhgHeLDuXxo2auDZegMiVMGGxwxcrb2PmiGyCngLxvLeGsZRq|
|**043587CF**	|tpub	|Extended Public Key|tpub67uA5wAUuv1ypp7rEY7jUZBZmwFSULFUArLBJrHr3amnymkUEYWzQJz13zLacZv33sSuxKVmerpZeFExapBNt8HpAqtTtWqDQRAgyqSKUHu|

>十六进制前缀**"00"**在转换为base58编码时并不会自然地转换为“1”。这种转换是在代码中手动执行的。

>你会注意到[WIF私钥](../../Keys/Private%20Key/WIF%20Private%20Key/WIF%20Private%20Key.md)使用相同的十六进制前缀，但产生不同的首字符。这是因为如果使用私钥创建压缩公钥（将生成与未压缩公钥不同的地址），我们还会在转换为base58之前还会附加一个**01**。这个额外的字节会影响base58结果中的首字符。

>[扩展密钥](../../HD%20Wallets/Extended%20Keys/Extended%20Keys.md)除了原始公钥和私钥之外还包含额外的元数据，这就是它们的base58字符串要长得多的原因。

https://en.bitcoin.it/wiki/List_of_address_prefixes

>前缀还会改变地址的首字符，因此只需查看地址本身就可以知道使用了什么类型的锁定脚本。

## 为什么要使用地址？
>地址是一种以人类可读的方式简写的锁定脚本。如果- echeveria（在IRC上）

如果不使用地址，将不得不向其他人发送完整的锁定[脚本](../../Script/Script.md)，例如：
```
76a914662ad25db00e7bb38bc04831ae48b4b446d1269888ac # P2PKH脚本
```
但是通过使用地址可以发送以下内容：
```
1AKDDsfTh8uY4X3ppy1m7jw1fVMBSMkzjP
```
它们都可以达到相同的效果，但地址提供了更便于人类阅读和理解的格式。更不用说它们**包含[校验和](../Checksum/Checksum.md)**，这意味着如果有人错误地写入地址，错误可以被检测到。

## 代码
**注意**：此代码需要[checksum.rb](https://github.com/in3rsha/learnmeabitcoin-code/blob/master/checksum.rb)和[base58_encode.rb](https://github.com/in3rsha/learnmeabitcoin-code/blob/master/base58_encode.rb)函数。
```ruby
def hash160_to_address(hash160, type=:p2pkh)
  prefixes = {
    p2pkh: '00',         # 1address - For standard bitcoin addresses
    p2sh:  '05',         # 3address - For sending to an address that requires multiple signatures (multisig)
    p2pkh_testnet: '6F', # (m/n)address
    p2sh_testnet:  'C4'  # 2address
  }

  prefix = prefixes[type]
  checksum = checksum(prefix + hash160)
  address = base58_encode(prefix + hash160 + checksum)

  return address
end

hash160 = '662ad25db00e7bb38bc04831ae48b4b446d12698'
puts hash160_to_address(hash160) # 1AKDDsfTh8uY4X3ppy1m7jw1fVMBSMkzjP
```

