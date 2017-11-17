# 概述(Synopsis)
`nginx`从启动到收到请求,再到请求结束,整个执行过程被划分为多个阶段,每个阶段分别处理自己的事情.形成一个执行链.

lua-nginx模块在实现lua执行的过程也是基于这些阶段分别对应执行.

# 阶段
#### 启动阶段
首先看下nginx启动的过程被分为了哪些阶段:
* 配置解析
* 初始化模块
* 初始化共享内存
* 监听端口
* 创建worker
* 初始化worker
* worker开始接受请求

以上是简化版的启动过程,这里只关注过程,也只关注主要过程,大致明白nginx启动的流程,nginx启动完成后,即可接受请求.
#### 请求处理阶段
下面是请求的多阶段处理过程(省略请求的解析):
* NGX_HTTP_POST_READ_PHASE:  
接收完请求头之后的第一个阶段，它位于uri重写之前，实际上很少有模块会注册在该阶段，默认的情况下，该阶段被跳过;
nginx中`realip`模块挂载在这个阶段,当接受完请求头之后会进入该阶段,用于对请求头处理.
* NGX_HTTP_SERVER_REWRITE_PHASE:  
	server级别的uri重写阶段，也就是该阶段执行处于server块内，location块外的重写指令，前面的章节已经说明在读取请求头的过程中nginx会根据host及端口找到对应的虚拟主机配置；
* NGX_HTTP_FIND_CONFIG_PHASE:  
	寻找location配置阶段，该阶段使用重写之后的uri来查找对应的location，值得注意的是该阶段可能会被执行多次，因为也可能有location级别的重写指令；
* NGX_HTTP_REWRITE_PHASE:  
location级别的uri重写阶段，该阶段执行location基本的重写指令，也可能会被执行多次；
* NGX_HTTP_POST_REWRITE_PHASE:  
location级别重写的后一阶段，用来检查上阶段是否有uri重写，并根据结果跳转到合适的阶段；
* NGX_HTTP_PREACCESS_PHASE:  
访问权限控制的前一阶段，该阶段在权限控制阶段之前，一般也用于访问控制，比如限制访问频率，链接数等；
* NGX_HTTP_ACCESS_PHASE:  
访问权限控制阶段，比如基于ip黑白名单的权限控制，基于用户名密码的权限控制等；
* NGX_HTTP_POST_ACCESS_PHASE:  
访问权限控制的后一阶段，该阶段根据权限控制阶段的执行结果进行相应处理；
* NGX_HTTP_TRY_FILES_PHASE:  
try_files指令的处理阶段，如果没有配置try_files指令，则该阶段被跳过；
* NGX_HTTP_CONTENT_PHASE:  
内容生成阶段，该阶段产生响应，并发送到客户端；CONTENT阶段有些特殊，它不像其他阶段只能执行固定的handler链，还有一个特殊的content_handler，每个location可以有自己独立的content handler，而且当有content handler时，CONTENT阶段只会执行content handler，不再执行本阶段的handler链。
* NGX_HTTP_LOG_PHASE:  
日志记录阶段，该阶段记录访问日志。进入该阶段表明该请求的响应已经发送到**系统发送缓冲区**。LOG阶段和其他阶段的不同点有两个，一是执行点是在释放请求资源ngx_http_free_request中，二是这个阶段的所有handler都会被执行。
#### 过滤阶段
在CONTENT阶段产生的数据被发往客户端（**系统发送缓存区**）之前，会先经过过滤。**也就是说,这个阶段是在LOG阶段之前执行的.**
* header filter
* body filter
* ...

# lua_nginx模块的阶段对应
![阶段对应图](../img/nginx/nginx_lua/lua_nginx_命令执行阶段.png)

更多可参考[nginx之lua_nginx模块的使用](../nginx笔记/nginx使用/nginx之lua_nginx模块的使用.md)