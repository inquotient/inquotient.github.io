---
title:  URLConnection 클래스
categories:
- Java Network Programming
feature_text: |
  ## URLConnection 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

URLConnection 클래스는 URL이 가리키는 리소스에 대해 활성화된 연결(connection)을 나타내는 추상 클래스다. URLConnection 클래스는 두 개의 서로 다르지만 연관된 목적을 가지고 있다. 첫째, 서버(특히 HTTP 서버)와 통신하는 데 URL 클래스보다 더 나은 제어 방법을 제공한다. URLConnection은 서버가 보낸 헤더를 확인하고 그에 따른 적절한 응답을 보낼 수 있다. 또한 클라이언트 요청에 사용된 헤더 필드를 설정할 수 있다. 마지막으로 URLConnection은 POST, PUT, 그리고 그 밖의 다른 HTTP 요청 메소드를 사용하여 웹 서버에 데이터를 보낼 수 있다.  

둘째, URLConnection 클래스는 URLStreamHandler 클래스를 포함하는 자바의 프로토콜 핸들러 메커니즘의 한 부분이다. 프로토콜 핸들러의 개념은 간단하다. 프로토콜을 처리하는 부분을 데이터를 처리하는 부분과 사용자 인터페이스를 제공하는 부분으로 따로 떼어 내 생각하는 것이다. 기본 java.net.URLConnection 클래스는 추상 클래스다. 따라서 특정 프로토콜을 지원하려면 서브클래스를 작성해야 한다. 이 서브클래스들은 애플리케이션에 의해 런타임(runtime) 시에 로딩될 수 있다. 예를 들어, 웹 브라우저가 컴프레스(compress)와 같은 낯선 프로토콜을 사용하는 사이트에 접속했을 때, 처리할 수 없다는 에러 메시지를 보여 주는 대신, 해당 프로토콜에 대한 핸들러를 다운로드받아 서버와 통신할 수도 있다.  

javv.net 패키지에는 추상 클래스인 URLConnection 클래스만 있으며, 나머지 구상 서브클래스들은 sun.net 패키지의 계층적인 구조에 숨겨져 있다. URLConnection 클래스에 있는 단일 생성자뿐만 아니라 많은 메소드와 필드가 protected로 선언되어 있다. 즉, URLConnection 클래스의 인스턴스나 서브클래스를 통해서만 접근이 가능하다. 여러분이 코드를 작성할 때 URLConnection 객체를 직접 인스턴스화하는 일은 거의 없다. 대신에 프로그램 실행 중에 사용 중인 프로토콜에 맞게 필요에 따라 해당 객체를 만들어 사용한다. 클래스(컴파일 시에는 종류를 알 수 없음)는 java.lang.Class 클래스의 forName()과 newInstance() 메소드를 사용하여 인스턴스화된다.  

자바 클래스 라이브러리에는 잘 설계된 URLConnection API가 없다. URLConnection 클래스의 문제점 중 하나는 HTTP 프로토콜과 너무 밀접하게 묶여 있다는 것이다. 예를 들어, 이 클래스는 각각의 파일이 전송되기 전에 MIME 헤더나 이와 비슷한 것이 먼저 전송될 것이라고 가정한다. 그러나 FTP와 SMTP 같은 대부분의 오래된 프로토콜들은 MIME 헤더를 사용하지 않는다.  

### 1. URLConnection 열기
<br/>
URLConnection 클래스를 사용하는 프로그램은 직접적으로 다음과 같은 기본적인 절차를 따른다.  

(1) URL 객체 생성.  
(2) 생성된 URL에 대한 URLConnection 객체를 얻기 위해 URL 객체의 openConnection() 메소드 호출.  
(3) 반환된 URLConnection 객체 설정.  
(4) 헤더 필드 읽기.  
(5) 입력 스트림을 구하고 데이터 읽기.  
(6) 출력 스트림을 구하고 데이터 읽기.  
(7) 연결 종료.  

항상 이 모든 절차를 수행해야 하는 것은 아니다. 예를 들어, URLConnection의 기본 설정으로 해당 URL을 처리할 수 있다면 3단계는 생략해도 된다. 그리고 서버가 제공하는 데이터만 필요하고 메타정보는 필요하지 않거나, 해당 프로토콜이 어떤 메타정보도 제공하지 않는 경우, 4단계 또한 생략해도 된다. 또한 서버에서 데이터를 받기만 하고 서버로 데이터를 보낼 필요가 없는 경우 6단계를 생략해도 된다. 프로토콜에 따라, 5단계와 6단계는 서로 순서를 바꾸거나 교차하여 사용될 수 있다.  

URLConnection 클래스의 단일 생성자는 protected로 선언되어 있다.  

+ protected URLConnection(URL url)  

따라서 새로운 종류의 URL(예를 들어, 프로토콜 핸들러를 작성하는 경우)을 처리하기 위해 URLConnection 클래스를 서브클래싱하는 것 이외에도, URL 클래스의 openConnection() 메소드를 호출하여 만들 수 있다. 예를 들어:  

```java
try {
  URL u = new URL("http://www.overcomingbias.com/");
  URLConnection uc = u.openConnection();
  // URL에서 읽기
} catch (MalformedURLException ex) {
  System.err.println(ex);
} catch (IOException ex) {
  System.err.println(ex);
}
```

URLConnection 클래스는 abstract로 선언되어 있다. 그러나 하나의 메소드를 제외한 모든 메소드는 이미 구현되어 있다. 이미 구현된 다른 메소드들도 사용자의 편의나 필요에 의해 오버라이드가 필요할 수도 있다. 꼭 구현해야 하는 유일한 메소드인 connect()  메소드는 서버에 대한 연결을 만들며, 서비스의 종류(HTTP, FTP 등)에 따라 다르게 구현된다. 예를 들어, sun.net.www.protocol.file.FileURLConnection의 connect() 메소드는 URL을 적절한 디렉터리에 있는 파일 이름으로 변환하며, 해당 파일에 대한 MIME 정보를 생성한다. 그리고 해당 파일에 대한 FileInputStream을 연다. 또 sun.net.www.protocol.http.HttpURLConnection의 connect() 메소드는 sun.net.www.http.HttpClient 객체를 생성하며, 이 객체는 서버에 연결하는 역할을 한다.  

+ public abstract void connect() throws IOException  

URLConnection이 처음 생성되면 연결되지 않은 상태다. 즉 로컬 호스트와 원격 호스트는 서로 데이터를 주고받을 수 없다. 두 호스트를 연결하는 소켓이 없다. connect() 메소드는 일반적으로 TCP 소켓을 사용하여 로컬 호스트와 원격 호스트가 서로 데이터를 주고받을 수 있도록 둘 사이에 연결을 만든다. 그러나 getInputStream(), getContent(), getHeaderField() 그리고 호스트 사이의 연결을 필요로 하는 다른 메소드들은 아직 연결되지 않은 경우에 메소드 내에서 직접 connect()를 호출한다. 따라서 connect() 메소드를 여러분이 직접 호출하는 일은 매우 드물다.  

### 2. 서버에서 데이터 읽기
<br/>
URLConnection 객체를 사용하여 URL로부터 데이터를 읽기 위한 최소한의 단계는 다음과 같다.  

(1) URL 객체 생성.  
(2) 생성된 URL에 대한 URLConnection 객체를 얻기 위해 URL 객체의 openConnection() 메소드 호출.  
(3) 반환된 URLConnection 객체의 getInputStream() 메소드 호출.  
(4) 일반적인 스트림 API를 사용하여 입력 스트림에서 읽기.  

getInputStream() 메소드는 서버가 보낸 데이터를 읽거나 분석할 수 있는 일반적인 InputStream 객체를 반환한다.  

URL 클래스와 URLConnection 클래스 사이는 다음 과 같은 큰 차이가 있다,  

+ URLConnection 클래스는 HTTP 헤더에 접근할 수 있다.
+ URLConnection 클래스는 서버로 보내는 요청 매개변수를 설정할 수 있다.
+ URLConnection 클래스는 서버로부터 데이터를 읽는 것뿐만 아니라 쓸 수도 있다.  

### 3. 헤더 읽기
<br/>
HTTP 서버는 요청에 대한 각 응답에 앞서 많은 정보를 헤더를 통해 제공한다. 예를 들어, 다음은 아파치 웹 서버가 반환하는 일반적인 HTTP 헤더의 내용이다.  

```
HTTP/1.1 301 Moved Permanently
Date: Sun, 21 Apr 2013 15:12:46 GMT
Server: Apache
Location: http://www.ibiblio.org/
Content-Length: 296
Connection: close
Content-Type: text/html; charset=iso-8859-1
```

여기에는 많은 양의 정보가 있다. 일반적으로 HTTP 헤더에는 요청된 문서의 콘텐츠 타입과 바이트 단위의 문서 크기, 콘텐츠를 인코딩하는 데 사용한 문자 집합, 현재 날짜와 시간, 콘텐츠의 만료일, 그리고 콘텐츠가 미자막으로 수정된 날짜에 대한 정보가 포함되어 있다. 그러나 이 정보는 서버에 따라 차이가 있다. 어떤 서버들은 각각의 요청에 대해 이 모든 정보를 보내는 반면에, 어떤 서버는 몇몇 정보만 보내거나 아무것도 보내지 않기도 한다.  

HTTP를 제외하고는 MIME 헤더를 사용하는 프로토콜은 매우 드물다. (기술적으로 말하면, HTTP 헤더도 실제로 MIME 헤더가 아니다. 단지 MIME 헤더와 많이 비슷해 보일 뿐이다.) 여러분이 직접 URLConnection의 서브클래스를 작성할 때, 아래에 나오는 메소드들로부터 의미 있는 값을 반환받기 위해서 종종 이 메소드들을 오버라이드할 필요가 있다. 이들 정보 중에서 가장 중요한 것은 콘텐츠 타입이다. URLConnection 클래스는 파일명이나 데이터의 앞부분 및 바이트를 통해 데이터의 콘텐츠 타입을 추측하는 몇몇 유틸리티 메소드를 제공한다.  

#### 3.1. 특정 헤더 필드 가져오기
<br/>
처음 6개의 메소드는 헤더로부터 아래에 나열된 특히 많이 사용되는 필드를 요청한다.  

+ Content-type
+ Content-length
+ Content-encoding
+ Date
+ Last-modified
+ Expires  

+ public String getContentType()  
getContentType() 메소드는 응답 본문의 MIME 미디어 타입을 반환한다. 이 메소드의 동작 여부는 웹 서버가 올바른 콘텐츠 타입을 보내는가에 달려 있다. 이 메소드는 예외를 발생시키지 않으며 콘텐츠 타입에 대한 정보가 없는 경우 널(null)을 반환한다. text/html은 여러분이 웹 서버에 접속할 때 마주치게 되는 가장 일반적인 콘텐츠 타입이다. 그 밖에 일반적으로많이 사용되는 콘텐츠 타입에는 text/plain, image/gif, application/xml, 그리고 image/jpeg가 있다.  

콘텐츠 타입이 몇몇 텍스트 형식인 경우, 이 헤더에 문서의 문자 인코딩을 나타내는 문자 집합(character set) 부분이 함께 포함될 수도 있다. 예를 들어:  

```
Content-Type: text/html; charset=UTF-8
```

또는  


```
Content-Type: text/html; charset=iso-2022-jp
```

이 경우에, getContentType() 메소드는 문자 인코딩 정보를 포함한 Content-type 필드의 전체 값을 반환한다.  

+ public int getContentLength()  
getContentLength() 메소드는 콘텐츠의 길이를 알려 준다. Content-length 헤더가 없는 경우 getContentLength() 메소드는 -1을 반환하며, 예외를 발생시키지 않는다. 읽어야 할 데이터의 크기를 정확히 알아야 할 때, 또는 데이터를 저장해 둘 충분한 크기의 버퍼를 미리 만들어야 할 때 사용된다.  

네트워크의 속도가 점점 더 빨리지고 파일이 점점 더 커짐에 따라, int의 최대값(약 2.1GB)을 초과하는 리소스를 어렵지 않게 찾을 수 있다. 이러한 경우에 getContentLength() 메소드는 -1을 반환한다. 자바 7에서는 getContentLength()와 동일하게 동작하며 반환 타입으로 int가 아닌 long을 반환하는 getContentLengthLong() 메소드를 제공하므로 충분히 큰 파일도 다룰 수 있다.  

+ public long getContentLengthLong() // 자바 7  

URL 클래스의 openStream() 메소드도 이론적으로는 같은 방법을 사용하여 GIF 이미지 또는 .class 바이트코드 파일 같은 바이너리 파일을 다운로드받을 수 있지만, 실제 이 방법에는 문제가 있다.  

HTTP 서버는 데이터 전송이 끝난 후 항상 명확하게 연결을 종료하는 것은 아니다. 그헣기 때문에 클라이언트는 읽기를 언제 멈춰야 하는지 알 수 없다. 바이너리 파일을 다운로드하기 위해서는 URLConnection의 getContentLength() 메소드를 사용하여 파일의 길이를 확인한 다음, 정확히 지정된 바이트를 읽는 것이 더 안정적이다.  

+ public String getContentEncoding()  
getContentEncoding() 메소드는 콘텐츠의 인코딩 방식을 알려 주는 String 타입을 반환한다. 인코딩되지 않은 채로 콘텐츠가 전송될 경우(HTTP 서버의 응답에서 일반적인 상황). 이 메소드는 널(null)을 반환한다. 이 메소드는 예외를 발생시키지 않는다. 웹에서 가장 일반적으로 사용되는 인코딩 방식은 x-gzip일 것이며, 이 방식으로 인코딩된 데이터는 java.util.zip.GZipInputStream으로 손쉽게 디코딩할 수 있다.  

콘텐츠 인코딩과 문자 인코딩은 서로 다르다. 문자 인코딩은 Content-type 헤더나 문서 내부 정보에 의해서 결정되며 바이트 단위로 인코딩되는 방법을 나타낸다. 반면에 콘텐츠 인코딩은 문서의 각 바이트들이 다른 바이트들과 어떤 방식으로 인코딩되어 있는가를 나타낸다.  

+ public long getDate()  
getDate() 메소드는 언제 문서가 전송되었는지를 나타내는 long 타입을 반환한다. 이 값은 그리니치 표준시(GMT, Greenwich Mean Time), 1970년 1월 1일 이후로 문서가 전송된 시간까지를 밀리초로 나타낸 값이다. 이 값을 java.util.Date 타입으로 변경할 수 있다. 예를 들어:  

```java
Date documentSent = new Date(uc.getDate());
```

이 시간은 서버의 입장에서 문서가 보내진 시간이다. 따라서 이 시간은 여러분의 로컬 장비의 시간과 맞지 않을 수 있다. HTTP 헤더에 Date 필드가 없는 경우 getDate()는 0을 반환한다.  

+ public long getExpiration()  
어떤 문서들은 서버 기준의 만료일(expires)을 가지고 있다. 이 만료일은 언제 해당 문서를 캐시에서 지우고 다시 서버에서 읽어야 하는지를 나타낸다. getExpiration() 메소드는 getDate()와 매우 유사하며, 반환값을 해석하는 방법에만 차이가 있다. 이 메소드는 GMT, 1970년 1월 1일 이후부터 문서가 파기될 날까지의 밀리초를 나타내는 long 타입을 반환한다. HTTP 헤더에 Expiration 필드가 없는 경우, getExpiration() 메소드는 해당 문서는 만료일이 없으며 캐시에 영원히 남아 있을 수 있음을 의미하는 0을 반환한다.  

+ public long getLastModified()  
날짜와 관련된 마지막 메소드인 getLastModified()는 문서가 마지막으로 변경된 날짜를 반환한다. 마찬가지로, 날짜는 1970년 1월 1일 이후의 밀리초를 나타낸다. HTTP 헤더에 Lastmodified 필드가 없는 경우(일반적으로 없는 경우가 많다), 이 메소드는 0을 반환한다.  

#### 3.2. 임의의 헤더 필드 가져오기
<br/>
앞에서 설명한 6개의 메소드는 헤더로부터 특정 필드의 값을 요청하지만, 이론적으로 하나의 메시지에 포함될 수 있는 헤더의 필드 수는 제한이 없다. 그렇기 때문에 임의의 헤더 필드로부터 값을 가져올 방법이 필요하다. 다음 다섯 개의 메소드는 헤더의 임의의 필드로부터 값을 가져온다. 사실, 앞에서 다룬 메소드는 여기서 다루는 메소드의 얇은 래퍼(wrapper) 메소드일 뿐이다. 자바를 설계한 사람들이 계획하지 않은 헤더 필드의 값을 가져와야 할 때 여기서 설명하는 메소드를 사용할 수 있다. 이 메소드는 요청된 헤더 필드가 있는 경우 해당 필드의 값을 반환하며, 그렇지 않은 경우 널(null)을 반환한다.  

+ public String getHeaderField(String name)  
getHeaderField() 메소드는 이름이 있는 헤더 필드의 값을 반환한다. 이때 헤더의 이름은 대소문자를 가리지 않으며 종료 콜론을 포함하지 않는다. 예를 들어 URLConnection 객체 uc의 해더 필드 Content-type과 Content-encoding의 값을 가져오기 위해 다음과 같이 할 수 있다.  

```java
String contentType = uc.getHeaderField("content-type");
String contentEncoding = uc.getHeaderField("content-encoding");
```

Date, Content-length, 또는 Expire 헤더를 가져오기 위해 같은 방법을 사용한다.  

```java
String data = uc.getHeaderField("date");
String expires = uc.getHeaderField("expires");
String contentLength = uc.getHeaderField("Content-length");
```

이 메소드는 앞의 절에서 언급한 getContentLength(), getExpirationDate(), getLastModified(), 그리고 getDate() 메소드처럼 int나 long이 아닌 항상 String을 반환한다. 숫자 값이 필요한 경우 String을 long이나 int로 변환해야 한다.  

getHeaderField()가 반환하는 값이 유효하다고 가정해서는 안 되며, 널(null)이 아닌지 항상 확인해야 한다.  

+ public String getHeaderFieldKey(int n)  
이 메소드는 n번째 헤더 필드(예: Content-length 또는 Server)의 키(헤더 필드의 이름)를 반환한다. 0번째 헤더 필드에는 널(null) 키를 가진 요청 메소드(request method)가 있기 때문에, 첫 번째 헤더는 1부터 시작한다. 예를 들어, URLConnection uc의 헤더에서 6번째 키를 가져오기 위해 다음과 같은 코드를 작성한다.  

```java
String header6 = uc.getHeaderFieldKey(6);
```

+ public String getHeaderField(int n)  
이 메소드는 n번째 헤더 필드의 값을 반환한다. HTTP에서 요청 메소드와 경로를 포함하고 있는 시작 라인이 헤더 필드 0번이며 첫 번째 실제 헤더는 1번에 위치한다.  

+ public long getHeaderFieldDate(String name, long default)  
이 메소드는 먼저 name 인자로 전달된 헤더 필드를 가져온 뒤 해당 필드의 값을 String에서 long으로 변환한다. 이때 long 값은 1970년 1월 1일 자정(GMT) 이후의 밀리초를 나타낸다. getHeaderFieldDate()는 날짜를 나타내는 헤더 필드를 가져올 때 사용될 수 있다. getHeaderFieldDate() 메소드는 문자열을 정수로 변경하기 위해 java.util.Date의 parseDate() 메소드를 사용한다. parseDate() 메소드는 대부분의 일반적인 날짜 형식을 잘 이해하고 변환하지만, 간혹 그렇지 못한 경우도 있다. - 예를 들어, 날짜가 아닌 다른 값을 가진 헤더 필드에 대해 호출할 경우, parseDate()가 날짜를 이해하지 못하거나 getHeaderFieldDate()가 요청된 헤더 필드를 찾을 수 없는 경우, getHeaderFieldDate()는 default 인자로 지정된 값을 반환한다. 예를 들어:  

```java
Date expires = new Date(uc.getHeaderFieldDate("expires", 0));
Date lastModified = new Date(uc.getHeaderFieldDate("last-modified", 0));
Date now = new Date(uc.getHeaderFieldDate("date", 0));
```

java.util.Date 클래스의 메소드를 사용하여 반환된 long 타입을 String 타입으로 변환할 수 있다.  

+ public int getHeaderFieldInt(String name, int default)  
이 메소드는 name 인자로 전달된 헤더 필드의 값을 가져와서 int로 변환된다. 인자로 전달된 헤더 필드를 찾을 수 없거나 정수로 변환할 수 없는 값을 포함하고 있는 경우 메소드 호출은 실패하며, getHeaderFieldInt()는 default 인자의 값을 반환한다. 이 메소드는 주로 Content-length 필드의 값을 가져오는 데 사용된다. 예를 들어, URLConnection uc에서 콘텐츠 길이를 가져오는 다음과 같은 코드를 작성할 수 있다.  

```java
int contentLength = uc.getHeaderFieldInt("content-length", -1);
```

이 코드에서 getHeaderFieldInt()는 Content-length 헤더 필드가 없는 경우 -1을 반환한다.  

### 4. 캐시
<br/>
웹 브라우저는 오랫동안 페이지나 이미지 파일에 대해서 캐시(cache)를 사용해 왔다. 사이트의 모든 페이지에서 로고 이미지를 반복해서 사용할 경우, 일반적으로 브라우저는 한 번만 원격 서버로부터 로고를 읽고 로그를 브라우저의 캐시에 저장한다. 그리고 해당 로고가 필요한 경우 원격 서버에 요청하지 않고 캐시로부터 읽는다. Expires와 Cache-control을 포함한 몇몇 HTTP 헤더를 사용하면 캐시를 제어할 수 있다.  

기본적으로 HTTP를 통해 GET으로 접근한 페이지는 캐시에 저장된다고 생각할 수 있지만, HTTPS나 POST를 통해 접근한 페이지는 일반적으로 캐시에 저장되지 않는다. 하지만 HTTP 헤더를 이용하여 이를 조정할 수 있다:  

+ Expires 헤더는 (주로 HTTP 1.0에서) 지정된 시간이 될 때까지 해당 데이터를 캐시에 저장해도 괜찮다는 의미다.  
+ Cache-control 헤더는 (HTTP 1.1) 자세한 캐시 정책을 제공한다.  
> max-age=[초]<br/>
캐시에 저장된 항목이 만료될 때까지 제공한다.<br/><br/>
s-maxage=[초]<br/>
공유 캐시에 저장된 항목이 만료될 때까지 남은 초 시간. 전용 캐시는 해당 항목을 더 오랫동안 저장할 수 있다.<br/><br/>
public<br/>
인증된 응답을 캐시에 저장할 수 있음을 나타내며, 그렇지 않은 경우 인증된 응답을 캐시에 저장할 수 없다.<br/><br/>
private<br/>
단일 사용자 캐시(전용 캐시)만 해당 응답을 저장할 수 있다. 공유 캐시는 저장하면 안 된다.<br/><br/>
no-cache<br/>
해당 항목은 여전히 캐시에 저장될 수는 있지만, 클라이언트는 접근할 때마다 ETag 또는 Last-modified 헤더를 사용하여 리소스의 상태를 재확인해야 한다.<br/><br/>
no-stroe<br/>
해당 항목을 저장하면 안 된다.<br/><br/>

Cache-control과 Expires 둘 모두 존재할 경우 Cache-control을 따른다. 서버는 충돌이 나지 않는 한 하나의 헤더에 다수의 Cache-control 헤더를 보낼 수 있다.  

+ Last-modified 헤더는 리소스가 마지막으로 변경된 날자를 나타낸다. 클라이언트는 HEAD 요청을 사용하여 이 값을 확인한 다음, 로컬에 저장된 복사본보다 더 최신인 경우에만 GET을 사용하여 전체 데이터를 가져온다.  

+ ETag 헤더(HTTP 1.1)는 해당 리소스가 변경되면 함께 변경되는 리소스에 대한 고유 식별자(unique identifier)다. 클라이언트는 HEAD 요청을 사용하여 이 값을 확인한 다음, 로컬에 저장된 복사본과 ETag 값이 다른 경우에만 GET을 사용하여 전체 데이터를 가져온다.  

#### 4.1. 자바를 위한 웹 캐시
<br/>
기본적으로 자바는 아무것도 캐시하지 않는다. URL 클래스가 사용할 시스템 기본 캐시를 설치하기 위해서는 아래 나열된 것들이 필요하다.  

+ ResponseCache의 구상(concrete) 서브클래스
+ CacheRequest의 구상 서브클래스
+ CacheResponse의 구상 서브클래스  

정적 메소드 ResponseCache.setDefault() 호출 시 여러분이 작성한 ResponseCache의 서브클래스를 전달하면, 여러분이 작성한 CacheRequest와 CacheResponse의 서브클래스와 함께 동작하는 ResponseCache의 서브클래스를 설치할 수 있다. 이 작업은 여러분이 작성한 캐시 객체를 시스템 기본값으로 설정한다. 자바 가상 머신은 시스템 단일 공유 캐시만 지원한다.  

캐시가 설치된 이후부터 시스템은 새로운 URL을 읽으려고 할 때마다, 먼저 캐시에 요청한다. 캐시가 요청한 콘텐츠를 반환할 경우, URLConnection은 원격 서버에 접속할 필요가 없다. 그리고 요청한 데이터가 캐시에 없는 경우, 프로토콜 핸들러는 해당 데이터를 원격 서버로부터 다운로드받는다. 다운로드가 완료되면 프로토콜 핸들러는 응답 내용을 캐시에 저장한다. 그 결과, 다음 요청 시 해당 콘텐츠를 더 빠르게 이용할 수 있다.  

ResponseCache 클래스 안에 있는 두 개의 추상 메소드는 시스템의 단일 캐시로부터 데이터를 가져오거나 저장한다.  

+ public abstract CacheResponse get(URI uri, String requestMethod, Map<String, List<String>> requestHeaders) throws IOException
+ public abstract CacheResponse put(URI uri, URLConnection connection) throws IOException  

put() 메소드는 OutputStream 객체를 감싼(wrapping) CacheRequest 클래스를 반환한다. OutputStream 객체는 프로토콜 핸들러가 읽은 데이터를 쓰는 데 사용된다. CacheRequest 클래스는 추상 클래스로 두 개의 메소드를 가지고 있다.  

+ public abstract OutputStream getBody() throws IOException
+ public abstract void abort()  

서브클래스에서 getBody() 메소드는 동시에 put() 메소드에 전달된 URI를 위한 캐시의 데이터 저장소를 가리키는 OutputStream을 반환해야 한다. 예를 들어, 데이터를 파일에 저장하려고 한다면, 해당 파일과 연결된 FileOutputStream을 반환해야 한다. 프로토콜 핸들러는 읽은 데이터를 이 OutputStream에 복사할 것이다. 만약 복사 중에 문제가 발생할 경우 (예를 들어, 서버가 갑자기 연결을 종료하는 경우), 프로토콜 핸들러는 abort() 메소드를 호출한다. 그러면 이 메소드는 해당 요청을 위해 캐시에 저장해 둔 모든 데이터를 제거해야 한다.  

ResponseCache 클래스가 제공하는 get() 메소드는 캐시로부터 데이터와 해더를 가져와서 CacheResponse 객체로 감싼 뒤 반환한다. 이 메소드는 요청된 URI가 캐시에 없는 경우 널(null)을 반환한다. 이 경우에 프로토콜 핸들러는 일반적인 경우처럼 해당 URI를 원격 서버로부터 가져온다. CacheRequest 클래스와 마찬가지로 CacheRequest 클래스도 추상 클래스이므로 서브클래스에서 메소드를 구현해야 한다. 이 클래스는 두 개의 메소드를 제공한다. 하나는 요청 데이터를 반환하고, 하나는 헤더를 반환한다. 서버 응답을 원본 그대로 저장할 때는, 이 둘을 모두 저장해야 한다. 헤더 정보는 HTTP 헤더 필드에 있는 이름을 키 값으로 하고, 해당 이름에 대응되는 목록을 값을 가지는 변경할 수 없는 맵(unmodifiable map) 형태로 반환되어야 한다.  

+ public abstract Map<String, List<String>> getHeaders() throws IOException
+ public abstract InputStream getBody() throws IOException  

자바는 한 번에 하니의 URL 캐시만 사용할 수 있다. 다음 ResponseCache.setDefault()와 ResponseCache.getDefault() 정적 메소드를 사용하여 기존 캐시를 변경하거나 새로운 캐시를 설치할 수 있다.  

+ public static Response getDefault()
+ public static void setDefault(ResponseCache responseCache)  

이 메소드는 동일한 자바 가상 머신에서 실행되는 모든 프로그램이 사용하는 단일 캐시를 설정한다.  

서버로부터 가져온 각각의 리소스는 만료되기 전까지 HashMap에 유지된다. 더 정교하게 구현된 클래스의 경우 우선순위가 낮은 스레드를 이용하여 만료된 문서를 스캔하고 제거하는 방법을 사용할 수 있다. 이 방법 이외에도 리소스를 큐에 저장하고 새로운 리소스를 저장하기 위한 공간이 필요할 때, 오래된 문서나 만료일에 가까운 문서를 제거하는 방법으로 구현할 수도 있다. 더욱더 정교한 구현에서는 심지어 저장된 각 문서들이 얼마나 자주 접근됐는지 추적해 보고 가장 오래된 문서, 가장 적게 사용된 문서를 지울 수도 있다.  

이미 언급한 바 있지만 자바 컬렉션 API 대신 파일 시스템을 사용하여 캐시를 구현할 수도 있다. 또한 데이터ㅂ에스에 캐시를 저장할 수도 있고, 일반적으로 잘 사용하지 않은 다른 많은 방법으로 구현할 수도 있다. 예를 들어, 특정 URL에 대한 요청을 지구 반대편에 있는 원격 서버가 아닌 로컬 서버로 리다이렉트(redirect)하여 본질적으로 로컬 웹 서버를 캐시처럼 사용할 수도 있다. 또는 ResponseCache 클래스가 시작될 때 일정 개수의 파일을 미리 내려 받아 캐시할 수도 있다. 이 파일들은 다른 파일들에 대한 캐시가 추가되면서 메모리에서 지워질 때까지 사용할 수 있다. 이 방식은 많은 다른 SOAP 요청을 처리하는 서버에서 유용하게 사용될 수 있다. 왜냐하면 이런 SOAP 요청들은 적은 수의 공통 스킴만을 사용하기 때문에 모두 캐시에 저장 가능하기 때문이다. 추상 ResponseCache는 이런 기능들과 다양한 사용 패턴을 지원할 만큼 충분히 유연하게 되어 있다.  

#### 4.2. 연결 설정하기
<br/>
URLConnection 클래스는 클라이언트가 서버에 요청을 생성하는 방법을 명확히 정의하는 7개의 protected 인스턴스 필드를 제공하며 다음과 같다.  

```java
protected URL url;
protected boolean doInput = true;
protected boolean doOutput = false;
protected boolean allowUserInteraction = defaultAllUserInteraction;
protected boolean useCaches = defaultUseCaches;
protected long ifModifiedSince = 0;
protected boolean connected = false;
```

예를 들어, doOutput이 true인 경우, 해당 URLConnection을 통해 데이터를 읽는 것뿐만 아니라 서버로 데이터를 쓸 수도 있다. useCaches가 false인 경우, 해당 연결을 통한 요청은 로컬 캐시를 거치지 않고 원격 서버로부터 새로 다운로드받는다.  

이 필드들은 모두 protected로 선언되어 있으므로, 이 필드들의 값은 메소드를 통해 접근하거나 변경할 수 있다.  

+ public URL getURL()
+ public void setDoInput(boolean doInput)
+ public boolean getDoInput()
+ public void setDoOutput(boolean doOutput)
+ public boolean getDoOutput()
+ public void setAllowUserInteraction(boolean allowUserInteraction)
+ public boolean getAllowUserInteraction()
+ public void setUseCaches(boolean useCaches)
+ public boolean getUserCaches()
+ public void setIfModifiedSince(long ifModifiedSince)
+ public boolean getIfModifiedSince()  

이 7개의 필드들은 URLConnection이 연결되기 전(좀 더 정확히 말하면 연결로부터 콘텐츠나 헤더를 읽기 전)에만 변경할 수 있다. 연결이 생성된 이후에 호출될 경우 필드에 값을 설정하는 대부분의 메소드는 IllegalStateException 예외를 발생시킨다. 대체로 연결이 열리기 전에만 URLConnection 객체의 속성을 설정할 수 있다.  

그 외에도 URLConnection의 모든 인스턴스의 기본 동작을 정의하는 몇몇 추가적인 메소드를 제공한다.  

+ public boolean getDefaultUseCaches()
+ public void setDefaultUseCaches(boolean defaultUseCaches)
+ public static void setDefaultAllowUserInteraction(boolean defaultAllUserInteraction)
+ public static boolean getDefaultAllowUserInteraction()
+ public static FileNameMap getFileNameMap()
+ public static void setFileNameMap(FileNameMap map)  

인스턴스 메소드와는 달리, 이 메소드들은 언제든지 호출할 수 있다. 그리고 새로운 기본 값은 이 메소드들이 호출된 이후에 생성된 URLConnection 객체에게만 적용된다.  

+ protected URL url  
url 필드는 URLConnection이 연결하고자 하는 URL을 나타낸다. 이 값은 URLConnection이 만들어질 때 생성자에 의해 설정되며 그 후에는 변경되어서는 안 된다. getURL 메소드를 호출하면 이 값을 구할 수 있다.  

+ protected URL url  
불리언(boolean) 필드 connected는 연결이 열려 있는 경우에는 true, 연결이 닫혀 있는 경우에는 false가 된다. URLConnection 객체는 생성 시 바로 연결이 열리지 않기 때문에 초기값은 false다. 이 변수는 java.net.URLConnection의 인스턴스와 서브클래스에서만 접근할 수 있다.  

connected 값을 직접 읽거나 변경하는 메소드는 제공되지 않는다. 그러나 URLConnection 클래스에서 실제 연결을 발생시키는 모든 메소드는 연결 시 이 변수를 true로 설정해야 한다. 이런 메소드에는 connect(), getInputStream(), getOutputStream()이 있다. 또한 URLConnection 클래스에서 연결을 닫는 모든 메소드는 이 변수를 false로 설정해야 한다. java.net.URLConnection에는 그런 메소드가 존재하지 않지만, java.net.HttpURLConnection 같은 몇몇 서브클래스들은 disconnect() 메소드를 제공한다.  

프로토콜 핸들러를 작성하기 위해 URLConnection의 서브클래스를 정리할 경우, 연결 후 connected를 true로 설정하고 연결 종료 후 false로 설정해야 한다. java.net.URLConnection 안에 있는 많은 메소드가 실행 시 이 변수를 참조한다. 만약 이 값이 잘못 설정될 경우 원인을 규명하기 어려운 심각한 버그들이 발생할 수 있다.  

+ protected boolean allowUserInteraction  
어떤 URLConnection은 직접적인 사용자의 입력을 필요로 한다. 예를 들어, 웹 브라우저 사용자 이름과 패스워드를 물어 보기도 한다. 그러나 많은 애플리케이션은 사용자가 컴퓨터 앞에 앉아서 직접 입력한다고 가정하지 않는다. 예를 들어, 컴색엔진 로봇은 어떤 사용자의 사용자 이름과 패스워드도 물어 보지 않고 백그라운드로 실행 중일 것이다. allowUserInteraction 필드는 이름에서 알 수 있듯이 사용자와의 상호작용이 허용되어 있는지는 나타낸다. 기본값은 false다.  

이 변수는 protected로 선언되어 있지만, public getAllowUserInteraction() 메소드를 사용하여 값을 읽을 수 있고 public setAllowUserInteraction() 메소드를 사용하여 값을 변경할 수 있다.  

> + public void setAllowUserInteraction(boolean allowUserInteraction)  
+ public boolean getAllowUserInteraction()
```

이 값이 true이면 사용자의 입력이 허용되며 false인 경우 사용자의 입력이 허용되지 않는다. 이 변수의 값은 언제든지 읽을 수 있지만 URLConnection이 연결되기 전에만 설정할 수 있다. URLConnection이 연결되어 있을 때 setAllowUserInteraction()을 호출하면 IllegalStateException 예외가 발생한다.  

예를 들어, 다음 코드는 웹 페이지에 대한 연결을 하나 만들고 필요한 경우 사용자에게 인증을 요청한다.  

```java
URL u = new URL("http://www.example/passwordProtectedPage.html");
URLConnection uc = u.openConnection();
uc.setAllowUserInteraction(true);
InputStream in = uc.getInputStream();
```

자바는 사용자 이름과 패스워드를 사용자에게 요청하기 위한 기본 그래픽 환경(GUI)을 제공하지 않는다. 만약 애플릿을 통해 요청을 보낼 경우, 브라우저에게 제공하는 기본 인증 대화상자를 사용할 수 있다. 독립적인 자바 애플리케이션인 경우 Authenticator를 먼저 설치해야 한다.  

이 대화상자에서 취소(cancel)를 선택하면 "401 Authorization Required" 에러와 서버가 인증되지 않은 사용자에게 보내는 메시지를 보게 된다. 그러나 확인을 누르고 재인증을 물어 볼 때 '아니오'라고 대답하여 인증을 거부하면 getInputStream()은 ProtocolException 예외를 발생시킨다.  

정적 메소드 getDefaultAllowUserInteraction()와 setDefaultAllowUserInteraction()는 allowUserInteraction 필드가 명확히 설정되지 않은 URLConnection 객체의 기본적인 동작을 결정한다. allowUserInteraction 필드는 정적 변수이기 때문에(즉, 인스턴스 변수가 아니라 클래스 변수) setDefaultAllowUserInteraction() 메소드 호출 후 생성된 모든 URLConnection 인스턴스의 기본 동작에 영향을 미친다.  

예를 들어, 다음은 getDefaultAllowUserInteraction() 메소드를 호출하여 기본적으로 사용자의 입력이 허용되는지 확인하는 코드다. 사용자의 입력이 기본적으로 허용되지 않는 경우, 사용자의 입력을 기본적으로 허용하기 위해 setDefaultAllowUserInteraction() 메소드를 사용한다.  

```java
if (!URLConnection.getDefaultAllowUserInteraction()) {
  URLConnection.setDefaultAllowUserInteraction(true);
}
```
+ protected boolean doInput  
URLConnection은 서버로부터 일거나, 서버에 쓰거나, 또는 둘 모두에 사용될 수 있다. protected 불리언(boolean) 필드인 doInput은 URLConnection이 읽기에 사용될 수 있는 경우 true이며, 읽기에 사용될 수 없는 경우 false이다. 이 필드의 기본 값은 true이다. protected로 선언된 이 변수에 접근하기 위해서는 public getDoInput() 메소드와 setDoInput() 메소드를 사용해야 한다.  

```java
public boolean getDoInput()
public void setDoOutput(boolean doOutput)
```

예를 들어:  

```java
try {
  URL u = new URL("http://www.oreilly.com");
  URLConnection uc = u.openConnection();
  if (!uc.getDoInput()) {
    uc.setDoInput(true);
  }
  // 연결로부터 읽기...
} catch (IOException ex) {
  System.err.println(ex);
}
```

+ protected boolean doOutput  
프로그램은 결과를 서버로 다시 보내기 위해 URLConnection을 사용할 수 있다. 예를 들어, POST 메소드를 사용하는 서버로 데이터를 보내야 하는 프로그램의 경우, URLConnection에서 출력 스트림을 구해서 보낼 수 있다. doOutput은 불리언(boolean) 타입의 protected로 선언되어 있으며, 출력으로 사용할 수 있는 경우 true이고 사용할 수 없으면 false다. 기본값은 false다. protected로 선언된 이 변수에 접근하기 위해서는 public getDoOutput() 메소드와 setDoOutput() 메소드를 사용해야 한다.  

> + public void setDoOutput(boolean doOutput)  
+ public boolean getDoOutput()

예를 들어:

```java
try {
  URL u = new URL("http://www.oreilly.com");
  URLConnection uc = u.openConnection();
  if (!uc.getDoOutput()) {
    uc.setDoOutput(true);
  }
  // 연결에 쓰기...
} catch (IOException ex) {
  System.err.println(ex);
}
```

http URL에 대해 doOutput 필드를 true로 설정하면, 요청 메소드가 GET에서 POST로 변경된다.  

+ protected long ifModifiedSince  
많은 클라이언트를, 특히 웹 브라우저와 프록시는 이전에 가져온 문서를 캐시에 저장한다. 사용자가 이전에 요청한 문서를 다시 요청하면, 웹 브라우저와 프록시는 캐시에서 저장된 문서를 가져온다. 그리나 해당 문서를 마지막으로 가져온 이후에 서버에 있는 원본이 변경되는 경우가 있을 수 있다. 이를 알 수 있는 유일한 방법은 서버에게 물어 보는 것이다. 클라이언트는 서버로 보내는 클라이언트 요청 HTTP 헤더에 특정일 이후에 문서가 변경되었는지를 묻는 If-Modified-Since에 명시된 시간 이후에 해당 문서가 변경된 경우 해당 문서를 보내고, 그렇지 않은 경우 문서를 보내지 않는다. 일반적으로 이 시간은 클라이언트가 문서를 마지막으로 가져온 시간이다. 예를 들어, 다음 클라이언트 요청은 2014년 10월 31일 오후 7시 22분 07초(GMT) 이후 문서가 변경된 경우에만 반환해 줄 것응 나타낸다.  

```
GET / HTTP/1.1
Host: login.ibiblio.org:56452
Accept: text/html, image/gif, image/jpeg, *; q= 2, */*; q=.2
Connection: close
If-Modified-Since: Fri, 31 Oct 2014 19:22:07 GMT
```

그 시간 이후 문서가 변경되었다면, 서버는 보통 때처럼 문서를 보내고, 그렇지 않은 경우 아래와 같은 "304 Not Modified" 메시지를 반환한다.  

```
HTTP/1.0 304 Not Modified
Server: WN/1.15.1
Date: Sun, 02 Nov 2014 16:26:16 GMT
Last-modified: Fri, 29 Oct 2004 23:40:06 GMT
```

그러면 클라이언트는 캐시로부터 해당 문서를 가져와서 보여 준다. 모든 웹 서버가 If-Modified-Since 헤더 필드에서 사용될 날짜[1970년 1월 1일 자정(GMT) 이후의 시간 밀리초]를 나타낸다. ifModifiedSince 필드는 protected로 선언되어 있기 때문에, 읽거나 수정하기 위해서는 getIfModifiedSince() 그리고 setIfModifiedSince() 메소드를 호출해야 한다.  

> + public boolean getIfModifiedSince()  
+ public void setIfModifiedSince(long ifModifiedSince)

+ protected boolean useCaches  
몇몇 클라이언트들, 특히 웹 브라이저는 서버에서 문서를 가져오지 않고 로컬 캐시에서 가져올 수 있다. 애플릿의 경우 브라우저의 캐시에 접근할 수도 있다. 독립적인 애플리케이션은 java.net.ResponseCache 클래스를 사용할 수 있다. useCaches 변수는 캐시의 사용이 가능한 경우, 캐시의 사용 유무를 결정한다. 이 변수의 기본값 true는 캐시가 사용될 것임을 의미한다. false는 캐시가 사용되지 않을 것임을 의미한다. useCaches는 protected로 선언되어 있기 때문에, 프로그램에서는 getUserCaches() 그리고 setUseCaches() 메소드를 사용하여 접근한다.  

> + public void setUseCaches(boolean)  
+ public boolean getUseCaches()
protected boolean connected

다음 코드는 다운로드받을 문서의 가장 최신 버전을 가져오기 위해 useCaches를 false로 설정하여 캐시 기능을 끈다.  

```java
try {
  URL u = new URL("http://www.sourcebot.com/");
  URLConnection uc = u.openConnection();
  uc.setUseCaches(false);
  // 문서 읽기...
} catch (IOException ex) {
  System.err.println(ex);
}
```

getDefaultUseCaches() 그리고 setDefaultUseCaches() 두 메소드는 useCaches 필드의 초기값을 정의한다.  

> + public void setDefaultUseCaches(boolean useCaches)  
+ public boolean getDefaultUseCaches()

비록 이 메소드들은 정적 메소드는 아니지만, URLConnection 클래스의 모든 인스턴스의 기본 캐시 사용 여부를 결정하는 정적 필드를 설정하거나 가져온다. 다음 코드는 기본적으로 캐시 기능을 끈다. 이 코드 실행 이후, 캐시 기능이 필요한 URLConnection은 명시적으로 setUseCaches(true)를 호출해 켜야 한다.  

```java
if (uc.getDefaultUseCaches()) {
  uc.setDefaultUseCaches(false);
}
```

#### 4.3. 타임아웃
<br/>
연결에 대한 타임아웃 값을 확인하거나 변경하는 네 가지 메소드가 있다. 즉, 타임아웃은 내부 소켓이 SocketTimeoutException 예외를 발생시키기 전에 원격 서버로부터 얼마나 오랫동안 응답을 대기할지를 나타낸다. 다음 네 가지 메소드가 있다.  

+ public void setConnectTimeout(int timeout)
+ public int getConnectTimeout()
+ public void setReadTimeout(int timeout)
+ public int getReadTimeout()  

setConnectTimeout() / getConnectTimeout() 메소드는 초기 연결 시 소켓이 기다리는 시간을 제어한다. setReadTimeout() / getReadTimeout() 메소드는 입력 스트림이 데이터의 도착을 기다리는 시간을 제어한다. 이 네 개의 메소드는 밀리초 단위로 타임아웃을 측정하며, 0은 타임아웃 제한이 없다는 의미로 해석한다. 타임아웃을 설정하는 두 메소드의 인자로 음수가 전달될 경우, 두 메소드는 IllegalStateException 예외를 발생시킨다.  

예를 들어, 다음 코드는 30초의 연결 타임아웃과 45초의 읽기 타임아웃을 설정한다.  

```java
URL u = new URL("http://www.example.org");
URLConnection uc = u.openConnection();
uc.setConnectTimeout(30000);
uc.setReadTimeout(45000);
```

### 5. 클라이언트 요청 HTTP 헤더 설정하기
<br/>
연결을 시도하기 전에 setRequestProperty() 메소드를 사용하여 HTTP 헤더에 새로운 헤더를 추가할 수 있다.  

+ public void setRequestProperty(String name, String value)  

setRequestProperty() 메소드는 해당 URLConnection 클래스의 헤더에 인자로 전달된 이름과 값의 필드를 추가한다. 이 메소드는 연결을 시도하기 전에만 사용해야 한다. 이 메소드는 이미 연결된 경우에 IllegalStateException 예외를 발생시킨다. getRequestProperty() 메소드는 해당 URLConnection 클래스가 사용하는 HTTP 헤더에서 인자로 지정된 필드의 값을 반환한다.  

HTTP는 단일 속성(property)에 다수의 값을 설정할 수 있다. 이 경우 각각의 값은 콤마로 구분된다.  

HTTP 프로토콜만이 이와 같은 헤더를 사용하기 때문에 이들 메소드는 연결된 URL이 HTTP URL일 때에만 의미가 있다. 이 헤더들은 NNTP와 같은 다른 프로토콜에서는 다른 의미를 가질 수도 있으며, 이 말은 곧 API 설계 자체가 얼마나 빈약한지를 보여 준다. 사실 이 메소드들은 일반적인 URLConnection 클래스의 일부가 아니라, 좀 더 구체적인 HttpURLConnection 클래스의 일부로 다뤄져야 한다.  

예를 들어, 웹 서버와 클라이언트는 쿠키를 사용하여 일시적인 정보를 저장한다. 쿠키는 단순히 이름-값 쌍의 집합이다. 서버는 응답 HTTP 헤더를 사용하여 쿠키를 클라이언트에게 보낸다. 이 시점부터 클라이언트는 그 서버에게 요청을 보낼 때마다 HTTP 요청 헤더에 다음과 비슷한 쿠키를 포함시킨다.  

Cookie: username-elharo; password=ACD0X9F23JJJn6G; session-100678945  

이 쿠키 필드는 세 개의 이름-값 쌍을 서버로 보낸다. 하나의 쿠키에 포함될 수 있는 이름-값 쌍의 개수에 특별한 제한은 없다. URLConnection 객체 uc가 있을 때 이 쿠키들은 다음 코드를 사용하여 연결에 추가할 수 있다.  

```java
uc.setRequestProperty("Cookie", "username-elharo; password=ACD0X9F23JJJn6G; session-100678945");
```

이 코드는 이미 같은 이름의 속성이 존재할 경우 기존 속성의 값을 변경한다. 기존 속성에 값을 추가하기 위해서는 대신 addRequestProperty() 메소드를 사용해야 한다.  

+ public void addRequestProperty(String name, String value)  

 사용할 수 있는 헤더의 목록은 따로 정해져 있지 않다. 일반적으로 서버들은 자신이 인식하지 못하는 헤더는 단순히 무시한다. HTTP 프로토콜은 헤더 필드의 이름과 값을 구성하는 데 일정한 제한을 두고 있다. 예를 들어, 이름은 공백 문자를 포함할 수 없고 값은 어떤 줄 바꿈 문자도 포함할 수 없다. 자바는 필드가 줄 바꿈 문자를 포함할 수 없도록 제한하고 있지만 잘 지켜지진 않는다. 필드가 줄 바꿈 문자를 포함할 경우 setRequestProperty() 그리고 addRequestProperty() 메소드는 IllegalStateException 예외를 발생시킨다. 이 외에는 특별한 제한이 없기 때문에 비정상적인 헤더를 서버로 보내기가 매우 쉬우며, 따라서 주의가 필요하다. 어떤 서버들은 비정상적인 헤더를 서버로 보내기가 매우 쉬우며, 따라서 주의가 필요하다. 어떤 서버들은 비정상적인 헤더도 문제없이 처리한다. 또 몇몇은 규칙에 맞지 않는 헤더는 무시하고 어쨋든 요청한 문서를 반환한다. 그러나 어떤 서버는 "400 Bad Request" 에러를 반환하기도 한다.  

 어떤 이유로 URLConnection에 있는 헤더를 확인해야 할 경우 다음 메소드를 사용한다.  

 + public String getRequestProperty(String name)  

 자바는 또한 연결에서 사용된 모든 요청 속성을 맵 형태로 반환하는 메소드를 제공한다.  

 + public Map<String, List<String>> getRequestProperties()  

 맵의 키는 헤더 필드의 이름이고, 키 값에 대응하는 값은 속성 값들의 목록이다. 이름과 값 모두 문자열 형태로 저장된다.  

### 6. 서버에 데이터 쓰기
<br/>
POST로 웹 서버에 폼을 제출하거나 PUT으로 파일을 업로드하는 것처럼 종종 URLConnection에 데이터를 써야 할 경우가 있다. getOutputStream() 메소드는 서버에 데이터를 전송하기 위해 쓸 수 있는 OutputStream을 반환한다.  

+ public OutputStream getOutputStream()  

URLConnection은 기본적으로 출력이 허용되지 않기 때문에, 출력 스트림을 요청하기 전에 setDoOutput(true)를 호출해야 한다. http URL에 대해 doOutput을 true로 설정하고 나면, 요청 메소드가 GET에서 POST로 변경된다. 그러나 GET 메소드는 검색 요청이나 페이지 이등과 같은 안전한 동작에만 사용해야 하며, 웹 페이지에 코멘트를 작성하거나 피자를 주문하는 것과 같은 리소스를 만들거나 수정하는 안전하지 않은 동작에는 GET 메소드를 사용해서는 안 된다.  

일단 OutputStream을 구하고 나면 BufferedOutputStream이나 BufferedWriter에 연결하여 버퍼 기능을 추가한다. 또한 원본 OutputStream보다 사용하기 편리한 DataOutputStream이나 OutputStreamWriter 같은 클래스에 연결할 수도 있다. 예를 들어:  

```java
try {
  URL u = new URL("http://www.somehost.com/cgi-bin/acgi");
  // 연결을 열고 POST 준비
  URLConnection uc = u.openConnection();
  uc.setDoOutput(true);

  OutputStream raw = uc.getOutputStream();
  OutputStream buffered = new BufferedOutputStream(raw);
  OutputStreamWriter out = new OutputStreamWriter(buffered, "8859_1");
  out.write("first=Julie&middle=&last=Harting&work=String+Quarter\r\n");
  out.flush();
  out.close()
} catch (IOException ex) {
  System.err.println(ex);
}
```

POST 메소드를 사용하여 데이터를 보내는 것은 GET 메소드를 사용하여 보내는 것만큼이나 쉽다. 먼저 setDoOutput(true)를 호출하여 URLConnection 클래스에서 출력할 수 있도록 설정하고, 쿼리 문자열을 URL에 추가하는 대신에 URLConnection 클래스의 getOutputStream() 메소드를 이용한다. 자바는 스트림이 닫히기 전까지 출력 스트림에 쓰인 모든 데이터를 버퍼에 저장한다. 이렇게 함으로써 Content-length 헤더의 값을 구할 수 있다. 클라이언트 요청과 서버 응답을 포함한 완전한 트랜잭션(transaction)은 다음과 같을 것이다.  

서버와 클라이언트 양쪽 모두의 설정을 변경할 수 있는 경우, 여러분이 원하는 어떤 다른 종류의 데이터 인코딩도 사용할 수 있다. 예를 들어, SOAP 그리고 XML-RPC 둘 모두 서버로 데이터를 전송할 때 x-www-form-url-encoded 쿼리 문자열이 아닌 XML로 전송한다.  

폼 데이터를 전송하는 데 다음 단계가 필요하다.  

(1) 서버 측 프로그램에게 보낼 이름-값 쌍을 결정한다.  
(2) 요청을 받아서 처리할 서버 측 프로그램을 작성한다. 특별한 사용자 정의 인코딩 타입을 사용하지 않는다면, 일반적인 브라우저와 HTML 폼을 사용하여 이 프로그램을 테스트할 수 있다.  
(3) 자바 프로그램에서 사용할 쿼리 문자열을 만든다.  
(4) 쿼리 문자열에서 사용할 이름과 값은 쿼리 문자열을 만든다. 쿼리 문자열에 추가하기 전에 URLEncoder.encode()를 통해 인코딩해야 한다.  
(5) 데이터를 받아들일 서버 측 프로그램의 URL에 대해 URLConnection을 연다.  
(6) setDoOutput(true)를 호출하여 doOutput 필드를 true로 설정한다.  
(7) URLConnection의 OutputStream에 쿼리 문자열을 쓴다.  
(8) URLConnection의 OutputStream을 닫는다.  
(9) URLConnection의 InputStream에서 서버의 응답을 읽는다.  

GET은 북마크나 링크될 수 있는 안전한 연산에만 사용되어야 하며, POST는 북마크나 링크될 수 없는 안전하지 않는 연산에 사용되어야 한다.  

getOutputStream() 메소드는 웹 서버에 파일을 저장하기 위한 PUT 요청 메소드를 위해서도 사용된다. 저장될 데이터는 getOutputStream()이 반환하는 OutputStream에 쓴다. 그러나 이것은 URLConnection의 서브클래스인 HttpURLConnection에서만 사용하는 것이기 때문에 PUT 명령에 대한 설명은 잠시 동안 미루어 두겠다.  

### 7. URLConnection의 보안 고려 사항
<br/>
URLConnection 객체는 네트워크 연결을 만들거나 파일을 읽고 쓸 때 일반적인 모든 보안 제약을 받는다. 예를 들어, 신뢰할 수 없는 애플릿은 자신을 보내 준 호스트에 대해서만 URLConnection 객체를 생성할 수 있다. 그러나 다양한 URL 스킴과 그에 대응하는 연결들은 각각 다른 보안 제약 사항이 있기 때문에 세부적으로 약간의 예외가 있을 수 있다. 예를 들어, 애플릿 자신의 jar 파일을 가리키는 jar URL은 허용되나, 로컬 하드 드라이브에 있는 파일에 대한 file URL은 허용되지 않는다.  

URL에 연결하려고 하기 전에, 연결이 허용되는지 알고 싶은 경우가 있다. 이러한 경우를 위해 URLConnection 클래스는 getPermission() 메소드를 제공한다.  

+ public Permission getPermission() throws IOException  

이 메소드는 URL에 연결하기 위해 어떤 허가(permission)가 필요한지 나타내는 java.security.Permission 객체를 반환한다. 어떤 허가도 필요 없는 경우 이 메소드는 널(null)을 반환한다(예를 들어, 보안 관리자가 없는 경우). URLConnection 클래스의 서브클래스는 java.security.Permission의 다양한 서브클래스를 반환한다. 예를 들어, 내장된 URL이 www.gwbush.com을 가리키는 경우, getPermission 메소드는 www.gwbush.org 호스트에 대해 연결과 주소 반환이 가능함을 의미하는 java.net.SocketPermission 클래스를 반환한다.  

### 9. MIME 미디어 타입 추측하기
<br/>
모든 프로토콜과 서버가 전송된 파일의 종류를 MIME 타입을 사용하여 알려 주는 것이 가장 이상적이다. 하지만 현실은 그렇지 않다. 우리는 MIME보다 오래된 FTP 같은 프로토콜도 처리해야 하며, 또한 MIME 헤더를 사용해야 함에도 불구하고 MIME 헤더를 제공하지 않거나 잘못된 헤더를 제공하는 HTTP 서버(보통 서버가 잘못 설정되어 있는 경우)도 처리해야 한다.  

URLConnection 클래스는 프로그램에서 사용 중인 데이터의 MIME 타입을 알아내는 데 도움이 되는 두 개의 정적 메소드를 제공한다. 콘텐츠 타입을 이용할 수 없거나 콘텐츠 타입이 틀렸다고 판단될 경우 이 메소드들을 사용할 수 있다. 이 두 메소드 중 첫 번째는 URLConnection.guessContentTypeFromName() 메소드다.  

+ public static String guessContentTypeFromName(String name)  

이 메소드는 객체의 URL에서 파일 이름의 확장자 부분을 기반으로 객체의 콘텐츠 타입을 추측한다. 이 메소드는 콘텐츠 타입을 추측하여 문자열(String)로 반환한다. 이 추측은 맞을 가능성이 높다. 사람들은 파일 이름을 생각할 때 일반적인 규칙을 따르기 때문이다.  

콘텐츠 타입을 추측하는 작업은 일반적으로 jre/lib 디렉터리에 위치한 content-types.properties 파일에 의해 결정된다. 유닉스에서 자바는 콘텐츠 타입의 추측을 돕기 위해 mailcap 파일 또한 참조한다.  

이 메소드가 결코 완전한 것은 아니다. 예를 들어, 이 메소드는 RDF(.rdf), XSL(.xsl) 같은 application/xml MIME 타입을 가져야 하는 XML 애플리케이션은 빠져 있다. 이 메소드는 또한 CSS 스타일시트(.css)파일에 대한 MIME 타입을 제공하지 않는다. 그러나 사용하기에 큰 무리는 없다.  

두 번째 MIME 타입 추측 메소드는 URLConnection.guessContentTypeFromName()이다.  

+ public static String guessContentTypeFromStream(InputStream in)  

이 메소드는 스트림에서 시작 몇 바이트를 살펴보고 콘텐츠 타입을 추측한다. 이 메소드가 동작하려면 InputStream에서 시작 몇 바이트를 읽고 다시 시작 위치로 돌아와야 하므로 해당 InputStream은 특정 위치로 이동(mark)을 지원해야 한다. 자바는 InputStream의 시작 16바이트를 확인하여 타입을 추측한다. 그러나 일반적으로 타입을 식별하는 데 16바이트까지 필요로 하지 않는다.  

이 추측 방법은 종종 guessContentTypeFromName()보다 신뢰하기 어려운 경우가 있다. 예를 들어 XML 파일이 XML 선언이 아닌 주석으로 시작할 경우 HTML 파일로 잘못 분류될 수 있다. 따라서 이 메소드는 최후의 수단으로 사용해야 한다.
