---
layout: post
title: MySQL 용어집(Glossary) 살펴보기
categories:
- Database
- MySQL
tags:
- Database
- MySQL
date: 2023-11-05 21:32 +0900
---
## Intro

트랜잭션을 뭘로 공부할까 고민하다가, 주로 사용하는 [MySQL 8.0 버전 용어집(Glossary)](https://dev.mysql.com/doc/refman/8.0/en/glossary.html)을 살펴보기로 했습니다.

Transaction 부터 시작해서 보다보니, 한도 끝도 없어서 그냥 생각날때마다 살펴보면서 업데이트하고 무지성으로 적어놓은 질문에 대한 답도 찾아보고 하겠습니다.

## Transaction

[Transaction](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_transaction)

>Transactions are atomic units of work that can be  **committed**  or  **rolled back**. When a transaction makes multiple changes to the database, either all the changes succeed when the transaction is committed, or all the changes are undone when the transaction is rolled back.
>
>Database transactions, as implemented by  `InnoDB`, have properties that are collectively known by the acronym  **ACID**, for atomicity, consistency, isolation, and durability.

### 번역

> 트랜잭션은 [커밋](#commit) 또는 [롤백](#rollback)될 수 있는 작업의 최소 단위입니다. 트랜잭션이 데이터베이스를 여러번 변경하는 경우, 트랜잭션이 커밋될 때 모든 변경 사항이 성공하거나 트랜잭션이 롤백될 때 모든 변경 사항이 실행 취소됩니다.
>
> `InnoDB`로 구현된 데이터베이스 트랜잭션은 ACID(Atomicity, Consistency, Isolation, Durability)라는 약어로 통칭되는 특성을 갖고 있습니다.

### 새로 알게된 것

- MySQL은 InnoDB로 트랜잭션을 구현하고 있다.

### 질문

- MySQL은 InnoDB로 트랜잭션을 어떻게 구현하고 있을까?
- 커밋(commit)이 되면, MySQL에서는 무슨 일이 일어날까?
- 롤백(rollback)이 되면, MySQL에서는 무슨 일이 일어날까?
- InnoDB에 의해 구현한 데이터베이스 트랜잭션이라면 다른 Storage Engine은 ACID 특성을 구현하지 않은건가?
	- [Alternative Storage Engine](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html) 페이지의 Table 16.1 Storage Engines Feature Summary를 보면 `InnoDB`와 `NDB`만 트랜잭션을 지원하고, NDB는 MySQL로 Cluster 를 구성할 때 사용하는 Storage Engine이다. [MySQL NDB Cluster 제품 페이지](https://www.mysql.com/products/cluster/features.html)에는 ACID를 지원한다고 명시하고 있다. 따라서 Cluster 구성이 아닌 상황에서 트랜잭션을 사용하려면 InnoDB 말고는 선택지가 없다.
- MySQL은 ACID 특성을 어떻게 구현했을까?
	- InnoDB는 doublewirte buffer 를 이용하여 Durability 지원
- InnoDB 는 Storage Engine인데, Storage Engine이 정확히 뭘까?

---

이후 용어는 알파벳 순으로 나열합니다.

## Categories

### A

- [ACID](#acid)
- [Autocommit](#autocommit)

### C

- [Commit](#commit)
- [Crash Recovery](#crash-recovery)

### D

- [Doublewrite Buffer](#doublewrite-buffer)

### L

- [LSN](#lsn)

### R

- [Redo](#redo)
- [Redo Log](#redo-log)
- [Rollback](#rollback)

### S

- [Storage Engine](#storage-engine)

### T

- [Transaction](#transaction)

### U

- [Undo](#undo)
- [Undo Log](#undo-log)

## Glossary

### ACID

[ACID](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_acid)

> An acronym standing for atomicity, consistency, isolation, and durability. These properties are all desirable in a database system, and are all closely tied to the notion of a  **transaction**. The transactional features of  `InnoDB`  adhere to the ACID principles.
>
> Transactions are  **atomic**  units of work that can be  **committed**  or  **rolled back**. When a transaction makes multiple changes to the database, either all the changes succeed when the transaction is committed, or all the changes are undone when the transaction is rolled back.
>
> The database remains in a consistent state at all times — after each commit or rollback, and while transactions are in progress. If related data is being updated across multiple tables, queries see either all old values or all new values, not a mix of old and new values.
> 
> Transactions are protected (isolated) from each other while they are in progress; they cannot interfere with each other or see each other's uncommitted data. This isolation is achieved through the  **locking**  mechanism. Experienced users can adjust the  **isolation level**, trading off less protection in favor of increased performance and  **concurrency**, when they can be sure that the transactions really do not interfere with each other.
> 
> The results of transactions are durable: once a commit operation succeeds, the changes made by that transaction are safe from power failures, system crashes, race conditions, or other potential dangers that many non-database applications are vulnerable to. Durability typically involves writing to disk storage, with a certain amount of redundancy to protect against power failures or software crashes during write operations. (In  `InnoDB`, the  **doublewrite buffer**  assists with durability.)

#### 번역

> 원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 내구성(Durability)을 나타내는 약어입니다. 이러한 속성은 모두 데이터베이스 시스템에서 바람직한 특성이며, 모두 **트랜잭션** 개념과 밀접하게 연관되어 있습니다. `InnoDB`의 트랜잭션 기능은 ACID 원칙을 준수합니다.
>
> 트랜잭션은 **커밋**하거나 **롤백**할 수 있는 작업의 **최소** 단위입니다. 트랜잭션이 데이터베이스를 여러 번 변경하는 경우 트랜잭션이 커밋될 때 모든 변경 사항이 성공하거나 트랜잭션이 롤백될 때 모든 변경 사항이 취소됩니다.
>
> 데이터베이스는 각 커밋 또는 롤백 이후와 트랜잭션이 진행되는 동안 항상 일관된 상태(consistent state)로 유지됩니다. 여러 테이블에서 관련 데이터가 업데이트되는 경우, 쿼리에는 이전 값과 새로운 값이 섞여서 표시되는 것이 아니라 모든 이전 값 또는 모든 새로운 값이 표시됩니다.
>
> 트랜잭션은 진행 중인 동안 서로 보호(격리)되며, 서로 간섭하거나 서로의 커밋되지 않은 데이터를 볼 수 없습니다. 이러한 격리는 **잠금(locking)** 메커니즘을 통해 이루어집니다. 숙련된 사용자는 트랜잭션이 실제로 서로 간섭하지 않는다고 확신할 수 있는 경우, **격리 수준(Isolation Level)**을 조정하여 성능과 **동시성(Concurrency)** 향상을 위해 보호 수준을 낮출 수 있습니다.
>
> 트랜잭션의 결과는 내구성이 있습니다. 커밋 작업이 성공하면 해당 트랜잭션에 의해 변경된 내용은 정전, 시스템 충돌, race conditions 또는 데이터베이스가 아닌 많은 애플리케이션이 취약한 기타 잠재적 위험으로부터 안전합니다. 내구성(Durability)에는 일반적으로 디스크 스토리지에 쓰는 작업이 포함되며, 쓰기 작업 중 정전이나 소프트웨어 충돌을 방지하기 위해 일정량의 중복성(Redundancy)을 갖습니다. (`InnoDB`에서는 **이중 쓰기 버퍼(doublewrite buffer)**가 내구성을 지원합니다.)

#### 새로 알게된 것

- 쓰기 작업을 할 때, 어느정도 중복으로 저장되기도 한다.
- InnoDB는 doublewrite buffer로 durability 를 지원한다.

#### 질문

- locking mechanism 하고 isloation level은 별개의 것으로 봐야겠지?
- doublewrite buffer는 뭐지?
  - [doublewrite buffer](#doublewrite-buffer)
- 트랜잭션 지원하지 않는 Storage Engine은 ACID 특성을 만족 못하나? doublewrite buffer 같은거도 없나? 그럼 트랜잭션 지원하는 NDB는 doublewrite buffer 말고 뭘 쓰지?

### Autocommit

[Autocommit](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_autocommit)

> A setting that causes a **commit** operation after each **SQL** statement. This mode is not recommended for working with `InnoDB` tables with **transactions** that span several statements. It can help performance for **read-only transactions** on `InnoDB` tables, where it minimizes overhead from **locking** and generation of **undo** data, especially in MySQL 5.6.4 and up. It is also appropriate for working with [`MyISAM`](https://dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html "16.2 The MyISAM Storage Engine") tables, where transactions are not applicable.

#### 번역

> 각 SQL 문 뒤에 커밋 작업을 수행하도록 하는 설정입니다. 이 모드는 여러 statement에 걸쳐있는 트랜잭션이 있는 InnoDB 테이블 작업에는 권장되지 않습니다. 이 모드는 특히 MySQL 5.6.4 이상에서 locking 및 undo data 생성으로 인한 오버헤드를 최소화하여 InnoDB 테이블에서 읽기 전용 트랜잭션의 성능을 개선할 수 있습니다. 트랜잭션이 적용되지 않는 MyISAM 테이블 작업에도 적합합니다.

#### 질문

- [Commit](#commit)에서 InnoDB는 커밋에 대해 낙관적 메커니즘을 사용한다고 했으니까, 기본적으로 undo 복구를 하는 수행하는 건가? Redo는 안하나?
	- [Redo](#redo) 참고
	- Undo 는 롤백 할때 사용하는 반면, Redo는 Crash Recovery 할때 사용한다.
	- Undo 와 Redo를 양자 택일의 관계로 잘못 생각한 질문
	  - 복구 기법 별로 Undo, Redo 모두 사용되는 것도 있고, 안되는 것도 있는걸 잊고 있었다.
- undo data가 undo log 말고 다른 것도 있나?
	- [Undo](#undo) 용어 설명을 보면, undo log를 undo tablespaces에 저장하는 것 외에는 아직 안보인다.(2023-11-03)
- MySQL 5.6.4 전에는 autocommit이 없었나?
	- [5.6 release note](https://downloads.mysql.com/docs/mysql-5.6-relnotes-en.pdf) 에서 5.6.2에도 autocommit이 Bug Fix에 등장(275/318 page)하는거 보면, 그렇지는 않은거 같다.

### Commit

[Commit](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_commit)

>A  **SQL**  statement that ends a  **transaction**, making permanent any changes made by the transaction. It is the opposite of  **rollback**, which undoes any changes made in the transaction.
>
>`InnoDB`  uses an  **optimistic**  mechanism for commits, so that changes can be written to the data files before the commit actually occurs. This technique makes the commit itself faster, with the tradeoff that more work is required in case of a rollback.
>
>By default, MySQL uses the  **autocommit**  setting, which automatically issues a commit following each SQL statement.

#### 번역

> 트랜잭션을 종료하여 트랜잭션에 의해 변경된 모든 내용을 영구적으로 적용하는 SQL 문입니다. 트랜잭션의 변경 사항을 취소하는 롤백과는 반대입니다.
>
> InnoDB는 커밋에 대해 낙관적(optimistic) 메커니즘을 사용하므로 커밋이 실제로 발생하기 전에 데이터 파일에 변경 사항을 기록할 수 있습니다. 이 기법을 사용하면 커밋 자체는 더 빨라지지만 롤백이 발생할 경우 더 많은 작업이 필요하다는 단점이 있습니다.
> 
> 기본적으로 MySQL은 autocommit 설정을 사용하여, 각 SQL 문 다음에 자동으로 커밋을 실행합니다.

#### 새로 알게된 것

- commit 도 SQL 문이다.
	- 생각해보니 옛날에 commit은 DCL로 배웠다. 그중에 commit, rollback, savepoint(checkpoint) 는 TCL(Transaction Control Language)이다.
- 커밋을 수행하는 특정한 메커니즘 명칭이 있다. 그리고 InnoDB는 낙관적 메커니즘을 사용한다.

#### 질문

- 커밋에 대한 낙관적 메커니즘은 구체적으로 어떻게 동작할까?
- 다른 커밋 메커니즘은 뭐가 있을까?
- 커밋의 낙관적 메커니즘이 JPA 사용할  Optimistic Locking 이랑은 어떤 관련이 있을까? JPA는 어플리케이션 레벨에서 구현한건가?
- 커밋이 실제로 발생하기 전에 데이터 파일에 변경 사항을 `기록할 수(can be written) 있다`는 거는 기록 안할수도 있다는 건가?
- 낙관적 메커니즘은 커밋이 왜 더 빨라질까? 상대적인 것을 의미하는건가?
	- 롤백은 이미 기록된걸 취소해야 되니까 더 많은 작업이 필요한거는 이해된다.
- 구체적으로 SQL을 설명하려면 어떻게 설명해야할까?

### Crash Recovery

[Crash Recovery](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_crash_recovery)

> The cleanup activities that occur when MySQL is started again after a  **crash**. For  **InnoDB**  tables, changes from incomplete transactions are replayed using data from the  **redo log**. Changes that were  **committed**  before the crash, but not yet written into the  **data files**, are reconstructed from the  **doublewrite buffer**. When the database is shut down normally, this type of activity is performed during shutdown by the  **purge**  operation.
>
> During normal operation, committed data can be stored in the  **change buffer**  for a period of time before being written to the data files. There is always a tradeoff between keeping the data files up-to-date, which introduces performance overhead during normal operation, and buffering the data, which can make shutdown and crash recovery take longer.

#### 번역

> **충돌(crash)** 후 MySQL이 다시 시작될 때 발생하는 정리(cleanup) 작업입니다. **InnoDB** 테이블의 경우, 불완전한 트랜잭션의 변경 사항은 **redo log**의 데이터를 사용하여 재생(replay)됩니다. 충돌 전에 **커밋**되었지만 아직 **data files**에 기록되지 않은 변경 사항은 **doublewrite buffer**에서 재구성(reconstruct)됩니다. 데이터베이스가 정상적으로 shutdown되면 이러한 유형의 작업은 shutdown 중에 **퍼지(purge)** 작업에 의해 수행됩니다.
>
> 정상 작동 중에 커밋된 데이터는 data files에 기록되기 전에 일정 기간 동안 **change buffer**에 저장될 수 있습니다. 정상 작동 중에 성능 오버헤드를 유발하는 data files를 최신 상태로 유지하는 것과 shutdown 및 crash recovery 시간이 길어질 수 있는 데이터 버퍼링 사이에는 항상 트레이드오프 관계가 있습니다.

#### 새로 알게된 것

- 데이터베이스를 정상적으로 종료할 때 purge 작업이라는 것을 수행한다.
- data files에 기록되기 전에 doublewrite buffer 뿐만 아니랄 change buffer에 저장될 수도 있다.

#### 질문

- purge 작업은 또 무엇인가?
- redo log는 이미 data files에 기록까지 완료된 데이터를 다루는 건가? doublewrite buffer는 data files에 아직 기록되지 않은 데이터를 다루는거고?
- change buffer랑 doublewrite buffer는 전혀 별개의 것인가? 그럼 총 세 번의 중복 쓰기가 될 수 있는건가?
- change buffer에 저장될 수 있으(can be stored)니까, 저장이 안되는 경우도 있는건가?
- 트레이드오프 관계가 있다면, 설정을 바꿔서 선택할 수 있는건가?
- data files에 바로 쓰기 작업하면 성능 상 더 안좋아서 성능 오버헤드를 유발한다는건가? 파일을 열었다 닫았다 자주해서 그러나... doublewrite buffer 도 디스크에 저장한다고 했는데, large sequential chunk 인지 뭔지 때문에 doublewrite buffer는 성능 오버헤드가 더 적은건가?

### Doublewrite Buffer

[Doublewrite Buffer](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_doublewrite_buffer)

> `InnoDB`  uses a file flush technique called doublewrite. Before writing  **pages**  to the  **data files**,  `InnoDB`  first writes them to a storage area called the doublewrite buffer. Only after the write and the flush to the doublewrite buffer have completed, does  `InnoDB`  write the pages to their proper positions in the data file. If there is an operating system, storage subsystem or  [**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html "4.3.1 mysqld — The MySQL Server")  process crash in the middle of a page write,  `InnoDB`  can find a good copy of the page from the doublewrite buffer during  **crash recovery**.
>
> Although data is always written twice, the doublewrite buffer does not require twice as much I/O overhead or twice as many I/O operations. Data is written to the buffer itself as a large sequential chunk, with a single  `fsync()`  call to the operating system.

### 번역

> `InnoDB`는 doublewrite라는 file flush 기술을 사용합니다. `InnoDB`는 **data files**에 **pages**를 쓰기 전에 먼저 doublewrite buffer라는 storage area에 pages를 write합니다. write와 doublewrite buffer에 대한 flush가 완료된 후에야 `InnoDB`는 pages를 data files의 적절한 위치에 write 합니다. page write 도중에 OS, storage subsystem 또는 mysqld 프로세스가 충돌하는 경우, `InnoDB`는 crash recovery 중에 doublewrite buffer에서 page의 올바른 복사본을 찾을 수 있습니다.
>
> 데이터는 항상 두 번 쓰여지지만, doublewrite buffer는 두 배의 I/O 오버헤드나 두 배의 I/O 작업을 필요로 하지 않습니다. 데이터는 OS에 대한` fsync()` 호출 한 번으로 버퍼 자체에 큰 순차 청크(large sequential chunk)로 기록됩니다.

#### 새로 알게된 것

- doublewrite은 file flush 기술이다.
- pages 라는 단위가 있다.
- [ACID](#acid) 설명만 봤을 때는 데이터가 항상 두 번 쓰여지는 것은 아닌줄 알았는데, 항상 두 번 쓰여진다.
- 두 번 쓰기는 하는데, I/O 작업을 두 배로하지는 않는다.
- large sequential chunk, fsync() 새로운 용어들

#### 질문

- flush가 buffer에 있는 내용을 전부 내보내는거 맞나?
- crash recovery 중에 사용되면 redo log하고는 무슨 관계가 있는걸까?
- page는 뭘 의미하는거지?
- data files 는 정확히 무엇을 가리키는거지?

### LSN

[LSN](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_lsn)

Acronym for  “log sequence number”. This arbitrary, ever-increasing value represents a point in time corresponding to operations recorded in the  **redo log**. (This point in time is regardless of  **transaction**  boundaries; it can fall in the middle of one or more transactions.) It is used internally by  `InnoDB`  during  **crash recovery**  and for managing the  **buffer pool**.

Prior to MySQL 5.6.3, the LSN was a 4-byte unsigned integer. The LSN became an 8-byte unsigned integer in MySQL 5.6.3 when the redo log file size limit increased from 4GB to 512GB, as additional bytes were required to store extra size information. Applications built on MySQL 5.6.3 or later that use LSN values should use 64-bit rather than 32-bit variables to store and compare LSN values.

In the  **MySQL Enterprise Backup**  product, you can specify an LSN to represent the point in time from which to take an  **incremental backup**. The relevant LSN is displayed by the output of the  **mysqlbackup**  command. Once you have the LSN corresponding to the time of a full backup, you can specify that value to take a subsequent incremental backup, whose output contains another LSN for the next incremental backup.

#### 번역

> "log sequence number”"의 약어입니다. 이 임의의 계속 증가하는 값은 **redo log**에 기록된 작업에 해당하는 시점을 나타냅니다. (이 시점은 **트랜잭션** 경계와 무관하며, 하나 이상의 트랜잭션 중간에 속할 수 있습니다.) 이 값은 `InnoDB`에서 **crash recovery** 및 **buffer pool** 관리를 위해 내부적으로 사용됩니다.
>
> MySQL 5.6.3 이전에는 LSN이 4-byte unsigned integer였습니다. redo log 파일 크기 제한이 4GB에서 512GB로 증가하면서 extra size 정보를 저장하는 데 추가적인 bytes가 필요했기 때문에 MySQL 5.6.3에서는 LSN이 8-byte unsigned integer가 되었습니다. LSN 값을 사용하는 MySQL 5.6.3 이상에서 구축된 애플리케이션은 32비트 변수가 아닌 64비트 변수를 사용하여 LSN 값을 저장 및 비교해야 합니다.
>
> **MySQL Enterprise Backup** 제품에서는 **증분 백업(incremental backup)**을 수행할 시점을 나타내기 위해 LSN을 지정할 수 있습니다. 관련 LSN은 **mysqlbackup** 명령의 출력에 표시됩니다. 전체 백업(full backup) 시점에 해당하는 LSN이 있으면 해당 값을 지정하여 후속 증분 백업을 수행할 수 있으며, 출력에는 다음 증분 백업에 대한 또 다른 LSN이 포함됩니다.

#### 새로 알게된 것

- LSN은 Log Sequence Number의 약자다.
- MySQL에는 Buffer Pool 이라는게 존재한다.
- LSN은 특정 트랜잭션에 종속적이지 않다.
- redo log 파일 크기에는 제한이 있고, 512GB까지 저장할 수 있다.
- LSN은 백업할 때도 사용하며, incremental backup 은 MySQL Enterprise Backup 제품에서 사용 가능하다.

#### 질문

- redo log는 그동안 수행한 쿼리를 전부 기록하는 건가? LSN 으로 카운팅해서 특정 갯수만큼 쿼리를 다시 실행하는게 재생(replay) 인가?
- buffer pool은 뭘까? 지금까지 본 doublewrite buffer, change buffer 말고도 buffer가 많아서, buffer 들 관리하는건가?
- redo log를 512GB까지 저장할 수 있으면, 더 커졌을 때 full backup 은 어떻게 하나? 파일 크기 제한이니까 파일 하나 더 만드나?

### Redo

[Redo](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_redo)

> The data, in units of records, recorded in the **redo log** when [DML](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_dml "DML") statements make changes to `InnoDB` tables. It is used during **crash recovery** to correct data written by incomplete **transactions**. The ever-increasing **LSN** value represents the cumulative amount of redo data that has passed through the redo log.

#### 번역

> DML 문이 InnoDB 테이블을 변경할 때 redo log에 기록되는 레코드 단위의 데이터입니다. 불완전한 트랜잭션에 의해 기록된 데이터를 수정하기 위해 crash recovery 중에 사용됩니다. 계속 증가하는 LSN 값은 redo log를 통과한 redo data의 누적 양을 나타냅니다.

#### 새로 알게된 것

- 시험 문제용으로 배웠을 때 undo와 redo는 양자 택일로 이해했는데, 데이터베이스에서 undo 와 redo를 둘 중 하나만 지원하는게 아니라 상황에 따라 적합한걸 사용한다.
- redo log는 crash recovery 중에 사용된다.
- redo log는 DML 문을 사용한다는 조건과 InnoDB가 테이블을 변경한다는 조건이 있다.
- redo log를 통과한 redo data의 누적 양을 나타내는 LSN이라는 값이 있다.

#### 질문

- 레코드 단위는 테이블에 있는 사용자 데이터를 말하는건가?
- LSN은 또 무엇인가?
- redo log를 통과했다는 의미가 뭐지?
- redo log를 통과한게 무슨 의미가 있길래 LSN에 기록을 하는거지?
- redo log에는 무슨 내용이 저장되는거지?
- DML을 INSERT, UPDATE, DELETE 말고 또 뭐가 있지?
- 롤백을 시작할 때 undo log가 기록되기 시작하고, 일반적으로 트랜잭션이 시작되면 redo log가 기록되는 건가?
- WAL 는 언제 등장하는건가?
- crash recovery 동작은 어떻게 수행할까?

### Redo Log

[Redo Log](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_redo_log)

> A disk-based data structure used during  **crash recovery**, to correct data written by incomplete  **transactions**. During normal operation, it encodes requests to change  `InnoDB`  table data, which result from SQL statements or low-level API calls. Modifications that did not finish updating the  **data files**  before an unexpected  **shutdown**  are replayed automatically.

> The redo log is physically represented on disk as a set of redo log files. Redo log data is encoded in terms of records affected; this data is collectively referred to as  **redo**. The passage of data through the redo log is represented by an ever-increasing  **LSN**  value.

> For more information, see  [Section 15.6.5, “Redo Log”](https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html "15.6.5 Redo Log")

#### 번역

> 불완전한 **트랜잭션**에 의해 쓰여진 데이터를 수정하기 위해 **crash recovery** 중에 사용되는 디스크 기반 데이터 구조입니다. 정상 작동 중에는 SQL 문 또는 low-level API 호출로 인해 발생하는 InnoDB 테이블 데이터 변경 요청을 인코딩합니다. 예기치 않은 **shutdown** 전에 **데이터 파일** 업데이트를 완료하지 못한 수정 사항은 자동으로 재생(replayed)됩니다.
>
> redo log는 물리적으로 디스크에 redo log 파일 집합으로 표시됩니다. redo log 데이터는 영향을 받는 레코드의 관점에서 인코딩되며, 이 데이터를 통칭하여 **redo**라고 합니다. redo log를 통한 데이터의 통행은 계속 증가하는 **LSN** 값으로 표시됩니다.

#### 새로 알게된 것

- redo log는 디스크에 기록이 된다.
- redo log는 InnoDB 테이블 데이터 변경 요청을 인코딩한다.

#### 질문

- 언제 어떻게 redo log를 작성하고, 어떻게 crash recovery 할때 쓰인다는거지?
- InnoDB 테이블 데이터 변경 시에만 redo log가 작성되나?
- InnoDB 테이블 데이터 변경 요청을 인코딩한다는게 무슨 의미지?
- 자동으로 재생(replayed)된다는게 무슨 의미지?
- 레코드의 관점에서 인코딩 된다는게 무슨 의미지?
- redo log에는 무슨 내용이 저장되는건가?
- 기존 지식과 함께 정리해보기
  - 시험용으로 외웠던 지식에 대입해보면, 불완전한 트랜잭션은 commit은 했지만 data files에 저장되지 않은 트랜잭션이기 때문에 redo log에 기록된 내용들을 재생해서 data files에 기록하고, commit조차 되지 않은 트랜잭션 또는 rollback 쿼리를 수행하면 undo log를 이용해서 낙관적인 메커니즘으로 어딘가에 기록된 내용들을 다 취소하는 것...? 맞는건지 확인이 필요하겠다.

### Rollback

[Rollback](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_rollback)

> A  **SQL**  statement that ends a  **transaction**, undoing any changes made by the transaction. It is the opposite of  **commit**, which makes permanent any changes made in the transaction.
>
> By default, MySQL uses the  **autocommit**  setting, which automatically issues a commit following each SQL statement. You must change this setting before you can use the rollback technique.

#### 번역

> 트랜잭션을 종료하여 트랜잭션에 의해 변경된 내용을 모두 취소하는 SQL 문입니다. 트랜잭션의 모든 변경 내용을 영구적으로 적용하는 커밋과는 반대입니다.
>
> 기본적으로 MySQL은 각 SQL 문 다음에 자동으로 커밋을 실행하는 autocommit 설정을 사용합니다. 롤백 기법을 사용하려면 먼저 이 설정을 변경해야 합니다.

#### 새로 알게된 것

- 롤백은 autocommit 모드에서는 안된다.
	- 당연하긴 한데, 롤백을 써본적이 없어서 생각해볼 일도 없었다.

#### 질문

- MySQL은 기본 설정이 autocommit인데, JPA에서 롤백은 어떻게 되는걸까? Application Level에서 되는건가?

### Storage Engine

[Storage Engine](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_storage_engine)

> A component of the MySQL database that performs the low-level work of storing, updating, and querying data. In MySQL 5.5 and higher,  **InnoDB**  is the default storage engine for new tables, superceding  `MyISAM`. Different storage engines are designed with different tradeoffs between factors such as memory usage versus disk usage, read speed versus write speed, and speed versus robustness. Each storage engine manages specific tables, so we refer to  [`InnoDB`](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html "Chapter 15 The InnoDB Storage Engine")  tables,  [`MyISAM`](https://dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html "16.2 The MyISAM Storage Engine")  tables, and so on.
>
> The  **MySQL Enterprise Backup**  product is optimized for backing up  `InnoDB`  tables. It can also back up tables handled by  `MyISAM`  and other storage engines.

#### 번역

> 데이터를 저장, 업데이트 및 질의하는 low-level 작업을 수행하는 MySQL 데이터베이스의 컴포넌트입니다. MySQL 5.5 이상에서는 **InnoDB**가 새 테이블의 기본 스토리지 엔진으로 `MyISAM`을 대체합니다. 각 스토리지 엔진은 메모리 사용량 대 디스크 사용량, 읽기 속도 대 쓰기 속도, 속도 대 견고성 등의 요소 간에 서로 다른 트레이드오프를 고려하여 설계되었습니다. 각 스토리지 엔진은 특정 테이블을 관리하므로 `InnoDB` 테이블, `MyISAM` 테이블 등을 참조합니다.

**MySQL Enterprise Backup** 제품은 InnoDB 테이블 백업에 최적화되어 있습니다. 또한 MyISAM 및 기타 스토리지 엔진에서 처리하는 테이블도 백업할 수 있습니다.

#### 새로 알게된 것

- MySQL Enterprise Backup은 InnoDB 테이블 백업에 최적화되어 있지만, 다른 Storage Engine도 백업이 가능하다.

#### 질문

- Storage Engine의 처리 결과물이 테이블 파일이라고 보면 되는건가?

### Undo

[Undo](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_undo)

> Data that is maintained throughout the life of a **transaction**, recording all changes so that they can be undone in case of a **rollback** operation. It is stored in **undo logs** either within the **system tablespace** (in MySQL 5.7 or earlier) or in separate **undo tablespaces**. As of MySQL 8.0, undo logs reside in undo tablespaces by default.

#### 번역

> 트랜잭션의 수명 동안 유지되는 데이터로, 롤백 작업 시 실행 취소할 수 있도록 모든 변경 사항을 기록합니다. undo logs는 system tablespace(MySQL 5.7 이하) 또는 별도의 undo tablespaces에 저장됩니다. MySQL 8.0부터 undo logs는 기본적으로 undo tablespaces에 저장됩니다.

#### 새로 알게된 것

- MySQL에서는 undo data가 롤백 작업에 사용된다.
- undo logs가 저장되는 undo tablespaces 라는게 별도로 존재한다.

#### 질문

- undo tablespaces 는 어떻게 확인하지?
	- [InnoDB Undo Tablesapces 문서](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-tablespaces.html)
- redo 는 언제하나?
	- [Redo](#redo)
- undo log를 롤백 작업에 사용하는 다른 데이터베이스는 뭐가 있나?
	- Oracle: _Redo_ is the information Oracle records in online (and archived) redo log files in order to “replay” your transaction in the event of a failure. _Undo_ is the information Oracle records in the undo segments in order to reverse, or roll back, your transaction.(출처: [Oreilly](https://www.oreilly.com/library/view/oracle-database-transactions/9781484207604/9781484207611_Ch06.xhtml#:~:text=Redo%20is%20the%20information%20Oracle,or%20roll%20back,%20your%20transaction.))
- undo tablespaces에 저장되는 undo log에는 어떤 값들이 기록될까?
- 왜 undo 에는 redo 처럼 LSN이 없을까?

### Undo Log

[Undo Log](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_undo_log)

> A storage area that holds copies of data modified by active  **transactions**. If another transaction needs to see the original data (as part of a  **consistent read**  operation), the unmodified data is retrieved from this storage area.
>
> In MySQL 5.6 and MySQL 5.7, you can use the  [`innodb_undo_tablespaces`](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_undo_tablespaces)  variable have undo logs reside in  **undo tablespaces**, which can be placed on another storage device such as an  **SSD**. In MySQL 8.0, undo logs reside in two default undo tablespaces that are created when MySQL is initialized, and additional undo tablespaces can be created using  [`CREATE UNDO TABLESPACE`](https://dev.mysql.com/doc/refman/8.0/en/create-tablespace.html "13.1.21 CREATE TABLESPACE Statement")  syntax.
>
> The undo log is split into separate portions, the  **insert undo buffer**  and the  **update undo buffer**.

#### 번역

> 활성 **트랜잭션**에 의해 수정된 데이터의 사본을 보관하는 저장 영역입니다. 다른 트랜잭션에서 원본 데이터를 확인해야 하는 경우(**consistent read** 작업의 일부로) 수정되지 않은 데이터가 이 저장 영역에서 검색됩니다.
>
> MySQL 5.6 및 MySQL 5.7에서는 `innodb_undo_tablespaces` 변수를 사용하여 undo logs가 **SSD**와 같은 다른 저장 장치에 배치할 수 있는 **undo tablespaces**에 상주하도록 할 수 있습니다. MySQL 8.0에서 undo logs는 MySQL이 초기화될 때 생성되는 두 개의 기본 undo tablespaces에 저장되며, `CREATE UNDO TABLESPACE` 구문을 사용하여 추가적인 undo tablespaces를 생성할 수 있습니다.
>
> undo log는 `insert undo buffer`와 `update undo buffer`로 분리되어 있습니다.

#### 새로 알게된 것

- [Undo](#undo)만 봤을 때는, 롤백할 때`만` Undo log가 쓰이는 걸로 이해했는데, 다른 트랜잭션이 읽기 작업 수행할 때도 쓰인다.
- MySQL 8.0 부터는 Undo Log를 저장하는 테이블이 기본적으로 2개다. 그리고 CREATE UNDO TABLESPACE 구문으로 추가로 만들 수 있다.
- undo log는 insert 할 때와 update 할 때 저장되며, 별도로 저장된다.

#### 질문

- MySQL 5.7 까지는 undo log를 메모리에 저장한건가?
- 기본 undo log 테이블이 2개인 이유가 insert undo buffer랑 update undo buffer 를 별도 테이블에 저장해서 그런건가?
- delete 할 때는 undo log가 필요 없나?
