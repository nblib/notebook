# 概述
监控并发送邮件.
### 参考
http://blog.csdn.net/baidu_zhongce/article/details/49151385
# 环境准备
安装`superlance`插件
```
easy_install superlance
```
或
```
pip install superlance
```
# 配置
添加一个测试进程和一个事件监听.
```
[program:top]
command=top -b
process_name=%(program_name)s
numprocs=1
directory=/tmp
umask=022
priority=999
autostart=false
autorestart=false
startsecs=10
startretries=3
exitcodes=0,2
stopsignal=TERM
stopwaitsecs=10
redirect_stderr=true

[eventlistener:crashmail]
command=crashmail -p top -m adf@example.com
events=PROCESS_STATE_EXITED
```
当 top 进程以外停止时,会发送邮件.

# 测试
启动后,查看top进程的pid
```
supervisorctl  pid top
```
然后使用kill命令杀死进程.这样会收到邮件