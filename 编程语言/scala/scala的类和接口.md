scala的类
===
## 简单的类
```
class User

val user1 = new User

```
## 类的定义
```
class Point(var x: Int, var y: Int) {

  def move(dx: Int, dy: Int): Unit = {
    x = x + dx
    y = y + dy
  }

  override def toString: String =
    s"($x, $y)"
}

val point1 = new Point(2, 3)
point1.x  // 2
println(point1)  // prints (2, 3)

```
* 构造方法直接在类名后

## 构造方法
```
class Point(var x: Int = 0, var y: Int = 0)

val origin = new Point  // x and y are both set to 0
val point1 = new Point(1)
val point2 = new Point(y=2)
println(point1.x)  // prints 1
```
* 构造方法可以包括默认值
* 可以通过在构造方法中使用变量名直接赋值

## 私有成员和get/set方法
```
class Point {
  private var _x = 0
  private var _y = 0
  private val bound = 100

  def x = _x
  def x_= (newValue: Int): Unit = {
    if (newValue < bound) _x = newValue else printWarning
  }

  def y = _y
  def y_= (newValue: Int): Unit = {
    if (newValue < bound) _y = newValue else printWarning
  }

  private def printWarning = println("WARNING: Out of bounds")
}

val point1 = new Point
point1.x = 99
point1.y = 101 // prints the warning
```
* get方法使用`x`
* set方法使用`x_`

## 接口
#### 定义接口
```
trait HairColor
```
最简单的接口
#### 接口中的方法和