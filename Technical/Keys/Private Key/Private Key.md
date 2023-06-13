# 私钥
一个随机生成的大数字。
![Private Key-1.png](img/Private%20Key-1%20(1).png)

私钥是一个随机数。它是一个256位的数字。

它被用作[公钥](../Public%20Key/Public%20Key.md)的源头。

>**永远不要使用由网站或其他人生成的私钥。始终在自己的计算机上秘密生成自己的私钥**。

## 生成私钥
>**生成私钥所需的只是可靠的随机源。**

Linux计算机上的一个简单的随机源是[/dev/urandom](https://linux.die.net/man/4/urandom)，它提供计算机的随机数据位。你所需要做的就是从中读取：
```ruby
# generate 256 bits of random data
urandom = File.open("/dev/urandom")    # urandom is a "file"
bytes = urandom.read(32)               # read 32 bytes from it (256 bits)
privatekey = bytes.unpack("H*")[0]     # the data is binary, so unpack it to hexadecimal

# print the private key
puts privatekey
```

**一个私钥可以是几乎任何256位的数字。**
当你创建一个公钥时，你的私钥会经过一个特殊的数学函数，而这个函数只能处理不超过256位的数字。其最大值为：
```
max = 115792089237316195423570985008687907852837564279074904382605163141518161494336

```
这个数字是**n-1**，其中**n**是比特币中使用的[椭圆曲线](https://cryptobook.nakov.com/digital-signatures/ecdsa-sign-verify-messages)上的点的数量。因此，当你生成一个256位的数字时，你需要检查它是否超过了这个最大值。

## 格式
十六进制私钥是32字节（64个字符）：
```
ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2
```
如果你为自己的个人使用生成私钥，那么这就是你所需要的全部。

### 钱包导入格式

然后，你可以将私钥转换为[WIF私钥](./WIF%20Private%20Key/WIF%20Private%20Key.md)，这样可以更轻松地复制和导入到[钱包](../../../Beginners/Getting%20Started/getting-started/getting%20started.md)中。