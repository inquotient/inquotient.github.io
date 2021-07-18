---
title:  CURSOR 표현식
categories:
- Unkind_SQL
feature_text: |
  ## M. CURSOR 표현식
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

CURSOR 표현식은 서브 쿼리의 결과를 중첩 커서(nested cursor)로 반환한다. 중첩 커서는 PL/SQL의 REF CURSOR와 동일한 유형이다.  

아래는 CURSOR 표현식을 사용한 쿼리다. 스칼라 서브 쿼리의 결과가 중첩 커서를 통해 다중 열, 다중 행으로 반환된다.  

```sql
SELECT a.deptno
     , CURSOR (SELECT x.job, x.sal, x.comm
               FROM emp x
               WHERE x.deptno = a.deptno) AS c1
FROM dept a;
```

예제를 위해 아래와 같이 테이블을 생성하자. 순서(seq)대로 금액(val)이나 비율(rat)을 할인하는 데이터가 입력된 테이블이다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (seq NUMBER, val NUMBER, rat NUMBER);

INSERT INTO t1 VALUES (1, 2000, NULL);
INSERT INTO t1 VALUES (2, NULL, 0.1);
INSERT INTO t1 VALUES (3, 2000, NULL);
INSERT INTO t1 VALUES (4, NULL, 0.2);
COMMIT;
```

아래와 같이 금액(i&#95;val)을 할인하는 함수를 생성하자. 두 번째 매개변수가 SYS&#95;REFCURSOR 타입이다.  

```sql
CREATE OR REPLACE FUNCTION f1 (
    i_val IN NUMBER
  , I_RC IN SYS_REFCURSOR
) RETURN NUMBER
IS
  l_val NUMBER;
  l_rat NUMBER;
  l_rst NUMBER := i_val;
BEGIN
  LOOP
    FETCH i_rc INTO l_val, l_rat;
    EXIT WHEN i_rc%NOTFOUND;
    CASE
      WHEN l_val IS NOT NULL THEN l_rst := l_rst - l_val;
      WHEN l_rat IS NOT NULL THEN l_rst := l_rst - (l_rst * l_rat);
      ELSE NULL;
    END CASE;
  END LOOP;

  CLOSE i_rc;

  RETURN l_rst;
END f1;
/
```

아래는 사용자 함수를 사용한 쿼리다. CURSOR 표현식으로 인수를 입력했다. 20000의 할인된 금액이 반환되는 것을 확인할 수 있다.  

```sql
SELECT f1(20000, CURSOR (SELECT val, rat FROM t1 ORDER BY seq)) AS c1 FROM DUAL;
```
