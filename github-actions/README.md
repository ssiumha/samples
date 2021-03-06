# Github Action

- source: [.github/workflows](../.github/workflows)
- workflow syntax: https://docs.github.com/en/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions#onevent_nametypes

## 공통

- run 스크립트 도중에 실패하면 이후 명령어, workflow는 실행되지 않는다
  - 하지만 post workflow는 실행된다

## 한 파일에 여러개의 trigger를 설정할 수 있다

```
on:
  pull_request:
    branches:
      - issue/*
  push:
    branches:
      - main
```

## on.push

```
on:
  push:
    branches:
      - main
    paths:
      - src/path/**/*
```

1. on.push 지정된 branches에 push가 발생했을 때 작동
2. PR이 만들어지는 것은 상관없고, PR이 머지될 때 동작한다
  - 거의 신규커밋에 대해 동작한다고 생각해도 될듯
3. on.pull_request는 fork 저장소 기준으로 deattach 된 HEAD에서 동작하지만,
  on.push는 실제 revision에서 동작한다.

  ```
  # graph
  *       717a884 20-12-23 13:41 ssiumha          Merge pull request #1 from ssiumha/test-pr-201223
  |\
  | *     477864c 20-12-23 13:40 ssiumha          pr test  (origin/test-pr-201223, test-pr-201223)
  |/
  *       7eceea5 20-12-23 13:39 ssiumha          type master -> main
  *       9ff1e5e 20-12-23 13:38 ssiumha          git push test

  # 일반적인 push: https://github.com/ssiumha/samples/actions/runs/439545836
  ==== GIT INFO ====
  * main 7eceea5 type master -> main
  origin	https://github.com/ssiumha/samples (fetch)
  origin	https://github.com/ssiumha/samples (push)
  ==================

  # PR이 머지된 경우: https://github.com/ssiumha/samples/actions/runs/439548089
  ==== GIT INFO ====
  * main 717a884 Merge pull request #1 from ssiumha/test-pr-201223
  origin	https://github.com/ssiumha/samples (fetch)
  origin	https://github.com/ssiumha/samples (push)
  ==================
  ```

4. 하나의 yml 파일을 '---'을 사용한 여러 문서로 구성하는건 불가능하다. 조건별로 파일이 나눠져야한다

## pull_request

```
on:
  pull_request:
    branches:
      - main
```

## github object examples

```
${{ github.event_name }}  # pull_request
${{ github.event.action }}  # unassigned

```

## ref 입력받기

```
on:
  workflow_dispatch:
    inputs:
      ref:
        description: ref
        required: false
jobs:
  deploy:
    runs-on: [ubuntu-18.04]
    steps:
      - if: github.event.inputs.ref == ''
        name: clone latest
        uses: actions/checkout@v2

      - if: github.event.inputs.ref != ''
        name: clone input ref
        uses: actions/checkout@v2
        with:
          ref: ${{ github.event.inputs.ref }}
```
