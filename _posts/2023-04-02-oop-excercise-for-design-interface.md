---
layout: post
title: 객체지향개발 연습2 - 인터페이스 설계
categories:
- OOP
tags:
- Java
- OOP
date: 2023-04-02 19:08 +0900
---
## Intro
처음 객체지향개발 도전([링크](https://limvik.github.io/posts/%EC%9E%91%EC%9D%80-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EB%A1%9C-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%EC%97%B0%EC%8A%B5%ED%95%98%EA%B8%B0/))한 것이 망한 이후에, 플래시 카드 학습에 대한 도메인 모델만 그려봤었습니다([링크](https://limvik.github.io/posts/%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5%EA%B0%9C%EB%B0%9C-%EC%97%B0%EC%8A%B51-%EB%8F%84%EB%A9%94%EC%9D%B8-%EB%AA%A8%EB%8D%B8-%EA%B7%B8%EB%A6%AC%EA%B8%B0/)).

그런데 마음만 앞서서 도메인 모델을 그린 후에야 기능적 요구사항을 생각 안했다는걸 깨달았습니다.

그래서 도메인 모델을 만들 때 어떤 기능을 생각했었는지 역으로 다시 추적하고, 인터페이스 설계까지 해봤습니다.

## 요구사항
![플래시 카드 도메인 모델](https://limvik.github.io/assets/img/2023-03-27-domain-model-for-flashcards.png)

1. 사용자가 보관함을 추가, 변경, 삭제할 수 있어야 한다.
2. 작은 보관함이 위치하는 큰 보관함을 변경할 수 있어야 한다.
3. 사용자가 카드를 추가, 변경, 삭제할 수 있어야 한다.
4. 카드가 위치하는 보관함을 변경할 수 있어야 한다.
5. 사용자가 학습한 카드에 대한 학습 기록을 남길 수 있어야 한다.
6. 사용자가 원하는 카드를 검색할 수 있어야 한다.

## 협력 설계 및 인터페이스 정리하기

### 협력 설계
먼저 협력을 설계하기 위해 도메인 모델을 그리면서 식별된 객체 간에 기능을 구현하기 위해 주고 받아야 할 메시지를 그려봤습니다.

![협력 설계](/assets/img/2023-04-02-oop-excercise-for-design-interface/2023-04-02-collaboration-design.svg)

이전에 도메인 모델만 그렸을 때는 사용자와 카드가 직접 연관 관계를 갖는게 맞나 고민했지만, 협력을 설계하면서 일반 종이를 먼저 카드로 만드는 것은 사용자이므로 사용자가 '카드를 생성하라'라는 메시지를 보내 카드를 만들도록 하였습니다.

### 인터페이스 정리하기

각 객체가 수신하는 메시지를 바탕으로 객체의 인터페이스를 정리하였습니다.

### 사용자
```java
public class User {
    // 학습하라
    public void startStudy(List<Plan> plan, List<Deck> decks, List<Card> cards) { }

}
```
### 보관함
```java
public class Deck {
    // 보관함을 관리하라(생성)
    public Deck(String deckName) { }
    
    // 보관함을 관리하라(수정)
    public boolean changeParentDeck() { }
    
    // 보관함을 관리하라(삭제)
    public boolean removeDeck(int deckId) { }
    
    // 카드를 분류하라
    public boolean classifyCards(int deckId, int cardId) { }
    
    // 카드를 찾아라(키워드 검색)
    public List<Card> findCards(String keywords) { }
    // 카드를 찾아라(플래너를 통해 card의 식별자를 알고 있는 경우)
    public List<Card> findCards(List<Integer> cardIdList) { }
    
    // 이름을 기록하라
    public boolean modifyDeckName(int deckId, String newDeckName) { }
}
```
### 카드
```java
public class Card {
    // 카드를 생성하라
    public Card() { }
    // 학습내용을 기록하라
    public boolean writeContents(Card card) { }
    // 카드 정보를 찾아라
    public Card findCardContents(int cardId) { }

}
```
### 플래너
```java
public class Planner {
    // 학습일정을 찾아라
    public List<Plan> searchStudyPlan(Date startDate, Date endDate) { }
    // 학습일정을 기록하라
    public boolean writePlan(Plan plan) { }

}
```
### 필기구
```java
public class Pen {
    // 내용을 전달하라(학습 일정)
    public Plan sendContents(Plan plan) { }
    // 내용을 전달하라(보관함 이름)
    public String sendContents(int deckId, String deckName) { }
    // 내용을 전달하라(학습 내용)
    public Card sendContents(int cardId, String front, String back) { }

}
```
### 기타
이미 앞에서 사용하여 확인하셨겠지만, 학습 일정을 개별적으로 다루기 위해서 Plan 클래스를 별도로 생성하였습니다.
```java
public class Plan {

}
```
## 아쉬운 점
뭔가 빠트린 기분도 들고, 필기구 객체는 정말 전달 말고는 아무것도 안하는데 빼는게 맞는건가 싶기도 합니다. 찜찜한 기분이 들지만, 책에서 배웠던 대로 `구현을 하면서 수정`해 가기로 하였습니다.

그리고 협력 설계를 몇 일 전에 미리 했었는데, 앞서 보셨던 대로 메시지 송수신 시 주고 받는 요소들도 표시를 하지 않았었습니다.

![협력 설계](/assets/img/2023-04-02-oop-excercise-for-design-interface/2023-04-02-collaboration-design.svg)

그래서 인터페이스 정리하면서 `뭘 주고 받아야되는지 다시 고민하느라 시간을 허비`했습니다. 아마 당시에는 메시지를 보면 뭘 주고 받아야 할지 기억이 날 거라 생각을 했던 것 같습니다.

사람의 기억력을 신뢰하면 안되는데, 종종 타협을 하게 되는 것 같습니다.

## Outro

처음 객체지향개발을 작게 도전한 것을 글로 쓸 때, 했던 일을 모두 한꺼번에 기록하느라 부담되기도 하고, 아쉬웠던 점에 대해 기억이 안나서 다 적지도 못했었습니다. 그런데 이렇게 했던 일을 짧은 단위로 기록하니까 부담도 적어지고, 기억이 휘발되지도 않아서 기록하기도 좋은 것 같습니다. 구현되는 기능이 별로 없어도 초보자에게는 사소한 것 하나까지 다 새롭고 배울것들이기 때문인가 봅니다.

다음 글도 객체 별로 끊어서 작성하던지 기준을 세우고 각각 별도의 글로 작성해 봐야겠습니다.
