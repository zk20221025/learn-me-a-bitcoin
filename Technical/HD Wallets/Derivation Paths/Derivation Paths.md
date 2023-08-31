# <center>派生路径</center>
<center>HD钱包如何派生密钥。</center>

[BIP 44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki), [BIP 49](https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki), [BIP 84](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki)

[扩展密钥](../Extended%20Keys/Extended%20Keys.md)有一个很酷的地方在于它们可以派生子密钥，这些子密钥可以再派生更多的子密钥，以此类推。这使你可以创建扩展密钥树，每个密钥都有其自己唯一的来自[主密钥](../Extended%20Keys/Extended%20Keys.md)的**派生路径**。

可以以任何方式派生密钥。但为了帮助钱包之间的兼容性，我们为[层次确定性钱包](../HD%20Wallets.md)中用于派生密钥的派生路径设置了一组结构。

最常用的派生路径包括：

* [BIP 44](#bip-44m440000): m/44'/0'/0' (for 1addresses)
* [BIP 49](#bip-49-m490000): m/49'/0'/0' (for 3addresses)
* [BIP 84](#bip-84-m840000): m/84'/0'/0' (for bc1addresses)

## 1. 符号
首先，我们有一种基本的符号表示法来描述特定扩展密钥的**派生路径**。例如：
```
m/0/1/3'
```
![derivation-paths-1.png](img/derivation-paths-1%20(1).png)

斜杠/表示树中的新级别（新的子级）。数字（例如0）表示从父级的子级编号。

关于符号的唯一其他需要知道的事情是，'或h表示[已硬化](../Extended%20Keys/Extended%20Keys.md)的[子扩展私钥](../Extended%20Keys/Extended%20Keys.md)（这意味着它创建的公钥无法通过[扩展公钥](../Extended%20Keys/Extended%20Keys.md)派生）：

* 0-**正常子级**（索引0）
* 0'-**硬化子级**（索引2147483648）

这个'只是为了避免我们写出硬化子级的完整索引号。例如，3'表示索引2147483651。

>提示：可以从单个扩展密钥派生多达4294967296个子级。前一半用于[普通子级](../Extended%20Keys/Extended%20Keys.md)，后一半用于[硬化子级](../Extended%20Keys/Extended%20Keys.md)。

>注意：派生硬化子级是默认设置。只有在需要相应的扩展公钥以派生相同的公钥时，才会派生普通子级。

## 2.钱包结构

为了帮助钱包之间保持一致性，[BIP 44](#bip-44m440000)引入了以下结构：
```
m / purpose' / coin_type' / account' / change / index
```
![derivation-paths-2.png](img/derivation-paths-2%20(1).png)

树中的层级有特定的含义。

>m: **主密钥**
主密钥（从[种子](../Mnemonic%20Seed/Mnemonic%20Seed.md)生成）。
>>m/44'：**目的**（加强）
这指定了即将使用的钱包结构。
目前钱包使用的有三种方案：

>>1. [BIP 44](#bip-44m440000)
>>2. [BIP 49](#bip-49-m490000)
>>3. [BIP 84](#bip-84-m840000)

>>提示：数字反映了BIP的编号。新方案可以使用不同的BIP编号。
>>>m/44'/0'：**币种类型**（加强）
密钥将用于的加密货币。  
不同的加密货币可以使用从种子派生的相同私钥和公钥。因此可以使用相同的种子和不同的派生路径而不是为不同的货币创建单独的种子（或在不同的链上使用相同的公钥）。
0' = 比特币
1' = 比特币（测试网）
2' = 莱特币
3' = 狗狗币
...
完整列表：https://github.com/satoshilabs/slips/blob/master/slip-0044.md  
这在硬件钱包中非常有用，可以使用单个种子并将其用于持有各种不同的硬币。
>>>>m/44'/0'/0'：**账户**（加强）
这允许你为资金创建单独的账户。默认帐户为0'。  
例如，可以使用相同的种子，但仍然创建单独的“账户”来接收付款。这些单独的账户中的硬币永远不会混合在一起。  
提示：可以在此级别的各种索引中显然创建许多不同的帐户。但为了简化恢复过程，钱包应按顺序创建帐户，并且如果之前的帐户未被使用，则不创建新帐户。
>>>>>m/44'/0'/0'/0：**更改**
我们使用的密钥和地址分为“接收”和“更改”。

>>>>>* **接收** = 0 - 我们将给人们用于接收付款的地址。
>>>>>* **更改** = 1 - 我们用于在进行交易时将找零发送回自己的地址。

>>>>>这意味着你始终能够识别来自外部付款的硬币。
>>>>>>m/44'/0'/0'/0/0：**索引**  
这些是你在钱包中实际使用的私钥和公钥的扩展密钥。

正如你所看到的，路径中的前几个级别只是用于以实用的方式构建分层确定性钱包。

用于地址的实际密钥在树的**最低级别**中。

## 3. 派生路径
以下是一些钱包实际使用的派生路径。

### **BIP 32: m/0'/0/0（已弃用）**
这是[BIP 32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#specification-wallet-structure)中最初的派生路径规范。

它只使用第一个**子级**来表示账户，下面的两个子级用于分离**外部**和**内部**地址。这些子级的子级用于它们的实际私钥和公钥以创建地址。
```
m / account' / external / index
```
![derivation-paths-3.png](img/derivation-paths-3%20(1).png)

这是一个简单明了的推导路径，但它不允许创建替代的推导路径方案。

这就是BIP 44、BIP 49和BIP 84的作用。

### **BIP 44：m/44'/0'/0'/0/0**
[BIP 44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)基于原始的BIP 32方案，包括一个目的[^1]（类似于版本号，用于识别即将到来的方案）以及币种类型，以便可以使用相同的种子生成不同加密货币的密钥。

公钥编码为1地址（[P2PKH](../../Script/P2PKH/P2PKH.md)）。

![derivation-paths-4.png](img/derivation-paths-4%20(1).png)

### **BIP 49: m/49'/0'/0'/0/0**
[BIP 49](https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki)使用与BIP 44相同的结构，但用于指示公钥应编码为3地址（P2WPKH-P2SH）。

![derivation-paths-5.png](img/derivation-paths-5%20(1).png)

### **BIP 49序列化**
BIP 49派生路径中的扩展密钥使用版本字节049d7878“yprv”或049d7cb2“ypub”进行[序列化](../Extended%20Keys/Extended%20Keys.md)。这允许你识别属于 BIP 49 方案一部分的扩展密钥。

例如：
```
yprvABrGsX5C9jant45o1Au7iHH54A8GXQH9SGhK5vkYKPUBDYsFy6KNUWX24moUE6KxoCh2qtZ8UpLaDWQiqt4aPdvvgjszQ4VrbLpfp5patGg
```

### **BIP 84: m/84'/0'/0'/0/0**
[BIP 84](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki)使用与BIP 44相同的结构，但用于指示公钥应编码为bc1地址（P2WPKH）。

![derivation-paths-6.png](img/derivation-paths-6%20(1).png)

>**BIP 84序列化**
BIP 84派生路径中的扩展密钥在[序列化](../Extended%20Keys/Extended%20Keys.md)。过程中使用版本字节04b2430c“zprv”或04b24746“zpub”。这可以在BIP 84方案的一部分时识别扩展密钥

例如：
```
zprvAWgYBBk7JR8GjMGuqXgjvNNaE8GiU2GeMPDXsKeRhPr4GegVDkUw6aBA5ym4DzytCqoqbN9gwUh86o2HZaUbBscXZ5aQyyKLs4tKCeThpsa
```

## 4. 示例
它接受[种子](../Mnemonic%20Seed/Mnemonic%20Seed.md)（助记符或十六进制数）和派生路径，并显示该路径中的私有扩展密钥的[地址](../../Keys/Address/Address.md)（以及接下来的几个子级）。

>**间隙限制**：从种子恢复钱包时，应仅检查**前20个**接收地址是否有余额。如果过去没有使用过任何地址，则可以将该账户视为未使用过。

>永远不要将你的实际种子输入到网站中。网站可以保存你输入的内容并使用它来窃取你的货币。

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
