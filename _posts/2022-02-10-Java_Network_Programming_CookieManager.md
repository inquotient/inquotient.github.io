---
title:  CookieManager 클래스
categories:
- Java Network Programming
feature_text: |
  ## CookieManager 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

자바 5는 쿠키를 저장하고 얻는 API를 정의한 추상 클래스인 java.net.CookieHandler를 제공한다. 그러나 자바 5에서 해당 추상 클래스에 대한 구현은 제공하지 않는다. 그래서 해당 클래스를 사용하기 위해서는 고단한 작업이 많이 발생한다.  

자바 6에서는 이러한 작업 없이 바로 사용할 수 있도록 CookieHandler를 서브클래싱하여 구상 클래스인 java.net.CookieManager를 제공한다. 그러나 자바에서 기본적으로 CookieManager가 활성화되어 있지 않기 때문에, 자바가 쿠키를 저장하거나 반환받기 전에 활성화시켜야 한다.  

```java
CookieManager manager = new CookieManager();
CookieHandler.setDefault(manager);
```

특정 사이트로부터 쿠키를 받거나 보내기만 한다면 이것으로 충분하다. 위 두 줄이 쿠키를 보내거나 받는 데 필요한 전부다. 이 코드처럼 CookieManager를 설치한 뒤에는, 자바는 URL 클래스를 사용하여 연결한 모든 HTTP 서버로부터 쿠키를 저장하고, 이어서 발생한 동일한 서버에 대한 요청 시 저장된 쿠키를 보낼 것이다.  

그러나 모든 쿠키가 아닌 좀 더 자세한 쿠키 허용 설정을 원할 경우 CookiePolicy를 지정함으로써 쿠키의 허용 범위를 지정할 수 있다. CookiePolicy에는 다음 세 개의 정책이 미리 정의되어 있다.  

+ CookiePolicy.ACCEPT_ALL 모든 쿠기 혀용
+ CookiePolicy.ACCEPT_NONE 쿠키 차단
+ CookiePolicy.ACCEPT_ORIGINAL_SERVER 서드파티 쿠키 차단  

예를 들어, 아래 코드는 자바에게 퍼스트파티 쿠키는 허용하되 서드파티 쿠키는 차단하도록 지시한다.  

```java
CookieManager manager - new CookieManager();
manager.setCookiePolicy(CookiePolicy.ACCEPT_ORIGINAL_SERVER);
CookieHandler.setDefault(manager);
```

즉, 위 설정은 여러분이 직접 접속한 서버에 대한 쿠키만을 허용하며, 다른 서버로부터 오는 서드파티 쿠키를 허용하지 않는다.  

이미 정의된 정책보다 더 세부적인 제어가 필요한 경우가 있다. 예들 들어, 알고 있는 몇몇 사이트에 대해서만 쿠키를 허용하고자 할 경우, CookiePolicy 인터페이스를 구현하여 shouldAccept() 메소드를 오버라이드한다.  

+ public boolean shouldAccept(URI uri, HttpCookie cookie)
