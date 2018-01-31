# 概述
隐藏响应头的字段,比如`server`
# 方法
使用`openresty`的模块`headers-more-nginx-module`
比如;
``` 
http{
      #setting Server header
        more_set_headers    "Server: Unkown";
}
```
设置`Server`内容为`Unkown`