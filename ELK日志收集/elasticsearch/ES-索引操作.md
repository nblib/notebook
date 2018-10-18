es的索引操作
===

# 创建索引
Create Index API用于在Elasticsearch中手动创建索引。
 Elasticsearch中的所有文档都存储在索引中。
最基本的命令如下：
```
PUT twitter
```
这个命令创建了一个默认的名为`twitter`的索引.
> 索引名称限制
>> 索引命名的内容有几个限制。 完整的限制列表如下：
> * 小写字母
> * 不能包含`\, /, *, ?, ", <, >, |, ` ` (space character), ,, #`
> * 7.0版本前可以包含`:`,以后的版本不再支持
> * 不能以`-,_,+`开头
> * 不能为`.`或`..`
> * 不能超过255个字节(多字节编码,也不能超过)

## 设置索引
创建的每个索引都可以具有与其关联的特定设置，在请求体中定义：
```
PUT twitter
{
    "settings" : {
        "index" : {
            "number_of_shards" : 3, 
            "number_of_replicas" : 2 
        }
    }
}
```
* `number_of_shards`默认为5(7.0版本后为1)
* `number_of_replicas`默认为1,也就是每个主分片都有一个副本.如上默认主分片为
5个,那么每个有一个副本,总共有五个副本

## 创建索引的响应结果
默认情况下，索引创建仅在每个分片的主副本已启动或请求超时时才返回对客户端
的响应。响应结果如下:
```
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "test"
}
``` 
`acknowledged`表明是否在集群中成功创建了索引，而`shards_acknowledged`
表明在超时之前是否为索引中的每个分片启动了必需数量的分片副本。值得注意的是`acknowledged`或`shads_acknowledged`为`false`时，索引依然创建成功。
这些值仅表示操作是否在超时之前完成。 有可能超时了,但是索引创建成功了,此时
`acknowledged`或`shards_acknowledged`为`false`.


我们也可以通过改变默认的索引设置,只等待指定数量的分片处于激活状态即可返回:
```
PUT test
{
    "settings": {
        "index.write.wait_for_active_shards": "2"
    }
}
```
* 设置为2,那么当创建索引时,当有两个分片处于激活状态即返回true

# 删除索引
删除索引通过如下:
```

DELETE /twitter
```
* 也可以通过通配符删除:比如`DELETE /_all`或者`DELETE /twi*`,这样会批量删除所有或多个索引

使用通配符删除索引有很大的风险,可以通过设置:
```
action.destructive_requires_name=true
```
这样删除索引必须使用完整的名称来删除,避免误删