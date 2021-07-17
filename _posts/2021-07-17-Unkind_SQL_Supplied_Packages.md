---
title:  제공 패키지
categories:
- Unkind_SQL
feature_text: |
  ## 32. 제공 패키지
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

오라클 데이터베이스는 애플리케이션 개발에 활용할 수 있는 다양한 패키지(supplied package)를 제공한다. PL/SQL 개발에 사용할 수 있는 패키지가 대부분이지만 일부 패키지는 SQL 개발에 활용할 수 있는 패키지가 대부분이지만 일부 패키지는 SQL 개발에 활용할 수 있다.  

+ 패키지  
패키지(package)는 PL/SQL로 작성된 프로그램의 모음이다. 패키지에 포함된 프로그램을 서브 프로그램(subprogram)이라고 한다. 서브 프로그램은 프로시저(procedure)와 함수(function)로 작성된다. 제공 패키지의 서브 프로그램은 다수의 데이터 타입을 처리할 수 있도록 오버로딩(overloading)되어 있다. 서브 프로그램은 [패키지].[서브 프로그램] 형식으로 사용할 수 있다. 패키지 레벨에서 사용할 수 있는 변수와 상수도 제공된다.  

### 32.1. DBMS&#95;CRYPTO 패키지
<br/>
DBMS&#95;CRYPTO 패키지는 데이터를 암복호화하는 기능을 제공한다.  

예제를 위해 SYS 사용자로 접속한 세션에서 SCOTT 사용자에게 DBMS&#95;CRYPTO 패키지에 대한 실행 권한을 부여하자.  

```sql
GRANT EXECUTE ON DBMS_CRYPTO TO scott;
```

#### 32.1.1. 패키지 함수
<br/>
##### 32.1.1.1. HASH 함수
<br/>
HASH 함수는 단방향 해시 함수다. typ에 지정한 해시 알고리즘으로 src에 대한 해시 값을 생성한다. typ은 MD(Message Digest) 방식과 SHA(Secure Hash Algorithm) 방식을 사용할 수 있다.  

```
DBMS_CRYPTO.HASH(src IN CLOB CHARACTER SET ANY_CS, typ IN PLS_INTEGER) RETURN RAW;
```

아래와 같은 해시 알고리즘을 사용할 수 있다.  

<table>
  <thead>
    <tr>
      <td>typ</td>
      <td>상수</td>
      <td>알고리즘</td>
      <td>해시 값</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>HASH&#95;MD4</td>
      <td>MD4</td>
      <td>128비트</td>
    </tr>
    <tr>
      <td>2</td>
      <td>HASH&#95;MD5</td>
      <td>MD5</td>
      <td>128비트</td>
    </tr>
    <tr>
      <td>3</td>
      <td>HASH&#95;SH1</td>
      <td>SHA-1</td>
      <td>160비트</td>
    </tr>
    <tr>
      <td>4</td>
      <td>HASH&#95;SHA256</td>
      <td>SHA-2</td>
      <td>256비트</td>
    </tr>
    <tr>
      <td>5</td>
      <td>HASH&#95;SH384</td>
      <td>SHA-2</td>
      <td>384비트</td>
    </tr>
    <tr>
      <td>6</td>
      <td>HASH&#95;SHS12</td>
      <td>SHA-2</td>
      <td>512비트</td>
    </tr>
  </tbody>
</table>
<br/><br/>

DBMS&#95;CRYPTO 패키지에서 사용할 수 있는 알고리즘은 상수 값으로 지정되어 있다. 아래와 같은 방법으로 상수 값을 확인할 수 있다.  

```sql
SET SERVEROUTPUT ON

EXEC DBMS_OUTPUT.PUT_LINE (DBMS_CRYPTO.HASH_SH1);
```

아래는 HASH 함수를 사용한 쿼리다. SHA-1 알고리즘을 사용했다.  

```sql
SELECT DBMS_CRYPTO.HASH('ABC', 3) AS c1 FROM DUAL;
```

##### 32.1.1.2. MAC 함수
<br/>
MAC 함수는 키를 사용하는 단방향 해시 함수다. MAC는 Message Authentiaction Code의 약자다.  

```
DBMS_CRYPTO.MAC (
      src IN CLOB_CHARACTER SET ANY_CS
    , typ IN PLS_INTEGER
    , key IN RAW
)
```

아래와 같은 해시 알고리즘을 사용할 수 있다.  

<table>
  <thead>
    <tr>
      <td>typ</td>
      <td>상수</td>
      <td>알고리즘</td>
      <td>해시 값</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>HMAC&#95;MD5</td>
      <td>MD5</td>
      <td>128비트</td>
    </tr>
    <tr>
      <td>2</td>
      <td>HMAC&#95;SH1</td>
      <td>SHA-1</td>
      <td>160비트</td>
    </tr>
    <tr>
      <td>3</td>
      <td>HMAC&#95;SHA256</td>
      <td>SHA-2</td>
      <td>256비트</td>
    </tr>
    <tr>
      <td>4</td>
      <td>HMAC&#95;SH384</td>
      <td>SHA-2</td>
      <td>384비트</td>
    </tr>
    <tr>
      <td>5</td>
      <td>HMAC&#95;SHS12</td>
      <td>SHA-2</td>
      <td>512비트</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래 쿼리는 MAC 함수를 사용한 쿼리다. SHA-1 알고리즘을 사용했다. CAST&#95;TO&#95;RAW 함수는 문자 값을 RAW 값으로 변환한다.  

```sql
SELECT DBMS_CRYPTO.MAC('ABC', 2, UTL_RAW.CAST_TO_RAW('12345678')) AS c1 FROM DUAL;
```

##### 32.1.1.3. ENCRYPT 함수
<br/>
ENCRYPT 함수는 양방향 암호화 함수다. 지정한 typ(암호화 알고리즘), key(키), iv(초기화 벡터)로 src를 암호화한다. 초기화 벡터는 블록 암호화 방식의 첫 번째 블록에 연결할 값이다.  

```
DBMS_CRYPTO.ENCRYPT (
    src IN RAW
  , typ IN PLS_INTEGER
  , key IN RAW
  , iv IN RAW DEFAULT NULL
) RUTURN RAW;
```

아래와 같은 암호화 알고리즘을 사용할 수 있다. DES(Data Encrytion Standard)와 AES(Advanced Encryption Standard)는 블록(block) 암호화 방식, RC4(Rivest Cipher 4)는 스트림(stream) 암호화 방식이다.  

<table>
  <thead>
    <tr>
      <td>typ</td>
      <td>상수</td>
      <td>설명</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>ENCRYPT&#95;DES</td>
      <td>DES 알고리즘 (56비트 키)</td>
    </tr>
    <tr>
      <td>2</td>
      <td>ENCRYPT&#95;3DES&#95;2KEY</td>
      <td>DES 알고리즘 (112비트 키, DES를 3번 수행)</td>
    </tr>
    <tr>
      <td>3</td>
      <td>ENCRYPT&#95;3DES</td>
      <td>DES 알고리즘 (56비트 키, DES를 3번 수행)</td>
    </tr>
    <tr>
      <td>6</td>
      <td>ENCRYPT&#95;AES128</td>
      <td>AES 알고리즘 (128비트 키)</td>
    </tr>
    <tr>
      <td>7</td>
      <td>ENCRYPT&#95;AES192</td>
      <td>AES 알고리즘 (192비트 키)</td>
    </tr>
    <tr>
      <td>8</td>
      <td>ENCRYPT&#95;AES256</td>
      <td>AES 알고리즘 (256비트 키)</td>
    </tr>
    <tr>
      <td>129</td>
      <td>ENCRYPT&#95;RC4</td>
      <td>세션에 따라 임의로 생성된 고유 키 생성</td>
    </tr>
  </tbody>
</table>
<br/><br/>

블록 암호화 방식은 데이터를 블록으로 자른 후 블록 단위로 암호화흘 수행한다. 블록을 처리하기 위해 아래와 같은 체인 방식(chaining modifier)을 사용할 수 있다.  

<table>
  <thead>
    <tr>
      <td>typ</td>
      <td>상수</td>
      <td>설명</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>256</td>
      <td>CHAIN&#95;ECB</td>
      <td>Electronic CodeBook</td>
    </tr>
    <tr>
      <td>512</td>
      <td>CHAIN&#95;CBC</td>
      <td>Cipher Block Chaining</td>
    </tr>
    <tr>
      <td>768</td>
      <td>CHAIN&#95;CFB</td>
      <td>Cipher-FeedBack</td>
    </tr>
    <tr>
      <td>1024</td>
      <td>CHAIN&#95;OFB</td>
      <td>Output-FeedBack</td>
    </tr>
  </tbody>
</table>
<br/><br/>

잘려진 마지막 블록의 길이를 맞추기 위해 아래와 같은 패딩 방식(padding modifier)을 사용할 수 있다.  

<table>
  <thead>
    <tr>
      <td>typ</td>
      <td>상수</td>
      <td>설명</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>PAD&#95;PKCS5</td>
      <td>PKCS5(Password-Based Cryptography Standard) 사용</td>
    </tr>
    <tr>
      <td>2</td>
      <td>PAD&#95;NONE</td>
      <td>채우지 않음 (블록 길이가 다르면 에러 발생)</td>
    </tr>
    <tr>
      <td>3</td>
      <td>PAD&#95;ZERO</td>
      <td>0으로 채움</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래는 미리 정해진 블록 암호화 방식의 묶음(suite)이다. 암호화 알고리즘 + 체인 방식 + 패딩 방식으로 구성된다.  

<table>
  <thead>
    <tr>
      <td>typ</td>
      <td>상수</td>
      <td>설명</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>4353</td>
      <td>DES&#95;CBC&#95;PKCS5</td>
      <td>ENCRYPT&#95;DES + CHAIN&#95;CBC + PAD&#95;PKCS5</td>
    </tr>
    <tr>
      <td>4355</td>
      <td>DES3&#95;CBC&#95;PKCS5</td>
      <td>ENCRYPT&#95;3DES + CHAIN&#95;CBC + PAD&#95;PKCS5</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래는 ENCRYPT 함수를 사용한 쿼리다. ENCRYPT&#95;3DES + CHAIN&#95;CBC + PAD&#95;PKCS5 방식을 사용했다.  

```sql
SELECT DBMS_CRYPTO.ENCRYPT (UTL_RAW.CAST_TO_RAW ('ABC'), 4353
                          , UTL_RAW.CAST_TO_RAW ('12345678')) AS c1
FROM DUAL;
```

##### 32.1.1.4. DECRYPT 함수
<br/>
DECRYPT 함수는 ENCRYPT 함수에 의해 암호화된 값을 복호화한다.  

```
DBMS_CRYPTO.DECRYPT (
    src IN RAW
  , typ IN PLS_INTEGER
  , key IN RAW
  , iv IN RAW DEFAULT NULL
) RETURN RAW;
```

아래는 DECRYPT 함수를 사용한 쿼리다. ENCRYPT&#95;DES + CHAIN&#95;CBC + PAD&#95;PKCS5 방식을 사용했다. CAST&#95;TO&#95;VARCHAR2 함수는 RAW 값을 VARCHAR2 값으로 변환한다.  

```sql
SELECT UTL_RAW.CAST_TO_VARCHAR2 (DBMS_CRYPTO.DECRYPT ('73946ADB1AF08D6', 4353, UTL_RAW.CAST_TO_RAW('12345678'))) AS c1
FROM DUAL;
```

##### 32.1.1.5. RANDOMBYTES 함수
<br/>
RANDOMBYTES 함수는 임의의 number&#95;bytes RAW 값을 반환한다.  

```
DMBS_CRYPTO.RANDOMBYTES (number_bytes IN POSITIVE) RETURN RAW;
```

아래는 RANDOMBYTES 함수를 사용한 쿼리다. 임의의 16 바이트 RAW 값을 생성했다.  

```sql
SELECT DBMS_CRYPTO.RANDOMBYTES (16) AS c1 FROM DUAL;
```

##### 32.1.1.6. RANDOMNUMBER 함수
<br/>
RANDOMNUMBER 함수는 임의의 128비트 정수 값을 반환한다.  

```
DBMS_CRYPTO.RANDOMNUMBER RETURN NUMBER;
```

아래는 RANDOMNUMBER 함수를 사용한 쿼리다.  

```sql
SELECT DBMS_CRYPTO.RANDOMNUMBER AS c1 FROM DUAL;
```

##### 32.1.1.7. RANDOMINTEGER 함수
<br/>
RANDOMINTEGER 함수는 임의의 BINARY&#95;INTEGER 값을 반환한다. BINARY&#95;INTEGER 타입은 -2³¹ ~ 2³¹의 범위를 가진다.  

```
DBMS_CRYPTO.RANDOMINTEGER RETURN BINARY_INTEGER;
```

아래는 RANDOMINTEGER 함수를 사용한 쿼리다.  

```sql
SELECT DBMS_CRYPTO.RANDOMINTEGER AS c1 FROM DUAL;
```

#### 32.1.2. 활용 예제
<br/>
DBMS&#95;CRYPTO 패키지 함수는 사용자 정의 함수를 통해 사용하는 것이 일반적이다. 사용자 정의 함수를 사용하면 키에 대한 보안을 확보할 수 있다.  

예제를 위해 아래와 같이 패키지를 생성하자. 12345678 문자열을 키(1&#95;key)로 사용했다.  

```sql
CREATE OR REPLACE PACKAGE pkg_crypto
IS
  FUNCTION fnc_hash (i_src IN VARCHAR2) RETURN RAW DETERMINISTIC;
  FUNCTION fnc_mac (i_src IN VARCHAR2) RETURN RAW DETERMINISTIC;
  FUNCTION fnc_encrypt (i_src IN VARCHAR2) RETURN RAW DETERMINISTIC;
  FUNCTION fnc_dencrypt (i_src IN RAW) RETURN VARCHAR2 DETERMINISTIC;
END pkg_crypto;
/

CREATE OR REPLACE PACKAGE BODY pkg_crypto
IS
  l_typ PLS_INTEGER := DBMS_CRYPTO.ENCRYPT_DES + DBMS_CRYPTO.CHAIN_CBC + DBMS_CRYPTO.PAD_PKCS5;
  l_key RAW(32) := UTL_RAW.CAST_TO_RAW('12345678');

  FUNCTION fnc_hash (i_src IN VARCHAR2) RETURN RAW DETERMINISTIC
  IS
    l_typ PLS_INTEGER := DBMS_CRYPTO.HASH_SH1;
  BEGIN
    RETURN DBMS_CRYPTO.HASH (i_src, l_typ);
  END fnc_hash;

  FUNCTION fnc_mac (i_src IN VARCHAR2) RETURN RAW DETERMINISTIC
  IS
    l_typ PLS_INTEGER := DBMS_CRYPTO.HASH_SH1;
  BEGIN
    RETURN DBMS_CRYPTO.MAC (i_src, l_typ);
  END fnc_mac;

  FUNCTION fnc_encrypt (i_src IN VARCHAR2) RETURN RAW DETERMINISTIC
  IS
  BEGIN
    RETURN DBMS_CRYPTO.ENCRYPT (UTL_RAW.CAST_TO_RAW(i_src), l_typ, l_key);
  END fnc_encrypt;

  FUNCTION fnc_dencrypt (i_src IN RAW) RETURN VARCHAR2 DETERMINISTIC
  IS
  BEGIN
    RETURN (UTL_RAW.CAST_TO_VARCHAR2(DBMS_CRYPTO.DENCRYPT (i_src, l_typ, l_key)));
  END fnc_dencrypt;
END pkg_crypto;
/
```

아래는 fnc&#95;hash 함수를 사용한 쿼리다.  

```sql
SELECT pkg_crypto.fnc_hash ('ABC') AS c1 FROM DUAL;
```

아래는 fnc&#95;mac 함수를 사용한 쿼리다.  

```sql
SELECT pkg_crypto.fnc_mac ('ABC') AS c1 FROM DUAL;
```

아래는 fnc&#95;encrypt 함수를 사용한 쿼리다.  

```sql
SELECT pkg_crypto.fnc_encrypt ('ABC') AS c1 FROM DUAL;
```

아래는 fnc&#95;dencrypt 함수를 사용한 쿼리다.  

```sql
SELECT pkg_crypto.fnc_dencrypt ('73946ADB1AF08D6') AS c1 FROM DUAL;
```

### 32.2. DBMS&#95;RANDOM 패키지
<br/>
DBMS&#95;RANDOM 패키지는 임의 값을 생성하는 기능을 제공한다.  

#### 32.2.1. NORMAL 함수
<br/>
NORMAL 함수는 표준 정규 분포에 속한 임의의 숫자 값을 반환한다.  

```
DBMS_RANDOM.NORMAL RETURN NUMBER;
```

아래는 NORMAL 함수를 사용한 쿼리다.  

```sql
SELECT DBMS_RANDOM.NORMAL AS c1 FROM DUAL;
```

#### 32.2.2. STRING 함수
<br/>
STRING 함수는 임의의 문자열을 반환한다. opt에 문자열의 유형, len에 문자열의 길이를 지정할 수 있다.  

```
DBMS_RANDOM.STRING (opt IN CHAR, len IN NUMBER) RETURN VARCHAR2;
```

아래는 opt 파라미터에 지정할 수 있는 옵션이다.  

+ U : 대문자 알파벳 문자
+ L : 소문자 알파벳 문자
+ A : 알파벳 문자
+ X : 대문자 알파벳 문자와 숫자
+ P : 출력이 가능한 모든 문자  

아래 쿼리는 출력이 가능한 모든 문자로 구성된 길이가 50인 문자열을 반환한다.  

```sql
SELECT DBMS_RANDOM.STRING('P', 50) AS c1 FROM DUAL;
```

#### 32.2.3. VALUE 함수
<br/>
VALUE 함수는 임의의 숫자 값을 반환한다. low와 high에 값의 범위를 지정할 수 있다. 범위를 지정하지 않으면 0 <= x < 1의 범위에서 소수점 이하 38자리의 숫자 값을 반환한다.  

```
DBMS_RANDOM.VALUE RETURN NUMBER;
DBMS_RANDOM.VALUE (low IN NUMBER, high IN NUMBER) RETURN NUMBER;
```

아래는 VALUE 함수를 사용한 쿼리다.  

```sql
SELECT DBMS_RANDOM.VALUE AS c1, DBMS_RANDOM.VALUE (1, 1000000) AS c2 FROM DUAL;
```

### 32.3. DBMS&#95;LOB 패키지
<br/>
DBMS&#95;LOB 패키지는 LOB(Large Objects)과 관련된 기능을 제공한다.  

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 AS SELECT TO_CLOB ('가123') AS c1, TO_CLOB ('가123') AS c2 FROM DUAL;
```

#### 32.3.1. COMPARE 함수
<br/>
COMPARE 함수는 lob&#95;1과 lob&#95;2가 일치하면 0, 일치하지 않으면 0이 아닌 숫자 값을 반환한다. offset&#95;1에 lob&#95;1의 시작 위치, offset&#95;2에 lob&#95;2의 시작 위치, amount에 비교할 길이를 지정할 수 있다.  

```
DBMS_LOB.COMPARE (
    lob_1 IN CLOB CHARACTER SET ANY_CS
  , lob_2 IN CLOB CHARACTER SET lob_1%CHARSET
  , amount IN INTEGER := DBMS_LOB.LOBMAXSIZE
  , offset_1 IN INTEGER := 1
  , offset_2 IN INTEGER := 1
) RETURN INTEGER;
```

20장에서 살펴 본 것처럼 CLOB 값에 비교 조건을 사용하면 에러가 발생한다.  

```sql
SELECT * FROM t1 WHERE c1 = c2;

ORA-00932: 일관성 없는 데이터 유형: -이(가) 필요하지만 CLOB임
```

COMPARE 함수를 사용하면 CLOB 값을 비교할 수 있다.  

```sql
SELECT * FROM t1 DBMS_LOB.COMPARE(c1, c2) = 0;
```

#### 32.3.2. GETLENGTH 함수
<br/>
GETLENGTH 함수는 lob&#95;loc의 길이를 반환한다.  

```
DBMS_LOB.GETLENGTH (lob_loc IN CLOB CHARACTER SET ANY_CS) RETURN INTEGER;
```

아래는 GETLENGTH 함수를 사용한 쿼리다. LENGTH 내장 함수를 사용할 수도 있지만, GETLENGTH 패키지 함수를 사용하는 편이 성능 측면에서 유리할 수 있다.  

```sql
SELECT DBMS_LOB.GETLENGTH(c1) AS c1, LENGTH(c1) AS c2 FROM t1;
```

#### 32.3.3. INSTR 함수
<br/>
INSTR 함수는 lob&#95;loc의 offset에서 우측으로 nth번째 pattern의 시작 위치를 반환한다.  

```
DBMS_LOB.INSTR (
    lob_loc IN CLOB CHARACTER SET ANY_CS
  , pattern IN VARCHAR2 CHARACTER SET lob_loc%CHARSET
  , offset IN INTEGER := 1
  , nth IN INTEGER := 1
) RETURN INTEGER;
```

아래는 INSTR 함수를 사용한 쿼리다. INSTR 내장 함수를 사용할 수도 있지만, INSTR 패키지 함수를 사용하는 편이 성능 측면에서 유리할 수 있다.  

```sql
SELECT DBMS_LOB.INSTR(c1, '1') AS c1, INSTR(c1, '1') AS c2 FROM t1;
```

#### 32.3.4. SUBSTR 함수
<br/>
SUBSTR 함수는 lob&#95;loc을 offset 위치에서 우측으로 amount 길이만큼 잘라낸 값을 반환한다.  

```
DMBS_LOB.SUBSTR (
    lob_loc IN CLOB CHARACTER SET ANY_CS
  , amount IN INTEGER := 32767
  , offset IN INTEGER := 1
) RETURN VARCHAR2 CHARACTER SET lob_loc%CHARSET;
```

아래는 SUBSTR 하수를 사용한 쿼리다. SUBSTR 내장 함수를 사용할 수도 있지만, SUBSTR 패키지 함수를 사용하는 편이 성능 측면에서 유리할 수 있다.  

```sql
SELECT DBMS_LOB.SUBSTR(c1, 2, 1) AS c1, SUBSTR(c1, 1, 2) AS c2 FROM t1;
```

### 32.4. DBMS&#95;METADATA 패키지
<br/>
DBMS&#95;METADATA 패키지는 데이터베이스 딕셔너리의 메타데이터(metadata)와 관련된 기능을 제공한다.  

#### 32.4.1. GET&#95;DDL 함수
<br/>
GET&#95;DDL 함수는 schema와 name에 해당하는 DDL 문을 생성한다. object&#95;type에 TABLE, CLUSTER, CONSTRAINT, REF&#95;CONSTRAINT, INDEX, VIEW, SEQUENCE, SYNONYM, COMMENT 등의 오브젝트 유형을 지정할 수 있다.  

```
DBMS_METADATA.GET_DDL (
    object_type IN VARCHAR2
  , name IN VARCHAR2
  , schema IN VARCHAR2 DEFAULT NULL
  , version IN VARCHAR2 DEFAULT 'COMPATIBLE'
  , model IN VARCHAR2 DEFAULT 'ORACLE'
  , transform IN VARCHAR2 DEFAULT 'DDL'
) RETURN CLOB;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER, CONSTRAINT t1_pk PRIMARY KEY (c1));
```

아래는 GET&#95;DDL 함수를 사용한 쿼리다. 테이블을 생성한 CREATE TABLE 문보다 상세한 DDL 문이 반환된다.  

```sql
SELECT DBMS_METADATA.GET_DDL('TABLE', 'T1', 'SCOTT') AS c1 FROM DUAL;
```

아래 쿼리는 t1&#95;pk 인덱스의 DDL 문을 생성한다.  

```sql
SELECT DBMS_METADATA.GET_DDL ('INDEX', 'T1_PK', 'SCOTT') AS c1 FROM DUAL;
```

#### 32.4.2. SET&#95;TRANSFORM&#95;PARAM 프로시저
<br/>
SET&#95;TRANSFORM&#95;PARAM 프로시저를 사용하면 DDL 문의 포맷을 변경할 수 있다.  

```
DBMS_METADATA.SET_TRANSFORM_PARAM (
    transform_handle IN NUMBER
  , name IN VARCHAR2
  , value IN BOOLEAN DEFAULT TRUE
  , object_type IN VARCHAR2 DEFAULT NULL
);
```

name에 아래와 같은 파라미터를 지정할 수 있다. 일부 파라미터는 종속 관계를 가진다.  

<table>
  <thead>
    <tr>
      <td>파라미터</td>
      <td>설명</td>
      <td>기본값</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>PRETTY</td>
      <td>들여쓰기와 줄 바꿈</td>
      <td>TRUE</td>
    </tr>
    <tr>
      <td>SQLTERMINATOR</td>
      <td>SQL 종료자(;)를 붙임</td>
      <td>FALSE</td>
    </tr>
    <tr>
      <td>CONSTRAINTS</td>
      <td>FK 제약 조건 외의 제약 조건을 포함</td>
      <td>TRUE</td>
    </tr>
    <tr>
      <td>REF&#95;CONSTRAINTS</td>
      <td>FK 제약 조건을 포함</td>
      <td>TRUE</td>
    </tr>
    <tr>
      <td>CONSTRAINT&#95;AS&#95;ALTER</td>
      <td>제약 조건을 ALTER TABLE 문으로 분리</td>
      <td>FALSE</td>
    </tr>
    <tr>
      <td>PARTITIONING</td>
      <td>파티셔닝 절을 포함</td>
      <td>TRUE</td>
    </tr>
    <tr>
      <td>SEGMENT&#95;ATTRIBUTES</td>
      <td>세그먼트 속성 절을 포함</td>
      <td>TRUE</td>
    </tr>
    <tr>
      <td>STORAGE</td>
      <td>저장 절을 포함</td>
      <td>TRUE</td>
    </tr>
    <tr>
      <td>TABLESPACE</td>
      <td>테이블 스페이스 절을 포함</td>
      <td>TRUE</td>
    </tr>
  </tbody>
</table>
<br/><br/>

SET&#95;TRANSFORM&#95;PARAM 프로시저를 사용한 예제다. 설정 값은 세션 레벨로 유지된다.  

```sql
DECLARE
  l_th NUMBER := DBMS_METADATA.SESSION_TRANSFORM;
BEGIN
  DBMS_METADATA.SET_TRANSFORM_PARAM(l_th, 'PRETTY', FALSE);
  DBMS_METADATA.SET_TRANSFORM_PARAM(l_th, 'SQLTERMINATOR', TRUE);
  DBMS_METADATA.SET_TRANSFORM_PARAM(l_th, 'CONSTRAINTS', FALSE);
  DBMS_METADATA.SET_TRANSFORM_PARAM(l_th, 'SETMENT_ATTRUBUTES', FALSE);
  DBMS_METADATA.SET_TRANSFORM_PARAM(l_th, 'STORAGE', FALSE);
END;
/
```

GET&#95;DDL 쿼리를 다시 수행하면 DDL 문의 포맷이 변경된 것을 확인할 수 있다.  

```sql
SELECT DBMS_METADATA.GET_DDL ('TABLE', 'T1', 'SCOTT') AS c1 FROM DUAL;
```

### 32.5. DBMS&#95;APPLICATION&#95;INFO 패키지
<br/>
DBMS&#95;APPLICATION&#95;INFO 패키지는 세션에 애플리케이션 정보를 설정할 수 있는 기능을 제공한다. 설정된 정보는 애플리케이션 디버깅, 성능 모니터링 등에 활용할 수 있다.  

#### 32.5.1. SET&#95;MODULE 프로시저
<br/>
SET&#95;MODULE 프로시저는 애플리케이션 명과 모듈 명을 설정한다.  

```
DBMS_APPLICATION_INFO.SET_MODULE (module_name IN VARCHAR2, action_name IN VARCHAR2);
```

아래는 SET&#95;MODULE 프로시저를 사용한 예제다.  

```sql
EXEC DBMS_APPLICATION_INFO.SET_MODULE ('M1', 'A1');
```

#### 32.5.2. SET&#95;ACTION 프로시저
<br/>
SET&#95;ACTION 프로시저는 모듈의 액션 명을 설정한다.  

```
DBMS_APPLICATION_INFO.SET_ACTION(action_name IN VARCHAR2);
```

아래는 SET&#95;ACTION 프로시저를 사용한 예제다.  

```sql
EXEC DBMS_APPLICATION_INFO.SET_ACTION('A2');
```

#### 32.5.3. SET&#95;CLIENT&#95;INFO 프로시저
<br/>
SET&#95;CLIENT&#95;INFO 프로시저는 클라이언트 애플리케이션에 대한 정보를 설정한다.  

```
DBMS_APPLICATION_INFO.SET_CLIENT_INFO (client_info IN VARCHAR2);
```

아래는 SET&#95;CLIENT&#95;INFO 프로시저를 사용한 예제다.  

```sql
EXEC DBMS_APPLICATION_INFO.SET_CLIENT_INFO ('C1');
```

V$SESSION 뷰에서 설정한 정보를 확인할 수 있다. 설정한 정보를 통해 어떤 모듈의 어떤 액션이 어떤 클라이언트에서 수행 중인지 쉽게 추적할 수 있다.  

```sql
SELECT module, action, client_info
FROM v$session
WHERE SID = SYS_CONTEXT('USERENV', 'SID');
```

### 32.6. DBMS&#95;SESSION 패키지
<br/>
DBMS&#95;SESSION 패키지는 세션에 대한 정보를 설정하거나 옵션을 변경하는 기능을 제공한다.  

#### 32.6.1. SET&#95;IDENTIFIER 프로시저
<br/>
SET&#95;IDENTIFIER 프로시저는 세션의 클라이언트 ID를 설정한다.  

```
DBMS_SESSION.SET_IDENTIFIER (client_id VARCHAR2);
```

아래는 SET&#95;IDENTIFIER 프로시저를 사용한 예제다.  

```sql
EXEC DBMS_SESSION.SET_IDENTIFIER ('I1');
```

V$SESSION 뷰의 client&#95;identifier 열에서 설정한 정보를 확인할 수 있다.  

```sql
SELECT client_identifier FROM v$session WHERE SID = SYS_CONTEXT ('USERENV', 'SID');
```

#### 32.6.2. CLEAR&#95;IDENTIFIER 프로시저
<br/>
CLEAR&#95;IDENTIFIER 프로시저는 세션의 클라이언트 ID를 해제한다.  

```
DBMS_SESSION.CLEAR_IDENTIFIER;
```

아래는 CLEAR&#95;IDENTIFIER 프로시저를 사용한 예제다.  

```sql
EXEC DBMS_SESSION.CLEAR_IDENTIFIER;
```

### 32.7. UTL&#95;RAW 패키지
<br/>
UTL&#95;RAW 패키지는 RAW 타입을 조작할 수 있는 기능을 제공한다.  

RAW 타입은 가변 길이 이전(binary) 데이터 타입이다. 최대 2000바이트의 이진 값을 저장할 수 있다.  

```
RAW(n)
```

#### 32.7.1. 패키지 함수
<br/>
UTL&#95;RAW 패키지는 아래와 같은 변환 함수를 제공한다.  

+ CAST&#95;TO&#95;RAW : VARCHAR2 값을 RAW 값으로 변환
+ CAST&#95;TO&#95;VARCHAR2 : RAW 값을 VARCHAR2 값으로 변환
+ CAST&#95;TO&#95;NVARCHAR2 : RAW 값을 NVARCHAR2 값으로 변환
+ CAST&#95;TO&#95;NUMBER : RAW 값을 NUMBER 값으로 변환
+ CAST&#95;TO&#95;BINARY&#95;FLOAT : RAW 값을 BINARY&#95;FLOAT 값으로 변환
+ CAST&#95;TO&#95;BINARY&#95;DOUBLE : RAW 값을 BINARY&#95;DOUBLE 값으로 변환
+ CAST&#95;TO&#95;BINARY&#95;INTEGER : RAW 값을 BINARY&#95;INTEGER 값으로 변환  

#### 32.7.2. 활용 예제
<br/>
아래와 같이 사용자 정의 함수를 생성하자. 아래 사용자 정의 함수는 지정한 데이터 타입(i&#95;typ)의 형식으로 저장된 RAW 값(i&#95;val)을 문자 값으로 변환한다. UTL&#95;RAW 패키지는 날짜 값 변환을 지원하지 않기 때문에 DBMS&#95;STATS.CONVERT&#95;RAW&#95;VALUE 프로시저를 사용했다. DBMS&#95;STATS 패키지는 통계 정보와 관련된 기능을 제공한다.  

```sql
CREATE OR REPLACE FUNCTION fnc_raw (
    i_typ IN VARCHAR2
  , i_val IN RAW
) RETURN VARCHAR2
IS
  FUNCTION f1 (i_val IN RAW) RETURN DATE
  IS
    l_val DATE;
  BEGIN
    DBMS_STATS.CONVERT_RAW_VALUE (i_val, l_val);
    RETURN l_val;
  END f1;
BEGIN
  RETURN
CASE
  WHEN i_val IS NULL THEN NULL
  WHEN i_typ = 'CHAR' THEN TO_CHAR(UTL_RAW.CAST_TO_VARCHAR2(i_val))
  WHEN i_typ = 'VARCHAR2' THEN TO_CHAR(UTL_RAW.CAST_TO_VARCHAR2(i_val))
  WHEN i_typ = 'NVARCHAR2' THEN TO_CHAR(UTL_RAW.CAST_TO_NVARCHAR2(i_val))
  WHEN i_typ = 'NUMBER' THEN TO_CHAR(UTL_RAW.CAST_TO_NUMBER(i_val))
  WHEN i_typ = 'BINARY_FLOAT' THEN TO_CHAR(UTL_RAW.CAST_TO_BINARY_FLOAT(i_val))
  WHEN i_typ = 'BINARY_DOUBLE' THEN TO_CHAR(UTL_RAW.CAST_TO_BINARY_DOUBLE(i_val))
  WHEN i_typ = 'DATE' THEN TO_CHAR(f1(i_val), 'YYYY-MM-DD HH24:MI:SS')
  WHEN i_typ LIKE 'TIMESTAMP%' THEN TO_CHAR(f1(i_val), 'YYYY-MM-DD HH24:MI:SS')
END;
END fnc_raw;
/
```

아래는 fnc&#95;raw 함수를 사용한 쿼리다. &#42;&#95;TAB&#95;COLUMNS 뷰의 low&#95;value 열은 RAW 타입으로 생성되어 있다. c1 열에서 RAW 값이 문자 값으로 변환된 것을 확인할 수 있다.  

```sql
SELECT column_name, data_type, low_value, fnc_mac(data_type, low_value) AS c1
FROM user_tab_columns a
WHERE table_name = 'EMP'
ORDER BY column_id;
```

### 32.8. UTL&#95;MATCH 패키지
<br/>
UTL&#95;MATCH 패키지는 문자열 일치와 관련된 기능을 제공한다. 해당 패키지로 문자열의 유사도 분석을 수행할 수 있다.  

#### 32.8.1. EDIT&#95;DISTANCE 함수
<br/>
EDIT&#95;DISTANCE 함수는 s1을 s2로 변환하는 데 필요한 edit distance 값을 반환한다. edit distance는 문자열을 다른 문자열로 변환하는데 필요한 최소의 작업 횟수다.  

```
UTL_MATCH.EDIT_DISTANCE(s1 IN VARCHAR2, s2 IN VARCHAR2) RETURN PLS_INTEGER;
```

아래는 EDIT&#95;DISTANCE 함수를 사용한 쿼리다. ABCDEF 문자열을 A1C2E3 문자열로 변환하기 위해서는 3번의 작업이 필요하다.  

```sql
SELECT UTL_MATCH.EDIT_DISTANCE('ABCDEF', 'A1C2E3') AS c1 FROM DUAL;
```

#### 32.8.2. EDIT&#95;DISTANCE&#95;SIMILARITY 함수
<br/>
EDIT&#95;DISTANCE&#95;SIMILARITY 함수는 edit distance 값으로 s1과 s2의 유사도를 계산한다. 0(일치하지 않음) <= x <= 100(완전 일치) 범위의 값이 반환된다.  

```
UTL_MATCH.EDIT_DISTANCE_SIMILARITY (s1 IN VARCHAR2, s2 IN VARCHAR2) RETURN PLS_INTEGER;
```

아래는 EDIT&#95;DISTANCE&#95;SIMILARITY 함수를 사용한 쿼리다. 6문자 중 3문자를 변경해야 하기 때문에 유사도가 50으로 계산된다.  

```sql
SELECT UTL_MATCH.EDIT_DISTANCE_SIMILARITY('ABCDEF', 'A1C2E3') AS c1 FROM DUAL;
```

#### 32.8.3. JARO&#95;WINKLER 함수
<br/>
JARO&#95;WINKLER 함수는 Jaro-Winkler 알고리즘으로 계산한 edit distance 값을 반환한다. Jaro-Winkler 알고리즘은 문자열의 유사도를 측정하기 위한 알고리즘이다.  

```
UTL_MATCH.JARO_WINKLER (s1 IN VARCHAR2, s2 IN VARCHAR2) RETURN BINARY_DOUBLE;
```

아래는 JARO&#95;WINKLER 함수를 사용한 쿼리다.  

```sql
SELECT UTL_MATCH.JARO_WINKLER('ABCDEF', 'A1C2E3') AS c1 FROM DUAL;
```

#### 32.8.4. JARO&#95;WINKLER&#95;SIMILARITY 함수
<br/>
JARO&#95;WINKLER&#95;SIMILARITY 함수는 Jaro-Winkler 알고리즘으로 edit distance 값으로 s1과 s2의 유사도를 계산한다. 0 (일치하지 않음) <= x <= 100 (완전 일치) 범위의 값이 반환된다.  

```
UTL_MATCH.JARO_WINKLER_SIMILARITY (s1 IN VARCHAR2, s2 IN VARCHAR2) RETURN PLS_INTEGER;
```

아래는 JARO&#95;WINKLER&#95;SIMILARITY 함수를 사용한 쿼리다. JARO&#95;WINKLER 함수의 결과를 백분위 값으로 계산한 값이 반환된다.  

```sql
SELECT UTL_MATCH.JARO_WINKLER_SIMILARITY ('ABCDEF', 'A1C2E3') AS c1 FROM DUAL;
```

### 32.9. UTL&#95;RECOMP 패키지
<br/>
UTL&#95;RECOMP 패키지는 무효화된 데이터베이스 오브젝트를 재컴파일한다. 뷰, 시노님, PL/SQL 모듈 등이 재컴파일된다.  

#### 32.9.1. RECOMP&#95;SERIAL 프로시저
<br/>
RECOMP&#95;SERIAL 프로시저는 무효화된 오브젝트를 재컴파일한다. schema에 널을 지정하면 데이터베이스 레벨로 동작한다.  

```
UTL_RECOMP.RECOMP_SERIAL (
    schema IN VARCHAR2 DEFAULT NULL
  , flags IN BINARY_INTEGER DEFAULT 0;
);
```

RECOMP&#95;SERIAL 프로시저는 아래와 같이 동작한다.  

<table>
  <thead>
    <tr>
      <td>UTL&#95;RECOMP.RECOMP&#95;SERIAL</td>
      <td>설명</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>('SCOTT');</td>
      <td>SCOTT 스키마의 INVALID 오브젝트를 재컴파일</td>
    </tr>
    <tr>
      <td>();</td>
      <td>데이터베이스의 INVALID 오브젝트를 재컴파일</td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 32.9.2. RECOMP&#95;PARALLEL 프로시저
<br/>
RECOMP&#95;PARALLEL 프로시저는 무효화된 오브젝트를 병렬로 재컴파일한다. threads에 병렬도를 지정할 수 있으며, 널을 지정하면 job&#95;queue&#95;processes 파라미터의 값이 사용된다.  

```
UTL_RECOMP.RECOMP_PARALLEL (
    threads IN BINARY_INTEGER DEFAULT NULL
  , schema IN VARCHAR2 DEFAULT NULL
  , flags IN BINARY_INTEGER DEFAULT 0
);
```

RECOMP&#95;PARALLEL 프로시저는 아래와 같이 동작한다.  

<table>
  <thead>
    <tr>
      <td>UTL&#95;RECOMP.RECOMP&#95;SERIAL</td>
      <td>설명</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>(2, 'SCOTT');</td>
      <td>SCOTT 스키마를 2개의 스레드로 재컴파일</td>
    </tr>
    <tr>
      <td>(2);</td>
      <td>데이터베이스를 2개의 스레드로 재컴파일</td>
    </tr>
    <tr>
      <td>(NULL, 'SCOTT');</td>
      <td>SCOTT 스키마를 job&#95;queue&#95;processes개의 스레드로 재컴파일</td>
    </tr>
    <tr>
      <td>(2);</td>
      <td>데이터베이스를 job&#95;queue&#95;processes개의 스레드로 재컴파일</td>
    </tr>
  </tbody>
</table>
<br/><br/>
