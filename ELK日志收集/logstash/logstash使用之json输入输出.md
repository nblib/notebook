# 概述
如果发送给logstash的数据内容为`json`格式,那么可以通过解析json内容,根据具体内容生成字段.方便分析和储存,比如:

有一个`json`内容为: ```{"name":"nihao"}```,我们需要获取这个记录然后通过logstash分析后,放到mysql数据库中.

一个简单的logstash输出内容为:
``` 
{
      "@version" => "1",
          "host" => "localhost",
    "@timestamp" => 2017-11-23T08:40:16.868Z,
       "message" => "{\"name\":\"nihao\"}"
}
```
也就是说,如果存到数据库中,会保存以上四个字段到表中,而我们的记录的内容被简单的记录到`message`字段中,如果我们需要分析这条
记录,还得自己取出来进行处理.

如果使用了logstash的json解析,那么会是这个样子:
``` 
{
      "@version" => "1",
          "host" => "localhost",
    "@timestamp" => 2017-11-23T08:40:16.868Z,
       "message" => "name=nihao",
       "name" => "nihao"
}
```
这样存到数据库中就多了`name`字段,分析这条记录的时候只要取出来就可以.如果不想要`message`字段,也可以省去:
``` 
{
      "@version" => "1",
          "host" => "localhost",
    "@timestamp" => 2017-11-23T08:40:16.868Z,
       "name" => "nihao"
}
```

### json输出
如果将一个输入的记录处理后,输出为json串: 

比如输入: "nihao",

那么输出的结果为:
``` 
{"@version":"1","host":"localhost","@timestamp":"2017-11-23T09:42:14.798Z","message":"nihao"}
```
配置信息如下:
``` 
input {
    stdin{}
}
output{
    stdout{
        codec => json
    }
}
```
除了输入的内容外,还添加了一些额外的信息.

# 使用
logstash处理json输入有两种方式:
* 直接在输入阶段进行处理
* 通过过滤器处理

> 二者区别: [输入阶段和过滤阶段处理数据的区别](logstash使用之输入阶段和过滤阶段处理数据的区别.md)

### 配置
##### 输入阶段处理
json_input_log.conf
``` 
input{
        stdin{
            codec => json
        }
}
output{
        stdout{codec => rubydebug}
}
```
##### 过滤阶段处理
json_filter_log.conf
``` 
input{
        stdin{}
}
filter{
    json{
        source => "message"
    }
}
output{
        stdout{codec => rubydebug}
}
```
可以看到,输入阶段处理是通过codec对输入内容进行处理,过滤阶段通过`json`过滤器插件进行处理
### 结果
##### 输入阶段处理
``` 
{"name":"hewe"}
{
          "name" => "hewe",
      "@version" => "1",
          "host" => "localhost",
    "@timestamp" => 2017-11-23T09:09:01.533Z
}
```
##### 过滤阶段处理
``` 
{"name":"hewe"}
{
      "@version" => "1",
          "host" => "localhost",
          "name" => "hewe",
    "@timestamp" => 2017-11-23T09:05:12.004Z,
       "message" => "{\"name\":\"hewe\"}"
}
```
###### 注意
**虽然开启调试显示的内容和json相似,但这只是调试显示的内容,数据在logstash中储存可能是一个对象,或者一个集合.具体输出json还是xml由output决定**

通过以上的对比,可以看到,过滤阶段比输入阶段多了一个字段`message`,这也就表明,当在输入阶段处理时,是直接把输入的内容解析成相应的字段,
而在过滤阶段处理时,输入的内容已经被放到`message`字段中,然后通过设置`source`解析message为相应的字段.

# 结语
输入输出阶段处理区别请看: [输入阶段和过滤阶段处理数据的区别](logstash使用之输入阶段和过滤阶段处理数据的区别.md)

具体使用哪一种分情况而定.
