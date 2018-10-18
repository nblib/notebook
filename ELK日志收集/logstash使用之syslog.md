# 概述

# 使用
``` 
input{
    syslog{
        port => 4699
    }
}
filter{
    kv{}
#   date{
#       match => ["savedate","yyyyMMdd"]
#       remove_field => ["method"]
#       tag_on_failure => ["savedateparse error"]
#       timezone => "Asia/Shanghai"
#       target => "gendate"
#   }
    syslog_pri{
    }
   mutate {
    # Renames the 'HOSTORIP' field to 'client_ip'
    rename => { "syslog_facility_code" => "facility" }
  }
}
output{
    stdout{
        codec => rubydebug
    }
#   elasticsearch{
#       hosts => ["10.169.3.219:9200"]
#   }
#   kafka {
#       bootstrap_servers => "10.169.0.214:9092"
#       codec => json
#        topic_id => "logstash_test"
#      }
}
```