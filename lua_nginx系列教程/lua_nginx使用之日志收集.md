# 概述
将日志记录发送到远程服务器,对日志记录进行分析.
# 环境准备
* openresty: lua_nginx集成环境
* lua-resty-logger-socket库: 用于发送日志到远程服务器
* logstash: 远程日志收集服务器
* elasticsearch: 日志储存和分析


# 使用
## nginx配置
```
location /log{
    content_by_lua_block{
        ngx.say("nihao")
    }
    log_by_lua_block{
        local logger = require "resty.logger.socket"
        if not logger.initted() then
            local ok, err = logger.init{
                host = '166.66.0.15',
                port = 3567,
                flush_limit = 1,
                drop_limit = 5678,
            }
            if not ok then
                ngx.log(ngx.ERR, "failed to initialize the logger: ",
                        err)
                return
            end
        end

        -- construct the custom access log message in
        -- the Lua variable "msg"

        local bytes, err = logger.log('access time : '..ngx.localtime()..'\n')
        if err then
            ngx.log(ngx.ERR, "failed to log message: ", err)
            return
        end
        --logger.flush()
    }
}
```
### lua-resty-logger-socket库
#### 方法
##### init
``` 
syntax: ok, err = logger.init(user_config)
```
用于初始化,使用之前必须初始化,可以通过,if判断是否初始化.

配置列表:
* `flush_limit`

    如果缓冲区的消息大小加上当前消息大小`>=`这个值(以byte为单位),将缓冲区的内容发送到远程服务器,默认值`4096`
    
*  `drop_limit`

    如果缓冲区的消息大小加上当前消息大小`>`这个值(以byte为单位),将会被丢弃,因为缓冲区大小有限.默认` 1048576 (1MB)`
* `timeout`
    
    设置超时,保证后续操作,包括连接操作,默认` 1000 (1 sec)`
    
* `host`

    log server host.
    
* `port`

    log server port number.

* `sock_type`

    IP protocol type to use for transport layer. Can be either "tcp" or "udp". Default is "tcp".

* `path`
  
  If the log server uses a stream-typed unix domain socket, path is the socket file path. Note that host/port and path cannot both be empty. At least one must be supplied.
  
*  `max_retry_times`
  
  最大重试次数.Max number of retry times after a connect to a log server failed or send log messages to a log server failed.
  
*  `retry_interval`
  
  重试间隔,The time delay (in ms) before retry to connect to a log server or retry to send log messages to a log server, default to 100 (0.1s).
  
*  `pool_size`
  
  连接远程服务器的连接池大小, Default to 10.
  
*  `max_buffer_reuse`
  
  Max number of reuse times of internal logging buffer before creating a new one (to prevent memory leak).
  
*  `periodic_flush`
  
  Periodic flush interval (in seconds). Set to nil to turn off this feature.
  
*  `ssl`
  
  Boolean, enable or disable connecting via SSL. Default to false.
  
*  `ssl_verify`
  
  Boolean, enable or disable verifying host and certificate match. Default to true.
  
*  `sni_host`
  
  Set the hostname to send in SNI and to use when verifying certificate match.
  
##### initted

``` 
syntax: initted = logger.initted()
```
是否初始化.在调用init方法前使用.决定是否调用init方法

###### log
``` 
syntax: bytes, err = logger.log(msg)
```
记录日志,日志将被记到缓冲区中,返回`bytes`,日志的大小. 如果`bytes`是空的,err保存错误信息,如果`bytes`不为空,err中为上一次的
错误信息.

##### flush
``` 
syntax: bytes, err = logger.flush()
```
清理缓冲区的内容到远程服务器,不需要手动调用,当缓冲区满了后,会自动调用.
## logstash配置
修改logstash.conf.
``` 
input{
    syslog {
        port => 3567
    }
}
filter {
  #Only matched data are send to output.
}
output {
  # For detail config for elasticsearch as output,
  # See: https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html
  elasticsearch {
    action => "index"          #The operation on ES
    hosts  => "10.169.3.219:9200"   #ElasticSearch host, can be array.
    index  => "accesslog"         #The index to write data to.
  }
}
```
* `input` 中,`port`表明监听的端口,和nginx中配置发送的端口相同.
* `output`为es的配置,将消息发送到es中.
## 测试
访问`http://localhost/log`,多次访问,然后进入es中查看,会发现索引`accesslog`中会存在记录:`access time : 2017-00-00`
## 注意的问题
nginx配置中,logger.log(msg),发送消息,msg需要添加`\n`换行符,否则,logstash以为是一条记录,会将后续的内容全部追加到一条记录中
``` 
local bytes, err = logger.log('access time : '..ngx.localtime()..'\n')
```