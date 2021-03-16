---
layout:	post
title: "密码数学基础"
subtitle: "Some basic theory"
date: 2021-03-04 20:00:00
author: "Yinwhe"
header-style: text
tags:
    - Cryptology
---



# 欧拉定理

若 `n, a` 为正整数且互素，那么有 $\alpha^{\phi(n)}\ =\ 1\ (mod\ n)$
- 其中 $\phi(n)$ 表示与`n`互素而比`n`小的自然数

## 证明

假定 `[1, n)` 中，与`n`互素的数分别为 $x_1,x_2,x_3,...,x_{\phi(n)}$
再令 $m_1=ax_1,m2=ax_2,...,m_{\phi(n)}=ax_{\phi(n)}$


那么可以证明的是：
1. $\forall (i,j)\in[1,n)\ => m_i\ne m_j\ (mod\ n)$
2. $\forall\ i\in[1,n)\ if\ m_i\equiv r_i\ (mod\ n)\ then\ (r_i,n)=1$

进而序列 $r_i$ 必然是序列 $x_i$ 的某个排序，那么就有


$$
\prod_{i=1}^nm_i\equiv \prod_{i=1}^nx_i\ (mod\ n)\  即\ a^{\phi(n)}x_1x_2...x_{\phi(n)}\equiv x_1x_2...x_{\phi(n)}\ (mod\ n)
$$


消去$x_i$序列即可得到$a^{\phi(n)}\equiv 1\ (mod\ n)$


**证明 `1`**


$$
if\ m_i\equiv m_j\ (mod\ n)\ \ then
$$

$$
ax_i\equiv ax_j\ (mod\ n) => n|a(x_i-x_j)
$$

$$
This\ is\ impossible\ for\  (a,n)=1\ and\ x_i-x_j<n
$$





**证明 `2`**


$$
m_i\equiv r_i\ (mod\ n)\ then
$$

$$
m_i=ax_i=kn+r_i,\ if\ (r_i,n)\ne1\ then
$$

$$
r_i=cl,n=dl,\ which\ means\ (r_i,n)=l>1
$$

$$
then\ l|ax_i
$$

$$
Which\ is\ also\ impossibe,\ (a,n)=(x_i,n)=1\ne l
$$

# 中国剩余定理

中国剩余定理给出了以下的一元线性同余方程组：

![image-20210304201647411](https://tva1.sinaimg.cn/large/008eGmZEly1go84yoxbqpj306f034dfu.jpg)

有解的判定条件，并用**构造法**给出了在有解情况下解的具体形式。

中国剩余定理说明：假设整数 $m_1,m_2,...,m_n$ 两两互素，则对任意的整数 $a_1,a_2,...,a_n$，方程组(S)有解，通解可以用如下的方式构造得到：


$$
Let\ M=\prod_{i=1}^nm_i,\ M_i=M/m_i
$$

$$
Let\ t_i=M_i^{-1}\ which\ means\ t_iM_i\equiv1\ (mod\ m_i)
$$

$$
Then\ x=\sum_{i=1}^na_it_iM_i+kM,\ k\in Z
$$



`x` 由构造给出，故这里直接验证是否为解即可，不再证明。

# Bezout定理

设`a, b`为整数，且`a, b`中至少有一个不等于`0`，令`(a,b)=d`，则一定存在整数`x, y`使得下式成立: `ax + by = d`

## 证明

$$
(a,b)=d=>d\mid(ax+by)
$$

$$
Let\ s\ be\ the\ smallest\ positive\ result\ of\ (ax+by),\ and\ ax_0+by_0=s
$$

$$
Let\ r\equiv a\ (mod\ s)=>\ a=qs+r,\ then
$$

$$
r=a-qs=a-q(ax_0+by_0)=(1-qx_0)a-qy_0b=ax_1+by_1,\ 0\le r\lt s
$$

$$
\because s\ is\ the\ smallest\ positive\ result\ of\ (ax+by)
$$

$$
\therefore r=0
$$

$$
\therefore s\mid a
$$

$$
For\ same\ reason\ s\mid b
$$

$$
\therefore s\mid d =>d\ge s
$$

$$
But\ s=ax_0+by_0=>d\mid s
$$

$$
We\ have\ s=d,\ then\ d=ax_0+by_0
$$