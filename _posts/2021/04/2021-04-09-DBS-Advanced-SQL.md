---
layout:	post
title: "DBS-Advanced-SQL"
subtitle: "Why so many operations!!!"
date: 2021-04-09 20:00:00
author: "Yinwhe"
header-style: text
tags:
    - Database
---



# Data Type and Schema

SQL 支持两种形式的用户定义数据类型。第一种称为独特类型（distinct type) ，另一种称为结构化数据类型(structured data type) 

- **Structured data types**
- **Distinct types** 使用 `create type` 来定义：

    ```sql
    Create type person_name as varchar (20)
    Create table student
                (sno char(10) primary key,
                sname person_name,
                ssex char(1),
                birthday date)
    Drop type person_name
    ```

- **Domain** - 在把用户定义类型加人到 SQL 之前， SQL 有一个相似但稍有不同的概念：域，它可以在基本类型上施加完整性约束：

    ```sql
    Create domain Dollars as numeric(12, 2) not null;
    Create domain Pounds as numeric(12,2);
    Create table employee
                (eno char(10) primary key,
                 ename varchar(15),
                 job varchar(10),
                 salary Dollars,
                 comm Pounds);
    ```

    `Domain` - Constraints, not strongly typed

    **类型**和**域**之间有两个重大的**差别**：

    1. 在域上可以声明约束，例如 not null，也可以为域类型变量定义默认值，然而在用户定义类型上不能声明约束或默认值。
    2. 域并不是强类型的。因此一个域类型的值可以被赋给另一个域类型，只要它们的基本类型是相容的。
- **Large-object Types**

    **`blob`** Binary Large Object - object是未解释的二进制数据的集合(其解释留给数据库系统之外的应用程序)

    **`clob`** Character Large Object - object是字符数据的集合

    `Query` - 当查询返回一个大对象时，返回的是一个指针而不是大对象本身，这样的操作更高效。

    ```sql
    Create table students
                (sid char(10) primary key,
                 name varchar(10),
                 gender char(1),
                 photo blob(20MB),
                 cv clob(10KB))
    ```

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gpdq6n4d29j31000lo3zz.jpg" alt="04-Advanced%20SQL%20030ef461766b4193bfac2e3735fcb19f/Untitled.png" style="zoom:50%;" />

# Integrity Constraints

完整性约束通过确保对数据库的授权更改不会导致数据一致性的丢失，防止对数据库的意外损害。

- 实体完整性、参照完整性和用户定义的完整性约束
- 完整性约束是数据库实例(Instance)必须遵循的
- 完整性约束由`DBMS`维护

## Domain Constraints

**Constraints on a single relation**

- Not null
- Primary key - 默认带有 Not null
- Unique - `unique (A1, A2, ..., An)` ，指出(A1, A2, .., An)作为一个候选码，即在关系中没有两个元组在该属性上取值相同。然而候选码属性可以为 `null`, 除非它们已被显式地声明为 `not null`
- Check (P), where P is a predicate - 通常用 `check` 子句来保证属性值满足指定的条件，实际上创建了一个强大的类型系统：

    ```sql
    CREATE table section
      (course_id varchar(8) not null,
       sec_id varchar(8) not null,
       semester varchar(8),
       year numeric(4,0),
       building varchar(8),
       primary key(course_id, sec_id, semester, year),
       check(semester in ('Fall', 'Winter', 'Spring', 'Summer'))
      );
    ```

## Referential Integrity

我们常常希望保证在一个关系中给定属性集上的取值也在另一关系的特定属性集的取值中出现，这种情况称为**参照完整性**`referential integrity`

令关系$r_1,r_2$的属性集分别为$R_1,R_2$，主码分别为$K_1,K_2$；如果要求对 $r_2$ 中任意元组 $t_2$ ，均存在 $r_1$ 中元组 $t_1$ 使得 $t_1.K_1=t_2.\alpha$，我们称$R_2$的子集$\alpha$为参照关系$r_1$中$K_1$的外码 (foreign key)。这种要求就被成为**参照完整性约束** `referential-integrity constraint`

一个简单的例子如下：

```sql
Create table account 
      (account-number char(10), 
       branch-name char(15), 
       balance integer, 
       primary key (account-number), 
       foreign key (branch-name) references branch); 

Create table depositor 
      (customer-name	char(20), 
       account-number	char(10), 
       primary key (customer-name, account-number), 
       foreign key (account-number) references account, 
       foreign key (customer-name) references customer);
```

当**违反**参照完整性约束时，通常的处理是拒绝执行导致完整性破坏的操作（即进行更新操作的事务被回滚）。但是，在 `foreign key` 子句中可以指明：如果被参照关系上的删除或更新动作违反了约束，那么系统必须采取一些步骤通过修改参照关系中的元组来恢复完整性约束，而不是拒绝这样的动作：

```sql
Create table account ( 
    ...
    foreign key (branch-name) references branch 
    [ on delete cascade] 
    [ on update cascade] 
    ...);
```

- 即对关系进行级联的删除或者更新
- SQL 还允许 `foreign key` 子句指明除 `cascade` 以外的其他动作，如果约束被违反：可将参照域置为null（用 `set null` 代替 `cascade`)；或者置为域的默认值（用 `set default`)

**Note** - Referential integrity is only checked at the end of a transaction

## Assertions

一个断言(`assertion`) 就是一个谓词，它表达了我们希望数据库总能满足的一个条件。**域约束**和**参照完整性约束**是断言的**特殊形式**。格式如下：

```sql
CREATE ASSERTION <assertion-name>
			CHECK <predicate>;
```

当创建断言时，系统要检测其**有效性**。如果断言有效，则今后只有不破坏断言的数据库修改才被允许。如果断言较复杂，则检测会带来相当大的开销。因此，使用断言应该特别小心。

```sql
--Example: The sum of all loan amounts for each branch must be less than 
--         the sum of all account balances at the branch.
CREATE ASSERTION sum-constraint CHECK 
(not exists (select * from branch B 
             where (select sum(amount) from loan L
                    where L.branch-name = B.branch-name) 
                 > (select sum(balance) from account 
                    where account.branch-name = B.branch-name)))
```

## Trigger

触发器(`trigger`) 是一条语句，当对数据库作修改时，它自动被系统执行。要设置触发器机制，必须满足两个要求：

- 指明什么条件下执行触发器。它被分解为一个引起触发器被检测的事件和一个触发器执行必须满足的条件。
- 指明触发器执行时的动作。

触发器可以用来实现未被 SQL 约束机制指定的某些完整性约束。

- An example

    ```sql
    CREATE TRIGGER overdraft_trigger after update on account 
      referencing new row as nrow
      for each row 
      when nrow.balance < 0
      begin atomic 
        insert into borrower 
          (select customer-name, account-number from depositor 
           where nrow.account-number = depositor.account-number) 
        insert into loan values 
          (nrow.account-number, nrow.branch-name, – nrow.balance) 
        update account set balance = 0 
          where account.account-number = nrow.account-number 
      end
    ```

- 在触发器代码中的 `for each row` 语句可以显式地在每一被更新的行上进行迭代。
- `referencing new row as` 语句建立了一个变量 `nrow` ，称为过渡变量(transition variable) ，用来存储被更新行的值。

    `referencing old row as` 子句则可以建立一个变量用来存储已经更新或删除行的旧值,

    ```sql
    Referencing old row as -- for deletes and updates
    Referencing new row as -- for inserts and updates
    ```

- `when` 语句指定一个条件，仅对于满足条件的元组系统才会执行触发器中的其余部分。
- `begin atomic...end` 语句用来将多行 SQL 语句集成为一个复合语句。
- 触发事件可以是**插入、删除或更新**，且还可以在事件发生前触发：

    ```sql
    -- 假设所插人分数的值为空白则将这个值用 null 值代替。
    create trigger setnull before update of takes
        referencing new row as nrow
        for each row
        when (nrow.grade = ' ')
        begin atomic
     		   set nrow.grade = null;
        end;
    ```

- 更新时的触发器可以被限制为特定的属性：

    ```sql
    CREATE TRIGGER overdraft_trigger after update of balance on account ...
    ```

# Authorization

**Security** - protection from malicious attempts to steal or modify data.

- Database system level - 身份验证和授权机制只允许特定的用户访问**所需的数据**。
- Operating system level - 操作系统的超级用户可以对数据库做任何他们想做的事情，这需要良好的操作系统级安全性。
- Network level - 必须使用加密来防止**窃听**(未经授权读取信息)、**伪装(**假装为授权用户或从授权用户发送消息)等
- Physical level - 物理的电脑访问使得入侵者破坏资料是可能的，这**需要传统的锁-钥安全**；电脑也必须被保护**免受洪水，火灾等**
- Human level - 必须对用户进行筛选，以确保授权用户**不会给入侵者提供访问权限**；用户应该接受**密码选择和保密**方面的培训

我们可能会给一个用户在数据库的某些部分授予几种形式的权限，对数据的授权包括：

- 授权**读取**数据。
- 授权**插人**新数据。
- 授权**更新**数据。
- 授权**删除**数据。

也可能会给一个用户访问数据库模式的权限，对shcema的授权包括：

- 索引授权(Index)——允许创建和删除索引。
- 资源授权(Resources)——允许创建新的关系。
- 变更授权(Alteration)——允许添加或修改关系中的属性。
- 删除授权(Drop)——允许删除关系。

SQL对**关系**的一些权限包括：

- `Select` - 允许对关系进行读访问，或者使用视图进行查询
- `Insert` - 能够插入元组。
- `Update` - 允许使用update语句进行更新。
- `Delete` - 允许删除元组。
- `References` - 能够在创建关系时声明foreign key。
- `All privileges` - 用作所有特权的缩写形式。

**一个创建了新关系的用户将自动被授予该关系上的所有权限。**

`SQL` 数据定义语言包括**授予**和**收回**权限的命令， `grant` 语句用来授予权限，基本形式为：

```sql
GRANT <privilege list> ON <table | view>
TO <user/role list>
[WITH GRANT OPTION]
```

`<user list>` is:

- user-ids
- public, which allows all valid users the privilege granted
- A role (more details)

属性列表是可选项，通过属性列表可以限制权限在特定属性上；如果省略属性列表，则授予的是关系中所有属性上的权限：

```sql
GRANT update (budget) on department to Amit, Satoshi;
```

`With grant option` - 允许被授予特权的用户将特权传递给其他用户。

使用 `revoke` 命令可以收回权限，其形式与 grant 类似：

```sql
REVOKE <privilege list> ON <table | view>
FROM <user list> [restrict | cascade]
```

- 撤销某个用户的权限可能会导致其他用户也失去该权限，这称为撤销的**级联**(cascade)；在大多数的数据库系统中，**级联是默认行为**。
- 可以通过 `restrict` 进行限制，如果当前存在且需要进行级联删除，则产生一个error。

## Role

通过创建一个对应的 `Role`，可以为一类用户一次指定允许的特权。

在SQL中可以这样创建Role：

```sql
create role instructor;
```

- 权限可以授予给Role或从role撤销，这一点和user是一样的：

    ```sql
    GRANT select on takes
    to instructor	
    ```

- Role也可以分配给用户，甚至分配给其他Role：

    ```sql
    CREATE role dean;
    GRANT dean to Amit;
    GRANT instructor to dean;
    ```

## Authorization on View

用户可以在视图上获得授权，而不必在视图定义中使用的关系上获得任何授权。

- 视图隐藏数据的能力既可以简化系统的使用，也可以通过允许用户只访问他们工作所需的数据来增强安全性。
- 可以使用关系级安全性和视图级安全性的组合来限制用户对用户需要的数据的访问。

例如，假设银行职员需要知道每个分行客户的姓名，但没有被授权查看特定的贷款信息，就可以通过View完成：

```sql
CREATE VIEW cust-loan as 
SELECT branchname, customer-name 
FROM borrower, loan 
WHERE borrower.loan-number = loan.loan-number

GRANT select ON cust-loan to clerk
```

- 视图的创建不需要资源的授权，因为没有创建真正的Relation。
- 创建视图的用户不需要获得该视图上的所有权限，他得到的那些权限也不会为他提供超越他已有权限的额外授权。

- **Some Limitations on SQL Authorizations**
    1. SQL不支持**元组级别**的授权。例如我们不能限制学生只查看(存储的元组)他们自己的成绩。
    2. 随着对数据库的**Web访问**的增长，**数据库访问主要来自应用服务器**。终端用户没有数据库用户id，他们都映射到相同的数据库用户id。
    3. 应用程序(如web应用程序)的所有用户可能都映射到同一个数据库用户。
    4. 在上述情况下，**授权的任务落在应用程序上**，SQL不支持。
        - **优点**：应用程序可以实现**细粒度**的授权，例如对单个元组的授权。
        - **缺点**：授权必须在应用程序代码中完成，并且可能分散在应用程序的各个部分。
        - **检查是否存在授权漏洞变得非常困难**，因为它需要读取大量的应用程序代码。

# Embedded SQL

`SQL` 标准定义了嵌入 SQL 到许多不同的语言中，例如 `C/C++, Cobol, Pascal, Java, Pl/I, Fortran SQL`。查询所嵌入的语言被称为宿主语言，宿主语言中使用的 SQL 结构被称为**嵌入式 SQL**

使用**宿主语言**写出的程序可以通过嵌人式 SQL 的语法访问和修改数据库中的数据，格式如下：

```sql
EXEC SQL <embedded SQL statement> END_EXEC
```

- 嵌人式 SQL 的确切语法**依赖于宿主语言**。

在执行任何 SQL 语句之前，程序必须**首先连接到数据库**。这是用下面语句实现的：

```sql
EXEC SQL connect to <server> user <user_name> using <password> END_EXEC
```

## Cursor

在嵌入的 SQL 语句中可以使用宿主语言的变量，不过前面要加上冒号 `:` 以区别于 SQL 变量。如此使用的变量必须声明在一个 `DECLARE` 区段里：

```sql
--单行查询 
EXEC SQL BEGIN DECLARE SECTION; 
char V_an[20], bn[20]; 
float  bal; 
EXEC SQL END DECLARE SECTION; 
...
scanf(“%s”, V_an);   -- 读入账号,然后据此在下面的语句获得bn, bal的值 
EXEC SQL SELECT branch_name, balance INTO :bn, :bal FROM account WHERE account_number = :V_an; 
END_EXEC
printf(“%s, %s, %s”, V_an, bn, bal);
--:V_an, :bn, :bal是宿主变量，可在宿主语言程序中赋值，从而将值带入SQL。宿主变量在宿主语言中使用时不加:号
```

为了表示**关系查询**，可以使用**声明游标** (`declare cursor`) 语句：

```sql
EXEC SQL
    declare c cursor for
    select ID, name
    from student
    where tot_cred > :credit_amount
END_EXEC
```

- 上述表达式中的变量`c`被称为该查询的**游标** ，我们使用这个变量来标识该查询，然后用 `open` 语旬来执行查询：

    ```sql
    EXEC SQL OPEN c END_EXEC
    ```

- 这条语句使得数据库系统执行这条查询并把执行结果存于一个**临时关系**中。当 `open` 语句被执行的时候，宿主变量(credit_amount) 的值就会被应用到查询中。
- 然后我们利用一系列的 `fetch` 语句把结果元组的值赋给宿主语言的变量。 `fetch` 语句要求结果关系的每一个属性有一个宿主变量**相对应：**

    ```sql
    EXEC SQL fetch c into :si, :sn END_EXEC
    ```

一条单一的 `fetch` 请求只能得到**一个元组**。如果我们想得到所有的结果元组，程序中必须包含对所有元组执行的一个循环。虽然关系在概念上是一个集合，查询结果中的元组还是有一定的**物理顺序**的。执行 SQL `open` 语句后，游标指向结果的第一个元组。执行一条 `fetch` 语句后，游标指向结果中的下一个元组。当后面不再有待处理的元组时，SQLCA 中变量 SQI.STATE 被置为 '02000' （意指不再有数据）

- 使用 `close` 语句来告诉数据库系统**删除用于保存查询结果的临时关系：**

    ```sql
    EXEC SQL close c END_EXEC
    ```

- An example

    ```sql
    Exec SQL include SQLCA; -- SQL通讯区，是存放语句的执行状态的数据结构，其中有 
    											-- 一个变量sqlcode指示每次执行SQL语句的返回代码（success, not_success）
    Exec SQL BEGIN DECLARE SECTION; 
    		char bn[20], bc[30]; 
    Exec SQL END DECLARE SECTION; 
    Exec SQL DECLARE branch_cur CURSOR FOR 
    		Select branch_name, branch_city From branch; 

    Exec SQL OPEN branch_cur; 
    		While (1) { Exec SQL FETCH branch_cur INTO :bn, :bc; 
    										if (sqlca.sqlcode <> SUCCESS) BREAK; 
    										-- 由宿主语句对bn, bc中的数据进行相关处理} 
    Exec SQL CLOSE branch_cur;
    ```

用于数据库修改 `update insert delete` 的嵌入式 SQL 表达式不返回结果，单行的操作可以如下：

```sql
Exec SQL BEGIN DECLARE SECTION; 
		char an[20]; 
		float bal; 
Exec SQL END DECLARE SECTION; 

... 
scanf(“%s, %d”, an, &bal);   -- 读入账号及要增加的存款额 
EXEC SQL update account set balance = balance + :bal 
where account_number = :an; 
...
```

- Cursor 操作的示例如下：

    ```sql
    EXEC SQL BEGIN DECLARE SECTION;  
    		char an[20]; 
    		float bal; 
    EXEC SQL END DECLARE SECTION;
    EXEC SQL DECLARE csr CURSOR FOR 
    		SELECT * 
    		FROM account 
    		WHERE branch_name = ‘Perryridge’ 
    		FOR UPDATE OF balance;
    ...
    --(To update tuple at the current location of cursor) 

    EXEC SQL OPEN csr; 
    While (1) { 
    		EXEC SQL FETCH csr INTO :an, :bn, :bal; 
    				if (sqlca.sqlcode <> SUCCESS) BREAK; 
    				-- 由宿主语句对an, bn, bal中的数据进行相关处理(如打印)
    		EXEC SQL update account 
    		set balance = balance + 100 
    		where CURRENT OF csr; 
    }
    ... 
    EXEC SQL CLOSE csr; 
    ...
    ```

# Function and Procedure

SQL 允许定义**函数、过程和方法**。定义可以通过 SQL 的有关过程的组件，也可以通过外部的程序设计语言，例如 Java C\C++等。

- 函数定义的示例如下：

    ```sql
    create function dept 01mt(depU1ame varchar(20)) 
    		returns integer 
    		begin 
    		declare d_ount integer; 
    				select count(*) into d.count 
    				from instructor 
    				where inslructor.dept_name= dept_name 
    		return d.£ount; 
    end
    ```

    表函数的示例如下：

    ```sql
    create function instructor_of (dept_name varchar(20)) 
    		  returns table ( 
          ID varchar (5), 
          name varchar (20), 
          dept_name varchar (20), 
          salary numeric (8,2))
    return table 
          (select ID, name, dept_name, salary 
           from instructor 
           where instructor.dept_name = instructor_of.dept_name);
    ```

    - **注意**，使用函数的参数时需要加上函数名作为前缀 `instructor_of dept＿name`

函数也可以写成`Procedure`：

```sql
create procedure dept_count_proc(in dept＿name varchar(20), out d_count integer)
		begin 
        select count(*) into d_count
        from instructor
        where instructor.dept_name = dept_count_proc.dept_name
		end
```

- 关键字 `in` 和 `out` 分别表示待赋值的参数和为返回结果而在过程中设置值的参数。

可以从一个 SQL 过程中或者从嵌人式 SQL 中使用 `call` 语句调用过程：

```sql
declare d_count integer
call dept_count_proc('Physics', d_count);
```

SQL 允许**多个过程同名**，只要同名过程的参数个数不同。名称和参数个数用于标识一个过程SQL **也允许多个函数同名**，只要这些同名的不同函数的参数个数不同，或者对于那些有相同参数个数的函数，至少有一个参数的类型不同。