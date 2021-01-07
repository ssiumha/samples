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
```
