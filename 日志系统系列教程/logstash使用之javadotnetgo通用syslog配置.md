# 概述
通用解析C#,go程序发过来的syslog日志
``` 
input{
        udp{
                port=> 3578
        }
}
filter{
 ruby{
                code => "event.set('date',Time.new.strftime('%Y-%m-%d'))"
        }
        grok {
        match => {
          "message" => "<%{POSINT:priority}>(?:%{SYSLOGTIMESTAMP:timestamp} |%{TIMESTAMP_ISO8601:timestamp8601} )?(?:%{SYSLOGFACILITY} )?(?:%{SYSLOGHOST:logsource}|%{UNIXPATH:logsource})+(?: %{SYSLOGPROG}|): %{GREEDYDATA:message}"
     }
        overwrite => ["message"]
    }
}
output{
         webhdfs {
    host => "l22-232-13"                 # (required)
    port => 14000                       # (optional, default: 50070)
    path => "/user/logstashtest/netlog/dt=%{date}/logstash-%{+HH}.log"  # (required)
    user => "logstash"                       # (required)
codec => plain { format => "%{message}"}
    compression => "gzip"
  }
#        stdout{codec=> rubydebug}
}
```
* `ruby` 用于调用ruby获取本地时区日期.
* match 适用于log4net的RemotesyslogAppender
* `output`的`codec`用于只输出自己需要的内容

**codec的`plain`和`line`都是自定义输出内容.不同的是line会自动行和行之间添加`\n`符,容易出现多余空行的问题**
java 配置
