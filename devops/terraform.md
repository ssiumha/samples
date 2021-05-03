# terrform

```bash
terraform init

terraform refresh
# 주의: 가끔 ami나 기타 update로 인해 실제 tf 수정과 무관한 변경이 발생할 수 있다
#       수정하기 전에 apply를 한번 때려보는 것도 좋다

# <update tf files...>
terraform plan -out plan.out | tee plan.std.out
terraform apply plan.out
```

## 기본 문법

### module

```
module "awesome_module_name" {
  source = "terraform-aws-modules/eks/aws"
  version = "6.0.2"

  ...
}
```

일종의 라이브러리. 특정 source를 extend 해서 여러 설정을 모아놓을 수 있다.
source는 local, github, terraform registry, s3 bucket 등등으로 설정할 수 있다.

사용된 값은 `module.awesome_module_name.cluster_id` 형태로 접근할 수 있다.


### resource

```
resource "<resource_type>" "awesome_resource_name" {
}
```

테라폼에서 사용되는 설정 자원 중 최소단위.
특정 타입에 이름을 붙여 이름에 따른 서로 다른 설정을 지정해놓을 수 있다.

`<resource_type>.awesome_resource_name` 형태로 다른 위치에서 불러들일 수 있다.
