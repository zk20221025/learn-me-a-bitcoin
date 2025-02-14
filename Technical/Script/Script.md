# <center>脚本</center>
<center>简单的编程语言。</center>

![Scpipt-1.png](img/Script-1%20(1).png)

**脚本**是一种简单的编程语言，用作[输出](../Transaction/Transaction%20Data/output/output.md)的锁定机制。

* 每个交易输出都会放置一个[**锁定脚本**](../Transaction/Transaction%20Data/output/scriptPubKey/scriptPubKey.md)。
* 必须提供**解锁脚本**来解锁输出（即在使用它作为新交易的[输入](../Transaction/Transaction%20Data/Input/input.md)时）。

只有当解锁脚本和锁定脚本一起运行后，验证通过，这个输出才会被解锁，可以作为新交易的输入使用。

## 什么是脚本语言？
脚本是一种非常基础的编程语言。它由两种类型的东西组成：

* **数据** - 例如公钥和签名。
* **OPCODES** - 对这些数据进行操作的简单函数。
  
以下是比特币中使用的传统P2PKH脚本的简单图示：

![Scpipt-2.png](img/Script-2%20(1).png)

>**提示**：这是[操作码](https://en.bitcoin.it/wiki/Script#Opcodes)的完整列表。

## 如何运行脚本？
>**完整的脚本需要从左到右按顺序运行。在运行时，它使用了一个称为“堆栈”的数据结构**。

1. 所有**数据**都会被推到**堆栈**上。

![Scpipt-3.gif](img/Script-3%20(1).gif)

2. **OPCODES**可以从堆栈中弹出元素，对它们执行某些操作，然后如果有需要，还可以选择性地将新元素“推”回到**堆栈**中。

![Scpipt-4.gif](img/Script-4%20(1).gif)

<center>DUP操作码复制堆栈顶部元素。</center>

## 什么使脚本有效？
**如果栈顶唯一剩下的元素是1（或更大）**,则脚本是有效的，

![Scpipt-5.gif](img/Script-5%20(1).gif)

>如果脚本满足以下条件，则无法执行：  
1.最终**堆栈为空**  
2.堆栈上剩下的唯一元素为**0**  
3.在执行结束时，堆栈上仍有**超过一个以上的元素**[^1]。  
4.脚本**提前退出**（例如，在[空数据](./NULL%20DATA/NULL%20DATA.md)脚本中使用**OP_RETURN**）。  

## 你可以在哪里找到比特币的脚本？

每个交易**输出**都放置了**锁定脚本**：

![Scpipt-6.png](img/Script-6%20(1).png)

如果你想在交易中花费某个**输入**，你必须为这个输入提供一个**解锁脚本**。

![Scpipt-7.png](img/Script-7%20(1).png)

每个节点都将结合并运行这两个脚本以确保它们有效。

>**首先运行解锁脚本!**  
尽管**解锁脚本**是在初始**锁定脚本**之后提供的，但实际上在运行这两个脚本时，解锁脚本实际上是**先运行**的。

![Scpipt-8.png](img/Script-8%20(1).png)

<center>当我们运行完整脚本时，解锁脚本会在锁定脚本之前执行。</center>

## 为什么我们要使用脚本？

* **问题**：为什么不只使用简单的公钥和签名比较来摆脱所有这些**操作码**和堆栈的麻烦呢？

* **答案**：因为**你可以使用不同的**操作码组合创建不同类型的锁定。

例如，你可以创建一些酷炫的锁定脚本：

### 1. 数学谜题
要使用此输出，需要提供**两个相加得到8**的数字。

![Scpipt-9.png](img/Script-9%20(1).png)

<center>这可能不是世界上最安全的锁定脚本。</center>

### 2. 哈希拼图
在这里，你只需要找到一个与锁定脚本**内部相同的哈希结果**。

![Scpipt-10.png](img/Script-10%20(1).png)

### 3. 哈希碰撞谜题
这是一个有趣的挑战。可以通过提供**两个不同的数据字符串，但它们产生相同的哈希结果**来解锁它。

换句话说，这可以鼓励人们寻找“[哈希碰撞](https://bitcointalk.org/index.php?topic=293382.0)”。

![Scpipt-11.png](img/Script-11%20(1).png)

这个脚本锁定的比特币可以在[**35Snmmy3uhaer2gTboc81ayCip4m9DT4ko**](https://learnmeabitcoin.com/explorer/address/35Snmmy3uhaer2gTboc81ayCip4m9DT4ko)找到。然而，该脚本已经被包装在P2SH锁定脚本中，因此你无法查看原始锁定脚本（除非有人解锁它们）。

>**这些锁定脚本不太常见。**[^2]。虽然这些脚本是有效的（并且可以开采到区块链上），但常规的比特币核心节点不会主动从其内存池中转发它们，这使得它们很难在第一时间被挖掘。

## 标准脚本
尽管可以使用各种组合的**操作码**创建各种不同的锁定脚本，但是在实际操作中，只有少数几种会被广泛使用。

* [P2PK](./P2PK/P2PK.md)（Pay To Pubkey）
* [P2PKH](./P2PKH/P2PKH.md)（Pay To Pubkey Hash）
* [P2MS](./P2MS/P2MS.md)（Pay To Multisig）
* [P2SH](./P2SH/P2SH.md)（Pay To Script Hash）
* [NULL DATA](./NULL%20DATA/NULL%20DATA.md)

![Scpipt-12.png](img/Script-12%20(1).png)

### 为什么节点不转发非标准脚本？

我知道，这很遗憾。

并非每种**操作码**的组合都经过测试。如果节点转发它们收到的每个非标准脚本，**就有可能被那些用验证时间长的脚本垃圾邮件攻击网络的人攻击。**[^3]。从而“堵塞”节点，使网络瘫痪。

另一方面，标准脚本经过了彻底的测试，可以快速验证。因此不转发非标准交易是一种**安全措施**。

>**非标准脚本是有效的，它们只是不会被主动转发。**  
即使非标准交易不会在内存池之间转发，**它仍然可以被挖掘到区块中**。节点不转发非标准交易，因为内存池可以在短时间内接收大量交易，而区块只能容纳有限数量的交易。  
如果想让非标准脚本交易被添加到区块链，需要直接发送给矿工或自己挖掘到区块链上。

## 总结
脚本是比特币中用于提供输出锁定机制的小型编程语言。

* 每个输出都被赋予一个“锁定脚本”。
* 然后，在想要花费该输出的交易中，必须提供一个“解锁脚本”。

当节点接收到花费交易时，它将组合这两个脚本并运行它们。如果在脚本完成后堆栈的顶部留下**1**（且没有其他东西），则脚本有效，输出可以被花费。

>脚本实际上是一个谓词。它只是一个可以计算结果为真或假的等式。谓词是一个又长又陌生的词，所以我称之为脚本。-[中本聪](https://bitcointalk.org/index.php?topic=195.msg1611#msg1611)

## 资源
* https://en.bitcoin.it/wiki/Script
* http://davidederosa.com/basic-blockchain-programming/bitcoin-script-language-part-one/
* https://bitcointalk.org/index.php?topic=293382.0

[^1]:[如果堆栈上有多个项目，是否可以花费脚本？](https://bitcoin.stackexchange.com/questions/92039/is-a-script-spendable-if-multiple-items-are-left-on-the-stack)
[^2]:[isStandard（）](https://github.com/bitcoin/bitcoin/blob/master/src/policy/policy.cpp)
[^3]:https://bitcoin.stackexchange.com/questions/73728/why-can-non-standard-transactions-be-mined-but-not-relayed/
