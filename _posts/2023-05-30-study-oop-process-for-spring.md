---
layout: post
title: Spring 공부를 위한 객체지향 프로세스 공부
categories:
- OOP
tags:
- OOP
- Java
date: 2023-05-30 22:58 +0900
---
## Intro

여러 책에 있는 내용 한 곳에 몰아 넣으면서 공부하다가 그대로 블로그 글로 올렸습니다.

## 다시 객체 지향 공부로

[앞서 글](https://limvik.github.io/posts/is-inversion-of-control-not-inversion-of-control-in-spring/)을 썼듯 Spring에 입문했습니다. 문제는 Spring 예제를 복사해다가 붙여넣든 어쩌든, 어떻게든 되게 만드는건 할 수 있겠는데, 실습파 보다는 학구파인지라 뭔가 만들면서도 찝찝합니다. 검색을 하다가 Spring 을 만든 로드 존슨(Rod Johnson)이 쓴 책 도입부만 조금 살펴 봤습니다.

Spring의 토대가 됐다는 Expert One-on-One J2EE Design and Development([링크](https://books.google.co.kr/books?id=N6vEfdJws3IC&printsec=frontcover&source=gbs_ge_summary_r&cad=0#v=onepage&q&f=false))에서는 객체 지향을 강조하고 있습니다.

> In particular, the message I'll try to get across will be that we should apply J2EE to realize OO design, not let J2EE technologies dictate object design.
> 특히 제가 전달하고자 하는 메시지는 J2EE 기술이 객체 설계를 좌우하는 것이 아니라, J2EE를 적용하여 OO 설계를 실현해야 한다는 것입니다.
> 16p.

그 다음으로 Spring 에 대한 책 Professional Java Development with the Spring Framework (Rod Johnson, Juergen Hoeller, Alef Arendsen etc.)([링크](https://books.google.co.kr/books?id=oMVIzzKjJCcC&printsec=frontcover&source=gbs_ge_summary_r&cad=0#v=onepage&q&f=false))에서는 필요한 선수 지식을 알려줬습니다. 다른 지식도 필요한게 있었지만 객체 지향과 Java 기능에 대해 언급한게 눈에 들어왔습니다.

>We assume sound knowledge of OO design and Java language features including reflection, inner classes, and dynamic proxies.
>리플렉션, 내부 클래스, 동적 프록시 등 OO 설계와 Java 언어 기능에 대한 지식이 충분하다고 가정합니다.

내부 클래스는 기본서에서 배웠지만, 리플렉션이나 동적 프록시가 Java의 기능으로 존재하는지는 처음 알게됐습니다. 그래서 객체 설계부터 직접 해보면서 리플렉션이나 동적 프록시를 사용해보려고, 결국 다시 객체 지향과 Java 공부로 돌아왔습니다.

참고로 『토비의 스프링』 책의 목차를 보니 리플렉션하고 동적 프록시에 대해서도 다루고 있습니다. 저는 아직 안봐서 모르겠지만, 오래됐는데도 괜히 사람들이 추천하는게 아니었던 것 같습니다.

## 참고한 책

이전에 『객체지향 사실과 오해』 를 읽고([후기 링크](https://limvik.github.io/posts/book-report-the-essence-of-object-orientation-roles-responsibilities-collaborations/))나서 나름 객체 지향으로 프로그래밍을 해봤는데([후기 링크](https://limvik.github.io/posts/review-a-console-flashcards-app-for-practice/)), 결과물이 만족스럽지 못했습니다.

그러다 Coursera([링크](https://www.coursera.org/learn/cs-programming-java)) 추천으로 알게된 프린스턴대 교수님(빨간색 알고리즘 책으로 유명하신 분)의 Java를 이용한 프로그래밍 입문서([링크](https://introcs.cs.princeton.edu/java/home/))가 있어서 데이터 타입 설계와 관련 부분만 간단하게 살펴보게 됐습니다.

정확한 책 이름은 『Introduction to Programming in Java - An Interdisciplinary Approach』 입니다. 프로그래밍 입문서라 객체라는 단어 자체를 강조하기 보다는 다양한 주제로 메서드 2~3개 있는 `데이터 타입`으로서 객체를 만드는 방법을 소개하며 `모듈화`의 이점에 대해 느껴볼 수 있게 해줍니다. 그래서 저 같은 초보자에게 딱 맞는 느낌입니다. (심지어 컴퓨터를 사용해본적 없는 사람도 대상으로 하지만, 수학은 좀 해야된다고 합니다. 그래서 전 많은 부분을 패스...) 물론 이거 읽고 갑자기 객체 지향 프로그래밍을 잘 하게 됐다는 것은 아닙니다. 『객체지향 사실과 오해』에서는 잘 와닿지 않던 부분이 저한테 조금 더 쉽게 이해가 되는 용어로 설명이 돼있을 뿐입니다.

추가로 『디자인 패턴의 아름다움』은 동적 프록시 공부하면서 프록시 패턴에 대해서도 공부할 겸 구매한 책인데, 객체 지향에 관련된 내용도 담고 있습니다. 굉장히 실무적으로 접근하고 있어 흥미롭습니다. 이 책도 같이 정리(라 쓰고 따라쓰기)해 보겠습니다.

## 도서별 객체 지향 개발 프로세스

### 『객체지향 사실과 오해』

『객체지향 사실과 오해』 에서 강조하는 단어는 협력, 책임, 역할, 메시지 등이 있습니다. 책의 마지막 챕터에서 진행한 간단한 객체 지향 설계 예제를 바탕으로 순서와 관련 내용을 아래 정리해 봤습니다.

1. 도메인 모델 설계
	- 객체 `식별` 및 타입(개념) 별 `분류`
		- 예) 아메리카노, 에스프레소 객체는 모두 `커피 타입`의 인스턴스
	- 분류 된 타입 간 `관계` 파악
		- 예) 바리스타 타입과 커피 타입 간의 관계
	- 초점: `어떤 타입`이 도메인을 구성하는지, 그리고 타입들 사이에 `어떤 관계`가 존재하는지를 파악하여 `도메인 이해`하기
2. 협력 설계
	- 협력: 객체들이 애플리케이션의 기능을 구현하기 위해 수행하는 상호작용 / 어떤 객체가 다른 객체에게 무엇인가를 요청하는 것 / 적절한 `객체`에게 적절한 `책임`을 할당하는 일
	- 책임: 객체가 협력에 참여하기 위해 수행하는 로직 / 객체가 협력에 참여하기 위해 수행해야 하는 `행위`를 상위 수준에서 `개략적으로 서술`한 것
	- `책임`을 결정한 후 실제로 `협력`을 정제하면서 이를 `메시지`로 `변환`할 때는 하나의 책임이 여러 메시지로 `분할`되는 것이 일반적
	- 객체 지향 목표는 훌륭한 `객체`를 설계하는 것이 아닌 훌륭한 `협력`을 설계하는 것
	- `협력` 관계 속에서 다른 객체에게 무엇을 `제공해야` 하고 다른 객체로부터 무엇을 `얻어야` 하는가라는 관점에서 접근할 때만 훌륭한 `책임` 수확 가능
	- `책임-주도 설계 방법`에서 역할, 책임, 협력을 식별하는 것은 애플리케이션이 수행하는 `기능`을 시스템의 `책임`으로 보는 것으로부터 시작
	- `사용자`의 관점에서 시스템의 `기능`을 명시하고, 사용자와 설계자가 공유하는 안정적인 구조를 기반으로 기능을 `책임`으로 변환하는 체계적인 절차 따르기
	- 시스템의 `기능`을 구현하기 위해 객체가 다른 객체에게 제공해야 하는 `메시지`에 대해 고민
	- `협력`을 바탕으로 첫 번째 `메시지` 선택
	- `메시지`를 먼저 선택하고 그 후에 `메시지`를 수신하기에 적절한 `객체` 선택하기(메시지를 처리하기에 적당한 타입을 `도메인 모델`에서 찾아보기)
	- 선택된 `객체`에 `책임`을 부여하고, `메시지`는 선택된 객체의 `공용 인터페이스`에 포함
	- 선택된 `객체`가 스스로 할 수 `있`는 일과 `없`는 일을 구분하여, 할 수 `없`는 일을 `대신` 할 수 있는 `객체`에 `메시지` 보내고 응답 받기
	- 위 과정을 반복하여 `협력`에 필요한 `객체`의 종류와 책임, 주고받아야 하는 `메시지`에 대한 `대략적인 윤곽` 잡기
3. 인터페이스 정리
	- `협력` 설계를 통해 얻어낸 것은 `객체`들의 `인터페이스`
	- 객체가 수신한 `메시지`가 객체의 `인터페이스` 결정
	- 객체가 어떤 `메시지`를 `수신 가능`하다는 것은 그 객체의 인터페이스 안에 `메시지`를 처리할 수 있는 `오퍼레이션`이 존재하는 것을 의미
		- 참고) 정리해보면 인터페이스는 수신하는 메시지를 나타내는 인수(arguments) 타입과 이를 처리하는 오퍼레이션인 메서드(method) 이름으로 구성됩니다. 그리고 Java에서 메서드 이름, 인수 타입 및 순서를 합쳐 Method Signature 라고 합니다. 바로 와닿지 않아 추가합니다.
4. 구현
	- 구현 중에 인터페이스는 `변경 가능`
		- 참고) Java 관점에서 Method Signature 변경 가능을 의미합니다. 하지만 책에서는 반환값에 대한 것도 고려하고 있어 책에서 말하는 인터페이스에는 Method Signature와 반환 타입도 포함되는 것으로 해석됩니다. 외부에 노출되는 모든 것이라 생각하는게 편하겠습니다.
	- 중요한 것은 설계가 아니라 `코드`
	- `협력`을 구상하는 단계에 너무 오랜 시간을 쏟지 말고 최대한 빨리 `코드`를 `구현`해서 설계의 `구현 가능 여부` 판단
	- 객체의 `속성`은 `캡슐화`
		- 인터페이스를 정하는 단계에서는 객체가 `어떤 속성`을 가지는지, 또 그 속성이 `어떤 자료 구조`로 구현됐는지를 고려하지 `않`는 것이 캡슐화를 위한 가장 훌륭한 방법

기타
- 인터페이스를 통해 실제로 상호작용을 해보지 않은 채 인터페이스의 모습을 정확하게 예측하는 것은 불가능에 가까움
- 설계가 제대로 그려지지 않는다면 고민하지 말고 실제 코드를 작성하면서 협력의 전체적인 밑그림 그리기

---

처음 읽을 때는 이해도 잘 안되고, 추상적이라는 느낌이 많이 들었는데 다른 객체 지향 관련 책을 보고난 후라 그런지 같은 텍스트인데도 새로운 느낌으로 다가옵니다.

다음은 프로그래밍 입문서에 나온 객체 지향 구현 방식을 살펴보겠습니다.

### 『Introduction to Programming in Java - An Interdisciplinary Approach』

제목이 너무 긴 관계로 『프로그래밍 입문서』라고 하겠습니다. 『프로그래밍 입문서』에서는 협력, 책임, 역할, 메시지와 같은 추상적인 단어들을 이용한 설명보다는 데이터 타입 하나를 구현하는 것에 더 초점을 맞춰 설명하였습니다. 일반적인 기본서에서 볼 수 있던 단어와는 매칭이 잘 안되는게 많아 책 내용 정리 겸 부연설명을 좀 많이 추가했습니다. 시작은 용어를 살펴 보는 것 부터 하겠습니다.

책의 웹 사이트([링크](https://introcs.cs.princeton.edu/java/33design/))와 Creating Data Type 단원의 강의 자료([링크](https://introcs.cs.princeton.edu/java/lectures/keynote/CS.9.CreatingDTs.pdf))를 직접 보시는 것도 좋겠습니다.

객체 지향 관련 용어 살펴보기

강의 자료 4페이지에 나온 간단한 객체 지향 프로그래밍에 대한 설명을 보면 아래와 같습니다. 

>Object-oriented programming (OOP)
>• Create your own data types.
>• Use them in your programs (manipulate objects) : An object holds a data type value. Variable names refer to objects.

객체 지향 프로그래밍은 `자신만의 데이터 타입`을 만들어서 자신의 프로그램에서 `사용(객체 조작)`하는 것이고, 객체는 데이터 타입의 `값`을 보유하며 `변수 이름`은 객체를 `참조`한고 합니다. Java 기본서 읽었으면 무리 없이 받아들일 만한 설명입니다.

그리고 같은 강의자료 페이지에 해당 강의에서 할 일은 `추상 데이터 타입(ADT, Abstract Data Type)`을 구현하는 것이고, 추상 데이터 타입은 클라이언트(ADT를 호출하는 코드)에서 표현(representation, 특정 문제를 해결하기 위해 데이터와 알고리즘을 표현하는 방식)이 `숨겨져` 있는 데이터 타입이라고 하고 있습니다. `캡슐화(encapsulation)`를 이야기 하고 있는게 유추 가능합니다.

데이터 타입이 무엇인가에 대해서는 책에서 조금 더 자세히 설명하고 있습니다.

>데이터 타입이란 `값의 집합`과 그 `값`에 정의된 `연산 집합`을 말합니다. int 및 double과 같은 기본 유형의 값과 연산은 Java에서 미리 정의되어 있습니다. 객체 지향 프로그래밍에서는 새로운 데이터 타입을 정의하기 위해 Java 코드를 작성합니다. 객체는 데이터 타입 `값`을 보유하는 엔티티로, 객체의 데이터 타입 `연산` 중 하나를 적용하여 이 데이터 타입 `값을 조작`할 수 있습니다. 새로운 데이터 타입을 정의하고 데이터 타입 값을 가진 객체를 조작하는 이러한 기능을 데이터 `추상화`라고도 하며, 2장의 기초가 된 원시 타입에 대한 절차적 프로그래밍 스타일을 자연스럽게 확장하는 `모듈식 프로그래밍 스타일`로 이어집니다. 데이터 타입을 사용하면 `데이터와 함수를 격리`시킬 수 있습니다. 이 장의 핵심은 `연산 내에서 데이터와 관련 작업을 명확하게 분리할 수 있다면 언제든 그렇게 해야 한다`는 것입니다.

그리고 객체지향 프로그래밍 출발점이 되는 기본 개념으로 아래와 같이 정리합니다.

>데이터 타입은 `값의 집합`과 그 `값`에 대해 정의된 `연산 집합`입니다. 우리는 데이터 타입을 독립적인 `모듈`로 구현하고 이를 사용하는 클라이언트 프로그램을 작성합니다. 객체는 데이터 타입의 `인스턴스`입니다. 객체는 `상태`, `행동`, `식별자`라는 세 가지 필수 속성으로 특징지어집니다. 객체의 `상태`는 데이터 타입의 `값`입니다. 객체의 `행동`은 데이터 타입의 `연산`에 의해 정의됩니다. 객체의 `식별자`는 객체가 저장되는 `메모리 내 위치`입니다. 객체 지향 프로그래밍에서는 생성자를 호출하여 객체를 생성한 다음 `인스턴스 메서드를 호출하여 객체의 상태를 수정`합니다. Java에서는 객체 `참조`를 통해 객체를 조작합니다.

『프로그래밍 입문서』에 나온 객체 지향의 용어를 어느정도 살펴봤으니 책에서 소개한 객체 지향 개발 방법을 정리해 보겠습니다. 첫 시작은 데이터 추상화와 API 설계이고, 강의 자료에서는 추상 데이터 타입(ADT)을 정의하고, API 테이블을 간단하게 한 페이지에 보여주고만 있는데, 책에서는 굉장히 중요성을 강조하고 있습니다.

1. 데이터 추상화
	- 데이터 타입을 `정의하는 과정`을 데이터 추상화라고 함
	- 필요한 데이터의 타입에 대한 `이해` 필요
	- 적절한 `추상화`를 식별하고 구현하는 것이 효과적인 객체 지향 프로그래밍의 핵심
	- 적절한 추상화로 작성된 코드는 거의 `자체 문서화(self-documenting)`될 수 있음
2. API(Application Programming Interface) 설계
	- API란? 사용 가능한 `Method Signature`에 대한 정확한 `명세(specification)`와 함께 비형식적(informal) 계약(해당 메서드가 수행해야 하는 작업에 대한 `설명`)에 대한 종합적인 정보
		- 비형식적(informal) 계약인 이유: 이상적으로는 API가 부수 효과(side effect)를 포함하여 가능한 모든 입력에 대한 동작을 명확하게 표현하고 구현이 명세를 충족하는지 확인할 수 있는 소프트웨어가 있어야 합니다. 안타깝게도 `명세 문제(specification problem)`라고 알려진 이론적 컴퓨터 과학의 근본적인 결과에 따르면 이 목표는 실제로 `달성하기가 불가능`합니다. 
	- 가장 중요하면서도 가장 어려운 단계는 API를 설계하는 것
	- API의 목적은 해당 데이터 타입을 사용하는 `클라이언트 프로그램을 작성하는 데 필요한 정보를 제공`하는 것
	- 좋은 API를 설계하는 데 들인 시간은 디버깅이나 코드 재사용을 통해 절약된 시간으로 반드시 `보상`받음
	- 프로그래머는 일반적으로 메서드가 수행해야 할 작업을 명확하게 명세(specification)하는 클라이언트와 구현(implementation) 간의 `계약(contract)`이라는 관점에서 생각
	- 클라이언트와 구현을 모두 스스로 작성할 때는 `자신과 계약`을 맺는 것이며, 이는 그 자체로 `디버깅`에 추가적인 도움을 제공하기 때문에 유용
	- API 설계의 기본 원칙은 클라이언트에게 `필요한 메서드만 제공`하고 다른 메서드는 제공하지 않는 것
	- API를 사용하여 클라이언트와 구현을 `분리`
	- 작은 프로그램을 작성할 때는 API를 명시하는 것이 과한 것처럼 보일 수 있지만, 언젠가 코드를 재사용해야 한다는 것을 알기 때문이 아니라 코드 중 일부를 재사용하고 싶을 가능성이 높고 어떤 코드가 필요할지 알 수 없기 때문에 `모든 프로그램을 재사용해야 한다는 생각으로 작성`하는 것을 고려
	- 데이터 타입이 `불변(immutable)`인지 여부를 API에 명시하여 클라이언트가 객체 값이 변경되지 않는다는 것을 알 수 있도록 하는 것이 이상적
3. 구현
	- 데이터 타입 구현은 클라이언트가 API를 제외하고는 데이터 타입에 대해 아무것도 모른다고 가정
	- 클라이언트와 구현 간의 `유일한 의존 지점이 API`임을 고집
	- 아래 순서대로 구현
		- 데이터 타입을 만들 클래스 파일의 main 메서드에 간단한 테스트 클라이언트(Test Client) 코드를 구현
			- 설계 가정 검증을 위해 Exception, Assertion 등을 활용 가능
		- 데이터 타입의 값인 인스턴스 변수(Instance Variables) 정의
		- 객체를 생성하고 초기화하기 위한 생성자(Constructors) 정의
		- 데이터 타입 연산(operation)인 메서드 구현하기(설계한 API 구현)

---

**기타**
- 데이터 타입은 구현의 `요구 사항`을 염두에 두고 문제의 필요에 따라 설계
- 프로그램 내에서 `데이터`와 관련 `연산`을 명확하게 `분리`할 수 있다면 언제든 그렇게 해야 함
- 크고 복잡한 애플리케이션을 다루는 것은 `데이터`와 데이터에서 수행해야 할 `연산을 이해`한 다음 이러한 `이해를 직접 반영하는 프로그램`을 작성하는 과정

---

### 막간의 깨달음

『객체지향 사실과 오해』 는 코드로 만들면서 계속해서 수정되는 것을 강조한 반면, 『프로그래밍 입문서』는 크게 강조하지 않았습니다. 프로그래밍 입문이라는 목적에 맞게 이상적으로 데이터 타입 설계 과정을 풀어내기 위함으로 생각됩니다.

도메인 모델 설계 등 일부 과정의 차이가 있지만 규모의 차이에서 비롯된 추가적으로 필요한 과정이라 생각되며, `추상화 후에 인터페이스를 설계하고 구현하는 큰 흐름은 유사`한 것으로 보입니다. 

그리고 『프로그래밍 입문서』를 읽으면서 『객체지향 사실과 오해』를 읽을 때 잘못 이해했던 부분을 바로잡을 수 있었습니다. 『객체지향 사실과 오해』만 읽었을 때는 객체의 `행동` 이라는 용어가 `메서드의 구현 부분까지 포함하는 것으로 잘못이해` 하고 있었습니다. 상태보다 행동이 우선하므로 이렇게 이해할 경우 『프로그래밍 입문서』에서 인스턴스 변수를 먼저 구현하라는 세부 지침과 충돌하게 됩니다.

그래서 『객체지향 사실과 오해』를 다시 읽다보니 '훌륭한 객체지향 설계는 `외부에 행동만을 제공`하고 데이터는 행동 뒤로 감춰야 한다. 이 원칙을 흔히 `캡슐화`라고 한다.' 라는 문장이 눈에 들어왔습니다. 행동이라는 것은 외부에 제공되는 것인데, 숨겨져야 할 구현이 외부에 공개된 다는 것은 말이 안된다는 걸 인지하지 못했던 겁니다. 프로그래밍 입문서에 있는 문장을 참고해보면 좀 더 명확해 집니다. '정보를 숨겨 클라이언트와 `구현을 분리`하는 과정을 `캡슐화`라고 합니다.'

용어를 혼동해서 메서드를 무조건 먼저 구현하고, 구현하면서 필요한 프로퍼티를 찾아야한다고 생각하고 실천했던 과거의 제가 안타까워집니다. 『프로그래밍 입문서』를 막 읽기 시작했을 무렵에 '클래스에서는 데이터 타입 값을 지정하고 데이터 타입 연산을 구현합니다.' 혹은 '먼저 인스턴스 변수를 선택한 다음 인스턴스 변수를 조작하는 코드를 작성하여 지정된 생성자 및 인스턴스 메서드를 구현합니다.' 라고 했을 때 고민에 빠지기도 했습니다.

### 중간 정리

두 책에 있는 정보가 배타적인 관계에 있는게 아니니 둘을 합쳐서 순서를 정리해보면 아래와 같습니다.

1. 도메인 모델 설계
	- 데이터 추상화 과정 포함
2. 협력 설계
3. API 설계
4. 테스트 구현
5. 인스턴스 변수 정의
6. 생성자 정의
7. 메서드 구현

그런데 문제는 이전에 도메인 모델 설계부터 잘못됐었던 경험이 있습니다. 저는 그 문제를 요구사항을 제대로 정의하지 않았기 때문이라고 생각했었습니다. 그리고 아직도 요구사항을 어떻게 정의해야 잘 정의하는 건지 잘 모르겠습니다. 그래서 요구 사항 분석에 대한 내용이 조금이나마 설명된 책이 있어 같이 정리해봅니다.

### 『디자인 패턴의 아름다움』

- 객체 지향 개발의 3단계
	1. 객체 지향 분석(Object Oriented Analysis)
	2. 객체 지향 설계(Object Oriented Design)
	3. 객체 지향 프로그래밍(Object Oriented Programming)
- 분석과 설계 앞에 객체 지향이 붙은 이유는 `객체`나 `클래스`에 대한 `요구 사항`을 분석하고 설계하기 때문이다.
- 분석과 설계라는 두 가지 단계를 거치면 프로그램이 어떤 클래스로 분해 구성되는지, 각각의 클래스가 어떤 속성과 메서드를 가지는지, 클래스끼리 상호 작용하는 인터페이스를 포함한 클래스 설계를 도출하게 된다.
- 간단히 정리하자면 객체 지향 `분석`은 `무엇`을 해야 하는지 알아내는 것이고, 객체 지향 `설계`는 그 일을 `어떻게` 해야 하는지를 정의하는 것이다. 그리고 객체 지향 `프로그래밍`은 앞에서 진행했던 분석과 설계의 결과를 `코드`로 구체화하는 것이다.

---

**객체 지향 개발 단계별 상세 내용**

1. **객체 지향 분석**
	- 실제 소프트웨어 개발 단계의 `요구 사항`도 대부분 명확하지 않다.
	- 객체 지향 분석에서 주요 분석 대상은 `요구 사항`이므로 객체 지향 분석을 풀어서 이야기하면 `요구 사항 분석`이라고 할 수 있다.
	- 사실 요구 사항 분석이든 객체 지향 분석이든 가장 먼저 해야 할 일은 `일반적인 요구 사항을 충분히 명확하고 실행 가능하도록 개선`하는 것이다.
	- 요구 사항 분석 작업은 하찮게 여겨지고 특별하게 정해진 규칙이 `없`기 때문에, 활용도가 그리 높지 않은 방법론에 얽매이는 대신 `사례`를 통해 요구 사항 분석에 대한 이해를 완벽하게 해서 `자신만의 분석 방법을 개발`하기 바란다.
	- 요구 사항 분석 프로세스는 지속적인 `반복 최적화 프로세스`다.
	- 요구 사항 분석을 위해 완벽한 해결 방법을 즉시 제공하는 대신, 먼저 `대략적인 기본 계획`을 제공한 다음 단계별로 천천히 `반복적으로 최적화`해야 한다.
	- `문제를 먼저 파악한 후 그 문제를 해결`하는 것은 매우 훌륭한 반복 최적화 방법이다.
	- 보안, 개발 비용, 시스템 성능에 미치는 영향 등을 고려했을 때 `합리적인 절충안`을 찾는다.
	- 비즈니스와 연관성이 낮은 기능을 개발할 때는 특정 제품에 특정 제품에 과도한 `의존`을 피하는 것이 좋다.
	- 객체 지향 분석이 완료되면 상세한 `요구 사항 명세`를 얻을 수 있다.
2. **객체 지향 설계**
	1. 책임과 기능을 나누고 어떤 클래스가 있는지 확인한다.
		- 객체 지향을 소개하는 대부분의 책은 클래스를 `식별`하는 방법, 다시 말해 요구 사항 명세에 있는 `명사`를 가능한 `후보 클래스`로 먼저 나열하고, 이를 `필터링`하는 방법을 언급한다. 초보자에게 이 방법은 간단하고 명확하며 따라하기 쉽다.
		- 다만 요구 사항 명세에 따라 관련된 `기능`을 하나씩 나열한 후, 어떤 기능들이 유사한 `책임`을 지고 동일한 `속성`을 사용하는지 확인하는 방법으로 클래스를 `분류`하는 방법을 추천한다.
		- 요구 사항 명세를 `단일 책임 원칙(Single Responsibility Principle)`에 따라 분해를 하면서, 분해되는 각각의 기능이 가지는 `책임은 가능한 작고 단순`해야 한다.
		- 요구 사항 명세를 분해하여 `기능 목록`을 얻는다.
		- 분해 결과는 실제 최종 클래스 구현과 다를 수 `있`다.
		- 객체 지향 프로그래밍은 본질적으로 반복을 통한 지속적인 최적화 프로세스다. 요구 사항에 따라 먼저 `대략적인 설계 계획`을 제공한 다음, 이를 기반으로 `반복적인 최적화`를 수행하여 더 `명확한 구현`을 해나간다.
		- 복잡한 요구 사항을 개발하려면 먼저 모듈을 나누고, `요구 사항`을 여러 개의 작고 독립적인 `기능 모듈로 나누는 작업`이 선행되어야 한다. 그런 다음 모듈 내부에서 객체 지향 설계를 해야 한다.
	2. 클래스, 클래스의 속성과 메서드 정의
		- 속성과 메서드를 정의하기 위해 앞 단계에서 얻은 클래스별 `기능 목록`에서 `동사`나 `명사`를 추출하고, 이어서 이를 `필터링`하는 방법을 사용한다. 모든 `명사`가 클래스의 속성으로 정의되는 것은 아니며, 일부는 `메서드 매개변수` 형태로 전달될 수 있다.
		- 위 방법을 통해 클래스의 속성과 메서드를 정의하고, 클래스의 `세부 사항`을 검토한다.
		- 세부 사항을 통해 정의한 클래스에 `비즈니스 모델 측면`에서 배치되어서는 안되는 속성과 메서드, 현재 요구 사항만 가지고는 개발이 불가능한 속성 및 메서드가 무엇인지 알 수 있으며 비즈니스 모델에서 클래스가 가져야 하는 속성 및 메서드도 알 수 있다.
	3. 클래스 간의 상호 작용 정의
		- UML 에 정의된 클래스 상호 작용을 활용하되 불필요할 정도로 세부적이므로 모두 사용할 필요는 없다.
	4. 클래스 연결 및 실행 엔트리 포인트 제공
		- 엔트리 포인트는 main() 함수일 수도 있고, 외부 호출을 위한 API 모음일 수도 있다.
		- 시스템에서 통합 실행되는 구성 요소인 경우 모든 구현 세부 사항을 캡슐화한 다음, 최상위 인터페이스 클래스를 설계하고, 외부에서 사용할 API를 노출한다.
3. **객체 지향 프로그래밍**
	- 객체 지향 프로그래밍 작업은 앞서 준비한 설계 사상을 코드로 `실체화`하는 것이다.

---

**기타**
- 일반적으로 대부분의 프로그래머는 종종 객체 지향 분석과 객체 지향 설계를 머릿속이나 간단한 스케치로 마친 후 바로 코드 작성을 시작할 뿐만 아니라, 코드 작성 도중에도 최적화와 리팩터링을 하는 경우가 많다.
- 모든 세부 사항과 상호 작용에 대해 한 번에 다 정의하는 것은 불가능하다. 코드를 작성하면서 여전히 코드를 뒤엎어 리팩터링하고, 또 이를 반복해야 한다.
- 결국 소프트웨어 개발은 본질적으로 지속적인 반복, 패치, 문제 발견, 문제 해결의 과정이며, 지속적인 리팩터링의 과정이다.
- `엄격하게 하나의 단계를 모두 마치고 다음 단계로 넘어가는 것이 불가능`하다는 것을 알아야 한다.

## 반성하기

이것 저것 보다보니, 이전에 나름 객체 지향 개발을 해본다고 했을 때 제가 뭘 잘못했었는지 조금이나마 보이기 시작합니다. 두 가지로 분류해 보자면 하나는 마음가짐의 문제, 하나는 지식 부족의 문제가 되겠습니다. 

마음가짐의 문제로는 각 단계에서 잘못된 것이 발생하는 것을 당연하게 받아들이지 못했고(내탓이오 내탓이오), 반복적으로 이전 단계로 되돌아가서 수정하는 것도 당연하게 받아들이지 못했습니다. 또 각 구분된 단계를 과할 정도로 고수하려고 했던 문제가 있었습니다. 그리고 지식 부족의 문제로는 문제가 문제인지 조차 인지하지 못하는 경우도 있었고, 문제가 있음을 알고 있지만 어떻게 고쳐야 할지 몰라 수정을 하지 못하기는 문제도 있었습니다.

마음가짐의 문제는 끊임 없는 훈련이 필요하며, 무엇이 문제이며 해결책이 무엇인지 제대로 파악하지 못한건 지식의 부족인지라 끊임 없는 공부밖에 답이 없겠습니다.

## 모두 함께 정리해보기

그럼 앞서 본 책들의 내용과 저의 생각들을 합쳐 이후 실습해볼 프로세스를 정리해보겠습니다.

### 지극히 개인적인 객체 지향 개발 프로세스

1. 문제 정의 및 요구사항 도출
	- 해결하려는 문제 정의
	- 용어 정의
	- 추상적인 기능 요구사항 도출
2. 요구사항 분석
	- 요구사항 우선순위 설정
	- 유저 시나리오, 유저 스토리 작성 후 `명사`와 `동사` 추출
3. 도메인 모델 설계
	- 요구사항 분석에서 추출한 `명사`를 `객체`로 만들고, `타입`으로 `분류`
	- 분류한 타입 간의 `관계` 존재 여부 파악
4. 협력 설계
	- 요구사항 분석에서 추출한 `동사`를 `협력`의 관점에서 파악하고, 시작  `메시지` 생성
	- 협력 시에 필요한 세부적인 `정보`와 해당 정보에 대한 `처리(행동)`를 하나의 단위로 `책임 목록` 작성
	- 도메인 모델을 참고하여 책임을 부여하기에 적합한 타입에 `책임 부여`
		- 단일 책임인지 여부는 시나리오 혹은 관점에 따라 달라질 수 있음
		- 무조건 책임을 작게 나눈다고 좋은 것은 아니므로, 가독성, 확장성, 재사용성, 유지 보수성 등을 고려하여 판단
	- 앞서 생성한 `메시지` 를 처리하기에 적합한 책임을 갖고 있는 타입에 `메시지 전송`
	- 메시지를 수신한 타입이 혼자서 처리할 수 없는 메시지인 경우 다른 타입에 메시지 전송
5. API 설계
	- `메시지` 및 `반환되는 정보`에 포함되는 `정보`에 대한 구체적인 데이터 타입 지정
	- API 테이블 대신 Javadoc 작성하기([참고할 글 링크](https://johngrib.github.io/wiki/java/javadoc/#boolean-%EB%A9%94%EC%86%8C%EB%93%9C%EC%9D%98-%EA%B2%BD%EC%9A%B0))
6. 테스트 구현
	- JUnit5 이용
	- 세부 구현 전: 메서드 마다 간단한 테스트 케이스 1개로 테스트 작성
	- 세부 구현 후: 경계값 테스트 등 메서드 구현에 맞춘 테스트
7. 인스턴스 변수 정의
8. 생성자 정의
9. 메서드 구현
10. 실행 엔트리 포인트 제공
11. 리팩터링 반복

### 지극히 개인적인 주의사항

1. 각 단계를 단 한번에 완벽하게 마무리 지으려고 하지 않기
2. 프로세스 자체가 상황에 적합하지 않을 가능성 염두에 두기

## Outro

뭔가 잘못 생각한게 있나 자꾸 시간쓰게 되는데, 만들면서 잘못된걸 발견해 보는게 더 빠를 것 같아서 마무리합니다.

문제는 『객체지향 사실과 오해』의 저자 분이 쓰신 『오브젝트』를 빠르게 훑어봤는데, 제가 이해한 추상 데이터 타입과는 또 다른 이야기를 풀어주셔서 실습 좀 해보고 추후에 천천히 읽으면서 잘못된 지식을 수정해 봐야겠습니다.

항상 주의할 점은 『오브젝트』 의 서문에서 `이 책에서 제시하는 방법이 객체지향을 구현하기 위한 유일한 방법이 아니라는 점`을 강조하셨고, 『Introduction to Programming in Java - An Interdisciplinary Approach』 에서도 `아직 가장 좋은 설계 아이디어가 무엇인지 대해 논의가 이루어지고 있다`고 하셨으므로 어떤 것이 정답이라는 생각은 갖지 않도록 해야겠습니다.