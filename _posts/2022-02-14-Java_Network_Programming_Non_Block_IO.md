---
title:  논블록 I/O
categories:
- Java Network Programming
feature_text: |
  ## 논블록 I/O
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

네트워크의 속도는 CPU, 메모리 심지어 디스크와 비교해도 매우 느리다. 요즘 최신 사양의 PC는 약 6Gbps의 속도로 CPU와 메모리 사이에 데이터가 이동한다. 그리고 디스크와의 데이터 이동은 이보다 훨씬 느리지만, 약 150Mbps의 속도로 여전히 훌륭한 속도가 보장된다. 이 수치는 이론적인 최대값이며, 메가바이트(MB)는 1,024×1,204를 의미하고 기가바이트(GB)는 1,024메가바이트를 의미한다. 하드웨어 제조사들은 종종 좀 더 간결하게 표현하기 위해 기가바이트의 크기를 1,000메가바이트로 줄이거나 메가바이트를 1,000,000바이트로 줄이기도 한다. 게다가 네트워킹 속도는 초당 바이트가 아닌 초당 비트 단위를 사용한다. 반면에 대부분의 LAN이 이론적인 수치보다 10배에서 100배 정도 느리게 동작하지만, 오늘날 가장 빠른 LAN의 이론적인 최고 속도는 150Mbps에 달한다. 그리고 공용 인터넷을 통과하는 속도는 LAN에서 이론적인 최고 속도는 150Mbps에 달한다. 그리고 공용 인터넷을 통과하는 속도는 LAN에서 경험할 수 있는 속도보다 많게는 약 10배까지 느리다.  

CPU가 느린 네트워크를 기다리지 않고 네트워크보다 앞서 달리게 하기 위한 전통적인 자바의 해결 방안은 버퍼링과 멀티스레드를 결합하는 것이다. 다수의 스레드가 동시에 다수의 서로 다른 연결을 통해 보낼 데이터를 생성한다. 그리고 네트워크가 데이터를 보낼 준비가 될 때까지 해당 데이터들을 버퍼에 저장해 둔다. 이 접근 방법은 고성능이 필요하지 않은 단순한 서버와 클라이언트 환경에서는 잘 동작한다. 그러나 멀티스레드를 생성할 때 드는 오버헤드와 스레드 전환 시 발생하는 오버헤드를 무시할 수 없다. 그리고 각각의 스레드는 약 1메가바이트의 메모리를 여분으로 필요로 한다. 초당 수천 개의 요청을 처리하는 대규모 서버 환경에서는 스레드가 사용하는 여분의 메모리와 다양한 오버헤드로 인해 연결마다 스레드를 할당하는 것이 쉽지 않다. 하나의 스레드가 다수의 연결을 담당하고, 데이터를 수신할 준비가 된 연결을 골라내서 처리하고, 그리고 다시 준비된 다음 연결을 골라내는 방법을 반복한다면 훨씬 더 빠를 것이다.  

이 방식이 잘 동작하기 위해서는 운영체제의 지원이 필요하다. 다행히도 요즘 출시되는 대부분의 운영체제들은 이러한 nonblock(non-blocking) I/O를 지원한다. 그러나 태블릿이나 휴대폰과 같은 일부 클라이언트 시스템에서는 제대로 지원되지 않을 수도 있다. 사실 이러한 기능을 제공하는 java.nio 패키지는 안드로이드에서도 제공되지만, 현재 자바 ME 프로파일의 일부가 아니며 앞으로도 그럴 계획이 없다. 하지만 이런 새로운 I/O API는 오직 서버 프로그램을 위해서 설계되었으먀, 서버에서만 문제가 되지 때문에 서버에 관해서 이야기하기 전에는 넌지시 언급만할 뿐 자세히 다루지는 않았다. 클라이언트나 P2P 시스템에서는 동시에 매우 많은 연결을 멀티스레드도 처리할 필요가 없고, 스트림 기반의 I/O가 프로세스 병목현상의 주요 원인이 된다.  

+ 자바의 NIO 지원  
잘 설계된 논블록 I/O가 멀티스레드, 멀티프로세스 설계 방식보다 극적으로 성능을 능가하던 시절이 있었다. 그 시절이 1990년대였다. 불행히도 자바는 2002년에 나온 1.4 버전 이전까지 논블록 I/O를 지원하지 않았다. 자바 5가 배포되던 2004년과 자바 6이 배포되던 2006년 무렵까지 운영체제의 네이티브 스레드가 지속적으로 개선되면서 컨텍스트 스위치와 동기화의 거의 모든 오버헤드가 제거됐다. 게다가 서버 메모리가 1만 개의 스레드를 동시에 실행할 수 있을 만큼 늘어났다. 그리고 멀티코어/멀티CPU 시스템을 최대로 활용하기 위해 멀티 스레드에 대한 요구가 일반화됐다. 또한 최신 자바 7과 자바 8의 64비트 가상 머신에서는 더 나은 멀티스레드 성능을 보여 준다. 최근에는 복잡한 NIO 기반의 구조가 단순한 스레드 기반의 구조보다 낫다고 말하기가 더욱 어려워졌다.  

그리고 NIO가 더 빠르지 않을까? 반드시 그런 것은 아니다. 리눅스에서 자바 6으로 실제 측정해 보면 전통적인 멀티스레드 I/O 설계 방식이 NIO 방식보다 30퍼센트 이상 성능이 뛰어나다.  

전통적인 I/O 방식보다 비동기 I/O 방식이 더 나은 성능을 보여 주는 상황이 있지 않을까? 아마도 있을 것이다. 한 가지 상황으로, 약 1만+의 연결을 동시에 오랫동안 유지하고 각각의 클라이언트가 작은 데이터를 드물게 전송하는 서버를 생각해 볼 수 있다. 이러한 예로, 전국적인 체인을 가진 편의점에 설치된 현금 등록기에서 보내는 거래 정보를 수집하는 서버를 생각해 볼 수 있다. 이 시나리오는 NIO를 위해 생각해 낸 것이다. 그리고 멀티코어/멀티CPU에 맞게 스레드를 활용하여 비동기 요청을 처리한다면 좀 더 효과적인 구현을 할 수 있을 것이다.  

그러나 항상 최적화에 대한 다음 두 가지 황금 규칙을 기억하도록 하자.  

(1) 최적화를 하지 말 것.
(2) (전문가에 한해서) 문제를 명확히 하고, 문제를 측정하고, 분명한 문제 해결 방법이 맞는지 확인 후에 시행할 것.  

### 1. 예제 클라이언트
<br/>
새로운 I/O API를 이용하는 클라이언트를 구현할 때, 새로운 java.nio.channels.SocketChannel 객체를 만들기 위해 정적 팩토리 메소드인 SocketChannel.open()을 호출하는 것으로 시작한다. 이 메소드를 호출할 때 인자로 연결할 호스트와 포트를 나타내는 java.net.SocketAddress 객체를 전달한다. 예를 들어, 다음 코드는 rama.poly.edu의 포트 19번에 채널을 연결한다.  

```java
SocketAddress rama = new InetSocketAddress("rama.poly.edu", 19);
SocketChannel client = SocketChannel.open(rama);
```

채널은 블록 모드(block mode)로 열리기 때문에 이 코드의 다음 줄은 연결이 성립되기 전까지 실행될 수 없다. 연결이 성립될 수 없는 경우 IOException 예외가 발생한다. 전통적인 클라이언트라면 이 시점에서 소켓의 입출력 스트림을 요청할 것이다. 그러나 여기서는 그렇게하지 않는다.  

채널을 사용하여 채널 자체에 직접 데이터를 쓸 수 있다. 바이트 배열 대신 ByteBuffer 객체를 써야 한다. 아스키 문자 74개를 문자열로 만들고자 한다면 (72개의 출력 가능한 문자와 캐리지리턴/라인피드), 정적 메소드인 allocate()를 사용하여 74바이트 용량의 ByteBuffer을 만들 수 있다.  

```java
ByteBuffer buffer = ByteBuffer.allocate(74);
```

이 ByteBuffer 객체를 채널의 read() 메소드에 전달한다. 채널은 소켓으로부터 읽은 데이터로 이 버퍼를 채운다. 그리고 이 메소드는 성공적으로 읽어서 버퍼에 저장한 바이트의 수를 반환한다.  

```java
int bytesRead = client.read(buffer);
```

기본적으로 이 코드는 InputStream과 마찬가지로 최소 1바이트를 읽거나 더 이상 읽을 데이터가 없는 경우 -1을 반환한다. 그리고 읽을 추가적인 바이트가 있는 경우 종종 더 읽기도 한다. 논블록(nonblock) 모드에서는 읽을 데이터가 없는 경우 즉시 0을 반환하며 논블록 모드로 진입하는 방법에 대해서는 곧 다룰 예정이다. 그러나 지금은 우선 InputStream처럼 블록(block) 모드로 동작한다. 이 메소드 또한 데이터를 읽다가 문제가 생기면 IOException 예외를 발생시킨다.  

버퍼에 데이터가 있다고 가정해 보자. 즉, n > 0인 경우 이 데이터를 System.out으로 출력할 수 있다. ByteBuffer로부터 바이트 배열을 뽑아낸 다음 System.out 같은 전통적인 OutputStream에 출력하는 방법이 있다. 그러나 순수 채널 기반의 방식을 사용하면 더 많은 정보를 얻을 수 있다. 순수 채널 방식을 사용할 경우 OutputStream인 System.out을 채널 유틸리티 클래스의 newChannel() 메소드를 사용하여 채널로 감싸 줘야 한다.  

```java
WritableByteChannel output = Channels.newChannel(System.out);
```

그런 다음에 읽은 데이터를 System.out으로 연결된 출력 채널로 쓸 수 있다. 그러나 쓰기 전에 출력 채널이 버퍼 데이터의 끝 위치가 아닌 처음 위치에서 읽을 수 있도록 해당 버퍼에 대해 먼저 flip() 메소드를 호출해야 한다.  

```java
buffer.flip();
output.write(buffer);
```

이때 쓸 수 있는 바이트의 양을 출력 채널에 알려 주지 않아도 된다. 버퍼는 자신이 얼마만큼의 데이터를 가지고 있는지 알고 있다. 그러나 일반적으로 출력 채널은 버퍼에 있는 모든 바이트를 쓴다고 보장하지 않는다. 흔하지 않지만 이러한 경우에 나머지 데이터를 계속해서 쓰거나 IOException 예외를 발생시킨다.  

읽기와 쓰기를 위해 매번 새로운 버퍼를 생성하지 않도록 해야 한다. 매번 새로운 버퍼를 생성할 경우 시스템 성능이 저하된다. 대신에 이미 생성된 버퍼를 재사용할 수 있다. 기존 버퍼에 새로운 데이터를 읽기 전에 기존 데이터를 지워 줘야 한다.  

```java
buffer.close();
```

clear() 메소드는 flip() 메소드와 약간 다르다. flip()은 버퍼에 데이터를 온전하게 남겨 두고, 해당 버퍼에 데이터를 읽기 위한 준비가 아닌 해당 버퍼의 데이터를 쓰기 위한 준비를 한다. clear()는 해당 버퍼를 초기 상태로 되돌린다. (실제로는 조금 단순하게 처리된다. 이전 데이터가 여전히 남아 있다. 강제로 덮어 쓰진 않지만, 새로운 데이터를 읽으면 곧바로 덮어 쓰인다.)  

```java
import java.nio.*;
import java.nio.channels.*;
import java.net.*;
import java.io.IOException;

public class ChargenClient {
  public static int DEFAULT_PORT = 19;

  public static void main(String[] args) {

    if (args.length == 0) {
      System.out.println("Usage: java ChargenClient host [post]");
      return;
    }

    int port;
    try {
      port = Integer.parseInt(args[1]);
    } catch (RuntimeException ex) {
      port = DEFAULT_PORT;
    }

    try {
      SocketAddress address = new InetSocketAddress(args[0], port);
      SocketChannel client = SocketChannel.open(address);
      ByteBuffer buffer = ByteBuffer.allocate(74);
      WritableByteChannel out = Channels.newChannel(System.out);

      while (client.read(buffer) != -1) {
        buffer.flip();
        out.write(buffer);
        buffer.clear();
      }
    } catch (IOException ex) {
      ex.printStackTrace();
    }
  }
}
```

클라이언트가 모든 입력을 출력으로 복사하는 것 이외에 추가적인 다른 어떤 작업을 하고자 할 경우 이 새로운 API가 필요하다. 연결은 블록 모드 또는 논블록 모드로 실행될 수 있으며, 논블록 모드에서 read() 메소드는 읽을 데이터가 없을 때 즉시 반환된다. 이렇게 되면 프로그램은 느린 연결로부터 데이터를 읽기 위해 기다리지 않아도 되며 추가적인 다른 일을 할 수 있게 된다. configureBlocking() 메소드 호출 시 인자를 true로 전달하면 블록 모드가 설정되고, false를 전달하면 논블록 모드가 된다. 다음은 연결을 논블록 모드로 변경하는 코드다.  

```java
client.configureBlocking(false);
```

논블록 모드에서 read() 메소도는 읽을 데이터가 없는 경우 즉시 0을 반환하기 때문에, 블록 모드와는 다르게 loop의 사용이 필요하다.  

### 2. 예제 서버
<br/>
채널과 버퍼는 클라이언트 프로그램에서도 충분히 잘 동작하지만, 많은 동시 연결을 효과적으로 처리해야 하는 서버 시스템을 위해 만들어졌다. 서버를 다루기 위해서는 클라이언트에서 사용된 버퍼와 채널 이외에도 한 가지 요소가 더 필요하다. 구체적으로 말하면 서버가 출력을 받거나 입력을 보낼 준비가 된 모든 연결을 찾을 수 있도록 도와주는 셀렉터(selector)가 필요하다.  

이 내용을 확인하기 위해 문자 발생기 프로토콜을 위한 간단한 서버를 구현해 보자. 먼저 새로운 I/O API를 이용하는 서버를 구현하기 위해 정적 팩토리 메소드인 ServerSocketChannel.open()를 호출하여 새로운 ServerSocketChannel 객체를 만들어야 한다.  

```java
ServerSocketChannel serverChannel = ServerSocketChannel.open();
```

처음에 이 채널은 실제 어떤 포트에도 바인드되지 않는다. 이 채널을 포트에 바인드하기 위해서는 채널로부터 socket() 메소드를 호출하여 ServerSocket 객체를 얻은 다음 bind()를 호출한다. 예를 들어, 다음 코드는 채널을 포트 19번의 서버 소켓에 바인드한다.  

```java
ServerSocket ss = serverChannel.socket();
ss.bind(new InetSocketAddress(19));
```

자바 7 이후 버전에서는 내장된 java.net.ServerSocket을 구하지 않고 직접 바인드할 수 있다.  

```java
serverChannel.bind(new InetSocketAddress(19));
```

일반적인 서버 소켓과 마찬가지로 19번 포트에 바인드하려면 유닉스(리눅스, 맥 OS X 포함)의 경우 루트 권한이 필요하다. 루트 권한이 없는 사용자는 1024 이상의 포트에만 바인드할 수 있다.  

서버 소켓 채널은 이제 포트 19번에서 들어오는 연결을 대기하고 있다. 이제 연결을 수용하기 위해서 accept() 메소드를 호출해야 하며 이 메소드는 SocketChannel 객체를 반환한다.  

```java
SocketChannel clientChannel = serverChannel.accept();
```

그리고 서버에서는 다수의 동시 연결을 허용할 수 있도록 클라이언트 채널을 논블록(nonblock)으로 만들어야 한다.  

```java
clientChannel.configureBlocking(false);
```

ServerSocketChannel 역시 논블록으로 만들어야 한다. 기본적으로 ServerSocketChannel의 accept()는 ServerSocket의 accept()처럼 들어온 연결이 있을 때까지 블록된다. 이 설정을 변경하기 위해서 accept()를 호출하기 전에 아래와 같이 false를 인자로 configureBlocking() 메소드를 호출한다.  

```java
serverChannel.configureBlocking(false);
```

nonblock.accept()는 들어오는 연결이 없는 경우 거의 즉시 널(null)을 반환한다. 이때 소켓을 사용하기 전에 반환된 소켓을 검사하여 NullPointerException 예외가 발생하지 않도록 주의해야 한다.  

이제 두 개의 열린 채널이 있다. 서버 채널과 클라이언트 채널. 이 두 채널을 모두 처리해야 한다. 그리고 이 두 채널은 명확하지 않은 시점에 실행된다. 게다가 서버 채널을 처리하면 계속해서 열린 클라리언트 채널이 생성된다. 전통적인 접근 방법으로는 각 연결에 스레드를 할당하여 처리할 수 있다. 히자만 그렇게 처리할 경우 스레드의 수는 클라이언트의 연결 수만큼 빠르게 증가한다. 새로운 I/O API에서는 대신 셀렉터를 이용하여 프로그램이 처리할 준비가 된 연결만을 찾아서 반복적으로 처리할 수 있다. 새로운 셀렉터를 생성하기 위해 정적 팩토리 메소드인 Selector.open()를 호춣한다.  

```java
Selector selector = Selector.open();
``

다음으로 각 채널을 셀렉터가 감시할 수 있도록 채널의 register() 메소드를 이용하여 셀렉터를 등록한다. 등록할 때 SelectionKey 클래스에 정의된 명명 상수를 사용하여 관심 있는 동작을 명시한다. 서버 소켓의 경우에는 관심 동작이 OP&#95;ACCEPT 하나밖에 없다. 즉, 서버 소켓 채널이 새로운 연결을 수용할 준비가 되었는지 묻는다.  

```java
serverChannel.register(selector, SelectionKey.OP_ACCEPT);
```

서버 채널과는 달리 클라이언트 채널은 채널에 데이터를 쓸 준비가 되었는지에 관심이 있으므로 OP&#95;WRITE 키를 사용한다.  

```java
SelectionKey key = clientChannel.register(selector, SelectionKey.OP_WRITE);
```

위 두 예제의 register() 메소드는 SelectionKey 객체를 반환한다. 하지만 여기서는 클라이언트 채널을 위한 키만 사용한다. 각각의 SelectionKey는 임의의 Object 타입을 첨부하고 있다. 이 Object는 일반적으로 연결의 현재 상태를 나타내는 객체를 보관하는 데 사용된다. 이 경우에는 채널이 네트워크에 쓰기 위한 버퍼를 저장하는 데 사용한다. 그리고 이 버퍼는 모두 비워진 다음 다시 채울 수 있으며, 각 버퍼로 복사될 데이터로 배열을 채운다.  

버퍼는 다음 줄을 쓰기 위해, 버퍼의 끝에 쓰기보다는 버퍼의 시작 부분으로 돌아와서 다시 쓴다. 두 개의 연속적인 데이터의 복사본을 저장한 배열을 만들어 두면, 전송할 시작 위치에 상관없이 모든 라인을 배열에서 연속적으로 이용할 수 있기 때문에 코드를 작성하기가 훨씬 쉽다.  

```java
byte[] rotation = new byte[95*2];
for (byte i = ' '; i <= '-'; i++) {
  rotation[i - ' '] = i;
  rotation[i + 95 - ' '] = i;
}
```

이 배열은 초기화된 이후에 읽기 용도로만 사용할 것이기 때문에, 다수의 채널에서 함께 사용할 수 있다. 각가의 채널은 이 배열의 데이터를 사용해 자신들의 버퍼를 채운다. 즉, rotation 배열의 처음 72바이트를 사용해 버퍼를 채운 다음, 라인을 구분하기 위한 캐리지리턴/라인피드를 추가한다. 다음으로 전송 준비를 위해 버퍼에 대해 flip() 메소드를 호출하고, 버퍼를 채널의 키에 첨부한다.  

```java
ByteBuffer buffer = ByteBuffer.allocate(74);
buffer.put(rotation, 0, 72);
buffer.put((byte) '\r');
buffer.put((byte) '\n');
buffer.flip();
key2.attach(buffer);
```

동작 준비가 된 채널이 있는지 확인하기 위해 셀렉터의 select() 메소드를 호출한다. 장시간 실행되는 서버의 경우, 일반적으로 select() 코드를 무한 루프 안에 위치시킨다.  

```java
while (true) {
  selector.select();
  // 셀렉트된 키를 처리한다.
}
```

셀렉터가 준비된 채널을 찾은 경우 셀렉터의 selectedKeys() 메소드는 준비된 채널당 하나의 SelectionKey를 포함하고 있는 java.util.Set을 반환한다. 준비된 채널이 없는 경우 빈 java.util.Set이 반환된다. 두 경우 모두, 반환된 Set과 java.util.Iterator를 사용하여 루프를 돈다.  

```java
Set<SelectionKey> readyKeys = selector.selectorKeys();
Iterator iterator = readyKeys.iterator();
while (iterator.hasNext()) {
  SelectionKey key = iterator.next();
  // 두 번 처리하지 않도록 처리한 키를 세트에서 제거한다.
  iterator.remove();
  // 채널에 필요한 작업을 수행한다.
}
```

세트에서 키를 제거함으로써 셀렉터에게 해당 키의 처리가 끝났음을 알리고 select() 호출시 다시 반환되지 않도록 한다. 셀렉터는 해당 채널이 다시 읽을 준비가 된 다음, select() 메소드가 호출될 때, 제거된 채널을 다시 준비된 채널에 추가하여 반환한다. 그러므로 사용이 끝난 채널은 준비된 채널에서 꼭 제거해야 한다.  

준비된 채널이 서버 채널인 경우, 프로그램은 새로운 소켓 채널을 수용하고 수용된 소켓 채널을 셀렉터에 추가한다. 준비된 채널이 소켓 채널인 경우, 프로그램은 버퍼가 허용할 수 있는 만큼 채널에 쓴다. 준비된 채널이 없는 경우, 셀렉터는 준비된 채널이 생길 때까지 기다린다. 결국 하나의 스레드, 즉 메인 스레드는 다수의 동시 연결을 처리하게 된다.  

이 경우에 서버 채널은 연결을 수용할 준비만 하고 클라이언트 채널은 쓸 준비만 하기 때문에, 클라이언트나 서버 채널 중에 어떤 채널이 선택되었는지 쉽게 구분할 수 있다. 그리고 이 둘 모두 I/O 연산이기 때무에, 다양한 이유로 IOException 예외가 발생할 수 있다. 그러므로 원한다면 이 코드를 try 블록으로 감쌀 수 있다.  

```java
try {
  if (key.isAcceptable()) {
    ServerSocketChannel server = (ServerSocketChannel) key.channel();
    SocketChannel connection = server.accept();
    connection.configureBlocking(false);
    connection.register(selector, SelectionKey.OP_WRITE);
    // 클라이언트 버퍼를 설정한다.
  } else if (key.isWritable()) {
    SocketChannel client = (SocketChannel) key.channel();
    // 클라이언트에게 데이터를 쓴다.
  }
}
```

채널에 데이터를 쓰는 것은 어렵지 않다. 키에 첨부된 버퍼를 구해서 ByteBuffer로 캐스팅하고 버퍼에 아직 쓰지 않은 데이터가 남아 있는지 확인하기 위해 hasRemaining() 메소드를 호출한다. 남아 있는 데이터가 있는 경우 출력하고, 남아 있는 데이터가 없는 경우 회전하는 배열에 있는 데이터의 다음 줄로 버퍼를 채우고 채워진 버퍼를 쓴다.  

```java
ByteBuffer buffer = (ByteBuffer) key.attachment();
if (!buffer.hasRemaining()) {
  // 이전에 보낸 줄의 시작 문자를 찾고
  // 다음에 전송할 줄로 버퍼를 채운다.
  buffer.rewind();
  int first = buffer.get();
  // 다음 시작 문자로 하나 증가시킨다.
  buffer.rewind();
  int position = first - ' ' + 1;
  buffer.put(rotation, position, 72);
  buffer.put((byte) '\r');
  buffer.put((byte) '\n');
  buffer.flip();
}
client.write(buffer);
```

다음 줄을 어디서부터 복사해야 하는지 찾는 알고리즘은 아스키 문자를 순서대로 저장하고 있는 rotation 배열을 사용한다. 이 알고리즘은 먼저 buffer.get() 메소드를 사용하여 버퍼로부터 데이터의 시작 1바이트를 읽는다. 그리고 rotation 배열의 첫 문자로 사용된 스페이스 문자에서 읽은 1바이트를 뺀다. 이 결과 나온 값은 현재 버퍼의 시작 위치에 해당하는 배열의 인덱스를 알려 준다. 그리고 이 값에 1을 더하여 다음 줄의 시작 위치를 결정하고 버퍼를 채운다.  

```java
import java.nio.*;
import java.nio.channels.*;
import java.net.*;
import java.util.*;
import java.io.IOException;

public class ChargenServer {

  public static int DEFAULT_PORT = 19;

  public static void main(String[] args) {

    int port;
    try {
      port = Integer.parseInt(args[0]);
    } catch (RuntimeException ex) {
      port = DEFAULT_PORT;
    }
    System.out.println("Listening for connections on port " + port);

    byte[] rotation = new byte[95*2];
    for (byte i = ' '; i <= '-'; i++) {
      rotation[i - ' '] = i;
      rotation[i + 95 - ' '] = i;
    }

    ServerSocketChannel serverChannel;
    Selector selector;
    try {
      serverChannel = ServerSocketChannel.open();
      ServerSocket ss = serverChannel.socket();
      InetSocketAddress address = new InetSocketAddress(port);
      ss.bind(address);
      serverChannel.configureBlocking(false);
      selector = Selector.open();
      serverChannel.register(selector, SelectionKey.OP_ACCEPT);
    } catch (IOException ex) {
      ex.printStackTrace();
      return;
    }

    while (true) {
      try {
        selector.select();
      } catch (IOException ex) {
        ex.printStackTrace();
        break;
      }

      Set<Selection> readyKeys = selector.selectKeys();
      Iterator<SelectionKey> iterator = readyKeys.iterator();
      while (iterator.hasNext()) {
        SelectionKey key = iterator.next();
        iterator.remove();
        try {
          if (key.isAcceptable()) {
            ServerSocketChannel server = (ServerSocketChannel) key.channel();
            SocketChannel client = server.accept();
            System.out.println("Accepted connection from " + client);
            client.configureBlocking(false);
            SelectionKey key2 = client.register(selector, SelectionKey.OP_WRITE);
            ByteBuffer buffer = ByteBuffer.allocate(74);
            buffer.put(rotation, 0, 72);
            buffer.put((byte) '\r');
            buffer.put((byte) '\n');
            buffer.flip();
            key2.attach(buffer);
          } else if (key.isWritable()) {
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer buffer = (ByteBuffer) key.attachment();
            if (!buffer.hasRemaining()) {
              // 다음 줄로 버퍼를 채운다.
              buffer.rewind();
              // 이전에 보낼 줄의 시작 문자를 구한다.
              int first = buffer.get();
              // 버퍼의 데이터를 변경할 준비를 한다.
              buffer.rewind();
              // rotation에서 새로운 시작 문자의 위치를 찾는다.
              int position = first - ' ' + 1;
              // rotation에서 buffer로 데이터를 복사한다.
              buffer.put(rotation, position, 72);
              // 버퍼의 마지막에 라인 구분자를 저장한다.
              buffer.put((byte) '\r');
              buffer.put((byte) '\n');
              // 버퍼를 출력할 준비를 한다.
              buffer.flip();
            }
            client.write(buffer);
          }
        } catch (IOException ex) {
          key.cancel();
          try {
            key.channel().close();
          } catch (IOException cex) P{}
        }
      }
    }
  }
}
```

최대 성능을 내기 위해 멀티스레드를 사용하는 것이 중요하다. 멀티스레드를 이용하면 서버가 멀티 CPU를 충분히 활용할 수 있다. 단일 CPU 환경에서조차 작업을 처리하는 스레드와 수용(accpet) 스레드를 분리하는 것은 좋은 방법이다. 스레드 풀은 새로운 I/O 모델에서도 여전히 잘 활용할 수 있다. 들어오는 연결을 수용하는 스레드는 수용된 연결을 풀에 있는 스레드가 처리할 수 있도록 큐에 추가한다.  

이렇게 하면 select() 메소드가 데이터를 수신할 준비가 완료된 연결을 선택해 주므로 셀렉터 없이 동작하는 것보다 훨씬 빠르다. 이 방법은 셀렉터가 수신할 데이터가 준비되지 않은 연결에 대해 시간을 낭비하지 않도록 보장해 주기 때문에, 동일한 작업을 셀렉터 없이 하는 것보다 여전히 더 빠르게 동작한다. 반면에 동기화 이슈가 발생할 수 있으므로 병목지점이 없다는 것을 증명한 후에 사용해야 한다.  

### 3. 버퍼
<br/>
스트림을 항상 버퍼링할 것을 권장했다. 충분히 큰 버퍼의 사용은 네트워크 프로그램의 성능에 가장 큰 영향을 미치는 부분이다. 새로운 I/O 모델에서는 더 이상 버퍼의 사용을 선택할 수 없다. 모든 I/O가 버퍼링된다. 게다가 버퍼는 새로운 I/O API의 기초를 이루고 있다. 데이터를 입출력 스트림으로 쓰거나 읽지 않고 대신 버퍼에 쓰고 읽는다. 버퍼는 버퍼링 스트림에서와 같이 단순한 바이트 배열로도 표현될 수 있다. 하지만 버퍼의 네이티브 구현에서는 버퍼를 하드웨어나 메모리에 직접 연결하거나, 아니면 매우 효율적인 다른 구현들을 이용하기도 한다.  

프로그래밍 관점에서 스트림과 채널의 가장 큰 차이점은 채널은 블록 기반인 데 반해 스트림은 바이트 기반이다. 스트림은 순서대로 한 바이트씩 제공하도록 설계되었다. 단지 성능을 위해 바이트 배열을 전달할 수는 있지만, 기본 개념은 한 번에 1바이트의 데이터를 전달하는 것이다. 반면에 채널은 버퍼 안에 있는 데이터의 블록을 전달한다. 바이트는 채널에서 읽고 쓰기 전에 먼저 버퍼에 저장되어야 한다. 그리고 데이터는 한 번에 버퍼씩 읽고 쓴다.  

스트림과 채널/버퍼의 두 번째 큰 차이점은 채널과 버퍼는 같은 객체에 대해서 읽기와 쓰기 모두를 지원하는 경향이 있다. 하지만 항상 그런 것은 아니다. 예를 들어, CD-ROM에 있는 파일을 가리키는 채널은 읽을 수는 있지만 쓸 수는 없다. 입력이 닫힌 소켓에 연결된 채널의 경우 쓸 수는 있지만 읽을 수는 없다. 만약 읽기만 할 수 있는 채널을 쓰려고 하거나, 쓸 수만 있는 채널을 읽으려고 하면, UnsupportedOperationException 예외가 발생한다. 하지만 네트워크 프로그램은 대체로 같은 채널에서 읽기와 쓰기가 모두 가능하다.  

버퍼의 내부적인 자세한 구현 방식을 몰라도(자세한 구현 방식은 구현에 따라 다르며, 호스트 운영체제의 하드웨어에 최적화된다). 배열과 같이 일반적인 기본 데이터 타입을 요소로 가지는 고정된 크기의 목록으로 생각할 수 있다. 그러나 실제 구현이 배열일 필요는 없다. 배열인 경우도 있고 아닌 경우도 있다. 자바의 기본 데이터 타입 중에서 불리언(boolean)을 제외한 나머지 모든 타입에 대해 Buffer의 서브클래스가 존재한다. ByteBuffer, CharBuffer, ShortBuffer, IntBuffer, LongBuffer, FloatBuffer, DoubleBuffer, 각 서브클래스에서 제공되는 메소드는 적절한 타입에 맞는 인자와 반환값을 가진다. 예를 들어, DoubleBuffer 클래스는 double 타입을 인자로 전달하거나 반환하는 메소드를 제공한다. IntBuffer 클래스는 double 타입을 인자로 전달하거나 반환하는 메소드를 제공한다. IntBuffer 클래스는 int 타입을 인자로 전달하거나 반환하는 메소드를 제공한다. 공통의 Buffer 슈퍼클래스는 버퍼에 포함된 데이터의 타입을 알 필요가 없는 메소드만을 제공한다. (타입을 인식하지 않는 것이 여기서는 문제가 되기도 한다.) 네트워크 프로그램의 경우 비록 가끔씩 ByteBuffer를 다른 타입으로 해석(overlay)하여 사용하는 프로그램이 있기도 하지만, 거의 항상 ByteBuffer를 사용한다.  

+ 위치(position)  
버퍼에서 읽거나 쓸 다음 위치를 나타낸다. 이 값은 0에서 시작하여 최대값은 버퍼의 크기와 같다. 이 값은 다음 두 메소드를 사용하여 설정하거나 가져올 수 있다.  

> + public final int position()<br/>
+ public final Buffer position(int newPosition)  

+ 용량(capacity)  
버퍼가 보유할 수 있는 최대 요소들의 수, 이 값은 버퍼가 생성될 때 설정되며 그 후로 변경할 수 없다. 다음 메소드를 사용하여 이 값을 읽을 수 있다.  

> + public final int capacity()  

+ 한도(limit)  
버퍼에서 접근할 수 있는 데이터의 끝. 버퍼의 용량이 더 남아 있더라도 한도 지점을 변경하기 전에는 한도를 넘어서서 읽거나 쓸 수 없다. 이 값은 다음 두 메소드를 사용하여 가져오거나 설정한다.  

> + public final limit()<br/>
+ public final Buffer limit(int newLimit)  

+ 표시(mark)  
버퍼에서 클라이언트에 제한된 인덱스 mark() 메소드를 호출하면 현재 위치에 표시가 설정되며, reset() 메소드를 호출하면 현재 위치가 표시된 위치로 설정된다.  

> + public final Buffer mark()<br/>
+ public final Buffer reset()  

위치(position)가 설정된 표시(mark) 이하로 설정되면, 표시는 버려진다.  

InputStream에서 읽는 것과는 달리 버퍼로부터 읽으면 실제로 버퍼 안에 있는 데이터는 어떤 식으로든 변경되지 않는다. 버퍼의 이러한 특성으로 인해 버퍼의 위치를 앞뒤로 이동하여 특정 위치에서부터 읽는 것이 가능하다. 마찬가지로 프로그램은 읽을 수 있는 데이터의 끝을 제어하기 위해 한도(limit)를 조정할 수 있다. 오직 용량만이 고정되어 있다.  

공통의 Buffer 슈퍼클래스는 또한 이들 공통 속성들을 참조하여 동작하는 몇몇 다른 메소드를 제공한다.  

clear() 메소드는 위치를 0으로 설정하고 한도를 용량으로 설정하여 버퍼를 비운다. 이렇게 함으로써 버퍼는 완전히 새롭게 채워질 수 있다.  

+ public final Buffer clear()  

그러나 clear() 메소드는 버퍼로부터 이전 데이터를 제거하지 않는다. 이전 데이터는 여전히 남아 있으며 절대적인 get() 메소드를 사용하거나 한도와 위치를 다시 변경하여 읽을 수 있다.  

rewind() 메소드는 위치를 0으로 설정하지만, 한도를 변경하지는 않는다.  

+ public final Buffer rewind()  

이 메소드는 버퍼를 다시 읽을 수 있도록 한다.  

flip() 메소드는 한도를 현재 위치로 설정하고 위치를 0으로 설정한다.  

+ public final Buffer flip()  

이 메소드는 버퍼를 채운 다음 버퍼의 내용을 내보내야 할 때 호출된다.  

마지막으로, 버퍼의 내용은 변경하지 않지만 버퍼에 관한 정보를 반환하는 두 개의 메소드를 제공한다. remaining() 메소드는 버퍼에서 현재 위치와 한도 사이에 있는 요소의 수를 반환한다. hasRemaining() 메소드는 남아 있는 요소의 수가 0보다 큰 경우 true를 반환한다.  

+ public final int remaining()
+ public final boolean hasRemaining()  

### 4. 버퍼 만들기
<br/>
버퍼 클래스의 계층 구조는 적어도 최상위 레벨이 아니고서는 다형성이 아닌 상속성에 기반을 두고 있다. 여러분은 보통 여러분이 다루고 있는 것이 IntBuffer인지, ByteBuffer인지, CharBuffer인지, 또는 다른 무엇인지 알 필요가 있다. 여러분은 공통의 Buffer 슈퍼클래스가 아닌 이러한 서브클래스 중 하나로 코드를 작성한다.  

각 타입별 버퍼 클래스는 다양한 방법으로 해당 타입의 서브클래스를 생성하는 몇몇 팩토리 메소드를 제공한다. 빈 버퍼는 일반적으로 allocate 메소드에 의해 생성된다. 데이터가 미리 채워진 버퍼는 wrap 메소드를 호출하여 만든다. allocate 메소드는 종종 입력에 유용하게 사용되고, wrap 메소드는 일반적으로 출력에 사용된다.  

#### 4.1. 할당
<br/>
기본 allocate() 메소드는 단순히 지정된 고정 용량을 가진 빈 버퍼를 새로 생성하여 반환한다. 예를 들어, 다음 코드는 바이트 버퍼와 정수 버퍼를 생성하며 각각의 크기는 100이다.  

```java
ByteBuffer buffer1 = ByteBuffer.allocate(100);
IntBuffer buffer2 = IntBuffer.allocate(100);
```

커서는 버퍼의 시작에 위치한다. (즉, 위치가 0이다.) allocate()에 의해 생성된 버퍼는 자바 배열로 구현되며, array()와 arrayOffset() 메소드로 접근할 수 있다. 예를 들어, 채널을 이용하여 큰 데이터 덩어리를 버퍼로 읽어 온 다음 버퍼로부터 배열을 가져와서 배열을 인자로 받는 다른 메소드에 사용할 수 있다.

```java
byte[] data1 = buffer1.array();
int[] data2 = buffer2.array();
```

array() 메소드는 해당 버퍼의 내부 데이터를 그대로 노출시키므로, 주의해서 사용해야 한다. 반환된 배열의 내용을 변경하면 버퍼에 그대로 반영되며, 반대도 마찬가지다. 여기서 일반적인 사용 패턴은 먼저 버퍼에 데이터를 채운다. 그리고 배열을 가져와서 배열을 조작한다. 이러한 패턴은 배열을 조작한 이후에 다시 버퍼에 무언가를 쓰지 않는 한 문제가 되지 않는다.  

#### 4.2. 직접 할당
<br/>
ByteBuffer 클래스는 버퍼에 대한 백업 배열을 생성하지 않는(즉, array() 메소드로 배열을 반환받을 수 없는) 추가적인 allocateDirect() 메소드를 제공한다. (ByteBuffer 클래스 이외의 다른 클래스는 제공하지 않는다.) 가상 머신은 DMA(Direct Memory Access)를 사용하여 이더넷 카드나 커널 메모리 등의 버퍼에 직접적으로 할당된 ByteBuffer를 구현한다. 꼭 필요한 것은 아니지만, 이 방식을 사용하여 I/O 연산의 성능을 향상시킬 수 있다. API 관점에서 보면, allocateDirect() 메소드는 allocate()와 동일하게 사용된다.  

```java
ByteBuffer buffer = ByteBuffer.allocateDirect(100);
```

직접 버퍼(direct buffer)에 array() 메소드와 arrayOffset() 메소드를 호출하면 UnsupportedOperationException 예외가 발생한다. 직접 버퍼는 버퍼의 크기가 큰(1MB 이상) 일부 가상 머신에서 훨씬 빨리 동작한다. 그러나 직접 버퍼는 간접 버퍼(indirect buffer)보다 생성하는 데 많은 비용이 들기 때문에 길게 사용하지 않도록 해야 한다. 자세한 내용은 가상 머신에 상당히 의존적이다. 그러나 대부분의 경우 성능 문제가 측정될 때까지 직접 버퍼를 사용하지 않는 것이 좋다.  

#### 4.3. 랩핑(Wrapping)
<br/>
이미 출력하고자 하는 데이터의 배열을 가지고 있다면 버퍼를 새로 만들고 채우는 것보다 배열을 버퍼로 감싸(wrapping)는 편이 낫다. 예를 들어:  

```java
byte[] data = "Some data".getBytes("UTF-8");
ByteBuffer buffer1 = ByteBuffer.wrap(data);
char[] text = "Some text".toCharArray();
CharBuffer buffer2 = CharBuffer.wrap(text);
```

여기서 버퍼는 배열에 대한 참조를 포함하고 있으며, 배열은 버퍼의 백업 배열처럼 제공된다. 랩핑 메소드로 생성된 버퍼는 직접 버퍼가 될 수 없다. 다시 한 번 말하지만 배열에 대한 변경은 버퍼에 반영되며, 반대도 역시 마찬가지다. 그러므로 해당 배열에 작업이 끝나기 전에 미리 랩핑하지 않도록 해야 한다.  

### 5. 채우기와 내보내기
<br/>
버퍼는 순차적인 접근을 위해 설계되었다. 각각의 버퍼는 position() 메소드로 확인할 수 있는 현재 위치를 가지고 있으며, 이 위치는 0에서 버퍼의 원소 개수 사이의 어딘가를 나타낸다는 사실을 다시 상기해 보도록 하자. 버퍼의 위치는 버퍼에 하나의 요소를 쓰거나 읽을 때마다 1씩 증가한다. 예를 들어, 용량 12를 가진 CharBuffer를 할당한다고 가정해 보자. 그리고 다섯 개의 문자로 버퍼를 채운다.  

```java
CharBuffer buffer = CharBuffer.allocate(12);
buffer.put('H');
buffer.put('e');
buffer.put('l');
buffer.put('l');
buffer.put('o');
```

이 버퍼의 위치는 현재 5가 된다. 이와 같은 작업을 "버퍼를 채운다"고 한다.  

버퍼는 버퍼가 가진 용량까지만 채울 수 있다. 만약 초기에 설정된 용량을 넘어서 채우려고 시도할 경우, put() 메소드는 BufferOverflowException 예외를 발생시킨다.  

위 상태의 버퍼에 대해 get()을 시도할 경우, 위치 5에 저장된 널(null) 문자(\u0000)를 반환받게 된다. 이 문자는 자바가 버퍼를 처음 초기화할 때 설정된 값이다. 여러분이 쓴 데이터를 다시 읽으려고 할 경우, 그 전에 먼저 해당 버퍼에 대해 flip()을 호출해야 한다.  

```java
buffer.flip();
```

이 메소드는 한도(limit)를 현재 위치로 설정하고, 위치(position)를 버퍼의 시작 위치인 0으로 설정한다. 이제 버퍼의 내용을 새로운 문자열로 내보낼 수 있다.  

```java
String result = "";
while (buffer.hasRemaining()) {
  result += buffer.get();
}
```

get() 호출 시마다 위치는 앞으로 1씩 이동한다. 위치가 한도(limit)에 도달하면, hasRemaining() 메소드는 false를 반환한다. 이와 같은 작업을 "버퍼를 내보낸다"고 한다.  

버퍼 클래스는 또한 버퍼의 위치를 변경하지 않고 버퍼 내의 특정 위치에서 데이터를 채우거나 내보내는 절대(absolute) 메소드를 제공한다. 예를 들어, ByteBuffer는 다음 두 메소드를 제공한다.  

+ public abstract byte get(int index)
+ public abstract ByteBuffer put(int index, byte b)  

이 두 메소드는 버퍼의 한도를 넘은 위치를 접근하려고 할 경우, IndexOutOfBoundsException 예외를 발생시킨다. 예를 들어, 절대 메소드를 사용하여 같은 작업을 다음과 같이 할 수 있다.  

```java
CharBuffer buffer = CharBuffer.allocate(12);
buffer.put(0, 'H');
buffer.put(1, 'e');
buffer.put(2, 'l');
buffer.put(3, 'l');
buffer.put(4, 'o');
```

그러나 위와 같이 절대 메소드로 데이터를 저장할 경우, 위치가 변경되지 않기 때문에 읽기 전에 flip() 호출하지 않아도 된다. 게다가 저장 순서도 의미가 없다. 아래는 위와 같은 작업을 한다.  

```java
CharBuffer buffer = CharBuffer.allocate(12);
buffer.put(1, 'H');
buffer.put(4, 'e');
buffer.put(0, 'l');
buffer.put(3, 'l');
buffer.put(2, 'o');
```

#### 5.1. 벌크 메소드
<br/>
아무리 버퍼라고 해도 한 번에 하나의 요소씩 채우고 비우는 것보다 데이터의 블록 단위로 작업하는 것이 훨씬 더 빠르다. 서로 다른 버퍼 클래스들은 각자의 요소 타입의 배열을 체우거나 내보내는 벌크(bulk) 메소드를 제공한다. 예를 들어, ByteBuffer는 미리 준비된 바이트 배열이나 서브 배열로부터 ByteBuffer를 채우거나 내보내는 put()과 get() 메소드를 제공한다.  

+ public ByteBuffer get(byte[] dst, int offset, int length)
+ public ByteBuffer get(byte[] dst)
+ public ByteBuffer put(byte[] array, int offset, int length)
+ public ByteBuffer put(byte[] array)  

이들 put() 메소드는 특정 배열이나 서브 배열로부터 데이터를 가져와 현재 위치에 넣는다. get() 메소드는 현재 위치에서 인자로 제공된 배열이나 서브 배열로 데이터를 읽어 온다. put()과 get() 메소드는 배열이나 서브 배열의 서브 배열의 길이에 따라서 위치가 증가한다. put() 메소드는 버퍼가 배열이나 서브 배열의 내용을 담을 만큼 충분한 공간이 없는 경우 BufferOverflowException 예외를 발생시킨다. 이 예외들은 모두 런타임 예외에 해당한다.  

#### 5.2. 데티어 변환
<br/>
자바에서 모든 데이터는 결국 바이트로 처리된다. int, double, float 등을 포함한 모든 기본 데이터 타입은 바이트로 쓰일 수 있다. 적절한 길이의 바이트의 연속은 기본 데이터 타입으로 해석될 수 있다. 예를 들어, 어떤 연속된 4바이트는 int나 float에 대응한다. (실제로 여러분이 어떤 타입으로 읽고 싶은지에 달렸다.) 연속된 8바이트는 long 또는 double에 대응한다. ByteBuffer 클래스는(오직 ByteBuffer 클래스만이) 기본 타입(boolean 제외)의 인자에 대응하는 바이트들로 버퍼를 채우는 상대적인 put() 메소드와 절대 put() 메소드를 제공한다. 그리고 기본 데이터 타입 형상에 필요한 적절한 바이트 수를 읽는 상대적인 get() 메소드와 절대 get() 메소드를 제공한다.  

+ public abstract char getChar()
+ public abstract ByteBuffer putChar(char value)
+ public abstract char getChar(int index)
+ public abstract ByteBuffer putChar(int index, char value)
+ public abstract short getShort()
+ public abstract ByteBuffer putShort(short value)
+ public abstract short getShort(int index)
+ public abstract ByteBuffer putShort(int index, short value)
+ public abstract int getInt()
+ public abstract ByteBuffer putInt(int value)
+ public abstract int getInt(int index)
+ public abstract ByteBuffer putInt(int index, int value)
+ public abstract long getLong()
+ public abstract ByteBuffer putLong(long value)  
+ public abstract long getLong(int index)
+ public abstract ByteBuffer putLong(int index, long value)
+ public abstract float getFloat()
+ public abstract ByteBuffer putFloat(float value)
+ public abstract float getFloat(int index)
+ public abstract ByteBuffer putFloat(int index, float value)
+ public abstract double getDouble()
+ public abstract ByteBuffer putDouble(double value)
+ public abstract double getDouble(int index)
+ public abstract ByteBuffer putDouble(int index, double value)  

전통적인 I/O에서 DataOutputStream과 DataInputStream에 의해 수행되던 작업들을 새로운 I/O 모델에서는 이들 메소드가 수행한다. 이 메소드들은 DataOutputStream과 DataInputStream에는 없는 추가적인 기능을 제공한다. 바로 연속된 바이트를 int, float, double 등으로 해석할 때, 빅엔디안으로 변환할지 리틀엔디안으로 변환할지 선택할 수 있다. 기본적으로 모든 값들은 빅엔디안으로 읽고 쓴다. (즉, 최상위 바이트가 먼저 온다.) 다음 두 개의 order() 메소드는 ByteOrder 클래스에 명명된 상수를 사용하여 버퍼의 바이트 순서를 설정하거나 확인한다. 예를 들어, 다음과 같이 버퍼를 리틀엔디안으로 변경할 수 있다.  

```java
if (buffer.order().equals(ByteOrder.BIG_ENDIAN)) {
  buffer.order(ByteOrder.LITTLE_ENDIAN);
}
```

문자 발생기 프로토콜 대신 바이너리 데이터를 생성하여 네트워크 테스트를 한다고 생각해보자. 이 테스트로 인해 아스키 문자 발생기 프로토콜에서는 발견되지 않았던 문제들이 부각될 수도 있다. 예를 들어, 모든 바이트에서 최상위 비트를 벗겨 내도록 설정된 오래된 게이트웨이는 매 3킬로바이트를 버리거나, 또는 예상치 못한 연속된 제어 문자로 인해 진단 모드로 전환할 수 있다. 이러한 문제들은 단지 이론적인 이야ㅇ기들은 아니다.  

모든 가능한 정수값을 보내어 네트워크에 이러한 문제가 없는지 확인할 수 있다. 이 테스트 방법은 가능한 4바이트 정수를 모두 테스트하는 데 거의 43억 번 반복한다. 수신 측에서는, 받은 데이터가 예상한 값이 맞는지 단순히 숫자를 비교하여 쉽게 확인할 수 있다. 어떤 문제가 발견될 경우, 정확히 어디서 문제가 발생했는지 쉽게 알 수 있다. 즉, 이 프로토콜은 다음과 같이 동작한다.  

(1) 클라이언트는 서버에 연결한다.  
(2) 서버는 즉시 4바이트 빅엔디안 정수를 0에서부터 보내기 시작하고 매번 1씩 증가시킨다. 이 값이 최대값을 넘어서 음수가 되면 다시 0에서 시작한다.  
(3) 서버는 무한히 실행된다. 클라이언트는 충분히 테스트를 한 후 연결을 종료한다.  

서버는 4바이트 길이의 정수를 ByteBuffer에 저정한다. 그리고 이 버퍼는 각 채널에 첨부된다. 각 채널이 쓰기가 가능한 상태인 경우, 버퍼의 내용을 채널로 내보낸다. 다음으로 버퍼의 위치를 앞으로 이동(rewind)하고, getInt() 메소드를 사용하여 버퍼의 내용을 읽는다. 프로그램은 이때 버퍼의 내용을 비우고 읽은 값을 1 증가시킨다. 그리고 putInt() 메소드를 사용하여 버퍼에 증가된 값을 저장한다. 마지막으로, flip() 메소드를 호출하여 내보낼 준비를 한다.  

#### 5.3. 뷰 버퍼
<br/>
SocketChannel에서 읽은 ByteBuffer가 하나의 특정 기본 데이터 타입만 포함할 경우, 뷰 버퍼(View Buffer)를 유용하게 사용할 수 있다. 뷰 버퍼는 내부의 ByteBuffer로부터 적절한 타입(예를 들어, DoubleBuffer, IntBuffer 등)으로 데이터를 가져오는 새로운 버퍼 객체이다. 뷰 버퍼를 변경하면 내장된 버퍼에 반영되며 반대도 마찬가지다. 그러나 각 버퍼는 자신만의 독립적인 한도(limit), 용량(capacity), 표시(mark) 그리고 위치(position)을 가진다. 뷰 버퍼는 ByteBuffer가 제공하는 다음 6개의 메소드 중 하나를 사용하여 만들어진다.  

+ public abstract ShortBuffer asShortBuffer()
+ public abstract CharBuffer asCharBuffer()
+ public abstract IntBuffer asIntBuffer()
+ public abstract LongBuffer asLongBuffer()
+ public abstract FloatBuffer asFloatBuffer()
+ public abstract DoubleBuffer asDoubleBuffer()  

뷰 버퍼 클래스의 메소드만을 사용하여 버퍼의 내용을 채우거나 내보낼 수도 있지만, 데이터는 IntBuffer의 내장된 원래 ByteBuffer를 사용한 채널에 읽고 써야 한다. SocketChannel 클래스는 ByteBuffer에 대해 읽고 쓰는 메소드만을 제공하므로 다른 종류의 버퍼에 대해서는 읽고 쓰지 못한다. 이 말은 곧 루프를 돌 때마다 ByteBuffer를 비워 줘야 함을 의미하며, 그렇지 않으면 버퍼가 가득 차서 프로그램이 종료된다. 여기서 두 버퍼의 위치와 한도는 따로 존재하므로 따로 고려해야 한다. 마지막으로, 논블록 모드에서 작업 중인 경우, 내부의 ByteBuffer에 있는 모든 데이터가 뷰 버퍼에서 읽거나 쓰기 전에 이미 내보내지지 않았는지 주의해야 한다. 논블록 모드에서 데이터를 내보낸 다음 버퍼가 여전히 int/double/char 등의 타입의 경계에 맞게 정렬되어 있다고 보장하지 않는다. 논블록 채널이 int 혹은 double의 반 바이트만 쓰는 것도 가능하다. 논블록 I/O를 사용할 경우, 뷰 버퍼에 추가적인 데이터를 쓰기 전에 이 문제를 반드시 확인해야 한다.  

#### 5.4. 버퍼 압축하기
<br/>
쓰기가 가능한 대부분의 버퍼는 compact() 메소드를 제공한다.  

+ public abstract ByteBuffer compact()
+ public abstract IntBuffer compact()
+ public abstract ShortBuffer compact()
+ public abstract FloatBuffer compact()
+ public abstract CharBuffer compact()
+ public abstract DoubleBuffer compact()  

(이 메소드가 이렇게 다양하지 않았다면, 이 6개의 메소드는 공통의 Buffer 슈퍼클래스에서 하나의 메소드로 대체될 수 있었을 것이다.) 압축하기는 버퍼에 남아 있는 데이터를 버퍼의 시작으로 이동시켜서 요소를 보관할 추가적인 공간을 확보한다. 시작 위치에 있던 기존 데이터는 덮어 쓰이고, 버퍼의 위치는 데이터의 끝으로 설정되므로 추가적인 데이터를 쓸 준비가 완료된다.  

압축하기는 복사할 때 특히 유용하게 사용된다. - 한 채널에서 읽고, 읽어 온 데이터를 논블록 I/O를 사용하여 다른 채널에 쓴다. 데이터를 버퍼로 읽어 온 다음. 버퍼의 내용을 출력한다. 그러고 나서 데이터를 압축한다. 그러고 나면 아직 출력되지 않은 데이터가 버퍼의 가장 앞에 위치하게 된다. 그리고 추가적인 데이터를 받기 위해 위치(position)를 버퍼에 남아 있는 데이터의 끝으로 설정한다. 이렇게 하면 하나의 버퍼를 사용하여 랜덤하게 읽기와 쓰기를 반복하는 것이 가능해진다. 또한 읽기를 여러 번 연속해서 처리하거나, 쓰기를 연속해서 처리하는 것도 가능하다. 만약 네트워크가 출력 준비는 마쳤으나 입력 준비가 되지 않은 경우, 이 방법을 활용할 수 있다.  

버퍼의 크기는 큰 차이를 만든다. 큰 버퍼를 사용하면 많은 버그를 감출 수 있다. 버퍼가 모든 테스트 케이스를 수용할 수 있을 만큼 충분히 크다면, 적절한 시점에 flip()을 호출하고 버퍼 내용을 내보내야 한다는 사실을 잊게 되며, 버그가 쉽게 감춰진다. 프로그램을 출시하기 전에 버퍼 크기를 중여 놓도록 하자. 이 경우에는 버퍼의 크기를 10으로 설정하여 테스트하였다. 이 테스트는 성능을 떨어트리기 때문에, 실제 제품에 이렇게 터무니 없이 작은 버퍼를 사용하지 않도록 해야 한다. 하지만 버퍼를 채울 때 제대로 동작하는지 확인하기 위해서는 버퍼 크기를 작게 하여 충분히 테스트해야 한다.  

#### 5.5. 버퍼 복사하기
<br/>
동일한 정보를 둘 이상의 채널에 전송하기 위해서는 버퍼를 복사해야 한다. 6가지 타입의 버퍼 클래스 각각이 제공하는 duplicate() 메소드가 이러한 일을 한다.  

+ public abstract ByteBuffer duplicate()
+ public abstract IntBuffer duplicate()
+ public abstract ShortBuffer duplicate()
+ public abstract FloatBuffer duplicate()
+ public abstract CharBuffer duplicate()
+ public abstract DoubleBuffer duplicate()  

이 메소드 호출 시 반환된 값은 실제 복제본은 아니다. 복사된 버퍼는 같은 데이터를 공유하며, 직접 버퍼가 아닌 경우 백업 배열 역시 공유한다. 한 버퍼의 내용을 변경하면 나머지 버퍼에도 반영된다. 따라서 대부분 버퍼에서 읽기만 할 때, 이 메소드를 사용한다. 그렇지 않으면 데이터가 수정되는 위치를 추적하기가 쉽지 않다.  

원본 버퍼와 복사된 버퍼는 비록 같은 데이터를 공유하긴 하지만 각각 독립된 표시(mark), 한도(limit) 그리고 위치(position)를 가지고 있다. 하나의 버퍼의 위치가 다른 버퍼의 위치보다 앞설 수도 있고 뒤처질 수도 있다.  

복사는 같은 데이터를 다수의 채널을 통해 병렬로 보내려고 할 때 유용하게 사용된다. 복사해서 전송할 원본 데이터를 메인 버퍼에 둔 다음 각 채널들은 이 버퍼의 복사본을 만든다. 그러고 나서 각 채널들은 각자의 복사본으로 각자의 속도로 파일을 전송하면 된다.  

#### 5.6. 버퍼 자르기
<br/>
버퍼를 자르는 것은 버퍼를 복사하는 것이 약간 변형된 것이다. 자르기 또한 새로운 버퍼를 생성하며, 해당 버퍼는 이전 버퍼와 데이터를 공유한다. 그러나 원본 버퍼의 현재 위치가 잘린 버퍼의 시작 위치가 되며, 잘린 버퍼의 용량은 원본 버퍼의 한도까지가 된다. 즉, slice()에 의해 생성된 버퍼는 원본 버퍼의 현재 위치에서 한도까지만 포함하고 있는 원본 버퍼의 부분 집합이다. 이 버퍼에 대해 rewind()를 호출하더라도 버퍼를 자르기 이전 원본 버퍼의 위치까지만 이동된다. 잘린 버퍼는 자신의 영역 이전의 원본 버퍼의 데이터를 볼 수 없다. 다시 한 번 타입별 6개의 버퍼 클래스는 각각 별도의 slice() 메소드를 제공한다.  

+ public abstract ByteBuffer slice()
+ public abstract IntBuffer slice()
+ public abstract ShortBuffer slice()
+ public abstract FloatBuffer slice()
+ public abstract CharBuffer slice()
+ public abstract DoubleBuffer slice()  

이 메소드는 헤더 다음에 데이터가 따라오는 프로토콜같이 여러 부분으로 쉽게 나뉘는 아주 큰 데이터 버퍼를 가지고 있을 때 유용하게 사용할 수 있다. 여러분은 헤더를 읽은 다음, 버퍼를 잘라서 데이터만 포함하고 있는 새로운 버퍼를 만들어 별도의 메소드나 클래스에 전달할 수 있다.  

#### 5.7. 표시와 리셋
<br/>
데이터를 다시 읽고 싶은 경우 입력 스트림과 마찬가지로 버퍼도 위치를 표시하거나 리셋할 수 있다. 하지만 입력 스트림과는 달리 일부 버퍼가 아닌 모든 버퍼에 적용된다. 이 메소드들은 다른 메소드와는 달리 관련된 메소드가 Buffer 슈퍼 클래스에 선언되어 있으며 다양한 서브클래스가 이를 상속하고 있다.  

+ public final Buffer mark()
+ public final Buffer reset()  

reset() 메소드는 표시가 설정되지 않을 경우 InvalidMarkException 런타임 예외를 발생시킨다. 표시는 또한 위치(position)가 표시 이전으로 설정될 때 해제된다.  

#### 5.8. 객체 메소드
<br/>
모든 버퍼 클래스는 일반적인 equals(), hashCode(), 그리고 toString() 메소드 등을 제공한다. 그들은 또한 Comparable 인터페이스를 구현하며, 따라서 compareTo() 메소드를 제공한다. 그러나 버퍼는 Serializable이나 Cloneable 인터페이스는 구현하지 않는다.  

두 버퍼가 다음 조건이 만족될 경우 같다고 간주된다.  

+ 두 버퍼가 동일한 타입이다(예를 들어, ByteBuffer는 결코 IntBuffer와는 같지 않지만 또 다른 ByteBuffer와는 같을 수 있다).
+ 두 버퍼에 남아 있는 요소의 수가 같다.
+ 두 버퍼가 같은 상대적인 위치에 남아 있는 요소들이 서로 같다.  

두 버퍼가 같은지 비교할 때 위치(position) 이전에 요소들, 버퍼의 용량, 한도, 또는 표시를 고려하지 않는다. 예를 들어, 다음 코드는 첫 번째 버퍼의 크기가 두 번째 버퍼의 두 배이지만 true를 출력한다.  

```java
CharBuffer buffer1 = CharBuffer.wrap("12345678");
CharBuffer buffer2 = CharBuffer.wrap("5678");
buffer1.get();
buffer1.get();
buffer1.get();
buffer1.get();
System.out.println(buffer1.equals(buffer2));
```

hashCode() 메소드는 같음을 비교하는 조건에 따라 구현되어 있다. 즉, 같은 두 버퍼는 같은 해시 코드를 가지며, 다른 두 버퍼는 같은 해시 코드를 가질 가능성이 없다. 그러나 버퍼의 해시 코드는 버퍼에 요소가 추가되거나 제거될 때마다 변경되기 때문에, 버퍼는 좋은 해시 테이블 키를 만들 수가 없다. (즉, 해시 테이블 키로 사용하기 어렵다.)  

비교는 각 버퍼에 남아 있는 요소들을 하나씩 비교하는 방법으로 구현된다. 대응하는 모든 요소들이 같다면 이 두 버퍼는 같다. 그렇지 않은 경우 서로 다른 요소의 첫 번째 쌍을 비교한 결과가 버퍼 비교의 결과가 된다. 서로 다른 요소를 찾기 전에 어느 한 버퍼의 요소가 부족한 경우, 즉 한 버퍼가 끝에 도달하고 나머지 한 버퍼는 아직 요소가 남아 있는 경우, 짧은 버퍼는 긴 버퍼보다 작다고 간주된다.  

toString() 메소드는 아래와 비슷해 보이는 문자열을 반환한다.  

java.nio.HeapByteBuffer[pos=0 lim=62 cap=62]  

이 메시지는 주로 디버깅에 유용하게 사용된다. 예외적으로 CharBuffer의 toString()은 좀다르게 동작하며, 버퍼에 남아 있는 문자들을 포함한 문자열을 반환한다.  

### 6. 채널
<br/>
채널은 파일, 소켓, 데이터그램 등과 같은 다양한 I/O 소스로부터 데이터 블록을 버퍼로 쓰거나 읽어 온다. 채널 클래스의 계층 구조는 여러 개의 인터페이스와 수많은 연산으로 뒤영켜 있다. 하지만 네트워크 프로그래밍을 위해서는 SocketChannel, ServerSocketChannel, 그리고 DatagramChannel 클래스 세 개가 가장 중요하다. 그리고 우리가 그동안 이야기한 TCP 연결을 위해서는 이 중에서도 앞의 두 개의 채널만 필요하다.  

#### 6.1. SocketChannel 클래스
<br/>
SocketChannel 클래스는 TCP 소켓에 대해 읽고 쓴다. 읽고 쓸 데이터는 ByteBuffer 객체로 먼저 인코딩되어야 한다. 각각의 SocketChannel은 Socket 객체와 연결되어 있으며 이 객체는 고급 설정을 위해 사용되지만, 기본 옵션만으로도 충분한 애플리케이션이라면 신경 쓰지 않아도 된다.  

##### 6.1.1. 연결하기
<br/>
SocketChannel 클래스는 public으로 선언된 생성자를 제공하지 않는다. 대신에 다음 두 정적 open() 메소드를 사용하여 새로운 SocketChannel 객체를 만들 수 있다.  

+ public static SocketChannel open(SocketAddress remote) throws IOException
+ public static SocketChannel open() throws IOException  

첫 번째 변형은 연결을 만들려고 시도하며, 연결이 만들어지거나 예외가 발생할 때까지 반환되지 않고 볼록된다. 예를 들어:  

```java
SocketAddress address = new InetSocketAddress("www.cafeaulait.org", 80);
SocketChannel channel = SocketChannel.open(address);
```

인자가 없는 버전은 즉시 연결하지 않고, 최초에 연결되지 않은 소켓을 반환한다. 이 소켓은 나중에 connect() 메소드를 호출하여 연결할 수 있다. 예를 들어:  

```java
SocketChannel channel = SocketChannel.open();
SocketAddress address = new InetSocketAddress("www.cafeaulait.org", 80);
channel.configureBlocking(false);
channel.connect();
```

논블록 채널의 connect() 메소드는 연결이 되기 전에 즉시 반환한다. 프로그램은 운영체제가 연결을 완료하는 동안 기다리지 않고 다른 일을 수행할 수 있다. 그러나 실제로 연결을 사용하기 전에 프로그램은 finishConnect() 메소드를 호출해야 한다.  

+ public abstract boolean finishConnect() throws IOException  

(이 메소드는 논블록 모드에서만 필요하다. 블록 채널에서 이 메소드를 호출하면 즉시 true가 반환된다. 연결이 맺어지고 사용할 준비가 된 경우 finishConnect() 메소드는 true를 반환한다.) 연결이 아직 맺어지지 않은 경우 finishConnect() 메소드는 false를 반환한다. 마지막으로, 연결을 맺을 수 없는 경우, 예를 들어 네트워크가 다운된 경우 이 메소드가 예외를 발생시킨다.  

프로그램에서 연결이 완료되었는지 확인이 필요한 경우 다음 두 메소드를 사용한다.  

+ public abstract boolean isConnected()
+ public abstract boolean isConnectionPending()  

isConnected() 메소드는 연결이 열려 있는 경우 true를 반환한다. isConnectionPending() 메소드는 연결이 아직 설정 중인 true를 반환한다.  

##### 6.1.2. 읽기
<br/>
SocketChannel에서 읽기 위해서는 먼저 채널이 데이터를 저장할 ByteBuffer를 만든다. 그리고 생성된 버퍼를 read() 메소드에 전달한다.  

+ public abstact int read(ByteBuffer dst) throws IOException  

채널은 채울 수 있는 만큼의 데이터로 버퍼를 채운다. 그러고 나서 저장한 바이트 수를 반환한다. 메소드가 스트림의 끝에 도달할 경우, 채널은 남아 있는 바이트로 버퍼를 채우고 다음 호출 시에 -1을 반환한다. 채널이 블록 모드인 경우, 이 메소드는 최소 1바이트를 읽어서 반환하며 예외가 발생한 경우 -1을 반환한다. 그러나 채널이 논블록 모드인 경우 0을 반환한다.  

데이터는 버퍼의 현재 위치에 저장되며, 이 위치는 저장될 때마다 자동으로 업데이트되기 때문에, 버퍼가 가득 찰 때까지 동일한 버퍼를 read() 메소드의 인자로 전달하여 호출할 수 있다. 예를 들어, 다음 루프는 버퍼가 가득 차거나 스트림이 끝에 도달할 때까지 읽는다.  

```java
while (buffer.hasRemaining() && channel.read(buffer) != -1);
```

한 소스로부터 여러 버퍼를 채우는 방식을 스캐터(scatter)라고 하는데, 다음 세도는 소캐터를 할 때 유용하게 사용할 수 있다. 다음 두 메소드는 인자로 ByteBuffer 객체의 배열을 전달받고 배열에 데이터를 차례대로 채운다.  

+ public final long read(ByteBuffer[] dsts) throws IOException
+ public final long read(ByteBuffer[] dsts, int offset, int length) throws IOException  

첫 번째 메소드는 버퍼 전체를 채운다. 두 번째 메소드는 첫 배열의 offset 위치에서 시작하여 length 길이만큼 채운다. 버퍼 배열에 데이터를 채우기 위해, 목록의 마지막 버퍼에 공간이 남아 있는 동안 루프를 돌기만 하면 된다. 예를 들어:  

```java
ByteBuffer[] buffers = new ByteBuffer[2];
buffers[0] = ByteBuffer.allocate(1000);
buffers[1] = ByteBuffer.allocate(1000);
while (buffers[1].hasRemaining() && channel.read(buffers) != -1);
```

##### 6.1.3. 쓰기
<br/>
소켓 채널은 읽기 메소드와 쓰기 메소드 모두를 제공한다. 일반적으로 이 메소드는 양방향으로 읽고 쓰기를 지원한다. 쓰기 위해서는 단지 ByteBuffer를 채우고 flip()을 호출한 다음, 쓰기 메소드 중 하나에 전달하기만 하면 된다. 그리고 쓰기 메소드는 버퍼의 데이터를 출력으로 내보낸다. 읽기는 이 과정의 반대로 수행된다.  

기본 write() 메소드는 인자로 단일 버퍼를 전달받는다.  

+ public abstract int write(ByteBuffer src) throws IOException  

읽기와 마찬가지로, 이 메소드는 채널이 논블록인 경우 버퍼의 내용을 완전히 출력한다고 보장하지 않는다. 그러나 버퍼는 커서 기반이기 때문에 버퍼 내용을 모두 내보낼 때까지 반복해서 호출하면 된다.  

```java
while (buffer.hasRemaining() && channel.write(buffer) != -1);
```

여러 버퍼로부터 한 소켓에 데이터를 쓰는 방법을 개더(gether)라고 하며, 다음 메소드는 개더를 하는 데 유용하게 사용된다.  

예를 들어, HTTP 헤더와 HTTP 본문을 서로 다른 버퍼에 저장하고 싶은 경우가 있다. 실제 구현은 두 개의 스레드를 사용하여 버퍼를 동시에 채우거나 오버랩드(overlapped) I/O를 사용할 수 있을 것이다. 다음 두 메소드는 ByteBuffer 객체의 배열을 인자로 전달받고, 각 배열을 차례대로 내보낸다.  

+ public final long write(ByteBuffer[] dsts) throws IOException
+ public final long write(ByteBuffer[] dsts, int offset, int length) throws IOException  

첫 번째 메소드는 모든 버퍼를 내보낸다. 두 번째 메소드는 첫 배열의 offset 위치에서 시작하여 length 길이만큼 내보낸다.  

##### 6.1.4. 종료하기
<br/>
일반적인 소켓과 마찬가지로 채널 사용이 끝난 다음에 사용한 포트와 다른 리소스들을 해제하기 위해 채널을 닫아 줘야 한다.  

+ public void close() throws IOException  

이미 닫힌 채널을 다시 닫을 경우 아무런 일도 일어나지 않는다. 닫힌 소켓에 읽기나 쓰기를 시도할 경우 채널은 예외를 발생시킨다. 채널이 닫혔는지 확실하지 않은 경우, isOpen()를 호출하여 확인해야 한다.  

+ public boolean isOpen()  

메소드 이름에서 알 수 있듯이 채널이 닫혀 있는 경우 false를 반환하고 열려 있는 경우 true를 반환한다(Channel 인터페이스에는 close()와 isOpen() 두 개의 메소드만 선언되며 있으며, 모든 채널 클래스에 의해 공유된다).  

자바 7번전에서는 SocketChannel 클래스가 AutoCloseable을 구현하고 있기 때문에, try-with-resources 구문을 사용할 수 있다.  

#### 6.2. ServerSocketChannel 클래스
<br/>
ServerSocketChannel 클래스는 들어오는 연결을 수용하기 위한 한 가지 목적으로 사용된다. 이 클래스에 대해 읽거나 쓸 수 없으며 연결할 수도 없다. 이 클래스가 제공하는 유일한 동작은 새로운 연결을 수용하는 것뿐이다. 클래스 자체는 단지 네 개의 메소드만 선언하고 있으며, 그중 accept()가 가장 중요한 메소드다. ServerSocketChannel는 또한 슈퍼클래스로부터 몇 개의 메소드를 상속하며, 대부분 들어오는 연결의 알림을 받기 위해 필요한 셀렉터 등록에 관련된 메소드다. 그리고 마지막으로, 다른 모든 채널들처럼 close() 메소드를 제공하며 호출 시 서버 소켓이 졸료된다.  

##### 6.2.1. 서버 소켓 채널 만들기
<br/>
정적 팩토리 ServerSocketChannel.open() 메소드는 새로운 ServerSocketChennel 객체를 만든다. 그러나 이 메소드의 이름은 약간 오해의 소지가 있다. 이 메소드는 실제 새로운 서버 소켓을 열지 않는다. 단지 객체를 만들 뿐이다. 반환된 객체를 사용하기 전에 해당 채널에 연결된 ServerSocket을 구하기 위해 socket() 메소드를 호출해야 한다. 이 시점에서, ServerSocket 메소드가 제공하는 다양한 set 메소드를 호출하여 버퍼 사이즈나 소켓 타임아웃과 같은 서버 옵션들을 설정할 수 있다. ServerSocket은 바인드할 포트 정보를 가진 SocketAddress를 사용하여 포트에 연결한다. 예를 들어, 다음 코드는 포트 80에 대해 ServerSocketChannel을 연다.  

```java
try {
  ServerSocketChannel server = ServerSocketChannel.open();
  ServerSocket socket = serverChannel.socket();
  SocketAddress address = new InetSocketAddress(80);
  socket.bind(address);
} catch (IOException ex) {
  System.err.println("Could not bind to port 80 because " + ex.getMessage());
}
```

자바 7에서, ServerSocketChannel 자체가 bind() 메소드를 제공하므로 코드가 더 간결해진다.  

```java
try {
  ServerSocketChannel server = ServerSocketChannel.open();
  SocketAddress address = new InetSocketAddress(80);
  server.bind(address);
} catch (IOException ex) {
  System.err.println("Could not bind to port 80 because " + ex.getMessage());
}
```

여기서 생성자 대신 팩토리 메소드를 사용함으로서 이 클래스에 대해 다른 가상 머신에서는 로컬 하드웨어와 운영체제에 좀 더 최적화된 구현을 제공할 수 있다. 그러나 사용자가 직접 이 팩토리를 구성할 수는 없다. open() 메소드는 동일한 가상 머신에서 항상 같은 클래스의 인스턴스를 반환한다.  

##### 6.2.2. 연결 수용하기
<br/>
ServerSocketChannel 객체를 열고 바인드하고 나면 accept()를 호출하여 들어오는 연결을 대기할 수 있다.  

+ public abstract SocketChannel accept() throws IOException  

accept() 메소드는 블록/논블록 모든 모드에서 동작할 수 있다. 블록 모드에서 accept() 메소드는 연결이 들어올 때까지 기다린다. 연결이 수용되면 accept() 메소드는 원격 클라리언트에 연결된 SocketChannel 객체를 반환한다. 스레드는 연결이 만들어질 때까지 아무런 일도 할 수 없다. 이러한 방식은 각 요청에 대해 즉시 응답하는 간단한 서버에 적절하다. accept()는 기본적으로 블록 모드로 동작한다.  

ServerSocketChannel는 또한 논블록 모드로 동작한다. 이 경우에, accept() 메소드는 들어오는 연결이 없는 경우 널(null)을 반환한다. 논블록 모드는 각 연결에 대해 많은 작업이 필요하며, 다수의 요청을 병렬로 처리해야 하는 서버에 적절하다. 논블록 모드는 보통 셀렉터와 함께 사용된다. ServerSocketChannel을 논블록 모드로 설정하려면, configureBlocking() 메소드에 인자를 false로 호출한다.  

accept() 메소드는 문제가 발생할 경우 IOException 예외를 발생시키도록 선언되어 있다. 아래의 런타임 예외뿐만 아니라, 더 자세한 문제를 나타내는 IOException의 몇몇 서브클래스들이 존재한다.  

+ ClosedChannelException  
ServerSocketChannel을 닫은 후에 다시 열 수 없다.  

+ AsynchrounousCloseException  
accept()가 실행 중인 동안 다른 스레드에서 해당 ServerSocketChannel을 종료하였다.  

+ ClosedByInterruptException  
블록된 ServerSocketChannel이 기다리는 동안 다른 스레드가 블록된 스레드를 인터럽트하였다.  

+ NotYetBoundException  
open() 메소드를 호출했지만 accept() 메소드를 호춣하기 전에 ServerSocketChannel의 ServerSocket에 바인드하지 않았다. 이 예외는 IOException이 아닌 런타임 예외다.  

+ SecurityException  
보안 관리자가 애플리케이션이 요청한 포트로 바인드하는 것을 거부했다.  

#### 6.3. Channels 클래스
<br/>
Channels은 전통적인 I/O 기반 스트림, reader, 그리고 writer 등을 채널로 감싸기 위한 간단한 유틸리티 클래스이다. 이 클래스는 프로그램의 성능을 위해, 프로그램의 일부에 새로운 I/O 모델을 사용하고자 할 때 유용하게 사용할 수 있으며, 스트림을 필요로 하는 이전 API와도 여전히 잘 연동된다. 이 클래스는 스트림을 채널로 변환하는 메소드를 제공하며, 채널을 스트림, reader, writer로 변환하는 메소드도 제공한다.  

+ public static InputStream newInputStream(ReadableByteChannel ch)
+ public static OutputStream newOutputStream(WritableByteChannel ch)
+ public static ReadableByteChannel newChannel(InputStream in)
+ public static WritableByteChannel newChannel(OutputStream out)
+ public static Reader newReader(ReadableByteChannel channel, CharsetDecoder decoder, int minimumBufferCapacity)
+ public static Reader newReader(ReadableByteChannel ch, String encoding)
+ public static Writer newWriter(WritableByteChannel ch, String encoding)  

SocketChannel 클래스는 위에서 볼 수 있는 ReadableByteChannel와 WritableByteChannel 두 인터페이스를 구현한다. ServerSocketChannel은 읽거나 쓸 수 없기 때문에 이 두 인터페이스를 구현하지 않는다.  

예를 들어, 현재 모든 XML API들은 XML 문서를 읽기 위해 스트림, 파일, reader, 그리고 다른 전통적인 I/O API를 사용한다. 만약 여러분이 SOAP 요청을 처리하도록 설계된 서버를 작성하고 있다면, 채널을 사용하여 HTTP 요청 본문을 읽고 SAX를 사용하여 XML을 분석하려고 할 것이다. 이 경우에 채널을 XMLReader의 parse() 메소드에 전달하기 전에 스트림으로 변환해야 한다.  

```java
SocketChannel channel = server.accept();
processHTTPHeader(channel);
XMLReader parser = XMLReaderFactory.createXMLReader();
parser.setContentHandler(someContentHandlerObject);
InputStream in = Channels.newInputStream(channel);
parser.parse(in);
```

#### 6.3. 비동기 채널(자바 7)
<br/>
자바 7에서 AsynchronousSocketChannel와 AsynchronousServerSocketChannel 클래스가 소개되었다. 이 두 클래스는 SocketChannel 그리고 ServerSocketChannel 클래스와 비슷하게 동작하며 거의 동일한 인터페이스를 제공한다. (그렇다고 이 두 클래스의 서브클래스는 아니다.)  

그러나 SocketChannel 그리고 ServerSocketChannel 클래스와는 달리 비동기 채널에 대한 읽기나 쓰기 시에 I/O가 완료되기도 전에 즉시 반환된다. 데이터를 읽거나 쓰는 것은 Future 또는 CompletionHandler에 의해 추가로 처리된다. connect()와 accept() 메소드 또한 비동기적으로 실행되며 Future를 반환한다. 셀렉터는 사용되지 않는다.  

예를 들어, 프로그램이 시작 시에 많은 초기화 작업을 수행해야 한다고 가정해 보자. 이 초기화 작업 중 일부는 각 연결마다 수 초가 걸리는 네트워크 연결이 포함되어 있다. 먼저 몇몇 비동기 작업을 병렬로 실행한 다음, 나머지 로컬 초기화 작업을 수행한다. 그리고 난 다음 네트워크 동작의 결과를 요청한다.  

```java
SocketAddress address = new InetSocketAddress(args[0], port);
AsynchronousSocketChannel client = AsynchronousSocketChannel.open();
Future<Void> connected = client.connect(address);

ByteBuffer buffer = ByteBuffer.allocate(74);

// 연결이 완료되길 기다린다.
connected.get();

// 연결로부터 읽는다.
Future<Integer> future = client.read(buffer);

// 핑료한 작업들을 한다.

// 읽기가 완료되길 기다린다.
future.get();

// 버퍼를 flip()하고 내보낸다.
buffer.flip();
WritableByteChannel out = Chennels.newChannel(System.out);
out.write(buffer);
```

이 접근 방법의 장점은 프로그램이 다른 일을 수행하는 동안 네트워크 연결이 병렬로 실행된다는 것이다. 병렬로 요청한 네트워크 작업의 결과를 처리할 준비가 된 경우, 그 전에 Future.get() 메소드를 호출하여 기다려야 한다. 스레드 풀과 callable을 이용하면 동일한 효과를 이룰 수 있지만, 이 방법이 좀 더 간단하며 버퍼와 함께 사용할 경우 좀 더 자연스럽다.  

이 접근 방법은 또한 결과를 특별한 순서로 받아야 할 경우 유용하게 사용할 수 있다. 그러나 결과의 순서에 상관없는 경우, 각 네트워크 연결에 대한 읽기를 다른 연결과 상관없이 처리할 수 있는 경우, 대신 CompletionHandler를 사용하는 것이 더 나을 수도 있다.  

예를 들어, 여러분이 웹 페이지를 긁어 와서 뒷단의 처리 서버로 페이지를 공급하는 검색 엔진 웹 스파이더(web spider)를 작성하고 있다고 생각해 보자. 이때 반환되는 응답의 순서는 중요하지 않기 때문에 많은 수의 AsynchronousSocketChannel 요청을 생성하고 각각에 뒷단에서 온 결과를 저장할 CompletionHandler를 제공할 수 있다.  

일반적인 CompletionHandler 인터페이스를 두 개의 메소드를 선언한다. 하나는 읽기가 성공적으로 완료된 경우 호출되는 completed()이고, 나머지 하나는 I/O 에러 시에 호출되는 failed()이다. 다음의 예는 수신된 내용을 System.out으로 출력하는 간단한 CompletionHandler이다.  

```java
class LineHandler implements CompletionHandler<Integer, ByteBuffer> {

  @Override
  public void completed(Integer result, ByteBuffer buffer) {
    buffer.flip();
    WritableByteChannel out = Channels.newChannel(System.out);
    try {
      out.write(buffer);
    } catch (IOException ex) {
      System.err.println(ex);
    }
  }

  @Override
  public void failed(Throwable ex, ByteBuffer attachment) {
    System.err.println(ex.getMessage());
  }
}
```

채널로부터 읽을 때 버퍼와 첨부, 그리고 CompletionHandler를 read() 메소드에 전달한다.  

```java
ByteBuffer buffer = ByteBuffer.allocate(74);
CompletionHandler<Integer, ByteBuffer> handler = new LineHandler();
channel.read(buffer, buffer, handler);
```

여기서는 첨부로 버퍼 자체를 전달했다. 이것이 네트워크로부터 읽은 데이터를 처리하기 위해 CompletionHandler에 전달하는 한 가지 방법이다. 또 다른 일반적인 방법으로는 익명의 내부 클래스로 CompletionHandler를 만들고 final 로컬 변수로 버퍼를 만드는 것이다. 그렇게 되면, 완료 핸들러 내부 범위에 있게 된다.  

비록 안전하게 멀티스레드에서 AsynchronousSocketChannel 또는 AsynchronousServerSocketChannel를 공유할 수는 있지만, 하나 이상의 스레드에서 동시에 이 채널로부터 읽고 쓸 수는 없다. (두 개의 다른 스레드에서 한쪽은 읽고 다른 한쪽은 쓸 수는 있다.) 다른 스레드에서 읽기를 붙잡고 있는데 또 다른 스레드에서 읽으려고 할 경우, read() 메소드는 ReadPendingException 예외를 발생시킨다. 마찬가지로, 다른 스레드에서 쓰기를 붙잡고 있는데 또 다른 스레드에서 읽으려고 할 경우, write() 메소드는 WritePendingException 예외를 발생시킨다.  

#### 6.4. 소켓 옵션(자바 7)
<br/>
자바 7과 함께, SocketChannel, ServerSocketChennel, AsynchronousServerSocketChannel, AsynchronousSocketChannel, 그리고 DatagramChannel 클래스 모두는 새로운 NetworkChannel 인터페이스를 구현한다. 이 인터페이스의 주 목적은 SO&#95;TIMEOUT, SO&#95;LINGER, SO&#95;SNDBUF, SO&#95;RCVBUF, 그리고 SO&#95;KEEPALIVE, TCP&#95;NODELAY 같은 다양한 TCP 옵션을 지원하는 것이다. 이 옵션들은 설정 대상이 소켓인지 채널인지에 상관없이, TCP 스택 내부적으로 같은 의미로 사용된다. 그러나 이 옵션들에 대한 인터페이스는 약간 다르다. 지원되는 옵션에 대한 개별적인 메소드 대신 채널 클래스 각각은 옵션을 설정하고, 확인하고, 지원되는 옵션 목을 가져오는 세 개의 메소드만 제공한다.  

+ <T> T getOption(SocketOption<T> name) throws IOException
+ <T> NetworkChannel setOption(SocketOption<T> name, T value) throws IOException
+ Set<SocketOption<?>> supportedOptions()  

SocketOption 클래스는 각 옵션의 이름과 타입을 나타내는 일반화(generic) 클래스이다. 타입 매개변수 <T>는 옵션이 boolean, Integer 또는 NetworkInterface인지 결정한다. StandardSocketOptions 클래스는 자바가 인지하는 11가지 옵션에 대해 각각의 상수를 제공한다.  

+ SocketOption<NetworkInterface> StandardSocketOptions.IP_MULTICASE_IF
+ SocketOption<Boolean> StandardSocketOptions.IP_MULTICASE_LOOP
+ SocketOption<Integer> StandardSocketOptions.IP_MULTICASE_TTL
+ SocketOption<Integer> StandardSocketOptions.IP_TOS
+ SocketOption<Boolean> StandardSocketOptions.SO_BROADCAST
+ SocketOption<Boolean> StandardSocketOptions.SO_KEEPALIVE
+ SocketOption<Integer> StandardSocketOptions.SO_LINGER
+ SocketOption<Integer> StandardSocketOptions.SO_RCVBUF
+ SocketOption<Boolean> StandardSocketOptions.SO_REUSEADDR
+ SocketOption<Integer> StandardSocketOptions.SO_SNDBUF
+ SocketOption<Boolean> StandardSocketOptions.TCP_NODELAY  

예를 들어, 다음 코드는 클라이언트 네트워크 채널을 열고 SO_LINGER를 240초로 설정한다.  

```java
NetworkChannel channel = SocketChannel.open();
channel.setOption(StandardSocketOptions.SO_LINGER, 240);
```

예를 들어, ServerSocketChannel은 SO&#95;REUSEADDR과 SO&#95;RCVBUF를 지원하지만 SO&#95;SNDBUF는 지원하지 않는다. 채널이 지원하지 않는 옵션을 설정할 경우 UnsupportedOperationException 예외가 발생한다.  

### 7. 준비된 것 선택하기
<br/>
네트워크 프로그래밍을 위한 새 I/O API의 두 번째 부분은 준비된 것을 선택하는 것이다. 준비된 채널을 선택함으로써 읽고 쓸 때 블록하지 않아도 된다. 이 내용은 주로 서버의 관삼사이지만, 여러 개의 창을 띄워서 동시에 여러 연결을 시도하는 웹 스파이더나 브라우저 같은 클라이언트 프로그램에서도 활용할 수 있다.  

준비된 것을 선택하기 위해서는 서로 다른 채널들이 Selector 객체에 등록되어야 하며, 이때 각 채널에 할당되는 SelectionKey가 사용된다. 그러고 나서 프로그램은 Selector 객체에게 작업을 수행할 준비가 된 채널 키의 세트를 요청한다.  

#### 7.1. Selector 클래스
<br/>
셀렉터에 있는 유일한 생성자는 protected로 선언되어 있다. 일반적으로 새로운 셀렉터는 정적 팩토리 메소드인 Selector.open()를 호출하여 생성된다.  

+ public static Selector open() throws IOException  

다음 단계는 채널을 셀렉터에 추가하는 것이다. Selector 클래스에는 채널을 추가하는 메소드가 없다. 반대로 register() 메소드가 SelectableChannel 클래스에 선언되어 있다. 모든 채널이 선택 가능한 것은 아니다. 특히 FileChannels은 선택할 수 없다. 그러나 모든 네트워크 채널은 선택 가능하다. 따라서 채널은 등록 메소드 중 하나에 전달된 셀렉터를 사용하여 셀렉터에 등록된다.  

+ public final SelectionKey register(Selector sel, int ops) throws ClosedChannelException
+ public final SelectionKey register(Selector sel, int ops, Object att) throws ClosedChannelException  

이러한 접근 방법은 뭔가 거꾸로 된 것 같지만, 사용하기 어렵지는 않다. 첫 번째 인자는 채널을 등록할 셀렉터다. 두 번째 인자는 채널에 대해 등록할 연산을 나타내는 SelectionKey 클래스 명명된 상수이다. SelectionKey 클래스는 연산의 종류를 선택할 때 사용되는 네 개의 명명된 비트 상수를 정의하고 있다.  

+ SelectionKey.OP&#95;ACCEPT
+ SelectionKey.OP&#95;CONNECT
+ SelectionKey.OP&#95;READ
+ SelectionKey.OP&#95;WRITE  

이들은 비트 플래그 정수 상수(1, 2, 4, 등등)이므로 하나의 셀렉터에 여러 동작을 등록해야할 경우, 비트 연산자를 사용하여 상수 값을 합칠 수 있다.  

```java
channel.register(selector, SelectionKey.OP_READ | SelectionKey.OP_WRITE);
```

선택적으로 제공 가능한 세 번째 인자는 키에 대한 첨부이다. 연결의 상태를 저장하는 객체를 전달하는 종종 사용된다. 예를 들어, 웹 서버를 구현하는 경우 서버가 클라이언트에게 보내고 있는 로컬 파일에 연결된 FileInputStream 또는 FileChannel을 첨부할 수 있을 것이다.  

서로 다른 채널을 셀렉터에 등록한 이후, 어느 채널이 처리할 준비가 됐는지 셀렉터에게 언제든지 물을 수 있다. 채널들은 몇몇 연산은 준비가 되어 있지만 준비되지 않은 연산도 있을 수 있다. 예를 들어, 한 채널이 읽을 준비는 됐는데 쓸 준비는 안 됐을 수 있다.  

준비된 채널을 선택하는 세 가지 메소드가 제공된다. 이 메소드들은 준비된 채널을 찾는데 기다리는 시간에 차이가 있다. 먼저 selectNow() 논블록 방식으로 채널을 찾는다. 지금 바로 준비된 연결이 없는 경우 즉시 반환된다.  

+ public abstract int selectNow() throws IOException  

다른 두 메소드는 블록 모드로 등록한다.  

+ public abstract int select() throws IOException
+ public abstract int select(long timeout) throws IOException  

이 두 메소드 중에 첫 번째 메소드는 최소 하나의 채널이 처리할 준비가 될 때까지 반환하지 않고 기다린다. 두 번째 메소드는 0을 반환하기 전에 최대 타임아웃 밀리초까지 준비된 채널을 기다린다. 이 메소드들은 준비된 채널이 없는 경우 딱히 할 일이 없는 프로그램에서 유용하게 사용할 수 있다.  

준비된 채널이 있는 경우, selectKeys() 메소드를 호출하여 준비된 채널을 구할 수 있다.  

+ public abstract Set<SelectionKey> selectedKeys()  

반환된 세트를 회전하여 차례대로 각 SelectionKey를 처리한다.  

그리고 또한 처리가 끝난 키는 셀렉터에게 처리가 끝났음을 알려 주기 위해 iterator에서 제거해야 한다. 그렇지 않으면 셀렉터는 아직 작업이 끝나지 않았다고 판단하고 루프 회전 시마다 계속해서 반환한다.  

마지막으로, 서버를 종료할 준비가 되거나 더 이상 셀렉터가 필요하지 않는 경우, 셀렉터를 닫는다.  

+ public abstract void close() throws IOExeption  

이 메소드는 해당 셀렉터와 관련된 모든 리소스를 해제한다. 더 중요한 것은, 셀렉터에 등록된 모든 key를 취소하고, 이 셀렉터의 select() 메소드 중 하나를 호출하여 블록되어 있던 모든 스레드에게 인터럽트를 발생시킨다.  

#### 7.2. SelectionKey 클래스
<br/>
SelectionKey 객체는 채널에 포인터로 제공된다. SelectionKey는 또한 객체를 첨부하고 있으며, 보통 해당 채널의 연결 상태를 보관하고 있다.  

SelectionKey 객체는 채널을 셀렉터에 등록할 때 register() 메소드 호출에 의해 반환된다. 그러나 일반적으로 이 참조를 보관할 필요는 없다. selectedKeys() 메소드는 같은 객체를 세트 안에 다시 반환된다. 단일 채널이 다수의 셀렉터에 등록될 수 있다.  

선택된 키 세트에서 SelectionKey를 구한 다름, 먼저 해당 키에 준비된 작업을 확인해야 한다. 해당 키에 준비된 작업은 다음 네 가지 중 하나이다.  

+ public final boolean isAcceptable()
+ public final boolean isConnectable()
+ public final boolean isReadable()
+ public final boolean isWritable()  

이 확인이 항상 필요한 것은 아니다. 셀렉터에 등록된 모든 키가 한 가지 동작만 대기할 경우, 반환된 키도 한 가지 동작만 준비되므로 꼭 확인할 필요는 없다. 그러나 셀렉터가 여러 종류의 준비 상태를 확인할 경우, 해당 채널을 처리하기 전에 어떤 동작이 해당 채널을 준비 상태로 만들었는지 확인이 필요하다. 채널은 또한 여러 개의 동작이 동시에 준비될 수도 있다.  

키와 관련된 채널의 준비된 작업이 확인되고 나면, channel() 메소드를 호출하여 채널을 구한다.  

+ public abstract SelectableChannel channel()  

상태 정보를 유지하기 위해 SelectionKey에 객체를 저장해 뒀다면, attachment() 메소드를 호출하여 이 객체를 얻을 수 있다.  

+ public final Object attachment()  

마지막으로, 해당 연결의 작업이 끝날 때 연결의 SelectionKey 객체를 셀렉터에서 제거하여 셀렉터가 리소스를 낭비하지 않도록 한다. cancel() 메소드를 호출하여 이 작업을 수행한다.  

+ public abstract void cancel()  

하지만 이 동작은 채널을 닫지 않은 경우에만 필요하다. 채널을 닫게 되면 자동으로 모든 셀렉터로부터 해당 채널의 모든 키를 해제한다. 마찬가지로 셀렉터를 닫게 되면 등록된 모든 키들이 더 이상 효력이 없어진다.
