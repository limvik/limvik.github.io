---
layout: post
title: 스프링에서 제어의 역전(Inversion of Control)은 제어의 역전이 아니다?
categories:
- Framework
- Spring
tags:
- Spring
- Java
- IoC
- DI
date: 2023-05-22 23:21 +0900
---
## Intro

Spring 을 입문하면서 제어의 역전(IoC, Inversion of Control)과 의존성 주입(Dependency Injection)에 대해서 배웠습니다. 중요한 개념이라 한 번 정리하려고 글을 작성하면서 여러가지 자료를 검토하다 보니, 제어의 역전과 의존성 주입은 설명하는 곳 마다 미묘하게 다른 느낌을 받아 조사한 것을 정리할 겸 글을 작성해봤습니다.

이후 제어의 역전은 IoC, 의존성 주입은 DI로 통일하겠습니다.

## 용어 정의 찾아보기

### 책에서 찾아보기 

아래는 제가 봤던 스프링 관련 책 중에 IoC에 대한 정의 입니다.

1. 없음, 아예 IoC라는 용어를 사용 안함
2. 다른 객체를 직접 생성하거나 제어하는 것이 아니라 외부에서 관리하는 객체를 가져와 사용하는 것
3. 서블릿이나 빈 등을 개발자가 코드에서 생성하지 않고 프레임워크가 직접 수행하는 방법

동일한 책에서 DI에 대한 정의입니다.

1. 의존하는 부분을 외부에서 주입하는 것
2. IoC를 구현하기 위해 사용하는 방법
3. 클래스 객체를 개발자가 코드에서 생성하지 않고 프레임워크가 생성하여 사용하는 방법

대체로 `DI를 IoC의 하위 카테고리로 설명`하고 있습니다. 

### 검색해보기

다른 분들이 블로그에 올린 글에서 IoC는 설계 원칙 혹은 디자인 패턴이고, DI는 IoC를 구현한 하나의 예라 서로 다른 것이라고 합니다.

Spring 6.0.9 Reference 문서([링크](https://docs.spring.io/spring-framework/reference/core/beans/introduction.html))를 보면 아래와 같이 나옵니다.

> This chapter covers the Spring Framework implementation of the Inversion of Control (IoC) principle. IoC is also known as dependency injection (DI).
>
>이 챕터는 IoC 원칙의 Spring 프레임워크 구현에 대해 다룹니다. IoC는 DI라고도 합니다.

Reference 문서 혼자만 다른 느낌입니다.

### 정리

포함 관계로 나타내 보자면 아래와 같습니다.

- 봤던 책, 블로그 검색 : IoC ⊃ DI
- Spring 6.0.9 Reference 문서 : IoC = DI

당연히 Reference 문서를 따라야 겠지만, 왜 이렇게 혼란스러운건지 궁금했습니다. 그래서 Spring 초창기 버전의 Reference 문서에는 뭔가 더 핵심적인 내용을 담지 않았을까 싶어서 찾아봤습니다.

## Spring 역사 공부

### Spring 과거 Reference 문서 찾아보기

Spring의 과거 문서를 살펴보니 1.1.1 버전에서부터 Reference 문서를 제공합니다. 문서의 Background Information 챕터의 Inversion of Control / Dependency Injection 섹션을 보면 아래와 같은 내용이 적혀있습니다.

---

>2004년 초, 마틴 파울러는 자신의 사이트 독자들에게 제어의 역전에 대해 이야기할 때 이렇게 물었습니다: "문제는, 그들이 통제의 어떤 측면을 뒤집고 있느냐는 것입니다". `IoC`라는 용어에 대해 이야기한 후 Martin은 패턴의 이름을 바꾸거나 적어도 좀 더 이해하기 쉬운 이름을 붙일 것을 제안하고 `DI`라는 용어를 사용하기 시작했습니다. 그의 글에서는 IoC 또는 DI에 대한 몇 가지 아이디어를 계속 설명합니다. 더 깊이 있는 인사이트가 필요하다면 http://martinfowler.com/articles/injection.html.

---

깊이 있는 인사이트를 얻고 싶어서 제시된 링크를 따라가 봤습니다.

### 마틴 파울러가 제시한 의존성 주입이라는 용어

[질문과 관련된 내용](https://martinfowler.com/articles/injection.html#InversionOfControl) 중 일부만 아래 붙여넣었습니다.

---

>이러한 컨테이너가 IoC를 구현하기 때문에 매우 유용하다고 이야기할 때 저는 매우 당황스럽습니다. IoC는 프레임워크의 일반적인 특성이므로, 이 경량 컨테이너가 `IoC를 사용하기 때문에 특별하다고 말하는 것은 내 차가 바퀴가 있어서 특별하다고 말하는 것과 같습니다`.
>
>...
>
>따라서 이 패턴에 대해 좀 더 `구체적인 이름이 필요`하다고 생각합니다. IoC라는 용어는 너무 일반적이어서 사람들이 혼동할 수 있습니다. 그 결과 다양한 IoC 옹호자들과 많은 논의를 거쳐 `DI`라는 이름을 정했습니다.
>
>...

---

글 시작 시 `이러한 컨테이너` 는 Spring을 포함한 당시 IoC 원칙을 구현했다고 하는 여러 프레임워크를 의미합니다.

같은 마틴 파울러의 웹 사이트에서 [IoC만 별도로 다룬 글](https://martinfowler.com/bliki/InversionOfControl.html) 중에는 아래와 같은 내용도 볼 수 있습니다.

---

>최근 IoC 컨테이너의 등장으로 인해 제어 역전의 의미에 대해 약간의 혼란이 있는데, 일부 사람들은 `일반적인 원칙과 이러한 컨테이너가 사용하는 특정 IoC 스타일(예: DI)을 혼동`하고 있습니다. IoC 컨테이너는 일반적으로 EJB의 경쟁자로 간주되기 때문에 이 이름은 다소 혼란스럽고 아이러니합니다. 하지만 EJB도 IoC를 똑같이(그 이상은 아니더라도) 사용합니다.

---

### 정리하기

위와 같은 자료들을 종합해볼 때 마틴 파울러는 IoC를 두 가지 의미로 사용하고 있는 것 같습니다.

- 당연한 현상으로서의 IoC : 마틴 파울러가 제시한 자동차 예시를 볼 때 프레임워크는 당연히 IoC 현상이 발생한다는 것을 암시합니다. 그리고 IoC만 별도로 작성한 글을 시작하면서 IoC는 프레임워크 전체에 나타나는 일반적인 현상이라고도 합니다. 이러한 IoC는 너무 일반적인 의미를 가진 용어라 합의 하에 DI로 이름을 정했다고 했습니다.
>Inversion of Control is a common `phenomenon` that you come across when extending frameworks.
- 설계 원칙으로서의 IoC : 앞서 보신대로 IoC만 별도로 작성한 글에서 IoC 컨테이너 때문에 사람들이 `일반 원칙(general principle)`과 IoC의 특정 스타일인 DI를 혼동한다고 했습니다. 용어의 사용을 중요시 하는 마틴 파울러를 고려해볼 때 `phenomenon`과 `principle`을 혼용해서 썼을 것 같지는 않습니다.

어떤 의미로 사용되든 IoC라는 용어가 IoC 컨테이너가 하는 일을 명확히 나타내지 않고 혼란스럽게 하니 DI라는 용어를 사용하기 시작한 것으로 보입니다.

하지만 Spring은 포기하지 않고 꿋꿋하게 IoC라는 용어를 사용하다보니 혼란이 지속됩니다.

글을 쓰면서도 제가 제대로 쓰고 있는건지 혼란스럽습니다.

## 추가 자료 탐색

Pro Spring 5(한글 번역판: 전문가를 위한 스프링 5) 라는 책을 발견해서 간단하게 훑어봤습니다.

>As you will see in Chapter 3, using the term dependency injection when referring to inversion of control is always correct. `In the context of Spring`, you can use the terms `interchangeably`, without any loss of meaning. --p.8
>At its core, `IoC, and therefore DI`, aims to offer a simpler mechanism for provisioning component dependencies (often referred to as an object’s collaborators) and managing these dependencies throughout their life cycles. --p.37

Spring의 문맥 내에서는 IoC나 DI나 같은 용어라고 합니다. 스스로를 DI Framework 라고 칭하는 문장도 있었습니다.

In the context of Spring 이라는 문장으로 IoC의 범위를 한정하는 것을 보니 설계 원칙으로서의 IoC나 현상으로서의 IoC와는 구분하기 위함이라 판단됩니다.

수많은 Spring 책 중에 한 권이 아닌가 싶을수도 있지만, 번역판의 광고 카피로 Spring 창시자인 로드 존슨이 인정한 유일한 책이라고 하고, 저자 중에 한 분은 Spring 개발진 중에 한 분이라고 하니 다른 책 보다는 더 신뢰할 수 있다고 생각됩니다.

결론적으로 Spring을 설명할 때는 DI가 IoC의 하위 카테고리가 아닌 그냥 동의어임을 알 수 있습니다. IoC를 아예 언급하지 않은 입문서가 현명했다고도 할 수 있겠습니다.

Spring의 IoC는 설계 원칙으로서 IoC에 영향을 받아 구현을 하기는 했으니까, 요즘 적당히 제품 이름에 AI 붙이듯 붙인게 아닐까 생각됩니다.

근데 이게 또 찾다보니 설계 원칙으로서의 IoC가 뭔지 궁금해집니다.

### 설계 원칙으로서의 IoC 간단히 살펴보기

설계 원칙으로서의 IoC 가 뭔지 자세히 보려면 위의 마틴 파울러가 작성한 글부터 시작해서 공부할게 산더미인 것 같아, 이번에는 Spring의 경쟁자(였던 것으로 보이는) PicoContainer에 써있는 IoC와 관련된 글을 간단하게 살펴봤습니다.

PicoContainer는 마틴 파울러의 글에서도 언급되며, 자신의 동료들이 많이 참여했다고 합니다.

[IoC 역사에 대한 글](http://picocontainer.com/inversion-of-control-history.html)을 보면 서두에 이런 언급을 합니다.

>온라인에서는 IoC가 DI로 이름이 변경되었는지 여부에 대해 약간의 혼란이 있습니다. 이는 사실이 아니며, DI는 IoC의 한 측면인 Component Assembly에만 해당합니다. 다른 두 가지는 Configuration과 Life Cycle입니다.

그리고 경쟁자인 Spring 과의 비교를 아래와 같이 썼습니다.

>Spring 프레임워크는 J2EE 프레임워크입니다. 따라서 DI와 Life Cycle 이 관심사 중 하나일 뿐입니다. 반면에 PicoContainer는 DI, Configuration 및 Life Cycle에만 관심이 있습니다.

자세한 내용은 모르겠지만, IoC를 구현했다고 할 수 있으려면 Component Assembly, Configuration, Life Cycle 을 구현해야 하는 것으로 보입니다.

그리고 IoC에 대해 정리한 문서를 보면, 목표는 의존성 해결임을 알 수 있습니다.

>IoC의 가장 중요한 측면은 의존성 해결이며, IoC를 둘러싼 대부분의 논의는 의존성 해결에만 집중되어 있습니다. 의존성의 잘못된 관리는 IoC가 해결하고자 하는 가장 큰 문제로 인식되고 있습니다.

지금은 간단하게 이정도만 살펴보고 넘어가야겠습니다.

## 보너스: 컨테이너는 또 뭔가?

또 따로 쓰기에는 분량이 애매할거 같아서 보너스로 씁니다.

IoC와 DI에 대해 찾다보면 컨테이너라는 단어가 항상 같이 붙어나오는데, 이름에서 기능을 유추할 수 있기도 하고 설명을 들으면 감이 오기는 합니다.

### Spring Reference 문서

[Spring reference 문서](https://docs.spring.io/spring-framework/docs/1.1.1/reference/beans.html)를 보면, 객체(beans)를 관리하는 일을 한다는 것을 유추할 수 있습니다.

>The `org.springframework.context.ApplicationContext` interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata.

그래도 어디 정리된게 있나 싶어서 찾아봤습니다. 정리된 것을 찾을 수는 없었지만 컨테이너라고 부를 수 있을만한 조건이 이것이 아닐까? 하고 유추해 볼 수는 있었습니다.

### JSR-330 : Dependency Injection for Java

Spring 을 보다보면 JSR-330([문서 링크](http://javax-inject.github.io/javax-inject/)) 을 심심치 않게 볼 수 있는데, Github([링크](https://github.com/javax-inject/javax-inject))에 보면 Spring 을 만든 로드 존슨이 리드한 프로젝트임을 알 수 있습니다. 그리고 패키지에 대해 설명한 문서([링크](http://javax-inject.github.io/javax-inject/api/index.html))에서 제일 아래를 보면 아래와 같이 나와있습니다.

>A "container", for some definition, can be an injector, but this package specification aims to minimize restrictions on injector implementations.
>
>일부 정의에 따라 "컨테이너"는 인젝터일 수 있지만, 이 패키지 사양은 인젝터 구현에 대한 제한을 최소화하는 것을 목표로 합니다.

Spring을 만든 사람이 주도한 프로젝트니 Spring을 만들 때의 생각이 반영되지 않았을까 생각이 됩니다.

그리고 제 수준에서 읽기에는 컨테이너라고 할 수 있으려면 최소한 의존성을 주입할 수 있어야 한다는 것으로 읽힙니다.

컨테이너가 의존성을 주입하는 것의 의미도 있다면, IoC를 붙인건 마케팅용이 아닐까 하는 생각이 듭니다. DI를 붙인다면 의존성을 주입할 때 DI를 사용하는 컨테이너라고 해석할 수 있겠습니다.

점점 소설로 들어가니 글을 마무리 해야겠습니다.

## Outro

용어가 혼란스러워 자료 찾아본다고 시간을 많이 쓴 것 같습니다. 쓸데 없는 호기심이 될지 도움이 되는 지식이 될지 더 공부해봐야 알겠습니다.

IoC 라는 용어가 사용되는 맥락에 따라 의미를 잘 해석하고, Spring 한정으로 IoC 는 DI로 치환해서 읽어야겠습니다.

방문자가 없는 블로그지만 혹시나 우연히 읽으시고 더 혼란스러워질 수도 있을 분들을 위해 참고 자료를 아래 정리합니다.

## 참고 자료

- 스프링 프레임워크 첫걸음
- 스프링 부트 3 백엔드 개발자 되기: 자바 편
- 자바 웹을 다루는 기술
- Pro Spring 5(한글판: 전문가를 위한 스프링 5)
- [https://docs.spring.io/spring-framework/reference/core/beans/introduction.html](https://docs.spring.io/spring-framework/reference/core/beans/introduction.html)
- [http://picocontainer.com/inversion-of-control-history.html](http://picocontainer.com/inversion-of-control-history.html)
- [http://picocontainer.com/inversion-of-control.html](http://picocontainer.com/inversion-of-control.html)
- [http://picocontainer.com/comparisons.html](http://picocontainer.com/comparisons.html)
- [https://martinfowler.com/bliki/InversionOfControl.html](https://martinfowler.com/bliki/InversionOfControl.html)
- [https://martinfowler.com/articles/injection.html](https://martinfowler.com/articles/injection.html)
