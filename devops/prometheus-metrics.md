# prometheus metrics

어떤 지표가 있고 어떤 지표를 쓸 수 있는지 작업하며 만난 exporter, metrics 정리

- [promlens](https://promlens.com)로 좀 더 편리하게 확인가능
- https://prometheus.io/docs/prometheus/latest/querying/api/
- /api/v1/targets/metadata 로 모든 target과 metric에 대한 metadata를 얻을 수 있다

```
    {
      "target": {
        "container": "controller",
        "endpoint": "metrics",
        "instance": "10.0.1.1:10254",
        "job": "http-nginx-ingress-ingress-nginx-controller-metrics",
        "namespace": "helm",
        "pod": "http-nginx-ingress-ingress-nginx-controller-6b769c8bdf-dsj6f",
        "service": "http-nginx-ingress-ingress-nginx-controller-metrics"
      },
      "metric": "nginx_ingress_controller_request_duration_seconds",
      "type": "histogram",
      "help": "The request processing time in milliseconds",
      "unit": ""
    },
```

```
# job 별 metric 가짓수 확인
curl -s localhost:9090/api/v1/targets/metadata | jq -r -c '.data[] | [.target.job, .metric] | join(" ")' \
  | sort | uniq | awk '{print $$1}' | uniq -c | sort -n

# 전체메트릭 출력
curl -s localhost:9090/api/v1/targets/metadata | jq -r -c '.data[] | [.target.job, .target.service, .metric, .type]' | sort
```

- summary 케이스
  - go_gc_duration_seconds
  - etcd_request_latencies_summary
  - grafana_api_dataproxy_request_all_milliseconds
  - http_request_duration_milliseconds
- histogram
  - alertmanager_http_request_duration_seconds
  - alertmanager_http_response_size_bytes
  - apiserver_response_sizes
  - apiserver_request_duration_seconds
  - rest_client_request_duration_seconds


## Kafka Overview

- https://grafana.com/grafana/dashboards/8582
- https://grafana.com/grafana/dashboards/5468

왠지 모르겠는데 metric이 kafka_log_log_size로 남고 있다.. (chart ver 11.8.6)

```
# kafka log size by topic
sum by(topic) (kafka_log_log_size)
```

## nginx ingress metircs

```
# 하위 99% Latency 취득해오기.
histogram_quantile(
  0.99,
  sum by (le, ingress) (
    rate( nginx_ingress_controller_request_duration_seconds_bucket {
            ingress!="",controller_pod=~"$controller",
            controller_class=~"$controller_class",
            controller_namespace=~"$namespace",
            ingress=~"$ingress"
          }[2m]
    )
  )
)
```

## elasticsearch-exporter metrics

- https://hub.docker.com/r/bitnami/elasticsearch-exporter
- https://github.com/justwatchcom/elasticsearch_exporter


## redis metrics

- redis-cli info 정보가 쌓임
- https://redis.io/topics/lru-cache
- http://antirez.com/news/109
- https://redis.io/commands/INFO

```
127.0.0.1:6379> info memory
# Memory
used_memory:3930208               # redis가 allocate해서 실제 사용중인 메모리
used_memory_human:3.75M
used_memory_rss:11366400          # OS가 redis에 할당해준 메모리, OOM 판정은 이쪽 값을 통해 발생한다
used_memory_rss_human:10.84M
used_memory_peak:4483880
used_memory_peak_human:4.28M
used_memory_peak_perc:87.65%
used_memory_overhead:1921456
used_memory_startup:810128
used_memory_dataset:2008752
used_memory_dataset_perc:64.38%
allocator_allocated:4048480
allocator_active:4476928
allocator_resident:7995392
total_system_memory:4130234368    # system memory인데 minikube 환경인 경우 전체 docker memory가 잡혔다
total_system_memory_human:3.85G
used_memory_lua:37888
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:4194304                 # 데이터 최대 메모리. write할때 used_memory가 이 값을 넘어서면 maxmemory policy에 따라 불필요한 키값이 제거된다
maxmemory_human:4.00M
maxmemory_policy:allkeys-lfu
allocator_frag_ratio:1.11
allocator_frag_bytes:428448
allocator_rss_ratio:1.79
allocator_rss_bytes:3518464
rss_overhead_ratio:1.42
rss_overhead_bytes:3371008
mem_fragmentation_ratio:2.89
mem_fragmentation_bytes:7436256
mem_not_counted_for_evict:4
mem_replication_backlog:1048576
mem_clients_slaves:41024
mem_clients_normal:20504
mem_aof_buffer:8
mem_allocator:jemalloc-5.1.0
active_defrag_running:0
lazyfree_pending_objects:0
lazyfreed_objects:0
```



## rails puma metrics

- https://github.com/puma/puma/blob/master/docs/architecture.md

```
+--PUMA---------------------------+
| +-----------------------------+ |
| | Control Application         | |
| +-----------------------------+ |
|                                 |
| +-----------------------------+ |
| | Runner                      | |
| +-----------------------------+ |
| +-----------------------------+ |
| | Worker [ Thread | Thread ]  | |
| +-----------------------------+ |
| +-----------------------------+ |
| | Worker [ Thread | Thread ]  | |
| +-----------------------------+ |
+---------------------------------+
```


- max_threads
  - 설정된 최대 스레드 갯수
- running
  - puma worker에서 작동중인 스레드 갯수
- pool_capacity
  - 현재 서버가 처리 가능한 요청 수
  - pool_capacity가 떨어지면 서버에 부하가 걸린 것으로 판단할 수 있다
- backlog
  - queue에 대기 가능한 요청 갯수
  - 기본값은 1024로 queue가 전부 차면 OS에서 요청처리를 거부한다


`(1 - pool_capacity / max_threads) * 100`를 사용해서 부하 정도를 계략적으로 측량할 수 있다.
