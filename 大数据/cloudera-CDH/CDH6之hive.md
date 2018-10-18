# CDH6中hive的使用
Hive可以在分布式存储中读取，写入和管理大型数据集。 使用与SQL非常相似的Hive查询语言（HiveQL），查询将被转换为一系列通过MapReduce或Apache Spark在Hadoop集群上执行的作业。

在CDH中使用hive:
* YARN提供的统一资源管理
* Cloudera Manager提供的简化部署和管理
* 共享安全性和治理，以满足Apache Sentry和Cloudera Navigator提供的合规性要求

## hive版本
* 使用hive2.1版本

## 使用spark作为引擎
默认使用的事MAPREDUCE,可以选择使用spark作为运行引擎.
1. 配置cloudera manager:
    ```
    * 打开hive->配置
    * 搜索 Spark On YARN Service
    * 选择spark
    * 保存配置,重启hive,重新部署客户端配置
    ```
2. 配置hive客户端
    ```
    * 打开hive->配置
    * 搜索hive.execution.engine
    * 选择spark
    * 保存配置,重启hive,重新部署客户端配置
    ```

配置完成重启后,进入hive的客户端机器,运行hive命令(如果已经运行,退出再进入,避免不生效),然后执行hive操作,此时进入yarn管理UI界面,可以看到Application Type 变成了SPARK