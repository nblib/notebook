logstash的监控
===
# 开启监控
* 版本: 6.4.0

`6.4.0`版本默认安装`x-pack`插件,可以直接使用监控功能,修改logstash的配置文件`logstash.yml`:
```
node.name: es_logstash

xpack.monitoring.enabled: true
#xpack.monitoring.elasticsearch.username: logstash_system
#xpack.monitoring.elasticsearch.password: password
xpack.monitoring.elasticsearch.url: ["http://127.0.0.1:9200"]
#xpack.monitoring.elasticsearch.ssl.ca: [ "/path/to/ca.crt" ]
#xpack.monitoring.elasticsearch.ssl.truststore.path: path/to/file
#xpack.monitoring.elasticsearch.ssl.truststore.password: password
#xpack.monitoring.elasticsearch.ssl.keystore.path: /path/to/file
#xpack.monitoring.elasticsearch.ssl.keystore.password: password
#xpack.monitoring.elasticsearch.ssl.verification_mode: certificate
#xpack.monitoring.elasticsearch.sniffing: false
#xpack.monitoring.collection.interval: 10s
xpack.monitoring.collection.pipeline.details.enabled: true
```
* 指定一个唯一nodename,便于监控中区分
* 开启监控功能
* 指定es的地址(注意如果不需要使用ssl,那么要用http而不是https,不然会报错)
* 如果需要查看pipeline的详细信息,最后一行开启

通过设置,logstash会发送运行状态信息到es中,通过`kibana`的监控可以看到logstash的状态
