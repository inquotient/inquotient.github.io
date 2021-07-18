---
title:  블로킹 처리
categories:
- Unkind_SQL
feature_text: |
  ## G. 블로킹 처리
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

블로킹이 장시간 지속되면 시스템 장애가 발생할 수 있다. 블로킹 처리 방법을 살펴보자.  

아래 쿼리를 사용하면 블로킹과 관련된 세션을 계층 구조로 조회할 수 있다.  

```sql
SELECT w1 AS (SELECT /* MATERIALIZE */ * FROM gv$session)
     , w2 AS (SELECT a.*, LEVEL AS lv, ROWNUM AS rn
              FROM w1 a
              STRAT WITH EXISTS (SELECT 1
                                 FROM w1 x
                                 WHERE x.blocking_instance = a.inst_id
                                 AND x.blocking_session = a.sid)
                         AND a.blocking_session = IS NULL
              CONNECT BY PRIOR a.inst_id = a.blocking_instance
                     AND PRIOR a.sid = a.blocking_session
              ORDER SIBLINGS BY a.inst_id, a.sid, a.wait_time_micro)
SELECT a.inst_id -- 인스턴스 ID
     , LPAD(' ', (a.lv - 1) * 2) || a.sid AS sid -- 세션 ID
     , a.serial# -- 세션 일련번호
     , b.spid -- OS 프로세스 ID
     , c.lock_type -- 락 유형
     , c.mode_requested -- 락 유형
     , c.sql_id -- SQL ID
       /*
     , a.prev_sql_id
     , CASE WHEN c.owner IS NOT NULL
            THEN c.owner || '.' || c.object_name
            ELSE b.lock_id1
       END AS object_name -- 오브젝트 명
     , a.event -- 대기 이벤트
     , a.wait_class -- 대기 이벤트 클래스
       */
     , a.seconds_in_wait -- 대기 시간(초)
FROM w2 a
   , gv$process b
   , dba_lock_internal c
   , dba_objects d
WHERE b.inst_id = a.inst_id
AND b.addr = a.paddr
AND c.session_id(+) = a.sid
AND c.mode_requested(+) = <> 'None'
AND d.object_id(+) = a.row_wait_obj#
ORDER BY rn;
```

오라클 데이터베이스가 설치된 서버에 접속할 수 있다면 OS 명령어를 사용하여 블로킹하고 있는 세션을 종료시킬 수 있다.  

오라클 데이터베이스가 Unix 계열의 서버에 설치되어 있다면 kill 명령어로 프로세스를 종료시킬 수 있다. ps 명령어와 grep 명령어로 프로세스를 확인한 후 kill -9 {spid} 형식으로 kill 명령어를 수행하면 된다.  

```shell
ps -ef | grep -v grep | grep sqlplus
...
kill -9 100
```

오라클 데이터베이스가 Windows 서버에 설치되어 있다면 orakill 명령어 프로세스를 종료시킬 수 있다. tasklist 명령어와 find 명령어로 프로세스를 확인한 후 orakill {sid} {spid} 형식으로 orakill 명령어를 수행하면 된다.  

```
tasklist | find "sqlplus"
...
orakill 100 1000
```

SQL 문으로도 세션을 종료시킬 수 있다. ALTER SYSTEM 문의 KILL SESSION 절, DISCONNECT SESSION 절, CANCEL SQL 절을 사용하면 된다.  

아래는 KILL SESSION 절의 구문이다. IMMEDIATE 키워드를 기술하면 세션이 종료될 때까지 대기하지 않고 즉시 제어를 반환한다.  

```
ALTER SYSTEM KILL SESSION 'session_id, serial_number [, @instance_id]' [IMMEDIATE];
```

아래 쿼리는 1번 인스턴스의 세션 일련번호가 10000인 100번 세션을 즉시 종료시킨다.  

```sql
ALTER SYSTEM KILL SESSION '100, 10000, @1' IMMEDIATE;
```

아래는 DISCONNECT SESSION 절의 구문이다. DISCONNECT SESSION 절은 서버 프로세스를 종료시킨다. IMMEDIATE 키워드를 사용하면 kill, orakill 명령어와 유사하게 동작한다. POST&#95;TRANSACTION 키워드를 사용하면 현재 수행중인 트랜잭션이 종료된 후에 서버 프로세스를 종료시킨다.  

```sql
ALTER TABLE DISCONNECT SESSION 'session_id, serial_number' {POST_TRANSACTION | IMMEDIATE};
```

아래 쿼리는 일련번호가 10000인 100번 세션을 즉시 종료한다. 인스턴스 ID를 지정할 수 없으므로 블로킹이 발생한 인스턴스에서 쿼리를 수행해야 한다.  

```sql
ALTER SYSTEM DISCONNECT SESSION '100, 10000' IMMEDIATE;
```

아래는 CANCEL SQL 절의 구문이다. sql&#95;id에 지정한 SQL 문을 종료시킨다. CANCEL SQL 절은 18.1 버전부터 사용할 수 있다.  

```
ALTER SYSTEM CANCEL SQL 'session_id serial_number @instance_id sql_id';
```

아래 쿼리는 일련번호가 10000인 100번 세션의 SQL ID가 58zls77k33skb인 SQL 문을 종료시킨다.  

```sql
ALTER SYSTEM CANCEL SQL '100 10000 @1 58zls77k33skb';
```
