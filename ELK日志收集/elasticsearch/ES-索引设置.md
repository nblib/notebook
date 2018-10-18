es的索引设置
===
# 索引设置
可以按索引设置索引级别设置。 设置可能是：
* static(静态): 当一个索引创建时指定好设置,或者设置一个被关闭的索引.后续不可改变.
* dynamic(动态设置): 可以动态改变索引的设置.

## 静态设置索引(创建索引时指定)
以下是与任何特定索引模块无关的所有静态索引设置的列表：

###### `index.number_of_shards`
索引应具有的主分片数。默认为5.此设置只能在索引创建时设置.
它不能在封闭索引上更改.注意：每个索引的分片数限制为1024。 此限制是一个安全限制，用于防止意外创建可能因资源分配而导致群集不稳定的索引。 
可以通过在作为群集一部分的每个节点上指定export ES_JAVA_OPTS =“ -  Des.index.max_number_of_shards = 128”系统属性来修改限制。

###### `index.shard.check_on_startup`
是否应在打开之前检查碎片是否有损坏。 检测到损坏时，它将阻止分片被打开。可接受值为：
* false: 不检查损坏.
* checksum: 检查物理损坏
* true: 检查物理和逻辑损坏.这将花费CPU和内存
* fix: 检查物理和逻辑的损坏.损坏的分区将被移除.**这会导致数据丢失**

###### `index.codec`
默认值使用LZ4压缩压缩存储的数据，但这可以设置为`best_compression`，
它使用`DEFLATE`获得更高的压缩比，但代价是存储的字段性能较慢。
如果要更新压缩类型，则在合并段之后将应用新压缩类型。
可以使用强制合并强制进行段合并。

###### `index.routing_partition_size`
自定义路由值可以转到的分片数。 默认为1，只能在索引创建时设置。 除非index.number_of_shards值也是1，否则此值必须小于index.number_of_shards。有关如何使用此设置的详细信息，请参阅路由到索引分区。

## 动态设置索引
以下是与任何特定索引模块无关的所有动态索引设置的列表：

###### ` index.number_of_replicas `
每一个主分片的副本数量.默认为1.

###### ` index.auto_expand_replicas `
根据群集中的数据节点数自动扩展副本数。 可设置下限和上限（例如0-5）或
全部用于上限（例如0-all）。 默认为false（即禁用）。 请注意，自动扩展的副
本数不考虑任何其他分配规则，例如`shard allocation awareness`, `filtering` or` total shards per node`，如果适用的规则阻止所有副本，这可能导致群集运行状况变
为黄色.

###### `index.refresh_interval`
执行刷新操作的频率，这会使索引的最近更改对搜索可见.默认为1秒.可以设置为-1以禁用刷新.

###### `index.max_result_window`
分页查询需要指定start和size,这个参数限制start+size的大小.默认值为`10000`.也就是设置一次最大取得结果必须小于`10000`,
它不仅限制了用户在一次查询中最多数据条数是1w条，并且限制了start+size 必须小于1w，也就是说，想取第9999条，往后的2条数
据是不可以的，因为 9999+2 > 10000。如此一来，一石二鸟，同时防止了一次取太多和深度分页两个问题。

###### `index.routing.rebalance.enable`
是否开启这个索引的分片rebalancing,可选值有:
* all(默认): 允许所有的分片rebalancing
* primaries: 只允许主分片
* replicas: 只允许副本分片
* none: 不允许rebalancing

