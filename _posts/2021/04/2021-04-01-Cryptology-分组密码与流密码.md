---
layout:	post
title: "加密模式"
subtitle: ""
date: 2021-04-01 18:00:00
author: "Yinwhe"
header-style: text
tags:
    - Cryptology
---

# 分组密码工作模式

## 电子密码簿ECB

使用块密码的自然方式是将一长段明文分成适当大小的**明文块**，并使用加密函数`EK()`分别处理每个块。这就是所谓的电子码本(ECB)操作模式。

明文P被分成更小的块`P = [P1, P2, ..., PL]`，密文为`C = [C1, C2, ..., CL]`，其中 `Cj = Ek(Pj)`

![%E5%88%86%E7%BB%84%E5%AF%86%E7%A0%81%E4%B8%8E%E6%B5%81%E5%AF%86%E7%A0%81%20a0b74cd9c9614b88a929b0fba2ddeeec/Picture1.png](https://tva1.sinaimg.cn/large/008eGmZEly1gp4djsrb3aj30ky0ga40t.jpg)

`Advantages` - 加密和解密过程均可以**并行处理**。

`Disadvantages` - 相同内容的明文段加密后得到的密文块是相同的。

## 密文块链接模式CBC

![%E5%88%86%E7%BB%84%E5%AF%86%E7%A0%81%E4%B8%8E%E6%B5%81%E5%AF%86%E7%A0%81%20a0b74cd9c9614b88a929b0fba2ddeeec/Picture2.png](https://tva1.sinaimg.cn/large/008eGmZEly1gp4djw0apxj30ll0gfdil.jpg)

加密过程 - $C_j = E_k(P_j \bigoplus C_{j-1})$

解密过程 - $P_j = D_k(C_j)\bigoplus C_{j-1}$

CBC模式的特点是：当前块的密文与前一块的密文有关; 加密过程只能串行处理; 解密过程可以并行处理;

## 密文反馈模式CFB

![%E5%88%86%E7%BB%84%E5%AF%86%E7%A0%81%E4%B8%8E%E6%B5%81%E5%AF%86%E7%A0%81%20a0b74cd9c9614b88a929b0fba2ddeeec/Picture1%201.png](https://tva1.sinaimg.cn/large/008eGmZEly1gp4djyw0duj30ls0gkdim.jpg)

明文被切分为 `8bits` 的部分 $P = [P_1,P_2,...,P_n]$，其中$P_j$都是8bits的；然后是一个`64bits`的种子数$X_1$，加密这样进行：

$C_j=P_j\bigoplus L_8(E_k(X_j))$

$X_{j+1}=R_{56}(X_j)\|\|C_j$

其中$L_8(X)$表示X的最左边8位，$R_{56}(X)$表示X的右边56位，$X\|\|Y$表示将Y接到X后面

解密这样进行：

$P_j=C_j\bigoplus L_8(E_k(X_j))$

$X_{j+1}=R_{56}(X_j)\|\|C_j$

**整体的加密的流程**
首先我们有一个`64bits`的$X_1$，用$E_k$对其进行加密后，提取左8bit与$P_1$进行XOR运算，从而得到$C_1$；然后将$X_1$的右`56bits`接上$C_1$形成新的$X_1$，在进行上述的运算；如此往复最终得到$C$

密文反馈模式的**优点**是可以**从密文传输的错误中恢复：**

- 比如要传输密文C1, C2, C3, ..., CK，现假定C1传输错误，把它记作C1'，则解密还原得到的P1有错。
- 若 X1 记作(*, *, *, *, *, *, *, *)
- 则 X2 = (*, *, *, *, *, *, *, C1')
- X3 = (*, *, *, *, *, *, C1', C2)
- X4 = (*, *, *, *, *, C1', C2, C3)
- X9 = (C1', C2, C3, C4, C5, C6, C7, C8)
- X10 = (C2, C3, C4, C5, C6, C7, C8, C9)
- 使用密钥X1,X2,X3,...,X9解密C1'C2C3C4C5C6C7C8C9还原得到的P1,P2,P3,...,P9全部有错，但是从P10开始的解密还原是正确的。