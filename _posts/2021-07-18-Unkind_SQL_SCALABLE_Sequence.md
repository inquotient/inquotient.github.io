---
title:  SCALABLE 시퀀스
categories:
- Unkind_SQL
feature_text: |
  ## K. SCALABLE 시퀀스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

18.1 버전부터 SCALABLE 시퀀스를 사용할 수 있다. SCALABLE 시퀀스는 right growing 인덱스의 블록 경합을 해소하는 용도로 사용할 수 있다.  

테스트를 위해 아래와 같이 시퀀스를 생성하자. 해당 기능은 12.2 버전에서도 사용할 수 있다. 기능은 구현되었지만 문서화되지 않은 것으로 보인다.  

```sql
DROP SEQUENCE s1;
CREATE SEQUENCE s1 MINVALUE 1 MAXVALUE 999 SCALE EXITEND;
```

SCALABLE 시퀀스는 &#42;&#95;SEQUENCES 뷰의 scale&#95;flag, extend&#95;flag 열이 Y로 표시된다.  

```sql
SELECT min_value, max_value, scale_flag, extend_flag
FROM user_sequences
WHERE sequence_name = 'S1';
```

SYS&#95;CONTEXT 함수로 인스턴스 번호와 세션 ID를 확인하자.  

```sql
SELECT SYS_CONTEXT('USERENV', 'INSTANCE') AS inst
     , SYS_CONTEXT('USERENV', 'SID') AS sid
FROM DUAL;
```

NEXTVAL 슈도 칼럼으로 시퀀스를 조회하면 특이한 형태의 값이 반환된다.  

```sql
SELECT s1.NEXTVAL AS c1 FROM DUAL;
```

아래는 SCALABLE 인덱스와 관련된 파라미터다. SCALABLE 인덱스의 첫 자리는 고정 값(1), 2 ~ 3자리는 인스턴스 번호(01), 4 ~ 6자리는 세션 ID(022), 나머지는 시퀀스 값이 반환된다. 인스턴스와 세션에 따라 선행 6자리 값이 달라지기 때문에 right growing 인덱스의 블록 경합이 해소된다. 시퀀스 값이 저장될 열의 길이는 MAXVALUE 길이 + 6으로 설정해야 한다.  

<table>
  <thead>
    <tr>
      <td>파라미터</td>
      <td>설명</td>
      <td>값</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>&#95;kqdsn&#95;instance&#95;digits</td>
      <td>인스턴스 번호 자릿수</td>
      <td>2</td>
    </tr>
    <tr>
      <td>&#95;kqdsc&#95;cpu&#95;digits</td>
      <td>세션 ID 자릿수</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
<br/><br/>

EXTEND 키워드를 지정하지 않고 SCALABLE 시퀀스를 생성해보자.  

```sql
DROP SEQUENCE s2;
CREATE SEQUENCE s2 MINVALUE 1 MAXVALUE 999 SCALE;
```

NEXTVAL 슈도 칼럼으로 시퀀스를 조회하면 에러가 발생한다.  

```sql
SELECT s2.NEXTVAL AS c1 FROM DUAL;

ORA-64603: S2에 대해 NEXTVAL을 인스턴스화할 수 없습니다. 시퀀스를 4자리 넓히거나 SCALE EXTEND를 사용하여 시퀀스를 변경하십시오.
```

에러 메시지에 따라 MAXVALUE가 7자리인 시퀀스를 생성해보자.  

```sql
DROP SEQUENCE s3;
CREATE SEQUENCE s3 MINVALUE 1 MAXVALUE 9999999 SCALE;
```

이제 에러가 발생하지 않는다. EXTEND 키워드를 사용하지 않으면 선행 6자리 값이 시퀀스 자릿수에 포함된다.  

```sql
SELECT s3.NEXTVAL AS c1 FROM DUAL;
```

12.2 이하 버전도 표현식을 통해 SCALABLE 시퀀스의 동작을 구현할 수 있다. 테스트를 위해 아래와 같이 시퀀스를 생성하자.  

```sql
DROP SEQUENCE s4;
CREATE SEQUENCE s4 MINVALUE 1 MAXVALUE 999;
```

아래와 같이 사용자 함수를 생성하자.  

```sql
CREATE OR REPLACE FUNCTION fnc_ss (
    i_owner IN VARCHAR2 -- 시퀀스 소유자
  , i_name IN VARCHAR2 -- 시퀀스 명
  , i_val IN VARCHAR2 DEFAULT 'NEXTVAL' -- 시퀀스 슈도 칼럼(CURRVAL, NEXTVAL)
  , i_len IN NUMBER DEFAULT 1e28 -- 시퀀스 길이
) RETURN NUMBER
IS
  l_max_val NUMBER;
  l_val NUMBER;
BEGIN
  EXECUTE IMMEDIATE 'SELECT ' || i_owner || '.' || i_name || '.' || i_val || ' FROM DUAL' INTO l_val;

  RETRUN ((1e5 + MOD(SYS_CONTEXT('USERENV', 'INSTANCE'), 100) * 1e3 + MOD(SYS_CONTEXT('USERENV', 'SID'), 1000)) * i_len) + l_val;
END fnc_ss;
/
```

아래는 fnc&#95;ss 함수를 사용한 쿼리다. SCALABLE 시퀀스와 동일한 값이 반환되는 것을 확인할 수 있다.  

```sql
SELECT fnc_ss ('SCOTT', 'S4', 'NEXTVAL', 1e3) AS c1
FROM DUAL;
```
