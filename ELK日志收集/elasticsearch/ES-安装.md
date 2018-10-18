ES的安装
===
# 环境
* centos
* es最新版(6.4)
* jdk1.8
# 下载es
* 下载方法1: https://www.elastic.co/downloads/elasticsearch
* 下载方法2:
    ```
    # 在命令行执行如下命令
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.0.zip
    ```

# 简单启动

## 前提
想要运行es,首先需要新建一个linux用户.不能使用root用户启动es

### 新建用户
```
# 新建用户
useradd es
```
### 为用户授权
解压 es压缩包后,将解压后的文件夹的所有者改为以上新建的用户
```
# 改变所有者
chown es ./elasticsearch -R
```
* es为新建的linux用户
* elasticsearch为es解压后的文件夹
* -R,递归授权,将这个文件夹中的所有文件的所有者改为新建的用户

## 修改配置
进入解压后的es文件夹中(使用新建的linux用户),编辑config文件夹中的`elasticsearch.yml`配置文件:
```
# 为了能远程访问,将此项设置为本机的ip地址
network.host: <IP地址>

# Centos6不支持SecComp，而ES5.2.0默认bootstrap.system_call_filter为true,在`elasticsearch.yml`在Memory下面添加:
bootstrap.system_call_filter: false
```
## 修改系统配置
```
# 切换为root用户,修改
vi /etc/security/limits.conf
# 增加(es为启动es的linux用户名)
es         soft    nproc     4096
*          soft    nofile   65536
```
修改虚拟内存
```
# 修改 /etc/sysctl.conf,添加
vm.max_map_count=262144
```

## 启动
进入es解压后的文件夹,启动es:
```
./bin/elasticsearch
```
* 此方法为前台启动

> 后台启动
>> ./bin/elasticsearch -d -p pid

## 测试
通过curl命令访问,或者浏览器访问:
```
http://<IP>:9200/
```
成功返回值:
```
{
  "name" : "Cp8oag6",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",
  "version" : {
    "number" : "6.4.0",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "f27399d",
    "build_date" : "2016-03-30T09:51:41.449Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "1.2.3",
    "minimum_index_compatibility_version" : "1.2.3"
  },
  "tagline" : "You Know, for Search"
}
```
# 遇到错误
如果遇到错误,可以先看下 [es使用遇到的错误](https://www.nblib.org/elasticsearch/es-starter-errors.html)
# 