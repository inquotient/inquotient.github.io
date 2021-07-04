categories:
- Unkind_SQL
feature_text: |
  ## 17. PIVOT 절과 UNPIOVT 절
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

PIVOT은 회전시킨다는 의미를 가지고 있다. PIVOT 절은 행을 열고 회전시키고, UNPIVOT 절은 열을 행으로 회전시킨다. PIVOT 절과 UNPIVOT 절은 11.1 버전부터 사용할 수 있다.  

### 17.1. PIVOT 절
<br/>
PIVOT 절은 행을 열로 전환한다.  

#### 17.1.1. 기본 문법
<br/>
PIVOT 절의 구문은 아래와 같다.  

```
PIVOT [XML]
      (aggregate_function (expr) [[AS] alias]
      [, aggregate_function (expr) [[AS] alias]]...
         FOR {column | (column [, column]...)}
         IN ( { { {expr | (expr [, expr]...)} [[AS] alias]}...
            | subquery
            | ANY [, ANY]...
            })
      )
```

+ aggregate_function : 집계할 열을 지정
+ FOR 절 : PIVOT할 열을 지정
+ IN 절 : PIVOT할 값을 지정  

아래는 PIVOT 절을 사용한 쿼리다. PIVOT 절은 집계 함수와 FOR 절에 지정되지 않은 열을 기준으로 집계되기 때문에 인라인 뷰를 통해 사용할 열을 지정해야 한다.  

```sql
SELECT *
FROM (SELECT job, deptno, sal FROM emp)
PIVOT (SUM (sal) FOR deptno IN (10, 20, 30))
ORDER BY 1;
```

아래 쿼리는 인라인 뷰에 yyyy 표현식을 추가했다. 행 그룹에 yyyy 표현식이 추가된 것을 확인할 수 있다.  

```sql
SELECT *
FROM (SELECT TO_CHAR(hiredate, 'YYYY') AS yyyy, job, deptno, sal FROM emp)
PIVOT (SUM (sal) FOR deptno IN (10, 20, 30))
ORDER BY 1, 2;
```

아래 쿼리는 집계 함수와 IN 절에 별칭을 지정했다. 별칭을 지정하면 결과 집합의 열명이 변경된다.  

```sql
SELECT *
FROM (SELECT job, deptno, sal FROM emp)
PIVOT (SUM (sal) AS sal FOR deptno IN (10 AS d10, 20 AS d20, 30 AS d30))
ORDER BY 1;
```

집계 함수와 IN 절에 지정한 별칭에 따라 열명이 부여된다. 집계 함수와 IN 절 모두 별칭을 지정하는 편이 바람직하다.  

SELECT 절에 부여된 열명을 지정하면 필요한 열만 조회할 수 있다.  

```sql
SELECT job, d20_sal
FROM (SELECT job, deptno, sal FROM emp)
PIVOT (SUM (sal) AS sal FOR deptno IN (10 AS d10, 20 AS d20, 30 AS d30))
WHERE d20_sal > 2500
ORDER BY 1;
```

PIVOT 절은 다수의 집계 함수를 지원한다. 아래 쿼리는 SUM 함수와 COUNT 함수를 함께 사용했다.  

```sql
SELECT *
FROM (SELECT job, deptno, sal FROM emp)
PIVOT (SUM (sal) AS sal, COUNT(*) AS cnt FOR deptno IN (10 AS d10, 20 AS d20))
ORDER BY 1;
```

FOR 절에도 다수의 열을 기술할 수 있다. 아래와 같이 IN 절에 다중 열을 사용해야 한다.  

```sql
SELECT *
FROM (SELECT TO_CHAR(hiredate, 'YYYY') AS yyyy, job, deptno, sal FROM emp)
PIVOT (SUM (sal) AS sal, COUNT(*) AS cnt
       FOR (deptno, job) IN ((10, 'ANALYST') AS d10a, (10, 'CLERK') AS d10c)
                            ,(10, 'ANALYST') AS d20a, (10, 'CLERK') AS d20c))
ORDER BY 1;
```

PIVOT 절에 XML 키워드를 기술하면 XML 포맷으로 결과가 반환된다. XML 키워드를 사용하면 IN 절에 서브 쿼리와 ANY 키워드를 사용할 수 있다. 아래는 서브 쿼리를 사용한 쿼리다.  

```sql
SELECT *
FROM (SELECT job, deptno, sal FROM emp)
PIVOT (SUM (sal) AS sal FOR deptno IN (SELECT deptno FROM dept))
ORDER BY 1;
```

ANY 키워드는 존재하는 값과 일치하는 10, 20, 30과 일치한다.  

```sql
SELECT *
FROM (SELECT job, deptno, sal FROM emp)
PIVOT XML (SUM (sal) AS sal FOR deptno IN (ANY))
ORDER BY 1;
```

XML 키워드를 기술해도 다수의 집계 함수와 다수의 열을 사용할 수 있다.  

```sql
SELECT *
FROM (SELECT job, deptno, sal FROM emp)
PIVOT XML (SUM (sal) AS sal, COUNT(*) AS cnt FOR (deptno, job) IN (ANY, ANY));
```

#### 17.1.2. 기존 방식
<br/>
11.1 이전 버전은 집계 함수와 DECODE 함수로 PIVOT을 수행할 수 있다.  

```sql
SELECT job
       , SUM(DECODE(deptno, 10, sal)) AS d10_sal
       , SUM(DECODE(deptno, 20, sal)) AS d20_sal
       , SUM(DECODE(deptno, 30, sal)) AS d30_sal
FROM emp
GROUP BY job
ORDER BY job;
```

### 17.2. UNPIVOT 절
<br/>
UNPIVOT 절은 PIVOT 절과 반대로 동작한다. 열을 행으로 전환한다.  

#### 17.2.1. 기본 문법
<br/>
UNPIVOT 절의 구문은 아래와 같다.  

```
UNPIOVT [{INCLUDE | EXCLUDE} NULLS]
        (    {column | (column [, col]...)})
         FOR {column | (column [, col]...)}
         IN ({column | (column [, col]...)} [AS {litera | (litera [, litera]...)}]
          [, {column | (column [, col]...)} [AS {litera | (litera [, litera]...)}]]...
            )
        )
```

+ UNPIVOT column : UNPIOVT된 값이 들어갈 열을 지정
+ FOR 절 : UNPIOVT된 값을 설명할 값이 들어갈 열을 지정
+ IN 절 : UNPIOVT할 열과 설명할 값의 리터럴 값을 지정  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
CREATE TABLE t1 AS
SELECT job, d10_sal, d20_sal, d10_cnt, d20_cnt
FROM (SELECT job, deptno, sal FROM emp WHERE job IN ('ANALYST', 'CLERK'))
PIVOT (SUM (sal) AS sal, COUNT(*) AS cnt FOR deptno IN (10 AS d10, 20 AS d20));
```

아래는 UNPIVOT 절을 사용한 쿼리다. d10&#95;sal, d20&#95;sal 열이 행으로 전환된다.  

```sql
SELECT job, deptno, sal
FROM t1
UNPIOVT (sal, FOR deptno IN (d10_sal, d20_sal))
ORDER BY 1, 2;
```

IN 절에 별칭을 지정하면 FOR 절에 지정한 열의 값을 변경할 수 있다. 아래 쿼리는 10, 20으로 값을 변경했다.  

```sql
SELECT job, deptno, sal
FROM t1
UNPIVOT (sal FOR deptno IN (d10_sal AS 10, d20_sal AS 20))
ORDER BY 1, 2;
```

아래와 같이 INCLUDE NULLS 키워드를 기술하면 UNPIOVT된 열의 값이 널인 행도 결과에 포함된다.  

```sql
SELECT job, deptno, sal
FROM t1
UNPIVOT INCLUDE NULLS (sal FOR deptno IN (d10_sal AS 10, d20_sal AS 20))
ORDER BY 1, 2;
```

UNPIVOT 절에 다수의 열을 지정할 수 있다. 아래 쿼리는 sal 열과 cnt 열을 UNPIVOT한다.  

```sql
SELECT *
FROM t1
UNPIOVT ((sal, cnt)) FOR deptno IN ((d10_sal, d10_cnt) AS 10, (d20_sal, d20_cnt) AS 20))
ORDER BY 1, 2;
```

FOR 절에도 다수의 열을 지정할 수 있다. PIVOT 절의 결과가 UNPIVOT 절에 인라인 뷰로 공급되는 방식이다. INCLUDE NULLS 키워드로 파티션 아우터 조인의 동작을 모방할 수 있다.  

```sql
SELECT *
FROM (SELECT job, deptno, sal, comm FROM emp)
PIVOT (SUM (sal) AS sal, SUM (comm) AS comm)
       FOR deptno IN (10 AS d10, 20 AS d20, 30 AS d30))
UNPIVOT INCLUDE NULLS
        ((sal, comm) FOR deptno IN ((d10_sal, d10_comm) AS 10
                                   ,(d20_sal, d20_comm) AS 20
                                   ,(d30_sal, d30_comm) AS 30))
ORDER BY 1, 2;
```

#### 17.2.2. 기존 방식
<br/>
11.1 이전 버전에서는 카티션 곱을 사용하여 UNPIVOT을 수행할 수 있다. 11장에서 살펴본 행복제 기법과 동일하다. UNPIOVT할 열의 개수만큼 행을 복제하고 DECODE 함수로 UNPIOVT할 열을 선택하는 방식이다.  

```sql
SELECT a.job
       , DECODE(b.lv, 1, 10, 2, 20) AS deptno
       , DECODE(b.lv, 1, a.d10_sal, 2, a.d20_sal) AS sal
       , DECODE(b.lv, 1, a.d10_cnt, 2, a.d20_cnt) AS cnt
FROM t1 a
     , (SELECT LEVEL AS lv FROM DUAL CONNECT BY LEVEL <= 2) b
ORDER BY 1, 2;
```

아래 쿼리에서 강조된 부분이 DECODE 함수로 선택한 값이다.  

```sql
SELECT a.job, b.lv, a.d10_sal, a.d20_sal, a.d10_cnt, a.d20_cnt
FROM t1 a
     , (SELECT LEVEL AS lv FROM DUAL CONNECT BY LEVEL <= 2) b
ORDER BY a.job, b.lv;
```

### 17.3. 활용 예제
<br/>
#### 17.3.1. 동적 PIVOT
<br/>
동적 PIVOT은 쿼리 수행 시점에 동적으로 열이 변경되는 PIVOT을 말한다. 쿼리는 수행 전에 열의 개수가 결정되어야 하기 때문에 정적(static) 쿼리로는 동적 PIVOT이 불가능하다. XML 키워드를 사용하면 반환되는 열이 1개로 고정되므로 동적 PIVOT이 가능하지만 활용도는 높지 않다. 결국 동적(dynamic) 쿼리를 사용해야 동적 PIVOT을 수행할 수 있다.  

아래 쿼리는 동적 PIVOT을 수행한다. SQL&#42;Plus는 대체 변수로 동적 쿼리를 생성할 수 있다. DEF[INE] 명령어로 대체 변수를 선언하고, &변수 형식으로 대체 변수를 참조했다.  

```sql
DEF v_af = 'SUM (sal) AS sal';
DEF v_il = 'deptno IN (10 AS d10, 20 AS d20)';

SELECT *
FROM (SELECT job, deptno, sal FROM emp)
PIVOT (&v_af FOR &v_il)
ORDER BY 1;
```

#### 17.3.2. 전체 열 UNPIVOT
<br/>
테이블의 전체 열을 UNPIVOT하여 값을 조회하거나 집계할 수 있다.  

아래 쿼리는 테이블의 전체 열을 UNPIVOT한다. 열이 많은 행을 조회할 때 유용하다. IN 절에 입력되는 값은 데이터 타입이 모두 동일해야 한다.  

```sql
WITH w1 AS (
SELECT TO_CHAR(empno) AS empno, ename, job, TO_CHAR(mgr) AS mgr
     , TO_CHAR(hiredate, 'YYYY-MM-DD') AS hiredate, TO_CHAR(sal) AS sal
     , TO_CHAR(comm) AS comm, TO_CHAR(deptno) AS deptno
FROM emp
WHERE empno = 7788)
SELECT *
FROM w1
UNPIVOT (value FOR column_name IN (empno, ename, job, mgr, hiredate, sal, comm, deptno));   
```

UNPIOVT 절을 사용하면 열 값의 분포를 계산할 수 있다. 아래 쿼리는 job 열과 deptno 열 값의 분포를 계산한다.  

```sql
SELECT column_name, value, COUNT(*) AS cnt
FROM (SELECT job, TO_CHAR(deptno) AS deptno FROM emp)
UNPIOVT (value FOR column_name IN (job, deptno))
GROUP BY column_name, value
ORDER BY 1, 2;
```

#### 17.3.3. 행렬 전환 집계
<br/>
DW 시스템은 행을 열로 전환한 후 수식을 통해 값을 계산하고, 계산된 값을 다시 행으로 전환하는 집계 방식을 사용한다.  

아래 쿼리는 DW 시스템에서 자주 사용하는 집계 쿼리 방식이다. w1에서 부서별 sal를 열로 PIVOT하고, w2에서 표현식으로 열을 계산한 후, 메인 쿼리에서 계산한 열을 행으로 UNPIOVT한다. 20번 부서의 CLERK은 타 부서 sal의 평균 대비 169% 수준의 sal를 받는 것으로 계산된다.  

```sql
WITH w1 AS (
SELECT job
     , NVL(d10_sal, 0) AS d10_sal
     , NVL(d20_sal, 0) AS d20_sal
     , NVL(d30_sal, 0) AS d30_sal
FROM (SELECT job, deptno, sal FROM emp)
PIVOT (SUM (sal) AS sal FOR deptno IN (10 AS d10, 20 AS d20, 30 AS d30)))
   , w2 AS (
SELECT a.*
     , d10_sal / NVL(NULLIF(d20_sal + d30_sal, 0) / 2, d10_sal) AS d10_ratio
     , d20_sal / NVL(NULLIF(d10_sal + d30_sal, 0) / 2, d10_sal) AS d20_ratio
     , d30_sal / NVL(NULLIF(d10_sal + d20_sal, 0) / 2, d10_sal) AS d30_ratio
FROM w1 a)
SELECT job, deptno, sal, ROUND(ratio, 2) * 100 AS ratio
FROM w2
UNPIVOT ((sal, ratio) For deptno IN ((d10_sal, d10_ratio) AS 10
                                   , (d20_sal, d20_ratio) AS 20
                                   , (d30_sal, d30_ratio) AS 30))
ORDER BY 1, 2;
```
