---
title:  URI 클래스
categories:
- Java Network Programming
feature_text: |
  ## URI 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

URI는 URL을 일반화(generalization)시킨 것이며 URL뿐만 아니라 URN(Uniform Resource Name)도 함께 포함한다. 실제로 사용되는 대부분의 URI는 URL이다. 그러나 XML과 같은 대부분의 스펙(specification)과 표준은 URI의 관점에서 정의된다. 자바에서 URI는 java.net.URI 클래스로 표현된다. 이 클래스는 다음 세 가지 점에서 java.net.URL 클래스와 구별된다.  


+ URI 클래스는 순수하게 리소스를 식별하고 URI를 분석하는 기능만 제공한다. URI가 참조하는 리소스를 가져오는 메소드는 제공하지 않는다.  

+ URI 클래스는 URL 클래스보다 관련된 스펙 및 표준 사항을 더 잘 따른다.  

+ URI 객체는 상대적인 URI를 표현할 수 있다. URL 클래스는 모든 URI를 저장하기 전에 절대 URL 형태로 변경한다.  

요약하자면, URL 객체는 네트워크 전송을 위한 애플리케이션 계층 프로토콜을 표현하는 객체인 반면, URI 객체는 순수하게 문자열 분석과 조작을 위한 객체이다. URI 클래스는 네트워크 전송을 위한 메소드가 없다. URL 클래스도 getFile()과 getRef() 같은 문자열 분석 메소드를 일부 제공하지만, 이들은 해당 스펙에 정의된 대로 정확하게 동작하지 않는다. 일반적으로 URL이 참조하는 콘텐츠를 전송할 경우 URL 클래스를 사용하고, 전송보다는 리소스를 식별할 목적으로 사용할 경우 URI 클래스를 사용한다. 예를 들어, XML 네임스페이스(namespace)를 표현하는 데 사용한다. URI와 URL 둘 모두 필요한 경우, URI의 toURL() 메소드와 URL의 toURI 메소드를 사용하여 상호 변환이 가능하다.  

### 1. URI 생성하기
<br/>
URI 객체는 문자열로부터 만든다. 전체 URI를 구성하는 단일 문자열을 생성자의 인자로 전달하거나 개별 요소를 전달할 수 있다.  

+ public URI(String uri) throws URISyntaxException
+ public URI(String scheme, String schemeSpecificPart, String fragment) throws URISyntaxException
+ public URI(String scheme, String host, String fragment) throws URISyntaxException
+ public URI(String scheme, String authority, String path, String query, String fragment) throws
+ public URI(String scheme, String userInfo, String host, String port, String path, String query, String fragment) throws URISyntaxException  

URL 클래스와는 달리 URI 클래스는 내장된 프로토콜 핸들러의 영향을 받지 않는다. URI가 문법적으로 문제가 없는 한, 자바가 URI 객체를 만들기 위해 URI의 프로토콜을 꼭 이해해야만 하는 것은 아니다. 그렇기 때문에 URL 클래스와는 달리 URI 클래스는 새로운 URI를 시험하는 용도로 사용될 수 있다.  

첫 번째 생성자는 적절한 문자열로부터 새 URI 객체를 생성한다. 예를 들어:  

```java
URI voice = new URI("tel:+1800-9988-9938");
URI web = new URI("http://www.xml.com/pub/a/2003/09/17/stax.html#id=_hbc");
URI book = new URI("urn:isbn:1-565-92870-9");
```

인자로 사용된 문자열이 URI 구문 규칙을 따르지 않는 경우 - 예를 들어, URI가 콜론으로 시작하는 경우 - 생성자는 URISyntaxException 예외를 발생시킨다. URISyntaxException는 확인 예외(checked exception)(코드에서 명시적인 예외 처리를 하지 않으면 컴파일이 허가되지 않음)이므로 처리(catch)하거나 생성자가 호출된 곳의 메소드가 이 예외를 던질(throw) 수 있도록 선언되어야 한다. 그러나 구문 규칙은 검사되지 않는다. URI 스펙과 달리 URI에서 사용할 수 있는 문자는 아스키로 제한되지 않는다. URI는 유니코드 문자도 포함할 수 있다. 문법적으로 URI는 제약 사항이 거의 없다. 특히 비아스키 문자를 인코딩할 필요가 없어졌으며 상대적인 URI도 허용된다. 거의 모든 문자열을 URI로 사용할 수 있다.  

스킴에 따라 다른 인자를 받는 두 번째 생성자는 주로 비계층형 URI에 사용한다. 스킴은 http, urn, tel과 같은 URI의 프로토콜을 말한다. 스킴은 아스키 문자, 숫자 그리고 더하기(+), 빼기(-), 마침표(.)를 조합해 만든다. 스킴의 첫 글자는 문자로 시작해야 한다. 스킴 인자에 널(null)을 전달하여 스킴을 생략하면 상대 URI를 만들 수 있다. 예를 들어:  

```java
URI absolute = new URI("http", "//www.ibiblio.org", null);
URI relative = new URI(null, "/javafaq/index.html", "today");
```

스킴에 따라 다른 인자를 받는 부분(scheme-specific part)은 URI 스킴의 구문 규칙에 따라 다르다. http URI인 경우와 mailto URIL인 경우, tel URI인 경우가 서로 다르다. URI 클래스는 이 부분에 허가되지 않은 문자가 입력될 경우 퍼센트 인코딩(percent-encoding)하기 때문에 사실상 구문 규칙 에러는 발생하지 않는다.  

마지막으로 세 번째 인자는 부위 지정자를 포함한다. 다시 말하지만 허가되지 않은 문자가 포함된 경우 자동으로 특수문자 처리(escaped)된다. 부위 지정자를 생략하고자 할 경우 간단히 널(null)을 인자로 전달하면 된다.  

세 변째 생성자는 http와 ftp URL 같은 계층적인 URI에 사용된다.  

이 생성자에서는 호스트와 경로(슬래시로 구분되는)가 URI의 스킴에 따라 다른 인자 부분을 구성한다. 예를 들어:  

```java
URI today = new URI("http", "www.ibiblio.org", "/javafaq/index.html", "today");
```

이 생성자는 http://www.ibiblio.org/javafaq/index.html#today URI를 생성한다.  

생성자가 인자로 전달된 요소들을 사용하여 정상적인 계층구조 URI를 생성할 수 없다면 - 예를 들어, 스킴이 존재할 경우 절대적인 URI가 될 수 있어야 하지만, 경로가 /로 시작하지 않는 경우 - URLSyntaxException 예외를 발생시킨다.  

네 번째 생성자는 기본적으로 세 번째 생성자와 같으며 쿼리 문자열이 추가되었다. 예를 들어:  

```java
URI today = new URI("http", "www.ibiblio.org", "/javafaq/index.html", "referrer=cnet&date=2014-02-23", "today");
```

이전과 마찬가지로 구문 규칙에 오류가 있는 경우 URISyntaxException이 발생하고 인자를 생략할 경우 널(null)을 전달하면 된다.  

다섯 번째 생성자는 이전의 계층적인 URI를 생성하는 두 생성자가 호출하는 기본 URI 생성자다. 이 생성자는 기관(authority)을 사용자 정보, 호스트 그리고 포트 부분으로 분리하며, 각 부분에는 각각의 구문 규칙이 있다. 예를 들어:  

```java
URI styles = new URI("ftp", "anonymous:elharo@ibiblio.org", "ftp.oreilly.com", "ftp.oreilly.com", 21, "/pub/stylesheet", null, null);
```

그러나 생성된 URI는 여전히 일반적인 URI 구문 규칙을 따라야 한다. 그리고 또다시 말하지만 생략할 인자에는 널(null)을 설정하면 된다.  

사용할 URI가 문법적으로 문제가 없고 어떠한 구문 규칙도 위반하지 않았다면, 대신 정적 팩토리 메소드인 URI.create()를 사용할 수 있다.  

이 메소드는 생성자와는 달리 URISyntaxException 예외를 발생시키지 않는다. 예를 들어, 아래 메소드 호출은 메일 주소를 패스워드로 사용하는 익명(anonymous) FTP 접근 URI를 생성한다.  

```java
URI styles = URI.create("ftp://anonymous:elharo%40ibiblio.org@ftp.oreilly.com:21/pub/stylesheet");
```

만약 인자로 전달된 URI에 문제가 있다고 판단될 경우, 이 메소드는 IllegalArgumentException 예외를 발생시킨다. 이 예외는 런타임 예외이므로 코드상에서 명시적으로 선언하거나 처리할 수 없다.  

### 2. URI 구성 요소
<br/>
URI 레퍼런스는 세 부분으로 구성된다. 스킴, 스킴에 따라 다른 부분, 그리고 부위 지정자, 일반적인 형식은 다음과 같다.  

scheme:scheme-specific-part:fragment  

스킴 부분이 생략될 경우, 이 URI 레퍼런스는 상대적인 URI이다. 부위 지정자가 생략될 경우, 이 URI는 순순한 URI이다(부위 지정자는 엄밀히 말하면 URI의 일부가 아니다.). URI 클래스는 각각의 URI 객체에 대해 이 세 부분을 반환하는 메소드를 제공한다.  

getXxx() 형태의 메소드는 먼저 퍼센트 인코딩된(percent-escaped) 문자를 디코딩한 후 각 부분을 반환하지만, getRawXxx() 형태의 메소드는 URI의 각 부분들을 인코딩된 형식으로 반환한다.  

+ public String getScheme()
+ public String getSchemeSpecificPart()
+ public String getRawSchemeSpecificPart()
+ public String getFragment()
+ public String getRawFragment()  

URI 스펙에는 URI에 허용된 아스키 문자만으로 스킴 이름을 구성하도록 요구하고, 스킴 이름에 대한 인코딩이 허가되지 않기 때문에 getRawScheme() 메소드는 존재하지 않는다.  

이 메소드들은 URI 객체에 관련된 구성 요소가 없는 경우 널(null)을 반환한다. 예를 들어, 스킴이 없는 상대적인 URI, 부위 지정자가 없는 http URI.  

스킴이 있는 URI는 절대적인 URI이다. 스킴이 없는 URI는 상대적인 URI이다. isAbsolute() 메소드는 URI가 절대적인 URI인 경우 true를 반환하고 상재거인 URI인 경우 false를 반환한다.  

+ public boolean isAbsolute()  

스킴에 따라 다른 부분의 구체적인 내용은 스킴의 타입에 따라 매우 다양하다. 예를 들어, tel URL에서 스킴에 따라 다른 부분은 전화번호 구문 규칙이 된다. 그러나 흔히 사용되는 file 그리고 http URL에서 스킴에 따라 다른 부분은 기관, 경로 그리고 쿼리 문자열로 구분되는 계층적인 형식을 가진다. 기관은 다시 사용자 정보, 호스트 그리고 포트로 나뉜다.  

isOpaque() 메소드는 URI가 계층적인 경우 false를 반환하고, 계층적이지 않을 경우 true를 반환한다. 즉, 불투명(opaque) URI인 경우 true를 반환한다.  

+ public boolean isOpaque()  

불토명 URI인 경우 스킴, 스킴에 따라 다른 부분, 그리고 부위 지정자에 대한 정보만 구할 수 있다. 계층적인 URI인 경우 계층적인 URI의 다른 모든 부분의 정보를 구할 수 있는 메소드가 제공된다.  

+ public String getAuthority()
+ public String getFragment()
+ public String getHost()
+ public String getPath()
+ public String getPort()
+ public String getQuery()
+ public String getUserInfo()  

이 메소드는 모두 해당 부분을 디코딩하여 반환한다. 즉, %3c와 같은 값이 원래 의미하는 <와 같은 값으로 반환된다. 만약 URI의 각 부분에 대해 디코딩 이전의 값이 필요한 경우 getRawXxx()와 같은 유사한 메소드가 제공된다.  

+ public String getRawAuthority()
+ public String getRawFragment()
+ public String getRawPath()
+ public String getRawQuery()
+ public String getRawUserInfo()  

URI 클래스는 URI 스펙에 명시된 것과는 달리 비아스키 문자를 먼저 인코딩하지 않는다. 그래서 URI 객체를 만들 때 애초에 인코딩된 문자열을 사용한 것이 아니라면 getRawXxx() 메소드로 반환받더라도 인코딩되지 않은 문자를 반환받게 된다.  

포트와 호스트 부분은 아스키 문자로만 구성되기 때문에 getRawPort() 그리고 getRawHost() 메소드는 존재하지 않는다.  

만약에 특정 URI가 이러한 정보를 포함하고 있지 않다면 - 예를 들어, 다음 http://www.example.com URI의 경우 사용자 정보, 결로, 포트 또는 쿼리가 없다 - 관련된 메소드는 널(null)을 반환한다. getPort() 메소드의 경우 예외다. 이 메소드가 정수(int)값을 반환하도록 선언되면서부터, 널(null)을 반환할 수 없게 됐다. 대신 포트가 없음을 나타내는 -1을 반환한다.  

여러 가지 기술적인 이유로 자바는 기관 부분에 있는 구문상의 오류를 초기에 알아내지 못할 수도 있다.  

이 상황이 발생할 경우, 즉각적인 증상으로 기관의 개별 부분(포트, 호스트, 사용자 정보)의 정보를 얻을 수 없게 된다. 이러한 경우, parseServerAuthority() 메소드를 호출하여 기관에 대한 분석을 다시 시도할 수 있다.  

+ public URI parseServerAuthority() throws URISyntaxException  

원래 URI 자체는 불변(immutable) 객체이기 때문에 변경할 수 없지만, 이 메소드가 반환하는 URI는 기관을 구성하는 사용자 정보, 호스트 그리고 포트의 분리된 정보를 가지게 된다. 메소드는 기관 정보를 분석할 수 없는 경우 URISyntaxException 예외를 발생시킨다.  

### 3. 상대 URI 변환하기
<br/>
URI 클래스는 상대적인 URI와 절대적인 URI를 서로 변환하기 위한 세 개의 메소드를 제공한다.  

+ public URI resolve(URI uri)
+ public URI resolve(String uri)
+ public URI relativize(URI uri)  

resolve() 메소드는 인자로 전달된 uri와 자신의 URI를 비교하고 완전한 경로를 만들기 위해 인자로 전달된 URI를 사용한다. 예를 들어, 아래 코드를 세 줄을 살펴보자.  

```java
URI absolute = new URI("httpL//www.example.com/");
URI relative = new URI("images/logo.png");
URI resolved = absolute.resolve(relative);
```

이 코드를 실행하고 나면 resolved는 다음 http://www.example.com/images/logo.png의 완전한 URI를 가지게 된다.  

위 코드에서 absolute가 절대적인 URI가 아닌 경우, resolve() 메소드는 새로운 상대적인 URI 객체를 만들어 반환한다. 예를 들어, 아래 코드를 살펴보자.  

```java
URI top = new URI("javafaq/books/");
URI resolved = top.resolve("jnp3/examples/07/index.html");
```

이 코드가 실행되면 다음 javafaq/books/jnp3/examples/07/index.html의 상대적인 URI가 생성된다.  

이 과정을 반대로 처리하는 것 또한 가능하다. 즉, 절대적인 URI로부터 상대적인 URI를 구해 낼 수 있다. relativize() 메소드는 전달된 인자로부터 메소드가 호출된 URI에 대해 상대적인 새로운 URI 객체를 생성한다.  

```java
URI absolute = new URI("http://www.example.com/images/logo.png");
URI top = new URI("http://www.example.com/");
URI relative = top.relativize(absolute);
```

위 코드의 실행 결과 상대적인 images/logo.png URI를 가진 relative 객체가 생성된다.  

### 4. URI 비교하기
<br/>
URI가 서로 같은지 확인하는 일은 꽤 빈법히 발생하며, 단순히 문자열 비교가 아닌 꽤 복잡한 처리가 된다. URI가 같기 위해서는 비교되는 URI 둘 모두 계층적이거나 불투명이어야 한다. 스킴과 기관 부분은 대소문자에 상관없이 비교된다. 즉, http와 HTTP는 동일한 스킴이며 www.example.com은 www.EXAMPLE.com과 동일한 기관이다. 그 외의 URI의 나머지 부분은 인코딩된 16진수를 제외하고 모두 대소문자를 구별한다. 두 URI를 비교하기 전에 인코딩된 값을 디코딩하지 않는다. 그래서 두 URI http://www.example.com/A와 http://www.example.com/%41은 같지 않다.  

hashCode() 메소드는 equals()와 같은 역할을 한다. 동일한 URI는 해시코드(hashcode) 값을 가지며 동일하지 않은 URI가 동일한 해시코드 값을 가질 가능성은 거의 없다.  

URI 클래스는 Comparable 인터페이스를 구현하기 때문에 URI 객체를 정렬하는 것이 가능하다. URI의 정렬은 다음 순서로 각 요소들의 문자열을 비교하여 결정된다.  

(1) 스킴이 서로 다른 경우 대소문자에 관계없이 스킴을 비교한다.  
(2) 스킴이 서로 같은 경우 계층적인 URI가 불투명 URI보다 더 낮은 것으로 간주된다.  
(3) 두 URI 모두 불투명인 경우 스킴에 따라 다른 부분으로 정렬을 결정한다.  
(4) 두 URI 모두 불투명이고 스킴에 따라 다른 부분까지 같은 경우 부위 지정자로 정렬을 결정한다.  
(5) 두 URI 모두 계층적인 경우 사용자 정보, 호스트, 포트순으로 기관에 따라 정렬된다. 이때 호스트는 대소문자를 가리지 않는다.  
(6) 스킴과 기관이 같은 경우 경로를 비교하여 결정한다.  
(7) 경로가 같은 경우 쿼리 문자열을 비교한다.  
(8) 쿼리 문자열이 같은 경우 부위 지정자를 비교한다.  

URI는 자신을 제외한 다른 타입과는 비교할 수 없다. URI가 아닌 다른 타입과 비교를 시도하면 ClassCastException 예외가 발생한다.  

### 5. URI를 문자열로 표현하기
<br/>
다음 toString()과 toASCIIString() 두 메소드는 URI 객체를 문자열로 변환한다.  

+ public String toString()
+ public String toASCIIString()  

toString() 메소드는 URI의 인코딩되지 않은 문자열 형태를 반환한다. 즉, 비아스키 문자가 인코딩되지 않은 채로 반환된다. 그래서 이 메소드의 호출 결과 생성된 문자열은 일부 아스키 문자만 사용하는 URI 범위를 벗어날 수 있다. 하지만 유니코드까지 표현할 수 있는 URI에는 포함된다. 이 형식은 데이터를 받는 용도가 아닌 사람이 읽기 쉬운 형태로 표현할 때 유용하게 사용할 수 있다.  

toASCIIString() 메소드는 URI의 인코딩된 문자열 형태를 반환한다. 비아스키 문자들은 항상 퍼센트 인코딩(percent-encoding)된다. 대부분의 상황에서 이 형태의 URI를 사용해야 한다. 그러나 비록 toString() 메소드가 사람이 더 읽기 쉬운 형태를 반환하긴 하지만, 여전히 정확한 URI 구문 규칙을 요구하지 않은 많은 곳에서 사용할 수 있다. toASCIIString() 메소드는 항상 구문 규칙에 맞는 URI를 반환한다.  

### 6. x-www-form-urlencoded
<br/>
웹 설계 당시 웹을 설계한 사람들은 운영체제 사이의 차이점을 처리해야 하는 과제에 직면했다. 이러한 운영체제 사이의 차이점은 URL을 사용하는 데 문제를 일으켰다. 예를 들어, 몇몇 운영체제는 파일 이름에 공백이 허용되지만 나머지 몇몇은 그렇지 않다. 그리고 대부분의 운영체제는 파일 이름에 # 기호가 허용되지만 URL에서 #은 파일 이름의 끝과 부위 지정자의 시작을 표시하는 데 사용된다. 또 다른 URL이나 다른 운영체제에서 특별한 의미를 가지는 다양한 문자들이 비슷한 문제를 가지고 있다. 게다가 웹이 발명되었을 당시에는 유니코드가 널리 사용되지 않았기 때문에 비아스키 문자를 처리할 수없는 시스템들이 많았다. 이러한 문제를 해결하기 위해, URL에서 사용된 문자들은 다음과 같이 정해진 일부 아스키 문자 집합으로 표현되어야 한다.  

+ 대문자 A-Z
+ 소문자 a-z
+ 숫자 0-9
+ 문장부호 문자 - _ . ! ~ * '(그리고 ,)  

/ & @ # ; $ + = 그리고 %와 같은 문자들 또한 사용되지만 다른 문자들과 달리 특별한 목적으로만 사용된다. 이러한 문자들이 경로나 쿼리 문자열의 일부로 사용될 경우 해당 문자나 문자열 전체를 인코딩해야 한다.  

인코딩은 매우 단순하다. 앞서 명시한 아스키 문자, 숫자 또는 문장부호 문자 이외의 모든 문자들은 먼저 바이트로 변환되고, 변환된 각 바이트는 % 기호와 두 개의 16진수로 표기된다. 스페이스의 경우 매우 자주 사용되기 때문에 조금은 특별하게 처리된다. %20으로 인코딩되는 것 외에도, 더하기 기호(+)로도 인코딩된다. 더하기 기호 자체는 %2B로 인코딩된다. / # = & 그리고 ? 문자는 URL의 요소를 구분하는 용도 외에 이름의 일부로 사용될 때는 동일하게 인코딩된다.  

URL 클래스는 자동으로 인코딩하거나 디코딩하지 않는다. 인코딩된 URL 객체를 만들 수 있고 금지된 아스키 문자와 비아스키 문자를 사용하는 URL 객체를 만들 수도 있다. 이러한 금지된 문자와 인코딩된 문자들은 getPath(), toExternalForm() 같은 메소드를 사용하여 출력될 때 자동으로 인코딩되거나 디코딩되지 않는다. URI 객체를 생성할 때 전달되는 문자열에서 인코딩이 필요한 문자들을 적절히 인코딩하는 것은 전적으로 URI 클래스를 사용하는 사용자에게 달려 있다.  

다행히 자바는 문자열을 인코딩하기 위한 URLEncoder와 URLDecoder 클래스를 제공한다.
