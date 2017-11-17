# 概述
`ngx.exit(status)`命令用于对当前请求的中断,中断指中断请求或者中断请求的某个阶段.


# 用法
``` 
ngx.exit(status)
```
比如:
``` 
ngx.exit(200)
ngx.exit(404)
ngx.exit(ngx.HTTP_OK)
```
# 执行上下文
* rewrite_by_lua, 
* access_by_lua, 
* content_by_lua, 
* header_filter_by_lua, 
* ngx.timer., 
* balancer_by_lua, 
* ssl_certificate_by_lua*
# 作用
* 当传入的status >= 200（200即为ngx.HTTP_OK），ngx.exit() 会中断当前请求，并将传入的状态码（status）返回给nginx。
* 当传入的status == 0（0即为ngx.OK）则 ngx.exit() 会中断当前执行的phrase（ngx-lua模块处理请求的阶段，如content_by_lua*），进而继续执行下面的phrase。
* 对于 ngx.exit() 需要进一步注意的是参数status的使用，status可以传入ngx-lua所定义的所有的HTTP状态码常量（如：ngx.HTTP_OK、ngx.HTTP_GONE、ngx.HTTP_INTERNAL_SERVER_ERROR等）和两个ngx-lua模块内核常量（只支持NGX_OK和NGX_ERROR这两个，如果传入其他的如ngx.AGAIN等则进程hang住）。
* 文档中推荐的 ngx.exit() 最佳实践是同 return 语句组合使用，目的在于增强请求被终止的语义（return ngx.exit(...)）。


当状态码>=200时,会退出**当前请求阶段**,返回给nginx,然后nginx会接着执行filter阶段和log阶段;当状态码为0时,会退出**当前请求阶段的当前阶段**.继续执行下一个阶段.
> 区别  
 当前请求阶段指的是从请求到来,到内容输出的这个阶段  
 当前请求阶段的当前阶段指access_by_lua,content_by_lua等阶段.  
 可查看[lua_nginx执行之各个阶段对应](lua_nginx执行之各个阶段对应.md)
# 举例
``` 
location / {
    access_by_lua_block{
        ngx.log(ngx.ERR,"this is access phase")
        ngx.exit(400)
    }
    content_by_lua_block{
        ngx.log(ngx.ERR,"this is content phase")
        ngx.say("output from content")
    }
    log_by_lua_block{
        ngx.log(ngx.ERR,"this is log phase")
    }
    header_filter_by_lua_block{
        ngx.log(ngx.ERR,"this is header phase")
    }
}
```
测试状态码400:
``` 
2017/11/17 02:36:05 [error] 3985#0: *27 [lua] access_by_lua(nginx.conf:22):2: this is access phase, client: 192.168.233.1, server: _, request: "GET /test HTTP/1.1", host: "foo.com"
2017/11/17 02:36:05 [error] 3985#0: *27 [lua] header_filter_by_lua:2: this is header phase, client: 192.168.233.1, server: _, request: "GET /test HTTP/1.1", host: "foo.com"
2017/11/17 02:36:05 [error] 3985#0: *27 [lua] log_by_lua(nginx.conf:29):2: this is log phase while logging request, client: 192.168.233.1, server: _, request: "GET /test HTTP/1.1", host: "foo.com"
```
由上,跳过了`content`阶段

测试100:
``` 
2017/11/17 02:38:37 [error] 3992#0: *30 [lua] access_by_lua(nginx.conf:22):2: this is access phase, client: 192.168.233.1, server: _, request: "GET /test HTTP/1.1", host: "foo.com"
2017/11/17 02:38:37 [error] 3992#0: *30 [lua] content_by_lua(nginx.conf:26):2: this is content phase, client: 192.168.233.1, server: _, request: "GET /test HTTP/1.1", host: "foo.com"
2017/11/17 02:38:37 [error] 3992#0: *30 [lua] header_filter_by_lua:2: this is header phase, client: 192.168.233.1, server: _, request: "GET /test HTTP/1.1", host: "foo.com"
2017/11/17 02:38:37 [error] 3992#0: *30 [lua] log_by_lua(nginx.conf:29):2: this is log phase while logging request, client: 192.168.233.1, server: _, request: "GET /test HTTP/1.1", host: "foo.com"
```
`content`阶段也被执行了.而且返回的最终状态码为"200"

参考: https://segmentfault.com/a/1190000004534300