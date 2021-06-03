# Docker

local 개발환경 구축할 때 삽질하기 쉬운 내용 모아두기


```
docker run -it --network=host edenhill/kafkacat:1.6.0 -b YOUR_BROKER -L
```

## dockerfile

### FROM

한 파일 안에 여러 FROM을 사용하면 FROM 단위로 멀티스테이지 구성이 된다.

FROM에 ARG 변수를 쓸 수 있는데, 이를 활용해서 arg를 넘겼을 때 빌드없이 registry에서 받아오게 하는 것도 가능

```
ARG IMAGE_NAME=img1

FROM registry-url/img:latest as img1
...

FROM $IMAGE_NAME as img2
...
```

- IMAGE_NAME이 registry-url/img:latest고 target이 img2 면 무조건 registry에서 다운받아옴
- IMAGE_NAME이 없으면 img1을 빌드하고 img2를 사용

### ONBUILD

```
ONBUILD RUN ...
ONBUILD COPY ...
```

처음 적힌 FROM 이미지에서는 실행되지 않지만, 만약 ONBUILD가 정의된 이미지를 FROM으로 가져와 사용하면
그 때 작동하기 시작하는 키워드.

상속처럼 사용할 수 있고 buildstage 사이에 중복되는 내용을 최소화하는데 쓸 수도 있을듯

### ADD

```
ADD <url|path> /app

# github 저장소 파일을 바로 image에 포함시키기
ADD https://github.com/<account>/<reponame>/archive/master.tar.gz /app
```

- COPY랑 다르게 URL에서 다운로드 받거나 tar.gz를 자동으로 압축해제하는 기능이 포함되어있다
- 일종의 사이드이펙트가 있는 기능이므로 필요한게 아니면 COPY를 쓰는게 좋을듯

### cache

cache는 기본적으로 명령 한줄 단위로 레이어가 구성되며
copy, add 같은 경우 파일의 checksum을 통해 캐시된 레이어를 사용할지 말지를 결정한다.

1. 거의 변화가 없는 runtime install
2. 자주 업데이트 안되고 무게가 큰 package.json, Gemfile 등의 copy & install
3. 변경이 자주 되는 실제 소스 파일

순으로 배치함으로써 캐시의 활용성을 높일 수 있다.

### env

```
ENV A=FOO B=BAR
```

기본적으로 설정될 환경변수, 외부에서 다른 환경변수를 받아올 수도 있다.
환경변수가 바뀌면 그 다음 명령부터 새로운 레이어를 만든다.

### cmd & entrypoint

docker image의 기본 실행 명령.

entrypoint가 좀 더 우선도가 높고, 기본 명령어 취급이며 양쪽다 사용할 경우
cmd는 entrypoint 뒤에 덧붙여지게 된다.

```
CMD ["World"]
ENTRYPOINT ["echo", "Hello"]
#> Hello World
```

docker run 이미지 이름 다음에 적는 인자는 전부 cmd로 들어가며 entrypoint를 덮어쓰려면
`docker run --entrypoint='...'` 형태로 별도 인자를 줘야한다.


## DOCKER_BUILDKIT
- https://github.com/moby/buildkit
- https://docs.docker.com/develop/develop-images/build_enhancements/

```
DOCKER_BUILDKIT=1 docker build --help
```

docker 18.09부터 buildkit 환경 변수를 설정해서 향상된 도커 빌드 시스템을 사용할 수 있다.
(`docker build --help`와 비교해보면 옵션도 다르다)

- 필요없는 빌드 스테이지 스킵
- 병렬 빌드
- 빌드간 변경된 파일만 전송
- 새 버전의 Dockerfile 구현체 사용
- 등등

## full template

```
gtar -X .dockerignore -czh . \
  | DOCKER_BUILDKIT=1 docker build --progress=plain \
    --file docker/Dockerfile \
    --target 0 \
    --build-args ARG_NAME1=fooooooooooo \
    --build-args ARG_NAME2=barrrrrrrrrr \
    --tag awesome-name:tag1 \
    --tag awesome-name:latest \
    -

```

## access to host

network가 bridge일 떄, host.docker.internal 로 host에 접근이 가능하다


## inspect

container 정보를 얻어오는 기능, json query 예시 등등

```
# container gateway IP 가져오기
# host.docker.internal 을 사용할 수 없는 상황에서 사용?
docker inspect -f "{{ .NetworkSettings.Gateway }}" "$CONTAINER_NAME"

# src_port를 지정하지 않았을 때 (docker ... --publish 61234 ), docker에서 임의로 할당한 포트 가져오기
docker inspect -f "{{(index (index .NetworkSettings.Ports \"$PORT/tcp\") 0).HostPort}}" "$CONTAINER_NAME"
```

## prune

더 이상 사용안하는 이미지, 캐시 등을 삭제

```
# https://docs.docker.com/engine/reference/commandline/system_prune/
# container, network, dangling images, build cache
docker sysetm prune

# volume 포함
docker sysetm prune --volumes

# volume만
docker volume prune

# crontab에서 쓰자
0 9 * * * docker system prune -f --volumes 2>&1 > /tmp/docker-system-prune
```


## gosu

```
exec gosu user bundle exec rails server
```

container 환경에서 쓰기 복잡한 유저, 권한 설정과 su, sudo 사용을 대체하기 위해 만들어진 툴.
특정 프로세스를 특정 권한으로 제한하기 위해 사용된다.


## export data

```
mkdir -p $(EXPORT_PATH)
docker run --rm -i \
  -v $(EXPORT_PATH):/export \
  --entrypoint bash \
  <IMAGE_NAME> \
  -c 'cp -r /build_result /export'
ls $(EXPORT_PATH)
```

주의: github action에서 쓰려고 하면 volume mount가 host 기준으로 되버리므로 cp를 써야한다


```
docker cp $(docker create --rm registry.example.com/ansible-base:latest):/home/ansible/.ssh/id_rsa ./hacked_ssh_key

docker create -ti --name dummy IMAGE_NAME bash
docker cp dummy:/path/to/file /dest/to/file
docker rm -f dummy
```
