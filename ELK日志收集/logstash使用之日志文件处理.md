# 概述
日志文件处理,是通过logstash将log文件中的内容经过处理,传输到别的地方.比如,从web应用输出的日志文件中提取内容发送到es中,进行
分析处理.
# 准备
* Filebeat: 读取文件的插件([下载](https://www.elastic.co/downloads/beats/filebeat))
* logstash
* es
* 日志实例文件(下载后,解压,将内容改名为`logstash-tutorial.log`)([下载地址](https://download.elastic.co/demos/logstash/gettingstarted/logstash-tutorial.log.gz))
# 开始
## 简单使用
### 配置`filebeat`
下载后,解压,然后进入目录编辑文件`filebeat.yml`
``` 
filebeat.prospectors:
- type: log
  # 必须设为true,不然不会工作
  enabled: true
  paths:
    - /path/to/file/logstash-tutorial.log 
#-------------------------- Elasticsearch output 先注释掉------------------------------
#output.elasticsearch:
  # Array of hosts to connect to.
  #hosts: ["localhost:9200"]

  # Optional protocol and basic auth credentials.
  #protocol: "https"
  #username: "elastic"
  #password: "changeme"

#----------------------------- Logstash output 开启--------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5044"]

  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]

  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"

  # Client Certificate Key
  #ssl.key: "/etc/pki/client/cert.key"
```
### 启动 `Filebeat`
``` 
./filebeat -e -c filebeat.yml -d "publish"
```
### 配置`logstash`
一个基本的配置文件为:
``` 
# The # character at the beginning of a line indicates a comment. Use
# comments to describe your configuration.
input {
}
# The filter part of this file is commented out to indicate that it is
# optional.
# filter {
#
# }
output {
}
```
新建一个文件,随便放置到哪里都可以.然后,编辑输入和输出
``` 
# 监听这个端口和filebeat发送的端口相同,
input {
    beats {
        port => "5044"
    }
}
# The filter part of this file is commented out to indicate that it is
# optional.
# filter {
#
# }
# 输出到控制台先,以debug模式输出
output {
    stdout { codec => rubydebug }
}
```
测试下配置文件是否正确
```
bin/logstash -f first-pipeline.conf --config.test_and_exit
```
### 运行logstash
``` 
bin/logstash -f /path/to/configFile.conf --config.reload.automatic
```
> 以自动重新加载模式启动,这样改了配置文件后,不用手动.
### 验证
启动后,会看到控制台输出内容
``` 
{
    "@timestamp" => 2017-11-09T01:44:20.071Z,
        "offset" => 325,
      "@version" => "1",
          "beat" => {
            "name" => "My-MacBook-Pro.local",
        "hostname" => "My-MacBook-Pro.local",
         "version" => "6.0.0"
    },
          "host" => "My-MacBook-Pro.local",
    "prospector" => {
        "type" => "log"
    },
        "source" => "/path/to/file/logstash-tutorial.log",
       "message" => "83.149.9.216 - - [04/Jan/2015:05:13:42 +0000] \"GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1\" 200 203023 \"http://semicomplete.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
          "tags" => [
        [0] "beats_input_codec_plain_applied"
    ]
}
...
```
可以看到,日志文件中的内容,被放到了`message`中
## 解析web日志使用
把日志整条信息放到一个字段中,看起来不好,可以使用web的插件,将内容解析出来.这个插件`grok`用于解析apache的日志格式.这里先简单使用
具体自定义以后再说.

### 编辑logstash的配置文件,添加`filter`:
``` 
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}
```
### 停止filebeat,然后删除注册文件(发送过的文件记录再这里,删除后,会重新发)
``` 
rm filebeat/data/registry
```
### 重启filebeat
``` 
./filebeat -e -c filebeat.yml -d "publish"
```
### 查看logstash
``` 
{
        "request" => "/presentations/logstash-monitorama-2013/images/kibana-search.png",
          "agent" => "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
         "offset" => 325,
           "auth" => "-",
          "ident" => "-",
           "verb" => "GET",
     "prospector" => {
        "type" => "log"
    },
         "source" => "/path/to/file/logstash-tutorial.log",
        "message" => "83.149.9.216 - - [04/Jan/2015:05:13:42 +0000] \"GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1\" 200 203023 \"http://semicomplete.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
           "tags" => [
        [0] "beats_input_codec_plain_applied"
    ],
       "referrer" => "\"http://semicomplete.com/presentations/logstash-monitorama-2013/\"",
     "@timestamp" => 2017-11-09T02:51:12.416Z,
       "response" => "200",
          "bytes" => "203023",
       "clientip" => "83.149.9.216",
       "@version" => "1",
           "beat" => {
            "name" => "My-MacBook-Pro.local",
        "hostname" => "My-MacBook-Pro.local",
         "version" => "6.0.0"
    },
           "host" => "My-MacBook-Pro.local",
    "httpversion" => "1.1",
      "timestamp" => "04/Jan/2015:05:13:42 +0000"
}
```
可以看到,把日志的内容解析成单个的字段:ip,name,响应状态等
## 发送日志到es中
只需要修改logstash的配置文件为:
``` 
input {
    beats {
        port => "5043"
    }
}
 filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}
# 这里改为es的监听位置
output {
    elasticsearch {
        hosts => [ "localhost:9200" ]
    }
}
```
然后会自动再es中添加`logstash-yyyy-MM-dd`的索引
