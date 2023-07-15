---
layout: post
title: Spring Security (5) Log와 Code 확인해보기
categories:
- Framework
- Spring
- Spring Security
tags:
- Spring
- Spring Security
date: 2023-07-09 21:11 +0900
---
## Intro

짧게 끊어서 쓰다보니 벌써 다섯 번째 가 됐습니다. Spring Security 무료 강의([링크](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0#curriculum))가 인프런에 있다는 정보를 입수했습니다. 아직 보지는 못했지만, 무료버프 받은거 치고도 평점이 높은걸 보니 괜찮은 강의인가 봅니다.

동영상 강의를 보고있으면 알고있다는 착각에 빠질 때가 많아서 별로 선호하지는 않지만, Spring Security를 개인적으로 어느정도 정리하고 나면 잘못알고 있는게 없는지 확인하고, 새로운 정보를 얻을 겸 들어봐야겠습니다.

그러면 이제 [이전 글](https://limvik.github.io/posts/study-spring-security-4-securityfilterchain-and-filters/)에 이어서 DefaultSecurityFilterChain에 있는 Filter가 순서대로 적용되는지, 프로젝트를 실행해서 요청을 보내 Log로 확인해보겠습니다.

## Log 설정하기

첫 번째 글([링크](https://limvik.github.io/posts/study-spring-security-1-just-do-it/))에서 사용한 프로젝트를 사용합니다.

로그를 확인하기 위해서 logback.xml 파일을 resources 디렉터리에 추가해주고 아래 내용을 추가해줍니다.

```xml
<configuration>
    <!-- 로그 출력을 콘솔로 설정 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 로거 설정 -->
    <root level="warn">
        <appender-ref ref="CONSOLE" />
    </root>
    <logger name="org.springframework.security" level="trace" additivity="false">
        <appender-ref ref="CONSOLE" />
    </logger>
</configuration>
```

default 비밀번호가 main thread에서 로그 레벨 WARN으로 출력되기 때문에 Spring Security Log 외에도 추가로 설정을 해줬습니다.

## 프로젝트 시작

그리고 시작해보면 아래와 같이 로그가 출력되는 것을 볼 수 있습니다. 여기에 작성하는 로그는 시간이 중요하지 않으므로 패턴에서 뺐습니다.

```
[main] TRACE o.s.s.c.a.a.c.AuthenticationConfiguration$EnableGlobalAuthenticationAutowiredConfigurer - Eagerly initializing {org.springframework.boot.autoconfigure.security.servlet.SpringBootWebSecurityConfiguration$WebSecurityEnablerConfiguration=org.springframework.boot.autoconfigure.security.servlet.SpringBootWebSecurityConfiguration$WebSecurityEnablerConfiguration@69f0b0f4}
[main] WARN  o.s.b.a.s.s.UserDetailsServiceAutoConfiguration - 

Using generated security password: 01fc7b16-231f-4258-ba1a-91ba59aea4bd

This generated password is for development use only. Your security configuration must be updated before running your application in production.

[main] INFO  o.s.s.web.DefaultSecurityFilterChain - Will secure any request with [org.springframework.security.web.session.DisableEncodeUrlFilter@6968c1d6, org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@7d986d83, org.springframework.security.web.context.SecurityContextHolderFilter@1f939a0f, org.springframework.security.web.header.HeaderWriterFilter@18b74ea, org.springframework.security.web.csrf.CsrfFilter@4a92c6a9, org.springframework.security.web.authentication.logout.LogoutFilter@6088451e, org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@5416f8db, org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@1665fa54, org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@77f991c, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@798dad3d, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@430b2699, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@17b37e9a, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@3a7e365, org.springframework.security.web.access.ExceptionTranslationFilter@14c93774, org.springframework.security.web.access.intercept.AuthorizationFilter@502a4156]
```

Spring Boot Auto Configuration 으로 Spring Security 가 설정되고, DefaultSecurityFilterChain에 있는 Filter로 모든 요청을 보호할 것이라고 Log에 기록된 것을 볼 수 있습니다.

### HTTP Request 인증 성공

Log에 출력된 비밀번호를 이용해서 인증 가능한 HTTP Request를 보내면, 아래와 같이 15개의 Filter가 순서대로 출력되는 것을 확인할 수 있습니다. 요청을 보내는 자세한 내용은 첫 번째 글([링크](https://limvik.github.io/posts/study-spring-security-1-just-do-it/#postman-%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-http-get-request))을 참고하시면 됩니다.

```
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Trying to match request against DefaultSecurityFilterChain [RequestMatcher=any request, Filters=[org.springframework.security.web.session.DisableEncodeUrlFilter@6968c1d6, org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@7d986d83, org.springframework.security.web.context.SecurityContextHolderFilter@1f939a0f, org.springframework.security.web.header.HeaderWriterFilter@18b74ea, org.springframework.security.web.csrf.CsrfFilter@4a92c6a9, org.springframework.security.web.authentication.logout.LogoutFilter@6088451e, org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@5416f8db, org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@1665fa54, org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@77f991c, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@798dad3d, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@430b2699, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@17b37e9a, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@3a7e365, org.springframework.security.web.access.ExceptionTranslationFilter@14c93774, org.springframework.security.web.access.intercept.AuthorizationFilter@502a4156]] (1/1)
[http-nio-8080-exec-2] DEBUG o.s.security.web.FilterChainProxy - Securing GET /hello
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking DisableEncodeUrlFilter (1/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking WebAsyncManagerIntegrationFilter (2/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking SecurityContextHolderFilter (3/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking HeaderWriterFilter (4/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking CsrfFilter (5/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.csrf.CsrfFilter - Did not protect against CSRF since request did not match CsrfNotRequired [TRACE, HEAD, GET, OPTIONS]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking LogoutFilter (6/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.logout.LogoutFilter - Did not match request to Ant [pattern='/logout', POST]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking UsernamePasswordAuthenticationFilter (7/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.UsernamePasswordAuthenticationFilter - Did not match request to Ant [pattern='/login', POST]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking DefaultLoginPageGeneratingFilter (8/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking DefaultLogoutPageGeneratingFilter (9/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.u.DefaultLogoutPageGeneratingFilter - Did not render default logout page since request did not match [Ant [pattern='/logout', GET]]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking BasicAuthenticationFilter (10/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.w.BasicAuthenticationFilter - Found username 'user' in Basic Authorization header
[http-nio-8080-exec-2] TRACE o.s.s.w.c.HttpSessionSecurityContextRepository - No HttpSession currently exists
[http-nio-8080-exec-2] TRACE o.s.s.w.c.SupplierDeferredSecurityContext - Created SecurityContextImpl [Null authentication]
[http-nio-8080-exec-2] TRACE o.s.s.w.c.SupplierDeferredSecurityContext - Created SecurityContextImpl [Null authentication]
[http-nio-8080-exec-2] TRACE o.s.s.authentication.ProviderManager - Authenticating request with DaoAuthenticationProvider (1/1)
[http-nio-8080-exec-2] DEBUG o.s.s.a.d.DaoAuthenticationProvider - Authenticated user
[http-nio-8080-exec-2] DEBUG o.s.s.w.a.w.BasicAuthenticationFilter - Set SecurityContextHolder to UsernamePasswordAuthenticationToken [Principal=org.springframework.security.core.userdetails.User [Username=user, Password=[PROTECTED], Enabled=true, AccountNonExpired=true, credentialsNonExpired=true, AccountNonLocked=true, Granted Authorities=[]], Credentials=[PROTECTED], Authenticated=true, Details=WebAuthenticationDetails [RemoteIpAddress=0:0:0:0:0:0:0:1, SessionId=null], Granted Authorities=[]]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking RequestCacheAwareFilter (11/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.s.HttpSessionRequestCache - matchingRequestParameterName is required for getMatchingRequest to lookup a value, but not provided
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking SecurityContextHolderAwareRequestFilter (12/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking AnonymousAuthenticationFilter (13/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking ExceptionTranslationFilter (14/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking AuthorizationFilter (15/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.i.RequestMatcherDelegatingAuthorizationManager - Authorizing SecurityContextHolderAwareRequestWrapper[ org.springframework.security.web.header.HeaderWriterFilter$HeaderWriterRequest@5cfb7d9c]
[http-nio-8080-exec-2] TRACE o.s.s.w.a.i.RequestMatcherDelegatingAuthorizationManager - Checking authorization on SecurityContextHolderAwareRequestWrapper[ org.springframework.security.web.header.HeaderWriterFilter$HeaderWriterRequest@5cfb7d9c] using org.springframework.security.authorization.AuthenticatedAuthorizationManager@4399d164
[http-nio-8080-exec-2] TRACE o.s.s.w.a.AnonymousAuthenticationFilter - Did not set SecurityContextHolder since already authenticated UsernamePasswordAuthenticationToken [Principal=org.springframework.security.core.userdetails.User [Username=user, Password=[PROTECTED], Enabled=true, AccountNonExpired=true, credentialsNonExpired=true, AccountNonLocked=true, Granted Authorities=[]], Credentials=[PROTECTED], Authenticated=true, Details=WebAuthenticationDetails [RemoteIpAddress=0:0:0:0:0:0:0:1, SessionId=null], Granted Authorities=[]]
[http-nio-8080-exec-2] DEBUG o.s.security.web.FilterChainProxy - Secured GET /hello
[http-nio-8080-exec-2] TRACE o.s.s.w.h.writers.HstsHeaderWriter - Not injecting HSTS header since it did not match request to [Is Secure]
```

FilterChainProxy 에서 Filter를 순서에 맞게 호출(invoking)하고, 각 Filter 는 각자 작업을 수행하면서 일부 Log를 남긴 것을 볼 수 있습니다.

FilterChainProxy를 호출하는 DelegatingFilterProxy 는 Spring Framework에 포함된 클래스이다 보니 Log에서는 안보입니다. 그래서 Debugger를 이용해서 확인해보면 아래와 같이 doFilter 메서드 안에서 FilterChainProxy를 불러오고, FilterChainProxy 안에는 DefaultSecurityFilterChain과 15개의 Filter가 포함되어 있는 것을 볼 수 있습니다.

![Debugger로 확인한 DelegatingFilterProxy 호출](/assets/img/2023-07-09-study-spring-security-5-check-logs-and-code/01-DelegatingFilterProxy-in-Debugger.png)

이렇게 공식 문서에서 확인했던대로 DelegatingFilterProxy -> FilterChainProxy -> SecurityFilterChain -> Filters 순서대로 흘러가고 있는 것을 확인할 수 있었습니다.

흐름을 보기로 했던 목적에 맞게 실패하는 요청도 해보겠습니다.

### HTTP Request 인증 실패

프로젝트를 다시 시작(비밀번호 새로 생성)하고, 다시 시작하기 전의 Authorization header 그대로 요청해서 인증에 실패한 로그를 보겠습니다.

```
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Trying to match request against DefaultSecurityFilterChain [RequestMatcher=any request, Filters=[org.springframework.security.web.session.DisableEncodeUrlFilter@4cad79bc, org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@2c63762b, org.springframework.security.web.context.SecurityContextHolderFilter@6d67f5eb, org.springframework.security.web.header.HeaderWriterFilter@29c17c3d, org.springframework.security.web.csrf.CsrfFilter@124dac75, org.springframework.security.web.authentication.logout.LogoutFilter@b0fd744, org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@696db620, org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@6ff0b1cc, org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@7a9eccc4, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@3f6bf8aa, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@3e1fd62b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@70025b99, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@78422efb, org.springframework.security.web.access.ExceptionTranslationFilter@6bcb12e6, org.springframework.security.web.access.intercept.AuthorizationFilter@27abb6ca]] (1/1)
[http-nio-8080-exec-2] DEBUG o.s.security.web.FilterChainProxy - Securing GET /hello
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking DisableEncodeUrlFilter (1/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking WebAsyncManagerIntegrationFilter (2/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking SecurityContextHolderFilter (3/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking HeaderWriterFilter (4/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking CsrfFilter (5/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.csrf.CsrfFilter - Did not protect against CSRF since request did not match CsrfNotRequired [TRACE, HEAD, GET, OPTIONS]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking LogoutFilter (6/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.logout.LogoutFilter - Did not match request to Ant [pattern='/logout', POST]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking UsernamePasswordAuthenticationFilter (7/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.UsernamePasswordAuthenticationFilter - Did not match request to Ant [pattern='/login', POST]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking DefaultLoginPageGeneratingFilter (8/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking DefaultLogoutPageGeneratingFilter (9/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.u.DefaultLogoutPageGeneratingFilter - Did not render default logout page since request did not match [Ant [pattern='/logout', GET]]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking BasicAuthenticationFilter (10/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.w.BasicAuthenticationFilter - Found username 'user' in Basic Authorization header
[http-nio-8080-exec-2] TRACE o.s.s.w.c.HttpSessionSecurityContextRepository - No HttpSession currently exists
[http-nio-8080-exec-2] TRACE o.s.s.w.c.SupplierDeferredSecurityContext - Created SecurityContextImpl [Null authentication]
[http-nio-8080-exec-2] TRACE o.s.s.w.c.SupplierDeferredSecurityContext - Created SecurityContextImpl [Null authentication]
[http-nio-8080-exec-2] TRACE o.s.s.authentication.ProviderManager - Authenticating request with DaoAuthenticationProvider (1/1)
[http-nio-8080-exec-2] DEBUG o.s.s.a.d.DaoAuthenticationProvider - Failed to authenticate since password does not match stored value
[http-nio-8080-exec-2] DEBUG o.s.s.w.a.w.BasicAuthenticationFilter - Failed to process authentication request
org.springframework.security.authentication.BadCredentialsException: 자격 증명에 실패하였습니다.
[http-nio-8080-exec-2] DEBUG o.s.s.w.a.DelegatingAuthenticationEntryPoint - Trying to match using RequestHeaderRequestMatcher [expectedHeaderName=X-Requested-With, expectedHeaderValue=XMLHttpRequest]
[http-nio-8080-exec-2] DEBUG o.s.s.w.a.DelegatingAuthenticationEntryPoint - No match found. Using default entry point org.springframework.security.web.authentication.www.BasicAuthenticationEntryPoint@2220c763
[http-nio-8080-exec-2] TRACE o.s.s.w.h.writers.HstsHeaderWriter - Not injecting HSTS header since it did not match request to [Is Secure]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Trying to match request against DefaultSecurityFilterChain [RequestMatcher=any request, Filters=[org.springframework.security.web.session.DisableEncodeUrlFilter@4cad79bc, org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@2c63762b, org.springframework.security.web.context.SecurityContextHolderFilter@6d67f5eb, org.springframework.security.web.header.HeaderWriterFilter@29c17c3d, org.springframework.security.web.csrf.CsrfFilter@124dac75, org.springframework.security.web.authentication.logout.LogoutFilter@b0fd744, org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@696db620, org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@6ff0b1cc, org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@7a9eccc4, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@3f6bf8aa, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@3e1fd62b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@70025b99, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@78422efb, org.springframework.security.web.access.ExceptionTranslationFilter@6bcb12e6, org.springframework.security.web.access.intercept.AuthorizationFilter@27abb6ca]] (1/1)
[http-nio-8080-exec-2] DEBUG o.s.security.web.FilterChainProxy - Securing GET /error
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking DisableEncodeUrlFilter (1/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking WebAsyncManagerIntegrationFilter (2/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking SecurityContextHolderFilter (3/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking HeaderWriterFilter (4/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking CsrfFilter (5/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking LogoutFilter (6/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.logout.LogoutFilter - Did not match request to Ant [pattern='/logout', POST]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking UsernamePasswordAuthenticationFilter (7/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.UsernamePasswordAuthenticationFilter - Did not match request to Ant [pattern='/login', POST]
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking DefaultLoginPageGeneratingFilter (8/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking DefaultLogoutPageGeneratingFilter (9/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking BasicAuthenticationFilter (10/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking RequestCacheAwareFilter (11/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.s.HttpSessionRequestCache - matchingRequestParameterName is required for getMatchingRequest to lookup a value, but not provided
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking SecurityContextHolderAwareRequestFilter (12/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking AnonymousAuthenticationFilter (13/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking ExceptionTranslationFilter (14/15)
[http-nio-8080-exec-2] TRACE o.s.security.web.FilterChainProxy - Invoking AuthorizationFilter (15/15)
[http-nio-8080-exec-2] TRACE o.s.s.w.a.i.RequestMatcherDelegatingAuthorizationManager - Authorizing SecurityContextHolderAwareRequestWrapper[ FirewalledRequest[ org.apache.catalina.core.ApplicationHttpRequest@26f70c7d]]
[http-nio-8080-exec-2] TRACE o.s.s.w.a.i.RequestMatcherDelegatingAuthorizationManager - Checking authorization on SecurityContextHolderAwareRequestWrapper[ FirewalledRequest[ org.apache.catalina.core.ApplicationHttpRequest@26f70c7d]] using org.springframework.security.authorization.AuthenticatedAuthorizationManager@45cf1a18
[http-nio-8080-exec-2] TRACE o.s.s.w.c.HttpSessionSecurityContextRepository - No HttpSession currently exists
[http-nio-8080-exec-2] TRACE o.s.s.w.c.SupplierDeferredSecurityContext - Created SecurityContextImpl [Null authentication]
[http-nio-8080-exec-2] TRACE o.s.s.w.c.SupplierDeferredSecurityContext - Created SecurityContextImpl [Null authentication]
[http-nio-8080-exec-2] TRACE o.s.s.w.a.AnonymousAuthenticationFilter - Set SecurityContextHolder to AnonymousAuthenticationToken [Principal=anonymousUser, Credentials=[PROTECTED], Authenticated=true, Details=WebAuthenticationDetails [RemoteIpAddress=0:0:0:0:0:0:0:1, SessionId=null], Granted Authorities=[ROLE_ANONYMOUS]]
[http-nio-8080-exec-2] TRACE o.s.s.w.a.ExceptionTranslationFilter - Sending AnonymousAuthenticationToken [Principal=anonymousUser, Credentials=[PROTECTED], Authenticated=true, Details=WebAuthenticationDetails [RemoteIpAddress=0:0:0:0:0:0:0:1, SessionId=null], Granted Authorities=[ROLE_ANONYMOUS]] to authentication entry point since access is denied
org.springframework.security.access.AccessDeniedException: Access Denied
[http-nio-8080-exec-2] DEBUG o.s.s.w.s.HttpSessionRequestCache - Saved request http://localhost:8080/error?continue to session
[http-nio-8080-exec-2] DEBUG o.s.s.w.a.DelegatingAuthenticationEntryPoint - Trying to match using And [Not [RequestHeaderRequestMatcher [expectedHeaderName=X-Requested-With, expectedHeaderValue=XMLHttpRequest]], MediaTypeRequestMatcher [contentNegotiationStrategy=org.springframework.web.accept.ContentNegotiationManager@37c4b7be, matchingMediaTypes=[application/xhtml+xml, image/*, text/html, text/plain], useEquals=false, ignoredMediaTypes=[*/*]]]
[http-nio-8080-exec-2] DEBUG o.s.s.w.a.DelegatingAuthenticationEntryPoint - Trying to match using Or [RequestHeaderRequestMatcher [expectedHeaderName=X-Requested-With, expectedHeaderValue=XMLHttpRequest], And [Not [MediaTypeRequestMatcher [contentNegotiationStrategy=org.springframework.web.accept.ContentNegotiationManager@37c4b7be, matchingMediaTypes=[text/html], useEquals=false, ignoredMediaTypes=[]]], MediaTypeRequestMatcher [contentNegotiationStrategy=org.springframework.web.accept.ContentNegotiationManager@37c4b7be, matchingMediaTypes=[application/atom+xml, application/x-www-form-urlencoded, application/json, application/octet-stream, application/xml, multipart/form-data, text/xml], useEquals=false, ignoredMediaTypes=[*/*]]], MediaTypeRequestMatcher [contentNegotiationStrategy=org.springframework.web.accept.ContentNegotiationManager@37c4b7be, matchingMediaTypes=[*/*], useEquals=true, ignoredMediaTypes=[]]]
[http-nio-8080-exec-2] DEBUG o.s.s.w.a.DelegatingAuthenticationEntryPoint - Match found! Executing org.springframework.security.web.authentication.DelegatingAuthenticationEntryPoint@798256c5
[http-nio-8080-exec-2] DEBUG o.s.s.w.a.DelegatingAuthenticationEntryPoint - Trying to match using RequestHeaderRequestMatcher [expectedHeaderName=X-Requested-With, expectedHeaderValue=XMLHttpRequest]
[http-nio-8080-exec-2] DEBUG o.s.s.w.a.DelegatingAuthenticationEntryPoint - No match found. Using default entry point org.springframework.security.web.authentication.www.BasicAuthenticationEntryPoint@2220c763
```

일부 메시지를 제거했는데도 많이 깁니다. Log 보신 분은 없을 것이라 생각되므로, 몇 가지 눈길이 갔던 것들만 살펴보겠습니다.

BasicAuthenticationFilter (10/15) 까지만 나오는 것을 볼 수 있습니다.  그리고 `자격 증명에 실패하였습니다.` 라는 메시지를 확인할 수 있습니다.

```
org.springframework.security.authentication.BadCredentialsException: 자격 증명에 실패하였습니다.
```

그리고 다시 /error 경로로 redirect 되는 것을 확인할 수 있습니다.

```
[http-nio-8080-exec-2] DEBUG o.s.security.web.FilterChainProxy - Securing GET /error
```

이후에 최종적으로 접근이 거부됐다는 로그를 확인할 수 있습니다.

```
org.springframework.security.access.AccessDeniedException: Access Denied
```

이러한 흐름의 일부를 문서에서 확인할 수 있습니다.

## 다시 공식 문서 살펴보기

Architecture 문서([링크](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-exceptiontranslationfilter))를 보면 ExceptionTranslationFilter 에 의해 다음과 같은 흐름이 나타난다고 명시하고 있습니다.

![ExceptionTranslationFilter 다이어그램](/assets/img/2023-07-09-study-spring-security-5-check-logs-and-code/02-ExceptionTranslationFilter.png)

DefaultSecurityFilterChain 에 있는 15개의 Filter 중에 14번째 순번을 갖는 Filter 입니다. 그런데 앞서 실패했을 때 Log 에는 10번째 Filter인 BasicAuthenticationFilter 이후 다른 Filter에 대한 Log는 볼 수 없습니다.

~~Debugger로 확인해보면 ExceptionTranslationFilter 가 호출되는 것을 볼 수 있습니다. 상황에 따라 Filter에서 Log가 안남는 경우가 있는 것으로 보입니다.
![Debugger로 확인한 ExceptionTranslationFilter 호출](/assets/img/2023-07-09-study-spring-security-5-check-logs-and-code/03-ExceptionTranslatinFilter-in-debugger.png)~~

**다음 글을 쓰다가 Log가 안남는게 아니라 호출되지 않는 것을 발견했습니다. GET /Error 가 호출됐을 때 ExceptionTranslationFilter 가 호출된 것을 제가 착각한 것으로 보입니다. 다음 글을 참고 부탁드립니다.([링크](https://limvik.github.io/posts/study-spring-security-6-securitycontextholder/#:~:text=Debugger%20%EB%A5%BC%20%ED%86%B5%ED%95%B4%20%ED%99%95%EC%9D%B8%ED%95%98%EB%8B%88%20this.ignoreFailure%20%EB%8A%94%20false%20%EB%A1%9C%20%EC%84%A4%EC%A0%95%EB%90%98%EC%96%B4%20%EC%9E%88%EC%96%B4%EC%84%9C%20%EB%8B%A4%EC%9D%8C%20Filter%EB%8A%94%20%ED%98%B8%EC%B6%9C%EB%90%98%EC%A7%80%20%EC%95%8A%EB%8A%94%20%EA%B2%83%EC%9D%84%20%EB%B3%BC%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.))**

ExceptionTranslationFilter 에 대해 문서에 적힌 내용과 다이어그램을 다시 보면, [`AuthenticationException`](https://docs.spring.io/spring-security/site/docs/6.1.1/api//org/springframework/security/core/AuthenticationException.html)이 던져지는 경우 Start Authentication, [`AccessDeniedException`](https://docs.spring.io/spring-security/site/docs/6.1.1/api/org/springframework/security/access/AccessDeniedException.html)이 던져지는 경우 Access Denied로 흘러갑니다.

![ExceptionTranslationFilter 다이어그램](/assets/img/2023-07-09-study-spring-security-5-check-logs-and-code/02-ExceptionTranslationFilter.png)

문서의 설명에 따르면, [`FilterSecurityInterceptor`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/access/intercept/FilterSecurityInterceptor.html)(문서 그대로 가져왔지만 현재는 deprecated 되었고, [AuthorizationFilter](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/access/intercept/AuthorizationFilter.html)를 사용합니다.) 또는 method security(나중에 다룰 예정)에서 AuthenticationException 이나 AccessDeniedException 이 던져지는 경우 각 Exception에 맞는 절차가 시작되고, AuthenticationException 이나 AccessDeniedException 이 던져지지 않는 경우 ExceptionTranslationFilter 는 아무것도 하지 않습니다.

코드를 보면 try 문에서 다음 filter를 호출하고 catch 문에서 Exception 종류를 식별하는 것을 볼 수 있습니다.

```java
// ExceptionTranslationFilter.java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
    try {
        chain.doFilter(request, response);
    } catch (IOException var7) {
        throw var7;
    } catch (Exception var8) {
        Throwable[] causeChain = this.throwableAnalyzer.determineCauseChain(var8);
        RuntimeException securityException = (AuthenticationException)this.throwableAnalyzer.getFirstThrowableOfType(AuthenticationException.class, causeChain);
        if (securityException == null) {
            securityException = (AccessDeniedException)this.throwableAnalyzer.getFirstThrowableOfType(AccessDeniedException.class, causeChain);
        }

        if (securityException == null) {
            this.rethrow(var8);
        }

        if (response.isCommitted()) {
            throw new ServletException("Unable to handle the Spring Security Exception because the response is already committed.", var8);
        }

        this.handleSpringSecurityException(request, response, chain, (RuntimeException)securityException);
    }

}
```

### 다시 관련 Log 살펴보기

인증에 성공했던 Log를 다시 확인해보면, 이미 인증된 사용자이기 때문에 SecurityContextHolder에 새로 저장하지 않는다고 합니다.

```
[http-nio-8080-exec-2] TRACE o.s.s.w.a.AnonymousAuthenticationFilter
Did not set SecurityContextHolder since already authenticated UsernamePasswordAuthenticationToken [Principal=org.springframework.security.core.userdetails.User
[Username=user, Password=[PROTECTED], Enabled=true, AccountNonExpired=true, credentialsNonExpired=true, AccountNonLocked=true, Granted Authorities=[]], Credentials=[PROTECTED], Authenticated=true, Details=WebAuthenticationDetails [RemoteIpAddress=0:0:0:0:0:0:0:1, SessionId=null], Granted Authorities=[]]
```

이미 인증된 상태이므로 AuthenticationException 이 발생하지 않은 것입니다.

만약 다이어그램 상의 Start Authentication 절차가 진행되는 경우 아래와 같은 절차가 진행됨을 문서에서 설명하고 있습니다.

① 인증된 사용자 정보가 저장되는 `SecurityContextHolder` 를 비우기

② `RequestCache` 에 인증 성공 시 되돌아갈 HttpServletRequest 를 저장

③ 로그인 페이지로 redirect 하거나 WWW-authenticate header 를 보내는 등 클라이언트로부터 인증 정보(credential)를 받기 위해 `AuthenticationEntryPoint` 사용

> 참고: 401(Unauthorized) 응답을 생성하는 서버는 반드시 하나 이상의 챌린지가 포함된 WWW-Authenticate header 필드를 보내야 합니다. ([RFC 7235](https://datatracker.ietf.org/doc/html/rfc7235#section-4.1))
>
>![postman 으로 401 응답을 받은 화면](/assets/img/2023-07-09-study-spring-security-5-check-logs-and-code/04-postman-401.png)

Start Authentication 절차에 대한 세부적인 내용은 Authentication 문서에서 살펴보기로 하고 여기서 끊어야겠습니다.

그전에 잘못된 인증정보를 입력한 경우를 간단히 보면, AccessDeniedException에 대한 Log를 확인할 수 있었습니다.

```
org.springframework.security.access.AccessDeniedException: Access Denied
```

## Outro

이번에도 Log가 대부분을 차지하고, 별다른 내용이 있는 글은 아니었습니다. 문서에 나온대로 잘 흘러가고 있다는 정도...? 아, ExceptionTranslationFilter 를 조금 들여다보기는 했었습니다.

한 번 코드와 로그를 확인해봤으니, 다시 큰 그림을 보고싶어집니다. DelegatingFilterProxy -> FilterChainProxy -> SecurityFilterChain -> Filters 로 이어지는 것을 직접 그리기는 힘들어서, 문서에 있는 그림을 적당히 이어붙여봤습니다.

![Architecture 다이어그램 이어붙인 그림](/assets/img/2023-07-09-study-spring-security-5-check-logs-and-code/05-unified.png)

흐름 상 다음은 Start Authentication 부분으로 넘어가야겠습니다. 문서도 Authentication 이 다음 주제이기도 합니다.
