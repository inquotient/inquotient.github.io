---
title: Top-N 쿼리
categories:
- Unkind_SQL
feature_text: |
  ## 15. Top-N 쿼리
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 15.1. 기본 문법
<br/>
#### 15.1.1. ROWNUM 방식
<br/>
ROWNUM 방식은 오라클 데이터베이스의 전통적인 Top-N 쿼리 방식이다. ORDER BY 절로 행을 정렬하고, 정렬된 행을 ROWNUM 슈도 칼럼으로 제한한다.  

RONUM 슈도 칼럼은 행이 반환되는 순서대로 순번을 반환한다. 1부터 시작하고 행이 반환될 때마다 순번이 증가한다.  

ROWNUM 슈도 칼럼은 1부터 시작하고 행이 반환될 때마다 순번이 증가한다.  

ROWNUM 슈도 칼럼은 < 조건이나 <= 조건을 사용해야 한다.  

아래 쿼리는 인라인 뷰에 의해 ORDER BY 절이 수행된 후 WHERE 절이 수행된다. 아래 쿼리가 ROWNUM 방식의 Top-N 쿼리다.  

```sql
SELECT empno, sal, ROWNUM AS rn
FROM (SELECT empno, sal FROM emp ORDER BY sal, empno)
WHERE ROWNUM <= 5;
```

아래 쿼리는 상위 6행에서 10행까지의 행을 조회한다. 한 페이지(v&#95;pr)를 5행으로 보면 두 번째 페이지(v&#95;pn)를 조회한 것이다. 이런 쿼리를 페이징 쿼리(pagination query)라고 한다.  

```sql
VAR v_pr NUMBER = 5;
VAR v_pn NUMBER = 2;

SELECT empno, sal, rn
FROM (SELECT empno, sal, ROWNUM AS rn
      FROM (SELECT empno, sal FROM emp ORDER BY sal, empno)
      WHERE ROWNUM <= :v_pr * :v_pn)
WHERE rn >= (:v_pr * (:v_pn - 1)) + 1;
```

아래 쿼리는 위 쿼리와 결과가 동일하지만 쿼리의 성능이 저하된다. ROWNUM 방식의 Top-N 쿼리는 ROWNUM 슈도 칼럼을 WHERE 절에 직접 기술해야 한다.  

```sql
SELECT empno, sal, rn
FROM (SELECT empno, sal, ROWNUM AS rn
      FROM (SELECT empno, sal FROM emp ORDER BY sal, empno))
WHERE rn BETWEEN (:v_pr * (:v_pn - 1)) + 1 AND :v_pr * v_pn;
```

경품 추첨 등 무작위로 n개의 행을 조회해야 하는 경우 ORDER BY 절에 DBMS&#95;RANDOM.VALUE 함수를 사용할 수 있다.  

```sql
SELECT empno, sal
FROM (SELECT empno, sal FROM emp ORDER BY DBMS_RANDOM.VALUE)
WHERE ROWNUM <= 3;
```

위 쿼리는 문맥 전환(context switching)에 의한 성능 저하가 발생할 수 있다. DBMS&#95;RANDOM.VALUE 함수 대신 ORA&#95;HASH 함수를 사용하면 쿼리의 성능을 개선할 수 있다. 테이블의 크기가 크다면 SAMPLE 절사용도 고려할 수 있다.  

```sql
SELECT empno, sal
FROM (SELECT empno, sal
      FROM emp -- SAMPLE BLOCK(1)
      ORDER BY ORA_HASH(empno, TO_CHAR(SYSTEMTIMESTAMP, 'FF9')
                             , TO_CHAR(SYSTEMTIMESTAMP, 'FF9')))
WHERE ROWNUM <= 3;
```

아래와 같이 존재 여부를 확인하는 쿼리에도 ROWNUM 슈도 칼럼을 사용할 수 있다. sal가 3000 이상인 행이 3행이지만 1행만 읽고 결과를 얻을 수 있다.  

```sql
SELECT NVL(MAX('Y'), 'N') AS c1 FROM emp WHERE sal >= 3000 AND ROWNUM <= 1;
```

중첩 서브 쿼리에 ROWNUM 슈도 칼럼을 사용하면 쿼리 변환(query transformation)에 제약이 생겨 쿼리의 성능이 저하될 수 있다.  

스칼라 서브 쿼리의 정렬 조건이 반환 값과 다른 경우 아래와 같이 KEEP 절을 사용해야 한다.  

```sql
SELECT a.deptno
       , (SELECT MAX(x.empno) KEEP(DENSE_RANK FIRST ORDER BY sal DESC)
          FROM emp x
          WHERE x.deptno = a.deptno) AS empno
FROM dept a;
```

KEEP 절을 사용하면 쿼리의 성능이 저하될 수 있다. 쿼리의 성능이 저하되는 경우 아래와 같이 스칼라 서브 쿼리를 중첩하여 ROWNUM 슈도 칼럼을 사용할 수 있다. 12.1 버전부터는 두 번째 쿼리와 같은 방식을 사용할 수 있다.  

```sql
SELECT a.deptno
       , (SELECT x.empno
          FROM (SELECT empno, deptno
                FROM emp
                ORDER BY sal DESC, empno DESC) x                               
          WHERE x.deptno = a.deptno
            AND ROWNUM <= 1) AS empno
FROM dept a;
```

```sql
SELECT a.deptno
       , (SELECT empno
          FROM (SELECT x.empno, x.deptno
                FROM emp x
                WHERE x.deptno = a.deptno
                ORDER BY x.sal DESC, x.empno DESC) x                               
          WHERE ROWNUM <= 1) AS empno
FROM dept a;
```

#### 15.1.2. 분석 함수 방식
<br/>
아래는 ROW&#95;NUMBER 함수를 사용한 Top-N 쿼리다.  

```sql
SELECT *
FROM (SELECT empno, sal, ROW_NUMBER() OVER(ORDER BY sal, empno) AS rn
      FROM emp)
WHERE rn <= 5
ORDER BY sal, empno;
```

아래는 ROW_NUMBER 함수를 사용한 페이징 쿼리다. ROWNUM 방식에 비해 간결하게 작성할 수 있다.  

```sql
SELECT *
FROM (SELECT empno, sal, ROW_NUMBER() OVER(ORDER BY sal, empno) AS rn FROM emp)
WHERE rn BETWEEN (:v_pr * (:v_pn - 1)) + AND :v_pr * :v_pn
ORDER BY sal, empno;
```

전체 건수가 필요한 경우 아래와 같이 작성할 수도 있다. COUNT 분석 함수에서 전체 행을 조회해야 하기 때문에 쿼리의 성능이 저하될 수 있다. 성능 개선을 위해 첫 페이지 로딩 시에만 전체 건수를 조회하는 쿼리를 별도로 수행하기도 한다.  

```sql
SELECT empno, sal, rn, cn
FROM (SELECT empno, sal
             , COUNT(*) OVER() AS cn
             , ROW_NUMBER() OVER(ORDER BY sal, empno) AS rn
      FROM emp)
WHERE CEIL(rn / :v_pr) = :v_pn
ORDER BY sal, empno;
```

PERCENT_RANK 함수를 사용하면 백분율에 의한 Top-N 쿼리를 작성할 수 있다. 아래 쿼리는 상위 25%의 행을 반환한다. 위 쿼리와 동일한 이유로 쿼리의 성능이 저하될 수 있다.  

```sql
SELECT empno, sal, pr
FROM (SELECT empno, sal,
             , PERCENT_RANK() OVER(ORDER BY sal, empno) AS pr
      FROM emp)
WHERE pr <= 0.25
ORDER BY sal, empno;
```

#### 15.1.3. ROW LIMITING 절
<br/>
```
[OFFSET offset {ROW | ROWS}]
[FETCH {FIRST | NEXT} [{rowcount | percent PERCENT}] {ROW | ROWS} {ONLY | WITH TIES}]
```
12.1 버전부터 ROW LIMITING 절로 Top-N 쿼리를 작성할 수 있다. ROW LIMITING 절은 ANSI 표준 SQL 문법이다.  

아래는 ROW LIMITING 절의 구문이다. ROW LIMITING 절은 ORDER BY 절 다음에 기술하며, ORDER BY 절과 함께 수행된다. ROW와 ROWS는 구분하지 않아도 된다.  

+ OFFSET offset : 건너뛸 행의 개수를 지정
+ FETCH : 반환할 행의 개수나 백분율을 지정
+ ONLY : 지정된 행의 개수나 백분율만큼 행을 반환
+ WITH TIES : 마지막 행에 대한 동순위를 포함해서 반환  

아래는 ROW LIMITING 절을 사용한 Top-N 쿼리다.  

```sql
SELECT empno, sal FROM emp ORDER BY sal, empno FETCH FIRST 5 ROWS ONLY;
```

아래는 ROW LIMITING 절을 사용한 페이징 쿼리다. 쿼리를 간결하게 작성할 수 있다.  

```sql
SELECT empno, sal
FROM emp
ORDER BY sal, empno
OFFSET :v_pr * (:v_pn - 1) ROWS FETCH NEXT :v_pr ROWS ONLY;
```

아래와 같이 OFFSET만 기술하면 건너뛴 행 이후 전체 행이 반환된다.  

```sql
SELECT empno, sal FROM emp ORDER BY sal, empno OFFSET 5 ROWS;
```

PERCENT 키워드를 사용하면 반환할 행의 백분율을 지정할 수 있다. 아래 쿼리는 상위 25%의 행을 반환한다.  

```sql
SELECT empno, sal FROM emp ORDER BY sal, empno FETCH FIRST 25 PERCENT ROWS ONLY;
```

아래 쿼리는 RANK 함수를 사용한 Top-N 쿼리와 동일하게 동작한다.  

```sql
SELECT empno, sal FROM emp ORDER BY sal FETCH FIRST 6 ROWS WITH TIES;
```

### 15.2. 고급 주제
<br/>
#### 15.2.1. Top-N 쿼리와 조인
<br/>
아래 쿼리는 조인 후 Top-N 처리를 수행한다. emp 테이블이 14건이므로 조인을 14번 수행하고 2건의 결과를 반환한다. emp 테이블과 dept 테이블은 조인 차수가 M:1이고, 아우터 조인으로 조인했기 때문에 emp 테이블을 Top-N 처리한 후 dept 테이블을 조인해도 동일한 결과를 얻을 수 있다.  

```sql
SELECT empno, sal, deptno, dname
FROM (SELECT a.empno, a.sal, a.deptno, b.dname
      FROM emp a, dept b
      WHERE b.deptno(+) = a.deptno
      ORDER BY a.sal, a.empno)
WHERE ROWNUM <= 2;
```

아래 쿼리는 위 쿼리와 결과가 동일하다. 인라인 뷰에서 Top-N 처리한 결과 집합으로 dept 테이블을 아우터 조인한다. 조인은 2번만 수행된다.  

```sql
SELECT a.empno, a.sal, a.deptno, b.dname
FROM (SELECT *
      FROM (SELECT empno, sal, deptno FROM emp ORDER BY sal, empno)
      WHERE ROWNUM <= 2) a
      , dept b
WHERE b.deptno(+) = a.deptno
ORDER BY a.sal, a.empno;
```

1개의 열만 조회할 경우 Top-N 처리 후 스칼라 서브 쿼리를 사용할 수 있다.  

```sql
SELECT a.empno, a.sal, a.deptno
       , (SELECT x.dname, FROM dept x WHERE x.deptno = a.deptno) AS dname
FROM (SELECT empno, sal, deptno FROM emp ORDER BY sal, empno) a
WHERE ROWNUM <= 2;
```

#### 15.2.2. Top-N 쿼리와 UNION ALL 연산자
<br/>
아래 쿼리는 dept 테이블과 emp 테이블을 UNION ALL 연산자로 연결한 결과 집합에 Top-N 처리를 수행한다. 결과 집합을 정렬해야 하므로 소트 부하가 발생할 수 있다.  

```sql
SELECT *
FROM (SELECT 1 AS tp, deptno AS no, dname AS name FROM dept)
      UNION ALL
      SELECT 2 AS tp, empno AS no, ename AS name FROM emp
      ORDER BY tp, no)
WHERE ROWNUM <= 3;
```

아래 쿼리처럼 데이터 집합 별로 Top-N 처리를 수행하면 소트 부하를 경감시킬 수 있다. UNION ALL 연산자는 순차적으로 수행된다. 아래 쿼리는 emp 테이블을 읽지 않고 결과를 반환한다.  

```sql
SELECT *
FROM (SELECT *
      FROM (SELECT 1 AS tp, deptno AS no, dname AS name FROM dept ORDER BY no)
      WHERE ROWNUM <= 3
      UNION ALL
      SELECT *
      FROM (SELECT 2 AS tp, empno AS no, ename AS name FROM emp ORDER BY no)
      WHERE ROWNUM <= 3)
WHERE ROWNUM <= 3;
```
