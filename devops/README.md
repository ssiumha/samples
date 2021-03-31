# Devops 이것저것

- [infra](./infra.md)
- [prometheus](./prometheus.md)
- [aws-cli](./aws-cli.md)


## TODO

- runbook
- prometheus local test
- jvm old gen gc

## etc

```
ab -c 10 -n 100 -s 6000 "https://xxxx/xxxx/xxxx"
```


## LogQL

grafana, loki 삽질용

```
# 로그 갯수 상위 10개
topk(
  10,
  sum by (job) (count_over_time({job!=""}[10m]))
)

# 로그 소스 기준 갯수 상위 15개 (sidekiq log)
topk(
  15,
  sum by (log_source) (count_over_time(
    {job=~".+-sidekiq-.+"} != "prometheus"
      | regexp "^[0-9:]+ sidekiq.1[| ]+(?P<log_ts>[0-9-]{10} [0-9:.]+) (?P<log_level>[IWE]) (?P<log_source>\\[.+?\\])(?P<log_class> \\[.+?\\])?(?P<log_body>.+)"
      | label_format log_source=`{{ regexReplaceAll "\\d+:processor " .log_source "" }}`
      [10m]
    ))
)

```
