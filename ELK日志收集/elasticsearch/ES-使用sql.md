es中使用sql
===
# 简介
es的sql目标是提供轻量级的SQL功能.ES的sql是一个X-PACK组件，允许类似于SQL的查询实时执行。
无论使用REST接口、命令行还是JDBC，任何客户端都可以使用SQL在Elasticsearch内本地搜索和聚合数据。
可以把Elasticsearch SQL看作一个翻译器，它理解SQL和Elasticsearch，并且通过利用Elasticsearch能力
使实时读取和处理数据变得容易。
> ES SQL 优点
> * 原生支持
> * 不需要额外的支持
> * 轻量和高效
  
# 简单使用

可以使用kibana的开发者工具进行测试.要开始使用 SQL，创建一个带有一些数据的索引来进行实验：
```
PUT /library/book/_bulk?refresh
{"index":{"_id": "Leviathan Wakes"}}
{"name": "Leviathan Wakes", "author": "James S.A. Corey", "release_date": "2011-06-02", "page_count": 561}
{"index":{"_id": "Hyperion"}}
{"name": "Hyperion", "author": "Dan Simmons", "release_date": "1989-05-26", "page_count": 482}
{"index":{"_id": "Dune"}}
{"name": "Dune", "author": "Frank Herbert", "release_date": "1965-06-01", "page_count": 604}
```
现在可以开始执行SQL使用`SQL REST API`:
```
POST /_xpack/sql?format=txt
{
    "query": "SELECT * FROM library WHERE release_date < '2000-01-01'"
}
```
将看到返回结果:
```
    author     |     name      |  page_count   | release_date
---------------+---------------+---------------+------------------------
Dan Simmons    |Hyperion       |482            |1989-05-26T00:00:00.000Z
Frank Herbert  |Dune           |604            |1965-06-01T00:00:00.000Z
```
# SQL和ES的数据结构映射关系
SQL | ES | 描述
---|---|---
column | field | es的field映射为一个列,注意的是,es中一个字段可能含有多个值,这种情况下,为了迎合sql,包含多值的字段将拒绝返回.
row | document | sql的row定义严格,而es的document则相对比较灵活
table | index | 
schema | implicit | 在RDBMS中，Schema主要是表的命名空间，通常用作安全边界。ES并没有为它提供一个等价的概念。然而，当启用安全性时，Elasticsearch自动应用安全强制，以便角色只看到它被允许看到的数据.
catalog or database | cluster instance | 一个数据库对应es的一个实例

# SQL REST API
SQL REST API接受JSON文档中的SQL，执行它并返回结果.例如：
```
POST /_xpack/sql?format=txt
{
    "query": "SELECT * FROM library ORDER BY page_count DESC LIMIT 5"
}
```
将会返回:
```
     author     |     name      |  page_count   |      release_date      
----------------+---------------+---------------+------------------------
Frank Herbert   |Dune           |604            |1965-06-01T00:00:00.000Z
James S.A. Corey|Leviathan Wakes|561            |2011-06-02T00:00:00.000Z
Dan Simmons     |Hyperion       |482            |1989-05-26T00:00:00.000Z
```
* formart=txt,指定返回的格式,txt返回适合人看的格式.可选的格式有:
    * json 或 application/json
    * yaml 或 application/yaml
    * smile 或 application/smile
    * cbor 或 application/cbor
    * txt 或 text/plain
    * csv 或 text/csv
    * tsv 或 text/tab-separated-values

## 分页查询
要使用分页查询,请求指定的返回格式需要为`json`或`yaml`,其余格式`CSV,tsv`不支持分页.
1. 请求第一页数据
    ```
    POST /_xpack/sql?format=json
    {
        "query": "SELECT * FROM library ORDER BY page_count DESC",
        "fetch_size": 2
    }
    ```
    将返回:
    ```
    {
      "columns": [
        {
          "name": "author",
          "type": "text"
        },
        {
          "name": "name",
          "type": "text"
        },
        {
          "name": "page_count",
          "type": "long"
        },
        {
          "name": "release_date",
          "type": "date"
        }
      ],
      "rows": [
        [
          "Frank Herbert",
          "Dune",
          604,
          "1965-06-01T00:00:00.000Z"
        ],
        [
          "James S.A. Corey",
          "Leviathan Wakes",
          561,
          "2011-06-02T00:00:00.000Z"
        ]
      ],
      "cursor": "o9TwAgFz5AFEbkYxWlhKNVZHaGxia1psZEdOb0JRQUFBQUFBQUJYLUZrMUJTMWt0YkhkV1VtMWhjMXBrY1ZkcmNIcERlRkVBQUFBQUFBQVhNUlpST0VRelUzZEVWbEpYUzBKZlNrbFhSWE40U2pCQkFBQUFBQUFBQ1VRV05FUnhibGxwWWtWUmRtRTRRM2MwVEVWa2JFUkRRUUFBQUFBQUFCWF9GazFCUzFrdGJIZFdVbTFoYzFwa2NWZHJjSHBEZUZFQUFBQUFBQUFKUlJZMFJIRnVXV2xpUlZGMllUaERkelJNUldSc1JFTkL/////DwQBZgZhdXRob3IBBHRleHQAAAFmBG5hbWUBBHRleHQAAAFmCnBhZ2VfY291bnQBBGxvbmcBAAFmDHJlbGVhc2VfZGF0ZQEEZGF0ZQEA"
    }
    ```
    * columns: 返回结果中包含的列和列类型.只有第一页返回,其他页不返回这个信息
    * rows: 请求返回的结果
    * cursor: 下一页的请求参数,具体使用下看
    
2. 请求下一页的数据
    当第一次请求时,会返回一个`cursor`,这个cursor用于请求剩余的数据,每次返回的数量和第一次请求的数量一样,直到最后一页
    ```
    POST /_xpack/sql?format=json
    {
        "cursor": "o9TwAgFz5AFEbkYxWlhKNVZHaGxia1psZEdOb0JRQUFBQUFBQUJYLUZrMUJTMWt0YkhkV1VtMWhjMXBrY1ZkcmNIcERlRkVBQUFBQUFBQVhNUlpST0VRelUzZEVWbEpYUzBKZlNrbFhSWE40U2pCQkFBQUFBQUFBQ1VRV05FUnhibGxwWWtWUmRtRTRRM2MwVEVWa2JFUkRRUUFBQUFBQUFCWF9GazFCUzFrdGJIZFdVbTFoYzFwa2NWZHJjSHBEZUZFQUFBQUFBQUFKUlJZMFJIRnVXV2xpUlZGMllUaERkelJNUldSc1JFTkL/////DwQBZgZhdXRob3IBBHRleHQAAAFmBG5hbWUBBHRleHQAAAFmCnBhZ2VfY291bnQBBGxvbmcBAAFmDHJlbGVhc2VfZGF0ZQEEZGF0ZQEA"
    }
    ```
    * 将第一页的返回的cursor作为请求体上传.
    
    返回结果:
    ```
    {
      "rows": [
        [
          "Dan Simmons",
          "Hyperion",
          482,
          "1989-05-26T00:00:00.000Z"
        ]
      ],
      "cursor": "o9TwAgFz5AFEbkYxWlhKNVZHaGxia1psZEdOb0JRQUFBQUFBQUJZZEZrMUJTMWt0YkhkV1VtMWhjMXBrY1ZkcmNIcERlRkVBQUFBQUFBQVhQUlpST0VRelUzZEVWbEpYUzBKZlNrbFhSWE40U2pCQkFBQUFBQUFBQ1VnV05FUnhibGxwWWtWUmRtRTRRM2MwVEVWa2JFUkRRUUFBQUFBQUFCWWVGazFCUzFrdGJIZFdVbTFoYzFwa2NWZHJjSHBEZUZFQUFBQUFBQUFKU1JZMFJIRnVXV2xpUlZGMllUaERkelJNUldSc1JFTkL/////DwQBZgZhdXRob3IBBHRleHQAAAFmBG5hbWUBBHRleHQAAAFmCnBhZ2VfY291bnQBBGxvbmcBAAFmDHJlbGVhc2VfZGF0ZQEEZGF0ZQEA"
    }
    ```
    * 可以看到请求第二页只返回数据,和cursor

3. 最后一页
    首先介绍下`cursor`,这个字段用于表示一次查询请求,当我们第一次查询时便为这次请求生成一个`cursor`,
    以后获取剩余的分页数据时,使用`cursor`标识要获取的是那个请求的剩余数据(同一个查询请求的剩余数据使用的cursor其实值相同).当没有剩余数据时,使用`cursor`请求会返回一个
    空的数组.不再包含`cursor`字段.此时,如果再次请求,将会返回一个错误结果.
    比如,当没有剩余结果时,再次请求,返回结果如下:
    ```
    {
      "rows": []
    }
    ```
    * 此时返回一个空的数组.没有`cursor`,
    
    如果再次请求的话,返回结果如下:
    ```
    {
      "error": {
        "root_cause": [
          {...}
        ]
        ...
      }
      ...
      "status": 404
    }
    ```
4. 分页请求的介绍
    分页请求当第一次请求到来时,es会记录当前请求的状态.然后为这个状态生成唯一标识`cursor`.
    后续请求想要获取剩余的数据,则使用`cursor`请求.es会维持着请求的状态直到返回了一个空数组.
    此时,es才会清除这次请求的状态,同时使cursor失效
    
  
es提供的这个功能,更使用与手机端我们常看到的`加载更多`的功能,而不是PC端的分页功能.因为这个功能每次
请求只能返回下一页,而不能获取上一页.

5. 提前结束查询
    ```
    POST /_xpack/sql/close
    {
        "cursor": "sDXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAEWYUpOYklQMHhRUEtld3RsNnFtYU1hQQ==:BAFmBGRhdGUBZgVsaWtlcwFzB21lc3NhZ2UBZgR1c2Vy9f///w8="
    }
    ```
    当不想查询后续的数据时,要及时清理查询状态,释放es的资源.
    
    
## SQL 转为es查询
es的sql支持也就是内部将sql转为es的查询语法.通过这个接口可以快速的查看转换后的es查询:
```
POST /_xpack/sql/translate
{
    "query": "SELECT * FROM library ORDER BY page_count DESC",
    "fetch_size": 10
}
```
将sql转化后的结果为:
```
{
    "size" : 10,
    "docvalue_fields" : [
        {
            "field": "page_count",
            "format": "use_field_mapping"
        },
        {
            "field": "release_date",
            "format": "epoch_millis"
        }
    ],
    "_source": {
        "includes": [
            "author",
            "name"
        ],
        "excludes": []
    },
    "sort" : [
        {
            "page_count" : {
                "order" : "desc"
            }
        }
    ]
}
```