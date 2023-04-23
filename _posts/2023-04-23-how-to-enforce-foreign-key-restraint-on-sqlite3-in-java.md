---
layout: post
title: Java에서 SQLite3 사용 시 외래 키(Foreign Key) 제약 설정하는 방법
categories:
- Database
- SQLite
tags:
- Java
- JDBC
- SQLite
date: 2023-04-23 17:41 +0900
---
## Intro

SQLite3 데이터베이스 만들 때 CREATE TABLE 구문에서 외래 키 제약을 설정했으니 당연히 제약이 동작할 거라고 생각했는데, 상위 테이블 데이터 삭제하는 테스트를 만들다가 안되는걸 발견했습니다.

## SQLite3 외래키 제약 기본 설정

문서([링크](https://www.sqlite.org/quirks.html#foreign_key_enforcement_is_off_by_default))를 보니 SQLite3의 기본적인 외래키 제약 설정은 `Off` 입니다. 

>  Foreign Key Enforcement Is Off By Default

SQLite3 에 외래키가 나중에 도입(version 3.6.19, 2019-10-14)됐다보니, 레거시 데이터베이스가 손상되지 않게 하기 위한 조치라고 하네요.

> To avoid breaking those legacy databases, foreign key constraint enforcement is turned off by default in SQLite.

## 외래키 제약 설정하기

SQLite3 에서 외래키 제약이 동작하도록 하려면, run-time 시 설정하는 방법과 compile-time에 설정하는 방법이 있습니다.

## Run-time 설정 방법

제 Java 코드의 일부를 예제로 보여드리면, 아래와 같습니다. 싱글턴 패턴으로 Database 연결을 유지하고 있습니다.

```java
private static DatabaseConnection instance;
private Connection connection;

private DatabaseConnection() throws SQLException {
    SQLiteConfig config = new SQLiteConfig();
    config.enforceForeignKeys(true);
    String dbPath = Paths.get("src", "main", "resources", "db", "flashcards.db").toString();
    connection = DriverManager.getConnection("jdbc:sqlite:" + dbPath, config.toProperties());
}
```

집중해서 보실 부분은 아래와 같습니다. `config` 객체를 만들어서 외래키 제약 설정을 `true`로 설정하고 DriverManager를 통해 연결을 가져올 때 `두` 번째 인수로 `config.toProperties()` 를 적용합니다.

```java
// 외래키 제약 설정
SQLiteConfig config = new SQLiteConfig();
config.enforceForeignKeys(true);

// 데이터베이스 연결
connection = DriverManager.getConnection("jdbc:sqlite:" + dbPath, config.toProperties());
```

## Compile-time 설정 방법

은... 제가 굳이 설명 안해도 Compile-time에 설정할 생각을 하시는 분들이라면 알아서 잘 하실거라고 생각됩니다.

아래는 그냥 한 번 chat GPT에 물어봐서 얻은 Compile-time option을 설정하는 방법에 대한 답변인데, 되는지 확인은 `안`해봤습니다. 혹시나 시도하실 분들을 위해 일단 붙여넣습니다. 

### chat GPT가 설명한 Compile-time Option 설정 방법
---

SQLite3는 컴파일 시간에 다양한 옵션을 설정할 수 있습니다. 이러한 옵션은 SQLite 라이브러리의 기능과 성능을 사용자 정의할 수 있게 해줍니다. 옵션을 설정하려면, SQLite 소스 코드를 직접 컴파일하고 설치해야 합니다.

다음은 SQLite3 컴파일 시간 옵션을 설정하는 방법입니다:

1. SQLite 소스 코드 다운로드: SQLite 소스 코드를 [공식 다운로드 페이지](https://www.sqlite.org/download.html)에서 다운로드 받습니다. "Source Code" 섹션에서 "sqlite-amalgamation-*.zip" 파일을 다운로드하십시오.

2. 압축 해제: 다운로드한 zip 파일을 압축 해제하고 적절한 디렉토리에 소스 코드를 저장합니다.

3. 컴파일 옵션 설정: 컴파일 옵션을 설정하려면, `sqlite3.c` 파일을 열고 특정 플래그를 정의하거나 수정합니다. 컴파일 시간 옵션은 `#define` 문으로 정의되며, 다음과 같은 형식입니다:

   ```c
   #define SQLITE_OPTION_NAME value
   ```

   예를 들어, `SQLITE_THREADSAFE` 옵션을 설정하려면 다음과 같이 작성합니다:

   ```c
   #define SQLITE_THREADSAFE 1
   ```

4. 컴파일 및 설치: 설정한 옵션으로 SQLite를 컴파일하고 설치합니다. UNIX 계열 시스템에서는 다음 명령어를 사용할 수 있습니다:

   ```sh
   gcc -c -O2 sqlite3.c -o sqlite3.o
   ar rcs libsqlite3.a sqlite3.o
   gcc -o sqlite3 shell.c sqlite3.o -lpthread -ldl
   ```

   이제 `sqlite3` 실행 파일과 `libsqlite3.a` 정적 라이브러리가 생성되었습니다. 필요한 경우 이 파일들을 시스템 라이브러리 디렉토리로 복사하거나 프로젝트에 직접 포함시킬 수 있습니다.

SQLite 컴파일 시간 옵션에 대한 자세한 내용은 [공식 문서](https://www.sqlite.org/compile.html)를 참조하십시오. 다양한 옵션과 값들이 설명되어 있습니다.

---
외래키 제약조건 Compile-time Option 문서([링크](https://www.sqlite.org/compile.html#default_foreign_keys))

## Outro

여태 잘 되는줄 알았는데... 데이터베이스 뜯어고치러 가야겠습니다.

## 참고자료

[https://stackoverflow.com/questions/9774923/how-do-you-enforce-foreign-key-constraints-in-sqlite-through-java](https://stackoverflow.com/questions/9774923/how-do-you-enforce-foreign-key-constraints-in-sqlite-through-java)
