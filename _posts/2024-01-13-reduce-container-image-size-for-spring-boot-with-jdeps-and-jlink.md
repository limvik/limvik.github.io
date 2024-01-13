---
layout: post
title: jdeps와 jlink를 통한 Spring Boot 컨테이너 이미지 사이즈 경량화
categories:
- Java
tags:
- Java
- Spring
- Docker
date: 2024-01-13 14:15 +0900
---
## Intro

개인 프로젝트를 실제 서비스 만드는 느낌을 내보려고, GitHub Private Repository에서 작업하고 있습니다.

GitHub Container Registry 는 컨테이너 이미지 저장 공간이 1GB 까지 무료인데, 서비스 별로 별도의 저장소를 만들고, 컨테이너 이미지도 각각 만들려고 하다보니 무료 공간이 부족합니다. 컨테이너 이미지 생성 시 `eclipse-temurin`을 사용해서 Spring Boot 컨테이너 이미지를 생성하면 사이즈가 약 459MB 라 2개까지 무료로 저장이 가능합니다.

비용 절감을 위해 `jdeps`와 `jlink`를 이용해서 컨테이너 이미지 사이즈를 약 `459MB`에서 약 `179MB` 까지 개선한 방법을 공유합니다.

## 환경

- Windows 11
- Spring Boot 3.1.5
- Java 17
  -  jlink 사용을 위해 최소 Java 11 필요
- Docker Desktop 4.26.1

## jdeps와 jlink

Java Command Line tool인 jdeps와 jlink를 잘 모르실수도 있으므로, 간단하게 살펴보고 넘어가겠습니다.

### jdeps

[jdeps](https://docs.oracle.com/en/java/javase/11/tools/jdeps.html)는 이름에서 느낌이 오신 분도 있겠지만, Java class 의존성(dependency) 분석기 입니다.

Command Line에서 jdeps를 사용하면 의존성을 출력할 수 있습니다. 아래는 powershell 에서 jdeps를 사용해 의존성을 출력한 결과입니다. 사용된 option에 따라 출력은 달라질 수 있습니다. option은 위 jdeps 링크에서 확인하실 수 있습니다. 사용한 option은 뒤에서 설명하겠습니다.

```powershell
& "$Env:JAVA_HOME\bin\jdeps.exe" `
--ignore-missing-deps `
--print-module-deps `
econome-0.0.1-SNAPSHOT.jar

java.base,java.logging
```

뒤에서 보시겠지만, 완벽하게 의존성을 분석할 수 있는 것은 아니기 때문에 일부 노가다가 필요합니다.

### jlink

[jlink](https://docs.oracle.com/en/java/javase/11/tools/jlink.html)는 Java 11에서 추가된 Custom JRE(Java Runtime Environment) 생성 도구입니다. Java 8까지는 모듈화되지 않았지만, Java 9부터 모듈화되기 시작하여 Java 11에서는 필요한 모듈(=필요한 class 파일)만 사용하는 JRE를 만들 수 있게 되었습니다. Java 모듈화와 관련된 자세한 내용은 [Project Jigsaw](https://openjdk.org/projects/jigsaw/) 등에서 살펴 볼 수 있습니다.

jlink도 jdeps와 마찬가지로 간단하게 사용해보고 넘어가겠습니다. 아래와 같이 powershell에서 명령어를 입력합니다. 상세한 option은 뒤에서 살펴보겠습니다.

```powershell
"$Env:JAVA_HOME\bin\jlink.exe" `
>> --add-modules java.base,java.logging `
>> --strip-debug `
>> --no-man-pages `
>> --no-header-files `
>> --compress=2 `
>> --output /javaruntime
```
그러면 아래와 같이 약 31MB의 Custom JRE가 C 드라이브에 생성된 것을 볼 수 있습니다. 모듈의 갯수가 적어서 사이즈가 굉장히 작아집니다.

![01.Custom JRE 상세 속성 화면](/assets/img/2024-01-13-reduce-container-image-size-for-spring-boot-with-jdeps-and-jlink/01.custom-jre-size.png)

## Dockerfile 수정

이제 기존의 Dockerfile을 jdeps와 jlink를 통해 Custom JRE를 사용하도록 수정해보겠습니다.

### 기존 Dockerfile

기존의 Dockerfile은 eclipse-temurin:17을 사용하여 컨테이너 이미지를 생성합니다.

```dockerfile
FROM eclipse-temurin:17 AS builder
LABEL authors="limvik"

WORKDIR workspace
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} econome.jar
RUN java -Djarmode=layertools -jar econome.jar extract

FROM eclipse-temurin:17
RUN useradd limvik
USER limvik
WORKDIR workspace
COPY --from=builder workspace/dependencies/ ./
COPY --from=builder workspace/spring-boot-loader/ ./
COPY --from=builder workspace/snapshot-dependencies/ ./
COPY --from=builder workspace/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```
그러면 아래와 같이 `459MB`의 컨테이너 이미지가 생성됩니다.

![02.docker image ls 명령어 결과 화면](/assets/img/2024-01-13-reduce-container-image-size-for-spring-boot-with-jdeps-and-jlink/02.docker-image-ls-before-reduce-size.png)

eclipse-temurin의 사이즈가 약 405MB니까, 프로젝트가 차지하는 용량은 약 52MB가 되겠습니다.

### Dockerfile에 jdeps와 jlink 추가

이제 앞서 봤던 jdeps와 jlink를 통해 Custom JRE로 컨테이너 이미지를 빌드하도록 수정한 결과입니다. 수정된 사항은 주석을 작성했습니다.

```dockerfile
FROM eclipse-temurin:17 AS builder
LABEL authors="limvik"

WORKDIR workspace
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} econome.jar
RUN java -Djarmode=layertools -jar econome.jar extract

# 디렉토리별로 jar 파일 및 class 파일의 목록 생성
RUN find ./dependencies -type f -name '*.jar' -exec echo -n {}: \; > classpath.info && \
    find ./spring-boot-loader -type f \( -name '*.jar' -or -name '*.class' \) -exec echo -n {}: \; >> classpath.info && \
    find ./snapshot-dependencies -type f -name '*.jar' -exec echo -n {}: \; >> classpath.info && \
    find ./application -type f \( -name '*.jar' -or -name '*.class' \) -exec echo -n {}: \; >> classpath.info

#jdeps를 이용한 의존성 분석
RUN $JAVA_HOME/bin/jdeps \
    # 재귀적으로 모든 run-time dependencies 분석
    --recursive \
    # Missing dependencies 관련 error 출력 제거 \
    --ignore-missing-deps \
    # jlink --add-modules option에서 사용할 수 있도록 modules를 ','로 구분한 형태로 출력
    --print-module-deps \
    # Multi-Release JAR 파일이 있는 경우 17버전 기준으로 의존성 분석
    --multi-release 17 \
    # class-path 지정
    --class-path $(cat classpath.info) \
    # 의존성 분석 path 지정 및 분석 결과 저장
    ./ > jre-deps.info

#jlink를 이용한 Custom JRE 생성
RUN $JAVA_HOME/bin/jlink \
    # 필요한 modules 추가
    --add-modules "$(cat jre-deps.info),java.management.rmi,java.prefs,java.se,java.security.sasl,java.xml.crypto,jdk.management,jdk.crypto.ec" \
    # 출력에서 debug information 제거
    --strip-debug \
    # man pages 제외
    --no-man-pages \
    # header files 제외
    --no-header-files \
    # 모든 resources를 ZIP으로 압축
    --compress=2 \
    # runtime image 생성 위치
    --output /javaruntime

# Base Image 지정
FROM debian:buster-slim
RUN useradd limvik
USER limvik
# Java 환경변수 설정
ENV JAVA_HOME=/opt/java/openjdk
ENV PATH "${JAVA_HOME}/bin:${PATH}"
# Custom JRE 복사
COPY --from=builder /javaruntime $JAVA_HOME
WORKDIR workspace
COPY --from=builder workspace/dependencies/ ./
COPY --from=builder workspace/spring-boot-loader/ ./
COPY --from=builder workspace/snapshot-dependencies/ ./
COPY --from=builder workspace/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

이렇게 Dockerfile을 수정하고나면, 컨테이너 이미지의 크기가 `179MB`까지 줄어듭니다.

![03.용량 감소 후 docker image ls 결과 화면](/assets/img/2024-01-13-reduce-container-image-size-for-spring-boot-with-jdeps-and-jlink/03.docker-image-ls-after-reduce-size.png)

의문이 생길법한 부분은 jdeps를 이용해 의존성 분석 결과를 jlink에 추가했는데도 불구하고, 추가적인 모듈을 직접 추가한 부분입니다. 앞서 말씀드렸듯 jdeps를 통해서 완벽한 의존성 분석이 불가능하기 때문에, 필요한 모듈을 직접 추가해주어야 합니다.

```dockerfile
#jlink를 이용한 Custom JRE 생성
RUN $JAVA_HOME/bin/jlink \
    # 필요한 modules 추가
    --add-modules "$(cat jre-deps.info),java.management.rmi,java.prefs,java.se,java.security.sasl,java.xml.crypto,jdk.management,jdk.crypto.ec" \
```

참고로 추가되는 전체 모듈 목록은 아래와 같습니다. 시각적 편의보다는 복붙의 편의를 위해 `,`로 구분한 목록을 올립니다.

```
java.base,java.compiler,java.datatransfer,java.xml,java.prefs,java.desktop,java.instrument,java.logging,java.management,java.security.sasl,java.naming,java.rmi,java.management.rmi,java.net.http,java.scripting,java.security.jgss,java.transaction.xa,java.sql,java.sql.rowset,java.xml.crypto,java.se,jdk.crypto.ec,jdk.jfr,jdk.management,jdk.unsupported
```

원래라면 노가다를 통해 필요한 모듈을 알아내야 했겠지만, 다행히도 노가다 결과를 공유해주신 분의 글([링크](https://kevin-park.medium.com/springboot-java17-with-jlink-ec0242910c36))이 있어 쉽게 목록을 만들 수 있었습니다.

모듈 관리가 불편하다면, 어느정도 공간 손실을 감수하고 JDK에 있는 모든 모듈을 추가해버리는 방법도 하나의 선택지가 될 수 있습니다.

### ALL-MODULE-PATH

`--add-modules`에 `ALL-MODULE-PATH` 를 작성해주면 됩니다. 의존하는 모든 모듈을 추가하므로 jdeps를 사용할 필요도 없습니다.

```dockerfile
FROM eclipse-temurin:17 AS builder
LABEL authors="limvik"

WORKDIR workspace
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} econome.jar
RUN java -Djarmode=layertools -jar econome.jar extract
#jlink를 이용한 Custom JRE 생성
RUN $JAVA_HOME/bin/jlink \
    # 전체 modules 추가
    --add-modules ALL-MODULE-PATH \
    # 출력에서 debug information 제거
    --strip-debug \
    # man pages 제외
    --no-man-pages \
    # header files 제외
    --no-header-files \
    # 모든 resources를 ZIP으로 압축
    --compress=2 \
    # runtime image 생성 위치
    --output /javaruntime

# Base Image 지정
FROM debian:buster-slim
RUN useradd limvik
USER limvik
# Java 환경변수 설정
ENV JAVA_HOME=/opt/java/openjdk
ENV PATH "${JAVA_HOME}/bin:${PATH}"
# Custom JRE 복사
COPY --from=builder /javaruntime $JAVA_HOME
WORKDIR workspace
COPY --from=builder workspace/dependencies/ ./
COPY --from=builder workspace/spring-boot-loader/ ./
COPY --from=builder workspace/snapshot-dependencies/ ./
COPY --from=builder workspace/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

확인해보면 아래와 같이 `211MB`이기는 하지만, `Base Image`와 `JDK 종류`에 따라 많이 달라질 수 있습니다.

![ALL-MODULE-PATH 적용 후 docker image ls 명령 출력 결과](/assets/img/2024-01-13-reduce-container-image-size-for-spring-boot-with-jdeps-and-jlink/04.docker-image-ls-with-all-module-path.png)

## Outro

jdeps로 의존성 분석이 완벽하게 안된다는 사실을 깨닫기까지 시간이 꽤 걸렸습니다. classpath 지정을 잘못한줄 알고 이런저런 시도를 하다가 Dockerfile이 조금 과하다 싶을 정도로 길어졌습니다.

정신 건강상 비용이나 저장 공간 압박이 심한 상황이 아니면, 적절한 Base Image와 JDK를 골라서 ALL-MODULE-PATH로 전체 모듈을 추가하는게 좋아보입니다.

다른 벤더의 JDK를 사용하면 오류가 발생할 수 있을 것 같습니다. 다른 JDK 사용하시는 분들은 아래 참고자료들도 확인해 보시는 것을 추천드립니다.

## 참고자료

- [Docker Hub eclipse-temurin official image](https://hub.docker.com/_/eclipse-temurin)
- [jdeps man page](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jdeps.html)
- [jlink man page](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jlink.html)
- [jlink를 사용하는 Java 런타임](https://learn.microsoft.com/ko-kr/java/openjdk/java-jlink-runtimes)
- [Java: Developing smaller Docker images with jdeps and jlink](https://levelup.gitconnected.com/java-developing-smaller-docker-images-with-jdeps-and-jlink-d4278718c550)
- [RedHat - Using jlink to customize Java runtime environment](https://access.redhat.com/documentation/en-us/red_hat_build_of_openjdk/21/html/using_jlink_to_customize_java_runtime_environment/index)
- [adoptium - Creating your own runtime using jlink](https://adoptium.net/blog/2021/10/jlink-to-produce-own-runtime/)
- [springboot-java17-with-jlink](https://kevin-park.medium.com/springboot-java17-with-jlink-ec0242910c36)
- [devocean - Spring Boot 컨테이너 이미지 사이즈 경량화 방법](https://devocean.sk.com/search/techBoardDetail.do?ID=165369&boardType=&query=jlink&searchData=&page=&subIndex=&idList=)
