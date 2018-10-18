# cloudera manager6安装指南
## 必须环境
* JDK8(oracle)
* python2.7+(不适用3.0+)
* Perl
* python-psycopg2(可选,如果使用的是PostgreSQL)
* iproute package(manager Agent必须安装,RHEL6需要iproute-2.6,RHEL7需要3.10)
* RHEL系统6.8,6.9,7.2,7.3,7.4,7.5
* nproc配置大于65536
* mysql5.7以及MySQL-shared-compat or MySQL-shared package
* utf-8编码数据库

## 软件
* [cloudera-manager](https://archive.cloudera.com/cm6/6.0.0/redhat6/yum/RPMS/x86_64/): 包括agent,daemons,server,server-db
* [CDH](https://archive.cloudera.com/cdh6/6.0.0/parcels/):包括CDH.parcel,CDH.parcel.sha256,manifest.sjon

## 安装manager
manager统一管理集群的所有节点,也是UI界面所在的主机
### 开始
1. 上传`cloudera-manager`和`CDH`的文件到主机上
    ```
    # 创建目录 /opt/cloudera/parcel-repo
    mkdir -p /opt/cloudera/parcel-repo
    # 将CDH相关的文件放入/opt/cloudera/parcel-repo
    # 重命名`.sha256`文件为`.sha`
    ```
1. 安装`cloudera-manager`
    ```
    # 安装cloudera-manager包,包括daemons,server,server-db(先不安装agent)
    rpm -ivh  cloudera-manager-*
    ```
1. 配置数据库
    ```
    # 在mysql中创建一个数据库`scm`,用于manager存数据
    CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
    GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'scm@123';
    
    # 运行manager的自动配置数据库脚本,这样自动配置好了manager需要连接数据库的配置
    # mysql指定数据库类型,-h指定数据库的地址,--scm-host指向当前manager的host,scm为数据库名,scm为连接数据库的用户名
    /opt/cloudera/cm/schema/scm_prepare_database.sh mysql -h mysql.example.com --scm-host manager.example.com scm scm
    ```
1. 启动`manager`
    ```
    # 以上都配置好后,可以启动manager了
    service cloudera-scm-server start
    ```
    这样就启动了`manager`,然后通过http://<host>:7180访问UI界面了

## 安装节点(要被manager管理的,部署hadoop等的机器)
1. 上传文件  
    ```
    将`cloudera-manager`的agent和daemons两个文件上传
    ```
2. 安装
    ```
    # 安装daemons和agent,一个被cdh-manager管理的机器需要这两个服务
    rpm -ivh  cloudera-manager-*
    ```
1. 编辑配置文件
    ```
    # 编辑agent的配置文件,将server_host指向manager所在的主机,这样manager才能发现并管理这个节点
    vi  /etc/cloudera-scm-agent/config.ini
    # 修改server_host为  server_host=<manager的host>
    ```
1. 启动
    ```
    #启动agent
    service  cloudera-scm-agent  start
    ```
## 通过ui界面管理
第一进入ui界面(http://<host>:7180)会开始引导界面
1. 登录
    ```
    访问需要登录,默认用户名/密码: admin/admin
    ```
1. 欢迎界面
    ```
    登录后便进入了欢迎界面,也就是通用的接受协议,选择cloudera版本,然后就进入了下一步
    ```
1. 创建集群引导界面
    ```
    欢迎界面后,开始创建集群的引导界面.
    ```
