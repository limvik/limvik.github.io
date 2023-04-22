---
layout: post
title: JUnit5 테스트 순서 지정하는 방법
categories:
- Java
- JUnit5
tags:
- Java
- JUnit5
date: 2023-04-22 13:33 +0900
---
## Intro

DAO(Data Access Object)에 CRUD 수행하는 코드를 만들면서 테스트 코드를 삽입, 조회, 수정, 삭제 순으로 순서를 지정해서 수행하기 위해 간편하게 chat GPT에 물어봤더니, 테스트 메서드(Method)에 @Order(숫자) 와 같이 Annotation 붙여주면 된다고 해서 그대로 했다가 오류를 못잡아서 쓸데 없이 시간 낭비를 했습니다. 코드에 이상이 없는 걸 확신하고 나서야 뒤늦게 JUnit5 공식 문서([링크](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-execution-order))를 확인하고 왜 그랬나 싶었습니다.

쓰다보면 또 한참 길어질 내용이 많아 보여서, 저와 같은 초보분들을 위해 간단한 사용법만 작성해 봤습니다.

## Test 실행 순서 지정

Test 실행 순서 지정하는 방법으로는 크게 Method Order와 Class Order 를 지정하는 방법이 있습니다. 그리고 숫자를 이용하거나 랜덤한 방식으로 하는 등 세부적으로 순서 지정 방식이 나뉩니다. 세부 방식은 Method Order에 곧 사라질 예정인 Alphanumeric 방식을 제외하고는 Method Order와 Class Order 모두 동일합니다.

이중에 숫자로 순서를 지정하는 방식만 살펴보겠습니다.

### Method Order

테스트 메서드의 순서를 지정하는 방식으로 6.0에서 제거될 예정이라는 Alphanumeric 순을 제외하고, 4 가지 방식으로 순서를 지정할 수 있습니다. 아래는 이중에 숫자로 순서를 지정하는 OrderAnnotation 예제입니다.

>MethodOrderer.OrderAnnotation: sorts test methods numerically based on values specified via the @Order annotation

```java
import org.junit.jupiter.api.MethodOrderer.OrderAnnotation;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestMethodOrder;

@TestMethodOrder(OrderAnnotation.class)
class OrderedTestsDemo {

    @Test
    @Order(1)
    void nullValues() {
        // perform assertions against null values
    }

    @Test
    @Order(2)
    void emptyValues() {
        // perform assertions against empty values
    }

    @Test
    @Order(3)
    void validValues() {
        // perform assertions against valid values
    }

}
```
보자마자 적용할 수 있을 정도로 굉장히 간단합니다. 그런데 클래스 위의 `@TestMethodOrder(OrderAnnotation.class)` 를 놓치기 쉽습니다. 저는 @Order 만 메서드에 적용하고 나서, 코드 로직의 문제인줄 알고 시간을 한참 보냈습니다.

### Class Order

Class Order는 테스트 클래스 내의 내부 클래스에 대해 테스트 순서를 적용할 때 적용합니다. 세부 순서 지정 방식은 Method Order 와 클래스 이름만 다를뿐 동일합니다. Class Order 에다가 Method Order 에 사용하는 클래스를 지정하지 않도록 주의하기는 해야겠습니다.

```java
import org.junit.jupiter.api.ClassOrderer;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestClassOrder;

@TestClassOrder(ClassOrderer.OrderAnnotation.class)
class OrderedNestedTestClassesDemo {

    @Nested
    @Order(1)
    class PrimaryTests {

        @Test
        void test1() {
        }
    }

    @Nested
    @Order(2)
    class SecondaryTests {

        @Test
        void test2() {
        }
    }
}
```
메서드가 아닌 클래스에 @Order Annotation을 붙여주고, `@Nested` Annotation도 함께 붙여주어야 합니다.

## Outro

저와 같은 개발 초보분들이 쓸데 없는데 시간 버리지 않기를 바라며 작성했습니다. chat GPT 소설 쓰는거에 주의한다고 하면서도 어느새 많이 의존하고 있었나봅니다. 공식 문서에 더 친해지도록 노력하는게 좋겠습니다.
