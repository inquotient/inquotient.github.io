---
title: DI와 스프링
categories:
- Spring
feature_text: |
  ## DI와 스프링
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
## 1. DI(Dependency Injection)
### 1.1. 의존(Dependency)이란?


기능을 실행하기 위해 다른 클래스(또는 타입)를 필요로 할 때 이를 의존(dependency)한다고 말함.  


### 1.2. 의존 객체를 직접 생성하는 방식의 단점


(1) 의존 객체를 변경하는 경우 의존 객체가 삽입되어 있는 모든 코드 변경을 해야함.  
(2) 아직 만들어지지 않은 의존 객체로 인해 테스트를 수행하지 못하는 문제가 발생함.  


### 1.3. DI를 사용하는 방식의 코드 : 의존 객체를 외부에서 조립
#### 1.3.1. DI란?


의존 객체를 외부로부터 전달받는 구현 방식  



### 1.4. 생성자 방식과 프로퍼티 설정 방식
#### 1.4.1. 프로퍼티 설정 방식


(1) 의존 객체를 전달받기 위해 메서드를 이용함.  
(2) 자바빈(JavaBeans)의 영향으로 프로퍼티 설정 방식은 setPropertyName() 형식의 메서드를 주로 사용함.
