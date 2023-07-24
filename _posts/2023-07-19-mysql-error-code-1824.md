---
layout: post
title: MySQL Error Code 1824. Failed to open the referenced table 'tablename'
categories: [Database, MySQL]
tags: [MySQL]
---
## Intro

기존에 Windows 에 MySQL 을 설치한 상태에서 이상없이 사용하고 있던 CREATE 문이 Linux 에 MySQL을 설치하고 사용하면서 오류가 발생했습니다.

Error Code로 검색해보니 기본적으로 참조 테이블을 못찾았기 때문이고, 구체적으로는 참조 테이블을 정말 만들지 않은 경우나, DB Engine 이 일치하지 않는 경우에 대해 언급하는 경우가 많았습니다.

그런데 저는 참조 테이블을 못찾은 이유가 대소문자 구분 문제 때문이었습니다.

## 요약

- MySQL은 테이블마다 테이블 이름을 갖는 파일을 만들어 저장합니다.
- Windows 는 파일 이름의 대소문자를 구분하지 않는 반면, Linux 는 대소문자를 구분합니다.
- 결론, Linux 사용 시에는 SQL에서 테이블 이름을 적을 때, 대소문자를 신경써야 합니다.

## 기존의 지식

기존에 배우기를 MySQL은 대소문자 구분을 안하기 때문에, 대소문자를 신경쓸 필요가 없다고만 알고 있었습니다.

그래서 저는 보통 개인적인 취향으로, CREATE 문을 작성할 때 테이블의 첫 글자를 대문자로 사용했습니다.

```sql
CREATE TABLE IF NOT EXISTS `Users` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `password` VARCHAR(255) NOT NULL,
    PRIMARY KEY(`id`)
)
```

대소문자를 구분하지 않는다고 알고 있었으니, 다른데는 그냥 편한대로 쓰는 경우가 많았습니다.

```sql
CREATE TABLE IF NOT EXISTS `Example` (
    `user_id` INT NOT NULL,
    PRIMARY KEY(user_id),
    FOREIGN KEY(user_id) REFERENCES users(id) ON DELETE CASCADE
)
```

REFERENCES 뒤에 `u`sers(id) 와 같이 첫 글자를 대문자로 사용하지 않고, 그냥 편한대로 썼습니다.

그러다 이번에 Linux 에 MySQL을 설치하고, Windows에서 사용하던 CREATE 문을 다시 실행시키면서 Error Code 1824를 마주하게 됐습니다.

>Error Code: 1824. Failed to open the referenced table 'users'

## 조사

### 환경 변화

구체적인 환경은 기존에 Windows 11 에 MySQL 8.0.32를 설치하고 사용했었습니다. 그리고 이번에는 CentOS 7.9에서 MySQL 8.0.34를 설치했습니다.

### MySQL 버전의 문제인가?

처음에는 MySQL 버전이 다르니, 뭔가 바뀌었나 해서 [Release note](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-34.html)를 뒤져봤는데, 대소문자와 관련된 내용은 전혀 없었습니다.

### 도와줘 GPT

GPT는 뭐 알려나 싶어서 질문했더니, 대소문자 문제일 수도 있다고 기존의 지식과 배치되는 이야기를 했습니다.

얘가 또 뭔 헛소리하나 싶어서, 조금 더 캐물어봤더니 관련 변수 값을 알려줍니다.

```sql
SHOW VARIABLES LIKE 'lower_case_table_names';
```

실행해보니 Windows 에 설치된 lower_case_table_names 는 `1`, CentOS 에 설치된 값은 `0` 을 출력합니다.

Read Only 변수인데 변수를 변경하는 방법도 알려줘서 시도하다가 MySQL이 실행도 안돼서 당황했었습니다.

그러다가 어디서 봤었는지는 기억이 안나지만, Linux는 파일의 대소문자를 구분하는 반면, Windows는 구분하지 않는다는 것이 갑자기 떠올랐습니다.

### 그럼 테이블 마다 파일을 하나씩 생성하는건가?

Linux와 Windows가 파일명 대소문자 구분하는 방식이 다르다는게 떠오르니, 테이블 마다 파일을 생성하는 건가 싶었습니다. 생각해보니 어떻게 저장될 것인가에 대해서는 생각해본적이 없었습니다.

MySQL이 어디에 테이블 파일을 저장하는지 찾아봤습니다.

Linux 의 경우 기본적으로 테이블이 저장되는 경로는 아래와 같습니다.

>/var/lib/mysql/

Windows의 경우는 아래와 같습니다.

>C:\ProgramData\MySQL\MySQL Server 8.0\Data
>
>Program Files가 아닌 ProgramData 이니 엄한데서 찾지 않으시길...

해당 경로에서 확인해보면 데이터베이스(MySQL은 스키마와 동의어)는 디렉토리로 구분하고, 테이블은 `*.ibd` 확장자를 가진 파일로 저장됩니다.

혹시 경로를 변경했는데, 기억이 안나시는 분은 아래와 같은 SQL을 입력해서 datadir 에 있는 경로를 확인하시면 됩니다.

```sql
SHOW VARIABLES WHERE Variable_Name LIKE "%dir" ;
```

Linux, Windows 모두 확장자가 동일하므로 Linux 만 사진을 첨부하겠습니다.

![Linux Command Line 에서 본 ibd 파일들](/assets/img/2023-07-19-mysql-error-code-1824/01-mysql-table-files-in-linux.png)

위 사진처럼 테이블 마다 파일이 생성됩니다. 그리고 Linux 의 경우 대소문자를 구분하므로, 사진처럼 대소문자를 구분하여 동일한 이름의 테이블을 생성할 수 있습니다.

Windows 에 설치된 MySQL에서 실험해보면 역시나 대소문자를 달리해도 같은 테이블명으로 처리됩니다.

MySQL 은 대소문자를 신경쓸 필요가 없다는 지식은 잘못됐다는 걸 뒤늦게 알았습니다. 작년에 정보처리기사 실기 시험에서 SQL에 대소문자 다르게 적어서 걱정하는 사람들이 있었는데, 모두 정답처리 됐던게 갑자기 기억납니다. 시험에서는 세부적인 제약 조건을 안줬기 때문이겠죠.

## Outro

앞으로는 SQL에 테이블 이름을 적을 때, 소문자로만 작성해야겠습니다.
