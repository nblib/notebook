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