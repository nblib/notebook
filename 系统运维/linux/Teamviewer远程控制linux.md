# 安装
* 官网下载`Teamviewer`
* 下载安装依赖: https://centos.pkgs.org/7/epel-x86_64/qt5-qtwebkit-5.6.2-1.el7.x86_64.rpm.html
* 使用命令安装
``` 
yum install ....rpm
```
* 测试
``` 
# 查看id
teamviewer --info
# 如果没有id,提示重启,需要设置密码
teamviewer --passwd
# 然后再查看
```