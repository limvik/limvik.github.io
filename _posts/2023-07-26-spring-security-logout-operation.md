---
layout: post
title: Spring Security 로그아웃(Logout) 하기
categories:
- Framework
- Spring
- Spring Security
tags:
- Spring
- Spring Security
date: 2023-07-26 21:03 +0900
---
## Intro

Spring Security 를 차근차근 하나씩 보고싶지만, 구현하기도 해야하니 Logout 절차 문서를 간단하게 살펴봤습니다.

구현을 한지 좀 됐지만, 감에 의한 구현으로 뭔가 찝찝해서 문서를 다시 살펴봤습니다.

## 요약

HttpServletRequest.logout() 메서드 사용하기

## 기존 구현

기본적으로 지원하는 경로인 POST /logout 를 통해 요청을 보내는게 적합한 경우도 있지만, ajax를 이용해서 페이지 내 UI 일부만 로그아웃 상태로 변경하기에는 html 응답까지 와버려 적합하지 않았습니다. 그래서 logout 요청을 별도로 보내기 위한 구현을 했습니다.

기존 구현은 SecurityContextHolder를 살펴본 후([작성 글 링크](https://limvik.github.io/posts/study-spring-security-6-securitycontextholder/))에 SecurityContext 를 비워버리면 되겠다는 생각을 했고, 실제로는 context 를 clear 한 후 session을 invalidate 했습니다.

```java
Optional.of(SecurityContextHolder.getContext()).ifPresentOrElse(
        context -> {
            SecurityContextHolder.clearContext();
            request.getSession(false).invalidate();
            },
        () -> context가 null 인 경우 처리 코드);
// 이외에는 null 체크와 응답 코드 지정 등 부수적인 코드입니다.
```

session을 invalidate 해주어야 하는 이유는 SecurityContext 에는 인증을 위해 사용자가 입력한 정보와 인증과 관련된 정보가 담겨있을 뿐, 인증 후 이용되는 정보는 다른 객체(나중에 Spring Security 진도 나가면 추가 예정)에 담겨 session 에 저장되기 때문입니다.

그런데 session은 null 체크 안했었네요.

### logout 메서드 찾아보기

그런데 로그아웃을 설마 구현안해놨겠나 싶어서 먼저 예전에 샀던 백엔드 책을보니, 아래와 같은 방식으로 로그아웃 하는게 보입니다.

```java
new SecurityContextLogoutHandler().logout(request, response, SecurityContextHolder.getContext().getAuthentication());
```

logout 메서드를 살펴보면 아래와 같습니다. Javadoc을 보면 response와 authentication은 안쓴다고 하니 null을 주고 로그아웃을 해보면 이상 없이 동작합니다. authentication은 심지어 메서드 내에 전혀 사용되고 있지 않죠.

```java
/**  
 * Requires the request to be passed in.
 * @param request from which to obtain a HTTP session (cannot be null)  
 * @param response not used (can be <code>null</code>)  
 * @param authentication not used (can be <code>null</code>)  
 */
@Override
public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
	Assert.notNull(request, "HttpServletRequest required");
	if (this.invalidateHttpSession) {
		HttpSession session = request.getSession(false);
		if (session != null) {
			session.invalidate();
			if (this.logger.isDebugEnabled()) {
				this.logger.debug(LogMessage.format("Invalidated session %s", session.getId()));
			}
		}
	}
	SecurityContext context = this.securityContextHolderStrategy.getContext();
	this.securityContextHolderStrategy.clearContext();
	if (this.clearAuthentication) {
		context.setAuthentication(null);
	}
	SecurityContext emptyContext = this.securityContextHolderStrategy.createEmptyContext();
	this.securityContextRepository.saveContext(emptyContext, request, response);
}
```

크게 보면 session invalidate 하는 부분, SecurityContext를 비우고 authentication에는 설정에 따라 null을 대입하는 부분, 빈 SecurityContext 를 만들어서 다시 또 저장하는 부분으로 나뉩니다.

SecurityContextHolder는 SecurityContextHolderStrategy 에 static 메서드를 위임한다고 하는 것을 살펴봤었기 때문에([링크](https://limvik.github.io/posts/study-spring-security-6-securitycontextholder/#:~:text=This%20class%20provides%20a%20series%20of%20static%20methods%20that%20delegate%20to%20an%20instance%20of%20SecurityContextHolderStrategy.)), SecurityContextHolderStrategy 에서 clearContext() 하는 거나 SecurityContextHolder 에서 clearContext() 하는 거나 동일하다는 것을 알 수 있습니다.

마지막 saveContext 메서드 내부를 살펴봤을 때, 버려진 코드인 느낌도 있어서 추후에 시간이 된다면 따로 다뤄보겠습니다.

### 근데 안됨

이것만 해도 session은 invalidate 돼서 UI가 변하니, 로그아웃이 되는 것으로 보입니다.

그런데 로그인 유지 기능, Spring Security 명칭으로는 `remember-me` 기능을 이용할 때는 로그아웃이 안되는 것을 확인할 수 있습니다.

로그아웃을 누를 때 마다 Session ID를 저장해놓은 쿠키 정보는 계속 갱신되는데, `remember-me` 쿠키가 계속 유지돼서 로그아웃이 안됩니다.

![Cookie 표시 화면](/assets/img/2023-07-26-spring-security-logout-operation/01-remember-me-cookie.png)

아마 책 예제에는 로그인 유지 기능이 없어서 그랬던 것 같습니다.

구글 검색 결과 1페이지에 있는 블로그만 간단하게 살펴봤는데, 딱히 눈에 들어오는게 없어서 공식 문서를 살펴봤습니다.

## Handling Logouts

공식 문서 Logout([링크](https://docs.spring.io/spring-security/reference/servlet/authentication/logout.html))를 보면, Spring Security 기본설정으로 로그아웃 기능을 이용하면 GET 또는 POST 방식으로 /logout 경로에 요청 시 로그아웃 절차를 수행한다고 하며, GET /logout 으로 접근하면 로그아웃 확인 페이지를 보여주는데, 사용할 필요 없다고 합니다.

![기본 로그아웃 화면](/assets/img/2023-07-26-spring-security-logout-operation/02-default-logout-template.png)

현실적으로도 직접 만든 페이지를 사용하지 기본 로그아웃 페이지를 사용하지는 않겠죠. Log Out 버튼을 누르면 POST 방식으로 /logout 경로에 HTTP 요청을 보냅니다.

![wireshark 로 본 POST /logout](/assets/img/2023-07-26-spring-security-logout-operation/03-default-logout-button-click.png)

### POST /logout 요청 시 기본 동작

문서에 이어서 나오는 내용은 POST /logout 요청 시 기본 동작에 대한 내용입니다.

`SecurityContextLogoutHandler` 를 이용한 과정이 많은 지분을 차지하기는 하지만, 추가적인 절차가 필요한 것을 확인할 수 있습니다.

If you request  `POST /logout`, then it will perform the following default operations using a series of  [`LogoutHandler`](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/web/authentication/logout/LogoutHandler.html)s:

-   Invalidate the HTTP session ([`SecurityContextLogoutHandler`](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/web/authentication/logout/SecurityContextLogoutHandler.html))
    
-   Clear the  [`SecurityContextHolderStrategy`](https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html#use-securitycontextholderstrategy)  ([`SecurityContextLogoutHandler`](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/web/authentication/logout/SecurityContextLogoutHandler.html))
    
-   Clear the  [`SecurityContextRepository`](https://docs.spring.io/spring-security/reference/servlet/authentication/persistence.html#securitycontextrepository)  ([`SecurityContextLogoutHandler`](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/web/authentication/logout/SecurityContextLogoutHandler.html))
    
-   Clean up any  [RememberMe authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/rememberme.html)  (`TokenRememberMeServices`  /  `PersistentTokenRememberMeServices`)
    
-   Clear out any saved  [CSRF token](https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html)  ([`CsrfLogoutHandler`](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/web/csrf/CsrfLogoutHandler.html))
    
-   [Fire](https://docs.spring.io/spring-security/reference/servlet/authentication/events.html)  a  `LogoutSuccessEvent`  ([`LogoutSuccessEventPublishingLogoutHandler`](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/web/authentication/logout/LogoutSuccessEventPublishingLogoutHandler.html))
    

Once completed, then it will exercise its default  [`LogoutSuccessHandler`](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/web/authentication/logout/LogoutSuccessHandler.html)  which redirects to  `/login?logout`.

그럼 이 절차도 알아서 호출해줄 메서드는 없을까 하면서 문서를 가장 밑으로 내려보니 다른 메서드가 보였습니다.

## HttpServletRequest.logout()

문서([링크](https://docs.spring.io/spring-security/reference/servlet/integrations/servlet-api.html#servletapi-logout)) 내용을 살펴 보면, 현재 사용자를 로그아웃하는 데 사용할 수 있는 메서드이고, 앞서 봤던 절차들을 수행하기는 하는데 개발자가 지정한 Spring Security configuration 에 따라 다르다고 합니다.

>You can use the  [`HttpServletRequest.logout()`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#logout%28%29)  method to log out the current user.
>
>Typically, this means that the  `SecurityContextHolder`  is cleared out, the  `HttpSession`  is invalidated, any “Remember Me” authentication is cleaned up, and so on. However, the configured  `LogoutHandler`  implementations vary, depending on your Spring Security configuration. Note that, after  `HttpServletRequest.logout()`  has been invoked, you are still in charge of writing out a response. Typically, this would involve a redirect to the welcome page.

wecome page 로 redirect 하는 등의 처리는 알아서 해줘야 한다고 하는데, 딱 제가 원하던 겁니다. 그리고 SecurityContextLogoutHandler 의 logout 메서드 뿐만 아니라 다른 정보들도 처리해준다는 것을 알 수 있습니다.

### 코드 살펴보기

그래도 궁금하니까 코드를 살펴봅니다.

HttpServlet3RequestFactory.java ([github 링크](https://github.com/spring-projects/spring-security/blob/main/web/src/main/java/org/springframework/security/web/servletapi/HttpServlet3RequestFactory.java#L269-L289))

```java
private class Servlet3SecurityContextHolderAwareRequestWrapper extends SecurityContextHolderAwareRequestWrapper {
    // 다른 메서드 생략
    @Override  
    public void logout() throws ServletException {  
        List<LogoutHandler> handlers = HttpServlet3RequestFactory.this.logoutHandlers;  
        if (CollectionUtils.isEmpty(handlers)) {  
           HttpServlet3RequestFactory.this.logger  
                   .debug("logoutHandlers is null, so allowing original HttpServletRequest to handle logout");  
           super.logout();  
           return;  
        }  
        Authentication authentication = HttpServlet3RequestFactory.this.securityContextHolderStrategy.getContext()  
              .getAuthentication();  
        for (LogoutHandler handler : handlers) {  
           handler.logout(this, this.response, authentication);  
        }  
    }
}
```

코드를 살펴보면 Logout handler 가 없으면 상위 logout 메서드를 호출하고, 있는 경우에는 모든 Logout handlers를 가져와서 각 handler 의 logout 메서드를 호출합니다.

디버거를 이용해서 살펴보면, 저는 별달리 Logout handler 를 custom 해서 추가하거나 한게 없기 때문에, 앞서 봤던 POST /logout 요청 시의 기본 동작에서 봤던 handlers를 볼 수 있습니다. 단독으로 직접 호출했던 SecurityContxtLogoutHandler가 눈에 들어옵니다.

![디버거를 이용해서 본 기본 Logout handlers](/assets/img/2023-07-26-spring-security-logout-operation/04-servlet-logout.png)

그리고 제 코드는 간단하게 수정할 수 있습니다.

```java
public 반환자료형 logout(HttpServletRequest request) {
    try {
        request.logout();
    } catch (ServletException e) {
        return 정상 처리되지 않은 경우 응답;  
    }
    return 정상 처리된 경우 응답;
}
```

## Outro

Spring Security 는 문서가 잘 돼있어서, 문서를 보는게 좋은 것 같습니다. 속도가 너무 느린게 단점이긴 하지만...

다음에는 logout을 어떤 url 경로로 작성할 것인지 다뤄보겠습니다.

애초에 REST API는 아니기 때문에, 큰 의미가 있을지는 모르겠지만, REST API best pratice 를 적용해볼 수 있을만한 것은 최대한 적용해보려고 하고 있습니다.

REST API의 best practice 중에 자원(resource)의 이름을 동사가 아닌 명사로 하라는 것이 있는데, logout 은 동사라 혼란에 빠지게 됩니다.

to be continued...
