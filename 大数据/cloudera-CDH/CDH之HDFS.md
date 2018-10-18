# CDH中的HDFS

## Balancer
HDFS提供的平衡数据分布的工具.一个运行中的集群HDFS中的数据分布可能会不均匀,比如新添加一个节点,上面没有数据,为了是数据分布平衡,通过运行Balancer可以分析块放置并平衡DataNode上的数据。 平衡器移动块直到认为集群是平衡的.比如一个datanode磁盘使用率时10%,而整个集群的磁盘使用率是40%.此时这个datanode的磁盘使用率和整个集群的使用率相差30%.此时数据分布在这个datanode上相对于整个集群来说是不均衡的.此时需要重新平衡下,往这个datanode上放点数据提高磁盘使用率.

至于什么条件下执行重新平衡操作,可以通过设置阈值限制,比如设置阈值为20%,当datanode的磁盘使用率和整个集群的使用率相差超过20%,则执行重平衡.比如:集群40%使用率,阈值20%,那么当datanode的使用率超出范围: 40-20% 到 40+20%,也就是datanode的使用率低于20%或高于60%,则执行重新平衡操作.
### 使用方法
进入HDFS 服务-> 点击HDFS名称右侧的操作按钮->重新平衡

### 配置重平衡阈值
点击HDFS的配置-> 点击Balancer范围->修改`重新平衡阈值`

### 配置重平衡操作的移动速度
重平衡操作就是将数据块从数据量多的datanode移动到相对少的datanode.移动过程势必占用线程,IO等资源.可以通过设置用于执行balancer操作的线程数来调节balancer执行速度.

配置用于balancer的线程数,需要同时配置balancer和datanode:
* 配置datanode
    * 进入HDFS 服务
    * 点击 配置
    * 搜索: hdfs-site.xml
    * 编辑 hdfs-site.xml 的 DataNode 高级配置代码段(以 XML 格式查看),添加:
        ```
        <property>
          <name>dfs.datanode.balance.max.concurrent.moves</name>
          <value>50</value>
        </property>
        ```
* 配置balancer
    * 进入HDFS 服务
    * 点击 配置
    * 搜索: hdfs-site.xml
    * 编辑 hdfs-site.xml 的 Balancer 高级配置代码段(以 XML 格式查看),添加:
        ```
        <property>
          <name>dfs.datanode.balance.max.concurrent.moves</name>
          <value>50</value>
        </property>
        ```

## NFS gateway配置和使用
nfs gateway 将一个机器节点设置为nfs服务的网关,其余机器可以通过访问这台机器配置挂载.

用途: 通过HDFS的NFS服务,我们可以将hdfs的文件系统通过挂载的方式挂载为本地文件系统.比如我们可以将hdfs文件系统挂载到本地机器的/hdfs目录,这样,我们可以像访问本地目录一样访问/hdfs目录,从而实现访问hdfs的文件系统.
#### 配置
* 添加一个NFS gateway角色
* 进入HDFS 配置-> NFS Gateway -> 编辑配置项
* 可以设置允许的主机和权限,比如:
    * 192.168.0.1 rw
    
    这样只允许指定IP访问,具有读写权限

#### 测试
```
# 输入以下命令测试
[root@centos218 ~]# rpcinfo -p $nfs_server_ip
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  52161  status
    100024    1   tcp  59606  status
    100005    1   udp   4242  mountd
    100005    2   udp   4242  mountd
    100005    3   udp   4242  mountd
    100005    1   tcp   4242  mountd
    100005    2   tcp   4242  mountd
    100005    3   tcp   4242  mountd
    100003    3   tcp   2049  nfs

# 查看挂载的目录和权限
showmount -e $nfs_server_ip
```
#### 使用
可以使用另一台机器(前提是被上面的权限配置允许),挂载hdfs文件到本地文件系统.
比如:
```
mount -t nfs -o vers=3,proto=tcp,nolock,noacl,sync <IP>:/  <本地目录>
```
* 使用nfs类型
* 使用tcp协议
* <IP>为NFS gateway的主机
* <本地目录>表示将hdfs文件系统挂载到本地那个目录
* **本地目录一定要是空的,不然会导致本地文件丢失**

成功运行以上命令后,可以向访问本地目录一样访问hdfs的目录以及文件,比如:
```
ls <挂载的目录>
cat <挂载的目录>/test.txt
....
```
#### 卸载目录
如果我们不需要挂载hdfs到本地目录可以输入如下命令:
```
umount <挂载的目录>
```

## 命令行上传文件
使用命令行上传文件,比如:
```
 hdfs dfs -put ./mysql-connector-java-5.1.47.jar /user/oozie/share/lib/lib_20180929142133/sqoop
```
当遇到`permission denied时,是因为文件夹需要指定用户的权限,比如上传到/user/oozie文件夹中,需要使用oozie用户,可以使用sudo命令:
```
sudo -u oozie hdfs dfs -put ./mysql-connector-java-5.1.47.jar /user/oozie/share/lib/lib_20180929142133/sqoop
```
* -u 指定用户为oozie

