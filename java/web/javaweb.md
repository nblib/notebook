##### HttpServletRequest的getQueryString()和getParameter()方法的区别
AAA|getQueryString()| getParameter()
---|---|---
方法|get|post,get
参数|url后所有的内容字符串 ,未进行解码|解码后的键值对

##### 获取请求参数的值和在request对象中set或者get属性区别
######   来源不同：
* 参数（parameter)  
是从客户端（浏览器）中由用户提供的，若是GET方法是从URL中
提供的，若是POST方法是从请求体（request body）中提供的；
* 属性（attribute）  
是服务器端的组件（JSP或者Servlet）利用requst.setAttribute（）设置的
###### 操作不同：
* 参数（parameter）的值只能读取不能修改，读取可以使用request.getParameter()读取；
* 属性（attribute）的值既可以读取亦可以修改，读取可以使用request.setAttribute(),设置可使用request.getAttribute()
###### 数据类型不同：
* 参数（parameter）不管前台传来的值语义是什么，在服务器获取时都以String类型看待，并且客户端的参数值只能是简单类型的值，不能是复杂类型，比如一个对象。
* 属性（attribute）的值可以是任意一个Object类型。

##### request,session,servlet生命周期
  1. request  
每次请求到了时初始化,相应后销毁
  1. session  
通过request调用getSession()方法时或者放值时创建,在web.xml中配置销毁时间
  1. servlet  
第一次访问path时创建,县创建request再创建的servlet
  1. listener  
系统初始化的时候创建