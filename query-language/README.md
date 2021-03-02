# Query

- 쿼리중에 찾기 귀찮은거 모아놓음

## XPath

- `=`는 완전 일치, 부분 일치는 contains 사용
- `/` : 직계자손만 체크 (css: `>`)
- `//` : 하위요소 중에서 체크

```
# div#article_body의 innerText
//div[@id='article_body']/text()

# innertText가 원문인 것을 찾음
//a[text()='원문']

# 자식 구조
//div[@class='sponsor']/a[text()='원문']

# 복수 조건 + 프로퍼티 추출
a[text()='원문'][@class='basic_doc']/@href

# class에서 부분 일치
div[contains(@class, 'some-class')]
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
