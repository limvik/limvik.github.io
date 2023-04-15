---
layout: post
title: 데이터베이스 설계 연습1-3 논리 데이터 모델링
categories:
- Database
- DataModeling
tags:
- Database
- DataModeling
- Anki
- Normalization
- ERD
date: 2023-04-15 23:23 +0900
---
## Intro

[이전 글](https://limvik.github.io/posts/database-design-1-2-conceptual-data-modeling/)에서 개념 데이터 모델에 대한 ERD를 그려보았습니다.

![플래시카드 개념 데이터 모델 ERD](/assets/img/2023-04-12-database-design-1-2-conceptual-data-modeling/2023-04-12-erd.png)

먼저 논리 데이터 모델링에 대한 이론을 다시 한 번 훑어보면서 대략적인 방향을 잡고, 이전에 수행한 개념 데이터 모델링을 바탕으로 논리 데이터 모델링을 수행해 보았습니다.

## 논리 데이터 모델링 이론 다시 훑어보기

### 기승전 관계형

이전에 전체적인 데이터베이스 설계 과정에 대해 조사한 글([링크](https://limvik.github.io/posts/database-design-1-1-data-modeling-research/))을 작성하면서 논리 데이터 모델링에 와서야 계층형, 관계형, 네트워크 등 데이터 모델의 종류를 선택하게 되는 것으로 조사를 했었는데, 저는 개념 데이터 모델링을 하면서도 기승전 `관계형`으로 진행을 했습니다. 위의 ERD를 가지고도 다른 형태의 데이터베이스로 설계 진행이 가능한걸 수도 있겠지만 아직 더 알아볼 타이밍은 아닌 것 같아서 그대로 진행하기로 하였습니다.

### 이론과 다른 실무1 : 논리 데이터 모델링의 중요도

이론과 실무는 당연히 다르지만 달라도 너무 다릅니다. AWS의 문서([링크](https://aws.amazon.com/ko/what-is/data-modeling/))를 보면, 논리 데이터 모델에 대한 설명에 아래와 같이 작성되어있습니다.

> 민첩한 팀은 `이 단계를 건너뛰고` 개념 모델에서 실제 모델로 바로 넘어가기도 합니다.

이 문장만 보면 논리 데이터 모델은 크게 중요한 단계로 안느껴집니다. 하지만 조금 더 이론적인 측면에서 작성되어 있는 DA 가이드의 `논리 데이터 모델링 이해`([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=285)) 문서를 보면 아래와 같이 논리 데이터 모델링의 중요성에 대해 언급합니다.

> 특히 데이터 모델링 과정에서 `가장 핵심이 되는 부분`이 `논리 데이터 모델링`이라고 할 수 있다. 데이터 모델링이란 모델링 과정이 아닌 별도의 과정을 통해서 조사하고 결정한 사실을 단지 개체-관계 다이어그램(ERD, Entity Relationship Diagram)이라는 그림으로 그려내는 과정을 말하는 것이 아니다. 시스템 구축을 위해서 가장 먼저 시작할 기초적인 업무조사를 하는 초기 단계부터 인간이 결정해야 할 대부분의 사항을 모두 정의하는 시스템 설계의 전 과정을 지원하는 과정의 도구라고 해야 할 것이다.

달라도 너무 다릅니다.

### 이론과 다른 실무2 : 식별자

제가 ERD CLOUD는 키가 없으면 관계 표시도 불가능해서 반 강제로 `기본키(PK, Primary Key)`와 `외래키(FK, Foreign Key)`를 표시했습니다. 하지만 `이론적`으로는 `식별자는 논리 데이터 모델링에서 확정`되고, 개념 데이터 모델링에서는 도출되는 핵심 속성 정도만 표시합니다. 아래는 DA 가이드의 `개념 데이터 모델 작성`([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=284)) 문서에 나와있는 개념 데이터 모델 예시 입니다. 관계와 일부 속성이 표시되어 있지만, `키`로 사용하는 속성이 무엇인지는 표시되어 있지 `않`습니다.

![개념 데이터 모델](https://dataonair.or.kr/publishing/img/dbguide/edu/da0502_0602.gif)

---

제가 개념 데이터 모델을 그리고서 혼란스러운 점들을 적어봤었는데, 그중에 아래와 같은 혼란스러운 점을 적어놨었습니다.

> 어디까지가 정확하게 개념 데이터 모델링이고, 어디부터가 논리 데이터 모델링의 시작이라고 봐야할지도 잘 모르겠습니다.

아무래도 이론과 실무의 차이로 경계가 애매하다보니 저한테 혼란스럽게 다가왔던 것으로 보입니다. 

---

어쨌거나 저는 제 상황에 맞춰 진행을 해야하니 정리를 해봐야겠습니다. 제가 개념 데이터 모델링에서 그린 ERD를 바탕으로 현재 논리적 데이터 모델링 단계에서 해야할 일을 정리해보면 아래와 같습니다.

1.  이전에 조사([링크](https://limvik.github.io/posts/how-to-store-tree-structure-in-database/))했던, 계층 구조를 데이터베이스에 저장하는 패턴을 사용하여 `보관함` 엔터티(entity) 수정
2. 추가적인 속성 식별
3. 속성의 (DBMS에 독립적인)데이터 유형 설정
4. 정규화

## 보관함 엔터티 수정

이전에 계층 구조를 데이터베이스에 저장하기 위한 패턴을 조사하였을 때, 적합해 보이는 것으로 Materialized Path와 Closure Table을 두고 고민하였습니다. 

사실 조사한 결과만 보면 아래와 같기 때문에 당연히 Materialized Path 이지만, 데이터베이스에서 참조 무결성 손실이 발생하는 만큼 어플리케이션에서 보완을 해야하므로 주저하게됩니다.

-   조회 : Nested Set > Storing Paths(또는 Materialized Path) > Closure Table > Parent-Child
-   수정 : Parent-Child > Storing Paths(또는 Materialized Path) > Closure Table > Nested Set
-   삽입 : Parent-Child > Storing Paths(또는 Materialized Path) > Closure Table > Nested Set
-   삭제 : Parent-Child > Storing Paths(또는 Materialized Path) > Closure Table > Nested Set

그래서 제가 자주 사용하여 신뢰할 수 있고, 오랜기간 검증된 플래시카드 어플리케이션인 Anki 를 따라하기 위해 코드를 살펴봤습니다. 

### Anki 코드 살펴보기

먼저 보관함(Deck)의 INSERT OR UPDATE DECK SQL([Github 링크](https://github.com/ankitects/anki/blob/fd81700679b09b4d1b10eb19361bff314494edfe/rslib/src/storage/deck/add_or_update_deck.sql))을 살펴보면 아래와 같습니다.

```sql
INSERT
  OR REPLACE INTO decks (id, name, mtime_secs, usn, common, kind)
VALUES (?, ?, ?, ?, ?, ?)
```

컬럼명만으로는 유추가 어려워 더 찾아보니, 아래와 같이 카드를 업데이트하는 코드([Github 링크](https://github.com/ankitects/anki/blob/9dc3cf216a8a180426aff2da05e1e4834a3688ef/pylib/tests/test_find.py))를 찾을 수 있었습니다.

```python
col.db.execute(
        "update cards set did = ? where id = ?", col.decks.id("Default::Child"), id
    )
```

Anki에서는 자식 Deck을 표시할 때 `::` 뒤에 자식 Deck 이름을 작성하는데, id를 정수가 아닌 문자열로 표시하여 Materialized Path 패턴을 사용함을 알 수 있었습니다.

그러면 저도 Anki를 따라 `Materialized Path 패턴을 적용`하기로 하고, 아래와 같이 `부모 식별자` 속성을 `제거`하고 `조상` 속성을 `추가`하여 ERD를 수정하였습니다.

![Materialized Path를 적용한 ERD](/assets/img/2023-04-15-database-design-1-3-logical-data-modeling/2023-04-15-materialized-path.png)

Anki 처럼 id를 문자열로 하여 조상 Deck까지 모두 나타낼까 하다가, 식별 관계에 있는 다른 엔터티에 긴 문자열을 모두 저장하는 것은 비효율적인 것으로 판단되어 `별도의 조상을 저장하기 위한 속성`을 추가하였습니다.

Anki의 Deck 테이블에는 사용자 식별자, Anki 용어로는 Profile 관련한 컬럼이 존재하지 않는데, 뇌피셜로 유추해보자면 root 보관함의 이름으로 Profile 관련 정보를 넣어두는게 아닐까 생각됩니다.

## 추가적인 속성 식별

추가적인 속성을 식별하기 전에 또 간단하게 DA 가이드에서 속성 정의에 대한 문서([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=286))를 훑어봤습니다.

눈에 들어오는 것들을 몇 가지 정리해보면 아래와 같습니다.

- 속성은 엔터티에서 관리되는 구체적인 정보 항목으로 더 이상 분리될 수 없는 최`소`의 데이터 보관 단위이다. 가능한 최소 단위까지 분할한 후 관리 필요에 따라 통합한다.
- 만약 이 속성이 없다면 다시는 재현할 수가 없을 때 이 속성을 `원시(Source) 속성`이라고 부른다. 재현이 불가능하다면 이것을 버리는 순간, 이미 정보는 소실되어 버리므로 절대 버려서는 안된다.
- `미래`의 시스템 구조를 찾아 나가는 과정

어플리케이션을 만들 때 미래에 어떻게 변하게 될지도 고려하면서 만들어야 하므로, 데이터 모델도 만들 때 부터 미래를 고려하라는 소리로 이해됩니다. 정확한 요구 사항이 있다면 기준으로 삼고 더 쉽게 진행할 수 있을거란 생각이 듭니다.

사용자, 플래너, 보관함, 카드 엔터티 각각 미래에 어떻게 변할지 생각하면서 속성을 식별해 봐야겠습니다.

### 사용자 엔터티

지금 어렴풋이 머릿속에 있는 요구사항으로는 온라인 서비스를 만들더라도 회원가입 없이도 오프라인에서 사용가능한 클라이언트는 존재하게 할 것이므로, 미래에도 현재 사용자 엔터티에서 추가적인 속성을 필요로 하는 일은 없을 것으로 예상됩니다.

### 플래너 엔터티

플래너를 통해 내가 얼마나 많은 시간을 학습했는지 돌아볼 수 있어야 하므로 `학습 시간` 속성을 추가하고, 학습 시 틀렸는지 맞았는지 등을 표시하기 위한 `학습 결과` 속성을 추가하였습니다.

![플래너 엔터티에 속성을 추가한 ERD](/assets/img/2023-04-15-database-design-1-3-logical-data-modeling/2023-04-15-add-planner-attribute.png)

`학습 결과`는 정수형으로 표시하였는데, 이산형으로 각 숫자별로 결과의 종류를 구분하는 것이 좋다고 판단하였기 때문입니다.

### 보관함 엔터티

추후에 보관함 별로 복습 주기를 다르게 한다던지 다양한 설정 정보를 저장하기 위한 `설정` 속성을 추가하였습니다.

![보관함 엔터티에 속성을 추가한 ERD](/assets/img/2023-04-15-database-design-1-3-logical-data-modeling/2023-04-15-add-deck-attribute.png)

`설정` 속성도 정수형으로 표시하였는데, 나중에 설정을 프리셋으로 저장해두고 사용하기 위함입니다. '리눅스 conf 파일 또는 VS Code에서 JSON(JavaScript Object Notation)으로 설정을 관리하는 것 처럼, 별도의 속성 설정 항목을 저장하는 파일을 만드는 방식으로 구현할 수 있지 않을까?' 하면서, 만들어 본적이 없어 상상만 해봤습니다.

### 카드 엔터티

카드의 손쉬운 검색을 위한 `태그` 속성과 카드 별로 메모를 남길 수 있도록 `메모` 속성을 추가하였습니다. 그리고 Anki 같은 경우에는 학습 반복 주기 계산에 한 가지 알고리즘만 사용하여, 해당 알고리즘에 사용되는 계수들을 카드 엔터티에 저장하지만 저는 알고리즘의 확장 혹은 변경 가능성을 고려하여 `알고리즘 종류` 속성을 추가하였습니다.

![카드 엔터티에 속성을 추가한 ERD](/assets/img/2023-04-15-database-design-1-3-logical-data-modeling/2023-04-15-add-card-attribute.png)

`알고리즘 종류` 속성 같은 경우 보관함에 적용해서, 같은 보관함에 있는 카드는 같은 알고리즘을 적용하는 것이 맞지 않을까 고민을 했습니다. 그런데 카드의 내용에 따라서 다른 주기로 보고 싶을 수도 있으므로 카드 엔터티에 추가하였습니다.

## 속성 데이터 유형 설정

속성 데이터 유형을 설정한다는 것이 처음에는 안 떠올랐는데, DA 가이드를 보다보니 시험문제에 자주 나오던 `도메인`을 설정하는 것이었습니다. 지식을 너무 때려 넣기만 해서, 지식 간의 연결이 잘 안되나 봅니다.

아래는 DA 가이드의 속성 정의 문서([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=286))에 나온 도메인에 관련된 설명입니다.

### 도메인

속성이 지닐 수 있는 값에 대한 업무적인 `제약 조건`으로 파악된 일련의 특성이다. 모든 영역에서 같은 도메인을 사용하는 것이 좋다. 속성이 기존 도메인 집합에 속해 있지 않는 경우에는 새로운 도메인을 추가한다. 도메인은 다음과 같은 속성들을 가진다.

- 데이터 타입
- 길이
- 허용 값(Permitted Value): 속성에 지정할 수 있는 모든 값들의 집합
- 디폴트 값 및 디폴트 알고리즘

### ERD 수정

현재 사용하는 ERD 툴에서 제약 조건을 표시할 방법이 없습니다(제가 기능을 몰라서 그럴지도...?). 그래서 `데이터 타입`과 `길이`만 수정하였습니다. 

![데이터 타입과 길이를 수정한 ERD](/assets/img/2023-04-15-database-design-1-3-logical-data-modeling/2023-04-15-modify-erd-datatype-and-length.png)

각 엔터티의 `id` 의 경우 데이터 길이를 어떻게 해야할지 고민하다가 ERD CLOUD에서 OKKY 의 ERD([링크](https://www.erdcloud.com/d/PK2Ae7d4asTRqHpHx))도 공개하고 있어서 동일하게 하였습니다.

Anki에서도 `id` 길이가 13자이고, 간단히 검색해보니 절대적인 기준은 없는 것으로 판단됩니다.

![Anki 카드 정보](/assets/img/2023-04-15-database-design-1-3-logical-data-modeling/2023-04-15-anki-card-id.png)

`사용자` 엔터티의 `이름` 속성은 한국에서 가장 긴 이름이 30자이지만, 외국인을 고려했을 때 스택오버플로우([링크](https://stackoverflow.com/questions/354763/common-mysql-fields-and-their-appropriate-data-types))에 외국인이 Full name을 고려한 길이를 70자로 예시를 줘서 동일하게 VARCHAR(70)으로 했습니다. 세계에서 가장 긴 이름은 백글자도 넘는다고 합니다.

![한국에서 가장 긴 이름](/assets/img/2023-04-15-database-design-1-3-logical-data-modeling/2023-04-15-longest-name-in-korea.png)

그냥 편하게 TEXT로 하고 싶은데, VARCHAR로 설정할 때와 TEXT로 설정할 때의 차이가 큰지 한 번 조사해 봐야겠습니다.

`보관함` 엔터티의 `조상` 속성은 조금 고민이 됐는데, 일단 깊이를 제한하는 것은 고려하지 않았기 때문에 얼마나 길어질지 알 수 없어 TEXT로 설정하였습니다.

`보관함` 엔터티의 `이름` 속성은 저는 책을 읽을 때도 주로 사용하기 때문에 가장 긴 책 이름을 찾아봤습니다. 기네스 웹페이지([링크](https://www.guinnessworldrecords.com/world-records/358711-longest-title-of-a-book))를 보니 책 이름이 27,978 자 입니다. 이를 고려해서 TEXT로 가야하나 싶지만, 어차피 어플리케이션에서 한 화면에 보관함은 한 줄로 표현할 것이기 때문에 적당히 VARCHAR(100)에서 끊었습니다. 요구사항이 정확했다면 조금 더 구체적인 근거를 갖고 설정할 수 있겠습니다. 2008년 자료([링크](https://blog.yes24.com/blog/blogMain.aspx?blogid=2008book&artSeqNo=1157612))이긴 하지만 국내에서 2008년에 출간된 가장 긴 이름의 책이 50글자라고 하니 국내 도서의 제목을 담기에는 큰 무리가 없을 것 같습니다.

이외에 INT(2)로 한 것과 태그는 굳이 길어질 일이 없다는 뇌피셜로 간단하게 정했습니다. 그리고 사용자와 플래너 관계를 기존에 `1:1` 관계로 표현해놔서 `1:다` 관계로 수정하였습니다. 현실적으로 하나의 플래너에 학습한 것을 기록하는 것을 생각하면서 `1:1` 관계를 표현한 것 같습니다. 엔터티 이름이 `학습 일정` 혹은 `학습 기록` 이 적절하지 않을까 고민이 됩니다.

---

허용값이나 디폴트 값 그리고 제약 조건 등은 물리 데이터 설계 시에 DDL(Data Definition Language)을(를) 작성하면서 구체화 해야겠습니다.

## 정규화

정규화를 할게 `카드` 엔터티의 `태그` 속성 밖에 없습니다. 태그는 여러 개를 갖을 수 있기 때문에 제 1 정규형을 적용하여 다른 엔터티로 분리하였습니다.

![정규화된 ERD](/assets/img/2023-04-15-database-design-1-3-logical-data-modeling/2023-04-15-erd-normalization.png)

`보관함` 엔터티의 `조상` 속성도 하나의 속성에 여러 정보를 갖고 있으니 제 1 정규형을 수행해야 할 대상이긴 합니다. 제 1 정규형을 적용한다면 Closure Table...이 되겠네요. 이런 관계가 있었군요.

정석적으로 수행하자면 `조상` 속성은 물리 데이터 설계에서 비정규화(De-normalization)를 하면서 추가됐어야 하는 속성이 되겠습니다.

## Outro

이론적인 내용을 모두 챙기자니 설계하는데만 이 작은 어플리케이션도 몇 주 걸릴 것 같고, 너무 구현에만 집중하면 나중에 뭔가 문제가  생길 것 같습니다. 이 중간 어딘가에서 최적점을 찾아내는게 개발자의 역할이겠죠? 그러기 위해서는 이론을 알고 무엇을 생략하거나 간략화할지 선택할 수 있어야 하지 않을까 생각됩니다.

물론 이번에는 뭣도 몰라서 많은 부분을 생략했습니다. 다시 한 바퀴 돌 때는 조금 더 나아져 있기를 바라봅니다.

생각했던 일정보다 많이 지연되고 있어 슬슬 마음이 조급해집니다. 아직 배우는 단계이니 참을성이 필요한 단계라 생각됩니다. 다음에는 이번에 하면서 잘못된 것은 없는지 다시 확인하고, 물리 데이터 모델을 작성해 보겠습니다. 

## 참고자료 정리

- DA 가이드 : 개념 데이터 모델 작성([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=284))
- DA 가이드 : 논리 데이터 모델링 이해([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=285))
- DA 가이드 : 속성 정의([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=286))
- DA 가이드 : 엔터티 상세화([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=287))
- DA 가이드 : 이력 관리 정의([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=288))
- DA 가이드 : 논리 데이터 모델 품질 검토([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=289))
- AWS : 데이터 모델링이란 무엇인가요? ([링크](https://aws.amazon.com/ko/what-is/data-modeling/))
