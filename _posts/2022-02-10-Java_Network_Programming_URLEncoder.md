---
title:  URLEncoder 클래스
categories:
- Java Network Programming
feature_text: |
  ## URLEncoder 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

문자열을 인코딩하기 위해 인코딩할 문자열과 문자 집합을 URLEncoder.encode() 메소드들 호출하면서 전달한다. 예를 들어:  

```java
String encoded = URLEncoder.encode("This*sting*has*asterisks", "UTF-8");
```

URLEncoder.encode() 메소드는 호출 시 전달한 문자열의 내용을 일부 변환한 복사본을 반환한다. 반환된 문자열에서 영숫자가 아닌 다른 문자들은 %와 16진수를 사용한 방식으로 인코딩된다. (스페이스, 밑줄, 하이픈, 마침표 그리고 별표 문자는 인코딩 대상에 포함되지 않는다.)  

또한 URLEncoder.encode() 메소드는 모든 비아스키 문자들을 인코딩한다. 스페이스는 더하기 기호로 변환된다. 이 메소드는 조금 지나치게 많은 것을 변환하는 경향이 있다. URLEncoder.encode() 메소드는 강세 표시(tilde), 단일 인용 부호, 느낌표 그리고 괄호와 같은 변환할 필요가 없는 것까지 변환한다. 그러나 이러한 문자들의 변환이 URL 스펙에서 금지되지 않기 때문에, 웹 브라우저는 조금은 과도하게 인코딩된 URL를 무리 없이 처리한다.  

이 메소드가 인자로 문자 집합을 입력할 수 있도록 되어 있지만, 항상 UTF-8을 사용하도록 해야 한다. UTF-8은 다른 어떤 문자 집합들보다 IRI 스펙, URI 클래스, 현대의 대부분의 웹 브라우저 그리고 많은 추가적인 소프트웨어들과 호환된다.
