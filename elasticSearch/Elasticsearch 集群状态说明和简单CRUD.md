## 状态说明
1.  该命令查看集群状态信息  (?v  显示信息带属性名)
```
GET  /_cat/health?v
```

| epoch | timestamp | cluster | status |  node.total | node.data | shards| pri | relo |init |unassign |pending_tasks|max_task_wait_time |active_shards_percent |
| -------- | -------- | -------- |-------- | -------- | -------- |-------- | -------- | -------- |-------- | -------- | -------- |-------- | -------- | -------- |
| 1545717544     | 13:59:04       |      ptx-es      |yellow     | 1     | 1     |543     | 543     | 0     |0     | 165     | 0     | - |76.7%

*   timestamp 当前时间 
*  cluster status  集群状态 （red不是所有索引的primary shard都是active状态的，部分索引有数据丢失了 。yellow 每个索引的primary shard都是active状态的,但是部分replica shard不是active状态。green 每个索引的primary shard和replica shard都是active状态的）
*  node.total  集群总节点数
*  node.data  数据节点数
*    shards  分片数
*    primary   (primary shard )  主分片数 
*    unassign 未分配的（replica 没有机器可以备份，就会出现）
*    active_shards_percent 激活的分片百分比 primary shard  / primary shard + unassign。

2.  查看集群中有哪些索引
```
GET /_cat/indices?v 
```


| health | status  |index   |uuid   |pri |  rep |   docs.count  |  docs.deleted    |   store.size  |   pri.store.size   |
| -------- | -------- | -------- |-------- | -------- | -------- |-------- | -------- | -------- |-------- | -------- | -------- |
| green     | open     | all_p95_hour     | kkK3iXkhSmOAPh8t_AQPfg|    2  |    0  |0       |  0|  522b| 522b|
* index 索引名字
* uuid 唯一标识
* pri （primary shard） 主分片数
* rep （replica shard ） 备份分片数
* docs.count  文档数
*  docs.deleted  删除文档数
*  store.size 索引存储的总容量
*   pri.store.size  主分片的总容量（如果没有设置备份数 这个应该跟store.size 一样大）。

3.   查看集群所在磁盘的分配状况   
```
GET  /_cat/allocation?v 
```
* >shards disk.indices disk.used disk.avail disk.total       disk.percent          host                     ip           node
* >543        3.6gb                 21.4gb    226.4gb     247.9gb            8                          10.10.0.116  10.10.0.116 ptx-1
* >165                                                                                   UNASSIGNED
* 索引所占空间（disk.indices），该节点中所有索引在该磁盘所点的空间。
* 磁盘使用容量（disk.used），已经使用空间21.4gb
* 磁盘可用容量（disk.avail），可用空间可以226.4gb
* 磁盘总容量（disk.total），总共容量247.9gb
* 磁盘便用率（disk.percent），磁盘使用率8%。

4.看看集群节点
```
GET /_cat/nodes?v
```
| ip| heap.percent | ram.percent |cpu| load_1m | load_5m |load_15m| node.role| master|name| 
| -------- | -------- | -------- |-------- | -------- | -------- |-------- | -------- | -------- |-------- | 
|  10.10.0.116     | 78     | 81     | 6     | 0.70     |   0.88     |  0.91     | mdi     | *     |     ptx-1     | 

## Elasticsearch  CRUD 
1.  创建索引 ：`PUT /test_index?pretty `。
2.  删除索引：`DELETE /test_index?pretty`。
3.  新增文档，建立索引
```
PUT /index/type/id
{
  "key": "val",
  "key2": "val"
}
```
4.查询文档
```
GET /index/type/id
```
5.修改文档：
```
PUT /index/type/id 
  {
  "key":"val1",
   "key2":"val"
    } 
```
需要把盖doucument下所有的  都带上 然后会把原来的数据覆盖也就是修改了 这里是把key对应的值改成了val1。
```
post /index/type/id/_update 
  {
  "key":"1"
  }
```
  只更新 key 的值改成了 1
6.删除文档：
```
DELETE /ecommerce/product/1
```