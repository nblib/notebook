# 概述
# 安装
## 环境准备
* linux
* python2.x
## 有网安装
### easy_install
```
 easy_install supervisor
```
### pip
```
pip install supervisor
```
## 无网安装
1. 下载依赖包
* setuptools (latest) from http://pypi.python.org/pypi/setuptools.
* meld3 (latest) from http://www.plope.com/software/meld3/.
* supervisor from https://pypi.python.org/packages/31/7e/788fc6566211e77c395ea272058eb71299c65cc5e55b6214d479c6c2ec9a/supervisor-3.3.3.tar.gz
* 下载
2. 复制到目标主机中
3. 解压每一个包,进入目录执行`python setup.py install` ,最后运行`supervisor`的.
# 测试
运行命令`echo_supervisord_conf`,会在terminal打印出一个配置文件sample.
