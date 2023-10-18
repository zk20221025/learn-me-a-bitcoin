# <center>HD钱包</center>
<center>使用单个种子生成密钥树。</center>

[BIP 32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)

**分层确定性钱包**（或“HD钱包”）是从单一来源生成所有密钥和地址的钱包。

* **确定性**意味着密钥和地址始终以相同的方式生成。
* **分层**意味着密钥和地址可以组织成树状结构。

但是这些钱包最重要的一个优点是，可以在不知道[私钥](../Keys/Private%20Key/Private%20Key.md)的情况下生成新的[公钥](../Keys/Public%20Key/Public%20Key.md)。

## 为什么使用HD钱包？

### 1. 单一备份
在基本钱包中，每当你想接收一些比特币时，都会独立生成**私钥和公钥**配对。

但这意味着每次收到新付款时都需要备份钱包。

![hd-wallets-1.gif](img/hd-wallets-1%20(1).gif)

但在分层确定性钱包中，只需要备份一次，你可以使用单个**种子**创建**主私钥**，并使用它来生成数十亿个（4,294,967,296）“子”**私钥和公钥**。

![hd-wallets-2.gif](img/hd-wallets-2%20(1).gif)

只要有**种子**，就能恢复出所有的密钥和地址，因为这个种子能生成的主私钥会以确定的方式生成所有的子私钥和公钥。这大大简化了钱包备份的过程。

### 2. 组织
分层确定性钱包的另一个优点是它的分层结构。

钱包中的每个**子密钥**也可以生成**自己的密钥**，这意味着你可以创建**树形结构**（或层次结构）来组织钱包中的密钥。

例如，你可以使用树的不同部分来管理不同的“账户”，这样可以更好地管理和组织你的资金。

![hd-wallets-3.gif](img/hd-wallets-3%20(1).gif)

### 3. 独立生成公钥。
但**主私钥**的真正酷炫之处在于它有对应的**主公钥**，并且这个主公钥可以在不知道私钥的情况下生成相同的子公钥。

![hd-wallets-4.png](img/hd-wallets-4%20(1).gif)

因此，**主公钥**使你可以在不同的计算机上（例如，Web商店服务器）以生成新的接收地址，而不必担心如果服务器遭到入侵导致私钥被盗。

>这对于像**硬件钱包**之类的东西非常有用，你希望将私钥保留在安全设备上，但同时也希望能够在不同的计算机上生成新的地址以接收付款。

这可能看起来像魔法，但这只是数学的应用。

## HD钱包如何工作？

### 1.种子

![hd-wallets-5.png](img/hd-wallets-5%20(1).png)

要创建一个HD钱包，需要生成64个随机字节作为**种子**。

```
种子：3d08def3b82123f428777b4fd9d9c4dec6eb029839eb0b11fd2fb99e36732c43d919b9be0c2493c3e0057dfbbaf861ecd41346bfacbcd1ecf995e4831d059eaf
```

>助记词句子  
为了使这个过程更易于用户理解和记忆，你可以使用**助记词句子**来代替原始的十六进制种子。

例如，上面的种子是从以下助记词句子创建的：
```
mnemonic: muffin sheriff judge garment pottery alpha emerge civil stage broken junior know
```
请参阅[助记词句子](./Mnemonic%20Seed/Mnemonic%20Seed.md)以获取详细信息。

### 2.主私钥

![hd-wallets-6.gif](img/hd-wallets-6%20(1).gif)

“主密钥”是通过将种子输入哈希函数（称为HMAC（基于哈希的消息验证码））来生成另一组64个字节而创建的。

我们使用这64个字节来创建我们的**主扩展私钥**。

* 前32个字节是**私钥**。
* 最后32个字节是**链码**。

**链码**只是额外的32个字节，我们将其与**私钥**配对以创建[扩展密钥](./Extended%20Keys/Extended%20Keys.md)。

>**为什么要对种子进行哈希处理**？虽然可以直接使用64个字节的种子来创建**主扩展私钥**。然而，未来的子扩展密钥是使用HMAC创建的，因此可以保持创建方式一致。

**扩展私钥**

```
种子：68402660a3b31cf735c533866d9f749078f45fa0433bded3a5c3c080c21bd2364fb6ddabd12450572dd4a0f046bd4302306cad79f7be4bf55457cd3e467a001c

主扩展私钥：
  私钥：17663ce2af7243429017cb1f49a465f02051790f78ca4b352529a706dadea639
  链码：   2699081c5a668d91fc2ecc7649fa88c1f478f51817bc0d554f4708e19afd9a64
```
**公钥**

嵌入此扩展密钥中的私钥可用于正常创建相应的公钥：
```
  私钥：17663ce2af7243429017cb1f49a465f02051790f78ca4b352529a706dadea639
  公钥：   03fdd74b988194d4aa79fec227ab8d685b1c37864fb0864603e93cd3295eba629e
```
但实际的**主扩展私钥**本身只是**私钥**和**链码**。

### 3. 子密钥(基本)
新的子私钥是通过将**扩展私钥**的内容（**私钥和链码**）通过HMAC函数生成的。每次还包含一个**索引编号**，这使我们能够从一个主密钥创建多个子密钥。

![hd-wallets-7.gif](img/hd-wallets-7%20(1).gif)

通过改变索引，哈希函数的结果完全不同的结果，这样就可以保证每个子私钥的唯一性。

```
种子: 68402660a3b31cf735c533866d9f749078f45fa0433bded3a5c3c080c21bd2364fb6ddabd12450572dd4a0f046bd4302306cad79f7be4bf55457cd3e467a001c

master extended private key:
  private key: 17663ce2af7243429017cb1f49a465f02051790f78ca4b352529a706dadea639
  chain code:  2699081c5a668d91fc2ecc7649fa88c1f478f51817bc0d554f4708e19afd9a64

  child 0:
    private key: a8e8f6ce276c4a34d8c29a2e187f311b18f05ba3fac25103827ccc55328f0aa8
    public key:  03d6377f0c7441f630bbe8c858cc35c6e8f9e5c7ee274f8994431c8ff9742d4dbf

  child 1:
    private key: 5edd81ce37382ed32c99b75b5786e8470eb97016f2126c78ec5a477573ec7001
    public key:  039764624f60f132e9fa95e141c5753f0baa2f12fc09d54a1e3f625d4cda7fc9b3

  子2：
    私钥：ff07d295c693c7480ca2412a7248dc46714309200d1b82b723e561fa3ac9e275 914c
    公钥：   02ac50ba135a5a1d0e210b4dcfd4bec5732bc18011dec3693360a5b798873e78f4
```

因此，通过将**主扩展私钥**与一个**索引编号**进行哈希运算，可以生成新**私钥**。

>扩展密钥可以生成2,147,483,648个这样的子密钥。

### 4. 子密钥（高级）
现在这才是有趣的部分。

假设我们想要一个**扩展私钥**来创建**子私钥和公钥**，同时也希望有一个对应的**扩展公钥**，它可以生成相同的**子公钥**，该怎么办？

**扩展公钥**
首先，我们需要构建**扩展公钥**。这只是从**扩展私钥**中获取的**公钥**，再加上相同的**链码**：

![hd-wallets-8.gif](img/hd-wallets-8%20(1).gif)

种子
```
seed: b1680c7a6ea6ed5ac9bf3bc3b43869a4c77098e60195bae51a94159333820e125c3409b8c8d74b4489f28ce71b06799b1126c1d9620767c2dadf642cf787cf36

master extended private key:
  private key: 081549973bafbba825b31bcc402a3c4ed8e3185c2f3a31c75e55f423e9629aa3
  chain code:  1d7d2a4c940be028b945302ad79dd2ce2afe5ed55e1a2937a5af57f8401e73dd

master extended public key:
  public key: 0343b337dec65a47b3362c9620a6e6ff39a1ddfa908abab1666c8a30a3f8a7cccc
  chain code: 1d7d2a4c940be028b945302ad79dd2ce2afe5ed55e1a2937a5af57f8401e73dd
```
**扩展私钥的子私钥**  
**主扩展私钥**通过将其对应的**扩展公钥**的内容通过HMAC函数进行处理，并将结果添加到原始**私钥**中来创建**子私钥**。

![hd-wallets-9.gif](img/hd-wallets-9%20(1).gif)

```
种子: 68402660a3b31cf735c533866d9f749078f45fa0433bded3a5c3c080c21bd2364fb6ddabd12450572dd4a0f046bd4302306cad79f7be4bf55457cd3e467a001c

master extended private key:
  private key: 17663ce2af7243429017cb1f49a465f02051790f78ca4b352529a706dadea639
  public key:  03fdd74b988194d4aa79fec227ab8d685b1c37864fb0864603e93cd3295eba629e
  chain code:  2699081c5a668d91fc2ecc7649fa88c1f478f51817bc0d554f4708e19afd9a64

  child 0:
    private key: 78fd1653cfc12fffdfc35f0b29ff6e28ec50ad99e5a0df663b77e282963c6618
    public key:  027961138b95306d5f320d00fb22516f722f24b1f697ef9cb38dbebd4783c15242

  child 1:
    private key: d2424ad9d80e5ab19fa58573c860cc65e28bc85382f18c50b070a1db236dc557
    public key:  028963ec6e4953280f2379f51c9d0ea4267d99f182edcdd68b3d182859612fde94

  child 2:
    private key: faa8e605af8864903b1031053cf8cf8eaf9e032538b83ad52a9f2487033f95d6
    public key:  02430c388a56b73135e68856e2ade0e7b2c167aa822c0b842c08bb991d95a59441
```

**扩展公钥的子公钥**
**主扩展公钥**将其内容通过HMAC函数进行处理，并将结果添加到原始**公钥**中来创建新的**子公钥**。

![hd-wallets-10.gif](img/hd-wallets-10%20(1).gif)

```
master extended public key:
  public key: 03fdd74b988194d4aa79fec227ab8d685b1c37864fb0864603e93cd3295eba629e
  chain code: 2699081c5a668d91fc2ecc7649fa88c1f478f51817bc0d554f4708e19afd9a64

  child 0:
    public key: 027961138b95306d5f320d00fb22516f722f24b1f697ef9cb38dbebd4783c15242

  child 1:
    public key: 028963ec6e4953280f2379f51c9d0ea4267d99f182edcdd68b3d182859612fde94

  child 2:
    public key: 02430c388a56b73135e68856e2ade0e7b2c167aa822c0b842c08bb991d95a59441
```
由于原始**私钥和公钥**都已经调整了同样的值，新的**子私钥和公钥**是对应的。

![hd-wallets-11.gif](img/hd-wallets-11%20(1).gif)

>扩展密钥可以生成2,147,483,648个子密钥。  
因此，扩展密钥总共可以派生4,294,967,296个子密钥（2,147,483,648个（索引**0到2147483647**）高级子密钥和2,147,483,648个（索引**2147483648到4294967295**）基本子密钥）。

有关更多详细信息，请参见[扩展密钥](./Extended%20Keys/Extended%20Keys.md)。

>**为什么要使用链码来扩展密钥？**

通过向密钥添加**链码**，意味着子密钥不再仅依赖于主公钥派生的。

例如，可以使用树中的**公钥**之一来接收付款，这将使其在[区块链](../Blockchain/blockchain.md)上可见。如果不使用链码，任何人都可以获取此**公钥**并派生其所有子密钥。

![hd-wallets-12.png](img/hd-wallets-12%20(1).png)

但是通过使用链码（不会影响区块链），其他人无法从公钥推导出其子密钥。

![hd-wallets-13.png](img/hd-wallets-13%20(1).png)

换句话说，**链码**是一些额外的保密数据，用于防止从主公钥派生出来的子密钥被破解。

>**请记住**：当我们将普通密钥与额外的**链码**相结合时，我们称之为“扩展密钥”。

>**HD钱包中的密钥是否相互关联？**

不是的，你无法知道树中的任何两个**公钥**（或地址）是否属于同一个钱包（即派生自相同的主扩展密钥）。

尽管子密钥是以特定方式从主扩展密钥派生的，但实际的**公钥**本身并没有任何相似之处。就好像它们是完全独立生成一样。

![hd-wallets-14.png](img/hd-wallets-14%20(1).png)

>**HD钱包中的密钥是否安全？**

是的，从HD钱包获得的所有**私钥和公钥**与传统方式生成的一样安全。

但是，应该**特别注意保护使用的扩展密钥**，因为任何拥有它们的人都可以派生它们的所有子密钥。

例如，如果你透露了你的**主扩展公钥**，其他人将能够查看你钱包中的地址。他们不能偷取任何东西，因为他们无法生成**私钥**，但他们仍然可以看到你拥有多少比特币。

![hd-wallets-15.png](img/hd-wallets-15%20(1).png)

此外，泄露**主扩展公钥和任何子私钥**会使其他人可以计算出**主扩展私钥**。他们就可以**生成钱包中的所有私钥**：

![hd-wallets-16.png](img/hd-wallets-16%20(1).png)

简而言之，尽量不要公开你的**主扩展公钥**。如果你这样做，其他人可以找到你钱包中的所有地址。如果你连**子私钥**也公开了，那么这与公开**主扩展私钥**一样糟糕。

## 扩展密钥是什么样子的？
为了使扩展密钥更易传递，我们将它们与一些附加数据一起序列化。

例如，这是**主扩展私钥**在序列化后的样子：
```
version:     0488ade4     # puts "xprv" or "xpub" at the start after encoding to base58
depth:       00           # how deep we are in the key tree
index:       00000000     # the index number
fingerprint: 00000000     # this is from the hash of the parent key
chain code:  92c022d5b43ed7ecf65ebe37c5754cae1583bb4559a894291ed58529885651e5
key:         00389bb0ee8f01c94f7c94594e813ae2eccb67b0a64bc44c32a2663a7c012edcb1 # prepend 00 for private keys

serialized: 0488ade400000000000000000092c022d5b43ed7ecf65ebe37c5754cae1583bb4559a894291ed58529885651e500389bb0ee8f01c94f7c94594e813ae2eccb67b0a64bc44c32a2663a7c012edcb1
```
可以通过将其转换为[base58](../Keys/Base58/Base58.md)（包括[校验和](../Keys/Checksum/Checksum.md)）来使其更方便：
```
xprv9s21ZrQH143K3X6emxCTwJs3G4wQW2qJUnJKEZPBUkbR6auSitdSzEcVjR47uUH7cqnuq7CTCDABsYcmNJfCZcVWeGti616FnkgBfXyDEtx
```
这是对于扩展密钥来说更有用的格式。它更容易在计算机之间共享和导入到钱包中。

有关详细信息，请参阅[扩展密钥序列化](./Extended%20Keys/Extended%20Keys.md)。

## 谁提出了HD钱包的概念？

[Gregory Maxwell](https://github.com/gmaxwell)最初提出了这个想法，即可以通过调整公钥来获取新的公钥，而无需知道它们的私钥，这也被称为同态派生。

>[FSF](https://www.fsf.org/)希望接受比特币捐赠，并希望为每个用户生成新的地址，但不希望在他们的Web服务器上存储私钥。- Gregory Maxwell

**Armory**是第一个实现同态派生的钱包，并引入了使用链码的概念。

[Pieter Wuille](https://github.com/sipa)提出了使用分层结构的想法，并在Armory使用的方案基础上构建了[BIP 32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)规范。

>HD钱包（BIP32）基于Armory的方案，但具有更大的灵活性（分层结构）和索引的随机访问（Armory的方案需要在派生地址号n之前生成所有N个地址）。- Pieter Wuille

## 什么是HD钱包的一些例子？
大多数现代钱包（自2013年以来）都是分层确定性钱包。以下是一些流行的例子：

* [Electrum](https://electrum.org/)
* [Samourai](https://samouraiwallet.com/)
* [Mycelium](https://wallet.mycelium.com/)
* [Trezor](https://trezor.io/)
* [Ledger](https://www.ledger.com/)

它们在创建钱包时为你提供种子，然后用其生成所有密钥和地址。

**派生路径**
然后根据以下结构生成实际密钥：
```
m/44'/0'/0'
m/49'/0'/0'
m/84'/0'/0'

```
有关更多细节，请参阅[派生路径](./Derivation%20Paths/Derivation%20Paths.md)。

## 总结
![hd-wallets-17.gif](img/hd-wallets-17%20(1).gif)

**分层确定性钱包**提供了一个有效的方法来生成新的[私钥](../Keys/Private%20Key/Private%20Key.md)和[公钥](../Keys/Public%20Key/Public%20Key.md)。

它的确定性在于，所有的子密钥都是每次以相同的方式从种子生成的，而且它是分层的，因为可以将密钥组织成树形结构（或层次结构）。额外的好处是可以在不知道私钥的情况下派生钱包中的公钥。

如果你对HD钱包的详细信息感兴趣，这里有一些更技术性的解释：

* [助记词句子](./Mnemonic%20Seed/Mnemonic%20Seed.md)（为你的HD钱包生成用户友好的种子。）
* [扩展密钥](./Extended%20Keys/Extended%20Keys.md)（创建主扩展密钥，并从中派生子密钥。）
* [导出路径](./Derivation%20Paths/Derivation%20Paths.md)（钱包用于组织密钥的常见层次结构。）

## 链接
* https://bitcointalk.org/index.php?topic=19137 (Gregory Maxwell)
* https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki (Pieter Wuille)


* https://iancoleman.io/bip39/ (一个令人惊叹的网络工具)
* https://github.com/lian/bitcoin-ruby/blob/master/lib/bitcoin/ext_key.rb (Ruby中的干净实现)
* https://www.youtube.com/watch?v=OVvue2dXkJo (James Chiang的演讲)


* https://www.cs.cornell.edu/~iddo/detwal.pdf (Gregory Maxwell关于确定性钱包的幻灯片)
* https://eprint.iacr.org/2014/998.pdf (Gus Gutoski和Douglas Stebila的有趣论文)
