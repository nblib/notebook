scala的List操作符的使用
===
#### ++
```
trait Pet {
  var name: String = "dddd"

  def out(): Unit
}

class Cat(catname: String) extends Pet {
  override def out() = println(catname)
}

class Dog(dogname: String) extends Pet {
  override def out() = println(dogname)
}

object Main {


  def main(args: Array[String]): Unit = {
    var li: List[Cat] = List[Cat](new Cat("1"), new Cat("2"))
    var li2: List[Dog] = List[Dog](new Dog("3"), new Dog("4"))
    var li3 = li ++ li2
    var li4 = li.++(li2)
    li3.foreach(p => p.out())
  }
}

```
返回一个新的list包含两个list中的所有元素.新的list中元素的类型为li和li2中元素的共有父类型(Pet)
#### ++:
```
val x = List(2)  //List
val y = mutable.LinkedList(1) //LinkedList
val z = x ++: y 
println(z)  //LinkedList(2, 1)
```
返回一个新的list包含两个list中的所有元素,和++不同的是,++:返回的新的list的类型(不是包含元素的类型)由第二个list决定,也就是后面那个list的类型决定,上面返回z的类型为LinkedList
#### +:和:+
```
var a = List(1)

println(a :+ 2) //打印: List(1, 2)
//println(a +: 2) 报错,2不是集合,不能向2中追加数据

val x = List(2)
val z = List(1)

println(x :+ z) //打印: List(2, List(1))
println(x +: z) //打印: List(List(2), 1)
```
返回list的副本并将另一个元素追加到list中.`:+`和`+:`的区别是,`:`靠近哪个元素则将那个元素当做list并将它的副本作为返回结果,如果那个元素不是list编译不通过.然后将另一个元素放到这个list的副本中.元素顺序不会改变.如上,`x:+z`或者`x+:z`,结果都是x相关在前,z相关在后

如上:
* `x:+z`,`:`靠近`x`,那么将`x`的副本作为返回的list,然后将`z`放到返回的list的中;
* `x+:z`,`:`靠近`z`,那么将`z`的副本作为返回的list,然后将`x`放到返回的list中;

#### /:
```
var a = List("i", "h", "a", "o")
var str = ("n" /: a) ((x, y) => x + y)
println(str) //打印: nihao
```
类似于`reduce`操作,只不过是第一次map的时候以"n"和list的第一个元素作为map的参数.

如上,执行如下:
* reduce("n","i"),执行`(x, y) => x + y`,返回"ni"
* reduce("ni","h"),执行`(x, y) => x + y`,返回"nih"
* reduce("nih","a"),执行`(x, y) => x + y`,返回"niha"
* reduce("niha","o"),执行`(x, y) => x + y`,返回"nihao"
* 最终返回字符串: "nihao"

#### ::
```
var a = List("i", "h", "a", "o")
println(a.::("n")) //打印: List(n, i, h, a, o)
println("n" :: a) //打印: List(n, i, h, a, o)
```
在list前添加一个元素
#### :::
```
var b = List(1, 2)
var c = List(3, 4)
println(b ::: c)  // 打印: List(1, 2, 3, 4)
```
将两个list合并

#### :\
```
var a = List("i", "h", "a", "o")
var str = (a :\ "n") ((x, y) => x + y)
println(str) //打印: ihaon
```
和`/:`类似,不过`/:`是添加到开头,而`:\`是添加到结尾
