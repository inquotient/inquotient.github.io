---
title:  근사 쿼리
categories:
- Unkind_SQL
feature_text: |
  ## 25. 근사 쿼리
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

BI(Bisiness Intelligence) 시스템은 근사값을 통한 추세 분석의 비중이 높은 시스템이다. 근사 쿼리를 사용하면 BI 시스템의 조회 성능을 개선할 수 있다. 근사 쿼리는 12.1 버전부터 사용할 수 있다. 18.1 버전에서 근사 쿼리 관련 기능이 대폭 개선되었다.  

### 25.1. 근사 함수
<br/>
근사 쿼리는 근사 함수(approximate function)를 사용한다. APPROX&#95;COUNT&#95;DISTINCT 함수는 12.1 버전부터 사용할 수 있고, 나머지 함수는 12.2 버전부터 사용할 수 있다.  

#### 25.1.1. APPROX&#95;COUNT&#95;DISTINCT 함수
<br/>
APPROX&#95;COUNT&#95;DISTINCT 함수는 COUNT (DISTINCT expr) 표현식의 근사값을 반환한다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 AS
SELECT LPAD(CEIL(ROWNUM / 10), 10, '0') AS c1 FROM XMLTABLE('1 to 10000000');
```

쿼리의 수행 시간을 측정하기 위해 SET 명령어로 TIMING 시스템 변수를 ON으로 설정하자.  

```sql
SET TIMING ON
```

아래에서 좌측 쿼리는 APPROX&#95;COUNT&#95;DISTINCT 함수, 우측 쿼리는 COUNT (DISTINCT expr) 표현식을 사용했다. 좌측 쿼리에서 천만 건 기준 99.70%의 근사값이 반환된 것을 확인할 수 있다.  

```sql
-- 0.65초
SELECT APPROX_COUNT_DISTINCT(c1) AS c1 FROM t1;

-- 3.40초
SELECT COUNT(DISTINCT c1) AS c1 FROM t1;
```

#### 25.1.2. APPROX&#95;COUNT&#95;DISTINCT&#95;DETAIL 함수
<br/>
APPROX&#95;COUNT&#95;DISTINCT&#95;DETAIL 함수는 expr의 고유한 개수에 대한 집계 결과를 BLOB 값으로 반환한다.  

```
APPROX_COUNT_DISTINCT_DETAIL (expr)
```

#### 25.1.3. APPROX&#95;COUNT&#95;DISTINCT&#95;AGG 함수
<br/>
APPROX&#95;COUNT&#95;DISTINCT&#95;AGG 함수는 detail에 대한 집계 결과를 BLOB 값으로 반환한다. detail은 APPROX&#95;COUNT&#95;DISTINCT&#95;DETAIL 함수로 생성한다.  

```
APPROX_COUNT_DISTINCT_AGG (expr)
```

#### 25.1.4. TO&#95;APPROX&#95;COUNT&#95;DISTINCT 함수
<br/>
TO&#95;APPROX&#95;COUNT&#95;DISTINCT 함수는 detail에 대한 고유한 근사값을 숫자 값으로 변환한다.  

```
TO_APPROX_COUNT_DISTINCT (detail)
```

예제를 위해 아래와 같이 테이블을 생성하자. APPROX&#95;COUNT&#95;DISTINCT&#95;DETAIL 함수로 c1 열을 생성했다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 AS
SELECT TO_CHAR(hiredate, 'YYYY') AS yyyy, TO_CHAR(hiredate, 'MM') AS mm
     , APPROX_COUNT_DISTINCT_DETAIL(deptno) AS c1
FROM emp
GROUP BY TO_CHAR(hiredate, 'YYYY'), TO_CHAR(hiredate, 'MM');
```

아래는 t1 테이블을 조회한 결과다. c1 열이 BLOB 값으로 반환된다.  

```sql
SELECT * FROM t1;
```

아래 쿼리는 yyyy 열을 기준으로 c1 열을 집계한다. APPROX&#95;COUNT&#95;DISTINCT&#95;AGG 함수로 집계한 값을 TO&#95;APPROX&#95;COUNT&#95;DISTINCT 함수를 통해 숫자 값으로 변환했다.  

```sql
SELECT yyyy, TO_APPORX_COUNT_DISTINCT(APPROX_COUNT_DISTINCT_AGG(c1)) AS c1
FROM t1
GROUP BY yyyy;
```

아래 쿼리는 mm 열을 기준으로 c1 열을 집계한다. 집계 기준이 다르더라도 c1 열에 저장된 detail 값을 통해 expr의 고유한 근사값을 집계할 수 있다.  

```sql
SELECT mm, TO_APPORX_COUNT_DISTINCT(TO_APPORX_COUNT_AGG(c1)) AS c1
FROM t1
GROUP BY mm;
```

#### 25.1.5. APPORX&#95;PERCENTILE 함수
<br/>
APPORX&#95;PERCENTILE 함수는 expr에 해당하는 백분위 근사값을 반환한다. expr에 숫자 값이나 날짜 값을 사용할 수 있다.  

```
APPORX_PERCENTILE (expr [DETERMINISTIC] [, {'ERROR_RATE' | 'CONFIDENCE'}]) WITHIN GROUP (ORDER BY expr [DESC | ASC])
```

+ DETERMINISTIC : 변하지 않는 결정적 결과를 계산 (expr에 숫자 값만 가능)
+ ERROR_RATE : 오류 비율을 반환
+ CONFIDENCE : 오류 비율에 대한 신뢰 수준을 반환  

아래는 APPORX&#95;PERCENTILE 함수를 사용한 쿼리다.  

```sql
SELECT APPORX_PERCENTILE (0.5) WITHIN GROUP (ORDER BY sal) AS c1
     , APPORX_PERCENTILE (0.5) WITHIN GROUP (ORDER BY hiredate) AS c2
     , APPORX_PERCENTILE (0.5 DETERMINISTIC) WITHIN GROUP (ORDER BY sal) AS c3
     , APPORX_PERCENTILE (0.5 'ERROR_RATE') WITHIN GROUP (ORDER BY sal) AS c4
     , APPORX_PERCENTILE (0.5 'CONFIDENCE') WITHIN GROUP (ORDER BY sal) AS c5
FROM emp
WHERE deptno = 30;
```

#### 25.1.6. APPROX&#95;PERCENTILE&#95;DETAIL 함수
<br/>
APPROX&#95;PERCENTILE&#95;DETAIL 함수는 expr의 백분위 값에 대한 집계 결과를 BLOB 값으로 반환한다.  

```
APPROX_PERCENTILE_DETAIL (expr [DETERMINISTIC])
```

#### 25.1.7. APPROX&#95;PERCENTILE&#95;AGG 함수
<br/>
APPROX&#95;PERCENTILE&#95;AGG 함수는 detail에 대한 집계 결과를 BLOB 값으로 반환한다. detail은 APPROX&#95;PERCENTILE&#95;DETAIL 함수로 생성한다.  

```
APPROX_PERCENTILE_AGG (expr)
```

#### 25.1.7. TO&#95;APPROX&#95;PERCENTILE 함수
<br/>
TO&#95;APPROX&#95;PERCENTILE 함수는 detail에 대한 백분위 근사값을 지정한 데이터 타입 값으로 반환한다.  

```
TO_APPORX_PERCENTILE (detail, expr, 'datatype' [, {'DESC' | 'ASC' | 'ERROR_RATE' | 'CONFIDENCE'}])
```

예제를 위해 아래와 같이 테이블을 생성하자. APPROX&#95;PERCENTILE&#95;DETAIL 함수로 c1 열을 생성했다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 AS
SELECT TO_CHAR(hiredate, 'YYYY') AS yyyy, TO_CHAR(hiredate, 'MM') AS mm
     , APPROX_PERCENTILE_DETAIL(sal) AS c1
FROM emp
GROUP BY TO_CHAR(hiredate, 'YYYY'), TO_CHAR(hiredate, 'MM');
```

아래는 t1 테이블을 조회한 결과다. c1 열이 BLOB 값으로 반환된다.  

```sql
SELECT * FROM t1;
```

아래 쿼리는 yyyy 열을 기준으로 c1 열을 집계한다. APPROX&#95;PERCENTILE&#95;AGG 함수로 집계한 값을 TO&#95;APPROX&#95;PERCENTILE 함수에 지정한 데이터 타입 값으로 변환했다.  

```sql
SELECT yyyy, TO_APPROX_PERCENTILE(APPROX_PERCENTILE_AGG(c1), 0.5, 'NUMBER') AS c1
FROM t1
GROUP BY yyyy;
```

아래 쿼리는 mm 열을 기준으로 c1 열을 집계한다. 집계 기준이 다르더라도 c1 열에 저장된 detail 값을 통해 expr의 백분위 근사값을 집계할 수 있다.  

```sql
SELECT mm, TO_APPORX_PERCENTILE(APPORX_PERCENTILE_AGG(c1), 0.5, 'NUMBER') AS c1
FROM t1
GROUP BY mm;
```

#### 25.1.8. APPROX&#95;MEDIAN 함수
<br/>
APPROX&#95;MEDIAN 함수는 expr의 근사 중앙값을 반환한다. expr에 숫자 값이나 날짜 값을 사용할 수 있다.  

```
APPROX_MEDIAN (expr [DETERMINISTIC] [, {'ERROR_RATE' | 'CONFIDENCE'}])
```

아래는 APPROX&#95;MEDIAN 함수를 사용한 쿼리다.  

```sql
SELECT APPROX_MEDIAN(sal) AS c1
     , APPROX_MEDIAN(hiredate) AS c2
     , APPROX_MEDIAN(sal DETERMINISTIC) AS c3
     , APPROX_MEDIAN(sal DETERMINISTIC, 'ERROR_RATE') AS c4
     , APPROX_MEDIAN(sal DETERMINISTIC, 'CONFIDENCE') AS c5
FROM emp
WHERE deptno = 30;
```

### 25.2. 초기화 파라미터
<br/>
오라클 데이터베이스는 근사 쿼리와 관련된 세 가지 초기화 파라미터를 제공한다.  

V$SES&#95;OPTIMIZER&#95;ENV 뷰에서 옵티마이저와 관련된 파라미터의 세션 설정 값을 확인할 수 있다. 근사 쿼리와 관련된 파라미터는 기본적으로 비활성화되어 있다.  

```sql
SELECT name, isdefault, value
FROM v$ses_optimizer_env
WHERE sid = SYS_CONTEXT('USERENV', 'SID')
AND name IN ('approx_for_aggregation', 'approx_for_count_distinct', 'apporx_for_percentile');
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 AS SELECT ROWNUM AS c1 FROM XMLTABLE ('1 to 10000000');
```

아래 쿼리는 16.99초가 소요되었다.  

```sql
SELECT COUNT(DISTINCT c1) AS c1 FROM t1;
```

아래 쿼리는 6.67초가 소요되었다.  

```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY c1) AS c1 FROM t1;
```

#### 25.2.1 APPROX&#95;FOR&#95;AGGREGATION 파라미터
<br/>
APPROX&#95;FOR&#95;AGGREGATION 파라미터는 APPROX&#95;FOR&#95;COUNT&#95;DISTINCT, APPROX&#95;FOR&#95;PERCENTILE 파라미터 값을 함께 설정한다.  

```
ALTER SESSION SET APPROX_FOR_AGGREGATION = {TRUE | FALSE};
```

아래와 같이 APPROX&#95;FOR&#95;AGGREGATION 파라미터를 TRUE로 설정해보자.  

```sql
ALTER SESSION SET APPROX_FOR_AGGREGATION = TRUE;
```

V$SES&#95;OPTIMIZER&#95;ENV 뷰에서 파라미터 값이 변경된 것을 확인할 수 있다.  

```sql
SELECT name, isdefault, value
FROM v$ses_optimizer_env
WHERE sid = SYS_CONTEXT('USERENV', 'SID')
AND name IN ('approx_for_aggregation', 'approx_for_count_distinct', 'approx_for_percentile');
```

#### 25.2.2. APPROX&#95;FOR&#95;COUNT&#95;DISTINCT 파라미터
<br/>
APPROX&#95;FOR&#95;COUNT&#95;DISTINCT 파라미터를 TRUE로 설정하면 쿼리에 사용된 COUNT(DISTINCT expr) 표현식이 APPROX&#95;COUNT&#95;DISTINCT 함수로 동작한다. 0.65초가 소요되었다. 수행 시간이 이전(16.99초)보다 단축된 것을 확인할 수 있다.  

```sql
SELECT COUNT(DISTINCT c1) AS c1 FROM t1;
```

#### 25.2.3. APPROX&#95;FOR&#95;PERCENTILE 파라미터
<br/>
APPROX&#95;FOR&#95;PERCENTILE 파라미터를 설정하면 PERCENTILE&#95;CONT, PERCENTILE&#95;DISC 함수가 APPROX&#95;PERCENTILE 함수로 동작한다. DETERMINISTIC을 지정하며 결정적 결과로 집계된다.  

```
ALTER SESSION SET APPROX_FOR_PERCENTILE = {NONE | PERCENTILE_CONT | PERCENTILE_CONT DETERMINISTIC | PERCENTILE_DISC | PERCENTILE_DISC DETERMINISTIC | ALL | ALL DETERMINISTIC};
```

+ NONE : 기본값
+ PERCENTILE&#95;CONT [DETERMINISTIC] : PERCENTILE&#95;CONT 함수를 변환
+ PERCENTILE&#95;DISC [DETERMINISTIC] : PERCENTILE&#95;DISC 함수를 변환
+ ALL [DETERMINISTIC] : PERCENTILE&#95;CONT 함수와 PERCENTILE&#95;DISC 함수를 변환  

아래 쿼리의 PERCENTILE&#95;CONT 함수는 APPROX&#95;PERCENTILE 함수로 동작한다. 0.75초가 소요되었다. 수행 시간이 이전(6.67초)보다 단축된 것을 확인할 수 있다.  

```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY c1) AS c1 FROM t1;
```
