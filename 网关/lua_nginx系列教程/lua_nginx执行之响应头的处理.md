# 问题
响应头内容过多,或者含有不想输出的内容.
# 解决思路
nginx的处理阶段最后两个阶段为`content`和`log`,一般在content执行完后,已经形成了返回信息.nginx为了处理返回信息,添加了两个过滤器:`header_filter`和`body_filter`
对返回的头和体进行处理,对应的,在lua中为`header_filter_by_lua_*`和`body_filter_by_lua_*`.但是这两个方法在不清楚的情况下会跳过执行,于是使用`openresty`的一个模块`Headers More Nginx Module`进行处理,
这个模块是内置到`openresty`中的,可以直接使用.更多这个模块的内容,参照:[Headers More Nginx Module](https://github.com/openresty/headers-more-nginx-module#directives)
# 解决方法
### 推荐方法
在`nginx.conf`配置文件中添加:
```
http{
     more_set_headers    "Server: my_server";
}
```
## 以下两个方法,当使用`ngx.exit(status)`退出时,会不执行.
### 使用openresy
* 在`http`,`server`,`location`中添加`header_filter_by_lua_block`,比如:
```
location /{
    header_filter_by_lua_block{
        ngx.header["Server"] = nil
    }
}
```
通过`ngx.header.*`或者`ngx.header[*]`获取响应头或者设置响应头,设置为`nil`表示不返回.
### 使用orange
在`orange.lua`中修改:
``` 
function Orange.header_filter()

    if ngx.ctx.ACCESSED then
        local now_time = now()
        ngx.ctx.ORANGE_WAITING_TIME = now_time - ngx.ctx.ORANGE_ACCESS_ENDED_AT -- time spent waiting for a response from upstream
        ngx.ctx.ORANGE_HEADER_FILTER_STARTED_AT = now_time
    end

    for _, plugin in ipairs(loaded_plugins) do
        plugin.handler:header_filter()
    end
    
    if ngx.ctx.ACCESSED then
        -- 在这里添加或删除请求头,比如下面这两个就可以删除
        ngx.header[HEADERS.UPSTREAM_LATENCY] = ngx.ctx.ORANGE_WAITING_TIME
        ngx.header[HEADERS.PROXY_LATENCY] = ngx.ctx.ORANGE_PROXY_LATENCY
    end
end
```
## 最复杂的方法,也是最彻底的方法
### 源头解决nginx
[修改nginx服务器类型实现简单伪装(隐藏nginx类型与版本等)](http://www.jb51.net/article/81216.htm)

