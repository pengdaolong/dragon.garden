1、查看集群状态是否可用（red 为不可用）
```
curl  -XGET http://localhost:9200/_cat/health
```

2、集群可不用时  查看index primary key 状态   如果有UNASSIGNED状态的primary index  就是导致集群不可用的原因
```
curl -XGET http://localhost:9200/_cat/shards?pretty |grep UNASSIGNED |awk '{if($3=="p") {print $0}}'
```

3、删除UNASSIGNED状态的primary index
```
curl -XDELETE 'http://192.168.91.221:9200/INDEX

cat es.txt |while read line ; do curl -XDELETE 'http://localhost:9200/$line'; done
```
