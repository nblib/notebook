### `load`
> 用于加载一个数据块.从字符串或者函数中加载一个代码块为方法并返回.
``` 
name = "zhang"
local content = [[
    print("nihao")
    local parms = name
    print(#parms)
]]
assert(loadstring(content))()
```
* local 变量无法在生成的方法中访问
* load()返回的是函数,需要调用的话,还需要加括号.
* 5.1.x版本load方法为`load(func[,chunkname])`从函数中加载,`loadstring(str[,chunkname])`从字符串中加载
* 5.3.x版本为`load (chunk [, chunkname [, mode [, env]]])`
    * `chunk`为函数或字符串
    * mode 为加载模式:"t"文本样式,"b"二进制样式,"bt"二进制和文本模式.
    * env 代码块需要的参数
