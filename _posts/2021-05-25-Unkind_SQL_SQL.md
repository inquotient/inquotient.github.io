---
title: SQL
categories:
- Unkind_SQL
feature_text: |
  ## 4. SQL
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 4.1. 역사
<br/>
SQL은 관계형 데이터베이스의 표준 언어다. SQL의 원래 이름은 SEQUEL이었다. SEQUEL은 System R의 데이터 관리를 위해 IBM에서 개발되었다.

SQL은 ANSI 표준이 늦게 정해진 탓에 RDBMS마다 문법이 조금씩 다르다.  

NoSQL 데이터베이스의 등장으로 SQL의 미래에 대한 회의적인 시각도 있었지만 Hive, Impala, Tajo 등 빅데이터 분석에 SQL을 사용할 수 있는 SQL on Hadoop 기술이 개발됨에 따라 SQL은 관계형 데이터베이스를 넘어 데이터 분석 분야의 표준 언어로 발전하고 있다.  

### 4.2. 특징
<br/>
SEQUEL(Structured English QUEry Language)은 '구조화된 영문 질의어'로 해석할 수 있다. 자연어에 가까운 프로그래밍 언어로 설계되어 프로그래머가 아니더라도 쉽게 접근할 수 있다.  

### 4.3. 종류
<br/>
+ SELECT : SELECT
+ DML(Data Manipulation Language) : INSERT, UPDATE, DELETE, MERGE
+ TCS(Transaction Control Statement) : COMMIT, ROLLBACK, SAVEPOINT, SET TRANSACTION
+ DDL(Data Definition Language) : CREATE, ALTER, DROP, TRUNCATE, COMMENT
+ DCL(Data Control Language) : GRANT, REVOKE
+ SCS(Session Control Language) : ALTER SESSION, SET ROLE  

### 4.4. 처리 과정
<br/>
SQL은 아래와 같은 과정으로 처리된다.  

(1) syntax check : SQL의 문법을 검사
(2) semantic check : 오브젝트와 권한의 존재 유무를 검사
(3) shared pool check : shared pool의 library cache에 SQL이 저장되어 있는지 검사
(4) optimization : SQL의 쿼리 변환과 최적화를 수행
(5) row source generator : SQL 엔진에 의해 수행될 프로그램 소스를 생성
(6) execution : SQL을 실행

### 4.5. 수행 과정
<br/>
SQL은 내부적으로 복잡한 과정을 통해 수행된다. SQL의 종류에 따라 수행 과정도 다르다.  

SELECT 문의 각 단계는 아래와 같이 동작한다. 1단계와 2단계 사이에서 앞서 살펴본 SQL의 처리 과정(파싱 -> 최적화 -> 실행)이 수행된다.  

(1) 클라이언트 프로세스가 서버 프로세스로 SELECT 문을 전달  
(2) buffer cache에 필요한 데이터 블록이 있는지 확인  
(3) 없으면 data file에서 데이터 블록을 읽어 buffer cache에 저장  
(4) 결과 집합을 클라이언트 프로세스에 전달  

DML 문은 아래의 과정으로 수행된다. LGWR 백그라운드 프로세스와 DBWn 백그라운드 프로세스는 주기적으로 버퍼의 내용을 파일로 저장한다.  

(1) 클라이언트 프로세스가 서버 프로세스로 DML 문을 전달  
(2) 언두 세그먼트를 할당하고 buffer cache에 필요한 블록이 있는지 확인  
(3) 없으면 data file에서 블록을 읽어 buffer cache에 저장하고 변경할 블록에 락을 설정
(4) 데이터 블록과 언두 블록의 변경 사항을 redo log buffer에 기록  
(5) 언두 블록에 변경 전 데이터를 저장하고 데이터 블록을 변경  
(6) 변경 결과를 클라이언트 프로세스에 전달  

COMMIT 문은 아래의 과정으로 수행된다. 변경된 블록이 data file에 모두 저장되지 않았더라도 redo log buffer가 online redo log 파일에 모두 저장되었다면 완료 여부를 반환한다.  

(1) 클라이언트 프로세스가 서버 프로세스로 COMMIT 문을 전달  
(2) 서버 프로세스가 LGWR 백그라운드 프로세스로 처리를 요청  
(3) redo log buffer가 online redo log 파일에 모두 저장되면 변경된 블록의 락을 해제
(4) 완료 여부를 클라이언트 프로세스에 전달`
