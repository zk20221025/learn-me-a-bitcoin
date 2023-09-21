# <center>数字签名(签名和验证)</center>
<center>证明签名和公钥是由同一个私钥创建的。</center>

## 签名
数字签名有两个部分：

1. 随机部分。
2. 签名部分（私钥+我们为其创建数字签名的交易数据）。

### 1. 随机部分
首先生成一个随机数。然后将其与椭圆曲线上的生成点相乘（与生成[公钥](../../Public%20keys.md)时使用相同的生成点）。

![Signing & Verifying-1.png](img/Signing%20&%20Verifying-1%20(1).png)

数字签名中的**随机**部分是最终得到的曲线上的点。但我们只需取其x坐标：

![Signing & Verifying-2.png](img/Signing%20&%20Verifying-2%20(1).png)

我们将其简称为“r”。

![Signing & Verifying-3.png](img/Signing%20&%20Verifying-3%20(1).png)

>这与创建私钥和公钥相同。但这里这样做是为了向数字签名添加随机元素。

现在已经完成了数字签名的一半（随机部分），但还没有使用私钥来做任何事情。这就是第二部分签名部分的作用。

## 签名部分
首先，获取私钥，并将私钥与r（刚刚找到的曲线上随机点的x坐标）相乘。

![Signing & Verifying-4.png](img/Signing%20&%20Verifying-4%20(1).png)

接下来，我们需要包含想要签名的内容。这被称为**消息**。在比特币中，**消息**是包含想要解锁的输出的整个交易数据的哈希值。

![Signing & Verifying-5.png](img/Signing%20&%20Verifying-5%20(1).png)

包括交易哈希将签名绑定到一个交易中（因此不能在不同的交易中使用）。

最后，为了更好地衡量，将所有这些除以最初随机选择的数字：

![Signing & Verifying-6.png](img/Signing%20&%20Verifying-6%20(1).png)

然后，我们就得到了数字签名中至关重要的“签名”部分。简称其为 s。  

![Signing & Verifying-7.png](img/Signing%20&%20Verifying-7%20(1).png)  

D先生签名。

现在有趣的是……

如果有人要求我们证明我们是否知道公钥的私钥，可以给他数字签名（r和s）作为证据。

但是这个数字签名怎么能作为证据呢？

## 验证
>在比特币中，这整个签名都放在交易的“解锁脚本”部分。用来创建签名的私钥是与输出地址锁定的私钥相关联的。

![Signing & Verifying-8.png](img/Signing%20&%20Verifying-8%20(1).png)

为了验证数字签名是使用正确的私钥生成的，**接收数字签名的人需要使用这两个部分（公钥和签名）来找到椭圆曲线上的两个新点。**

### 点1

将**消息**按**s**分割。第一个点就是生成点乘以这个值：

![Signing & Verifying-9.png](img/Signing%20&%20Verifying-9%20(1).png)

### 点2
将**r**除以**s**。第二个点就是**公钥**乘以这个值：

![Signing & Verifying-10.png](img/Signing%20&%20Verifying-10%20(1).png)

### 最后...
现在，如果将这两个点相加，将得到曲线上的第三个点：

![Signing & Verifying-11.png](img/Signing%20&%20Verifying-11%20(1).png)

如果这个第三个点的x坐标与开始时随机点的x坐标相同，那么这就证明数字签名是使用与这个公钥相关联的私钥创建的。

![Signing & Verifying-12.png](img/Signing%20&%20Verifying-12%20(1).png)

## 相关页面
* [ECDSA](../../../../../../Technical/Keys/ECDSA/ECDSA.md)-有关如何在比特币中签署交易的技术细节。

## 资源
* [Bitcoin 101 - The Magic of Signing & Verifying](https://www.youtube.com/watch?v=U2bw_N6kQL8)一段优秀的介绍性视频，涵盖了签名生成和验证的实际数学内容。