# 概述
使用错误页面处理,统一对全站的错误页面进行处理和展示.比如4XX和5XX错误.
# 处理方法
使用nginx核心包中的`error_page`指令.
###### 内部跳转(静态页面)
通过内部跳转到指定的url
``` 
error_page 404             /404.html;
error_page 500 502 503 504 /50x.html;
```
如上,会将状态码404,跳转到404.html,500,502等跳转到50x.html页面,如果400.html和50x.html时静态页面,那么返回码会保持不变

如果需要改变返回码:
``` 
error_page 404 =200 /empty.gif;
```
将状态码改为200
####### 内部跳转(动态页面)
``` 
error_page 404 = /404.php;
```
比如跳转到一个php程序,那么返回的状态码由php程序决定
###### 反向代理到错误处理服务器
``` 
location / {
    error_page 404 = @fallback;
}

location @fallback {
    proxy_pass http://backend;
}
```
或者直接链接到指定的url
``` 
error_page 403      http://example.com/forbidden.html;
error_page 404 =301 http://example.com/notfound.html;
```