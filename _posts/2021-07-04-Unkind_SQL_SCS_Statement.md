---
title:  SCS 문
categories:
- Unkind_SQL
feature_text: |
  ## 22. SCS 문
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

SCS은 Session Control Statement의 약자다. 세션 제어문으로 해석할 수 있다. SCS 문을 사용하면 사용자 세션의 속성을 동적으로 제어할 수 있다. 세션은 데이터베이스에 로그인한 사용자의 상태를 나타내는 논리적인 개체다.  

### 22.1. 기본 문법
<br/>
ALTER SESSION 문으로 사용자 세션의 옵션과 파라미터를 설정할 수 있다. 파라미터 서정과 관련된 ALTER SESSION SET 절을 살펴보자.  

```
ALTER SESSION
  {ADVISE {COMMIT | ROLLBACK | NOTHING}
  | CLOSE DATABASE LINK dblink
  | {ENABLE | DISABLE} COMMIT IN PROCEDURE
  | {ENABLE | DISABLE | FORCE} PARALLEL {DML | DDL | QUERY} [PARALLEL integer]
  | {ENABLE RESUMABLE [TIMEOUT integer] [NAME string] | DISABLE RESUMABLE}
  | alter_session_set_clause};
```

+ ADVISE : 원격 데이터에비스에 권고를 보내 분산 트랜잭션을 제어
+ CLOSE DATABASE LINK : 명시적으로 DB 링크를 닫음
+ COMMIT IN PROCEDURE : PL/SQL 함수와 프로시저 내의 COMMIT과 ROLLBACK을 제어
+ PARALLEL : 병렬 DML, DDL, QUERY를 제어
+ RESUMABLE : 테이블스페이스가 부족한 경우 재개 가능한 대기 시간을 설정  

#### 22.1.1. ALTER SESSION SET 절
<br/>
ALTER SESSION SET 절은 사용자 세션의 파라미터를 설정한다. 파라미터는 초기화 파라미터와 세션 파라미터로 구분된다.  

```
ALTER SESSION SET { {parameter_name = parameter_value}...}
```

##### 22.1.1.1. 초기화 파라미터
<br/>
초기화 파라미(initialization parameter)는 데이터베이스 기동 시 사용되는 파라미터다. 일부 초기화 파라미터는 세션 레벨에서 동적으로 변경할 수 있다. 세션 레벨에서 변경할 수 있는 초기화 파라미터는 대부분 NLS 설정이나 성능과 관련된 파라미터다.  

V$PARAMETER 뷰에서 초기화 파라미터에 대한 정보를 조회할 수 있다. 아래 쿼리는 세션 레벨에서 변경할 수 있는 초기화 파라미터를 조회한다.  

```sql
SELECT name, value FROM v$parameter WHERE isses_modifiable = 'TRUE' ORDER BY num;
```

V$NLS&#95;PARAMETERS 뷰에서 NLS 파라미터를 조회할 수 있다. NLS는 National Language Support의 약자다. 말 그대로 자국어 지원을 위한 파라미터다.  

```sql
SELECT parameter, value FROM v$nls_parameter;
```

V$NLS&#95;VALID&#95;VALUES 뷰에서 NLS 파라미터의 설정 가능 값에 대한 정보를 조회할 수 있다.  

```sql
SELECT parameter, COUNT(*) AS cnt FROM v$nls_valid_values GROUP BY parameter;
```

아래는 세션 레벨에서 변경할 수 있는 NLS 파라미터다. 일부 파라미터는 다른 파라미터의 영향을 받는다.  

<table>
  <thead>
    <tr>
      <td>파라미터</td>
      <td>설정</td>
      <td>영향</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>NLS&#95;LANGUAGE</td>
      <td>언어</td>
      <td>NLS&#95;LANG 환경 변수</td>
    </tr>
    <tr>
      <td>NLS&#95;TERRITORY</td>
      <td>지역</td>
      <td>NLS&#95;LANG 환경 변수</td>
    </tr>
    <tr>
      <td>NLS&#95;CURRENCY</td>
      <td>로컬 통화 기호</td>
      <td>NLS&#95;TERRITORY</td>
    </tr>
    <tr>
      <td>NLS&#95;DUAL&#95;CURRENCY</td>
      <td>이중 통화 기호</td>
      <td>NLS&#95;TERRITORY</td>
    </tr>
    <tr>
      <td>NLS&#95;ISO&#95;CURRENCY</td>
      <td>국제 통화 기호</td>
      <td>NLS&#95;TERRITORY</td>
    </tr>
    <tr>
      <td>NLS&#95;NUMERIC&#95;CHARACTERS</td>
      <td>숫자 문자</td>
      <td>NLS&#95;TERRITORY</td>
    </tr>
    <tr>
      <td>NLS&#95;CALENDAR</td>
      <td>달력</td>
      <td></td>
    </tr>
    <tr>
      <td>NLS&#95;DATE&#95;LANGUAGE</td>
      <td>날짜 언어</td>
      <td>NLS&#95;LANGUAGE</td>
    </tr>
    <tr>
      <td>NLS&#95;DATE&#95;FORMAT</td>
      <td>DATE 형식</td>
      <td>NLS&#95;TERRITORY</td>
    </tr>
    <tr>
      <td>NLS&#95;TIMESTAMP&#95;FORMAT</td>
      <td>TIMESTAMP 형식</td>
      <td>NLS&#95;TERRITORY</td>
    </tr>
    <tr>
      <td>NLS&#95;TIMESTAMP&#95;TZ&#95;FORMAT</td>
      <td>TIMESTAMP WITH TZ 형식</td>
      <td>NLS&#95;TERRITORY</td>
    </tr>
    <tr>
      <td>NLS&#95;SORT</td>
      <td>정렬 순서</td>
      <td>NLS&#95;LANGUAGE</td>
    </tr>
    <tr>
      <td>NLS&#95;COMP</td>
      <td>비교 순서</td>
      <td></td>
    </tr>
    <tr>
      <td>NLS&#95;LENGTH&#95;SEMANTICS</td>
      <td>문자 타입 길이</td>
      <td></td>
    </tr>
    <tr>
      <td>NLS&#95;NCHAR&#95;CONV&#95;EXCP</td>
      <td>NLS 문자 변환 예외</td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>

###### 21.1.1.1.1. NLS&#95;LANGUAGE 파라미터
<br/>
NLS&#95;LANGUAGE 파라미터는 언어를 설정한다.  

```
ALTER SESSION SET NLS_LAGUAGE = language;
```

아래 쿼리는 NLS&#95;LANGUAGE 파라미터를 ENGLISH로 설정한다. NLS&#95;LANGUAGE 파라미터를 변경하면 오라클 메시지가 설정한 언어로 변경된다.  

```sql
ALTER SESSION SET NLS_LANGUAGE = ENGLISH;
```

V$NLS&#95;PARAMETER 뷰에서 NLS&#95;DATE&#95;LANGUAGE 파라미터도 함께 변경된 것을 확인할 수 있다.  

```sql
SELECT parameter, value
FROM v$nls_parameters
WHERE parameter IN ('NLS_LANGUAGE', 'NLS_DATE_LANGUAGE');
```

NLS&#95;DATE&#95;LANGUAGE 파라미터는 월, 요일의 명칭, 날짜 약어(AM, PM, AD, DC) 등에 영향을 준다.  

```sql
WITH w1 AS (SELECT DATE '2050-01-01' AS c1 FROM DUAL)
SELECT TO_CHAR(c1, 'MONTH') AS c1, TO_CHAR(c1, 'MON') AS c2
     , TO_CHAR(c1, 'DAY') AS c3, TO_CHAR(c1, 'DY') AS c4
     , TO_CHAR(c1, 'D') AS c5
FROM w1;
```

###### 22.1.1.1.2. NLS&#95;TERRITORY 파라미터
<br/>
NLS&#95;TERRITORY 파라미터는 지역을 설정한다.  

```
ALTER SESSION SET NLS_TERRITORY = territory;
```

아래 쿼리는 NLS&#95;TERRITORY 파라미터를 AMERICA로 설정한다.

```sql
ALTER SESSION SET NLS_TERRITORY = AMERICA;
```

V$NLS&#95;PARAMETER 뷰에서 NLS&#95;TERRITORY 파라미터에 영향을 받는 파라미터를 확인할 수 있다.  

```sql
SELECT parameter, value
FROM v$nls_parameters
WHERE parameter IN ('NLS_TERRITORY', 'NLS_CURRENCY', 'NLS_DUAL_CURRENCY', 'NLS_ISO_CURRENCY'
                  , 'NLS_DATE_FORMAT', 'NLS_TIMESTAMP_FORMAT', 'NLS_TIMESTAMP_TZ_FORMAT');
```

아래 쿼리로 통화 관련 파라미터가 변경된 것을 확인할 수 있다.  

```sql
SELECT TO_CHAR(1, '9$') AS c1, TO_CHAR(1, '9L') AS c2
     , TO_CHAR(1, '9U') AS c3, TO_CAHR(1, '9C') AS c4
FROM DUAL;
```

아래 쿼리로 날짜 관련 파라미터가 변경된 것을 확인할 수 있다.  

```sql
SELECT DATE '2050-01-01' AS c1
     , TIMESTAMP '2050-01-01 00:00:00' AS c2
     , TIMESTAMP '2050-01-01 00:00:00 +09:00' AS c3
FROM DUAL;
```

###### 22.1.1.1.3. NLS&#95;DATE&#95;FORMAT 파라미터
<br/>
NLS&#95;DATE&#95;FORMAT 파라미터는 DATE 형식을 설정한다. 날짜 값의 반환 형식과 형식이 지정되지 않은 변환 함수의 기본 포맷으로 사용된다.  

```
ALTER SESSION SET NLS_DATE_FORMAT = "format";
```

아래 쿼리는 에러가 발생한다. 문자 값(19870419)이 포맷(DD-MON-RR)과 일치하지 않아 암시적 데이터 변환이 실패했기 때문이다.  

```sql
SELECT ename FROM emp WHERE hiredate = '19870419';

ORA-01861: literal does not match format string
```

NLS&#95;DATE&#95;FORMAT 파라미터를 YYYY-MM-DD HH24:MI:SS로 변경해보자.  

```sql
ALTER SESSION SET NLS_DATE_FORMAT = 'YYYY-MM-DD HH24:MI:SS';
```

쿼리를 다시 수행하면 에러가 발생하지 않는다. NLS&#95;DATE&#95;FORMAT 파라미터의 설정값에 따라 쿼리의 결과가 달라질 수 있는 것이다. 로컬 개발 환경과 운영 서버 환경의 NLS 파라미터가 일치하지 않는 경우 동일한 현상이 발생할 수 있다.  

```sql
SELECT ename FROM emp WHERE hiredate = '19870419';
```

아래와 같이 TO&#95;DATE 함수를 통해 명시적 데이터 변환을 수행해야 한다.  

```sql
SELECT ename FROM emp WHERE hiredate = TO_DATE('19870419', 'YYYYMMDD');
```

아래 뷰에서도 NLS 파라미터를 확인할 수 있다.  

+ NLS&#95;DATEBASE&#95;PARAMETERS : 데이터베이스의 영구적인 NLS 파라미터
+ NLS&#95;INSTANCE&#95;PARAMETERS : 인스턴스의 NLS 파라미터
+ NLS&#95;SESSION&#95;PARAMETERS : 사용자 세션의 NLS 파라미터 (V$PARAMETER 뷰와 동일)  

##### 22.1.1.2. 세션 파라미터
<br/>
세션 파라미터(session parameter)는 세션 레벨에서만 설정할 수 있는 파라미터다.  

###### 22.1.1.2.1. CONSTRAINT[S] 파라미터
<br/>
CONSTRAINT[S] 파라미터는 DEFFERABLE 제약 조건의 상태를 설정한다. 실제로 SET CONSTRAINT 문과 동일하게 동작한다.  

```
ALTER SESSION SET CONSTRAINT[S] = {IMMEDIATE | DEFFERED | DEFAULT};
```

###### 22.1.1.2.2. CURRENT&#95;SCHEMA 파라미터
<br/>
CURRENT&#95;SCHEMA 파라미터는 세션의 현재 스키마를 설정한다.  

```
ALTER SESSION SET CURRENT_SCHEMA = schema;
```

SCOTT 사용자로 로그인한 세션에서 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
DROP TABLE u1.t1 PURGE;

CREATE TABLE t1 AS SELECRT 1 AS c1 FROM DUAL;
CREATE TABLE u1.t1 AS SELECRT 2 AS c1 FROM DUAL;
```

아래 쿼리를 수행하면 현재 스키마(SCOTT)에 속한 t1 테이블이 조회된다.  

```sql
SELECT * FROM t1;
```

세션의 현재 스키마를 U1 사용자로 설정해보자.  

```sql
ALTER SESSION SET CURRENT_SCHEMA = u1;
```

쿼리를 다시 수행하면 U1 스키마의 t1 테이블이 조회된다.  

```sql
SELECT * FROM t1;
```

SYS&#95;CONTEXT 함수로 현재 세션의 세션 사용자와 현재 스키마를 확인할 수 있다.  

```sql
SELECT SYS_CONTEXT('USERENV', 'SESSION_USER') AS c1
     , SYS_CONTEXT('USERENV', 'CURRENT_SCHEMA') AS c2
FROM DUAL;
```

###### 22.1.1.2.3. ISOLATION&#95;LEVEL 파라미터
<br/>
ISOLATION&#95;LEVEL 파라미터는 트랜잭션 고립화 수준을 설정한다. SET TRANSACTION 문과 동일하게 동작한다.  

```
ALTER SESSION SET ISOLATION_LEVEL = {SERIALIZABLE | READ COMMITTED};
```

###### 22.1.1.2.4. TIME&#95;ZONE 파라미터
<br/>
TIME&#95;ZONE 파라미터는 세션 시간대를 설정한다.  

```
ALTER SESSION SET TIME_ZONE = {'[+|-]hh:mi' | LOCAL | DBTIMEZONE | 'time_zone_region'};
```

현재 세션 시간대는 +09:00다. SYSDATE 함수와 CURRENT&#95;DATE 함수의 결과가 동일하다. CURRENT&#95;DATE 함수는 세션 시간대의 날짜 값을 반환한다.  

```sql
SELECT SESSIONTIMEZONE AS c1, SYSDATE AS c2, CURRENT_DATE AS c2 FROM DUAL;
```

아래와 같이 세션 시간대를 +08:00로 설정해보자.  

```sql
ALTER SESSION SET TIME_ZONE = '+08:00';
```

쿼리를 다시 수행해보면 SYSDATE 함수와 CURRENT_DATE 함수의 결과가 달라진 것을 확인할 수 있다.  

```sql
SELECT SESSIONTIMEZONE AS c1, SYSDATE AS c2, CURRENT_DATE AS c2 FROM DUAL;
```
