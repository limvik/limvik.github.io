---
layout: post
title: Spring Security (2) 흐름 살펴보기 - Servlet Filter
categories:
- Framework
- Spring
- Sprirng Security
tags:
- Spring
- Spring Security
- Servlet Filter
date: 2023-07-06 16:44 +0900
---
## Intro

[이전 글](https://limvik.github.io/posts/study-spring-security-1-just-do-it/)에서 간단하게 맛보기 겸 Spring Security를 사용해봤습니다.

그리고 저는 그 사이에 팀 프로젝트를 하면서 여기저기 널려있는 예제와 감으로 적당히 ID/Password 로그인과 구글, 카카오, 네이버 로그인이 가능은 하도록 만들어봤습니다.

하면서 가장 힘들었던 것은 Spring Security 의 흐름을 잘 모르다보니 '혹시 여기가 호출되는건가?' 하면서 감으로 실험해보고 기능을 구현할 수 밖에 없었습니다.

그래서 큰 흐름을 먼저 살펴보려고 합니다.

## Spring Security 문서 살펴보기

[RTFM](https://en.wikipedia.org/wiki/RTFM)(Read The Fucking Manual) 이라는 말이 있죠. 그래서 욕먹기 전에 문서부터 살펴보겠습다.

### Overview

Spring Security 공식 문서([링크](https://docs.spring.io/spring-security/reference/index.html))의 Overview를 보면 첫 번째 문단에는 아래와 같은 내용이 있습니다.

>Spring Security is a framework that provides authentication, authorization, and protection against common attacks. With first class support for securing both imperative and reactive applications, it is the de-facto standard for securing Spring-based applications.

간단하게 번역기를 돌려보면 아래와 같습니다.

>Spring Security는 인증, 인가, 일반적인 공격에 대한 보호 기능을 제공하는 프레임워크입니다. 명령형 애플리케이션과 반응형 애플리케이션 모두에 대한 보안을 최고 수준으로 지원하는 이 프레임워크는 Spring 기반 애플리케이션 보안을 위한 사실상의 표준입니다.

인증, 인가, 일반적인 공격에 대한 보호 기능은 세부적인 내용이 될테고, 프레임워크이므로 Spring을 처음 배울 때 처럼 흐름을 이해해야겠습니다.

명령형 애플리케이션과 반응형 애플리케이션을 문서에서 굳이 언급하는 이유는 이전 글에서 사용했던 것과 같이 Servlet을 이용한 방식과 WebFlux 등을 이용하여 반응형 애플리케이션을 만들 때 Spring Security 를 사용하는 방식이 다르기 때문입니다. 아직 WebFlux는 본적조차 없기 때문에 반응형 애플리케이션을 만드려면 또 따로 공부해야된다는 점을 알아둬야겠습니다.

문서를 보면 Servlet Application과 Reactive Application 메뉴로 별도로 다루고 있는 것을 볼 수 있습니다.

![Spring Security 공식 문서의 왼쪽 메뉴창](/assets/img/2023-07-06-study-spring-security-2-servlet-filter/01-spring-security-doc-menu.png)

그럼 지금 당장 사용하고 있고, 잘 해야 할 Servlet Application에 대해 살펴보겠습니다.

### Servlet Application

알고 계시겠지만 이전 글에서 당연하다는 듯 사용했던 방식이 Servlet Application 입니다. 그리고 해당 문서에 나온 내용([링크](https://docs.spring.io/spring-security/reference/servlet/index.html))을 보면 아래와 같습니다.

>Spring Security는 표준 Servlet Filter를 사용하여 Servlet Container와 통합됩니다. 즉, Servlet Container에서 실행되는 모든 애플리케이션에서 작동합니다. 좀 더 구체적으로 말하면, Spring Security를 활용하기 위해 Servlet 기반 애플리케이션에서 Spring을 사용할 필요는 없습니다.

Servlet 기반으로 작동하니까 굳이 Spring을 사용하지 않아도 Spring Security를 사용할 수 있다고 합니다. (아직 초보라 굳이 Servlet 만으로 개발할 일이 있는지는 잘 모르겠습니다.)

그리고 Servlet Filter 와 Servlet Container에 대해 언급하는데, 글 보다는 그림이 이해하기 쉽겠죠.

![Servlet Filter Diagram](/assets/img/2023-07-06-study-spring-security-2-servlet-filter/02-servlet-filter-diagram.jpg)

출처: [http://j2eetutorials.50webs.com/filters.html](http://j2eetutorials.50webs.com/filters.html)

사용자의 요청이 먼저 Filter를 거치게 되고, Servlet 에 도달한 후에 다시 응답에 대한 Filter 를 거쳐 사용자에게 도달하게 된다는 것을 볼 수 있습니다.

Spring 을 사용해봐서 아래 그림이 더 친숙하게 느껴집니다. Front Controller 인 Dispatcher Servlet 보다 먼저 Filter 가 위치하는 것을 볼 수 있습니다.

![Spring Filter Chain Diagram](/assets/img/2023-07-06-study-spring-security-2-servlet-filter/03-spring-filter-chain.png)

출처: [https://www.knowledgefactory.net](https://www.knowledgefactory.net/2022/03/how-to-get-request-url-pattern-template-that-has-been-hit.html)

Spring MVC를 배우기 시작할 때는 아래 같은 그림을 더 많이 보게되는데, Spring Security 는 이 앞단에 위치하면서 Servlet Filter 로서 보안과 관련된 처리를 하게 됩니다.

![Spring MVC Diagram](/assets/img/2023-07-06-study-spring-security-2-servlet-filter/04-spring-mvc-diagram.png)

출처: [https://terasolunaorg.github.io/](guideline/5.0.1.RELEASE/en/Overview/SpringMVCOverview.html#overview-of-spring-mvc-processing-sequence)

Servlet Application에 적용하는 Spring Security는 Servlet Filter가 Spring Security 의 가장 큰 덩어리라고 생각해볼 수 있다고 판단돼서 Servlet Filter를 한 번 훑어보고 가겠습니다.

Spring 을 배우기 전에 Servlet을 배우면서 Servlet Filter를 스쳐지나가듯 본적이 있기는 합니다. HTTP 요청에 대해 공통적인 처리를 적용하기 위해 사용하였었는데, 복습할 겸 GPT-3.5가 건네준 간단한 예제를 살펴보겠습니다.

### Servlet Filter 예제

Filter를 직접 만드는게 아니라면 세부적인 것보다는 큰 흐름이 중요하다고 생각돼서 간단한 설명으로 대체합니다. `@WebFilter` 에 모든 경로(`"/*"`)에 대해 Filter를 적용할 것을 선언하였고, doFilter 메서드 앞, 뒤로 전처리 로직과 후처리 로직이 있어 앞서 다이어그램에서 봤던대로 흘러가고 있음을 볼 수 있습니다.

```java
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter("/*")
public class LoggingFilter implements Filter {
    @Override
    public void init(FilterConfig config) throws ServletException {
        // 필터 초기화 로직
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        // 요청 전처리 로직
        System.out.println("요청이 도착했습니다. IP 주소: " + request.getRemoteAddr());

        // 다음 필터 또는 서블릿 호출
        chain.doFilter(request, response);

        // 응답 후처리 로직
        System.out.println("응답이 전송되었습니다.");
    }

    @Override
    public void destroy() {
        // 필터 해제 로직
    }
}

```

그리고 이 filter를 등록하기 위해서 web.xml 파일을 작성하거나, Servlet 3.0 부터는 리스너(Listener)를 이용할 수 있습니다. xml 파일은 길어서 리스너를 이용한 코드를 보여드리겠습니다.

```java
import javax.servlet.*;
import javax.servlet.annotation.WebListener;

@WebListener
public class FilterRegistrationListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        // 필터 등록
        FilterRegistration.Dynamic filter = sce.getServletContext().addFilter("LoggingFilter", LoggingFilter.class);
        // 필터 매핑
        filter.addMappingForUrlPatterns(null, true, "/*");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        // 필터 해제
        // (일반적으로는 필요하지 않음)
    }
}
```

다이어그램으로 보자면 아래와 같습니다.
![Filter and Listener Diagram](/assets/img/2023-07-06-study-spring-security-2-servlet-filter/05-servlet-filter-and-listener.gif)

출처: [https://docs.oracle.com/](https://docs.oracle.com/cd/B14099_19/web.1012/b14017/filters.htm#i1000654)

그러면 아래와 같이 Servlet 은 앞서 filter 에서 doFilter 메서드에 인수로 사용했던 request와 response 를 받게 됩니다.

```java
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 서블릿에서 요청 처리 로직
        response.getWriter().println("Hello, world!");
    }
}
```

코드를 실행해보진 않았지만 기억과 비슷하기도 하고, 흐름을 보려고 했던거니 코드 검증은 하지 않고 넘어가겠습니다. 

Filter 이야기가 조금 길어지기는 했지만 Spring Security 가 어떤 방식으로 Servlet Filter 를 이용해서 HTTP 요청을 가로채고, 보안 기능을 적용하겠다는 것을 어렴풋이나마 알 수 있습니다.

이전 글에서 봤던 것 중에 간단하게나마 흔적을 보자면 SecurityProperties.java([Github 링크](https://github.com/spring-projects/spring-boot/blob/ce8253ea951eec2e857f3a9d9f6c3135029f91c8/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/SecurityProperties.java))에서 Filter의 우선순위를 정하는 코드들을 살펴볼 수 있습니다.

```java
import org.springframework.boot.web.servlet.filter.OrderedFilter;

/**
 * Default order of Spring Security's Filter in the servlet container (i.e. amongst
 * other filters registered with the container). There is no connection between this
 * and the {@code @Order} on a {@code SecurityFilterChain}.
 */
public static final int DEFAULT_FILTER_ORDER = OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER - 100;
```

### Architecture

Servlet Application 의 Architecture 문서([링크](https://docs.spring.io/spring-security/reference/servlet/architecture.html))를 눌러보면 바로 아래와 같은 그림을 볼 수 있습니다.

![Spring Security 공식 문서에 나와있는 Servlet Filter Chain 그림](/assets/img/2023-07-06-study-spring-security-2-servlet-filter/06-filterchain.png)

그리고 글을 쓰면서 몇 가지 눈길이 가는 문장을 살펴보면 아래와 같습니다.

>Spring MVC 애플리케이션에서 Servlet은 DispatcherServlet의 인스턴스입니다. `하나`의 Servlet은 최대 `하나`의 HttpServletRequest 및 HttpServletResponse를 처리할 수 있습니다. 그러나 `둘 이상의 Filter를 사용`할 수 있습니다.
>
>Filter는 오직 다운스트림 Filter 인스턴스와 Servlet에만 영향을 미치므로 각 Filter가 호출되는 `순서`가 매우 중요합니다.

Spring Security 의 가장 큰 껍데기는 Filter고, 하나의 Filter가 아닌 여러 개의 Filter 로 구성되어 있으며 Filter가 적용되는 순서가 중요하다는 것을 알 수 있습니다. Architecture 문서 조금만 내려보면 바로 조금 더 자세히 알 수 있습니다.

## Outro

이번에는 이렇게 간단하게 Filter 를 살펴본 것으로 마무리 하려고합니다. Filter 라는 큰 덩어리를 살펴봤으니 다음에는 문서의 Architecture 와 관련된 내용을 훑어보면서 Spring Security에 어떤 종류의 Filter 가 있는지 살펴보겠습니다.
