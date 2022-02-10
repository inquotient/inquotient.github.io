---
title:  ProxySelector 클래스
categories:
- Java Network Programming
feature_text: |
  ## ProxySelector 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

실행 중인 각각의 자바 가상 머신에는 서로 다른 네트워크 연결에 필요한 프록시 서버를 찾기 위해 사용되는 하나의 java.net.ProxySelector 객체가 있다. 기본적으로 설정된 ProxySelector는 단지 시스템 설정과 URL의 프로토콜만을 살펴보고 다른 호스트에 대한 연결 방법을 결정한다. 그러나 ProxySelector 클래스의 서브클래스를 작성하여 기본 셀렉터를 변경할 수 있다. 그리고 프로토콜, 호스트, 경로, 시간 등의 조건에 기반하여 다른 프록시를 선택하도록 할 수 있다.  

이 클래스의 핵심은 추상 select() 메소드다.  

+ public abstract List<Proxy> select(URI uri)  

자바는 이 메소드에 연결할 호스트에 대한 URI 객체(URL 객체가 아니다)를 전달한다. URL 클래스로 만든 연결의 경우 이 객체는 일반적으로 http://www.example.com 또는 ftp://ftp.example.com/pub/files와 같은 형태로 구성된다. Socket 클래스로 만든 순수 TCP 연결의 경우 이 URI는 socket://host:port와 같은 형태로 구성된다. 예를 들어, socket://www.example.com:80. 그때 ProxySelector 객체는 이 전달된 URI 객체의 타입에 적절한 프록시를 선택하여 List<Proxy> 타입으로 반환한다.  

이 클래스의 두 번째 추상 메소드인 connectFailed()는 직접 구현해서 사용해야 한다.  

+ public void connectFailed(URI uri, SocketAddress address, IOEXception ex)  

이 메소드는 콜백 메소드며, 프록시 서버가 연결을 만들 수 없을 때 프로그램에게 알리기 위해 사용된다.  

ProxySelector를 변경하기 위해서 다음 코드처럼 정적 ProxySelector.setDefault() 메소드에 새로운 셀렉터를 전달하면 된다.  

```java
ProxySelector selector = new LocalProxySelector();
ProxySelector.setDefault(selector);
```

이 메소드가 호출된 시점부터는 해당 가상 머신에 의해 열린 모든 커넥션은 사용할 올바른 프록시를 선택하기 위해 변경된 ProxySelector에게 요청하게 된다. 그리고 일반적으로 공유된 환경에서 실행 중인 코드에서는 이 메소드를 호출하지 않도록 해야 한다. 예를 들어, 서블릿에서 ProxySelector를 변경할 경우 동일한 컨테이너에서 실행 중인 모든 서블릿의 ProxySelector를 변경하게 되므로 서블릿에서 ProxySelector를 변경하지 않도록 해야 한다.
