# 数据加载过程
`kong`使用了三种数据储存方式:
* `local`: 使用`lua-nginx`模块的共享内存储存
* `cluster`: 使用数据库形式储存,目前可用的数据库为"cassandra"和"postgres"
* `redis`: 使用`redis`储存数据

rate-limiting默认使用`cluster`储存,通过`config.policy`设置
插件配置信息,储存在本地作为缓存和数据库中,具体数据库由`database`决定


### 读取配置文件
1. `kong.init.lua`中,init方法用于初始化,调用`kong.conf_loader`解析配置文件,获取包含配置信息的`config`表
1. 调用`kong.dao.factory`,根据配置表,获取数据库类型,生成具体的数据库连接对象`db`,返回包含`db`的封装对象`dao`
1. 将配置信息,数据库信息存放到`singletons`全局对象中.
### 遍历插件
1. 调用`access`方法时,会调用`kong.core.plugins_iterator`的方法,遍历获取插件名称,插件的配置.遍历方法:[lua迭代器](../lua/lua迭代器.md)
1. 遍历时,首先回去加载到的插件名称,然后调用`kong.core.plugins_iterator`的`load_plugin_configuration`方法
1. 获取缓存的key,调用` singletons.dao.plugins:cache_key`获取key,实际上调用的是`kong.dao.dao`的`cache_key`方法.方法作用就是
将所有的参数组成一个格式化的字符串.
1. 根据获取的key去获取缓存,调用`singletons.cache:get`方法.同时传入一个方法用于当在缓存中找不到目标时调用获取数据.
实际调用的是`kong.cache`的`get`方法,然后又调用了`kong.mlcache`的get方法
1. `kong.mlcache`的get方法,从`lru`缓存中尝试获取,多次尝试后,调用回掉方法:` local ok, err = pcall(cb, ...)`,返回值第一个为
状态码:true或false,第二位为函数的返回值或者错误原因.
1. 回调函数实际为从数据库中获取,而获取后,会放入到缓存中.
1. 执行完毕后,返回插件和插件的配置信息
### 调用插件方法
1. 插件调用响应的方法,比如:access方法等,并将配置作为参数传递过去.
### 插件运行数据储存
比如,Rate-limiting插件运行时,需要记录访问次数等,保存数据通过`config.policy`设置储存位置.