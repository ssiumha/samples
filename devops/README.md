# Devops 작업하며 만난 이슈들

- [kube](./kube.md)
- [helm](./helm.md)
- [terraform](./terraform.md)

- [prometheus](./prometheus.md)
- [prometheus-metrics](./prometheus-metrics.md)
- [kafka](./kafka.md)

- [aws](./aws.md)
- [aws-cli](./aws-cli.md)


- https://www.robustperception.io/blog

- https://github.com/ContainerSolutions/k8s-deployment-strategies


## 기타

- pod당 memory는 최대 2~4GB 정도가 이상적?

## TODO

- runbook
- prometheus local test
- jvm old gen gc

- https://grafana.com/docs/grafana/latest/datasources/tempo/
  - App tracer
- otel: https://opentelemetry.io/docs/concepts/
- jaeger: https://medium.com/jaegertracing/jaeger-and-opentelemetry-1846f701d9f2


## links

- https://kubernetes.github.io/ingress-nginx/examples/auth/oauth-external-auth/
- oauth2-proxy


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

## memory

프로세스에서 사용하는 공유라이브러리는 한번만 메모리에 올려놓고 여러 프로세스간에 공유해서 사용하기 때문에
일반적으로 어떤 프로세스가 사용하는 메모리 사용량을 정확하게 측정할 수 없다.

이때문에 여러 프로세스가 떠있을 때 메모리를 전부 취합하면 실제 사용메모리보다 많은 값을 얻게 된다.
그래서 여러 관점의 메모리 값을 통해서 사용량을 유추해야하는듯.

보통 다음과 같은 크기 관계를 갖는다: VSS >= RSS >= PSS >= USS

- VSS (virtual set size)
  - 프로세스의 가상메모리 크기
  - 실질적으로 프로세스가 가질 수 있는 최대크기를 나타낸다

- RSS (resident set size)
  - 프로세스가 물리메모리에서 점유중인 크기
  - 전체 공유메모리가 포함되며 공유메모리가 어느정도인지 판단할 수는 없다

- PSS (proportional set size)
  - USS + (공유 메모리 / 공유하는 프로세스수)
  - 공유 메모리를 대략적으로 나눠서 추정값을 얻는다

- USS (uniq set size)
  - 프로세스가 공유하지 않는 고유한 Private 메모리 크기

### memory 정보 얻기

```
java -XX:+PrintFlagsFinal -version | grep Heap
```

## network trouble shoothing

서버에 네트워크 문제 발생시 확인할 목록

TODO

```
ping

curl

lsof

nslookup
host

dig
```
