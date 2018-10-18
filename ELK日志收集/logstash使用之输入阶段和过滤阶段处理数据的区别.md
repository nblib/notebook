# 概述
阅读此文之前,先阅读[logstash入门之工作流程](logstash入门之工作流程.md),了解下codec和filter.

codec相当于一个编码解码的工具.对输入和输出的数据进行处理,而filter中也有好多类似于这个功能的插件.比如:
* codec中有`json codec plugin`,filter中有`json filter plugin`

codec作用于输入阶段可以对输入的内容比如json进行解析,而filter中同样也有可以处理json的插件.具体使用哪一个需要视情况而定.

# 对比
比如处理一个json串:`{"name":"hewe"}`

### codec
codec在输入阶段处理时,结果类似以下:
``` 
{
          "name" => "hewe",
      "@version" => "1",
          "host" => "localhost",
    "@timestamp" => 2017-11-23T09:09:01.533Z
}
```
可以看到,直接将输入的json串解析出来放到了字段对应的位置中.在以后的步骤中处理时,看不到原始数据的.
### filter
结果类似如下:
``` 
{
      "@version" => "1",
          "host" => "localhost",
          "name" => "hewe",
    "@timestamp" => 2017-11-23T09:05:12.004Z,
       "message" => "{\"name\":\"hewe\"}"
}
```
如果不设置删除message,可以看到,输入的原始数据`message`

codec和filter处理的结果来看,如果在后续的处理中需要使用原始数据,可以使用filter来决定是否保留原始数据.

# 不同情形应用
比如有一个日志如下: 
```
this is a log record :{"name":"hewe"}
```
可以看到,这里的记录是一个普通的文本中包含了一个json字符串,这种情况下如果使用`codec`处理的话就会报错,因为这个记录并非一个
json格式,而我们要解析其中的json串时,只能使用filter来进行处理:先提取出里面的json串,然后在解析.

# 总结
* codec使用比较简单,在输入阶段就将输入的内容解析成单个字段,不保留原始数据,如果原始数据不是json格式那么解析就会出错.
* filter处理相对灵活,可以提取普通文本包含的json串,同时可以选择保留原始数据进行处理.

如果确定输入的内容为json字符串,使用codec好些,不确定或者需要对原始数据进行处理,使用filter好些.