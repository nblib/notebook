#概述
在`openresty`中对通过`gzip`压缩的请求体的处理,通过`lua-zlib`模块.
#准备
* linux
* zlip
* cmake
#安装
##### zlib的安装
查看是否安装
``` 
rpm -qa | grep zlib
```
没有安装的话
``` 
yum -y install zlib
```
##### cmake安装
下载,解压cmake包
``` 
wget https://cmake.org/files/v3.10/cmake-3.10.0-rc4-Linux-x86_64.tar.gz
tar zxf cmake-3.10.0-rc4-Linux-x86_64.tar.gz
```
配置为环境变量`/path/to/cmake/bin`
##### lua-zlib安装
下载,编译,编译
``` 
git clone https://github.com/brimworks/lua-zlib.git
cmake -DLUA_INCLUDE_DIR=/usr/local/openresty/luajit/include/luajit-2.1
make
```
编译完后,会在文件夹中发现`zlib.so`文件,把文件复制到`/path/to/openresty/lualib`文件夹下.
#测试
在nginx配置文件中添加:
``` 
location /testgzip{
	content_by_lua{
		local zlib = require "zlib"
		local encoding = ngx.req.get_headers()["Content-Encoding"]
		ngx.req.read_body()
		if encoding == "gzip"
		then
		    local body = ngx.req.get_body_data()
		    if body
		    then
		        local stream = zlib.inflate()
		        local r = stream(body)
		        ngx.say("unzipData:")
		        ngx.print(r)
		    end
		else
		    ngx.say("not press by gzip")
		end
	}
}
```
`nginx`运行起来,然后使用http请求发送压缩的请求,查看返回结果,如果和压缩前的内容一样,就成功了.
