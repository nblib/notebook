# influxDB命令行基本使用
通过命令行可以连接`influxDB`,类似于mysql的命令行工具

## 连接数据库
* 数据库安装在本机:
    ```
    influx -precision rfc3339
    ```
* 连接远程数据库
    ```
    influx -host sample.host -precision rfc3339
    ```

参数:`-precision`为设置显示时间格式,如果没有设置,返回的时间类型字段显示为时间戳,使用`rfc3339`返回的时间格式为:` 2018-10-11T06:37:53.439100393Z`

## 显示数据库/创建数据库
连接上数据库后可以查看数据库,创建数据库:
* 查看数据库
    ```
    # 显示有哪些数据库
    SHOW DATABASES
    # 创建数据库
    CREATE DATABASE mydb1
    ```
> _internal
>> `_internal`数据库为influxDB用来存内部运行指标的数据库

## 使用数据库
```
USE mydb1
```

## 增/查数据
数据库中一条记录包含如下:
* time: 一个时间戳
* measurement: 度量指标(类似于一般数据库中的表)
* field: 至少一个字段,是一个key-value类型,用于保存真正的值.不会为field创建索引
* tags: 为一条记录打一个标签,可以有0个或多个.tags会创建索引

一条记录的格式:
```
<measurement>[,<tag-key>=<tag-value>...] <field-key>=<field-value>[,<field2-key>=<field2-value>...] [unix-nano-timestamp]
```
比如:
```
 cpu,host=serverA,region=us_west value=0.64
```
* cpu相当于表名
* host和region为tag
* value为保存的filed

插入,查找,删除数据:
```
# 插入一条数据
INSERT cpu,host=serverA,region=us_west value=0.64
# 查看数据
SELECT "host", "region", "value" FROM "cpu"
# 往另一个表中插入数据
INSERT temperature,machine=unit42,type=assembly external=25,internal=37
# 查看所有内容
SELECT * FROM "temperature"
# 使用表名通配符,同时查看多个表中的多条记录
# SELECT * FROM /.*/ LIMIT 10
# 带有查询条件
SELECT * FROM "cpu_load_short" WHERE "value" > 0.9
# 删除数据
delete from "cpu" where host='serverA'
```
* 删除数据的条件不能是field,因为field没有索引.但是可以是tags
* 查询数据的条件可以是field