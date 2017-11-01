elk-安装配置
===

### `es`安装
##### 下载安装包
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.3.tar.gz
tar -xzf elasticsearch-5.6.3.tar.gz
cd elasticsearch-5.6.3/ 
```
##### 在命令行运行
```
#前台模式
./bin/elasticsearch
#后台模式,pid为保存pid的文件
./bin/elasticsearch -d -p pid
#通过kill命令,和pid停止
```
##### 在命令行配置
```
./bin/elasticsearch -d -Ecluster.name=my_cluster -Enode.name=node_1
```
##### 测试
```
curl 'http://localhost:9200/'
```
### `logstash`的安装
##### 环境
* linux
* jdk
##### 下载安装包
[logstash下载地址](https://www.elastic.co/downloads/logstash)
##### 解压
##### 测试
```
# 在控制台输入,在控制台输出
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```
### `kibana`安装
##### 下载
[下载地址](https://artifacts.elastic.co/downloads/kibana/kibana-5.6.3-linux-x86_64.tar.gz)
##### 配置
编辑`config/kibana.yml`文件,设置`elasticsearch.url`为es的地址
##### 运行
`./bin/kibana`

##### 测试
访问`http://localhost:5601`