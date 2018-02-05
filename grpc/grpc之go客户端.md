# go版客户端
## 环境
* 如果可以翻墙,直接使用命令`go get google.golang.org/grpc`,会自动下载grpc和相关的依赖
* 如果不能翻墙,则到github上下载相应的源码,再复制到相应的目录中.比如:从` https://github.com/grpc/grpc-go`上clone源码,然后在`GOPATH/src/`下新建文件夹`google.golang.org/grpc`,然后将clone下的`grpc-go`文件夹中的内容复制到`grpc`中.需要的依赖包和相应的位置:
    * google.golang.org/grpc   对应的代码地址在： https://github.com/grpc/grpc-go 
    * golang.org/x/net/context 对应的代码地址在： https://github.com/golang/net
    * google.golang.org\genproto 对应的代码地址在: https://github.com/google/go-genproto.git
    * golang.org/x/net  对应代码地址为: https://github.com/golang/net
    * golang.org/x/text 对应代码地址为: https://github.com/golang/text
    
     如果运行时,还缺少包,则在git上找到相应的包然后再复制到相应的目录
## 过程
1. 下载protoc编译`proto`文件的编译工具: https://github.com/google/protobuf/releases,找到合适系统的版本`prtoc-XXX.zip`
1. 下载`proto-gen-go`插件,通过命令` go get -u github.com/golang/protobuf/protoc-gen-go`,下载后会在`GOPATH/bin`下看到文件
1. 将上述文件位置加入到系统变量中
1. 通过命令`protoc --go_out=. hello.proto`生成`hello.pb.go`文件,命令解释:
    * `--go_out=.`表示在当前目录下生成go文件
    * `hello.proto`指定根据哪个proto文件生成
1. 将生成的go文件复制到项目中.
1. 编写客户端调用程序,查看项目`demo-grpc-client-go`下的`app_test`中方法`TestHello`
## 运行
启动服务端,然后启动客户端调用方法.