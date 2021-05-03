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

## Load Balancer

- https://aws.amazon.com/ko/elasticloadbalancing/features/
- https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html#network-load-balancer-components
- https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html

- kube
  - https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/guide/targetgroupbinding/targetgroupbinding/
  - https://github.com/kubernetes/ingress-nginx/blob/master/charts/ingress-nginx/values.yaml
- proxy protocol 관련
  - https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt
  - https://docs.nginx.com/nginx/admin-guide/load-balancer/using-proxy-protocol/
  - https://www.ergton.com/nginx-ingress-nlb-with-proxy-protocol-v2-deployed-to-eks.html

ALB, NLB, GLB, CLB(deprecated) 4종류가 존재한다. (21.04 기준)
대충 features 참고하면 되고 NLB를 eks로 띄우는 작업 때문에 그쪽 위주로 일단 추가.


```
ky -nhelm svc/nginx-ingress-controller -ojson | jq -r '.status.loadBalancer.ingress[0].hostname'

aws elbv2 wait load-balancer-available

aws elbv2 describe-load-balancers --names xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
  | jq -r '.LoadBalancers[0].LoadBalancerArn'

aws elbv2 describe-target-groups \
  --load-balancer-arn arn:aws:elasticloadbalancing:ap-northeast-2:000000000000:loadbalancer/net/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/xxxxxxxxxxxxxxxx

aws elbv2 modify-target-group-attributes \
  --attributes Key=proxy_protocol_v2.enabled,Value=true
  --target-group-arn arn:aws:elasticloadbalancing:ap-northeast-2:000000000000:targetgroup/xxxxxxxxxxxxxxxxxxxxxxxxxxxx/xxxxxxxxxxxxxxxx
```

### NLB

- 기본적으로 proxy protocol v2가 적용되어있고 STMP 같은 비HTTP, TCP 환경에서도 Source IP를 알아낼 수 있다?
- Global Accelerator를 기능없이, ELB 중 유일하게 Static IP, Elastic IP 할당가능
