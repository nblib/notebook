# 概况
lua_nginx模块的使用
# 环境准备
安装`open resty`或`lua_nginx_module`,查看查看[openresty的安装](../安装教程/openresty安装.md)
#简单用例
用例位置: [lua_ngx_demo](../example/lua_ngx_demo)  
运行命令:
```
cd lua_ngx_demo
nginx -p `pwd`/ -c conf/nginx.conf
```
* 进入`lua_ngx_demo`目录下面,运行`nginx`命令,`-p`指定运行路径,这样nginx基于这个目录生成日志等文件,`-c`指定配置文件.

#注意事项
* `ngx.location.capture`和 `ngx.location.capture_multi`命令的目标地址的`location`中不能包含 `add_before_body`, `add_after_body`, `auth_request`, `echo_location`, `echo_location_async`, `echo_subrequest`, or `echo_subrequest_async`命令
* 由于`nginx`内部限制, `cosocket` API 在下面的语句中不能使用:  `set_by_lua*`, `log_by_lua*`, `header_filter_by_lua*`, and `body_filter_by_lua`, 现在也不能在`init_by_lua`,`init_worker_by_lua*`中使用,但是后续可能会可以.
* '\\'在`nginx`配置文件中时,想要使用它,需要转义,如果同时在`lua`的代码中时,需要两次转义,比如:"\\\\\\\\d+"最后被转义为"\\d+",但是如果在`*_by_lua_file`,也就是在单独的文件中,而不在`nginx`配置 文件中时,则只需要两个反斜杠就可以,因为那样`nginx`不解析.
* `nginx`可能会中断一个请求当遇到下面的情况(等等):
    * 400 (Bad Request)
    * 405 (Not Allowed)
    * 408 (Request Timeout)
    * 413 (Request Entity Too Large)
    * 414 (Request URI Too Large)
    * 494 (Request Headers Too Large)
    * 499 (Client Closed Request)
    * 500 (Internal Server Error)
    * 501 (Not Implemented)
> 这意味着正常运行的阶段会被跳过，例如重写或访问阶段。这也意味着，无论如何，运行后的阶段，如logbylua，都无法访问那些通常在这些阶段中设置的信息。

# lua_nginx模块的命令
## 命令运行阶段
[阶段对应图](lua_nginx_命令执行阶段.png)
## 常用命令
#### init_by_lua_*
**syntax**: init_by_lua <lua-script-str>   
**context**: http  
**phase**: loading-config  
在加载配置文件的时候运行,一般用于预先加载需要的`lua module`.
#### init_worker_by_lua_*
**syntax**: init_worker_by_lua <lua-script-str>   
**context**: http    
**phase**:  starting-worker  
在每个工作进程启动前运行,通常用于创建一个工作进程的周期性任务,比如健康检查
#### set_by_lua_*
**syntax**: set_by_lua $res <lua-script-str> [$arg1 $arg2 ...]   
**context**: server, server if, location, location if    
**phase**:  rewrite  
通过lua语句为`$res`赋值,`$arg1`等为传给`lua`语句的参数
#### content_by_lua_*
**syntax**: content_by_lua <lua-script-str>   
**context**: location, location if    
**phase**:  content  
内容处理,不能和其他相同阶段的命令一块使用,比如:`proxy_pass`
#### rewrite_by_lua_*
**syntax**: rewrite_by_lua <lua-script-str>   
**context**: http, server, location, location if    
**phase**:  rewrite tail  
运行在重写阶段的最后,也就是运行在`nginx_rewriter_module`的后面.如果`nginx_rewriter_module`中重定向了,那么`rewrite_by_lua`就不会执行了.
#### access_by_lua_*
**syntax**:  access_by_lua <lua-script-str>   
**context**:  http, server, location, location if    
**phase**:  access tail  
运行在`access`阶段的最后.
#### header_filter_by_lua_*
**syntax**:  header_filter_by_lua <lua-script-str>   
**context**:   http, server, location, location if    
**phase**:  output-header-filter  
用于过滤返回的头信息,可以添加或者修改.
#### body_filter_by_lua_*
**syntax**:   body_filter_by_lua <lua-script-str>   
**context**:   http, server, location, location if    
**phase**:   output-body-filter  
用于操纵返回结果,返回结果放在`ngx.arg[1]`中,内容输出结束标志符放在`ngx.arg[2]`中,通过设置第二个参数为true,可以结束返回.如果在lua中改变了返回结果的长度,一定要把头信息中的`content_length`置为`nil`.

#### log_by_lua_*
运行在日志记录阶段
#### balancer_by_lua_*
运行在`content`阶段