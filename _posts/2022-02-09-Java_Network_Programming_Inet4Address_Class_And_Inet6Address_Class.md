---
title:  Inet4Address 클래스와 Inet6Address 클래스
categories:
- Java Network Programming
feature_text: |
  ## Inet4Address 클래스와 Inet6Address 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

자바는 IPv4 주소와 IPv6 주소를 구별하기 위해 Inet4Address와 Inet6Address 두 클래스를 사용한다.  

+ public final class Inet4Address extends InetAddress
+ public final class Inet6Address extends InetAddress  

대부분의 경우 해당 주소가 IPv4인지 IPv6인지 신경 쓰지 않아도 된다. 자바 프로그램이 실행되는 애플리케이션 계층에서는 특히 필요한 경우는 거의 발생하지 않는다(그리고 심지어 해당 객체가 IPv4인지 IPv6인지 확인이 필요한 경우에도 instanceof 연산자를 사용하는 것보다 getAddress() 메소드에 의해 반환되는 바이트 배열의 크기를 검사하는 것이 더 빠르다). Inet4Address 클래스는 InetAddress에 있는 몇몇 메소드를 오버라이드했으나 이 메소드의 일반적인 동작방식을 변경하지는 않는다. Inet6Address 클래스도 이와 비슷하지만 슈퍼클래스에서는 제공되지 않는 새로운 메소드 isIPv4CompatibleAddress()를 추가했다.  

+ public boolean isIPv4CompatibleAddress()  

이 메소드는 InetAddress 객체가 IPv4와 호환 가능한 IPv6 주소인 경우 true를 반환한다. 이 말은 IPv6 객체이면서 0:0:0:0:0:0:0:xxxx와 같이 마지막 4바이트만 0이 아닌 주소에 대해서 true를 반환한다는 의미다. 이러한 주소에 대해서 getBytes() 메소드로 반환받은 배열에서 4바이트를 떼어 내서 Inet4Address 객체를 생성하는 데 사용할 수도 있지만 그렇게까지 할 필요는 없다.
