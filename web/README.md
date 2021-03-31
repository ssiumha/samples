# Web

- [css](./css.md)
- [js](./js.md)


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


## 코딩말고 기타 웹 환경적인 부분들

### open graph

```
<meta property="og:title" content="title..">
```

head에 정의되는 다른 사이트에서 링크되었을 때 보여줄 스니펫 정보.
대부분의 경우 일정 시간 캐싱을 하며 참조자로 updated_time 을 사용하기도 한다.
페이스북 같은 경우 이미지 주소와 updated_time을 참조해서 캐시를 판단하는듯?

meta 정보 갱신이 잘안되면 툴에서 강제로 가능: https://developers.facebook.com/tools/debug/

페이스북에서 이미 게시글에 등록된 OG 메타정보는 수정되지 않는다.


### DNS

- www.example.com와 example.com은 같은 SSL 인증서를 사용할 수 있다
- *.example.com과 sub.example.com이 함께 정의되어 있다면 명시된 sub 도메인을 찾은 뒤, wildcard로 라우팅된다. (RFC 1912)
  - 문서내 예시는 MX지만 CANME도 동일하게 적용됨
  - RFC6126 - wildcard는 가장 왼쪽에 하나만 존재할 수 있다

