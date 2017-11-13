# 概述
这里请求转发,指lua_nginx中,改变请求的url的方法.lua-nginx模块中,提供改变url的方法有:
* ngx.exec
* ngx.req.set_uri
* ngx.redirect

### ngx.exec
对URI参数内部重定向和类似于`echo-nginx-module`模块的`echo_exec`指令。
```
 ngx.exec('/some-location');
 ngx.exec('/some-location', 'a=3&b=5&c=6');
 ngx.exec('/some-location?a=3&b=5', 'c=6');
```
另外，Lua表可以通过args参数ngx_lua进行URI连接。
``` 
ngx.exec("/foo", { a = 3, b = "hello world" })
```
也可以用在命名的`location`上,但是参数会被忽略,在下面的例子中,`GET /foo/file.php?a=hello`将返回 "hello" 而不是 "goodbye":
``` 
 location /foo {
     content_by_lua_block {
         ngx.exec("@bar", "a=goodbye");
     }
 }

 location @bar {
     content_by_lua_block {
         local args = ngx.req.get_uri_args()
         for key, val in pairs(args) do
             if key == "a" then
                 ngx.say(val)
             end
         end
     }
 }
```
这个命令终止本次请求,必须在调用`ngx.send_header`之前.

推荐紧随`return`语句使用:
``` 
return ngx.exec(...);
```
### ngx.redirect
返回302等重定向状态码
```
return ngx.redirect()
```
### ngx.req.set_uri
重写当前请求的uri,第二个参数为`是否跳转`,并且第二个参数只能在`rewrite_by_lua_*`阶段中设置为true,在其他阶段设置为true将会
报错.
比如:
```
rewrite ^/foo last;
```
等价于
```
ngx.req.set_uri("/foo",true)
```
比如
``` 
rewrite ^ /foo break;
```
等价于
``` 
ngx.req.set_uri("/foo", false)
```
设置参数
``` 
 ngx.req.set_uri_args({a = 3})
 ngx.req.set_uri("/foo", true)
```

### 还有ngx.location.capture
发送一个同步非阻塞的子请求,类似于"HttpClient".