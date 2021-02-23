# infra

## terrform

```bash
terraform init

terraform refresh
# <update tf files...>
terraform plan -out plan.out | tee plan.std.out
terraform apply plan.out
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
```

## helm

```
helm status -n<namespace> <release_name>

helm diff upgrade <release_name> <helm_chart> -n<namespace> --values=values.yaml --allow-unreleased

helm get values <release_name> -n<namespace>

helm history -n<namespace> <release_name> --max 3
```
