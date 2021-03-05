---
layout:	post
title: "均摊分析"
subtitle: ""
date: 2021-03-02 20:00:00
author: "Yinwhe"
header-style: text
tags:
    - Data Structure
---

<details>
  <summary>Source Web</summary>
  <ul>
    <li><a href="https://www.cnblogs.com/alantu2018/p/9208720.html">AlanTu</a></li>
    <li><a href="https://www.dazhuanlan.com/2019/12/16/5df701614a2c9/">Dazhuanlan</a></li>
  </ul>
</details>


# Definition

对于一个操作的序列来讲，**`平摊分析`**得出的是在特定问题中这个序列下每个操作的**`平摊开销`**。

平摊分析可以用来证明在一系列操作中，通过对所有操作求平均之后，即使其中单一的操作具有较大的代价，平均代价还是小的。

综合来说有**三种分析方法** - 聚集分析、记账方法、势能方法

## 聚集分析(Aggregate Analysis)

`聚类分析`：证明对所有的n，由n个操作所构成的序列的总时间在最坏情况下为$T(n)$，每一个操作的平均成本为$\frac{T(n)}{n}$

以栈操作为例，有三个操作 `push(O(1)) pop(O(1)) multipop(O(N))`，如果只是单纯的看每个操作最坏情况为O(N)，那么N个操作的复杂度就是$O(N^2)$，显然有问题。

假定有一个`multipop(n)` 的操作，那么之前一定有至少 `n` 个 `push` 操作，总的操作复杂度其实应该是 O(2N)，那么均摊复杂度就是$\frac{O(2N)}{N+1}=O(1)$。所以三个操作的均摊成本都是$O(1)$

- “起始情况是空栈”是一个十分重要的前提！没有这个前提则分析结果不成立。

## 记账方法(Accounting Method)

`记账方法`：基本思路为，对于每一个有实际成本操作OP而言，摊还成本$\hat{C_{OP}}$被分配使得对于n个操作的任意序列，有$T(n)=\sum_{i=1}^nC_i\le\sum_{i=1}^n\hat{C_i}$；如果$\hat{C}>C$，那么额外的部分就可以被存储为预付的存款`credit`，这笔存款可以在之后对于的操作时被用。

- 这样的要求实质上是使得存款不会为负。

- 定义`实际成本`$C_i$是某一个操作真正的时间开销；

- 定义`摊还成本`$\hat{C_i}$是为一个操作赋予的“开支”。

仍然以上述的栈为例：实际代价 push(1), pop(1), multipop( min(k, stack) )

然后可以这样赋予摊还代价：push(2), pop(0), multipop(0)

那么当push的时候，实际上会存入1个credit，而pop会消耗一个，multipop会消耗k个，对于n个操作，其最多的均摊代价为2n。

**关于摊还成本$\hat{C_i}$的选取?**

摊还成本的选取不是随意的，需要满足：
操作序列的总摊还代价给出了序列总真实代价的上界，即数据结构所关联的信用必须一直非负
$\sum_{i=1}^nC_i\le\sum_{i=1}^n\hat{C_i}$，否则证明就是无效的。

- 在满足上述条件的$\hat{C_i}$下，$\sum_{i=1}^n\hat{C_i}$其实就是`n`个操作的上界。



## 势能方法(Potential Method)

`势能方法`：在平摊分析中，势能方法不是将已预付的工作作为存在数据结构特定对象中存款来表示，而是将存款总体上表示成一种“势能”或“势”，它在需要时可以释放出来，以支付后面的操作。**势是与整个数据结构而不是其中的个别对象发生联系的。**

关于势，需要自己定义一个势能函数作为衡量，其赋值给一个状态而不是一个操作。

定义**`势能函数`**为 $\Phi(S)$，S是状态集合，那么有$\Phi(S_i)-\Phi(S_{i-1})=\hat{C_i}-C_i$

**`均摊成本`**设置为$\hat{C_i}=C_i+\Phi(S_i)-\Phi(S_{i-1})$

进而得到 $\sum_{i=1}^n\hat{C_i}=\sum_{i=1}^nC_i+\Phi(S_n)-\Phi(S_0)$



还是以刚刚的栈为例：

定义势函数为栈内元素的个数

- 对于push操作：$\hat{C_i} = C_i+1=2$
- 对于pop操作：$\hat{C_i}=C_i-1=0$
- 对于multipop操作：$\hat{C_i}=C_i-k=0$

即每个操作的摊还代价都是 $O(1)$，因此n个操作的摊还代价就是 $O(n)$




换`Spray Tree`为例：

定义 $\Phi(S) = \sum logS(i)$，其中 `S(i)` 表示以`i`为节点的子树的总结点数。

![Amortized%20Analysis%20bddc5911c8514f62b1cc8886eb754791/Untitled.png](https://tva1.sinaimg.cn/large/e6c9d24ely1go5sklad1pj20zk0q3tfe.jpg)

Essential Explanation - 这里的`R`表示的是`log(Num(X))` ，`Num(X)`是节点`X`为根节点的子树总节点数；最后有 R(T) = log(N)，`0≤R(X)≤R(T)`