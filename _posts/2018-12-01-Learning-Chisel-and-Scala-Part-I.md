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

> 本文采用[知识共享 署名-非商业性使用-禁止演绎 4.0 国际协议授权（CC BY-NC-ND 4.0）](https://creativecommons.org/licenses/by-nc-nd/4.0/)，转载请注明出处.

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
  - 构成字符: `字母`，`数字`，`下划线`，`特殊字符`(如*，+，π，φ等，但**不包括[]和.两个**)及`反引号`('ESC'键下面)五类 (注：对于非特殊字符中的周期符号(.)，在对象引用方法时需要留意，scala中例化对象时与java不同，若class没有定义初始化参数，那么例化时不加()，如`val cli = new AClass`，但在例化的同时引用其中的方法，则必须加括号，如`val cli = new AClass().foo()`
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
| Byte | Signed integer| 1 byte| -128 | 127 |
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
    
<img width="1200" height="700" src="data:image/jpg;base64,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/wBBNdr4L/5FrSP+wfD/AOgLXFa9/wAi/qP/AF7Sf+gmu18F/wDItaR/2D4f/QFry8w+NHrZd8DOlooorzz0AooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigArzDxN/yVP8A7g6/+jTXp9eYeJv+Sp/9wdf/AEaa3w38WJhiv4MiWiiivdPACiiigAooooAKKKKACuK8UaYPFuunQySILKye4ds8CeQFYs/QBmrta4ix8E2+rz3+q6/bTpe3d07Ki3DJsiHyxg7WwTgZ/GsqqckopXNqLUW5N2L9n4jmb4etrRQNeW1q5ljkz/rYwQwP4g/nVJfFmvQLp+oajokNtpV5KkP+uJmj3nCuwxgA8cdRmq6eGr7TNM8VaLY2rvp91AZLEmQEl2TDoSTnqB1rW8R6Xe33hewtLaAyTxT2zOm4DAVhu6ntWd6lvNL8TS1Pm9X+Bd07W5J9a1jTbyNInsWR42GcPCy5Dc9wQQaf4a1W41vRY9RnhWFZ3doVGc+VuIUnPcgZ/GuY+IFpdx6hp9zprKtxqaNpEo7lJOQ3/AcMc+9dxaWsVlZwWkC7YYY1jRfQAYFaQcnNp9P1M5qKgpLr+m5NRRRWpiFFFFABRRRQAUUUUAZ+vf8AIv6j/wBe0n/oJrtfBf8AyLWkf9g+H/0Ba4rXv+Rf1H/r2k/9BNdr4L/5FrSP+wfD/wCgLXl5h8aPWy74GdLRRRXnnoBRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABXmHib/kqf/cHX/wBGmvT68w8Tf8lT/wC4Ov8A6NNb4b+LEwxX8GRLRRRXungBRRRQAUUUUAFFFFABRRRQAUUUUAU7jTLS71Czvp4y89nvMB3HClhgnHQnHrVyiiiyQXbCiiigAooooAKKKKACiiigDP17/kX9R/69pP8A0E12vgv/AJFrSP8AsHw/+gLXFa9/yL+o/wDXtJ/6Ca7XwX/yLWkf9g+H/wBAWvLzD40etl3wM6WiiivPPQCiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACvLvGhubH4gpqH9n3txbNpgiD28BkAbzCcHHtXqNIyhlKnoRiqhNwkpImcFOLi+p47/wAJCP8AoEav/wCAbUf8JCP+gRq//gG1es/YY/7zUfYY/wC81dX16qcn1Cl5nk3/AAkI/wCgRq//AIBtR/wkI/6BGr/+AbV6z9hj/vNR9hj/ALzUfXqofUKXmeTf8JCP+gRq/wD4BtR/wkI/6BGr/wDgG1es/YY/7zUfYY/7zUfXqofUKXmeTf8ACQj/AKBGr/8AgG1H/CQj/oEav/4BtXrP2GP+81H2GP8AvNR9eqh9QpeZ5N/wkI/6BGr/APgG1H/CQj/oEav/AOAbV6z9hj/vNR9hj/vNR9eqh9QpeZ5N/wAJCP8AoEav/wCAbVR03x1pur+f9gs9SuPIfZJ5dtuwe3Q98H8q7L4m6jNpXh6LTNLZm1rWphY2SA8gtwz+wUHr2JFczc+G7X4UeIfD2p2jMNFu400vVX6ASdUnP1OcnsPrR9eqh9QpeY7/AISEf9AjV/8AwDaj/hIR/wBAjV//AADavWfsMf8Aeaj7DH/eaj69VD6hS8zyb/hIR/0CNX/8A2o/4SEf9AjV/wDwDavWfsMf95qPsMf95qPr1UPqFLzPJv8AhIR/0CNX/wDANqP+EhH/AECNX/8AANq9Z+wx/wB5qPsMf95qPr1UPqFLzPJv+EhH/QI1f/wDaj/hIR/0CNX/APANq9Z+wx/3mo+wx/3mo+vVQ+oUvM8m/wCEhH/QI1f/AMA2o/4SEf8AQI1f/wAA2r1n7DH/AHmo+wx/3mo+vVQ+oUvM8c1TWGvNKu7aLSNW8yWFkXdaNjJGK9L8IRyQ6BpcUqMkiWMSsrDBUhFyCK2PsMf95qkitlhfcCScY5rCrWlVd5HRSoxpK0SaiiisjUKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAoopCAQQRkHgigDynw7qmneK/iZqfim8v7VNP0gHTtKSWZV3N/y1mAJ75wD3B9q7HxIfDvijw5f6Nd6tYGG6iMe77Qh2N1Vhz1BAP4VnH4QeASST4ct+fSWQf+zVxPw4+HHhHWoPEzajosU5tNfurWDdI42RIE2rw3bJoA7T4V+JJdb8LNYX0qvqujymxuyrbt+3hHB7hgOvcg13NYXh7wb4f8KNcNoemR2bXAUSlGZt23OPvE+prdoAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAK87+En/Hv4w/7Gi9/kleiV538JP+Pfxh/wBjRe/ySgD0SiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigArzv4Sf8e/jD/saL3+SV6JXnfwk/wCPfxh/2NF7/JKAPRKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACvN9C0zU/E9zrl1N4m1e1Fvqs9tHFbzBUCLgjjHv+lekVxnw7/wBT4k/7D11/7JQA/wD4Qi9/6HHxB/3/AB/hR/whF7/0OPiD/v8Aj/CuwooA4/8A4Qi9/wChx8Qf9/x/hR/whF7/ANDj4g/7/j/CuwooA4//AIQi9/6HHxB/3/H+FH/CEXv/AEOPiD/v+P8ACuwooA4//hCL3/ocfEH/AH/H+FH/AAhF7/0OPiD/AL/j/CuwooA4/wD4Qi9/6HHxB/3/AB/hR/whF7/0OPiD/v8Aj/CuwooA4/8A4Qi9/wChx8Qf9/x/hR/whF7/ANDj4g/7/j/CuwooA4//AIQi9/6HHxB/3/H+FH/CEXv/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5X/hA9C/6Aun/wDfof4Uf8IHoX/QF0//AL9D/CuqooA5X/hA9C/6Aun/APfof4Uf8IHoX/QF0/8A79D/AArqqKAOV/4QPQv+gLp//fof4Uf8IHoX/QF0/wD79D/CuqqpqmpWujaVd6leyeXbWsTSyN7AZ496APHvGvhPTdc8V6V4I0mxtbaSVTe6lcQRgNDbrwBnHBZv/Zexq38N9A028sb7QNX0qxbWtDnNtcM0QzKnWOT6Ed/bPeuh+Fum3U9jf+MNVj26n4hl+0BT/wAsrccRIPbbz9CPSqnjkHwd400nx1CCtlKRp2sAdPKY/JKf9045/wB0UAdD/wAIHoX/AEBdP/79D/Cj/hA9C/6Aun/9+h/hXUghlDKQQRkEd6WgDlf+ED0L/oC6f/36H+FH/CB6F/0BdP8A+/Q/wrqqKAOV/wCED0L/AKAun/8Afof4Uf8ACB6F/wBAXT/+/Q/wrqqKAOV/4QPQv+gLp/8A36H+FH/CB6F/0BdP/wC/Q/wrqqKAOV/4QPQv+gLp/wD36H+FH/CB6F/0BdP/AO/Q/wAK6qigDmrXwdpNlcpc2ul2UMycrIkYBX6HFdBbxtFFtbGc9qlooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigArzv4Sf8e/jD/saL3+SV6JXnfwk/49/GH/Y0Xv8AJKAPRKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAK5vxz4T/wCE18MS6IdRlsUlkR3kjQNuCnO0jIyM4PXsK6SigDxfxj4Y8W+EfBl/q8HxC1OYWUa7IBAqAgsFxkHjGa0ZPhl4i1/QhBqHxD1Ga1u4VMsD2qFSCAcferpfixGsnws8QK3QW278QwI/lXQeHnaTwzpUjnLNZwkn1JQUAHh/Sn0Pw/YaXJeSXjWkKw+fIMM4UYGR9OK0qKKACiiigAoo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sKK4/wD4WHZf9APxB/4Lz/jR/wALDsv+gH4g/wDBef8AGgDsKK4//hYdl/0A/EH/AILz/jR/wsOy/wCgH4g/8F5/xoA7CiuP/wCFh2X/AEA/EH/gvP8AjR/wsOy/6AfiD/wXn/GgDsKK4/8A4WHZf9APxB/4Lz/jR/wsOy/6AfiD/wAF5/xoA7CiuP8A+Fh2X/QD8Qf+C8/40f8ACw7L/oB+IP8AwXn/ABoA7CiuMm+JOm28TSz6RrsUSDLO9iQqj3JNdfbzpdW0VxESY5UDqSMcEZFAElFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFMl/wBTJ/umn0yX/Uyf7poA5L4W/wDJONJ/7bf+jnrsK4/4W/8AJONJ/wC23/o567CgAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACvO/jj/ySTV/9+D/0cleiV538cf8Akkmr/wC/B/6OSgD0SiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACvO/hJ/x7+MP+xovf5JXoled/CT/AI9/GH/Y0Xv8koA9EooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigArjPDP8AyUTxr/vWf/otq7OuM8M/8lE8a/71n/6LagDs6KKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACuM+HP/AB4a5/2Grn/2WuzrjPhz/wAeGuf9hq5/9loA7OiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAOP+Kn/JL/EP/Xof5itzw1/yKukf9eUP/oArD+Kn/JL/ABD/ANeh/mK3PDX/ACKukf8AXlD/AOgCgDUooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigDm/H//ACIes/8AXuf5itXRP+QBpv8A16xf+gisrx//AMiHrP8A17n+YrV0T/kAab/16xf+gigC/RRRQAUUUUAFFFFABRRRQAUUUUAFFUNb1BtK0LUNQRBI9rbvMqMcBiqk4P5VwFt438YXdpDcx6do/lyosi5eTOCMjvVwpyn8KuROpGHxOx6dRXm3/CX+M/8AoH6L/wB9yUf8Jf4z/wCgfov/AH3JV/V6v8rM/rNL+ZHpNFebf8Jf4z/6B+i/99yU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4pzPB/divV++vtvHPuSB2oA9MrmfH3if/hE/CN3qEQ33r4gsogMmSd+EAHfHXHoDXTV5n/yPPxc/v6L4U/75lvm/nsA/Aj3oAx/BFhf/AAx8X2Wh6rdPPaeI7cS+a5yI79R+8TP+0D17/LXslcp8Q/DMnijwlPb2hKanasLuwlHBSdOVwe2eR+NWvBHiaPxd4SstWACTuvl3MXQxzLw647c8j2IoA6GiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigDjfG/gW48X32kXcGuzabJpkjSxBIFlBkOMPhj1GOOtcd4r03xr4dn0GNfiDez/wBp6pFYEmyiXyw4Pze+MdK9jrz/AOJ3/H54I/7GW1/k9AFDVfhZr+uJbJqnj+9uFtp1uIf9AiUpIvRgQcg816bIrNEyq5RipAYDJB9adRQB5fo/ws8QaBayW2lfEC8toZZWmcCwiYu7dWJJJJOBWZ4Q03xr4lg1eRviDe2/2DVJ7AAWUT7xGR83PTOelex15/8ACn/jz8V/9jLe/wA1oA0PAvgebwa+rPNrU2pNqU4uJN8CxKsnO5gFOMtkZ/3RTfG/gW48X32kXcGuzabJpkjSxBIFlBkOMPhj1GOOtdlRQB454r03xr4dn0GNfiDez/2nqkVgSbKJfLDg/N74x0rU1X4Wa/riWyap4/vbhbadbiH/AECJSki9GBByDzV/4nf8fngj/sZbX+T16BQA2RWaJlVyjFSAwGSD615jo/ws8QaBayW2lfEC8toZZWmcCwiYu7dWJJJJOBXqFFAHjnhDTfGviWDV5G+IN7b/AGDVJ7AAWUT7xGR83PTOeldn4F8DzeDX1Z5tam1JtSnFxJvgWJVk53MApxlsjP8Auis/4U/8efiv/sZb3+a16BQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFeffFBgt34IJ/6Ga0H6NXoNed/FX/j48D/9jRZ/zagD0SiiigArz74TsDaeLAO3ia9B/NK9Brzv4Sf8e/jD/saL3+SUAeiUUUUAeffFBgt34IJ/6Ga0H6NXoNed/FX/AI+PA/8A2NFn/Nq9EoAKKKKAPPvhOwNp4sA7eJr0H80r0GvO/hJ/x7+MP+xovf5JXolABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAV538Vf+PjwP/2NFn/Nq9Erzv4q/wDHx4H/AOxos/5tQB6JRRRQAV538JP+Pfxh/wBjRe/ySvRK87+En/Hv4w/7Gi9/klAHolFFFAHnfxV/4+PA/wD2NFn/ADavRK87+Kv/AB8eB/8AsaLP+bV6JQAUUUUAed/CT/j38Yf9jRe/ySvRK87+En/Hv4w/7Gi9/kleiUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUVz3jPX7jw34fa/tYIppjKkSrKTt+Y4ycUAdDRXm3/CX+M/8AoH6L/wB9yUf8Jf4z/wCgfov/AH3JW31er/KzD6zS/mR6TXnfxV/4+PA//Y0Wf82qL/hL/Gf/AED9F/77krn/ABRc+L/EcmivJaaRH/Zmpw367ZH+Yx54Oe3NH1er/Kw+s0v5ke00V5t/wl/jP/oH6L/33JR/wl/jP/oH6L/33JR9Xq/ysPrNL+ZHpNed/CT/AI9/GH/Y0Xv8kqL/AIS/xn/0D9F/77krn/Ctz4v8Mx6ukdppEn9oanNftvkf5TJt4GO3y0fV6v8AKw+s0v5ke00V5t/wl/jP/oH6L/33JR/wl/jP/oH6L/33JR9Xq/ysPrNL+ZEvxV/4+PA//Y0Wf82r0SvFvFFz4v8AEcmivJaaRH/Zmpw367ZH+Yx54Oe3NdB/wl/jP/oH6L/33JR9Xq/ysPrNL+ZHpNFebf8ACX+M/wDoH6L/AN9yUf8ACX+M/wDoH6L/AN9yUfV6v8rD6zS/mRL8JP8Aj38Yf9jRe/ySvRK8W8K3Pi/wzHq6R2mkSf2hqc1+2+R/lMm3gY7fLXQf8Jf4z/6B+i/99yUfV6v8rD6zS/mR6TRXm3/CX+M/+gfov/fclMl8Z+MYonkbT9G2opY4eTt+NH1er/KP6xS/mR6ZRWR4Y1h9e8OWOpSokctxHuZEPAOSOPyrXrE2CiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACuJ+Kn/Inr/1+Q/+hV21cT8VP+RPX/r8h/8AQqa3E9jLooor6M+aCiiigAooooAKKKKACiiigAooooAKKKKACiiigAqC9/48Lj/rk38jU9QXv/Hhcf8AXJv5GlLYcd0b3w5/5FHSf+uJ/ma7OuM+HP8AyKOk/wDXE/zNdnXzh9KFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFcT8VP+RPX/r8h/8AQq7auJ+Kn/Inr/1+Q/8AoVNbiexl0UUV9GfNBRRRQAUUUUAFcj4b8S3t7rN1Y6mIlWWSY2DouN6xyMjof9oYU/Rq66uGj0m7uPCslxaRlNUsdRubq03DBYiZ/l7cOpI/EGsqjkmmjWmotNM2/D+uG80eO61CWNJJbyW2jwMBiJGVR9cCtC+v4Y1urVJ1F4lq04jB+YLyA30yK4jTBc/8ITpd69lcKIdXN1NF5ZLpGZnyduMnG7PA6VotKb/xjqV3BDP9n/sXyUlaIqHYOxO3PX7386iNR8qXoaSprmb9Q0zV7+a48FrJcuwvbGaS5Bx+8YIhBP4k/nXaVwekW063PgMtBIPK0+cSZQ/ITHHgH0rvKui3Z3/rREVkk1bz/NhRRRWpiFFFFABUF7/x4XH/AFyb+RqeoL3/AI8Lj/rk38jSlsOO6N74c/8AIo6T/wBcT/M12dcZ8Of+RR0n/rif5muzr5w+lCiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACuJ+Kn/ACJ6/wDX5D/6FXbVxPxU/wCRPX/r8h/9CprcT2Muiiivoz5oKKKKACiiigAooooAKKKKACiiigAooooAKKKKACoL3/jwuP8Ark38jU9QXv8Ax4XH/XJv5GlLYcd0b3w5/wCRR0n/AK4n+Zrs64z4c/8AIo6T/wBcT/M12dfOH0oUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAVxHxVIHg0EnAF3Dkn/ert6r31haalata31tFcQMQWjlUMpxyODQgZ5l/amn/8/wDa/wDf5f8AGj+1NP8A+f8Atf8Av8v+NdbJ4F0BpGKaLp4U9B5I/wAKb/wgehf9AXT/APv0P8K9H+0Jfynnf2dH+Y5T+1NP/wCf+1/7/L/jR/amn/8AP/a/9/l/xrq/+ED0L/oC6f8A9+h/hR/wgehf9AXT/wDv0P8ACj+0Jfyh/Z0f5jlP7U0//n/tf+/y/wCNH9qaf/z/ANr/AN/l/wAa6v8A4QPQv+gLp/8A36H+FH/CB6F/0BdP/wC/Q/wo/tCX8of2dH+Y5T+1NP8A+f8Atf8Av8v+NH9qaf8A8/8Aa/8Af5f8a6v/AIQPQv8AoC6f/wB+h/hR/wAIHoX/AEBdP/79D/Cj+0Jfyh/Z0f5jk21bTUUs2oWiqBkkzKAP1pRqunEZF/akH/psv+NZfjnwtpuqeItH8E6Tp9rb3N6ftV9PBGA0Fqh9ccFjwPp71L8OfD2myJqfhfWNLsZNY0OfynkeIZnhbmOT3yP6Z60v7Ql/KH9nR/mL/wDamn/8/wDa/wDf5f8AGj+1NP8A+f8Atf8Av8v+NdX/AMIHoX/QF0//AL9D/Cj/AIQPQv8AoC6f/wB+h/hT/tCX8of2dH+Y5T+1NP8A+f8Atf8Av8v+NH9qaf8A8/8Aa/8Af5f8a6v/AIQPQv8AoC6f/wB+h/hR/wAIHoX/AEBdP/79D/Cj+0Jfyh/Z0f5jlP7U0/8A5/7X/v8AL/jR/amn/wDP/a/9/l/xrq/+ED0L/oC6f/36H+FH/CB6F/0BdP8A+/Q/wo/tCX8of2dH+Y5T+1NP/wCf+1/7/L/jUN5qdg1jcAX1sSY2AAlX0+tdj/wgehf9AXT/APv0P8KP+ED0L/oC6f8A9+h/hSeYSa+EFl8U/iK3w5/5FHSf+uJ/ma7Os/T9NTT0jihijigjXaiRjAUewrQrzz0QooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAMjxP4isvCnh281q/LGC2XOxcbnYnCqPckipPDuvWfibw/ZazYMTb3Ue9Q3VT0Kn3BBB+lcNr3/ABXHxRsfDq/PpHh/bf6j/dkuD/qoz9Ov/fQ7U7wuv/CEfETUPCbnbpWsbtR0rsEf/lrCPpwwHYD3oA9KoorF8W+Irfwp4Xv9aucFbaMlEJ/1jnhV/EkCgDOj+IGjSfEKTwYrP9vSHzPM42F8bjGO+7b83511deMn4ealD8NotaTP/CZxXR1xpSvztMfmaE+23Ax03D3r1Hw1r1r4n8OWGs2Z/c3cQfbnOxujKfcEEfhQBq1keJ/EVl4U8O3mtX5YwWy52Ljc7E4VR7kkVr15nr3/ABXHxRsfDq/PpHh/bf6j/dkuD/qoz9Ov/fQ7UAdz4d16z8TeH7LWbBibe6j3qG6qehU+4IIP0rTrzXwuv/CEfETUPCbnbpWsbtR0rsEf/lrCPpwwHYD3r0qgArlI/iBo0nxCk8GKz/b0h8zzONhfG4xjvu2/N+daPi3xFb+FPC9/rVzgrbRkohP+sc8Kv4kgV5mfh5qUPw2i1pM/8JnFdHXGlK/O0x+ZoT7bcDHTcPegD2aisrw1r1r4n8OWGs2Z/c3cQfbnOxujKfcEEfhWrQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFV7++t9M0+5v7uQR21tE0srn+FVGSasVz3jXwwfGPhi40T+0JbFJ2UvLGgYkA52kZHBwO9AHO/C+xuNSXUvHGpxlb7Xpd0CN1htF4jUfUDPv8pqD4go/hTxPpHj+2U+RCRYauqj71s5+Vz/utj6/L6VieJ/CPivwl4MvtStviFqciWFvmO3FuqLgYAUEHgYq3D8OvEviPw3D9u+IepSW9/aq0sD2qspDqCR973oA9YR0ljWSNg6MAyspyCD0Ip1ZXhrRm8PeG7DR3vJLz7HEIhPIuCwHTjtgYH4Vq0AFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAVi+LfEVv4U8LahrVxgrbREohP33PCr+LECtqq97YWepWj2l/aQXVs+N8M8YdGwcjKng80AeW/DPxL4U0Lwt9o1TxPpZ1rVJWvtQZrld3mOchT6YGBj1zR8SfE3hbWvDiXmkeKNLGuaTKL2wZbldxdeqdedw4x3OK2/HPgzwtaeAvEFzbeGtHhni0+d45Y7GJWRghIIIXII9af4M8GeFbrwL4euLjwzo008umWzySSWETM7GJSSSVyST3oA6Xwt4gtvFPhmw1q14juogxXOdjDhl/BgR+FebeN/Eej6v8UtJ8O6rqVraaPoxF/em4lCLNPj93Hz1xkEj0J9K9Zs7K0060jtLG1gtbaPISGCMIi5OeFHA5rOvfCfhvUrt7u/8P6VdXL43zT2Ubu2BgZYjJ4oAz/+Fj+Cv+ho0r/wJX/GuK8CeItH0n4k6r4Z0vVLW80fVc3+n+RKHWGU5MsXHTOCwHoPenR+FPDh+PEunHQNK+wjw55wtvscfl+Z9oA37cY3Y4zjOK9EsfCnhzS7tLvT9A0q0uUyFmt7OON1yMHDAA9KAGeLfEVv4U8LahrVxgrbREohP33PCr+LECvP/hn4l8KaF4W+0ap4n0s61qkrX2oM1yu7zHOQp9MDAx65r1K9sLPUrR7S/tILq2fG+GeMOjYORlTwea4jxz4M8LWngLxBc23hrR4Z4tPneOWOxiVkYISCCFyCPWgDE+JPibwtrXhxLzSPFGljXNJlF7YMtyu4uvVOvO4cY7nFeh+FvEFt4p8M2GtWvEd1EGK5zsYcMv4MCPwrmvBngzwrdeBfD1xceGdGmnl0y2eSSSwiZnYxKSSSuSSe9drZ2Vpp1pHaWNrBa20eQkMEYRFyc8KOBzQB5N438R6Pq/xS0nw7qupWtpo+jEX96biUIs0+P3cfPXGQSPQn0ruP+Fj+Cv8AoaNK/wDAlf8AGtC98J+G9Su3u7/w/pV1cvjfNPZRu7YGBliMnivPI/Cnhw/HiXTjoGlfYR4c84W32OPy/M+0Ab9uMbscZxnFADfAniLR9J+JOq+GdL1S1vNH1XN/p/kSh1hlOTLFx0zgsB6D3r12six8KeHNLu0u9P0DSrS5TIWa3s443XIwcMAD0rXoAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigDj/ip/wAkv8Q/9eh/mK3PDX/Iq6R/15Q/+gCsP4qf8kv8Q/8AXof5itzw1/yKukf9eUP/AKAKANSiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigDm/iD/wAk58Sf9gy4/wDRZqTwJ/yTzwz/ANgq1/8ARS0nj1Q/w78Sg/8AQLuT+UTGl8Cf8k88M/8AYKtf/RS0AdBRRRQB53H/AMnEzf8AYr/+3Ir0SvPgoX9odiP4vCuT/wCBWP6V6DQAVzfxB/5Jz4k/7Blx/wCizXSVz3j1Q/w78Sg/9Au5P5RMaAF8Cf8AJPPDP/YKtf8A0UtdBXP+BP8Aknnhn/sFWv8A6KWugoAK87j/AOTiZv8AsV//AG5FeiV58FC/tDsR/F4Vyf8AwKx/SgD0GiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAOP8Aip/yS/xD/wBeh/mK3PDX/Iq6R/15Q/8AoArD+Kn/ACS/xD/16H+Yrc8Nf8irpH/XlD/6AKANSiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigDn/AB3/AMk88Tf9gq6/9FNSeAmD/Dvw0R/0C7YflEopfHf/ACTzxN/2Crr/ANFNUfw+/wCSc+G/+wZb/wDosUAdJRRRQB58zBf2h1H97wtj/wAms/0r0GvO5P8Ak4mH/sV//bk16JQAVz/jv/knnib/ALBV1/6Kaugrn/Hf/JPPE3/YKuv/AEU1ACeAmD/Dvw0R/wBAu2H5RKK6Gub+H3/JOfDf/YMt/wD0WK6SgArz5mC/tDqP73hbH/k1n+leg153J/ycTD/2K/8A7cmgD0SiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAOP+Kn/ACS/xD/16H+Yrc8Nf8irpH/XlD/6AKw/ip/yS/xD/wBeh/mK3PDX/Iq6R/15Q/8AoAoA1KKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAOf8d/8k88Tf9gq6/8ARTVH8Pv+Sc+G/wDsGW//AKLFSeO/+SeeJv8AsFXX/opqj+H3/JOfDf8A2DLf/wBFigDpKKKKAPO5P+TiYf8AsV//AG5NeiV53J/ycTD/ANiv/wC3Jr0SgArn/Hf/ACTzxN/2Crr/ANFNXQVz/jv/AJJ54m/7BV1/6KagCP4ff8k58N/9gy3/APRYrpK5v4ff8k58N/8AYMt//RYrpKACvO5P+TiYf+xX/wDbk16JXncn/JxMP/Yr/wDtyaAPRKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooA4/4qf8AJL/EP/Xof5itzw1/yKukf9eUP/oArD+Kn/JL/EP/AF6H+Yrc8Nf8irpH/XlD/wCgCgDUooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAOK+JV9fWel6VHY3s9o1zqMcEkkLbW2kNnn8vyrmPsWr/8AQ1a1/wB//wD61dB8Uf8Ajy0D/sMQ/wAmqjXoYKlCcW5K552NrTpySi7Gb9i1f/oata/7/wD/ANaj7Fq//Q1a1/3/AP8A61aVFdv1Wj/KcX1qt/MZF3pGoX1nPaXPiXWJbeeNopY2nyHVhgg8dCDSWejX+n2UFna+JdYit4I1jijWfAVQMADj0rYoo+q0f5Q+tVv5jN+xav8A9DVrX/f/AP8ArUfYtX/6GrWv+/8A/wDWrSoo+q0f5Q+tVv5jBPhy4OsjVz4g1b+0BB9mFx5w3eXu3bc46Z5q59i1f/oata/7/wD/ANatKij6rR/lD61W/mM37Fq//Q1a1/3/AP8A61RXekahfWc9pc+JdYlt542iljafIdWGCDx0INa9FH1Wj/KH1qt/MY9no1/p9lBZ2viXWIreCNY4o1nwFUDA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u36xS/mOH6tV/lZs0Vjf8JCP+gRq/wD4BtR/wkI/6BGr/wDgG1H1il/MH1ar/KzZorG/4SEf9AjV/wDwDaj/AISEf9AjV/8AwDaj6xS/mD6tV/lZs0Vjf8JCP+gRq/8A4BtR/wAJCP8AoEav/wCAbUfWKX8wfVqv8rNmisb/AISEf9AjV/8AwDaj/hIR/wBAjV//AADaj6xS/mD6tV/lZs0Vjf8ACQj/AKBGr/8AgG1H/CQj/oEav/4BtR9YpfzB9Wq/ys2aKxv+EhH/AECNX/8AANqP+EhH/QI1f/wDaj6xS/mD6tV/lZs0Vjf8JCP+gRq//gG1H/CQj/oEav8A+AbUfWKX8wfVqv8AKyzr3/Iv6j/17Sf+gmu18F/8i1pH/YPh/wDQFrzfVNYa80q7totI1bzJYWRd1o2MkYr0vwhHJDoGlxSoySJYxKysMFSEXIIrzsbUjOScXc9LA05Qi1JWOioooriO4KKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKrNZozsxZuTmiigBPsMf95qPsMf95qKKAD7DH/eaj7DH/eaiigA+wx/3mo+wx/3moooAPsMf95qPsMf95qKKAD7DH/eaj7DH/eaiigA+wx/3mo+wx/3moooAPsMf95qPsMf95qKKAD7DH/eaj7DH/eaiigA+wx/3mqSK2WF9wJJxjmiigCaiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooA//2Q=="  />

Fig-1. Scala core types hierarchy

</div>

---

#### 4. Scala运算符
  scala支持的运算符和优先级与其他语言相比没有特殊之处，只是**不支持三元运算符(? :)**，详细内容请参考相关材料.

---

### III. Expression and Built-in control structure
#### 1. 表达式(Expression)

  表达式是函数式程序的基础构件，因为表达式实现了函数式编程中的一个核心思想，即新值存储在新的value中，而不是修改已存在的varible，这什么意思？
  
  在探讨这个问题之前，可以先回顾一下其他编程范式中程序逻辑的实现方式，如在命令式编程中，我们首先会抽象出一系列处理逻辑，然后将变量按照特定顺序依次通过，最后变量中的值就是程序的功能体现，如简单的自增运算`x = x + 1`. 
  
  相比之下，函数式编程则采用了完全不同的实现方式，即针对问题建模，其首先构建出类似数学上的函数方程来描述功能逻辑，之后将函数方程分解成一系列的低阶子函数的运算链条，最后通过求解子函数链条得到最终的功能输出，而在数学中，函数的定义是`f: x -> y`，是描述一个集合到另一个集合的"映射"，例如，集合`A`中的元素`a`在法则`f`作用下，映射到集合`B`中的元素`b`上，显然，在得到输出`b`的同时，输入`a`并未发生改变，所以，`x = x + 1`在数学上是不成立的. 绕了这么大个圈子，就是要说明scala中的表达式描述了这种"映射"关系，其根据输入val产生新的输出val，而非修改已存在的值. 当然，scala也支持面向对象，所以我们依然可以用表达式实现命令式编程逻辑.

##### 定义表达式
  表达式就是个具有返回值的代码单元，表达式的返回值可作为另一个表达式的输入值或保存在value/variable中，由此可以重新定义value和variable的语法，即
  
```scala
//syntax

val <identifier> [: <type>] = <expression> 
var <identifier> [: <type>] = <expression> 
```

   多条表达式可以构成一个表达式块，内部可以定义局部val或var，表达式块的最后一个表达式为整个表达式块的返回值; 表达式块有如下两种常用的表示形式：
  
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
//syntax: 使用关键字to和until，使用to则取值范围包含结尾的<ending integer>，until则不包含结尾元素，可选的数字间隔参数[by increment]

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
    
* Iterator guard：上例我们已经见识了加`guard`的迭代器了，实际上，就是加了迭代条件，这和C++,Java中`for(init ; condition ; changed value)`的条件检查一致. 与`match`类似，这里`if`后条件表达式的括号也是可选的.
   
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

  函数字面量简化掉了命名函数的定义过程，占位符则是在**函数字面量**的基础上，进一步简化掉了参数列表，占位符使用下划线`_`表示，所谓"占位"是用下划线按位置顺序占据参数列表中参数的位置，以达到替代表示的目的，占位符的使用需满足两个条件：
  
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

* Partially Applied Functions：除了具有默认值参数的函数，大多数语言在调用函数时，都要提供完整的参数，即使是具有默认值的函数，也只能使用函数定义时指定的一个特定默认值，那么函数的灵活性就局限在了被调函数. 

如果主调函数是多参数的，且在一些应用场景下，部分参数为常数或测试时要做增量测试，即灵活性需要掌握在主调函数手中，那么现有大多数语言都是无法满足这个需求的. 但Scala中的部分参数调用机制是可以实现这种逻辑的，这也许得益于其对函数式的支持，因为数学上这种情况非常多(作者本科数学专业，所以有些了解)，例如数学分析中的多元函数，无论求解极限还是微积分都是要将多元转化一元来实现，所以作为函数式语言，部分参数调用机制是十分必要的.

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

  `by-name parameters`主要用来优化高阶函数参数，调用高阶函数时，这类参数能够接受常规类型`value`和函数类型值，这种形式的参数进一步提升了高阶函数的灵活性，但有一个潜在的**性能风险**：如果调用时传递的是函数类型值，那么高阶函数体内每次访问对应参数时会反复调用，所以可能会带来不一致的问题，而且对执行代价比较大的函数，如数据库查询操作，会带来性能下降问题，所以在定义这类参数时要注意或减少对这类参数的访问.

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
