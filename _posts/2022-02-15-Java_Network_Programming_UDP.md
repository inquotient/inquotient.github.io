---
title:  UDP
categories:
- Java Network Programming
feature_text: |
  ## UDP
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

### 1. UDP 프로토콜
<br/>
자바의 UDP 구현은 두 클래스로 나뉜다. DatagramPacket 클래스는 데이터의 바이트를 데이터그램이라고 불리는 UDP 패킷에 채워 넣고 수신 받은 데이터그램을 꺼낸다. DatagramSocket은 UDP 데이터그램을 수신할뿐만 아니라 전송도 한다. 데이터를 전송하기 위해서는 데이터를 자바의 UDP 구현은 두 클래스로 나뉜다. DatagramPacket 클래스는 데이터의 바이트를 데이터그램이라고 불리는 UDP 패킷에 채워 넣고 수신 받은 데이터그램을 꺼낸다. DatagramSocket은 UDP 데이터그램을 수신할뿐만 아니라 전송도 한다. 데이터를 전송하기 위해서는 데이터를 DatagramPacket에 넣고 DatagramSocket에서 DatagramPacket 객체를 가져와서 패킷의 내용을 확인한다. 소켓 자체는 매우 간단하다. UDP에서 전송될 주소를 포함한 데이터그램에 관한 모든 것이 패킷 자체에 저장되어 있다. 소켓은 대기하거나 전송할 로컬 포트만 알고 있으면 된다.  

이러한 작업의 구분은 TCP에서 사용되는 Socket 그리고 ServerSocket 클래스와는 대조적이다. 먼저 UDP는 두 호스트 사이의 고유의 연결이라는 개념이 없다. 하나의 소켓이 하나의 포트에 대해 들어오거나 보내는 모든 데이터를 수신하거나 보낸다. 그리고 원격 호스트가 누구인지에 대한 구분도 없다. 단일 DatagramSocket은 많은 독립적인 호스트로부터 데이터를 보내거나 받는다. UDP 소켓은 TCP처럼 단일 연결을 위해 할당되지 않는다. 사실 UDP는 두 호스트 사이의 연결이라는 개념 자체가 없다. UDP는 개별 데이터그램에 대해서만 알고 있다. 누가 무슨 데이터를 보냈는지 알아내는 것은 전적으로 애플리케이션의 책임이다. 두 번째, TCP 소켓은 네트워크 연결을 스트림처럼 다룬다. 소켓에서 가져온 입출력 스트림을 사용하여 데이터를 보내거나 받는다. UDP는 이러한 방식을 지원하지 않는다. 여러분은 항상 개별 데이터그램 패킷으로 작업해야 한다. 단일 데이터그램에 채워 넣는 모든 데이터는 단일 패킷으로 전송되고, 이 데이터그램은 전송되거나 덩이리째 유실된다. 하나의 패킷은 다음 패킷과 아무런 연관이 없다. 두 패킷이 있는 경우 어느 패킷을 먼저 보내야 할지 결정할 만한 아무런 근거가 없다. 데이터가 순서대로 정렬된 큐가 스트림의 경우에는 필요하지만, 대신 데이터그램은 각자의 방식으로 버스에 올라타는 수많은 군중들처럼, 가능한 한 빨리 수신자에게 전송하기 위한 시도만을 한다.  

### 2. UDP 클라이언트
<br/>
```java
import java.io.*;
import java.net.*;

public class DaytimeUDPClient {

  private final static int PORT = 13;
  private static final String HOSTNAME = "time.nist.gov";

  public static void main(String[] args) {
    try (DatagramSocket socket = new DatagramSocket(0)) {
      socket.setSoTimeout(10000);
      InetAddress host = InetAddress.getByName(HOSTNAME);
      DatagramPacket request = new DatagramPacket(new byte[1], 1, host, PORT);
      DatagramPacket response = new DatagramPacket(new byte[1024], 1024);
      socket.send(request);
      socket.receive(response);
      String result = new String(response.getData(), 0, response.getLength(), "US-ASCII");
      System.out.println(result);
    } catch (IOException ex) {
      ex.printStackTrace();
    }
  }
}
```

### 3. UDP 서버
<br/>
```java
import java.net.*;
import java.util.Date;
import java.util.logging.*;
import java.io.*;

public class DaytimeUDPServer {

  private final static int PORT = 13;
  private final static Logger audit = Logger.getLogger("requests");
  private final static Logger errors = Logger.getLogger("errors");

  public static void main(String[] args) {
    try {DatagramSocket socket = new DatagramSocket(PORT)} {
      while (true) {
        try {
          DatagramPacket request = new DatagramPacket(new byte[1024], 1024);
          socket.receive(request);

          String daytime = new Date().toString();
          byte[] data = daytime.getBytes("US-ASCII");
          DatagramPacket response = new DatagramPacket(data, data.length, request.getAddress(), request.getPort());
          socket.send(response);
          audit.info(daytime + " " + request.getAddress());
        } catch (IOException | RuntimeException ex) {
          errors.log(Level.SEVERE, ex.getMessage(), ex);
        }
      }
    } catch (IOException ex) {
      errors.log(Level.SEVERE, ex.getMessage(), ex);
    }
  }
}
```

### 4. DatagramPacket 클래스
<br/>
UDP 데이터그램을 만들기 위해 IP 데이터그램에 추가하는 것이 많지 않다. UDP 해더는 8바이트밖에 추가하지 않는다. UDP 헤도는 출발지와 목적지의 포트 번호, IP 헤더 뒤에 따라오는 모든 데이터의 길이, 그리고 선택적인 체크섬이 포함된다. 포트 번호는 두 바이트 부호 없는 정소루 제공되기 때문에 호스트당 6만 5,536개의 서로 다른 UDP 포트를 사용할 수 있다. 이 포트는 TCP가 사용하는 6만 5,536개의 서로 다른 UDP 포트를 사용할 수 있다. 이 포트는 TCP가 사용하는 6만 5,536개의 서로 다른 UDP 포트를 사용할 수 있다. 이 포트는 TCP가 사용하는 6만 5,536포트와는 다른 포트이다. 길이 또한 2-바이트 부호 없는 정수이기 때문에 데이터그램의 길이는 6만 5,536에서 헤더 길이 8바이트를 뺀 것으로 제한된다. 그러나 이 부분은 IP 헤더의 데이터그램 길이 필드와 중복되며, IP 헤더에 있는 데이터그램 길이 필드는 6만 5,467에서 6만 5,507로 데이터그램을 제한한다. (정확한 숫자는 IP 헤더의 길이에 따라 다르다.) 체크섬 필드는 선택 사항이지만, 애플리케이션 계층 프로그램에서 사용하거나 접근할 수 없다. 데이터에 대한 체크섬이 실패할 경우 네이티브 네트워크 소프트웨어는 조용히 데이터그램을 버린다. 송신자나 수신자 모두 알림을 받지 않는다. UDP는 결국 신뢰할 수 없는 프로토콜이다.  

UDP 데이터그램이 담을 수 있는 데이터의 최대량이 이론적으로 6만 5,507바이트이지만, 실제는 이것보다 훨씬 적은 크기가 사용된다. 많은 플랫폼에서 실제 8,192(8K)바이트로 제한된다. 그리고 실제 구현들은 데이터와 헤더를 포함하여 576바이트를 초과하는 데이터그램을 받도록 요구되지 않는다. 따라서 8K 이상의 데이터를 가진 UDP 패킷을 주고받는 임의의 프로그램은 조심스럽게 다뤄야 한다. 대부분의 경우, 큰 패킷은 단순히 8K의 데이터로 잘라 비린다. 이러한 제한은 큰 패킷에 비해 성능 측면에서 마이너스 요소지만, 안전을 위해, UDP 패킷의 데이터 부분은 512바이트보다 작게 유지해야 한다. (이 부분은 TCP 데이터그램에도 마찬가지로 문제가 된다. 그러나 Socket과 ServerSocket이 제공하는 스트림 기반의 API는 이러한 문제를 프로그래머가 신경 쓰지 않도록 해 준다.)  

자바에서 UDP 데이터그램은 DatagramPacket 클래스의 인스턴스로 표현된다.  

+ public final class DatagramPacket extends Object  

이 클래스는 IP 헤더로부터 출발지와 목적지 주소를 가져오거나 설정하는 메소드, 출발지와 목적지 포트를 가져오거나 설정하는 메소드, 데이터를 설정하거나 가져오는 메소드, 그리고 데이터의 길이를 설정하는 메소드를 제공한다. 이 외에 나머지 헤더 필드는 순수 자바 코드로는 접근할 수 없다.  

#### 4.1. 생성자
<br/>
DatagramPacket은 패킷을 사용하는 목적에 따라 다른 생성자를 사용한다. 즉, 보낼 때와 받을 때 다른 생성자를 사용한다. 그동안 보았던 생성자와는 조금 차이가 있다. 일반적으로 생성자는 객체를 생성할 때, 다른 종류의 정보를 제공하기 위해 오버로드되지만, 이 경우는 다른 용도로 사용될 객체를 만들기 위해 사용된다. DatagramPacket 클래스의 6개의 모든 생성자는 공통적으로 데이터그램 데이터를 보관할 바이트 배열과 바이트 배열에 저장된 데이터의 길이를 나타내는 정수를 인자로 받는다. 데이터그램을 수신할 경우에는, 이 두 인자만 제공하면 된다. 소켓은 네트워크로부터 데이터그램을 받을 때, DatagramPacket 객체의 배열 버퍼에 최대 지정된 길이만큼 데이터그램의 데이터를 보관한다.  

DatagramPacket의 또 다른 종류의 생성자들은 네트워크를 통해 전송할 데이터그램을 만들기 위해 사용된다. 이 생성자들 역시 배열 버퍼와 길이를 인자로 받으며 추가적으로 패킷이 보내질 호스트의 주소와 포트를 인자로 받는다. 이 생성자에 대해서는 보낼 데이터를 포함하고 있는 바이트 배열과 목적지 주소와 포트를 생성자의 인자로 전달한다. DatagramSocket의 경우 TCP와는 달리 목적지 주소아 포트가 패킷에 저장되어 있다. (TCP에서는 소켓에 목적지 주소와 포트가 저장되어 있다.)  

##### 4.1.1. 수신 데이터그램을 위한 생성자
<br/>
다음 두 개의 생성자는 네트워크로 데이터를 수신하기 위한 DatagramPacket 객체를 생성한다.  

+ public DatagramPacket(byte[] buffer, int length)
+ public DatagramPacket(byte[] buffer, int offset, int length)  

첫 번째 생성자가 사용될 경우, 소켓이 데이터그램을 수신할 때 소켓은 데이터그램의 데이터 부분을 버퍼의 시작 위치인 buffer[0]에서부터 저장하기 시작하고, 패킷이 완료되거나 버퍼 안에 length 바이트만큼 쓰일 때까지 저장한다. 예를 들어, 다음 코드는 최대 8,192바이트까지의 데이터그램을 수신하는 DatagramPacket 객체를 생성한다.  

```java
byte[] buffer = new byte[8192];
DatagramPacket dp = new DatagramPacket(buffer, buffer.length);
```

두 번째 생성자를 사용할 경우, 대신 buffer[offset]부터 저장하기 시작한다. 그 외에 이 두 생성자는 동일한 기능을 한다. 두 번째 생성자에서 length는 buffer.length에서 offset을 뺀 것보다 작거나 같아야 한다. DatagramPacket 객체 생성 시 버퍼의 크기를 초과하는 길이를 인자로 전달할 경우 IllegalArgumentException 예외가 발생한다. 이 예외는 RuntimeException이므로 이 예외를 처리하는 코드를 작성할 필요는 없다. buffer.length에서 offset을 뺀 것보다 작은 크기로 DatagramPacket 객체를 생성하는 것은 문제가 되지 않는다. 이 경우에 데이터그램이 수신되면 버퍼의 처음 위치부터 길이만큼 채워진다.  

생성자는 버퍼가 얼마나 큰지 신경 쓰지 않으며 메가바이트 단위의 데이터를 가진 DatagramPacket 객체를 만들 수도 있다. 그러나 내부의 네이티브 네트워크 소프트웨어는 그렇게 큰 데이터를 지원할 만큼 친절하지 않다. 그리고 대부분의 네이티브 UDP 구현은 데이터그램당 8,192바이트보다 큰 데이터를 지원하지 않는다. 이론적으로 IPv4 데이터그램에서는 6만 5,536바이트이다. 그러나 실제로, DNS와 TFTP 같은 많은 UDP 기반의 프로토콜들은 데이터그램당 512바이트 또는 그 이하의 패킷을 사용한다. 일반적으로 사용되는 가장 큰 데이터의 크기는 NFS에서 사용되는 8,192바이트이다. 여러분이 볼 수 있는 거의 모든 UDP 데이터그램은 8K보다 작은 데이터를 사용한다. 사실 많은 운영체제들은 8K 이상의 데이터를 가진 UDP 데이터그램을 지원하지 않으며 이보다 클 경우 패킷을 자르거나 버린다. 데이터그램이 너무 커서 잘리거나 버려질 경우, 여러분의 자바 프로그램은 이 내용에 대해서 아무런 통보도 받지 못한다. 따라서 8,192바이트 이상의 데이터를 가진 DatagramPacket 객체를 생성하지 않도록 해야 한다.  

##### 4.1.2. 송신 데이터그램을 위한 생성자
<br/>
다음 네 개의 생성자는 네트워크를 통해 데이터를 보낼 때 사용되는 DatagramPacket 객체를 생성한다.  

+ public DatagramPacket(byte[] data, int length, InetAddress destination, int port)
+ public DatagramPacket(byte[] data, int offset, int length, InetAddress destination, int port)
+ public DatagramPacket(byte[] data, int length, SocketAddress destination)
+ public DatagramPacket(byte[] data, int offset, int length, SocketAddress destination)  

각각의 생성자는 다른 호스트로 전송될 새로운 DatagramPacket을 생성한다. 패킷은 data 배열의 0 또는 offset 위치에서부터 length 바이트만큼의 데이터로 채워진다. data.length보다 큰 길이 값을(또는 data.length - offset 보다 큰) 인자로 DatagramPacket을 생성하려고 할 경우, 생성자는 IllegalArgumentException 예외를 발생시킨다. 그러나 인자로 전달된 데이터 배열의 크기보다 작은 length를 전달하는 것은 문제가 되지 않는다. 이 경우에 length만큼의 데이터만 네트워크로 전송된다. InetAddress와 SocketAddress 객체는 패킷을 전송할 호스트를 가리키며 정수 인자 port는 전송할 호스트의 포트 번호이다.  

+ 데이터그램 크기 선택하기  
하니의 패킷에 얼마만큼의 데이터를 채워야 할지는 상황에 따라 다르다. 일부 프로토콜에서는 패킷의 크기를 명시한다. 예를 들어, rlogin은 사용자가 입력한 각 문자들을 입력 즉시 원격 시스템으로 전송한다. 그 결과 패킷은 입력된 단일 바이트와 추가적인 헤더의 몇 바이트로 아주 작다. 다른 애플리케이션의 경우 좀 더 단순하다. 예를 들어, 파일 전송은 큰 버퍼를 사용할수록 효과적이며, 유일한 요구 사항은 허용되는 최대 패킷 크기를 초과하지 않도록 파일을 패킷으로 분할하는 것뿐이다.  

여러 가지 요인들이 최적화된 패킷 크기를 결정하는 데 영향을 미친다. 라디오 네트워크와 같은 고도로 신뢰할 수 없는 네트워크의 경우, 전송 중에 손상될 가능성이 적은 작은 크기의 패킷이 더 선호된다. 반면에 매우 빠르고 신뢰할 수 있는 LAN의 경우 가능한 한 큰 사이즈의 패킷을 사용하는 것이 좋다. 여러 종류의 네트워크를 고려할 때ㅐ 8킬로바이트, 즉 8,192바이트가 가장 적절한 패킷의 크기다.  

DatagramPacket을 생성하기 전에 데이터를 byte 배열로 변환하고, 변환된 배열을 data에 위치시키는 것이 일반적이지만, 반드시 그렇게 해야 하는 것은 아니다. 데이터그램이 생성되고 난 후, 그리고 데이터그램이 전송되기 전에 data를 변경하면, 데이터그램 안에 있는 데이터가 변경된다. 데이터는 DatagramPacket 내부 버퍼로 복사되지 않기 때문이다. 몇몇 애플리케이션에서는 이러한 점을 이용할 수도 있다. 예를 들어, 시간이 지남에 따라 변경되는 데이터를 data에 저장하고 1분마다 현재 데이터그램(가장 최근의 데이터)을 전송한다. 그러나 여러분이 원하지 않을 때, 데이터가 변경되지 않도록 하는 것이 더 중요하다. 특히 프로그램이 멀티스레드이고, 서로 다른 스레드가 데이터 버퍼에 쓰려고 할 경우, 원하지 않는 변경이 일어나지 않도록 하는 것이 중요하다. 이러한 경우라면, DatagramPacket을 생성하기 전에 데이터를 임시 버퍼로 복사해야 한다. 예를 들어, 다음 코드는 UTF-8로된 "This is a test" 문자열로 채워진 새로운 DatagramPacket을 생성한다. 패킷은 www.ibiblio.org의 7번 포트(에코 포트)로 향한다.  

```java
String s = "This is a test";
byte[] data = s.getBytes("UTF-8");

try {
  InetAddress ia = InetAddress.getByName("www.ibiblio.org");
  int port = 7;
  DatagramPacket dp = new DatagramPacket(data, data.length, ia, port);
  // 패킷을 전송한다.
} catch (IOException ex) {
}
```

대부분의 경우, 새로운 DatagramPacket을 생성할 때 데이터를 바이트 배열로 변경하는 부분이 가장 어렵다. 이 코드에서는 문자열을 전송하기 때문에 java.lang.String()의 getBytes() 메소드를 사용한다. java.io.ByteArrayOutputStream 클래스 또한 데이터그램에 포함될 데이터를 준비하는 데 매우 유용하게 사용할 수 있다.  

##### 4.1.3. get 메소드
<br/>
DatagramPacket은 데이터그램의 실제 데이터와 헤더의 몇몇 필드를 가져오는 6개의 메소드를 제공한다. 이 메소드는 대부분 수신된 데이터그램에 사용된다.  

+ public InetAddress getAddress()  
getAddress() 메소드는 원격 호스트의 주소를 포함하고 있는 InetAddress 객체를 반환한다. 인터넷에서 수신된 데이터그램인 경우, 데이터그램을 보낸 장비의 주소(출발지 주소)가 된다. 그리고 로컬에서 생성된 데이터그램인 경우, 데이터그램이 향하고 있는 주소(목적지 주소)를 반환한다. 일반적으로 수신자가 응답을 보내기 위해 UDP 데이터그램을 보낸 호스트를 확인하는 용도로 사용된다.  

+ public int getPort()  
getPort() 메소드는 원격 포트를 나타내는 정수를 반환한다. 인터넷에서 수신된 데이터그램인 경우, 패킷을 보낸 호스트의 포트가 반환된다. 그리고 로컬에서 생성된 데이터그램인 경우, 원격 장비에 패킷이 향하는 포트 번호이다.  

+ public SocketAddress getSocketAddress()  
getSocketAddress() 메소드는 원격 호스트의 IP 주소와 포트를 포함하고 있는 SocketAddress 객체를 반환한다. getInetAddress() 메소드와 마찬가지로 인터넷에서 수신된 데이터그램인 경우, 데이터글매을 보낸 호스트의 주소(출발지 주소)가 반환된다. 그리고 로컬에서 생성된 데이터그램인 경우, 데이터그램이 향하고 있는 호스트의 주소(목적지 주소)를 반환한다. 이 메소드는 일반적으로 응답을 보내기 전에 UDP 데이터그램을 보낸 호스트의 주소와 포트를 확인하기 위해 사용된다. getAddress()와 getPort()를 호출하는 것과 큰 차이는 없다. 또한 논블록 I/O를 사용할 경우, DatagramChannel 클래스는 SocketAddress는 수용하지만 InetAddress와 포트는 수용하지 않는다.  

+ public byte[] getData()  
getData() 메소드는 데이터그램의 데이터를 포함한 바이트 배열을 반환한다. 반환된 배열은 프로글매에서 사용하기 전에 종종 다른 데이터 형식으로 변환이 필요하다. 한 가지 방법으로는 바이트 배열을 문자열로 변경하는 것이다. 예를 들어, 네트워크에서 수신된 DatagramPacket 객체 dp가 있는 경우, 다음과 같이 UTF-8로 변환할 수 있다.  

```java
String s = new String(dp.getData(), "UTF-8");
```

데이터그램에 저장된 데이터가 텍스트가 아닌 경우, 조금 복잡해진다. 한 가지 방법은 getData()에 의해 반환된 바이트 배열을 ByteArrayInputStream으로 변환하는 것이다. 예를 들어:  

```java
InputStream in = new ByteArrayInputStream(packet.getData(), packet.getOffset(), packet.getLength());
```

ByteArrayInputStream를 생성할 때 오프셋과 길이를 명시해야 한다. ByteArrayInputStream() 생성자를 인자로 배열만 전달해서 사용하지 않도록 해야 한다. packet.getData() 메소드가 반환하는 배열은 네트워크로 받은 데이터로 채워진 것 이외의 여분의 공간이 있을 수 있다. 이 여분의 공간에는 DatagramPacket이 생성될 때 채워진 임의의 값이 들어 있다.  

변환된 ByteArrayInputStream은 다시 DataInputStream에 연결될 수 있다.  

```java
DataInputStream din = new DataInputStream(in);
```

이 데이터는 이제 DataInputStream의 readInt(), readLong(), readChar() 그리고 다른 메소드를 사용하여 읽을 수 있다. 물론 이것은 데이터그램의 송신자가 자바와 똑같은 데이터 형식을 사용한다고 가정한다. 보통 송신자가 자바로 작성된 경우가 이에 해당한다. 그리고 종종 다른 언어로 작성된 경우에도 데이터 형식이 일치하기도 한다. (요즘 컴퓨터들은 자바와 같은 부동소수점 형식을 사용한다. 그리고 대부분의 네트워크 프로토콜은 네트워크 바이트 순서로 2의 보수 정수를 표기하며 이 포맷 역시 자바와 일치한다.)  

+ public int getLength()  
getLength() 메소드는 데이터그램에 저장된 데이터의 바이트 수를 반환한다. 이 값은 getData() 메소드가 반환한 배열의 길이와 같지는 않다. getLength()가 반환한 정수 값은 getData()가 반환한 배열의 길이보다 작을 수 있다.  

+ public int getOffset()  
이 메소드는 getData() 메소드를 호출했을 때 반환된 배열에서 데이터가 시작되는 위치를 가리킨다.  

##### 4.1.4. set 메소드
<br/>
대부분의 경우 앞서 소개한 6개의 생성자만 이용해도 데이터그램을 생성하기에 충분하다. 그러나 또한 자바는 데이터그램이 생성된 다음에 데이터, 원격 주소, 그리고 원격 포트를 변경할 수 있는 몇몇 메소드를 제공한다. 이 메소드는 새로운 DatagramPacket 객체를 생성하고 가비지 컬렉트되는 데 소요되는 시간이 성능에 영향을 미칠 경우 사용할 수 있다. 새로운 객체를 생성하는 것보다 재사용하는 것이 훨씬 빠른 경우들이 있다. 예를 들어, 네트워크 대전 게임에서 총알이 발사될 때마다 또는 총알이 몇 센티미터를 이동할 때마다 데이터그램을 재전송할 경우, 매번 새로운 객체를 만들기보다 이미 생성된 객체를 재사용하는 편이 속도 향상에 도움이 된다. 그러나 결국 주목할 만한 속도의 향상을 위해서는 상대적으로 느린 회선보다 빠른 회선을 사용하는 편이 낫다.  

+ public void setData(byte[] data)  
setData() 메소드는 UDP 데이터그램에 적재된 데이터를 변경한다. 하나의 데이터그램에 담을 수 없는 큰 파일을 원격 호스트로 보낼 경우 유용하게 사용할 수 있다. 반복해서 동일한 DatagramPacket 객체를 전송하고, 반복할 때마다 적재된 데이터를 변경할 수 있다.  

+ public void setData(byte[] data, int offset, int length)  
이 메소드는 setData() 메소드를 오버로드한 변형이며, 많은 양의 데이터를 전송할 수 있는 또 다른 방법이다. 새로운 배열을 여러 번 전송하는 대신에 모든 데이터를 하나의 배열에 넣고 한 번에 한 조각씩 보낸다. 예를 들어, 다음 루프는 큰 배열을 512바이트 덩어리씩 전송한다.  

```java
int offset = 0;
DatagramPacket dp = new DatagramPacket(bigarray, offset, 512);
int bytesSent = 0;
while (bytesSent < bigarray.length) {
  socket.send(dp);
  bytesSent += dp.getLength();
  int bytesToSend = bigarray.length - bytesSent;
  int size = (bytesToSend > 512) ? 512: bytesToSend;
  dp.setData(bigarray, bytesSent, size);
}
```

반면에 이 방법은 데이터가 반드시 전달된다고 믿거나, 아니면 데이터가 제대로 전달되지 않아도 무시해야 한다. 이 방법을 사용할 때 각 패킷에 시퀀스 번호를 붙이거나 다른 태그를 달아서 표시하는 것은 쉽지 않은 문제다.  

+ public void setAddress(InetAddress remote)  
setAddress() 메소드는 데이터그램 패킷의 목적지 주소를 변경한다. 이 메소드를 사용하면 같은 데이터그램을 다수의 다른 수신자에게 전송할 수 있다. 예를 들어:  

```java
String s = "Really Important Message";
byte[] data = s.getBytes("UTF-8");
DatagramPacket dp = new DatagramPacket(data, data.length);
dp.setPort(2000);
int network = "128.238.5.";
for (int host = 1; host < 255; host++) {
  try {
    InetAddress remote = InetAddress.getByName(network + host);
    dp.setAddress(remote);
    socket.send(dp);
  } catch (IOException ex) {
    // 에러를 생략하고 계속해서 다음 호스트로 전송한다.
  }
}
```

이런 방법이 적절한지는 애플리케이션마다 다르다. 위 코드처럼 로컬 네트워크에 연결된 모든 호스트로 전송할 경우, 이 방법보다는 로컬 브로드캐스트 주소를 사용하여 네트워크가 이 일을 하게끔 하는 것이 낫다. 로컬 브로드캐스트 주소는 IP 주소의 서브넷 ID 부눈을 모두 1로 설정하는 것이다. 예를 들어, 폴리테크닉 대학의 네트워크 주소는 128.238.0.0이다. 따라서 이 대학의 브로드캐스트 주소는 128.238.255.255가 된다. 128.238.255.255로 전송되는 데이터그램은 같은 네트워크에 있는 모든 호스트로 복사된다. (라우터나 방화벽이 이를 차단할 수도 있지만, 네트워크 환경에 따라 다르다.)  

호스트들이 좀 더 넓게 퍼져 있는 경우에는 멀티캐스팅을 사용하는 편이 나을 것이다. 실제로 멀티캐스팅은 여기서 설명한 같은 DatagramPacket 클래스를 사용한다. 그러나 호스트 주소 대신 멀티캐스트 주소를 사용하고 DatagramSocket 대신 MulticastSocket을 사용한다.  

+ public void setPort(int port)  
setPort() 메소드는 데이터그램의 목적지 포트 번호를 변경한다. 한 가지 예로 FSP 같은 UDP 기반의 특정 서비스가 실행 중인 열린 포트를 찾는 포트 스캐너 애플리케이션에서 사용할 수 있다. 또 다른 가능성은 일부 네트워크 게임이나 화상회의 서버에서 모든 클라이언트들이 다른 호스트의 다른 포트에서 실행되고 있는 경우 데이터를 전송하기 전에 포트를 변경하는 용도로 사용할 수 있다. 이 경우에 setPort() 메소드와 함께 setAddress() 메소드도 필요하다.  

+ public void setAddress(SocketAddress remote)  
setSocketAddress() 메소드는 데이터그램이 패킷이 전송될 목적지의 주소와 포트를 변경한다. 응답을 보낼 때 이 메소드를 사용할 수 있다. 예를 들어, 다음 코드는 데이터그램 패킷을 수신하고 같은 주소로 "Hello there"을 포함한 패킷을 응답한다.  

```java
DatagramPacket input = new DatagramPacket(new byte[8192], 8192);
socket.receive(input);
DatagramPacket output = new DatagramPacket("Hello there".getBytes("UTF-8"), 11);
SocketAddress address = input.getSocketAddress();
socket.send(output);
```

물론 SocketAddress 대신 InetAddress 객체와 포트를 사용하여 같은 코드를 작성할 수 있다. 다만 코드가 조금 길어진다.  

+ public void setLength(int length)  
setLength() 메소드는 단순히 채워지지 않은 공간이 아닌 데이터그램의 데이터 부분에 해당하는 내부 버퍼의 데이터 길이를 변경한다. 데이터그램이 수신되면 데이터그램의 길이는 들어온 데이터의 길이로 설정된다.  

이 말은 곧, 같은 DatagramPacket을 사용하여 다른 데이터그램을 수신할 경우, 처음 받은 바이트 수 이상을 받을 수 없도록 제한됨을 의미한다. 즉, 처음 10바이트의 데이터그램을 받고 나면, 그 다음부터 받게 되는 모든 데이터그램은 10바이트에서 잘린다. 처음 9바이트의 데이터그램을 받고 나면, 그 다음부터 받게 되는 모든 데이터그램은 9바이트에서 잘린다. 이 메소드를 사용하여 버퍼의 길이를 리셋하여 이후의 데이터그램이 잘리지 않도록 할 수 있다.  

### 5. DatagramSocket 클래스
<br/>
DatagramPacket을 보내거나 받기 위해서는, 데이터그램 소켓을 열어야 한다. 자바에서 데이터그램 소켓은 DatagramSocket 클래스를 통해 만들어지고 접근하다.  

+ public class DatagramSocket extends Object  

모든 데이터그램 소켓은 로컬 포트에 바인드되며, 이 포트로 들어오는 연결을 기다록, 나가는 데이터글매의 헤더에 저장한다. 여러분이 클라이언트를 작성하고 있다면, 로컬 포트가 뭔지 신경 쓰지 않아도 된다. 생성자를 호출할 때 시스템은 사용하지 않은 임의의 포트를 할당해 준다. 이 포트는 나가는 모든 데이터그램에 저장되며 서버가 응답을 보낼 때 사용된다.  

여러분이 서버를 작성하고 있다면, 클라이언트는 서버가 어떤 포트에서 데이터그램의 수신을 대기하고 있는지 알아야 한다. 따라서 서버는 DatagramSocket을 생성할 때, 대기할 로컬 포트를 명시한다. 그 외에는 클라이언트와 서버가 사용하는 소켓이 같다. 두 소켓은 임의의 포트를 사용하는지, 아니면 알고 잇는 포트를 사용하는지가 유일한 차이점이다. UDP에서는 DatagramServerSocket 같은 클래스는 제공되지 않는다.  

#### 5.1. 생성자
<br/>
DatagramSocket 생성자들은 DatagramPacket 생성자들처럼 다양한 상황에서 사용된다. 첫번째 생성자는 임의의 로컬 포트에 대해 데이터그램 소켓을 연다. 두 번째 생성자는 알고 있는 포트를 사용하여 모든 로컬 네트워크 인터페이스에서 대기하는 데이터그램 소켓을 연다. 마지막 두 생성자는 특정 네트워크 인터페이스에 대해 알고 있는 로컬 포트를 사용하여 데이터그램 소켓을 연다. 모든 생성자들은 로컬 주소와 포트만을 처리한다. 원격 주소와 포트는 DatagramPacket 안에 저장되며, DatagramSocket에는 저장되지 않는다. 사실 하나의 DatagramSocket으로 다수의 원격 호스트와 포트로부터 데이터그램을 수신할 수 있다.  

+ public DatagramSocket() throws SocketException  
이 생성자는 임의의 포트에 바인드된 소켓을 생성한다. 예를 들어:  

```java
try {
  DatagramSocket client = new DatagramSocket();
  // 패킷 전송
} catch (SocketException ex) {
  System.err.println(ex);
}
```

서버와 대화를 시작하는 클라이언트에서 이 생성자를 사용할 수 있다. 이 생성자를 사용할 때는 소켓이 어떤 포트에 바인드되었는지 신경 쓰지 않아도 된다. 서버는 수신된 데이터글매의 헤더 정보를 참조하여 클라이언트에게 응답을 보낸다. 그리고 시스템 할당해 주는 포트를 사용할 경우, 직접 사용되지 않은 포트를 찾아야 하는 수고스러움도 덜 수 있다. 어떤 이유로 로컬 포트를 알아야 하는 경우가 있다. 이럴 때는 getLocalPort() 메소드를 호출하여 확인할 수 있다.  

동일한 소켓을 사용하여 서버가 응답으로 보내는 데이터그램을 받을 수 있다. 생성자는 소켓이 포트로 바인드할 수 없는 경우 SocketException 예외를 발생시킨다. 생성자가 예외를 발생시키는 경우는 극히 드물다. 시스템이 이용 가능한 포트를 직접 선택하므로, 소켓을 열 수 없는 경우는 잘 발생하지 않는다.  

+ public DatagramSocket(int port) throws SocketException  
이 생성자는 port 인자로 명시된 특정 포트에서 수신 데이터그램을 기다리는 소켓을 생성한다. 이미 알고 있는 포트를 사용하는 서버를 작성할 때 이 생성자를 사용할 수 있다. SocketException는 소켓을 생성할 수 없을 때 예외를 발생시킨다. 생성자가 실패하는 경우는 일반적으로 두 가지가 있다. 명시된 포트가 이미 사용 중인 경우 또는 1024보다 작은 포트에 연결을 시도하면서 적절한 권한이 없는 경우가 있다. 즉, 유닉스 시스템에서 루트 권한이 아닌 경우다.  

TCP 포트와 UDP 포트는 서로 아무런 연관이 없다. 두 개의 서로 다른 프로그램이 하나는 UDP를 사용하고, 또 다른 하나는 TCP를 사용할 경우, 서로 같은 포트 번호를 사용할 수 있다.  

+ public DatagramSocket(int port, InetAddress interface) throws SocketException  
이 생성자는 주로 멀티홈 호스트(multi-homed host)에서 사용된다. 이 생성자는 특정 포트와 특정 네트워크 인터페이스에서 대기 중인 소켓을 생성한다. port 인자는 이 소켓이 데이터그램을 대기할 포트 번호이다. TCP 소켓처럼, 유닉스 시스템에서 1024 이하의 포트를 사용하는 DatagramSocket을 생성할 경우 루트 권한이 필요하다. address 인자는 호스트의 네트워크 주소에 대응하는 InetAddress 객체이다. 소켓을 생성할 수 없는 경우 SocketException 예외가 발생한다. 이 생성자는 일반적으로 다음의 세 가지 이유로 소켓 생성에 실패한다. 명시된 포트가 이미 사용 중인 경우, 권한 없이 1024 이하의 포트에 연결을 시도한 경우, 또는 명시된 주소가 시스템의 네트워크 인터페이스 주소가 아닌 경우이다.  

+ public DatagramSocket(SocketAddress interface) throws IOException  
이 생성자는 SocketAddress로부터 네트워크 인터페이스 주소와 포트를 읽는다는 것만을 제외하고는 바로 이전의 생성자와 유사하다. 예를 들어, 다음 코드는 로컬 루프백 주소에서만 대기 중인 소켓을 생성한다.  

```java
SocketAddress address = new InetSocketAddress("127.0.0.1", 9999);
DatagramSocket socket = new DatagramSocket(address);
```

+ protected DatagramSocket(DatagramSocketImpl impl) throws SocketException  
이 생성자를 사용하면 서브클래스가 기본 구현을 맹목적으로 받아들이지 않고 자신만의 UDP 프로토콜 구현을 제공할 수 있게 된다. 다른 네 개의 생성자들이 만든 소켓과는 달리, 이 소켓은 초기에 포트에 바인드되지 않는다. 이 소켓을 사용하기 전에, bind() 메소드를 사용하여 소켓을 SocketException에 바인드해야 한다.  

> + public void bind(SocketAddress addr) throws SocketException  

이 메소드에 널(null)을 전달할 경우, 사용 가능한 아무 주소와 포트로 바인드된다.  

#### 5.2. 데이터그램 보내기와 받기
<br/>
DatagramSocket 클래스의 주 목적은 UDP 데이터그램을 보내고 받는 것이다. 하나의 소켓으로 보내기와 받기 모두를 할 수 있다. 사실, 하나의 소켓으로 동시에 다수의 호스트에 대해 보내거나 받을 수 있다.  

+ public void send(DatagramPacket dp) throws IOException  
DatagramPacket과 DatagramSocket을 생성하고 나면, 소켓의 send() 메소드에 패킷을 전달하여 보낼 수 있다. 예를 들어, DatagramSocket 객체 theSocket과 DatagramPacket 객체 theOutput이 있을 때, 소켓을 사용하여 theOutput을 다음과 같이 전송한다.  

```java
theSocket.send(theOutput);
```

데이터를 보내는 데 문제가 발생할 경우, 이 메소드는 IOExeption 예외를 발생시킨다. 그러나 UDP의 비신뢰 기반이라는 특성 때문에 패킷이 목적지에 도착하지 않더라도 예외가 발생하지 않는다. 호스트의 네이티브 네트워크 소프트웨어가 지원하는 것보다 큰 데이터그램을 전송하려고 할 경우, IOExeption 예외가 발생한다. 그러나 발생하지 않을 수도 있다. 이것은 운영체제에 구현된 네이티브 UDP 소프트웨어에 따라 다르게 동작한다. 이 메소드는 또한 SecurityManager가 패킷을 보낼 호스트와 통신을 차단할 경우 SecurityException 예외가 발생한다. 이 문제는 주로 애플릿이나 간접적으로 로드된 코드에서 발생한다.  

+ public void receive(DatagramPacket dp) throws IOExeption  
이 메소드는 네트워크로부터 하나의 UDP 데이터글매을 수신하고 미리 준비된 DatagramPacket 객체인 dp에 저장한다. ServerSocket 클래스에서 제공하는 accept() 메소드처럼 이 메소드는 데이터그램이 도착할 때까지 호출한 스레드를 블록시킨다. 여러분의 프로그램에서 데이터그램을 기다리는 것 이외에 다른 작업을 처리해야 할 경우, 분리된 스레드에서 이 메소드를 호출해야 한다.  

인자로 제공되는 데이터그램의 버퍼는 수신된 데이터를 저장하기에 충분히 커야 한다. 그렇지 않은 경우 receive() 메소드는 버퍼가 담을 수 있는 만큼의 데이터를 저장하고 나머지를 버린다. UDP 데이터그램에서 데이터 부분이 차지할 수 있는 최대 크기는 6만 5,507바이트임을 기억해 두도록 하자. (IP 데이터그램의 최대 크기인 6만 5,535바이트에서 IP 헤더 크기인 20바이트를 빼고, 다시 UDP 해더 크기인 8바이트를 뺀 값이다.) UDP를 사용하는 몇몇 애플리케이션 프로토콜들은 패킷에 담을 수 있는 최대 바이트 수를 제한하고 있다. 예를 들어, NFS는 최대 패킷 크기로 8,192바이트를 사용한다.  

데이터를 수신하는 데 문제가 있는 경우, receive() 메소드는 IOException 예외를 발생시킨다. 하지만 실제로 패킷 손실과 같은 TCP 스트림을 종료시키는 문제는 자바가 알아채기 전에 네트워크나 네트워크 스택에 의해 조용히 버려지기 때문에 이 예외는 거의 발생하지 않는다.  

+ public void close()  
DatagramSocket 객체의 close() 메소드를 호출하면 수켓이 사용 중인 포트가 해제된다. 스트림과 TCP 소켓에서와 마찬가지로 finally 블록을 사용하여 사용이 끝난 데이터그램 소켓이 알아서 닫히도록 할 수 있다.  

```java
DatagramSocket server = null;
try {
  server = new DatagramSocket();
  // 소켓을 사용한다.
} catch (IOException ex) {
  System.err.println(ex);
} finally {
  try {
    if (server != null) server.close();
  } catch (IOException ex) {
  }
}
```

자바 7에서 DatagramSocket은 AutoCloseable 인터페이스를 구현하므로 try-with-resources 구문을 사용할 수 있다.  

```java
try (DatagramSocket server = new DatagramSocket()) {
  // 소켓을 사용한다...
}
```

DatagramSocket을 실행 흐름 중에 닫는 것도 나쁘지 않다. 특히 프로그램이 상당한 시간동안 계속해서 실행될 경우, 불필요한 소켓을 닫아 주는 것이 중요하다. 프로그램이 DatagramSocket 사용 후 즈기 종료될 경우, 소켓을 명시적으로 닫을 필요는 없다. 이때 소켓은 가비지 컬렉션에 의해 자동으로 종료된다. 그러나 우연히 좋아서 메모리가 부족한 상황과 겹치지 않는 한, 자바는 소켓이나 포트가 부족하달고 해서 가비지 컬렉터를 실행시키지 않는다. 불필요한 소켓을 닫는 것은 문제가 되지 않으며, 좋은 프로그래밍 습관이기도 하다.  

+ public int getLocalPort()  
DatagramSocket의 getLocalPort() 메소드는 소켓이 대기 중인 로컬 포트를 나타내는 정수를 반환한다. 임의의 포트로 DatagramSocket 객체를 생성한 경우 할당된 포트를 확인하기 위해 이 메소드를 사용할 수 있다. 예를 들어:  

```java
DatagramSocket ds = new DatagramSocket();
System.out.println("The socket is using port " + ds.getLocalPort());
```

+ public InetAddress getLocalAddress()  
DatagramSocket의 getLocalAddress() 메소드는 소켓이 바인드된 로컬 주소를 나타내는 InetAddress 객체를 반환한다. 이 메소드는 실제로 거의 사용되지 않는다. 일반적으로 이미 알고 있거나, 어느 주소로 소켓이 대기 중인지 알아야 할 경우가 잘 없다.  

+ public SocketAddress getLocalSocketAddress()  
getLocalSocketAddress() 메소드는 소켓이 연결된 로컬 인터페이스와 포트를 감싸고 있는 SocketAddress 객체를 반환한다. getLocalAddress()처럼 이 메소드도 실제 필요한 상황을 상상하기가 쉽지 않다. 아마도 이 메소드는 setLocalSocketAddress() 메소드와 함께 병렬처리를 위해 존재하는 것 같다.  

#### 5.3. 연결 관리하기
<br/>
TCP 소켓과 달리 데이터그램 소켓은 대화 상대에 대해서 그렇게 엄격하지 않다. 사실 UDP 소켓은 기본적으로 누구하고도 대화할 수 있다. 그러나 보통 여러분은 이러한 방식을 원하진 않을 것이다. 예를 들어, 애플릿은 제공한 호스트에 대해서만 데이터그램을 보내고 받기가 허용된다. 그리고 NFS 또는 FSP 클라이언트의 경우 통신 중인 서버로부터만 패킷을 수용해야 한다. 또한 네트워크 게임의 경우 같은 게임을 하고 있는 사람들로부터 전송되는 데이터그램만을 대기해야 한다. 다음 다섯 개의 메소드를 사용하며, 선택된 호스트에 대해서만 데이터그램을 보내거나 수신할 수 있다.  

+ public void connect(InetAddress host, int port)  
connect() 메소드는 TCP와는 달리 실제 연결을 만들지는 않다. 그러나 이 메소드는 해당 DatagramSocket 객체가 인자로 전달된 원격 호스트와 포트에 대해서만 패킷을 보내거나 받을 수 있도록 명시한다. 다른 호스트 또는 포트로 패킷을 보내려고 할 경우 IllegalArgumentException 예외가 발생한다. 다른 호스트나 포트로부터 패킷이 수신될 경우 버려지며 예외나 다른 알림은 발생하지 않는다.  

connect() 메소드 호출 시 SecurtityManager에 의해 보안 검사가 수행된다. 가상 머신이 해당 호스트와 포트로 데이터 전송을 허가할 경우 보안 검사는 조용히 통과한다. 그렇지 않을 경우에는 SecurityException 예외가 발생한다. 그러나 연결이 완료된 이후에는 해당 DatagramSocket 객체에 대해 send()와 receive()를 호출해도 더 이상 보안 검사를 하지 않는다.  

+ public void disconnect()  
disconnect() 메소드는 연결된 DatagramSocket의 연결을 해제한다. 해당 소켓은 연결이 해제된 이후에 다시 모든 호스트와 포트로부터 데이터를 보내거나 받을 수 있게 된다.  

+ public int getPort()  
DatagramSocket 객체가 연결된 경우에만 이 메소드는 연결된 원격 포트를 반환한다. 그렇지 않은 경우 -1을 반환한다.  

+ public InetAddress getInetAddress()  
DatagramSocket 객체가 연결된 경우에만 getInetAddress() 메소드는 연결된 원격 호스트의 주소를 반환한다. 그렇지 않은 경우 널(null)을 반환한다.  

+ public InetAddress getRemoteSocketAddress()  
DatagramSocket 객체가 연결된 경우 getRemoteSocketAddress() 메소드는 연결된 원격 호스트의 주소를 반환한다. 그렇지 않은 경우 널(null)을 반환한다.  

### 6. 소켓 옵션
<br/>
자바는 6개의 UDP 소켓을 위한 옵션을 제공한다.  

+ SO&#95;TIMEOUT  
SO&#95;TIMEOUT은 receive() 메소드가 패킷을 대기하는 밀리초 단위의 시간이다. 이 시간이 지나면 IOException의 서브클래스인 InterruptedIOException 예외가 발생한다. 이 값은 음수가 아닌 값을 사용해야 한다. SO&#95;TIMEOUT으로 0이 설정될 경우 receive() 메소드는 타임아웃 없이 무한히 대기한다. 이 값은 setSoTimeout() 메소드를 사용하여 변경할 수 있으며 getSoTimeout() 메소드를 사용하여 설정된 값을 확인할 수 있다.  

> + public void setSoTimeout(int timeout) throws SocketException<br/>
+ public int getSoTimeout() throws IOException  

기본값이 0이기 때문에 결코 타임아웃이 발생하지 않는다. 그리고 사실 SO&#95;TIMEOUT의 설정이 필요한 경우가 거의 없다. 응답 시간에 제한이 있는 보안 프로토콜을 구현하는 경우 이 설정이 필요할 수도 있다. 또한 응답 시간을 체크하여 대화 중인 호스트가 죽었는지를 판단하는 데 사용할 수도 있다.  

setSoTimeout() 메소드는 데이터그램 소켓에 대한 SO&#95;TIMEOUT 필드를 설정한다. 타임아웃이 발생할 경우 블록 모드인 receive() 메소드를 호출하기 전에 이 옵션을 설정해야 한다. receive() 메소드가 데이터그램을 대기하고 있는 동안에는 이 옵션을 변경할 수 없다. timeout 인자는 0보다 크거나 같아야 한다. 예를 들어:  

```java
try {
  byte[] buffer = new byte[2056];
  DatagramPacket dp = new DatagramPacket(buffer, buffer.length);
  DatagramSocket ds = new DatagramSocket(2048);
  ds.setSoTimeout(30000); // 최대 30초까지 블록
  try {
    ds.receive(dp);
    // 수신된 패킷 처리
  } catch (SocketTimeoutException ex) {
    ss.close();
    System.err.println("No connection within 30 seconds");
  }
} catch (SocketException ex) {
  System.err.println(ex);
} catch (IOException ex) {
  System.err.println("Unexpected IOException: " + ex);
}
```

getSoTimeout() 메소드는 DatagramSocket 객체의 현재 SO&#95;TIMEOUT 필드값을 반환한다. 예를 들어:  

```java
public void printSoTimeout(DatagramSocket ds) {
  int timeout = ds.getSoTimeout();
  if (timeout > 0) {
    System.out.println(ds + " will time out after " + timeout + "milliseconds.");
  } else if (timeout == 0) {
    System.out.println(ds + " will never time out.");
  } else {
    System.out.println("Something is seriously wrong with " + ds);
  }
}
```

+ SO&#95;RCVBUF  
DatagramSocket의 SO&#95;RCVBUF 옵션은 TCP 소켓의 SO&#95;RCVBUF와 밀접하게 관련되어 있다. 이 옵션은 네트워크 I/O에 사용된 버퍼의 크기를 결정한다. 보통 큰 버퍼는 더 많은 데이터를 저장할 수 있기 때문에 성능을 개선시키는 효과가 있다. 버퍼가 가득 찼을 때 재전송되는 TCP 데이터그램과 달리 UDP는 버려지기 때문에 충분히 큰 수신 버퍼를 사용할 경우 TCP보다는 UDP에서 더 좋은 효과를 볼 수 있다. 게다가 SO&#95;RCVBUF는 애플리케이션이 수신할 수 있는 데이터그램 패킷의 최대 크기를 설정한다. 수신 버퍼의 크기에 맞지 않은 패킷은 자동으로 버려진다.  

> + public void setReceiveBufferSize(int size) throws SocketException<br/>
+ public int getReceiveBufferSize() throws SocketException  

setReceiveBufferSize() 메소드는 현재 소켓의 버퍼 입력에 사용할 바이트 수를 제안한다. 그러나 실제 내부 구현은 이 제안을 무시할 수 있다. 예를 들어, 많은 4.3 BSD 기반의 시스템은 최대 수신 버퍼의 크기가 52K이고, 이 값보다 높게 설정할 수 없도록 한다. 다른 시스템의 경우 약 240K로 제한된 경우도 있다. 자세한 내용은 플랫폼에 따라 많은 차이가 있다. 따라서 setReceiveBufferSize() 메소드를 호출하여 설정한 다음에 getReceiveBufferSize()를 호출하면 실제 설정된 수신 버퍼의 크기를 확인할 수 있다, getReceiveBufferSize() 메소드는 현재 소켓의 입력에 사용되는 버퍼의 크기를 반환한다.  

이 두 메소드는 내부 소켓 구현이 SO&#95;RCVBUF 옵션을 지원하지 않을 경우 SocketException 예외를 발생시킨다. 비POSIX 운영체제에서 이러한 경우가 발생할 수 있다. setReceiveBufferSize() 메소드는 0 또는 0보다 작은 인자가 전달될 경우 IllegalArgumentException 예외가 발생시킨다.  

+ SO&#95;SNDBUF  
DatagramSocket은 네트워크 출력에 사용될 송신 버퍼의 크기를 제안하거나 설정된 크기를 확인하는 메소드를 제공한다.  

> + public void setSendBufferSize(int size) throws SocketException<br/>
+ public int getSendBufferSize() throws SocketException  

setSendBufferSize() 메소드는 이 소켓의 출력 버퍼의 크기를 제안한다. 그러나 실제 내부 구현은 이 제안을 무시할 수 있다. 따라서 setSendBufferSize() 메소드 호출 후 getSendBufferSize()를 호출하면 실제 반영된 크기를 확인할 수 있다.  

이 두 메소드는 내부 소켓 구현이 SO&#95;SNDBUF 옵션을 지원하지 않을 경우 SocketException 예외를 발생시킨다. setSendBufferSize() 메소드는 또한 0보다 작은 인자가 전달될 경우 IllegalArgumentException 예외를 발생시킨다.  

+ SO&#95;REUSEADDR  
UDP 소켓에서 SO&#95;REUSEADDR 옵션은 TCP의 해당 옵션과는 다른 의미를 가진다. UDP에서 SO&#95;REUSEADDR은 다수의 데이터그램 소켓이 동시에 같은 포트와 주소로 바인드할 수 있는지를 제어한다. 다수의 소켓이 같은 포트로 바인드도리 경우 수신될 패킷은 바인드된 모든 소켓으로 복사된다. 이 옵션은 다음 두 메소드로 제어된다.  

> + public void setReuseAddress(boolean on) throws SocketException
+ public boolean getReuseAddress() throws SocketException  

이 옵션이 안정적으로 동작하기 위해서는 새로운 소켓이 포트에 바인드되기 전에 setResueAddress() 메소드가 호출되어야 한다. 이 말은 곧 DatagramImpl을 인자로 받는 protected 생성자를 사용하여 소켓이 연결되지 않은 상태로 생성되어야 한다는 의미다. 즉, 이 메소드는 일반적인 DatagramSocket과는 동작하지 않는다. 재사용 가능 포트는 멀티캐스트 소켓에서 가장 일반적으로 사용된다. 데이터그램 채널 또한 재사용 가능 포트를 설정할 수 있는 연결되지 않은 데이터그램 소켓을 생성할 수 있다.  

+ SO&#95;BROADCAST  
SO&#95;BROADCAST 옵션은 해당 소켓이 192.168.254.255와 같은 브로드캐스트 주소로 패킷을 보내거나 받을 수 있는지를 제어한다. 여기서 192.168.254.255는 로컬 주소로 192.168.254.&#42;을 사용하는 네트워크를 위한 로컬 네트워크 브로드캐스트 주소이다. UDP 브로드캐스팅은 종종 DHCP와 같은 프로토콜에 사용된다. DHCP 클라이언트는 로컬 네트워크에 있는 미리 주소를 알지 못하는 서버와 통신을 해야 한다. 이 옵션은 다음 두 메소드로 제어된다.  

> + public void setBroadcast(boolean on) throws SocketException<br/>
+ public boolean getBroadcast() throws SocketException  

라우터와 게이트웨이는 일반적으로 브로드캐스트 메시지를 전달하지 않지만, 로컬 네트워크에서는 여전히 잘 전달된다. 기본적으로 이 옵션은 켜져 있지만 다음과 같이 끌 수 있다.  

```java
socket.setBroadcast(false);
```

이 옵션은 소켓이 바인드된 다음에도 변경될 수 있다.  

일부 구현에서는, 소켓이 브로드캐스트 패킷을 받을 수 없는 특정 주소로 바인드되기도 한다. 다시 말해, 브로드캐스트 패킷을 받기 위해서는 DatagramPacket(InetAddress address, int port) 생성자가 아닌 DatagramPacket(int port) 생성자를 사용해야 한다. SO&#95;BROADCAST 옵션을 true로 설정하는 것뿐만 아니라 이러한 것도 고려해야 한다.  

+ IP&#95;TOS  
트래픽 클래스는 각 IP 패킷 헤더에 있는 IP&#95;TOS 필드의 값으로 결정된다. 이 옵션은 TCP와 UDP가 기본적으로 동일한 의미로 사용된다. 결국 TCP와 UDP는 IP 위에 구현되기 때문에 실제 패킷의 라우트 경로와 우선순위는 IP에 의해 결정된다.  

DatagramSocket과 Socket에서 setTrafficClass()와 getTrafficClass() 두 메소드는 실제 아무런 차이가 없다. 단지 이 두 클래스는 공통의 슈퍼클래스가 없기 때문에 여기서 같은 기능을 반복해서 구현한다. 이 두 메소드를 사용하면 해당 소켓의 서비스 클래스를 확인하거나 설정할 수 있다.  

> + public int getTrafficClass() throws SocketException<br/>
+ public void setTrafficClass(int getTrafficClass) throws SocketException  

트래픽 클래스는 0에서 255 사이의 정수값으로 제공된다. 이 인자는 TCP 헤더의 8비트로 복사되므로, 인자의 하위 바이트만을 사용해야 한다. 그리고 범위를 벗어난 값을 사용할 경우 IllegalArgumentException 예외가 발생한다.  

이 옵션에 대한 자바 공식 문서는 심각하게 오래된 상태다. 그리고 이 문서는 네 가지 트래픽 클래스를 위한 비트 필드를 기반으로 한 QoS 기법을 설명하고 있다. 저렴한 비용, 높은 신뢰성, 최대 처리량, 그리고 최소 지연, 이 기법은 널리 구현되지 못했으며 앞으로도 마찬가지일 것이다.  

다음 코드는 트래픽 클래스로 10111000을 설정하여 소켓이 EF(Expedited Forwarding)을 사용하도록 설정한다.  

```java
DatagramSocket s = new DatagramSocket();
s.setTrafficClass(0xB8) // 바이너리로 10111000
```

내부 소켓 구현은 이러한 요청에 대해 실제 구현이 강제되지 않는다. 일부 구현에서는 이러한 값들을 완전히 무시된다. 특히 안드로이드의 경우 setTrafficClass() 메소드는 아무런 동작을 하지 않는다. 네이티브 네트워크 스택이 요청된 서비스 클래스를 제공할 수 없는 경우, 자바는 SocketException 예외를 발생시킬 수 있지만 필수는 아니다.  

### 7. DatagramChannel 클래스
<br/>
SokcetChannel과 ServerSocketChannel 클래스가 논블록 TCP 애플리케이션에서 사용되는 것과 같은 방법으로 DatagramChannel 클래스가 논블록 UDP 애플리케이션에서 사용된다. SocketChannel과 ServerSocketChannel 클래스처럼, DatagramChannel 클래스는 셀렉터에 등록 가능하도록 만들어 주는 SelectionChannel 클래스의 서브클래스다. 이 클래스는 단일 스레드에서 다수의 클라이언트와 통신을 관리해야 하는 서버에서 유용하게 사용된다. 그러나 UDP는 그 자체가 이미 비동기적인 특성을 갖고 있기 때문에 TCP에 비해 그 효과가 크지 않다. UDP에서 단일 데이터그램 소켓은 다수의 클라이언트에 대해 모든 입출력 요청을 처리할 수 있다. 이러한 작업을 DatagramChannel을 사용하여 비동기적인 방법으로 처리할 수 있다. DatagramChannel의 메소드는 바로 받거나 보낼 데이터가 없는 경우 즉시 반환된다.  

#### 7.1. DatagramChannel 사용하기
<br/>
DatagramChannel 클래스는 기존 UDP API를 거의 완벽하게 대체한다. 자바 6과 이전 버전에서는 채널을 포트에 바인드하기 위해 여전히 DatagramSocket 클래스를 사용해야 했지만 자바 7부터는 이것조차 사용할 필요가 없다. SocketChannel 클래스 하나로 바이트 버퍼를 읽고 쓸 수 있다.  

##### 7.1.1. 소켓 열기
<br/>
java.nio.channels.DatagramChannel 클래스는 public으로 선언된 생성자를 제공하지 않는다. 대신 정적 open() 메소드를 사용하여 DatagramChannel 객체를 생성한다. 예를 들어:  

```java
DatagramChannel channel = DatagramChannel.open();
```

이 채널은 초기에 어떤 포트로도 바인드되지 않는다. 이 채넝릉 바인드하기 위해서는 socket() 메소드를 사용하여 채널에 연결된 DatagramSocket 객체에 접근해야 한다. 예를 들어 다음 코드는 채널을 포트 3141로 바인드한다.  

```Java
SocketAddress address = new InetSocketAddress(3141);
DatagramSocket socket = channel.socket();
socket.bind(address);
```

자바 7에서는 편의를 위해 bind() 메소드를 DatagramChannel에 추가하였다. 그래서 더 이상 DatagramSocket을 사용할 필요가 없다. 예를 들어:  

```java
SocketAddress address = new InetSocketAddress(3141);
channel.bind(address);
```

##### 7.1.2. 수신하기
<br/>
receive() 메소드는 채널로부터 하나의 데이터그램 패킷을 읽어 ByteBuffer에 저장한다. 그리고 이 메소드는 패킷을 보낸 호스트의 주소를 반환한다.  

+ public SocketAddress receive(ByteBuffer dst) throws IOException  

채널이 블록 모드(기본적으로 블록 모드이다)인 경우 이 메소드는 패킷을 읽을 때까지 반환되지 않는다. 채널이 논블록 모드인 경우 이 메소드는 읽을 패킷이 없는 경우 즉시 널(null)을 반환한다.  

수신된 데이터그램 패킷이 인자로 전달된 버퍼보다 큰 경우 나머지 데이터는 아무런 알림도 없이 버려진다. BufferOverflowException나 이와 유사한 어떠한 예외도 받을 수 없다. UDP의 신뢰할 수 없는 특성을 잘 보여 준다. 이러한 동작은 시스템 내에 신뢰할 수 없는 추가적인 계층이 있음을 보여 준다. 데이터가 네트워크를 통해 안전하게 전송되는 경우에도 여전히 프로그램 안에서 손실될 수 있다.  

##### 7.1.3. 보내기
<br/>
send() 메소드는 ByteBuffer로부터 하나의 데이터그램 패킷을 두 번째 인자로 제공된 주소로 보내기 위해 채널에 쓴다.  

+ public int send(ByteBuffer src, SocketAddress target) throws IOException  

동일한 데이터를 다수의 클라이언트에게 보내고자 할 경우 소스 ByteBuffer를 재사용할 수 있다. 다만 재사용하기 전에 해당 버퍼를 되감아(flip()) 줘야 한다.  

send() 메소드는 쓰인 바이트 수를 반환한다. 이 반환 값은 버퍼에서 전송할 수 있는 바이트의 수를 나타내는 값이거나 0이다. 채널이 논블록 모드이고 데이터를 바로 보낼 수 없는 경우에 0을 반환한다. 그 외에 채널이 블록 모드인 경우 send()는 단순히 버퍼 안의 모든 데이트를 보낼 때까지 기다린다.  

##### 7.1.4. 연결하기
<br/>
데이터그램 채널을 열고 나면 connect() 메소드를 사용하여 해당 채널을 특정 원격 호스트에 연결할 수 있다.  

```java
SocketAddress remote = new InetSocketAddress("time.nist.gov", 37);
channel.connect(remote);
```

이렇게 생성된 채널은 연결된 호스트에 대해서만 데이터를 주고받을 수 있다. SocketChannel의 connect() 메소드와는 달리, UDP는 비연결 프로토콜이기 때문에, 이 메소드만으로는 어떠한 패킷도 보내거나 받을 수 없다. 이 메소드는 단지 전송 준비가 된 데이터가 있을 때 패킷을 보낼 호스트를 설정하기만 한다. 따라서 connect() 메소드는 블록되지 않고 거의 즉시 반환된다. 여기서는 finishConnect()나 isConnectionPending()와 같은 메소드는 필요가 없다. DatagramSocket이 연결된 경우에만 true를 반환하는 isConnected() 메소드가 있다.  

+ public boolean isConnected()  

이 메소드는 DatagramChannel이 하나의 호스트에만 사용할 수 있도록 제한되어 있는지 여부를 알려 준다. SocketChannel과는 달리 DatagramChannel은 데이터를 전송하거나 수신하기 위해 연결되지는 않는다.  

마지막으로, disconnect() 메소드는 연결을 종료한다.  

+ public DatagramChannel disconnect() throws IOException  

애초에 열린 것이 아무것도 없기 때문에 이 메소드는 실제 아무것도 종료하지 않는다. 다만 이 메소드를 호춯하고 나면 해당 채널은 다른 호스트로 연결될 수 있다.  

##### 7.1.5. 읽기
<br/>
특수 목적의 receive() 메소드 이외에, DatagramChannel는 일반적인 세 개의 read() 메소드를 제공한다.  

+ public int read(ByteBuffer dst) throws IOException
+ public long read(ByteBuffer[] dsts) throws IOException
+ public long read(ByteBuffer[] dsts, int offset, int length) throws IOException  

그러나 이 메소드는 연결된 채널에서만 사용할 수 있다. 즉, 이 메소드 중 하나를 호출하기 전에 connect() 메소드를 호춯하여 채널을 특정 호스트에 연결해야 한다. 이 메소드는, 첫 번째 패킷이 도착하기 전까지는 동신 대상을 알 수 없고, 동시에 다수의 호스트로부터 입력을 받아야 하는 서버보다는 통신해야 할 대상을 미리 알고 있는 클라이언트에게 사용하기가 더 적합하다.  

이 세 개의 메소드 각각은 네트워크로부터 하나의 데이터그램 패킷만을 읽는다. 데이터그램의 데이터로부터 인자로 제공된 ByteBuffer에 저장할 수 있는 만큼만 저장한다. 각 메소드는 읽은 바이트 수를 반환하거나 채널이 닫혔을 때 -1을 반환한다. 그리고 이 메소드는 다음을 포함한 몇몇 이유로 0을 반환할 수도 있다.  

+ 채널이 논블록 모드이고 준비된 패킷이 없는 경우
+ 데이터그램 패킷에 담긴 데이터가 없는 경우
+ 버퍼가 가득 찬 경우  

receive() 메소드와 마찬가지로 데이터그램 패킷이 인자로 전달한 ByteBuffer보다 더 많은 데이터를 가지고 있는 경우, 초과된 데이터는 아무런 알림도 없이 버려진다. BufferOverflowException를 포함한 어떠한 예외도 받을 수 없다.  

##### 7.1.6. 쓰기
<br/>
물론 DatagramChannel 클래스는 모든 쓰기 가능한 스캐터(scatter) 채널에 대해 send() 메소드를 대신해 사용할 수 있는 세 개의 write 메소드를 제공한다.  

+ public int write(ByteBuffer src) throws IOException
+ public long write(ByteBuffer[] dsts) throws IOException
+ public long write(ByteBuffer[] dsts, int offset, int length) throws IOException  

이 메소드는 연결된 채널에서만 사용할 수 있다. 그렇지 않으면 해당 채널은 패킷을 어디로 보내야 할지 알 수가 없다. 각각의 메소드는 연결을 통해 단일 데이터그램 패킷을 전송한다. 이 메소드들은 버퍼의 내용을 완벽하게 쓴다고 보장하지 않는다. 다행히도 버퍼는 커서 기반이기 때문에 버퍼의 내용을 오나전히 전송하기 위해 단순히 반복해서 메소드를 호출하기만 하면 된다. 예를 들어:  

```java
while (buffer.hasRemaining() && channel.write(buffer) != -1);
```

##### 7.1.7. 닫기
<br/>
일반 데이터그램 소켓과 마찬가지로 채널의 사용이 끝나면 관련된 포트와 리소스를 해제하기 위해 닫아 줘야 한다.  

+ public void close() throws IOException  

이미 닫힌 채널을 다시 닫는 경우 아무런 일도 발생하지 않는다. 닫힌 채널에 대해 읽거나 쓰기를 시도할 경우 예외가 발생한다. 채널이 이미 닫혔는지 확실하지 않을 경우 isOpen() 메소드로 확인할 수 있다.  

+ public boolean isOpen()  

채널이 닫힐 경우 이 메소드는 false를 반환하고, 열린 경우 true를 반환한다.  

모든 채널들과 마찬가지로 자바 7에서 DatagramChannel은 AutoCloseable 인터페이스를 구현하므로 try-with-resources 구문을 이용할 수 있다. 자바 7 이전 버전에서는 finally 블록 안에서 채널을 닫을 수 있드며 지금까지 많이 보았던 패턴이다.  

자바 6 이전 버전에서:  

```java
DatagramChannel channel = null;
try {
  channel = DatagramChannel.open();
  // 채널 사용
} catch (IOException ex) {
  // 예외 처리
} finally {
  if (channel != null) {
    try {
      channel.close();
    } catch (IOException ex) {
      // 무시한다.
    }
  }
}
```

그리고 자바 7 이후 버전에서:  

```java
try (DatagramChannel channel = DatagramChannel.open()) {
  // 채널 사용
} catch (IOException ex) {
  // 예외 처리
}
```

##### 7.1.8. 소켓 옵션 // 자바 7
<br/>
자바 7 이후 버전에서, DatagramChannel은 다음 표에 나열된 8가지 소켓 옵션을 제공한다.  

<table>
  <thead>
    <tr>
      <td>옵션</td>
      <td>타입</td>
      <td>상수</td>
      <td>목적</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>SO&#95;SNDBUF</td>
      <td>StandardSocketOptions.SO&#95;SNDBUF</td>
      <td>Integer</td>
      <td>송신 데이터그램 패킷의 버퍼 크기</td>
    </tr>
    <tr>
      <td>SO&#95;RCVBUF</td>
      <td>StandardSocketOptions.SO&#95;RCVBUF</td>
      <td>Integer</td>
      <td>수신 데이터그램 패킷의 버퍼 크기</td>
    </tr>
    <tr>
      <td>SO&#95;REUSEADDR</td>
      <td>StandardSocketOptions.SO&#95;REUSEADDR</td>
      <td>Boolean</td>
      <td>주소 재사용 On/Off</td>
    </tr>
    <tr>
      <td>SO&#95;BROADCAST</td>
      <td>StandardSocketOptions.SO&#95;BROADCAST</td>
      <td>Boolean</td>
      <td>브로드캐스트 메시지 On/Off</td>
    </tr>
    <tr>
      <td>IP&#95;TOS</td>
      <td>StandardSocketOptions.IP&#95;TOS</td>
      <td>Boolean</td>
      <td>트래픽 클래스</td>
    </tr>
    <tr>
      <td>IP&#95;MULTICAST&#95;IF</td>
      <td>StandardSocketOptions.IP&#95;MULTICAST&#95;IF</td>
      <td>NetworkInterface</td>
      <td>멀티캐스트를 사용하기 위한 로컬 네트워크 인터페이스</td>
    </tr>
    <tr>
      <td>IP&#95;MULTICAST&#95;TTL</td>
      <td>StandardSocketOptions.IP&#95;MULTICAST&#95;TTL</td>
      <td>Integer</td>
      <td>멀티캐스트 데이터그램의 TTL 값</td>
    </tr>
    <tr>
      <td>IP&#95;MULTICAST&#95;LOOP</td>
      <td>StandardSocketOptions.IP&#95;MULTICAST&#95;LOOP</td>
      <td>Boolean</td>
      <td>멀티캐스트 데이터그램의 루프백 On/Off</td>
    </tr>
  </tbody>
</table>
<br/><br/>

이 옵션들은 다음 세 개의 메소드를 사용하여 설정하거나 확인할 수 있다.  

+ public <T> DatagramChannel setOption(SocketOption<T> name, T value) throws IOException
+ public <T> T getOption(SocketOption<T> name) throws IOException
+ public Set<SocketOption<?>> supportedOptions()  

supportedOptions() 메소드는 사용 가능한 소켓 옵션을 나열한다. getOption() 메소드는 이 옵션들의 현재 설정된 값을 알려 준다. 그리고 setOption()을 사용하여 이 옵션들을 변경할 수 있다. 예를 들어, 브로드캐스트 메시지를 전송한다고 가정해 보자. SO&#95;BROADCAST 옵션이 기본적으로 꺼져 있지만, 아래와 같이 이 옵션을 켤 수 있다.
