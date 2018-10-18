es中的Mapping(翻译)
===

# 简介
映射是定义文档及其包含的字段的存储和索引方式的过程。 例如，使用映射来定义：
* 应将哪些字符串字段视为全文字段。
* 哪些字段包含数字，日期或地理位置。
* 是否应将文档中所有字段的值索引到catch-all `_all`字段中。
* 日志类型的格式.
* 用于控制动态添加字段的映射的自定义规则。

同语言的数据类型相比，mapping还有一些其他的含义，mapping不仅告诉ES一个field中是什么类型的值， 它还告诉ES如何索引数据以及数据是否能被搜索到。
# Mapping Type
每个索引都有一种映射类型，用于确定文档的索引方式。一个映射类型有:
* Meta-fields
    元字段用于自定义文档的元数据关联的处理方式。 元字段的示例包括文档的`_index`，`_type`，_id和`_source`字段。
* Fields or properties
    映射类型包含与文档相关的字段或属性列表。
    
# Field datatypes
每个字段都有一个数据类型，可以是：
* 一个简单的类型，如`text`，`keyword`，`date`，`long`，`double`，`boolean`或`ip`。
* 支持JSON的分层特性的类型，例如`object`或`nested`。
* 或者像geo_point，geo_shape或completion这样的特殊类型。

为不同目的以不同方式索引相同字段通常很有用。 
例如，`string`类型字段可以被索引为用于全文搜索的文本字段，
并且可以被索引为用于排序或聚合的关键字字段。
 或者，可以使用标准分析器，英语分析器和法语分析器索引字符串字段。

# 简单使用
当新建一个index时,指定创建一个mapping:
```
#创建一个mapping,在创建index时,指定mapping
PUT wechat
{
  "mappings": {
    "user": {
      "properties": {
        "name": { "type": "text" },
        "age": {  "type": "integer" },
        "email": { "type": "keyword" }
      }
    }
  }
}
```
* 此时,创建了一个新的index,包含一个`user`类型
* `user`类型包含三个字段,分别为`name`,`age`,`email`

可以通过sql查看创建好的文档类型:
```
POST /_xpack/sql?format=txt
{
  "query": "DESCRIBE wechat"
}
# 返回结果
    column     |     type      
---------------+---------------
age            |INTEGER        
email          |VARCHAR        
name           |VARCHAR        
```
可以添加一条数据:
```
#往新建的index中添加数据
PUT /wechat/user/1
{
  "name":"hewe",
  "age":11,
  "email": "tingshage@163.com"
}
```
查看数据:
```
# 获取刚添加的数据
GET /wechat/user/_search
```

# mapping types 介绍
**在Elasticsearch 6.0.0或更高版本中创建的索引可能只包含单个映射类型。
 在具有多种映射类型的5.x中创建的索引将继续像以前一样在Elasticsearch 6.x中运行。
  映射类型将在Elasticsearch 7.0.0中完全删除。**
## 什么是mapping type(映射类型)
自Elasticsearch的第一个版本以来，每个文档都存储在单个索引中并分配了单个映射类型。
映射类型用于表示被索引的文档或实体的类型，例如，`twitter`索引可能具有`user`类型和`tweet`类型。
每种映射类型都可以有自己的字段，因此`user`类型可能具有`full_name`字段，`user_name`字段和
`email`字段，而`tweet`类型可以具有`content`字段，`tweeted_at`字段以及和`user`类型一样
有一个`user_name`字段。 每个文档都有一个包含类型名称的`_type`元字段，通过在URL中
指定类型名称，搜索可以限制为一种或多种类型：
```
GET twitter/user,tweet/_search
{
  "query": {
    "match": {
      "user_name": "kimchy"
    }
  }
}
```
`_type`字段与文档的`_id`结合以生成`_uid`字段，因此具有相同
`_id`的不同类型的文档可以存在于单个索引中。映射类型也用于在
文档之间建立`父子关系`，因此类型`question`的文档可以是类型为`answer`的文档的父文档。

## 为什么要移除mapping type
最初，我们谈到了一个“索引”类似于SQL数据库中的“数据库”，而“类型”等同于“表”。这是一个糟糕的比喻，导致错误的假设。 在SQL数据库中，表彼此独立。
 一个表中的列与另一个表中具有相同名称的列无关。 映射类型中的字段不是这种情况。
 
在Elasticsearch索引中，不同映射类型中具有相同名称的字段在内部由相同的Lucene字段支持。
换句话说，使用上面的示例，`user`类型中的`user_name`字段存储在与`tweet`类型中的`user_name`字段
完全相同的字段中，并且两个`user_nam`e字段在两种类型中必须具有相同的映射（定义）。

例如，当希望删除一个类型的日期字段和同一索引中另一个类型的布尔字段时，这可能会导致不好处理。

最重要的是，在同一索引中存储具有很少或没有共同字段的不同实体会导致稀疏数据并干扰Lucene
有效压缩文档的能力。

出于这些原因，决定从Elasticsearch中删除映射类型的概念。

## 替代 mapping type
### 为每个文档类型创建一个索引
第一种方法是每个文档类型都有一个索引。 您可以将`twitter`存储在`twitter`索引,将用户储存在`user`索引中，
而不是将`twitter`和`user`存储在单个`twitter`索引中。 指数完全相互独立，因此指数之间不存在字段类型冲突。

这样做有以下优点:
* 数据更可能的稠密聚集,更有利于luncene的压缩
* 在全文搜索中的统计评分更可能的准确,因为同一个索引中只有一个类型的实体

同时,每个索引可以很好的扩容,比如,`user`和`twtter`存在不同的索引中,当`twitter`数量多时,可以自由的给`twitter`
扩容,而不影响到`user`

### 自定义字段类型
当然，群集中可以存在多少个主分片是有限的，因此您可能不希望浪费整个分片只收集几千个文档。
 在这种情况下，您可以实现自己的自定义类型字段，该字段的工作方式与旧的`_type`类似。

比如,使用上面的`user`和`twitter`例子,自定义字段类型,使用方法如下:
```
PUT twitter
{
  "mappings": {
    "user": {
      "properties": {
        "name": { "type": "text" },
        "user_name": { "type": "keyword" },
        "email": { "type": "keyword" }
      }
    },
    "tweet": {
      "properties": {
        "content": { "type": "text" },
        "user_name": { "type": "keyword" },
        "tweeted_at": { "type": "date" }
      }
    }
  }
}

PUT twitter/user/kimchy
{
  "name": "Shay Banon",
  "user_name": "kimchy",
  "email": "shay@kimchy.com"
}

PUT twitter/tweet/1
{
  "user_name": "kimchy",
  "tweeted_at": "2017-10-24T09:00:00Z",
  "content": "Types are going away"
}

GET twitter/tweet/_search
{
  "query": {
    "match": {
      "user_name": "kimchy"
    }
  }
}
```