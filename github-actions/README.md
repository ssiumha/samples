# Github Action

- source: [.github/workflows](../.github/workflows)
- workflow syntax: https://docs.github.com/en/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions#onevent_nametypes

## on.push

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
