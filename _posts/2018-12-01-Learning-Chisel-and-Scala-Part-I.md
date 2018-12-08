---
layout:     post
title:      Learning Chisel and Scala
subtitle:   Scala Part I
date:       2018-12-01
author:     Max
header-img: img/post-gray-background.jpg
catalog: true
tags:
    - Chisel
    - Scala
---

### I. Preface
NVDLA源码分析是个漫长的痛并快乐着的过程，所以忙里偷闲的加入到RISC-V的学习中，想通过阅读几个RV开源处理器的源码，深刻体会一下`RV ISA`，`SpinalHDL`和`Chisel`. 但当阅读Chisel的官方Cheatsheet时，感觉还是要学习一下`Scala`，一方面，scala作为chisel基础，要玩转chisel，scala必不可少，另一方面，官网的`"A Short Users Guide to Chisel "`，内容太简洁，缺少了语法的一般性定义，编写和调试可能会感觉无从下手，所以有了本文对Scala的介绍.

作者在学习过程中，主要参考了`Learning Scala`和`Programming in Scala, 3rd Edition`，将自认为的核心内容在本文进行介绍，结合个人理解在内容编排上做了一些调整，对一些Scala特性会与C++/Java进行一些简单对比，辅助理解，不足与遗漏之处，请路过的小伙伴指出. (Scala的一些参考书在Blog的[books repo](https://github.com/VVViy/VVViy.github.io/tree/master/books/scala)可以找到，仅供个人参考，请勿用于商业用途)

由于scala内容比较多，所以会分成两篇介绍，在第二篇文末作者将对Scala语言中一些容易造成混淆和存在关联性的内容进行简单总结，便于查找区分，如`=>`操作符可应用于Match控制逻辑，也可应用于函数字面量，还可以用于import package时的alias等.

---

### II. Data type
#### 1. Value and variable
##### Value

  `Value`是一个具有特定类型的存储单元，定义赋值后，其值保持不变且不可重复赋值. 就定义而言，`value`似乎就是其他编程语言中的常量值，但实际上，Scala中的`value`是数学意义上的概念，可以看作是一个数学常量的符号表示，如`x=5`. `value`是函数式编程的基础元素，函数式程序中几乎都是使用`value`实现计算逻辑，后续会随内容逐步介绍.

```scala
//syntax: val关键字，scala编译器具备根据<data>推断<type>的能力，所以<type>是可选的

val <identifier> [: <type>] = <data>      

//example 1

scala> val aval = "hello"
<console>: aval: String = hello

//example 2

scala> val bval: String = "world"
<console>: bval: String = world
```
##### Lazy value

```scala
//syntax: lazy关键字修饰val定义惰性值，只有在第一次被访问该值时才用<data>初始化value，而非定义时，将在介绍class属性时进一步说明

lazy val <identifier> [: <type>] = <data>
```
##### Variable

  `Variable`与其他编程范式中定义的变量相同，表示堆或栈上分配的一段存储空间，记录了值在生命周期中的存储状态变化，只要存储空间不被回收，便可以重复赋值. 对于`value`和`variable`之间的关系，做个不成熟的比喻，`variable`表示数学中的定义域或值域，即在取值范围内，`variable`可以取任意值，而`value`表示取值范围内的某一点.
  
```scala
//syntax: var关键字，虽然可对变量重复赋值，但只能赋予定义类型或兼容类型的值

var <identifier> [: <type>] = <data>    

//example

scala> var avar = 1
<console>: avar: Int = 1

scala> avar = 2
<console>: avar: Int = 2
```

##### 转型
  - 与其他强类型语言相似，scala支持低精度向高精度数值类型自动转换
  - 但不支持自动由高精度向低精度的截断转型，需要使用to<Type>方法进行人工转型，如
    
```scala
scala> val cval: Byte = 1
<console>: cval: Byte = 1

scala> val dval: Double = cval
<console>: dval: Double = 1.0

scala> val eval: Int = dval
<console>: error: type mismatch;

scala> val eval: Int = dval.toInt
<console>: eval: Int = 1
```

##### 命名规则
  - 构成字符: `字母`，`数字`，`下划线`，`特殊字符`(如*，+，π，φ等，但**不包括[]和.两个**)及`反引号`('ESC'键下面)五类 (注：对于非特殊字符中的周期符号(.)，在对象引用方法时需要留意，scala中例化对象时与java不同，若class没有定义初始化参数，那么例化时不加()，如`val cli = new AClass`，但在例化的同时引用其中的方法，则必须加括号，如`val cli = new AClass().foo)`
  - 首字符: 只能以`字母`，`特殊字符`或\``开头，如
  
```scala
scala> val π = 3.14159
<console>: π: Double = 3.14159

scala> val $ = "USD currency symbol"
<console>: $: String = USD currency symbol

scala> val o_O = "Hmm"
<console>: o_O: String = Hmm

scala> val 50cent = "$0.50"
<console>:1: error: Invalid literal number val 50cent = "$0.50" ^

scala> val a.b = 25
<console>:7: error: not found: value a val a.b = 25

scala> val `a.b` = 4
<console>: a.b: Int = 4
```

---

#### 2. 数值类型
 Scala中包含以下6种数值类型，与其他高级编程语言不同的是，scala中没有`built-in`类型，全部都是`class`.

Table 1. Numeric types

| Name | Description | Size | Min | Max |
|------|-------------|------|-----|-----|
| Byte | Signed integer| 1 byte| -127 | 128 |
| Short | Signed integer | 2 bytes | -32768 | 32767 |
| Int | Signed integer | 4 bytes | -2<sup>31</sup> | 2<sup>31</sup> - 1 |
| Long | Signed integer | 8 bytes | -2<sup>63</sup> | 2<sup>63</sup> - 1 |
| Float | Signed floating point | 4 bytes | | |
| Double | Signed floating point | 8 bytes | | |

##### 数值类型字符
  在定义变量/常量时，使用各数值类型对应的特殊字符等价于显式指定类型参数，即

Table 2. Numeric literals

| Literal | Type | Description |
|---------|------|-------------|
| 5 | Int | 默认整数类型 |
| 0x0f | Int | "0x"表示16进制，但无8进制字符 |
| 5l | Long | "l" |
| 5.0 | Double | 默认浮点数类型 |
| 5f | Float | "f" |
| 5d | Double | "d" |

```scala
//example

scala> val fval = 5d
<console>: fval: Double = 5.0
```

---

#### 3. Scala核心类型继承体系
  Scala中核心的`Type`体系如Fig-1所示，各类型含义及所有类型支持的方法可简述如下

Table 3. Core nonnumeric types

| Name | Decription | Instantiable |
|--------|-------------|:--------------:|
| Any | Root | No |
| AnyVal | "值"类型抽象根类，运行时分配在heap或stack上 | No |
| AnyRef | 引用类型抽象根类，分配在heap上 | No |
| Nothing | 唯一一个可以作为Type使用的类型，一般在函数或方法异常退出时的返回值类型 | No |
| Null | "空"类型，如val a: String = null 或 val b：String = " ", 表明该a/b未指向内存中任何实例 | No |
| Char | 兼容数值类型 | Yes |
| Boolean | scala中的Boolean类型只有true和false两个值，**不支持其他类型向true或false的自动转换**，如非空字符串不会转换为true，数值0也不会被当作false等 | Yes |
| String | 在scala中属于复合数据类型(collection)，是字符的集合 | Yes |
| Unit | 类似于C++中的void关键字，说明函数或表达式无返回值，可以为常量或变量赋Unit类型值，Unit的表示方法为空括号"()"，如 scala> val nov=(), nov: Unit=() | No |


Table 4. Common type operations

| Name | Example | Description |
|--------|-----------|--------------|
| asInstanceOf[\<type>] | 5.asInstanceOf[Long] | 将调用类型值转换为目标类型，若类型不兼容将报错 |
| getClass | (7.0 / 5).getClass | 返回调用值的类型 |
| isIstanceOf | (5.0).isInstanceOf[Float] | 根据调用类型值与目标类型值是否一致，返回相应布尔值 |
| hashCode | "A".hashCode | 计算特定值的哈希值 |
| to<type> | 20.toByte; 47.toFloat | 主要用于数值类型中高精度向低精度截断后的值 |
| toString | (3.0).toString | 继承自Java的toString方法，由于scala中没有内置类型，皆为class，所有类型值都有该方法 |
    
<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%235-%231.jpg?raw=true" width="1200" height="700" />

Fig-1. Scala core types hierarchy

</div>

---

#### 4. Scala运算符
  scala支持的运算符和优先级与其他语言相比没有特殊之处，只是**不支持三元运算符(? :)**，详细内容请参考相关材料.

---

### III. Expression and Built-in control structure
#### 1. 表达式(Expression)

  表达式是函数式程序的基础构成，因为表达式实现了函数式编程中的一个核心思想，即新值存储在新的value中，而不是修改已存在的varible，这什么意思？
  
  在探讨这个问题之前，可以先回顾一下其他编程范式中程序逻辑的实现方式，如在命令式编程中，我们首先会抽象出一系列处理逻辑，然后将变量按照特定顺序依次通过，最后变量中的值就是程序的功能体现，如简单的自增运算`x = x + 1`. 
  
  相比之下，函数式编程则采用了完全不同的实现方式，即针对问题模型，其首先构建出类似数学上的函数方程来描述功能逻辑，之后将函数方程分解成一系列的低阶子函数的运算链条，最后通过求解子函数链条得到最终的功能输出，而在数学中，函数的定义是`f: x -> y`，是描述一个集合到另一个集合的"映射"，例如，集合`A`中的元素`a`在法则`f`作用下，映射到集合`B`中的元素`b`上，显然，在得到输出`b`的同时，输入`a`并未发生改变，所以，`x = x + 1`在数学上是不成立的. 绕了这么大个圈子，就是要说明scala中的表达式描述了这种"映射"关系，其根据输入val产生新的输出val，而非修改已存在的值. 当然，scala也支持面向对象，所以我们依然可以用表达式实现命令式编程逻辑.

##### 定义表达式
  表达式就是个具有返回值的代码单元，表达式的返回值可作为另一个表达式的输入值或保存在value和variable中，由此可以重新定义value和variable的语法，即
  
```scala
//syntax

val <identifier> [: <type>] = <expression> 
var <identifier> [: <type>] = <expression> 
```

   多条表达式可以构成一个表达式块，内部可以包含局部val或var用于表达式块内部，表达式块的最后一个表达式为整个表达式块的返回值; 表达式块有如下两种常用的表现形式：
  
```scala
//方式一：使用{}的一般形式

{ expr1 ; expr2 ; ... } 

或

{ expr1  //多行可省略分号

| expr2  //左侧的"|"是REPL中为标识多行语句自动添加的符号

| ...
|}
```

```scala
//方式二：省略{}并用分号间隔，仅限单行

expr1 ; expr2 ; ...
```

##### 嵌套表达式(Nested expression)
  
  表达式可嵌套，内嵌语句需用{}进行分割，{}也同时界定了不同内嵌层次语句块中value和variable的作用域，如
  
```scala
scala> { val a = 1; { val b = a * 2; { val c = b + 4; c } } }
<console>: res0: Int = 6     //REPL中自动生成的res0可在后续输入中显式使用，类似于matlab中的ans
```
  
##### 语句(Statements)

  语句是没有返回值版的表达式，Scala中的println()和val或var的定义式都是语句，语句虽然没有返回值，但有返回类型——`Unit`，如
  
```scala
scala> val aval = println("aloha")
<console>: aloha
aval: Unit = ()
```

---

#### 2. Scala内置控制表达式
##### if...else

  Scala中条件控制语句支持`if`和`if else`两种，定义形式如下:
  
```scala
//syntax: if后面的表达式可以是单行或多行表达式块，且有返回值

if (<Boolean expression>) <expression>

//example

scala> val result = if ( false ) "what does this return?"
<console>: result: Any = ()
```

注意到，上例返回值为Fig-1中的`root`类型，这是因为布尔条件为真时，if返回类型为`String`(`AnyRef`子类)，为假时返回`Unit`(`AnyVal`子类)，编译器无法确定准确的返回类型，只能返回根类型.

```scala
//syntax: if...else不存在类型不确定的问题，且可嵌套，从而形成了if...else...if...else...

if (<Boolean expression>) <expression>
else <expression>

//example

scala> val x = 10; val y = 20
<console>: x: Int = 10
y: Int = 20

scala> val max = if (x > y) x else y
<console>: max: Int = 20
```

##### match
  
  Scala中的`match`表达式结构类似于C++，Java中的`switch...case`或verilog中的`case...endcase`，定义形式如下，但从实现方式上来讲，更像verilog，因为`match`至多只能与一个`case`匹配，而不需要额外添加`break`控制，`match`匹配遵守位置优先原则，执行完匹配的`case`后立即退出；另外，与其他高级编程语言不同的是，`match`不仅仅能与"值"比较，还能与"类型、正则式、数值范围及数据类型内部成员"进行比较匹配.
  
```scala
//syntax

<expression> match {
       case <pattern match> => <expression or expression block>
      [case...]
}

//example

scala> val x = 10; val y = 20 
<console>: x: Int = 10 
y: Int = 20

scala> val max = x > y match { 
     | case true => x 
     | case false => y 
     | }
<console>: max: Int = 20
```

* 匹配项组合：`match`支持在同一个`case`中存在多个匹配项，与匹配项中任一`pattern`匹配，便执行该`case`，定义如下：
  
```scala
//syntax

case <pattern 1> | <pattern 2> .. => <expression or expression block>

//example

scala> val day = "MON"
<console>: day: String = MON

scala> val kind = day match {
     | case "MON" |  "TUE" | "WED" | "THU" | "FRI" => "weekday" //除最左侧REPL自动添加的多行符号，其余"|"为逻辑或
     
     | case "SAT" | "SUN" => "weekend"
     | }
<console>: kind: String = weekday
```
  
* 默认匹配项：容易发现，在`match`定义式中并没有`default`选项，那么在上述例子中，如果没有匹配项，那么会报`scala.MatchError`错误. 为了避免报错，Scala提供两种方式处理`default`，即值绑定和使用下划线通配符，其中下划线通配符可与任何input匹配，但二者不是绑定关系，所以无法对下划线进行引用.
   
```scala
//syntax：值绑定

case <identifier> => <expression or expression block> //<identifier>是与match前<expression>返回值绑定在一起的任意合法标识符

//example

scala> val message = "ok"
scala> val status = message match {
     | case "false" => 200
     | case other => {       //other标识符与值"ok"绑定
     
     | println(s"couldn't parse $other") //println内部使用了字符串内插的一种格式s，将在介绍String类型的时候介绍
     
     | -1 }
     | }
<console> couldn't parse ok
status: Int = -1
```

```scala
//syntax: wildcard underscore

case _ => <expression or expression block> //case关键字与下划线间有空格


//example

scala> val message = "Unauthorized"
<console>: message: String = Unauthorized

scala> val status = message match {
     | case "Ok" => 200
     | case _ => {
     |     println(s"Couldn't parse $message")  //由于无法引用下划线，这里直接引用了输入message
     
     |     -1
     | }
     | }
<console>: Couldn't parse Unauthorized
status: Int = -1
```
  
* Pattern guard：是将`if`控制逻辑添加到`case pattern`，只有在满足特定条件时才进行匹配，定义如下，需要注意的是，`if`后条件表达式的括号是可选的.

```scala
//syntax

case <pattern> if <Boolean expression> => <expression or expression block>

//example

scala> val resp:String = null
scala> resp match {
     | case s if s != null =>println(s"recieved '$s'")
     | case s => println("error")
     | }
<console>: error
```
  
* 类型匹配：顾名思义，是与数据类型匹配，定义如下，匹配类型`<type>`前的`<indentifier>`定义了一个合法名称的局部变量`variable`，且变量名必须以**小写字母开头**，另外，数据类型匹配支持多态.
   
```scala
//syntax

case <identifier>: <type> => <expression or expression block>

//example

scala> val x: Int = 12180
<console>: x: Int = 12180

scala> val y: Any = x   //子类对象赋值父类引用

<console>: y: Any = 12180  

scala> y match {
     | case x: String => s"'x'"
     | case x: Double => f"$x%.2f"
     | case x: Float => f"$x%.2f"
     | case x: Long => s"${x}l"
     | case x: Int => s"${x}i"   //匹配时，是与子类对象的实际类型匹配
     
     | }
<console>: res9: String = 12180i
```
  
##### for loop
  
  Scala中最常用的迭代控制表达式，定义如下：
  
```scala
//syntax: 定义式与其他语言中的for循环差异比较大，特别是变量x在迭代器中的移动采用<-操作符，后面对内部元素逐一介绍

for (<identifier> <- <iterator>) [yield] [<expression or expression block>]

//example

scala> for (x <- 1 to 7) yield { s"Day $x:" }
<console>: res10: scala.collection.immutable.IndexedSeq[String] 
           = Vector(Day 1:, Day 2:, Day 3:, Day 4:, Day 5:, Day 6:, Day 7:)
```
  
* Range复合数据类型: `for`循环中的迭代器是一个复合数据类型`Range`，表示一段连续的取值范围，与`python`的`range()`函数类似，定义形式有以下3种:
   
```scala
//syntax: 使用关键字to和unti，使用to则取值范围包含结尾的<ending integer>，until则不包含结尾元素，可选的数字间隔参数[by increment]

<starting integer> [to|until] <ending integer> [by increment]

//example: to manner

scala> 1 to 3 by 1
<console>: res5: scala.collection.immutable.Range = Range（1，3）

//example: until manner

scala> 1 until 3
<console>: res5: scala.collection.immutable.Range = Range（1，2）
```

```scala
//syntax：直接使用Range类创建，等效于使用until关键字

Range(<starting integer>, <ending integer>, [by increment])

//example

scala> Range(1, 5, 1)
<console>: res5: scala.collection.immutable.Range = Range（1，3）
```
    
* yield关键字: 虽然和`python`中的`yield`同名，但这里的`yield`可不是一个生成器指示符，它只是将表达式的计算结果收集起来，统一保存为Vector复合数据类型对象，就像`for`语法定义的例子中所显示的.
    
* 迭代器的表示形式：`for`定义中的`(<identifier> <- <iterator>)`存在两种表示形式用来支持多迭代器及其他合法项，即使用括号`(item1 ; item2 ; ...)`，内部用分号分割，另一种则使用花括号，每个`item`占一行，结尾分号可选，如
   
```scala
//example: parentheses based

scala> val quote = "Faith,Hope,,Charity" 
<console>: quote: String = Faith,Hope,,Charity

scala> for ( t <- quote.split(",") ; if t != null ; if t.size > 0 ) { println(t) }
<console>: Faith 
Hope
Charity
```

```scala
//example：curly-braces based

scala> val quote = "Faith,Hope,,Charity" 
<console>: quote: String = Faith,Hope,,Charity

scala> for { 
     | t <- quote.split(",") 
     | if t != null 
     | if t.size > 0 
     | }
     | { println(t) }
<console>: Faith 
Hope
Charity
```
    
* Iterator guard：上列我们已经见识了加`guard`的迭代器了，实际上，就是加了迭代条件，这和C++,Java中`for(init ; condition ; changed value)`的条件检查一致. 与`match`类似，这里`if`后条件表达式的括号也是可选的.
   
```scala
//syntax

for (<identifier> <- <iterator> if <Boolean expression>) ...
```
    
* 嵌套迭代器：这个与一般的循环嵌套有点区别，循环嵌套中每层循环可能要执行不同的逻辑，但这里只是迭代器嵌套，适合一些高等代数中矩阵的运算.
   
```scala
//example

scala> for { x <- 1 to 2
    |  y <- 1 to 3 }
    |  { print(s"($x,$y) ") 
    |}
<console>: (1,1) (1,2) (1,3) (2,1) (2,2) (2,3)
```
    
* 值绑定：所谓值绑定与C++，Java中`for(init ; condition ; changed value)`d `init`初始化值相似，但功能要更强，其语法定义如下，在迭代器中建立的局部value能够简化掉`for`表达式块中的很多工作，而且其不仅可以用于定义其他嵌套迭代器，还可用于`iterator guard`，其他绑定值以及表达式块.
    
```scala
//syntax

for (<identifier 1> <- <iterator>; <identifier 2> = <expression>) ...

//example

scala> val powersOf2 = for (i <- 0 to 8; pow = 1 << i) yield pow
<console>: powersOf2: scala.collection.immutable.IndexedSeq[Int] = Vector(1, 2, 4, 8, 16, 32, 64, 128, 256)
```
   
##### while loop与do...while loop

  这两种形式与其他语言中的很类似，不过多介绍.
  
```scala
//syntax

while (<Boolean expression>) statement  //注意这里是语句，而非表达式


//example 1

scala> var x = 10; while (x > 0) x -= 1 
<console>: x: Int = 0

//example 2

scala> do println(s"Here I am, x = $x") while (x > 0) 
<console>: Here I am, x = 0
```

---

### IV. Functions and Functional programming
#### 1. 函数
   
   Scala中的函数就是添加了名称的表达式，同样是Scala函数式编程范式的基础结构. 
   
   定义函数的目的就是为了提高代码复用率，特别是在函数式程序中，在前面介绍表达式时，我们提到了函数式的数学逻辑，就是将复杂函数方程分解成一系列的低阶子函数运算链条，通过求解子函数链条得到最终的功能输出，这里的子函数等价于Scala函数，所以我们在定义函数时应尽量遵守设计模式中的"单一功能"原则，使函数尽量短小，功能单一，这样不仅能提高复用率，而且也符合数学逻辑，因为多功能的Scala函数定义等价于数学上未完全分解的子函数. 
   
##### 纯函数(Pure functions)

  纯函数是具有数学意义上的函数，也是函数式程序中的主体部分，其主要特征包括：
  
  - 有一个或多个输入参数;
  - 函数体内部只使用输入参数进行运算;
  - 有返回值;
  - 相同输入相同输出;
  - 不使用或影响函数体外部变量;
  - 函数返回值不会被外部变量影响.

  然而，一个功能健全的程序不可能不受外部数据的影响，如文件, 数据库, 网络数据流，所以可以考虑的基本原则是最小化非纯函数数量.

##### 函数的一般定义形式
  
```scala
//syntax: 因为Scala编译器的类型推断能力，函数类型也是可选的

def <identifier>(<identifier>: <type>[, ... ]): [<type>] = <expression or expression block>

//example

scala> def multiplier(x: Int, y: Int): Int = { x * y }
<console>: multiplier: (x: Int, y: Int)Int

scala> multiplier(6, 7)
<console>: res0: Int = 42
```
  
##### 无参数函数的两种定义形式

```scala
//方式一：无参数列表相关符号

def <identifier>: [<type>] = <expression>

//example

scala> def hi: String = "hi" 
<console>: hi: String

scala> hi   //第一章介绍Scala核心类型结构时，提到过Scala的所有类型都继承了Java的toString方法

<console>: res2: String = hi
```

```scala
//方式二：使用空括号定义

def <identifier>()[: <type>] = <expression>

//example

scala> def hi(): String = "hi" 
<console>: hi: ()String

scala> hi()
<console>: res1: String = hi 

scala> hi
<console>: res2: String = hi
```

相较于第一种无参数定义形式，第二种方式显然更好，因为在使用括号调用函数时，很容易与val或var相区分，另外，方式二的例子中，函数调用时的括号是可选的，但需要**注意**的是如果按照第一种定义形式定义无参函数，那么调用时不能带括号.

##### 参数列表
  
* 参数默认值：与其他语言类似，可以在定义Scala函数参数时，指定部分或全部参数的默认值，这样在调用函数时，可以省略部分参数值.
  
```scala
//syntax

def <identifier>(<identifier>: <type> = <value>): <type>

//example

scala> def greet(prefix: String = "", name: String) = s"$prefix$name"
<console>: greet: (prefix: String, name: String)String
```
    
* Vararg参数：与Java中的同名参数相似，Scala也支持定义可变长度参数列表，即在参数类型后追加通配符`*`来匹配0或多个同类型参数. 在函数实现的表达式块中，vararg参数被当作复合数据对象使用，可直接应用于`for`的迭代器中. 
  
```scala
//example

scala> def sum(items: Int*): Int = {
     | var total = 0
     | for (i <- items) total += i
     | total  //表达式块的最后一条作为返回值
     
     | }
<console>: sum: (items: Int*)Int

scala> sum(10, 20, 30) //匹配3个Int类型值

<console>: res11: Int = 60

scala> sum() //匹配0个

<console>: res12: Int = 0
```
    
* 参数分组：用括号对参数列表分别封装进行分组，这种处理似乎没什么亮点，但在后面介绍`Partially Applied Functions and Currying`时可以看到，这是一种符合设计模式的处理方式，即将稳定的和易变的逻辑分别封装，降低耦合度.
  
```scala
//example

scala> def max(x: Int)(y: Int) = if (x > y) x else y 
<console>: max: (x: Int)(y: Int)Int

scala> val larger = max(20)(39) 
<console>: larger: Int = 39
```
    
##### 函数调用
  
* 命名参数与位置参数：对于熟悉verilog的小伙伴来说，这两个概念并不陌生，在例化模块实例时肯定会用到其中一种, 实际上，在C++等语言介绍默认参数值也会讲到. 命名参数调用就是在调用函数时同时给出参数名称和参数值，而位置调用则是按照参数列表中参数顺序提供参数值，但在调用存在默认参数值的函数时需要注意，应该先位置参数值，后命名参数值调用相对稳妥.
  
```scala
//example 1：命名与位置参数调用

scala> def greet(prefix: String, name: String) = s"$prefix $name" 
<console>: greet: (prefix: String, name: String)String

scala> val greeting1 = greet("Ms", "Brown")  //位置参数调用

<console>: greeting1: String = Ms Brown

scala> val greeting2 = greet(name = "Brown", prefix = "Mr") //命名参数调用 

<console>: greeting2: String = Mr Brown

//example 2：混合调用

scala> def greet(prefix: String = "", name: String) = s"$prefix$name"
<console>: greet: (prefix: String, name: String)String

scala> val greeting1 = greet(name = "Paul") //必须使用命名调用，否则，编译器无法确定给定参数值是属于哪个参数

<console>: greeting1: String = Paul

//exaple 3：混合调用

scala> def greet(name: String, prefix: String = "") = s"$prefix$name"
<console>: greet: (name: String, prefix: String)String

scala> val greeting2 = greet("Ola") //调整参数列表中参数位置后，可以不适用命名参数调用

<console>: greeting2: String = Ola
```
  
* 表达式块参数：表达式块调用是将调用函数参数值的计算过程整体作为调用参数进行传递，这种方式不仅可以极大提高代码的可读性，而且对于一些"一次性计算"的函数调用，可以通过这种缩短参数生命周期的方式，节省系统资源. 表达式块调用的背后，需要先计算表达式块，得到返回值后，作为参数值调用函数. 需要**注意**的是，使用表达式块调用时，()要改用{}.
    
```scala
//syntax

<function identifier> <expression block>

//example

scala> def formatEuro(amt: Double) = f"€$amt%.2f"
<console>: formatEuro: (amt: Double)String

scala> formatEuro(3.4645) //值调用

<console>: res4: String = €3.46

scala> formatEuro { val rate = 1.32; 0.235 + 0.7123 + rate * 5.32 } //表达式块调用

<console>: res5: String = €7.97
```
    
* return关键字：Scala函数也有`return`关键字，但与C++等不同的是，其作用是检测到异常输入时，提前返回，阻止程序继续运行.
  
```scala
//example

scala> def safeTrim(s: String): String = {
| if (s == null) return null
| s.trim()
| }
<console>: safeTrim: (s: String)String
```
  
##### Procedure
  没有返回值的函数，即`Unit`类型函数，如果函数体的表达式是个语句，且未显式定义函数类型，那么就会被编译器推断为`procedure`.

```scala
//example

scala> def log(d: Double) = println(f"Got value $d%.2f") //隐式推断

<console>: log: (d: Double)Unit

scala> def log(d: Double): Unit = println(f"Got value $d%.2f") //显式声明

<console>: log: (d: Double)Unit

scala> def log(d: Double) { println(f"Got value $d%.2f") } //非正式版procedure定义

<console>: log: (d: Double)Unit

scala> def foo = { val he = "heja" } //val或var的定义属于语句

<console>: foo: Unit
```

##### 递归函数(Recursive functions)

  递归函数在函数式程序中比较常见，因为它提供了一种不使用变量就能实现对值进行迭代计算的途径，很多Scala中的数据结构也用了递归. 本身没什么特别，使用时注意`stack overflow`.

```scala
//example

scala> def power(x: Int, n: Int): Long = {
     | if (n >= 1) x * power(x, n-1)
     | else 1
     | }
<console>: power: (x: Int, n: Int)Long

scala> power(2, 8)
<console>: res6: Long = 256
```

##### 嵌套函数(Nested functions)

  在之前的表达式一节中，介绍表达式是可嵌套的，函数作为命名版的表达式，当然也是可嵌套，内嵌函数是局部函数，可以直接在函数体内使用，而且可以"重载".
  
```scala
//example 1

scala> def max (x: Int, y: Int) = {
     | def mul (x: Int, y: Int) = x * y
     | if (x > mul(x, y)) x else mul(x, y)
     | }
<console>: max: (x: Int, y: Int)Int

scala> max(3,4)
<console>: res10: Int = 12

//example 2

scala> def max(a: Int, b: Int, c: Int) = {
     | def max(x: Int, y: Int) = if (x > y) x else y //虽然内外函数名相同，但参数列表不同，编译可以区分
     
     | max(a, max(b, c))                             //即使函数名和参数列表都相同，也不会冲突，因为在外部函数体内内嵌函数优先级高于外部函数
     
     | }
<console>: max: (a: Int, b: Int, c: Int)Int

scala> max(42, 181, 19) 
<console>: res10: Int = 181
```

##### 泛型函数(Generic functions)

  熟悉面向对象语言的对泛型肯定不陌生，要是没泛型，也就没有C++的STL了. 简单的说，泛型就是函数逻辑与参数类型无关. Scala也支持泛型，定义函数时将函数类型声明为变量，调用函数时，除了参数值，还可以选择传递类型参数，用于指示函数参数类型或返回值类型.
  
```scala
//syntax

def <function-name>[type-name](parameter-name>: <type-name>): <type-name>...

//example

scala> def identity[A](a: A): A = a
identity: [A](a: A)A

scala> val s: String = identity[String]("Hello") //调用时，显式声明函数类型

s: String = Hello

scala> val s: String = identity("Hello") //Scala编译器有类型推断能力，所以可以省略类型说明

s: String = Hello
```

---

#### 2. 函数式与电路
   看完Scala再研究Chisel，有点明白为什么UCB用函数式语言作为Chisel的基底，细想之下，数字电路的实现逻辑确实和函数式是相似的，寄存器中的操作数经过一系列的组合逻辑，最后得到一个输出信号，这就是函数式的程序逻辑，如[Chisel wiki: Functional Module Creation](https://github.com/freechipsproject/chisel3/wiki/Functional-Module-Creation)，路过的小伙伴有何思路，留言聊聊?

```scala
//functional sub-module

object Mux2 {
   def apply(sel: UInt, in0: UInt, in1: UInt) = {
       val m = Module(new Mux2)
       m.io.in0 := in0
       m.io.in1 := in1
       m.io.sel := sel
       m.io.out
   }
}

//macro module

class Mux4 extends Module {
  val io = IO(new Bundle {
    val in0 = Input(UInt(1.W))
    val in1 = Input(UInt(1.W))
    val in2 = Input(UInt(1.W))
    val in3 = Input(UInt(1.W))
    val sel = Input(UInt(2.W))
    val out = Output(UInt(1.W))
  })
  io.out := Mux2(io.sel(1),
                 Mux2(io.sel(0), io.in0, io.in1),   //此处确实契合了两个本不相关的概念
                 
                 Mux2(io.sel(0), io.in2, io.in3))
}

```
---

#### 3. First-class function
  `first-class function`是函数式的最核心实现基础，没有对它的支持就无法实现函数式，维基上对它的解释挺抽象的，这里我们引用一下书上的定义，如下

> One of the core values of functional programming is that functions should be first-class. The term indicates that they are not only 
> declared and invoked but can be used in every segment of the language as just another data type. A first-class function may, as with 
> other data types, be created in literal form without ever having been assigned an identifier; be stored in a container such as a 
> value, variable, or data structure; and be used as a parameter to another function or used as the return value from another function.

   简单总结下来就是，`first-class function`是一种数据类型，和`Int，Double`一样可以定义value，variable，可以用在任何类型能够使用的地方. 这就成函数式的核心基础了?
   
   可以再回顾之前在介绍表达式时分析得到的数学逻辑，将复杂函数方程分解成一系列的低阶子函数运算链条，通过求解子函数链条得到最终的功能输出，实际上，子函数链条的连接正是通过`first-class function`，举个栗子来看看，假设要用函数式程序实现下式：
   
<div align="center">
    
f(x, y) = e<sup>sin(x<sup>2</sup> - y)</sup>

</div>

按照我们分析过的数学逻辑，我们应该做如下分解，构成一个从低阶子函数到最终输出的计算链条，在不使用变量的情况下，这个链条是如何连接的? 是不是可以通过使用`first-class function`的概念，依次在后级表达式中定义前级函数的参数，最后只要传递最低阶函数的对象便可完成了呢. 另一方面，以函数作为参数或/和返回值的函数称为"高阶函数". 这些在Scala中都是支持的，下面逐一介绍.

<div align="center">
    
y<sup>1/2</sup> ---> (x - y<sup>1/2</sup>)(x + y<sup>1/2</sup>) ---> sin((x - y<sup>1/2</sup>)(x + y<sup>1/2</sup>)) ---> e<sup>sin((x - y<sup>1/2</sup>)(x + y<sup>1/2</sup>))</sup> = f(x , y)     

</div>

##### 函数类型与值

* 定义函数类型值的一般定义形式：函数的类型是由其输入参数类型和返回值类型组合而成的.

```scala
//syntax：如果函数的参数列表中只有一个输入参数，可以省略掉输入参数的括号

([<type>, ...]) => <type>  //函数输入参数类型列表 => 函数返回值类型


//example

scala> def double(x: Int): Int = x * 2
<console>: double: (x: Int)Int

scala> val myDouble: (Int) => Int = double  //注意这里的定义值的方式 val <identifier> : <function type> = <function identifier>

<console>： myDouble: Int => Int = <function1>


scala> val yourDouble: Int => Int = double  //只有一个输入参数，省略掉括号

<console>： yourDouble: Int => Int = <function1>

scala> myDouble(5)                 //myDouble就是个普通的value，但却有函数的功能

<console>: res1: Int = 10

scala> val myDoubleCopy = myDouble //在使用方式上，函数定义的值与Int等其他类型值没有区别

<console>: myDoubleCopy: Int => Int = <function1>

scala> myDoubleCopy(5)
<console>: res2: Int = 10
```

* 函数类型隐式定义：上面例子定义`myDouble`时，显式声明了该`value`的函数类型，也可以使用下划线通配符来简化类型的书写.

```scala
//syntax

val <identifier> = <function name> _ //注意<function name>和下划线之间有空格


//example

scala> def double(x: Int): Int = x * 2
<console>: double: (x: Int)Int

scala> val myDouble = double _      //简化函数类型值的定义形式

<console>: myDouble: Int => Int = <function1>
```

* 无参数函数类型：定义这种函数类型值时许注意，在定义函数时要使用带括号的版本，否则在不使用函数字面量的情况下，无法定义值. (这种方式以被官网声明弃用，但依然可以使用，最好使用后面介绍的函数字面量)

```scala
//example

scala> def logStart() = "=" * 50 + "\nStarting NOW\n" + "=" * 50 //如果定义成 def logStart = ...，下面定义值时便会报错

<console>: logStart: ()String

scala> val start: () => String = logStart    //在新版2.12.7 REPL中将报warning

<console>: start: () => String = <function0>

scala> println( start() )
<console>:
==================================================
Starting NOW
==================================================
```

##### 高阶函数(Higher-order functions)

  本节开头提到过高阶函数，就是以函数作为参数或/和返回值的函数，如

```scala
//example

scala> def safeStringOp(s: String, f: String => String) = { //注意函数类型参数的形式，para：函数输入参数类型列表 => 函数返回值类型

     | if (s != null) f(s) else s
     | }
<console>: safeStringOp: (s: String, f: String => String)String

scala> def reverser(s: String) = s.reverse
<console>: reverser: (s: String)String

scala> safeStringOp(null, reverser) //和定义函数类型值一样，直接使用函数名调用

<console>: res4: String = null

scala> safeStringOp("Ready", reverser)
<console>: res5: String = ydaeR
```

##### 函数字面量(Function literal)
  
  前面的例子中，是先定义函数再定义变量，实际上，很多函数式中的子函数出现频率并不高，那么将其定义为函数以备复用的目的就会打折，这种情况下，选择用本节介绍的`函数字面量`，或`无名函数、Lambda表达式，Lambdas`等等，特别在Scala中，编译器会根据函数字面量的输入参数个数用*function0，function1, ...functionN*这样的别名来表示.
  
* 定义

```scala
//syntax

([<identifier>: <type>, ... ]) => <expression>

//example

scala> val doubler = (x: Int) => x * 2  //从编译器类型推断的角度，有了参数列表和函数体也就知道了输入和输出类型

<console>: doubler: Int => Int = <function1> 

scala> val doubled = doubler(22)
<console>: doubled: Int = 44
```

* 无参数函数字面量：在前面介绍无参数函数定义变量时，曾提到新版编译器会对一般定义形式报`warning`，同时推荐使用函数字面量的方式定义变量，即

```scala
//example

scala> def logStart() = "=" * 50 + "\nStarting NOW\n" + "=" * 50 //前面的例子中必须使用无参数括号，否则无法定义函数类型的value和variable

<console>: logStart: ()String

scala> def logStartNoParentheses = "=" * 50 + "\nStarting NOW\n" + "=" * 50 //但使用函数字面量则不受有无括号的限制

<console>: logStart: ()String

scala> val start = () => "=" * 50 + "\nStarting NOW\n" + "=" * 50  //方式一

<console>: start: () => String = <function0>

scala> println( start() )
==================================================
Starting NOW
==================================================

scala> val start = () => logStart()  //方式二不具有一般意义，只有定义了有名函数才有此等价定义

<console>: start: () => String = <function0>

scala> val start = () => logStartNoParentheses   //无参数函数定义value

<console>: start: () => String = <function0>

```

* 高阶函数调用：因为没有显式的命名函数定义，所以在高阶函数调用时，函数字面量的定义放在函数调用中的，有种特殊情况是，当函数字面量只有一个输入参数时，输入参数的类型和括号都是可选的，因为函数字面量的输入和输出类型都已经在高阶函数中定义过了，所以很容易推断，但要注意该省略形式**仅限于单输入参数**的情况.

```scala
//example

scala> def safeStringOp(s: String, f: String => String) = {
     | if (s != null) f(s) else s
     | }
<console>: safeStringOp: (s: String, f: String => String)String

scala> safeStringOp("Ready", (s: String) => s.reverse)
<console>: res8: String = ydaeR

scala> safeStringOp("Ready", s => s.reverse)  //省略单输入参数的类型和括号

<console>: res10: String = ydaeR
```

**====Tips====**

    函数字面量背后的原理涉及到类和一些特殊的类方法，所以在下一篇blog介绍完类之后，会再次谈函数字面量的一些进阶玩法.
  
##### 占位符(Placeholder)

  函数字面量简化掉了命名函数的定义过程，占位符是则**函数字面量**的基础上，进一步简化掉了参数列表，占位符使用下划线`_`表示，所谓"占位"是用下划线按位置顺序占据参数列表中参数的位置，以达到替代表示的目的，占位符的使用需满足两个条件：
  
  - 函数字面量的输入输出类型必须在字面量外部显式定义;
  - 参数列表中的每个参数最多只能被使用一次.
  
```scala
//example 1

scala> val doubler: Int => Int = _ * 2  //等号后_ * 2就是简化版字面量.函数的类型在val类型说明中指定，仅有的一个输入参数也只被使用一次，所以满足应用条件

<console>: doubler: Int => Int = <function1>
```

```scala
//example 2: 应用于高阶函数

scala> def safeStringOp(s: String, f: String => String) = {
     | if (s != null) f(s) else s
     | }
<console>: safeStringOp: (s: String, f: String => String)String

scala> safeStringOp("Ready", (s: String) => s.reverse) //字面量原始版

<console>: res8: String = ydaeR

scala> safeStringOp("Ready", s => s.reverse)  //单参数简化版

<console>: res10: String = ydaeR

scala> safeStringOp("Ready", _.reverse) //占位符简化版，对比原始版容易理解，函数输入输出类型已在高阶函数参数中指定，占位符替代了s, (s: String)已无表示必要

<console>: res12: String = ydaeR
```

```scala
//example 3: 占位符是按位置顺序替代

scala> def combination(x: Int, y: Int, f: (Int,Int) => Int) = f(x,y) 
<console>: combination: (x: Int, y: Int, f: (Int, Int) => Int)Int

scala> combination(23, 12, _ * _)  //两输入，按高阶函数参数中函数类型参数的位置占位

<console>: res13: Int = 276
```

```scala
//example 4: 应用于高阶泛型函数

scala> def tripleOp[A,B](a: A, b: A, c: A, f: (A, A, A) => B) = f(a,b,c)
<console>: tripleOp: [A, B](a: A, b: A, c: A, f: (A, A, A) => B)B

scala> tripleOp[Int,Int](23, 92, 14, _ * _ + _)
<console>: res15: Int = 2130

scala> tripleOp[Int,Boolean](93, 92, 14, _ > _ + _)
<console>: res17: Boolean = false
```

##### Partially Applied Functions and Currying

* Partially Applied Functions：除了具有默认值参数的函数，大多数语言在调用函数时，都要提供完整的参数，即使是具有默认值的函数，也是在定义函数时指定一个值，那么函数的灵活性就局限在了被调函数. 

如果主调函数是多参数的，且在一些应用场景下，部分参数为常数或测试时要做增量测试，即灵活性需要掌握在主调函数手中，那么现有大多数语言都是无法满足这个需求的. 但Scala中的部分参数调用机制是可以实现这种逻辑的，这也许得益于其对函数式的支持，因为数学上这种情况非常多(作者本科数学专业，所以有些了解)，例如数学分析中的多元函数，无论求解极限还是微积分都是要将多元转化一元来实现的，所以作为函数式语言，部分参数调用机制是十分必要的.

```scala
//syntax：使用函数时用下划线通配符标识可变参数，可变参数类型不可省略，这里实际上是重定义了函数，所以类型用来生成函数

([value], _: <type>,...)

//example：对比完整调用与部分调用

scala> def factorOf(x: Int, y: Int) = y % x == 0  //完整调用

<console>: factorOf: (x: Int, y: Int)Boolean

scala> val f = factorOf _   //如果两个参数值都将改变  

<console>: f: (Int, Int) => Boolean = <function2>

scala> val x = f(7, 20)    //提供两个参数值才能正常调用

<console>: x: Boolean = false

scala> val multipleOf3 = factorOf(3, _: Int)  //部分调用，使用时标明可变与不可变参数

<console>: multipleOf3: Int => Boolean = <function1>

scala> val y = multipleOf3(78)  //调用时仅给出一个参数值即可

<console>: y: Boolean = true
```

* Currying: 更灵活且符合设计模式的方式是对参数列表分组，这样对于一个拥有多输入参数的函数，方便管理静态参数与动态参数，应用时只改变动态分组的参数.

```scala
//example

scala> def factorOf(x: Int)(y: Int) = y % x == 0
<console>: factorOf: (x: Int)(y: Int)Boolean

scala> val isEven = factorOf(2) _ //定义静态参数值

<console>: isEven: Int => Boolean = <function1>

scala> val z = isEven(32) //应用动态分组参数

<console>: z: Boolean = true
```

##### By-Name Parameters

  `by-name parameters`是用来优化高阶函数参数的，调用高阶函数时，这类参数能够接受常规类型`value`和函数类型值，这种形式的参数进一步提升了高阶函数的灵活性，但有一个潜在的**性能风险**是，如果调用时传递的是函数类型值，那么高阶函数体内每次访问对应参数时会反复调用，所以可能会带来不一致的问题，而且对执行代价比较大的函数，如数据库查询操作，会带来性能下降问题，所以在定义这类参数时要注意或减少对这类参数的访问.

```scala
//syntax：灵活性在于这类参数仅限定返回值类型，而输入类型无要求，这符合之前分析的数学上子函数链条的计算逻辑，后一个子函数的输入只与前一函数输出相关.

<identifier>: => <type>

//example

scala> def doubles(x: => Int) = {   //传入的函数参数的返回值必须是Int类型

     | println("Now doubling " + x)
     | x * 2
     | }
<console>: doubles: (x: => Int)Int

scala> doubles(5)       //value作为参数值

<console>: Now doubling 5
res18: Int = 10

scala> def f(i: Int) = { println(s"Hello from f($i)"); i }
<console>: f: (i: Int)Int

scala> doubles( f(8) )  //函数作为参数值，返回值类型为Int

<console>: Hello from f(8)
Now doubling 8
Hello from f(8)
res19: Int = 16
```

##### Partial Functions

  `partial`函数与`total`函数相对应，可以从数学上的函数定义域来理解这二者的差异，数学上有些函数定义域为整个实数域`R`，如x<sup>2</sup>，那么在调用这类函数时，提供定义域内任何值，函数都能够正确处理，这类函数称为`total function`；而另外一些函数的定义域为R<sup>+</sup>，如`ln(x)`，那么一旦传入0或负数，函数是无法处理的，即函数只能应用于部分输入，这类函数就称为`partial function`.
  
  Scala中的`partial function`就是描述这类函数的，个人理解，这种方式就是将其他编程语言中需要进行异常检测处理的内容搬到了函数定义中，像除0，负数开根号等，部分函数属于函数字面量的一种.
  
```scala
//example

scala> val statusHandler: Int => String = {
     | case 200 => "Okay"
     | case 400 => "Your Error"
     | case 500 => "Our error"
     | }
<console>: statusHandler: Int => String = <function1>

scala> statusHandler(200)
<console>：res20: String = Okay

scala> statusHandler(400)
res21: String = Your Error

scala> statusHandler(401)  //非限定输入会报匹配错误，====partial函数允许使用match中介绍的下划线通配符来提供默认选项====

scala.MatchError: 401 (of class java.lang.Integer)
at $anonfun$1.apply(<console>:7)
at $anonfun$1.apply(<console>:7)
... 32 elided
```

##### 函数字面量块参数

  前面介绍过使用表达式语句块作为函数调用的参数值，同样的，在高阶函数调用时也可以使用函数字面量块作为参数值，准确的说，还是使用的表达式块，只不过表达式块是函数字面量的组成部分.

```scala
//example 1：参数混合调用

scala> def safeStringOp(s: String, f: String => String) = {
     | if (s != null) f(s) else s
     | }
<console>: safeStringOp: (s: String, f: String => String)String

scala> val uuid = java.util.UUID.randomUUID.toString
<console>: uuid: String = bfe1ddda-92f6-4c7a-8bfc-f946bdac7bc9

scala> val timedUUID = safeStringOp(uuid, { s =>  //单参数与块参数混合调用

     | val now = System.currentTimeMillis
     | val timed = s.take(24) + now
     | timed.toUpperCase
     | })
<console>: timedUUID: String = BFE1DDDA-92F6-4C7A-8BFC-1394546043987
```

```scala
//example 2: 使用参数分组

scala> def safeStringOp(s: String)(f: String => String) = {
     | if (s != null) f(s) else s
     | }
<console>：safeStringOp: (s: String)(f: String => String)String

scala> val timedUUID = safeStringOp(uuid) { s =>
     | val now = System.currentTimeMillis
     | val timed = s.take(24) + now
     | timed.toUpperCase
     | }
<console>: timedUUID: String = BFE1DDDA-92F6-4C7A-8BFC-1394546915011
```
