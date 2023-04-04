---
layout: post
title: 객체지향개발 연습4 - 현황판, 필기구 객체 구현
categories:
- OOP
tags:
- OOP
- Java
date: 2023-04-04 22:25 +0900
---
## Intro
[이전 글](https://limvik.github.io/posts/oop-practice-implements-user-object/)에서 사용자 객체를 구현하다가 새롭게 식별하게 된 현황판(Board) 객체를 구현해 보려고 합니다. 그리고 남은 시간에는 필기구(Pen) 객체까지 구현해 보았습니다.

```java
public class Board {

    private List<Deck> decks; // 보관함 목록
    private List<Plan> plans; // 학습 일정 목록

    // 보관함 목록 및 보관함 별 학습 대상 카드의 개수를 보여라
    public void showBoard() { }

}
```
## 현황판 객체 구현

먼저 사용자 객체로부터 메시지를 수신한 후 보여줄 데이터를 불러옵니다.

### 보관함 목록 및 학습 일정 불러오기

먼저 간단하게 아래와 같이 보관함과 플래너 객체의 인스턴스를 임시로 생성하게 하였습니다.

```java
public class Board {

    private List<Deck> decks; // 보관함 목록
    private List<Plan> plans; // 학습 일정 목록

    // 보관함 목록 및 보관함 별 학습 대상 카드의 개수를 보여라
    public void showBoard() {

        // 보관함 목록 불러오기
        decks = new Deck().getDeckList();

        // 오늘 학습 일정 불러오기
        LocalDate today = LocalDate.now();
        plans = new Planner().searchStudyPlan(today, today);

    }

}
```
하지만 사용자 객체는 현황판에 있는 정보를 바탕으로 보관함 객체와 상호작용 하고, 학습 후에는 플래너 객체에 학습 일정 기록도 해야하므로 이전에 인터페이스를 통해 아무것도 받지 않도록 한 생각이 잘못되었음을 알게됐습니다.

그래서 아래와 같이 수정하였습니다.

```java
public class Board {

    private List<Deck> decks; // 보관함 목록
    private List<Plan> plans; // 학습 일정 목록

    // 보관함 목록 및 보관함 별 학습 대상 카드의 개수를 보여라
    public void showBoard(Deck deck, Planner planner) {

        // 보관함 목록 불러오기
        decks = deck.getDeckList();

        // 오늘 학습 일정 불러오기
        LocalDate today = LocalDate.now();
        plans = planner.searchStudyPlan(today, today);

    }

}
```

그리고 메시지를 보내는 사용자도 수정하였습니다.

```java
public class User {

    // 학습하라
    public User(Board board, Deck deck, Planner planner) {
        
        // 보관함 목록 및 보관함 별 학습 대상 카드 개수 확인하기
        board.showBoard(deck, plan);

    }

}
```
그럼 이제 어떻게 수신한 정보를 보여줄 것인가에 대한 세부 구현은 세부 구현을 할 때 하기로 하고, 생각보다 금방 끝나버려서 존재에 의문이 들어 짧게 끝날 것 같은 필기구 객체를 구현하기로 했습니다.

## 필기구 객체 구현

### 기존 인터페이스 정리

필기구 객체 인터페이스를 정리했던 결과는 아래와 같습니다.
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
### 학습 일정 전달 구현

역시나 필기구 객체의 존재 이유에 대한 의문이 드는 구현입니다. 일단 진행해 보자면, 사용자에게 돌려주어야 할 정보를 뜬금없이 Plan으로 해놓아서 boolean 형으로 수정하였습니다. 이전에 인터페이스 설계 글([링크](https://limvik.github.io/posts/oop-excercise-for-design-interface/#%EC%95%84%EC%89%AC%EC%9A%B4-%EC%A0%90))에서 써놓았던 대로 협력 설계 시에 주고받는 요소를 표시해놓지 않은 영향으로 판단됩니다.

```java
public class Pen {

    // 내용을 전달하라(학습 일정)
    public boolean sendContents(Planner planner, Plan plan) {
        return planner.writePlan(plan);
    }
    // 나머지 생략

}
```

플래너 객체를 다시 확인해보면 모두 데이터베이스와 상호작용하므로, 플래너 객체는 세부 구현만 남아 별도의 구현 글을 작성할 필요는 없을 것으로 보입니다.

```java
public class Planner {

    // 학습일정을 찾아라
    public List<Plan> searchStudyPlan(LocalDate startDate, LocalDate endDate) {
        // 데이터베이스에서 불러오기
    }

    // 학습일정을 기록하라
    public boolean writePlan(Plan plan) {
        // 데이터베이스에 저장
    }

}
```

### 보관함 이름 전달 구현

마찬가지로 반환형을 수정하였습니다.

```java
public class Pen {
	
    // 학습 일정 전달, 학습 내용 전달 생략
    // 내용을 전달하라(보관함 이름)
    public boolean sendContents(Deck deck, int id, String name) {
        return deck.modifyDeckName(id, name);
    }

}
```

보관함 객체도 플래너 객체와 마찬가지로 데이터베이스와 상호작용하여 지금 구현할 것은 없습니다.

```java
public class Deck {

    // 나머지 생략
    // 이름을 기록하라
    public boolean modifyDeckName(int id, String name) {
        // 데이터베이스 수정
    }
    
}
```

### 학습 내용 전달 구현

마지막으로 학습 내용 전달 시에도 반환형을 수정하고, 카드 id만 전달 받으면 카드를 다시 찾아야 하므로, 카드 인스턴스를 받는 것으로 수정하였습니다.

```java
public class Pen {

    // 나머지 생략
    // 내용을 전달하라(학습 내용)
    public boolean sendContents(Card card, String front, String back) {
        card.setFront(front);
        card.setBack(back);
        return card.writeContents(card);
    }

}
```

그리고 카드 객체에는 필기구 객체와의 협력에 맞춰 식별자, 앞/뒷면 내용에 대한 속성과 앞/뒷면 setter 를 추가하였습니다. 기록을 할 때는 데이터베이스와 상호작용 해야하므로 현재는 구현하지 않았습니다.

```java
public class Card {

    private int id; // 카드 식별자
    private String front; // 카드 앞면 내용
    private String back; // 카드 뒷면 내용
    
    // 카드를 생성하라
    public Card() { }
    
    // 학습내용을 기록하라
    public boolean writeContents(Card card) {
        // 데이터베이스에 기록
    }
    
    // 카드 정보를 찾아라
    public Card findCardContents(int cardId) { }
    
    // 카드 앞면 내용 설정
    public void setFront(String front) {
        this.front = front;
    }
    
    // 카드 뒷면 내용 설정
    public void setBack(String back) {
        this.back = back;
    }

}
```
존재 이유에 대한 의문이 있기는 하지만, 필기구 객체 구현도 마무리 되었습니다.

```java
public class Pen {

    // 내용을 전달하라(학습 일정)
    public boolean sendContents(Planner planner, Plan plan) {
        return planner.writePlan(plan);
    }

    // 내용을 전달하라(보관함 이름 수정)
    public boolean sendContents(Deck deck, int id, String name) {
        return deck.modifyDeckName(id, name);
    }

    // 내용을 전달하라(학습 내용)
    public boolean sendContents(Card card, String front, String back) {
        card.setFront(front);
        card.setBack(back);
        return card.writeContents(card);
    }

}
```

## Outro

시간이 남아서 더 하다가 너무 길어졌네요.

여튼, 이렇게 필기구 객체 구현까지 마무리되고 나니 사용자 객체가 수신해야 할 메시지가 `카드를 추가하라`, `카드 내용을 수정하라`, `보관함 이름을 수정하라` 등 다양하게 존재하겠구나 라는 생각이 떠오릅니다. 점점 바보가 되가는 기분이 듭니다.

다음 글에서는 사용자 객체가 수신해야 할 추가 식별된 메시지를 추가하고, 보관함 객체와 카드 객체도 마무리할 수 있으면 해봐야겠습니다.
