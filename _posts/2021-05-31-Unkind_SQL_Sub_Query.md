---
title: 서브 쿼리
categories:
- Unkind_SQL
feature_text: |
  ## 12. 서브 쿼리
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 12.1. 중첩 서브 쿼리
<br/>
WHERE 절과 HAVING 절에 사용하는 서브 쿼리다.  

+ 비상관 서브 쿼리(uncorrelated subquery)  
메인 쿼리와 관계 없음
+ 상관 서브 쿼리(uncorrelated subquery)  
메인 쿼리와 관계 있음
+ 단일 행 서브 쿼리  
서브 쿼리가 단일 행을 반환
+ 서브 쿼리가 다중 행을 반환

#### 12.1.1. 비상관 서브 쿼리
<br/>
메인 쿼리와 상관 관계가 없는 서브 쿼리다. 서브 쿼리의 WHERE 절에 메인 쿼리와의 조인 조건이 존재하지 않는다.  \\

##### 12.1.1.1. 단일 행 비상관 서브 쿼리
<br/>
단일 행을 반환하는 비상관 서브 쿼리로 주로 비교 조건을 사용한다.  

##### 12.1.1.2. 다중 행 비상관 서브 쿼리
<br/>
다중 행을 반환하는 비상관 서브 쿼리로 주로 IN 조건을 사용한다.  

NOT IN 조건에 널이 포함된 다중 열을 사용하면 의도하지 않은 결과가 반환된다.  

#### 12.1.2. 상관 서브 쿼리
<br/>
메인 쿼리와 상관 관계가 있는 서브 쿼리다. 서브 쿼리의 WHERE 절에 메인 쿼리와의 조인 조건이 존재한다.  

##### 12.1.2.1. 단일 행 상관 서브 쿼리
<br/>
단일 행을 반환하는 상관 서브 쿼리다. 주로 비교 조건을 사용한다.  

##### 12.1.2.2. 다중 행 상관 서브 쿼리
<br/>
다중 행을 반환하는 상관 서브 쿼리다. 주로 EXISTS 조건을 사용한다.  

테이블이나 테이블 별칭으로 한정하지 않은 열은 쿼리 블록 내의 테이블에 해당 열이 존재하는지를 먼저 확인한다.  

+ 세미 조인(semi join)  
조인이 1번이라도 성공하면 행을 반환하는 조인 방식  
+ 안티 조인(anti join)  
조인이 1번이라도 성공하면 행을 반환하지 않는 조인 방식  

#### 12.1.3. 사용 기준
<br/>
서브 쿼리의 결과를 리터럴과 비교하면 쿼리의 성능이 저하될 수 있다.  

NOT IN 조건과 NOT EXISTS 조건은 서브 쿼리의 널 존재 여부에 따라 결과가 달라질 수 있다.  

NOT EXISTS 조건은 다중 열에 널이 존재하더라도 정상적인 결과를 반환한다. NOT IN 조건보다 NOT EXISTS 조건을 사용하는 편이 바람직하다.  

### 12.2 스칼라 서브 쿼리
<br/>
SELECT 절에 사용하는 서브 쿼리로 결과가 없으면 널을 반환한다.  

### 12.3. 인라인 뷰
<br/>
FROM 절에 사용하는 서브 쿼리  

### 12.4. 사용기준
<br/>
사용 기준은 조인 차수와 관련이 있다.  

+ 조인  
조인 기준의 행이 줄어들거나 늘어날 수 있음
+ 중첩 서브 쿼리  
메인 쿼리의 행이 줄어들 수 있지만 늘어나지는 않음
+ 스칼라 서브 쿼리  
메인 쿼리의 행이 변하지 않음
+ 인라인 뷰  
메인 쿼리의 행이 줄어들거나 늘어날 수 있음(조인과 동일)  

스칼라 서브 쿼리는 메인 쿼리와 서브 쿼리의 조인 차수가 1:M일 때 사용하는 것이 일반적이다.  

### 12.5. WITH 절
<br/>
#### 12.5.1. SUBQUERY FACTORING 절
<br/>
```
WITH query_name AS (subquery) [, query_name AS (subquery)]
SELECT * FROM query_name;
```
SUBQUERY FACTORING 절을 사용하면 서브 쿼리에 이름을 부여하고, 이름이 부여된 서브 쿼리를 메인 쿼리에서 반복 사용할 수 있다.  

SUBQUERY FACTORING 절에 기술한 서브 쿼리를 2번 이상 사용하면 서브 쿼리의 결과 집합이 임시 영역에 저장된다.  

#### 12.5.2. PL/SQL 선언
<br/>
12.1 버전부터 WITH 절에 PL/SQL 함수와 프로시저를 선언할 수 있다.  

### 12.6. 신규 기능
<br/>
#### 12.6.1. LATERAL 인라인 뷰
<br/>
LATERAL 인라인 뷰를 사용하면 인라인 뷰에 메인 쿼리의 열을 기술할 수 있다.  

```sql
SELECT a.deptno, a.dname, b.empno, b.ename
FROM dept a,
     LATERAL
     (SELECT x.* FROM emp x WHERE x.deptno = a.deptno) b;
```

LATERAL 인라인 뷰 뒤에 (+) 기호를 기술하면 아우터 조인으로 조인된다.  

```sql
SELECT a.deptno, a.dname, b.empno, b.ename
FROM dept a,
     LATERAL
     (SELECT x.* FROM emp x WHERE x.deptno = a.deptno)(+) b;
```

LATERAL 인라인 뷰에 GROUP BY 절이 없는 집계 함수를 사용하면 메인 쿼리의 모든 행이 반환된다. GROUP BY 절이 없는 집계 함수는 항상 결과를 반환하기 때문이다.  

```sql
SELECT a.deptno, a.dname, b.sal
FROM dept a,
     LATERAL
     (SELECT SUM(x.sal) AS sal FROM emp x WHERE x.deptno = a.deptno) b;
```

GROUP BY 절을 기술하면 의도한 결과를 얻을 수 있다.  

```sql
SELECT a.deptno, a.dname, b.sal
FROM dept a,
     LATERAL
     (SELECT SUM(x.sal) AS sal FROM emp x WHERE x.deptno = a.deptno GROUP BY()) b;
```

#### 12.6.2. CROSS APPLY 절
<br/>
CROSS APPLY 절은 CROSS JOIN의 변형이다.  

아래는 CROSS APPLY 절을 사용한 쿼리다. LATERAL 인라인 뷰의 이너 조인과 결과가 동일하다.  

```sql
SELECT a.deptno, a.dname, b.empno, b.ename
FROM dept a,
     CROSS APPLY
     (SELECT x.* FROM emp x WHERE x.deptno = a.deptno) b;
```

#### 12.6.3. OUTER APLLY 절
<br/>
OUTER APLLY 절은 LEFT OUTER JOIN의 변형이다.  

아래는 OUTER APLLY 절을 사용한 쿼리다. LATERAL 인라인 뷰의 아우터 조인과 결과가 동일하다.  

```sql
SELECT a.deptno, a.dname, b.empno, b.ename
FROM dept a,
     OUTER APPLY
     (SELECT x.* FROM emp x WHERE x.deptno = a.deptno) b;
```
