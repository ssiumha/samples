# AWS

AWS 보면서 만나는 이것저것

- instance 별 가용NI, 가용IP 갯수 https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html
- 가격 계산기: https://calculator.aws/#/createCalculator/EC2

```
# 21-04-09 기준, 가격은 대략적

m5.large : 2-CPU 8-GiB Max-10-Gbps | NI-Max:3, PrivateIP-Per-NI:10
  - 예약 인스턴스 : 54$/월
  - 온디맨드      : 90$/월

m5.xlarge : 4-CPU 16-Gib
  - 예약 인스턴스 : 110$/월
  - 온디맨드      : 170$/월

gp2 - ssd | 용량당가격 | 최대처리량/볼륨 250MiB/s | 최대처리량/인스턴스 1,750MiB/s | 블록크기 16KiB
  - GB당  : 0.114$/월
  - 100GB : 11.4$/월
  - 200GB : 22.8$/월

gp3 - ssd | 사용량초과시 추가요금 |
  - GB당  : 0.0912$/월
  - 3000 IOPS까지 무료, 초과시 IOPS당 0.0057$
  - 125MB/s까지 무료, 초과시 0.0456$
  - 100GB : 9.2$/월
```
