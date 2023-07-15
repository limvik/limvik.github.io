---
layout: post
title: JPA/Hibernate에서 @Column(columnDefinition = "TEXT") 사용하지 않고 MySQL TEXT 타입 설정하기
categories:
- Java
- JPA
tags:
- Java
- JPA
- Spring
- MySQL
- Oracle
- Docker
date: 2023-07-15 17:45 +0900
---
## Intro

Spring을 공부하고 있다보니 JPA가 너무 자주보여서 궁금해서 한 번 써봤습니다.

MySQL 을 사용해서 TEXT 타입을 설정하려고 찾아보니, 찾아본거에 한해서는 대체로 @Lob이나 @Column(columnDefinition = "TEXT")를 설정하라고 합니다.

그런데 MySQL에서는 @Lob만 설정하면 TEXT가 안되고, columnDefinition에 TEXT를 지정하는 건 뭔가 마음에 안듭니다.

## 뭐가 문제야?

JPA를 사용하는 이유가 데이터베이스에 대해 신경 쓸 부분을 줄여서 비즈니스 로직에 집중할 수 있게 해주는 장점이 있지만, 개인적으로는 이식성이 높다는 게 큰 장점 중에 하나가 아닌가 생각합니다.

그런데 @Column(columnDefinition = "TEXT") annotation을 사용해서 TEXT 타입을 컬럼의 데이터 타입으로 특정하면, TEXT 타입이 없는 데이터베이스로 마이그레이션할때는 또 하나씩 다 바꿔줘야하는 문제가 발생할 것으로 생각했습니다.

그래서 TEXT 타입이 있는 MySQL과 TEXT 타입이 없는 Oracle database 에서 @Column(columnDefinition = "TEXT")이 아닌 다른 annotation을 사용해서 MySQL의 TEXT 타입, Oracle의 유사한 타입인 CLOB 타입으로 설정하는 실험을 해봤습니다.

참고로 실행 환경은 Spring Boot 3.1.1과 Spring Data JPA/Hibernate 를 사용하였습니다.

## JPA with MySQL

먼저 @Column(columnDefinition = "TEXT")을 사용하지 않고, 다른 annotation을 사용하는 방법을 찾아갔던 과정을 살펴보겠습니다.

### @Lob

검색해보면 쉽게 볼 수 있는 한 가지 방법인 @Lob 만 먼저 선언해보면, TEXT가 아닌 `TINYTEXT`로 설정이 되는 것을 볼 수 있습니다.

```java
@Data
@Entity
public class Card {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Lob
    private String front;

    @Lob
    private String back;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "learner")
    private Learner learner;

}
```

아래는 MySQL을 사용하여 시작한 경우의 로그입니다.

>Hibernate: create table card (id bigint not null auto_increment, learner bigint, back `tinytext`, front `tinytext`, primary key (id)) engine=InnoDB

### TINYTEXT

TINYTEXT 에 대한 MySQL의 문서([링크](https://dev.mysql.com/doc/refman/8.0/en/string-type-syntax.html#:~:text=TINYTEXT%20%5BCHARACTER%20SET%20charset_name%5D%20%5BCOLLATE%20collation_name%5D))를 보면, 아래와 같습니다.

>A [`TEXT`](https://dev.mysql.com/doc/refman/8.0/en/blob.html "11.3.4 The BLOB and TEXT Types") column with a maximum length of 255 (28 − 1) characters. The effective maximum length is less if the value contains multibyte characters. Each [`TINYTEXT`](https://dev.mysql.com/doc/refman/8.0/en/blob.html "11.3.4 The BLOB and TEXT Types") value is stored using a 1-byte length prefix that indicates the number of bytes in the value.
>
>최대 길이가 255(2^8 - 1)자인 TEXT 열입니다. 값에 멀티바이트 문자가 포함된 경우 유효 최대 길이는 이보다 짧습니다. 각 TINYTEXT 값은 값의 바이트 수를 나타내는 1바이트 길이 접두사를 사용하여 저장됩니다.

긴 문자열을 저장하고자 TEXT를 사용하는데, 목적과는 많이 멀다는 것을 알 수 있습니다. TEXT 타입도 문서([링크](https://dev.mysql.com/doc/refman/8.0/en/string-type-syntax.html#:~:text=M%20bytes%20long.-,TEXT,-%5B%28M%29%5D%20%5BCHARACTER))에 있는 내용을 한 번 보겠습니다.

### TEXT

>A  [`TEXT`](https://dev.mysql.com/doc/refman/8.0/en/blob.html "11.3.4 The BLOB and TEXT Types")  column with a maximum length of 65,535 (2^16  − 1) characters. The effective maximum length is less if the value contains multibyte characters. Each  [`TEXT`](https://dev.mysql.com/doc/refman/8.0/en/blob.html "11.3.4 The BLOB and TEXT Types")  value is stored using a 2-byte length prefix that indicates the number of bytes in the value.
>
>An optional length  _`M`_  can be given for this type. If this is done, MySQL creates the column as the smallest  [`TEXT`](https://dev.mysql.com/doc/refman/8.0/en/blob.html "11.3.4 The BLOB and TEXT Types")  type large enough to hold values  _`M`_  characters long.
>
>최대 길이가 65,535(2^16 - 1)자인 TEXT 열입니다. 값에 멀티바이트 문자가 포함된 경우 유효 최대 길이는 이보다 짧습니다. 각 TEXT 값은 값의 바이트 수를 나타내는 2바이트 길이 접두사를 사용하여 저장됩니다.
>
>이 타입에 대해 선택적 길이 `M`을 지정할 수 있습니다. 이렇게 하면 MySQL은 값을 `M`만큼의 길이로 저장할 수 있을 만큼 충분히 큰 가장 작은 TEXT 유형으로 열을 생성합니다.

두 내용을 살펴 본 후에 저는 TINYTEXT의 prefix 길이가 1바이트이고, TEXT는 2바이트라는 점에 주목해서 @Column의 length 속성을 수정해보기로 했습니다.

### @Column

@Column annotation을 살펴보면, length 속성 default 는 255로 설정되어 있습니다.

```java
int length() default 255;
```

딱 TINYTEXT의 최대 길이 입니다. 그래서 엄하게 큰 값을 주면, TEXT 타입으로 변경하려는 의도가 표현되지 않을 것 같아서 256으로 설정해서 다시 코드를 작성했습니다.

```java
@Data
@Entity
public class Card {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Lob
    @Column(nullable = false, length = 256)
    private String front;

    @Column(nullable = false, length = 256)
    private String back;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "learner")
    private Learner learner;

}
```

front는 @Lob과 같이 사용하고, back은 @Lob 을 제거하여 어떻게 되는지 확인해봤습니다.

>Hibernate: create table card (id bigint not null auto_increment, learner bigint, `back varchar(256)` not null, `front text` not null, primary key (id)) engine=InnoDB

그 결과 @Lob과 같이 사용했을 때 TEXT가 되고, @Lob이 없으면 VARCHAR(256)이 되는 것을 볼 수 있습니다.

MySQL 에서는 @Lob과 @Column에서 length 를 255보다 큰 값으로 설정하여, 개인적으로 마음에 안드는 @Column(columnDefinition = "TEXT") 없이 TEXT 타입으로 설정할 수 있는 것을 확인할 수 있었습니다.

다음으로 Oracle database에서 확인해보겠습니다.

## Oracle database

Oracle database 도 처음 써보는데, Docker 도 처음 써봐서 엄청 애를 먹었습니다. 저 같은 초보자를 위해 Oracle database 를 Docker 에서 세팅하는 방법을 글 최하단에 첨부하겠습니다.

먼저 앞서 실행했던 `Card` 클래스 그대로, Oracle database로 변경만 해서 실행한 결과 아래와 같이 clob 타입으로 설정되는 것을 볼 수 있습니다.

>Hibernate: create table card (id number(19,0) generated as identity, `back varchar2(256 char)` not null, `front clob` not null, learner number(19,0), primary key (id))

그런데 @Lob은 딱봐도 Oracle database 에 초점을 맞춘 annotation 같은 느낌이 듭니다.

### @Lob만 단독으로 해보기

front에서 @Column annotation을 지우고 다시 해봤습니다.

```java
@Lob
private String front;
```

>Hibernate: create table card (id number(19,0) generated as identity, learner number(19,0), back varchar2(256 char) not null, `front clob`, primary key (id))

역시나 clob으로 잘 됩니다.

심지어 length를 기본값인 255로 설정해도 clob으로 설정됩니다.

이거 하나만 가지고 판단하긴 좀 그렇지만, JPA는 Oracle database 와 궁합이 잘 맞지 않을까 싶습니다. JPA가 Oracle 손아귀에 오래 있었으니 당연한 이야기인가...

### @Column(columnDefinition = "TEXT")

설치한김에 Oracle database에서 @Column(columnDefinition = "TEXT") 를 사용해보면, 아래와 같이 부적합하다는 메시지를 볼 수 있습니다.

> org.hibernate.tool.schema.spi.CommandAcceptanceException: Error executing DDL "create table card (id number(19,0) generated as identity, back varchar2(256 char) not null, front TEXT, learner number(19,0), primary key (id))" via JDBC [ORA-00902: 데이터유형이 부적합합니다]

## Outro

TEXT 타입에 대한 이식성을 보장하자면, @Lob과 @Column(length = 256) 을 같이 선언해주는게 좋을 것 같습니다. 그런데 TEXT 타입에도 length를 지정할 수 있다고 문서에 써있었는데, TEXT 타입에 length를 지정하려면 @Column(columnDefinition = "TEXT(600)") 처럼 사용할 수 밖에 없겠습니다. TEXT 타입에 굳이 길이를 지정할 일이 있는지는 아직 경험이 부족해서 잘 모르겠습니다. GPT-3.5는 TEXT 타입은 길이 지정 안된다고 우기네요.

MySQL 문서([링크](https://dev.mysql.com/doc/refman/8.0/en/column-count-limit.html#:~:text=Row%20Size%20Limit%20Examples))에 관련 예제가 있기는 합니다. 예제에서 VARCHAR 타입은 하나의 row에 사용할 수 있는 최대 사이즈가 65,535 bytes 여서, TEXT 타입을 같이 섞어서 사용합니다.

```sql
mysql> CREATE TABLE t (a VARCHAR(10000), b VARCHAR(10000),
       c VARCHAR(10000), d VARCHAR(10000), e VARCHAR(10000),
       f VARCHAR(10000), g VARCHAR(6000)) ENGINE=InnoDB CHARACTER SET latin1;
ERROR 1118 (42000): Row size too large. The maximum row size for the used
table type, not counting BLOBs, is 65535. This includes storage overhead,
check the manual. You have to change some columns to TEXT or BLOBs

mysql> CREATE TABLE t (a VARCHAR(10000), b VARCHAR(10000),
       c VARCHAR(10000), d VARCHAR(10000), e VARCHAR(10000),
       f VARCHAR(10000), g TEXT(6000)) ENGINE=InnoDB CHARACTER SET latin1;
Query OK, 0 rows affected (0.02 sec)
```

개인적으로는 MySQL도 @Lob만 사용하면 TEXT 타입으로 하고, @Column 을 같이 사용할 때 length 속성이 TEXT 타입의 길이가 되는게 맞지 않을까 싶기는 한데, 이제 막 써본 상태에서 결론을 내리기는 어려울 것 같습니다.

그리고 아래는 앞서 말씀드린대로 Docker와 Oracle database 처음 써보시는 분들을 위해 세팅 방법을 추가로 첨부합니다.

뭔가 뻘짓을 거하게 한 것 같지만, 미루고 미루던 Docker 사용을 해봤으니 조금이나마 위로가 됩니다.

## Oracle Database 세팅 방법

제가 Windows 11을 사용하므로, Windows 11을 기준으로 설명드립니다.

개발자용으로 Oracle database는 23c를 무료로 제공한다는 글([링크](https://blogs.oracle.com/database/post/oracle-database-23c-free))이 있어서, 다운로드 페이지([링크](https://www.oracle.com/database/free/download/))를 참고하여 Docker image를 다운받았습니다.

Docker 가 설치되어 있지 않다면, Docker Desktop([링크](https://www.docker.com/products/docker-desktop/)) 을 설치합니다. 설치하고 나면 Windows 에서도 powershell 이나 command prompt 에서 docker 명령어를 사용할 수 있습니다.

### docker image 다운로드

아래와 같은 명령어를 입력하여 docker image를 다운받습니다.

```
docker pull container-registry.oracle.com/database/free
```

![docker image 다운로드 화면](/assets/img/2023-07-15-mysql-set-text-type-in-jpa-not-using-column-columndefinition-text/01-docker-pull.png)

### docker container 실행

아래와 같이 입력하여 container를 실행시킵니다. 저는 8080 port로 변경했는데, 과거에 oracle 설치하다가 취소했을 때 제대로 롤백이 안됐는지 1521 port가 충돌이 발생해서 8080 port로 설정했습니다.

```
docker run -p 8080:1521 container-registry.oracle.com/database/free:latest
```

### Oracle database 접속

docker 에 terminal로 접속하려면 또 설정해야할게 많으니, 기존에 세팅이 되어있으신 분이 아니라면 docker desktop 에서 container를 선택하고, terminal 메뉴로 이동합니다. container 실행 설정을 따로 안해놓으면 이름이 랜덤으로 설정되기 때문에, container 이름은 다를 수도 있습니다.

![docker desktop 에서 container 선택](/assets/img/2023-07-15-mysql-set-text-type-in-jpa-not-using-column-columndefinition-text/02-docker-desktop-containers.png)

![terminal 메뉴 위치를 강조한 화면 캡쳐](/assets/img/2023-07-15-mysql-set-text-type-in-jpa-not-using-column-columndefinition-text/03-docker-desktop-terminal.png)

그러면 Oracle database를 처음 사용해보는 사람은 뭘 해야하는지 막막해집니다.

참고자료([링크](https://stackoverflow.com/questions/25741027/default-username-and-password-for-oracle-database))를 통해 알아낸 방법을 보겠습니다.

terminal에 아래와 같은 명령어를 입력해서 데이터베이스에 접속합니다.

```
sqlplus / as sysdba
```

### 사용자 생성 및 권한 부여

데이터베이스 접속 이후에는 아래 명령어를 입력하여 사용자를 만들어줍니다.

```sql
CREATE USER <username> IDENTIFIED BY <password>;
!-- 예) CREATE USER C##limvik IDENTIFIED BY limvik;
```

예를 들어드린 CREATE 문장에서 C## 을 붙인 이유는 안붙이면 아래 에러가 떠서 그렇습니다. 당장 Oracle을 주력으로 사용할 것도 아니고, 테스트용이니 하라는대로 합니다.

>ORA-65096: common user or role name must start with prefix C##

그리고 권한을 몽땅 줍니다.

```sql
GRANT ALL PRIVILEGES TO <username>;
!-- 예) GRANT ALL PRIVILEGES TO C##limvik
```

그러면 Grant succeeded. 라는 문장을 볼 수 있습니다.

### Spring Boot 설정

다음은 Spring Boot 에서 properties를 설정할때는 아래와 같이 해줍니다. 위에서 생성한 사용자 예제를 기준으로 작성하였습니다.

```
spring.datasource.url=jdbc:oracle:thin:@localhost:8080:FREE
spring.datasource.username=C##limvik
spring.datasource.password=limvik
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
```

url에서 FREE를 써야할 곳에 구버전의 ORACLE_SID인 XE(eXpress Edition) 사용하지 않도록 주의합니다.
