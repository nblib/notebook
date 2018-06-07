# 记录日志

## 编辑haproxy配置
```
global
        log 127.0.0.1 local0
defaults
        log global
```
## 开启syslog监听
编辑`/etc/rsyslog.conf`
```
# 取消注释
$ModLoad imudp
$UDPServerRun 514
#添加日志文件位置
local0.*       /data/logs/haproxy.log

```
编辑`/etc/sysconfig/syslog`
```
SYSLOGD_OPTIONS="-r -m 0"
```
* -r:打开接受外来日志消息的功能,其监控514 UDP端口;
-x:关闭自动解析对方日志服务器的FQDN信息,这能避免DNS不完整所带来的麻烦;
* -m:修改syslog的内部mark消息写入间隔时间(0为关闭),例如240为每隔240分钟写入一次"--MARK--"信息;
* -h:默认情况下,syslog不会发送从远端接受过来的消息到其他主机,而使用该选项,则把该开关打开,所有
接受到的信息都可根据syslog.conf中定义的@主机转发过去
## 创建日志文件
> touch /data/logs/haproxy.log
## 重启
```
systemctl restart rsyslog.service
/data/haproxy/sbin/haproxy -f /data/haproxy/etc/haproxy.cfg -st `cat /data/haproxy/haproxy.pid` 
```
