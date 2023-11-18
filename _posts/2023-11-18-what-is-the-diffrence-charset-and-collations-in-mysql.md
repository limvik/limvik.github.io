---
layout: post
title: Character Sets와 Collations의 차이
categories:
- Database
- MySQL
tags:
- Database
- MySQL
- Collations
date: 2023-11-18 20:40 +0900
---
## Intro

MySQL을 사용하면서 Collation을 지정할 때, 한글을 사용한다면 `utf8mb4_unicode_ci`를 사용하는게 좋다고, 같이 팀프로젝트 하던 팀원의 이야기를 듣고서 여태 큰 고민없이 사용해왔습니다.

그런데 Collation의 뜻 조차 잘 모른다는 것을 최근에야 인지했습니다. 그래서 Collation의 의미를 찾아보고, MySQL Manual에 있는 Character Sets 와 Collations에 대해 살펴보면서 팀원의 말이 맞았던건지도 살펴보겠습니다.

## MySQL Manual

개요 페이지인 [Chapter 10 Character Sets, Collations, Unicode](https://dev.mysql.com/doc/refman/8.2/en/charset.html) 부터 살펴보겠습니다. 궁금한거 위주로만 간단하게 짚어보면서 넘어갈 예정이라, 자세한 내용은 Manual을 참고해주세요!

### MySQL의 Character Sets, Collations 기본값

> The default MySQL server character set and collation are `utf8mb4` and `utf8mb4_0900_ai_ci`, but you can specify character sets at the server, database, table, column, and string literal levels.

기본값은 Character Set의 경우 `utf8mb4`, Collation의 경우 `utf8mb4_0900_ai_ci` 입니다.

`컬럼이나 문자열 리터럴 수준에서도 별도로 지정할 수 있다`는 것은 처음 알게 됐습니다.

```sql
# column levels
CREATE TABLE t1
(
    col1 VARCHAR(5)
      CHARACTER SET latin1
      COLLATE latin1_german1_ci
);

ALTER TABLE t1 MODIFY
    col1 VARCHAR(5)
      CHARACTER SET latin1
      COLLATE latin1_swedish_ci;

# string literal levels
SELECT 'abc';
SELECT _latin1'abc';
SELECT _binary'abc';
SELECT _utf8mb4'abc' COLLATE utf8mb4_danish_ci;
```

### Character Sets과 Collations 의 의미

Character Sets는 인코딩과 그에 해당하는 문자를 의미하는 것을 알겠지만, Collations는 단어 자체가 익숙하지 않습니다.

네이버 사전에서 Collation을 찾아보면 눈에들어오는 뜻이 몇 가지 있습니다.

1. (원문 따위와의) 대조 (확인).
2. (서적의) 페이지 순서 조사, 낙장(落張) 조사.

출판쪽에서 주로 사용되는 용어가 아닌가 싶습니다. 페이지를 비교하면서 페이지 순서가 맞는지 조사하는 느낌?

Manual([링크](https://dev.mysql.com/doc/refman/8.2/en/charset-general.html))로 다시 돌아가보면, Collation은 문자열 비교를 위해 사용됩니다.

> A character set is a set of symbols and encodings. A collation is a set of rules for comparing characters in a character set.

character set은 symbols와 encodings의 집합이고, collation은 character set 에서 `characters 를 비교하기 위한 규칙의 집합`이라고 적혀있습니다.

이어서 Manual의 예제를 살펴보겠습니다. 아래부터는 원문이 굳이 필요하지 않으면 생략하겠습니다.

> 4개의 글자가 있는 알파벳이 있다고 가정해 보겠습니다: A, B, a, b. 각 문자에 숫자를 부여합니다: A = 0, B = 1, a = 2, b = 3입니다. 문자 A는 Symbol이고 숫자 0은 A의 Encoding이며 4개의 문자와 Encoding을 모두 조합한 것이 Character Set입니다.  
>
> 두 문자열 값인 A와 B를 비교하고 싶다고 가정해 보겠습니다. 가장 간단한 방법은 Encoding을 살펴보는 것입니다: 0은 1보다 작으므로 A가 B보다 작다고 말합니다. 방금 한 일은 Character Set에 Collation을 적용하는 것입니다. Collation은 규칙 집합입니다(이 경우 규칙은 하나뿐입니다): "Encoding을 비교한다." 가능한 모든 Collation 중 가장 간단한 Collation을 Binary Collation이라고 부릅니다.

나머지 내용은 현실은 이렇게 만만하지 않지만, MySQL은 잘 해결할 수 있음을 언급하고 있습니다.

### MySQL에서 지원하는 Character Set과 Collations 확인 방법

그 다음 페이지인 [10.2 Character Sets and Collations in MySQL](https://dev.mysql.com/doc/refman/8.2/en/charset-mysql.html)은 간단하게 살펴보겠습니다. Character Set 과 Collation 보는 방법을 다룹니다.

```sql
SHOW CHARACTER SET;
SHOW CHARACTER SET  LIKE  'utf%';
SHOW COLLATION WHERE Charset  =  'utf8mb4';
```

그리고 여러가지 이야기와 함께 조언으로 마무리 합니다.

> 부적절한 Collation을 선택하지 않으려면 대표적인 데이터 값과 몇 가지 비교를 수행하여 주어진 Collation이 예상한 방식으로 값을 정렬하는지 확인합니다.

그러면 MySQL 기본값인 `utf8mb4_0900_ai_ci` 와 제가 한글을 사용하기 위해 주로 사용하는 `utf8mb4_unicode_ci` 를 한번 예상한 방식으로 값을 정렬하는지 비교해보겠습니다.

## utf8mb4_unicode_ci vs utf8mb4_0900_ai_ci 정렬 비교

### Table 생성

#### utf8mb4_unicode_ci

```sql
CREATE TABLE test.test1 (
    id INT auto_increment PRIMARY KEY,
    name varchar(255) NULL
)
ENGINE=InnoDB
DEFAULT CHARSET=utf8mb4
COLLATE=utf8mb4_unicode_ci;
```

#### utf8mb4_0900_ai_ci

```sql
CREATE TABLE test.test2 (
    id INT auto_increment PRIMARY KEY,
    name varchar(255) NULL
)
ENGINE=InnoDB
DEFAULT CHARSET=utf8mb4
COLLATE=utf8mb4_0900_ai_ci;
```

### 데이터 삽입

```sql
mysql> INSERT INTO test1 (name) VALUES ('가나다'),('ㄱㅏ나다'),('ㄱㅏㄴㅏㄷㅏ');
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> INSERT INTO test1 (name) VALUES ('라마바'),('ㄹㅏ마바'),('ㄹㅏㅁㅏㅂㅏ');
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> INSERT INTO test1 (name) VALUES ('가나다'),('ㄱㅏ나다'),('ㄱㅏㄴㅏㄷㅏ');
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> INSERT INTO test2 (name) VALUES ('라마바'),('ㄹㅏ마바'),('ㄹㅏㅁㅏㅂㅏ');
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

### 데이터 조회

#### utf8mb4_unicode_ci

```sql
mysql> SELECT * FROM test1;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 가나다       |
|  2 | ㄱㅏ나다     |
|  3 | ㄱㅏㄴㅏㄷㅏ |
|  4 | 라마바       |
|  5 | ㄹㅏ마바     |
|  6 | ㄹㅏㅁㅏㅂㅏ |
+----+--------------+
6 rows in set (0.00 sec)
```

#### utf8mb4_0900_ai_ci

```sql
mysql> SELECT * FROM test2;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | 가나다       |
|  2 | ㄱㅏ나다     |
|  3 | ㄱㅏㄴㅏㄷㅏ |
|  4 | 라마바       |
|  5 | ㄹㅏ마바     |
|  6 | ㄹㅏㅁㅏㅂㅏ |
+----+--------------+
6 rows in set (0.00 sec)
```

엄... 잘 되는데...? 뭐가 문제라서 한글을 사용할 때 다른 Collation을 사용하는 걸까요?

앞의 데이터는 utf8mb4_`general`\_ci 와 비교를 해주신 감사한 분의 글([링크](https://rastalion.me/mysql-8-0-1-%EB%B2%84%EC%A0%84%EB%B6%80%ED%84%B0-%EC%B1%84%ED%83%9D%EB%90%9C-utf8mb4_0900_ai_ci%EC%9D%98-%ED%95%9C%EA%B8%80-%EC%82%AC%EC%9A%A9%EC%97%90-%EB%8C%80%ED%95%9C-%EB%AC%B8%EC%A0%9C%EC%A0%90/))에서 가져왔고, 다른 문제를 언급해주셔서 제가 주로 사용했던 utf8mb4_`unicode`_ci 실험으로 아래 소개드립니다.

### utf8mb4_unicode_ci 데이터 조회

이상 없이 원하는대로 결과가 나오는 것을 볼 수 있습니다.

```sql
mysql> SELECT id, name, length(name), hex(name) FROM test1
    -> WHERE name='가나다';
+----+--------+--------------+--------------------+
| id | name   | length(name) | hex(name)          |
+----+--------+--------------+--------------------+
|  1 | 가나다 |            9 | EAB080EB8298EB8BA4 |
+----+--------+--------------+--------------------+
1 row in set (0.00 sec)

mysql> SELECT id, name, length(name), hex(name) FROM test1
    -> WHERE name='ㄱㅏ나다';
+----+----------+--------------+--------------------------+
| id | name     | length(name) | hex(name)                |
+----+----------+--------------+--------------------------+
|  2 | ㄱㅏ나다 |           12 | E384B1E3858FEB8298EB8BA4 |
+----+----------+--------------+--------------------------+
1 row in set (0.00 sec)

mysql> SELECT id, name, length(name), hex(name) FROM test1
    -> WHERE name='ㄱㅏㄴㅏㄷㅏ';
+----+--------------+--------------+--------------------------------------+
| id | name         | length(name) | hex(name)                            |
+----+--------------+--------------+--------------------------------------+
|  3 | ㄱㅏㄴㅏㄷㅏ |           18 | E384B1E3858FE384B4E3858FE384B7E3858F |
+----+--------------+--------------+--------------------------------------+
1 row in set (0.00 sec)
```

### utf8mb4_0900_ai_ci 데이터 조회

앞에서와 동일하게 조회했는데, `가나다`, `ㄱㅏ나다`, `ㄱㅏㄴㅏㄷㅏ`를 모두 출력하는 것을 볼 수 있습니다.

```sql
mysql> SELECT id, name, length(name), hex(name) FROM test2
    -> WHERE name='가나다';
+----+--------------+--------------+--------------------------------------+
| id | name         | length(name) | hex(name)                            |
+----+--------------+--------------+--------------------------------------+
|  1 | 가나다       |            9 | EAB080EB8298EB8BA4                   |
|  2 | ㄱㅏ나다     |           12 | E384B1E3858FEB8298EB8BA4             |
|  3 | ㄱㅏㄴㅏㄷㅏ |           18 | E384B1E3858FE384B4E3858FE384B7E3858F |
+----+--------------+--------------+--------------------------------------+
3 rows in set (0.00 sec)

mysql> SELECT id, name, length(name), hex(name) FROM test2
    -> WHERE name='ㄱㅏ나다';
+----+--------------+--------------+--------------------------------------+
| id | name         | length(name) | hex(name)                            |
+----+--------------+--------------+--------------------------------------+
|  1 | 가나다       |            9 | EAB080EB8298EB8BA4                   |
|  2 | ㄱㅏ나다     |           12 | E384B1E3858FEB8298EB8BA4             |
|  3 | ㄱㅏㄴㅏㄷㅏ |           18 | E384B1E3858FE384B4E3858FE384B7E3858F |
+----+--------------+--------------+--------------------------------------+
3 rows in set (0.00 sec)

mysql> SELECT id, name, length(name), hex(name) FROM test2
    -> WHERE name='ㄱㅏㄴㅏㄷㅏ';
+----+--------------+--------------+--------------------------------------+
| id | name         | length(name) | hex(name)                            |
+----+--------------+--------------+--------------------------------------+
|  1 | 가나다       |            9 | EAB080EB8298EB8BA4                   |
|  2 | ㄱㅏ나다     |           12 | E384B1E3858FEB8298EB8BA4             |
|  3 | ㄱㅏㄴㅏㄷㅏ |           18 | E384B1E3858FE384B4E3858FE384B7E3858F |
+----+--------------+--------------+--------------------------------------+
3 rows in set (0.00 sec)
```

Collation은 이처럼 단순히 비교에만 사용되는 것은 아니라는 것을 알 수 있습니다.

그럼 utf8mb4_`unicode`\_ci 와 utf8mb4_`general`_ci는 어떤 차이가 있을까요? 

## utf8mb4_unicode_ci vs utf8mb4_general_ci

잘 정리해주신 분의 글([링크](https://sir.kr/pg_tip/17446))이 있어서 참고해보면, 대표적으로 정렬 부분에서의 차이는 다음과 같습니다.

> utf8mb4_unicode_ci는 문자열을 정렬할 때 `유니코드 코드 포인트`를 기준으로 하여 정렬합니다. 이는 각 문자의 고유한 식별자를 기반으로 정렬하므로, 다양한 언어와 문자를 `정확하게 정렬`할 수 있습니다. 반면 utf8mb4_general_ci는 문자의 `바이트 시퀀스`를 기준으로 정렬하며, 일부 언어나 문자의 정렬 순서가 제대로 반영되지 않을 수 있습니다.

정확한 정렬을 할 수 있다면 당연히 그만큼 상대적으로 많은 자원을 소모하게 됩니다.

글에서 나온 예로는 프랑스어나 독일어에서의 문제점을 지적해주셨는데, 글로벌 서비스라면 고민해볼 문제로 보입니다.

해외의 다른 의견([링크](https://devpress.csdn.net/mysqldb/62fb7a08c677032930800294.html))으로는  utf8mb4_general_ci가 애초에 컴퓨터 처리속도가 굉장히 느린 시절에 정렬의 정확성을 포기하고 단순화 시킨 Collation이기 때문에 최신 컴퓨터에서는 무시할 수 있는 수준이고, 둘 중에 고르라면 utf8mb4_unicode_ci를 쓰는게 낫다는 의견이 있습니다.

그런데 이미 한국에서 utf8mb4_general_ci 는 많이 검증이 됐으므로, 한국향 서비스라면 utf8mb4_general_ci 를 사용하는게 나을 것으로 생각됩니다.

그런데 utf8mb4_general_ci나 utf8mb4_unicode_ci 모두 옛날 UCA(Unicode Collation Algorithm)을 사용하여 성능이 utf8mb4_0900_ai_ci에 비해서 좋지 않다고 합니다.

## 다시 MySQL Manual

다시 MySQL Manual로 돌아가보면, 아래와 같이 언급하고 있습니다.

> MySQL은 http://www.unicode.org/reports/tr10/ 에 설명된 UCA(Unicode Collation Algorithm)에 따라 xxx_unicode_ci Collation을 구현합니다. 이 Collation은 버전 4.0.0 UCA 가중치 키(http://www.unicode.org/Public/UCA/4.0.0/allkeys-4.0.0.txt)를 사용합니다. xxx_unicode_ci Collation은 Unicode Collation Algorithm을 부분적으로만 지원합니다. 일부 문자가 지원되지 않으며 결합 마크가 완전히 지원되지 않습니다. 이는 베트남어, 요루바어, 나바호어와 같은 언어에 영향을 줍니다. 결합된 문자는 문자열 비교에서 단일 유니코드 문자로 작성된 동일한 문자와 다른 것으로 간주되며, 두 문자의 길이가 다른 것으로 간주됩니다(예: CHAR_LENGTH() 함수에서 반환되거나 결과 집합 메타데이터에서 반환되는 경우).

결합된 문자라는게 뭔지 이해는 안되지만, char_length()를 출력해보면 한글은 utf8mb4_unicode_ci와 utf8mb4_0900_ai_ci 에서 모두 동일합니다.

```sql
mysql> SELECT id, name, length(name), char_length(name), hex(name) FROM test1;
+----+--------------+--------------+-------------------+--------------------------------------+
| id | name         | length(name) | char_length(name) | hex(name)                            |
+----+--------------+--------------+-------------------+--------------------------------------+
|  1 | 가나다       |            9 |                 3 | EAB080EB8298EB8BA4                   |
|  2 | ㄱㅏ나다     |           12 |                 4 | E384B1E3858FEB8298EB8BA4             |
|  3 | ㄱㅏㄴㅏㄷㅏ |           18 |                 6 | E384B1E3858FE384B4E3858FE384B7E3858F |
|  4 | 라마바       |            9 |                 3 | EB9DBCEBA788EBB094                   |
|  5 | ㄹㅏ마바     |           12 |                 4 | E384B9E3858FEBA788EBB094             |
|  6 | ㄹㅏㅁㅏㅂㅏ |           18 |                 6 | E384B9E3858FE38581E3858FE38582E3858F |
+----+--------------+--------------+-------------------+--------------------------------------+
6 rows in set (0.00 sec)

mysql> SELECT id, name, length(name), char_length(name), hex(name) FROM test2;
+----+--------------+--------------+-------------------+--------------------------------------+
| id | name         | length(name) | char_length(name) | hex(name)                            |
+----+--------------+--------------+-------------------+--------------------------------------+
|  1 | 가나다       |            9 |                 3 | EAB080EB8298EB8BA4                   |
|  2 | ㄱㅏ나다     |           12 |                 4 | E384B1E3858FEB8298EB8BA4             |
|  3 | ㄱㅏㄴㅏㄷㅏ |           18 |                 6 | E384B1E3858FE384B4E3858FE384B7E3858F |
|  4 | 라마바       |            9 |                 3 | EB9DBCEBA788EBB094                   |
|  5 | ㄹㅏ마바     |           12 |                 4 | E384B9E3858FEBA788EBB094             |
|  6 | ㄹㅏㅁㅏㅂㅏ |           18 |                 6 | E384B9E3858FE38581E3858FE38582E3858F |
+----+--------------+--------------+-------------------+--------------------------------------+
6 rows in set (0.00 sec)
```

이어서 보겠습니다.

> 4.0.0 보다 높은 UCA 버전을 기반으로 하는 Unicode Collation에는 Collation 이름에 해당 버전이 포함됩니다. 예시:
>
> -   `utf8mb4_unicode_520_ci`  is based on UCA 5.2.0 weight keys ([http://www.unicode.org/Public/UCA/5.2.0/allkeys.txt](http://www.unicode.org/Public/UCA/5.2.0/allkeys.txt)),
>    
> -   `utf8mb4_0900_ai_ci`  is based on UCA 9.0.0 weight keys ([http://www.unicode.org/Public/UCA/9.0.0/allkeys.txt](http://www.unicode.org/Public/UCA/9.0.0/allkeys.txt)).
>    
> [`LOWER()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_lower) 및 [`UPPER()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_upper) 함수는 인수의 Collation에 따라 case folding을 수행합니다. 대문자와 소문자 버전이 있는 문자는 인수 Collation이 충분히 높은 UCA 버전(4.0.0 보다 높은 버전)을 사용하는 경우에만 이 함수에 의해 변환됩니다.

아직 이해되지 않는 내용이 좀 있기는 하지만, 사용할 수만있다면 기본값인 `utf8mb4_0900_ai_ci` 을 사용하는게 좋아보입니다.

`euckr_korean_ci` 이라는 한국어 전용도 있지만, 옛날에는 윈도우 쓰면서도 문자가 깨져서 euckr을 utf8으로 일단 바꾸고 시작하는 경우가 많았던 기억이 납니다.

뭔가 이야기가 다른 곳으로 흘러갔지만, 한글을 사용한다면 선택지가 utf8mb4_unicode_ci, utf8mb4_general_ci 가 있다는 점을 기억해야겠습니다.

그런데 사실 `가나다`, `ㄱㅏㄴㅏㄷㅏ` 를 동일하게 취급해도 상관 없다면 기본값인 `utf8mb4_0900_ai_ci` 를 사용해도 되지 않나 싶습니다.

여튼 이정도에서 마무리하고, Collation 이름에서 0900이 UCA 버전을 의미한다는 것을 알게됐는데, 그럼 나머지 뒤에 붙는 것들은 무엇인지 Manual을 살펴보겠습니다.

### Collation Naming Conventions

간단하게 Manual([링크](https://dev.mysql.com/doc/refman/8.0/en/charset-collation-names.html))에 있는 표로 정리하겠습니다.

| Suffix | Meaning |
|--|--|
| `_ai` | Accent-insensitive |
| `_as` | Accent-sensitive |
| `_ci` | Case-insensitive |
| `_cs` | Case-sensitive |
| `_ks` | Kana-sensitive |
| `_bin` | Binary |

ci 가 Case Insensitive 였을 줄이야...

같은 페이지에 있는 UCA 9.0.0 버전과 UCA 4.0.0 버전의 Default Unicode Collation Element Table 이라는 것을 간단하게 보고 마무리하겠습니다.

#### 9.0.0

```
1100  ; [.3BF5.0020.0002] # HANGUL CHOSEONG KIYEOK
3131  ; [.3BF5.0020.0004] # HANGUL LETTER KIYEOK
3200  ; [*0317.0020.0004][.3BF5.0020.0004][*0318.0020.0004] # PARENTHESIZED HANGUL KIYEOK
3260  ; [.3BF5.0020.0006] # CIRCLED HANGUL KIYEOK
FFA1  ; [.3BF5.0020.0012] # HALFWIDTH HANGUL LETTER KIYEOK
320E  ; [*0317.0020.0004][.3BF5.0020.0004][.3C73.0020.0004][*0318.0020.0004] # PARENTHESIZED HANGUL KIYEOK A
326E  ; [.3BF5.0020.0006][.3C73.0020.0006] # CIRCLED HANGUL KIYEOK A
1101  ; [.3BF6.0020.0002] # HANGUL CHOSEONG SSANGKIYEOK
3132  ; [.3BF6.0020.0004] # HANGUL LETTER SSANGKIYEOK
FFA2  ; [.3BF6.0020.0012] # HALFWIDTH HANGUL LETTER SSANGKIYEOK
1102  ; [.3BF7.0020.0002] # HANGUL CHOSEONG NIEUN
3134  ; [.3BF7.0020.0004] # HANGUL LETTER NIEUN
3201  ; [*0317.0020.0004][.3BF7.0020.0004][*0318.0020.0004] # PARENTHESIZED HANGUL NIEUN
3261  ; [.3BF7.0020.0006] # CIRCLED HANGUL NIEUN
FFA4  ; [.3BF7.0020.0012] # HALFWIDTH HANGUL LETTER NIEUN
```

#### 4.0.0

```
1100  ; [.1D62.0020.0002.1100] # HANGUL CHOSEONG KIYEOK
3131  ; [.1D62.0020.0004.3131] # HANGUL LETTER KIYEOK; QQK
3200  ; [*0288.0020.0004.3200][.1D62.0020.0004.3200][*0289.0020.001F.3200] # PARENTHESIZED HANGUL KIYEOK; QQKN
3260  ; [.1D62.0020.0006.3260] # CIRCLED HANGUL KIYEOK; QQK
FFA1  ; [.1D62.0020.0012.FFA1] # HALFWIDTH HANGUL LETTER KIYEOK; QQK
320E  ; [*0288.0020.0004.320E][.1D62.0020.0004.320E][.1DBE.0020.001F.320E][*0289.0020.001F.320E] # PARENTHESIZED HANGUL KIYEOK A; QQKN
326E  ; [.1D62.0020.0006.326E][.1DBE.0020.0006.326E] # CIRCLED HANGUL KIYEOK A; QQKN
1101  ; [.1D63.0020.0002.1101] # HANGUL CHOSEONG SSANGKIYEOK
3132  ; [.1D63.0020.0004.3132] # HANGUL LETTER SSANGKIYEOK; QQK
FFA2  ; [.1D63.0020.0012.FFA2] # HALFWIDTH HANGUL LETTER SSANGKIYEOK; QQK
1102  ; [.1D64.0020.0002.1102] # HANGUL CHOSEONG NIEUN
3134  ; [.1D64.0020.0004.3134] # HANGUL LETTER NIEUN; QQK
3201  ; [*0288.0020.0004.3201][.1D64.0020.0004.3201][*0289.0020.001F.3201] # PARENTHESIZED HANGUL NIEUN; QQKN
3261  ; [.1D64.0020.0006.3261] # CIRCLED HANGUL NIEUN; QQK
FFA4  ; [.1D64.0020.0012.FFA4] # HALFWIDTH HANGUL LETTER NIEUN; QQK
```

9.0.0에서 정보가 줄어들면서 4.0.0 에서는 구분 가능하던 글자가 9.0.0에서는 동일한 글자로 취급되는건가(?) 하는 뇌피셜이 생겨납니다.

## Outro

Collation 이 뭔지 잠깐 살펴보려다 여기까지 와버렸습니다. 손가락 가는대로 정보를 더 찾아보고 싶기는 하지만, 우선순위 높은 일들이 많이 남아있기에 이정도에서 마무리하겠습니다. 라고 적고 마무리 하려고했지만, utf8mb4-unicode-ci 와 utf8mb4-general-ci 성능 차이가 얼마나 날지 궁금해져서 간단하게 다른 사람 벤치마크 자료를 찾아봤습니다. 자세한 내용은 아래 링크를 참고해주세요.

[charset-and-collation-settings-impact-on-mysql-performance](https://www.percona.com/blog/charset-and-collation-settings-impact-on-mysql-performance/)

utf8mb4-unicode-ci < utf8mb4-general-ci < utf8mb4_0900_ai_ci 이기는 한데, unicode-ci와 general-ci는 이걸 유의미하게 볼 수 있는건지는 판단이 잘 서지 않습니다. utf8mb4_0900_ai_ci 와 utf8mb4-unicode-ci 를 비교해보면, `가나다` 랑 `ㄱㅏㄴㅏㄷㅏ` 를 같은 문자로 취급하는 정도는 감수해볼만 할지도...?

Collation을 선택할 때 내가 원하는 조회 및 정렬 결과가 나오는지 테스트해보고, 성능 좋은 Collation을 고르면 된다. 라고 정리하며 글을 정말로 진짜 마무리하겠습니다.
