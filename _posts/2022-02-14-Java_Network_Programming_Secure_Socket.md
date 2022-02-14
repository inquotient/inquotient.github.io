---
title:  보안 소켓
categories:
- Java Network Programming
feature_text: |
  ## 보안 소켓
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

JSEE를 사용할 경우 알고리즘 협상 방법, 키 교환 방법, 인증 방법, 그리고 데이터 암호화를 포함한 낮은 수준의 세부 사항에 대해서 신경 쓰지 않아도 된다. JSEE를 사용하여 보안 통신을 위한 통신 협상과 필요한 암호화를 투명하게 처리하는 소켓과 서버 소켓을 만들 수 있다. 여러분은 단지 소켓과 스트림을 통해 데이터를 보내기만 하면 된다. JSSE(Java Secure Socket Extension)는 네 개의 패키지로 나뉜다.  

+ javax.net.ssl  
보안 네트워크 통신을 위한 자바 API를 정의하는 추상 클래스다.  

+ javax.net  
보안 소켓을 생성하기 위해 생성자 대신 사용되는 추상 소켓 팩토리 클래스다.  

+ java.security.cert  
SSL에 필요한 공개키 인증서를 다루는 클래스다.  

+ com.sun.net.ssl  
JSEE에 대한 썬 마이크로시스템즈의 참조 구현 안에 있는 암호화 알고리즘과 프로토콜을 구현한 구상 클래스다. 기술적으로 이 부분은 JSEE 표준의 일부가 아니다. 다른 구현자들은 자신들의 패키지로 이 패키지를 대체할 수 있다. 예를 들어, CPU를 많이 사용하는 키 생성 및 암호화 프로세스의 속도를 향상시키기 위해 네이티브 코드를 사용하는 패키지들이 있을 수 있다.  

### 1. 보안 클라이언트 소켓 만들기
<br/>
기본적인 세부 사항에 대해서 크게 신경 쓰지 않는다면, SSL 소켓을 사용하여 이미 준비된 보안 서버와 대화하는 것은 매우 간단하다. java.net.Socket 객체를 생성하는 대신, javax.net.ssl.SSLSocketFactory로부터 createSocket() 메소드를 사용하여 소켓 객체를 얻을 수 있다. SSLSocketFactory는 추상 팩토리 디자인 패턴을 추상 클래스다. 정적 SSLSocketFactory.getDefault() 메소드를 호출하면 SSLSocketFactory의 인스턴스를 얻을 수 있다.  

```java
SocketFactory factory = SSLSocketFactory.getDefault();
Socket socket = factory.createSocket("login.ibiblio.org", 7000);
```

이 메소드는 SSLSocketFactory의 인스턴스를 반환하거나, 구상 서브클래스가 없는 경우 InstantiationException 예외를 발생시킨다. 팩토리에 대한 참조를 구한 다음에는 SSLSocket을 만들기 위한 다섯 개의 오버로드된 createSocket() 메소드 중 하나를 사용한다.  

+ public abstract Socket createSocket(String host, int port) throws IOException, UnknownHostException
+ public abstract Socket createSocket(InetAddress host, int port) throws IOException  
+ public abstract Socket createSocket(String host, int port, InetAddress interface, int localPort) throws IOException, UnknownHostException
+ public abstract Socket createSocket(InetAddress host, int port, InetAddress interface, int localPort) throws IOException, UnknownHostException
+ public abstract Socket createSocket(Socket proxy, String host, int port, boolean autoClose) throws IOException  

처음 두 개의 메소드는 인자로 전달된 호스트와 포트에 연결된 소켓을 만들고 반환하거나 연결할 수 없는 경우 IOException 예외를 발생시킨다. 세 번째와 네 번째 메소드는 지정된 로컬 네트워크 인터페이스와 포트로부터 지정된 호스트와 포트에 연결된 소켓을 반환한다. 그러나 마지막 createSocket() 메소드는 약간 다르다. 이 메소드의 첫 번째 인자로 프록시 서버에 연결된 소켓 객체를 전달한다. 이 메소드는 첫 번째 인자로 전달된 프록시 서버를 통해 지저아된 호스트와 포트로 연결된 소켓을 반환한다. autoClose 인자는 이 소켓이 닫힐 때, 내부 프록시 소켓에 대한 연결을 닫을지 여부를 결정한다. autoClose가 true인 경우 내부 프록시 소켓이 닫히며, false인 경우 닫히지 않는다.  

이 메소드들이 반환하는 소켓은 실제 javax.net.ssl.SSLSocket 클래스 객체이며, 이 클래스는 java.net.Socket 클래스의 서브클래스다. 그러나 여러분은 이러한 사실을 알 필요가 없다. 보안 소켓이 생성된 다음에는 getInputStream(), getOutputStream() 등의 메소드를 통해 다른 소켓들처럼 사용할 수 있다. 예를 들어, 주문을 받는 서버가 login.ibiblio.org의 7000번 포트에서 대기하고 있다고 가정해 보자. 각 주문은 단일 TCP 연결을 사용하여 아스키 문자로 보내진다. 서버는 주문을 받고 연결을 종료한다. (실제 서비스의 경우 주문을 보내면 서버로부터 적절한 응답 코드가 필요하지만 여기서는 그런 자세한 내용은 생략한다.) 클라이언트가 보낸 주문은 다음과 같다.  

클라이언트와 서버에 복잡하고 에러를 유발하는 코드에 대해 부담 없이 암호화를 하는 가장 간단한 방법은 바로 보안 소켓을 사용하는 것이다. 다음 코드는 보안 소켓을 통해 주문을 보낸다.  

```java
SSLSocketFactory factory = (SSLSocketFactory) SSLSocketFactory.getDefault();
Socket socket = factory.createSocket("login.ibiblio.org", 7000);

Writer out = new OutputStreamWriter(socket.getOutputStream(), "US-ASCII");
out.write("Name: John Smith\r\n");
out.write("Product-ID: 67X-89\r\n");
out.write("Address: 1280 Deniston Blvd, NY NY 10003\r\n");
out.write("Card number: 4000-1234-5678-9017\r\n");
out.write("Expires: 08/05\r\n");
out.flush();
```

### 2. 암호화 조합 선택하기
<br/>
JSSE 구현들마다 서로 다른 인증과 알고리즘의 암호화 조합(Cipher Suites)을 제공한다. 예를 들어, IAIK의 iSaSiLk는 256비트 AES 암호화를 제공하는 반면, 오라클의 자바 7에서 번들로 제공하는 구현의 경우 128비트 AES 암호화만 제공한다.  

JDK와 함께 제공되는 JSSE(Java Secure Socket Extension)에는 실제로 강력한 256비트 암호화가 포함되어 있지만, JCE Unlimited Strength Jurisdiction Policy Files을 설치하기 전에는 사용할 수 없다. 이 파일들이 필요한 법률적인 이유에 대해서는 따로 언급하지 않는다.  

SSLSocketFactory 클래스가 제공하는 getSupportedCipherSuites() 메소드는 해당 소켓에서 이용 가능한 알고리즘의 조합을 알려 준다.  

+ public abstract String[] getSupportedCipherSuites()  

그러나 이해 가능한 모든 암호화 조합이 해당 연결에 반드시 허용되는 것은 아니다. 일부는 너무 취약하여 사용할 수 없도록 꺼져 있는 경우도 있다. SSLSocketFactory 클래스가 제공하는 getEnabledCipherSuites() 메소드는 해당 소켓에서 실제 사용할 수 있는 암호화 조합을 알려 준다.  

+ public abstract String[] getEnabledCipherSuites()  

사용되는 실제 조합은 연결 시에 클라이언트와 서버 사이에 협상으로 결정된다. 클라이언트와 서버가 어떤 조합에도 동의하지 않는 상황도 발생할 수 있다. 해당 조합이 서버와 클라이언트 양쪽 모두에서 사용할 수 있지만 어느 한쪽 또는 양쪽 다 해당 조합을 사용하기 위해 필요한 키와 인증서가 없는 경우가 있을 수 있다. 어떤 경우든 createSocket() 메소드는 IOException의 서브클래스인 SSLException 예외를 발생시킨다. setEnabledCipherSuites() 메소드를 통해 클라이언트가 사용하고자 하는 조합을 변경할 수 있다.  

+ public abstract void setEnabledCipherSuites(String[] suites)  

이 메소드의 인자는 사용하고자 하는 조합의 목록이어야 한다. 각 조합의 이름은 getSupportedCipherSuites()에 의해 나열된 조합의 이름 중 하나여야 한다. 이 메소드가 제공하지 않는 조합의 이름을 사용할 경우, IllegalArgumentException 예외가 발생한다. 오라클의 JDK 1.7은 다음과 같은 암호화 조합을 지원한다.  

+ TLS&#95;ECDHE&#95;ECDSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256
+ TLS&#95;ECDHE&#95;RSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256
+ TLS&#95;RSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256
+ TLS&#95;ECDH&#95;ECDSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256
+ TLS&#95;ECDH&#95;RSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256
+ TLS&#95;DHE&#95;RSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256
+ TLS&#95;DHE&#95;DSS&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256
+ TLS&#95;ECDHE&#95;ECDSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;ECDHE&#95;RSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;RSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;ECDH&#95;ECDSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;ECDH&#95;RSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;DHE&#95;RSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;DHE&#95;DSS&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;ECDHE&#95;ECDSA&#95;WITH&#95;RC4&#95;128&#95;SHA
+ TLS&#95;ECDHE&#95;RSA&#95;WITH&#95;RC4&#95;128&#95;SHA
+ SSL&#95;RSA&#95;WITH&#95;RC4&#95;128&#95;SHA
+ TLS&#95;ECDH&#95;ECDSA&#95;WITH&#95;RC4&#95;128&#95;SHA
+ TLS&#95;ECDH&#95;RSA&#95;WITH&#95;RC4&#95;128&#95;SHA
+ TLS&#95;ECDHE&#95;ECDSA&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ TLS&#95;ECDHE&#95;RSA&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ SSL&#95;RSA&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ TLS&#95;ECDH&#95;ECDSA&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ TLS&#95;ECDH&#95;RSA&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ SSL&#95;DHE&#95;RSA&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ SSL&#95;DHE&#95;DSS&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ SSL&#95;RSA&#95;WITH&#95;RC4&#95;128&#95;MD5
+ TLS&#95;EMPTY&#95;RENEGOTIATION&#95;INFO&#95;SCSV
+ TLS&#95;DH&#95;anon&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256
+ TLS&#95;ECDH&#95;anon&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;DH&#95;anon&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;ECDH&#95;anon&#95;WITH&#95;RC4&#95;128&#95;SHA
+ SSL&#95;DH&#95;anon&#95;WITH&#95;RC4&#95;128&#95;MD5
+ TLS&#95;ECDH&#95;anon&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ SSL&#95;DH&#95;anon&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ TLS&#95;RSA&#95;WITH&#95;NULL&#95;SHA256
+ TLS&#95;ECDHE&#95;ECDSA&#95;WITH&#95;NULL&#95;SHA
+ TLS&#95;ECDHE&#95;RSA&#95;WITH&#95;NULL&#95;SHA
+ SSL&#95;RSA&#95;WITH&#95;NULL&#95;SHA
+ TLS&#95;ECDH&#95;ECDSA&#95;WITH&#95;NULL&#95;SHA
+ TLS&#95;ECDH&#95;RSA&#95;WITH&#95;NULL&#95;SHA
+ TLS&#95;ECDH&#95;anon&#95;WITH&#95;NULL&#95;SHA
+ SSL&#95;RSA&#95;WITH&#95;NULL&#95;MD5
+ SSL&#95;RSA&#95;WITH&#95;DES&#95;CBC&#95;SHA
+ SSL&#95;DHE&#95;RSA&#95;WITH&#95;DES&#95;CBC&#95;SHA
+ SSL&#95;DHE&#95;DSS&#95;WITH&#95;DES&#95;CBC&#95;SHA
+ SSL&#95;DH&#95;anon&#95;WITH&#95;DES&#95;CBC&#95;SHA
+ SSL&#95;RSA&#95;EXPORT&#95;WITH&#95;RC4&#95;40&#95;MD5
+ SSL&#95;DHE&#95;RSA&#95;EXPORT&#95;WITH&#95;DES40&#95;CBC&#95;SHA
+ SSL&#95;DHE&#95;DSS&#95;EXPORT&#95;WITH&#95;DES40&#95;CBC&#95;SHA
+ SSL&#95;DH&#95;anon&#95;EXPORT&#95;WITH&#95;DES40&#95;CBC&#95;SHA
+ TLS&#95;KRB5&#95;WITH&#95;RC4&#95;128&#95;SHA
+ TLS&#95;KRB5&#95;WITH&#95;RC4&#95;128&#95;MD5
+ TLS&#95;KRB5&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ TLS&#95;DRB5&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;MD5
+ TLS&#95;KRB5&#95;WITH&#95;DES&#95;CBC&#95;SHA
+ TLS&#95;KRB5&#95;WITH&#95;DES&#95;CBC&#95;MD5
+ TLS&#95;KRB5&#95;EXPORT&#95;WITH&#95;RC4&#95;40&#95;SHA
+ TLS&#95;KRB5&#95;EXPORT&#95;WITH&#95;RC4&#95;40&#95;MD5
+ TLS&#95;KRB5&#95;EXPORT&#95;WITH&#95;DES&#95;CBC&#95;40&#95;SHA
+ TLS&#95;KRB5&#95;EXPORT&#95;WITH&#95;DES&#95;CBC&#95;40&#95;MD5  

각각의 이름은 네 부분으로 분할된 알고리즘으로 구성되어 있다. 프로토콜, 키 교환 알고리즘, 암호화 알고리즘 그리고 체크섬. 예를 들어, 다음 SSL&#95;DH&#95;anon&#95;EXPORT&#95;WITH&#95;DES40&#95;CBC&#95;SHA의 이름은 SSL v3(Secure Sockets Layer Version); 키 협정을 위한 DiffieHellman 메소드; 인증 없음; 40비트 키의 DES 암호화; 암호화 블록 체이닝(CBC; Cipher Block Chaining), 그리고 보안 해시 알고리즘(SHA, Secure Hash Algorithm) 체크섬을 의미한다.  

기본적으로 JDK 1.7 구현은 모든 암호화된 인증 조합이 사용 가능하도록 설정되어 있다(이 목록에서 처음 28개 항목). 인증이 필요 없는 트랜잭션이 필요하거나 인증은 필요하나 암호화가 필요 없는 트랜잭션이 필요한 경우, 해당 조합을 사용할 수 있도록 setEnabledCipherSuites() 메소드를 명시적으로 호출하여 설정해야 한다. 그리고 NSA가 여러분의 메시지를 읽고자 한다면 모를까 일반적으로 조합의 이름에 NULL, ANON 또는 EXPORT와 같은 단어가 들어간 조합의 사용은 피하고 싶을 것이다.  

TLS&#95;ECDHE&#95;ECDSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256 조합 정도면 보통 알려진 대부분의 공격에 대해 안전하며, TLS&#95;ECDHE&#95;ECDSA&#95;WITH&#95;AES&#95;256&#95;CBC&#95;SHA256를 설정할 경우 좀 더 안전하다. 일반적으로 TLS&#95;ECDHE로 시작하고 SHA256 또는 SHA384로 끝나는 조합은 오늘날 널리 사용되는 가장 강력한 암호화 조합을 나타낸다. 그리고 나머지 조합들은 다양한 레벨의 공격으로부터 표적이 된다.  

DES/AES와 RC4 기반의 암호 사이에는 키 길이 이외에도 중요한 차이점이 있다. DES와 AES는 블록 암호 알고리즘이다. 즉, 여러 비트를 한 번에 암호화된다. DES는 항상 64비트 단위로 암호화된다. 64비트가 안 되는 경우 인코더는 여분의 비트를 덧붙인다(padding). AES는 128, 192 또는 256비트 단위의 블록을 암호화할 수 있지만, 데이터가 볼록 크기의 배수가 안 될 경우 마찬가지로 모자란 비트만큼 덧붙여서 인코딩한다. DES/AES는 보안 HTTP 그리고 FTP와 같은 파일 전송 애플리케이션에서는 문제가 되지 않는다. 그러나 채팅이나 텔넷과 같은 사용자 중심의 프로토콜에서는 문제가 될 수 있다. RC4는 한 번에 1바이트씩 암호화가 가능하고 한 번에 1바이트씩 전송이 필요한 프로토콜에서 사용하기 적합한 스트림 암호 알고리즘이다.  

예를 들어, 에드거가 64비트 정도의 암호는 금방 깰 수 있는 매우 강력한 병렬 컴퓨터를 갖고 있고, 거스와 안젤라가 이 사실을 알고 있다고 가정해 보자. 게다가 거스와 안젤라는 에드거가 그들의 회선에 침입하여 ISP나 전화회사에 협박성 메일을 보낼 수 있다는 의심을 하고 있다. 그래서 그들은 중간자 공격에 취약한 익명의 연결을 사용하지 않기로 한다. 그리고 안전을 위해 거스와 안젤라는 가장 강력한 암호화 조합인 TLS&#95;ECDHE&#95;ECDSA&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256 하나만 사용하기로 결정한다. 다음 코드는 연결에 대해 하나의 조합만을 사용 가능하도록 제한한다.  

```java
String[] strongSuites = {"TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256"};
socket.setEnabledCipherSuites(strongSuites);
```

연결의 반대측에서 이 암호화 프로토콜을 지원하지 않는 경우, 읽거나 쓰려고 할 때 취약한 채널을 통해 뜻하지 않게 기밀 데이터가 노출되는 상황을 방지하기 위해 소켓은 예외를 발생시킨다.  

### 3. 이벤트 핸들러
<br/>
네트워크 통신은 대부분의 컴퓨터의 속도와 비교하면 느리다. 인증된 네트워크 통신은 이보다 더 느리다. 필요한 키를 생성하고 보안 연결을 설정하는 데 수 초 이상이 걸린다. 따라서 해당 연결을 비동기로 처리하면 좀 더 효율적일 것이다. JSSE는 클라이언트와 서버 사이의 연결(handshaking)이 완료될 때 프로그램에게 알려 주기 위해 표준 자바 이벤트 모델을 사용한다. 이 패턴은 이미 괘 익숙한 방법이다. 핸드세이크 완료(handshake-complete) 이벤트 알림을 받기 위해서는 단지 HandshakeCompletedListener 인터페이스를 구현하기만 하면 된다.  

+ public interface HandshakeCompletedListener extends java.util.EventListener

이 인터페이스는 handshakeCompleted() 메소드를 선언한다.  

+ public void handshakeCompleted(HandshakeCompletedEvent event)  

이 메소드는 HandshakeCompletedEvent를 인자로 전달받는다.  

+ public class HandshakeCompletedEvent extends java.util.EventObject  

HandshakeCompletedEvent 클래스는 이벤트에 대한 정보를 얻기 위한 다음 네 개의 메소드를 제공한다.  

+ public SSLSession getSession()
+ public String getCipherSuite()
+ public X509Certificate[] getPeerCertificateChain() throws SSLPeerUnverifiedException
+ public SSLSocket getSocket()  

특히 HandshakeCompletedListener 객체는 특정 SSLSocket의 addHandshakeCompletedListener()와 removeHandshakeCompletedListener() 메소드를 사용하여 해당 SSLSocket에서 발생하는 핸드세이크 완료 이벤트를 받을 수 있다.  

+ public abstract void addHandshakeCompletedListener(HandshakeCompletedListener listener)
+ public abstract void removeHandshakeCompletedListener(HandshakeCompletedListener listener) throws IllegalArgumentException  

### 4. 세션 관리
<br/>
SSL은 웹 서버에서 가장 일반적으로 사용된다. 웹 연결은 일시적이며, 모든 페이지는 분리된 소켓을 통해 요청된다. 예를 들어, 아마존(amazon.com)에서 상품을 구매하려면 7개의 페이지를 거쳐야 하며, 주소를 변경하거나 선물 포장을 할 경우 페이지 수는 늘어난다. 이 모든 페이지에 대해 안전한 연결을 설정하는 데 10초 이상이 걸린다고 생각해 보자. 두 호스트 사이에 보안 연결을 하는 데는 많은 오버헤드가 발생하기 때문에, SSL은 보안 연결을 여러 소켓을 통해 확장하는 세션을 사용할 수 있다. 같은 세션에 있는 다른 소켓들은 같은 공개키와 개인키 조합을 사용한다. 아마존에 대한 보안 연결이 7개의 소켓을 사용할 경우, 이 7개의 소켓은 동일한 세션 내에서 같은 키를 사용하여 만들어진다. 이 세션에서 처음 연결을 시도한 소켓만이 키 생성과 교환의 오버헤드를 겪게 된다.  

JSSE를 사용하는 프로그래머라면 세션을 사용하기 위해 추가적으로 필요한 작업은 없다. 일정 시간 내에 단일 호스트의 동일한 포트에 대해 다수의 보안 소켓을 열 경우, JSSE는 자동으로 세션의 키를 재사용한다. 그러나 높은 보안이 요구되는 애플리케이션에서는 소켓 사이의 세션 공유를 끄거나 강제로 세션 재인증을 요구하기도 한다. JSSE에서 세션은 SSLSession 인터페이스의 인스턴스로 표현된다. 이 인터페이스가 제공하는 다양한 메소드를 사용하여 세션이 생성된 시간과 마지막 접속 시간 확인, 세션 무효화, 해당 세션에 관련된 정보를 구할 수 있다.  

+ public byte[] getId()
+ public SSLSessionContext getSessionContext()
+ public long getCreationTime()
+ public long getLastAccessedTime()
+ public void invalidate()
+ public void putValue(String name, Object value)
+ public Object getValue(String name)
+ public void removeValue(String name)
+ public String[] getValueNames()
+ public X509Certificate[] getPeerCertificateChain() throws SSLPeerUnverifiedException
+ public String getCipherSuite()
+ public String getPeerHost()  

SSLSocket의 getSession() 메소드는 이 소켓이 속해 있는 세션을 반환한다.  

+ public abstract SSLSession getSession()  

그러나 어떤 경우에 세션이 보안의 위험 요소가 되기도 한다. 각 트랜잭션마다 새로운 키를 할당하는 것이 더욱 안전하다. 만약 성능 좋은 하드웨어를 갖추고 있고 시스템을 안전하게 보호하고 싶은 경우, 세션의 사용을 중지시킬 수 있다. 소켓이 세션을 생성하지 못하도록 false를 인자로 setEnabledSessionCreation() 메소드를 호출한다.  

+ public abstract void setEnableSessionCreation(boolean allowSessions)  

getEnableSessionCreation() 메소드는 세션 내에 여러 소켓을 생성할 수 있는 경우 true를 반환하고, 그렇지 않은 경우 false를 반환한다.  

+ public abstract boolean getEnableSessionCreation()  

드물게, 재인증이 필요한 경우가 있다. 즉, 이전에 세션을 생성할 때 사용한 모든 인증서와 키를 버리고 다시 인증하고 싶은 경우이다. startHandshake() 메소드가 이러한 기능을 한다.  

+ public abstract void startHandshake() throws IOException  

### 5. 클라이언트 모드
<br/>
일반적으로 대부분의 보안 통신에서 서버가 적절한 인증서로 자신을 스스로 인증해야 한다. 그러나 클라이언트는 그럴 필요가 없다. 즉, 보안 서버를 통해 아마존에서 책을 구입할 때, 해커의 서버가 아니라 진짜 아마존 서버인지를 서버가 직접 필자가 사용 중인 브라이주에게 증명해야 한다. 그러나 필자는 아마존 서버에게 내가 진 앨리어트 해럴드(Elliote Rusty Harold)임은 증명하지 않아도 된다. 책음 구매하기 위해 클라이언트가 인증서를 구매하고 설치해야 한다면 이는 매우 번거로운 일이 될 것이다. 그러나 이와 같은 불균형적인 인증 방법으로 인해 신용카드 범죄가 발생하기도 한다. 이와 같은 문제를 피하기 위해 소켓 스스로 인증을 요구할 수 있다. 그러나 이러한 방식은 일반적으로 공개된 서비스에는 적용되지 않는다. 그러나 내부적으로 높은 보안이 요구되는 특정 애플리케이션에 한해서 사용될 수 있다.  

setUseClientMode() 메소드는 해당 소켓이 첫 핸드세이크 시에 인증이 필요한지 여부를 결정한다. 이 메소드의 이름은 약간 오해의 소지가 있다. 이 메소드는 클라이언트와 서버 소켓 모두에서 사용될 수 있다. 그러나 true가 인자로 전달되면, 해당 소켓이 클라이언트 모드(소켓이 실제 클라이언트 측인지 서버 측인지에 상관없이)임을 의미하며 스스로 인증을 제공하지 않는다. false가 인자로 전달되면, 스스로 인증을 시도한다.  

+ public abstract void setUseClientMode(boolean mode) throws IllegalArgumentException  

이 속성은 소켓에 함 번만 설정될 수 있으며, 두 번째 시도할 경우 IllegalArgumentException 예외가 발생한다.  

getUseClientMode() 메소드는 해당 소켓이 첫 핸드세이크에서 인증을 사용할 것인지를 알려 준다.  

+ public abstract boolean getUseClientMode()  

서버 측 보안 소켓(즉, SSLServerSocket의 accept() 메소드가 반환하는 소켓)은 서버로 연결하는 모든 클라이언트에게 스스로 인증을 요청하기 위해 setNeedClientAuth() 메소드를 사용한다.  

+ public abstract void setNeedClientAuth(boolean needsAuthentication) throws IllegalArgumentException  

이 메소드는 해당 소켓이 서버 측 소켓이 아닌 경우 IllegalArgumentException 예외를 발생시킨다.  

getNeedClientAuth() 메소드는 클라이언트 측에 인증을 요구할 경우 true를 반환하고, 그렇지 않은 경우 false를 반환한다.  

+ public abstract boolean getNeedClientAuth()  

### 6. 보안 서버 소켓 만들기
<br/>
보안 클라이언트 소켓은 보안 통신에 필요한 절반일 뿐이며, 나머지 절반은 SSL을 사용할 수 있는 서버 소켓이다. 이 소켓은 javax.net.SSLServerSocket 클래스의 인스턴스이다.  

+ public abstract class SSLServerSocket extends ServerSocket  

SSLSocket과 마찬가지로 이 클래스의 모든 생성자는 protected로 선언되어 있으며 인스턴스는 추상 팩토리 클래스인 javax.net.SSLServerSocketFactory에 의해 생성된다.  

+ public abstract class SSLServerSocketFactory extends ServerSocketFactory  

또한 SSLSocketFactory처럼 SSLServerSocketFactory의 인스턴스는 정적 메소드인 SSLServerSocketFactory.getDefault()에 의해 반환된다.  

+ public static ServerSocketFactory getDefault()  

그리고 SSLSocketFactory처럼 SSLServerSocketFactory는 세 개의 오버로드된 createServerSocket() 메소드를 제공한다. 이 메소드는 SSLServerSocket의 인스턴스를 반환하므로 java.net.ServerSocket 생성자와 비교하면 이해하기가 쉬울 것이다.  

+ public abstract ServerSocket createServerSocket(int port) throws IOException
+ public abstract ServerSocket createServerSocket(int port, int queueLength) throws IOException
+ public abstract ServerSocket createServerSocket(int port, int queueLength, InetAddress interface) throws IOException  

보안 소켓을 생성하고 사용하는 일은 어렵지 않지만, 불행하게도 이것이 전부는 아니다. SSLServerSocketFactory.getDefault()가 반환하는 팩토리는 일반적으로 서버 인증만 지원하며 암호화는 지원하지 않는다. 암호화 지원을 추가하기 위해서는 서버 쪽 보안 소켓에 많은 초기화 설정이 필요하다. 정확한 설정 방법은 구현에 따라 차이가 있다. 썬 마이크로시스템즈에서 만든 참조 구현에서는 com.sun.net.ssl.SSLContext 객체가 보안 서버 소켓을 만드는 역할을 전담하고 있다. 자세한 내용은 JSSE 구현마다 차이가 있다면, 참조 구현에서 보안 서버 소켓을 만들 경우 다음과 같이 한다.  

(1) keytool을 사용하여 공개키와 인증서를 생성한다.  
(2) Comono와 같은 신뢰할 수 있는 서드파티 기관에 돈을 지불하고 인증서를 인증 받는다.  
(3) 사용할 알고리즘에 대한 SSLContext을 생성한다.  
(4) 사용할 인증서 데이터 소스에 대한 TrustManagerFactory를 생성한다.  
(5) 사용할 키 데이터 타입에 대한 KeyManagerFactory를 생성한다.  
(6) 키와 인증서 데이터베이스를 위한 KeyStore를 생성한다.  
(7) 키와 인증서로 KeyStore 객체를 채운다. 예를 들어, 암호화 시에 사용된 암호(passphrase)를 사용하여 파일 시스템에서 읽는다.  
(8) KeyManagerFactory를 KeyStore와 KeyStore의 암호(passphrase)로 초기화한다.  
(9) KeyManagerFactory에서 키 매니저, TrustManagerFactory에서 신뢰 매니저, 그리고 난수 소스를 초기화한다. [기본값을 사용하고자 할 경우 마지막 두 개는 널(null)일 수도 있다.]  

다음 예제는 이 절차를 확인하는 SecureOrderTaker 프로그램이며, 주문을 받고 System.out으로 주문 내용을 출력한다. 물론 실제 상황에서는 좀 더 흥미로운 주문을 해 볼 수 있을 것이다.  

```java
import java.io.*;
import java.net.*;
import java.security.*;
import java.security.cert.CertificateException;
import java.util.Arrays;

import javax.net.ssl.*;

public class SecureOrderTaker {

  public final static int PORT = 7000;
  public final static String algorithm = "SSL";

  public static void main(String[] args) {

    try {
      SSLContext context = SSLContext.getInstance(algorithm);

      // 참조 구현에서는 X.509 키만 지원한다.
      KeyManagerFactory kmf = KeyManagerFactory.getInstance("SunX509");

      // 오라클의 기본 키스토어 유형
      KeyStore ks = KeyStore.getInstance("JKS");

      // 보안 문제로, 모든 키스토어는 디스크에서 읽어 오기 전에
      // 패스(passphrase)로 암호화되어야 한다.
      // 패스는 char[] 배열에 저장되어 있으며 가비지 컬렉터를
      // 기다리지 않고 재빨리 메모리에서 지워야 한다.
      char[] password = System.console().readPassword();
      ks.load(new FileInputStream("jnp4e.keys"), password);
      kmf.init(ks, password);
      context.init(kmf.getKeyManagers(), null, null);

      // 패스워드 제거
      Arrays.fill(password, '0');

      SSLServerSocketFactory factory = context.getServerSocketFactory();

      SSLServerSocket server = (SSLServerSocket) factory.createServerSocket(PORT);

      // 익명(인증되지 않음) 암호화 조합 추가
      String[] supported = server.getSupportedCipherSuites();
      String[] anonCipherSuitesSupported = new String[supported.length];
      int numAnonCipherSuitesSupported = 0;
      for (int i = 0; i < supported.length; i++) {
        if (supported[i].indexOf("_anon_") > 0) {
          anonCipherSuitesSupported[numAnonCipherSuitesSupported++] = supported[i];
        }
      }

      String[] oldEnabled = server.getEnabledCipherSuites();
      String[] newEnabled = new String[oldEnabled.length + numAnonCipherSuitesSupported];
      System.arraycopy(oldEnabled, 0, newEnabled, 0, oldEnabled.length);
      System.arraycopy(anonCipherSuitesSupported, 0, newEnabled, oldEnabled.length, numAnonCipherSuitesSupported);

      server.setEnabledCipherSuites(newEnabled);

      // 모든 설정이 끝나고 이제 실제 통신에 초점을 맞춘다.
      while (true) {
        // 이 소켓은 보안이 적용된 소켓이지만 소켓 사용 방법에 별 차이가 없다.
        try (Socket theConnection = server.accept()) {
          InputStream in = theConnection.getInputStream();
          int c;
          while ((c = in.read()) != -1) {
            System.out.write(c);
          }
        } catch (IOException ex) {
          ex.printStackTrace();
        }
      }
    } catch (IOException | KeyManagementException | KeyStoreException | NoSuchAlgorithmException | CertificateException | UnrecoverableKeyException ex) {
      ex.printStackTrace();
    }
  }
}
```

이 예제는 현재 작업 디렉터리에 있는 jnp4e.keys라는 파일로부터 필요한 키와 인증서를 읽는다. 이 파일은 비밀번호 "2andnotafnord"로 보호되어 있다. 이 예제에서는 이 파일을 생성하는 방법은 보여 주지 않는다. 이 파일은 JDK와 함께 번들로 제공되는 keytool 프로그램을 사용하여 만들며 다음과 같이 실행한다.  

이 과정이 끝나면 jnp4e.keys 파일이 생성되며 이 파일에 여러분의 공개키가 포함되어 있다. 그러나 이렇게 생성된 키를 GeoTrust나 GoDaddy와 같은 신뢰할 수 있는 업체로부터 인증받지 않는다면 아무도 믿지 않을 것이다. 인증된 인증서를 구매하기 전에 JSSE를 둘러보고 싶다면, 오라클은 testkyes라는 이름의 인증된 키스토어 파일을 제공한다. 이 파일은 패스워드 "passphrase"로 보호되어 있으머 몇몇 JSSE 샘플을 함께 제공한다. 그러나 이러한 테스트 키를 실제 제품에 적용하지 않도록 해야 한다.  

다른 접근 방법은 인증을 요구하지 않은 암호화 조합을 이용하는 것이다. JDK는 아래와 같은 인증을 요구하지 않은 몇몇 조합들을 제공한다.  

+ SSL&#95;DH&#95;anon&#95;EXPORT&#95;WITH&#95;DES40&#95;CBC&#95;SHA
+ SSL&#95;DH&#95;anon&#95;EXPORT&#95;WITH&#95;RC4&#95;40&#95;MD5
+ SSL&#95;DH&#95;anon&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ SSL&#95;DH&#95;anon&#95;WITH&#95;DES&#95;CBC&#95;SHA
+ SSL&#95;DH&#95;anon&#95;WITH&#95;RC4&#95;128&#95;MD5
+ TLS&#95;DH&#95;anon&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;DH&#95;anon&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA256
+ TLS&#95;ECDH&#95;anon&#95;WITH&#95;3DES&#95;EDE&#95;CBC&#95;SHA
+ TLS&#95;ECDH&#95;anon&#95;WITH&#95;AES&#95;128&#95;CBC&#95;SHA
+ TLS&#95;ECDH&#95;anon&#95;WITH&#95;NULL&#95;SHA
+ TLS&#95;ECDH&#95;anon&#95;WITH&#95;RC4&#95;129&#95;SHA  

이 조합들은 중간자 공격에 취약하다는 이유로 기본적으로 사용할 수 없도록 설정되어 있다. 그러나 인증서를 구매하지 않고 간단한 프로그램을 작성해 보고자 한다면 충분히 활용할 수 있다.  

### 7. SSLServerSocket 설정하기
<br/>
일단 SSLServerSocket를 성공적으로 생성하고 초기화한 다음에는, java.net.ServerSocket에서 상속받은 메소드만으로도 충분히 많은 애플리케이션을 만들 수 있다. 그러나 소켓 동작의 변경이 필요한 순간이 발생한다. SSLSocket와 같이 SSLServerSocket도 암호화 조합을 선택하고, 세션을 관리하고, 클라이언트가 스스로 인증이 필요한지를 결정하는 메소드를 제공한다. 이 메소드의 대부분은 SSLSocket에 있는 같은 이름의 메소드와 유사하게 동작한다. 차이점은 이 메소드들은 서버 측에서 동작한다는 것과 SSLServerSocket에 의해 수용된 소켓에는 기본적으로 설정된다는 것이다. 경우에 따라, SSLServerSocket을 변경하여 수용된 모든 소켓의 설정을 변경하지 않고, 수용된 특정 SSLSocket의 메소드를 호출하여 해당 소켓의 설정만을 변경할 수도 있다.  

#### 7.1. 암호화 조합 선택하기
<br/>
SSLServerSocket 클래스는 SSLSocket 클래스와 마찬가지로 어떤 암호화 조합이 지원되는지, 어떤 암호화 조합을 사용할 수 있는지를 확인하는 다음 세 가지 메소드를 제공한다.  

+ public abstract String[] getSupportedCipherSuites()
+ public abstract String[] getEnabledCipherSuites()
+ public abstract void setEnabledCipherSuites(String[] suites)  

이 메소드는 SSLSocket에서 비슷한 이름의 메소드가 사용하는 동일한 조합 이름을 사용한다. 차이점이라면 이 메소드의 호출 결과는 SSLSocket 하나가 아닌 SSLServerSocket에 의해 수용된 모든 소켓에 적용된다는 것이다.

#### 7.2. 세션 관리
<br/>
세션을 설정하기 위해서는 클라이언트와 서버가 모두 동의해야 한다. 서버 쪽에서는 허용 여부를 설정하기 위해 setEnableSessionCreation() 메소드를 사용하고, 현재 허용되어 있는지 확인하기 위해 getEnableSessionCreation() 메소드를 사용한다.  

+ public abstract void setEnableSessionCreation(boolean allowSessions)
+ public abstract boolean getEnableSessionCreation()  

기본적으로 세션 생성은 허용되어 있다. 서버가 세션 생성을 허용하지 않을 경우에도 세션 생성을 원하는 클라이언트는 여전히 연결할 수 있지만, 다만 세션을 얻지 못하고 모든 소켓에 대해 다시 핸드세이크를 해야 한다. 마찬가지로 클라이언트가 세션 생성을 거부하고 서버가 허용할 경우, 이 둘은 여전히 서로 대화를 할 수는 있지만 세션을 생성할 수는 없다.  

#### 7.3. 클라이언트 모드
<br/>
SSLServerSocket 클래스는 클라이언트 소켓이 서버에게 스스로를 인증해야 하는지를 확인하거나 설정하는 두 메소드를 제공한다. setNeedClientAuth() 메소드 호출 시 true를 인자로 전달하면, 클라이언트가 스스로를 인증할 수 있는 연결만 허용한다. false를 인자로 호출하면, 클라이언트에 대한 인증이 요구되지 않는다. 기본값은 false이다. 만약 어떤 이유로 이 설저의 현재 값이 필요한 경우, getNeedClientAuth() 메소드를 호출하면 알 수 있다.  

+ public abstract void setNeedClientAuth(boolean flag)
+ public abstract boolean getNeedClientAuth()  

setUseClientMode() 메소드는 소켓이 비록 SSLServerSocket에서 생성되었지만, 해당 소켓이 통신의 인증과 협상과 같은 상황에서 클라이언트로 처리되어야 함을 나타낸다. 예를 들어 FTP 세션에서 클라이언트 프로그램은 서버로부터 데이터를 받기 위한 서버 소켓을 열지만, 해당 소켓은 클라이언트처럼 동작해야 한다. getUseClientMode() 메소드는 SSLServerSocket이 클라이언트 모드인 경우 true를 반환하고, 아닌 경우 false를 반환한다.  

+ public abstract void setUseClientMode(boolean flag)
+ public abstract boolean getUseClientMode()
