# Elasticsearch


## Overview

TODO
- https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html
- https://esbook.kimjmin.net/08-aggregations/8.2-bucket-aggregations


구조

```
          +--------------------------------------------------------+
          | Cluster                                                |
          |                                                        |
          |  +-------------+   +-------------+   +-------------+   |
          |  | Node        |   | Node        |   | Node        |   |
          |  |             |   |             |   |             |   |
          |  | ---         |   | ---         |   | ---         |   |
Index1 => |  | Shard1-1    |   | Shard1-2    |   | Shard1-3    |   |
          |  | Shard1-3    |   | Shard1-1    |   | Shard1-2    |   |
          |  |             |   |             |   |             |   |
          |  | ---         |   | ---         |   | ---         |   |
Index2 => |  | Shard2-1    |   | Shard2-2    |   | Shard2-3    |   |
          |  | Shard2-3    |   | Shard2-1    |   | Shard2-2    |   |
          |  |             |   |             |   |             |   |
          |  +-------------+   +-------------+   +-------------+   |
          |                                                        |
          +--------------------------------------------------------+


ES        := DB
Index     := Table
Document  := Row

```

- Shard 단위로 연산이 이뤄지기 때문에 Shard 갯수가 너무 적거나 많게 만들면 안된다
  - 권장하는 Shard 크기는 10GB ~ 50GB: https://www.elastic.co/guide/en/elasticsearch/reference/7.10/size-your-shards.html

- 7.0 이전에는 Index가 DB 개념이고 Type이 Table 개념이었으나
  의도했던 개념과 실사용이 달라서 삭제됨 (https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html)


- keyword: 입력된 문자열 전체를 심볼 하나로 판단. 전문일치용 타입
- text: 입력된 문자열을 token 단위로 파싱해서 색인을 만듬, 부분일치용 타입
- https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html

## Query

기본 쿼리 형태:

```
curl -H 'Content-Type: application/json' -XPOST http://localhost:9200/index_name/new/ -d'{
}'
```

http://localhost:9200/product/_search?pretty
http://localhost:9200/product/_search


```
SELECT * FROM product
{ "query": { "match_all": {} } }

SELECT * FROM product LIMIT 10 OFFSET 5
{ "from": 5, "size": 10, "query": { "match_all": {} } }

SELECT name, cnt FROM product
{ "_source": ["name", "cnt"], "query": { "match_all": {} } }

https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html#request-body-search-collapse
SELECT DISTINCT name FROM product
{ "collapse": { "field": "name" }, "query": { "match_all": {} } }

SELECT * FROM product WHERE name = 'asdf'
{ "query": { "term": { "name": { "value": "asdf" } } } }

SELECT * FROM product WHERE name IN ('asdf', 'qwer')
{ "query": { "term": { "name": ["asdf", "qwer"] } } }

SELECT * FROM product WHERE name LIKE '%L%'
{ "query": { "wildcard": { "name": { "value": "*L*"} } } }

SELECT * FROM product WHERE cnt >= 10
{ "query": { "range": { "cnt": { "gte": 10 } } } }

AND: must
OR: should
NOT: must_not
SELECT * FROM product WHERE name = 'asdf' AND cnt >= 10
{ "query": { "bool": { "must": [ {"term":{"name":"asdf"}}, {"range":{"cnt":{"gte":10}}} ]} } }

SELECT * FROM product WHERE cnt >= 10 AND (name = 'asdf' OR price = 100)
{ "query": { "bool": { "must": [
  { "range":{"cnt":{"gte":10}} },
	{ "bool": {"should": [ {"term":{"name":"asdf"}}, {"term":{"price":100}} ] }}
]} } }


SELECT * FROM product ORDER BY name ASC
{ "query": { "match_all": {} }, "sort": [ { "product_name": { "order": "asc" } } ] }
```


## 데이터 집계

```
# https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html
# https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics.html
SELECT AVG(cnt) FROM product
{ "aggs": { "cnt_avg": { "avg": { "field": "cnt" } } } }


SELECT name FROM product GROUP BY name
{ "aggs": { "name_aggs": { "terms": { "field": "name", "size": 10 } } } } # 상위 10개까지의 값에 대해 group by

SELECT AVG(cnt) FROM product GROUP BY name
{ "aggs": { "name_aggs": { "terms": { "field": "name", "size": 10 }, "aggs": { "cnt_avg": { "avg": { "field": "cnt"} } }  } } }


# query로 반환되는 hits와 집계 결과인 aggregations는 별도의 항목
# 따라서 size를 0으로 하면 hits만 반환값이 없어져 리스폰이 가벼워진다
# (aggregations는 조건에 맞는 데이터만 집계한다)
SELECT AVG(cnt) FROM product WHERE name = 'asdf'
{
	"size": 0,
	"query": {
		"term": {
			"name": { "value": "asdf" }
		}
	},
	"aggs": {
		"cnt_avg": {
			"avg": { "field": "cnt" }
		}
	}
}

# TODO: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-pipeline-bucket-selector-aggregation.html
SELECT name, SUM(cnt) FROM product
  GROUP BY name
  HAVING SUM(cnt) > 10
{
  "aggs": {
    "name_aggs": {
      "terms": {
        "field": "name",
        "size": 10
      },
      "aggs": {
        "count_sum": {
          "sum": {
            "field": "count"
          }
        },
        "count_sum_bucket": {
          "bucket_selector": {
            "buckets_path": {
              "count_sum": "count_sum"
            },
            "script": "params.count_sum > 10"
          }
        }
      }
    }
  }
}
```



