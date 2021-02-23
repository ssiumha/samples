# bash

## if condition

```bash
# condition exit code
run command;
if [ $? -ne 0 ]; then
  echo 'run failed';
fi

if ! run command; then
  echo 'run failed';
fi


# [ vs [[
# [ : 빌트인 커맨드, `/bin/[`
#   - [ ] 사이에 작성되는 모든 값은 인자 단위로 취급된다
# [[ : 쉘 문법
# TODO 환경변수와 리터럴 문자열을 사용한 예제 필요
```

## read

```bash
# $REPLY 변수로 입력을 받아올 수 있다
read
if [ "$REPLY" != "y" ]
then
    echo "pass.."
    exit 0
fi

# while이랑 쓰기
<...> | while read -r field1 field2; do
  echo $field1 $field2;
done
```


## watch

- watch는 옵션을 제외한 모든 입력을 문자열로 받아서 실행한다
- pipline을 문자열로 변환해서 넘길 수 있다
- 대신 `watch awk "{ print 1 }"` 같은 경우도 `watch awk { print 1 }`로 치환되서 실행되니 주의가 필요

```bash
watch kubectl get po -A '|' grep -v Running

watch --color -- kubectl get all -A '|' awk "' \
  /staging/{ printf \"\033[32m\" }; \
  /production/{ printf \"\033[33m\" }; \
  1; \
  { printf \"\033[0m\"} \
'"
```


## git

```bash
# 히스토리 로그 검색 (특정파일 제외)
git log --patch --stat --grep xxxxxx -- . ":(exclude)*.json"
git log --patch --stat --grep xxxxxx -- . ":(exclude)*/specific-filename.yaml"

# 히스토리 패치 grep
git log --patch -G 'ASDFASDF'

# 지워진 파일만 보기
git log --patch --diff-filter=D
```


## get public ip

```bash
dig +short myip.opendns.com @resolver1.opendns.com
curl https://checkip.amazonaws.com

curl https://api.ipify.org
curl https://api.ipify.org?format=json # json format: {"ip":"xxx.xxx.xxx.xxx"}
curl https://api64.ipify.org  # IPv6
```
