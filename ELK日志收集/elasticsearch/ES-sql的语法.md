es中使用sql的语法
===
# SELECT
es主要支持的sql语法为`select`
## 语法
```
SELECT select_expr [, ...]
[ FROM table_name ]
[ WHERE condition ]
[ GROUP BY grouping_element [, ...] ]
[ HAVING condition]
[ ORDER BY expression [ ASC | DESC ] [, ...] ]
[ LIMIT [ count ] ]
```
* 当前只支持从一个表中查询,当然 table_name可以使用`pattern`,比如`log*`,通过这样实现多表
* 指定了`where`条件,那么所有不符合条件的行将被过滤掉

以上列出的用法和sql中的使用方法一样.

1. 普通计算查询
    ```
    SELECT 1 + 1 AS result
    ```
2. 从表中查询
    ```
    # 查询指定字段
    SELECT emp_no FROM emp LIMIT 1;
    # 查询所有字段
    SELECT * FROM emp
    ```
3. 从pattern的表中查询
    ```
    # 所有符合"e*p"的正则表达式的表都会被查询,比如"e1p","e2p"
    SELECT emp_no FROM "e*p" LIMIT 1;
    ```
4. 聚合
    ```
    SELECT MIN(salary) AS min, MAX(salary) AS max, MAX(salary) - MIN(salary) AS diff FROM emp GROUP BY languages HAVING diff - max % min > 0 AND AVG(salary) > 30000;
    ```
5. 只取指定数量的记录
    ```
    SELECT first_name, last_name, emp_no FROM emp LIMIT 1;
    ```
    * limit的参数只能是一个数字或者`*`
    * 如果需要分段去,可以使用es自带的分页查询

* 更多查询例子: https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-syntax-select.html#sql-syntax-limit
* 可用的聚合,操作和方法:  https://www.elastic.co/guide/en/elasticsearch/reference/current/_comparison_operators.html

## 查看表信息
1. 显示当前有哪些表
    ```
    SHOW TABLES [ LIKE? pattern? ]?
    # 比如
    SHOW TABLES LIKE 'em_';
    SHOW TABLES;
    ```
1. 查看表结构
    ```
    POST /_xpack/sql?format=txt
    {
      "query": "DESCRIBE library"
    }
    ```
    返回结果
    ```
        column     |     type      
    ---------------+---------------
    author         |VARCHAR        
    author.keyword |VARCHAR        
    name           |VARCHAR        
    name.keyword   |VARCHAR        
    page_count     |BIGINT         
    release_date   |TIMESTAMP      
    ```
2. 查看指定的字段和字段类型
    ```
    POST /_xpack/sql?format=txt
    {
      "query": "show columns from library"
    }
    ```
## 查看可用的函数
```
SHOW FUNCTIONS [ LIKE? pattern? ]?
```
显示所有支持的sql的函数,可以通过`like`筛选过滤