#概述
选择器和规则是为了对拦截的uri进行分组,selector用于分组,规则用于具体拦截,比如用户中心和商品中心,url分别为:`http://localhost/user`, `http://localhost/product`,如果针对用户和商品分别处理那么:
* selector1为"/user/",ruler1为"/user/info/"
* selector2为"/product/",ruler1为"/product/info/"
selector作用仅仅是用于分组规划,方便管理
### 涉及到的问题
###### 全流量和分流量
* 全流量: 访问网站的所有路径都拦截
* 分流量: 只拦截指定的url的访问
######  规则中的`match`和`=`
* match : 指的是正则表达式匹配
* = : 完全相等(比如: `http://www.baidu.com/user`和`/user`相等)
参照[条件判断模块](http://orange.sumory.com/docs/usages/judge/)
###### 索引式和模板式
参照 [变量提取模块的使用](http://orange.sumory.com/docs/usages/extractor/)
###### 