---
title:  InetAddress 클래스
categories:
- Java Network Programming
feature_text: |
  ## InetAddress 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

java.net.InetAddress 클래스는 IPv4와 IPv6 주소에 대한 자바의 고수준 표현 방식이다. 이 클래스는 Socket, ServerSocket, DatagramSocket, DatagramPacket 등을 포함한 대부분의 다른 네트워킹 클래스에 의해 사용된다. InetAddress 클래스는 보통 호스트네임과 IP 주소 모두를 포함하고 있다.  

### 1. InetAddress 객체 생성하기
<br/>
InetAddress 클래스에는 public으로 선언된 생성자가 존재하지 않는다. 대신 InetAddress는 호스트네임을 IP주소로 변환하기 위해 DNS 서버로 연결하는 정적 팰토리(static factory) 메소드를 제공한다. 가장 일반적인 InetAddress.getByName()이 있다.  

```java
InetAddress address = InetAddress.getByName("www.oreilly.com");
```

이 메소드는 단지 InetAddress의 private 문자열 필드를 설정하는 것으로 끝나지 않으며, 이름과 숫자로 된 주소를 검색하기 위해 실제 로컬 DNS 서버로 접속한다[이미 이전에 검색된 주소의 경우, 검색 결과가 저장(cache)되어 있을 수 있으며 이러한 경우에는 네트워크 연결이 발생하지 않는다]. DNS 서버가 주소를 찾지 못할 경우, 이 메소드는 IOException의 서브클래스인 UnknownHostException을 발생시킨다.  

그리고 반대로 IP 주소로 호스트네임을 검색할 수도 있다.  

```java
InetAddress address = InetAddress.getByName("208.201.239.100");
System.out.println(address.getHostName());
```

검색한 주소에 대한 호스트네임이 없는 경우, getHostName() 메소드는 단순히 검색에 사용한 주소를 다시 반환한다.  

www.oreilly.com이 실제 두 개의 주소를 가진다고 언급했다. getByName()이 둘 중 어느 것을 반환할지는 확실하지 않다. 만약 특정 호스트에 대한 모든 주소가 필요한 경우에는 getAllByName() 메소드를 대신 사용할 수 있으며, 이 메소드는 주소의 배열을 반환한다.  

```java
try {
  InetAddress[] addresses = InetAddress.getAllByName("www.oreilly.com");
  for (InetAddress address: addresses) {
    System.out.println(address);
  }
} catch (UnknownHostException ex) {
  System.out.println("Could not find www.oreilly.com");
}
```

마지막으로, getLocalHost() 메소드는 코드가 실행 중인 호스트에 대한 InetAddress 객체를 반환한다.  

```java
InetAddress me = InetAddress.getLocalHost();
```

이 메소드는 "elharo.laptop.corp.com"과 "192.1.254.68" 같은 실제 호스트네임과 IP 주소를 얻기 위해 DNS로 연결을 시도한다. 그러나 해당 정보를 얻는 데 실패할 경우, 메소드는 대신 루프백(loopback) 주소를 반환한다.  

숫자로 된 주소를 알고 있는 경우, 해당 주소로 InetAddress.getByAddress()를 호출하면 DNS에 대한 쿼리 과정 없이 InetAddress 객체를 만들 수 있다. 이 메소드는 존재하지 않거나 검색할 수 없는 호스트에 대한 InetAddress 객체를 만들 수 있다. 이 메소드는 존재하지 않거나 검색할 수 없는 호스트에 대한 InetAddress 객체를 만들어야 할 경우 사용할 수 있다.  

+ public static InetAddress getByAddress(byte[] addr) throws UnknownHostException
+ public static InetAddress getByAddress(String hostname, byte[] addr) throws UnknownHostException  

첫 번째 InetAddress.getByAddress() 팩토리 메소드는 호스트네임 없이 IP 주소만으로 InetAddress 객체를 만든다. 두 번째 InetAddress.getByAddress() 메소드는 주어진 IP 주소와 호스트네임으로 InetAddress 객체를 만든다.  

```java
byte[] address = {107, 23, (byte) 216, (byte) 196};
InetAddress lessWrong = InetAddress.getByAddress(address);
InetAddress lessWrongWithname = InetAddress.getByAddress("lesswrong.com", address);
```

위 코드에서 두 개의 큰 바이트 값에 적용된 캐스팅을 주의해서 보자.  

다른 팩토리 메소드와는 달리 이 두 메소드는 실제 호스트가 존재하는지 또는 호스트네임과 IP 주소가 올바른 연결이 맞는지 보장하지 않는다. 두 메소드는 주소 인자로 전달된 바이트 배열이 (4 또는 16바이트가 아닌) 잘못된 크기인 경우에만 UnknownHostException을 발생시킨다. 이 두 메소드는 도메인 네임 서버를 이용할 수 없는 환경이거나, 정확하지 않은 임의의 정보로 InetAddress 객체를 만들고자 할 경우 유용하게 사용할 수 있다.  

#### 1.1. 캐시
<br/>
DNS 검색(lookup)은 최종 쿼리 서버에 도착하기까지 여러 중간 단계의 서버를 거쳐야 하고 종종 중간에 손실되는 경우도 있기 때문에 상대적으로 비용이 많이 드는 명령에 해당한다. 그래서 InetAddress 클래스는 검색의 결과를 저장(cache)한다. InetAddress 클래스는 특정 호스트에 대한 주소를 검색하여 저장한 다음에는 추가 요청에 대해서 다시 검색하지 않는다. 특히 같은 호스트에 대한 새로운 InetAddress 객체를 만든 경우에도 마찬가지다. 작성한 프로그램이 실행 중인 동안에 해당 호스트의 IP가 변경되지 않는다면 문제가 되지 않는다.  

호스트를 찾을 수 없는 에러가 발생할 경우에는 약간 복잡하다. 호스트에 대한 검색을 처음 시도하면 종종 실패하기도 하지만, 곧바로 다시 시도하면 성곧하는 경우가 많다. 이러한 상황은 보통 첫 번째 시도가 원격 DNS 서버로부터 전송 중에 타임아웃이 되어 에러가 발생하고 그러는 사이에 검색 결과가 로컬 서버에 도착한다. 그리고 뒤이은 재시도는 로컬 서버에 저장된 결과를 바로 이용하여 성공하게 된다. 이러한 이유로 자바는 실패한 DNS 쿼리에 대해서는 10초 동안만 저장한다.  

그리고 이 시간은 networkaddress.cache.ttl 그리고 networkaddress.cache.negative.ttl 두 개의 시스템 속성으로 제어된다. networkaddress.cache.ttl은 성공한 DNS 쿼리에 대해 자바가 저장할 시간을 초 단위로 설정하고 networkaddress.cache.negative.ttl는 실패한 DNS 쿼리에 대해 자바가 저장할 시간을 초 단위로 설정한다. 저장 시간 내에 같은 호스트에 대해 검색을 다시 시도할 경우 항상 같은 값이 반환되며 networkaddress.cache.negative.ttl에 -1을 설정할 경우 타임아웃 없이 영원히 저장된다.  

InetAddress 클래스가 내부에 자체적으로 저장하는 것 외에도, 로컬 호스트, 로컬 도메인 네임 서버 그리고 인터넷 여기저기에 있는 다른 DNS 서버들 또한 다양한 쿼리에 대한 결과를 저장하고 있다. 자바로는 이러한 곳에 저장된 애용을 제어할 방법이 없다. 그래서 결국, 호스트네임에 대한 IP 주소의 변경이 인터넷을 통해 확산되는 데 몇 시간씩 걸리기도 한다. 그동안에 여러분이 작성한 프로그램은 DNS의 변경 사항에 따라서 UnknownHostException, NoRouteToHostException, 그리고 ConnectException을 포함한 다양한 예외 사항이 발생할 수 있다.  

#### 1.2. IP 주소로 검색하기
<br/>
IP 주소 문자열을 인자(argument)로 getByName() 메소드를 호출하면, 이 메소드는 DNS에 확인 없이 요청된 IP 주소에 대한 InetAddress 객체를 만든다. 이것은 바로 실제 존재하지 않고 접속할 수 없는 호스트에 대한 InetAddress 객체를 생성하는 것이 가능하다는 의미다. 그리고 이렇게 생성된 InetAddress 객체의 호스트네임은 이 객체를 생성할 때 인자로 넘겨준 IP 주소로 설정된다. 그리고 실제 호스트네임을 위한 DNS 검색은 getHostName()을 통해 명시적으로 호스트네임이 요청될 때만 수행된다. 이것이 바로 마침표로 구분된 네 자리 주소인 208.201.239.37로부터 www.oreilly.com이 검색되는 방법이다. 호스트네임이 요청되고 DNS 검색이 수행될 때, 지정된 IP 주소의 호스트를 찾을 수 없을 경우 호스트네임 값은 기존에 마침표로 구분된 네 자리 주소로 남아 있게 된다. 그러나 UnknownHostException은 발생하지 않는다.  

호스트네임을 사용하는 것이 IP를 직접 사용하는 것보다 좀 더 안정적이다. 일반적인 서비스들은 서비스를 제공하는 동안 호스트네임이 변경되는 일은 잘 발생하지 않지만, 그에 비해 서비스 중에 IP가 변경하는 일은 드물지 않게 발생한다. 여러분이 www.oreilly.com 같은 호스트네임 또는 208.201.239.37 같은 IP 주소 사이에서 선택해야 할 상황이 발생한다면 항상 호스트네임을 선택하는 것이 좋다. IP 주소는 호스트네임을 사용할 수 없는 상황에서만 사용하도록 하자.  


#### 1.3. 보안 이슈
<br/>
호스트네임으로부터 새로운 InetAddress 객체를 생성하는 일은 DNS 검색(lookup)을 필요로 하기 때문에 잠재적인 보안 문제에 노출되어 있다. 기본 보안 관리자(security manager)의 통제 아래에 있는 신뢰되지 않은 애플릿은 애플릿을 제공한 호스트에 대한 IP 주소만을 검색할 수 있으며, 경우에 따라 로컬 호스트에 대한 주소를 얻는 것이 가능하기도 하다. 신뢰되지 않은 코드는 다른 호스트네임에 대한 InetAddress 객체의 생성이 허가되지 않는다. 이러한 사실은 InetAddress.getByName() 메소드, InetAddress.getAllByName() 메소드, InetAddress.getLocalHost() 메소드 또는 다른 어떤 메소드를 사용해도 마찬가지다. 비록 신뢰되지 않은 코드라도 문자열 형식의 IP 주소를 사용하면 InetAddress 객체를 생성할 수 있지만, 이 객체도 해당 주소에 대해 DNS 검색은 할 수 없다.  

신뢰되지 않은 코드는 코드베이스(codebase) 이외의 다른 호스트에 대한 네트워크 연결을 만드는 것이 금지되어 있기 때문에 제3의 호스트에 대한 임의의 DNS 검색이 허가되지 않는다. 임의의 DNS 검색은 제3의 호스트와 통신을 원하는 프로그램에 의해 비밀 채널을 여는 용도로 사용될 수 있기 때문이다. 예를 들어, www.bigisp.com으로부터 설치된 애플릿이 다음 "macfaq.dialup.cloup9.net is vulnerable" 메시지를 crackersinc.com으로 보내려고 한다고 가정해보자. 이를 위해서 단지 macfaq.dialup.cloud9.net.is.vulnerable.crackersinc.com에 대한 DNS 정보를 요청하기만 하면 된다. 그러면 애플릿은 호스트네임을 검색하기 위해 로컬 DNS 서버에 접속할 것이다. 비록 존재하지 않는 호스트지만, 크래커는 crackersinc.com으로부터 DNS 에 접속할 것이다. 비록 존재하지 않는 호스트지만, 크래커는 crackersinc.com으로부터 DNS 에러 로그를 통해 크래킹에 사용할 특정 메시지를 가져올 수 있게 된다. 이러한 방식은 압축, 오류 정정, 암호화, 또 다른 사이트로 메시지를 메일로 보내는 사용자 정의 DNS 서버 등의 요소에 의해 좀 더 복잡해질 수 있지만, 이 상태로도 보안 위협의 개념을 증명하기에는 충분하다. 결국 임의의 DNS 검색은 정보를 누출할 가능성 때문에 금지된다.  

신뢰되지 않은 코드도 InetAddress.getLocalHost()를 호출할 수 있다. 그러나 애플릿과 같은 이러한 환경에서 getLocalHost()는 실제 호스트네임이 아닌 항상 localhost/127.0.0.1을 반환한다. 애플릿이 실행 중인 컴퓨터가 의도적으로 방화벽 뒤에 숨겨져 있을 수 있기 때문에 애플릿은 실제 호스트네임과 주소를 찾는 것이 금지되어 있다. 이러한 경웨 애플릿은 웹서버가 이미 구하지 못한 정보를 얻기 위한 채널이 될 수 없다.  

다른 보안 검사 항목들처럼, DNS 검색 제한도 신뢰받은 코드에 대해서는 허가된다. 호스트가 DNS 검색을 할 수 있는지 여부는 SecurityManager 클래스의 checkConnect() 메소드를 사용하여 확인할 수 있다.  

+ public void checkConnect(String hostname, int port)  

이 메소드 호출 시 포트 인자로 -1을 지정하면 특정 호스트에 대한 DNS 검색이 가능한지 여부를 검사한다. (포트 인자가 -1보다 큰 경우에는 지정된 호스트의 지정된 포트에 접속이 가능한지 여부를 검사한다.) 호스트 인자는 www.oreilly.com 같은 호스트네임이거나 마침표로 구분된 208.201.239.37 같은 IP 주소일 수도 있다. 그리고 또한 FEDC::DC:0:7076:10 같은 16진수 IPv6 주소도 사용할 수 있다.  

### 2. Get 메소드
<br/>
InetAddress 클래스는 호스트네임을 문자열로 반환하고 IP 주소를 문자열과 바이트 배열로 반환하는 네 개의 Get 메소드를 제공한다.  

+ public String getHostName()
+ public String getCanonicalHostName()
+ public byte[] getAddress()
+ public String getHostAddress()  

그러나 이에 대응하는 setHostName() 그리고 setAddress() 메소드는 제공되지 않으며, java.net 이외의 패키지에서는 InetAddress 객체가 생성된 이후에 객체의 필드를 변경할 수 없다. 이러한 특징은 InetAddress 클래스를 변경 불가능하고 스레드로부터 안전하게 만든다.  

getHostName() 메소드는 InetAddress 객체의 의해 표현되는 IP 주소에 해당하는 호스트네임을 포함한 String을 반환한다. 만약 장비가 호스트네임을 가지고 있지 않거나 보안 관리자가 이름 검색을 막을 경우, 마침표로 구분된 네 자리 IP 주소가 반환된다. 이 메소드의 사용 방법은 다음과 같다.

```java
InetAddress machine = InetAddress.getLocalHost();
String localhost = machine.getHostName();
```

getCanonicalHostName() 메소드도 이와 유사하지만 좀 더 적극적으로 DNS에 접근한다. getHostName()은 호스트네임을 모르고 있다고 판단될 때만 DNS에 접근하지만, getCanonicalHostName은 가능하면 DNS에 요청하여 정보를 가져오며 이미 저장된 호스트네밍이 있는 경우 갱신한다. 이 메소드이 사용 방법은 다음과 같다.  

```java
InetAddress machine = InetAddress.getLocalHost();
String localhost = machine.getCanonicalHostName();
```

getCanonicalHostName() 메소드는 특히 호스트네임보다는 마침표로 구분된 네 자리 IP 주소에 대해 특히 유용하다.  

```java
InetAddress ia = InetAddress.getByName("208.201.239.100");
System.out.println(ia.getCanonicalHostName());
```

getHostAddress() 메소드는 IP 주소를 마침표로 구분된 네 자리 형식의 문자열로 반환한다.  

```java
try {
  InetAddress me = InetAddress.getLocalHost();
  String dottedQuad = me.getHostAddress();
  System.out.println("My address is " + dottedQuad);
} catch (UnknownHostException ex) {
  System.out.println("I'm sorry. I don't know my own address.")
}
```

장비의 IP 주소 확인이 필요한 경우 드물긴 하지만 네트워크 바이트 순서의 바이트 배열로 IP 주소를 반환하는 getAddress() 메소드를 사용할 수도 있다. 최상위 바이트(마침표로 구분된 네 자리 형식 주소의 첫 바이트)가 배열의 처음이나 0번째 요소에 위치한다. IPv6 주소 사용을 고려한다면 이 배열의 길이에 대해서 추측하지 말고 배열의 길이 필드를 사용해야 한다.  

```java
InetAddress me = InetAddress.getLocalHost();
byte[] address = me.getAddress();
```

반환된 바이트 배열은 부호 없는 바이트 값이며, 부호 없는 바이트는 잠재적인 문제를 내포하고 있다. C와는 달리 자바는 기본 데이터 타입으로 부호 없는 바이트를 제공하지 않는다. 그래서 바이트 배열에서 127보다 큰 값은 음수로 다뤄진다. 그 결과 getAddress()에서 반환된 바이트 배열로 어떤 작업을 하려고 할 경우, byte를 int로 늘린 후 처리해야 한다. 아래는 byte 값을 int로 반환하는 방법 중 하나다.  

```java
int unsigendByte = sigendByte < 0 ? signedByte + 256: signedByte;
```

여기서 signedByte는 양수이거나 음수일 수 있다. 조건 연선자 ?는 signedByte가 음수인지 검사한 다음 음수일 경우 signedByte 값에 256을 더하여 양수로 만든다. 그렇지 않은 경우, 즉 signedByte가 0보다 크거나 같은 경우에는 변경 없이 그대로 둔다. 이 코드에서 signedByte는 256을 더하기 전에 자동으로 정수 타입으로 변환되기 때문에 바이트 범위를 넘어서는 값이 버려지는 문제(wraparound)는 발생하지 않는다.  

바이트 배열 상태의 IP 주소를 살펴보게 되는 경우는 흔치 않지만 주소의 타입을 확인해야 할 경우 사용된다. getAddress()에 의해 반환되는 바이트 배열의 숫자를 검사하면 처리하는 주소가 IPv4인지 IPv6인지 확인할 수 있다. 다음은 IP 주소의 버전을 확인하는 예제다.  

```java
public class AddressTest {
  public static int getVersion(InetAddress ia) {
    byte[] address = ia.getAddress();
    if (address.length == 4) return 4;
    else if (address.length = 16) return 16;
    else return -1;
  }
}
```

### 3. 주소 타입
<br/>
자바는 InetAddress 객체가 어느 타입의 주소에 속하는지 검사하기 위한 10개의 메소드를 제공한다.  

+ public boolean isAnyLocalAddress()
+ public boolean isLoopbackAddress()
+ public boolean isLinkLocalAddress()
+ public boolean isSiteLocalAddress()
+ public boolean isMulticastAddress()
+ public boolean isMCGlobal()
+ public boolean isMCNodeLocal()
+ public boolean isMCLinkLocal()
+ public boolean isMCSiteLocal()
+ public boolean isMCOrgLocal()  

isAnyLocalAddress() 메소드는 인자로 전달된 주소가 와일드카드(wildcard) 주소인 경우 true를 반환하고, 그렇지 않을 경우 false를 반환한다. 와일드카드 주소는 로컬 시스템의 어떤 주소와도 매치된다. 이 메소드는 시스템이 다수의 이더넷 카드가 있거나 이더넷 카드와 802.11 무선랜 인터페이스가 함께 있는 경우처럼 다수의 네트워크 인터페이스가 있을 때 중요하게 사용된다. IPv4에서 와일드카드 주소는 0.0.0.0이고 IPv6에서 와일드카드 주소는 0:0:0:0:0:0:0:0이다. (줄여서 ::으로도 쓸 수 있다.)  

isLoopbackAddress() 메소드는 인자로 전달된 주소가 루프백 주소인 경웨 true를 반환하고 그렇지 않을 경우 false를 반환한다. 루프백 주소는 어떠한 물리적인 하드웨어 장치 없이 IP 계층을 통해서 프로그램이 실행 중인 컴퓨터에 직접 연결할 수 있다. 게다가 루프백 주소로 연결함으로써 잠재적인 버그나 존재하지 않는 이더넷, PPP, 그리고 다른 드라이버들을 우회하여 테스트할 수 있으며, 환경 및 다른 요인들로부터 문제를 격리시키는 데 도움이 된다. 루프백 주소에 대한 연결은 동일한 시스템에 할당된 일반적인 IP 주소로 연결하는 것과는 차이가 있다. IPv4에서 루프백 주소는 127.0.0.1이 사용되고 IPv6에서는 0:0:0:0:0:0:0:1이 사용된다. (줄여서 ::1로도 쓸 수 있다.)  

isLinkLocalAddress() 메소드는 인자로 전달된 주소가 IPv6 링크로컬 주소(link-local address)일 경우 true를 반환하고 그렇지 않을 경우 false를 반환한다. 이 주소는 IPv6 네트워크에서 자동 설정 목적으로 사용되며 IPv4 네트워크에서 DHCP와 유사하지만 서브를 필요로 하지 않는다. 라우터는 링크로컬 주소로 지정된 패킷에 대해서는 로컬 서브넷을 넘어서 전달하지 않는다. 모든 링크로컬 주소는 FE80:0000:0000:0000의 8바이트로 시작한다. 나머지 8바이트는 로컬 주소로 채워지며, 종종 이더넷 카드 제조사로부터 할당받은 이더넷 맥(MAC) 주소에서 복사된다.  

isSiteLocalAddress() 메소드는 인자로 전달된 주소가 IPv6 사이트로컬 주소(site-local address)일 경우 true를 반환하고 그렇지 않을 경우 false를 반환한다. 사이트로컬 주소는 로컬링크 주소와 유사하나 로컬링크와는 달리 라우터에 의해 사이트 또는 캠퍼스 내에서 전달될 수 있고 사이트를 넘어서 전달될 수 없다는 점이 다르다. 사이트로컬 주소는 FEC0:0000:0000:0000의 8바이트로 시작한다. 나머지 8바이트는 로컬 주소로 채워지며, 종종 이더넷 카드 제조사로부터 할당받은 이더넷 맥(MAC) 주소에서 복사된다.  

isMulticastAddress() 메소드는 인자로 전달된 주소가 멀티캐스트 주소일 경우 true를 반환하고 그렇지 않을 경우 false를 반환한다. 멀티캐스팅은 특정한 하나의 컴퓨터가 아닌 등록된 모든 컴퓨터에게 데이터를 전송한다. IPv4에서 멀티캐스트 주소는 224.0.0.0에서 239.255.255.255 범위를 가진다. IPv6에서 멀티캐스트 주소는 바이트 FF로 시작한다.  

isMCGlobal() 메소드는 인자로 전달된 주소가 글로벌 멀티캐스트 주소일 경우 true를 반환하고 그렇지 않을 경우 false를 반환한다. 글로벌 멀티캐스트 주소는 전 세계를 범위로 가입자를 가질 수 있다. 모든 멀티캐스트 주소는 FF로 시작하며 IPv6에서 글로벌 멀티캐스트 주소는 잘 알려진 영구 할당 주소인지 또는 임시 할당 주소인지에 따라 FF0E 또는 FF1E로 시작한다. IPv4에서 모든 멀티캐스트 주소는 글로벌 범위를 가지며 이 메소드도 이러한 내용을 충분히 고려하고 있다. IPv4는 범위를 제어가힉 위해 주소보다는 TTL(time-to-live)값을 사용한다.  

isMCOrgLocal() 메소드는 인자로 전달된 주소가 조직 또는 단체 범위(organization-wide)의 멀티캐스트 주소일 경우 true를 반환하고 그렇지 않을 경우 false를 반환한다. 조직 또는 단체 범위 멀티캐스트 주소는 회사 또는 단체의 모든 사이트를 범위로 가입자를 가질 수 있으나 조직이나 단체 바깥에서는 가입할 수 없다. 조직 로컬 멀티캐스트 주소는 잘 알려진 영구 할당 주소인지 또는 임시 할당 주소인지에 따라 FF08 또는 FF18으로 시작한다.  

isMCSiteLocal() 메소드는 인자로 전달된 주소가 사이트 범위(site-wide)의 멀티캐스트 주소일 경우 true를 반환하고 그렇지 않을 경우 false를 반환한다. 사이트 범위 주소가 지정된 패킷만이 패킷이 포함된 로컬 사이트 안에서 전송된다. 사이트 범위 멀티캐스트 주소는 잘 알려진 영구 할당 주소인지 또는 임시 할당 주소인지에 따라 FF08 또는 FF15으로 시작한다.  

isMCLinkLocal() 메소드는 인자로 전달된 주소가 서브넷 범위 멀티캐스트 주소일 경우 true를 반환하고 그렇지 않을 경우 false를 반환한다. 링크로컬 주소로 지정된 패킷만이 패킷이 포함된 서브넷 안에서 전송된다. 링크로컬 멀티캐스트 주소는 잘 알려진 영구 할당 주소인지 또는 임시 할당 주소인지에 따라 FF02 또는 FF12로 시작한다.  

isMCNodeLocal() 메소드는 인자로 전달된 주소가 인터페이스로컬(interface-local) 멀티캐스트 주소일 경우 true를 반환하고 그렇지 않을 경우 false를 반환한다. 인터페이스로컬 주소로 지정된 패킷은 자신이 만들어진 네트워크 인터페이스를 넘어 전송되지 않으며, 같은 노드에 있는 다른 네트워크 인터페이스조차 전송되지 않는다. 주로 네트워크 디버깅이나 테스트를 목적으로 사용된다. 인터페이스로컬 멀티캐스트 주소는 잘 알려진 영구 할당 주소인지 또는 임시 할당 주소인지에 따라 FE01 또는 FE11로 시작한다.  

메소드 이름이 지금 널리 사용되는 용어들과는 다소 차이가 있다. IPv6 프로토콜의 초기 문서들은 이러한 종류의 주소를 "노드로컬(node-local)" 주소라고 불렀고 그래서 메소드 이름이 "isMCNodeLocal"이 된 것이다. IPNG 작업 그룹(working group)은 실제로 JDK에 이 메소드가 추가되기 전에 이름을 변경했지만 썬 마이크로시스템즈는 이 사항을 제때 전달받지 못했다.  

### 4. 도달 가능성 검사하기
<br/>
InetAddress 클래스는 현재의 호스트로부터 특정 노드가 접근 가능한지 (예들 들어, 내트워크 연결을 맺을 수 있는지) 검사하는 두 개의 isReachable() 메소드를 제공한다. 실제 연결 시도시에 연결은 방화벽, 프록시 서버, 라우터 오작동 그리고 회선 손상 또는 단순히 원격 호스트가 꺼져 있는 등 많은 이유로 차단될 수 있다.  

+ public boolean isReachable(int timeout) throws IOException
+ public boolean isReachable(NetworkInterface interface, int ttl, int timeout) throws IOException  

이 두 메소드는 지정된 주소가 도달 가능한지 확인하기 위해 traceroute의 사용(좀 더 정확하게는 ICMP 에코 요청)을 시도한다. 호시트가 타임아웃 이내에 응답할 경우, 이 메소드는 true를 반환하고, 그렇지 않을 경우 false를 반환한다. 네트워크 에러가 발생할 경우에는 IOException이 발생한다. 두 번째 변형 메소드는 또한 연결을 시도할 로컬 네트워크 인터페이스와 TTL 지정이 가능하다[TTL은 패킷이 버려지지 않고 갈 수 있는 최대 네트워크 홉(hop)을 의미한다].  

### 5. 객체 메소드
<br/>
다른 모든 클래스처럼, java.net.InetAddress 클래스 역시 java.lang.Object를 상속한다. 따라서 java.net.InetAddress 클래스는 java.net.Object 클래스에 있는 모든 메소드를 사용할 수 있다. java.net.InetAddress는 좀 더 특별한 기능을 제공하기 위해 아래 세 가지 메소드를 오버라이드한다.  

+ public boolean equals(Object o)
+ public int hashCode()
+ public String toString()  

equals() 메소드는 전달된 객체가 InetAddress 클래스의 객체이고 같은 IP 주소를 가리키고 있으면 true를 반환한다. 그러나 호스트네임까지 같을 필요는 없다. 예를 들어, www.ibiblio.org에 대한 InetAddress 객체와 www.cafeaulait.org에 대한 InetAddress 객체는 같은 IP 주소를 가리키기 때문에 true를 반환한다. www.ibiblio.org와 helios.ibiblio.org에 대한 InetAddress 객체를 만들고 이 두 객체가 같은 장비인지 확인한다.  

```java
import java.net.*;

public class IBiblioAliases {

  public static void main (String args[]) {
    try {
      InetAddress ibiblio = InetAddress.getByName("www.ibiblio.org");
      InetAddress helios = InetAddress.getByName("helios.ibiblio.org");
      if (ibiblio.equals(helios)) {
        System.out.println("www.ibiblio.org is the same as helios.ibiblio.org");
      } else {
        System.out.println("www.ibiblio.org is not the same as helios.ibiblio.org");
      }
    } catch (UnknownHostException ex) {
      System.out.println("Host lookup failed.");
    }
  }
}
```

hashCode() 메소드는 equals() 메소드와 일관된 메소드다. hashCode() 메소드는 IP 주소로부터 계산된 int 값을 반환하며 호스트네임은 계산에 사용되지 않는다. 두 개의 InetAddress 객체가 같은 IP 주소를 가지고 있다면 서로 호스트네임이 다르더라도 같은 해시 코드 값을 반환한다.  

잘 정의된 다른 클래스들처럼 java.net.InetAddress 클래스도 객체의 호스트네임이나 IP 주소에 대한 문자열을 반환하는 toString() 메소드를 제공한다. System.out.println()에 InetAddress 객체를 전달할 때 은연중에 이 메소드가 호출되고 있다. 이미 보았다시피 toString() 메소드가 생성하는 문자열은 다음과 같은 형태다.  

```
호스트네임/마침표로 구분된 네 자리 IP주소
```

모든 InetAddress 객체가 호스트네임을 가지는 건 아니다. 자바 1.3과 그 이전 버전에서는 호스트네임이 없는 경우 마침표로 구분된 네 자리 IP 주소로 대체되며, 자바 1.4와 그 이후 버전에서는 호스트네임이 빈 문자열로 설정된다.
