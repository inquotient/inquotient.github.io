---
title: WHERE 절
categories:
- Unkind_SQL
feature_text: |
  ## 7. WHERE 절
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

#### 7.6.1. LNNVL 함수
<br/>
```
LNNVL(condition)
```

condition이 FALSE나 UNKNOWN이면 TRUE, TRUE면 FALSE를 반환한다. WHERE 절과 CASE 표현식에 사용할 수 있다.  

아래는 LNNVL 함수를 사용한 쿼리다. comm이 널이거나 0인 행을 조회한다.  

```sql
SELECT ename, comm FROM emp WHERE LNNVL(comm <> 0);
```

#### 7.7. 조건 우선순위
<br/>
(1) 연산자  
(2) 비교 조건(=, <>, >, <, >=, <=)  
(3) IN 조건, LIKE 조건, BETWEEN 조건, 널 조건  
(4) 논리 조건(NOT)
(5) 논리 조건(AND)
(6) 논리 조건(OR)  
