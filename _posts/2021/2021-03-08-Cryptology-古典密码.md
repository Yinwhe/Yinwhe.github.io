---
layout:	post
title: "古典密码"
subtitle: ""
date: 2021-03-08 22:00:00
author: "Yinwhe"
header-style: text
tags:
    - Cryptology
---

# 单表密码

即只使用一张密码字母表，且**`明文字母`**与**`密文字母`**有**`固定的对应关系`**，可使用**`频率分析法`**进行破解。

## 凯撒加密

也叫加法加密

- 加密算法: $y = (x-'A'+3) \% 26 + 'A'$
- 解密算法: $x = (y-'A'+23) \% 26 + 'A'$

## 乘法密码

加密算法: $y = x*k \% n$

解密算法: $x = y*k^{-1} \% n$，其中$k^{-1}$是$k$的乘法逆元

## 仿射密码

加密算法: $y = (x*k_1 + k_2) \% n$

解密算法: $x=(y-k_2)*k_1^{-1}\%n$，其中$k_1^{-1}$是$k_1$的乘法逆元

加法加密和乘法加密都是仿射加密的特例。

# 多表密码

多表密码是对每个明文字母采用不同的单表代换，即同一明文字母对应多个密文字母

## Playfair体制

Playfair体制的是一个`5*5`的矩阵，构造方式如下：

1. 构造字母表`{a,b,c,d,e,f,g,h,i,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z}` 的一个置换，其中`i`与`j`通用
2. 将置换后的字母表按行排列成 `5*5` 的矩阵

加密的时候，需要在明文串的适当位置插入一些特定的字母，如`q`，使得明文长度为`偶数`，并将字母按照`两个一组`进行分组，每组中`两个字母应该不同`；对字母组`m1m2`设加密后的密文为`c1c2`

1. 如果`m1`和`m2`在**同一行**，那么`c1`和`c2`分别为`m1 m2` **右侧**的字母
2. 如果`m1`和`m2`在**同一列**，那么`c1`和`c2`分别为`m1 m2` **下侧**的字母
3. 如果`m1`和`m2`在**不同的行列**，那么`c1`和`c2`分别为`m1 m2` 构成矩形的另外两个字母，`c1`和`m1`同行，`c2`和`m2`同行。

## Vigenere体制

就是多表简单加法密码，设明文$m=m_1m_2...m_n$，密钥为$k=k_1k_2...k_n$，那么密钥 $c=c_1c_2...c_n$，其中$c_i=(m_i+k_i)\ mod\ 26$

- 如果密钥长度较短时，可以周期性的重复使用。

## Beaufort体制

这个和Vigenere体制很相似，也是多表简单加密算法，设明文$m=m_1m_2...m_n$，密钥为$k=k_1k_2...k_n$，那么密钥 $c=c_1c_2...c_n$，其中$c_i=(k_i+25-m_i)\ mod\ 26$

## Vernam体制

该体制在加密前首先将明文编码为`(0,1)`字符串，设明文$m=m_1m_2...m_i...$，密钥为$k=k_1k_2...k_i...$，则密文$c=c_1c_2...c_i...$，其中$c_i=m_i\bigoplus k_i$ (模2加法，也即异或)

- Note: $a\bigoplus b \bigoplus b = a$，故有$k_i=m_i\bigoplus c_i$，如果密钥重复使用，很容易破解

## Hill体制

实际上是广义仿射密码的一个特例，主要思想是将`n`个明文字母通过线性变换转换成`n`个密文字母，解密则只需要一次逆变换即可，密钥就是变换矩阵。

设明文$m=(m_1,m_2,...,m_n)\in Z_{26}^n$，密文$c=(c_1,c_2,...,c_n)\in Z_{26}^n$，密钥为$Z_{26}$上的n\*n可逆矩阵$K=(k_{ij})_{n*n}$，那么有：

- $c=mK\ (mod\ 26)$

- $m=cK^{-1}\ (mod\ 26)$

## Enigma

几个组件介绍如下：

`Plugboard` - 即键盘下的一些连线，其起到的做用为：

如果将 `A, B` 用线连接，那么当按下`A`的时候，实际上输入的是`B`，按下`B`则实际输入的`A`；同样当输出为`B`时，实际显示的是`A`，输出为`A`时实际显示的`B`。

```cpp
char plug[27]	="BADCEFGHIJKLMNOPQRSTUVWXYZ";
							/*ABCDEFGHIJKLMNOPQRSTUVWXYZ*/
```

`Rotor` - 即密码的转轮，这个部分比较复杂且重要，其起到的实际是表的效果；Enigma有`5`个Rotor，每个Rotor上的表是固定的，解密或者解密时需要安装`3`个：

这里以装入 `I,II,III` 三个轮为例：

```
RotorI
**EKMFLGDQVZNTOWYHXUSPAIBRCJ
ABCDEFGHIJKLMNOPQRSTUVWXYZ**

RotorII
**AJDKSIRUXBLHWTMCQGZNPYFVOE
ABCDEFGHIJKLMNOPQRSTUVWXYZ**

RotorIII
**BDFHJLCPRTXVZNYEIWGAKMUSQO
ABCDEFGHIJKLMNOPQRSTUVWXYZ**

Reflector
**YRUHQSLDPXNGOKMIEBFZCWVJAT
ABCDEFGHIJKLMNOPQRSTUVWXYZ**

Rotor IV
**ESOVPZJAYQUIRHXLNFTGKDCMWB      
ABCDEFGHIJKLMNOPQRSTUVWXYZ**

Rotor V
**VZBRGITYUPSDNHLXAWMJQOFECK
ABCDEFGHIJKLMNOPQRSTUVWXYZ**
```

1. 当输入`A`的时候，这里不考虑plugboard，那么先在`Rotor I`中**从下向上**进行查表，得到`E`
2. 然后再在`Rotor II`中进行查表得到`S`，在`Rotor III`中得到`G`
3. 反射轮是一个特殊的轮，是不可更改的，在Enigma内部的，`G`经过反射轮得到`L`
4. 此时需要反过来再依次查表一边，不过这一次是**从上往下**进行对应，如在`Rotor III`中得到`F`，在`Rotor II`中得到`W`，在`Rotor I`中得到`N`，最后就输出了`N`

    这一步实际上是为了解密，按照一样的方法将`N`输入是可以得到`A`的

然后是**轮内部**和**外部密码**设置的问题，每个齿轮有一个内部设置叫`ring setting`，外部设置叫`MessageKey`

假定`Rotor I`的RingSetting=B，MessageKey=A；现在按键盘A的时候, A进入Rotor I后, 要做以下运算:

```cpp
char c = 'A';
int delta = MessageKey-RingSetting;
c = ((c-'A')+delta+26)%26 + 'A';
```

此时c='Z'，接下去拿c去查`Rotor I`的表得J；当J要出去的时候，还得减去delta，即:

```cpp
c = c-delta;
```

即c=K；所以从齿轮I出来的字母就是K；另外两个齿轮的处理一样。整体处理的流程如下：

```
In -> A + delta1 -> I :(A’ - delta1)+delta2 
   -> II: (A’’ - delta2)+delta3
   -> III:(A’’’- delta3)
   -> reflect: a
   -> a + delta3 -> III: (a’- delta3)+delta2 
   -> II: (a’’ - delta2)+delta1
   -> I:  (a’’’ -delta1) ->
Out
```

最后还有一个`Rotor`转的问题：