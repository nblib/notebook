# influxDB HTTP API使用

## 数据库操作
```
# 创建数据库
curl -i -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE mydb"
# 查看数据库
curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=mydb" --data-urlencode "q=show databases"
```
## 写数据
```
# 插入一条数据(数据最后字段为手动指定时间戳)
curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'cpu_load_short,host=server01,region=us-west value=0.64 1434055562000000000'

# 批量插入数据,每条数据使用新行区分
curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'cpu_load_short,host=server02 value=0.67
cpu_load_short,host=server02,region=us-west value=0.55 1422568543702900257
cpu_load_short,direction=in,host=server01,region=us-west value=2.0 1422568543702900257'
```
写数据时,也可以上传文件中的数据:
* 创建文件`cpu_data.txt`:
    ```
    cpu_load_short,host=server02 value=0.67
    cpu_load_short,host=server02,region=us-west value=0.55 1422568543702900257
    cpu_load_short,direction=in,host=server01,region=us-west value=2.0 1422568543702900257
    ```
* 上传数据:
    ```
    curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary @cpu_data.txt
    ```

#### 响应状态汇总
当响应状态码为:
* 2xx: 比如:`204`,写入成功
* 4xx: influxDB不能理解命令
* 5xx: 系统超负载

## 读数据
* URL: `/query`
* 参数: 
    * db: 指定数据库
    * q: 指定查询语句
    * pretty<可选>: true,友好的展示,生产环境不要使用,会增加网络消耗
* 请求方法:
    * GET: 参数拼接到url后
    * POST: 参数可以为URL后的参数,或者为`application/x-www-form-urlencoded`

比如:
```
# 查询数据
curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=mydb" --data-urlencode "q=SELECT \"value\" FROM \"cpu_load_short\" WHERE \"region\"='us-west'"
```

返回结果:
```
{
    "results": [
        {
            "statement_id": 0,
            "series": [
                {
                    "name": "cpu_load_short",
                    "columns": [
                        "time",
                        "value"
                    ],
                    "values": [
                        [
                            "2015-01-29T21:55:43.702900257Z",
                            2
                        ],
                        [
                            "2015-01-29T21:55:43.702900257Z",
                            0.55
                        ],
                        [
                            "2015-06-11T20:46:02Z",
                            0.64
                        ]
                    ]
                }
            ]
        }
    ]
}

```
* 返回结果包含在results中,如果有错误,会返回一个`error`以及错误信息
* statement_id为查询语句的id,当多个查询语句时,便于区分是哪个语句的结果

#### 多查询语句
如果使用多查询语句,那么每个语句使用`;`分割,比如:
```
{
    "results": [
        {
            "statement_id": 0,
            "series": [
                {
                    "name": "cpu_load_short",
                    "columns": [
                        "time",
                        "value"
                    ],
                    "values": [
                        [
                            "2015-01-29T21:55:43.702900257Z",
                            2
                        ],
                        [
                            "2015-01-29T21:55:43.702900257Z",
                            0.55
                        ],
                        [
                            "2015-06-11T20:46:02Z",
                            0.64
                        ]
                    ]
                }
            ]
        },
        {
            "statement_id": 1,
            "series": [
                {
                    "name": "cpu_load_short",
                    "columns": [
                        "time",
                        "count"
                    ],
                    "values": [
                        [
                            "1970-01-01T00:00:00Z",
                            3
                        ]
                    ]
                }
            ]
        }
    ]
}
```
#### 返回数据的时间格式
默认返回结果时间格式为`rfc3339`,比如:`2015-08-04T19:05:14.318570484Z`,可以通过参数`epoch=[h,m,s,ms,u,ns]`设置格式:
```
curl -G 'http://localhost:8086/query' --data-urlencode "db=mydb" --data-urlencode "epoch=s" --data-urlencode "q=SELECT \"value\" FROM \"cpu_load_short\" WHERE \"region\"='us-west'"
```