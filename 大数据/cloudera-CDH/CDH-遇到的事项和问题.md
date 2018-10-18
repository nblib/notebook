#### 运行命令行时出现提示permission denied时
出现权限不足,是因为需要特定的用户权限,可以:
```
sudo -u spark spark-shell
```
以上是使用spark时,如果使用hive的话对应为hive用户
#### 装好后主机状态不好
```
# 6.0
rm -f /var/lib/cloudera-scm-agent/cm_guid 
# 5.X
rm -f /opt/cloudera-manager/cm-5.11.1/lib/cloudera-scm-agent/cm_guid
```
#### hdfs 命令不能使用
缺少gateways,点击hdfs->添加角色->选择gateway

#### Exception in thread "main" java.lang.RuntimeException: core-site.xml not found

通过CDH配置了,在机器上命令行运行时,仍然需要hadoop的环境变量:
```
# 编辑/etc/profile添加
export HADOOP_CONF_DIR=/etc/hadoop/conf

# 保存后
source /etc/profile
```

#### spark 运行提示 Failing this attempt.Diagnostics: [2018-09-28 16:56:04.983]File file:/root/.sparkStaging/application_1538124779081_0001/__spark_conf__.zip does not exist

本机上,未配置hadoop环境变量

#### spark 提示 连接不上 8032 

```
##只能在 yarn的	ResourceManager 上运行spark命令行(后来测试并非如此)
```

#### spark-shell 运行后卡住不动了
可以开启日志级别为`INFO`:
```
# 编辑
vi /etc/spark/conf/log4j.properties 
# 修改级别
shell.log.level=INFO
```
#### spark-shell或hdfs命令往hdfs中写文件permission dened 
不能使用root用户,可以切换到`su hdfs`再运行
#### 启动hdfs或hdfs的datanode失败
可能是`/dfs/`文件夹下有旧的文件,`rm -rf /dfs/*`删除旧文件重新启动

#### sqoop 命令行运行提示:`ERROR: Invalid HADOOP_MAPRED_HOME`

指定的hadoop的环境变量:`HADOOP_CONF_DIR`有问题,并非`HADOOP_MAPRED_HOME`配置问题.
解决方法:
```
# 编辑/etc/profile
export HADOOP_CONF_DIR=/etc/hadoop/conf
# source
source /etc/profile
```