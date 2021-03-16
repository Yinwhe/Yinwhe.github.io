---
layout:	post
title: "DBS-Relation"
subtitle: ""
date: 2021-03-13 22:00:00
author: "Yinwhe"
header-style: text
tags:
    - Database
---

# Structure of Relational Database

> 关系数据库由 `表(table)` 的集合构成，每个表有唯一的名字

在关系模型的术语中，关系 `relation` 用来指代表，而元组 tuple 用来指代行；类似地，属性(`attribute`) 指代的是表中的列。

- 一个`Relation`就是一个`n元元组(n-tuples)`；在数学术语中，元组只是一组值的序列（或列表）。
- 对于关系的每个属性，都存在一个允许取值的集合，称为该属性的域(`domain`)
- 关系每个属性`Attribute`都应该有名字，且应该在允许的域`Domain`中取值。
- 属性的值必须是`Atomic`的，这是关系理论的第一范式。
    - Multivalued attribute不是原子的
    - Composite attribute也不是原子的
- 特殊值`null`是每个域的成员

## Concept About Relation

`Relation`有两个重要的概念： `Relation Schema`和`Relation Instance`

- `Schema`描述的是Relation的结构，如： `Student-schema = (sid, name, sex, age, dept)`
- `Instance`则是一个实际存在的数据快照
- **Properties of Relation**
    1. The **order** of tuples is **irrelevant** (tuples may be stored in arbitrary).
    2. **No duplicated** tuples in a relation.
    3. Attribute values are **atomic**.

## Key

**`超码 (superkey)`** - 是一个或多个属性的集合，这些属性的组合可以在一个关系中**唯一**地标识一个元组。

- 但超码不是唯一的，标识同一个元组可以有多个超码
- 超码的超集都是超码

**`候选码 (Candidate Key)`** - 超码中可能包含无关紧要的属性；我们通常只对这样的一些超码感兴趣，它们的任意真子集都不能成为超码。这样的最小超码称为**候选码**

**`主码 (Primary Key)`** - 指被**人为选中的候选码**

**`外码 (Foreign Key)`** - 一个关系模式(如$r_1$)可能在它的属性中包括另一个关系模式(如$r_2$)的主码。这个属性在$r_1$上称作参照$r_2$的**外码**。

- 关系$r_1$也称为外码依赖的**参照关系** (referencing relation)
- $r_2$叫做外码的**被参照关系** (referenced relation)

# Fundamental Relation-Algebra Operations

## `Select`

写为$\sigma _p(r)$，$p$被成为`选择谓语(Selection Predicate)`，定义为 $\sigma _p(r)=\{t\mid t\in r\ and\ p(t) \}$

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goim1hyog7j30iy0ga755.jpg" alt="02-Relation%20Model%2097277abeb4be47f395f9ae597d342a63/Untitled.png" style="zoom: 50%;" />

## `Project`

即投射，写为 $\prod_{A1,A2,...,Ak}(r)$，其中$A1,A2,...,Ak$是属性名称，表示投射出所选择的属性作为表

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goim1jde41j30m00kydgz.jpg" alt="02-Relation%20Model%2097277abeb4be47f395f9ae597d342a63/Untitled%201.png" style="zoom:45%;" />

## `Union`

写为 $r\bigcup s$，定义为 $r\bigcup s=\{ t\mid t\in r\ or\ t\in s \}$

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goim26vqzhj30oi0k00tm.jpg" alt="02-Relation%20Model%2097277abeb4be47f395f9ae597d342a63/Untitled%202.png" style="zoom: 50%;" />

## `Difference`

写为 $r-s$，定义为 $r-s=\{t\mid\ t\in r\ and\ t\notin s  \}$

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goim2d79h8j30oi0h8t9f.jpg" alt="02-Relation%20Model%2097277abeb4be47f395f9ae597d342a63/Untitled%203.png" style="zoom:50%;" />

## `Cartesian Product`

写为 $r\times s$，定义为 $r\times s=\{ \{ tq\}\mid t\in r\ and\ q\in s \}$

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goim2zajm8j30r20lamz5.jpg" alt="02-Relation%20Model%2097277abeb4be47f395f9ae597d342a63/Untitled%204.png" style="zoom:40%;" />

## `Rename`

写为 $\rho_x (E)$，即将表$E$重命名为$x$；如果写成 $\rho_{x(A1,A2,...,Ak)}(E)$，即对$E$及其Attributes都重命名

# Additional Relation-Algebra Operations

## `Set-Intersection`

写为 $r\bigcap s$，定义为 $r\bigcap s=\{ t\mid t\in r\ and\ t\in s \}$

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goim39nv13j30na0f6t9b.jpg" alt="02-Relation%20Model%2097277abeb4be47f395f9ae597d342a63/Untitled%205.png" style="zoom:40%;" />

- Note -  $r\bigcap s = r-(r-s)$

## `Natural Joint`

## `Division`

写为$r\div s$，定义比较复杂，省略吧

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goim3iju3vj30su0ia0ts.jpg" alt="02-Relation%20Model%2097277abeb4be47f395f9ae597d342a63/Untitled%206.png" style="zoom:40%;" />

## `Assignment`

即赋值符$\leftarrow$，将右值赋给左值，例如 $r\div s$ 可以写为：

$temp1\ \leftarrow\ \prod_{R-S}(r)$

$temp2\ \leftarrow\ \prod_{R-S}((temp1\times s)-\prod_{R-S,\ S}(r))$

$result=temp1-temp2$

- **`Assignment`必须赋值给临时变量**

# Extended Relational-Algebra Operations

## `Generalized Projection`

写为$\prod_{F1,F2,...,Fn}(E)$，其允许在F中参入包含常数或者属性的**算术表达式**

如给定一个relation，`credit_info(customer-name, limit, credit_balance)`，找出用户能继续借的款项： $\prod_{customer-name,limit-credit\_balance}(credit\_info)$

## `Aggregate`

`Aggregate Function`有如下几种：

- avg: average value
- min: minimum value
- max: maximum value
- sum: sum of values
- count: number of values

Aggregate Operation写为 $\_{G1,G2,...,Gn}g\_{F1(A1),F2(A2),...,Fn(An)}(E)$，其中 $\_{G1,G2,...,Gn}$是一列属性，$Fi$是`Aggregate Function`，而$Ai$则是参与运算的属性：

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goim3x5hp0j30x40hs0uw.jpg" alt="02-Relation%20Model%2097277abeb4be47f395f9ae597d342a63/Untitled%207.png" style="zoom:50%;" />

- 需要注意的是，Aggregation返回的结果是没有名称的，可以通过`as`来命名：$\_{branch-name}g\_{sum(balance)\ as\ Sum-balance}(Account)$

# Modification of the Database

## `Deletion`

可以写为 $r\leftarrow r-E$，如删除所有在`Perryridge`银行的账户信息可写为：$account\leftarrow account-\sigma_{branch-name="Perryridge"}(account)$

## `Insertion`

可写为$r\leftarrow r\bigcup E$，如$account\leftarrow account\bigcup\{ ('Perryridge', A-973,1200) \}$就插入了一个Item

## `Update`

可写为$r\leftarrow \prod_{F1,F2,...,Fn}(r)$，这样就不会改变其他属性的值，如给每个账户发放5%的利息可写为：$account\leftarrow \prod_{account-number,branch-name,balance*1.05}(account)$