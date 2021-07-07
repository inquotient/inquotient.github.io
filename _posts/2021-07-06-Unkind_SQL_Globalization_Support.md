---
title:  세계화 지원
categories:
- Unkind_SQL
feature_text: |
  ## 23. 세계화 지원
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

세계화 지원(globalization support)은 하나의 데이터베이스를 다수의 국가에서 사용할 수 있도록 지원하는 기능이다.  

### 23.1. 시간대 지원
<br/>
지구는 협정 세계시(Coordinate Universal Time, UTC)인 그리니치 표준시(Greenwich Mean Time, GMT)를 기준으로 24개의 시간대로 나눠져 있다. SYSDATE 함수로 날짜 값을 저장하면 데이터베이스 서버의 날짜 값이 저장되어 서버로 접속한 세션 시간대의 날짜 값을 관리하지 못하는 문제가 발생할 수 있다. 시간대 지원(time zone support)을 통해 날짜 값에 시간대 변위 값을 포함시키면 GMT를 기준으로 해당 시간대의 날짜 값을 계산할 수 있다.  

예제를 위해 NLS 파라미터를 아래와 같이 설정하자.  

```sql
ALTER SESSION SET NLS_DATE_FORMAT = 'YYYY-MM-DD HH24:MI:SS';
ALTER SESSION SET NLS_TIMESTAMP_FORMAT = 'YYYY-MM-DD HH24:MI:SS.FF';
ALTER SESSION SET NLS_TIMESTAMP_TZ_FORMAT = 'YYYY-MM-DD HH24:MI:SS.FF TZH:TZM';
```

#### 23.1.1. 데이터 타입
<br/>
##### 23.1.1.1. TIMESTAMP WITH TIME ZONE 타입
<br/>
TIMESTAMP WITH TIME ZONE 타입은 시간대 변위 값이 포함된 TIMESTAMP 타입이다. fractional&#95;seconds&#95;precision의 범위는 0 ~ 9, 기본값은 6이다.  

```
TIMESTAMP [(fractional_seconds_precision)] WITH TIME ZONE
```

##### 23.1.1.2. TIMESTAMP WITH LOCAL TIME ZONE 타입
<br/>
TIMESTAMP WITH LOCAL TIME ZONE 타입은 시간대 변위 값을 데이터베이스 시간대로 정규화하여 저장한다. 조회시 세션 시간대로 변환된 값이 반환된다. fractional&#95;seconds&#95;precision의 범위는 0 ~ 9, 기본값은 6이다.  

```
TIMESTAMP [(fractional_seconds_precision)] WITH LOCAL TIME ZONE
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;

CARETE TABLE t1 (c1 TIMESTAMP, c2 TIMESTAMP WITH TIME ZONE, c3 TIMESTAMP WITH LOCAL TIME ZONE);
```

&#42;&#95;TAB&#95;COLUMNS 뷰에서 데이터 타입을 확인할 수 있다.  

```sql
SELECT column_name, date_type, data_length, data_scale
FROM user_tab_columns
WHERE table_name = 'T1';
```

c1, c2, c3 열에 동일한 값을 입력해보자.  

```sql
INSERT INTO t1 VALUES (TIMESTAMP '2050-01-01 09:00:00 +09:00'
                     , TIMESTAMP '2050-01-01 09:00:00 +09:00'
                     , TIMESTAMP '2050-01-01 09:00:00 +09:00');
```

t1 테이블을 조회하면 TIMESTAMP WITH TIME ZONE 타입인 c2 열만 시간대가 표시된다.  

```sql
SELECT * FROM t1;
```

아래와 같이 세션 시간대를 +08:00으로 변경해보자.  

```sql
ALTER SESSION SET TIME_ZONE = '+08:00';
```

t1 테이블을 다시 조회하면 TIMESTAMP WITH LOCAL TIME ZONE 타입인 c3 열의 값이 변경된 것을 확인할 수 있다. 세션 시간대에 따라 날짜 값이 변환되어 반환된다.  

```sql
SELECT * FROM t1;
```

#### 23.1.2. 날짜 함수
<br/>
##### 23.1.2.1. EXTRACT 함수
<br/>
EXTRACT 함수는 expr에서 시간대와 관련된 정보를 추출한다.  

```
EXTRACT ({TIMEZONE_HOUR | TIMEZONE_MINUTE | TIMEZONE_REGION | TIMEZONE_ABBR} FROM {expr})
```

아래 쿼리는 EXTRACT 함수를 사용한 쿼리다.  

```sql
WITH w1 AS (SELECT TIMESTAMP '2050-01-01 00:00:00 Asia/Seoul' AS c1 FROM DUAL)
SELECT EXTRACT (TIMEZONE_HOUR FROM c1) AS c1
     , EXTRACT (TIMEZONE_MINUTE FROM c1) AS c2
     , EXTRACT (TIMEZONE_REGION FROM c1) AS c3
     , EXTRACT (TIMEZONE_ABBR FROM c1) AS c4
FROM w1;
```

##### 23.1.2.2. TZ&#95;OFFSET 함수
<br/>
TZ&#95;OFFSET 함수는 시간대 오프셋 문자 값을 반환한다.  

```
TZ_OFFSET ({'time_zone_name' | '{+ | -}hh:mi' | SESSIONTIMEZONE | DBTIMEZONE})
```

아래 쿼리의 c1 열은 서울의 시간대를 반환한다. c2, c3 열은 각각 SESSIONTIMEZONE과 DBTIMEZONE의 시간대를 반환한다.  

```sql
SELECT TZ_OFFSET('Asia/Seoul') AS c1
     , TZ_OFFSET(SESSIONTIMEZONE) AS c2
     , TZ_OFFSET(DBTIMEZONE) AS c3
FROM DUAL;
```

V$TIMEZONE&#95;NAMES 뷰에서 time&#95;zone&#95;name에 대한 정보를 조회할 수 있다.  

```sql
SELECT tzname, tzabbrev FROM v$timezone_names;
```

##### 23.1.2.3. DBTIMEZONE 함수
<br/>
DBTIMEZONE 함수는 데이터베이스 시간대의 문자 값을 반환한다. 데이터베이스 시간대는 데이터베이스를 생성할 때 설정할 수 있다.  

아래는 DBTIMEZONE 함수를 사용한 쿼리다.  

```sql
SELECT DBTIMEZONE AS c1 FROM DUAL;
```

##### 23.1.2.4. SESSIONTIMEZONE 함수
<br/>
SESSIONTIMEZONE 함수는 세션 시간대의 문자 값을 반환한다.  

아래 쿼리에서 세션 시간대(c1)와 데이터베이스 시간대(c2)가 다른 것을 확인할 수 있다.  

```sql
SELECT SESSIONTIMEZONE AS c1, DBTIMEZONE AS c2 FROM DUAL;
```

##### 23.1.2.5. CURRENT&#95;DATE 함수
<br/>
CURRENT&#95;DATE 함수는 세션 시간대의 날짜 값을 반환한다.  

아래 쿼리에서 CURRENT&#95;DATE 함수와 SYSDATE 함수의 결과가 다른 것을 확인할 수 있다.  

```sql
SELECT CURRENT_DATE AS c1, SYSDATE AS c2 FROM DUAL;
```

##### 23.1.2.5. CURRENT&#95;TIMESTAMP 함수
<br/>
CURRENT&#95;TIMESTAMP 함수는 세션 시간대의 TIMESTAMP WITH TIME ZONE 값을 반환한다. precision의 범위는 0 ~ 9, 기본값은 6이다.  

아래 쿼리에서 CURRENT&#95;TIMESTAMP 함수와 SYSTIMESTAMP 함수의 결과가 다른 것을 확인할 수 있다.  

```sql
SELECT CURRENT&#95;TIMESTAMP AS c1, SYSTIMESTAMP AS c2 FROM DUAL;
```

##### 23.1.2.6. LOCALTIMESTAMP 함수
<br/>
LOCALTIMESTAMP 함수는 세션 시간대의 TIMESTAMP 값을 반환한다. timestamp&#95;precision의 범위는 0 ~ 9, 기본값은 6이다.  

아래는 LOCALTIMESTAMP 함수를 사용한 쿼리다.  

```sql
SELECT LOCALTIMESTAMP AS c1, CURRENT_TIMESTAMP AS c2 FROM DUAL;
```

##### 23.1.2.7. FROM&#95;TZ 함수
<br/>
FROM&#95;TZ 함수는 timestamp&#95;value와 time&#95;zone&#95;value에 해당하는 TIMESTAMP WITH TIME ZONE 값을 반환한다.  

```
FROM_TZ(timestamp_value, time_zone_value)
```

아래는 FROM&#95;TZ 함수를 사용한 쿼리다.  

```sql
SELECT FROM_TZ(TIMESTAMP '2050-01-01 00:00:00'. '+09:00') AS c1 FROM DUAL;
```

##### 23.1.2.8. SYS&#95;EXTRACT&#95;UTC 함수
<br/>
SYS&#95;EXTRACT&#95;UTC 함수는 datetime&#95;with&#95;timezone에 해당하는 UTC 값을 반환한다.  

```
SYS_EXTRACT_UTC (datetime_with_timezone)
```

아래 쿼리로 한국이 오전 9시일 때 영국 그리니치 지역이 자정인 것을 확인할 수 있다.  

```sql
SELECT SYS_EXTRACT_UTC(TIMESTAMP '2050-01-01 09:00:00 +09:00') AS c1 FROM DUAL;
```

##### 23.1.2.9. NEW&#95;TIME 함수
<br/>
NEW&#95;TIME 함수는 timezone1 시간대의 date에 해당하는 timezone2 시간대의 DATE 값을 반환한다.  

```
NEW_TIME (date, timezone1, timezone2)
```

아래는 NEW&#95;TIME 함수를 사용한 쿼리다.  

```sql
SELECT NEW_TIME(DATE '2050-01-01', 'PST', 'GMT') AS c1 FROM DUAL;
```

timezone1, timezone2에 아래의 시간대를 사용할 수 있다.  

<table>
  <thead>
    <tr>
      <td>지역</td>
      <td>표준시</td>
      <td>일광절약시간</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>대서양(atlantic)</td>
      <td>AST</td>
      <td>ADT</td>
    </tr>
    <tr>
      <td>베링(bering)</td>
      <td>BST</td>
      <td>BDT</td>
    </tr>
    <tr>
      <td>중부(central)</td>
      <td>CST</td>
      <td>CDT</td>
    </tr>
    <tr>
      <td>동부(eastern)</td>
      <td>EST</td>
      <td>EDT</td>
    </tr>
    <tr>
      <td>그리니치(greenwich)</td>
      <td>GMT</td>
      <td></td>
    </tr>
    <tr>
      <td>알래스카-하와이(alaska-hawaii)</td>
      <td>HST</td>
      <td>HDT</td>
    </tr>
    <tr>
      <td>산악(mountain)</td>
      <td>MST</td>
      <td>MDT</td>
    </tr>
    <tr>
      <td>뉴펀들랜드(newfoundland)</td>
      <td>NST</td>
      <td></td>
    </tr>
    <tr>
      <td>태평양(pacific)</td>
      <td>PST</td>
      <td>PDT</td>
    </tr>
    <tr>
      <td>유콘(yukon)</td>
      <td>YST</td>
      <td>YDT</td>
    </tr>
  </tbody>
</table>
<br/><br/>

##### 23.1.2.10. TO&#95;TIMESTAMP&#95;TZ 함수
<br/>
TO&#95;TIMESTAMP&#95;TZ 함수는 fmt 형식의 char를 TIMESTAMP WITH TIME ZONE 값으로 변환한다.  

```
TO_TIMESTAMP_TZ (char [, fmt [, 'nlsparam']])
```

<table>
  <thead>
    <tr>
      <td>포맷 요소</td>
      <td>설명</td>
      <td>TO&#95;&#42; 날짜 함수</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>TZH</td>
      <td>시간대 시</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>TZM</td>
      <td>시간대 분</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>TZR</td>
      <td>시간대 지역 정보</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>TZD</td>
      <td>일광절약시간</td>
      <td>Y</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래는 TO&#95;TIMESTAMP&#95;TZ 함수를 사용한 쿼리다.  

```sql
SELECT TO_TIMESTAMP_TZ ('2050-01-02 12 +08:00', 'YYYY-MM-DD HH24 TZH:TZM') AS c1 FROM DUAL;
```

#### 22.1.3. 날짜 표현식
<br/>
날짜 표현식(datetime expression)을 사용하면 TIMESTAMP 값에 시간대를 설정할 수 있다.  

```
expr AT {LOCAL | TIME ZONE {'[+ | -]hh:mi' | DBTIMEZONE | 'time_zone_name' | expr}}
```

아래는 날짜 표현식을 사용한 쿼리다.  

```sql
SELECT c1, c1 AT TIME ZONE '+09:00' AS c2 FROM t1;
```

### 23.2. 다국어 지원
<br/>
오라클 데이터베이스에서 다국어를 사용하려면 UTF8, AL32UTF8, AL16UTF16 등의 유니코드 캐릭터 셋을 데이터베이스 캐릭터 셋으로 사용해야 한다. 데이터베이스 캐릭터 셋이 유니코드 캐릭터 셋이 아닌 경우에는 NCHAR, NVARCHAR2, NCLOB 타입을 통해 UTF8, AL16UTF16 등의 유니코드 캐릭터 셋을 내셔널 캐릭터 셋으로 사용할 수 있다.  

#### 23.2.1. 캐릭터 셋
<br/>
캐릭터 셋(character set)은 정보를 표현하기 위한 글자 집합이다. 오라클 데이터베이스는 데이터베이스를 생성할 때 데이터베이스 캐릭터 셋과 내셔널 캐릭터 셋을 지정한다.  

NLS&#95;DATABASE&#95;PARAMETERS 뷰에서 캐릭터 셋을 확인할 수 있다.  

```sql
SELECT parameter, value
FROM nls_database_parameters
WHERE parameter IN ('NLS_CHARACTERSET', 'NLS_NCHAR_CHARACTERSET');
```

국내의 경우 주로 아래의 캐릭터 셋을 사용한다. 캐릭터 셋에 따라 사용할 수 있는 한글의 개수가 다르다. KO16MSWIN949 캐릭터 셋부터 11,172자(KO16KSC5601 + 확장 8,822자)의 한글을 사용할 수 있다. KO16MSWIN949 캐릭터 셋은 기존 2,350자의 한글은 제대로 정렬되지만, 확장 8,822자의 한글은 제대로 정렬되지 않는 문제가 있다. 한글 순서와 유니코드 순서가 일치하지 않기 때문이다. 캐릭터 셋에 따라 한글이 저장되는 크기도 다르다. UTF8, AL32UTF8 캐릭터 셋은 한글 저장에 3바이트를 사용한다.  

<table>
  <thead>
    <tr>
      <td>캐릭터 셋</td>
      <td>한글 개수</td>
      <td>한글 저장</td>
      <td>내셔널 캐릭터 셋</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>KO16KSC5601</td>
      <td>2,350자</td>
      <td>2바이트</td>
      <td>N</td>
    </tr>
    <tr>
      <td>KO16MSWIN949</td>
      <td>11,172자</td>
      <td>2바이트</td>
      <td>N</td>
    </tr>
    <tr>
      <td>UTF8</td>
      <td>11,172자</td>
      <td>3바이트</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>AL32UTF8</td>
      <td>11,172자</td>
      <td>3바이트</td>
      <td>N</td>
    </tr>
    <tr>
      <td>AL16UTF16</td>
      <td>11,172자</td>
      <td>2바이트</td>
      <td>Y</td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 23.2.2. 데이터 타입
<br/>
오라클 데이터베이스는 내셔널 캐릭터 셋을 사용할 수 있는 NCHAR, NVARCHAR2, NCLOB 등의 데이터 타입을 제공한다. 해당 데이터 타입은 두 가지 목적으로 사용된다. 데이터베이스 캐릭터 셋이 KO16KSC5601인 경우에는 확장 한글을 사용하기 위해, KO16MSWIN949인 경우에는 다국어 지원을 위해 사용한다.  

예제를 위해 아래와 같이 테이블을 생성하자. 이번 예제는 SQL Developer에서 수행햐야 한다. SQL&#42;Plus 기본 환경은 유니코드를 지원하지 않는다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 VARCHAR2(10), c3 NVARCHAR2(10));
```

아래와 같이 데이터를 입력하자. 国은 國의 간체다. 두 번째 INSERT 문의 c3 값 앞쪽에 N을 기술했다. 내셔널 캐릭터 셋을 사용하기 위해서는 문자 리터럴 앞쪽에 n이나 N을 기술해야 한다.  

```sql
INSERT INTO t1 VALUES (1, '国', '国');
INSERT INTO t1 VALUES (1, '国', N'国');
COMMIT;
```

t1 테이블을 조회해보면 c1이 2인 행의 c3 값만 정상적인 것을 확인할 수 있다.  

```sql
SELECT * FROM t1;
```

#### 23.2.3. 관련 함수
<br/>
내셔널 캐릭터 셋과 관련하여 아래의 함수를 사용할 수 있다.  

+ NCHR : number에 해당하는 내셔널 캐릭터 셋의 문자 값을 반환
+ TO&#95;NCHAR (character) : character를 내셔널 캐릭터 셋의 문자 값으로 변환
+ TO&#95;NCHAR (number) : number를 내셔널 캐릭터 셋의 문자 값으로 변환
+ TO&#95;NCHAR (datetime) : datetime을 내셔널 캐릭터 셋의 문자 값으로 변환
+ TO&#95;NCLOB : char를 내셔널 캐릭터 셋의 CLOB 값으로 변환
