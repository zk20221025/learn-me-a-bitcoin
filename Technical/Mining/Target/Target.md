# <center>目标值</center>
<center>开采区块所需的数字如下。</center>

![target-1.png](img/target-1%20(1).png)

目标值是在[挖掘](../Mining.md)中使用的数字，[区块哈希值](../../Block/block-hash/block-hash.md)必须低于该数字才能将该区块添加到[区块链](../../Blockchain/blockchain.md)中。

目标值每2016个区块（大约两周）进行一次调整，以尝试确保**平均每10分钟**挖掘出一个区块。这样可以保证区块产生的时间一致，并使新比特币的发行量保持稳定。
>为了应对硬件速度的增加和随时间推移人们对运行节点的兴趣变化，通过调整工作量证明的难度，使得每小时产生的区块数量保持在一个平均水平。以达到每小时平均区块数量的目标值。如果区块生成过快，难度会增加。-[中本聪](https://nakamotoinstitute.org/bitcoin/)

**当前目标值**:
```
0000000000000000000561020000000000000000000000000000000000000000
```
[区块 800,564](https://learnmeabitcoin.com/explorer/blockchain/800564)
>目标值是[十六进制](../../Other/Hexadecimal/hexadecimal.md)表示的，但它仍然是一个数字。

## 1. 目标值什么时候调整?
目标值**每2016个区块**进行一次调整。这大约是每两周一次（因为两周内有20160分钟）。

## 2. 目标值如何计算？
第一个区块的目标值设置为：
```
00000000ffff0000000000000000000000000000000000000000000000000000
```
区块 [0](https://learnmeabitcoin.com/explorer/blockchain/0)

>这个初始（和最大）目标值被直接写入到每一个比特币节点的源代码中[^1]。这可能是Satoshi Nakamoto对于一个足够困难的目标值的最佳猜测，这将导致新区块之间的10分钟间隔。

![target-2.png](img/target-2%20(1).png)

每经过2016个区块，每个节点将查看最近2016个区块之间的时间，然后计算它们的平均挖掘速度是否快于或慢于10分钟。

![target-3.png](img/target-3%20(1).png)

>每个区块的[区块头](../../Block/block-header/block-header.md)包含一个时间戳。这个时间戳采用[Unix](https://en.wikipedia.org/wiki/Unix_time)时间，即自1970年1月1日以来的秒数。

如果在此期间挖掘的区块速度比每10分钟一次更快，目标值将向下调整，以使在下一个区块周期中更难达到目标值。

![target-4.png](img/target-4%20(1).png)

相反地，如果挖掘区块速度慢于每10分钟一次，目标值会上调，以便在下一个区块周期内更容易达到目标值。

![target-5.png](img/target-5%20(1).png)

为了保证新区块的生成速度在大约每10分钟一个，每个网络节点会定期地重新计算和调整挖矿难度，以适应网络中矿工数量的变化。

请见本页底部的[代码](#代码)，这是一个真实的示例。

### 是什么导致区块的挖掘速度比每10分钟快或慢？
首先，[挖矿](../Mining.md)是不可预测的，因此你永远不知道矿工何时会找到区块哈希值小于当前目标值的下一个区块。

其次（也是最重要的），矿工可以随时加入和离开网络，这会影响在10分钟内挖掘新区块的可能性。网络上的矿工越多，进行的哈希运算就越多，新区块在10分钟内被挖掘的可能性就越大。

## 3. 所有节点共享同一个目标值吗？
网络上的每个节点都独立运作，因此没有统一的目标值在网络中传递。

然而，因为所有节点都采用最长的区块链作为它们的区块链，它们最终会计算出与其他节点相同的目标值，所以**这使得所有节点最终都会共享相同的当前目标值**。

例如，当你第一次运行比特币时，你的节点会下载初始的区块并在此过程中计算目标值，因为你接收的区块和其他人一样，所以你最终计算出的目标值也会和其他人一样。

![target-6.png](img/target-6%20(1).png)

此外，所有节点持续共享区块链的相同视图（因为它们始终采用[可用的最长区块链](../../Blockchain/longest-chain/longest-chain.md)）。因此，随着新区块的不断挖掘，所有的节点也会持续计算出相同的目标值。

![target-7.png](img/target-7%20(1).png)

尽管节点独立计算目标值，但它们每个节点都共享同一条区块链，并计算出相同的目标值。

>每个人都使用相同的链数据进行相同的计算，因此他们在链上的同一环节获得相同的结果。-[中本聪](https://satoshi.nakamotoinstitute.org/emails/cryptography/10/#selection-73.260-73.383)

## 4. 为什么比特币使用目标值？
目标值是**调节将新区块添加到区块链的速度**。

这有两个好处：

### **i. 它给区块提供了在网络中传播的时间。**

理想的情况是矿工尽可能地在为扩展相同的区块链而工作。为了实现这一点，我们需要在下一个块被开采之前留出时间让新区块在网络上传播。

![target-8.png](img/target-8%20(1).png)

如果区块的挖出速度比它们在网络上广播的速度更快，会导致网络中出现多个并行的区块链。其中只有一个将成为[最长的链](../../Blockchain/longest-chain/longest-chain.md)，因此在这些被舍弃的区块链上挖矿的矿工就相当于浪费了他们的计算资源，最终由于[链重组](../../Blockchain/chain-reorganisation/chain-reorganisation.md)而被抛弃。

![target-9.png](img/target-9%20(1).png)

区块之间的时间延迟使它们能够在网络中传播，以便更多的矿工可以采用最长可用的区块链，从而将网络的挖矿能力集中在扩展相同的区块链上。

![target-10.png](img/target-10%20(1).png)

>如果广播实际上比预期的慢，为了避免浪费资源，目标区块生成时间可能需要增加。我们希望区块的传播时间要比生成时间短得多，否则节点将花费太多时间处理过时的区块。-[中本聪](https://satoshi.nakamotoinstitute.org/emails/cryptography/12/#selection-63.209-63.512)

### **ii. 新比特币的持续发行。**
比特币是一种货币，因此以固定比率发现的新比特币数量可以提供稳定性。

![target-11.png](img/target-11%20(1).png)

<center>由于目标值设定，你可以确信只要网络运行，新的比特币将以可预测的速率铸造出来。</center>

>硬币必须以某种方式进行初始分配，恒定的速率似乎是最好的公式。-[中本聪](https://satoshi.nakamotoinstitute.org/emails/cryptography/5/#selection-57.0-57.97)

## 5.为什么区块之间要间隔10分钟？
我认为除了Satoshi之外，没有人知道为什么会选择**10分钟**的时间间隔。

我猜测这既是一个足够长的时间，可以让区块在网络中传播（来最大限度地减少[链重组](../../Blockchain/chain-reorganisation/chain-reorganisation.md)），同时又不必等待太长时间才能将新交易挖掘到区块链上。**10分钟**是一个很好记的时间。

## 6. 你可以在哪里找到目标值？
目标值被存储在每个[区块头](../../Block/block-header/block-header.md)中的[位](../../Block/block-header/bits/bits.md)字段中。

![target-11.png](img/target-12%20(1).png)

<center>目标值以紧凑格式存储以节省空间。</center>

## **命令**
>**bitcoin-cli getblocktemplate**  
要想获取当前目标值，最简单方法就是请求一个用于[挖掘](../Mining.md)的区块模板，它就会包含当前目标值：
```
$ bitcoin-cli getblocktemplate '{"rules": ["segwit"]}' | grep target
  "target": "00000000000000000005dd010000000000000000000000000000000000000000",
```

>**bitcoin-cli getdifficulty**  
同样，你可以询问当前的[难度](../../../Beginners/How%20Bitcoin%20Works/2.Mining/3.Difficulty/Difficulty.md)，然后将其转换为目标值：
```
$ bitcoin-cli getdifficulty
49549703178593
```

>**bitcoin-cli getblockheader**  
另外，正如之前提到的，在挖掘时的目标值以[位](../../Block/block-header/bits/bits.md)格式存储在每个[区块头](../../Block/block-header/block-header.md)中（你可以将其转换为完整的目标值）。

```
$ bitcoin-cli getblockheader 000000000000000002e9533a4fe03bb251b3fdb30ffaa384aad133b7fae594cf
{
  "hash": "000000000000000002e9533a4fe03bb251b3fdb30ffaa384aad133b7fae594cf",
  "confirmations": 207763,
  "height": 401184,
  "version": 4,
  "versionHex": "00000004",
  "merkleroot": "b4afa0502a55fdfa4a5cda6ea2bc546ba527d276ea9c7e848b8cd478cd9b6607",
  "time": 1457133956,
  "mediantime": 1457132303,
  "nonce": 3778923481,
  "bits": "1806f0a8",
  "difficulty": 158427203767.3917,
  "chainwork": "00000000000000000000000000000000000000000012da32364c8f47d519604d",
  "nTx": 845,
  "previousblockhash": "000000000000000003c4a4d9c62b3a7f4893afe14eef8a6a377229d23ad4b1ea",
  "nextblockhash": "00000000000000000037ab74c8db9b7a9d7de039210d8eafaeb44ba35adfb624"
}
```

>>**提示**：你可以使用以下命令获取链中顶部区块的目标值：**$ bitcoin-cli getblockheader $(bitcoin-cli getblockhash \$(bitcoin-cli getblockcount))**

>>**注意**：通过请求挖矿模板得到的难度目标，是针对当前这个区块的，而不一定是下一个即将产生的区块所使用的难度目标

## **代码**
这是用于计算新目标值的Ruby代码示例。该代码使用区块[401,184](https://learnmeabitcoin.com/explorer/blockchain/401184)和[403,199](https://learnmeabitcoin.com/explorer/blockchain/403199)的时间戳来计算出区块[403,200](https://learnmeabitcoin.com/explorer/blockchain/403200)的新目标值。
```ruby
# 403,200 - NEW TARGET
# 403,199              | last block
#                      |
#                      |
#                      |
# 401,184 - NEW TARGET | first block (target = 0x000000000000000006f0a8000000000000000000000000000000000000000000)

# 1. Get the timestamps for the first and last block in the target adjustment period
first = 1457133956 # block 401,184
last  = 1458291885 # block 403,199

# 2. Work out the ratio of the actual time against the expected time
actual = last - first     # 1157929 (number of seconds between first and last block)
expected = 2016 * 10 * 60 # 1209600 (number of seconds expected between 2016 blocks)
ratio = actual.to_f / expected.to_f

# 3. Limit the adjustment by a factor of 4 (to prevent massive changes from one target to the next)
ratio = 0.25 if ratio < 0.25
ratio = 4 if ratio > 4

# 4. Multiply the current target by this ratio to get the new target
current_target = 0x000000000000000006f0a8000000000000000000000000000000000000000000
new_target = (current_target * ratio)

# 5. Don't let the target go above the maximum target
max_target = 0x00000000ffff0000000000000000000000000000000000000000000000000000
new_target = max_target if new_target > max_target

# 5. Truncate the target, because the official target is the truncated "bits" format stored in the block header
# This code is a bit rough, because it's working with strings when I should really be working with actual bytes.
new_target = new_target.to_i.to_s(16) # convert from decimal to hexadecimal
new_target = new_target.size % 2 != 0 ? '0' + new_target : new_target # make sure it's an even number of characters (i.e. bytes)
truncated = new_target.scan(/../).each_with_index.map { |byte, i| byte = i >= 3 ? "00" : byte }.join # set all bytes apart from first 3 to zeros
# e.g. 6a4c316c01f354000000000000000000000000000000000 <- full precision
# e.g. 6a4c3000000000000000000000000000000000000000000 <- official target

# 6. Display the full target (with leading zeros)
target = truncated.rjust(64, '0')
puts target
# 000000000000000006a4c3000000000000000000000000000000000000000000
```

>**提示**：你可以使用**bitcoin-cli getblockheader <hash>**获取区块的时间戳（ 你可以使用**bitcoin-cli getblockhash <height>**获取特定高度的区块哈希）。

>**官方目标值**：官方目标值是存储在区块头中的截断目标值（在[位](../../Block/block-header/bits/bits.md)字段中），因此它不是在调整前一个目标值后获得的精确的目标值。

>**偏差**：你可能会注意到新目标值是使用2015个块的时间计算的（而不是你所预期的2016个）。这是代码中存在的实现错误，今天仍然存在。

>**4的倍数**：目标值调整受到4的倍数的限制，以防止从一个目标值到下一个目标值的过度大的调整。

## 链接
* [pow.cpp](https://github.com/bitcoin/bitcoin/blob/master/src/pow.cpp)-**CalculateNextWorkRequired()**函数会调整目标值。
* [chainparams.cpp](https://github.com/bitcoin/bitcoin/blob/master/src/chainparams.cpp)-原始目标值（也是可能的最高目标值）由**consensus.powLimit**设定。
* https://en.bitcoin.it/wiki/Target
* [比特币的目标值是如何设置的？由谁来完成？(bitcoin.stackexchange.com)](https://bitcoin.stackexchange.com/questions/77875/how-is-the-target-in-bitcoin-set-who-does-this)
* [为什么不在每个区块上重新定位？ (bitcoin.stackexchange.com)](https://bitcoin.stackexchange.com/questions/9305/why-not-retarget-on-every-block)
* [为什么目标块时间选择为10分钟？](https://bitcoin.stackexchange.com/questions/1863/why-was-the-target-block-time-chosen-to-be-10-minutes)
* https://bitcoin.stackexchange.com/questions/1511/gaming-the-off-by-one-bug-difficulty-re-target-based-on-2015-instead-of-2016

## 谢谢
感谢[David Harding](https://dtrt.org/)指出，你可以通过**bitcoin-cli getblocktemplate**直接从bitcoind获取目标值。

[^1]:https://github.com/bitcoin/bitcoin/blob/master/src/chainparams.cpp#L106