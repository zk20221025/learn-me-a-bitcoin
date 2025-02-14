# <center>十六进制</center>
<center>一组包括字母a到f的数字。</center>

十六进制值使用字母A-F以及数字0-9来表示数字。  
为了说明：  
|十进制数|十六进制|
|---|---|
|0	|0|
|1	|1|
|2	|2|
|3	|3|
|4	|4|
|5	|5|
|6	|6|
|7	|7|
|8	|8|
|9	|9|
|10	|A|
|11	|B|
|12	|C|
|13	|D|
|14	|E|
|15	|F|

然后在达到数字“F”之后，你可以继续使用两位数字，如下所示：
|十进制数|十六进制|
|---|---|
|16	|10|
|17	|11|
|18	|12|
|19	|13|
|20	|14|
|21	|15|
|22	|16|
|23	|17|
|24	|18|
|25	|19|
|26	|1A|
|27	|1B|
|28	|1C|
|29	|1D|

你可以自己想出剩下的部分。

这些额外的字母意味着你可以用十六进制以更少的字符表示比十进制更大的数字。  
例子：
```
10         = a
100        = 64
1000       = 3e8
10000      = 2710
100000     = 186a0
1000000    = f4240
10000000   = 989680
100000000  = 5f5e100
1000000000 = 3b9aca00
```
>不要被字母代表的数字概念吓到……它们仍然是数字。如果你需要用一个字符来表示10到15的数字，你可能就需要使用字母了。

>Hexa = 6，Deci = 10。因此，_hexadeci_mal指的是这个数字系统中有16个不同的字符。

>当十六进制数用于编程时，它们通常带有前缀“0x”（这是标准的C++表示法，而比特币是用C++编写的，所以就这样）。因此，如果你在某些代码中看到像0x1000这样的数字，它意味着它是十六进制。你可以将其转换为我们常用的十进制来理解它的值。

## 转换
这是如何在脑海中计算十六进制的方法：

![hexadecimal-1.png](img/hexadecimal-1%20(1).gif)

如果想要得到正确的答案，可以使用这些工具：

* [十进制->十六进制](https://learnmeabitcoin.com/tools/dechex)
* [十六进制->十进制](https://learnmeabitcoin.com/tools/hexdec)

有时我喜欢使用命令行：
```
# decimal to hexadecimal
echo "obase=16; 15" | bc

# hexadecimal to decimal
echo "ibase=16; F" | bc
```

## 资源
你可以阅读关于十六进制数的知识，但只有通过实际去使用它，并且在使用过程中犯错误，然后纠正错误，你才能真正地理解它。

这里有一些好的学习资源。

* [十六进制数字简要解释](http://seggleston.com/1/web-development/hex-numbers)
* [二进制、十进制和十六进制数字的实用指南](http://www.myhome.org/pg/numbers.html)
