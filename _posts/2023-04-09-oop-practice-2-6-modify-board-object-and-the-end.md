---
layout: post
title: 객체지향개발 연습2-6 - 현황판 객체 수정 및 연습 마무리
categories:
- OOP
tags:
- OOP
- Java
date: 2023-04-09 20:45 +0900
---
## Intro
[이전 글](https://limvik.github.io/posts/oop-practice-2-5-modify-something-wrong/)에서 대격변이 있었고, 다시 첫 메시지(`사용자 목록을 보여라`)를 받는 객체로 현황판(Board) 객체를 선택하여 이에 맞게 수정하고, 영향을 받는 객체들을 일부 수정한 후 이번 두번째 객체지향 연습에 대한 글 작성을 마무리하였습니다.

## 현황판 객체 수정

### 기존 현황판 객체 구현 상태

기존 현황판 객체는 사용자 객체로부터 요청을 받아 보관함 목록을 보이도록 했었습니다.

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
### 현황판 객체 구현 수정

사용자 선택을 위한 첫 메시지인 `사용자 목록을 보여라` 를 현황판 객체가 수신하여야 합니다. 이를 위해 아래와 같은 사용자 목록을 위한 속성과 메서드(method)를 추가하였습니다.
```java
private List<User> users; // 사용자 목록

// 사용자 목록을 보여라
public void showUserList() {

    // 사용자 목록 불러오기
    users = /* 데이터베이스 호출 */;

    // 사용자 목록 시현하기
        
}
```

기존 showBoard() 메서드도 아래와 같이 수정하였습니다. 선택된 사용자 정보를 사용하도록 수정하였고, 하나의 보관함에서 보관함 목록 전체 정보를 얻어온다는 것이 맞지 않다고 생각되어 데이터베이스에서 불러올 수 있도록 수정하였습니다.

```java
// 보관함 목록 및 보관함 별 학습 대상 카드 갯수를 보여라
public void showBoard(int userSequenceNumber, Planner planner) {

    // 선택된 사용자 정보 식별
    User user = users.get(userSequenceNumber);

    // 사용자의 보관함 전체 목록 불러오기
    decks = /* 데이터베이스 호출 */;

    // 오늘 학습 일정 불러오기
    LocalDate today = LocalDate.now();
    plans = planner.searchStudyPlan(user.getId(), today, today);

    // 보관함 목록 및 보관함 별 학습 대상 카드 갯수 시현하기

}
```
그리고 보관함, 학습 일정, 사용자 목록 상태 수정을 위한 setters를 추가하였지만, 글에는 별도로 작성하지 않았습니다.

## 보관함 객체 수정

현황판 객체로부터 수신하던 메시지가 사라졌으므로, 해당 메서드를 삭제하고, 일부 인터페이스를 수정하였습니다.

```java
public class Deck {

    private int id; // 보관함 식별자
    private String name; // 보관함 이름
    private int parentId; // 부모 보관함 식별자
    private List<Integer> childIds; // 자식 보관함 식별자 목록
    private List<Integer> cardIds; // 포함한 카드 식별자 목록

    // 보관함을 관리하라(생성)
    public Deck(String name, int parentId) { }
    
    // 보관함을 관리하라(이동)
    public boolean changeParentDeck(int parentId) { }
    
    // 보관함을 관리하라(삭제)
    public boolean removeDeck() { }
    
    // 카드를 분류하라
    public boolean classifyCards(int cardId, int to) { }
    
    // 카드를 찾아라(키워드 검색)
    public List<Card> findCards(String keywords) { }
    // 카드를 찾아라(플래너를 통해 card의 식별자를 알고 있는 경우)
    public List<Card> findCards(List<Integer> cardIdList) { }
    
    // 이름을 기록하라
    public boolean modifyDeckName(int deckId, String name) {
        // 데이터베이스 수정
    }

}
```

## 플래너 객체 수정

다음으로 플래너 객체도 사용자 식별자를 인터페이스를 통해 수신하여, 지정된 사용자에 대한 정보만 읽거나 쓸 수 있도록 하였습니다.

```java
public class Planner {

    // 학습일정을 찾아라
    public List<Plan> searchStudyPlan(int userId, LocalDate startDate, LocalDate endDate) {
        // 데이터베이스에서 불러오기
    }

    // 학습일정을 기록하라
    public boolean writePlan(int userId, Plan plan) {
        // 데이터베이스에 저장
    }

}

```

## 마무리

뭔가 급하게 마무리하는 느낌이지만, 이제는 세부 구현을 하면서 설계에서 망가지면 얼마나 괴로워지는지 보기위해 최대한 만들어볼 생각입니다. 이 이상 갈팡질팡하는 것은 글로 남겨도 미래의 저에게 큰 도움은 안될 것 같습니다. 세부 구현하면서 기록으로 남길만한게 있다면 짧게나마 글로 써볼 생각입니다.

잘못되긴 했지만 뭔가 머리를 뚜드려 맞으면서 배운 느낌입니다. 메시지가 객체를 선택해야된다는 것은 책을 볼때는 끄덕끄덕했지만 실천을 못해서 이상한 도메인 모델을 만들었고, 곁가지로 프로그램에 무조건 사용자 객체가 있어야 하는 것은 아니라는 점도 배웠습니다. 갈팡질팡하지 않기 위해서 명확한 요구사항의 중요함도 깨달을 수 있었습니다.

다음에 다시 만들면 목업이나 프로토타입도 시각적으로 만들어 보면서 내가 만들고 싶은게 무엇인지 명확히하고, 요구사항도 구체적으로 정리해봐야겠습니다. 그리고 다음에는 메시지를 만들면서 주고 받아야할 데이터를 빠트리지 않도록 커뮤니케이션 다이어그램도 공부해서 활용해 볼 생각입니다.

그럼 다음에 다시 할 프로세스를 정리해보겠습니다.

1. 목업 또는 프로토타입 생성
2. 요구사항 정리
3. 도메인 모델 작성
4. 협력 설계 시 커뮤니케이션 다이어그램 작성
5. 구현

그런데 당장은 세부 구현을 하려면 데이터베이스가 필요하니 데이터 모델링을 해봐야겠습니다. 뭔지도 모르고 글로만 배웠던걸 해야하니 많은 삽질이 예상됩니다.
