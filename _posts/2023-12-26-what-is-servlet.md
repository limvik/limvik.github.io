---
layout: post
title: 서블릿(Servlet) 이란?
categories:
- Java
- Servlet
tags:
- Java
- Servlet
- CGI
date: 2023-12-26 16:38 +0900
---
## Intro

Java Servlet, 이제는 Jakarta Servlet 이라 불러야겠죠. 여튼, Servlet이 무엇인가? 했을 때 뭐라 해야할지 잘 안떠올라서 찾아본 내용들을 정리해봤습니다.

## Servlet 단어 뜯어보기

이전에 Java Applet 단어에 뒤에 붙은 `-let`이 무엇을 의미하는지 궁금해서 찾아봤을 때, `작은(small)` 을 뜻하는 의미였습니다. Applet은 그래서 작은 Java Application Code를 의미한다고 할 수 있습니다. (예전에 찾아뒀던 자료 출처가 사라졌는데, 다시 찾아도 잘 안나와서 일단 출처는 패스하겠습니다.)

이러한 의미를 Servlet에 갖다 붙여보면, 작은 Java Server Code를 의미한다고 할 수 있겠습니다.

## Servlet 정의 찾아보기

여기저기서 Servlet을 정의하고 있는데, 크게 두 가지 부류로 나눠볼 수 있었습니다. 하나는 `HTTP 처리에 한정`하는 경우, 다른 하나는 `프로토콜과 상관없이 요청과 응답을 처리`하는 기술로 넓게 정의하는 경우가 있었습니다.

### Servlet Interface

Servlet Interface에 작성된 Javadoc([링크](https://jakarta.ee/specifications/servlet/6.0/apidocs/jakarta.servlet/jakarta/servlet/servlet))에는 아래와 같은 문장이 있습니다.

> A servlet is a small Java program that runs within a Web server. Servlets receive and respond to requests from Web clients, usually across HTTP, the HyperText Transfer Protocol.
>
> Servlet은 Web server에서 실행되는 작은 Java program 입니다. Servlet은 보통 HTTP를 통해 Web clients의 요청을 수신하고 응답합니다.

여기서는 HTTP로 한정하지는 않지만, 주로 HTTP 요청을 처리한다고 언급합니다.

Servlet Interface를 구현한게 [HttpServlet](https://jakarta.ee/specifications/servlet/6.0/apidocs/jakarta.servlet/jakarta/servlet/http/httpservlet) 말고는 없기는 합니다.

#### within a Web server?

위 정의에서 개인적으로는 이 문장이 좀 이상하게 느껴집니다. Web server는 동적 content를 serving 하기는 하지만, servlet이 Web server 내부에서 실행된다는게 잘 받아들여지지 않습니다.

그래서 Java EE 5 시절 문서를 보니, Web server는 아무래도 `Web application server` 까지 포함한 의미로 사용되고 있는게 아닐까 생각됩니다.

![01.Java Web Application Request Handling](/assets/img/2023-12-26-what-is-servlet/01.java-web-application-request-handling.gif)  
출처: [The Java EE 5 Tutorial](https://docs.oracle.com/javaee/5/tutorial/doc/geysj.html)

### Java EE 5 Tutorial

옛날 문서 본김에 나와있는 정의도 살펴보겠습니다.

> Servlets are Java programming language classes that dynamically process requests and construct responses.
> 
> Servlets는 요청을 동적으로 처리하고 응답을 구성하는 Java 프로그래밍 언어 클래스입니다.

여기서는 `동적으로 요청을 처리한다는 점`을 언급한게 앞서 본 정의와는 조금 차이점이 있습니다.

[What Is a Servlet?](https://docs.oracle.com/javaee/5/tutorial/doc/bnafe.html) 섹션에 있는 내용도 살펴보면 아래와 같습니다.

> A **servlet** is a Java programming language class that is used to extend the capabilities of servers that host applications accessed by means of a request-response programming model. Although servlets can respond to any type of request, they are commonly used to extend the applications hosted by web servers. For such applications, Java Servlet technology defines HTTP-specific servlet classes.
> 
> Servlet 은 요청-응답 프로그래밍 모델을 통해 액세스하는 애플리케이션을 호스팅하는 서버의 기능을 확장하는 데 사용되는 Java 프로그래밍 언어 클래스입니다. Servlet 은 모든 유형의 요청에 응답할 수 있지만, 일반적으로 웹 서버에서 호스팅하는 애플리케이션을 확장하는 데 사용됩니다. 이러한 애플리케이션을 위해 Java Servlet 기술은 HTTP 전용 Servlet 클래스를 정의합니다.

다음으로는 최신 Spec 문서에 있는 정의를 살펴보겠습니다.

### Servlet 6.0 Specification

Spec 문서에 나와있는 정의([링크](https://jakarta.ee/specifications/servlet/6.0/jakarta-servlet-spec-6.0#what-is-a-servlet))를 보면 아래와 같습니다.

> A servlet is a Jakarta technology-based web component, managed by a container, that generates dynamic content.
>
> Servlet은 동적 content를 생성하는 container에 의해 관리되는, Jakarta 기술 기반의 web component 입니다.

Spec을 소개하는 웹사이트([링크](https://jakarta.ee/specifications/servlet/6.0/))에는 아래와 같은 정의도 있습니다.

> Jakarta Servlet defines a server-side API for handling HTTP requests and responses.
> 
> Jakarta Servlet은 HTTP 요청 및 응답을 처리하기 위한 서버측 API를 정의합니다.

Spec 문서는 동적 content를 생성하는 것에 집중했고, 웹사이트에서는 HTTP 요청 및 응답을 처리하는 것에 집중한 것으로 보입니다.

### 정리해 보기

위 정의들을 바탕으로 정리해보면 Servlet의 키워드는 `동적 content`, `server-side`, `요청과 응답`, `HTTP` 정도가 되겠습니다. HTTP가 특수한 경우라 하기에는 HTTP 말고는 다른 사용 사례를 찾아볼 수 없어 키워드에 포함시켰습니다.

키워드로 문장을 만들어보자면, `Servlet은 HTTP 요청을 받아 동적 content를 생성하여 응답하는 server-side 기술`이라고 할 수 있겠습니다.

HTTP 로 한정한 것에 공격이 들어온다면, Servlet 자체는 특정 프로토콜에 한정된 기술이 아니지만, 실질적으로 HTTP 외에는 사용되는 곳이 없다고하면 되겠습니다.

여기서 끝내기에는 뭔가 아쉬워서 Servlet 등장 배경을 살펴보겠습니다.

## Servlet 등장 배경

뇌피셜로는 클라이언트 측에서 사용되는 Applet([1995년 Release](https://en.wikipedia.org/wiki/Java_applet))이 성공하고, 서버 측에서 사용할 기술이 필요해서 Servlet([1996년 Release](https://en.wikipedia.org/wiki/Jakarta_Servlet))을 만들지 않았을까 생각이 됩니다. 하지만 뇌피셜일 뿐 나름 공식적인 등장 배경을 살펴보겠습니다.

### The Java EE 5 Tutorial

다시 구 버전의 문서로 돌아옵니다.

[Java Servlet Technology](https://docs.oracle.com/javaee/5/tutorial/doc/bnafd.html) 섹션을 보면, 등장 배경이 잘 나와있습니다.

> As soon as the web began to be used for delivering services, service providers recognized the need for dynamic content. Applets, one of the earliest attempts toward this goal, focused on using the client platform to deliver dynamic user experiences. At the same time, developers also investigated using the server platform for this purpose. Initially, Common Gateway Interface (CGI) scripts were the main technology used to generate dynamic content. Although widely used, CGI scripting technology has a number of shortcomings, including platform dependence and lack of scalability. To address these limitations, Java Servlet technology was created as a portable way to provide dynamic, user-oriented content.
> 
> 웹이 서비스 제공에 사용되기 시작하자마자 서비스 제공업체는 동적 content의 필요성을 인식했습니다. 이러한 목표를 향한 초기 시도 중 하나인 Applets는 클라이언트 플랫폼을 사용하여 동적 사용자 경험을 제공하는 데 중점을 두었습니다. 동시에 개발자들은 이러한 목적으로 서버 플랫폼을 사용하는 방안도 검토했습니다. 초기에는 동적 콘텐츠를 생성하는 데 주로 사용된 기술이 CGI(Common Gateway Interface) 스크립트였습니다. 널리 사용되기는 했지만 CGI 스크립팅 기술에는 플랫폼 종속성, 확장성 부족 등 여러 가지 단점이 있었습니다. 이러한 한계를 해결하기 위해 사용자 중심의 동적 content를 제공할 수 있는 portable 한 방법으로 Java Servlet 기술이 개발되었습니다.

요약하자면, 동적 content 를 제공하기 위해 사용하던 `CGI script 기술의 단점을 해결`하기 위해 Java Servlet을 개발한 것입니다.

CGI가 뭔지부터 봐야겠습니다.

### CGI

여기서 자세히 다룰 내용은 아니므로, Wikipedia에 있는 내용을 살펴보겠습니다.

#### CGI 등장 배경

> Initially, there were no standardized methods for data exchange between a browser, the HTTP server with which it was communicating and the scripts on the server that were expected to process the data and ultimately return a result to the browser. As a result, mutual incompatibilities existed between different HTTP server variants that undermined script  [portability](https://en.wikipedia.org/wiki/Software_portability "Software portability").
>
>Recognition of this problem led to the specification of how data exchange was to be carried out, resulting in the development of CGI. Web page-generating programs invoked by server software that adheres to the CGI specification are known as  _CGI scripts_, even though they may actually have been written in a non-scripting language, such as  [C](https://en.wikipedia.org/wiki/C_(programming_language) "C (programming language)").
>
> 초기에는 브라우저, 브라우저와 통신하는 HTTP 서버, 데이터를 처리하여 궁극적으로 브라우저에 결과를 반환할 것으로 예상되는 서버의 스크립트 간에 데이터를 교환하는 표준화된 방법이 없었습니다. 그 결과 서로 다른 HTTP 서버 변형 간에 상호 비호환성이 존재하여 스크립트 이식성이 약화되었습니다.  
>
> 이러한 문제점을 인식한 사람들은 데이터 교환 방식에 대한 규격을 정하기 시작했고, 그 결과 CGI가 개발되었습니다. CGI 사양을 준수하는 서버 소프트웨어에 의해 호출되는 웹 페이지 생성 프로그램은 실제로는 C와 같은 비스크립트 언어로 작성되었더라도 CGI 스크립트라고 알려져 있습니다.

C로 작성된 CGI Script 예제

```c
#include <stdio.h>

int main()
{
  printf("Content-type: text/html\n\n");
  printf("<html>\n");
  printf("<body>\n");
  printf("<h1>Hello there!</h1>\n");
  printf("</body>\n");
  printf("</html>\n");
  return 0;
}
```

출처: [howstuffworks - How CGI Scripting Works](https://computer.howstuffworks.com/cgi3.htm)

간단하게 이야기하자면, `Web 에서 동적 Content를 제공하기 위한 표준 사양`이라고 할 수 있습니다.

그럼 이 CGI라는 기술에 구체적으로 어떤 단점이 있었을까요?

#### CGI의 단점

CGI의 대표적인 단점은 Wikipedia의 Alternatives 섹션([링크](https://en.wikipedia.org/wiki/Common_Gateway_Interface#Alternatives))에 잘 나와있습니다.

> For each incoming HTTP request, a Web server creates a new CGI [process](https://en.wikipedia.org/wiki/Process_(computing) "Process (computing)") for handling it and destroys the CGI process after the HTTP request has been handled. Creating and destroying a process can be expensive: consume CPU time and memory resources than the actual work of generating the output of the process, especially when the CGI program still needs to be [interpreted](https://en.wikipedia.org/wiki/Interpret "Interpret") by a virtual machine. For a high number of HTTP requests, the resulting workload can quickly overwhelm the Web server.
>
> 웹서버는 들어오는 각 HTTP 요청에 대해 이를 처리하기 위한 `새 CGI 프로세스를 생성`하고 HTTP 요청이 처리된 후 `CGI 프로세스를 삭제`합니다. 프로세스를 생성하고 소멸하는 데는 비용이 많이 들 수 있습니다. 특히 가상 머신에서 CGI 프로그램을 해석해야 하는 경우 프로세스의 출력을 생성하는 실제 작업보다 CPU 시간과 메모리 리소스를 더 많이 소비합니다. HTTP 요청이 많은 경우, 이로 인한 워크로드가 웹 서버를 빠르게 압도할 수 있습니다.

`각 요청마다 프로세스를 생성하고 삭제`한다고 합니다. Tomcat 의 `Thread per Request` 모델도 자원 소모가 많다고 칭얼칭얼 하는데, 프로세스를 요청마다 생성하고 삭제한다니 어마어마한 자원소모가 있겠습니다.

### 정리

딱히 정리할 내용은 없는 관계로 다시 반복해보자면, `CGI의 단점을 해결하기 위해 Servlet이 개발`되었습니다. 대표적으로 Process per Request인 CGI의 비효율적인 자원 소모를 Thread per Request 방식으로 해결합니다.

CGI대비 성능도 좋은데다가 클라이언트에서는 Applet, 서버에서는 Servlet으로 하나의 언어를 이용하여 웹 개발을 할 수 있다는 장점도 있으니, 당시에 Java가 인기를 끌 수 밖에 없었을 것 같습니다.

## Outro

이렇게 정리하고보니, 흐릿흐릿 하던 Servlet의 정의가 그나마 명확해진 느낌이 듭니다.

Servlet하면 빼놓을 수 없는 Spring의 [DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html)도 같이 다뤄보면 좋겠지만, 다음에 따로 다뤄보도록 하겠습니다.
