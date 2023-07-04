---
layout: post
title: Spring Security (1) 맛보기
categories:
- Framework
- Spring
- Sprirng Security
tags:
- Spring
- Spring Security
date: 2023-07-04 14:32 +0900
---
## Intro

취업 준비 해야할 타이밍에 Spring Security 공부하는게 맞나 싶기는 하지만, 팀 프로젝트에 들어가면서 회원 관리 기능을 맡게되어 Spring Security를 안할 수는 없게 됐습니다.

공부를 안하고도 예제를 적당히 수정해가면서 기능 구현은 가능해서 조원들을 위해 로그인/로그아웃/회원가입이 작동은 하게 구현해놨지만, 다른 기능을 추가하려고 하니 어디서부터 건드려야될지 감이 잘 안잡혀서 Spring Security 구조부터 차근차근 살펴보는게 좋을 것 같아 따로 공부를 시작했습니다. 저와 비슷한 수준에 있는 분들한테 도움이 될 것 같아 공유 겸 글 작성을 시작합니다.

## 일단 시작해서 맛보기

보시는 분은 저처럼 Spring Boot는 사용해봤지만 Spring Security는 처음 사용해보는 것이라 가정하고 일단 프로젝트를 시작해보겠습니다.

### 의존성 라이브러리 추가

Spring Boot 3.1.1에 Spring Web 그리고 Spring Security 의존성을 추가한 프로젝트를 만듭니다.

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- 나머지 생략 -->
</dependencies>
```

그런데 보안을 위해 추가한 Spring Security 최신 버전에 취약점(vulnerability)이 있습니다.

![Spring Security 에 표시된 취약점 2개](/assets/img/2023-07-04-study-spring-security-1-just-do-it/01-vulnerability-in-spring-security.png)

찝찝하긴 하지만 현재 저에겐 대안이 없으니 진행해 봅니다.

### RestController

특별할 것 없이 문자열 Hello를 반환하는 RestController 를 추가합니다.

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello";
    }

}
```

### 웹 브라우저에서 HTTP GET Request

프로젝트 실행 후에 브라우저에서 http://localhost:8080/hello 를 입력해보면 Spring Web 만 사용할 때와는 다르게 로그인 화면이 나타나는 것을 볼 수 있습니다. Spring Security에서 기본적으로 지원해주는 로그인 양식입니다.

![Spring Security 라이브러리가 기본적으로 지원하는 로그인 양식](/assets/img/2023-07-04-study-spring-security-1-just-do-it/02-please-sign-in.png)

Spring Security에 기본적으로 설정된 username은 `user` 이고, password는 프로젝트 실행 시에 Command Line에 표시된 password 입니다. 이 password 는 프로젝트를 시작할 때마다 변경됩니다.

```
...
Using generated security password: 0c182e07-6d08-46ba-bc81-909c418e8a96

This generated password is for development use only. Your security configuration must be updated before running your application in production.
...
```

Username 에는 user, Password 에는 Command Line 에서 표시된 password 를 입력한 후 `Sign in` 버튼을 누르면 hello 문자열이 표시된 것을 볼 수 있습니다.

![웹 브라우저에 표시된 hello 문자열](/assets/img/2023-07-04-study-spring-security-1-just-do-it/03-hello-in-web-browser.png)

### Postman 을 이용한 HTTP GET Request

RestController 를 사용하면서 브라우저 주소창에 직접 입력할 일은 없겠죠.

일단 Postman을 이용해서 http://localhost:8080/hello 에 GET Request 를 보내보면, 응답 코드가 401 - Unauthorized 로 표시되는 것을 볼 수 있습니다.

![Postman 을 이용한 GET 요청으로 받은 401 응답 코드](/assets/img/2023-07-04-study-spring-security-1-just-do-it/04-401-in-postman.png)

로그인 화면도 없는데 어떻게 해야 할까요?

`Authorization` Header 에 Base 64로 인코딩해서 추가해줘야 합니다.

![인코딩 웹 사이트에서 Base 64로 인코딩하는 화면](/assets/img/2023-07-04-study-spring-security-1-just-do-it/05-encode-user-cridential.png)

[https://www.base64encode.org/](https://www.base64encode.org/)

위 화면과 같이 `user:0c182e07-6d08-46ba-bc81-909c418e8a96` 양식으로 인코딩하면 아래와 같은 값이 나옵니다.

```
dXNlcjowYzE4MmUwNy02ZDA4LTQ2YmEtYmM4MS05MDljNDE4ZThhOTY=
```

인코딩된 위 값을 Authorization header에 추가해서 보내면 Hello 문자열을 응답으로 받은 것을 볼 수 있습니다.

```
Authorization: Basic dXNlcjowYzE4MmUwNy02ZDA4LTQ2YmEtYmM4MS05MDljNDE4ZThhOTY=
```

![Postman에 Authorization header 추가 후 요청 결과](/assets/img/2023-07-04-study-spring-security-1-just-do-it/06-200-in-postman.png)

> Authorization header 를 위와 같이 작성한 이유가 궁금하시면, 아래 MDN 문서를 참고해주세요.
> 
> [MDN 문서 링크](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization#basic_authentication)

## 하지만 훤히 잘 보이는 정보

여기저기 블로그 글을 살펴보다 보면 Spring Security 를 의존성 라이브러리 목록에 추가하는 순간 많은 보안 문제를 해결해줄 것 같지만 그렇지는 않습니다.

아래처럼 같은 네트워크에 있으면 누구나 사용할 수 있는 툴([Wireshark](https://www.wireshark.org/))을 이용해서 손쉽게 사용자 정보를 얻을 수 있습니다.

![Wireshark를 이용하여 패킷을 확인한 결과](/assets/img/2023-07-04-study-spring-security-1-just-do-it/07-wireshark.png)

>참고로 앞서 MDN 문서를 보시면 basic authentication 방식은 항상 HTTPS 를 이용하여 추가적인 보안 조치를 할 것을 권장하고 있습니다.

Spring Security 는 Spring 사용 시 보안 조치를 편하게 할 수 있는 도구라 생각해야지, 자동으로 보안 조치를 해주는 마법이라 생각하면 안되겠습니다.

## Outro

시간에 쫓기다보니 평소보다 글을 짧게 끊어서 작성해야겠습니다. Spring Security 말고도 정리해야할게 많은데 우선순위를 정하기가 쉽지 않습니다.
