# <center>派生路径</center>
<center>HD钱包如何派生密钥。</center>

[BIP 44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki), [BIP 49](https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki), [BIP 84](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki)

[扩展密钥](../Extended%20Keys/Extended%20Keys.md)有一个很酷的地方在于它们可以派生子密钥，这些子密钥可以再派生更多的子密钥，以此类推。这使你可以创建扩展密钥**树**，每个密钥都有自己来自[主密钥](../Extended%20Keys/Extended%20Keys.md)的唯一**派生路径**。

你可以以任何方式派生密钥。但为了保证不同钱包之间的兼容性，我们为[分层确定性钱包](../HD%20Wallets.md)中用于派生密钥的派生路径设定了一种固定的方式。

最常用的派生路径包括：

* [BIP 44](#bip-44m440000): **m/44'/0'/0'** (for **1addresses**)
* [BIP 49](#bip-49-m490000): **m/49'/0'/0'** (for **3addresses**)
* [BIP 84](#bip-84-m840000): **m/84'/0'/0'** (for **bc1addresses**)

## 1. 符号
首先，我们有一种基本的符号表示法来描述特定扩展密钥的**派生路径**。例如：
```
m/0/1/3'
```
![derivation-paths-1.png](img/derivation-paths-1%20(1).png)

斜杠/表示树中的新层级（新的子级）。数字（例如**0**）表示从父级到子级的编号。

关于符号的唯一需要知道的事情是，'或h表示[已加固](../Extended%20Keys/Extended%20Keys.md)的[子扩展私钥](../Extended%20Keys/Extended%20Keys.md)（这意味着它创建的公钥无法通过[扩展公钥](../Extended%20Keys/Extended%20Keys.md)派生）：

* **0**-**正常子级**（索引**0**）
* **0'**-**加固子级**（索引**2147483648**）

这个'只是为了避免我们写出加固子级的完整索引编号。例如，**3'**表示索引**2147483651**。

>**提示**：可以从单个扩展密钥派生多达**4294967296**个子级。前一半用于[普通子级](../Extended%20Keys/Extended%20Keys.md)，后一半用于[加固子级](../Extended%20Keys/Extended%20Keys.md)。

>**注意**：派生**加固**子级是**默认**设置。只有在需要相应的扩展公钥以派生相同的公钥时，才会派生普通子级。

## 2.钱包结构

为了帮助钱包之间保持一致性，[BIP 44](#bip-44m440000)引入了以下结构：
```
m / purpose' / coin_type' / account' / change / index
```
![derivation-paths-2.png](img/derivation-paths-2%20(1).png)

树中的各个层级有特定的含义。

>m: **主密钥**
主密钥（从[种子](../Mnemonic%20Seed/Mnemonic%20Seed.md)生成）。
>>**m/44'：目的**（加固）
这指定了即将使用的钱包结构。
目前钱包使用的有三种方案：

>>1. [BIP 44](#bip-44m440000)
>>2. [BIP 49](#bip-49-m490000)
>>3. [BIP 84](#bip-84-m840000)

>>**提示**：数字反映了BIP（比特币改进提案）的编号。新方案可以使用不同的BIP编号。
>>>**m/44'/0'：币种类型**（加固）
表示这些密钥将用于哪种加密货币。    
不同的加密货币可以使用从同一种子派生的相同**私钥和公钥**。因此可以使用相同的种子和不同的派生路径，而不是为不同的货币创建单独的种子（或在不同的链上使用相同的公钥）。  
**0' = 比特币**  
**1' = 比特币（测试网）**  
**2' = 莱特币**  
**3' = 狗狗币**  
完整列表：https://github.com/satoshilabs/slips/blob/master/slip-0044.md  
这在硬件钱包中非常有用，可以使用一个种子并将其用于持有各种不同的硬币。
>>>>**m/44'/0'/0'：账户**（加固）
这允许你为资金创建单独的账户。默认账户为**0'**。  
例如，可以使用相同的种子，但仍然创建单独的“账户”来接收付款。这些单独的账户中的硬币永远不会混淆。  
**提示**：可以在此级别的各种索引中创建许多不同的账户。但为了简化恢复过程，钱包应按顺序创建账户，并且如果之前的账户未被使用，则不应创建新账户。
>>>>>**m/44'/0'/0'/0：找零**
我们使用的密钥和地址分为“接收”和“找零”。

>>>>>* **接收** = **0** - 我们将给人们用于接收付款的地址。
>>>>>* **找零** = **1** - 我们用于在进行交易时将找零发送回自己的地址。

>>>>>这意味着你始终能够识别来自外部付款的代币。
>>>>>>**m/44'/0'/0'/0/0：索引**  
这些是你在钱包中实际使用的**私钥和公钥**的扩展密钥。

正如你所看到的，路径中的前几个级别只是用于以实用的方式构建分层确定性钱包。

用于地址的实际密钥在树的**最低级别**中。

## 3. 派生路径
以下是一些钱包实际使用的派生路径。

### **BIP 32: m/0'/0/0（已弃用）**
这是[BIP 32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#specification-wallet-structure)中最初的派生路径规范。

它只使用第一个**子级**来表示**账户**，下面的两个子级用于分离**外部**和**内部**地址。这些子级的子级用于生成实际的**私钥和公钥**以创建**地址**。
```
m / account' / external / index
```
![derivation-paths-3.png](img/derivation-paths-3%20(1).png)

这是一个简单明了的路径派生，但它不允许创建替代派生路径的方案。

这就是BIP 44、BIP 49和BIP 84的作用。

### **BIP 44：m/44'/0'/0'/0/0**
[BIP 44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)基于原始的BIP 32方案上，增加了一个**目的**字段[^1]（类似于版本号，用于识别即将到来的方案）以及**币种类型**字段，以便可以使用相同的种子生成不同加密货币的密钥。

**公钥**编码为**1addresses**（[P2PKH](../../Script/P2PKH/P2PKH.md)地址（Pay To Pubkey Hash））。

![derivation-paths-4.png](img/derivation-paths-4%20(1).png)

### **BIP 49: m/49'/0'/0'/0/0**
[BIP 49](https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki)使用与BIP 44相同的结构，但它指定了**公钥**应编码为**3addresses**（P2WPKH-P2SH（付费见证公钥哈希（嵌套在付费脚本哈希中）））。

![derivation-paths-5.png](img/derivation-paths-5%20(1).png)

### **BIP 49序列化**
BIP 49派生路径中的扩展密钥使用版本字节**049d7878**“yprv”或**049d7cb2**“ypub”进行[序列化](../Extended%20Keys/Extended%20Keys.md)。这允许你识别一个扩展密钥是否是BIP 49 方案的一部分。

例如：
```
yprvABrGsX5C9jant45o1Au7iHH54A8GXQH9SGhK5vkYKPUBDYsFy6KNUWX24moUE6KxoCh2qtZ8UpLaDWQiqt4aPdvvgjszQ4VrbLpfp5patGg
```

### **BIP 84: m/84'/0'/0'/0/0**
[BIP 84](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki)使用与BIP 44相同的结构，但用于指示**公钥**应编码为**bc1addresses**（P2WPKH（付费见证公钥哈希））。

![derivation-paths-6.png](img/derivation-paths-6%20(1).png)

>**BIP 84序列化**
BIP 84派生路径中的扩展密钥在[序列化](../Extended%20Keys/Extended%20Keys.md)过程中使用版本字节**04b2430c**“zprv”或**04b24746**“zpub”。这使你可以识别一个扩展密钥是否是BIP 84方案的一部分。

例如：
```
zprvAWgYBBk7JR8GjMGuqXgjvNNaE8GiU2GeMPDXsKeRhPr4GegVDkUw6aBA5ym4DzytCqoqbN9gwUh86o2HZaUbBscXZ5aQyyKLs4tKCeThpsa
```

## 4. 示例
它接受[种子](../Mnemonic%20Seed/Mnemonic%20Seed.md)（助记词句子或十六进制数）和派生路径，并显示该路径中的私有扩展密钥的[地址](../../Keys/Address/Address.md)（以及接下来的几个子级）。

>**Gap Limit**：从种子恢复钱包时，只需要检查**前20个**接收地址是否有余额。如果这些地址没有使用过，则可以认为该账户未使用过。

>永远不要将你的种子输入到网站中。网站可以保存你输入的内容并使用它来窃取你的货币。

## 5. 代码
为了节省代码，这些代码片段使用方便的库（[bitcoin-ruby](https://github.com/lian/bitcoin-ruby)、[hdkeychain](https://github.com/btcsuite/btcutil/tree/master/hdkeychain)）来派生[扩展密钥](../Extended%20Keys/Extended%20Keys.md)。
### Ruby
```ruby
require 'bitcoin' # sum gem install bitcoin-ruby

seed = "67f93560761e20617de26e0cb84f7234aaf373ed2e66295c3d7397e6d7ebe882ea396d5d293808b0defd7edd2babd4c091ad942e6a9351e6d075a29d4df872af"

# ------
# BIP 44
# ------
# Note: Hardened keys start at 2**31 (the second half of the 2**32 possible children).

m = Bitcoin::ExtKey.generate_master(seed.htb) # convert hex to binary
purpose   = m.derive(2**31+44) # m/44'
coin_type = m.derive(2**31+44).derive(2**31+0) # m/44'/0'
account   = m.derive(2**31+44).derive(2**31+0).derive(2**31+0) # m/44'/0'/0'
receiving = m.derive(2**31+44).derive(2**31+0).derive(2**31+0).derive(0) # m/44'/0'/0'/0

20.times do |i| 
    puts receiving.derive(i).addr # m/44'/0'/0'/0/*
end
```

### Go
```go
package main

import (
    "encoding/hex" // byte array to hex string
    "fmt"

    "github.com/btcsuite/btcd/chaincfg" // chaincfg.MainNetParams
    "github.com/btcsuite/btcutil/hdkeychain" // https://godoc.org/github.com/btcsuite/btcutil/hdkeychain
)

func main() {

    // From Seed
    seedhex := "67f93560761e20617de26e0cb84f7234aaf373ed2e66295c3d7397e6d7ebe882ea396d5d293808b0defd7edd2babd4c091ad942e6a9351e6d075a29d4df872af"
    seed, _ := hex.DecodeString(seedhex) // hex to bytes
    // fmt.Println("seed: ", seedhex)

    // Generate Seed
    // seed, _ := hdkeychain.GenerateSeed(uint8(16))
    // fmt.Println(hex.EncodeToString(seed)) // bytes to hex

    // ------
    // BIP 44
    // ------
    // m
    m, _ := hdkeychain.NewMaster(seed, &chaincfg.MainNetParams)

    // m/44h
    purpose, _ := m.Child(hdkeychain.HardenedKeyStart + 44)

    // m/44h/0h
    coin, _ := purpose.Child(hdkeychain.HardenedKeyStart + 0)

    // m/44h/0h/0h
    account, _ := coin.Child(hdkeychain.HardenedKeyStart + 0)

    // m/44h/0h/0h/0
    receiving, _ := account.Child(0) // 0 = receiving, 1 = change

    // m/44h/0h/0h/0/*
    for i := 0; i < 20; i++ {
        index, _ := receiving.Child(uint32(i)) // takes an unsigned integer
        address, _ := index.Address(&chaincfg.MainNetParams)
        fmt.Println(address)
    }
}
```

## 链接
* [BIP 44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)（[Marek Palatinus](https://github.com/slush0)，[Pavol Rusnak](https://rusnak.io/)）
* [BIP 49](https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki)（Daniel Weigl）
* [BIP 84](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki)（Pavol Rusnak）


* [Ian Coleman BIP39 Tool](https://iancoleman.io/bip39/)（用于从种子派生路径的出色交互式工具。）

[^1]:[BIP 43 (Purpose Field)](https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki)
