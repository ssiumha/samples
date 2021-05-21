# helm

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


