为Docker容器指定自定义网段的固定IP地址
===
Docker容器运行的时候默认会自动分配一个默认网桥所在网段的IP地址。但很多时候我们可能需要让容器运行在预先指定的静态IP地址上，因为早期的版本不支持静态IP，因此网上大部分方法都是借助pipework等去实现，然而在最新的版本中，Docker已经内嵌支持在启动时指定静态IP了。

---
1. 安装最新版的Docker 
2. 创建自定义网络  
```
docker network create --subnet=166.66.0.0/16 demonet
```
3. 在你自定义的网段选取任意IP地址作为你要启动的container的静态IP地址
```
docker run -d -i -t --net demonet --ip 166.66.0.11 centos /bin/bash
```
> 如果发现ip不是指定的ip,可能是命令输错了,或ip输错了.