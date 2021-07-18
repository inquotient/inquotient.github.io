---
title:  복합 FK 제약 조건
categories:
- Unkind_SQL
feature_text: |
  ## H. 복합 FK 제약 조건
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

복합 FK 제약 조건(composite foreign key constraint)은 FK 제약 조건이 다중 열을 참조한다. 아래의 세 가지 일치 조건으로 동작할 수 있다.  

+ MATCH SIMPLE : 참조하는 열에 널이 있으면 일치하지 않아도 됨
+ MATCH FULL : 참조하는 열이 모두 널이거나 모두 일치해야 함
+ MATCH PARTIAL : 참조하는 열 중에서 널이 아닌 열은 모두 일치해야 함  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 CASCADE CONSTRAINTS PURGE;
DROP TABLE t2 CASCADE CONSTRAINTS PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);
CREATE TABLE t2 (c0 NUMBER, c1 NUMBER, c2 NUMBER);

ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY (c1, c2);
ALTER TABLE t2 ADD CONSTRAINT t2_pk PRIMARY KEY (c0);
ALTER TABLE t2 ADD CONSTRAINT t2_f1 FOREIGN KEY (c1, c2) REFERENCES t1 (c1, c2);

INSERT INTO t1 VALUES (1, 1);
INSERT INTO t1 VALUES (2, 2);
COMMIT;
```

### H.1. MATCH SIMPLE 일치 조건
<br/>
오라클 데이터베이스의 FK 제약 조건은 MATCH SIMPLE 일치 조건으로 동작한다.  

아래와 같이 데이터를 입력해보자. 참조하는 열에 널이 없고 모두 일치하지 않는 두 번째 INSERT 문에서 "ORA-02291: 무결성 제약조건(SCOTT.T2_F1)이 위배되었습니다- 부모 키가 없습니다" 에러가 발생한다.  

```sql
INSERT INTO t2 VALUES (1, 1, 1);
INSERT INTO t2 VALUES (2, 1, 2); -- ORA-02291
INSERT INTO t2 VALUES (3, 1, NULL);
INSERT INTO t2 VALUES (4, 2, NULL);
INSERT INTO t2 VALUES (5, 3, NULL);
INSERT INTO t2 VALUES (6, NULL, 2);
INSERT INTO t2 VALUES (7, NULL, NULL);
```

t2 테이블의 c0이 5인 행에서 부모 테이블에 존재하지 않은 값(3)이 입력된 것을 확인할 수 있다.  

```sql
SELECT * FROM t2;
```

다음 예제를 위해 롤백을 수행하자.  

```sql
ROLLBACK;
```

### H.2. MATCH FULL 일치 조건
<br/>
FK 제약 조건을 MATCH FULL 일치 조건으로 동작시키려면 CHECK 제약 조건을 생성해야 한다.  

아래와 같이 t1 테이블에 CHECK 제약 조건을 추가하자.  

```sql
ALTER TABLE t2 ADD CONSTRAINT t2_c1
CHECK ((c1 IS NULL AND c2 IS NULL) OR (c1 IS NOT NULL AND c2 IS NOT NULL));
```

아래와 같이 데이터를 입력해보자. 참조하는 열이 모두 널이 아니거나 모두 일치하지 않은 세 번째 ~ 여섯 번째 쿼리에서 "ORA-02290: 체크 제약조건(SCOTT.T2_C1)이 위배되었습니다" 에러가 발생한다.  

```sql
INSERT INTO t2 VALUES (1, 1, 1);
INSERT INTO t2 VALUES (2, 1, 2); -- ORA-02291
INSERT INTO t2 VALUES (3, 1, NULL); -- ORA-02291
INSERT INTO t2 VALUES (4, 2, NULL); -- ORA-02291
INSERT INTO t2 VALUES (5, 3, NULL); -- ORA-02291
INSERT INTO t2 VALUES (6, NULL, 2); -- ORA-02291
INSERT INTO t2 VALUES (7, NULL, NULL);
```

t2 테이블에서 참조하는 열이 모두 널이거나 모두 일치하는 것을 확인할 수 있다.  

```sql
SELECT * FROM t2;
```

다음 예제를 위해 롤백을 수행하고 CHECK 제약 조건을 삭제하자.  

```sql
ROLLBACK;
ALTER TABLE t2 DROP CONSTRAINT t2_c1;
```

### H.3. MATCH PARTIAL 일치 조건
<br/>
FK 제약 조건을 MATCH PARTIAL 일치 조건으로 동작시키려면 트리거를 생성해야 한다.  

```sql
CREATE OR REPLACE TRIGGER trg_t2_b_iu_1
BEFORE INSERT OR UPDATE OF c1, c2 ON t2 FOR EACH ROW
DECLARE
  l_c1 NUMBER := 1;
  l_c2 NUMBER := 1;
BEGIN
  IF (:NEW.c1 IS NOT NULL AND (INSERTING OF UPDATING ('c1'))) THEN
    SELECT COUNT(1) INTO l_c1 FROM t1 WHERE c1 = :NEW.c1 AND ROWNUM <= 1;
  END IF;

  IF (:NEW.c2 IS NOT NULL AND (INSERTING OR UPDATING('c2'))) THEN
    SELECT COUNT(1) INTO l_c2 FROM t1 WHERE c2 = :NEW.c2 AND ROWNUM <= 1;
  END IF;

  IF 0 IN (l_c1, l_c2) THEN
    RAISE_APPLICATION_ERROR (-20001, '무결성 제약 조건에 위배됩니다- 부모 키가 없습니다');
  END IF;
END;
/
```

아래와 같이 데이터를 입력하자. 참조하는 열 중에서 널이 아닌 열이 일치하지 않는 다섯 번째 쿼리에서 "ORA-2000: 무결성 제약 조건에 위배됩니다- 부모 키가 없습니다" 에러가 발생한다.  

```sql
INSERT INTO t2 VALUES (1, 1, 1);
INSERT INTO t2 VALUES (2, 1, 2); -- ORA-02291
INSERT INTO t2 VALUES (3, 1, NULL);
INSERT INTO t2 VALUES (4, 2, NULL);
INSERT INTO t2 VALUES (5, 3, NULL); -- ORA-02291
INSERT INTO t2 VALUES (6, NULL, 2);
INSERT INTO t2 VALUES (7, NULL, NULL);
COMMIT;
```

t2 테이블을 조회한 결과는 아래와 같다.  

```sql
SELECT * FROM t2;
```

부모 테이블인 t1 테이블의 행을 삭제하면 전체 열이 일치하는 행은 삭제되지 않지만, 부분 열이 일치하는 행은 삭제된다. MATCH PARTIAL 일치 조건이 위배된다.  

```sql
DELETE FROM t1 WHERE c1 = 1 AND c2 = 1;

ORA-02292: 무결성 제약조건(SCOTT.T2_F1)이 위배되었습니다- 자식 레코드가 발견되었습니다.

DELETE FROM t1 WHERE c1 = 2 AND c2 = 2;
```

다음 예제를 위해 롤백을 수행하자.  

```sql
ROLLBACK;
```

아래와 같이 t1 테이블에도 트리거를 생성하자.  

```sql
CREATE OR REPLACE TRIGGER trg_t1_b_ud_1
BEFORE UPDATE OF c1, c2 OR DELETE ON t1 FOR EACH ROW
DECLARE
  l_c1 NUMBER := 0;
  l_c2 NUMBER := 0;
BEGIN
  IF (DELETING OR UPDATING ('c1')) THEN
    SELECT COUNT(1) INTO l_c1 FROM t2 WHERE c1 := OLD.c1 AND ROWNUM <= 1;
  END IF;

  IF (DELETING OR UPDATING('c2')) THEN
    SELECT COUNT(1) INTO l_c2 FROM t2 WHERE c2 = :OLD.c2 AND ROWNUM <= 1;
  END IF;

  IF 1 IN (l_c1, l_c2) THEN
    RAISE_APPLICATION_ERROR (-20002, '무결성 제약 조건에 위배됩니다- 자식 레코드가 발견되었습니다');
  END IF;
END;
/
```

부모 테이블에서 부분 열이 일치하는 행을 삭제하면 아래와 같이 에러가 발생한다.  

```sql
DELETE FROM t1 WHERE c1 = 2 AND c2 = 2;

ORA-20002: 무결성 제약 조건에 위배됩니다- 자식 레코드가 발견되었습니다
```
