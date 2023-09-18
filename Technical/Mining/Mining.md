# <center>挖矿</center>
<center>向区块链添加新的区块。</center>

![Mining-S-1.png](img/Mining-S-1%20(1).png)

**挖矿**是尝试将新的[交易](../Transaction/Transaction%20Data/Transaction%20Data.md)[块](../../Beginners/How%20Bitcoin%20Works/2.Mining/2.Blocks/Blocks.md)添加到[区块链](../Blockchain/blockchain.md)上的过程。它是一个网络范围内的竞争，网络上的任何节点都可以尝试将新的交易块添加到链上。

每个新挖掘出的块都会在网络上广播，每个节点在将其添加到其区块链之前都会独立验证它。

![Mining-S-2.png](img/Mining-S-2%20(1).png)

<center>节点使用新区块更新他们的区块链。</center>

在添加新块后，每个挖矿节点都会重新启动该过程，以尝试在链中的这个新块之上进行构建。因此，区块链的不断建立，归功于网络中节点的协作努力。

>这个系统的设计是平均每10分钟挖掘出一个新的区块。

## 1. 挖矿是如何工作的？
挖矿过程首先用节点[内存池](../Node/Memory%20Pool/Memory%20Pool.md)中的交易填满[候选区块](../Node/Candidate%20Block/Candidate%20Block.md)。

如果挖矿节点成功解决了这个候选区块的工作量证明问题，这个候选区块会被添加到区块链上（然后发送给其他人，让他们也可以将其添加到他们的区块链上）。

![Mining-S-3.png](img/Mining-S-3%20(1).png)

接下来为这个[候选块](../Block/block-header/block-header.md)构建一个区块头。这基本上是块内所有数据的简短总结，其中包括我们想要在其上构建的区块链中现有区块的引用。

![Mining-S-4.png](img/Mining-S-4%20(1).png)

你可以通过[区块哈希](../Block/block-header/block-header.md)来引用先前的区块，而区块中所有交易的摘要都包含在[默克尔根](../Block/block-header/merkle-root/merkle-root.md)中。

现在准备开始“挖掘”这个块。为了做到这一点，将这个块的块头放入[SHA256](https://en.wikipedia.org/wiki/SHA-2)[哈希函数](../Other/Hash%20Function/Hash%20Function.md)中，并希望它输出的数字低于当前[目标](../Mining/Target/Target.md)。

尝试一下！- SHA256  
哈希函数基本上是一个小型计算机程序，它接收任意数量的数据，将其混淆，并产生一个不可预测的结果。不同的数据块产生不同的结果。

![sha256.png](img/sha256.png)

>**注意**：哈希函数的结果以[十六进制](../Other/Hexadecimal/hexadecimal.md)显示，但可以轻松地转换为十进制数。

>**注意**：在挖掘中，数据会通过哈希函数处理两次，因此这不会产生相同的结果（但对于说明目的效果很好）。

![Mining-S-5.png](img/Mining-S-5%20(1).png)

<center>目标是你的区块哈希必须低于的数字，才能将该区块添加到区块链上。</center>

如果块头的哈希值不低于目标值，你可以通过增加块头中的[nonce](../Block/block-header/Nonce/Nonce.md)字段来继续尝试，这样你可以保持相同的基本块头，又可以得到完全不同的哈希结果。如果幸运的话，你可能会得到低于当前目标值的哈希值。

![Mining-S-6.png](img/Mining-S-6%20(1).png)

<center>挖矿过程基本上是尽可能快地对块头进行哈希运算，以尝试成为第一个低于目标值的节点。</center>

如果你成功获取了一个低于目标值的块哈希，则可以将你的块广播到网络上。每个节点在接收到新块后会确认该块头哈希低于目标值，然后将这个“挖出”的区块添加到他们的区块链上。

![Mining-S-7.png](img/Mining-S-7%20(1).png)

<center>恭喜，你刚刚将一批交易挖掘到了区块链上。</center>

每个节点将停止在自己的候选区块上工作，然后构建一个新的区块（从其内存池中获取新的交易），并开始尝试在这个新块上构建。

![Mining-S-8.png](img/Mining-S-8%20(1).png)

<center>矿工开始尝试将下一批交易添加到链上。</center>

因此，矿工们不断地独立（又协作的）工作，以新的交易块扩展区块链。

>挖矿的过程通常被称为**工作量证明**。  
“工作量证明”一词只是指得到一个块哈希值低于目标值需要付出努力，并且一旦你做到了，任何人都可以检查哈希值是否低于目标值。  
换句话说，哈希函数被用来证明你在块上执行了所需的“工作”量。

>工作量证明涉及扫描一个值，当进行哈希运算时（例如使用SHA-256），哈希值以一定数量的零位开始。-[**中本聪**](https://bitcoin.org/bitcoin.pdf)

## 2. 谁能挖矿?
**任何节点**都可以尝试挖掘区块，每个节点都有成功的机会。因此，我们进行了一场全网络范围的竞争，任何网络上的节点都可能将下一批交易添加到区块链上。

![Mining-S-9.png](img/Mining-S-9%20(1).png)

然而，虽然任何人都可以尝试挖矿，但能够尽快地执行哈希计算的能力确实会提高成功挖掘新区块的机会。

![Mining-S-10.png](img/Mining-S-10%20(1).png)

>任何人在任何时间内找到解决方案的机会与他们的 CPU 算力成正比。-[中本聪](https://satoshi.nakamotoinstitute.org/emails/cryptography/13/#selection-173.284-173.369)

因此，拥有最多处理能力（或“哈希能力”）的矿工很可能比那些无法快速哈希计算的人挖掘更多的块。因此，尽管任何人都可以挖掘，但它往往集中在拥有专业挖掘硬件和廉价电力的人之间。

## 3. 为什么要开采区块？
因为如果你能够挖掘一个区块，你就可以获得**区块奖励**。

你知道，当你构建一个候选区块时，你可以在区块顶部放置自己的特殊交易。这被称为[coinbase交易](../Transaction/Coinbase%20Transaction/Coinbase%20Transaction.md)，它允许你发送给自己一定数量的之前不存在的比特币。

![Mining-S-11.png](img/Mining-S-11%20(1).png)

<center>Coinbase交易实际上允许你创建新的比特币并将它们发送给自己。</center>

如果你挖到了这个区块，一旦该区块在[最长的链](../Blockchain/longest-chain/longest-chain.md)中达到100个区块深度，你就可以花费从coinbase交易中获得的比特币。

![Mining-S-12.png](img/Mining-S-12%20(1).png)

因此，这个**区块奖励作为激励**，促使矿工挖掘新的区块，并不断尝试扩展已知的最长区块链。

![Mining-S-13.png](img/Mining-S-13%20(1).png)

>**提示**：仅在 coinbase 交易中才允许发送尚不存在的比特币，这使它们成为所有新比特币的来源。

>**提示**：挖掘引入新比特币进入网络的事实是为什么这个过程被称为“挖掘”，尽管从实际角度来看，它更多地涉及将新交易添加到区块链中。

## 4. 挖掘一个区块需要多长时间？
该系统被设计成，网络中的一个矿工平均**每10分钟**成功挖出一个新区块。

![Mining-S-14.png](img/Mining-S-14%20(1).png)

这个时间由[目标](./Target/Target.md)控制，区块哈希值必须进入该目标值才能被允许进入区块链。

![Mining-S-15.png](img/Mining-S-15%20(1).png)

每个节点都知道当前区块链高度的当前目标。

如果在两周的时间内，挖矿的速度比平均值10分钟更快（例如，因为更多的矿工加入网络），则目标值将向下调整，以便更难挖掘一个区块，因此**区块之间的平均时间恢复**到约10分钟左右。

![Mining-S-16.png](img/Mining-S-16%20(1).png)

<center>每个节点独立调整目标，但如果它们每个节点具有相同的区块链，则会计算出相同的目标。</center>

因此，目标定期进行调整，以保持每个新区块之间的时间间隔为10分钟，这样可以**保持新区块可以预测**并使新比特币的发行量保持稳定。

## 5. 我们为什么要挖矿?
因为这个系统**允许网络上的计算机解决冲突，而无需中央计算机或权威机构来处理它们**。

你看，比特币在独立计算机的网络上运行，因此可能会产生两个冲突的交易（将同一比特币发送到不同的地方），并同时将它们插入网络上的不同节点。一些节点将首先收到**交易A**，而其他节点将首先收到**交易B**。

![Mining-S-17.png](img/Mining-S-17%20(1).png)

<center>所有计算机如何才能就哪些交易应该被记录在区块链中达成一致？</center>

由于挖矿机制的存在，**只有其中一笔交易会被记录在区块链上**。

最终，网络中的一个节点会从其内存池中挖出一个交易块，然后将该块广播到网络的其余部分。当节点接收到该块时，它们会将其添加到自己的链中，并从其**内存池中删除任何冲突的交易**。

![Mining-S-18.png](img/Mining-S-18%20(1).png)

因此，挖矿过程充当了一种跨计算机网络的交易排序机制，挖掘出的块对于区块链中的内容具有最终决定权。还有一个好处，由于任何人都可以进行挖矿，因此网络上没有任何单个节点能够完全控制哪些交易被记录在区块链上。

>除非一个单独的矿工获得了[大部分的挖矿算力](../../Technical/Blockchain/51-attack/51-attack.md)...

## 6.你如何挖掘一个区块？
你首先要构建一个区块的[区块头](../Block/block-header/block-header.md)用于你的块。例如，这是[第100,000个区块](https://learnmeabitcoin.com/explorer/block/000000000003ba27aa200b1cecaad478d2b00432346c3f1f3986da1afd33e506)的区块头的开头：
```
0100000050120119172a610421a6c3011dd330d9df07b63616c2cc1f1cd00200000000006657a9252aacd5c0b2940996ecff952228c3067cc38d4885efb5a4ac4247e9f337221b4d4c86041b00000000
```

现在你已经得到了区块头，你尝试通过将其通过SHA256哈希函数（两次）来“挖掘”它。你不断增加[nonce](../Block/block-header/Nonce/Nonce.md)值，以尝试获得低于[目标值](../Mining/Target/Target.md)的结果。

例如：
```
Nonce     Hash256
--------  -------
00000000: 5bd0d617b30a972407ad69a845cd74fb201d940cd45acc15fcd4761493bc3ae2
01000000: 6879c316d8a96269825111bb0616331307bb6677b2af55127922d8c568e4b2db
02000000: 34d69cb489442234ec54462e18262bf1a9c756a4e7909e68edf979a0cb39a3fa
03000000: 8cf5e032093cfcbf4f6b443608631dd33699fca13dcbd6118992f9d451b70dd8
04000000: a32d73e3f2fd0579c44080cb6b1717582d37b8f47a8445922dee0996b503c04c
05000000: 70b11dabc90a107918a55eff7e940d41b0ce1924aeed2f0b7ccb2d4ee60e1617
06000000: b38afc30567703629226557a7748bea449693156e0b116b03a8442ccc3b9005e
07000000: 2a6008f39daa1238388eadcab111c6b556c3d89a46558c98f8ed32746fb5d7b8
08000000: 248e33c82a744786e7e16336612221850e32f8e6cb09b2eb0b0730ac6beb71b4
09000000: a4aea05d2750e49e8f95f5608293f6b4f45bd34aee51e86893dd8c0230d19185
0a000000: ce2225a69a5bf2dbb25cbb27fda78a8c4c3ac5280ec7996426eeefeb0e5e1ecb
...
```
最终，你可能会找到一个nonce值，它会产生比目标值更小的哈希结果：
```
Nonce    Hash256
-------- -------
0f2b5710: 000000000003ba27aa200b1cecaad478d2b00432346c3f1f3986da1afd33e506
```
>提示：在[区块头](../Block/block-header/block-header.md)中，nonce是一个4字节的字段，其中数字以[十六进制](../Other/Hexadecimal/hexadecimal.md)表示，并以[小端字节](../Other/Little-endian/Little-Endian.md)顺序排列。

### 示例代码
这里有一些简单的Ruby代码，展示了如何挖掘区块（就像上面的那个）。它非常简单。唯一棘手的部分是在哈希之前以正确的格式获取[块头](../Block/block-header/block-header.md)数据。
```ruby
require 'digest/sha2'

# -----------------
# Utility Functions
# -----------------

# The hash function used in mining (convert hexadecimal to binary first, then SHA256 twice)
def hash256(data)
  binary = [data].pack("H*")
  hash1 = Digest::SHA256.digest(binary)
  hash2 = Digest::SHA256.hexdigest(hash1)
end

# Convert a number to fit inside a field that is a specific number of bytes e.g. field(1, 4) = 00000001
def field(data, size)
  hex = data.to_i.to_s(16).rjust(size * 2, '0')
end

# Reverse the order of bytes (happens often when working with raw bitcoin data)
def reversebytes(data)
  data.scan(/../).reverse.join
end

# ------------
# Block Header
# ------------

# Target (optional)
target = '000000000004864c000000000000000000000000000000000000000000000000'

# Block Header (Fields)
version    = '1'
prevblock  = '000000000002d01c1fccc21636b607dfd930d31d01c3a62104612a1719011250'
merkleroot = 'f3e94742aca4b5ef85488dc37c06c3282295ffec960994b2c0d5ac2a25a95766'
time       = '1293623863'  # Unixtime (29 Dec 2010, 11:57:43)
bits       = '1b04864c'
nonce      = 0             # 274148111

# Block Header (Serialized)
header = reversebytes(field(version, 4)) + reversebytes(prevblock) + reversebytes(merkleroot) + reversebytes(field(time, 4)) + reversebytes(bits)

# -----
# Mine!
# -----
loop do
  # hash the block header
  attempt = header + reversebytes(field(nonce, 4))
  result = reversebytes(hash256(attempt))

  # show result
  puts "#{nonce}: #{result}"

  # end if we get a block hash below the target
  if result.to_i(16) < target.to_i(16)
    break
  end
  
  # increment the nonce and try again...
  nonce += 1
end
```

### 指令
>bitcoin-cli getblocktemplate
该命令从节点的[内存池](../Node/Memory%20Pool/Memory%20Pool.md)中获取交易，并返回你开始挖掘新块所需的数据。
>>**注意**：此命令返回关键块头信息，如**前一个块**、**时间**和**位**，但你需要自己选择交易并构建[默克尔根](../Block/block-header/merkle-root/merkle-root.md)。

>bitcoin-cli submitblock [hexdata]  
将你的区块发送到网络。

>bitcoin-cli getmininginfo  
向你展示一些有趣的挖矿信息。  
如果你事先运行bitcoin-cli getblocktemplate命令，它还会向你显示有多少来自内存池的交易将被包含在下一个区块中。
```
$ bitcoin-cli getmininginfo

{
    "blocks": 790843,
    "currentblockweight": 3995794,
    "currentblocktx": 2630,
    "difficulty": 49549703178592.68,
    "networkhashps": 3.447695144262456e+20,
    "pooledtx": 50592,
    "chain": "main",
    "warnings": ""
}
```

## 链接
* https://en.bitcoin.it/wiki/Proof_of_work
* https://en.bitcoin.it/wiki/BIP_0022