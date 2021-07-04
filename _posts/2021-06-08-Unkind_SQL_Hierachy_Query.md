---
title:  계층 쿼리
categories:
- Unkind_SQL
feature_text: |
  ## 16. 계층 쿼리
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

계층 쿼리를 사용하면 순환 관계를 가진 데이터를 조회할 수 있다.  

+ 부모 노드 : 현재 노드의 직전 상위 노드
+ 자식 노드 : 현재 노드의 직후 하위 노드
+ 루트 노드 : 부모 노드가 존재하지 않는 노드
+ 브랜치 노드 : 부모 노드와 자식 노드가 존재하는 노드
+ 리프 노드 : 자식 노드가 존재하지 않는 노드  

순환 관계는 계층의 깊이에 따라 레벨이 부여된다. 루트 노드는 레벨이 1이고 계층이 전개될수록 레벨이 증가한다.  

### 16.1. 계층 쿼리 절
<br/>
#### 16.1.1. 기본 문법
<br/>
계층 쿼리 절은 WHERE 절 다음에 기술하며, FROM 절이 수행된 후 수행된다. START WITH 절과 CONNECT BY 절로 구성되며, START WITH 절이 수행된 후 CONNECT BY 절이 수행된다. START WITH 절은 생략이 가능하다.  

+ START WITH 절  
루트 노드를 생성하며 1번만 수행
+ CONNECT BY 절  
루트 노드의 하위 노드를 생성하며 조회 결과가 없을 때까지 반복 수행  

아래는 계층 쿼리 절에서 사용할 수 있는 연산자, 슈도 칼럼, 함수다.  

<table>
  <thead>
    <tr>
      <td>유형</td>
      <td>항목</td>
      <td>설명</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2">연산자</td>
      <td>PRIOR</td>
      <td>직전 상위 노드의 값을 반환</td>
    </tr>
    <tr>
      <td>CONNECT&#95;BY&#95;ROOT</td>
      <td>루트 노드의 값을 반환</td>
    </tr>
    <tr>
      <td rowspan="3">슈도 칼럼</td>
      <td>LEVEL</td>
      <td>현재 레벨을 반환</td>
    </tr>
    <tr>
      <td>CONNECT&#95;BY&#95;ISLEAF</td>
      <td>리프 노드인 경우 1, 아니면 0을 반환</td>
    </tr>
    <tr>
      <td>CONNECT&#95;BY&#95;ISCYCLE</td>
      <td>루프가 발생한 경우 1, 아니면 0을 반환</td>
    </tr>
    <tr>
      <td>함수</td>
      <td>SYS&#95;CONNECT&#95;BY&#95;PATH</td>
      <td>루프 노드에서 현재 노드까지의 경로를 반환</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래는 계층 쿼리 절을 사용한 쿼리다. START WITH 절로 mgr가 존재하지 않는 행을 조회하고, CONNECT BY 절로 현재 노드의 mgr가 직전 상위 노드의 empno인 행을 반복해서 조회한다. ename 열은 LEVEL 슈도 칼럼과 LPAD 함수를 사용하여 계층의 레벨에 따라 값을 들여쓰기했다.  

```sql
SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr, PRIOR empno AS empno_p
START WITH mgr IS NULL       -- mgr가 존재하지 않는 행
CONNECT BY mgr = PRIOR empno -- mgr가 부모 노드의 empno인 행;
```

##### 16.1.1.1. SYS&#95;CONNECT&#95;BY&#95;PATH 함수
<br/>
루트 노드에서 현재 노드까지의 column을 char로 구분하여 연결한 값을 반환한다. column 값에 char가 포함되어 있으면 ORA-30004 에러, 연결한 값의 길이가 4000보다 길면 ORA-01489 에러가 발생한다.  

+ ORA-30004  
... 구분 기호를 열 값의 일부로 사용할 수 없습니다.
+ ORA-01489  
문자열 연결의 결과가 너무 깁니다.  

아래는 CONNECT&#95;BY&#95;ROOT 연산자, CONNECT&#95;BY&#95;ISLEAF 슈도 칼럼, SYS&#95;CONNECT&#95;BY&#95;PATH 함수를 사용한 쿼리다.  

```sql
SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr
       , CONNECT_BY_ISLEAF AS lf
       , CONNECT_BY_ROOT ename as rt
       , SYS_CONNECT_BY_PATH(ename, ',') AS pt
FROM emp
START WITH mgr IS NULL
CONNECT BY mgr = PRIOR empno;
```

#### 16.1.2. 동작 원리
<br/>
계층 쿼리 절은 START WITH 절로 루트 노드를 생성한 후, 결과가 없을 때까지 CONNECT 절을 반복 수행하여 하위 노드를 생성한다.  

#### 16.1.3. 전개 방향
<br/>
순환 관계는 순방향 또는 역방향으로 전개할 수 있다. 순방향 전개와 역방향 전개는 데이터 모델 상의 전개 방향이 반대일 뿐 동작 원리는 동일하다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>전개 방향</td>
      <td>START WITH 절</td>
      <td>CONNECT BY 절</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>순방향 전개</td>
      <td>부모 -> 자식</td>
      <td>부모 노드 조회</td>
      <td>PK에 PRIOR 기술</td>
    </tr>
    <tr>
      <td>역방향 전개</td>
      <td>자식 -> 부모</td>
      <td>자식 노드 조회</td>
      <td>FK에 PRIOR 기술</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 쿼리는 순방향으로 계층을 전개한다. 순방향 전개는 START WITH 절로 부모 노드를 조회하고, CONNECT BY 절을 통해 자식 노드로 계층을 전개한다. PK(empno)에 PRIOR 연산자를 기술하여 현재 노드의 mgr가 보모 노드의 empno인 행을 조회한다.  

```sql
SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr
FROM emp
START WITH mgr IS NULL
CONNECT BY mgr = PRIOR empno;
```

아래 쿼리는 역방향으로 계층을 전개한다. 역방향 전개는 START WITH 절로 자식 노드를 조회하고, CONNECT BY 절을 통해 부모 노드로 계층을 전개한다. FK(empno)에 PRIOR 연산자를 기술하여 현재 노드의 mgr가 자식 노드의 empno인 행을 조회한다.  

```sql
SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr
FROM emp
START WITH ename = 'ADMAS'
CONNECT BY empno = PRIOR mgr;
```

#### 16.1.4. 계층 쿼리
<br/>
계층 쿼리 절은 형제 노드의 행을 정렬하기 위해 SIBLINGS 키워드를 제공한다.  

계층 쿼리 절에 ORDER BY 절을 사용하면 계층 구조와 무관하게 행이 정렬된다.  

```sql
SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr, sal
FROM emp
START WITH mgr IS NULL
CONNECT BY mgr = PRIOR empno
ORDER BY sal;
```

ORDER BY 절에 SIBLINGS 키워드를 사용하면 형제 노드 내에서만 행이 정렬되기 때문에 계층 구조를 유지한 상태로 행을 정렬할 수 있다. CLARK, BLAKE, JONES는 형제 노드다. sal 순서로 행이 정렬된다. BLAKE의 하위 노드인 JAMES ~ ALLEN도 sal 순서로 행이 정렬된다.  

```sql
SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr, sal
FROM emp
START WITH mgr IS NULL
CONNECT BY mgr = PRIOR empno
ORDER SIBLINGS BY sal;
```

#### 16.1.5. 루프 처리
<br/>
부모 노드가 현재 노드의 자식 노드로 연결되면 루프(loop)가 발생한다. 계층 쿼리 절은 루프를 처리하기 위해 NOCYCLE 키워드와 CONNECT&#95;BY&#95;ISCYCLE 슈도 칼럼을 제공한다.  

아래 쿼리는 CONNECT BY 절에 NOCYCLE 키워드를 기술했다. CONNECT BY 절에 NOCYCLE 키워드를 기술하면 루프가 발생한 노드를 전개하지 않는다. ic 열이 1인 행이 루프가 발생한 행이다.  

```sql
SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr, CONNECT_BY_ISCYCLE AS ic
FROM emp_l
START WITH empno = 7839
CONNECT BY NOCYCLE mgr = PRIOR empno;
```

### 16.2. 재귀 서브 쿼리 팩토링
<br/>
11.2 버전부터 재귀 서브 쿼리 팩토링(recursive subquery factoring) 기능을 사용할 수 있다. 재귀 서브 쿼리 팩토링은 ANSI 표준 SQL 문법이다. 계층 쿼리 절보다 복잡하지만 다양한 기능을 사용할 수 있다.  

#### 16.2.1. 기본 문법
<br/>
```
WITH query_name([c_alias [, c_alias]...]) AS (subquery) [search_clause] [cycle_clause]
```
재귀 서브 쿼리 팩토링은 WITH 절을 사용한다. 재귀 서브 쿼리 팩토링의 WITH 절은 서브 쿼리, SEARCH 절, CYCLE 절로 구성된다.  

서브 쿼리는 UNION ALL 연산자로 구성된다. UNION ALL 연산자의 상단 쿼리가 START WITH 절, 하단 쿼리가 CONNECT BY 절의 역할을 수행한다. WITH 절에 정의한 서브 쿼리를 하단 쿼리와 조인함으로써 재귀적으로 조인이 수행되는 방식으로 동작한다.  

아래 쿼리는 순방향으로 계층을 전개한다. 현재 노드의 mgr가 부모 노드의 empno인 행을 조회한다.  

```sql
WITH w1(empno, ename, mgr, lv) AS (
SELECT empno, ename, mgr, 1 AS lv
FROM emp
WHERE mgr IS NULL     -- START WITH 절
UNION ALL
SELECT c.empno, c.ename, c.mgr, p.lv + 1 AS lv
FROM w1 p, emp c
WHERE c.mgr = p.empno -- CONNECT BY 절
)
SELECT lv, empno, LPAD(' ', lv - 1, ' ') || ename AS ename, mgr FROM w1;
```

아래 쿼리는 역방향으로 계층을 전개한다. 현재 노드의 empno가 자식 노드의 mgr인 행을 조회한다.  

```sql
WITH w1(empno, ename, mgr, lv) AS (
SELECT empno, ename, mgr, 1 AS lv
FROM emp
WHERE ename = 'ADAMS' -- START WITH 절
UNION ALL
SELECT c.empno, c.ename, c.mgr, p.lv + 1 AS lv
FROM w1 p, emp c
WHERE c.empno = p.mgr -- CONNECT BY 절
)
SELECT lv, empno, LPAD(' ', lv - 1, ' ') || ename AS ename, mgr FROM w1;
```

재귀 서브 쿼리 팩토링은 계층 정보를 조회하기 위한 연산자와 슈도 칼럼을 제공하지 않는다. 아래 쿼리의 표현식으로 계층 정보를 조회할 수 있다.  

```sql
WITH w1 (empno, ename, mgr, lv, empno_p, rt, pt) AS (
SELECT empno, ename, mgr
       , 1 AS lv          -- LEVEL
       , NULL AS empno_p  -- PRIOR
       , ename AS rt      -- CONNECT_BY_ROOT
       , ename AS pt      -- SYS_CONNECT_BY_PATH
FROM emp
WHERE mgr IS NULL
UNION ALL
SELECT c.empno, c.ename, c.mgr
       , p.lv + 1 AS lv               -- LEVEL
       , p.empno AS empno_p           -- PRIOR
       , p.rt                         -- CONNECT_BY_ROOT
       , p.pt || ',' || c.ename AS pt -- SYS_CONNECT_BY_PATH
FROM w1 p, emp c
WHERE c.mgr = p.empno)
SEARCH DEPTH FIRST BY empno SET so
SELECT lv, empno, LPAD(' '. lv - 1, ' ') || ename AS ename, mgr, empno_p, rt, pt
       , CASE WHEN lv - LEAD(lv) OVER(ORDER BY so) < 0
              THEN 0
              ELSE 1
         END AS lf  -- CONNECT_BY_ISLEAF
FROM w1
ORDER BY so;
```

#### 16.2.2. 계층 정렬
<br/>
```
SEARCH {DEPTH | BREADTH} FIRST BY c_alias [, c_alias]... SET ordering_column
```
재귀 서브 쿼리 팩토링은 계층을 정렬하기 위해 SEARCH 절을 제공한다. BREADTH 방식과 DEPTH 방식을 사용할 수 있으며, FIRST BY 뒤에 기술된 c&#95;alias에 따라 행이 정렬된다. ordering&#95;column은 정렬 순번이 반환될 열을 지정한다.  

+ BREADTH : 자식 행을 반환하기 전에 형제 행을 반환 (기본값)
+ DEPTH : 형제 행을 반환하기 전에 자식 행을 반환 (= 계층 쿼리 절)  

아래는 BREADTH 방식을 사용한 쿼리다. BREADTH 방식은 너비(breadth)를 기준으로 계층을 탐색한다. 2레벨이 모두 반환된 후 3레벨이 반환된다.  

```sql
WITH w1 (empno, ename, mgr, lv) AS (
SELECT empno, ename, mgr, 1 AS lv FROM emp WHERE mgr IS NULL
UNION ALL
SELECT c.empno, c.ename, c.mgr, p.lv + 1 AS lv
FROM w1 p, emp c
WHERE c.mgr = p.empno)
SEARCH BREADTH FIRST BY empno SET so
SELECT lv, empno, LPAD(' ', lv - 1, ' ') || ename AS ename, mgr, so
FROM w1
ORDER BY so;
```

아래는 DEPTH 방식을 사용한 쿼리다. DEPTH 방식은 깊이(depth)를 기준으로 계층을 탐색한다. 계층 쿼리 절과 결과가 동일하다.  

```sql
WITH w1(empno, ename, mgr, lv) AS (
SELECT empno, ename, mgr, 1 AS lv FROM emp WHERE mgr IS NULL
UNION ALL
SELECT c.empno, c.ename, c.mgr, p.lv + 1 AS lv
FROM w1 p, emp c
WHERE c.mgr = p.empno)
SEARCH DEPTH FIRST BY empno SET so
SELECT lv, empno, LPAD(' ', lv - 1, ' ') || ename AS ename, mgr, so
FROM w1
ORDER BY so;
```

#### 16.2.3. 루프 처리
<br/>
```
CYCLE c_alias [, c_alias]... SET cycle_mark_c_alias TO cycle_value DEFAULT no_cycle_value
```
재귀 서브 쿼리 팩토링은 루프를 처리하기 위해 CYCLE 절을 제공한다. CYCLE 절은 상위 노드에 동일한 c&#95;alias 값이 존재하면 루프가 발생한 것으로 인식한다.  

+ c&#95;alias [, c&#95;alias] : 루프 여부를 확인할 열
+ cycle&#95;mark&#95;c&#95;alias : 루프 여부를 반환할 열
+ cycle&#95;value : 루프가 발생한 경우 반환할 값
+ no&#95;cycle&#95;value : 루프가 발생하지 않은 경우 반환할 값  

아래는 CYCLE 절을 사용한 쿼리다. 상위 노드(LV = 1)에 현재 노드(LV = 4)의 empno인 7839와 동일한 empno가 존재하기 때문에 루프가 발생한 것으로 인식한다. 계층 쿼리 절과 달리 루프가 발생한 노드까지 반환된다. ic 열이 1인 행이 루프가 발생한 행이다.  

```sql
WITH w1 (empno, ename, mgr, lv) AS (
SELECT empno, ename, mgr, 1 AS lv FROM emp_l WHERE empno = 7839
UNION ALL
SELECT c.empno, c.ename, c.mgr, p.lv + 1 AS lv
FROM w1 p, emp_l c
WHERE c.mgr = p.empno)
SEARCH DEPTH FIRST BY empno SET so
CYCLE empno SET ic TO '1' DEFAULT '0'
SELECT lv, empno, LPAD(' ', lv - 1, ' ') || ename AS ename, mgr, ic
FROM w1
ORDER BY so;
```

재귀 서브 쿼리 팩토링은 계층 전개와 무관한 열로도 루프 여부를 확인할 수 있다. 아래 쿼리는 deptno 열로 루프 여부를 확인한다. SCOTT과 FORD는 상위 노드인 JONSE의 deptno가 20이므로 둘 다 로프가 발생한 것으로 인식된다.  

```sql
WITH w1 (empno, ename, mgr, deptno, lv) AS (
SELECT empno, ename, mgr, deptno, 1 AS lv FROM emp WHERE mgr IS NULL
UNION ALL
SELECT c.empno, c.name, c.mgr, c.deptno, p.lv + 1 AS lv
FROM w1 p, emp c
WHERE c.mgr = p.empno)
SEARCH DEPTH FIRST BY empno SET so
CYCLE deptno SET ic TO '1' DEFAULT '0'
SELECT lv, empno, LPAD(' ', lv - 1, ' ') || ename AS ename, mgr, deptno, ic
FROM w1
ORDER BY so;
```

### 16.3 고급 주제
<br/>
#### 16.3.1. 노드 제거
<br/>
CONNECT BY 절이나 WHERE 절에 조건을 기술하면 조건을 만족하지 않는 노드를 제거할 수 있다.  

아래 쿼리는 CONNECT BY 절에 empno <> 7698 조건을 기술했다. CONNECT BY 절에 조건을 기술하면 조건을 만족하지 않는 노드와 해당 노드의 모든 하위 노드가 제거된다. 계층 전개 시점에 노드가 제거되기 때문에 하위 노드까지 제거되는 것이다.  

```sql
SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr
FROM emp
START WITH mgr IS NULL
CONNECT BY mgr = PRIOR empno
AND empno <> 7698;
```

아래 쿼리는 WHERE 절에 empno <> 7698 조건을 기술했다. WHERE 절에 조건을 기술하면 조건을 만족하지 않는 노드만 제거된다. CONNECT BY 절이 수행된 후 WHERE 절이 수행되기 때문이다.  

```sql
ELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr
FROM emp
WHERE empno <> 7698
START WITH mgr IS NULL
CONNECT BY mgr = PRIOR empno;
```

#### 16.3.2. 다중 루트 노드
<br/>
계층 쿼리 절은 1개 이상의 루트 노드를 가질 수 있다.  

아래 쿼리는 job이 MANAGER인 행으로 루트 노드를 생성하고, 순방향으로 계층을 전개한다. JONES, BLAKE, CLARK이 루트 노드로 생성된다.  

```sql
SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr
FROM emp
START WITH job = 'MANAGER'
CONNECT BY mgr = PRIOR empno;
```

아래 쿼리는 20번 부서에서 부하가 없는 사원으로 루트 노드를 생성하고, 역방향으로 계층을 전개한다. SMITH와 ADAMS가 루트 노드로 생성된다.  

```sql
SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, mgr
FROM emp a
START WITH deptno = 20
AND NOT EXISTS (SELECT 1 FROM emp x WHERE x.mgr = a.empno)
CONNECT BY empno = PRIOR mgr;
```

#### 16.3.3. 다중 속성 순환 관계
<br/>
순환 관계는 1개 이상의 속성을 관계 속성으로 가질 수 있다.  
#### 16.3.4. 계층 쿼리와 조인
<br/>
계층 쿼리는 세 가지 방식으로 조인을 수행할 수 있다.  

아래 쿼리는 계층을 전개한 후 조인을 수행한다. 인라인 뷰를 사용한 조인과 동일하다.  

```sql
SELECT a.lv, a.empno, a.ename, b.dname
FROM (SELECT LEVEL AS lv, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename
             , deptno, ROWNUM AS rn
      FROM emp
      START WITH mgr IS NULL
      CONNECT BY mgr = PRIOR empno) a
      , dept b
WHERE b.deptno = a.deptno
ORDER BY a.rn;
```

아래 쿼리는 조인을 수행한 후 계층을 전개한다. b.loc = 'NEW YORK' 조건처럼 계층을 전개할 대상을 먼저 정의해야 할 때 사용할 수 있다.  

```sql
SELECT LEVEL AS lv, ym, empno, LPAD(' ', LEVEL - 1, ' ') || ename AS ename, deptno, dname
FROM (SELECT a.*, b.dname
      FROM emp a, dept b
      WHERE b.deptno = a.deptno
        AND b.loc = 'NEW YORK')
START WITH mgr IS NULL
CONNECT BY mgr = PRIOR empno;
```

WHERE 절에 조인 조건을 기술하면 조인을 수행한 후 계층을 전개한다. 위 쿼리처럼 인라인 뷰를 사용하는 편이 명시적이다.  

```sql
SELECT LEVEL AS lv, a.empno, LPAD(' ', LEVEL - 1, ' ') || a.ename AS ename, a.deptno, b.dname
FROM emp a, dept b
WHERE b.deptno = a.deptno
START WITH a.mgr IS NULL
CONNECT BY a.mgr = PRIOR a.empno;
```

ANSI 조인 문법도 조인을 수행한 후 계층을 전개한다.  

```sql
SELECT LEVEL AS lv, a.empno, LPAD(' ', LEVEL - 1, ' ') || a.ename AS ename, a.deptno, b.dname
FROM emp a
JOIN dept b
ON b.deptno = a.deptno
START WITH a.mgr IS NULL
CONNECT BY a.mgr = PRIOR a.empno;
```

아래 쿼리는 계층 전개 시점에 조인을 수행한다. START WITH 절과 CONNECT BY 절에 조인 조건을 기술해야 한다. b.loc = 'DALLAS' 조건처럼 계층 전개 중 노드를 제한해야 할 때 사용할 수 있다.

```sql
SELECT a.*, b.dname
FROM emp a, dept b
START WITH a.mgr IS NULL
AND b.deptno = a.deptno
CONNECT BY a.mgr = PRIOR a.empno
AND b.deptno = a.deptno
AND b.loc = 'DALLAS';
```

### 16.4. 활용 예제
<br/>
#### 16.4.1. 순번 생성
<br/>
계층 쿼리를 사용하여 순번을 가진 테이블을 생성할 수 있다. 행 복제 시 해당 기법을 활용할 수 있다.  

아래 쿼리는 100까지의 순번을 반환한다. START WITH 절을 생략했기 때문에 DUAL 테이블의 전체 행이 루트 노드로 생성된다. DUAL 테이블이 1행이므로 로트 노드가 1행으로 생성되고, 1행과 1행을 카티션 곱한 결과는 1행이므로 LEVEL <= 100 조건을 만족할 때까지 1행씩 레벨이 증가하는 것이다.  

```sql
SELECT LEVEL AS lv FROM DUAL CONNECT BY LEVEL <= 100;
```

아래와 같이 XMLTABLE 함수를 사용해도 동일한 결과를 얻을 수 있다.  

```sql
SELECT ROWNUM AS rn FROM XMLTABLE('1 to 100');
```

#### 16.4.2. 변경 이력
<br/>
계층 쿼리를 응요하면 값의 변경 이력을 확인할 수 있다.  

아래 표는 코드 변경 이력을 표현하고 있다.  

<table>
  <thead>
    <tr>
      <td>최초</td>
      <td>205001</td>
      <td>205004</td>
      <td>205007</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>A</td>
      <td>B</td>
      <td>C</td>
      <td>D</td>
    </tr>
    <tr>
      <td>I</td>
      <td>J</td>
      <td>K</td>
      <td></td>
    </tr>
    <tr>
      <td>X</td>
      <td>Y</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 쿼리는 코드의 최종 변경 코드를 조회한다. START WITH 절이 생략되어 전체 행이 루트 노드로 생성된다. CONNECT&#95;BY&#95;ROOT 연산자로 루트 노드를 조회하고, CONNECT&#95;BY&#95;ISLEAF 슈도 칼럼으로 리프 노드만 조회했다.  

```sql
SELECT bf, af, ym
FROM (SELECT ym, CONNECT_BY_ROOT bf AS bf, af, CONNECT_BY_ISLEAF AS lf
      FROM t1
      CONNECT BY bf = PRIOR af)
WHERE lf = 1
ORDER BY 1;
```

아래 쿼리는 최초 코드의 변경 정보를 조회한다. 최초 코드로 루트 노드를 생성하기 위해 START WITH 절에서 변경 전 코드가 존재하지 않는 행을 조회한 후 순방향으로 계층을 전개했다.  

```sql
SELECT bf, cd, ym, cn
FROM (SELECT CONNECT_BY_ROOT bf AS bf)
             , SUBSTR(SYS_CONNECT_BY_PATH(af, ','), 2) AS cd
             , SUBSTR(SYS_CONNECT_BY_PATH(ym, ','), 2) AS ym
             , LEVEL AS cn
             , CONNECT_BY_ISLEAF AS lf
      FROM t1 a
      START WITH NOT EXISTS (SELECT 1 FROM t1 x WHERE x.af = a.bf)
      CONNECT BY bf = PRIOR af)
WHERE lf = 1
ORDER BY 1;
```

아래 쿼리는 최종 코드의 변경 정보를 조회한다. 최종 코드로 루트 노드를 생성하기 위해 START WITH 절에서 변경 후 코드가 존재하지 않는 행을 조회한 후 역방향으로 계층을 전개했다.  

```sql
SELECT af, cd, ym, cn
FROM (SELECT CONNECT_BY_ROOT af AS af
             , SUBSTR(SYS_CONNECT_BY_PATH(bf, ','), 2) AS cd)
             , SUBSTR(SYS_CONNECT_BY_PATH(ym, ','), 2) AS ym)
             , LEVEL AS cn
             , CONNECT_BY_ISLEAF AS lf
      FROM t1 a
      START WITH NOT EXISTS (SELECT 1 FROM t1 x WHERE x.bf = a.af)
      CONNECT BY bf = PRIOR af)
WHERE lf = 1
ORDER BY 1;
```

#### 16.4.3. 생성 순서
<br/>
계층 쿼리를 사용하면 순차적으로 계산되는 계정의 생성 순서를 결정할 수 있다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

<table>
  <thead>
    <tr>
      <td>CD</td>
      <td>C1</td>
      <td>C2</td>
      <td>C3</td>
      <td>C4</td>
    </tr>
  </thead>
  <thead>
    <tr>
      <td>A</td>
      <td>B</td>
      <td>C</td>
      <td>D</td>
      <td>E</td>
    </tr>
    <tr>
      <td>B</td>
      <td>F</td>
      <td>G</td>
      <td>H</td>
      <td></td>
    </tr>
    <tr>
      <td>C</td>
      <td>I</td>
      <td>J</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>D</td>
      <td>K</td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>E</td>
      <td>B</td>
      <td>C</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>F</td>
      <td>C</td>
      <td>D</td>
    </tr>
  </thead>
</table>
<br/><br/>

계정(cd)은 계산계정(c1, c2, c3, c4)으로 계산된다. 계정 A는 계정 B, C, D, E로 계산되고, 계정 B는 계정 F, G, H로 계산된다. 계정 A를 계산히기 위해서는 계정 B를 먼저 생성해야 한다.  

아래 쿼리로 계정의 생성 순서를 결정할 수 있다. 먼저 생성할 필요가 없는 계산계정으로 생성되는 계정을 루트 노드로 생성한 후 역방향으로 계층을 전개했다. 계정 A는 다섯 번째로 생성해야 한다.  

```sql
SELECT cd, MAX(LEVEL) AS lv
FROM t1 a
START WITH NOT EXISTS (SELECT 1 FROM t1 x WHERE x.cd IN (a.c1, a.c2, a.c3, a.c4))
CONNECT BY PRIOR cd IN (c1, c2, c3, c4)
GROUP BY cd
ORDER BY 2, 1;
```

#### 16.4.3. 누적 연산
<br/>
재귀 서브 쿼리 팩토링을 사용하면 상위 노드의 값을 누적 연산할 수 있다.  

```sql
WITH w1 (empno, ename, mgr, sal, lv, c1) AS (
SELECT empno, ename, mgr, sal, 1 AS lv, sal AS c1
FROM emp
WHERE mgr IS NULL
UNION ALL
SELECT c.empno, c.ename, c.mgr, c.sal, p.lv + 1 AS lv, p.c1 + c.sal AS c1
FROM w1 p, emp c
WHERE c.mgr = p.empno)
SEARCH DEPTH FIRST BY empno SET so
SELECT lv, empno, LPAD(' ', lv - 1, ' ') || ename AS ename, mgr, sal, c1
FROM w1
ORDER BY so;
```

계층 쿼리 절은 부모 노드의 값만 참조할 수 있기 때문에 누적 연산이 불가능하다.  
