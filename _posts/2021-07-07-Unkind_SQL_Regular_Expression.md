---
title:  정규 표현식
categories:
- Unkind_SQL
feature_text: |
  ## 24. 정규 표현식
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

정규 표현식은 문자열의 규칙을 표현하는 검색 패턴으로 주로 문자열 검새과 치환에 사용된다. 오라클 데이터베이스는 10.1 버전부터 정규 표현식을 지원한다.  

### 24.1. 기본 문법
<br/>
#### 24.1.1. POSIX 연산자
<br/>
오라클 데이터베이스는 정규 표현식의 POSIX 연산자(POSIX operator)를 지원한다.  

POSIX는 Portable Operating System Interface의 약자로 시스템 간 호환성을 위해 미리 정의된 인터페이스를 의미한다.  

##### 24.1.1.1. 기본 연산자  
<br/>
아래는 정규 표현식의 기본 연산자다.  

<table>
  <thead>
    <tr>
      <td>연산자</td>
      <td>영문</td>
      <td>설명</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>.</td>
      <td>dot</td>
      <td>모든 문자와 알치 (newline 제외)</td>
    </tr>
    <tr>
      <td>|</td>
      <td>or</td>
      <td>대체 문자를 구분</td>
    </tr>
    <tr>
      <td>\\</td>
      <td>backslash</td>
      <td>다음 문자를 일반 문자로 처리</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래는 dot 연산자를 사용한 쿼리다. REGEXP&#95;SUBSTR 함수는 문자열에서 일치하는 패턴을 반환한다. 조금 후에 자세히 살펴보자. c1, c2, c3, c4 열의 dot 연산자는 각각 a, b, c, d 문자와 일치한다. c4 열은 세 번째 문자인 c가 패턴의 세 번째 문자인 b와 일치하지 않기 때문에 널을 반환한다.  

```sql
SELECT REGEXP_SUBSTR('aab', 'a.b') AS c1
     , REGEXP_SUBSTR('abb', 'a.b') AS c2
     , REGEXP_SUBSTR('acb', 'a.b') AS c3
     , REGEXP_SUBSTR('abc', 'a.b') AS c4
FROM DUAL;
```

아래는 or 연산자를 사용한 쿼리다. c3 열은 c는 a 또는 b와 일치하지 않고, c6 열의 bc는 ab 또는 cd와 일치하지 않기 때문에 널을 반환한다. or 연산자는 기술 순서에 따라 패턴을 일치시킨다. c7 열은 a, c8 열은 aa가 일치한다.  

```sql
SELECT REGEXP_SUBSTR('a', 'a|b') AS c1 -- a 또는 b
     , REGEXP_SUBSTR('b', 'a|b') AS c2
     , REGEXP_SUBSTR('c', 'a|b') AS c3
     , REGEXP_SUBSTR('ab', 'ab|cd') AS c4 -- ab 또는 cd
     , REGEXP_SUBSTR('cd', 'ab|cd') AS c5
     , REGEXP_SUBSTR('bc', 'ab|cd') AS c6
     , REGEXP_SUBSTR('aa', 'a|aa') AS c7 -- a 또는 aa
     , REGEXP_SUBSTR('aa', 'aa|a') AS c8
FROM DUAL;
```

아래는 backslash 연산자를 사용한 쿼리다. c1 열은 | 문자가 or 연산자로 동작하여 a 문자가 일치하지만, c2 열은 | 문자가 일반 문자로 처리되어 a|b 문자열이 일치한다.  

```sql
SELECT REGEXP_SUBSTR('a|b', 'a|b') AS c1
     , REGEXP_SUBSTR('a|b', 'a\|b') AS c2
FROM DUAL;
```

##### 24.1.1.2. 앵커
<br/>
앵커(anchor)는 검색 패턴의 시작과 끝을 지정한다. 정규 표현식의 행 처리 방시에 따라 일치 범위가 달라진다. 문자열에 다중 행인 경우 단일 행 방식은 문자열을 하나의 행으로, 다중 행 방식은 각각의 행을 하나의 행으로 처리한다.  

<table>
  <thead>
    <tr>
      <td>연산자</td>
      <td>영문</td>
      <td>단일 행 방식</td>
      <td>다중 행 방식</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>^</td>
      <td>carrot</td>
      <td>문자열의 시작</td>
      <td>행의 시작</td>
    </tr>
    <tr>
      <td>$</td>
      <td>dollar</td>
      <td>문자열의 끝</td>
      <td>행의 끝</td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래는 carrot 연산자를 사용한 쿼리다. CHR(10) 표현식은 개행(line feed) 문자를 반환한다. c1, c2, c3, c4 열은 모두 문자열의 첫 번째 자리부터 검색을 수행한다. c1, c2 열은 단일 행 방식으로 동작한다. c1 열은 전체 문자열의 시작 문자인 a, c2 열은 전체 문자열의 두 번째 시작 문자가 존재하지 않으므로 널을 반환한다. c3 열과 c4 열은 REGEXP_SUBSTR 함수의 일치 옵션에 m을 기술하여 다중 행 방식으로 동작한다. c3 열은 첫째 줄의 시작 문자인 a, c4 열은 둘째 줄의 시작 문자인 c를 반환한다.  

```sql
SELECT REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '^.', 1, 1) AS c1
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '^.', 1, 2) AS c2
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '^.', 1, 1, 'm') AS c3
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '^.', 1, 2, 'm') AS c4
FROM DUAL;
```

아래는 dollar 연산자를 사용한 쿼리다. c1 열은 전체 문자열의 끝 문자인 d, c2 열은 전체 문자열의 두 번째 끝 문자가 존재하지 않기 때문에 널, c3 열은 첫째 줄의 끝 문자인 b, c4 열은 둘째 줄의 끝 문자인 d를 반환한다.  

```sql
SELECT REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '.$', 1, 1) AS c1
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '.$', 1, 2) AS c2
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '.$', 1, 1, 'm') AS c3
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '.$', 1, 2, 'm') AS c4
FROM DUAL;
```

##### 24.1.1.3. 수량사
<br/>
수량사(quantifier)는 선행 표현식의 일치 횟수를 지정한다. 패턴을 최대로 일치시키는 탐욕적(greedy) 방식으로 동작한다.  

+ ? : 0회 또는 1회 일치
+ &#42; : 0회 또는 그 이상의 횟수로 일치
+ + : 1회 또는 그 이상의 횟수로 일치
+ {m} : m회 일치
+ {m,} : 최소 m회 일치
+ {,m} : 최대 m회 일치
+ {m,n} : 최소 m회, 최대 n회 일치  

아래는 ?, &#42;, + 연산자를 사용한 쿼리다. 주석에 일치할 수 있는 문자열을 기술했다. c3 열은 b가 0회 또는 1회 일치해야 하기 때문에 널을 반환한다.  

```sql
SELECT REGEXP_SUBSTR('ac', 'ab?c') AS c1 -- ac, abc
     , REGEXP_SUBSTR('abc', 'ab?c') AS c2
     , REGEXP_SUBSTR('abbc', 'ab?c') AS c3
     , REGEXP_SUBSTR('ac', 'ab*c') AS c4 -- ac, abc, abbc, abbbc, ...
     , REGEXP_SUBSTR('abc', 'ab?c') AS c5
     , REGEXP_SUBSTR('abbc', 'ab*c') AS c6
     , REGEXP_SUBSTR('ac', 'ab+c') AS c7 -- abc, abbc, abbbc, abbbbc, ...
     , REGEXP_SUBSTR('abc', 'ab+c') AS c8
     , REGEXP_SUBSTR('abbc', 'ab+c') AS c9
FROM DUAL;
```

아래는 {m}, {m,}, {m,n} 연산자를 사용한 쿼리다. c5 열은 a가 최소 4회, 최대 5회 일치해야 하기 때문에 널을 반환한다.  

```sql
SELECT REGEXP_SUBSTR('ab', 'a{2}') AS c1 -- aa
     , REGEXP_SUBSTR('aab', 'a{2}') AS c2
     , REGEXP_SUBSTR('aab', 'a{3,}') AS c3 -- aaa, aaaa, ...
     , REGEXP_SUBSTR('aaab', 'a{3,}') AS c4
     , REGEXP_SUBSTR('aaab', 'a{4,5}') AS c5 -- aaaa, aaaaa
     , REGEXP_SUBSTR('aaaab', 'a{4,5}') AS c6
FROM DUAL;
```

##### 24.1.1.4. 서브 표현식
<br/>
서브 표현식(subexpression)은 표현식을 소괄호로 묶은 표현식이다. 서브 표현식은 하나의 단위로 처리된다.  

+ (epxr) : 괄호 안의 표현식을 하나의 단위로 취급  

아래는 서브 표현식을 사용한 쿼리다. c1 열은 ab, c2 열은 b가 1회 이상 반복된다. c3 열은 b 또는 c, c4 열은 ab 또는 cd가 대체된다.  

```sql
SELECT REGEXP_SUBSTR('ababc', '(ab)+c') AS c1 -- ab, ababc, ...
     , REGEXP_SUBSTR('ababc', 'ab+c') AS c2 -- abc, abbc, ...
     , REGEXP_SUBSTR('abd', 'a(b|c)d') AS c3 -- abc, acd
     , REGEXP_SUBSTR('abd', 'ab|cd') AS c4 -- ab, cd
FROM DUAL;
```

##### 24.1.1.5. 역 참조
<br/>
역 참조(back reference)을 사용하면 일치한 서브 표현식을 다시 참조할 수 있다. 반복되는 패턴을 검색하거나 서브 표현식의 위치를 변경하는 용도로 사용할 수 있다.  

+ \n : n번째 서브 표현식과 일치, n은 1에서 9 사이의 정수  

아래는 역 참조를 사용한 쿼리다. c1, c2, c3 열의 패턴은 abxab, cdxcd 문자열과 일치한다. c4, c5, c6 열은 동일한 문자열이 1회 이상 반복되는 패턴을 검색한다.  

```sql
SELECT REGEXP_SUBSTR('abxab', '(ab|cd)x\1') AS c1 -- abxab, cdxcd
     , REGEXP_SUBSTR('cdxcd', '(ab|cd)x\1') AS c2
     , REGEXP_SUBSTR('abxef', '(ab|cd)x\1') AS c3
     , REGEXP_SUBSTR('ababab', '(.*)\1+') AS c4
     , REGEXP_SUBSTR('abcabc', '(.*)\1+') AS c5
     , REGEXP_SUBSTR('abcabd', '(.*)\1+') AS c6
FROM DUAL;
```

##### 24.1.1.6. 문자 리스트
<br/>
문자 리스트(character list)ㄴ,ㄴ 문자를 대괄호로 묶은 표현식이다. 문자 리스트 중 1문자만 일치하면 패턴이 일치한 것으로 처리된다. 문자 리스트에서 하이픈(-)은 범위 연산자로 동작한다.  

+ [char...] : 문자 리스트 중 1문자와 일치
+ [^char...] : 문자 리스트에 포함되지 않은 한 문자와 일치  

아래는 문자 리스트를 사용한 쿼리다. c1, c2, c3 열의 패턴은 ac, bc 문자열과 일치한다. c4, c5, c6 열의 패턴은 ac, bc가 아닌 문자열과 일치한다.  

```sql
SELECT REGEXP_SUBSTR('ac', '[ab]c') AS c1 -- ac, bc
     , REGEXP_SUBSTR('bc', '[ab]c') AS c2
     , REGEXP_SUBSTR('cc', '[ab]c') AS c3
     , REGEXP_SUBSTR('ac', '[^ab]c') AS c4 -- ac, bc가 아닌 문자열
     , REGEXP_SUBSTR('bc', '[^ab]c') AS c5
     , REGEXP_SUBSTR('cc', '[^ab]c') AS c6
FROM DUAL;
```

아래는 문자 리스트의 범위 연산자를 사용한 쿼리다. c4 열은 첫 번째 문자는 숫자가 아닌 문자, 두 번째 문자는 소문자가 아닌 문자와 일치해야 하기 때문에 널을 반환한다.  

```sql
SELECT REGEXP_SUBSTR('1a', '[0-9][a-z]') AS c1
     , REGEXP_SUBSTR('9z', '[0-9][a-z]') AS c2
     , REGEXP_SUBSTR('aA', '[^0-9][^a-z]') AS c3
     , REGEXP_SUBSTR('Aa', '[^0-9][^a-z]') AS c4
FROM DUAL;
```

##### 24.1.1.7. POSIX 문자 클래스
<br/>
오라클 정규 표현식은 POSIX 문자 클래스를 지원한다. POSIX 문자 클래스는 문자 리스트에 사용해야 한다.  

<table>
  <thead>
    <tr>
      <td>문자 클래스</td>
      <td>설명</td>
      <td>동일</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>[:digin:]</td>
      <td>숫자</td>
      <td>[0-9]</td>
    </tr>
    <tr>
      <td>[:lower:]</td>
      <td>소문자</td>
      <td>[a-z]</td>
    </tr>
    <tr>
      <td>[:upper:]</td>
      <td>대문자</td>
      <td>[A-Z]</td>
    </tr>
    <tr>
      <td>[:alpha:]</td>
      <td>영문자</td>
      <td>[a-zA-Z]</td>
    </tr>
    <tr>
      <td>[:alnum:]</td>
      <td>영문자와 숫자</td>
      <td>[0-9a-zA-Z]</td>
    </tr>
    <tr>
      <td>[:xdigit:]</td>
      <td>16진수</td>
      <td>[0-9a-fA-F]</td>
    </tr>
    <tr>
      <td>[:punct:]</td>
      <td>구두점 기호</td>
      <td>[&[:alnum:][:cntrl:]]</td>
    </tr>
    <tr>
      <td>[:blank:]</td>
      <td>공백 문자</td>
      <td></td>
    </tr>
    <tr>
      <td>[:space:]</td>
      <td>공간 문자 (space, enter, tab)</td>
      <td></td>
    </tr>
    <tr>
      <td>[:cntrl:]</td>
      <td>제어 문자 (아스키 < 32, 아스키 > 126)</td>
      <td></td>
    </tr>
    <tr>
      <td>[:print:]</td>
      <td>출력이 가능한 모든 문자 (아스키 32 ~ 126)</td>
      <td></td>
    </tr>
    <tr>
      <td>[:graph:]</td>
      <td>[:print:]에서 space 제외</td>
      <td></td>
    </tr>
  </tbody>
</table>
<br/><br/>

아래는 POSIX 문자 클래스를 사용한 쿼리다. c6 열은 [:punct] 클래스를 사용하여 특수 문자를 검색한다.  

```sql
SELECT REGEXP_SUBSTR('gF1', '[[:digit:]]') AS c1
     , REGEXP_SUBSTR('gF1', '[[:alpha:]]') AS c2
     , REGEXP_SUBSTR('gF1', '[[:lower:]]') AS c3
     , REGEXP_SUBSTR('gF1', '[[:upper:]]') AS c4
     , REGEXP_SUBSTR('gF1', '[[:alnum:]]') AS c5
     , REGEXP_SUBSTR('gF1', '[[:xdigit:]]') AS c6
     , REGEXP_SUBSTR('gF1', '[[:punct:]]') AS c7
FROM DUAL;
```

아래는 [:blank:], [:space:] 문자 클래스를 사용한 쿼리다. CHR(9) 표현식은 탭(tab) 문자를 반환한다. [:blank:] 클래스에 탭이 포함되어 있지 않기 때문에 c2 열은 널을 반환한다.  

```sql
SELECT REGEXP_SUBSTR('a b', 'a[[:blank:]]b') AS c1
     , REGEXP_SUBSTR('a' || CHR(9) || 'b', 'a[[:blank:]]b') AS c2
     , REGEXP_SUBSTR('a b', 'a[[:space:]]b') AS c3
     , REGEXP_SUBSTR('a' || CHR(9) || 'b', 'a[[:space:]]b') AS c4
FROM DUAL;
```

아래는 [:cntrl:], [:print:], [:graph:] 문자 클래스를 사용한 쿼리다. c2 열에서 [:print:] 클래스에 공백 문자가 포함된 것을 확인할 수 있다.  

```sql
SELECT REGEXP_SUBSTR('a b', 'a[[:cntrl:]]b') AS c1
     , REGEXP_SUBSTR('a b', 'a[[:print:]]b') AS c2
     , REGEXP_SUBSTR('a b', 'a[[:graph:]]b') AS c3
FROM DUAL;
```

#### 24.1.2. PERL 정규 표현식 연산자
<br/>
오라클 데이터베이스는 PERL 정규 표현식 연산자(PERL, regular expression operator)를 지원한다. 펄(Perl)은 래리 펄(Larry Wall)이 1987년에 개발한 인터프리터 방식의 프로그래밍 언어다.  

아래의 PERL 정규 표현식 연산자는 POSIX 문자 클래스와 유사하게 동작한다.  

<table>
  <thead>
    <tr>
      <td>연산자/td>
      <td>설명</td>
      <td>동일</td>
    </tr>
  </thead>
  <thead>
    <tr>
      <td>\d</td>
      <td>숫자</td>
      <td>[[:digit:]]</td>
    </tr>
    <tr>
      <td>\D</td>
      <td>숫자가 아닌 모든 문자</td>
      <td>[^[:digit:]]</td>
    </tr>
    <tr>
      <td>\w</td>
      <td>숫자와 영문자 (underbar 포함)</td>
      <td>[[:alnum]]</td>
    </tr>
    <tr>
      <td>\W</td>
      <td>숫자와 영문자가 아닌 모든 문자 (underbar 제외)</td>
      <td>[^[:alnum:]]</td>
    </tr>
    <tr>
      <td>\s</td>
      <td>공백 문자</td>
      <td>[[:space]]</td>
    </tr>
    <tr>
      <td>\S</td>
      <td>공백 문자가 아닌 모든 문자</td>
      <td>[^[:space:]]</td>
    </tr>
  </thead>
</table>
<br/><br/>

아래는 \d 연산자와 \D 연산자를 사용한 쿼리다. c1, c2열의 패턴은 '(숫자3자리) 숫자3자리-숫자4자리' 패턴의 전화번호를 검색한다. c5 열은 세 번째 문자인 2가 숫자이므로 널을 반환한다.  

```sql
SELECT REGEXP_SUBSTR('(650) 555-0100', '^\(\d{3}\) \d{3}-d{4}$') AS c1
     , REGEXP_SUBSTR('650 555-0100', '^\(\d{3}\) \d{3}-d{4}$') AS c2
     , REGEXP_SUBSTR('b2b', '\w\d\D') AS c3
     , REGEXP_SUBSTR('b2_', '\w\d\D') AS c4
     , REGEXP_SUBSTR('b22', '\w\d\D') AS c5
FROM DUAL;
```

아래는 \w 연산자와 \W 연산자를 사용한 쿼리다. c1, c2 열의 패턴은 '숫자문자@숫자문자' 다음에 '.숫자문자'가 1회 이상 반복되는 패턴의 이메일 주소를 검색한다. c4 열은 \W에 해당하는 문자가 문자열에 존재하지 않기 때문에 널을 반환한다.  

```sql
SELECT REGEXP_SUBSTR('jdoe@company.co.uk', '\w+@\w+(\.\w+)+') AS c1
     , REGEXP_SUBSTR('jdoe@company', '\w+@\w+(\.\w+)+') AS c2
     , REGEXP_SUBSTR('to: bill', '\w+\W\s\w+') AS c3
     , REGEXP_SUBSTR('to bill', '\w+\W\s\w+') AS c4
FROM DUAL;
```

아래는 \s 연산자와 \S 연산자를 사용한 쿼리다. c1 열은 \s 연산자가 공백 문자, c4 열은 \S 연산자가 공백이 아닌 문자(.)와 일치한다.  

```sql
SELECT REGEXP_SUBSTR('(a b )', '\(\w\s\w\s)') AS c1
     , REGEXP_SUBSTR('(a b )', '\(\w\s\w\S)') AS c2
     , REGEXP_SUBSTR('(a,b.)', '\(\w\s\w\s)') AS c3
     , REGEXP_SUBSTR('(a,b.)', '\(\w\s\w\S)') AS c4
FROM DUAL;
```

아래의 PERL 정규 표현식 연산자는 앵커와 유사하게 동작한다. 다중 행 모드를 좀 더 세밀하게 처리할 수 있다.  

+ \A : 다중 행 모드와 관계없이 문자열의 시작과 일치
+ \Z : 다중 행 모드와 관계없이 문자열의 끝과 일치 (개행 문자면 앞 문자)
+ \z : 다중 행 모드와 관계없이 문자열의 끝과 일치 (개행 문자면 널)  

아래는 \A 연산자를 사용한 쿼리다. c3 열은 다중 행 모드와 관계없이 문자열의 시작과 일치하는 패턴을 검색하기 때문에 널을 반환한다. c4 열은 두 번째 행의 시작 문자인 b를 반환한다.  

```sql
SELECT REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '\A\w', 1, 1, 'm') AS c1
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '^\w', 1, 1, 'm') AS c2
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '\A\w', 1, 2, 'm') AS c3
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '^\w', 1, 2, 'm') AS c4
FROM DUAL;
```

아래는 \Z 연산자와 \z 연산자를 사용한 쿼리다. 전체 문자열의 끝 문자가 개행 문자이므로 c1 열은 b, c2 열은 널을 반환한다. c3, c4 열을 각각 첫째 줄과 둘쩨 줄의 끝 문자를 반환한다.  

```sql
SELECT REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '\w\Z', 1, 1, 'm') AS c1
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '\w\z', 1, 1, 'm') AS c2
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '\w\Z', 1, 2, 'm') AS c3
     , REGEXP_SUBSTR('ab' || CHR(10) || 'cd', '\w\z', 1, 2, 'm') AS c4
FROM DUAL;
```

아래의 PERL 정규 표현식 연산자는 수량사와 유사하게 동작한다. 패턴을 최소로 일치시키는 비탐욕적(nongreedy) 방식으로 동작한다.  

+ ?? : 0회 또는 1회 일치
+ &#42;? : 0회 또는 그 이상의 횟수로 일치
+ +? : 1회 또는 그 이상의 횟수로 일치
+ {m}? : m회 일치
+ {m,}? : 최소 m회 일치
+ {,m}? : 최대 m회 일치
+ {m,n}? : 최소 m회, 최대 n회 일치  

아래는 ??, &#42;?, +? 연산자를 사용한 쿼리다. c1 열은 패턴을 최소로 일치시키는 nongreedy 방식을 사용했기 때문에 aa까지만 일치한다. 패턴을 최대로 일치시키는 greedy 방식을 사용한 c2 열은 aaa까지 일치한다.  

```sql
SELECT REGEXP_SUBSTR('aaaa', 'a??aa') AS c1 -- nongreedy 방식
     , REGEXP_SUBSTR('aaaa', 'a?aa') AS c2 -- greedy 방식
     , REGEXP_SUBSTR('xaxbxc', '\w*?x\w') AS c3
     , REGEXP_SUBSTR('xaxbxc', '\w*x\w') AS c4
     , REGEXP_SUBSTR('abxcxd', '\w+?x\w') AS c5
     , REGEXP_SUBSTR('abxcxd', '\w+x\w') AS c6
FROM DUAL;
```

아래는 {m}?, {m,}?, {m,n}? 연산자를 사용한 쿼리다. c1, c2 열은 m회 일치를 사용했기 때문에 greedy 방식과 관계없이 결과가 동일하다. c3 열은 최소 일치므로 aa, c4 열은 최대 일치이므로 aaaaa와 일치한다. c5 열은 최소로 aa, c6 열은 최대로 aaaa와 일치한다.  

```sql
SELECT REGEXP_SUBSTR('aaaa', 'a{2}?') AS c1 -- nongreedy 방식
     , REGEXP_SUBSTR('aaaa', 'a{2}') AS c2 -- greedy 방식
     , REGEXP_SUBSTR('aaaaa', 'a{2,}?') AS c3
     , REGEXP_SUBSTR('aaaaa', 'a{2,}') AS c4
     , REGEXP_SUBSTR('aaaaa', 'a{2,4}?') AS c5
     , REGEXP_SUBSTR('aaaaa', 'a{2,4}') AS c6
FROM DUAL;
```

### 24.2. 정규 표현식 조건과 함수
<br/>
#### 24.2.1. REGEXP&#95;LIKE 조건
<br/>
REGEXP&#95;LIKE 조건은 source&#95;char가 pattern과 일치하면 TRUE, 일치하지 않으면 FALSE를 반환한다.  

```
REGEXP_LIKE (source_char, paterrn [, match_param])
```

+ source&#95;char : 검색 문자열
+ pattern : 검색 패턴
+ match&#95;param : 일치 옵션  

아래의 일치 옵션을 사용할 수 있다. 기본값은 c다. icnmx 형식으로 다수의 옵션을 함께 지정할 수도 있다.  

+ i : 대소문자 무시  
+ c : 대소문자 구분
+ n : dot(.)를 개행 문자와 일치
+ m : 다중 행 모드 (앵커(^, $)에 영향)
+ x : 검색 패턴의 공백 문자를 무시  

아래는 REGEXP&#95;LIKE 조건을 사용한 쿼리다. first&#95;name 값이 Ste로 시작하고 v나 ph 다음에 en으로 끝나는 행을 검색한다.  

```sql
SELECT first_name, last_name
FROM hr.employees
WHERE REGEXP_LIKE (first_name, '&Ste(v|ph)en$');
```

아래 쿼리는 last&#95;name 값에 [aeiou]가 연속되는 패턴이 존재하는 행을 검색한다. Bloom은 o가 2회 반복된다.  

```sql
SELECT last_name FROM hr.employees WHERE REGEXP_LIKE (last_name, '([aeiou])\1', 'i');
```

CHECK 제약 조건에 REGEXP_LIKE 함수를 활용할 수 있다. 아래의 CHECK 제약 조건은 유효한 전화번호와 패턴을 검사한다. p&#95;number 열에 '(숫자3자리) 숫자3자리-숫자4자리' 패턴과 일치하는 전화번호만 저장된다.  

```sql
DROP TABLE contacts PURGE;

CREATE TABLE contacts (
    l_name VARCHAR2(30)
  , p_number VARCHAR2(30)
  , CONSTRAINT c_contacts_pnf CHECK (REGEXP_LIKE(p_number, '&\(\d{3}\) \d{3}-\d{4}$')));
```

아래에서 첫 번째, 두 번째를 제외한 나머지 쿼리는 "ORA-02290: 체크 제약조건 (SCOTT.C&#95;CONTACTS&#95;PNF)이 위배되었습니다" 에러가 발생한다.  

```sql
INSERT INTO contacts (p_number) VALUES ('(650) 555-0100');
INSERT INTO contacts (p_number) VALUES ('(215) 555-0100');
INSERT INTO contacts (p_number) VALUES ('650 555-0100'); -- ORA-02290
INSERT INTO contacts (p_number) VALUES ('650 555-0100'); -- ORA-02290
INSERT INTO contacts (p_number) VALUES ('650 555-0100'); -- ORA-02290
INSERT INTO contacts (p_number) VALUES ('(650)555-0100'); -- ORA-02290
INSERT INTO contacts (p_number) VALUES (' (650) 555-0100'); -- ORA-02290
```

#### 24.2.2. REGEXP&#95;REPLACE 함수
<br/>
REGEXP&#95;REPLACE 함수는 source&#95;char에서 일치한 pattern을 replace&#95;string으로 변경한 문자 값을 반환한다.  

```
REGEXP_REPLACE(source_char, pattern, [, replace_string, [, position [, occurrence [, match_param]]]])
```

+ source&#95;char : 검색 문자열
+ pattern : 검색 패턴
+ replace&#95;string : 변경 문자열
+ position : 검색 시작 위치 (기본값은 1)
+ occurrence : 패턴 일치 횟수 (기본값은 1)
+ match&#95;param : 일치 옵션  

아래는 REGEXP&#95;REPLACE 함수를 사용한 쿼리다. '숫자3자리,숫자3자리,숫자4자리' 패턴을 '(첫 번째 일치) 두 번째 일치-세 번째 일치' 형식으로 변경한다. 일치한 패텅이 없으면 원본 값을 반환한다.  

```sql
SELECT phone_number
     , REGEXP_REPLACE(phone_number
                    , '([[:digit:]]{3})\.([[:digit:]]{3})\.([[:digit:]]{4})')
                    , '(\1) \2-\3') AS c1
FROM hr.employees
WHERE employee_id IN (144, 145);
```

아래 쿼리는 한 문자를 문자 뒤 공백이 있는 동일한 문자로 변경한다.  

```sql
SELECT country_name, REGEXP_REPLACE(country_name, '(.)', '\1') AS c1
FROM hr.countries
WHERE country_id LIKE 'A%';
```

아래 쿼리는 연속된 공백을 하나의 공백으로 변경한다. '(){2,}' 패턴 대신 ' +' 패턴을 사용해도 동일한 결과를 얻을 수 있다.  

```sql
SELECT REGEXP_REPLACE('500 Oracle Parkway, Redwood Shores, CA', '( ){2,}', ' ') AS c1
FROM DUAL;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE famous_people PURGE;
CREATE TABLE famous_people (names VARCHAR2(20));

INSERT INTO famous_people VALUES ('John Quincy Adams');
INSERT INTO famous_people VALUES ('Harry S. Truman');
INSERT INTO famous_people VALUES ('John Adams');
INSERT INTO famous_people VALUES (' John Quincy Adams');
INSERT INTO famous_people VALUES ('John_Quincy_Adams');
COMMIT;
```

아래 쿼리는 역 참조를 사용하여 'first middle last' 패턴의 이름을 'last, first middle' 형식으로 변경한다.  

```sql
SELECT names, REGEXP_REPLACE(names, '^(\S+)\s(\S+)\s(\S+)$', '\3, \1 \2') AS c1
FROM famous_people;
```

#### 24.2.3. REGEXP&#95;SUBSTR 함수
<br/>
REGEXP&#95;SUBSTR 함수는 source&#95;char에서 일치한 pattern을 반환한다.  

```
REGEXP_SUBSTR (source_char, pattern [, position [, occurrence [, match_param [, subexpr]]]])
```

+ source&#95;char : 검색 문자열
+ pattern : 검색 패턴
+ position : 검색 시작 위치 (기본값은 1)
+ occurrence : 패턴 일치 횟수 (기본값은 1)
+ match&#95;param : 일치 옵션
+ subexpr : 서브 표현식 (0은 전체 패턴, 1 이상은 서브 표현식, 기본값은 0)  

아래는 REGEXP&#95;SUBSTR 함수를 사용한 쿼리다. 쉼표(,)로 구분된 문자열을 검색한다.  

```sql
SELECT REGEXP_SUBSTR('500 Oracle Parkway, Redwood Shores, CA', ',[^.]+,') AS c1
FROM DUAL;
```

아래 쿼리는 URL 패턴과 일치한 문자열을 반환한다.  

```sql
SELECT REGEXP_SUBSTR('http://www.example.com/products', 'http://([[:alnum:]]+\.?){3,4}/?') AS c1
FROM DUAL;
```

아래 쿼리는 일치한 서브 표현식을 반환한다. 강조한 부분이 일치한 서브 표현식이다. 서브 표현식의 순서는 좌측 괄호의 순서와 동일하다.  

```sql
SELECT REGEXP_SUBSTR('1234567890', '(123)(4(56)(78))', 1, 1, 'i', 1) AS c1
     , REGEXP_SUBSTR('1234567890', '(123)(4(56)(78))', 1, 1, 'i', 4) AS c2
FROM DUAL;
```

아래 쿼리는 일치 옵션을 사용한 쿼리다. c1 열은 dot(.)를 개행 문자와 일치시켰기 때문에 패턴이 일치한다. c2 열은 다중 행 모드로 동작하기 때문에 둘째 줄 ac와 일치한다. c3 열은 검색 패턴의 공백 문자를 무시하기 때문에 패턴이 일치한다.  

```sql
SELECT REGEXP_SUBSTR('a', || CHR(10) || 'd', 'a.d', 1, 1, 'n') AS c1
     , REGEXP_SUBSTR('ab', || CHR(10) || 'AC', '^a', 1, 2, 'm') AS c2
     , REGEXP_SUBSTR('abcd', 'a b c d', 1, 2, 'x') AS c3
FROM DUAL;
```

#### 24.2.4. REGEXP&#95;INSTR 함수
<br/>
REGEXP&#95;INSTR 함수는 source&#95;char에서 일치한 pattern의 시작 위치를 정수로 반환한다.  

```
REGEXP_INSTR (source_char, pattern [, position [, occurrence [, return_opt [, match_param [, subexpr]]]]])
```

+ source&#95;char : 검색 문자열
+ pattern : 검색 패턴
+ position : 검색 시작 위치 (기본값은 1)
+ occurrence : 패턴 일치 횟수 (기본값은 1)
+ return&#95;opt : 반환 옵션 (0은 시작 위치, 1은 다음 위치, 기본값은 0)
+ match&#95;param : 일치 옵션
+ subexpr : 서브 표현식 (0은 전체 패턴, 1 이상은 서브 표현식, 기본값은 0)  

아래는 REGEXP&#95;INSTR 함수를 사용한 쿼리다. 주석으로 문자열의 위치를 표시했다. 강조한 부분이 일치한 문자열이다. c1 열은 공백 문자로 구분된 여섯 번째 문자열(CA)의 시작 위치를 반환한다. c2 열은 s나 r이나 p로 시작하는 7자리 영문 문자열을 검색한다. 두 번째로 일치한 문자열(Redwood)의 다음 위치를 반환한다.  

```sql
SELECT REGEXP_INSTR ('500 Oracle Parkway, Redwood Shores, CA', '[^ ]+', 1, 6) AS c1
     , REGEXP_INSTR ('500 Oracle Parkway, Redwood Shores, CA'
                   , '[s|r|p][[:alpha:]]{6}', 3, 2, 1, 'i') AS c2
FROM DUAL;
```

아래 쿼리는 서브 표현식의 시작 위치를 반환한다. 강조한 부분이 일치한 서브 표현식이다.  

```sql
SELECT REGEXP_INSTR('1234567890', '(123)(4(56)(78))', 1, 1, 0, 'i', 1) AS c1
     , REGEXP_INSTR('1234567890', '(123)(4(56)(78))', 1, 1, 0, 'i', 2) AS c2
     , REGEXP_INSTR('1234567890', '(123)(4(56)(78))', 1, 1, 0, 'i', 4) AS c3
FROM DUAL;
```

#### 24.2.5. REGEXP&#95;COUNT 함수
<br/>
REGEXP&#95;COUNT 함수는 source&#95;char 일치한 pattern의 횟수를 반환한다.  

```
REGEXP_COUNT (source_char, pattern [, position [, match_param]])
```

+ source&#95;char : 검색 문자열
+ pattern : 검색 패턴
+ position : 검색 시작 위치 (기본값은 1)
+ match&#95;param : 일치 옵션  

아래는 REGEXP&#95;COUNT 함수를 사용한 쿼리다. 강조한 부분이 일치한 문자열이다. c1 열은 3회, c2 열은 3회 일치하다.  

```sql
SELECT REGEXP_COUNT('123123123123123', '123', 1) AS c1
     , REGEXP_COUNT('123123123123') AS c2
FROM DUAL;
```

아래는 일치 옵션을 사용한 쿼리다. 강조한 부분이 일치한 문자다. c1 열은 대소문자를 구분하여 않기 때문에 3회, c2 열은 대소문자를 구분하기 때문에 2회 일치한다.  

```sql
SELECT REGEXP_COUNT('Albert Einstein', 'e', 1, 'i') AS c1
     , REGEXP_COUNT('Albert Einstein', 'e', 1, 'c') AS c2
FROM DUAL;
```

### 24.3 활용 예제
<br/>
예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);

INSERT INTO t1 VALUES (1, '010-1234-5678');
INSERT INTO t1 VALUES (2, '010-123-4567');
INSERT INTO t1 VALUES (3, '010-123-456');
INSERT INTO t1 VALUES (4, '012-1234-5678');
COMMIT;
```

아래 쿼리는 REGEXP&#95;REPLACE 함수를 사용하여 휴대폰 번호의 두 번째 자리를 마스킹(masking)한다. 패턴과 일치하지 않는 휴대폰 번호는 마스킹되지 않는다.  

```sql
SELECT a.*
     , REGEXP_REPLACE(a.c2
                    , '^(010|011|016|017|019)-([0-9]{3,4})-([0-9]{4})$'
                    , '\1-****-\3') AS c3
FROM t1 a;
```

예제를 위해 아래와 같이 테이블을 생성하자.  

```sql
DROP TABLE t1 PURGE;
CREATE TABLE t1 (c1 NUMBER, c2 VARCHAR2(14));

INSERT INTO t1 VALUES (1, '790822-1234567');
INSERT INTO t1 VALUES (2, '791322-1234567');
INSERT INTO t1 VALUES (3, '790832-1234567');
INSERT INTO t1 VALUES (4, '790822-0123456');
INSERT INTO t1 VALUES (5, '790231-1234567');
COMMIT;
```

아래 쿼리는 REGEXP_REPLACE 함수를 사용하여 주민등록번호의 뒷자리를 마스킹한다. 패턴과 일치하지 않는 주민등록번호는 마스킹되지 않는다. 패턴의 한계로 인해 월에 대한 날짜 처리(790231)가 불가능하다.  

```sql
SELECT a.*
     , REGEXP_REPLACE(a.c2
                    , '^([0-9]{2})(0[1-9]|1[0-2])(0[1-9]|[12][0-9]|3[01])-([1-4])([0-9]{6})$'
                    , '\1\2\3-\4******') AS c3
FROM t1 a;
```

아래는 쉼표(,)로 구분된 문자열을 행으로 분리하는 쿼리다. 정규 표현식과 행 복제를 사용했다. 아래 쿼리 외에도 다양한 방식을 사용할 수 있다.  

```sql
WITH w1 AS (SELECT '1,2,3' AS c1 FROM DUAL)
SELECT REGEXP_SUBSTR(a.c1, '[^,]+', 1, b.lv) AS c1
FROM w1 a
   , (SELECT LEVEL AS lv FROM DUAL CONNECT BY LEVEL <= 10) b
WHERE b.lv <= REGEXP_COUNT(a.c1, ',') + 1;
```
