---
layout: post
title: 리스코프 치환 원칙과 다형성의 차이점
categories:
- OOP
tags:
- OOP
- LSP
- Polymorphism
date: 2024-01-07 22:43 +0900
---
## Intro

문득 리스코프 치환 원칙(Liskov's Substitution Principle, LSP)과 다형성(Polymorphism)의 차이가 명확하게 떠오르지 않아서 찾아본 결과를 공유합니다.

## 정의

각각의 정의부터 살펴보겠습니다.

### 리스코프 치환 원칙

밥 아저씨(Robert C. Martin)의 저서 `Agile Software Development, Principles, Patterns, and Practices(한글판: 클린 소프트웨어)`를 보면 아래와 같이 정의합니다.

> SUB TYPES MUST BE SUBSTITUTABLE FOR THEIR BASE TYPES.
>
> SUB TYPES 는 반드시 BASE TYPES로 치환할 수 있어야 합니다.

같은 책 바로 아래에 리스코프 교수의 원문도 언급하고 있습니다.

> What is wanted here is something like the following substitution property: If for each object o_1 of type S there is an object o_2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when o_1 is substituted for o_2 then S is a subtype of T.
>
> 여기서 원하는 것은 다음과 같은 치환 속성과 같은 것입니다: S 타입의 각 객체 o_1에 대해 T 타입의 객체 o_2가 존재하고, T에 대해 정의된 모든 프로그램 P에 대해 o_1이 o_2로 치환될 때 P의 동작이 변경되지 않는다면 S는 T의 Sub Type입니다.

### 다형성

다형성은 객체 지향 프로그래밍의 개념으로 변수(Variable), 함수(Function) 또는 객체(Object)가 여러 형태를 취할 수 있는 능력을 의미합니다.

Compile time 다형성의 예로 오버로딩(Overloading), Runtime 다형성의 예로 오버라이딩(Overriding)이 있습니다.

## 분류 상의 차이

리스코프 치환 원칙은 객체 지향 설계 원칙인 SOLID에서 L을 담당하고 있습니다. 반면, 다형성은 객체 지향 패러다임의 특성 중 하나로 오버로딩(Overloading)과 오버라이딩(Overriding) 같은 코드를 구현하는 방식이라 할 수 있습니다.

리스코프 치환 원칙이 `설계` 시에 지켜져야 할 `원칙`이라면, 다형성은 객체 지향 패러다임에서 `구현`을 통해 나타나는 `특성`입니다.

따라서 리스코프 치환 원칙을 지키지 않는다고 해서 프로그램에 오류가 발생하지는 않지만, 다형성은 프로그래밍 언어의 문법에 맞추어 제대로 구현하지 않을 경우 프로그램에서 오류가 발생할 수 있습니다.

## 예제

### 다형성을 이용했지만, 리스코프 치환 원칙은 위반한 경우

타조(Ostrich)는 새(Bird)이기는 하지만, 날 수는 없는 대표적인 새 입니다. 이러한 경우를 이용한 Java 예제를 살펴보겠습니다.

```java
class Bird {
    public void fly() {
        System.out.println("This bird can fly.");
    }
}

class Sparrow extends Bird {
    // Sparrow는 Bird를 상속받아 fly 메서드 사용
}

class Ostrich extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Ostrich cannot fly");
    }
}

public class Main {
    public static void testBird(Bird bird) {
        bird.fly();
    }

    public static void main(String[] args) {
        Sparrow sparrow = new Sparrow();
        Ostrich ostrich = new Ostrich();

        testBird(sparrow);
        testBird(ostrich); // 예외 발생: 리스코프 치환 원칙 위반
    }
}
```

객체 지향 프로그래밍의 다형성을 이용하여 `testBird()` 메서드의 인수로 부모 클래스인 Bird를 인수로 받아 `fly()` 메서드를 호출합니다.

그런데 타조(Ostrich)는 날 수 없으므로, fly 라는 행위를 할 수 없어 예외가 발생합니다. Base Type에서는 Sub Type이 fly라는 행위를 수행하는 것으로 설계하였지만, `설계와 다른 상황이 발생`한 것입니다. 이는 Sub Type을 Base Type으로 치환할 수 없으므로, 리스코프 치환 원칙을 위반했음을 의미합니다.

다음으로, 위 예제를 수정하여 다형성을 이용하면서, 리스코프 치환 원칙을 준수한 경우를 살펴보겠습니다.

### 다형성을 이용하면서, 리스코프 치환 원칙을 준수한 경우

모든 새가 날 수 있는 것은 아니므로, Bird 클래스에서 fly 라는 행위는 공통 행위 혹은 동작이 아닙니다. 따라서 날 수 `있`는 새(FlyingBird)와 날 수 `없`는 새(NonFlyingBird)로 나누어 fly는 날 수 `있`는 새만 수행하도록 합니다.

```java
abstract class Bird {
    // 공통 특성 및 동작
    public abstract void eat();
}

abstract class FlyingBird extends Bird {
    // 날 수 있는 새에 해당하는 특성 및 동작
    public void fly() {
        System.out.println("This bird can fly.");
    }
}

abstract class NonFlyingBird extends Bird {
    // 날 수 없는 새에 해당하는 특성 및 동작
}

class Sparrow extends FlyingBird {
    @Override
    public void eat() {
        System.out.println("Sparrow is eating.");
    }
}

class Ostrich extends NonFlyingBird {
    @Override
    public void eat() {
        System.out.println("Ostrich is eating.");
    }
}

public class Main {
    public static void testBird(Bird bird) {
        bird.eat();
        if (bird instanceof FlyingBird flyingBird) {
            flyingBird.fly();
        }
    }

    public static void main(String[] args) {
        Bird sparrow = new Sparrow();
        Bird ostrich = new Ostrich();

        testBird(sparrow);
        testBird(ostrich);
    }
}
```

이렇게 수정함으로써, Sub Type을 Base Type으로 치환할 수 있고, LSP를 만족하게 됩니다.

하지만 이 방법이 유일한 방법은 아닙니다.

위에서 언급한 밥 아저씨의 책과 『디자인 패턴의 아름다움』이라는 책을 보면, LSP를 설명하기 위해서 `계약에 따른 설계(Design By Contract)`에 대해 이야기합니다. 이에 대해 『디자인 패턴의 아름다움』에 나온 내용은 아래와 같습니다.

> ... 하위 클래스를 설계할 때는 상위 클래스의 `동작 규칙`을 따라야 한다. 상위 클래스는 함수의 `동작 규칙`을 정의하고 하위 클래스는 함수의 내부 구현 논리를 변경할 수 있지만 함수의 원래 `동작 규칙은 변경할 수 없다`. 여기서 말하는 `동작 규칙`에는 함수가 구현하기 위해 선언한 것, 입력, 출력, 예외에 대한 규칙, 주석에 나열된 모든 특수 사례 설명이 포함된다. 사실 여기에서 언급된 상위 클래스와 하위 클래스 간의 관계는 인터페이스와 구현 클래스 간의 관계로 대체될 수도 있다. p.137 3.3.3 리스코프 치환 원칙을 위반하는 안티 패턴

글을 보면 예외에 대한 규칙, 주석도 언급하고 있으므로 이를 통해 간단하게 해결할 수도 있습니다. 상위 클래스의 주석에 날지 못하는 새에 대한 구현도 필요하다고 언급을 하거나, 날지 못하는 새의 경우 예외를 발생시키는 것을 명시할 수도 있습니다.

저는 위와 같은 상황에서는 별도의 클래스로 명확하게 분리해 주는 것이 유지보수하기에 더 적합하다고 생각하는데, 개인이라면 일관성있게, 팀 이라면 규칙을 만드는게 중요할 것이라 생각됩니다.

## Outro

다형성은 코드를 구현하는 방식으로써 언어에 따라 문법은 조금씩 다르게 나타날 수도 있지만, 리스코프 치환 원칙은 설계 원칙으로써 상위 클래스에서 하위 클래스의 설계 방식을 설명하는 추상적인 개념임을 알 수 있었습니다.

저와 같은 혼란을 겪은 분들께 도움이 되었으면 합니다.

## 참고 자료

- [principle wiki](http://principles-wiki.net/principles:liskov_substitution_principle)
- [Educative - What is polymorphism](https://www.educative.io/answers/what-is-polymorphism)
- [디자인 패턴의 아름다움](https://product.kyobobook.co.kr/detail/S000202093794)
- [클린 소프트웨어](https://product.kyobobook.co.kr/detail/S000001875106)
