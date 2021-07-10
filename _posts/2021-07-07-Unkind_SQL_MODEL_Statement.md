---
title:  MODEL 절
categories:
- Unkind_SQL
feature_text: |
  ## 26. MODEL 절
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

MODEL 절은 쿼리 결과로 다차원 배열을 생성하고, 배열 표현식을 통해 새로운 값을 계산한다. MODEL 절을 사용하면 기본 연산에서 연립 방정식까지 다양한 연산을 수행할 수 있다. MODEL 절은 10.1 버전부터 사용할 수 있다.  

+ 모델링 작업
전통적인 모델링 작업은 엑셀과 같은 스프레드시트(spreadsheet)를 사용한다. 데이터베이스에서 스프레드시트로 데이터를 전송하고, 수식을 통해 값을 계산한 후, 계산한 결과를 데이터베이스에 저장하는 과정을 반복한다. 스프레드시트는 처리할 수 있는 데이터 크기에 한계가 있으며, 수식의 변경에 따라 일관성 없는 데이터가 생성될 수 있다. MODEL 절을 사용하면 스프레드시트로 데이터를 전송하지 않아도 되기 때문에 데이터 크기의 제한 없이 모델링 작업을 수행할 수 있다. 가장 큰 장점은 스프레드시트의 수식이 쿼리로 대체되기 때문에 일관성 있는 데이터를 생성할 수 있다는 점이다.  

### 26.1. 기본 문법
<br/>
아래는 MODEL 절의 구문이다. MODEL 절은 GROUP BY 절과 ORDER BY 절 사이에 기술하며, GROUP BY 절과 HAVING 절이 수행된 후 수행된다. SELECT 절과 ORDER BY 절은 MODEL 절이 수행된 후 수행된다.  

```
MODEL
  [cell_reference_options] -- 셀 참조 옵션 (전역)
  [return_rows_options] -- 행 반환 옵션
  [reference_model]... -- 참조 모델
  [MAIN main_model_name] -- MAIN 절
  [PARTITION BY (expr [c_alias] [, expr [c_alias]]...)] -- PARTITION BY 절
   DIMENSION BY (expr [c_alias] [, expr [c_alias]]...) -- DEMENSION BY 절
   MEASURES (expr [c_alias] [, expr [c_alias]]...)]...) -- MEASURES 절
  [cell_reference_options] -- 셀 참조 옵션 (지역)
  [RULES [rule_options]] (rule [, rule]...) -- RULES 절 / 규칙 옵션
```

+ MAIN 절 : 메인 모델의 이름을 지정
+ PARTITION BY 절 : 외부 쿼리의 결과를 파티션으로 분할
+ DIMENSION BY 절 : 차원 열을 지정
+ MEASURES 절 : 계산 열을 지정
+ RULES 절 : MEASURES 절에 지정한 열을 규칙에 따라 계산  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE sales_view PURGE;

CREATE TABLE sales_view AS
SELECT e.country_name AS country, b.prod_name AS product, d.calendar_year AS year
     , SUM(a.amount_sold) AS sales, COUNT(a.amount_sold) AS cnt
     , MAX(d.calendar_year) KEEP (DENSE_RANK FIRST ORDER BY SUM(a.amount_sold) DESC) OVER(PARTITION BY e.country_name, b.prod_name) AS best_year
     , MAX(d.calendar_year) KEEP (DENSE_RANK LAST ORDER BY SUM(a.amount_sold) DESC) OVER(PARTITION BY e.country_name, b.prod_name) AS worst_year
FROM sh.sales a, sh.products b, sh.customers c, sh.times d, sh.countries e
WHERE b.prod_id = a.prod_id
AND c.cust_id = a.cust_id
AND d.time_id = a.time_id
AND e.country_id = c.country_id
GROUP BY e.country_id, b.prod_id, d.calendar_year
DESC sales_view;
```

아래는 sales&#95;view 테이블을 조회한 결과다.  

```sql
SELECT * FROM sales_view;
```

아래는 MODEL 절을 사용한 쿼리다.  

```sql
SELECT country, product, year, sales -- (1)
FROM sales_view
WHERE country IN ('Italy', 'Japan')
AND product IN ('Bounce', 'Y Box')
AND year IN (2000, 2001)
MODEL PARTITION BY (country) -- (2)
      DIMENSION BY (product, year) -- (3)
      MEASURES (sales) -- (4)
      RULES ( -- (5)
          sales['Bounce', 2002] = sales['Bounce', 2001] + sales['Bounce', 2000]
        , sales['Y Box', 2002] = sales['Y Box', 2001] + sales['Y Box', 2000])
ORDER BY country, product, year; -- (6)
```

위 쿼리는 다음과 같은 순서로 수행된다.  
(1) MODEL 절 위쪽의 오부 쿼리를 수행한다.  
(2) PARTITION BY 절로 외부 쿼리의 결과를 파티션으로 분할한다. 파티션은 독립적으로 계산된다.  
(3) DIMENSION BY 절로 차원 열을 지정한다. 차원 열은 파티션의 셀(cell)을 식별하기 위해 사용한다.  
(4) MEASURES 절로 계산 열을 지정한다. 계산 열은 하나의 셀로 식별된다.  
(5) RULES 절로 MEASURES 절에 지정한 열을 규칙에 따라 계산한다. 기존 값은 갱신하고, 신규 값은 삽입한다.  

#### 26.1.1. RULES 절
<br/>
RULES 절은 MEASURES 절에 지정한 열을 규칙에 따라 계산한다. 아래의 MODEL 절을 가정하고 예제를 살펴보자.  

```
MODEL
  PARTITION BY (country)
  DIMENSION BY (product, year)
  MEASURES (sales, best_year, worst_year)
  RULES (rule [, rule]...)
```

##### 26.1.1.1. 셀 참조
<br/>
MEASURES 절에 지정한 열은 셀 참조(cell referencing)로 조회할 수 있다. DIMENSION BY 절에 지정한 열에 대한 condition이나 expr을 지정할 수 있다.  

```
measure_column [{condition | expr} [, {condition | expr}]...]
```

+ condition : 기호 참조(symbolic reference)
+ expr : 위치 참조(positional reference)  

아래는 기호 참조를 사용한 셀 참조다.  

```
sales[product = 'Bounce', year = 2000]
```

기호 참조는 다양한 조건을 사용할 수 있다.  

```
sales[product = 'Bounce', year >= 2001]
```

아래는 위치 참조를 사용한 셀 참조다. 위치 참조는 등가 비교만 가능하다.  

```
sales['Bounce', 2001]
```

##### 26.1.1.2. 규칙
<br/>
규칙(rule)은 MEASURES 절에 지정한 열을 계산한다.  

```
cell_assignment = expr
```

###### 26.1.1.2.1. 단일 셀 참조
<br/>
단일 셀 참조는 규칙의 좌측이 단일 셀 참조, 우측은 단일 셀 참조나 리터럴로 구성된다.  

아래 규칙은 규칙의 우측에 리터럴을 사용했다.  

```
sales[product = 'Finding Fido', year = 2002] = 100000
```

아래 규칙은 규칙의 우측에 단일 셀 참조를 사용했다.  

```
sales['Bounce', 2002] = sales['Bounce', 2001] * 1.2
```

단일 셀 참조 간의 연산도 가능하다.  

```
sales[product = 'Finding Fido', year = 2002] = sales['Standard Mouse', year = 2001] + sales['Finding Fido', 2001] * 0.8
```

###### 26.1.1.2.2. 우측 다중 셀 참조
<br/>
우측 다중 셀 참조는 규칙의 좌측이 단일 셀 참조, 우측은 다중 셀 참조로 구성된다. 규칙의 우측이 다중 셀 참조이므로 집계 함수를 사용해야 규칙의 좌측에 단일 값을 할당할 수 있다.  

아래 규칙은 규칙의 우측에 MAX 함수를 사용했다.  

```
sales['Bounce', 2002] = MAX (sales)['Bounce', year BETWEEN 1998 AND 2001] + 100
```

아래 규칙은 규칙의 우측에 PERCENTILE&#95;DISC 함수를 사용했다.  

```
sales[product = 'Finding Fido', year = 2002] = PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY sales) [product IN ('Finding Fido', 'Student Mouse', 'Boat'), year < 2002] * 1.3
```

아래 규칙은 규칙의 우측에 AVG 함수를 사용했다.  

```
sales['Bounce', 2002] = AVG(sales * weight)['Bounce', year BETWEEN 1998 AND 2001]
```

###### 26.1.1.2.3. 좌측 다중 셀 참조
<br/>
좌측 다중 셀 참조는 규칙의 좌측이 다중 셀 참조, 우측은 단일 셀 참조나 리터럴로 구성된다.  

아래 규칙은 규칙의 좌측에 다중 셀 참조, 우측에 단일 셀 참조를 사용했다.  

```
sales['Standard Mouse', year > 2000] = sales['Finding Fido', year = 2000] * 0.2
```

###### 26.1.1.2.4. CV 함수
<br/>
CV 함수는 규칙의 좌측에 대한 현재 값(Current Value)을 반환한다. CV 함수는 규칙의 우측에 사용할 수 있다.  

```
CV([dimension_column])
```

아래는 CV 함수를 사용한 쿼리다.  

```
sales[product = 'Standard Mouse Pad', year > 2000] = sales[CV(product), CV(year)] + (sales['Finding Fido', 2000] * 0.2)
```

위 규칙은 아래 규칙과 논리적으로 동일하다. CV 함수는 위치 참조로 동작한다.  

```
  sales[product = 'Standard Mouse Pad', 2001] = sales['Standard Mouse Pad', 2001] + (sales['Finding Fido', 2000] * 0.2)
, sales[product = 'Standard Mouse Pad', 2002] = sales['Standard Mouse Pad', 2002] + (sales['Finding Fido', 2000] * 0.2)
```

CV 함수는 인수 없이도 사용할 수 있다.  

```
sales[product = 'Standard Mouse Pad', year > 2000] = sales[CV(), CV()] + (sales['Finding Fido', 2000] * 0.2)
```

CV 함수에 대한 연산도 가능하다.  

```
sales[product IN ('Finding Fido', 'Bounce'), year BETWEEN 2002 AND 2003] = sales[CV(product), CV(year) - 10] * 2
```

위 규칙은 아래 규칙과 논리적으로 동일하다.  

```
sales['Finding Fido', 2002] = sales['Finding Fido', 1992] * 2
, sales['Finding Fido', 2003] = sales['Finding Fido', 1993] * 2
, sales['Bounce', 2002] = sales['Bounce', 1992] * 2
, sales['Bounce', 2003] = sales['Bounce', 1993] * 2
```

###### 26.1.1.2.5. IS ANY 조건
<br/>
IS ANY 조건은 널을 포함한 모든 값과 일치하는 셀을 참조한다. 규칙의 양측에 사용할 수 있다.  

```
[dimension_column IS] ANY
```

아래는 IS ANY 조건을 사용한 규칙이다. 모든 product의 2002년 sales을 2001년 sales보다 10% 높은 값으로 계산한다.  

```
sales[product IS ANY, 2002] = sales[CV(product), 2001] * 1.1
```

아래와 같이 위치 참조 형식을 사용할 수도 있다.  

```
sales[ANY, 2002] = sales[CV(), 2001] * 1.1
```

위 표현식은 기호 참조로 동작한다. 내부적으로 아래와 같이 해석되기 때문이다.  

```
sales[product IS NULL OR product IS NOT NULL, 2002] = sales[CV(), 2001] * 1.1
```

###### 26.1.1.2.6. 중첩 셀 참조
<br/>
셀 참조는 중첩해서 사용할 수 있다.  

아래 규칙은 sales 셀 참조의 기호 참조에 best&#95;year 셀 참조를 사용했다.  

```
sales['Bounce', 2001] = sales[product = 'Bounce', year = best_year['Bounce', 2001]]
```

아래 규칙은 Standard Mouse Pad의 2001년 sales을 Finding Fido의 sales가 최고인 year와 최저인 year에 해당하는 Standard Mouse Pad의 sales 평균 값으로 계산한다.  

```
sales['Standard Mouse Pad', 2001] = (sales[CV(), best_year['Finding Fido', CV(year)]] + sales[CV(), worst_year['Finding Fido', CV(year)]]) / 2
```

아래 셀 참조는 에러가 발생한다. 중첩 셀 참조는 단일 셀을 참조해야 하며 셀 참조 내부에 집계 함수를 사용할 수 없다.  

```
sales['Bounce', MAX(best_year)['Bounce', ANY]]
```

아래 셀 참조도 에러가 발생한다. 중첩 셀 참조는 1단계만 중첩할 수 있다.  

```
sales['Bounce', best_year['Bounce', best_year['Bounce', 2001]]]
```

#### 26.1.2. 규칙 옵션
<br/>
규칙 옵션은 동작 옵션, 순서 옵션, MODEL ITERATE 절로 구성된다.  

```
[RULES [{UPDATE | UPSERT [ALL]}] -- 동작 옵션
       [{AUTOMATIC | SEQUENTIAL} ORDER] -- 순서 옵션
       [model_iterate_clause]] -- MODEL ITERATE 절
```

##### 26.1.2.1. 동작 옵션
<br/>
동작 옵션은 규칙에서 셀이 동작하는 방식을 결정한다. UPSERT는 위치 참조에 사용해야 한다.  

```
[{UPDATE | UPSERT [ALL]}]
```

+ UPDATE : 좌측에서 참조하는 셀이 존재하면 갱신, 존재하지 않으면 무시
+ UPSERT : 좌측에서 참조하는 셀이 존재하면 갱신, 존재하지 않으면 생성 (기본값)
+ UPSERT ALL : UPSERT의 동작과 유사하나 비교, IN, ANY 등 추가적인 구문을 지원  

아래 규칙은 Bounce의 2001년 sales가 존재하므로 기존 셀 값이 갱신된다.  

```sql
RULE UPDATE (sales['Bounce', 2001] = sales['Bounce', 1999] + sales['Bounce', 2000])
```

아래 규칙은 UPDATE 옵션을 사용했다. UPDATE 옵션을 사용했기 때문에 Bounce의 2002년 sales가 존재하지 않는 경우 무시된다.  

```sql
RULE UPDATE (sales[prod = 'Bounce', year = 2002] = sales['Bounce', 2000] + sales['Bounce', 2001])
```

아래 규칙은 UPSERT 옵션을 사용했기 때문에 신규 셀이 생성된다.  

```sql
RULE UPSERT (sales['Bounce', year = 2002] = sales['Bounce', 2001] * 1.1)
```

UPSERT ALL은 비교, IN, ANY 등 추가적인 구문을 지원한다. 아래의 순서로 동작한다.  

(1) 기호 참조를 만족하는 셀 찾기  
(2) 기호 참조를 만족하는 셀의 고유한 차원 값 찾기  
(3) 위치 참조를 사용한 차원 값으로 교차 곱을 생성
(4) 신규 셀을 UPSERT  

아래의 데이터로 동작 원리를 살펴보자.  

<table>
  <thead>
    <tr>
      <td colspan="3">DIMENSION</td>
      <td>MEASURE</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>prod</td>
      <td>time</td>
      <td>city</td>
      <td>sales</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2002</td>
      <td>x</td>
      <td>10</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2003</td>
      <td>x</td>
      <td>15</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2002</td>
      <td>y</td>
      <td>21</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2003</td>
      <td>y</td>
      <td>24</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래는 UPSERT ALL 옵션을 사용한 규칙이다.  

```sql
RULE UPSERT ALL (sales[ANY, ANY, 'z'] = sales[CV(product), CV(time), 'y'])
```

위 규칙은 아래의 순서로 동작한다.  

(1) 기호 참조인 IS ANY 조건을 사용한 prod 열과 time 열을 식별  
(2) prod 열의 고유 값은 1, 2, time 열의 2002, 2003  
(3) (1, 2002, 'z'), (1, 2003, 'z'), (2, 2002, 'z'), (2, 2003, 'z') 교차 곱 생성  
(4) UPSERT 수행  

최종적으로 아래와 같이 해석된다.  

```
sales[1, 2002, 'z'] = sales[CV(product), CV(time), 'y']
, sales[1, 2003, 'z'] = sales[CV(product), CV(time), 'y']
, sales[2, 2002, 'z'] = sales[CV(product), CV(time), 'y']
, sales[2, 2003, 'z'] = sales[CV(product), CV(time), 'y']
```

결과는 아래와 같다. 음영으로 표시된 부분이 생성된 신규 셀이다.  

<table>
  <thead>
    <tr>
      <td colspan = "3">DIMENSION</td>
      <td>MEASURE</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>prod</td>
      <td>time</td>
      <td>city</td>
      <td>sales</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2002</td>
      <td>x</td>
      <td>10</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2003</td>
      <td>x</td>
      <td>15</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2002</td>
      <td>y</td>
      <td>21</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2003</td>
      <td>y</td>
      <td>24</td>
    </tr>
    <tr>
      <td>1</td>
      <td>2002</td>
      <td>z</td>
      <td></td>
    </tr>
    <tr>
      <td>1</td>
      <td>2003</td>
      <td>z</td>
      <td></td>
    </tr>
    <tr>
      <td>2</td>
      <td>2002</td>
      <td>z</td>
      <td>21</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2003</td>
      <td>z</td>
      <td>24</td>
    </tr>
  </tbody>
</table>
<br/><br/>

규칙의 앞쪽에 동작 옵션을 지정하면 동작 옵션이 지역 범위로 적용된다. 지역 범위로 동작 옵션을 지정하면 전역 범위의 동작 옵션이 무시된다.  

```
[{UPDATE | UPSERT [ALL]}] cell_assignment = expr
```

아래 RULE 절은 첫 번째, 두 번째 규칙의 동작 옵션을 지역 범위로 지정했다.  

```sql
RULE UPDATE (
    UPDATE sales['Bounce', 2001] = sales['Bounce', 2000] * 1.1
  , UPSERT sales['Y Box', 2001] = sales['Y Box', 2000] * 1.1
  , sales['Mouse Pad', 2001] = sales['Mouse Pad', 2000] * 1.1)
```

##### 26.1.2.2. 순서 옵션
<br/>
순서 옵션은 규칙의 평가 순서를 결정한다.  

```
[{AUTOMATIC | SEQUENTIAL} ORDER]
```

+ SEQUENTIAL ORDER : RULES 절에 기술된 순서로 평가 (기본값)
+ AUTOMATIC ORDER : 규칙의 종속성에 따라 순서를 평가  

아래는 SEQUENTIAL ORDER 옵션을 사용한 규칙이다. SEQUENTIAL ORDER 옵션을 사용한 모델을 순차 정렬 모델이라고 한다.  

```sql
RULES SEQUENTIAL ORDER (
    sales['Bounce', 2001] = sales['Bounce', 2000] + sales['Bounce', 1999] -- (1)
  , sales['Bounce', 2000] = 50000 -- (2)
  , sales['Bounce', 1999] = 40000 -- (3)
)
```

아래는 AUTOMATIC ORDER 옵션을 사용한 규칙이다. AUTOMATIC ORDER 옵션을 사용한 모델을 자동 정렬 모델이라고 한다. 두 번째, 세 번째 규칙은 계산 순서가 변경될 수 있따.  

```sql
RULES AUTOMATIC ORDER (
    sales['Bounce', 2001] = sales['Bounce', 2000] + sales['Bounce', 1999] -- (3)
  , sales['Bounce', 2000] = 50000 -- (1) or (2)
  , sales['Bounce', 1999] = 40000 -- (2) or (1)
)
```

아래 규칙은 "ORA-32630: 자동 순서 MODEL에 여러 지정이 있음" 에러가 발생한다. sales['Bounce', 2001]에 값이 여러 번 할당되어 결과가 달라질 수 있기 때문이다. 자동 정렬 모델은 measure 값이 1번만 할당되어야 한다.  

```sql
RULES AUTOMATIC ORDER (
    sales['Bounce', 2001] = sales['Bounce', 2000] + sales['Bounce', 1999] -- (?)
  , sales['Bounce', 2001] = 50000 -- (?)
  , sales['Bounce', 2001] = 40000 -- (?)
)
```

#### 26.1.3. 셀 참조 옵션
<br/>
셀 참조 옵션으로 널 처리와 유일성 확인에 대한 옵션을 설정할 수 있다.  

```
[{IGNORE | KEEP} NAV] -- NAV 처리
[UNIQUE {DIMENSION | SINGLE REFERENCE}] -- 유일성 확인
```

##### 26.1.3.1. NAV 처리
<br/>
NAV 처리 옵션은 값이 널이거나 누락된 셀의 처리 방식을 결정한다. NAV는 Not Available Value의 약자다.  

```
[{IGNORE | KEEP} NAV]
```

+ IGNORE NAV : 기본값으로 처리 (숫자: 0, 문자: 빈 문자열, 날짜: 2001-01-01, 기타: 널)
+ KEEP NAV : 널로 처리 (기본값)  

아래는 IGNORE NAV 옵션을 사용한 쿼리다. Poland에 Bounce 상품이 없지만 sales는 0이 반환된다.  

```sql
SELECT product, year, sales
FROM sales_view
WHERE country = 'Poland'
AND product = 'Bounce'
MODEL IGNORE NAV
      DEMENSION BY (product, year)
      MEASURES (sales sales)
      RULES (sales['Bounce', 2003] = sales['Bounce', 2002] + sales['Bounce', 2001]);
```

##### 26.1.3.2. 유일성 확인
<br/>
유일성 확인 옵션은 모델의 유일성 확인 방식을 결정한다. UNIQUE DIMENSION이 기본값이다. UNIQUE DIMENSION 옵션을 사용하면 고유성 검사로 인해 쿼리의 성능이 저하될 수 있다.  

```
[UNIQUE {DIMENSION | SINGLE REFERENCE}]
```

+ UNIQUE DIMENSION : PARTITION BY 절과 DIMENSION BY 절의 조합으로 고유성을 확인
+ UNIQUE SINGLE REFERENCE : 규칙의 우측 단일 셀 참조의 고유성을 확인  

아래 데이터로 예제를 수행해보자. country, product 열의 조합이 고유하지 않다.  

```sql
SELECT country, product, sales
FROM sales_view
WHERE country = 'France'
AND product = 'Bounce';
```

아래는 UNIQUE DIMENSION 옵션을 사용한 쿼리다. PARTITION BY 절에 country, DIMENSION BY 절에 product를 지정했기 때문에 차원의 주소가 고유하지 않아 에러가 발생한다.  

```sql
SELECT country, product, sales
FROM sales_view
WHERE country = 'France'
AND product = 'Bounce'
MODEL UNIQUE DIMENSION
      PARTITION BY (country)
      DIMENSION BY (product)
      MEASURES (sales)
      RULES (sales['Bounce'] = MAX (sales)['Bounce'] * 1.24);
```

아래는 UNIQUE SINGLE REFERENCE 옵션을 사용한 쿼리다. 입력되는 데이터가 고유하지 않지만 규칙의 우측에 집계 함수를 사용했기 때문에 에러가 발생하지 않는다.  

```sql
SELECT country, product, sales
FROM sales_view
WHERE country = 'France'
AND product = 'Bounce'
MODEL UNIQUE SINGLE REFERENCE
      PARTITION BY (country)
      DIMENSION BY (product)
      MEASURES (sales)
      RULES (sales['Bounce'] = MAX(sales)['Bounce'] * 1.24);
```

UNIQUE SINGLE REFERENCE 옵션을 사용하더라도 규칙의 우측에 단일 셀 참조를 사용하면 에러가 발생한다.  

```sql
SELECT country, product, sales
FROM sales_view
WHERE country = 'France'
AND product = 'Bounce'
MODEL UNIQUE SINGLE REFERENCE
      PARTITION BY (country)
      DIMENSION BY (product)
      MEASURES (sales)
      RULES (sales['Bounce'] = sales[CV()] * 1.24);

ORA-32638: MODEL 차원의 주소 지정이 고유하지 않음
```

#### 26.1.4. 행 반환 옵션
<br/>
행 반환 옵션은 행 반환 방식을 결정한다.  

```
RETURN {ALL | UPDATED} ROWS
```

+ RETURN ALL ROWS : 전체 행을 반환 (기본값)
+ RETURN UPDATED ROWS : 입력 또는 수정된 행만 반환  

아래 쿼리는 RETURN UPDATED ROWS 옵션을 지정했기 때문에 입력 또는 수정된 행만 반환한다.  

```sql
SELECT country, product, year, sales
FROM sales_view
WHERE country IN ('Italy', 'Japan')
AND product IN ('Bounce', 'Y Box')
AND year IN (2000, 2001)
MODEL RETURN UPDATED ROWS
      PARTITION BY (country)
      DEMENSION BY (product, year)
      MEASURES (sales)
      RULES (sales['Bounce', 2002] = sales['Bounce', 2001] + sales['Bounce', 2000]
           , sales['Y Box', 2002] = sales['Y Box', 2001] + sales['Y Box', 2000])
ORDER BY country, product, year;
```

#### 26.1.5. 참조 모델
<br/>
아래는 참조 모델의 구문이다. 참조 모델은 메인 모델에서 참조할 수 있는 읽기 전용 다차원 배열이다. 참조 모델은 외부 쿼리와 관계를 가질 수 없다.  

```
REFERENCE reference_model_name
  ON (subquery)
  DIMENSION BY (expr [c_alias] [, expr [c_alias]]...) -- DEMENSION BY 절
  MEASURES (expr [c_alias] [, expr [c_alias]]...) -- MEASURES 절
  [cell_reference_options] -- 셀 참조 옵션 (지역)
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE dollar_conv_tbl PURGE;
CREATE TABLE dollar_conv_tbl(country VARCHAR2(30), exchange_rate NUMBER);

INSERT INTO dollar_conv_tbl VALUES ('Poland', 0.25);
INSERT INTO dollar_conv_tbl VALUES ('France', 0.14);
COMMIT;
```

아래는 참조 모델을 사용한 쿼리다. measure 값은 {model&#95;name}.{measure&#95;name} 형식으로 참조할 수 있다. 참조 모델을 사용하는 경우 셀 참조 옵션을 전역이나 지역으로 설정할 수 있다.  

```sql
SELECT country, year, sales, dollar_sales
FROM sales_view
WHERE country IN ('France', 'Poland')
GROUP BY country, year
MODEL KEEP NAV -- 셀 참조 옵션 (전역)
REFERENCE conv_ref
ON (SELECT country, exchange_rate FROM dollar_conv_tbl)
DIMENSION BY (country)
MEASURES (exchange_rate)
IGNORE NAV -- 셀 참조 옵션 (지역)
MAIN conversion
  DIMENSION BY (country, year)
  MEASURES (SUN(sales) AS sales, SUM(sales) AS dollar_sales)
  RULES (
      dollar_sales['France', 2001] = sales[CV(country), 2000] * 1.02 * conv_ref.exchange_rate['France']
    , dollar_sales['Poland', 2001] = sales('Poland', 2000) * 1.05 * exchange_rate['Poland']);
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE growth_rate_tbl PURGE;
CREATE TABLE growth_rate_tbl (country VARCHAR2(30), year NUMBER, growth_rate NUMBER);

INSERT INTO growth_rate_tbl VALUES ('Poland', 2002, 2.5);
INSERT INTO growth_rate_tbl VALUES ('Poland', 2003, 5);
INSERT INTO growth_rate_tbl VALUES ('France', 2002, 3);
INSERT INTO growth_rate_tbl VALUES ('France', 2003, 2.5);
COMMIT;
```

아래 쿼리는 다수의 참조 모델을 사용한다.  

```sql
SELECT country, year, sales, dollar_sales
FROM sales_view
WHERE country = 'France'
GROUP BY country, year
MODEL RETURN UPDATED ROWS -- 셀 참조 옵션 (전역)
REFERENCE conv_ref
ON (SELECT country, exchange_rate FROM dollar_conv_tbl)
DIMENSION BY (country AS c)
MEASURES (exchange_rate)
REFERENCE growth_ref
ON (SELECT country, year, growth_rate FROM growth_rate_tbl)
DIMENSION BY (country AS c, year AS y)
MEASURES (growth_rate)
MAIN projection
  DIMENSION BY (country, year)
  MEASURES (SUN(sales) AS sales, 0 AS dollar_sales)
  RULES (
      dollar_sales[ANY, 2001] = sales[CV(country), 2000] * growth_rate[CV(country), CV(year)] * exchange_rate[CV(country)]);
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE year_2_seq PURGE;

CREATE TABLE year_2_seq (i, year) AS
SELECT ROW_NUMBER() OVER(ORDER BY calendar_year) AS i, calendar_year AS year
FROM (SELECT DISTINCT calendar_year FROM sh.times);
```

year&#95;2&#95;seq 테이블을 조회한 결과는 아래와 같다.  

```sql
SELECT * FROM year_2_seq;
```

아래 쿼리는 y2i 참조 모델과 i2y 참조 모델의 measure 값을 중첩되어 참조한다.  

```sql
SELECT country, product, year, sales, prior_period
FROM sales_view
WHERE country = 'Frnace'
AND product = 'Bounce'
MODEL
REFERENCE y2i ON (SELECT year, i FROM year_2_seq)
DIMENSION BY (year AS y)
MEASURES (i)
REFERENCE i2y ON (SELECT year, i FROM year_2_seq)
DIMENSION BY (i)
MEASURES (year AS y)
MAIN projection2
PARTITION BY (country)
DIMENSION BY (product, year)
MEASURES (sales, CAST(NULL AS NUMBER) AS prior_period)
RULES (prior_period[ANY, ANY]) = sales[CV(product), i2y.y[y2i.i[CV(year)] - 1]])
ORDER BY country, product, year;
```

### 26.2. 고급 주제
<br/>
#### 26.2.1. 널과 누락된 셀 처리
<br/>
셀에서 널을 참조해야 할 경우 IS ANY 조건과 IS NULL 조건을 사용할 수 있다.  

<table>
  <thead>
    <tr>
      <td>조건</td>
      <td>위치 참조</td>
      <td>기호 참조</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>IS ANY</td>
      <td>sales[ANY]</td>
      <td>sales[product IS ANY]</td>
    </tr>
    <tr>
      <td>IS NULL</td>
      <td>sales[NULL]</td>
      <td>sales[product IS NULL]</td>
    </tr>
  </tbody>
</table>
<br/><br/>

규칙에서 널과 누락된 셀을 처리하기 위해 IS PRESENT 조건, PRESENTV 함수, PRESENTVVN 함수를 사용할 수 있다.  

##### 26.2.1.1. IS PRESENT 조건
<br/>
IS PRESENT 조건은 cell&#95;reference가 존재하면 TRUE, 존재하지 않으면 FALSE를 반환한다.  

```
cell_reference IS PRESENT
```

##### 26.2.1.2. PRESENTV 함수
<br/>
PRESENTV 함수는 cell&#95;reference가 존재하면 expr1, 존재하지 않으면 expr2를 반환한다. 규칙의 우측에 사용할 수 있다.  

```
PRESENTV (cell_reference, expr1, expr2)
```

아래는 PRESENTV 함수를 사용한 규칙이다.  

```sql
PRESENTV (sales['Bounce', 2000], sales['Bounce', 2000] * 1.1, 100)
```

##### 26.2.1.3. PRESENTNNV 함수
<br/>
PRESENTNNV 함수는 cell&#95;reference가 존재하거나 널이 아니면 expr1, 존재하지 않거나 널이면 expr2를 반환한다. 규칙의 우측에 사용할 수 있다.  

```
PRESENTNNV (cell_reference, expr1, expr2)
```

아래는 PRESENTNNV 함수를 사용한 규칙이다.  

```sql
PRESENTNNV (saels['Bounce', 2000], sales['Bounce', 2000] * 1.1, 100)
```

아래는 IS PRESENT 조건을 사용한 규칙이다.  

```sql
CASE
  WHEN sales['Bounce', 2000] IS PRESENT AND sales['Bounce', 2000] IS NOT NULL
  THEN 1.1 * sales['Bounce', 2000]
  ELSE 100
END
```

#### 26.2.2. FOR 루프
<br/>
FOR 루프를 사용하면 규칙을 간결하게 표현할 수 있다. FOR 루프는 RULES 절의 양측에 사용할 수 있으며, 좌측에 사용할 경우 위치 참조로 처리된다.  

##### 26.2.2.1. FOR LOOP 절
<br/>
###### 26.2.2.1.1. 단일 열 IN 조건
<br/>
단일 열 IN 조건의 FOR 루프 구문은 아래와 같다.  

```
FOR dimension IN (value [, value]...)
```

아래는 단일 열 IN 조건의 FOR 루프 구문을 사용한 규칙이다. IN 조건에 기술한 product의 2002년 sales을 2001년 sales보다 19% 높은 값으로 계산한다.    

```sql
sales[FOR product IN ('Bounce', ..., 'Y Box'), 2002] = sales[CV(product), 2001] * 1.1
```

위 규칙은 아래 규칙으로 해석된다. FOR 루프 구문을 사용하지 않으면 product 별로 규칙을 반복 기술해야 한다.  

```sql
  sales['Bounce', 2002] = 1.1 * sales['Bounce', 2001]
, ...
, sales['Y Box', 2002] = 1.1 * sales['Y Box', 2001])
```

아래와 같이 FOR 루프 구문을 중첩할 수도 있다.  

```sql
sales[FOR product IN ('Bounce', ..., 'Y Box'), FOR year IN (2002, 2003)] = sales[CV(product), CV(year) - 1] * 1.1
```

위 규칙은 아래 규칙으로 해석된다.  

```sql
  sales['Bounce', 2002] = sales[CV(product), CV(year) - 1] * 1.1
, sales['Bounce', 2003] = sales[CV(product), CV(year) - 1] * 1.1
, ...
, sales['Y Box', 2002] = sales[CV(product), CV(year) - 1] * 1.1
, sales['Y Box', 2003] = sales[CV(product), CV(year) - 1] * 1.1
```

###### 26.2.2.1.2. 다중 열 IN 조건
<br/>
다중 열 IN 조건은 FOR 루프 구문은 아래와 같다.  

```
FOR (dimension_column [, dimension_column]...)
IN ((literal [, literal]...) [(literal [, literal]...)]...)
```

아래는 다중 열 IN 조건의 FOR 루프 구문으로 사용한 규칙이다.  

```sql
sales[FOR(product, year) IN (('Bounce', 2002), ..., ('Y Box', 2003))] = sales[CV(product), CV(year) - 1] * 1.1
```

위 규칙은 아래 규칙으로 해석된다.  

```sql
  sales['Bounce', 2002] = sales[CV(product), CV(year) * 1.1]
, ...
, sales['Y Box', 2003] = sales[CV(product), CV(year) - 1] * 1.1
```

###### 26.2.2.1.3. 단일 열 서브 쿼리
<br/>
단일 열 서브 쿼리의 FOR 루프 구문은 아래와 같다. subquery는 상관 서브 쿼리를 사용할 수 없으며 결과가 10,000행 미만이어야 한다. WITH 절은 사용할 수 없다.  

```
FOR dimension IN (subquery)
```

아래는 단일 열 서브 쿼리의 FOR 루프 구문을 사용한 규칙이다.  

```sql
sales[FOR product IN (SELECT name FROM sh.products) FOR year IN (2002, 2003)] = sales[CV(product), CV(year) - 1] * 1.1
```

FOR 루프가 아닌 규칙에 서브 쿼리를 사용하면 에러가 발생한다.  

```sql
SELECT *
FROM sales_view
WHERE country = 'Poland'
MODEL DIMENSION BY (product, year)
      MEASURES (sales)
      RULES UPSERT (sales['Bounce', 2003] = sales['Bounce', 2002] + (SELECT SUM(sales) FROM sales_view));

ORA-32620: MODEL 규칙 내에 잘못된 하위 질의가 있음
```

위 쿼리는 아래 쿼리로 변경할 수 있다. MEASURES 절에는 서브 쿼리를 사용할 수 있다.

###### 26.2.2.1.4. 다중 열 서브 쿼리
<br/>
다중 열 서브 쿼리의 FOR 루프 구문은 아래와같다.  

```
FOR (dimension_column [, dimension_column]...) IN (subquery)
```

아래는 다중 열 서브 쿼리의 FOR 루프 구문을 사용한 쿼리다.  

```sql
SELECT country, product, year, s
FROM sales_view
MODEL RETURN UPDATED ROWS
      DIMENSION BY (country, product, year)
      MEASURES (sales AS s)
      RULES UPSERT (s[FOR (country, product, year) IN (SELECT DISTINCT 'new_country', product, year
                                                       FROM sales_view
                                                       WHERE product = 'Bounce')]
                                                   = s['France', CV(), CV()])
ORDER BY country, year, product;
```

###### 26.2.2.1.5. FROM TO 방식
<br/>
FROM TO 방식은 FOR 루프 구문은 아래와 같다.  

```
FOR dimension_column FROM literal TO literal {INCREMENT | DECREMENT} literal
```

아래는 FROM TO 방식의 FOR 루프 구문을 사용한 규칙이다.  

```sql
sales['Bounce', FOR year FROM 2002 TO 2004 INCREMENT 1] = sales['Bounce', year = CV(year) - 1] * 1.2
```

위 규칙은 아래 규칙으로 해석된다.  

```sql
  sales['Bounce', 2002] = sales['Bounce', 2001] * 1.2
, sales['Bounce', 2003] = sales['Bounce', 2002] * 1.2
, sales['Bounce', 2004] = sales['Bounce', 2003] * 1.2
```

FOR 루프는 음수 증분 값을 사용할 수 없다. DECREMENT 키워드를 사용하면 앞쪽 literal이 뒤쪽 literal보다 커야 한다.  

```sql
FOR d FROM 2001 TO 2005 INCREMENT -1 -- 오류
FOR d FROM 2005 TO 2001 INCREMENT 1 -- 오류
FOR d FROM 2005 TO 2001 DECREMENT 1 -- 정상
```

###### 26.2.2.1.6. LIKE FROM TO 방식
<br/>
LIKE FROM TO 방식의 FOR 루프 구문은 아래와 같다.  

```
FOR dimension LIKE pattern FROM literal TO literal {INCREMENT | DECREMENT} literal
```

아래는 LIKE FROM TO 방식의 FOR 루프 구문을 사용한 규칙이다.  

```sql
sales[FOR product LIKE 'product-%' FROM 1 TO 3 INCREMENT 1, 2002] = sales[CV(product), 2001] * 1.2
```

위 규칙은 아래 규칙으로 해석된다.  

```sql
  saels['product-1', 2002] = sales['product-1', 2001] * 1.2
, saels['product-2', 2002] = sales['product-2', 2001] * 1.2
, saels['product-3', 2002] = sales['product-3', 2001] * 1.2
```

##### 26.2.2.2. FOR 루프의 순서
<br/>
FOR 루프는 순서 옵션에 따라 평가 순서가 달라질 수 있다.  

아래 규칙을 예로 들어보자.  

```sql
sales['Bounce', FOR year FROM 2004 TO 2001 DECREMENT 1] = sales['Bounce', CV(year) - 1] * 1.1
```

순사 정렬 모델에서는 아래와 같이 평가된다.  

```sql
  sales['Bounce', 2004] = sales['Bounce', 2003] * 1.1
, sales['Bounce', 2003] = sales['Bounce', 2002] * 1.1
, sales['Bounce', 2002] = sales['Bounce', 2001] * 1.1
, sales['Bounce', 2001] = sales['Bounce', 2000] * 1.1
```

순차 정렬 모델에서는 아래와 같이 평가된다.  

```sql
  sales['Bounce', 2001] = sales['Bounce', 2000] * 1.1
, sales['Bounce', 2002] = sales['Bounce', 2001] * 1.1
, sales['Bounce', 2003] = sales['Bounce', 2002] * 1.1
, sales['Bounce', 2004] = sales['Bounce', 2003] * 1.1
```

##### 26.2.2.3. FOR 루프의 전개
<br/>
규칙의 좌측에 FOR 루프 구문을 사용하면 FOR 루프를 사용한 규칙이 전개된다. 전개된 규칙의 우측은 단일 셀 참조로 변경된다. FOR 루프의 전개 시점은 동작 옵션과 참조 방식에 따라 달라질 수 있다.  

UPDATE 옵션과 UPSERT 옵션은 참조 방식에 따라 전개 시점이 달라진다. UPDATE ALL 옵션은 무조건 실행 계획 생성 단계에서 전개된다.  

+ 실행 : 규칙의 좌측이 단일 셀 참조로 전개될 수 있는 경우
+ 실행 계획 생성 : 규칙의 좌측이 단일 셀 참조로 전개될 수 없는 경우  

아래 규칙은 FOR 루프가 실행 단계에 전개된다.  

```sql
RULE UPSERT (sales[FOR product IN ('prod1', 'prod2'), 2002] = sales[CV(product), 2001] * 1.2)
```

위 규칙은 아래 규칙으로 전개된다. 규칙의 좌측이 단일 셀 참조다.  

```sql
RULE UPSERT (sales['prod1', 2002] = sales['prod1', 2001] * 1.2
           , sales['prod2', 2002] = sales['prod2', 2001] * 1.2)
```

아래 규칙은 FOR 루프가 실행 계획 생성 단계에 전개된다.  

```sql
FOR UPSERT (sales[FOR product IN ('prod1', 'prod2'), year >= 2002] = sales[CV(product), 2001] * 1.2)
```

위 규칙은 아래 규칙으로 전개된다. 규칙의 좌측이 다중 셀 참조다.  

```sql
RULE UPSERT (sales['prod1', year >= 2002] = sales['prod1', 2001] * 1.2 = sales['prod2', year >= 2002] = sales['prod2', 2001] * 1.2)
```

아래 규칙은 에러가 발생한다. FOR 루프 표현식에 ITERATION&#95;NUMBER 함수를 사용하여 실행 계획 생성 단계에서 값을 알 수 없기 때문이다. FOR 루프가 실행 계획 생성 단계에 전개될 경우 FOR 루프 표현식에 중첩 셀 참조, 참조 모델, ITERATION&#95;NUMBER 함수 등을 사용하면 에러가 발생한다.  

```sql
sales[FOR product LIKE 'prod%' FROM ITERATION_NUMBER TO ITERATION_NUMBER + 1, year >= 2003] = sales[CV(product), 2002] * 1.2
```

아래 규칙은 FOR 루프 표현식에 ITERATION&#95;NUMBER 함수를 사용했지만 FOR 루프 전개에 사용되지 않아 에러가 발생하지 않는다.  

```sql
sales[FOR product IN ('prod' || ITERATION_NUMBER, 'prod' || (ITERATION_NUMBER + 1)), year >= 2003] = sales[CV(product), 2002] * 1.2
```

위 규칙은 아래 규칙으로 전개된다.  

```sql
  sales['prod' || ITERATION_NUMBER, year >= 2003] = sales[CV(product), 2002] * 1.2
, sales['prod' || (ITERATION_NUMBER + 1), year >= 2003] = sales[CV(product), 2002] * 1.2
```

#### 26.2.3. MODEL ITERATE 절
<br/>
MODEL ITERATE 절을 사용하면 number&#95;of&#95;iterations에 도달하거나 condition이 만족할 때까지 규칙을 반복 평가할 수 있다. condition에 PREVIOUS 함수와 ITERATION&#95;NUMBER 함수를 사용할 수 있다.  

```
ITERATE (number_of_iterations) [UNTIL (condition)]
```

아래는 ITERATE 절을 사용한 쿼리다. 1024를 2로 4회 나눈 결과를 반환한다.  

```sql
SELECT x, s
FROM DUAL
MODEL DIMENSION BY (1 AS x)
      MEASURES (1024 AS s)
      RULES UPDATE ITERATE (4) (s[1] = s[1] / 2);
```

##### 26.2.3.1. PREVIOUS 함수
<br/>
PREVIOUS 함수는 반복 시작 시점의 cell&#95;reference의 값을 반환한다.  

```
PREVIOUS (cell_reference)
```

##### 26.2.3.2. ITERATION&#95;NUMBER 함수
<br/>
ITERATION&#95;NUMBER 함수는 0부터 시작하는 반복 횟수를 반환한다.  

아래 쿼리는 1024를 2로 1000회까지 나누되 직전 값과 현재 값의 차이가 1보다 작으면 평가를 중단한다.  

```sql
SELECT x, s, iterations
FROM DUAL
MODEL DIMENSION BY (1 AS x)
      MEASURES (1024 AS s, 0 AS iterations)
      RULES ITERATE (1000) UNTIL ABS (PREVIOUS (s[1]) - s[1]) < 1 (s[1] = s[1] / 2, iterations[1] = ITERATION_NUMBER);
```

아래 쿼리는 2002년부터 2004년까지 sales를 각각 3년 전의 sales로 계산한다. ITERATION&#95;NUMBER 함수를 셀 참조에 사용했다.  

```sql
SELECT country, product, year, sales
FROM sales_view
WHERE country = 'France'
AND product = 'Bounce'
MODEL PARTITION BY (country)
      DIMENSION BY (product, year)
      MEASURES (sales)
      RULES ITERATE (3) (sales['Bounce', 2002 + ITERATION_NUMBER] = sales['Bounce', 1999 + ITERATION_NUMBER])
ORDER BY country, product, year;
```

#### 26.2.4. 규칙 종속
<br/>
자동 정렬 모델은 규칙의 종속 관계에 따라 평가 순서를 결정한다. 평가 순서를 결정하는 과정에서 순환 종속이 발견되면 에러가 발생한다.  

아래 규칙은 순환 종속이 존재하기 때문에 "ORA-32634: 자동 순서 MODEL 평가가 수렴되지 않음" 에러가 발생한다.  

```sql
  sales['Bounce', 2001] = sales['Y Box', 2001] * 1.5
, sales['Y Box', 2001] = 100000 / sales['Bounce', 2001]
```

아래 규칙은 자기 순환 종속이 존재한다. 위 규칙과 동일한 에러가 발생한다.  

```sql
sales['Bounce', 2002] = 25000 / sales['Bounce', 2002]
```

아래의 쿼리는 순환 종속이 존재하지 않는다. 주석에 표시한 순서로 규칙이 평가된다.  

```sql
SELECT country, product, year, sales
FROM sales_view
WHERE country = 'Frnace'
MODEL RETURN UPDATED ROWS
      PARTITION BY (country)
      DIMENSION BY (product, year)
      MEASURES (sales)
      RULES AUTOMATIC ORDER (sales['SUV', 2001] = 10000 -- (1) or (2)
                           , sales['Standard Mouse', 2001] = sales['Finding Fido', 2001] * 0.10 + sales['Boat', 2001] * 0.50 -- (4)
                           , sales['Boat', 2001] = sales['Finding Fido', 2001] * 0.25 + sales['SUV', 2001] * 0.75 -- (3)
                           , sales['Finding Fido', 2001] = 20000) -- (2) or (1)
ORDER BY country, product, year;
```

#### 26.2.5. 정렬 규칙
<br/>
규칙의 좌측이 다중 셀 참조인 경우 규칙의 평가 순서에 따라 결과가 달라질 수 있다. 규칙의 좌측에 ORDER BY 절을 기술하면 정렬 규칙(ordered rule)을 지정할 수 있다.  

```
[{UPDATE | UPSERT [ALL]}] cell_assignment [order_by_clause] = expr
```

아래 데이터로 예제를 수행해보자.  

```sql
SELECT calendar_year AS t, SUM(amount_sold) AS s
FROM sh.sales, sh.times
WHERE sales.time_id = times.time_id
GROUP BY calendar_year;
```

아래 쿼리는 IS ANY 조건을 사용했다. 규칙의 평가 순서에 따라 결과가 달라질 수 있기 때문에 에러가 발생한다.  

```sql
SLEECT t, s
FROM sh.sales, sh.times
WHERE sales.time_id = times.time_id
GROUP BY calendar_year
MODEL DIMENSION BY (calendar_year AS t)
      MEASURES (SUM(amount_sold) AS s)
      RULES SEQUENTIAL ORDER (s[ANY] = s[CV(t) - 1])
ORDER BY t;
```

아래 쿼리는 에러가 발생하지 않는다. 규칙의 좌측에 ORDER BY 절을 기술하여 규칙의 평가 순서를 지정했다.  

```sql
SELECT t, s
FROM sh.sales, sh.times
WHERE sales.time_id = times.time_id
GROUP BY calendar_year
MODEL DIMENSION BY (calendar_year AS t)
      MEASURES (SUM(amount_sold) AS s)
      RULES SEQUENTIAL ORDER (s[ANY] ORDER BY t DESC = s[CV(t) - 1])
ORDER BY t;
```

위 쿼리의 RULS 절은 아래의 순서로 평가된다.  

```
RULES (s[2001] = s[2000]
     , s[2000] = s[1999]
     , s[1999] = s[1998]
     , s[1998] = s[1997])
```

아래 쿼리는 자동 정렬 모델을 사용했다. 자동 정렬 모델은 종속성에 따라 규칙을 평가하기 때문에 s 열의 값이 반환되지 않는다.  

```sql
SELECT t, s
FROM sh.sales, sh.times
WHERE sales.time_id = times.time_id
GROUP BY calendar_year
MODEL DIMENSION BY (calendar_year AS t)
      MEASURES (SUM(amount_sold) AS s)
      RULES AUTOMATIC ORDER (s[ANY] = s[CV(t) - 1])
ORDER BY t;
```

위 쿼리의 RULS 절은 아래의 순서로 평가된다.  

```
RULES (s[1998] = s[1997]
     , s[1999] = s[1998]
     , s[2000] = s[1999]
     , s[2001] = s[2000])
```

자동 정렬 모델에도 ORDER BY 절을 기술할 수 있다. 아래 쿼리는 의도한 결과가 반환된다.  

```sql
SELECT t, s
FROM sh.sales, sh.times
WHERE sales.time_id = times.time_id
GROUP BY calendar_year
MODEL DIMENSION BY (calendar_year AS t)
      MEASURES (SUM(amount_sold) AS s)
      RULES AUTOMATIC ORDER (s[ANY] ORDER BY t DESC = s[CV(t) - 1])
ORDER BY t;
```

#### 26.2.6. 분석 함수
<br/>
규칙의 우측에 분석 함수를 사용할 수 있다. 분석 함수를 사용하면 규칙의 좌측에 FOR 루프와 ORDER BY 절을 사용할 수 없다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE sales_rollup_time PURGE;

CREATE TABLE sales_rollup_time AS
SELECT d.country_name AS country
     , c.calendar_year AS year, c.calendar_quarter_desc AS quarter
     , GROUPING_ID (c.calendar_year, c.calendar_quarter_desc) AS gid
     , SUM(a.amount_sold) AS sale, COUNT(a.amount_sold) AS cnt
FROM sh.sales a, sh.customers b, sh.times c, sh.countries d
WHERE b.cust_id = a.cust_id
AND c.time_id = a.time_id
AND d.country_id = b.country_id
GROUP BY d.country_name, c.calendar_year, ROLLUP(c.calendar_quarter_desc)
ORDER BY gid, country, year, quarter;
```

sales&#95;rollup&#95;time 테이블의 구조는 아래와 같다.  

```sql
DESC sales_rollup_time
```

아래는 규칙의 우측에 분석 함수를 사용한 쿼리다. country, year 별로 sales의 누적 합계 값을 계산한다.  

```sql
SELECT country, year, quarter, sale, cusm
FROM sales_rollup_time
WHERE country = 'United Kingdom'
MODEL DIMENSION BY(country, year, quarter)
      MEASURES (sale, gid, 0 As csum)
      RULES (csum[ANY, ANY, ANY] = SUM(sale) OVER (PARTITION BY country, DECODE(gid, 0, year, NULL) ORDER BY year, quarter ROWS UNBOUNDED PRECEDING))
ORDER BY country, gid, year, quarter;
```

MEASURES 절에도 분석 함수를 사용할 수 있다. 분석 함수가 외부 쿼리에서 수행된다.  

```sql
WITH w1 AS (
  SELECT country, product, year, s, rnk
  FROM sales_view
  WHERE country = 'France'
  AND product = 'Bounce'
  MODEL PARTITION BY (country)
        DIMENSION BY (product, year)
        MEASURES (sales AS s, year AS y, RANK() OVER(ORDER BY sales) AS rnk)
        RULES UPSERT (
            s['Bounce Increase 90-99', 2001] = REGR_SLOPE(s, y)['Bounce', year BETWEEN 1990 AND 2000]
          , s['Bounce', 2001] = s['Bounce', 2000] * (1 + s['Bounce Increase 90-99', 2001])))
SELECT country, product, year, s, rnk
FROM w1
WHERE product <> 'Bounce Increase 90-99'
ORDER BY country, year, rnk, product;
```

### 26.3. 활용 예제
<br/>
MODEL 절의 활용 예제를 살펴보자.  

아래 쿼리는 Italy와 Spain의 product별 sales의 차이를 계산한다.  

```sql
SELECT product, country, sales
FROM sales_view
WHERE country IN ('Italy', 'Spain')
AND product IN ('Bounce', 'Finding Fido')
GROUP BY product, country
MODEL PARTITION BY (product)
      DIMENSION BY (country)
      MEASURES (SUM(sales) AS sales)
      RULES UPSERT (sales['DIFF ITALY-SPAIN'] = sales['Italy'] - sales['Spain']);
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE sales_view2 PURGE;

CREATE TABLE sales_view2 AS
SELECT e.country_name AS country, b.prod_name AS product
     , d.calendar_year AS year, d.calendar_month_name AS month
     , SUM(a.amount_sold) AS sale, COUNT(a.amount_sold) AS cnt
FROM sh.sales a, sh.products b, sh.customers c, sh.times d, sh.countries e
WHERE b.prod_id = a.prod_id
AND c.cust_id = a.cust_id
AND d.time_id = a.time_id
AND e.country_id = c.country_id
GROUP BY e.country_name, b.prod_name, d.calendar_year, d.calendar_month_name;
```

sales&#95;view2 테이블의 구조는 아래와 같다.  

```sql
DESC sales_view2
```

아래 쿼리는 10월, 11월의 sales로 12월의 country별 예상 sales을 계산한다.  

```sql
SELECT country, SUM(sales) AS sales
FROM (SELECT product, country, month, sales
      FROM sales_view2
      WHERE year = 2000
      AND month IN ('October', 'November')
      MODEL PARTITION BY (product, country)
            DIMENSION BY (month)
            MEASURES (sales AS sales)
            RULES (sales['December'] = (sales['Nomvmber'] / sales['October']) * sales['November']))
GROUP BY GROUPING SETS ((), (country));
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE cash_flow PURGE;
CREATE TABLE cash_flow (year DATE, i NUMBER, prod VARCHAR2(3), amount NUMBER);

INSERT INTO cash_flow VALUES (TO_DATE('1999', 'YYYY'), 0, 'vcr', -100.00);
INSERT INTO cash_flow VALUES (TO_DATE('2000', 'YYYY'), 1, 'vcr', 12.00);
INSERT INTO cash_flow VALUES (TO_DATE('2001', 'YYYY'), 2, 'vcr', 10.00);
INSERT INTO cash_flow VALUES (TO_DATE('2002', 'YYYY'), 3, 'vcr', 20.00);
INSERT INTO cash_flow VALUES (TO_DATE('1999', 'YYYY'), 0, 'dvd', -200.00);
INSERT INTO cash_flow VALUES (TO_DATE('2000', 'YYYY'), 1, 'dvd', 22.00);
INSERT INTO cash_flow VALUES (TO_DATE('2001', 'YYYY'), 2, 'dvd', 12.00);
INSERT INTO cash_flow VALUES (TO_DATE('2002', 'YYYY'), 3, 'dvd', 14.00);
COMMIT;
```

아래 쿼리는 순현재가치(NPV: Net Present Value)를 계산한다. 할인율을 0.14로 가정한다.  

```sql
SELECT yearm i, prod, amount, npv
FROM cash_flow
MODEL PARTITION BY (prod)
      DIMENSION BY (i)
      MEASURES (amount, 0 AS npv, year)
      RULES (
          npv[0] = amount[0]
        , npv[i != 0] ORDER BY i = amount[CV()] / POWER(1.14, CV(i)) + npv[CV(i) - 1]);
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE ledger PURGE;
CREATE TABLE ledger (account VARCHAR2(20), balance NUMBER(10.2));

INSERT INTO ledger VALUES ('Salary', 100000);
INSERT INTO ledger VALUES ('Capital_gains', 15000);
INSERT INTO ledger VALUES ('Net', 0);
INSERT INTO ledger VALUES ('Tax', 0);
INSERT INTO ledger VALUES ('Interest', 0);
COMMIT;
```

아래 쿼리는 연립 방정식을 계산한다. MODEL ITERATE 절을 사용하여 규칙을 100회 반복 평가한다. 순수익(Net)은 급여(Salary)에서 이자(Interest)와 세금(Tax)을 차감한다. 세금은 급여에서 이자를 공제한 금액의 38%에 자본소득(Capital_gains)의 28%를 더한다. 이자는 순이익의 30%다.  

```sql
SELECT account, s
FROM ledger
MODEL DIMENSION BY (account)
      MEASURES (balance s)
      RULES ITERATE (100) (
          s['Net'] = s['Salary'] - s['Interest'] - s['Tax']
        , s['Tax'] = (s['Salary'] - s['Interest']) * 0.38 + s['Capital_gains'] * 0.28
        , s['Interest'] = s['Net'] * 0.30);
```

아래 쿼리는 2001년의 sales에 대한 회귀분석을 수행한다. REGR&#95;SLOPE 함수를 사용했다.  

```sql
SELECT *
FROM (SELECT country, product, year, projected_sale, sales
      FROM sales_view
      WHERE country IN ('Italy', 'Japan')
      AND product IN ('Bounce')
      MODEL PARTITION BY (country)
            DIMENSION BY (product, year)
            MEASURES (sales sales, year y, CAST(NULL AS NUMBER) pro-jected_sale)
            RULES (projected_sale[FOR product IN ('Bounce'), 2001] = sales[CV(), 2000] + REGR_SLOPE(sales, y)[CV(), year BETWEEN 1998 AND 2000]))
ORDER BY country, product, year;
```
