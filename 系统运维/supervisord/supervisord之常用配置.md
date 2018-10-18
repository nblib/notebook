# 概述
基本配置内容和目录结构
# 目录结构
```
-supervisor_conf
----supervisor.conf
----process_conf
--------test.conf
--------test2.conf
```
* `supervisorconf`是总配置文件.引入单独的配置文件
* `process_conf`放置单独的配置文件
# 配置
```
[unix_http_server]
file=/tmp/supervisor/supervisor.sock   ; the path to the socket file

[inet_http_server]         ; inet (TCP) server disabled by default
port=127.0.0.1:9001        ; ip_address:port specifier, *:port for all iface
username=user              ; default is no username (open server)
password=123               ; default is no password (open server)

[supervisord]
logfile=%(here)s/logs/supervisord.log ; main log file; default $CWD/supervisord.log
logfile_maxbytes=50MB        ; max main logfile bytes b4 rotation; default 50MB
logfile_backups=10           ; # of main logfile backups; 0 means none, default 10
loglevel=info                ; log level; default info; others: debug,warn,trace
pidfile=%(here)s/logs/supervisord.pid ; supervisord pidfile; default supervisord.pid
nodaemon=false               ; start in foreground if true; default false
minfds=1024                  ; min. avail startup file descriptors; default 1024
minprocs=200                 ; min. avail process descriptors;default 200

[include]
files =process_conf/*.conf

[eventlistener:crashmail]
command=crashmail -o faildMSG -m 272489473@qq.com
events=PROCESS_STATE_EXITED
```
* 日志文件记录在`supervisor_conf`的`logs`目录中
* 使用`include`导入`process_conf`中的单独配置文件
