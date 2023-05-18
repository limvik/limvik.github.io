---
layout: post
title: Java 형 변환 연산자 우선순위
categories:
- Java
tags:
- Java
- OperatorPrecedence
date: 2023-05-18 22:33 +0900
---
## 문제

다음은 Java 코드입니다.

```java
long dividend = 10_000_000_000L;
long divisor = 10L;
int quotient = (int) dividend / divisor;
```

위에서 quotient의 값은 무엇일까요? `1_000_000_000` 일까요?

jshell 에서 연산해본 결과값은 `141_006_540` 입니다.

## 연산 순서

```java
(int) dividend / divisor;
```

위 코드에서는 먼저 dividend 를 int 로 형 변환하게 됩니다. 그럼 `10_000_000_000` 는 int의 최대값인 `2_147_483_647` 를 초과하게 되므로, 오버플로우가 발생합니다.

jshell 에서 출력된 결과값은 `1_410_065_408` 이고, 이를 `10L` 로 나누면 앞서 본, `141_006_540` 이 됩니다.

## 올바른 연산

올바른 연산 결과를 얻기 위해서는 아래와 같이 괄호를 사용하여 나눗셈이 먼저 수행되도록 해야 합니다.

```java
int quotient = (int) (dividend / divisor);
```
아래는 jshell 에서 연산을 수행한 결과입니다.

![jshell에서 연산을 수행한 결과 화면](/assets/img/2023-05-18-java-cast-operator-precedence/01.test-in-jshell.png)

## 연산 우선순위

모두 알고 계시듯 괄호로 둘러 싼 표현식이 가장 먼저 평가가되고, 그 다음은 단항 연산자가 평가됩니다. 그리고 그 다음은 사칙 연산자가 아닌 형 변환(type casting) 연산자가 우선 평가됩니다.

![대학 교재에 나와 있는 연산자 우선순위](/assets/img/2023-05-18-java-cast-operator-precedence/02.oprators-precedence-in-book.png)

출처 : [https://introcs.cs.princeton.edu/java/11precedence/](https://introcs.cs.princeton.edu/java/11precedence/)

뭔가 불안하다 싶으면 괄호로 가독성을 해치지 않는 선에서 묶어주는게 좋겠습니다.

## 잡설

프로그래머스에서 코딩테스트 문제([링크](https://school.programmers.co.kr/learn/courses/30/lessons/87390))를 풀다가 로직이 아무리 생각해도 이상 없는 것 같은데, 계속 틀려서 시간을 많이 소모했습니다.

여태 형 변환을 별로 할 일이 없어서 그런지 형 변환 연산자의 우선순위에 대해서 생각해본적이 없던 것 같습니다. 대체로 아래와 같이 Oracle 웹 사이트에 나와있는 정도의 수준이 보통의 Java 서적에서 볼 수 있는 것이다보니, 추가로 생각해볼 생각을 하지 못한 것으로 보입니다. 저의 부족함이겠죠.

![Oracle 웹 사이트에 나와있는 연산자 우선순위](/assets/img/2023-05-18-java-cast-operator-precedence/03.operators-precedence-in-oracle-webpage.png)

출처: [oracle.com](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html)

코딩테스트 문제를 꾸준히 풀어보는게 지식의 구멍을 채우기에 좋은 것 같습니다.
