handler模块
===
**Handler模块就是接受来自客户端的请求并产生输出的模块。**
##### handler模块处理的结果通常有三种情况: 
**处理成功，处理失败（处理的时候发生了错误）或者是拒绝去处理**。在拒绝处理的情况下，这个location的处理就会由默认的handler模块来进行处理。
例如，当请求一个静态文件的时候，如果关联到这个location上的一个handler模块拒绝处理，就会由默认的ngx_http_static_module模块进行处理，该模块是一个典型的handler模块。
### 模块配置结构
对于模块配置信息的定义，命名习惯是ngx_http_<module name>_(main|srv|loc)_conf_t。
```
typedef struct
{
    ngx_str_t hello_string;
    ngx_int_t hello_counter;
}ngx_http_hello_loc_conf_t;
```
### 模块配置指令
一个模块的配置指令是定义在一个静态数组中的。
``` 
static ngx_command_t ngx_http_hello_commands[] = {
   {
        ngx_string("hello_string"),
        NGX_HTTP_LOC_CONF|NGX_CONF_NOARGS|NGX_CONF_TAKE1,
        ngx_http_hello_string,
        NGX_HTTP_LOC_CONF_OFFSET,
        offsetof(ngx_http_hello_loc_conf_t, hello_string),
        NULL },

    {
        ngx_string("hello_counter"),
        NGX_HTTP_LOC_CONF|NGX_CONF_FLAG,
        ngx_http_hello_counter,
        NGX_HTTP_LOC_CONF_OFFSET,
        offsetof(ngx_http_hello_loc_conf_t, hello_counter),
        NULL },

    ngx_null_command
};
```