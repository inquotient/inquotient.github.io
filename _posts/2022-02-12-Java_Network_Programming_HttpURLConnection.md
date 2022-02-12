---
title:  HttpURLConnection 클래스
categories:
- Java Network Programming
feature_text: |
  ## HttpURLConnection 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

java.net.HttpURLConnection 클래스는 URLConnection 클래스이 추상 서브클래스다. 이 클래스는 http URL을 처리할 때 도움이 되는 몇몇 추가적인 메소드를 제공한다. 특히 이 클래스는 요청 메소드를 가져오거나 설정하는 메소드, 리다이렉트를 따를 것인지를 결정하는 메소드, 응답 코드와 메시지를 가져오는 메소드, 그리고 프록시 서버가 사용되었는지를 알아보는 메소드를 제공한다. 또한, 이 클래스에는 다양한 HTTP 응답 코드에 해당하는 상수가 정의되어 있다. 마지막으로, 이 클래스는 URLConnection 슈퍼클래스의 getPermission() 메소드를 오버라이드하며, 이 메소드의 의미를 변경하지는 않는다.  

이 클래스는 추상 클래스이고 이 클래스의 유일한 생성자는 protected로 선언되어 있기 때문에 HttpURLConnection의 인스턴스를 직접 만들 수 없다.  

그러나 http URL을 사용하는 URL 객체를 만들고 해당 클래스의 openConnection() 메소드를 호출하여 반환받은 URLConnection 객체는 HttpURLConnection의 인스턴스가 될 수 있다. 반환받은 URLConnection을 HttpURLConnection으로 다음과 같이 캐스팅할 수 있다.  

```java
URL u = new URL("http://lesswrong.com/");
URLConnection uc = u.openConnection();
HttpURLConnection http = (HttpURLConnection) uc;
```

또는 다음과 같이 한 단계를 줄일 수 있다.  

```java
URL u = new URL("http://lesswrong.com/");
HttpURLConnection http = (HttpURLConnection) u.openConnection();;
```

### 1. 요청 메소드
<br/>

기본적으로, HttpURLConnection은 GET 메소드를 사용하지만 setRequestMethod() 메소드를 사용하여 이를 변경할 수 있다.  

+ public void setRequestMethod(String method) throws ProtocolException  

method 인자는 아래 7개 중 하나를 사용해야 하며 대소문자를 구별한다.  

+ GET
+ POST
+ HEAD
+ PUT
+ DELETE
+ OPTIONS
+ TRACE  

위에 나열된 메소드 이외의 인자가 전달되면, IOException의 서브클래스인 java.net.ProtocolException 예외가 발생한다. 그러나 일반적으로 단순히 요청 메소드를 설정하는 것만으로는 충분하지 않다. 여러분이 하려는 일에 따라 HTTP 헤더를 수정하거나 메시지 본문 또한 제공해야 할 필요가 있다. 예를 들어, 폼 전송은 Content-length 헤더를 함께 제공해야 한다. 우리는 이미 GET 메소드와 POST 메소드에 대해서 충분히 살펴봤으므로 나머지 다섯 개 메소드에 대해서 알아보자.  

어떤 웹 서버는 표준이 아닌 몇몇 추가적인 요청 메소드를 지원한다. 예를 들어, WebDAV 기능을 위해 서버는 PROPFIND, MKCOL, COPY, MOVE, LOCK, UNLOCK, 메소드를 제공해야 한다. 그러나 자바는 이들 중 어떤 것도 지원하지 않는다.  

### 2. 서버와 연결 끊기
<br/>
HTTP 1.1은 단일 TCP 소켓을 사용하여 다수의 요청과 응답을 주고받을 수 있는 Keep-Alive(HTTP 연결 재사용)를 지원한다. 그러나 Keep-Alive를 사용하면 서버는 클라아인트에게 데이터의 마지막 바이트를 전송한 후에도 연결을 쉽게 닫을 수 없다. 클라이언트가 추가적인 요청을 보낼 수 있기 때문이다. 서버는 타임아웃을 설정하고 5초 동안 활동이 없으면 연결을 종료하는 방법을 사용할 수도 있지만, 결과적으로 클라이언트가 작업이 끝났을 때 스스로 연결을 종료해야 한다.  

HttpURLConnection 클래스는 HTTP Keep-Alive 기능을 명시적으로 끄지 않는 한 투명하게 지원한다. 즉, HttpURLConnection 클래스는 서버가 연결을 닫기 전에 동일한 서버에 다시 연결할 경우 기존에 연결된 소켓을 재사용한다. HttpURLConnection 클래스의 disconnect() 메소드는 특정 호스트와 대화가 끝난 시점에 클라이언트가 서버와의 연결을 끊을 수 있도록 한다.  

+ public abstract void disconnect()  

disconnect() 메소드는 호출 시 해당 연결을 아직 열고 있는 스트림이 있는 경우 해당 스트림을 종료시킨다. 그러나 반대로 스티림을 닫는다고 해서 연결을 닫거나 끊지는 않는다.  

### 3. 서버 응답 처리하기
<br/>
HTTP 서버 응답의 첫 번째 줄은 숫자 코드와 응답의 나타내는 메시지를 포함하고 있다. 예를 들어, 가장 일반적인 응답인 200 OK는 요청된 문서를 찾았다는 의미다.  

또 다른 친숙한 응답으로는 404 Not Found가 있다. 이 응답은 요청한 URL이 가리키는 문서가 더 이상 존재하지 안흔다는 의미다.  

이것 외에도 일반적이지는 않지만 많은 응답 코드가 존재한다. 예를 들어, 코드 301은 요청된 리소스가 새로운 위치로 영구적으로 이동했음을 의미하고, 이 응답을 받은 브라우저는 새로운 위치로 다시 요청하고(redirect) 이전 위치에 대한 북마크가 있는 경우 업데이트해야 한다. 예를 들어:  

종종 응답 메시지에서 숫자로 된 응답 코드만 필요한 경우가 있다. HttpURLConnection 클래스는 이 응답 코드를 int 타입으로 반환하는 getResponseCode() 메소드를 제공한다.  

+ public int getResponseCode() throws IOException  

응답 코드 다음에 오는 텍스트 문자열은 응답 메시지라고 불리며, 이 값은 getResponseMessage() 메소드를 호출 시 반환된다.  

+ public String getResponseMessage() throws IOException  

HTTP 1.0은 16가지의 응답 코드를 정의하며, HTTP 1.1은 이를 확장하여 40가지의 응답 코드를 정의한다. 이 응답 코드 중에서 404와 같은 코드는 이미 잘 알려져 있어 숫자만으로 충분히 의미를 전달할 수 있지만, 그 외의 나머지 응답 코드는 우리에게 친숙하지 않다. HttpURLConnection 클래스는 일반적인 응답 코드 36개를 HttpURLConnection.OK 그리고 HttpURLConnection.NOT&#95;FOUND와 같이 명명된 상수로 제공한다.  

#### 3.1. 에러 조건
<br/>
getErrorStream() 메소드를 호출하면 이 에러 페이지를 포함한 InputStream을 반환받을 수 있다. 그리고 getErrorStream() 메소드는 에러가 발생하지 않았거나 반환할 데이터가 없는 경우 널(null)을 반환한다.  

+ public InputStream getErrorStream()  

일반적으로 getErrorStream() 메소드는 getInputStream() 메소드 호출이 실패한 경우 catch 블록 안에서 사용한다.  

#### 3.2. 리다이렉트
<br/>
300번대 응답 코드는 모두 일종의 리다이렉트(방향 재지정)를 의미한다. 즉, 요청된 리소스가 처음 예상했던 위치에는 더 이상 존재하지 않으며 다른 위치로 이동했음을 나타낸다. 300번대 응답 코드를 받은 대부분의 브라우저는 해당 리소스의 새로운 위치로부터 자동으로 요청하여 읽는다. 그러나 이 방법은 사용자에게 아무런 통보 없이 사용자를 신뢰할 수 있는 사이트에서 신뢰할 수 없는 사이트로 이동시킬 가능성이 있기 때문에 보안상 문제가 될 수 있다. 기본적으로 HttpURLConnection은 리다이렉트를 따라간다. 그러나 HttpURLConnection 클래스는 리다이렉트를 따라갈지 여부를 결졍할 수 있는 두 개의 정적 메소드를 제공한다.  

+ public static boolean getFollowRedirects()
+ public static void setFollowedRedirects(boolean follow)  

getFollowRedirects 메소드는 리다이렉트가 허용된 경우 true를 반환하고, 그렇지 않은 경우 false를 반환한다. setFollowedRedirects() 메소드는 호출 시 true를 인자로 전달하면 HttpURLConnection 객체가 리다이렉트를 따라가게 만들며, false를 인자로 전달하면 리다이렉트를 따라가지 않도록 만든다. 이 두 메소드는 정적 메소드이기 때문에, 이 메소드가 호출된 이후에 생성된 모든 HttpURLConnection 객체의 동작에 영향을 준다. setFollowedRedirects() 메소드는 보안 관리자가 변경을 허가하지 않을 경우 SecurityException 예외를 발생시킨다. 특히 애플릿에서는 이 값의 변경이 허용되지 않는다.  

자바는 개별 인스턴스 단위로 리다이렉트를 설정하는 다음 두 개의 메소드를 제공한다.  

+ public boolean getInstanceFollowRedirects()
+ public void setInstanceFollowRedirects(boolean followRedirects)  

HttpURLConnection 클래스의 인스턴스는 setInstanceFollowRedirects()가 호출되지 않을 경우, 기본적으로 클래스 메소드인 HttpURLConnection.setFollowedRedirects()에 의해 설정된 동작을 따른다.  

#### 3.3. 프록시
<br/>
방화벽 안에 있는 사용자나 AOL 또는 많은 거대 ISP 업체의 사용자들은 프록시(proxy) 서버를 통해 웹에 접근한다. usingProxy() 메소드는 특정 HttpURLConnection이 프록시를 통해 웹에 접근하는지 여부를 알려 준다.  

+ public abstract boolean usingProxy()  

이 메소드는 프록시가 사용될 경우 true를 반환하고 그렇지 않으면 false를 반환한다. 어떤 경우에는 캐시의 목적이 아닌 보안상의 이유로 프록시를 사용하기도 한다.  

#### 3.4. 스트리밍 모드
<br/>
HTTP 서버로 전송되는 모든 요청에는 HTTP 헤더가 포함된다. Content-length는 이 헤더에 있는 필드 중 하나로 요청 본문의 바이트 수를 나타낸다. 헤더는 요청 본문보다 먼저 전송되기 때문에 헤더를 작성하기 위해서는 아직 준비되지 않은 요청 본문의 길이를 먼저 알아야 하는 문제가 생긴다. 일반적으로 자바는 이 상황을 HttpURLConnection에서 얻은 OutputStream에 쓴 모든 데이터를 스트림이 닫힐 때까지 보관(cache)하는 방법으로 해결한다. 스트림이 닫히는 시점에 자바는 요청 본문의 크기를 알 수 있으며, 이 정보로 Content-length 헤더를 작성한다.  

이 방법은 일반적인 웹 폼에 대한 응답으로 보내는 작은 요청에 대해서는 잘 동작한다. 그러나 매우 긴 폼에 대한 응답이나 몇몇 SOAP 메시지의 경우 이 방법을 사용하기에 부담이 된다. 이 방식은 HTTP PUT을 사용하여 큰 문서를 보내려고 할 때 매우 느리며 비효율적으로 동작한다. 자바가 전송할 마지막 데이터를 기다리지 않고 그전에 시작 바이트를 먼저 전송할 수 있다면 매우 효율적일 것이다. 자바는 이 문제에 대해 두 가지 해결책을 제공한다.  

전송할 데이터의 크기를 이미 알고 있는 경우, 예를 들어 HTTP PUT을 사용하여 로컬 파일을 업로드하는 경우, HttpURLConnection 객체에 데이터의 크기를 미리 알려 주는 방법이 있다. 그리고 미리 데이터의 크기를 알 수 없는 경우, 대신 '청크 분할 전송 인코딩(chunked transfer encoding)' 방법을 사용할 수 있다.  

청크 분할 전송 인코딩에서 요청의 본문은 다수의 조각으로 분할되어 보내지며, 각 자각은 자신의 콘텐츠 길이를 가진다. 청크 분할 전송 인코딩 방법을 사용하기 위해서는 URL을 연결하기 전에 원하는 청크의 크기를 인자로 setChunckedStreamingMode() 메소드를 호출하기만 하면 된다.  

+ public void setChunckedStreamingMode(int chunkLength)  

이 메소드가 호출된 후에 자바는 이 책에 있는 다른 예제들과는 약간 다른 형태의 HTTP를 사용한다. 하지만 자바 프로그래머는 이 차이에 대헛 신경 쓸 필요가 없다.  

저수준(raw) 소켓이 아닌 URLConnection을 사용하고 서버가 청크 분할 전송을 지원하는 한, 여러분이 작성한 기존 코드는 변경 없이 잘 동작할 것이다. 그러나 청크 분할 전송 인코딩은 리다이렉션과 인증이 필요한 사이트에서 문제가 된다. 만약 여러분이 청크로 분할된 파일을 리다이렉트된 URL이나 패스워드 인증을 요구하는 사이트에 전송할 경우, HttpRetryException 예외가 발생한다. HttpRetryException 예외 발생 시 새로운 URL이나 적절한 인증 정보를 가지고 기존 사이트에 다시 요청을 시도해야 한다. 그리고 필요한 모든 작업은 여러분이 일반적으로 사용하는 HTTP 프로토콜 핸들러가 지원하지 않으므로 수동으로 해야 한다. 따라서 꼭 필요한 경우가 아니라면 청크 분할 전송 인코딩을 사용하지 않도록 하자. 이 말은 곧 대부분의 성능 관련 조언과 마찬가지로, 기본 비스트리밍(non-streaming) 방식이 확실한 병목지점이라고 증명될 때까지는 이 최적화 방식을 사용하지 말아야 한다는 의미다.  

요청 데이터의 크기를 미리 알 수 있는 경우에는 이 정보를 HttpURLConnection 객체에 전달하여 연결을 최적화할 수 있다. 데이터의 크기를 미리 전달할 경우 자바는 해당 데이터를 네트워크로 즉시 스트리밍할 수 있다. 그렇지 않은 경우 콘텐츠의 길이를 결정하기 위해 여러분이 쓰는 모든 데이터를 저장(cache)해야 하며, 스트림을 닫은 후에나 네트워크로 전송할 수 있다. 여러분이 전송할 데이터가 얼마나 큰지 정확히 알고 있다면, 이 값을 인자로 setFixedLengthStreamingMode() 메소드를 호출하면 된다.  

+ public void setFixedLengthStreamingMode(int contentLength)
+ public void setFixedLengthStreamingMode(long contentLength)  

실제 전송할 데이터의 크기가 int의 최대 크기보다 큰 경우 자바 7 이후 버전에서는 long 타입을 대신 사용할 수 있다.  

자바는 이 숫자를 Content-length HTTP 헤더 필드에 사용한다. 그러나 이 메소드 호출 후 여기에 지정된 값보다 작거나 많은 데이터를 전송할 경우, 자바는 IOException 예외를 발생시킨다. 물론 이 예외는 먼저 이 메소드를 호출할 때가 아닌, 나중에 데이터를 쓸 때 발생한다. setFixedLengthStreamingMode() 메소드 자체는 인자로 음수가 전달될 경우 IllegalStateException 예외를 발생시키며, 연결이 맺어진 뒤에 호출되거나 이미 청크 분할 전송 인코딩이 설정된 경우 IllegalStateException 예외를 발생시킨다. [동일한 요청에 대해 청크 분할 전송 인코딩과 고정 길이 스트리밍 모드(fixed-length streaming mode)을 함께 사용할 수 없다.]  

고정 길이 스트리밍 모드는 서버 측과는 전혀 상광없다. 서버는 콘텐츠의 길이만 정확하면 어떻게 설정되었는지에 대해서는 신경 쓰지 않는다.  

그러나 청크 분할 전송 인코딩 방식과 마찬가지로 고정 길이 스트리밍 모드 또한 인증과 리다이렉션 시에는 문제가 발생한다. 요청된 URL에 대해 인증이나 리다이렉션이 요구될 경우 마찬가지로 HttpURLConnection 예외가 발생한다. 이 예외를 받으면 수동으로 재시도해야 한다. 그러므로 꼭 필요한 경우가 아니라면 이 모드 또한 사용하지 않도록 하자.
