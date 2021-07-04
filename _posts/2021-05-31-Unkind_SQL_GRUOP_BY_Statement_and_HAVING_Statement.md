---
title: GROUP BY 절과 HAVING 절
categories:
- Unkind_SQL
feature_text: |
  ## 10. GROUP BY 절과 HAVING 절
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

#### 10.2.1. ROLLUP
<br/>
```
ROLLUP(expr [, expr]... | ([expr [, expr]]))
```

ROLLUP은 지정한 표현식의 계층별 소계와 총계를 집계한다.  

#### 10.2.2. CUBE
<br/>
```
ROLLUP(expr [, expr]... | ([expr [, expr]]))
```

지정한 표현식의 모든 조홥을 집계한다.  

아래는 CUBE를 사용한 쿼리다.  

```sql
SELECT deptno, COUNT(*) AS c1
FROM emp
WHERE sal > 2000
GROUP BY CUBE (deptno, job)
ORDER BY 1, 2;
```

#### 10.2.3. GROUPING SETS
<br/>
```
GROUPING SETS({rollup_cube_clause | grouping_expression_list})
```

GROUPING SETS은 지정한 행 그룹으로 행을 집계한다. 행 그룹으로 ROLLUP과 CUBE를 사용할 수도 있다.  

아래는 GROUPING SETS를 사용한 쿼리다.  

```sql
SELECT deptno, job, COUNT(*) AS c1
FROM emp
WHERE sal > 2000
GROUP BY GROUPING SETS(deptno, job)
ORDER BY 1, 2;
```

아래는 GROUPING SETS에 ROLLUP을 사용한 쿼리다.  

```sql
SELECT deptno, job, COUNT(*) AS c1
FROM emp
WHERE sal > 2000
GROUP BY GROUPING SETS(deptno, ROLLUP(job))
ORDER BY 1, 2;
```

#### 10.2.4 조합 열
<br/>
하나의 단위로 처리되는 열의 조합으로 아래와 같이 동작한다.  

+ ROLLUP((a, b))  
(a, b), ()
+ ROLLUP(a, (b, c))
(a, b, c), (a), ()
+ ROLLUP((a, b), c)  
(a, b, c), (a, b), ()  

#### 10.2.5. 연결 그룹
<br/>
연결 그룹(concatenated gruoping)을 사용하면 행 그룹을 간결하게 작성할 수 있다. 연결 그룹은 아래와 같이 동작한다.  

+ a, ROLLUP(b)  
(a, b), (a)  
+ a, ROLLUP(b, c)  
(a, b, c), (a, b), (a)  
+ a, ROLLUP(b), ROLLUP(c)  
(a, b, c), (a, b), (a, c), (a)
+ GROUPING SETS(a, b), GROUPING SETS(c, d)  
(a, c), (a, d), (b, c), (b, d)  

#### 10.2.6. 관련 함수
<br/>
##### 10.2.6.1. GROUPING 함수
<br/>
```
GROUPING(expr)
```
expr이  행 그룹에 포함되면 0, 포함되지 않으면 1을 반환한다. 널로 반환되는 행 그룹에 값을 설정하거나 결과의 정렬 순서를 조정할 수 있다.  

아래 쿼리에서 c1 열은 () 행 그룹, c2 열은 deptno 행 그룹과 () 행 그룹에 포함되지 않기 때문에 각각의 행 그룹에서 1이 반환된다.  

```sql
SELECT deptno, job, COUNT(*) AS c1,
       GROUPING(deptno) AS g1, GROUPING (job) AS g2
FROM emp
GROUP BY ROLLUP(deptno, job)
ORDER BY 1, 2;
```

##### 10.2.6.2. GROUPING&#95;ID 함수
<br/>
```
GROUPING_ID(expr [, expr]...)
```

GROUPING 함수의 결과 값을 연결한 값의 비트 백터에 해당하는 숫자 값을 반환한다.  

##### 10.2.6.3. GROUP&#95;ID 함수
<br/>
```
GROUP_ID()
```

중복되지 않은 행 그룹은 (), 중복된 행 그룹은 1을 반환한다. 중복된 행 그룹을 제거할 때 사용할 수 있다.  

```sql
SELECT deptno, job, COUNT(*) AS c1, GROUP_ID() AS gi
FROM emp
WHERE sal > 2000
GROUP BY deptno, ROLLUP(deptno, job);
```
