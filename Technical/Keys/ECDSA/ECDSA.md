# ECDSA
椭圆曲线数字签名算法
![ECDSA-1.png](img/ECDSA-1%20(1).png)

比特币使用一种名为ECDSA的数字签名系统来控制比特币的所有权。

简而言之，数字签名系统允许您生成自己的私钥/公钥对，并使用私钥生成数字签名，证明您是公钥的所有者，而无需透露私钥。

比特币中使用该系统允许人们接收和发送比特币。任何人都可以生成自己的密钥对，然后任何人都可以将比特币发送（或“锁定”）到您的公钥。没有人能够窃取这些比特币，因为只有拥有此公钥的正确私钥的人才能生成有效签名以“解锁”比特币并将其发送给其他人。

自1970年以来，创建数字签名的能力已经存在，这要归功于[RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem))的发明。在1994年，[DSA](https://en.wikipedia.org/wiki/Digital_Signature_Algorithm)发布为数字签名系统的标准。 ECDSA只是使用椭圆曲线密码学实现DSA，因为椭圆曲线的数学允许更高效的签名创建和验证。

我不知道足够的密码学知识来解释ECDSA为什么有效，但我制作了这个页面，以向您展示如何在比特币中创建自己的私钥和公钥，并使用它们来签署自己的交易。

代码使用**Ruby**编写，并且我已尽可能简化了图表和方程式。

>对于Satoshi Nakamoto来说，了解数字签名系统的工作原理细节并不是必要的。他们只需要知道它能够工作，并且可以将其用作在他们正在构建的系统中发送和接收货币的机制。比特币的第一个版本使用OpenSSL库提供创建和验证数字签名的功能。

## 完整的ECDSA代码（Ruby）
```ruby
require "digest" # for hashing transaction data so we can sign it
require "securerandom" # for generating random nonces when signing

# -------------------------
# Elliptic Curve Parameters
# -------------------------
# y² = x³ + ax + b
$a = 0
$b = 7

# prime field
$p = 2 ** 256 - 2 ** 32 - 2 ** 9 - 2 ** 8 - 2 ** 7 - 2 ** 6 - 2 ** 4 - 1

# number of points on the curve we can hit ("order")
$n = 115792089237316195423570985008687907852837564279074904382605163141518161494337

# generator point (the starting point on the curve used for all calculations)
$G = {
  x: 55066263022277343669578718895168534326250603453777594175500187360389116729240,
  y: 32670510020758816978083085130507043184471273380659243275938904335757337482424,
}

# ---------------
# Modular Inverse: Ruby doesn't have a built-in modinv function
# ---------------
def inverse(a, m = $p)
  m_orig = m         # store original modulus
  a = a % m if a < 0 # make sure a is positive
  prevy, y = 0, 1
  while a > 1
    q = m / a
    y, prevy = prevy - q * y, y
    a, m = m % a, a
  end
  return y % m_orig
end

# ------
# Double: add a point to itself
# ------
def double(point)
  # slope = (3x₁² + a) / 2y₁
  slope = ((3 * point[:x] ** 2 + $a) * inverse((2 * point[:y]), $p)) % $p # using inverse to help with division

  # x = slope² - 2x₁
  x = (slope ** 2 - (2 * point[:x])) % $p

  # y = slope * (x₁ - x) - y₁
  y = (slope * (point[:x] - x) - point[:y]) % $p

  # Return the new point
  return { x: x, y: y }
end

# ---
# Add: add two points together
# ---
def add(point1, point2)
  # double if both points are the same
  if point1 == point2
    return double(point1)
  end

  # slope = (y₁ - y₂) / (x₁ - x₂)
  slope = ((point1[:y] - point2[:y]) * inverse(point1[:x] - point2[:x], $p)) % $p

  # x = slope² - x₁ - x₂
  x = (slope ** 2 - point1[:x] - point2[:x]) % $p

  # y = slope * (x₁ - x) - y₁
  y = ((slope * (point1[:x] - x)) - point1[:y]) % $p

  # Return the new point
  return { x: x, y: y }
end

# --------
# Multiply: use double and add operations to quickly multiply a point by an integer value (i.e. a private key)
# --------
def multiply(k, point = $G)
  # create a copy the initial starting point (for use in addition later on)
  current = point

  # convert integer to binary representation
  binary = k.to_s(2)

  # double and add algorithm for fast multiplication
  binary.split("").drop(1).each do |char| # from left to right, ignoring first binary character
    # 0 = double
    current = double(current)

    # 1 = double and add
    current = add(current, point) if char == "1"
  end

  # return the final point
  current
end

# ----
# Sign
# ----
def sign(private_key, hash, nonce = nil)
  # generate random number if not given
  if nonce == nil
    loop do
      nonce = SecureRandom.hex(32).to_i(16)
      break if nonce < $n # make sure random numer is below the number of points of the curve
    end
  end

  # r = the x value of a random point on the curve
  r = multiply(nonce)[:x] % $n

  # s = nonce⁻¹ * (hash + private_key * r) mod n
  s = (inverse(nonce, $n) * (hash + private_key * r)) % $n # this breaks linearity (compared to schnorr)

  # signature is [r, s]
  return { r: r, s: s }
end

# ------
# Verify
# ------
def verify(public_key, signature, hash)
  # point 1
  point1 = multiply(inverse(signature[:s], $n) * hash)

  # point 2
  point2 = multiply((inverse(signature[:s], $n) * signature[:r]), public_key)

  # add two points together
  point3 = add(point1, point2)

  # check x coordinate of this point matches the x-coordinate of the random point given
  return point3[:x] == signature[:r]
end

# -------------------
# Create A Public Key
# -------------------
# Example private key (in hexadecimal)
private_key = "f94a840f1e1a901843a75dd07ffcc5c84478dc4f987797474c9393ac53ab55e6"

# Public key is the generator point multiplied by the private key
point = multiply(private_key.to_i(16))

# convert x and y values of the public key point to hexadecimal
x = point[:x].to_s(16).rjust(64, "0") # pad with zeros to make sure it's 64 characters (32 bytes)
y = point[:y].to_s(16).rjust(64, "0")

# uncompressed public key (use full x and y coordinates) OLD FORMAT!
public_key_uncompressed = "04" + x + y

# compressed public key (use a prefix to indicate whether y is even or odd)
if (point[:y] % 2 == 0)
  public_key_compressed = "02" + x # y is even
else
  public_key_compressed = "03" + x # y is odd
end

#puts public_key_compressed #=> 024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1

# ------------------
# Sign A Transaction
# ------------------
# A basic structure for holding the transaction data
def tx(scriptsig)
  # Need to calculate a byte indicating the size of upcoming scriptsig in bytes (rough code but does the job)
  size = (scriptsig.length / 2).to_s(16).rjust(2, "0")

  # Raw unsigned transaction data with the scriptsig field (you need to know the correct position)
  return "0100000001b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b00000000#{size}#{scriptsig}ffffffff01983a0000000000001976a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac00000000"
end

# Private key and public key for the locked up bitcoins we want to spend
private_key = "f94a840f1e1a901843a75dd07ffcc5c84478dc4f987797474c9393ac53ab55e6" # sha256("learnmeabitcoin1")
public_key = "024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1"

# NOTE: Need to remove all existing signatures from the transaction data first if there are any

# Put original scriptpubkey as a placeholder in to the scriptsig for the input you want to sign (required)
scriptpubkey = "76a9144299ff317fcd12ef19047df66d72454691797bfc88ac" # just one input in this transaction
transaction = tx(scriptpubkey)

# Append sighash type to transaction data (required)
transaction = transaction + "01000000"

# Get a hash of the transaction data (because we sign the hash of data and not the actual data itself)
hash = Digest::SHA256.hexdigest(Digest::SHA256.digest([transaction].pack("H*")))

# Use elliptic curve mathematics to sign the hash using the private key and nonce
signature = sign(private_key.to_i(16), hash.to_i(16), 123456789) # using a fixed nonce for testing (unsafe)

# Use the low s value (BIP 62: Dealing with malleability)
if (signature[:s] > $n / 2)
  signature[:s] = $n - signature[:s]
end

# Encode the signature in to DER format (slightly awkward format used for signatures in bitcoin transactions)
r = signature[:r].to_s(16).rjust(64, "0")  # convert r to hexadecimal
s = signature[:s].to_s(16).rjust(64, "0")  # convert s to hexadecimal
r = "00" + r if (r[0, 2].to_i(16) >= 0x80) # prepend 00 if first byte is 0x80 or above (DER quirk)
s = "00" + r if (s[0, 2].to_i(16) >= 0x80) # prepend 00 if first byte is 0x80 or above (DER quirk)
der = ""                                   # string for holding our der encoding
r_len = (r.length / 2).to_s(16).rjust(2, "0") # get length of r (in bytes)

s_len = (s.length / 2).to_s(16).rjust(2, "0") # get length of s (in bytes)
der << "02" << r_len << r << "02" << s_len << s   # Add to DER encoding (0x20 byte indicates an integer type in DER)
der_len = (der.length / 2).to_s(16).rjust(2, "0") # get length of DER data (in bytes)
der = "30" + der_len + der # Final DER encoding (0x30 byte incatetes compound object type)

# Append sighashtype to the signature (required) (01 = ALL)
der = der + "01" # without it you get "mandatory-script-verify-flag-failed (Non-canonical DER signature) (code 16)"

# Contruct full unlocking script (P2PKH scripts need original public key the bitcoins were locked to): <size> {signature} <size> {public_key}
scriptsig = (der.length / 2).to_s(16) + der + (public_key.length / 2).to_s(16) + public_key

# Put the full scriptsig in to the original transaction data
transaction = tx(scriptsig)

# Show the signed transaction
puts transaction #=> 0100000001b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b000000006a473044022008f4f37e2d8f74e18c1b8fde2374d5f28402fb8ab7fd1cc5b786aa40851a70cb02201f40afd1627798ee8529095ca4b205498032315240ac322c9d8ff0f205a93a580121024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1ffffffff01983a0000000000001976a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac00000000

# Send the transaction in to the bitcoin network
# $ bitcoin-cli sendrawtransaction [raw transaction data]
```

## 目录
* 椭圆曲线
  * 参数
* 椭圆曲线数学
  * 模反元素
  * 双倍
  * 相加
  * 相乘
* ECDSA
  * 密钥生成
  * 签名
  * 验证
* 比特币中的ECDSA
  1. 创建公钥
  2. 签署交易

### 椭圆曲线
![ECDSA-2.png](img/ECDSA-2%20(1).png)
椭圆曲线。

ECDSA使用椭圆曲线作为数字签名系统的基础。

简而言之，公钥和签名只是椭圆曲线上的点。如果这两个点都是从同一个私钥（一个大数）创建的，它们之间将有一个几何联系，证明创建签名的人也创建（或“拥有”）公钥。

#### 参数
有许多[不同的曲线](http://www.secg.org/sec2-v2.pdf)可以用于ECDSA。Satoshi选择了secp256k1曲线，其参数如下：
```ruby
# y² = x³ + ax + b
$a = 0
$b = 7

# prime field
$p = 2 ** 256 - 2 ** 32 - 2 ** 9 - 2 ** 8 - 2 ** 7 - 2 ** 6 - 2 ** 4 - 1

# number of points on the curve we can hit ("order")
$n = 115792089237316195423570985008687907852837564279074904382605163141518161494337

# generator point (the starting point on the curve used for all calculations)
$G = {
  x: 55066263022277343669578718895168534326250603453777594175500187360389116729240,
  y: 32670510020758816978083085130507043184471273380659243275938904335757337482424,
}
```

* 椭圆曲线是由**方程**y² = x³ + ax + b描述的一组点，因此a和b变量就是从这里得来的。不同的曲线将具有不同的系数值，而a=0和b=7是特定于secp256k1的系数。
* **质数模数**p只是一个数字，用于在进行数学计算时保持所有数字在特定范围内（再次强调，这是特定于secp256k1的）。它是一个质数是密码学能够工作的关键因素，但这是一个旁白。
* 我们可以到达**曲线上的n个点**。这也被称为“阶”，它小于p，并基于选择的生成器点（见下文）。
* 最后，每个曲线都有一个**生成器点**G，它基本上是进行大多数数学运算时使用的起始点。选择该点的确切原因未知[^1]，但通常是因为它提供了高阶（见上文）并且没有任何固有的密码学弱点。

![ECDSA-3.png](img/ECDSA-3%20(1).png)
每条曲线都有一个生成点G。

#### **注**：有限域上的椭圆曲线。

我在本教程中使用的图表显示一个平滑的椭圆曲线，如下所示：
![ECDSA-4.png](img/ECDSA-4%20(1).png)
椭圆曲线在实数上（展示x = 1.123，y = 2.90107701845366的示例点）

然而，在比特币中使用的实际曲线更像这样的散点图：
![ECDSA-5.png](img/ECDSA-5%20(1).png)
椭圆曲线在有限域模47上（展示x=17，y=19的样例点）

这是由于比特币中使用的曲线是在整数有限域上的（即使用模p将数字限制在一定范围内），这打破了在使用实数时可以获得的[连续曲线](https://www.mathsisfun.com/numbers/real-numbers.html)。然而，即使这些图表看起来非常不同，你可以在这**两个曲线上执行的数学操作仍然以相同的方式工作**。

当然，secp256k1曲线具有非常大的p值，因此它更接近于以下类似的图形，除了想象中有与宇宙中的原子一样多的点：
![ECDSA-6.png](img/ECDSA-6%20(1).png)
有限域上的椭圆曲线模2503

但为什么要使用有限域？

因为，在计算机上实现加密时，使用有限域中的整数（例如1、2、3、4、...、p）比使用无限数量的实数（例如0.911722707844879、2.90107701845366、...）更容易处理。在计算机上使用所有这些小数可能会失去精度，因此使用有限的整数集更适合加密。

在本教程的其余部分中，我将展示平滑曲线，因为它在说明目的上看起来更好。

#### 注意：为什么它被称为secp256k1？

这是椭圆曲线密码学中使用的特定曲线之一的绰号：

sec = 高效密码学标准-一个开发商业密码学标准的协会。
p = 素数-用于创建有限域的素数。
256 = 256位-使用的素数域大小。
k = Koblitz-特定类型的曲线。
1 =此类别中的第一个曲线。

### 椭圆曲线数学
您可以在椭圆曲线上的点上执行几个数学运算。主要的两个操作是double（）和add（），这些操作可以组合以执行multiply（）。

这些操作是椭圆曲线密码学的构建块，它们用于在ECDSA中生成公钥和签名。

1. 模反演
2. 双倍
3. 添加
4. 乘

####  模反演
在对曲线上的点执行double（）和add（）操作之前，我们首先需要能够在有限域中找到数字的模反演（请耐心等待）。

您看，在椭圆曲线密码学中，实际上没有直接的“除法”操作，因为所有数学都发生在数字的有限域内。

但是，在有限域中，可以**乘以数字的倒数**以获得与除法相同的结果。
![ECDSA-7.gif](img/ECDSA-7%20(1).gif)
29是47有限域中13的模反元素。
换句话说，在有限域中，如果你从一个特定的数字开始，乘以另一个数字，你可以通过再次乘以所使用的数字的逆元素“倒退”到你开始的数字。

>**当你在一个有限域中有一个质数个元素时，这总是有效的**。质数不能被任何其他数除尽，因此它将结果从模乘法平均分布到域中的每个数字上（不会重复或遗漏某些数字）。因此，通过使用质数作为模数，您可以保证有限域中的每个数字都有一个乘法逆元素（或“除法”操作）。

显然，这是进入椭圆曲线数学的混乱的第一步，但只需将“查找逆元素”视为模算术中的基本工具即可。

然而，并不是所有编程语言都有内置的“模反元素”函数，这就是为什么有时你必须自己实现一个以开始使用椭圆曲线数学，例如在Ruby中：
```ruby
def inverse(a, m = $p)
  m_orig = m         # store original modulus
  a = a % m if a < 0 # make sure a is positive
  prevy, y = 0, 1
  while a > 1
    q = m / a
    y, prevy = prevy - q * y, y
    a, m = m % a, a
  end
  return y % m_orig
end
```
>这个函数使用扩展欧几里得算法（您不需要知道它的工作原理）来寻找一个数的模反元素。这只是比试图通过暴力方法找到反元素更快的方法。

>即将出现的double()和add()方程中包含除法/，因此我们需要能够在素域中计算出一个数的模反元素，然后再继续前进。

>一个数的模反元素通常在数学方程中用⁻¹表示。
![ECDSA-8.png](img/ECDSA-8%20(1).png)
在即将出现的数学方程中，有时会在模p（对素数取模）下找到一个数的反元素，有时会在模n（对曲线上的点数取模）下找到。这取决于方程中使用的模数。

#### 双倍
“Double”一点就是“将”一点加上它自己。

从视觉角度来看，“将”一点加倍，您需要在给定点处绘制曲线的切线，然后找到该线与曲线相交的点（只有一个），然后将该点沿着x轴反射。
![ECDSA-9.png](img/ECDSA-9%20(1).png)
P是用于曲线上一般点的字母。
s在此处指切线的斜率。
![ECDSA-10.png](img/ECDSA-10%20(1).png)
椭圆曲线点加倍。椭圆曲线操作。

>正如您所看到的那样，在这里您实际上并没有“加倍”点的x、y坐标值（就像您在日常算术中所做的那样）。此页面上的“加倍”、“加”和“乘”术语是指我们在椭圆曲线上使用的特定点操作。因此，即使它们与正常数学操作具有相同的名称，在椭圆曲线数学领域中，它们是完全不同的。
有时这可能会有点令人困惑，因为在这些等式中也有日常“加”和“乘”操作。诀窍是记住，每当这些操作在一个点上时，我们使用椭圆曲线操作。当这些操作涉及两个整数时，它就是日常算术。
```ruby
def double(point)
  # slope = (3x₁² + a) / 2y₁
  slope = ((3 * point[:x] ** 2 + $a) * inverse((2 * point[:y]), $p)) % $p # using inverse to help with division

  # x = slope² - 2x₁
  x = (slope ** 2 - (2 * point[:x])) % $p

  # y = slope * (x₁ - x) - y₁
  y = (slope * (point[:x] - x) - point[:y]) % $p

  # Return the new point
  return { x: x, y: y }
end
```

#### 添加
正如预期的那样，在椭圆曲线数学中，“加法”并不是简单的整数加法，但它仍然被称为“加法”。

从视觉上看，要将两个点“加”在一起，您需要在它们之间画一条线，然后找到该线与曲线相交的点（只有一个），然后将该点关于x轴取反。
![ECDSA-11.png](img/ECDSA-11%20(1).png)
Q只是用来表示曲线上第二个一般点的字母。
![ECDSA-12.png](img/ECDSA-12%20(1).png)
椭圆曲线点加法。
```ruby
def add(point1, point2)
  # double if both points are the same
  if point1 == point2
    return double(point1)
  end

  # slope = (y₁ - y₂) / (x₁ - x₂)
  slope = ((point1[:y] - point2[:y]) * inverse(point1[:x] - point2[:x], $p)) % $p

  # x = slope² - x₁ - x₂
  x = (slope ** 2 - point1[:x] - point2[:x]) % $p

  # y = slope * (x₁ - x) - y₁
  y = ((slope * (point1[:x] - x)) - point1[:y]) % $p

  # Return the new point
  return { x: x, y: y }
end
```

#### 乘
现在我们可以“加倍”和“相加”曲线上的点，我们现在可以取任何曲线上的点并将其“乘以”一个整数，以得到一个全新的点。**这个操作是椭圆曲线密码的核心**。

椭圆曲线乘法的最简单方法是重复“将一个点加倍”直到达到您想要乘的数字，这在某种程度上是有效的，但是当乘以大数字（例如比特币中使用的数字）时，这些增量add()操作会使这种方法变得不可能慢。

幸运的是，有一种更快的方法可以在椭圆曲线上执行乘法...

**双倍加算法**
![ECDSA-13.gif](img/ECDSA-13%20(1).gif)
3 * P与一次double()操作和一次add()操作相同：3P = 2P + P。

使用双倍加算法可以更快地进行乘法，其中你可以有效地使用双倍和加法，以尽可能少的操作**达到目标倍数**。

例如，如果你从2开始，想要到达128，进行6次double()操作比进行64次add()操作更快。

但是，如何知道你需要多少次double和add操作才能到达目标倍数呢？

惊人地，如果将任何整数转换为其二进制表示，其中的1和0将提供一个序列的double()和add()操作图，使你能够达到该倍数。
从左到右忽略第一个数字：
* 0 = double
* 1 = double and add。
例如：
```
e.g. 1 * 21

21 = 10101 (binary)
      │││└ double and add = 21
      ││└─ double         = 10
      │└── double and add = 5
      └─── double         = 2
                            1  <- start here
```
无论如何，下面是在 Ruby 代码中使用双倍加算法进行椭圆曲线乘法的示例：
```ruby
def multiply(k, point = $G)
  # create a copy the initial starting point (for use in addition later on)
  current = point

  # convert integer to binary representation
  binary = k.to_s(2)

  # double and add algorithm for fast multiplication
  binary.split("").drop(1).each do |char| # from left to right, ignoring first binary character
    # 0 = double
    current = double(current)

    # 1 = double and add
    current = add(current, point) if char == "1"
  end

  # return the final point
  current
end
```
>大多数ECDSA中的乘法操作都从生成点G开始。

>每个乘法操作都从double()开始。这是因为你从一个单点（例如生成点）开始。

### ECDSA
现在我们知道了椭圆曲线的结构，并且我们有了一个用于点乘法的multiply()操作，我们实际上可以将其作为创建数字签名的系统的基础。

以下系统称为椭圆曲线数字签名算法，简称ECDSA。

* 密钥生成
* 签名
* 验证

#### 密钥生成
我们使用椭圆曲线乘法创建密钥对：

* 私钥d-一个在[0...n-1]之间的大随机生成数
* 公钥Q-**生成点G**乘以此随机数d。
![ECDSA-14.png](img/ECDSA-14%20(1).gif)
d是私钥（一个整数）
G是生成点（一个点）
Q是公钥（一个点）
![ECDSA-15.png](img/ECDSA-15%20(1).png)
在椭圆曲线加密中，私钥只是一个大的随机整数（小于曲线上的点数），其对应的公钥只是曲线上的一个点。

例如：
```private key = 112757557418114203588093402336452206775565751179231977388358956335153294300646
public key  = {
    x: 33886286099813419182054595252042348742146950914608322024530631065951421850289, 
    y: 9529752953487881233694078263953407116222499632359298014255097182349749987176
}
```
**陷门函数**：您可以将您的公钥提供给任何人，而您的私钥将保持保密。
![ECDSA-16.png](img/ECDSA-16%20(1).png)

重要的一点是，给定公钥点Q，没有简单的方法可以确定用于创建它的私钥d。唯一确定私钥的方法是手动将生成器点G乘以不同的数字，看是否可以得到相同的公钥，而这种暴力方法如果有人使用非常大的数字作为他们的私钥，将是不可能的。

因此，椭圆曲线乘法被称为陷门函数（因为单向易行而逆向困难），这是所有公钥密码学的关键组成部分。

此外，私钥和公钥之间的单向数学连接意味着您可以独立使用它们来计算以后在椭圆曲线上相同的点，这在构建用于创建数字签名的系统时非常有用。

#### 签名
要签署一条消息，您需要三件事：

1. 随机数k — 这引入了随机元素到我们的签名中，这对于安全性很重要。这意味着我们生成的每个签名都将是不同的，即使我们两次签署相同的消息。
2. 消息哈希z — 这是我们要签署的消息的哈希。*哈希消息*为我们提供了一个小而独特的指纹，签署这个指纹比签署大块数据更有效率。您可以选择使用哪种哈希算法，但与secp256k1最常用的是SHA-256。
3. 私钥d — 公钥的源（我们已经公开了）。

然后，实际的签名由两部分组成：

* r — **曲线上的随机点**。我们将随机数k乘以生成器点，以获得随机点R。我们实际上只使用这个点的x坐标，并称其为小写r。
* s — **伴随随机点的数字**。这是一个由消息哈希z和私钥d的组合创建的唯一数字，也与随机点r绑定在一起。
![ECDSA-16.gif](img/ecdsa-16%20(1).gif)
![ECDSA-17.png](img/ECDSA-17%20(1).png)

⁻¹符号表示该数字的模反元素。在这里，模乘法逆元是在曲线上的点数模下找到的。

这两个[r，s]值是“数字签名”。

例如：
```
random number   (k): 12345
message:             ECDSA is the most fun I have ever experienced
sha256(message) (z): 103318048148376957923607078689899464500752411597387986125144636642406244063093
private key     (d): 112757557418114203588093402336452206775565751179231977388358956335153294300646

random point (k*G = R): {
  x = 108607064596551879580190606910245687803607295064141551927605737287325610911759,
  y = 6661302038839728943522144359728938428925407345457796456954441906546235843221
}
signature: r = R[x], s = k⁻¹ * (z + r * d): {
  r = 108607064596551879580190606910245687803607295064141551927605737287325610911759, 
  s = 73791001770378044883749956175832052998232581925633570497458784569540878807131
}
```
简而言之，唯一的 s 值提供了到随机生成点 r 的路径。

你可以将这两个信息提供给别人，从公钥点 Q 开始，他们可以使用 s 值帮助他们到达随机点 r。关键在于，只有拥有相应私钥 d 的人才能创建有效的到该点的路径。

这个路径也将消息哈希 z 编码进去，这就是让我们为消息创建签名的有效方法；除非知道创建它的私钥，否则没有人能够通过消息哈希创建从公钥到曲线上随机点的路径。
```ruby
def sign(private_key, hash, nonce = nil)
  # generate random number if not given
  if nonce == nil
    loop do
      nonce = SecureRandom.hex(32).to_i(16)
      break if nonce < $n # make sure random numer is below the number of points of the curve
    end
  end

  # r = the x value of a random point on the curve
  r = multiply(nonce)[:x] % $n

  # s = nonce⁻¹ * (hash + private_key * r) mod n
  s = (inverse(nonce, $n) * (hash + private_key * r)) % $n # this breaks linearity (compared to schnorr)

  # signature is [r, s]
  return { r: r, s: s }
end
```
>**Nonce**：在密码学中，随机数有时被称为“nonce”，它代表“一次性数字”。

**为什么需要每次生成一个随机点？（数学解释）**

>如果您多次使用相同的k值（即您为多个签名使用相同的随机点），任何人都可以从您的签名中计算出您的私钥。

例如，假设我们得到了两条使用相同的k值生成的已签名消息。

对于每个已签名消息，我们都有消息哈希z以及来自各自签名的r和s值：
```
Signed Message 1: (z₁, r₁, s₁)
Signed Message 2: (z₂, r₂, s₂)
```
然而，由于每次使用相同的k值生成随机点（R = k * G），因此这些签名中的r值（R的x坐标）也将相同：
```
Signed Message 1: (z₁, r, s₁)
Signed Message 2: (z₂, r, s₂)
```
那么我们如何利用这些信息来计算私钥d？

首先，我们知道每个签名中的s值是使用s = k⁻¹(z + r * d) mod n计算的，因此：
```
s₁ = k⁻¹(z₁ + r * d) mod n
s₂ = k⁻¹(z₂ + r * d) mod n
```
由于这两个方程现在具有相同的k值，因此我们可以将它们作为一组同时方程来解决，以求出k的值。

为此，我们首先重新排列第二个方程，以使r * d单独出现：
```
s₂ = k⁻¹(z₂ + r * d) mod n
r * d = k * s₂ - z₂ mod n
```
然后我们可以将此代入第一个方程，并重排以获得k：
```
s₁ = k⁻¹(z₁ + r * d) mod n
s₁ = k⁻¹(z₁ + (k * s₂ - z₂)) mod n
k = (z₁ - z₂) * (s₁ - s₂)⁻¹ mod n
```
>请记住，乘以 (s₁ - s₂)⁻¹ 意味着乘以 (s₁ - s₂) 的模逆元，这在椭圆曲线数学中等同于“除法”。

当我们计算出 k 后，我们可以再次使用它在 s = k⁻¹(z + r * d) mod n 中计算出 d。

因此，重新排列第一个方程（您可以使用任何一个）以获得单独的 d：
```
s₁ = k⁻¹(z₁ + r * d) mod n
d = (k * s₁ - z₁) * r⁻¹ mod n
```
由于我们已经知道了（z₁，r，s₁），并且刚刚计算出了k，因此我们可以将它们全部代入此方程式中，以求出私钥d。

用数学符号表示，私钥恢复的过程如下：
![ECDSA-18.png](img/ECDSA-18%20(1).png)
**因此，请确保每次创建签名时都使用安全随机的k值**。如果有人发现您在为同一公钥签署不同消息时使用了相同的r值，他们只需要几毫秒就可以恢复您的私钥。

>[2011年，黑客们发现索尼在生成其签名时使用了相同的k值](https://arstechnica.com/gaming/2010/12/ps3-hacked-through-poor-implementation-of-cryptography/)，因此他们成功获取了PS3的私钥。

以下是在Ruby中使用相同的k值从两个签名中恢复私钥的示例：
```ruby
require 'digest' # used for hashing messages before signing

# Note: This code uses the previously defined inverse(), double(), add(), multiply(), and sign() functions

# -------------
# Sign Messages
# -------------

# 0. Keys
prv = 1111222233334444555566667777888899990000 # any old private key
pub = multiply(prv)

# 1. Create first signed message
k    = 800000
z1   = Digest::SHA256.hexdigest("Just a simple message.").to_i(16)
sig1 = sign(prv, z1, k)

# 2. Create second signed message
k    = 800000 # Using the same value for k!
z2   = Digest::SHA256.hexdigest("I have used the same k value.").to_i(16)
sig2 = sign(prv, z2, k)

# --------------------
# Private Key Recovery
# --------------------
# k = (z₁ - z₂) * (s1 - s₂)⁻¹  mod n
# d = (k * s₁ - z₁) * r⁻¹    mod n

# 1. Work out k (note: result may be the additive inverse of original k, but it still works fine)
k_calculated = ((z1 - z2) * inverse(sig1[:s] - sig2[:s], $n)) % $n

# 2. Work out d (the original private key)
d_calculated = ((k_calculated * sig1[:s] - z1) * inverse(sig1[:r], $n)) % $n
puts d_calculated #=> 1111222233334444555566667777888899990000
```

#### 验证
您可以通过以下三个方面验证消息及其签名：

1. **公钥** Q - 这是声称创建签名的人的公钥。
2. **消息** - 签名的数据。我们可以自己进行哈希以获得消息哈希 z。
3. **签名** [r，s] - 这是为上述消息创建的签名，据称由具有公钥的私钥的人创建。

然后，我们使用这三个数据来计算曲线上的两个点：

* **点1**。从生成器点 G 开始，乘以 inverse(s) * z。
* **点2**。从公钥点 Q 开始，乘以 inverse(s) * r。
现在，我们可以将这些点相加以得到**点3**：
![ECDSA-19.png](img/ecdsa-19%20(1).gif)
![ECDSA-20.png](img/ECDSA-20%20(1).png)

如果第三点与给定的随机点相匹配，则签名有效。

例如：
```
message:             ECDSA is the most fun I have ever experienced
sha256(message) (z): 103318048148376957923607078689899464500752411597387986125144636642406244063093
signature (r,s): {
  r = 108607064596551879580190606910245687803607295064141551927605737287325610911759,
  s = 73791001770378044883749956175832052998232581925633570497458784569540878807131
}
public key (Q): {
  x = 33886286099813419182054595252042348742146950914608322024530631065951421850289,
  y = 9529752953487881233694078263953407116222499632359298014255097182349749987176
}

verification (s⁻¹ * z)G + (s⁻¹ * r)Q: {
  x = 108607064596551879580190606910245687803607295064141551927605737287325610911759, <- matches r (x-coordinate of random point)
  y = 6661302038839728943522144359728938428925407345457796456954441906546235843221
}
```
换句话说，此消息的签名只能由创建公钥的实际私钥持有者创建。除了知道该公钥的私钥d的人以外，没有人可以给您可以与公钥Q组合使用以达到随机点R的s值。

如果更改签名消息的内容或尝试将签名与不同的公钥一起使用，则生成的第三个点将无法与签名中给出的随机点匹配，签名验证将失败。
```ruby
def verify(public_key, signature, hash)
  # point 1
  point1 = multiply(inverse(signature[:s], $n) * hash)

  # point 2
  point2 = multiply((inverse(signature[:s], $n) * signature[:r]), public_key)

  # add two points together
  point3 = add(point1, point2)

  # check x coordinate of this point matches the x-coordinate of the random point given
  return point3[:x] == signature[:r]
end
```
**为什么这个方法有效？（数学解释）**
签名：

创建签名的人首先使用随机数k生成曲线上的随机点：
```
R = k * G
```
然后，他们使用他们的私钥d和消息z的哈希（以及R的x坐标r和随机数k）计算一个辅助数字。
```
s = k⁻¹ * (z + r * d)v
```
验证：

下列方程式允许您使用公钥 Q、消息 z 的哈希值和给定的 s 值计算相同的点：
```
R = (s⁻¹ * z)G + (s⁻¹ * r)Q
```
我们现在可以重新排列这个方程，并替换一些值来证明这个方程确实能够到达相同的点。

首先，公钥Q是d * G，因此：
```
R = (s⁻¹ * z)G + (s⁻¹ * r)d*G
```
如果我们重新排列这个方程，我们可以得到：
```
R = (s⁻¹ * z)G + (s⁻¹ * r * d)G
R = s⁻¹ * (z + r * d) * G
```
现在，请记住 s = k⁻¹ * (z + r * d)。如果我们重新排列这个式子，让 k 单独出现，我们得到 k = s⁻¹ * (z + r * d)，并将其代入上面的方程中：
```
R = k * G
```
这就是用于生成随机点的相同计算方法。

### ECDSA在比特币中的应用
在比特币中，ECDSA用于以下两种情况：

1. 创建公钥
2. 签署交易
在这两种情况下，您只是使用与上述相同的椭圆曲线数学。

大多数情况下，唯一棘手的部分是将数据转换为正确的格式。

#### 1. 创建公钥
![ECDSA-21.png](img/ecdsa-21%20(1).png)
公钥可以放置在*输出*的**锁定脚本**上。

公钥只是生成点乘以您的私钥。

因此，它是由x和y坐标组成的**曲线上的一个点**：
```
private key = 112757557418114203588093402336452206775565751179231977388358956335153294300646
public key  = {
    x: 33886286099813419182054595252042348742146950914608322024530631065951421850289, 
    y: 9529752953487881233694078263953407116222499632359298014255097182349749987176
}
```
将这个点转换为比特币中常用的格式时，您只需将每个坐标转换为*十六进制*即可：
```
public key  = {
    x: 4aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1, 
    y: 1511a626b232de4ed05b204bd9eccaf1b79f5752e14dd1e847aa2f4db6a52768
}

public key = 4aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b11511a626b232de4ed05b204bd9eccaf1b79f5752e14dd1e847aa2f4db6a52768
```
>不要忘记通过必要的前导零来将每个十六进制坐标填充到32个字节（64个字符）。

然而，ECDSA中使用的椭圆曲线的一个有趣之处是每个x坐标都将有两个可能的y坐标（一个是偶数，另一个是奇数）。
![ECDSA-22.png](img/ECDSA-22%20(1).png)
因此，我们可以使用前缀来指示给定点使用的两个可能y坐标中的哪一个，而不必一直使用完整的y坐标。这将减少格式化公钥的大小，并减少构建交易所需的数据量。
```
Public key prefixes:

02 = compressed (y is even)
03 = compressed (y is odd)
04 = uncompressed (full y is included)
```
通过我们的示例公钥，我们可以看到 y 坐标是偶数，因此我们可以在完整的 x 坐标旁边使用 02 前缀来缩短编码的公钥：
```
public key (uncompressed) = 044aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b11511a626b232de4ed05b204bd9eccaf1b79f5752e14dd1e847aa2f4db6a52768
public key (compressed)   = 024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1
```
这些缩短的公钥被称为**压缩公钥**，它们是比特币中使用的首选格式。

最后，正如前面提到的，这些公钥可以用于接收比特币，方法是将它们放入交易中输出的 *scriptPubKey*（“锁定脚本”）中。这实际上将一定数量的比特币与一个公钥联系起来，只有这个公钥的所有者才能在以后花费它们。

以下代码展示了如何将私钥转换为可用于接收比特币的压缩公钥：
```ruby
# Example private key (in hexadecimal)
private_key = "f94a840f1e1a901843a75dd07ffcc5c84478dc4f987797474c9393ac53ab55e6"

# Public key is the generator point multiplied by the private key
point = multiply(private_key.to_i(16))

# convert x and y values of the public key point to hexadecimal
x = point[:x].to_s(16).rjust(64, "0") # pad with zeros to make sure it's 64 characters (32 bytes)
y = point[:y].to_s(16).rjust(64, "0")

# uncompressed public key (use full x and y coordinates) OLD FORMAT!
public_key_uncompressed = "04" + x + y

# compressed public key (use a prefix to indicate whether y is even or odd)
if (point[:y] % 2 == 0)
  public_key_compressed = "02" + x # y is even
else
  public_key_compressed = "03" + x # y is odd
end

#puts public_key_compressed #=> 024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1
```

>私钥只是一个大的随机生成的数字。如果你有一个私钥的十六进制字符串，你可以将它转换回整数，以便用于椭圆曲线乘法。

####2. 签署交易
![ECDSA-23.png](img/ECDSA-23%20(1).png)

当您想向某人发送比特币时，您需要能够提供数字签名，证明您拥有比特币锁定的公钥的私钥。

这包括：

1. 构建交易。这将是我们签名的消息。
2. 使用您的私钥对此消息进行签名。
3. 将此签名放回交易中。
在比特币中签署交易可能有点棘手，但大多数情况下只是将数据格式正确地输入。以下是签署传统比特币交易的完整步骤：
![ECDSA-24.png](img/ECDSA-24%20(1).png)

0. 创建交易
1. 移除现有的解锁脚本
2. 将锁定脚本作为占位符
3. 在交易数据中附加签名哈希类型
4. 对交易数据进行哈希运算
5. 签名交易哈希
6. 使用低 s 值
7. 对签名进行 DER 编码
8. 在 DER 编码的签名后附加签名哈希类型
9. 构建解锁脚本
10. 将解锁脚本插入到交易中。

**0. 创建交易。**
首先，您需要创建一些交易数据，以花费您拥有的比特币。

这基本上是选择要**花费的输入**，然后创建要将这些已花费的输入**锁定到的输出**。换句话说，它描述了硬币的移动。
![ECDSA-25.png](img/ECDSA-25%20(1).png)

我没有在这里解释比特币*交易数据*结构，所以如果你从没看过原始交易，这部分可能会让你感到困惑，但这就是一个交易的样子。
```
Raw Transaction (Unsigned):

version: 01000000
inputs:  01
  txid: b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b
  vout: 00000000
  scriptsigsize: 00
  scriptsig: 
  sequence: ffffffff
outputs: 01
  amount: 983a000000000000
  scriptpubkeysize: 19
  scriptpubkey: 76a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac
locktime: 00000000
```

>*TXID*用于引用原始交易数据中的输入实际上是反向字节顺序。这是比特币的一个怪癖。

在这里，我通过引用先前交易的txid和特定的vout（输出）选择了要花费的比特币，并创建了一个包含一定数量比特币和新脚本公钥锁的新输出。此输出使用了*P2PKH*锁定，但现在这不重要。

最重要的是，注意**输入的scriptSig目前为空**。这是我们的签名将要放置的解锁脚本的位置。

无论如何，我们现在准备好签署此交易数据并“解锁”我们选择的输入了...

**1. 删除现有的解锁脚本**
当我们签署交易时，我们只签署描述硬币移动的数据。因此，如果您已经为其他输入创建了scriptSig（解锁脚本），请暂时将其从交易中删除。

在我的交易中，我只解锁了一个输入，因此不需要删除任何现有的scriptSig。

**2. 将锁定脚本作为占位符**
在签署输入之前，我们需要将该输入的原始锁定脚本（scriptpubkey）放置到即将放置签名的地方（scriptsig）。

我不完全确定为什么要这样做，但我猜这有助于识别我们要签名的特定输入，并显示我们知道要花费的输出的原始锁定脚本。

无论如何，在查看此*输入*的原始交易数据时，我们可以看到它具有76a9144299ff317fcd12ef19047df66d72454691797bfc88ac的*P2PKH*锁定脚本，因此我们将其放置到scriptsig中作为占位符：
```
Raw Transaction (Unsigned):

version: 01000000
inputs:  01
  txid: b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b
  vout: 00000000
  scriptsigsize: 19
  scriptsig: 76a9144299ff317fcd12ef19047df66d72454691797bfc88ac
  sequence: ffffffff
outputs: 01
  amount: 983a000000000000
  scriptpubkeysize: 19
  scriptpubkey: 76a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac
locktime: 00000000
```
>不要忘记，这个占位符也会暂时调整scriptsigsize的值。

**3. 将签名哈希类型追加到交易数据中。**
![ECDSA-26.png](img/ECDSA-26%20(1).png)
```
Signature Hash Types:

0x01 = SIGHASH_ALL
0x02 = SIGHASH_NONE
0x03 = SIGHASH_SINGLE
0x81 = SIGHASH_ANYONECANPAY | SIGHASH_ALL
0x82 = SIGHASH_ANYONECANPAY | SIGHASH_NONE
0x83 = SIGHASH_ANYONECANPAY | SIGHASH_SINGLE
```
此时，我们可以通过在事务数据末尾添加签名哈希类型（SIGHASH）来指示我们将签署多少交易结构。

最常见的是SIGHASH_ALL（0x01），它表示签名**覆盖交易中的所有输入和输出**，这意味着没有人可以在以后添加任何其他输入或输出。

因此，这就是我们的原始未签名交易的样子：
```
Raw Transaction (Unsigned):

version: 01000000
inputs:  01
  txid: b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b
  vout: 00000000
  scriptsigsize: 19
  scriptsig: 76a9144299ff317fcd12ef19047df66d72454691797bfc88ac
  sequence: ffffffff
outputs: 01
  amount: 983a000000000000
  scriptpubkeysize: 19
  scriptpubkey: 76a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac
locktime: 00000000
sighash: 01000000
```
>这里的签名哈希是4个字节长，并且采用*小端字节*顺序。

>根据您选择的签名哈希类型，您可能需要调整交易数据。但对于SIGHASH_ALL，当前的交易结构已准备好进行签名。

第四步：对交易数据进行哈希
现在我们已经准备好了未签名的原始交易，可以对其进行哈希以便签名。

比特币在哈希时使用双重SHA-256（也称为“hash256”），因此如果我们对当前的未签名交易进行序列化并进行哈希，我们会得到：
```message: 0100000001b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b000000001976a9144299ff317fcd12ef19047df66d72454691797bfc88acffffffff01983a0000000000001976a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac0000000001000000

hash256(message): a6b4103f527dfe43dfbadf530c247bac8a98b7463c7c6ad38eed97021d18ffcb
```
如果我们将这个哈希转换为整数，我们会得到：
```
hash256(message): 75402077471587956851360588120356244127735644006942973877340910814730793844683
```
>hash256（message）是sha256（message）的简写形式。在比特币中，几乎总是使用双重SHA-256来哈希数据。

**5. 签署交易哈希**
现在，我们可以像对任何其他消息进行ECDSA签名一样对此消息哈希z进行签名。我们只需要我们的私钥d和一个随机生成的数字k。

这是此交易的签名示例：
```
random number    (k): 123456789
hash256(message) (z): 75402077471587956851360588120356244127735644006942973877340910814730793844683
private key      (d): 112757557418114203588093402336452206775565751179231977388358956335153294300646

random point (k*G = R): {
  x = 4051293998585674784991639592782214972820158391371785981004352359465450369227,
  y = 88166831356626186178414913298033275054086243781277878360288998796587140930350
}

signature: r = R[x], s = k⁻¹ * (z + r * d): {
  r = 4051293998585674784991639592782214972820158391371785981004352359465450369227, 
  s = 101656099268479774907861155236876278987061611115278341531512875302287938750185
}
```

**6. 使用低s值**
有趣的是，ECDSA实际上有两个可能的s值可以生成有效的签名。我们将其中一个称为“高”s值，另一个称为“低”s值。

在数学术语中，另一个有效的s值只是在n的有限域中当前s值的加法逆元。因此，在进行签名验证时，这两个s值实际上都会得到相同的随机点R的x坐标。
![ECDSA-27.png](img/ECDSA-27%20(1).png)
在签名验证过程中，s值的加法逆元计算出曲线上的相反点，但第三个点的x坐标仍然相同。

然而，在比特币中，这个特性有点烦人，因为它意味着任何人都可以在我们将交易发送到网络后反转s值，这将改变结果的*TXID*。这并不会改变交易的实际结构（资金以相同的方式移动到相同的位置），但它确实意味着我们失去了在区块链中可靠跟踪交易的能力。

因此，为了帮助解决高/低s值问题，[BIP 62](https://github.com/bitcoin/bips/blob/master/bip-0062.mediawiki)引入了一条规则，即所有签名默认使用低s值，否则交易将被视为非标准交易，节点将不会中继。

如果您的s值在有限域n的上半部分（即s > n/2），则您会知道您已经得到了“高”s值。如果是这种情况（例如我们的s值），我们只需从n中减去s值即可得到“低”s值。
```
n     = 115792089237316195423570985008687907852837564279074904382605163141518161494337

s     = 101656099268479774907861155236876278987061611115278341531512875302287938750185  <- high s value
n - s = 14135989968836420515709829771811628865775953163796562851092287839230222744152   <- low s value
```
在Ruby代码中，这非常简单：
```ruby
if (signature[:s] > $n / 2)
  signature[:s] = $n - signature[:s]
end
```
正如我所说，两个s值都是有效的 - 只是在比特币中，我们使用低s值来帮助防止交易可塑性。

**7. 对签名进行DER编码**
下一步是将**签名格式化**，使其准备好放入比特币交易中。

比特币使用[DER编码](https://www.itu.int/ITU-T/studygroups/com17/languages/X.690-0207.pdf)来进行签名，这有点冗长，但这是Satoshi选择的，因此我们现在在编码交易签名时必须使用它。

为什么Satoshi决定使用DER编码呢？...

>我猜Satoshi不知道ECDSA签名的内部结构，只是使用了OpenSSL给他的内容。如果它不需要硬分叉更改（需要网络上的每个钱包和验证节点升级），我们早就改变了它。[-Pieter Wuille](https://bitcoin.stackexchange.com/a/14421/24926)

无论如何，在比特币中DER编码签名基本上涉及将r和s值转换为*十六进制*，并在它们之间添加一些字节以指示数据的长度和类型。还有一个怪癖，如果r或s的第一个字节为0x80或以上，我们分别在它们前面添加一个零0x00字节。但除此之外，它非常简单。

因此，如果这是我们的原始签名：
```
signature: {
  r = 4051293998585674784991639592782214972820158391371785981004352359465450369227, 
  s = 14135989968836420515709829771811628865775953163796562851092287839230222744152
}
```
这是DER编码的外观：
```
der encoded:
type: 30
  length: 44
  type:   02
    length: 20
    r:      08f4f37e2d8f74e18c1b8fde2374d5f28402fb8ab7fd1cc5b786aa40851a70cb
  type:   02
    length: 20
    s:      1f40afd1627798ee8529095ca4b205498032315240ac322c9d8ff0f205a93a58
```
类型为0x30和0x20的字节始终相同。在将签名编码为DER格式时，只需要更改长度和实际数据字段即可。

有关更多详细信息，请参阅[stackexchange.com](https://bitcoin.stackexchange.com/questions/77191/what-is-the-maximum-size-of-a-der-encoded-ecdsa-signature)上的帖子。

**8.将签名哈希类型附加到DER编码的签名**
在将签名编码为DER格式后，我们再次附加SIGHASH类型，以指示此签名适用于交易数据的多少。这必须与我们在步骤3中选择的签名哈希类型相同。

因此，我们编码后的签名现在看起来像这样：
```
der encoded (with SIGHASH):
type: 30
  length: 44
  type:   02
    length: 20
    r:      08f4f37e2d8f74e18c1b8fde2374d5f28402fb8ab7fd1cc5b786aa40851a70cb
  type:   02
    length: 20
    s:      1f40afd1627798ee8529095ca4b205498032315240ac322c9d8ff0f205a93a58
sighash: 01
```
>这里的SIGHASH类型是1个字节，尽管我们在哈希前附加的SIGHASH类型是4个字节。这只是比特币的另一个怪癖。[^3]

我们在哈希之前附加SIGHASH类型的原因是，它允许我们在创建签名之前承诺该SIGHASH类型。换句话说，如果有人将第二个值更改为SIGHASH_ANYONECANPAY之类的值（这意味着任何人都可以添加更多的输入到交易中），它将不匹配我们在哈希和签署初始交易数据时选择的SIGHASH值。

无论如何，如果我们序列化所有这些数据，我们编码的签名看起来像：
```
der encoded (with SIGHASH):
3044022008f4f37e2d8f74e18c1b8fde2374d5f28402fb8ab7fd1cc5b786aa40851a70cb02201f40afd1627798ee8529095ca4b205498032315240ac322c9d8ff0f205a93a5801
```

**9. 构造解锁脚本**
作为最后一步，我们需要将此签名放入一个脚本中，以解锁输入。
![ECDSA-28.png](img/P2PKH-2%20(1).gif)

我不会在这里涉及脚本的机制，但是让我们假设我们选择的输入具有以下P2PKH锁定脚本（脚本公钥）：
```
scriptpubkey:

asm: OP_DUP OP_HASH [length] [public key hash] OP_EQUALVERIFY OP_CHECKSIG
hex: 76a9144299ff317fcd12ef19047df66d72454691797bfc88ac
```
简而言之，该输入已被锁定到我们公钥的哈希值。要解锁它，我们需要提供有效的签名和原始公钥。

因此，如果这是我们的公钥和签名：
```
public key: 024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1
signature:  3044022008f4f37e2d8f74e18c1b8fde2374d5f28402fb8ab7fd1cc5b786aa40851a70cb02201f40afd1627798ee8529095ca4b205498032315240ac322c9d8ff0f205a93a5801
```
这是我们的解锁脚本（scriptsig）的样子：
```scriptpubkey:

asm: [signature length] [signature] [public key length] [public key]
hex: 473044022008f4f37e2d8f74e18c1b8fde2374d5f28402fb8ab7fd1cc5b786aa40851a70cb02201f40afd1627798ee8529095ca4b205498032315240ac322c9d8ff0f205a93a580121024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1
```
这只是DER编码的签名，后跟公钥，每个前面都有0x47和0x21字节以指示它们的长度。

**10. 将解锁脚本插入交易中**
最后，我们将这个解锁脚本插入到交易中：
```
Raw Transaction (Signed):

version: 01000000
inputs:  01
  txid: b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b
  vout: 00000000
  scriptsigsize: 6a
  scriptsig: 473044022008f4f37e2d8f74e18c1b8fde2374d5f28402fb8ab7fd1cc5b786aa40851a70cb02201f40afd1627798ee8529095ca4b205498032315240ac322c9d8ff0f205a93a580121024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1
  sequence: ffffffff
outputs: 01
  amount: 983a000000000000
  scriptpubkeysize: 19
  scriptpubkey: 76a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac
locktime: 00000000
```

**注意**：对于每个要解锁的输入，重复步骤1-9。

最后，如果我们序列化此交易数据，我们得到：
```
Raw Transaction (Signed):

0100000001b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b000000006a473044022008f4f37e2d8f74e18c1b8fde2374d5f28402fb8ab7fd1cc5b786aa40851a70cb02201f40afd1627798ee8529095ca4b205498032315240ac322c9d8ff0f205a93a580121024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1ffffffff01983a0000000000001976a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac00000000
```
这是一个有效的交易，我们可以将其发送到比特币网络中。

bitcoin-cli sendrawtransaction
将一些原始交易数据发送到比特币网络。例如：
```
$ bitcoin-cli sendrawtransaction 0100000001b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b000000006a473044022008f4f37e2d8f74e18c1b8fde2374d5f28402fb8ab7fd1cc5b786aa40851a70cb02201f40afd1627798ee8529095ca4b205498032315240ac322c9d8ff0f205a93a580121024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1ffffffff01983a0000000000001976a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac00000000
```

以下代码展示了如何使用上述步骤签署交易：
```ruby
# A basic structure for holding the transaction data
def tx(scriptsig)
  # Need to calculate a byte indicating the size of upcoming scriptsig in bytes (rough code but does the job)
  size = (scriptsig.length / 2).to_s(16).rjust(2, "0")

  # Raw unsigned transaction data with the scriptsig field (you need to know the correct position)
  return "0100000001b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b00000000#{size}#{scriptsig}ffffffff01983a0000000000001976a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac00000000"
end

# Private key and public key for the locked up bitcoins we want to spend
private_key = "f94a840f1e1a901843a75dd07ffcc5c84478dc4f987797474c9393ac53ab55e6" # sha256("learnmeabitcoin1")
public_key = "024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1"

# NOTE: Need to remove all existing signatures from the transaction data first if there are any

# Put original scriptpubkey as a placeholder in to the scriptsig for the input you want to sign (required)
scriptpubkey = "76a9144299ff317fcd12ef19047df66d72454691797bfc88ac" # just one input in this transaction
transaction = tx(scriptpubkey)

# Append sighash type to transaction data (required)
transaction = transaction + "01000000"

# Get a hash of the transaction data (because we sign the hash of data and not the actual data itself)
hash = Digest::SHA256.hexdigest(Digest::SHA256.digest([transaction].pack("H*")))

# Use elliptic curve mathematics to sign the hash using the private key and nonce
signature = sign(private_key.to_i(16), hash.to_i(16), 123456789) # using a fixed nonce for testing (unsafe)

# Use the low s value (BIP 62: Dealing with malleability)
if (signature[:s] > $n / 2)
  signature[:s] = $n - signature[:s]
end

# Encode the signature in to DER format (slightly awkward format used for signatures in bitcoin transactions)
r = signature[:r].to_s(16).rjust(64, "0")  # convert r to hexadecimal
s = signature[:s].to_s(16).rjust(64, "0")  # convert s to hexadecimal
r = "00" + r if (r[0, 2].to_i(16) >= 0x80) # prepend 00 if first byte is 0x80 or above (DER quirk)
s = "00" + r if (s[0, 2].to_i(16) >= 0x80) # prepend 00 if first byte is 0x80 or above (DER quirk)
der = ""                                   # string for holding our der encoding
r_len = (r.length / 2).to_s(16).rjust(2, "0") # get length of r (in bytes)

s_len = (s.length / 2).to_s(16).rjust(2, "0") # get length of s (in bytes)
der << "02" << r_len << r << "02" << s_len << s   # Add to DER encoding (0x20 byte indicates an integer type in DER)
der_len = (der.length / 2).to_s(16).rjust(2, "0") # get length of DER data (in bytes)
der = "30" + der_len + der # Final DER encoding (0x30 byte incatetes compound object type)

# Append sighashtype to the signature (required) (01 = ALL)
der = der + "01" # without it you get "mandatory-script-verify-flag-failed (Non-canonical DER signature) (code 16)"

# Contruct full unlocking script (P2PKH scripts need original public key the bitcoins were locked to): <size> {signature} <size> {public_key}
scriptsig = (der.length / 2).to_s(16) + der + (public_key.length / 2).to_s(16) + public_key

# Put the full scriptsig in to the original transaction data
transaction = tx(scriptsig)

# Show the signed transaction
puts transaction #=> 0100000001b7994a0db2f373a29227e1d90da883c6ce1cb0dd2d6812e4558041ebbbcfa54b000000006a473044022008f4f37e2d8f74e18c1b8fde2374d5f28402fb8ab7fd1cc5b786aa40851a70cb02201f40afd1627798ee8529095ca4b205498032315240ac322c9d8ff0f205a93a580121024aeaf55040fa16de37303d13ca1dde85f4ca9baa36e2963a27a1c0c1165fe2b1ffffffff01983a0000000000001976a914b3e2819b6262e0b1f19fc7229d75677f347c91ac88ac00000000

# Send the transaction in to the bitcoin network
# $ bitcoin-cli sendrawtransaction [raw transaction data]
```
准备签名数据可能会很棘手，因此可能需要尝试几次才能做到正确。DER编码看起来尤其复杂，但归根结底，它只是签名的r和s值之间插入一些字节，以指示数据的长度。

## 总结
ECDSA的最佳学习方法是尝试自己编写代码。

最困难的部分通常不是椭圆曲线数学，而是为在比特币交易中使用的签名准备和格式化签名。此外，并非所有编程语言都可以轻松处理大数，因此您可能需要使用特殊函数执行椭圆曲线运算。

除此之外，代码并不像您最初想象的那么困难。

当然，我不建议在您最近的关键系统中使用此代码，但它应该帮助您开始创建自己的公钥并在比特币中签署自己的交易。

玩得开心。

## 链接

### 参考资料

* [NIST.FIPS.186-4.pd](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf)f-由NIST官方发布的数字签名标准。包含DSA、RSA和ECDSA的大纲。
* [sec2-v2.pdf](https://www.secg.org/sec2-v2.pdf)-SECG建议用于椭圆曲线密码学的曲线列表。包含用于比特币中使用的secp256k1曲线的参数。
  
### 可视化

* [椭圆曲线绘图仪](https://kebekus.gitlab.io/ellipticcurve/)-一个小而酷的程序，可让您玩弄简单的椭圆曲线操作。这就是我用来帮助创建本页上的图表的工具。
* [Sage Math](https://www.sagemath.org/)-一个带有椭圆曲线绘图函数的大型数学库。我用它来显示实数和有限域上的椭圆曲线图。
* [交互式椭圆曲线操作](https://andrea.corbellini.name/ecc/interactive/modk-add.html)-由Andrea Corbellini创建的一个很酷的Web工具，可让您在实数和有限域上可视化椭圆曲线加法和双倍操作。
  
### 解释

* [椭圆曲线密码学](https://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/)：温和的介绍-Andrea Corbellini的出色的四部分介绍椭圆曲线密码学。一个很好的开始。
* [介绍椭圆曲线](https://jeremykun.com/2014/02/08/introducing-elliptic-curves/)-由一个深入了解为什么在密码学中使用它们的人介绍的椭圆曲线。
* [椭圆曲线密码学简介](https://www.purplealienplanet.com/node/27)-另一种介绍ECC的方法。比上面两个指南短，但我发现它很有帮助。
* [ECDSA算法如何工作](https://kakaroto.ca/2012/01/how-the-ecdsa-algorithm-works/)-特别是ECDSA的简明解释。
* 
### 代码

以下是我发现有用的不同编程语言中的ECDSA实现：

* Python：https://github.com/wobine/blackboard101/blob/master/EllipticCurvesPart5-TheMagic-SigningAndVerifying.py

* Python：https://github.com/andreacorbellini/ecc/blob/master/scripts/ecdsa.py
* Ruby：https://github.com/DavidEGrayson/ruby_ecdsa
* PHP：https://github.com/BitcoinPHP/BitcoinECDSA.php

### 其他

* [从Secp256k1签名中恢复私钥](https://crypto.stackexchange.com/questions/57846/recovering-private-key-from-secp256k1-signatures)-Thomas Pornin提供的简洁的数学解释，介绍了如何在某人多次使用相同的随机点进行签名时恢复私钥。
* [比特币的签名类型](https://raghavsood.com/blog/2018/06/10/bitcoin-signature-types-sighash)-SIGHASH-清晰地解释了比特币中可用的不同签名散列类型。
* [bitcoin.it/wiki/OP_CHECKSIG](https://en.bitcoin.it/wiki/OP_CHECKSIG)-如何准备交易数据以在使用不同SIGHASH类型时进行哈希。
* [https：//plan99.net/~mike/satoshi-emails/thread3.html](https://plan99.net/~mike/satoshi-emails/thread3.html)-Satoshi给Mike Hearn的电子邮件，解释了他/她选择secp256k1的原因。


>我必须承认，这个项目在发布之前经过了2年的开发，我只能在许多问题上花费有限的时间。我找到了关于SHA和RSA推荐大小的指导，但没有关于相对较新的ECDSA的任何东西。我将RSA的推荐密钥大小转换为ECDSA的等效密钥大小，但然后增加了整个应用程序，以使其具有256位安全性。我没有找到任何推荐曲线类型，所以我只是…选择了一个。希望有足够的密钥大小弥补任何不足。-中本聪


[^1]:https://crypto.stackexchange.com/a/60421/56860
[^2]:https://b10c.me/blog/006-evolution-of-the-bitcoin-signature-length/
[^3]:https://bitcoin.stackexchange.com/questions/48108/why-are-sighash-flags-signed-as-4-bytes-when-only-1-byte-is-included-in-the-tran