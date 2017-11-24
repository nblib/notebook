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
log4net的日志,使用logstash接收后会出现无法解析`facility`和`severity`的情况,可以配置logstash添加插件
``` 
filter{
    syslog_pri{
    }
}
```
* `syslog_pri`用于解析`log4net`的`PRI`字段,解析出来`facility`和`severity`,生成的结果如下:
``` 
{
                "severity" => 0,
    "syslog_severity_code" => 5,
         "syslog_facility" => "user-level",
    "syslog_facility_code" => 1,
                 "message" => "<11>log4net_demo.vshost.exe: ERROR log4net_demo.Program: eeeeeeeeeeeeeeee",
                "priority" => 0,
         "syslog_severity" => "notice",
                    "tags" => [
        [0] "_grokparsefailure_sysloginput"
    ],
              "@timestamp" => 2017-11-24T10:10:44.080Z,
                "@version" => "1",
                    "host" => "192.168.233.1",
                "facility" => 0,
          "severity_label" => "Emergency",
          "facility_label" => "kernel"
}
```
* syslog生成`syslog_`开头的解析结果