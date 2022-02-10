---
title:  PasswordAuthentication 클래스
categories:
- Java Network Programming
feature_text: |
  ## PasswordAuthentication 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

PasswordAuthentication은 읽기만 가능한 사용자 이름과 패스워드 두 개의 속성만 제공하는 매우 단순한 최종(final) 클래스다. 사용자 이름은 문자열(String) 타입으로 저장되고, 패스워드는 문자(char) 배열 타입으로 저정되며 더 이상 필요하지 않을 때 지워질 수 있다.  

문자열 타입은 지워지고 난 후에도 가비지 컬렉터가 동작할 때까지 메모리에 남아 있으며, 심지어 가비지 컬렉터가 동작하고 난 이후에도 로컬 시스템 메모리 어딘가에, 혹은 디스크의 어딘가에 존재할 수 있다. 이와 같은 일은 패스워드 문자열을 가지고 있는 메모리 블록이 어떤 사점에 가상 메모리 공간으로 스왑(swap)되었을 때 발생할 수 있다. 사용자 이름과 패스워드 둘 모두 생성자에서 설정된다.  

+ public PasswordAuthentication(String username, char[] password)  

각각의 속성은 다음 메소드로 접근할 수 있다.  

+ public String getUserName()
+ publc char[] getPassword()
