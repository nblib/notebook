# CDH配置LZO压缩
#### 前提
* 确保机器本身安装了LZO
    * 如果没有安装LZO,可以用过命令安装`yum install lzo lzo-devel `
#### 添加parcel
打开Cloudera Manager 管理页面.点击右上角 parcel,点击配置,添加如下url:
```
https://archive.cloudera.com/gplextras6/6.0.0/parcels/
```
添加后,分配,激活到各个机器上,然后就可以在需要使用到的地方配置使用.