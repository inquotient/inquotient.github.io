---
title:  문자열 결합
categories:
- Unkind_SQL
feature_text: |
  ## E. 문자열 결합
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

일반적인 문자열 결합은 LISTAGG 함수, 4000바이트 이상의 문자열은 XMLAGG 함수와 JSON&#95;ARRAYAGG 함수를 사용하면 된다. 중복 값 제거나 누적 결합이 필요한 경우에는 사용자 정의 집계 함수를 사용해야 한다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);

INSERT INTO t1 VALUES (1, 1);
INSERT INTO t1 VALUES (2, 1);
INSERT INTO t1 VALUES (2, 2);
INSERT INTO t1 VALUES (3, 2);
INSERT INTO t1 VALUES (3, 1);
INSERT INTO t1 VALUES (3, 1);
COMMIT;
```

9.2 버전까지는 계층형 쿼리로 문자열을 결합했다.  

```sql
SELECT c1, LTRIM(MAX(SYS_CONNECT_BY_PATH(c2, ',')), ',') AS c2
FROM (SELECT c1, c2, ROW_NUMBER() OVER(PARTITION BY c1 ORDER BY c2) AS rn FROM t1)
STRAT WITH rn = 1
AND rn - 1 = PRIOR rn
GROUP BY c1
ORDER BY c1;
```

10.1 버전부터 29장에서 살펴본 XMLAGG 함수를 사용할 수 있다. c3 열처럼 값을 CLOB 타입으로 변환하면 4000바이트 이상의 문자열을 결합할 수 있다.  

```sql
SELECT c1
     , LTRIM(XMLCAST(XMLAGG(XMLELEMENT(x, ',', c2) ORDER BY c2) AS VARCHAR2(4000)), ',') AS c2
     , LTRIM(XMLCAST(XMLAGG(XMLELEMENT(x, ',', c2) ORDER BY c2) AS CLOB), ',') AS c3
FROM t1
GROUP BY c1;
```

11.1 버전부터 9장에서 살펴본 LISTAGG 함수를 사용할 수 있다.  

```sql
SELECT c1, LISTAGG(c2, ',') WITHIN GROUP (ORDER BY c2) AS c2
FROM t1
GROUP BY c1;
```

12.2 버전부터 30장에서 살펴본 JSON&#95;ARRAYAGG 함수를 사용할 수 있다.  

```sql
SELECT c1
     , REGEXP_REPLACE (JSON_ARRAYAGG (c2 ORDER BY c2 RETURNING VARCHAR2(4000))
                     , '\[|"|\]') AS c2
     , REGEXP_REPLACE (JSON_ARRAYAGG (c2 ORDER BY c2 RETURNING CLOB)
                     , '\[|"|\]') AS c3
FROM t1
GROUP BY c1;
```

오라클 데이터베이스는 사용자 정의 집계 함수(user-defined aggregate function) 기능을 제공한다. 문자열을 결합하는 사용자 정의 함수를 생성하고 동작을 테스트해보자.  

아래는 t&#95;stragg 사용자 정의 타입을 생성하는 코드다. 톰 카이트(Tom Kyte)가 작성한 stragg 함수를 참조했다.  

```sql
CREATE OR REPLACE TYPE t_stragg AS OBJECT (
    g_str VARCHAR2 (32767)
  , STATIC FUNCTION odciaggregateinitialize (
      sctx IN OUT t_stragg
    ) RETURN NUMBER
  , MEMBER FUNCTION odciaggregateiterate (
        self IN t_stragg
      , returnvalue IN VARCHAR2
      , flags IN NUMBER
    ) RETURN NUMBER
  , MEMBER FUNCTION odciaggregatemerge (
      self IN OUT t_stragg
    , sctx2 IN t_stragg
  ) RETURN NUMBER
);
/
```

아래는 t&#95;stragg 사용자 정의 타입의 본체(body)를 생성하는 코드다.  

```sql
CREATE OR REPLACE TYPE BODY t_stragg
IS
  STATIC FUNCTION odciaggregateinitialize (
    sctx IN OUT t_stragg
  ) RETURN NUMBER
  IS
  BEGIN
    sctx := t_stragg (NULL);
    RETURN ODCICONST.SUCCESS;
  END;

  MEMBER FUNCTION odciaggregateiterate (
      self IN OUT t_stragg
    , value IN VARCHAR2
  ) RETURN NUMBER
  IS
  BEGIN
    IF (g_str IS NOT NULL) THEN
      g_str := g_str || ',' || value;
    ELSE
      g_str := value;
    END IF;

    RETURN ODCICONST.SUCCESS;
  END;

  MEMBER FUNCTION odciaggregateminate (
      self IN t_stragg
    , returnvalue OUT VARCHAR2
    , flags IN NUMBER
  ) RETURN NUMBER
  IS
  BEGIN
    returnvalue := g_str;
    RETURN ODCICONST.SUCCESS;
  END;

  MEMBER FUNCTION odciaggregatemerge (
      self IN OUT t_stragg
    , sctx2 IN t_stragg
  ) RETURN NUMBER
  IS
  BEGIN
    IF (sctx2.g_str IS NOT NULL) THEN
      self.g_str := self.g_str || ',' || sctx2.g_str;
    END IF;

    RETURN ODCICONST.SUCCESS;
  END;
END;
/
```

아래 쿼리는 stragg 사용자 정의 집계 함수를 생성한다. stragg 함수는 10.1 ~ 11.2 버전에서 사용할 수 있는 WMSYS.WM_CONCAT 함수와 유사하게 동작한다. WMSYS.WN_CONCAT 함수는 문서화되지 않은 함수이므로 사용하지 않는 편이 바람직하다.  

```sql
CREATE OR REPLACE FUNCTION stragg (i_value IN VARCHAR2) RETURN VARCHAR2
PARALLEL_ENABLE AGGREGATE USING t_stragg;
/
```

아래는 stragg 함수를 사용한 쿼리다. 결합된 문자열이 정렬되지 않는다.  

```sql
SELECT c1, stragg(c2) AS c2 FROM t1 GROUP BY c1;
```

사용자 정의 집걔 함수는 DISTINCT 키워드를 사용할 수 있다.  

```sql
SELECT c1, stragg (DISTINCT c2) AS c2 FROM t1 GROUP BY c1;
```

LISTAGG 함수는 REGEXP_REPLACE 함수를 통해 중복 값을 제거할 수 있다.  

```sql
SELECT c1
     , REGEXP_REPLACE(LISTAGG(c2, ',') WITHIN GROUP (ORDER BY c2)
                    , '([^,]+),\1', '\1') AS c2
FROM t1
GROUP BY c1;
```

사용자 정의 집계 함수는 KEEP 절을 사용할 수 있다.  

```sql
SELECT c1, stragg(c2) KEEP (DENSE_RANK FIRST ORDER BY c2) AS c2 GROUP BY c1;
```

사용자 정의 집계 함수는 분석 함수로 사용할 수 있다. 문자열이 누적 결합된다.  

```sql
SELECT c1, stragg(DISTINCT c2) AS c2 FROM t1 GROUP BY c1;
```

LISTAGG 함수는 REGEXP&#95;REPLACE 함수를 통해 중복 값을 제거할 수 있다.  

```sql
SELECT c1
     , REGEXP_REPLACE(LISTAGG(c2, ',') WITHIN GROUP (ORDER BY c2)
                      , '([^,]+),\1', '\1') AS c2
FROM t1
GROUP BY c1;
```

사용자 정의 집계 함수는 KEEP 절을 사용할 수 있다.  

```sql
SELECT c1, stragg(c2) KEEP (DENSE_RANK FIRST ORDER BY c2) AS c2 FROM t1 GROUP BY c1;
```

사용자 정의 집계 함수는 분석 함수로 사용할 수 있다. 문자열이 누적 결합된다.  

```sql
SELECT c1, c2, stragg(c2) OVER (PARTITION BY c1 ORDER BY c2) AS c2 FROM t1;
```

LISTAGG 함수는 문자열이 누적 결합되지 않는다.  

```sql
SELECT c1, c2
     , LISTAGG(c2, ',') WITHIN GROUP (ORDER BY c2) OVER (PARTITION BY c1) AS c2
FROM t1;
```

아래 쿼리는 정렬된 값으로 문자열을 결합한다.  

```sql
SELECT c1, MAX(c2) AS c2
FROM (SELECT c1, stragg(c2) OVER (PARTITION BY c1 ORDER BY c2) AS c2 FROM t1)
GROUP BY c1;
```

stragg 함수는 결합된 문자열의 길이가 32KB보다 긴 경우 에러가 발생한다.  

```sql
WITH w1 AS (SELECT 'X' AS c1 FROM DUAL CONNECT BY LEVEL <= 16385)
SELECT LENGTH (stragg(c1)) AS c1 FROM w1;

ORA-06502: PL/SQL: 수치 또는 값 오류: 문자열 버퍼가 너무 작습니다.  
```

32KB보다 긴 문자열을 결합하려면 CLOB 값을 반환하는 사용자 정의 집계 함수를 생성해야 한다. 아래는 t&#95;clobagg 사용자 정의 타입을 생성하는 코드다.  

```sql
CREATE OR REPLACE TYPE t_clobagg AS OBJECT (
    g_clob CLOB
  , STATIC FUNCTION odciaggregateinitialize (
      sctx IN OUT t_clobagg
    ) RETURN NUMBER
  , MEMBER FUNCTION odciaggregateiterate (
        self IN OUT t_clobagg
      , value IN CLOB
    ) RETURN NUMBER
  , MEMBER FUNCTION odciaggregateterminate (
        self IN t_clobagg
      , returnvalue OUT CLOB
      , flags IN NUMBER
    ) RETURN NUMBER
  , MEMBER FUNCTION odciaggregatemerge (
        self IN OUT t_clobagg
      , ctx2 IN t_clobagg
  ) RETURN NUMBER
);
/
```

아래는 t&#95;clobagg 사용자 정의 타입의 본체(body)를 생성하는 코드다.  

```sql
CREATE OR REPLACE TYPE BODY t_clobagg
IS
  STATIC FUNCTION odciaggregateinitialize (
    sctx IN OUT t_clobagg
  ) RETURN NUMBER
  IS
    l_clob CLOB;
  BEGIN
    DBMS_LOB.CREATETEPORARY (l_clob, TRUE, DBMS_LOB.CALL);
    sctx := t_clobagg(l_clob);
    RETURN ODCICONST.SUCCESS;
  END;

  MEMBER FUNCTION odciaggregateiterate (
      self IN OUT t_clobagg
    , value IN CLOB
  ) RETURN NUMBER
  IS
  BEGIN
    IF DBMS_LOB.GETLENGTH(self.g_clob) > 0 THEN
       DBMS_LOB.APPEND(self.g_clob, ',');
    END IF;

    DBMS_LOB.APPEND(self.g_clob, value);
    RETURN ODCICONST.SUCCESS;
  END;

  MEMBER FUNCTION odciaggregateterminate (
      self IN t_clobagg
    , returnvalue OUT CLOB
    , flags IN NUMBER
  ) RETURN NUMBER
  IS
  BEGIN
    returnvalue := self.g_clob;
    RETURN ODCICONST.SUCCESS;
  END;

  MEMBER FUNCTION odciaggregatemerge (
      self IN OUT t_clobagg
    , ctx2 IN t_clobagg
  ) RETURN NUMBER
  IS
  BEGIN
    DBMS_LOB.APPEND(self.g_clob, ctx2.g_clob);
    RETURN ODCICONST.SUCCESS;
  END;
END;
/
```

아래 쿼리는 clobagg 사용자 정의 집계 함수를 생성한다.  

```sql
CREATE OR REPLACE FUNCTION clobagg (i_value IN VARCHAR2) RETURN CLOB
PARALLEL_ENABLE AGGREGATE USING t_clobagg;
/
```

아래는 clobagg 함수를 사용한 쿼리다. 32KB보다 긴 문자열을 결합되는 것을 확인할 수 있다. clobagg 함수는 CLOB 값을 결합하기 때문에 stragg 함수에 비해 수행 속도가 느리다.  

```sql
WITH w1 AS (SELECT 'X' AS c1 FROM DUAL CONNECT BY LEVEL <= 16385)
SELECT LENGTH(clobagg(c1)) AS c1 FROM w1;
```
