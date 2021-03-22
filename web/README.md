# Web

코딩말고 기타 웹 환경적인 부분들

## open graph

```
<meta property="og:title" content="title..">
```

head에 정의되는 다른 사이트에서 링크되었을 때 보여줄 스니펫 정보.
대부분의 경우 일정 시간 캐싱을 하며 참조자로 updated_time 을 사용하기도 한다.
페이스북 같은 경우 이미지 주소와 updated_time을 참조해서 캐시를 판단하는듯?

meta 정보 갱신이 잘안되면 툴에서 강제로 가능: https://developers.facebook.com/tools/debug/

페이스북에서 이미 게시글에 등록된 OG 메타정보는 수정되지 않는다.


## DNS

- www.example.com와 example.com은 같은 SSL 인증서를 사용할 수 있다


