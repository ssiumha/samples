# prometheus metrics

어떤 지표가 있고 어떤 지표를 쓸 수 있는지 작업하며 만난 exporter, metrics 정리

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
