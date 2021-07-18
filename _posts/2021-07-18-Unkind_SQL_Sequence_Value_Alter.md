---
title:  시퀀스 값 변경
categories:
- Unkind_SQL
feature_text: |
  ## J. 시퀀스 값 변경
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

데이터 이행 시 시퀀스를 동기화하기 위해 시퀀스 값을 변경해야 할 경우가 있다. 시퀀스 값 변경에 대한 내용을 살펴보자.  

예제를 위해 아래와 같이 시퀀스를 생성하자.  

```sql
DROP SEQUENCE s1;
CREATE SEQUENCE s1;
```

아래는 시퀀스 값을 변경하는 프로시저다. 20장에서 살펴본 내용으로 코드를 작성했다.  

```sql
CAREATE ORE REPLACE PROCEDURE prc_chg_seq (
    i_owner IN VARCHAR2
  , i_name IN VARCHAR2
  , i_val IN NUMBER
)
IS
  l_seq VARCHAR2(100) := i_owner || '.' || i_name;
  l_ibc NUMBER;
  l_ibn NUMBER;
  l_nv NUMBER;
BEGIN
  EXECUTE IMMEDIATE 'SELECT ' || i_val || ' - ' || l_seq || '.NEXTVAL - 1 FROM DUAL' INTO l_ibn;

  IF l_ibn <> 0 THEN
    SELECT increment_by INTO l_ibc
    FROM all_sequences
    WHERE sequence_owner = i_owner
    AND sequence_name = i_name;

    EXECUTE IMMEDIATE 'ALTER SEQUENCE ' || l_seq || ' INCREMENT BY ' || l_ibn || CASE i_val WHEN 1 THEN ' MINVALUE 0' END;
    EXECUTE IMMEDIATE 'SELECT ' || l_seq || '.NEXTVAL FROM DUAL' INTO l_nv;
    EXECUTE IMMEDIATE 'ALTER SEQUENCE ' || l_seq || ' INCREMENT BY ' || l_ibc;
  END IF;
END prc_chg_seq;
```

현재 시퀀스 값은 1이다.  

```sql
SELECT s1.NEXTVAL AS s1 FROM DUAL;
```

아래 예제는 시퀀스 값을 100으로 변경한다.  

```sql
EXEC prc_chg_seq ('SCOTT', 'S1', 100);

SELECT s1.NEXTVAL AS s1 FROM DUAL;
```

아래 예제는 시퀀스 값을 200으로 변경한다.  

```sql
EXEC prc_chg_seq('SCOTT', 'S1', 200);

SELECT s1.NEXTVAL AS s1 FROM DUAL;
```

아래 예제는 시퀀스 값을 다시 100으로 변경한다.  

```sql
EXEC prc_chg_seq ('SCOTT' , 'S1', 100);

SELECT s1.NEXTVAL AS s1 FROM DUAL;
```

아래 예제는 시퀀스 값을 1로 변경한다.  

```sql
EXEC prc_chg_seq ('SCOTT' , 'S1', 1);

SELECT s1.NEXTVAL AS s1 FROM DUAL;
```

시퀀스 값을 1로 변경하면 MIN&#95;VALUE 값이 0으로 변경된다. CYCLE 옵션을 사용하는 시퀀스는 재성성하는 편이 바람직하다.  

```sql
SELECT min_value
FROM all_sequences
WHERE sequence_owner = 'SCOTT'
AND sequence_name = 'S1';
```
