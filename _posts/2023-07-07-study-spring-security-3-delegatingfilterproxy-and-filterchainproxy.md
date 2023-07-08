---
layout: post
title: Spring Security (3) DelegatingFilterProxy, FilterChainProxy
categories:
- Framework
- Spring
- Spring Security
tags:
- Spring
- Spring Security
- Servlet Filter
date: 2023-07-07 16:41 +0900
---
## Intro

Servlet Filter에 대해 살펴봤던 [이전 글](https://limvik.github.io/posts/study-spring-security-2-servlet-filter/)에 이어서 Spring Security 를 살펴보겠습니다.

## DelegatingFilterProxy

이전 글에서 보던 Architecture 문서([링크](https://docs.spring.io/spring-security/reference/servlet/architecture.html))를 이어서 보면, 가장 먼저 나오는 것은 DelegatingFilterProxy 입니다. 문서에서 DelegatingFilterProxy를 아래와 같이 다이어그램으로 표현하고 있습니다.

![DelegatingFilterProxy가 포함된 다이어그램](/assets/img/2023-07-07-study-spring-security-3-delegatingfilterproxy-filterchainproxy/01-delegatingfilterproxy-diagram.png)

여기서 주목해야 할 점은 DelegatingFilterProxy가 Bean Filter를 포함하고 있다는 것입니다.

### Bean Filter 를 바로 사용하지 않는 이유

Bean Filter가 바로 나타나지 않고 한 번 감싼 이유는 Servlet Container 가 Spring의 Bean을 인식할 수 없기 때문입니다. 그래서 DelegatingFilterProxy가 Servlet Container에 Filter로 등록된 후 클래스 이름에 맞게 할 일을 위임하는데, 일을 위임받는 대상이 Bean Filter가 됩니다.

그리고 DelegatingFilterProxy 문서([링크](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/DelegatingFilterProxy.html))를 보면 jakarta.servlet.Filter 인터페이스를 상속받고 있는 것을 알 수 있습니다.

![DelegatingFilterProxy 문서 화면](/assets/img/2023-07-07-study-spring-security-3-delegatingfilterproxy-filterchainproxy/02-DelegatingFilterProxy-doc.png)

그래서 DelegatingFilterProxy를 ServletContainer에 등록하여 요청을 가로챈 후, Architecture 문서에서 의사 코드(Pseudo Code)로 표현한 것과 같이 Spring의 Bean이 등록되어있는 ApplicationContext 에서 Bean Filter 를 가져와서 작업을 수행하게 됩니다.

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
	Filter delegate = getFilterBean(someBeanName); 
	delegate.doFilter(request, response); 
}
```

그리고 문서에서는 이러한 방식의 장점을 설명하는데 그 내용은 아래와 같습니다.

>DelegatingFilterProxy의 또 다른 장점은 `Filter` bean 인스턴스 조회를 지연시킬 수 있다는 점입니다. 이는 컨테이너가 시작되기 `전`에 `Filter` 인스턴스를 등록해야 하기 때문에 중요합니다. 그러나 Spring은 일반적으로 ContextLoaderListener를 사용하여 Spring Bean을 로드하는데, 이 작업은 Filter 인스턴스를 등록한 `후`에야 수행됩니다.

의사 코드가 아닌 실제 DelegatingFilterProxy 코드를 살펴보면 아래와 같습니다.

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {

    // Lazily initialize the delegate if necessary.
    Filter delegateToUse = this.delegate;
    if (delegateToUse == null) {
        synchronized (this.delegateMonitor) {
            delegateToUse = this.delegate;
            if (delegateToUse == null) {
                WebApplicationContext wac = findWebApplicationContext();
                if (wac == null) {
                    throw new IllegalStateException("No WebApplicationContext found: " +
                        "no ContextLoaderListener or DispatcherServlet registered?");
                }
                delegateToUse = initDelegate(wac);
            }
            this.delegate = delegateToUse;
        }
    }

    // Let the delegate perform the actual doFilter operation.
    invokeDelegate(delegateToUse, request, response, filterChain);
}

protected void invokeDelegate(Filter delegate, ServletRequest request, ServletResponse response, FilterChain filterChain)  
throws ServletException, IOException {  
  
    delegate.doFilter(request, response, filterChain);  
}
```

의사 코드보다 조금 길어지기는 하지만 의사코드에 작성된대로 기능을 수행하는 것을 볼 수 있습니다.

### 정리

다시 한 번 정리하자면 Filter 는 Servlet Container 에 등록이 되는데, Servlet Container 는 Spring Bean을 인식하지 못하기 때문에 DelegatingFilterProxy 를 Servlet Container 에 등록하고 요청을 가로챈 후 Bean Filter를 불러와 작업을 수행합니다.

그런데 조금 의문이 생깁니다. DelegatingFilterProxy 는 Spring에 포함된 클래스입니다.

이게 의문이 생기는 이유는 이전 글에서 Servlet Application 메뉴의 Overview([링크](https://docs.spring.io/spring-security/reference/servlet/index.html))에 Spring 을 사용하지 않아도 Spring Security를 사용할 수 있다고 적혀있던 것을 봤었기 때문입니다. 그래서 '그럼 Spring Security 만 단독으로 사용할 수는 없는거 아닌가?' 하는 의문이 듭니다.

별 쓸데 없는 궁금증인거 같기는 하지만, 간단하게 살펴보고 가겠습니다.

## Servlet Application 에 Spring 없이 Spring Security 를 사용할 수 있을까?

### 문서 살펴보기

DelegatingFilterProxy 문서를 보면 설명의 마지막 줄에 이런 내용이 있습니다.

>This class was originally inspired by Spring Security's `FilterToBeanProxy` class, written by Ben Alex.

Spring Security 의 FilterToBeanProxy 에 영감을 받아 만들었다고 합니다. 그래서 Spring Security 프로젝트에서 FilterToBeanProxy 문서([링크](https://docs.spring.io/spring-security/site/docs/2.0.7.RELEASE/apidocs/org/springframework/security/util/FilterToBeanProxy.html))를 찾아봤더니 Deprecated 되었다고 다시 Spring Framework의 DelegatingFilterProxy를 사용하라고 합니다.

>**Deprecated.**  _use DelegatingFilterProxy instead_

사람 놀리는 느낌...?

### Spring Security 에서 Proxy 붙은 클래스 찾아보기

하지만 문서에서 괜히 Spring Security만 사용할 수 있다고 표현한거는 아닐테니 Spring Security 에서 proxy를 찾아봅니다.

그래서 등장하는게 FilterChainProxy 입니다. DelegatingFilterProxy 와 마찬가지로  jakarta.servlet.Filter 을 상속받고 있고, 또 마찬가지로 GenericFilterBean 을 상속받고 있습니다.

![FilterChainProxy 문서 화면](/assets/img/2023-07-07-study-spring-security-3-delegatingfilterproxy-filterchainproxy/03-FilterChainProxy-doc.png)

그리고 혹시나해서 Spring Security의 dependency 파일([Github 링크](https://github.com/spring-projects/spring-security/blob/main/core/spring-security-core.gradle))을 찾아보니, Spring Bean과 Spring DI container 를 사용하기 위한 spring-core, spring-beans, spring-context, spring-aop 가 보였습니다.

```groovy
apply plugin: 'io.spring.convention.spring-module'

dependencies {
    management platform(project(":spring-security-dependencies"))
    api project(':spring-security-crypto')
    api 'org.springframework:spring-aop'
    api 'org.springframework:spring-beans'
    api 'org.springframework:spring-context'
    api 'org.springframework:spring-core'
    api 'org.springframework:spring-expression'
...
```

api 로 선언한 것을 저는 처음 봐서 GPT-3.5 설명을 같이 추가합니다.

>
>Gradle에서 `api`로 선언한 의존성은 다른 모듈 또는 프로젝트가 이 모듈을 사용할 수 있도록 공개된 의존성을 의미합니다.
>
>`api` 구성은 Gradle에서 제공하는 의존성 구성 중 하나입니다. `api`로 의존성을 선언하면 해당 모듈의 API(공개된 인터페이스 및 클래스)가 의존성 그래프를 통해 다른 모듈에 노출됩니다. 다른 모듈은 이 API를 사용할 수 있으며, 해당 모듈에 대한 의존성이 변경되면 다시 빌드하고 업데이트할 필요가 있습니다.
>
>따라서, 위의 Gradle 스크립트에서 `api`로 선언된 의존성은 `spring-security-crypto`, `spring-aop`, `spring-beans`, `spring-context`, `spring-core` 등이 다른 모듈에서 사용할 수 있도록 공개된 의존성이라고 할 수 있습니다.

### 정리

이걸 Spring을 안쓴다고 할 수 있는지는 좀 의문이 들기는 하지만, Spring Security 만 단독으로 사용할 때는 FilterChainProxy 를 DelegatingFilterProxy 대신 사용할 수 있지 않을까? 생각만 해봅니다. 굳이 그럴 일은 없을 것 같아서 이렇게 간단하게만 살펴보고 넘어가겠습니다.

다음에 마주할 Bean Filter는 바로 방금 언급된 FilterChainProxy 입니다.

## FilterChainProxy

FilterChainProxy 의 문서를 보면 아래와 같이 DelegatingFilterProxy 을 통해서 Servlet Container Filter Chain에 연결됨을 명시하고 있습니다.

>The `FilterChainProxy` is linked into the servlet container filter chain by adding a standard Spring `DelegatingFilterProxy` declaration in the application `web.xml` file.

Architecture 문서도 DelegatingFilterProxy에 이어서 바로 FilterChainProxy 를 설명하고 있습니다.

![FilterChainProxy가 포함된 다이어그램](/assets/img/2023-07-07-study-spring-security-3-delegatingfilterproxy-filterchainproxy/04-filterchainproxy-diagram.png)

그리고 문서에서는 간단하게 아래와 같이 설명하고 있습니다.

>Spring Security의 Servlet 지원은 FilterChainProxy에 포함되어 있습니다. FilterChainProxy는 Spring Security에서 제공하는 특별한 `Filter`로, SecurityFilterChain을 통해 많은 `Filter` 인스턴스에 위임할 수 있습니다. FilterChainProxy는 Bean이기 때문에 일반적으로 DelegatingFilterProxy로 래핑됩니다.

결국 FilterChainProxy도 위임하기 때문에 특별히 설명할 것은 없어 보이긴 합니다. FilterChainProxy 이후에 위임받는 SecurityFilterChain 들의 역할을 각각 다루는게 맞겠죠.

FilterChainProxy 가 Override 하는 doFilter() 메서드는 좀 길기도 하고 아직은 눈에 잘 들어오는게 없어서 [Github 링크](https://github.com/spring-projects/spring-security/blob/01c8a2233655a7e8b5b3aa1fec4032d0d36ace15/web/src/main/java/org/springframework/security/web/FilterChainProxy.java#L182)로 대체하겠습니다.

## 정리

이번에는 Proxy 까지만 살펴본 것으로 정리해보려고 합니다.

Servlet Container 메커니즘 상 Spring Bean을 인식할 수 없기 때문에, Spring 에서는 DelegatingFilterProxy를 Servlet Container에 Filter로 등록한 후 DelegatingFilterProxy를 통해 Bean 으로 등록된 Filter 를 불러와 호출합니다. 이 과정에서 Spring Security 전용 Filter 인 FilterChainProxy 가 호출되며, FilterChainProxy는 다시 SecurityFilterChain을 호출하여 로직을 수행합니다.

앞서 봤듯 DelegatingFilterProxy가 Spring Security 의 FilterToBeanProxy 에 영감을 받아 나중에 만들어진 것으로 봐서는 Spring에서 처음에는 Filter를 사용하지 않다가 Filter 사용 수요가 생겨서, Filter 를 관리하기 위한 Spring 전용 Filter를 만들게 되지 않았나 조심스레 추측해 봅니다.

## Outro

조금씩 Spring Security에 다가가고 있는 느낌이 듭니다. 근데 또 한층 더 까보면 어마어마한 양으로 당황하게 만들겠죠. 조급해 하지 않고 조금씩이라도 알아가는게 좋겠습니다.