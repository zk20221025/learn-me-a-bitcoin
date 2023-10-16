# <center>私钥</center>
<center>一个大的随机生成的数字。</center>

![Private Key-1.png](img/Private%20Key-1%20(1).png)

**私钥**是一个随机数。它是一个**256位**的数字。

私钥是生成[公钥](../Public%20Key/Public%20Key.md)的基础。

>**永远不要使用由网站或其他人生成的私钥。始终在自己的计算机上秘密生成自己的私钥**。

## 生成私钥
>**生成私钥所需的只是一个可靠的随机源。**

在Linux计算机上，一个简单的随机源是[**/dev/urandom**](https://linux.die.net/man/4/urandom)，它是一个提供随机数据的设备。你可以通过读取这个设备来获取随机数，从而生成私钥：
```ruby
# generate 256 bits of random data
urandom = File.open("/dev/urandom")    # urandom is a "file"
bytes = urandom.read(32)               # read 32 bytes from it (256 bits)
privatekey = bytes.unpack("H*")[0]     # the data is binary, so unpack it to hexadecimal

# print the private key
puts privatekey
```

**私钥几乎可以是任何256位的数字。**  
创建公钥时，私钥会经过一个特殊的数学函数处理，而这个函数只能处理不超过256位的数字。其最大值为：
```
max = 115792089237316195423570985008687907852837564279074904382605163141518161494336

```
这个数字是**n-1**，其中**n**是比特币中使用的[椭圆曲线](https://cryptobook.nakov.com/digital-signatures/ecdsa-sign-verify-messages)上的点的数量。因此，当生成一个256位的数字时，你需要检查它是否超过了这个最大值。

## 格式
一个十六进制私钥由32字节（或64个字符）组成：
```
ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2
```
如果你要为自己的个人使用生成私钥，那么这就是你所需要的全部。

### 钱包导入格式

然后，为了复制和导入到[钱包](../../../Beginners/Getting%20Started/getting-started/getting%20started.md)中，你可以将私钥转换为[WIF私钥](./WIF%20Private%20Key/WIF%20Private%20Key.md)。