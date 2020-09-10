## Elasticsearch的多种搜索

1. **query string search**（生产中很少使用）
>   搜索所有商品   `GET /ecommerce/product/_search 。`
在页面中执行  `http://localhost:9200/ecommerce/product/_search?q=_id:1`  （ ?q=_id:1  编号为1 的商品）。
took：耗费了几毫秒
timed_out：是否超时，这里是没有
_shards：数据拆成了5个分片，所以对于搜索请求，会打到所有的primary shard（或者是它的某个replica shard也可以）
hits.total：查询结果的数量
hits.max_score：score的含义，就是document对于一个search的相关度的匹配分数，越相关，就越匹配，分数也高
hits.hits：包含了匹配搜索的document的详细数据。

2. **query DSL （Domain Specified Language）**
> 搜索所有商品：
 ```
GET /ecommerce/product/_search
{
  "query": { "match_all": {} }
}
```
>查询名称包含yagao的商品，同时按照价格降序排序
```
GET /ecommerce/product/_search
	{
		"query" : {
			"match" : {
				"name" : "yagao"
			}
		},
		"sort": [
			{ "price": "desc" }
		]
	}
```
>分页查询商品，总共3条商品，假设每页就显示1条商品，现在显示第2页，所以就查出来第2个商品
```
GET /ecommerce/product/_search
{
  "query": { "match_all": {} },
  "from": 1,
  "size": 1
}
```
>指定要查询出来商品的名称和价格就可以
```
GET /ecommerce/product/_search
{
  "query": { "match_all": {} },
  "_source": ["name", "price"]
}
```
3. **query filter**
>  搜索商品名称包含yagao，而且售价在10 和40 直接的商品
```
GET /ecommerce/product/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "name": "yagao"
        }
      },
      "filter": {
        "range": {
          "price": {
            "gte": 10,
            "lte": 40
          }
        }
      }
    }
  }
}
```
4. **full-text search 全文检索** 
> 全文检索
```
GET /ecommerce/product/_search
{
    "query" : {
        "match" : {
            "producer" : "yagao producer"
        }
    }
}
```
> 首先es会对 /ecommerce/product/ 下所有document中的producer进行拆分，建立倒排索引。

|  field | documentid |
| -------- | -------- |
| special | 4  | 
| yagao | 4  | 
| producer| 1,2,3,4|  

>  ```"producer" : "yagao producer"```  这里将 yagao producer 进行拆分 拿着yagao和producer到倒排索引中进行查找匹配,只要匹配上一个就可以了。

5. **phrase search**
```
GET /ecommerce/product/_search
{
    "query" : {
        "match_phrase" : {
            "producer" : "yagao producer"
        }
    }
}
```
>  要求输入的搜索串，必须在指定的字段文本中，完全包含一模一样的，才可以算匹配，才能作为结果返回。

6. **highlight search**
```
GET /ecommerce/product/_search
{
    "query" : {
        "match" : {
            "producer" : "producer"
        }
    },
    "highlight": {
        "fields" : {
            "producer" : {}
        }
    }
}
```
> 将商品中producer fields 中包含producer的数据查出将 producer高亮显示。

## Elasticsearch 聚合
* 执行 聚合操作时  需要将文本field的fielddata属性设置为true
```
PUT /ecommerce/_mapping/product
{
  "properties": {
    "tags": {
      "type": "text",
      "fielddata": true
    }
  }
}
```
* 计算tags fields 中各个标签对应的商品数量
```
GET  /ecommerce/product/_search
{
  "aggs": {
    "group_by_tags": {
      "terms": { "field": "tags" }
    }
  }
}
```
```
  "aggregations": {
    "group_by_tags": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "fangzhu",
          "doc_count": 3
        },
        {
          "key": "meibai",
          "doc_count": 2
        },
        {
          "key": "qingxin",
          "doc_count": 1
        }
      ]
    }
  }
```
* 对名称中包含yagao的商品，计算每个tags中对应标签商品数量
```
GET /ecommerce/product/_search
{
  "size": 0,
  "query": {
    "match": {
      "name": "yagao"
    }
  },
  "aggs": {
    "all_tags": {
      "terms": {
        "field": "tags"
      }
    }
  }
}
```
* 先分组，再算每组的平均值，计算每个tag下的商品的平均价格
```
GET /ecommerce/product/_search
{
    "size": 0,
    "aggs" : {
        "group_by_tags" : {
            "terms" : { "field" : "tags" },
            "aggs" : {
                "avg_price" : {
                    "avg" : { "field" : "price" }
                }
            }
        }
    }
}
```
* 计算每个tag下的商品的平均价格，并且按照平均价格降序排序
```
GET /ecommerce/product/_search
{
    "size": 0,
    "aggs" : {
        "all_tags" : {
            "terms" : { "field" : "tags", "order": { "avg_price": "desc" } },
            "aggs" : {
                "avg_price" : {
                    "avg" : { "field" : "price" }
                }
            }
        }
    }
}
```
* 按照指定的价格范围区间进行分组，然后在每组内再按照tag进行分组，最后再计算每组的平均价格
```
GET /ecommerce/product/_search
{
  "size": 0,
  "aggs": {
    "group_by_price": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "from": 0,
            "to": 20
          },
          {
            "from": 20,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_tags": {
          "terms": {
            "field": "tags"
          },
          "aggs": {
            "average_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        }
      }
    }
  }
}
```