# CDH6中使用oozie
添加oozie的服务
## 开启web管理界面
oozie依赖`ext-2.2`,下载[ext-2.2](http://tiny.cloudera.com/oozie-ext-2.2)

解压到运行oozie的机器上:
```
# 解压
cd /var/lib/oozie
unzip ext-2.2.zip
ls
##显示内容:ext-2.2  ext-2.2.zip  mysql-connector-java.jar  tomcat-deployment
```

## 设置时区为东八区
* 首先打开HUE配置->搜索: `time_zone` -> 设置为`Asia/Shanghai`
* 重启HUE
* 打开oozie的web控制台->设置->时间设置为`CST(Asia/Shanghai)`
* 打开oozie配置-> 搜索`oozie-site.xml`-> 添加值 :
    ```
    名称: oozie.processing.timezone
    值: GMT+0800
    ```
    或者以xml形式添加:
    ```
    <property>
    <name>oozie.processing.timezone</name>
    <value>GMT+0800</value>
    <description>设置时区为东八区</description>
    </property>
    ```
* 重启oozie

## 提交workflow发生错误
提交workflow发生错误:
```
org.apache.hadoop.yarn.exceptions.InvalidResourceRequestException: Invalid resource request, requested resource type=[memory-mb] < 0 or greater than maximum allowed allocation. Requested resource=<memory:2048, vCores:1>, maximum allowed allocation=<memory:1024, vCores:2>, please note that maximum allowed allocation is calculated by scheduler based on maximum resource of registered NodeManagers, which might be less than configured maximum allocation=<memory:2048, vCores:2>
```
* 可在yarn的resourcemanager查看输出日志
意思是提交的任务需要2G内存,1个CPU,而yarn只允许最大2GB,而每个节点只允许1GB,所以要调节一下两项:

* yarn.scheduler.maximum-allocation-mb 修改为4GB
* yarn.nodemanager.resource.memory-mb 修改为2GB