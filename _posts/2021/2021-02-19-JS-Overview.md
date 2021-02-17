---
layout:	post
title: "JS-概述"
subtitle: "As a brief review"
date: 2021-02-19 22:00:00
author: "Yinwhe"
header-style: text
tags:
    - JavaScript
---

[TOC]



# 1. Basic

> JavaScript 是解释型语言。

<details><summary>Js的两种插入方式</summary>
  <ul>
    <li> 内部js - 即在&lt;script&gt;标签之间的js</li>
    <li> 外部js - 如&lt;script src="myScript.js"&gt;&lt;/script&gt;</li>
  </ul>
</details>
<details><summary>Js输出方式</summary>
  <ol>
    <li> 使用 window.alert()弹出警告框</li>
    <li> 使用 document.write()写到HTML文档</li>
    <li> 使用 innerHTML 写入到HTMl元素</li>
    <li> 使用 console.log()写入到浏览器控制台</li>
  </ol>
</details>
## 1.1 数据类型

`js`包括两类数据类型 - <u>原始数据</u> 和 <u>复杂数据</u>

- 原始数据包括 `string number boolean undefined` 这些是`typeof`可以返回的类型
- 复杂数据包括 `function object` 也是`typeof`可以返回的类型，`null Array`也都属于`object`

另外，其数据类型是**动态的**，即相同变量可以用作不同类型



**Undefined 和 Null**

- undefined 与 null 的值相等，但类型不相等：

```javascript
typeof undefined              // undefined
typeof null                   // object
null === undefined            // false
null == undefined             // true
```



**使用`var`定义与不使用的区别**

- `var num=1;`是在<u>当前域</u>中声明变量，如果在方法中声明，则为局部变量，如果是在全局域中声明，则为全局变量
- `num=1;`事实上是对属性赋值操作。首先，它会尝试在当前作用域链（如在方法中声明，则当前作用域链代表全局作用域和方法局部作用域etc）中解析num，如果在任何当前作用域链中找到num，则会执行对num属性赋值，如果没有找到num，它才会在全局对象（即当前作用域链的最顶层对象，如window对象）中创造num属性并赋值。



## 1.2 运算符

| 优先级权重 |                       运算符                       |
| :--------: | :------------------------------------------------: |
|     17     |                      . [] new                      |
|     16     |                         ()                         |
|     15     |                       ++ --                        |
|     14     |    !、~、+(单目)、-(单目)、typeof、void、delete    |
|     13     |                      %、*、/                       |
|     12     |                  +(双目)、-(双目)                  |
|     11     |                    <<、>>、>>>                     |
|     10     |                    <、<=、>、>=                    |
|     9      |                    !=、、!、===                    |
|     8      |                         &                          |
|     7      |                         ^                          |
|     6      |                         \|                         |
|     5      |                         &&                         |
|     4      |                        \|\|                        |
|     3      |                         ?:                         |
|     2      | =、+=、-=、*=、/=、%=、<<=、>>=、>>>=、&=、^=、\|= |
|     1      |                         ,                          |



## 1.3 流程控制

基本流程控制的语法与`C`基本相同，如`if-else, switch-case, for, while, do-while`，需要简单提及的是`for-in`，这一语句用于遍历对象的属性：

```javascript
var person = {fname:"Bill", lname:"Gates", age:62}, text="", x;
for (x in person) {
  //每一次遍历中x的值为对象的属性名
	text += person[x];
}
```

**JavaScript 标签**

如需标记 JavaScript 语句，将标签名和冒号置于语句之前：

```js
label:{
	//statement
}
```

`break` 和 `continue` 语句是仅有的可<u>跳出</u>代码块语句语法：

```js
break labelname;
continue labelname;
```

- continue 语句（不论有无标签引用）只能用于**跳过一个迭代**。
- break 语句，如果没有标签引用，只能用于**跳出一个循环或一个 switch**。
- 如果有标签引用，则 break 语句可用于**跳出任意代码块**





## 1.4 数值 Number

`Js`只有一种数值类型，即`number`，其始终是64位的浮点数，故并不总是100%精确。

<details><summary>关于数字和字符串的<b>相加</b></summary>
  <ul>
    <li>从左到右的顺序计算</li>
    <li>若为两数，则是数字相加</li>
    <li>若存在至少一个字符串，则为字符串级联</li>
    <li>对于其他计算符，则字符串被转换为数字</li>
  </ul>
</details>

<br/>

<font size=5>NaN</font>

**NaN** 属于 `JavaScript` 保留词，指示某个数不是合法数

```js
x = 100 / 'a';
document.write(x);
// output NaN
```

使用 `isNaN()` 确定数值是否为数

- 假如在数学运算中使用了 NaN，则结果也将是 NaN，如果在字符串加法中，则为串连接
- **typeof NaN 会返回 number**



<font size=5>Infinity</font>

**Infinity**（-Infinity）是 `JavaScript` 在计算数时超出最大可能数范围时返回的值。

```js
var myNumber = 2;
while (myNumber != Infinity) {          // 执行直到 Infinity
    myNumber = myNumber * myNumber;
}
```

- 除以`0`也会生成 Infinity



<font size=5>数值方法</font>

| Name                | Function                                        |
| ------------------- | ----------------------------------------------- |
| num.toString(radix) | 以字符串的形式返回以radix为基数进行转换后的数字 |
| num.toExponential() |返回字符串值，它包含已被四舍五入并使用指数计数法的数字；括号里是需要的小数位数|
| num.toFixed() |返回字符串值，它包含了指定小数位数的数字|
| num.Precision() |返回字符串值，它包含了指定的有效位数|



<font size=5>全局方法</font>

即可以用于所有js数据类型的函数，以下是一些处理数字时最相关的方法

|Name|Function|
|----|--------|
|Number()|可用于把 JavaScript 变量转换为数值(相当于valueOf())|
|parseFloat()|解析一段字符串并返回数值。允许开头空格。只返回首个数字|
|parseInt()|解析一段字符串并返回整数；允许开头空格；只返回首个数字|



## 1.5 字符串

使用单引号或双引号定义

- 可以在字符串中使用引号，只要该引号不匹配字符串的引号即可



**字符串长度**

- 可以使用内置属性 **length** 来计算字符串的长度

  ```js
  str = "test string"
  document.write(str.length);
  ```

**访问**

通过`[]`访问，但建议使用`charAt()`方法

- 如果找不到字符，`[]` 返回 undefined，而 `charAt()` 返回空字符串。

<font color="red">Note: 这是只读的，str[0] = "A" 不会产生错误（但也是无效的）</font>



## 1.6 函数

JavaScript 函数通过 `function` 关键词进行定义，函数名可包含字母、数字、下划线和美元符号（规则与变量名相同）。

**`Note`:** 函数可被重复定义，后定义的会覆盖之前定义的函数。

---

<font size=5>函数表达式/匿名函数</font>

函数表达式可以在**变量中存储**

```js
var fc = function (a, b) {return a + b};
var x = fc(1, 2);
```

- 此时函数表达式中的函数名没有意义，不可调用；故也不需要加上函数名

---

<font size=5>函数提升Hoisting</font>

`Hoisting` 是 `JavaScript` 将**声明**移动到当前作用域顶端的默认行为。Hoisting 应用于变量声明和函数声明，正因如此，**JavaScript 函数能够在声明之前被调用**：

```js
myFunction(5);
function myFunction(y) {
      return y * y;
}
```

**`Note`** - 使用<u>表达式定义</u>的函数不会被提升

---

<font size=5>自调用函数</font>

**`函数表达式`**可以作为<u>自调用</u>，假如表达式后面跟着 ()，函数表达式会自动执行。

- **无法对函数声明进行自调用。**

- 需要在函数周围添加括号，以指示它是一个函数表达式

```js
  (x = function fc() {document.write("hello!");})();
  x();
  //调用两次
```

---

<font size=5>箭头函数</font>

箭头函数允许使用简短的语法来编写函数表达式，不需要 `function` 关键字、`return` 关键字和花括号。

```js
const x = (x, y) => x * y; //ES6
```

<details><summary>Note</summary>
  <ol>
    <li>箭头函数没有自己的 this。它们不适合定义对象方法。</li>
    <li>箭头函数未被提升，它们必须在使用前进行定义。</li>
    <li>使用 const 比使用 var 更安全，因为函数表达式始终是常量值。</li>
    <li>如果函数是单个语句，则能省略 return 关键字和大括号，但保留它们是一个好习惯：
      <br/>const x = (x, y) => {return x*y};</li>
  </ol>
</details>
---

<font size=5>参数</font>

<details><summary>参数规则</summary>
  <ul>
    <li>JavaScript 函数定义不会为参数（parameter）规定数据类型。</li>
    <li>JavaScript 函数不会对所传递的参数（argument）实行类型检查。</li>
    <li>JavaScript 函数不会检查所接收参数（argument）的数量。</li>
  </ul>
</details>

故如果调用参数时**省略了参数**（少于被声明的数量），则丢失的值被设置为：**undefined**，有时这是可以接受的，但是有时最好给参数指定默认值：

```js
function f(x, y){
    x = (x === undefined) ? 0 : x;
    y = (y === undefined) ? 0 : y;
}
```

- 当然也可以像`C++`那样使用默认参数

---

<font size=5>arguments对象</font>

`JavaScript` 函数有一个名为 `arguments` 对象的内置对象，`arguments` 对象包含函数调用时使用的参数数组。通过使用该对象可以实现**变长参数函数。**

```js
function sum() {
    var i, sum=0;
    for (i = 0;i<arguments.length;i++)
        sum += arguments[i];
    return sum;
}
document.write(sum(1,2,3));    //6
```

---

<font size=5>this关键词</font>

在 JavaScript 中，被称为 `this` 的事物，指的是“拥有”当前代码的对象。

this 的值，在函数中使用时，是“拥有”该函数的对象。当不带拥有者对象调用对象时，this 的值成为全局对象，在 web 浏览器中，全局对象就是浏览器对象。

- 本例 this 的值返回 window 对象：

```jsx
function f(){
    return this;
}
var x = f();
// x此时即window对象
```



## 1.7 闭包

> 闭包是指有权访问另一个函数作用域中的变量的函数。

**闭包有3个特性**

1. 函数嵌套函数
2. 函数内部可以引用函数外部的参数和变量
3. 数和变量不会被垃圾回收机制回收



利用自调用函数初始化值，并返回真正需要的函数?
```jsx
var add = (function f1() {
    var counter = 0;
    return function f2() {return counter += 1;}
})();

add();
add();
add();
// 计数器目前是 3
```

`原理`：f2被赋给一个全局变量，故其会一直存在内存当中，而f2依存于f1，则f1也一直存在于内存中



## 1.8 Object

>  Js中，除了原始值外，都是对象

**定义** - Js中的对象被`{}`包括，<u>属性</u>或<u>方法</u>都通过**键值对**的形式进行定义，可以把方法也视作属性，即函数表达式的形式。

```js
obj = {
  name:"test",
  getName:funcion(){return this.name;}
}
```

**访问** - 属性和方法的访问可以通过`.`和`[]`；通过`[]`访问时，要加上`""`才行

```js
obj.age=18;
obj["getAge"]=function(){return this.age;}
```

- 属性和方法的**`添加`**都可以直接访问的形式进行；
- 此时方法的定义将允许使用`this`

**删除** - 使用`delete`关键词进行删除

- delete 关键词会同时删除属性的值和属性本身。
- **delete 操作符被设计用于`对象`，无法删除定义的函数与变量。**
- delete 操作符不应被用于预定义的 JavaScript 对象属性，这样做会使应用程序崩溃。
- delete 关键词不会删除被继承的属性，但是如果删除了某个原型属性，则将影响到所有从原型继承的对象。

---

<font size=5>数据属性</font>

属性就是与对象相关的值，而**`数据属性`**就是属性的属性，用于描述属性的行为特性

- **[[Configurable]]**：能否被delete删除属性重新定义
- **[[Enumerable]]**：能否被for-in枚举
- **[[Writable]]**：能否修改属性值
- **[[Value]]**：数据的数据值
- **[[set]]**
- **[[get]]**

`Object.getOwnPropertyDescriptor()`方法可以查看属性的默认数据属性。

`Object.defineProperty()`可以修改默认属性，包含三个参数：属性所在对象，属性名称，描述符对象。

![Screen Shot 2021-02-18 at 5.29.28 PM](https://tva1.sinaimg.cn/large/008eGmZEgy1gnrtghoh5uj30x204kmye.jpg)

---

<font size=5>访问器</font>

即`Getter`和`Setter`，同时`get`和`set`也是一种属性

- **`get` 关键词 -** 可以在访问方法时，不需要使用括号

- **`set` 关键词 -** 可以用来更改设置属性，不需要使用括号

```js
obj = {
  val:1,
  get getV(){return val;},
  set setV(v){val=v;}
}
obj.setV=3;
document.write(obj.getV);
```

<details>
  <summary>为什么使用访问器？</summary>
  <ol>
    <li>它提供了更简洁的语法</li>
    <li>它允许属性和方法的语法相同</li>
    <li>它可以确保更好的数据质量</li>
    <li>有利于后台工作</li>
  </ol>
</details>

---

<font size=5>构造器</font>

> 对象构造器类似于一个类

```jsx
//Learn from example
function Person(name, age){
    this.name = name;
    this.age = age;
}
p1 = new Person("test", 20);
```

- 如果要向构造器中增加属性/方法，只能写在构造器中，而不能通过`.property`的形式创建，这与已有对象不同
- 一般构造函数首字母为大写，与其他函数区分

<details>
  <summary>内建构造器</summary>
  <ol>
    <li>Object()</li>
    <li>String()</li>
    <li>Number()</li>
    <li>Boolean()</li>
    <li>Array()</li>
    <li>RegExp()</li>
    <li>Function()</li>
    <li>Date()</li>
  </ol>
</details>

<details>
  <summary>构造函数创建对象的过程</summary>
  <ol>
    <li>当使用了构造函数，并且new 构造函数()，那么就后台执行了new Object()；</li>
    <li>将构造函数的作用域给新对象，(即new Object()创建出的对象)，而函数体内的this代表new Object()出来的对象。</li>
    <li>执行构造函数内的代码；</li>
    <li>反回新对象(后台直接返回)。</li>
  </ol>
</details>

<details>
  <summary>Notice</summary>
  <ol>
    <li>构造函数和普通函数的唯一区别，就是他们调用的方式不同。构造函数也是函数，必须用new 运算符来调用，否则就是普通函数。</li>
    <li>this就是代表当前作用域对象的引用。如果在全局范围this就代表window对象，如果在构造函数体内，就代表当前的构造函数所声明的对象。</li>
  	<li>这种方法解决了函数识别问题，但消耗内存问题没有解决。同时又带来了一个新的问题，全局中的this在对象调用的时候是Box本身，而当作普通函数调用的时候，this 又代表window。即this作用域的问题。</li>
  </ol>
</details>
---

<font size=5>原型</font>

每个**函数**都有一个`prototype`属性，这个属性是一个对象，它的用途是包含可以由特定类型的所有实例共享的属性和方法。逻辑上可以这么理解：`prototype` 通过调用构造函数而创建的那个对象的原型对象。

- 使用原型的好处可以让所有对象实例共享它所包含的属性和方法。也就是说，不必在构造函数中定义对象信息，而是可以直接将这些信息添加到原型中。

**使用 prototype 属性** - 向构造器中添加属性（添加方法基本相同）

```jsx
Person.prototype.weight = 120;
I = new Person("test", 18);
document.write(I.weight);
//120
```

**原型对象**

每个`javascript`对象都有一个原型对象，这个对象在不同的解释器下的实现不同。比如在*firefox*下，每个对象都有一个隐藏的\_\_proto\_\_属性，这个属性就是“原型对象”的引用。

**原型链**

由于原型对象本身也是对象，根据上边的定义，它也有自己的原型，而它自己的原型对象又可以有自己的原型，这样就组成了一条链，这个就是原型链。

`JavaScritp`引擎在访问对象的属性时，如果在对象本身中没有找到，则会去原型链中查找，如果找到，直接返回值，如果整个链都遍历且没有找到属性，则返回`undefined`。原型链一般实现为一个链表，这样就可以按照一定的顺序来查找。

**\_\_proto\_\_与prototype**

JS在创建对象（不论是普通对象还是函数对象）的时候，都有一个叫做\_\_proto\_\_的内置属性，用于**指向**创建它的函数对象的原型对象prototype。

```js
function Person(name){
  this.name = name;
}
Person.prototype.getName = function (){
  return this.name;
}
I = new Person("test");
document.write(I.__proto__ === Person.prototype);//true
document.write(Person.prototype.__proto__ == Object.prototype);//true
```

- Object.prototype对象也有\_\_proto\_\_属性，但它比较特殊，为`null`

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gnso32ve1tj319h0b7q38.jpg)

**constructor**

**原型对象**`prototype`中都有个预定义的`constructor`属性，用来引用它的函数对象。这是一种循环引用

```jsx
person.prototype.constructor === person //true
Function.prototype.constructor === Function //true
Object.prototype.constructor === Object //true
```

**进一步理解**

```jsx
function Task(id) {
    this.id = id;
}

Task.prototype.status = "STOPPED";
Task.prototype.execute = function (args) {
    return "execute task_" + this.id + "[" + this.status + "]:" + args;
}

var task1 = new Task(1);
var task2 = new Task(2);

task1.status = "ACTIVE";
task2.status = "STARTING";
document.write(task1.execute("task1"))
document.write(task2.execute("taks2"))

/*
execute task_1[ACTIVE]:task1
execute task_2[STARTING]:task2
*/
```

- 构造器会自动为task1,task2两个对象设置原型对象Task.prototype，这个对象被Task(在此为构造器)的prototype属性引用，参看下图中的箭头指向。

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnsr57o90uj30he0pkt9p.jpg" alt="image-20210219125511903" style="zoom:50%;" />



**总结：**实例对象的\_\_proto\_\_指向其构造函数的原型，本质上也是一个对象；构造函数原型的`constructor`指向对应的构造函数；构造函数的`prototype`获得构造函数的原型。



# 2. Other Details

## 2.1 var/let/const

`const`和`let`是基本类似的，只是`const`定义常量，`let`可以定义变量；`var`和`let`/`const`的最大区别在于作用域上，以下分析。

**块作用域**

通过`var`关键词声明的变量没有**块作用域**；在块`{}`内声明的变量可以从块之外进行访问；但使用`let`声明的变量不能在外访问。

```jsx
{ 
  var x = 10; 
  let y = 20;
}
// 此处可以使用 x，但不能使用 y
```

**循环作用域**

```jsx
let i = 7;
for (let i = 0; i < 10; i++) {
  // 一些语句
  var x = 10;
  let y = 20;
}
// 此处 i 为 7；可以访问y，不能访问x
```

**函数作用域**

在函数内声明变量时，使用 `var` 和 `let` 很相似，它们都有**函数作用域**

```jsx
function myFunction() {
  var carName = "porsche";   // 函数作用域
}
function myFunction() {
  let carName = "porsche";   // 函数作用域
}
```

**全局作用域**

如果在块外声明声明，那么 `var` 和 `let` 也很相似，它们都拥有**全局作用域**

```
var x = 10;       // 全局作用域 
let y = 6;        // 全局作用域
```

- 但一点区别是，此时`x`属于`window`，而`y`不属于`window`，不能通过`window.y`调用

<details>
  <summary>关于重新声明</summary>
  <ul>
    <li>在相同的作用域，或在相同的块中，通过 let重新声明一个 var变量是不允许的</li>
    <li>在相同的作用域，或在相同的块中，通过 let 重新声明一个 let 变量是不允许的</li>
    <li>在相同的作用域，或在相同的块中，通过 var 重新声明一个 let 变量是不允许的</li>
    <li>在不同的作用域或块中，通过 let 重新声明变量是允许的</li>
  </ul>
</details>

---

<font size=5>Const</font>

`const` 定义的变量与 `let` 变量类似，但不能重新赋值，故必须在声明时赋值。

`const`没有定义常量值，它定义了对值的常量引用。因此，不能更改常量原始值，但**可以更改常量对象的属性**

```jsx
const PI = 3.141592653589793; 
PI = 3.14;// 会出错 
// 可以创建 const 对象：
const car = {type:"porsche", model:"911", color:"Black"};
// 可以更改属性：
car.color = "White";
// 可以添加属性：
car.owner = "Bill";
```

- 但是无法重新为常量对象赋值

## 2.2 Hoisting

**JavaScript 声明会被提升**

在 JavaScript 中，可以在使用变量之后对其进行声明，换句话说，可以在声明变量之前使用它。

```jsx
x = 5; 				// 把 5 赋值给 x
document.write(x);
var x; 				// 声明 x
```

- Hoisting 是 JavaScript 将所有声明提升到当前作用域顶部的默认行为（提升到当前脚本或当前函数的顶部）

**`Note`** - 用 `let` 或 `const` 声明的变量和常量不会被提升

**JavaScript 初始化不会被提升**

```jsx
var x = 5; // 初始化 x
document.write( x + ' ' + y);
var y = 7; // 初始化 y 
```

- 只有声明（var y）而不是初始化（=7）被提升到顶部，由于 hoisting，y 在其被使用前已经被声明，但是由于未对初始化进行提升，y 的值仍是未定义。
- 如果去掉最后一行，则会无显示，表明`hoist`应该是存在的。

## 2.3 严格模式

`"use strict"`  定义 JavaScript 代码应该以“严格模式”执行；它不算一条语句，而是一段文字表达式，更早版本的 JavaScript 会忽略它。

<details><summary>严格模式下禁止的事项</summary>
  <ul>
    <li>在不声明变量（或对象）的情况下使用变量</li>
    <li>删除变量（或对象、函数）</li>
    <li>重复参数名</li>
    <li>八进制数值文本</li>
    <li>转义字符</li>
    <li>写入只读属性</li>
    <li>字符串 "eval" ，"arguments"不可用作变量</li>
    <li>with语句是不允许的</li>
    <li>不允许 eval() 在其被调用的作用域中创建变量</li>
    <li>在类似 f() 的函数调用中，this 的值是全局对象。在严格模式中，现在它成为了undefined</li>
  </ul>
</details>

通过在脚本或函数的开头添加 `"use strict";` 来声明严格模式

```jsx
"use strict";
x = 3.14;       // 这会引发错误，因为 x 尚未声明
```

- `"use strict"` 指令只能在脚本或函数的开头识别。

## 2.3 this

JavaScript `this` 关键词指的是它所属的对象。它拥有不同的值，具体取决于它的使用位置：

- 在方法中，`this` 指的是所有者对象。
- 单独的情况下，`this` 指的是全局对象。
- 在函数中，`this` 指的是全局对象，严格模式下，`this` 是 `undefined`

- 在事件中，`this` 指的是接收事件的元素。
- 像 `call()` 和 `apply()` 这样的方法可以将 `this` 引用到任何对象。

**`总的来说，this指向拥有这段代码的对象`**

