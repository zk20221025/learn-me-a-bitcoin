# 扩展密钥
可以从中派生子级的私钥和公钥。
[BIP 32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)
![Extended Keys-1.png](img/Extended%20Keys-1%20(1).png)

扩展密钥是一种[私钥](../../Keys/Private%20Key/Private%20Key.md)或[公钥](../../Keys/Public%20Key/Public%20Key.md)，您可以在[分层确定性钱包](../HD%20Wallets.md)中使用它来派生新的密钥。

因此，您可以拥有一个单独的扩展私钥，并将其用作钱包中所有子私钥和公钥的来源。此外，相应的扩展公钥将生成相同的子公钥。

## 1. 主扩展密钥
您的第一个扩展密钥（主密钥）是通过将种子输入HMAC-SHA512哈希函数创建的。

>您可以将HMAC视为一种**哈希函数**，允许您传递数据以及附加的秘密密钥以生成一组新的随机字节。

![Extended Keys-2.gif](img/Extended%20Keys-2%20(1).gif)
在创建我们的主扩展密钥时，实际上我们不需要使用密钥，因此我们只使用任意字符串“Bitcoin seed”。[^1]
HMAC函数返回64字节的数据（这是完全不可预测的）。我们将其分成两半，以创建我们的主扩展私钥：

* 左半部分将是私钥，就像任何其他私钥一样。
* 右半部分将是链码，只是额外的32个字节的随机数据。
  
>链码是生成子密钥所需的。如果您获得了私钥但没有链码，您将无法推导出后代密钥（从而保护它们）。

### 扩展私钥

因此，扩展私钥最终只是普通私钥与链码相结合。
![Extended Keys-3.png](img/Extended%20Keys-3%20(1).png)

### 扩展公钥
我们还可以创建相应的扩展公钥。这只涉及到取私钥并计算其对应的公钥，然后与同一链代码配对。
![Extended Keys-4.png](img/Extended%20Keys-4%20(1).png)

这样，我们就得到了初始的主扩展私钥和主扩展公钥。

>**提示**：正如你所看到的，扩展密钥本身并没有什么特别之处；它们只是一组共享相同链码（额外的32字节熵）的普通密钥。扩展密钥的真正魔力在于如何生成它们的子密钥。

**代码（Ruby）**
```ruby
require 'openssl' # HMAC
require 'ecdsa'   # private key to public key

# ----
# Seed
# ----
seed = "67f93560761e20617de26e0cb84f7234aaf373ed2e66295c3d7397e6d7ebe882ea396d5d293808b0defd7edd2babd4c091ad942e6a9351e6d075a29d4df872af"
puts "seed: #{seed}"
puts

# --------------------
# Generate Master Keys
# --------------------
# seed
# |
# m

# HMAC
hmac = OpenSSL::HMAC.hexdigest(OpenSSL::Digest::SHA512.new, "Bitcoin seed", [seed].pack("H*")) # digest, key, data
master_private_key = hmac[0..63] # left side of digest
master_chain_code = hmac[64..-1] # right side of digest

# > The SHA512-HMAC function is reused because it is already part of the standard elsewhere, but it takes a key in addition to the data being hashed.
# As the key can be arbitrary, we opted to use to make sure the key derivation was Bitcoin-specific. -- Pieter Wuille

# Get Public Key (multiply generator point by private key)
master_public_key = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(master_private_key.to_i(16)) # multiply generator point by private key
master_public_key = ECDSA::Format::PointOctetString.encode(master_public_key, compression: true).unpack("H*")[0] # encode to compressed public key format

puts "master_chain_code:  #{master_chain_code}"  #=> 463223aac10fb13f291a1bc76bc26003d98da661cb76df61e750c139826dea8b
puts "master_private_key: #{master_private_key}" #=> f79bb0d317b310b261a55a8ab393b4c8a1aba6fa4d08aef379caba502d5d67f9
puts "master_public_key:  #{master_public_key}"  #=> 0252c616d91a2488c1fd1f0f172e98f7d1f6e51f8f389b2f8d632a8b490d5f6da9
```

## 2. 扩展密钥树
所有扩展密钥都可以派生子扩展密钥。

* 扩展私钥可以生成具有新私钥和公钥的子密钥。
* 扩展公钥只能生成具有新公钥的子密钥。
每个子密钥也有一个索引号（最多2 ** 32）。
![Extended Keys-5.png](img/Extended%20Keys-5%20(1).png)
扩展公钥的酷炫之处在于它们可以生成与扩展私钥相同的公钥。

为了安全起见，您可以从扩展私钥派生出两种类型的子密钥：

1. 普通子密钥-扩展私钥和扩展公钥可以生成相同的公钥。
索引0到2147483647（所有可能的子密钥的前一半）
2. 硬化子密钥-只有扩展私钥可以生成公钥。
索引2147483648到4294967295（所有可能的子密钥的后一半）

换句话说，硬化子密钥为您提供了创建“秘密”或“内部”公钥的选项，因为扩展公钥无法派生它们。

## 3. 子扩展密钥派生
扩展私钥和扩展公钥都可以派生子密钥，每个子密钥都有自己独特的索引号。

有三种方法可以派生子密钥：

1. 普通子扩展私钥
2. 硬化子扩展私钥
3. 普通子扩展公钥
>提示：派生的子扩展密钥（和父密钥）彼此独立。换句话说，您不会知道扩展树中的两个公钥以任何方式相互连接。 

### 1. 普通子扩展私钥
![Extended Keys-6.png](img/Extended%20Keys-6%20(1).png)
“标量加法”就是指传统算术加法。

1. **计算公钥**。（这样，相应的扩展公钥就可以将相同的数据放入HMAC函数中，以便生成它们的子级。）
2. **使用介于0和2147483647之间的索引**。这个范围内的索引被指定为正常的子扩展密钥。
3. **将数据和密钥通过HMAC**。
   * 数据=公钥+索引（连接）
   * 密钥=链码

**新链码**是HMAC结果的最后32个字节。这只是一组唯一的字节，我们可以用它来生成新的链码。

**新私钥**是HMAC结果的前32个字节，加上原始私钥。这基本上只是将原始私钥增加了一个随机的32字节数字。我们对新**私钥**进行模运算，以保持新私钥在椭圆曲线的有效数字范围内。

因此，总之，我们使用父扩展私钥中的数据（公钥+索引，链码），并将其通过HMAC函数产生一些新的随机字节。我们使用这些新的随机字节来构造下一个私钥。

**代码（Ruby）**
```ruby
require 'openssl' # HMAC
require 'ecdsa'   # private key to public key

# ---------------------------------
# Normal Child Extended Private Key
# ---------------------------------
# m
# |- m/0
# |- m/1
# |- m/2
# ...

parent_chain_code  = "463223aac10fb13f291a1bc76bc26003d98da661cb76df61e750c139826dea8b"
parent_private_key = "f79bb0d317b310b261a55a8ab393b4c8a1aba6fa4d08aef379caba502d5d67f9"
parent_public_key  = "0252c616d91a2488c1fd1f0f172e98f7d1f6e51f8f389b2f8d632a8b490d5f6da9"
i = 0 # child index number

# Prepare data and key to put through HMAC function
data = [parent_public_key].pack("H*") + [i].pack("N") # public key + index
key = [parent_chain_code].pack("H*") # chain code is key for hmac

hmac = OpenSSL::HMAC.hexdigest(OpenSSL::Digest::SHA512.new, key, data) # digest, key, data
il = hmac[0..63]  # left side of intermediate key [32 bytes]
ir = hmac[64..-1] # right side of intermediate key [32 bytes]

# Chain code is last 32 bytes
child_chain_code = ir

# Check the chain code is valid.
if child_chain_code.to_i(16) >= ECDSA::Group::Secp256k1.order
    raise "Chain code is greater than the order of the curve. Try the next index."
end

# Calculate child private key
child_private_key = (il.to_i(16) + parent_private_key.to_i(16)) % ECDSA::Group::Secp256k1.order # (il + parent_key) % n
child_private_key = child_private_key.to_s(16).rjust(64, '0') # convert to hex (and make sure it's 32 bytes long)

# Work out the corresponding public key too (optional)
child_public_key = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(child_private_key.to_i(16)) # work out the public key for this too
child_public_key = ECDSA::Format::PointOctetString.encode(child_public_key, compression: true).unpack("H*")[0] # encode to compressed public key format

# Results
puts "child_chain_code:   #{child_chain_code}"  #=> 05aae71d7c080474efaab01fa79e96f4c6cfe243237780b0df4bc36106228e31
puts "child_private_key:  #{child_private_key}" #=> 39f329fedba2a68e2a804fcd9aeea4104ace9080212a52ce8b52c1fb89850c72
puts "child_public_key:   #{child_public_key}"  #=> 030204d3503024160e8303c0042930ea92a9d671de9aa139c1867353f6b6664e59
```

### 2. 硬化子扩展私钥
![Extended Keys-7.png](img/Extended%20Keys-7%20(1).png)

1. **使用介于2147483647和4294967295之间的索引**。这些索引被指定为强化子扩展密钥。
2. **通过HMAC将数据和密钥传递。**
   * 数据=私钥+索引（连接）
   * 密钥=链码
**新链码**是HMAC结果的最后32个字节。

**新的私钥**是从HMAC结果的前32个字节中添加到原始私钥中的结果。这只是将原始私钥增加一个随机的32字节数。

然而，这个强化的子密钥是通过将**私钥**放入HMAC函数中构造的（扩展公钥无法访问），这意味着通过这种方式派生的子扩展私钥将具有无法通过相应的扩展公钥派生的公钥。

>除非您需要能够生成没有私钥访问权限的公钥，否则强化派生应该是默认设置。– Pieter Wuille2[^2]

代码(Ruby)
```ruby
require 'openssl' # HMAC
require 'ecdsa'   # private key to public key

# -----------------------------------
# Hardened Child Extended Private Key
# -----------------------------------
# m
# ...
# |- m/2147483648
# |- m/2147483649
# |- m/2147483650
# ...

parent_chain_code  = "463223aac10fb13f291a1bc76bc26003d98da661cb76df61e750c139826dea8b"
parent_private_key = "f79bb0d317b310b261a55a8ab393b4c8a1aba6fa4d08aef379caba502d5d67f9"
i = 2147483648 # child index number (must between 2**31 and 2**32-1)

# Prepare data and key to put through HMAC function
data = ["00"].pack("H*") + [parent_private_key].pack("H*") + [i].pack("N")  # 0x00 + private_key + index
key = [parent_chain_code].pack("H*") # chain code is key for hmac

hmac = OpenSSL::HMAC.hexdigest(OpenSSL::Digest::SHA512.new, key, data) # digest, key, data
il = hmac[0..63]  # left side of intermediate key [32 bytes]
ir = hmac[64..-1] # right side of intermediate key [32 bytes]

# Chain code is last 32 bytes
child_chain_code = ir

# Check the chain code is valid.
if child_chain_code.to_i(16) >= ECDSA::Group::Secp256k1.order
    raise "Chain code is greater than the order of the curve. Try the next index."
end

# Calculate child private key
child_private_key = (il.to_i(16) + parent_private_key.to_i(16)) % ECDSA::Group::Secp256k1.order # (il + parent_key) % n
child_private_key = child_private_key.to_s(16).rjust(64, '0') # convert to hex (and make sure it's 32 bytes long)

# Work out the corresponding public key too (optional)
child_public_key = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(child_private_key.to_i(16)) # work out the public key for this too
child_public_key = ECDSA::Format::PointOctetString.encode(child_public_key, compression: true).unpack("H*")[0] # encode to compressed public key format

puts "child_chain_code:   #{child_chain_code}"  #=> cb3c17166cc30eb7fdd11993fb7307531372e565cd7c7136cbfa4655622bc2be
puts "child_private_key:  #{child_private_key}" #=> 7272904512add56fef94c7b4cfc62bedd0632afbad680f2eb404e95f2d84cbfa
puts "child_public_key:   #{child_public_key}"  #=> 0355cff4a963ce259b08be9a864564caca210eb4eb35fcb75712e4bba7550efd95
```

### 3. 正常子扩展公钥
![Extended Keys-8.png](img/Extended%20Keys-8%20(1).png)

1. **使用0到2147483647之间的索引**。这些索引用于普通子扩展密钥。
2. **将数据和密钥通过HMAC进行处理。**
   * 数据=公钥+索引（连接）
   * 密钥=链代码

**新链代码**是HMAC结果的最后32个字节。这将是与上面的普通子扩展私钥相同的链代码，因为如果您回顾一下，就会发现我们将相同的输入放入HMAC函数中。

**新的公钥**是原始公钥点加上HMAC结果的前32个字节作为曲线上的点（乘以生成器以将其作为点）。

因此，总之，我们将与生成子扩展私钥时相同的数据和密钥放入HMAC函数中。然后，通过[椭圆曲线点加法](../../Keys/ECDSA/ECDSA.md)，可以计算出子公钥，其使用与子扩展私钥相同的HMAC结果的前32个字节（这意味着它对应于子扩展私钥中的私钥）。

**代码（Ruby）**
```ruby
require 'openssl' # HMAC
require 'ecdsa'

# --------------------------------
# Normal Child Extended Public Key
# --------------------------------
# m -------- p
#            |- p/0
#            |- p/1
#            |- p/3

parent_chain_code = "463223aac10fb13f291a1bc76bc26003d98da661cb76df61e750c139826dea8b"
parent_public_key = "0252c616d91a2488c1fd1f0f172e98f7d1f6e51f8f389b2f8d632a8b490d5f6da9"
i = 0 # child index number

if i >= 2**31
    raise "Can't create hardened child public keys from parent public keys."
end

# Prepare data and key to put through HMAC function
key = [parent_chain_code].pack("H*")
data = [parent_public_key].pack("H*") + [i].pack("N") # 32-bit unsigned, network (big-endian) byte order

hmac = OpenSSL::HMAC.hexdigest(OpenSSL::Digest::SHA512.new, key, data)
il = hmac[0..63]  # left side of intermediate key [32 bytes]
ir = hmac[64..-1] # right side of intermediate key [32 bytes]

# Chain code is last 32 bytes
child_chain_code = hmac[64..-1]

if il.to_i(16) >= ECDSA::Group::Secp256k1.order
    raise "Result of digest is greater than the order of the curve. Try the next index."
end

# Work out the child public key
point_hmac   = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(il.to_i(16))                               # convert hmac il to a point
point_public = ECDSA::Format::PointOctetString.decode([parent_public_key].pack("H*"), ECDSA::Group::Secp256k1) # convert parent_public_key to a point
point = point_hmac.add_to_point(point_public)                                                                  # point addition

if (point == ECDSA::Group::Secp256k1.infinity)
    raise "Child public key point is at point of infinitiy. Try the next index."
end

child_public_key = ECDSA::Format::PointOctetString.encode(point, compression: true).unpack("H*")[0] # encode to compress public key

puts "child_chain_code:   #{child_chain_code}"
puts "child_public_key:   #{child_public_key}"
```

### 4. 硬化子扩展公钥

不可能。

## 4. 为什么这样做有效？

换句话说，从扩展公钥派生的公钥如何与从扩展私钥派生的私钥相对应？

好的，对于两个子扩展密钥，我们将相同的输入放入HMAC函数中，因此我们得到相同的数据作为结果。然后，我们使用这些数据的前32个字节（基本上是一个数字）：

* 通过这个数字将父私钥增加，以创建子私钥。

* 通过相同的数量将父公钥增加，以创建子公钥。

由于椭圆曲线数学的工作方式，子私钥将对应于子公钥。
![Extended Keys-9.png](img/Extended%20Keys-9%20(1).png)

**请再说一遍？**
首先，记住公钥只是椭圆曲线上的生成点与私钥相乘的结果：
![Extended Keys-10.png](img/Extended%20Keys-10%20(1).png)

现在，如果你将这个私钥增加一个数字（即HMAC结果的前32个字节），我们就会得到一个新的私钥。当你将这个新私钥乘以生成点时，我们就会得到新的公钥：
![Extended Keys-11.png](img/Extended%20Keys-11%20(1).png)

同样地，如果您将原始公钥加上相同的数字（作为曲线上的点），您将得到与上面相同的新公钥。
![Extended Keys-12.png](img/Extended%20Keys-12%20(1).png)

代码（Ruby）
```ruby
require 'ecdsa'   # (ECDSA Math) sudo gem install ecdsa

# Original Private Key and Public Key
private_key = 12345
public_key  = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(private_key)
number = rand(1000000) # used to modify the private key and public key independently

# Child Public Key 1: (Private Key + Number) * Generator
private_and_number = private_key + number
child_public_key1  = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(private_and_number)

# Child Public Key 2: Public Key + (Number * Generator)
number_point       = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(number)
child_public_key2  = public_key.add_to_point(number_point)

# Results
puts ECDSA::Format::PointOctetString.encode(child_public_key1, compression: true).unpack("H*") #=> e.g. 022861a4809b2d8eb9269abea605f47db91deb9f5385bcc10b94aac6a0b81cba3d
puts ECDSA::Format::PointOctetString.encode(child_public_key2, compression: true).unpack("H*") #=> e.g. 022861a4809b2d8eb9269abea605f47db91deb9f5385bcc10b94aac6a0b81cba3d
puts child_public_key1 == child_public_key2 #=> true
```

**安全提示**
由于普通子密钥的派生方式，如果您拥有扩展公钥和任何子私钥，则有可能推导出父扩展私钥。
![Extended Keys-13.png](img/Extended%20Keys-13%20(1).png)
换句话说，如果你的扩展公钥是公开的，一定要非常小心不要泄露子私钥。如果你泄露了，任何人都可以倒推计算出扩展私钥，并从该层级中的**所有子密钥**中窃取比特币。

>提示：这就是为什么硬化子密钥很有用，因为在树的一个层级中丢失一个子私钥永远不会使其他子私钥面临被派生的风险。

代码（Ruby）
```ruby
require 'openssl'

# ------
# Secret
# ------
parent_private_key = "081549973bafbba825b31bcc402a3c4ed8e3185c2f3a31c75e55f423e9629aa3"

# --------------------
# Revealed Information
# --------------------

# parent extended public key
parent_public_key = "0343b337dec65a47b3362c9620a6e6ff39a1ddfa908abab1666c8a30a3f8a7cccc"
parent_chain_code = "1d7d2a4c940be028b945302ad79dd2ce2afe5ed55e1a2937a5af57f8401e73dd"

# child private key
child_private_key = "c41cd73a9df26db3bccfad12e9bbe66d4d619b6c9510f43a618fbef27fa78ce4" # index=0

# ------------------------------------------
# We can now work out the parent private key
# ------------------------------------------

# 1. Get the left side of the HMAC from the parent extended public key
key  = [parent_chain_code].pack("H*")
data = [parent_public_key].pack("H*") + [0].pack("N") # the 0 refers to the same index as the child private key
hmac = OpenSSL::HMAC.hexdigest(OpenSSL::Digest::SHA512.new, key, data)
hmac_left = hmac[0..63]

# 2. Calculate the parent private key (child private key minus the hmac left)
n = 115792089237316195423570985008687907852837564279074904382605163141518161494337 # order of the curve
calculated_key = (child_private_key.to_i(16) - hmac_left.to_i(16)) % n

# -------
# Results
# -------
puts "parent private key: #{parent_private_key.to_i(16)}"
puts
puts "child private key:  #{child_private_key.to_i(16)}"
puts "hmac left:          #{hmac_left.to_i(16)}"
puts "calculated:         #{calculated_key}" #=> 081549973bafbba825b31bcc402a3c4ed8e3185c2f3a31c75e55f423e9629aa3
```

## 5. 序列化
扩展密钥可以被序列化以便于传递。这些序列化的数据包括私钥/公钥和链码，以及一些附加的元数据。
![Extended Keys-14.png](img/Extended%20Keys-14%20(1).png)

一个序列化的密钥包含以下字段：
|4 bytes	|Version	|Places “xprv” 0488ade4 or “xpub” 0488b21e at the start.|
|---|---|---|
|1 byte	|Depth	|How many derivations deep this extended key is from the master key.|
|4 bytes|	Parent Fingerprint	|The first 4 bytes of the hash160 of the parent’s public key. This helps to identify the parent later.|
|4 bytes|	Child Number	|The index number of this child from the parent.|
|32 bytes|	Chain Code	|The extra 32 byte secret. This prevents others from deriving child keys without it.|
|33 bytes|	Key	|The private key (prepend 0x00) or public key.|
注意事项：
>版本字节：
0488ade4 = xprv
0488b21e = xpub
049d7878 = yprv（用于BIP 49派生路径中的扩展密钥）
049d7cb2 = ypub
04b2430c = zprv（用于BIP 84派生路径中的扩展密钥）
04b24746 = zpub

>主扩展密钥：
深度= 00
指纹= 00000000
子编号= 00000000

>在密钥字段中，私钥（32字节）以0x00开头，以将其扩展到与公钥（33字节）相同的长度。

然后在此数据中添加[校验和](../../Keys/Checksum/Checksum.md)（以帮助检测错误），最后将所有内容转换为[Base58](../../Keys/Base58/Base58.md)（以创建人类友好的扩展密钥格式）。

扩展私钥的格式如下：
```
xprv9tuogRdb5YTgcL3P8Waj7REqDuQx4sXcodQaWTtEVFEp6yRKh1CjrWfXChnhgHeLDuXxo2auDZegMiVMGGxwxcrb2PmiGyCngLxvLeGsZRq
```
扩展公钥的样子是这样的：
```
xpub67uA5wAUuv1ypp7rEY7jUZBZmwFSULFUArLBJrHr3amnymkUEYWzQJz13zLacZv33sSuxKVmerpZeFExapBNt8HpAqtTtWqDQRAgyqSKUHu
```
正如您所看到的，它们非常长，但这是因为它们包含了关于扩展密钥的额外有用信息。

>**提示**：指纹、深度和子编号并不是派生子扩展密钥所必需的——它们只是帮助您识别当前密钥的父项和在树中的位置。

>**注意**：子编号的4字节字段是扩展密钥仅限于派生索引在0和4,294,967,295（0xffffffff）之间的子项的原因。

**代码（Ruby）**
```ruby
# -----
# Utils - Needed for creating the fingerprint and checksum, and converting hex string to Base58
# -----
require 'digest'

def create_fingerprint(parent_public_key)
    hash160 = Digest::RMD160.digest(Digest::SHA256.digest([parent_public_key].pack("H*"))) # hash160 it
    fingerprint = hash160[0...4].unpack("H*").join # take first 4 bytes (and convert to hex)
    return fingerprint
end

def hash256(hex)
    binary = [hex].pack("H*")
    hash1 = Digest::SHA256.digest(binary)
    hash2 = Digest::SHA256.digest(hash1)
    result = hash2.unpack("H*")[0]
    return result
end

def checksum(hex)
    hash = hash256(hex) # Hash the data through SHA256 twice
    return hash[0...8]  # Return the first 4 bytes (8 characters)
end

def base58_encode(hex)
    @chars = %w[
      1 2 3 4 5 6 7 8 9
    A B C D E F G H   J K L M N   P Q R S T U V W X Y Z
    a b c d e f g h i j k   m n o p q r s t u v w x y z
    ]
    @base = @chars.length

    i = hex.to_i(16)
    buffer = String.new

    while i > 0
        remainder = i % @base
        i = i / @base
        buffer = @chars[remainder] + buffer
    end

    # add '1's to the start based on number of leading bytes of zeros
    leading_zero_bytes = (hex.match(/^([0]+)/) ? $1 : '').size / 2

    ("1"*leading_zero_bytes) + buffer
end

# --------------------------
# Extended Key Serialization
# --------------------------
parent_public_key = "0252c616d91a2488c1fd1f0f172e98f7d1f6e51f8f389b2f8d632a8b490d5f6da9" # needed to create fingerprint

chain_code  = "05aae71d7c080474efaab01fa79e96f4c6cfe243237780b0df4bc36106228e31" # m/0
private_key = "39f329fedba2a68e2a804fcd9aeea4104ace9080212a52ce8b52c1fb89850c72" # m/0

# version + depth + fingerprint + childnumber + chain_code + key + (checksum)
#
# version:     puts xprv or xpub at the start
# depth:       how many times this child has been derived from master key (0 = master key)
# fingerprint: created from parent public key (allows you to spot adjacent xprv and xpubs)
# childnumber: the index of this child key from the parent
# chain_code:  the current chain code being used for this key
# key:         the private or public key you want to create a serialized extended key for (prepend 0x00 for private)

version     = "0488ade4" # private = 0x0488ade4 (xprv), public = 0x0488b21e (xpub)
depth       = "01"
fingerprint = create_fingerprint(parent_public_key) #=> "018c1259"
childnumber = "00000000"
chain_code  = chain_code
key         = "00" + private_key  # prepend 00 to private keys (to make them 33 bytes, the same as public keys)

serialized = version + depth + fingerprint + childnumber + chain_code + key
extended_private_key = base58_encode(serialized + checksum(serialized))

puts "extended_private_key: #{extended_private_key}" #=> xprv9tuogRdb5YTgcL3P8Waj7REqDuQx4sXcodQaWTtEVFEp6yRKh1CjrWfXChnhgHeLDuXxo2auDZegMiVMGGxwxcrb2PmiGyCngLxvLeGsZRq
```

## 代码
以下是创建、派生和序列化扩展密钥的完整代码片段。
### Ruby
```require 'openssl' # HMAC
require 'ecdsa'   # private key to public key

# ----
# Seed
# ----
seed = "67f93560761e20617de26e0cb84f7234aaf373ed2e66295c3d7397e6d7ebe882ea396d5d293808b0defd7edd2babd4c091ad942e6a9351e6d075a29d4df872af"
puts "seed: #{seed}"
puts

# --------------------
# Generate Master Keys
# --------------------
# seed
# |
# m

# HMAC
hmac = OpenSSL::HMAC.hexdigest(OpenSSL::Digest::SHA512.new, "Bitcoin seed", [seed].pack("H*")) # digest, key, data
master_private_key = hmac[0..63] # left side of digest
master_chain_code = hmac[64..-1] # right side of digest

# > The SHA512-HMAC function is reused because it is already part of the standard elsewhere, but it takes a key in addition to the data being hashed.
# As the key can be arbitrary, we opted to use to make sure the key derivation was Bitcoin-specific. -- Pieter Wuille

# Get Public Key (multiply generator point by private key)
master_public_key = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(master_private_key.to_i(16)) # multiply generator point by private key
master_public_key = ECDSA::Format::PointOctetString.encode(master_public_key, compression: true).unpack("H*")[0] # encode to compressed public key format

puts "master_chain_code:  #{master_chain_code}"  #=> 463223aac10fb13f291a1bc76bc26003d98da661cb76df61e750c139826dea8b
puts "master_private_key: #{master_private_key}" #=> f79bb0d317b310b261a55a8ab393b4c8a1aba6fa4d08aef379caba502d5d67f9
puts "master_public_key:  #{master_public_key}"  #=> 0252c616d91a2488c1fd1f0f172e98f7d1f6e51f8f389b2f8d632a8b490d5f6da9
puts

# --------------------------
# Child Extended Private Key
# --------------------------
# m
# |- m/0
# |- m/1
# |- m/2
# .
# .
# .
# |- m/2147483648 (hardened - m/0')
parent_private_key = master_private_key
parent_chain_code  = master_chain_code
i = 0 # child number

# Normal (parent public key can create child public key for this private key)
if i < 2**31
    point = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(parent_private_key.to_i(16)) # multiply generator point by private key
    point = ECDSA::Format::PointOctetString.encode(point, compression: true).unpack("H*")[0] # encode to compressed public key format
    data = [point].pack("H*") + [i].pack("N")     # point(private_key) i
end

# > This means that it is possible to give someone a parent extended key (public key + chain code), and they can derive all public keys in the chain. -- Pieter Wuille

# Hardened (parent public key cannot create public key for this private key)
if i >= 2**31
  data = ["00"].pack("H*") + [parent_private_key].pack("H*") + [i].pack("N")  # 0x00 private_key i
end

# > Hardened derivation should be the default unless there is a good reason why you need to be able to generate public keys without access to the private key. -- Pieter Wuille
# > You cannot derive hardened public keys from xpubs, because hardening is specifically designed to make exactly that impossible. -- Pieter Wuille

# Put data and key through hmac
key = [parent_chain_code].pack("H*") # chain code is key for hmac
hmac = OpenSSL::HMAC.hexdigest(OpenSSL::Digest::SHA512.new, key, data) # digest, key, data
il = hmac[0..63]  # left side of intermediate key [32 bytes]
ir = hmac[64..-1] # right side of intermediate key [32 bytes]

# Chain code is last 32 bytes
child_chain_code = ir

if child_chain_code.to_i(16) >= ECDSA::Group::Secp256k1.order
    raise "Chain code is greater than the order of the curve. Try the next index."
end

# Work out the corresponding public key too (optional)
child_private_key = (il.to_i(16) + parent_private_key.to_i(16)) % ECDSA::Group::Secp256k1.order # (il + parent_key) % n
child_private_key = child_private_key.to_s(16).rjust(64, '0') # convert to hex (and make sure it's 32 bytes long)

if child_private_key.to_i(16) == 0
    raise "Child private key is zero. Try the next index."
end

child_public_key = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(child_private_key.to_i(16)) # work out the public key for this too
child_public_key = ECDSA::Format::PointOctetString.encode(child_public_key, compression: true).unpack("H*")[0] # encode to compressed public key format

puts "child_chain_code:   #{child_chain_code}"
puts "child_private_key:  #{child_private_key}"
puts "child_public_key:   #{child_public_key}"
puts

# -------------------------
# Child Extended Public Key
# -------------------------
# m -------- p
#            |- p/0
#            |- p/1
#            |- p/3
parent_public_key = master_public_key
parent_chain_code = master_chain_code
i = 0 # child number

if i >= 2**31
    raise "Can't create hardened child public keys from parent public keys."
end

key = [parent_chain_code].pack("H*")
data = [parent_public_key].pack("H*") + [i].pack("N") # 32-bit unsigned, network (big-endian) byte order

# Put data and key through hmac
hmac = OpenSSL::HMAC.hexdigest(OpenSSL::Digest::SHA512.new, key, data)
il = hmac[0..63]  # left side of intermediate key [32 bytes]
ir = hmac[64..-1] # right side of intermediate key [32 bytes]

# Chain code is last 32 bytes
child_chain_code = hmac[64..-1]

if il.to_i(16) >= ECDSA::Group::Secp256k1.order
    raise "Result of digest is greater than the order of the curve. Try the next index."
end

# Work out the child public key
point_hmac   = ECDSA::Group::Secp256k1.generator.multiply_by_scalar(il.to_i(16))                               # convert hmac il to a point
point_public = ECDSA::Format::PointOctetString.decode([parent_public_key].pack("H*"), ECDSA::Group::Secp256k1) # convert parent_public_key to a point
point = point_hmac.add_to_point(point_public)                                                                  # point addition

if (point == ECDSA::Group::Secp256k1.infinity)
    raise "Child public key point is at point of infinitiy. Try the next index."
end

child_public_key = ECDSA::Format::PointOctetString.encode(point, compression: true).unpack("H*")[0] # encode to compress public key

puts "child_chain_code:   #{child_chain_code}"
puts "child_public_key:   #{child_public_key}"
puts

# --------------------------
# Extended Key Serialization
# --------------------------

# Utils - Needed for creating the fingerprint and checksum, and converting hex string to Base58
# -----
require 'digest'

def create_fingerprint(parent_public_key)
    hash160 = Digest::RMD160.digest(Digest::SHA256.digest([parent_public_key].pack("H*"))) # hash160 it
    fingerprint = hash160[0...4].unpack("H*").join # take first 4 bytes (and convert to hex)
    return fingerprint
end

def hash256(hex)
    binary = [hex].pack("H*")
    hash1 = Digest::SHA256.digest(binary)
    hash2 = Digest::SHA256.digest(hash1)
    result = hash2.unpack("H*")[0]
    return result
end

def checksum(hex)
    hash = hash256(hex) # Hash the data through SHA256 twice
    return hash[0...8]  # Return the first 4 bytes (8 characters)
end

def base58_encode(hex)
    @chars = %w[
      1 2 3 4 5 6 7 8 9
    A B C D E F G H   J K L M N   P Q R S T U V W X Y Z
    a b c d e f g h i j k   m n o p q r s t u v w x y z
    ]
    @base = @chars.length

    i = hex.to_i(16)
    buffer = String.new

    while i > 0
        remainder = i % @base
        i = i / @base
        buffer = @chars[remainder] + buffer
    end

    # add '1's to the start based on number of leading bytes of zeros
    leading_zero_bytes = (hex.match(/^([0]+)/) ? $1 : '').size / 2

    ("1"*leading_zero_bytes) + buffer
end

# ---------
# Serialize
# ---------
parent_public_key = master_public_key # needed to create fingerprint

chain_code  = child_chain_code # m/0
private_key = child_private_key # m/0

# version + depth + fingerprint + childnumber + chain_code + key + (checksum)
#
# version:     puts xprv or xpub at the start
# depth:       how many times this child has been derived from master key (0 = master key)
# fingerprint: created from parent public key (allows you to spot adjacent xprv and xpubs)
# childnumber: the index of this child key from the parent
# chain_code:  the current chain code being used for this key
# key:         the private or public key you want to create a serialized extended key for (prepend 0x00 for private)

version     = "0488ade4" # private = 0x0488ade4 (xprv), public = 0x0488b21e (xpub)
depth       = "01"
fingerprint = create_fingerprint(parent_public_key) #=> "018c1259"
childnumber = "00000000" # 4 byte hexadecimal string
chain_code  = chain_code
key         = "00" + private_key  # prepend 00 to private keys (to make them 33 bytes, the same as public keys)

serialized = version + depth + fingerprint + childnumber + chain_code + key
extended_private_key = base58_encode(serialized + checksum(serialized))

puts "extended_private_key: #{extended_private_key}"
```

### PHP
```php
<?php
// Note: Requires https://github.com/Bit-Wasp/secp256k1-php

// ------------------
// Seed to Master Key
// ------------------

$seed = "67f93560761e20617de26e0cb84f7234aaf373ed2e66295c3d7397e6d7ebe882ea396d5d293808b0defd7edd2babd4c091ad942e6a9351e6d075a29d4df872af";
echo "seed: $seed".PHP_EOL;
echo PHP_EOL;

// Hash it through HMAC-SHA512 and split in to two halves
// 
//             seed
//              |
//            |----|
//            |HMAC|
//            |----|
//              |
// |    priv    | chain code |
// 
$hmac = hash_hmac("sha512", hex2bin($seed), "Bitcoin seed"); // algo, data, key, raw_output (optional)
$master_private_key = substr($hmac, 0, 64); // first half is private key
$master_chain_code = substr($hmac, 64);     // second half is chain code
//
// m (priv, chain_code)
// 
echo "master_chain_code:  $master_chain_code".PHP_EOL;
echo "master_private_key: $master_private_key".PHP_EOL;

// Use elliptic curve mathematics to go from private key to the public key
// 
// priv -> pub
// 
$context = secp256k1_context_create(SECP256K1_CONTEXT_SIGN | SECP256K1_CONTEXT_VERIFY); // set curve context
$point = null; secp256k1_ec_pubkey_create($context, $point, hex2bin($master_private_key)); // get public key point
$serialized = ''; secp256k1_ec_pubkey_serialize($context, $serialized, $point, SECP256K1_EC_COMPRESSED); // serialize to compressed public key
$master_public_key = bin2hex($serialized);

// 
// m (priv, pub, chain_code)
// 
echo "master_public_key:  $master_public_key".PHP_EOL;
echo PHP_EOL;

// ---------------------------------------------------------
// Parent Extended Private Key to Child Extended Private Key (m/0)
// ---------------------------------------------------------
// m (priv, pub, chain code)
// |
// m/0 (priv, pub, chain code)
$parent_private_key = $master_private_key; // get private key of parent
$parent_chain_code  = $master_chain_code;   // get chain code of parent
$i = 0;                                    // the index of this child from parent

// Normal Child (extended public key can derive its public keys)
if ($i < 2**31) { // first half of possible indexes are normal children
    // Get public key point for the parent private key
    $parent_public_key = $master_public_key; // you can derive this from the parent private key using the elliptic curve mathematics above
    // The `data` for the upcoming HMAC is the parent public key + index
    //  data = | parent public key (33 bytes) | i (4 bytes) |
    $data = pack("H*", $parent_public_key).pack("N", $i);
}

// Hardened Child (extended public key cannot derive its public keys)
if ($i >= 2**31 ) { // second half of possible indexes are hardened children
    //  data = | 00 | parent private key (32 bytes) | i (4 bytes) |
    $data = pack("H*", "00").pack("H*", $master_private_key).pack("N", $i);
}

// Put data (public key or private key) and chain code through HMAC-SHA512
$hmac = hash_hmac("sha512", $data, hex2bin($parent_chain_code)); // algo, data, key
$left = substr($hmac, 0, 64);
$right = substr($hmac, 64);

// Child chain code is right side of HMAC-SHA512 
// m
// |
// m/0 (chain code)
$child_chain_code = $right;
echo "child_chain_code:   $child_chain_code".PHP_EOL;

// Check child chain code is valid (not greater than the number of points on the curve)
if (hexdec($child_chain_code) >= 115792089237316195423570985008687907852837564279074904382605163141518161494337) {
    throw new Exception("Child chain code is greater than or equal to the order of the elliptic curve. Try next index.");
}

// Calculate child private key (left side of hmac + parent private key) % order
function bchexdec($hex) { // need a utility function to work with addition of large hexadecimal numbers
    if(strlen($hex) == 1) {
        return hexdec($hex);
    } else {
        $remain = substr($hex, 0, -1);
        $last = substr($hex, -1);
        return bcadd(bcmul(16, bchexdec($remain)), hexdec($last));
    }
}

function bcdechex($dec) { // need a utility function to convert large integer back to hexadecimal
    $last = bcmod($dec, 16);
    $remain = bcdiv(bcsub($dec, $last), 16);

    if($remain == 0) {
        return dechex($last);
    } else {
        return bcdechex($remain).dechex($last);
    }
}

// child private key = (left + parent private key) % order
// m
// |
// m/0 (priv)
$add = bcadd(bchexdec($left), bchexdec($parent_private_key)); // add left side of hmac to parent private key
$mod = bcmod($add, "115792089237316195423570985008687907852837564279074904382605163141518161494337"); // modulus the order of the curve
$child_private_key = str_pad(bcdechex($mod), 64, "0", STR_PAD_LEFT); // ensure it is 64 chars (prepend zeros)

// Check child private key is valid (not zero)
if (hexdec($child_private_key === 0)) {
    throw new Exception("Child private key is zero (so it's invalid). Try next index.");
}
echo "child_private_key:  $child_private_key".PHP_EOL;

// calculate corresponding child public key from this child private key
// m
// |
// m/0 (pub)
$context = secp256k1_context_create(SECP256K1_CONTEXT_SIGN | SECP256K1_CONTEXT_VERIFY); // set curve context
$point = null; secp256k1_ec_pubkey_create($context, $point, hex2bin($child_private_key)); // get public key point
$serialized = ''; secp256k1_ec_pubkey_serialize($context, $serialized, $point, SECP256K1_EC_COMPRESSED); // serialize to compressed public key
$child_public_key = bin2hex($serialized);
echo "child_public_key:   $child_public_key".PHP_EOL; // note may need to pad hex
echo PHP_EOL;

// -------------------------------------------------------
// Parent Extended Public Key to Child Extended Public Key (m/0)
// -------------------------------------------------------
// m -------- p (pub, chain code)
//            |
//            p/0 (pub, chain code)
$parent_public_key = $master_public_key; // get public key of parent
$parent_chain_code = $master_chain_code; // get chain code of parent
$i = 0;                                  // the index of this child from parent

// Check that you're not trying to create a hardened child public key (extended public keys cannot access public keys of hardened extended private key children)
if ($i >= 2**31) {
    throw new Exception("Cannot create a hardened extended child public key. Hardened children of extended private key are private.");
}

// Put data (parent public key + i) and chain code through HMAC-SHA512
$hmac = hash_hmac("sha512", pack("H*", $parent_public_key).pack("N", $i), hex2bin($parent_chain_code)); // same data as when creating normal child extended private key
$left = substr($hmac, 0, 64);
$right = substr($hmac, 64);

// Child chain code is right side of HMAC-SHA512 
// m -------- p
//            |
//            |p/0 (chain code)
$child_chain_code = $right;
echo "child_chain_code:   $child_chain_code".PHP_EOL;

// Check child chain code is valid (not greater or equal to than the number of points on the curve)
if (hexdec($child_chain_code) >= 115792089237316195423570985008687907852837564279074904382605163141518161494337) {
    throw new Exception("Child chain code is greater than or equal to the order of the elliptic curve. Try next index.");
}

// Calculate child public key: point(parent public key) + hmac left
// m -------- p
//            |
//            |p/0 (pub)

// point (parent public key)
$context = secp256k1_context_create(SECP256K1_CONTEXT_SIGN | SECP256K1_CONTEXT_VERIFY);                 // set curve context
$point_public = null; secp256k1_ec_pubkey_parse($context, $point_public, hex2bin($parent_public_key));  // point(parent public key)

// add point(hmac left) to point(parent public key)
secp256k1_ec_pubkey_tweak_add($context, $point_public, hex2bin($left)); // add hmac left to the parent public key point
$add = ''; secp256k1_ec_pubkey_serialize($context, $add, $point_public, SECP256K1_EC_COMPRESSED); // serialize to compressed key
$child_public_key = unpack("H*", $add)[1];
echo "child_public_key:   $child_public_key".PHP_EOL;
echo PHP_EOL;

// ----------------------
// Serialize Extended Key 
// ----------------------
$parent_public_key = $master_public_key; // needed for creating fingerprint

$chain_code = $child_chain_code;
$private_key = $child_private_key;

// | version | depth  | parent fingerprint |  index  | chain code | key (private or public) |
// | 4 bytes | 1 byte |      4 bytes       | 4 bytes |  32 bytes  |        33 bytes         |  = 78 bytes

function create_fingerprint($parent_public_key) {
    $hash160 = hash("ripemd160", hash("sha256", hex2bin($parent_public_key), true)); // hash160 the parent public key
    return substr($hash160, 0, 8); // return first 4 bytes (8 hex characters)
}

$version     = "0488ade4";               // 0488ade4 = private (xprv), 0488b21e = public (xpub)
$depth       = "01";                     // how many times this child has been derived from master key (0 = master key)
$fingerprint = create_fingerprint($parent_public_key); // fingerprint of parent key - first 4 bytes of hash160(parent public key) (00000000 = master key)
$index       = "00000000";               // the index of this child key from the parent (4 byte hexadecimal string)
$chain_code  = $chain_code;              // chain code for this key
$key         = "00".$private_key;        // private or public key you want to create a serialized extended key for (prepend 0x00 for private)

// Serialize the data
$serialized = $version.$depth.$fingerprint.$index.$chain_code.$key;

// Create a checksum for it (hash twice through sha256 and take first 4 bytes)
$checksum = substr(hash("sha256", hash("sha256", hex2bin($serialized), true)), 0, 8);

// Base58 encode the serialized+checksum
function base58_encode($hex){
    $base58chars = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz";

    if (strlen($hex) == 0) {
        return '';
    }

    // Convert the hex string to a base10 integer
    $num = gmp_strval(gmp_init($hex, 16), 58);

    // Check that number isn't just 0 - which would be all padding.
    if ($num != '0') {
        $num = strtr($num, '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuv', $base58chars);
    }
    else {
        $num = '';
    }

    // Pad the leading 1's
    $pad = '';
    $n = 0;
    while (substr($hex, $n, 2) == '00') {
        $pad .= '1';
        $n += 2;
    }

    return $pad . $num;
}

$master_extended_private_key = base58_encode($serialized.$checksum);
echo "master_extended_private_key: $master_extended_private_key".PHP_EOL;
```

## 相关
* [ECDSA](../../Keys/ECDSA/ECDSA.md)——如何使用私钥和公钥签名（和验证）数据。

## 链接
* [BIP 32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)（[Pieter Wuille](https://github.com/sipa)的原始规范。此页面只是原始BIP的重写。）
* [Ian Coleman BIP39工具](https://iancoleman.io/bip39/)（从种子派生扩展密钥的绝佳工具）
* [Bitcoin Ruby：ext_key.rb](https://github.com/lian/bitcoin-ruby/blob/master/lib/bitcoin/ext_key.rb)（Ruby中的不错实现）

## StackExchange问题

* [是什么让扩展公钥或私钥？](https://bitcoin.stackexchange.com/questions/78295/what-makes-an-extended-public-or-private-key)
* [为什么硬化的xpub密钥无法生成子公钥？](https://bitcoin.stackexchange.com/questions/92562/why-hardended-xpub-key-cannot-generate-child-public-key)
* [MAC和HMAC之间有什么区别？](https://crypto.stackexchange.com/questions/6523/what-is-the-difference-between-mac-and-hmac)

[^1]:https://bitcoin.stackexchange.com/questions/36917/hmac-bitcoin-seed-for-bip32/36937#36937
[^2]:https://bitcoin.stackexchange.com/questions/62533/key-derivation-in-hd-wallets-using-the-extended-private-key-vs-hardened-derivati/84007#84007