# <center>WIF私钥</center>
<center>一个易于分享的私钥格式。</center>

[私钥](../Private%20Key.md)可以被转换成“钱包导入格式”，这使得它**更容易复制和移动**（因为它更短，并包含用于检测错误的校验和）。

>**你永远不应该在网站上输入你的私钥，或使用由网站生成的私钥**。网站可以保存这些私钥，并使用它们来窃取你发送到其地址的任何比特币。

## 如何创建WIF私钥。
WIF私钥是标准[私钥](../Private%20Key.md)，但加了一些额外的东西：

1. **版本字节**前缀 - 表示私钥要在哪个网络上使用。
    * 0x80 = 主网
    * 0xEF = 测试网
2. **压缩字节**后缀（可选）- 表示私钥是否用于创建[压缩公钥](../../Public%20Key/Public%20Key.md)。
   * 0x01
3. **[校验和](../../Checksum/Checksum.md)** - 在你键入私钥时，用于检测错误输入或者打字错误。

然后将这些内容全部转换为[Base58](../../Base58/Base58.md)，这样就缩短了整个内容，并使其更容易转录...

![WIF Private Key-1.png](img/WIF%20Private%20Key-1.png)

>WIF私钥只是另一种表示原始私钥的方式。如果你有WIF私钥，你随时可以将其转换回原始格式。

### 代码

注意：此代码需要[checksum.rb](https://github.com/in3rsha/learnmeabitcoin-code/blob/master/checksum.rb)和[base58_encode.rb](https://github.com/in3rsha/learnmeabitcoin-code/blob/master/base58_encode.rb)函数。
```ruby
# Convert Private Key to WIF
privatekey = "ef235aacf90d9f4aadd8c92e4b2562e1d9eb97f0df9ba3b508258739cb013db2"
extended = "80" + privatekey + "01"
extendedchecksum = extended + checksum(extended)
wif = base58_encode(extendedchecksum)

puts wif
```
