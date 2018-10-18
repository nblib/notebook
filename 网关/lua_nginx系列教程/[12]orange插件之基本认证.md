基本认证
===
插件名称 : Basic Auth

---
##作用
通过对请求头中添加的认证信息进行认证拦截的插件
## 使用方法
1. 添加选择器
2. 添加规则
3. 编辑规则,添加拦截的url,然后设置用户名密码,可以设置多个,只要匹配一个就认证通过:
![编辑规则](../img/nginx/nginx_lua/orange/baseauth/基本认证.png)
4. 访问时,请求头中添加一个字段:"Authorization",值为"用户名密码"的base64编码结果:
![请求](../img/nginx/nginx_lua/orange/baseauth/请求.png)
5. 服务端会验证用户名和密码,进行拦截.