---
layout: post
title: 배포 연습하면서 발생한 문제들
categories:
- etc
tags:
- JUnit5
- MyBatis
- Spring
date: 2023-08-03 18:21 +0900
---
## Intro

팀 프로젝트 발표 시기가 다가와서 슬슬 발표용으로 서버에 배포를 해야 했습니다. 다른 환경에서 실행시킨다는게 굉장히 골치 아픈 일이라는걸 경험해볼 수 있었습니다.

이전에도 Linux와 Windows 에서 파일이름 대소문자 처리가 다른 문제 때문에 데이터베이스 오류가 나서 글([링크](https://limvik.github.io/posts/mysql-error-code-1824/))을 작성했었는데, 다른 문제들도 간단하게 정리해놓을 겸 작성해 봅니다.

## 문제1. JUnit5 에서 @Disabled 가 동작하지 않음

Jenkins 때문에 긴장했었는데, [Jenkins를 활용한 스프링부트 앱 CI/CD 쉽게 시작하기](https://devocean.sk.com/blog/techBoardDetail.do?ID=163889) 라는 글을 남겨주신 분 덕분에 일단 빌드를 시작할 수 있게는 만들었습니다. 하지만 빌드를 시작하니 보이는 것은 테스트 실패 메시지...

> **Task :test**
>
>UserControllerTest > testSignupPage() `FAILED`
>
>UserControllerTest > testLoginPage() `FAILED`

각 메서드에 @Disabled 로 표시해놨었는데, 빌드 시에 테스트가 실행되면서 FAILED가 됐습니다. 팀 프로젝트할 때 테스트를 적극적으로 안하고 초반에 잠깐 작성했더니, 배포하면서 문제가 발생했습니다.

```java
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    @Disabled
    public void testLoginPage() throws Exception {
        mockMvc.perform(get("/user/login"))
                .andExpect(status().isOk())
                .andExpect(view().name("login"))
                .andExpect(content().string(containsString("hello")));
    }

    @Test
    @Disabled
    public void testSignupPage() throws Exception {
        mockMvc.perform(get("/user/signup"))
                .andExpect(status().isOk())
                .andExpect(view().name("signup"))
                .andExpect(content().string(containsString("hello")));
    }
}
```

원래 안되던 테스트라 @Disabled 로 표시해놨기 때문에, 그냥 지워버리면서 해결하기는 했지만, Spring 관련 테스트를 작성하는 능력이 부족해서 그런 것으로 예상됩니다.

아래와 같이 @SpringBootTest 를 추가하면 테스트가 수행되지 않습니다.

```java
@SpringBootTest
@WebMvcTest(UserController.class)
public class UserControllerTest {
```

하지만 또다른 문제가 발생합니다.

![테스트는 무시되었지만, 예외 발생한 화면](/assets/img/2023-08-03-problems-in-deploy-exercise/01-disabled-test.png)

```
java.lang.IllegalStateException: Configuration error: found multiple declarations of @BootstrapWith for test class
```

Spring 테스트 코드 작성하는 법을 배우는게 문제 원인을 파악하는 지름길이 되겠습니다.

## 문제2. MyBatis에서 사용할 수 있는 Parameter 이름이 다름

팀 프로젝트에서 MyBatis를 사용 중 입니다.  그리고 DTO를 만들지 않고서 불러오는 경우가 있습니다.

```java
// repository
List<Map<String, String>> ㅎㅎ(long userId, long page);
```

```xml
<-- mapper -->
<select id="ㅎㅎ" parameterType="long" resultType="java.util.HashMap">
    SELECT id, title, created_datetime AS datetime
    FROM contents
    WHERE user_id = #{userId}
    ORDER BY created_datetime DESC LIMIT #{page}, 10
</select>
```

그런데 이렇게하면 오류가 발생합니다. 참고로 Windows 11 환경입니다.

![Windows 에서 파라미터명과 동일한 이름을 mapper에 사용시 발생하는 MyBatis 오류 화면](/assets/img/2023-08-03-problems-in-deploy-exercise/02-mybatis-in-windows.png)

그래서 arg0 와 arg1을 사용하면, Linux(Cent OS 7.9)에서는 아래와 같은 오류가 발생합니다.

![Linux(Cent OS) 에서 파라미터명과 동일한 이름을 mapper에 사용시 발생하는 MyBatis 오류 화면](/assets/img/2023-08-03-problems-in-deploy-exercise/03-mybatis-in-linux.png)

왜 그런지 궁금하긴 하지만, 깊게 파보는게 큰 영양가는 없을 것 같은 관계로 어느쪽이든 동일하게 사용 가능한 param1, param2 를 사용하여 수정했습니다.

정말 생각지도 못한데서 오류가 발생했습니다.

## 문제3. @PropertySource가 동작을 안함

로컬에서 일부 API 키만 별도로 api.properties 라는 파일을 만들어서 사용하고 있었습니다.

```java
@Configuration
@PropertySource("classpath:api.properties")
@Getter
public class ApiConfig {

    @Value("${api.captcha.client-id}")
    private String apiCaptchaClientId;

    @Value("${api.captcha.client-secret}")
    private String apiCaptchaClientSecret;
}
```

그런데 이게 Linux 환경으로 옮기고 부터는 api.properties 파일은 인식하는데 값을 불러오지를 못합니다.

마찬가지로 원인이 굉장히 궁금하지만, 꼭 properties를 나누어 써야만 하는 것은 아니라서 application.properties 로 API 키를 옮겨서 @PropertySource는 제거하였습니다.

혹시나해서 encoding 도 utf-8으로 지정해 봤지만...! 실패...

```java
@PropertySource(value = "classpath:api.properties", encoding = "utf-8")
```

## 문제4. 배포 시 API키는 어디에...?

AWS 는 키만 따로 관리할 수 있는 서비스가 있다고 들은 것 같은데, ncloud 에는 별도의 서비스가 보이지 않습니다.

그래서 properties 에 변수명을 사용하면 OS 에 저장한 환경 변수 값을 불러올 수 있다는 것을 어디선가 본적이 있어서 이 방법을 사용하기로 했습니다.

### spring-dotenv

먼저 로컬에서도 변수를 사용할 수 있도록 spring-dotenv를 사용합니다.

Spring-dotenv([github 링크](https://github.com/paulschwarz/spring-dotenv), [maven repository 링크](https://mvnrepository.com/artifact/me.paulschwarz/spring-dotenv/4.0.0))를 라이브러리를 추가합니다.

gradle 사용하는 경우,

```groovy
implementation 'me.paulschwarz:spring-dotenv:4.0.0'
```

maven 사용하는 경우,

```xml
<dependency>
    <groupId>me.paulschwarz</groupId>
    <artifactId>spring-dotenv</artifactId>
    <version>4.0.0</version>
</dependency>
```

다음으로 `resources` 디렉토리에 `.env ` 파일을 생성하고 `변수명=값` 과 같은  형식으로 작성하면, 변수명으로 지정한 값을 application.properties에서 불러올 수 있습니다. 이때 `.env` 파일을 `.gitignore` 에 추가하는걸 잊지 않아야 합니다.

MySQL 의 `.env` 파일을 예로 들어보면 아래와 같습니다.

```properties
#.env 파일
SPRING_DATASOURCE_URL=jdbc:mysql://ip주소:port번호/db이름
SPRING_DATASOURCE_USERNAME=사용자이름
SPRING_DATASOURCE_PASSWORD=비밀번호
SPRING_DATASOURCE_DRIVER=com.mysql.cj.jdbc.Driver
```

다음으로 application.properties 파일을 아래와 같이 수정합니다.

```properties
#application.properties 파일
spring.datasource.url=${SPRING_DATASOURCE_URL}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD}
spring.datasource.driver-class-name=${SPRING_DATASOURCE_DRIVER}
```

### 서버 환경 변수 추가하기

마지막으로 배포 서버(Linux)의 터미널에서 아래와 같은 명령어를 시작해주면, 배포된 서버에서는 `.env` 파일 없이도 값을 그대로 사용할 수 있습니다.

```bash
export SPRING_DATASOURCE_URL=jdbc:mysql://ip주소:port번호/db이름
export SPRING_DATASOURCE_USERNAME=사용자이름
export SPRING_DATASOURCE_PASSWORD=비밀번호
export SPRING_DATASOURCE_DRIVER=com.mysql.cj.jdbc.Driver
```

그런데 단순히 이렇게 입력만 하면 서버 재부팅 시에 값이 사라집니다.

수업 시간에는 CentOS 사용 시 /etc/profile 내에 추가하는 것으로 배웠는데, 검색해보면 대체로 /etc/bash.bashrc 에 추가하는게 검색 결과로 많이 보입니다. 환경 변수 추가하는 방법을 정리해놓으신 분의 글([링크](https://hec-ker.tistory.com/302))을 보면, 차이가 있기는 한데 이 차이가 어떠한 영향을 미치는지는 따로 찾아봐야할 것 같습니다.

제가 배운 방식을 적어놓겠습니다.

1. `vim /etc/profile` 명령어 입력
2. 빈 공간에 export 명령어와 함께 `변수명=값` 양식으로 추가(예. export SPRING_DATASOURCE_URL=jdbc:mysql://ip주소:port번호/db이름)
3. 저장(:wq) 후 bash에서 `source /etc/profile` 명령어 입력하여 환경 변수 적용

그러면 재부팅 시에도 명령어에 따라 환경 변수가 자동으로 추가됩니다.

## Outro

막상 쓰고보니 별거 없는데, 쓸데 없이 시간 쓴 것 같은 느낌도 듭니다.

정리해보자면 테스트 코드 작성 능력이 부족하고, 환경 변화로 인한 문제를 방지하려면, 개발 시작 부터 Docker 환경에서 개발하는게 낫지 않을까 싶습니다. 문제는 Docker를 한 번 써본게 전부라는 점인데... 시작한지 얼마 안된 프로젝트를 바로 Docker로 해보는게 좋겠습니다.
