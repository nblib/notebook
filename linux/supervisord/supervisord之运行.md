# 概述
运行`supervisord`,安装好后,默认系统变量中会包含运行`supervisord`的命令.
# 运行
## 添加一个`program`
打开配置文件,在最后添加如下内容:
```
[program:foo]
command=/bin/cat
```
## 根据配置文件运行
`supervisord`默认启动会在如下位置寻找配置文件:
1. $CWD/supervisord.conf
1. $CWD/etc/supervisord.conf
1. /etc/supervisord.conf
1. /etc/supervisor/supervisord.conf (since Supervisor 3.3.0)
1. ../etc/supervisord.conf (Relative to the executable)
1. ../supervisord.conf (Relative to the executable)

可以手动指定自己的配置文件.

启动命令:
```
supervisord # 使用默认配置文件启动
supervisord -c /path/to/your/config  # 指定配置文件
```
> 可以添加`-n`启动参数启动,用于前台启动,方便观察输出.默认是后台启动.

# 运行`supervisorctl`
* `supervisorctl`可以控制/远程控制`supervisord`的行为.
* 默认`supervisorctl`是一次性命令,比如`supervisorctl stop all`,也可以通过加参数`-i`从而进行交互式控制.
* 在非交互模式运行,会返回0表示运行成功,非零表示出现错误.
* 如果`supervisord`开启了安全认证,那么,开启交互模式需要登陆

## 常用参数
> -c
* 通过配置文件运行
> -i
* 交互模式运行
> -s
* 指定要连接的远程supervisord服务
> -u, -p
* 指定用户名和密码

使用方法,比如:
```
#交互模式登陆
supervisorctl -i -u root -p 123
```
## 常用控制命令
控制命令可以在交互模式/非交互模式运行,非交互模式下,比如:
```
supervisorctl stop all
```
交互模式下,通过参数进入交互模式
```
supervisorctl -i
```
在交互模式下可以输入控制命令控制supervisord行为

常用命令:
> add <name> [...]
* 从配置文件中添加一个进程/组,比如配置文件编辑后重新加载,新添加的进程和remove的进程不会启动,需要通过这个命令启动.
> remove <name> [...]
* 移除一个进程
> update
* 从新加载配置文件,这样会运行配置文件中的所有进程,包括remove的.
> reread
* 重新加载配置文件,但不会重新运行进程.
> clear <name> <name>
* 清除进程的日志文件
> fg <process>
* 把一个进程调到前台运行,ctrl+c退出前台模式
> start/stop/status  <name> <name>
* 开启/关闭/状态  进程
> tail [-f] <name> [stdout|stderr] (default stdout)
* 查看进程的标准输出和错误输出.
> signal <signal name> <name>
* 发送信号到supervisord,通过信号控制,比如`supervisorctl signal SIGQUIT


