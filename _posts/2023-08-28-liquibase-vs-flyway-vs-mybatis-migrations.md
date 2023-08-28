---
layout: post
title: Liquibase, Flyway, MyBatis Migration 무엇을 사용할까?
categories:
- Database
- Tools
tags:
- Liquibase
- Flyway
- Database Migration
date: 2023-08-28 16:26 +0900
---
## Intro

이전에 Swagger Generator 를 이용해서 프로젝트를 생성하는 글([링크](https://limvik.github.io/posts/how-to-use-swagger-generator-6-6-0-in-spring-boot-3-x/))을 적었었습니다. 책 예제가 옛날거라 문제를 해결하면서 적었던 건데... 여튼 이게 주 내용은 아니고, 책에서 Flyway 를 사용했지만, 뭔지도 모르고 일단 예제에 있으니까 사용했었습니다.

당시에는 프로젝트만 만들고 중단해서 Flyway가 무슨 목적으로 사용되는건지는 자세히 보지 않았었습니다. 그래서 이번 기회에 Flyway 가 뭔지 확인해보고, Flyway 그냥 쓰면 되는건지 확인해볼 겸 검색한 자료들을 정리해봤습니다.

## Database Migration 도구

당시에는 migration 디렉터리에 sql 파일을 넣길래 단순히 데이터베이스 제품을 다른걸로 바꿀 때 편하게 해주는건가 했습니다. 그런데 지금 생각해보니 그건 JPA의 이점인데, 굳이 Flyway 같은게 또 필요한가 의문이 들어 GPT에 물어봤습니다. 그중에서도 핵심만 보자면, 아래와 같습니다.

>데이터베이스 마이그레이션은 데이터베이스의 스키마나 데이터를 한 버전에서 다른 버전으로 변경하는 과정을 의미합니다.

Wikipedia 의 Flyway 페이지([링크](https://en.wikipedia.org/wiki/Flyway_%28software%29))에는 `Flyway is an open-source database-migration tool.` 이라고 나와있고, Flyway 홈페이지([링크](https://flywaydb.org/))에는 `Version control for your database.` 라고 소개하고 있습니다.

데이터베이스 자체를 변경하는데 도움을 주는게 아니라, `스키마 버전 관리`를 해주는 것이었습니다. MySQL 에서는 database와 schema 를 동의어로 사용하는데, MySQL 처럼 database와 schema를 동의어로 사용하는 것 같습니다.

Wikipedia 의 database-migration에 링크가 걸려있어 눌러보니, schema-migration 이라고도 한답니다.

>In [software engineering](https://en.wikipedia.org/wiki/Software_engineering "Software engineering"), a **schema migration** (also **database migration**, **database change management**) refers to the management of [version-controlled](https://en.wikipedia.org/wiki/Version_control "Version control"), incremental and reversible changes to [relational](https://en.wikipedia.org/wiki/Relational_database "Relational database")  [database schemas](https://en.wikipedia.org/wiki/Database_schema "Database schema"). A schema migration is performed on a database whenever it is necessary to update or revert that database's schema to some newer or older version.

진작에 알았으면 테이블 변경 할 때마다 팀원들한테 테이블 수정 SQL 전달하느라 고생하지 않아도 됐겠습니다.

그런데 선택지가 Flyway 밖에 없는지 궁금해집니다.

일단 GPT에 물어보니 나오는 것이 몇 가지 있었는데, Flyway를 포함해서 Liquibase, MyBatis Migrations가 있었습니다.

## 제품 살펴보기

MyBatis를 사용하면서도 존재조차 몰랐던 MyBatis Migrations가 신기해서 먼저 살펴봅니다.

### MyBatis Migrations

공식 문서([링크](https://mybatis.org/migrations/index.html))는 MyBatis와 동일한 스타일의 문서로 구성이 돼있습니다.

Maven 저장소([링크](https://mvnrepository.com/artifact/org.mybatis/mybatis-migrations))를 보면, 처참한 사용량...

![MyBatis Migrations의 Maven 저장소 화면](/assets/img/2023-08-28-liquibase-vs-flyway-vs-mybatis-migrations/01-mybatis-migrations.png)

엄... 조용히 MyBatis Migrations 는 닫아줍니다.

### Liquibase vs Flyway

해외에서도 선택장애가 오는지 두 개 비교한 글이 좀 보입니다.

[Database migration: Flyway vs Liquibase - LinkedIn](https://www.linkedin.com/pulse/database-migration-flyway-vs-liquibase-rafael-porto-rodrigues/)  
[Liquibase vs Flyway - Baeldung](https://www.baeldung.com/liquibase-vs-flyway)  

요약하자면 Liquibase가 기능이 더 많기는 한데, Flyway가 사용법이 간편한 것에 비해 Liquibase 는 설정을 위해 배워야할게 좀 있습니다.

Liquibase의 Snapshot 기능([문서 링크](https://docs.liquibase.com/commands/inspection/snapshot.html)) 때문에 끌리기는 한데, 다른 것도 새롭게 해봐야할 것이 많아 고민이 됩니다. 그래서 다른 글을 더 찾아보니 작성자분이 직접 사용해보면서 비교해보신 좋은 글이 있습니다.

[DB 형상관리툴 비교 - Flyway와 Liquibase](https://341123.tistory.com/20)

직접 사용한걸 보니, 확실히 Flyway가 간단해보입니다.

## 최종 선택

Liquibase가 기능이 다양해서 익혀두면 좋을 것 같기는 한데, 지금은 여기에 시간들일 때가 아닌 것 같아서 상대적으로 간단하게 사용할 수 있는 `Flyway`를 사용해야겠습니다.

## Outro

개인적으로 데이터베이스 버전 관리하는게 좋다고 느껴졌는데, 딱히 이에 대해 중요성을 언급하는 것은 본적이 없어서 쓸 데 없는데 시간 보내는거 아닌가 싶기도 합니다. 저는 직접 불편함을 느꼈던 것을 해결해줄 도구라 적극적으로 사용해보려고 합니다.

그런데 문득 하나의 데이터베이스를 사용하면서 프로젝트를 분리해서 작업하는 경우 관리하기 어렵겠다는 생각도 듭니다. 개인 프로젝트에 사용할 예정이니 일단 사용해보면서 뭐가 좋고, 나쁜지 직접 겪어봐야겠습니다.
