---
title:  DCL 문
categories:
- Unkind_SQL
feature_text: |
  ## 21. DCL 문
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

DCL은 Data Control Language의 약자다. 데이터 제어어로 해석할 수 있다. DCL 문을 사용하면 데이터에 대한 권한을 부여하거나 취소할 수 있다.  

### 21.1. 사용자
<br/>
사용자(user)는 데이터베이스에 로그인할 수 있는 계정이다. 사용자는 데이터베이스 오브젝트를 소유할 수 있다. 사용자가 소유하고 있는 오브젝트와 논리적인 집합을 스키마(schema)라고 한다.  

CREATE USER 문, ALTER USER 문, DROP USER 문은 DDL 문에 속한다.  

#### 21.1.1. 기본 문법
<br/>
##### 21.1.1.1. CREATE USER 문
<br/>
CREATE USER 문은 사용자를 생성한다.  

```
CREATE USER user IDENTIFIED BY password
  [ DEFAULT TABLESPACE tablespace
  | TEMPORARY TABLESPACE {tablespace | tablespace_group_name}
  | {QUOTA {size_clause | UNLIMITED} ON tablespace}...
  | PROFILE profile
  | PASSWORD EXPIRE
  | ACCOUNT {LOCK | UNLOCK}];
```

+ IDENTIFIED BY : 사용자의 패스워드를 지정
+ DEFAULT TABLESPACE : 사용자의 기본 테이블스페이스를 지정
+ TEMPORARY TABLESPACE : 사용자의 기본 임시 테이블스페이스를 지정
+ QUOTA : 사용자가 할당할 수 있는 테이블스페이스의 한도를 설정
+ PROFILE : 사용자의 프로필을 지정
+ PASSWORD EXPIRE : 사용자가 패스워드를 변경하도록 만료시킴
+ ACCOUNT : 사용자의 계정을 잠그거나 잠금을 해제  

SYS 사용자로 로그인한 세션에서 아래 쿼리를 수행하면 U1 사용자가 생성된다.  

```sql
CREATE USER u1 IDENTIFIED BY 'u1';
```

###### 21.1.1.1.1. ORA-65096 에러
<br/>
12.1 버전부터 멀티 테넌드(multitenant) 기능을 사용할 수 있다. 멀티 테넌트 기능을 사용하면 데이터베이스가 CDB(Container DB)로 동작한다. CDB는 다우스이 PDB(Pluggable DB)를 포함할 수 있는 데이터베이스다. CDB는 일반 사용자(common user), PDB는 로컬 사용자(local user)를 사용한다.  

CDB에서 사용자를 생성하면 "ORA-65096: 공통 사용자 또는 롤 이름이 부적합합니다." 에러가 발생할 수 있다. 아래와 같이 c##을 접두어로 붙이면 에러가 발생하지 않는다.  

```sql
CREATE USER c##u1 IDENTIFIED BY 'u1';
```

&#95;oracle&#95;script 파라미터를 TRUE로 변경하면 접두어를 붙이지 않아도 에러가 발생하지 않는다.  

```sql
ALTER SESSION SET '_oracle_script' = TRUE;
```

##### 21.1.1.2. ALTER USER 문
<br/>
ALTER USER 문은 사용자를 변경한다. IDENTIFIED BY 절 외에도 CREATE USER 문에 사용할 수 있는 모든 절을 사용할 수 있다.  

```
ALTER USER user IDENTIFIED BY password [REPLACE old_password];
```

아래 쿼리는 U1 사용자의 계정 잠금을 해제한다.  

```sql
ALTER USER u1 ACCOUNT UNLOCK;
```

##### 21.1.1.3. DROP USER 문
<br/>
DROP USER 문은 사용자를 삭제한다. 스키마에 오브젝트가 존재하면 사용자를 삭제할 수 없다. CASCADE 키워드를 기술하면 속한 모든 오브젝트가 함께 삭제된다.  

```
DROP USER user [CASCADE];
```

아래 쿼리는 U1 사용자를 삭제한다.  

```sql
DROP USER u1 CASCADE;
```

### 21.2. 권한
<br/>
권한은 시스템 권한(system privilege)과 오브젝트 권한(object privilege)으로 구분된다.  

+ 시스템 권한 : 특정 데이터베이스 작업을 수행할 수 있는 권한
+ 오브젝트 권한 : 특정 오브젝트를 접근할 수 있는 권한  

#### 21.2.1. 기본 문법
<br/>
##### 21.2.1.1. GRANT 문
<br/>
GRANT 문은 사용자나 롤에 권한을 부여한다.  

###### 21.2.1.1.1. 시스템 권한
<br/>
아래는 시스템 권한에 대한 GRANT 문의 구문이다. ALL PRIVILEGE는 SELECT ANY DICTIONARY, ALTER DATABASE LINK, ALTER PUBLIC DATABASE LINK 권한을 제외한 모든 시스템 권한을 부여한다. PUBLIC은 모든 사용자에게 권한을 부여한다.  

```
GRANT {system_privilege | role} | ALL PRIVILEGE}
    [,{system_privilege | role} | ALL PRIVILEGE}]...
    TO {user | role | PUBLIC} [, {user | rele | PUBLIC}]...
[WITH {ADMIN | DELEGATE} OPTION];
```

GRANT 문은 아래의 두 가지 옵션을 사용할 수 있다. WITH DELEGATE OPTION은 12.1 버전부터 사용할 수 있다.  

+ WITH ADMIN OPTION : 부여 받은 권한이나 롤을 다른 사용자에게 부여하거나 취소 가능
+ WITH DELEGATE OPTION : 부여 받은 롤을 자신의 프로그램 유닛에 부여하거나 취소 가능  

예제를 위해 U1 사용자를 생성하고, U1 사용자에게 DBA 롤을 부여하자.  

```sql
CREATE USER u1 IDENTIFIED BY 'u1';
GRANT DBA TO u1;
```

다른 창에서 U1 사용자로 로그인하자.  

```cmd
명령프롬프트

C:\>sqlplus u1/u1@ora12cr2
```

U1 세션에서 U2 사용자를 생성하자.  

```sql
CREATE USER u2 IDENTIFIED BY 'u2';
```

다른 창에서 U2 사용자로 로그인하면 에러가 발생한다. CREATE SESSION 권한이 없기 때문이다.  

```cmd
명령프롬프트

C:\>sqlplus u2/u2@ora12cr2

ORA-01045: user U2 lacks CREATE SESSION privilege; logon denied
```

U1 세션에서 U2 사용자로 CREATE SESSION 권한을 부여하자.  

```sql
GRANT CREATE SESSION TO u2;
```

이제 U2 사용자로 접속할 수 있다.  

```cmd
명령프롬프트

C:\>sqlplus u2/u2@ora12cr2
```

U2 세션에서 t2 테이블을 생성하면 에러가 발생한다.  

```sql
CREATE TABLE t2 (c1 NUMBER);

ORA-01301: 권한이 불충분합니다.
```

U1 세션에서 U2 사용자로 CREATE TABLE 권한을 부여하자.  

```sql
GRANT CREATE TABLE TO u2;
```

U2 세션에서 테이블을 생성할 수 있다.  

```sql
CREATE TABLE t2 (c1 NUMBER);
```

U2 세션에서 데이터를 입력하면 에러가 발생한다.  

```sql
INSERT INTO t2 VALUES (1);

ORA-01950: 테이블스페이스 'USERS'에 대한 권한이 없습니다.  
```

U1 세션에서 U2 사용자로 UNLIMITED TABLESPACE 권한을 부여하자.  

```sql
GRANT UNLIMITED TABLESPACE TO u2;
```

U2 세션에서 t2 테이블에 데이터를 입력할 수 있다.  

```sql
INSERT INTO t2 VALUES (1);
```

&#42;&#95;SYS&#95;PRIVS 뷰에서 시스템 권한에 대한 정보를 조회할 수 있다.  

```sql
SELECT privilege, admin_option FROM dba_sys_privs WHERE grantee = 'U2';
```

SYSTEM&#95;PRIVILEGE&#95;MAP 테이블에서 전체 시스템 권한을 확인할 수 있다.  

```sql
SELECT name FROM system_privilege_map;
```

아래는 지금까지 부여한 시스템 권한에 대한 설명이다. ANY가 포함된 시스템 권한은 모든 스키마에 대한 권한이 부여된다.  

+ CREATE SESSION : 데이터베이스에 접속할 수 있는 권한
+ CREATE TABLE : 테이블을 생성할 수 있는 권한
+ CREATE ANY TABLE : 모든 스키마에 테이블을 생성할 수 있는 권한
+ UNLIMITED TABLESPACE : 모든 테이블스페이스를 제한 없이 사용할 수 있는 권한  

###### 21.2.1.1.2. UNLIMITED TABLESPACE 권한
<br/>
UNLIMITED TABLESPACE 권한은 모든 테이블스페이스를 제한 없이 사용할 수 있는 권한이다. 일반 사용자가 모든 테이블스페이스에 대해 사용 권한을 가지는 것은 운영 측면에서 바람직하지 못하다.  

아래와 같이 특정 테이블스페이스에 대한 한도를 설정하는 편이 바람직하다.  

```sql
ALTER USER u1 QUOTA UNLIMITED ON users;
```

###### 21.2.1.1.3. 오브젝트 권한
<br/>
아래는 오브젝트 권한에 대한 GRANT 문의 구문이다. WITH GRANT OPTION 절을 사용하면 부여 받은 오브젝트 권한을 다른 사용자나 롤을 부여하거나 취소할 수 있다.  

```
GRANT {object_privilege | ALL [PRIVILEGES]} [(column [, column]...)]
   [, {object_privilege | ALL [PRIVILEGES]} [(column [, column]...)]]...
   ON {[schema.]object | USER user [, user]... | DIRECTORY directory_name}
   TO {user | role | PUBLIC} [, {user | role | PUBLIC}]...
[WITH GRANT OPTION];
```

U1 세션에서 t1 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
```

U2 세션에서 U1 사용자가 소유하고 있는 t1 테이블을 조회하면 에러가 발생한다. t1 테이블에 대한 SELECT 권한이 없기 때문이다.  

```sql
SELECT * FROM u1.t1;

ORA-00942: 테이블 또는 뷰가 존재하지 않습니다.  
```

U1 세션에서 U2 사용자로 t1 테이블에 대한 SELECT 권한을 부여하자.  

```sql
GRANT SELECT ON t1 TO u2;
```

U2 세션에서 u1.t1 테이블을 조회할 수 있다.  

```sql
SELECT * FROM u1.t1;
```

U2 세션에서 u1.t1 테이블에 데이터를 입력하면 에러가 발생한다.  

```sql
INSERT INTO u1.t1 VALUES (1, 1);

ORA-01031: 권한이 불충분합니다.
```

U1 세션에서 U2 사용자로 t1 테이블에 대한 INSERT 권한을 부여하자.  

```sql
GRANT INSERT ON t1 TO u2;
```

U2 세션에서 u1.t1 테이블에 데이터를 입력할 수 있다.  

```sql
INSERT INTO u1.t1 VALUES (1, 1);
```

&#42;&395;TAB&#95;PRIVS 뷰에서 오브젝트 권한에 대한 정보를 조회할 수 있다.  

```sql
SELECT owner, table_name, grantor, privilege, grantable, type
FROM dba_tab_privs
WHERE grantee = 'U2';
```

U1 세션에서 u1.t1 테이블을 갱신하면 에러가 발생한다.  

```sql
UPDATE u1.t1 SET c2 = 2 WHERE c1 = 1;

ORA-01031: 권한이 불충분합니다
```

U1 세션에서 U2 사용자로 t1 테이블의 c2 열에 대한 UPDATE 권한을 부여하자.  

```sql
GRANT UPDATE (c2) ON t2 TO u2;
```

U2 세션에서 u1.t1 테이블의 c2 열을 갱신할 수 있다.  

```sql
UPDATE u1.t2 SET c2 = 2 WHERE c1 = 1;
```

c2 열에 대한 UPDATE 권한만 부여했기 때문에 c1 열을 갱신하면 에러가 발생한다.  

```sql
UPDATE u1.t1 SET c1 = 2 WHERE c1 = 1;

ORA-01031: 권한이 불충분합니다
```

&#42;&#95;COL&#95;PRIVS 뷰에서 칼럼 레벨 오브젝트 권한에 대한 정보를 조회할 수 있다.  

```sql
SELECT owner, table_name, column_name, grantor, privilege, grantable
FROM dba_col_privs
WHERE grantee = 'U2';
```

V$OBJECT&#95;PRIVILEGE 뷰에서 전체 오브젝트 권한을 확인할 수 있다.  

```sql
SELECT object_type_name, privilege_name FROM v$object_privilege;
```

아래는 오브젝트 별로 사용할 수 있는 오브젝트 권한이다.  

<table>
  <thead>
    <tr>
      <td>권한</td>
      <td>TABLE</td>
      <td>VIEW</td>
      <td>SEQUENCE</td>
      <td>PROCEDURE</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>ALTER</td>
      <td>Y</td>
      <td></td>
      <td>Y</td>
      <td></td>
    </tr>
    <tr>
      <td>DEBUG</td>
      <td></td>
      <td></td>
      <td></td>
      <td>Y</td>
    </tr>
    <tr>
      <td>DELETE</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>EXECUTE</td>
      <td></td>
      <td></td>
      <td></td>
      <td>Y</td>
    </tr>
    <tr>
      <td>INDEX</td>
      <td>Y</td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>INSERT</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>READ</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>REFERENCES</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td>SELECT</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
    </tr>
    <tr>
      <td>UPDATE</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>

###### 21.2.1.1.4. 전용 시너님
<br/>
다른 사용자가 소유하고 있는 오브젝트에 전용 시너님을 생성하면 스키마를 기술하지 않고 다른 사용자의 오브젝트를 조회할 수 있다.  

U1 세션에서 아래와 같이 시너님을 생성하자.  

```sql
CREATE SYNONYM u2.t1 FOR u1.t1;
```

U2 세션에서 스키마를 기술하지 않고 U1 사용자의 t1 테이블을 조회할 수 있다.  

##### 21.2.1.2. REVOKE 문
<br/>
REVOKE 문은 부여된 권한을 취소한다.  

###### 21.2.1.2.1. 시스템 권한
<br/>
아래는 시스템 권한에 대한 REVOKE 문의 구문이다.  

```
REVOKE {system_privilege | role | ALL PRIVILEGES}
     [ {system_privilege | role | ALL PRIVILEGES}]...
FROM {user | role | PUBLIC} [, {user | role | PUBLIC}]...;
```

아래 쿼리는 U2 사용자에게 부여한 CREATE TABLE 권한과 UNLIMITED TABLESPACE 권한을 취소한다.  

```sql
REVOKE CREATE TABLE, UNLIMITED TABLESPACE FROM u2;
```

&#42;&#95;SYS&#95;PRIVS 뷰에서 권한이 취소된 것을 확인할 수 있다.  

```sql
SELECT privilege, admin_option FROM dba_sys_privs WHERE grantee = 'U2';
```

###### 21.2.1.2.2. 오브젝트 권한
<br/>
아래는 오브젝트 권한에 대한 REVOKE 문의 구문이다.  

```
REVOKE {object_privilege | ALL [PRIVILEGES]}
     [ {object_privilege | ALL [PRIVILEGES]}]...
     ON {[schema.]object | USER user [, user]... | DIRECTORY directory_name}
FROM {user | role | PUBLIC} [, {user | role | PUBLIC}]...
[CASCADE CONSTRAINTS | FORCE];
```

+ CASCADE CONSTRAINTS : FK 제약 조건과 관련하여 REFERENCES 권한을 취소
+ FORCE : 사용자 정의 타입과 관련하여 EXECUTE 권한을 취소  

아래 쿼리는 U2 사용자에게 부여한 t1 테이블에 대한 모든 오브젝트 권한을 취소한다.  

```sql
REVOKE ALL ON t1 FROM t2;
```

U2 세션에서 u1.t1 테이블을 조회하면 에러가 발생한다.  

```sql
SELECT * FROM u1.t1;

ORA-00942: 테이블 또는 뷰가 존재하지 않습니다.
```

### 21.3. 롤
<br/>
롤(role)은 권한과 롤의 모음이다. 사용자에게 롤을 부여하면 롤에 부여된 권한과 롤이 사용자에게 부여된다. 롤을 사용하면 권한을 효율적으로 관리할 수 있다.  

#### 21.3.1. 기본 문법
<br/>
##### 21.3.1.1. CREATE ROLE 문
<br/>
CREATE ROLE 문은 롤을 생성한다.  

```
CREATE ROLE role [NOT IDENTIFIED | IDENTIFIED BY password];
```

U1 세션에서 r1 롤을 생성하자.  

```sql
CREATE ROLE r1;
```

아래 쿼리는 r1 롤에 CREATE SESSION 권한과 CREATE TABLE 권한을 부여한다.  

```sql
GRANT CREATE SESSION, CREATE TABLE TO r1;
```

&#42;&#95;SYS&#95;PRIVS 뷰에서 롤에 부여된 시스템 권한을 확인할 수 있다.  

```sql
SELECT privilege, admin_option FROM dba_sys_privs WHERE grantee = 'R1';
```

아래와 같이 U1 세션에서 r2 롤을 생성하자.  

```sql
CREATE ROLE r2;
```

r2 롤에 t1 테이블에 대한 SELECT 권한과 INSERT 권한, c2 열에 대한 UPDATE 권한을 부여하자.  

```sql
GRANT SELECT, INSERT, UPDATE (c2) ON t1 TO r2;
```

&#42;&#95;TAB&#95;PRIVS 뷰에서 롤에 부여된 오브젝트 권한을 확인할 수 있다.  

```sql
SELECT owner, table_name, grantor, privilege, grantable, type
FROM dba_tab_privs
WHERE grantee = 'R2';
```

&#42;&#95;COL&#95;PRIVS 뷰에서 롤에 부여된 칼럼 레벨 오브젝트 권한을 확인할 수 있다.  

```sql
SELECT owner, table_name, column_name, grantor, privilege, grantable
FROM dba_col_privs
WHERE grantee = 'R2';
```

롤에 롤을 부여할 수도 있다. 아래 쿼리는 r2 롤을 r1 롤에 부여한다.  

```sql
GRANT r2 TO r1;
```

&#42;&#95;ROLE&#95;PRIVS 뷰에서 롤의 권한에 대한 정보를 조회할 수 있다.  

```sql
SELECT granted_role, admin_option FROM dba_role_privs WHERE grantee = 'R1';
```

&#42;&#95;ROLES 뷰에서 롤에 대한 정보를 조회할 수 있다.  

```sql
SELECT role, password_required FROM dba_roles WHERE role IN ('R1', 'R2');
```

U2 사용자에게 r1 롤을 부여하자.  

```sql
GRANT r1 TO u2;
```

U2 사용자로 다시 접속하면 u1.t1 테이블을 조회할 수 있다. 권한은 즉시 적용되지만 롤은 다시 잡속해야 적용된다.  

```cmd
C:\sqlplus u2/u2@ora12cr2
```

##### 21.3.1.2. ALTER ROLE 문
<br/>
ALTER ROLE 문을 사용하면 롤에 패스워드를 설정하거나 설정된 패스워드를 해제할 수 있다.  

```
ALTER ROLE role [NOT IDENTIFIED | IDENTIFIED BY password];
```

아래 쿼리는 r1 롤에 패스워드를 설정한다.  

```sql
ALTER ROLE r1 IDENTIFIED BY 'r1';
```

##### 21.3.1.3. DROP ROLE 문
<br/>
DROP ROLE 문은 롤을 삭제한다.  

```
DROP ROLE role;
```

아래 쿼리는 r2 롤을 삭제한다.  

```sql
DROP ROLE r2;
```

##### 21.3.1.4. 권한 중복 부여
<br/>
권한은 사용자와 롤에 부여할 수 있기 때문에 중복되어 부여될 수 있다. 앞서 살벼본 예제에서 CREATE SESSION 권한은 U2 사용자와 r1 롤에 부여되었고, r1 롤은 다시 U2 사용자에게 부여되었다.  

아래 쿼리는 시스템 권한과 롤을 계층 구조로 조회한다. CREATE SESSION 권한이 중복 부여된 것을 확인할 수 있다.  

```sql
WITH w1 AS (
  SELECT NULL AS p, username AS c, 'U' AS t FROM dba_users UNION
  SELECT grantee AS p, privilege AS c, 'P' AS t1 FROM dba_sys_privs UNION
  SELECT grantee AS p, granted_role AS c, 'R' AS t FROM dba_role_privs)
SELECT  t, LPAD(' ', (LEVEL - 1) * 4) || c AS privilege
FROM w1
STRAT WITH p IS NULL
AND c = 'U2'
CONNECT BY p = PRIOR c;
```

#### 21.3.2. SET ROLE 문
<br/>
SET ROLE 문은 현재 세션에서 롤을 활성화하거나 비활성화한다.  

SET ROLE 문은 SCS 문이다.  

```
SET ROLE
  {role [IDENTIFIED BY password] [, role [IDENTIFIED BY password]]...
  | ALL [EXCEPT role, [, role]...]
  | NONE};
```

앞서 살펴본 예제에서 r1 롤에 패스워드를 설정했다. 패스워드가 설정된 롤은 &#42;&#95;ROLES 뷰의 password&#95;required 열이 YES로 표시된다.  

```sql
SELECT role, password_required FROM dba_roles WHERE role = 'R1';
```

U2 사용자로 다시 접속한 후 t3 테이블을 생성하면 에러가 발생한다.  

```cmd
C:\sqlplus u2/u2@ora12cr2

CREATE TABLE t3 (c1 NUMBER);

ORA-01031: 권한이 불충분합니다
```

SET ROLE 문을 사용해서 r1 롤을 활성화하면 테이블을 생성할 수 있다.  

```sql
SET ROLE r1 IDENTIFIED BY 'r1';

CREATE TABLE t3 (c1 NUMBER);
```

SESSION&#95;ROLES 뷰와 SESSION&#95;PRIVS 뷰에서 현재 세션의 활성화된 권한과 롤을 조회할 수 있다.  

```sql
SELECT * FROM session_privs;

SELECT * FROM session_roles;
```

다음 예제를 위해 r1 롤의 패스워드를 해제하자.  

```sql
ALTER ROLE r1 NOT IDENTIFIED;
```

##### 21.3.2.1. DEFAULT ROLE 절
<br/>
DEFAULT ROLE 절을 사용하면 사용자의 기본 롤을 지정하거나 해제할 수 있다. 패스워드가 설정되지 않은 롤은 부여받은 사용자의 기본 롤로 설정된다.  

```
ALTER USER user DEFAULT ROLE {role [, role]... | ALL [EXCEPT role [, role]...] | NONE};
```

기본 롤로 설정된 롤은 &#42;&#95;ROLE&#95;PRIVS 뷰의 default&#95;role 열이 YES로 표시된다.  

```sql
SELECT granted_role, default_role FROM dba_role_privs WHERE grantee = 'U2';
```

아래 쿼리는 모든 기본 롤을 해제한다.  

```sql
ALTER USER u2 DEFAULT ROLE NONE;
```

U2 사용자로 다시 접속한 다음 t4 테이블을 생성하면 에러가 발생한다.  

```cmd
C:\sqlplus u2/u2@ora12cr2

CREATE TABLE t4 (c1 NUMBER);
```

SET ROLE 문으로 전체 롤을 활성화하면 테이블을 생성할 수 있다.  

```sql
SET ROLE ALL;

CREATE TABLE t4 (c1 NUMBER);
```

##### 21.3.2.2. 권한과 롤
<br/>
권한은 시스템 권한, 오브젝트 권한, 칼럼 레벨 오브젝트 권한으로 구분된다. 권한은 사용자와 롤에 부여할 수 있고, 롤은 롤에 부여할 수 있다.  

권한과 롤은 복잡한 관계 구조로 인해 관련된 뷰도 많다. 아래 표에서 관련된 뷰를 확인할 수 있다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>DBA</td>
      <td>ALL</td>
      <td>USER</td>
      <td>ROLE</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>&#42;&#95;SYS&#95;PRIVS</td>
      <td>Y</td>
      <td></td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>&#42;&#95;TAB&#95;PRIVS</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>&#42;&#95;COL&#95;PRIVS</td>
      <td>Y</td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
    </tr>
    <tr>
      <td>&#42;&#95;ROLE&#95;PRIVS</td>
      <td>Y</td>
      <td></td>
      <td>Y</td>
      <td>Y</td>
    </tr>
    <tr>
      <td>&#42;&#95;TAB&#95;PRIVS&#95;MADE</td>
      <td></td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
    </tr>
    <tr>
      <td>&#42;&#95;TAB&#95;PRIVS&#95;RECD</td>
      <td></td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
    </tr>
    <tr>
      <td>&#42;&#95;COL&#95;PRIVS&#95;MADE</td>
      <td></td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
    </tr>
    <tr>
      <td>&#42;&#95;COL&#95;PRIVS&#95;RECD</td>
      <td></td>
      <td>Y</td>
      <td>Y</td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>
