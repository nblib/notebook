# 概述
# 使用
## 配置
### hdfs配置
* 开启 `webhdfs`,在`Namenode`和`Datanode`中编辑`Hadoop/etc/hadoop/hdfs-site.xml`:
    ```
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
    ```
> 如果namenode处于safemode,关闭:`hadoop dfsadmin -safemode leave`
### logstash配置
编辑配置文件:
```
input{
        stdin{}
}
output{
         webhdfs {
    host => "192.168.233.135"                 # (required)
    port => 50070                       # (optional, default: 50070)
    path => "/user/logstash/dt=%{+YYYY-MM-dd}/logstash-%{+HH}.log"  # (required)
    user => "root"                       # (required)
    compression => "snappy"
  }
        stdout{codec=>rubydebug}
}
```
* user 必须是hdfs允许的用户
* compression设置压缩格式,有两种:`snappy`和`gzip`
## 测试
*运行`logstash`
* 输入`nihao`
* **等待一会**,在hdfs中查看目录下的文件`/user/logstash/dt=%{+YYYY-MM-dd}/logstash-%{+HH}.log`
# 问题
1. AccessControllException 访问拒绝

查看logstash中的user是否被hadoop允许
2. logstash报错`serverError`

查看hdfs是否正常允许,是否处于安全模式,处于安全模式,则关闭安全模式
3. 输出的日志内容为 @timestamp host message 的格式

添加`codec`:
```
output{
         webhdfs {
    host => "l22-232-13"                 # (required)
    port => 14000                       # (optional, default: 50070)
    path => "/user/logstash1/dt=%{+YYYY-MM-dd}/logstash-%{+HH}.log"  # (required)
    user => "logstash"                       # (required)
codec => plain { format => "%{message}"}
#    compression => "snappy"
  }
#       stdout{codec=> rubydebug}
}
```
# 用到的命令
* `./bin/logstash -f config/loghdfs.conf`
> 根据配置文件启动`logstash
* `nohup ./bin/logstash -f config/loghdfs.conf --config.reload.automatic > loginfo.out & echo $! > pid.out`
> 后台启动,并记录pid
* `hadoop fs -ls /user/logstash`
> 查看目录
* `hadoop fs -tail /user/logstash/dt=2018-01-16/logstash-02.log`
> 查看文件末尾
* `hadoop dfsadmin -safemode leave`
> 关闭安全模式
* `curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=CREATE`
> 创建文件请求,会返回用于接收文件的Datanode的位置,返回的location内容:`http://emp2:50075/webhdfs/v1/logstash?op=CREATE&namenoderpcaddress=boss:9000&overwrite=false`
* `curl -i -X PUT -T <LOCAL_FILE> "http://emp2:50075/webhdfs/v1/logstash?op=CREATE&namenoderpcaddress=boss:9000&overwrite=false"`
> 创建文件请求获取到的reponse中的location作为这里的url,发送文件