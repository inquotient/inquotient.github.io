---
title: SELECT 문
categories:
- Unkind_SQL
feature_text: |
  ## 5. SELECT 문
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

#### 5.2.3. SAMPLE절
<br/>
테이블을 샘플링하여 조회할 수 있다. 대용량 테이블에 대한 통계 값을 생성할 때 활용할 수 있다.  

```sql
SAMPLE [BLOCK] (sample_percent) [SEED (seed_value)]
```

+ BLOCK  
블록 샘플링을 사용 (지정하지 않으면 로우 샘플링)
+ sample_percent  
샘플링 비율 (0.000001 <= sample_percent < 100)
+ SEED (seed_value)  
항상 동일한 샘플을 반환 (seed_value는 0 ~ 4294967295 범위의 정수)

아래는 SAMPLE절을 사용한 쿼리다.

```sql
SELECT * FROM dept SAMPLE(30);
```

#### 5.3.5. 슈도 칼럼
<br/>
테이블에 저장되지 않은 의사 칼럼으로 쿼리 수행 시점에 값이 계산된다. 구문에 따라 사용할 수 있는 슈도 칼럼이 다르다.  

+ 일반 : ROWID, ROWNUM, ORA&#95;ROWSCN
+ 계층 쿼리 : LEVEL, CONNECT&#95;BY&#95;ISLEAF, CONNECT&#95;BY&#95;ISCYCLE
+ 시퀀스 : CURRVAL, NEXTVAL
+ 버전 쿼리 : VERSIONS&#95;STARTSCN, VERSIONS&#95;STARTTIME, ...

##### 5.3.5.1. ROWID
<br/>
데이터베이스에서 행을 식별할 수 있는 고유 값으로 오브젝트, 파일, 블록, 행 번호의 조합으로 계산된다.  

+ 데이터 오브젝트 번호(data object number) : 6자리
+ 상대적 파일 번호(relative file number) : 3자리
+ 블록 번호(block number) : 6자리
+ 블록 내의 행 번호(row number) : 3자리

##### 5.3.6.1. 힌트
<br/>
옵티마이저에 명령을 전달하는 특별한 형태의 주석으로 힌트의 대부분은 실행 계획을 수립할 때 사용되지만, 일부 힌트는 SQL 문의 동작을 제어한다.  

+ 단일 행 힌트 : --+로 시작함
+ 다중 행 힌트 : /&#42;+로 시작해서 &#42;/로 끝남

##### 5.3.6.2. V$SQL&#95;HINT 뷰
<br/>
V$SQL&#95;HINT 뷰에서 힌트에 대한 정보를 조회할 수 있다. 12.2 버전을 기준으로 352개의 힌트가 존재한다.
