#插件列表
* 自定义监控
* URL重定向
* URL重写
* Basic Auth(基本认证)
* Key Auth(关键字认证)
* Signature Auth(签名认证)
* Rate Limiting 访问限速
* Property Rate Limiting 访问限速,防刷
* WAF 防火墙
* 代理 & 分流 & ABTesting

### 自定义监控
针对指定的请求地址,主机,客户端IP,请求头,请求引用,代理,参数等条件进行监控,获取统计数据报表
### URL重定向
301,302的重定向,可参考[URL重定向插件使用示例](http://orange.sumory.com/docs/usages/redirect/)
### URL重写
内部重定向,客户端地址栏不变,可参考[URI重写插件使用示例](http://orange.sumory.com/docs/usages/rewrite/)
### Basic Auth(基本认证)
基于Basic Auth的简单认证
### Key Auth(关键字认证)
通过获取请求地址,主机,客户端IP,请求头,请求引用,代理,参数等条件中的值来进行验证,比如要求请求头中包含`ni->wo`的键值对,那么如果请求头中没有的话,就会验证失败.
### Signature Auth(签名认证)
采用md5加密请求参数,进行验证,可参考[Signature Auth(签名认证)](https://github.com/sumory/orange/issues/72)
### Rate Limiting (访问限速)
根据指定的条件对访问进行限制,比如1秒可以请求多少次等.
### Property Rate Limiting (访问限速,防刷)
和 Rate Limiting 访问限速 差不多
### WAF (防火墙)
对指定条件进行拒绝访问
### (代理)(分流)ABTesting
将指定的条件请求交由设置的upstream处理.