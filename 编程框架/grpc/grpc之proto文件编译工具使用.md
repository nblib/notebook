# 概述
用于根据proto文件生成不同语言的类和方法.
# 用法
> go_out=.
* 指定输出目录
> protoc --go_out=. ./hello.proto
* 根据文件生成go语言版本

> protoc -I ./protofile --go_out=. ./hello.proto
* 扫描依赖文件,生成
`-I`选项用于proto文件相互依赖包含的情况下指定需要扫描的文件夹,从而导入依赖防止报错.
