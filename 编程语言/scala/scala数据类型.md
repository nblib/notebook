scala数据类型
===
* `Any`是所有类型的父类,直接继承它的子类有:`AnyVal`和`AnyRef`
* `AnyVal`是所有值类型的父类
* `AnyRef`是所有引用类型的父类,等同于java的`Object`
* 直接继承`AnyVal`的子类有:`Double,Float,Long,Int,Short,Byte,Unit,Boolean,Char`
* 直接继承`AnyRef`的子类有:`List,Option,用户的类`
* `Null`所有`AnyRef`的子类的子类
* `Nothing`是所有类型的子类

## 类型转换
以下类型之间可以直接转化:
```
Byte->Short->Int->Long->Float->Double
Char->Int
```
* 只能按箭头方向直接转,不能逆向直接转

## Nothing 和 Null
Nothing本身是所有类型的子类,没有意义,没有值,语义上讲,scala中每个方法都应该有返回值,但是,以下情况:
```
def get(index:Int):Int = {
    if(x < 0) throw new Exception(...)
    else ....
}
```
抛异常的时候没有返回值,但是为了解释的通`每个方法都应该有返回值`这句话,引入了`Nothing`,也就是说,抛异常虽然没有返回值,但是我们可以说返回了`Nothing`

至于`Null`是一个类,它只有唯一的一个实例,就是`null`,Null的实例`null`的存在是为了和jvm中的其他语言,比如java中的null交互.`scala`中不建议使用`null`

#### 变量
```
var name : String = "nihao"
```
* 可以不声明类型,会自动推断
#### 常量 
```
val myStr2 : String = "Hello World!"
```
* 不可改变
#### 基本数据类型
Byte、Char、Short、Int、Long、Float、Double和Boolean
* 都是类
* Int的全名是scala.Int。对于字符串，Scala用java.lang.String类来表示字符串。
* Scala允许对“字面量”直接执行方法
#### 操作符
```
scala> val sum1 = 5 + 3 //实际上调用了 (5).+(3)
sum1: Int = 8
scala> val sum2 = (5).+(3) //可以发现，写成方法调用的形式，和上面得到相同的结果
sum2: Int = 8
```
* Scala中并没有提供++和–操作符
* 前者是后者的简写形式，这里的+是方法名，是Int类中的一个方法