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

- 하나의 Shard는 여러개의 Segement로 이뤄져있다
  - 역색인 정보가 담겨져 있는 검색 처리의 최소단위
  - 데이터가 추가될 때 메모리에 모아두었다가 디스크에 새 Segement를 작성하고 refresh 되며 검색이 가능해진다
  - Segement는 불변이며 데이터가 삭제되도 마크만 하고 실제로 지우진 않는다
  - Segement Merge
    - 삭제 마크된 Segement가 임계치를 넘어가거나 너무 많은 수의 Segement가 생기면 병합 작업이 진행된다
    - 합쳐지는 동안 들어오는 쿼리는 기존 Segement를 참조하며 병합은 복제된 Segement를 사용해서 이뤄진다
      - 병합 처리를 위한 disk 여유공간 필요
    - Segement Merge는 무거운 작업이므로 튜닝할 때 주의해야함

  ```
  merge, segment에 관련된 설정 확인하기:
  GET /<index>/_settings?include_defaults=true
  GET /_stats
  ```

- 7.0 이전에는 Index가 DB 개념이고 Type이 Table 개념이었으나
  의도했던 개념과 실사용이 달라서 삭제됨 (https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html)


- keyword: 입력된 문자열 전체를 심볼 하나로 판단. 전문일치용 타입
- text: 입력된 문자열을 token 단위로 파싱해서 색인을 만듬, 부분일치용 타입
- https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html


## Trouble Shooting

- refresh 작업은 index 갯수에 비례해서 늘어나며 Thread Pool operations completed 수치가 치솟으며 다른 작업에 영향을 줄 수 있다
  - 대규모 reindex를 진행하며 index가 대량으로 늘어났을 때, JVM 메모리 사용량과 Old GC가 늘어날 수 있다
  - index segments가 늘어나며 전체적인 처리가 느려진다

- Old GC가 낮은 빈도로 처리되는지 보는게 중요하다?

- ES 내부에 자체적으로 OOM 발생을 막기 위한 circuit breaker가 존재한다. JVM 지표와 함께 메모리 위험발생을 알 수 있는 지표가 되준다
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html#parent-circuit-breaker

- 메모리 사이즈를 잘 설정하자
  - https://www.elastic.co/guide/en/elasticsearch/guide/2.x/heap-sizing.html#_give_less_than_half_your_memory_to_lucene

## MISC

- term, match, match_phrase 동작 차이 확인하기, kibana에서는 match_phrase가 기본 검색으로 작동한다
  - term은 inverted index에 저장된 token 중에 일치하는 것을 찾고
  - match는 검색할 키워드를 analyze 하고서 검색한다
  - match_phrase는 쿼리 문자열이 순서를 유지한채 본문에서 나타나는 경우를 찾는다


## Node

### Master

- 각 노드가 갖고 있는 인덱스의 메타데이터, 샤드위치, 클러스터 상태 등을 관리
- Split brain 이슈를 피하기 위해서 node 갯수는 홀수개로 유지하는게 좋다

### Ingest

- 문서를 인덱싱하기 전에 pre-process 하는 노드
- index, bulk 요청을 받아서 처리하고 다시 넘긴다


### Coordinating

- 검색 요청이 들어왔을 때 해당되는 데이터가 있는 노드로 라우팅해주는 역할을 한다
- 각 노드의 처리 결과를 취합해서 반환한다
- 메모리, CPU 자원을 많이 쓰는 편

### Data

- 실제 데이터가 나뉘어서 들어가 있는 노드

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



