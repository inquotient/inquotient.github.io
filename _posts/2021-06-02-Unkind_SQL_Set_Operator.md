---
title: 집합 연산자
categories:
- Unkind_SQL
feature_text: |
  ## 13. 집합 연산자
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

#### 13.3.1. OR 조건 성능 개선
<br/>
OR 조건을 사용한 쿼리는 다수의 조건으로 인해 쿼리의 성능이 저하될 수 있다. UNION ALL 연산자로 데이터 집합을 분리함으로써 쿼리의 성능을 개선할 수 있다.  

#### 13.3.2. 옵션 조건 성능 개선
<br/>
UNION ALL 연산자로 옵션 조건의 성능을 개선할 수 있다.  

#### 13.3.3. FULL OUTER JOIN 성능 개선
<br/>
11.1 이전 버전에서 FULL OUTER JOIN을 수행하면 조인이 여러 번 수행되어 쿼리의 성능이 저하되었다. 성능 개선을 위해 FULL OUTER JOIN을 UNION ALL 연산자로 변경하는 기법을 사용할 수 있다.  

모든 FULL OUTER JOIN을 UNION ALL 연산자로 변경할 수 있는 것은 아니다. FULL OUTER JOIN의 조인 차수의 1:1인 경우만 FULL OUTER JOIN을 UNION ALL 연산자로 변경할 수 있다. 조인 차수가 1:M이면 M쪽 결과 집합이 1쪽으로 그룹핑되기 때문에 쿼리의 결과가 달라진다.  

#### 13.3.4. INTERSECT, MINUS 연산자 성능 개선
<br/>
UNION, INTERSECT, MINUS 연산자는 중복 값을 제거하기 위해 데이터 집합을 정렬한다. 대량 데이터에 대한 소트가 발생하면 쿼리 성능이 저하될 수 있다. INTERSECT, MINUS 연산자의 서브 쿼리를 사용하여 소트 발생량을 감소시킬 수 있다.  

INTERSECT 연산자는 EXISTS 조건으로 변경할 수 있다. DISTINCT 키워드로 중복 값을 제거해야 한다. 널을 허용하는 열은 LNNVL 함수를 사용해야 한다.  

```sql
SELECT c1, c2 FROM t1
INTERSECT
SELECT c1, c2 FROM t2;
```

```sql
SELECT DISTINCT a.c1, a.c2
FROM t1 a
WHERE EXISTS (SELECT 1
              FROM t2 x
              WHERE x.c1 = a.c1
              AND LNNVL(x.c2 <> a.c2));
```

MINUS 연산자는 NOT EXISTS 조건으로 변경할 수 있다. 열이 많을수록 쿼리가 길어지는 단점이 있다.  

```sql
SELECT c1, c2 FROM t1
MINUS
SELECT c1, c2 FROM t2;
```

```sql
SELECT DISTINCT a.c1, a.c2
FROM t1 a
WHERE NOT EXISTS (SELECT 1
                  FROM t2 x
                  WHERE x.c1 = a.c1
                  AND LNNVL(x.c2 <> a.c2));
```
