---
title: 조인
categories:
- Unkind_SQL
feature_text: |
  ## 11. 조인
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

##### 11.1.2.2. 아우터 조인
<br/>
아우터 기준이 아닌 일반 조건에 (+)기호를 기술하지 않으면 아우터 조인이 이너 조인으로 변경된다.  

##### 11.1.3. 조인차수
<br/>
조인되는 테이블의 차수로 데이터 모델의 관계 차수(cardinality)와 유사한 개념이지만, 조인 차수는 조인 조건, 조인 기준에 따라 변경될 수 있다.  

등가 조인 시 관계 차수가 1:M이고 조인 기준이 1이면 조인 기준의 행이 M으로 늘어날 수 있다.  

등가 조인 시 관계 차수가 1:M이고 조인 기준이 M이면 조인 기준의 행이 늘어나지 않는다.  

등가 조인 시 관계 차수가 1:M이고 M쪽의 PK가 모두 등호(=)로 입력되면 조인 차수는 1:1이 된다.  

### 11.2 기술 순서
<br/>
#### 11.2.1. FROM 절
<br/>
FROM 절의 테이블은 데이터의 논리적인 흐름에 따라 기술해야 한다. 데이터의 논리적인 흐름은 아래의 규칙을 통해 결정할 수 있다. 데이터 모델과 업무 요건을 이해해햐 테이블의 순서를 결정할 수 있는 것이다.  

```text
데이터 모델에 따라 조인 순서를 결정하고, 업무 요건에 따라 조인 순서를 조정한다.
```

#### 11.2.2 WHERE 절
<br/>
FROM 절에 첫 번째로 기술된 테이블의 일반 조건을 기술한 다음, FROM 절에 기술된 테이블의 순서에 다라 조인 조건과 일반 조건의 순서로 조건을 기술한다. 조인 조건은 가능한 PK와 FK 순서대로 기술하고, 먼저 조회된 테이블의 값이 입력되는 형태로 작성한다.  
