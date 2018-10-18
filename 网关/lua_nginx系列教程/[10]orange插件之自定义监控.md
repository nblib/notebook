自定义监控  
===
插件名称 : monitor

---
##作用
对指定的url的请求次数,QPS(每秒请求次数),请求状态等等进行统计.
## 使用方法
1. 打开插件
2. 添加一个选择器,比如对用户中心感兴趣,如下:
![添加选择器](../img/nginx/nginx_lua/orange/monitor/添加选择器.png)
3. 选中选择器,添加一个规则:
![添加规则](../img/nginx/nginx_lua/orange/monitor/添加规则.png)
4. 编辑规则,注意:规则和select中的拦截没有关系,比如:selector中配置"^/user",那么ruler想要拦截用户信息的话,必须"^/user/info"
不能省去"^/user"写"^/info"
![编辑规则](../img/nginx/nginx_lua/orange/monitor/编辑规则.png)
5. 开启选择器和 规则后,访问统计页面,可以看到统计结果:
![统计视图](../img/nginx/nginx_lua/orange/monitor/统计视图.png)
点击这里可以查看表格视图:
![表格视图](../img/nginx/nginx_lua/orange/monitor/表格视图.png)
