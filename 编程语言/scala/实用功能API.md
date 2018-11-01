#### 打印
```
print("My name is:")
println("My name is:")
val i = 7
println(i)
val i = 5;
val j = 8;
printf("My name is %s. I hava %d apples and %d eggs.\n","Ziyu",i,j)
```
#### 文件读取
```
scala> import scala.io.Source
import scala.io.Source //这行是Scala解释器执行上面语句后返回的结果
scala> val inputFile = Source.fromFile("output.txt")
inputFile: scala.io.BufferedSource = non-empty iterator  //这行是Scala解释器执行上面语句后返回的结果
scala> val lines = inputFile.getLines //返回的结果是一个迭代器
lines: Iterator[String] = non-empty iterator  //这行是Scala解释器执行上面语句后返回的结果
scala> for (line <- lines) println(line)
```
#### 字符串格式化
```
//字符串前面加s
val name : String = "hewe"
val tip : String = s"nihao,${name}"
//输出:   nihao,hewe
//字符串前加f
val tip : String = f"nihao,${name}%s"
//输出: nihao,hewe

//字符串前加raw,原生输出字符串,不做转化
val tip : String = raw"\n"
//输出: \n
```