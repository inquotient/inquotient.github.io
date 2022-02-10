---
title:  CookieStore 클래스
categories:
- Java Network Programming
feature_text: |
  ## CookieStore 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

때로 로컬에 쿠키를 저장하고 꺼내야 할 필요가 있다. 예를 들어, 애플리케이션은 종료할 때, 쿠키를 로컬 디스크에 저장하고 다음 실행 시에 저장된 쿠키를 다시 읽어 사용한다. CookieManager의 getCookieStore() 메소드는 CookieManager 클래스의 저장소를 반환한다.  

```java
CookieStore store = manager.getCookieStore();
```

CookieStore 클래스를 사용하면 쿠키를 추가, 삭제 및 나열할 수 있다. 그래서 여러분은 HTTP 요청과 응답의 일반적인 흐름을 벗어나 전송된 쿠키를 제어할 수 있다.  

+ public void add(URI uri, HttpCookie cookie)
+ public List<HttpCookie> get(URI uri)
+ public List<HttpCookie> getCookies()
+ public List<URI> getURIs()
+ public boolean remove(URI uri, HttpCookie cookie)
+ public boolean removeAll()  

저장소의 각각의 쿠키들은 HttpCookie 객체에 담겨 있다.
