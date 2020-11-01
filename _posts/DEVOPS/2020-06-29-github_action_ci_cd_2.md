---
title: '[Devops] Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 2'
toc: true
toc-stick: true
search: true
categories:
  - devops
tags:
  - CI/CD
  - github action
  - Continuous Integration
  - Continuous Deployment
  - Continuous Delivery
  - devops
  - automation
  - AWS
excerpt: 'github action, AWS S3, AWS CodeDeploy, AWS EC2를 사용하여 CI/CD 환경을 구성해봅시다'
---

본 포스팅은 '스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱'의 내용을 기반으로 작성한 내용입니다.  

[Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 1](https://stalker5217.github.io/devops/github_action_ci_cd_1/)  
[Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 2](https://stalker5217.github.io/devops/github_action_ci_cd_2/)  
[Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 3](https://stalker5217.github.io/devops/github_action_ci_cd_3/)  
[Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 4](https://stalker5217.github.io/devops/github_action_ci_cd_4/)   

## AWS S3 연동하기

AWS S3는 파일 서버 역할을 한다. github action에서 빌드한 결과를 저장해 놓는 기능을 할 예정이며 flow는 다음과 같다.

![cicd_flow](/assets/images/devops/github_action_ci_cd/cicd_flow.png)


AWS는 외부 서비스의 접근을 기본적으로 허용하지 않는다. 따라서 IAM(Identify & Access Management)를 사용하여 접근 권한을 획득해야 한다.

## IAM 설정

AWS IAM에서 사용자 그룹에서 '사용자 추가'를 한다. 여기서 사용자란 AWS 외부에서 접속할 수 있는 권한을 줄 대상을 말한다.

![iam_user_1](/assets/images/devops/github_action_ci_cd/iam_user_1.png)

<br/>

![iam_user_2](/assets/images/devops/github_action_ci_cd/iam_user_2.png)

해당 페이지에서 'AmazonS3FullAccess', 'AWSCodeDepolyFullAccess' 두 가지 요소를 활성화 시킨다.

<br/>

![iam_user_3](/assets/images/devops/github_action_ci_cd/iam_user_3.png)

<br/>

![iam_user_4](/assets/images/devops/github_action_ci_cd/iam_user_4.png)

<br/>

그러면 해당 권한을 가진 사용자가 생성되고 ACCESS_KEY와 SECRET_KEY가 발급이 된다.  

권한 설정을 했으니 이제 실제로 S3에서 저장될 버킷을 생성한다.

## S3 Bucket 설정

S3에서 '버킷 만들기'를 한다. 3단계 권한 설정에서 모든 퍼블릭 액세스 차단 이외에는 디폴트 옵션을 사용하였다.

![create_s3_bucket](/assets/images/devops/github_action_ci_cd/create_s3_bucket.png)


## github 설정

이제 github repository에서 해당 키 값들을 세팅한다.
action에 키 값을 그대로 노출하는 것은 보안상 문제가 되므로 repository의 'Settings -> Secret'에서 해당 키 값을 등록해준다.

![github_secret](/assets/images/devops/github_action_ci_cd/github_secret.png)

workflow에 S3로 파일을 옮기는 job을 추가해 준다.

``` yaml
# This workflow will build a Java project with Gradle
# For more information see: https://help.github.com/actions/language-and-framework-guides/building-and-testing-java-with-gradle

# Repo Action 페이지에 나타날 이름 
name: Spring Boot & Gradle CI/CD

# Event Trigger
# master branch에 push 또는 pull request가 발생할 경우 동작
# branch 단위 외에도, tag나 cron 식 등을 사용할 수 있음
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    # 실행 환경 지정
    runs-on: ubuntu-latest

    # Task의 sequence를 명시한다.
    steps:
      - uses: actions/checkout@v2

      - name: Set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      # Build
      - name: Build with Gradle
        run: ./gradlew clean build

      # 전송할 파일을 담을 디렉토리 생성
      - name: Make Directory for deliver
        run: mkdir deploy

      # Jar 파일 Copy
      - name: Copy Jar
        run: cp ./build/libs/*.jar ./deploy/

      # 압축파일 형태로 전달
      - name: Make zip file
        run: zip -r -qq -j ./springboot-intro-build.zip ./deploy

      # S3 Bucket으로 copy
      - name: Deliver to AWS S3
        env:
          AWS_ACCESS_KEY_ID: ${{'{{' }} secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{'{{' }} secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws s3 cp \
          --region ap-northeast-2 \
          --acl private \ 
	  ./springboot-intro-build.zip s3://springboot-intro-build/


```

이제 workflow 동작은 프로젝트 파일을 압축하고 S3로 파일을 이관하는 작업이 추가되었다.

![workflow_result](/assets/images/devops/github_action_ci_cd/workflow_result2.png)  
![aws_s3_deliver_result](/assets/images/devops/github_action_ci_cd/aws_s3_deliver_result.png)

이로써 github action을 사용한 S3 Bucket 파일 이관이 완료되었다. 
이어지는 포스팅에서는 CodeDeploy를 통해 EC2에 배포하는 작업을 진행한다.

<br/>

참고
- 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 프리렉