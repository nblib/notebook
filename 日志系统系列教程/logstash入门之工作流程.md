# 概述
logstash 执行分为三个阶段:input -> filter -> output.input阶段生成事件,过滤阶段修改事件内容,输出阶段将内容输出到别的地方
.input和output支持codec(编码).`codec`可以将输入的内容解码,输出的内容编码.这样可以不用使用filter阶段的编码过滤器.
# Inputs
使用`Inputs`获取数据到logstash中,比如:
* file: 从文件中获取数据
* syslog: 监听端口获取系统日志.

等等...
# filters
在logstash运行的中间阶段,对数据进行过滤修改,可以通过条件语句联合使用多个过滤器,针对不同的数据进行过滤.
# Outputs
执行的最后阶段.可以输出内容到多个地方,一旦所有的输出执行完成.那么这个流程就会结束.
# codec
基本的流过滤,作用于input和output,可以对输入/输出数据进行简单的处理,比如json数据的解析编译等等.
