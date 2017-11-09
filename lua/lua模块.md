#lua模块
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
* 导入一个模块相当于导入一个表结构的变量,通过这个表结构调用这个模块的内容.
* 整个lua虚拟机运行阶段,一个模块会被导入一次,也就是说多次导入的使用的都是一个模块的变量比如:
    * demo1导入demo2,然后给demo2添加了一个name属性,那么当demo3导入demo2的时候,还是有name属性
* 模块导出的