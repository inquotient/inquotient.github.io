---
title:  XML 개발
categories:
- Unkind_SQL
feature_text: |
  ## 29. XML 개발
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

XML(eXtensible Markup Language)은 WWW에서 데이터를 표현하기 위해 W3C에 의해 개발된 표준 언어다. 오라클 데이터베이슨 9.2 버전부터 오라클 XML DB 기능을 제공하고 있다.  

### 29.1. 기본 문법
<br/>
#### 29.1.1. XMLType
<br/>
XMLType은 XML 값을 저장하기 위한 데이터 타입이다. XMLType으로 열을 생성하면 생성자 함수와 멤버 함수를 사용할 수 있다.  

아래 쿼리는 t1 테이블의 c2 열을 XMLType으로 생성한다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 XMLTYPE);
```

&#42;&#95;XML&#95;TAB&#95;COLS 뷰에서 XMLType 열에 대한 정보를 조회할 수 있다.  

```sql
SELECT column_name, xmlshema, schema_owner, storage_type, anyschema, nonschema
FROM user_xml_tab_cols
WHERE table_name = 'T1';
```

아래 생성자 함수는 CLOB 값을 입력 받는다.  

```
CONSTRUCTOR FUNCTION xmltype (xmldata IN CLOB, ...)
RETURN SELF AS RESULT DETERMINISTIC;
```

아래 쿼리는 CLOB 값을 XMLType 열에 데이터를 삽입힌다.  

```sql
INSERT INTO t1 VALUES (1, XMLTYPE ('<emp><ename>SCOTT</ename><job>ANALYST</job></emp>'));
```

아래 생성자 함수는 BFILE 값을 입력 받는다. XMLType 생성자 함수는 매개변수가 다양한 데이터 타입으로 오버로딩(overloading)되어 있다.  

```
CONSTRUCTOR FUNCTION xmltype (xmldata IN BFILE, csid IN NUMBERM, ...)
RETURN SELF AS RESULT DETERMINISTIC;
```

BFILENAME 함수를 사용하면 외부 XML 파일을 적재할 수 있다. NLS&#95;CHARSET&#95;ID 함수는 캐릭터 셋 ID를 반환한다.  

#### 29.1.2. XMLType 멤버 함수
<br/>
아래와 같은 XMLType의 멤버 함수를 사용할 수 있다.  

<table>
  <thead>
    <tr>
      <td>멤버 함수</td>
      <td>구문</td>
      <td>반환</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>EXTRACT</td>
      <td>MEMBER FUNCTION extract (xpath IN VARCHAR2)</td>
      <td>XMLTYPE</td>
    </tr>
    <tr>
      <td>EXISTSNODE</td>
      <td>MEMBER FUNCTION existsnode (xpath IN VARCHAR2)</td>
      <td>NUMBER</td>
    </tr>
    <tr>
      <td>GETNUMBERVAL</td>
      <td>MEMBER FUNCTION getnumberval ()</td>
      <td>NUMBER</td>
    </tr>
    <tr>
      <td>GETSTRINGVAL</td>
      <td>MEMBER FUNCTION getstringval ()</td>
      <td>VARCHAR2</td>
    </tr>
    <tr>
      <td>GETCLOBVAL</td>
      <td>MEMBER FUNCTION getclobval ()</td>
      <td>CLOB</td>
    </tr>
  </tbody>
</table>
<br/><br/>

각각의 멤버 함수는 아래와 같이 동작한다. EXTRACT 함수와 EXISTSNODE 함수는 XPath 표현식을 사용한다.  

+ EXTRACT : XPath 표현식의 결과를 반환
+ EXISTSNODE : XPath 표현식의 결과가 존재하면 1, 존재하지 않으면 0을 반환
+ GETSTRINGVAL : XMLType 인스턴스의 값을 문자 타입으로 반환
+ GETNUMBERVAL : XMLType 인스턴스의 값을 숫자 타입으로 반환
+ GETCLOBVAL : XMLType 인스턴스의 값을 CLOB 타입을 반환  

#### 29.1.3. XPath 표현식
<br/>
XPath 표현식은 XML 값을 조회하기 위한 표현식이다. XPath 표현식은 노드 트리 구조의 XML 값을 조회한다.  

아래는 자주 사용되는 XPath 구문과 함수다.  

+ / : 루트 노드를 선택
+ / : 자식 노드를 선택
+ // : 현재 노드의 모든 자식 노드를 조회
+ . : 현재 노드를 선택
+ .. : 현재 노드의 부모 노드를 선택
+ @ : 현재 노드이 애트리뷰트(attribute)를 선택
+ &#42; : 모든 노드와 일치
+ @&#42; : 모든 애트리뷰트와 일치
+ | : 복수 노드를 선택
+ [ ] : 술어 표현식을 기술
+ position() : 엘리먼트(element)의 위치를 반환
+ last() : 엘리먼트의 개수를 반환
+ text() : 엘리먼트의 내용(content)를 반환  

예제를 위해 아래와 같이 테이블을 생성하자. XML 함수는 조금 후에 살펴보자.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 AS
SELECT a.deptno AS c1
     , XMLELEMENT ("emps",
                   SELECT XMLAGG (XMLELEMENT ("emp", XMLATTRUBUTES (x.empno AS "empno")
                                                   , XMLFOREST (x.ename AS "ename"
                                                              , x.job AS "job"
                                                              , x.mgr AS "mgr"
                                                              , x.hiredate AS "hiredate"
                                                              , x.sal AS "sal"
                                                              , x.comm AS "comm"))
                          ORDER BY x.empno)
                   FROM emp x
                   WHERE x.deptno = a.deptno)) AS c2
FROM dept a;
```

아래는 t1 테이블을 조회한 결과다. XML 값이 저장된 것을 확인할 수 있다.  

```sql
SELECT * FROM t1;
```

예제를 위해 20, 30, 40번 부서를 삭제하도록 하자.  

```sql
DELETE FROM t1 WHERE c1 IN (20, 3O, 40);
COMMTI;
```

아래는 EXTRACT 멤버 함수를 사용한 쿼리다. /emps/emp/ename 노드를 조회한다.  

```sql
SELECT a.c2 EXTRACT('/emps/emp/ename') AS c2 FROM t1 a;
```

아래 쿼리는 job 노드의 모든 하위 노드를 조회한다.  

```sql
SELECT a.c2.EXTRACT('//job') AS c2 FROM t1 a;
```

아래 쿼리는 /emps/emp/ename 노드와 /emps/emp/job 노드를 함께 조회한다.  

```sql
SELECT a.c2.EXTRACT('/emps/emp/ename|//job') AS c2 FROM t1 a;
```

아래 쿼리는 ename 노드의 부모 노드를 조회한다.  

```sql
SELECT a.c2.EXTRACT('/emps/*/ename/..') AS c2 FROM t1 a;
```

아래 쿼리는 /emps/emp 노드의 empno 애트리뷰트 값을 조회한다. 3개의 값(7782, 7839, 7934)이 조회되었다.  

```sql
SELECT a.c2.EXTRACT ('/emps/emp/@empno') AS c2 FROM t1 a;
```

아래 쿼리는 job 엘리먼트의 내용이 PRESIDENT인 /emps/emp/ename 노드를 조회한다.  

```sql
SELECT a.c2.EXTRACT ('/emps/emp[job = "PRESIDENT"]/ename') AS c2 FROM t1 a;
```

아래 쿼리는 empno 애트리뷰트 값이 7934인 /emps/emp/ename 노드를 조회한다.  

```sql
SELECT a.c2.EXTRACT ('/emps/emp[@empno=7934]/ename') AS c2 FROM t1 a;
```

아래 쿼리는 job 엘리먼트의 내용이 PRESIDENT거나 sal 엘리먼트의 내용이 2000 이상인 /emps/emp/ename 노드를 조회한다.  

```sql
SELECT a.c2.EXTRACT ('/emps/emp[job = "PRESIDENT" or sal >= 2000]') AS c2
FROM t1 a;
```

아래 쿼리는 첫 번째 /emps/emp/ename 노드를 조회한다.  

```sql
SELECT a.c2.EXTRACT ('/emps/emp[1]/ename') AS c2 FROM t1 a;
```

아래 쿼리는 첫 번째, 두 번째 /emps/emp/ename 노드를 조회한다.  

```sql
SELECT a.c2.EXTRACT ('/emps/emp[position() <= 2]/ename') AS c2 FROM t1 a;
```

아래 쿼리는 마지막 /emps/emp/ename 노드를 조회한다.  

```sql
SELECT a.c2.EXTRACT ('/emps/emp[last()]/ename') AS c2 FROM t1 a;
```

#### 29.1.4. XML 함수
<br/>
오라클 데이터베이스는 다양한 XML 함수를 제공한다. 대부분의 XML 함수는 ANSI 표준 SQL 문법을 준수하는 SQL/XML 표준 함수다. XML 함수는 용도에 따라 생성 함수와 조회 함수로 구분할 수 있다.  

##### 29.1.4.1. XML 생성 함수
<br/>
XML 생성 함수는 관계형 데이터로 XML 데이터를 생성한다.  

###### 29.1.4.1.1. XMLELEMENT 함수
<br/>
XMLELEMENT 함수는 XML 엘리먼트를 생성한다.  

```
XMLELEMENT ([NAME] identifier [, XML_attribute_clause] [, value_expr [[AS] c_alias]]...)
```

+ identifier : 엘리먼트의 태그(tag)를 지정
+ XML&#95;attribute&#95;clause : 엘리먼트의 애트리뷰트를 지정
+ value&#95;expr : 엘리먼트의 내용을 지정  

아래는 XMLELEMENT 함수를 사용한 쿼리다. ename 엘리먼트를 생성한다.  

```sql
SELECT XMLELEMENT ("ename", ename) AS c1 FROM emp WHERE empno = 7788;
```

XMLELEMENT 함수를 중첩해서 사용하면 중첩 엘리먼트를 생성할 수 있다.  

```sql
SELECT XMLELEMENT ("emp", XMLELEMENT ("ename", ename)
                        , XMLELEMENT ("job", job)) AS c1
FROM emp
WHERE empno = 7788;
```

XML ATTRIBUTES 절(XML&#95;attributes&#95;clause)은 XML 엘리먼트의 애트리뷰트를 생성한다.  

```
XMLATTRUBUTES (value_expr [[AS] c_alias] [, value_expr [[AS] c_alias]]...)
```

아래는 XML ATTRIBUTES 절을 사용한 쿼리다. ename과 job을 애트리뷰트로 생성했다. 큰 따옴표(")를 사용하면 대소문자를 구분할 수 있다.  

```sql
SELECT XMLELEMENT ("emp", XMLATTRUBUTES (ename, job AS "job")) AS xml
FROM emp
WHERE empno = 7788;
```

아래 쿼리는 애트리뷰트와 중첩 엘리먼트를 함께 생성한다.  

```sql
SELECT XMLELEMENT ("emp", XMLELEMENT (empno AS "empno")
                        , XMLELEMENT ("ename", ename)
                        , XMLELEMENT ("job", job)) AS c1
FROM emp
WHERE empno = 7788;
```

###### 29.1.4.1.2. XMLFOREST 함수
<br/>
XMLFOREST 함수는 다수의 XML 엘리먼트를 생성한다.  

```
XMLFOREST (value_expr [AS c_alias] [, value_expr [AS c_alias]]...)
```

아래는 XMLFOREST 함수를 사용한 쿼리다.  

```sql
SELECT XMLFOREST (ename, job AS "job") AS c1 FROM emp WHERE empno = 7788;
```

아래 쿼리는 XMLFOREST 함수를 XMLELEMENT 함수의 인수로 사용했다.  

```sql
SELECT XMLELEMENT ("emp", XMLATTRUBUTES (empno AS "empno")
                        , XMLFOREST (ename AS "ename", job AS "job")) AS c1
FROM emp
WHERE empno = 7788;
```

###### 29.1.4.1.3. XMLCONCAT 함수
<br/>
XMLCONCAT 함수는 다수의 XMLType 인스턴스를 연결하여 하나의 XML fragment를 생성한다.  

```
XMLCONCAT (XMLType_instance [, XMLType_instance]...)
```

아래는 XMLCONCAT 함수를 사용한 쿼리다.  

```sql
SELECT XMLCONCAT (XMLELEMENT ("ename", ename), XMLELEMENT ("job", job)) AS c1
FROM emp
WHERE empno = 7788;
```

###### 29.1.4.1.4. XMLAGG 함수
<br/>
XMLAGG 함수는 XML 엘리먼트를 집계한다.  

```
XMLAGG (XMLType_instance [order_by_clause])
```

아래는 XMLAGG 함수를 사용한 쿼리다.  

```sql
SELECT XMLAGG (XMLELEMENT ("ename"m ename) ORDER BY empno) AS c1
FROM emp
WHERE deptno = 10;
```

아래 쿼리는 XMLAGG 함수를 XMLELEMENT 함수의 인수로 사용했다.  

```sql
SELECT XMLELEMENT ("emps", XMLAGG (XMLELEMENT ("emp", XMLATTRUBUTES (empno AS "empno")
                                                    , XMLFOREST (ename AS "ename", job AS "job"))
                                   ORDER BY empno)) AS c1
FROM emp
WHERE deptno = 10;
```

###### 29.1.4.1.5. XMLPI 함수
<br/>
XMLPI 함수는 XML 처리 명령어(XML Processing Instruction)를 생성한다.  

```
XMLPI ([NAME] identifier [, value_expr])
```

아래는 XMLPI 함수를 사용한 쿼리다.  

```sql
SELECT XMLPI (NAME "XMLPI", 'pi') AS c1 FROM DUAL;
```

###### 29.1.4.1.6. XMLCOMMENT 함수
<br/>
XMLCOMMENT 함수는 XML 주석을 생성한다.  

```
XMLCOMMENT (value_expr)
```

아래는 XMLCOMMENT 함수를 사용한 쿼리다.  

```sql
SELECT XMLCOMMENT ('comment') AS c1 FROM DUAL;
```

###### 29.1.4.1.7. XMLSERIALIZE 함수
<br/>
XMLSERIALIZE 함수는 XMLType의 value&#95;expr을 datatype에 지정한 데이터 타입으로 변환한다. datatype은 VARCHAR2, CLOB, BLOB 타입을 지정할 수 있다. 기본값은 CLOB 타입이다.  

```
XMLSERIALIZE ({DOCUMENT | CONTENT} value_expr [AS datatype])
```

+ DOCUMENT : value&#95;expr이 유효한 XML 문서임
+ CONTENT : value&#95;expr이 유효한 XML 내용임 (유효한 XML 문서는 아닐 수 있음)

아래는 XMLSERIALIZE 함수를 사용한 쿼리다.  

```sql
SELECT XMLSERIALIZE (CONTENT XMLFOREST (ename AS "ename", job AS "job")) AS c1
FROM emp
WHERE empno = 7788;
```

###### 29.1.4.1.8. XMLPARSE 함수
<br/>
XMLPARSE 함수는 value&#95;expr로 XMLType 인스턴스를 생성한다. WELLFORMED를 지정하면 유효성 검사를 수행하지 않는다.  

```
XMLPARSE ({DOCUMENT | CONTENT} value_expr [WELLFORMED])
```

아래는 XMLPARSE 함수를 사용한 쿼리다.  

```sql
SELECT XMLPARSE (CONTENT '<ename>SCOTT</ename><job>ANALYST</job>') as c1 FROM DUAL;
```

###### 29.1.4.1.9. XMLCOLATTVAL 함수
<br/>
XMLCOLATTVAL 함수는 XMLFOREST 함수와 유사하게 동작한다. 열명으로 애트리뷰트 값을 생성한다.  

```
XMLCOLATTVAL (value_expr [AS c_alias] [, value_expr [AS c_alias]]...)
```

아래는 XMLCOLATTVAL 함수를 사용한 쿼리다. column 앨리먼트가 생성된다.  

```sql
SELECT XMLCOLATTVAL (empno, ename AS "ename") AS c1 FROM emp WHERE empno = 7788;
```

###### 29.1.4.1.10. XMLCDATA 함수
<br/>
XMLCDATA 함수는 value&#95;expr을 CDATA 섹션(character data section)으로 생성한다.  

```
XMLCDATA (value_expr)
```

아래는 XMLCDATA 함수를 사용한 쿼리다.  

```sql
SELECT XMLELEMENT ("ename", XMLCDATA (ename)) AS c1 FROM emp WHERE empno = 7788;
```

###### 29.1.4.1.11. SYS&#95;XMLAGG 함수
<br/>
SYS&#95;XMLAGG 함수는 XMLAGG 함수와 유사하게 동작한다. ROWSET 엘리먼트가 생성되며 XML 포맷을 지정할 수 있다.  

```
SYS_XMLAGG (epxr [, fmt])
```

아래는 SYS&#95;XMLAGG 함수를 사용한 쿼리다.  

```sql
SELECT SYS_XMLAGG (XMLELEMENT ("ename", ename)) AS c1 FROM emp WHERE deptno = 10;
```

##### 29.1.4.2. XML 조회 함수
<br/>
XML 조회 함수는 XQuery 표현식으로 XML 값을 조회한다.  

###### 29.1.4.2.1. XMLQUERY 함수
<br/>
XMLQUERY 함수는 XQuery&#95;string으로 expr을 조회한 결과를 반환한다.  

```
XMLQUERY (XQuery_string
          PASSING [BY VALUE] expr [AS identifier] [, expr [AS identifier]]...
          RETURNING CONTENT [NULL OF EMPTY])
```

아래는 XMLQUERY 함수를 사용한 쿼리다. 첫 번째 /emps/emp/ename 노드를 조회한다.  

```sql
SELECT XMLQUERY ('/emps/emp[1]/ename' PASSING c2 RETURNING CONTENT) AS c2 FROM t1;
```

###### 29.1.4.2.2. XMLTABLE 함수
<br/>
XMLTABLE 함수는 XQuery&#95;string으로 expr을 조회한 결과를 테이블 형태로 반환한다.  

```
XMLTABLE (XQuery_string
          PASSING [BY VALUE] expr [AS identifier] [, expr [AS identifier]]...
          [COLUMNS column datatype [PATH string] [DEFAULT expr]
                [, column datatype [PATH string] [DEFAULT expr]]...])
```

아래는 XMLTABLE 함수를 사용한 쿼리다.  

```sql
SELECT b.* FROM t1 a, XMLTABLE ('/emps/emp[1]/ename' PASSING a.c2) b;
```

아래 쿼리는 COLUMNS 절을 사용하여 XML 값을 테이블 형태로 조회한다.  

```sql
SELECT b.*, a.c1 AS deptno
FROM t1 a
   , XMLTABLE ('/emps/emp' PASSING a.c2
               COLUMNS empno NUMBER(4) PATH '//@empno'
                     , ename VARCHAR2(10) PATH '//ename'
                     , job VARCHAR2(9) PATH '//job'
                     , mgr NUMBER(4) PATH '//mgr'
                     , hiredate DATE PATH '//hiredate'
                     , sal NUMBER(7,2) PATH '//sal'
                     , comm NUMBER(7,2) PATH '//comm' DEFAULT 0) b;
```

###### 29.1.4.2.3. XMLEXISTS 함수
<br/>
XMLEXISTS 함수는 XQuery&#95;string으로 expr을 조회한 결과가 존재하면 TRUE, 존재하지 않으면 FALSE를 반환한다.  

```
XMLEXISTS (XQuery_string [PASSING [BY VALUE] expr [AS identifier]
                                          [, expr [AS identifier]]...])
```

아래는 XMLEXISTS 함수를 사용한 쿼리다. empno 애트리뷰트 값이 7934인 emps/emp/ename 노드를 조회한다.  

```sql
SELECT XMLQUERY ('/emps/emp/ename' PASSING c2 RETURNING CONTENT) AS c2
FROM t1
WHERE XMLEXISTS ('/emps/emp[@empno=7934]' PASSING c2);
```

###### 29.1.4.2.4. XMLCAST 함수
<br/>
XMLCAST 함수는 value&#95;expression을 datatype에 지정한 데이터 타입으로 변환한다.  

```
XMLCAST (value_expression AS datatype)
```

아래는 XMLCAST 함수를 사용한 쿼리다.  

```sql
SELECT XMLCAST (XMLQUERY ('/emps/emp[1]/ename' PASSING c2 RETURNING CONTENT)
                AS VARCHAR2(10)) AS c2
FROM t1;
```

#### 29.1.5. XQuery 표현식
<br/>
XQuery 표현식은 XML 값을 조회하는 표현식이다. 앞서 살펴본 XPath 표현식은 XQuery 표현식의 한 종류다.  

##### 29.1.5.1. FLWOR 표현식
<br/>
FLWOR 표현식을 사용하면 SQL과 유사한 방식으로 XML 값을 조회할 수 있다.  

+ for : 반복해서 변수를 바인딩
+ let : 변수를 바인딩
+ where : 바인딩한 변수를 필터링
+ order by : 필터링된 결과를 정렬
+ return : 정렬된 결과를 반환  

아래는 FLWOR 표현식을 사용한 쿼리다. sal 엘리먼트의 내용이 2000 이상인 emp 엘리먼트에 속한 ename 엘리먼트를 반환한다.  

```sql
SELECT XMLQUERY ('for $x in //emp
                  let $sal := 2000
                  where $x/sal >= $sal
                  order by $x/sal descending
                  return $x/ename'
                 PASSING c2
                 RETURNING CONTENT) AS c2
FROM t1;
```

##### 29.1.5.2. UPDATING 표현식
<br/>
UPDATING 표현식은 XML 데이터를 변경한다. 다양한 하위 표현식을 사용할 수 있다.  

###### 29.1.5.2.1. insert 표현식
<br/>
insert 표현식은 노드를 삽입한다.  

```
insert node <node> {before | after | {[as {first | last}] into}} <node>
```

아래는 insert 표현식을 사용한 쿼리다. 첫 번째 emp 엘리먼트의 마지막에 comm 엘리먼트 노드를 삽입한다.  

```sql
UPDATE t1
SET c2 = XMLQUERY ('copy $i := $p1 modify
                    (for $j in $i//emp[1]
                     return insert node $p2 as last into $j)
                    return $i'
                   PASSING c2 AS "p1", XMLTYPE('<comm>0</comm>') AS "p2"
                   RETURNING CONTENT);
```

###### 29.1.5.2.2. replace 표현식
<br/>
아래 replace 표현식은 노드를 변경한다.  

```
replace node <node> with <node>
```

아래는 replace 표현식을 사용한 쿼리다. 첫 번째 emp 엘리먼트에 속한 comm 엘리먼트 노드를 <commision>100</commision>으로 변경한다.  

```sql
UPDATE t1
SET c2 = XMLQUERY ('copy $i := $p1 modify
                    (for $j in $i//emp[1]/comm
                     return replace node $j with $p2)
                    return $i'
                   PASSING c2 AS "p1"
                         , XMLTYPE('<commision>100</commision>') AS "p2"
                   RETURNING CONTENT);
```

아래 replace 표현식은 엘리먼트의 값을 변경한다.  

```
replace value of node <node> with string
```

아래 쿼리는 첫 번째 emp 엘리먼트에 속한 commision 엘리먼트 노드의 값을 200으로 변경한다.  

```sql
UPDATE t1
SET c2 = XMLQUERY ('copy $i := $p1 modify
                    (for $j in $i//emp[1]/commision
                     return replace value of node $j with $p2)
                    return $i'
                   PASSING c2 AS "p1", 200 AS "p2"
                   RETURNING CONTENT);
```

###### 29.1.5.2.3. rename 표현식
<br/>
rename 표현식은 노드의 태그를 변경한다.  

```
rename node <node> as string
```

아래는 rename 표현식을 사용한 쿼리다. 첫 번째 emp 엘리먼트에 속한 commision 엘리먼트 노드의 태그를 comm으로 변경한다.  

```sql
UPDATE t1
SET c2 = XMLQUERY ('copy $i := $p modify
                    (for $j in $i//emp[1]/commision
                     return rename node $j as "comm")
                    return $i'
                   PASSING c2 AS "p"
                   RETURNING CONTENT);
```

###### 29.1.5.2.4. delete 표현식
<br/>
delete 표현식은 노드를 삭제한다.  

```
delete nodes <node>
```

아래는 delete 표현식을 사용한 쿼리다. 첫 번재 emp 엘리먼트에 속한 comm 엘리먼트 노드를 삭제한다.  

```sql
UPDATE t1
SET c2 = XMLQUERY ('copy $i := $p modify
                    delete nodes $i//emp[1]/comm
                    return $i'
                   PASSING c2 AS "p"
                   RETURNING CONTENT);
```

##### 29.1.5.3. 산술 표현식
<br/>
산술 표현식(arithmetic expression)은 산술식은 계산한다.  

아래는 산술 표현식을 사용한 쿼리다. 문자열로 입력된 산술식을 계산할 수 있다.  

```sql
SELECT XMLCAST (XMLQUERY ('((1 + 2) * 3) div 4' RETURNING CONTENT) AS NUMBER) AS c1
FROM DUAL;
```

##### 29.1.5.4. XQuery 시퀀스
<br/>
XQuery 시퀀스는 시퀀스를 생성한다.  

아래는 XQuery 시퀀스를 사용한 쿼리다. 쉼표(,)로 구분된 값으로 시퀀스를 생성했다.  

```sql
SELECT * FROM XMLTABLE ('1,2,3' COLUMNS c1 NUMBER PATH '.');
```

아래 쿼리는 4부터 6까지의 시퀀스를 생성한다.  

```sql
SELECT * FROM XMLTABLE ('4 to 6' COLUMNS c1 NUMBER PATH '.');
```

아래 쿼리는 홀수 시퀀스를 생성한다.  

```sql
SELECT * FROM XMLTABLE ('for $i in 1 to 5 where $i mod 2 = 1 return $i'
                        COLUMNS c1 NUMBER PATH '.');
```

아래 쿼리는 피보나치 수열 시퀀스를 생성한다.  

```sql
SELECT * FROM XMLTABLE ('for $i in 1 to 2, $j in 1 to 2 let $v := $i * $j return $v'
                        COLUMNS c1 NUMBER PATH '.');
```

아래 쿼리는 문자열 시퀀스를 생성한다.  

```sql
SELECT * FROM XMLTABLE ('"A", "BC", "DEF"' COLUMNS c1 VARCHAR2 (3) PATH '.');
```

##### 29.1.5.5. XQuery 확장 함수
<br/>
XQuery 표현식은 다양한 확장 함수를 사용할 수 있다.  

###### 29.1.5.5.1. fn:matches 함수
<br/>
fn:matches 함수는 $pattern과 일치하는 $input을 반환한다.  

```
fn:matches ($input, $pattern, [,$flags])
```

아래는 fn:matches 함수를 사용한 쿼리다. K가 포함된 ename 엘리먼트를 조회한다.  

```sql
SELECT XMLQUERY ('/emps/emp/ename[fn:matches(text(), "K")]'
                 PASSING c2 RETURNING CONTENT) AS c2
FROM t1;
```

###### 29.1.5.5.2. fn:replace 함수
<br/>
fn:replace 함수는 $input에 포함된 모든 $pattern을 $replacement로 변경한다.  

```
fn:replace ($input, $pattern, $replacement [, $flags])
```

아래는 fn:replace 함수를 사용한 쿼리다. 두 번째 emp 엘리먼트에 속한 ename 엘리먼트의 내용에 포함된 K를 R로 변경한다.  

```sql
SELECT XMLQUERY ('fn:replace(/emps/emp[2]/ename, "K", "R")'
                 PASSING c2 RETURNING CONTENT) AS c2
FROM t1;
```

###### 29.1.5.5.2. fn:tokenize 함수
<br/>
fn:tokenize 함수는 $pattern을 구분자로 $input를 분리한다.  

```
fn:tokenize ($input, $pattern [, $flags])
```

아래는 fn:tokenize 함수를 사용한 쿼리다.  

```sql
SELECT XMLQUERY ('fn:tokenize($p. ",")' PASSING '1,2,3' AS "p"
                 RETURNING CONTENT) AS c1
FROM DUAL;
```

아래 쿼리는 XMLTABLE 함수로 쉼표(,)로 구분된 문자열을 행으로 변환한다.  

```sql
SELECT *
FROM XMLTABLE ('fn:tokenize($p, ",")' PASSING '1,2,3' AS "p"
               COLUMNS c1 NUMBER PATH '.');
```

아래와 같이 열에 저장된 문자열도 행으로 변환할 수 있다. COLUMN&#95;VALUE 슈도 칼럼에 TRIM 함수를 사용하면 ANYDATA 값이 문자 값으로 변환된다.  

```sql
WITH w1 AS (SELECT '1,2,3' AS c1 FROM DUAL)
SELECT TRIM(b.COLUMN_VALUE) AS c1
FROM w1 a, XMLTABLE ('fn:tokenize($p, ",")' PASSING a.c1 AS "p") b;
```

구분자가 쉼표(,)인 경우 아래와 같이 XQuery 시퀀스를 사용할 수도 있다.  

```sql
SELECT TRIM(COLUMN_VALUE) AS c1 FROM XMLTABLE ('1,2,3');
```

###### 29.1.5.5.3. fn:distinct-value 함수
<br/>
fn:distinct-value 함수는 $arg에서 고유한 값으로 구성된 시퀀스를 반환한다.  

```
fn:distinct-value ($arg [, $collation])
```

아래는 fn:distinct-value 함수를 사용한 쿼리다.  

```sql
SELECT XMLQUERY ('fn:distinct-value (fn:tokenize($p, ","))'
                 PASSING '1,2,3,1,2,3' AS "p" RETURNING CONTENT) AS c1
FROM DUAL;
```

###### 29.1.5.5.4. fn:string-join 함수
<br/>
fn:string-join 함수는 문자열 시퀀스를 결합한다.  

```
fn:string-join ($arg1, $arg2)
```

아래는 fn:string-join 함수를 사용한 쿼리다.  

```sql
SELECT *
FROM XMLTABLE ('fn:string-join(
                  for $i in fn:distinct-values(fn:tokenize($p, ","))
                  order by $i
                  return $i
                , ",")'
               PASSING '3,2,1,3,2,1' AS "p"
               COLUMNS c1 VARCHAR2(4000) PATH '.');
```

###### 29.1.5.5.5. fn:collection 함수
<br/>
fn:collection 함수는 URI 형식으로 입력된 $arg의 노드 시퀀스를 반환한다. URI는 Uniform Resource Identifier의 약자로 인터넷에 있는 자원을 나타내는 유일한 주소를 나타낸다.  

```
fn:collection ($arg)
```

아래는 fn:collection 함수를 사용한 쿼리다. SCOTT 스키마를 속한 dept 테이블의 노드 시퀀스를 반환한다.  

```sql
SELECT XMLQUERY ('for $i in fn:collection("oradb:/SCOTT/DEPT")/ROW[DEPTNO = 10] return $i'
                 RETURNING CONTENT) AS c1
FROM DUAL;
```

아래 쿼리는 FLOWR 표현식으로 dept 테이블과 emp 테이블을 조인한다.  

```sql
SELECT XMLQUERY ('for $i in fn:collection("oradb:/SCOTT/DEPT")/ROW[DEPTNO = 10]
                    , $j in fn:collection("oradb:/SCOTT/EMP")
                  where $j//DEPTNO = $i//DEPTNO
                  and $J//JOB = "PRESIDENT"
                  return ($i//DEPTNO, $j//EMPNO)'
                 RETURNING CONTENT) AS c1
FROM DUAL;
```

#### 29.1.6. DBMS&#95;XMLGEN 패키지
<br/>
DBMS&#95;XMLGEN 패키지는 쿼리의 결과로 CLOB 타입이나 XMLType의 XML 데이터를 생성한다.  

##### 29.1.6.1. GETXML 함수
<br/>
GETXML 함수 subquery의 결과로 CLOB 타입의 XML 데이터를 생성한다.  

```
DBMS_XMLGEN.GETXML (subquery IN VARCHAR2, dtdorschema IN number := NONE) RETURN CLOB;
```

아래는 GETXML 함수를 사용한 쿼리다. VERSION 정보가 포함되며 ROWSET 엘리먼트와 ROW 엘리먼트가 생성된다.  

```sql
SELECT DBMS_XMLGEN.GETXML ('SELECT empno, ename FROM emp WHERE deptno = 10') AS c1
FROM DUAL;
```

##### 29.1.6.2. GETXMLTYPE 함수
<br/>
GETXMLTYPE 함수는 sqlquery의 결과로 XMLType의 XML 데이터를 생성한다.  

```
DBMS_XMLGEN.GETXMLTYPE (sqlquery IN VARCHAR2, dtdorschema IN number := NONE) RETURN SYS.XMLTYPE;
```

아래는 GETXML 함수를 사용한 쿼리다. GETXML 함수와 결과가 동일하다.  

```sql
SELECT DBMS_XMLGEN.GETXMLTYPE ('SELECT empno, ename FROM emp WHERE deptno = 10') AS c1
FROM DUAL;
```

### 29.2. 활용 예제
<br/>
XML 개발 기능을 응용하면 SQL의 제약을 극복할 수 있다.  

#### 29.2.1. 문자열 결합
<br/>
LISTAGG 함수는 결과 값이 4000바이트로 제한된다. XMLAGG 함수를 사용하면 4000바이트 이상의 문자열을 CLOB 값으로 집계할 수 있다.  

```sql
SELECT LTRIM (XMLCAST (XMLAGG (XMLELEMENT (x, ',', ename) ORDER BY ename) AS CLOB), ',') AS c1
FROM emp;
```

#### 29.2.2. 동적 쿼리
<br/>
GETXMLTYPE 함수를 사용하면 동적 쿼리를 수행할 수 있다. 아래 쿼리는 GETXMLTYPE 함수를 사용하여 SCOTT 스키마의 dept 테이블과 emp 테이블의 건수를 조회한다.  

```sql
SELECT table_name
     , DBMS_XMLGEN.GETXMLTYPE('SELECT COUNT(*) AS c FROM ' || owner || '.' || table_name).EXTRACT('/ROWSET/ROW/C/text()').GETNUMBERVAL() AS cnt
FROM all_tables
WHERE owner = 'SCOTT'
AND table_name IN ('DEPT', 'EMP');
```

#### 29.2.3. LONG 값 조회
<br/>
예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 VARCHAR2(1) DEFAULT 'A');
```

&#42;&#95;TAB&#95;COLS 뷰의 data&#95;default 열은 LONG 타입이다. LONG 값에 SQL 연산을 수행하면 에러가 발생한다.  

```sql
SELECT 'DEFAULT' || data_default AS c1 FROM user_tab_cols WHERE table_name = 'T1';

ORA-00932: 일관성 없는 데이터 유형: CHAR이(가) 필요하지만 LONG임
```

XMLTABLE 함수를 사용하면 LONG 값을 VARCHAR2 타입으로 변환할 수 있다. 아래 쿼리는 data&#95;default 열에 SQL 연산을 수행해도 에러가 발생하지 않는다.  

```sql
SELECT 'DEFAULT ' || data_default AS c1
FROM XMLTABLE ('ROWSET/ROW'
               PASSING (SELECT DBMS_XMLGEN.GETXMLTYPE (q'[SELECT data_default FROM user_tab_cols WHERE table_name = 'T1']') AS c1 FROM DUAL)
               COLUMNS data_default VARCHAR2 (4000) PATH 'DATA_DEFAULT');
```
