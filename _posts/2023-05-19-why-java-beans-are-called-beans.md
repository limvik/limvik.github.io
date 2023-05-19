---
layout: post
title: 왜 Java Beans 의 객체는 Beans라고 부르나요?
categories:
- Java
tags:
- Java
- JavaBeans
date: 2023-05-19 23:31 +0900
---
## Intro

요즘 Java Framework 인 Spring에 입문하면서 Bean 객체를 XML에서 설정해보고, Annotation으로 설정하기도 해보고 있습니다.

근데 쓰잘데기 없기는 하지만 왜 Bean이라고 이름을 지었는지 문득 궁금해서 찾아봤습니다.

## GPT-4, Bing chat, Bard 에 물어보기

GPT-4 와 Bing chat은 못 찾겠다고 포기를 선언했고, Bard는 근거는 없지만 그럴듯하게 써줍니다.

>Beans라는 이름은 1996년 Java Beans 1.0 릴리스를 공동 작업한 Sun Microsystems의 엔지니어 그룹인 Java Beans 팀에 의해 선택되었습니다. 팀은 Beans라는 이름이 작고 유연하며 재사용 가능한 컴포넌트를 잘 설명한다고 생각했습니다.
>
>Java Beans 팀은 Beans라는 이름을 선택한 이유에 대한 공식적인 문서를 발표하지 않았습니다. 그러나 팀의 멤버인 Bill Joy는 2004년 인터뷰에서 Beans라는 이름이 콩과 식물의 작고 유연한 특성에서 영감을 받았다고 말했습니다. Joy는 콩과 식물이 작고 유연하며 다양한 조건에서 자랄 수 있다고 말했습니다. 그는 Java Beans도 작고 유연하며 다양한 애플리케이션에서 사용할 수 있다고 말했습니다.

근거 자료 내놓으라니까 도망칩니다.

>저는 단지 언어 모델일 뿐이고, 그것을 이해하고 응답하는 능력이 없기 떄문에 도와드릴 수가 없습니다.

혹시나 해서 최근 chat GPT에 추가된 웹 브라우징 기능으로 2004년 인터뷰 찾아봐 달라고 했는데 작은 단서도 못찾았습니다.

## 검색해보기

인터뷰는 직접 찾아봐도 잘 나오질 않아서 stack overflow 에서 찾아보니 근거는 없지만 그래도 가장 공감되는 글([링크](https://stackoverflow.com/questions/18609030/why-java-beans-are-called-beans))을 찾았습니다.

아래에 글 내용과 제가 찾은 내용을 합쳐서 정리해봤습니다.

### Java

Java가 인도네시아의 Java coffee 에서 따왔다는 것은 많이 나와있어서 Java 개발자분들이라면 한 번쯤 들어보셨을 것 같습니다. 못들어 보셨어도 Java의 브랜드 로고가 커피라 대충 커피와 관련됐을 거라는 생각은 해보셨겠죠?

![Java Logo](https://upload.wikimedia.org/wikipedia/en/thumb/3/30/Java_programming_language_logo.svg/121px-Java_programming_language_logo.svg.png)

출처: [Wikipedia](https://en.wikipedia.org/wiki/Java_%28programming_language%29)

뇌피셜이긴 하지만, 그래서 Java EE가 이클립스 재단으로 넘어가서 리브랜딩 했을 때 인도네시아의 지역명인 Jakarta 로 변경하지 않았나 싶습니다.

### Java Beans in Jar

커피를 잘 모르긴 하지만 대충 알고 있기로는 커피 콩을 분쇄해서 필터에 넣고 뜨거운 물을 붓는 방식으로 만드는걸로 알고 있습니다.

이런 커피를 만들기 위한 커피 콩은 아래 그림처럼 무색 투명한 유리병에 보관합니다.

![무색 투명한 유리병에 들어있는 커피 콩 사진](/assets/img/2023-05-19-why-java-beans-are-called-beans/beans-in-jar.jpg)

이러한 유리병을 영어로 jar 라고 합니다.

그리고 Java는 배포할 때 JAR(Java ARchive) 파일로 만들어서 배포합니다.

이게 이렇게 연결되니까 뭔가 재밌다는 생각이 들었습니다. 이게 뭐라고 쓸데없이 검색하는데 시간을 썼는지 모르겠습니다. Spring이 별로 재미가 없었나...

## Outro

Stack overflow 글에도 별로 Up vote가 없는 걸 보면, 별로 궁금한 사람이 없는 것 같습니다. 저는 왜 갑자기 쓸데없는 호기심이 발동했는지 모르겠습니다.

Spring이나 열공해야겠습니다.
