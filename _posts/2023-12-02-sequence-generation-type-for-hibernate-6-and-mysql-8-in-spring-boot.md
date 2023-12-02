---
layout: post
title: Spring Boot 에서 Hibernate 6 와 MySQL 8 사용 시 GenerationType.SEQUENCE 설정 방법
categories:
- Java
- JPA
tags:
- Java
- JPA
- Hibernate
- MySQL
date: 2023-12-02 22:28 +0900
---
## Intro

테스트 중에 상대적으로 소요시간이 오래 걸리는 테스트가 있습니다.

![01.Slow Test Result](/assets/img/2023-12-02-hibernate-6-mysql-8-generation-type-sequence/01.slow-test.png)

통계 정보가 잘 나오는지 테스트 하다보니, 테스트용 과거 데이터를 `INSERT`하는데 시간이 많이 걸려서 테스트 시간을 줄이고, 덤으로 운영 시에도 데이터베이스 서버와 통신 횟수를 줄이기 위해 Generation Type을 `IDENTITY`에서 `SEQUENCE`로 변경하여 관련 내용을 공유합니다.

## Generation Type IDENTITY의 문제

MySQL에서는 Sequence Table이 생성되지 않다보니 Generation Type을 `IDENTITY`로 설정해왔습니다. 그런데, `IDENTITY`에서는 `INSERT` 작업을 Batch 처리할 수 없는 문제가 있습니다.

공식 문서([링크](https://docs.jboss.org/hibernate/orm/6.4/userguide/html_single/Hibernate_User_Guide.html#best-practices-jdbc-batching))에서는 아래와 같이 언급하고 있습니다.

>If the underlying database supports sequences, you should always use them for your Hibernate entity identifiers.
>
>Only if the relational database does not support sequences (e.g. MySQL 5.7), you should use the  `IDENTITY`  generators. However, you should keep in mind that the  `IDENTITY`  generators disables JDBC batching for  `INSERT`  statements.

Hibernate 검색하면 자주 보이는 Vlad Mihalcea 님의 블로그 글([링크](https://vladmihalcea.com/jpa-persist-and-merge/))에 따르면, entity가 저장됐을 때는 반드시 실행 중인 Persistence Context에 연결해야 하는데, Persistence Context는 entity의 Map 역할을 하고, 이 Map의 key는 entity type과 entity identifier로 구성됩니다. 이는 repository에서 상속받을 때 볼 수 있습니다.

```java
public interface BudgetPlanRepository extends JpaRepository<BudgetPlan, Long>
```

그런데 Generation Type을 `IDENTITY`로 설정한 경우 entity의 id 값을 `INSERT` 후에야 알 수 있기 때문에 persist method가 호출되면 바로 `INSERT` 문이 실행됩니다.

아래는 원문입니다.

> Whenever an entity is persisted, Hibernate must attach it to the currently running Persistence Context which acts as a  `Map`  of entities. The  `Map`  key is formed of the entity type (its Java  `Class`) and the entity identifier.
>
>For  `IDENTITY`  columns, the only way to know the identifier value is to execute the SQL INSERT. Hence, the INSERT is executed when the  `persist`  method is called and cannot be disabled until flush time.
>
>For this reason, Hibernate disables  [JDBC batch inserts](https://vladmihalcea.com/how-to-batch-insert-and-update-statements-with-hibernate/)  for entities using the  `IDENTITY`  generator strategy.

처음에는 `IDENTITY` 로 설정한 상태에서 문제를 해결해보려고 했습니다. 최근 올라온 Vlad Mihalcea 님의 블로그 글([링크](https://vladmihalcea.com/batch-insert-mysql-hibernate/))을 포함해서 구글링해서 나오는 글들을 참고했는데, `IDENTITY` 고수하기 위해 개인적으로는 과해 보이는 작업을 해야한다거나 결국엔 `SEQUENCE`나 `JdbcTemplate`을 이용하는 방식이었습니다.

그래서 공식 문서에서도 추천하는대로 Generation Type을 `SEQUENCE`로 변경하였습니다. Hibernate 6 부터는 각 entity 마다 sequence table을 별도로 만들어주어야 하는 문제가 있기는 하지만, 방향을 틀어서 `INSERT` 작업을 Batch 처리할 때 마다 entity 별로 JdbcTemplate을 사용해야한다면 애초에 JPA를 사용하지 않는게 맞다고 생각되어 Generation Type을 `SEQUENCE`로 변경하였습니다. 

물론 테스트에서만 Batch 작업을 한다면 JdbcTemplate 도 괜찮은 선택이라 생각됩니다.

## 설정 작업

### Spring 설정

JPA 설정만 살펴보겠습니다.

`hbm2ddl`과 `batch_size`는 필수고, 이외에는 아시겠지만 자신의 프로젝트에 맞게 선택 사항입니다.

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        hbm2ddl:
          auto: create
        jdbc:
          batch_size: 100
```

#### Batch Size

`spring.jpa.properties.jdbc.batch_size`는 100으로 설정했는데, batch size를 결정하는 구체적인 요소를 파악하지 못했습니다.

다른 분의 블로그 글([링크](https://kim-solshar.tistory.com/71))을 보면 DBA 요청에 의해서 설정하는 경우도 있고, 우아한 기술블로그([링크](https://techblog.woowahan.com/2695/))에 있는 글을 보면 MySQL에는 서버로 전송할 수 있는 최대 허용 패킷 크기인 `max_allowed_packet`이 있어 이를 고려해서 설정해야겠습니다.

> [max_allowed_packet](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_allowed_packet)의 기본값은 67108864 bytes고, 최대값은 1073741824 bytes 입니다.

#### hbm2ddl

임시로 sequence table을 자동 생성하기 위해 `spring.jpa.properties.hbm2ddl.auto` 를 `create`로 설정했습니다.

공식문서([링크](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#best-practices-schema))에도 나왔듯 `update` 옵션은 production 환경에서는 적합하지 않아 쓰지 말라고하니 `update`로 설정한다면 주의해야겠습니다.

> Although Hibernate provides the `update` option for the `hibernate.hbm2ddl.auto` configuration property, this feature is not suitable for a production environment.

문서에서 FlyWay나 Liquibase 같은 migration 도구 사용을 강추하고 있고 사용도하고 있는데, 저는 일단 테스트를 위해 hbm2ddl.auto를 설정합니다.

>You should always use an automatic schema migration tool and have all the migration scripts stored in the Version Control System.

### Entity 설정

Entity에서 Generation Type을 SEQUENCE로 변경해줍니다.

```java
@Id  
@GeneratedValue(strategy = GenerationType.SEQUENCE)  
private Long id;
```

그리고 실행시켜보면, 아래와 같이 entity 별로 `_seq` 가 post fix 로 붙은 Sequence Table이 생성된 것을 볼 수 있습니다. 내부에는 `BIGINT` type의 `next_val` 이라는 하나의 컬럼만 존재합니다.

![02.Sequence Table](/assets/img/2023-12-02-hibernate-6-mysql-8-generation-type-sequence/02.created-schema.png)

### MySQL Connector/j 설정

1개의 Query로 만들어주는 것은 MySQL Connector/j 이기 때문에 설정이 필요합니다.

아래는 JDBC 경로를 Testcontainer 용으로 설정한 것이긴 한데, rewriteBatchedStatements=true 부터 보시면 됩니다.

> jdbc:tc:mysql:8.0.35:///?user=root?password=test?rewriteBatchedStatements=true&profileSQL=true&logger=Slf4JLogger&maxQuerySizeToLog=99999

- `rewriteBatchedStatements=true` 는 반드시 설정해주어야 Batch 처리가 됩니다. true로 설정해야 `INSERT`나 `UPDATE` 시에 개별 Query가 아닌 1개의 Query로 합쳐서 요청합니다.
- `profileSQL=true` 는 실제로 전송되는 SQL을 출력합니다.
- `logger=Slf4JLogger` 는 log를 출력할 logger를 설정합니다.
- `maxQuerySizeToLog=99999` 는 log에 출력할 Query의 최대 길이를 설정합니다. 기본 값은 2048 입니다.

설정에 자세한 설명은 MySQL 매뉴얼([링크](https://dev.mysql.com/doc/connector-j/en/connector-j-reference-configuration-properties.html))을 통해 확인하실 수 있습니다.

## 실행

이제 `INSERT` 작업을 Batch 처리하여 테스트를 다시 실행해봅니다. 코드에서는 `saveAll()` 메서드를 호출합니다.

![03.Test Again](/assets/img/2023-12-02-hibernate-6-mysql-8-generation-type-sequence/03.apply-batch-and-test-again.png)

주요 타겟이었던 테스트의 소요 시간이 736ms 에서 284ms 로 줄었습니다. 오차 감안하면 대략 400ms 감소, 50% 넘게 줄었습니다. 그리고 log를 보면 hibernate 는 각 `INSERT` 마다 Query를 출력하지만, MySQL Connector의 log를 보면 하나의 `INSERT` 문으로 전송된 것을 볼 수 있습니다.

```
Hibernate: 
    insert 
    into
        expenses
        (amount,category_id,datetime,exclude_in_total,memo,user_id,id) 
    values
        (?,?,?,?,?,?,?)
Hibernate: 
    insert 
    into
        expenses
        (amount,category_id,datetime,exclude_in_total,memo,user_id,id) 
    values
        (?,?,?,?,?,?,?)
2023-12-02T21:01:26.793+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] insert into expenses (amount,category_id,datetime,exclude_in_total,memo,user_id,id) values (10000,5,'2023-11-16 21:01:26.690175',null,'오늘 지출',2,136),(10000,6,'2023-11-16 ...
```
`INSERT` 문이 너무 길어서 뒤에는 생략했습니다.

그런데 `INSERT` 할 데이터의 갯수가 많을 때는 Sequence Table 호출이 여러번 되는 것을 볼 수 있습니다.

```
Hibernate: 
    select
        next_val as id_val 
    from
        expenses_seq for update
2023-12-02T21:01:26.695+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] select next_val as id_val from expenses_seq for update [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 1, connection-id: 10, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)]
2023-12-02T21:01:26.695+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [FETCH]  [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 0, connection-id: 10, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)]
Hibernate: 
    update
        expenses_seq 
    set
        next_val= ? 
    where
        next_val=?
2023-12-02T21:01:26.697+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] update expenses_seq set next_val= 151 where next_val=101 [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 2, connection-id: 10, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
2023-12-02T21:01:26.697+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [FETCH]  [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 0, connection-id: 10, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
2023-12-02T21:01:26.702+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] commit [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 5, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.commit(ConnectionDelegate.java:11)]
2023-12-02T21:01:26.702+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [FETCH]  [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 0, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.commit(ConnectionDelegate.java:11)]
2023-12-02T21:01:26.703+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] SET autocommit=1 [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 1, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.setAutoCommit(ConnectionDelegate.java:11)]
2023-12-02T21:01:26.703+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [FETCH]  [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 0, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.setAutoCommit(ConnectionDelegate.java:11)]
2023-12-02T21:01:26.707+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] SET autocommit=0 [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 1, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.setAutoCommit(ConnectionDelegate.java:11)]
2023-12-02T21:01:26.708+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [FETCH]  [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 0, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.setAutoCommit(ConnectionDelegate.java:11)]
Hibernate: 
    select
        next_val as id_val 
    from
        expenses_seq for update
2023-12-02T21:01:26.709+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] select next_val as id_val from expenses_seq for update [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 1, connection-id: 10, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)]
2023-12-02T21:01:26.710+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [FETCH]  [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 0, connection-id: 10, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)]
Hibernate: 
    update
        expenses_seq 
    set
        next_val= ? 
    where
        next_val=?
2023-12-02T21:01:26.712+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] update expenses_seq set next_val= 201 where next_val=151 [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 2, connection-id: 10, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
2023-12-02T21:01:26.712+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [FETCH]  [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 0, connection-id: 10, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
2023-12-02T21:01:26.718+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] commit [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 5, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.commit(ConnectionDelegate.java:11)]
2023-12-02T21:01:26.718+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [FETCH]  [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 0, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.commit(ConnectionDelegate.java:11)]
2023-12-02T21:01:26.720+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] SET autocommit=1 [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 1, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.setAutoCommit(ConnectionDelegate.java:11)]
2023-12-02T21:01:26.720+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [FETCH]  [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 0, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.setAutoCommit(ConnectionDelegate.java:11)]
2023-12-02T21:01:26.724+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [QUERY] SET autocommit=0 [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 1, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.setAutoCommit(ConnectionDelegate.java:11)]
2023-12-02T21:01:26.724+09:00  INFO 38524 --- [    Test worker] MySQL                                    : [FETCH]  [Created on: Sat Dec 02 21:01:26 KST 2023, duration: 0, connection-id: 10, statement-id: -1, resultset-id: 0,	at org.testcontainers.jdbc.ConnectionDelegate.setAutoCommit(ConnectionDelegate.java:11)]

...
```

`next_val` 이 50씩 증가하는게 눈에 띕니다.

```sql
update expenses_seq set next_val=151 where next_val=101
update expenses_seq set next_val=201 where next_val=151
```

allocation size의 기본값이 50이라 그렇습니다.

다음과 같이 entity class에서 `@SequenceGenerator` annotation을 설정하여 allocation size를 변경할 수 있습니다.

```java
@Id  
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "expenses_generator")  
@SequenceGenerator(name = "expenses_generator", sequenceName = "expenses_seq", allocationSize = 100)  
private Long id;
```

그러면 다음과 같이 allocationSize에 설정한 값대로 증가하는 것을 볼 수 있습니다.

```sql
update expenses_seq set next_val=401 where next_val=301
```

매번 다음 id 값이 필요할 때마다 데이터베이스에 요청을 보내는 비효율적인 일을 하지 않도록 sequence block으로 예약하는 것이라 설명한 블로그 글([링크](https://passionate.tistory.com/59))이 있는데, 출처가 김영한님 책이라 그래도 제일 신뢰가 가는 것 같습니다. 제가 본 것에 한해서는 다른 곳의 글도 다 비슷비슷 합니다.

## 이미 운영 중인 데이터가 있다면?

저는 개인 프로젝트고 운영 중인 어플리케이션이 아니다보니, 실 운영 데이터가 없어 SEQUENCE로 그냥 바꾸면 됩니다. 실 운영 중이라면 기존의 id 값이 있으니 조금 더 설정을 해줘야 합니다.

크게 어렵지는 않습니다. `@SequenceGenerator` 에 `initialValue`를 설정해주면 됩니다.

예를 들어, 기존의 id가 최대값이 42 였다면, +1 을 해서 initialValue를 43으로 설정해줍니다.

```java
@Id  
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "expenses_generator")  
@SequenceGenerator(name = "expenses_generator", sequenceName = "expenses_seq", allocationSize = 100, initialValue = 43)  
private Long id;
```

log를 보면 아래와 같이 initialValue가 반영된 것을 볼 수 있습니다.

```sql
update expenses_seq set next_val= 343 where next_val=243
```

자세한 내용은 [migration guide](https://github.com/hibernate/hibernate-orm/blob/6.0/migration-guide.adoc#defaults-for-implicit-sequence-generators)를 살펴보는 것을 추천합니다.

## Outro

Hibernate 6 부터는 entity 별로 Sequence Table이 추가된다는 단점이 있기는 하지만, 1개의 컬럼에 1개의 값이 있는 Table이라 `IDENTITY` 를 굳이 고수할 필요는 없어 보입니다.

## 참고자료

- [How do persist and merge work in JPA](https://vladmihalcea.com/jpa-persist-and-merge/)
- [How to batch INSERT statements with MySQL and Hibernate](https://vladmihalcea.com/batch-insert-mysql-hibernate/)
- [MySQL Connector/J Developer Guide](https://dev.mysql.com/doc/connector-j/en/)
- [MySQL System Variables - max_allowed_packet](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_allowed_packet)
- [MySQL 환경의 스프링부트에 하이버네이트 배치 설정 해보기](https://techblog.woowahan.com/2695/)
- [About @GenerationType AUTO/SEQUENCE in MySQL](https://discourse.hibernate.org/t/about-generationtype-auto-sequence-in-mysql/7717)
- [Hibernate Migration Guide - Defaults for implicit sequence generators](https://github.com/hibernate/hibernate-orm/blob/6.0/migration-guide.adoc#defaults-for-implicit-sequence-generators)
- [Hibernate Migration Guide - Implicit Identifier Sequence and Table Name](https://github.com/hibernate/hibernate-orm/blob/6.0/migration-guide.adoc#implicit-identifier-sequence-and-table-name)
- [시퀀스 allocationSize 정리](https://passionate.tistory.com/59)
