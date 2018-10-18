es重要的配置项
===

# es相关的配置
es相关的配置主要位于`config/elasticsearche.yml`文件中

## 数据和日志保存路径(path.data,path.logs)
```
path:
  logs: /var/log/elasticsearch
  data: /var/data/elasticsearch
```
* 通过`path: logs`配置日志输出目录
* 通过`path: data`配置数据保存路径

可以设置多个数据保存路径,多个路径将都被用来保存数据(属于同一个shard的file将被保存到相同的路径下)
```
path:
  data:
    - /mnt/elasticsearch_1
    - /mnt/elasticsearch_2
    - /mnt/elasticsearch_3
```

## 集群名称(cluster.name)
设置集群的名称通过`cluster.name`,一个节点只能设置一个名称,也只能加入到一个集群中.
集群的默认名称为`elasticsearch`,建议修改.不同集群,建议使用不同的名称,不然容易导致节点被加入到
错误的集群中.
```
cluster.name: logging-prod
```

## 节点名称(node.name)
默认es生成一个UUID,使用前七个字符作为节点id,节点id将被保存,当节点重启后不会改变.
所以,默认的node.name也不会改变.建议设置一个有意义的node.name:
```
node.name: prod-data-2
```
节点名称,也可以设置为HOSTNAME:
```
node.name: ${HOSTNAME}
```

## 节点host(network.host)
默认es绑定环回地址-比如127.0.0.1和[::1],这样节点只能运行为单机模式,其他节点无法访问.为了部署为集群模式,或者可以远程访问
需要修改这个配置项绑定到一个非环回接口,比如绑定到对外ip.比如:
```
network.host: 192.168.1.23
```

## 发现设置(Discovery settings)
es使用一个常用的发现实现`Zen discovery`来实现节点之间构成集群和主节点选举.主要有两个配置.

### discovery.zen.ping.unicast.hosts
在没有任何配置下,es将绑定可用的环回接口,并且扫描本机的9300到9305端口,来发现其他部署在本机的节点.当需要构建一个集群,每个节点
部署在不同的服务器上时,需要指定集群上的几个可以访问的节点,来让当前的节点加入到集群中.例子如下:
```
discovery.zen.ping.unicast.hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 
   - seeds.mydomain.com
```
    * 可以指定目标ip和端口
    * 可以指定目标的ip,这样默认使用ransport.profiles.default.port
    * 同时也可以使用hostname

### discovery.zen.minimum_master_nodes
设置一个节点最少发现`discovery.zen.minimum_master_nodes`个节点时才能选举形成一个新的集群.这样设置为了避免出现`split brain`的情况
> split brain
>> 脑裂.在es中,假设我们有三个节点:`1`,`2`,`3`,当出现网络断开的情况下.`3`失去了与其他两个节点的联系.此时如果`minimum_master_nodes`设置为1,
那么节点`3`只发现了自己这1个节点,就会开始选举生成新的集群.导致变成了两个集群:`1,2`和`3`.除非重启,并且3能连接到`1,2`,才会重新合为一个集群
.这样导致数据不一致.而此时如果配置项为2,那么`3`想要选举,必须同时发现两个节点,也就是除了它自己还要在发现一个节点,才能开始选举.

为了避免`split brain`,这个配置项应该设置为:
```
(master_eligible_nodes / 2) + 1
```
比如有3个节点,那么这个值设置为`2`,如果4个节点,那么设置为3.这样当出现故障时,保证还是一个集群,不会变为多个集群.

# JVM配置
默认情况下,es告诉JVM使用1GB的最小和最大堆内存.当用于生产环境时,需要适当调整,保证足够的堆内存可以使用.es通过指定
`Xms`(最小堆内存)和`Xmx`(最大堆内存)来分配堆内存.而堆内存的大小则依赖于服务器可用的内存.最佳实践规则:
* 设置`Xms`和`Xmx`的大小一样
* 越大的堆内存对于es来说可以储存越多的缓存.但是,太大的堆内存意味着JVM的垃圾回收将变得耗时
* `Xmx`的大小不要超过服务器可用内存的50%.保证有足够的空间供系统内核文件缓存.
* `Xmx`不要超过JVM的 `oops`的大小,64为机子上大约为32GB.也就是`Xmx`不要超过32GB.最好低于26GB

> oops(jVM 对象指针压缩)
>> 参考: http://kavy.iteye.com/blog/1998684

## GC logging
默认es开启GC logs,配置在`jvm.option`中.默认路径和es的日志输出路径相同.默认配置:单个Gc日志文件64M,最大占用2gb磁盘大小

# 系统相关配置
默认情况下es为开发模式.系统相关的设置默认可以启动.只不过会在日志中警告.如果配置了一个network配置项比如:`network.host`会自动开启
生产环境模式,将对系统配置进行检查,如果不符合要求,会报错.

以下配置在生产环境中需要注意:
* 禁用内存交换
* 增加文件描述符
* 保证足够的虚拟内存
* 保证足够的可用线程数量
