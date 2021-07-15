---
title:  JSON 개발
categories:
- Unkind_SQL
feature_text: |
  ## 30. JSON 개발
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

JSON(JavaScript Object Notation)은 오브젝트, 배열, 스칼라 데이터 등을 표현할 수 있는 텍스트 기반의 개방형 데이터 교환 포맷이다. 오라클 데이터베이스는 12.1 버전부터 JSON 개발 기능을 제공한다. 18.1 버전에서 JSON 개발 관련 기능이 대폭 개선되었다.  

### 30.1. 기본 문법
<br/>
#### 30.1.1. JSON 열
<br/>
JSON 값은 VARCHAR2, CLOB, BLOB 타입에 저장할 수 있다. JSON 값을 저장할 열에 IS JSON 조건으로 CHECK 제약 조건을 생성하면 well-formed JSON 값이 저장되는 것을 보장할 수 있다.  

예제를 위해 아래와 가티 테이블을 생성하자. c2 열에 CHECK 제약 조건을 생성했다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 CLOB, CONSTRAINT t1_c1 CHECK (c2 IS JSON (STRICT)));
```

&#42;&#95;JSON&#95;COLUMNS 뷰에서 JSON 열에 대한 정보를 조회할 수 있다.  

```sql
SELECT table_name, column_name, format, data_type
FROM user_json_columns
WHERE table_name = 'T1';
```

예제를 위해 아래와 같이 데이터를 입력하자.  

```sql
INSERT INTO t1
SELECT 1 AS c1
     , JSON_OBJECT (KEY 'dept' VALUE JSON_ARRAYAGG (JSON_OBJECT (KEY 'deptno' VALUE dpetno
                                                               , KEY 'dname' VALUE dname
                                                               , KEY 'loc' VALUE loc))) AS c2
FROM dept;
```

아래는 t1 테이블을 조회한 결과다. c2 열에 JSON 값이 저장된 것을 확인할 수 있다.  

```sql
SELECT * FROM t1;
```

c2 열에 JSON 형식이 아닌 값을 입력하면 에러가 발생한다.  

```sql
INSERT INTO t1 VALUES (2, 'X');

ORA-02290: 체크 제약조건(SCPTT.T1_C1)이 위배되었습니다.
```

#### 30.1.2. 점 표기법
<br/>
점 표기법(dot notation)으로 JSON 값을 조회할 수 있다. VARCHAR2 값이 반환된다.  

아래는 점 표기법을 사용한 쿼리다. c1 열은 deptno 오브젝트를 조회한다. c2, c3 열은 deptno 값을 조회한다.  

```sql
SELECT a.c2.dept AS c1
     , a.c2.dept.deptno AS c2
     , a.c2.dept[*].deptno AS c3
FROM t1 a;
```

아래 쿼리의 c1 열은 첫 번째 배열의 deptno 값을 조회할 수 있다. c2 열은 첫 번째와 두 번째 배열, c3 열은 첫 번째부터 세 번째까지 배열, c4 열은 첫 번째 배열과 두 번째부터 네 번째까지 배열의 deptno 값을 조회한다.  

```sql
SELECT a.c2.dept[0].deptno AS c1
     , a.c2.dept[0,1].deptno AS c2
     , a.c2.dept[0 to 2].deptno AS c3
     , a.c2.dept[0,1 to 3].deptno AS c4
FROM t1 a;
```

#### 30.1.3. SQL/JSON Path 표현식
<br/>
SQL/JSON Path 표현식으로도 JSON 값을 조회할 수 있다. XPath 표현식과 유사하게 동작한다.  

<table>
  <thead>
    <tr>
      <td>표현식</td>
      <td>설명</td>
      <td>XPath</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>$</td>
      <td>루트 오브젝트를 선택</td>
      <td>/</td>
    </tr>
    <tr>
      <td>.</td>
      <td>자식 오브젝트를 선택</td>
      <td>/</td>
    </tr>
    <tr>
      <td>..</td>
      <td>현재 오브젝트의 모든 자식 오브젝트를 조회</td>
      <td>//</td>
    </tr>
    <tr>
      <td>@</td>
      <td>현재 오브젝트를 선택</td>
      <td>.</td>
    </tr>
    <tr>
      <td>&#42;</td>
      <td>모든 오브젝트와 일치</td>
      <td>&#42;</td>
    </tr>
    <tr>
      <td>$[,]</td>
      <td>복수 오브젝트를 선택</td>
      <td>|</td>
    </tr>
    <tr>
      <td>[ ]</td>
      <td>배열 연산자를 기술</td>
      <td>[ ]</td>
    </tr>
    <tr>
      <td>( )</td>
      <td>스크립트 표현식을 기술</td>
      <td></td>
    </tr>
  </tbdoy>
</table>
<br/><br/>

#### 30.1.4. SQL/JSON 조건
<br/>
SQL/JSON 조건은 JSON 값을 검사한다.  

##### 30.1.4.1. IS JSON 조건
<br/>
IS JSON 조건은 expr이 well-formed JSON 값이면 TRUE, 아니면 FALSE를 반환한다.  

```
expr IS [NOT] JSON [FORMAT JSON] [STRICT | LAX] [{WITH | WITHOUT} UNIQUE KEYS]
```

+ expr : 평가할 JSON 값
+ FORMAT JSON : expr이 BLOB 타입인 경우 기술
+ STRICT : 엄격한(strict) JSON 구문을 적용
+ LAX : 느슨한(lax) JSON 구문을 적용 (기본값)
+ WITH UNIQUE KEYS : 각 오브젝트의 키가 고유해야 함
+ WITHOUT UNIQUE KEYS : 각 오브젝트의 키가 고유하지 않아도 됨 (기본값)  

아래는 JSON 조건을 사용한 쿼리다.  

```sql
SELECT c1 FROM t1 WHERE c2 IS JSON;
```

##### 30.1.4.2. JSON&#95;EXISTS 조건
<br/>
JSON&#95;EXISTS 조건은 expr에 JSON&#95;basic&#95;path&#95;expression에 해당하는 JSON 값이 존재하면 TRUE, 존재하지 않으면 FALSE를 반환한다.  

```
JSON_EXISTS (expr [FORMAT JSON], JSON_basic_path_expression
             [PASSING expr AS identifier [, expr AS identifier]...]
             [{ERROR | TRUE | FALSE} ON ERROR])
```

+ PASSING : Path 표현식이 값을 전달
+ ON ERROR : expr이 well-formed JSON 값이 아니면 지정한 결과를 반환  

아래는 JSON&#95;EXISTS 조건을 사용한 쿼리다.  

```sql
SELECT c1 FROM t1 WHERE JSON_EXISTS (c2, '$.dept.deptno');
```

아래 쿼리는 결과가 반환되지 않는다. JSON 배열의 다섯 번째 값이 존재하지 않기 때문이다. JSON 배열은 0부터 시작한다.  

```sql
SELECT c1 FROM t1 WHERE JSON_EXISTS (c2, '$.dept[4]');
```

아래는 선택적 필터 표현식(optional filter expression)을 사용한 쿼리다. 선택적 표현식은 ?(...) 형식으로 기술한다. deptno가 50인 값이 존재하지 않기 때문에 결과가 반환되지 않는다.  

```sql
SELECT c1 FROM t1 WHERE JSON_EXISTS (c2, '$dept?(@.deptno == 50)');
```

아래 쿼리는 dname이 X로 시작하는 JSON 값을 조회한다.  

```sql
SELECT c1
FROM t1
WHERE JSON_EXISTS (c2, '$.dept?(@.dname starts with $p)' PASSING 'X' AS "p");
```

아래 쿼리는 deptno가 10이고 dname이 ACCOUNTING인 JSON 값을 조회한다.  

```sql
SELECT c1
FROM t1
WHERE JSON_EXISTS (c2, '$.dept?(@.deptno == 10 && @.deptno == "ACCOUNTING")');
```

아래 쿼리는 deptno가 50이거나 dname이 ACCOUNTING인 JSON 값을 조회한다.  

```sql
SELECT c1
FROM t1
WHERE JSON_EXISTS (c2, '$.dept?(@.deptno == 50 || @.dname == "ACCOUNTING")');
```

#### 30.1.5. JSON 함수
<br/>
JSON 함수를 사용하면 JSON 데이터를 생성하거나 조회할 수 있다. 대부분의 JSON 함수는 표준 SQL/JSON 함수다. SQL/JSON 함수는 용도에 따라 생성 함수와 조회 함수로 구분할 수 있다.  

##### 30.1.5.1. JSON 생성 함수
<br/>
JSON 생성 함수는 관계형 데이터로 JSON 데이터를 생성한다.  

###### 30.1.5.1.1. JSON&#95;OBJECT 함수
<br/>
JSON&#95;OBJECT 함수는 string과 expr을 오브젝트 멤버로 포함하는 JSON 오브젝트를 반환한다. size를 생략하면 VARCHAR2(4000) 타입으로 결과가 반환된다.  

```
JSON_OBJECT ([KEY] string VALUE expr [FORMAT JSON]
          [, [KEY] string VALUE expr [FORMAT JSON]]...
             [{NULL | ABSENT} ON NULL] RETURNING VARCHAR2 [(size [BYTE | CHAR])])
```

+ NULL ON NULL : expr이 널이면 널을 반환
+ ABSENT ON NULL : expr이 널이면 값을 생략 (기본값)  

아래는 JSON&#95;OBJECT 함수를 사용한 쿼리다. deptno와 dname를 키로 생성했다.  

```sql
SELECT JSON_OBJECT (KEY 'deptno' VALUE deptno, KEY 'dname' VALUE dname) AS c1
FROM dept;
```

###### 30.1.5.1.2. JSON&#95;OBJECTAGG 함수
<br/>
JSON&#95;OBJECTAGG 함수는 string과 expr을 오브젝트 멤버로 포함하는 단일 JSON 오브젝트를 반환한다. JSON&#95;OBJECTAGG 함수는 집계 함수로 동작한다.  

```
JSON_OBJECT ([KEY] string VALUE expr [FORMAT JSON]
             [{NULL | ABSENT} ON NULL]
             [RETURNING {VARCHAR2 [(size [BYTE | CHAR])] | CLOB}])
```

아래는 JSON&#95;OBJECTAGG 함수를 사용한 쿼리다. deptno 값을 키로 생성했다.  

```sql
SELECT JSON_OBJECTAGG (KEY TO_CHAR(deptno) VALUE dname) AS c1 FROM dept;
```

###### 30.1.5.1.3. JSON&#95;ARRAY 함수
<br/>
JSON&#95;ARRAY 함수는 expr을 JSON 값으로 포함하는 JSON 배열을 반환한다.  

```
JSON_ARRAY (expr [FORMAT JSON] [, expr [FORMAT JSON]]...
            [{NULL | ABSENT} ON NULL]
            [RETURNING VARCHAR2 [(size [BYTE | CHAR])]])
```

아래는 JSON&#95;ARRAY 함수를 사용한 쿼리다.  

```sql
SELECT JSON_ARRAY (deptno
                 , JSON_OBJECT (KEY 'dname' VALUE dname)
                 , JSON_OBJECT (EKY 'loc' VALUE loc)) AS c1
FROM dept;
```

###### 30.1.5.1.4. JSON&#95;ARRAYAGG 함수
<br/>
JSON&#95;ARRAYAGG 함수는 expr을 JSON 값으로 포함하는 단일 JSON 배열을 반환한다. JSON&#95;ARRAYAGG 함수는 집계 함수로 동작한다.  

```sql
JSON_ARRAYAGG (expr [FORMAT JSON] [order_by_cluase]
               [{NULL | ABSENT} ON NULL]
               [RETURNING {VARCHAR2 [(size [BYTE | CHAR])] | CLOB}])
```

아래는 JSON&#95;ARRAYAGG 함수를 사용한 쿼리다.  

```sql
SELECT JSON_ARRAYAGG (dname ORDER BY dname) AS c1 FROM dept;
```

##### 30.1.5.2. JSON 조회 함수
<br/>
JSON 조회 함수는 SQL/JSON Path 표현식을 통해 JSON 값을 조회한다.  

###### 30.1.5.2.1. JSON&#95;VALUE 함수
<br/>
JSON&#95;VALUE 함수는 expr에서 JSON&#95;basic&#95;path&#95;expression에 해당하는 스칼라 JSON 값을 조회한다. JSON&#95;value&#95;return&#95;type에 지정한 데이터 타입으로 결과가 반환된다.  

```
JSON_VALUE (expr [FORMAT JSON], JSON_basic_path_expression
            [RETURNING JSON_value_return_type [ASCII]
            [{ERROR | NULL | DEFAULT litera} ON ERROR]
            [{ERROR | NULL | DEFAULT litera} ON EMPTY]])
```

+ ON ERROR : 에러가 발생한 경우에 대한 처리
+ ON EMPTY : JSON 값이 존재하지 않을 경우에 대한 처리  

JSON&#95;value&#95;return&#95;type을 아래와 같이 지정할 수 있다. ASCII를 지정하면 표준 ASCII 유니코드를 사용한다.  

```
{VARCHAR2 [(size [BYTE | CHAR])] | NUMBER [(precision [, scale])] | DATE | TIMESTAMP | TIMESTAMP WITH TIME ZONE | SDO_GEOMETRY}
```

아래는 JSON&#95;VALUE 함수를 사용한 쿼리다. 첫 번째 배열의 deptno 값을 조회한다.  

```sql
SELECT JSON_VALUE (c2, '$.dept[0].deptno' ERROR ON ERROR) AS c1 FROM t1;
```

다중 값이 반환되면 에러가 발생한다.  

```sql
SELECT JSON_VALUE (c2, '$.dept.deptno') AS c1 FROM t1;

ORA-40470: JSON_VALUE가 복수 값으로 평가되었습니다.
```

아래 쿼리는 에러가 발생하지 않는다. ON ERROR 옵션의 기본값을 50으로 지정했다.  

```sql
SELECT JSON_VALUE (c2, '$.dept.deptno' DEFAULT '50' ON ERROR) AS c1 FROM t1;
```

###### 30.1.5.2.2. JSON&#95;QUERY 함수
<br/>
JSON&#95;QUERY 함수는 expr에서 JSON&#95;basic&#95;path&#95;expression에 해당하는 하나 이상의 JSON 값을 문자열로 반환한다.  

```
JSON_QUERY (expr [FORMAT JSON], JSON_basic_path_expression
            [RETURNING VARCHAR2 [(size [BYTE | CHAR])]] [PRETTY] [ASCII]
            [WITHOUT [ARRAY] WRAPPER | WITH [UNCONDITIONAL | CONDITIONAL] [ARRAY] WRAPPER]
            [{ERROR | NULL | EMPTY | EMPTY ARRAY | EMPTY OBJECT} ON ERROR]
            [{ERROR | NULL | EMPTY | EMPTY ARRAY | EMPTY OBJECT} ON ERROR])
```

+ PRETTY : 개행 문자가 들여쓰기 사용
+ WITHOUT WRAPPER : JSON 오브젝트나 JSON 배열인 경우 기술 (기본값)
+ WITH WRAPPER : JSON 값인 경우 기술
+ WITH CONDITIONAL WRAPPER : JSON 값인 경우에만 래퍼(wrapper)를 사용
+ EMPTY ARRAY : 빈 JSON 배열([])을 반환
+ EMPTY OBJECT : 빈 JSON 오브젝트({})를 반환  

아래는 JSON&#95;QUERY 함수를 사용한 쿼리다.  

```sql
SELECT JSON_QUERY (c2, '$.dept.deptno' WITH WRAPPER) AS c1 FROM t1;
```

WITH WRAPPER 옵션을 지정하지 않고 다중 값을 조회하면 널이 반환된다.  

```sql
SELECT JSON_QUERY (c2, '$.dept.deptno') AS c1 FROM t1;
```

###### 30.1.5.2.3. JSON&#95;TABLE 함수
<br/>
JSON&#95;TABLE 함수는 expr에서 JSON&#95;basic&#95;path&#95;expression에 해당하는 결과를 테이블 형태로 반환한다.  

```
JSON_TABLE (expr [FORMAT JSON], JSON_basic_path_expression
            [{ERROR | NULL | DEFAULT literal} ON ERROR]
            COLUMNS (JSON_column_definition [, JSON_column_definition]...))
```

아래의 JSON&#95;column&#95;definition 절을 사용할 수 있다.  

+ JSON&#95;exists&#95;column : JSON&#95;EXISTS 조건과 동일하게 동작
+ JSON&#95;query&#95;column : JSON&#95;QUERY 함수와 동일하게 동작
+ JSON&#95;value&#95;column : JSON&#95;VALUE 함수와 동일하게 동작
+ JSON&#95;nested&#95;path : 중첩 JSON 오브젝트나 JSON 배열에서 JSON 값을 조회
+ ordinality&#95;column : 행 번호를 반환  

각각의 구문은 아래와 같다.  

+ JSON&#95;exists&#95;column : column&#95;name JSON&#95;value&#95;return&#95;type EXISTS PATH JSON&#95;basic&#95;path&#95;expression [JSON&#95;exists&#95;on&#95;error&#95;clause]
+ JSON&#95;query&#95;column : column&#95;name JSON&#95;value&#95;return&#95;type FORMAT JSON [JSON&#95;query&#95;wrapper&#95;clause] PATH JSON&#95;basic&#95;path&#95;expression [JSON&#95;exists&#95;on&#95;error&#95;clause]
+ JSON&#95;value&#95;column : column&#95;name JSON&#95;value&#95;return&#95;type PATH JSON&#95;basic&#95;path&#95;expression [JSON&#95;exists&#95;on&#95;error&#95;clause]
+ ordinality&#95;column : column&#95;name FOR ORDINALITY  

아래는 JSON&#95;value&#95;column 절을 사용한 쿼리다. JSON 값이 열로 반환된다.  

```sql
SELECT b.*
FROM t1 a
   , JSON_TABLE (a.c2, '$.dept[*]' COLUMNS (deptno NUMBER(2) PATH '$.deptno'
                                          , dname VARCHAR2(14) PATH '$.dname'
                                          , loc VARCHAR2(13) PATH '$.loc')) b
```

아래는 JSON&#95;exists&#95;column 절, JSON&#95;query&#95;column 절, ordinality&#95;column 절을 사용한 쿼리다. c1 열은 deptno 키가 존재하는지 여부, c2 열은 deptno 값, c3 열은 행 번호를 반환한다.  

```sql
SELECT b.*
FROM t1 a
   , JSON_TABLE (a.c2, '$.dept[*]' COLUMNS (c1 VARCHAR2(5) EXISTS PATH '$.deptno'
                                          , c2 VARCHAR2(80) FORMAT JSON WITH WRAPPER PATH '$.deptno'
                                          , c3 FOR ORDINALITY)) b;
```

아래는 JSON&#95;nested&#95;path 절을 사용한 쿼리다. 중첩 JSON 오브젝트를 조회할 수 있다.  

```sql
SELECT *
FROM JSON_TABLE ('{a:1, b:{c:2, d:3}}', '$' COLUMNS (c1 NUMBER PATH '$.a'
                                                   , NESTED PATH '$.b' COLUMNS (c2 NUMBER PATH '$.c'
                                                                              , c3 NUMBER PATH '$.d')));
```

### 30.2. 활용 예제
<br/>
JSON 개발 기능의 활용 예제를 살펴보자.  

LISTAGG 함수는 결과 값이 4000바이트로 제한된다. JSON&#95;ARRAYAGG 함수와 REGEXP&#95;REPLACE 함수를 사용하면 4000바이트 이상의 문자열을 CLOB 값으로 집계할 수 있다.  

```sql
SELECT REGEXP_REPLACE (JSON_ARRAYAGG (ename ORDER BY ename RETURNING CLOB), '\[|"|\]') AS c1 FROM emp;
```
