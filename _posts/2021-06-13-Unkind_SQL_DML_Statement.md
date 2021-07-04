---
title:  DML 문
categories:
- Unkind_SQL
feature_text: |
  ## 18. DML 문
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

DML은 Data Manipulation Language의 약자다. 데이터 조작어로 해석할 수 있다. DML 문을 사용하면 테이블에 신규 행을 삽입하거나, 테이블의 기존 행을 갱신, 삭제할 수 있다.  

### 18.1. INSERT 문
<br/>
INSERT 문을 사용하면 테이블에 신규 행을 삽입할 수 있다. INSERT 문은 삽입할 테이블의 개수에 따라 단을 테이블 INSERT 문과 다중 테이블 INSERT 문으로 구분할 수 있다.  

#### 18.1.1. 단일 테이블 INSERT
<br/>
하나의 테이블에 행을 삽입한다. VALUES 절과 서브 쿼리 방식을 사용할 수 있다.  

##### 18.1.1.1. VALUES 절
<br/>
INSERT 문에 VALUES 절을 사용하면 단일 행을 삽입할 수 있다.  

```
INSERT INTO {table | view | subquery} [t_alias] [(column [, column]...)]
       VALUES ({expr | DEFAULT} [, {expr | DEFAULT}]...)
```

예제를 위해 아래와 같이 테이블을 생성하자. deptno 열의 기본값을 20으로 지정했다.  

```sql
CREATE TABLE t1 (empno NUMBER(4), ename VARCHAR2(10), deptno NUMBER(2) DEFAULT 20);
```

아래 쿼리는 t1 테이블에 단일 행을 삽입한다. INTO 절에 삽입할 열을 지정했고, VALUES 절에 삽입할 값을 기술했다.  

```sql
INSERT INTO t1(empno, ename, deptno) VALUES (7369M 'SMITHM', 20);
```

아래 쿼리는 INTO 절에 empno 열만 지정했다. INTO 절에 지정되지 않은 열은 기본값이 있으면 기본값, 기본값이 없으면 널이 삽입된다.  

```sql
INSERT INTO t1 (empno) VALUES (7566);
```

아래 쿼리는 INTO 절에 열을 지정하지 않았다. INTO 절에 열을 지정하지 않으면 VALUES 절에 전체 열의 값을 기술해야 한다.  

```sql
INSERT INTO t1 VALUES (7788, 'SCOTT', 20);
```

기본값이 지정된 열은 VALUES 절에 DEFAULT 키워드를 기술할 수 있따. 아래 쿼리는 기본값이 지정된 deptno 열에 DEFAULT 키워드를 기술했다.  

```sql
INSERT INTO t1 VALUES (7876, 'ADAMS', DEFAULT);
```

열 개수와 값 개수가 일치하지 않으면 에러가 발생한다.  

INTO 절에 열을 지정하는 편이 쿼리의 안정성 측면에서 바람직하다.  

##### 18.1.1.2. 서브 쿼리
<br/>
INSERT 문에 서브 쿼리를 사용하면 서브 쿼리의 결과를 테이블에 삽입할 수 있다. 서브 쿼리의 결과가 다중 행이면 다중 행이 삽입된다.  

```sql
INSERT INTO {table | view | subquery} [t_alias] [(column [, column]...)]
```

아래는 INSERT 문에 서브 쿼리를 사용한 쿼리다. job이 ANALYST인 행을 t1 테이블에 삽입했다.  

```sql
INSERT INTO t1 (empno, ename)
SELECT empno, ename FROM emp WHERE job = 'ANALYST';
```

서브 쿼리에 UNION ALL 연산자를 사용하면 여러 테이블의 행을 삽입할 수도 있다.  

```sql
INSERT INTO t1 (empno, ename)
SELECT empno, ename FROM emp WHERE job = 'PRESIDENT'
UNION ALL
SELECT deptno, dname FROM dept WHERE deptno = 10;
```

#### 18.1.2. 다중 테이블 INSERT 문
<br/>
다수의 테이블에 행을 삽입한다. 조건의 유무에 따라 무조건 INSERT 문과 조건부 INSERT 문으로 구분할 수 있다.  

##### 18.1.2.1. 무조건 INSERT 문
<br/>
무조건(unconditional) INSERT 문은 INTO 절에 지정한 모든 테이블에 서브 쿼리의 결과를 삽입한다.  

```sql
INSERT ALL {INTO table [(column [, column]...)] [VALUES ({expr | DEFAULT} [, {expr | DEFAULT}]...)]}...
subquery
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
CREATE TABLE t1 (empno NUMBER(4), job VARCHAR2(9));
CREATE TABLE t2 (empno NUMBER(4), mgr NUMBER(4));
```

아래 쿼리는 서브 쿼리의 결과를 t1, t2 테이블에 삽입한다.  

```sql
INSERT ALL
INTO t1 (empno, job) VALUES (empno, job)
INTO t2 (empno, mgr) VALUES (empno, mgr)
SELECT * FROM emp WHERE deptno = 10;
```

무조건 INSERT 문을 사용하면 PIVOT한 데이터를 삽입할 수 있다. 예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
CREATE TABLE t1 (deptno NUMBER(2), tp VARCHAR2(3), sal NUMBER(7, 2));
```

아래 쿼리는 INTO 절에 t1 테이블을 4번 기술했다. 서브 쿼리의 1행이 t1 테이블에 4번 삽입된다. 아래와 같은 INSERT 문을 PIVOT INSERT 문으로 부르기도 한다.  

```sql
INSERT ALL
INTO t1 VALUES (deptno, 'MIN', sal_min)
INTO t1 VALUES (deptno, 'MAX', sal_max)
INTO t1 VALUES (deptno, 'SUM', sal_sum)
INTO t1 VALUES (deptno, 'AVG', sal_avg)
SELECT deptno
     , MIN (sal) AS sal_min, MAX (sal) AS sal_max
     , SUM (sal) AS sal_sum, AVG (sal) AS sal_avg
FROM emp
GROUP BY deptno;
```

아래와 같이 UNPIVOT 절을 사용해도 동일한 결과를 얻을 수 있지만, PIVOT INSERT 문을 사용하는 편이 성능 측면에서 효율적이다.  

```sql
INSERT INTO t1 (deptno, tp, sal)
SELECT *
FROM (SELECT deptno
           , MIN (sal) AS sal_min, MAX (sal) AS sal_max
           , SUM (sal) AS sal_sum, AVG (sal) AS sal_avg
      FROM emp
      GROUP BY deptno)
UNPIVOT (sal FOR tp IN (sal_min AS 'MIN', sal_max AS 'MAX'
                      , sal_sum AS 'SUM', sal_avg AS 'AVG'));
```

##### 18.1.2.2. 조건부 INSERT 문
<br/>
조건부(conditional) INSERT 문은 서브 쿼리의 결과에서 condition을 만족하는 행을 INTO 절에 지정한 테이블에 삽입한다. 모든 condition을 만족하지 않은 행은 ELSE 절에 지정한 테이블에 삽입되고, ELSE 절이 기술되지 않았다면 무시된다.  

```sql
INSERT [ALL | FIRST]
 WHEN condition THEN
 INTO table [(column [, column]...)] [VALUES ([expr | DEFAULT] [, {expr | DEFAULT}]...)]
[WHEN condition THEN
 INTO table [(column [, column]...)] [VALUES ([expr | DEFAULT] [, {expr | DEFAULT}]...)]]
[ELSE condition THEN
 INTO table [(column [, column]...)] [VALUES ([expr | DEFAULT] [, {expr | DEFAULT}]...)]]
subquery
```

조건부 INSERT 문은 ALL 방식과 FIRST 방식을 사용할 수 있다.  

+ ALL : 조건을 만족하는 모든 테이블에 삽입 (기본값)
+ FIRST : 조건을 만족하는 첫 번째 테이블에 삽입  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
CREATE TABLE t1 AS SELECT empno, ename, sal FROM emp WHERE 0 = 1;
CREATE TABLE t2 AS SELECT * FROM t1;
CREATE TABLE t3 AS SELECT * FROM t1;
```

아래는 ALL 방식을 사용한 쿼리다. 조건을 만족하는 모든 테이블에 서브 쿼리의 결과 삽입된다.  

```sql
INSERT ALL
WHEN sal >= 2000 THEN INTO t1
WHEN sal >= 3000 THEN INTO t2
ELSE INTO t3
SELECT empno, ename, sal FROM emp WHERE deptno = 10;
```

아래는 FIRST 방식을 사용한 쿼리다. FIRST 방식은 WHEN 절의 기술 순서에 주의해야 한다.  

```sql
INSERT FIRST
WHEN sal >= 2000 THEN INTO t1
WHEN sal >= 3000 THEN INTO t2
ELSE INTO t3
SELECT empno, ename, sal FROM emp WHERE deptno = 10;
```

### 18.2. UPDATE 문
<br/>
UPDATE 문을 사용하면 테이블의 기존 행을 갱신할 수 있다.  

```
UPDATE {table | view | subquery} [t_alias]
SET {column = {expr | (subquery) | DEFAULT} | (column [, column]...) = (subquery)}
 [, {column = {expr | (subquery) | DEFAULT} | (column [, column]...) = (subquery)}]...
WHERE condition;
```

예제를 위해 아래와 같이 테이블을 생성하자. t1, t2 테이블은 1:M 비식별 관계를 가진다.  

```sql
CREATE TABLE t1 AS SELECT deptno, dname, 0 AS sal, 0 AS comm FROM dept;
CREATE TABLE t2 AS SELECT empno, ename, sal, comm, deptno FROM emp;

ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY (deptno);
ALTER TABLE t2 ADD CONSTRAINT t2_pk PRIMARY KEY (empno);
ALTER TABLE t2 ADD CONSTRAINT t2_fk PRIMARY KEY (deptno) REFERENCES t1 (deptno);
```

아래 쿼리는 40번 부서의 sal를 10000, comm을 1000으로 갱신한다.  

```sql
UPDATE t1 SET sal = 10000, comm = 1000 WHERE deptno = 40;
```

아래 쿼리는 SET 절에 다중 열 서브 쿼리를 사용했다. 서브 쿼리의 결과로 값이 갱신된다.  

```sql
UPDATE t1 a
SET (a.sal, a.comm) = (SELECT SUM(x.sal), SUM(x.comm) FROM t2 x WHERE x.deptno = a.deptno);
```

아래와 같이 WHERE 절에 상관 서브 쿼리를 사용하면 불필요한 갱신을 방지할 수 있다.  

```sql
UPDATE t1 a
SET (a.sal, a.comm) = (SELECT SUM(x.sal), SUM(x.comm) FROM t2 x WHERE x.deptno = a.deptno)
WHERE EXISTS (SELECT 1 FROM t2 x WHERE x.deptno = a.deptno);
```

UPDATE 문에도 인라인 뷰를 사용할 수 있다. 아래 쿼리는 위 쿼리와 결과가 동일하다. 인라인 뷰를 통해 t2 테이블을 1번만 읽고 갱신을 수행했다. 위 쿼리에 비해 성능 측면에서 효율적이다.  

```sql
UPDATE (SELECT a.sal, a.comm, b.sal AS sal_n, b.comm AS comm_n
        FROM t1 a
           , (SELECT deptno, SUM(sal) AS sal, SUM(comm) AS comm
              FROM t2
              GROUP BY deptno) b
        WHERE b.deptno = a.deptno)
SET sal = sal_n, comm = comm_n;
```

위 쿼리는 11.2 이하 버전에서 "ORA-01779: 키-보존된 것이 아닌 테이블로 대응한 열을 갱신할 수 없습니다" 에러가 발생한다. t2 테이블을 조인 조건인 deptno 열로 그룹핑했기 때문에 조인 차수가 1:1이다. t1 테이블의 단일 행이 단일 값으로 갱신되는 것이 보장되므로 불필요한 에러를 발생시킨 것이다. 12.1 버전 부터는 에러가 발생하지 않고 11.2 이하 버전에서는 아래와 같이 MERGE 문을 사용해야 한다. 10.2 이하 버전에서 BYPASS&#95;UJVC 힌트를 사용하면 에러가 발생하지 않는다. 해당 힌트는 문서화되지 않은 힌트다. MERGE 문을 사용하는 편이 바람직하다.  

```sql
MERGE INTO t1 t
USING (SELECT deptno, SUM(sal) AS sal, SUM(comm) AS comm
       FROM t2
       GROUP BY deptno) s
ON (t.deptno = s.deptno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal, t.comm = s.comm;
```

아래 쿼리는 버전과 관계없이 에러가 발생한다. 갱신할 테이블(t1)과 나머지 테이블(t2)의 조인 차수가 1:M이면, 1쪽 테이블(t1)의 값이 M쪽 테이블(t2)의 값으로 여러 번 갱신될 수 있기 때문이다.  

```sql
UPDATE (SELECT a.sal, a.comm, b.sal AS sal_n, b.comm AS comm_n
        FROM t1 a, t2 b
        WHERE b.deptno = a.deptno)
SET sal = sal_n, comm = comm_n;
```

M쪽 테이블(t2)의 값을 갱신하면 에러가 발생하지 않는다. M쪽 테이블(t2)의 값이 1쪽 테이블(t1)의 값으로 1번만 갱신되는 것이 보장되기 때문이다. 1:M 조인 차수에서 M쪽 테이블의 행이 늘어나지 않는 것과 동일한 원리다.  

```sql
UPDATE (SELECT a.sal, a.comm, b.sal AS sal_n, b.comm AS comm_n
        FROM t2 a, t1 b
        WHERE b.deptno = a.deptno)
SET sal = sal_n, comm = comm_n;
```

t1 테이블의 PK 제약 조건을 삭제하고 동일한 쿼리를 수행하면 에러가 발생한다. t1 테이블에 PK 제약 조건이 없기 때문에 t2 테이블의 값이 1번만 갱신되는 것을 보장할 수 없기 때문이다.  

아래 쿼리는 버전과 관계없이 정상적으로 동작한다. t1 테이블과 dept 테이블이 deptno 열로 등가 조인되기 때문에 조인 차수가 1:1이다. t1 테이블의 단일 행이 단일 값으로 갱신되는 것이 보장된다.  

```sql
UPDATE (SELECT a.*, b.dname AS dname_n
        FROM t1 a, dept b
        WHERE b.deptno = a.deptno)
SET dname = dname_n;
```

### 18.3. DELETE 문
<br/>
DELETE 문을 사용하면 테이블의 기존 행을 삭제할 수 있다.  

```sql
DELETE FROM {table | view | subquery} [t_alias]
WHERE condition;
```

아래 쿼리는 t1 테이블에서 10번 부서를 삭제한다.  

```sql
DELETE FROM t1 WHERE deptno = 10;
```

WHERE 절을 기술하지 않으면 테이블의 모든 행이 삭제된다.  

```sql
DELETE FROM t1;
```

#### 18.3.1. 실수 방지
<br/>
UPDATE 문과 DELETE 문은 수행하기 전에 갱신 또는 삭제할 행을 반드시 확인해야 한다. 먼저 SELECT 문을 작성하고, SELECT 문을 UPDATE 문이나 DELETE 문으로 변경하는 방식을 사용해야 실수를 방지할 수 있다.  

### 18.4. MERGE 문
<br/>
MERGE 문을 사용하면 테이블에 신규 행을 삽입하거나, 테이블의 기존 행을 갱신, 삭제할 수 있다.  

#### 18.4.1. 기본 문법
<br/>
MERGE 문의 구문은 아래와 같다. USING 절에 지정한 소스 테이블을 INTO 절에 지정한 타깃 테이블과 ON 절의 조건으로 조인한 후, 조인이 성공하면 MERGE UPDATE 절, 조인이 실패하면 MERGE INSERT 절을 수행한다.  

```
MERGE INTO {table | view | (subquery)} [t_alias]
USING {table | view | (subquery)} [t_alias]
ON (condition)
WHEN MATCHED THEN
     UPDATE
        SET column = {expr | DEFAULT} [, column = {expr | DEFAULT}]...
    [WHERE condition]
    [DELETE
     WHERE condition]
WHEN NOT MATCHED THEN
     INSERT [(column [, column]...)]
     VALUES ({expr | DEFAULT} [, {expr | DEFAULT}]...)
    [WHERE condition];
```

+ INTO 절 : 갱신 또는 삽입할 타깃 테이블
+ USING 절 : 갱신 또는 삽입에 사용할 소스 테이블
+ ON 절 : 갱신 또는 삽입을 결정하는 조건
+ MERGE UPDATE 절 : ON 절의 조건이 만족하는 경우 수행될 구문
+ MERGE INSERT 절 : ON 절의 조건이 만족하지 않는 경우 수행될 구문  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
CREATE TABLE t2 AS SELECT empno, ename, job, sal FROM emp WHERE deptno = 20;
CREATE TABLE t1 AS SELECT * FROM t2 WHERE empno IN (7369, 7566);
CREATE TABLE t3 AS SELECT * FROM t2 WHERE empno = 7369;
CREATE TABLE t4 AS SELECT '2050' AS yyyy, a.* FROM t2 a UNION ALL
                   SELECT '2051' AS yyyy, a.* FROM t2 a;

ALTER TABLE t1 CONSTRAINT t1_pk PRIMARY KEY (empno);
ALTER TABLE t2 CONSTRAINT t2_pk PRIMARY KEY (empno);
ALTER TABLE t3 CONSTRAINT t3_pk PRIMARY KEY (empno);
ALTER TABLE t4 CONSTRAINT t4_pk PRIMARY KEY (yyyy, empno);
```

아래의 MERGE 문을 수행하면 조인에 성공한 행은 갱신되고, 실패한 행은 삽입된다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500
WHEN NOT MATCHED THEN
     INSERT (t.empno, t.ename, t.job)
     VALUES (s.empno, s.ename, s.job);
```

##### 18.4.1.1. 선택 작업
<br/>
10.1 버전부터 MERGE UPDATE 절과 MERGE INSERT 절을 선택적으로 사용할 수 있다.  

아래는 MERGE UPDATE 절만 사용한 쿼리다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500;
```

위 MERGE 문은 아래 UPDATE 문과 결과가 동일하다.  

```sql
UPDATE (SELECT a.sal, b.sal, AS sal_n FROM t1 a, t2 b WHERE b.empno = a.empno)
SET sal = sal_n - 500;
```

아래는 MERGE INSERT 절만 사용한 쿼리다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.empno = s.empno)
WHEN NOT MATCHED THEN
     INSERT (t.empno, t.ename, t.job)
     VALUES (s.empno, s.ename, s.job);
```

위 MERGE 문은 아래 INSERT 문과 결과가 동일하다.  

```sql
INSERT INTO t1 (empno, ename, job, sal)
SELECT empno, ename, job, sal
FROM t2 a
WHERE NOT EXISTS (SELECT 1 FROM t1 x WHERE x.empno = a.empno);
```

##### 18.4.1.2. WHERE 절
<br/>
10.1 버전부터 MERGE UPDATE 절과 MERGE INSERT 절에 WHERE 절을 사용할 수 있다.  

아래는 MERGE UPDATE 절에 WHERE 절을 사용한 쿼리다. WHERE 절에 타깃 테이블(t1)의 일반 조건을 기술했다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500
     WHERE t.job = 'CLERK';
```

아래와 같이 인라인 뷰를 사용해도 동일한 결과를 얻을 수 있다.  

```sql
MERGE INTO (SELECT * FROM t1 WHERE job = 'CLERK') t
USING t2 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500
     WHERE t.job = 'CLERK';
```

아래 쿼리는 MERGE UPDATE 절의 WHERE 절에 소스 테이블(t2)의 일반 조건을 기술했다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500
     WHERE s.job = 'CLERK';
```

아래와 같이 인라인 뷰를 사용해도 동일한 결과를 얻을 수 있다.  

```sql
MERGE INTO t1 t
USING (SELECT * FROM t1 WHERE job = 'CLERK') s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500
     WHERE s.job = 'CLERK';
```

아래 쿼리와 같이 조건에 스소 테이블과 타깃 테이블의 열을 함께 사용할 경우에는 MERGE UPDATE 절의 WHERE 절에 조건을 기술해야 한다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500
     WHERE (    (t.job = 'CLERK' AND s.sal >= 1000)
            OR  (t.job <> 'CLERK'));
```

MERGE INSERT 절의 WHERE 절에는 소스 테이블(t2)의 일반 조건만 기술할 수 있다.  

MERGE INSERT 절을 사용할 경우 타깃 테이블(t1)의 인라인 뷰에 일반 조건을 기술하지 않아야 한다.  

아래 쿼리는 MERGE INSERT 절의 WHERE 절에 소스 테이블(t2)의 일반 조건을 기술했다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     INSERT (t.empno, t.ename, t.job)
     VALUES (s.empno, s.ename, s.job)
     WHERE s.job = 'CLERK';
```

아래와 같이 인라인 뷰를 사용해도 동일한 결과를 얻을 수 있다. MERGE UPDATE 절, MERGE INSERT 절의 WHERE 절보다 인라인 뷰에 일반 조건을 기술하는 편이 성능 측변에서 유리할 수 있다.  

```sql
MERGE INTO t1 t
USING (SELECT * FROM t2 WHERE job = 'CLERK') s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     INSERT (t.empno, t.ename, t.job)
     VALUES (s.empno, s.ename, s.job);
```

##### 18.4.1.3. DELETE 절
<br/>
10.1 버전부터 MERGE UPDATE 절에 DELETE 절을 기술할 수 있다. DELETE 절은 MERGE UPDATE 절로 갱신된 값을 기준으로 행을 삭제한다.  

아래는 MERGE UPDATE 절에 DELETE 절을 사용한 쿼리다. 갱신된 sal가 2000보다 작은 행을 삭제한다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500
     WHERE s.job = 'CLERK'
     DELETE
     WHERE t.sal < 2000;
```

아래는 MERGE 문의 모든 절을 사용한 쿼리다. MERGE UPDATE 절에 의해 t1, t2 테이블에 모두 존재하는 행 중 job이 CLERK인 행의 sal가 갱신되고, 갱신된 sal가 2000보다 작은 행이 삭제된다. MERGE INSERT 절에 의해 t2 테이블에만 존재하는 행 중 job이 CLERK인 행이 삽입된다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500
     WHERE s.job = 'CLERK'
     DELETE
     WHERE t.sal < 2000
WHEN NOT MATCHED THEN
     INSERT (t.empno, t.ename, t.job)
     VALUES (s.empno, s.ename, s.job)
     WHERE s.job = 'CLERK';
```

#### 18.4.2. 고급 주제
<br/>
##### 18.4.2.1. 조인 차수
<br/>
MERGE 문도 UPDATE 문처럼 조인 차수에 따라 에러가 발생할 수 있다.  

아래 쿼리는 에러가 발생한다. t1, t4 테이블의 조인 차수가 1:M이므로 t1 테이블의 값이 t4 테이블의 값으로 여러 번 갱신될 수 있기 때문이다.  

```sql
MERGE INTO t1 t
USING t4 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500;
```

아래 쿼리는 에러가 발생하지 않는다. ROW&#95;NUMBER 함수를 사용하여 조인 차수를 1:1로 변경했다.  

```sql
MERGE INTO t1 t
USING (SELECT *
       FROM (SELECT a.*
                  , ROW_NUMBER() OVER(PARTITION BY a.empno ORDER BY a.yyyy DESC) AS rn
             FROM t4 a)
       WHERE rn = 1) s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500;
```

##### 18.4.2.2. 조인 조건
<br/>
ON 절에 기술된 열은 갱신할 수 없다. 무한 루프가 발생할 수 있기 때문이다.  

갱신할 테이블이 동일 테이블이면 ROWID 슈도 칼럼을 사용할 수 있다.  

```sql
MERGE INTO t1 t
USING (SELECT empno + ROW_NUMBER() OVER(ORDER BY empno) AS empno_n FROM t1) s
ON (t.ROWID = s.ROWID)
WHEN MATCHED THEN
     UPDATE
     SET t.empno = s.empno_n;
```

다른 테이블을 기준으로 ON 절에 기술된 열을 갱신하려면 USING 절에서 갱신할 테이블을 조인해야 한다. 아래 쿼리는 인라인 뷰(t3)와 t1 테이블을 조인했다.  

```sql
MERGE INTO t1 t
USING (SELECT s.emono_n, b.ROWID AS rid
       FROM (SELECT empno + ROW_NUMBER() OVER(ORDER BY empno) AS empno_n
             FROM t3) a
          , t1 b
       WHERE b.empno = a.empno) s
ON (t.ROWID = s.rid)
WHEN MATCHED THEN
     UPDATE
     SET t.empno = s.empno_n;
```

##### 18.4.2.3. 일반 조건
<br/>
MERGE 문의 ON 절은 SELECT 문의 WHERE 절과 유사하게 동작한다. 조인 조건과 일반 조건을 기술할 수 있지만 조인 조건만 사용하는 편이 성능 측면에서 효율적이다.  

아래 쿼리는 ON 절에 타깃 테이블(t1)의 일반 조건(t.job = 'CLERK')을 기술했다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.job = 'CLERK' AND t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500;
```

아래 쿼리는 위 쿼리와 동일하게 동작한다. 아래 쿼리가 가독성 측면에서 바람직하다.  

```sql
MERGE INTO (SELECT * FROM t1 WHERE job = 'CLERK') t
USING t2 s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500;
```

아래 쿼리는 ON 절에 소스 테이블(t2)의 일반 조건(a.job = 'CLERK')을 기술했다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.empno = s.empno AND s.job = 'CLERK')
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500;
```

아래와 같이 인라인 뷰를 사용하는 편이 바람직하다.  

```sql
MERGE INTO t1 t
USING (SELECT * FROM t2 WHERE job = 'CLERK') s
ON (t.empno = s.empno)
WHEN MATCHED THEN
     UPDATE
     SET t.sal = s.sal - 500;
```

MERGE INSERT 문을 사용할 경우 ON 절에 타깃 테이블(t1)의 일반 조건을 기술하면 에러가 발생할 수 있다. 아래 쿼리는 에러가 에러가 발생한다. ON 절이 FALSE인 행이 삽입되므로 t.job <> 'CLERK' OR <> t.empno = s.empno 조건이 TRUE인 행이 삽입될 수 있다. job이 CLERK이 아니고 empno가 동일한 행이 TRUE로 평가되므로 무결성 제약 조건 에러가 발생한 것이다.  

```sql
MERGE INTO t1 t
USING t2 s
ON (t.job = 'CLERK' AND t.empno = s.empno)
WHEN MATCHED THEN
     INSERT (t.empno, t.ename, t.job)
     VALUES (s.empno, s.ename, s.job);
```

ON 절에 소스 테이블(t2)의 일반 조건을 기술해도 위 쿼리와 동일한 이유로 에러가 발생할 수 있다.  

##### 18.4.2.4. 아우터 조인
<br/>
ON 절의 조인 조건에 (+) 기호를 기술하면 소스 테이블과 타깃 테이블이 아우터 조인으로 조인된다. 타깃 테이블의 전체 행을 갱신할 때 사용할 수 있다.  

아래 쿼리는 ON 절의 조인 조건에 (+) 기호를 기술했다. 타깃 테이블(t1)을 아우터 기준으로 소스 테이블(t3)을 아우터 조인한다.  

```sql
MERGE INTO t1 t
USING t3 s
ON (t.empno = s.empno(+))
WHEN MATCHED THEN
     UPDATE
     SET t.sal = NVL(s.sal - 500, 0);
```

아래 쿼리는 위 쿼리와 결과가 동일하다. 상관 서브 쿼리를 사용했기 때문에 쿼리 성능이 저하될 수 있다.  

```sql
UPDATE t1 a
SET sal = NVL((SELECT x.sal FROM t3 x WHERE x.empno = a.empno) - 500, 0);
```

아래와 같이 인라인 뷰를 사용하면 MERGE 문과 동일한 성능을 보장할 수 있다.  

```sql
UPDATE (SELECT a.sal, b.sal AS sal_n FROM t1 a, t3 b WHERE b.empno(+) = a.empno)
SET sal = NVL(sal_n - 500, 0);
```

### 18.5. DML 에러 로깅
<br/>
DML 문에서 에러가 발생하면 해당 DML 문에 의한 변경 사항이 모두 롤백된다. 99행까지 변경한 후 마지막 1행에서 에러가 발생하면 99행이 모두 롤백되는 것이다. 이런 상황을 피하기 위해 DML 에러 로깅(DML error logging) 기능을 사용할 수 있다. DML 에러 로깅 기능은 10.2 버전부터 사용할 수 있다.  

DML 에러 로깅 구문은 아래와 같다. DML 에러 로깅 기능은 DML 수행 시 에러가 발생하면 에러를 로그 테이블에 기록한 후, 다음 행에 대한 DML을 계속 진행한다.  

```
LOG ERRORS [INFO [schema.] table] [(simple_expression)]
[REJECT LIMIT {integer | UNLIMITED}]
```

+ INFO : 에러 로깅 테이블을 지정
+ simple&#95;expression : 에러 태그로 사용할 값을 지정
+ REJECT LIMIT : integer로 에러와 한계 값을 지정 (기본값은 0 또는 UNLIMITED)  

예제를 위해 아래와 같이 테이블을 생성하자. t1 테이블의 PK가 empno이므로 중복된 empno를 삽입하면 에러가 발생한다.  

```sql
CREATE TABLE t1 (empno NUMBER(4), ename VARCHAR2(10), sal NUMBER(7, 2)
               , CONSTRAINT t1_pk PRIMARY KEY (empno));
```

DBMS&#95;ERRLOG.CREATE&#95;ERROR&#95;LOG 프로시저로 로그 테이블을 생성할 수 있다. 아래 코드는 e1 테이블을 t1 테이블의 로그 테이블로 생성한다.  

```sql
BEGIN
  DBMS_ERRLOG.CREATE_ERROR_LOG (dml_table_name => 'T1'
                              , err_log_table_name => 'E1');
END;
/
```

아래 쿼리는 모두 에러가 발생한다. 첫 번째 쿼리는 5자리 정수부를 저장할 수 있는 sal에 100000을 삽입했기 때문에 에러가 발행했고, 두 번째 쿼리는 t1 테이블에 존재하고 있는 empno(7782)를 삽입했기 때문에 에러가 발생했다. 세 번째 쿼리도 두 번째 쿼리와 동일한 원인으로 에러가 발생했다.  

```sql
INSERT INTO t1 VALUES (7839, 'KING', 100000);

INSERT INTO t1 VALUES (7782, 'CLERK', 2450);

INSERT INTO t1 SELECT empno, ename, sal FROM emp WHERE deptno = 10;
```

아래와 같이 에러 로깅 절을 기술하면 에러가 발생하지 않는다. 세 번째 쿼리는 3행 중 empno가 중복된 1행은 무시되고, 중복되지 않은 2행은 정삭적으로 삽입된다.  

```sql
INSERT INTO t1 VALUES (7839, 'KING', 100000)
LOG ERRORS INTO e1('1') REJECT LIMIT UNLIMITED;

INSERT INTO t1 VALUES (7782, 'CLARK', 2450)
LOG ERRORS INTO e1('2') REJECT LIMIT UNLIMITED;

INSERT INTO t1 SELECT empno, ename, sal FROM emp WHERE deptno = 10
LOG ERRORS INTO e1('3') REJECT LIMIT UNLIMITED;
```

e1 테이블에서 에러에 대한 정보를 확인할 수 있다.  

```sql
SELECT ora_err_mesg$, ora_err_optyp$, ora_err_tag$, empno, sal FROM e1;
```

로그 테이블은 아래와 같이 직접 삭제해야 한다.  

```sql
DROP TABLE e1 PURGE;
```

#### 18.5.1. IGNORE&#95;ROW&#95;ON&#95;DUPKEY&#95;INDEX 힌트
<br/>
IGNORE&#95;ROW&#95;ON&#95;DUPKEY&#95;INDEX 힌트를 사용하면 PK 제약 조건이나 UNIQUE 제약 조건에 위배되는 행을 무시할 수 있다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
CREATE TABLE t1 (c1 NUMBER, CONSTRAINT t1_u1 UNIQUE (c1));

INSERT INTO t1 VALUES (1);
COMMIT;
```

아래 쿼리는 t1 테이블의 c1 열에 중복 값인 1인 행을 삽입했기 때문에 에러가 발생한다.  

```sql
INSERT INTO t1 SELECT LEVEL FROM DUAL CONNECT BY LEVEL <= 2;
```

아래와 같이 IGNORE&#95;ROW&#95;ON&#95;DUPKEY&#95;INDEX 힌트를 사용하면 에러가 발생하지 않는다. DML 에러 로깅과 유사하게 동작한다.  

```sql
INSERT /*+ IGNORE_ROW_ON_DUPKEY_INDEX(T1 T1_U1) */
INTO
SELECT LEVEL FROM DUAL CONNECT BY LEVEL <= 2;
```
