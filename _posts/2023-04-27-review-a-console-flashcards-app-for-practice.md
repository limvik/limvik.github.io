---
layout: post
title: 연습용 Console 플래시카드 앱 제작 후기
categories:
- OOP
tags:
- OOP
- Java
- Flashcards
date: 2023-04-27 22:44 +0900
---
## Intro

앞서 진행했던 설계들을 바탕으로 Java를 이용해 플래시카드(Flashcards) 어플리케이션을 만들어봤습니다. 설계는 멍멍이한테 준 느낌이지만... '그래도 안하는 것보다는 더 나은 과정을 경험하지 않았나'하고 자평하고 있습니다.

![플래시카드 어플리케이션 시연](/assets/img/2023-04-27-review-console-flashcards-app-for-practice/2023-04-27-application-demo.gif)

생각했던 기능을 다 만들지는 않았지만, 이후 과정은 코드 복붙 반복에 만들자마자 소프트웨어의 생명이 끝납니다. 그래서 고도화 하는게 큰 의미가 없다고 생각이 되어 '사용자 추가 - 카드 보관함 생성 - 카드 추가 및 학습' 정도의 프로세스만 진행되도록 만들고 중단하였습니다.

특히 중단한 이유 중에 하나는 『클린 코드』를 읽다보니 아래와 같은 내용을 읽고 고민해보니, 제가 더 나은 코드라고 생각하면서 리팩토링해도 지식이 부족한 상태에서는 뇌피셜에 기반한 나만의 클린 코드가 될 것 같았기 때문입니다.

> "깨끗한 코드를 어떻게 작성할까?" 깨끗한 코드가 무엇인지 모르면 깨끗한 코드를 만들려고 애써봤자 소용이 없다. ... 깨끗한 코드와 나쁜 코드를 구분할 줄 안다고 깨끗한 코드를 작성할 줄 안다는 뜻은 아니다.

마치 영어 리딩은 가능해도 스피킹은 한 마디 조차 어려운 느낌...?

여튼, 객체 지향 연습을 위한 작은 어플리케이션이지만 초보자인 저한테는 새롭게 배운 것도 있고 아직 능력이 많이 부족하다는 것을 느낄 수 있었습니다.

## 막혔던 부분

잘한 것과 못한 것을 구분할 능력도 아직은 부족하기 때문에, 시간을 허비했던 부분과 만들고나서 왜 이렇게 만들었지 싶었던 대표적인 것 두 가지를 작성해봤습니다.

### 1. 외래 키 제약(Foreign Key Constraints)

아래는 논리 데이터 모델링을 하면서([링크](https://limvik.github.io/posts/database-design-1-4-physical-data-modeling/)) 그렸던 ERD(Entity Relation Diagram) 입니다.

![ERD](/assets/img/2023-04-15-database-design-1-3-logical-data-modeling/2023-04-15-erd-normalization.png)

대표적으로 카드(Cards) 테이블을 생성하는데 사용했던 CREATE 문을 보면 아래와 같습니다. 

```sql
--카드 테이블 생성
CREATE TABLE "Cards" (
	"id"		INTEGER NOT NULL,
	"deck_id"	INTEGER NOT NULL,
	"user_id"	INTEGER NOT NULL,
	"front"		TEXT NOT NULL,
	"back"		TEXT,
	"memo"		TEXT,
	"algorithm_type"INTEGER DEFAULT 1,
	"create_date"	TEXT DEFAULT 'datetime(''now'')',
	"card_type"	INTEGER NOT NULL,
	PRIMARY KEY("id","deck_id","user_id"),
	FOREIGN KEY("deck_id") REFERENCES "Decks"("id") ON DELETE CASCADE,
	FOREIGN KEY("user_id") REFERENCES "Decks"("user_id") ON DELETE CASCADE
);
```

마지막 줄에서 외래 키(foreign key) 제약 설정 시에 `user_id` 컬럼의 references로 `보관함(Decks)` 테이블에 있는 `user_id`를 설정하였습니다. 이로인해 `상`위 테이블의 개체를 `삭제`했는데도 외래키 제약으로 상위 테이블 개체를 삭제할 수 없는 현상이 나타났습니다. 

MySQL에서는 같은 방식으로 외래키 제약을 걸어도 이상 없이 동작을 하다보니, 어디서 문제를 찾아야 할지 판단이 되지 않아 시간을 많이 보냈습니다. 그런데 해결 방법은 간단했습니다. 혹시나 해서 아래와 같이 `user_id`는 references 테이블을 사용자(Users) 테이블로 변경하여 해당 컬럼의 원 소유주(?)를 참조하게 하였습니다. (정확한 용어를 찾지 못해 죄송합니다.)

```sql
-- 변경 전
FOREIGN KEY("user_id") REFERENCES "Decks"("user_id") ON DELETE CASCADE
-- 변경 후
FOREIGN KEY("user_id") REFERENCES "Users"("id") ON DELETE CASCADE
```

관련 내용을 찾기가 어려운데, 아무래도 데이터베이스 세부 구현 상의 차이가 아닐까 생각을 해봅니다. 물리 데이터 모델링을 수행하면서 각 데이터베이스에 맞는 세부 사항을 조정할 수 있는 지식이 부족했다고 정리해볼 수 있겠습니다.

지금과 같은 상황에서 외래키 참조를 어떤 테이블에 하는게 맞는 것인가에 대해서는 아직 찾지 못했습니다. ERD를 그리는데 사용했던 [ERDCLOUD](https://www.erdcloud.com/)에서 SQL 내보내기 기능을 사용하면, 제가 처음에 CREATE 문을 작성했던 대로 Users 테이블이 아닌 Decks 테이블을 참조 테이블로 사용하고 있기는 합니다.

![ERDCLOUD SQL 내보내기 화면](/assets/img/2023-04-27-review-console-flashcards-app-for-practice/2023-04-27-export-sql-in-erdcloud.png)

현재까지는 `데이터베이스에 맞춰서 작성`해야 한다는 것으로 결론을 내려야겠습니다.

---

참고로 SQLite는 외래 키 제약 설정이 기본적으로 off 되어 있어 설정을 별도로 해주어야 해서 관련 글([링크](https://limvik.github.io/posts/how-to-enforce-foreign-key-restraint-on-sqlite3-in-java/))을 별도로 작성하였습니다.

### 2. 잘못 적용한 MVC(Model-View-Controller)

원래 MVC 라는걸 시험 문제로만 봤지, 자세한 것은 잘 몰라서 적용해볼 생각은 없었습니다. 그런데 도메인 모델 설계를 했으니 `Model`이 존재하게 됐고, 사용자 입력을 하나의 클래스에서 처리하려고 하다보니 입력을 처리하는 `Controller` 클래스를 만들게 됐습니다. 그리고 각 사용자, 보관함, 카드 등 각 객체를 삽입, 수정, 삭제 하는 것 마다 다른 메뉴 화면이 필요했습니다. 화면이 너무 많아지니 각각의 화면을 관리할 클래스가 필요해서 `View` 라는 이름을 붙인 클래스를 만들었습니다.

![VS Code에서 본 패키지 구조](/assets/img/2023-04-27-review-console-flashcards-app-for-practice/2023-04-27-project-package-structure.png)

문제는 MVC 패턴을 적용하겠다는 생각은 없고, 그냥 이름만 그렇게 붙였을 뿐이라는 생각과 빨리 마무리 해야겠다는 조급함의 환장의 콜라보로 View 클래스는 문자열 보관함으로 전락했습니다.

```java
package com.limvik.view.user;

import com.limvik.enums.UserMenu;
import com.limvik.view.View;

public class UserMenuView implements View {

    private static final String WELCOME = "사용자 메뉴 입니다. 원하시는 메뉴를 선택해주세요.";
    private static final String MENU_GUIDE = "위 메뉴 중 하나를 입력 후 엔터 키를 눌러주세요. 예) 1\n>";
    private static final String ERROR = "정확한 메뉴 번호를 입력하세요. 예) 1\n>";
    private static final String LOADING = "선택하신 화면으로 이동합니다.";

    @Override
    public void printFirstMessage() {
        System.out.println(WELCOME);
    }

    @Override
    public void printLoading() {
        System.out.println(LOADING);
    }

    @Override
    public void printMenu() {
        int menuNumber = 1;

        for (UserMenu userMenu : UserMenu.values()) {
            System.out.println(menuNumber + ". " + userMenu.getDescription());
            ++menuNumber;
        }

        System.out.print(MENU_GUIDE);
    }

    @Override
    public void printError() {
        System.out.print(ERROR);
    }
    
}
```

그리고 어느새 도메인 모델 중 하나의 클래스에서 대부분의 로직을 처리해버리고 있었습니다.

![VS Code에서 본 board 클래스, 다른 클래스는 텅텅 비었는데 얘만 난리](/assets/img/2023-04-27-review-console-flashcards-app-for-practice/2023-04-27-god-class.png)

그러다 『클린 코드』를 읽으면서 아차 싶었습니다.

> 어째서 나쁜 코드를 짰는가? 급해서? 서두르느라? 아마 그랬으리라. 제대로 짤 시간이 없다고 생각해서, 코드를 다듬느라 시간을 보냈다가 상사한테 욕먹을까봐, 지겨워서 빨리 끝내려고, 다른 업무가 너무 밀려 후딱 해치우고 밀린 업무로 넘어가려고 ... 모두가 겪어본 상황이다.
> ...
> 나쁜 코드를 양산하면 기한을 맞추지 못한다. 오히려 엉망진창인 상태로 인해 속도가 곧바로 늦어지고, 결국 기한을 놓친다. 기한을 맞추는 유일한 방법은, 그러니까 빨리 가는 유일한 방법은, 언제나 코드를 최대한 깨끗하게 유지하는 습관이다.

어떻게 해야하나 고민하다가, 다시 뒤집어 엎는 것은 만들자 마자 버려질 소프트웨어에 시간 낭비라 생각돼서 남아있는 View는 제약이 좀 있지만 View를 다시 만든 다면 어떻게 만들지 고민하고 만들어봤습니다. 새로만든 View의 메뉴를 출력하는 일부를 보자면, 아래와 같습니다. 계속 해서 도메인 모델에 포함시켰던 사용자와 상호작용하는 코드를 View로 가져왔습니다.

```java
    @Override
    public void printMenu() {
        
        // 진입 메시지 출력
        View.clearScreen();
        printDeckName();
        printFirstMessage();
        View.pause(1);
        while (!cards.isEmpty()) {
            View.clearScreen();
    
            // 카드 앞면 출력
            printCardFront();
    
            // 카드 뒷면 출력
            printCardBack();
    
            // 메뉴 출력 및 사용자 선택 입력받기
            for (var menu : StudyCardMenu.values()) {
                System.out.println(menu.ordinal() + 1 + ". " + menu.getDescription());
            }
            System.out.print(MENU_GUIDE);
    
            var menu = (StudyCardMenu) InputController.getMenuInput(this, StudyCardMenu.values());
            switch(menu) {
                case WRONG, CORRECT:
                    // 틀렸을 때 혹은 맞았을 때의 가중치를 반영한 다음 학습일정 계산
                    break;
                case BEFORE:
                    return;
                case EXIT:
                    exit();
                    break;
            }
        } // end while
        printLoading();
        View.pause(1);
    }
```

---

MVC같은 `패턴`은 큰 어플리케이션에서나 적용하는 거라는 고정관념이 있었습니다. 그래서 그냥 이름만 MVC로 붙였을 뿐이라는 생각과 함께 코드를 작성했습니다.

그 당시에 사고 과정이 다르게 흘러갔으면 조금 더 만족스러운 결과를 얻을 수 있지 않았을까 생각됩니다.

1. 지금 만들고 있는게 MVC 패턴하고 유사하다고 볼 수 있는건가?
2. MVC 패턴이 정확히 무엇인가?
3. 내 프로젝트에 어떻게 하면 적용시켜볼 수 있을까?

구현 중에 생겨난 조급함과 성급함이, 크고 어려워보이는 걸 작은거에 적용해서 배워보려는 프로젝트의 취지도 잊게 한 것 같습니다.

시작 할때만 해도 Console에서 이런 칼라 그래프도 출력하면서 즐기면서 하려고 했는데, 배워야 할게 쌓여가는 와중에 속도를 못내다 보니 조급해졌던 것 같습니다. 조급해 한다고 달라질게 없는데, 나이를 먹어도 감정을 제어하는게 쉽지 않습니다.

![학습 일정 표시에 쓰려고 했던 막대 그래프](/assets/img/2023-04-27-review-console-flashcards-app-for-practice/2023-04-27-bar-graph.png)

## 개선한 부분

잘 된건지는 판단할 능력이 부족하기 때문에, 그래도 이전보다는 개선한 부분을 적어봤습니다.

### 1. 메뉴에 enum 클래스 사용

처음에 정말 아주 작게 객체 지향 연습([글 링크](https://limvik.github.io/posts/%EC%9E%91%EC%9D%80-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EB%A1%9C-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%EC%97%B0%EC%8A%B5%ED%95%98%EA%B8%B0/))한다고 만들었을 때는 메뉴도 한 개밖에 없어서 문자열로 만들었습니다.

```java
public static final String MENU = """
                                  메뉴를 선택해주세요.
                                  1. 학습 시작
                                  2. 종료
                                  >""";
```

아주 작으니 더 합리적일 수도 있지만, 연습하면서는 확장할 경우에 대해 생각해보는 것도 좋지 않았나 생각했습니다. 그래서 Console에서 문자열 메뉴를 관리하기에는 enum 클래스를 사용하는게 좋다고 생각해서, 이번에는 시작부터 enum 클래스를 사용해서 메뉴를 만들었습니다.

개선한 부분에 포함시키기는 했지만 문제가 없지는 않았습니다. 메뉴가 늘어나는 상황을 기능을 추가하는 경우만 생각해서 실행시간(runtime)에 선택지가 가변적으로 변하는 경우를 대응할 수 없었습니다.

그래서 `동적(Dynamic) enum`이라고 불리는 유사 enum(?)을 알게돼서 사용자 추가/삭제 등 으로 인해 선택지가 동적으로 변하는 경우에 대응하였습니다. enum 은 아니지만 클래스로 enum을 흉내내는 방식입니다.

```java
package com.limvik.enums;

import java.util.LinkedHashMap;
import java.util.Map;

public class DeckMenu implements Menu {

    private static final Map<String, DeckMenu> VALUES = new LinkedHashMap<>();
    public static final String BEFORE = "이전 메뉴로 돌아가기";
    public static final String START = "현재 보관함에서 학습 시작하기";
    public static final String NEW_CARD = "현재 보관함에 새로운 카드 추가하기";
    public static final String NEW_DECK = "새로운 하위 보관함 만들기";
    public static final String EDIT_DECKNAME = "현재 보관함 이름 수정하기";
    public static final String DELETE_DECK = "현재 보관함 삭제하기";
    public static final String EXIT = "종료";

    private final String name;

    private DeckMenu(String name) {
        this.name = name;
    }

    public static void create(String name) {
        DeckMenu newMenu = new DeckMenu(name);
        VALUES.put(name, newMenu);
    } 

    public static DeckMenu[] values() {
        return VALUES.values().toArray(new DeckMenu[0]);
    }

    public String getName() {
        return name;
    }

    public static void clearDeckList(boolean isRoot) {
        VALUES.clear();
        setDefaultMenus(isRoot);
    }

    private static void setDefaultMenus(boolean isRoot) {
        VALUES.put(BEFORE, new DeckMenu(BEFORE));
        if(!isRoot) {
            VALUES.put(START, new DeckMenu(START));
            VALUES.put(NEW_CARD, new DeckMenu(NEW_CARD));
        }
        VALUES.put(NEW_DECK, new DeckMenu(NEW_DECK));
        if(!isRoot) {
            VALUES.put(EDIT_DECKNAME, new DeckMenu(EDIT_DECKNAME));
            VALUES.put(DELETE_DECK, new DeckMenu(DELETE_DECK));
        }
        VALUES.put(EXIT, new DeckMenu(EXIT));
    }

    // 선택된 메뉴 유효성 검사용 정규표현식 반환
    @Override
    public String getMenuRegex() {
        int max = DeckMenu.values().length;
        if (max <= 9) {
            return "[1-" + max + "]";
        } else {
            StringBuilder patternBuilder = new StringBuilder();
            patternBuilder.append("[1-9]");
            
            for (int i = 1; i <= max/10; ++i) {
                patternBuilder.append("|").append(i).append("[0-").append(max%10).append("]");
            }
    
            return patternBuilder.toString();
        }
    }
    
}
```
정규표현식을 반환하는 getMenuRegex() 메서드는 모두 동일한 로직 사용이 가능해서 이 부분이 조금 아쉽기는 합니다.

그래도 모든 메뉴가 같은 Menu 인터페이스를 상속받아 입력을 InputController 클래스에서 하나의 로직으로 처리할 수 있었습니다.

```java
    public static Menu getMenuInput (View view, Menu[] menus) {

        // 유효성 검사용 정규표현식 설정
        Pattern pattern = Pattern.compile(menus[0].getMenuRegex());
        // 입력 및 유효성 검사
        Scanner scanner = getInstance().getScanner();
        while (!scanner.hasNext(pattern)) {
            // 오류 안내 메시지 출력
            view.printError();
            // 잘못된 입력 버퍼에서 제거
            scanner.nextLine();
        }
        
        // 입력받은 메뉴 번호 저장
        int menuNum = scanner.nextInt();
        // 개행문자 제거
        scanner.nextLine();

        return menus[menuNum - 1];
    }
```

### 2. JUnit 5 사용

개선했다는 것에 포함하기 부끄러운 수준이지만, 처음으로 사용을 해봤다는 것에 의의를 뒀습니다.

처음에 테스트 순서 지정하는 것에서도 헤매서 글([링크](https://limvik.github.io/posts/how-to-apply-test-execution-order-in-junit5/))을 쓰기도 했습니다.

테스트를 만든 것도 데이터베이스 입출력이 유일한데, 외래키 제약이 작동하는지 여부를 테스트할 생각은 왜 못했는지 모르겠습니다.

개선한 거에 포함시켰는데 잘 안됐던 것만 계속 쓰고있네요.

사실 더 많은 테스트를 만들어보고 싶었는데, 어떻게 흔히 말하는 Testable 한 코드를 만들 수 있는지 잘 몰랐습니다. 역시나 조급함에 알아볼 생각도 안했던 것 같습니다. 아 또... 잘 안된걸...

이번에는 사용해봤다는 것에 의의를 두고, 다음에는 Testable 한 코드를 만들어 봐야겠다는 욕구(?)를 만들어 냈다는 긍정적인 마무리를 하겠습니다.

## 다음에는 어떻게?

### 1. 브랜치 전략 적용해보기

git을 사용하기는 했지만 single mainline branch 방식으로 진행했습니다. 조그마한 개인 프로젝트에는 합리적인 방식이라고 git과 관련된 책에서도 언급합니다.

그런데 문제는 구현하다가 막히면 다른 것부터 먼저 해보고 싶기도 하고, 코드를 보다가 못봐주겠어서 정리를 좀 해주고 싶을 때도 있고, 기계 처럼 한 번에 하나의 작업을 하기가 어렵습니다. 이렇게 여러군데 찔러놓고 커밋을 하려면 난감해집니다. 커밋도 언제 해야하는지 명확한 기준이 무엇인지 아직 이해하지 못한 상황에서 하나의 브랜치만 사용하는게 더 혼란스럽게 만들었습니다.

그래서 현재 브랜치 전략이 다양하게 있다는 사실만 알고 있어서 다음에는 적합한 브랜치 전략을 조사하고 적용해보려고 합니다.

### 2. Testable 한 코드 작성하기

앞서도 언급했듯이 Testable 한 코드를 어떻게 작성해야 할지도 몰라서 테스트를 많이 작성해보지 못했습니다. 그리고 Testable 한 코드가 되면 이번처럼 한 클래스에 나도 모르게 몽땅 때려넣는 일을 방지할 수 있을거라 생각돼서 다음에는 꼭 Testable 한 코드를 작성해보려고 합니다. 무엇으로 공부해야 가장 효율적일지 고민됩니다.

### 3. 설계 툴 교체

도메인 모델을 그릴 때는 이쁘게 그려볼라고 Figma 로 그려보기도 했는데, 수정하기 불편해서 변경사항이 발생해도 반영을 안하게 됐었던 것 같습니다. 그리고 ERD 도 개념적 설계를 할 때 ERDCLOUD는 키 없이 관계 표현이 안돼서 불편함이 있었습니다.

그래서 찾아본 [diagrams.net](https://diagrams.net) 은 완전 무료에 가입도 필요없습니다. 그리고 기능도 다양해서 ERD, UML 도 그릴 수 있고 도메인 모델을 그리는데도 편리하게 사용할 수 있을 것으로 판단됩니다.

구현하면서 변경된 사항을 적용한 문서가 없으니, 물어볼곳도 지도도 없이 모르는 길을 찾아가는 느낌이었습니다. 이러한 상황을 방지할 수 있기를 기대해 봅니다.

### 4. GPT로 코드 리뷰하기

머리속으로 해야지 해야지 하면서 결국에는 한 번도 안했습니다. 코드 리뷰와 관련된 프롬프트도 수집하면서 혼자서 아무 생각 없이 코드를 작성하는 것을 방지해봐야겠습니다. 제한적이긴 하겠지만 혼자만의 생각에 빠져 코딩하는 것 보다는 나을 것 같습니다.

### 5. 요구사항 문서로 정리하기

머릿속에만 있는 애매모호한 요구사항 때문에 고통받기도 했습니다. 그래서 정형 명세 기법에 대해 조사해보기도 했는데, 하면 좋지만 지금 해야 할 일은 아닌거라 생각돼서 문서로 정리해 두는 것으로 셀프 타협했습니다.

다음에는 요구사항을 문서로 정리하고, 구현 중 변경사항도 반영해 가면서 오락가락하는 일 없도록 해야겠습니다.

앞서 정리한 다음에 할 것들은 결국 구현할 때 구현만 하지 않기로 요약할 수 있을 것 같습니다.

### 6. 프로토타입 만들기

이번에 그저 문자열로 구성된 메뉴라고 우습게 봤다가 대혼란을 겪었습니다. 그래서 다음에는 시각적으로 제 생각이 맞는지 확인을 먼저 하기 위해 프로토타입을 만들어 보려고 합니다.

### 7. 더 크게 만들기

다음에는 웹/모바일/데스크탑 모두 만들면서 개인적으로 주로 사용하는 Anki의 불편한 점들을 개선해볼 생각입니다.

이번에는 만들어지는 순간이 사망하는 순간인 소프트웨어를 만드느라 좀 느슨하게 한게 있었는데, 다음엔 만들어지는 순간 사망하는 소프트웨어가 아니니 좀 더 진지하게 만들어볼 수 있을거라 기대됩니다.

## Outro

생각대로 흘러가지 않아 아쉬운 점도 많지만, 또 개선할 점도 많이 찾은 과정이었습니다. 그리고 개발적인 것 만큼이나 조급해지지 않게 감정 조절 잘 하는 것도 하나의 과제가 될 것 같습니다. 욕심내지 않고 꾸준히 감당할 수 있을 만큼 개선해 나가야겠습니다.
