# Prometheus

- https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

서버, 서비스 모니터링을 하기 위해 metric, alert을 관리해주는 모니터링 솔루션.
kube-prometheus-stack으로 prometheus, grafana, alertmanager, exporter 등을 한번에 관리할 수 있다.

## PromQL

- https://awesome-prometheus-alerts.grep.to/rules
- https://prometheus.io/docs/prometheus/latest/querying/functions/

```
# vector
<instant_query>{labels}[<range>:<resolution>] offset <duration>
ex) up[1h:10m] offset 1w29h30m

# functions

<instant-vector> : 순수한 메트릭 값
<range-vector> : [4m] 등으로 시간구간을 줘서 가공된 값

## 반환은 전부 instant-vector
delta( range-vector ) : 구간내 처음과 맨끝의 값 차이를 구한다
idelta( range-vector) : 구간내 값중 가장 마지막 2개의 차이를 구한다
increase( range-vector ) : 구간내의 값의 증가량을 구한다
rate( range-vector ) : 구간 양끝값을 사용한 증가율을 구한다 (0~1)
irate( range-vector ) : 구간내 값중 가장 마지막 2개의 증가율을 구한다 (0~1).
                        rate에 비해 값의 변화량이 급격한 경향이 있어서 피크값 체크에 유용

## 조건검사 함수
absent( instant-vector ) : 값이 존재하지 않다면 1을 반환. metric이 없는 경우를 체크 가능
changes( range-vector ) : 값이 변화한 횟수를 반환
resets( range-vector ) : 값이 리셋된 횟수를 반환

clamp_max, clamp_min(instant-vector, scalar) : scalar보다 크거나/작은 값을 제거한다

# https://github.com/prometheus/prometheus/blob/main/promql/quantile.go
histogram_quantile(<0-1>, <instant vector>) : 하위 n%에 해당하는 값을 가져올 수 있다

# group_by 합산
sum by (<labels>) (<metrics>)
sum (metrics) by (<labels>)


# and or not
<expr> and vector(1) # intersection
<expr> or vector(0) # union
<expr> unless vector(0) # complement
```

## AlertManager

- https://github.com/prometheus/alertmanager
- https://prometheus.io/docs/alerting/latest/configuration
- https://awesome-prometheus-alerts.grep.to/rules

* alertmanager는 내부 컨테이너의 config-reloader를 통해 설정을 갱신하는데, 만약 설정이 잘못되어있다면 반영되지 않고 멈춘다
  - alertmanager:9093/#/status에서 설정이 어떤 상태인지 확인할 수 있다
* `kubectl logs -f -nXXX pod/alertmanager-prometheus -c alertmanager` 혹은 `-c config-reloader`로 내부 컨테이너 로그를 읽을 수 있다
* 강제 리로드를 -XPOST alertmanager:9093/-/reload 로 할 수 있지만, helm 환경에서는 config-reloader가 처리해주므로 신경안써도 된다
* alert를 직접 트리거할 수 있다. template, route 테스트에 유용

  ```
  # https://prometheus.io/docs/alerting/latest/clients/
  curl -XPOST http://localhost:9093/api/v1/alerts \
    -d '[
      { "status": "firing",
        "labels": { "alertname": "테스트", "service": "curl", "severity": "warning", "instance": "0" },
        "annotations": {"summary": "This is a summary","description": "This is a description." },
        "startsAt": "2021-03-24T01:16:36+09:00",
        "endsAt": "2021-03-24T01:18:36+09:00",
      }
    ]'
  ```


prometheus에서 트리거된 rule에 따라 알림을 email, slack, webhook으로 처리해주는 서비스.
실제 설정은 prometheus에서 해줘야하고 rule에 지정된 group에 따라 어떻게 발송할지를 설정한다.

template은 golang template을 사용하며 Annotations, Labels, Alerts 값을 사용할 수 있다.

```
global:
  resolve_timeout: 5m
  slack_api_uri: https://hooks.slack.com/services/XXXXXXXXX/XXXXXXXXXXX/XXXXXXXXXXXXXXXXXXXXXXXX

route:
  receiver: 'slack-alert'

  group_by: ['alertname'] # alertname으로 groups[].name 값을 사용할 수 있다

  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1m

  routes:
  - receiver: 'slack-alert'
    group_wait: 10s
    match:
      alertname: mysql-critical
    # 정규식으로도 검색가능
    # match_re:
    #   service: mysql.+ 또는 service: mysql|redis
    # group_by 등등 route에 지정된 설정을 덮어쓰기할 수 있다
  - receiver: 'null' # alert을 발송하지 않고 버린다

receivers:
- name: slack-alert
  slack_configs:
  - channel: #alert-dev
    username: alert-bot
    #title: ''
    #text: '{{ template "custom_title" . }}{{- "\n" -}}{{ template "custom_slack_message" . }}'

templateFiles:
  template_1.tmpl: |-
    {{ ... }}
```

- route에 정의를 해놓은 설정들은 routes에도 전부 반영이 된다? 개별 설정이 필요할 때만 명시하면 된다

- group_by: rule의 label을 어떤 단위로 묶어서 처리할지 지정한다.
  - group_by 처리되면 같은 라벨을 갖고 있는 룰을 하나의 alert으로 보낸다
- group_wait: 트리거 되고, 얼마나 있다가 전송할지. 그 사이에 새로 발생한 알림들을 모았다가 처리한다
- group_interval: group을 어느 단위로 평가할지 지정, 너무 길게 하면 resolve 반응도 늦어진다
- match: 적어놓은 라벨 key: value가 일치하는 대상에 대해서만 처리한다 (route 관리)
- send_resolved: alert이 복구 되었을 때 메시지를 보낼지 여부


- template data구조: https://prometheus.io/docs/alerting/latest/notifications/
- template playground: https://juliusv.com/promslack/
- https://medium.com/quiq-blog/better-slack-alerts-from-prometheus-49125c8c672b
- go template을 사용하며 `{{-` 나 `-}}` 로 공백을 신경쓰며 작성해야한다

```
# template에서 사용할 수 있는 값

# slack에 알림보내기
# @here로는 bot에서 평문텍스트로 나와서 <!___> 형태로 만들어줘야한다
<!here>

# 기타 변수들
{{ .Alerts.Resolved }}
{{ .Alerts.Firing }}

{{ range .Alerts }}
  # firing, resolved?  값을 가질 수 있음
  {{ .Status }}
  {{ if eq .Status "firing" }}danger{{ else }}good{{ end }}

  # labels에 직접 정의한 값과 쿼리시 가져올 수 있는 값들
  {{ .Labels.serverity }}

  # annotations는 메시지 작성과 사용자가 메시지 구분을 하기 위해서 사용됨
  {{ .Annotations.summary }}
{{ end }}


# group by 로 묶여진 라벨을 표시 (group_by: [a, b] 면 a, b가 있다)
{{ .GroupLabels }}


# exmaple
{{ define "cluster" }}{{ .ExternalURL | reReplaceAll ".*alertmanager\\.(.*)" "$1" }}{{ end }}

{{ define "slack.myorg.text" }}
{{- $root := . -}}
{{ range .Alerts }}
  *Alert:* {{ .Annotations.summary }} - `{{ .Labels.severity }}`
  *Cluster:*  {{ template "cluster" $root }}
  *Description:* {{ .Annotations.description }}
  *Graph:* <{{ .GeneratorURL }}|:chart_with_upwards_trend:>
  *Runbook:* <{{ .Annotations.runbook }}|:spiral_note_pad:>
  *Details:*
    {{ range .Labels.SortedPairs }} • *{{ .Name }}:* `{{ .Value }}`
    {{ end }}
{{ end }}
{{ end }}
```

## Rules

```
additionalPrometheusRulesMap:
  <rule-name>: # prometheusrule/prometheus-stack-<rule-name> 형태로 생성된다
    groups:
      - name: mysql-restart
        rules:
          - alert: MysqlRestarted
            # record: my_record
            expr: mysql_global_status_uptime  < 300
            for: 1m
            labels:
              severity: warning
            annotations:
              summary: MYSQL RESTARTED
          - alert: InstanceDown
            expr: up == 0
            for: 30s
            labels:
              severity: page
            annotations:
              description: '{{ $labels.instance }} : {{ $labels.job }} has been down for more than 30s'
              summary: 'Instance {{ $lables.instance }} down'

            # 사용할 수 있는 변수 목록
            # $labels -> 쿼리 결과로 나온 metric의 label 정보. $labels 하나만 작성하면 전부 출력 가능
            # $value -> expr을 평가할 때 사용한 값?
```

- expr: promql, |- 을 사용해서 text로 넘길 수 있다
- for: 이 시간 이상 지속될 경우 트리거
- label: alertmanager에서 group by 등으로 사용할 수 있는 라벨, expr에 쿼리된 label에 추가되서 적용됨
- annotations: 부가 정보 작성

- https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/

alert rules 말고 record rules 또한 지정할 수 있는데,
record는 복잡하고 빈번하게 발생하는 쿼리를 새 이름으로 metric을 만들어 부하를 줄이고 편의성을 주는 기능이다.
