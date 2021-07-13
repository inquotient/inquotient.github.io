---
title: 사용자 정의 타입
categories:
- Unkind_SQL
feature_text: |
  ## 28. 사용자 정의 타입
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

사용자 정의 타입은 오라클 데이터베이스가 제공하는 객체 지향 기능이다. 사용자 정의 타입은 주로 PL/SQL 개발에 사용하지만 일부 기능은 SQL 개발에 활용할 수 있다. 오라클 데이터베이스는 오브젝트 타입, REF 타입, VARRAY, 중첩 테이블 등의 사용자 정의 타입을 제공한다.  

### 28.1. 기본 문법
<br/>
사용자 정의 타입은 CREATE TYPE 문으로 생성한다.  

```
CREATE [OR REPLACE] TYPE plsql_type_source;
```

CREATE TYPE BODY 문은 사용자 정의 타입의 메서드(method)를 생성한다. 메소드를 포함한 사용자 정의 타입은 PL/SQL로 구현된다.  

```
CREATE [OR REPLACE] TYPE BODY plsql_type_body_source;
```

#### 28.1.1. 오브젝트 타입
<br/>
오브젝트 타입(object type)는 구조체(structure)와 유사한 사용자 정의 타입이다. 하나 이상의 속성(attribute)으로 구성되며, 내장 데이터 타입(built-in data type)과 사용자 정의 타입을 속성으로 사용할 수 있다.  

아래 쿼리는 2개의 NUMBER 타입 속성으로 구성된 오브젝트 타입을 생성한다.  

```sql
CREATE OR REPLACE TYPE trc1 AS OBJECT (c1 NUMBER, c2 NUMBER);
/
```

아래는 trc1 타입을 사용한 쿼리다. 오브젝트 타입의 속성은 {table}.{column}.{attribute} 형식으로 조회할 수 있다.  

```sql
WITH w1 AS (SELECT trc1 (1, 2) AS r1 FROM DUAL)
SELECT r1, a.r1 AS r1_c1, a.r1.c2 AS r1_c2 FROM w1 a;
```

아래와 같이 데이터 저장된다.  

<table>
  <thead>
    <tr>
      <td>TRC1.C1</td>
      <td>TRC1.C2</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 28.1.2. 중첩 테이블
<br/>
중첩 테이블(nested table)은 배열(array)과 유사한 사용자 정의 타입이다. 정렬되지 않은 요소(element)로 구성되며, 내장 데이터 타입과 사용자 정의 타입을 요소로 사용할 수 있다.  

아래 쿼리는 NUMBER 타입으로 구성된 중첩 테이블(tnt1)을 생성한다. 하나의 내장 데이터 타입으로 구성된 중첩 테이블을 스칼라 중첩 테이블(scalar nested table)이라고 한다.  

```sql
CREATE OR REPLACE TYPE tnt1 IS TABLE OF NUMBER;
/
```

아래 쿼리는 tnt1 타입을 사용한 쿼리다.  

```sql
SELECT tnt1 (1, 2, 3) AS c1 FROM DUAL;
```

아래와 같이 데이터가 저장된다.  

+ TNT1(1) : 1
+ TNT1(2) : 2
+ TNT1(3) : 3  

아래 쿼리는 오브젝트 타입(trc1)을 요소로 가지는 중첩 테이블(tnt2)을 생성한다.  

```sql
CREATE OR REPLACE TYPE tnt2 IS TABLE OF trc1;
```

아래 쿼리는 tnt2 타입을 사용한 쿼리다.  

```sql
SELECT tnt2 (trc1 (1, 2), trc1 (3, 4), trc1 (5, 6)) AS c1 FROM DUAL;
```

아래와 같이 데이터가 저장된다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>TRC1.C1</td>
      <td>TRC1.C2</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>TNT2(1)</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <td>TNT2(2)</td>
      <td>3</td>
      <td>4</td>
    </tr>
    <tr>
      <td>TNT2(3)</td>
      <td>5</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 쿼리는 중첩 테이블(tnt1)을 요소로 가지는 중첩 테이블(tnt3)을 생성한다.  

```sql
CREATE OR REPLACE TYPE tnt3 IS TABLE OF tnt1;
/
```

아래 쿼리는 tnt3 타입을 사용한 쿼리다.  

```sql
SELECT tnt3 (tnt1 (1, 2, 3), tnt1 (4, 5, 6)) AS c1 FROM DUAL;
```

아래와 같이 데이터가 저장된다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>TNT1(1)</td>
      <td>TNT1(2)</td>
      <td>TNT1(3)</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>TNT3(1)</td>
      <td>1</td>
      <td>2</td>
      <td>3</td>
    </tr>
    <tr>
      <td>TNT3(2)</td>
      <td>4</td>
      <td>5</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 쿼리는 오브젝트 타입으로 구성된 중첩 테이블(tnt2)을 요소로 가지는 중첩 테이블(tnt4)을 생성한다.  

```sql
CREATE OR REPLACE TYPE tnt4 IS TABLE OF tnt2;
/
```

아래 쿼리는 tnt4 타입을 사용한 쿼리다.  

```sql
SELECT tnt4 (tnt2 (trc1 (1, 2), trc1 (3, 4))
           , tnt2 (trc1 (5, 6), trc1 (7, 8))) AS c1
FROM DUAL;
```

아래와 같이 데이터가 저장된다.  

<table>
  <thead>
    <tr>
      <td rowspan="2"></td>
      <td colspan="2">TNT2(1)</td>
      <td colspan="2">TNT2(2)</td>
    </tr>
    <tr>
      <td>TRC1.C1</td>
      <td>TRC1.C2</td>
      <td>TRC1.C1</td>
      <td>TRC1.C2</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>TNT4(1)</td>
      <td>1</td>
      <td>2</td>
      <td>3</td>
      <td>4</td>
    </tr>
    <tr>
      <td>TNT4(2)</td>
      <td>5</td>
      <td>6</td>
      <td>7</td>
      <td>8</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 쿼리는 중첩 테이블(tnt1)을 속성으로 가지는 오브젝트 타입(trc2)을 생성한다.  

```sql
CREATE OR REPLACE TYPE trc2 AS OBJECT (c1 NUMBER, c2 tnt1);
/
```

아래 쿼리는 trc2 타입을 사용한 쿼리다.  

```sql
SELECT trc2 (1, tnt1 (2, 3, 4)) AS c1 FROM DUAL;
```

아래와 같이 데이터가 저장된다.  

<table>
  <thead>
    <tr>
      <td>TRC2.C1</td>
      <td rowspan="2">TRC2.C2</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td colspan="3">1</td>
      <td>TNT1(1)</td>
      <td>2</td>
    </tr>
    <tr>
      <td>TNT1(2)</td>
      <td>3</td>
    </tr>
    <tr>
      <td>TNT1(3)</td>
      <td>4</td>
    </tr>
  </tbdoy>
</table>
<br/><br/>

&#42;&#95;TYPES 뷰에서 사용자 정의 타입에 대한 정보를 조회할 수 있다.  

```sql
SELECT type_name, typecode, attributes FROM user_types;
```

&#42;&#95;TYPE&#95;ATTRS 뷰에서 사용자 정의 타입의 속성에 대한 정보를 조회할 수 있다.  

```sql
SELECT type_name, attr_name, attr_type_owner, attr_type_name
FROM user_type_attrs
ORDER BY type_name, attr_no;
```

일반적이지 않지만 중첩 테이블을 사용하는 오브젝트 테이블(object table)을 생성할 수도 있다. 오브젝트 테이블은 물리 저장 구조로 사용하지 않는 것이 일반적이다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 tnt1) NESTED TABLE t2 STORE AS t1_c2;
```

&#42;&#95;TABLES 뷰에서 t1 테이블과 t1&#95;c2 중첩 테이블이 생성된 것을 확인할 수 있다. 중첩 테이블은 nested 열이 YES로 표시된다.  

```sql
SELECT table_name, nested FROM user_tables WHERE table_name IN ('T1', 'T1_C2');
```

&#42;&#95;TAB&#95;COLUMNS 뷰에서 사용자 정의 타입(tnt1)을 확인할 수 있다.  

```sql
SELECT column_name, data_type, data_type_owner
FROM user_tab_columns
WHERE table_name = 'T1';
```

오브젝트 테이블은 암시적으로 인덱스를 생성한다.  

```sql
SELECT table_name, index_name, index_type
FROM user_indexes
WHERE table_name IN ('T1', 'T1_C2');
```

예제를 위해 아래와 같이 데이터를 입력하자.  

```sql
INSERT INTO t1 VALUES (1, tnt1 (1, 2, 2));
INSERT INTO t1 VALUES (2, tnt1 (1, 2, 3));
INSERT INTO t1 VALUES (3, tnt1 ());
INSERT INTO t1 VALUES (4, NULL);
COMMIT;
```

### 28.2. MULTISET 조건
<br/>
MULTISET 조건(multiset condition)을 사용하면 중첩 테이블을 검색할 수 있다.  

#### 28.2.1. IS A SET 조건
<br/>
IS A SET 조건은 nested&#95;table의 요소가 고유하면 TRUE, 고유하지 않으면 FALSE를 반환한다.  

```
nested_table IS [NOT] A SET
```

아래는 IS A SET 조건을 사용한 쿼리다.  

```sql
SELECT * FROM t1 WHERE c2 IS A SET;
```

#### 28.2.2. IS EMPTY 조건
<br/>
IS EMPTY 조건은 nested&#95;table의 요소가 비어있으면 TRUE, 비어있지 않으면 FALSE를 반환한다.  

```
nested_table IS [NOT] EMPTY
```

아래는 IS EMPTY 조건을 사용한 쿼리다.  

```sql
SELECT * FROM t1 WHERE c2 IS EMPTY;
```

IS A SET 조건과 IS NOT EMPTY 조건을 함께 사용하면 중첩 테이블을 요소가 고유하고 비어있지 않은 행을 검색할 수 있다.  

```sql
SELECT * FROM t1 WHERE c2 IS A SET AND c2 IS NOT EMPTY;
```

#### 28.2.3. MEMBER 조건
<br/>
MEMBER 조건은 expr이 nested&#95;table의 요소에 속하면 TRUE, 속하지 않으면 FALSE를 반환한다.  

```
expr [NOT] MEMBER [OF] nested_table
```

아래는 MEMBER 조건을 사용한 쿼리다.  

```sql
SELECT * FROM t1 WHERE 2 MEMBER OF c2;
```

#### 28.2.4. SUBMULTISET 조건
<br/>
SUBMULTISET 조건은 nested&#95;table이 nested&#95;table2의 부분 집합이면 TRUE, 부분 집합이 아니면 FALSE를 반환한다.  

```
nested_table [NOT] SUBMULTISET [OF] nested_table2
```

아래는 SUBMULTISET 조건을 사용한 쿼리다. (1, 3)는 (1, 2, 3)의 부분 집합이다.  

```sql
SELECT * FROM t1 WHERE tnt1 (1, 3) SUBMULTISET OF c2;
```

### 28.3. 컬렉션 함수
<br/>
컬렉션 함수(collection function)를 사용하면 중첩 테이블을 조작할 수 있다.  

#### 28.3.1. SET 함수
<br/>
SET 함수는 nested&#95;table의 중복 요소가 제거된 결과를 반환한다.  

```
SET (nested_table)
```

아래는 SET 함수를 사용한 쿼리다.  

```sql
SELECT SET (tnt1 (1, 2, 3)) AS c1
     , SET (tnt2 (trc1 (1, 1), trc1(2, 2), trc1 (2, 2))) AS c2
FROM DUAL;
```

#### 28.3.2. CARDINALITY 함수
<br/>
CARDINALITY 함수는 nested&#95;table 요소의 개수를 반환한다.  

```
CARDINALITY (nested_table)
```

아래는 CARDINALITY 함수를 사용한 쿼리다.  

```sql
SELECT CARDINALITY (tnt1 (1, 2, 3)) AS c1
     , CARDINALITY (tnt2 (trc1 (1, 1), trc1 (2, 2), trc1 (2, 2))) AS c2
FROM DUAL;
```

#### 28.3.3. POWERMULTISET 함수
<br/>
POWERMULTISET 함수는 중첩 테이블(expr)의 모든 부분 집합을 반환한다. 중첩 테이블의 중첩 테이블이 반환된다.  

```
POWERMULTISET (expr)
```

아래 쿼리는 에러가 발생한다.  

```sql
SELECT POWERMULTISET (tnt1 (1, 2, 3)) AS c1 FROM DUAL;

ORA-22833: 일시적 유형을 지속적 유형으로 데이터형 변환해야 함
```

POWERMULTISET 함수는 CAST 함수와 함께 사용해야 한다.  

```sql
SELECT CAST (POWERMULTISET (tnt1 (1, 2, 3)) AS tnt3) AS c1 FROM DUAL;
```

#### 28.3.4. POWERMULTISET&#95;BY&#95;CARDINALITY 함수
<br/>
POWERMULTISET&#95;BY&#95;CARDINALITY 함수도 중첩 테이블(expr)의 모든 부분 집합을 반환한다. 부분 집합의 차수(cardinality)를 지정할 수 있다.  

```
POWERMULTISET_BY_CARDINALITY (expr, cardinality)
```

아래는 POWERMULTISET&#95;BY&#95;CARDINALITY 함수를 사용한 쿼리다. POWERMULTISET&#95;BY&#95;CARDINALITY 함수도 CAST 함수와 함께 사용해야 한다.  

```sql
SELECT CAST (POWERMULTISET_BY_CARDINALITY (tnt1 (1, 2, 3), 2) AS tnt3) AS c1 FROM DUAL;
```

#### 28.3.5. COLLECT 함수
<br/>
COLLECT 함수는 column의 집계 결과를 중첩 테이블로 반환한다.  

```
COLLECT ([DISTINCT | UNIQUE] column [ORDER BY expr])
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t2 PURGE;
CREATE TABLE t2 (c1 NUMBER);

INSERT INTO t2 VALUES (2);
INSERT INTO t2 VALUES (2);
INSERT INTO t2 VALUES (1);
COMMIT;
```

아래는 COLLECTION 함수를 사용한 쿼리다. CAST 함수를 함께 사용하지 않으면 데이터 타입이 암시적으로 생성된다.  

```sql
SELECT COLLECT (c1) AS c1 FROM t2;
```

&#42;&#95;TYPES 뷰에서 암시적으로 생성된 데이터 타입을 확인할 수 있다. 암시적으로 생성된 데이터 타입은 일정 시점이 지나면 자동으로 삭제된다.  

```sql
SELECT type_name, typecode, instantiable
FROM user_types
WHERE type_name = 'ST00001lDX3R79QMqeFNW/B5BITQ=';
```

COLLECTION 함수도 CAST 함수와 함께 사용해야 한다.  

```sql
SELECT CAST (COLLECT (c1) AS tnt1) AS c1 FROM t2;
```

DISTINCT 키워드를 사용하면 중복 요소를 제거할 수 있다.  

```sql
SELECT CAST (COLLECT (DISTINCT c1) AS tnt1) AS c1 FROM t2;
```

ORDER BY 절을 사용하면 요소를 정렬할 수 있다.  

```sql
SELECT CAST (COLLECT (c1 ORDER BY c1) AS tnt1) AS c1 FROM t2;
```

DISTINCT 키워드와 ORDER BY 절을 함께 DISTINCT 키워드가 동작하지 않는다.  

```sql
SELECT CAST (COLLECT (DISTINCT c1 ORDER BY c1) AS tnt1) AS c1 FROM t2;
```

#### 28.3.6. CAST 함수
<br/>
CAST 함수를 MULTISET 키워드와 함께 사용하면 스칼라 서브 쿼리를 중첩 테이블로 변환할 수 있다.  

```
CAST (MULTISET (subquery) AS type_name)
```

아래는 CAST 함수를 사용한 쿼리다.  

```sql
SELECT CAST (MULTISET (SELECT c1 FROM t2) AS tnt1) AS c1 FROM DUAL;
```

아래 쿼리는 서브 쿼리에서 중복 값을 제거하고 행을 정렬한 후 중첩 테이블을 생성한다.  

```sql
SELECT CAST (MULTISET (SELECT DISTINCT c1 FROM t2 ORDER BY c1) AS tnt1) AS c1 FROM DUAL;
```

### 28.4. TABLE 컬렉션 표현식
<br/>
TABLE 컬렉션 표현식(table collection expression)을 사용하면 중첩 테이블을 테이블 형태로 조회할 수 있다. (+) 기호를 기술하면 아우터 조인으로 조인된다.  

```
TABLE (collection_expression) [(+)]
```

아래는 TABLE 컬렉션 표현식을 사용한 쿼리다. COLUMN&#95;VALUE 열은 스칼라 중첩 테이블을 TABLE 컬렉션 표현식의 인수로 입력했을 때 반환되는 슈도 칼럼이다.  

```sql
SELECT * FROM TABLE (tnt1 (1, 2, 3));
```

아래 쿼리는 오브젝트 타입으로 구성된 중첩 테이블을 조회한다.  

```sql
WITH w1 AS (SELECT tnt2 (trc1 (1, 2), trc1 (3, 4), trc1 (5, 6)) AS c1 FROM DUAL)
SELECT b.* FROM w1 a, TABLE (a.c1) b;
```

아래 쿼리는 오브젝트 타입으로 구성된 중첩 테이블을 요소로 가지는 중첩 테이블을 조회한다. TABLE 컬렉션 표현식을 2번 사용했다.  

```sql
WITH w1 AS (SELECT tnt3 (tnt2 (trc1 (1, 2), trc1 (3, 4))
                       , tnt2 (trc1 (5, 6), trc1 (7, 8))) AS c1
            FROM DUAL)
   , w2 AS (SELECT b.COLUMN_VALUE AS c1 FROM w1 a, TABLE (a.c1) b)
SELECT b.* FROM w2 a, TABLE (a.c1) b;
```

아래 쿼리는 t1 테이블의 c2 열과 TABLE 컬렉션 표현식을 조인한다. c2 열은 중첩 테이블이다.  

```sql
SELECT a.c1, b.column_value AS c2 FROM t1 a, TABLE (a.c2) b;
```

아래 쿼리는 t1 테이블의 c2 열과 TABLE 컬렉션 표현식을 아우터 조인한다.  

```sql
SELECT a.c1, b.column_value AS c2 FROM t1 a, TABLE (a.c2) (+) b;
```

### 28.5. 활용 예제
<br/>
사용자 정의 타입의 활용 예제를 살펴보자. 사용자 정의 타입을 사용하면 SQL의 제약을 극복할 수 있다.  

예제를 위해 아래와 같이 오브젝트 타입(object type)을 생성하자.  

```sql
CREATE OR REPLACE TYPE trc_emp1 AS OBJECT (sal NUMBER (7, 2), comm NUMBER (7, 2));
```

아래 쿼리는 오브젝트 타입을 통해 스칼라 서브 쿼리에서 다중 열을 조회한다.  

```sql
SELECT a.deptno, a.dname, a.emp_rc.sal AS sal, a.emp_rc.comm AS comm
FROM (SELECT a.*
           , (SELECT trc_emp1 (SUM(x.sal), SUM(x.comm))
              FROM emp x
              WHERE x.deptno = a.deptno) AS emp_rc
      FROM dept a) a
ORDER BY a.deptno;
```

다웆 ㅇ행을 조회하기 위해서는 중첩 테이블을 사용해야 한다. 예제를 위해 아래와 같이 오브젝트 타입(trc_emp2)과 중첩 테이블(tnt_emp2)을 생성하자.  

```sql
CREATE OR REPLACE TYPE trc_emp2 AS OBJECT (job VARCHAR2(9), sal NUMBER(7, 2), comm NUMBER(7, 2));
/

CREATE OR REPLACE TYPE tnt_emp2 IS TABLE OF trc_emp2;
```

아래 쿼리는 중첩 테이블을 통해 스칼라 서브 쿼리에서 다중 열, 다중 행을 조회한다.  

```sql
SELECT a.deptno, a.dname, b.job, b.sal, b.comm
FROM (SELECT a.*
           , CASE (MULTISET (SELECT x.job, SUM(x.sal), SUM(x.comm)
                             FROM emp x
                             WHERE x.deptno = a.deptno
                             GROUP BY x.job) AS tnt_emp2) AS emp_nt
      FROM dept a) a
         , TABLE (a.emp_nt) b
ORDER BY a.deptno, b.job;
```
