# 目标
你需要达到的数字以挖掘一个区块。
![target-1.png](img/target-1%20(1).png)

目标是在*挖掘*中使用的一个数字，*区块哈希*必须低于该数字才能将该区块添加到*区块链*中。

目标每2016个区块（大约两周）进行一次调整，以尝试确保**平均每10分钟**挖掘一个区块。因此，它创建了区块之间的一致时间和新比特币的一致发行（通过区块奖励）。
>为了弥补硬件速度增加和随时间变化的节点运行兴趣的差异，工作证明难度由移动平均值确定，旨在每小时生成平均数量的区块。如果生成得太快，难度会增加。-[中本聪](https://nakamotoinstitute.org/bitcoin/)

**当前目标**:
```
00000000000000000005dd010000000000000000000000000000000000000000
```
*Block 788,837*
>目标是*十六进制*表示的，但它仍然是一个数字。

## 1. 目标什么时候调整?
目标值每2016个区块进行一次调整。这大约是每两周一次（因为两周内有20160分钟）。

以下是比特币历史上每次目标调整的日期和目标值：
比特币目标历史

## 2. 目标如何计算？
第一个区块的目标已设定为：
```
00000000ffff0000000000000000000000000000000000000000000000000000
```
Block *0*

>这个初始（和最大）目标值是硬编码到每个比特币节点的源代码中的[^1]。这可能是Satoshi对于一个足够困难的目标的最佳猜测，这个目标会导致新块之间的10分钟间隔。

![target-2.png](img/target-2%20(1).png)

每经过2016个区块，每个节点将查看最近2016个区块的时间，然后计算它们的平均挖掘速度是否快于或慢于10分钟。
![target-3.png](img/target-3%20(1).png)

>每个区块的*区块头*包含一个时间戳。这个时间戳以[Unix](https://en.wikipedia.org/wiki/Unix_time)时间表示，即自1970年1月1日以来的秒数。

如果在此期间挖掘的区块速度比每10分钟更快，目标将调整向下，以使在下一个区块周期中更难达到目标。
![target-4.png](img/target-4%20(1).png)

相反地，如果挖掘区块速度慢于每10分钟，目标值会上调，以便在下一个区块周期内更容易达到目标值以下。
![target-5.png](img/target-5%20(1).png)

因此，每个节点定期重新计算目标，以在矿工随时间加入和离开网络的情况下强制执行新块之间的10分钟间隔。
Try it! - Target Recalculator
```
```

目标最多可以调整4倍。

请见本页底部的代码[^code]，这是一个真实的示例。

### 是什么导致区块的挖掘速度比每10分钟快或慢？
首先，*挖矿*是不可预测的，因此您永远不知道矿工何时会找到下一个区块哈希小于当前目标的区块。

其次（也是最重要的），矿工可以随时加入和离开网络，这会影响在10分钟内挖掘新区块的可能性。网络上有更多的矿工，就会有更多的哈希运算，新区块在10分钟内被挖掘的可能性就越大。

## 3. 所有节点共享同一个目标吗？
网络上的每个节点都独立运作，因此没有“单一目标值”在网络中传递。

然而，因为所有节点都采用最长的区块链作为它们的区块链，它们最终会计算出与其他人相同的目标值，因此**实际上所有节点最终都会共享相同的当前目标值**。

例如，当您第一次运行比特币时，您的节点将执行初始块下载并在执行过程中计算目标值。由于您收到的区块与其他人相同，因此您最终也会计算出相同的当前目标值。
![target-6.png](img/target-6%20(1).png)

此外，所有节点持续共享区块链的相同视图（因为它们始终采用*可用的最长区块链*）。因此，随着每个新块的挖掘，节点也将持续计算相同的目标值。
![target-7.png](img/target-7%20(1).png)

因此，尽管节点独立计算目标，它们每个节点都共享相同的区块链，并计算相同的目标值。

>每个人都使用相同的链数据进行相同的计算，因此他们在链的相同链接处获得相同的结果。-[中本聪](https://satoshi.nakamotoinstitute.org/emails/cryptography/10/#selection-73.260-73.383)

## 4. 为什么比特币使用目标？
目标是**调节将新区块添加到区块链的速度**。

这有两个好处：

### **i. 它给区块时间在网络中传播。**

最好让矿工尽可能地在扩展相同的区块链上工作。为了实现这一点，我们需要允许新区块在下一个区块被挖掘之前在网络中传播的时间。
![target-8.png](img/target-8%20(1).png)

如果挖掘出的区块比它们在网络中传播的速度更快，那么将导致矿工经常建立竞争链。其中只有一个将成为*最长的链*，因此一些矿工将浪费能源努力在竞争链上进行建设，最终由于*链重新组织*而被抛弃。
![target-9.png](img/target-9%20(1).png)

因此，区块之间的时间延迟使它们能够在网络中传播，以便更多的矿工可以采用最长可用的区块链，从而将网络的挖矿能力集中在扩展相同的区块链上。
![target-10.png](img/target-10%20(1).png)

如果广播实际上比预期更慢，目标区块生成时间可能需要增加，以避免浪费资源。我们希望区块通常能够在生成时间的远远短于传播时间内传播，否则节点将花费太多时间在过时的区块上工作。-[中本聪](https://satoshi.nakamotoinstitute.org/emails/cryptography/12/#selection-63.209-63.512)

### **ii. 新比特币的持续发行。**
比特币是一种货币，因此固定引入系统的新比特币数量可以提供稳定性。
![target-11.png](img/target-11%20(1).png)
由于目标设定，您可以确信只要网络运行，新的比特币将以可预测的速率铸造出来。

>硬币必须以某种方式进行初始分配，恒定的速率似乎是最好的公式。-[中本聪](https://satoshi.nakamotoinstitute.org/emails/cryptography/5/#selection-57.0-57.97)

## 5.为什么区块之间要间隔10分钟？
我认为除了Satoshi之外，没有人知道为什么会选择**10分钟**的时间。

我猜测这个时间足够长，可以让区块在网络中传播（以最小化链重组），同时又不必等待太长时间才能将新交易挖掘到区块链上。而10是一个漂亮的整数。

## 6. 你可以在哪里找到目标？
目标被存储在每个*区块头*中的*比特*字段中。
![target-11.png](img/target-12%20(1).png)
目标以紧凑格式存储以节省空间。

## **命令**
>bitcoin-cli getblocktemplate
这是获取当前目标的最简单方法。当您请求一个*挖掘*的区块模板时，它也会返回当前目标：
```
$ bitcoin-cli getblocktemplate '{"rules": ["segwit"]}' | grep target
  "target": "00000000000000000005dd010000000000000000000000000000000000000000",
```

>bitcoin-cli getdifficulty
同样，您可以询问当前的难度，然后将其转换为目标难度：
```
$ bitcoin-cli getdifficulty
49549703178593
```

>bitcoin-cli getblockheader
另外，如上所述，在挖掘时的目标以位格式存储在每个块头中（您可以将其转换为完整的目标值）。
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

>>**提示**：您可以使用以下命令获取链中顶部块的目标值：$ bitcoin-cli getblockheader $(bitcoin-cli getblockhash $(bitcoin-cli getblockcount))

>>**注意**：这将为您提供该块的目标值，而不一定是用于下一个块的当前目标值。

## **代码**
这是用于计算新目标值的Ruby代码示例。该代码使用*401,184*和*403,199*块的时间戳来计算*403,200*块的新目标值。
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

>**提示**：您可以使用bitcoin-cli getblockheader <hash>获取区块的时间戳（您可以使用bitcoin-cli getblockhash <height>获取特定高度的区块哈希）。

>**官方目标**：官方目标是存储在块头中的截断目标（在比特字段中），因此它实际上不是调整前一个目标后获得的完整精度目标。

>**偏差**：您可能会注意到新目标是使用2015个块的时间计算的（而不是您预期的2016个）。这是代码中存在的实现错误。

>**4的倍数**：目标调整受到4的倍数的限制，以防止从一个目标到下一个目标的过度大的调整。

## 链接
* [pow.cpp](https://github.com/bitcoin/bitcoin/blob/master/src/pow.cpp)-CalculateNextWorkRequired()函数会调整目标。
* [chainparams.cpp](https://github.com/bitcoin/bitcoin/blob/master/src/chainparams.cpp)-原始目标值（也是可能的最高目标值）由共识.powLimit设定。
* https://en.bitcoin.it/wiki/Target
* [比特币的目标是如何设置的？由谁来完成？(bitcoin.stackexchange.com)](https://bitcoin.stackexchange.com/questions/77875/how-is-the-target-in-bitcoin-set-who-does-this)
* [为什么不在每个区块上重新定位？ (bitcoin.stackexchange.com)](https://bitcoin.stackexchange.com/questions/9305/why-not-retarget-on-every-block)
* [为什么目标块时间选择为10分钟？](https://bitcoin.stackexchange.com/questions/1863/why-was-the-target-block-time-chosen-to-be-10-minutes)
* https://bitcoin.stackexchange.com/questions/1511/gaming-the-off-by-one-bug-difficulty-re-target-based-on-2015-instead-of-2016

## 谢谢
感谢[David Harding](https://dtrt.org/)指出，您可以通过bitcoin-cli getblocktemplate直接从bitcoind获取目标。

[^1]:https://github.com/bitcoin/bitcoin/blob/master/src/chainparams.cpp#L106