#概述
 Lua 提供了元表(Metatable)，允许我们改变table的行为，每个行为关联了对应的元方法。
#本质
元表本质为给table添加了一个键值对,键为`__metatable`,值为一个表,当对原始表操作的时候,就会去这个表中找键值对,比如`__index`,`__call`等键,然后找到对应的值,值可以是一个值或方法,结构大致如下:
```table
{
    "name": "hewe",
    "age": 12,
    "__metatable":{
        "__index": "23",
        "__call":function(){
        
        },
        ...
    },
    ...
}
```
#定义
通过两个函数进行操作元表:
* `setmetatable(table, metatable)`  
对指定table设置元表(metatable)，如果元表(metatable)中存在__metatable键值，setmetatable会失败。
* `getmetatable(table)`  
返回对象的元表(metatable)。

``` 
mytable = {}                          -- 普通表 
mymetatable = {}                      -- 元表
setmetatable(mytable,mymetatable)     -- 把 mymetatable 设为 mytable 的元表 
getmetatable(mytable)                 -- 这回返回mymetatable
```
#`metatable`常用键
### `__index` 元方法
* 当你通过键来访问 table 的时候，如果这个键没有值，那么Lua就会寻找该table的metatable（假定有metatable）中的__index 键。
* 如果__index包含一个表格，Lua会在表格中查找相应的键。
* 如果__index包含一个函数的话，Lua就会调用那个函数，table和键会作为参数传递给函数。
``` 
$ lua
Lua 5.3.0  Copyright (C) 1994-2015 Lua.org, PUC-Rio
> other = { foo = 3 } 
> t = setmetatable({}, { __index = other }) 
> t.foo
3
> t.bar
nil
```
###`__newindex` 元方法
当你给表的一个缺少的索引赋值，解释器就会查找__newindex 元方法：如果存在则调用这个函数而不进行赋值操作。
### `__call` 元方法
`__call` 元方法在 Lua 调用一个值时调用。以下实例演示了计算表中元素的和：
``` 
mytable = setmetatable({10},{
    __call = function(mytable,arg1,arg2,arg3)
        mytable.sum = mytable[1] + arg1 + arg2 + arg3
        sum = mytable.sum
        return sum
    end
})
print(mytable(1,2,3))
```
>`call`方法的参数和返回值都可以自定义,只要在调用的时候`mytable()`相应处理即可.
###### 查找表元素规则
Lua查找一个表元素时的规则，其实就是如下3个步骤:
1. 在表中查找，如果找到，返回该元素，找不到则继续
2. 判断该表是否有元表，如果没有元表，返回nil，有元表则继续。
3. 判断元表有没有__index方法，如果__index方法为nil，则返回nil；如果__index方法是一个表，则重复1、2、3；如果__index方法是一个函数，则返回该函数的返回值。
