---
title:  클라이언트 소켓
categories:
- Java Network Programming
feature_text: |
  ## 클라이언트 소켓
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

### 1. 소켓 사용하기
<br/>
#### 1.1. 소켓으로 서버에서 읽기
<br/>
소켓을 사용하여 daytime 출력 결과를 얻는 방법을 보도록 하자. 먼저 time.nist.gov의 13번 포트에 대한 소켓을 연다.  

```java
Socket socket = new Socket("time.nist.gov", 13);
```

이 코드는 객체를 만들 뿐만 아니라 실제로 네트워크를 통한 연결을 생성한다. 연결 시에 시간 초과가 발생하거나 해당 서버의 13번 포트가 대기하고 있지 않아서 실패할 경우, 생성자는 IOException 예외를 발생시킨다. 그래서 일반적으로 이 코드를 try 블록으로 감싼다. 자바 7에서 Socket 클래스는 AutoCloseable 인터페이스를 구현하고 있으므로, try-with-resources 구문을 사용하여 Socket 객체를 생성할 수 있다.  

```java
try (Socket socket = new Socket("time.nist.gov", 13)) {
  // 연결된 소켓에서 읽기...
} catch (IOException ex) {
  System.err.println("Could not connect to time.nist.gov");
}
```

자바 6과 이전 버전에서는 소켓이 가지고 있는 리소스를 해제하기 위해 finally 블록에서 명시적으로 소켓을 닫아야 한다.  

```java
Socket socket = null;
try {
  socket = new Socket(hostname, 13);
  // 연결된 소켓에서 읽기...
} catch (IOException ex) {
  System.err.println(ex);
} finally {
  if (socket != null) {
    try {
      socket.close();
    } catch (IOException ex) {
      // 무시한다
    }
  }
}
```

다음 단계는 선택 사항이지만 설정할 것을 권한다. setSoTimeout() 메소드를 사용하여 연결에 대한 타임아웃을 설정할 수 있다. 타임아웃은 밀리초 단위로 설정되며 다음 코드는 15초 동안 응답이 없을 경우 해당 소켓을 타임아웃시킨다.  

```java
socket.setSoTimeout(15000);
```

소켓은 서버가 연결을 거부하거나 라우터가 목적지로 패킷을 보내는 방법을 찾지 못할 경우 재빨리 ConnectionException나 NoRouterToHostException 예외를 발생시키지만, 이 두 경우 모두 서버가 연결을 받아들인 다음 명시적인 소켓의 종료 없이 대화를 멈추는 경우와 같은 비정상적인 상황을 대처하지는 못한다. 소켓에 타임아웃을 설정하는 것은 소켓에 대한 읽고 쓰기가 느린 경우에도 지정된 밀리초 이상을 소요하지 않을 것임을 나타낸다. 프로그램이 서버에 연결되어 있는 동안 서버가 과부하나 다양한 문제로 응답이 없는 경우, 지정된 밀리초가 지나면 프로그램은 소켓으로부터 SocketTimeoutException 예외를 받게 된다. 어느 정도의 타임아웃을 설정해야 할지는 여러분이 작성하는 프로그램과 여러분이 서버의 응답에 얼마나 민감한지에 따라 달라진다. 15초라는 시간은 로컬 인트라넷 서버의 응답으로는 꽤 긴 시간이지만, time.nist.gov와 같은 부하가 많은 공공 서버에 대해서는 다소 짧은 편이다.  

소켓은 열고 타임아웃을 설정하고 나서, getInputStream() 메소드를 호출하면 소켓으로부터 바이트를 읽는 데 사용할 수 있는 InputStream이 반환된다. 일반적으로 서버는 어떤 바이트 값이라도 보낼 수 있지만, 아래 코드에서는 서버가 아스키 문자만 보낸다고 가정하고 작성하였다.  

```java
InputStream in = socket.getInputStream();
StringBuilder time = new StringBuilder();
InputStreamReader reader = new InputStreamReader(in, "ASCII");
for (int c = reader.read(); c != -1; c = reader.read()) {
  time.append((char) c);
}
System.out.println(time);
```

### 1.2. 소켓으로 서버에 쓰기
<br/>
#### 1.2.1. 한쪽이 닫힌 소켓
<br/>
close() 메소드는 소켓의 입력과 출력을 모두 닫는다. 때때로 연결의 입력이나 출력 중에 어느 하나만 닫고 싶은 경우가 있다. shutdownInput() 그리고 shutdownOutput() 메소드는 연결의 한쪽만 닫는 데 사용된다.  

+ public void shutdownInput() throws IOException
+ public void shutdownOutput() throws IOException  

이 두 메소드 중 어느 것도 실제로 소켓을 닫지는 않는다. 대신 스트림이 끝에 도달한 것처럼 보이도록 소켓에 연결된 스트림을 조정한다. 입력을 닫은 이후에 입력 스트림에서 읽기를 시도할 경우 -1이 반환된다. 출력을 닫은 이후에 소켓에 추가적인 쓰기를 시도할 경우 IOException 예외가 발생한다.  

finger, whois 그리고 HTTP와 같은 많은 프로토콜이 클라이언트가 서버로 요청을 보내는 것으로 시작한다. 그러고 나서 클라이언트는 서버가 보낸 응답을 읽는다. 상황에 따라 클라이언트가 요청을 보내고 난 다음에 더 이상 전송할 내용이 없을 경우 출력 스트림을 닫는 것도 가능할 것이다. 예를 들어, 다음 코드는 HTTP 서버에 요청을 보낸 다음 더 이상 쓸 핑요가 없으므로 출력을 닫는다.  

```java
try {Socket connection = new Socket("www.oreilly.com", 80)} {
  Writer out = new OutputStreamWriter(connection.getOutputStream(), "8859_1");
  out.write("GET / HTTP 1.0\r\n\r\n");
  out.flush();
  connection.shutdownOutput();
  // 서버로부터 응답 읽기...
} catch (IOException ex) {
  ex.printStackTrace();
}
```

소켓의 입력 또는 출력의 어느 한쪽을 닫았거나 양쪽 모두 닫은 경우에도 소켓의 사용이 끝나면 close() 메소드를 호출하여 명시적으로 소켓을 종료시켜야 한다는 사실을 명심해야 한다. 소켓의 한쪽을 닫는 메소드는 단지 소켓의 스트림에만 영향을 줄 뿐 실제 소켓에는 영향을 주지 않으며, 소켓의 포트와 같은 사용한 리소스를 해제하지 않는다.  

isInputShutdown() 그리고 isOutputShutdown() 메소드는 각각 출력 스트림과 입력 스트림이 닫혔는지 열렸는지 유무를 반환한다. 소켓에 데이터를 읽거나 쓸 수 있는지 좀 더 구체적인 확인이 필요할 때 isConnected() 또는 isClosed()보다 이 메소드를 사용할 수 있다.  

+ public boolean isInputShutdown()
+ public boolean isOutputShutdown()  

### 2. 소켓 생성과 연결
<br/>
java.net.Socket 클래스는 클라이언트 측의 TCP 기능을 수행하기 위한 자바의 기본 클래스다. 그리고 URL, URLConnection, Applet, JEditorPane 같은 TCP 네트워크 연결을 생성하는 클라이언트 기반의 클래스들 역시 내부적으로 결국에는 java.net.Socket의 메소드를 호출한다. java.net.Socket 클래스 자체는 호스트 운영체제의 로컬 TCP 스택과 통신을 위해 네이티브 코드를 사용한다.  

#### 2.1. 기본 생성자
<br/>
각각의 소켓 생성자는 연결할 호스트와 포트를 인자로 전달받는다. 호스트 인자는 InetAddress 또는 String 타입으로 전달되며, 포트는 1에서 65355까지의 int 타입으로 전달된다.  

+ public Socket(String host, int port) throws UnknownHostException, IOException
+ public Socket(InetAddress host, int port) throws IOException  

이 생성자들은 소켓을 연결한다(즉, 생성자가 반환되기 전에 원격 호스트에 대한 실제 연결이 생성된다). 생성자가 다양한 이유로 소켓을 연결할 수 없는 경우, 생성자는 IOException 또는 UnknownHostException 예외를 발생시킨다. 예를 들어:  

```java
try {
  Socket toOReilly = new Socket("www.oreilly.com", 80);
  // 데이터를 보내고 받기...
} catch (UnknownHostException ex) {
  System.err.println(ex);
} catch (IOException ex) {
  System.err.println(ex);
}
```

생성자에서 호스트 인자는 단지 String으로 표현된 호스트네임이다. 도메인 네임 서버(DNS)가 동작하지 않거나 호스트네임을 주소로 변환할 수 없는 경우, 생성자는 UnknownHostException 예외를 발생시킨다. 그 외에 다양한 이유로 소켓을 열 수 없는 경우, 생성자는 IOException 예외를 발생시킨다. 연결 시도가 실패하는 데는 다양한 원인이 있다.  

다음 세 개의 생성자는 연결되지 않은 소켓을 생성한다. 이 생성자들은 내부 소켓이 동작에 대한 더 자세한 제어를 제공한다. 예를 들어, 다른 프록시 서버나 암호화 방법을 선택할 수 있다.  

+ public Socket()
+ public Socket(Proxy proxy)
+ protected Socket(SocketImpl impl)  

#### 2.2. 연결에 사용할 로컬 인터페이스 지정하기
<br/>
다음 두 개의 생성자는 연결할 호스트와 포트 이외에도 연결에 사용할 인터페이스와 로컬 포트를 인자로 받는다.  

+ public Socket(String host, int port, InetAddress interface, int localPort) throws IOException, UnknownHostException
+ public Socket(InetAddress host, int port, InetAddress interface, int localPort) throws IOException  

이 소켓 생성자는 마지막 두 개의 인자로 전달된 로컬 네트워크 인터페이스와 포트로부터, 처음 두 인자로 지정된 호스트와 포트로 접속한다. 네트워크 인터페이스는 이더넷 카드와 같은 물리적인 장치이거나, 하나 이상의 IP 주소를 가진 멀티홈 호스트(multi-home host)처럼 가상의 장치일 수 있다. localPort 인자로 0이 전달될 경우 자바는 1024에서 65355 사이의 사용 가능한 임의의 포트를 선택한다.  

데이터를 보내기 위해 특정 인터페이스를 선택하는 경우는 일반적이지는 않지만, 가끔 필요한 경우가 있다. 로컬 주소를 명시적으로 지정해야 하는 상황의 한 가지 예로 이중(dual) 이더넷 포트를 사용하는 라우터/방화벽이 있다. 라우터/방화벽은 하나의 인터페이스를 통해 외부의 연결을 받아들이고, 처리한 다음 다른 인터페이스를 통해 로컬 네트워크로 전달한다. 그리고 독자 여러분이 주기적으로 에러 로그를 프린터로 출력하거나 내부 메일 서버로 전송하는 프로그램을 작성하고 있다고 가정해 보자. 이러한 경우 해당 패킷이 외부 인터페이스가 아닌 내부 인터페이스로 전송되도록 다음과 같이 할 수 있다.  

```java
try {
  InetAddress inward = InetAddress.getByName("router");
  Socket socket = new Socket("mail", 25, inward, 0);
  // 소켓을 이용한 데이터를 보내거나 받는 작업들...
} catch (IOException ex) {
  System.err.println(ex);
}
```

위 코드에서 로컬 포트 숫자로 0을 전달함으로써, 어떤 임의의 포트를 사용해도 괜찮으나, 로컬 호스트네임 "router"에 연결된 네트워크 인터페이스를 사용할 것임을 요청했다.  

이 생성자는 이전의 생성자와 같은 이유로 IOException 또는 UnknownHostException 예외를 발생시킨다. 이외에도 소켓이 요청된 로컬 네트워크 인터페이스를 바인드(bind)할 수 없는 경우 IOException 예외를 발생시킨다. (이 메소드의 예외 조항에 구체적으로 명시되어 있지는 않지만, 아마도 IOException의 서브클래스인 BindException이 발생할 것이다.) 예를 들어, a.example.com에서 실행 중인 프로그램은 b.example.org에서 접속할 수 없다. 이런 점을 이용하면 컴파일된 프로그램이 미리 정해진 호스트에서만 실행되도록 의도적으로 제한할 수 있다. 이 방법은 각 컴퓨터에 맞게 설정된 배포가 필요하며, 확실히 값싼 제품에 대해 과도한 기능이다. 게다가 자바 프로그램은 쉽게 디스어셈플(disassemble), 디컴파일(decompile), 그리고 리버스 엔지니어(reverse engineer)할 수 있기 때문에, 이 구조가 확실히 안전하지는 않다. 그럼에도 불구하고 소프트웨어 라이선스를 적용하기 위한 방법으로 종종 사용된다.  

#### 2.3. 연결되지 않은 소켓 생성하기
<br/>
지금까지 이야기한 모든 생성자는 소켓 객체를 생성하고 원격 호스트에 대한 네트워크 연결을 여는 두 작업을 함께 수행한다. 가끔 이 두 작업을 따로 수행해야 할 필요가 있다. Socket 생성자를 아무런 인자 없이 호출하면, 생성자는 연결되지 않은 소켓을 반환한다.  

+ public Socket()  

소켓 생성 이후에 SocketAddress를 인자로 connect() 메소드 중 하나를 호출하여 연결할 수 있다. 예를 들어:  

```java
try {
  Socket socket = new Socket();
  // 소켓 옵션 입력
  SocketAddress address = new InetSocketAddress("time.nist.gov", 13);
  socket.connect(address);
  // 연결된 소켓으로 작업...
} catch (IOException ex) {
  System.err.println(ex);
}
```

connect() 메소드의 두 번째 인자로 연결 타임아웃 시간을 밀리초 단위로 설정할 수 있다.  

+ public void connect(SocketAddress endpoint, int timeout) throws IOException  

기본 타임아웃 값은 0이며, 무한히 대기한다.  

이 생성자는 기본적으로 다른 종류의 소켓을 사용하기 위해 존재하며, 또한 소켓 연결 이전에 변경해야 적용되는 옵션을 설정할 때 필요하다. 그러나 필자가 생각하는 이 생성자의 가장 큰 장점은 try - catch - finally 블록에서 코드를 깔끔하게 정리할 수 있다는 것이다. 특히 자바 7 이전 버전에서 더욱 효과적이다. 인자가 없는 생성자는 예외를 발생시키지 않기 때문에 finally 블록에서 소켓을 닫을 때 불필요한 널(null) 체크를 하지 않아도 된다. 인자로 필요한 생성자를 호출할 경우 대부분 다음과 같이 코드를 작성한다.  

```java
Socket socket = null;
try {
  socket = new Socket(SERVER, PORT);
  // 소켓을 이용한 작업
} catch (IOException ex) {
  System.err.println(ex);
} finally {
  if (socket !- null) {
    try {
      socket.close();
    } catch (IOException ex) {
      // 무시한다
    }
  }
}
```

인자가 없는 생성자를 호춣할 경우에는 다음과 같이 코드를 작성할 수 있다.  

```java
Socket socket = new Socket();
SocketAddress address = new InetSocketAddress(SERVER, PORT);
try {
  socket.connect(address);
  // 소켓을 이용한 작업
} catch (IOException ex) {
  System.err.println(ex);
} finally {
  try {
    socket.close();
  } catch (IOException ex) {
    // 무시한다
  }
}
```

자바 7 버전의 AutoCloseable를 이용했을 때만큼은 아니지만, 코드가 꽤 깔끔해졌다.  

#### 2.4. 소켓 주소
<br/>
SocketAddress 클래스는 연결 끝점(endpoint)을 나타낸다. 이 클래스는 기본 생성자를 제외하고는 아무런 메소드를 제공하지 않는 빈 추상 클래스다. SocketAddress 클래스는 적어도 이론적으로는 TCP와 비TCP 소켓 둘 모두에 사용될 수 있다. 실제로는 현재 TCP/IP 소켓만이 지원되며, 여러분이 사용하는 소켓 주소들은 모두 InetSocketAddress의 인스턴스이다.  

SocketAddress 클래스의 기본적인 의도는 원본 소켓이 연결이 끊어지거나 가비지 컬렉터에 의해 사라진 경우에도, 새로운 소켓을 생성하는 데 재사용할 수 있는 IP 주소와 포트 같은 일시적인 소켓 연결 정보를 위한 간편한 저장소를 제공하는 것이다. 이를 위해 Socket 클래스는 SocketAddress 객체를 반환하는 두 개의 메소드를 제공한다(getRemoteSocketAddress() 메소드는 연결된 시스템에 대한 주소를 반환하고 getLocalSocketAddress() 메소드는 연결을 만든 곳의 주소를 반환한다).  

+ public SocketAddress getRemoteSocketAddress()
+ public SocketAddress getLocalSocketAddress()  

이 두 메소드는 소켓이 아직 연결되지 않은 경우에 널(null)을 반환한다. 예를 들어, 먼저 Yahoo!에 연결한다. 그리고 주소를 저장한다.  

```java
Socket socket = new Socket("www.yahoo.com", 80);
SocketAddress yahoo = socket.getRemoteSocketAddress();
socket.close();
```

나중에 이 주소를 사용하여 Yahoo!에 다시 연결할 수 있다.  

```java
Socket socket2 = new Socket();
socket2.connect(yahoo);
```

InetSocketAddress 클래스(이 클래스는 JDK에서 ScoketAddress의 유일한 서브클래스이며, 필자가 지금까지 본 유일한 서브클래스이기도 하다)는 일반적으로 클라이언트의 경우에 호스트와 포트 또는 서버의 경우에 포트만 사용하여 만들어진다.  

+ public InetSocketAddress(InetAddress address, int port)
+ public InetSocketAddress(String host, int port)
+ public InetSocketAddress(int port)  

그리고 DNS에서 해당 호스트를 검색하는 과정을 생략하기 위해 정적 팩토리 메소드 InetSocketAddress.createUnresolved() 또한 사용할 수 있다.  

+ public static InetSocketAddress createUnresolved(String host, int port)  

InetSocketAddress는 객체를 확인하는 데 사용할 수 있는 일부 get 메소드를 제공한다.  

+ public final InetAddress getAddress()
+ public final int getPort()
+ public final String getHostName()  

#### 2.5. 프록시 서버
<br/>
Socket 클래스의 마지막 생성자는 인자로 전달된 프록시 서버를 통해 연결하는 연결되지 않은 소켓을 생성한다.  

+ public Socket(Proxy proxy)  

일반적으로 소켓이 사용하는 프록시 서버는 socksProxyHost와 socksProxyPort 시스템 속성에 의해 제어된다. 그리고 이 속성들은 시스템에 있는 모든 소켓에 적용된다. 그러나 이 생성자에 의해 생성된 소켓은 대신 인자로 지정된 프록시 서버를 사용한다. 주목해야 할 점은, 인자로 Proxy.NO&#95;PROXY를 전달하면 모든 프록시 서버 설정을 무시하고 원격 호스트에 직접 연결할 수 있다. 물론 방화벽이 직접적인 연결을 차단할 경우, 자바도 어떻게 할 방법이 없으며 연결은 실패한다.  

특정 프록시 서버를 사용하기 위해서는 해당 서버의 주소를 지정하면 된다. 예를 들어, 다음 코드는 login.ibiblio.org 호스트에 접속하기 위해 myproxy.example.com에 위치한 SOCKS 프록시 서버를 사용한다.  

```java
SocketAddress proxyAddress = new InetSocketAddress("myproxy.example.com", 1080);
Proxy proxy = new Proxy(Proxy.Type.SOCKS, proxyAddress);
Socket s = new Socket(proxy);
SocketAddress remote = new InetSocketAddress("login.ibiblio.org", 25);
s.connect(remote);
```

자바는 저수준 프록시 타입 중에서 SOCKS 프록시를 유일하게 지원한다. 그리고 전송 계층에서 동작하는 SOCKS 프록시 이외에도 애플리케이션 계층에서 동작하는 고수준 Proxy.Type.HTTP 역시 지원한다. 마지막으로 Proxy.Type.DIRECT는 프록시를 사용하지 않는 연결을 나타낸다.  

### 3. 소켓 정보 얻기
<br/>
소켓 객체는 get 메소드를 사용하여 접근할 수 있는 몇 가지 속성을 제공한다.  

+ 원격 주소
+ 원격 포트
+ 로컬 주소
+ 로컬 포트  

다음은 이러한 속성에 접근할 수 있는 get 메소드다.  

+ public InetAddress getInetAddress()
+ public int getPort()
+ public InetAddress getLocalAddress()
+ public int getLocalPort()  

set 메소드는 존재하지 않는다. 이 속성들은 소켓이 연결되자마자 설정되며 고정되어 있다.  

getInetAddress() 그리고 getPort() 메소드는 소켓이 연결된 원격 호스트와 포트를 알려 준다. 또는 연결이 종료된 경우 연결된 당시의 호스트와 포트를 알려 준다. getLocalAddress()와 getLocalPort() 메소드는 연결이 시작된 네트워크 인터페이스와 포트를 알려 준다.  

표준 위원회에서 미리 할당된 "알려진 포트(well-known port)"를 사용하는 원격 포트와 달리, 로컬 포트는 일반적으로 실행 시점에 이용 가능한 포트 중에서 시스템에 의해 결정된다. 이러한 구조로 인해, 단일 시스템에서 실행 중인 많은 클라이언트가 동시에 같은 서비스에 접근하는 것이 가능해진다. 로컬 포트는 로컬 호스트의 IP 주소와 함께 밖으로 나가는 IP 패킷에 박혀 있다. 그래서 서버가 클라이언트의 올바른 포트로 응답을 보낼 수 있다.  

#### 3.1. 종료된 소켓과 연결된 소켓
<br/>
isClosed() 메소드는 소켓이 닫혀 있는 경우 true를 반환하고 그렇지 않은 경우 false를 반환한다. 소켓의 상태가 불확실한 경우 IOException이 발생할 위험을 감소하는 것보다 이 메소드를 사용하여 확인하는 것이 좋다. 예를 들어:  

```java
if (socket.isClosed()) {
  // 소켓이 닫혀 있는 경우 처리...
} else {
  // 소켓이 열려 있는 경우 처리...
}
```

그러나 이 확인 방법이 완벽하지는 않다. 만약에 소켓이 최초에 연결된 적이 없는 경우 소켓은 확실히 열려 있지는 않지만 isClosed() 메소드는 false를 반환한다.  

Socket 클래스는 또한 isConnected() 메소드를 제공한다. 이 메소드의 이름은 약간 오해의 소지가 있다. 이 메소드는 해당 소켓이 현재 원격 호스트에 연결되었는지 알려 주지 않는다. 대신 소켓이 원격 호스트에 연결된 적이 있는지 여부를 알려 준다. 소켓의 최근 연결 요청이 성공했다면 소켓이 닫힌 후에도 true를 반환한다.  

소켓이 현재 열려 있는지 확인하기 위해서는 isConnected() 반환값이 true이고 isClosed() 반환값이 false인 두 가지 조건을 모두 확인해야 한다. 예를 들어:  

```java
boolean connected = socket.isConnected() && ! socket.isClosed();
```

마지막으로, isBound() 메소드는 소켓이 로컬 시스템의 나가는 포트에 성공적으로 바인딩되었는지 여부를 알려 준다. isConnected() 메소드는 소켓의 원격 끝 부분을 참조하는 반면, isBound() 메소드는 소켓의 로컬 시스템 끝 부분을 참조한다.  

#### 3.2. toString() 메소드
<br/>
Socket 클래스는 java.lang.Object의 표준 메소드에서 유일하게 toString() 메소드만을 오버라이드한다. Socket 클래스의 toString() 메소드는 다음과 같은 문자열을 생성한다.  

Socket[add=www.oreilly.com/192.112.208.11,port=80,localport=50055]  

이 정보는 주로 디버깅에 유용하게 사용된다. 이 문자열의 형식은 언제든지 변경될 수 있기 때문에 이 형식의 의존적인 코드를 작성해서는 안 된다. 이 문자열의 모든 부분은 get 메소드를 사용하여 직접 접근할 수 있다. [구체적으로는 getInetAddress(), getPort(), 그리고 getLocalPort() 메소드가 있다.]  

소켓은 연결이 유지되는 한 가장 최근 정보만을 유지하는 일시적인 객체이기 때문에, 이 정보들을 해시 테이블에 저장하거나 서로 비교해야 할 충분한 이유가 없다. 그래서 Socket은 equals() 또는 hashCode() 같은 메소드를 오버라이드하지 않으며, 이 두 메소드는 Object 클래스에서 제공되는 의미 그대로 사용된다. 결국 두 소켓 객체가 같은 객체일 때만 서로 같다.  

### 4. 소켓 옵션 설정하기
<br/>
소켓 옵션은 자바 Socket 클래스 내부의 네이티브 소켓이 데이터를 보내거나 받는 방법을 지정한다. 자바는 클라이언트 측 소켓에 대해 다음 9가지 옵션을 제공한다.  

+ TCP&#95;NODELAY
+ SO&#95;BINDADDR
+ SO&#95;TIMEOUT
+ SO&#95;LINGER
+ SO&#95;SNDBUF
+ SO&#95;RCVBUF
+ SO&#95;KEEPALIVE
+ OOBINLINE
+ IP&#95;TOS  

낯설어 보이는 이 옵션의 이름은 소켓을 처음 개발한 버클리 유닉스의 C 헤더 파일에 있는 상수의 이름을 빌려 왔기 때문이다. 게다가 익숙한 자바 명명 규칙으로 변환하지 않고 전통적인 유닉스 C 명명 규칙을 그대로 따르고 있다. 예를 들어, SO&#95;SNDBUF는 실제 "Socket Option Send Buffer Size(소켓의 전송 버퍼 크기 옵션)"을 의미한다.  

#### 4.1. TCP&#95;NODELAY
<br/>
+ public void setTcpNoDelay(boolean on) throws SocketException
+ public boolean getTcpNoDelay() throws SocketException  

TCP&#95;NODELAY 설정을 true로 하면 패킷의 크기에 상관없이 가능한 한 빨리 패킷을 전송한다. 일반적으로 작은 패킷은 전송하기 전에 큰 패킷 하나로 합쳐진다. 그리고 또 다른 패킷을 보내기 전에 로컬 호스트는 원격 호스트로부터 이전 패킷에 대한 확인(ACK)을 기다린다. 이것이 바로 잘 알려진 네이글 알고리즘(Nagle's algorithm)이다. 네이글 알고리즘의 문제는 원격 시스템이 로컬 시스템으로 확인(ACK)을 충분히 빨리 보내지 않은 경우, 작은 단위의 데이터를 지속적으로 보내야 하는 애플리케이션의 경우 느려지는 것이다. 특히 서버에서 실시간으로 클라이언트 측의 마우스 움직임을 추적해야 한는 게임이나 네트워크 애플리케이션과 같은 GUI 프로그램에서 더욱 문제가 된다. 실제로 느린 네트워크에서는 지속적인 버퍼링으로 인해 심지어 단순한 타이핑마저 느려질 수 있다. TCP&#95;NODELAY 옵션을 true로 설정하면 이러한 버퍼링 구조를 사용하지 않으며 모든 패킷이 준비되는 즉시 전송된다.  

setTcpNoDelay(true)는 소켓의 버퍼링 옵션을 끈다. setTcpNoDelay(false)은 버퍼링 옵션을 다시 켠다. getTcpNoDelay()는 버퍼링 옵션이 켜 있는 경우 false를 반환하고 반대인 경우 true를 반환한다. 예를 들어, 다음은 소켓의 버퍼링 설정 상태를 확인하고 버퍼링을 끈다(즉, TCP&#95;NODELAY를 설정한다).  

```java
if (!s.getTcpNoDelay()) s.setTcpNoDelay(true);
```

위에 사용된 두 메소드는 내부의 소켓 구현에서 TCP&#95;NODELAY 옵션을 지원하지 않을 경우 SocketException 예외를 발생시키도록 선언되어 있다.  

#### 4.2. SO&#95;LINGER
<br/>
+ public void setSoLinger(boolean on, int seconds) throws SocketException
+ public int getSoLinger() throws SocketException  

SO&#95;LINGER 옵션은 소켓이 닫힐 때, 전송되지 않은 데이터그램을 어떻게 처리할지 결정한다. 기본적으로 close() 메소드는 호출 즉시 반환된다. 그러나 시스템은 내부적으로 아직 전송되지 않은 데이터를 계속해서 전송한다. 링거(linger) 시간이 0으로 설정될 경우, 소켓이 닫힐 때 아직 전송되지 않은 패킷은 버려진다. SO&#95;LINGER 옵션은 켜 있고 링거 타임이 정수값인 경우, close() 메소드는 지정된 시간 동안 데이터를 보내고 응답을 받기 위해 블록(block)된다. 그리고 지정된 시간이 초과될 경우, 소켓은 닫히고 아직 남아 있는 데이터는 보내지 않으며 응답을 기다리지 않는다.  

이 두 메소드는 내부의 소켓 구현이 SO&#95;LINGER 옵션을 지원하지 않을 경우 SocketException 예외를 발생시킨다. setSoLinger() 메소드는 또한 링거 타임을 음수로 전달할 경우 IllegalStateException 예외를 발생시킨다. 그러나 getSoLinger() 메소드는 옵션이 설정되지 않은 경우 -1을 반환하거나, 남아 있는 데이터를 모두 전송하는 데 필요한 시간을 반환한다. 예를 들어, 다음 코드는 지연 시간이 설정되어 있지 않은 경우, 소켓 s에 대해 링거 타임아웃을 4분으로 설정한다.  

```java
if (s.getTcpSoLinger() == -1) s.setSoLinger(true, 240);
```

최대 링거 타임은 6만 5,535초이며 플랫폼에 따라 더 작을 수도 있다. 최대값보다 큰 값이 전달될 경우 최대 링거 타임으로 자동으로 맞춰진다. 사실 6만 5,535초(18시간이 넘는다)는 일반적으로 필요한 지연 시간보다 훨씬 긴 시간이다. 일반적으로 플랫폼 기본값을 사용하는 편이 좋다.  

#### 4.3. SO&#95;TIMEOUT
<br/>
+ public void setSoTimeout(int millseconds) throws SocketException
+ public int getSoTimeout() throws SocketException  

일반적으로 소켓에서 데이터를 읽으려고 할 때 read() 호출은 충분한 바이트를 읽을 때까지 블록된다. SO&#95;TIMEOUT을 설정하면 read() 메소드의 호출이 지정된 밀리초 이상 블록되지 않느다. 타임아웃이 발생하면 InteruptedIOException 예외가 발생하므로 이 예외를 처리할 수 있도록 해야 한다. 하지만 예외가 발생한 뒤에도 소켓은 여전히 연결을 유지하고 있다. read() 메소드는 호출이 실패한 경우에도, 다시 호출할 수 있으며, 이전 실패와 상관없이 호출이 성공할 수 있다.  

타임아웃은 밀리초로 설정된다. 타임아웃의 기본 값은 0이며 무한 대기를 의미한다. 예를 들어, 다음 코드는 소켓 객체 s의 타임아웃이 아직 설정되지 않은 경우 18만 밀리초를 설정하는 코드다.  

```java
if (s.getSoTimeout() == 0) s.setSoTimeout(180000);
```

이 두 메소드는 내부 소켓 구현이 SO&#95;TIMEOUT 옵션을 지원하지 않는 경우 SocketException 예외를 발생시킨다. 그리고 또한 setSoTimeout() 메소드는 지정된 타임아웃 값이 음수일 경우 IllegalStateException 예외를 발생시킨다.  

#### 4.4. SO&#95;SNDBUF 그리고 SO&#95;RCVBUF
<br/>
TCP는 네트워크 성능을 향상시키기 위해 버퍼를 사용한다. 네트워크 속도가 빠른 경우 즉, 속도가 10Mbps 이상인 경우, 큰 버퍼가 성능에 유리하고 전화 연결과 같이 네트워크 속도가 느린 경우 작은 버퍼가 성능에 더 유리하다. 일반적으로 FTP 혹은 HTTP와 같은 파일 전송 프로토콜에서는 크기가 큰 데이터를 지속적으로 보내므로 버퍼의 크기가 큰 편이 유리하다. 반면에 지속적인 상호작용이 일어나는 텔넷이나 게임에서는 버퍼의 크기가 작은 편이 유리하다. BSD 4.2와 같은 파일의 크기가 작고 네트워크가 느리던 시대에 설계된 상대적으로 오래된 운영체제의 경우, 2킬로바이트 버퍼를 사용한다. 윈도우 XP는 1만 7,520바이트 버퍼를 사용하며 요즘 운영체제에서는 128킬로바이트가 일반적인 기본값이다.  

달성 가능한 최대 대역폭은 버퍼 크기를 대기 시간(latency)으로 나눈 값과 같다. 예를 들어 윈도우 XP에서 두 호스트 사이의 지연 시간이 500ms라고 가정해 보자. 그때 대역폭은 17520바이트 / 0.5초 = 35040바이트 / 초 = 273.75킬로비트 / 초가 된다. 이 값이 곧 네트워크의 속도에 상관없이 어떤 소켓의 최대 속도가 된다. 이 값은 전화 접속을 사용하는 경우 충분히 빠르고, ISDN(Integrated Services Digital Network)을 사용하는 경우 나쁘지 않다. 그러나 DSL 회선이나 FiOS(미국 버라이즌의 통신 상품명)를 사용하는 경우 많이 부족하다.  

대기시간을 줄이면 속도를 향상시킬 수 있다. 그러나 대기 시간은 네트워크 하드웨어의 기능이고 여러분의 애플리케이션에서 제어할 수 있는 요소가 아니다. 반면에 버퍼 크기는 제어할 수 있다. 예를 들어, 버퍼 크기를 두 배인 256킬로바이트로 늘리면 최대 대역폭은 두 배인 초당 4메가비트로 증가한다. 물론 네트워크 자체의 최대 대역폭 제한이 존재한다. 버퍼를 너무 높게 설정하면 프로그램이 네트워크가 처리할 수 있는 것보다 더 바르게 데이터를 전송하려고 한다. 이런 경우 네트워크가 혼잡해져 패킷이 손실되며 결국 성능저하로 이어진다. 따라서 최대 대역폭이 필요한 경우 버퍼 크기를 연결의 대기 시간과 맞출 필요가 있으며, 결국 이 값은 네트워크의 대역폭보다 조금 작은 값이 된다.  

특정 호스트에 대한 지연 시간을 수동으로 측정하기 위해 ping을 사용하거나, 프로그램 내에서 InetAddress.isReachable()를 호출하여 시간을 측정할 수 있다.  

SO&#95;SNDBUF 옵션은 네트워크의 입력에 사용된 수신 버퍼의 크기를 제안한다. SO&#95;SNDBUF 옵션은 네트워크의 출력에 사용된 송신 버퍼의 크기를 제안한다.  

+ public void setReceiveBufferSize(int size) throws SocketException, IllegalArgumentException
+ public int getReceiveBufferSize() throws SocketException
+ public void setSendBufferSize(int size throws SocketException, IllegalArgumentException
+ public int getSendBufferSize() throws SocketException  

메소드 선언에는 송신 버퍼와 수신 버퍼를 각각 설정할 수 있는 것처럼 보이지만, 일반적으로 버퍼는 이 두 값보다 작게 설정된다. 예를 들어, 송신 버퍼의 크기를 64K로 설정하고 수신 버퍼의 크기를 128K로 설정하면, 송신과 수신 모두 64K의 버퍼 크기를 가지게 될 것이다. 그리고 설정된 수신 버퍼의 크기를 128K라고 알려 주지만, 내부 TCP 스택은 실제로 64K를 사용할 것이다.  

setReceiveBufferSize() / setSendBufferSize 메소드는 이 소켓에 대한 출력을 버퍼링하기 위해 사용할 바이트 수를 제안한다. 그러나 내부 구현은 자유롭게 이 제안을 무시하거나 조정할 수 있다. 특히 유닉스와 리눅스 시스템은 종종 최대 버퍼 크기를 명시하고 있는데, 일반적으로 64K 또는 256K이고, 더 큰 값의 소켓은 허용되지 않는다. 더 큰 값을 설정하려고 시도하면 자바는 버퍼 크기를 가능한 한 최대값으로 설정한다. 리눅스의 내부 구현에서 드물게 요청된 크기의 두 배를 설정하는 경우도 있다. 예를 들어, 64K 버퍼를 요청한다면 대신 128K 버퍼를 얻을 수도 있다.  

이 두 메소드는 전달된 인자가 0보다 작거나 0인 경우, IllegalArgumentException 예외를 발생시킨다. 그리고 또한 이 두 메소드는 SocketException 예외를 발생시키도록 선언되어 있지만, SocketException 예외가 IllegalArgumentException 예외와 발생 이유가 같고, IllegalArgumentException 예외가 먼저 처리되도록 만들기 때문에, 실제로 처리할 일은 없을 것이다.  

일반적으로 프로그램이 이용 가능한 대역폭을 충분히 활용하지 못할 경우(예를 들어, 25Mbps 인터넷 연결을 사용하고 있지만, 1.5Mbps의 속도로 데이터가 전송될 때), 버퍼 크기를 늘리도록 하자. 그러나 네트워크의 한쪽 방향으로 부하가 쏠리는 경우를 제외한 대부분의 상황에서는 기본값을 사용하는 것이 좋다. 특히 요즘 운영체제들은 버퍼 크기를 네트워크에 맞게 동적으로 조정하기 위해 TCP 윈도우 스케일링(TCP window scaling)을 사용한다. (자바에서 제어할 수는 없다.) 대부분의 성능 관련 조언과 마찬가지로, 문제를 정확히 측정하기 전까지는 경험에 의거하여 설정을 바꾸지 않도록 해야 한다. 그렇다 하더라도 개별 소켓의 버퍼 크기를 조절하는 것보다는 운영체제 수준에서 최대로 허용되는 버퍼 크기를 늘림으로써 더 나은 속도를 얻을 수 있다.  

#### 4.5. SO&#95;KEEPALIVE
<br/>
SO&#95;KEEPALIVE 옵션이 설정된 경우, 클라이언트는 이따금씩 유휴 연결(idle connection)을 통해 데이터 패킷을 보내어(일반적으로 두 시간마다 한 번씩 보낸다) 서버와의 연결을 유지한다. 서버가 이 패킷에 대해 응답을 보내지 않을 경우, 클라이언트는 응답을 받을 때까지 11분 이상을 계속해서 시도한다. 그리고 12분 내에 응답을 받지 못할 경우, 클라이언트는 소켓을 닫는다. SO&#95;KEEPALIVE 설정이 없다면 네트워크 통신이 활발하지 않은 클라이언트는 서버가 장애로 종료된 상황에도 아무런 알림을 받지 못하고 계속 실행된다. 아래는 SO&#95;KEEPALIVE 설정을 변경하거나 현재의 상태를 확인하는 메소드다.  

+ public void setKeepAlive(boolean on) throws SocketException
+ public boolean getKeepAlive() throws SocketException  

SO&#95;KEEPALIVE의 기본 상태는 false이다. 아래 코드는 SO&#95;KEEPALIVE 설정이 켜 있는 경우 설정을 끈다.  

```java
if (s.getKeepAlive()) s.setKeepAlive(false);
```
#### 4.6. OOBINLINE
<br/>
TCP는 단일 바이트를 긴급하게 전송하는 기능을 제공한다. 이 데이터는 전송 즉시 보내진다. 그리고 수신자는 긴급 데이터를 수신했을 때 알림을 받으며, 이미 도착한 다른 데이터를 처리하기 전에 긴급 데이터를 먼저 처리해야 한다. 자바는 긴급한 데이터를 보내고 받는 기능을 지원한다. 전송 메소드의 이름은 메소드의 기능을 잘 반영한 sendUrgentData()이다.  

+ public void sendUrgentData(int data) throws IOException  

이 메소드는 인자로 전달된 값의 하위 8비트를 거의 즉시 전송한다. 필요하다면 현재 캐시에 저장된 데이터를 먼저 플러시(flush)한다.  

수신 측에서 긴급 데이터에 대해 응답하는 방법은 약간 혼동될 수 있으며, 플랫폼과 API에 따라 다양하다. 몇몇 시스템의 경우 일반적인 데이터와는 별도로 긴급 데이터를 수신한다. 그러나 좀 더 일반적이고 현대적인 접근 방법은 긴급 데이터를 일반 데이터 수신 큐에 적잘한 순서로 저장하고 애플리케이션에게 긴급 데이터가 있음을 알린다. 그리고 애플리케이션이 큐에서 해당 데이터를 찾게 하는 것이다.  

기본적으로 자바는 소켓에서 수신된 긴급 데이터를 무시한다. 그러나 긴급 데이터를 보통의 데이터와 함께 수신하고자 할 경우 다음 메소드를 사용하여 OOBINLINE 옵션을 설정하면 된다.  

+ public void setOOBInline(boolean on) throws SocketException
+ public boolean getOOBInline() throws SocketException  

OOBINLINE의 기본 값은 false이다. 아래 코드는 OOBINLINE 설정이 꺼져 있는지 확인하고 켠다.  

```java
if (!s.getOOBInline()) s.setOOBInline(true);
```

일단 OOBINLINE이 설정되면 긴급 데이터 도착 시 일반적인 방법으로 읽을 수 있도록 소켓의 입력 스트림에 저장된다. 자바는 이 데이터를 일반 데이터와 구별하지 않는다. 이러한 구조는 이상적인 방법과는 거리가 조금 있지만, 프로그램에서 정상적인 데이터 스트림에서 사용되지 않는 특별한 의미가 부여된 특정 바이트(예를 들어, Ctrl-C)를 사용할 경우, 해당 바이트를 좀 더 빠르게 전송하는 것이 가능해진다.  

#### 4.7. SO&#95;REUSEADDR
<br/>
소켓이 종료될 때, 로컬 포트를 즉시 해제하지 않는 경우가 있다. 특히 소켓을 닫을 때 열린 연결이 있는 경우 포트가 즉시 해제되지 않을 수 있다. 소켓은 종료될 때 아직 네트워크를 통해 해당 포트로 전송 중인 나머지 패킷이 있는 경우에는 때로 일정 시간 동안 기다린다. 시스템은 늦게 도착한 패킷을 위해 특별한 뭔가를 하지는 않는다. 다만 시스템은 같은 포트에 바운드(bound)된 새 프로세스에게 늦게 패킷이 전달되지 않도록 한다.  

소켓이 임의의 포트를 사용할 때는 큰 문제가 되지 않지만, 잘 알려진 포트(well-known port)를 사용할 때는 문제가 된다. 일정 시간 동안 다른 소켓이 해당 포트를 사용할 수 없도록 막기 때문이다. SO&#95;REUSEADDR 옵션이 켜진 경우(기본적으로 꺼져 있다) 이전 소켓으로 전송된 데이터가 남아 있는 경우에도 또 다른 소켓이 해당 포트를 바인드(bind)할 수 있다.  

자바에서 이 옵션은 다음 두 메소드로 제어된다.  

+ public void setReuseAddress(boolean on) throws SocketException
+ public boolean getReuseAddress() throws SocketException  

이 기능의 동작을 위해서는 새로운 소켓이 포트에 바인드되기 전에 setReuseAddress() 메소드가 호출되어야 한다. 이 말은 곧 먼저 안지가 없는 소켓 생성자를 사용하여 연결되지 않은 상태의 소켓을 만들어야 한다는 의미다. 그러고 나서 setReuseAddress(true)를 호출한 다음 connect() 메소드를 호출하여 연결한다. 이전에 연결된 소켓과 이전의 주소를 재사용하는 새 소켓은 SO&#95;REUSEADDR을 true로 설정해야 효과를 볼 수 있다.  

#### 4.8. IP&#95;TOS 서비스 클래스
<br/>
인터넷 서비스마다 서로 다른 네트워크 성능이 필요하다. 예를 들어, 화상 채팅은 좋은 성능을 위해 상대적으로 높은 대역폭과 낮은 지연 시간(latency)이 필요한 반면, 이메일은 낮은 대역폭의 연결에서도 잘 전송되며 심지어 몇 시간 동안 지연되더라도 큰 문제가 되지 않는다. VOIP의 경우 비디오보다 작은 대역폭을 필요로 하지만, 패킷 전달 지연 시간의 편차가 거의 없어야 한다. 사람들이 항상 성능이 가장 좋은 서비스를 요청하지 않도록 서비스의 클래스마다 차등을 두어 가격을 책정하는 것이 현명하다.  

서비스의 클래스는 IP 헤더에서 IP&#95;TOS라고 불리는 8비트 필드에 저장된다. 자바는 이 값을 확인하거나 설정할 수 있는 다음 두 개의 메소드를 제공한다.  

+ public int getTrafficClass() throws SocketException
+ public void setTrafficClass(int trafficClass) throws SocketException  

setTrafficClass() 메소드의 인자인 트래픽 수준(Traffic Class)은 0에서 255까지의 정수로 전달된다. 전달된 정수를 TCP 헤더의 8비트 필드로 복사하기 위해, 인자의 하위 한 바이트만 사용한다. 그리고 범위를 벗어난 값이 전달된 경우 IllegalArgumentException 예외가 발생한다.  

21세기 TCP 스택에서 이 바이트의 상위 6비트는 DSCP(Differentiated Services Code Point) 값을 포함하고 있으며, 하위 2비트는 ECN(Explicit Congestion Notification) 값을 포함하고 있다. 따라서 DSCP는 최대 64의 서로 다른 트래픽 수준을 위한 공간을 제공한다. 그러나 이 64개의 서로 다른 DSCP 값의 의미는 개별 네트워크와 라우터에서 지정하기에 따라 달라진다. 다음 표는 흔히 사용되는 네 개의 값을 나타낸다.  

<table>
  <thead>
    <tr>
      <td>홉별 행위(PHB, Per Hop Behavior)</td>
      <td>이진값</td>
      <td>목적</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>기본</td>
      <td>00000</td>
      <td>최선의 노력(Best-effort)으로 전송</td>
    </tr>
    <tr>
      <td>신속한 전달(expedited Forwarding)</td>
      <td>101110</td>
      <td>낮은 손실, 낮은 지연, 낮은 지터 트래픽, 가끔 네트워크 용량의 30퍼센트 이하로 제한된다.</td>
    </tr>
    <tr>
      <td>확실한 전달(Assured Forwarding)</td>
      <td>xxx000</td>
      <td>IPv4 TOS 헤더의 처음 3비트에 저장된 값에 대한 하위 호환성</td>
    </tr>
  </tbody>
</table>
<br/><br/>

예를 들어, VOIP 서비스의 경우 신속한 전달(EF, Expedited Forwarding) PHB가 좋은 선택이다. EF 트래픽은 종종 다른 모든 트래픽 클래스보다 엄격한 우선순위 큐를 제공한다. 다음 코드는 트래픽 클래스를 10111000으로 설정하여 해당 소켓으로 EF를 사용하도록 설정한다.  

```java
Socket s = new Socket("www.yahoo.com", 80);
s.setTrafficClass(0xBB); // 바이너리로 10111000
```

여기에 지정된 이진값에서 하위 2비트는 네트워크 혼잡 알림(Explicit Congestion Notification) 비트이며 0으로 설정한다.  

확실한 전달(Assured Forwarding)은 실제 12개의 서로 다른 값으로 되어 있으며, 이 값은 다음 표에서 보는 것처럼 네 개의 클래스로 분리된다. 이 값들은 네트워크가 혼잡할 때 송신자가 버릴 패킷의 상대적인 우선순위를 나타낸 수 있도록 한다. 클래스 내에서 우선순위가 낮은 패킷은 우선순위가 높은 패킷보다 먼저 버려진다. 우선순위가 낮은 클래스가 완전히 고갈되지 않지만 클래스 사이에 우선순위가 높은 클래스의 패킷은 기본 제공된다. 클래스들 사이에서 클래스들 사이에서 비록 낮은 우선순위 클래스가 완전히 고갈되지 않더라도 높은 우선순위 클래스의 패킷이 먼저 제공된다.  

<table>
  <thead>
    <tr>
      <td></td>
      <td>클래스 1 (낮은 우선순위)</td>
      <td>클래스 2</td>
      <td>클래스 3</td>
      <td>클래스 4 (높은 우선순위)</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>낮은 손실 비율</td>
      <td>AF11 (001010)</td>
      <td>AF21 (010010)</td>
      <td>AF31 (011010)</td>
      <td>AF41 (100010)</td>
    </tr>
    <tr>
      <td>중간 손실 비율</td>
      <td>AF12 (001100)</td>
      <td>AF22 (010100)</td>
      <td>AF32 (011100)</td>
      <td>AF42 (100110)</td>
    </tr>
    <tr>
      <td>높은 손실 비율</td>
      <td>AF13 (001110)</td>
      <td>AF23 (010110)</td>
      <td>AF33 (011110)</td>
      <td>AF43 (100110)</td>
    </tr>
  </tbody>
</table>
<br/><br/>

예를 들어, 다음 코드는 세 개의 소켓을 서로 다른 전달 특성을 갖도록 설정한다. 네트워크가 꽤 혼잡한 경우에도, 높은 손실 비율과 클래스 4에 해당하는 소켓 1은 거의 모든 데이터를 전송할 수 있다. 낮은 손실 비율과 클래스 1에 해당하는 소켓 2은 소켓 1만큼 빠르진 않겠지만 문제없이 데이터를 보낼 수 있다. 그리고 높은 손실 비율과 클래스 1에 해당하는 소켓 3은 소켓 2가 더 이상 손실되지 않을 정도로 혼잡이 개선될 때까지 완전히 차단된다.  

```java
Socket s1 = new Socket("www.example.com", 80);
s1.setTrafficClass(0x26); // 바이너리로 00100110
Socket s2 = new Socket("www.example.com", 80);
s1.setTrafficClass(0x0A); // 바이너리로 00100110
Socket s3 = new Socket("www.example.com", 80);
s1.setTrafficClass(0x0E); // 바이너리로 00100110
```

DSCP 값에 대한 서비스가 엄격하게 보장되지 않는다. 실제로 DSCP 값이 일부 네트워크에서 제대로 처리되기도 하지만, ISP를 통해 전달될 때 대부분 무시된다.  

이 옵션에 대한 자바 공식 문서는 심각하게 오래된 상태다. 그리고 이 문서는 네 가지 트래픽 클래스를 위한 비트 필드를 기반으로 QoS 기법을 설명하고 있다. 저렴한 비용, 높은 신뢰성, 최대 처리량, 그리고 최소 지연. 이 기법은 널리 구현되지 못했으며 앞으로도 마찬가지일 것이다. 이 값을 저장했던 TCP 헤더 부분은 여기서 설명된 DSCP 및 EN 값을 저장하기 위해 용도가 변경되었다. 이 값들을 설정하고자 할 경우 PHP의 상위 3비트에 저장하고 나머지 비트를 0으로 설정하면 된다.  

내부 소켓 구현들이 이러한 요청을 꼭 처리할 필요는 없다. 단지 우너하는 정책에 대해 TCP 스택에 제안을 할 뿐이다. 대부분의 구현들은 이 값들을 완전히 무시한다. 특히 안드로이드의 경우 setTrafficClass()는 아무런 일도 하지 않는다. 만약 TCP 스택이 요청된 서비스 클래스를 제공할 수 없는 경우 항상 그런 것은 아니지만 SocketException 예외를 발생시킬 수도 있다.  

우선순위를 표현하는 또 다른 방법으로 setPerformancePreference() 메소드를 사용하여 연결 시간, 지연 그리고 대역폭에 대해서 상대적인 우선순위를 할당할 수 있다.  

+ public void setPerformancePreference(int connectionTime, int latency, int bandwidth)  

예를 들어, connectionTime이 2이고, latency가 1이고, bandwidth가 3이면, 최대 대역폭이 가장 중요한 특성이고, 최소 지연 시간이 가장 덜 중요하고, 연결 시간이 중간이다. connectionTime이 2이고, latency가 2이고, bandwidth가 3이면, 최소 지연 시간과 연결 타임은 같은 수준으로 중요한 반면 최대 대역폭이 가장 중요하다. 특정 가상 머신에서 이 부분의 구현 방법은 구현에 따라 달라질 수 있으며, 사실 몇몇 구현에서는 아무런 동작도 하지 않는다.  

### 5. 소켓 예외
<br/>
Socket 클래스의 대부분 메소드는 IOException 예외 또는 서브클래스인 java.net.SocketException 예외를 발생시키도록 선언되어 있다.  

그러나 문제가 발생했다는 사실을 아는 것만으로는 문제를 처리하기에 충분하지 않다. 원격 호스트가 연결을 거부했다면 바빠서일까? 아니면 해당 포트에 대기 중인 서비스가 없어서일까? 연결 시도 시 타임아웃이 발생하면 네트워크가 혼잡해서일까? 아니면 호스트에 장애가 발생해서일까? 무엇이 잘못됐는지, 원인이 무엇인지 더 자세한 정보를 제공하는 SocketException의 몇몇 서브클래스가 제공된다.  

+ public class BindException extends SocketException
+ public class ConnectionException extends SocketException
+ public class NoRouterToHostException extends SocketException  

Socket 또는 ServerSocket 객체를 만들려고 할 때, 로컬 포트가 이미 사용 중이거나, 포트를 사용할 충분한 권한이 없는 경우 BindException 예외가 발생한다.  

원격 호스트에 대한 연결이 거부될 경우 ConnectException 예외가 발생하며, 일반적으로 해당 호스트가 바쁘거나 해당 포트에서 대기 중인 서비스가 없는 경우에 발생한다. 마지막으로, NoRouterToHostException 예외는 연결이 타임아웃되었음을 의미한다.  

java.net 패키지는 또한 IOException의 직접적인 서브클래스인 ProtocolException 예외를 제공한다.  

+ public class ProtocolException extends IOException  

이 예외는 네트워크로부터 TCP/IP 표준을 일부 위반한 데이터가 수신되었을 때 발생한다.  

여기서 소개한 예외 클래스들이 다른 예외 클래스에서 제공하지 않는 특별한 메소드를 제공하지는 않지만, 보다 자세한 에러 메시지를 제공하거나 잘못된 작업의 재시도 시 성공 가능성을 결정하기 위해 이 서브클래스들을 이용할 수 있다.  

### 6. GUI 애플리케이션 소켓
<br/>
자바 GUI 애플리케이션이면 데드락과 속도 저하를 피하기 위해 다음 두 가지 규칙을 따라야 하며, SwingWorker는 바로 이 문제를 해결해 준다.  

+ 스윙(Swing) 컴포넌트에 대한 모든 업데이트는 이벤트 디스패치 스레드에서 일어나야 한다.  

+ 느린 블로킹 연산, 특히 I/O 같은 연산이 이벤트 디스패치 스레드에서 일어나지 않도록 해야 한다. 이 외에도, 이벤트 디스패치 스레드 안에서 응답이 느린 서버에 요청할 경우 애플리케이션 전체가 멈출 수 있다.  

I/O를 수행하는 코드로 인해 GUI를 업데이트할 수 없거나 그 반대의 경우가 발생하기 때문에 이 두 작업은 서로 다른 스레드에서 처리해야 한다.  

이 역설적인 상황을 회피하는 다양한 방법이 있지만, 자바 6 이전 버전에서는 다소 복잡하다. 그러나 자바 6과 그 이후 버전에서는 쉽게 해결된다. SwingWorker의 서브클래스를 정의하고 다음 두 메소드를 오버라이드한다.  

(1) doInBackground() 메소드는 I/O 같은 긴 연산을 수행한다. 이 메소드는 GUI와 상호작용을 하지 않는다. 이 메소드는 어떤 타입이라도 반환할 수 있으며 어떤 예외라도 발생시킬 수 있다.  
(2) done() 메소드는 doInBackground() 메소드가 반환된 후에 이벤트 디스패치 스레드에서 자동으로 호출된다. 그래서 이 메소드는 GUI를 업데이트할 수 있다. 이 메소드는 doInBackground()에 의해 계산된 반환값을 가져오는 get() 메소드를 호출할 수 있다.
