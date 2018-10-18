gzip文件在hdfs中储存机制
===
# 描述
gzip是一种不可分割的压缩算法,但是hdfs是按照块储存的.那么使用gzip压缩的文件在hdfs中怎么储存.

假设一个gzip压缩文件大小为1GB
## gzip文件在hdfs中怎么储存
如果block size 是64MB,文件是1GB,那么这个文件会被分成`1024/64=15.625 `,也就是16个block储存在hdfs中.并且所有的block储存在一个datanode上.
## 多少个block被创建
1GB/64MB = 15.625 ~ 16DFS blocks
## 副本怎么保存
同其他文件一样.如果replication factor is 3.那么1st第一个副本保存在当前机器上,第二个副本保存在不同机架的机器上,第三个副本保存在和第一个副本所属的同一个机架上的机器上.
## gzip的可分割和不可分割
gzip在hdfs上储存可以分割为多个block,但是它的不可分割表现在,当mapreduce时,不支持并行处理,也就是只能由一个mapreduce处理.