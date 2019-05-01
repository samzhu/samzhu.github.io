---
title: 設定 GitLab Runner 進行打包作業
categories:
  - DevOps
tags:
  - GitLab
  - Docker
  - CI
---
設定好 Runner 後, 我們就來準備 CI 的腳本啦  
我手邊都是 Java 專案, 就以 Maven 打包來說明吧

<!--more-->

首先看我們完整的 .gitlab-ci.yml, 再分解說明

``` yml
stages:
  - pre_build
  - maven_build
  - docker_build
  - push_dev
  - push_public_release

pre-build:
  image: centos:7
  stage: pre_build
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
  before_script:
    - . ci-template/shell/ci-function.sh
    - get_project_info
  script:
    - write_to_env_file
    - write_to_build_vars
  artifacts:
    paths:
      - .env
      - build-vars.sh

maven-build:
  image: maven:3.6.0-jdk-8-alpine
  stage: maven_build
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
    MAVEN_OPTS: "-Djava.awt.headless=true -Dmaven.repo.local=.m2/repository"
    MAVEN_CLI_OPTS: "-s .m2/settings.xml -B "
  before_script:
    - source build-vars.sh
    - mkdir -p .m2
    - cp ci-template/maven/settings.xml ./.m2/settings.xml
    - rm -f src/main/resources/bootstrap*.yml
    - |
      sed "s/    name: APPLICATION_NAME/    name: ${PROJECT_NAME}/g" ci-template/config/bootstrap_template.yml > src/main/resources/bootstrap.yml
    - rm -f src/main/resources/logback-spring.xml && rm -f src/main/resources/logback.xml
    - cp ci-template/config/logback-spring.xml src/main/resources/logback-spring.xml
    - ls src/main/resources/
    - cat src/main/resources/bootstrap.yml
  #cache:
  #  paths:
  #    - .m2/repository
  script:
    - mvn $MAVEN_CLI_OPTS package
  artifacts:
    paths:
      - target/*.jar

# 日常打包使用, 必推地端
# https://git.ini.usc.edu/help/ci/yaml/README.md#git-submodule-strategy
# https://docs.gitlab.com/ce/ci/git_submodules.html#configuring-the-gitmodules-file
docker-build:
  image: docker:latest
  stage: docker_build
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
  before_script:
    - source build-vars.sh
    - cp ci-template/docker/Dockerfile ./Dockerfile
    - export DOCKER_IMAGE_NAME=${BUILD_IMAGE_NAME}
    - export DOCKER_IMAGE_TAG=${BUILD_IMAGE_TAG}
    - build_arg=$(cat .env | sed -e '/^#/d' -e '/^\s*$/d' | awk -F= '{print "--build-arg",$0 }' | sed -e ':a;N;$!ba;s/\n/ /g')
  script:
    - docker build ${build_arg} -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} --rm=true .
    - docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_HUB_CSNT}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
    - docker login -u ${DOCKER_HUB_CSNT_USER} -p ${DOCKER_HUB_CSNT_PASSWORD} ${DOCKER_HUB_CSNT_SCHEME}://${DOCKER_HUB_CSNT}
    - docker push ${DOCKER_HUB_CSNT}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
    - docker logout ${DOCKER_HUB_CSNT}

# 將下載的 Image Push 到別的 Registry
.job_image_push_template: &job_image_push
  image: docker:latest
  script:
    - source build-vars.sh
    - . ci-template/shell/ci-function.sh
    - job_image_push_template_check_variable
    - docker login -u ${OLD_HUB_USER} -p ${OLD_HUB_PASSWORD} ${OLD_HUB_SCHEME}://${OLD_HUB}
    - docker pull ${OLD_HUB}/${OLD_IMAGE_NAME}:${OLD_IMAGE_TAG}
    - docker logout ${OLD_HUB}
    - docker tag ${OLD_HUB}/${OLD_IMAGE_NAME}:${OLD_IMAGE_TAG} ${NEW_HUB}/${NEW_IMAGE_NAME}:${NEW_IMAGE_TAG}
    - docker login -u ${NEW_HUB_USER} -p ${NEW_HUB_PASSWORD} ${NEW_HUB_SCHEME}://${NEW_HUB}
    - docker push ${NEW_HUB}/${NEW_IMAGE_NAME}:${NEW_IMAGE_TAG}
    - docker logout ${NEW_HUB}

# 推到 DEV 環境
push-dev:
  <<: *job_image_push
  stage: push_dev
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
    OLD_HUB_USER: ${DOCKER_HUB_CSNT_USER}
    OLD_HUB_PASSWORD: ${DOCKER_HUB_CSNT_PASSWORD}
    OLD_HUB_SCHEME: ${DOCKER_HUB_CSNT_SCHEME}
    OLD_HUB: ${DOCKER_HUB_CSNT}
    NEW_HUB_USER: ${DOCKER_HUB_DEV_USER}
    NEW_HUB_PASSWORD: ${DOCKER_HUB_DEV_PASSWORD}
    NEW_HUB_SCHEME: ${DOCKER_HUB_DEV_SCHEME}
    NEW_HUB: ${DOCKER_HUB_DEV}

# 發佈版本推到 發佈 環境
push-public-release:
  <<: *job_image_push
  stage: push_public_release
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
    OLD_HUB_USER: ${DOCKER_HUB_CSNT_USER}
    OLD_HUB_PASSWORD: ${DOCKER_HUB_CSNT_PASSWORD}
    OLD_HUB_SCHEME: ${DOCKER_HUB_CSNT_SCHEME}
    OLD_HUB: ${DOCKER_HUB_CSNT}
    NEW_HUB_USER: ${DOCKER_HUB_PUB_USER}
    NEW_HUB_PASSWORD: ${DOCKER_HUB_PUB_PASSWORD}
    NEW_HUB_SCHEME: ${DOCKER_HUB_PUB_SCHEME}
    NEW_HUB: ${DOCKER_HUB_PUB}
  only:
    variables:
      - $CI_COMMIT_TAG =~ /\d*[.]\d*[.]\d*[.]RELEASE/
```

首先 stages
``` yml
stages:
  - pre_build
  - maven_build
  - docker_build
  - push_dev
  - push_public_release
```
就是定義 Runner 的執行順序跟步驟

再來看我們第一個步驟 pre-build
``` yml
pre-build:
  image: centos:7
  stage: pre_build
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
  before_script:
    - . ci-template/shell/ci-function.sh
    - get_project_info
  script:
    - write_to_env_file
    - write_to_build_vars
  artifacts:
    paths:
      - .env
      - build-vars.sh
```

GIT_SUBMODULE_STRATEGY: recursive
設定 Runner 需連同 submodule 一起拉下來  
因為有些共用的腳本是放在 git submodule  

如何加上 git submodule  
``` bash
git submodule add ../../customer-service/ci-template.git
git status
git commit -a -m "first commit with submodule ci-template"
git push
```

如何更新 git submodule
``` bash
git submodule update --remote
git status
git commit -a -m "update with submodule ci-template"
git push
```

那 ci-template/shell/ci-function.sh 的內容
``` bash
#!/bin/sh

function get_project_info(){  
  if [ "$DEBUG" == "true" ]; then
    xmllint --format pom.xml
  fi
  get_project_name
  get_project_version
}

function get_project_name(){
  PROJECT_NAME=$(xmllint --xpath '/*[local-name()="project"]/*[local-name()="artifactId"]/text()' pom.xml)
}

function get_project_version(){
  PROJECT_VERSION=$(xmllint --xpath '/*[local-name()="project"]/*[local-name()="version"]/text()' pom.xml)
}

function get_build_cache_key(){
  BUILD_CACHE_KEY=$(md5sum pom.xml | awk '{print $1}')
}

# 將變數寫到 .env 檔, docker-build 打包使用
function write_to_env_file(){
  echo 'PROJECT_NAME='${PROJECT_NAME} >> .env
  echo 'PROJECT_VERSION='${PROJECT_VERSION} >> .env
  if [ "$DEBUG" == "true" ]; then
    cat .env
  fi
}

# 寫入打包命名資訊
function write_to_build_vars(){
  echo "export PROJECT_NAME=${PROJECT_NAME};" >> build-vars.sh
  echo "export PROJECT_VERSION=${PROJECT_VERSION};" >> build-vars.sh
  echo "export BUILD_IMAGE_NAME=csnt/${PROJECT_NAME};" >> build-vars.sh
  get_build_image_tag
  if [ "$DEBUG" == "true" ]; then
    cat build-vars.sh
  fi
}

# 取得tag的名稱, 如果沒有 tag 則用 rc 流水號, 如果有打標用打標的名稱
function get_build_image_tag(){
  if [ -z ${CI_COMMIT_TAG+x} ]; then
    echo "export BUILD_IMAGE_TAG=${PROJECT_VERSION}.rc.${CI_PIPELINE_ID};" >> build-vars.sh
  else
    echo "export BUILD_IMAGE_TAG=${CI_COMMIT_TAG};" >> build-vars.sh
  fi
}

function job_image_push_template_check_variable(){
  job_image_push_template_check_old_image_name
  job_image_push_template_check_old_image_tag
  job_image_push_template_check_new_image_name
  job_image_push_template_check_new_image_tag
}

function job_image_push_template_check_old_image_name(){
  if [ -z ${OLD_IMAGE_NAME+x} ]; then
    export OLD_IMAGE_NAME=${BUILD_IMAGE_NAME}
  fi
}

function job_image_push_template_check_old_image_tag(){
  if [ -z ${OLD_IMAGE_TAG+x} ]; then
    export OLD_IMAGE_TAG=${BUILD_IMAGE_TAG}
  fi
}

function job_image_push_template_check_new_image_name(){
  if [ -z ${NEW_IMAGE_NAME+x} ]; then
    export NEW_IMAGE_NAME=${BUILD_IMAGE_NAME}
  fi
}

function job_image_push_template_check_new_image_tag(){
  if [ -z ${NEW_IMAGE_TAG+x} ]; then
    export NEW_IMAGE_TAG=${BUILD_IMAGE_TAG}
  fi
}
```
主要就是取得 專案名 跟 定義版號


``` yml
  script:
    - write_to_env_file
    - write_to_build_vars
```
取得完成之後就將資訊寫入 .env 跟 build-vars.sh 這兩個檔案

``` yml
  artifacts:
    paths:
      - .env
      - build-vars.sh
```
最後將這兩個變數定義擋收集起來, 之後每一個步驟, Runner 會再放到目前工作目錄中

## maven-build

接下來是 java spring 專案的打包
``` yml
maven-build:
  image: maven:3.6.0-jdk-8-alpine
  stage: maven_build
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
    MAVEN_OPTS: "-Djava.awt.headless=true -Dmaven.repo.local=.m2/repository"
    MAVEN_CLI_OPTS: "-s .m2/settings.xml -B "
  before_script:
    - source build-vars.sh
    - mkdir -p .m2
    - cp ci-template/maven/settings.xml ./.m2/settings.xml
    - rm -f src/main/resources/bootstrap*.yml
    - |
      sed "s/    name: APPLICATION_NAME/    name: ${PROJECT_NAME}/g" ci-template/config/bootstrap_template.yml > src/main/resources/bootstrap.yml
    - rm -f src/main/resources/logback-spring.xml && rm -f src/main/resources/logback.xml
    - cp ci-template/config/logback-spring.xml src/main/resources/logback-spring.xml
    - ls src/main/resources/
    - cat src/main/resources/bootstrap.yml
  #cache:
  #  paths:
  #    - .m2/repository
  script:
    - mvn $MAVEN_CLI_OPTS package
  artifacts:
    paths:
      - target/*.jar
```

maven 打包指令大家就自己 google 一下  

before_script 這邊我們又用到 ci-template 的 git submodule 了

maven/settings.xml 的內容
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<settings
    xmlns="http://maven.apache.org/SETTINGS/1.1.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd">
    <servers>
        <server>
            <id>nexus-snapshots</id>
            <username>admin</username>
            <password>****admin123****</password>
        </server>
        <server>
            <id>nexus-releases</id>
            <username>admin</username>
            <password>****admin123****</password>
        </server>
        <server>
            <id>releases</id>
            <username>admin</username>
            <password>****admin123****</password>
        </server>
    </servers>
    <mirrors>
        <mirror>
            <id>nexus-public</id>
            <name>nexus-public</name>
            <url>http://nexus.samchu.com/repository/maven-public/</url>
            <mirrorOf>*</mirrorOf>
        </mirror>
    </mirrors>
</settings>
```

接下來刪除所有資源檔
```
rm -f src/main/resources/bootstrap*.yml  
```

我從 ci-template 複製並輸出專案名稱到專案內一起打包
```
sed "s/    name: APPLICATION_NAME/    name: ${PROJECT_NAME}/g" ci-template/config/bootstrap_template.yml > src/main/resources/bootstrap.yml  
```

config/bootstrap_template.yml
``` yml
spring:
  application:
    name: APPLICATION_NAME
  cloud:
    config:
      discovery:
         service-id: config-server-cs
         enabled: true
    inetutils:
      ignoredInterfaces:
        - docker0
        - veth.*
eureka:
  client:
    service-url:
      defaultZone: ${csnt.eureka.client.defaultzone}
    registry-fetch-interval-seconds: 5
  instance:
    prefer-ip-address: ${csnt.eureka.client.preferipaddress:true}
    lease-expiration-duration-in-seconds: 15
    lease-renewal-interval-in-seconds: 5
```

接著移除原始 log 設定, 放入適合 Elastic Search 的格式(JSON)  
再透過 Docker log -> Fluent Bit 轉送(後面再說明)
```
- rm -f src/main/resources/logback-spring.xml && rm -f src/main/resources/logback.xml
- cp ci-template/config/logback-spring.xml src/main/resources/logback-spring.xml
```

config/logback-spring.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    <property name="FILE_LOG_PATTERN" value="${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    <property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
    <!-- 標準輸出但是 JSON 格式 -->
    <appender name="CONSOLE_JSON" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <charset>UTF-8</charset>
            <layout class="ch.qos.logback.contrib.json.classic.JsonLayout">
                <timestampFormat>yyyy-MM-dd'T'HH:mm:ss.SSSX</timestampFormat>
                <timestampFormatTimezoneId>Etc/UTC</timestampFormatTimezoneId>
                <jsonFormatter class="ch.qos.logback.contrib.jackson.JacksonJsonFormatter">
                    <prettyPrint>false</prettyPrint>
                </jsonFormatter>
                <appendLineSeparator>true</appendLineSeparator>
            </layout>
        </encoder>
    </appender>
    <!-- 寫檔 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <charset>UTF-8</charset>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <cleanHistoryOnStart>${LOG_FILE_CLEAN_HISTORY_ON_START:-false}</cleanHistoryOnStart>
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz</fileNamePattern>
            <maxFileSize>${LOG_FILE_MAX_SIZE:-1GB}</maxFileSize>
            <maxHistory>${LOG_FILE_MAX_HISTORY:-30}</maxHistory>
            <totalSizeCap>${LOG_FILE_TOTAL_SIZE_CAP:-1GB}</totalSizeCap>
        </rollingPolicy>
    </appender>
    <root level="INFO">
        <appender-ref ref="FILE"/>
        <appender-ref ref="CONSOLE_JSON"/>
    </root>
</configuration>
```

再進行打包
```
- mvn $MAVEN_CLI_OPTS package
```

最後也是需要把產出的 jar 檔收集起來
``` yml
  artifacts:
    paths:
      - target/*.jar
```

## docker-build
這階段我有寫個備註 日常打包使用, 必推地端  

因為我們有分兩個 docker registry, 一個是日常打包 阿薩布魯 測試 實驗版 image 都會丟到 地端的 docker registry, 但當要定版就會寶這一次的包版 再往外推給 QA 用的 docker registry, 他們才不會看到一堆 rc 版, 基本上 QA 就只會看到 RELEASE

``` yml
# 日常打包使用, 必推地端
# https://git.ini.usc.edu/help/ci/yaml/README.md#git-submodule-strategy
# https://docs.gitlab.com/ce/ci/git_submodules.html#configuring-the-gitmodules-file
docker-build:
  image: docker:latest
  stage: docker_build
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
  before_script:
    - source build-vars.sh
    - cp ci-template/docker/Dockerfile ./Dockerfile
    - export DOCKER_IMAGE_NAME=${BUILD_IMAGE_NAME}
    - export DOCKER_IMAGE_TAG=${BUILD_IMAGE_TAG}
    - build_arg=$(cat .env | sed -e '/^#/d' -e '/^\s*$/d' | awk -F= '{print "--build-arg",$0 }' | sed -e ':a;N;$!ba;s/\n/ /g')
  script:
    - docker build ${build_arg} -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} --rm=true .
    - docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_HUB_CSNT}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
    - docker login -u ${DOCKER_HUB_CSNT_USER} -p ${DOCKER_HUB_CSNT_PASSWORD} ${DOCKER_HUB_CSNT_SCHEME}://${DOCKER_HUB_CSNT}
    - docker push ${DOCKER_HUB_CSNT}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
    - docker logout ${DOCKER_HUB_CSNT}
```
這階段就是很普通常見的 Docker build 然後再推到 registry

docker/Dockerfile
``` dockerfile
FROM openjdk:8u201-jdk-alpine3.9
VOLUME ["/pinpoint-agent","/tmp"]
ARG PROJECT_NAME=${PROJECT_NAME}
ARG PROJECT_VERSION=${PROJECT_VERSION}
COPY target/${PROJECT_NAME}-${PROJECT_VERSION}.jar app.jar

# install font packages, for excel
RUN apk --no-cache add msttcorefonts-installer fontconfig && \
    update-ms-fonts && \
    fc-cache -f

ENTRYPOINT java -XX:+UnlockExperimentalVMOptions \
  -XX:+UseCGroupMemoryLimitForHeap -Djava.security.egd=file:/dev/./urandom ${JAVA_OPTS} -jar /app.jar
```

pinpoint 是我們用的監控 後面再講, 這邊還有加裝字型檔 msttcorefonts-installer 如果有要輸出 Excel 就比較會需要用到

搭配這階段的腳本  
必須去 Gitlab 上的該專案 Settings -> CI / CD -> Variables 寫上相關變數
![Imgur](https://i.imgur.com/rbbRwkk.png)

範例值
```
DOCKER_HUB_CSNT_SCHEME
http

DOCKER_HUB_CSNT
nexus.samchu.com:15000

DOCKER_HUB_CSNT_USER
admin

DOCKER_HUB_CSNT_PASSWORD
********
```

以上基本的打包已完成

接下來就是腳本模板
``` yml
# 將下載的 Image Push 到別的 Registry
.job_image_push_template: &job_image_push
  image: docker:latest
  script:
    - source build-vars.sh
    - . ci-template/shell/ci-function.sh
    - job_image_push_template_check_variable
    - docker login -u ${OLD_HUB_USER} -p ${OLD_HUB_PASSWORD} ${OLD_HUB_SCHEME}://${OLD_HUB}
    - docker pull ${OLD_HUB}/${OLD_IMAGE_NAME}:${OLD_IMAGE_TAG}
    - docker logout ${OLD_HUB}
    - docker tag ${OLD_HUB}/${OLD_IMAGE_NAME}:${OLD_IMAGE_TAG} ${NEW_HUB}/${NEW_IMAGE_NAME}:${NEW_IMAGE_TAG}
    - docker login -u ${NEW_HUB_USER} -p ${NEW_HUB_PASSWORD} ${NEW_HUB_SCHEME}://${NEW_HUB}
    - docker push ${NEW_HUB}/${NEW_IMAGE_NAME}:${NEW_IMAGE_TAG}
    - docker logout ${NEW_HUB}
```

以下的寫法就是執行上面 job_image_push_template 但 image 來源跟目的地可以動態調整
``` yml
# 推到 DEV 環境
push-dev:
  <<: *job_image_push
  stage: push_dev
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
    OLD_HUB_USER: ${DOCKER_HUB_CSNT_USER}
    OLD_HUB_PASSWORD: ${DOCKER_HUB_CSNT_PASSWORD}
    OLD_HUB_SCHEME: ${DOCKER_HUB_CSNT_SCHEME}
    OLD_HUB: ${DOCKER_HUB_CSNT}
    NEW_HUB_USER: ${DOCKER_HUB_DEV_USER}
    NEW_HUB_PASSWORD: ${DOCKER_HUB_DEV_PASSWORD}
    NEW_HUB_SCHEME: ${DOCKER_HUB_DEV_SCHEME}
    NEW_HUB: ${DOCKER_HUB_DEV}
```

這個 only 就是當打 Tag 的時候 tag name 為 1.1.0.RELEASE 才作動, 表示這是我們定版要推出去給 QA
``` yml
# 發佈版本推到 發佈 環境
push-public-release:
  <<: *job_image_push
  stage: push_public_release
  variables:
    GIT_SUBMODULE_STRATEGY: recursive
    OLD_HUB_USER: ${DOCKER_HUB_CSNT_USER}
    OLD_HUB_PASSWORD: ${DOCKER_HUB_CSNT_PASSWORD}
    OLD_HUB_SCHEME: ${DOCKER_HUB_CSNT_SCHEME}
    OLD_HUB: ${DOCKER_HUB_CSNT}
    NEW_HUB_USER: ${DOCKER_HUB_PUB_USER}
    NEW_HUB_PASSWORD: ${DOCKER_HUB_PUB_PASSWORD}
    NEW_HUB_SCHEME: ${DOCKER_HUB_PUB_SCHEME}
    NEW_HUB: ${DOCKER_HUB_PUB}
  only:
    variables:
      - $CI_COMMIT_TAG =~ /\d*[.]\d*[.]\d*[.]RELEASE/
```

好的, 以上就是打包動作流程, 那作到這邊你或許缺個 Nexus

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/tw/"><img alt="創用 CC 授權條款" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/3.0/tw/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">SAM的程式筆記</span>由<a xmlns:cc="http://creativecommons.org/ns#" href="https://blog.samchu.dev/" property="cc:attributionName" rel="cc:attributionURL">朱尚禮</a>製作，以<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/tw/">創用CC 姓名標示-非商業性-相同方式分享 3.0 台灣 授權條款</a>釋出。