# logstash-JDBC的使用

## 使用配置
```
input {
  jdbc {
    statement => "SELECT id, name FROM t_book_catalog WHERE id > :sql_last_value"
    use_column_value => true # 是否使用tracking_column的值做为 属性 sql_last_value 的值,默认保存在`$HOME/.logstash_jdbc_last_run`
    tracking_column => "id" # 追踪的字段名
    tracking_column_type => "numeric" #追踪的字段的类型,可选: numeric timestamp
    # ... other configuration bits
    jdbc_driver_library => "/Users/hewe/Soft/logstash-1/mysql-connector-java-5.1.46.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://localhost:3306/hewe1?useSSL=false"
    jdbc_user => "root"
    jdbc_password => "hewe0824"
    schedule => "* * * * *"
  }
}
output {
	stdout{codec=> rubydebug}
}
```
## 示例

### 下次查询根据上次查询的结果进行查询
#### 使用场景
* 我们的数据库中数据为ID自增,那么我们这次要查询的数据要排除掉上一次的数据,因为不想同样的数据差两次.
* 按时间查询.上次查询到的数据最后一条时间字段为`2018-10-17 15:00:00`,下次查询我们希望从`2018-10-17 15:00:00`开始查询新的数据.

#### 配置
```
input {
  jdbc {
    statement => "SELECT id, name FROM t_book_catalog WHERE id > :sql_last_value"
    use_column_value => true # 是否使用tracking_column的值做为 属性 sql_last_value 的值,默认保存在`$HOME/.logstash_jdbc_last_run`
    tracking_column => "id" # 追踪的字段名
    tracking_column_type => "numeric" #追踪的字段的类型,可选: numeric timestamp
    # ... other configuration bits
    jdbc_driver_library => "/Users/hewe/Soft/logstash-1/mysql-connector-java-5.1.46.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://localhost:3306/hewe1?useSSL=false"
    jdbc_user => "root"
    jdbc_password => "hewe0824"
    schedule => "* * * * *"
  }
}
output {
	stdout{codec=> rubydebug}
}
```
* statement 为查询语句,上次查询的最后一条结果要保存的字段保存到`:sql_last_value`中
* schedule 设置为没分钟执行一次

当前数据库中有一条数据:`id: 1,name: aaa`,第一次查询会查找到id为1 的数据,然后`logstash`将提取查找到的数据的`tracking_column`字段的值(id:1),然后将数据1保存到`last_run_metadata_path`中,当下次查询时,`sql_last_value`属性的值为上次保存的值1,这样执行我们以上的sql,就会为`SELECT id, name FROM t_book_catalog WHERE id > 1`.

#### 输出结果
```
[2018-10-18T15:52:01,499][INFO ][logstash.inputs.jdbc     ] (0.012007s) SELECT id, name FROM t_book_catalog WHERE id > 0
{
            "id" => 1,
    "@timestamp" => 2018-10-18T07:52:01.551Z,
      "@version" => "1",
          "name" => "test"
}
{
            "id" => 2,
    "@timestamp" => 2018-10-18T07:52:01.562Z,
      "@version" => "1",
          "name" => "aaaa"
}
[2018-10-18T15:53:00,086][INFO ][logstash.inputs.jdbc     ] (0.000891s) SELECT id, name FROM t_book_catalog WHERE id > 2
{
            "id" => 3,
    "@timestamp" => 2018-10-18T07:53:00.098Z,
      "@version" => "1",
          "name" => "bbbb"
}

```
每一分钟执行一次,第一次查到两条数据,最后id为2,然后第二次查询的时候的查询条件为`>2`

### 分页查询
#### 使用场景
* 一次查询的数据量太大,使用limit查询一次查询几条,分为多次查询
  
#### 配置
```
input {
  jdbc {
    statement => "SELECT id, name FROM t_book_catalog"
    jdbc_paging_enabled => true
    jdbc_page_size => 2
    clean_run => true

    # ... other configuration bits
    jdbc_driver_library => "/Users/hewe/Soft/logstash-1/mysql-connector-java-5.1.46.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://localhost:3306/hewe1?useSSL=false"
    jdbc_user => "root"
    jdbc_password => "hewe0824"
    schedule => "* * * * *"
  }
}
output {
	stdout{codec=> rubydebug}
}
```
* jdbc_paging_enabled: 开启分页查询
* jdbc_page_size: 每次返回结果数量

#### 运行结果
```
[2018-10-18T16:26:01,413][INFO ][logstash.inputs.jdbc     ] (0.005882s) SELECT version()
[2018-10-18T16:26:01,450][INFO ][logstash.inputs.jdbc     ] (0.001684s) SELECT version()
[2018-10-18T16:26:01,589][INFO ][logstash.inputs.jdbc     ] (0.008246s) SELECT count(*) AS `count` FROM (SELECT id, name FROM t_book_catalog) AS `t1` LIMIT 1
[2018-10-18T16:26:01,619][INFO ][logstash.inputs.jdbc     ] (0.000870s) SELECT * FROM (SELECT id, name FROM t_book_catalog) AS `t1` LIMIT 2 OFFSET 0
[2018-10-18T16:26:01,654][INFO ][logstash.inputs.jdbc     ] (0.000501s) SELECT * FROM (SELECT id, name FROM t_book_catalog) AS `t1` LIMIT 2 OFFSET 2
[2018-10-18T16:26:01,661][INFO ][logstash.inputs.jdbc     ] (0.000430s) SELECT * FROM (SELECT id, name FROM t_book_catalog) AS `t1` LIMIT 2 OFFSET 4
[2018-10-18T16:26:01,667][INFO ][logstash.inputs.jdbc     ] (0.000459s) SELECT * FROM (SELECT id, name FROM t_book_catalog) AS `t1` LIMIT 2 OFFSET 6
[2018-10-18T16:26:01,674][INFO ][logstash.inputs.jdbc     ] (0.000385s) SELECT * FROM (SELECT id, name FROM t_book_catalog) AS `t1` LIMIT 2 OFFSET 8
{
    "@timestamp" => 2018-10-18T08:26:01.656Z,
          "name" => "cccc",
            "id" => 4,
      "@version" => "1"
}
{
    "@timestamp" => 2018-10-18T08:26:01.669Z,
          "name" => "ggggg",
            "id" => 8,
      "@version" => "1"
}
{
    "@timestamp" => 2018-10-18T08:26:01.636Z,
          "name" => "test",
            "id" => 1,
      "@version" => "1"
}
{
    "@timestamp" => 2018-10-18T08:26:01.662Z,
          "name" => "dddd",
            "id" => 5,
      "@version" => "1"
}
{
    "@timestamp" => 2018-10-18T08:26:01.676Z,
          "name" => "hhhhh",
            "id" => 9,
      "@version" => "1"
}
{
    "@timestamp" => 2018-10-18T08:26:01.676Z,
          "name" => "iiii",
            "id" => 10,
      "@version" => "1"
}
{
    "@timestamp" => 2018-10-18T08:26:01.647Z,
          "name" => "aaaa",
            "id" => 2,
      "@version" => "1"
}
{
    "@timestamp" => 2018-10-18T08:26:01.662Z,
          "name" => "eeee",
            "id" => 6,
      "@version" => "1"
}
{
    "@timestamp" => 2018-10-18T08:26:01.656Z,
          "name" => "bbbb",
            "id" => 3,
      "@version" => "1"
}
{
    "@timestamp" => 2018-10-18T08:26:01.669Z,
          "name" => "ffffff",
            "id" => 7,
      "@version" => "1"
}
```

## 常用配置项介绍
- tracking_column
    > 保存上次运行状态时要保存的列名

- schedule
    > 设置查询数据的周期,最小单位为分钟,也就是最快为没分钟执行一次.配置cron字符串为 "* * * * *",如果没有设置,默认为启动后执行一次,后续不再执行.
## 调试
* 刚启动的时候,如果driver没有找到或数据库连接配置错误,不会报错,只有第一次获取数据的时候才会报错

## 错误
1. dup Fixnum错误
    1. 错误场景:
        ```
        首先开启`use_column_value`后,保存上次查询结果到`last_run_metadata_path`中,然后关闭,当启用分页查询时,报错:`dup Fixnum`

        ```
    2. 解决办法:
        1. 删除上次缓存,也就是删除`last_run_metadata_path`中内容
        2. 添加`clean_run => true`
