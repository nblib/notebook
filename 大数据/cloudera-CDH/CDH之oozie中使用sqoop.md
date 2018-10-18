## 问题
##### 找不到数据库driver
oozie中使用sqoop并非CDH自带的sqoop.
```
上传 Driver 到 hdfs 的目录 /user/oozie/share/lib/sqoop 下，如果是Cloudera and HDP，则是/user/oozie/share/lib/lib_${timestamp}/sqoop下 
```
然后重启oozie service
