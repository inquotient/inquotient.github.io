---
title:  서버 소켓
categories:
- Java Network Programming
feature_text: |
  ## 서버 소켓
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

### 1. ServerSocket 사용하기
<br/>
#### 1.1. 서버 소켓 닫기
<br/>
서버 소켓의 사용이 끝났으면 해당 소켓을 닫아야 한다. 특히 프로그램이 서버 소켓 사용 후에 바로 종료하지 않고 한동안 계속 실행되어야 할 경우 더 중요하다. 사용이 끝난 서버 소켓을 닫지 않으면 다른 프로그램이 해당 포트를 사용할 수 없다. ServerSocket을 닫는 것과 Socket을 닫는 것을 혼동하지 않도록 해야 한다. ServerSocket을 닫으면 사용 중인 로컬 호스트의 포트가 해제되며 다른 서버가 해당 포트를 바인드할 수 있게 된다. 또한 해당 ServerSocket을 통해 수용된 모든 소켓의 연결이 끊어진다.  

서버 소켓은 프로그램이 종료될 때 자동으로 닫힌다. 그래서 ServerSocket의 사용이 끝나고 곧바로 프로그램을 종료할 예정이라면 꼭 닫아야 할 필요는 없다. 하지만 종료한다고 문제가 되지는 않는다. 프로그래머들은 종종 try-finally 블록에서 "널이 아닌 경우 종료하는(close-if-not-null)" 패턴을 동일하게 따른다. 이미 이전 스트림과 클라이언트 측 소켓을 다룰 때 본적이 있을 것이다.  

```java
Serversocket server = null;
try {
  server = new ServerSocket(port);
  // ...서버 소켓을 이용한 작업
} finally {
  if (server != null) {
    try {
      server.close();
    } catch (IOException ex) {
      // 무시한다
    }
  }
}
```

이 코드를 예외가 발생하지 않고 실제 포트에 바인딩되지 않는 인자가 없는 ServerSocket() 생성자를 사용하여 약간 개선할 수 있다. 대신 ServerSocket() 객체가 생성된 이후에 서버 주소에 바인딩하기 위해 bind() 메소드를 따로 호출해야 한다.  

```java
ServerSocket server = new ServerSocket();
try {
  SocketAddress address = new InetSocketAddress(port);
  server.bind(address);
  // ... 서버 소켓을 이용한 작업
} finally {
  try {
    server.close();
  } catch (IOException ex) {
    // 무시한다
  }
}
```

자바 7에서 ServerSocket은 AutoCloseable 인터페이스를 구현하고 있으므로, try-with-resources 구문을 이용할 수 있다.  

```java
try (ServerSocket server = new ServerSocket(port)) {
  // ...서버 소켓을 이용한 작업
}
```

서버 소켓은 닫힌 후에 심지어 같은 포트라도 다시 연결할 수 없다. 즉, 한 번 닫힌 서버 소켓은 재사용할 수 없다. isClosed() 메소드는 ServerSocket이 닫힌 경우 true를 반환하고, 그렇지 않은 경우 false를 반환한다.  

+ public boolean isClosed()  

인자 없는 ServerSocket() 생성자로 만들어졌고 아직 어느 포트에도 바인딩되지 않은 ServerSocket 객체는 닫힌 것으로 간주되지 않는다. 이 객체에 대해 isClosed() 메소드를 호출하면 false가 반환된다. isBound() 메소드는 ServerSocket 객체가 포트에 바인딩되었는지 여부를 알려 준다.  

+ public boolean isBound()  

isBound() 메소드는 ServerSocket이 포트에 바인딩된 적이 있는 경우 현재 닫혀 있더라도 true를 반환한다. ServerSocket이 현재 열려 있는지 확인이 필요한 경우 isBound()의 반환값이 true이고 isClosed()의 반환값이 false인 두 가지 조건 모두를 확인해야 한다. 예를 들어:  

```java
public static boolean isOpen(ServerSocket ss) {
  return ss.isBound() && !ss.isClosed();
}
```

### 2. 로그 남기기
<br/>
서버는 오랜 기간 자동으로 실행되며, 그동안 서버에서 일어난 일들에 대해 확인이 필요한 경우가 있다. 이러한 이유로 최소한 일정 기간 동안의 서버 로그를 저장하는 것이 좋다.  

#### 2.1. 무엇을 로그로 남길 것인가
<br/>
일반적으로 다음 두 가지 기본적인 내용을 로그에 남긴다.  

+ 요청
+ 서버 에러  

사실 대부분의 서버는 두 로그 항목을 서로 다른 파일에 기록한다. 일반적으로 감사(audit) 로그의 경우 서버에 연결된 각 연결마다 하나의 로그를 남긴다. 하나의 연결로 다수의 명령을 수행하는 서버의 경우, 대신 명령마다 하나의 로그를 남기기도 한다. 예를 들어, dict 서버는 클라이언트가 검색하는 각 단어마다 하나의 로그를 남긴다.  

에러 로그는 주로 서버 운영 중에 발생하는 예측되지 않은 예외를 기록한다. 예를 들어, NullPointerException 같은 에러가 발생할 경우, 이 메시지는 곧 수정해야 할 서버의 버그를 나타내므로 에러 로그에 남겨야 한다. 에러 로그는 클라이언트가 갑자기 연결을 끊거나 잘못된 요청을 보내는 경우와 같은 클라이언트 에러는 포함하지 않으며, 일반적으로 이러한 로그는 요청 로그에 남긴다. 에러 로그는 예상치 못한 예외가 발생한 경우에만 사용한다.  

에러 로그는 항상 빈 상태로 유지하는 것이 가장 이상적이며, 기록된 모든 로그는 결국 확인하고 해결해야 할 버그를 나타낸다. 에러 로그의 확인 결과 버그가 아니며 의도된 동작으로 판단될 경우, 해당 로그를 남기는 코드를 제거해야 한다. 에러 로그에 잘못된 오류들이 차기 시작하면 더 이상 샆펴보지 않게 되며 에러 로그는 쓸모 없게 된다.  

같은 이유로, 개발 환경이 아닌 실제 서비스 환경에서 디버그 로그를 남기지 않도록 해야 한다. 메소드 진입 시마다 또는 조건이 일치할 때마다 로그를 남기지 않도록 해야 한다. 이러한 로그는 아무도 쳐다보지 않으며, 공간만 낭비할 뿐 실제 문제를 놓치게 된다. 디버깅을 위해 메소드 수준의 로그가 필요한 경우, 따로 분리된 파일에 기록하고 실제 서비스 환경에서는 설정 파일을 통해 끄도록 해야 한다.  

더욱 발전된 로그 시스템의 경우 특정 레벨의 로그만을 보여 주거나 특정 코드 영역의 로그만 보여 주는 로그 분석 툴을 제공한다. 이러한 툴을 이용하면 단일 로그 파일이나 데이터베이스를 이용하여 로그 파일을 관리하는 것이 가능하며, 심지어 하나의 로그를 다양한 바이너리나 프로그램이 공유하는 것도 가능해진다. 하지만 이러한 툴을 이용하는 경우에도 여전히 로그를 남기는 기본 원칙은 그대로 적용된다. 그러므로 아무도 보지 않거나 문제의 초점을 흐리고 혼란스럽게 하는 로그를 만들지 않도록 해야 한다.  

언젠가 필요할지 모른다는 만일의 경우를 대비하여 생각할 수 있는 모든 것을 로그로 남기는 일반적인 잘못된 패턴을 따르지 않도록 해야 한다. 실제로 프로그래머들은 서비스 환경에서 발생한 문제를 디버깅하는 데 어떤 로그를 남겨야 할지 잘 예측하지 못한다. 일단 문제가 발생하면 어떤 메시지가 필요한지 분명해지지만 이것을 미리 예측하기란 쉽지 않다. 만일의 경우를 대비하여 로그를 남기는 것은 문제가 발생했을 때, 아주 큰 의미 없는 데이터 속에서 의미 있는 데이터를 찾아 헤매겠다는 것을 의미한다.  

#### 2.2. 로그를 남기는 방법
<br/>
자바 1.3 이전에 개발된 많은 프로그램들은 여전히 log4j와 Apache Commons Logging 같은 서드파티 로그 라이브러리를 사용한다. 그러나 자바 1.4 이후부터는 대부분의 경우에 java.util.logging 패키지로 대체할 수 있다. 복잡한 서드파티 라이브러리에 대한 의존성을 피하기 위해 가능한 한 이 라이브러리를 이용하는 것이 좋다.  

Logger를 필요할 때 로드할 수도 있지만, 보통 아래와 같이 클래스당 하나의 객체를 생성해두는 편이 좋다.  

```java
private final static Logger auditLogger = Logger.getLogger("requests");
```

Logger는 스레드 환경에서 안전하기 때문에 Logger 객체를 공유된 정적 필드에 저장해도 문제가 되지 않는다. 사실은 Logger 객체가 스레드 사이에 공유되지 않더라도, 로그를 저장하는 파일이나 데이터베이스는 공유된다. 이 사실은 고도의 멀티스레드 서버에서 매우 중요하다.  

다수의 Logger 객체가 같은 로그에 출력할 수는 있지만, 각각의 Logger는 항상 정확히 하나의 로그에만 남긴다. 무슨 로그를 어디에 남길지는 외부 설정에 따라 달라진다. 일반적으로 파일에 기록하며 이 파일의 이름이 "reqeusts"일 수도 있고 아닐 수도 있다. 그 외에도 데이터베이스나 다른 서버에서 실행 중인 SOAP 서비스, 또는 같은 호스트나 다른 호스트에서 실행 중인 자바 프로그램일 수도 있다.  

Logger 객체를 만들고 나면, Logger가 제공하는 몇몇 메소드를 사용하여 로그를 쓸 수 있다. 가장 기본 메소드가 log()이다. 예를 들어, 다음 catch 블록은 예측되지 않은 런타임 예외에 대해 가장 높은 레벨의 로그를 남긴다.  

```java
catch (RuntimeException ex) {
  logger.log(Level.SEVERE, "unexpected error " + ex.getMessage(), ex);
}
```

catch 블록에서 로그를 남길 때, 메시지뿐만 아니라 예외를 함께 포함하는 것은 선택 사항이지만 일반적으로 예외도 함께 포함하여 로그를 남긴다.  

java.util.logging.Level에는 로그의 심각성에 따라 7개의 상수로 명명된 레벨이 내림차순으로 정의되어 있다.  

+ Level.SEVERE (가장 높은 값)
+ Level.WARNING
+ Level.INFO
+ Level.CONFIG
+ Level.FINE
+ Level.FINER
+ Level.FINEST (가장 낮은 값)  

낮은 레벨은 디버깅 목적으로만 사용해야 하며 서비스 중인 시스템에서는 사용하지 않도록 해야 한다. 정보, 심각, 경고를 포함한 각각의 레벨에 대한 로그를 편리하게 남길 수 있는 메소드가 제공된다. 예를 들어, 다음 코드는 날짜와 원격 호스트의 주소를 포함한 로그를 남긴다.  

```java
logger.info(new Date() + " " + connection.getRemoteSocketAddress());
```

각 로그 레코드는 편리한 어떤 형식이라도 사용할 수 있다. 일반적으로 각 레코드는 시간, 클라이언트 주소, 처리된 요청에 대한 정보를 포함한다. 로그 메시지가 에러를 나타낼 경우, 발생한 예외를 나타내는 정보를 포함한다. 자바는 메시지가 남겨진 코드의 위치를 자동으로 채우므로 이 부분에 대해서 여러분은 신경 쓰지 않아도 된다.  

java.util.logging.config.file 시스템 속성은 로그를 제어하는 로그 설정 파일을 가리킨다. 가상 머신을 실행할 때 "-Djava.util.logging.config.file=파일 이름"을 인자로 전달하여 이 속성을 설정할 수 있다. 예를 들어, 맥 OS X에서는 이 값이 Info.plist 파일의 VMOptions에서 설정된다.  

```xml
<key>Java</key>
<dict>
    <key>VMOptions</key>
    <array>
      <string>Djava.util.logging.config.file=/opt/daytime/logging.properties</string>
    </array>
</dict>
```

다음 에제는 다음 사항을 포함하고 있는 로그 설정 파일의 샘플이다.  

+ 로그는 파일에 기록한다.
+ 요청 로그는 기본 레벨이 INFO이며, /var/logs/daytime/requests.log에 기록된다.  
+ 에러 로그는 기본 레벨이 SEVERE이며, /var/logs/daytime/requests.log에 기록된다.  
+ 로그 파일의 크기는 10메가바이트로 제한되며, 가득 차면 다른 파일로 교체(rotate)된다.  
+ 현재 로그 파일과 이전 로그 파일, 두 개의 로그 파일을 보관한다.  
+ 기본 텍스트 형식을 사용한다(XML 아님).
+ 로그 파일의 각 라인은 레벨 메세지 타임스탬프(level message timestamp) 형식으로 구성된다.  

```
handle=java.util.logging.FileHandler
java.util.logging.FileHandler.pattern = /var/logs/daytime/requests.log
java.util.logging.FileHandler.limit = 10000000
java.util.logging.FileHandler.count = 2
java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter
java.util.logging.SimpleFormatter.format=%4$s: %5$s [%1$tc]%n

requests.level = INFO
audit.level = SEVERE
```

### 3. 서버 소켓 만들기
<br/>
public으로 선언된 네 개의 ServerSocket 생성자가 있다.  

+ public ServerSocket(int port) throws BindException, IOException
+ public ServerSocket(int port, int queueLength) throws BindException, IOException
+ public ServerSocket(int port, int queueLength, InetAddress bindAddress) throws BindException, IOException
+ public ServerSocket() throws BindException, IOException  

이 생성자들은 포트와 들어오는 연결 요청을 보관하는 큐의 길이, 그리고 바인딩할 로컬 네트워크 인터페이스를 명시한다. 일부 생성자는 기본 큐의 길이와 바인딩할 주소를 사용하긴 하지만, 기본적으로 모두 같은 기능을 한다.  

예를 들어, 포트 80에서 HTTP 서버에 의해 사용될 서버 소켓을 만들 경우 다음과 같이 작성할 수 있다.  

```java
ServerSocket httpd = new ServerSocket(80);
```

포트 80을 사용하고 동시에 수용되지 않은 연결을 최대 50개까지 보관할 수 있는 큐를 가진 서버 소켓을 만들 경우 다음과 같이 작성할 수 있다.  

```java
ServerSocket httpd = new ServerSocket(80, 50);
```

큐의 크기로 운영체제가 허용하는 것보다 큰 값을 전달한 경우 전달된 값은 무시되고 운영체제의 최대값이 사용된다.  

다수의 네트워크 인터페이스와 IP 주소를 가진 호스트의 경우, 기본적으로 서버 소켓은 모든 인터페이스와 IP 주소의 해당 포트에 대해 대기한다. 그러나 세 번째 인자로 특정 로컬 IP 주소를 추가하여 해당 주소에 대해서만 바인딩할 수 있다. 즉, 지정된 주소로 들어오는 연결에 대해서만 서버 소켓이 대기하게 된다. 호스트의 다른 주소를 통해 들어오는 연결에 대해서는 대기하지 않는다.  

예를 들어, login.ibiblio.org 리눅스 서버는 노스캐롤라이나(North Carolina)에 있다. 이 서버는 IP 주소 152.2.210.122 인터넷에 연결되어 있다. 그리고 이 서버의 두 번째 이더넷 카드의 IP 주소는 192.168.210.122이고, 이 주소는 외부에서 보이지 않으며 로컬 네트워크에서만 보인다. 만약 여러분이 이 호스트에 같은 네트워크에서 오는 로컬 연결에 대해서만 응답하는 서버를 실행시키고자 한다면, 아래와 같이 152.2.210.122의 5776은 제외하고, 192.168.210.122의 5776 포트에서만 대기하는 서버 소켓을 생성할 수 있다.  

```java
InetAddress local = InetAddress.getByName("192.168.210.122");
ServerSocket httpd = new ServerSocket(5776, 10, local);
```

포트로 인자로 전달받는 세 개의 생성자에서, 포트 번호로 0을 전달할 경우 시스템은 사용할 수 있는 포트를 임의로 선택한다. 시스템에 의해 포트가 선택될 경우 해당 포트를 미리 알 수 없기 때문에 때로 익명 포트(anonymous port)라고 부른다. (하지만 포트가 선택된 이후에는 해당 포트를 알 수 있다.) 익명 포트는 FTP와 같은 다수의 소켓을 사용하는 프로토콜에서 종종 유용하게 사용된다. FTP 수동(passive) 모드에서 클라이언트는 먼저 서버의 21번 포트로 연결한다. 그리고 파일 전송이 필요한 경우 서버는 임의의 포트를 바인딩하여 대기한다. 서버는 이미 연결된 21번 포트를 통해 데이터 전송 포트를 클라이언트에게 알려 준다. 그리고 데이터 포트는 세션마다 변경될 수 있으며 미리 알 필요가 없다. [능동(active) FTP 모드의 경우, 데이터 전송을 위해 클라이언트가 임의의 포트를 열고 서버의 연결을 대기한다는 것 이외에는 큰 차이가 없다.]  

이 세 개의 생성자는 IOException 예외를 발생시키며, 특히 소켓을 생성할 수 없거나 요청된 포트로 바인딩할 수 없는 경우 BindException 예외가 발생한다. ServerSocket을 생성할 때 IOException이 발생하는 경우는 대부분 다음 두 가지 중 하나다. 요청된 포트를 다른 서버 소켓(다른 프로그램일 수도 있다)이 이미 사용 중이거나, 유닉스(리눅스와 맥 OS X 포함) 환경에서 루트 권한 없이 1에서 1023 사이의 포트에 바인딩을 시도한 경우다.  

#### 3.1. 바인딩 없이 서버 소켓 만들기
<br/>
인자가 없는 ServerSocket 생성자는 실제 포트에 바인딩되지 않는 ServerSocket 객체를 생성한다. 따라서 이 생성자를 통해 생성된 ServerSocket 객체는 처음에는 어떠한 연결도 수용할 수 없다. 이 객체는 생성 이후에 bind() 메소드를 사용하여 따로 바인딩해야 한다.  

+ public void bind(SocketAddress endpoint) throws IOException
+ public void bind(SocketAddress endpoint, int queueLength) throws IOException  

이러한 기능은 포트에 바인딩하기 전에 서버 소켓 옵션을 설정하는 프로그램에서 주로 사용된다. 몇몇 옵션의 경우 서버 소켓이 바인딩된 다음에는 변경할 수 없다. 일반적인 사용 패턴은 다음과 같다.  

```java
ServerSocket ss = new ServerSocket();
// 소켓 옵션을 설정한다...
SocketAddress http = new InetSocketAddress(80);
ss.bind(http);
```

SocketAddress의 인자로 널(null)을 전달하여 임의의 포트를 선택할 수도 있다. 이는 ServerSocket() 생성자의 포트 번호에 0을 전달하는 것과 같다.  

### 4. 서버 소켓 정보 가져오기
<br/>
ServerSocket은 서버 소켓에 의해 사용 중인 로컬 주소와 포트를 알려 주는 두 가지 get 메소드를 제공한다. 이 두 메소드는 네트워크 인터페이스를 명시하지 않았거나 익명 포트를 사용하여 서버 소켓을 생성한 경우 유용하게 사용된다. 예를 하나 들면, FTP 세션의 데이터 연결이 이 경우에 해당한다.  

+ public InetAddress getInetAddress()  

이 메소드는 (로컬 호스트에서 실행 중인) 서버에 의해 사용 중인 주소를 반환한다. 로컬 호스트가 단일 IP 주소를 가지고 경우, 이 메소드의 반환값은 InetAddress.getLocalHost() 메소드의 반환값과 같다. 로컬 호스트가 하나 이상의 IP 주소를 가지고 있는 경우, 호스트의 IP 주소 중 하나가 반환되며 어떤 주소가 반환될지는 예측할 수 없다. 예를 들어, 다음과 같이 사용한다.  

```java
ServerSocket httpd = new ServerSocket(80);
InetAddress ia = httpd.getInetAddress();
```

ServerSocket이 네트워크 인터페이스에 아직 연결되지 않은 경우, getInetAddress()는 널(null)을 반환한다.  

+ public int getLocalPort()  

ServerSocket 생성자의 포트 번호로 0번을 전달하면 익명의 포트 번호로 대기하게 된다. 이때 이 메소드를 사용하면 실제 대기 중인 포트를 확인할 수 있다. 그리고 이 메소드는 여러분의 위치를 다른 동료에게 알리기 위한 수단으로 P2P 멀티 소켓 프로그램에서 사용할 수 있다. 또는 서버가 특정 연산을 수행하기 위한 몇몇 하위 서버들을 실행한 경우, 상위 서버가 클라이언트에게 몇 번 포트를 통해 하위 서버들을 찾을 수 있는지 알려 주는 경우를 생각해 볼 수 있다. 물론 익명 포트를 사용하지 않을 때도 getLocalPort() 메소드를 사용할 수 있지만 필요한 경우는 거의 없다.  

최소한 이 시스템에서는 포트 번호가 정확하게 랜덤하지는 않다. 그러나 실행해 보기 전까지는 여전히 알 수가 없다.  

ServerSocket이 네트워크 인터페이스에 아직 연결되지 않은 경우, getInetAddress()는 널(null)을 반환한다.  

+ public int getLocalPort()  

ServerSocket 생성자의 포트 번호로 0번을 전달하면 익명의 포트 번호로 대기하게 된다. 이때 이 메소드를 사용하면 실제 대기 중인 포트를 확인할 수 있다. 그리고 이 메소드는 여러분의 위치를 다른 동료에게 알리기 위한 수단으로 P2P 멀티 소켓 프로그램에서 사용할 수 있다. 또는 서버가 특정 연산을 수행하기 위한 몇몇 하위 서버들을 실행할 경우, 상위 서버가 클라이언트에게 몇 번 포트를 통해 하위 서버들을 찾을 수 있는지 알려 주는 경우를 생각해 볼 수 있다. 물론 익명 포트를 사용하지 않을 때도 getLocalPort() 메소드를 사용할 수 있지만 필요한 경우는 거의 없다.  

대부분의 자바 객체들처럼, toString() 메소드를 사용하여 ServerSocket 객체를 출력할 수 있다. ServerSocket 클래스의 toString() 메소드에 의해 반환된 문자열은 다음과 같다.  

addr은 서버 소켓이 연결된 로컬 네트워크 인터페이스의 주소를 나타낸다. 일반적으로 모든 인터페이스에 연결된 경우 이 값은 0.0.0.0이다. 포트는 항상 0이다. localport는 서버가 연결을 대기하고 있는 로컬 포트다. 이 메소드는 때로 디버깅에 유용하게 사용된다. 그러나 그 이상은 아니다. 이 메소드에 의존하지 않도록 하자.  

### 5. 소켓 옵션
<br/>
Socket 옵션은 ServerSocket 클래스가 의존하는 네이티브 소켓이 데이터를 주고받는 방법을 명시한다. 서버 소켓을 위해 자바는 다음 세 개의 옵션을 제공한다.  

+ SO&#95;TIMEOUT
+ SO&#95;REUSEADDR
+ SO&#95;RCVBUF  

또한 이 옵션들을 통해 네트워크 입출력의 성능을 높일 수가 있다.  

#### 5.1. SO&#95;TIMEOUT
<br/>
SO&#95;TIMEOUT은 accept() 메소드가 들어오는 연결을 대기하는 밀리초 단위의 시간의 양이다. 이 시간이 지나면 java.io.IOException 예외가 발생한다. SO&#95;TIMEOUT이 0인 경우, accept() 메소드는 타임아웃 없이 무한히 대기한다. SO&#95;TIMEOUT의 기본값은 0이다.  

일반적으로 SO&#95;TIMEOUT의 설정이 필요한 경우는 드물다. 클라리언트와 서버 사이에 다수의 연결이 필요하고 일정 시간 이내에 응답이 이뤄져야 하는 복잡한 보안 프로토콜을 구현할 때 필요할 수도 있다. 그러나 대부분의 서버들은 무한정 클라리언트의 연결을 기다리도록 설계되어 있다. 그래서 대부분의 서버들은 기본 타임아웃 값인 0을 사용한다. 이 값을 변경하고자 할 경우, 해당 서버 소켓의 setSoTimeout()를 호출하여 SO&#95;TIMEOUT 필드의 값을 설정할 수 있다.  

+ public void setSoTimeout(int timeout) throws SocketException
+ public int getSoTimeout() throws IOException  

초읽기는 accept() 메소드가 호출되는 시점부터 시작된다. 그리고 타임아웃이 발생하면 accept() 메소드는 IOException의 서브클래스인 SocketTimeoutException 예외를 발생시킨다. 이 옵션은 accept() 메소드를 호출하기 전에 설정해야 한다. accept() 메소드가 연결을 대기하고 있는 동안에는 타임아웃 값을 변경할 수 없다. timeout 인자는 0보다 크거나 같아야 한다. 그렇지 않은 경우 IllegalArgumentException 예외가 발생한다. 예를 들어:  

```java
try (ServerSocket server = new ServerSocket(port)) {
  server.setSoTimeout(30000); // 30초의 볼록 설정
  try {
    Socket s = server.accept();
    // 연결된 소켓으로 작업
    // ...
  } catch (SocketTimeoutException ex) {
    System.err.println("No connection within 30 seconds");
  }
} catch (IOException ex) {
  System.err.println("Unexpected IOException: " + e);
}
```

getSoTimeout() 메소드는 서버 소켓의 현재 SO&#95;TIMEOUT 값을 반환한다. 예를 들어:  

#### 5.2. SO&#95;REUSEADDR
<br/>
서버 소켓의 SO&#95;REUSEADDR 옵션은 새로운 소켓이 바인딩하려는 포트에 대해 이전에 바인딩된 소켓으로 전송 중인 데이터가 있는 경우에도 해당 포트를 바인딩할 수 있는지 여부를 결정한다. 예상할 수 있듯이 이 옵션을 설정하고 확인하는 두 가지 메소드가 제공된다.  

+ public boolean getReuseAddress() throws SocketException
+ public void setReuseAddress(boolean on) throws SocketException  

기본값은 플랫폼에 따라 달라질 수 있다. 다음 코드는 새로운 ServerSocket을 만들고 getReuseAddress() 메소드를 호출하여 기본값을 확인한다.  

```java
ServerSocket ss = new ServerSocket(10240);
System.out.println("Reusable: " + ss.getReuseAddress());
```

#### 5.3. SO&#95;RCVBUF
<br/>
SO&#95;RCVBUF 옵션은 서버 소켓에 의해 수용(accept)된 클라이언트 소켓의 기본 수신 버퍼의 크기를 설정한다. 이 값은 다음 두 메소드를 사용하여 설정하거나 확인할 수 있다.  

+ public int getReceiveBufferSize() throws SocketException
+ public void setReceiveBufferSize(int size) throws SocketException  

서버 소켓에 SO&#95;RCVBUF을 설정하는 것은 accept()에 의해 반환된 개별 소켓의 setReceiveBufferSize()를 호출하는 것과 같다(서버 소켓이 수용된 이후에는 수신 버퍼의 크기를 변경할 수 없다는 것은 제외), 이 옵션은 스트림에서 개별 IP 패킷의 크기를 제안한다. 비록 대부분의 경우 기본값으로도 충분하지만, 빠른 연결의 경우 더 많은 버퍼를 필요로 할 것이다.  

이 옵션은 수신 버퍼의 크기를 64K 이상으로 설정하는 경우가 아니라면, 서버 소켓이 포트에 바운드되기 전이나 후 언제든지 설정할 수 있다. 64K 이상을 설정할 경우, 바운드되지 않은 ServerSocket에 대해 설정해야 한다. 예를 들어:  

```java
ServerSocket ss = new ServerSocket();
int receiveBufferSize = ss.getReceiveBufferSize();
if (receiveBufferSize < 131072) {
  ss.setReceiveBufferSize(131072);
}
ss.bind(new InetSocketAddress(8000));
```

#### 5.4. 서비스 클래스
<br/>
인터넷 서비스의 종류마다 서로 다른 네트워크 성능이 요구된다. 예를 들어, 스포츠 비디오의 실시간 스트리밍은 상대적으로 높은 대역폭이 필요하다. 반면에 영화는 여전히 높은 대역폭이 필요하지만, 실시간 스트리밍에 비해 오는 정도의 지연(delay)과 대기 시간(latency)이 발생해도 서비스에 문제가 되지 않는다. 이메일은 매우 낮은 대역폭에서도 전송되며, 심지어 몇 시간 늦게 전송되더라도 큰 문제가 되지 않는다.  

TCP에서는 네 개의 일반적인 트래픽 클래스를 정의한다.  

+ 저렴한 비용(Low cost)
+ 높은 신뢰성(High reliability)
+ 최대 처리량(Maximum throughput)
+ 최소 지연(Minimum delay)  

이러한 트래픽 클래스를 소켓에 요청할 수 있다. 예를 들어, 저렴한 비용으로 이용 가능한 최소한의 지연을 요청할 수 있다. 이러한 요청은 모두 명확하지 않으며 상대적이고, 정확한 서비스가 제공된다는 보장은 없다. 그리고 모든 라우터와 네이티브 TCP 스택이 이러한 클래스를 지원하는 것은 아니다.  

setPerformancePreference() 메소드는 이 서버에 수용된 소켓을 위한 연결 시간, 지연, 대역폭에 대한 상대적인 우선순위를 표현한다.  

+ public void setPerformancePreference(int connectionTime, int latency, int bandwidth)  

예를 들어, connectionTime을 2로, latency을 1로, 그리고 bandwidth을 3으로 설정하여 최대 대역폭이 가장 중요하고, 최소 지연이 가장 덜 중요하고, 연결 시간이 중간임을 나타낸다.  

```java
ss.setPerformancePreference(2, 1, 3);
```

사용 중인 가상 머신에서 이 부분을 정확히 어떻게 구현했는지는 구현에 따라 다를 수 있다. 내부 소켓 구현이 이러한 요청을 곡 준수해야 하는 것은 아니다. 이 메소드 호출은 단지 필요한 정책에 대해서 제안을 할 뿐이다. 안드로이드를 포함한 많은 구현에서는 이러한 값들을 완전히 무시한다.
