---
layout:	post
title: "Rust-Entry"
subtitle: ""
date: 2021-04-01 21:00:00
author: "Yinwhe"
header-style: text
tags:
    - Rust
---



## **环境-Cargo**

查看文档：`cargo doc --open`，然后回打开本地浏览器查看文件包含所有依赖的文档。

编译：`cargo build`

检查语法：`cargo check`

运行：`cargo run`

# Basic Grammar

## 变量与可变性

**变量** - `let` 定义的变量是不可变的，`let mut` 定义的变量是可变的

**常量** - 使用 `const` 定义，Rust 常量的命名规范是使用下划线分隔的大写字母单词，并且可以在数字字面值中插入下划线来提升可读性，如 

```rust
#![allow(unused)]
fn main() {
	const MAX_POINTS: u32 = 100_000;
}
```

**隐藏** - 定义一个与之前变量同名的新变量，新变量就会隐藏之前的变量，如：

```rust
fn main() {
    let x = 5;
    let x = x + 1;
    let x = x * 2;
    println!("The value of x is: {}", x);
}
```

隐藏与将变量标记为 `mut` 是有区别的。当不小心尝试对变量重新赋值时，如果没有使用 `let` 关键字，就会导致编译时错误；`mut` 与隐藏的另一个区别是，当再次使用 `let` 时，实际上创建了一个新变量，我们可以改变值的类型，但复用这个名字。

## 数据类型

有两类数据类型子集：标量（scalar）和复合（compound）

Rust 是 **静态类型**语言，在编译时就必须知道所有变量的类型。根据值及其使用方式，编译器通常可以推断出想要用的类型。

当多种类型均有可能时，必须增加类型注解，像这样：`let guess: u32 = "42".parse().expect("Not a number!");`

**标量类型**包括 整型、浮点型、布尔类型和字符类型



**整型**

| 长度 | 有符号 | 无符号 |
| ---- | ------ | ------ |
|8-bit|i8|u8|
|16-bit|i16|u16|
|32-bit|i32|u32|
|64-bit|i64|u64|
|128-bit|i128|u128|
|arch|isize|usize|



**整型字面量**

| 数字字面值 | 例子 |
| ---------- | ---- |
|Decimal (十进制)|98_222|
|Hex (十六进制)|0xff|
|Octal (八进制)|0o77|
|Binary (二进制)|0b1111_0000|
|Byte (单字节字符)(仅限于u8)|b'A'|

- 允许使用 `_` 进行分割，方便读数

**浮点数**有 `f32` `f64` 两种，默认是 `f64`

**布尔类型** `bool` 包括 `true` `false` 两个值

**字符类型** `char` 由单引号指定，`Rust` 的 `char` 类型的大小为四个字节，并代表了一个 `Unicode` 标量值，这意味着它可以比 `ASCII` 表示更多内容。

---

**复合类型**包括 `元组` 和 `数组`

**元组**是一个将多个其他类型的值组合进一个复合类型的主要方式。

- 元组长度固定：一旦声明，其长度不会增大或缩小。
- 使用包含在**圆括号**中的**逗号**分隔的值列表来创建一个元组。
- 元组中的每一个位置都有一个类型，而且这些不同值的类型也不必是相同的。

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

访问元组可以通过`解构`和`索引`两种方式：

```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);
		let (a, b, c) = tup;
    let five_hundred = x.0;
    let six_point_four = x.1;
    let one = x.2;
}
```

**数组**是固定长度的，一旦声明，它们的长度不能增长或缩小；与元组不同，数组中的每个元素的类型必须相同。

- Rust 中，数组中的值位于中括号内的逗号分隔的列表中

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
		let b: [i32; 5] = [1, 2, 3, 4, 5];
		let c = [3; 5];// c = [3,3,3,3,3];
}
```

- 数组的访问可以通过 `[]` ，越界的访问会导致panic

---

## 函数

Rust 中的函数定义以 `fn` 开始并在函数名后跟一对圆括号。大括号告诉编译器哪里是函数体的开始和结尾。

- Rust 代码中的函数和变量名使用 snake case 规范风格。在 snake case 中，所有字母都是小写并使用下划线分隔单词。
- Rust 不关心函数定义于何处，只要定义了就行。

**函数参数**

函数也可以被定义为拥有参数，参数是函数签名的一部分。在函数签名中，必须声明每个参数的类型。

```rust
fn main() {
    another_function(5, 6.0);
}

fn another_function(x: i32, y: f32) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
}
```

**函数返回值**

Rust不对返回值命名，但要在箭头 `->` 后声明它的类型。

- 在 Rust 中，函数的返回值等同于函数体最后一个**表达式**的值。
- 使用 `return` 关键字和指定值，可从函数中提前返回；但大部分函数隐式的返回最后的表达式。

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();
    println!("The value of x is: {}", x);
}
```

**注意理解Rust中的表达式，**函数调用就是一个表达式

## 注释

普通注释，其内容将被编译器忽略掉：

- `//` 单行注释，注释内容直到行尾。
- `/* */` 块注释， 注释内容一直到结束分隔符。

文档注释，其内容将被解析成 HTML 帮助文档:

- `///` 为接下来的项生成帮助文档。
- `//!` 为注释所属于的项（如 crate、模块或函数）生成帮助文档。

## 控制流

`if-else` 语句，其可以作为一个表达式：

```rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

- 不过当语句中两个值的类型不同时，是无法通过编译的，因为无法在编译时确定number的类型

Rust 有三种循环：`loop`、`while` 和 `for`

`loop` 循环会一直持续直到要求停止：

```rust
fn main() {
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 { break counter * 2;}
    };
    println!("The result is {}", result);
}
```

`while` 循环通过其后的条件进行判断：

```rust
fn main() {
    let mut number = 3;
    while number != 0 {
        println!("{}!", number);
        number = number - 1;
    }
    println!("LIFTOFF!!!");
}
```

`for` 循环会对一个集合的每个元素执行一些代码：

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```

# 所有权

所有权规则：

1. Rust 中的每一个值都有一个被称为其 **所有者**（owner）的变量。
2. 值在任一时刻有且只有一个所有者。
3. 当所有者（变量）离开作用域，这个值将被丢弃 (Drop)。

Rust的所有权是很有意思的，其主要表现在能否被`copy`上：

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```

- 这里的`s1`会报错，因为`String`的ownership已经被`s2`拿走了，`s1`就失效了
- 同时也意味着String并没有被复制，所以并不影响运行速度

如果要保证仍然能使用s1，则需要使用`clone`：

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

更有趣的是，这种性质在函数上也是会发生的，如：

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效
    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，所以不会有特殊操作

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。不会有特殊操作
```

如下是一些 `Copy` 的类型：

- 所有整数类型，比如 `u32`。
- 布尔类型，`bool`，它的值是 `true` 和 `false`。
- 所有浮点数类型，比如 `f64`。
- 字符类型，`char`。
- 元组，当且仅当其包含的类型也都是 `Copy` 的时候。比如，`(i32, i32)` 是 `Copy` 的，但 `(i32, String)` 就不是。

## 引用

为了在调用函数后，不至于每次都来回移动所有权，rust有引用这样一个方式：

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

- reference要使用 `&` 符，相应的还有 mutable reference `&mut`, dereference `*`
- 将获取引用作为函数参数称为**借用**
- 其实质表现为：

![https://kaisery.github.io/trpl-zh-cn/img/trpl04-05.svg](https://kaisery.github.io/trpl-zh-cn/img/trpl04-05.svg)

**Note** - 引用默认是不允许修改值的，如果要修改，需要使用**可变引用** `&mut` 

- 不过可变引用有一个很大的限制：在特定作用域中的特定数据只能有一个可变引用
- 也不能在拥有不可变引用的同时拥有可变引用
- 注意一个引用的作用域从声明的地方开始一直持续到最后一次使用为止。例如，因为最后一次使用不可变引用在声明可变引用之前，所以如下代码是可以编译的：

```rust
let mut s = String::from("hello");

let r1 = &s; // 没问题
let r2 = &s; // 没问题
let r3 = &mut s; //大问题
println!("{} and {}", r1, r2);
// 此位置之后 r1 和 r2 不再使用

let r3 = &mut s; // 没问题
println!("{}", r3);
```

## Slice

**字符串 slice**是 `String` 中**一部分值**的引用，它看起来像这样：

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

可以使用一个由中括号中的 `[starting_index..ending_index]` 指定的 range 创建一个 slice，其中 `starting_index` 是 slice 的第一个位置，`ending_index` 则是 slice 最后一个位置的后一个值。

- 字符串字面值其实是slice，如 `let s = "Hello, world!";` 故`s`不可被更改。

# 结构体

定义结构体，需要使用 `struct` 关键字并为整个结构体提供一个名字，接着在 `{}` 中，定义每一部分数据的名字和类型，称为 **字段（field）**

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

**创建**一个实例需要以结构体的名字开头，接着在大括号中使用 `key: value` 的形式提供字段，其中 `key` 是字段的名字， `value` 是需要存储在字段中的数据值：

```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

使用**字段初始化简写语法**来写一个专门创建实例的函数：

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

- 此时的变量名和struct里的名字是相同的

从**已有**的实例创建新的实例：

```rust
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

- `..` 语法指定了剩余未显式设置值的字段应有与给定实例对应字段相同的值。

## 元组结构体

元组结构体有着结构体名称提供的含义，但没有具体的字段名，只有字段的类型：

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black  = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

- 元组结构体实例类似于元组：可以将其解构为单独的部分，也可以使用 `.` 后跟索引来访问单独的值

## 方法

- Learn From Example

    ```rust
    struct Rectangle {
        width: u32,
        height: u32,
    }

    impl Rectangle {
        fn area(&self) -> u32 {
            self.width * self.height
        }
    		fn can_hold(&self, other: &Rectangle) -> bool {
            self.width > other.width && self.height > other.height
        }
    }
    ```

方法通过 `impl(implement)` 进行定义，同时参数前面中要加入 `&self, self &mut self` ，具体取决于需求。

## 关联函数

`impl` 块的另一个有用的功能是：允许在 `impl` 块中定义**不以** `self` 作为参数的函数。这被称为 **关联函数**（*associated functions*），因为它们与结构体相关联。它们仍是函数而不是方法，因为它们并不作用于一个结构体的实例。

关联函数经常被用作返回一个结构体新实例的构造函数。例如可以提供一个关联函数，它接受一个维度参数并且同时作为宽和高，这样可以更轻松的创建一个正方形 `Rectangle` 而不必指定两次同样的值：

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

let sq = Rectangle::square(3);
```

# 枚举

一个简单的定义和使用如下：

```rust
enum IpAddrKind {
    V4,
    V6,
}

let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

除此意外，枚举类型还可以**带有数据**：

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

- **可以看出不同枚举带有的数据可以是不同的类型**

甚至可以更加复杂：

```rust
enum Message {
    Quit,//没有数据
    Move { x: i32, y: i32 },//一个匿名结构体
    Write(String),//String
    ChangeColor(i32, i32, i32),//三个i32
}
```

除此之外，枚举类型也可以用 `impl` 来定义方法：

```rust
impl Message{
    fn call(&self){
        print!("Test!");
    }
}
```

## match

- Learn From Example

    ```rust
    enum Coin {
        Penny,
        Nickel,
        Dime,
        Quarter,
    }

    fn value_in_cents(coin: Coin) -> u8 {
        match coin {
            Coin::Penny => 1,
            Coin::Nickel => 5,
            Coin::Dime => 10,
            Coin::Quarter => 25,
        }
    }
    ```

`match` 分支有两个部分：一个模式和一些代码：第一个分支的模式是值，而之后的 `=>` 运算符将模式和将要运行的代码分开。

当 `match` 表达式执行时，它将结果值按顺序与每一个分支的模式相比较。如果模式匹配了这个值，这个模式相关联的代码将被执行。如果模式并不匹配这个值，将继续执行下一个分支，

`Rust` 也提供了一个模式用于不想列举出所有可能值的场景，使用特殊的模式 `_` 替代：

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    _ => (),
}
```

- 需要注意的是， `_` 应该被放在最后，否则其后的`item`将无法被匹配

## if let

`if let` 获取通过等号分隔的一个模式和一个表达式。它的工作方式与 `match` 相同，这里的表达式对应 `match` 而模式则对应第一个分支：

```rust
let some_u8_value = 0u8;
if let Some(3) = some_u8_value {
    println!("three");
}
```

- 可以认为 `if let` 是 `match` 的一个语法糖，它当值匹配某一模式时执行代码而忽略所有其他值。

可以在 `if let` 中包含一个 `else` ， `else` 块中的代码与 `match` 表达式中的 `_` 分支块中的代码相同，这样的 `match` 表达式就等同于 `if let` 和 `else` ：

```rust
let some_u8_value = 0u8;
if let Some(3) = some_u8_value {
    println!("three");
}else{
	println!("No");
}
```