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
   
* 构造与访问: `List`类型有两种构造方法，如下例所示，`List`属于顺序列表，所以访问内部元素可以直接使用`index`，**注意**，与`Tuple`不同，`List index`的起始值为0.

```scala
//example 1: 使用List直接创建对象

scala> val numbers = List(32, 95, 24, 21, 17)
<console>: numbers: List[Int] = List(32, 95, 24, 21, 17)

scala> val colors = List("red", "green", "blue")
<console>: colors: List[String] = List(red, green, blue)

scala> colors(0)
<console>: res0: String = red
```

```scala
//example 2: 使用双冒号::操作符创建对象

scala> val numbers = 1 :: 2 :: 3 :: Nil
<console>: numbers: List[Int] = List(1, 2, 3)
```

1) ::操作符: 如上所示，使用::与Nil可以创建等价的List对象，::操作符实际上是`List`的成员函数，其接受1个元素作为List对象的"head"元素，主调对象则为"tail"，如下例1，此外，::操作符具有右关联特性，所以追加元素时，需置于操作符左侧，如下例2.

```scala
//example 1: 调用::方法

scala> val first = Nil.::(1) //元素1是新建List的head元素，Nil作为主调对象是新建List的tail元素

<console>: first: List[Int] = List(1)

scala> first.tail == Nil
<console>: res0: Boolean = true
```

```scala
//example 2: ::右关联性

scala> val second = 2 :: first  //左侧添加新元素，成为新的head元素

<console>: second: List[Int] = List(2, 1)

scala> second.tail == first
<console>: res1: Boolean = true
```

2) Nil：`Nil`是一个flag，表明当前位置是`List`最后一个元素的下一个位置，类似于C++顺序容器中的`end()`，`Nil`本身是不可变的，是`List[Nothing]`的`SingleTone`对象，`Nothing`在前一篇的Fig-1中介绍过，是Scala类结构中最底层的类型，所以`List[Nothing]`可以兼容任意类型，所以可以和::操作符一起创建任意类型的`List`.

```scala
//example

scala> val l: List[Int] = List()
<console>: l: List[Int] = List()

scala> l == Nil  //空List就是以Nil结尾的列表

<console>: res0: Boolean = true
```

3) head()与tail()：上面提到了`List`的`head`和`tail`元素，分别表示`List`的左侧的第一个元素，和剩下的所有元素(注意tail不表示列表最右侧元素)，二者对应函数head()与tail().

```scala
example

scala> val colors = List("red", "green", "blue")
<console>: colors: List[String] = List(red, green, blue)

scala> colors.head
<console>: res0: String = red

scala> colors.tail
<console>: res1: List[String] = List(green, blue)
```

* 泛型：从上面的例子中国可以看到，定义的所有`List`对象，其类型都是`List[Type]`的，也就是说Scala中的复合类型像C++等语言一样，也是支持泛型的，而且，我们不仅能够定义如`Int，String`等常规类型的列表，还能定义复合类型的列表(其他collection类型都支持泛型).

```scala
//example 1: List[Tuple]

scala> val keyValues = List(('A', 65), ('B',66), ('C',67))
<console>: keyValues: List[(Char, Int)] = List((A,65), (B,66), (C,67))
```

```scala
//example 2: List[List]

scala> val oddsAndEvents = List(List(1, 3, 5), List(2, 4, 6))
<console>: oddsAndEvents: List[List[Int]] = List(List(1, 3, 5), List(2, 4, 6))
```

* 常用函数: Table 2中罗列了一些`List`常用的成员函数.


Table 2. Common operations on List

| Name | Example | Description |
|------|---------|-------------|
| :+ | List(1,3,4,5) :+ 6, reas: List(1,3,4,5,6) | 左关联操作符，元素从右侧追加，正好与::相反. |
| ::: | List(1,2) ::: List(2,3), res: List(1,2,2,3) | 追加List，右关联. |
| ++ | List(1,2) ++ Set(2,3), res: List(1,2,2,3) | 追加其他collection类型，左关联 |
| == | List(1,2) == List(1,2), res: true | 等价比较，返回布尔.
| distinct | List(1,2,2).distinct, res: List(1,2) | 返回无重复元素版本列表. |
| drop | List('a','b','c') drop 2, res: List('c') | 从列表中去除前2个元素的新列表. |
| dropRight | List('a','b','c') dropRight 2, res: List('a') | drop反向操作 |
| filter | List(23,8,14) filter (_ > 18), res: List(23) | 返回条件过滤后的新列表. |
| flatten | List(List(1,2),List(3,4)).flatten, res: List(1,2,3,4) | 返回多列表元素构成的单一列表. |
| partition | List(1,2,3,4) partition (_ > 3), res: List(List(4),List(1,2,3)) | flatten逆向操作，符合条件的在前. |
| reverse | List(1,2,3).reverse, res: List(3,2,1) | 逆转列表. |
| slice | List(2,3,5,7) slice (1,3), res: List(3, 5) | 截取原列表指定范围内的元素，不包含有边界元素.  |
| sortBy | List("apple", "to") sortBy (_.size), res: List("to","apple")| 按照排序函数对列表排序. |
| sorted | List("apple","to").sorted, res: List("apple","to") | 按照元素类型本身的规则顺序(字母表中a在t前). |
| splitAt | List(2,3,5,7) splitAt 2, res: List(List(2,3),List(5,7)) | 以splitAt指定的参数为界，将列表元素划分为两个List元素列表. |
| take | List(2,3,5,7,11) take 3, res: List(2,3,5) | 返回前3个元素构成的新List. |
| takeRight | List(2,3,5,7,11) takeRight 3, res: List(5,7,11) | take反向操作. |
| zip | List(1,2) zip List("a","b"), res: List((1,"a"),(2,"b")) | 相同index的元素构成tuple，作为列表元素. |
| size | List(1,2,3).size, res: 3 | 返回列表元素数量. |
| isEmpty | List().isEmpty, res: true | 判断列表是否为空，返回布尔. |

上述函数功能基本都很清晰，不做过多解释，有一个有意思的是`++`操作符，这个操作符在其他collection类型中也支持，那么如果将例子中的顺序调换，便会生成合并后的Set对象，如下，说明++操作符的输出类型由主调对象的类型决定，实际上，Scala中的操作符都是函数(方法)，因为Scala中所有的类型都是类，也就是说`A ++ B`背后是由`A.++ B`实现的. 

**通过Scala的类型推断能力，能够使用++构建包含混合类型的的`List`，如例2.**

```scala
//example 1

scala> Set(2,3) ++ List(1,2)
<console>: res0: scala.collection.immutable.Set[Int] = Set(2, 3, 1)
```

```scala
//example 2

scala> List(1,2) ++ Set(" hello")  
<console>: res0: List[Any] = List(1, 2, hello) //因为类型不一致，直接推断出root类型


scala> val t = res0(2) //取出字符串

<console>: t: Any = " hello"  //变成了多态形式，即父类引用指向子类对象

scala> t match {
     | case x: String => s"is ${x}s"
     | case x: Any => s"is${x}A"
     | }
<console>: res1: String = is hellos   //说明Scala可以玩出很多可能性

```

* 高阶函数: collection类型内置了很多高阶函数方法，如Table 2中的`partition, sortBy`, collection遍历函数——`foreach()`等等，就功能角度，可以分为`mapping List`和`reducing List`两大类，即列表映射和列表规约.

1）Mapping List: 列表映射方法中，最基本也是最常用的方法是`map`，`map`方法使用函数字面量参数作用于`List`对象内部的每一个元素，每个元素的输出作为新的`List`元素，即由一个List映射到另一个List，二者具有相同的`size`，只是元素或元素类型不同. 类似的高阶函数映射方法还包括`select，flatMap`.

```scala
//example 1: map

scala> val sizes = colors.map( (c: String) => c.size ) //将原来的String类型元素逐一替换为size方法输出的Int类型元素

<console>: sizes: List[Int] = List(3, 5, 4)
```

```scala
//example 2: select

scala> List(0, 1, 0) collect {case 1 => "ok"} 
<console>: res0: List[String] = List(ok)
```

```scala
//example 3: flatMap

scala> List("milk,tea") flatMap (_.split(','))  //具有map函数映射功能，flat表示由函数字面量输出构成的List内部元素为非复合类型的单一列表

<console>: res1: List[String] = List(milk, tea)
```

2) Reducing List: 列表规约是指将函数字面量参数作用于全部List内部元素，进行统一操作，最终得到唯一的输出，如例1中的`reduce`方法，一些数学规约、布尔规约及通用规约相关的操作如Table 3 ~ Table 5所示.

```scala
//example 1：reduce

scala> val numbers = List(32, 95, 24, 21, 17)
<console>: numbers: List[Int] = List(32, 95, 24, 21, 17)

scala> val total = numbers.reduce( (a: Int, b: Int) => a + b ) //规约就是指按照传入的函数字面量的逻辑，生成唯一的一个输出，这里是求和

<console>: total: Int = 189
```

Table 3. Math reduction ops

| Name | Example | Description |
|------|---------|-------------|
| max | List(1,2,3).max, res: 3 | 返回列表中最大值. |
| min | List(1.1, 2.2, 3.3).min, res: 1.1 | 返回最小值. |
| product | List(1,2,3).product, res: 6 | 返回乘积. |
| sum | List(1,2,3).sum, res: 6 | 求和 |

Table 4. Boolean reduction ops

| Name | Example | Description |
|------|---------|-------------|
| contains | List(1,2,3).contains 2, res: true | 查找 |
| endWith | List(0,1,2,3).endWith List(3,4), res: false | 确认是否以给定List结尾 |
| exists | List(24,17,22) exits (_ < 18), res: true | 确认列表中是否存在满足函数字面量的元素 |
| forall | List(24,17,22) forall (_ < 18), res: false | 确认列表中全部元素是否满足函数字面量的定义 |
| startsWith | List(0,4,3) startsWith List(0), res: true | 确认列表是否以指定列表开始 |

Table 5. Generic list reduction ops

| Name | Example | Description |
|------|---------|-------------|
| fold | List(4,5,6).fold(1)(_ + _), res: 16 | 参数组1中给出了运算起始值，参数组2则指定了规约运算，即从1开始将列表元素累加 |
| foldLeft | List(4,5,6).foldLeft(1)(_ + _), res: 16 | 类似于fold方法，但运算顺序是从左至右，对假发不显著 |
| foldRight | List(4,5,6).foldRight(1)(_ + _), res: 16 | foldLeft的逆向运算 |
| reduce | List(4,5,6).reduce(_ + _), res: 15 | 默认0为起始 |
| reduceLeft | List(4,5,6).reduceLeft(_ + _), res: 15 | 从左至右 |
| reduceRight | List(4,5,6).reduceRight(_ + _), res: 15 | 从右至左 |
| scan | List(4,5,6).scan(0)(_ + _), res: List(0, 4, 9, 16) | 给定起始值与列表元素逐一相加，输出值构成新的列表 |
| scanLeft |  List(4,5,6).scanLeft(0)(_ + _), res: List(0, 4, 9, 16) | 从左至右 |
| scanRight |  List(4,5,6).scanRight(0)(_ + _), res: List(15, 11, 6, 0) | 从右至左 | 

表中介绍的三类操作实际上差不多，但既然同时存在于Scala中，应该是在不同应用域下有不同的限制，作者没有深挖，也不想深挖，毕竟不是要做Scala程序员，只是掌握基础核心内容罢了. 另外，注意到三类操作都有左右顺序之分，这一方面是简化一些特殊运算的形式，另一方面是因为`List`属于链式存储结构，学过数据结构的都知道，链式存储相较于顺序存储，其删除和添加的性能都非常高，但是查找操作效率很低，所以左右操作顺序代表了不同的性能需求，不想当Scala程序员的话，就别管它了.

##### Immutable Stack and Queue

##### Map collection
   `Map`类型(注意与map方法相区分)与C++，Java中的类似，是一个不可变、支持泛型的键值对存储结构，且要求键具有唯一性，支持父类`Iterable`中定义的方法.
   
* 构造与访问：键值对的关联使用二元`Tuple`的方式创建，即

```scala
//example

scala> val colorMap = Map("red" -> 0xFF0000, "green" -> 0xFF00, "blue" -> 0xFF) //构造

<console>: colorMap: scala.collection.immutable.Map[String,Int] = Map(red -> 16711680, green -> 65280, blue -> 255)

scala> val redRGB = colorMap("red") //通过key访问value

<console>: redRGB: Int = 16711680

scala> val cyanRGB = colorMap("green") | colorMap("blue") //算术运算

<console>: cyanRGB: Int = 65535

scala> for (pairs <- colorMap) { println(pairs) } //应用于for循环

<console>: (red,16711680) (green,65280) (blue,255)
```

##### Set collection
   `Set`类型与C++，Java中的类似，是一个不可变、无重复元素、无序、支持泛型的复合数据类型，且与`Map`类似，都支持`Iterable`父类中定义的方法.

* 构造与访问: `Set`的构造很简单，直接调用`Set`例化即可，但对`Set`内部元素的访问，由于其属于无序集合，所以不能像`List(index)`那样直接访问，`Set(item)`等同于调用了`contains`函数，是判断`item`是否存在于Set中，所以可以通过查找元素的方式访问，也可以通过迭代整个集合访问内部元素.

```scala
//example

scala> val unique = Set(10, 20, 30, 20, 20, 10)
unique: scala.collection.immutable.Set[Int] = Set(10, 20, 30)

scala> unique(0)
<console>: res0: Boolean = false //0并未在集合中

scala> unique(10)
<console>: res1: Boolean = true

scala> val sum = unique.reduce( (a: Int, b: Int) => a + b )  //支持高阶函数

<console>: sum: Int = 60

scala> unique.foreach((i: Int) => println(i))  //使用高阶函数进行遍历

<console>:
10
20
30
```

##### Matching

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

### IX. Review Function

#### 1. Function value

---

#### 2. Implicit parameters

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
