---
title:  문자열 분리
categories:
- Unkind_SQL
feature_text: |
  ## L. 문자열 분리
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

문자열 분리에 대한 내용을 살펴보자.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 AS SELECT '1,2,3' AS c1 FROM DUAL;
```

아래는 전통적인 문자열 분리 쿼리다. 쿼리가 다소 복잡하지만 속도는 가장 빠르다.  

```sql
SELECT SUBSTR(a.c1
            , INSTR(a.c1, ',', 1, b.lv) + 1
            , INSTR(a.c1, ',', 1, b.lv) + 1) - INSTR(a.c1, ',', 1, b.lv) - 1) AS c1
FROM (SELECT /*+ NO_MERGE */
             ',' || c1 || ',' AS c1
           , LENGTH(c1) - LENGTH(REPLACE(c1, ',')) + 1 AS cn
      FROM t1) a
   , (SELECT LEVEL AS lv FROM DUAL CONNECT BY LEVEL <= 10) b
WHERE b.lv <= a.cn;
```

아래는 24장에서 살펴본 정규 표현식을 사용한 문자열 분리 쿼리다. 문자열이 긴 경우 쿼리의 성능이 저하될 수 있다.  

```sql
SELECT REGEXP_SUBSTR(a.c1, '[^,]+', 1, b.lv) AS c1
FROM (SELECT /*+ NO_MERGE */ a.*, REGEXP_COUNT(a.c1, ',') + 1 AS cn FROM t1 a) a
   , (SELECT LEVEL AS lv FROM DUAL CONNECT BY LEVEL <= 10) b
WHERE b.lv <= a.cn;
```

아래는 29장에서 살펴본 XMLTABLE 함수를 사용한 문자열 분리 쿼리다. 위 쿼리에 비해 구문이 간결하지만 문맥 전환으로 인한 성능 저하가 발생할 수 있다.  

```sql
SELECT TRIM(b.COLUMN_VALUE) AS c1
FROM t1 a, XMLTABLE ('ora:tokenize($p, ",")' PASSING a.c1 AS "p") b;
```

사용자 정의 함수를 생성하면 쿼리를 간결하게 작성할 수 있다. 예제를 위해 아래와 같이 사용자 정의 타입과 사용자 정의 함수를 생성하자.  

```sql
CREATE OR REPLACE TYPE ntt_varchar2 IS TABLE OF VARCHAR2(4000);
/

CREATE OR REPLACE FUNCTION fnc_split (i_val IN VARCHAR2, i_del IN VARCHAR2 DEFAULT ',')
  RETURN ntt_varchar2 PIPELINED
IS
  l_tmp VARCHAR2(32767) := i_val || i_del;
  l_pos PLS_INTEGER;
BEGIN
  LOOP
    l_pos := INSTR(l_tmp, i_del);
    EXIT WHEN NVL(l_pos, 0) = 0;
    PIPE ROW (SUBSTR(l_tmp, 1, l_pos - 1));
    l_tmp := SUBSTR(l_tmp, l_pos + 1);
  END LOOP;
END fnc_split;
```

아래 쿼리는 fnc&#95;split 함수와 TABLE 컬렉션 표현식을 사용하여 문자열을 분리한다.  

```sql
SELECT b.COLUMN_VALUE AS c1 FROM t1 a, TABLE (fnc_split(a.c1)) b;
```

fnc&#95;split 함수는 가변적인 IN 조건에 활용할 수 있다. 아래 쿼리는 바인드 변수를 사용하여 deptno가 10 또는 20인 행을 조회한다.  

```sql
VARIABLE v1 VARCHAR2(100);

EXEC :v1 := '10,20';

SELECT empno, ename, deptno
FROM emp
WHERE deptno IN (SELECT COLUMN_VALUE FROM TABLE (fnc_split(:v1)));
```

아래 쿼리는 deptno가 30 또는 40인 행을 조회한다.  

```sql
EXEC :v1 := '30,40';

SELECT empno, ename, deptno
FROM emp
WHERE deptno IN (SELECT COLUMN_VALUE FROM TABLE (fnc_split(:v1)));
```
