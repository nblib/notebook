### `moduleName:func(param)`
当调用一个模块的方法时,需要把自己作为参数比如:
``` 
function getName(self)
    return self.name
end

person.getName(person)
```
每次这样写比较麻烦,所以使用`:`,省略了`self`参数:
``` 
function getName(self)
    return self.name
end

person:getName()
```
在函数中通过`self`调用自己.
### `funcName{name = "zhang"}`或`funcName[["nihao]]`
调用函数时,函数名后面没有小括号,而是直接跟字符串或表.  
当函数只有一个参数,而且参数为字符串或表的时候,可以省略圆括号,使用如上的格式调用.函数名后面必须有{或[分割.
```
function showName(name)
    print(name)
end
-- invoke
showName[["zhang"]]
```