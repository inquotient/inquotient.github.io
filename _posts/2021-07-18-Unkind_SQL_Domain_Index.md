---
title:  도메인 인덱스
categories:
- Unkind_SQL
feature_text: |
  ## I. 도메인 인덱스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

도메인 인덱스(domain index)는 특정 도메인을 위해 설계된 인덱스다. 오라클 데이터베이스는 텍스트 검색을 위한 Oracle Text Index를 제공한다. 관련 내용을 간단히 살펴보자. 상세한 내용은 Oracle Text Reference를 참조하자.  

### I.1. CONTEXT 인덱스 타입
<br/>
CONTEXT 인덱스 타입은 대량 텍스트를 위한 인덱스 타입이다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE doc PURGE;
CREATE TABLE doc (id NUMBER, content VARCAHR2(4000));

INSERT INTO doc VALUES (1, 'ZYX ABC');
INSERT INTO doc VALUES (1, 'ZYX ABC ABC');
INSERT INTO doc VALUES (1, 'ZYX DEF');
COMMIT;
```

아래 구문으로 CONTEXT 인덱스를 생성할 수 있다.  

```sql
CREATE INDEX doc_x1 ON doc (content) INDEXTYPE IS CTXSYS.CONTEXT;
```

아래 쿼리는 content에 ABC가 포함된 행을 조회한다. CONTAINS 연산자로 텍스트를 검색할 수 있다. SCORE 연산자는 정렬을 위해 사용한다.  

```sql
SELECT a.*, SCORE(1) AS sc
FROM doc a
WHERE CONTAINS (a.content, 'ABC', 1) > 0
ORDER BY SCORE(1);
```

아래 쿼리는 content 열에 ABC나 DEF가 포함된 행을 조회한다. CONTAINS 연산자에 AND(&), OR(|), NOT(~), 누적(,), 대체(=) 등의 연산자를 사용할 수 있다.  

```sql
SELECT a.*, SCORE(1) AS sc
FROM doc a
WHERE CONTAINS (a.content, 'ABC|DEF', 1) > 0
ORDER BY SCORE(1);
```

CONTEXT 인덱스 타입은 자동으로 갱신되지 않는다. DML 작업 후 아래와 같이 CTX&#95;DDEL.SYNC&#95;INDEX 프로시저를 수행해야 한다.  

```sql
EXEC CTX_DDL.SYNC_INDEX ('doc_x1');
```

인덱스 단편화 해소를 위해 CTX_DDL.OPTIMIZE&#95;INDEX 프로시저를 사용할 수 있다. 아래의 세 가지 옵션으로 동작한다.  

<table>
  <thead>
    <tr>
      <td>옵션</td>
      <td>설명</td>
      <td>CTX&#95;DLL.OPTIMIZE&#95;INDEX</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>FAST</td>
      <td>단편화만 해소</td>
      <td>('doc_x1', 'FAST');</td>
    </tr>
    <tr>
      <td>FULL</td>
      <td>전체 인덱스를 최적화</td>
      <td>('doc_x1', 'FULL');</td>
    </tr>
    <tr>
      <td>TOKEN</td>
      <td>특정 토큰만 최적화</td>
      <td>('doc_x1', 'TOKEN', token => 'ABC');
    </tr>
  </tbody>
</table>
<br/><br/>

### I.2. CTXCAT 인덱스 타입
<br/>
CTXCAT 인덱스 타입은 소량 텍스트를 위한 인덱스 타입이다. 인덱스 셋을 생성하면 정형 데이터와 결합된 형태의 인덱스를 생성할 수 있다.  

테스트를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE doc PURGE;
CREATE TABLE doc (id NUMBER, content VARCAHR2(4000), price NUMBER);

INSERT INTO doc VALUES (1, 'ZYX ABC', 10);
INSERT INTO doc VALUES (1, 'ZYX ABC ABC', 20);
INSERT INTO doc VALUES (1, 'ZYX DEF', 30);
COMMIT;
```

아래 프로시저를 인덱스 셋을 생성할 수 있다. price 열을 인덱스 셋에 추가했다.  

```sql
EXEC CTX_DDL.DROP_INDEX_SET('DOC_IS');
EXEC CTX_DDL.CREATE_INDEX_SET('DOC_IS');
EXEC CTX_DDL.ADD_INDEX('DOC_IS', 'PRICE');
```

아래 구문으로 CTXCAT 인덱스를 생성할 수 있다. CTXCAT 인덱스는 DML 작업 시 자동으로 갱신되므로 인덱스 동기화 작업이 불필요하다.  

```sql
CREATE INDEX doc_x1 ON doc (content) INDEXTYPE IS CTXSYS.CTXCAT
PARAMETERS ('INDEX SET DOC_IS');
```

아래 쿼리는 content에 ABC가 포함되어 있고, price가 10 이상 15 이하인 행을 조회한다. CTXCAT 인덱스 타입은 CATSEARCH 연산자를 사용한다.  

```sql
SELECT *
FROM doc
WHERE CATSEARCH(content, 'ABC', 'price BETWEEN 10 AND 15') > 0;
```

아래 쿼리는 content에 ABC가 포함되어 있고, price가 10 이상 30 이하인 행을 조회한다.  

```sql
SELECT *
FROM doc
WHERE CATSEARCH(content, 'ABC', 'price BETWEEN 10 AND 30') > 0;
```

### I.3. CTXRULE 인덱스 타입
<br/>
CTXRULE 인덱스 타입은 규칙에 따라 카테고리를 생성할 수 있는 인덱스 타입이다.  

테스트를 위해 아래와 같이 카테고리 테이블(cat), 문서 테이블(doc), 문서 카테고리 테이블(doc_cat)을 생성하자.  

```sql
DROP TABLE cat PURGE;
DROP TABLE doc PURGE;
DROP TABLE doc_cat PURGE;

CREATE TABLE cat (id NUMBER, name VARCHAR2(100), query VARCAHR2(4000));
CREATE TABLE doc (id NUMBER, name VARCHAR2(100), content VARCAHR2(4000));
CREATE TABLE doc_cat (doc_id NUMBER, cat_id NUMBER);
```

아래와 같이 데이터를 입력하자. query 열에 ABOUT 연산자를 사용했다.  

```sql
INSERT INTO cat VALUES (1, 'ABC', 'ABOUT(ABC)');
INSERT INTO cat VALUES (2, 'ABC', 'ABOUT(DEF)');
COMMIT;
```

아래 구문으로 CTXRULE 인덱스를 생성할 수 있다. CTXRULE 인덱스는 CONTEXT 인덱스처럼 DML 작업 후 인덱스 동기화 작업이 필요하다.  

```sql
CREATE INDEX cat_x1 ON cat (query) INDEXTYPE IS CTXSYS.CTXRULE;
```

아래와 같이 trg&#95;doc 트리거를 생성하자. trc&#95;doc 트리거는 문서 테이블(doc)에 신규 행이 삽입될 때 카테고리 테이블(cat)을 참조하여 문서 카테고리 테이블(doc&#95;cat)에 문서 카테고리 데이터를 삽입한다. query 열에 MATCHES 연산자를 사용했다.  

```sql
CREATE OR REPLACE TRIGGER trg_doc
BEFORE INSERT ON doc FOR EACH ROW
BEGIN
  FOR c1 IN (SELECT id FROM cat WHERE MATCHES (query, :NEW.content) > 0)
  LOOP
    BEGIN
      INSERT INTO doc_cat (doc_id, cat_id) VALUES (:NEW.id, c1.id);
    EXCEPTION
      WHEN OTHERS THEN NULL;
    END;
  END LOOP;
END;
/
```

아래와 같이 데이터를 입력해보자.  

```sql
INSERT INTO doc VALUES (1, 'ABC 1', 'ZYX ABC');
INSERT INTO doc VALUES (2, 'ABC 2', 'ZYX ABC ABC');
INSERT INTO doc VALUES (3, 'DEF 1', 'ZYX DEF');
COMMIT;
```

아래 쿼리로 카테고리에 해당하는 문서를 조회할 수 있다.  

```sql
SELECT c.*
FROM cat a, doc_cat b, doc c
WHERE a.name = 'ABC'
AND b.cat_id = a.id
AND c.id = b.doc_id;
```
