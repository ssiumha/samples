# infra

## terrform

```bash
terraform init

terraform refresh
# <update tf files...>
terraform plan -out plan.out | tee plan.std.out
terraform apply plan.out
```


## helm

```
helm status -n<namespace> <release_name>

helm diff upgrade <release_name> <helm_chart> --namespace <namespace> --values=values.yaml --allow-unreleased
helm diff rollback --namespace <namespace> <release_name> <revision>

helm get values <release_name> --namespace <namespace>

helm history --namespace <namespace> <release_name> --max 3

# {{ .Chart }}, {{ .Release }} 에 접근할 수 있지만 다른 템플릿 기능은 쓰지 못할 때 이용
helm template . --show-only values.yaml.tmpl > values.yaml

helm rollback --namespace data --wait kafka 99
```


## kube

- https://kubernetes.github.io/ingress-nginx/examples/auth/basic/
- https://github.com/ContainerSolutions/k8s-deployment-strategies

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
```
### Affinity

pod가 새로 뜰 때, 어떤 node를 선택할지 사용자가 임의로 룰을 지정하는 기능.

requiredDuringSchedulingIgnoredDuringExecution
preferredDuringSchedulingIgnoredDuringExecution


### Ingress

#### cert-manager wildcard certificate
- https://cert-manager.io/docs/configuration/acme/dns01/
- https://letsencrypt.org/docs/faq/#does-let-s-encrypt-issue-wildcard-certificates

#### Rewrite

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


