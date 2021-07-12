---
title:  MATCH&#95;RECOGNIZE 절
categories:
- Unkind_SQL
feature_text: |
  ## 27. MATCH&#95;RECOGNIZE 절
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

MATCH&#95;RECOGNIZE 절은 행에 대한 패턴 일치(pattern matching) 기능을 수행한다. 행의 패턴을 인식하는 기능은 가격, 거래량, 기타 행동 패턴을 인식해야 하는 금융 프로그램과 비정상적 동작을 감지해야 하는 보안 프로그램 등에 활용할 수 있다. MATCH&#95;RECOGNIZE 절은 12.1 버전부터 사용할 수 있다.  

### 27.1. 기본 문법
<br/>
아래는 MATCH&#95;RECOGNIZE 절의 구문이다. MATCH&#95;RECOGNIZE 절은 FROM 절과 ORDER BY 절 사이에 기술한다.  

```
MATCH_RECOGNIZE (
  [row_pattern_partition_by] -- (1) PARTITION BY 절
  [row_pattern_order_by] -- (2) ORDER BY 절
  [row_pattern_measures] -- (8) MEASURES 절
  [row_pattern_rows_per_match] -- (7) ROWS PER MATCH 절
  [row_pattern_skip_to] -- (6) SKIP TO 절 -> (5)
  PATTERN (row_pattern) -- (5) PATTERN 절
  [row_pattern_subset_clause] -- (4) SUBSET 절
  DEFINE row_pattern_definition_list -- (3) DEFINE 절
)
```

+ PARTITION BY 절 : 파티션을 지정
+ ORDER BY 절 : 정렬 순서를 지정
+ MEASURES 절 : 반환할 표현식을 정의
+ ROWS PER MATCH 절 : 일치한 패턴의 반환 방식을 지정
+ SKIP TO 절 : 패턴 일치를 다시 시작할 위치를 지정
+ PATTERN 절 : 일치시킬 패턴을 지정
+ SUBSET 절 : 결합 패턴 변수를 정의
+ DEFINE 절 : 패턴 변수의 조건을 정의  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE ticker PURGE;
CREATE TABLE ticker (symbol VARCHAR2 (10), tstamp DATE, price NUMBER);

INSERT INTO ticker VALUES ('ACME', DATE '2011-04-01', 12);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-02', 17);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-03', 19);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-04', 21);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-05', 25);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-06', 12);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-07', 15);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-08', 20);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-09', 24);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-10', 25);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-11', 19);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-12', 15);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-13', 25);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-14', 25);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-15', 14);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-16', 12);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-17', 14);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-18', 24);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-19', 23);
INSERT INTO ticker VALUES ('ACME', DATE '2011-04-20', 22);
COMMIT;
```

아래는 MATCH&#95;RECOGNIZE 절로 V 모양 패턴을 검색하는 쿼리다.  

```sql
SELECT * FROM ticker -- (1)
MATCH_RECOGNIZE (
  PARTITION BY symbol -- (2)
  ORDER BY tstamp -- (3)
  MEASURES -- (8)
      strt.tstamp AS start_tstamp
    , FINAL_LAST (down.tstamp) AS bottom_tstamp
    , FINAL_LAST (up.tstamp) AS end_tstamp
    , MATCH_NUMBER() AS match_num
    , CLASSIFIER() AS var_match
  ALL ROWS PER MATCH -- (7)
  AFTER MATCH SKIP TO LAST up -- (6) -> (5)
  PATTERN (strt down+ up+) -- (5)
  DEFINE -- (4)
      down AS down.price < PREV (down.price)
    , up AS up.price > PREV (up.price)) mr
ORDER BY mr.symbol, mr.match_num, mr.tstamp; -- (9)
```

MATCH&#95;RECOGNIZE 절은 아래의 순서로 동작한다.  

(1) 외부 쿼리를 조회  
(2) 조회 결과를 파티션으로 분할  
(3) 파티션 별로 행을 정렬  
(4) 패턴 변수와 조건을 확인  
(5) 파티션 별로 패턴을 조회하고 패턴이 일치하면 6번, 일치하지 않으면 7번을 수행  
(6) 패턴 일치를 다시 시작할 위치에서 5번을 수행  
(7) 일치 패턴의 반환 방식을 확인  
(8) 반환할 표현식을 계산  
(9) 결과를 반환  

#### 27.1.1. PARTITION BY 절
<br/>
PARTITION BY 절은 파티션을 지정한다.  

```
PARTITION BY column [, column]...
```

#### 27.1.2. ORDER BY 절
<br/>
ORDER BY 절은 행의 정렬 순서를 지정한다.  

```
ORDER BY column [, column]
```

#### 27.1.3. ROWS PER MATCH 절
<br/>
ROWS PER MATCH 절은 일치한 패턴의 반환 방식을 지정한다. 기본값은 ONE ROW PER MATCH다.  

```
{ONE ROW PER MATCH | ALL ROWS PER MATCH}
```

+ ONE ROW PER MATCH : 일치한 패턴 별로 1행을 출력 (기본값)
+ ALL ROWS PER MATCH : 일치한 전체 행을 출력  

#### 27.1.4. MEASURE 절
<br/>
MEASURE 절은 반환할 표현식을 정의한다.  

```
MEASURE expr AS c_alias [, expr AS c_alias]
```

#### 27.1.5. PARTTERN 절
<br/>
PARTTERN 절은 일치시킬 패턴을 지정한다.  

```
PATTERN (row_pattern)
```

##### 27.1.5.1. 패턴
<br/>
패턴(row_pattern)은 하나 이상의 패턴 용어로 구성된다. 패턴 용어는 수직선(|)으로 대체될 수 있다.  

```
row_pattern_term [| row_pattern_term]
```

아래 패턴은 a 패턴 변수와 일치한다.  

```sql
PATTERN (a)
```

아래 패턴은 a 패턴 변수나 b 패턴 변수나 c 패턴 변수와 일치한다.  

```sql
PATTERN (a | b | c)
```

##### 27.1.5.2. 패턴 용어
<br/>
패턴 용어(row_pattern_term)는 하나 이상의 패턴 요소로 구성된다. 패턴 용어는 공백 문자로 구분되며 기술 순서에 따라 일치된다.  

```
row_pattern_factor [row_pattern_factor]
```

아래 패턴은 a, b 패턴 변수와 차례대로 일치된다.  

```sql
PATTERN (a b)
```

아래 패턴은 a, b 패턴 변수와 차례대로 일치되거나 c 패턴 변수와 일치된다.  

```sql
PATTERN (a b | c)
```

##### 27.1.5.3. 패턴 요소
<br/>
패턴 요소(row_pattern_factor)는 패턴 기본 요소와 패턴 수량사로 구성된다. 패턴 수량사는 선택적으로 기술할 수 있다.  

```
row_pattern_primary [row_pattern_quantifier]
```

##### 27.1.5.4. 패턴 기본 요소
<br/>
패턴 기본 요소(row_pattern_primary)는 아래와 같다.  

+ variable_name : 패턴 변수 또는 결합 패턴 변수
+ ^ : 파티션 첫 행의 앞쪽 위치와 일치
+ $ : 파티션 끝 행의 뒤쪽 위치와 일치
+ ( [row_pattern] ) : 패턴을 한 단위로 처리, ()는 빈 패턴
+ {- row_pattern -} : 제외
+ row_pattern_permute : 순열  

패턴 기본 요소는 아래와 같이 동작한다.  

+ PATTERN (^a+$) : 파티션의 모든 행이 a 패턴 변수와 1회 이상 일치
+ PATTERN ((a b){3} c) : (a b) 그룹이 3회 일치하고 c 패턴 변수가 일치  

##### 27.1.5.5. 패턴 수량사
<br/>
패턴 수량사(row_pattern_quantifier)는 일치의 반복 횟수를 지정한다. 수량사 뒤쪽에 물음표(?)를 기술하면 비탐욕적(nongreedy) 방식으로 동작한다.  

+ ?[?] : 0회 또는 1회 반복
+ &#42;[?] : 0회 또는 그 이상의 횟수로 반복
+ +[?] : 1회 또는 그 이 상의 횟수로 반복
+ {m}[?] : m회 반복
+ {m,}[?] : 최소 m회 반복
+ {,m}[?] : 최대 m회 반복
+ {m,n}[?] : 최소 m회, 최대 n회 반복  

패턴 수량사는 아래와 같이 동작한다.  

+ PATTERN (a&#42;) : a 패턴 변수와 0회 이상을 반복해서 일치 (0회인 경우 빈 일치)
+ PATTERN (a{3,6}) : a 패턴 변수와 최소 3회, 최대 6회를 반복해서 일치
+ PATTERN (a{,4}) : a 패턴 변수와 최대 4회를 반복해서 일치
+ PATTERN (x y&#42; z) : y 패턴 변수가 0회 또는 그 이상 횟수를 최대로 반복해서 일치
+ PATTERN (x y&#42;? z) : y 패턴 변수가 0회 또는 그 이상 횟수를 최대로 반복해서 일치  

##### 25.1.5.6. 패턴 요소 우선순위
<br/>
아래의 패턴 요소는 아래의 우선순위를 가진다.  

(1) 기본 요소  
(2) 수량사  
(3) 연결, 공란( )  
(4) 대체, 수직선(|)  

아래 패턴은 우선순위에 딸 두 번째 열의 패턴과 동일하고, 세 번째 열의 패턴과 동일하지 않다.  

<table>
  <thead>
    <tr>
      <td>패턴</td>
      <td>동일</td>
      <td>동일하지 않음</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>PATTERN (a b | c d)</td>
      <td>PATTERN ((a b | c d))</td>
      <td>PATTERN (a (b | c) d)</td>
    </tr>
    <tr>
      <td>PATTERN (a b&#42;)</td>
      <td>PATTERN (a (b&#42;))</td>
      <td>PATTERN ((a b)&#42;)</td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 26.1.6. SUBSET 절
<br/>
SUBSET 절은 결합 패턴 변수(union row pattern variables)을 정의한다. SUBSET 절에 정의한 결합 패턴 변수는 MEASURE 절과 DEFINE 절에 사용할 수 있다.  

```
SUBSET variable_name = (variable_name [, variable_name]...)
    [, variable_name = (variable_name [, variable_name]...)]
```

SUBSET 절에 다수의 결합 패턴 변수를 정의할 수 있다. 아래 예제에서 xy 결합 패턴 변수는 x 패턴 변수와 y 패턴 변수의 결합이고, wz 겷합 변수는 w 패턴 변수와 z 패턴 변수의 결합이다.  

```sql
PATTERN (w&#43; x&#43; y&#43; z&#43;)
SUBSET xy = (x, y), wz = (w, z)
```

#### 26.1.7. DEFINE 절
<br/>
DEFINE 절은 패턴 변수와 조건을 정의한다.  

```
DEFINE variable_name AS condition [, variable_name AS condition]
```

아래 DEFINE 절은 up 패턴 변수와 down 패턴 변수를 정의한다. PREV 함수는 이전에 일치한 패턴의 값을 반환한다. up 패턴 변수는 현재 price가 이전 price보다 높아야 하고, down 패턴 변수는 현재 price가 이전 price보다 낮아야 한다.  

```sql
DEFINE
    up AS up.price > PREV (up.price)
  , down AS down.price < PREV (down.price)
```

DEFINE 절에 결합 패턴 변수를 사용할 수도 있다.  

```sql
PATTERN (a b&#43; c&#43; d)
SUBSET xy = (x, y), wz = (w, z)
DEFINE
    a AS price > 100
  , b AS b.price > a.price
  , c AS c.price > AVG(b.price)
  , d AS d.price > MAX(bc.price)
```

#### 27.1.8. SKIP TO 절
<br/>
SKIP TO 절은 패턴 일치를 다시 시작하는 위치를 지정한다. 아래의 다섯 가지 방식으로 사용할 수 있다.  

```
AFTER MATCH { SKIP TO NEXT ROW | SKIP TO LAST ROW | SKIP TO FIRST variable_name | SKIP TO LAST variable_name | SKIP TO variable_name}
```

+ SKIP TO NEXT ROW : 일치한 패턴의 첫 행의 다음 행부터 시작
+ SKIP TO LAST ROW : 일치한 패턴의 끝 행의 다음 행부터 시작 (기본값)
+ SKIP TO FIRST pattern&#95;variable : 패턴 변수의 첫 행부터 시작
+ SKIP TO LAST pattern&#95;variable : 패턴 변수의  끝 행부터 시작
+ SKIP TO pattern&#95;variable : SKIP TO LAST pattern&#95;variable 방식과 동일  

아래 SKIP TO 절은 a 패턴 변수가 빈 일치일 경우 에러가 발생한다. 빈 일치(empty match)는 ?, &#42; 패턴 수량사를 사용한 패턴 변수가 0회 일치한 것이다.  

```sql
AFTER MATCH SKIP TO a PATTERN (x a&#42; x)
```

아래 SKIP TO 절은 무한 루프가 발생하므로 에러가 발생한다.  

```sql
AFTER MATCH SKIP TO x PATTERN (x y&#43; z)
```

아래 SKIP TO 절은 a 패턴 변수가 빈 일치일 경우 에러가 발생하고, 일치한 경우에도 무한 루프가 발생하므로 에러가 발생한다.  

```sql
AFTER MATCH SKIP TO a PATTERN (a&#42; x)
```

아래 SKIP TO 절은 에러가 발생하지 않는다. a 패턴 변수나 b 패턴 변수와 일치하면 (a | b) 그룹에서 패턴 일치를 다시 시작할 수 있다.  

#### 27.1.9. MEASURE 절과 DEFINE 절의 표현식
<br/>
MEASURE 절과 DEFINE 절에 다양한 표현식을 사용할 수 있다.  

##### 27.1.9.1. 열 참조
<br/>
열 참조(row pattern column reference)는 패턴 변수를 사용한다. 패턴 변수를 기술하지 않으면 범용 패턴 변수가 사용된다. 범용 패턴 변수(universal row pattern variable)는 암시적으로 생성되는 모든 패턴 변수의 결합 패턴 변수다.  

열 참조는 아래와 같이 동작한다.  

+ a.price : a 패턴 변수와 price 열
+ price : 범용 패턴 변수와 price 열 (전체 패턴)  

##### 27.1.9.2. 패턴 인식 함수
<br/>
MODEL 절은 패턴 인식 함수(row pattern recognition function)를 제공한다. MEASURE 절과 DEFINE 절에 사용할 수 있다. 패턴 인식 함수는 파티션 별로 동작한다.  

###### 27.1.9.2.1. CLASSIFIER 함수
<br/>
CLASSIFIER 함수는 일치된 패턴 변수를 반환한다. ONE ROW PER MATCH 방식인 경우 마지막으로 일치한 행의 패턴 변수를 반환한다.  

```
CLASSIFIER ()
```

###### 27.1.9.2.2. MATCH&#95;NUMBER 함수
<br/>
MATCH&#95;NUMBER 함수는 일치된 패턴 순번을 반환한다. 0부터 1씩 증가한다.  

```
MATCH_NUMBER ()
```

###### 27.1.9.2.3. FIRST 함수
<br/>
FIRST 함수는 패턴 변수와 일치한 offset번째 행의 표현식을 반환한다. 일치한 패턴 변수가 없거나 offset번째 행이 존재하지 않으면 넝릉 반환한다. 논리적인 offset으로 동작하며 offset의 기본값은 0이다.  

```
[RUNNING | FINAL] FIRST (expr, [, offset])
```

FIRST 함수는 아래와 같이 동작한다.  

+ FIRST (a.price) : a 패턴 변수와 일치한 첫 번째 행의 price를 반환
+ FIRST (a.price, 1) : a 패턴 변수와 일치한 두 번째 행의 price를 반환
+ FIRST (a.price + a.tax) : a 패턴 변수와 일치한 첫 번째 행의 price + tax를 반환  
+ FIRST (a.price + b.tax) : 에러 발생  

아래 표의 R1, R3, R5 행은 a 패턴 변수와 일치한다. FIRST 함수는 패턴 변수와 일치한 행에서 동작한다.  

<table>
  <thead>
    <tr>
      <td>행</td>
      <td>price</td>
      <td>일치</td>
      <td>함수</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>R1</td>
      <td>10</td>
      <td>a</td>
      <td>FIRST (a.price, 0), FIRST (a.price)</td>
    </tr>
    <tr>
      <td>R2</td>
      <td>20</td>
      <td>b</td>
      <td></td>
    </tr>
    <tr>
      <td>R3</td>
      <td>30</td>
      <td>a</td>
      <td>FIRST (a.price, 1)</td>
    </tr>
    <tr>
      <td>R4</td>
      <td>40</td>
      <td>c</td>
      <td></td>
    </tr>
    <tr>
      <td>R5</td>
      <td>50</td>
      <td>a</td>
      <td>FIRST (a.price, 2)</td>
    </tr>
  </tbody>
</table>
<br/><br/>

###### 27.1.9.2.4. LAST 함수
<br/>
FIRST 함수는 패턴 변수와 일치한 offset번째 행의 표현식을 반환한다. 일치한 패턴 변수가 없거나 offset번째 행이 존재하지 않으면 넝릉 반환한다. 논리적인 offset으로 동작하며 offset의 기본값은 0이다.  

```
[RUNNING | FINAL] LAST (expr, [, offset])
```

아래 표의 R1, R3, R5 행은 a 패턴 변수와 일치한다. LAST 함수는 패턴 변수와 일치한 행에서 동작한다.  

<table>
  <thead>
    <tr>
      <td>행</td>
      <td>price</td>
      <td>일치</td>
      <td>함수</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>R1</td>
      <td>10</td>
      <td>a</td>
      <td>LAST (a.price, 2)</td>
    </tr>
    <tr>
      <td>R2</td>
      <td>20</td>
      <td>b</td>
      <td></td>
    </tr>
    <tr>
      <td>R3</td>
      <td>30</td>
      <td>a</td>
      <td>LAST (a.price, 1)</td>
    </tr>
    <tr>
      <td>R4</td>
      <td>40</td>
      <td>c</td>
      <td></td>
    </tr>
    <tr>
      <td>R5</td>
      <td>50</td>
      <td>a</td>
      <td>LAST (a.price, 0), LAST (a.price)</td>
    </tr>
  </tbody>
</table>
<br/><br/>

###### 27.1.9.2.5. PREV 함수
<br/>
PREV 함수는 마지막으로 일치한 패턴 변수의 첫 행에서 offset행 이전의 표현식을 반환한다. 일치한 패턴 변수가 없거나 offset 이전 행이 존재하지 않으면 널을 반환한다. 물리적인 offset으로 동작하며 패턴 변수에 종속되지 않는다. offset의 기본값은 1이다.  

```
PREV (expr [, offset])
```

PREV 함수는 아래와 같이 동작한다.  

+ PREV (a.price, 0) : a.price 반환
+ PREV (a.price, 1) : a 패턴 변수와 일치한 첫 행에서 1행 이전의 price 반환
+ PREV (a.price, 2) : a 패턴 변수와 일치한 첫 행에서 2행 이전의 price 반환  

아래 a 패턴 변수는 절대 일치하지 않는다. 일치하지 않은 a 패턴 변수의 이전 행은 존재하지 않는다.  

```sql
DEFINE a AS PREV (a.price) > 100
```

아래 a 패턴 변수는 마지막으로 일치한 b 패턴 변수 이전 행의 price가 100보다 큰 경우 일치한다.  

```sql
DEFINE a AS PREV (b.price) > 100
```

아래 PREV 함수는 하나 이상의 열을 참조한다.  

```sql
DEFINE a AS PREV (a.price + a.tax) < 100
```

###### 27.1.9.2.6. NEXT 함수
<br/>
NEXT 함수는 마지막으로 일치한 패턴 변수의 첫 행에서 offset행 이후의 표현식을 반환한다. 일치한 패턴 변수가 없거나 offset 이후 행이 존재하지 않으면 널을 반환한다. 물리적인 offset으로 동작하며 패턴 변수에 종속되지 않는다. offset의 기본값은 1이다.  

```
NEXT (expr [, offset])
```

래 a 패턴 변수는 마지막으로 일치한 a 패턴 변수 이전 행의 price가 100보다 큰 경우 일치한다.  

```sql
DEFINE a AS PREV (a.price) > 100
```

아래 a 패턴 변수는 price가 마지막으로 일치한 a 패턴 변수의 이전 2행, 이후 2행의 price 평균 값보다 큰 경우 일치한다.  

```sql
DEFINE a AS a.price > ( PREV (a.price, 2) + PREV (a.price, 1) + NEXT (a.price, 1) + NEXT (a.price, 2)) / 4
```

PREV 함수와 NEXT 함수는 R3 행을 기준으로 아래와 같이 동작한다.  

<table>
  <thead>
    <tr>
      <td>행</td>
      <td>price</td>
      <td>일치</td>
      <td>함수</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>R1</td>
      <td>10</td>
      <td>a</td>
      <td>PREV (b.price, 2)</td>
    </tr>
    <tr>
      <td>R2</td>
      <td>20</td>
      <td>b</td>
      <td>PREV (b.price, 1)</td>
    </tr>
    <tr>
      <td>R3</td>
      <td>30</td>
      <td>a</td>
      <td>PREV (b.price, 0), NEXT (b.price, 0)</td>
    </tr>
    <tr>
      <td>R4</td>
      <td>40</td>
      <td>c</td>
      <td>NEXT (b.price, 1)</td>
    </tr>
    <tr>
      <td>R5</td>
      <td>50</td>
      <td>a</td>
      <td>NEXT (b.price, 2)</td>
    </tr>
  </tbody>
</table>
<br/><br/>

##### 27.1.9.3. 집계 함수
<br/>
MEASURES 절과 DEFINE 절에 COUNT, MIN, MAX, SUM, AVG 등의 집계 함수를 사용할 수 있다.  

```
[RUNNING | FINAL] aggregate_function
```

집계 함수는 아래와 같이 동작한다.  

+ SUM(price) : 전체 price 범용 패턴 변수 사용
+ SUM(a.price) : a 패턴 변수와 일치한 행의 price
+ SUM(a.price + a.tax) : a 패턴 변수와 일치한 행의 price + tax
+ COUNT (&#42;) : 전체 행
+ COUNT (a.&#42;) : a 패턴 변수와 일치한 행
+ COUNT (a.price) : a 패턴 변수와 일치한 행의 price (널 제외)  

아래 집계 함수는 에러가 발생한다. 볌용 패턴 변수와 a 패턴 함수를 함께 사용했기 때문이다.  

```sql
SUM (price + a.tax)
```

##### 27.1.9.4 RUNNING 키워드와 FINAL 키워드
<br/>
FIRST 함수, LAST 함수, 집계 함수에 RUNNING 키워드나 FINAL 키워드를 기술할 수 있다. 기본값은 RUNNING이다.  

<table>
  <thead>
    <tr>
      <td>키워드</td>
      <td>설명</td>
      <td>사용</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>RUNNING</td>
      <td>누적 값을 반환 (기본값)</td>
      <td>MEASURES 절, DEFINE 절</td>
    </tr>
    <tr>
      <td>FINAL</td>
      <td>최종 값을 반환</td>
      <td>MEASURES 절</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 MATCH&#95;RECOGNIZE 절을 예로 들어보자.  

```sql
MEASURES
    RUNNING AVG (a.price) AS r_avg -- 누적 값
  , FINAL AVG (a.price) AS f_avg -- 최종 값
PATTERN (a+)
DEFINE a AS a.price >= AVG (a.price)
```

데이터가 아래와 같다면 R1, R2, R3 행이 a 패턴 변수와 일치한다. MEASURES 절은 누적 값이나 최종 값을 선택하여 반환할 수 있다. DEFINE 절에 사용한 집계 함수는 누적 값을 반환한다.  

<table>
  <thead>
    <tr>
      <td>행</td>
      <td>price</td>
      <td>a</td>
      <td>일치</td>
      <td>r&#95;avg</td>
      <td>f&#95;avg</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>R1</td>
      <td>10</td>
      <td>10 &gt;= 10 (= 10/1)</td>
      <td>a</td>
      <td>10</td>
      <td>13</td>
    </tr>
    <tr>
      <td>R2</td>
      <td>16</td>
      <td>16 &gt;= 13 (=(10+16)/2)</td>
      <td>a</td>
      <td>13</td>
      <td>13</td>
    </tr>
    <tr>
      <td>R3</td>
      <td>13</td>
      <td>13 &gt;= 13 (=(10+16+13)/3)</td>
      <td>a</td>
      <td>13</td>
      <td>13</td>
    </tr>
    <tr>
      <td>R4</td>
      <td>9</td>
      <td>9 &gt;= 12 (=(10+16+13+9)/4)</td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 MATCH&#95;RECOGNIZE 절을 예로 들어보자.  

```sql
PATTERN (a? b+)
DEFINE
    a AS a.price > 100
  , b AS b.price > COUNT (a.*) * 50
```

데이터가 아래와 같다면 R1, R2, R3 행이 b 패턴 변수와 일치한다. a 패턴 변수는 ? 수량사에 의해 빈 일치된다.  

<table>
  <thead>
    <tr>
      <td>행</td>
      <td>price</td>
      <td>a</td>
      <td>b</td>
      <td>일치</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>R1</td>
      <td>60</td>
      <td>60 &gt; 100</td>
      <td>60 &gt; 0 (=0&#42;50)</td>
      <td>b</td>
    </tr>
    <tr>
      <td>R2</td>
      <td>70</td>
      <td>70 &gt; 100</td>
      <td>70 &gt; 0 (=0&#42;50)</td>
      <td>b</td>
    </tr>
    <tr>
      <td>R3</td>
      <td>40</td>
      <td>40 &gt; 100</td>
      <td>80 &gt; 0 (=0&#42;50)</td>
      <td>b</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 MATCH&#95;RECOGNIZE 절을 예로 들어보자.  

```sql
PATTERN (a? b+)
DEFINE
    a AS COUNT (b.*) > 3
  , b AS b.price > 10
```

데이터가 아래와 같다면 R2, R3, R4, R5 행이 b 패턴 변수와 일치하지만 전체 패턴은 일치하지 않는다. a 패턴 변수는 + 수량사에 의해 적어도 1회 이상 일치해야 하기 때문이다.  

<table>
  <thead>
    <tr>
      <td>행</td>
      <td>price</td>
      <td>a</td>
      <td>b</td>
      <td>일치</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>R1</td>
      <td>2</td>
      <td>0 &gt; 3</td>
      <td>2 &gt; 10</td>
      <td></td>
    </tr>
    <tr>
      <td>R2</td>
      <td>11</td>
      <td>0 &gt; 3</td>
      <td>11 &gt; 10</td>
      <td>b</td>
    </tr>
    <tr>
      <td>R3</td>
      <td>12</td>
      <td>1 &gt; 3</td>
      <td>12 &gt; 10</td>
      <td>b</td>
    </tr>
    <tr>
      <td>R4</td>
      <td>13</td>
      <td>2 &gt; 3</td>
      <td>13 &gt; 10</td>
      <td>b</td>
    </tr>
    <tr>
      <td>R5</td>
      <td>14</td>
      <td>3 &gt; 3</td>
      <td>14 &gt; 10</td>
      <td>b</td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 27.1.10. 행 패턴 반환
<br/>
MATCH&#95;RECOGNIZE 절을 사용한 쿼리는 ROWS PER MATCH 절에 따라 아래의 순서로 결과를 반환한다.  

+ ONE ROW PER MATCH : PARTITION 절의 열 -> MEASURE 절의 열
+ ALL ROWS PER MATCH : PARTITION 절의 열 -> MEASURE 절의 열 -> 입력 테이블의 나머지 열  

### 27.2. 고급 주제
<br/>
#### 27.2.1. 중첩 탐색 함수
<br/>
FIRST 함수와 LAST 함수는 PREV 함수와 NEXT 함수로 중첩할 수 있다.  

```
{PREV | NEXT} ([RUNNING | FINAL] {FIRST | LAST} (expr [, offset]) [, offset])
```

아래 함수를 예로 들어보자.  

```sql
PREV (LAST (a.price + a.tax, 1), 3)
```

위 함수는 아래와 같이 동작한다. LAST 함수로 행(R4)을 찾은 후 PREV 함수로 행(R1)을 탐색한다. 결과는 11(=10+1)이다.  

<table>
  <thead>
    <tr>
      <td>행</td>
      <td>price</td>
      <td>tax</td>
      <td>일치</td>
      <td>함수</td>
    </tr>
  </thead?
  <tbody>
    <tr>
      <td>R1</td>
      <td>10</td>
      <td>1</td>
      <td></td>
      <td>PREV (expr, 3)</td>
    </tr>
    <tr>
      <td>R2</td>
      <td>20</td>
      <td>2</td>
      <td>a</td>
      <td></td>
    </tr>
    <tr>
      <td>R3</td>
      <td>30</td>
      <td>3</td>
      <td>b</td>
      <td></td>
    </tr>
    <tr>
      <td>R4</td>
      <td>40</td>
      <td>4</td>
      <td>a</td>
      <td>LAST (expr, 1)</td>
    </tr>
    <tr>
      <td>R5</td>
      <td>50</td>
      <td>5</td>
      <td>c</td>
      <td></td>
    </tr>
    <tr>
      <td>R6</td>
      <td>60</td>
      <td>6</td>
      <td>a</td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 27.2.2. 빈 일치와 불일치 행
<br/>
ALL ROWS PER MATCH 방식은 빈 일치(empty matches)와 불일치 행(unmatched rows)의 반환 여부를 지정할 수 있다. 기본값은 SHOW EMPTY MATCHES다.  

```
ALL ROWS PER MATCH {SHOW EMPTY MATCHES | OMIT EMPTY MATCHES | WITH UNMATCHED ROWS}
```

<table>
  <thead>
    <tr>
      <td>옵션</td>
      <td>일치</td>
      <td>빈 일치</td>
      <td>불일치 행</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>SHOW EMPTY MATCHES</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
    </tr>
    <tr>
      <td>OMIT EMPTY MATCHES</td>
      <td>Y</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>WITH UNMATCHED ROWS</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 패턴을 예로 들어보자. 패턴의 시작에 있는 a&#42; 패턴 요소는 파티션의 첫 행부터 b 패턴 요소가 일치한 이전 행가지 일치하지 않을 수 있다.  

```sql
PATTERN (a* b a* c)
```

데이터가 아래와 같다면 옵션(SHOW, OMIT, WITH)에 따라 Y로 표시된 행이 반환된다.  

<table>
  <thead>
    <tr>
      <td>행</td>
      <td>price</td>
      <td>일치</td>
      <td>SHOW</td>
      <td>OMIT</td>
      <td>WITH</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>R1</td>
      <td>10</td>
      <td>빈 일치</td>
      <td>Y</td>
      <td></td>
      <td>Y</td>
    </tr>
    <tr>
      <td>R2</td>
      <td>20</td>
      <td>빈 일치</td>
      <td>Y</td>
      <td></td>
      <td>Y</td>
    </tr>
    <tr>
      <td>R3</td>
      <td>30</td>
      <td>b</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>R4</td>
      <td>40</td>
      <td>a</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>R5</td>
      <td>50</td>
      <td>c</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>R6</td>
      <td>60</td>
      <td>불일치</td>
      <td></td>
      <td></td>
      <td>Y</td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 27.2.3. 제외
<br/>
{-와 -} 사이에 패턴을 기술하면 결과에서 해당 패턴을 제외(exclusion)할 수 있다. ALL ROWS PER MATCH WITH UNMATCHED ROWS 방식에서 사용할 수 있다.  

```
{- row_pattern -}
```

아래 패턴은 첫 번째, 두 번째 패턴 요소와 마지막 패턴 요소를 제외한다.  

```sql
PATTERN ({- a b -} a c {- a -})
```

데이터가 아래와 같다면 R2, R3, R4, R5, R6 행이 패턴과 일치하지만 Y로 표시된 R2, R3, R4 행은 결과에서 제외된다.  

<table>
  <thead>
    <tr>
      <td>행</td>
      <td>price</td>
      <td>일치</td>
      <td>제외</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>R1</td>
      <td>10</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>R2</td>
      <td>20</td>
      <td>a</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>R3</td>
      <td>30</td>
      <td>b</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>R4</td>
      <td>40</td>
      <td>a</td>
      <td></td>
    </tr>
    <tr>
      <td>R5</td>
      <td>50</td>
      <td>c</td>
      <td></td>
    </tr>
    <tr>
      <td>R6</td>
      <td>60</td>
      <td>a</td>
      <td>Y</td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 27.2.4. 순열
<br/>
PERMUTE 구문은 순열로 구성된 패턴을 생성한다.  

```
PERMUTE (row_pattern [, row_pattern]...)
```

좌측 패턴은 우측 결과로 해석된다.  

+ PATTERN (PERMUTE (a, b, c)) : PATTERN ( a b c | a c b | b a c | b c a | c a b | c b a)
+ PATTERN (PERMUTE (x{3}, b c?, d)) : PATTERN ( (x{3} b c? d) | (x{3} d b c?) | (b c? x{3} d) | (b c? d x{3}) | (d x{3} b c?) | (d b c? x{3}))  

### 27.3. 활용 예제
<br/>
MATCH&#95;RECOGNIZE 절의 활용 예제를 살펴보자.  

아래는 ONE ROW PER MATCH 방식을 사용한 쿼리다. ONE ROW PER MATCH 옵션을 사용했기 때문에 일치 패턴 별로 1행을 반환한다. DEFINE 절에 기술되지 않은 패턴 변수(strt)는 무조건 일치한다.  

```sql
SELECT * FROM ticker
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      strt.tstamp AS start_tstamp
    , LAST (down.tstamp) AS bottom_tstamp
    , LAST (up.tstamp) AS end_tstamp
  ONE ROW PER MATCH
  AFTER MATCH SKIP TO LAST up
  PATTERN (strt down+ up+)
  DEFINE
      down AS down.price < PREV (down.price)
    , up AS up.price > PREV (up.price)) mr
ORDER BY mr.symbol, mr.start_tstamp;
```

아래는 MEASURE 절에 집계 함수를 사용한 쿼리다.  

```sql
SELECT * FROM ticker
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      MATCH_NUMBER () AS match_num
    , CLASSIFIER () AS var_match
    , FINAL COUNT (up.tstamp) AS up_days
    , FINAL COUNT (tstamp) AS total_days
    , RUNNING COUNT (tstamp) AS cnt_days
    , price - strt.price AS price_dif
  ALL ROW PER MATCH
  AFTER MATCH SKIP TO LAST up
  PATTERN (strt down+ up+)
  DEFINE
      down AS down.price < PREV (down.price)
    , up AS up.price > PREV (up.price)) mr
ORDER BY mr.symbol, mr.start_tstamp;
```

아래 쿼리는 하락, 상승, 하락, 상승을 나타내는 W 모양 패턴을 검색한다.  

```sql
SELECT * FROM ticker
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      MATCH_NUMBER () AS match_num
    , CLASSIFIER () AS var_match
    , strt.tstamp AS start_tstamp
    , FINAL LAST (up.tstamp) AS end_tstamp
  ALL ROW PER MATCH
  AFTER MATCH SKIP TO LAST up
  PATTERN (strt down+ up+ down+ up+)
  DEFINE
      down AS down.price < PREV (down.price)
    , up AS up.price > PREV (up.price)) mr
ORDER BY mr.symbol, mr.start_tstamp, mr.tstamp;
```

아래는 결합 패턴 변수(stdn)를 사용한 쿼리다.  

```sql
SELECT * FROM ticker
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      FIRST () AS strt_time
    , LAST () AS bottom
    , AVG (stdn.price) AS stdn_avgprice
  ONE ROW PER MATCH
  AFTER MATCH SKIP TO LAST up
  PATTERN (strt down+ up+)
  SUBSET stdn = (strt, down)
  DEFINE
      up AS up.price > PREV (up.price)
    , down AS down.price < PREV (down.price));
```

아래는 RUNNING 키워드와 FINAL 키워드를 사용한 쿼리다.  

```sql
SELECT m.symbol, m.tstamp, m.price, m.runningavg, m.finalavg FROM ticker
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      RUNNING AVG (a.price) AS runningavg
    , FINAL AVG (a.price) AS finalavg
  ALL ROWS PER MATCH
  PATTERN (a+)
  DEFINE a AS a.price >= AVG (a.price)) m;
```

아래는 제외({- -})를 사용한 쿼리다. 상승 구간인 b 패턴 변수만 반환한다.  

```sql
SELECT m.symbol, m.tstamp, m.matchno, m.classfr, m.price, m.avgp FROM ticker
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      FINAL AVG (s.price) AS avgp
    , CLASSIFIER () AS classfr
    , MATCH_NUMBER () AS matchno
  ALL ROWS PER MATCH
  AFTER MATCH SKIP TO LAST b
  PATTERN ({- a -} b+ {- c+ -})
  SUBSET s = (a, b)
  DEFINE
      a AS a.price >= 10
    , b AS b.price > PREV (b.price)
    , c AS c.price <= PREV (c.price)) m;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE ticker3wave PURGE;
CREATE TABLE ticker3wave (symbol VARCHAR2(10), tstamp DATE, price NUMBER);

INSERT INTO ticker3wave VALUES ('ACME', '2011-04-01', 1000);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-02', 775);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-03', 900);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-04', 775);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-05', 900);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-06', 775);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-07', 900);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-08', 775);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-09', 800);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-10', 550);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-11', 900);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-12', 800);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-13', 1100);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-14', 800);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-15', 550);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-16', 800);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-17', 875);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-18', 950);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-19', 600);
INSERT INTO ticker3wave VALUES ('ACME', '2011-04-20', 300);
COMMIT;
```

아래 쿼리는 정해진 비율(24%) 이하로 값이 하락한 패턴을 검색한다.  

```sql
SELECT * FROM ticker3wave
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      b.tstamp AS timestamp
    , a.price AS aprice
    , b.price AS bprice
    , ((b.price - a.price) * 100) / a.price AS pctdrop
  ONE ROW PER MATCH
  AFTER MATCH SKIP TO b
  PATTERN (a b)
  DEFINE b AS (b.price - a.price) / a.price < -0.24);
```

아래 쿼리는 정해진 비율(8%) 이하로 값이 하락한 후 원래 값 이상으로 상승한 패턴을 검색한다.  

```sql
SELECT * FROM ticker3wave
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      a.tstamp AS start_timestamp
    , a.price AS start_price
    , b.price AS drop_price
    , count (c.*) + 1 AS cnt_days
    , d.tstamp AS end_timestamp
    , d.price AS end_price
  ONE ROW PER MATCH
  AFTER MATCH SKIP PAST LAST ROW
  PATTERN (a b c* d)
  DEFINE
      b AS (b.price - a.price) / a.price < -0.08
    , c AS c.price < a.price
    , d AS d.price >= a.price);
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE tickervu PURGE;
CREATE TABLE tickervu (symbol VARCHAR2(10), tstamp DATE, price NUMBER);

INSERT INTO tickervu VALUES ('ACME', '2011-04-01', 12);
INSERT INTO tickervu VALUES ('ACME', '2011-04-02', 17);
INSERT INTO tickervu VALUES ('ACME', '2011-04-03', 19);
INSERT INTO tickervu VALUES ('ACME', '2011-04-04', 21);
INSERT INTO tickervu VALUES ('ACME', '2011-04-05', 25);
INSERT INTO tickervu VALUES ('ACME', '2011-04-06', 12);
INSERT INTO tickervu VALUES ('ACME', '2011-04-07', 15);
INSERT INTO tickervu VALUES ('ACME', '2011-04-08', 20);
INSERT INTO tickervu VALUES ('ACME', '2011-04-09', 24);
INSERT INTO tickervu VALUES ('ACME', '2011-04-10', 25);
INSERT INTO tickervu VALUES ('ACME', '2011-04-11', 19);
INSERT INTO tickervu VALUES ('ACME', '2011-04-12', 15);
INSERT INTO tickervu VALUES ('ACME', '2011-04-13', 25);
INSERT INTO tickervu VALUES ('ACME', '2011-04-14', 25);
INSERT INTO tickervu VALUES ('ACME', '2011-04-15', 14);
INSERT INTO tickervu VALUES ('ACME', '2011-04-16', 12);
INSERT INTO tickervu VALUES ('ACME', '2011-04-17', 12);
INSERT INTO tickervu VALUES ('ACME', '2011-04-18', 24);
INSERT INTO tickervu VALUES ('ACME', '2011-04-19', 23);
INSERT INTO tickervu VALUES ('ACME', '2011-04-20', 22);
COMMIT;
```

아래 쿼리는 V 모양 패턴과 U 모양 패턴을 검색한다.  

```sql
SELECT * FROM tickervu
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      strt.tstamp AS start_tstamp
    , down.tstamp AS bottom_tstamp
    , up.tstamp AS end_tstamp
  ONE ROW PER MATCH
  AFTER MATCH SKIP TO LAST up
  PATTERN (strt down+ flat* up+)
  DEFINE
      down AS down.price < PREV (down.price)
    , flat AS flat.price = PREV (flat.price)
    , up AS up.price > PREV (up.price)) mr
ORDER BY mr.symbol, mr.start_tstamp;
```

아래 쿼리는 엘리엇 파동(eliott wave) 패턴을 검색한다. 엘리엇 파동은 Λ 모양이 반복되는 패턴이다. Λ Λ Λ Λ Λ 모양 패턴을 검색한다.  

```sql
SELECT mr_elliott.* FROM ticker3wave
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      count(*) AS cnt, count(p.*) AS p, count(q.*) AS q
    , count(r.*) AS r, count(s.*) AS s, count(t.*) AS t
    , count(u.*) AS u, count(v.*) AS v, count(w.*) AS w
    , count(x.*) AS x, count(y.*) AS y, count(z.*) AS z
    , CLASSIFIER () AS cls, MATCH_NUMBER () AS mno
  ALL ROWS PER MATCH
  AFTER MATCH SKIP TO LAST z
  PATTERN (p q+ r+ s+ t+ u+ v+ w+ x+ y+ z+)
  DEFINE
      q AS q.price > PREV (q.price), r AS r.price < PREV (r.price)
    , s AS s.price > PREV (s.price), t AS t.price < PREV (t.price)
    , u AS u.price > PREV (u.price), v AS v.price < PREV (v.price)
    , w AS w.price > PREV (w.price), x AS x.price < PREV (x.price)
    , y AS y.price > PREV (y.price), z AS z.price < PREV (z.price)) mr_elliott
ORDER BY symbol, tstamp;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE tickerwavemulti PURGE;
CREATE TABLE tickerwavemulti (symbol VARCHAR2(10), tstamp DATE, price NUMBER);

INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-01', 36.25);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-02', 36.47);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-03', 36.36);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-04', 36.25);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-05', 36.36);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-06', 36.70);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-07', 36.50);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-08', 36.66);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-09', 36.98);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-10', 37.08);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-11', 37.43);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-12', 37.68);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-13', 37.66);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-14', 37.32);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-15', 36.16);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-16', 37.98);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-17', 37.19);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-18', 37.45);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-19', 37.79);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-20', 37.49);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-21', 37.30);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-22', 37.08);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-23', 37.34);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-24', 37.54);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-25', 37.69);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-26', 37.60);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-27', 37.93);
INSERT INTO tickerwavemulti VALUES ('ACME', '2010-05-28', 38.17);
COMMIT;
```

아래 쿼리도 엘리엇 파동 패턴을 검색한다. 연속 상승, 연속 하락의 각각 최소 3회, 최대 4회 반복되는 Λ Λ 모양 패턴을 검색한다.  

```sql
SELECT mr_ew.* FROM tickerwavemulti
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      v.tstamp AS start_t, z.tstamp AS end_t
    , COUNT (v.price) AS v, COUNT (w.price) AS w, COUNT (x.price) AS x
    , COUNT (y.price) AS y, COUNT (z.price) AS z
    , MATCH_NUMBER () AS mno
  ALL ROWS PER MATCH
  AFTER MATCH SKIP TO LAST z
  PATTERN (v w{3,4} x{3,4} y{3,4} z{3,4})
  DEFINE
      w AS w.price > PREV (w.price), x AS x.price < PREV (x.price)
    , y AS y.price > PREV (y.price), z AS z.price < PREV (z.price)) mr_ew
ORDER BY symbol, tstamp;
```

아래 쿼리는 W 모양 패턴을 검색한다. 중간 상승 부분인 r에서 패턴 일치를 다시 시작한다.  

```sql
SELECT mr_w.* FROM ticker3wave
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES
      MATCH_NUMBER () AS mno, p.tstamp AS start_t, t.tstamp AS end_t
    , MAX (p.price) AS p, MIN (q.price) AS q, MAX (r.price) AS r
    , MIN (s.price) AS s, MAX (t.price) AS t
  ALL ROWS PER MATCH
  AFTER MATCH SKIP TO LAST r
  PATTERN (p q+ r+ s+ t+)
  DEFINE
      q AS q.price < PREV (q.price), r AS r.price > PREV (r.price)
    , s AS s.price < PREV (s.price), t AS t.price > PREV (t.price)) mr_w
ORDER BY symbol, mno, tstamp;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE stock04 PURGE;

CREATE TABLE stock04 (
    symbol VARCHAR2(10)
  , tstamp TIMESTAMP
  , price NUMBER
  , volume NUMBER);

INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:00:00.000000', 35, 35000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:05:00.000000', 35, 15000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:10:00.000000', 35, 5000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:11:00.000000', 35, 42000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:16:00.000000', 35, 7000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:19:00.000000', 35, 5000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:20:00.000000', 35, 5000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:33:00.000000', 35, 55000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:36:00.000000', 35, 15000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:48:00.000000', 35, 15000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 12:59:00.000000', 35, 15000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 01:09:00.000000', 35, 55000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 01:19:00.000000', 35, 55000);
INSERT INTO stock04 VALUES ('ACME', '2010-01-01 01:29:00.000000', 35, 15000);
COMMIT;
```

아래 쿼리는 한 시간 동안 3만 주 이상의 거래가 3회 반복된 주식의 패턴을 검색한다.  

```sql
SELECT * FROM stock04
MATCH_RECOGNIZE (
  PARTITION BY symbol
  ORDER BY tstamp
  MEASURES SUM (a.volume) AS sum_of_large_valumes
  ALL ROWS PER MATCH
  AFTER MATCH SKIP PAST LAST ROW
  PATTERN (a {- b* -} a {- b* -} a)
  DEFINE
      a AS (a.volume > 30000 AND a.tstamp - FIRST (a.tstamp) < '0 01:00:00.00')
    , b AS (b.volume <= 30000 AND b.tstamp - FIRST (a.tstamp) < '0 01:00:00.00'));
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE events PURGE;
CREATE TABLE events (time_stamp NUMBER, user_id VARCHAR2(10));

INSERT INTO events VALUES (1, 'Mary');
INSERT INTO events VALUES (11, 'Mary');
INSERT INTO events VALUES (23, 'Mary');
INSERT INTO events VALUES (34, 'Mary');
INSERT INTO events VALUES (44, 'Mary');
INSERT INTO events VALUES (53, 'Mary');
INSERT INTO events VALUES (63, 'Mary');
INSERT INTO events VALUES (3, 'Richard');
INSERT INTO events VALUES (13, 'Richard');
INSERT INTO events VALUES (23, 'Richard');
INSERT INTO events VALUES (33, 'Richard');
INSERT INTO events VALUES (43, 'Richard');
INSERT INTO events VALUES (54, 'Richard');
INSERT INTO events VALUES (63, 'Richard');
INSERT INTO events VALUES (2, 'Sam');
INSERT INTO events VALUES (12, 'Sam');
INSERT INTO events VALUES (22, 'Sam');
INSERT INTO events VALUES (32, 'Sam');
INSERT INTO events VALUES (43, 'Sam');
INSERT INTO events VALUES (47, 'Sam');
INSERT INTO events VALUES (48, 'Sam');
INSERT INTO events VALUES (59, 'Sam');
INSERT INTO events VALUES (60, 'Sam');
INSERT INTO events VALUES (68, 'Sam');
COMMIT;
```

아래 쿼리는 클릭 로그를 세션화한다. 세션화(sessionization)는 사용자의 활동을 고유한 세션으로 정의하는 작업이다.  

```sql
SELECT time_stamp, user_id, session_id FROM events
MATCH_RECOGNIZE (
  PARTITION BY user_id
  ORDER BY time_stamp
  MEASURES MATCH_NUMBER () AS session_id
  ALL ROWS PER MATCH
  PATTERN (b s*)
  DEFINE s AS s.time_stamp - PREV (time_stamp) <= 10)
ORDER BY user_id, time_stamp;
```

아래는 위 쿼리를 집계한 결과다.  

```sql
SELECT session_id, user_id,  start_time, no_of_events, duration FROM events
MATCH_RECOGNIZE (
  PARTITION BY user_id
  ORDER BY time_stamp
  MEASURES
      MATCH_NUMBER () AS session_id
    , COUNT (*) AS no_of_events
    , FIRST (time_stamp) AS start_time
    , LAST (time_stamp) AS duration
  PATTERN (b s*)
  DEFINE s AS s.time_stamp - PREV (time_stamp) <= 10)
ORDER BY user_id, session_id;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE my_cdr PURGE;

CREATE TABLE my_cdr (
    caller NUMBER
  , callee NUMBER
  , start_time NUMBER
  , end_time NUMBER);

INSERT INTO my_cdr VALUES (1, 7, 1354, 1575);
INSERT INTO my_cdr VALUES (1, 7, 1603, 1829);
INSERT INTO my_cdr VALUES (1, 7, 1857, 2301);
INSERT INTO my_cdr VALUES (1, 7, 2320, 2819);
INSERT INTO my_cdr VALUES (1, 7, 2840, 2964);
INSERT INTO my_cdr VALUES (1, 7, 64342, 64457);
INSERT INTO my_cdr VALUES (1, 7, 85753, 85790);
INSERT INTO my_cdr VALUES (1, 7, 85808, 85985);
INSERT INTO my_cdr VALUES (1, 7, 86011, 86412);
INSERT INTO my_cdr VALUES (1, 7, 86437, 86546);
INSERT INTO my_cdr VALUES (1, 7, 163436, 163505);
INSERT INTO my_cdr VALUES (1, 7, 163534, 163967);
INSERT INTO my_cdr VALUES (1, 7, 163982, 164454);
INSERT INTO my_cdr VALUES (1, 7, 214677, 214764);
INSERT INTO my_cdr VALUES (1, 7, 214782, 215248);
INSERT INTO my_cdr VALUES (1, 7, 216056, 216271);
INSERT INTO my_cdr VALUES (1, 7, 216297, 216728);
INSERT INTO my_cdr VALUES (1, 7, 216747, 216853);
INSERT INTO my_cdr VALUES (1, 7, 261138, 261463);
INSERT INTO my_cdr VALUES (1, 7, 261493, 261864);
INSERT INTO my_cdr VALUES (1, 7, 261890, 262098);
INSERT INTO my_cdr VALUES (1, 7, 262115, 262655);
INSERT INTO my_cdr VALUES (1, 7, 301931, 302226);
INSERT INTO my_cdr VALUES (1, 7, 302248, 302779);
INSERT INTO my_cdr VALUES (1, 7, 302804, 302992);
INSERT INTO my_cdr VALUES (1, 7, 303015, 303258);
INSERT INTO my_cdr VALUES (1, 7, 303283, 303337);
INSERT INTO my_cdr VALUES (1, 7, 383019, 383378);
INSERT INTO my_cdr VALUES (1, 7, 383407, 383534);
INSERT INTO my_cdr VALUES (1, 7, 424800, 425096);
COMMIT;
```

아래 쿼리는 연결이 끊긴 전화 통화를 세션화한다. 동일한 송신자와 수신자가 60초 내에 다시 통화한 패턴을 검색한다.  

```sql
SELECT caller, callee, start_time, effective_call_duration
     , (end_time - start_time) - effective_call_duration AS total_interruption_duration
     , no_of_restarts, session_id
FROM my_cdr
MATCH_RECOGNIZE (
  PARTITION BY caller, callee
  ORDER BY start_time
  MEASURES
      a.start_time AS start_time
    , end_time AS end_time
    , SUM(end_time - start_time) AS effective_call_duration
    , COUNT(b.*) AS no_of_restarts
    , MATCH_NUMBER () AS session_id
  PATTERN (a b*)
  DEFINE b AS b.start_time - PREV (b.end_time) < 60);
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE event_log PURGE;

CREATE TABLE event_log (
    time DATE
  , userid VARCHAR2(30)
  , amount NUMBER(10)
  , event VARCHAR(10)
  , transfer_to VARCHAR2(10));

INSERT INTO event_log VALUES ('2012-01-01', 'john', 1000000, 'deposit', NULL);
INSERT INTO event_log VALUES ('2012-01-05', 'john', 1200000, 'deposit', NULL);
INSERT INTO event_log VALUES ('2012-01-06', 'john', 1000, 'deposit', 'bob');
INSERT INTO event_log VALUES ('2012-01-15', 'john', 1500, 'deposit', 'bob');
INSERT INTO event_log VALUES ('2012-01-20', 'john', 1500, 'deposit', 'allen');
INSERT INTO event_log VALUES ('2012-01-23', 'john', 1000, 'deposit', 'tim');
INSERT INTO event_log VALUES ('2012-01-26', 'john', 1000000, 'deposit', 'tim');
INSERT INTO event_log VALUES ('2012-01-27', 'john', 500000, 'deposit', NULL);
COMMIT;
```

아래 쿼리는 의심스러운 자금 이동의 패턴을 검색한다. 최초 2000 미만의 금액을 이체한 후, 2000 이하의 금액을 다른 수신자에게 2회 이상 이체하고, 첫 이체일로부터 30일, 마지막 이체일로부터 10일 이내 시점의 이체 금액이 1000000 이상이고, 이전 이체 금액의 합계가 20000보다 작은 경우를 의심스러운 자금 이동이라고 가정한다.  

```sql
SELECT * FROM (SELECT * FROM event_log WHERE event = 'transfer')
MATCH_RECOGNIZE (
  PARTITION BY userid
  ORDER BY time
  ALL ROWS PER MATCH
  PATTERN (z x{2,} y)
  DEFINE z AS (event = 'transfer' AND amount < 2000),
         x AS (event = 'transfer' AND amount < 2000 AND PREV (x.transfer_to) <> x.transfer_to),
         y AS (event = 'transfer' AND amount < 1000000 AND LAST (x.time) - z.time < 30 AND y.time - LAST (x.time) < 10 AND SUM(x.amount) + z.amount < 20000));
```
