es中的一些常用请求
===
```
# 批量创建样例数据
PUT /demo/cate/_bulk?refresh
{"index":{"_id":"Leviathan Wakes"}}
{"name":"Leviathan Wakes","author":"James S.A. Corey","release_date":"2011-06-02","page_count":561}
{"index":{"_id":"Hyperion"}}
{"name":"Hyperion","author":"Dan Simmons","release_date":"1989-05-26","page_count":482}
{"index":{"_id":"Dune"}}
{"name":"Dune","author":"Frank Herbert","release_date":"1965-06-01","page_count":604}

# 根据条件查询样例数据
POST /_xpack/sql?format=txt
{
    "query": "SELECT * FROM library WHERE release_date < '2000-01-01'"
}

# 限制每次获取的数据数量
POST /_xpack/sql?format=json
{
    "query": "SELECT * FROM libr* ORDER BY page_count DESC",
    "fetch_size": 1
}
# 获取第一次请求的后续数据
POST /_xpack/sql?format=json
{
    "cursor": "o9TwAgFz5AFEbkYxWlhKNVZHaGxia1psZEdOb0JRQUFBQUFBQUIyV0ZsRTRSRE5UZDBSV1VsZExRbDlLU1ZkRmMzaEtNRUVBQUFBQUFBQU1neFkwUkhGdVdXbGlSVkYyWVRoRGR6Uk1SV1JzUkVOQkFBQUFBQUFBSFpjV1VUaEVNMU4zUkZaU1YwdENYMHBKVjBWemVFb3dRUUFBQUFBQUFBeUVGalJFY1c1WmFXSkZVWFpoT0VOM05FeEZaR3hFUTBFQUFBQUFBQUFiQ3haTlFVdFpMV3gzVmxKdFlYTmFaSEZYYTNCNlEzaFL/////DwQBZgZhdXRob3IBBHRleHQAAAFmBG5hbWUBBHRleHQAAAFmCnBhZ2VfY291bnQBBGxvbmcBAAFmDHJlbGVhc2VfZGF0ZQEEZGF0ZQEA"
}

# 简单的sql
POST /_xpack/sql?format=txt
{
  "query": "select 1 + 1 as sum"
}
# 通过sql限制获取条数
POST /_xpack/sql/translate
{
    "query": "SELECT author,page_count FROM library limit 2"
}
# 获取表结构
POST /_xpack/sql?format=txt
{
  "query": "DESCRIBE library"
}
# 查询表中的列信息
POST /_xpack/sql?format=txt
{
  "query": "show columns from library"
}


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

#往新建的index中添加数据
PUT /wechat/user/1
{
  "name":"hewe",
  "age":11,
  "email": "tingshage@163.com"
}

# 获取刚添加的数据
GET /wechat/user/_search

# 再添加一条
PUT /wechat/user/3
{
  "name":"hewe",
  "email": "tingshage@163.com",
  "addr": "beijing"
}

# 查看创建好的结构
POST /_xpack/sql?format=txt
{
  "query": "DESCRIBE wechat"
}
```