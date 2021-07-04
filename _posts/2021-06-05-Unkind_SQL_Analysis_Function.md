---
title: 분석 함수
categories:
- Unkind_SQL
feature_text: |
  ## 14. 분석 함수
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

분석 함수는 집계 함수의 확장 기능으로 생각할 수 있다. 집계 함수는 행 그룹으로 값을 집계하고, 분석 함수는 파티션과 윈도우로 값을 집계한다. 집계 함수는 행 그룹 별로 단일 행을 반환하기 때문에 데이터 집합이 변경되지만, 분석 함수는 데이터 집합을 변경하지 않고 값을 집계한다. 데이터 집합이 변경되지 않기 때문에 원본 값과 집계 값을 함께 분석할 수 있다.  

### 14.1 기본 문법
<br/>
분석 함수는 OVER 키워드를 사용한다. 집계 함수에 OVER 키워드를 기술하면 분석 함수로 동작한다. OVER 키워드에 ANALYTIC 절을 기술할 수 있다.  

```
analytic_function ([arguments]) OVER (analytic_clause)
```

ANALYTIC 절은 QUERY PARTITION 절, OEDER BY 절, WINDOWING 절로 구성된다.  

```
[query_partition_clause][order_by_clause [windowing_clause]]
```

#### 14.1.1. QUERY PARTITION 절
<br/>
```
PARTITION BY expr [, expr]...
```

expr로 파티션을 지정할 수 있다. QUERY PARTITION 절은 GROUP BY 절과 유사하게 동작한다. 파티션은 행 그룹과 유사하다. 분석을 위한 정적(static) 그룹으로 생각할 수 있다. QUERY PARTITION 절을 생략하면 전체 행이 하나의 파티션으로 동작한다.  

아래 쿼리의 c1 열은 job을 파티션으로 지정했다. sal가 3개의 파티션(CLERK, MANAGER, SALESMAN)으로 집계되어 각각 950, 2850, 5600을 반환한다. c2 열은 파티션을 지정하지 않았다. 전체 행이 하나의 파티션으로 처리되어 9400이 반환된다.  

```sql
SELECT empno, job, sal
       , SUM(sal) OVER(PARTITION BY job) AS c1
       , SUM(sal) OVER() AS c2
FROM emp
WHERE deptno = 30
ORDER BY 2, 1;
```

#### 14.1.2. ORDER BY 절
<br/>
```
ORDER BY expr [ASC | DESC] [NULLS FIRST | NULLS LAST] [, expr [ASC | DESC] [NULLS FIRST | NULLS LAST]]...
```

아래 쿼리의 c1 열은 sal의 오름차순 정렬에 대한 누적 합계 값을 반환한다.  

```sql
SELECT empno, sal, SUM(sal) OVER(ORDER BY sal, empno) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 2, 1;
```

#### 14.1.3. WINDOWINNG 절
<br/>
```
{ROWS | RANGE}
{BETWEEN {UNBOUNDED PRECEDING | CURRENT ROW | value_expr {PRECEDING | FOLLOWING} }
     AND {UNBOUNDED PRECEDING | CURRENT ROW | value_expr {PRECEDING | FOLLOWING} }
| {UNBOUNDED PRECEDING | CURRENT ROW | value_expr PRECEDING} }
```

WINDOWING 절로 파티션의 윈도우를 지정할 수 있다. 윈도우는 현재 행에 따라 범위가 변경될 수 있다. 윈도우는 파티션 내의 동적(dynamic) 그룹으로 생각할 수 있다.  

윈도우는 ROWS 방식이나 RANGE 방식으로 지정할 수 있다. 동작 방식에 아래와 같은 차이가 있다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>윈도우 기준</td>
      <td>동일한 정렬 값</td>
      <td>value&#42;expr</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ROWS</td>
      <td>물리적 행</td>
      <td>다른 값을 반환</td>
      <td>정렬 행의 위치</td>
    </tr>
    <tr>
      <td>RANGE</td>
      <td>논리적 범위</td>
      <td>같은 값을 반환</td>
      <td>정렬 값</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래는 윈도우의 범위를 지정하는 키워드다.  

+ BETWEEN ... AND ... : 윈도우의 시작과 끝
+ UNBOUNDED PRECEDING : 앞쪽 끝
+ UNBOUNDED FOLLOWING : 뒤쪽 끝
+ CURRENT ROW : 현재 행
+ ROWS ... value&#42;expr PRECEDING : 현재 행에서 앞쪽으로 value&#42;expr만큼 이동
+ RANGE ... value&#42;expr PRECEDING : 현재 값에서 value&#42;expr을 가감
+ ROWS ... value&#42;expr FOLLOWING : 현재 행에서 뒤쪽으로 value&#42;expr만큼 이동
+ RANGE ... value&#42;expr FOLLOWING : 현재 값에서 value&#42;expr을 가감  

RANGE 방식에 value&#42;expr를 사용하면 현재 값에서 value&#42;expr을 가감한다.  

윈도우의 범위를 지정하는 키워드를 조합하면 아래의 15개 방식으로 윈도우의 범위를 지정할 수 있다. 두 가지 윈도우 방식을 고려하면 30개의 조합이 가능하다.  

(1) BETWEEN UNBOUNDED PRECEDING AND value&#42;expr PRECEDING  
(2) BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW (13번과 동일)  
(3) BETWEEN UNBOUNDED PRECEDING AND value&#42;expr FOLLOWING  
(4) BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING  
(5) BETWEEN value&#42;expr PRECEDING AND value&#42;expr PRECEDING  
(6) BETWEEN value&#42;expr PRECEDING AND CURRENT ROW (14번과 동일)  
(7) BETWEEN value&#42;expr PRECEDING AND value&#42;expr FOLLOWING  
(8) BETWEEN value&#42;expr PRECEDING AND UNBOUNDED FOLLOWING  
(9) BETWEEN CURRENT ROW AND value&#42;expr FOLLOWING  
(10) BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING  
(11) BETWEEN value&#42;expr FOLLOWING ROW AND value&#42;expr FOLLOWING  
(12) BETWEEN value&#42;expr FOLLOWING ROW AND UNBOUNDED FOLLOWING  
(13) UNBOUNDED PRECEDING  
(14) value&#42;expr PRECEDING  
(15) CURRENT ROW  

#### 14.1.4 KEEP 키워드
<br/>
```
analytic_function ([arguments]) KEEP (DENSE_RANK {FIRST|LAST} ORDER BY expr) [OVER ([query_partition_clause])]
```

분석 함수도 KEEP 키워드를 사용할 수 있다. KEEP 키워드를 사용하면 ANALYTIC 절에서 QUERY PARTITION 절만 사용할 수 있다.  

아래 쿼리는 job 파티션 별 sal가 최소인 행을 대상으로 comm의 최고 값을 조회한다. SALESMAN 파티션의 경우 sal가 1250인 WARD와 MARTIN이 집계 대상이다. 해당 행에서 comm의 최고 값인 1400이 반환된다.  

```sql
SELECT empno, job, sal, comm
       , MAX(comm) KEEP (DENSE_RANK FIRST ORDER BY sal) OVER (PARTITION BY job) c1
FROM emp
WHERE deptno = 30
ORDER BY 2, 3, 4;
```

#### 14.1.5. 제약 사항
<br/>
RANGE 방식에 value&#42;expr을 지정하면 ORDER BY 절에 숫자 값이나 날짜 값을 사용해야 한다. 문자 값을 사용하면 값을 계산할 수 없기 때문에 에러가 발생한다.  

RANGE 방식에 value&#42;expr을 지정하면 정렬 표현식을 1개만 사용할 수 있다. 다수의 정렬 표현식을 사용하면 에러가 발생한다.  

아래 쿼리의 c1 열은 현재부터 직전 90일, c2 열은 현재부터 직전 3개월의 기간에 대한 sal의 합계 값을 계산한다. 정렬 표현식이 날짜 값인 경우 value&#42;expr에 숫자 값과 인터벌 값을 사용할 수 있다.  

```sql
SELECT hiredate, sal
       , SUM(sal) OVER (ORDER BY hiredate RANGE 90 PRECEDING) AS c1
       , SUM(sal) OVER (ORDER BY hiredate RANGE INTERVAL '3' MONTH PRECEDING) AS c2
FROM emp
WHERE deptno = 30
ORDER BY 1;
```

분석함수는 SELECT 절과 ORDER BY 절에 사용할 수 있다.  

아래와 같이 인라인 뷰를 사용하면 WHERE 절에서 분석 함수의 결과 값을 사용할 수 있다.  

```sql
SELECT deptno, ename, sal, c1
FROM (SELECT a.*, SUM(a.sal) OVER(PARTITION BY a.deptno) AS c1 FROM emp a)
WHERE c1 >= 10000
ORDER BY 1, 2;
```

아래 쿼리의 c2, c3 열에 사용한 분석 함수는 그룹핑이 완료된 후 수행된다.  

```sql
SELECT deptno
       , SUM(sal) AS c1, SUM(SUM(sal)) OVER() AS c2, COUNT(*) OVER() AS c3
FROM emp
GROUP BY deptno
ORDER BY 1;
```

아래와 같이 인라인 뷰를 통해 집계 쿼리와 분석 함수를 분리하는 편이 가독성 측면에서 바람직하다.  

```sql
SELECT deptno, SUM(c1) AS c1, COUNT(*) OVER() AS c3
FROM (SELECT deptno, SUM(sal) OVER(PARTITION BY deptno) AS c1 FROM emp GROUP BY deptno)
ORDER BY 1;
```

아래의 쿼리는 deptno별 sal의 합계 값을 집계한다. 분석 함수로 결과 집합을 생성한 후 중복 값을 제거하기 위해 DISTINCT 키워드를 사용했다.  

```sql
SELECT DISTINCT deptno, SUM(sal) OVER (PARTITION BY deptno) AS c1
FROM emp;
```

행 그룹으로 집계할 경우에는 분석 함수가 아닌 GROUP BY 절을 사용해야 한다.  

### 14.2. 분석 함수
<br/>
오라클 데이터베이스는 다양한 분석 함수를 제공한다. 분석 함수의 대부분의 집계 함수에 OVER 키워드를 키술하는 방식으로 사용한다.  
#### 14.2.1. 기본 함수
<br/>
기본 함수는 자주 사용하는 분석 함수다.  
##### 14.2.1.1. COUNT 함수
<br/>
```
COUNT ( {* | [DISTINCT | ALL] expr} ) OVER (analytic_clause)
```

전체 행의 개수나 expr의 개수를 반환한다.  

아래는 COUNT 함수를 사용한 쿼리다. c1 열은 job별 행의 개수를 반환한다.  

```sql
SELECT job, COUNT(*) OVER(PARTITION BY job) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 1;
```

##### 14.2.1.2. MIN 함수
<br/>
```
MIN(expr) OVER(analytic_clause)
```

expr의 최저 값을 반환한다.  

아래는 MIN 함수를 사용한 쿼리다. c1 열은 comm의 최저 값을 반환한다. PK인 empno 열을 ORDER BY 절에 지정했다. ROWS 방식은 정렬 값이 고유하지 않으면 결과가 변경될 수 있으므로 고유한 정렬 값을 지정해야 한다.  

```sql
SELECT empno, sal, comm
       , MIN(comm) OVER(ORDER BY sal, empno ROWS UNBOUNDED PRECEDING) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 2, 1;
```

##### 14.2.1.3. MAX 함수
<br/>
```
MAX(expr) OVER(analytic_clause)
```

expr의 최고 값을 반환한다.  

아래는 MAX 함수를 사용한 쿼리다. c1 열은 comm의 최고 값을 반환한다. RANGE 방식을 사용했기 때문에 sal 1250인 행의 c1 값이 동일하다.  

```sql
SELECT empno, sal, comm, MAX(comm) OVER(ORDER BY sal) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 2, 1;
```

##### 14.2.1.4. SUM 함수
<br/>
```
SUM([DISTINCT | ALL] expr) OVER(analytic_clause)
```

expr의 합계 값을 반환한다.  

아래는 SUM 함수를 사용한 쿼리다. c1 열은 전체 합계 값, c2 열은 누적 합계 값, c3 열은 누적 곱을 반환한다.  

```sql
SELECT empno, sal
       , SUM(sal) OVER() AS c1
       , SUM(sal) OVER(ORDER BY sal, empno) AS c2
       , EXP(SUM(LN(sal))) OVER(ORDER BY sal, empno) AS c3
FROM emp
WHERE deptno = 30
ORDER BY 2, 1;
```

##### 14.2.1.5. AVG 함수
<br/>
```
AVG([DISTINCT | ALL] expr) OVER(analytic_clause)
```

아래는 AVG 함수를 사용한 쿼리다. c1 열은 ROWS 방식을 사용해 1행 전후의 이동 평균(Moving Average, MA) 값을 반환한다. c2 열은 윈도우에 속한 행의 개수를 반환한다. JAMES는 앞쪽 행, BLAKE는 뒤쪽 행이 없기 때문에 윈도우에 2개의 행만 존재한다.

```sql
SELECT empno, ename, sal
       , AVG(sal) OVER(ORDER BY sal, empno ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS c1
       , COUNT(*) OVER(ORDER BY sal, empno ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS c2
FROM emp
WHERE deptno = 30
ORDER BY 3, 1;
```

아래 쿼리는 c1 열에 RANGE 방식을 지정했다. 윈도우의 범위는 sal에서 300을 뺀 값과 300을 더한 값 사이다. JAMES는 윈도우의 범위가 650 ~ 1250이므로 윈도우에 3개의 행이 존재한다.  

```sql
SELECT empno, ename, sal
       , AVG(sal) OVER(ORDER BY sal, empno RANGE BETWEEN 300 PRECEDING AND 300 FOLLOWING) AS c1
       , COUNT(*) OVER(ORDER BY sal, empno RANGE BETWEEN 300 PRECEDING AND 300 FOLLOWING) AS c2
FROM emp
WHERE deptno = 30
ORDER BY 3, 1;
```

#### 14.2.2. 통계 함수
<br/>
##### 14.2.2.1. STDDEV 함수
<br/>
```
STDDEV([DISTINCT | ALL] expr) OVER(analytic_clause)
```

expr의 표준편차를 반환한다.  

아래는 STDDEV 함수를 사용한 쿼리다.  

```sql
SELECT job, sal, STDDEV(sal) OVER(PARTITION BY job) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 2, 1;
```

##### 14.2.2.2. VARIANCE 함수
<br/>
```
VARIANCE([DISTINCT | ALL] expr) OVER(analytic_clause)
```

expr의 분산을 반환한다.  

아래는 VARIANCE 함수를 사용한 쿼리다.  

```sql
SELECT job, sal, VARIANCE(sal) OVER(PARTITION BY job) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 2, 1;
```

이외에도 아래의 통계 함수를 분석 함수로 사용할 수 있다.  

+ STDDEV&#95;POP : 모집단 표준편차
+ STDDEV&#95;SAMP : 누적 표본 표준편차
+ VAR&#95;POP : 모집단 분산
+ VAR&#95;SAMP : 표본 분산
+ COVAR&#95;POP : 모집단 공분산
+ COVAR&#95;SAMP : 표본 공분산
+ CORR : Pearson's 상관 계수
+ REGR&#95;&#42; : 선형회귀(Linear Regression)  

#### 14.2.3. 순위 함수
<br/>
순위 함수는 정렬 조건에 따른 순위를 반환한다. 순위 집계 함수는 가상의 행을 생성하지만, 순위 분석 함수는 실제의 행으로 값을 계산한다.  
##### 14.2.3.1. RANK 함수
<br/>
```
RANK() OVER([query_partition_clause] order_by_clause)
```

order_by_clause에 따른 순위를 반환한다. expr이 동일하면 동순위를 부여하고, 다음 순위는 동순위의 개수만큼 건너뛴다.  

아래는 RANK 함수를 사용한 쿼리다. WARD와 MARTIN은 sal가 동일하므로 동순위인 2위가 부여된다. 다음 순위인 TURNER는 3위를 건너뛰고 4위가 부여된다.  

```sql
SELECT empno, ename, sal, RANK() OVER(ORDER BY sal) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 3, 1;
```

##### 14.2.3.2. DENSE&#95;RANK 함수
<br/>
```
DENSE_RANK() OVER([query_partition_clause] order_by_clause)
```

order_by_clause에 따른 순위를 반환한다. 동순위가 존재하더라도 다음 순위를 이어서 부여한다.  

아래는 DENSE_RANK 함수를 사용한 쿼리다. WARD와 MARTIN은 sal가 동일하므로 동순위인 2위가 부여된다. 다음 순위인 TURNER는 3위가 부여된다.  

##### 14.2.3.3. ROW&#95;NUMBER 함수
<br/>
```
ROW_NUMBER() OVER([query_partition_clause] order_by_clause)
```
order_by_clause에 따른 고유한 순번을 반환한다. 정렬 값이 동일하더라도 다른 순번을 부여하기 때문에 결과 값이 변경될 수 있다.  

아래는 ROW_NUMBER 함수를 사용한 쿼리다. WARD와 MARTIN은 sal가 1250으로 동일하기 때문에 c1 값이 달라질 수 있다. c2 열처럼 고유한 정렬 값을 지정해야 한다.  

```sql
SELECT empno, ename, sal
       , ROW_NUMBER() OVER(ORDER BY sal) AS c1
       , ROW_NUMBER() OVER(ORDER BY sal, empno) AS c2
FROM emp
WHERE deptno = 30
ORDER BY 3, 1;
```

##### 14.2.3.4. NTILE 함수
<br/>
```
NTILE(expr) OVER([query_partition_clause] order_by_clause)
```

order_by_clause에 따라 행을 정렬하고, expr의 개수만큼 버킷을 생성한 후, 행에 해당하는 버킷 번호를 할당한다.  

아래는 NTILE 함수를 사용한 쿼리다. c2 열은 expr을 2로 지정했기 때문에 sal가 950 ~ 1250인 행은 1번 버킷, 1500 ~ 2850인 행은 2번 버킷에 속한다. c4 열은 나머지가 2(=6/4/)이므로 앞쪽 2개의 버킷에 행이 더 할당되고, c5 열은 나머지가 1(=6/5)이므로 앞쪽 1개의 버킷에 행이 더 할당된다. c6, c7 열은 expr이 행의 개수 이상이므로 순번이 부여된다.  

```sql
SELECT sal
       , NTILE(1) OVER(ORDER BY sal) AS c1, NTILE(2) OVER(ORDER BY sal) AS c2
       , NTILE(3) OVER(ORDER BY sal) AS c3, NTILE(4) OVER(ORDER BY sal) AS c4
       , NTILE(5) OVER(ORDER BY sal) AS c5, NTILE(6) OVER(ORDER BY sal) AS c6
       , NTILE(7) OVER(ORDER BY sal) AS c7
FROM emp
WHERE deptno = 30;
ORDER BY 1;
```

NTILE 함수로 생성한 버킷으로 행을 집계할 수 있다. 아래 쿼리는 4개의 행 그룹으로 sal를 집계한다.  

```sql
SELECT c1, COUNT(*) AS c2, SUM(sal) AS c3
FROM (SELECT sal, NTILE(4) OVER(ORDER BY sal) AS c1 FROM emp WHERE deptno = 30)
GROUP BY c1
ORDER BY c1;
```

ROW&#95;NUMBER 함수로 생성한 순번에 CEIL 함수를 사용하면 다른 형태의 행 그룹을 생성할 수 있다. 아래 쿼리도 4개의 행 그룹으로 sal를 집계한다.  

```sql
SELECT c1, COUNT(*) AS c2, SUM(sal) AS c3
FROM (SELECT sal
             , (ROW_NUMBER() OVER(ORDER BY sal, empno) / (COUNT(*) OVER() / 4)) AS c1)
      FROM emp
      WHERE deptno = 30)
GROUP BY c1
ORDER BY 1;
```

ROW&#95;NUMBER 함수로 생성한 순번에 MOD 함수를 사용하면 또 다른 형태의 행 그룹을 생성할 수 있다. 아래 쿼리도 4개의 행 그룹으로 sal를 집계한다.  

```sql
SELECT c1, COUNT(*) AS c2, SUM(sal) AS c3
FROM (SELECT sal
             , MOD(ROW_NUMBER() OVER(ORDER BY sal, empno), 4) AS c1
      FROM emp
      WHERE deptno = 30)
GROUP BY c1
ORDER BY 1;
```

##### 14.2.3.5. CUME&#95;DIST 함수
<br/>
```
CUME_DIST() OVER([query_partition_clause] order_by_clause)
```

누적 분포 값을 반환한다. 누적분포 값은 0 < y <= 1의 범위를 가진다.  

아래는 CUME&#95;DIST 함수를 사용한 쿼리다. c1 열은 c2 열의 표현식으로 계산된다. sal가 1500인 행을 기준으로 보면 sal가 1500 이하인 행이 4행이므로 전체 6행 중 67%의 분포를 차지한다.  

```sql
SELECT sal
       , CUME_DIS() OVER(ORDER BY sal) AS c1
       , COUNT(*) OVER(ORDER BY sal) / COUNT(*) OVER() AS c2
FROM emp
WHERE deptno = 30
ORDER BY sal;
```

##### 14.2.3.6. PERCENT&#95;RANK 함수
<br/>
```
PERCENT_RANK() OVER([query_partition_clause] order_by_clause)
```

백분위 순위 값을 반환한다. 백분위 순위는 순위의 대상을 100건으로 가정했을 때의 상대 순위다. 백분위 순위 값은 0 <= y <= 1의 범위를 가진다.  

아래는 PERCENT&#95;RANK 함수를 사용한 쿼리다. c1 열은 c2 열의 표현식으로 계산된다. sal가 1500 미만인 행은 3행으로 전체 6행에서 현재 행을 제외한 5행 중 60%(=3/5)의 백분위 순위를 가진다.  

```sql
SELECT sal
       , PERCENT_RANK() OVER(ORDER BY sal) AS c1
       , (RANK() OVER(ORDER BY sal) - 1) / (COUNT(*) OVER() - 1) AS c2
FROM emp
WHERE deptno = 30
ORDER BY 1;
```

##### 14.2.3.7. RATIO&#95;TO&#95;REPORT 함수
<br/>
```
RATIO_TO_REPORT(expr) OVER([query_partition_clause])
```

expr의 합계에 대한 현재 expr의 비율을 반환한다. expr이 널이면 널을 반환한다.  

아래는 RATIO_TO_REPORT 함수를 사용한 쿼리다. c1 열은 c2 열의 표현식으로 계산된다.  

```sql
SELECT sal
       , RATIO_TO_REPORT(sal) OVER() AS c1
       , sal / SUM(sal) OVER() AS c2
FROM emp
WHERE deptno = 30
ORDER BY sal;
```

#### 14.2.4. 분포 함수
<br/>
##### 14.2.4.1. PERCENTILE&#95;CONT 함수
<br/>
```
PERCENTILE_CONT(expr) WITHIN GROUP(ORDER BY expr [DESC | ASC]) [OVER (query_partition_clause)]
```
연속 분포 모델에서 expr에 지정한 백분위 값에 해당하는 값을 계산한다. expr은 0 ~ 1의 범위를 지정할 수 있다.  

아래는 PERCENTILE_CONT 함수를 사용한 쿼리다. SALESMAN 파티션에서 0.5에 해당하는 행은 MARTIN과 TUNNER다. 연속 분포 모델을 가정하므로 c1 열은 1375(=(1250+1500)/2)으로 계산된다.

```sql
SELECT job, ename, sal
       , PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY sal) OVER(PARTITION BY job) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 1, 3, empno;
```

##### 14.2.4.2. PERCENTILE&#95;DISC 함수
```
PERCENTILE_DISC(expr) WITHIN GROUP(ORDER BY expr [DESC | ASC]) [OVER (query_partition_clause)]
```

이산 분포 모델에서 expr에 지정한 백분위 값에 해당하는 값을 계산한다. expr은 0 ~ 1의 범위를 지정할 수 있다.  

아래는 PERCENTILE_DISC 함수를 사용한 쿼리다. SALESMAN 파티션에서 0.5에 해당하는 행은 MARTIN과 TUNNER다. 이산 분포 모델을 가정하므로 c1 열은 1250과 1500 중 1250을 반환한다.  

```sql
SELECT job, ename, sal
       , PERCENTILE_DISC(0.5) WITHIN GROUP(ORDER BY sal) OVER(PARTITION BY job) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 1, 3, empno;
```

##### 14.2.4.3. MEDIAN 함수
<br/>
```
MEDIAN(expr) [OVER(query_partition_clause)]
```

연속 분포 모형을 가정한 중앙값을 반환한다. PERCENTILE_CONT(0.5) 표현식과 결과가 동일하다.  

아래는 MEDIAN 함수를 사용한 쿼리다. c1 열은 연속 분포 모형을 가정한 중앙값을 반환한다.  

```sql
SELECT job, sal, MEDIAN(sal) OVER(PARTITION BY job) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 1, 2;
```

#### 14.2.5. 순차 함수
<br/>
##### 14.2.5.1. FIRST&#95;VALUE 함수
<br/>
```
FIRST_VALUE(expr) [IGNORE NULLS] OVER(analytic_clause)
```

윈도우 첫 행의 expr을 반환한다. IGNORE NULLS 키워드를 기술하면 널이 무시된다.  

아래는 FIRST_VALUE 함수를 사용한 쿼리다. c1 열은 SALESMAN 파티션 첫 행의 sal가 1600이므로 1600을 반환한다.  

```sql
SELECT job, hiredate, sal
       , FIRST_VALUE(sal) OVER(ORDER BY hiredate) AS c1
FROM emp a
WHERE deptno = 30
ORDER BY 1, 2;
```

아래 쿼리의 c2 열은 널이 아닌 첫 번째 comm를 반환하지만 sal가 950인 행은 널이 반환된다. 자신의 이전 행에 널이 아닌 값이 존재하지 않기 때문이다. 모든 행에 값을 채우려면 c3 열처럼 윈도우를 전체 파티션으로 지정해야 한다.  

```sql
SELECT sal, comm
       , FIRST_VALUE(comm) OVER(ORDER BY sal) AS c1
       , FIRST_VALUE(comm) IGNORE NULLS OVER(ORDER BY sal) AS c2
       , FIRST_VALUE(comm) IGNORE NULLS OVER(ORDER BY sal RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS c3
FROM emp a
WHERE deptno = 30
ORDER BY 1, 2;
```

##### 14.2.5.2. LAST&#95;VALUE 함수
<br/>
```
LAST_VALUE(expr) [IGNORE NULLS] OVER(analytic_clause)
```

윈도우 끝 행의 expr을 반환한다. IGNORE NULLS 키워드를 기술하면 널이 무시된다.  

아래는 LAST_VALUE 함수를 사용한 쿼리다. c1 열은 sal와 동일한 값이 반환된다. 윈도우가 기본값인 RANGE UNBOUNDED PRECEDING으로 동작하기 대문이다. c2 열처럼 RANGE BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING으로 윈도우 절을 기술해야 의도한 결과를 얻을 수 있다.  

```sql
SELECT job, hiredate, sal
       , LAST_VALUE(sal) OVER(PARTITION BY job ORDER BY hiredate) AS c1
       , LAST_VALUE(sal) OVER(PARTITION BY job ORDER BY hiredate RANGE BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS c2
FROM emp a
WHERE deptno = 30
ORDER BY 1, 2, 3;
```

아래 쿼리의 c1, c2 열은 결과가 동일하다. c1 열처럼 FIRST_VALUE 함수의 정렬 기준을 반대로 기술하면 LAST_VALUE 함수와 동일한 결과를 얻을 수 있다.  

```sql
SELECT job, hiredate, sal
       , FIRST_VALUE(sal) OVER(PARTITION BY job ORDER BY hiredate DESC) AS c1
       , LAST_VALUE(sal) OVER(PARTITION BY job ORDER BY hiredate RANGE BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS c2
FROM emp a
WHERE deptno = 30
ORDER BY 1, 2, 3;
```

##### 14.2.5.3. NTH&#95;VALUE 함수
<br/>
```
NTH_VALUE(measure_expr, n) [FROM {FIRST | LAST}] [IGNORE NULLS] OVER(analytic_clause)
```

윈도우 n번째 행의 measure&#95;expr을 반환한다. FIRST는 윈도우의 첫 행, LAST는 윈도우의 끝 행부터 검색을 시작한다. 기본값은 FIRST다. NTH_VALUE 함수는 11.1 버전부터 사용할 수 있다.  

아래는 NTH_VALUE 함수를 사용한 쿼리다. c1 열은 FIRST&#95;VALUE 함수와 결과가 동일하다. c2 열은 윈도우 세 번째 행의 sal인 2850을 반환한다.  

```sql
SELECT hiredate, sal  
       , NTH_VALUE(sal, 1) OVER(ORDER BY hiredate) AS c1
       , NTH_VALUE(sal, 3) OVER(ORDER BY hiredate) AS c2
FROM emp
WHERE deptno = 30
ORDER BY hiredate;
```

아래 쿼리의 c1 열은 FROM LAST 키워드가 기술되어 LAST&#95;VALUE 함수와 동일하게 RANGE BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING으로 윈도우를 지정해야 한다. c2 열처럼 정렬 조건을 반대로 기술하는 방식을 사용하자.  

```sql
SELECT job, hiredate, sal
       , NTH_VALUE(sal, 3) FROM LAST OVER(ORDER BY hiredate) RANGE BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS c1
       , NTH_VALUE(sal, 3) OVER(ORDER BY hiredate DESC) AS c2
FROM emp
WHERE deptno = 30
ORDER BY 1;
```

#### 14.2.6. 기타 함수
<br/>
##### 14.2.6.1. LAG 함수
```
LAG(value_expr [, offset [, default]]) [IGNORE NULLS] OVER([query_partition_clause] order_by_clause)
```

현재 행에서 offset 이전 행의 value&#95;expr을 반환한다. offset은 행 기준이며 기본 값은 1이다. default에 이전 행이 없을 경우 반환할 값을 지정할 수 있다. default의 기본값은 널이다.  

아래는 LAG 함수를 사용한 쿼리다. c1 열은 직전 행의 sal, c2 열은 3행 이전 행의 sal를 반환한다.  

```sql
SELECT hiredate, sal  
       , LAG(sal) OVER(ORDER BY hiredate) AS c1
       , LAG(sal, 3) OVER(ORDER BY hiredate) AS c2
FROM emp
WHERE deptno = 30
ORDER BY 1;
```

아래 쿼리의 c1 열은 기본값이 999이므로 2행 이전 행이 존재하지 않는 ALLEN, WARD에서 999가 반환된다. MARTIN은 이전 행이 널이지만 IGNORE NULLS 옵션으로 인해 두 번째 행의 값인 500이 반환된다.  

```sql
SELECT ename, hiredate, comm
       , LAG(comm, 2, 999) IGNORE NULLS OVER(ORDER BY hiredate) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 2;
```

##### 14.2.6.2. LEAD 함수
<br/>
```
LEAD(value_expr [, offset [, default]]) [IGNORE NULLS] OVER([query_partition_clause] order_by_clause)
```

현재 행에서 offset 이후 행의 value&#95;expr을 반환한다.  

아래는 LEAD 함수를 사용한 쿼리다. c1 열은 다음 행의 sal, c2 열은 3행 이후 행의 sal를 반환한다.  

```sql
SELECT hiredate, sal
       , LEAD(sal) OVER(ORDER BY hiredate) AS c1
       , LEAD(sal, 3) OVER(ORDER BY hiredate) AS c2
FROM emp
WHERE deptno = 30
ORDER BY 1;
```

아래 쿼리의 c1, c2 열은 결과가 동일하다. LEAD 함수는 LAG 함수와 반대로 동작하므로 내림차순으로 정렬하면 LAG 함수와 동일한 결과를 얻을 수 있다.  

```sql
SELECT hiredate, sal
       , LAG(sal) OVER(ORDER BY hiredate DESC) AS c1
       , LEAD(sal) OVER(ORDER BY hiredate) AS c1
FROM emp
WHERE deptno = 30
ORDER BY hiredate;
```

LAG 함수와 LEAD 함수는 행 기준으로 동작하므로 분석 함수의 정렬 값이 고유하지 않으면 값이 무작위로 변경될 수 있다. 아래 예제는 쿼리의 ORDER BY 절을 변경했을 뿐인데 c1 값이 변경되었다. JAMES의 c1 값이 첫 번째 쿼리는 500, 두 번째 쿼리는 1400이다.  

```sql
SELECT empno, ename, sal, comm
       , LEAD(comm) OVER(ORDER BY sal) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 3, 1;
```

```sql
SELECT empno, ename, sal, comm
       , LEAD(comm) OVER(ORDER BY sal) AS c1
FROM emp
WHERE deptno = 30
ORDER BY 3, 2;
```

##### 14.2.6.3. LISTAGG 함수
<br/>
```
LISTAGG(measure_expr [, 'delimiter'] [listagg_overflow_clause]) WITHIN GROUP (order_by_clause) [OVER query_partition_clause]
```

measure&#95;expr를 order_by_clause로 정렬한 후 delimiter로 구분하여 연결한 값을 반환한다. delimiter의 기본값은 널이다.  

아래는 LISTAGG 함수를 사용한 쿼리다. c1 열은 결합된 문자열을 반환한다. LISTAGG 함수는 윈도우 함수가 아니기 때문에 문자열을 누적하여 결합할 수 없다.  

```sql
SELECT job, ename
       , LISTAGG(ename, ',') WITHIN GROUP (ORDER BY ename) OVER(PARTITION BY job) AS c1
FROM emp
WHERE deptno = 30
ORDER BY job, ename;
```
