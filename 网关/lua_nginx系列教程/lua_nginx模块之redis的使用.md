# 摘要(Synopsis)
# 注意
* 不能在`init_worker_*`,`init_by_lua_*`阶段使用
* 域名解析出现超时,或不能解析的问题,建议使用`ip`
``` 
# you do not need the following line if you are using
# the OpenResty bundle:
lua_package_path "/path/to/lua-resty-redis/lib/?.lua;;";

server {
    location /test {
        content_by_lua_block {
            local redis = require "resty.redis"
            local red = redis:new()

            red:set_timeout(1000) -- 1 sec

            -- or connect to a unix domain socket file listened
            -- by a redis server:
            --     local ok, err = red:connect("unix:/path/to/redis.sock")

            local ok, err = red:connect("127.0.0.1", 6379)
            if not ok then
                ngx.say("failed to connect: ", err)
                return
            end

            ok, err = red:set("dog", "an animal")
            if not ok then
                ngx.say("failed to set dog: ", err)
                return
            end

            ngx.say("set result: ", ok)

            local res, err = red:get("dog")
            if not res then
                ngx.say("failed to get dog: ", err)
                return
            end

            if res == ngx.null then
                ngx.say("dog not found.")
                return
            end

            ngx.say("dog: ", res)

            red:init_pipeline()
            red:set("cat", "Marry")
            red:set("horse", "Bob")
            red:get("cat")
            red:get("horse")
            local results, err = red:commit_pipeline()
            if not results then
                ngx.say("failed to commit the pipelined requests: ", err)
                return
            end

            for i, res in ipairs(results) do
                if type(res) == "table" then
                    if res[1] == false then
                        ngx.say("failed to run command ", i, ": ", res[2])
                    else
                        -- process the table value
                    end
                else
                    -- process the scalar value
                end
            end

            -- put it into the connection pool of size 100,
            -- with 10 seconds max idle time
            local ok, err = red:set_keepalive(10000, 100)
            if not ok then
                ngx.say("failed to set keepalive: ", err)
                return
            end

            -- or just close the connection right away:
            -- local ok, err = red:close()
            -- if not ok then
            --     ngx.say("failed to close: ", err)
            --     return
            -- end
        }
    }
}
```
## 方法
#### `redis`方法
所有的`redis`方法都对应这[官方命令行方法](http://redis.io/commands)  

所有redis方法当成功时,会返回一个单一的结果,失败时,第一个结果为`nil`,第二值为字符串,描述错误原因.
``` 
local res, err = red:get("key")
```
根据[redis协议](),返回结果,会响应做以下处理:
* 单一字符串,返回string类型结果,去掉开头的"+"
* integer类型,返回number类型的结果
* 错误结果,返回一个false值,和一个string类型的描述信息
* 不为空的`bulk`响应,返回string串,空的响应返回`ngx.null`
#### `new`
```
syntax: red, err = redis:new()
```
创建一个redis实例,如果失败,返回nil和描述信息
#### `connect`
``` 
syntax: ok, err = red:connect(host, port, options_table?)

syntax: ok, err = red:connect("unix:/path/to/unix.sock", options_table?)
```
连接到远程的主机或者unix本地的domain socket(同一台主机进程之间通信),但,每次连接前,都会在连接池中寻找上次创建的空闲连接

最后一个参数可用于指定连接池的名称,```{pool = "define_name"}```,不指定,默认使用`<host>:<port>`或`<unix-socket-path>`
#### `set_timeout`
``` 
syntax: red:set_timeout(time)
```
为后续操作设置超时保护(单位:ms),包括`connect`方法
#### `set_keepalive`
``` 
syntax: ok, err = red:set_keepalive(max_idle_timeout, pool_size)
```
把当前redis连接立即放到`cosocket`连接池中.

通过参数指定连接空闲时间和连接池最大容量,每个工作进程对应一个连接池,互不影响,成功返回`1`,失败返回`nil`和描述信息

调用这个方法在想关闭连接的时候,调用后,redis实例会转化为关闭状态,后续调用这个对象的操作(非`connect`),将会返回`closed`error

#### `get_reused_times`
``` 
syntax: times, err = red:get_reused_times()
```
获取连接被重用了多少次.只要是从连接池获取的连接,这个方法的返回值都不为0,可用来判断一个连接是否是从连接池中获取的.

#### `close`
``` 
syntax: ok, err = red:close()
```
关闭当前连接.成功返回`1`,失败返回`nil`和错误信息

## 认证
``` 
local redis = require "resty.redis"
local red = redis:new()
red:set_timeout(1000) -- 1 sec
local ok, err = red:connect("127.0.0.1", 6379)
if not ok then
    ngx.say("failed to connect: ", err)
    return
end
local res, err = red:auth("foobared")
if not res then
    ngx.say("failed to authenticate: ", err)
    return
end
```