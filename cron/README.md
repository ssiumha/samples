
# Cron

TODO: linux에서도 같은지 점검 필요

- Good Tools
  - https://crontab.guru
  - https://crontab-generator.org


기타
- PATH가 /usr/bin:/bin 2개 뿐이라 기본적으로 system 환경 위주로 사용해야함
- 최소 단위는 1분, 설정을 잘못하면 1분씩 기다려야하니 테스트 편의상 스크립트 파일로 만드는걸 추천
- 매 타이밍마다 해당 명령을 실행시킨다. 만약 명령이 1분 이상 걸리거나 새 프로세스를 띄운다면 의도치 않는 결과를 초래할 수 있음
  - 간단하게 `[ -z "$(ps -ef | egrep '[p]rocess') | head -n1" ]` 형태로 중복실행을 막을 수도 있고 file lock을 만들어도 된다
  - `* * * * *`과 조합해서 프로세스가 없으면 무조건 띄우는 야매 데몬처럼 쓸 수도 있다


```cron
# format:
* - minute (0-59)
* - hour (0-23)
* - day of the month (1-31)
* - month (1-12 || JAN, FEB ...)
* - day of the week (0-6, 0 is Sunday || SUN, MON, TUE ...)
command - command to execute


* * * * *           # 매 1분 마다
0 * * * *           # 매시간 0분 마다
10,20,30 * * * *    # 매시간 10, 20, 30분 마다
10-20/5 * * * *     # 매시간 10분~20분 사이 5분마다 (n:10, n:15, n:20)
* 10 * * MON        # 매주 월요일 10:00~10:59 사이 1분 마다 (테스트 안해봄)
```

## Mac

- base env: macOS Catalina 10.15.6

```bash
# 저장하고 에디터를 닫으면 권한을 묻고 즉시 반영됨. service restart 등등 필요없음
crontab -e

# list up
crontab -l
```

```
# 0/1 * * * * env > ~/cron_env.txt
> OUTPUT
SHELL=/bin/sh
USER=ssiumha
PATH=/usr/bin:/bin
PWD=/Users/ssiumha
SHLVL=1
HOME=/Users/ssiumha
LOGNAME=ssiumha
_=/usr/bin/env


# 스크립트 실행
* * * * * . ~/.script/minute.sh &>> ~/.script/minute.log
```
