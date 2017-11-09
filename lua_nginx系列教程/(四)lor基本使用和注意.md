#基本使用
参考[lor官方文档](http://lor.sumory.com/guide/index.html)
#使用说明
支持的HTTP 请求方法,参考lor源码中的`lor/lib/methods.lua`:
``` 
local supported_http_methods = {
    get = true, -- work well
    post = true, -- work well
    head = true, -- no test
    options = true, -- no test
    put = true, -- work well
    patch = true, -- no test
    delete = true, -- work well
    trace = true, -- no test
    all = true -- todo:
}
```
需要扩展方法可以在这里添加即可.