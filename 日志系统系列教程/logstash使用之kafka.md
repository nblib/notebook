# 概述
# 使用
### 配置
新建logstash配置文件: kafka_log.conf
``` 
input{
    stdin{}
}
output{
    # 测试,用于观察
    stdout{
        codec => rubydebug
    }
    kafka {
        bootstrap_servers => "10.169.0.214:9092,10.169.0.218:9092,10.169.0.219:9092"
        codec => json
        topic_id => "logstash_test"
      }
}
```
* bootstrap_servers:配置kafka服务器
* codec: 这里编码为json,可以设置别的
* topic_id: 对应服务器上创建的topic

新建一个消费者: [demo-kafka](https://github.com/nblib/demo-kafka)

### 测试
启动logstash:
``` 
./bin/logstash -f config/kafka_log.conf
```
启动消费者
``` 
运行 demo-kafka 的 DemoKafkaApplication 的main方法
```
可以看到
``` 

2017-11-24 14:39:46.784  INFO 15588 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.AbstractCoordinator  : Successfully joined group logstash_test with generation 2
2017-11-24 14:39:46.785  INFO 15588 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : Setting newly assigned partitions [logstash_test-0] for group logstash_test
2017-11-24 14:39:46.788  INFO 15588 --- [ntainer#0-0-C-1] o.s.k.l.KafkaMessageListenerContainer    : partitions assigned:[logstash_test-0]
```
输入消息,在logstash的控制台输入消息:
``` 
nihao
```
可以看到,logstash控制台输出:
``` 
nihao
{
      "@version" => "1",
          "host" => "localhost",
    "@timestamp" => 2017-11-24T07:09:01.967Z,
       "message" => "nihao"
}
```
然后,消费者,demo-kafka控制台输出:
``` 
2017-11-24 15:09:00.670  INFO 15588 --- [ntainer#0-0-C-1] d.h.demokafka.consumer.LogstashConsumer  : receive data : {"@version":"1","host":"localhost","@timestamp":"2017-11-24T07:09:01.967Z","message":"nihao"}

```
