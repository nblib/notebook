# 概述
grok插件,用于从一个字段中通过正则表达式提取需要的内容到指定的字段中.
# 语法
``` 
%{NUMBER:duration} %{IP:client}
```
比如:
``` 
filter {
    grok {
        match {
            "message" => "%{NUMBER:duration} %{IP:client}"
        }
    }
}
```
* `NUMBER`,`IP`: 这两个为要引用的正则表达式的名称,是默认定义的正则表达式的名称(在[这里定义](https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns)的),这里直接引用的.
* `duration`,`client`为匹配到后的内容要存放的字段名.
* `message`: 要从哪一个先用的字段中提取.

如上,比如传入一条内容为`message="23 119.29.2.209"`,那么匹配后结果为
``` 
"message" => "23 119.29.2.209"
"duration" => 23
"client" => "119.29.2.209"
```
### 自定义正则表达式
##### 在配置里定义
``` 
(?<queue_id>[0-9A-F]{10,11})
```
* `queue_id`: 为匹配到后的内容要存放的字段名.
* `[0-9A-F]{10,11}`: 自定义的正则表达式
##### 在外面的文件里配置
新建一个文件,名称自定义`pattern_file`,内容为表达式名称,表达式:
``` 
POSTFIX_QUEUEID [0-9A-F]{10,11}
```
然后在配置文件中引用:
``` 
filter {
  grok {
    patterns_dir => ["/path/to/pattern_file"]
    match => { "message" => "%{POSTFIX_QUEUEID:field_name}" }
  }
}
```
