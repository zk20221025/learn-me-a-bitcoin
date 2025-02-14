# <center>UTXO</center>
<center>未花费的交易输出</center>

![UTXO-1.png](img/UTXO-1-svg.png)

[输出](../Transaction%20Data/output/output.md)在交易中被“用完”后就不能再次使用。

未花费的输出仍然<span style="color: Orange">有效</span>。只要你可以解锁它们就可以**用于新的交易**。这就是已花费的输出和**未花费的输出**（UTXO）之间存在区别的原因。
## UTXO 在哪里使用？

### 1. 验证交易
[节点](../../../Beginners/How%20Bitcoin%20Works/1.Network/Nodes/Nodes.md)将通过检查其输入是否已经被花费来验证它接收到的交易。

因此，如果你想创建自己的比特币交易，你必须在交易[输入](../Transaction%20Data/Input/input.md)中使用UTXO集。

![UTXO-2.png](img/UTXO-2-svg.png)

<center>这个新交易是指未花费的输出。</center>

然而，如果你尝试使用已经在另一个交易中被使用过的交易输出，你的交易将会被节点拒绝。

![UTXO-3.png](img/UTXO-3-svg.png)

<center>这个新交易是指已经被花费的输出。节点将拒绝这个交易。</center>

### 2. 地址余额

如果你想计算一个[地址](../../Keys/Address/Address.md)的余额，需要将所有[锁定](../Transaction%20Data/output/scriptPubKey/scriptPubKey.md)在该地址上的未花费输出相加。

![UTXO-4.png](img/UTXO-4-svg.png)

## UTXO数据库
由于UTXO被用于验证节点接收到的每个交易，因此UTXO被存储在一个特定的数据库中。
```
~/.bitcoin/chainstate/
```
这种设置方式使得你的节点能够通过检查一笔交易的输入是否在UTXO数据库中存在，从而快速地验证交易。

## 注释
>**gettxoutsetinfo**

此命令会给你提供有关此UTXO数据库的统计信息:
```
$ bitcoin-cli gettxoutsetinfo

{
  "height": 454974,
  "bestblock": "000000000000000001afbfe1209aee1882e8a8930bc1e020c4a37d98f72640d7",
  "transactions": 17707300,
  "txouts": 45845906,
  "bytes_serialized": 1791822745,
  "hash_serialized": "ed1bdcca4e08eb8270800d08b2f9dd7ce926f4ff9b1d7304b71c7a075a69098c",
  "total_amount": 16187023.59866495
}
```
>当你运行bitcoind时，UTXO数据库也会被加载到RAM中，这有助于加快交易的验证速度。  
**什么是RAM?**  
RAM是**随机存取存储器**。  
RAM是一种可以快速读取数据的临时存储设备，比从硬盘读取数据更快。  
>>可以在比特币的配置文件**bitcoin.conf**中，通过**dbcache=**选项来更改用于UTXO数据库的RAM数量（默认为100MB）。增加这个值将加快节点验证新交易的时间。

## 资源
### 链接
* [UTXO database size chart - statoshi.info](http://statoshi.info/dashboard/db/unspent-transaction-output-set?panelId=8&fullscreen)
* [http://gavinandresen.ninja/utxo-uhoh](http://gavinandresen.ninja/utxo-uhoh)
* [https://bitcoin.org/en/developer-reference#gettxoutsetinfo](https://bitcoin.org/en/developer-reference#gettxoutsetinfo)