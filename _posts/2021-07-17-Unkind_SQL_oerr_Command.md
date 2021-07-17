---
title:  oerr 명령어
categories:
- Unkind_SQL
feature_text: |
  ## D. oerr 명령어
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

oerr 명령어로 에러에 대한 내용을 확인할 수 있다. 12.1 버전부터 Windows에서도 oerr 명령어를 사용할 수 있다.  

12.2.0.1 버전에서 oerr 명령어를 사용하면 아래와 같이 에러가 발생한다.  

```
C:\>oerr
ORACLE_HOME not set. Contact Oracle Support Services.
```

where 명령어로 oerr 명령어의 경로를 확인할 수 있다.  

```
C:\>where oerr
```

아래는 oerr.bat 파일의 내용이다. perl 명령어로 oerr.pl 파일을 실행하는 것을 확인할 수 있다.  

```bat
setlocal

set OH=$ORACLE_HOME%

if ($OR$)==() (
  echo ORACLE_HOME not set. Contact Oracle Support Services.
  goto out
)

$OH$\perl\bin\perl $OH$\bin\oerr.pl %1 %2 %3 %4 %5

:out
endlocal
```

아래와 같이 코드를 변경하자.  

```bat
setlocal

if ($ORACLE_HOME$)==() (
  set ORACLE_HOME=$-dp0\..
)

%ORACLE_HOME%\perl\bin\perl %ORACLE_HOME%\bin\oerr.pl %1 %2 %3 %4 %5

endlocal
```

oerr 명령어를 수행하면 정상적으로 동작하는 것을 확인할 수 있다.  

```
C:\>where oerr
```

오라클 데이터베이스는 에러에 대한 정보를 조회할 수 있는 메타데이터를 제공하지 않는다. oraus.msg 파일의 내요을 테이블에 입력하여 메타데이터를 직접 생성해보자.  

```sql
DROP DIRECTORY dir_mesg;

CREATE OR REPLACE DIRECTORY dir_mesg AS 'C:\app\ora12cr2\product\12.2.0\dbhome_1\rdbms\mesg';

DROP TABLE ext_oraus PURGE;

CREATE TABLE ext_oraus (line NUMBER, text VARCHAR2(4000))
ORGANIZATION_EXTERNAL (
  TYPE ORACLE_LOADER
  DEFAULT DIRECTORY dir_mesg
  ACCESS PARAMETERS (
    RECORDS DELIMITED BY NEWLINE
    NOBADFILE NOLOGFILE NODISCARDFILE
    FILEDS TERMINATED BY '!@#' (
      line RECNUM,
      text POSITION(1:4000)
    )
  )
  LOCATION ('oraus.msg')
) REJECT LIMIT UNLIMITED;
```

oraus.msg 파일을 파싱할 패키지를 생성하자.  

```sql
CREATE OR REPLACE PACKAGE pkg_oerr
IS
  TYPE t_oerr_rc IS RECORD (code NUMBER, message VARCHAR2(4000));
  TYPE t_oerr_nt IS TABLE OF t_oerr_rc;
  TYPE t_oerr_text_rc IS RECORD (code NUMBER, line NUMBER, text VARCHAR2(4000));
  TYPE t_oerr_text_nt IS TABLE OF t_oerr_text_rc;
  FUNCTION fnc_oerr RETURN t_oerr_nt PIPELINED;
  FUNCTION fnc_oerr_text RETURN t_oerr_text_nt PIPELINED;
END pkg_oerr;

CREATE OR REPLACE PACKAGE BODY pkg_oerr
IS
  FUNCTION fnc_oerr
    RETURN t_oerr_nt PIPELINED
  IS
    l_oerr t_oerr_rc;
  BEGIN
    FOR c1 IN (SELECT TO_NUMBER (REGEXP_SUBSTR(text, '^[0-9]+')) AS code
                    , REGEXP_SUBSTR(text, '"(.+)"', 1, 1, 'i', 1) AS message
               FROM ext_oraus
               WHERE REGEXP_SUBSTR(text, '^[0-9].+'))
    LOOP
      l_oerr := c1;
      PIPE ROW (l_oerr);
    END LOOP;
  END fnc_oerr;

  FUNCTION fnc_oerr_text
    RETURN t_oerr_text_nt PIPELINED
  IS
    l_oerr_text t_oerr_text_rc;
  BEGIN
    FOR c1 IN (SELECT TO_NUMBER (REGEXP_SUBSTR(text, '^[0-9]+')) AS code
                    , SUBSTR(text, 4) AS text
               FROM ext_oraus
               WHERE REGEXP_LIKE(text, '^[0-9].+')
               OR REGEXP_LIKE(text, '^//.+'))
    LOOP
      CASE WHEN c1.code IS NOT NULL
           THEN l_oerr_text.code := c1.code;
      ELSE l_oerr_text.line := l_oerr_text.line + 1;
           l_oerr_text.text := c1.text;
           PIPE ROW (l_oerr_text);
      END CASE;
    END LOOP;
  END fnc_oerr_text;
END pkg_oerr;
/
```

아래 쿼리로 파싱한 결과를 테이블에 입력하자.  

```sql
DROP TABLE t_oerr PURGE;
DROP TABLE t_oerr_text PURGE;

CREATE TABLE t_oerr AS SELECT * FROM TABLE (pkg_oerr.fnc_oerr);
CREATE TABLE t_oerr_text AS SELECT * FROM TABLE (pkg_oerr.fnc_oerr_text);

ALTER TABLE t_oerr ADD CONSTRAINT t_oerr_pk PRIMARY KEY (code);
ALTER TABLE t_oerr_text ADD CONSTRAINT t_oerr_text_pk PRIMARY KEY (code, line);
```

t&#95;oerr 테이블을 조회하면 에러 메시지를 확인할 수 있다.  

```sql
SELECT message FROM t_oerr WHERE code = 1l
```

t&#95;oerr&#95;text 테이블을 조회하면 에러에 대한 원인과 권장 조치를 확인할 수 있다.  

```sql
SELECT text FROM t_oerr_text WHERE code = 1 ORDER BY line;
```
