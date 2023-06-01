# WIF私钥
一个易于分享的私钥格式。

一个私钥可以被转换成“钱包导入格式”，这基本上使得它更容易复制和移动（因为它更短，并包含用于检测错误的校验和）。

>**你永远不应该在网站上输入你的私钥，或使用由网站生成的私钥**。网站可以保存这些私钥，并使用它们来窃取你发送到其地址的任何比特币。

## 如何创建WIF私钥。
WIF私钥是一个标准的*私钥*，但加了一些额外的东西：

1. **版本字节**前缀 - 表示私钥要在哪个网络上使用。
    * 0x80 = 主网
    * 0xEF = 测试网
2. **压缩字节**后缀（可选）- 表示私钥是否用于创建压缩公钥。
   * 0x01
3. *校验*和 - 有助于检测输入私钥时的错误/打字错误。
然后将所有这些内容转换为*Base58*，缩短整个内容并使其更容易转录...
![WIF Private Key-1.png](img/WIF%20Private%20Key-1.png)

>一个WIF私钥只是另一种表示原始私钥的方式。如果你有一个WIF私钥，你总是可以将其转换回其原始格式。

代码
注意：此代码需要[checksum.rb](https://github.com/in3rsha/learnmeabitcoin-code/blob/master/checksum.rb)和[base58_encode.rb](https://github.com/in3rsha/learnmeabitcoin-code/blob/master/base58_encode.rb)函数。
```ruby
# Convert Private Key to WIF
privatekey = "ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2"
extended = "80" + privatekey + "01"
extendedchecksum = extended + checksum(extended)
wif = base58_encode(extendedchecksum)

puts wif
```
