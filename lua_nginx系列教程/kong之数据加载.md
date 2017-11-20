# 数据加载过程
`kong`使用了三种数据储存方式:
* `local`: 使用`lua-nginx`模块的共享内存储存
* `cluster`: 使用数据库形式储存,目前可用的数据库为"cassandra"和"postgres"
* `redis`: 使用`redis`储存数据

rate-limiting默认使用`cluster`储存,通过`config.policy`设置
插件配置信息,储存在本地作为缓存和数据库中,具体数据库由`database`决定
