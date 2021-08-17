---
title:  Flashback 기술
categories:
- Unkind_SQL
feature_text: |
  ## 31. Flashback 기술
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

Flashback 기술을 사용하면 과거 시점의 데이터를 조회하거나, 백업을 복구하지 않고 과거 시점으로 데이터를 되돌릴 수 있다. 주로 DBA가 사용하는 기능이지만 일부 기능을 애플리케이션 개발에 활용할 수 있다.  

### 31.1. Flashback 기능
<br/>
Flashback 기술은 용도에 따라 아래의 여섯 가지 기능으로 구분된다.  

<table>
  <thead>
    <tr>
      <td>용도</td>
      <td>기술</td>
      <td>사용</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">AP 개발</td>
      <td>Flash Query</td>
      <td>언두 세그먼트</td>
    </tr>
    <tr>
      <td>Flash Version Query</td>
      <td>언두 세그먼트</td>
    </tr>
    <tr>
      <td>Flash Transaction Query</td>
      <td>언두 세그먼트</td>
    </tr>
    <tr>
      <td rowspan="3">DB 관리</td>
      <td>Flash Table</td>
      <td>언두 세그먼트</td>
    </tr>
    <tr>
      <td>Flash Drop</td>
      <td>Recycle Bin</td>
      <td>Flash Database</td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 31.1.1. Flash Query
<br/>
Flash Query를 사용하면 expr에 지정한 시점의 데이터를 조회할 수 있다. Flash Query는 AS OF 절을 사용한다.  

```
[schema.]table AS OF {SCN | TIMESTAMP} expr
```

테스트를 위해 아래와 같이 테이블을 생성하자. Flash Table 기능을 사용하려면 ROW MOVEMENT 기능을 활성화해야 한다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER) ENABLE ROW MOVEMENT;
```

먼저 현재 SCN과 TIMESTAMP 값을 확인하자.  

```sql
SELECT current_scn, SYSTIMESTAMP FROM v$database;
```

1분 후 아래와 같이 데이터를 입력하자.  

```sql
INSERT INTO t1 VALUES (1, 1);
COMMIT;
```

아래는 Flash Query를 사용한 쿼리다. 2050-01-01 00:00:00 시점으로 조회했기 때문에 결과가 반환되지 않는다.  

```sql
SELECT * FROM t1 AS OF TIMESTAMP TIMESTAPM '2050-01-01 00:00:00';
```

아래와 같이 SCN을 사용할 수도 있다.  

```sql
SELECT * FROM t1 AS OF SCN 10000000;
```

조회 시점에 해당하는 언두 세그먼트의 공간이 재사용된 경우 에러가 발생한다.  

```sql
SELECT * FROM t1 AS OF TIMESTAMP TIMESTAMP '2049-12-01 00:00:00';

ORA-08180: 지정된 시간에 준하여 스냅샷을 찾을 수 없음
```

##### 31.1.1.1. DBMS&#95;FLASHBACK 패키지
<br/>
DBMS&#95;FLASHBACK 패키지는 Flashback과 관련된 다양한 기능을 제공한다. 자주 사용되는 아래의 서브 프로그램을 살펴보자.  

+ GET&#95;SYSTEM&#95;NUMBER 함수 : 현재 SCN을 반환
+ ENABLE&#95;AT&#95;SYSTEM&#95;CHANGE&#95;NUMBER 프로시저 : 세션 flashback을 활성화 (SCN)
+ ENABLE&#95;AT&#95;TIME 프로시저 : 세션 flashback을 활성화 (TIMESTAMP)
+ DISABLE 프로시저 : 세션 flashback을 비활성화  

아래는 GET&#95;SYSTEM&#95;CHANGE&#95;NUMBER 함수를 사용한 쿼리다. 현재 SCN을 확인할 수 있다.  

```sql
SELECT DBMS_FLASHBACK.GET_SYSTEM_CHANGE_NUMBER AS scn FROM DUAL;
```

아래는 ENABLE&#95;AT&#95;TIME 프로시저를 사용한 쿼리다. 세션 flashback을 활성화하면 Flashback Query와 동일한 결과를 얻을 수 있다. 해당 기능을 활용하면 트랜잭션 수준 읽기 일관성을 보장할 수 있다.  

```sql
EXEC DBMS_FLASHBACK.ENABLE_AT_TIME (TIMESTAMP '2050-01-01 00:00:00');

SELECT * FROM t1;

EXEC DBMS_FLASHBACK.DISABLE;
```

#### 31.1.2. Flashback Version Query
<br/>
Flashback Version Query를 사용하면 지정한 기간의 데이터 버전을 조회할 수 있다.  

```
[schema.]table VERSIONS BETWEEN {SCN | TIMESTAMP} {expr | MINVALUE}
                            AND {expr | MAXVALUE}
```

아래는 Flashback Version Query에서 사용할 수 있는 슈도 칼럼이다.  

+ versions&#95;starttime : 버전이 생성된 시간
+ versions&#95;endtime : 버전이 만료된 시간
+ versions&#95;startscn : 버전이 생성된 SCN
+ versions&#95;endscn : 버전이 만료된 SCN
+ versions&#95;xid : 버전의 트랜잭션 ID
+ versions&#95;operation : 버전의 오퍼레이션 (I: INSERT, D: DELETE, U: UPDATE)  

테스트를 위해 아래와 같이 데이터를 갱신하자.  

```sql
UPDATE t1 SET c2 = 2 WHERE c1 = 1;
COMMIT;
```

1분 후 아래와 같이 데이터를 삭제하자.  

```sql
DELETE t1 WHERE c1 = 1;
COMMIT;
```

아래는 Flashback Version Query 기능을 사용한 쿼리다. 3개의 데이터 버전이 존재하는 것을 확인할 수 있다.  

```sql
SELECT c2, versions_starttime, versions_endtime, versions_xid, versions_operation
FROM t1 VERSIONS BETWEEN TIMESTAMP TIMESTAMP '2050-01-01 00:00:00' AND MAXVALUE
ORDER BY versions_starttime;
```

아래와 같이 SCN을 사용할 수도 있다.  

```sql
SELECT c2, versions_startscn, versions_endscn, versions_xid, versions_operation
FROM t1 VERSIONS BETWEEN SCN 10000000 AND MAXVALUE
ORDER BY versions_startscn;
```

#### 31.1.3. Flashback Transaction Query
<br/>
Flashback Transaction Query를 사용하면 단일 트랜잭션이나 특정 기간의 트랜잭션에 의한 변경 정보를 조회할 수 있다.  

아래는 Flashback Transaction Query를 사용한 쿼리다.  

```sql
SELECT start_timestamp, commit_timestamp, logon_user, operation, undo_sql
FROM flashback_transaction_query
WHERE xid = HEXTORAW ('CCCCCCCCCCCCCCCC')
ORDER BY undo_change#;
```

minimal supplemental logging 기능이 활성화되어야 Flashback Transaction Query의 모든 정보를 조회할 수 있다.  

```sql
SELECT supplemental_log_data_min FROM v$database;
```

아래의 ALTER DATABASE 문으로 minimal supplemental logging 기능을 활성화할 수 있다.  

```sql
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
```

#### 31.1.4. Flashback Table
<br/>
Flashback Table을 사용하면 지정한 시점으로 테이블을 되돌릴 수 있다. RESTORE POINT은 사용자가 생성할 수 있는 복원 시점이다. RESTORE POINT는 CREATE RESTORE POINT 문으로 생성할 수 있다.  

```
FLASHBACK TABLE [schema.]table [, [schema.]table]...
  TO { {SCN | TIMESTAMP} expr | RESTORE POINT restore_point}}
     [{ENABLE | DISABLE} TRIGGERS];
```

아래는 Flashback Table 기능을 사용한 쿼리다. t1 테이블을 2050-01-01 00:01:00 시점으로 되돌린다.  

```sql
FLASHBACK TABLE t1 TO TIMESTAMP TIMESTAMP '2050-01-01 00:01:00';
```

아래와 같이 SCN 값을 사용할 수도 있다.  

```sql
FLASHBACK TABLE t1 TO SCN 10000010;
```

아래는 t1 테이블을 조회한 결과다.  

```sql
SELECT * FROM t1;
```

#### 31.1.5. Flashback Drop
<br/>
Flashback Drop을 사용하면 삭제된 테이블을 복원할 수 있다.  

```
FLASHBACK TABLE [schema.]table [, [schema.]table]... TO BEFORE DROP [RENAME TO table];
```

아래는 PURGE 문의 구문이다. RECYCLEBIN은 현재 사용자의 휴지통, DBA&#95;RECYCLEBIN은 전체 휴지통을 비운다.  

```
PURGE { {TABLE table | INDEX index}
      | {RECYCLEBIN | DBA_RECYCLEBIN}
      | TABLESPACE tablespace [USER username]};
```

테스트를 위해 아래와 같이 PURGE 문으로 휴지통을 비우고 테이블을 삭제하자. t1 테이블은 PURGE 키워드를 사용하지 않았다. PURGE 키워드를 사용하지 않으면 삭제한 테이블이 휴지통에 보관된다.  

```sql
PURGE RECYCLEBIN;

DROP TABLE t1;
DROP TABLE t2 PURGE;
```

&#42;&#95;RECYCLEBIN 뷰에서 삭제된 t1 테이블을 확인할 수 있다.  

```sql
SELECT object_name, original_name, operation, droptime FROM user_recyclebin;
```

아래는 Flashback Drop 기능을 사용한 쿼리다. 삭제된 t1 테이블을 t2 테이블로 복원한다. 동일한 테이블이 여러 번 삭제된 경우 가장 최근에 삭제된 테이블이 복원된다. 특정 시점에 삭제된 테이블을 복원하려면 오브젝트 명(BIN$dJ4flKy0TRKtxDVDEoz6Pg==$0)을 사용해야 한다.  

```sql
FLASHBACK TABLE t1 TO BEFORE DROP RENAME TO t2;
```

아래는 t2 테이블을 조회한 결과다.  

```sql
SELECT * FROM t2;
```

#### 31.1.6. Flashback Database
<br/>
Flashback Database 기능은 지정한 시점으로 데이터베이스를 되돌린다.  

```
FLASHBACK [STANDBY] DATABASE [database]
    {TO { {SCN | TIMESTAMP} expr | RESTORE POINT restore_point}}
  | {TO BEFORE { {SCN | TIMESTAMP} expr | RESETLOGS}};
```

Flashback Database 기능을 활성화해야 FLASHBACK DATABASE 문을 사용할 수 있다. 아래 쿼리로 활성화 여부를 확인할 수 있다.  

```sql
SELECT flashback_on FROM v$database;
```

SYSDBA 사용자로 로그인하여 아래의 명령어와 구문을 수행하면 해당 기능을 활성화할 수 있다.  

```cmd
conn /as sysdba

shutdown immediate
startup mount
ALTER DATABASE ARCHIVELOG;
ALTER DATABASE FLASHBACK ON;
ALTER DATABASE OPEN;
```

SYSDBA 사용자로 로그인하여 아래 명령어와 구문을 수행하면 데이터베이스를 10분 전으로 되돌릴 수 있다.  

```cmd
conn /as sysdba

shutdown immediate
startup mount
FLASHBACK DATABASE TO TIMESTAMP (SYSTIMESTAMP - INTERVAL '10' MINUTE);
ALTER DATABASE OPEN RESETLOGS;
```

### 31.2. Temporal Validity
<br/>
Temporal Validity 기능을 사용하면 유효한 시점이나 기간에 해당하는 데이터를 조회할 수 있다.  

#### 31.2.1. 관리 구문
<br/>
##### 31.2.1.1. PERIOD FOR 절
<br/>
PERIOD FOR 절은 신규 테이블에 valid time dimension을 생성한다.  

```
PERIOD FOR valid_time_column [(start_time_column, end_time_column)];
```

아래는 PERIOD FOR 절을 사용한 쿼리다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, PERIOD FOR vt);
```

&#42;&#95;TAB&#95;COLS 뷰에서 vt&#95;start 열, vt&#95;end 열, vt 열이 생성된 것을 확인할 수 있다. 3개의 열 모두 INVISIBLE 칼럼으로 생성된다.  

```sql
SELECT column_name, data_type, nullable, hidden_column
FROM user_tab_cols
WHERE table_name = 'T1'
ORDER BY internal_column_id;
```

아래와 같이 기존 열을 지정할 수도 있다.  

```sql
DROP TABLE t1 PURGE;
```

##### 31.2.1.2. ADD PERIOD FOR 절
<br/>
ADD PERIOD FOR 절은 기존 테이블에 valid time dimension을 추가한다.  

```
ADD (PERIOD FOR valid_time_column [(start_time_column, end_time_column)]);
```

아래는 ADD PERIOD FOR 절을 사용한 쿼리다. 하나의 테이블에 다수의 valid time dimension을 생성할 수도 있다.  

```sql
ALTER TABLE t1 ADD PERIOD FOR vt2 (c4, c5);
```

##### 31.2.1.3. DROP PERIOD FOR 절
<br/>
DROP PERIOD FOR 절은 테이블의 valid time dimension을 삭제한다.  

```
DROP (PERIOD FOR valid_time_column);
```

아래는 DROP PERIOD FOR 절을 사용한 쿼리다.  

```sql
ALTER TABLE t1 DROP (PERIOD FOR vt2);
```

##### 31.2.2.1. AS OF PERIOD FOR 절
<br/>
AS OF PERIOD FOR 절은 지정한 시점의 유효한 데이터를 조회한다.  

```
[schema.]table AS OF PERIOD FOR valid_time_column_expr
```

테스트를 위해 아래와 같이 데이터를 입력하자.  

```sql
INSERT INTO t1 (c1, c2, c3) VALUES (1, DATE '2000-01-01', DATE '2010-01-01');
INSERT INTO t1 (c1, c2, c3) VALUES (2, DATE '2000-01-01', DATE '2010-01-01');
INSERT INTO t1 (c1, c2, c3) VALUES (3, DATE '2000-01-01', DATE '9999-12-31');
COMMIT;
```

아래는 AS OF PERIOD FOR 절을 사용한 쿼리다. 2010-01-01 시점의 유효한 데이터를 조회한다.  

```sql
SELECT c1, c2, c3 FROM t1 AS OF PERIOD FOR vt1 DATE '2010-01-01';
```

아래 쿼리는 2050-01-01 시점의 유효한 데이터를 조회한다.  

```sql
SELECT * FROM t1 AS OF PERIOD FOR vt1 DATE '2050-01-01';
```

위 쿼리는 아래 쿼리의 동일하게 동작한다.  

```sql
SELECT * FROM t1 WHERE c2 <= DATE '2050-01-01' AND c3 > DATE '2050-01-01';
```

##### 31.2.2.2. VERSIONS PERIOD FOR 절
<br/>
AS OF PERIOD FOR 절은 지정한 기간의 유효한 데이터를 조회한다.  

```
[schema.]table VERSIONS PERIOD FOR valid_time_column BETWEEN {expr | MINVALUE} AND {expr | MAXVALUE}
```

아래는 VERSIONS PERIOD FOR 절을 사용한 쿼리다.  

```sql
SELECT c1, c2, c3
FROM t1 VERSIONS PERIOD FOR vt1 BETWEEN DATE '2010-01-01' AND DATE '2050-01-01';
```

위 쿼리는 아래 쿼리와 동일하게 동작한다.  

```sql
SELECT * FROM t1 WHERE c2 <= DATE '2050-01-01' AND c3 > DATE '2050-01-01';
```

#### 31.2.3. DBMS&#95;FLASHBACK&#95;ARCHIVE 패키지
<br/>
DBMS&#95;FLASHBACK&#95;ARCHIVE 패키지는 FDA(Flashback Data Archive)와 관련된 기능을 제공한다. Temporal Validity와 관련된 ENABLE&#95;AT&#95;VALID&#95;TIME 프로시저를 살펴보자.  

ENABLE&#95;AT&#95;VALID&#95;TIME 프로시저는 세션 valid time flashback을 활성화한다.  

```
DBMS_FLASHBACK_ARCHIVE.ENABLE_AT_VALID_TIME (
    level IN VARCHAR2
  , query_time IN TIMESTAMP DEFAULT SYSTIMESTAMP);
```

level은 아래와 같이 지정할 수 있다.  

+ ALL : 전체 데이터를 조회 (기본값)
+ CURRENT : 현재 시점의 데이터를 조회
+ ASOF : 지정한 시점의 데이터를 조회  

아래는 ENALBE&#95;AT&#95;VALID&#95;TIME 프로시저를 사용한 쿼리다. 현재 시점의 데이터를 조회한다.  

```sql
EXEC DBMS_FLASHBACK_ARCHIVE.ENABLE_AT_VALID_TIME ('CURRENT');

SELECT * FROM t1;
```

아래 쿼리는 2050-01-01 시점의 유효한 데이터를 조회한다.  

```sql
EXEC DBMS_FLASHBACK_ARCHIVE.ENABLE_AT_VALID_TIME ('ASOF', DATE '2050-01-01');

SELECT * FROM t1;
```

level을 ALL로 설정하면 전체 데이터가 조회된다.  

```sql
EXEC DBMS_FLASHBACK_ARCHIVE.ENABLE_AT_VALID_TIME ('ALL');
```
