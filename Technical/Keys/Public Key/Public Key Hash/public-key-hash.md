# <center>公钥哈希</center>
<center>将公钥进行哈希处理来缩短它。</center>

![public-key-hash-1.png](img/public-key-hash-1%20(1).png)

**公钥哈希**是对[公钥](../Public%20Key.md)进行哈希运算得到的版本。

在比特币交易中，人们通常不会直接使用公钥，而是使用经过哈希运算的公钥哈希，它比原始公钥**短**，这样可以更加**安全且方便**。

它也是[地址](../../Address/Address.md)的“原始”版本。

## 如何创建公钥哈希？
只需将公钥通过[SHA256](https://learnmeabitcoin.com/tools/sha256)和[RIPEMD160](https://learnmeabitcoin.com/tools/ripemd160)[哈希函数](../../../Other/Hash%20Function/Hash%20Function.md)处理即可：

![public-key-hash-2.png](img/public-key-hash-2%20(1).png)

有时它被称为**HASH160（公钥）**，因为这比写**RIPEMD160（SHA256（公钥））**更简单。  
就是这样。

例子：

```
publickey          = 02b4632d08485ff1df2db55b9dafd23347d1c47a457072a1e87be26896549a8737
hash160(publickey) = 93ce48570b55c42c2af816aeaba06cfee1224fae
```
### 为什么使用RIPEMD160？

因为[RIPEMD160](https://en.wikipedia.org/wiki/RIPEMD)生成的是160位（**20字节**）摘要，比原始公钥（[未压缩](../Public%20Key.md)的是**65字节**，[压缩](../Public%20Key.md)的是**33字节**）更小。

![public-key-hash-3.png](img/public-key-hash-3%20(1).png)

这意味着最终创建的[地址](../../../Keys/Address/Address.md)将比完整的公钥包含更少的字符，从而更容易传递。

>之所以将其与**SHA256**一起使用，是因为**RIPEMD160**本身并不是安全性最强的哈希函数，结合SHA256可以提高安全性。

## 比特币中如何使用公钥哈希？
当你想要接收比特币时，你需要给对方你的**公钥哈希**。然后对方将把它放入交易[输出](../../../Transaction/Transaction%20Data/output/output.md)的[锁定代码](../../../Transaction/Transaction%20Data/output/scriptPubKey/scriptPubKey.md)中。

这将创建一个[P2PKH](../../../Script/P2PKH/P2PKH.md)锁定脚本。

![public-key-hash-4.png](img/public-key-hash-4%20(1).png)

当你想要使用这些比特币（将它们发送给新的交易对象）时，你只需将原始**公钥**以及**数字签名**放入交易[输入](../../../Transaction/Transaction%20Data/Input/input.md)的解锁代码中，就可以解锁这些比特币了。

![public-key-hash-5.png](img/public-key-hash-5%20(1).png)

因此，当节点来验证这个交易时，它会：

1. 检查**公钥**所提供的**公钥哈希值**是否正确。  
2. 如果检查通过，它会再按照常规方式验证**公钥**和**签名**是否匹配。

因此，与[P2PK](../../../Script/P2PK/P2PK.md)（Pay To PubKey）锁定不同，它不仅仅检查**签名**与**公钥**的匹配性，还会预先**检查公钥的哈希值**。

这就是为什么这种锁定系统被称为[P2PKH](../../../Script/P2PKH/P2PKH.md)（Pay To PubKey Hash）的原因。

### 代码
```ruby
require 'digest' # Hash Functions Library

publickey = '02b4632d08485ff1df2db55b9dafd23347d1c47a457072a1e87be26896549a8737'

binary = [publickey].pack("H*") # Convert to binary first before hashing
sha256 = Digest::SHA256.digest(binary)
ripemd160 = Digest::RMD160.digest(sha256)
hash160 = ripemd160.unpack("H*")[0] # Convert back to hex

puts hash160 # 93ce48570b55c42c2af816aeaba06cfee1224fae
```

## 常见问题
### 为什么要对公钥进行哈希处理？

这是因为比特币的发明者Satoshi就是这样设计比特币交易的。

>关于为什么地址是公钥哈希，你需要问Satoshi。-[ Pieter Wuille](https://bitcoin.stackexchange.com/a/72201/24926)

可能是因为Satoshi当初不知道可以使用压缩公钥（**33字节**而不是**65字节**），所以对公钥进行哈希处理的方式是为了创建一个更短的（**20字节**）版本，以便将其提供给其他人。

>简而言之，比特币地址是公钥的哈希值。- [Satoshi Nakamoto](https://satoshi.nakamotoinstitute.org/posts/bitcointalk/threads/134/#7)

![public-key-hash-6.png](img/public-key-hash-6%20(1).png)

### 另一种理论：额外的安全性
另一种理论是使用Hash160提供了额外的安全保障。

如果我们在接收比特币时立即公开了我们的公钥，那么保护你免受攻击者试图获取你的私钥的“唯一”措施就是[椭圆曲线](../../ECDSA/ECDSA.md)。

![public-key-hash-7.png](img/public-key-hash-7%20(1).png)

虽然从椭圆曲线乘法逆推出私钥是极其困难的，但攻击者仍可以尝试。

如果我提供的是公钥的哈希版本，攻击者就必须破解**RIPEMD160和SHA256**哈希函数，并解决椭圆曲线问题。

![public-key-hash-8.png](img/public-key-hash-8%20(1).png)

现在你还需要破解两种不同的哈希函数。

当比特币存储在区块链中时，哈希函数就像是额外的障碍，攻击者必须跨越它们才能尝试获取我们的私钥（并窃取我们的比特币）。

**那么椭圆曲线的保护力度还不够吗？**

实际上，它提供了极好的保护。

由于椭圆曲线乘法的特性，从公钥到私钥的反向计算非常困难。这被称为“椭圆曲线离散对数问题”。

然而，如果通过某种数学奇迹解决了这个问题，仍然有两个不同的哈希函数可以保护私钥。

**但是不是仍然会公开公钥吗？**

是的。但在这个系统中，公钥只在最后一刻公开，即当我们要消费比特币时。

理论上，如果有人想推导出你的私钥，在你的交易传播到网络并被挖掘到区块前，他们只有很少的时间来完成。因此，这比一开始就暴露公钥更加安全。

## 感谢
感谢[Pieter Wuille](https://twitter.com/pwuille)解释[为什么我们使用公钥的哈希值](https://bitcoin.stackexchange.com/a/72201/24926)而不是原始公钥。