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
해결책1. 제약이 적용되지 않는 예외
```sql
UPDATE t ... WHERE col = (SELECT * FROM (SELECT ... FROM t ...) AS dt ...);
```
해결책2. 다중 테이블(multi-table) 사용
```sql
UPDATE /*+ NO_MERGE(discounted) */ t AS a,
(SELECT ... FROM t ... WHERE col = ?) AS b
...
WHERE a.col = b.col ...;
```
Hint 사용하지 않고, WHERE 절을 외부에서 선언하는 방법
```sql
UPDATE t AS a,
(SELECT ... FROM t ...) AS b
...
WHERE a.col = ?
AND b.col = ? ...;
```
해결책3. SELF JOIN
```sql
UPDATE t AS a,
JOIN t AS b ON b.col = ?
...
WHERE a.col = ? ...;
```
## Intro

이번주는 MySQL 수업을 듣고 있습니다. 강사님이 MySQL에 익숙하지 않으셔서(?) Error 를 몇 가지 볼 수 있었습니다. 구문만 보면 잘 작동할 것 같은데, Error가 발생하니 정리해봤습니다.

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
첫번째로는 subquery 제약과 관련된 공식 문서에서 subquery 제약이 적용되지 않는 `예외`적인 구문을 아래와 같이 예시로 제시하였습니다.
```sql
UPDATE t ... WHERE col = (SELECT * FROM (SELECT ... FROM t...) AS dt ...);
```
제가 수업 중에 오류가 났던 구문은 아래와 같이 수정할 수 있겠습니다.
```sql
UPDATE emp_copy
SET hire_date=(SELECT * FROM (SELECT hire_date FROM emp_copy WHERE name='jack') as x)
WHERE name='홍길동';
```

두번째로는 `다중 테이블(multi-table)`을 이용한 방법을 update 문 공식 문서에서 제공하고 있습니다.

제공된 예제를 보면 Hint를 사용하고 있습니다.

기본적으로 subquery를 통해 얻어진 파생 테이블은 optimizer에 의해 가장 바깥에 있는 테이블로 merge가 되기 때문에 `NO_MERGE` Hint를 사용하여 파생 테이블의 materialization을 강제합니다. merge를 해제하면 자동으로 materialization을 수행합니다.
>Materialization은 데이터베이스 쿼리 최적화에서 사용되는 기법 중 하나로, 쿼리의 일부분을 실행하고 그 결과를 일시적으로 저장한 다음 이를 사용하여 쿼리의 나머지 부분을 완료하는 과정입니다. 이렇게 하면 쿼리의 전체 성능을 향상시킬 수 있습니다. by chatGPT
{: .prompt-tip }

```sql
UPDATE /*+ NO_MERGE(discounted) */ items,
       (SELECT id FROM items
        WHERE retail / wholesale >= 1.3 AND quantity < 100)
        AS discounted
    SET items.retail = items.retail * 0.9
    WHERE items.id = discounted.id;
```
제가 오류가 났던 SQL 구문은 아래와 같이 수정할 수 있겠습니다.
```sql
UPDATE /*+ NO_MERGE(y) */ emp_copy x,
(SELECT hire_date FROM emp_copy WHERE name='jack') as y
SET x.hire_date=y.hire_date
WHERE name='홍길동';
```
마찬가지로 다중 테이블을 사용하지만 subquery의 WHERE 절을 바깥으로 뺀 방법도 제시하고 있습니다.

아래와 같이 할 경우 기본적으로 materialization을 수행하기 때문에 `NO_MERGE` hint를 사용할 필요가 없다고 합니다.
```sql
UPDATE items,
       (SELECT id, retail / wholesale AS markup, quantity FROM items)
       AS discounted
    SET items.retail = items.retail * 0.9
    WHERE discounted.markup >= 1.3
    AND discounted.quantity < 100
    AND items.id = discounted.id;
```
제가 오류났던 SQL 구문은 아래와 같이 수정할 수 있겠습니다.
```sql
UPDATE emp_copy x,
(SELECT hire_date, name FROM emp_copy) as y
SET x.hire_date=y.hire_date
WHERE x.name='jack' AND y.name='홍길동';
```
마지막으로, `SELF JOIN`을 이용한 방법도 GPT를 통해 추천받았습니다.
```sql
UPDATE emp_copy e1
JOIN emp_copy e2 ON e2.name='jack'
SET e1.hire_date = e2.hire_date
WHERE e1.name = '홍길동';
```
## Outro
직관적으로 생각했을 때 값이 대입되기 전에 subquery 가 실행돼서 값으로 사용될 수 있을 것 같은데... 안되네요.

chatGPT는 동일 테이블 지정하지 못하게 한 이유를 아래와 같이 설명했는데, 근거 자료 달라니까 MySQL 공식문서 보고 추정했다고 합니다. 락이 걸리는 문제라면 납득이 되기도 하는데, Oracle DB에서는 된다고 하니 초보자는 대혼란 입니다.
>MySQL에서 테이블을 수정하려면 해당 테이블에 대한 락을 획득해야 합니다. 이 락은 다른 동시 작업을 차단하여 데이터의 일관성을 유지합니다. 그러나 동일한 테이블에 대한 서브쿼리를 사용하려면 해당 테이블에 대한 동시 액세스가 필요합니다. 이렇게 되면 락이 충돌하게 되어 작업이 완료되지 않을 수 있습니다.

추가로 생기는 의문도 있고 하지만, 이 이상 파고들어가는건 필요할 때 하는게 좋겠습니다. 

## 참고자료
1. [https://dev.mysql.com/doc/refman/8.0/en/subquery-errors.html](https://dev.mysql.com/doc/refman/8.0/en/subquery-errors.html)
2. [https://dev.mysql.com/doc/refman/8.0/en/subquery-restrictions.html](https://dev.mysql.com/doc/refman/8.0/en/subquery-restrictions.html)
3. [https://bugs.mysql.com/bug.php?id=6980](https://bugs.mysql.com/bug.php?id=6980)
4. [https://dev.mysql.com/doc/refman/8.0/en/update.html](https://dev.mysql.com/doc/refman/8.0/en/update.html)
5. [https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html#optimizer-hints-table-level](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html#optimizer-hints-table-level)
