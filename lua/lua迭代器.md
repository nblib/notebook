### 自定义迭代器
```
#!/usr/bin/lua

tab = {
    i = 0,
    name = "Nihao",
    age = 23
}
function getName(self)
    self.i = self.i + 1
    if self.i > 2 then return nil end
    return "name", self.name
end
local meta = {__call = getName}
function iter()
    return setmetatable(tab,meta)
end
for i,v in iter() do
    print(i,v)
end
```
1. for循环初始化,初始化,调用iter()方法,获取,真正的迭代方法,和状态变量(可以没有)等.
2. iter()方法,返回了一个加工了元表的表,这个表有一个元方法,`__call`,当把表作为方法调用时会调用这个方法比如: table_name()
3. for获取到真正的迭代方法后,开始调用获取第一个值,由上可知,它将调用getName方法.返回了两个值分别赋值给i,v
3. 一直循环调用getName方法,直到返回的值为`nil`.