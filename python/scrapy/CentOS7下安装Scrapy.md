CentOS7下安装Scrapy
===

### 更新yum(可选)
```
yum -y update
```
### 安装gcc及扩展包
```
yum install gcc libffi-devel python-devel openssl-devel
```
### 安装开发工具包
```
yum groupinstall -y development
```
### 安装libxslt-devel支持lxml

```
yum install libxslt-devel
```
### 安装pip

```
yum -y install python-pip
```
### 安装Scrapy

```
pip install scrapy
```
