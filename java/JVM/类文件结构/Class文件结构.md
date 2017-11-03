Class文件结构
===

### class文件
* 一组以**8位字节**为基本单位的二进制流
* 中间没有分隔符
* 当需要储存8字节以上的的空间的数据项时,则按照高位在前原则分割为若干个8字节储存.
* 使用类似结构体的伪结构储存,只有两种数据类型: **无符号数和表**.
> 无符号数  
基本数据类型,u1,u2,u4,u8分别代表1,2,4,8个字节,用来描述数字,索引引用,数量值或者UTF-8编码构成字符串值.  

> 表是有多个无符号数或其他表作为数据项构成的符合数据类型.所有表都习惯性的以"_info"结尾,整个class文件本质上时一张表.

##### 文件格式

类型 | 名称 | 数量 | 含义 | 描述
---|---|---|---|---
u4 | magic | 1 | 魔数 | 确定这个文件是否可被虚拟机接受,用于身份识别(值为0xCAFFBABE)
u2 | minor_version | 1 | 次版本号 | 从45开始,每个大版本发布主版本号加1,高版本向下兼容,但不能运行以后的
u2 | major_version | 1 | 主版本号 | 从45开始,每个大版本发布主版本号加1,高版本向下兼容,但不能运行以后的
u2 | constant_pool_count | 1 | 常量池容量计数值 | 从1开始计数,用于标记常量池数量
cp_info | constant_pool | constant_pool_count - 1 | 常量集合
u2 | access_flags | 1 | 访问标识符
u2 | this_class | 1 | 类索引 | 用于确定这个类的全限定名,指向一个CONSTANT_Class_Info的类描述符常量
u2 | super_class | 1 | 父类索引 | 确定这个父类的全限定名,指向一个CONSTANT_Class_Info的类描述符常量
u2 | interfaces_count | 1 | 接口数量 | 没有实现接口,为0,后面的索引表不占任何字节
u2 | interfaces | interfaces_count | 接口集合
u2 | fields_count | 2 | 字段数量 
field_info | fields | fields_count | 字段集合 | 用于描述接口或类中声明的变量,包括类变量和实例变量,不包括局部变量
u2 | methods_count | 1 | 方法数量
method_info | methods | methods_count | 方法集合
u2 | attributes_count | 1 | 属性数量
attribute_info | attributes | attributes_count | 属性集合
