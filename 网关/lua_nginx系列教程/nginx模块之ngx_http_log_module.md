# 概述
日志对于统计排错来说非常有利的。本文总结了nginx日志相关的配置如access_log、log_format、open_log_file_cache、log_not_found、log_subrequest、rewrite_log、error_log。
nginx有一个非常灵活的日志记录模式。每个级别的配置可以有各自独立的访问日志。日志格式通过log_format命令来定义。ngx_http_log_module是用来定义请求日志格式的。

# 文本地址
[nginx日志配置 | 运维生存时间](http://www.ttlsa.com/linux/the-nginx-log-configuration/)