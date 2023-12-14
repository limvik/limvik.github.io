---
layout: post
title: Spring Embedded Tomcat으로 실행하는 WAR 패키지 컨테이너 만들기
categories:
- Framework
- Spring
- JSP
tags:
- JSP
- Spring
- Docker
date: 2023-12-05 17:10 +0900
---
## Intro

Spring Boot에서 JSP를 사용하려면 별도로 설정해줘야 할 것도 있고, 배포할 때도 WAR(Web Archive)로 패키징 해서 외부 Tomcat 서버에 WAR 파일을 붙여넣기 해야하는 등 귀찮은 일이 많았습니다. (해보지는 않았지만 Executable WAR를 생성해서 java -jar 명령어로 실행도 가능하다고 합니다.)

Docker도 조금 배웠고, 문득 첫 프로젝트에 사용했던 JSP가 기억이 나서 Spring, JSP 프로젝트를 컨테이너화 하는 방법을 찾아봤습니다. 그런데 `maven` 에서는 `bootWar` 가 안돼 Executable WAR 파일을 만들 수 없어서 외부 Tomcat 서버가 필요하다는 글을 포함해서, 컨테이너 단독으로 실행하는게 잘 안된다는 글이 많이 보입니다.

JSP로 프로젝트를 하는 건 아니지만, Docker 연습도 할 겸, Spring Boot를 이용한 JSP 프로젝트 WAR 파일을 Embedded Tomcat을 이용하는 컨테이너로 만드는 방법, 그리고 개발 환경에서 Static 파일 변경 내용을 실행 중인 컨테이너에 바로 반영하는 방법을 공유합니다.

## 환경

- Windows 11
- Spring Boot 3.2.0
  - 최소 2.5.0 이상
- JDK 17
  - Spring Boot 2 인 경우, JDK 8을 사용해도 잘 되는 것으로 보입니다([https://mio-java.tistory.com/67](https://mio-java.tistory.com/67)). 필요한 것은 Layertools 입니다.
- Docker Desktop 4.25.2
  - Compose Watch 기능을 위해 최소 4.24.0 이상 필요

## Build 설정

Gradle과 Maven에서의 설정 방법을 살펴봅니다. 간단하게 [Spring Initilizr](https://start.spring.io/#!type=gradle-project&language=java&platformVersion=3.2.0&packaging=war&jvmVersion=17&groupId=com.limvik&artifactId=jsp-docker&name=jsp-docker&description=JSP%20project%20with%20docker&packageName=com.limvik.jsp-docker&dependencies=web)를 이용해서 spring-boot-starter-web만 추가했고, JSP 파일을 Servlet으로 변환해주는 `Jasper` 는 목록에 없으므로 직접 추가합니다. Spring Boot 3.2.0 내장 Tomcat 버전에 맞춰서 10.1.16으로 추가했습니다.

프로젝트 하는게 주 목적은 아니다보니, JSTL은 추가 안했습니다.

### Gradle

```groovy
plugins {
    id 'java'
    id 'war'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.limvik'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    // https://mvnrepository.com/artifact/org.apache.tomcat.embed/tomcat-embed-jasper
    implementation 'org.apache.tomcat.embed:tomcat-embed-jasper:10.1.16'

}

tasks.named('test') {
    useJUnitPlatform()
}
```

### Maven

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.limvik</groupId>
    <artifactId>jsp-docker-maven</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>jsp-docker-maven</name>
    <description>jsp-docker-maven</description>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.tomcat.embed/tomcat-embed-jasper -->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <version>10.1.16</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

## Spring 설정

이미 알고 계시겠지만, Spring Boot 프로젝트에서 JSP 파일 인식을 위해 경로를 지정해줍니다.

```yaml
spring:  
  application:  
    name: jsp-docker  
  mvc:  
    view:  
      prefix: /WEB-INF/views/  
      suffix: .jsp
```

## JSP 및 JS 파일 추가

테스트 용으로 아주 간단한 JSP 파일과 JS 파일을 추가합니다. 혹시 처음 하시는 분이 볼지도 모르니 JSP 파일의 위치를 보여드리겠습니다. 앞서 Spring 설정 파일에 추가한 경로는 src/main/`webapp` 디렉터리 내에 있어야 합니다. 반면, JS 파일과 같은 다른 static 파일은 src/main/resources/`static` 디렉터리 내에 있어야 합니다.

![01.프로젝트 구조](/assets/img/2023-12-05-jsp-with-spring-embedded-tomcat-by-docker/01.project-structure.png)

그리고 JSP 파일은 아래와 같이 간단하게 작성해줍니다.

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html lang="ko">
<script src="/jsp-docker.js"></script>
<head>
    <title>JSP-Docker</title>
</head>
<body>
    <h1>JSP-Docker</h1>
    <h3>스프링 부트 with JSP</h3>
</body>
</html>
```

다음으로 간단하게 경고창 띄우는 JS 내용을 작성합니다.

```javascript
window.onload = function () {  
    alert("jsp-docker-test");  
}
```

## Dockerfile 작성

이제 컨테이너로 만들기 위해서 Dockerfile을 작성합니다. Gradle과 Maven의 WAR 파일 빌드 결과가 저장되는 기본 경로가 다르므로, 2개의 Dockerfile을 작성하겠습니다.

### Gradle용 Dockerfile

build한 WAR 파일을 복사해와서 layertools를 이용해서 WAR 파일의 압축을 풀고, 보안을 위해 사용자를 추가한 후, 압축을 푼 파일들을 복사해오고, `WarLauncher` 를 실행합니다.

Docker는 Layer 단위로 캐싱합니다. application 디렉터리에는 사용자가 자주 변경하는 Java 파일과 static 파일 등이 위치하여, 효율적으로 Docker 이미지를 빌드하고, 배포 시에도 변경된 Layer만 전송하여 데이터 전송량을 절약할 수 있습니다.

> layertools는 WAR파일의 경우 Spring Boot 2.5.0 버전부터 사용 가능합니다.  
> WAR 파일 layertools 지원 관련 GitHub Issue([링크](https://github.com/spring-projects/spring-boot/issues/22195)) 및 커밋([GitHub 링크](https://github.com/spring-projects/spring-boot/commit/1245e5eec9179338b9c7663d777ef201d8f065aa))

```dockerfile
# 빌드 스테이지
FROM eclipse-temurin:17 AS builder
LABEL authors="limvik"

WORKDIR workspace
ARG WAR_FILE=build/libs/jsp-docker-0.0.1-SNAPSHOT.war
COPY ${WAR_FILE} jsp-docker.war
RUN java -Djarmode=layertools -jar jsp-docker.war extract

# 실행 스테이지
FROM eclipse-temurin:17
RUN useradd limvik
USER limvik
WORKDIR workspace

# layertools로 추출된 레이어들 복사
COPY --from=builder workspace/dependencies/ ./
COPY --from=builder workspace/spring-boot-loader/ ./
COPY --from=builder workspace/snapshot-dependencies/ ./
COPY --from=builder --chown=limvik:limvik workspace/application/ ./

ENTRYPOINT ["java", "org.springframework.boot.loader.launch.WarLauncher"]
```

WAR 파일을 layertools를 이용한 결과는 아래와 같습니다. application에 JSP 파일이 있는`WEB-INF/views/` 가 있는 것을 확인할 수 있습니다.

```text
- "dependencies":
  - "WEB-INF/lib/"
- "spring-boot-loader":
  - "org/"
- "snapshot-dependencies":
- "application":
  - "META-INF/"
  - "WEB-INF/classes/"
  - "WEB-INF/classpath.idx"
  - "WEB-INF/layers.idx"
  - "WEB-INF/views/"
```

### Maven용 Dockerfile

큰 차이는 없지만 WAR_FILE의 경로를 Maven 으로 build 시 기본 경로인 target 으로 변경하였습니다. 이외의 내용은 모두 동일합니다.

```dockerfile
# 빌드 스테이지
FROM eclipse-temurin:17 AS builder
LABEL authors="limvik"

WORKDIR workspace
ARG WAR_FILE=target/jsp-docker-maven-0.0.1-SNAPSHOT.war
COPY ${WAR_FILE} jsp-docker.war
RUN java -Djarmode=layertools -jar jsp-docker.war extract

# 실행 스테이지
FROM eclipse-temurin:17
RUN useradd limvik
USER limvik
WORKDIR workspace

# layertools로 추출된 레이어들 복사
COPY --from=builder workspace/dependencies/ ./
COPY --from=builder workspace/spring-boot-loader/ ./
COPY --from=builder workspace/snapshot-dependencies/ ./
COPY --from=builder --chown=limvik:limvik workspace/application/ ./

ENTRYPOINT ["java", "org.springframework.boot.loader.launch.WarLauncher"]
```

Dockerfile 마지막 `COPY` 명령어는 `--chown` 을 사용하여 파일의 소유자를 변경하였는데, 이는 docker compose에서 JSP 파일과 JS 파일 등 static 파일의 변경사항을 개발환경에서 실시간으로 반영하기 위함입니다. 실제 컨테이너 배포용으로 Dockerfile을 사용한다면, 보안을 위해 `--chown` 은 제거해야 합니다.

혹은 개발용으로 `RUN useradd limvik`, `USER limvik`, `--chown=limvik:limvik` 을 제거하여 Root 사용자를 사용하는 Dockerfile을 별도로 만드는 것도 괜찮겠습니다.

## 개발 환경 구성을 위한 Docker Compose 작성

글 목적에 집중하기 위해 Spring 프로젝트만 작성하겠습니다.

단독 컨테이너인데 Docker Compose를 사용한다면, 개인적인 주 목적은 `변경사항에 대한 실시간 반영`입니다. Docker Compose 의 `Watch` 기능을 사용하여 변경사항을 실행 중인 Container에 바로 반영할 수 있습니다.

먼저 전체적으로 보면 아래와 같습니다.

```yaml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    restart: always
    container_name: jsp-docker
    develop:
      watch:
        - action: sync+restart
          path: ./src/main/webapp/WEB-INF
          target: ./workspace/WEB-INF
        - action: sync+restart
          path: ./src/main/resources/static
          target: ./workspace/WEB-INF/classes/static
        - action: rebuild
          path: ./build/libs
    ports:
      - '8080:8080'
```

develop의 watch 속성 내에 `JSP 파일이 변경`된 경우, `JS, CSS 등 static 파일이 변경`된 경우, `Dependency 또는 Java 코드 등이 변경되어 새롭게 빌드`한 경우에 대해 실시간으로 반영하도록 작성하였습니다.

`action` 속성의 경우 이름대로 무엇을 할지 지정합니다. `sync` 만 지정하는 경우 변경된 파일을 복사하여 container 내부 저장소로 복사만 합니다. `sync+restart`로 지정하는 경우 container를 재시작까지 합니다. Spring Boot 프로젝트는 재시작을 해야 static 파일 변경 사항이 반영되므로 `sync+restart`로 설정합니다. 그리고 새롭게 WAR 파일을 build한 경우에는 이 WAR 파일을 이용해서 Docker image 가 새롭게 build 되도록 `rebuild` 로 설정합니다.

`path` 속성의 경우 변경 사항을 감지할 로컬 경로를 지정합니다.

`target` 속성의 경우 변경된 로컬 파일을 붙여넣을 container 상의 경로를 지정합니다. rebuild의 경우 새롭게 image를 build 하여 변경사항을 모두 반영하므로 target을 별도로 지정할 필요가 없습니다. 이외에는 JSP 파일이 있는 `/workspace/WEB-INF`와 static 파일이 있는 `./workspace/WEB-INF/classes/static` 경로를 각각 지정해줍니다.

`ignore` 속성은 여기 작성하지 않았지만, 변경이 발생해도 action을 수행하지 않을 경로를 지정할 수 있습니다. 이전에 작성한 Watch 관련 글([링크](https://limvik.github.io/posts/how-to-use-docker-compose-watch/#configuration))에 간단한 사용법을 참고하실 수 있습니다.

Maven을 사용하는 Compose 파일 작성 시에는 rebuild 의 path를 Maven에 맞추어 수정해야합니다. 그리고 저는 Dockerfile을 같은 프로젝트에 두어서 Maven용 Dockerfile은 Dockerfile-maven으로 변경하였습니다.

```yaml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile-maven
    restart: always
    container_name: jsp-docker-maven
    develop:
      watch:
        - action: sync+restart
          path: ./src/main/webapp/WEB-INF
          target: ./workspace/WEB-INF
        - action: sync+restart
          path: ./src/main/resources/static
          target: ./workspace/WEB-INF/classes/static
        - action: rebuild
          path: ./target
    ports:
      - '8080:8080'
```

### WAR 파일 Build 및 Compose 파일 실행

Windows의 경우 `프로젝트 경로`에서 명령어를 아래와 같이 작성하여 실행할 수 있습니다.

#### Maven Build

```powershell
./mvnw package
```

#### Gradle Build

```powershell
./gradlew build
```

#### Compose 실행

```powershell
docker compose -p jsp-docker-gradle watch
```

파일 명을 지정할 경우에는 `-f` 옵션을 사용할 수 있습니다.

```powershell
docker compose -f compose-maven.yml -p jsp-docker-maven watch
```

`-p` 는 프로세스 이름을 지정하는 것인데, watch를 종료해도 가끔 프로세스가 종료되지 않아 프로세스 이름을 바꿔가면서 실행해야하는 경우가 있어서 프로세스 이름을 지정합니다.

다음은 위의 docker compose 명령을 실행하고, 변경사항을 실시간으로 적용했을 때 Log입니다.

```text
[+] Building 1.8s (15/15) FINISHED                                                                                                                                                                                   docker:default
 => [app internal] load .dockerignore                                                                                                                                                                                          0.0s
 => => transferring context: 2B                                                                                                                                                                                                0.0s 
 => [app internal] load build definition from Dockerfile-maven                                                                                                                                                                 0.0s 
 => => transferring dockerfile: 752B                                                                                                                                                                                           0.0s 
 => [app internal] load metadata for docker.io/library/eclipse-temurin:17                                                                                                                                                      0.0s 
 => [app builder 1/4] FROM docker.io/library/eclipse-temurin:17                                                                                                                                                                0.0s 
 => [app internal] load build context                                                                                                                                                                                          0.7s 
 => => transferring context: 23.48MB                                                                                                                                                                                           0.7s 
 => CACHED [app builder 2/4] WORKDIR workspace                                                                                                                                                                                 0.0s
 => [app builder 3/4] COPY target/jsp-docker-maven-0.0.1-SNAPSHOT.war jsp-docker.war                                                                                                                                           0.1s 
 => [app builder 4/4] RUN java -Djarmode=layertools -jar jsp-docker.war extract                                                                                                                                                0.7s
 => CACHED [app stage-1 2/7] RUN useradd limvik                                                                                                                                                                                0.0s
 => CACHED [app stage-1 3/7] WORKDIR workspace                                                                                                                                                                                 0.0s 
 => CACHED [app stage-1 4/7] COPY --from=builder workspace/dependencies/ ./                                                                                                                                                    0.0s 
 => CACHED [app stage-1 5/7] COPY --from=builder workspace/spring-boot-loader/ ./                                                                                                                                              0.0s 
 => CACHED [app stage-1 6/7] COPY --from=builder workspace/snapshot-dependencies/ ./                                                                                                                                           0.0s
 => [app stage-1 7/7] COPY --from=builder --chown=limvik:limvik workspace/application/ ./                                                                                                                                      0.0s 
 => [app] exporting to image                                                                                                                                                                                                   0.0s 
 => => exporting layers                                                                                                                                                                                                        0.0s 
 => => writing image sha256:eba19da7a58883c092cfd65254e7dd69a4ebcdfa1dae942803f4df0ece952b73                                                                                                                                   0.0s 
 => => naming to docker.io/library/jsp-docker-maven-app                                                                                                                                                                       0.0s 
[+] Running 2/2
 ✔ Network jsp-docker-maven_default  Created                                                                                                                                                                                  0.2s 
 ✔ Container jsp-docker-maven         Started                                                                                                                                                                                  0.0s 
watching [C:\Users\funco\IdeaProjects\jsp-docker\src\main\webapp\WEB-INF C:\Users\funco\IdeaProjects\jsp-docker\src\main\resources\static C:\Users\funco\IdeaProjects\jsp-docker\target]

[+] Restarting 1/1
 ✔ Container jsp-docker-maven  Started                                                                                                                                                                                         0.9s 
Syncing app after changes were detected:
  - C:\Users\funco\IdeaProjects\jsp-docker\src\main\webapp\WEB-INF\views\index.jsp
  - C:\Users\funco\IdeaProjects\jsp-docker\src\main\webapp\WEB-INF\views
[+] Restarting 1/1
 ✔ Container jsp-docker-maven  Started                                                                                                                                                                                         0.7s 
Syncing app after changes were detected:
  - C:\Users\funco\IdeaProjects\jsp-docker\src\main\resources\static\jsp-docker.js
[+] Restarting 1/1
 ✔ Container jsp-docker-maven  Started
```

주의할 점은 Devtools 를 사용하더라도 WAR 파일로 패키징 후에는 (배포 시에도 사용하도록 변경하지 않는 이상) static 파일이 `캐싱(Caching)`될 수 있으므로, 캐싱하지 않도록 설정해주어야 합니다.

혹은 Devtools를 개발 환경에서는 일단 패키징을 해도 사용하도록 설정을 변경해야 합니다. Devtools를 변경하는게 제일 편할 것 같기는 합니다. 예를들어 Gradle의 경우 `developmentOnly` 에서 `implementation`으로 변경합니다. 대신 Production에 반영하지 않도록 주의합니다.

## Outro

개인적으로는 Docker Compose 의 Watch 에서 `sync` 를 써보고 싶었는데, 겸사겸사 써볼 수 있었습니다. JSP를 프로젝트에 사용할 일이 있을지는 모르겠지만, 사용하게 된다면 개인적으로는 이렇게 환경 세팅을 할 계획입니다.

배포용으로는 Dockerfile 에서 `--chown` 제거하는 것 잊지 마시고, JSP 로 개발하시고 배포하시는데 도움되셨길 바랍니다.

전체 프로젝트는 [GitHub 저장소](https://github.com/limvik/jsp-docker)에 올려두었습니다.
