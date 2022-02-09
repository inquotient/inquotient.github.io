---
title:  URL 클래스
categories:
- Java Network Programming
feature_text: |
  ## URL 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

java.net.URL 클래스는 http://www.locates.com/이나 ftp://ftp.redhat.com/pub/와 같은 URL(Uniform Resource Locator)을 추상화한 것이다. 이 클래스는 java.lang.Object를 확장한 것이며, 더 이상 서브클래싱할 수 없는 final 클래스다. URL 클래스는 다른 종류의 URL 인스턴스를 설정하기 위해 상속보다는 전략 디자인 패턴(Strategy Design Pattern)을 사용한다. 프로토콜 핸들러들이 전략들(Strategies)이고, URL 클래스는 선택된 여러 전략들을 통해서 스스로 문맥(context)을 형성한다.  

비록 UR:을 문자열 저장하는 일이 어려운 일은 아니지만, URL 객채는 스킴(프로토콜), 호스트네임, 포트, 경로, 쿼리 문자열 그리고 부위 지정자 필드를 포함하고 있고, 각각의 필드는 독립적으로 설정되어 있다고 생각하는 것이 이해하는 데 좀 더 도움이 된다. 사실, 자바의 각 버전에 따라 차이가 있긴 하지만 java.net.URL 클래스는 실제 이러한 방식으로 구성되어 있다.  

URL은 불변(immutable) 클래스이기 때문에 객체가 생성된 후에는 객체의 필드를 변경할 수 없다. 이러한 특성은 URL 클래스를 스레드 환경에서 안전한 클래스를 만드는 효과가 있다.  

### 1. URL 클래스 만들기
<br/>
InetAddress 객체와 달리, jaava.net.URL 클래스는 생성자를 사용하여 인스턴스를 생성할 수 있다. java.net.URL은 여러 종류의 생성자를 제공하며 생성자마다 요구하는 정보에 차이가 있다.  

+ public URL(String url) throws MalformedURLException
+ public URL(String protocol, String hostname, String file) throws MalformedURLException
+ public URL(String protocol, String host, int port, String file) throws MalformedURLException
+ public URL(URL base, String relative) throws MalformedURLException  

어떤 생성자를 사용할지는 여러분이 가지고 있는 정보와 그 정보의 형태에 따라 달라진다. 이 모든 생성자들을 지원되지 않는 프로토콜의 URL을 생성하려고 하거나 URL 형식에 문제가 있는 경우 공통적으로 MalformedURLException 예외를 발생시킨다.  

지원되는 프로토콜의 정확한 목록은 Java의 구현에 따라 차이가 있다. 모든 가상 머신에서 사용할 수 있는 프로토콜로는 http와 file이 있으며, file은 다루기 까다롭기로 악명 높다. 오늘날 자바는 https, jar 그리고 ftp 프로토콜 또한 지원한다. 몇몇 가상 머신의 경우 doc, netdoc, systemresource 그리고 verbatim 같은 자바에 의해 내부적으로 사용자 정의 프로토콜뿐만 아니라 mailto와 gopher도 지원한다.  

여러분이 필요한 포로토콜이 특정 가상 머신에서 지원되지 않는다면, 프로토콜 핸들러를 설치하여 URL 클래스가 해당 프로토콜을 지원하도록 할 수 있다. 하지만 실제로 이 방법은 들이는 노력에 비해서 유용하지 못하다. 차라리 해당 프로토콜을 지원하는 다른 라이브러리를 사용하는 편이 낫다.  

생성자는 URL 스킴을 확인하는 것 이외에 해당 URL의 구성이 올바른지 확인하지 않는다. 생성된 URL의 유효성은 전적으로 프로그래머에게 달려 있다. 예를 들어, 자바는 HTTP URL에 있는 호스트네임에 공백이 없는지 또는 쿼리 문자열이 x-www-form-URL-encoded 형식이 맞는지 검사하지 않는다. 그리고 또한 자바는 mailto URL이 실제로 메일 주소를 포함하고 있는지 검사하지 않는다. 존재하지 않는 호스트와 존재하지만 접근이 허가되지 않은 호스트에 대해서도 확인하지 않으므로 해당 URL도 생성할 수 있다.  

#### 1.1. 문자열을 이용하여 URL 생성하기
<br/>
URL 클래스의 제일 간단한 생성자의 경우 단일 인자(single argument)로 아래와 같이 완전한 형식의 URL 문자열을 전달받는다.  

+ public URL(String url) throws MalformedURLException  

다른 일반적인 생성자들처럼 new 연산자와 함께 사용하며, MalformedURLException을 발생시킨다. 아래 코드는 문자열을 이용하여 URL 객체를 생성하고 예외를 처리한다.  

```java
try {
  URL u = new URL("http://www.audubon.org/");
} catch (MalformedURLException ex) {
  System.out.println(ex);
}
```

RMI(Remote Method Ivocation)와 JDBC(Java Database Connectivity)가 지원되지 않는 것처럼 보이지만 조금 다르다. JDK는 이 두 가지 프로토콜을 지원한다. 그러나 이 두 프로토콜른 java.rmi와 java.sql의 다양한 부분을 통해서 각각 지원되며, 다른 프로토콜처럼 URL 클래스를 통해서 사용할 수 없다. (하지만 썬 마이크로시스템즈가 어떠한 이유로 의도적으로 다른 프로토콜과는 다른 복잡한 메커니즘을 선택한 것이 아니라면 필자도 왜 이런 방법을 선택했는지 잘 모르겠다.)  

다른 자바 7 가상 머신들도 비슷한 결과를 보여 주지만, 오라클 코드베이스가 아닌 가상머신들의 경우 가상 머신마다 지원하는 프로토콜이 다소 다른 경우도 있다. 예를 들어, 안드로이드의 달빅(Dalvik) 가상 머신은 기본적인 http, https, file, ftp 그리고 jar 프로토콜들만 지원한다.  

#### 1.2. URL의 구성요소를 이용하여 URL 생성하기
<br/>
URL을 구성하는 프로토콜, 호스트네임, 파일을 각각 지정하여 URL 객체를 생성할 수도 있다.  

+ public URL(String protocol, String hostname, String file) throws MalformedURLException  

이 생성자는 포트로 -1을 설정하기 때문에 해당 프로토콜의 기본 포트가 사용된다. 파일 인자는 슬래시로 시작해야 하고 결로와 파일 이름 그리고 선택적으로 부위 지정자를 포함한다. 초기 슬래시를 빠트리는 실수를 하는 경우가 많으며, 발견하기 어려운 실수 중 하나이다. 이 생성자도 모든 URL 생성자들처럼 MalformedURLException을 발생시킨다. 예를 들어:  

```java
try {
  URL u = new URL("http", "www.eff.org", "/blueribbon.html#intro");
} catch (MalformedURLException ex) {
  throw new RuntimeException("shouldn't happen. all VMs recognize http");
}
```

이 예제는 HTTP 프로토콜(포트 80)에 대한 기본 포트를 사용하는, http://www.eff.org/blueribbon.html#intro 주소를 가리키는 URL 객체를 생성한다. 파일 항목은 앵커(anchor) 태그의 이름을 함께 포함한다. 그리고 이 코드는 가상 머신이 HTTP 프로토콜을 지원하지 않는 경우 발생하는 예외를 처리한다. 그러나 이러한 예외는 실제 상황에서는 거의 발생하지 않는다.  

간혹 기본 포트를 사용하지 않는 경우에는 다음 생성자를 사용하면 포트를 정수로 명시할 수 있으며 나머지 인자는 동일하다. 예를 들어, 다음 코드는 명시적으로 포트 8000이 지정된, http://fourier.dur.ac.uk:8000/~dma3mjh/jsci/ 주소를 가리키는 URL 객체를 생성한다.  

```java
try {
  URL u = new URL("http", "fourier.dur.ac.uk", 8000, "/~dma3mjh/jsci/");
} catch (MalformedURLException ex) {
  throw new RuntimeException("shouldn't hapeen; all VMs recognize http");
}
```

#### 1.3. 상대적인 URL 생성하기
<br/>
다음 생성자는 기반이 되는 URL과 상대적인 URL로부터 완전한 URL 하나를 생성한다.  

+ public URL(URL base, String relative) throws MalformedURLException  

예를 들어, 여러분이 http://www.ibiblio.org/javafaq/index.html 문서를 분석하는 중에 전체 URL이 아닌 파일명만 포함한 mailinglists.html 페이지에 대한 링크를 만났다고 생각해보자. 이런 경우에, 부족한 정보를 채우기 위해 해당 링크를 포함하고 있는 페이지의 URL을 사용할 수 있다. 생성자는 http://www.ibiblio.org/javafaq/mailinglists.html 같은 새로운 URL를 생성한다. 예를 들어:  

```java
try {
  URL u1 = new URL("http://www.ibiblio.org/javafaq/index.html");
  URL u2 = new URL(u1, "mailinglists.html");
} catch (MalformedURLException ex) {
  System.err.println(ex);
}
```

u1의 경로에서 파일 이름이 제거되고 새 파일 이름인 mailinglists.html가 u2를 만들기 위해 추가된다. 이 생성자는 같은 디렉터리 안에 있는 전체 파일 목록에 대한 루프를 처리할 때 특히 유용하다. 첫 번재 파일에 대한 URL 객체를 생성한 다음, 처음 생성한 URL 객체의 파일 이름을 변경하여 나머지 파일에 대한 URL 객체를 만드는 데 사용할 수 있다.  

#### 1.4. URL 객체를 생성하는 다른 방법들
<br/>
여기서 언급된 생성자들 외에도, 자바 클래스 라이브러리에는 URL 객체를 반환하는 많은 메소드들이 존재한다. 자바 애플릿의 경우 getDocumentBase()는 애플릿을 포함하고 있는 페이지 URL을 반환하고, getCodeBase()는 애플릿 .class 파일에 대한 URL을 반환한다.  

java.io.File 클래스는 인자로 전달된 파일과 매칭되는 file URL을 반환하는 toURL() 메소드를 제공한다. 이 메소드에 의해 반환되는 URL의 정확한 형식은 플랫폼에 따라 차이가 있다. 예를 들어, 윈도우의 경우 file:/D:/JAVA/JNP4/05/ToURLTest.java과 같이 형식으로 반환한다. 그러나 리눅스와 유닉스 계열에서는 file:/home/elharo/books/JNP04/05/ToURLTest.java과 같은 형식으로 반환한다. 실제로도 file URL은 플랫폼과 프로그램에 매우 의존적이다. 자바 file URL은 종종 웹 브라우저나 다른 프로그램의 URL과 교환해서 사용할 수 없는 경우가 생기며, 심지어 다른 플랫폼에서 실행되는 같은 자바 프로그램이 서로 교환할 수 없는 경우도 있다.  

클래스 로더(Class loader)는 클래스 로딩뿐만 아니라 이미지나 오디오 파일 같은 리소스를 로딩하는 데도 사용된다. ClassLoader.getSystemResource(String name) 정적 메소드는 읽을 수 있는 단일 리소스로부터 URL을 반환한다. ClassLoader.getSystemResource(String name) 메소드는 읽을 수 있는 리소스의 URL 객체 목록을 포함하는 Enumeration 객체를 반환한다. 그리고 마지막으로, 인스턴스 메소드 getResource(String name)는 인자로 전달된 리소스를 찾기 위해 참조된 클래스 로더에 의해 사용된 경로를 탐색한다. 이러한 메소드에 의해 반환된 URL은 file URL, HTTP URL 아니면 다른 스킴일 수도 있다. 리소스의 전체 경로는 /com/macfaq/sounds/swale.au 또는 com/macfaq/images/headshot.jpg와 같은 마침표 대신 슬래시로 된 자바 패키지 이름과 같은 구성이다. 자바 가상 머신은 요청된 리소스를 클래스 경로(classpath)와 JAR 아카이브 안에서 찾으려고 할 것이다.  

클래스 라이브러리의 여기저기에는 URL 객체를 반환하는 몇몇 다른 메소드들이 있지만, 대부분 단순히 URL 객체를 반환하는 역할만 한다. 이전 장에서 이미 객체를 생성할 때 사용한 적이 있기 때문에 여러분은 이미 알고 있는 내용일 것이다. 예를 들어, javax.swing.JEditorPane 클래스의 getPage() 메소드와 java.net.URLConnection 클래스의 getURL() 메소드가 있다.  

### 2. URL에서 데이터 가져오기
<br/>
URL 자체로는 별로 흥미로울 것이 없다. 정말 흥미로운 것은 URL이 가리키는 문서 안에 포함된 데이터다. URL 클래스는 URL로부터 데이터를 가져오기 위한 몇몇 메소드를 제공한다.  

+ public InputStream openStream() throws IOException
+ public URLConnection openConnection() throws IOException
+ public URLConnection openConnection(Proxy proxy) throws IOException
+ public Object getContent() throws IOException
+ public Object getContent(Class[] classes) throws IOException  

이 메소드들 중에서 가장 기본이 되고 일반적으로 사용되는 메소드는 openStream()이며, 이 메소드는 데이터를 읽을 수 있는 InputStream을 반환한다. 단순한 다운로드 이상의 제어가 필요한 경우, 대신 openConnection을 호출할 수 있으며, 이 메소드는 설정 변경 후 해당 연결로부터 InputStream을 얻을 수 있는 URLConnection을 호출할 수 있으며, 이 메소드는 설정 변경 후 해당 연결로부터 InputStream을 얻을 수 있는 URLConnection을 반환한다. 마지막으로, getContent() 메소드를 사용하여 URL 객체의 콘텐츠를 요청할 수 있으며, 이 메소드는 String이나 Image와 같은 좀 더 완전한 객체를 반환하며, 한편으로 단순히 InputStream을 반환하기도 한다.  

+ public final InputStream openStream() throws IOException  
openStream() 메소드는 URL에 의해 참조된 리소스에 연결하고 서버와 클라이언트의 연결에 필요한 작업(handshaking)을 처리한 다음, 데이터를 읽을 수 있는 InputStream을 반환한다. 반환된 InputStream을 통해서 읽은 데이터는 URL이 참조하는 원본(볂환 또는 해석되지 않은) 그대로의 데이터다. 아스키 텍스트 파일을 읽고 있다면 아스키, HTML 파일을 읽고 있다면 원본 HTML, 이미지 파일을 읽고 있다면 바이너리 이미지 데이터 등등이다. 이 데이터에는 통신에 필요한 HTTP 헤더나 다른 프로토콜 관련된 어떠한 정보도 포함되지 않는다. 반환된 InputStream은 기존에 다른 InputStream을 사용해서 읽던 방식과 동일한 방식으로 읽는다. 예를 들어:  

```java
try {
  URL u = new URL("http://www.lolcats.com");
  InputStaream in = u.openStream();
  int c;
  while ((c = in.read()) != -1) System.out.write(c);
} catch (IOException ex) {
  System.err.println(ex);
}
```

이 예제는 IOException뿐만 아니라 URL 생성자에서 발생하는 MalformedURLException도 함께 처리할 수 있다. 이것은 MalformedURLException이 IOException의 서브클래스이기 때문이다.  

대부분의 네트워크 스트림처럼 스트림의 안정적인 종료(closing)에는 약간의 노력이 필요하다. 자바 6과 그 이전 버전에서는 해제 패턴(dispose pattern)을 일반적으로 사용했다. 스트림 변수를 try 블록 바깥에 선언하고 널(null)로 설정한다. 그리고 finally 블록 안에서 해당 변수가 널(null)이 아닐 때 닫는다. 예를 들어:  

```java
InputStream in = null;
try {
  URL u = new URL("http://www.locates.com");
  in = u.openStream();
  int c;
  while ((c = in.read()) != -1) System.out.write(c);
} catch (IOException ex) {
  System.err.println(ex);
} finally {
  try {
    if (in != null) {
      in.close();
    } catch (IOException ex) {
      // 무시한다
    }
  }
}
```

자바 7에서는 try-with-resources 구문을 사용함으로써 좀 더 명확해진다.  

```java
try {
  URL u = new URL("http://www.locates.com");
  try (InputStream in = u.openStream()) {
    int c;
    while ((c = in.read()) != -1) System.out.write(c);
  }
} catch (IOException ex) {
  System.err.println(ex);
}
```

+ public URLConnection openConnection() throws IOException  
openConnection() 메소드는 지정된 URL에 대한 소켓을 열고 URLConnection 객체를 반환한다. URLConnection은 네트워크 리소스에 대한 열린 연결을 의미한다. 이 메소드는 호출이 실패할 경우, IOException을 발생시킨다. 예를 들어:  

```java
try {
  URL u = new URL("https://news.ycombinator.com/");
  try {
    URLConnection uc = u.openConnection();
    InputStream in = uc.getInputStream();
    // 데이터를 읽는다...
  } catch (IOException ex) {
    System.err.println(ex);
  }
} catch (MalformedURLException ex) {
  System.err.println(ex);
}
```

서버와 직접 통신이 필요한 경우에만 이 메소드를 사용해야 한다. URLConnection는 서버로부터 보내진 모든 것에 대한 접근 권한을 여러분에게 준다. 문서 자체의 원본 데이터에 대한 접근뿐만 아니라[예를 들어, HTML, 평문(plain text), 바이너리 이미지 데이터], 프로토콜에 명시된 모든 메타데이터(metadata)에 접근할 수 있다. 예를 들어, HTTP 또는 HTTPS 스킴인 경우, URLConnection은 HTML 텍스트 이외에 HTTP 헤더에도 접근할 수 있다. 또한 URLConnection 클래스는 URL로부터 읽는 것뿐만 아니라 쓰는 것도 가능하다. 예를 들어 mailto URL로 메일을 보내거나 폼 데이터(form data)를 전송하는 경우가 있다.  

이 메소드의 오버로드된 변형은 연결이 통과할 프록시 서버를 지정하는 인자가 제공된다.  

+ public URLConnection openConnection(Proxy proxy) throws IOException

이 메소드에 전달된 프록시 설정은 socksProxyHost, socksProxyPort, http.proxyHost, http.proxyPort, http.nonProxyHosts와 같은 시스템 속성의 프록시 설정보다 우선한다. 프로토콜 핸들러가 프록시를 지원하지 않는 경우, 인자로 전달된 프록시는 무시되고 가능하다면 직접적으로 연결이 만들어진다.  

+ public final Object getContent() throws IOException  
getContent() 메소드는 URL이 참조하는 데이터를 다운로드받는 세 번째 방법이다. getContent() 메소드는 URL이 참조하는 데이터를 받고 해당 타입의 객체로 만들려고 시도한다. URL이 ASCII나 HTML 파일과 같은 텍스트 종류를 참조하는 경우, 일반적으로 InputStream의 한 종류를 반환한다. URL이 GIF 또는 JPEG 같은 이미지를 참조하는 경우, getContent() 메소드는 일반적으로 java.awt.ImageProducer 객체를 반환한다. 이러한 전혀 다른 두 클래스를 통합할 수 있는 것은 이것들이 실제 텍스트나 그림 파일 자체가 아니라 프로그램에서 그런 것을 생성할 수 있도록 해 주는 수단이기 때문이다.  

```java
URL u = new URL("http://mesola.obspm.fr/");
Object o = u.getContent();
// Object를 적절한 타입으로 캐스팅하며 처리...
```

getContent() 메소드는 서버로부터 받은 데이터의 헤더에서 Content-type 필드를 확인하는 방법으로 동작한다. 서버가 MIME 헤더를 사용하지 않거나 Content-type 같은 필드를 보내지 않을 경우, 데이터를 읽을 수 있는 InputStream의 한 종류를 반환한다. 객체가 반환되지 못할 경우 IOException이 발생한다.  

+ public final Object getContent(Class[] classes) throws IOException  
URL 클래스의 콘텐츠 핸들러는 리소스에 대해 반환 가능한 클래스의 목록을 제공하는 방법을 제공한다. getContent() 메소드의 오버로드된 변형은 콘텐츠가 어떤 클래스 타입으로 반환될지 선택할 수 있다. 이 메소드는 URL의 콘텐츠를 인자로 전달된 목록에서 최초로 가능한 형식으로 반환하려고 시도한다. 예를 들어, HTML 파일을 첫 번째로 String, 그리고 두 번째로 Reader, 세 번째로 InputStream 순서로 반환되기를 선호할 경우 다음과 같이 코드를 작성할 수 있다.  

```java
URL u = new URL("http://www.nwu.org");
Class<?>[] types = new Class[3];
types[0] = String.class;
types[1] = Reader.class;
types[2] = InputStream.class;
Object o = u,getContent(types);
```

콘텐츠 핸들러가 해당 리소스를 문자열 표현으로 반환하는 방법을 알고 있을 경우 해당 리소스를 String으로 반환하고, 문자열 표현으로 반환하는 방법을 모를 경우 Reader로 반환을 시도한다. 그리고 Reader로 표현하는 방법 역시 모를 경우 InputStream으로 반환한다. 반환된 객체는 타입 확인을 위해 instanceof를 사용하여 테스트해야 한다. 예를 들어:  

```java
if (o instanceof String) {
  System.out.println(o);
} else if (o instanceof Reader) {
  int c;
  Reader r = (Reader) o;
  while ((c = r.read()) != -1) System.out.print((char) c);
  r.close();
} else if (o instanceof InputStream) {
  int c;
  InputStream in = (InputStream) o;
  while ((c = in.read()) != -1) System.out.write(c);
} else {
  System.out.println("Error: unexpected type " + o.getClass());
}
```

### 3. URL 구성 요소 나누기
<br/>
URL은 다음 5가지 요소로 구성된다.  

+ 스킴(scheme) 또는 프로토콜
+ 기관(authority)
+ 경로(path)
+ 부위 지정자(fragment identifier) 또는 섹션(section) 또는 참조(ref)
+ 쿼리 문자열  

기관은 사용자 정보, 호스트 그리고 포트로 더 세분화될 수 있다.  

URL의 이러한 요소를 읽을 수 있는 9개의 public 메소드가 제공된다. getFile(), getHost(), getPort(), getProtocol(), getRef(), getQuery(), getPath(), getUserInfo(), getAuthority().  

+ public String getProtocol()  
getProtocol() 메소드는 URL의 스킴을 포함한 String을 반환한다.  

+ public String getHost()  
getHost() 메소드는 URL의 호스트네임을 포함한 String을 반환한다.  

+ public int getPort()  
getPort() 메소드는 URL에 명시된 포트 번호를 정수로 반환한다. URL에 포트 번호가 없는 경우 getPort() 메소드는 명시된 포트 번호가 없음을 의미하는 -1을 반환하고, URL은 해당 프로토콜의 기본 포트를 사용한다.  

+ public int getDefaultPort()  
getDefaultPort() 메소드는 URL에 명시된 포트가 없을 때 해당 URL의 프로토콜이 사용하는 기본 포트 번호를 반환한다. 기본 포트가 정의되지 않은 프로토콜의 경우 getDefaultPort()는 -1을 반환한다.  

+ public String getFile()  
getFile() 메소드는 URL의 경로 부분을 포함한 String을 반환한다. 자바는 URL의 경로와 파일 부분을 나누어 처리하지 않는다는 것을 기억하자. 호스트네임 다음의 첫 번재 슬래시(/)로부터 부위 지정자의 시작을 나타내는 # 기호 이전까지의 문자 전체가 파일 부분으로 간주된다.  

URL에 파일 부분이 없는 경우 자바는 파일을 빈 문자열로 설정한다.  

+ public String getPath()  
getPath() 메소드는 getFile() 메소드와 거의 유사하다. 즉, URL의 경로와 파일 부분을 포함한 String을 반환한다. 그러나 getFile() 메소드와 달리 반환하는 String에 쿼리 문자열이 포함되지 않으며 단지 경로만 포함된다.  

메소드 이름을 보고 예상한 것과는 달리 getPath() 메소드는 디렉터리 경로만을 반환하지 않으며, getFile() 메소드는 파일 이름만을 반환하지 않는다는 사실을 기억해야 한다. getPath()와 getFile() 둘 다 전체 경로와 파일 이름을 반환한다. 이 두 메소드의 유일한 차이점은 getFile()은 쿼리 문자열도 같이 반환하지만 getPath()는 그렇지 않다는 것이다.  

+ public String getRef()  
getRef() 메소드는 URL의 부위 지정자 부분을 반환한다. URL에 부위 지정자 부분이 없는 경우 이 메소드는 null을 반환한다. 아래 코드에서 getRef()는 xtocid902914 문자열을 반환한다.  

+ public String getQuery()  
getQuery() 메소드는 URL의 쿼리 문자열을 반환한다. URL에 쿼리 문자열이 없는 경우 getQuery() 메소드는 널(null)을 반환한다.  

+ public String getUserInfo()  
종종 URL에 사용자 이름과 패스워드 정보가 포함된 경우가 있다. 이 정보는 스킴과 호스트 사이에 위치한다. 사용자 이름과 구분하는 데 &#64; 심벌이 사용된다. 그러나 대부분의 경우에 URL에 패스워드를 포함하는 것은 보안에 매우 취약하다. URL에 아무런 사용자 정보가 없는 경우 getUserInfo() 메소드는 널(null)을 반환한다.  

Mailto URL에 대해 이 메소드를 사용하면 기대하는 것처럼 동작하지 않는다. Mailto URL의 경우 다음과 같은 mailto:elharo@ibiblio.org URL에서 "elharo@ibiblio.org"는 경로이며, 사용자 정보나 호스트가 아니다. 이 경우 URL은 메시지를 보내는 호스트와 사용자가 아닌 메시지의 원격 수신자를 명시하기 때문이다.  

+ public String getAuthority()  
URL의 스킴과 경로 사이에 있는 것이 기관(authority)이다. URI의 이 부분은 해당 리소스를 제공하는 기관을 가리킨다. 대부분의 경우 기관에는 사용자 정보, 호스트, 포트 번호가 포함된다. 그러나 모든 URL이 항상 모든 구성 요소를 포함하고 있는 것은 아니다. getAuthority() 메소드는 URL상에 표시된 기관을 반환하며 사용자 정보와 포트는 있을 수도 있고 없을 수도 있다.  

### 4. URL 비교하기
<br/>
URL 클래스는 일반적인 클래스에서 제공하는 equals와 hashCode() 메소드를 제공한다. 이러한 메소드는 여러분의 예상과 거의 비슷하게 동작한다. 두 개의 URL이 같은 호스트, 포트, 경로, 부위 지정자와 같은 리소스를 가리키고 있다면 이 둘은 같다고 생각할 수 있다. 그러나 여기에 한 가지 놀라운 사실이 있다. equals() 메소드는 실제로 해당 호스트에 대한 DNS 쿼리를 시도한다. 그래서 예를 들어, http://www.ibiblio.org/ URL과 http://ibiblio.org/ URL은 같다고 판단한다.  

이 이야기는 URL의 equals() 메소드가 잠재적인 블로킹 I/O 동작임을 의미한다! 이러한 이유로, java.util.HashMap과 같이 equals() 메소드를 사용하는 데이터 구조에 URL을 저장하는 것을 피해야 한다. 이러한 경우에는 java.net.URI를 사용하는 것이 좋으며, 필요에 따라 URI와 URL을 서로 변환해야 한다.  

반면에 equals() 메소드는 두 URL에 의해 식별된 리소스를 비교조차 하지 않는다. 예를 들어, http://www.oreilly.com/는 http://www.oreilly.com/index.html와 같지 않다. 그리고 http://www.oreilly.com:80는 http://www.oreilly.com/와 같지 않다.  

URL 클래스는 또한 두 URL이 같은 리소스를 참조하고 있는지 확인하기 위한 sameFile() 메소드를 제공한다.  

+ public boolean sameFile(URL other)  

비교 방식은 기본적으로 equals()와 같고 DNS 쿼리 역시 포함하지만, sameFile()은 부위 지정자를 고려하지 않는다. 다음 http://www.oreilly.com/index.html#p1 URL과 http://www.oreilly.com/index.html#p2 URL에 대해 equals() 메소드는 false를 반환하지만 sameFile() 메소드는 true를 반환한다.  

### 5. URL 변환하기
<br/>
URL 클래스는 인스턴스를 다른 형식으로 변환하는 세 가지 메소드를 제공한다. toString(), toExternalForm() 그리고 toURI().  

다른 클래스들과 마찬가지로 java.net.URL 클래스도 toString() 메소드를 제공한다. toString() 메소드는 항상 http://www.cafeaulait.org/javatutorial.html과 같은 완전한 URL 형태를 반환한다. toString()을 명시적으로 호출하는 경우는 흔하지 않으며, 출력 구문에서 toString() 메소드가 은엲중에서 호출된다. 출력 구문 이외에는 toString() 대신 toExternalForm()을 더 많이 사용한다.  

+ public String toExternalForm()  

toExternalForm() 메소드는 URL 객체를 HTML 링크 또는 웹 브라우저의 URL 열기 대화상자에서 사용할 수 있는 문자열로 변환한다.  

toExternalForm() 메소드는 URL을 사람이 읽을 수 있는 문자열 형태로 반환한다. 이 기능은 toString() 메소드와 동일하다. 사실 toString() 메소드가 하는 일은 toExternalForm() 메소드를 반환하는 것이 전부다.  

마지막으로, toURI() 메소드는 URL 객체를 동등한 URI 객체로 변환한다.  

+ public URI toURI() throws URISyntaxException  

절대화(absolutization)나 인코딩 같은 연산 작업에는 선택의 여지가 있는 경우 URL보다 URI 클래스를 사용하는 편이 낫다. 그리고 또한 해시 테이블(hash table)이나 다른 데이터 구조에 URL을 저장하는 경우에도 논블로킹(non-blocking) equals() 메소드를 제공하는 URI 클래스를 사용하는 편이 낫다. URL 클래스는 주로 서버로부터 콘텐츠를 다운로드받는 요도로 사용되어야 한다.
