#概述
nginx+lua开发,安装和配置
#nginx安装
### 环境
1. 操作系统: linux
### 准备
1. 安装需要依赖linx系统库:  `readline-devel pcre-devel openssl-devel gcc`
1. `OpenResty`源码包,官网下载地址,这里已附带,点击下载[openresty-1.11.2.4.tar.gz](../安装包/openresty-1.11.2.4.tar.gz)
### 安装
####### 下载tar包并解压
```$xslt
tar zxf openresty-1.11.2.4.tar.gz
```
###### 进入解压后的目录,配置
可以通过`--prefix=/path/to/location`参数指定安装位置,默认为`/usr/local/openresty`,
> 开启`http_stub_status_module`,Orange的监控插件需要统计http的某些状态数据
```$xslt
cd openresty-1.11.2.4
./configure --with-http_stub_status_module
```
###### 编译和安装
```$xslt
make
make install
```
###### 找到安装目录,默认为`/usr/local/openresty`,进入后可以看到有`bin`文件夹,`nginx`文件夹等
###### 配置环境变量
添加环境变量,`orange`启动的时候会在环境变量中寻找这两个变量并调用.
```$xslt
vi /etc/profile

export PATH=$PATH:/usr/local/openresty/bin:/usr/local/openresty/nginx/sbin
```