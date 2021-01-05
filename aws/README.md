# AWS

- 주로 aws-cli 관련 내용
- 일단 얼기설기 모아놓고 나중에 다시 사용해볼 때 갱신하기

TODO: 명령별로 필요 IAM Role 찾아두기

## Initialization

- https://console.aws.amazon.com/iam/home#/users

```bash
# version1이 필요하면 awscli@1로 설치 가능
brew install awscli

# result path: ~/.aws/credentials
aws configure

# Using docker
# 320MB를 희생하고 공식 이미지를 사용한다
# * 파이프가 전부 이미지 내부 쉘에서 실행되서 호스트 머신의 툴을 쓸 수 없다 (python, awk 정도까지만 가능)
# * amazon/aws-cli -> 공식문서에서 권장하는 이미지: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-docker.html
# * banst/awscli   -> jq가 포함된 alpine 베이스 이미지 (56MB)
# * tip: `alias aws="docker run ..."`
docker run --rm -it -v ~/.aws:/root/.aws -v $(pwd):/aws amazon/aws-cli s3 ls
```

## ECR

- https://docs.aws.amazon.com/cli/latest/reference/ecr/get-login-password.html

```bash
# Login
aws ecr get-login-password | docker login -u AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com

# describe images
aws ecr describe-images --repository-name <name> \
    | jq -r '.imageDetails[] | "\(.imagePushedAt|todate) | \(.repositoryName) | \(.imageTags|join(", ")) | \(.imageDigest)"' \
    | column -ts "|" | sort
```

## EKS

- https://ap-northeast-2.console.aws.amazon.com/eks/home?region=ap-northeast-2#/clusters

```bash
# Load config (~/.kube/config)
aws eks --region ap-northeast-2 update-kubeconfig --name <name>
kubectl config use-context arn:aws:eks:ap-northeast-2:<aws_account_id>:cluster/<name>
kubectl config view
kubectl config view --minify --flatten --context=arn:aws:eks:ap-northeast-2:xxxxxx:cluster/xxxxxx

# no using kubeconfig with AWS
# ref: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/
kubectl \
  --server $(aws eks describe-cluster --name <name> | jq -r '.cluster.endpoint') \
  --token $(aws --region ap-northeast-2 eks get-token --cluster-name <name> | jq -r '.status.token') \
  --insecure-skip-tls-verify \
  get all

# using docker image
# docker cmd 부분의 환경변수는 현재 shell 값이 평가되지만 명령은 내부에서 평가된다
# docker image 안에서 aws 포함작업 하는게 귀찮으니 환경변수로 넘기는게 편할듯
SERVER=<...>
TOKEN=<...>
docker run --rm bitnami/kubectl:1.19.6 \
  --server $SERVER --token $TOKEN --insecure-skip-tls-verify \
  get all
```

## S3

```bash
# Get bucket list
aws s3 list

# Upload recursive & public access
aws s3 cp ./src/. s3://<bucket_name>/ --recursive --acl public-read

# Cloud front remove all cache
# https://console.aws.amazon.com/cloudfront/home
aws cloudfront create-invalidation --distribution-id <distribution_id> --paths "/*"
```
