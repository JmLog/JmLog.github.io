---
title: "[DevOps] CI/CD를 활용하여 자동배포 구축하기"
layout: single
categories: devops
tag: ['devops', 'ci/cd']
toc: true
toc_label: "목차"
toc_icon: "fas fa-list-ul"
author_profile: false
sidebar:
    nav: "docs"
---

**CI(Continuous Integration) - 지속적인 통합 / CD(Continuous Deployment) - 지속적인 배포**에 대해 알아보겠습니다.

개발자가 매번 코드를 테스트하고 배포한다면 그것만으로도 상당한 시간이 필요합니다. 이를 대체해주는 기능이 존재한다면 개발자가 다른 곳에 쓰는 시간보다 개발에 집중하는 시간이 더 길어지고 효율적으로 코드를 구성할 수 있을 것입니다.

자동 배포 기능을 위해 `Aws`에 `CodeDeploy`를 활용합니다.

## IAM 역할 생성하기

<p style="color:rgba(68, 131, 97, 1); font-style:italic;"> 역할 > 역할 생성</p>

![devops_cicd_1](/assets/images/devops/cicd/devops_cicd_1.png)

일반 사용 사례를 `EC2`로 지정하고 다음으로 넘어갑니다.

![devops_cicd_2](/assets/images/devops/cicd/devops_cicd_2.png)

![devops_cicd_3](/assets/images/devops/cicd/devops_cicd_3.png)

위와같이 정책 2개를 선택하여 `IAM`을 만들어줍니다.

- AmazonS3FullAccess
- AWSCodeDeployFullAccess

## EC2에 CodeDeploy Agent를 설치합니다.

저는 `Amazon Linux`를 사용중임으로 아래와 같이 설치합니다.

```php
sudo yum update
sudo yum install ruby
sudo yum install wget
cd /home/ec2-user
wget https://bucket-name.s3.region-identifier.amazonaws.com/latest/install
chmod +x ./install
//# 최신 버전의 CodeDeploy를 설치하려면 아래를 수행합니다.
sudo ./install auto
```

설치를 마첬으면 `CodeDeploy Agent`가 실행중인지 확인합니다.

```php
sudo service codedeploy-agent status
```

자세할 설치 가이드는 아래 경로를 참고해주세요.

[Amazon Linux 또는 RHEL용 CodeDeploy 에이전트 설치 - AWS CodeDeploy](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/codedeploy-agent-operations-install-linux.html)

## ****CodeDeploy 애플리케이션 생성****

<p style="color:rgba(68, 131, 97, 1); font-style:italic;">CodeDeploy > 애플리케이션 > 애플리케이션 생성</p>

![devops_cicd_4](/assets/images/devops/cicd/devops_cicd_4.png)

## CodeDeploy Group 생성

<p style="color:rgba(68, 131, 97, 1); font-style:italic;">CodeDeploy > 애플리케이션 > Deploy > 배포 그룹 생성</p>

![devops_cicd_5](/assets/images/devops/cicd/devops_cicd_5.png)

배포 그룹 이름을 정하고 서비스 역할에 위에서 생성해준 역할을 선택합니다.

![devops_cicd_6](/assets/images/devops/cicd/devops_cicd_6.png)

서버가 이중화가 되어있지 않고 테스트 배포임으로 배포 유형은 현재위치를 선택 해줍니다.
`EC2` 인스턴스를 체크하고 배포하려 하는 `EC2`를 선택합니다.
아래쪽 로드밸런서 활성화는 체크를 풀어주고 “배포 그룹 생성”을 눌러주세요!

여기까지 `CodeDeploy` 생성은 끝났습니다.

## Github Action

애플리케이션 제작중인 소스를 `AWS S3`에 알집으로 올려서 `CodeAgent`를 통해 EC2에 배포를 진행합니다.
일련의 과정을 위해 `workflow`를 작성합니다.

![devops_cicd_7](/assets/images/devops/cicd/devops_cicd_7.png)

새로운 `workflow`를 만들고 아래와 같이 작성합니다.

```php
name: deploy

on:
  push:
    branches: [ dev ]

  workflow_dispatch:

env:
  S3_BUCKET_NAME: "생성한 S3버킷 이름"
  AWS_REGION: ap-northeast-2
  CODEDEPLOY_NAME: "생성한 CodeDeploy 이름"
  CODEDEPLOY_GROUP: "생성한 CodeDeployGroup 이름"

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Make zip file
        run: zip -r ./$GITHUB_SHA.zip .
        shell: bash

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.ACCESS_KEY }}
          aws-secret-access-key: ${{ secrets.SECRETS_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Upload to S3
        run: aws s3 cp --region ${{ env.AWS_REGION }} ./$GITHUB_SHA.zip s3://$S3_BUCKET_NAME/$GITHUB_SHA.zip

      - name: Code Deploy
        run: aws deploy create-deployment --application-name $CODEDEPLOY_NAME --deployment-config-name CodeDeployDefault.AllAtOnce --deployment-group-name $CODEDEPLOY_GROUP --s3-location bucket=$S3_BUCKET_NAME,bundleType=zip,key=$GITHUB_SHA.zip
```

`Github`에 예민한 정보가 다른 사용자에게 유출될 가능성이 있어서 `Github`에서 제공하는 `Secrets`으로 설정하는게 좋습니다.

- <p style="color:rgba(68, 131, 97, 1); font-style:italic;">Setting > Secrets and variables > Action</p>

`CI/CD` 기능을 모두 설정했습니다. 이제 설정 해놓은 대로 “dev”브랜치에 푸쉬를 하면 자동으로 `CodeDeploy`가 `EC2`에 배포를 실행합니다.

![devops_cicd_8](/assets/images/devops/cicd/devops_cicd_8.png)

이것으로 포스팅을 마치겠습니다. 감사합니다!