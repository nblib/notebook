#语法
* {{expression}}  
转化`expression`为`expression`中的值,如果`expression`是一个方法,便会执行这个方法,最后结果为这个方法的返回值.

如果最后结果中有html符号,会被转义:
``` 
["&"] = "&amp;",
["<"] = "&lt;",
[">"] = "&gt;",
['"'] = "&quot;",
["'"] = "&#39;",
["/"] = "&#47;"
```  
* {*expression*}  
转化`expression`为`expression`中的值,如果`expression`是一个方法,便会执行这个方法,最后结果为这个方法的返回值.

不会转义html的符号
* {%lua code %}  
执行 lua代码
* {(template)}  
内部嵌入别的模板文件.`teplate`为**模板的文件名的字符串**,如果没有找到文件,那么返回的结果为文件名.比如:
``` 
<body>
{(test.html)}
</body>
```
如果没有找到:
``` 
<body>
test.html
</body>
```
* {[expression]}  
内部嵌入别的模板文件.`teplate`为**寻找模板所在位置的参数**
> {[expression]}和{(template)}的区别  
`expression`为lua的代码,比如:`getFile("te.html")`或者一个字符串`"tp.html"`,因为它会被直接解析为lua语句执行.  
`template`为模板文件的文件名的字符串,因为它会被直接解析为字符串.
* {-blockTag-}...{-blockTag-}   
用于定义一个代码块,可以在多个地方通过`{%block["blockTag"]%}`进行引用这个模块.
* {-verbatim-}...{-verbatim-} and {-raw-}...{-raw-}  
在这之间的代码,不会被模板渲染,也就是原样输出,不解析
* {# comments #}  
用于注释,不会在返回结果中看到.
#内置变量
* `___`  
存放解析后的模板内容
* `context`  
存放此次渲染需要的内容的数组,比如变量等都存在于这个数组中
* `include`  
嵌入变量的帮助函数
* `layout`  
模板视图
* `blocks`  
代码块存放数组
* `template`  
模板数组
# 工作原理
将一个模板文件解析为`lua`代码,然后依次执行代码.
代码将一个模板文件分割放入一个数组`__`中,非语法内容单独作为一个数组元素,语法内容分别单独作为一个数组元素.

比如,如下的一个模板文件:
```
<html>
John say:  {{message}}
<h1>{{info}}</h1>
</html>
```
模板渲染时,首先寻找第一个`{`,找到后,把它之前的静态内容放到数组中:
``` 
__[1] = [=[
<html>
John say:   
]=]
```
* `__`是一个数组
* `[=[]=]`表示一个原生字符串,这样写,可以省略转义字符`\n`等,换行会自动保留,也就是输入的什么样子,保存的就是什么样子.

然后把`{{}}`之间的内容放到下一个位置:
```
__[2] = message
```
依次往下,最后数组是这个样子:
``` 
___[1]=[=[
<html>
John say:  ]=]
___[2]=message
___[3]=[=[

<h1>]=]
___[4]=info
___[5]=[=[
</h1>
</html>
]=]
```
如上,也是模板生成的字符串中的一部分.也就是说解析模板文件生成一个字符串.然后,关键的一步,模板调用了`load()`方法,
这个方法重要的作用是,将一个字符串解析为一个`lua`代码片段去执行.也就是说以上的字符串成为了一段代码,顺序执行:
``` 
--开始执行代码
___[1]=[=[
<html>
John say:  ]=]
--由于这是个代码,所以,message会自动寻找为"message"的变量
___[2]=message
___[3]=[=[

<h1>]=]
___[4]=info
___[5]=[=[
</h1>
</html>
]=]
```
执行完成后,`__`数组的内容也就成为如下,假设传入的message和info值为"nihao"和"world":
``` 
___[1]=[=[
<html>
John say:  ]=]
___[2]="nihao"
___[3]=[=[

<h1>]=]
___[4]="world"
___[5]=[=[
</h1>
</html>
]=]
```
最后,调用`table.concat()`方法将数组连接起来,也就是最终的输出结果:
```
<html>
John say:  nihao
<h1>{world</h1>
</html>
```
# 综合表达式的实例
原始模板文件内容:
``` 
<html>
<h1>{{title}}</h1>
<h1>{*description*}</h1>
<ul>
    {% for _, keyword in ipairs(keywords) do %}
    <li>{{keyword}}</li>
    {% end %}
</ul>
{(part.html)}
{[getFile(part.html)]}
{-content-}
<ul>
    {% for _, addr in ipairs(addrs) do %}
    <li>{{addr}}</li>
    {% end %}
</ul>
{-content-}
{-raw-}
local name = "nihao"
{-raw-}
{# this is comment #}
</html>
```
模板解析后(只是解析为lua代码,真正的结果没有列出)的内容:
``` 
--上下文,所有关于这个模板的信息都存放在这里,比如有一个变量:"name",在模板中可以通过"{{name}}或者{{context.name}}调用
context=... or {}
-- 定义这个函数,用于模板嵌套,循环解析模板
local function include(v, c) return template.compile(v)(c or context) end
-- 初始化数组,布局,块等变量
local ___,blocks,layout={},blocks or {}
--以下是解析模板的内容,
--没有表达式的内容
___[#___+1]=[=[
<html>
<h1>]=]
--表达式:{{title}},这个表达式会调用"escape"方法,对html的一些符号转义,然后调用"output"方法,就是相当于"toString()"方法,
___[#___+1]=template.escape(title)
--没有表达式的内容
___[#___+1]=[=[
</h1>
<h1>]=]
--表达式:{{description}},这个表达式会调用"output"方法,就是相当于"toString()"方法,
___[#___+1]=template.output(description)
--没有表达式的内容
___[#___+1]=[=[
</h1>
<ul>
]=]
-- 表达式:{%%},也就是将里面的内容直接转化成lua代码
for _, keyword in ipairs(keywords) do
___[#___+1]=[=[
    <li>]=]
___[#___+1]=template.escape(keyword)
___[#___+1]=[=[
</li>
]=]
end
--没有表达式的内容
___[#___+1]=[=[
</ul>
]=]
--表达式:{(part.html)},引入嵌套的模板文件,也就是调用include,"part.html"作为字符串为模板文件的名称作为参数,解析嵌套的模板文件,会寻找"part.html"模板文件
___[#___+1]=include([=[part.html]=])
--没有表达式的内容
___[#___+1]=[=[

]=]
--表达式:{[getFile(part.html)]},引入嵌套的模板文件,也就是调用include,解析嵌套的模板文件,参数不同上面的{()},并非字符串,而是作为了一段lua代码作为参数,可以理解为方法的返回值作为了参数
___[#___+1]=include(getFile(part.html))
--没有表达式的内容
___[#___+1]=[=[

]=]
--表达式: {-content-},"content"可以自己随便取个名字,但是不能和"raw","verbatim"重名.这个模块的作用是将一段模板内容作为一个模块,可以在别的地方调用.
本质是将内容放到了"block"数组中,使用的时候根据数组名获取即可.
blocks["content"]=include[=[
<ul>
    {% for _, addr in ipairs(addrs) do %}
    <li>{{addr}}</li>
    {% end %}
</ul>]=]
-- 表达式:{-raw-},表示为原生的html内容,不会被模板解析,可以用来存放前端的代码等.
___[#___+1]=[=[
local name = "nihao"
]=]
--没有表达式的内容

___[#___+1]=[=[
</html>
]=]
--最后将数组连接起来,作为返回值.也就是生成的页面的内容
return layout and include(layout,setmetatable({view=table.concat(___),blocks=blocks},{__index=context})) or table.concat(___)
```
