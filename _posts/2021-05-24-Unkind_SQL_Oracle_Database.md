---
title: 오라클 데이터베이스
categories:
- Unkind_SQL
feature_text: |
  ## 3. 오라클 데이터베이스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 3.1. 개념
<br/>
오라클 사에서 개발한 ORDBMS 제품  

#### 3.1.1. 사용자
<br/>
데이터베이스에 로그인할 수 있는 계정  

#### 3.1.2. 오브젝트  
<br/>
논리적인 데이터 구조로서, 사용자에게 종속될 수 있다.  

+ 스키마  
사용자에게 종속된 오브젝트의 논리적 집합
+ 세그먼트  
저장 공간이 필요한 오브젝트
+ 스키마 오브젝트 : 테이블, 클러스터, 인덱스, 뷰, 시퀀스, 시노님, 오브젝트 타입
+ 비 스키마 오브젝트 : 사용자, 롤, 디렉터리
+ 세그먼트 오브젝트 : 테이블, 클러스터, 인덱스
+ 비 세그먼트 오브젝트 : 뷰, 시퀀스, 시노님, 오브젝트 타입, 사용자, 롤, 디렉터리  

#### 3.1.3. 테이블
<br/>
데이터를 구성하는 기본 단위로 행과 열로 구성된다.

#### 3.1.4. 데이터 타입
<br/>
열은 데이터 타입을 지정할 수 있다.

+ 문자 : CHAR, VARCHAR2, CLOB, LONG, NCHR, NVARCHAR2, NCLOB
+ 숫자 : NUMBER, BINARY&#95;FLOAT, BINARY&#95;DOUBLE
+ 날짜 : DATE, TIMESTAMP, INTERNAL
+ 이진 : BLOB, BFILE, LONG RAW, RAW
+ 기타 : ROWID, UROWID  

#### 3.1.5. 데이터 무결성
<br/>
데이터의 정확성과 일관성이 유지되고 있는 상태  

+ 개체 무결성  
엔터티의 인스턴스가 속성이나 속성의 조합으로 식별되어야 함.  
(DBMS 기능 : PK 제약 조건, UNIQUE 제약 조건, NOT NULL 제약 조건)
+ 참조 무결성
자식 엔터티의 외래 식별자가 부모 엔터티의 기본 식별자에 존재해야 함.
(DBMS 기능 : FK 제약 조건, 트리거)
+ 범위 무결성  
속성값이 지정한 범위에 유효해야 함.  
(DBMS 기능 : 데이터 타입, 기본값, CHECK 제약 조건)
+ 사용자 정의 무결성  
개체 무결성, 참조 무결성, 범위 무결성에 속하지 않는 무결성
(DBMS 기능 : 트리거)

#### 3.1.6. 트랜잭션
<br/>
함께 수행해야 하는 작업의 논리적 단위  

+ 원자성(Atomicity)  
트랜잭션의 작업은 모두 수행되거나 모두 수행되지 않아야 함.
+ 일관성(Consistency)  
트랜잭션이 완료되면 데이터 무결성이 일관되게 보장되어야 함.
+ 격리성(Isolation)  
트랜잭션이 다른 트랜잭션으로부터 격리되어야 함.
+ 지속성(Durability)  
트랜잭션이 완료되면 장애가 발생하더라도 변경 내용이 지속되어야 함.

#### 3.1.7. 정적 데이터 딕셔너리 뷰
<br/>
데이터베이스의 메타데이터를 조회할 수 있는 읽기 전용 뷰  

정적 데이터 딕셔너리 뷰는 3개의 세트로 구성된다. 12.1 버전부터 컨테이너 데이터베이스에서 사용하는 CDB&#95;로 시작하는 뷰도 사용할 수 있다.  

<table>
	<thead>
		<tr>
			<td>접두어</td>
			<td>사용</td>
			<td>오브젝트</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>DBA&#95;</td>
			<td>데이터베이스 관리자</td>
			<td>모든 오브젝트</td>
		</tr>
		<tr>
			<td>ALL&#95;</td>
			<td>모든 사용자</td>
			<td>사용자에게 권한이 있는 오브젝트</td>
		</tr>
		<tr>
			<td>USER&#95;</td>
			<td>모든 사용자</td>
			<td>사용자가 소유한 오브젝트</td>
		</tr>
	</tbody>
</table>
<br/>

#### 3.1.8. 동적 성능 뷰
<br/>
데티어베이스의 동적 정보를 조회할 수 있는 읽기 전용 뷰로 V$로 시작하며 X$로 시작하는 동적 성능 테이블도 사용한다. 동적 성능 테이블은 SYS 사용자가 소유하고 있다. RAC의 경우 GV$로 시작하는 뷰에서 인스턴스별 정보를 조회할 수 있다.  

+ DICTIONARY  
정적 데이터 딕셔너리 뷰에 대한 정보를 제공
+ V$FIXED&#95;TABLE  
동적 성능 뷰와 동적 성능 테이블에 대한 정보를 제공

### 3.2. 구조
<br/>
#### 3.2.1. 데이터베이스와 인스턴스
<br/>
오라클 데이터베이스는 하나의 데이터베이스와 하나 이상의 인스턴스로 구성된다. 데이터베이스는 데이터를 저장하는 파일의 모음이다. 인스턴스는 SGA와 백그라운드 프로세스로 구성된다.  

오라클 데이터베이스는 아래의 상태를 거쳐 단계적으로 기동된다.  

<table>
	<thead>
		<tr>
			<td>순서</td>
			<td>상태</td>
			<td>설명</td>
			<td>관련 파일</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>1</td>
			<td>SHUTDOWN</td>
			<td>인스턴스가 없음</td>
			<td></td>
		</tr>
		<tr>
			<td>2</td>
			<td>NOMOUNT</td>
			<td>인스턴스가 시작됨</td>
			<td>parameter file</td>
		</tr>
		<tr>
			<td>3</td>
			<td>MOUNT</td>
			<td>데이터베이스의 상태를 검사</td>
			<td>control file</td>
		</tr>
		<tr>
			<td>4</td>
			<td>OPEN</td>
			<td>데이터베이스가 열림</td>
			<td>data file, online redo file</td>
		</tr>
	</tbody>
</table>
<br/>

+ single 서버  
하나의 데이터베이스와 하나의 인스턴스로 구성된 오라클 데이터베이스  
+ RAC(Real Application Clusters)  
하나의 데이터베이스와 2개 이상의 인스턴스로 구성하여 고가용성(High Availability, HA)과 성능 향상을 도모하는 기술  
+ V$DATABASE 뷰  
데이터베이스에 대한 정보를 조회할 수 있는 뷰
+ V$INSTANCE 뷰  
인스턴스에 대한 정보를 조회할 수 있는 뷰  

#### 3.2.2. 프로세스 구조
<br/>
오라클 데이터베이스의 프로세스는 백그라운드 프로세스와 서버 프로세스로 구분된다. 백그라운드 프로세스는 인스턴스가 시작될 때, 서버 프로세스는 사용자가 데이터베이스 서버에 접속할 때 생성된다. 서버 프로세스는 클라이언트 프로세스와 통신할 수 있다.  

백그라운드 프로세스는 데이터베이스 운여에 필요한 백그라운드 작업을 수행한다.  주요 백그라운드 프로세는 아래와 같이 동작한다.  

+ PMON(process monitor)  
다른 프로세스를 감시 (프로세스 정리)  
+ SMON(system monitor)  
시스템 레벨의 정리 작업을 담당 (인스턴스 복구)
+ CKPT(checkpoint)  
control file과 data file 헤더를 갱신하고 DBWn에 신호를 보냄
+ DBWn(database writer)  
database buffer cache의 dirty buffer를 data file에 저장
+ ARCH(archiver)  
online redo log 파일을 archived redo log 파일로 보관

#### 3.2.3. 메모리 구조
<br/>
+ SGA(System Global Area)  
백그라운드 프로세스와 서버 프로세스의 공유 메모리 영역으로, 인스턴스가 시작될 때 할당되고 종료될 때 해제된다.
+ PGA(Program Global Area)  
서버 프로세스의 독점 메모리 영역으로, 서버 프로세스가 생성될 때 할당되고 종료될 때 헤제된다.  

##### 3.2.3.1. SGA 주요 요소
<br/>
+ database buffer cache  
data file에서 읽은 data block의 복사본을 캐싱하고 변경을 기록
+ redo log buffer  
DB의 변경 사항을 online redo log 파일에 기록하기 전에 버퍼링
+ shared pool  
다양한 유형의 프로그램 데이터를 캐싱
+ library cache  
실행 가능한 SQL, PL/SQL 코드를 저장
+ data dictionary cache  
데이터 딕셔너리 정보를 캐시
+ large pool  
shared pool보다 큰 메모리를 할당하기 위한 영역  

PGA는 세션 정보(session memory)와 SQL 관련 데이터(private SQL area)가 저장된다.  

#### 3.2.4. 저장 구조
<br/>
오라클 데이터베이스는 물리 저장 구조와 논리 저장 구조가 분리되어 있다. 물리 구조를 변경해도 논리 구조에 영향을 주지 않는다.

##### 3.2.4.1. 물리 저장 구조
<br/>
물리 저장 구조(physical storage structure)는 파일로 저장되며 OS에서 확인할 수 있다. archived redo log 파일처럼 데이터베이스에 속하지 않는 파일도 존재한다.  

물리 저장 구조의 파일은 아래와 같다.  

+ data file  
세그먼트(테이블, 인덱스)가 저장되는 파일
+ control file  
데이터베이스의 물리적인 구성 요소에 대한 제어 파일
+ online redo log  
데이터베이스의 변경 사항을 저장하는 파일 세트
+ archived redo log  
online redo log의 오프라인 사본

##### 3.2.4.2. 논리 저장 구조
<br/>
논리 저장 구조(logical storage structure)는 오라클 데이터베이스 내부에서 관리된다. 논리 저장 구조와 물리 저장 구조는 아래의 관계를 가진다. 세그먼트는 하나의 테이블스페이스에 속하고, 다수의 데이터 파일에 저장될 수 있다.  

논리 저장 구조의 단위는 아래와 같다.  

+ 블록(data block)  
데이터를 저장하는 가장 작은 논리적 단위
+ 익스텐트(extent)  
논리적으로 연속된 data block의 집합 (공간을 확장하는 단위)
+ 세그먼트(segment)  
오브젝트에 할당된 extent의 집합
+ 테이블스페이스(tablespace)  
세그먼트를 포함하는 데이터베이스 저장 단위  

#### 3.2.5. 네트워크 구조
<br/>
오라클 데이터베이스는 아래와 같은 네트워크 구조를 가진다. 리스너(listener)는 데이터베이스 서버에서 동작하는 프로그램으로 오라클 데이터베이스의 접속을 처리한다. listener.ora 파일은 리스너의 설정 파일이다. tnsnames.ora 파일은 클라이언트 설정 파일로 데이터베이스 서버의 접속 정보가 저장된다.  

리스너 방식의 접속은 아래의 순서로 진행된다.  

(1) 클라이언트 프로세스가 데이터베이스 서버에 접속을 요청
(2) 리스너가 클라이언트 프로세스의 접속 요청을 수락하고 서버 프로세스를 생성
(3) 리스너가 서버 프로세스와 클라이언트 프로세스를 연결한 후 다른 접속 요청을 대기

#### 3.2.6. 애플리케이션 구조
<br/>
오라클 데이터베이스는 네트워크 구조의 단계(tier)에 따라 세 가지 유형의 애플리케이션 구조를 가질 수 있다.  

각각의 애플리케이션의 구조는 아래와 같이 동작한다. multitier 구조는 일정 수 이상의 커넥션을 유지할 수 있도록 커넥션 풀(connection pool)을 사용한다. 커넥션 풀을 사용하면 서버 프로세스 생성과 PGA 할당에 대한 부하를 경감시킬 수 있다.  

(1) local : 데이터베이스 서버 내에서 직접 접속  
(2) client-server : 클라이언트에서 데이터베이스 서버로 접속 (2-tier)  
(3) multitier : 애플리케이션 서버를 통해서 데이터베이스 서버로 접속 (3-tier)
