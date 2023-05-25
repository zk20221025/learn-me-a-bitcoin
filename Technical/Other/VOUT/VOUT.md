# VOUT
比特币交易中输出的向量。
![VOUT-1.png](img/VOUT-1.png)

一个vout基本上是一个交易输出的索引号码。

## 用途

您可以使用txid和vout来唯一选择一个输出，在新交易中用作输入。
![VOUT-2.png](img/VOUT-2.png)
输出编号2，请。

## 注意事项
>在编程中，计数从0开始。因此，如果我们想使用现有交易的第一个输出，我们将vout设置为0。（第二个输出将为1。）

>**vout**中的“v”代表向量，因为比特币源代码中的输出存储在[向量](http://www.cplusplus.com/reference/vector/vector/)数据结构中（基本上是一个[数组](https://www.go4expert.com/articles/array-vector-stack-data-structures-t27921/)）。

## 链接

* https://www.quora.com/Why-do-array-indexes-start-with-0-zero-in-many-programming-languages
