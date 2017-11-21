# 正则选项
``` 
a             锚定模式，只从头开始匹配. 
d             DFA模式，或者称最长字符串匹配语义，需要PCRE 6.0+支持.

D             允许重复的命名的子模式，该选项需要PCRE 8.12+支持，例如
                local m = ngx.re.match("hello, world",
                                       "(?<named>\w+), (?<named>\w+)",
                                       "D")
                -- m["named"] == {"hello", "world"}

i             大小写不敏感模式.

j             启用PCRE JIT编译, 需要PCRE 8.21+ 支持，并且必须在编译时加上选项--enable-jit，
                为了达到最佳性能，该选项总是应该和'o'选项搭配使用.		  

J             启用PCRE Javascript的兼容模式，需要PCRE 8.12+ 支持. 

m             多行模式.

o             一次编译模式，启用worker-process级别的编译正则表达式的缓存.

s             单行模式.

u             UTF-8模式. 该选项需要在编译PCRE库时加上--enable-utf8 选项.

U             与"u" 选项类似，但是该项选禁止PCRE对subject字符串UTF-8有效性的检查.

x             扩展模式
```
# 命令
### ngx.re.match
```
syntax: captures, err = ngx.re.match(subject, regex, options?, ctx?, res_table?)
```
```
context: init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, balancer_by_lua*, ssl_certificate_by_lua*, ssl_session_fetch_by_lua*, ssl_session_store_by_lua*
```
返回匹配到的第一个结果
``` 
 local m, err = ngx.re.match("hello, 1234", "[0-9]+")
 if m then
     -- m[0] == "1234"
 else
     if err then
         ngx.log(ngx.ERR, "error: ", err)
         return
     end
     ngx.say("match not found")
 end
```
多组匹配:
``` 
 local m, err = ngx.re.match("hello, world", "(world)|(hello)|(?<named>howdy)")
 -- m[0] == "hello"
 -- m[1] == false
 -- m[2] == "hello"
 -- m[3] == false
 -- m["named"] == false
```
### ngx.re.find
```
syntax: from, to, err = ngx.re.find(subject, regex, options?, ctx?, nth?)
```
```
context: init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, balancer_by_lua*, ssl_certificate_by_lua*, ssl_session_fetch_by_lua*, ssl_session_store_by_lua*
```
返回匹配到的结果的开始到结束的索引,索引从1开始计数.第五个参数默认为0,如果为2,则返回匹配到的第二个组的索引.
``` 
 local str = "hello, 1234"
 local from, to = ngx.re.find(str, "([0-9])([0-9]+)", "jo", nil, 2)
 if from then
     ngx.say("matched 2nd submatch: ", string.sub(str, from, to))  -- yields "234"
 end
```
### ngx.re.gmatch
``` 
syntax: iterator, err = ngx.re.gmatch(subject, regex, options?)
```
``` 
context: init_worker_by_lua*, set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, balancer_by_lua*, ssl_certificate_by_lua*, ssl_session_fetch_by_lua*, ssl_session_store_by_lua*
```
和 match相同,不过返回的结果是所有的结果,返回一个迭代器,用于遍历'
``` 
local iterator, err = ngx.re.gmatch("hello, world!", "([a-z]+)", "i")
 if not iterator then
     ngx.log(ngx.ERR, "error: ", err)
     return
 end

 local m
 m, err = iterator()    -- m[0] == m[1] == "hello"
 if err then
     ngx.log(ngx.ERR, "error: ", err)
     return
 end

 m, err = iterator()    -- m[0] == m[1] == "world"
 if err then
     ngx.log(ngx.ERR, "error: ", err)
     return
 end

 m, err = iterator()    -- m == nil
 if err then
     ngx.log(ngx.ERR, "error: ", err)
     return
 end
```