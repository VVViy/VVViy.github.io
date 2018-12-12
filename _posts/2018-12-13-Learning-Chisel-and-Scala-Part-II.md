---
layout:     post
title:      Learning Chisel and Scala
subtitle:   Scala Part II
date:       2018-12-12
author:     Max
header-img: img/post-gray-background.jpg
catalog: true
tags:
    - Chisel
    - Scala
---   

### V. Advanced Data Type: Collections
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

1）::操作符: 如上所示，使用::与Nil可以创建等价的List对象，::操作符实际上是`List`的成员函数，其接受1个元素作为List对象的"head"元素，主调对象则为"tail"，如下例1，此外，::操作符具有右关联特性，所以追加元素时，需置于操作符左侧，如下例2.

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

2）Nil：`Nil`是一个flag，表明当前位置是`List`最后一个元素的下一个位置，类似于C++顺序容器中的`end()`，`Nil`本身是不可变的，是`List[Nothing]`的`SingleTone`对象，`Nothing`在前一篇的Fig-1中介绍过，是Scala类结构中最底层的类型，所以`List[Nothing]`可以兼容任意类型，所以可以和::操作符一起创建任意类型的`List`.

```scala
//example

scala> val l: List[Int] = List()
<console>: l: List[Int] = List()

scala> l == Nil  //空List就是以Nil结尾的列表

<console>: res0: Boolean = true
```

3）head()与tail()：上面提到了`List`的`head`和`tail`元素，分别表示`List`的左侧的第一个元素，和剩下的所有元素(注意tail不表示列表最右侧元素)，二者对应函数head()与tail().

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
| :: | 3 :: List(1,2), res: List(3,1,2) | 右关联操作符，元素左侧追加 |
| :+ | List(1,3,4,5) :+ 6, reas: List(1,3,4,5,6) | 左关联操作符，元素从右侧追加，正好与::相反 |
| ::: | List(1,2) ::: List(2,3), res: List(1,2,2,3) | 追加List，右关联 |
| ++ | List(1,2) ++ Set(2,3), res: List(1,2,2,3) | 追加其他List或collection类型，左关联 |
| == | List(1,2) == List(1,2), res: true | 等价比较，返回布尔 |
| distinct | List(1,2,2).distinct, res: List(1,2) | 返回无重复元素版本列表 |
| drop | List('a','b','c') drop 2, res: List('c') | 从列表中去除前2个元素的新列表 |
| dropRight | List('a','b','c') dropRight 2, res: List('a') | drop反向操作 |
| filter | List(23,8,14) filter (_ > 18), res: List(23) | 返回条件过滤后的新列表 |
| flatten | List(List(1,2),List(3,4)).flatten, res: List(1,2,3,4) | 返回多列表元素构成的单一列表 |
| partition | List(1,2,3,4) partition (_ > 3), res: List(List(4),List(1,2,3)) | flatten逆向操作，符合条件的在前 |
| reverse | List(1,2,3).reverse, res: List(3,2,1) | 逆转列表 |
| slice | List(2,3,5,7) slice (1,3), res: List(3, 5) | 截取原列表指定范围内的元素，不包含有边界元素  |
| sortBy | List("apple", "to") sortBy (_.size), res: List("to","apple")| 按照排序函数对列表排序 |
| sorted | List("apple","to").sorted, res: List("apple","to") | 按照元素类型本身的规则顺序(字母表中a在t前) |
| splitAt | List(2,3,5,7) splitAt 2, res: List(List(2,3),List(5,7)) | 以splitAt指定的参数为界，将列表元素划分为两个List元素列表 |
| take | List(2,3,5,7,11) take 3, res: List(2,3,5) | 返回前3个元素构成的新List |
| takeRight | List(2,3,5,7,11) takeRight 3, res: List(5,7,11) | take反向操作 |
| zip | List(1,2) zip List("a","b"), res: List((1,"a"),(2,"b")) | 相同index的元素构成tuple，作为列表元素 |
| size | List(1,2,3).size, res: 3 | 返回列表元素数量 |
| isEmpty | List().isEmpty, res: true | 判断列表是否为空，返回布尔 |

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

2）Reducing List: 列表规约是指将函数字面量参数作用于全部List内部元素，进行统一操作，最终得到唯一的输出，如例1中的`reduce`方法，一些数学规约、布尔规约及通用规约相关的操作如Table 3 ~ Table 5所示.

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
| max | List(1,2,3).max, res: 3 | 返回列表中最大值 |
| min | List(1.1, 2.2, 3.3).min, res: 1.1 | 返回最小值 |
| product | List(1,2,3).product, res: 6 | 返回乘积 |
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
  栈和队列是我们很熟悉的数据结构，写软件的小伙伴几乎天天打交道，二者应用特性决定了可变应用价值，所以这里的`immutable`版本栈和队列没什么实际用处，实际上，Scala中的`immutable`和`mutable`中的复合数据类型都是可以相互转化的，在介绍可变类型时将介绍相互转换. 此外，在Fig-1中还有很多的immutable类型复合类型，如红黑树，这里就不一一介绍了，如果后续发现`RISC-V`相关项目源码中使用到了本文未介绍的复合结构，会增量添加进来.

##### Map collection
   `Map`类型(注意与map方法相区分)与C++，Java中的类似，是一个不可变、支持泛型的键值对存储结构，且要求键具有唯一性，上面介绍的`List`操作，`Map`也基本都支持，还有一些常用的方法如Table 6所示.
   
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

Table 6. Common Map operations

| Name | Example | Dscription |
|------|---------|------------|
| get | Map(1->"st", 2->"nd") get 1/Map(1->"st", 2->"nd")(1), res: st | 由键访问值 |
| getOrElse | Map(1->"st", 2->"nd") getOrElse ("rd", 2), res: 2 | 根据"rd"查找，找到返回对应值，否则返回默认值2 |
| contains | Map(1->"st", 2->"nd") contains 2, res: true | 判断2是否为Map键 |
| + | Map(1->"st", 2->"nd") + (3->"rd")/Map(1->"st", 2->"nd") ++ (3->"rd", 4->"th"), res: Map(1->"st", 2->"nd",3->"rd")/Map(1->"st", 2->"nd",3->"rd",4->"th") | 追加元素或子map |
| ++ | Map(1->"st", 2->"nd") ++ List((3,"rd")), res: Map(1->"st", 2->"nd", 3->"rd") | 追加其他Map或复合类型元素 |
| - | Map(1->"st", 2->"nd") - 1/Map(1->"st", 2->"nd",3->"rd") - (1,2), res: Map(2->"nd")/Map(3->"rd") | 根据键删除元素 |
| -- | Map(1->"st", 2->"nd") -- List(1), res: Map(2->"nd") | 删除其他复合类型指定的键所对应的Map元素 |
| keys | Map(1->"st", 2->"nd").keys, res: Set(1,2) | 返回键构成的集合 |
| values | Map(1->"st", 2->"nd").values, res: MapLike.DefaultValuesIterable(1, 2, 3) | 由值构成Iterable集合 |

这里我们要再讨论一下`++`，这个操作符真是很有意思，前面我们聊过`List ++ Set`的组合形式，最终的类型取决于主调对象的类型，但是对于`Map`还有进一步的推断，看下例. 由于`Map`内部元素的构造本身就是通过`二元Tuple`实现的，所以其他以`二元Tuple`为元素的复合类型都能够与`Map ++`，而不改变类型，一旦元素结构不符合二元元组结构，这时就会将`Map`内部元素向其他能够以二元元组作为元素的复合类型转化.

```scala
//example 1

scala> Map(1->"st", 2->"nd") ++ Map(3->"rd") //同类型++，输出依然是Map

<console>: res0: scala.collection.immutable.Map[Int,String] = Map(1 -> st, 2 -> nd, 3 -> rd)  

//example 2

scala> Map(1->"st", 2->"nd") ++ List((3，"rd")) //List类型内部元素为Tuple，输出也是Map

<console>: res1: scala.collection.immutable.Map[Int,String] = Map(1 -> st, 2 -> nd, 3 -> rd)  

//example 3
scala> Map(1->"st", 2->"nd") ++ List(3, "rd") //List内部元素不是Tuple了，输出变成了List，且原Map中的元素在List中变成了二元元组

<console>: res2: scala.collection.immutable.Iterable[Any] = List((1,st), (2,nd), 3, rd)  

```

##### Set collection
   `Set`类型与C++，Java中的类似，是一个不可变、无重复元素、无序、支持泛型的复合数据类型，且与`Map`类似，支持`List`中介绍的方法，其他一些常用方法如Table 7所示.

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

Table 7. Common Set operations

| Name | Example | Description |
|------|---------|-------------|
| contains | Set(1,2,3) contains 4, res: false | 判断元素存在性，等价于index查找 |
| subsetOf | Set(1,2) subsetOf Set(1,2,3), res: true | 判断子集存在性 |
| + | Set(1,2) + 3/Set(1,2) + (3,4), res: Set(1,2,3)/Set(1,2,3,4) | 追加元素或子集合 |
| ++ | Set(1,2) ++ List(2,3), res: Set(1,2,3) | 追加其他Set或复合类型元素 |
| - | Set(1,2,3) - 2/Set(1,2,3) - (1,2), res: Set(1)/Set(3) | 删除原有元素，构成新Set |
| -- | Set(1,2,3) -- List(1,2), res: Set(3) | ++逆向操作 |
| empty | Set(1,2,3).empty, res: Set() | 清空集合构成新Set |
| & 或 intersect | Set(1,2,3) & Set(1) 或 Set(1,2,3) intersect Set(1), res: Set(1) | 求交集 |
| \| 或 union | Set(1,2,3) \| Set(4) 或 Set(1,2,3) union Set(4), res: Set(1,2,3,4) | 求并集 |
| &~ 或 diff | Set(1,2,3) &~ Set(3) 或 Set(1,2,3) diff Set(3), res: Set(1,2) | 去交集，求补集 |

##### Casting and Matching

* 转型：scala中提供了一些方法能够在`List, Set, Map, String`间转化，即

Table 8. Collections casting operations

| Name | Example | Description |
|------|---------|-------------|
| mkString | List(1,2,3).mkString("$ "), res: 1$ 2$ 3 | 用List中的元素和mkString中定义的分隔符生成字符串 |
| toString | List(1,2,3).toString, res: String = List(1,2,3) | 转换为字符串, 例子中的List是字符，不是类型名 |
| toMap | Set(1->true, 2->true).toMap, res: Map(1->true, 2->true) | —— |
| toSet | List(1,2,2,3).toSet, res: Set(1,2,3) | —— |
| toList | Map("a"->1, "b"->2).toList, res: List(("a",1),("b",2)) | —— |

* 模式匹配：本节介绍将复合类型应用于前一篇blog介绍的`match`逻辑表达式中，从下面的例子可以看到，复合类型作为元素的集合，与`match`结合使用非常灵活.

```scala
//example 1: 元素匹配

scala> val statuses = List(500, 404)
<console>: statuses: List[Int] = List(500, 404)

scala> val msg = statuses.head match {  //使用List中的首元素进行匹配，无特别之处

     | case x if x < 500 => "okay"
     | case _ => "whoah, an error"
     | }
<console>: msg: String = whoah, an error
```

```scala
//example 2: 使用collection方法做模式匹配

scala> val msg = statuses match {
     | case x if x contains(500) => "has error" //在guard中使用contains方法
     | case _ => "okay"
     | }
<console>: msg: String = has error
```

```scala
//example 3: 匹配collection对象

scala> val msg = statuses match {
     | case List(404, 500) => "not found & error" //List作为匹配参数
     
     | case List(500, 404) => "error & not found"
     | case List(200, 200) => "okay"
     | case _ => "not sure what happened"
     | }
<console>: msg: String = error & not found
```

```scala
//example 4: 值绑定

scala> val msg = statuses match {
     | case List(500, x) => s"Error followed by $x" //x与404绑定
     
     | case List(e, x) => s"$e was followed by $x"
     | }
<console>: msg: String = Error followed by 404
```
---

```scala
//example 5: head + tail分裂匹配

scala> val head = List('r','g','b') match {
     | case x :: xs => x   //x与List('r','g','b') 的head绑定，xs与tail绑定
     
     | case Nil => ' '
     | }
<console>: head: Char = r
```

  前面介绍`Tuple`时提到过，其不是`collection`体系下的类型，但是其行为与`collections`类型非常相似，所以`Tuple`也可应用于`match`控制结构，`Tuple`可以保存不同类型的元素，在有些应用场景下更加灵活.
  
```scala
//exampe

scala> val code = ('h', 204, true) match {
     | case (_, _, false) => 501
     | case ('c', _, true) => 302
     | case ('h', x, true) => x
     | case (c, x, true) => {
     |   println(s"Did not expect code $c")
     |   x
     |  }
     | }
<console>: code: Int = 204
```

#### 2. Mutable collections
  前一节介绍了不可变的复合类型，对应的也有可变类型版本，如Table 9所示. 需要再次说明的是，`collection.immutable package`是自动添加到当前命名空间内的，而`collection.mutable package`未自动添加左右在未`import package`的情况下，只能写出完整包名进行引用类型.
  
Table 9. Mutable collection types

| Immutable type | Mutable type |
|----------------|--------------|
| collection.immutable.List | collection.mutable.Buffer |
| collection.immutable.Set | collection.mutable.Set |
| collection.immutable.Map | collection.mutable.Map |

##### Buffer, Set, and Map

* 构造方法：可变类型的构造方式有三种：类名创建、使用不可变类型转换以及使用`collection builer`.

```scala
//example 1.1: 直接例化

scala> val nums = collection.mutable.Buffer(1) //创建包含一个元素的可变列表

<console>: nums: scala.collection.mutable.Buffer[Int] = ArrayBuffer(1)

scala> for (i <- 2 to 10) nums += i //为可变列表添加元素

scala> println(nums)
<console>: Buffer(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

//example 1.2

scala> val nums = collection.mutable.Buffer[Int]() //构建空列表，由于不包含初始元素，无法进行类型推断，所以必须在创建的时候指明类型

<console>: nums: scala.collection.mutable.Buffer[Int] = ArrayBuffer()

scala> for (i <- 1 to 10) nums += i
scala> println(nums)
Buffer(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```

```scala
//example 2: immutable与mutable相互转化, List, Set, Map都可以使用toBuffer转换为Buffer类型

scala> val conv = Map("a"->1,"b"->2).toBuffer  //Map转换为Buffer

<console>: conv: scala.collection.mutable.Buffer[(String, Int)] = ArrayBuffer((a,1), (b,2))  //Map内部元素为二元Tuple


//相互转化

scala> conv.toMap  
<console>: res0: scala.collection.immutable.Map[String,Int] = Map(a -> 1, b -> 2)

scala> conv.toList
<console>: res1: List[(String, Int)] = List((a,1), (b,2))

scala> conv.toSet
<console>: res2: scala.collection.immutable.Set[(String, Int)] = Set((a,1), (b,2))

```

```scala
//example 3：collection builder, 若创建mutable对象的目的是为了转化为immutable对象可以考虑该方法

scala> val nSet = Set.newBuilder[Char] //第一步：任意不可变类型使用newBuilder方法创建一个Builder(实际上builder是一个简化形式的Buffer)

<console>: nSet: scala.collection.mutable.Builder[Char,scala.collection.immutable.
Set[Char]] = scala.collection.mutable.SetBuilder@726dcf2c

scala> nSet += 'h'  //第二步：nSet此时是一个"可变类型对象"，所以可以添加元素

<console>: res0: nSet.type = scala.collection.mutable.SetBuilder@d13d812

scala> nSet ++= List('e', 'l', 'l', 'o')
<console>: res1: nSet.type = scala.collection.mutable.SetBuilder@d13d812

scala> val helloSet = nSet.result //第三步：调用result方法，转化为immutable

<console>: helloSet: scala.collection.immutable.Set[Char] = Set(h, e, l, o)
```

##### Arrays
  `Array`与`String`类似，并不是Scala中定义的类型，而是对Java的Array基础上加了一层`wrapper`，`Array`本身是一个固定尺寸，内部元素可变，`index-based`的复合类型，而且支持`Iterable`分支下定义的方法.
  
```scala
//example

scala> val colors = Array("red", "green", "blue")
<console>: colors: Array[String] = Array(red, green, blue)

scala> colors(0) = "purple"  //起始index为0

scala> colors  //调用默认的toString方法

<console>: res0: Array[String] = Array(purple, green, blue)
```

##### Seq
  在Fig-1中可以看到，`Seq`是序列类型的根几点，本身为抽象类不可例化对象，而若使用`Seq`例化对象实际上会例化`List`对象，所以也可以将`Seq`看作是`List`的"快捷键"，类似的，`IndexedSeq`也是`Vector`类型的快捷键.
  
```scala
//example

scala> val links_1 = Seq('C','M','Y','K')
<console>: links_1: Seq[Char] = List(C, M, Y, K)

scala> val links_2 = IndexedSeq(1,2,3,43)
<console>: links_2: IndexedSeq[Int] = Vector(1, 2, 3, 43) //Vector类似于C++中的vector，是一个动态数组


scala> links_2(3)  //数组index起始值为0

<console>: Int = 43
```

##### Streams
  在上一篇blog开头介绍了一种特殊的`lazy value`，其初始化过程是在第一次访问时进行的，而非定义时，同样地，`Streams`类型是一个`lazy collection`，其由起始元素加一个生成元素的迭代函数两部分构成. 在定义`Stream`类型对象时，不会主动生成元素，只有在后续访问时才会生成指定数量的元素，且每一个元素只在第一次访问时生成一次，之后不会重复生成. 创建`Streams`类型对象有两种方式，即调用`Stream.cons`和使用`#::`操作符.
  
```scala
//example 1: 调用Stream.cons创建无界Streams

scala> def inc(i: Int): Stream[Int] = Stream.cons(i, inc(i+1)) //通过Stream.cons创建Stream，第一个参数是生成元素值，第二个是生成器

<console>: inc: (i: Int)Stream[Int]

scala> val s = inc(1) //指明尺寸为1

<console>: s: Stream[Int] = Stream(1, ?) //？表示可以"无穷"生成元素


scala> val l = s.take(5).toList  //调用Stream的take方法生成指定尺寸stream，这里的1已经生成，2~5在本次访问时产生

<console>: l: List[Int] = List(1, 2, 3, 4, 5)

scala> s
<console>: res1: Stream[Int] = Stream(1, 2, 3, 4, 5, ?)
```
```scala
//example 2: 使用#::操作符创建Streams，右关联

scala> def inc(head: Int): Stream[Int] = head #:: inc(head+1)
<console>: inc: (head: Int)Stream[Int]

scala> inc(10).take(10).toList
<console>: res0: List[Int] = List(10, 11, 12, 13, 14, 15, 16, 17, 18, 19)
```

```scala
//example 3: 使用Stream.empty创建有界Streams

scala> def to(head: Char, end: Char): Stream[Char] = (head > end) match { //加了结尾参数进行限定

     | case true => Stream.empty
     | case false => head #:: to((head+1).toChar, end)
     | }
<console>: to: (head: Char, end: Char)Stream[Char]

scala> val hexChars = to('A', 'F').take(20).toList
<console>: hexChars: List[Char] = List(A, B, C, D, E, F)
```

##### Option(Monadic)
  最后介绍的复合类型是"SingleTon collection"，即这类复合类型内部至多有一个元素，是一种非白即黑的二元对立逻辑，称为`monadic collections`，`Option，Try，and Future`都属于此类，其中，`Try`和其他高级编程语言类似，主要用于异常处理，这里不作介绍，用到再查都可以，`Future`的作用是使具体任务在后台运行，类似于Shell下`&`修饰的任务，所以这里主要介绍`Option`.

* 定义：假设我们要判断一个元素是否存在与一个集合中，那么只有两种结果：存在，不存在，这就是`Option`类似所描述的，即判断一个元素是否存在，诸如此类的二元判断. 但`Option`是一个抽象类，是不可例化的，所以真正做判断的是其两个子类：`Some`和`None`，前者是泛型类型，包含一个值，意为"存在"，后者为空.

```scala
//example 1

scala> var x: String = "Indeed"
<console>: x: String = Indeed

scala> var a = Option(x) //判断x是否为空

<console>: a: Option[String] = Some(Indeed) //Some指明不为空，且值为Indeed

scala> x = null
x: String = null

scala> var b = Option(x)
b: Option[String] = None //None判断为空
```

```
//example 2: 使用isDefined和isEmpty做等价判断

scala> println(s"a is defined? ${a.isDefined}")
<console>: a is defined? true

scala> println(s"b is not defined? ${b.isEmpty}")
<console>: b is not defined? true
```

* 应用：Option常取代"null"，用作安全运算检查，即先检查运算是否合法，再决定是否使用运算结果.

```scala
//example 1

scala> def divide(amt: Double, divisor: Double): Option[Double] = { //返回类型为Option，这样检查返回值类型便知是否为合法运算

     | if (divisor == 0) None
     | else Option(amt / divisor)
     | }
<console>: divide: (amt: Double, divisor: Double)Option[Double]

scala> val legit = divide(5, 2)
<console>: legit: Option[Double] = Some(2.5)

scala> val illegit = divide(3, 0)
<console>: illegit: Option[Double] = None
```

```scala
//example 2: scala为一些类型内置了Option选项的方法，如读取空列表的head元素，可以使用Table

scala> List().head  //直接使用head会报异常

<console>: java.util.NoSuchElementException: head of empty list
  at scala.collection.immutable.Nil$.head(List.scala:426)
  at scala.collection.immutable.Nil$.head(List.scala:423)
  ... 28 elided

scala> List().headOption  //使用headOption则有特定返回值判断

res1: Option[Nothing] = None
```

* 访问元素：在上节中判断运算合法性后，还是要取出结果，可以使用Table 10中安全方式.

```scala
//example

scala> def divide(amt: Double, divisor: Double): Option[Double] = {
     | if (divisor == 0) None
     | else Option(amt / divisor)
     | }
<console>: divide: (amt: Double, divisor: Double)Option[Double]

scala> val legit = divide(5, 2)
<console>: legit: Option[Double] = Some(2.5)

scala> val illegit = divide(3, 0)
<console>: illegit: Option[Double] = None
```

Table 10. Safe Option extractions

| Name | Example | Description |
|------|---------|-------------|
| fold | divide(3,0).fold(-1)(x => x), res: -1 | 根据返回值判断是否为None，若为None则返回指定值-1，否则进行函数字面量参数指定的运算，本例直接返回x，Table 5中的方法都可类似应用 |
| getOrElse | divide(3,0) getOrElse -1, res: -1 | 类似的检查函数返回值是否为None，为None则输出指定的默认值-1，否则返回实际结果 |
| match expression | divide(3,0) match { case Some(x) => x; case None => -1}, res: -1 | —— |

---

### VI. Classes
  作者假设路过的小伙伴有面向对象基础，基础内容就不介绍了，直接介绍定义和有差异的内容.
  
* 定义：与其他高级语言相比，Scala类定义形式除了构造函数，其他并无特别之处，其完整定义形式如下，除了`class <identifier>`其他参数都是可选的，下面逐一讲解其中的参数.

```scala
//syntax

class <identifier> [type-parameters]
                   [([val|var] <identifier>: <type> = <expression>[, ... ])]
                   [extends <identifier>[type-parameters](<input parameters>)]
                   [{ fields and methods }]                          
```

1）继承与类成员: 用如下形式便能定义一个简单的类，在最后的类成员中前两个是常见的类属性(val和var)和类方法，最后还有个嵌套类，Scala支持嵌套类，实际上，在Scala中，**表达式、函数和类之间都是可以相互嵌套的**，嵌套类可以访问其父类.

```scala
//syntax

class <identifier> [extends <identifier>] [{ fields, methods, and classes }]

//example 1：简单类与实例

scala> class Child extends Parent {val m_value; var m_variable; def foo = "fool"; class Nest {println("This is a nested class")}}
<console>: defined class Child

scala> val lm = new Child  //例化对象也使用new关键字

<console>: lm: Child = Child@memory_addr

//example 2: class nested in expression

scala> val te = {val tes = 18; class Test {var est = tes + 3}}
<console>: te: Unit = ()
```

2）类参数：通过如下形式定义类参数可以作为构造参数初始化类内参数，若未使用`val/var`，则参数仅在实例化阶段使用，之后不可访问，若使用，则构成了一个域，初始化后可以继续访问，而且可以设置默认参数，比较方便的一种方式是将类属性作为类参数进行声明，若父类包含类参数，继承时要提供相应初始化参数，需要**注意**的是在实例化带有类参数的对象时，必须提供参数，即没有C++那样的默认构造函数，若类本身无参数，实例化时可省略括号.

```scala
//syntax

class <identifier> ([val|var] <identifier>: <type> = <expression>[, ... ])
                   [extends <identifier>(<input parameters>)]
                   [{ fields and methods }]
                   
//example 1：无默认参数

scala> class Car(val make: String, var reserved: Boolean) {
     | def reserve(r: Boolean): Unit = { reserved = r }
     | }
<console>: defined class Car

scala> val t = new Car("Toyota", reserved = false) //对于混合提供类参数，顺序遵守先位置参数，后命名参数，且命名参数顺序任意

<console>: t: Car = Car@4eb48298

scala> val tt = new Car  //实例化时必须提供参数

<console>:12: error: not enough arguments for constructor Car: (make: String, reserved: Boolean)Car.
Unspecified value parameters make, reserved.
       val tt = new Car
       
scala> t.reserved = true //访问类成员使用周期符号"."

<console>: t.reserved: Boolean = true
```

```scala
//example 2: 为父类提供类参数

scala> class Lotus(val color: String, reserved: Boolean) extends Car("Lotus", reserved)
<console>: defined class Lotus
```

```scala
//example 3: 设置类参数默认值

scala> class Car(val make: String, var reserved: Boolean = true, val year: Int = 2015) {
     | override def toString = s"$year $make, reserved = $reserved"
     | }
<console>: defined class Car

scala> val l = new Car("Lexus", year = 2010) //第二个默认为位置参数，第三个参数必须使用命名参数

<console>: l: Car = 2010 Lexus, reserved = true
```

3）泛型：定义形式便是开头介绍的类的完整定义形式.

```scala
//example

scala> class Singular[A](element: A) extends Traversable[A] {
     | def foreach[B](f: A => B) = f(element)
     | }
<console>: defined class Singular

scala> val p = new Singular[String]("Planes") //也可省略"[String]"，通过编译器的类型推断功能指定A

<console>: p: Singular[String] = (Planes)
```

Table 11. Common key words of class definition

| Name | Example |Description |
|------|---------|------------|
| new | new AClass | 例化对象 |
| extends | class Child extends parent | 与Java一样，继承关键字 |
| override | override def toString = ... | 重写父类方法关键字 |
| this | —— | 与Java/C++一样，表示指向当前对象的引用/指针 |
| super | —— | 与Java一样，表示指向当前对象内父类对象的引用 |

#### 1. Abstract class
  Scala沿用了Java的抽象类，抽象类声明或定义了若干核心属性和方法，使用`abstract`关键字定义，抽象类不可例化，是实现多态的基础机制. 以抽象类为父类的子类要为抽象类中只提供声明部分的属性和方法添加实现，否则，即使子类未显式声明为`abstract`，依然为抽象类. 虽然抽象类中也可以定义属性和方法，但抽象类一般作为"一个类别"的超类存在，所以很少定义成员，若定义了类成员，子类可以选择不重新定义.
  
```scala
//example

scala> abstract class Car {
     | val year: Int                 //声明，无定义
     
     | val automatic: Boolean = true //定义
     
     | def color: String             //声明，无定义
     
     | }
<console>: defined class Car

scala> new Car() //抽象类不可例化
<console>:9: error: class Car is abstract; cannot be instantiated new Car()

scala> class RedMini(val year: Int) extends Car { //子类重写

     | def color = "Red"
     | }
<console>: defined class RedMini

scala> val m: Car = new RedMini(2005)
<console>: m: Car = RedMini@5f5a33ed
```

---

#### 2. Anonymous class
  在上一篇blog介绍函数时，我们介绍过一种"无名函数"，给那些复用率不高，但还需要一些处理逻辑的情况，在类中，也有这样一种一次性的"无名类"，其实际为无名子类，即在实例化的同时，实现(对于抽象父类)或重写(对于一般非抽象类)了父类方法，不得不说这个Scala的这个操作很方便(父类方法中不能有类参数, 作者使用了几种写法均为成功建立带参数的无名子类，有成功的小伙伴记得评论).

```scala
//example 1: 抽象父类

scala> abstract class Listener { def trigger }
<console>: defined class Listener

scala> val myListener = new Listener {  //实现方式：在实例化语句后紧跟父类待实现或重写的方法定义. 实际上，这是个两步走过程，
                                        //第一步编译器生成包含重定义方法的自动化子类，第二步例化该自动化子类对象

     | def trigger { println(s"Trigger at ${new java.util.Date}") }
     | }
<console>: myListener: Listener = $anon$1@59831016
```

```scala
//example 2: 非抽象父类

scala> class A { def foo = "i\'m " + "father"} //不能使用类参数

<console>: defined class A

scala> val anon = new A {override def foo = "i\'m " + "child"}
<console>: anon: A = $anon$1@3e65c397

scala> anon.foo
<console>: res0: String = i'm child
```

```scala
//example 3: 观察者模式更方便

scala> abstract class Listener { def trigger }
<console>: defined class Listener

scala> class Listening {
     | var listener: Listener = null
     | def register(l: Listener) { listener = l } //将无名类定义放到函数调用中
     
     | def sendNotification() { listener.trigger }
     | }
<console>: defined class Listening

scala> val notification = new Listening()
<console>: notification: Listening = Listening@66596c4c

scala> notification.register(new Listener {  //无名类对象参数

     | def trigger { println(s"Trigger at ${new java.util.Date}") }
     | })
```

---

#### 3. More field and method types
* 函数重载：函数重载是指一系列具有不同输入参数列表(输入参数数量不同或类型不同)的同名函数.

```scala
//example

scala> class Printer(msg: String) {
     | def print(s: String): Unit = println(s"$msg: $s")
     | def print(l: Seq[String]): Unit = print(l.mkString(", "))
     | }
<console>: defined class Printer

scala> new Printer("Today's Report").print("Foggy" :: "Rainy" :: "Hot" :: Nil)
<console>: Today's Report: Foggy, Rainy, Hot
```

* apply函数：该函数是Scala中应用非常广泛的一类函数，`Chisel API`中有大量的`apply`函数. `apply`函数在调用时可以省略函数名.

```scala
//example
scala> class Multiplier(factor: Int) {
     | def apply(input: Int) = input * factor //定义apply函数
     
     | }
<console>: defined class Multiplier

scala> val tripleMe = new Multiplier(3)

<console>: tripleMe: Multiplier = Multiplier@339cde4b

scala> val tripled = tripleMe.apply(10) //显示调用

<console>: tripled: Int = 30

scala> val tripled2 = tripleMe(10)      //使用"对象名()"间接调用apply函数

<console>: tripled2: Int = 30
```

* lazy value: 在上一篇blog开头就介绍了惰性value，只是那里没有给出合适的应用，这里结合类来看其应用意义. 先回顾一下惰性值的特征，对于一个类，其属性成员一般是在例化对象时进行初始化，但`lazy val`并不遵守此规，其有且只有在第一次被访问时才会被初始化.

```scala
//example

scala> class RandomPoint {
     | val x = { println("creating x"); util.Random.nextInt }
     | lazy val y = { println("now y"); util.Random.nextInt }  //定义lazy value
     | }
<console>: defined class RandomPoint

scala> val p = new RandomPoint() //实例化时，会为对象内类成员分配内存空间并初始化，但lazy value未在其中

<console>: creating x
p: RandomPoint = RandomPoint@6c225adb

scala> println(s"Location is ${p.x}, ${p.y}") //当lazy value被访问时，才被初始化

now y
<console>: Location is 2019268581, -806862774

scala> println(s"Location is ${p.x}, ${p.y}") //第一次初始化后，值便稳定

<console>: Location is 2019268581, -806862774
```

---

#### 4. Packaging
  
* 源码打包：Scala沿用了Java基于`package`的代码组织方式，按照目录层级安置源码，代码打包的方式有两种.

```scala
//syntax: 第一种是在源文件开头加入package关键字的包定义，<identifier>的命名应该按照Java的定义，即机构属性+机构名+功能分类，如com.netflix.utilities.

package <identifier>
source code

//example

package bobsrockets.navigation //源文件开头的包定义

class Navigator ...  //源码

```

```scala
//syntax: 第二种使用package定义块区域，只有包含在区域内的源码才会打包在对应的路径下，这使得可以在同一文件中定义属于不同包的源码，但显然不是好的管理方式

package <identifier> {
    source code
}

//example

package com {
      package oreilly {           //逐层嵌套
      
                     class Config(val baseUrl: String = "http://localhost")
      }
}
```

* 访问打包类：访问包中的类有两种基本方式，完整路径引用及`import`指定的`package`.

```scala
//example 1: 完整路径引用

scala> val d = new java.util.Date
<console>: d: java.util.Date = Wed Jan 22 16:42:04 PDT 2014
```

```scala
//syntax: 导入指定包中类到当前命名空间，Scala中import的是一个语句，因为它没有返回值，可以放在任何可以使用语句的地方，如可以在使用处就近安置

import <package>.<class>

//example 2: 导入包中的一个类

scala> import java.util.Date
<console>: import java.util.Date

scala> val d = new Date //可省略路径

<console>: d: java.util.Date = Wed Jan 22 16:49:17 PDT 2014
```

```scala
//example 3: 使用下划线通配符导入整个包，在Java中使用的*，注意区分

scala> import collection.mutable._
<console>: import collection.mutable._
```

```scala
//syntax: 导入类集合，导入整个包可能带来命名冲突，所以可以选择只导入需要的类

import <package>.{<class 1>[, <class 2>...]}

//example 4

scala> import collection.mutable.{Queue,ArrayBuffer} //很明确导入了哪些类

<console>: import collection.mutable.{Queue, ArrayBuffer}

```

```scala
//syntax: 定义包别名，上一种方式还有一种潜在同名冲突问题，如前面介绍Scala中有immutable和mutable两个版本的collection，如同名Map类型，一旦导入mutable包，那么使用Map实例化便不会再例化immutable版本的对象，可以使用设置别名的方式加以规避，或使用完整路径名.

import <package>.{<original name>=><alias>}

//example 5

scala> import collection.mutable.{Map=>MutMap}
<console>: import collection.mutable.{Map=>MutMap}

scala> val m1 = Map(1 -> 2)
<console>: m1: scala.collection.immutable.Map[Int,Int] = Map(1 -> 2)

scala> val m2 = MutMap(2 -> 3)
<console>: m2: scala.collection.mutable.Map[Int,Int] = Map(2 -> 3)
```

---

#### 5. Privacy control
* 访问控制： Scala中定义的类默认具有"public"的访问权限，即任何代码都可以访问. 若要都一些类成员，甚至`package`的访问权限加以限制，可以对其增加访问控制. Scala中也有`protcted`和`private`两种限制访问机制，即类外均不可访问，前者子类可访问保护成员，后者子类也不可访问保护成员.

```scala
//example 1: protcted关键字

scala> class User { protected val passwd = util.Random.nextString(10) }
<console>: defined class User

//example 2: private关键字

scala> class User(private var password: String) {
     | def update(p: String) {
     |    println("Modifying the password!")
     |    password = p
     |  }
     | def validate(p: String) = p == password
     | }
<console>: defined class User
```

* 限制访问修饰符：前面是在类级别对成员的保护，增加限制访问修饰符可以在`package，class，instance`各级别控制访问. 简单的说，在受限访问字段前添加访问限定后，被授权的访问范围可以public的访问受保护字段，然而一旦出了访问受限修饰符划定的范围之后，受保护字段将不可访问.

```scala
//example 3: 访问控制修饰符

package com.oreilly {
     private[oreilly] class Config {   //指明Config类在oreilly包内是public的，包外是private的
     
         val url = "http://localhost"
     }
     
     class Authentication {
         private[this] val password = "jason"   //同类对象不可访问，其他可访问
         
         def validate = password.size > 0       
     }

     class Test {
          println(s"url = ${new Config().url}")
     }
}

scala> val valid = new com.oreilly.Authentication().validate
<console>: valid: Boolean = true

scala> new com.oreilly.Test      //Test类位于com.reilly包内，所以可以例化Config的实例

<console>:
url = http://localhost
res0: com.oreilly.Test = com.oreilly.Test@4c309d4d

scala> new com.oreilly.Config   //因为指明在com.reilly包内的类才能访问，所以这里不能直接例化实例

<console>:8: error: class Config in package oreilly cannot be
accessed in package com.oreilly
new com.oreilly.Config
```
---

#### 6. Final and Sealed classes
  `Final, sealed`主要用于控制子类继承，i）`final`关键字可添加在类、类属性、类方法前表明这些内容都不能被子类重写；ii）`sealed`相较于`final`宽松一些，其用于修饰类，如`sealed class A`，表明只有在`A`所在的源文件中定义的类才能成为`A`的子类，源文件范围外定义的类则不能成为其子类，前面介绍过的`Option`类型就是`abstract and sealed`.
  
---

### VII. Special Classes
  本章介绍几种特殊的Scala类，这些特殊类为常规类提供了应用扩展和管理的能力.

#### 1. Objects
  
* 定义：`Object`作为常规类扩展的一种类，具有Table 12罗列的特征.

```scala
//syntax: 由于object由编译器自动初始化，所以object定义不能包含类初始化参数.

object <identifier> [extends <identifier>]
                    [{ fields, methods, and classes }]

//example

scala> object Hello { println("in Hello"); def hi = "hi" }   
<console>: defined object Hello

scala> println(Hello.hi)   
in Hello        //第一次被访问会自动初始化

<console>: hi

scala> println(Hello.hi) //之后访问不会再初始化

<console>: hi
```

Table 12. Objects features

| Features | Description |
|----------|-------------|
| SingleTon | object类只能例化一个对象 |
| 无类参数 | 由于编译器自动例化，所以object不能指定初始化类参数. |
| 自动例化 | object对象创建不需要使用new关键字，直接访问类名即可，因为object类在第一次被访问时由JVM自动初始化，不需要显式初始化，所以，在未被访问之前，object对象不可访问. |
| 管理静态/全局类成员 | object类将Java/C++中的静态类成员或全局成员独立封装，与常规类中解耦出来，因为这些成员本身与类绑定，而不依赖于对象，所以这种管理方式更合理. |
| 单向继承 | object并非完成于class解耦，其作为管理类公共资源的类型，是对常规类的一部分及扩充，所以object可以继承class，但class不能继承object. |
| 最佳成员 | 最适合用于object的方法类型是纯函数及直接作用于input/output接口的函数，因为这类方法通常与类属性无关(仅限参考). |

* Apply方法与伴生object：之前介绍的`apply`方法同样可以应用于`oblect`类，而且这是在Scala和Chisel API中广泛应用的组合方式，这种组合应用可以直接通过调用`oblect_class()`来构建对象或其他应用，如在复合类型中介绍例化`List`对象时，就是应用了这种组合方式通过"对象工厂模式"创建实例的.

`Compinion object`实现了对常规类的扩展，实际上，`List`的对象工厂模式，也是通过`List`伴生的`object apply()`作为入口实现的. 常规类与其伴生`object`拥有相同的名称，而且必须定义在同一源文件中，而且从访问控制的角度，常规类与伴生`object`被看作是一个访问点，所以二者可以相互访问`private`和`protected`属性和方法成员.

```scala
//example 1

scala> class Multiplier(val x: Int) { def product(y: Int) = x * y }
<console>: defined class Multiplier

scala> object Multiplier { def apply(x: Int) = new Multiplier(x) } //对象工厂

<console>: defined object Multiplier

scala> val tripler = Multiplier(3) //通过调用伴生的object对象来创建常规类Multiplier的对象，而不是使用new，这个就是chisel wiki中推荐的省略new的方式

<console>: tripler: Multiplier = Multiplier@5af28b27

scala> val result = tripler.product(13)
<console>: result: Int = 39
```

```scala
//example 2: 访问控制

scala> :paste
// Entering paste mode (ctrl-D to finish)

object DBConnection {
      private val db_url = "jdbc://localhost"
      private val db_user = "franken"   
      private val db_pass = "berry"

      def apply() = new DBConnection
}                                                          

class DBConnection {
        private val props = Map(  
             "url" -> DBConnection.db_url, //通过类名直接访问伴生object的私有变量
             
             "user" -> DBConnection.db_user,
             "pass" -> DBConnection.db_pass
         )
         println(s"Created new connection for " + props("url"))
}

// Exiting paste mode, now interpreting.

defined object DBConnection
defined class DBConnection

scala> val conn = DBConnection()
Created new connection for jdbc://localhost
<console>: conn: DBConnection = DBConnection@4d27d9d
```

* main函数：之前的例子都是直接在`REPL`控制台加交互完成的，和学习`Python`一样，测试一些用法还行，但不可能写复杂程序. 实际的应用程序编写还是要使用`sbt, maven`等项目管理程序实现的，当然，对于简单的程序也可以自组织源文件的编译和执行，Scala中的编译执行和Java很像. 而入口点`main`函数是通过`object`实现的.

```scala
//example

$ cat > Date.scala object Date {
          def main(args: Array[String])  //定义main函数
          
          {
              println(new java.util.Date)
          }
        }

$ scalac Date.scala  //编译

$ scala Date  //运行，如果要为main提供参数，可以跟在Date类后添加

Mon Sep 01 22:03:09 PDT 2014
```

---

#### 2. Implicit class
  
* 定义：`Implicit class`是具有转型能力的一种类，即能够将一个A类型的实例转化为B类型实例，这样便可以将A类对象当做B类对象使用，对常规类进行了类型扩展.
  
  隐式类的实现需要满足一定条件，即当用户程序访问不属于A类对象的属性成员或方法成员时，编译器便会在当前的`namespace`下查找是否存在隐式类提供相应的转换，而隐式类的特征是：i）以对象为参数；ii）提供了A类对象不存在的被访问属性或方法. 一旦查找到匹配的隐式转换，编译器会自动完成调用，若未找到则报编译错误.
  
```scala
//syntax: 直接在类定义加implicit关键字

implicit class <identifier> ...

//example

scala> object IntUtils {
     |   implicit class Fishies(val x: Int) {   //将Int类型转换为Fishies类型
     
     |           def fishes = "Fish" * x
     |   }
     | }
<console>: defined object IntUtils

scala> import IntUtils._   //使用之前需要将其添加到当前的namespace中

<console>: import IntUtils._

scala> println(3.fishes) //Int类型参数转换为Fishies class以访问类内方法

<console>: FishFishFish
```

* 限制：上例中看到隐式类是在`object`内部定义的，这便是隐式类的约束之一，其约束具体包括：

  - 隐式类必须在`object，class，trait`中定义，其中在`object`内定义最容易`import`;
  - 隐式类的唯一对象参数必须是非隐式的，如上例中的`Int`；
  - 隐式类的名称不能与当前`namespace`中的其他`object，class，trait`命名冲突.

一般来说，在`object`类中定义隐式类是比较合理的，因为`object`本身是不可继承的类型，所以不必担心隐式类的"wrapper"存在隐式转换，可以做到安全管理. 但scala中的但·scala.Predef object·是一个例外，该`object`的部分成员是scala的库函数，所以会自动添加到当前`namespace`中，这些自动添加的成员中包含了一些支持表达式语法的隐式转换，即之前介绍创建二元`Tuple`时引入的`->`符号，其简化版定义如下，但是这种情况很少，只要避免做一些类似"运算符重载"的事情即可.

```scala

implicit class ArrowAssoc[A](x: A) {
      def ->[B](y: B) = Tuple2(x, y)
}
```
---

#### 3. Case classes
  `Case class`是一种具有`data-based`特征的类，定义如下，具体特征见Table 13.

```scala
//syntax: 注意参数化列表中不包含val参数类型，因为默认情况下，case class会将参数转化为val，所以没有必要单独声明val变量，但依然可以使用var声明变量.

case class <identifier> ([var] <identifier>: <type>[, ... ])  
                                                     
                        [extends <identifier>(<input parameters>)]
                        [{ fields and methods }]
```

Table 13. Case class features

| Features | Description |
|----------|-------------|
| 可例化 | case class可实例化对象 |
| 自动化 | case class会自动生成伴生的object，并在class和伴生object中自动生成一些方法成员，如Table 14. |
| 参数依赖 | 所有方法的自动化生成依赖于case class的类参数列表，编译器会根据类参数列表迭代扫描每一个类属性，格式化生成方法. |
| 适用性 | case class很适合作为数据存储的类，且不适合被继承，因为被子类中的父类属性不会用于创建自动化方法. |

Table 14. Generated case class methods

| Name | Location | Description |
|------|----------|-------------|
| apply | Object | 实例化工厂 |
| copy | Class | 返回一个实例副本，调用时可以设定参数使返回的副本在原实例上进行改造. |
| equals | Class | 判断如果两个对象属性域是否一致，也可以替代使用 ==. |
| hashCode | Class | 返回对象属性域的哈希值 |
| unapply | Object | 可以和apply方法一样不用方法名调用，作用是将实例中的属性抽取出来构成Tuple，便于将case class应用于match结构. |

注意：用户可以自定义这些方法成员，甚至自定义伴生`object`.

```scala
//example 1：自动化生成方法示例

scala> case class Character(name: String, isThief: Boolean)
<console>: defined class Character

scala> val h = Character("Hadrian", true)
<console>: h: Character = Character(Hadrian,true)

scala> val r = h.copy(name = "Royce")
<console>: r: Character = Character(Royce,true)

scala> h == r
<console>: res0: Boolean = false

scala> h match {
         | case Character(x, true) => s"$x is a thief" //应用unapply分解类属性，将第一个参数用于值绑定，第二个参数做匹配
         
         | case Character(x, false) => s"$x is not a thief"
         | }
<console>: res1: String = Hadrian is a thief
```

```scala
//example 2: data-based

scala> :paste

abstract class Expr
case class Var(name: String) extends Expr
case class Number(num: Double) extends Expr
case class UnOp(operator: String, arg: Expr) extends Expr
case class BinOp(operator: String, left: Expr, right: Expr) extends Expr
```

---

#### 4. Traits
  `Trait`在scala中的作用是支持类的多继承，即`class, case class, object, trait`至多只能继承一个常规类，但可以**继承多个`trait`类，同时，`trait`不可例化**. 熟悉Java的小伙伴应该了解，Java是不支持多继承的，但是可以实现多个`interface`，所以这里的`trait`估计应该是以`Java interface`为原型的. Table 15简单整理了`trait`的特征.

Table 15. Trait features

| Feature | Description |
|---------|-------------|
| 多继承 | —— |
| 不可例化 | —— |
| 无类参数 | 由于trait不可例化，所以与object类似，无类参数，但不同于object的是可以使用泛型. |

* 定义

```scala
//syntax

trait <identifier> [type-parameters]  //泛型

                   [extends <identifier>[type-parameters]]
                   [{ fields, methods, and classes }]
                   
//example

scala> trait HtmlUtils {
     |   def removeMarkup(input: String) = {
     |      input
     |      .replaceAll("""</?\w[^>]*>""","")； .replaceAll("<.*>","")
     |   }
     | }
<console>: defined trait HtmlUtils

scala> class Page(val s: String) extends HtmlUtils {
     |  def asPlainText = removeMarkup(s)  //直接访问父类方法
     
     | }
<console>: defined class Page

scala> new Page("<html><body><h1>Introduction</h1></body></html>").asPlainText
<console>: res2: String = Introduction
```

* 多继承：当类多继承时，使用`with`关键字，如下例，多继承有严格的顺序，即先`extends class/trait`，再`with trait...`.

Q：基于单继承机制JVM的scala是如何实现多继承的？

A：实际上，scala的编译器在进入JVM前，做了一次1 subclass vs m super classes到的1 subclass vs 1 super classes queue的队列匹配，即子类与每一个父类依次进行单继承来满足JVM，队列实现了scala的多继承，这一过程称为线性化，而类继承的书面顺序与编译器中的顺序稍有差异，即子类是按照书面顺序从右到左进行继承的，如

```scala
//example 1

scala> trait Base { override def toString = "Base" }
<console>: defined trait Base

scala> class A extends Base { override def toString = "A->" + super.toString }
<console>: defined class A

scala> trait B extends Base { override def toString = "B->" + super.toString }
<console>: defined trait B

scala> trait C extends Base { override def toString = "C->" + super.toString }
<console>: defined trait C

scala> class D extends A with B with C { override def toString = "D->" + super.toString } //书面多继承顺序D->A->B->C

<console>: defined class D

scala> new D()
<console>: res0: D = D->C->B->A->Base //编译器中依次单继承的顺序
```

之所以讨论编译顺序的问题，是因为有的设计中存在方法或属性重写的问题，只有搞清编译顺序才能妥善处理.

```scala
//example 2：重写顺序

scala> class RGBColor(val color: Int) { def hex = f"$color%06X" }
defined class RGBColor

scala> val green = new RGBColor(255 << 8).hex
green: String = 00FF00

scala> trait Opaque extends RGBColor { override def hex = s"${super.hex}FF" }
defined trait Opaque     

scala> class Paint(color: Int) extends RGBColor(color) with Opaque
defined class Paint

scala> val red = new Paint(128 << 16).hex
<console>: red: String = 800000FF

//因为编译顺序Paint->Opaque->RGBColor，继承顺序指明了对象的内存模型，如下，所以类参数从最里层的父类开始做初始化，而hex函数的调用从外层依次向里层查找，这就理解输出为什么为800000FF.


================Paint instance memory model==================
 __________________
|  ______________  |               
| |  __________  | |    
| | |          | | |
| | |          | | |
| | |_RGBColor_| | |
| |____Opaque____| |
|__________________|
  Paint instance
=============================================================

scala> val blue = new Overlay(192).hex
blue: String = 0000C033
```

* Self types
  `Self type`是`trait`的一个助记符，用于指定可以继承`trait`的类和子类，一旦声明`self type`那么只有指定类和子类才能继承`trait`，其他类不可以.
  
  `Self type`的意义在于，弥补了`trait`无类参数的缺陷，因为`trait`本身不带类参数，所以`trait`无法从带有类参数的`class`继承，但使用`selftype`可以将当前的`trait`声明被继承类的子类来继承带参数的`class`，而且可以直接引用`class`成员.
  
```scala
//syntax: <identifier>的标准用法是self，当然也可以使用其他的非scala关键字合法名称，但应尽量使用标准标识符.

trait ..... { <identifier>: <type> => .... }

//example 1

scala> class A { def hi = "hi" }
<console>: defined class A

scala> trait B { self: A =>  //声明self type

     | override def toString = "B: " + hi
     | }
defined trait B

scala> class C extends B    //C不是B中标识的A class或其子类，所以不能继承

<console>:9: error: illegal inheritance;self-type C does not conform to B's selftype B with A class C extends B ^

scala> class C extends A with B //C是A的子类，可以继承B

<console>: defined class C
```

```scala
//example 2: 弥补缺陷

scala> class TestSuite(suiteName: String) { def start() {} }
<console>: defined class TestSuite

scala> trait RandomSeeded { self: TestSuite =>
     |    def randomStart() {
     |       util.Random.setSeed(System.currentTimeMillis)
     |       self.start()   //通过self直接调用指定类成员
     
     |    }
     | }
<console>: defined trait RandomSeeded
```

* Instantiation with Traits：另外一种使用`trait`扩展类应用的方式是在类初始化的时候，类似于设计模式中的装饰者模式，原有类的设计不需要改动，以其为基础出现新的需求时，使用加"wrapper"的方式做功能加法，但一般都需要有公共超类. 但这里使用的例化时添加要灵活的多.

唯一的应用限制就是只能使用`with`关键字，不能使用`extends`，毕竟是在初始化阶段，且不是真的要通过继承实现功能扩展.

```scala
//example 1

scala> class A
defined class A

scala> trait B { self: A => }  //self type不是必须

defined trait B

scala> val a = new A with B  //例化对象时使用trait继承. 这里实际上是编译器自动生成了一个无名类，即anonymousClass extends A with B，之后实例化对象两步完成，但对用户是透明的.

a: A with B = $anon$1@26a7b76d
```

在例化时扩展类的方式，相当于提供了一种动态功能注入的能力，毕竟原型类已经定义，这就使得同一个类的两个对象可能拥有不同的行为，如下例，**scala确实强大**.

```scala
//example 2

scala> class User(val name: String) {
     |     def suffix = ""
     |     override def toString = s"$name$suffix"
     | }
<console>: defined class User

scala> trait Attorney { self: User => override def suffix = ", esq." }
<console>: defined trait Attorney

scala> trait Wizard { self: User => override def suffix = ", Wizard" }
<console>: defined trait Wizard

scala> val h = new User("Harry P") with Wizard
<console>: h: User with Wizard = Harry P, Wizard

scala> val g = new User("Ginny W") with Attorney
<console>: g: User with Attorney = Ginny W, esq.
```
---

### VIII. Review Function
  之前，我们介绍过函数的诸多种类用法，这里进一步阐述背后原理和其他类型扩展.
  
#### 1. Function value
  函数字面量实际上是`FunctionX[Y] class`的实例，其中，`X`取值范围`0~22`表示输入参数个数，类型参数`Y`定义输入和输出类型；换句话说，当定义一个函数字面量时，scala编译器实际上将该字面量转换为一个`class`的`apply`方法，该`class`继承自`FunctionX`，这种操作是为了兼容JVM，即所有函数都作为类的方法进行调用执行.
  
```scala
//example

scala> val hello1 = (n: String) => s"Hello, $n"
<console>: hello1: String => String = <function1>

scala> val h1 = hello1("Function Literals")
<console>: h1: String = Hello, Function Literals

scala> val hello2 = new Function1[String,String] {  //对比验证

     |      def apply(n: String) = s"Hello, $n"
     | }
<console>: hello2: String => String = <function1>

scala> val h2 = hello2("Function1 Instances")     //调用apply方法

h2: String = Hello, Function1 Instances

scala> println(s"hello1 = $hello1, hello2 = $hello2")
hello1 = <function1>, hello2 = <function1>        //一个东西
```
  
---

#### 2. Implicit parameters
  对于使用部分参数调用函数的方法，我们介绍过两种，一种是使用部分参数调用函数的方式，即将函数参数列表中的部分参数设为定值，另一部分则动态提供调用参数值；第二种是为参数列表中的部分参数提供默认值. 本节介绍一种新的调用方式，能够完美整合上述两种方式——隐式参数.
  
隐式参数函数调用的实现通过在被调函数参数列表中将部分参数设置为`implicit`，主调函数在函数体内定义一个局部变量，当函数调用未指定`implicit`标记的参数或变量时，便可使用主调函数的局部变量进行填充，由此，如何设置被调参数的主动权就完全在主调函数一边，这要比修改被调函数来配合实现要灵活的多.

```scala
//example

scala> object Doubly {
     |   def print(num: Double)(implicit fmt: String) = { //参数分组，将隐式与非隐式参数分隔
     
     |       println(fmt format num)
     |   }
     | }
<console>: defined object Doubly

scala> Doubly.print(3.724) //参数不全，无法调用

<console>:9: error: could not find implicit value for parameter fmt: String Doubly.print(3.724)

scala> Doubly.print(3.724)("%.1f")  //显式传递参数

<console>: 3.7

//隐式调用对比

scala> case class USD(amount: Double) {
     |    implicit val printFmt = "%.2f"   //主调函数定义局部value，用于填充隐式参数，注意implicit在主调被调函数都要指明
     
     |    def print = Doubly.print(amount)
     | }
<console>: defined class USD

scala> new USD(81.924).print
<console>: 81.92
```

---

### IX. Advanced Features of Data Type
  本章补充介绍`Type`的几个高级应用.
  
#### 1. Type aliases
`Type alias`实际上就是C语言中的`typedef`，即为以定义的类设置一个有意义的别名，唯一的限制是只能在`objects, classes, traits`内部定义`alias`，但不能定义`object`的别名，此外，如果已存在类有类参数，那么既可以在别名类中保留原有参数，也可以对参数进行修改.
```scala
//syntax: 使用type关键字定义

type <identifier>[type parameters] = <type name>[type parameters]

//example

scala> object TypeFun {
     | type Whole = Int
     | val x: Whole = 5
     |
     | type UserInfo = Tuple2[Int,String]
     | val u: UserInfo = new UserInfo(123, "George")
     |
     | type T3[A,B,C] = Tuple3[A,B,C]
     | val things = new T3(1, 'a', true)
     | }
<console>: defined object TypeFun

scala> val x = TypeFun.x
<console>: x: TypeFun.Whole = 5

scala> val u = TypeFun.u
<console>: u: TypeFun.UserInfo = (123,George)

scala> val things = TypeFun.things
<console>: things: (Int, Char, Boolean) = (1,a,true)
```

---

#### 2. Abstract types
  `Type alias`是解决单个类的问题，`abstract types`则是解决0或多个类的别名问题，其工作方式与alias类似，但`abstract types`是抽象的，不能够例化对象. 一般应用于泛型设计中，用于指定可接受的`type`范围，或应用于抽象类的类型声明.
  
```scala
//example 1：版本一

scala> trait Factory { type A; def create: A }  //type A声明A为抽象类

<console>: defined trait Factory

scala> trait UserFactory extends Factory {
     | type A = User
     | def create = new User("")
     | }
<console>: defined trait UserFactory
```

```scala
//example 2: 版本二

scala> trait Factory[A] { def create: A }  //这里使用泛型代替了版本一的形式，所以，可以说abstract type是泛型设计的另一种描述形式

defined trait Factory

scala> trait UserFactory extends Factory[User] { def create = new User("") }
defined trait UserFactory
```

---

#### 3. Bounded types
  `Chisel API`中用到不少`bounded types`，其表示仅能使用指定的类、基类或子类进行设计应用. 包括`upper bound`和`lower bound`，前者指定了能够作为`type parameter`的类型只能是指定类型和其子类（一般指定最高的父类），后者则指定能接受的最低阶的类，一般指向子类，尽管实际执行的类型比声明的要低.
  
```scala
//syntax: 操作符<:

<identifier> <: <upper bound type>

//example

scala> class BaseUser(val name: String)
<console>: defined class BaseUser

scala> class Admin(name: String, val level: String) extends BaseUser(name)
<console>: defined class Admin

scala> class Customer(name: String) extends BaseUser(name)
<console>: defined class Customer

scala> class PreferredCustomer(name: String) extends Customer(name)
<console>: defined class PreferredCustomer

scala> def check[A <: BaseUser](u: A) { if (u.name.isEmpty) println("Fail!") }  //指定可接受参数类型是BaseUser或其子类对象

<console>: check: [A <: BaseUser](u: A)Unit

scala> check(new Customer("Fred"))

scala> check(new Admin("", "strict"))
<console>: Fail!
```

相较于严格的`upper bound`还有一种相对宽松的`view bound`使用`<%`操作符标识，`view bound`支持隐式转换，即输入类型可以不是指定的基类或其子类，但是允许通过隐式转换转为可接受类型，而`upper bound`不支持隐式转换.

```scala
//syntax: 操作符>:

<identifier> >: <lower bound type>

//example

scala> def recruit[A >: Customer](u: Customer): A = u match {
     | case p: PreferredCustomer => new PreferredCustomer(u.name)
     | case c: Customer => new Customer(u.name)
     | }
<console>: recruit: [A >: Customer](u: Customer)A

scala> val customer = recruit(new Customer("Fred"))
<console>: customer: Customer = Customer@4746fb8c

scala> val preferred = recruit(new PreferredCustomer("George"))  //函数定义中指定最低阶类为Custom，但这里传入其子类，

<console>: preferred: Customer = PreferredCustomer@4cd8db31      //虽然会正确返回子类对象，但val只能使用限定的Customer定义

```

---

#### 4. Type variance
  上面介绍有关泛型的定义时，`type parameter`是一个定值，本节介绍的`type variance`允许类型变量用于泛型定义，其主要体现了"类型的转换"，是一种比上一节介绍的`bounded types`稍宽松的类型限定机制.
  
  `Type variance`主要解决的是下例所示的一类问题.

```scala
//example 1

scala> class Car { override def toString = "Car()" } 
defined class Car

scala> class Volvo extends Car { override def toString = "Volvo()" } 
defined class Volvo

scala> val c: Car = new Volvo() 
c: Car = Volvo()

scala> case class Item[A](a: A) { def get: A = a } 
defined class Item

scala> val c: Item[Car] = new Item[Volvo](new Volvo)   //即子类Volvo可以赋值给与父类Car的value以实现多态，但在类的泛型定义中，泛型声明与类定义是特定组合，即使type parameter是相互兼容，也不能匹配.

<console>:12: error: type mismatch; found : Item[Volvo] required: Item[Car]
Note: Volvo <: Car, but class Item is invariant in type A. You may wish to define A as +A instead. (SLS 4.5) val c: Item[Car] = new Item[Volvo](new Volvo)
```
  
要fix这个问题，便需要使用`type variance`，即将类定义中的泛型定义为`type covariant`，这个协变类型能够自动将非兼容类型转化为声明类型的基类，以实现泛型多态.

```scala
//example 2: 定义class，case class，trait等可继承的类时，在泛型定义的类型前加+

scala> case class Item[+A](a: A) { def get: A = a }  

<console>: defined class Item

scala> val c: Item[Car] = new Item[Volvo](new Volvo) //编译器自动寻求子类型Volvo向基类Car转化，一旦发现可转化，便可正确执行多态

<console>: c: Item[Car] = Item(Volvo())

scala> val auto = c.get  //虽然实际传入的对象是子类对象，但由于声明的类型是基类，所以返回值依然是基类

auto: Car = Volvo()
```

需要注意的是`covariant type`可以定义方法成员返回值类型，但不能定义方法输入参数类型，否则会报如下错误.

```scala
//example 3

scala> class Check[+A] { def check(a: A) = {} }
<console>:7: error: covariant type A occurs in contravariant position in type A of value a
class Check[+A] { def check(a: A) = {} }
```

错误信息说的很清楚，方法输入参数的类型需要`contravariant type`，逆变类型，就是从基类向子类转变的声明（这个逆变我没太搞明白，父类向子类转型?!），其定义形式如下.

```scala
//example 4: 使用-号

scala> class Check[-A] { def check(a: A) = {} } 
<console>: defined class Check
```

---

### X. Some Tips


| Item | Description |
|------|-------------|
| =>操作符应用| (1) match表达式 <br> (2) 函数型变量定义 <br> (3) 函数字面量 <br> (4) By-name parameter <br> (5) package import alias |
| 可嵌套元素 | (1) 表达式 <br> (2) 函数 <br> (3) 类 <br> (4) 第二种packaging |
| 无名函数/类 | (1) 函数字面量 <br> (2) 抽象类 <br> (3) instantiation with trait |
| 类的扩展途径 | (1) 继承 <br> (2) 聚合 <br> (3) 伴生object <br> (4) 多继承 <br> (5)self type trait <br> (6) instantiation with trait <br> (7) 使用隐式类 |
| 类内定义的元素 | (1) selftype <br> (2) implicit class <br> (3) type alias <br> (4) abstract types |
| 被忽视的符号 | (1) -> <br> (2) <- <br> (3) :: <br> (4) #:: <br> (5) >: <br> (6) <: <br> (7) <% |

---

**====说明====**
            
    (1) 因为学习Scala的目的还是为Chisel服务，并没有打算把Scala背后的原理和各种应用弄的很透，
        所以，两篇blog主要以"Learning Scala"为主来提取内容，同时参考"Programming in scala 3rd"
        避免核心内容缺失，之后也对比了Scala官网的cheatsheet，基本覆盖了全部基础内容，可能在
        collection那部分少了几种，一个是因为类型太多，另一个是个人觉得不常用.
    (2) 写这两篇blog主要是因为之前在Chisel讨论中有小伙伴提出关于Scala和函数式的一些问题，回答了
        一些，简单整理后结合教材写了这篇blog，后续可能遇到新问题后还会添加进来，使内容更加完善，
        帮助理解学习过程中遇到的问题.
    (3) 本来下一篇想把Chisel简单整理一下，但是目前素材不多，准备分析一个小的Chisel开源core后，
        再结合Chisel API介绍，可能更有实用价值.
