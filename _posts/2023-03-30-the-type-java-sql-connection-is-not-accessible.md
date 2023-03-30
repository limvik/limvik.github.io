---
layout: post
title: The type java.sql.Connection is not accessible
categories:
- Java
tags:
- Java
- MySQL
- Module
date: 2023-03-30 20:55 +0900
---
## 요약
Java 9 이상에서는 java.sql 과 같은 다른 모듈을 사용하려면, module-info.java 파일에 사용할 모듈을 선언해 주어야합니다.
```java
module projectname {
	requires java.sql;
}
```
급하면 그냥 module-info.java 파일을 지워버리면 됩니다.

## Intro
오늘 MySQL에 Java를 이용해서 연결하는 수업을 들었습니다. 수업에는 JDK 17을 사용하지만 수업은 Java 8 기준이고, 강사님이 Java 8까지만 익숙하셔서 모듈로 인한 오류를 볼 수 있었습니다.

## The type java.sql.Connection is not accessible
Java 9 이상 부터 모듈화가 되었고, LTS 기준으로는 Java 11 이상의 버전을 사용한다면 다른 모듈 사용 시 module-info.java 에 모듈과 관련된 선언을 해주어야 합니다.

아래는 jdbc 라고 이름 지은 Java 프로젝트의 module-info.java 파일에 java.sql 모듈을 추가한 코드입니다.
```java
module jdbc {
	requires java.sql;
}
```
그냥 module-info.java를 삭제해버려도 동작은 잘 됩니다.

## java.base 는 왜 잘 되나?
java.lang 등이 있는 java.base는 자동으로 module에 추가 됩니다. 아래는 module-info.class 파일의 내용입니다. 위에서 처럼 requires java.sql; 만 추가했는데도 불구하고, 컴파일 후에는 자동으로 requires java.base; 가 들어가 있습니다.

![enter image description here](/assets/img/2023-03-30-java-sql-connection-not-accessible/2023-03-30-java-module-java-base.png)

## Outro
기본서에서 모듈에 대해 읽어볼 때, 모듈화 하는게 쉽지 않겠다고 생각했습니다. 신입 Java 개발자에게 모듈화 능력을 바라지는 않을 것 같아서 모듈화에 대해 공부하는 걸 뒤로 미루고 있지만, Java 개발자라면 언젠가는 공부하고 시도해 봐야하지 않을까 싶습니다.

간단하게나마 살펴보실 분은 오라클 공식 홈페이지에 있는 짧은 듯 짧지 않은 아래 글을 참고해 보시는 것도 좋을 것 같습니다.

[https://www.oracle.com/corporate/features/understanding-java-9-modules.html](https://www.oracle.com/corporate/features/understanding-java-9-modules.html)
