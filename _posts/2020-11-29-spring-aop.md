---
title: 스프링 AOP
categories:
- Spring
feature_text: |
  ## 스프링 AOP
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

어플리케이션은 다양한 공통 기능을 필요로 한다. 로깅과 같은 기본적인 기능에서부터 트랜잭션이나 보안과 같은 기능에 이르기까지 어플리케이션 전반에 걸쳐 적용되는 공통 기능이 존재한다. 이들 공통 기능들은 어떤 특정 모듈에서만 필요로 하는 것이 아니라, 어플리케이션 전반에 걸쳐 필요한 기능이다. 또한, 이런 공통 기능들은 어플리케이션의 핵심 비즈니스 로직과는 구분되는 기능이다. 핵심 비즈니스 기능과 구분하기 위해 공통 기능을 공통 관심 사항(cross-cutting concern)이라고 표현하며, 핵심 로직을 핵심 관심사항(core concern)이라고 표현한다.  
<br/>
### 1. AOP 소개
<br/>
Aspect Oriented Programming, 줄용서 AOP는 문제를 바라보는 관점을 기준으로 프로그래밍하는 기법을 말한다. AOP는 문제를 해결하기 위한 핵심 관심 사항과 전체에 적용되는 공통 관심 사항을 기준으로 프로그래밍 함으로써 공통 모듈을 여러 코드에 쉽게 적용할 수 있도록 도와준다.  

AOP를 구현하는 다양한 방법이 존재하지만, 기본적인 개념은 공통 관심 사항을 구현한 코드를 핵심 로직을 구현한 코드 안에 삽입하는 것이다.  

AOP 기법에서는 핵심 로직을 구현한 코드에서 공통 기능을 직접적으로 호출하지 않는다. 핵심 로직을 구현한 코드를 컴파일하거나, 컴파일 된 클래스를 로딩하거나, 또는 로딩한 클래스의 객체를 생성할 때 AOP가 적용되어 핵심 로직 구현 코드 안에 공통 기능이 삽입된다.  

AOP 프로그래밍에서는 AOP 라이브러리가 공통 기능을 알맞게 삽입해주기 때문에, 개발자는 핵심 로직을 구현할 때 트랜잭션 적용이나 보안 검사와 같은 공통 기능을 처리하기 위한 코드를 핵심 로직 코드에 삽입할 필요가 없다. 핵심 로직을 구현한 코드에 공통 기능 관련 코드가 포함되어 있지 않으므로 적용해야 할 공통 기능이 변경되더라도 핵심 로직 구현 코드를 변경할 필요가 없다. 단지, 공통 기능 코드를 변경한 뒤 핵심 로직 구현 코드에 적용만 하면 된다.  
#### 1.1. AOP 용어
+ Joinpoint  
Advice를 적용 가능한 지점을 의미한다. 메서드 호출, 필드 값 변경 등이 Joinpoint에 해당한다.
+ Pointcut  
Joinpoint의 부분 집합으로서 실제로 Advice가 적용되는 Joinpoint를 나타낸다. 스프링에서는 정규 표현식이나 AspectJ의 문법을 이용하여 Pointcut을 나타낸다.
+ Advice  
언제 공통 관심 기능을 핵심 로직에 적용할 지를 정의하고 있다. 예를 들어 '메서드를 호출하기 전'(언제)에 '트랜잭션 시작'(공통 기능) 기능을 적용한다는 것을 정의하고 있다.
+ Weaving  
Advice를 핵심 로직 코드에 적용하는 것을 말한다.
+ Aspect  
여러 객체에 공통으로 적용되는 기능이다. 트랜잭션이나 보안 등이 Aspect의 좋은 예이다.

#### 1.2. 세 가지 Weaving 방식
<br/>
+ 컴파일 시에 Weaving하기  
AspectJ에서 사용하는 방식으로 컴파일 방식에서는 핵심 로직을 구현한 자바 소스 코드를 삽입하면, 컴파일 결과 AOP가 적용된 클래스 파일이 생성된다. 컴파일 방식을 제공하는 AOP 도구는 공통 코드를 알맞은 위치에 삽입할 수 있도록 도와주는 컴파일러나 IDE를 함께 제공한다.
+ 클래스 로딩 시에 Weaving하기  
AOP 라이브러리는 JVM이 클래스를 로딩할 때 클래스 정보를 변경할 수 있는 에이전트를 제공한다. 이 에이전트는 로딩한 클래스의 바이너리 정보를 변경하여 알맞은 위치에 공통 코드를 삽입한 새로운 클래스 바이너리 코드를 사용하도록 한다. 즉, 원본 클래스 파일은 변경하지 않고 클래스를 로딩할 때에 JVM이 변경된 바이트 코드를 사용하도록 함으로써 AOP를 적용한다. AspectJ는 컴파일 방식과 더불어 클래스 로딩 방식을 함께 지원하고 있다.
+ 런타임 시에 Weaving하기  
소스 코드나 클래스 정보 자체를 변경하지 않는다. 대신, 프록시를 이용하여 AOP를 적용한다. 프록시 기반의 AOP는 핵심 로직을 구현한 객체에 직접 접근하는 것이 아니라 중간에 프록시를 생성하여 프록시를 통해서 핵심 로직을 구현한 객체에 접근하게 된다. 이때, 프록시는 핵심 로직을 실행하기 전 또는 후에 공통 기능을 적용하는 방식으로 AOP를 구현하게 된다. 프록시 기반에서는 메서드가 호출될 때에만 Advice를 적용할 수 있기 때문에 필드 값 변경과 같은 Joinpoint에 대해서는 적용할 수 없는 한계가 있다.

### 2. 스프링에서의 AOP
<br/>
스프링은 자체적으로 프록시 기반의 AOP를 지원하고 있다. 따라서, 스프링 AOP는 메서드 호출 Joinpoint만을 지원한다. 필드 값 변경과 같은 Joinpoint를 사용하고 싶다면 AspectJ와 같이 다양한 Joinpoint를 지원하는 AOP 도구를 사용해야 한다.  

스프링은 완전한 AOP 기능을 제공하는 것이 목적이 아니라 엔터프라이즈 어플리케이션을 구현하는 데 필요한 기능을 제공하는 것을 목적으로 하고 있다.

스프링 AOP의 또 다른 특징은 자바 기반이라는 점이다. AspectJ는 Aspect를 위한 별도의 문법을 제공하고 있는 반면에 스프링은 별도의 문법을 익힐 필요 없이 자바 언어만을 이용하면 된다.  

스프링은 세 가지 방식으로 AOP를 구현할 수 있도록 하고 있다.
+ XML 스키마 기반의 POJO 클래스를 이용한 AOP 구현  
+ AspectJ에서 정의한 &#64;Aspect 애노테이션 기반의 AOP 구현  
+ 스프링 API를 이용한 AOP 구현  

어떤 방식을 사용하더라도 내부적으로는 프록시를 이용하여 AOP가 구현되므로 메서드 호출에 대해서만 AOP를 적용할 수 있다는 것에 유의하자. (즉, AspectJ에서 정의한 &#64;Aspect 애노테이션을 사용하더라도 메서드 호출과 관련된 Pointcut만 사용 가능하다.)

#### 2.1. 프록시를 이용한 AOP 구현
<br/>
스프링은 Aspect의 적용 대상(target)이 되는 객체에 대한 프록시를 만들어 제공하며, 대상 객체를 사용하는 코드는 대상 객체에 직접 접근하지 않고 프록시를 통해서 간접적으로 접근하게 된다. 이 과정에서 프록시는 공통 기능을 실행한 뒤 대상 객체의 실제 메서드를 호출하거나 또는 대상 객체의 실제 메서드를 호출한 후에 공통 기능을 실행하게 된다.  

대상 객체는 결구 스프링 빈 객체가 되는데, 스프링은 설정 정보를 이용해서 어떤 빈 객체에 Aspect를 적용할지의 여부를 지정한다. 스프링 컨테이너를 초기화하는 과정에서 설정 정보에 지정한 빈 객체에 대한 프록시 객체를 생성하고, 원본 빈 객체 대신에 프록시 객체를 사용하도록 한다.  

프록 시 객체를 생성하는 방식은 대상 객체가 인터페이스를 구현하고 있느냐 없느냐 여부에 따라 달라진다. 대상 객체가 인터페이스를 구현하고 있다면, 스프링은 자바 리플렉션 API가 제공하는 java.lang.reflect.Proxy를 이용하여 프록시 객체르르 생성한다. 이때 생성된 프록시 객체는 대상 객체와 동일한 이터페이스를 구현하게 되며, 클라이언트는 인터페이스를 통해서 필요한 메서드를 호출하게 된다. 하지만, 인터페이스를 기반으로 프록시 객체를 생성하기 때문에 인터페이스에 정의되어 있지 않은 메서드에 대해서는 AOP가 적용되지 않는 점에 유의해야 한다.  

대상 객체가 인터페이스를 구현하고 있지 않다면, 스프링은 CGLIB를 이용하여 클래스에 대한 프록시 객체를 생성한다. CGLIB는 대상 클래스를 상속받아 프록시를 구현한다. 따라서, 대상 클래스가 final인 경우 프록시를 생성할 수 없으며, final인 메서드에 대해서는 AOP를 적용할 수 없게 된다.  

#### 2.2. 구현 가능한 Advice 종류
<br/>
##### 2.2.1. 스프링에서 구현 가능한 Advice 종류
+ Before Advice  
대상 객체의 메서드 호출 전에 공통 기능을 실행한다.
+ After Returning Advice  
대상 객체의 메서드가 익셉션 없이 실행된 이후에 공통 기능을 실행한다.
+ After Throwing Advice  
대상 객체의 메서드를 실행하는 도중 익셉션이 발생한 경우에 공통 기능을 실행한다.
+ After Advice  
대상 객체의 메서드를 실행하는 도중에 익셉션이 발생했는지의 여부에 상관없이 메서드 실행 후 공통 기능을 실행한다.(try-catch-finally의 finally 블록과 비슷하다.)
+ Around Advice  
대상 객체의 메서드 실행 전, 후 또는 익셉션 발생 시점에 공통 기능을 실행하는데 사용된다.

### 3. XML 스키마 기반 AOP 퀵 스타트
<br/>
+ 스프링 AOP를 사용하기 위한 의존을 추가한다.  
+ 공통 기능을 제공할 클래스를 구현한다.
+ XML 설정 파일에 &#60;aop:config&#62;를 이용해서 Aspect를 설정한다. Advice를 어떤 Pointcut에 적용할지를 지정하게 된다.

스프링 AOP를 사용하려면, AOP를 구현하는데 필요한 의존을 추가해주어야 한다.

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-aop</artifactId>
	<version>4.0.4.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.aspectj</groupId>
	<artifactId>aspectjweaver</artifactId>
	<version>1.7.4</version>
</dependency>
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
				http://www.springframework.org/schema/beans/spring-beans.xsd
				http://www.springframework.org/schema/aop
				http://www.springframework.org/schema/aop/spring-aop.xsd">

	<!-- 공통 기능을 제공할 클래스를 빈으로 등록 -->
	<bean id="빈식별자" class="완전한 클래스명"/>

	<!-- Aspect 설정: Advice를 어떤 Pointcut에 적용할 지 설정 -->
	<aop:config>
		<aop:aspect id="애스펙트식별자" ref="빈식별자">
			<aop:pointcut id="포인트컷식별자" ref="AspectJ표현식"/>
			<aop:around pointcut-ref="프인트컷식별자" method="메소드명" />
		</aop:aspect>
	</aop:config>
</beans>
```

#### 3.1. Advice 정의 관련 태그
+ &#60;aop:before&#62;  
메서드 실행 전에 적용되는 Advice를 정의한다.
+ &#60;aop:after-returning&#62;  
메서드가 정상적으로 실행된 후에 적용되는 Advice를 정의한다.
+ &#60;aop:after-throwing&#62;  
메서드가 익셉션을 발생시킬 때 적용되는 Advice를 정의한다. try-catch 블록에서 catch 블록과 비슷하다.
+ &#60;aop:after&#62;  
메서드가 정상적으로 실행되는지 또는 익셉션을 발생시키는지 여부에 상관없이 적용되는 Advice를 정의한다. try-catch-finally에서 finally 블록과 비슷하다.
+ &#60;aop:around&#62;
메서드 호출 이전, 이후, 익셉션 발생 등 모든 시점에 적용 가능한 Advice를 정의한다.

#### 4.2. Advice 타입별 클래스 작성
<br/>
&#60;aop:before&#62;, &#60;aop:around&#62; 등의 태그는 &#60;aop:aspect&#62; 태그의 ref 속성에 전달된 빈 객체의 메서드를 실행함으로써 Advice를 대상 객체에 적용한다.

각 타입의 Advice에 따라서 메서드의 구현 방법이 조금씩 차이가 난다.

##### 3.1.1. Before Advice
<br/>
Before Advice를 사용하려면 &#60;aop:before&#62; 태그를 이용한다.
```java
<aop:config>

	...

	<aop:aspect id="애스펙트식별자" ref="빈식별자">
		<aop:before pointcut-ref="포인트컷식별자" method="메소드명"/>
	</aop:aspect>

	...

</aop:config>
```

대상 객체 및 호출되는 메서드에 대한 정보 또는 전달되는 파라미터에 대한 정보가 필요하다면, org.aspectj.lang.JoinPoint 타입의 파라미터를 메서드에 전달한다.  

Before Advice를 구현한 메서드는 일반적으로 리턴 타입이 void인데, 그 이유는 리턴 값을 갖더라도 실제 Advice의 적용 과정에서 사용되지 않기 때문이다.

Before Advice를 사용할 경우 코드 실행 흐름은 다음과 같다.

+ 빈 객체를 사용하는 코드에서 스프링이 생성한 AOP 프록시의 메서드를 호출  
+ AOP 프록시는 &#60;aop:before&#62;에서 지정한 메서드를 호출  
+ AOP 프록시는 Aspect 기능 실행 후, 실제 빈 객체의 메서드를 호출  

Before Advice 용도로 메서드를 구현할 때 주의할 점은 메서드에서 익셉션을 발생시킬 경우 대상 객체의 메서드가 호출되지 않는다는 점이다. 따라서 메서드를 실행하기 전에 접근 권한을 검사해서 접근 권한이 없을 경우 익셉션을 발생시키는 기능을 구현하는 데 Before Advice가 적합하다.

##### 3.1.2. After Returning Advice
<br/>
After Returning Advice는 대상 객체의 메서드가 정상적으로 실행된 후에 공통 기능을 적용하고 싶을 때 사용되는 Advice로서, 다음과 같이 &#60;aop:after-returning&#62; 태그를 이용하여 설정한다.

```xml
<aop:config>

	...

	<aop:aspect id="애스펙트식별자" ref="빈식별자">
		<aop:after-returning pointcut-ref="포인트컷식별자" method="메소드명" returning="리턴 값을 전달받을 파라미터명"/>
	</aop:aspect>

	...

</aop:config>
```

After Returning Advice를 사용할 경우 코드 실행 흐름은 다음과 같다.
+ 빈 객체를 사용하는 코드에서 스프링이 생성한 AOP 프록시의 메서드를 호출
+ AOP 프록시는 실제 빈 객체의 메서드를 호출
+ AOP 프록시는 &#60;aop:after-returning&#62;에서 지정한 메서드 호출

Advice를 구현한 메서드는 returning 속성에 명시한 이름을 갖는 파라미터를 이용해서 리턴 값을 전달 받게 된다.  

```java
public void 메소드명(Object 파라미터명) {
	// 대상 객체의 메서드가 정상적으로 실행된 이후에 적용할 기능 구현

	...

}
```

만약 리턴된 객체가 특정 타입인 경우에 한해서 메서드를 실행하고 싶다면 다음과 같이 한정하고 싶은 타입의 파라미터를 사용하면 된다.

```java
public void 메소드명(클래스명 파라미터명) {

	...

}

```

대상 객체 및 호출되는 메서드에 대한 정보나 전달되는 파라미터에 대한 정보가 필요한 경우 다음과 같이 org.aspectj.lang.JoinPoint를 파라미터로 추가한다.
```java
public void 메소드명(JoinPoint joinPoint, Object 파라미터명) {

	...

}
```

##### 3.1.3. After Throwing Advice
<br/>
After Throwing Advice는 대상 객체의 메서드가 익셉션을 발생시킨 경우에 적용되는 Advice로서 다음과 같이 &#60;aop:after-throwing&#62; 태그를 이용하여 설정한다.

```xml
<aop:config>

	...

	<aop:aspect id="애스펙트식별자" ref="빈식별자">
		<aop:after-throwing pointcut-ref="포인트컷식별자" method="메소드명" throwing="리턴 값을 전달받을 파라미터명"/>
	</aop:aspect>

	...

</aop:config>
```

Aspect 구현 클래스는 다음과 같이 &#60;aop:after-throwing&#62; 태그에 명시한 메서드를 구현한다.

```java
public void 메소드명(Object 파라미터명) {
	// 대상 객체의 메서드가 정상적으로 실행된 이후에 적용할 기능 구현

	...

}
```

Afater Throwing Advice의 실행 흐름은 After Return Advice와 동일하다. 차이점이 있다면, After Return Advice는 AOP 대상 객체의 메서드가 정상적으로 실행될 때 사용되는 반면에 After Throwing Advice는 AOP 대상 객체의 메서드가 익셉션을 발생시킬 때 사용된다는 점이다.

대상 객체의 메서드가 발생시킨 익셉션 객체가 필요한 경우 throwing 속성에 익셉션 객체를 전달받을 파라미터의 이름을 명시하면 된다.

```java
public void 메소드명() {
	// 대상 객체의 메서드가 익셉션을 발생시킨 경우에 적용할 기능 구현

	...

}
```

Advice 구현 메서드에서 발생된 익셉션을 사용하려면 &#60;aop:after-throwing&#62; 태그의 throwing 속성에 명시한 이름을 갖는 파라미터를 추가한다.

```java
public void 메소드명(Throwable 파라미터명) {

	...

}
```

만약 특정 타입의 익셉션에 대해서만 처리하고 싶다면, Throwable이나 Exception이 아니라 처리하고 싶은 익셉션 타입을 파라미터로 지정하면 된다.

대상 객체 및 호출되는 메서드에 대한 정보나 전달되는 파라미터에 대한 정보가 필요한 경우 다음과 같이 org.aspectj.lang.JoinPoint를 파라미터로 추가한다.

```java
public void 메소드명(JoinPoint joinPoint, Exception 파라미터명) {

	...

}
```

##### 3.1.4. After Advice
<br/>
After Advice는 대상 객체의 메서드가 정상적으로 실행되었는지 아니면 익셉션을 발생시켰는지의 여부에 상관없이 메서드 실행이 종료된 이후에 적용되는 Advice로서 try-catch-finally 블록에서 finally 블록과 비슷한 기능을 수행한다. 다음과 같이 &#60;aop:after&#62; 태그를 이용하여 After Advice를 설정한다.

```xml
<aop:config>

	...

	<aop:aspect id="애스펙트식별자" ref="빈식별자">
		<aop:after pointcut-ref="포인트컷식별자" method="메소드명"/>
	</aop:aspect>

	...

</aop:config>
```

Aspect로 사용될 빈 클래스는 다음과 같이 &#60;aop:after&#62; 태그에 명시한 메서드를 구현해주면 된다. 이 메서드는 파라미터를 갖지 않는다.

```java
public void 메소드명() {

	...

}
```

대상 객체 및 호출되는 메서드에 대한 정보나 전달되는 파라미터에 대한 정보가 필요한 경우 다음과 같이 org.aspectj.lang.JoinPoint를 파라미터로 명시하면 된다.

```java
public void 메소드명(JoinPoint joinPoint) {

	...

}
```

##### 3.1.5. Around Advice
<br/>
Around Advice는 앞서 살펴 본 Before, After Returning, After Throwing, After Advice를 모두 구현할 수 있는 Advice로서, 다음과 같이 &#60;aop:around&#62; 태그를 이용하여 Around Advice를 설정한다.

```xml
<aop:config>

	...

	<aop:aspect id="애스펙트식별자" ref="빈식별자">
		<aop:around pointcut-ref="포인트컷식별자" method="메소드명"/>
	</aop:aspect>

	...

</aop:config>
```

Around Advice를 구현한 메서드는 org.aspectj.lang.ProceedingJoinPoint 타입을 첫번째 파라미터로 지정해야 한다. 그렇지 않을 경우 초기화 과정에서 익셉션이 발생한다.

### 5. &#64;Aspect 애노테이션 기반 AOP 퀵 스타트
<br/>
aop 스키마가 XML 설정을 이용해서 Advice, Pointcut 등을 설정하는 방식이라면, &#64;Aspect 애노테이션은 자바 코드에서 AOP를 설정하는 방식이다. &#64;Aspect 애노테이션은 자바 코드에서 AOP를 설정하는 방식이다. &#64;Aspect 애노테이션을 이용해서 AOP를 구현하는 과정은 XML 스키마 기반의 AOP를 구현하는 과정과 거의 유사하며, 차이점은 다음과 같다.

+ &#64;Aspect 애노테이션을 이용해서 Aspect 클래스를 구현한다. 이때 Aspect 클래스는 Advice를 구현한 메서드와 Pointcut을 포함한다.  
+ XML 설정에서 &#60;aop:aspectj-autoproxy/&#62;를 설정한다. &#64;Configuration 기반 자바 설정을 이용한다면 &#64;EnableAspectJAutoProxy를 설정한다.  

&#64;Aspect 애노테이션을 이용할 경우 XML 설정 파일에서 Pointcut을 설정하는 것이 아니라 클래스에 Pointcut을 정의한다.

```java
@Aspect
public class ProfilingAspect {

	@Pointcut("AspectJ 표현식")
	private void 포인트컷메소드명() {
	}

	@Around("포인트컷메소드명()")
	public Object 메소드명1(ProceedingJoinPoint joinPoint) throws Throwable {
		
		...

		Object result = joinPoint.proceed();
			
		...

		return result;
	}
}
```

&#64;Pointcut 애노테이션은 Pointcut을 정의하는 AspectJ 표현식을 값으로 가지며, &#64;Pointcut 애노테이션이 적용된 메서드는 리턴 타입이 void이어야 한다.  

&#64;Pointcut 애노테이션은 Pointcut을 정의하면, Advice 관련 애노테이션에서 해당 메서드 이름을 이용해서 Pointcut을 사용할 수 있게 된다. Around Advice를 구현하려면 &#64;Around 애노테이션의 값으로 &#64;Pointcut 애노테이션을 적용한 메서드의 이름을 지정한다.  

### 6. &#64;Aspect 애노테이션을 이용한 AOP
#### 6.1. Advice 타입별 클래스 작성
<br/>
XML 스키마 방식을 사용할 경우 &#60;aop:around&#62;, &#60;aop:before&#62;와 같은 태그를 이용해서 Advice 타입을 지정했는데, &#64;Aspect 애노테이션을 사용할 경우에는 &#64;Before나 &#64;Around와 같은 애노테이션을 이용해서 Aspect 구현 메서드에 Advice 타입을 직접 지정한다.

##### 6.1.1. Before Advice
<br/>
```java
@Aspect
public class 클래스명 {

	...

	@Before("AspectJ표현식 또는 @Pointcut이 적용된 메소드")
	public void 메소드명1() {
		...
	}

	...

	@Before("AspectJ표현식 또는 @Pointcut이 적용된 메소드")
	public void 메소드명2(JoinPoint joinPoint) {
		...
	}

	...

}
```

&#64;Before 애노테이션의 값으로는 AspectJ의 Pointcut 표현식이나 또는 &#64;Pointcut 애노테이션이 적용된 메서드 이름이 올 수 있다.

##### 6.1.2. After Returning Advice
<br/>
```java
@Aspect
public class 클래스명 {

	...

	@AfterReturning("AspectJ표현식 또는 @Pointcut이 적용된 메소드")
	public void 메소드명1() {
		...
	}

	...

	@AfterReturning(pointcut = "AspectJ표현식 또는 @Pointcut이 적용된 메소드", returning = "파라미터명")
	public void 메소드명2(Object 파라미터명) {
		...
	}

	...

	@AfterReturning("AspectJ표현식 또는 @Pointcut이 적용된 메소드", returning = "파라미터명")
	public void 메소드명3(클래스명 파라미터명) {
		...
	}

	...

	@AfterReturning("AspectJ포현식 또는 @Pointcut이 적용된 메소드", returning = "파라미터명")
	public void 메소드명4(JoinPoint joinPoint, 클래스명 파라미터명) {
		...
	}

	...

}
```

##### 6.1.3. After Throwing Advice
<br/>
```java
@Aspect
public class 클래스명 {

	...

	@AfterThrowing("AspectJ표현식 또는 @Pointcut이 적용된 메소드")
	public void 메소드명1() {
		...
	}

	...

	@AfterThrowing(pointcut = "AspectJ표현식 또는 @Pointcut이 적용된 메소드", throwing = "파라미터명")
	public void 메소드명2(Throwable 파라미터명) {
		...
	}

	...

	@AfterThrowing("AspectJ표현식 또는 @Pointcut이 적용된 메소드", throwing = "파라미터명")
	public void 메소드명3(익셉션클래스명 파라미터명) {
		...
	}

	...

	@AfterReturning("AspectJ포현식 또는 @Pointcut이 적용된 메소드", throwing = "파라미터명")
	public void 메소드명4(JoinPoint joinPoint, 익셉션클래스명 파라미터명) {
		...
	}

	...

}
```

##### 6.1.4. After Advice
<br/>
```java
@Aspect
public class 클래스명 {

	...

	@After("AspectJ표현식 또는 @Pointcut이 적용된 메소드")
	public void 메소드명1() {
		...
	}

	...

	@After("AspectJ표현식 또는 @Pointcut이 적용된 메소드")
	public void 메소드명2(JoinPoint joinPoint) {
		...
	}

	...

}
```

##### 6.1.5. Around Advice
<br/>
&#64;Around 애노테이션을 이용하면 Around Advice를 구현할 수 있다. Around Advice를 구현한 메서드는 org.aspectj.lang.ProceedingJoinPoint 타입을 첫 번째 파라미터로 지정해야 한다. 그렇지 않을 경우 익셉션이 발생한다.  

```java
@Aspect
public class 클래스명 {

	...

	@Before("AspectJ표현식 또는 @Pointcut이 적용된 메소드")
	public void 메소드명(ProceedingJoinPoint joinPoint) {
		...
	}

	...

}
```

#### 6.2. &#64;Pointcut 애노테이션을 이용한 Pointcut 설정
<br/>
XML 스키마를 사용하는 경우와 동일하게 &#64;Aspect 애노테이션을 사용하는 경우에도 &#64;Pointcut 애노테이션을 이용해서 Pointcut 설정을 재사용할 수 있다.

```java
@Aspect
public class 클래스명 {

	...

	@Pointcut("포인트컷표현식")
	private void 메소드명1() {}

	@Around("메소드명1()")
	public Object 메소드명2(ProceedingJoinPoint joinPoint) throws Throwable {
		...
	}

	...

}
```

위 코드에서 &#64;Pointcut 애노테이션은 Pointcut 표현식을 값으로 가지며, &#64;Pointcut 애노테이션이 적용된 메서드는 리턴 타입이 void이어야만 한다. 일반적으로 &#64;Pointcut 애노테이션이 적용된 메서드는 위 코드에서와 같이 메서드 몸체에 코드를 갖지 않는다. (정확하게는 코드를 가져도 의미가 없다.)  

&#64;Pointcut 애노테이션을 이용해서 Pointcut을 정의하면, 위 코드처럼 Advice 관련 애노테이션에서 &#64;Pointcut 애노테이션이 적용된 메서드(이하, &#64;Pointcut 메서드)의 이름을 이용해서 Pointcut을 참조할 수 있다. 이때 메서드 이름은 다음과 같이 범위에 따라서 알맞게 입력해야 한다.  
+ 같은 클래스에 위치한 &#64;Pointcut 메서드는 '메서드이름'만 입력
+ 같은 패키지에 위치한 &#64;Pointcut 메서드는 '클래스단순이름.메서드이름'을 입력
+ 다른 패키지에 위치한 &#64;Pointcut 메서드는 '완전한클래스이름.메서드이름'을 입력  

#### 6.3. 자바 설정에서 &#64;EnableAspectJAutoProxy를 이용한 &#64;Aspect 적용하기
<br/>
&#64;Configuration을 이용한 자바 코드 스프링 설정 방식을 사용할 경우, &#64;EnableAspectJAutoProxy 애노테이션을 사용하면 &#64;Aspect가 적용된 빈 객체를 Aspect로 사용할 수 있게 된다.  

```java
@Configuration
@EnableAspectJAutoProxy
public class 클래스명 {
	
	...

	@Bean
	public Aspect클래스명 메소드명() {
	
		...

		return Aspect클래스명 생성구문;
	}

	...

}
```

### 7. JoinPoint 사용
<br/>
Around Advice를 제외한 나머지 Advice 타입을 구현한 메서드는 org.aspectj.lang.JoinPoint 객체를 선택적으로 사용할 수 있다. 단, JoinPoint를 파라미터로 사용할 때에는 반드시 첫 번째 파라미터로 지정해주어야 한다. 만약, JoinPoint를 두 번째나 그 뒤에 위치한 파라미터로 지정하면 스프링은 익셉션을 발생시킨다.  

JoinPoint 인터페이스는 호출되는 대상 객체, 메서드 그리고 전달되는 파라미터 목록에 접근할 수 있는 메서드를 제공하고 있으며, 이는 다음과 같다.
+ Signature getSignature() : 호출되는 메서드에 대한 정보를 구한다.
+ Object getTarget() : 대상 객체를 구한다.
+ Object[] getArgs() : 파라미터 목록을 구한다.

org.aspectj.lang.Signature 인터페이스는 호출되는 메서드와 관련된 정보를 제공하기 위해 다음과 같은 메서드를 정의하고 있다.
+ String getName() : 메서드의 이름을 구한다.
+ String toLongString() : 메서드를 완전하게 표현한 문장을 구한다.(메서드의 리턴 타입, 파라미터 타입이 모두 표시된다.)
+ String toShortString() : 메서드를 축약해서 표현한 문장을 구한다.(기본 구현은 메서드의 이름만을 구한다.)

Around Advice의 경우 org.aspectj.lang.ProceedingJoinPoint를 첫 번째 파라미터로 전달받는데, ProceedingJoinPoint 인터페이스는 프록시 대상 객체를 호출할 수 있는 proceed() 메서드를 제공하고 있다. ProceedingJoinPoint는 JoinPoint 인터페이스를 상속받고 있으므로 Around Advice 역시 앞서 설명한 메서드와 Signature를 이용하여 대상 객체, 메서드 및 전달되는 파라미터에 대한 정보를 구할 수 있다.

### 8. 타입을 이용한 파라미터 접근
<br/>
JoinPoint의 getArgs() 메서드를 이용하면 대상 객체의 메서드를 호출할 때 사용한 인자에 접근할 수 있다고 했는데, Advice 메서드에서 직접 파라미터를 이용해서 메서드 호출시 사용된 인자에 접근할 수도 있다. 파라미터를 이용해서 대상 객체의 메서드를 호출할 때 사용한 인자에 접근하려면 다음과 같이 두 가지 작업을 진행해주면 된다.
+ Advice 구현 메서드에 인자를 전달받을 파라미터를 명시한다.
+ Pointcut 표현식에서 args() 명시자를 사용해서 인자 목록을 지정한다.

```java
public class 클래스명 {

	...

	public void 메소드명(파라미터타입1 파라미터명1, 파라미터타입2 파라미터명2) {
		...
	}

	...

}
```

```xml
<bean id="빈식별자" class="완전한클래스명"/>

<aop:config>
	<aop:aspect id="애스펙트식별자" ref="빈식별자">
		<aop:after-returning pointcut="args(파라미터명1, 파라미터명2)" method="메소드명" />
	<aop:aspect/>
</aop:config>
```

위 설정에서 args() 명시자가 의미하는 것은 다음과 같다.
+ 대상 객체의 메서드 호출시 인자가 두 개 전달되고, 이 중 첫 번째 인자는 지정한 메서드의 파라미터타입1과 타입이 같고, 두 번째 인자는 파라미터타입2와 타입이 같다.  

메서드 선언에서 사용된 타입이 args() 명시자와 매칭되는 타입과 다르다 하더라도, 살제로 메서드에 전달되는 인자의 타입이 args() 명시자를 통해서 지정한 것과 동일하다면 Advice가 적용된다.

&#64;Aspect 애노테이션을 사용하는 경우에도 XML 스키마를 사용할 때처럼 Pointcut 표현식에 args() 명시자를 사용하면 된다.

```java
public class 클래스명 {

	...

	@AfterReturning(pointcut = "args(파라미터명1, 파라미터명2)", returning="파라미터명3")
	public void 메소드명1(파라미터타입1 파라미터명1, 파라미터타입2 파라미터명2, 파라미터타입3 파라미터명3) {
		...
	}

	...

	@AfterReturning(pointcut = "args(파라미터명4, 파라미터명5)", argNames="파라미터명4,파라미터명5", returning="파라미터명6")
	public void 메소드명2(파라미터타입4 파라미터명4, 파라미터타입5 파라미터명5, 파라미터타입6 파라미터명6) {
		...
	}

	...

	@AfterReturning(pointcut = "args(파라미터명7, 파라미터명8)", argNames="파라미터명7,파라미터명8", returning="파라미터명9")
	public void 메소드명3(JoinPoint joinPoint, 파라미터타입1 파라미터명1, 파라미터타입2 파라미터명2, 파라미터타입3 파라미터명3) {
		...
	}

	...

}
```

#### 8.1. 인자의 이름 매핑 처리
<br/>
앞서 args() 명시자를 이용해서 메서드 호출시 사용된 인자를 파라미터로 전달받을 수 있다고 했다. args() 명시자에 지정한 이름과 Advice 구현 메서드의 파라미터 이름이 일치하는 지의 여부를 확인하는 순서는 다음과 같다.  
(1) Advice 애노테이션 태그의 argNames 속성이나 Advice 설정 XML 스키마의 arg-names 속성세서 명시한 파라미터 이름을 사용한다.  
(2) argName 속성이 없을 경우, 컴파일할 때 생성되는 디버그 정보를 이용해서 파라미터 이름이 일치하는 지의 여부를 확인한다.  
(3) 디버그 옵션이 없을 경우 파라미터 개수를 이용해서 일치 여부를 유추한다.  

argNames 속성은 Advice 구현 메서드의 파라미터 이름을 입력할 때 사용된다. argNames 속성은 모든 파라미터의 이름을 순서대로 표시해서 Pointcut 표현식에서 사용된 이름이 몇 번째 파라미터인지 검색할 수 있도록 한다.  

만약 첫 번째 파라미터의 타입이 JoinPoint나 ProceedingJoinPoint라면, JoinPoint 타입의 파라미터 이름을 포함하지 않는다.

XML 스키마를 사용하는 경우 다음과 같이 arg-nmaes 속성을 이용해서 파라미터 이름을 지정한다.

```xml
<aop:after-returning pointcut="args(파라미터명1,파라미터명2)" method="메소드명1" returning="파라미터명3" arg-names="jointPoint,파라미터명4,파라미터명5"/>
```

argNames 속성 또는 arg-names 속성을 지정하지 않은 경우에는 디버그 정보를 이용한다.  

마지막으로 디버그 정보도 없는 경우에는 파라미터 개수를 이용해서 유추한다.

(1) ~ (3)까지 모두 해당되지 않는다면 IllegalArgumentException을 발생한다.

### 9. AOP 프록시 객체 생성 방식 설정
<br/>
```xml
<aop:config proxy-target-class="true">
	...
</aop:config/>

<aop:aspectj-autoproxy proxy-target-class="true"/>
```

```java
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class 클래스명 {

	...

}
```

각 설정 방식에서 프록시 대상을 크래스로 사용할지 여부를 true로 지정해주면 실제 생성되는 프록시는 인터페이스가 아닌 클래스를 상속받아 생성된다.

### 10. AspectJ의 Pointcut 표현식
<br/>
스프링은 공통 기능인 Aspect를 지정할 Pointcut을 지정하기 위해 AspectJ의 문법을 사용한다.

AspectJ는 Pointcut을 명시할 수 있는 다양한 명시자를 제공하는데, 스프링은 메서드 호출과 관련된 명시자(designator)만을 지원하고 있다.

#### 10.1. execution 명시자
<br/>
execution 명시자는 Advice를 적용할 메서드를 명시할 때 사용되며, 기본 형식은 다음과 같다.  

execution(수식어패턴? 리런타입패턴 클래스이름패턴?메서드이름패턴(파라미터패턴)  

'수식어패턴' 부분은 생략 가능한 부분으로서 public, protected 등이 온다. 스프링 AOP의 경우 public 메서드에만 적용 가능하기 때문에 사실상 public 이외의 값은 의미가 없다.  

'리턴타입패턴' 부분은 리턴 타입을 명시한다. '클래스이름패턴'과 '이름패턴' 부분은 클래스 이름 및 메서드 이름을 패턴으로 명시한다. '파라미터패턴' 부분은 매칭될 파라미터에 대해서 명시한다.  

각 패턴은 &#39;&#42;&#39;을 이용하여 모든 값을 표현할 수 있다. 또한, '..'을 이용하여 0개 이상이라는 의미를 표현할 수 있다.  

다음은 몇 가지 예이다.  
+ execution(public void set&#42;(..))  
리턴 타입이 void이고 메서드 이름이 set으로 시작하고, 파라미터가 0개 이상인 메서드 호출 파라미터 부분에 '..'을 사용하여 파라미터가 0개 이상인 것을 표현하였다.
+ execution(&#42; net.madvirus.spring4.chap06..&#42;.&#42;())  
net.madviurs.spring4.chap06 패키지의 파라미터가 없는 모든 메서드 호출
+ execution(&#42; net.madvirus.spring4.chap06..&#42;.&#42;(..))  
net.madvirus.spring4.chap06 패키지 및 하위 패키지에 있는, 파라미터가 0개 이상인 메서드 호출. 패키지 부분에 '..'을 사용하여 해당 패키지 또는 하위 패키지를 표현하였다.
+ execution(Integer net.madvirus.spring4.chap06.board.WriteArticleService.write(..))  
리턴 타입이 Integer인 WriteArticleService 인터페이스의 write() 메서드 호출
+ execution(&#42; get&#42;(&#42;))  
이름이 get으로 시작하고 1개의 파리머트를 갖는 메서드 호출
+ execution(&#42; get&#42;(&#42;, &#42;))  
이름이 get으로 시작하고 2개의 파라미터를 갖는 메서드 호출
+ execution(&#42; read&#42;(Integer, ..))  
메서드 이름이 read로 시작하고, 첫 번째 파리머터 타입이 Integer이며, 1개 이상의파라미터를 갖는 메서드 호출

#### 10.2. within 명시자
<br/>
within 명시자는 특정 타입에 속하는 메서드를 Pointcut으로 설정할 때 사용된다. 다음은 설정 예이다.  
+ within(net.madvirus.spring4.chap06.board.WriteArticleService)  
WriteArticleService 인터페이스의 모든 메서드 호출
+ within(net.madvirus.spring4.chap06.board.&#42;)  
net.madvirus.spring4.chap06.board 패키지에 있는 모든 메서드 호출
+ within(net.madvirus.spring4.chap06..&#42;)  
net.madvirus.spring4.chap06 패키지 및 그 하위 패키지에 있는 모든 메서드 호출

#### 10.3. bean 명시자
<br/>
bean 명시자는 스프링에서 추가적으로 제공하는 명시자로서, 스프링 빈 이름을 이용하여 Pointcut을 정의한다. bean 명시자는 빈 이름의 패턴을 갖는다. 다음은 설정 예이다.  
+ bean(writeArticleService)  
이름이 writeArticleService인 빈의 메서드 호출
+ bean(&#42;ArticleService)  
이름이 ArticleService로 끝나는 빈의 메서드 호출

#### 10.4. Pointcut의 조합
<br/>
각각의 표현식은 '&&' 또는 '||'연산자를 이용하여 연결할 수 있다.
```java
@AfterThrowing(pointcut = "execution(public * get*()) && execution(public void set*(..))" )
public void 메소드명() {
	...
}
```

XML 스키마를 이용하여 Aspect를 설정하는 경우에도 다음과 같이 '&&' 또는 '||' 연산자를 사용할 수 있다.
```xml
<aop:pointcut id="포인트컷식별자" expression="execution(public * get*()) &amp;&amp; execution(public void set*(..))"/>
```

설정 파일은 XML 문서이기 때문에 값에 들어가는 '&&'를 '&#38;amp;&#38;amp;'로 표현하였다. 이렇게 입력하는 것은 불편하므로, 스프링은 설정 파일에서 '&&'나 '||' 대신 'and'와 'or'를 사용할 수 있도록 하고 있다. 따라서, 위 XML 설정을 다음과 같이 입력할 수도 있다.
```xml
<aop:pointcut id="포인트컷식별자" expression="execution(public * get*()) and execution(public void set*(..))"/>
```

&#64;Pointcut 애노테이션을 사용하는 경우 메서드 이름을 이용해서 손쉽게 두 개 이상의 Pointcut을 조합할 수 있다.

&#64;Pointcut 애노테이션을 이용하면 XML 스키마를 이요하는 경우에 비해 Pointcut의 조합 및 재사용이 보다 쉽기 때문에 공통으로 사용되는 Pointcut은 &#64;Pointcut 애노테이션을 이용해서 표현하는 것이 좋다.

### 11. Advice 적용 순서
<br/>
하나의 JoinPoint에 한 개 이상의 Advice가 적요될 경우, 순서를 명시적으로 지정할 수 있다. Advice의 적용 순서를 명시적으로 지정하는 첫 번째 방법은 Advice 구현 클래스에 다음과 같이 &#64;Order 애노테이션을 적용하거나 Ordered 인터페이스를 구현하는 것이다.
+ org.springframework.annotation.Order 애노테이션을 적용한다.
+ org.springframework.core.Ordered 인터페이스를 구현한다.

```java
@Aspect
public class 클래스명 implments Ordered {

	...

	@Around("execution(public * *..ReadArticleService.*(..))")
	public 클래스명 메소드명(ProceedingJoinPoint joinPoint) throws Throwable {
		...
	}

	...

	@Override
	public int getOrder() {
		return 2;
	}

	...

}
```

&#64;Order 애노테이션을 사용할 경우 다음과 같이 &#64;Order 애노테이션의 값으로 적용 순서 값을 지정해주면 된다.
```java
@Aspect
@Order(3)
public class 클래스명 {
	...
}
```

XML 스키마를 사용할 경우 &#60;aop:aspect&#62; 태그의 order 속성을 사용해서 Advice 순서를 지정할 수 있다.

```xml
<aop:aspect id="애스펙트식별자" ref="빈식별자" order="2">
	<aop:around pointcut="애스펙트표현식 또는 @Pointcut이 적용된 메소드명()" method="메소드명" />
</aop:aspect>
```

order 값이 낮은 Advice의 우선순위가 더 높다. 즉, 순서 값이 1인 Advice가 순서 값이 2인 Advice보다 우선순위가 높다. 하나의 메서드 호출에 대해 두 개 이상의 Advice가 적용될 경우, 우선순위가 높은(즉, order 값이 작은) Advice가 먼저 적용된다.
