---
title:  NetworkInterface 클래스
categories:
- Java Network Programming
feature_text: |
  ## NetworkInterface 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

NetworkInterface 클래스는 로컬 IP 주소를 나타낸다. 이 클래스는 또한 (방화벽이나 라우터에서 일반적으로 사용되는) 추가적인 이더넷 카드와 같은 물리적인 인터페이스나 장비의 또 다른 주소처럼 같은 물리적인 하드웨어에 연결된 가상 인터페이스일 수도 있다. NetworkInterface 클래스는 인터페이스에 상관없이 모든 로컬 주소를 열거하는 메소드와 NetworkInterface로부터 InetAddress 객체를 생성하는 메소드를 제공한다. 이렇게 생성된 InetAddress 객체는 소켓이나 서버 소켓 등을 만드는 데 사용될 수 있다.  

### 1. 팩토리 메소드
<br/>
NetworkInterface 객체는 물리적인 하드웨어와 가상의 주소를 표현하기 때문에 임의대로 생성할 수 없다. NetworkInterface 객체는 InetAddress 클래스처럼 특정 네트워크 인터페이스와 연관된 NetworkInterface 객체를 반환하는 정적 팩토리 메소드를 제공하여 IP 주소 또는 인터페이스의 이름으로 요청하거나 열거형(enumeration)으로 반환받을 수도 있다.  

+ public static NetworkInterface getByName(String name) throws SocketException  
getByName() 메소드는 특정 이름을 가진 네트워크 인터페이스의 NetworkInterface 객체를 반환하며 해당 이름을 가진 인터페이스가 없는 경우 널(null)을 반환한다. getByName() 메소드는 내부 네트워크 스택에서 관련된 네트워크 인터페이스를 찾다가 문제가 발생한 경우, SocketException이 발생한다. 그러나 일반적으로는 거의 발생하지 않는다.  

인터페이스의 이름은 플랫폼에 따라 다르며 일반적인 유닉스 시스템에서 이더넷 인터페이스 이름은 eth0, eth1 같은 이름을 가진다. 그리고 로컬 루프백 주소는 대부분 "lo" 같은 이름이 사용된다. 윈도우의 경우 특정 네트워크 인터페이스의 밴더와 모델명에서 유래한 "CE31" 그리고 "ELX100" 같은 이름이 사용된다. 예를 들어, 아래 예제는 유닉스 시스템에서 메인 이더넷 인터페이스를 찾는 코드다.  

```java
try {
  NetworkInterface ni = NetworkInterface.getByName("eth0");
  if (ni == null) {
    System.out.println("No such interface: eth0");
  }
} catch (SocketException ex) {
  System.out.println("Could not list sockets.");
}
```

+ public static NetworkInterface getByInetAddress(InetAddress address) throws SocketException  
getByInetAddress() 메소드는 지정된 IP 주소와 관련된 네트워크 인터페이스를 나타내는 NetworkInterface 객체를 반환한다. 이 메소드가 호출된 로컬 호스트에 해당 IP 주소와 관련된 네트워크 인터페이스가 없는 경우 널을 반환한다. 그리고 메소드 호출 시에 오류가 발생할 경우 SocketException이 발생한다. 예를 들어, 아래 예제 코드는 로컬 루프백 주소와 관련된 네트워크 인터페이스를 찾는다.  

```java
try {
  InetAddress local = InetAddress.getByName("127.0.0.1");
  NetworkInterface ni = NetworkInterface.getByInetAddress(local);
  if (ni == null) {
    System.out.println("That's weird. No local loopback address.");
  }
} catch (SocketException ex) {
  System.err.println("Could not list network interfaces.");
} catch (UnknownHostException ex) {
  System.err.println("That's weird. Could not lookup 127.0.0.1.");
}
```

+ public static Enumeration getNetworkInterfaces() throws SocketException  
getNetworkInterfaces() 메소드는 로컬 호스트의 네트워크 인터페이스 전체를 나열하는 java.util.Enumeration을 반환한다.  

### 2. Get 메소드
<br/>
NetworkInterface 객체를 구하면 바로 IP 주소와 이름을 확인할 수 있다. 그리고 이 작업이 NetworkInterface 객체를 가지고 할 수 있는 일의 거의 전부다.  

+ public Enumeration getInetAddress()  
단일 네트워크 인터페이스는 하나 이상의 IP 주소에 연결될 수 있다. 이러한 상황이 요즘에 일반적이지 않지만 충분히 발생할 수 있다. getInetAddress() 메소드는 인터페이스가 연결된 각각의 IP 주소를 위한 InetAddress 객체를 포함한 java.util.Enumeration을 반환한다.  

+ public String getName()  
getName() 메소드는 eth0 또는 lo 같은 특정 NetworkInterface 객체의 이름을 반환한다.  

+ public String getDisplayName()  
getDisplayName() 메소드는 "Ethernet Card 0"와 같은 특정 네트워크 인터페이스에 대하여 좀 더 이해하기 쉬운 방식의 이름을 반환한다. 그러나 필자의 유닉스 장비에서 이 메소드를 테스트하면 항상 getName() 같은 이름을 반환한다. 윈도우에서 테스트해 보면 "Local Area Connection" 또는 "Local Area Connection 2"같은 좀 더 친근한 이름을 볼 수 있다.
