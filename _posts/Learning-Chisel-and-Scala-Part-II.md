---
layout:     post
title:      Learning Chisel and Scala
subtitle:   Scala Part II
date:       2018-11-24
author:     Max
header-img: img/post-gray-background.jpg
catalog: true
tags:
    - Chisel
    - Scala
---   

### V. Advanced data type: Collections
   Scala沿用了Java的`collection`称谓，作者就称其为复合类型了，C++里称STL中此类数据类型为"容器"，意思差不多. Scala中复合类型的继承体系如Fig-1所示，图中省略了很多中间父类和同级的兄弟类型，内容实在太多，这里仅对照C++ STL介绍一些常用的类型，即Table-1中罗列类型，详细的类型可以查看Scala官方[API](https://docs.scala-lang.org/api/all.html).
   
   Scala中的复合类型可分成两大类: `immutable`和`mutable`，即定义后不可变类型和可变类型，类似于`value`和`variable`的关系. 需要注意的是，`collection.immutable package`会自动添加到当前的`namespace`中，所以可以直接使用类型名称进行定义，但`collection.mutable package`不会自动添加，定义时需要写出完整的包路径或手动`import package`后直接使用类型名.
   
<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%235-%232.jpg?raw=true">

Fig-1. Collections hierarchy

</div>

Table 1. Collections

| Name | Description |
|------|-------------|
| Seq | The root of all sequences. Shortcut for List() |
| Set | A set of unordered and unique objects |
| Map | A key-value dictionary with unique key |
| IndexedSeq | The root of indexed sequences. Shortcut for Vector() |
| Vector | A list backed by an Array instance for indexed access |
| Range | A range of integers |
| LinearSeq | The root of linear sequences |
| List | A singly linked list of elements |
| Queue | A FIFO list |
| Stack | A LISO list |
| Stream | A lazy list. Elements are added as they are accessed |
| String | A collection of characters |

#### 1. Immutable collections
##### String
   Scala中的`String`类型是在`java.lang.String`基础上加了一个`wrapper`添加了一些Scala的新特性.

* 创建字符串：与`python`类似，Scala可分别使用`" "`和`""" """`创建单行字符串和多行字符串，如

```scala
//example 1：single line

scala> val str = "hello world"
<console>：str: String = hello world

//example 2: multiline

scala> val str_m = """ hello
     | world """
<console>: str_m: String = 
" hello
world "
```

* 转义字符与运算符：Scala字符串同样支持转义字符和一些算术运算符，如`==，+，*`等

```scala
//example 1: escaped characters

scala> val str = "hello \nworld"
<console>: str: String = 
hello
world

//example 2: support math operators

scala> println("hello "*3)
<console>: hello hello hello

scala> val str = "aloha"
<console>: str: String = aloha

scala> str_e + " heja he" == "aloha heja he"
<console>: res0:Boolean = true

scala> val fval: Double = 3.1415926
<console>: fval: Double = 3.1415926
scala> "PI is: " + fval
<console>: res0: String = PI is: 3.1415926 
```

* 字符串内插：Scala中的字符串内插不仅可以插入其他字符串，还可以插入其他类型的`value`和`variable`，还能够使用格式控制. 实现方式包括以下两种：

```scala
//syntax: 类似于shell中的变量引用

s"characters + ${value/variable}" //s开头，${}引用变量，而且{}这里也是可选的


//example

scala> val item = "apple" 
<console>: item: String = apple

scala> s"How do you like them ${item}s?" 
<console>: res0: String = How do you like them apples?

scala> s"Fish n chips n vinegar, ${"pepper "*3}salt" 
<console>: res1: String = Fish n chips n vinegar, pepper pepper pepper salt
```

```scala
//syntax: 类似于c语言中的printf函数

f"characters + ${value/variable}with format" //f开头，在引用变量上加格式控制


//example

scala> val item = "apple" 
<console>: item: String = apple

scala> f"I wrote a new $item%.3s today" 
<console>: res2: String = I wrote a new app today

scala> f"Enjoying this $item ${355/113.0}%.5f times today" 
<console>: res3: String = Enjoying this apple 3.14159 times today
```

##### Tuple classes
   Scala的`Tuple`是一个包含至少两个相同或不同类型值的容器，类似于`python`中的元组类型，实际上，`Tuple`并不属于`collection`类型，其实际上是一系列的`case class`构成的(后面介绍case class)，即每次定义`Tuple`对象时，都是例化了形如`TupleX[Y]`的一个对象，其中`X`取值范围为`1~22`表示内部元素的个数，`Y`则指定了`X`个输入参数，从背后的原理也说明，只能定义不超过22个参数的`Tuple`对象. 之所以和`collection`放在一起，主要考虑都属于多元素的"容器"，而且`Tuple`是介绍的类型中唯一一种能包含不同类型值的类型.
   
* 构造方式：Tuple有三种构造方式，即

```scala
//example 1: 括号内的逗号表达式

scala> val lyric = ("country","road", "take me", 2, "home")
<console>: lyric：(Sring, String, String, Int, String) = (country, road, take me, 2, home) 
```

```scala
//example 2: 使用操作符->构造二元组，是二元组

scala> val lyric = "country" -> 2
<console>: lyric: (String, Int) = (country, 2)  //如果连续多个就会生成嵌套Tuple


scala> val lyric = "country" -> "road" -> 2
<console>: lyric: ((String, String), Int) = ((country, road), 2) //如果这是你想要的，你也可以这么干

```

```scala
//example 3：直接使用TupleX[Y]

scala> val lyric = Tuple3("country", "road", 2)
<console>: lyric: (String, String, Int) = (country, road, 2) 
```

* 访问内部元素：使用`_index`形式访问`Tuple`内部元素，需要**注意**的是`index`起始点为1，如

```scala
//example

scala> val tval = 1 -> "0xff0000"
<console>: tval: (Int, String) = (1, 0xff0000)

scala> tval._1
<console>: res0: Int = 1
```

##### List collection
   List是`immutable`类型中最常用的数据类型，而且`List`支持的大多数方法(函数)，`Set`和`Map`类型基本也都支持. 因此，下面主要通过`List`类型来介绍一些方法的使用.
   
* 构造与访问: 

```scala
//example 1: 使用List直接创建对象

scala> val numbers = List(32, 95, 24, 21, 17)
<console>: numbers: List[Int] = List(32, 95, 24, 21, 17)

scala> val colors = List("red", "green", "blue")
<console>: colors: List[String] = List(red, green, blue)
```

```scala
//example 2: 使用双冒号::操作符创建对象

scala> val numbers = 1 :: 2 :: 3 :: Nil
numbers: List[Int] = List(1, 2, 3)

```

* 成员函数

1）常用函数: head(), tail()

```scala
//example
```

2）高阶函数: foreach(), map(), reduce()

```scala

```

* 进阶特性

* 算术运算

##### Immutable Stack and Queue

##### Map collection
   `Map`类型与C++，Java中的类似，是一个不可变、支持泛型的键值对存储结构，且要求键具有唯一性，支持父类`Iterable`中定义的方法. 
   
* 构造与访问：键值对的关联使用二元`Tuple`的方式创建，即

```scala
//example

scala> val colorMap = Map("red" -> 0xFF0000, "green" -> 0xFF00, "blue" -> 0xFF)
colorMap: scala.collection.immutable.Map[String,Int] = Map(red -> 16711680, green -> 65280, blue -> 255)

scala> val redRGB = colorMap("red") 
redRGB: Int = 16711680

scala> val cyanRGB = colorMap("green") | colorMap("blue") 
cyanRGB: Int = 65535

scala> for (pairs <- colorMap) { println(pairs) } 
(red,16711680) (green,65280) (blue,255)
```

##### Set collection
   `Set`类型与C++，Java中的类似，是一个不可变、无重复元素、无序、支持泛型的复合数据类型，且与`Map`类似，都支持`Iterable`父类中定义的方法.

* 构造与访问

```scala
//example

scala> val unique = Set(10, 20, 30, 20, 20, 10)
unique: scala.collection.immutable.Set[Int] = Set(10, 20, 30)

scala> val sum = unique.reduce( (a: Int, b: Int) => a + b )
sum: Int = 60
```

##### Converting and Matching

* 转型

* 复合类型模式匹配

---

#### 2. Mutable collections

##### Buffer

* 构造方法

* immutable与mutable互转

##### Arrays

##### Seq

##### Streams

##### Option(Monadic)

 Try and Future

---

### VI. Classes

* 定义

* 例化

#### 1. Abstract class

---

#### 2. Anonymous class

---

#### 3. More field and method toyes

---

#### 4. Privacy control

* 关键字

* 私有访问修饰符

---

#### 5. Final and Sealed classes

---

#### 6. Implicit class

---

### VII. Special classes: Objects, Case classes, and Traits

#### 1. Objects

---

#### 2. Case classes

---

#### 3. Traits

---

### VIII. Package

#### 1. Packaging

---

#### 2. Package objects

---

#### 3. Importing instance members

---

### IX. Review Tuple and Function

#### 1. Tuple class

---

#### 2. Function value

---

#### 3. Implicit parameters

---

### X. Advanced features of data type

#### 1. Type aliases

---

#### 2. Abstract types

---

#### 3. Bounded types

---

#### 4. Type variance

---

### XI. Some tips


| Item | Description |
|------|-------------|
| =>操作符应用|  |
| 下划线通配符的应用 |  |
| 可嵌套类型 |  |
| 无名函数/类 |  |
| 函数类型的进化 |  |
| class扩展途径 |  |
| 类内定义的元素 |  |
| 被忽视的符号 |  |

---

**====说明====**


