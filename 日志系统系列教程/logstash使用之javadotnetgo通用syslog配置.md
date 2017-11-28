# 概述
通用解析java,C#,go程序发过来的syslog日志
``` 
input{
    udp{
        port => 4699
    }
}
filter{
    grok {
        match => {
            "message" => "<%{POSINT:priority}>(?:%{SYSLOGTIMESTAMP:timestamp} |%{TIMESTAMP_ISO8601:timestamp8601} )?(?:%{SYSLOGFACILITY} )?(?:%{SYSLOGHOST:logsource}|%{UNIXPATH:logsource})+(?: %{SYSLOGPROG}|): %{GREEDYDATA:message}"
        }
        overwrite => ["message"]
    }
    syslog_pri {
        syslog_pri_field_name => "priority"
    }
}
output{
    stdout{
        codec => rubydebug
    }
}
```