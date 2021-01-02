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
    
<img src="data:image/jpg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD/2wBDAAgGBgcGBQgHBwcJCQgKDBQNDAsLDBkSEw8UHRofHh0aHBwgJC4nICIsIxwcKDcpLDAxNDQ0Hyc5PTgyPC4zNDL/2wBDAQkJCQwLDBgNDRgyIRwhMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjIyMjL/wAARCAGZAfgDASIAAhEBAxEB/8QAHwAAAQUBAQEBAQEAAAAAAAAAAAECAwQFBgcICQoL/8QAtRAAAgEDAwIEAwUFBAQAAAF9AQIDAAQRBRIhMUEGE1FhByJxFDKBkaEII0KxwRVS0fAkM2JyggkKFhcYGRolJicoKSo0NTY3ODk6Q0RFRkdISUpTVFVWV1hZWmNkZWZnaGlqc3R1dnd4eXqDhIWGh4iJipKTlJWWl5iZmqKjpKWmp6ipqrKztLW2t7i5usLDxMXGx8jJytLT1NXW19jZ2uHi4+Tl5ufo6erx8vP09fb3+Pn6/8QAHwEAAwEBAQEBAQEBAQAAAAAAAAECAwQFBgcICQoL/8QAtREAAgECBAQDBAcFBAQAAQJ3AAECAxEEBSExBhJBUQdhcRMiMoEIFEKRobHBCSMzUvAVYnLRChYkNOEl8RcYGRomJygpKjU2Nzg5OkNERUZHSElKU1RVVldYWVpjZGVmZ2hpanN0dXZ3eHl6goOEhYaHiImKkpOUlZaXmJmaoqOkpaanqKmqsrO0tba3uLm6wsPExcbHyMnK0tPU1dbX2Nna4uPk5ebn6Onq8vP09fb3+Pn6/9oADAMBAAIRAxEAPwD3+iiqrXhV2UR5wcdaALVFVPtp/wCeR/P/AOtR9tP/ADyP5/8A1qALdFVPtp/55H8//rUfbT/zyP5//WoAt0VU+2n/AJ5H8/8A61H20/8API/n/wDWoAt0VU+2n/nkfz/+tR9tP/PI/n/9agC3RVT7af8Ankfz/wDrUfbT/wA8j+f/ANagC3RVT7af+eR/P/61H20/88j+f/1qALdFVPtp/wCeR/P/AOtR9tP/ADyP5/8A1qALdFVPtp/55fr/APWo+2n/AJ5H8/8A61AFuiqn20/88j+f/wBaj7af+eR/P/61AFuiqn20/wDPI/n/APWo+2n/AJ5H8/8A61AFuiqn20/88j+f/wBaj7af+eR/P/61AFuiqn20/wDPI/n/APWo+2n/AJ5H8/8A61AFuiqn20/88j+f/wBaj7af+eR/P/61AFuiqn20/wDPI/n/APWo+2n/AJ5H8/8A61AFuiqn20/88j+f/wBaj7af+eX6/wD1qALdFVPtp/55H8//AK1H20/88j+f/wBagC3RVT7af+eR/P8A+tR9tP8AzyP5/wD1qALdFVPtp/55H8//AK1H20/88j+f/wBagC3RVT7af+eR/P8A+tR9tP8AzyP5/wD1qALdFVPtp/55H8//AK1H20/88j+f/wBagC3RVT7af+eR/P8A+tR9tP8AzyP5/wD1qALdFVPtp/55H8//AK1H20/88j+f/wBagC3RVT7aSMiL9f8A61H20/8API/n/wDWoAt0VU+2n/nkfz/+tR9tP/PI/n/9agC3RVT7af8Ankfz/wDrUfbT/wA8j+f/ANagC3RVT7af+eR/P/61H20/88j+f/1qALdFVPtp/wCeR/P/AOtR9tP/ADyP5/8A1qALdFVPtp/55H8//rUfbT/zyP5//WoAt0VU+2n/AJ5H8/8A61H20/8API/n/wDWoAt0VT+3Y6xf+PVbRtyK2MZGaAFooooAKzW/10n+8a0qzW/10n+8aAMzU9fsNIuYLa6N0086M8cdtZzXDFVIDHEatgAsvX1qfTdUs9Wtmns5HZVcxuskTRujDqrI4DKehwQOCD0NYWt6ZfX/AIw0x7S9vbBI7C5Vrq2ijYBi8OFPmIy84Jxwfl69a5u503Uj4aa2vLeaa8j1VZNXlnsHuo71dpCyLCm3zE4i+RPu7ec4ORbf13sNr+vkem0V5he6Uf8AhGtIhvLaa5WNbjyYrjQZbiBVZvkUwJIZImC4CMT8q7gcE4q3r1jPcKjX2j3Mt2+kxx6Yscb3Bs7z5t370A+W2TF+8Yj7h54NOwj0SivPL6K8iu9RsHsb2W4udZsbtJIrV2iMa/Zw7GQDaMFGyM54zjHNNj0OODSUuLzR3lgl1i5k1OIWrSSTw+ZN5W6MKWkQM0bAAHjnGKOl/wCun+f4B0/rz/yPRaK8r1zSJLie0Njp9za2H2PZpscuky3UttL5rHdHiRfszYMZUvgAAA7dpFepruCKGOWxycYzR0uHWwtFFFIAooooAKKKKAGt95Pr/Q1DeX1tp8KzXUnlxvKkSnaTl3YKo49SQKmb7yfX+hrG8V2txdaH/o0LTSQXNvc+Un3nWOVHYD3wpwO5o66gW9Q1vTtKZlvbjyitvJdH5GbEUe3e3APTcvHXninyatZxXEUEjyJJNMYIg0LgO+wyYU4wRtB5HGQRnPFcV4otz4sg1Caz028lhg0e6gC3VjJCZJnMbIESRQzEeXnIGM4wc1HZ6bMNVhktdNuIbMav5sS/ZWiCRf2dsB2kDaN3y8gYPHXij7N/L9R2X5/kehQzLcQRzIHCyKGAkRkYA+qsAQfYjNMvr2306wuL27k8u3t4zJK+0naoGScDk/hXm+jW2mxzeHzqfh+7jvNOsLf9+NGmeSSbywAplWM4VB2J+8e22subSLi/h1KOHw+YRc6RdLNbrpkyN9oyjIJZpOLlwQxVwOucE5pta29QirvXyPXop0naQIJP3bbSWjZQeAeCR8wweoyM5HUGpa84GnL9mujJpM7aGdWhllsxYv8AvLcWkariHbuZVkCZULxsPHBFLZ+HRqF3psWpaS8umpa37QQXMJKRI00ZgRlIwpCfdUjK4xgFeE9P68riWqR6NRWR4WW5Xwlo63iyrdCyiEqyghw2wZ3A85z61r05KzaEtgooopDCiiigApq/ef6/0FOpq/ef6/0FAEF1qFrZSW0dxKI2uZfJiBBO5sE446dOp9h1IqzXHeI9G1nXNUuJLC4t7VLO3EcBuLV5C8pZZNyEOoGCkY3YbnPHBrOji1G5luF/se9ayggl1OCEloGeWaPHkhuCrhmmz3G5DS6f1/Xcdtf6/rsd9NMkCB3EhBZV+SNnOScDhQTjnk9AOTgVHZX1tqEDT2snmRrI8RO0j5kYqw59GBFeYWOn3Ilu0s9LMNm9xpkiR2ukTWUYZLnMhKScswXbl8DIA9Kt3djJ/Y9vbXuiiZWvb+QSXenTXscZNwxTNvHgkspJEhwFAOD81N6K/wDXQP6/Bnov2uA3xst/+kCMSlMH7pJGc9OoNJbX1tdzXUMEm+S1l8qYbSNr7VbHPXhlPHrXmk2nmfStOOq6JqF3f/2FHBbsbWRzFdjOMnB8t88hyRt55GebOqaXqj2mpia3LwtrMMt0sti90k8Qto1J8lSplUSBchT/AAk84Io/r8bflqH9fhc9KorD8IW01p4ZtYZjJ8pcxq9sbcpGXJRRGWYqoXAAJyAACAeK3KbVmJBRRRSAKKKKACiiigBsf+rX6CqJ1vThpNzqhuP9CtvNE0mxvl8slX4xk4KnoOccVej/ANWv0FcFcLPF4Y1rww1jftf3c12sDR2kjQus0jsrmUDy1AD8hmBGDx0yDVup2Mer2Us0sMckjvDN5Mu2FyEfYJOTjAG0jnpkgZzxT7DU7XU4Y5rRpJIZYUnjkMTqrI+dpBIAzxyOo4yBkVwSaK1r4xmeDSpI2bWPPeeO1IV4zYuobeBgjzC/fhm5+9zRt7KS00aOPUdBnulk0fTbZUm0x7lYpR525mjCsSUByRjqQOM0+l/T8Qt/XzPVqpx6paTXUttE8jyxT/Z5AsTkI+wPgnGANpHJ4yQM54rzLxDY2smjpZaLoM4W1sDHYz3OjXUj+YC2RGuF+zvna3mkfMSODt40rPSWh8ZG4g0mWB5NYFy84tGUGNrFgGZ8YP7wvkZyGbnBbkS3/rqLp/XY9Horyk6Vdf2FfR2WlXkOoDR7mLVXFq8Zu7o42kHH75iwkIZScBu24Cuv0DSY9I8TanFZWP2Swks7ZwI49qPLmUOc93xsyevTNFgen9en+Zop4m0hzfAXZH2FWectE6japIJUkYcAgglc88da1VYOisM4IyMgg/kelcBrnhy4t7q5fTWv5Eija5IKBlQmQuI4lC/Od5aTB3HKIvRsV1Phqe+uNGWS/M7SebII5LiHypZIw5CM6YG1iuCRtX6DpSW39f1/XUHua9FFFABRRRQAyX+H6/0NaUX+pj/3RWbL/D9f6GtKL/Ux/wC6KAH0UUUAFZ21mml2qT8x6CtGq1t/rp/97/GgCDy3/uN+VHlv/cb8qne/to9Sh09pcXU0TzRx7TyiFQxz04Lr+dCX9tJqM2npLm6gjSWSPaflVywU56clG/KgCDy3/uN+VHlv/cb8qz7vxrodlEssst60TStCJIdOuJV8wSGMrlEIDbwQB34xnIrZs7uK/tI7mFZ1jfoJ4Hhcc45RwGH4igNtCt5b/wBxvyo8t/7jflWhRQBn+W/9xvyo8t/7jflWhRQBn+W/9xvyo8t/7jflWhRQBn+W/wDcb8qPLf8AuN+VaFFAGf5b/wBxvyo8t/7jflWhRQBmtG+5Pkbr6exp3lv/AHG/Krr/AH4/97+hp9AGf5b/ANxvyo8t/wC435VoUUAZ/lv/AHG/Kjy3/uN+VaFFAGf5b/3G/Kjy3/uN+VaFFAGf5b/3G/Kjy3/uN+VP1LWNP0hbc6hdxwfaZ0t4Ax5kkc4VVA5J/kMk8Cq0/iXSbbVl0yW5dblnWMkQSGNXYZVGkC7FYjGFLAnIwORkAm8t/wC435UeW/8Acb8qedY08a0uj/a4zqLQG4+zg5YRggbj6DJGM9ecdDUOm+IdM1a5lt7K4Z5Ixu+aF0Ei5xujLACRc8blJHI55FAD/Lf+435UeW/9xvyqAeKNFaz1C7TUIpLfT5vs9y8YLBZfl+QYHzN8yjC5OTjrxTP+Er0f+zP7QE1wYvP+zmNbOYzCX+4YQnmBsc4K9OenNAFry3/uN+VNWN9z/I3X09hVqxvYtQtEuoUnSN84E9u8L8HHKOAw/EVKn35P97+goApeW/8Acb8qPLf+435VoUUAZ/lv/cb8qPLf+435VoUUAZ/lv/cb8qPLf+435VoUUAZ/lv8A3G/Kjy3/ALjflWhWRH4n0iXVv7NS4k+0GRolY28gieRclkWUrsZhg5UMSNp44NAE/lv/AHG/Kjy3/uN+VVbXxXpF4s8kcl0kEEbSyXM9lNFBtU4JEroEb8Cc9aWHxVo81jdXnnzxR2pUTJcWssMoLfdxG6hzuJwuAdx4GTQBZ8t/7jflR5b/ANxvyqn/AMJdov2Brz7ROFWcW5hNpMJ/NIyE8nb5m7HzY29OenNSS+KdIi0y31H7RLLb3GfKEFtLLIcfe/dopcbejZHyng4NHmBY8t/7jflR5b/3G/Kq914p0azNr5t5lbmNZUeKJ5EWM4Ad2UEIhzwzEDrzwa2KAuZscb+UvyN0HaneW/8Acb8quw/6iP8A3R/Kn0AZ/lv/AHG/Kjy3/uN+VaFFAGf5b/3G/Kjy3/uN+VaFFAGf5b/3G/Kjy3/uN+VaFFAGf5b/ANxvyo8t/wC435VoUjsERnIJCjJ2gk/gByaAKHlv/cb8qPLf+435VVsPFOl6lqS6dCL+O6aNpVjudOuLfKKQCQZEUcEjv3q7p2safqzXY0+7jufsk5t5zGchJAASuehI3DOOh46g0AM8t/7jflR5b/3G/Kn3GrWNtqMOnyzgXk0TzRwqpZmRMbjgD3H17VSsPFOl6lqS6dCL+O6aNpVjudOuLfKKQCQZEUcEjv3o3AmmRgFJUgZ7j2NaMX+pj/3RUN9/qV/3v6Gpov8AUx/7ooAfRRRQAVWtv9dP/vf41ZqpA6pNPuYLlu5+tAHP65dDTfG2lahNa30lqthdQtJa2U1xtdnhIBEasRkK3X0qIaklj4nn1uSz1NtP1CwhiieLTp3kR43lyrxKhdMiQEFlA4PtnrfOi/56J/30KPOi/wCeif8AfQo6W/rr/mN6/wBf12OHbS75PBNpEbSfz5dajvDDsy8cb3wl+YDOMKcn0wfSu7pnnRf89E/76FHnRf8APRP++hT6W/rp/kLfUfRTPOi/56J/30KPOi/56J/30KQD6KZ50X/PRP8AvoUedF/z0T/voUAPopnnRf8APRP++hR50X/PRP8AvoUAPopnnRf89E/76FHnRf8APRP++hQA+imedF/z0T/voUedF/z0T/voUAD/AH4/97+hp9QvLGWj/eJw3Pzexp/nRf8APRP++hQA+imedF/z0T/voUedF/z0T/voUAPopnnRf89E/wC+hR50X/PRP++hQA+imedF/wA9E/76FHnRf89E/wC+hQBznjHSo7u1s7qGwWa+jvrMCVId0ixC5jZhkDIXjJ7cZrn9Ut9QbxTdRJbagTLqlrPFaR2zmzuI18vdNJMBhHXa2F3qMxplGz83ofnRf89E/wC+hR50X/PRP++hTi7f16f5A9Uc1PpCQ+PLW6s7FYEm0+7+0XEMIUGVngwWYDliFPXk7fasrw1Zz3d5ocE+m3ECaTpMthei5t2RHkYxLtUkBZFPlscrlcEetd150X/PRP8AvoUedF/z0T/voUlorf11/wA2Ns4Q6ObPTPE8UFlc2kMWpwT2YtLTftEcVvtZIuN6AocqvJCsF5xVOPTftVvcapro1ySG41VblH061mtpDttxEGMSZnRM7gAMt0JOCa9H86L/AJ6J/wB9Cjzov+eif99Cj/gfp/kFzK8Mrcro/wC/NyYzLIbb7Vv80Q7js37/AJ92Mfe+b15zWqn35P8Ae/oKPOi/56J/30KYksYaT94nLcfN7CgRNRTPOi/56J/30KPOi/56J/30KAH0Uzzov+eif99Cjzov+eif99CgB9FM86L/AJ6J/wB9Cjzov+eif99CgB9cLaahHq3iFIJdO1LT7OxuZDZ2q6VOgml+YGZ5dnlhTuYqN3JO5jkgDt/Oi/56J/30KPOi/wCeif8AfQpWDocJ4chTSGgXR7fxJJbWto4vLfURPuZht2LGJSIy2d/+qO33xtqM6g1nc6zrkelavrEMi2/lC602SOZZldtqKnlhvLQMG3hCRk8ueB3/AJ0X/PRP++hR50X/AD0T/voU2B5+wZtMTWYptUbVl1BZ5rg6HcFFYxGPAtmCyNEFO35CSCQxP3qpw6LLa2+nahq41ryJGv2f7BHMk4aaZZELLB+8QEKSR0BwGr0zzov+eif99Cjzov8Anon/AH0KB3/r+vU84v7fWl0u9ttSsbq51LWNBhs1eGEyILkCQMrsoIjGZVbc2F+9g8V6PChjgjRm3MqgE+uBR50X/PRP++hR50X/AD0T/voU27i/r8v8gh/1Ef8Auj+VPqGKWMQoDIgIUfxU/wA6L/non/fQpAPopnnRf89E/wC+hR50X/PRP++hQA+imedF/wA9E/76FHnRf89E/wC+hQA+imedF/z0T/voUedF/wA9E/76FAD6KZ50X/PRP++hR50X/PRP++hQBhaLZPdX2t6hf253XU7WqRzRkYt48qowRyrMZH9Dvqv4dtU0W78QILGS2tG1KNbZIbZtuwwQoCqqPuggjI4GDnGDXS+dF/z0T/voUedF/wA9E/76FC0+635AcFDpviOL4hWGoXmm2UiyG5El3DdyNsi+QRqQYQFwBwu45Jc5FdFoNtNJqesardxOk09ybeESIVKwRfKoGexbe+e++tvzov8Anon/AH0KPOi/56J/30KFoD1Ib7/Ur/vf0NTRf6mP/dFV7yRGiUK6k7ux9jViL/Ux/wC6KAH0UUUAFVrb/XT/AO9/jVmq1t/rp/8Ae/xoAlNxAtyls00YndC6xFhuZQQCQOpAJGT7ihbiBrl7dZozPGqu8QYblU5wSOoBwcH2PpXKa7q2m6P4+0e41TULWxgbTbtBJdTLEpbzIDjLEDPB49qiXX9GsPGF1qt1q1jDpuoadbra3r3CCCVo5Jt6rITtLDevGf5GhbJ/11/yG9L/ANdjdu/FfhzT2Vb3X9KtmbcFE15GhO1irYyezAg+hBFRy+M/C0KRPL4l0eNZk3xl76IB1yRkfNyMgjPsa5YxungK0dlZVn1+K4j3KVJSTUA6HB5GVYH8ataiviD/AITjXJPD8+nJcppVqfKvbd5FlbfPtAZZF2d+cN19qOl/62v+oW3t/Wtjsf7QsjZR3ou7f7LKEMc/mjY+4gLhs4OSQB65FOF7am+axFzCbtYxKYBIPMCE43beuM8ZrzW3hl1zw7pmkaGkV0FFxdajHdzm1MMzlwUIRH2lZHchcHHljk8E1BrN0dfm1nA/tFNOtdPnVBuC3DvcREY4489U/Cnbt/X9MXS/9b/5anq1reW19bi4s7mG4hJKiSFw6kgkEZHHBBB9xU1ct8PreO08K/ZohiOG+vI0HsLmQCupoYdWFFFFIAooooAKKKKAGP8Afj/3v6Gn0x/vx/739DT6ACiiigAooooAKKKKACiiigAqE3dssssRuIhJCgkkQuMohzhiOwO1uT6H0qavPn0eysdc8ZW1tJJaLdaRA01yVkuZAzG4Bc5JZyBjAz0AA4FJ7DSv+H5nW2nibQdQt7m4stb025htV33EkN3G6wrzy5Bwo4PJ9DUn9vaP/ZP9rf2tYf2b/wA/n2lPJ67fv529eOvWuAvNRU6DdQ2mvLrumWxs5BqBMDR2sgmGd5hQKY0Cq7KecZyy5BDbbxHo2l2d3quoz2V7cSawTY3ccwgtLmcwKpdCSVRFXcrMWflWIJYharTX+u3+f/D9Ev6+656ZbXVve2sdzaTxT28q7o5YnDI49QRwRTk+/J/vf0FYvhFLRdCElpqFnfieeWeWeydWh8x3LMEwTwCcevc81tJ9+T/e/oKGA+iiikAUUUUAFFFFABRRTZGZY2ZULsASFBALH05oAja7tkmeJ7iJZEj810LgFU5+YjsODz7VRtPE2g6hb3NxZa3ptzDarvuJIbuN1hXnlyDhRweT6GvPLb7fNrmvjVdP1DTby+0Qm6unEU4gy0gG1I5GZlUcAAZO0kgZybs13FcaFew2/iW31jS7JLW4F9OInhjlSUHynaBAAmFUkkEoDuORQttf61f+XzG9/wCvL/M7k6/ow0kasdXsBppOBefaU8knO37+dvXjr1p1zrek2enRajdapZQWMu0x3MtwixPuGRhicHI6c1xWjanZxW+r63qN3pNrHd6jvsb0gzWcUv2dULrIdgIyGXdlNxyB15zYDHb2mkXF1ry6Vp6i/jXUgIxHO7yqysnmqyKrgOVGD8oIVj1I9P68gR6Vc6tptlNaw3WoWkEt2222SWZVaY8cICfmPI6eoq5XlOoXQTRr6HU7aKyv9T8OQQWFmE8vdMPNzFEv94M8Z2jkZX0r1OEOIIxIcyBQGPqcc02rE/1+X9fIIf8AUR/7o/lT6ZD/AKiP/dH8qfSGFFFFABRRRQAUUUUAFFFFABVTT9U0/VrdrjTb+1vYVYoZLaZZFDDqMqSM89KpeK4Z7jwhrMNqjPPJZSqiICWYlDwAO56CsTQdU0y78Tahqmn3ls2kvZWdsJ0kAjaffJhAem8B0GOvIFC1YPRX/r+v8jpotY0yfU5tMh1Kzk1CFd0tqk6mVBxyyA5A5HUdxVn7RCbk2wmj88IJDFuG4KTgHHXGQRn2riLrUNPm8WaTFpVxZTtZ3Mxk0q2j8q5ikZJA8snPCEnoVUEuG3HIBreFJNR/4WDeSano99a3t1p0clw80kDIhEj7VXZIx2AfKOM8EkDOSLVpf1s2D0v/AF1sd3ff6lf97+hqaL/Ux/7oqG+/1K/739DU0X+pj/3RQA+iiigAqtbf66f/AHv8as1UgRXmn3KGw3cfWgC3RTPJi/55p/3yKPJi/wCeaf8AfIoAfRTPJi/55p/3yKPJi/55p/3yKAH0UzyYv+eaf98ijyYv+eaf98igB9FM8mL/AJ5p/wB8ijyYv+eaf98igB9FM8mL/nmn/fIo8mL/AJ5p/wB8igB9FM8mL/nmn/fIo8mL/nmn/fIoAfRTPJi/55p/3yKPJi/55p/3yKAB/vx/739DT6heKMNH+7Tlufl9jT/Ji/55p/3yKAH0UzyYv+eaf98ijyYv+eaf98igB9FM8mL/AJ5p/wB8ijyYv+eaf98igB9FM8mL/nmn/fIo8mL/AJ5p/wB8igB9FM8mL/nmn/fIo8mL/nmn/fIoAfRTPJi/55p/3yKPJi/55p/3yKAH0UzyYv8Anmn/AHyKPJi/55p/3yKAH0xPvyf739BR5MX/ADzT/vkUxIoy0n7tOG4+X2FAE1FM8mL/AJ5p/wB8ijyYv+eaf98igB9FM8mL/nmn/fIo8mL/AJ5p/wB8igB9FM8mL/nmn/fIo8mL/nmn/fIoAfRTPJi/55p/3yKPJi/55p/3yKAH0UzyYv8Anmn/AHyKPJi/55p/3yKAH0UzyYv+eaf98ijyYv8Anmn/AHyKAH0UzyYv+eaf98ijyYv+eaf98igAh/1Ef+6P5U+oYoozChMaElR/DT/Ji/55p/3yKAH0UzyYv+eaf98ijyYv+eaf98igB9FM8mL/AJ5p/wB8ijyYv+eaf98igB9FM8mL/nmn/fIo8mL/AJ5p/wB8igB9FM8mL/nmn/fIo8mL/nmn/fIoAfRTPJi/55p/3yKPJi/55p/3yKAH0UzyYv8Anmn/AHyKPJi/55p/3yKAIb7/AFK/739DU0X+pj/3RVe8jRYlKooO7sPY1Yi/1Mf+6KAH0UUUAFVrb/XT/wC9/jVmq1t/rp/97/GgDjfF+nf2n420WD+xdK1fFhdt5GpvtjX54PmB8uT5ucdO559aE3h2C48b3FrH4S8OXsNvpdmDBdsFjtsvPkRDyGyOvZOg/D0Y28DXKXLQxmdEKLKVG5VJBIB6gEgZHsKFt4FuXuFhjE8iqjyhRuZRnAJ6kDJwPc+tC0SX9df8xt3v/Xb/ACHRxpFGscaKkaAKqqMAAdABTqKKBBRRRQAUUUUAFFFFABRRRQAUUUUAMf78f+9/Q0+mP9+P/e/oafQAUUUUAFFFFABRRRQAV5hq6xt4rvNS+zQt9l1izgOoMf8AS7csIh5MS94m38ncv+sf5Gxk+n1Rk0bS5tUj1SXTbOTUIl2x3bQKZUHPAfGQOT37mhaSTDo0cvB4d0STx7NeWujadajSYvMaeK2SNpLmUEkswGTtTn6yZ6gVmeHtHttBS3XVV8PyLqNhK8+p6bAbeQINpZmuN+XVtw+YbOQD349Fjt4IZZpYoY0kmYNKyqAXIAALHucADnsBVO00LR9Pe5ey0qxtnuv+PhobdEM3X7+B83U9fU0LQfW5xkMreEb/AF6Cw0iK28yK3ktbLS4GniUMzx+c6RoGDHGWAU8IMMxzjK0mx/4SDw6ulQXNu+3Xp3l/t6ykP2kDzGx5T7PMbOGK5G3BzjGK9L03SNM0aBoNL060sYnbe0drAsSs3TJCgc8Ulzo2lXtnJZ3WmWc9rLIZZIZYFZHcnJYqRgnPOaOt3/Wqf6C6WX9aNFLwoEj0U2sdnZ2q2s8tvssoPJhYq5BZEydoJycZODnk1sJ9+T/e/oKbb20FnbR21rDHBBEoSOKJAqoo6AAcAU5Pvyf739BTYD6KKKQBRRRQAUUUUAFeeWOlPpmujWpIPD2oTX+pXEcM9vZ5uY8+ZjNxu52qm1lCjHIzxz6HVGHRdKt9Tl1OHTLKLUJRiS6S3USuOOC4GT0HftStqHSx5nAWsNGtLzRMJfX3hq6ur6WI4aWdRHtlc9TIGaQZOT1q42j+XY6v4et10K1ku0spvOSF4rWRZJSux495DyEIRkFTJuUHGBXodppenWE9zPZ2FrbzXT77iSGFUaZueXIHzHk8n1qGDw/otrp8+n2+kWENjcEma2jtkWOQnAJZQMHoOvpVPX+vNuw/6/I4afTIre1XSG0XSLiC01RDcWNhBHZxahugLACKR9rOuVYqz8hM9gKisINMufD9lBqemjU5hc3kOmaK5WWLYJiFbkYCxqAu88KCQuSyg97/AMI9ov8AZP8AZP8AY+n/ANm5z9j+zJ5Oc5zsxt689OtMvPDOgajHbx32h6ZdJbp5cCz2kbiJf7q5HyjgcCkL+vzOB1fTFs9P1O31SZLnUNH8PxS2E7Es0U2ZcyRFskNuWMZzkgLmvUIS5gjMgxIVBYehxzVI6Do7LZKdJsCLA5swbZP9H6f6vj5Og6Y6CtCm2H9fl/XzGQ/6iP8A3R/Kn0yH/UR/7o/lT6QBRRRQAUUUUAFFFFABRRRQBxWl6NAPG3iO0vpZdTiubC0aYX22QMC8/wAu3AUKAAMAAdzkkkzeBtF023S+1ux020sRqMpEMdtAsSi3QlY+FA5blyevzgdAK6oW8C3D3CwxieRQjyBRuZRnAJ6kDJwPc+tRjT7IWkVoLO3FtCVMcIiXYhUgrhcYGCARjpihA9X/AF2ONg0vTrjxKL3SIC0tlcyy6hrJAMkzEMDbKwwZApIBH3VCKvLD5c7wJGlvrWlSi2t7f+0NJeYS27bnv8NGfOuf7snz8DMnLv8APxg9rB4V8O2uoC/t9A0uK9DlxcR2cayBj1O4DOTk81ZstG0vTbi4uLDTbO1muW3TyQQKjSnJOWIGWOSevrRHT+vL+v8Ah9QlqS33+pX/AHv6Gpov9TH/ALoqG+/1K/739DU0X+pj/wB0UAPooooAKqQKGmnyTw3ZiPWrdVrb/XT/AO9/jQBN5S+r/wDfZ/xo8pfV/wDvs/41xfi1p5fFukWi22s3kD2VzI1tpd+bViyvCA7HzYgQAxGMn73Sueiu725vNKtLi38SXiRx6gHsrXUjDcR+XPGE8yQTIJCqnGd7E7u/Wha/j+A2j1Xyl9X/AO+z/jR5S+r/APfZ/wAa85tNamTwO00c+pC31S+8ixQtLdXttCceYGKb3Mi7ZjjLFeASMYDY/El3c6v4WvHurmGKK3uEv4ZN8Qd1ligLOjYxgvvG4ZANPrb+tri6HpHlL6v/AN9n/Gjyl9X/AO+z/jXJ+Ar27vl12a7uJpd+oCSJZXLeUjwRSBB6Ab+g966+hqwDPKX1f/vs/wCNHlL6v/32f8afRSAZ5S+r/wDfZ/xo8pfV/wDvs/40+igBnlL6v/32f8aPKX1f/vs/40+igBnlL6v/AN9n/Gjyl9X/AO+z/jT6KAIXjXdHy/Lf3z6Gn+Uvq/8A32f8aH+/H/vf0NPoAZ5S+r/99n/Gjyl9X/77P+NPooAZ5S+r/wDfZ/xo8pfV/wDvs/40+igBnlL6v/32f8aPKX1f/vs/40+igBnlL6v/AN9n/Gjyl9X/AO+z/jT6xvFcOp3Hh24g0iN5LyRo0UJcmAhS67z5g5X5d3IyfQE0MEa3lL6v/wB9n/Gjyl9X/wC+z/jXERrDDpN1aane3mlxWNyr3yx6rc3TTBkGxI52IlGSV+VQCSNoB3cwXtjcp4PikvJNWXUJJzb6dE2p3EUiiWTEQmMci7yqkFsknCkZJyaPQDvvKX1f/vs/40eUvq//AH2f8a5/WdN1O38FDTNIkury9VYYRLLeNHK671Ds033gdu45GT6DtWHGZd9n4el+22byal5d75er3FyWX7O8qhJ3KyKDtXIG3ow6EknWwLa7O88pfV/++z/jTEjXdJy/Df3z6CsfwhdT3OhutxK8zW15c2qSucs6RzOiknucKAT3xk1tp9+T/e/oKA8g8pfV/wDvs/40eUvq/wD32f8AGn0UAM8pfV/++z/jR5S+r/8AfZ/xp9FADPKX1f8A77P+NHlL6v8A99n/ABp9FADPKX1f/vs/40eUvq//AH2f8afRQAzyl9X/AO+z/jR5S+r/APfZ/wAafWdrkYl0iYPqJ06AYae5DbCsQILgPkbMqCN2cjORyKGCL3lL6v8A99n/ABo8pfV/++z/AI158bHXbzSRHpEeoTabPqKvAt7qk1vMLYRncWm5mVWkAIHLYPOBwLMjWj6Jb/brzVLaK2lmtTYWuo3Ek9xdbuAk+4SyAYbAOBg5YALwf1/X329Q/r8/8r+h3HlL6v8A99n/ABo8pfV/++z/AI15zqD6zFpmoPqGo3UWoaJocV3H5VwVVpz5pZpApCyj90q4YFfvYHNejQuZII3ZdrMoJHpkU7B/X5f5jIo1MKHL8qP4zT/KX1f/AL7P+NEP+oj/AN0fyp9IBnlL6v8A99n/ABo8pfV/++z/AI0+igBnlL6v/wB9n/Gjyl9X/wC+z/jT6KAGeUvq/wD32f8AGjyl9X/77P8AjT6KAGeUvq//AH2f8aPKX1f/AL7P+NPpGyVIUgNjgkZxQA3yl9X/AO+z/jR5S+r/APfZ/wAa87nmvNLg1W90jVL+9+w6dcfb7yedpIZLoAFfLRyVUqQ5YJhV+6ckYXf0FZNO8Taho8d1dXNollb3Sm6uHndJHaVW+dyTghFOM4HOBzQgen9en+Z0vlL6v/32f8aPKX1f/vs/4159Ddaha+K7JZX1lb651Se3uRMZPshtykjReWD+6yFSM5T5hht3U1q6XDPpPjT7EZtUNrPaOTJf3RnF3MrKS8Y3ERYDNlcIDkbVwvAtbf15/wBeYPT+vO39eR0l4gWJSC33u7E9jViL/Ux/7oqG+/1K/wC9/Q1NF/qY/wDdFAD6KKKACq1t/rp/97/GrNVIFJmnw5X5u2PegCrqnh3TtYuYLm6F0s8CMkctrezW7BWILDMbqSCVXr6U6y8P6Zp81vNbWxWW3jkjR2kdmIkZWcsSSWZmUEs2Tnvya0Njf89X/If4UbG/56v+Q/woWgFC20HTbPUZr+3tilxNI0jkSMV3sAGYJnaCQoyQBnn1NV7vwnol9PeTXNl5j3kUkU+ZXAZZAiuMA4GRGnIwePc1r7G/56v+Q/wo2N/z1f8AIf4UAQWenWlhJcyWsPltcyCWbDE7mCqgPPT5VUcelWqZsb/nq/5D/CjY3/PV/wAh/hQA+imbG/56v+Q/wo2N/wA9X/If4UAPopmxv+er/kP8KNjf89X/ACH+FAD6KZsb/nq/5D/CjY3/AD1f8h/hQA+imbG/56v+Q/wo2N/z1f8AIf4UAD/fj/3v6Gn1C6Nuj/ev970Hofan7G/56v8AkP8ACgB9FM2N/wA9X/If4UbG/wCer/kP8KAH0UzY3/PV/wAh/hRsb/nq/wCQ/wAKAH0UzY3/AD1f8h/hRsb/AJ6v+Q/woAfVXUNOtdUs2tbyMvExDfK7IysDkMrKQVYEAgggip9jf89X/If4UbG/56v+Q/woAw5/BmiXNrbwTRXjfZ5/tEcv9oXAm8zbt3GUPvY7eBknA4FXodDsYYbSLFzKLSYzwtcXcszq+GGSzsS3DHgkj8hV7Y3/AD1f8h/hRsb/AJ6v+Q/woAqX2j2WpJMl2krrKEBAnddpQkqyYI2MCc7lweBzwMVW8L6S+m/YHiuHj80Tea13KZxIOj+cW8zcAMZ3dOOnFauxv+er/kP8KNjf89X/ACH+FAEVlZW+nWcVpaRCOCIYVQSfckk8kk5JJ5JJJqVPvyf739BRsb/nq/5D/CmIjbpP3r/e9B6D2oAmopmxv+er/kP8KNjf89X/ACH+FAD6KZsb/nq/5D/CjY3/AD1f8h/hQA+imbG/56v+Q/wo2N/z1f8AIf4UAPopmxv+er/kP8KNjf8APV/yH+FAD6oaxo1jr2nmx1GJ5Lcusm1JniO5SGU7kIIwQD17Vc2N/wA9X/If4UbG/wCer/kP8KAMc+E9Ka0Fs51GRVk81JJNTuWljbGMpKZN6ZBIIUgHJz1qO48GaJctZu8V4ktojpDLBqFxFIA5BfLo4ZixAJLEknqa3Njf89X/ACH+FGxv+er/AJD/AAoAybjwpo921q1zBNK1sgjVnupSZEyDtlO796uRnD7h19TWzTNjf89X/If4UbG/56v+Q/woAIf9RH/uj+VPqGJGMKfvXHyjjA/wp+xv+er/AJD/AAoAfRTNjf8APV/yH+FGxv8Anq/5D/CgB9FM2N/z1f8AIf4UbG/56v8AkP8ACgB9FM2N/wA9X/If4UbG/wCer/kP8KAH02RFljaNs7WBU4JBwfcUmxv+er/kP8KNjf8APV/yH+FG4GLpfg/RtHh8izjvPswiMItp9QuJodh4K+XI7Jj8Kn0/w3pml27Q2cdwgZ0dna7leQ7MbV3sxbaMY2524JGME1p7G/56v+Q/wo2N/wA9X/If4UAZkHhrSbbU5NQjtm8+RnYq0ztGrP8AfZYyxRWbnJUAnJz1NGm+GtL0i48+0hmDqhjj865lmEKHGVjDsRGvA4UAcD0Faexv+er/AJD/AAo2N/z1f8h/hQBDff6lf97+hqaL/Ux/7oqveKREuXZvm6HHofarEX+pj/3RQA+iiigAqtbf66f/AHv8as1Wtv8AXT/73+NAGFrT6jdeLdN0q11e606CWyuLiRrWOFmdkeJV5kjcAYdugFZ1xqWvLaXenxXNzM+n6gsN3f2tvG9z9nMQkDrHgqXBZVYBDkZKpkgDe1bw+dT1K11CHVb7Trq2ikhV7UQncjlSQRJG46ovTFQ/8InaixSKO9vY7xLg3X9oq6eeZiNrOcqUOVO3bt2gYAAwMC2/rv8A5D/r8DlbrxDfx2lhCviDUGhk1Vrd7mDTc34j8h5AkkBgO1twHIj5TB7k11HhS51C5hvjdy309qk4FncX9qLeeRNi7tyBEwA+4AlFyPXqX2/hW2iura8nvby6vIbo3TXEzIGlfymiAYKoUAK3AULyMnJznep/1+Qn/X4hRRRSAKKKKACiiigAooooAKKKKAGP9+P/AHv6Gn0x/vx/739DT6ACiiigAooooAKKKKACiiigAqK5uYbO1lubiRY4YkLyO3RVAyTUtRXVrb31rJa3cEVxbyrtkilQOjj0IPBFD8gRwp8QeI7jTfEbpb3Iuo7m3SztrWFGmhjkCdmG0uAxY7vlBz2FFxrd3Z6XFDda1qenn7esV/ealHaebZoYyy/NGphAYhAGYNjeQecY27bwNounPqEmkw/2VNesjGWwSOFotgGAuFxtyMlWDAknI5qf/hGiLN0XWtTS9eYTPqCNEszkDaAyiPyyu3jaUx3+9zQAnhDVJ9X0AT3E3nyRXE1uJ9mwzLHIyrIVwACwAJwAOeABxW2n35P97+gqvpmnQ6VYR2kDOyqWZpJDl5HZizMx9SxJOMDnjFWE+/J/vf0FNgPooopAFFFFABRRRQAUUUUAFUtUjv5rBotOuEt7h2VTM4z5aZG5lBBBYLnGRjOM5HFXazPEGiR+IdHm0ya7urWKUje9sVDMAc7TuVgVPQgjBGR0NJjRyDa7qSadhNaup9Pk1EQQ6pFbRSXU0XlFmMUaoVlxINuVjPy7jggb6ik8SanLo+lebq11b+al0WuLC0jnuZDE4WPfDtfZkZL4UBWwpKdD0z+FWltIIptd1OS4tZhLa3e23WSA7SpChYghUqSMMp6/TEf/AAhdtFDbfY9T1Kzu4RMGvIWjMs3mtvk370ZPmYBuFGMcYHFN7f12/r/ggnt/X9f1sYd14g1u40qfUYr6O3fTdFh1KWK3WOSG6kcOzKWIb93iPAKMD8x5PFd9FIJoY5QMB1DAH3rnZ/A+mSwW9tDPeWtrHaJZTQQyKVuYFOVjk3KTgZblSrfMeea6UDAwOlU7a2J7f12/4IyH/UR/7o/lT6ZD/qI/90fyp9SMKKKKACiiigAooooAKKKKAOK/tnxHDqXiiO6SF3tNOjubG0skMpDN5wAyVBdm2LxgAdAOpN3wnd3rPNbarcayb7ykl8jU0tAQpyNyfZxjBIIwxyMDgd9WXQ7eW/v7zzrlJr22jtnMUuwoqFyCpHIb94ec9hUVloL2jyTyavf3V4+xTdTrCHEatu8sBY1XacnJxu568DAt/wCv69Qlrt/W3/BOXtfE92vim3gm1gSTT6jPZ3GkGJFFpEokMcmQu8EiNTlmKtvOAOMa2i69far4uvIyVXSTZRzWa7fmcGR1MhPXDbeB/dweprQXw3CdV+2XN/fXcSyPLDZ3Lq8MLuCrFfl3HhmADMQAxAA4xFp3gvw9pGuNq2m6VZ2dwYBABb28caqMkkjaoOTnBOeQBQul/wCtP8wlre39a/5Gtff6lf8Ae/oami/1Mf8Auiob7/Ur/vf0NTRf6mP/AHRQA+iiigAqpBu86faQPm7jPrVuq1t/rp/97/GgCbEv99P++D/jRiX++n/fB/xrn9e8SvoeqLG8KyWq6bc3rhQfMZomjCqpzjnee3pTY9Q1/Tb/AE9dZOnz21/L5I+xwvG1rIVLKCWZvMU7Su7CYODjngXTz/4YHodFiX++n/fB/wAaMS/30/74P+NPooAZiX++n/fB/wAaMS/30/74P+NPooAZiX++n/fB/wAaMS/30/74P+NPooAZiX++n/fB/wAaMS/30/74P+NPooAZiX++n/fB/wAaMS/30/74P+NPooAZiX++n/fB/wAaMS/30/74P+NPooAhcSbo/nT73Hy+x96fiX++n/fB/wAaH+/H/vf0NPoAZiX++n/fB/xoxL/fT/vg/wCNPooAZiX++n/fB/xoxL/fT/vg/wCNPooAZiX++n/fB/xoxL/fT/vg/wCNPooAZiX++n/fB/xoxL/fT/vg/wCNPooAZiX++n/fB/xoxL/fT/vg/wCNOYkKSFLEDgDvXGTeJ9Z0WSV9ajsJB/Z01+1raqyyWmzBCO5ZlcHJXdhOVOARnBcaVzscS/30/wC+D/jRiX++n/fB/wAa4228V6idLvHeaxuLvzreC2/0Oe12tM4QFo5Tl0BOQ6sA+CBtxmm3Xi++sLeaxuWtv7Ui1AWP2iK0lkjbMPnBxArF2O35dgYnPOcUf1+X+aEtTtMS/wB9P++D/jTEEm6T50+9z8vsPeqmg30upaLb3U8kEkrgh2gR0UkEj7j/ADIeOVblTkZOM1eT78n+9/QUNWAMS/30/wC+D/jRiX++n/fB/wAafRQAzEv99P8Avg/40Yl/vp/3wf8AGn0UAMxL/fT/AL4P+NGJf76f98H/ABp9FADMS/30/wC+D/jRiX++n/fB/wAafRQAzEv99P8Avg/40Yl/vp/3wf8AGn1BeTy21nLNBaS3cqLlYImVWc+gLEKPxIoAkxL/AH0/74P+NGJf76f98H/Gub0nxPdTeE21TUrWJL0XM9strbMWDyLM0aIpPUnaOcAdTgDpjSeNdTfSfDrqYLa81O3knm8vTLi+CBNowscTBsZYfMTjj3oegHe4l/vp/wB8H/GjEv8AfT/vg/41xWpeMrmC5t7a0vbB2S0huprhrGeSKYOWHWMn7OnyEl33AZHXBruB0osBDEJPJTDpjaP4f/r0/Ev99P8Avg/40Q/6iP8A3R/Kn0AMxL/fT/vg/wCNGJf76f8AfB/xp9FADMS/30/74P8AjRiX++n/AHwf8afRQAzEv99P++D/AI0Yl/vp/wB8H/Gn0UAMxL/fT/vg/wCNGJf76f8AfB/xp9c5/bmqP41XRjp6W1k9nNLFcSuHeV0aMZCq3CfvO5DE9lxydbAdBiX++n/fB/xoxL/fT/vg/wCNcguseJrXUdWjkk0/VYtNsjLIlpYyQO05G5IlJlkydoyeMjcuM54ueE9fudZmvYpbzTtRhgEbJf6dGyQsWB3RYLv864BOG6OMgGgHodHiX++n/fB/xoxL/fT/AL4P+NcfbeKtRbV7VpjYNYXuoXGnw20asLiN4t/zs5bDZ8s5UKMb15OOXeFvFN9rOq/ZriSwmVrYzSxWsbrJp0gYDyJyzHLnJ7If3bfL6C1B6f18jprwP5S7mUjd2XHY+9WIv9TH/uiob7/Ur/vf0NTRf6mP/dFAD6KKKACq1t/rp/8Ae/xqzVAT+RNL8u7LeuKAKmq+HINY1JLm5lPkfYbiyeELgsJTGdwbPBGz07+1QW+galLf2c2saxHfQ2DmS2jjtPJZn2lQ8p3MHYAn7oQZJOOgGr9v/wCmX/j3/wBaj7f/ANMv/Hv/AK1C0B6lyiqf2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AFyiqf2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AFyiqf2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AFyiqf2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AFyiqf2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AFyiqf2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AFl/vx/wC9/Q0+qLXuSp8v7pz972p32/8A6Zf+Pf8A1qALlFU/t/8A0y/8e/8ArUfb/wDpl/49/wDWoAuUVT+3/wDTL/x7/wCtR9v/AOmX/j3/ANagC5RVP7f/ANMv/Hv/AK1H2/8A6Zf+Pf8A1qALlFU/t/8A0y/8e/8ArUfb/wDpl/49/wDWoAtSB2jYRsFcghWIyAexx3rk9J8I6la2V9Zarqtlfw38brdzpYPFczMwxuaQzOOBwFCgAYAwBiui+3/9Mv8Ax7/61H2//pl/49/9agLmC/hO7u4rubUNVik1OVIEhube1MSReS5kjJjLtuO8kt8wBHAA61Ivhe7SB7oajB/bT3n203ZtCYd/l+Vt8rfnb5fGN+c857Vtfb/+mX/j3/1qPt//AEy/8e/+tQBFouljSNO+zmXzZXlknmkC7Q0kjl3IXJ2jLHAycDHJ61eT78n+9/QVW+3/APTL/wAe/wDrU1b3BY+X945+97UAXqKp/b/+mX/j3/1qPt//AEy/8e/+tQBcoqn9v/6Zf+Pf/Wo+3/8ATL/x7/61AFyiqf2//pl/49/9aj7f/wBMv/Hv/rUAXKKp/b/+mX/j3/1qPt//AEy/8e/+tQBcoqn9v/6Zf+Pf/Wo+3/8ATL/x7/61AGRF4OsW02O0vJrmUw3lxdxS29xLbMjSu7EZjcHgOV68+gzUWm+GNT0DRLDTtG1tE+zoyyG+t3uUkyc5A81WUjsA+3BPHcbn2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AbnP3Pgx/sX2Kx1BILWeyFjfCW38x5ohu5Rgy7H+d+SGHI44rqkRY41jUYVQAB7Cqv2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AFmH/UR/7o/lT6ope7EVfLzgY+9Tvt/wD0y/8AHv8A61AFyiqf2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AFyiqf2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AFyiqf2/8A6Zf+Pf8A1qPt/wD0y/8AHv8A61AFys2bSvN8R2mr+dj7PazW/lbPveY0bZznjHl9Md/apvt//TL/AMe/+tR9v/6Zf+Pf/Wo63AzW8MRTaHrOm3Fwzf2rJO8syLtYCTgDknO1Qq++3p2qPS9A1KyvrnUbnUrSe/njggLRWRiiEMbMcbPMJ3ne/wA27A4+Xgg632//AKZf+Pf/AFqPt/8A0y/8e/8ArULQHqYS+D9/iWTVbq4tJF8xpUMVgsVwxKlFEkwP7xVVmCjaD93JJHJoHhKfSL2ymudRiuYtOs2srJIrXyWETFSfMbc29v3a8gKOpxzxu/b/APpl/wCPf/Wo+3/9Mv8Ax7/61C02B67j77/Ur/vf0NTRf6mP/dFUZ7nzkC7MYOeuavRf6mP/AHRQA+iiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAZ58XnND5qeaqh2TcNwU5AJHodp/I+lO3L6j86xE/5HLUf+wfaf8Aoy4rToAsbl9R+dG5fUfnVeigCxuX1H50bl9R+dV6KALG5fUfnRuX1H51XooAsbl9R+dG5fUfnVeigCxuX1H50bl9R+dV6KALG5fUfnSLIjKGV1KkZBB61BUFl/x4W/8A1yX+VAF/cvqPzo3L6j86r0UAWNy+o/OjcvqPzqvRQBY3L6j86Ny+o/Oq9FAFjcvqPzo3L6j86r0UAWNy+o/OjcvqPzqvRQBY3L6j86QyIoyzqBnHJqCoLv8A1K/9dY//AEMUAX9y+o/OjcvqPzqvRQBY3L6j86Ny+o/Oq9FAFjcvqPzo3L6j86r0UAWNy+o/OjcvqPzqvRQBY3L6j86Ny+o/Oq9FAFjcvqPzo3L6j86r0UAT+Ym4LvXcRkDNLuX1H51Qb/j/AIf+uT/zSp6ALG5fUfnRuX1H51XooAsbl9R+dG5fUfnVeigCxuX1H50bl9R+dV6KALG5fUfnRuX1H51XooAsbl9R+dG5fUfnVeigCxuX1H50gkRhlXUjOODUFQWn+pb/AK6yf+hmgDQ69KKbH9wUUAOooooAKKKKACiiigAorn/GX/IDtv8AsK6b/wClsNL4j+XVfDE7YWKLVSZHPAXda3CLk+7Oqj3YUAb9FcpJpGm65401hdR0+01C0jsbFNtxCsqLMr3RIwwIDBZF98OPWo/h9pFlYfD7R59MsrKzvLzSrZ5Z0tlBlk8oENJtwX5Ynk55PIzQBpp/yOWo/wDYPtP/AEZcVp1w+oy6paeLLoXN7E8rWNud1rE0K7d8+AQXbJ68579KP7Rvf+fub/vs1x1cZGnNwa2OKtjo0puDWx3FFcP/AGje/wDP3N/32aP7Rvf+fub/AL7NZ/2jDsZf2lD+VncUVw/9o3v/AD9zf99mj+0b3/n7m/77NH9ow7B/aUP5WdxRXD/2je/8/c3/AH2aP7Rvf+fub/vs0f2jDsH9pQ/lZ3FFcP8A2je/8/c3/fZo/tG9/wCfub/vs0f2jDsH9pQ/lZ3FFcP/AGje/wDP3N/32aP7Rvf+fub/AL7NH9ow7B/aUP5WdxUFl/x4W/8A1yX+Vcd/aN7/AM/c3/fZpFv7tVCrczBQMABzxR/aMOwf2lD+Vnc0Vw/9o3v/AD9zf99mj+0b3/n7m/77NH9ow7B/aUP5WdxRXD/2je/8/c3/AH2aP7Rvf+fub/vs0f2jDsH9pQ/lZ3FFY/h+eWe3mM0ryEPgFjnHFR+KWZNNTyZJo7mSVYYWjmdMMxHUKRngHrXZSqKpBSXU7qVRVIKa6m5RXOTm+tPE+jWzXjtaOsirHk5bbEOXP8RySfwHemJdXGlFUv7iWZYX3yGNyxJ8vOCTjjCu5HbKgZ4rQ0OmooooAKgu/wDUr/11j/8AQxU9QXf+pX/rrH/6GKAJ6KKKACiiigAooooAKKKKACiiigAooooAgb/j/h/65P8AzSp6gb/j/h/65P8AzSp6ACiiigAooooAKKKKACiiigAooooAKgtP9S3/AF1k/wDQzU9QWn+pb/rrJ/6GaAL0f3BRRH9wUUAOooooAKKKz9R1P7BfaTbeT5n9oXbW27djy8Qyy7sY5/1WMcfez2wQDQorPm1PyvENnpPk5+02k9z5u77vlPCu3GOc+dnOeNvfPGHrPivUrC01+7stJtLmDQ3cXPnXzQs6rbxT5QCJgSRIVwSPug554AOkvrCz1Ozks7+0gu7WTG+GeMSI2CCMqeDggH8KpWnhnQLC2ubaz0PTbeC6ULcRQ2kaLMBnAcAYYDJ6+prMm1zxJFqNnpv9iaUb65inuMf2rJ5axxGFfvfZ8liZum3AC9TnAU+LLhNEv7h9MQ6laXi2H2SO53JLM+zYFk2j5f3i5JUYw2RxQBvWOn2Wl2q2un2dvaW6klYreJY0GeuAABUsEENrbxW9vFHDBEgSOONQqooGAABwAB2rnB40tmvfDUK2rmLXbeSdZd2PI2iPCsMdS0gX2P6WNA8VW2uW+r3Jj+y22nXj2xllkG10VEfzc/wqQ+ee3NAFW802LUfGV75ruvl6fa4247yXH+FTf8I1a/8APab8x/hUWn6rYaj4x1FrG8t7pP7PthvgkEigiSfIJB6/MOPet+sZUKU3drUwlh6U5OUldmL/AMI1a/8APab8x/hR/wAI1a/89pvzH+FTN4gsFvbm1Ds720DzysoyqhTgj61YtdRiu2RESQOwcsrAfJtbaQcH1BxjOcGl9Vo/yi+qUf5Sj/wjVr/z2m/Mf4Uf8I1a/wDPab8x/hW1RR9Vo/yh9Uo/ymL/AMI1a/8APab8x/hR/wAI1a/89pvzH+FbVFH1Wj/KH1Sj/KYv/CNWv/Pab8x/hR/wjVr/AM9pvzH+FbVFH1Wj/KH1Sj/KYv8AwjVr/wA9pvzH+FH/AAjVr/z2m/Mf4VtUUfVaP8ofVKP8pi/8I1a/89pvzH+FR2/h62ltopGlmBdAxwR3H0reqCy/48Lf/rkv8qPqtH+UPqlH+UzP+Eatf+e035j/AAo/4Rq1/wCe035j/Ctqij6rR/lD6pR/lMX/AIRq1/57TfmP8KP+Eatf+e035j/Ctqij6rR/lD6pR/lKlhp8enxukTOwY5O7FTywRT7PNiSTYwdN6g7WHQj0PvUlFaxioqy2NoxUVyx2I3gikljleJGkjz5blQSmeDg9s0j2tvKpWSCJ1Zg5DIDlh0P14HNS0VRQUUUUAFQXf+pX/rrH/wChip6gu/8AUr/11j/9DFAE9FFFABRRRQAUUUUAFFFFABRRRQAUUUUAQN/x/wAP/XJ/5pU9QN/x/wAP/XJ/5pU9ABRRRQAUUUUAFFFFABRRRQAUUUUAFQWn+pb/AK6yf+hmp6gtP9S3/XWT/wBDNAF6P7gooj+4KKAHUUUUAFYfiKw1K6l0e70uK0mn0+9Nw0V1O0KupgmiwGVHIOZQenY1uUUAc3Laa/Pc2urmy0yPU7VZrdbYX0jwyQyeWxJk8kFW3Rr/AAMMD34juvDl7eeE/Elo726anrkUxfazGGORoFhUBsZKhUTJxzycDpXUUUAc9rfhWy8Qa/p91qdlZXtjbWlzC0NzGHIkkaEqygggYEbjOQRkY6nGdD4KY2um6RNIkejabPJLALOaS3mbjEQJj2kFQ8mSG5KqTnccdlRQB57d+AtTSMQ6bdwJHZR3J01riV3dHee2uI95IJIEkMgJyTtK9ckDofCXh2Tw3bXtszxvFLNC0JUknalrBCd2R1LRMe/BHfNdDRQBzeD/AMLCuzjj+yoOf+2s1bdZif8AI5aj/wBg+0/9GXFadJKzYkrNmV/ZTf8ACSNqGIvszWZgKdyxfcTjGMGov7Iuo7mGW3kjhUS5eONiqhAy7QABg/KGGDxlyewraopjCiiigAooooAKKKKACiiigAqCy/48Lf8A65L/ACqeoLL/AI8Lf/rkv8qAJ6KKKACiiigAooooAKKKKACiiigAqC7/ANSv/XWP/wBDFT1Bd/6lf+usf/oYoAnooooAKKKKACiiigAooooAKKKKACiiigCBv+P+H/rk/wDNKnqBv+P+H/rk/wDNKnoAKKKKACiiigAooooAKKKKACiiigAqC0/1Lf8AXWT/ANDNT1Baf6lv+usn/oZoAvR/cFFEf3BRQA6iiigAooooAKKKKACsPxnPNa+BfENxbyyQzxaZcvHJGxVkYRMQQRyCD3rcqvf2NvqenXNheR+Za3UTQzJuI3IwIYZHIyCelAHD6rBJY+Drma3sfEdnNNqFjC1vc6u0s8iG5iUiN/tDiPcHZeHXPfAwa6nRbZrPRWNvZ38MzlnFtql+08gboAZN8uFOAflJAznGcir95Y2+oQLDdR+ZGsscwG4jDxusiHj0ZVPvjnirFAHA3uo6tbeLLszx29rK1jbjbBKZlKh58HLIuDyeMfjUn9uaj/z8f+OL/hV3UNL/ALS8ZXn77y/L0+2/hznMlx7+1Sf8Ix/0+f8AkL/69ebiKeJdRuF7ep5eIpYmVVune3r/AMEzv7c1H/n4/wDHF/wo/tzUf+fj/wAcX/CtH/hGP+nz/wAhf/Xo/wCEY/6fP/IX/wBesfY4vz+//gmPscZ3f3/8EbpGp3l1qCxTTbkKk42gfyFbl21wtsxtI0efgKHbC8nkn6Dn8KztP0T7Ddif7RvwCNuzH9a169DDRqRhapuejhY1IwtV3+85EatqEvg211EXki3srmNAiJh2MhUZBU9AO2Kv3FzqdhLIJ5wYTEESVlXG4KpL4HPGJWIPGAoHetBNGsI7S2tVgxDbSiaFd7fK4JOc5yeSetWZrSC4JM0YfMbRkEnG1sZGPfFdB0i208dzAssTlkORkqQcg4OQcYOQeKlqK3t4rWFYYVKoCTyxJyTkkk8kkkmpaACiiigAqCy/48Lf/rkv8qnqCy/48Lf/AK5L/KgCeiiigAooooAKKKKACiiigAooooAKgu/9Sv8A11j/APQxU9QXf+pX/rrH/wChigCeiiigAooooAKKKKACiiigAooooAKKKKAIG/4/4f8Ark/80qeoG/4/4f8Ark/80qegAooooAKKKKACiiigAooooAKKKKACoLT/AFLf9dZP/QzU9QWn+pb/AK6yf+hmgC9H9wUUR/cFFADqKKKACiiigArH1bXjpmo2lhBpV9qN1dRSzKlqYRtSMxhiTJIg6yr0z3rYrj/Fml3F34h0i9XTNVvrWC0uopBpl+LWRHd4CuW86IlcRvxk8gcdKALZ8YxS/YEsdH1O9nvEuH8iLyUeHyJFjlD+ZIoyHcLwT0Papm8WWqaJe6jLZXscllOttNZMimcTNs2RgBipLeYmMNg7hzWBY+DruaTQ49Q+2wQWtnqCl7bUJIpYTLPC8MbPG4ZyEUgnkEpk5OCZIfC2pva6fpBluLOK1vHup9TikSWW6ZABE7eYHyzbgTuU4MRxgbaAN7/hLNLN7oVsHkJ1uB57NwvyFVVW+Y54yHXH/wCqrGi6/Za9/aH2LzCLC8ezlZ1wC6qrEr6rhhg964W78J+ILSO1jsoWu30eK6NhPLJGpnJuLWeJCBtAyEljPCgBewIz0Xg3w5No2m6pYahAskU80X+s2uJ1FnbxuSPd0kBB6/QigCzJd29r4yv/AD5Qm7T7XGe/7y4q7/a1h/z8p+tcfq+n2Wm+LLiGws7e1iaxt3KQRKilt8wzgDrwPyqOvOr4ydOo4JHmYjGzpVHBJaHaf2tYf8/KfrR/a1h/z8p+tcXRWP8AaFTsjH+0qnZHaf2tYf8APyn61JDqFpcSCOKdXc9AK4etPQf+QtH/ALrfyq6eOnOai0tTSlj6k5qLS1OpubmGztnuLiQRxIMsx7VUbW9NSxa9e6VYEcoxZSCGHVduM59sVPfvdR2MzWUXm3IH7tCQOfXkgcdevaucvfDrTeF5oYbOR9QZy+6dkDszOpduGKjIX16CvUPVOifUrSOaaJ5sPCheQbTwAATzjngjgc8j1q0DkA+vqKx9S0xpZ2mtIVEzIzs7NwzADYuM9yFJOOiAGr+n/afsSfa8+dls7tucbjtzt4zjGccZoAs0UUUAFQWX/Hhb/wDXJf5VPUFl/wAeFv8A9cl/lQBPRRRQAUUUUAFFFFABRRRQAUUUUAFQXf8AqV/66x/+hip6gu/9Sv8A11j/APQxQBPRRRQAUUUUAFFFFABRRRQAUUUUAFFFFAEDf8f8P/XJ/wCaVPUDf8f8P/XJ/wCaVPQAUUUUAFFFFABRRRQAUUUUAFFFFABUFp/qW/66yf8AoZqeoLT/AFLf9dZP/QzQBej+4KKI/uCigB1FFFABRRRQAUUUUAFFc3rR1C68VaVpdpq93p0Etld3ErWscLM7RvbqoPmxuAMSt0A7Vlyap4gXTryy8+eabTtUFpdX1nbI9w0BgWVZFiIK78yRocKR95gB0AB3FFed3fiK8/svTAmuah5MurtaSXNrp2698sW0kmySAwna4dRnbH90BuMnHSeG7vUpNKvZbsXt0sc7fY3ubdbe4uIgikbkIQK2/eoyF4AJxmgBwRX8ZajuUNjT7TGRn/lpcVpeTF/zyT/vkVyNzrd5F4svH+wS2jNY2wMd0UZiA8/I8t2GOT3zxU//AAkd7/ch/wC+T/jXLUxVKEnGW5yVcXSpzcZbnT+TF/zyT/vkUeTF/wA8k/75Fcx/wkd7/ch/75P+NH/CR3v9yH/vk/41H12h/SM/r1D+kdP5MX/PJP8AvkUojjU5VFB9QKztHv5r+KVpggKsANoxUXiPV7jR9Oa4trXzWAyXb7icgc9yTu4A9/SumnKM4qUTrpyjOKnE2KKwpddkHiI6chgjjjeNG80Nly6lvlI4B6YB6880601i4bykvkitnL5cyAoAu0YHJ67yVB6HaSK0NDbooooAKKKKACoLL/jwt/8Arkv8qnqCy/48Lf8A65L/ACoAnooooAKKKKACiiigAooooAKKKKACoLv/AFK/9dY//QxU9QXf+pX/AK6x/wDoYoAnooooAKKKKACiiigAooooAKKKKACiiigCBv8Aj/h/65P/ADSp6gb/AI/4f+uT/wA0qegAooooAKKKKACiiigAooooAKKKKACoLT/Ut/11k/8AQzU9QWn+pb/rrJ/6GaAL0f3BRRH9wUUAOooooAKKKKACiiigDH1bQTqeo2l/Bqt9p11axSwq9oITuSQxlgRJG46xL0x3qBPCVlFpq20VzeLcrcG7/tDepuDOVKmQkqVJKkrjbt28AAACt+igDDtvC9rbyWs73V3cXMF61+88rLunlMLQZfCgYCNgBQuNq++dysqbxJpUFkt29y7RtcSWqLHBJJI8sbMjqqKpZiCjdAeAT05q3p+oWmqWaXdlMJYXJAOCCCDggg8gggggjIIoA5nVNNl1Hxld+U6L5en22d2e8k/+FH/CNXX/AD2h/M/4VoGeGHxlqHmypHnT7TG5gM/vLir/ANutP+fqD/v4K5qmFpTlzS3MJ4GFWXO09TA/4Rq6/wCe0P5n/Cj/AIRq6/57Q/mf8K3/ALdaf8/UH/fwUfbrT/n6g/7+Co+pUP6ZH9mU/wCVlbSNPl0+KRZHRi7AjbmptT0+LVdPlsp2dY5MZKEA8EHuD6U/7daf8/UH/fwUfbrT/n6g/wC/grphGMIqMdjqp0HTioxTsVJ9Cs7jU1vnMgkDpIUDDazICFY8ZyMnvT7zR7e9EnmvLmRtxZSMgbSm0cdMM3/fRNWPt1p/z9Qf9/BR9utP+fqD/v4Kq6L9nPsWKKr/AG60/wCfqD/v4KPt1p/z9Qf9/BRdB7OfYsUVX+3Wn/P1B/38FH260/5+oP8Av4KLoPZz7FioLL/jwt/+uS/ypPt1p/z9Qf8AfwVDZ3tqtlbg3MIIjUEGQelF0Hs59i9RVf7daf8AP1B/38FH260/5+oP+/goug9nPsWKKr/brT/n6g/7+Cj7daf8/UH/AH8FF0Hs59ixRVf7daf8/UH/AH8FH260/wCfqD/v4KLoPZz7Fiiq/wButP8An6g/7+Cj7daf8/UH/fwUXQezn2LFFV/t1p/z9Qf9/BR9utP+fqD/AL+Ci6D2c+xYqC7/ANSv/XWP/wBDFJ9utP8An6g/7+Cobq9tTCoFzCf3kZ4kH98UXQezn2L1FV/t1p/z9Qf9/BR9utP+fqD/AL+Ci6D2c+xYoqv9utP+fqD/AL+Cj7daf8/UH/fwUXQezn2LFFV/t1p/z9Qf9/BR9utP+fqD/v4KLoPZz7Fiiq/260/5+oP+/go+3Wn/AD9Qf9/BRdB7OfYsUVX+3Wn/AD9Qf9/BR9utP+fqD/v4KLoPZz7Fiiq/260/5+oP+/go+3Wn/P1B/wB/BRdB7OfYVv8Aj/h/65P/ADSp6ote2v22I/aYcCNxnzB6rU3260/5+oP+/goug9nPsWKKr/brT/n6g/7+Cj7daf8AP1B/38FF0Hs59ixRVf7daf8AP1B/38FH260/5+oP+/goug9nPsWKKr/brT/n6g/7+ClF5aswVbmEknAAkHNF0HJPsT0U13SKNpJHVEQFmZjgADqSaoz67o9r5f2jVbGLzYxLH5lwi70PRhk8g+tDaW4Qpzn8KbNCiq41CyNxLbi7t/PhXfJH5g3IvqRnIH1p0V5azrE0NzDIsyloyjghwOpGOo57UXQnCS1aJqgtP9S3/XWT/wBDNT1Baf6lv+usn/oZpkl6P7gooj+4KKAHUUUUAFFFFABRRRQAUUUUAcFYQXWkT2Wq3NjePbwXmrxSxxW7ySIJrwvHKIwCzLtTHygnEgPTJrd8PWss8Gs3M8E9rDqd400ULExSJH5UceeCChYoz9iN3ODmugooA8w8RWMOn+LJYoXuGU2MDEz3EkzZ3zfxOxOOOmcVRrupbO2u/GV/9ogSXbp9rt3DOMyXFXP7H03/AJ8oP++BWE6TlK57WEzOFCiqbi9P8zzmivRv7H03/nyg/wC+BR/Y+m/8+UH/AHwKn2D7nR/bNL+VnnNFejf2Ppv/AD5Qf98Cj+x9N/58oP8AvgUewfcP7Zpfys85or0b+x9N/wCfKD/vgUf2Ppv/AD5Qf98Cj2D7h/bNL+VnnNFejf2Ppv8Az5Qf98Cj+x9N/wCfKD/vgUewfcP7Zpfys85or0b+x9N/58oP++BR/Y+m/wDPlB/3wKPYPuH9s0v5Wec0V6N/Y+m/8+UH/fAqK00nT3s4GazhLGNSSUHPFHsH3D+2aX8rPPqK9G/sfTf+fKD/AL4FH9j6b/z5Qf8AfAo9g+4f2zS/lZ5zRXo39j6b/wA+UH/fAo/sfTf+fKD/AL4FHsH3D+2aX8rPOaK9G/sfTf8Anyg/74FH9j6b/wA+UH/fAo9g+4f2zS/lZ5zRXo39j6b/AM+UH/fAo/sfTf8Anyg/74FHsH3D+2aX8rPOaK9G/sfTf+fKD/vgUf2Ppv8Az5Qf98Cj2D7h/bNL+VnnNFejf2Ppv/PlB/3wKiudJ09YlK2cIPmIOEHdgKPYPuH9s0v5WefUV6N/Y+m/8+UH/fAo/sfTf+fKD/vgUewfcP7Zpfys85or0b+x9N/58oP++BR/Y+m/8+UH/fAo9g+4f2zS/lZ5zRXo39j6b/z5Qf8AfAo/sfTf+fKD/vgUewfcP7Zpfys85or0b+x9N/58oP8AvgUf2Ppv/PlB/wB8Cj2D7h/bNL+VnnNFejf2Ppv/AD5Qf98Cj+x9N/58oP8AvgUewfcP7Zpfys85or0b+x9N/wCfKD/vgUf2Ppv/AD5Qf98Cj2D7h/bNL+VnnNFegtpOn/bIl+xw7TG5I2D1X/Gpf7H03/nyg/74FHsH3D+2aX8rPOaK9G/sfTf+fKD/AL4FH9j6b/z5Qf8AfAo9g+4f2zS/lZ5zRXo39j6b/wA+UH/fAo/sfTf+fKD/AL4FHsH3D+2aX8rPOas6d/yE7T/rsn8xXe/2Ppv/AD5Qf98CnJpWno6ulnCrKcghRwaFQd9yZ5xTlFrlZn+LtMGqeGb6HbM7pDJJHFETmRwjbQQPvckHHqBWHLp93d6Z4L0qW0n+zhY5bzMZwnlRAhX44y3GD3FdxRWkqSk7nm0MdOlBQSvZtrybVvw3POk8Ny3PjOVBFeiwea9e6M0OwYmjVfkkyQ4J6DgjByK1baPUNL1O3a2tZp4ZZ3h3zREuqGUF2O3AQsXkfJGCI1AHIrsKKlUEtjapmlSokpq6Stb79fXX7vvCoLT/AFLf9dZP/QzU9QWn+pb/AK6yf+hmtzyy9H9wUUR/cFFADqKKKACiiigAooooAKKKKACiiigDmLrUYbDxlfedHdvv0+1x9ntJZsYkuOuxTjr361Y/4SGz/wCfbVf/AAU3X/xurUFzpDeJbtIdSt31RoI45rRZ0LoiF2BKfeH+tPJ9q1aAOdl8T6dbwyTTRanHFGpd3fS7kKqjkkkx8AU//hIbP/n21X/wU3X/AMbrclijuIZIZo0kikUo6OoKsp4IIPUGn0AYH/CQ2f8Az7ar/wCCm6/+N0weJ9OMzQiLUzKih2QaXc7gpyASPL6Ha2PofSuipgijEzTCNBK6hGcKNxUZIBPoNzY+p9aAMP8A4SGz/wCfbVf/AAU3X/xuj/hIbP8A59tV/wDBTdf/ABut+igDnYvE+nXEMc0MWpyRSKHR00u5Ksp5BBEfINP/AOEhs/8An21X/wAFN1/8brciijt4Y4YY0jijUIiIoCqo4AAHQCn0Ac6fE+nCZYTFqYldS6odLudxUYBIHl9BuXP1HrT/APhIbP8A59tV/wDBTdf/AButwxRmZZjGhlRSiuVG4KcEgH0O1c/QelPoAwP+Ehs/+fbVf/BTdf8Axuq9h4ksH061eOHU3RoUKuml3LKwwOQRHgj3rXute0ex1GHTrvVrG3vp9vk201wiSSbjhdqk5OTwMdTUWl3OkWQh8PWmpW8t1YwJGbczo0yoqgAso5HGOcDrQBV/4SGz/wCfbVf/AAU3X/xumP4n06N40ki1NWlbZGG0u5BdsE4H7vk4BP0BroqY8UcjxvJGjNE2+MsoJRsEZHocEj6E0AYf/CQ2f/Ptqv8A4Kbr/wCN0f8ACQ2f/Ptqv/gpuv8A43W/RQBzsXifTpkLxRanIoZkJTS7kgMpII/1fUEEH3FP/wCEhs/+fbVf/BTdf/G63Ioo4UKRRpGpZnIRQAWYkk/Ukkn3NPoA51/E+nRvGkkWpq0rbIw2l3ILtgnA/d8nAJ+gNP8A+Ehs/wDn21X/AMFN1/8AG63HijkeN5I0Zom3xllBKNgjI9DgkfQmn0AYH/CQ2f8Az7ar/wCCm6/+N0xPE+nSPIkcWps0TbJAul3JKNgHB/d8HBB+hFdFTEijjeR440VpW3yFVALtgDJ9TgAfQCgDD/4SGz/59tV/8FN1/wDG6r3viSwSBS8OpoPOiGX0u5AyXUAcx9SeAO5OK6OeeK2gknuJUihjUvJJIwVUUckkngAVn69PpUNlENX1GCxgNxE6STTrEGdHWRVy3ByUHHXGaAKn/CQ2f/Ptqv8A4Kbr/wCN0f8ACQ2f/Ptqv/gpuv8A43WldaxpljLbxXepWdvJcnECTTqhlP8Asgn5uo6etXaAOdTxPp0jyJHFqbNE2yQLpdySjYBwf3fBwQfoRT/+Ehs/+fbVf/BTdf8AxutxIo43keONFaVt8hVQC7YAyfU4AH0Ap9AHOy+J9Ot4ZJpotTjijUu7vpdyFVRySSY+AKf/AMJDZ/8APtqv/gpuv/jdbksUdxDJDNGkkUilHR1BVlPBBB6g0+gDA/4SGz/59tV/8FN1/wDG6YPE+nGZoRFqZlRQ7INLudwU5AJHl9DtbH0PpXRUwRRiZphGgldQjOFG4qMkAn0G5sfU+tAGH/wkNn/z7ar/AOCm6/8AjdMl8T6dbwyTTRanHFGpd3fS7kKqjkkkx8AV0VMlijuIZIZo0kikUo6OoKsp4IIPUGgDD/4SGz/59tV/8FN1/wDG6P8AhIbP/n21X/wU3X/xut+o5J4oniSSVEeZtkaswBdsFsD1OATx2B9KAOcfxJYDUYUMOphzDIQh0u53EApkgeXkgZGT2yPWrH/CQ2f/AD7ar/4Kbr/43U17qGhWviSyW91ezt9TMLQ29rLcojyiV06ITuJLRgDHuOaux6tpsupSabHqFo9/EN0lqsymVB6lM5A5HagDM/4SGz/59tV/8FN1/wDG6ZF4n064hjmhi1OSKRQ6Oml3JVlPIIIj5BroqZFFHbwxwwxpHFGoRERQFVRwAAOgFAGH/wAJDZ/8+2q/+Cm6/wDjdMPifThMsJi1MSupdUOl3O4qMAkDy+g3Ln6j1roqYYozMsxjQyopRXKjcFOCQD6HaufoPSgDD/4SGz/59tV/8FN1/wDG6P8AhIbP/n21X/wU3X/xut+igDnYvE+nTIXii1ORQzISml3JAZSQR/q+oIIPuKf/AMJDZ/8APtqv/gpuv/jdbkUUcKFIo0jUszkIoALMSSfqSST7mn0Ac6/ifTo3jSSLU1aVtkYbS7kF2wTgfu+TgE/QGn/8JDZ/8+2q/wDgpuv/AI3W48UcjxvJGjNE2+MsoJRsEZHocEj6E0+gDA/4SGz/AOfbVf8AwU3X/wAbqvZeJLB4GKQ6m486UZTS7kjIdgRxH1B4I7EYrp6rWNlHp9u0MTOytNLMS5Gd0kjOfwyxx7UAN0+9ivrYyRJcIqsVInt5IWz16OoJHPXpRVuigAooooAKKKKACiiigAooooAKKKKAPPNMubZ9e0vTklibVbXXtRuLuEY8xIHFzsZh1CkPBgnrx6V6HRRQAUUUUAFFFFABRRRQAUUUUAFFFFAHEXs4sPGNzLZ62ZNSubq1jbSDGi7oPkV2+Yb3Cq0j7kIUHIOSDVPTLm2fXtL05JYm1W117Ubi7hGPMSBxc7GYdQpDwYJ68eleh0UAFFFFABRRRQAUUUUAFFFFABRRRQBy3xHsLe/+HmvC4VnWCwuJ0UOQN6xPtJAPzYPODkZAPUCm+Jb7T9L8TaVe6xPBBpv2G8hZ7ggIZWMBVee5RZQB35HeurooA8kunFh4ZmsdUkSLUbzwjZ2tjHN9+W6UTBlQHq29oTgc9D2r1uiigAooooAKKKKACiiigAooooAK5bxDYW//AAl3hPUSrG5F/JAGLkhU+yXTEBc4GTjJAydoz0FdTRQBxF7OLDxjcy2etmTUrm6tY20gxou6D5FdvmG9wqtI+5CFByDkg1T0y5tn17S9OSWJtVtde1G4u4RjzEgcXOxmHUKQ8GCevHpXodFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUVieK5p7fR4JLeWSJ/wC0rBWaNip2G7iDgkdipIPsTQBt0VwnxCvdTtEum029e1aPw7qdxkbioZGtyGwGHzhS4Vv4SScHkF6ajfnxLa2s1x80XiAW0xhaRUmH9lGU/IzttXechQcDAPLZYgHcUVwuhar4ivbPwy1td6a0N7Y21zcW72k0kkMJjUuTM05JJbIUspJzzu2sayNZ8Y6zPpOuWtvqGnmT+xLu+hubO1nCwGLYCqyswEpIk4dcbSASpBAIB6jRXGprWqR3t5pyPanUJdXjsEuHjkaFT9gjnd/LMmQPlcBVYDkEkncxSPxDr2pXWm2On/2dBcTRah9oluIXkQSWs8cOUUOp2sWY4JyARzxggHZ0Vy1zr1xe/D2w1eAfZbnVIbNUKnPkNctGgIPfaZM/hXLeLbnX/DOq3GpJPfLYqdkM7XfmQrALUgqYSdzSrIrTFsZKIRu/hoA9SorjBre7TbC7t11G1k03UbfTru1vnzIwl8tB5mGZWOJopA2SfflhXZ0AFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAVFc20N5bSW1zEssMqlXRhkMD2qWigDIg8MaTBZ3lr5M00d5EYbg3V1LO7xkEFN0jFguCeAcc0+Dw7pltLFLHA5liuBdLJJPI7GUQfZwxLMSx8r5ec56nnmtSigDAtfBujWNxbz2iX0DW8cMaJFqVwseyJQqKyCTawAGMEHPOc5NEPgrw/B5uyxYrLaSWRV7iRlWCTbuiUFiFT5RgLgDnGMmt+igDIPhnSjaTW3kzbZpUmeQXUvm+Ysaxq4k3bw2xFGQcnnOSTmxbaLp9nLaS29uEe0hkghO9jtSRkZwcnkkxqSTk5HXk5v0UAZU+gWcnhkaDAGt7SO3WCDaxZoQgAQgnJJUqpGc9OamudItdRiiGpwx3bpC8TZUhGDrtf5MkcjI5yQCRnk5v0UAYEPhOwtBZw2e+K0guvtksbu8r3EoXCF5HYsduAec/dQcAYO/RRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFABRRRQAUUUUAFFFFAH//Z" width="1200" height="700" />

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
