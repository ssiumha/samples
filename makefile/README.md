# Makefile

source: [Makefile](./Makefile)

- compile용 툴이지만 명령어 집합셋 관리하기 좋아서 npm에서 script 쓰듯이 사용하고 있다
- 파일 컴파일에 관한 내용은 필요해지면 그때..


## 병렬처리 주의하기

- MAKEFLAGS나 실행옵션에 `-j<n>` 을 주면 타겟단위로 병렬로 동작한다

```makefile
dependence1:
  sleep 3 && echo 1

dependence2:
  echo 2

main: dependence1
main: dependence2
  echo 3

# output>
# 2
# 1
# 3
```

- 병렬처리를 하면서 순서를 보장하려면 다음 방법들이 있다

```makefile
# 1. NOTPARALLEL 옵션으로 해결
# https://www.gnu.org/software/make/manual/html_node/Special-Targets.html#Special-Targets
.NOTPARALLEL: main


# 2. target으로 의존성을 정의하지 않음
...
main:
  $(MAKE) dependence1
  $(MAKE) dependence2
  echo 3


# 3. 의존성을 잘 구성해서 해결
dependence1:
  sleep 3 && echo 1

dependence2: dependence1
  echo 2

main: dependence2
  echo 3
```


