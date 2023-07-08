---
layout: post
title: Spring Security (4) SecurityFilterChain 과 Filters
categories:
- Framework
- Spring
- Spring Security
Tags:
- Spring
- Spring Security
date: 2023-07-08 21:28 +0900
---
## Intro

[이전 글](https://limvik.github.io/posts/study-spring-security-3-delegatingfilterproxy-and-filterchainproxy/)에서 DelegatingFilterProxy와 FilterChainProxy를 살펴봤습니다.

마지막에는 다음과 같이 정리했었습니다.

>Servlet Container 메커니즘 상 Spring Bean을 인식할 수 없기 때문에, Spring 에서는 DelegatingFilterProxy를 Servlet Container에 Filter로 등록한 후 DelegatingFilterProxy를 통해 Bean 으로 등록된 Filter 를 불러와 호출합니다. 이 과정에서 Spring Security 전용 Filter 인 FilterChainProxy 가 호출되며, FilterChainProxy는 다시 SecurityFilterChain을 호출하여 로직을 수행합니다.

이번에는 마지막에서 언급된 SecurityFilterChain에 대해 조사해보겠습니다.

## SecurityFilterChain

이전 글에서 봤던 아래 다이어그램만 보면 SecurityFilterChain이라는 단일 클래스가 존재할 것 같지만 그렇지 않습니다.

![FilterChainProxy가 포함된 다이어그램](/assets/img/2023-07-07-study-spring-security-3-delegatingfilterproxy-filterchainproxy/04-filterchainproxy-diagram.png)

Spring Security 공식 문서의 Servlet Application 에서 Architecture 문서([링크](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-securityfilterchain))를 보면 SecurityFilterChain 내에 여러 Filter가 존재함을 위 다이어그램 바로 다음에 표시하고 있습니다.

![SecurityFilterChain 의 이전보다 세부 다이어그램](/assets/img/2023-07-08-study-spring-security-4-SecurityFilterChain-and-Filters/01-spread-securityfilterchain.png)

SecurityFilterChain 은 interface 입니다. 별도로 Custom 하지 않는다면, 아래 사진에 All Known Implementing Classes: 에 명시된 DefaultSecurityFilterChain이 적용됩니다.

![SecurityFilterChain 문서](/assets/img/2023-07-08-study-spring-security-4-SecurityFilterChain-and-Filters/02-SecurityFilterChain-doc.png)

[링크](https://docs.spring.io/spring-security/site/docs/6.1.1/api/org/springframework/security/web/SecurityFilterChain.html)

그런데 또 위 다이어그램만 보면 SecurityFilterChain은 1개만 사용 가능한 것 같습니다.

실제로는 아래 그림과 같이 요청 URL 별로 `첫 번째로 매칭`되는 URL 에 해당하는 SecurityFilterChain을 만들어서 적용할 수 있습니다. 그래서 URL에 따라 적용되는 Filter가 다를 수 있습니다. 예를 들어, 웹 사이트의 홈페이지처럼 누구든지 접근 가능해야 한 경우와 API를 요청하는 경우는 다른 Filter가 적용되겠죠.

![SecurityFilterChain 의 이전보다 더 세부적인 다이어그램](/assets/img/2023-07-08-study-spring-security-4-SecurityFilterChain-and-Filters/03-more-spread-securityfilterchain.png)

## SecurityFilterChain의 종류와 순서

Spring Security 공식 문서의 다음 순서인 Filter의 종류를 살펴보겠습니다.

### Filter의 종류

-   [`ForceEagerSessionCreationFilter`](https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html#session-mgmt-force-session-creation)
    
-   `ChannelProcessingFilter`
    
-   `WebAsyncManagerIntegrationFilter`
    
-   `SecurityContextPersistenceFilter`
    
-   `HeaderWriterFilter`
    
-   `CorsFilter`
    
-   `CsrfFilter`
    
-   `LogoutFilter`
    
-   `OAuth2AuthorizationRequestRedirectFilter`
    
-   `Saml2WebSsoAuthenticationRequestFilter`
    
-   `X509AuthenticationFilter`
    
-   `AbstractPreAuthenticatedProcessingFilter`
    
-   `CasAuthenticationFilter`
    
-   `OAuth2LoginAuthenticationFilter`
    
-   `Saml2WebSsoAuthenticationFilter`
    
-   [`UsernamePasswordAuthenticationFilter`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/form.html#servlet-authentication-usernamepasswordauthenticationfilter)
    
-   `DefaultLoginPageGeneratingFilter`
    
-   `DefaultLogoutPageGeneratingFilter`
    
-   `ConcurrentSessionFilter`
    
-   [`DigestAuthenticationFilter`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/digest.html#servlet-authentication-digest)
    
-   `BearerTokenAuthenticationFilter`
    
-   [`BasicAuthenticationFilter`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/basic.html#servlet-authentication-basic)
    
-   [`RequestCacheAwareFilter`](https://docs.spring.io/spring-security/reference/servlet/architecture.html#requestcacheawarefilter)
    
-   `SecurityContextHolderAwareRequestFilter`
    
-   `JaasApiIntegrationFilter`
    
-   `RememberMeAuthenticationFilter`
    
-   `AnonymousAuthenticationFilter`
    
-   `OAuth2AuthorizationCodeGrantFilter`
    
-   `SessionManagementFilter`
    
-   [`ExceptionTranslationFilter`](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-exceptiontranslationfilter)
    
-   [`AuthorizationFilter`](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html)
    
-   `SwitchUserFilter`

굉장히 많은 Filter가 있는 것을 알 수 있습니다. 실제 Filter 를 등록하는 코드([Github 링크](https://github.com/spring-projects/spring-security/blob/01c8a2233655a7e8b5b3aa1fec4032d0d36ace15/config/src/main/java/org/springframework/security/config/annotation/web/builders/FilterOrderRegistration.java))를 보면 몇 가지 더 있기는 한데 Deprecated 된 Filter 입니다.

### Filter의 순서

문서에도 언급되어 있지만, 코드를 보시면 위 목록이 Filter가 적용되는 순서임을 알 수 있습니다.

```java
// FilterOrderRegistration.java
// ...
Step order = new Step(INITIAL_ORDER, ORDER_STEP);
put(DisableEncodeUrlFilter.class, order.next());
put(ForceEagerSessionCreationFilter.class, order.next());
put(ChannelProcessingFilter.class, order.next());
// ...
```

문서에서는 Filter 의 순서를 알 필요는 없지만, 도움이 되는 경우가 있다면서 그 경우가 무엇인지는 알려주지 않습니다. 저도 아직 걸음마 단계라 도움이 되는 경우가 무엇인지 모르기 때문에,  GPT-3.5의 도움을 받아 아래 붙여넣도록 하겠습니다.

---

Spring Security에서 Filter의 순서를 알면 도움이 되는 몇 가지 경우가 있습니다:

1. 인증과 권한 부여:
   - Filter 체인의 순서에 따라 인증(Authentication)과 권한 부여(Authorization)가 처리됩니다. 일반적으로 인증 필터가 먼저 실행되어 사용자를 인증하고, 그 다음 권한 부여 필터가 실행되어 인증된 사용자에게 적절한 권한을 부여합니다.

2. 보안 취약점 대응:
   - 일부 필터는 특정 보안 취약점을 방지하기 위해 반드시 특정 위치에 있어야 합니다. 예를 들어, CSRF(Cross-Site Request Forgery) 공격을 방지하기 위한 CsrfFilter는 필터 체인의 앞부분에 위치해야 합니다.

3. 사용자 정의 필터:
   - 사용자 정의 필터를 구현하여 특정 동작을 수행하거나 로직을 추가할 수 있습니다. 이러한 사용자 정의 필터를 적절한 순서에 배치하여 원하는 동작을 수행할 수 있습니다.

4. 성능 최적화:
   - 필터의 순서는 성능 최적화에도 영향을 미칠 수 있습니다. 더 빈번하게 발생하는 요청을 먼저 처리하는 필터를 앞쪽에 위치시키면 성능을 향상시킬 수 있습니다. 예를 들어, 정적 파일 요청을 처리하는 필터는 정적 파일 요청이 자주 발생하므로 체인의 앞부분에 위치시킬 수 있습니다.

5. 다양한 인증 및 인가 방식:
   - Filter의 순서를 조정하여 다양한 인증 및 인가 방식을 구현할 수 있습니다. 예를 들어, OAuth2 인증 방식을 사용하는 경우 OAuth2 인증 필터를 앞부분에 배치하여 인증을 처리하고, 다른 필터는 인가 및 추가 로직을 처리할 수 있습니다.

필터 체인에서의 순서는 Spring Security 구성 파일(Java 구성 또는 XML 구성)에서 조정할 수 있습니다. `HttpSecurity` 객체의 `addFilterBefore()` 또는 `addFilterAfter()` 메서드를 사용하여 필터를 특정 위치에 추가하거나 순서를 조정할 수 있습니다.

---

순서가 있어야 하는 것은 당연한 이야기로 느껴지기도 하지만, 혹시나 당연하게 느껴지지 않을 분들을 위해 Spring Security가 제공한다는 기능부터 다시 살펴보는게 좋겠습니다.

Spring Security 문서의 Overview([링크](https://docs.spring.io/spring-security/reference/index.html))에 써있던 것 처럼 Spring Security는 인증, 인가 및 일반적인 공격에 대한 보호를 제공하기 위해 Servlet Application에서는 Filter 를 사용합니다.

>Spring Security is a framework that provides [authentication](https://docs.spring.io/spring-security/reference/features/authentication/index.html), [authorization](https://docs.spring.io/spring-security/reference/features/authorization/index.html), and [protection against common attacks](https://docs.spring.io/spring-security/reference/features/exploits/index.html).

일단 일반적인 공격에 대한 보호는 제쳐두고, `인증(authentication)`이나 `인가(authorizatoin)`라는 단어를 생각해봐야 합니다. 이 단어들은 실생활에서도 자연스럽게 접할 수 있는 단어이기는 합니다.

실생활에서의 예를 들어보자면, 오피스 빌딩에 방문 계획이 있어 방문자로 등록되어 있는 경우에 데스크에서 방문자 임을 `인증(authentication)`하고, 방문 가능한 `권한이 부여(authorization)`되어 있는 출입카드를 받아 권한이 있는 곳만 출입카드를 이용하여 방문이 가능합니다.

만약 순서가 바뀐다면 문제가 있겠죠? '당신이 누군지는 모르겠지만, 일단 방문하시기로 하셨다니 들어가 계세요.' 라고 한다면 출입통제하는 의미가 없을겁니다.

마찬가지로 Spring Security의 Filter 도 오피스 빌딩의 출입을 통제하듯 인증, 인가가 순서대로 적용되어야 합니다. 예를 들어, 로그인 하기 전에 회원에게만 제공하는 서비스를 이용 가능하게 하지는 않습니다.

실제로 Filter가 추가되는 것은 Spring Security 를 의존성 라이브러리 목록에 추가하고 Spring Boot 프로젝트를 실행 시키면 아래와 같은 로그를 확인할 수 있습니다.

아래는 Spring Security 맛보기 글([링크](https://limvik.github.io/posts/study-spring-security-1-just-do-it/))에서 사용했던 프로젝트(Spring Security 의존성 추가 및 hello 문자열을 반환하는 RestController가 있는 간단한 프로젝트) 실행 시 로그입니다. 실제로는 모두 한 줄로 나열되어 있어서 저는 사실 글을 쓰면서 Filter 로그가 표시된다는걸 알았습니다.

```
2023-07-08T20:25:11.740+09:00  INFO 18352 --- [           main] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [org.springframework.security.web.session.DisableEncodeUrlFilter@4fe533ff,
org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@5377414a,
org.springframework.security.web.context.SecurityContextHolderFilter@1ee47d9e,
org.springframework.security.web.header.HeaderWriterFilter@340d6d89,
org.springframework.security.web.csrf.CsrfFilter@1b2df3aa,
org.springframework.security.web.authentication.logout.LogoutFilter@7978e022,
org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@15f35bc3,
org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@2e766822,
org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@4e83a98,
org.springframework.security.web.authentication.www.BasicAuthenticationFilter@20a7953c,
org.springframework.security.web.savedrequest.RequestCacheAwareFilter@5dc0ff7d,
org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@13dbed9e,
org.springframework.security.web.authentication.AnonymousAuthenticationFilter@17aa8a11,
org.springframework.security.web.access.ExceptionTranslationFilter@480b57e2,
org.springframework.security.web.access.intercept.AuthorizationFilter@76a805b7]
```

DefaultSecurityFilterChain 이 모든 요청에 대해 적용되고 총 15개의 Filter가 등록되어 있는 것을 확인할 수 있습니다.

## Outro

이번 글은 내용 자체는 별로 없었습니다. 요약해 보자면 SecurityFilterChain에 등록되는 Filter 종류가 굉장히 많고, 순서가 중요하다는 것과 URL 에 따라 다른 Filter가 적용될 수 있다는 것이었습니다.

다음에는 직접 요청을 보내서 Filter가 실제로 순서대로 적용되는지 확인해보겠습니다.
