# NULL DATA
将数据存储在比特币交易中。
![nulldata-1.png](img/nulldata-1.png)
“NULL DATA”是一种标准的锁定脚本，您可以使用它来在**区块链上存储数据**。

>**警告**：此锁定脚本无法解锁，因此我不建议将其用作任何数量比特币的锁定。

## 它是如何工作的？

**OP_RETURN**操作码**立即结束脚本的执行**并将其标记为无效。[^1]
因此，任何带有NULL DATA锁定脚本的输出都是**无法花费**的：
![nulldata-2.png](img/nulldata-2.png)
您无法解锁此锁定脚本。

## 那么 NULL DATA 锁定脚本有什么用途呢？
您可以使用 NULL DATA 进行**数据存储**，因为*标准脚本*允许在末尾进行数据推送。

因此，如果您想向交易中添加任意数据，请包含一个额外的（空）输出，并在其上放置一个 NULL DATA 锁定脚本：
![nulldata-3.png](img/nulldata-3.png)

你可以在NULL DATA脚本中存储什么数据？
>一个交易只能包含一个空数据锁定脚本，才能被视为标准交易（也就是节点会转发它）。

你可以在空数据脚本中存储任何数据，最多可以存储80字节[^2]。但大多数人只是将**文本字符串**编码到其中。例如：
|scriptPubKey | OP_RETURN 68656c6c6f20776f726c64|
|---|---|

>**这是一个文本字符串吗？**
*交易数据*通常以*十六进制*格式表示，因此仅凭外观并不总是能立即确定数据的含义。
但如果它是文本，则可以通过将每个字节从十六进制转换为其[ASCII字符代码](http://www.asciitable.com/)来读取它：
```
hexadecimal = 68656c6c6f20776f726c64
ascii       = h e l l o   w o r l dv
```
ASCII字符代码表:
![nulldata-4.png](img/nulldata-4.png)
当然，你可以存储多达80个字节的任何数据（不仅仅是ASCII字符代码形式的文本）。你只需要知道如何解码它以查看它是什么。

## 你在哪里可以找到NULL DATA脚本？
任何人都可以使用NULL DATA脚本向交易中添加一些任意数据，因此你可以在区块链中找到它们散落在各处。

>在浏览区块链时，寻找空输出，因为NULL DATA脚本几乎总是放置在空输出上（因为锁定使输出无法花费）。

以下是一些人使用脚本存储文本字符串的示例（将鼠标悬停在数据推送上即可查看ASCII文本）：
Example 1 - “hello world”
|scriptPubKey|OP_RETURN 636861726c6579206c6f766573206865696469|
|---|---|

Example 2 - “charley loves heidi”
|scriptPubKey|OP_RETURN 636861726c6579206c6f766573206865696469|
|---|---|

Example 3 - “家族も友達もみんなが笑顔の毎日がほしい”
| scriptPubKey|OP_RETURN e5aeb6e6978fe38282e58f8be98194e38282e381bfe38293e381aae3818ce7ac91e9a194e381aee6af8ee697a5e3818ce381bbe38197e38184|
|---|---|

## 为什么要将NULL DATA引入标准脚本？

在**比特币0.9.0**中引入了NULL DATA锁定脚本，以妥协地允许人们在交易中包含任意数据。

你知道，在NULL DATA脚本出现之前，人们通过使用现有的标准锁定脚本来向交易中添加数据。例如，您可以始终使用*P2PKH*脚本，将您的任意数据放在应该放置**散列公钥**的位置：

|scriptPubKey|OP_DUP OP_HASH160 00000000006c6561726e6d6561626974636f696e OP_EQUALVERIFY OP_CHECKSIG|
|---|---|

然而，这种方法的问题在于它看起来像一个标准的锁定脚本，因此输出将存储在UTXO数据库中，这是一种浪费RAM的方式，因为输出实际上是**不可花费**的。

因此，这就是NULL DATA的作用所在：
|scriptPubKey|OP_RETURN 6c6561726e6d6561626974636f696e|

这个脚本包含**OP_RETURN**，这意味着它可以被**证明是不可花费**的。因此，没有必要将其存储在UTXO数据库中，这可以节省宝贵的RAM空间。

>在区块链上进行数据存储并不理想，但**你无法阻止人们这样做**。因此，NULL DATA脚本作为比以前方法更为合理的数据存储替代方案。
>>关于OP_RETURN：在0.9版本中，关于OP_RETURN功能和区块链中的数据存在一些混淆和误解。这个改变并不是支持在区块链中存储数据。OP_RETURN的改变创建了一个可证明可裁剪的输出，以避免存储任意数据（如图像）作为永远不能花费的交易输出的数据存储方案，从而增加比特币的UTXO数据库负担。
[比特币0.9.0版本发布说明](https://bitcoin.org/en/release/v0.9.0)

## 注意事项
**必须对OP_RETURN进行评估才能使脚本失败。**
脚本中存在**OP_RETURN**操作码并不会自动使脚本无效。

例如，在下面的脚本中，**OP_RETURN**在永远不会被评估的分支中，因此脚本是有效的：
![nulldata-5.png](img/nulldata-5.jpg)

您可以在此处查看此脚本在*P2SH*中的*示例*。

感谢[Vincenzo Iovino](https://sites.google.com/site/vincenzoiovinoit)指出这一点。

[^1]:https://bitcoin.stackexchange.com/questions/49303/op-return-marks-transaction-or-output-as-invalid
[^2]:https://bitcoin.org/en/release/v0.12.0#relay-any-sequence-of-pushdatas-in-opreturn-outputs-now-allowed
