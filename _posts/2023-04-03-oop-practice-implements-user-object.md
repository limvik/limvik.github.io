---
layout: post
title: 객체지향개발 연습2-3 - 사용자 객체 구현
categories:
- OOP
tags:
- OOP
- Java
date: 2023-04-03 23:36 +0900
---
## Intro
어디부터 구현을 시작하는게 좋을 것인가 고민하다가, 프로그램의 시작 지점인 `학습하라` 메시지를 받는 사용자 객체 그리고 사용자 객체와 협력하는 객체의 일부분 부터 구현하기 시작하였습니다.

결국엔 원형이 남아있지를 않은데, 그 과정을 의식의 흐름대로 작성했습니다. 글의 목적이 1차적으로는 미래의 제가 과거의 제 생각을 참고하는데 있기 때문에 장황하게 적었습니다.

## 기존 인터페이스 정리
사용자 객체의 인터페이스를 정리했던 결과는 아래와 같습니다.
```java
public class User {
    // 학습하라
    public void startStudy(List<Plan> plan, List<Deck> decks, List<Card> cards) { }

}
```
## 구현
### 학습 일정
사용자 객체의 구현을 시작하자 마자 잘못된 점을 발견했습니다.

사용자 객체는 학습 시작 시 플래너 객체에 메시지를 보내서 학습 일정(plan) 목록을 받아와야 하는데,  그냥 plan 목록을 인터페이스를 통해 받는 것으로 선언하였습니다.

그래서 먼저 플래너 인스턴스를 받아, 학습 일정을 요청하는 것으로 아래와 같이 변경하였습니다. 오늘 학습할 것을 확인해야 하므로 학습 일정의 시작과 끝이 오늘인 카드를 요청하게 하였습니다.

```java
public class User {
    
    //학습하라
    public void startStudy(Planner planner, List<Deck> decks, List<Card> cards) {
    
    // 오늘 학습 일정 불러오기
    LocalDate today = LocalDate.now();
    List<Plan> plans = planner.searchStudyPlan(today, today);
        
    }

}
```
그리고 기존의 플래너 객체의 인터페이스는 Date 클래스를 사용했지만, 수업시간에 Date 클래스는 하위호환을 위해서 남겨놓았을 뿐이라고 배웠기 때문에, Java 8에서 도입된 LocalDate 클래스로 변경하였습니다.
```java
public class Planner {

    // 학습일정을 찾아라
    public List<Plan> searchStudyPlan(LocalDate startDate, LocalDate endDate) { }
    // 학습일정을 기록하라
    public boolean writePlan(Plan plan) { }

}
```
사용자는 플래너 객체를 통해 어느 보관함에 있는 어느 카드를 공부했는지와 다음에는 언제 공부할지 기록하므로, Plan 클래스에는 아래와 같은 속성을 추가하였습니다.
```java
public class Plan {
    
    private int deckId; // 보관함 식별자
    private int cardId; // 카드 식별자
    private LocalDate studyDate; // 학습 일시
    private LocalDate nextStudyDate; // 다음 학습 일시
        
}
```
### 보관함
다음도 동일한 잘못을 수정했습니다. 보관함(Deck) 객체에 메시지를 보내지 않고, 보관함 목록을 받는 것으로 인터페이스를 정리했었습니다. 그래서 학습 일정을 수정했던과 마찬가지로 보관함 인스턴스를 받아 보관함 목록을 요청하는 것으로 변경하였습니다. 또한, 카드 목록도 별도로 받는 것으로 했었지만, 처음 프로그램에 진입하였을 때 사용자는 각 보관함 별로 카드가 몇개나 있는지 정도만 식별하면 되므로 위 코드에서 카드 목록을 전부 받는 것은 제거하였습니다. 프로그램을 사용할 때의 사용자 스토리를 고려하지 못한 결과로 보입니다.

```java
public class User {

    //학습하라
    public void startStudy(Planner planner, Deck deck) {
    
        // 오늘 학습 일정 불러오기
        LocalDate today = LocalDate.now();
        List<Plan> plans = planner.searchStudyPlan(today, today);
        
        // 보관함 목록 불러오기
        List<Deck> decks = deck.getDeckList();
        
    }

}
```
그리고 협력 설계 시 보관함 목록을 요청하는 메시지를 빠트려서 보관함 객체에 getDeckList() 메서드를 생성하지 않았었기 때문에, 아래와 같이 새롭게 추가하였습니다.  사용자가 보관함을 식별할 수 있도록 id 속성을 추가하고, 보관함은 보관함을 포함할 수 있으므로, 부모 보관함과 자식 보관함의 id를 속성으로 추가하였습니다. 자연스럽게 따라올 보관함 이름도 추가하였습니다. 또한 보관하고 있는 카드 목록에 대한 정보로 카드의 id 목록을 속성으로 추가하였습니다.
```java
public class Deck {
	
    private int id; // 보관함 식별자
    private String name; // 보관함 이름
    private int parentId; // 부모 보관함 식별자
    private List<Integer> childIds; // 자식 보관함 식별자 목록
    private List<Integer> cardIds; // 포함한 카드 식별자 목록
    
    // 나머지는 생략
    // ...
    // 보관함 목록을 찾아라
    public List<Deck> getDeckList() { }

}
```
### 메서드
다음으로는 startStudy() 메서드를 사용자 객체의 생성자(Constructor)로 변경하였습니다. 프로그램이 시작되어 사용자가 `학습하라`라는 메시지를 받았을 때는 어떤 보관함이 있고, 각 보관함 별로 공부해야 할 카드가 몇 개나 되는지 식별할 수 있는 상태가 만들어져야 한다고 생각했고, 그게 사용자 인스턴스가 생성됐을 때라고 생각했습니다.
```java
public class User {

    // 학습하라
    public User(Planner planner, Deck deck) {

        // 오늘 학습 일정 목록 불러오기
        final var today = LocalDate.now();
        List<Plan> plans = planner.searchStudyPlan(today, today);

        // 보관함 목록 불러오기
        List<Deck> decks = deck.getDeckList();

    }

}
```
### 뒤집어 엎기, 새로운 객체의 등장
그런데 학습 일정 목록 plans와 보관함 목록 decks를 사용자 객체의 속성으로 갖고 가는 것이 맞는가에 대한 의문이 들었습니다.

사용자 머리속의 기억이라 가정할까 하다가, 조금 더 현실과 가깝게 만들기로 하였습니다. 많은 정보가 있는 목록이 있다면 정보가 기록된 현황판(Board) 객체가 별도로 존재하고 사용자가 현황판을 참고하는 것이 맞다고 판단하고 현황판 객체를 추가하였습니다.
```java
public class Board {

    private List<Deck> decks; // 보관함 목록
    private List<Plan> plans; // 학습 일정 목록

    // 보관함 목록 및 보관함 별 학습 대상 카드의 개수를 보여라
    public void showBoard() { }

}
```
현황판 객체에 사용자 객체에 있던 decks와 plans를 옮겨 온 것을 볼 수 있습니다. 그리고 사용자는 프로그램이 시작됐을 때 아무 정보도 없으므로 인터페이스를 통해 현황판 객체가 수신하는 인수는 별도로 없는 것으로 했습니다.

그래서 사용자 객체는 구현 결과 굉장히 간단해 졌습니다.
```java
public class User {

    // 학습하라
    public User(Board board) {
    
	// 보관함 목록 및 보관함 별 학습 대상 카드 개수 확인하기
        board.showBoard();

    }

}
```
나중에 프론트엔드도 배워서 현황판을 보여 주는 방식을 변경하는 일이 발생하는 것을 고려했을 때 괜찮은 판단이었다고 생각됩니다. 어떻게 보여줄 것인지는 현황판 객체를 구현하면서 고민해 봐야겠습니다.

## Outro

구현하면서 갈팡질팡 했던 가장 큰 이유는 요구사항이 제대로 정의되지 않았기 때문이 아닌가 생각이 듭니다. 정보처리기사 공부할 때 요구사항 개발 단계와 관리 단계로 나누고, 개발 프로세스는 도분명확, 관리 프로세스는 협선변확으로 외우던게 생각납니다. 그만큼 요구사항을 개발하고 관리하는게 쉬운 일이 아니기 때문이라 생각됩니다. 그런데 쉬운 일이 아님에도 투자한 시간은 별로 안되니 당연한 결과겠네요.

그래도 이번에도 망했다고 버리지 않고, 할 수 있는데 까지 해본 후에 또 같은 주제로 반복해서 만들어보면서 개선해보려 합니다.

다음엔 갑자기 튀어나온 현황판 객체부터 구현해 보겠습니다.
