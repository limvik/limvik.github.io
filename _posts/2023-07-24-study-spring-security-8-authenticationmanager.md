---
layout: post
title: Spring Security (8) AuthenticationManager
categories:
- Framework
- Spring
- Spring Security
tags:
- Spring
- Spring Security
date: 2023-07-24 21:23 +0900
---
## Intro

[이전 글](https://limvik.github.io/posts/study-spring-security-7-securitycontext-authentication-grantedauthority/)에서 SecurityContext, Authentication, GrantedAuthority를 살펴봤고, 이어서 공식 문서([링크](https://limvik.github.io/posts/study-spring-security-7-securitycontext-authentication-grantedauthority/)) 순서대로 [AuthenticationManager](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/authentication/AuthenticationManager.html) 부터 살펴보겠습니다.

## AuthenticationManager

이전 글에서 AuthenticationManager는 인증 절차를 수행하겠구나... 하면서 넘어갔습니다.

### 문서 살펴보기

공식 문서([Servlet Authentication Architecture](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationmanager:~:text=AuthenticationManager%20--,the%20API%20that%20defines%20how%20Spring%20Security%E2%80%99s%20Filters%20perform%20authentication.,-ProviderManager%20-%20the%20most))의 가장 상단에 나온 한 줄짜리 설명에는 이렇게 나옵니다.

>the API that defines how Spring Security’s Filters perform authentication.
>
>Spring Security의 Filters가 인증을 수행하는 방법을 정의하는 API입니다.

AuthenticationManager interface API 문서에도 아주 간단하게 설명합니다.

>Processes an [`Authentication`](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/core/Authentication.html "interface in org.springframework.security.core") request.

Override 할 메서드가 한 개 밖에 없어서, 그대로 붙여넣겠습니다.

|Modifier and Type|Method|Description|
|--|--|--|
|Authentication|authenticate(Authentication authentication)|Attempts to authenticate the passed Authentication object, returning a fully populated Authentication object (including granted authorities) if successful.|

메서드 설명을 보면, 전달된 Authentication 객체(이전 글에서 봤던 Token들)의 인증 시도 후 성공하면 완전히 채워진 Authentication 객체(부여된 권한 포함)를 반환한다고 써있습니다.

완전히 채워진(fully populated)의 의미가 잘 와닿지는 않습니다. Authentication 객체의 주요 목적 중 하나가 `사용자가 인증을 위해 제공한 credentials를 AuthenticationManager의 입력으로 제공` 하는 것이라는 이전의 정보([링크](https://limvik.github.io/posts/study-spring-security-7-securitycontext-authentication-grantedauthority/#authentication-%EA%B0%9D%EC%B2%B4%EC%9D%98-%EC%A3%BC%EC%9A%94-%EB%AA%A9%EC%A0%81))를 바탕으로 추측해보면 Principal과 Credentials 를 바탕으로 인증을 수행하고, Authorities 를 채워넣는다는 의미라고 보면 되겠습니다.

![SecurityContextHolder](/assets/img/2023-07-12-study-spring-security-6-SecurityContextHolder/01-securitycontextholder.png)

이미 문서에서 한 줄로 정리했지만, 다시 또 정리해 보자면 AuthenticationManager는 `어떻게` 인증할 것인지 정의한다고 할 수 있겠습니다.

그리고 공식 문서의 AuthenticationManager 섹션([링크](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationmanager))을 보면, 아래와 같이 AuthenticationManager 가 필요한 경우와 필요 없는 경우를 설명하고 있습니다.

>The [`Authentication`](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authentication) that is returned is then set on the [SecurityContextHolder](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-securitycontextholder) by the controller (that is, by [Spring Security’s  `Filters`  instances](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters)) that invoked the `AuthenticationManager`. If you are not integrating with Spring Security’s `Filters` instances, you can set the `SecurityContextHolder` directly and are not required to use an `AuthenticationManager`.

controller(Spring Security의 Filters 인스턴스)에 의해서 SecurityContextHolder에 Authentication 이 저장되는 경우에는 AuthenticationManager가 호출되고 사용되지만, 개발자가 직접 SecurityContextHolder에 Authentication을 저장하는 경우에는 사용할 필요가 없다고 합니다.

### 이전에 본 Filters 살펴보기

이전에 봤던 Filters 중에 Auhtentication 이 붙은 Filter가 문서에서 말하는 Controller라고 보면 되겠습니다. 아래 DefaultSecurityFilterChain 로그에서는 UsernamePassword`Authentication`Filter, Basic`Authentication`Filter, Anonymous`Authentication`Filter 가 보입니다.

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

### BasicAuthenticationFilter

BasicAuthenticationFilter는 간단하게 다루기도 했었습니다([링크](https://limvik.github.io/posts/study-spring-security-6-securitycontextholder/#basicauthenticationfilter)). 다이어그램을 다시 잠깐 보면, 위에서 controller라 칭하는 BasicAuthenticationFilter 가 Authentication 객체인 UsernamePasswordAuthenticationToken 을 AuthenticationManager 객체를 호출하면서 입력으로 제공합니다. 그리고 이제 AuthenticationManager 에서 Token을 어떻게 처리하는지 보려고 하고 있는 것입니다.

![BasicAuthenticationFilter Diagram](/assets/img/2023-07-12-study-spring-security-6-SecurityContextHolder/03-basicauthenticationfilter.png)

Filter의 `doFilterInternal` 메서드에서 AuthenticationManager 에 인증을 요청하는 것을 볼 수 있습니다.

```java
// BasicAuthenticationFilter.java
if (authenticationIsRequired(username)) {
	Authentication authResult = this.authenticationManager.authenticate(authRequest);
	SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();
```

### AnonymousAuthenticationFilter

AnonymousAuthenticationFilter는 인증할 정보가 없으니 바로 Authentication 객체를 SecurityContext에 저장하기는 합니다.

```java
// AnonymousAuthenticationFilter.java
private SecurityContext defaultWithAnonymous(HttpServletRequest request, SecurityContext currentContext) {
	Authentication currentAuthentication = currentContext.getAuthentication();
	if (currentAuthentication == null) {
		Authentication anonymous = createAuthentication(request);
		if (this.logger.isTraceEnabled()) {
			this.logger.trace(LogMessage.of(() -> "Set SecurityContextHolder to " + anonymous));
		}
		else {
			this.logger.debug("Set SecurityContextHolder to anonymous SecurityContext");
		}
		SecurityContext anonymousContext = this.securityContextHolderStrategy.createEmptyContext();
		anonymousContext.setAuthentication(anonymous);
		return anonymousContext;
	}
	else {
		if (this.logger.isTraceEnabled()) {
			this.logger.trace(LogMessage.of(() -> "Did not set SecurityContextHolder since already authenticated "
					+ currentAuthentication));
		}
	}
	return currentContext;
}
```

### 다시 문서로

다음으로 문서에 AuthenticationManager 에 관하여 써있는 마지막 문장을 보면 아래와 같습니다.

> While the implementation of `AuthenticationManager` could be anything, the most common implementation is `ProviderManager`.

AuthenticationManager의 구현은 무엇이든 될 수 있지만, 가장 흔한 구현은 [ProviderManager](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/authentication/ProviderManager.html) 라고 합니다.

Intellij 로 보면, ProviderManager 도 보이고, 다양한 AuthenticationManager 구현이 있지만 Test 내부에 있는 것이 많이 보입니다.

![Intellij로 본 AuthenticationManager 구현](/assets/img/2023-07-24-study-spring-security-8-AuthenticationManager/01-AuthenticationManager-impl-ProviderManager.png)

Test 외에도 여러 Parser, Resolver 등 클래스의 내부 클래스로 구현된 것들을 볼 수 있습니다.

![Intellij로 본 AuthenticationManager 구현2](/assets/img/2023-07-24-study-spring-security-8-AuthenticationManager/02-AuthenticationManager-impls.png)

ProviderManager 를 사용하지 않는 구현들은 각 개별적인 상황이 필요할 때 살펴보면 될것 같고, 문서 순서대로 ProviderManager를 봐야하는데... 문서를 보니 내용이 길어질 것 같아 예고편 마냥 맛만 보고 가겠습니다.

## ProviderManager

공식 문서(링크) 첫 문단에 있는 문장 중에는 아래와 같이 ProviderManager는 AuthenticationProvider 인스턴스의 목록에 위임한다고 합니다.

> `ProviderManager` delegates to a `List` of [`AuthenticationProvider`](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationprovider) instances.

![ProviderManager with AuthenticationProviders Diagram](/assets/img/2023-07-24-study-spring-security-8-AuthenticationManager/03-providermanager-and-authentication-providers.png)

마치 SecurityFilterChain 내에 여러 Filter가 순서대로 처리되는 것과 같은 패턴으로 보입니다.

## Outro

ProviderManager는 정말 맛만 보고 마무리 합니다. Servlet Authentication Architecture 내용을 언제 다 보나 했는데, 섹션 기준으로는 벌써 절반을 넘겼습니다(9개 중에 5개). 다음에는 맛만 본 AuthenticationManager의 구현 중 하나인 ProviderManager를 살펴보겠습니다.
