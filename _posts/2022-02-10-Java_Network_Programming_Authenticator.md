---
title:  Authenticator 클래스
categories:
- Java Network Programming
feature_text: |
  ## Authenticator 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

java.net 패키지는 HTTP 인증으로 보호된 사이트에 사용자 이름과 패스워드를 제공하기 위해 Authenticator 클래스를 제공한다.  

+ public abstract class Authenticator extends Object  

Authenticator는 추상 클래스이므로 서브클래스를 통해서 사용해야 한다. Authenticator 서브클래스의 구현에 따라 다양한 방법으로 정보를 입력받을 수 있다. 예를 들어, 콘솔 프로그램은 System.in으로 사용자 이름과 패스워드를 입력받는다. GUI 프로그램은 대화상자를 통해 입력받으며, 자동화된 프로그램은 암호화된 파일로부터 사용자 이름을 입력받을 수 있다.  

여러분이 작성한 Authenticator의 서브클래스를 Authenticator.seetDefault() 정적 메소드의 인자로 전달하여 기본 인증자(authenticator)로 설치할 수 있다.  

+ public static void setDefault(Authenticator a)  

예를 들어, DialogAuthenticator라는 이름의 서브클래스를 작성한 경우 아래와 같이 설치한다.  

```java
Authenticator.setDefault(new DialogAuthenticator());
```

이 코드는 한 번만 실행하면 된다. 이 시점부터 URL 클래스는 사용자 이름과 패스워드가 필요할 때, Authenticator.requestPasswordAuthentication() 정적 메소드를 사용하여 서브클래스 DialogAuthenticator에게 요청한다.  

+ public static PasswordAuthentication requestPasswordAuthentication(InetAddress address, int port, String protocol, String prompt, String scheme) throws SecurityException  

address 인자는 인증이 필요한 호스트의 주소이며, port 인자는 해당 호스트의 포트 번호다. 그리고 protocol 인자는 해당 사이트에 접근할 수 있는 애플리케이션 계층 프로토콜이다. HTTP 서버는 prompt를 제공한다. prompt 인자는 일반적으로 인증이 필요한 영역(realm)의 이름을 나타낸다. (www.ibiblio.org와 같은 규모가 큰 웹 서버에는 다수의 영역이 있고, 각 영역은 서로 다른 사용자 이름과 패스워드를 요구한다.) scheme 인자는 사용되는 인증 스킴이다. (여기서 말하는 스킴은 프로토콜에서 말하는 스킴이 아닌 HTTP 인증 스킴을 말한다. 일반적으로 basic을 사용한다.)  

신뢰할 수 없는 애플릿은 이름과 패스워드를 요청할 수 없다. 그리고 신뢰할 수 있는 애플릿은 요청할 수는 있지만, 해당 애플릿이 requestPasswordAuthentication NetPermission을 소유하고 있을 때만 가능하다. 그렇지 않은 경우 Authenticator.requestPasswordAuthentication()은 SecurityException 예외를 발생시킨다.  

Authenticator 서브클래스는 getPasswordAuthentication() 메소드를 꼭 오버라이드(override)해야 한다. 이 메소드 안에서 여러분은 사용자 또는 다른 경로로부터 이름과 패스워드를 수집하고, 수집한 내용을 java.net.PasswordAuthentication 클래스의 인스턴스로 반환해야 한다.  

+ protected PasswordAuthentication getPasswordAuthentication()  

특정 요청에 대해 인증을 원하지 않을 경우 널(null)을 반환할 수 있다. 이때 자바는 서버에게 해당 연결에 대한 인증 방법을 알 수 없다고 알린다. 잘못된 사용자 이름 또는 패스워드를 제출할 경우, 자바는 다시 getPasswordAuthentication() 메소드를 호출하여 올바른 데이터를 제공할 추가적인 기회를 준다. 일반적으로 다섯 번의 재입력 기회가 있다. 그 후에 openStream() 메소드는 ProtocolException 예외를 발생시킨다.  

사용자 이름과 패스워드는 동일한 가상 머신 세션에 캐시된다. 일단 해당 영역(realm)에 대한 올바른 인증 정보가 설정된 이후에는 인증을 다시 요청하지 않도록 해야 한다. 그러나 패스워드 영역이 0으로 채워져 명시적으로 삭제된 경우에는 다시 요청할 수 있다.  

요청에 대한 좀 더 자세한 정보가 필요한 경우 슈퍼클래스인 Authenticator로부터 상속받은 다음 메소드를 호출하여 추가적인 정보를 얻을 수 있다.  

+ protected final InetAddress getRequestingSite()
+ protected final int getRquestingPort()
+ protected final String getRequestingProtocol()
+ protected final String getRequestingPrompt()
+ protected final String getRequestingScheme()
+ protected final String getRequestingHost()
+ protected final String getRequestingURL()
+ protected Authenticator.RequestorType getRequestorType()  

이 메소드들은 requestPasswordAuthentication() 메소드에 대한 마지막 호출 시 제공된 정보를 반환하거나, 해당 정보가 없는 경우 널(null)을 반환한다. (getRequestPort() 메소드는 포트 정보가 없는 경우 -1을 반환한다)  

getRequestingURL() 메소드는 인증 요청 시 사용된 완전한 URL을 반환한다. - 서로 다른 파일에 대해 다른 이름과 패스워드를 사용하는 사이트에서 유용한 정보, getRequestorType() 메소드는 인증을 요청한 대상이 서버인지 프록시 서버인지를 나타내는 두 가지 명명된 상수 (즉, Authenticator.RequestorType.PROXY 또는 Authenticator.RequestorType.SERVER) 중 하나를 반환한다.
