# <center>脚本</center>
<center>一个小型编程语言。</center>

![Scpipt-1.png](img/Script-1%20(1).png)

脚本是一种小型编程语言，用作[输出](../Transaction/Transaction%20Data/output/output.md)的锁定机制。

* 每个输出都会放置一个[锁定脚本](../Transaction/Transaction%20Data/output/scriptPubKey/scriptPubKey.md)。
* 必须提供一个**解锁脚本**来解锁输出（即在使用它作为[输入](../Transaction/Transaction%20Data/Input/input.md)时）。

如果一个完整的脚本（解锁+锁定）是有效的，则输出为“已解锁”，可以花费。

## 什么是脚本语言？
脚本是一种非常基本的编程语言。它由两种类型的东西组成：

* **数据** - 例如公钥和签名。
* **OPCODES** - 对数据进行操作的简单函数。
  
以下是比特币中使用的典型P2PKH脚本的简单图示：
![Scpipt-2.png](img/Script-2%20(1).png)

>**提示**：这是一个完整的[操作码](https://en.bitcoin.it/wiki/Script#Opcodes)列表。

## 如何运行脚本？
>**完整的脚本从左到右运行。在运行时，它使用了一个称为“堆栈”的数据结构**。

1. **数据**总是被推到**堆栈**上。

![Scpipt-3.gif](img/Script-3%20(1).gif)

2. **OPCODES**可以从堆栈中弹出元素，对其进行处理，然后可选择性地将新元素“推”回到**堆栈**中。

![Scpipt-4.gif](img/Script-4%20(1).gif)

<center>DUP操作码复制堆栈顶部元素。</center>

## 什么使脚本有效？
**如果栈顶且唯一剩下的元素是1（或更大）**,则脚本是有效的，
![Scpipt-5.gif](img/Script-5%20(1).gif)

>如果脚本满足以下条件，则无法执行：
1.最终堆栈为空
2.堆栈顶部元素为0
3.在执行结束时，堆栈上仍有超过一个以上的元素[^1]。
4.脚本过早退出（例如，在[空数据](./NULL%20DATA/NULL%20DATA.md)脚本中使用OP_RETURN）。

## 你可以在哪里找到比特币的脚本？

每个交易**输出**都放置了一个**锁定脚本**：
![Scpipt-6.png](img/Script-6%20(1).png)

每个你想在交易中花费的输入都必须提供一个解锁脚本：
![Scpipt-7.png](img/Script-7%20(1).png)

每个节点都将结合并运行这两个脚本以确保它们有效。

>**首先运行解锁脚本!**
尽管解锁脚本是在初始锁定脚本之后提供的，但实际上在运行这两个脚本时，我们将其放在第一位。

![Scpipt-8.png](img/Script-8%20(1).png)
<center>当我们运行完整脚本时，解锁脚本会在锁定脚本之前执行。</center>

## 为什么我们要使用脚本？

* **问题**：为什么不只使用简单的公钥和签名比较，摆脱所有这些操作码和堆栈的麻烦呢？

* **答案**：因为你可以使用不同的操作码组合创建不同类型的锁定。

例如，以下是一些酷炫的锁定脚本：

### 1. 数学谜题
要使用此输出，你需要提供**两个相加得到8**的数字。

![Scpipt-9.png](img/Script-9%20(1).png)
<center>这可能不是世界上最安全的锁定脚本。</center>

### 2. 哈希拼图
在这里，你只需要找到一个与锁定脚本**内部相同的哈希结果**。
![Scpipt-10.png](img/Script-10%20(1).png)

### 3. 哈希碰撞谜题
这是一个很酷的谜题。你可以通过提供两个不同的数据字符串来解锁它，这些字符串会产生**相同的哈希结果**。

换句话说，它是一种激励，鼓励人们寻找“[哈希碰撞](https://bitcointalk.org/index.php?topic=293382.0)”。
![Scpipt-11.png](img/Script-11%20(1).png)
这个脚本锁定的比特币可以在**35Snmmy3uhaer2gTboc81ayCip4m9DT4ko**找到。然而，该脚本已经被包装在一个P2SH锁定脚本中，因此你无法实际查看原始锁定脚本（除非有人解锁它们）。

>**这些锁定脚本不是标准的。**[^2]。虽然这些脚本是有效的（并且可以开采到区块链上），但典型的比特币核心节点不会从其内存池中中继它们，这使得它们很难在第一时间被挖掘。

## 标准脚本
尽管可以使用各种组合的**操作码**创建各种不同的锁定脚本，但大多数节点只会中继少量的“标准脚本”：

* [P2PK](./P2PK/P2PK.md)（支付到公钥）
* [P2PKH](./P2PKH/P2PKH.md)（支付到公钥哈希）
* [P2MS](./P2MS/P2MS.md)（支付到多重签名）
* [P2SH](./P2SH/P2SH.md)（支付到脚本哈希）
* [NULL DATA](./NULL%20DATA/NULL%20DATA.md)
![Scpipt-12.png](img/Script-12%20(1).png)

### 为什么节点不转发非标准脚本？

我知道，这很遗憾。

然而，并非每种**操作码**的组合都经过测试。因此，如果节点转发它们收到的每个非标准脚本，就会引入某人通过垃圾邮件攻击网络的风险，**这些脚本需要很长时间才能验证**[^3]。这可能会“阻塞”节点并使网络停止运行。

另一方面，标准脚本经过了彻底的测试，可以快速验证。因此，不转发非标准交易的整个过程只是一种**安全措施**。

>**非标准脚本是有效的，它们只是不会被主动转发。**
即使非标准交易不会在内存池之间转发，它仍然可以被挖掘到区块中。节点不转发非标准交易，因为内存池可以在短时间内接收大量交易，而区块只能容纳有限数量的交易。
因此，如果你想将具有非标准脚本的交易添加到区块链中， 你需要将其直接发送给矿工，他们将为你进行挖掘，或者自己将其挖掘到区块链中。

## 总结
脚本是比特币中用于提供输出锁定机制的小型编程语言。

* 每个输出都被赋予一个“锁定脚本”。
* 然后，在想要花费该输出的交易中，必须提供一个“解锁脚本”。
当节点接收到花费交易时，它将组合这两个脚本并运行它们。如果在脚本完成后堆栈的顶部留下**1**（且没有其他东西），则脚本有效，输出可以被花费。

>该脚本实际上是一个谓词。它只是一个等式，可以评估为真或假。谓词是一个长而不熟悉的词，所以我称之为脚本。-[中本聪](https://bitcointalk.org/index.php?topic=195.msg1611#msg1611)

## 资源
* https://en.bitcoin.it/wiki/Script
* http://davidederosa.com/basic-blockchain-programming/bitcoin-script-language-part-one/
* https://bitcointalk.org/index.php?topic=293382.0

[^1]:[如果堆栈上有多个项目，是否可以花费脚本？](https://bitcoin.stackexchange.com/questions/92039/is-a-script-spendable-if-multiple-items-are-left-on-the-stack)
[^2]:[isStandard（）](https://github.com/bitcoin/bitcoin/blob/master/src/policy/policy.cpp)
[^3]:https://bitcoin.stackexchange.com/questions/73728/why-can-non-standard-transactions-be-mined-but-not-relayed/
