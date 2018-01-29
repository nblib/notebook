# 概述
调试`openresty`方法和过程
# 步骤
## 环境
* openresty
* ZeroBrane
## 下载软件
[ZeroBrane](http://notebook.kulchenko.com/)
## 创建`nginx`配置文件
``` 
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    lua_package_path '/opt/zbstudio/lualibs/?/?.lua;/opt/zbstudio/lualibs/?.lua;;';
    lua_package_cpath '/opt/zbstudio/bin/linux/x64/clibs/?.so;;';
    lua_code_cache off;
    server {
        listen 8080;
        location /hellolua {
           default_type 'text/plain';
           content_by_lua_file '/home/hewe/Documents/lua_demo/content.lua';
        }
    }
}

```
* 其中`/opt/zbstudio`为安装的ZeroBrane的安装位置
* 如果环境时`OSX`,修改`?.so`为` ?.dylib`,`windows`修改为`?.dll`
* `content.lua`为要调试的lua文件
## 创建要调试的lua文件
``` 
require('mobdebug').start('192.168.1.22')
local name = ngx.var.arg_name or "Anonymous"
ngx.say("Hello, ", name, "!")
ngx.say("Done debugging.")
require('mobdebug').done()
```
* `192.168.1.22`为本机的ip,不可以使用`localhost`
## 配置 `zeroBrane`
1. 确保勾选`project->start debugger server`
2. 打开要调试文件`content.lua`,点击`Project | Project Directory | Set From Current File.`
## 启动`nginx`
启动`openresty`后,使用浏览器进入`localhost/hellolua`,如果配置正确,这个时候可以看到`ZeroBrane`中已经进入了断点调试模式.
# 远程调试
比如要调试文件`content.lua.
## 配置运行openresty的机器
* 复制`<ZBS>/lualibs/mobdebug/mobdebug.lua`到` content.lua`同一个目录下.
* 复制`<ZBS>/lualibs/socket.lua`到` content.lua`同一个目录下.
* 复制`<ZBS>/bin/clibs/socket/core.so`到content.lua同一个目录下的`socket`文件夹中.

最后的目录结构:
```
- content.lua
- mobdebug.lua
- socket.lua
- socket
---- core.so
```
* content.lua中,`require('mobdebug').start('10.169.3.135')`,ip填写要运行`ZeroBrane`的主机的ip地址
## 配置`zeroBrane`运行的主机
复制一份content.lua到机器上
1. 确保勾选`project->start debugger server`
2. 打开要调试文件`content.lua`,点击`Project | Project Directory | Set From Current File.`
## 启动`nginx`
启动`openresty`后,使用浏览器进入`localhost/hellolua`,如果配置正确,这个时候可以看到`ZeroBrane`中已经进入了断点调试模式.

# 注意问题
* 如果需要实时修改调试而不重启openresty,确保配置`lua_code_cache off;`
* 远程调试`redis`时,由于调试和运行不在同一位置,会出现超时情况,这时,设置redis.timeout()时间长一些,比如一百秒:`100000`
# 参考
http://notebook.kulchenko.com/zerobrane/debugging-openresty-nginx-lua-scripts-with-zerobrane-studio