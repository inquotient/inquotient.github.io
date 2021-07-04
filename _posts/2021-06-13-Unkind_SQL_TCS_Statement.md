---
title:  TCS 문
categories:
- Unkind_SQL
feature_text: |
  ## 19. TCS 문
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

TCS는 Transaction Control Statement의 약자다. 트랜잭션 제어문으로 해석할 수 있다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (cd NUMBER, v1 NUMBER) ROWDEPENDENCIES;

INSERT INTO t1 VALUES (1, 50);
INSERT INTO t1 VALUES (2, 50);
COMMIT;
```

+ ROWDEPENDENCIES 절  
테이블 생성 시 ROWDEPENDENCIES 절을 기술하면 row-level dependency tracking 기능이 활성화된다. 해당 기능을 활성화하면 SCN이 행 수준으로 저장되며 행의 길이가 6바이트씩 증가한다. 기본값은 NOROWDEPENDENCIES로 SCN이 블록 수준으로 저장된다.  

### 19.1 트랜잭션
<br/>
트랜잭션(transaction)은 함께 수행해야 하는 작업의 논리적인 단위다. 계좌이체 트랜잭션은 출금 계좌의 잔고를 차감하는 작업과 입금 계좌의 잔고를 증가시키는 작업으로 구성된다. 두 작업은 반드시 하나의 트랜잭션으로 수행되어야 한다.  

아래 쿼리는 1번 계좌에서 2번 계좌로 10원을 이체하는 트랜잭션을 수행한다. 1번 계좌의 잔고에서 10원을 빼는 쿼리와 2번 계좌의 잔고에 10원을 더하는 쿼리가 하나오ㅢ 트랜잭션으로 수행되었다. 두 쿼리가 하나의 트랜잭션으로 수행되지 않으면 잔고의 합계가 달라질 수 있다.  

#### 19.1.1. 구조
<br/>
트랜잭션은 DML 문이나 SET TRANSACTION 문이 실행되면 시작되고, COMMIT 문이나 ROLLBACK 문이 실행되면 종료된다. 트랜잭션이 시작되면 내부적으로 언두 세그먼트(undo segment)가 할당되고, 트랜잭션에 트랜잭션 ID(XID)가 부여된다. 트랜잭션 ID는 언두 세그먼트의 번호, 슬롯, 시퀀스의 조합으로 생성된다.  

V$TRANSACTION 뷰에서 트랜잭션에 대한 정보를 조회할 수 있다. 아래 쿼리는 결과가 반환되지 않는다. 수행 중인 트랜잭션이 존재하지 않기 때문이다.  

```sql
SELECT * FROM v$transaction;
```

V$TRANSACTION 뷰를 조회해보면 트랜잭션이 시작된 것을 확인할 수 있다. start&#95;scn 열은 트랜잭션이 시작된 SCN을 반환한다.  

```sql
SELECT xid, xidusn, xidslot, xidsqn, start_date, start_scn FROM v$transaction;
```

COMMIT 문을 수행하면 트랜잭션이 종료된다.  

```sql
COMMIT;
```
V$TRANSACTION 뷰를 다시 조회하면 결과가 반환되지 않는다.  

```sql
SELECT * FROM v$transaction;
```

#### 19.1.2. SCN
<br/>
SCN(System Change Number)은 오라클 데이터베이스의 논리적 TIMESTAMP다. 데이터베이스 내부의 작업 순서를 식별하는 용도로 사용된다. 트랜잭션도 내부적으로 SCN을 사용한다.  

ORA&#95;ROWSCN 슈도 칼럼은 행의 SCN을 반환한다. cd가 3인 행의 ORA&#95;ROWSCN 값이 앞서 살펴본 V$TRANSACTION 뷰의 START&#95;SCN 값과 동일한 것을 확인할 수 있다. SCN&#95;TO&#95;TIMESTAMP 함수를 사용하면 SCN 값을 TIMESTAMP 값으로 변환할 수 있다.  

```sql
SELECT cd, v1, ORA_ROWSCN, SCN_TO_TIMESTAMP(ORA_ROWSCN) AS c1 FROM t1;
```

현재 SCN은 V$DATABASE 뷰의 checkpoint&#95;change# 열에서 확인할 수 있다.  

```sql
SELECT checkpoint_change# FROM v$database;
```

#### 19.1.3. ORA-08181 에러
<br/>
SCN&#95;TO&#95;TIMESTAMP 함수는 SYS.SMON&#95;SCN&#95;TIME 테이블을 참조한다. 해당 테이블은 데이터베이스가 시작된 시간으로부터 최대 120시간(5일) 동안의 SCN을 저장한다. 5일 이전의 SCN에 SCN&#95;TO&#95;TIMESTAMP 함수를 사용하면 "ORA-08181: 지정된 번호는 적합한 시스템 변경 번호가 아님" 에러가 발생할 수 있다.  

아래는 SYS.SMON&#95;SCN&#95;TIME 테이블을 조회한 결과다.  

```sql
SELECT time_do, scn_bas FROM sys.smon_scn_time ORDER BY scn_bas DESC;
```

### 19.2. 기본 문법
<br/>
#### 19.2.1. COMMIT 문
<br/>
COMMIT 문은 현재 트랜잭션의 변경 내용을 데이터베이스에 영구적으로 저장하고 트랜잭션을 종료한다.  

```
COMMIT [WORK] [[COMMENT string] | [WRITE [WAIT | NOWAIT] [IMMEDIATE | BATCH]] | FORCE string [, integer]];
```

아래 쿼리는 DELETE 문을 수행한 후 COMMIT 문을 수행했다. cd가 3인 행이 영구적으로 삭제되었다.  

```sql
DELETE FROM t1 WHERE cd = 3;
```

아래는 잘못된 계좌이체 트랜잭션의 예시다. 1번 계좌의 잔고에서 10을 빼고 COMMIT 문을 수행했기 때문에 하나의 트랜잭션으로 처리되지 않았다. 트랜잭션의 원자성이 보장되지 않기 때문에 데이터 일관성이 위배될 수 있다.  

```sql
UPDATE t1 SET v1 = v1 - 10 WHERE cd = 1;
COMMIT;
UPDATE t1 SET v1 = v1 + 10 WHERE cd = 2;
```

아래 예제는 2개의 UPDATE 문을 하나의 트랜잭션으로 처리했다. COMMIT 문으로 트랜잭션을 종료했기 대문에 트랜잭션에 의한 변경 내용이 영구적으로 저장된다.  

```sql
UPDATE t1 SET v1 = v1 - 10 WHERE cd = 1;
UPDATE t1 SET v1 = v1 + 10 WHERE cd = 2;
COMMIT;
```

#### 19.2.2. ROLLBACK 문
<br/>
ROLLBACK 문은 현재 트랜잭션의 변경 내용을 모두 최소하고 트랜잭션을 종료한다.  

```
ROLLBACK [WORK] [TO [SAVEPOINT] savepoint | FORCE string];
```

아래와 같이 t1 테이블의 전체 행을 삭제해보자.

```sql
DELETE FROM t1;
```

t1 테이블을 조회해보면 행이 존재하지 않는 것을 확인할 수 있다.  

```sql
SELECT * FROM t1;
```

아래와 같이 롤백을 수행해보자.  

```sql
ROLLBACK;
```

t1 테이블을 다시 조회해보면 변경 사항이 모두 롤백된 것을 확인할 수 있다. ROLLBACK 문은 언두 세그먼트에 저장된 변경 이전 데이터를 통해 데이터를 복구한다.  

```sql
SELECT * FROM t1;
```

#### 19.2.3. SAVEPOINT 문
<br/>
SAVEPOINT 문은 롤백할 수 있는 저장점을 생성한다.  

```
SAVEPOINT savepoint;
```

아래 쿼리는 SAVEPOINT 문으로 s1, s2 저장점을 생성한 후 DELETE 문으로 전체 행을 삭제했다.  

```sql
UPDATE t1 SET v1 = v1 - 10 WHERE cd = 2;
SAVEPOINT s1;
UPDATE t1 SET v1 = v1 + 10 WHERE cd = 1;
SAVEPOINT s2;
DELETE FROM t1;
SELECT * FROM t1;
```

아래 쿼리는 s2 저장점으로 트랜잭션을 롤백한다. t1 테이블을 조회해보면 두 번째 UPDATE 문이 적용된 시점으로 데이터가 롤백된 것을 확인할 수 있다.  

```sql
ROLLBACK TO SAVEPOINT s2;
SELECT * FROM t1;
```

아래 쿼리는 s1 저장점으로 트랜잭션을 롤백한다. t1 테이블을 조회해보면 첫 번째 UPDATE 문이 적욛된 시점으로 데이터가 롤백된 것을 확인할 수 있다.  

```sql
ROLLBACK TO SAVEPOINT s1;
SELECT * FROM t1;
```

아래 쿼리는 에러가 발생한다. 특정 저장점(s1)으로 롤백하면 해당 저장점 이후에 생성된 모든 저장점(s2)이 제거되기 때문이다.  

```sql
ROLLBACK TO SAVEPOINT s2;
```

아래 예제의 두 번째 UPDATE 문은 NUMBER 타입인 v1 열에 문자 값을 입력했기 때문에 오류가 발생했다.  

```sql
UPDATE t1 SET v1 = v1 + 10 WHERE cd = 1;

UPDATE t1 SET v1 = 'A' WHERE cd = 1;
```

t1 테이블을 조회해보면 첫 번째 UPDATE 문의 변경 내용이 롤백되지 않은 것을 확인할 수 있다. 오라클 데이터베이스는 DML 문이 수행될 때마다 내부적으로 저장점을 생성하고, 에러가 발생하면 직전의 저장점을 문장 단위 롤백(statement-level rollback)을 수행한다.  

```sql
SELECT * FROM t1;
```

#### 19.2.4. 암시적 커밋
<br/>
DDL 문을 수행 전에 트랜잭션을 커밋하고 종료 후 다시 트랜잭션을 커밋하는 것을 말한다.  

아래는 DDL 문의 동작에 대한 슈도 코드다.  

```sql
BEGIN
  COMMIT;
  DDL 문;
  COMMIT;
EXCEPTION
  WHEN OTHERS THEN ROLLBACK; RAISE;
```
예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t2 PURGE;
CREATE TABLE t2 (c1 NUMBER);
```

아래와 같이 INSERT 문을 수행하고, DDL 문인 CREATE TABLE 문을 수행한 후, 롤백을 수행하자.  

```sql
INSERT INTO t2 VALUES (1);

CREATE TABLE t2 (c1 NUMBER);

ROLLBACK;
```

t2 테이블을 조회해보면 데이터가 롤백되지 않은 것을 확인할 수 있다. CREATE TABLE 문에 의해 트랜잭션이 커밋되기 때문에 INSERT 문에 의한 변경이 롤백되지 않은 것이다.  

```sql
SELECT * FROM t2;
```

### 19.3. 데이터 동시성
<br/>
데이터 동시성(data concurrency)은 다수의 사용자가 동일한 데이터에 동시에 접근할 수 있는 것을 말한다.  

#### 19.3.1. 락킹 메커니즘
<br/>
오라클 데이터베이스는 데이터 동시성을 보장하기 위해 락(lock)을 사용한다. 락은 자원의 사용을 직렬화하기 위한 방법 중 하나다. 락 외에도 래치(latch)와 뮤텍스(mutex) 등을 사용하여 자원의 사용을 직렬화한다.  

오라클 데이터베이스는 다양한 락을 사용한다. V$LOCK&#95;TYPE 뷰에서 락의 종류를 조회할 수 있다. DML 문은 TM 락과 TX 락을 사용한다. TM 락(table lock)은 테이블, TX 락(row lock)은 트랜잭션에 설정되는 락이다. TX 락은 로우 레벨 락(low level)과 연결된다. 로우 레벨 락은 데이터 블록의 로우에 설정된다. 하나의 TX 락이 다수의 로우 레벨 락과 연결되는 구조다.

```sql
SELECT type, name, description FROM v$lock_type ORDER BY type;
```

락은 종류에 따라 아래의 모드로 설정될 수 있다. Y로 표시된 항목은 자원을 동시에 사용할 수 있는 조합이다. DML 문은 TM 락을 RX 모드, TX 락을 X 모드로 설정한다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>RS</td>
      <td>RX</td>
      <td>S</td>
      <td>SRX</td>
      <td>X</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>RS (Row Share)</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
    </tr>
    <tr>
      <td>RX (Row eXclusive)</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>S (Share)</td>
      <td>Y</td>
      <td></td>
      <td>Y</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>SRX (Share Row eXclusive)</td>
      <td>Y</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>X (eXclusive)</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>

2개의 세션에서 아래의 쿼리를 순서대로 수행해보자. S1 세션의 SID는 100, S2 세션의 SID는 200이다. S1 세션은 cd가 1인 행, S2 세션은 cd가 2인 행을 갱신한다.  

```sql
--S1 (100)
SELECT USERENV('SID') FROM DUAL; -- 100
UPDATE t1 SET v1 = v1 + 10 WHERE cd = 1;
```

```sql
--S2 (200)
SELECT USERENV('SID') FROM DUAL; -- 200
UPDATE t1 SET v1 = v1 - 10 WHERE cd = 2;
```

DBA&#95;LOCK 뷰 또는 V$LOCK 뷰에서 현재 설정된 락에 대한 정보를 조회할 수 있다. 또 다른 세션에서 DBA&#95;LOCK 뷰를 조회해보자. 아래 쿼리는 S1 세션(SID = 100)의 락 정보를 조회한다. TM 락은 RX 모드, TX 락은 X 모드로 설정되어 있다.  

```sql
SELECT lock_type, mode_held, mode_requested, blocking_others
FROM dba_lock
WHERE session_id = 100
AND lock_type in ('DML', 'Transaction');
```

아래 쿼리는 S2 세션(SID = 200)의 락 정보를 조회한다. TM 락은 RX 모드, TX 락은 X 모드로 설정되어 있다. TM 락이 동일한 테이블(t1)에 설정되었지만 RX 모드는 서로 호환되기 때문에 블로킹되지 않고 락을 획득할 수 있다. TX 락의 X 모드는 서로 호환되지 않지만 다른 행을 갱신했기 때문에 블로킹되지 않는다. 갱신되는 행에 로우 레벨 락을 설정하고, 로우 레벨 락을 X 모드의 TX 락에 연결한다.  

```sql
SELECT lock_type, mode_held, mode_requested, blocking_others
FROM dba_lock
WHERE session_id = 200
AND lock_type in ('DML', 'Transaction');
```

롤백을 수행하면 트랜잭션이 종료되고, 트랜잭션이 종료되면 락도 해제된다. DBA&#95;LOCK 뷰를 다시 조회하면 락이 해제된 것을 확인할 수 있다.  

```sql
SELECT lock_type, mode_held, mode_requested, blocking_others
FROM dba_lock
WHERE session_id IN (100, 200)
AND lock_type in ('DML', 'Transaction');
```

아래의 예제를 순서대로 수행해보자. 동일한 행(cd = 1)을 갱신했기 때문에 S2 세션은 S1 세션의 락이 해제될 때까지 블로킹(blocking)된다.  

아래 쿼리는 S1 세션(SID = 100)의 락 정보를 조회한다. blocking&#95;others 열에서 TX 락이 다른 세션을 블로킹하고 있는 것을 확인할 수 있다.  

```sql
SELECT lock_type, mode_held, mode_requested, blocking_others
FROM dba_lock
WHERE session_id = 100
AND lock_type in ('DML', 'Transaction');
```

아래 쿼리는 S2 세션(SID = 200)의 락 정보를 조회한다. TM 락은 S1 세션의 RX 모드와 호환되기 때문에 블로킹되지 않고 락을 획득할 수 있다. TX 락은 동일한 행을 갱신했기 때문에 블로킹된 상태로 X 모드를 요청하고 있다.  

```sql
SELECT lock_type, mode_held, mode_requested, blocking_others
FROM dba_lock
WHERE session_id = 200
AND lock_type in ('DML', 'Transaction');
```

다음과 같은 과정으로 S2 세션이 블로킹된다. 실제 수행 과정은 훨씬 더 복잡하다.

(1) 로우 레벨 락을 설정하기 위해 블록을 방문하여 로우 헤더(c1 = 1)를 확인  
(2) 로우 레벨 락이 설정되어 있는 것을 확인  
(3) 로오 레벨 락과 연결된 S1 세션의 TX 락이 X 모드인 것을 확인  
(4) 호환되지 않은 모드이므로 대기 목록에 트랜잭션을 등록하고 대기(블로킹)  

또 다른 세션에서 아래 쿼리를 수행하면 에러가 발생한다. DROP TABLE 문은 DDL 문이다. DDL 문은 TM 락을 X 모드로 설정한다. RX 모드와 X 모드는 호환되지 않기 때문에 DML 문이 수행중인 오브젝트에 DDL 문을 수행하면 에러가 발생한다.  

```sql
DROP TABLE t1;
```

S1 세션에서 롤백을 수행하면 S2 세션의 블로킹이 해제된다.  

#### 19.3.2. 동시성 제어
<br/>
multitier 환경에서는 개발자가 직접 데이터 동시성을 제어해야 한다.  

두 세션에서 아래 쿼리를 순서대로 실행해보자. S2 세션의 갱신 결과가 유실된 것을 확인할 수 있다. 이런 현상을 lost update라고 한다. 동시성 제어를 통해 lost update를 방지할 수 있다.  

```sql
---S1
COLUMN v1 NEW_VALUE v_v1;

SELECT v1 FROM t1 WHERE cd = 1;

---S2
UPDATE t1
SET v1 = v1 + 10
WHERE cd = 1;
COMMIT;

UPDATE t1
SET v1 = &v_v1 + 10
WHERE cd = 1;

SELECT v1 FROM v1 WHERE cd = 1;
```

##### 19.3.2.1. 비관적 동시성 제어
<br/>
비관적 동시성 제어(pessimistic concurrency control 또는 pessimistic locking)은 다수의 사용자가 동일한 데이터를 동시에 갱신하는 일이 빈번하다고 가정한다.  

###### 19.3.2.1.1. FOR UPDATE 절
<br/>
비관적 동시성 제어는 FOR UPDATE 절을 사용한다. SELECT 문에 FOR UPDATE 절을 기술하면 조회한 행에 로우 레벨 락이 설정된다.  

```
FOR UPDATE [OF [{table | view}.]column [, [{table | view}.]column]...]
           [{NOWAIT | WAIT integer | SKIP LOCKED}]
```

S1 세션에서 FOR UPDATE 절로 cd가 1인 행에 로우 레벨 락을 설정하고, S2 세션에서도 동일한 쿼리를 수행해보자. S2 세션은 S1 세션의 락이 해제될 때까지 블로킹된다. 갱신할 앵에 로우 레벨 락을 설정하고, 갱신할 값을 계산하는 작업을 수행한 후, 행을 갱신하는 업무에서 활요할 수 있다.  

```sql
---S1
SELECT * FROM t1 WHERE cd = 1 FOR UPDATE;
---작업

---S2
SELECT * FROM t1 WHERE cd = 1 FOR UPDATE;
---블로킹

UPDATE t1 SET v1 = v1 + 10 WHERE cd = 1;
COMMIT;

---S2
UPDATE t1 SET v1 = v1 - 10 WHERE cd = 1;
COMMIT;
```

아래 쿼리는 S2 세션에서 FOR UPDATE 절에 NOWAIT를 지정했기 때문에 즉시 에러가 발생한다.  

```sql
---S1
SELECT * FROM t1 WHERE cd = 1 FOR UPDATE;

---S2
SELECT * FROM t1 WHERE cd = 1 FOR UPDATE NOWAIT;

---S1
COMMIT;
```

아래 쿼리는 S2 세션에서 FOR UPDATE 절에 WAIT 10으로 지정했기 때문에 10초 후 에러가 발생한다. 블로킹이 발생하면 동시성이 저하될 수 있다. FOR UPDATE 절에 NOWAIT이나 WAIT integer를 지정하여 에러를 발생시킨 후 에러에 대한 예외 처리를 통해 동시성을 향상시킬 수 있다.  

```sql
---S1
SELECT * FROM t1 WHERE cd = 1 FOR UPDATE;

---S2
SELECT * FROM t1 WHERE cd = 1 FOR UPDATE WAIT 10;

---S1
COMMIT;
```

FOR UPDATE 절에 SKIP LOCKED를 지정하면 락이 설정된 행을 조회하지 않는다. 아래 예제의 S2 세션은 락이 설정되지 않은 c1이 2인 행만 조회된다. SKIP LOCKED는 동시성 제어보다 다중 큐(multiconsumer queue)를 구현할 때 사용되는 기능이다.  

```sql
---S1
SELECT * FROM t1 WHERE cd = 1 FOR UPDATE;
---S2
SELECT * FROM t1 FOR UPDATE SKIP LOCKED;
COMMIT;

---S1
COMMIT;
```

아래 쿼리는 조인이 포함된 SELECT 문에 FOR UPDATE 절을 사용했다. FOR UPDATE 절에 열을 지정하면 열을 지정한 테이블의 행에만 락이 설정된다.  

```sql
---S1
SELECT *
FROM emp a, dpet b
WHERE b.deptno = a.deptno
FOR UPDATE OF a.deptno;

---S2
SELECT * FROM dpet FOR UPDATE NOWAIT;

SELECT * FROM emp FOR UPDATE NOWAIT;

---S1
COMMIT;

---S2
COMMIT;
```

###### 19.3.2.1.2. 선분 이력 변경
<br/>
선분 이력 변경은 기존 이력을 단절(update)하고, 신규 이력을 생성(insert)하는 두 가지 작업을 하나의 트랜잭션으로 수행해야 한다. FOR UPDATE 절로 갱신은 방지할 수 있지만, 삽입은 방지할 수 없다. 이런 경우 FOR UPDATE 절로 부모 테이블에 락을 설정하는 기법을 사용할 수 있다.  

예제를 위해 아래와 같이 테이블을 생성하자. tp 테이블이 부모 테이블, tc 테이블이 선분 이력을 관리하는 자식 테이블이다. tc 테이블의 PK는 코드(cd), 시작일자(bd)다.  

```sql
DROP TABLE tp PURGE;
DROP TABLE tc PURGE;

CREATE TABLE tp (cd VARCHAR2(1), CONSTRAINT tp_pk PRIMARY KEY (cd));
CREATE TABLE tc (cd VARCHAR2(1) --- 코드
               , bd DATE        --- 시작일자
               , ed DATE        --- 종료일자
               , v1 NUMBER      --- 값
               , CONSTRAINT tc_pk PRIMARY KEY (cd, bd)
               , CONSTRAINT tc_f1 FOREIGN KEY (cd) REFERECES tp (cd));

INSERT INTO tp VALUES ('A');
INSERT INTO tc VALUES ('A', DATE '2050-01-01', DATE '9999-12-31', 1);
COMMIT;
```

아래 방식으로 선분 이력을 갱신할 수 있다. 자식 테이블(tc)을 갱신하기 전에 부모 테이블(tp)에 락을 설정했기 때문에 데이터 동시성을 확보할 수 있다. 다른 세션에서 동시에 cd가 A인 선분 이력을 변경하면 부모 테이블(tp)에 락을 설정하는 단계에서 블로킹된다.

```sql
SELECT * FROM tp WHERE cd = 'A' FOR UPDATE;

UPDATE tc --- 기조 이력 단절
SET ed = DATE '2050-02-01' - 1
WHERE cd = 'A'
AND ed = DATE '9999-12-31';

INSERT
INTO tc --- 신규 이력 생성
VALUES ('A', DATE '2050-02-01', DATE '9999-12-31', 2);

COMMIT;
```

tc 테이블의 PK를 코드(cd), 최종일자(ed)로 변경해보자. 성능상의 이유로 PK에 최종일자 속성을 사용하기도 한다.  

```sql
ALTER TABLE tc DROP CONSTRAINT tc_pk;
ALTER TABLE tc ADD CONSTRAINT tc_pk PRIMARY KEY (cd, ed);
```

아래 방식으로 선분 이력을 갱신할 수 있다. 기본 식별자가 변경되지 않아야 한다는 원칙을 준수하기 위해 다소 복잡한 방식으로 처리해야 한다.  

```sql
SELECT * FROM tp WHERE cd = 'A' FOR UPDATE;

INSERT INTO tc --- 기존 이력 단절
SELECT cd, bc, DATE '2050-03-01' - 1 AS ed, v1
FROM tc
WHERE cd = 'A'
AND ed = DATE '9999-12-31';

UPDATE tc --- 신규 이력 생성
SET bd = DATE '2050-03-01', v1 = 3
WHERE cd = 'A'
AND cd = DATE '9999-12-31';

COMMIT;
```

##### 19.3.2.2. 낙관성 동시성 제어
<br/>
낙관성 동시성 제어(optimistic concurrency control 또는 optimistic locking)은 다수의 사용자가 동일한 데이터를 동시에 갱신하는 일이 드물다고 가정한다.  

###### 19.3.2.2.1. 칼럼 확인 방식
<br/>
칼럼 확인 방식은 변경할 값을 조회하여 저장하고, 값을 변경하기 전에 저장한 값의 변경 여부를 확인하는 방식이다. 변경할 값이 많은 경우 모든 값을 비교해야 하기 때문에 쿼리가 길어질 수 있다.  

아래 쿼리는 칼럼 확인 방식으로 낙관적 동시성 제어를 구현했다. S1 세션에서 값을 조회한 후 S2 세션에서 값을 변경했기 때문에 S1 세션의 UPDATE 문에서 행이 갱신되지 않았다.  

```sql
--- S1
COLUMN vl NEW_VALUE v_v1;

SELECT v1 FROM t1 WHERE cd = 1;

--- S2
UPDATE t1 SET v1 = v1 + 10 WHERE cd = 1;
COMMIT;

--- S1
UPDATE t1
SET v1 = v1 + 10
WHERE cd = 1
AND v1 = &v_v1;
```

###### 19.3.2.2.2. 버전 확인 방식
<br/>
버전 확인 방식은 버전 정보를 변수에 저장하고, 값을 변경하기 전에 저장한 버전 정보의 변경 여부를 확인하는 방식이다. 버전 정보만 확인하면 되기 때문에 쿼리를 간결하게 작성할 수 있는 장점이 있지만, 추가적인 버전 속성을 생성해야 하는 단점도 있다.  

예제를 위해 아래와 같이 열을 추가하자. 버전 정보는 주로 시스템 속성으로 관리된다.  

```sql
ALTER TABLE t1 ADD md DATE DEFAULT DATE '2050-01-01' NOT NULL;
```

아래 쿼리는 버전 확인 방식으로 낙관적 동시성 제어를 구현한다. 갱신할 열이 많더라도 버전 정보가 저장된 md 열만 비교하면 된다.  

```sql
--- S1
COLUMN md NEW_VALUE md

SELECT md FROM t1 WHERE cd = 1;

--- S2
UPDATE t1
SET v1 = v1 - 10, md = SYSDATE
WHERE cd = 1;

COMMIT;

--- S1
UPDATE t1
SET v1 = v1 - 10
WHERE cd = 1
AND md = TO_DATE('&md', 'YYYY-MM-DD HH24:MI:SS');
```

### 19.4. 데이터 일관성
<br/>
데이터 일관성(data consistency)은 트랜잭션에 의한 변경을 일관된 상태로 볼 수 있는 것을 의미한다. 데이터 동시성과 데이터 일관성은 트레이드오프(tradeoff) 관계를 가진다. 동시성이 높아지면 일관성이 낮아지고, 일관서잉 높아지면 동시성이 낮아진다.  

+ 다중 버전 읽기 일관성  
오라클 데이터베이스는 다중 버전 읽기 일관성(Multiversion Read consistency, MVRC)을 지원한다. 다중 버전 읽기 일관성은 쿼리가 시작된 시점의 데이터를 조회할 수 있게 해주는 기능이다. 쿼리가 시작된 시점의 SCN과 블록의 SCN을 비교하여 쿼리가 시작된 이후에 블록이 변경되었다면 언두 세그먼트의 변경 이전 데이터를 사용하여 쿼리가 시작된 시점의 데이터를 반환한다.  

아래 예제를 순서대로 수행해보자. S1 세션에서 데이터를 변경했더라도 커밋 전까지는 S2 세션에서 변경 전 데이터가 조회된다. 커밋을 수행하기 전까지 변경된 데이터는 데이터를 변경한 세션에서만 조회할 수 있다.  

```sql
--- S2
SELECT v1 FROM t1 WHERE cd = 1; -- 40

--- S1
UPDATE t1 SET v1 = v1 + 10
WHERE cd = 1;
SELECT v1 FROM t1 WHERE cd = 1; -- 50

--- S2
SELECT v1 FROM t1 WHERE cd = 1; -- 40

--- S1
COMMIT;

--- S2
SELECT v1 FROM t1 WHERE cd = 1; -- 50
```

언두 세그먼트의 공간은 일정 시점이 지나면 재사용될 수 있다. 쿼리 시작 시점의 변경 이전 데이터가 저장된 공간이 재사용되면 해당 시점의 데이터를 반환할 수 없기 때문에 "ORA-01555: 너무 이전 스냅샷" 에러가 발생한다. 다중 버전 읽기 일관성의 부작용으로 생각할 수 있다.  

#### 19.4.1. 문장 수준 읽기 일관성
<br/>
문장 수준 읽기 일관성(statement-level read consistency)는 단일 쿼리 수준의 읽기 일관성이다. 오라클 데이터베이스는 다중 버전 읽기 일관성을 지원하기 때문에 락을 사용하지 않고 문장 수준 읽기 일관성을 보장한다.  

예제를 위해 SYS 사용자로 접속한 세션에서 SCOTT 사용자에게 DBMS&#95;LOCK 패키지의 실행 권한을 부여해야 한다.  

```sql
GRANT EXECUTE ON DBMS_LOCK TO scott;
```

아래와 같이 함수를 생성하자.  

```sql
CREATE OR REPLACE FUNCTION f1 (i_seconds IN NUMBER) RETURN NUMBER
IS
BEGIN
  DBMS_LOCK.SLEEP (i_seconds);
  RETURN 1;
END f1;
/
```

S1 세션의 첫 번째 쿼리는 f1 함수로 인해 10초 후 결과가 반환된다. 그 사이에 S2 세션에서 데이터를 변경하고 커밋을 수행하더라도 쿼리가 시작한 시점의 값(50)이 반환된다. t1 테이블을 다시 조회하면 변경된 값(40)이 반환된다.  

```sql
--- S1
SELECT *
FROM t1
WHERE cd = 1
AND f1 (10) = 1;

--- S2
UPDATE t1
SET v1 = v1 - 10
WHERE cd = 1;
COMMIT;

--- S1
SELECT * FROM t1 WHERE cd = 1;
```

#### 19.4.2. 트랜잭션 수준 읽기 일관성
<br/>
트랜잭션 수준 읽기 일관성(transaction-level read consistency)은 말 그대로 트랜잭션 수준의 읽기 일관성이다. 동일한 트랜잭션 내의 쿼리는 다시 조회하더라도 결과가 동일해야 한다. 오라클 데이터베이스는 기본적으로 트랜잭션 수준 읽기 일관성을 보장하지 않는다. 트랜잭션 수준 읽기 일관성을 보장하려면 트랜잭션 고립화 수준을 변경해야 한다.  

ANSI/ISO SQL 표준은 아래 네 가지의 트랜잭션 고립화 수준(transaction isolation level)을 정의하고 있다. 오라클 데이터베이스는 read committed, serialize 수준을 지원한다. 기본값은 read committed다.  

+ read uncommitted  
커밋되지 않은 데이터를 읽는 것을 허용
+ read committed  
커밋된 데이터만 읽는 것을 허용
+ repeatable reads  
트랜잭션 내에서 읽은 데이터는 갱신과 삭제를 금지  
+ serializable  
트랜잭션 내에서 읽은 테이블에 삽입을 금지  

아래는 읽기 일관서잉 보장되지 않을 때 발생할 수 있는 읽기 이상 현상(read phenomena)이다.  

+ dirty read  
동일한 쿼리가 커밋되지 않은 데이터를 읽어 다른 결과를 반환
+ non-repeatable read  
동일한 쿼리가 갱신 또는 삭제에 의해 다른 결과를 반환함
+ phantom read  
동일한 쿼리가 삽입에 의해 다른 결과를 반환  

트랜잭션 고립화 수준에 따라 아래와 같은 읽기 이상 현상이 발생할 수 있다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>dirty read</td>
      <td>non-repeatable read</td>
      <td>phantom read</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>read uncommitted</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>read committed</td>
      <td></td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>repeatable reads</td>
      <td></td>
      <td></td>
      <td>Y</td>
    </tr>
    <tr>
      <td>serializable</td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>

오라클 데이터베이스는 repeatable reads 수준을 지원하지 않지만 FOR UPDATE 절을 사용하여 non-repeatable read 현상을 회피할 수 있다. 아래 예제는 S1 세션에서 FOR UPDATE 절을 사용했기 때문에 S2 세션에서 값을 갱신할 수 없다. S1 세션에서 t1 테이블을 반복해서 조회하더라도 값이 변경되지 않는 것이 보장된다.  

```sql
--- S1
SELECT * FROM t1 FOR UPDATE;

--- S2
UPDATE t1 SET v1 = v1 + 10 WHERE cd = 1;
--- 블로킹

--- S1
SELECT SUM(v1) AS v1 FROM t1; --- 100
SELECT SUM(v1) AS v1 FROM t1; --- 100

--- S2
ROLLBACK;

--- S1
COMMIT;
```

아래 예제는 S1 세션에서 FOR UPDATE 절을 사용했지만 S2 세션에서 신규 행을 삽입하는 것을 방지할 수 없기 때문에 변경된 값(150)이 조회된다. FOR UPDATE 절을 사용하더라도 phantom read 현상을 방지할 수 없다.  

```sql
--- S1
SELECT * FROM t1 FOR UPDATE;

SELECT SUM(v1) AS val FROM t1; -- 100

--- S2
INSERT INTO t1(cd, v1) VALUES (3, 50);
COMMIT;

--- S1
SELECT SUM(v1) AS v1 FROM t1; -- 150
COMMIT;
```

##### 19.4.2.1. SET TRANSACTION 문
<br/>
트랜잭션 고립화 수준을 serializable 수준으로 설정하려면 SET TRANSACTION 문을 사용해야 한다. SET TRANSACTION 문의 구문은 아래와 같다. 기본값은 READ WRITE, READ COMMITTED다.  

```
SET TRANSACTION { {READ {ONLY | WRITE}
                 | ISOLATION LEVEL {SERIALIZE | READ COMMITTED}
                 | USE ROLLBACK SEGMENT rollback_segment}
                 } [NAME string]
                | NAME string};
```

+ READ ONLY  
읽기 전용 트랜잭션으로 설정 (트랜잭션 수준 읽기 일관성 보장)  
+ READ WRITE  
읽기 쓰기 트랜잭션으로 설정 (문자 수준 읽기 일관성 보장)  
+ SERIALIZE  
트랜잭션 고립화 수준을 SERIALIZE로 설정
+ READ COMMITTED  
트랜잭션 고립화 수준을 READ COMMITTED로 설정  

아래 예제는 SET TRANSACTION 문을 사용하여 트랜잭션 고힙화 수준을 SERIALIZE로 설정했기 때문에 phantom read 현상이 발생하지 않는다. COMMIT을 수행하면 트랜잭션이 종료된다.  

```sql
--- S1
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT SUM(v1) AS val FROM t1; -- 150

--- S2
INSERT INTO t1(cd, v1) VALUES (3, 50);
COMMIT;

--- S1
SELECT SUM(v1) AS v1 FROM t1; -- 150
COMMIT;
SELECT SUM(v1) AS v1 FROM t1; -- 200
```

아래 예제는 S1 세션에서 에러가 발생한다. 트랜잭션 고립화 수준을 SERIALIZABLE로 설정하면 다른 트랜잭션에 의해 변경 후 커밋된 데이터를 변경할 수 없다.  

```sql
--- S1
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

--- S2
UPDATE t1 SET v1 = v1 + 10 WHERE cd = 1;
COMMIT;

--- S1
UPDATE t1 SET v1 = v1 - 10 WHERE cd = 1;
ROLLBACK;
```
