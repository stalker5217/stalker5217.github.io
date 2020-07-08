---
title: '[Devops] Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 4'
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

본 포스팅은 '스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱'의 내용을 기반으로 작성한 내용입니다.  

[Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 1](https://stalker5217.github.io/devops/github_action_ci_cd_1/)  
[Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 2](https://stalker5217.github.io/devops/github_action_ci_cd_2/)  
[Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 3](https://stalker5217.github.io/devops/github_action_ci_cd_3/)  
[Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 4](https://stalker5217.github.io/devops/github_action_ci_cd_4/)  

## 무중단 배포를 위한 nginx  

마지막으로 설정할 내용은 새로운 서비스를 올릴 때 발생하는 서비스 중단 시간을 해결하기 위한 무중단 배포 설정과 배포 자동화이다.

무중단 설정은 nginx의 reverse proxy 기능을 사용하여 진행할 것이고 Blue-Green 배포 전략을 사용한다.
해당 포스팅은 2개의 서비스를 각각 8081, 8082 포트를 사용하여 진행할 것이다.

> Blud-Green Deployments
완전히 동일한 두 개의 운영환경(Blue, Green)이 필요하다. 현재 운영 중인 서비스가 Blue 환경이라고 했을 때, 신규 배포는 Green 환경에 배포한다.
Green 환경에 배포가 정상적으로 이루어졌다면 모든 트래픽은 이제 Green 환경으로 향하게 된다.  
배포 이 후 문제가 생겼다면 트래픽을 모두 Blue로 돌리면 빠른 rollback도 가능하다.

<br/>

## nginx 설정

먼저, nginx를 통한 접근이 가능하도록 EC2의 보안그룹에 80 포트를 추가해준다.

![nginx_1](/assets/images/devops/github_action_ci_cd/nginx_1.png)


EC2에서 nginx를 설치하고 실행해 준다.

```sudo yum install nginx```


http(80), https(443)으로 접속하면 nginx에서 서비스가 올라간 8080포트로 서비스를 전달하도록 하기 위해 아래 설정을 한다.

```sudo vim /etc/nginx/conf.d/service-url.inc```로 파일 생성

```
# service-url.inc
# 아래 파일로 nginx가 바라볼 포트를 지정한다.
# 내용은 배포시 마다 실행되는 scripts로 동적으로 변경된다.

set $service_url http://127.0.0.1:8081;
```


``` sudo vim /etc/nginx/nginx.conf ```로 설정 파일 수정

``` conf
http {
    server {
        include /etc/nginx/conf.d/service-url.inc;

        location / {
                # service_url로 요청 전달!
                proxy_pass $service_url;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }
    }
}
```

설정을 마친 후 nginx를 재시작 해준다.  

```sudo service nginx start```


## 프로젝트 설정

nginx와 연결할 port를 정하는 API를 작성한다.

``` java
@RequiredArgsConstructor
@RestController
public class ProfileController {
    private final Environment env;

    @GetMapping("/profile")
    public String profile() {
        List<String> profiles = Arrays.asList(env.getActiveProfiles());
        List<String> realProfiles = Arrays.asList("real1", "real2");
        String defaultProfile = profiles.isEmpty()? "default" : profiles.get(0);

        // real, real1, real2 중 하나라도 있으면 그 값 반환
        return profiles.stream()
                .filter(realProfiles::contains)
                .findAny()
                .orElse(defaultProfile);
    }
}
```
그리고 real1, real2에 8081, 8082 포트가 할당되도록 application.properties 파일을 작성한다.

``` properties
# application-real1.properties
server.port=8081
...
```

``` properties
# application-real2.properties
server.port=8081
...
```

## 배포 스크립트 작성

``` sh
#!/usr/bin/env bash

# profile.sh
# 미사용 중인 profile을 잡는다.

function find_idle_profile()
{
    # curl 결과로 연결할 서비스 결정
    RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)

    if [ ${RESPONSE_CODE} -ge 400 ] # 400 보다 크면 (즉, 40x/50x 에러 모두 포함)
    then
        CURRENT_PROFILE=real2
    else
        CURRENT_PROFILE=$(curl -s http://localhost/profile)
    fi

    # IDLE_PROFILE : nginx와 연결되지 않은 profile
    if [ ${CURRENT_PROFILE} == real1 ]
    then
      IDLE_PROFILE=real2
    else
      IDLE_PROFILE=real1
    fi

    # bash script는 값의 반환이 안된다.
    # echo로 결과 출력 후, 그 값을 잡아서 사용한다.
    echo "${IDLE_PROFILE}"
}

# 쉬고 있는 profile의 port 찾기
function find_idle_port()
{
    IDLE_PROFILE=$(find_idle_profile)

    if [ ${IDLE_PROFILE} == real1 ]
    then
      echo "8081"
    else
      echo "8082"
    fi
}
```

``` sh
# !/usr/bin/env bash

# stop.sh
# 서버 중단을 위한 스크립트

ABSPATH=$(readlink -f $0)
# ABSDIR : 현재 stop.sh 파일 위치의 경로
ABSDIR=$(dirname $ABSPATH)
# import profile.sh
source ${ABSDIR}/profile.sh

IDLE_PORT=$(find_idle_port)

echo "> $IDLE_PORT 에서 구동중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

if [ -z ${IDLE_PID} ]
then
  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 ${IDLE_PID}
  sleep 5
fi
```


``` sh
#!/usr/bin/env bash

# start.sh
# 서버 구동을 위한 스크립트

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

REPOSITORY=/home/ec2-user/app
PROJECT_NAME=springboot-intro

echo "> Build 파일 복사"
echo "> cp $REPOSITORY/deploy/*.jar $REPOSITORY/"

cp $REPOSITORY/deloy/*.jar $REPOSITORY/

echo "> 새 어플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

IDLE_PROFILE=$(find_idle_profile)

echo "> $JAR_NAME 를 profile=$IDLE_PROFILE 로 실행합니다."
nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-$IDLE_PROFILE.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=$IDLE_PROFILE \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

``` sh
#!/usr/bin/env bash

# health.sh
# nginx 연결 설정 변경 전 health-check 용도

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh

IDLE_PORT=$(find_idle_port)

echo "> Health Check Start!"
echo "> IDLE_PORT: $IDLE_PORT"
echo "> curl -s http://localhost:$IDLE_PORT/profile "
sleep 10

for RETRY_COUNT in {1..10}
do
  RESPONSE=$(curl -s http://localhost:${IDLE_PORT}/profile)
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)

  if [ ${UP_COUNT} -ge 1 ]
  then # $up_count >= 1 ("real" 문자열이 있는지 검증)
      echo "> Health check 성공"
      switch_proxy
      break
  else
      echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
      echo "> Health check: ${RESPONSE}"
  fi

  if [ ${RETRY_COUNT} -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi

  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
```

``` sh
#!/usr/bin/env bash

# switch.sh
# nginx 연결 설정 스위치

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

function switch_proxy() {
    IDLE_PORT=$(find_idle_port)

    echo "> 전환할 Port: $IDLE_PORT"
    echo "> Port 전환"
    # nginx와 연결한 주소 생성
    # | sudo tee ~ : 앞에서 넘긴 문장을 service-url.inc에 덮어씀
    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc

    echo "> 엔진엑스 Reload"
    # nignx reload. restart와는 다르게 설정 값만 불러옴
    sudo service nginx reload
}
```

## appspec.yml 설정

codedeploy 실행 시 작성한 스크립트를 동작시킨다.

``` yml
# appspec.yml

version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/deploy

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  AfterInstall:
    - location: stop.sh # nginx와 연결되지 않은 스프링 부트 종료.
      timeout: 60
      runas: ec2-user
  ApplicationStart:
    - location: start.sh # nginx와 연결되어 있지 않은 포트로 스프링 부트 시작.
      timeout: 60
      runas: ec2-user
  ValidateService:
    - location: health.sh # 새 서비스 health check.
      timeout: 60
      runas: ec2-user
```



## github 설정

배포에 필요한 파일들을 모두 담도록 workflow를 수정한다.

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
        run: cp ./appspec.yml ./deploy/

      # script file Copy
      - name: Copy shell
        run: cp ./scripts/* ./deploy/

      # 압축파일 형태로 전달
      - name: Make zip file
        run: zip -r -qq -j ./springboot-intro-build.zip ./deploy/

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
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws deploy create-deployment \
          --application-name springboot-intro \
          --deployment-group-name springboot-intro-deploy-group \
          --file-exists-behavior OVERWRITE \
          --s3-location bucket=springboot-intro-build,bundleType=zip,key=springboot-intro-build.zip \
          --region ap-northeast-2
```

2개의 서비스가 가동 중인 것을 확인할 수 있으며, 하나는 실제 서비스 하나는 무중단배포를 위한 유휴서비스이다.  


![non_stop_deployment](/assets/images/devops/github_action_ci_cd/non_stop_deployment.png)


<br/>

참고
- 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 프리렉