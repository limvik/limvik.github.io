---
layout: post
title: Java 8과 Java 11 이후 버전의 Character 클래스 toString() 메서드 차이점
categories:
- Java
tags:
- Java
- Programmers
- CodingTest
date: 2023-06-11 12:29 +0900
---
## Intro

한동안 팀 프로젝트 하느라 정신이 없다가, 이제 좀 마무리가 돼서 코딩테스트 쉬운 옛날 문제도 다시 보고 있습니다.

프로그래머스 시저 암호 문제([링크](https://school.programmers.co.kr/learn/courses/30/lessons/12926))를 풀었는데, 문자열 내의 문자를 특정 수 만큼 다음 위치에 있는 문자를 출력해야 했습니다.

예전에는 if-else 문으로만 풀었었는데, 이번에는 stream으로 풀어봤습니다. 문자열의 [chars()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#chars%28%29) 메서드를 사용하면 문자의 code point를 요소로 갖는 [IntStream](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/IntStream.html)을 반환해서 code point를 다시 String으로 변환할 때 아래와 같이 변환을 했습니다.

```java
String.valueOf((char) codepoint)
```

이유는 모르겠지만 뭔가 좀 별로 마음에 안들어서 문서를 찾아봤더니, Java 11부터는 Character 클래스에 Overloading 된 toString() 메서드가 하나 더 있었습니다.

## Character 클래스

### Java 8

Java 8 의 Character 클래스 문서([링크](https://docs.oracle.com/javase/8/docs/api/java/lang/Character.html))에서 toString() 메서드를 보시면 아래와 같이 두 가지 메서드 밖에 없습니다.

![Java 8 Character 클래스 문서의 toString() 메서드](/assets/img/2023-06-11-difference-of-toString%28%29-method-in-Character-class-between-java8-and-after-java11/2023-06-11-java8-Character-class-doc.png)

boxing 된 char 를 문자열로 반환하거나, char 타입을 인수로 넘기면 String으로 변환하여 줍니다.

### Java 11

Java 11의 Character 클래스 문서([링크](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html))에서 toString() 메서드를 보시면 아래와 같이 하나 더 메서드가 추가된 것을 볼 수 있습니다.

![Java 11 Character 클래스 문서의 toStrign() 메서드](/assets/img/2023-06-11-difference-of-toString%28%29-method-in-Character-class-between-java8-and-after-java11/2023-06-11-java11-Character-class-doc.png)

int 타입의 code point 를 받아서 문자열로 변환해 주는 메서드가 추가됐다는 것을 알 수 있습니다.

## 확인

Java 8 설치하기는 귀찮아서 jdoodle online compiler([링크](https://www.jdoodle.com/online-java-compiler/))를 이용해서 버전 별로 확인해 보았습니다.

간단하게 대문자 A의 code point를 이용하여 출력하는 코드를 작성했습니다.

```java
public class MyClass {
    public static void main(String args[]) {
      System.out.println(Character.toString(65));
    }
}
```

당연한 결과이긴 하지만 Java 8 인 int를 인수로 받는 메서드가 없어 오류가 발생하고, Java 11 부터는 이상 없이 A가 출력됐습니다.

### Java 8 실행 결과

![Java 8 코드 실행 결과](/assets/img/2023-06-11-difference-of-toString%28%29-method-in-Character-class-between-java8-and-after-java11/2023-06-11-java8-toString-by-int.png)

### Java 11 실행 결과

![Java 11 코드 실행 결과](/assets/img/2023-06-11-difference-of-toString%28%29-method-in-Character-class-between-java8-and-after-java11/2023-06-11-java11-toString-by-int.png)

스크린 샷을 첨부는 안했지만 Java 9과 10에서도 오류가 발생하는 걸 보니 Java 11부터 추가된 것으로 보입니다.

## 기타

Java 17 의 Character 클래스 문서([링크](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Character.html))도 한 번 봤는데, Java 버전별로 지원하는 Unicode 버전도 다른 것을 언급하고 있습니다. 혹시나 Unicode 를 직접 조작해야할 일이 있다면 고려해야겠습니다.

![Java 17 Character 클래스 문서의 Unicode Version 정리표](/assets/img/2023-06-11-difference-of-toString%28%29-method-in-Character-class-between-java8-and-after-java11/2023-06-11-java17-unicode-version-in-Character-class-doc.png)

## Outro

이유는 모르겠지만 갑자기 char 타입으로 형 변환 하기가 싫어서 문서를 살펴 보고 글 까지 쓰게 됐습니다.

쓸데없는 짓 한거 같기도 하네요. ㅎㅎ
