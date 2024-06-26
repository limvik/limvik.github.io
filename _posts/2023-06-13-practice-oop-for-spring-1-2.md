---
layout: post
title: Spring을 위한 객체 지향 연습 1-2. 시작하자마자 수정
categories:
- OOP
tags:
- OOP
date: 2023-06-13 21:00 +0900
---
## Intro

[이전 글](https://limvik.github.io/posts/study-oop-process-for-spring/)에서 API 테이블까지 그려봤는데, Spring을 사용해봐서 그런지 생성/수정/삭제가 플래시카드라는 도메인 객체에서 처리해야 할 일인가에 대한 의문이 생겼습니다.

## 이전에 작업한 코드와 Spring의 Service 클래스 비교

이전에 Java 만 배운 상태에서는 아래와 같이 나름대로 만든 도메인 객체에서 바로 DAO(Data Access Object) 클래스를 통해 데이터베이스에 접근했습니다.

```java
public class Card {

...
    
    // 학습내용을 보관하라
    public boolean writeContents(int userId, int deckId) {
        // 데이터베이스에 기록
        try {
            var cardDAO = new CardDAO(DatabaseConnection.getInstance().getConnection());
            return cardDAO.insertCard(userId, deckId, this) == 1;
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
        return false;
    }

...

```

하지만 Spring에서는 Service를 통해 데이터베이스에 접근합니다. 간단한 예제를 보자면 아래와 같습니다.

```java
@Service
public class UserService {
	@Autowired
    private final UserRepository userRepository;

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
}
```

뿐만 아니라 Spring MVC를 배우면서 `Service 클래스`에서 `비즈니스 로직`을 처리한다고 배웠습니다.

그런데 도메인 모델을 설계하는 이유가 여러가지가 있겠지만, 도메인 모델을 설계하고 `도메인 객체`를 사용함으로써 복잡한 `비즈니스 로직`을 캡슐화하는 것으로 배웠습니다.

여기서 지식 충돌이 발생하니 진행하기가 어려웠는데, 다행히 기존에 있던 책(『디자인 패턴의 아름다움』)에 관련된 내용이 있었습니다.

## 빈약한 도메인 모델 vs 풍성한 도메인 모델

이에 대한 설명은 '객체지향 프로그래밍처럼 보이지만 실제로는 절차적 프로그래밍'이라는 섹션의 하위 섹션에 있는 내용입니다. 빈약한 도메인 모델이 절차적 프로그래밍 스타일이기 때문인데, 그 전에 MVC 모델을 간단하게 살펴봅니다.

### MVC(Model-View-Controller)

MVC 모델은 이름 그대로 `Model` 계층, `View` 계층, `Controller` 계층으로 나누어 개발하지만, 프론트엔드와 백엔드가 분리되면서 백엔드에서는 `Controller` 계층, `Service` 계층, `Repository` 계층으로 나누는 경향이 있습니다. Controller 계층은 프론트엔드에 인터페이스를 노출시키고, Service 계층은 비즈니스 로직을 담당합니다. 그리고 Repository 계층은 데이터 처리를 담당합니다.

### 빈약한 도메인 모델(anemic domain model)

MVC 모델의 각 계층에서는 메서드 없이 데이터만 갖는 VO(View Object), BO(Business Object), Entity 클래스를 정의할 수 있습니다. 이 DTO(Data Transfer Object)는 데이터와 해당 데이터에 대한 연산을 한 곳에 통합하여 캡슐화하는 객체 지향과는 거리가 멀고, 데이터와 프로세스를 분리하는 절차적 프로그래밍 스타일입니다.

이렇게 비즈니스 로직이 포함되지 않은 DTO를 빈약한 도메인 모델이라고 하며, 이를 이용한 개발 방식을 '빈약한 도메인 모델 기반의 개발 방식'이라고 책에서 소개하고 있습니다. 그리고 빈약한 도메인 모델 기반의 개발 방식이 Spring 을 이용한 백엔드 개발 시 흔히 사용하는 Service 클래스에서 비즈니스 로직을 처리하는 방식입니다.

지금까지 DTO를 보면서 했던 제 생각은 'record 가 등장한 것 처럼, 데이터만 있는 데이터 클래스가 필요할 수도 있는 것 아닌가?' 였는데, DTO를 객체 지향 스타일을 위반한 것으로 볼 수 있다니 역시나 아직 갈 길이 멀다는 느낌이 듭니다.

### 풍성한 도메인 모델(rich domain model)

빈약한 도메인 모델에서는 DTO 클래스에서 데이터만 포함하고 Service 클래스에서 비즈니스 로직을 포함하는 방식이었다면, 풍성한 도메인 모델에서는 Service 계층의 BO 클래스를 도메인 클래스로 구성하고 비즈니스 로직을 도메인 클래스에 포함시킵니다.

위에서 BO 클래스만 언급을 했는데, 그 이유는 나머지 VO나 Entity는 빈약한 도메인 모델로 설계하는 것이 합리적이기 때문이라고 책에서 소개합니다. VO 같은 경우 주로 다른 시스템으로 데이터를 보내기 위한 인터페이스의 데이터 전송 캐리어로 사용되므로 비즈니스 논리를 포함하지 않으며, Entity는 데이터베이스의 데이터가 저장되어 Service 계층에 전달되자마자 BO 또는 도메인 클래스로 변환되므로 수명 주기가 짧아 수정될 여지가 없기 때문입니다.

책에서는 이러한 개발 방식을 '풍성한 도메인 모델에 기반한 DDD(Domain Driven Design) 개발 방식'이라 소개하고 있습니다. DDD가 붙은 이유는 DDD를 통해 합리적인 도메인 설계를 얻기 위함입니다. DDD 자체가 중요한 것은 아니기 때문에 책에서도 한 페이지 조금 안되는 분량으로 간단하게 언급하였습니다.

### 정리

책에서 여러가지 차이점을 소개하고 있지만, 제가 해결하고자 했던 의문점은 해소되었기 때문에 여기서 정리하겠습니다.

책의 저자분은 풍성한 도메인 모델은 복잡한 비즈니스 로직을 다룰 때 적합하고, 단순한 비즈니스 로직이라면 기존의 빈약한 도메인 모델을 사용하는 것 또한 합리적인 선택임을 이야기 합니다. 그리고 조금 더 큰 그림에서 보자면 객체 지향적이냐 절차적이냐가 중요한게 아니라 궁극적인 목표는 유지하기 쉽고, 읽기 쉽고, 재사용이 쉬우며, 확장하기 쉬운 고품질 코드를 작성하는 것임을 언급하고 있습니다.

객체 지향 프로그래밍 시에 함수형 프로그래밍을 사용하듯 절차적 프로그래밍 스타일도 객체 지향 프로그래밍을 하면서 필요할 때 사용할 수 있는 요소라고 생각하는게 좋겠습니다.

## 그럼 내 API 테이블은 어떻게 수정되어야 할까?

책의 저자 분의 스타일로 VO 클래스와 Entity 클래스 그리고 비즈니스 로직이 포함된 도메인 클래스를 사용하는 방식을 따라해본다면 Flashcard 도메인 클래스에는 Constructor와 Getter 말고는 필요가 없습니다. 테이블은 생략...!

나중에 기능을 추가한다면 복습 간격을 계산하는 로직이 포함될 수 있겠습니다. 어쨌거나 현재는 생성/수정/삭제를 하는데 Flashcard 도메인 클래스가 데이터만 포함된 BO 클래스가 되어도 상관없습니다.

다음 단계는 도메인 모델보다는 데이터베이스에 연결하고 작업하는 것을 주로 고려하게 되겠습니다.

## 지금의 나는 도메인 모델 설계를 안하는게 나을까?

문득 도메인 모델을 설계하고 하는게 DDD 개발 방식을 기반으로 진행을 하는 것이라면, DDD를 잘 알지도 못하는데 안하는게 나을까 싶은 생각도 듭니다.

하지만 어설퍼도 도메인 설계를 해보는게 이후에 DDD를 공부하고 내가 뭘 잘못하고 있었는지 파악하기도 쉽고, 도메인 모델을 데이터베이스 설계(AWS 문서에 따르면 [개념적 데이터 모델을 도메인 모델이라고도 합니다.](https://aws.amazon.com/ko/what-is/data-modeling/#:~:text=%EA%B0%9C%EB%85%90%EC%A0%81%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%AA%A8%EB%8D%B8%EC%9D%84%20%EB%8F%84%EB%A9%94%EC%9D%B8%20%EB%AA%A8%EB%8D%B8%EC%9D%B4%EB%9D%BC%EA%B3%A0%EB%8F%84%20%ED%95%A9%EB%8B%88%EB%8B%A4.))시에 활용하기도 좋아서 계속 해볼 생각입니다.

## Outro

논외로 지금까지 저의 시행착오에 대해 생각해보면 도메인 모델을 시스템에 포함된 일부분으로 봐야하는데, 도메인 모델을 시스템 자체로 보고있었기 때문이라 생각됩니다. 도메인 모델에서는 뷰(View)가 어떻고 데이터베이스에는 어떻게 저장할 것인지 등을 고려하지 않는데, 도메인 모델에서 모든걸 해결해 보려다가 지금까지의 시행착오가 있었던 것으로 판단됩니다.

시행착오 과정에 시간이 많이 소모되는 것 같은데, 문제 정의할 때 생각했던 모르거나 잘못알고 있던 지식을 발견하는 시간을 줄이는게 지금 저한테 가장 필요한 일이었네요.
