---
title:  Proxy 클래스
categories:
- Java Network Programming
feature_text: |
  ## Proxy 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

많은 시스템들이 프록시를 통해 웹과 웹이 아닌 다른 인터넷 서비스들을 접근한다. 프록시 서버는 로컬 클라이언트로부터 원격 서버에 대한 요청을 받는다. 그리고 프록시 서버는 원격 서버에 요청하고 결과를 다시 로컬 클라이언트에게 전달한다. 때로는 로컬 네트워크 구성과 같은 사적인 정보의 외부 노출을 방지하기 위해 프록시를 사용한다. 또 다른 경우에는 외부 사이트에 대한 요청을 필터링하여 금지된 사이트의 접근을 차단하기 위해 사용한다. 예를 들어, 초등학교에서 http://www.playboy.com과 같은 사이트에 대한 접근 차단이 필요한 경우가 있다. 그리고 또 다른 경우에는, 다수의 로컬 사용자로부터 같은 페이지에 대한 반복된 요청을 매번 원격 서버에서 다운로드하지 않고 로컬 캐시로부터 다운로드받게 하여 순수하게 성능을 향상시킬 목적으로 사용한다.  

Proxy 클래스는 자바 프로그램 내에서 더 세밀하게 프록시 서버를 제어할 수 있게 한다. 특히 원격 호스트마다 다른 프록시를 사용하도록 설정할 수 있다. 자바 코드 내에서 프록시 서버 자체는 java.net.Proxy 클래스의 인스턴스로 표현된다.  

여전히 HTTP, SOCKS 그리고 직접 연결(프록시를 사용하지 않음)을 포함한 세 종류의 프록시만 존재하며, enum 타입인 Proxy.Type 안에 세 가지 상수로 표현된다.  

+ Proxy.Type.DIRECT
+ Proxy.Type.HTTP
+ Proxy.Type.SOCKS  

타입 이외에, 프록시 서버의 주소와 포트 같은 추가적인 정보들은 SocketAddress 객체로 제공된다. 예를 들어, 아래 코드는 proxy.example.com 호스트의 80포트에 대한 HTTP 프록시 서버를 나타내는 Proxy 객체를 생성한다.  

```java
SocketAddress address = new InetSocketAddress("proxy.example.com", 80);
Proxy proxy = new Proxy(Proxy.Type.HTTP, address);
```

비록 프록시 객체의 종류는 세 가지밖에 없지만, 호스트마다 다른 프록시를 사용하기 위해 같은 타입의 다수의 프록시를 사용할 수 있다.  

### 1. 시스템 속성
<br/>
기본적인 프록시의 사용을 위해, 몇몇 시스템 속성이 로컬 프록시 서버의 주소를 가리키도록 설정하면 된다. 만약 순수 HTTP 프록시를 사용하는 경우 http.proxyHost 설정을 프록시 서버의 도메인 이름이나 IP 주소로 설정하고 http.proxyPort 설정을 프록시 서버의 포트로 설정한다(기본값은 80). 이 설정을 변경하는 방법으로는 자바 코드 내에서 System.setProperty() 메소드를 호출하거나 프로그램 실행 시에 -D 옵션을 사용하는 방법을 포함하여 다양한 방법이 있다.  

아래 예제는 프록시 서버를 192.168.254.254 그리고 포트를 9000으로 설정한다.  

```shell
java -Dhttp.proxyHost=192.168.254.254 -Dhttp.proxyPort=9000 com.domain.Program
```

사용자 이름과 패스워드를 필요로 하는 프록시의 경우 Authenticator를 설치해야 한다.  

만약 프록시가 설정된 환경에서 특정 호스트를 프록시를 통하지 않고 직접 연결하기 원할 경우 http.nonProxyHosts 시스템 속성을 해당 서버의 호스트네임이나 IP 주소로 설정한다. 하나 이상의 호스트를 설정해야 할 경우 버티컬바(|)로 호스트네임을 구분한다.  

```java
System.setProperty("http.proxyHost", "192.168.254.254");
System.setProperty("http.proxyPort", "9000");
System.setProperty("http.nonProxyHosts", "java.oreilly.com|xml.oreilly.com");
```

특정 도에인 내의 모든 호스트나 서브도메인을 프록시 사용에서 배제해야 할 경우 와일드 카드로 별표를 사용한다. 예를 들어, 다음 명령은 oreilly.con 도메인 내의 호스트만 프록시 사용에서 배제한다.  

```shell
java -Dhttp.proxyHost=192.168.254.254 -Dhttp.nonProxyHosts=*.oreilly.com com.domain.Program
```

FTP 프록시 서버를 사용할 경우 ftp.proxyHost, ftp.proxyPort, 그리고 ftp.nonProxyHosts 설정을 같은 방법으로 한다.  

자바는 HTTP와 FTP 이외의 다른 애플리케이션 계층 프록시를 지원하지 않지만, 모든 TCP 연결을 위한 전송 계층 SOCKS 프록시를 사용할 경우 socksProxyHost 그리고 socksProxyPort 시스템 속성으로 프록시를 설정할 수 있다. 자바는 SOCKS 프록시에 대해서는 특정 호스트를 배제시키는 옵션을 제공하지 않으므로 모든 접근이 프록시를 사용하거나 사용하지 않도록 결정된다.
