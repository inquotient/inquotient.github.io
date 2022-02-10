---
title:  URLDecoder 클래스
categories:
- Java Network Programming
feature_text: |
  ## URLDecoder 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

URLDecoder 클래스는 x-www-form-url-encoded 형식으로 인코딩된 문자열을 디코딩하는 정적 decode() 메소드를 제공한다. 즉, 이 메소드는 모든 더하기 부호를 공백으로 변환하고, 모든 % 인코딩된 문자를 대응하는 문자로 변환한다.  

+ public static String decode(String s, String encoding) throws UnSupportedEncodingException  

이 메소드 호출 시 어떤 인코딩 타입을 사용해야 할지 확실치 않은 경우 UTF-8을 사용하도록 하자. 다른 인코딩 타입보다 옳을 가능성이 높다.  

이 메소드는 디코딩 시에 두 개의 16진수가 뒤따라오지 않는 % 부호가 있거나, 잘못된 순서로 디코딩될 경우 IllegalArgumentException 예외를 발생시킨다.  

URLDecoder는 인코딩되지 않은 문자들은 건드리지 않기 때문에 먼저 각 부분을 나누지 않고 전체 URL을 한 번에 전달할 수 있다. 예를 들어:  

```java
String input = "http://www.google.com/" + "search?hl=en&as_q=Java&as_epq=I%2F0";
String output = URLDecoder.decode(input, "UTF-8");
System.out.println(output);
```
