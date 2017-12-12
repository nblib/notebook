# 概述
一个实时应用作业被打包到一个`storm`的`topology`中,一个`topology`类似于一个
`MapReduce`作业,关键区别是,`MapReduce`作业会结束,而`topology`不会结束,除非被关闭.
# 流分组
流分组用于决定哪个steam被哪个bolt消费和怎么被消费.
比如:
```
builder.setBolt("word-normalizer", new WordNormalizer())
.shuffleGrouping("word-reader");
```
### Shuffle Grouping
接收一个参数(数据源的id,比如spout的id)指明数据从哪里获取,然后发送数据到一个随机选择的
bolt中,保证每一个bolt获取相同数量的tuples.
###### 用途
适合数学计算,但是如果一个操作不能被分开计算,比如统计总数,就不适合了.
### Fields Grouping
基于tuple中的`Field`控制tuple被发送到哪个bolt,确保同一个Field下的相同的值被发送到同一个
bolt中(原理为使用hash判断值是否相同)
###### 用途
类似例子`world count`统计相同单词出现的次数.
### All Grouping
发送tuple的副本到每一个bolt中.
###### 用途
发送类似通知信号的内容给每一个bolt.比如清除缓存的信号,这样每个bolt收到后执行相应的处理
