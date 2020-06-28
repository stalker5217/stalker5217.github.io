---
title: '[Devops] Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 1'
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


## CI/CD  

서비스 기능 개발 이후에는 빌드, 테스트, 소스 병합, 릴리즈, 배포라는 일련의 과정을 거친다.
하지만 이런 작업들은 생각보다 굉장히 번거로운 작업이며 프로젝트 규모가 클수록 괴로움은 증가한다.  

CI/CD는 Continuous Integration, Continuous Deliver, Continuous Deployment를 나타내는 용어이며
위 과정들을 자동화하여 불필요한 공수를 줄이고 보다 빠른 서비스 제공을 할 수 있는 효과를 가질 수 있다.

![ci_cd](/assets/images/devops/cicd.png)


CI/CD 구축을 지원하는 툴은 생각보다 많다.
- Jenkins
- Travis CI
- Circle CI
- Google Cloud Build
- AWS CodeBuild
- ...


각 툴은 요금 체계부터 특성이 모두 다르므로 프로젝트에 적합한 것을 선택하면 된다.
이 포스팅에서는 github action을 사용하여 구축을 시도한다.

## Github Action이란?

github action은 18년에 공개되어 베타 테스트를 거쳐 19년 부터 서비스를 하고 있는 비교적 최신 기술이다. 
아래는 github에서 제공하는 개요이다.

> Automate, customize, and execute your software development workflows right in your repository with GitHub Actions. 
You can discover, create, and share actions to perform any job you'd like, including CI/CD, and combine actions in a completely customized workflow.

github action은 ci/cd만을 위한 도구는 아니고, 다양한 workflow를 작성할 수 있다고 되어있다. 
일례로 cron 식을 사용하여 batch job을 실행할 수 있는 기능을 제공한다.
또한, 이는 github 자체에서 제공하는 기능이기 때문에 다른 서드파티를 사용하는 것보다 관리해야할 포인트가 줄어든다는 이점을 가진다.


## Workflow 작성하기

![workflow_start](/assets/images/devops/workflow_start.png)

repository의 github action 탭에 들어가면 다음과 같이 repository에 있는 language를 감지하여 화면이 나타난다.
적용할 프로젝트가 spring boot & gradle 기반이므로 표기된 workflow를 선택한다.

``` yml
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
    
    - name: Build with Gradle
      run: ./gradlew clean build
```

해당 파일을 생성하고, 지정한대로 master branch에 push를 해보면 github action 탭에서 다음과 같은 내용을 확인할 수 있다.

![workflow](/assets/images/devops/workflow_result.png)

이로써 github action을 사용한 CI 구축이 완료되었다. 이어지는 포스팅에서 CD를 구축한다.