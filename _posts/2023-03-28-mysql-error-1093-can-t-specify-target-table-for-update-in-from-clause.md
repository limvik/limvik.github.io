---
layout: post
title: MySQL Error 1093 - Can't specify target table for update in FROM clause
categories:
- Database
- MySQL
tags:
- MySQL
- Error
- Subquery
date: 2023-03-28 23:24 +0900
---
## 요약

MySQL Error 1093은 MySQL에서 테이블 수정 시 subquery에 있는 SELECT 문의 FROM 절에서 동일한 테이블을 지정한 경우에 발생합니다.

MySQL은 아래와 같은 구문에 제약이 있습니다.
```sql
DELETE FROM t WHERE ... (SELECT ... FROM t ...);
UPDATE t ... WHERE col = (SELECT ... FROM t ...);
{INSERT|REPLACE} INTO t (SELECT ... FROM t ...);
```
아래는 위 구문의 예외로 제약이 적용되지 않습니다.
```sql
UPDATE t ...  WHERE col =  (SELECT  *  FROM  (SELECT  ...  FROM t...)  AS dt ...);
```
SELF JOIN을 통해 해결할 수도 있습니다.

## Intro

이번주는 MySQL 수업을 듣고 있습니다. 강사님이 MySQL에 익숙하지 않으셔서(?) Error 를 몇 가지 볼 수 있었습니다. 구문만 보면 될 것 같은데, Error가 발생하니 정리해봤습니다.

## MySQL Error 1093

아래는 Error가 발생한 SQL 구문입니다.
```sql
UPDATE emp_copy
SET hire_date=(SELECT hire_date FROM emp_copy WHERE name='jack')
WHERE name='홍길동';
```
emp_copy 테이블을 수정을 시도하면서 subquery에서 SELECT 문을 사용하였고, 해당 SELECT 문의 FROM 절에는 같은 테이블인 emp_copy 를 지정하였습니다.

공식문서에서는 아래와 같이 언급하고 있습니다.

This error occurs in cases such as the following, which attempts to modify a table and select from the same table in the subquery:
```sql
UPDATE t1 SET column2 = (SELECT MAX(column1) FROM t1);
```
In general, you cannot modify a table and select from the same table in a subquery. For example, this limitation applies to statements of the following forms:
```sql
DELETE FROM t WHERE ... (SELECT ... FROM t ...);
UPDATE t ... WHERE col = (SELECT ... FROM t ...);
{INSERT|REPLACE} INTO t (SELECT ... FROM t ...);
```
옛날에는 문서화도 잘 안됐었는지 버그 목록에 올라왔던 기록이 있습니다.([링크](https://bugs.mysql.com/bug.php?id=6980))
## 해결책
공식 문서에서 subquery 제약이 적용되지 않는 예외적인 구문을 아래와 같이 예시로 제시하였습니다.
```sql
UPDATE t ... WHERE col = (SELECT * FROM (SELECT ... FROM t...) AS dt ...);
```
제가 수업 중에 오류가 났던 구문은 아래와 같이 수정할 수 있겠습니다.
```sql
UPDATE emp_copy
SET hire_date=(SELECT * FROM (SELECT hire_date FROM emp_copy WHERE name='jack') as x)
WHERE name='홍길동';
```
SELF JOIN을 이용한 방법도 GPT를 통해 추천받았습니다.
```sql
UPDATE emp_copy e1
JOIN emp_copy e2 ON e2.name='jack'
SET e1.hire_date = e2.hire_date
WHERE e1.name = '홍길동';
```
## Outro
직관적으로 생각했을 때 값이 대입되기 전에 subquery 가 실행돼서 값으로 사용될 수 있을 것 같은데... 안되네요. 왜 그런지는 더 공부해봐야 알 것 같습니다.

그리고 검색을 하다보니 Oracle에서는 이상없이 동작을 한다고 합니다. 강사님이 아무래도 Oracle에 익숙하신 분이신가 봅니다.

## 참고자료
1. [https://dev.mysql.com/doc/refman/8.0/en/subquery-errors.html](https://dev.mysql.com/doc/refman/8.0/en/subquery-errors.html)
2. [https://dev.mysql.com/doc/refman/8.0/en/subquery-restrictions.html](https://dev.mysql.com/doc/refman/8.0/en/subquery-restrictions.html)
3. [https://bugs.mysql.com/bug.php?id=6980](https://bugs.mysql.com/bug.php?id=6980)
