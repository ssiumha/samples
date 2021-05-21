# kube

- https://docs.pixielabs.ai : opensource kube monitoring tool

- https://kubernetes.github.io/ingress-nginx/examples/auth/basic/
- https://github.com/ContainerSolutions/k8s-deployment-strategies

- https://codeberg.org/hjacobs/kubernetes-failure-stories : https://k8s.af
- https://ymmt.hatenablog.com/entry/k8s-things

- kube resources.limits.cpu throttling issue
  - https://speakerdeck.com/daikurosawa/understanding-cpu-throttling-in-kubernetes-to-improve-application-performance-number-k8sjp
  - https://medium.com/omio-engineering/cpu-limits-and-aggressive-throttling-in-kubernetes-c5b20bd8a718
    - kr: https://da-nika.tistory.com/192
  - https://erickhun.com/posts/kubernetes-faster-services-no-cpu-limits/

100ms 동안 얼마나 점유했는가?가 기준이니까 CPU limit에 1000ms를 넣으면 0.1초 점유가 된단건가?


```bash
# context switch 없이 다른 환경 사용하기
kubectl --context <...>

# 내부에 pod를 띄우고 DB에 붙어서 tsv로 뱉기
# kubectl exec -it mysql-console -- mysql \
kubectl run mysql-console --rm -it --restart='Never' --image mysql:5.7 -- mysql \
    --batch --compress --default-character-set=utf8 \
    -h <host> \
    -u <user_name> -p<password> <database_name> \
    -e '
        SELECT ...
        FROM ...
        LIMIT ...
     ' | tee result-$(date '+%y%m%dT%H%M%S').tsv;


# 진행한 패치 롤백, 이력
kubectl rollout undo <...>
kubectl rollout history <...>

# 내부 값으로 filter
kubectl get po -A --field-selector "spec.nodeName=ip-00-00-00-00"

# 위에꺼가 잘 안되면 jq로 필터
kubectl get po -A -ojson | jq '.items[] | select(.metadata.annotations["prometheus.io/scrape"] == "true")'


# service 대상으로 로그 보기
kubectl logs --all-containers=true -f -n<namespace> service/<service_name>


# drain으로 node 갈아치우기
#   - EKS 환경에서 kubectl delete node를 쳐도 AGS쪽 인스턴스에는 남아서 scaleup/down에 영향을 끼칠 때가 있다
#     이때는 직접 node를 terminate 시켜줘야한다
#
# cordon으로 해당 node에 pod가 못뜨게 막은 다음, drain으로 pod를 전부 재할당하고 노드를 제거한다
kubectl cordon <node/ip-00-00-00-00>
kubectl drain --ignore-daemonsets --delete-local-data <node/ip-00-00-00-00>

kubectl delete <node/ip-00-00-00-00>


# EKS All Consume IP
# https://medium.com/better-programming/amazon-eks-is-eating-my-ips-e18ea057e045
# https://ap-northeast-2.console.aws.amazon.com/vpc/home?region=ap-northeast-2
# https://github.com/aws/amazon-vpc-cni-k8s/blob/master/docs/eni-and-ip-target.md
# node가 새로 뜰 때 VPC에서 할당할 수 있는 모든 IP를 써버릴 때가 있다. 그럴때 일단 썻던거
kubectl -n kube-system set env daemonset aws-node WARM_IP_TARGET=2


# livenessProbe
# 아래와 같이 정의하면 describe로 봤을 때 `Liveness: http-get http://:probe/health` 로 표시된다.
# 하지만 실제로 livenessProbe가 작동할 때는 localhost가 아닌 pod ip에 대해 쿼리를 보내므로, ip 대역에 대한 host allow가 필요하다
# ex) config.hosts << /\A10\.\d+\.\d+\.\d+\z/
livenessProbe:
  httpGet
    path: /health
    port: probe


# 실제 사용중인 volume 가져오기
kubectl get pods -A -o=json \
  | jq -c '.items[] | {name: .metadata.name, namespace: .metadata.namespace, claimName: .spec |  select( has ("volumes") ).volumes[] | select( has ("persistentVolumeClaim") ).persistentVolumeClaim.claimName }'


# label selector
kubectl get pod -l app=redis
kubectl get pod -l app!=redis
kubectl describe node -l 'beta.kubernetes.io/instance-type in (m5.large, m5.xlarge)'

# label column
#   - 기본 정보 이외에 추가적인 label 정보를 출력시켜줌
#   - 복수개의 type에 대해 쿼리할 때 없다고 에러가 발생하지 않음
kubectl get node,ds -A -Lbeta.kubernetes.io/instance-type

# explain
# - Resource Kind의 describe 명세를 가져올 수 있다
# - 근데 거의 kind, meta, status라서 좀 미묘
kubectl explain Certificate

# pod 정보를 node 이름이랑 같이 가져오기
kubectl get pod -A -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName


# get raw
## etcd healthcheck
kubectl get --raw=/healthz/etcd

## query api server
kubectl get --raw=/apis
kubectl get --raw=/logs/kube-apiserver.log
```


## Affinity

- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
- https://cstoku.dev/posts/2018/k8sdojo-18/#topology-key

스케쥴러가 새 pod를 띄울 때 어디에 띄울지를 결정하는 룰을 지정한다.

- 만약 스케쥴링 되는 resource.request보다 node의 CPU, 메모리가 적다면 먼저 제외하고 처리될듯?
  - 혹은 require로 설정되어있다면 node 안에 있는 pod를 부족한 리소스만큼 이전시킬지도 모르겠다
- 스케쥴할 때 특정 정보를 보고 선택/제외시킬 수 있다 (podAffinity, podAntiAffinity, nodeAffinity)
- 현재 떠있는 pod를 조건으로 걸거나 node의 label을 조건으로 걸 수 있다 (pod affinity, node affinity)
- 가중치를 기준으로 러프하게 계산하거나 필수 조건으로 만들 수도 있다 (preferred, required)

- preferred는 규칙에 맞을 때마다 가중치를 더하고, 가장 높은 가중치 node에 스케쥴링한다
  - 규칙마다 weight 필드로 가중치를 지정할 수 있다 (1~100)
- matchExpressions에서 사용가능한 operator
  - 일치하는 값이 있는지: In, NotIn
  - 값의 유무: Exists, DoesNotExists
- topologyKey
  - node의 label을 이용해서 만드는 Affinity 규칙 적용 '범위'
  - topologyKey로 사용가능한 node의 label
    - hostanme
    - zone - ex) zone 마다 하나씩 뜨도록
    - type - ex) 특정 type에 대해서만 뜨도록


```
# beta::app=memcached pod가 있는 node에만 스케쥴링한다 (강제)
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - topologyKey: kubernetes.io/hostname
      namespaces: beta
      labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
            - memcached

# app=kafka가 없는 node에만 스케쥴링한다 (강제)
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - topologyKey: kubernetes.io/hostname
        labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - kafka
```

## Ingress

### cert-manager wildcard certificate
- https://cert-manager.io/docs/configuration/acme/dns01/
- https://letsencrypt.org/docs/faq/#does-let-s-encrypt-issue-wildcard-certificates

### Rewrite

- https://kubernetes.github.io/ingress-nginx/examples/rewrite/
- https://www.nginx.com/blog/creating-nginx-rewrite-rules/
- https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/

test.example.com/foo 를 www.example.com/foo 로 redirect 시키는 설정 (host 변경)


```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-example-com
  namespace: default
  annotations:
    kubernetes.io/ingress.class: http-nginx-ingress
    nginx.ingress.kubernetes.io/server-snippet: |
      rewrite ^ https://www.example.com$request_uri redirect;

# - redirect(302) 대신 permanent(301)를 사용하면 브라우저에 캐시가 남아서 변경 대응이 힘들 수 있음
# - 특정 path에 대해서만 동작시키고 싶은 경우
#     rewrite ^/tabs/.*? https://www.example.com$request_uri redirect;

spec:
  tls:
    - hosts:
        - test.example.com
      secretName: test-example-com-tls
  rules:
    - host: test.example.com
      http:
        paths:
          - path: /
            backend:
              serviceName: test-web-server-service
              servicePort: 8080

---

apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: test-example-com-tls
  namespace: default
spec:
  secretName: test-example-com-tls
  duration: 2160h # 90d
  renewBefore: 360h # 15d
  commonName: test-example-com-tls
  dnsNames:
    - test.example.com
  issuerRef:
    name: letsencrypt-production
    kind: ClusterIssuer
```
