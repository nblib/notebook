# 概述
日期插件`Date filter plugin`用于对logstash接收到的字段中的日期进行处理,可以使用处理后的日期作为logstash的`timestamp`.

日期进行处理后,可以在kibana中用于统计和分析.

# 简单使用
``` 
input{
        stdin{
        }
}
filter{
    date{
        match => ["message","yyyyMMdd"]
    }
}
output{
        stdout{codec => rubydebug}
}
```
* 从标准控制台接收输入
* 使用`date`插件过滤,这里是一个简单的配置,使用`match`对`message`字段中的内容进行时间转化,默认将转化后的结果放到了`@timestamp`中.
* 使用标准控制台输出,输出使用了`rubydebug`插件,目的是输出调试结果看起来方便,内容也全面一些.

使用效果:
``` 
20171124
{
      "@version" => "1",
          "host" => "localhost",
    "@timestamp" => 2017-11-23T16:00:00.000Z,
       "message" => "20171124"
}
20171120
{
      "@version" => "1",
          "host" => "localhost",
    "@timestamp" => 2017-11-19T16:00:00.000Z,
       "message" => "20171120"
}

```
如上,`20171124`和`20171120`是控制台输入的,其余内容时`rubydebug`插件输出的内容.可以看到:
* 输入的内容默认保存在了`message`字段中
* `@timestamp`随着message中的日期而改变了.
* `@timestamp`虽然改变了,但是和设置的时间不一样.

`date`插件默认的是将时间转化为 `Zulu/UTC`标准时间,也就是说,上面的时间我设置的是`20171120`,指的是当前时区的时间的`20171120 00:00:00`
但是,到了logstash中,被转为标准时间,也就是当前时区的时间`-8`小时,为`20171119 16:00:00`.

# 选项
`date`插件特有的选项如下:

* `local`
    * `string`类型
    * 没有默认值
    
    用于指定本地方言,比如设置为`en`,`en-US`等.主要用于解析非数字的月,和天,比如`Monday`,`May`等.如果是时间日期都是数字的话,不用关心这个值.
    
* `match`
    * `array`类型
    * 默认为`[]`
    
    用于将指定的字段按照指定的格式解析.比如:
    ``` 
    match => ["createtime", "yyyyMMdd","yyyy-MM-dd"]
    ```
    第一个值为字段名,其余值为解析的格式,如果有多个可能的格式,可以设置多个.
    
* `tag_on_failure`
    * array类型
    * 默认为`["_dateparsefailure"]`
    
    添加一个值到`tags`字段中,如果日期解析失败.
    
* `target`
    * string类型
    * 默认为`@timestamp`
    
    用于指定转化后的日期保存的字段名
    
* `timezone`
    * string类型
    * 没有默认值
    
    用于为要被解析的时间指定一个时区,值为时区的`canonical ID`,可以在[这里](http://joda-time.sourceforge.net/timezones.html)看到可以使用的值.
  
    一般不用设置,因为会根据当前系统的时区获取这个值.   
    
    这里设置的时区并不是logstash最终储存的时间的时区,logstash最终储存的时间为UTC标准时间.
    
    比如这里设置时间为`20171120`:
    * 如果时区为`Asia/Shanghai`那么转化后的时间为`2017-11-19T16:00:00.000Z`;
    * 如果时区为`Europe/Vienna`那么转化后的时间为`2017-11-19T23:00:00.000Z`;
    
# 总结

提供一个解析通用的日期配置:
``` 
filter{
    date{
        match => ["fieldName", "yyyyMMdd","yyyy-MM-dd"]
        target => "fieldName1"
        timezone => "Asia/Shanghai"
    }
}
```
