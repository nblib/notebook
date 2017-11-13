#介绍
lor是基于`open resty`的web框架,具有一下特点:
* 高性能  
  运行在OpenResty上,可充分使用Nginx/OpenResty的库和插件
* 易开发  
  使用Lua作为主要编码语言,开发效率可媲美PHP等传统Web开发语言
* 低门槛  
采用简单易用的Sinatra风格路由,并提供丰富的中间件功能
#安装
```$xslt
git clone https://github.com/sumory/lor
cd lor
make install
```
默认`lor`的运行时lua文件会被安装到`/usr/local/lor`下， 命令行工具`lord`被安装在`/usr/local/bin`下。
如果希望自定义安装目录， 可参考如下命令自定义路径：
```$xslt
make install LOR_HOME=/path/to/lor LORD_BIN=/path/to/lord
```
执行默认安装后, `lor`的命令行工具`lord`就被安装在了`/usr/local/bin`下, 通过`which lord`查看:
```$xslt
which lord
```
#测试
查看帮助命令:
```$xslt
$ lord -h
lor ${version}, a Lua web framework based on OpenResty.

Usage: lord COMMAND [OPTIONS]

Commands:
 new [name]             Create a new application
 start                  Starts the server
 stop                   Stops the server
 restart                Restart the server
 version                Show version of lor
 help                   Show help tips
```
新建一个测试项目,执行lord new lor_demo，则会生成一个名为lor_demo的示例项目，然后执行：
```$xslt
cd lor_demo
lord start
```
之后访问http://localhost:8888/， 即可。显示欢迎界面,表示测试成功.
#注意问题
在以`root`身份运行的时候,可能会出现`404 Not found`问题,原因是权限不够,只需要在配置文件`conf/dev-nginx.conf`中的根位置添加如下:
```
user root;
```
即可