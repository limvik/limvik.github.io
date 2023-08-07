---
layout: post
title: Spring Boot로 REST API 만들어보기 (2)-2 로그인
categories:
- API
- REST
tags:
- Spring
- Spring Security
- JPA
- JWT
- JUnit5
date: 2023-08-07 21:38 +0900
---
## Intro

JWT가 어떻게 생겼는지 살펴봤으니([링크](/posts/making-rest-api-with-spring-boot-2-1-jwt/)), 이어서 로그인 과제를 구현해 봐야겠습니다. 요구사항부터 다시 봐야겠습니다.

역시나 글은 난장판이므로, 코드가 필요하신 분은 Github([링크](https://github.com/limvik/wanted-pre-onboarding-backend))을 참고해 주세요.

## 요구사항

### 과제 2. 사용자 로그인 엔드포인트

- 사용자가 올바른 이메일과 비밀번호를 제공하면, 사용자 인증을 거친 후에 JWT(JSON Web Token)를 생성하여 사용자에게 반환하도록 해주세요.
- 과제 1과 마찬가지로 회원가입 엔드포인트에 이메일과 비밀번호의 유효성 검사기능을 구현해주세요.

다시보니 두번째 요구사항은 로그인에 실패한 경우 중에 하나로, 크게 보면 첫번째 요구사항에 포함되는 요구사항이라고 볼 수 있겠습니다.

간단하게 Sequence Diagram으로 과정을 그려보면서 시작해보겠습니다.

### Sequence Diagram

로그인에 성공하는 아름다운 경우만 그려봤습니다.

![로그인 요구사항 Sequence Diagram](/assets/img/2023-08-07-making-rest-api-with-spring-boot-2-2-login/01-login-sequence-diagram.png)

성공한 경우만 그렸더니 별거 없어보이지만, 당연히 별거가 많겠죠.

## 구현 자료 찾아보기

이전에 Spring Security 를 조금이나마 공부해서 로그인 요청 -> Filter -> Authentication(Token) 객체 생성 -> AuthenticationManager 를 이용한 인증 및 인가-> 인증 성공 시 SecurityContextHolder에 저장 순으로 흘러가겠다는 것은 대략적으로 알 수 있습니다.

그런데 구체적인 과정은 감이 안옵니다. 만들려면 이미 있는 구현체들 참고해서 만들수야 있겠지만, 뭔가 빼먹을 가능성이 높기 때문에 다른 분들의 JWT 로그인 구현 과정을 참고하는게 좋겠습니다.

- [Spring Boot JWT Example](https://www.techgeeknext.com/spring/spring-boot-security-token-authentication-jwt)
- [Spring Security JWT 토큰으로 인증하기](https://petaverse.pe.kr/entry/Spring-Security-JWT-%ED%86%A0%ED%81%B0%EC%9C%BC%EB%A1%9C-%EC%9D%B8%EC%A6%9D%ED%95%98%EA%B8%B0?category=1113161)
- [Spring Boot 2로 Rest api 만들기(8) SpringSecurity를 이용한 인증 및 권한부여](https://www.daddyprogrammer.org/post/636/springboot2-springsecurity-authentication-authorization/#google_vignette)

검색해보니 이미 많은 글이 있습니다. 또 너무 많이 보면 갈팡질팡 할거 같아서, 이정도로 참고하면서 구현해봐야겠습니다.

그럼 먼저 로그인 엔드포인트 경로(path) 부터 지정해 보겠습니다.

## 로그인 엔드포인트 경로 지정

이전에 Logout에서 어쩔 수 없이 리소스(Resource) 자리에 동사를 사용([링크](/posts/how-to-set-endpoint-name-and-http-method-for-logout-rest-api/))하기는 했었는데, 이번에는 클라이언트 입장에서 볼때 Token을 가져오는 거라 동사를 쓰지 않아도 되지 않을까...? 하는 생각이 듭니다.

참고한 자료들에 서는 /authenticate, /signin 등의 동사를 사용했습니다. 토큰을 사용하는 유사한 구조인 OAuth 의 Auth가 Authorization 인 것에 유추해볼 때 /authenticate을 사용하는 것은 좀 아닌 것 같습니다.

OAuth2 구조인 카카오, 네이버 로그인 API 를 참조해보면 아래와 같습니다.

- Authorization URI
	- https://kauth.kakao.com/oauth/authorize
	- https://nid.naver.com/oauth2.0/authorize
- Token URI
	- https://kauth.kakao.com/oauth/token
	- https://nid.naver.com/oauth2.0/token

인가 코드를 받을 때는 동사인 authorize 를 사용해서 받아오고, token을 받을 때는 명사인 token을 사용해서 받아옵니다.

카카오 로그인 API 문서([링크](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api#before-you-begin-process))에 Sequence Diagram 이 잘 그려져 있어서, 요청되는 순서를 확인할 수 있습니다.

![카카오 OAuth2 sequence diagram](/assets/img/2023-08-07-making-rest-api-with-spring-boot-2-2-login/02-kakaologin_sequence.png)

그런데 지금은 OAuth2 구조로 만들고 있는 것은 아니니, 서버는 클라이언트가 HTTP 요청을 통해 전송한 email과 password로 인증(authentication) 절차를 거쳐서 인가(authorization)를 하고, token을 만들어서 응답해줘야 합니다. authorize와 token, 두 개를 동시에 요청하는 거나 다름 없습니다.

여러가지 일을 해야하니 이번에도 아무래도 동사로 가는 수 밖에 없겠습니다. 요구사항에서 로그인이라는 용어를 사용했으니 `login` 으로 하겠습니다.

마지막으로 `/user/login` 을  할 것인가, `/login` 으로 할 것인가 고민이 됩니다. `/login` 으로 꼭 해야될 이유는 찾지 못해서 일단은 기존 방식인 `/user/login` 으로 진행합니다.

카카오 API도 logout 은 `/user/logout` 으로 경로를 지정([문서 링크](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api#logout))하고 있는 것 보니, 나쁜 선택은 아니라는 생각이 듭니다.

## HTTP

경로는 지정했으니 HTTP 요청을 어떻게 구성할 것인지 정해야 겠습니다.

### method

먼저 적합한 HTTP method는 `POST` 밖에 선택지가 없습니다. GET 로 비밀번호를 노출시킬 수는 없고, 수정/삭제를 하는 것도 아니니 `POST` 로 구현합니다.

### 성공 시 response status code

로그인에 성공했을 때 Token을 반환하면서, 서버에서 생성은 했지만, 생성해서 저장한 것은 아니니 201(Created)는 어울리지 않습니다. 

MDN 문서([링크](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200))를 보니, 200(Ok)가 가장 적합합니다.

> The resource describing the result of the action is transmitted in the message body
>
> 작업 결과를 설명하는 리소스가 메시지 본문으로 전송됩니다.

카카오, 네이버 로그인도 200 으로 응답하는 걸 보니, 조금 안심이 됩니다.

### 실패 시 response status code

로그인 실패 시에는 어떤 response status code 를 사용해야 할까요? 별로 신경 안쓰는지 네이버, 카카오 API 문서에도 없습니다.

로그인 실패도 다양한 경우가 있겠지만, 유효성 검사에 실패한 경우 회원가입 시에는 400(Bad Request)로 응답하기로 했습니다. 로그인 시에도 동일하게 처리할 계획입니다.

email이나 password와 같은 자격 증명 정보가 틀린 경우는 고민이 됩니다. FusionAuth 라는 곳의 API 문서를 보면 사용자 정보를 찾을 수 없거나 password를 틀린 경우 404(Not Found) 를 사용합니다. 판단이 안서서 GPT에 물어보니 404(Not Found)는 요청 또는 엔드포인트 자체가 잘못되었음을 의미해서 안좋다고 합니다.

404 의 MDN 문서([링크](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404))를 보면 GPT 가 말한 수준의 의미는 없습니다.

> The HTTP **`404 Not Found`** response status code indicates that the server cannot find the requested resource.
>

User 라는 resource를 찾을 수 없으니 적합하지 않은가 생각이 되지만, 401(Unauthorized) MDN 문서의 내용을 보니 저에게는 401이 가장 적합해 보입니다.

>The HyperText Transfer Protocol (HTTP) **`401 Unauthorized`** response status code indicates that the client request has not been completed because it lacks valid authentication credentials for the requested resource.
>
>HTTP 401 Unauthorized response status code는 요청된 리소스에 대한 유효한 인증 자격 증명이 없기 때문에 클라이언트 요청이 완료되지 않았음을 나타냅니다.

FusionAuth는 Authorization Header에 API Key가 없는 경우도 401 응답으로 하는 것으로 보아 OAuth2 절차로 보입니다. 정상적인 API Key가 포함되있는데, 로그인 정보가 틀릴 경우는 404가 적합해 보이긴 합니다.

그럼 저는 response status code를 아래와 같이 정리합니다.

- 로그인 정보가 유효성 검사를 통과하지 못한 경우: 400(Bad Request)
- 존재하지 않는 email이거나 password를 틀린 경우: 401(Unauthorized)

또 보안기사 공부할 때 email, password 중 뭐가 틀렸는지 정보를 제공하는 것은 공격자에게 유용한 정보를 제공하는게 될 수 있다고 배웠으므로, 단순히 유효하지 않은 정보를 제공하였다는 것 외에는 정보를 반환하지 않습니다.

## 로그인 엔드포인트 구현

어디로 요청할지 정했고, 응답 코드도 정했으니 간단하게 테스트 먼저 구현합니다.

### 200 응답만 기대하는 테스트 구현하기

이전과 같이 작은 단계별로 테스트를 구현합니다. 어떠한 조건이든 간에 200 응답을 하는 테스트입니다.

```java
@Test
void shouldCreateANewTokenIfUserExist() {
    User user = new User(
            null,
            "limvik@limvik.com",
            "password",
            null);

    userRepository.save(user);

    User loginRequestedUser = new User(null, "limvik@limvik.com", "password", null);

    ResponseEntity<String> createResponse = restTemplate
            .postForEntity("/api/v1/user/login", loginRequestedUser, String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

}
```

역시나 당연하게도 실패합니다. 그리고 Spring Security로 인해서 403을 반환합니다.

```
expected: 200 OK
 but was: 403 FORBIDDEN
```

### 200 응답만 하도록 구현하기

Controller에서 먼저 경로를 추가합니다.

아직 응답에 포함할 정보를 결정하지 않았으니, 임시로 Void를 사용합니다.

```java
@PostMapping("/login")
public ResponseEntity<Void> login(@RequestBody User user) {
    return ResponseEntity.ok().build();
}
```

다음으로 SecurityConfig 의 FilterChain을 수정해줍니다. login 경로를 누구나 접근 가능하도록 허용합니다.

```java
@Bean
public SecurityFilterChain userFilterChain(HttpSecurity http) throws Exception {
    return http.authorizeHttpRequests(auth -> {
        auth.requestMatchers(new AntPathRequestMatcher("/api/v1/user")).permitAll();
        auth.requestMatchers(new AntPathRequestMatcher("/api/v1/user/login")).permitAll();})
            .csrf(AbstractHttpConfigurer::disable)
            .build();
}
```

그리고 다시 테스트를 해보면, PASSED 가 시현되는 것을 볼 수 있습니다.

```
UserRequestTests > shouldCreateANewTokenIfUserExist() PASSED
```

### 존재하는 사용자일 경우에만 200 응답을 기대하는 테스트 구현하기

비밀번호가 틀린 사용자에 대해 401(Unauthorized) 응답을 기대하는 테스트를 추가합니다.

```java
@Test
void shouldNotCreateANewTokenIfUserDoesNotExist() {
    User user = new User(
            null,
            "limvik@limvik.com",
            "password",
            null);

    userRepository.save(user);

    User loginRequestedUser = new User(null, "limvik@limvik.com", "pass", null);

    ResponseEntity<String> createResponse = restTemplate
            .postForEntity("/api/v1/user/login", loginRequestedUser, String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.UNAUTHORIZED);
}
```

당연하게도 아래와 같은 테스트 결과가 나옵니다.

```
expected: 401 UNAUTHORIZED
 but was: 200 OK
```

### 존재하는 사용자일 경우에만 200 응답을 하도록 구현하기

먼저 Controller에 사용자 존재여부를 확인하는 로직을 추가합니다. 존재하는 사용자가 아닐 경우에는 앞서 정했던 대로 401(Unauthorized)를 반환하도록 합니다.

```java
@PostMapping("/login")
public ResponseEntity<Void> login(@RequestBody User user) {
    if (userService.isExist(user))
        return ResponseEntity.ok().build();
    
    return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
}
```

그리고 Service 에서는 사용자 존재여부를 확인하는 로직을 추가합니다.

```java
@Transactional(readOnly = true)
public boolean isExist(User user) {
    return userRepository.exists(Example.of(user));
}
```

그리고 결과는 PASSED

```
UserRequestTests > shouldNotCreateANewTokenIfUserDoesNotExist() PASSED
```

하지만 이렇게 진행이 되면 안되죠. Spring Security를 통해 사용자 로그인 정보를 받아서 인증하고, 인가한 후에 JWT 를 생성해서 반환해야 합니다.

제가 공부한 순서대로 라면 'Filter -> AuthenticationManager -> ProviderManager -> AuthenticationProvider -> 인증 성공/실패한 경우 처리'가 되어야 합니다.

순서대로 보자면 사용자의 요청을 가로챌 Filter를 먼저 구현해야 합니다. 그 전에 RFC 문서를 보면서 JWT 를 구현할게 아닌 이상 JWT 라이브러리를 추가해야겠습니다.

### 어떤 JWT 라이브러리를 사용할까?

[jwt.io](https://jwt.io/libraries?language=Java) 에서 Java 라이브러리만 필터링해서 확인할 수 있습니다.

jwt.io 페이지를 만든 okta 에서 만든 라이브러리 java-jwt([Github 링크](https://github.com/auth0/java-jwt))와 앞서 API 문서를 봤던 FusionAuth 에서 만든 라이브러리 fusionauth-jwt([Github 링크](https://github.com/fusionauth/fusionauth-jwt)), 그리고 개인이 만들었는지 사람 이름이 적혀있는 라이브러리 jjwt([Github 링크](https://github.com/jwtk/jjwt))가 눈에 들어옵니다.

페이지에 나와있는 비교표를 보면, 대동소이 합니다. JWA 문서([링크](https://datatracker.ietf.org/doc/html/rfc7518#section-3.1))에서 JWS 알고리즘 중 required 또는 recommended 라고 표시된 알고리즘은 모두 지원해서 알고리즘 지원 여부로 판단하기도 애매합니다. 그래서 저는 예제 찾기도 쉽고, Github Star도 많은 jjwt 라이브러리를 사용하기로 했습니다.

라이브러리를 build.gradle에 추가해줍니다.

```
implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'  
runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
```

그럼 먼저 순서대로 Filter 부터 구현해 보겠습니다.

### EmailPasswordAuthenticationFilter 파일 생성

OncePerRequestFilter 나 AbstractAuthenticationProcessingFilter 같은 abstract class를 상속 받아서 사용할 계획입니다. AbstractAuthenticationProcessingFilter는 RememberMe 와 같이 불필요한 것들을 미리 구현하고 있어서 탈락시키고, Request 당 한번만 실행될 수 있는 코드만 미리 구현된 OncePerRequestFilter 를 상속받기로 하였습니다.

세부 내용을 모두 이해하지는 못하지만, BasicAuthenticationFilter 등에서 이미 많이 사용돼온 Filter 이므로 믿고 사용하기로 합니다.

다음으로 Spring Security의 UsernamePasswordAuthenticationFilter 의 이름을 참고해서 `EmailPasswordAuthenticationFilter` 를 추가합니다. 

UsernamePasswordAuthenticationFilter를 그대로 사용해볼까 고민했지만, AbstractAuthenticationProcessingFilter를 상속받고 있고, Javadoc 에 Form 제출에 대한 Filter 임을 명시하고 있어 EmailPasswordAuthenticationFilter를 별도로 만들었습니다.

```java
package com.limvik.wantedpreonboardingbackend.securityfilter;

import org.springframework.web.filter.OncePerRequestFilter;

public class EmailPasswordAuthenticationFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

    }
}
```

Filter 에서 먼저 해야할 일은 인증을 받을 Authentication 객체를 만들어 주는 일 입니다. Spring Security는 Authentication 인터페이스의 구현체에 Token 이라는 이름을 붙입니다.

EmailPasswordAuthenticationToken 을 만들면 좋겠지만, Spring Security에서 인증 시에 사용하는 Java API인 Principal([API 문서 링크](https://docs.oracle.com/javase/8/docs/api/java/security/Principal.html))에는 getName() 메서드가 있어서 name이라는 용어를 피할 수 없기 때문에, UsernamePasswordAuthenticationToken 을 그대로 사용하기로 결정했습니다.

결국 name에서 벗어나지는 못합니다. 다 직접 구현하면 바꿀 수 있겠지만, 보통 일은 아니라 일단 미뤄두도록 하겠습니다.

그럼 Filter를 구현하기 전에 Token으로 변환해주는 Converter를 먼저 구현하도록 하겠습니다.

### EmailPasswordAuthenticationConverter 구현

Spring Security 의 `AuthenticationConverter` 인터페이스를 구현합니다.

Security Filter 는 `DispatcherServlet` 보다도 우선순위가 높으므로, 간편하게 `@RequestBody` 와 같은 annotation을 이용할 수가 없습니다. 그래서 직접 body 의 JSON을 parsing 해서 가져옵니다.

```java
package com.limvik.wantedpreonboardingbackend.converter;  
  
import com.fasterxml.jackson.core.JsonProcessingException;  
import com.fasterxml.jackson.databind.JsonNode;  
import com.fasterxml.jackson.databind.ObjectMapper;  
import jakarta.servlet.http.HttpServletRequest;  
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;  
import org.springframework.security.web.authentication.AuthenticationConverter;  
  
import java.io.BufferedReader;  
import java.io.IOException;  
import java.io.InputStream;  
import java.io.InputStreamReader;  
  
public class EmailPasswordAuthenticationConverter implements AuthenticationConverter {  
  
    private static final String EMAIL_PROPERTY = "email";  
  
    private static final String PASSWORD_PROPERTY = "password";  
  
    @Override  
  public UsernamePasswordAuthenticationToken convert(HttpServletRequest request) {  
        var loginUserInfo = getLoginUserInfoFromBody(getBody(request));  
        if (isValidUserInfo(loginUserInfo))  
            return UsernamePasswordAuthenticationToken.unauthenticated(loginUserInfo.email(), loginUserInfo.password());  
        else  
 return null;  
    }  
  
    private String getBody(HttpServletRequest request) {  
  
        StringBuilder stringBuilder = new StringBuilder();  
  
        try {  
            InputStream inputStream = request.getInputStream();  
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));  
            bufferedReader.lines().forEach(stringBuilder::append);  
        } catch (IOException e) {
            throw new RuntimeException(e.getMessage());  
        }  
  
        return stringBuilder.toString();  
    }  
  
    private LoginUserInfo getLoginUserInfoFromBody(String body) {  
        JsonNode jsonNode = null;  
        try {  
            jsonNode = new ObjectMapper().readTree(body);  
        } catch (JsonProcessingException e) {  
            throw new RuntimeException(e);  
        }  
        String email = jsonNode.get(EMAIL_PROPERTY).asText();  
        String password = jsonNode.get(PASSWORD_PROPERTY).asText();  
        return new LoginUserInfo(email, password);  
    }  
  
    private record LoginUserInfo(String email, String password) {}  
  
    private boolean isValidUserInfo(LoginUserInfo userInfo) {  
        return userInfo.email().matches("^[^@]*@[^@]*$") && userInfo.password().length() >= 8;  
    }  
}
```

또한 유효성 검사를 Converter에서 수행하여 통과하지 못할 경우 null을 반환합니다. Filter에서 null 일 경우 처리할 로직을 구현합니다.

그럼 다시 Filter로 돌아가서 null 반환에 대한 로직을 구현하겠습니다.

### EmailPasswordAuthenticationFilter 구현

유효성 검사를 통과하지 못하여 반환되는 token 이 null 일 경우, 응답 코드로 400(Bad Request)를 지정하고, response body 에 JSON 으로 error 메시지를 전달합니다. 그리고 return 을 하여 굳이 다음 filter로 넘어가지 않도록 합니다. 코드 밑에 설명을 이어서 붙이겠습니다.

```java
package com.limvik.wantedpreonboardingbackend.securityfilter;

import com.limvik.wantedpreonboardingbackend.converter.EmailPasswordAuthenticationConverter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.context.SecurityContextHolderStrategy;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

public class EmailPasswordAuthenticationFilter extends OncePerRequestFilter {

    private final SecurityContextHolderStrategy securityContextHolderStrategy = SecurityContextHolder
            .getContextHolderStrategy();

    private final AuthenticationManager authenticationManager;

    public EmailPasswordAuthenticationFilter(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        UsernamePasswordAuthenticationToken token = new EmailPasswordAuthenticationConverter().convert(request);

        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");

        if (token == null) {
            response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            String jsonResponse = "{\"error\": \"이메일에 @가 포함되었는지, 또는 비밀번호가 8자리 이상인지 확인해주세요.\"}";
            response.getWriter().write(jsonResponse);
            return;
        }

        try {
            securityContextHolderStrategy.getContext().setAuthentication(authenticationManager.authenticate(token));
            filterChain.doFilter(request, response);
        } catch (AuthenticationException e) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            String jsonResponse = "{\"error\": \"일치하는 정보를 찾을 수 없습니다.\"}";
            response.getWriter().write(jsonResponse);
            return;
        }
    }

}
```

먼저 인증 실패 시(AuthenticationException 발생 시), 앞서 정했던 401(Unauthroized)로 응답하도록 설정하고, error 메시지를 함께 보내도록 합니다.

인증 성공 시에는 SecurityContextHolder에 저장합니다. 그 이유는 인증된 사용자 정보를 Controller로 보내기 위함입니다. Filter 에서 처리하는게 맞지 않을까 고민이 되기도 했는데, 회원가입 시에 반환할 정보를 Controller 에서 추가하고 있어서 통일하는게 좋다고 생각했습니다.

Controller 를 간단하게 미리 수정하고 가겠습니다.

### UserController 수정

`@AuthenticationPrincipal` 로 인증된 User 객체를 받습니다. @RequestBody 를 사용하면 오류가 발생합니다. EmailPasswordAuthenticationConverter 에서 request 의 InputStream을 이미 사용했기 때문입니다.

```java
@PostMapping("/login")
public ResponseEntity<Void> login(@AuthenticationPrincipal User user) {
    // JWT 만들어서 응답에 추가 예정
    SecurityContextHolder.clearContext();
    return ResponseEntity.ok().build();
}
```

이후 JWT 를 만들 때 로직을 추가할 예정이고, SecurityContextHolder에 있는 인증된 사용자의 정보를 제거하는 코드를 추가했습니다.

이제는 접근 시에 Filter를 거칠 수 있도록 SecurityConfig 를 다시 설정해줘야겠습니다.

### SecurityConfig 수정

하는 김에 전체적으로 수정할 겸 회원가입 시 적용되는 Filter의 목록을 로그로 확인해봅니다.

```
DEBUG o.s.security.web.FilterChainProxy - Securing POST /api/v1/user
TRACE o.s.security.web.FilterChainProxy - Invoking DisableEncodeUrlFilter (1/10)
TRACE o.s.security.web.FilterChainProxy - Invoking WebAsyncManagerIntegrationFilter (2/10)
TRACE o.s.security.web.FilterChainProxy - Invoking SecurityContextHolderFilter (3/10)
TRACE o.s.security.web.FilterChainProxy - Invoking HeaderWriterFilter (4/10)
TRACE o.s.security.web.FilterChainProxy - Invoking LogoutFilter (5/10)
TRACE o.s.s.w.a.logout.LogoutFilter - Did not match request to Or [Ant [pattern='/logout', GET], Ant [pattern='/logout', POST], Ant [pattern='/logout', PUT], Ant [pattern='/logout', DELETE]]
TRACE o.s.security.web.FilterChainProxy - Invoking RequestCacheAwareFilter (6/10)
TRACE o.s.s.w.s.HttpSessionRequestCache - matchingRequestParameterName is required for getMatchingRequest to lookup a value, but not provided
TRACE o.s.security.web.FilterChainProxy - Invoking SecurityContextHolderAwareRequestFilter (7/10)
TRACE o.s.security.web.FilterChainProxy - Invoking AnonymousAuthenticationFilter (8/10)
TRACE o.s.security.web.FilterChainProxy - Invoking ExceptionTranslationFilter (9/10)
TRACE o.s.security.web.FilterChainProxy - Invoking AuthorizationFilter (10/10)
```

회원가입 할 때 SecurityContextHolder에 저장할 정보는 없고, Logout 할 일도 없고, 익명 사용자 인증도 할 필요가 없습니다. 따라서 관련 Filter를 제거하고, REST API 이므로 Session 은 필요 없기 때문에 Session 쿠키를 만들지 않도록 Session 관련 Filter를 추가합니다.

그리고 login 시에 사용하는 Filter 는 또 다르므로, 분리하기 위해 securityMatcher 를 이용하여 경로를 지정합니다.

```java
@Bean
public SecurityFilterChain signUpFilterChain(HttpSecurity http) throws Exception {
    return http
            .securityMatcher("/api/v1/user")
            .authorizeHttpRequests(auth -> {
                auth.requestMatchers(new AntPathRequestMatcher("/api/v1/user", "POST")).permitAll();})
            .csrf(AbstractHttpConfigurer::disable)
            .logout(AbstractHttpConfigurer::disable)
            .securityContext(AbstractHttpConfigurer::disable)
            .anonymous(AbstractHttpConfigurer::disable)
            .sessionManagement(config -> config.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .build();
}
```

그러면 다음과 같이 3개 삭제하고, 1개 추가해서 총 10개에서 8개가 됩니다.

```
TRACE o.s.security.web.FilterChainProxy - Invoking DisableEncodeUrlFilter (1/8)
TRACE o.s.security.web.FilterChainProxy - Invoking WebAsyncManagerIntegrationFilter (2/8)
TRACE o.s.security.web.FilterChainProxy - Invoking HeaderWriterFilter (3/8)
TRACE o.s.security.web.FilterChainProxy - Invoking RequestCacheAwareFilter (4/8)
TRACE o.s.security.web.FilterChainProxy - Invoking SecurityContextHolderAwareRequestFilter (5/8)
TRACE o.s.security.web.FilterChainProxy - Invoking SessionManagementFilter (6/8)
TRACE o.s.security.web.FilterChainProxy - Invoking ExceptionTranslationFilter (7/8)
TRACE o.s.security.web.FilterChainProxy - Invoking AuthorizationFilter (8/8)
```

다음은 로그인용 SecurityFilterChain을 추가해야 합니다.

앞서 순서를 'Filter -> AuthenticationManager -> ProviderManager -> AuthenticationProvider -> 인증 성공/실패한 경우 처리' 순으로 한다고 말씀드렸습니다.

기본적으로 Spring Security에 구현되어 있는`ProviderManager`는 AuthenticationManager의 구현체이므로 그대로 사용하고(순서에서는 두 개를 구분해 놓았지만 하나로 봐야합니다.), AuthenticationProvider의 구현체로는 UsernamePasswordAuthenticationToken을 처리할 수 있는 `DaoAuthenticationProvider` 를 사용합니다.

```java
@Bean
public SecurityFilterChain loginFilterChain(HttpSecurity http) throws Exception {
    return http
            .securityMatcher("/api/v1/user/login")
            .authorizeHttpRequests(auth -> {
                auth.requestMatchers(new AntPathRequestMatcher("/api/v1/user/login", "POST")).permitAll();})
            .csrf(AbstractHttpConfigurer::disable)
            .logout(AbstractHttpConfigurer::disable)
            .anonymous(AbstractHttpConfigurer::disable)
            .sessionManagement(config -> config.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .addFilterAfter(new EmailPasswordAuthenticationFilter(getProviderManager()),
                    RequestCacheAwareFilter.class)
            .build();
}

private ProviderManager getProviderManager() {
    return new ProviderManager(getDaoAuthenticationProvider());
}

private DaoAuthenticationProvider getDaoAuthenticationProvider() {
    var daoAuthenticationProvider = new DaoAuthenticationProvider();
    daoAuthenticationProvider.setUserDetailsService(new UserService(userRepository, passwordEncoder()));
    daoAuthenticationProvider.setPasswordEncoder(passwordEncoder());
    return daoAuthenticationProvider;
}
```

authorizeHttpRequest 만 설정해주면 경로에 따라 SecurityFilterChain이 적용되는 줄 잘못 알고 있었는데, `securityMatcher` 메서드로 설정해야 된다는 걸 뒤늦게 알았습니다([유튜브 영상](https://www.youtube.com/watch?v=Yn2oCmHI7hc)). 

로그인 시에는 SecurityContextHolder에 인증 정보를 저장해야 하므로 Filter를 살려둡니다.

문제는 DaoAuthenticationProvider를 사용하기 위해 UserService의 인스턴스를 SecurityConfig 에서 생성한 것입니다. UserService에서 PasswordEncoder를 주입받고 있다보니 순환 참조라고 오류가 발생합니다. 설정으로 되게할 수는 있는데, 괜히 하지말라는거 하는게 좀 찝찝해서 인스턴스를 생성하는 방식으로 구현했습니다.

다음으로 DaoAuthenticationProvider를 사용하려면 UserDetails 와 UserDetailsService를 구현해주어야 합니다.

### UserDetails, UserDetailsService

먼저 UserDetails 를 User 객체에서 구현합니다. 현재 별도의 인증 절차는 없으므로, 로그인 할 수만 있으면 getAuthorities 메서드에서 `USER`의 권한을 반환 합니다. 그리고 가입하자마자 활성화돼야 하므로 나머지 메서드는 모두 true를 반환합니다. getUsername()은 어쩔 수 없지만 email을 반환하도록 합니다.

```java
@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity
public class User implements UserDetails {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    @Pattern(regexp = "^[^@]*@[^@]*$", message = "{user.email.pattern}")
    private String email;

    @Column(nullable = false)
    @Length(min = 8, message = "{user.password.length}")
    private String password;

    @CreationTimestamp
    private LocalDateTime createdAt;

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority("ROLE_USER"));
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public String getUsername() {
        return email;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

`@JsonProperty(access = JsonProperty.Access.WRITE_ONLY)` 는 JSON 변환 시 결과로 출력하지 않게 하는 annotation 입니다.

다음으로 UserDetailsService 를 UserService에서 구현합니다. loadUserByUsername 메서드만 Override하면 됩니다.

```java
@RequiredArgsConstructor
@Service
public class UserService implements UserDetailsService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    @Transactional
    public User createUser(User newUser) {
        newUser.setPassword(passwordEncoder.encode(newUser.getPassword()));
        return userRepository.save(newUser);
    }

    @Transactional(readOnly = true)
    public boolean isExist(User user) {
        return userRepository.exists(Example.of(user));
    }

    @Transactional(readOnly = true)
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userRepository.findByEmail(username);
    }
}
```

findByEmail은 기본적으로 지원하는 것은 아니므로, UserRepository에 추가합니다.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    User findByEmail(String email);
}
```

### 테스트 수정

테스트 코드 일부는 수정이 필요하기도 하고, 한 번 테스트를 수행하고 진행하는게 좋을 것 같습니다.

처음에 저장되는 사용자의 계정은 암호화한 상태로 저장하게 수정하였습니다.

```java
@Test
void shouldCreateANewTokenIfUserExist() {
    User user = new User(
            null,
            "limvik@limvik.com",
            passwordEncoder.encode("password"),
            null);

    userRepository.save(user);

    User loginRequestedUser = new User(null, "limvik@limvik.com", "password", null);

    ResponseEntity<String> createResponse = restTemplate
            .postForEntity("/api/v1/user/login", loginRequestedUser, String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

}

@Test
void shouldNotCreateANewTokenIfUserDoesNotExist() {
    User user = new User(
            null,
            "limvik@limvik.com",
            passwordEncoder.encode("password"),
            null);

    userRepository.save(user);

    User loginRequestedUser = new User(null, "lim@vik.com", "password", null);

    ResponseEntity<String> createResponse = restTemplate
            .postForEntity("/api/v1/user/login", loginRequestedUser, String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.UNAUTHORIZED);
}
```

그리고 없는 사용자를 테스트하는데, 이전에 password만 pass 로 하고, email은 동일하게 해서 유효성 검사를 통과 못하게되어 있었습니다. 비밀번호는 password 로 설정하여 일치하게 만들고, 다른 email(lim@vik.com)로 테스트가 수행되도록 수정하였습니다.

그리고 테스트를 수행해보면, 일치하는 사용자 정보가 없는 경우에 대한 테스트(shouldNotCreateANewTokenIfUserDoesNotExist)만 다음과 같은 에러가 발생합니다.

```
org.springframework.web.client.ResourceAccessException: I/O error on POST request for "http://localhost:13899/api/v1/user/login": cannot retry due to server authentication, in streaming mode
Caused by: java.net.HttpRetryException: cannot retry due to server authentication, in streaming mode
```

이와 유사한 예외가 발생하는 것을 정리해 놓은신 분의 글([링크](https://github.com/HomoEfficio/dev-tips/blob/master/Spring%20Security%20%EB%A1%9C%EA%B7%B8%EC%9D%B8%20%EC%8B%A4%ED%8C%A8%20%EC%B2%98%EB%A6%AC%20%EC%8B%9C%20java.net.HttpRetryException:%20cannot%20retry%20due%20to%20server%20authentication,%20in%20streaming%20mode.md))을 보니, 이유가 비슷한 것 같습니다. 재밌는건 Postman 을 이용해서 존재하지 않는 사용자로 로그인 요청해보면 이상없이 잘 동작합니다.

여러가지 방법을 시도해 봤지만, TestRestTemplate 을 이용한 방법으로는 아직 제 실력으로 문제가 해결되지 않아 @Disabled 처리하고, MockMvc 테스트로 전환했습니다.

```java
@Test
@Disabled("TestRestTemplate 이용 시 오류가 해결되지 않아 MockMvc 테스트를 별도로 작성합니다.")
void shouldNotCreateANewTokenIfUserDoesNotExist() {
```

다음은 존재하지 않는 사용자에 대한 MockMvc 테스트 입니다. 비밀번호가 틀린 경우도 추가해주었습니다.

```java
package com.limvik.wantedpreonboardingbackend;

import com.limvik.wantedpreonboardingbackend.domain.User;
import com.limvik.wantedpreonboardingbackend.repository.UserRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
public class UserLoginTests {
    @Autowired
    MockMvc mockMvc;

    @Autowired
    UserRepository userRepository;

    @Test
    void shouldNotCreateANewTokenIfUserDoesNotExist() throws Exception {
        mockMvc.perform(post("/api/v1/user/login")
            .contentType(MediaType.APPLICATION_JSON)
            .content("{\"email\":\"lim@vik.com\", \"password\":\"password\"}"))
            .andExpect(status().isUnauthorized());
    }

    @Test
    void shouldNotCreateANewTokenIfWrongPassword() throws Exception {

        userRepository.save(new User(null, "limvik@limvik.com", "password", null));

        mockMvc.perform(post("/api/v1/user/login")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"email\":\"limvim@limvik.com\", \"password\":\"wrongPassword\"}"))
                .andExpect(status().isUnauthorized());
    }
}

```

테스트 결과는 아래와 같이 기대하는 401(Unauthorized)와 함께 PASSED 가 시현됩니다.

```
UserLoginTests > shouldNotCreateANewTokenIfWrongPassword() PASSED
UserLoginTests > shouldNotCreateANewTokenIfUserDoesNotExist() PASSED
```

### 클라이언트에게 어떤 정보를 반환해야 할까

먼저 JWT 를 만들기 전에 클라이언트에게 어떤 정보를 반환해야 할지 고민이됩니다. 이럴땐 다른 사람이 만든걸 찾아봐야겠죠.

간단하게 카카오 로그인 API 의 토큰 받기 응답([링크](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api#request-token-response))을 살펴봅니다.

| 이름 | 타입 | 필수 |
|--|--|--|
| token_type | String | O |
| access_token | String | O |
| id_token | String | X |
| expires_in | Integer | O |
| refresh_token | String | O |
| refresh_token_expires_in | Integer | O |
| scope | String | X |

저는 OAuth 2.0은 아니므로 access token 과 refresh token 을 모두 제외합니다. token_type 설명을 보면 bearer 고정이라 적혀있는데, bearer RFC 문서를 보니, OAuth 2.0 을 위한 것으로 판단돼서 jwt 를 고정하기로 합니다. 마지막 scope 는 Spring Security 를 이용해서 역할 기반 접근 통제를 하고 있으므로, role 로 교체하겠습니다.

정리해 보자면 body에 JSON으로 포함해서 클라이언트에 보낼 정보는 아래와 같습니다.

| 이름 | 타입 | 설명 | 필수 |
|--|--|--|--|
| tokenType | String | `jwt` 고정| O |
| token | String | 토큰 | O |
| expiresIn | Integer | 토큰 만료 시간(밀리초) | O |
| role | String | 역할 | O |

현재 요구사항으로는 role이 필요 없겠지만, 추후에 admin 등 확장하는 걸 고려해서 포함했습니다. expiresIn 은 필요 없는 것 같기는 한데, 빼야할 이유도 찾지 못해서 일단 가지고 갑니다.

그리고 jwt 에 claim 으로 포함할 정보는 아래와 같습니다.

| 이름 | 타입 | 설명 | 필수 |
|--|--|--|--|
| iss | String | `limvik` 고정| O |
| sub | Integer | 데이터베이스 상의 사용자 id | O |
| iat | Integer | 토큰 발급 시각 | O |
| exp | Integer | 토큰 만료 시각 | O |
| email | String | 사용자 email | O |

### 로그인에 성공한 경우 응답 테스트 수정

Response status code 가 200(OK) 인 것을 확인하는 테스트 뒤에 각 JSON 에 있는 property를 확인하는 테스트 코드를 작성했습니다. 유효기간(expiresIn)은 기획에 따라 많이 바뀔 수 있다고 생각해서, 그냥 적당히 마음에 드는 하루로 했습니다.

```java
@Test
void shouldCreateANewTokenIfUserExist() {
    User user = new User(
            null,
            "limvik@limvik.com",
            passwordEncoder.encode("password"),
            null);

    userRepository.save(user);

    User loginRequestedUser = new User(null, "limvik@limvik.com", "password", null);

    ResponseEntity<String> createResponse = restTemplate
            .postForEntity("/api/v1/user/login", loginRequestedUser, String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

    DocumentContext documentContext = JsonPath.parse(createResponse.getBody());
    String tokenType =  documentContext.read("$.tokenType");
    assertThat(tokenType).isEqualTo("jwt");
    String token = documentContext.read("$.token");
    assertThat(token).isNotBlank();
    Number tokenExpirationTime = documentContext.read("$.expiresIn");
    assertThat(tokenExpirationTime).isEqualTo((int)Duration.ofDays(1).toSeconds());
    String role = documentContext.read("$.role");
    assertThat(role).contains("ROLE_USER");
}
```

테스트 결과는 당연하게도 expected 조차 못보고 예외가 발생합니다.

```
java.lang.IllegalArgumentException: json string can not be null or empty
```

### Controller 응답 수정

record 로 응답을 보낼 LoginResponse 를 만들어 줍니다. 그리고 아직 JWT 는 만들지 않았으니 dummy 로 "token" 문자열을 만든 후 응답을 보냅니다.

```java
@PostMapping("/login")
public ResponseEntity<LoginResponse> login(@AuthenticationPrincipal User user) {
    var loginResponse = new LoginResponse("jwt",
            "token",
            Duration.ofDays(1).toSeconds(),
            user.getAuthorities().toString());
    SecurityContextHolder.clearContext();
    return ResponseEntity.ok(loginResponse);
}

public record LoginResponse(String tokenType, String token, long expiresIn, String role) {}
```

그리고 다시 테스트를 해보면, PASSED!

```
UserRequestTests > shouldCreateANewTokenIfUserExist() PASSED
```

다음 과제를 고려한다면, 회원이 접근할 수 있는 게시판 링크를 같이 전달해주면 좋겠습니다. 게시판 과제 수행하면서 같이 수정하기로 하고, 드디어 JWT 를 추가해봅니다.

### 로그인에 성공한 경우 응답 테스트 JWT 반영

아직 JWT 에 관한게 하나도 없지만, 의사 코드라도 적당히 써서 JWT를 확인하는 테스트 코드를 작성하겠습니다.

뭐 넣기로 했는지 올라가기 귀찮으니까, 여기에 다시 붙여넣습니다.

| 이름 | 타입 | 설명 | 필수 |
|--|--|--|--|
| iss | String | `limvik` 고정| O |
| sub | Integer | 데이터베이스 상의 사용자 id | O |
| iat | Integer | 토큰 발급 시각 | O |
| exp | Integer | 토큰 만료 시각 | O |
| email | String | 사용자 email | O |

jwtProvider 라는 이름으로 jwt를 만들어주는 클래스를 만들기로 계획합니다. Claims 는 JWT 라이브러리에 있는 클래스라 IDE의 도움을 받아 쉽게 테스트를 작성할 수 있었습니다.

위에서 작성한 테스트에서 String token 을 불러왔으므로, 그대로 사용해서 유효성 검사를 한 후에 Claims 를 불러와 각각을 테스트 합니다. 토큰 발급 시각과 만료 시각은 정확하게 체크할 수 없으니, 발급 시각이 만료 시각보다 앞인지 확인합니다.

```java
assertThat(jwtProvider.valid(token)).isTrue();
Claims claims = jwtProvider.getClaims(token);
assertThat(claims.getIssuer()).isEqualTo("limvik");
assertThat(Long.parseLong(claims.getSubject())).isEqualTo(1L);
assertThat(claims.getIssuedAt()).isBefore(claims.getExpiration());
assertThat(claims.getIssuedAt()).isBefore(new Date());
assertThat(claims.getExpiration()).isAfter(new Date());
assertThat(claims.get("email")).isEqualTo("limvik@limvik.com");
```

jwtProvider 가 없어서 오류나므로, 굳이 테스트 결과를 추가하지는 않겠습니다.

### JwtConfig

JwtProvider를 추가하기 전에, JWT와 관련된 값을 불러오기 위해서 JwtConfig 클래스를 추가합니다.

```java
package com.limvik.wantedpreonboardingbackend.config;

import lombok.Getter;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.time.Duration;

@Getter
@Component
@ConfigurationProperties("jwt")
public class JwtConfig {

    public static final long JWT_EXPIRATION_MS = Duration.ofDays(1).toMillis();

    @Value("${jwt.issuer}")
    private String jwtIssuer;

    @Value("${jwt.key}")
    private String jwtKey;

}
```

issuer는 어차피 고정된 값이므로, 같이 추가합니다. 앞서 Duration을 정할 때 toSeconds로 했었는데 new Date() 시에 milliseconds 로 생성된다는걸 깜빡했었습니다. 뒤에서 수정한 테스트코드는 다시 보도록 하겠습니다.

### JwtProvider

valid 메서드를 만들려다가, 다른 분들이 구현한걸 보니 valid 할 때 parsing 하고, Claims 를 불러올 때 다시 parsing을 하길래 비효율적인 것 같아 Claims를 먼저 불러오고, 토큰 만료 시간이 지났는지 여부를 체크하는 메서드로 valid 를 만들었습니다.

그리고 옛날 버전의 예제를 보면, String을 secret key로 사용하는데, 이제는 Keys 클래스를 이용해서 암호화된 key를 만들어주어야 합니다. secret key 만들려고 찾아보기까지 했는데, 큰 의미는 없어졌습니다. [https://jwt-keys.21no.de/](https://jwt-keys.21no.de/)

```java
package com.limvik.wantedpreonboardingbackend.jwt;

import com.limvik.wantedpreonboardingbackend.config.JwtConfig;
import com.limvik.wantedpreonboardingbackend.domain.User;
import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

import java.nio.charset.StandardCharsets;
import java.util.Date;
import java.util.Optional;

@RequiredArgsConstructor
@Component
public class JwtProvider {

    private final JwtConfig jwtConfig;

    public String generateToken(User user) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + JwtConfig.JWT_EXPIRATION_MS);
        return Jwts.builder()
                .setHeaderParam(Header.TYPE, Header.JWT_TYPE)
                .setIssuer(jwtConfig.getJwtIssuer())
                .setIssuedAt(now)
                .setExpiration(expiryDate)
                .setSubject(user.getId().toString())
                .claim("email", user.getEmail())
                .signWith(Keys.hmacShaKeyFor(jwtConfig.getJwtKey().getBytes(StandardCharsets.UTF_8)))
                .compact();
    }

    public Optional<Claims> getClaims(String token) {
        try {
            return Optional.ofNullable(Jwts.parserBuilder()
                    .requireIssuer(jwtConfig.getJwtIssuer())
                    .setSigningKey(Keys.hmacShaKeyFor(jwtConfig.getJwtKey().getBytes(StandardCharsets.UTF_8)))
                    .build()
                    .parseClaimsJws(token).getBody());
        } catch (JwtException e) {
            return Optional.empty();
        }
    }

    public boolean valid(Claims claims) {
        return claims.getExpiration().before(new Date());
    }

}
```

### 테스트 수정하기

valid 메서드를 Claims 를 불러온 이후로 변경하였습니다. 그리고 valid 메서드가 만기 시간이 지났는지 확인하는 메서드이므로, 만기 시간을 확인하는 테스트는 제거했습니다.

사용자 id가 auto increment로 설정되다보니, subject 확인 시 예상하는 id와 다른 경우가 있습니다. 그래서 @DirtiesContext 를 이용하여 테스트 메서드 실행 전에 Context를 초기화 하도록 수정했습니다. ([참고한 글 링크](https://github.com/HomoEfficio/dev-tips/blob/master/Spring%20Data%20JPA%20%ED%85%8C%EC%8A%A4%ED%8A%B8%20%EC%8B%9C%20auto-increment%20%EB%AC%B8%EC%A0%9C.md))

```java
@Test
@DirtiesContext(methodMode = DirtiesContext.MethodMode.BEFORE_METHOD)
void shouldCreateANewTokenIfUserExist() {
    // 나머지 생략

    // JWT claims 확인
    assertThat(Jwts.parserBuilder().build().isSigned(token)).isTrue();
    Claims claims = jwtProvider.getClaims(token).orElseThrow();
    assertThat(jwtProvider.valid(claims)).isTrue();
    assertThat(claims.getIssuer()).isEqualTo(jwtConfig.getJwtIssuer());
    assertThat(Long.parseLong(claims.getSubject())).isEqualTo(1L);
    assertThat(claims.getIssuedAt()).isBefore(claims.getExpiration());
    assertThat(claims.getIssuedAt()).isBefore(new Date());
    assertThat(claims.get("email")).isEqualTo("limvik@limvik.com");
}
```

### Postman

Postman 으로 회원가입/로그인 기능을 테스트해보면, 아래와 같은 JSON 응답을 받을 수 있습니다.

```json
{
    "tokenType": "jwt",
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpc3MiOiJsaW12aWsiLCJpYXQiOjE2OTE0MDkyODEsImV4cCI6MTY5MTQ5NTY4MSwic3ViIjoiMSIsImVtYWlsIjoibGltdmlrQGxpbXZpay5jb20ifQ.jzwDXJ40y0IJb_atMzlyit1OetFviEAQxb4edl_Z0v2yULBVqAehTBwWJhzhzucgBgkXGHFlNk7HYCTB04NjBA",
    "expiresIn": 86400000,
    "role": "[ROLE_USER]"
}
```

### 리팩터링. 데이터베이스 분리

테스트시에는 h2 데이터베이스를 사용하도록 분리하였습니다.

이 과정에서 오류가 발생했습니다. /test/resource 디렉토리에 application.properties 파일을 추가하였는데, `Error executing DDL` 오류가 발생하면서 테이블을 생성하지 못했습니다.

다행히 h2 데이터베이스 예약어라서 사용할 수 없는 `user` 를 사용했기 때문이라는 것을 검색하자마자 다른 분의 글([링크](https://jeongkyun-it.tistory.com/177))에서 찾을 수 있었습니다.

그래서 @Table annotation을 사용하여 User 클래스에 Table 생성 시 이름이 users 가 되도록 수정하였습니다.

```java
@Table(name = "users")
public class User implements UserDetails {
```

### 리팩터링. 하려다 만거

테스트 코드에 @Transactional 을 붙이시는 분을 보고, '아 빼먹었다' 하고 생각했습니다. 그런데, @Transactional 처리 하는 이유가 데이터 무결성을 보장하기 위해서로 알고있는데, 테스트는 어차피 테스트 시에만 사용하고 사라질 데이터라 테스트 속도만 느려지게 할 것 같아서 추가하지 않았습니다.

## Outro

일요일까지 제출은 꿈 같은 일이었습니다. 다음 가산점은 11일 까지인데, 과연...!!

시간이 그래도 좀 들었는데, 토큰을 도난당하는 순간 보안이 뻥 뚫려버리는 문제를 어떻게 해결해야 할지 잘 모르겠다는게 아쉽습니다. 과제 완성을 우선하고, 차근차근 알아가야겠죠.
