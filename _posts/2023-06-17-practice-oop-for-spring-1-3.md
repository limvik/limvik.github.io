---
layout: post
title: Spring을 위한 객체 지향 연습 1-3. 첫 기능 구현
categories:
- OOP
tags:
- OOP
- Java
date: 2023-06-17 20:16 +0900
---
## Intro

플래시카드의 생성/수정/삭제 기능을 구현하는 것을 진행하고 있었던 [이전 글](https://limvik.github.io/posts/practice-oop-for-spring-1-2/)에 이어서 Javadoc을 작성하기도 하고, 테스트도 작성하면서 나름의 객체지향 프로세스를 이어서 진행해봤습니다.

물론 계획대로 되지는 않았습니다.

## 4. API 설계 ~ 8. 메서드 구현

원래 계획대로라면 4. API 설계, 5. 테스트 구현, 6. 인스턴스 변수 정의, 7. 생성자 정의, 8. 메서드 구현이 순서대로 진행돼야 했습니다. 어디서부턴가 한 스텝 꼬이고나서 오르락 내리락 하면서 구분의 의미가 없어졌습니다. 물론 애초에 단계를 오르내리려고 하기는 했지만, 테스트 구현보다 메서드 구현이 먼저 나와버린게 아쉬웠습니다.

### API 설계하다 냅다 구현해버리기

이전 글에서 드디어 도메인 모델에서 데이터베이스 접근 코드를 분리해야 함을 깨달아서 도메인 객체인 Flashcard의 API 설계 테이블이 텅텅 비어버렸습니다.

Javadoc이라도 작성해야지 하면서 작성했는데, 아직 설계 초기 단계라 딱히 적을 것도 없어서 뭔가 아쉬워서 id랑 front, back은 Flashcard에 당연히 있어야지 하면서 추가하게 되고, getter도 추가하게 됐습니다. 그리고 나서 아차 싶었죠... 구현을 한 것이 거의 없으니 재작업을 해도 크게 문제되지는 않을 것으로 판단되어 이대로 진행했습니다.

```java
/**
 * 이 클래스는 플래시카드(Flashcard) 객체를 정의합니다.
 */
class Flashcard {
    private Long id;
    private CardFace frontFace;
    private CardFace backFace;

    Flashcard(Long id, CardFace frontFace, CardFace backFace) {
        this.id = id;
        this.frontFace = frontFace;
        this.backFace = backFace;
    }

    public Long getId() {
        return id;
    }

    /**
     * 플래시카드 앞면에 저장되어 있는 내용을 반환합니다.
     * @return 플래시카드 앞면에 저장되어 있는 내용 반환
     */
    public String getFrontContents() {
        return frontFace.getContents();
    }

    /**
     * 플래시카드 뒷면에 저장되어 있는 내용을 반환합니다.
     * @return 플래시카드 뒷면에 저장되어 있는 내용 반환
     */
    public String getBackContents() {
        return backFace.getContents();
    }
}
```

### 설계 결정과정

API 테이블 만들때만 해도 Flashcard 인터페이스를 만들고, 다양한 종류의 Flashcard를 구현하려고 했습니다. 하지만 Flashcard가 어떤 종류이든 그것은 앞면과 뒷면으로 구성된 형식 안에서 벌어질 일이기 때문에 Flashcard의 종류에 따라 비즈니스 로직이 달라지지는 않는다고 판단하여 class로 구현했습니다.

frontFace와 backFace는 String으로 데이터 타입을 설정하려다가, 추후에 웹으로 옮겼을 때는 악의적인 스크립트 등이 포함된 문자열을 필터링해야 하는데, 필터링 행위를 Flashcard의 비즈니스 로직으로서 다루는 것은 맞지 않다고 판단돼서 별도의 CardFace 클래스를 만들었습니다.

```java
/**
 * 이 클래스는 플래시카드(Flashcard)에 포함되는 내용이 저장되는 객체를 정의합니다.
 * 플래시카드의 한쪽 면에 표시될 콘텐츠가 저장 됩니다.
 */
public class CardFace {

    private String contents;

    CardFace(String contents) {
        this.contents = contents;
    }

    /**
     * 저장 가능한 콘텐츠일 경우 콘텐츠를 저장하고 true를 반환합니다.
     * @param contents 콘텐츠 내용
     * @return 저장 가능한 콘텐츠일 경우 true 그렇지 않을 경우 false
     */
    public boolean setContents(String contents) {
        if (contents != null) {
            this.contents = contents;
            return true;
        }
        return false;
    }

    /**
     * 저장되어 있는 콘텐츠를 불러옵니다.
     * @return 저장된 콘텐츠 문자열(String) 반환
     */
    public String getContents() {
        return contents;
    }

    /**
     * 콘텐츠가 저장되어 있지 않은 경우 true를 반환합니다.
     * @return 콘텐츠가 저장되어 있지 않은 경우 true 그렇지 않을 경우 false
     */
    public boolean isEmpty() {
        return contents == null || contents.length() == 0;
    }
}
```

### 테스트 구현

사실 getter 밖에 없어서 테스트를 구현한다는게 큰 의미가 없기는 했지만, 새로 배운 JUnit 메서드인 assertAll()을 써보고 싶어서 그냥 인스턴스 생성하는 테스트를 만들었습니다.

```java
public class FlashcardTest {
    
    @Test
    @DisplayName("Flashcard 객체 생성 테스트")
    public void flashcardTest() {
        long testId = 1L;
        String testFront = "front";
        String testBack = "back";
        
        Flashcard flashcard = new Flashcard(testId,
        new CardFace(testFront),
        new CardFace(testBack));

        assertAll("정상적으로 객체가 생성되지 않았습니다.",
        () -> assertEquals(testId, flashcard.getId()),
        () -> assertEquals(testFront, flashcard.getFrontContents()),
        () -> assertEquals(testBack, flashcard.getBackContents()));
    }
}
```

## 데이터베이스 접근 코드 구현

나중을 생각하면 도메인 모델에 대한 전체적인 그림을 그리고 데이터베이스 접근 코드를 작성해야겠지만, 전체적인 그림을 그린다고 해도 수정하게 될거고, 현재 프로세스는 설계와 코드 작성을 작게 반복하면서 진행하기로 했으므로 데이터베이스 접근 코드도 필요한 만큼만 구현했습니다.

### 테이블 만들기

플래시카드가 당장 하는 일은 없지만 구현하고 있던 기능이 플래시카드 생성/수정/삭제이므로 이에 대한 플래시카드 테이블은 아래와 같이 작성할 수 있습니다.

![ER Diagram](/assets/img/2023-06-17-practice-oop-for-spring-1-3/01.flashcards_diagram.svg)

### 데이터베이스 접근 코드 작성을 위한 패키지 구조 만들기

아직 Spring을 사용하는 단계는 아니지만, MVC 구조에 익숙해질 겸 이전에 사용하던 DAO(Data Access Object)로 이름 짓던 것은  repository로 이름을 지었습니다. 그리고 도메인 클래스 하나로 모든 곳을 누비며 사용하던 것에서 탈피하여, repository에서 데이터베이스 데이터 전달에만 사용하기 위해 새롭게 배운 Entity 클래스를 도입해 봤습니다.

![VS Code에서 본 패키지 구조](/assets/img/2023-06-17-practice-oop-for-spring-1-3/02.package-structure.png)

### Entity 클래스

책에서는 Entity 클래스가 빈약한 도메인 모델로 설계될 때 캡슐화 특성을 위반하고 임의로 수정될 위험이 있지만 Entity 가 서비스 계층에 전달되자마자 Domain이나 BO(Business Object)로 변환돼서 수명주기가 짧아 임의로 수정될 여지가 없다고 했습니다. 하지만 그 짧은 순간도 안심하고 사용할 수 있게(나도 모르게 내가 헛짓거리 하지 않게) Entity 클래스는 record 클래스로 작성하였습니다. 일반 클래스로도 불변성을 달성할 수 있겠지만, 절대 변경하지 않겠다는 의도를 명확하게 표현하기에는 record 클래스가 더 적합하다고 판단됩니다. 귀찮은 getter, setter 선언 안해도 되는건 보너스.

```java
public record FlashcardEntity (long id, String front, String back) {}
```

record 클래스에 직접 setter를 설정하는 경우를 제외하고 record 클래스의 불변성이 보장되지 않는 경우도 있는지 확인해야겠지만, 일단은 JEP의 Summary에 나와있는 요약을 믿고 진행해보기로 합니다.

>[JEP 395: Records](https://openjdk.org/jeps/395)
>**Summary**
>Enhance the Java programming language with  [records](http://cr.openjdk.java.net/~briangoetz/amber/datum.html), which are classes that `act as transparent carriers for immutable data`. Records can be thought of as  _nominal tuples_.

그리고 Entity 클래스가 책에서 나왔다고 그대로 따라하는건 아닙니다. 팀 프로젝트를 하면서 DTO(Data Transfer Object) 하나로 여기저기 사용하다보니 DTO가 점점 거대해지는 경험을 해서, 파일을 하나 더 만들게 되는 귀찮음을 감수하고라도 데이터베이스 쪽에서 사용하는 Entity와 비즈니스 로직을 처리하는 도메인 객체인 Domain(또는 BO)을 분리하는게 좋겠다고 판단했습니다.

### Repository 구현

데이터베이스 연결 인스턴스는 싱글턴 패턴으로 사용하고, 흔히 볼 수 있는 JDBC를 사용하는 DAO 클래스 코드에 Javadoc을 달아보기도 했습니다.

```java
public class FlashcardRepo {
    private Connection connection;

    FlashcardRepo() {
        try {
            connection = DatabaseConnection.getInstance().getConnection();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * 신규 플래시카드를 1개 저장하고, 성공 시 true를 반환합니다.
     * @param flashcardEntity 신규 플래시카드 콘텐츠가 저장된 Entity 객체
     * @return 신규 플래시카드 1개 저장 성공 시 true, 그렇지 않을 경우 false 반환
     */
    public boolean createFlashcard(FlashcardEntity flashcardEntity) {

        String sql = "INSERT INTO flashcards(front, back) VALUES (?, ?)";
        int rowCount = 0;
        try (PreparedStatement statement = connection.prepareStatement(sql)) {
            statement.setString(1, flashcardEntity.front());
            statement.setString(2, flashcardEntity.back());
            rowCount = statement.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }

        return rowCount == 1;

    }
    
    ...
    // 수정, 삭제, 조회 코드 생략
}
```

### 테스트 구현

테스트도 간단하게 구현했습니다.

Given-When-Then 패턴을 따라야하나 싶기도 했는데, 아직 기능을 완전히 구현한 것도 아니고 지금의 구현을 확인하는 것에 충실하면 된다고 생각해서 간단하게 작성했습니다.

```java
public class FlashcardRepoTest {

    private static DatabaseConnection databaseConnection;
    private static FlashcardRepo flashcardRepo;

    private static final long[] id = {1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L, 9L, 10L};
    private static final String[] front = {"front1", "front2", "front3", "front4", "front5", "front6", "front7", "front8", "front9", "front10"};
    private static final String[] back = {"back1", "back2", "back3", "back4", "back5", "back6", "back7", "back8", "back9", "back10"};

    @BeforeAll
    public static void setup() throws SQLException {
        databaseConnection = DatabaseConnection.getInstance();
        flashcardRepo = new FlashcardRepo();
    }

    @AfterEach
    public void cleanup() throws SQLException {
        String sql = "DELETE FROM flashcards";
        databaseConnection.getConnection().prepareStatement(sql).executeUpdate();
        sql = "DELETE FROM sqlite_sequence WHERE name='flashcards'";
        databaseConnection.getConnection().prepareStatement(sql).executeUpdate();
    }

    @AfterAll
    public static void tearDown() throws SQLException {
        String sql = "VACUUM";
        databaseConnection.getConnection().prepareStatement(sql).executeUpdate();
        databaseConnection.closeConnection();
    }

    @Test
    @DisplayName("플래시카드 데이터 삽입 테스트")
    public void createFlashcard() {
        assertTrue(flashcardRepo.createFlashcard(new FlashcardEntity(id[0], front[0], back[0])),
                                                 () -> "플래시카드 삽입에 성공해야 하지만 실패");
    }
    
    ...
    // 수정, 삭제, 조회 테스트 코드 생략
}
```
나머지 코드는 [github](https://github.com/limvik/simple-flashcards/tree/main/flashcards-for-study-oop)

## Outro

굉장히 작게 하다보니 테스트가 별 의미없는 것 같게 느껴지기도 하고, 하지 않는게 나은거 같은 생각도 들었습니다. 그런데 만약 테스트가 없었을 때 제가했을 행동을 생각해보면 테스트를 작성하는게 효율적입니다. 테스트가 없었다면 저는 아마 생성/수정/삭제 코드가 제대로 돌아가는지 테스트하려고 임시로 생성/수정/삭제를 선택하는 메뉴를 만들고, 손수 테스트하고 있었을 겁니다.

또 아직 뭔가 테스트를 먼저 만든다는게 익숙하지 않고 getter, setter 구현이 전부였다보니 구현부터 해버리기도 했지만, 기능 구현을 하게되면 자연스럽게 테스트를 먼저 구현하게 되지 않을까 생각됩니다. 매번 코딩 테스트 문제 만드는 느낌으로...?

이제 다음 글에서는 또 새로운 기능을 추가하면서 맞닥드리는 문제들을 기록하려고 하다가, 어차피 공부 목적으로 시작한거 리플렉션(reflection)에 대해 공부해서 코드에 적용해보고, JDBC에서 MyBatis 그리고 Hibernate 까지 전환해보려고 합니다.
