---
title: '[Devops] Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 3'
toc: true
toc-stick: true
search: true
categories:
  - devops
tags:
  - CI/CD
  - github action
  - Continuous Integration
  - Continuous Depolyment
  - Continuous Delivery
  - devops
  - automation
excerpt: 'github action, AWS S3, AWS CodeDeploy를 사용하여 CI/CD 환경을 구성해봅시다'
---

## AWS CodeDeploy & AWS EC2 연동하기

배포 시스템인 codedeploy를 통해 ec2 instance에 배포를 가능하게 하는 작업을 설정한다.

## EC2 설정

둘 사이의 접근을 가능하게 위해서는 AWS IAM에서 '역할 만들기'를 한다. 
사용자 등록이 AWS 외부에서부터의 접근을 허가하는 것이라면, 역할은 AWS 내에서의 접근 권한이라고 볼 수 있으며 등록 예시는 다음과 같다.  

- 다른 계정의 IAM 사용자
- AWS 리소스에서 작업을 수행해야 하는 EC2 인스턴스에서 실행 중인 애플리케이션 코드
- 계정 내 리소스에서 작업을 수행하여 기능을 제공해야 하는 AWS 서비스
- SAML을 통해 인증 연동을 사용하는 사내 디렉토리의 사용자

![iam_role_1](/assets/images/devops/github_action_ci_cd/iam_role_1.png)

<br/>

![iam_role_2](/assets/images/devops/github_action_ci_cd/iam_role_2.png)  

<br/>

![iam_role_3](/assets/images/devops/github_action_ci_cd/iam_role_3.png)  

<br/>

![iam_role_4](/assets/images/devops/github_action_ci_cd/iam_role_4.png)  

<br/>

역할을 등록하고 나서는 EC2에서 해당 역할을 사용하도록 저장한다.

![ec2_role_change](/assets/images/devops/github_action_ci_cd/ec2_role_change.png)

역할을 수정하고 인스턴스를 재부팅해준다.(인스턴스 상태 > 재부팅)

재부팅이 완료되면 EC2 Instance에서 다음 명령어를 통해 codedeploy를 설치한다.

```aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2```

정상적으로 다운로드 시 아래 메시지가 출력된다.  

```download: s3://aws-codedeploy-ap-northeast-2/latest/install to ./install```

이제 실행권한을 주고 실행을 하여 설치를 마무리한다.

```chmod +x ./install```

```sudo ./install auto```

설치 확인을 위해 다음 명령어를 입력하면

```sudo service codedeploy-agent status```

아래 메시지가 출력된다.

```The AWS CodeDeploy agent is running as PID ****```

## CodeDeploy 설정

EC2와 마찬가지로, CodeDeploy에서도 역할을 생성하고 설정해야한다.

![iam_role_5](/assets/images/devops/github_action_ci_cd/iam_role_5.png)  

codedeploy는 권한 종류가 AWSCodeDeployRole 한 가지이니 그냥 선택해서 만들면 된다.

<br/>

이제 codedeploy 서비스에서 애플리케이션 생성을 한다.

![codedeploy_1](/assets/images/devops/github_action_ci_cd/codedeploy_1.png)  

<br/>

애플리케이션을 생성하고, 배포 그룹 생성을 이어 진행한다.

![codedeploy_2](/assets/images/devops/github_action_ci_cd/codedeploy_2.png) 

<br/>

![codedeploy_3](/assets/images/devops/github_action_ci_cd/codedeploy_3.png)  
한 대의 서버이기 때문에 해당 옵션을 선택한다.

<br/>

![codedeploy_4](/assets/images/devops/github_action_ci_cd/codedeploy_4.png)  

<br/>

이제 프로젝트 폴더에 appspec.yml 파일을 생성한다. 
이 파일은 codedeploy가 서버 환경에 설치를 할 수 있도록 동작을 정의한 내용이다.

``` yml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/deploy
```

그리고, 위치할 디렉토리를 생성한다.

```
mkdir /home/ec2-user/app/deploy
```


<br/>


## github 설정

이제 github action에 deploy job을 추가한다.

```yml
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

      # appspec.yml Copy
      - name: Copy appspec
        run: cp appspec.yml ./deploy/

      # 압축파일 형태로 전달
      - name: Make zip file
        run: zip -r -qq -j ./springboot-intro-build.zip ./deploy

      # S3 Bucket으로 copy
      - name: Deliver to AWS S3
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws s3 cp \
          --region ap-northeast-2 \
          --acl private \
          ./springboot-intro-build.zip s3://springboot-intro-build/

      # Deploy
      - name: Deploy
        env:
          AWS_ACCESS_KEY_ID: ${{'{{' }} secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{'{{' }} secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws deploy create-deployment \
          --application-name springboot-intro \
          --deployment-group-name springboot-intro-deploy-group \
          --file-exists-behavior OVERWRITE \
          --s3-location bucket=springboot-intro-build,bundleType=zip,key=springboot-intro-build.zip \
          --region ap-northeast-2
```


![deploy_success](/assets/images/devops/github_action_ci_cd/deploy_success.png)  

이로써 github action에서 부터 EC2 Instance까지 파일 이관을 하는 서비스 간의 연동이 완료되었다.
이어지는 포스팅에서는 스크립트로 서비스 배포를 자동화 및 무중단배포를 구축하는 작업을 진행한다.


<br/>

참고
- 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 프리렉