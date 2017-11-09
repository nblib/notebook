#概述
`nginx`的执行阶段分为多个阶段,每一个阶段执行的时候会调用挂载在这个阶段的模块和方法.

`lua_nginx_module`模块通过挂载在不同阶段与nginx相结合,将nginx的阶段对应为lua的命令.而orange又基于`lua_nginx_module`模块,对应了自己的方法.大致描述如下:
![阶段对应](img/nginx_lua_orange对应阶段.png)
当一个请求到来时,比如重写阶段`rewrite`,当nginx执行到这个阶段的时候,会执行绑定这个阶段的模块也就是lua模块中的`rewrite_by_lua*`中的代码,而orange则将自己对应的`Orange.rewrite()`方法放到了这里,于是就执行了Orange方法

当orange执行`Orange.rewrite()`方法时,遍历初始化找到的插件,然后调用插件的`hanlder:rewrite()`方法.

由上可知,自定义插件时只需要重写感兴趣的方法即可,
# 自定义插件
### 自定义健康检查插件
##### 环境准备
自定义健康检查插件是基于`resty.upstream.healthcheck`模块的一个插件,所以需要这个模块,默认是有的.
##### 文件准备
健康检查需要两个文件:`api.lua`和`handler.lua`,其中:
* api用于对外提供API接口
* handler也就是插件,用于重写方法,在相应的阶段做处理
##### 代码编写
在`orange/orange/plugins`下新建一个文件夹`health`要和插件名称对应,将上面提到到两个文件放到这里,然后编写.
###### api.lua
``` 
-- 获取API父类
local BaseAPI = require("orange.plugins.base_api")
-- 获取健康检查模块
local hc = require "resty.upstream.healthcheck"

--新建一个api,第一个是名字,基本没用,第二个参数表示是一个自定义路径的api
local api = BaseAPI:new("health-check", 2)
--自定义api路径
api:get("/health/stat",function ( store )
	-- body
	return function ( req, res, next )
		-- 直接返回健康检查模块返回的页面
		res:send(hc.status_page())
	end
end)

```
###### handler.lua
``` 
-- 加载模块和初始化
local BasePlugin = require("orange.plugins.base_handler")
local hc = require "resty.upstream.healthcheck"
local HealthCheckHandler = BasePlugin:extend()
--优先级
HealthCheckHandler.PRIORITY = 2000

function HealthCheckHandler:new(store)
    HealthCheckHandler.super.new(self, "health-plugin")
    self.store = store
end
--在初始化worker阶段调用这个插件
function HealthCheckHandler:init_worker()
	-- 
	local ok, err = hc.spawn_checker{
            shm = "healthcheck",  -- defined by "lua_shared_dict"
            upstream = "foo.com", -- defined by "upstream"
            type = "http",

            http_req = "GET /hello HTTP/1.0\r\nHost: foo.com\r\n\r\n",
                    -- raw HTTP request for checking

            interval = 2000,  -- run the check cycle every 2 sec
            timeout = 1000,   -- 1 sec is the timeout for network operations
            fall = 3,  -- # of successive failures before turning a peer down
            rise = 2,  -- # of successive successes before turning a peer up
            valid_statuses = {200, 302},  -- a list valid HTTP status code
            concurrency = 10,  -- concurrency level for test requests
        }
        if not ok then
            ngx.log(ngx.ERR, "failed to spawn health checker: ", err)
            return
        end
end
```
####### nginx.conf
由于健康检查模块需要共享内存,所以在nginx配置文件的`HTTP`下添加如下内容,这个名字要和上面`handler`中`hc.spawn_checker`的shm的值相同:
```
 lua_shared_dict healthcheck 1m;
```
这样一个插件编写完成,同时还需要在`conf/orange.conf`中添加这个插件的名称,和上面的文件夹名称相同:`health`
###测试
启动orange,访问`ip:9999/health/stat`可以看到如下信息,表示成功:
``` 
Upstream default_upstream (NO checkers)
    Primary Peers
        127.0.0.1:8001 up
        127.0.0.1:8001 up
    Backup Peers

Upstream foo.com
    Primary Peers
        127.0.0.1:8081 up
        127.0.0.1:8081 up
    Backup Peers
```