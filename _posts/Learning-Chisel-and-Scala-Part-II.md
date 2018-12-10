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
| & / intersect | Set(1,2,3) & Set(1)/Set(1,2,3) intersect Set(1), res: Set(1) | 求交集 |
| \| / union | Set(1,2,3) | Set(4)/Set(1,2,3) union Set(4), res: Set(1,2,3,4) | 求并集 |
| &~ / diff | Set(1,2,3) &~ Set(3)/Set(1,2,3) diff Set(3), res: Set(1,2) | 去交集，求补集 |

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

1）定义：假设我们要判断一个元素是否存在与一个集合中，那么只有两种结果：存在，不存在，这就是`Option`类似所描述的，即判断一个元素是否存在，诸如此类的二元判断. 但`Option`是一个抽象类，是不可例化的，所以真正做判断的是其两个子类：`Some`和`None`，前者是泛型类型，包含一个值，意为"存在"，后者为空.

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

2）应用：Option常取代"null"，用作安全运算检查，即先检查运算是否合法，再决定是否使用运算结果.

```scala
example 1

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

3）访问元素：在2）中判断运算合法性后，还是要取出结果，可以使用Table 10中安全方式.

```scala
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

Table 10.Safe Option extractions

| Name | Example | Description |
|------|---------|-------------|
| fold | divide(3,0).fold(-1)(x => x), res: -1 | 根据返回值判断是否为None，若为None则返回指定值-1，否则进行函数字面量参数指定的运算，本例直接返回x，Table 5中的方法都可类似应用 |
| getOrElse | divide(3,0) getOrElse -1, res: -1 | 类似的检查函数返回值是否为None，为None则输出指定的默认值-1，否则返回实际结果 |
| match expression | divide(3,0) match { case Some(x) => x; case None => -1}, res: -1 | —— |

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
| 被忽视的符号 | (1) #:: |

---

**====说明====**
