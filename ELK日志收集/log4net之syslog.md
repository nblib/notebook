# 概述
使用log4net通过网络传输`syslog`日志
# 配置
``` 
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <configSections>
    <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net"/>
  </configSections>
  <log4net>
    <!-- 控制台前台显示日志 -->
    <appender name="ColoredConsoleAppender" type="log4net.Appender.ColoredConsoleAppender">
      <mapping>
        <level value="ERROR" />
        <foreColor value="Red, HighIntensity" />
      </mapping>
      <mapping>
        <level value="Info" />
        <foreColor value="Green" />
      </mapping>
      <layout type="log4net.Layout.PatternLayout">
        <conversionPattern value="%n%date{HH:mm:ss,fff} [%-5level] %m" />
      </layout>

      <filter type="log4net.Filter.LevelRangeFilter">
        <param name="LevelMin" value="Info" />
        <param name="LevelMax" value="Fatal" />
      </filter>
    </appender>
    <appender name="RemoteSyslogAppender" type="log4net.Appender.RemoteSyslogAppender">
      <remoteAddress value="192.168.233.129" />
      <remotePort value="4699" />
      <layout type="log4net.Layout.PatternLayout" value="%-5p %type: %m%n"/>
    </appender>
    <root>
      <!--(高) OFF > FATAL > ERROR > WARN > INFO > DEBUG > ALL (低) -->
      <level value="all" />
      <appender-ref ref="RemoteSyslogAppender"/>
    </root>
  </log4net>
</configuration>
```
* 通过使用`log4net.Appender.RemoteSyslogAppender`进行传输

# 和`logstash`结合
和`logstash`结合使用的话,`RemoteSyslogAppender`会发送如下格式的内容:
``` 
<9>log4net_demo.vshost.exe: FATAL log4net_demo.Program: fffffffffffffffff
```
* <9>: 表示优先级,结合了`syslog`标准的`facility`和`severity`,具体计算过程为:9的二进制的低三位表示`severity`,即`severity`
为1,按照`syslog`标准为`error`级别,其余位右移三位表示`facility`,即`facility`为1,表示`user-level`级别的信息.
* `log4net_demo.vshost.exe`: 表示日志的输出源
* `:`: 表示分割符,后面的内容为真正输出的日志内容.

> `syslog`可以参考: http://blog.csdn.net/smstong/article/details/8919803

如上,`RemoteSyslogAppender`的syslog格式(并非在log4net中配置的格式)为: `<优先级><日志源>:<日志内容>`,然而logstash解析的syslog
格式为`<优先级><时间> 主机 日志源:<日志内容>`',和log4net输出不同,解析会出错.

### 解决方法
有两种:
* 修改`RemoteSyslogAppender`为`UdpAppender`,自己定义格式;
* 修改logstash的接收为`udp`,然后自己定义pattern,

下面介绍第二种:

log4net的日志,使用logstash接收后会出现无法解析`facility`和`severity`的情况,修改input,配置logstash添加插件
``` 
input {
    udp{
        port => 3423
    }
}
filter{
     grok {
         match => {
             "message" => "<%{POSINT:priority}>"
         }
     }   
     syslog_pri {
         syslog_pri_field_name => "priority"
     }
}
```
* 使用grok自定义 `match`提取message中的优先级编号,放到字段`priority`中
* `syslog_pri`用于解析`priority`字段,解析出来`facility`和`severity`,生成的结果如下:
``` 
{
              "@timestamp" => 2017-11-27T06:28:17.833Z,
    "syslog_severity_code" => 1,
         "syslog_facility" => "user-level",
                "@version" => "1",
                    "host" => "192.168.233.1",
    "syslog_facility_code" => 1,
                 "message" => "<9>log4net_demo.vshost.exe: FATAL log4net_demo.Program: fffffffffffffffff",
                "priority" => "9",
         "syslog_severity" => "alert"
}
```
* syslog生成`syslog_`开头的解析结果

#### 提取日志源和覆盖`message`
下面是从logstash的插件`logstash-input-syslog`中提取出来的几个表达式,用于提取日志内容中的日志源和真正记录的内容覆盖原始的
`message`:
``` 
 grok {
        match => {
            "message" => "<%{POSINT:priority}>%{SYSLOGHOST:logsource}+:%{GREEDYDATA:message}"
        }
        overwrite => ["message"]
    }
```
输入:`<23>demo.vshost.exe:this is log`生成如下:
``` 
{
              "@timestamp" => 2017-11-27T09:16:16.942Z,
    "syslog_severity_code" => 7,
         "syslog_facility" => "mail",
                "@version" => "1",
                    "host" => "192.168.233.129",
    "syslog_facility_code" => 2,
                 "message" => "this is log",
                "priority" => "23",
               "logsource" => "demo.vshost.exe",
         "syslog_severity" => "debug"
}
```
##### 注意
存在`%{SYSLOGHOST:logsource}`为路径的情况时,比如`/LM/W3SVC/7`,可以使用正则匹配两个
``` 
"message" => "<%{POSINT:priority}>(?:%{UNIXPATH:logsource}|%{SYSLOGHOST:logsource}):%{GREEDYDATA:message}"
```
* `UNIXPATH`,匹配带有`/`的字符串
* `SYSLOGHOST`匹配带有`.`的字符串

将二者添加为或关系可以选择性的匹配.