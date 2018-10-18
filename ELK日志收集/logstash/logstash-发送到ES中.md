
logstash发送到es
===

# 发送到es
logstash 发送数据到elasticsearch
## demo
```
input{
	stdin{}
}
output{
	elasticsearch {
		index => "logmsg_%{+YYYY_MM_dd}"
		hosts => "127.0.0.1"
	}
	# stdout{codec=> rubydebug}
}
```
* 从控制台输入
* 输出到es中,index指定输出到哪个index中,如果index不存在将自动创建
* index名称中可以用表达式,比如上面格式为每天创建一个index

默认logstash将使用自带的文档类型创建索引.有时候,需要自定义文档类型,可以在es中自定义一个
mapping,指定index_patterns,比如:
```
# 新建logmsg模板
PUT _template/template_apilog
{
  "index_patterns": ["logmsg*"],
  "settings": {
    "number_of_shards": 2
  },
  "mappings": {
    "doc": {
      "properties": {
        "accesstime": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss"
        },
        "url":{
          "type": "text"
        },
        "qstr":{
          "type": "text"
        },
        "exetime":{
          "type": "float"
        }
        "reqhost":{
          "type": "keyword"
        }
      }
    }
  }
}
```
* 指定index_patterns,当新建的index符合这个格式,就会默认使用这个模板
* setting可以自定义设置,比如上面设置根据这个模板建立的索引,创建2个主分片
* doc为文档名称
* properties中为字段定义

当logstash中新建的索引符合这个模板时,会使用这个模板创建索引.如果logstash中的字段
不存在于模板中时,会自动添加新的字段.

## 发送es常用设置参数
###### action
发送到es的行为,可选值为:
* index: 保存一个文档到索引中
* delete: 根据id删除一个文档
* create: 保存文档到索引中,如果这个id的文档已经存在,则失败
* update: 更新文档根据id
* 可以这个为一个表达式,这样根据字段值动态决定行为,比如: `%{foo}`,这样会将foo的值作为action的值

###### hosts
指定es的地址.比如:
* "127.0.0.1"
* ["127.0.0.1:9200","127.0.0.2:9200"]
* ["http://127.0.0.1"]
* ["https://127.0.0.1:9200"]
* ["https://127.0.0.1:9200/mypath"]

###### index
指定索引的名称模板,默认值为"logstash-%{+YYYY.MM.dd}",即按天创建索引