# 自动重新加载配置
为了可以自动检测配置文件的变动和自动重新加载配置文件,需要在启动的时候使用以下命令:
```
./bin/lagstash -f configfile.conf --config.reload.automatic
```
默认,检测配置文件的间隔时间是`3秒`,可以通过以下命令改变
```
--config.reload.interval <second>
```
如果已经运行了没有提供自动重启的logstash,可以发送一个挂起命令给logstash重新加载配置文件:
```
kill -1 <pid>
```
# 配置文件自动重载工作原理
* 检测到配置文件变化
* 通过停止所有输入停止当前pipline
* 用新的配置创建一个新的管道
* 检查配置文件语法是否正确
* 检查所有的输入和输出是否可以初始化
* 检查成功使用新的pipeline替换当前的pipeline,
* 检查失败,使用旧的继续工作.

在重载过程中,`jvm`没有重启.