# 使用工具
* JAVA_HOME/jdk/bin/jvisualvm
# 开启远程监控
编辑tomcat启动文件
```
# cd /root/apache-tomcat-8.0.17/bin
# vi catalina.sh   #找到JAVA_OPTS，在下面添加，添加的位置并没要要求
#JAVA_OPTS="$JAVA_OPTS -Dorg.apache.catalina.security.SecurityListener.UMASK=`umask`"
JAVA_OPTS="$JAVA_OPTS-Dcom.sun.management.jmxremote 
        -Dcom.sun.management.jmxremote.port=9999 
        -Dcom.sun.management.jmxremote.authenticate=false 
        -Dcom.sun.management.jmxremote.ssl=false 
        -Djava.rmi.server.hostname=192.168.1.156" # 为本机ip
# ./startup.sh 
# netstat -antp |grep 9999    #查看端口是否监听
```
# 远程监控
打开jvisualvm：
* 双击远程，随便输入名称，点击确定
* 右键新建的远程，添加JMX连接
* 输入ip：port

添加后，连接成功后，双击会显示监控界面。