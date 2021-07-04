---
title: 집계 함수
categories:
- Unkind_SQL
feature_text: |
  ## 9. 집계 함수
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 9.1. 기본 함수
<br/>
#### 9.1.1. COUNT 함수
<br/>
```
COUNT({* | [DISTINCT | ALL] expr})
```

+ COUNT(&#42;) : 전체 행의 개수를 반환
+ COUNT(expr) : 널이 아닌 expr의 개수를 반환
+ COUNT(DISTINCT expr) : 널이 아닌 expr의 고유한 개수를 반환  

아래 퀴리에서 c1 열은 전체 행의 개수, c2 열은 comm의 개수, c3 열은 deptno의 고유한 개수를 반환한다. 전체 행이 하나의 행 그룹으로 처리되어 하나의 행이 반환된다.  

```sql
SELECT COUNT(*) AS c1, COUNT(comm) AS c2, COUNT(DISTINCT deptno) AS c3 FROM emp;
```

#### 9.1.2. MIN 함수
<br/>
```
MIN(expr)
```

expr의 최저 값을 반환한다.  

아래는 MIN 함수를 사용한 쿼리다. c3 열에서 널이 무시되는 것을 확인할 수 있다.  

```sql
SELECT MIN(ename) AS c1, MIN(hiredate) AS c2, MIN(comm) AS c3 FROM emp;
```

아래 쿼리에서 COUNT 함수를 사용한 c1 열은 0, MIN 함수를 사용한 c2 열은 널이 반환되었다. COUNT 함수 이외의 기본 함수는 expr이 모두 널이면 널을 반환한다.  

```sql
SELECT COUNT(comm) AS c1, MIN(comm) AS c2 FROM emp WHERE deptno = 10;
```

#### 9.1.3. MAX 함수
<br/>
```
MAX(expr)
```

expr의 최고 값을 반환한다.  

아래는 MAX 함수를 사용한 쿼리다.  

```sql
SELECT MAX(ename) AS c1, MAX(hiredate) AS c2, MAX(comm) AS c3 FROM emp;
```

집계 함수를 사용한 쿼리는 WHERE 절을 만족하는 행이 없더라도 하나의 행을 반환한다.  

#### 9.1.4. SUM 함수
<br/>
```
SUM([DISTINCT | ALL] expr)
```

expr의 합계 값을 반환한다. expr은 숫자 값만 입력할 수 있다.  

아래는 SUM 함수를 사용한 쿼리다.  

```sql
SELECT SUM(sal) AS c1, SUM(DISTINCT sal) AS c2, SUM(comm) AS c3 FROM emp;
```

#### 9.1.5. AVG 함수
<br/>
```
AVG([DISTINCT | ALL] expr)
```

expr의 평균 값을 반환한다.  

아래는 AVG 함수를 사용한 쿼리다. c3, c4 열의 값이 다르다. 평균 값에 대한 널 포함 여부는 업무에 따라 달라질 수 있다.  

```sql
SELECT AVG(sal) AS c1, AVG(DISTINCT sal) AS c2,
       AVG(comm) AS c3, AVG(NVL(comm, 0)) AS c4
FROM DUAL;
```

### 9.2. 통계 함수
<br/>
#### 9.2.1. STDDEV 함수
<br/>
```
STDDEV([DISTINCT | ALL] expr)
```

expr의 표준편차를 반환한다.  

아래는 STDDEV 함수를 사용한 쿼리다.  

```sql
SELECT STDDEV(sal) AS c1 FROM emp;
```

#### 9.2.2. VARIANCE 함수
<br/>
```
VARIANCE([DISTINCT | ALL] expr)
```

expr의 분산을 반환한다.  

아래는 VARIANCE 함수를 사용한 쿼리다.  

```sql
SELECT VARIANCE(sal) AS c1 FROM emp;
```

#### 9.2.3. STATS&#95;MODE 함수
<br/>
```
STATS_MODE(expr)
```

expr의 최빈값을 반환한다.  

아래는 VARIANCE 함수를 사용한 쿼리다.  

```sql
SELECT STATS_MODE(sal) AS c1 FROM emp;
```

이외에도 다양한 통계 함수를 사용할 수 있다.  

+ STDDEV&#95;POP : 모집단 표준편차
+ STDDEV&#95;SAMP : 누적 표본 표준편차
+ VAR&#95;POP : 모집단 분산
+ VAR&#95;SAMP : 표본 분산
+ COVAR&#95;POP : 모집단 공분산
+ COVAR&#95;SAMP : 표본 공분산
+ CORR : Pearson's 상관 계수
+ CORR&#95;&#42; : Spearman's rho 상관 계수, Kendall's tau-b 상관 계수
+ REGR&#95;&#42; : 선형회귀(Linear Regression)
+ STATS&#95;BINOMIAL&#95;TEST : 이항검정
+ STATS&#95;CROSSTAB : 교차분석
+ STATS&#95;F&#95;TEST : F 검정
+ STATS&#95;KS&#95;TEST : Kolmogorov-Smirnov 검정
+ STATS&#95;NW&#95;TEST : Mann Whitney 검정
+ STATS&#95;T&#95;TEST&#95;&#42; : T 검정
+ STATS&#95;WSR&#95;TEST : Wilcoxon Signed Ranks 검정

### 9.3. 순위 함수
<br/>
#### 9.3.1. RANK 함수
<br/>
```
RANK(expr, [, expr]...) WITHIN GROUP (ORDER BY expr, [, expr]...)
```

expr에 대한 순위를 반환한다. expr이 동일하면 동순위를 부여하고, 다음 순위는 동순위의 수만큼 건너뛴다.  

아래 쿼리에서 c1 열은 sal가 1500이므로 4위, c2 열은 sal가 1500, comm이 500이므로 5위가 반환된다.  

```sql
SELECT RANK(1500) WITHIN GROUP (ORDER BY sal) AS c1,
       RANK(1500, 500) WITHIN GROUP (ORDER BY sal, comm) AS c2
FROM emp
WHERE deptno = 30;
```

#### 9.3.2. DENSE&#95;RANK 함수
<br/>
```
DENSE_RANK(expr, [, expr]...) WITHIN GROUP (ORDER BY expr, [, expr]...)
```

expr에 대한 순위를 반환한다. 동순위가 존재하더라도 다음 순위를 이어서 부여한다.  

아래 쿼리의 c1 열은 4위가 아니라 3위를 반환한다.  

```sql
SELECT DENSE_RANK(1500) WITHIN GROUP (ORDER BY sal) AS c1,
       DENSE_RANK(1500, 500) WITHIN GROUP (ORDER BY sal, comm) AS c2
FROM emp
WHERE deptno = 30;
```

#### 9.3.3. CUME&#95;DIST 함수
<br/>
```
CUME_DIST(expr, [, expr]...) WITHIN GROUP (ORDER BY expr, [, expr]...)
```

expr의 누적 분포(cumulative distribution) 값을 반환한다. 누적 분포 값은 0 < y <= 1의 범위를 가진다.  

아래는 CUME_DIST 함수를 사용한 쿼리다.  

```sql
SELECT CUME_DIST(1500) WITHIN GROUP (ORDER BY sal) AS c1,
       CUME_DIST(1500, 500) WITHIN GROUP (ORDER BY sal, comm) AS c2
FROM emp
WHERE deptno = 30;
```

#### 9.3.4. PERCENT&#95;RANK 함수
<br/>
```
PERCENT_RANK(expr, [, expr]...) WITHIN GROUP (ORDER BY expr, [, expr]...)
```

expr의 백분위 순위 값을 반환한다. 백분위 순위 값은 0 <= y <= 1의 범위를 가진다.  

```sql
SELECT PERCENT_RANK(1500) WITHIN GROUP (ORDER BY sal) AS c1,
       PERCENT_RANK(1500, 500) WITHIN GROUP (ORDER BY sal, comm) AS c2
FROM emp
WHERE deptno = 30;
```

### 9.4 분포 함수
<br/>
#### 9.4.1. PERCENTILE&#95;CONT 함수
<br/>
```
PERCENTILE_CONT(expr) WITHIN GROUP (ORDER BY expr)
```

연속 분포 모델에서 expr에 해당하는 백분위 값을 반환한다. expr은 0 ~ 1의 범위를 지정할 수 있다.  

아래는 PERCENTILE_CONT 함수를 사용한 쿼리다. 연속 분포 모델을 가정하므로 c1 열은 1375(=(1250+1500)/2)를 반환한다. c2 열도 동일한 방식으로 계산된다.  

```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY sal) AS c1,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY hiredate) AS c2
FROM emp
WHERE deptno = 30;
```

#### 9.4.2. PERCENTILE&#95;DISC 함수
<br/>
```
PERCENTILE_DISC(expr) WITHIN GROUP (ORDER BY expr)
```

이산 분포 모델에서 expr에 해당하는 백분위 값을 반환한다. expr은 0 ~ 1의 범위를 지정할 수 있다.  

아래는 PERCENTILE_DISC 함수를 사용한 쿼리다. 이산 분포 모델을 가정하므로 c1 열은 1250과 1500 중 1250을 반환한다. c2 열도 동일한 방식으로 계산된다.  

```sql
SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY sal) AS c1,
       PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY hiredate) AS c2
FROM emp
WHERE deptno = 30;
```

#### 9.4.3. MEDIAN 함수
<br/>
```
MEDIAN(expr)
```

연속 분포 모형을 가정한 중앙값을 반환한다. PERCENTILE_CONT(0.5) 표현식과 결과가 동일하다.  

아래는 MEDIAN 함수를 사용한 쿼리다.  

```sql
SELECT MEDIAN(sal) AS c1 FROM emp WHERE deptno = 30;
```

### 9.5. 기타 함수
<br/>
#### 9.5.1. LISTAGG 함수
<br/>
```
LISTAGG(measure_expr [, 'delimiter'][listagg_overflow_clause]) WITHIN GROUP(order_by_clause)
```

measure_expr를 order_by_clause로 정렬한 후 delimiter로 구분하여 연결한 값을 반환한다. delimiter의 기본값은 널이다.  

아래는 LISTAGG 함수를 사용한 쿼리다.  

```sql
SELECT LISTAGG(ename, ',') WITHIN GROUP (ORDER BY ename) AS c1
FROM emp
WHERE deptno = 30;
```

연결된 문자열이 4000자보다 길면 에러가 발생한다.  

LISTAGG OVERFLOW 절을 사용하면 4000자 이상의 문자열을 처리할 수 있다. LISTAGG OVERFLOW 절은 12.2 버전부터 사용할 수 있다.  

+ ON OVERFLOW ERROR  
문자열이  4000자보다 길면 에러를 발생시킴(기본값)
+ ON OVERFLOW TRUNCATE  
문자열이 4000자보다 길면 문자열을 잘라냄
+ truncation-indicator  
줄임 기호를 지정함(기본값은 ...)
+ WITH COUNT  
잘려진 문자수를 표시함(기본값)
+ WITHOUT COUNT  
잘려진 문자수를 표시하지 않음  

아래는 LISTAGG OVERFLOW 절을 사용한 쿼리다. 문자열이 '...(31)'을 포함한 4000자로 잘린다.  

```sql
WITH w1 AS (SELECT LEVEL AS c1, 'X' AS c2 FROM DUAL CONNECT BY LEVEL <= 4001)
SELECT LISTAGG (c2 ON OVERFLOW TRUNCATE) WITHIN GROUP (ORDER BY c1) AS c1 FROM w1;
```

### 9.6. KEEP 키워드
<br/>
```
aggregate_function KEEP (DENSE_RANK {FIRST | LAST} ORDER BY expr)
```
KEEP 키워드를 사용하면 행 그룹의 최저 또는 최고 순위 행으로 집계를 수행할 수 있다. 기본 함수와 일부 통계 함수에 사용할 수 있다.  

+ DENSE&#95;RANK FIRST  
정렬된 행 그룹에서 최저 순위 행을 지정
+ DENSE&#95;RANK LAST  
정렬된 행 그룹에서 최고 순위 행을 지정  

아래는 KEEP 절을 사용한 쿼리다. c1, c2 열은 최저 순위 행, c3, c4 열은 최고 순위 행을 각각 MIN, MAX 함수로 집계한다.  

```sql
SELECT MIN(hiredate) KEPP (DENSE_RANK FIRST ORDER BY sal) AS c1,
       MAX(hiredate) KEPP (DENSE_RANK FIRST ORDER BY sal) AS c2,
       MIN(hiredate) KEPP (DENSE_RANK LAST ORDER BY sal) AS c3,
       MAX(hiredate) KEPP (DENSE_RANK LAST ORDER BY sal) AS c4
FROM emp
WHERE deptno = 20;
```
