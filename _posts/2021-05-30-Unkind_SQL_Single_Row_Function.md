---
title: 단일 행 함수
categories:
- Unkind_SQL
feature_text: |
  ## 6. 단일 행 함수
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 6.1. 문자 함수
<br/>
#### 6.1.1. CHR 함수
<br/>
```
CHR(n)
```

n에 해당하는 데이터베이스 캐릭터셋의 문자 값을 반환한다. 문자로 입력할 수 없는 특수 문자를 입력할 때 사용할 수 있다.  

아래는 CHR 함수를 사용한 쿼리다. CHR(10) 표현식은 줄 바꿈(Line Feed) 문자를 의미한다.

```sql
SELECT 'AB' || CHR(10) || ' C' AS c1 FROM DUAL;
```

#### 6.1.2. LOWER 함수
<br/>
```
LOWER(char)
```

char를 소문자로 변경한다.

아래는 LOWER 함수를 사용한 쿼리다.

```sql
SELECT LOWER('abC') FROM DUAL;
```

#### 6.1.3. UPPER 함수
<br/>
```
UPPER(char)
```

char를 대문자로 변경한다.  

아래는 UPPER 함수를 사용한 쿼리다.

```sql
SELECT UPPER('abC') FROM DUAL;
```

#### 6.1.3. INITCAP 함수
<br/>
```
INITCAP(char)
```

char에 포함된 단어의 첫 글자는 대문자, 나머지는 소문자로 변경한다. 단어는 공백이나 숫자와 알파벳이 아닌 문자를 기준으로 구분된다.

```sql
SELECT INITCAP('abC de') AS c1, INITCAP('abC,de') AS c2 FROM DUAL;
```

#### 6.1.4. LPAD 함수
<br/>
```
LPAD(expr1, n [, expr2])
```

expr1의 길이를 좌측으로 n만큼 늘린 후, 늘어난 공간을 expr2로 반복해서 채운다. expr2의 기본값은 공백이다.  

아래는 LPAD 함수를 사용한 쿼리다. c1 열은 좌측 1자리가 공백으로 채워진다. c2 열은 expr1이 n보다 크기 때문에 expr1이 n의 길이로 잘린다. c3 열은 좌측 3자리가 12로 반복해서 채워진다.  

```sql
SELECT LPAD('AB', 3) AS c1, LPAD('AB', 1) AS c2, LPAD('AB', 5, '12') AS c3 FROM DUAL;
```

#### 6.1.5. RPAD 함수
<br/>
```
RPAD(expr1, n [, expr2])
```

expr1의 길이를 우측으로 n만큼 늘린 후, 늘어난 공간을 expr2로 반복해서 채운다. expr2의 기본값은 공백이다.  

아래는 RPAD 함수를 사용한 쿼리다. c1 열은 우측 1자리가 공백으로 채워진다. c2 열은 expr1이 n보다 크기 때문에 expr1이 n의 길이로 잘린다. c3 열은 우측 3자리가 12로 반복해서 채워진다.  

```sql
SELECT RPAD('AB', 3) AS c1, RPAD('AB', 1) AS c2, RPAD('AB', 5, '12') AS c3 FROM DUAL;
```

#### 6.1.6. LTRIM 함수
<br/>
```
LTRIM(char [, set])
```

char의 좌측부터 set에 포함되지 않은 문자를 만날 때까지 set에 포함된 문자를 제거한다. char를 한 문자씩 set과 비교한다. set의 기본값은 공백(' ')이다.  

아래는 LTRIM 함수를 사용한 쿼리다.  

```sql
SELECT LTRIM(' A ') AS c1, LTRIM('ABC', 'BA') AS c2, LTRIM('ABC', 'BC') AS c3 FROM DUAL;
```

#### 6.1.7. RTRIM 함수
<br/>
```
RTRIM(char [, set])
```

char의 우측부터 set에 포함되지 않은 문자를 만날 때까지 set에 포함된 문자를 제거한다. char를 한 문자씩 set과 비교한다. set의 기본값은 공백(' ')이다.  

아래는 RTRIM 함수를 사용한 쿼리다.  

```sql
SELECT RTRIM(' A ') AS c1, RTRIM('ABC', 'BA') AS c2, RTRIM('ABC', 'BC') AS c3 FROM DUAL;
```

#### 6.1.8. TRIM 함수
<br/>
```
TRIM([{{LEADING | TRAILING | BOTH} [trim_character] | trim_character} FROM] trim_source)
```

trim_source의 좌측이나 우측이나 양측에서 trim_character가 아닌 문자를 만날 때까지 trim_character를 제거한다. 위치의 기본값은 BOTH다. trim_character은 한 문자만 지정할 수 있으며 기본값은 공백이다.  

아래는 TRIM 함수를 사용한 쿼리다.  

```sql
SELECT TRIM(LEADING 'A' FROM 'AAB ') AS c1,
       TRIM(LEADING FROM 'AAB ') AS c2,
       TRIM(TRAILING 'B' FROM 'AAB ') AS c3,
       TRIM(TRAILING FROM 'AAB ') AS c4,
       TRIM(BOTH 'A' FROM 'AAB ') AS c5,
       TRIM(BOTH FROM 'AAB ') AS c6,
       TRIM('A' FROM 'AAB ') AS c7,
       TRIM('AAB ') AS c8
FROM DUAL;
```

#### 6.1.9. SUBSTR 함수
<br/>
```
SUBSTR(char, position, [, substring_length])
```
SUBSTR 함수는 char를 position 위치에서 우측으로 substring_length만큼 자른다. substring_length를 생략하면 끝까지 잘린다. position이 음수인 경우 뒤쪽에서 좌측으로 음수만큼 이동한 위치에서 우측으로 자른다. position이 char의 길이보다 크면 널을 반환한다.  

```sql
SELECT SUBSTR('123456', 4) AS c1,
       SUBSTR('123456', 4, 2) AS c2,
       SUBSTR('123456', -4) AS c3,
       SUBSTR('123456', -4, 2) AS c4,
       SUBSTR('123456', 7) AS c5
FROM DUAL;
```

SUBSTR 함수는 아래와 같은 패밀리(family) 함수를 가지고 있다.  

+ SUBSTRB : 바이트를 사용
+ SUBSTRC : 유니코드를 사용
+ SUBSTR2 : UCS2 코드 포인트를 사용
+ SUBSTR4 : UCS4 코드 포인트를 사용  

SUBSTRB 함수는 고정 길이(fixed lenght) 문자열을 가공할 때 사용할 수 있다. 아래 쿼리에서 c1 열은 문자 길이로 잘려 4가 반환되지만, c2 열은 바이트 길잉로 잘려 34r가 반환된다.

```sql
SELECT SUBSTR('가234', 4) AS c1, SUBSTRB('가234', 4) AS c2 FROM DUAL;
```

#### 6.1.10. REPLACE 함수
<br/>
```
REPLACE(char, search_string [, replacement_string])
```

char에 포함된 search_string를 replacement_string으로 변경한다. replacement_string의 기본값은 널이다.  

아래는 REPLACE 함수를 사용한 쿼리다.  

```sql
SELECT REPLACE('ABCABC', 'AB') AS c1,
       REPLACE('ABCABC', 'AB', 'X') AS c2,
       REPLACE('ABCABC', 'AB', 'XYZ') AS c3
FROM DUAL;
```

#### 6.1.11. TRANSLATE 함수
<br/>
```
TRANSLATE(expr, from_string, to_string)
```

expr의 문자를 from_string 문자와 대응되는 to_string 문자로 변환한다. from_string 문자와 일치하지 않는 문자는 변환하지 않는다.  

아래는 TRANSLATE 함수를 사용한 쿼리다. c1 열의 B는 널, c2 열의 B는 공란과 대응된다. c3 열의 Z는 일치하는 문자가 없으므로 무시된다.  

```sql
SELECT TRANSLATE('AAABBC', 'AB', 'X') AS c1,
       TRANSLATE('AAABBC', 'AB', 'X ') AS c2,
       TRANSLATE('AAABBC', 'AB', 'XYZ') AS c3
FROM DUAL;
```

TRANSLATE 함수를 사용하면 특정 값을 제거하거나 추출할 수 있다. 아래 쿼리에서 c1 열은 숫자 값을 제거하고, c2 열은 숫자 값을 추출한다.  

```sql
WITH w1 AS (SELECT 'A1B2C3' AS c1 FROM DUAL)
SELECT c1,
       TRANSLATE(c1, '!1234567890', '!') AS c2,
       TRANSLATE(c1, '1234567890' || c1, '1234567890') AS c3
FROM w1;
```

#### 6.1.12. ASCII 함수
<br/>
```
ASCII(char)
```

char 첫 문자의 아스키 값을 십진수로 반환한다.  

```sql
SELECT ASCII('A') AS c1, ASCII('BC') AS c2, CHR(ASCII('A') + 1) AS c3 FROM DUAL;
```

#### 6.1.13. INSTR 함수
<br/>
```
INSTR(string, substring, [, position [, occurence]])
```

string의 position에서 우측으로 occurence번째 substring의 시작 위치를 반환한다. position과 occurence의 기본값은 1이다.  

아래는 INSTR 함수를 사용한 쿼리다. 강조한 부분이 시작 위치다. c2 열처럼 substring을 찾을 수 없거나 c3 열처럼 position이 string의 길이보다 긴 경우 0을 반환한다. c6, c7 열처럼 position이 음수인 경우 position이 음수인 경우 position에서 좌측으로 substring을 검색한다.  

```sql
SELECT INSTR('ABABABAB', 'AB') AS c1,
       INSTR('ABABABAB', 'AC') AS c2,
       INSTR('ABABABAB', 'AB', 9) AS c3,
       INSTR('ABABABAB', 'AB', 1, 2) AS c4,
       INSTR('ABABABAB', 'AB', 3, 2) AS c5,
       INSTR('ABABABAB', 'AB', -1, 2) AS c6,
       INSTR('ABABABAB', 'AB', -3, 2) AS c7
FROM DUAL;
```

아래 쿼리의 c2, c3 열은 쉼표(,)를 구분자로 c1 열을 잘라낸다. c4, c5 열처럼 정규 표현식을 사용하면 쿼리를 간결하게 작성할 수 있다.  

```sql
WITH w1 AS (SELECT ',A,B,' AS c1 FROM DUAL)
SELECT c1,
       SUBSTR(c1, INSTR(c1, ',', 1, 1) + 1
                , INSTR(c1, ',', 1, 2) - INSTR(c1, ',', 1, 1) - 1) AS c2,
       SUBSTR(c1, INSTR(c1, ',', 1, 2) + 1
                , INSTR(c1, ',', 1, 3) - INSTR(c1, ',', 1, 2) - 1) AS c3,
       REGEXP_SUBSTR(c1, '[^,]+', 1, 1) AS c4,
       REGEXP_SUBSTR(c1, '[^,]+', 1, 2) AS c5,
FROM DUAL;
```

#### 6.1.14. LENGTH 함수
<br/>
```
LENGTH(char)
```

char의 길이를 반환한다.  


아래는 LENGTH 함수를 사용한 쿼리다.  

```sql
SELECT LENGTH('AB') AS c1, LENGTH('AB  ') AS c2 FROM DUAL;
```

LENGTH 함수도 SUBSTR 함수처럼 패밀리 함수를 가지고 있다. c1 열은 2(글자), c2 열은 3(바이트)를 반환한다.  

```sql
SELECT LENGTH('A가') AS c1, LENGTHB('A가') AS c2 FROM DUAL;
```

아래 쿼리의 c2, c3 열은 c1 문자열에 포함된 쉼표(,)의 개수를 계산한다. c4 열처럼 정규 표현식을 사용하면 쿼리를 간결하게 작성할 수 있다.  

```sql
WITH w1 AS (SELECT ',A,B,' AS c1 FROM DUAL)
SELECT c1,
       LENGTH(c1) - LENGTH(REPLACE(c1, ',')) AS c2,
       LENGTH(TRANSLATE(c1, ',' || c1, ',')) AS c3,
       REGEXP_COUNT(c1, ',') AS c4
FROM DUAL;
```

### 6.2. 숫자 함수
<br/>
#### 6.2.1. ABS 함수
<br/>
```
ABS(n)
```
n의 절대값을 반환한다.  

아래는 ABS 함수를 사용한 쿼리다.  

```sql
SELECT ABS(0) AS c1, ABS(10) AS c2, ABS(-10) AS c3 FROM DUAL;
```

#### 6.2.2. SIGN 함수
<br/>
```
SIGN(n)
```

n의 부호를 반환한다. n이 양수면 1, 음수면 -1, 0이면 0을 반환한다.  

아래는 SIGN 함수를 사용한 쿼리다.  

```sql
SELECT SIGN(0) AS c1, SIGN(10) AS c2, SIGN(-10) AS c3 FROM DUAL;
```

#### 6.2.3. ROUND(number) 함수
<br/>
```
ROUND(n1, [, n2])
```

n1을 n2자리로 반올림한다. n2가 양수면 소수부, 음수면 정수부를 반올림한다. n2의 기본값은 0이다.  

아래는 ROUND 함수를 사용한 쿼리다. 강조한 부분이 반올림되는 영역이다.  

```sql
SELECT ROUND(15.59) AS c1, ROUND(15.59, 1) AS c2, ROUND(15.59, -1) AS c3 FROM DUAL;
```

#### 6.2.4. TRUNC(number) 함수
<br/>
```
TRUNC(n1, [, n2])
```

n1을 n2자리로 버린다. n2가 양수면 소수부, 음수면 정수부를 버린다. n2의 기본값은 0이다.  

아래는 TURNC 함수를 사용한 쿼리다.  

```sql
SELECT TRUNC(15.59) AS c1, TRUNC(15.59, 1) AS c2, TRUNC(15.59, -1) AS c3 FROM DUAL;
```
#### 6.2.5. CEIL 함수
<br/>
```
CEIL(n)
```

n보다 크거나 같은 정수의 최소값을 반환한다.  

아래는 CEIL 함수를 사용한 쿼리다.  

```sql
SELECT CEIL(15.5) AS c1, CEIL(16) AS c2 FROM DUAL;
```

#### 6.2.6. FLOOR 함수
<br/>
```
FLOOR(n)
```

n보다 작거나 같은 정수의 최대값을 반환한다.  

아래는 FLOOR 함수를 사용한 쿼리다.  

```sql
SELECT FLOOR(15.5) AS c1, FLOOR(16) AS c2 FROM DUAL;
```

#### 6.2.7. MOD 함수
<br/>
```
MOD(n1, n2)
```

n1을 n2로 나눈 나머지를 반환한다. n2가 0이면 n1을 반환한다. n1이나 n2가 음수면 절대값으로 계산한 값에 n1의 부호를 적용한다.  

아래는 MOD 함수를 사용한 쿼리다.  

```sql
SELECT MOD(11, 4) AS c1, MOD(11, -4) AS c2,
       MOD(-11, 4) AS c3, MOD(-11, -4) AS c4, MOD(11, 0) AS c5
FROM DUAL;
```

#### 6.2.8. REMAINDER 함수
<br/>
```
REMAINDER(n1, n2)
```
n1을 n2로 나눈 나머지를 반환한다. n2가 0이면 에러가 발생한다.  

MOD 함수와 REMAINDER 함수는 계산 공식이 상이하다.  

+ MOD 함수 : n1 - (n2 &#42; FLOOR (n1 / n2))
+ REMAINDER 함수 : n1 - (n2 &#42; ROUND (n1 / n2))  

아래는 REMAINDER 함수를 사용한 쿼리다.  

```sql
SELECT REMAINDER(11, 4) AS c1, REMAINDER(11, -4) AS c2,
       REMAINDER(-11, 4) AS c3, REMAINDER(-11, -4) AS c4
FROM DUAL;
```

#### 6.2.9. POWER 함수
<br/>
```
POWER(n1, n2)
```

n1을 n2로 거듭제곱한 값을 반환한다.  

아래 쿼리의 c2 열은 바이트를 기가 바이트(GB)로 환산한다.  

```sql
SELECT POWER(2, 10) AS c1, 107374182400 / POWER (1024, 3) AS c2 FROM DUAL;
```

#### 6.2.10. BITAND 함수
<br/>
```
BITAND(expr1, expr2)
```
expr1, expr2의 비트 AND 연산 값을 반환한다. 비트 AND 연산은 비교되는 비트가 모두 1이면 1이고, 그렇지 않으면 0이다.  

아래 쿼리는 6(110)과 3(011)의 비트 AND 연산 값은 2(010)다.  

```sql
SELECT BITAND(6, 3) AS c1 FROM DUAL;
```

#### 6.2.11. WIDTH&#95;BUCKET 함수
<br/>
```
WIDTH_BUCKET(expr, min_value, max_value, num_buckets)
```

min_value ~ max_value의 범위에 대해 num_buckets개의 버킷을 생성한 후, expr이 속한 버킷을 반환한다. 버킷은 반개구간(semi-open interval)으로 생성한다.  

아래는 WIDTH&#95;BUCKET 함수를 사용한 쿼리다. expr이 min_value 미만이면 0, max_value이상이면 num_buckets + 1 값을 반환한다. WITH 절에서 CONNECT BY 절로 6행의 순번을 생성했다.  

```sql
WITH w1 AS (SELECT LEVEL AS c1 FROM CONNECT BY LEVEL <= 6)
SELECT c1, WIDTH_BUCKET (c1, 2, 6, 2) AS c2 FROM w1;
```

위 커리는 아래와 같은 버킷을 생성한다.  

<table>
  <thead>
    <tr>
      <td>c1</td>
      <td>c2</td>
      <td>버킷</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>언더플로우 버킷: 0</td>
    </tr>
    <tr>
      <td>2</td>
      <td>1</td>
      <td>1번 버킷: [2,4)</td>
    </tr>
    <tr>
      <td>3</td>
      <td>1</td>
      <td>1번 버킷</td>
    </tr>
    <tr>
      <td>4</td>
      <td>2</td>
      <td>2번 버킷: [4,6)</td>
    </tr>
    <tr>
      <td>5</td>
      <td>2</td>
      <td>2번 버킷</td>
    </tr>
    <tr>
      <td>6</td>
      <td>3</td>
      <td>오버플로우 버킷: num_buckets + 1</td>
    </tr>
  </tbody>
</table>
<br/><br/>

지금까지 살펴본 숫자 함수 외에도 초월 함수와 삼각 함수 등 다양한 숫자 함수를 사용할 수 있다.  

+ SQRT(n)  
n의 제곱근을 반환
+ EXP(n)  
e의 n제곱을 반환
+ LN(n)  
n의 자연로그 값을 반환
+ LOG(n1, n2)  
밑을 n1으로 하는 n2의 로그 값을 반환
+ SIN(n)  
n의 사인(sine) 값을 반환
+ ASIN(n)  
n의 역 사인(arc sine) 값을 반환
+ SINH(n)  
n의 쌍곡선 사인(hyperbolic sine) 값을 반환
+ COS(n)  
n의 코사인(cosine) 값을 반환
+ ACOS(n)  
n의 역 코사인(arc cosine) 값을 반환
+ COSH(n)  
n의 쌍곡선 코사인(hyperbolic cosine) 값을 반환
+ TAN(n)  
n의 탄젠트(tangent) 값을 반환
+ ATAN(n)  
n의 역 탄젠트(arc tangent) 값을 반환
+ ATAN2(n1, n2)  
n1/n2의 역 탄젠트(arc tangent) 값을 반환
+ TANH(n)  
n의 쌍곡선 탄젠트(hyperbolic tangent) 값을 반환  

### 6.3. 날짜 함수
<br/>
#### 6.3.1. SYSDATE 함수
<br/>
데이터베이스 서버의 날짜 값을 반환한다. 초가 포함된 DATE 타입이 반환된다.  

#### 6.3.2. SYSTIMESTAMP 함수
<br/>
시간대가 포함된 데이터베이스 서버의 날짜 값을 반환한다. 소수점 이하 초가 포함된 SYSTIMESTAMP WITH TIME ZONE 타입이 반환된다.  

#### 6.3.3. NEXT&#95;DAY 함수
<br/>
```
NEXT_DAY(date, char)
```
date 이후 char에 지정된 요일에 해당하는 가장 가까운 날짜 값을 반환한다.  

char는 아래와 같이 지정할 수 있다.  
일(1), 월(2), 화(3), 수(4), 목(5), 금(6), 토(7)  

아래는 NEXT&#95;DAY 함수를 사용한 쿼리다.  

```sql
SELECT NEXT_DAY(DATE '2050-01-01', '2') AS c1 FROM DUAL;
```

char는 요일 형식으로도 지정할 수 있다. 아래 쿼리는 에러가 발생하지 않는다.  

```sql
SELECT NEXT_DAY(DATE '2050-01-01', '월') AS c1 FROM DUAL;
```

요일 형식은 NLS 파라미터에 따라 에러가 발생할 수 있기 때문에 사용하지 않는 편이 바람직하다.  

#### 6.3.4. LAST&#95;DAY 함수
<br/>
```
LAST_DAY(date)
```

date가 속한 월의 월말일을 반환한다.  

아래는 LAST&#95;DAY 함수를 사용한 쿼리다.  

```sql
SELECT LAST_DAY(DATE '2050-02-15') AS c1 FROM DUAL;
```

#### 6.3.4. ADD&#95;MONTH 함수
<br/>
```
ADD_MONTH(date, integer)
```

date에서 integer로 지정한 월을 가감한 날짜 값을 반환한다. date가 월말일이면 결과도 월말일로 계산된다.  

아래는 ADD_MONTH 함수를 사용한 쿼리다. c2 열은 2050-01-31에 1달을 더한 2050-02-28, c3 열은 2050-02-28에 1달을 더한 2050-03-31이 반환된다.  

```sql
SELECT ADD_MONTH(DATE '2050-01-15' . -1) AS c1,
       ADD_MONTH(DATE '2050-01-31' . 1) AS c2,
       ADD_MONTH(DATE '22050-02-28' . 1) AS c3
FROM DUAL;
```

월말일을 고려하지 않고 월을 가감해야 한다면 사용자 정의 함수를 사용해야 한다.  

```sql
CREATE OR REPLACE FUNCTION fnc_add_month(i_date IN DATE, i_integer IN NUMBER)
  RETURN DATE
IS
  l_date := ADD_MONTH(i_date, i_integer);
BEGIN
  RETURN CASE
              WHEN TO_CHAR(i_date, 'DD') < TO_CHAR(l_date, 'DD')
              THEN i_date + NUMTOYMINTERVAL(i_integer, 'MONTH')
              ELSE l_date
         END;
END fnc_add_month;
/
```

#### 6.3.5. MONTHS&#95;BETWEEN 함수
<br/>
```
MONTHS_BETWEEN(date1, date2)
```

date1과 date2 사이의 개월수를 반환한다. date1, date2의 일자(day)가 같거나 모두 월말일이면 정수, 그렇지 않으면 날짜의 차이를 31로 나눈 값을 반환한다. date2가 date1보다 크면 음수를 반환한다.  

아래는 MONTHS_BETWEEN 함수를 사용한 쿼리다.  

```sql
SELECT MONTHS_BETWEEN(DATE '2050-04-15', DATE '2050-01-15') AS c1,
       MONTHS_BETWEEN(DATE '2050-04-30', DATE '2050-01-31') AS c2,
       MONTHS_BETWEEN(DATE '2050-04-30', DATE '2050-01-15') AS c3,
       MONTHS_BETWEEN(DATE '2050-04-15', DATE '2050-04-30') AS c4
FROM DUAL;
```

#### 6.3.6. EXTRACT 함수
<br/>
```
EXTRACT({YEAR | MONTH | DAY | HOUR | MINUTE | SECOND} FROM expr)
```

expr에서 날짜 정보를 추출한다.  

아래는 EXTRACT 함수를 사용한 쿼리다. 결과가 NUMBER 타입으로 반환된다.  

```sql
WITH w1 AS (SELECT TIMESTAMP '2050-01-02 12:34:56.789' AS dt FROM DUAL)
SELECT EXTRACT(YEAR FROM dt) AS c1, EXTRACT(MONTH FROM dt) AS c2,
       EXTRACT(DAY FROM dt) AS c3, EXTRACT(HOUR FROM dt) AS c4,
       EXTRACT(MINUTE FROM dt) AS c5, EXTRACT(SECOND FROM dt) AS c6
FROM w1;
```

#### 6.3.7. ROUND(date) 함수
<br/>
```
ROUND(date, [, fmt])
```

fmt를 기준으로 date를 반올림한다. fmt의 기본값은 DD다.  

아래는 ROUND 함수를 사용한 쿼리다.  

```sql
WITH w1 AS (SELECT TO_DATE ('20510816123131', 'YYYYMMDDHH24MISS') AS dt FROM DUAL)
SELECT ROUND(dt) AS c1, ROUND(dt, 'DD') AS c2 FROM w1;
```

<table>
  <thead>
    <tr>
      <td>포맷 요소</td>
      <td>설명</td>
      <td>ROUND 기준</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>CC</td>
      <td>세기</td>
      <td>연도의 끝 두 자리 51년</td>
    </tr>
    <tr>
      <td>YY</td>
      <td>년</td>
      <td>7월 1일</td>
    </tr>
    <tr>
      <td>Q</td>
      <td>분기</td>
      <td>분기의 둘째 달 16일</td>
    </tr>
    <tr>
      <td>MM</td>
      <td>월</td>
      <td>16일</td>
    </tr>
    <tr>
      <td>DD</td>
      <td>일</td>
      <td>12시</td>
    </tr>
    <tr>
      <td>HH</td>
      <td>시</td>
      <td>31분</td>
    </tr>
    <tr>
      <td>MI</td>
      <td>분</td>
      <td>31초</td>
    </tr>
    <tr>
      <td>D</td>
      <td>주</td>
      <td>일요일</td>
    </tr>
    <tr>
      <td>WW</td>
      <td>주</td>
      <td>연도의 첫째 요일과 같은 그 주의 날짜</td>
    </tr>
    <tr>
      <td>W</td>
      <td>주</td>
      <td>당월의 첫째 요일과 같은 그 주의 날짜</td>
    </tr>
    <tr>
      <td>IY</td>
      <td>년(ISO 기준)</td>
      <td>7월 1일</td>
    </tr>
    <tr>
      <td>IW</td>
      <td>주(ISO 기준)</td>
      <td>연도의 첫째 요일과 같은 그 주의 날짜</td>
    </tr>
  </tbody>
</table>
<br/><br/>

ISO 기준의 년과 주에 관련된 포맷 요소는 연초일이 7일 중 4일 이상이 포함된 주의 월요일이다.

#### 6.3.8. TRUNC(date) 함수
<br/>
```
TRUNC(date, [, fmt])
```

fmt를 기준으로 date를 버린다. fmt는 ROUND (date) 함수와 동일하다.  

```sql
WITH w1 AS (SELECT TO_DATE ('20510816123131', 'YYYYMMDDHH24MISS') AS dt FROM DUAL)
SELECT TRUNC(dt) AS c1, TRUNC(dt, 'DD') AS c2 FROM w1;
```

#### 6.3.9. 소수점 이하 초의 유실
<br/>
DATE 타입을 반환하는 날짜 함수에 TIMESTAMP 값을 사용하면 소수점 이하 초가 유실된다.  

### 6.4. 변환 함수
<br/>
#### 6.4.1. 데이터 변환
<br/>
+ 명시적 데이터 변환(explicit data conversion)  
사용자가 변환 함수를 통해 값의 데이터 타입을 변경하는 것
+ 암시적 데이터 변환(implicit data conversion)  
값의 데이터 타입이 오라클 데이터베이스에 의해 자동으로 변환되는 것  

아래 표는 데이터 변환이 가능한 조합이다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>CHAR</td>
      <td>VARCHAR2</td>
      <td>DATE</td>
      <td>INTERVAL</td>
      <td>NUMBER</td>
      <td>CLOB</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>CHAR</td>
      <td></td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>VARCHAR2</td>
      <td>Y</td>
      <td></td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>DATE</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>INTERVAL</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>NUMBER</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>CLOB</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래는 데이터 타입이 변환되는 우선순위다. 정확한 결과를 위해 에러가 발생할 수 있는 데이터 타입으로 값을 변환하는 방식이다.  

(1) DATE, INTERVAL  
(2) NUMBER  
(3) CHAR, VARCHAR2, CLOB  
(4) 기타  

#### 6.4.2. TO&#95;CHAR(number) 함수
<br/>
```
TO_CHAR(n, [, fmt [, 'nlsparam']])
```

expr을 fmt 형식의 문자 값으로 변환한다.

아래는 자주 사용되는 포맷 요소다.  

+ 0 : 앞쪽이나 뒷쪽에 0을 출력
+ 9 : 한자리 숫자
+ , : 구분자
+ . : 소수점
+ S : 부호(양수면 +, 음수면 -)
+ G : 구분자(Group seperator)(파라미터 : NLS&#95;NUMERIC&#95;CHARACTERS)
+ D : 소수점(Decimal character)(파라미터 : NLS&#95;NUMERIC&#95;CHARACTERS)
+ $ : 달러
+ L : 로컬 통화 기호(파라미터 : NLS&#95;CURRENCY)
+ U : 이중 통화 기호(파라미터 : NLS&#95;DUAL&#95;CURRENCY)
+ C : 국제 통화 기호(파라미터 : NLS&#95;ISO&#95;CURRENCY)
+ FM : 양 측 공백을 제거한 값을 반환(Fill Mode)

```sql
SELECT TO_CHAR(12, '0') AS c1, TO_CHAR(12, '00') AS c2,
       TO_CHAR(12, '000') AS c3, TO_CHAR(12, '9') AS c4,
       TO_CHAR(12, '99') AS c5, TO_CHAR(12, '999') AS c6
FROM DUAL;
```

아래는 S 포맷 요소를 사용한 쿼리다. 명시적으로 부호를 표시할 때 사용할 수 있다.

```sql
SELECT TO_CHAR(1, '9') AS c1, TO_CHAR(1, 'S9') AS c2,
       TO_CHAR(-1, '9') AS c3, TO_CHAR(-1, 'S9') AS c4
FROM DUAL;
```

아래는 구분자, 소수점 포맷 요소를 사용한 퀴리다. c1 열에서 결과가 반올림된 것을 확인할 수 있다.  

```sql
SELECT TO_CHAR(1234.5, '9.999') AS c1, TO_CHAR(1234.5, '9,990.00') AS c2
FROM DUAL;
```

아래는 G, D 포맷 요소를 사용한 쿼리다.  

```sql
SELECT TO_CHAR(1234.5, '999G999G990D00') AS c1 FROM DUAL;
```

아래는 통화와 관련된 포맷 요소를 사용한 쿼리다.  

```sql
SELECT TO_CHAR(1, '9$') AS c1, TO_CHAR(1, '9L') AS c2,
       TO_CHAR(1, '9U') AS c3, TO_CHAR(1, '9C') AS c4
FROM DUAL;
```

아래는 FM 포맷 한정자를 사용한 쿼리다. TO&#95;CHAR 함수로 숫자 값에 서식을 지정했다.  

```sql
SELECT TO_CHAR(1234.5, 'FM999,999,990.00') AS c1 FROM DUAL;
```

#### 6.4.3. TO&#95;CHAR(datetime) 함수
<br/>
```
TO_CHAR({datetime | interval} [, fmt [, 'nlsparam']])
```

+ -/,.;: : 문장부호(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ "text" : 텍스트  
+ YYYY : 년(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ MM : 월(01~12)(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ DD : 일(01~31)(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ HH : 시(12시간)(01~12)(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ HH24 : 시(24시간)(01~23)(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ MI : 분(00~59)(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ SS : 초(00~59)(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ FF[1..9] : 소수점 이하 초(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ AM, PM : 오전, 오후(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ Q : 연중 분기(1~4)
+ WW : 연중 주(1~53)
+ DDD : 연중 일자(1~365)(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ W : 월중 주(1~5)(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ SSSSS : 자정 이후 초(0~86399)(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ MONTH : 월 이름(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ MON : 월 약명(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ DAY : 요일 이름(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ DY : 요일 약명(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ D : 요일 숫자(TO&#95;DATE, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ 함수에서도 사용 가능)
+ IYYY : ISO 기준 연도
+ IW : ISO 기준 연중 주  

아래는 TO&#95;CHAR(datetime) 함수를 사용한 쿼리다.  

```sql
WITH w1 AS (SELECT TO_DATE('20500102123456', 'YYYYMMDDHH24MISS') AS dt FROM DUAL)
SELECT TO_CHAR(dt, 'YYYY') AS c1,
       TO_CHAR(dt, 'YYYYMM') AS c2,
       TO_CHAR(dt, 'YYYYMMDD') AS c3,
       TO_CHAR(dt, 'YYYYMMDDHH24MISS') AS c4,
       TO_CHAR(dt, 'YYYY-MM-DD HH24:MI:SS') AS c5,
       TO_CHAR(dt, 'HH24"H" MI"M" SS"S"') AS c6
FROM w1;
```

아래는 특정 시점 이후 기간을 표시하는 포맷 요소를 사용한 쿼리다.  

```sql
WITH w1 AS (SELECT TO_DATE('2050041512', 'YYYYMMDDHH24') AS dt FROM DUAL)
SELECT TO_CHAR(dt, 'YYYY-Q') AS c1, TO_CHAR(dt, 'YYYY-WW') AS c2,
       TO_CHAR(dt, 'YYYY-DDD') AS c3, TO_CHAR(dt, 'YYYYMM-W') AS c4,
       TO_CHAR(dt, 'SSSSS') AS c5
FROM w1;
```

아래 쿼리에서 c5 열을 제외한 나머지 열은 NLS 파라미터 값에 따라 결과가 달라질 수 있다.  

```sql
WITH w1 AS (SELECT DATE '2050-01-01' AS dt FROM DUAL)
SELECT TO_CHAR(dt, 'MON') AS c1, TO_CHAR(dt, 'MONTH') AS c2,
       TO_CHAR(dt, 'DY') AS c3, TO_CHAR(dt, 'DAY') AS c4,
       TO_CHAR(dt, 'D') AS c5
FROM w1;
```

#### 6.4.4. TO&#95;NUMBER 함수
<br/>
```
TO_NUMBER(expr, [, fmt [, 'nlsparam']])
```

fmt 형식의 expr을 숫자 값으로 변환한다.  

```sql
SELECT TO_NUMBER('+1,234.50', 'S999,990.00') AS c1 FROM DUAL;
```

#### 6.4.5. TO&#95;DATE 함수
<br/>
```
TO_DATE(char, [, fmt [, 'nlsparam']])
```

fmt 형식의 char를 DATE 값으로 변환한다. TO&#95;DATE 함수도 TO&#95;CHAR 함수처럼 반드시 포맷을 지정해야 한다. 포맷을 지정하지 않으면 NLS&#95;DATE&#95;FORMAT 파라미터에 따라 에러가 발생할 수 있다.  

아래는 TO&#95;DATE 함수를 사용한 쿼리다. c1 열처럼 년만 지정한 경우 현재 월의 월초일이 반환된다. c2 열처럼 연월만 지정한 경우 지정한 월의 월초일이 반환된다.  

```sql
SELECT TO_DATE(dt, 'YYYY') AS c1,
       TO_DATE(dt, 'YYYYMM') AS c2,
       TO_DATE(dt, 'YYYYMMDD') AS c3,
       TO_DATE(dt, 'YYYYMMDDHH24MISS') AS c4
FROM DUAL;
```

TO&#95;DATE 함수는 fmt이 정확하게 일치하지 않아도 데이터가 변환된다.  

```sql
SELECT TO_DATE('2050-1-2 3:4:5', 'YYYY-MM-DD HH24:MI:SS') AS c1 FROM DUAL;
```

+ FX : char와 fmt이 정확히 일치해야 함(Format eXact)  

FX 포맷 한정자를 사용하면 fmt이 정확히 일치하는 경우에만 데이터가 변환된다.  

아래는 FX 포맷 요소를 사용한 쿼리다. fmt이 정확히 일치하지 않기 때문에 에러가 발생한다. FX 포맷 요소는 엄격한 데이터 변환에 활용할 수 있다.  

```sql
SELECT TO_DATE('2050-01-02 13:14:15', 'FXYYYY-MM-DD HH24:MI:SS') AS c1 FROM DUAL;
```

#### 6.4.6. TO&#95;TIMESTAMP 함수
<br/>
```
TO_TIMESTAMP(char, [, fmt [, 'nlsparam']])
```

fmt 형식의 char를 TIMESTAMP 값으로 변환한다.  

아래는 TO&#95;TIMESTAMP 함수를 사용한 쿼리다.  

```sql
SELECT TO_TIMESTAMP('2050-01-02 12:34:56.789', 'YYYY-MM-DD HH24:MI:SS.FF3') AS c1 FROM DUAL;
```

#### 6.4.7. TO&#95;YMINTERVAL 함수
<br/>
```
TO_YMINTERVAL('{sql_format | ym_iso_format}')
```

문자 값을 YEAR TO MONTH 인터벌 값으로 변환한다.  

+ sql&#95;format : [+ | -]years-months
+ ym&#95;iso&#95;format : [-]P[years Y][months M]

아래는 TO&#95;YMINTERVAL 함수를 사용한 쿼리다. c1 열은 sql&#95;format, c2 열은 ym&#95;iso&#95;format을 사용했다.  

```sql
SELECT TO_YMINTERVAL('1-11') AS c1, TO_YMINTERVAL('P1Y11M') AS c2 FROM DUAL;
```

#### 6.4.8. TO&#95;DSINTERVAL 함수
<br/>
```
TO_DSINTERVAL('{sql_format | ym_iso_format}')
```

문자 값을 DAY TO SECOND 인터벌 값으로 변환한다.  

+ sql&#95;format : [+ | -]days-hours:minutes:seconds[.frac&#95;secs]
+ ds&#95;iso&#95;format : [-]P[days D][T[hours H][minutes M][seconds[.frac&#95;secs]S]]

아래는 TO&#95;DSINTERVAL 함수를 사용한 쿼리다. c1 열은 sql&#95;format, c2 열은 ym&#95;iso&#95;format을 사용했다.  

```sql
SELECT TO_DSINTERVAL('1 23:59:59.999999999') AS c1,
       TO_DSINTERVAL('P1DT23H59M59.999999999S') AS c2
FROM DUAL;
```

#### 6.4.9. NUMTOYMINTERVAL 함수
<br/>
```
NUMTOYMINTERVAL(n, 'interval_unit')
```

n을 YEAR TO MONTH 인터벌 값으로 변환한다.  

아래는 NUMTOYMINTERVAL 함수를 사용한 쿼리다.  

```sql
SELECT NUMTOYMINTERVAL(2, 'YEAR') AS c1, NUMTOYMINTERVAL(23, 'MONTH') AS c2
FROM DUAL;
```

#### 6.4.10. NUMTODSINTERVAL 함수
<br/>
```
NUMTODSINTERVAL(n, 'interval_unit')
```

n을 DAY TO SECOND 인터벌 값으로 변환한다.  

아래는 NUMTODSINTERVAL 함수를 사용한 쿼리다.  

```sql
SELECT NUMTODSINTERVAL(10, 'DAY') AS c1, NUMTODSINTERVAL(24, 'HOUR') AS c2,
       NUMTODSINTERVAL(60, 'MINUTE') AS c3, NUMTODSINTERVAL(60, 'SECOND') AS c4,
       NUMTODSINTERVAL(0.1, 'SECOND') AS c5
FROM DUAL;
```

#### 6.4.11. CAST 함수
<br/>
```
CAST (expr AS type_name [, fmt [, 'nlsparam']])
```

expr을 type&#95;name에 지정한 데이터 타입으로 변환한다. type&#95;name은 NUMBER, DATE, TIMESTAMP, TIMESTAMP WITH TIME ZONE, TIMESTAMP WITH LOCAL TIME ZONE을 지정할 수 있다. fmt는 12.2 버전부터 사용할 수 있다.  

아래는 CAST 함수를 사용한 쿼리다.  

```sql
SELECT CAST('123' AS VARCHAR2(5)) AS c1,
       CAST('123.456' AS NUMBER(6,3)) AS c2,
       CAST('2050-01-02' AS DATE) AS c3,
       CAST('2050-01-02 12:34:56' AS TIMESTAMP(2)) AS c4,
       CAST('2050-01-02 12:34:56 +08:00' AS TIMESTAMP(2) WITH TIME ZONE) AS c5
FROM DUAL;
```

아래는 fmt를 사용한 쿼리다.  

```sql
SELECT CAST('1,234.50' AS NUMBER(7,2), 'S999,990.00') AS c1 FROM DUAL;
```

#### 6.4.12. VALIDATE&#95;CONVERSION 함수
<br/>
```
VALIDATE_CONVERSION(expr AS type_name [, fmt [, 'nlsparam']])
```

expr이 type_name에 지정한 데이터 타입으로 변환될 수 있으면 1, 변환될 수 없으면 0을 반환한다. expr이 널이면 1을 반환한다. type&#95;name은 NUMBER, DATE, TIMESTAMP, TIMESTAMP WITH TIME ZONE, TIMESTAMP WITH LOCAL TIME ZONE, INTERVAL YEAR TO MONTH, INTERVAL DAY TO SECOND을 지정할 수 있다. VALIDATE&#95;CONVERSION 함수는 12.2 버전부터 사용할 수 있다.  

아래는 VALIDATE&#95;CONVERSION 함수를 사용한 쿼리다.  

```sql
SELECT VALIDATE_CONVERSION('20500131' AS DATE, 'YYYYMMDD') AS c1,
       VALIDATE_CONVERSION('20500132' AS DATE, 'YYYYMMDD') AS c2
FROM DUAL;
```

지금까지 살펴본 변환 함수 외에도 다양한 변환 함수를 사용할 수 있다.  

+ BIN&#95;TO&#95;NUM(expr [, expr]...) : 비트 백터를 해당하는 숫자로 변환
+ TO&#95;LOB(long&#95;column) : LONG 또는 LONG RAW 값을 LOB 값으로 변환
+ TO&#95;SINGLE&#95;BYTE(char) : 멀티 바이트 문자를 싱글 바이트 문자로 변환
+ TO&#95;MULTI&#95;BYTE(char) : 싱글 바이트 문자를 멀티 바이트 문자로 변환
+ ROWIDTOCHAR(char) : ROWID를 문자로 변환
+ CHARTOROWID(char) : 문자를 ROWID로 변환
+ SCN&#95;TO&#95;TIMESTAMP(number) : SCN을 TIMESTAMP 값으로 변환
+ TIMESTAMP&#95;TO&#95;SCN(timestamp) : TIMESTAMP 값을 SCN으로 변환  

#### 6.4.13. 변환 함수의 신규 기능  
<br/>
12.2 버전부터 변환 함수에 에러가 발생할 경우 반환할 return&#95;value을 지정할 수 있다. TO&#95;NUMBER, TO&#95;TIMESTAMP, TO&#95;TIMESTAMP&#95;TZ, TO&#95;DSINTERVAL, TO&#95;YMINTERVAL, CASE 함수에 사용할 수 있다.  

```
TO_*(char [DEFAULT return_value ON CONVERSION ERROR][, fmt [, 'nlsparam']])
```

아래는 DEFAULT return&#95;value ON CONVERSION ERROR 절을 사용한 쿼리다.  

```sql
SELECT TO_DATE('205012', DEFAULT '999912' ON CONVERSION ERROR, 'YYYYMM') AS c1,
       TO_DATE('205013', DEFAULT '999912' ON CONVERSION ERROR, 'YYYYMM') AS c2
FROM DUAL;
```

### 6.5. 널 관련 함수
<br/>
#### 6.5.1. NVL 함수
<br/>
```
NVL(expr1, expr2)
```

expr1이 널이 아니면 expr1, 널이면 expr2를 반환한다.  

아래는 NVL 함수를 사용한 쿼리다.  

```sql
SELECT NVL(1, 2) AS c1, NVL(NULL, 2) AS c2 FROM DUAL;
```

NVL 함수로 수치 연산의 널을 처리할 수 있다.  

```sql
SELECT ename, sql, comm, sal + comm AS c1, sal + NVL(comm, 0) AS c2 FROM emp;
```

#### 6.5.2. NV2 함수
<br/>
```
NVL2(expr1, expr2, expr3)
```

expr1이 널이 아니면 expr2, 널이면 expr3를 반환한다.  

아래는 NVL2 함수를 사용한 쿼리다.  

```sql
SELECT NVL(1, 2, 3) AS c1, NVL(NULL, 2, 3) AS c2 FROM DUAL;
```

#### 6.5.3. COALESCE 함수
<br/>
```
COALESCE(expr [, expr]...)
```

널이 아닌 첫 번째 expr을 반환한다. NVL 함수의 기능을 확장한 함수다.  

아래는 COALESCE 함수를 사용한 쿼리다.  

```sql
SELECT COALESCE(1, 2, 3) AS c1, COALESCE(NULL, 2, 3) AS c2,
       COALESCE(NULL, NULL, 3) AS c3
FROM DUAL;
```

#### 6.5.4. NULLIF 함수
<br/>
```
NULLIF(expr1, expr2)
```

expr1과 expr2가 다르면 expr1, 같으면 널을 반환한다.  

아래는 NULLIF 함수를 사용한 쿼리다.  

```sql
SELECT NULLIF(1, 1) AS c1, NULLIF(1, 2) AS c2 FROM DUAL;
```

### 6.6. 비교함수
<br/>
#### 6.6.1. LEAST 함수
<br/>
```
LEAST(expr [, expr]...)
```

expr 중 최소값을 반환한다. expr에 널이 입력되면 널을 반환한다.  

아래는 LEAST 함수를 사용한 쿼리다.  

```sql
SELECT LEAST(1, 2, 3) AS c1, LEAST('A', 'AB', 'ABC') AS c2, LEAST(1, NULL) AS c3
FROM DUAL;
```

#### 6.6.2. GREATEST 함수
<br/>
```
GREATEST(expr [, expr]...)
```

expr 중 최대값을 반환한다. expr에 널이 입력되면 널을 반환한다.  

아래는 LEAST 함수를 사용한 쿼리다.  

```sql
SELECT GREATEST(1, 2, 3) AS c1, GREATEST('A', 'AB', 'ABC') AS c2, GREATEST(1, NULL) AS c3
FROM DUAL;
```

LEAST 함수와 GREATEST 함수는 expr의 데이터 타입이 동일해야 한다. 데이터 타입이 다른 expr은 첫 번째 expr의 데이터 타입으로 변환된다.  

### 6.7. 엔코딩 디코딩 함수
<br/>
#### 6.7.1. DECODE 함수
<br/>
```
DECODE(expr, search, result, [, search, result] ... [, default])
```

expr과 search가 일치하면 result, 모두 일치하지 않으면 default를 반환한다. default를 지정하지 않으면 널을 반환한다.  

아래는 DECODE 함수를 사용한 쿼리다.  

```sql
SELECT DECODE(1, 1, 'A', 2, 'B', 'C') AS c1,
       DECODE(2, 1, 'A', 2, 'B', 'C') AS c2,
       DECODE(3, 1, 'A', 2, 'B', 'C') AS c3
FROM DUAL;
```

expr과 데이터 타입이 다른 search는 expr의 데이터 타입으로 변환된다.  

result와 default의 데이터 타입은 첫 번째 result의 데이터 타입과 동일해야 한다.  

DECODE 함수는 널을 평가할 수 있지만 NVL2 함수를 사용하는 편이 바람직하다.  

DECODE 함수가 중첩되면 쿼리의 가독성이 나빠지고 성능이 저하된다.  

결합 연산자(|)를 사용하면 DECODE 함수의 중첩을 제거할 수 있다.  

숫자 값이나 문자 값에 결합 연산자를 사용하면 암시적 데이터 변환이 발생할 수 있다. 결합 연산자보다 CASE 표현식을 사용하는 편이 바람직하다.  

#### 6.7.2. DUMP 함수
<br/>
```
DUMP(expr [, return_fmt [, start_position [, length]]])
```

return&#95;fmt은 아래와 같이 지정할 수 있다.

+ 8진수(8), 10진수(10), 16진수(16), 문자의 각 바이트(17)  

아래는 DUMP 함수를 사용한 쿼리다. 문자 값 ABC는 데이터 타입이 CHAR, 길이가 3, 캐릭터셋이 KO16MSWIN949, 16진수로 A는 41, B는 42, C는 43인 것을 확인할 수 있다.  

```sql
SELECT DUMP('ABC', 16) AS c1, DUMP('ABC', 1016, 2, 2) AS c2 FROM DUAL;
```

데이터 타입의 코드는 아래와 같다.

+ VARCHAR2(1)
+ NUMBER(2)
+ LONG(8)
+ DATE(12)
+ RAW(23)
+ LONG RAW(24)
+ ROWID(69)
+ CHAR(96)
+ BINARY&#95;FLOAT(100)
+ BINARY&#95;DOUBLE(101)
+ CLOB(112)
+ BLOB(113)
+ BFILE(114)
+ TIMESTAMP(180)
+ TIMESTAMP WITH TIME ZONE(181)
+ INTERVAL YEAR TO MONTH(182)
+ INTERVAL DAY TO SECOND(183)
+ UROWID(208)
+ TIMESTAMP WITH LOCAL TIME ZONE(231)

#### 6.7.3. VSIZE 함수
<br/>
```
VSIZE(expr)
```

expr의 바이트 크기를 반환한다.  

아래는 VSIZE 함수를 사용한 쿼리다. 열에 저장된 값의 실제 크기를 확인할 수 있다.  

```sql
SELECT VSIZE(12345) AS c1, VSIZE('ABC') AS c2,
       VSIZE('가나다') AS c3, VSIZE(SYSDATE) AS c4,
       VSIZE(SYSTIMESTAMP) AS c5, VSIZE(TRUNC(SYSTIMESTAMP)) AS c6
FROM DUAL;
```

#### 6.7.4. ORA&#95;HASH 함수
<br/>
```
ORA_HASH(expr, [, max_bucket [, seed_value]])
```

expr에 대한 해시 값을 생성한다. max&#95;bucket은 0 ~ 4294967295의 범위를 지정할 수 있으며 기본값은 4294967295다. seed&#95;value도 동일한 범위를 가지며 기본값은 0이다.  

아래는 ORA&#95;HASH 함수를 사용한 쿼리다.  

```sql
SELECT ORA_HASH(123) AS c1, ORA_HASH(123, 100) AS c2,
       ORA_HASH(123, 100, 100) AS c3
FROM DUAL;
```

#### 6.7.5. STANDARD&#95;HASH 함수
<br/>
```
STANDARD_HASH(expr, [, 'method'])
```

method에 지정한 해시 알고리즘으로 expr의 해시 값을 생성한다. method는 SHA1, SHA256, SHA384, SHA512, MD5 알고리즘을 지정할 수 있으며 기본값은 SHA1이다.  

아래는 STANDARD&#95;HASH 함수를 사용한 쿼리다.  

```sql
SELECT STANDARD_HASH(123, 'SHA256') AS c1 FROM DUAL;
```

#### 6.8 환경 식별자 함수
<br/>
##### 6.8.1. USER 함수
<br/>
로그인한 사용자의 이름을 반환한다.

```sql
SELECT USER AS c1 FROM DUAL;
```

##### 6.8.2. UID 함수
<br/>
로그인한 사용자의 ID을 반환한다.

```sql
SELECT UID AS c1 FROM DUAL;
```

##### 6.8.3. SYS&#95;GUID 함수
<br/>
```
SYS_GUID()
```

전역 고유 식별자를 16바이트 RAW 값으로 반환한다.

```sql
SELECT SYS_GUID() AS c1 FROM DUAL;
```

##### 6.8.4. USERENV 함수
<br/>
```
USERENV('parameter')
```

사용 가능한 parameter는 아래와 같다.  

+ ISDBA : DBA 권한 로그인 여부
+ CLIENT&#95;INFO : 클라이언트 정보
+ TERMINAL : 클라이언트 OS 식별자
+ LANG : 언어 약명
+ LANGUAGE : 언어 지역 및 데이터베이스 문자 집합  

아래는 USERENV 함수를 사용한 쿼리다.  

```sql
SELECT USERENV('ISDBA') AS c1, USERENV('CLIENT_INFO') AS c2,
       USERENV('TERMINAL') AS c3, USERENV('LANG') AS c4,
       USERENV('LANGUAGE') AS c5
FROM DUAL;
```

##### 6.8.5. SYS&#95;CONTEXT 함수
<br/>
```
SYS_CONTEXT('namespace', 'parameter' [, length])
```

namespace에 속한 parameter 값을 반환한다. 미리 정의되어 있는 USERENV 네임스페이스에 속한 파라미터는 다음과 같다.  

+ SERVER&#95;HOST : 클라이언트가 접속되어 있는 서버의 IP 주소
+ IP&#95;ADDRESS : 클라이언트가 접속되어 있는 기기의 IP 주소
+ DB&#95;UNIQUE&#95;NAME : 데이터베이스 이름 (DB&#95;UNIQUE&#95;NAME 초기화 파라미터)
+ DB&#95;NAME : 데이터베이스 이름 (DB&#95;NAME 초기화 파라미터)
+ DB&#95;DOMAIN : 데이터베이스 도메인 (DB&#95;DOMAIN 초기화 파라미터)
+ INSTANCE : 인스턴스 번호
+ INSTANCE&#95;NAME : 인스턴스 이름
+ HOST : 클라이언트 호스트
+ OS&#95; : 클라이언트 OS 사용자
+ TERMINAL : 클라이언트 OS 식별자
+ CLIENT&#95;PROGRAM&#95;NAME : 클라이언트 프로그램
+ LANG : 언어 약명
+ LANGUAGE : 언어와 지역, 데이터베이스 캐릭터셋
+ NLS&#95;CALENDAR : 달력
+ NLS&#95;CURRENCY : 통화
+ NLS&#95;DATE&#95;FORMAT : 날짜 형식
+ NLS&#95;DATE&#95;LANGUAGE : 날짜 언어
+ NLS&#95;SORT : 언어 정렬 기준
+ NLS&#95;TERRITORY : 지역
+ SERVICE&#95;NAME : 서비스 이름(설정 : 리스너)
+ MODULE : 프로그램 모듈(DMBS&#95;APPLICATIOHN&#95;INFO)
+ ACTION : 프로그램 모듈의 액션(DMBS&#95;APPLICATIOHN&#95;INFO)
+ CLIENT&#95;INFO : 클라이언트 정보(DMBS&#95;APPLICATIOHN&#95;INFO)
+ CLIENT&#95;IDENTIFIER : 클라이언트 식별자(설정 : DBMS&#95;SESSION)

##### 6.8.6. V$SQLFN&#42;뷰
<br/>
오라클 데이터베이스는 내장 SQL 함수와 관련된 동적 성능 뷰를 제공한다.  

V$SQLFN&#95;METADATA 뷰에서 SQL 함수에 대한 정보를 조회할 수 있다.  

```sql
SELECT func_id, name, minargs, maxargs, datatype, version
FROM v$sqlfn_metadata;
```

V$SQLFN&#95;ARG&#95;METADATA 뷰에서 SQL 함수의 인수에 대한 정보를 조회할 수 있다.  

```sql
SELECT func_id, argnum, datatype FROM v$sqlfn_arg_metadata;
```
