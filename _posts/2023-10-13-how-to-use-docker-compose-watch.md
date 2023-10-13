---
layout: post
title: Docker Compose Watch를 이용한 로컬 코드 변경 실시간 반영 방법
categories:
- Tools
- Docker
tags:
- Docker
- Spring
- Spring Boot
date: 2023-10-13 13:36 +0900
---
## Intro

Docker 초보라 그런지 개발할 때 Docker Compose 로 실행 중인 컨테이너에 로컬 코드 변경사항을 바로 반영하는게 쉽지 않습니다.

그러다 매뉴얼에서 [Compose Watch](https://docs.docker.com/compose/file-watch/)를 보게되어 사용해봤는데, 설정방법도 쉽고 편리해서 공유합니다.

Docker 초보다 보니 초보자의 눈높이에서 매뉴얼에 나온 내용을 간단히 살펴보면서 사용방법 위주로 작성해 보겠습니다. 뒷 부분에서는 Spring Boot 기반으로 일부 기능만 사용한 예제를 작성하지만, 예제를 참고해서 매뉴얼 보시면 어렵지 않게 적용하실 수 있을거라 생각됩니다. Docker 잘 하시는 분들은 바로 매뉴얼로 직행하시는걸 추천드립니다.

## Compose Watch 매뉴얼 살펴보기

간단하게 매뉴얼에 있는 내용을 살펴보겠습니다.

### 사용 가능 버전

2.22(2023-08-03 release) 버전 이상에서 동작한다고 적혀있습니다.

>Compose Watch is available in Docker Compose version 2.22 and later.

release note에는 [2.24.0(2023-09-28)](https://docs.docker.com/desktop/release-notes/#4240) 버전 부터 모든 사용자가 사용 가능하다고 명시되어 있습니다.

> Compose Watch is now available to all users. For more information, see  [Use Compose Watch](https://docs.docker.com/compose/file-watch/).

나온지 얼마 안된 따끈따끈한 기능입니다.

### path rules

watch를 compose 파일에서 사용하려면 아래와 같은 경로 규칙(path rules)을 고려해야 합니다. 

`watch`  adheres to the following file path rules:

-   All paths are relative to the build context
-   Directories are watched recursively
-   Glob patterns aren't supported
-   Rules from  `.dockerignore`  apply
    -   Use  `include`  /  `exclude`  to override
    -   Temporary/backup files for common IDEs (Vim, Emacs, JetBrains, & more) are ignored automatically
    -   `.git`  directories are ignored automatically

항목 중 몇 가지를 살펴보겠습니다.

#### Directories are watched recursively

아래서 보겠지만, 변경을 감지할 경로를 직접 지정해야합니다. `Directories are watched recursively` 의 의미는 compose 파일에 지정한 디렉터리를 재귀적으로 감시한다는 의미가 되겠습니다.

#### Glob patterns aren't supported

Dockerfile 등에서 아래와 같이 `*` 와 같은 패턴을 통해 모든 파일을 지정하는 등의 패턴은 지원하지 않는다는 것을 의미합니다.

```dockerfile
ARG JAR_FILE=build/libs/*.jar
```

path rules는 이정도 살펴보고 다음으로 넘어가겠습니다.

### Compose Watch versus bind mounts

저는 bind mounts 기능을 사용해본적이 없고(여기서 처음 봤고...), Watch 사용법만 집중하기 위해 굳이 찾아보면서 자세한 내용을 적지는 않겠습니다.

매뉴얼에서는 Compose Watch는 세부적으로 디렉터리를 지정할 수 있고, 특정 파일이나 디렉터리는 무시할 수 있어서 컨테이너 환경에서 개발할 때 bind mounts 보다 더 적합함을 이야기하고 있습니다.

### Configuration

이제 설정 방법을 살펴봅니다.

개인적으론 예제를 먼저 보는게 더 나은 것 같아서 매뉴얼의 compose 파일 예제를 가져오겠습니다.

아래는 프로젝트의 로컬 경로입니다.

```
myproject/
├── web/
│   ├── App.jsx
│   └── index.js
├── Dockerfile
├── compose.yaml
└── package.json
```

그리고 compose 파일 입니다.

```yaml
services:
  web:
    build: .
    command: npm start
    develop:
      watch:
        - action: sync
          path: ./web
          target: /src/web
          ignore:
            - node_modules/
        - action: rebuild
          path: package.json
```

`develop` 속성(attribute) 안에 `watch` 속성이 있고, 그 안에 여러 속성이 있는 것을 볼 수 있습니다.

앞서봤던 path rules는 당연히 `path` 속성에 적용됩니다. watch는 예제에서 보시는바와 같이 두 가지 `action`을 할 수 있습니다. 각각 살펴보겠습니다.

#### sync

`sync`는 지정된 파일 또는 디렉터리 내의 파일이 변경되면 바로 반영하는 action입니다. 프론트엔드 작업을 할 때 `핫 리로드(Hot Reload)` 와 같은 기능을 통해 실시간 반영하는데 사용하기 좋습니다. 꼭 핫 리로드가 아니더라도 빌드가 필요하지 않은 파일이 변경됐을 때 바로 반영하는데 사용하기 좋아보입니다.

`path` 속성에는 로컬 경로를 지정해줍니다. 이전에 path rules 에 나온대로 Glob pattern을 사용할 수는 없고, 디렉토리를 지정하면 디렉터리 내부의 모든 파일에 대해 변경사항을 감시하고 반영합니다.

`target` 속성은 컨테이너에서의 경로를 의미합니다. 초보라면 Dockerfile 에서 COPY 명령어를 통해 로컬 파일을 복사해온 경로라 이해하면 쉬울 것 같습니다.

마지막으로 `ignore` 속성은 굳이 설명을 안드려도 되겠지만, 변경이 발생해도 반영하지 않을 경로입니다. 매뉴얼 예제에 나온 경로는 Node.js와 관련된 경로라고 합니다.

#### rebuild

`rebuild`는 말 그대로 build를 다시하는 action입니다. 프로젝트를 다시 build 해주는 것은 아니고, `docker image를 다시 build`합니다.

`path` 속성은 마찬가지로 변경을 감시할 파일 또는 디렉터리 내의 파일을 지정하는 속성입니다.

예제에 따라 수동으로 작업을 한다면, `package.json`을 변경한 후에  `docker compose up --build web` 을 실행한 것과 같습니다.

### User Watch

compose 파일에서 watch 섹션을 작성했다면, `docker compose watch` 명령어를 사용하여 실행할 수 있습니다.

이제 매뉴얼을 통해 기본적인 사용법은 살펴봤으니, Spring Boot 예제로 살펴보겠습니다.

## Compose Watch with Spring Boot

### Dockerfile

Compose Watch 사용법하고는 상관 없지만, 일단 compose 파일에서 Dockerfile을 사용하여 추가합니다.

```dockerfile
FROM eclipse-temurin:17 AS builder
LABEL authors="limvik"

WORKDIR workspace
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} limvik.jar
RUN java -Djarmode=layertools -jar limvik.jar extract

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

### compose.yaml

compose 파일을 보시면 watch 기능을 필요한 서비스에만 지정하고 있습니다.

```yaml
version: "3.8"  
  
services:  
  mysql:  
    image: 'mysql:8.0.34'  
    container_name: limvik_db  
    restart: always  
    environment:  
      - 'MYSQL_DATABASE=blog'  
      - 'MYSQL_PASSWORD=password'  
      - 'MYSQL_ROOT_PASSWORD=rootpassword'  
      - 'MYSQL_USER=limvik'  
    command:  
      - --character-set-server=utf8mb4  
      - --collation-server=utf8mb4_unicode_ci  
    ports:  
      - '33306:3306'  

  springboot:  
    build:  
      context: .  
      dockerfile: Dockerfile  
    restart: always  
    container_name: blog_app  
    develop:  
      watch:  
        - action: rebuild  
          path: ./build/libs  
    depends_on:  
      mysql:  
        condition: service_started  
    ports:  
      - '9001:9001'  

  adminer:  
    image: adminer  
    restart: always  
    ports:  
      - '8081:8080'
```

이 예제에서는 action을 `rebuild`만 사용하고 있습니다. JSP나 Thymeleaf 등으로 프론트 작업을 한다면, static 파일이 있는 경로에 `sync`를 사용하면 됩니다.

여기서 저는 빌드된 jar 파일이 저장되는 경로인 `./build/libs`를 지정하여, 빌드 도구를 통해 Spring Boot 프로젝트 빌드가 완료되면 Docker image를 다시 빌드하도록 지정하였습니다.

### Use Watch

제가 Watch를 사용할 때 입력하는 명령어를 살펴보겠습니다.

먼저 watch를 실행시키는 명령어입니다.

> docker compose -p limvik watch

-p 를 통해 process 이름을 지정해주고 있습니다. 아직 기능이 조금 불안정해서 그런지 compose down을 해도 다시 실행할 때 동일한 process 이름이 있다고 하면서 실행이 안될때가 있어서 process 이름을 바꿔가면서 실행해야할 때가 있습니다.

![process still running](/assets/img/2023-10-13-how-to-use-docker-compose-watch/01.process-still-running.png)

명령어를 실행시키고 나면, 다음과 같이 감시하고있는 경로가 표시됩니다.

![watching directories](/assets/img/2023-10-13-how-to-use-docker-compose-watch/02.command-watch.png)

코드를 변경한 후에 `gradle build` 명령어 등을 통해 새롭게 빌드된 파일로 변경하면, 아래와 같이 자동으로 감지하고 Docker image를 `rebuild` 합니다.

![rebuild](/assets/img/2023-10-13-how-to-use-docker-compose-watch/03.gradlew-build.png)

`gradle clean build`와 같이 clean을 먼저 해버리면 빌드된 jar 파일이 없는 상태가 감지되므로, 주의해야 합니다. clean build 를 하여 오류가 발생하면, 다음 변경 사항이 생겨도 반영이 안돼서 compose를 다시 실행시켜주어야 합니다. 아직 기능이 불안정한 것 같습니다.

종료 시에는 아래와 같이 up을 했을 때와 동일하게 down 명령어를 사용할 수 있습니다.

> docker compose -p limvik down

## Outro

compose watch가 나온지 얼마 안돼서 불안정한 느낌도 있지만, 개인적으로는 사용함으로써 얻는 편리함이 더 큰 것 같습니다.

특히나 시각적으로 결과물을 자주 확인해야 하는 프론트작업을 할 때, 더욱 유용할 것 같습니다.

컨테이너에 실시간 변경 사항 반영 때문에 고민하고 계시던 Docker 초보 동지분들은 적용해보시는걸 추천드립니다.
