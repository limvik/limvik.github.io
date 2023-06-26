---
layout: post
title: Spring Boot 3.x 에서 swagger-generator 6.6.0 사용하는 방법
categories:
- Framework
- Spring
tags:
- Spring
- Swagger
- Gradle
date: 2023-06-26 23:45 +0900
---
## Intro

새로운 팀 프로젝트는 Spring Boot 3 버전대에서 시작할 예정이라 swagger를 Spring Boot 3 버전에서 설치해 봤습니다. 보고있던 책([링크](https://www.packtpub.com/product/modern-api-development-with-spring-and-spring-boot/9781800562479))이 2021년에 출간된 책이다보니 책에 나온 swagger 관련 라이브러리들이 구버전이기도 했고, 최신 버전의 라이브러리들도 Jakarta EE를 지원하지 않아 Java EE 라이브러리 추가가 필요했습니다. 그래서 간단하게 추가로 필요한 Java EE 라이브러리를 기록해 봤습니다.

참고로 Github 상의 Milestone을 보면 7월 말 즈음 릴리즈 될 openapi-generator 7.0.0 에서는 jakarta EE 로 교체 예정입니다. 그때까지 임시로 사용하시려면 참고하시면 되겠습니다.

## build.gradle

책에 있는 build.gradle 파일([링크](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-and-Spring-Boot/blob/main/Chapter03/build.gradle))은 3년 전이 마지막 업데이트라 책에 나왔던 라이브러리도 모두 글 쓰는 시점의 최신 버전으로 설정하였습니다.

아래 build.gradle 파일의 [github 링크](https://github.com/limvik/practice/blob/main/api/build.gradle)도 첨부 합니다.

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.1'
    id 'io.spring.dependency-management' version '1.1.0'
    id 'org.hidetake.swagger.generator' version '2.19.2'
}

group = 'com.limvik'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

swaggerSources {
    def typeMappings = 'URI=URI'
    def importMappings = 'URI=java.net.URI'
    eStore {
        def apiYaml = "${rootDir}/src/main/resources/api/openapi.yaml"
        def configJson = "${rootDir}/src/main/resources/api/config.json"
        inputFile = file(apiYaml)
        def ignoreFile = file("${rootDir}/src/main/resources/api/.openapi-generator-ignore")
        code {
            language = 'spring'
            configFile = file(configJson)
            rawOptions = ['--ignore-file-override', ignoreFile, '--type-mappings', typeMappings,
                          '--import-mappings', importMappings] as List<String>
            components = [models: true, apis: true, supportingFiles: 'ApiUtil.java']
        }
    }
}

compileJava.dependsOn swaggerSources.eStore.code
sourceSets.main.java.srcDir "${swaggerSources.eStore.code.outputDir}/src/main/java"
sourceSets.main.resources.srcDir "${swaggerSources.eStore.code.outputDir}/src/main/resources"

processResources {
    dependsOn 'generateSwaggerCodeEStore'
}

repositories {
    mavenCentral()
}

dependencies {
    // https://mvnrepository.com/artifact/org.openapitools/openapi-generator-cli
    swaggerCodegen 'org.openapitools:openapi-generator-cli:6.6.0'
    // https://mvnrepository.com/artifact/io.swagger.core.v3/swagger-annotations
    implementation 'io.swagger.core.v3:swagger-annotations:2.2.12'
    // https://mvnrepository.com/artifact/org.openapitools/jackson-databind-nullable
    implementation 'org.openapitools:jackson-databind-nullable:0.2.6'
    // https://mvnrepository.com/artifact/javax.validation/validation-api
    implementation 'javax.validation:validation-api:2.0.1.Final'
    // https://mvnrepository.com/artifact/javax.annotation/javax.annotation-api
    implementation 'javax.annotation:javax.annotation-api:1.3.2'
    // https://mvnrepository.com/artifact/javax.xml.bind/jaxb-api
    implementation 'javax.xml.bind:jaxb-api:2.3.1'
    // https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api
    compileOnly 'javax.servlet:javax.servlet-api:4.0.1'

    implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-xml'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-hateoas'
    compileOnly 'org.springframework.boot:spring-boot-starter-validation'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

저자분은 gradle plugin 중에 [openapi-generator-gradle-plugin](https://github.com/OpenAPITools/openapi-generator/tree/master/modules/openapi-generator-gradle-plugin)은 github issue 수가 많은 것 대비 해결이 안돼서 [gradle-swagger-generator-plugin](https://github.com/int128/gradle-swagger-generator-plugin) 을 사용한다고 하셨습니다. 그런데 gradle-swagger-generator-plugin 은 사용하는 사람이 적어서 issue가 적은거 아닌가 싶은 생각이 들기는 하지만, 아직 판단이 안서서 그대로 따라 사용했습니다.

그리고 버전 외에 책 예제와 다르게 추가된 것만 따로 아래 정리해보면 아래와 같습니다.

```groovy
processResources {
    dependsOn 'generateSwaggerCodeEStore'
}

dependencies {
    // https://mvnrepository.com/artifact/javax.validation/validation-api
    implementation 'javax.validation:validation-api:2.0.1.Final'
    // https://mvnrepository.com/artifact/javax.annotation/javax.annotation-api
    implementation 'javax.annotation:javax.annotation-api:1.3.2'
    // https://mvnrepository.com/artifact/javax.xml.bind/jaxb-api
    implementation 'javax.xml.bind:jaxb-api:2.3.1'
    // https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api
    compileOnly 'javax.servlet:javax.servlet-api:4.0.1'
}
```

processResources 안에 dependsOn으로 설정해준 것은 Gradle 8.0 부터는 작업 의존성을 필수적으로 명시해주어야 한다고 해서 추가했습니다.

그리고 dependencies를 보면 다 Java EE 관련 라이브러리 입니다. 나름 java EE 에서는 최신 버전으로 추가했습니다.

계획 상으로는 7월 말즈음 릴리즈([Milestone 링크](https://github.com/OpenAPITools/openapi-generator/milestones)) 될 openapi-generator 7.0.0 에서는 Spring Boot 3.x와 Jakarta 라이브러리를 지원할 것으로 예상됩니다([관련 PR 링크](https://github.com/OpenAPITools/openapi-generator/pull/13620)).

## config.json

plugin 설정 파일로 config.json 을 사용하는데, 책에 나온 것으로는 안돼서 plugin readme 파일에 나온 것도 봤는데 책과 동일하게 문제의 원인이었던 library가 spring-mvc로 되어있었습니다.

```json
{
  "library": "spring-mvc",
}
```

그래서 이걸 spring-mvc 에서 spring-boot로 변경해주면 됩니다.

```json
{
    "library": "spring-boot",
}
```

수정한 파일의 전체 내용은 Github([링크](https://github.com/limvik/practice/blob/main/api/src/main/resources/api/config.json)) 참고해 주시고, library만 수정하면 되니까 굳이 전체 내용을 확인하실 필요는 없을 것 같습니다.

이렇게 의존성 추가하고 몇 가지 수정만 하면 Spring Boot 3.x 에서도 swagger가 이상 없이 동작하는 걸 확인했습니다.

## 초보자(나)를 위한 몇 가지 정리

### gradle build 시 주의 사항

maven만 쓰다가 gradle 쓰면서 쓸데없이 시간 버린게 있어서 정리합니다.

gradlew 라고 시작하면서 명령어를 제시해줄 때가 있는데, cli에서 실행하는 명령어가 아니라 스크립트 파일 이름입니다. 그래서 프로젝트 디렉터리에 가서 ./gradlew 이렇게 시작해야 합니다.

그리고 maven 쓰다 넘어오면 clean `install` 을 입력하면서 왜 안되지 할텐데, gradle은 다릅니다.

```
./gradlew clean build
```

뒤에 `build` 입니다. 저 같은 사람이 많았는지 꽤 질문이 많더군요.

### RESTful API, Swagger 그리고 OAS

책을 보기 전엔 사람들이 왜이렇게 swag을 찾나 했는데, API 만들려고 그랬던걸 이제서야 알았습니다.

RESTful API는 Roy Thomas Fielding 이라는 분이 거의 200페이지에 가까운 논문([링크](https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf))을 쓰시면서 등장합니다.

그런데 논문이다 보니 어떤 하나의 방식을 제시한 것이지 이렇게 해야만 한다는 표준을 만든게 아닙니다.

표준이 없다보니 관련 업체들이 멋대로들 사용하다가 같이 합의보고 Open API Initiative([링크](https://www.openapis.org/))를 만듭니다. 그리고 이때 Initiative 를 만들면서 Swagger 2.0 Specification 을 OpenAPI Specification (OAS) 로 이름을 변경합니다([관련 내용 링크](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#open-api-initiative)). 그리고 현재 글을 쓰는 시점에는 OAS 3.1 까지 나왔습니다.

표준 규격이 정해지니 자동으로 코드까지 생성해주는 툴이 나오게 된겁니다.

3.0과 3.1 사이에도 차이점이 꽤 있어서 무슨 버전을 써야하는지 초보자는 또 혼란에 빠집니다.

### swagger 관련 라이브러리 설명

GPT-3.5 의 도움을 받아 참고용으로 붙여넣습니다.

각각의 의존성 라이브러리가 하는 기능은 다음과 같습니다:

1. `org.openapitools:openapi-generator-cli:6.6.0`
   - OpenAPI Generator CLI 라이브러리입니다.
   - OpenAPI Generator는 OpenAPI(Swagger) 스펙으로부터 API 클라이언트, 서버, 문서 등을 자동으로 생성하는 도구입니다.
   - 이 라이브러리는 OpenAPI Generator CLI를 사용할 수 있도록 제공합니다.

2. `io.swagger.core.v3:swagger-annotations:2.2.12`
   - Swagger 어노테이션 라이브러리입니다.
   - Swagger 어노테이션은 Java 코드에서 Swagger 스펙을 정의하기 위해 사용됩니다.
   - 예를 들어, `@Api`, `@ApiOperation`, `@ApiModel` 등의 어노테이션을 사용하여 API 엔드포인트, 매개변수, 모델 등을 정의할 수 있습니다.
   - Swagger 어노테이션을 사용하면 Swagger UI와 같은 도구에서 자동으로 API 문서를 생성할 수 있습니다.

3. `org.openapitools:jackson-databind-nullable:0.2.6`
   - Jackson DataBind Nullable 라이브러리입니다.
   - Jackson은 Java 객체와 JSON 간의 직렬화 및 역직렬화를 처리하는 데 사용되는 강력한 라이브러리입니다.
   - 이 라이브러리는 Jackson에 Nullable 기능을 추가합니다.
   - Nullable 어노테이션을 사용하여 필드가 null일 수 있음을 명시할 수 있으며, 직렬화 및 역직렬화 시에도 null 값을 올바르게 처리할 수 있습니다.
   - 이는 OpenAPI(Swagger) 스펙과 같이 null이 허용되는 필드를 처리하는 데 유용합니다.

## Outro

근데 이번 팀 프로젝트 때문에 준비했지만 이번 팀 프로젝트에는 사용하지 못할 것 같습니다. 초보자 팀이라 OAS를 살펴보다가 팀원 분들이 사라지실 것 같아서 제 개인 프로젝트에 사용해보는 걸로 결정했습니다. 팀 프로젝트 끝나갈 때 즈음 이런게 있다고 소개해주는게 좋을 것 같습니다.

이전에 프론트엔드 팀 프로젝트에서 열심히 하시긴 했는데, 제일 힘들어하시던 분이 아무 소식도 전하지 않고 사라져 버리셔서 꽤나 충격이었습니다. 이번 팀 프로젝트에는 팀원 분들이 사라지시더라도 취업 등 좋은 이유로 사라지시길 바라봅니다.
