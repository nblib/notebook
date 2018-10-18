docker redis 集群搭建
===
###### 环境准备
1. centos7 mini
2. 关闭 selinux
    ```
    vi /etc/selinux/config  //修改SELINUX=disabled
    reboot
    ```
3. yum install docker
4. systemctl enable docker
5. systemctl start docker
6. groupadd docker
7. usermod -a -G docker XXX
8. systemctl restart docker

---

1. 配置文件设置  
配置redis.conf文件
```
port  7000                                                
bind 本机ip                                       //默认ip为127.0.0.1 需要改为其他节点机器可访问的ip 否则创建集群时无法访问对应的端口，无法创建集群
daemonize    no                               //redis后台运行
pidfile  /var/run/redis_7000.pid          //pidfile文件对应7000,7001,7002
cluster-enabled  yes                           //开启集群  把注释#去掉
cluster-config-file  nodes_7000.conf   //集群的配置  配置文件首次启动自动生成 7000,7001,7002
cluster-node-timeout  15000                //请求超时  默认15秒，可自行设置
appendonly  yes                           //aof日志开启  有需要就开启，它会每次写操作都记录一条日志　
```
编写后,创建每个redis实例对应的文件夹,存放自己的配置文件,比如/home/ting/redis1,/home/ting/redis2,/home/ting/redis3文件夹,这些文件夹下放redis.conf文件
2. redis容器配置和启动  
    挂载配置文件和启动:  
     ```
     docker run -d -v /home/ting/redis4/redis.conf:/usr/local/etc/redis/redis.conf --net demoredis --ip 155.66.0.14 -p 7004:7000  --name redis4 redis redis-server /usr/local/etc/redis/redis.conf
     ```
3. 安装ruby  
    1. 下载ruby包  
    2. 安装需要依赖包  
    ```
    yum -y groupinstall "Development Tools"
    yum -y install gdbm-devel libdb4-devel libffi-devel libyaml libyaml-devel ncurses-devel openssl-devel readline-devel tcl-devel
    ```
    3. 编译和安装
    ```
     ./configure
     make
     make install
     ruby -v #查看版本确认安装好ruby
     gem list #查看是否有redis包
     gem install redis # 安装redis包
    ```
4. redis-trib.rb创建集群
```
//在外面的主机上下载一个redis包,获取redis/src下的redis-trib.rb
//运行命令
./redis-trib.rb create --replicas 1 155.66.0.11:7000 155.66.0.12:7000 155.66.0.13:7000 155.66.0.14:7000 155.66.0.15:7000 155.66.0.16:7000
```
5. 测试
```
//进入容器,连接redis
redis-cli -c -h 155.66.0.11 -p 7000
//查看集群信息
cluster info
```