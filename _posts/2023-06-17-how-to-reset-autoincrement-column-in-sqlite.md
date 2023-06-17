---
layout: post
title: SQLite 에서 AUTOINCREMENT 컬럼 초기화하는 방법
categories:
- Database
- SQLite
tags:
- SQLite
date: 2023-06-17 16:07 +0900
---
## Intro

SQLite 에서는 AUTOINCREMENT 설정 시 새로운 테이블(`sqlite_sequence`)이 생성되는데, 이 테이블에서 AUTOINCREMENT가 설정된 정보가 담긴 row를 찾아 제거해야 자동으로 증가하여 저장되는 숫자를 초기화할 수 있습니다.

## Summary

```sql
DELETE FROM sqlite_sequence WHERE name='테이블이름';
```

## 확인해보기

### CREATE

아래와 같이 id에 AUTOINCREMENT가 설정된 테이블을 생성합니다.

```sql
CREATE TABLE "flashcards" (
	"id"	INTEGER NOT NULL,
	"front"	TEXT NOT NULL,
	"back"	TEXT,
	PRIMARY KEY("id" AUTOINCREMENT)
)
```

DB browser 를 통해 확인해보면, 아래와 같이 자동으로 sqlite_sequence 테이블이 생성된 것을 확인할 수 있습니다.

![DB browser의 테이블 목록 화면](/assets/img/2023-06-17-how-to-reset-AUTOINCREMENT-column-in-SQLite/01.sqlite_sequence-in-db-browser.png)

### INSERT

테스트용 데이터를 아래와 같이 삽입합니다.

```sql
INSERT INTO flashcards(front, back) VALUES("front", "back");
```

### SELECT

그리고 확인해보면, id가 1로 자동으로 저장되어 있습니다.

```sql
SELECT * FROM flashcards;
```

![DB browser에서 flashcards 테이블을 전체 조회한 결과](/assets/img/2023-06-17-how-to-reset-AUTOINCREMENT-column-in-SQLite/02.select-from-flashcards.png)

sqlite_sequence 테이블도 확인해보면, name 컬럼에는 AUTOINCREMENT를 설정한 테이블 이름인 flashcards 가 있고, 1개를 삽입했기 때문에 seq 컬럼에는 1이 저장되어 있습니다.

```sql
SELECT * FROM sqlite_sequence;
```

![DB browser에서 sqlite_sequence 테이블을 전체 조회한 결과](/assets/img/2023-06-17-how-to-reset-AUTOINCREMENT-column-in-SQLite/03.select-sqlite_sequence.png)

### DELETE

아래와 같이 flashcards 에 있는 데이터를 전부 DELETE 구문을 이용하여 삭제를 해봅니다.

```sql
DELETE FROM flashcards;
```

하지만 다시 INSERT를 하면, id는 2부터 삽입이 되는 것을 확인할 수 있습니다.

![DB browser에서 flashcards 테이블 데이터를 전부 지우고 삽입한 후 전체 조회 한 결과](/assets/img/2023-06-17-how-to-reset-AUTOINCREMENT-column-in-SQLite/04.select-from-flashcards-after-delete-from-flashcards.png)

이는 sqlite_sequence 테이블에 flashcards 테이블의 sequence 번호가 남아있기 때문입니다.

그래서 id를 다시 1부터 삽입하려면 아래와 같이 sqlite_sequence 테이블에서 flashcards 테이블의 sequence 번호를 삭제해 주어야 합니다.

```sql
DELETE FROM sqlite_sequence WHERE name="flashcards";
```

그러면 다시 1부터 증가시키면서 저장이 되는 것을 확인할 수 있습니다.

### UPDATE

원한다면, seq 값을 바꿔서 다음에 저장될 sequence 번호를 변경할 수도 있습니다.

```sql
UPDATE sqlite_sequence SET seq = 100 WHERE name = "flashcards";
```

![DB browser에서 sqlite_sequence 번호를 변경하고 데이터를 삽입한 후 전체 조회 한 결과](/assets/img/2023-06-17-how-to-reset-AUTOINCREMENT-column-in-SQLite/05.update.png)

## Outro

데이터베이스 테스트 코드를 작성하면서 저번에 sqlite를 사용할 때는 AUTOINCREMENT를 사용하지 않았었다는걸 발견해서 정리할 겸 작성해 봤습니다.
