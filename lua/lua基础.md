lua 基础
===

### 变量
#####三种类型：
1. 全局变量
1. 局部变量
1. 表中的域。

> **Lua 中的变量全是全局变量，那怕是语句块或是函数里，除非用 local 显式声明为局部变量。**
##### 默认值为 `nil`

##### 赋值
```lua
a = "hello" .. "hello"
a,b = 10, 2
a,b = b, a
```
##### 索引
```lua
t[i]
t.i                 -- 当索引为字符串类型时的一种简化写法
gettable_event(t,i) -- 采用索引访问本质上是一个类似这样的函数调用
```

### 循环
##### `while`循环
```
while(condition)
do
   statements
end
```
##### 数值`for`循环
```
for var=exp1,exp2,exp3 do  
    <执行体>  
end  
```
```
for i=1,f(x) do
    print(i)
end
 
for i=10,1,-1 do
    print(i)
end

```
> for的三个表达式在循环开始前一次性求值，以后不再进行求值。比如上面的f(x)只会在循环开始前执行一次，其结果用在后面的循环中。

##### 泛型`for`循环
``` 
--打印数组a的所有值  
for i,v in ipairs(a) 
    do print(v) 
end  
```
> 在lua中pairs与ipairs两个迭代器的用法相近，但有一点是不一样的：
  pairs可以遍历表中所有的key，并且除了迭代器本身以及遍历表本身还可以返回nil;
  但是ipairs则不能返回nil,只能返回数字0，如果遇到nil则退出。它只能遍历到表中出现的第一个不是整数的key
##### `repeat until`
```
--[ 变量定义 --]
a = 10
--[ 执行循环 --]
repeat
   print("a的值为:", a)
   a = a + 1
until( a > 15 )
```
### 流程控制

``` 
if( 布尔表达式 1)
then
   --[ 在布尔表达式 1 为 true 时执行该语句块 --]

elseif( 布尔表达式 2)
then
   --[ 在布尔表达式 2 为 true 时执行该语句块 --]

elseif( 布尔表达式 3)
then
   --[ 在布尔表达式 3 为 true 时执行该语句块 --]
else 
   --[ 如果以上布尔表达式都不为 true 则执行该语句块 --]
end
```
### 函数
##### 函数的定义
``` 
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
    function_body
    return result_params_comma_separated
end
```
* optional_function_scope:   
该参数是可选的制定函数是全局函数还是局部函数，未设置该参数默认为全局函数，如果你需要设置函数为局部函数需要使用关键字 local。
* function_name:   
指定函数名称。
* argument1, argument2, argument3..., argumentn:   
函数参数，多个参数以逗号隔开，函数也可以不带参数。
* function_body:   
函数体，函数中需要执行的代码语句块。  
* result_params_comma_separated:   
函数返回值，Lua语言函数可以返回多个值，每个值以逗号隔开。

##### 多返回值
``` 
> s, e = string.find("www.runoob.com", "runoob") 
> print(s, e)
```
##### 可变参数
``` 
function average(...)
   result = 0
   local arg={...}
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. #arg .. " 个数")
   return result/#arg
end

print("平均值为",average(10,5,3,4,5,6))
```
### 运算符
##### 普通运算符和`java`没啥区别
##### 特殊

操作符 | 描述 | 实例
---|---|---
and |逻辑与操作符。 |  若 A 为 false，则返回 A，否则返回 B。	(A and B) 为 false。
or |	逻辑或操作符。  | 若 A 为 true，则返回 A，否则返回 B。	(A or B) 为 true。
not |	逻辑非操作符。 | 与逻辑运算结果相反，如果条件为 true，逻辑非为 false。	not(A and B) 为 true。

.. | 连接字符串 | a= "aa".."bb"
\# | 一元运算符,返回字符串或表的长度 | #"hello" 返回5

### 字符串

Lua 语言中字符串可以使用以下三种方式来表示：
* 单引号间的一串字符。
* 双引号间的一串字符。
* [[和]]间的一串字符。
``` 
string1 = "Lua"
print("\"字符串 1 是\"",string1)
string2 = 'runoob.com'
print("字符串 2 是",string2)

string3 = [["Lua 教程"]]
print("字符串 3 是",string3)
```

###  数组
数组，就是相同数据类型的元素按一定顺序排列的集合，可以是一维数组和多维数组。
Lua 数组的索引键值可以使用整数表示，数组的大小不是固定的。

> **在 Lua 索引值是以 1 为起始，但你也可以指定 0 开始。**

##### 多维数组

``` 
-- 初始化数组
array = {}
for i=1,3 do
   array[i] = {}
      for j=1,3 do
         array[i][j] = i*j
      end
end

-- 访问数组
for i=1,3 do
   for j=1,3 do
      print(array[i][j])
   end
end
```
### 迭代器
迭代器（iterator）是一种对象，它能够用来遍历标准模板库容器中的部分或全部元素，每个迭代器对象代表容器中的确定的地址
在Lua中迭代器是一种支持指针类型的结构，它可以遍历集合的每一个元素。
##### 泛型 `for` 迭代器
``` 
for k, v in pairs(t) do
    print(k, v)
end
```
范性for的执行过程：
1. 首先，初始化，计算in后面表达式的值，表达式应该返回范性for需要的三个值：迭代函数、状态常量、控制变量；与多值赋值一样，如果表达式返回的结果个数不足三个会自动用nil补足，多出部分会被忽略。
1. 第二，将状态常量和控制变量作为参数调用迭代函数（注意：对于for结构来说，状态常量没有用处，仅仅在初始化时获取他的值并传递给迭代函数）。
1. 第三，将迭代函数返回的值赋给变量列表。
1. 第四，如果返回的第一个值为nil循环结束，否则执行循环体。
1. 第五，回到第二步再次调用迭代函数

Lua 的迭代器包含以下两种类型：
* 无状态的迭代器
* 多状态的迭代器
##### 无状态的迭代器
无状态的迭代器是指不保留任何状态的迭代器，因此在循环中我们可以利用无状态迭代器避免创建闭包花费额外的代价。
> 无状态迭代器的典型的简单的例子是ipairs
``` 
function square(iteratorMaxCount,currentNumber)
   if currentNumber<iteratorMaxCount
   then
      currentNumber = currentNumber+1
   return currentNumber, currentNumber*currentNumber
   end
end

for i,n in square,3,0
do
   print(i,n)
end
```
##### 多状态的迭代器
很多情况下，迭代器需要保存多个状态信息而不是简单的状态常量和控制变量，最简单的方法是使用闭包，还有一种方法就是将所有的状态信息封装到table内，将table作为迭代器的状态常量，因为这种情况下可以将所有的信息存放在table内，所以迭代函数通常不需要第二个参数。
``` 
array = {"Lua", "Tutorial"}

function elementIterator (collection)
   local index = 0
   local count = #collection
   -- 闭包函数
   return function ()
      index = index + 1
      if index <= count
      then
         --  返回迭代器的当前元素
         return collection[index]
      end
   end
end

for element in elementIterator(array)
do
   print(element)
end
```
### talbe(表)
* table 是 Lua 的一种数据结构用来帮助我们创建不同的数据类型，如：数字、字典等。
* Lua table 使用关联型数组，你可以用任意类型的值来作数组的索引，但这个值不能是 nil。
* Lua table 是不固定大小的，你可以根据自己需要进行扩容。
* Lua也是通过table来解决模块（module）、包（package）和对象（Object）的。 例如string.format表示使用"format"来索引table string。
``` 
-- 初始化表
mytable = {}

-- 指定值
mytable[1]= "Lua"

-- 移除引用
mytable = nil
-- lua 垃圾回收会释放内存
```

### 模块和包
Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。

``` 
-- 文件名为 module.lua
-- 定义一个名为 module 的模块
module = {}
 
-- 定义一个常量
module.constant = "这是一个常量"
 
-- 定义一个函数
function module.func1()
    io.write("这是一个公有函数！\n")
end
 
local function func2()
    print("这是一个私有函数！")
end
 
function module.func3()
    func2()
end
 
return module
```
##### 加载模块--`require` 函数
```require("<模块名>") 或 require "<模块名>"```
``` 
-- test_module.lua 文件
-- module 模块为上文提到到 module.lua
require("module")
 
print(module.constant)
 
module.func3()

//或者给加载的模块定义一个别名变量
-- test_module2.lua 文件
-- module 模块为上文提到到 module.lua
-- 别名变量 m
local m = require("module")
 
print(m.constant)
 
m.func3()
```
##### 加载机制
require 用于搜索 Lua 文件的路径是存放在全局变量 package.path 中，当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始这个环境变量。

如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。

当然，如果没有 LUA_PATH 这个环境变量，也可以自定义设置，在当前用户根目录下打开 .profile 文件（没有则创建，打开 .bashrc 文件也可以），例如把 "~/lua/" 路径加入 LUA_PATH 环境变量里
##### C 包
Lua和C是很容易结合的，使用C为Lua写包。
与Lua中写包不同，C包在使用以前必须首先加载并连接，在大多数系统中最容易的实现方式是通过动态连接库机制。
Lua在一个叫loadlib的函数内提供了所有的动态连接的功能。这个函数有两个参数:库的绝对路径和初始化函数。
``` 
local path = "/usr/local/lua/lib/libluasocket.so"
local f = loadlib(path, "luaopen_socket")
```
> 如果加载动态库或者查找初始化函数时出错，loadlib将返回nil和错误信息。