---
layout:	post
title: "Leftist&Skew Tree"
subtitle: ""
date: 2021-03-27 12:00:00
author: "Yinwhe"
header-style: text
tags:
    - Data Structure
---

# Leftist Tree

## Definition

定义 Null Path Length， 即 `NPL(x)` 为X到一个没有双孩子节点的**最短**距离，NPL(Null) = -1

- $Npl(X)=min\{ Npl(C)+1\ for\ all\ C\ as\ children\ of\ X \}$

定义`Leftist Heaps`为，对于任何一个节点，其**左子节点**的`Npl`不能小于**右子节点**的`Npl`

![Leftist%20Heaps%20&%20Skew%20Heaps%2072d252863c8c44f79a986f9cdf8df5e2/Untitled.png](https://tva1.sinaimg.cn/large/008eGmZEly1goybqa917hj30m207y0t5.jpg)

**[Theorem]** - A leftist tree with $r$ nodes on the right path must have at least $2^r-1$ nodes.

- 包含N个节点的Leftist Tree的右路径最多包含$\lfloor log(N+1)\rfloor$ 个节点

## Merge

> Insert 可以认为是一种特殊的 Merge

每一个 Merge 分为三步（MinHeap）：

1. 比较两个根节点大小
2. 将大节点接到小节点的右边
3. 如果小节点的右支不为空，则以其右子树继续以上操作

![Leftist%20Heaps%20&%20Skew%20Heaps%2072d252863c8c44f79a986f9cdf8df5e2/Untitled%201.png](https://tva1.sinaimg.cn/large/008eGmZEly1goybqb0fr1j30zk0rgdmx.jpg)

Code如下：

```cpp
PriorityQueue  Merge ( PriorityQueue H1, PriorityQueue H2 )
{ 
	if ( H1 == NULL )   return H2;	
	if ( H2 == NULL )   return H1;	
	if ( H1->Element < H2->Element )  return Merge1( H1, H2 );
	else return Merge1( H2, H1 );
}

static PriorityQueue
Merge1( PriorityQueue H1, PriorityQueue H2 )
{ 
	if ( H1->Left == NULL ) 	/* single node */
		H1->Left = H2;	        /* H1->Right is already NULL 
				                     and H1->Npl is already 0 */
	else {
		H1->Right = Merge( H1->Right, H2 );     /* Step 1 & 2 */
		if ( H1->Left->Npl < H1->Right->Npl )
			SwapChildren( H1 );                 	/* Step 3 */
		H1->Npl = H1->Right->Npl + 1;
	}
	return H1;
}
```

对于删除，只需要在删除root后，Merge两个子树即可

$T=O(log N)$

# Skew Tree

> A simple version of leftist tree

对任何连续M个操作，最多花费O(MlogN)的时间，其维护的方式是：

`Merge` - Always swap the left and right children except that the largest of all the nodes on the right paths does not have its children swapped.

Skew Tree 的优点是不需要额外的空间来维护路径长度，也不需要测试来确定何时交换子对象。

## Amortized Analysis

定义 $\phi(D_i)=number\ of\ heavy\ nodes$

- `Heavy Node` 的右子树节点个数至少是该node所有节点个数的一半；否则成为`Light Node`
- 那么只有处在右支path的上的Node会在Merge时发生Heavy\Light的转换
- 且在右支上的Heavy Node一定会转换成Light Node

现假定有两棵树 $H_1=l_1+h_1,H2=l_2+h_2$，其中 `h,l` 分别为**右支path**上的Heavy和Light节点数

- 则对于一次操作的$C_i$有 $C_i\le T_{worst}=l_1+h_1+l_2+h_2$
- 操作前有 $\phi=h_1+h_2+h$，操作后有 $\phi '\le l_1+l_2+h$
- 那么 $\hat{C}=C+\phi'-\phi\le T_{worst}+\phi'-\phi=2(l_1+l_2)$

`[Lemma]` A tree with K light nodes on its right path has at least $2^k-1$ nodes

- 那么有$2(l_1+l_2)=O(log N)$