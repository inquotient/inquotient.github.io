---
title: 빈 라이프사이클과 빈 범위
categories:
- Spring
feature_text: |
  ## 빈 라이플사이클과 빈 범위
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
#### 빈 라이프사이클을 관리하는 방법
+ 스프링이 제공하는 특정 인터페이스를 상속받아 빈을 구현함.
+ 스프링 설정에서 특정 메서드를 호출하라고 지정함.

### 1. 빈 객체의 라이프사이클
#### 1.1. 빈 라이프사이클 개요
(1) 빈 객체 생성  
(2) 빈 프로퍼티 설정  
(3) BeanNameAware.setBeanName()  
(4) ApplicationContextAware.setApplicationContext()  
(5) BeanPostProcessor의 초기화 전 처리  
(6) 초기화  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &#8211;&nbsp;&#64;PostConstruct 메서드  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &#8211;&nbsp;InitializingBean.afterPropertiesSet()  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &#8211;&nbsp;커스텀 init 메서드  
(7) BeanPostProcessor의 초기화 후 처리  
(8) 빈 객체 사용  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &#8211;&nbsp;&#64;PreDestroy 메서드  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &#8211;&nbsp;DisposableBean.destroy() 메서드  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &#8211;&nbsp;커스텀 destroy 메서드  

#### 1.2. InitializingBean 인터페이스와 DisposableBean 인터페이스
(1) org.springframework.beans.factory.InitializingBean : 빈의 초기화 과정에서 실행될 메서드를 정의
(2) org.springframework.beans.factory.DisposableBean : 빈의 소멸 과정에서 실행될 메서드를 정의

```java
public class 클래스명 implements InitializingBean, DisposableBean {
	
	...

	@Override
	public void afterPropertiesSet() throws Exception {
		...
	}

	@Override
	public void destroy() throws Exception {
		...
	}
}
```

#### 1.3. &#64;PostConstruct 애노테이션과 &#64;PreDestroy 애노테이션

```java
public class 클래스명 {

	...

	@PostConstruct
	public void 메소드명1() {
		...
	}

	@PreDestroy
	public void 메소드명2() {
		...
	}
}
```

```xml
<context:annotation-config/>
```

초기화와 소멸 과정에서 사용될 메서드는 파라미터를 가져서는 안 된다.


#### 1.4. 커스텀 init 메서드와 커스텀 destroy 메서드

```xml
<bean id="빈식별자" class="init/destroy 메서드가 설정된 완전한 클래스명" init-method="메소드명1" destroy-method="메소드명2"/>
```

```java
@Bean(initMethod = "메소드명1", destroyMethod = "메소드명2")
... 메소드명 ...
```

해당 클래스에 메소드명이 존재해야 하며 해당 메소드는 파라미터를 가져서는 안 된다.

#### 1.5. ApplicationContextAware 인터페이스와 BeanNameAware 인터페이스

+ org.springframework.context.ApplicationContextAware
&nbsp;&nbsp;&nbsp;&nbsp; 이 인터페이스를 상속받은 빈 객체는 초기화 과정에서 컨테이너(ApplicationContext)를 전달받는다.
+ org.springframework.context.BeanNameAware
&nbsp;&nbsp;&nbsp;&nbsp; 이 인터페이스를 상속받은 빈 객체는 초기화 과정에서 빈 이름을 전달받는다.

```java
public class 클래스명 implements ApplicationContextAware {

	...

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		...
	}
}
```

```java
public class 클래스명 implements BeanNameAware {

	...

	@Override
	public void setBeanName(String name) {
		...
	}
}
```

### 2. 빈 객체 범위(scope)
#### 2.1. 싱글톤 범위
별도 설정을 하지 않을 경우 스프링은 빈 객체를 한 번만 생성한다.

```xml
<bean id="빈식별자" class="완전한 클래스 이름" scope="singleton"/>
```

```java
@Bean
@Scope("singleton")
public 주입할클래스명 빈식별자() {
	...
	return 주일할클래스 생성;
}
```

#### 2.2. 프로토타입 범위

```xml
<bean id="빈식별자" class="완전한 클래스 이름" scope="prototype"/>
```

```java
@Bean
@Scope("prototype")
public 주입할클래스명 빈식별자() {
	...
	return 주일할클래스 생성;
}
```
스프링 컨테이너는 프로토타입 범위를 가진 빈의 초기화까지만 관리를 한다. 즉, 스프링 컨테이너를 종료한다고 해서 생성된 프로토타입 빈 객체의 소멸 과정이 실행되지는 않는다.
