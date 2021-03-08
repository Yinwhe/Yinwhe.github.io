---
layout:	post
title: "DBS-Overview"
subtitle: "Introduction"
date: 2021-03-07 12:00:00
author: "Yinwhe"
header-style: text
tags:
    - Database
---



# Definition

Database
- A collection of interrelated data, relevant to an enterprise.
- A large collection of integrated and persistent data (DB) [R. Ramakrishnan, J. Gehrhe].
- A collection of information that exists over a long period of time, often many years [Ullman].
- 长期存储在计算机内、有组织的、可共享的数据集合[萨师煊，王珊].

Database Management System **`DBMS` =** Database + A set of programs used to access, update and manage the data in database.

<details>
  <summary>数据库管理系统的特性：</summary>
	<ul>
    <li>数据访问的<b>高效Efficiency</b>和<b>可扩展性scalability</b></li>
    <li>缩短应用开发时间</li>
    <li>数据<b>独立性independence</b>（物理/逻辑的独立性）</li>
    <li>数据<b>完整性integrity</b>和安全性</li>
    <li><b>并发Concurrent</b>访问和鲁棒性</li>
  </ul>
</details>

**Access Method**

通过交互式的工具进行访问，工具一般由 **`DBMS(Database Management System)`** 提供

通过开发工具，调用ODBC/JDBC进行访问(VC++, PB, Delphi, ASP, JSP, PHP, etc)

<details>
  <summary>文件处理系统FPS和DBMS的主要区别</summary>
  <ol>
    <li>这两个系统都包含一个数据集合和一组访问该数据的程序。数据库管理系统协调对数据的物理和逻辑访问，而文件处理系统仅协调物理访问。</li>
    <li>数据库管理系统通过确保授权给所有程序访问的所有程序的物理数据块来减少数据重复的数量，而文件处理系统中一个程序编写的数据可能无法被另一个程序读取。</li>
    <li>数据库管理系统旨在允许对数据的灵活访问（即查询），而文件处理系统旨在允许对数据的预定访问（即已编译程序）。</li>
    <li>数据库管理系统旨在协调多个用户同时访问相同数据。文件处理系统通常设计为允许一个或多个程序同时访问不同的数据文件。在文件处理系统中，只有两个程序都具有对文件的只读访问权限，两个程序才能同时访问该文</li>
  </ol>
</details>




# View of Data

一些`abstraction`

- Physical Level：最低层次的抽象，描述数据实际上是怎样存储的
- Logical Level：比物理层层次稍高的抽象，描述数据库中存储什么数据及这些数据间存在什么关系。
- View Level：最高层次的抽象，只描述整个数据库的某个部分，应用程序隐藏数据类型的细节；出于安全目的，视图也可以隐藏信息

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gob6stv58aj30t80h4myl.jpg" alt="Overview%20ab5400a1837e4a1e8134c1c2f89fa3e2/Untitled.png" style="zoom:33%;" />

## Schemas and Instances

特定时刻存储在数据库中的信息的集合称作数据库的一个**`实例 (instance)`** 。而数据库的总体设计称作数据库**`模式 (schema)`** ；数据库模式即使发生变化，也不频繁。

Schema和Instance 的关系可以类似于编程语言的数据类型和实际变量，类型规定了存储方式，变量则是实际的数据。

**`Schemas`的一些模式**

1. **`物理模式(physical schema)`** 在物理层描述数据库的设计
2. **`逻辑模式 (logical schema)`** 在逻辑层描述数据库的设计
3. **`子模式 (subschema)`** 描述了数据库的不同视图。

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gob6t0dfgdj30yg0n60v2.jpg" alt="Overview%20ab5400a1837e4a1e8134c1c2f89fa3e2/Untitled%201.png" style="zoom:33%;" />

## Independence

更改当前`Schemas`而不会影响更高层设计的能力，包括 **Physical Independence** 和 **Logical Independence** 

`Physical Independence` - 更改物理层Schemas而不会迫使逻辑Schemas改变

- App是基于Logical的，其并不关心数据的组织架构，故不会受到影响
- **这是使用`DBMS`的一大重要的原因**

`Logical Independence` - 更改逻辑schemas而不会迫使App改变

- 这一点很难达到，因为App往往基于logical schema实现

## Data Model

> 数据库结构的基础是**`数据模型(data model)`** 。

**`数据模型`**是一个描述**数据**、**数据联系**、**数据语义**以及**一致性约束**的概念工具的集合。

- data structure
- data relationships
- data semantics
- data constraints

数据模型可以被划分为**`4类`**

- **`关系模型(relational model)`**  - 关系模型用表的集合来表示数据和数据间的联系。
- **`实体－联系模型(entity-relationship model)` -** **ER模型**很重要！
- **`基千对象的数据模型(object-based data model)` -** 面向对象的数据模型可以看成是 E-R 模型增加了封装、方法（和对象标识等概念后的扩展。
- **`半结构化数据模型 (semistructured data model)`** - 半结构化数据模型允许那些相同类型的数据项含有不同的属性集的数据定义。
- **`网状数据模型(network data model)`**和**`层次数据模型 (hierarchical data model)`** - 先于关系数据模型出现。这些模型和底层的实现联系很紧密，并且使数据建模复杂化。

# Database Language

三种主要类型：

- **`DDL` -** Data Definition Language
- **`DML` -** Data Manipulation Language
- **`DCL` -** Data Control Language

## DDL

DDL定义一个**数据库模式(Scheme)** ，也指定数据库的**存储结构、访问方式**和**一致性限制**。

DDL 的**输出**放在**`数据字典 (data dictionary)`**中，数据字典包含了元数据(metadata) ，元数据是关于数据的数据。

- 可把数据字典看作一种特殊的表，这种表只能由数据库系统本身（不是常规的用户）来访问和修改。在读取和修改实际的数据前，数据库系统先要参考数据字典。

## DML

DML使得用户可以**访问**或**操纵**数据

- Retrieve data from the database
- Insert / delete / update data in the database

通常有**`两类`**基本的数据操纵语言

- **`过程化DML(Procedural DML)` -** 要求用户指定需要什么数据以及如何获得这些数据。(e.g., C, Pascal, Java, etc.).
- **`非过程化DML(Nonprocedural DML)`** - 只要求用户指定需要什么数据，而不指明如何获得这些数据。(e.g., SQL, Prolog, etc.).

**`SQL(Structured Query Language )`** 是当前使用最广泛的**非过程化**`query`语句

- SQL = DDL + DML + DCL

# Database Design

1. 需求分析 - 需要什么样的数据、操作等
2. **概念数据库设计** – 使用**`E-R模型`**或类似的高级数据模型对数据和约束进行描述。
3. 逻辑数据库设计– 将概念设计转换为DB的某个schema
4. 模型优化 – 关系标准化，检查冗余和相关的异常关系结构
5. 物理数据库设计 – 索引、查询、集群和数据库调优。
6. 创建并初始化数据库&安全设计 – 加载初始数据进行测试、识别不同的用户组及他们的角色

## Entity-Relationship (E-R) Model

实体－联系 (E-R) 数据模型使用一组称作**`实体`**的基本对象，以及这些对象间的**`联系`**。

- **`实体`**是现实世界中可区别于其他对象的一件”事情”或一个“物体”；通过**`属性(attribute)`**集合来描述
- **`联系`**是几个实体之间的关联

同一类型的所有实体的集合称作**`实体集 (entity set)`** ，同一类型的所有联系的集合称作**`联系集(relationship set)`**

# User & Administrator

**`Naive users`** – invoke one of the permanent application programs that have been written previously by a high level language.

- E.g. people accessing database over the web, bank tellers, clerical staff.

**`Application programmers`** – interact with system via SQL calls.

**`Sophisticated users`** – form requests in a database query language.

- E.g. Online Analytical Processing (OLAP), Data mining.

**`Specialized users`** – write specialized database applications that do not fit into the traditional data processing framework.

- E.g. CAD, Expert System (ES), KDB.

**`Database administrator (DBA)`** - A special user having **central control** over database and programs accessing those data.

- DBA has the highest privilege for the database.
- DBA coordinates all the activities of the database system.
- DBA controls all users authority to the database.
- DBA has a good understanding of the enterprise’s information resources and requirements.
- Database administrator's **duties/functions** include:
    - Schema definition
    - Storage structure and access method definition
    - Schema and physical organization modification
    - Granting of authorization for data access
    - Routing maintenance

        Monitoring performance and responding to changes in requirements;
        Security for the database (periodically backup database, recovery when failure)

# Transaction Management

**`事务 (transaction)`**是数据库应用中完成单一逻辑功能的操作集合。

四个重要的特性 **`ACID`**

- Atomicity - 事物要么发生结束，要么就不发生
- Consistence - 前后一致
- Isolation - 独立性
- Durability

# Query Process

**`Query Processor`** includes DDL interpreter, DML compiler, and query processing.

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gob6u87bb2j30qi0fwadx.jpg" alt="Overview%20ab5400a1837e4a1e8134c1c2f89fa3e2/Untitled%202.png" style="zoom:33%;" />