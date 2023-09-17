---
layout: post
title: VARCHAR(255) 그냥 써도 문제 없을까?
categories:
- Database
- MySQL
tags:
- Database
- MySQL
date: 2023-09-08 12:23 +0900
---
## Intro

이전에 테이블 설계했던 것을 회고하는 글([링크](/posts/review-design-for-table/))을 작성하고, 왜 VARCHAR(255)를 사용하는지 알아보기로 했었습니다. 새로운 개인 프로젝트에서 데이터베이스 설계 전에 알아볼 겸 정리합니다.

그런데 명확한 근거를 찾기는 쉽지가 않습니다. 대체로 카더라 식의 정보가 많았습니다. 또 데이터베이스 제품마다 근거가 달라질 수 있어 제가 사용하는 MySQL 위주로 찾아보았습니다.

자료를 찾아본 결과로는 MySQL 최신버전을 사용 중이고, innoDB 를 storage engine으로 사용하면서 설정을 건든게 없다면 그냥 VARCHAR(255) 써도 일반적인 상황에서는 별 문제가 없습니다.

뭔가 단서를 많이 붙였는데, 아래에 제가 궁금해서 찾아본 자료들을 참고해보시면 좋겠습니다.

## 왜 VARCHAR(255)로 설정하나요?

### 역사적인 이유

과거 MySQL 5 버전 이전에는 VARCHAR(255) 가 최대값이었고(대다수의 RDBMS도 비슷했다고 합니다), 길이를 확정하기 어려울 때 최대값으로 지정하였습니다.

### 인덱스(Index)

VARCHAR, TEXT, BLOB과 같은 가변길이 데이터타입(Data Type)은 정확한 크기를 알 수 없기도 하고, 크기가 너무 커서 인덱스를 설정할 수 없는 경우가 있습니다. 그래서 데이터의 앞 부분 일부만 고정된 길이로 인덱스(index key prefix)를 설정하는 기능을 지원합니다.

아래 SQL은 customer 테이블의 name 컬럼(nonbinary string type)에 앞 10글자만 인덱스로 지정하는 예제입니다.

```sql
CREATE INDEX part_of_name ON customer (name(10));
```
출처: [https://dev.mysql.com/doc/refman/8.0/en/create-index.html](https://dev.mysql.com/doc/refman/8.0/en/create-index.html)

과거에는 innoDB 사용시 prefix 최대값(index key prefix length limit)이 767 bytes 였습니다. 이는 UTF-8 사용 시 한 글자가 최대 3bytes 이고 255글자 x 3bytes = 765 bytes 가 되어 전부 인덱스로 지정할 수 있지만, 256 x 3 = 768 bytes 이므로 제한된 크기를 넘어가게 됩니다. 

이후 utf8mb4 등 4bytes 인 문자로 인해 더 큰 크기의 지원이 필요해졌고, innoDB 사용시 prefix 최대값은 row format 설정에 따라 다르긴 하지만 기본 설정값은 3072 bytes 로 증가하였습니다.

> inno db row format 상세 내용 문서([링크](https://dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html))  

### 정리

결과적으로 과거 데이터베이스의 제한으로 인해 사용하던 값이 관례로 굳어져서 사용되는 것으로 보입니다.

### 참고 자료
- [Is there a good reason I see VARCHAR(255) used so often (as opposed to another length)?](https://stackoverflow.com/questions/1217466/is-there-a-good-reason-i-see-varchar255-used-so-often-as-opposed-to-another-l)

## 성능 상의 문제는 없나요?

설정에 따라 달라질 수 있습니다.

Subquery 또는 Window Function을 사용하는 등 여러 상황에서 임시 테이블을 만들 때 설정에 따른 영향을 받을 수 있습니다.

### Internal Temporary Table Use in MySQL

Subquery, Window Function 등의 작업 시 임시 테이블이 만들어지는데 이때 두 종류의 임시 테이블이 있습니다. 하나는 `in-memory internal temporary table`이고, 다른 하나는 `on-disk internal temporary table` 입니다. 이름에 무엇인지 설명이 나와있으므로 별도의 설명은 생략하겠습니다.

#### in-memory internal temporary table

MySQL에서 in-memory internal temporary table 을 관리하는 storage engine으로는 `TempTable`과 `MEMORY` 가 있습니다.

`MEMORY` storage engine 사용 시에는 VARCHAR 에 설정한 최대 길이까지 남는 부분을 채워서 고정 길이의 CHAR로 만듭니다. 그래서 임시 테이블을 만들 때 무조건 VARCHAR 최대 길이만큼 메모리를 사용하게 돼서 자원을 비효율적으로 사용하게 됩니다.

`TempTable` storage engine은 MySQL 8 버전에서 도입됐으며, MEMORY storage engine 과는 달리 최대 길이까지 남는 부분을 채우는 패딩을 하지 않고 VARCHAR 등 가변길이 데이터타입을 효율적으로 저장합니다.

MySQL 8 에서 기본값은 `TempTable` 이므로 굳이 바꾸지 않는 이상은 고려할 필요가 없겠습니다.

> internal_tmp_mem_storage_engine 설정값 문서([링크](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_internal_tmp_mem_storage_engine))

#### on-disk internal temporary table

MySQL 에서 on-disk internal temporary table을 관리하는 storage engine으로는 `innoDB` 와 `MyISAM`이 있습니다.

innoDB 와 MyISAM 에서는 MEMORY storage engine 처럼 VARCHAR 의 최대 길이로 고정해서 사용하도록 설정할 수도 있고, TempTable 처럼 고정하지 않고 메모리를 효율적으로 사용하도록 설정할 수도 있습니다.

innoDB의 경우 row format을 설정하여 이를 전환할 수 있는데, 기본값(DYNAMIC)이 메모리를 효율적으로 사용하는 설정입니다.

> inno db row format 설정값 문서([링크](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_default_row_format))  
> inno db row format 상세 내용 문서([링크](https://dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html))  

MySQL 8.0.15까지는 CTE를 사용하는 경우 등 `MyISAM`을 강제하는 경우도 있지만, MySQL 8.0.16 부터는 `innoDB` 외에는 사용할 수 없습니다. 최신 버전 사용시에는 row format만 잘 고려하면 되겠습니다.

> on-disk 만 사용되는 경우 등 자세한 내용은 레퍼런스 문서([링크](https://dev.mysql.com/doc/refman/8.0/en/internal-temporary-tables.html))를 참고해주세요.

### 예외적인 경우의 성능 문제

Optimizing Data Size 문서([링크](https://dev.mysql.com/doc/refman/8.0/en/data-size.html#:~:text=in%20every%20column.-,Row%20Format,-InnoDB%20tables%20are))를 보면, 아래와 같은 내용이 있습니다.

>The compact family of row formats, which includes `COMPACT`, `DYNAMIC`, and `COMPRESSED`, decreases row storage space at the cost of increasing CPU use for some operations. If your workload is a typical one that is limited by cache hit rates and disk speed it is likely to be faster. If it is a rare case that is limited by CPU speed, it might be slower.

innoDB 의 row format 설정값이 기본값인 DYNAMIC을 포함해서 COMPACT 이거나 COMPRESSED 일 때, row 저장 공간을 절약하기 위해 CPU 작업이 증가하게 됩니다. 작업이 cache hit rates 나 disk speed 에 의해 제한되는 경우 더 빨라지지만, CPU 속도에 의해서 제한되는 작업의 경우 더 느려질 수도 있습니다.

CPU 속도가 문제가 되는 수준의 연산 작업을 하는거면, 빅데이터 쪽으로 넘어가야 하지 않을까... 어렴풋이 추측해봅니다.

### 참고 자료
- [Are there disadvantages to using a generic varchar(255) for all text-based fields?](https://stackoverflow.com/questions/262238/are-there-disadvantages-to-using-a-generic-varchar255-for-all-text-based-field)
- [MySQL - varchar length and performance](https://dba.stackexchange.com/questions/76469/mysql-varchar-length-and-performance)

## 결론

시작부터 결론을 말씀드리기는 했지만, `innoDB 외의 다른 storage engine 또는 다른 데이터베이스를 사용하는 경우는 전혀 고려되지 않은 결론`이었습니다. 최대한 데이터 길이에 맞게 잘 설정하는게 가장 좋기는 하겠지만, 그게 쉽지 않으니까 과거부터 VARCHAR(255)를 사용해 왔겠죠? 사용하는 데이터베이스의 매뉴얼을 잘 보고 결정해야겠습니다. 다른 데이터베이스로 변경할 때 좀 괴롭기는 하겠습니다.

저는 MySQL을 사용하는 동안 설정을 변경할 이유가 없고, innoDB를 storage engine으로 사용하는 경우, 길이 설정이 애매할 때 VARCHAR(255)를 사용할 계획입니다.

## Outro

VARCHAR(255) 써도 되는거야 마는거야 정도만 알고 싶은 마음과, 더 깊게 알고 싶은 마음이 오락가락해서 내용의 깊이도 뭔가 애매한거 같습니다.

더 깊이있는 내용이 필요할 때 위에 있는 키워드로 더 찾아보는걸로 하고, 이만 마무리하겠습니다.
