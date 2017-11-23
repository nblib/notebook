# 概述
logstash作为日志消息的中间件,获取数据后经过过滤再发送到别的地方.
# 简单实用
比如,获取数据从控制台,然后输出到控制台.
### 编写配置文件
log_std.conf
``` 
input{
        stdin{}
}
output{
        stdout{}
}
```
* stdin/stdout: 为控制台的输入和输出
### 测试配置文件
通过命令测试配置文件是否正确:
``` 
bin/logstash -f first-pipeline.conf --config.test_and_exit

```
### 启动
```
bin/logstash -f /path/to/configFile.conf
# --config.reload.automatic,以自动重新加载模式启动,这样改了配置文件后,不用手动.使用了`stdin`插件的时候,不能使用
bin/logstash -f /path/to/configFile.conf --config.reload.automatic

```
> 以自动重新加载模式启动,这样改了配置文件后,不用手动.**使用了`stdin`插件的时候,不能使用**

### 测试
启动后
``` 
[root@localhost logstash-5.6.3]# ./bin/logstash -f config/simple_log.conf 
Sending Logstash's logs to /home/ting/logstash-5.6.3/logs which is now configured via log4j2.properties
[2017-11-23T16:30:07,768][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"fb_apache", :directory=>"/home/ting/logstash-5.6.3/modules/fb_apache/configuration"}
[2017-11-23T16:30:07,771][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"netflow", :directory=>"/home/ting/logstash-5.6.3/modules/netflow/configuration"}
[2017-11-23T16:30:08,017][INFO ][logstash.pipeline        ] Starting pipeline {"id"=>"main", "pipeline.workers"=>2, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>250}
[2017-11-23T16:30:08,052][INFO ][logstash.pipeline        ] Pipeline main started
The stdin plugin is now waiting for input:
[2017-11-23T16:30:08,398][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
nihao
2017-11-23T08:30:20.521Z localhost nihao
```
输入nihao,会在下面看到输出的`2017-11-23T08:30:20.521Z localhost nihao`

### 调试使用的配置
log_std.conf
``` 
input{
        stdin{}
}
# filter可以不需要,这里只是为了对比下,这个filter的作用: 解析输入的消息中的键值对,然后放到target的字段中.
filter{
    kv => {
        target => "kv"
    }
}
output{
        stdout{
            codec => rubydebug
        }
}
```
为了方便看到效果,一般输出内容通过使用`rubydebug`插件查看,可以**看到数据再logstash处理后还没有输出的内容样式.**

比如,以上的例子,输入内容和输出内容消息内容一样看似没有什么变化,如果我们加入了别的插件,这样的结果看不出来插件运行效果.

所以,这里使用debug模式输出内容,可以看到具体的处理数据:

##### 启动
启动后,我们可以看到,输入内容和输出内容如下:
``` 
name=nihao
{
      "@version" => "1",
          "host" => "localhost",
    "@timestamp" => 2017-11-23T08:40:16.868Z,
            "kv" => {
        "name" => "nihao"
    },
       "message" => "name=nihao"
}
```

对比下上面的例子的输出: 上一个例子的输出:
``` 
2017-11-23T08:30:20.521Z localhost name=nihao
```
可以看到,未使用调试输出的内容只是输出了`时间,主机,和消息内容`,插件执行的结果`kv`没有显示.

# 注意
**虽然开启调试显示的内容和json相似,但这只是调试显示的内容,数据在logstash中储存可能是一个对象,或者一个集合.具体输出json还是xml由output决定**
