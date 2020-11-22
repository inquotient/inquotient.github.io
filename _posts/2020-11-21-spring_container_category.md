---
title: 스프링 컨테이너 종류
categories:
- Spring
feature_text: |
  ## 스프링 컨테이너 종류
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

## 2. 스프링 컨테이너 종류
### 2.1. BeanFactory 계열 인터페이스


단순히 컨테이너에서 객체를 생성하고 DI를 처리하는 기능만 제공함.


### 2.2. ApplicationContext 계열 인터페이스


편리한 트랜잭션 처리, 자바 코드 기반 스프링 설정, 애노테이션을 사용한 빈 설정, 스프링을 이용한 웹 개발, 메시지 처리 등의 다양한 부가 기능을 제공함.


#### 2.2.1. GenericXmlApplicationContext


XML 파일을 설정 정보로 사용하는 스프링 컨테이너 구현 클래스로 독립형 어플리케이션을 개발할 때 사용함.


#### 2.2.2. AnnotationConfigApplicationContext


자바 코드를 설정 정보로 사용하는 스프링 컨테이너로 독립형 어플리케이션을 개발할 때 사용함.


#### 2.2.3. GenericGroovyApplicationConetxt


그루비 언어로 작성된 설정 정보를 사용하는 스프링 컨테이너로 독립형 어플리케이션을 개발할 때 사용함.


#### 2.2.4. XmlWebApplicationContext


웹 어플리케이션을 개발할 때 사용하는 스프링 컨테이너로서 XML 파일을 설정 정보로 사용함.


#### 2.2.5. AnnotationConfigWebApplicationContext


웹 어플리케이션을 개발할 때 사용하는 스프링 컨테이너로서 자바 코드를 설정 정보로 사용함.
