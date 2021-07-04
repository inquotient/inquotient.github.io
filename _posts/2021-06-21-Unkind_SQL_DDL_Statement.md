---
title:  DDL 문
categories:
- Unkind_SQL
feature_text: |
  ## 20. DDL 문
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

DDL은 Data Definition Language의 약자다. 데이터 정의어로 해석할 수 있다. DDL 문을 사용하면 데이터베이스 오브젝트를 생성, 변경, 삭제할 수 있다.  

오라클 데이터베이스는 다양한 데이터베이스 오브젝트를 제공한다. 자주 사용되는 테이블, 인덱스, 파티션, 뷰, 시퀀스, 시너님, 데이터베이스 링크를 살펴보자.  

### 20.1. 테이블
<br/>
테이블은 데이터를 저장하기 위한 오브젝트다. 열과 행으로 구성되며, 제약 조건과 인덱스를 생성할 수 있다.  

#### 20.1.1. 기본 문법
<br/>
##### 20.1.1.1. CREATE TABLE 문
<br/>
CREATE TABLE 문은 테이블을 생성한다. 열 정의 방식과 서브 쿼리 방식을 사용할 수 있다.  

###### 201.1.1.1.1. 열 정의 방식
<br/>
```
CREATE TABLE [schema.]table (
  column datatype [DEFAULT [ON NULL] expr]
  [, column datatype [DEFAULT [ON NULL] expr]]...) [TABLESPACE tablespace];
```

아래 쿼리는 t1 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (
  c1 NUMBER
  , c2 NUMBER(2) DEFAULT 2
  , c3 NUMBER(3) DEFAULT 3 NOT NULL);
```

&#42;&#95;TABLES 뷰에서 테이블에 대한 정보를 조회할 수 있다.  

```sql
SELECT table_name FROM user_tables WHERE table_name = 'T1';
```

&#42;&#95;TAB&#95;COLUMNS 뷰에서 열에 대한 정보를 조회할 수 있다.  

```sql
SELECT column_name, data_type, data_precision, data_scale, nullable, data_default
FROM user_tab_columns
WHERE table_name = 'T1'
ORDER BY column_id;
```

###### 20.1.1.1.2. 서브 쿼리 방식
<br/>
아래는 서브 쿼리 방식의 CREATE TABLE 구문이다. 테이블이 생성된 후 서브 쿼리의 결과가 테이블에 입력된다. 서브 쿼리 방식을 줄여서 CTAS(Create Table As Select)라고 부르기도 한다.  

```
CREATE TABLE [schema.]table [(
  column [DEFAULT [ON NULL] expr]
  [, column [DEFAULT [ON NULL] expr]]...)] [TABLESPACE tablespace]
AS subquery;
```

아래 쿼리는 CTAS 방식으로 테이블을 생성한다. 0 = 1 조건은 항상 FALSE기 때문에 서브 쿼리의 결과가 테이블에 입력되지 않는다. 테이블만 생성할 때 사용할 수 있다.  

```sql
DROP TABLE t2 PURGE;

CREATE TABLE t3 AS SELECT * FROM t1 WHERE 0 = 1;
```

CTAS 방식은 서브 쿼리 결과의 데이터 타입과 NOT NULL 제약 조건만 참조하고 기본값은 무시한다. &#42;&#95;TAB&#95;COLUMNS 뷰의 data&#95;default 열에서 기본값이 지정되지 않은 것을 확인할 수 있다.  

```sql
SELECT column_name, data_type, data_precision, data_scale, nullable, data_default
FROM user_tab_columns
WHERE table_name = 'T2'
ORDER BY column_id;
```

아래와 같이 열명(c4)과 기본값을 지정할 수 있다.  

```sql
DROP TABLE t2 PURGE;

CREATE TABLE t2 (c1, c2 DEFAULT 2, c4 DEFAULT 3) AS SELECT * FROM t1 WHERE 0 = 1;
```

&#42;&#95;TAB&#95;COLUMNS 뷰의 data&#95;default 열에서 기본값이 지정된 것을 확인할 수 있다.  

```sql
SELECT column_name, data_type, data_precision, data_scale, nullable, data_default
FROM user_tab_columns
WHERE table_name = 'T2'
ORDER BY column_id;
```

CTAS 방식이 서브 쿼리에 리터럴을 사용하면 데이터 타입이 암시적으로 결정된다. c3, c4 열처럼 CAST 함수를 사용하여 데이터 타입을 명시적으로 지정할 수 있다.  

```sql
DROP TABLE t2 PURGE;

CREATE TABLE t2
AS SELECT 1234 AS c1
        , 'AB' AS c2
        , CAST(1234 AS NUMBER(10)) AS c3
        , CAST('AB' AS VARCHAR2(10)) AS c4
FROM DUAL;
```

&#42;&#95;TAB&#95;COLUMNS 뷰에서 CAST 함수를 사용하지 않은 숫자 리터럴(c1)은 길이가 지정되지 않은 NUMBER 타입, 문자 리터럴(c2)은 값 길이의 CHAR 타입으로 생성된 것을 확인할 수 있다.  

```sql
SELECT column_name, data_type, data_precision, data_scale, nullable, data_default
FROM user_tab_columns
WHERE table_name = 'T2';
```

아래 쿼리는 에러가 발생한다. 널에 대한 데이터 타입의 길이를 결정할 수 없기 때문이다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 AS SELECT NULL AS c1 FROM DUAL;
```

CAST 함수로 데이터 타입을 지정하면 에러가 발생하지 않는다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 AS
SELECT CAST(NULL AS NUMBER) AS c1, CASE(NULL AS VARCHAR2(1)) AS c2 FROM DUAL;
```

###### 20.1.1.1.3. 테이블스페이스와 익스텐트
<br/>
테이블, 인덱스, 파티션은 세그먼트 오브젝트다. 세그먼트는 하나의 테이블스페이스에 저장된다. 하나의 세그먼트는 다수의 익스텐트로 구성되며, 하나의 익스텐트는 다수의 블록으로 구성된다.  

예제를 위해 아래와 같이 테이블을 생성하자. DDL 문은 세그먼트가 저장될 테이블스페이스를 지정할 수 있다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 TABLESPACE USERS AS
SELECT LEVEL AS c1 FROM DUAL CONNECT BY LEVEL <= 10000;
```

&#42;&#95;TABLES 뷰의 tablespace&#95;name 열에서 테이블이 저장된 테이블스페이스를 확인할 수 있다.  

```sql
SELECT tablespace_name FROM user_tables WHERE table_name = 'T1';
```

&#42;&#95;SEGMENTS 뷰에서 세그먼트에 대한 정보를 조회할 수 있다. t1 테이블은 크기가 196,608 바이트고, 24개의 블록으로 구성되어 있다.  

```sql
SELECT segment_type, tablespace_name, bytes, blocks
FROM user_segments
WHERE segment_name = 'T1';
```

&#42;&#95;EXTENTS 뷰에서 익스텐트에 대한 정보를 조회할 수 있다. t1 테이블은 3개의 익스텐트로 구성되어 있으며, 각각의 익스텐트 8개의 블록으로 구성되어 있다.  

```sql
SELECT extent_id, bytes, blocks FROM user_extents WHERE segment_name = 'T1';
```

세그먼트를 생성하는 DDL 문에 테이블스페이스를 지정하지 않으면 사용자의 기본 테이블스페이스에 세그먼트가 저장된다.  

```sql
SELECT default_tablespace FROM dba_users WHERE username = 'SCOTT';
```

과거에는 성능 개선을 위한 I/O 분산의 목적으로 테이블과 인덱스의 테이블스페이스를 별도로 구성하는 경우가 많았다. 최근에는 스토리지 레벨에서 데이터가 스트라이핑(striping)되도록 디스크 볼륨을 구성하기 때문에 성능보다 관리 측면에서 테이블스페이스를 설계하는 것이 일반적이다.  

###### 20.1.1.1.4. 단위 표시
<br/>
크가가 큰 숫자 값은 해석이 어렵다. 이런 경우 SI 단위로 값을 표시하면 가독성을 높일 수 있다.  

아래는 SI 단위를 정리한 표다.  

<table>
  <thead>
    <tr>
      <td>접두어</td>
      <td>기호</td>
      <td>배수</td>
      <td>10ⁿ</td>
      <td>십진수</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>킬로(Kilo)</td>
      <td>K</td>
      <td>천</td>
      <td>10³</td>
      <td>1,000</td>
    </tr>
    <tr>
      <td>메가(Mega)</td>
      <td>M</td>
      <td>백만</td>
      <td>10^6</td>
      <td>1,000,000</td>
    </tr>
    <tr>
      <td>기가(Giga)</td>
      <td>G</td>
      <td>십억</td>
      <td>10^9</td>
      <td>1,000,000,000</td>
    </tr>
    <tr>
      <td>테라(Tera)</td>
      <td>T</td>
      <td>조</td>
      <td>10¹²</td>
      <td>1,000,000,000,000</td>
    </tr>
    <tr>
      <td>페타(Peta)</td>
      <td>P</td>
      <td>천조</td>
      <td>10^15</td>
      <td>1,000,000,000,000,000</td>
    </tr>
    <tr>
      <td>엑사(Exa)</td>
      <td>E</td>
      <td>백경</td>
      <td>10^18</td>
      <td>1,000,000,000,000,000,000</td>
    </tr>
    <tr>
      <td>제타(Zetta)</td>
      <td>Z</td>
      <td>십해</td>
      <td>10^21</td>
      <td>1,000,000,000,000,000,000,000</td>
    </tr>
    <tr>
      <td>요타(Yetta)</td>
      <td>Y</td>
      <td>자</td>
      <td>10^24</td>
      <td>1,000,000,000,000,000,000,000,000</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래와 같이 사용자 정의 함수를 생성하자.  

```sql
CREATE OR REPLACE FUNCTION fnc_unit (
  i_val IN NUMBER
  , i_div IN NUMBER DEFAULT 1024
)
  RETURN VARCHAR2
IS
  l_val NUMBER := i_val;
  l_idx NUMBER := 1;
BEGIN
  CASE
    WHEN i_val IS NULL THEN
      RETURN NULL;
    ELSE
      LOOP
        EXIT WHEN l_val < i_div;
        l_val := l_val / i_div;
        l_idx := l_idx + 1;
      END LOOP;
      RETURN TO_CHAR (ROUND(l_val, 2)
                    , LPAD('9', LENGTH(i_div) - 1, '9') || '0.90')
          || TRANSLATE(l_idx, '123456789', ' KMGTPEZY');
  END CASE;
END fnc_unit;
/
```

아래는 fnc_unit 함수를 사용한 쿼리다. SI 단위로 값을 표시되는 것을 확인할 수 있다.  

```sql
SELECT initial_extent, fnc_unit (initial_extent) AS c1
     , max_extents   , fnc_unit (max_extents) AS c2
     , num_rows      , fnc_unit (num_rows, 1000) AS c3
FROM user_tables
WHERE table_name = 'EMP';
```

##### 20.1.1.2. ALTER TABLE 문
<br/>
ALTER TABLE 문은 테이블, 열, 제약 조건을 변경한다.  

###### 20.1.1.2.1. RENAME 절
<br/>
RENAME 절은 테이블 명을 변경한다.  

```
ALTER TABLE [schema.]table RENAME TO new_table_name;
```

아래 쿼리는 t2 테이블의 테이블 명을 t1으로 변경한다.  

```sql
DROP TABLE t1 PURGE;

ALTER TABLE t2 RENAME TO t1;
```

###### 20.1.1.2.2. MOVE 절
<br/>
MOVE 절은 세그먼트의 데이터를 재배치한다. INCLUDING ROWS와 UPDATE INDEXES는 12.2. 버전부터 사용할 수 있다.  

```
ALTER TABLE [schema.]table MOVE
  [INCLUDING ROWS where_clause]
  [ONLINE]
  [TABLESPACE tablespace]
  [UPDATE INDEXES] [(index [, index])];
```

+ INCLUDING ROWS  
재비치한 데이터의 조건을 지정
+ ONLINE  
DDL 문이 수행되는 동안 DML 문이 수행될 수 있도록 허용  
+ TABLESPACE  
테이블스페이스를 지정
+ UPDATE INDEXES  
인덱스도 함께 재구성  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 AS SELECT ROWNUM AS c1 FROM XMLTABLE ('1 to 10000');
```

아래 쿼리는 c1 <= 5000 조건에 해당하는 데이터를 users 테이블스페이스에 재비치한다. 테이블의 데이터를 재배치하면 행의 ROWID가 변경되기 때문에 종속된 인덱스를 재구축해야 한다.  

```sql
ALTER TABLE t1 MOVE INCLUDING ROWS WHERE c1 <= 5000 TABLESPACE users;
```

조건에 해당하지 않은 데이터는 재배치되지 않고 제거된다.  

```sql
SELECT COUNT(*) AS c1 FROM t1;
```

###### 20.1.1.2.3. READ ONLY
<br/>
READ ONLY는 테이블을 읽기 전용 테이블로 변경한다.  

```sql
ALTER TABLE [schema.]table {READ ONLY | READ WRITE};
```

+ READ ONLY : 테이블을 읽기 전용으로 변경
+ READ WRITE : 테이블을 읽기 쓰기로 변경 (기본값)  

아래 쿼리는 t1 테이블을 읽기 전용 테이블로 변경한다.  

```sql
ALTER TABLE t1 READ ONLY;
```

읽기 전용 테이블에 DML 작업을 수행하면 에러가 발생한다.  

```sql
INSERT INTO t1 (c1) VALUES (1);
```

&#42;&#95;TABLES 뷰의 read&#95;only 열에서 읽기 전용 여부를 확인할 수 있다.  

```sql
SELECT table_nane, read_only FROM user_tables WHERE table_name = 'T1';
```

##### 20.1.1.3. DROP TABLE 문
<br/>
DROP TABLE 문은 테이블을 삭제한다.  

```
DROP TABLE [schema.]table [CASCADE CONSTRAINTS] [PURGE];
```

+ CASCADE CONSTRAINTS : 테이블을 참조하는 FK 제약 조건을 함께 삭제
+ PURGE : recycle& bin을 사용하지 않고 테이블을 즉시 삭제  

아래 쿼리는 t1 테이블과 t1 테이블을 참조하는 FK 제약 조건을 즉시 삭제한다.  

```sql
DROP TABLE t1 CASCADE CONSTRAINTS PURGE;
```

##### 20.1.1.4. TRUNCATE TABLE 문
<br/>
TRUNCATE TABLE 문은 테이블을 초기화한다. 테이블의 저장 공간을 해제하기 때문에 DDL 문으로 분류되며 롤백이 불가능하다. 기본값은 DROP STORAGE다.  

```
TRUNCATE TABLE [schema.]table [{DROP [ALL] | REUSE} STORAGE] [CASCADE];
```

+ DROP STORAGE : MINEXTENTS 파라미터에 의해 할당된 공간을 제외한 공간을 해제
+ DROP ALL STORAGE : MINEXTENTS 파라미터에 의해 할당된 공간을 포함한 모든 공간을 해제
+ REUSE STORAGE : 삭제된 행의 공간을 유지
+ CASCADE : ON DELETE CASCADE FK 제약 조건으로 참조하는 테이블을 TRUNCATE  

아래 쿼리는 t1 테이블을 TRUNCATE한다. 테이블의 크기가 큰 경우 DROP ALL STORAGE를 사용하면 수행 시간을 단축시킬 수 있다.  

```sql
TRUNCATE TABLE t1;
```

#### 20.1.2. 테이블 유형
<br/>
오라클은 다양한 유형의 테이블을 제공한다. 구조, 분산, 저장 기준에 따라 아래와 같이 구분할 수 있다.  

+ 구조 : 힙 구조 테이블, 인덱스 구조 테이블, 익스터널 테이블
+ 분산 : 클러스터 테이블, 파티션 테이블
+ 저장 : 영구 테이블, 임시 테이블

##### 20.1.2.1. 힙 구조 테이블
<br/>
힙 구조 테이블(heap-organized table)은 힙(heap) 구조에 데이터를 저장한다. 힙 구조는 임의의 위치에 데이터를 저장한다. 지금까지 살펴본 테이블은 모두 힙 구조 테이블이다. 힙 구조 테이블은 오라클 데이터베이스의 기본 테이블이다.  

```
CREATE TABLE [schema.]table (
  column datatype [DEFAULT [ON NULL] expr]
  [, column datatype [DEFAULT [ON NULL] expr]]...
   , CONSTRAINT constraint_name PRIMARY KEY (column [, column]...))
ORGANIZATION HEAP;
```

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);
```

##### 20.1.2.2. 인덱스 구조 테이블
<br/>
인덱스 구조 테이블(index-organized table)은  b-tree 인덱스 구조에 데이터를 저장한다. PK 제약 조건의 열 순서에 따라 데이터가 정렬되어 저장된다. 주로 OLAP 시스템의 조회 성능을 개선하기 위해 사용한다. 인덱스 구조 테이블을 줄여서 IOT라고 부르기도 한다.  

```
CREATE TABLE [schema.]table (
  column datatype [DEFAULT [ON NULL] expr]
  [, column datatype [DEFAULT [ON NULL] expr]]...
   , CONSTRAINT constraint_name PRIMARY KEY (column [, column]...))
ORGANIZATION INDEX;
```

아래 쿼리는 t1 테이블을 IOT로 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, CONSTRAINT t1_pk PRIMARY KEY (c1))
ORGANIZATION INDEX;
```

&#42;&#95;OBJECTS 뷰에서 데이터베이스 오브젝트에 대한 정보를 조회할 수 있다. t1 오브젝트의 data&#95;object&#95;id 열이 널이다.  

```sql
SELECT object_name, data_object_id, object_type
FROM user_objects
WHERE object_name IN ('T1', 'T1_PK');
```

&#42;&#95;SEGMENTS 뷰에서 세그먼트에 대한 정보를 조회할 수 있다. t1 테이블은 논리적으로 존재하기 때문에 세그먼트가 조회되지 않는다. 물리적으로 PK 제약 조건이 사용하는 t1&#95;pk 인덱스에 저장된다.  

```sql
SELECT segment_name, segment_type
FROM user_segments
WHERE segment_name IN ('T1', 'T1_PK');
```

IOT는 &#42;&#95;TABLES 뷰의 iot&#95;type 열이 IOT로 표시된다. IOT는 인덱스 외부 영역인 오버플로우 영역에 저장될 수 있다. 오버플로우 영역은 IOT&#95;OVERFLOW로 표시된다.  

```sql
SELECT table_name, iot_type FROM user_tables WHERE table_name = 'T1';
```

#### 20.1.2.3. 익스터널 테이블
<br/>
익스터널 테이블(external table)은 외부 데이터를 조회하거나 외부에 데이터를 저장할 수 있는 데이터를 저장할 수 있는 테이블이다. 주로 DW 시스템의 ETL 작업에 사용된다. 익스터널 테이블은 두 가지 액세스 드라이버를 사용할 수 있다. ORACLE&#95;DATAPUMP 액세스 드라이버는 범위를 벗어나는 이야기이므로 ORACLE&#95;LOADER 액세스 드라이버만 살펴보자.  

```sql
CREATE TABLE [schema.]table (
  column datatype [DEFAULT [ON NULL] expr]
  [, column datatype [DEFAULT [ON NULL] expr]]...)
ORGANIZATION EXTERNAL (
  [TYPE access_driver_type]
    DEFAULT DIRECTORY directory
  [ACCESS PARAMETERS (opaque_format_spec)]
    LOCATION ([directory:] 'location_specifier'
           [, [directory:] 'location_specifier']...)
) [REJECT LIMIT {integer | UNLIMITED}];
```

+ ORACLE&#95;LOADER : SQL&#42;Loader를 통해 외부 파일에 대한 읽기 전용 작업을 수행
+ ORACLE&#95;DATADUMP : Data Pump를 통해 데이터를 로드(load) 또는 언로드(unload)  

예제를 위해 SYS 사용자로 로그인한 세션에서 디렉터리를 생성하고, 디렉터리에 대한 모든 권한을 SCOTT 사용자에 부여하자.  

```sql
CREATE OR REPLACE DIRECTORY dir_ext AS 'c:\app\ora12cr2\admin\ora12cr2\ext';

GRANT ALL ON DIRECTORY dir_ext TO scott;
```

&#42;&#95;DIRECTORIES 뷰에서 디렉터리에 대한 정보를 조회할 수 있다.  

```sql
SELECT owner, directory_name, directory_path
FROM all_directories
WHERE directory_name = 'DIR_EXT';
```

&#42;&#95;TAB&#95;PRIV 뷰에서 디렉터리 권한에 대한 정보를 조회할 수 있다.  

```sql
SELECT grantee, owner, table_name, grantor, privilege, type
FROM user_tab_privs
WHERE table_name = 'DIR_EXT';
```

예제를 위해 c:\app\ora12cr2\admin\ora12cr2\ext 위치에 아래의 내용으로 ext&#95;dept,.txt 파일을 생성하자.  

```
10,ACCOUNTING,NEW YORK
20,RESEARCH,DALLAS
30,SALES,CHICAGO
40,OPERATIONS,BOSTON
```

##### 20.1.2.3.1. ORACLE&#95;LOADER
<br/>
아래 쿼리는 ORACLE&#95;LOADER 액세스 드라이버로 익스터널 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (deptno NUMBER(2), dname VARCHAR2(14), loc VARCHAR2(13))
ORGANIZATION EXTERNAL (
  TYPE ORACLE_LOADER
  DEFAULT DIRECTORY dir_ext
  ACCESS PARAMETERS (
    RECORDS DELIMITED BY NEWLINE
    NOBADFILE NOLOGFILE NODISCARDFILE
    FILEDS TERMINATED BY ',')
  LOCATION ('ext_dept.ext')
) REJECT LIMIT UNLIMITED;
```

t1 테이블을 조회하면 ext&#95;dept.txt 파일의 내용을 조회할 수 있다.  

&#42;&#95;EXTERNAL&#95;TABLES 뷰에서 익스터널 테이블에 대한 정보를 조회할 수 있다.  

```sql
SELECT type_name, default_directory_name, access_parameters
FROM user_external_tables
WHERE table_name = 'T1';
```

&#42;&#95;EXTERNAL&#95;LOCATIONS 뷰에서 익스터널 테이블 위치에 대한 정보를 조회할 수 있다.  

```sql
SELECT location, directory_name FROM user_external_locations WHERE table_name = 'T1';
```

##### 20.1.2.3.2. PREPROCESSOR
<br/>
11.2 버전부터 PREPROCESSOR 기능을 사용할 수 있다. PREPROCESSOR 기능을 사용하면 데이터베이스 서버의 OS 명령어를 수행하고 수행 결과를 테이블로 조회할 수 있다.  

PREPROCESSOR 기능으로 압축 파일에 대한 ETL 작업을 수행해보자. 예제를 위해 c:\app\ora12cr2\admin\ora12cr2\ext 위치에 아래 내용으로 ext&#95;unzip.cmd 파일을 생성하자. 첫 번재 인수(%1)로 입력된 압축 파일을 해체하는 명령어다.  

```
@echo off
c:\app\ora12cr2\product\12.2.0\dbhome_1\bin\unzip.exe -qc %1
```

아래와 같이 zip 명령어로 ext&#95;dept.txt 파일을 압축하자.  

```
zip ext_dept.zip ext_dept.txt
```

아래 쿼리는 PREPROCESSOR 기능을 사용하는 익스터널 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (deptno NUMBER(2), dname VARCHAR2(14), loc VARCHAR2(13))
ORGANIZATION EXTERNAL (
  TYPE ORACLE_LOADER
  DEFAULT DIRECTORY dir_ext
  DEFAULT PARAMETERS (
    RECORDS DELIMITED BY NEWLINE
    PREPROCESSOR dir_ext: 'ext_unzip.cmd'
    NOBADFILE NOLOGFILE NODISCARDFILE
    FIELDS TERMINATED BY ',')
  LOCATION ('ext_dept.zip')
) REJECT LIMIT UNLIMITED;
```

t1 테이블을 조회하면 ext&#95;dept.zip 파일의 내용을 조회할 수 있다.  

```sql
SELECT * FROM t1;
```

PREPROCESSOR 기능으로 디렉터리의 파일 목록을 조회해보자. 예제를 위해 c:\app\ora12cr2\admin\ora12cr2\ext 위치에 아래 내용으로 ext&#95;dir.cmd 파일을 생성하자. 지정한 경로의 파일을 조회하는 명령어다.  

```
@echo off
dir \-c c:\app\ora12cr2\diag\rdbms\ora12cr2\ora12cr2\trace
```

아래와 같이 익스터널 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (
  yyyymmdd VARCHAR2(10)
  , ampm VARCHAR2(10)
  , hhmi VARCHAR2(10)
  , file_size NUMBER
  , file_name VARCHAR2(255)
) ORGANIZATION EXTERNAL (
  TYPE ORACLE_LOADER
  DEFAULT DIRECTORY dir_ext
  ACCESS PARAMETERS (
    RECORDS DELIMITED BY NEWLINE
    PREPROCESSOR dir_ext:'ext_dir.cmd'
    LOAD WHEN file_size != '<DIR>'
    NOBADFILE NOLOGFILE NODISCARDFILE
    DISABLE_DIRECTORY_LINK_CHECK
    FIELDS TERMINATED BY WHITESPACE
  ) LOCATION ('ext_dir.cmd')
) REJECT LIMIT UNLIMITED;
```

t1 테이블을 조회하면 ext&#95;dir.cmd 명령어의 수행 결과를 조회할 수 있다.  

```sql
SELECT TO_DATE(yyyymmdd || ampm || hhmi, 'YYYY-MM-DDAMHH:MI') AS file_date
     , file_size, file_name
FROM t1
ORDER BY 1 DESC;
```

##### 20.1.2.4. 클러스터 테이블
<br/>
클러스터 테이블(clustered table)은 클러시터에 데이터를 저장한다. 클러스터(cluster)는 클러스터 키가 동일한 데이터를 동일한 위치에 저장하는 세그먼트이다. 하나의 블록에 다수의 테이블 행이 저장될 수도 있다. 관리상의 어려움으로 인해 활용도는 그리 높지 않다.  

CREATE CLUSTER 문은 클러스터를 생성한다. expr은 해시 함수로 사용할 표현식이다.  

```
CREATE CLUSTER [schema.]cluster (
  column datatype [SORT]
  [, column datatype [SORT]]...)
  [ { {INDEX | [SINGLE TABLE] HASHKEYS integer [HASH IS expr] } }...]
[TABLESPACE tablespace];
```

클러스터는 클러스터 키를 처리하는 방식에 따라 아래와 같이 구분된다.  

+ INDEX : 인덱스 클러스터
+ HASHKEYS integer : 해시 클러스터
+ SINGLE TABLE : 단일 테이블 해시 클러스터
+ SORT : 정렬 해시 클러스터  

DROP CLUSTER 문은 클러스터를 삭제한다. INCLUDING TABLES를 기술하면 클러스터에 속한 전체 테이블을 삭제한다.  

```
DROP CLUSTER [schema.]cluster [INCLUDING TABLES [CASCADE CONSTRAINTS]];
```

TRUNCATE CLUSTER 문은 클러스터를 초기화한다.  

```
TRUNCATE CLUSTER [schema.]cluster [{DROP | REUSE} STORAGE];
```

클러스터 테이블은 아래의 CREATE TABLE 문으로 생성할 수 있다.  

```
CREATE TABLE [schema.]table (
  column datatype [DEFAULT [ON NULL] expr]
  [, column datatype [DEFAULT [ON NULL] expr]]...)
CLUSTER cluster (column [, column]...);
```

##### 20.1.2.4.1. 인덱스 클러스터
<br/>
인덱스 클러스터(indexed cluster)는 데이터의 저장과 조회에 인덱스를 사용한다.  

아래 쿼리는 인덱스 클러스터를 생성한다.  

```sql
DROP CLUSTER c1# INCLUDING TABLES;

CREATE CLUSTER c1# (c1 NUMBER) INDEX;
```

아래 쿼리는 c1# 클러스터에 저장되는 인덱스 클러스터 테이블을 생성한다. t1, t2 테이블은 동일한 클러스터 블록에 저장된다. t1, t2 테이블을 클러스터 키로 조인하면 쿼리의 조회 성능이 향상될 수 있다.  

```sql
DROP TABLE t1 PURGE;
DROP TABLE t2 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER) CLUSTER c1# (c1);
CREATE TABLE t2 (c1 NUMBER, c2 NUMBER) CLUSTER c1# (c1);
```

&#42;&#95;CLUSTERS 뷰에서 클러스터에 대한 정보를 조회할 수 있다.  

```sql
SELECT cluster_name, cluster_type FROM user_clusters WHERE cluster_name = 'C1#';
```

&#42;&#95;CLU&#95;COLUMNS 뷰에서 클러스터 테이블과 클러스터 키에 대한 정보를 조회할 수 있다.  

```sql
SELECT cluster_name, clu_column_name, table_name, tab_column_name
FROM user_clu_columns
WHERE cluster_name = 'C1#';
```

&#42;&#95;CLU&#95;TABLES 뷰의 cluster&#95;name 열에서도 테이블이 저장된 클러스터를 확인할 수 있다.  

```sql
SELECT table_name, cluster_name FROM user_tables WHERE table_name IN ('T1', 'T2');
```

&#42;&#95;INDEXES 뷰에서 클러스터 인덱스에 대한 정보를 조회할 수 있다.  

```sql
SELECT index_type, table_owner, table_name, table_type
FROM user_indexes
WHERE index_name = 'C1#_X1';
```

클러스터 테이블은 동일한 클러스터에 속한 다른 테이블과 블록을 공유하기 때문에 개별 테이블을 TRUNCATE할 수 없다.  

```sql
TRUNCATE TABLE t1;
```

TRUNCATE CLUSTER 문을 수행하면 클러스터에 저장된 모든 테이블이 TRUNCATE된다.  

```sql
TRUNCATE CLUSTER c1#;
```

##### 20.1.2.4.2. 해시 클러스터
<br/>
해시 클러스터(hash cluster)는 데이터의 저장과 조회에 해시 함수를 사용한다.  

아래 쿼리는 해시 클러스터를 생성한다.  

```sql
DROP CLUSTER c1# INCLUDING TABLES;
CREATE CLUSTER c1# (c1 NUMBER) HASHKEYS 100;
```

아래 쿼리는 해시 클러스터 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;
DROP TABLE t2 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER) CLUSTER c1# (c1);
CREATE TABLE t2 (c1 NUMBER, c2 NUMBER) CLUSTER c1# (c1);
```

&#42;&#95;CLUSTERS 뷰에서 클러스터에 대한 정보를 조회할 수 있다.  

```sql
SELECT cluster_name, cluster_type FROM user_clusters WHERE cluster_name = 'C1#';
```

&#42;&#95;CLU&#95;COLUMNS 뷰에서 클러스터 테이블과 클러스터 키에 대한 정보를 조회할 수 있다.  

```sql
SELECT cluster_name, clu_column_name, table_name, tab_column_name
FROM user_clu_columns
WHERE cluster_name = 'C1#';
```

&#42;&#95;TABLES 뷰의 cluster&#95;name 열에서도 테이블이 저장된 클러스터를 확인할 수 있다.  

```sql
SELECT table_name, cluster_name FROM user_tables WHERE table_name IN ('T1', 'T2');
```

&#42;&#95;INDEXES 뷰에서 클러스터 인덱스에 대한 정보를 조회할 수 있다.  

```sql
SELECT index_type, table_owner, table_name, table_type
FROM user_indexes
WHERE index_name = 'C1#_X1';
```

클러스터 테이블은 동일한 클러스터에 속한 다른 테이블과 블록을 공유하기 때문에 개별 테이블을 TRUNCATE할 수 없다.  

```sql
TRUNCATE TABLE t1;
```

TRUNCATE CLUSTER 문을 수행하면 클러스터에 저장된 모든 테이블이 TRUNCATE된다.  

```sql
TRUNCATE CLUSTER c1#;
```

##### 20.1.2.4.3. 해시 클러스터
<br/>
해시 클러스터(hash cluster)는 데이터의 저장과 조회에 해시 함수를 사용한다.  

아래 쿼리는 해시 클러스터를 생성한다.  

```sql
DROP CLUSTER c1# INCLUDING TABLES;
CREATE CLUSTER c1# (c1 NUMBER) HASHKEYS 100;
```

아래 쿼리는 해시 클러스터 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;
DROP TABLE t2 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER) CLUSTER c1# (c1);
CREATE TABLE t2 (c1 NUMBER, c2 NUMBER) CLUSTER c1# (c1);
```

해시 클러스터는 &#42;&#95;CLUSTERS 뷰의 cluster&#95;type 열이 HASH로 표시된다.  

```sql
SELECT cluster_name, cluster_type, function, hashkeys
FROM user_clusters
WHERE cluster_name = 'C1#';
```

##### 20.1.2.4.4. 단일 테이블 해시 클러스터
<br/>
단일 테이블 해시 클러스터(single-table hash cluster)는 클러스터에 단일 테이블만 저장할 수 있는 해시 클러스터다. 일반 해시 클러스터보다 조회 성능이 뛰어나다.  

아래 쿼리는 단일 테이블 해시 클러스터를 생성한다.  

```sql
DROP CLUSTER c1# INCLUDING TABLES;
CREATE CLUSTER c1# (c1 NUMBER) SINGLE TABLE HASHKEYS 100;
```

아래 쿼리는 해시 클러스터 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER) CLUSTER c1# (c1);
```

단일 테이블 해시 클러스터는 &#42;&#95;CLUSTERS 뷰의 single&#95;table 열이 Y로 표시된다.  

```sql
SELECT cluster_name, single_table FROM user_clusters WHERE cluster_name = 'C1#';
```

##### 20.1.2.4.5. 정렬 해시 클러스터
<br/>
정렬 해시 클러스터(sorted hash cluster)는 SORT 키워드를 지정한 열로 데이터를 정렬하여 클러스터에 저장한다.  

아래 쿼리는 정렬 해시 클러스터를 생성한다. c2 열로 데이터를 정렬한다.  

```sql
DROP CLUSTER c1# INCLUDING TABLES;
CREATE CLUSTER c1# (c1 NUMBER, c2 NUMBER SORT) HASHKEYS 100 HASH IS c1;
```

아래 쿼리는 해시 클러스터 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, c3 NUMBER) CLUSTER c1# (c1, c2);
```

&#42;&#95;CLUSTER&#95;HASH&#95;EXPRESSIONS 뷰에서 해시 함수 표현식(HASH IS c1)에 대한 정보를 조회할 수 있다.  

```sql
SELECT hash_expression FROM user_cluster_hash_expressions WHERE cluster_name = 'C1#';
```

##### 20.1.2.5. 임시 테이블
<br/>
임시 테이블(temporary table)은 트랜잭션 또는 세션 레벨로 관리되는 테이블이다. OLAP 시스템이나 DW 시스템에서 중간 집계를 저장하는 용도로 사용한다. 임시 테이블이 아닌 테이블을 영구 테이블(parmanent table)이라고 한다.  

```
CREATE GLOBAL TEMPORARY TABLE [schema.]table (
  column datatype [DEFAULT [ON NULL] expr]
  [, column datatype [DEFAULT [ON NULL] expr]]...)
[ON COMMIT {DELETE | PRESERVE} ROWS];
```

+ ON COMMIT DELETE ROWS : 트랜잭션 레벨로 데이터를 저장
+ ON COMMIT PRESERVE ROWS : 세션 레벨로 데이터를 저장  

아래 쿼리는 트랜잭션 레벨의 임시 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;
CREATE GLOBAL TEMPORARY TABLE t1 (c1 NUMBER) ON COMMIT DELETE ROWS;
```

트랜잭션 레벨의 임시 테이블은 트랜잭션이 종료되면 테이블이 초기화된다.  

```sql
INSERT INTO t1 VALUES (1);

SELECT * FROM t1;

COMMIT;

SELECT * FROM t1;
```

아래 쿼리는 세션 레벨의 임시 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;
CREATE GLOBAL TEMPORARY TABLE t1 (c1 NUMBER) ON COMMIT PRESERVE ROWS;
```

세션 레벨의 임시 테이블은 세션이 종료되면 테이블이 초기화된다.  

```sql
INSERT INTO t1 VALUES (1);
COMMIT;

SELECT * FROM t1;

-- 다시 접속
SELECT * FROM t1;
```

임시 테이블은 트랜잭션 또는 세션 레벨로 관리되기 대문에 하나의 임시 테이이블을 다수의 세션에서 동시에 사용할 수 있다.  

### 20.2. 열
<br/>
테이블을 구성하는 기본 단위다. 테이블은 하나 이상의 열로 구성된다.  

#### 20.2.1. 기본 문법
<br/>
ALTER TABLE 문으로 열을 변경할 수 있다.  

##### 20.2.1.1. ADD 절
<br/>
ADD 절은 열을 추가한다.  

```
ALTER TABLE [schema.]table ADD (column datatype [DEFAULT [ON NULL] expr]
                             [, column datatype [DEFAULT [ON NULL] expr]]...);
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER);
```

아래 쿼리는 t1 테이블에 c2, c3 열을 추가한다.  

```sql
ALTER TABLE t1 ADD (c2 NUMBER(2), c3 VARCHAR2(2));
```

&#42;&#95;TAB&#95;COLUMNS 뷰에서 추가된 열을 확인할 수 있다.  

```sql
SELECT column_name, data_type, data_length, data_precision, data_scale
FROM user_tab_columns
WHERE table_name = 'T1';
```

##### 20.2.1.2. MODIFY 절
<br/>
MODIFY 절은 열을 수정한다.  

```
ALTER TABLE [schema.]table MODIFY (column datatype [DEFAULT [ON NULL] expr]
                                [, column datatype [DEFAULT [ON NULL] expr]]...);
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER(2), c2 NUMBER(2), c3 VARCHAR2(2), c4 DATE);

INSERT INTO t1 VALUES (1, 1, 'A', DATE '2050-01-01');
COMMIT;
```

아래 쿼리는 c1 열의 기본값을 1로 수정한다.  

```sql
ALTER TABLE t1 MODIFY (c1 DEFAULT 1);
```

NUMBER 타입은 열에 값이 존재하는 경우 데이터 타입의 길이를 축소할 수 없다.  

```sql
ALTER TABLE t1 MODIFY (c2 NUMBER(1));
```

VARCHAR2 타입은 열에 값이 존재하더라도 값의 실제 크기까지 데이터 타입의 길이를 축소할 수 있다.  

```sql
ALTER TABLE t1 MODIFY (c3 VARCHAR2(1));
```

데이터 타입의 길이를 늘리는 것은 값의 유무나 크기와 관계없이 가능하다.  

```sql
ALTER TABLE t1 MODIFY (c2 NUMBER(3), c3 VARCHAR2(3));
```

DATE 타입과 TIMESTAMP 타입은 상호 변경이 가능하지만 TIMESTAMP 타입을 DATE 타입으로 변경하면 소수점 이하 초가 유실된다.  

```sql
ALTER TABLE t1 MODIFY (c4 TIMESTAMP);
```

&#42;&#95;TAB&#95;COLUMNS 뷰에서 열이 수정된 내용을 확인할 수 있다.  

##### 20.2.1.3. RENAME 절
<br/>
RENAME 절은 열명을 변경한다.  

```
ALTER TABLE [schema.]table RENAME COLUMN old_name TO new_name;
```

아래 쿼리는 c4 열의 열명을 c5로 변경한다.  

```sql
ALTER TABLE t1 RENAME COLUMN c4 TO c5;
```

t1 테이블을 조회하면 열명이 변경된 것을 확인할 수 있다.  

##### 20.2.1.4. DROP 절
<br/>
DROP 절은 열을 삭제한다.  

```
ALTER TABLE [schema.]table DROP (column [, column]...)
[CASCADE CONSTRAINTS] [CHECKPOINT integer];
```

아래 쿼리는 c3, c5 열을 삭제한다.  

```sql
ALTER TABLE t1 DROP (c3, c5);
```

t1 테이블을 조회하면 열이 삭제된 것을 확인할 수 있다.  

##### 20.2.1.5. SET UNUSED 절
<br/>
SET UNUSED 절은 열을 미사용 열로 변경한다.  

```
ALTER TABLE [schema.]table SET UNUSED (column [, column]...) [CASCADE CONSTRAINTS] [ONLINE];
```

아래 쿼리는 c2 열을 미사용 열로 변경한다.  

```sql
ALTER TABLE t1 SET UNUSED (c2);
```

t1 테이블을 조회하면 미사용 열로 변경된 c2 열이 조회되지 않는 것을 확인할 수 있다.  

&#42;&#95;UNUSED&#95;COL&#95;TABS 뷰에서 미사용 열이 존재하는 테이블을 조회할 수 있다.  

```sql
SELECT * FROM user_unused_col_tabs;
```

&#42;&#95;TAB&#95;COLS 뷰를 조회해보자. 미사용 열로 지정한 c2 열이 SYS&#95;C% 형식의 이름을 가진 INVISIBLE 칼럼으로 변경된 것을 확인할 수 있다.  

```sql
SELECT column_name, hidden_column FROM user_tab_cols WHERE table_name = 'T1';
```

DROP 절로 미사용 열을 삭제할 수 있다.  

```
ALTER TABLE [schema.]table DROP {UNUSED COLUMNS | COLUMNS CONTINUE} [CHECKPOINT integer];
```

+ UNUSED COLUMNS : 테이블의 미사용 열을 모두 삭제
+ COLUMNS CONTINUE : 중단된 지점부터 미사용 열의 삭제 작업을 계속 진행
+ CHECKPOINT integer : 체크 포인트를 수행할 행의 개수를 지정 (기본값은 512)  

체크 포인트는 database buffer cache의 변경된 블록(dirty block)을 data file에 기록하기 위한 일련의 작업을 말한다. integer를 크게 설정하면 수행 시간을 단축할 수 있지만 언두 세그먼트 공간을 더 많이 소비한다.  

아래 쿼리는 t1 테이블의 미사용 열을 모두 삭제한다.  

```sql
ALTER TABLE t1 DROP UNUSED COLUMNS;
```

&#42;&#95;UNUSED&#95;COL&#95;TABS 뷰에서 미사용 열이 삭제된 것을 확인할 수 있다.  

##### 20.2.1.6. DDL 작업  
<br/>
열을 삭제하면 전체 행을 갱신해야 하기 때문에 DDL 문이 장시간 수행될 수 있다. DDL 문은 TM 락을 X 모드로 획득하기 때문에 장시간 수행될 경우 테이블 사용이 제한되어 장애가 발생할 수 있다. SET UNUSED 절로 삭제할 열을 미사용 열로 변경하고, 운영 외 시간에 DROP 절로 미사용 열을 삭제하는 방식을 사용해야 한다. 일정 규모 이상의 시스템은 정기 PM(Prevention Maintenance) 시점에 DDL 작업을 수행한다.  

#### 20.2.2. 데이터 타입
<br/>
오라클 데이터베이스는 다양한 데이터 타입을 제공한다. 주로 VARCHAR2, NUMBER, DATE, TIMESTAMP, INTERVAL, CLOB 타입을 사용한다. CHAR 타입과 LONG 타입은 사용하지 않는 편이 바람직하다. 공간 절약과 성능 개선을 위해 VARCHAR2(1) 대신 CHAR(!)을 사용해야 한다는 의견도 있다. 차이는 미미하다. VARCHAR2 타입을 사용하는 편이 여러모로 바람직하다.  

+ 문자 : CHAR, VARCAHR2, CLOB, LONG, NCHR, NVARCHAR2, NCLOB
+ 숫자 : NUMBER, BINARY&#95;FLOAT, BINARY&#95;DOUBLE
+ 날짜 : DATA, TIMESTAMP, INTERVAL
+ 이진 : BLOB, BFILE, LONG RAW, RAW
+ 기타 : ROWID, UROWID  

##### 20.2.2.1. VARCHAR2 타입
<br/>
VARCHAR2 타입은 가변 길이 문자 데이터 타입이다. size는 1 ~ 4000의 범위를 가진다. BYTE는 바이트, CHAR는 문자의 개수를 의미한다.  

```
VARCHAR2 (size [BYTE | CHAR])
```

먼저 V$NLS&#95;PARAMETERS 뷰에서 문자 타입과 관련된 파라미터를 확인하자.  

```sql
SELECT parameter, value
FROM v$nls_parameters
WHERE parameter IN ('NLS_CHARACTERSET', 'NLS_LENGTH_SEMANTICS');
```

아래 쿼리는 에러가 발생한다. 2바이트로 저장되는 한글을 크기가 1바이트인 c1 열에 입력했기 때문이다.  

```sql
INSERT INTO t1 (c1) VALUES ('가');
```

동일한 값을 크기가 2바이트인 c2 열에 입력하면 에러가 발생하지 않는다.  

```sql
INSERT INTO t1 (c2) VALUES ('가');
```

##### 20.2.2.2. NUMBER 타입
<br/>
NUMBER 타입은 가변 길이 숫자 데이터 타입이다. 길이에 따라 1 ~ 22 바이트의 저장 공간을 사용한다.  

```
NUMBER [(p [, s])]
```

+ p : 정수부 (범위는 1 ~ 38)
+ s : 소수부 (범위는 -84 ~ 127, 기본값은 0)  

NUMBER 타입은 지정한 크기에 따라 아래의 값을 저장할 수 있다. 정수부는 p ~ s로 계산된다. 주로 강조한 크기 형식으로 사용한다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER(3, 2));
```

크기를 지정하지 않은 c1 열은 data&#95;precision 열과 data_scale 열이 널로 표시된다.  

```sql
SELECT column_name, data_type, data_length, data_precision, data_scale
FROM user_tab_columns
WHERE table_name = 'T1';
```

데이터 타입의 크기는 범위 무결성과 관련이 있다. 아래 쿼리는 데이터 타입의 크기보다 큰 값을 입력했기 때문에 에러가 발생한다.  

VSIZE 함수로 저정된 값이 크기를 조회할 수 있다. c1 열은 21바이트, c2 열은 3바이트를 사용한다. 저장 공간의 불필요한 낭비를 막기 위해 NUMBER 타입의 크기를 지정하는 편이 바람직하다.  

```sql
SELECT VSIZE(c1) AS c1, VSIZE(c2) AS c2 FROM t1;
```

##### 20.2.2.3. DATE 타입
<br/>
DATE 타입은 고정 길이 날짜 데이터 타입이다. B.C 4712년 1월 1일 ~ A.D 9999년 12월 31일의 범위에 있는 일자와 초 단위 시간을 저장할 수 있다. 저장 공간은 7바이트를 사용한다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 DATE, c2 DATE);
```

&#42;&#95;TAB&#95;COLUMNS 뷰의 data&#95;length 열에서 7바이트가 사용되는 것을 확인할 수 있다.  

```sql
SELECT column_name, data_type, data_length
FROM user_tab_columns
WHERE table_name = 'T1';
```

##### 20.2.2.4. TIMESTAMP 타입
<br/>
TIMESTAMP 타입은 가변 길이 날짜 데이터 타입이다. 소수점 이하 초 단위의 날짜 값을 저장할 수 있다. fractional&#95;seconds&#95;precision의 범위는 0 ~ 9, 기본값은 6이다. 정밀도에 따라 7 ~ 11 바이트의 저장 공간을 사용한다.  

```
TIMESTAMP [(fractional_seconds_precision)]
```

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 TIMESTAMP);
```

&#42;&#95;TAB&#95;COLUMNS 뷰의 data&#95;length 열에서 11바이트가 사용되는 것을 확인할 수 있다.  

```sql
SELECT column_name, data_type, data_length
FROM user_tab_columns
WHERE table_name = 'T1';
```

##### 20.2.2.5. INTERVAL 타입
<br/>
INTERVAL 타입은 날짜 값의 기간을 저장한다. INTERVAL YEAR TO MONTH 타입과 INTERVAL DAY TO SECOND 타입을 사용할 수 있다.  

INTERVAL YEAR TO MONTH 타입은 연, 월 단위의 기간을 저장한다. year&#95;precision의 범위는 0 ~ 9, 기본값은 2다. 5바이트의 저장 공간을 사용한다.  

```
INTERVAL YEAR [(year_precision)] TO MONTH
```

INTERVAL DAY TO SECOND 타입은 일, 시, 분, 초 단위의 기간을 저장한다. day&#95;precision의 범위는 0 ~ 9, 기본값은 2다. fractional&#95;seconds&#95;precision의 범위는 0 ~ 9, 기본값은 6이다. 11바이트의 저장 공간을 사용한다.  

```
INTERVAL DAY [(day_precision)] TO SECOND [(fractional_seconds_precision)]
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 INTERVAL YEAR TO MONTH, c2 INTERVAL DAY TO SECOND);

INSERT INTO t1 VALUES (INTERVAL '1-11' YEAR TO MONTH
                     , INTERVAL '1 12:34:56.789' DAY TO SECOND);
COMMIT;
```

&#42;&#95;TAB&#95;COLUMNS 뷰의 data&#95;type 열에서도 데이터 타입의 크기를 확인할 수 있다.  

```sql
SELECT column_name, data_type, data_length, data_precision, data_scale
FROM user_tab_columns
WHERE table_name = 'T1';
```

INTERVAL 타입은 주로 날짜 값 연산을 위한 기간 값을 저장하기 위해 사용된다.  

```sql
SELECT c1, c2
     , TIMESTAMP '2050-01-01 00:00:00' + c1 AS c3
     , TIMESTAMP '2050-01-01 00:00:00' + c2 AS c4
FROM t1;
```

##### 20.2.2.6. CLOB 타입
<br/>
CLOB 타입은 가변 길이 문자 타입이다. CLOB은 Character Large OBject의 약자다. 최대 (4GB - 1byte) &#42; block size 크기의 문자열을 저장할 수 있다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 CLOB);

INSERT INTO t1 VALUES (TO_CLOB(LPAD('A', 4000, 'A')) || LPAD('B', 4000, 'B'));
COMMIT;
```

&#42;&#95;LOBS 뷰에서 LOB에 대한 정보를 조회할 수 있다. CLOB 타입은 별도의 LOB 세그먼트에 저장되며, LOB 세그먼트를 검색하기 위해 LOB 인덱스가 생성된다.  

```sql
SELECT column_name, segment_name, index_name, FROM user_lobs WHERE table_name = 'T1';
```

&#42;&#95;SEGMENTS 뷰에서 LOB 세그먼트와 LOB 인덱스가 생성된 것을 확인할 수 있다.  

```sql
SELECT segment_name, segment_type
FROM user_segments
WHERE segment_name IN ('SYS_LOB...', 'SYSLOB...');
```

&#42;&#95;INDEXES 뷰에서 LOB 인덱스를 조회할 수 있다. index&#95;type 열이 LOB으로 표시된다.  

```sql
SELECT index_type FROM user_indexes WHERE index_name = 'SYS_IL...'
```

CLOB 타입은 값의 크기가 4000바이트 이하면 테이블 세그먼트에 값을 저장한다. CLOB 타입에 DISABLE STORAGE IN ROW 옵션을 지정하면 4000바이트 이하의 값도 별도의 세그먼트에 저장할 수 있다. 기본값은 ENABLE STORAGE IN ROW다.

```sql
DROP TABLE t2 PURGE;

CREATE TABLE t2 (c1 CLOB, c2 CLOB);
LOB(c1) STORE AS (ENABLE STORAGE IN ROW);
LOB(c2) STORE AS (DISABLE STORAGE IN ROW);
```

아래 쿼리는 에러가 발생한다. CLOB 값은 비교 조건을 사용할 수 없다.  

```sql
SELECT * FROM t1 WHERE c1 = 'A';
```

LIKE 조건은 에러가 발생하지 않는다. 오라클 데이터베이스는 LOB 타입을 지원하기 위해 DBMS&#95;LOB 패키지를 제공한다.  

###### 20.2.2.6.1. EMPTY&#95;CLOB 함수
<br/>
Java에서 CLOB 타입을 사용하는 경우 EMPTY_CLOB 함수로 초기화하는 것이 일반적이다.  

테스트를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 CLOB);

INSERT INTO t1 VALUES (1, NULL);
INSERT INTO t1 VALUES (2, EMPTY_CLOB());
INSERT INTO t1 VALUES (3, 'X');
COMMIT;
```

아래는 t1 테이블을 조회한 결과다. c1이 1, 2인 행의 c2 값이 없는 것으로 표시된다.  

CLOB 타입으로 생성된 열은 아래의 세 가지 상태를 가질 수 있다.  

+ null : 로케이터(locator)와 값(value)이 없음
+ empty : 로케이터는 존재하지만 값이 없음
+ populated : 로케이터와 값이 존재함  

IS NULL 조건으로 조회하면 c1이 1인 행(상태가 null)이 반환된다. EMPTY&#95;CLOB 함수로 초기화된 값은 반환되지 않는다.  

```sql
SELECT * FROM t1 WHERE c2 IS NULL;
```

IS NOT NULL 조건으로 조회하면 c1이 2, 3인 행(상태가 empty나 populated)이 반환된다.  

```sql
SELECT * FROM t1 WHERE c2 IS NOT NULL;
```

EMPTY&#95;CLOB 함수로 초기화된 행(상태가 empty)을 조회하기 위해서는 아래와 같이 LENGTH 함수를 사용하면 된다.  

```sql
SELECT * FROM t1 WHERE LENGTH(c2) = 0;
```

상태가 null이나 empty인 행을 조회하기 위해서는 아래와 같이 NULLIF 함수를 사용하면 된다.  

```sql
SELECT * FROM t1 WHERE NULLIF(LENGTH(c2), 0) IS NULL;
```

##### 20.2.2.7. CHAR 타입
<br/>
CHAR 타입은 고정 길이 문자 데이터 타입이다. size의 범위는 1 ~ 2000, 기본값은 1이다.  

```
CHAR [(size [BYTE | CHAR])]
```

예제를 위해 아래와 같이 테이블을 생성하자. CHAR 타입은 고정 길이로 저장되기 때문에 문자열이 데이터 타입의 크기보다 작은 경우 문자열의 뒤쪽을 공백으로 채워서 저장한다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 CHAR(2), c2 CHAR(3));

INSERT INTO t1 VALUES ('A', 'A');
INSERT INTO t1 VALUES ('A', 'B ');
COMMIT;
```

아래 쿼리에서 문자열 'B'와 'B '는 동일한 값으로 평가된다. 데이터 타이브이 크기가 다른 CHAR 타입은 크기가 작은 열의 뒤쪽을 공백으로 채워 동일한 크기로 만든 후 문자열을 비교한다.  

```sql
SELECT c1, c2, c2 || 'Z' AS c3 FROM t1 WHERE c1 = c2;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 CHAR(2), c2 VARCHAR2(3));

INSERT INTO t1 VALUES ('A', 'A');
INSERT INTO t1 VALUES ('A', 'B ');
COMMIT;
```

아래 쿼리에서 문자열 'A'와 'A'는 다른 값, 'B'와 'B '는 동일한 값으로 평가된다. c1 열의 뒤쪽이 공백으로 채워지기 때문이다.  

```sql
SELECT c1, c2 FROM t1 WHERE c1 = c2;
```

TRIM 함수를 사용해야 정확한 결과를 얻을 수 있다. CHAR 타입과 VARCHAR 타입을 함께 사용하면 아래와 같은 불필요한 처리가 필요하다.  

```sql
SELECT c1, c2 FROM t1 WHERE TRIM(c1) = c2;
```

##### 20.2.2.8. LONG 타입
<br/>
LONG 타입은 가변 길이 문자 타입이다. 최대 2GB나 2³¹ - 1 바이트 크기의 문자열을 저장할 수 있다. LONG 타입은 이전 버전과의 호환성을 위해 유지되고 있다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 LONG);

INSERT INTO t1 VALUES('A');
COMMIT;
```

WHERE 절에 LONG 타입을 사용하면 에러가 발생한다.  

```sql
SELECT * FROM t1 WHERE c1 = 'A';
```

CREATE TABLE 문에 TO&#95;LOB 함수를 사용하면 LONG 타입을 CLOB 타입으로 변환할 수 있다. TO&#95;LOB 함수는 CREATE TABLE AS SELECT 문(CTAS)과 INSERT AS SELECT 문에만 사용할 수 있다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 AS SELECT TO_LOB(c1) AS c1 FROM t1;
```

#### 20.2.3. 기본값
<br/>
열은 기본값을 지정할 수 있다. 아래의 두 가지 유형을 사용할 수 있다. DEFAULT ON NULL는 12.1 버전부터 사용할 수 있다.  

+ DEFAULT : 지정되지 않거나 DEFAULT 키워드가 기술된 경우 expr을 입력
+ DEFAULT ON NULL : 지정되지 않거나 DEFAULT 키워드가 널이 기술된 경우 expr을 입력

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER DEFAULT 2, c3 NUMBER DEFAULT ON NULL 3);

INSERT INTO t1 (c1) VALUES (1);
INSERT INTO t1 VALUES (2, DEFAULT, DEFAULT);
INSERT INTO t1 VALUES (3, NULL, NULL);
COMMIT;
```

아래 쿼리는 c2 열에 널이 존재하기 때문에 에러가 발생한다.  

```sql
ALTER TABLE t1 MODIFY c2 DEFAULT 1 NOT NULL;
```

아래 쿼리를 수행해보자.  

```sql
ALTER TABLE t1 MODIFY c3 DEFAULT 1;
ALTER TABLE t1 ADD (c4 NUMBER DEFAULT 1 NOT NULL);
ALTER TABLE t1 ADD (c5 NUMBER DEFAULT 1 NULL);
```

t1 테이블을 조회하면 기존의 열에 기본값을 지정하면 이후 입력된 행에만 기본값이 적용되고, 추가한 열은 이전 행도 기본값이 적용되는 것을 확인할 수 있다.  

+ Metadata-Only DEFAULT  
11.1 이전 버전까지 테이블에 기본값이 지정된 열을 추가하면 기존 행을 갱신해야 하기 때문에 DDL 문이 장시간 수행되어 장애가 발생할 수 있었다. 장애를 방지하기 위해 11.1 버전부터 기본값이 지정된 NOT NULL 칼럼을 추가하면 메타데이터(metadata)만 반영하고 기존 행을 갱신하지 않도록 기능이 개선되었다. 12.1 버전부터는 NULLABLE 칼럼도 메타데이터만 반영하도록 기능이 개선되었다.  

#### 20.2.4. 열 유형
<br/>
오라클 데이터베이스는 특별한 유형의 열을 제공한다.  

##### 20.2.4.1. VIRTUAL 칼럼
<br/>
VIRTUAL 칼럼은 물리적으로 저장되지 않는 열이다. 11.1 버전부터 사용할 수 있다.  

```
column [datatype] [GENERATED ALWAYS] AS (column_expression) [VIRTUAL]
```

아래 쿼리는 c3 열을 VIRTUAL 칼럼으로 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (
  c1 NUMBER
  , c2 NUMBER
  , c3 NUMBER GENERATED ALWAYS AS (c1 + c2) VIRTUAL);

INSERT INTO t1 (c1, c2) VALUES (1, 2);
COMMIT;
```

아래 쿼리에서 c3 열은 c1 + c2 값인 3을 반환한다.  

```sql
SELECT * FROM t1;
```

VIRTUAL 칼럼에 데이터를 입력하면 에러가 발생한다.  

```sql
INSERT INTO t1(c3) VALUES(3);
```

&#42;&#95;TAB&#95;COLS 뷰에서 VIRTUAL 칼럼에 대한 정보를 조회할 수 있다. VIRTUAL 칼럼은 물리적으로 저장되지 않기 때문에 segment&#95;column&#95;id 열이 널이다.  

```sql
SELECT column_name, data_type, column_id, data_default, virtual_column, segment_column_id
FROM user_tab_cols
WHERE table_name = 'T1'
ORDER BY internal_column_id;
```

##### 20.2.4.2. INVISIBLE 칼럼
<br/>
INVISIBLE 칼럼은 숨겨진 열이다. 12.1 버전부터 사용할 수 있다.  

```
column [datatype] [VISIBLE | INVISIBLE]
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER INVISIBLE, c2 NUMBER);
```

INSERT 문에 열 목록을 지정하지 않으면 VISIBLE 칼럼에만 데이터가 입력된다.  

```sql
INSERT INTO t1 VALUES (1);
INSERT INTO t1(c1, c2) VALUES(2, 2);
COMMIT;
```

아래에서 애스터리스크(&#42;)로 조회한 좌측 쿼리는 c2 열만 조회된다. 우측 쿼리처럼 c1 열을 직접 기술하면 INVISIBLE 칼럼을 조회할 수 있다.  

```sql
SELECT * FROM t1;
SELECT c1, c2 FROM t1;
```

&#42;&#95;TAB&#95;COLS 뷰에서 INVISIBLE 칼럼에 대한 정보를 조회할 수 있다. column&#95;id 열은 널, hidden&#95;column 열은 YES로 표시된다.  

```sql
SELECT column_name, column_id, segment_column_id, internal_column_id, hidden_column
FROM user_tab_cols
WHERE table_name = 'T1'
ORDER BY internal_column_id;
```

c1 열을 VISIBLE 칼럼으로 수정해보자.  

```sql
ALTER TABLE t1 MODIFY c1 VISIBLE;
```

&#42;&#95;TAB&#95;COLS 뷰에서 c1 열이 VISIBLE 칼럼으로 변경된 것을 확인할 수 있다. column&#95;id 값이 마지막 column&#95;id 값에 이어서 부여된다.  

```sql
SELECT column_name, column_id, segment_column_id, internal_column_id, hidden_column
FROM user_tab_cols
WHERE table_name = 'T1'
ORDER BY internal_column_id;
```

t1 테이블을 조회해보면 열 순서가 변경된 것을 확인할 수 있다. 오라클 데이터베이스는 column&#95;id 값의 순서대로 열을 반환한다.  

c2 열을 INVISIBLE 칼럼으로 변경한 후 다시 VISIBLE 칼럼으로 변경해보자.  

```sql
ALTER TABLE t1 MODIFY c1 INVISIBLE;
ALTER TABLE t1 MODIFY c1 VISIBLE;
```

&#42;&#95;TAB&#95;COLS 뷰의 column&#95;id 열이 변경된 것을 확인할 수 있다.  

```sql
SELECT column_name, column_id, internal_column_id
FROM user_tab_cols
WHERE table_name = 'T1'
ORDER BY internal_column_id;
```

##### 20.2.4.3. 열 순서 변경
<br/>
INVISIBLE 칼럼에 의해 column&#95;id 값이 변경되는 동작을 응용하면 열 순서를 임의로 변경할 수 있다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c3 NUMBER, c2 NUMBER, c1 NUMBER);

INSERT INTO t1 VALUES(3, 2, 1);
COMMIT;
```

아래와 같이 열 순서를 변경할 수 있는 프로시저를 생성하자.  

```sql
CREATE OR REPLACE prc_chg_col (
  i_owner IN VARCHAR2
  , i_table_name IN VARCHAR2
  , i_column_name IN VARCHAR2
  , i_column_id IN VARCHAR2
)
IS
  l_sql_text VARCHAR2(4000) := 'ALTER TABLE ' || i_owner || '.' || i_table_name || ' MODIFY ';
BEGIN
  EXECUTE IMMEDIATE l_sql_text || i_column_name || ' INVISIBLE';
  EXECUTE IMMEDIATE l_sql_text || i_column_name || ' VISIBLE';

  FOR c1 IN (SELECT column_name
             FROM all_tab_columns
             WHERE owner = i_table_name
               AND table_name <> i_table_name
               AND column_name <> i_column_name
               AND column_id >= i_column_id
             ORDER BY column_id)
  LOOP
    EXECUTE IMMEDIATE l_sql_text || c1.i_column_name || ' INVISIBLE';
    EXECUTE IMMEDIATE l_sql_text || c1.i_column_name || ' VISIBLE';
  END LOOP;
END prc_chg_col;
/
```

아래와 같이 프로시저를 수행하면 c1 열은 첫 번째, c2 열은 두 번째로 열 순서가 변경된다.  

```sql
EXEC prc_chg_col ('SCOTT', 'T1', 'C1', 1);

EXEC prc_chg_col ('SCOTT', 'T1', 'C2', 2);
```

### 20.3. 제약 조건
<br/>
제약 조건(constraint)은 데이터 무결성을 보장할 수 있는 기능이다. 테이블에 다수의 제약 조건을 생성할 수 있다.  

#### 20.3.1. 기본 문법
<br/>
제약 조건은 inline 또는 out of line 방식으로 생성할 수 있다. inline 방식은 열 레벨, out of line 방식은 테이블 레벨에 제약 조건을 기술한다.  

아래 쿼리는 inline 방식으로 NOT NULL 제약 조건을 생성한다. NOT NULL 제약 조건은 inline 방식으로만 생성할 수 있다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER CONSTRAINT t1_n1 NOT NULL);
```

아래 쿼리는 out of line 방식으로 CHECK 제약 조건을 생성한다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, CONSTRAINT t1_c1 CHECK(c1 > c2));
```

아래 쿼리는 out of line 방식을 사용한 ALTER TABLE 문이다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER);

ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY (c1);
```

##### 20.3.1.1. ADD 절
<br/>
ADD 절은 제약 조건을 추가한다.  

```
ALTER TABLE [schema.]table ADD {out_of_line_constraint}...;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER NOT NULL, c2 NUMBER);
```

아래 쿼리는 t1 테이블에 t1&#95;pk 제약 조건을 추가한다.  

```sql
ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY (c1);
```

##### 20.3.1.2. MODIFY 절
<br/>
MODIFY 절은 제약 조건을 변경한다.  

```
ALTER TABLE [schema.]table MODIFY CONSTRAINT constraint_name
  [INITALLY {IMMEDIATE | DEFERRED}]
  [ENABLE | DISABLE] [VALIDATE | NOVALIDATE]
  [CASCADE];
```

##### 20.3.1.3. RENAME 절
<br/>
RENAME 절은 제약 조건 명을 변경한다.  

```
ALTER TABLE [schema.]table RENAME CONSTRAINT old_name TO new_name;
```

아래 쿼리는 t1&#95;pk 제약 조건 명을 t1&#95;u1으로 변경한다.  

```sql
ALTER TABLE t1 RENAME CONSTRAINT t1_pk TO t1_u1;
```

##### 20.3.1.4. DROP 절
<br/>
DROP 절은 제약 조건을 삭제한다.  

```
ALTER TABLE [schema.]table DROP CONSTRAINT constraint_name
  [CASCADE]
  [{KEEP | DROP} INDEX]
  [ONLINE];
```

+ CASCADE : 참조하고 있는 FK 제약 조건을 함께 삭제
+ KEEP INDEX : 제약 조건을 삭제할 때 제약 조건이 사용하고 있는 인덱스를 유지
+ DROP INDEX : 제약 조건을 삭제할 때 제약 조건이 사용하고 있는 인덱스를 삭제

#### 20.3.2. 제약 조건 유형
<br/>
##### 20.3.2.1. NOT NULL 제약 조건
<br/>
NOT NULL 제약 조건은 지정한 열에 널이 존재하지 않는 것을 보장한다.  

```
ALTER TABLE [schema.]table MODIFY column [CONSTRAINT constraint_name] [NOT] NULL;
```

예제를 위해 아래와 같이 테이블을 생성하자. c1 열에 NOT NULL 제약 조건을 생성했다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER CONSTRAINT t1_n1 NOT NULL, c2 NUMBER, c3 NUMBER);
```

아래 쿼리는 c1 열에 널이 입력되어 에러가 발생한다.  

```sql
INSERT INTO t1(c1) VALUES(1);
```

아래와 같이 c2 열에 NOT NULL 제약 조건, c3 열에 기본값을 지정한 NOT NULL 제약 조건을 생성하자.  

```sql
ALTER TABLE t1 MODIFY (c2 NOT NULL, c3 DEFAULT 0 NOT NULL);
```

기본값을 지정한 c3 열은 값을 입력하지 않아도 에러가 발생하지 않는다.  

```sql
INSERT INTO t1(c1, c2) VALUES(1, 1);
```

아래 쿼리는 c3 열의 NOT NULL 제약 조건을 삭제한다.  

```sql
ALTER TABLE t1 MODIFY c3 NULL;
```

&#42;&#95;TAB&#95;COLUMNS 뷰의 nullable 열에서 NOT NULL 여부를 확인할 수 있다.  

```sql
SELECT column_name, nullable FROM user_tab_columns WHERE table_name = 'T1';
```

&#42;&#95;CONSTRAINTS 뷰에서 제약 조건에 대한 정보를 조회할 수 있다. c2 열처럼 제약 조건 명을 지정하지 않으면 제약 조건 명이 자동으로 부여된다.  

```sql
SELECT constraint_name, constraint_type, search_condition_vc, generated
FROM user_constraints
WHERE table_name = 'T1'
ORDER BY search_condition_vc;
```

&#42;&#95;CONSTRAINTS 뷰의 constraint&#95;type 열의 값은 아래와 같다.  

+ C : NOT NULL 제약 조건, CHECK 제약 조건
+ U : UNIQUE 제약 조건
+ P : PK 제약 조건
+ R : FK 제약 조건
+ V : 뷰에 대한 WITH CHECK OPTION
+ O : 뷰에 대한 WITH READ ONLY  

##### 20.3.2.2. UNIQUE 제약 조건
<br/>
UNIQUE 제약 조건은 열이나 열의 조합이 고유한 것을 보장한다.  

```
ALTER TABLE [schema.]table ADD CONSTRAINT constraint_name UNIQUE (column [, column]...);
```

예제를 위해 아래와 같이 테이블을 생성하자. c1, c2 열의 조합에 UNIQUE 제약 조건을 생성했다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, CONSTRAINT t1_u1 UNIQUE (c1, c2));
```

아래와 같이 데이터를 입력해보자. 첫 번째, 두 번째 쿼리는 동일하게 널을 입력했지만 에러가 발생하지 않는다. UNIQUE 제약 조건의 전체 열에 널이 입려되면 중복을 검사하지 않는다. 마지막 쿼리는 중복 값을 입력했기 때문에 에러가 발생했다.  

```sql
INSERT INTO t1 VALUES(NULL, NULL);
INSERT INTO t1 VALUES(NULL, NULL);
INSERT INTO t1 VALUES(1, NULL);
INSERT INTO t1 VALUES(1, NULL);
```

&#42;&#95;CONSTRAINTS 뷰에서 UNIQUE 제약 조건이 t1&#95;u1 인덱스를 사용하고 있는 것을 확인할 수 있다.  

```sql
SELECT constraint_name, constraint_type, index_owner, index_name
FROM user_constraints
WHERE table_name = 'T1'
ORDER BY constraint_name;
```

&#42;&#95;CONS&#95;COLUMNS 뷰에서 제약 조건의 열에 대한 정보를 조회할 수 있다.  

```sql
SELECT constraint_name, column_name, position
FROM user_cons_columns
WHERE table_name = 'T1'
ORDER BY constraint_name, position;
```

##### 20.3.2.3. PK 제약 조건
<br/>
PK 제약 조건은 열이나 열의 조합으로 행을 고유하게 식별할 수 있는 것을 보장한다. NOT NULL 제약 조건과 UNIQUE 제약 조건이 합쳐진 제약 조건으로 볼 수 있다. PK 제약 조건은 테이블에 하나만 생성할 수 있다.  

```
ALTER TABLE [schema.]table ADD CONSTRAINT constraint_name PRIMARY KEY (column [, column]...);
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);
```

아래 쿼리는 t1 테이블에 PK 제약 조건을 추가한다. c1 열로 PK 제약 조건을 생성했다.  

아래와 같이 데이터를 입력해보자. 두 번째 쿼리처럼 중복 값을 입력하거나, 세 번째 쿼리처럼 널을 입력하면 에러가 발생한다.  

```sql
INSERT INTO t1 VALUES(1, 1);
INSERT INTO t1 VALUES(1, 2);

INSERT INTO t1 VALUES(NULL, 3);
```

&#42;&#95;CONSTRAINTS 뷰에서 PK 제약 조건이 t1&#95;pk 인덱스를 사용하고 있는 것을 확인할 수 있다.  

```sql
SELECT constraint_name, constraint_type, index_owner, index_name
FROM user_constraints
WHERE table_name = 'T1'
```

PK 제약 조건의 열은 NOT NULL 제약 조건이 없더라도 nullable 열이 N으로 표시된다.  

```sql
SELECT column_name, nullable FROM user_tab_columns WHERE table_name = 'T1';
```

아래와 같이 PK 제약 조건을 삭제해보자.  

```sql
ALTER TABLE t1 DROP CONSTRAINT t1_pk;
```

&#42;&#95;TAB&#95;COLUMNS 뷰를 다시 조회해보면 c1 열의 nullable 열이 Y로 변경된 것을 확인할 수 있다.  

```sql
SELECT column_name, nullable FROM user_tab_columns WHERE table_name = 'T1';
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1(c1 NUMBER NOT NULL, c2 NUMBER, CONSTRAINT t1_pk PRIMARY KEY(c1));
```

&#42;&#95;CONSTRAINTS 뷰에서 NOT NULL 제약 조건이 생성된 것을 확인할 수 있다.  

```sql
SELECT constraint_name, constraint_type, index_owner, index_name
FROM user_constraints
WHERE table_name = 'T1'
```

아래와 같이 PK 제약 조건을 삭제해보자.  

```sql
ALTER TABLE t1 DROP CONSTRAINT t1_pk;
```

&#42;&#95;TAB&#95;COLUMNS 뷰를 다시 조회해보면 PK 제약 조건의 존재 여부와 관계 없이 c1 열의 nullable 열이 N로 표시된 것을 확인할 수 있다.  

```sql
SELECT column_name, nullable FROM user_tab_columns WHERE table_name = 'T1';
```

###### 20.3.2.4.1. PK 제약 조건과 NOT NULL 제약 조건
<br/>
PK 제약 조건의 열은 NOT NULL 제약 조건을 명시적으로 생성해야 한다.  

예제를 위해 아래와 같이 테이블을 생성하자. c1 열에만 NOT NULL 제약 조건을 지정했다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER NOT NULL, c2 NUMBER, c3 VARCHAR2(4000));

INSERT INTO t1
SELECT ROWNUM AS c1, ROWNUM AS c2, LPAD('X', 4000, 'X') AS c3
FROM XMLTABLE('1 to 100000');

COMMIT;

CREATE UNIQUE INDEX t1_u1 ON t1(c1);
CREATE UNIQUE INDEX t1_u2 ON t1(c2);
```

쿼리의 수행 시간을 측정하기 위해 SET 명령어로 TIMING 시스템 변수를 ON으로 설정하자.  

```sql
SET TIMING ON
```

c1 열로 PK 제약 조건을 생성하면 0.01초만에 제약 조건이 생성된다. NOT NULL 제약 조건으로 c1 열에 널이 존재하지 않는 것이 보장되고, t1&#95;u1 인덱스로 값이 고유한 것이 보장되기 때문에 별도로 데이터를 검증할 필요가 없다.  

```sql
ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY(c1);
```

c2 열로 PK 제약 조건을 생성하면 9.30초가 소요된다. c2 열에 NOT NULL 제약 조건이 없기 때문에 널 존재 여부를 검증하기 위해 전체 테이블을 읽어야 한다.  

```sql
ALTER TABLE t1 DROP CONSTRAINT t1_pk;
ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY (c2);
```

아래 쿼리로 NOT NULL 제약 조건이 생성되지 않은 PK 제약 조건의 열을 조회할 수 있다.  

```sql
SELECT a.owner, a.table_name, a.column_name
FROM dba_tab_columns a
WHERE a.nullable = 'N'
AND NOT EXISTS (SELECT 'X'
                FROM dba_constraints x
                WHERE x.owner = a.owner
                AND x.table_name = a.table_name
                AND x.search_condition_vc = '"' || a.column_name || '" IS NOT NULL');
```

###### 20.3.2.4.2. 중복 값 삭제
<br/>
중복 값이 존재하지 않아야 PK 제약 조건을 생성할 수 있다. 중복 값이 존재한다면 중복 값을 삭제해야 한다.  

예제를 위해 아래와 같이 테이블을 생성하자. c1 열에 중복 값이 존재한다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);

INSERT INTO t1 VALUES(1, 1);
INSERT INTO t1 VALUES(2, 2);
INSERT INTO t1 VALUES(2, 3);
INSERT INTO t1 VALUES(3, 4);
INSERT INTO t1 VALUES(3, 5);
INSERT INTO t1 VALUES(3, 6);
COMMIT;
```

c1 열로 PK 제약 조건을 생성하면 에러가 발생한다.  

```sql
ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY(c1);
```

아래 쿼리로 중복 값을 삭제할 수 있다. ORDER BY 절로 유지할 행의 우선순위를 지정할 수 있다.  

```sql
DELETE
FROM t1
WHERE ROWID IN(SELECT rid
               FROM (SELECT ROWID AS rid
                          , ROW_NUMBER() OVER(PARTITION BY c1 ORDER BY) AS rn
                     FROM t1)
               WHERE rn > 1);
```

t1 테이블을 조회햐면 중복 값이 삭제된 것을 확인할 수 있다.  

c1 열에 중복 값이 존재하지 않기 때문에 PK 제약 조건이 정상적으로 생성된다.  

```sql
ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY(c1);
```

##### 20.3.2.4. FK 제약 조건
<br/>
FK 제약 조건은 부모 테이블과 자식 테이블의 참조 무결성을 보장한다.  

```
ALTER TABLE [schema.]table ADD CONSTRAINT constraint_name
  FOREIGN KEY (column [, column]...)
  REFERENCES [schema.]object [(column [, column]...)
  [ON DELETE {CASCADE | SET NULL}];
```

FK 제약 조건은 부모 테이블의 행이 갱신 또는 삭제될 때 수행할 규칙을 지정할 수 있다. 오라클 데이터베이스는 삭제에 대한 NO ACTION, CASCADE, SET NULL 규칙만 지정할 수 있다. 기본값은 NO ACTION이다. 나머지 규칙은 트리거로 구현해야 한다.  

+ NO ACTION : 에러가 발생함
+ CASCADE : 자식 테이블의 행을 갱신 또는 삭제함
+ SET NULL : 자식 테이블의 행의 값을 널로 설정함
+ SET DEFAULT : 자식 테이블의 행의 값을 기본값으로 설정함
+ RESTRICT : 참조하는 행을 갱신 또는 삭제할 수 없음  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 CASCADE CONSTRAINTS PURGE;
DROP TABLE t2 CASCADE CONSTRAINTS PURGE;
DROP TABLE t3 CASCADE CONSTRAINTS PURGE;
DROP TABLE t4 CASCADE CONSTRAINTS PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, CONSTRAINTS t1_pk PRIMARY KEY (c1));
CREATE TABLE t2 (c1 NUMBER);
CREATE TABLE t3 (c1 NUMBER);
CREATE TABLE t4 (c1 NUMBER);
```

아래 쿼리는 에러가 발생한다. FK 제약 조건에서 참조하는 부모 테이블의 열은 UNIQUE 제약 조건이나 PK 제약 조건이 생성되어 있어야 한다.  

```sql
ALTER TABLE t2 ADD CONSTRAINT t2_f1 FOREIGN KEY (c1) REFERENCES t1 (c2);
```

아래와 같이 FK 제약 조건을 생성하자.  

```sql
ALTER TABLE t2 ADD CONSTRAINT t2_f1 FOREIGN KEY (c1) REFERENCES t1 (c1);
ALTER TABLE t3 ADD CONSTRAINT t3_f1 FOREIGN KEY (c1) REFERENCES t1 (c1) ON DELETE CASCADE;
ALTER TABLE t4 ADD CONSTRAINT t4_f1 FOREIGN KEY (c1) REFERENCES t1 (c1) ON DELETE SET NULL;
```

아래 쿼리는 에러가 발생한다. 부모 테이블에 존재하지 않는 값을 자식 테이블에 입력했기 때문이다.  

```sql
INSERT INTO t2 VALUES (1);
```

부모 테이블인 t1 테이블에 아래와 같이 데이터를 입력하자.  

```sql
INSERT INTO t1 VALUES (1, 1);
```

INSERT 문을 다시 수행하면 에러가 발생하지 않는다.  

```sql
INSERT INTO t2 VALUES (1);
```

자식 테이블이 참조하고 있는 부모 테이블의 값은 갱신이 불가능하다.  

```sql
UPDATE t1 SET c1 = 2 WHERE c1 = 1;
```

삭제 역시 불가능하다.  

```sql
DELETE FROM t1 WHERE c1 = 1;
```

아래와 같이 데이터를 입력해보자. t2 테이블에는 데이터를 입력하지 않았다.  

```sql
INSERT INTO t1 VALUES (2, 2);
INSERT INTO t3 VALUES (2);
INSERT INTO t4 VALUES (2);
```

t1 테이블의 c1이 2인 행을 삭제해보자. 에러가 발생하지 않는다.  

```sql
DELETE FROM t1 WHERE c1 = 2;
```

t3 테이블은 FK 제약 조건이 ON DELETE CASCADE 규칙으로 생성되었기 때문에 부모 테이블의 삭제된 행이 함께 삭제된다. t4 테이블은 FK 제약 조건이 ON DELETE SET NULL 규칙으로 생성되었기 때문에 값이 널로 갱신된다.  

&#42;&#95;CONSTRAINTS 뷰에서 참조하고 있는 제약 조건과 DELETE 규칙을 확인할 수 있다.  

```sql
SELECT constraint_name, constraint_type, r_owner, r_constraint_name, delete_rule
FROM user_constraints
WHERE table_name IN ('T2', 'T3', 'T4')
ORDER BY constraint_name;
```

자식 테이블에서 참조하는 부모 테이블의 열을 삭제하면 에러가 발생한다.  

```sql
ALTER TABLE t1 DROP COLUMN c1;
```

ALTER TABLE 문에 CASCADE CONSTRAINTS 절을 지정하면 삭제할 열을 참조하고 있는 자식 테이블의 FK 제약 조건을 함께 삭제할 수 있다.  

```sql
ALTER TABLE t1 DROP COLUMN c1 CASCADE CONSTRAINTS;
```

자식 테이블에서 참조하는 부모 테이블 역시 삭제할 수 없다.  

```sql
DROP TABLE t1 PURGE;
```

DROP TABLE 문에 CASCADE CONSTRAINTS를 지정하면 삭제할 테이블을 참조하는 자식 테이블의 FK 제약 조건을 함께 삭제할 수 있다.  

```sql
DROP TABLE t1 CASCADE CONSTRAINTS PURGE;
```

다음 예제를 위해 아래와 같이 테이블을 삭제하자.  

```sql
DROP TABLE t1 CASCADE CONSTRAINTS PURGE;
DROP TABLE t2 CASCADE CONSTRAINTS PURGE;
DROP TABLE t3 CASCADE CONSTRAINTS PURGE;
DROP TABLE t4 CASCADE CONSTRAINTS PURGE;
```

###### 20.3.2.4.1. FK 제약 조건과 인덱스
<br/>
FK 제약 조건을 생성한 열은 반드시 인덱스를 생성해야 한다. 인덱스를 생성하지 않으면 블로킹으로 인한 장애가 발생할 수 있다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 CASCADE CONSTRAINTS PURGE;
DROP TABLE t2 CASCADE CONSTRAINTS PURGE;

CREATE TABLE t1 (c1 NUMBER, CONSTRAINT t1_pk PRIMARY KEY (c1));
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER
  , CONSTRAINT t1_pk PRIMARY KEY (c1)
  , CONSTRAINT t1_f1 FOREIGN KEY (c2) REFERENCES t1 (c1));

INSERT INTO t1 VALUES (1);
INSERT INTO t1 VALUES (2);
INSERT INTO t1 VALUES (3);
INSERT INTO t2 VALUES (1, 1);
COMMIT;
```

아래 예제는 S1 세션에서 자식 테이블의 값(1)을 갱신한 후, S2 세션에서 부모 테이블의 값(3)을 갱신한다. 서로 다른 값을 갱신해도 블로킹이 발생하는 것을 확인할 수 있다.  

```sql
--- S1
UPDATE t2 SET c2 = 2 WHERE c1 = 1;

--- S2
UPDATE t1 SET c1 = 4 WHERE c1 = 3;
--- 블로킹

--- S1
ROLLBACK;

--- S2
ROLLBACK;
```

아래와 같이 인덱스를 생성하자.  

```sql
CREATE INDEX t2_x1 ON t2 (c2);
```

동일한 예제를 다시 수행하면 블로킹이 발생하지 않는 것을 확인할 수 있다.  

```sql
--- S1
UPDATE t2 SET c2 = 2 WHERE c1 = 1;

--- S2
UPDATE t1 SET c1 = 4 WHERE c1 = 3;

--- S1
ROLLBACK;

--- S2
ROLLBACK;
```

##### 20.3.2.5. CHECK 제약 조건
<br/>
CHECK 제약 조건은 열에 저장된 값이 condition을 만족하는 것을 보장한다.  

```
ALTER TABLE [schema.]table ADD CONSTRAINT constraint_name CHECK (condition);
```

예제를 위해 아래와 같이 테이블을 생성하자. c1 열에 CHECK 제약 조건을 생성했다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, CONSTRAINT t1_c1 CHECK (c1 < c2));
```

아래와 같이 데이터를 입력해보자. 두 번째 쿼리는 CHECK 제약 조건의 결과가 UNKNOWN이므로 에러가 발생하지 않는다. CHECK 제약 조건의 결과가 FALSE인 세 번째 쿼리는 에러가 발생한다.  

```sql
INSERT INTO t1 VALUES (1, 2);
INSERT INTO t1 VALUES (2, NULL);
INSERT INTO t1 VALUES (3, 2);
```

예제를 위해 아래와 같이 테이블을 생성하자. c1 열을 VARCHAR2(8) 타입으로 생성했다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 VARCHAR2(8));
```

아래 쿼리는 c1 열에 유효한 날짜만 입력될 수 있는 CHECK 제약 조건을 생성한다.  

```sql
ALTER TABLE t1 ADD CONSTRAINT t1_c1
                   CHECK (VALIDATE_CONVERSION (c1 AS DATE, 'YYYYMMDD') = 1);
```

아래와 같이 데이터를 입력해보자. 두 번째 쿼리는 20500132가 유효한 날짜가 아니기 때문에 에러가 발생한다.  

```sql
INSERT INTO t1 VALUES ('20500131');
INSERT INTO t1 VALUES ('20500132');
```

&#42;&#95;CONSTRAINTS 뷰의 search&#95;condition&#95;vc 열에서 CHECK 제약 조건이 condition을 확인할 수 있다.  

```sql
SELECT constraint_name, constraint_type, search_condition_vc
FROM user_constraints
WHERE table_name = 'T1';
```

#### 20.3.3. 제약 조건 상태
<br/>
제약 조건은 필요에 따라 지연 상태와 활성 상태를 설정할 수 있다.  

##### 20.3.3.1. 제약 조건 지연
<br/>
MODIFY CONSTRAINT 절로 제약 조건의 지연 상태를 설정할 수 있다.  

```
ALTER TABLE [schema.]table MODIFY CONSTRAINT constraint_name
  [[[NOT] DEFERRABLE] [INITALLY {IMMEDIATE | DEFERRED}];
```

아래의 세 가지 조합이 가능하다.  

<table>
  <thead>
    <tr>
      <td>DEFERRABLE</td>
      <td>DEFERRED</td>
      <td>설명</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>NOT DEFERRABLE</td>
      <td>IMMEDIATE</td>
      <td>제약 조건을 지연시키지 않음</td>
    </tr>
    <tr>
      <td>DEFERRABLE</td>
      <td>IMMEDIATE</td>
      <td>구문이 완료될 때까지 제약 조건을 지연</td>
    </tr>
    <tr>
      <td>DEFERRABLE</td>
      <td>DEFERRED</td>
      <td>트랜잭션이 완료될 때까지 제약 조건을 지연</td>
    </tr>
  </tbody>
</table>
<br/><br/>

예제를 위해 아래와 같이 테이블을 생성하자. c1 열에 NOT NULL 제약 조건을 생성했다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER CONSTRAINT t1_n1 NOT NULL);
```

&#42;&#95;CONSTRAINTS 뷰에서 NOT DEFERRABLE IMMEDIATE 상태가 기본값인 것을 확인할 수 있다.  

```sql
SELECT deferrable, deferred FROM user_constraints WHERE table_name = 'T1';
```

아래와 같이 t1&#95;n1 제약 조건을 삭제하고 DEFERRABLE 제약 조건을 생성하자.  

```sql
ALTER TABLE t1 DROP CONSTRAINT t1_n1;
ALTER TABLE t1 MODIFY c1 CONSTRAINT t1_n1 NOT NULL DEFERRABLE;
```

제약 조건이 DEFERRABLE IMMEDIATE 상태로 생성되는 것을 확인할 수 있다.  

```sql
SELECT deferrable, deferred FROM user_constraints WHERE table_name = 'T1';
```

제약 조건이 DEFERRABLE IMMEDIATE 상태면 INSERT 문이 완료될 때 에러가 발생한다.  

```sql
INSERT INTO t1 VALUES (NULL);
```

제약 조건을 DEFERRABLE DEFERRED 상태로 변경해보자.  

```sql
ALTER TABLE t1 MODIFY CONSTRAINT t1_n1 INITALLY DEFERRED;
```

제약 조건이 DEFERRABLE DEFERRED 상태면 커밋이 수행될 때 에러가 발생한다.  

```sql
INSERT INTO t1 VALUES (NULL);
COMMIT;
```

아래처럼 먼저 값을 입력하고 후속 처리로 값을 갱신하는 방식에 활용할 수 있다.  

```sql
INSERT INTO t1 VALUES (NULL);
UPDATE t1 SET c1 = 1 WHERE c1 IS NULL;
COMMIT;
```

###### 20.3.3.1.1. SET CONSTRAINT 문
<br/>
SET CONSTRAINT 문은 DEFERRABLE 제약 조건의 상태를 설정한다.  

```
SET {CONSTRAINT | CONSTRAINTS} {constraint [, constraint]... | ALL} {IMMEDIATE | DEFERRED};
```

아래 쿼리는 DEFERRABLE DEFERRED 제약 조건의 상태를 IMMEDIATE로 설정한다. 제약 조건은 DEFERRED 상태지만 IMMEDIATE 방식으로 동작한다.  

```sql
SET CONSTRAINT ALL IMMEDIATE;

INSERT INTO t1 VALUES (NULL);
```

아래와 같이 t1&#95;n1 제약 조건을 DEFERRABLE IMMEDIATE 상태로 변경하자.  

```sql
ALTER TABLE t1 MODIFY CONSTRAINT t1_n1 DEFERRABLE IMMEDIATE;
```

아래 쿼리는 DEFERRABLE IMMEDIATE 제약 조건을 DEFERRED 상태로 설정한다. 제약 조건은 IMMEDIATE 상태지만 DEFERRED 방식으로 동작한다. 값을 입력하고 후속 처리로 값을 갱신하기 위해 제약 조건을 DEFERRABLE IMMEDIATE 상태로 생성하고 필요 시 제약 조건의 상태를 DEFERRABLE DEFERRED 상태로 설정하는 방식을 사용한다.  

```sql
SET CONSTRAINT ALL DEFERRED;

INSERT INTO t1 VALUES (NULL);

COMMIT;
```

##### 20.3.3.2. 제약 조건 비활성화
<br/>
MODIFY CONSTRAINT 절로 제약 조건의 활성화 상태를 설정할 수 있다.  

```
ALTER TABLE [schema.]table MODIFY CONSTRAINT constraint_name
  [ENABLE | DISABLE] [VALIDATE | NOVALIDATE];
```

아래의 네 가지 조합이 가능하다.  

<table>
  <thead>
    <tr>
      <td>STATUS</td>
      <td>VALIDATED</td>
      <td>설명</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ENABLE</td>
      <td>VALIDATE</td>
      <td>제약 조건이 활성화되어 있고 데이터도 검증됨</td>
    </tr>
    <tr>
      <td>DISABLE</td>
      <td>NOVALIDATE</td>
      <td>제약 조건이 비활성화되어 있고 데이터도 검증되지 않음</td>
    </tr>
    <tr>
      <td>ENABLE</td>
      <td>NOVALIDATE</td>
      <td>제약 조건이 활성화되어 있지만 데이터는 검증되지 않음</td>
    </tr>
    <tr>
      <td>DISABLE</td>
      <td>VALIDATE</td>
      <td>제약 조건이 비활성화되어 있지만 데이터는 검증됨</td>
    </tr>
  </tbody>
</table>
<br/><br/>

예제를 위해 아래와 같이 테이블을 생성하자. c1 열에 NOT NULL 제약 조건을 생성했다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER CONSTRAINT t1_n1 NOT NULL);
```

&#42;&#95;CONSTRAINTS 뷰에서 ENABLED VALIDATED 상태가 기본값인 것을 확인할 수 있다.  

```sql
SELECT status, validated FROM user_constraints WHERE table_name = 'T1';
```

아래와 같이 데이터를 입력해보자. c2 열에 널을 입력하면 에러가 발생한다.  

```sql
INSERT INTO t1 VALUES (1, 1);
INSERT INTO t1 VALUES (2, NULL);
```

아래 쿼리는 제약 조건을 비활성화한다.  

```sql
ALTER TABLE t1 MODIFY CONSTRAINT t1_n1 DISABLE;
```

제약 조건을 비활성화하면서 제약 조건이 DISABLE NOVALIDATE 상태로 변경된다. 성능상의 이유로 FK 제약 조건의 상태를 DISABLED NOVALIDATE로 생성하고 메타데이터를 통해 참조 무결성을 직접 검증하기도 한다.  

```sql
SELECT status, validated FROM user_constraints WHERE table_name = 'T1';
```

제약 조건이 DISABLED NOVALIDATED 상태면 제약 조건을 위배한 데이터를 입력해도 에러가 발생하지 않는다.  

```sql
INSERT INTO t1 VALUES (2, NULL);
```

제약 조건을 활성화하면 검증 과정에서 에러가 발생한다.  

```sql
ALTER TABLE t1 MODIFY CONSTRAINT t1_n1 ENABLE;
```

아래 쿼리는 제약 조건을 ENABLE NOVALIDATE 상태로 변경한다.  

```sql
ALTER TABLE t1 MODIFY CONSTRAINT t1_n1 ENABLE NOVALIDATE;
```

ENABLE NOVALIDATE 상태는 제약 조건이 검증되지 않은 상태다.  

```sql
SELECT * FROM t1;
```

제약 조건은 활성화되어 있기 때문에 제약 조건을 위반한 데이터를 입력하면 에러가 발생한다.  

```sql
INSERT INTO t1 VALUES (3, NULL);
```

제약 조건을 위배한 데이터를 삭제해보자.  

```sql
DELETE FROM t1 WHERE c2 IS NULL;
```

제약 조건을 활성화해도 에러가 발생하지 않는다.  

```sql
ALTER TABLE t1 MODIFY CONSTRAINT t1_n1 ENABLE;
```

아래와 같이 제약 조건을 DISABLE VALIDATE 상태로 변경해보자.  

```sql
ALTER TABLE t1 MODIFY CONSTRAINT t1_n1 DISABLE VALIDATE;
```

제약 조건이 DISABLE VALIDATE 상태면 테이블에 대한 DML 작업이 불가능하다.  

```sql
INSERT INTO t1 VALUES (4, 4);
```

### 20.4. 인덱스
<br/>
인덱스(index)는 데이터를 빠르게 검색하기 위한 오브젝트다. 인덱스는 테이블에 종속되어 있으며 하나의 테이블에 다수의 인덱스를 생성할 수 있다.  

오라클 데이터베이스는 다양한 종류의 인덱스를 제공한다.  

+ 일반 인덱스 : b-tree 구조에 정렬된 열 값과 ROWID를 저장하는 인덱스
+ 비트맵 인덱스 : 비트맵을 사용하여 열 값을 저장하는 인덱스
+ 함수 기반 인덱스 : 열의 일부에 함수나 표현식을 사용한 인덱스
+ 도메인 인덱스 : 애플리케이션 도메인에 따라 사용자가 생성할 수 있는 인덱스  

아래와 같은 일반 인덱스를 사용할 수 있다.  

+ IOT : 인덱스 구조 테이블(index-organized table)
+ 리버스 키 인덱스 : 열 값이 반전(reverse)되어 저장된 인덱스
+ 내림차순 인덱스 : 열의 일부가 역순으로 정렬된 인덱스
+ 클러스터 인덱스 : 인덱스 클러스터의 검색을 위해 사용되는 인덱스  

인덱스는 열의 개수에 따라 아래와 같이 구분된다.  

+ 단일 인덱스(single) : 1개의 열로 구성된 인덱스
+ 복합 인덱스(composite) : 2개 이상의 열로 구성된 인덱스  

값의 고유함에 따라 아래와 같이 구분할 수도 있다.  

+ UNIQUE 인덱스 : 고유한 값으로 구성된 인덱스
+ NONUNIQUE 인덱스 : 고유하지 않을 수 있는 값으로 구성된 인덱스  

#### 20.4.1. 기본 문법
<br/>
##### 20.4.1.1. CREATE INDEX 문
<br/>
CREATE INDEX 문은 인덱스를 생성한다. 아래는 일반 인덱스를 생성하는 구문이다.  

```
CREATE [UNIQUE] INDEX [schema.]index ON [schema.]table [t_alias]
  ({column | column_expression} [ASC | DESC]
[, {column | column_expression} [ASC | DESC]]...)
   [USABLE | UNUSABLE] [{ONLINE | REVERSE}] [TABLESPACE tablespace];
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, c3 NUMBER);
```

아래 쿼리는 인덱스를 생성한다. t1&#95;u1 인덱스는 단일 UNIQUE 인덱스, t1&#95;x1 인덱스는 복합 NONUNIQUE 인덱스다.  

```sql
CREATE UNIQUE INDEX t1_u1 ON t1 (c1);

CREATE INDEX t1_x1 ON t1 (c2, c3);
```

&#42;&#95;INDEXES 뷰에서 인덱스에 대한 정보를 조회할 수 있다. 일반 인덱스는 index&#95;type 열이 NORMAL로 표시된다.  

```sql
SELECT index_name, index_type, uniqueness FROM user_indexes WHERE table_name = 'T1';
```

&#42;&#95;IND&#95;COLUMNS 뷰에서 인덱스 열에 대한 정보를 조회할 수 있다.  

```sql
SELECT index_name, column_name, column_position, descend
FROM user_ind_columns
WHERE table_name = 'T1'
ORDER BY index_name, column_position;
```

##### 20.4.1.2. ALTER INDEX 문
<br/>
ALTER INDEX 문은 인덱스를 변경한다.  

###### 20.4.1.2.1. UNUSABLE 절
<br/>
UNUSABLE 절은 인덱스의 상태를 UNUSABLE 상태로 변경한다. UNUSABLE 상태의 인덱스는 사용이 불가능하며 DML 작업이 수행되어도 갱신되지 않는다.  

```
ALTER INDEX [schema.]index UNUSABLE [ONLINE];
```

아래 쿼리는 t1&#95;x1 인덱스를 UNUSABLE 상태로 변경한다.  

```sql
ALTER INDEX t1_x1 UNUSABLE;
```

&#42;&#95;INDEXES 뷰에서 t1&#95;x1 인덱스의 status 값이 UNUSABLE로 변경된 것을 확인할 수 있다.  

```sql
SELECT index_name, status FROM user_indexes WHERE table_name = 'T1';
```

아래와 같이 데이터를 입력해보자. 에러가 발생하지 않는다.  

```sql
INSERT INTO t1 VALUES (1, 1, 1);
```

t1&#95;u1 인덱스를 UNUSABLE 상태로 변경해보자.  

```sql
ALTER TABLE t1_u1 UNUSABLE;
```

데이터를 입력하면 에러가 발생한다. UNIQUE 인덱스가 비활성화되면 테이블에 데이터를 입력할 수 없다.  

###### 20.4.1.2.2. REBUILD 절
<br/>
REBUILD 절은 인덱스를 재구축한다. 단편화 해소나 테이블스페이스 변경을 위해 사용한다.  

```
ALTER TABLE [schema.]index REBUILD [ONLINE] [TABLESPACE tablespace];
```

아래 쿼리는 t1&#95;u1 인덱스와 t1&#95;x1 인덱스를 재구축한다.  

```sql
ALTER INDEX t1_u1 REBUILD;

ALTER INDEX t1_x1 REBUILD;
```

&#42;&#95;INDEXES 뷰의 status 값이 VALID로 변경된 것을 확인할 수 있다.  

```sql
SELECT index_name, status FROM user_indexes WHERE table_name = 'T1';
```

###### 20.4.1.2.3. 인덱스와 DML 작업
<br/>
인덱스는 DML 작업의 성능을 저하시킨다. 대량 데이터를 입력해야 할 경우 인덱스를 UNUSABLE 상태로 변경하고, 데이터를 입력한 후, 다시 인덱스를 REBUILD하는 방식을 사용하기도 한다. UNIQUE 인덱스를 UNUSABLE 상태로 변경하면 데이터를 입력할 수 없기 때문에 삭제 후 재생성해야 한다. 이런 이유로 DW 시스템에서는 NONUNIQUE 인덱스로 PK 제약 조건을 생성하기도 한다.  

###### 20.4.1.2.4. RENAME 절
<br/>
RENAME 절은 인덱스 명을 변경한다.  

```
ALTER TABLE [schema.]index RENAME TO new_name;
```

아래 쿼리는 t1&#95;x1 인덱스의 인덱스 명을 t1&#95;x2로 변경한다.  

```sql
ALTER TABLE t1_x1 RENAME TO t1_x2;
```

&#42;&#95;INDEXES 뷰에서 인덱스 명이 변경된 것을 확인할 수 있다.  

```sql
SELECT index_name FROM user_indexes WHERE table_name = 'T1';
```

##### 20.4.1.3. DROP INDEX 문
<br/>
DROP INDEX 문은 인덱스를 삭제한다.  

```
DROP INDEX [schema.]index [ONLINE];
```

아래 쿼리는 t1&#95;x2 인덱스를 삭제한다.  

```sql
DROP INDEX t1_x2;
```

&#42;&#95;INDEXES 뷰에서 인덱스가 삭제된 것을 확인할 수 있다.  

```sql
SELECT index_name FROM user_indexes WHERE table_name = 'T1';
```

#### 20.4.2. 인덱스 유형
<br/>
##### 20.4.2.1. 비트맵 인덱스
<br/>
비트맵 인덱스(bitmap index)는 비트맵을 사용하여 인덱스 열 값을 저장하는 인덱스다. 주로 DW 시스템에서 사용되며, OLTP 시스템에서는 사용하지 않는 것이 일반적이다. 비트맵 인덱스는 압축된 형태로 저장된다. 하나의 값이 다수의 행과 연결되어 있기 때문에 락으로 인한 경합이 발생할 수 있다.  

아래 쿼리는 비트맵 인덱스를 생성한다. t1&#95;x1 인덱스는 비트맵 인덱스, t1&#95;x2 인덱스는 비트맵 조인 인덱스(bitmap join index)로 생성했다.  

```sql
DROP TABLE t1 PURGE;
DROP TABLE t2 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, CONSTRAINT t1_pk PRIMARY KEY (c1));
CREATE TABLE t2 (c1 NUMBER, c2 NUMBER, CONSTRAINT t2_pk PRIMARY KEY (c1));

CREATE BITMAP INDEX t1_x1 ON t1 (c2);
CREATE BITMAP INDEX t1_x2 ON t1 (t2.c2) FROM t1, t2 WHERE t2.c1 = t1.c1;
```

비트맵 인덱스는 index&#95;type이 BITMAP, 비트맵 조인 인덱스는 join&#95;index 열이 YES로 표시된다.  

```sql
SELECT index_name, index_type, uniqueness, join_index
FROM user_indexes
WHERE table_name = 'T1';
```

&#42;&#95;JOIN&#95;IND&#95;COLUMNS 뷰에서 비트맵 조인 인덱스의 조인 열을 조회할 수 있다.  

```sql
SELECT index_name, index_table_name, inner_table_column, outer_table_name, outer_table_column
FROM user_join_ind_columns;
```

IOT에 비트맵 인덱스를 생성하면 에러가 발생한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, CONSTRAINT t1_pk PRIMARY KEY (c1)) ORGANIZATION INDEX;
```

아래와 같이 IOT 매핑 테이블을 생성해야 비트맵 인덱스를 생성할 수 있다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, CONSTRAINT t1_pk PRIMARY KEY (c1)) ORGANIZATION INDEX MAPPING TABLE;

CREATE BITMAP INDEX t1_x1 ON t1 (c2);
```

&#42;&#95;OBJECTS 뷰에서 IOT 매핑 테이블을 확인할 수 있다.  

```sql
SELECT b.object_name, b.object_id, b.object_type
FROM user_objects a, user_objects b
WHERE a.object_name = 'T1'
AND b.object_name = 'SYS_IOT_MAP_' || a.object_id;
```

##### 20.4.2.2. 함수 기반 인덱스
<br/>
함수 기반 인덱스(function-based index)는 인덱스 열의 일부에 함수나 표현식을 사용한 인덱스다. 일반 인덱스나 비트맵 인덱스로 생성할 수 있다. 함수 기반 인덱스를 줄여서 FBI라고 부르기도 한다.  

아래 쿼리는 FBI를 생성한다. t1&#95;x1 인덱스는 일반 FBI, t1&#95;x2 인덱스는 비트맵 FBI로 생성했다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);

CREATE INDEX t1_x1 ON t1 (c1 + c2);
CREATE BITMAP INDEX t1_x2 ON t1 (TRUNC (c1));
```

&#42;&#95;INDEXES 뷰의 index&#95;type 열이 일반 FBI는 FUNCTION-BASED NORMAL, 비트맵 FBI는 FUNCTION-BASED BITMAP으로 표시된다.  

```sql
SELECT index_name, index_type
FROM user_indexes
WHERE table_name = 'T1';
```

&#42;&#95;IND&#95;EXPRESSIONS 뷰에서 FBI의 표현식을 조회할 수 있다.  

```sql
SELECT * FROM user_ind_expressions WHERE table_name = 'T1';
```

&#42;&#95;IND&#95;COLUMNS 뷰에서 t1&#95;x1 인덱스는 SYS&#95;NC00003$ 열, t1&#95;x2 인덱스는 SYS&#95;NC00004$ 열을 사용하고 있는 것을 확인할 수 있다.  

```sql
SELECT index_name, column_name FROM user_ind_columns WHERE table_name = 'T1';
```

FBI는 내부적으로 INVISIBLE VIRTUAL 칼럼을 생성하고 해당 열로 인덱스를 생성한다.  

```sql
SELECT column_name, data_default, hidden_column, virtual_column, user_generated
FROM user_tab_cols
WHERE table_name = 'T1';
```

##### 20.4.2.3. 내림차순 인덱스
<br/>
내림차순 인덱스(descending index)는 인덱스 열의 일부를 내림차순으로 정렬한 인덱스다. 일반 인덱스는 열을 오름차순으로 정렬한다.  

아래 쿼리는 내림차순 인덱스를 생성한다. c1 열은 오름차순, c2 열은 내림차순으로 정렬된다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);

CREATE TABLE t1_x1 ON t1 (c1, c2 DESC);
```

내림차순 인덱스는 FBI로 생성된다.  

```sql
SELECT index_name, index_type FROM user_indexes WHERE table_name = 'T1';
```

##### 20.4.2.4. 리버스 키 인덱스
<br/>
리버스 키 인덱스(reverse key index)는 인덱스 열의 바이트를 반전시킨 일반 인덱스다. 순차 값 입력에 의한 경합을 해소하기 위해 사용할 수 있다. 일반 인덱스의 선두 열에 순차 값을 입력하면 b-tree 우측에만 값이 입력되기 때문에 블록 경합이 발생할 수 있다. 이런 현상이 발생하는 인덱스를 right growing 또는 right hand 인덱스라고 한다.  

아래 쿼리는 리버스 키 인덱스를 생성한다. t1&#95;x1 인덱스는 일반 리버스 키 인덱스, t1&#95;x2 인덱스는 일반 함수 기반 리버스 키 인덱스다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);

CREATE INDEX t1_x1 ON t1 (c1) REVERSE;
CREATE INDEX t1_x2 ON t1 (c1 + c2) REVERSE;
```

&#42;&#95;INDEXES 뷰의 index&#95;type 열이 일반 리버스 키 인덱스는 NORMAL/REV, 일반 함수 기반 리버스 키 인덱스는 FUNCTION-BASED NORMAL/REV로 표시된다.  

```sql
SELECT index_name, index_type FROM user_indexes WHERE table_name = 'T1';
```

#### 20.4.3. 인덱스와 제약 조건
<br/>
PK 제약 조건과 UNIQUE 제약 조건은 내부적으로 인덱스를 사용한다. 인덱스를 사용하지 않으면 제약 조건을 검증할 때마다 전체 테이블을 읽어야 하기 때문이다.  

##### 20.4.3.1. 자동 생성
<br/>
PK 제약 조건과 UNIQUE 제약 조건은 생성 시 사용할 수 있는 인덱스가 없으면 자동으로 인덱스를 생성한다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (
  c1 NUMBER PRIMARY KEY
, c2 NUMBER CONSTRAINT t1_u1 UNIQUE
, c3 NUMBER UNIQUE DEFERRABLE
, c4 NUMBER CONSTRAINT t1_u2 UNIQUE DEFERRABLE
);
```

&#42;&#95;INDEXES 뷰에서 자동으로 생성된 인덱스를 확인할 수 있다. 제약 조건 명을 지정하지 않으면 SYS&#95;C로 시작하는 제약 조건과 인덱스가 생성된다. DEFERRABLE 제약 조건은 NONUNIQUE 인덱스만 사용할 수 있다.  

```sql
SELECT index_name, uniqueness FROM user_indexes WHERE table_name = 'T1';
```

아래와 같이 제약 조건을 비활성화해보자. KEEP INDEX 절이나 DROP INDEX 절을 지정하지 않으면 NOT DEFERRABLE 제약 조건이 사용하는 인덱스는 삭제되고, DEFERRABLE 제약 조건이 사용하는 인덱스는 유지된다.  

```sql
ALTER TALBE t1 MODIFY CONSTRAINT SYS_C0000001 DISABLE;
ALTER TALBE t1 MODIFY CONSTRAINT t1_u1 DISABLE KEEP INDEX;
ALTER TALBE t1 MODIFY CONSTRAINT SYS_C0000002 DISABLE;
ALTER TALBE t1 MODIFY CONSTRAINT t1_u2 DISABLE DROP INDEX;
```

&#42;&#95;INDEXES 뷰에서 SYS&#95;C0000001, t1&#95;u2 인덱스가 삭제된 것을 확인할 수 있다.  

```sql
SELECT index_name, uniqueness FROM user_indexes WHERE table_name = 'T1';
```

##### 20.4.3.2. 수동 생성
<br/>
PK 제약 조건과 UNIQUE 제약 조건은 생성 시 사용할 수 있는 인덱스가 있으면 해당 인덱스를 사용한다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER NOT NULL, c2 NUMBER NOT NULL, c3 NUMBER, c4 NUMBER);

CREATE UNIQUE INDEX t1_pk ON t1 (c1, c2);
CREATE INDEX t1_x1 ON t1 (c3, c4);
```

USING INDEX 절을 사용하면 제약 조건이 사용할 인덱스를 명시적으로 지정할 수 있다.  

```sql
ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY (c1, c2) USING INDEX t1_pk;
```

존재하지 않는 인덱스를 지정하면 에러가 발생한다.  

```sql
ALTER TABLE t1 ADD CONSTRAINT t1_u1 UNIQUE (c3, c4) USING INDEX t1_u1;
```

&#42;&#95;CONSTRAINTS 뷰에서 t1&#95;pk 제약 조건이 t1&#95;pk 인덱스를 사용하고 있는 것을 확인할 수 있다.  

```sql
SELECT index_name FROM user_constraints WHERE constraint_name = 'T1_PK';
```

아래와 같이 t1&#95;pk 제약 조건을 비활성화해보자.  

```sql
ALTER TABLE t1 MODIFY CONSTRAINT t1_pk DISABLE;
```

수동으로 생성된 인덱스는 제약 조건이 비활성화되어도 삭제되지 않는다. 인덱스를 생성한 후 제약 조건을 생성하는 편이 관리 측면에서 바람직하다.  

```sql
SELECT index_name FROM user_indexes WHERE table_name = 'T1';
```

아래와 가티 USING INDEX 절을 기술하지 않고 t1_u1 제약 조건을 생성해보자.  

```sql
ALTER TABLE t1 ADD CONSTRAINT t1_u1 UNIQUE (c4, c3);
```

&#42;&#95;CONSTRAINTS 뷰에서 t1&#95;u1 제약 조건이 t1&#95;x1 인덱스를 사용하고 있는 것을 확인할 수 있다. UNIQUE 제약 조건이 NONUNIQUE 인덱스를 사용하고 있는 것이다. 열 순서가 다르더라도 제약 조건을 검증할 수 있는 인덱스라면 사용이 가능하다.  

```sql
SELECT index_name FROM user_constraints WHERE constraint_name = 'T1_U1';
```

아래와 같이 테이블과 인덱스를 생성한 후 제약 조건을 추가해보자. t1&#95;x1 인덱스를 t1&#95;pk 인덱스보다 먼저 생성했다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER NOT NULL, c2 NUMBER, c3 NUMBER);

CREATE INDEX t1_x1 ON t1 (c1, c2);
CREATE UNIQUE INDEX t1_pk ON t1 (c1);

ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY (c1);
```

&#42;&#95;CONSTRAINTS 뷰에서 t1&#95;pk 제약 조건이 t1&#95;x1 인덱스를 사용한 것을 확인할 수 있다. 제약 조건을 검증할 수 있는 인덱스가 다수라면 먼저 생성된 인덱스를 사용한다. 이런 동작 방식은 성능 저하의 원인이 될 수도 있다. USING INDEX 절을 사용하여 인덱스를 지정하는 편이 바람직하다.  

```sql
SELECT index_name FROM user_constraints WHERE constraint_name = 'T1_PK';
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER NOT NULL, c2 NUMBER);

CREATE UNIQUE INDEX t1_pk ON t1 (c1);

ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY (c1);
ALTER TABLE t1 ADD CONSTRAINT t1_u1 UNIQUE (c2);
```

SYS.OBJ$ 테이블의 spare1 값이 6이면 수동으로 생성된 인덱스, 0이면 자동으로 생성된 인덱스다.  

```sql
SELECT name, spare1 FROM sys.obj$ WHERE name LIKE 'T1_%';
```

##### 20.4.3.3. 테이블 명 변경
<br/>
테이블 전체 데이터를 재생성해야 할 때가 있다. 이런 경우 지속적인 서비스를 위해 신규 테이블에 데이터를 생성하고, 기존 테이블 명을 다른 이름으로 변경한 후, 신규 테이블을 기존 테이블 명으로 변경하는 방식을 사용하기도 한다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
DROP TABLE t2 PURGE;
DROP TABLE t3 PURGE;

CREATE TABLE t1 (c1 NUMBER);
CREATE TABLE t2 (c1 NUMBER);
CREATE UNIQUE INDEX t1_pk ON t1 (c1);
CREATE UNIQUE INDEX t2_pk ON t2 (c1);
ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY (c1) USING INDEX t1_pk;
ALTER TABLE t2 ADD CONSTRAINT t2_pk PRIMARY KEY (c1) USING INDEX t2_pk;
```

아래 쿼리는 t1 테이블의 제약 조건 명, 테이블 명, 인덱스 명을 변경한 후, t2 테이블을 t1 테이블로 변경한다.  

```sql
ALTER TABLE t1 RENAME CONSTRAINT t1_pk TO t3_pk;
ALTER TABLE t1 RENAME TO t3;
ALTER INDEX t1_pk RENAME TO t3_pk;

ALTER TABLE t2 RENAME CONSTRAINT t2_pk TO t1_pk;
ALTER TABLE t2 RENAME TO t1;
ALTER INDEX t2_pk RENAME TO t1_pk;
```

### 20.5. 파티션
<br/>
파티션(partition)은 서브 오브젝트(subobject)다. 다수의 물리적 파티션이 하나의 논리적 오브젝트로 관리된다. 테이블, 제약 조건, 인덱스에 대한 지식이 있어야 파티션을 이해할 수 있다.  

#### 20.5.1. 파티션 테이블
<br/>
파티션 테이블(partitioned table)은 다수의 물리적 테이블 파티션을 하나의 논리적 테이블로 사용할 수 있는 오브젝트다. 테이블 파티션은 파티션 키로 분할되며 테이블과 동일한 구조로 생성된다. 파티션 테이블은 OLTP 시스템의 DML 경합 해소, DW 시스템의 조회 성능 개선, 효율적인 ILM(Information Lifecycle Management) 등의 목적으로 사용된다.  

##### 20.5.1.1. 파티션 유형
<br/>
버전 별로 다양한 파티션을 사용할 수 있다.  

+ 8 : RANGE 파티션, HASH 파티션
+ 9 : LIST 파티션
+ 11.1 : SYSTEM 파티션, INTERVAL 파티션, REFERENCE 파티션
+ 12.1 : INTERVAL-REFERENCE 파티션  

###### 20.5.1.1.1. RANGE 파티션
<br/>
RANGE 파티션은 파티션 키 값의 범위로 파티션을 분할한다. RANGE 파티션은 기간별로 입력되는 데이터를 분할할 때 사용할 수 있다.  

```
PARTITION BY RANGE (column [, column]...)
    PARTITION [partition] VALUES LESS THAN ({literal | MAXVALUE} [, {...}...])
 [, PARTITION [partition] VALUES LESS THAN ({literal | MAXVALUE} [, {...}...])]
```

아래 쿼리는 RANGE 파티션 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 DATE)
PARTITION BY RANGE (c2) (
    PARTITION p1 VALUES LESS THAN (DATE '2050-02-01')
  , PARTITION p2 VALUES LESS THAN (DATE '2050-03-01'));
```

아래와 같이 데이터를 입력해보자. 세 번째 쿼리는 데이터를 삽입할 파티션이 존재하지 않기 때문에 에러가 발생한다.  

```sql
INSERT INTO t1 VALUES (1, DATE '2050-01-01');
INSERT INTO t1 VALUES (1, DATE '2050-02-01');
INSERT INTO t1 VALUES (1, DATE '2050-03-01');
```

아래와 같이 t1 테이블을 다시 생성하자. 마지막 파티션(p3)의 값을 MAXVALUE로 지정했다. MAXVALUE는 직전 파티션의 literal보다 큰 최고 값을 의미한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 DATE)
PARTITION BY RANGE (c2) (
    PARTITION p1 VALUES LESS THAN (DATE '2050-02-01')
  , PARTITION p2 VALUES LESS THAN (DATE '2050-03-01')
  , PARTITION p3 VALUES LESS THAN (MAXVALUE));
```

다시 데이터를 입력해보자. 세 번째 쿼리도 에러가 발생하지 않는다.  

```sql
INSERT INTO t1 VALUES (1, DATE '2050-01-01');
INSERT INTO t1 VALUES (1, DATE '2050-02-01');
INSERT INTO t1 VALUES (1, DATE '2050-03-01');
COMMIT;
```

&#42;&#95;PART&#95;TABLES 뷰에서 파티션 테이블에 대한 정보를 조회할 수 있다.  

```sql
SELECT partition_type, partition_count, partition_key_count
FROM user_part_tables
WHERE table_name = 'T1';
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 테이블 파티션에 대한 정보를 조회할 수 있다.  

```sql
SELECT partition_name, high_value
FROM user_tab_partitions
WHERE table_name = 'T1'
ORDER BY partition_position;
```

&#42;&#95;PART&#95;KEY&#95;COLUMNS 뷰에서 파티션 키에 대한 정보를 조회할 수 있다.  

```sql
SELECT object_type, column_name, column_position
FROM user_part_key_columns
WHERE NAME = 'T1';
```

테이블 파티션은 &#42;&#95;OBJECTS 뷰의 subobject&#95;name 열에 파티션 명이 표시되고, object&#95;type 열이 TABLE&#95;PARTITION으로 표시된다.  

```sql
SELECT object_name, subobject_name, object_type
FROM user_object
WHERE object_name = 'T1';
```

t1 테이블은 &#42;&#95;SEGMENTS 뷰에서 조회되지 않는다. 파티션 테이블은 비 세그먼트 오브젝트로 논리적으로만 존재하고 데이터는 물리적인 테이블 파티션에 저장된다.  

```sql
SELECT segment_name, partition_name, segment_type, tablespace_name
FROM user_segments
WHERE segment_name = 'T1';
```

파티션 확장 절(partition extension clause)을 사용하면 지정한 파티션만 조회할 수 있다. FOR 키워드는 11.1 버전부터 사용할 수 있다.  

```
{ PARTITION (partition)
| PARTITION FOR (partition_key_value [, partition_key_value]...)
| SUBPARITION (subpartition)
| SUBPARITION FOR (subpartition_key_value [, subpartition_key_value]...)}
```

아래 쿼리는 파티션 확장 절을 사용하여 t1 테이블의 p1 파티션을 조회한다.  

```sql
SELECT * FROM t1 PARTITION (p1);
```

아래 쿼리는 파티션 키 값이 2050-02-10에 해당하는 t1 파티션을 조회한다.  

```sql
SELECT * FROM t1 PARTITION FOR (DATE '2050-02-10');
```

다중 열로도 RANGE 파티션을 생성할 수 있다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER)
PARITION BY RANGE (c1, c2) (
    PARTITION p1 VALUES LESS THAN (1, 1)
  , PARTITION p2 VALUES LESS THAN (1, 2)
  , PARTITION p3 VALUES LESS THAN (2, 1)
  , PARTITION p4 VALUES LESS THAN (2, 2)
  , PARTITION p5 VALUES LESS THAN (MAXVALUE, MAXVALUE));
```

아래와 같이 데이터를 입력해보자.  

```sql
INSERT INTO t1 VALUES (0, 0);
INSERT INTO t1 VALUES (0, 1);
INSERT INTO t1 VALUES (0, 2);
INSERT INTO t1 VALUES (1, 0);
INSERT INTO t1 VALUES (1, 1);
INSERT INTO t1 VALUES (1, 2);
INSERT INTO t1 VALUES (2, 0);
INSERT INTO t1 VALUES (2, 1);
INSERT INTO t1 VALUES (2, 2);
INSERT INTO t1 VALUES (3, 0);
INSERT INTO t1 VALUES (3, 1);
INSERT INTO t1 VALUES (3, 2);
COMMIT;
```

아래 쿼리로 행에 해당하는 파티션을 조회할 수 있다. TBL$OR$IDX$PART$NUM 함수는 오라클 데이터베이스가 내부적으로 사용하는 문서화되지 않은 함수다. 세 번째 매개변수에 파티션은 1, 서브 파티션은 3을 입력해야 한다.  

```sql
SELECT a.*, TBL$OR$IDX$PART$NUM (SCOTT.T1, 0, 1, 0, ROWID) AS pn
FROM t1 a
ORDER BY 1, 2;
```

아래와 같이 MAXVALUE를 사용해서 테이블을 생성한 후 데이터를 입력해보자.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER)
PARITION BY RANGE (c1, c2) (
    PARTITION p1 VALUES LESS THAN (1, 1)
  , PARTITION p2 VALUES LESS THAN (1, MAXVALUE)
  , PARTITION p3 VALUES LESS THAN (2, 1)
  , PARTITION p4 VALUES LESS THAN (2, MAXVALUE)
  , PARTITION p5 VALUES LESS THAN (MAXVALUE, MAXVALUE));
```

###### 20.5.1.1.2. HASH 파티션
<br/>
HASH 파티션은 파티션 키 값에 대한 해시 함수의 결과 값으로 파티션을 분할한다. 해시 함수의 결과 값에 따라 데이터가 무작위로 분산된다. OLTP 시스템의 블록 경합을 해소하기 위해 사용할 수 있다.  

```
PARTITION BY HASH (column [, column]...) PARTITIONS hash_partition_quantity
```

아래 쿼리는 HASH 파티션 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER) PARTITION BY HASH (c1) PARTITIONS 3;
```

&#42;&#95;TAB&#95;PARTITIONS 뷰를 조회해보면 파티션 명이 자동으로 부여된 것을 확인할 수 있다.  

```sql
SELECT partiton_name, high_value FROM user_tab_partitions WHERE table_name = 'T1';
```

아래의 구문을 사용하면 파티션 명을 지정해서 HASH 파티션 테이블을 생성할 수 있다. HASH 파티션은 다수의 파티션을 생성하기 때문에 파티션 명을 지정하지 않는 것이 일반적이다.  

```
PARTITION BY HASH (column [, column]...) (
    PARTITION [partition]
 [, PARTITION [partition]]...)
```

아래 쿼리는 파티션 명을 지정해서 HASH 파티션 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER)
PARTITION BY HASH (c1) (PARTITION p1, PARTITION p2, PARTITION p3);
```

###### 20.5.1.1.3. LIST 파티션
<br/>
LIST 파티션은 파티션 키 값의 목록으로 파티션을 분할한다. 키 값이 고정적이며 데이터 분포가 균등하지 않은 데이터를 분할할 때 사용할 수 있다.  

```
PARTITION BY LIST (column [, column]...) (
    PARTITION [partition] VALUES ({literal | NULL} [, {literal | NULL}]... | DEFAULT)
 [, PARTITION [partition] VALUES ({literal | NULL} [, {literal | NULL}]... | DEFAULT]...)
```

아래 쿼리는 LIST 파티션 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER)
PARTITION BY LIST (c2) (
    PARTITION p1 VALUES (1, 2)
  , PARTITION p2 VALUES (3, 4)
  , PARTITION p3 VALUES (DEFAULT));
```

12.2 버전부터 다중 열로도 LIST 파티션을 생성할 수 있다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER)
PARTITION BY LIST (c2) (
    PARTITION p1 VALUES ((1, 1), (1, 2))
  , PARTITION p2 VALUES ((2, 1), (3, 4))
  , PARTITION p3 VALUES (DEFAULT));
```

아래 쿼리는 자동 LIST 파티션 테이블을 생성한다. 자동 LIST 파티션도 12.2 버전부터 사용할 수 있다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER)
PARTITION BY LIST (c2) AUTOMATIC (
    PARTITION p1 VALUES (1)
  , PARTITION p2 VALUES (2));
```

아래와 같이 데이터를 입력해보자.  

```sql
INSERT INTO t1 VALUES (1, 1);
INSERT INTO t1 VALUES (2, 2);
INSERT INTO t1 VALUES (3, 3);
COMMIT;
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 자동으로 파티션이 추가된 것을 확인할 수 있다.  

```sql
SELECT partition_name, high_value
FROM user_tab_partitions
WHERE table_name = 'T1';
```

###### 20.5.1.1.4. SYSTEM 파티션
<br/>
SYSTEM 파티션은 파티션 키가 없는 파티션이다. 애플리케이션에서 임의로 데이터를 분할할 때 사용할 수 있다.  

```
PARTITION BY SYSTEM (PARTITION [partition] [, PARTITION [partition]...])
```

아래 쿼리는 SYSTEM 파티션 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER)
PARTITION BY SYSTEM (PARTITION p1, PARTITION p2, PARTITION p3);
```

아래 쿼리는 에러가 발생한다. ORA-14701 : 시스템 방식에 의해 분할된 테이블의 DML에는 분할 영역으로 확장된 이름 또는 바인드 변수를 사용해야 합니다.  

```sql
INSERT INTO t1 VALUES (1);
```

SYSTEM 파티션은 파티션 단위로만 데이터를 입력할 수 있다.  

```sql
INSERT INTO t1 PARTITION (p1) VALUES (1);
```

조회는 테이블, 파티션 양쪽 모두 가능하다.  

```sql
SELECT * FROM t1;

SELECT * FROM t1 PARTITION (p1);
```

아래 구문을 사용하면 파티션 개수를 지정해서 SYSTEM 파티션을 생성할 수 있다. 자동으로 파티션 명이 부여되기 때문에 활용도가 높지 않다.  

```sql
PARTITION BY SYSTEM PARTITIONS integer
```

아래 쿼리는 파티션 개수를 지정해서 SYSTEM 파티션 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER) PARTITION BY SYSTEM PARTITIONS 3;
```

###### 20.5.1.1.5. INTERVAL 파티션
<br/>
INTERVAL 파티션은 RANGE 파티션의 확장 기능이다. 기간에 해당하는 데이터가 입력되면 파티션이 자동으로 추가된다.  

```
PARTITON BY RANGE (column [, column]...) [INTERVAL (expr)] (
    PARTITION [partition] VALUES LESS THAN ({literal | MAXVALUE} [, {...}...])
 [, PARTITION [partition] VALUES LESS THAN ({literal | MAXVALUE} [, {...}...])]...)
```

아래 쿼리는 INTERVAL을 1달로 지정한 INTERVAL 파티션 테이블을 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 DATE)
PARTITION BY RANGE (c2) INTERVAL (NUMTOYMINTERVAL (1, 'MONTH')) (
  PARTITION p1 VALUES LESS THAN (DATE '2050-02-01'));
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 파티션이 1개인 것을 확인할 수 있다.  

```sql
SELECT partition_name, high_value
FROM user_tab_partitions
WHERE table_name = 'T1'
ORDER BY partition_position;
```

아래와 같이 데이터를 입력해보자.  

```sql
INSERT INTO t1 VALUES (1, DATE '2050-01-01');
INSERT INTO t1 VALUES (1, DATE '2050-03-01');
INSERT INTO t1 VALUES (1, DATE '2050-02-01');
COMMIT;
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 파티션이 추가된 것을 확인할 수 있다. 파티션 명은 자동으로 부여된다.  

```sql
SELECT partition_name, high_value
FROM user_tab_partitions
WHERE table_name = 'T1'
ORDER BY partition_position;
```

###### 20.5.1.1.6. REFERENCE 파티션
<br/>
REFERENCE 파티션은 FK 제약 조건을 통해 부모 테이블과 동일한 구조의 파티션을 생성한다. 부모 테이블의 파티션이 변경되면 REFERENCE 파티션 테이블도 함께 변경된다.  

```
PARTITION BY REFERENCE (constraint) [(reference_partition_desc...)]
```

아래 쿼리는 t2 테이블을 t1 테이블의 REFERENCE 파티션 테이블로 생성한다. t1 테이블의 파티션 키인 c2 열은 DATE 타입이다. t2 테이블에는 해당 열이 존재하지 않는다.  

```sql
DROP TABLE t2 CASCADE CONSTRAINTS PURGE;
DROP TABLE t1 CASCADE CONSTRAINTS PURGE;

CREATE TABLE t1 (c1 NUMBER NOT NULL, c2 DATE, CONSTRAINT t1_pk PRIMARY KEY (c1))
PARTITION BY RANGE (c2) (
    PARTITION p1 VALUES LESS THAN (DATE '2050-02-01')
  , PARTITION p2 VALUES LESS THAN (DATE '2050-03-01')
  , PARTITION p3 VALUES LESS THAN (MAXVALUE));

CREATE TABLE t2 (c1 NUMBER NOT NULL, c2 NUMBER NOT NULL, c3 NUMBER
               , CONSTRAINT t2_pk PRIMARY KEY (c1, c2)
               , CONSTRAINT t2_f1 FOREIGN KEY (c1) REFERENCES t1 (c1))
PARTITION BY REFERENCE (t2_f1);
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 t2 테이블이 분할된 것을 확인할 수 있다.  

```sql
SELECT partition_name, high_value FROM user_tab_partitions WHERE table_name = 'T2';
```

아래와 같이 데이터를 입력해보자.  

```sql
INSERT INTO t1 VALUES (1, DATE '2050-01-01');
INSERT INTO t1 VALUES (2, DATE '2050-02-01');
INSERT INTO t1 VALUES (3, DATE '2050-02-02');
INSERT INTO t2 VALUES (1, 1, 1);
INSERT INTO t2 VALUES (2, 2, 2);
INSERT INTO t2 VALUES (3, 3, 3);
COMMIT;
```

아래 쿼리에서 t1 테이블의 p2 파티션에 c2가  2050-02-01, 2050-02-02인 행이 저장된 것을 확인할 수 있다.  

```sql
SELECT * FROM t1 PARTITION (p2);
```

아래 쿼리에서 t2 테이블의 p2 파티션에 c1이 2, 3인 행이 저장된 것을 확인할 수 있다.  

```sql
SELECT * FROM t2 PARTITION (p2);
```

REFERENCE 파티션은 PARTITION FOR 절을 사용할 수 없다.  

```sql
SELECT * FROM t2 PARTITION FOR (DATE '2050-02-01');
```

###### 20.5.1.1.7. INTERVAL-REFERENCE 파티션
<br/>
12.1 버전부터 INTERVAL 파티션도 REFERENCE 파티션을 사용할 수 있다. 이전 버전까지는 RANGE, HASH, LIST 파티션만 REFERENCE 파티션을 사용할 수 있었다.  

아래 쿼리는 t2 테이블을 INTERVAL 파티션 테이블인 t1 테이블의 REFERENCE 파티션 테이블로 생성한다.  

```sql
DROP TABLE t2 CASCADE CONSTRAINTS PURGE;
DROP TABLE t2 CASCADE CONSTRAINTS PURGE;

CREATE TABLE t1 (c1 NUMBER NOT NULL, c2 DATE, CONSTRAINT t1_pk PRIMARY KEY (c1))
PARTITION BY RANGE (c2) INTERVAL (NUMTOYMINTERVAL (1, 'MONTH')) (
  PARTITION p1 VALUES LESS THAN (DATE '2050-02-01'));

CREATE TABLE t2 (c1 NUMBER NOT NULL, c2 NUMBER NOT NULL, c3 NUMBER)
               , CONSTRAINT t2_pk PRIMARY KEY (c1, c2)
               , CONSTRAINT t2_f1 FOREIGN KEY (c1) REFERENCES t1 (c1))
PARTITION BY REFERENCE (t2_f1);
```

##### 20.5.1.2. COMPOSITE 파티션
<br/>
COMPOSITE 파티션(composite partitioning)은 파티션의 조합이다. 파티션이 서브 파티션으로 분할된다. 지금까지 살펴본 파티션은 모두 단일 레벨 파티션(single-level partitioning)이다.  

버전 별로 아래의 조합이 가능하다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>HASH</td>
      <td>LIST</td>
      <td>RANGE</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>RANGE</td>
      <td>8</td>
      <td>9</td>
      <td>11.1</td>
    </tr>
    <tr>
      <td>LIST</td>
      <td>11.1</td>
      <td>11.1</td>
      <td>11.1</td>
    </tr>
    <tr>
      <td>INTERVAL</td>
      <td>11.1</td>
      <td>11.1</td>
      <td>11.1</td>
    </tr>
    <tr>
      <td>HASH</td>
      <td>12.1</td>
      <td>12.1</td>
      <td>12.1</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 쿼리는 RANGE-HASH 파티션 테이블을 생성한다.  

```sql
DROP TABLE t2 CASCADE CONSTRAINTS PURGE;
DROP TABLE t1 CASCADE CONSTRAINTS PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 DATE, c3 NUMBER)
PARTITION BY RANGE (c2) SUBPARTITION BY HASH (c3) SUBPARTITIONS 2 (
    PARTITION p1 VALUES LESS THAN (DATE '2050-02-01')
  , PARTITION p2 VALUES LESS THAN (DATE '2050-03-01')
  , PARTITION p3 VALUES LESS THAN (MAXVALUE));
```

&#42;&#95;PART&#95;TABLES 뷰에서 서브 파티션 유형과 기본 개수를 확인할 수 있다.  

```sql
SELECT partitioning_type, subpartitioning_type, partition_count, def_subpartition_count
FROM user_part_tables
WHERE table_name = 'T1';
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 파티션 별 서브 파티션의 개수를 확인할 수 있다.  

```sql
SELECT composite, partition_name, subpartition_count, partition_position
FROM user_tab_partitions
WHERE table_name = 'T1'
ORDER BY partition_position;
```

&#42;&#95;TAB&#95;SUBPARTITIONS 뷰에서 테이블 서브 파티션에 대한 정보를 조회할 수 있다.  

```sql
SELECT partition_name, subpartition_name, partition_position, subpartition_position
FROM user_tab_subpartitions
WHERE table_name = 'T1'
ORDER BY partition_position, subpartition_position;
```

&#42;&#95;SUBPART&#95;KEY&#95;COLUMNS 뷰에서 서브 파티션 키에 대한 정보를 조회할 수 있다.  

```sql
SELECT column_name, column_position FROM dba_subpart_key_columns WHERE name = 'T1';
```

아래 쿼리는 RANGE-LIST 파티션 테이블을 생성한다. 서브 파티션을 반복 기술해야 하므로 구문이 길어질 수 있다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 DATE, c3 NUMBER)
PARTITION BY RANGE (c2) SUBPARTITION BY LIST (c3) (
    PARTITION p1 VALUES LESS THAN (DATE '2050-02-01') (
        SUBPARTITION p1_s1 VALUES (1, 2)
      , SUBPARTITION p1_s2 VALUES (DEFAULT))
  , PARTITION p2 VALUES LESS THAN (DATE '2050-03-01') (
      SUBPARTITION p2_s1 VALUES (1, 2)
    , SUBPARTITION p2_s2 VALUES (DEFAULT))
  , PARTITION p3 VALUES LESS THAN (MAXVALUE) (
      SUBPARTITION p3_s1 VALUES (1, 2)
    , SUBPARTITION p3_s2 VALUES (DEFAULT)));
```

서브 파티션 템플릿(subpartition template)을 사용하면 구문을 간결하게 작성할 수 있다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 DATE, c3 NUMBER)
PARTITION BY RANGE (c2) SUBPARTITION BY LIST (c3)
    SUBPARTITION TEMPLATE (
        SUBPARTITION s1 VALUES (1, 2)
      , SUBPARTITION s2 VALUES (DEFAULT)) (
          PARTITION p1 VALUES LESS THAN (DATE '2050-02-01')
        , PARTITION p2 VALUES LESS THAN (DATE '2050-03-01')
        , PARTITION p3 VALUES LESS THAN (MAXVALUE));
```

서브 파티션 명이 {partition}&#95;{subpartition} 형태로 부여된다.  

```sql
SELECT partition_name, subpartition_name
FROM user_tab_subpartitions
WHERE table_name = 'T1';
```

아래와 같이 데이터를 입력해보자.  

```sql
INSERT INTO t1 VALUES (1, DATE '2050-01-01', 1);
INSERT INTO t1 VALUES (1, DATE '2050-01-01', 3);
COMMIT;
```

서브 파티션도 파티션 확장 절을 사용할 수 있다.  

```sql
SELECT * FROM t1 SUBPARTITION (p1_s1);
```

FOR 키워드를 사용하면 파티션 키와 서브 파티션 키의 값을 모두 입력해야 한다.  

```sql
SELECT * FROM t1 SUBPARTITION FOR (DATE '2050-01-10', 3);
```

##### 20.5.1.3. 파티션 IOT
<br/>
파티션 IOT(partitioned index-organized table)은 IOT로 파티션을 생성한다.  

아래 쿼리는 파티션 IOT을 생성한다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 DATE, c3 NUMBER, CONSTRAINT t1_pk PRIMARY KEY (c1, c2))
ORGANIZATION INDEX
PARTITION BY RANGE (c2) (
    PARTITION p1 VALUES LESS THAN (DATE '2050-02-01')
  , PARTITION p2 VALUES LESS THAN (DATE '2050-03-01')
  , PARTITION p3 VALUES LESS THAN (MAXVALUE));
```

##### 20.5.1.4. 파티션 해시 클러스터 테이블
<br/>
파티션 해시 클러스터 테이블(partitioned hash clustered table)은 클러스터로 파티션을 생성한다. 12.1.0.2 버전부터 사용할 수 있다.  

아래 쿼리는 파티션 해시 클러스터 테이블을 생성한다.  

```sql
DROP CLUSTER c1# INCLUDING TABLES;
DROP TABLE t1 PURGE;

CREATE CLUSTER c1# (c1 NUMBER, c2 NUMBER) HASHKEYS 100 HASH IS c1
PARTITION BY RANGE (c2) (
    PARTITION p1 VALUES LESS THAN (100)
  , PARTITION p2 VALUES LESS THAN (200)
  , PARTITION p3 VALUES LESS THAN (MAXVALUE));

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, c3 NUMBER) CLUSTER c1# (c1, c2);
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 파티션에 대한 정보를 조회할 수 있다.  

```sql
SELECT partition_name, high_value FROM user_tab_partitions WHERE table_name = 'T1';
```

#### 20.5.2. 파티션 인덱스
<br/>
파티션 인덱스(partitioned index)도 다수의 물리적 인덱스 파티션을 하나의 논리적 인덱스로 사용할 수 있는 오브젝트다. 인덱스 파티션은 파티션 키로 분할되며 인덱스와 동일한 구조로 생성된다.  

인덱스는 파티션 여부와 테이블 종속에 따라 아래와 같이 구분할 수 있다.  

+ 비 파티션 인덱스 : 파티셔닝되지 않은 인덱스
+ 로컬 파티션 인덱스 : 파티션 테이블과 동일한 구조로 파티셔닝된 인덱스
+ 글로벌 파티션 인덱스 : 테이블과 관계없이 파티셔닝된 인덱스  

아래 표에서 테이블 별로 생성할 수 있는 인덱스 유형을 확인할 수 있다. 로컬 파티션 인덱스는 파티션 테이블에 종속되어 있기 때문에 파티션 테이블이 변경되면 함께 변경된다. 비 파티션 인덱스와 글로벌 파티션 인덱스는 파티션 테이블이 변경되면 UNUSABLE 상태로 변경된다.  

파티션 인덱스는 파티션 키가 인덱스의 선두 열이면 PREFIXED 파티션 인덱스, 선두 열이 아니면 NONPREFIXED 파티션 인덱스로 구분된다. NONPREFIXED 글로벌 파티션 인덱스는 생성할 수 없다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>PREFIXED</td>
      <td>NONPREFIXED</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>로컬 파티션 인덱스</td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>글로벌 파티션 인덱스</td>
      <td>Y</td>
      <td>N</td>
    </tr>
  </tbody>
</table>
<br/><br/>

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, c3 NUMBER)
PARTITION BY RANGE (c1) SUBPARTITON BY HASH (c2) SUBPARTITIONS 2 (
    PARTITION p1 VALUES LESS THAN (1)
  , PARTITION p2 VALUES LESS THAN (2)
  , PARTITION p3 VALUES LESS THAN (3)
  , PARTITION p4 VALUES LESS THAN (MAXVALUE));

INSERT INTO t1 VALUES (1, 1, 1);
INSERT INTO t1 VALUES (2, 2, 2);
INSERT INTO t1 VALUES (3, 3, 3);
INSERT INTO t1 VALUES (4, 4, 4);
COMMIT;
```

아래와 같이 UNIQUE 인덱스를 생성하자. t1&#95;u3 인덱스는 에러가 발생한다. UNIQUE 로컬 파티션 인덱스는 인덱스 열에 파티션 키가 포함되어야 한다.  

```sql
CREATE UNIQUE INDEX t1_u1 ON t1 (c1);
CREATE UNIQUE INDEX t1_u2 ON t1 (c1, c2) LOCAL;
CREATE UNIQUE INDEX t1_u3 ON t1 (c2) LOCAL;

ORA-14039 : 열을 분할영역한 것은 UNIQUE 인덱스로 키 열의 부분 집합을 폼 합니다.
```

아래와 같이 NONUNIQUE 인덱스를 생성하자. t1&#95;x4 인덱스는 에러가 발생한다. 글로벌 파티션 인덱스는 PREFIXED 인덱스만 생성할 수 있다.  

```sql
CREATE INDEX t1_x1 ON t1 (c2, c3) LOCAL;
CREATE INDEX t1_x2 ON t1 (c3) LOCAL;
CREATE INDEX t1_x3 ON t1 (c3, c2) LOCAL GLOBAL PARTITION BY HASH (c3) PARTITIONS 2;
CREATE INDEX t1_x4 ON t1 (c1, c3) LOCAL GLOBAL PARTITION BY HASH (c3) PARTITIONS 2;

ORA-14038: GLOBAL로 분할영역된 인덱스는 접두사이어야 합니다.
```

&#42;&#95;INDEXES 뷰의 partitioned 열에서 인덱스의 파티션 여부를 확인할 수 있다.  

```sql
SELECT index_name, uniqueness, partitioned, status
FROM user_indexes
WHERE table_name = 'T1'
ORDER BY index_name;
```

&#42;&#95;PART&#95;INDEXES 뷰에서 파티션 인덱스에 대한 정보를 조회할 수 있다.  

```sql
SELECT index_name, partitioning_type, subpartitioning_type, locality, alignment
FROM user_part_indexes
WHERE tabel_name = 'T1'
ORDER BY index_name;
```

&#42;&#95;PART&#95;IND&#95;PARTITIONS 뷰에서 인덱스 파티션에 대한 정보를 조회할 수 있다.  

```sql
SELECT index_name, composite, partition_name, partition_position, status
FROM user_ind_partitions
WHERE index_name IN ('T!_U2', 'T1_X3')
ORDER BY index_name, subpartition_position;
```

&#42;&#95;PART&#95;IND&#95;SUBPARTITIONS 뷰에서 인덱스 서브 파티션에 대한 정보를 조회할 수 있다.  

```sql
SELECT partition_name, subpartition_name, partition_position, subpartition_position, status
FROM user_ind_subpartitions
WHERE index_name = 'T1_U2'
ORDER BY partition_position, subpartition_position;
```

#### 20.5.3. 관리 구문
<br/>
파티션은 다수의 관리 구문을 사용할 수 있다. RANGE 파티션을 기준으로 자주 사용되는 구문을 살펴보자.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER)
PARTITION BY RANGE (c2) (
    PARTITION p1 VALUES LESS THAN (2)
  , PARTITION p2 VALUES LESS THAN (3));

CREATE INDEX t1_x1 ON t1 (c1);
CREATE INDEX t1_x2 ON t1 (c2) LOCAL;

INSERT INTO t1 VALUES (1, 1);
INSERT INTO t1 VALUES (2, 2);
COMMIT;
```

##### 20.5.3.1. ADD 절
<br/>
ADD 절은 파티션을 추가한다.  

```
ALTER [scehma.]table ADD PARTITION [partition] partition_clause
                      [, PARTITION [partition] partition_clause]...;
```

아래 쿼리는 t1 테이블에 p4 파티션을 추가한다.  

```sql
ALTER TABLE t1 ADD PARTITION p4 VALUES LESS THAN (MAXVALUE);

INSERT INTO t1 VALUES (3, 1);
COMMIT;
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 파티션이 추가된 것을 확인할 수 있다.  

```sql
SELECT partition_name, high_value FROM user_tab_partitions WHERE table_name = 'T1';
```

아래 쿼리는 에러가 발생한다. 마지막 파티션보다 키 값이 큰 파티션만 추가할 수 있다.  

```sql
ALTER TABLE t1 ADD PARTITION p3 VALUES LESS THAN (4);

ORA-14074: 분할영역 유지 작업에 분할역역 범위가 너무 적습니다.
```

##### 20.5.3.2. DROP 절
<br/>
DROP 절은 파티션을 삭제한다. 12.1 버전부터 ADD 절, DROP 절, MERGE 절, SPLIT 절, TRUNCATE 절에 다수의 파티션을 기술할 수 있다.  

```
ALTER [schema.]table DROP {PARTITION | PARTITIONS} {partition [, partition]}
  [update_index_clauses];
```

UPDATE INDEX 절은 인덱스에 대한 작업을 설정한다. 아래의 네 가지 유형으로 사용할 수 있다. DROP 절은 INVALIDATE GLOBAL INDEXES, UPDATE GLOBAL INDEXES 유형만 사용할 수 있다.  

```
{UPDATE | INVALIDATE} GLOBAL INDEXES | UPDATE INDEXES [(index [, index]...)]
```

+ INVALIDATE GLOBAL INDEXES : 글로벌 인덱스를 갱신하지 않고 UNUSABLE로 변경 (기본값)
+ UPDATE GLOBAL INDEXES : 글로벌 인덱스를 갱신
+ UPDATE INDEXES : 모든 글로벌 인덱스와 로컬 인덱스를 갱신
+ UPDATE INDEXES (...) : 지정된 인덱스를 갱신  

아래 쿼리는 p4 파티션을 삭제한다.  

```sql
ALTER TABLE t1 DROP PARTITION p4;
```

ADD 절을 제외한 관리 구문은 파티션 확장 절을 사용할 수 있다. 아래 쿼리는 위 쿼리와 결과가 동일하다.  

```sql
ALTER TABLE t1 DROP PARTITION FOR (5);
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 파티션이 삭제된 것을 확인할 수 있다.  

```sql
SELECT partition_name, high_value FROM user_tab_partitions WHERE table_name = 'T1';
```

다음 예제를 위해 아래와 같이 p3 파티션을 추가하자.  

```sql
ALTER TABLE t1 ADD PARTITION p3 VALUES LESS THAN (5);

INSERT INTO t1 VALUES (3, 3);
INSERT INTO t1 VALUES (4, 4);
```

##### 20.5.3.3. MERGE 절
<br/>
MERGE 절은 파티션을 병합한다. INCLUDING ROWS는 12.2 버전부터 사용할 수 있다.  

```
ALTER [schema.]table MERGE PARTITIONS
  partition {, partition [, partition]... | TO partition}
  [INTO partition]
  [INCLUDING ROWS where_clause]
  [update_index_clauses]
```

아래 쿼리는 p2, p3 파티션을 p3 파티션으로 병합한다.  

```sql
ALTER TABLE t1 MERGE PARTITIONS p2, p3 INTO PARTITION p3;
```

아래와 같이 범위를 지정할 수도 있다.  

```sql
ALTER TABLE t1 MERGE PARTITIONS p2 TO p3 INTO PARTITION p3;
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 파티션이 병합된 것을 확인할 수 있다.  

```sql
SELECT partition_name, high_value FROM user_tab_partitions WHERE table_name = 'T1';
```

##### 20.5.3.4. SPLIT 절
<br/>
SPLIT 절은 파티션을 분할한다. ONLINE은 12.2 버전부터 사용할 수 있다.  

```
ALTER [schema.]table SPLIT partition
    {AT (literal [, literal]...) | VALUES (list_values)} [INTO (partition, partition)]
    [INCLUDING ROWS where_clause]
    [update_index_clauses]
    [ONLINE];
```

아래 쿼리는 3을 기준으로 p3 파티션을 p2, p3 파티션으로 분할한다.  

```sql
ALTER TABLE t1 SPLIT PARTITION p3 AT (3) INTO (PARTITION p2, PARTITION p3);
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 파티션이 분할된 것을 확인할 수 있다.  

```sql
SELECT partition_name, high_value FROM user_tab_partitions WHERE table_name = 'T1';
```

12.2 버전부터 하나의 파티션을 다수의 파티션으로 분할할 수 있다.  

```
SPLIT partition INTO (partition [, partition]..., partition)
```

아래 쿼리는 p3 파티션을 p3, p4 파티션으로 분할한다.  

```sql
ALTER TABLE t1 SPLIT PARTITION p3 INTO (
    PARTITION p3 VALUES LESS THAN (4)
  , PARTITION p4);
```

&#42;&#95;TAB&#95;PARTITIONS 뷰에서 파티션이 분할된 것을 확인할 수 있다.  

```sql
SELECT partition_name, high_value FROM user_tab_partitions WHERE table_name = 'T1';
```

##### 20.5.3.5. EXCHANGE 절
<br/>
EXCHANGE 절은 파티션을 교체한다. CASCADE는 12.1 버전부터 사용할 수 있다.  

```
ALTER [schema.]table EXCHANGE {partition | subpartition} WITH TABLE [schema.] table
  [{INCLUDING | EXCLUDING} INDEXES]
  [{WITH | WITHOUT} VALIDATION]
  [EXCEPTIONS INTO [schema.]table]
  [update_index_clauses]
  [CASCADE];
```

+ {INCLDING | EXCLUDING} INDEXES : 인덱스 포함 여부를 결정
+ {WITH | WITHOUT} VALIDATION : 검증 여부를 결정
+ EXCEPTIONS INTO [schema.]table : 예외 처리
+ CASCADE : REFERENCE 파티션에 대한 종속 작업을 수행  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t2 PURGE;
CREATE TABLE t2 AS SELECT * FROM t1 WHERE 0 = 1;
CREATE TABLE t2_x1 ON t2 (c2);

INSERT INTO t2 VALUES (5, 1);
INSERT INTO t2 VALUES (6 ,2);
COMMIT;
```

아래 쿼리는 p1 파티션을 t2 테이블로 교체한다.  

```sql
ALTER TABLE t1 EXCHANGE PARTITION p1 WITH TABLE t2 INCLUDING INDEXES WITHOUT VALIDATION;
```

t1 테이블의 p1 파티션을 조회해보자. 파티션 범위에 속하지 않는 데이터(c2 = 2)가 p1 파티션에 입력된 것을 확인할 수 있다. WITHOUT VALIDATION은 대상 데이터에 대한 검증을 수행한 후 사용해야 한다.  

```sql
SELECT * FROM t1 PARTITION (p1);
```

##### 20.5.3.6. MOVE 절
<br/>
MOVE 절은 파티션을 재배치한다. ONLINE은 12.1 버전부터 사용할 수 있다.  

```
ALTER [schema.]table MOVE PARTITION partition
  [TABLESPACE tablespace]
  [INCLUDING ROWS where_clause]
  [update_index_clauses]
  [ONLINE];
```

아래 쿼리는 p1 파티션을 재배치한다. INCLUDING ROWS를 사용했다.  

```sql
ALTER TABLE t1 MOVE PARTITION p1 INCLDING ROWS WHERE c2 < 2;
```

t1 테이블의 p1 파티션에서 c2가 2인 행이 제거된 것을 확인할 수 있다.  

```sql
SELECT * FROM t1 PARTITION (p1);
```

##### 20.5.3.7. RENAME 절
<br/>
RENAME 절은 파티션 명을 변경한다.  

```
ALTER [schema.]table RENAME {PARTITION partition | SUBPARTITION subpartition} TO new_name;
```

아래 쿼리는 p4 파티션 명을 p5로 변경한다.  

```sql
ALTER TABLE t1 RENAME PARTITION p4 TO p5;
```

테이블 파티션 명을 변경하더라도 로컬 파티션 인덱스의 파티션 명은 변경되지 않는다. 아래와 같이 직접 변경해야 한다.  

```sql
ALTER INDEX t1_x2 RENAME PARTITION p4 TO p5;
```

##### 20.5.3.8. TRUNCATE 절
<br/>
TRUNCATE 절은 파티션을 초기화한다.  

```
ALTER [schema.]table TRUNCATE {PARTITION | PARTITIONS} {partition [, partition]}
  [{DROP [ALL] | REUSE} STORAGE]
  [update_index_clauses]
  [CASCADE];
```

아래 쿼리는 p5 파티션의 전체 행을 삭제한다.  

```sql
ALTER TABLE t1 TRUNCATE PARTITION p5;
```

##### 20.5.3.9. MODIFY 절
<br/>
MODIFY 절은 파티션의 속성을 변경한다. READ ONLY는 12.2 버전부터 사용할 수 있다.  

```
ALTER [schema.]table MODIFY partition
  { {ADD | DROP} VALUES (list_values)
  | [REBUILD] UNUSABLE LOCAL INDEXES
  | {READ ONLY | READ WRITE}
  | INDEXING {ON | OFF} };
```

+ {ADD | DROP} VALUES (list_values) : 리스트 파티션에 값을 추가하거나 삭제함
+ [REBUILD] UNUSABLE LOCAL INDEXES : 로컬 인덱스를 비활성화하거나 재구축함
+ {READ ONLY | READ WRITE} : 읽기 전용 또는 읽기 쓰기를 설정
+ INDEXING {ON | OFF} : 인덱싱 여부를 설정  

아래 쿼리는 p1 파티션을 읽기 전용 파티션으로 변경한다.  

```sql
ALTER TABLE t1 MODIFY PARTITION p1 READ ONLY;
```

읽기 전용 파티션에 DML 작업을 수행하면 에러가 발생한다.  

```sql
INSERT INTO t1 VALUES (1, 3);

ORA-14466: 읽기 전용 분할 영역이나 하위 분할 영역의 데이터는 수정할 수 없습니다.
```

#### 20.5.4. 관리 구문과 인덱스
<br/>
관리 구문을 수행하면 인덱스가 UNUSABLE 상태로 변경될 수 있다.  

&#42;&#95;INDEXES 뷰에서 t1&#95;x1 인덱스가 UNUSABLE 상태인 것을 확인할 수 있다.  

```sql
SELECT status FROM user_indexes WHERE index_name = 'T1_X1';
```

&#42;&#95;IND&#95;PARTITIONS 뷰에서 t1&#95;x2 인덱스의 파티션 중의 일부가 UNUSABLE 상태인 것을 확인할 수 있다.  

```sql
SELECT partition_name, status FROM user_ind_partitions WHERE index_name = 'T1_X2';
```

관리 구문은 아래와 같이 인덱스의 상태를 변경한다. 관리 구문에 UPDATE INDEX 절을 사용하면 인덱스를 바로 갱신한다.  

<table>
  <thead>
    <tr>
      <td>절</td>
      <td>글로벌</td>
      <td>로컬</td>
      <td>비고</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ADD</td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>DROP</td>
      <td>UNUSABLE</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>MERGE</td>
      <td>UNUSABLE</td>
      <td>UNUSABLE</td>
      <td></td>
    </tr>
    <tr>
      <td>SPLIT</td>
      <td>UNUSABLE</td>
      <td>UNUSABLE</td>
      <td>ONLINE 사용시 모두 갱신</td>
    </tr>
    <tr>
      <td>EXCHANGE</td>
      <td>UNUSABLE</td>
      <td>UNUSABLE</td>
      <td>INCLUDING INDEXES 사용시 로컬 인덱스 갱신</td>
    </tr>
    <tr>
      <td>MOVE</td>
      <td>UNUSABLE</td>
      <td>UNUSABLE</td>
      <td>ONLINE 사용시 모두 갱신</td>
    </tr>
    <tr>
      <td>RENAME</td>
      <td>TRUNCATE</td>
      <td>UNUSABLE</td>
      <td></td>
    </tr>
    <tr>
      <td>MODIFY</td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>

UPDATE INDEX 절을 사용하지 않고 아래와 같이 ALTER INDEX REBUILD 문을 직접 수행하는 것이 일반적이다.  

```sql
ALTER INDEX t1_x1 REBUILD;
ALTER INDEX t1_x2 REBUILD PARTITION p1;
ALTER INDEX t1_x2 REBUILD PARTITION p2;
ALTER INDEX t1_x2 REBUILD PARTITION p3;
```

#### 20.5.5. 신규 기능
<br/>
12.1 버전부터 파티션 인덱스의 신규 기능을 사용할 수 있다.  

##### 20.5.5.1. 비동기 글로벌 인덱스 관리
<br/>
12.1 버전부터 파티션을 삭제하거나 TRUNCATE하면 비동기 글로벌 인덱스 관리(asynchronous global index maintenance) 기능이 동작한다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER)
PARTITION BY RNAGE (c2) (
    PARTITION p1 VALUES LESS THAN (2)
  , PARTITION p2 VALUES LESS THAN (MAXVALUE));

CREATE INDEX t1_x1 ON t1 (c1);

INSERT INTO t1 VALUES (1, 1);
INSERT INTO t1 VALUES (2, 2);
COMMIT;
```

UPDATE INDEX 절을 사용하여 파티션을 TRUNCATE해보자.  

```sql
ALTER TABLE t1 TRUNCATE PARTITION p1 UPDATE GLOBAL INDEXES;
```

&#42;&#95;INDEXES 뷰에서 t1&#95;x1 인덱스의 status 열은 VALID, orphaned&#95;entries 열은 YES로 표시된 것을 확인할 수 있다. orphaned&#95;entries 열이 YES인 인덱스는 SYS.PMO&#95;DEFERRED&#95;GIDX&#95;MAINT&#95;JOB 작업을 통해 매일 2:00에 자동으로 정리(cleanup)된다.  

```sql
SELECT status, orphaned_entries FROM user_indexes WHERE index_name = 'T1_X1';
```

아래와 같이 DBMS&#95;SCHEDULER.RUN&#95;JOB 프로시저를 수행하면 orphaned&#95;entries 열이 YES인 인덱스를 수동으로 정리할 수도 있다.  

```sql
EXEC DBMS_SCHEDULER_RUN_JOB ('PMO_DEFERRED_GIDX_MAINT_JOB');
```

DBMS&#95;PART.CLEANUP&#95;GIDX 프로시저를 사용할 수도 있다. 해당 프로시저를 수행하면 테이블의 인덱스 중 orphaned&#95;entries 열이 YES인 인덱스가 정리된다.  

```sql
EXEC DBMS_PART.CLEANUP_GIDX('SCOTT', 'T1');
```

&#42;&#95;INDEXES 뷰를 다시 조회하면 orphaned entries 값이 NO로 변경된 것을 확인할 수 있다.  

```sql
SELECT status, orphaned_entries FROM user_indexes WHERE index_name = 'T1_X1';
```

아래의 ALTER INDEX 문으로도 인덱스를 정리할 수 있다. COALESCE CLEANUP 절은 인덱스 블록 내의 orphaned entries를 정리한다.  

```sql
ALTER INDEX t1_x1 REBUILD;
ALTER INDEX t1_x1 COALESCE CLEANUP;
```

##### 20.5.5.2. 부분 인덱스
<br/>
12.1 버전부터 부분 인덱스(partial indexes)를 생성할 수 있다. 부분 인덱스는 일부 테이블 파티션에 생성할 수 있는 인덱스다. 과거 파티션의 인덱스를 생성하지 않음으로써 저장 공간을 절약할 수 있다.  

예제를 위해 아래와 같이 테이블을 생성하자. p2 파티션은 INDEXING을 OFF로 설정했다.  

```sql
DROP TABLE t1 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER)
PARTITION BY RANGE (c2) (
    PARTITION p1 VALUES LESS THAN (2) INDEXING ON
  , PARTITION p2 VALUES LESS THAN (MAXVALUE) INDEXING OFF);

INSERT INTO t1 VALUES (1, 1);
INSERT INTO t1 VALUES (2, 2);
COMMIT;
```

아래와 같이 인덱스를 생성하자. t1&#95;x2 인덱스와 t1&#95;x4 인덱스는 부분 인덱스로 생성된다. 부분 인덱스는 INDEXING이 ON으로 설정된 파티션만 인덱싱한다. p2 파티션에 속한 데이터를 조회하는 경우 t1&#95;x2, t1&#95;x4 인덱스를 사용할 수 없다. 해당 기능은 옵티마이저의 Table Elimination 쿼리 변환에 의해 동작한다. p1 파티션에 속한 데이터를 조회하는 경우에는 t1&#95;x2, t1&#95;x4 인덱스를 사용할 수 있다.  

```sql
CREATE INDEX t1_x1 ON t1 (c1) GLOBAL INDEXING FULL;
CREATE INDEX t1_x2 ON t1 (c2) GLOBAL INDEXING PARTIAL;
CREATE INDEX t1_x3 ON t1 (c1, c2) LOCAL INDEXING FULL;
CREATE INDEX t1_x4 ON t1 (c2, c1) LOCAL INDEXING PARTIAL;
```

&#42;&#95;INDEXES 뷰의 indexing 열에서 INDEXING 유형을 확인할 수 있다.  

```sql
SELECT index_name, partitioned, indexing, status
FROM user_indexes
WHERE table_name = 'T1';
```

&#42;&#95;IND&#95;PARTITIONS 뷰에서 부분 인덱스로 생성된 t1&#95;x4 인덱스의 p2 파티션만 UNUSABLE 상태인 것을 확인할 수 있다.  

```sql
SELECT index_name, partition_name, status
FROM user_ind_partitions
WHERE index_name IN ('T1_X3', 'T1_X4');
```

##### 20.5.5.3. 온라인 전환
<br/>
ALTER TABLE 문을 사용하면 비 파티션 테이블을 파티션 테이블로 전환할 수 있다. 12.2 버전부터 온라인 전환(online conversion)을 사용할 수 있다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);
CREATE INDEX t1_x1 ON t1 (c1);
CREATE INDEX t1_x2 ON t1 (c2);

INSERT INTO t1 VALUES (1, 1);
INSERT INTO t1 VALUES (2, 2);
INSERT INTO t1 VALUES (3, 3);
COMMIT;
```

아래 쿼리는 t1 테이블을 파티션 테이블로 전환한다. t1&#95;x1 인덱스는 글로벌 인덱스, t1&#95;x2 인덱스는 로컬 인덱스로 전환된다.  

```sql
ALTER TABLE t1 MODIFY
PARTITION BY RANGE (c2) (
    PARTITION p1 VALUES LESS THAN (2)
  , PARTITION p2 VALUES LESS THAN (3)
  , PARTITION p3 VALUES LESS THAN (MAXVALUE))
ONLINE
UPDATE INDEXES (t1_x1 GLOBAL, t1_x2 LOCAL);
```

### 20.6. 뷰
<br/>
뷰(view)는 SELECT 문을 데이터베이스에 저장한 오브젝트다. 쿼리에서 테이블처럼 사용할 수 있다.  

#### 20.6.1. 기본 문법
<br/>
##### 20.6.1.1. CREATE VIEW 문
<br/>
CREATE VIEW 문은 뷰를 생성한다.  

```
CREATE [OR REPLACE] [[NO] FORCE] VIEW [schema.]view [(alias [, alias]...)]
AS subquery
[WITH {READ ONLY | CHECK OPTION} [CONSTRAINT constraint]];
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, CONSTRAINT t1_pk PRIMARY KEY (c1));

INSERT INTO t1 VALUES (1, 1);
INSERT INTO t1 VALUES (2, 2);
COMMIT;
```

아래 쿼리는 v1 뷰를 생성한다.  

```sql
CREATE OR REPLACE VIEW v1 AS SELECT * FROM t1 WHERE c2 = 1;
```

v1 뷰를 조회하면 c2가 1인 행만 반환된다.  

아래와 같이 v1 뷰에 데이터를 입력해보자.  

```sql
INSERT INTO v1 VALUES (3, 3);
INSERT INTO v1 VALUES (4, 1);
COMMIT;
```

입력한 데이터 중 c2가 1인 행만 조회된다.  

&#42;&#95;VIEWS 뷰에서 뷰에 대한 정보를 조회할 수 있다.  

```sql
SELECT text_length, text_vc FROM user_view WHERE view_name = 'V1';
```

&#42;&#95;TAB&#95;COLUMNS 뷰에서 뷰 열에 대한 정보를 조회할 수 있다.  

```sql
SELECT column_name, data_type, nullable FROM user_tab_columns WHERE table_name = 'V1';
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t2 PURGE;
DROP TABLE t3 PURGE;

CREATE TABLE t2 (c1 NUMBER, CONSTRAINT t2_pk PRIMARY KEY (c1));
CREATE TABLE t3 (c1 NUMBER, c2 NUMBER, CONSTRAINT t3_pk PRIMARY KEY (c1, c2));

INSERT INTO t2 VALUES (1);
INSERT INTO t3 VALUES (1, 1);
INSERT INTO t3 VALUES (1, 2);
COMMIT;
```

아래 쿼리는 v2 뷰를 생성한다. 서브 쿼리에서 t2, t3 테이블을 조인했다.  

```sql
CREATE OR REPLACE VIEW v2 AS
SELECT a.c1 AS ac1, b.c1 AS bc1, b.c2 AS bc2 FROM t2 a, t3 b WHERE b.c1 = a.c1;
```

v2 뷰를 조회하면 조인에 성공한 행만 반환된다.  

아래와 같이 v2 뷰에 데이터를 입력하면 에러가 발생한다. t2, t3 테이블의 조인 차수가 1:M이기 때문에 t2 테이블에 데이터를 입력할 수 없다.  

```sql
INSERT INTO v2 VALUES (3, 3, 3);

ORA-01779: 키-보존된것이 아닌 테이블로 대응한 열을 수정할 수 없습니다.
```

아래와 같이 데이터를 입력해보자. t3 테이블에만 데이터가 입력되기 때문에 에러가 발생하지 않는다.  

```sql
INSERT INTO v2 (c1b, c2b) VALUES (3, 3);
```

&#42;&#95;UPDATABLE&#95;COLUMNS 뷰에서 열에 대한 DML 가능 여부를 조회할 수 있다.  

```sql
SELECT column_name, updatable, insertable, deletable
FROM user_updatable_columns
WHERE table_name = 'V2';
```

###### 20.6.1.1.2. WITH CHECK OPTION
<br/>
WITH CHECK OPTION은 서브 쿼리의 WHERE 절에 해당하지 않은 데이터가 생성되는 것을 방지한다.  

아래 쿼리는 c1이 4인 행의 c2 열을 4로 갱신한다.  

```sql
UPDATE v1 SET c2 = 4 WHERE c1 = 4;
```

v1 뷰를 조회해보면 c2 = 1 조건에 의해 갱신된 행이 조회되지 않는다.  

아래와 같이 WITH CHECK OPTION으로 뷰를 생성해보자.  

```sql
CREATE OR REPLACE VIEW v1 AS SELECT * FROM t1 WHERE c2 = 1 WITH CHECK OPTION;
```

서브 쿼리의 WHERE 절에 위배되는 갱신을 수행하면 에러가 발생한다.  

```sql
UPDATE v1 SET c2 = 0 WHERE c1 = 1;

ORA-01402: 뷰의 WITH CHECK OPTION의 조건에 위배됩니다.
```
INSERT 문 역시 에러가 발생한다.  

```sql
INSERT INTO v1 VALUES (5, 5);

ORA-01402: 뷰의 WITH CHECK OPTION의 조건에 위배 됩니다.
```

&#42;&#95;CONSTRAINTS 뷰에서 뷰의 제약 조건을 조회할 수 있다. constraint&#95;type 열이 V로 표시된다.  

```sql
SELECT constraint_name, constraint_type FROM user_constraints WHERE table_name = 'V1';
```

인라인 뷰에도 WITH CHECK OPTION을 사용할 수 있다. 아래 쿼리는 UPDATE 문의 인라인 뷰에 WITH CHECK OPTION을 사용했다.  

```sql
UPDATE (SELECT * FROM t1 WHERE c2 = 1 WITH CHECK OPTION) SET c2 = 0;

ORA-01402: 뷰의 WITH CHECK OPTION의 조건에 위배됩니다
```

아래 쿼리는 INSERT 문의 인라인 뷰에 WITH CHECK OPTION을 사용했다.  

```sql
INSERT INTO (SELECT * FROM t1 WHERE c2 = 1 WITH CHECK OPTION) VALUES (5, 5);

ORA-01402: 뷰의 WITH CHECK OPTION의 조건에 위배됩니다
```

###### 20.6.1.1.3. WITH READ ONLY
<br/>
WITH READ ONLY를 기술하면 뷰가 읽기 전용으로 생성된다.  

아래와 같이 WITH READ ONLY로 v1 뷰를 생성하자.  

```sql
CREATE OR REPLACE VIEW v1 AS SELECT * FROM t1 WHERE c2 = 1 WITH READ ONLY;
```

v1 뷰에 DML 작업을 수행하면 에러가 발생한다.  

```sql
DELETE FROM v1;

ORA-42399: 읽기 전용 뷰에서는 DML 작업을 수행할 수 없습니다.  
```

&#42;&#95;CONSTRAINTS 뷰의 constraint_type 열이 O로 표시된다.  

```sql
SELECT constraint_name, constraint_type FROM user_constraints WHERE table_name = 'V1';
```

&#42;&#95;VIEWS 뷰의 read&#95;only 열에서도 읽기 전용 여부를 확인할 수 있다.  

```sql
SELECT read_only FROM user_views WHERE view_name = 'V1';
```

###### 20.6.1.1.4. 오브젝트 종속성
<br/>
오브젝트는 다른 오브젝트와 종속성을 가질 수 있다. 뷰의 경우 서브 쿼리에서 사용하고 있는 테이블과 종속 관계를 가진다.  

아래 &#42;&#95;DEPENDENCIES 뷰에서 오브젝트 종속성에 대한 정보를 조회할 수 있다.  

```sql
SELECT name, type, referenced_owner, referenced_name, referenced_type
FROM user_dependencies
WHERE name in ('V1', 'V2');
```

##### 20.6.1.2. ALTER VIEW 문
<br/>
ALTER VIEW 문은 뷰를 컴파일 한다.  

```
ALTER VIEW [schema.]view COMPILE;
```

예제를 위해 c2 열의 열명을 c3로 변경하자.  

```sql
ALTER TABLE t1 RENAME COLUMN c2 TO c3;
```

뷰의 서브 쿼리에서 t1 테이블의 c2 열을 참조하기 때문에 v1 뷰를 조회하면 에러가 발생한다.  

무효화된 오브젝트는 &#42;&#95;OBJECTS 뷰의 status 열이 INVALID로 표시된다.  

```sql
SELECT object_type, status FROM user_objects WHERE object_name = 'V1';
```

c3 열의 열명을 c2로 변경한 후 v1 뷰를 컴파일해보자.  

```sql
ALTER TABLE t1 RENAME COLUMN c3 TO c2;
ALTER VIEW v1 COMPILE;
```

&#42;&#95;OBJECTS 뷰의 status 열이 VALID로 변경된 것을 확인할 수 있다.  

```sql
SELECT object_type, status FROM user_objects WHERE object_name = 'V1';
```

###### 20.6.1.2.1. DBMS&#95;UTILITY.COMPILE&#95;SCHEMA 프로시저
<br/>
DBMS&#95;UTILITY.COMPILE&#95;SCHEMA 프로시저는 지정한 schema의 컴파일 가능한 오브젝트를 컴파일한다. compile&#95;all를 false로 지정하면 무효화된 오브젝트만 컴파일한다.  

```sql
DBMS_UTILITY.COMPILE_SCHEMA (
    schema IN VARCHAR2
  , compile_all IN BOOLEAN DEFAULT TRUE
  , reuse_settings IN BOOLEAN DEFAULT FALSE);
```

아래 예제는 SCOTT 스키마의 컴파일 가능한 모든 오브젝트를 컴파일한다.  

```sql
EXEC DBMS_UTILITY.COMPILE_SCHEMA (schema => 'SCOTT');
```

DBMS&#95;UTILITY.COMPILE&#95;SCHEMA 프로시저 대신 UTL&#95;RECOMP 패키지를 사용하면 무효화된 오브젝트에 대한 상세한 처리가 가능하다.  

##### 20.6.1.3. DROP VIEW 문
<br/>
DROP VIEW 문은 뷰를 삭제한다.  

```
DROP VIEW [schema.]view;
```

아래 쿼리는 v1 뷰를 삭제한다.  

```sql
DROP VIEW v1;
```

#### 20.6.2. 활용 예제
<br/>
뷰의 활용 예제를 살펴보자.  

##### 20.6.2.1. 데이터 보안성
<br/>
뷰를 사용하면 데이터 보안성(data security)을 높일 수 있다.  

예제를 위해 아래와 같이 뷰를 생성하자. job의 CLERK인 행의 empno, ename, hiredate 열을 조회할 수 있는 뷰다.  

```sql
CREATE OR REPLACE VIEW v_emp AS
SELECT empno, ename, hiredate FROM emp WHERE job = 'CLERK' WITH CHECK OPTION;
```

v&#95;emp 뷰에 대한 권한을 부여받은 사용자는 뷰의 열과 조건에 해당하는 데이터만 조회할 수 있으며, DML 작업도 해당 데이터에 대해서만 수행할 수 있다.  

##### 20.6.2.2. 데이터 독립성
<br/>
뷰를 사용하면 데이터 독립성(data independence)을 높일 수 있다.  

예제를 위해 아래와 같이 뷰를 생성하자. 부서별 sal의 합계 값을 조회할 수 있는 뷰다.  

```sql
CREATE OR REPLACE VIEW v_dept AS
SELECT a.deptno AS dept_no, a.dname AS dept_nm, b.sal
FROM dept a
   , (SELECT deptno, SUM (sal) AS sal FROM emp GROUP BY deptno) b
WHERE b.deptno = a.deptno
WITH READ ONLY;
```

다수의 애플리케이션에서 뷰를 사용한다고 가정해보자. 테이블이나 쿼리가 변경되는 경우 애플리케이션 수정없이 뷰만 수정하면 변경 사항을 쉽게 반영할 수 있다. 쿼리를 간결하게 작성할 수 있는 부가적인 장점도 있다.  

아래와 같이 뷰를 변경해보자. dname의 열명이 loc로 변경되었다고 가정하자. 조인도 아우터 조인으로 변경했다.  

```sql
CREATE OR REPLACE VIEW v_dept AS
SELECT a.deptno AS dept_no, a.loc AS dept_nm, b.sal
FROM dept a
   , (SELECT deptno, SUM (sal) AS sal FROM emp GROUP BY deptno) b
WHERE b.deptno(+) = a.deptno
WITH READ ONLY;
```

v&#95;dept 뷰를 다시 조회해보자. 결과가 변경된 것을 확인할 수 있다. 애플리케이션 수정없이 변경 사항을 반영할 수 있는 것이다.  

### 20.7. 시퀀스
<br/>
시퀀스는 정수 순번 값을 생성하는 오브젝트다.  

#### 20.7.1. 기본 문법
<br/>
##### 20.7.1.1. CREATE SEQUENCE 문
<br/>
CREATE SEQUENCE 문은 시퀀스를 생성한다.  

```
CREATE SEQUENCE [schema.]sequence
  [ {INCREMENT BY | START WITH} integer]
  | {MAXVALUE integer | NOMAXVALUE}
  | {MINVALUE integer | NOMINVALUE}
  | {CYCLE | NOCYCLE}
  | {CACHE integer | NOCACHE}
  | {ORDER | NOORDER}
  | {SESSION | GLOBAL}];
```

+ INCREMENT BY integer : 시퀀스 사이의 간격 (기본값은 1)
+ START WITH integer : 시퀀스의 시작 번호 (기본값은 1)
+ MAXVALUE integer : 시퀀스가 생성할 수 있는 최대 값
+ NOMAXVALUE : 시퀀스가 오름차순이면 1027, 내림차순이면 -1 (기본값)
+ MINVALUE integer : 시퀀스가 생성할 수 있는 최소 값
+ NOMINVALUE : 시퀀스가 오름차순이면 1, 내림차순이면 -1026 (기본값)
+ CYCLE : 최대 값이나 최소 값에 도달했을 때 순환해서 생성
+ NOCYCLE : 최대 값이나 최소 값에 도달했을 때 에러를 발생 (기본값)
+ CACHE integer | NOCACHE : 미리 할당해서 메모리에 저장하는 값의 개수 (기본값은 20)
+ ORDER | NOORDER : RAC 환경에서 순서를 보장하거나 보장하지 않음
+ SESSION | GLOBAL : 세션 또는 글로벌 시퀀스로 생성  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER NOT NULL, c2 NUMBER, CONSTRAINT t1_p1 PRIMARY KEY (c1));
```

아래 쿼리는 s1 시퀀스를 생성한다.  

```sql
DROP SEQUENCE s1;
CREATE SEQUENCE s1;
```

시퀀스 값은 아래의 슈도 칼럼으로 조회할 수 있다.  

+ NEXTVAL : 사용 가능한 다음 시퀀스 값을 반환
+ CURRVAL : 현재 시퀀스 값을 반환  

아래 쿼리는 에러가 발생한다.  

```sql
SELECT s1.CURRVAL AS c1 FROM DUAL;
```

CURRVAL 슈도 칼럼은 NEXTVAL 슈도 칼럼을 사용한 후 사용해야 한다.  

```sql
SELECT s1.NEXTVAL AS c1 FROM DUAL;
```

CURRVAL 슈도 칼럼을 다시 조회하면 에러가 발생하지 않는다.  

```sql
SELECT s1.CURRVAL AS c1 FROM DUAL;
```

아래와 같이 데이터를 입력해보자. 시퀀스는 주로 PK 값을 생성하기 위해 사용한다.  

```sql
INSERT INTO t1 VALUES (s1.NEXTVAL, 1);
INSERT INTO t1 VALUES (s1.NEXTVAL, 2);
COMMIT;
```

&#42;&#95;SEQUENCES 뷰에서 시퀀스에 대한 정브를 조회할 수 있다.  

```sql
SELECT min_value, max_value, increment_by, cache_size, last_number
FROM user_sequences
WHERE sequence_name = 'S1';
```

##### 20.7.1.2. ALTER SEQUENCE 문
<br/>
ALTER SEQUENCE 문은 시퀀스를 변경한다.  

```
ALTER SEQUENCE [schema.]sequence
  ( INCREMENT BY integer
  | {MAXVALUE integer | NOMAXVALUE}
  | {MINVALUE integer | NOMINVALUE}
  | {CYCLE | NOCYCLE}
  | {CACHE integer | NOCACHE}
  | {SESSION | GLOBAL} );
```

아래 쿼리는 시퀀스의 값을 101으로 변경한다.  

```sql
SELECT s1.NEXTVAL AS c1 FROM DUAL;

ALTER SEQUENCE s1 INCREMENT BY 96;

SELECT s1.NEXTVAL AS c1 FROM DUAL;

ALTER SEQUENCE s1 INCREMENT BY 1;
```

아래 쿼리는 시퀀스의 값을 초기화한다.  

```sql
SELECT s1.NEXTVAL AS c1 FROM DAUL;

ALTER SEQUENCE s1 INCREMENT BY -101 MINVALUE 0;

SELECT s1.NEXTVAL AS c1 FROM DUAL;

ALTER SEQUENCE s1 INCREMENT BY 1;
```

##### 20.7.1.3. DROP SEQUENCE 문
<br/>
DROP SEQUENCE 문 시퀀스를 삭제한다.  

```
DROP SEQUENCE [schema.]sequence_name;
```

아래 쿼리는 s1 시퀀스를 삭제한다.  

```sql
DROP SEQUENCE s1;
```

#### 20.7.2. 시퀀스 유형
<br/>
##### 20.7.2.1. 전역 시퀀스
<br/>
전역 시퀀스(global sequence)는 시스템 레벨의 시퀀스이다. 전역 시퀀스는 시스템에 종속되며 여러 세션에서 공유할 수 있다.  

예제를 위해 아래와 같이 전역 시퀀스를 생성하자.  

```sql
DROP SEQUENCE s1;
CREATE SEQUENCE s1;
```

아래 쿼리를 2개의 세션에서 순서대로 수행해보자. 5번 줄에서 NEXTVAL 값이 증가한 것을 확인할 수 있다. 전역 시퀀스는 다른 세션의 영향을 받는다.  

```sql
--- S1
SELECT s1.NEXTVAL AS c1 FROM DUAL;

--- S2
SELECT s1.NEXTVAL AS c1 FROM DUAL;

--- S1
SELECT s1.CURRVAL AS c1 FROM DUAL;

--- S2
SELECT s1.CURRVAL AS c1 FROM DUAL;

--- S1
SELECT s1.NEXTVAL AS c1 FROM DUAL;

--- S2
SELECT s1.NEXTVAL AS c1 FROM DUAL;
```

##### 20.7.2.2. 세션 시퀀스
<br/>
세션 시퀀스(session sequence)는 레벨의 시퀀스이다. 세션 시퀀스는 세션에 종속되며 세션이 종료되면 초기화된다.  

예제를 위해 아래와 같이 세션 시퀀스를 생성하자.  

```sql
DROP SEQUENCE s1;
CREATE SEQUENCE s1 SESSION;
```

아래 쿼리를 2개의 세션에서 순서대로 수행해보자. 세션 시퀀스는 다른 세션의 영향을 받지 않는다.  

```sql
--- S1
SELECT s1.NEXTVAL AS c1 FROM DUAL;

--- S2
SELECT s1.NEXTVAL AS c1 FROM DUAL;

--- S1
SELECT s1.CURRVAL AS c1 FROM DUAL;

--- S2
SELECT s1.CURRVAL AS c1 FROM DUAL;

--- S1
SELECT s1.NEXTVAL AS c1 FROM DUAL;
```

#### 20.7.3. 신규 기능
<br/>
12.1 버전부터 시퀀스의 신규 기능을 사용할 수 있다. 기본값과 IDENTITY 절을 차례대로 살펴보자. 18.1 버전에 right growing 인덱스의 블록 경합을 해소할 수 있는 SCALABLE 시퀀스 기능이 추가되었습니다.  

##### 20.7.3.1. 기본값
<br/>
12.1 버전부터 시퀀스를 열의 기본값으로 사용할 수 있다.  

예제를 위해 아래와 같이 시퀀스와 테이블을 생성하자. c3 열은 DEFAULT ON NULL로 기본값을 지정했다.  

```sql
DROP SEQUENCE s1;
DROP SEQUENCE s2;

CREATE SEQUENCE s1;
CREATE SEQUENCE s2;

DROP TABLE t1 PURGE;

CREATE TABLE t1 (
    c1 NUMBER
  , c2 NUMBER DEFAULT scott.s1 NEXTVAL
  , c3 NUMBER DEFAULT ON NULL scott.s2 NEXTVAL);
```

아래와 같이 데이터를 입력해보자.  

```sql
INSERT INTO t1 (c1) VALUES (1);
INSERT INTO t1 VALUES (2, 9, 9);
INSERT INTO t1 VALUES (3, NULL, NULL);
INSERT INTO t1 VALUES (4, DEFAULT, DEFAULT);
COMMIT;
```

아래는 t1 테이블을 조회한 결과다. 널을 입력한 c1이 3인 c3 열에 시퀀스 값이 입력된 것을 확인할 수 있다.  

```sql
SELECT * FROM t1;
```

아래 쿼리는 관계를 가진 테이블에 시퀀스를 설정한다.  

```sql
DROP SEQUENCE s1;
DROP SEQUENCE s2;

CREATE SEQUENCE s1;
CREATE SEQUENCE s2;

DROP TABLE t1 PURGE;
DROP TABLE t2 PURGE;

CREATE TABLE t1 (
    c1 NUMBER DEFAULT scott.s1 CURRVAL NOT NULL
  , c2 NUMBER DEFAULT scott.s2 NEXTVAL NOT NULL
  , c3 NUMBER);

ALTER TABLE t1 ADD CONSTRAINT t1_pk PRIMARY KEY (c1);
ALTER TABLE t2 ADD CONSTRAINT t2_pk PRIMARY KEY (c1, c2);
ALTER TABLE t2 ADD CONSTRAINT t2_f1 PRIMARY KEY (c1) REFERENCES t1 (c1);
```

아래와 같이 데이터를 입력해보자.  

```sql
INSERT INTO t1 (c2) VALUES (1);
INSERT INTO t2 (c3) VALUES (2);
INSERT INTO t2 (c3) VALUES (3);
INSERT INTO t1 (c2) VALUES (4);
INSERT INTO t2 (c3) VALUES (5);
INSERT INTO t2 (c3) VALUES (6);
COMMIT;
```

##### 20.7.3.2. IDENTITY 절
<br/>
IDENTITY 절은 IDENTITY 열을 생성한다. IDENTITY 열은 시퀀스가 결합된 열이다.  

```
GENERATED [ALWAYS | BY DEFAULT [ON NULL]] AS IDENTITY [(
  {START WITH (integer | LIMIT VALUE)
  | INCREMENT BY integer
  | (MAXVALUE integer | NOMAXVALUE)
  | (MINVALUE integer | NOMINVALUE)
  | (CYCLE | NOCYCLE)
  | (CACHE integer | NOCACHE)
  | (ORDER | NOORDER) })];
```

+ ALWAYS : 시스템만 시퀀스를 할당 (기본값)
+ BY DEFAULT : 시스템이 시퀀스를 할당하지만 직접 값을 입력할 수도 있음
+ LIMIT VALUE : 시퀀스가 오름차순이면 최대 값, 내림차순이면 최소 값으로 시퀀스를 설정  

아래 쿼리는 t1 테이블의 c2 열을 IDENTITY 열로 생성한다. IDENTITY 열은 테이블에 하나만 생성할 수 있다.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER GENERATED ALWAYS AS IDENTITY);
```

아래와 같이 테이터를 입력해보자. 두 번째, 세 번째 쿼리는 에러가 발생한다.  

```sql
INSERT INTO t1 (c1) VALUES (1);
INSERT INTO t1 VALUES (2, 9);

ORA-32795: generated always ID 열에 삽입할 수 없습니다.  

INSERT INTO t1 VALUES (3, NULL);

ORA-32795: generated always ID 열에 삽입할 수 없습니다.  

INSERT INTO t1 VALUES (4, DEFAULT);
```

아래는 t1 테이블을 조회한 결과다. c1이 4인 행의 c2 값이 2다. 에러가 발생해도 IDENTITY 열의 시퀀스 값이 증가하지 않는 것을 확인할 수 있다.  

&#42;&#95;TAB&#95;IDENTITY&#95;COLS 뷰에서 IDENTITY 열에 대한 정보를 조회할 수 있다.  

```sql
SELECT * FROM user_tab_identity_cols WHERE table_name = 'T1';
```

&#42;&#95;TAB&#95;IDENTITY&#95;COLS 뷰에서 IDENTITY 열에 대한 정보를 조회할 수 있다.  

```sql
SELECT * FROM user_tab_identity_cols WHERE table_name = 'T1';
```

&#42;&#95;TAB&#95;COLUMNS 뷰의 identity&#95;column 열에서도 IDENTITY 열 여부를 조회할 수 있다. nullable 열은 N, data&#95;default 열은 내부적으로 사용하는 시퀀스가 반환된다. IDENTITY 열을 생성하면 시퀀스가 자동으로 생성된다.  

&#42;&#95;SEQUENCES 뷰에서 IDENTITY 열이 사용하는 시퀀스를 조회할 수 있다.  

```sql
SELECT min_value, max_value, increment_by, cache_size, last_number
FROM user_sequences
WHERE sequence_name = 'ISEQ$$######';
```

아래 쿼리는 c2 열의 시퀀스를 c2의 최대값으로 변경한다.  

```sql
ALTER TABLE t1 MODIFY c2 GENERATED ALWAYS AS IDENTITY (START WITH LIMIT VALUE);
```

아래 쿼리는 IDENTITY 열을 일반 열로 변경한다.  

```sql
ALTER TABLE t1 MODIFY c2 DROP IDENTITY;
```

#### 20.7.4. 활용 예제
<br/>
시퀀스의 활용 예제를 살펴보자.  

##### 20.7.4.1. 시퀀스와 다중 테이블 INSERT 문
<br/>
다중 테이블 INSERT 문에 시퀀스를 사용해보자.  

예제를 위해 아래와 같이 테이블과 시퀀스를 생성하자.  

```sql
DROP TABLE t1 PURGE;
DROP TABLE t2 PURGE;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);
CREATE TABLE t2 (c1 NUMBER, c2 NUMBER);

DROP SEQUENCE s1;
CREATE SEQUENCE s1;
```

아랭 쿼리는 에러가 발생한다. 다중 테이블 INSERT 문은 VALUES 절에 시퀀스를 기술해야 한다.  

```sql
INSERT ALL
INTO t1
INTO t2
SELECT 1, s1.NEXTVAL FROM DUAL CONNECT BY LEVEL <= 2;

ORA-02287: 시퀀스 번호는 이 위치에 사용할 수 없습니다.  
```

아래와 같이 쿼리를 수행해보자.  

```sql
INSERT ALL
  INTO t1 VALUES (1, s1.NEXTVAL)
  INTO t2 VALUES (2, s1.NEXTVAL)
SELECT * FROM DUAL CONNECT BY LEVEL <= 2;

INSERT ALL
  INTO t1 VALUES (2, s1.NEXTVAL)
  INTO t2 VALUES (2, s1.CURRVAL)
SELECT * FROM DUAL CONNECT BY LEVEL <= 2;

INSERT ALL
  INTO t1 VALUES (3, s1.CURRVAL)
  INTO t2 VALUES (3, s1.NEXTVAL)
SELECT * FROM DUAL CONNECT BY LEVEL <= 2;

INSERT ALL
  INTO t1 VALUES (4, s1.CURRVAL)
  INTO t2 VALUES (4, s1.CURRVAL)
SELECT * FROM DUAL CONNECT BY LEVEL <= 2;

COMMIT;
```

아래는 t1, t2 테이블을 조회한 결과다. CURRVAL 슈도 칼럼만 기술한 마지막 쿼리(c1 = 4) 외에는 모두 순차 값이 입력된 것을 확인할 수 있다. 구문이 보호하므로 NEXTVAL 슈도 칼럼만 사용하는 편이 바람직하다.  

```sql
SELECT * FROM t1;
SELECT * FROM t2;
```

사용자 정의 함수를 사용하면 쿼리를 간결하게 작성할 수 있다. 예제를 위해 아래와 같이 사용자 정의 함수를 생성하자.  

```sql
CREATE OR REPLACE FUNCTION f1 RETURN NUMBER
IS
BEGIN
  RETURN s1.NEXTVAL;
END f1;
/
```

쿼리를 수행하기 전에 t1 테이블과 t2 테이블을 TRUNCATE하자.  

```sql
TRUNCATE TABLE t1;
TRUNCATE TABLE t2;
```

사용자 정의 함수는 SELECT 절에 기술할 수 있다.  

```sql
INSERT ALL
  INTO t1
  INTO t2
SELECT 5, f1 FROM DUAL CONNECT BY LEVEL <= 2;
```

t1, t2 테이블에 순차 값이 입력된 것을 확인할 수 있다. 사용자 정의 함수를 사용하면 쿼리는 간결하게 작성할 수 있지만 문맥 전환으로 인한 성능 저학 발생할 수 있다.  

```sql
SELECT * FROM t1;
SELECT * FROM t2;
```

### 20.8. 시너님
<br/>
시너님(synonym)은 데이터베이스 오브젝트에 동의어를 부여할 수 있는 오브젝트다. 시너님을 사용하면 긴 이름의 오브젝트를 짧은 이름으로 참조하거나, 다른 소유자의 오브젝트를 스키마를 기술하지 않고 참조할 수 있다.  

#### 20.8.1. 기본 문법
<br/>
##### 20.8.1.1. CREATE SYNONYM 문
<br/>
CREATE SYNONYM 문은 시너님을 생성한다. PUBLIC을 기술하면 공용 시너님, 기술하지 않으면 전용 시너님이 생성된다.  

```
CREATE [OR REPLACE] [PUBLIC] SYNONYM [schema.]synonym FOR [schema.]object;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
DROP TABLE t2 PURGE;
DROP TABLE t3 PURGE;

CREATE TABLE t2 AS SELECT 2 AS c1 FROM DUAL;
CREATE TABLE t3 AS SELECT 3 AS c1 FROM DUAL;
```

아래 쿼리는 t2 테이블에 대한 t1 시너님을 생성한다.  

```sql
CREATE OR REPLACE SYNONYM t1 FOR t2;
```

t1 시너님을 조회하면 t2 테이블이 조회된다.  

```sql
SELECT * FROM t1;
```

아래 쿼리는 t3 테이블에 대한 t1 시너님을 생성한다.  

```sql
CREATE OR REPLACE SYNONYM t1 FOR t3;
```

t1 시너님을 조회하면 t3 테이블이 조회된다. 테이블에 대한 시너님을 사용하면 테이블을 계층화(layering)할 수 있다.  

```sql
SELECT * FROM t1;
```

&#42;&#95;SYNONYMS 뷰에서 시너님에 대한 정보를 조회할 수 있다.  

```sql
SELECT synonym_name, table_owner, table_name
FROM user_synonyms
WHERE synonym_name = 'T1';
```

전용 시너님과 동일한 이름의 오브젝트를 생성하면 에러가 발생한다.  

```sql
CREATE TABLE t1 (c1 NUMBER);

ORA-00955: 기존의 객체가 이름을 사용하고 있습니다.
```

##### 20.8.1.2. ALTER SYNONYM 문
<br/>
ALTER SYNONYM 문은 시너님을 컴파일한다.  

```
ALTER [PUBLIC] SYNONYM [schema.]synonym COMPILE;
```

아래 쿼리는 t1 시너님을 컴파일한다.  

```sql
ALTER SYNONYM t1 COMPILE;
```

##### 20.8.1.3. DROP SYNONYM 문
<br/>
DROP SYNONYM 문은 시너님을 삭제한다. FORCE 옵션을 사용하면 시너님을 참조하고 있는 오브젝트가 존재하더라도 시너님을 삭제할 수 있다.  

```
DROP [PUBLIC] SYNONYM [schema.]synonym [FORCE];
```

아래 쿼리는 t1 시너님을 삭제한다.  

```sql
DROP SYNONYM t1;
```

#### 20.8.2. 시너님 유형
<br/>
##### 20.8.2.1. 전용 시너님
<br/>
전용 시너님(private synonym)은 특정 사용자에 종속되 시너님이다. 기본 문법에서 살펴본 시너님이 전용 시너님이다.  

##### 20.8.2.2. 공용 시너님
<br/>
공용 시너님(public synonym)은 모든 사용자가 사용할 수 있는 시너님이다. 공용 시너님은 스키마를 기술하지 않고 사용할 수 있다.  

&#42;&#95;SYNONYMS 뷰에서 DUAL 테이블을 조회해보자. DUAL 테이블은 SYS 사용자가 소유하고 있는 테이블이다. 공용 시서님이 생성되어 있기 때문에 스키마를 기술하지 않고 사용할 수 있는 것이다.  

```sql
SELECT owner, synonym_name, table_onwer, table_name
FROM all_synonyms
WHERE table_name = 'DUAL';
```

SCOTT 사용자에 dual 테이블을 사용해보자.  

```sql
CREATE TABLE dual (c1 NUMBER);
```

dual 테이블을 조회하면 결과가 반환되지 않는다. 쿼리의 구문은 해석할 때 공용 시너님보다 소유자 오브젝트의 우선순위가 높기 때문이다. 소유자 오브젝트, 전용 시너님, 공용 시너님 순서로 구문을 해석한다.  

```sql
SELECT * FROM dual;
```

SCOTT 사용자에 생성한 dual 테이블을 삭제하자.  

```sql
DROP TABLE dual PURGE;
```

### 20.9. 데이터베이스 링크
<br/>
데이터베이스 링크(database link)는 원격 데이터베이스의 오브젝트를 액세스할 수 있는 오브젝트다. 데이터베이스 링크를 줄여서 DB 링크라고 부르기도 한다. 주로 분산 데이터베이스(distributed database) 환경에서 사용된다.  

#### 20.9.1. 기본 문법
<br/>
##### 20.9.1.1. CREATE DATABASE LINK 문
<br/>
CREATE DATABASE LINK 문은 DB 링크를 생성한다. PUBLIC을 기술하면 공용 DB 링크, 기술하지 않으면 전용 DB 링크가 생성된다. connect&#95;string은 접속 문자열이나 tnsnames.ora 파일에 기술된 TNS명을 기술할 수 있다.  

```
CREATE [PUBLIC] DATABASE LINK dblink
  [CONNECT TO user IDENTIFIED BY password]
  [USING connect_string];
```

DB 링크는 사용자(user) 지정 여부에 따라 아래의 두 가지 방식으로 생성할 수 있다. 접속 사용자 방식은 원격 데이터베이스에 현재 접속한 사용자와 동일한 계정이 존재해야 한다.  

+ 접속 사용자 빙식 : 현재 접속한 사용자와 동일한 사용자와 패스워드로 접속
+ 지정 사용자 방식 : 지정한 사용자와 패스워드로 접속  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 AS SELECT 1 AS c1 FROM DUAL;
```

아래 쿼리로 DB 링크를 생성할 수 있다. dl1 DB 링크는 접속 사용자 방식, dl2 DB 링크는 지정 사용자 방식으로 생성했다. 예제를 위해 dl2 DB 링크의 패스워드를 틀리게 지정했다.  

```sql
DROP DATABASE LINK dl1;
DROP DATABASE LINK dl2;

CREATE DATABASE LINK dl1 USING 'ora12cr2';

CREATE DATABASE LINK dl2 CONNECT TO scott IDENTIFIED BY lion USING 'ora12cr2';
```

아래와 같이 connect&#95;string에 접속 문자열을 직접 기술할 수도 있다.  

```sql
CREATE DATABASE LINK d1 USING
'(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=127.0.0.1)(PORT=1521))
              (CONNECT_DATA=(SERVICE_NAME=ORA12CR2)))';
```

아래 쿼리는 dl1 DB 링크를 통해 t1 테이블을 조회한다. DB 링크는 테이블, 뷰, 시퀀스, PL/SQL 오브젝트 등의 뒤쪽에 @dblink 형식으로 사용할 수 있다.  

```sql
SELECT * FROM t1@dl1;
```

아래 쿼리는 에러가 발생한다. dl2 DB 링크의 패스워드가 틀렸기 때문이다.  

```sql
SELECT * FROM t1@dl2;

ORA-01017: 사용자명/비밀번호가 부적합, 로그온할 수 없습니다.  
ORA-02063: line가 선행됨 (DL2로 부터)
```

ALL&#95;DB&#95;LINKS 뷰에서 현재 사용자가 사용할 수 있는 DB 링크에 대한 정보를 조회할 수 있다.  

```sql
SELECT owner, db_link, username, host FROM all_db_links WHERE db_link_ IN 'DL_';
```

##### 20.9.1.2. ALTER DATABASE LINK 문
<br/>
ALTER DATABASE LINK 문은 데이터베이스 링크를 변경한다.  

```
ALTER [PUBLIC] DATABASE LINK dblink
  [CONNECT TO user IDENTIFIED BY password]
  [USING connect_string];
```

아래 쿼리는 dl2 DB 링크의 패스워드를 tiger로 변경한다.  

```sql
ALTER DATABASE LINK dl2 CONNECT TO scott IDENTIFIED BY tiger;
```

dl2 DB 링크를 다시 사용하면 에러가 발생하지 않는다.  

```sql
SELECT * FROM t1@d12;
```

##### 20.9.1.3. DROP DATABASE LINK 문
<br/>
DROP DATABASE LINK 문은 데이터베이스 링크를 삭제한다.  

아래 쿼리는 dl2 DB 링크를 삭제한다.  

```sql
DROP DATABASE LINK dl2;
```

#### 20.9.2. 고급 주제
<br/>
##### 20.9.2.1. 세션 관리
<br/>
DB 링크를 사용하면 원격 데이터베이스에 새로운 세션이 생성된다. DB 링크를 사용한 세션이 종료될 때까지 원격 데이터베이스이 세션이 종료되징 않는다. 원격 데이터베이스에 과도한 세션이 생성될 수 있다. 커넥션 풀을 사용한 multitier 구조의 애플리케이션에서 특히 문제가 될 수 있다.  

아래와 같이 dl1 DB 링크로 t1 테이블을 조회해보자.  

```sql
SELECT * FROM t1@dl1;
```

V$DBLINK 뷰에서 현재 세션의 열린 DB 링크에 대한 정보를 조회할 수 있다. in&#95;transaction 열이 YES로 표시된다.  

```sql
SELECT db_link, logged_on, open_cursors, in_transaction FROM v$dblink;
```

in&#95;transaction 열이 YES로 표시된 DB 링크가 존재하면 커밋이나 롤백을 수행해야 한다.  

V$DBLINK 뷰를 다시 조회하면 in&#95;transaction 열이 NO로 변경된 것을 확인할 수 있다.  

```sql
SELECT db_link, logged_on, open_cursors, in_transaction FROM v$dblink;
```

DB 링크를 완전히 닫으려면 아래와 같이 ALTER SESSION 문을 수행해야 한다.  

```sql
ALTER SESSION CLOSE DATABASE LINK dl1;
```

V$DBLINK 뷰를 다시 조회하면 DB 링크가 닫힌 것을 확인할 수 있다.  

```sql
SELECT db_link, logged_on, open_cursors, in_transaction FROM v$dblink;
```

DBA&#95;&#95;LINK&#95;SOURCES 뷰에서 현재 데이터베이스로 열린 DB 링크의 소스 데이터베이스에 대한 정보를 조회할 수 있다.  

```sql
SELECT db_name, host_name, ip_address, last_logon_time, logon_count
FROM dba_db_link_sources;
```

##### 20.9.2.2. 오브젝트 관리
<br/>
쿼리에 직접 DB 링크를 기술하는 코딩 패턴은 유지보수 측면에서 바람직하지 못하다. DB 링크를 사용하는 오브젝트에 대한 시너님을 생성하고 시너님을 통해 오브젝트를 액세스하는 편이 바람직하다.  

아래와 같이 DB 링크를 사용한 테이블에 시너님을 생성하자.  

```sql
CREATE OR REPLACE SYNONYM t1_dl1 FOR t1@dl1;
```

시너님을 조회하면 DB 링크를 기술하지 않고 원격 데이터베이스의 테이블을 조회할 수 있다.  

```sql
SELECT * FROM t1_dl1;
```

### 20.10 COMMENT 문
<br/>
COMMENT 문은 테이블, 뷰, 열 등의 데이터베이스 오브젝트에 주석을 생성한다.  

```
COMMENT ON { COLUMN [schema.](table | view).column
           | TABLE [schema.]{table | view}} IS string;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);
```

아래 쿼리는 t1 테이블과 t1 테이블의 c1 열에 주석을 생성한다.  

```sql
CONNECT ON TABLE t1 IS 'T1 Table';

COMMENT ON COLUMN t1.c1 IS 'C1 Column';
```

&#42;&#95;TAB&395;COMMENTS 뷰에서 테이블과 뷰의 주석을 조회할 수 있다.  

```sql
SELECT table_type, comments FROM user_tab_comments WHERE table_name = 'T1';
```

&#42;&#95;COL&#95;COMMENTS 뷰에서 열의 주석을 조회할 수 있다.  

```sql
SELECT column_name, comments FROM user_col_comments WHERE table_name = 'T1';
```

아래와 가팅 빈 문자('') 지정하면 주석이 삭제된다.  

```sql
COMMENT ON TABLE t1 IS '';
COMMENT ON COLUMN t1.c1 IS '';
```
