---
title: 스프링 시큐리티를 이용한 웹 보안
categories:
- Spring
feature_text: |
  ## 스프링 시큐리티를 이용한 웹 보안
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 1. 웹 보안과 스프링 시큐리티
<br/>
웹 어플리케이션을 개발할 때 프로그래머를 힘들게 만드는 것 중 하나를 꼽으라면 보안을 들 수 있다. 허가된 사용자만 접근할 수 있도록 제한하고, 이를 위해 현재 사용자갸 누구인지 확인해야 한다. 중요한 정보는 HTTPS와 같은 프로토콜을 이용하여 암호화해서 주고 받기도 하고, 사용자의 로그인 암호나 신용카드 번호와 같은 민감한 정보는 DBMS에 암호화하여 저장하기도 한다.  

보안 관련 영역 중에서 웹 어플리케이션 개발자의 코딩과 직결된 세 가지를 꼽자면 다음과 같은 것들이 있다.  

+ 인증(Authentication) 처리 : 현재 사용자가 누구인지 확인하는 과정으로, 입반적인 웹 어플리케이션은 아이디/암호를 이용해서 인증을 처리한다.
+ 인가(Authorization) 처리 : 현재 사용자가 특정 대상(URL, 기능 등)을 사용(접근)할 권한이 있는지 검사한다.  
+ UI 처리 : 권한이 없는 사용자가 접근했을 때, 알맞은 에러 화면을 보여주거나 로그인 폼과 같이 인증을 위한 화면으로 이동시킨다.  

보통 웹 어플리케이션에서는 로그인을 통해 인증을 수행한다. 로그인에 성공하면 인증 정보를 세션이나 쿠키 같은 곳에 보관하고, 이후 요청에서는 동일 인증 정보를 이용해서 사용자가 누구인지 식별한다.  

일단 사용자가 누군지 식별하면, 그 사용자가 현재 기능을 사용할 수 있는지 여부를 검사한다. 웹에서는 단순하게 URL별로 접근 권한을 부여하는 방법을 사용할 수 있다. 예를 들어, 현재 사용자가 고객 관리 기능을 위한 "/admin/member/list"라는 URL에 접근해서 회원 목록을 조회할 수 있는지 여부를 검사할 수 있다. 이를 검사할 수 있으면서 사용자가 접근 가능한 URL 목록을 갖고 있어야 하는데, 보통 이를 위해 역할 (role)이란 개념을 도입한다. 예를 들어, "/admin/member/list", "/admin/board/list"는 '관리자'라는 역할이 접근할 수 있다고 정의하고, '사용자1'이 '관리자'라는 역할을 갖는다고 해보자. 이 경우 '사용자1' "/admin/member/list"에 접근하면, '관리자'가 접근할 수 있는 URL이므로 접근을 허용한다.  

만약 접근 권한이 없으면 권한이 없다는 메시지를 보여주거나, 아직 인증 전이면 인증을 거치도록 로그인 폼을 보여주기도 한다.  

지금까지 설명한 세 가지-인증, 인가, 권한 없을 때 UI 처리-는 각 웹 어플리케이션마다 매우 유사한 구조를 갖는다. 따라서, 매번 새롭게 구현하기 보다는 기본 틀을 만들고 어플리케이션마다 다른 부분만 알맞게 구현함으로써 설계, 코드 작성 등에 드는 시간을 줄일 수 있을 것이다. 그리고, 스프링 시큐리티(Spring Security) 프로젝트는 정확히 이러한 목적으로 만들어졌다.  

스프링 시큐리티는 보편적인 인증, 인가, UI 처리에 대한 기본 구현을 제공하고 있으며, 일부 변경할 수 있는 확장 지점을 제공하고 있다. 따라서, 프로그래머는 처음부터 인증과 인가를 위한 코드를 만들기보다는 스프링 시큐리티가 제공하는 틀을 재사용하고 필요한 부분만 커스터마이징함으로써, 보다 빠르게 인증과 인가 부분의 구현을 마무리 할 수 있다.  

또한, 스프링 시큐리티는 암호화 기능도 제공하고 있다. 따라서, 암호화 기능에 대한 특별한 제약이 없다면, 스프링 시큐리티가 제공하는 암호화 기능을 사용해서, 비밀번호, 결제 정보 등을 암호화해서 보관할 수 있다.  

스프링 시큐리티를 사용하는 것이 대단히 어려운 것은 아니지만, 모든 것이 그렇듯 스프링 시큐리티에 대한 기본적인 지식은 있어야 스프링 시큐리티를 잘 사용할 수 있다.  

### 2. 스프링 시큐리티 퀵 스타트
<br/>
#### 2.1. 퀵 스타트 예제의 보안 요구
<br/>
#### 2.2. 메이븐 의존 설정
<br/>
&#60;dependencyManagement&62; 태그는 스프링 버전을 좀 더 쉽게 맞추는데 도움을 준다.  

#### 2.3. 스프링 시큐리티 XML 설정
<br/>
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:sec="http://www.springframework.org/schema/security"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
					http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">

	<sec:http use-expressions="true">
		<sec:intercept-url pattern="/admin/**" access="hasAuthority('ROLE_ADMIN)"/>
		<sec:intercept-url pattern="/manager/**" access="hasRole('ROLE_MANAGER')" />
		<sec:intercept-url pattern="/member/**" access="isAuthenticated()" />
		<sec:intercept-url pattern="/**" access="permitAll"/>
		<sec:form-login />
		<sec:logout />
	</sec:http>
	
	<sec:authentication-manager>
		<sec:authentication-provider>
			<sec:user-service>
				<sec:user name="bkchoi" password="1234" authorities="ROLE_USER"/>
				<sec:user name="manager" password="qwer" authorities="ROLE_MANAGER"/>
				<sec:user name="admin" password="admin" authorities="ROLE_ADMIN.ROLE_USER" />
			<sec:user-service>
		</sec:authentication-provider>
	</sec:authentication-manager>
</beans>
```

&#60;http&#62; 태그가 스프링 시큐리티 설정의 핵심이다. 이 태그에서 스프링 시큐리티와 관련된 거의 모든 설정을 처리한다. &#60;http&#62; 태그의 use-expression 속성의 값을 true로 지정했는데, 이는 &#60;intercept-url&#62; 태그의 access 속성에서 스프링 시큐리티가 제공하는 SpEL(스프링 표현식)을 사용할 수 있도록 만들어준다. 표현식을 사용하면 접근 IP 제한과 같이 좀 더 풍부한 접근 제한을 설정할 수 있다.  

&#60;intercept-url&#62; 태그는 접근 권한을 설정할 때 사용한다. pattern 속성은 접근 경로를 Ant 패턴으로 설정하며, access 속성은 해당 경로 패턴에 누가 접근 가능한지를 설정한다. 예를 들어, &#60;intercept-url&#62; 태그는 "/admin/"으로 시작하는 경로는 'ROLE&#95;ADMIN' 권한을 가진 사용자가 접근할 수 있다고 설정하는 태그이다. hasAuthority()와 hasRole()은 같은 의미를 갖는다. hasRole('ROLE&#95;MANAGER')는 ROLE&#95;MANAGER 권한을 가진 사용자가 접근 가능하게 설정한다. isAuthenticated()는 인증된 사용자만 접근 가능하도록 설정하며, permitAll은 누구나 접근 가능하다는 것을 뜻한다.  

&#60;form-login&#62; 태그는 다음의 두 기능을 제공한다.  

+ 인증된 사용자만 허용되는 자원(경로)에 접근할 때, 로그인 폼을 보여준다.
+ 로그인 폼에서 아이디/암호를 전송하면, 로그인(인증) 처리를 한다.  

&#60;logout&#62; 태그는 로그아웃 처리를 위한 기능을 추가한다.  

&#60;user&#62; 태그는 한 사용자를 설정하는데 사용하며 다음과 같이 세 개의 속성을 값으로 갖는다.  

name 속성은 사용자 이름을, password 속성은 암호를, authorities 속성은 사용자가 갖는 권한 목록을 지정한다. authorities 속성에 명시한 권한 이름을 앞서 설정한 &#60;intercept-url&#62; 태그에서 사용하게 된다.  

#### 2.4. 스프링 MVC 및 관련 설정
<br/>
#### 2.5. DispatcherServlet 설정과 스프링 시큐리티를 위한 web.xml 설정
<br/>
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="spring4-chap16" version="3.0">
  <display-name>spring4-chap16</display-name>
  
  <context-param>
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:/spring-security.xml</param-value>
  </context-param>
  
  <listener>
  	<listener-class>
  		org.springframework.web.context.ContextLoader
  	</listener-class>
  </listener>
  
  <servlet>
  	<servlet-name>dispatcher</servlet-name>
  	<servlet-class>
  		org.springframework.web.servlet.DispatcherServlet
  	</servlet-class>
  	<init-param>
  		<param-name>contextConfigLocation</param-name>
  		<param-value>
  			classpath:/spring-mvc.xml
  		</param-value>
  	</init-param>
  	<load-on-startup>1</load-on-startup>
  </servlet>
  
  <servlet-mapping>
  	<servlet-name>dispatcher</servlet-name>
  	<url-pattern>/</url-pattern>
  </servlet-mapping>
  
  <filter>
  	<filter-name>springSecurityFilterChain</filter-name>
  	<filter-class>
  		org.springframework.web.filter.DelegatingFilterProxy
  	</filter-class>
  </filter>
  <filter-mapping>
  	<filter-name>springSecurityFilterChain</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
  
  <filter>
  	<filter-name>encodingFilter</filter-name>
  	<filter-class>
  		org.springframework.web.filter.CharacterEncodingFilter
  	</filter-class>
  	<init-param>
  		<param-name>encoding</param-name>
  		<param-value>UTF-8</param-value>
  	</init-param>
  </filter>
  <filter-mapping>
  	<filter-name>encodingFilter</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
```

먼저 첫 번째로 스프링 시큐리티를 위한 스프링 설정은 루트 어플리케이션 컨텍스트에서 사용해야 한다. 스프링 시큐리티가 제공하는 JSP용 커스텀 태그 라이브러리가 정상 동작하려면 스프링 시큐리티의 주요 구성 요소가 루트 어플리케이션에 위치해야 한다.  

두 번째로 눈여겨볼 부분은 DelegationFilterProxy다. 이 필터는 스프링 빈 객체를 필터로 쓰고 싶을 때 사용하는데, 위 설정에서 사용할 스프링 빈의 이름을 "springSecurityFilterChain"로 설정했다. 여기서 "springSecurityFilterChain"이라는 이름을 갖는 스프링 빈을 설정한 적이 없는데 이 필터가 왜 출현했을까?  

실제 "springSecurityFilterChain"이라는 이름의 스프링 빈은 스프링 시큐리티 설정파일에서 스프링 시큐리티 네임스페이스를 처리하는 과정에서 등록된다. 스프링 시큐리티 네임스페이스를 사용하면 내부적으로 FilterChainProxy 객체를 스프링 빈으로 등록하는데, 이 FilterChainProxy 빈의 이름이 "springSecurityFilterChain"이다. 스프링 시큐리티의 웹 모듈은 여러 서블릿 필터를 이용해서 접근 제어, 로그인/로그아웃 등의 기능을 제공하는데, FilterChainProxy은 이들 보안 관련 서블릿 필터들을 묶어서 실행해주는 기능을 제공한다.  

FilterChainProxy 프록시가 여러 보안 관련 서블릿 필터를 묶어서 실행한다고 했는데, 그렇다면 보안 관련 서블릿 필터는 또 어디에 있을까? 사실 이미 앞에서 보안 관련 서블릿 필터를 빈으로 등록하기 위한 설정을 했다. 스프링 시큐리티 설정에서 보안 관련 서블릿 필터를 생성하기 위한 설정을 했다.  

스프링 시큐리티 네임스페이스 핸들러는 입력받은 설정 정보를 이용해서 보안 관련 서블릿 필터 체인을 생성한다. 예를 들어, &#60;intercept-url&#62; 태그로 입력 받은 설정을 사용해서 FilterSecurityInterceptor 필터를 생성하고, &#60;form-login&#62; 설정을 이용해서 폼 기반 로그인 요청을 처리하는 UsernamePaswordAuthenticationFilter를 생성한다. 비슷하게 &#60;logout&#62; 설정은 LogoutFilter 필터를 생성하는데 사용된다. 이렇게 생성한 필터는 체인을 형성하고, FilterChainProxy는 클라이언트의 웹 요청이 들어오면 이 체인을 이용해서 접근 제어를 하게 된다.  

#### 2.6. 예제에서 사용할 JSP 코드
<br/>
#### 2.7. 접근 제어 적용 확인
<br/>
### 3. 스프링 시큐리티 구조 개요
<br/>
스프링 시큐리티의 설정 자체는 간단하지만, 스프링 시큐리티가 지원하지 않는 인증 방식을 사용해야 한다거나 HttpSession이 아닌 다른 장소에 인증 객체를 보관하기 위해서는 스프링 시큐리티의 동작 방식을 이해하고 그중 필요한 부분의 기능을 알맞게 변경할 수 있어야 한다. 스프링 시큐리티의 구조는 복잡하진 않지만 그렇다고 단순하지도 않기 때문에, 자신의 환경에 스프링 시큐리티를 적용할 수 있으려면, 스프링 시큐리티의 주요 구성 요소와 전반적인 동작 방식에 대한 이해가 필요하다.  

#### 3.1. SecurityContext, SecurityContextHolder, Authentication, GrantedAuthority  
<br/>
org.springfraemwork.security.core.Authentication은 스프링 시큐리티에서 현재 어플리케이션에 접근한 사용자(더 정확하게는 웹 브라우저, REST로 접근한 외부 시스템 등)의 보안 관련 정보를 보관하는 역할을 한다. 예를 들어, Authentication은 사용자의 인증 여부, 사용자가 가진 권한(authority), 이름 및 접근 주체(principal)에 대한 정보를 제공한다. 스프링 시큐리티는 이 정보를 이용해서 사용자가 요청한 자원(URL 등)에 접근할 수 있는지 여부를 판단한다.  

Principal을 보통 '주체'로 많이 번역하는데 위키피디아에 따르면 Principal을 "An entity that can be authenticated by a computer system or network, It is referred to as a security principal in Java and Microsoft literature."로 정의하고 있다. 즉, 인증 가능한 개체를 Principal이라고 정의하고 있다.  

스프링 시큐리티가 Authentication을 사용하려면 어딘가에서 Authentication 객체를 가져와야 하는데, 이 때 사용하는 것이 org.springframework.security.core.context.SecurityContextHolder 클래스다. SecurityContextHolder 클래스의 getContext() 메서드는 SecurityContext 객체를 리턴하며, 이 SecurityContext 객체의 getAuthentication() 메서드를 이용해서 Authentication 객체를 구할 수 있다.  

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication auth = context.getAuthentication();
```

org.springframework.security.core.context.SecurityContext는 Authentication 객체를 보관하는 역할을 한다. 스프링 시큐리티는 웹 브라우저로부터 요청이 들어오면 서블릿 필터를 이용해서 SecurityContext에 Authentication 객체를 설정한다. 이를 위한 서블릿 필터를 직접 구현한다면 아마도 다음과 같은 코드를 만들어야 할 것이다.  

```java
// 웹 요청을 가장 먼저 받는 필터에서 Authentication을 생성해서 SecurityContext에 보관
Authentication auth = someMethodForGettingAuth(request, response);
try {
	SecurityContextHolder.getContext().setAuthentication(auth);
	chain.doFilter(request, response);
} finally {
	SecurityContextHolder.clearContext();
}
```

위 코드에서 someMethodForGettingAuth() 메서드는 HttpSession이나 다른 외부 저장소 등에서 현재 접속한 클라이언트에 해당하는 Authentication 객체를 생성하도록 구현하게 될 것이다. 위와 같이 동작하는 서블릿 필터를 직접 만들수도 있지만, 대부분은 스프링 시큐리티가 제공하는 SecurityContextPersistenceFilter 클래스의 기능을 활용해서 SecurityContext를 설정하기 위한 코드 양을 줄일 수 있다.  

스프링 시큐리티는 SecurityContextPersistenceFilter를 가장 먼저 적용해서 SecurityContext에 Authentication 객체를 보관한다. 이후 스프링 시큐리티는 SecurityContext를 이용해서 현재 접속한 사용자의 Authentication 객체를 구하고, Authentication 객체가 표현하는 주체가 접속한 자원에 접근할 수 있는지 여부를 판단하게 된다.  

SecurityContextHolder는 기본으로 ThreadLocal을 이용해서 SecurityContext 객체를 보관한다. 따라서, 하나의 웹 요청을 처리하는 쓰레드는 같은 SecurityContext 객체(그리고 Authentication) 객체를 사용하게 된다.  

##### 3.1.1. Authentication 인터페이스
<br/>
스프링 시큐리티가 제공하는 코드가 아닌 코드를 직접 작성하여 org.springframework.security.core.Authentication 객체를 사용해야 할 때가 있다. 이런 경우에는 앞서 살펴본 SecurityContextHolder를 이용해서 Authentication 객체를 구하면 된다.  

```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
```

Authentication 객체를 구했다면, Authentication 객체가 제공하는 메서드를 이용해서 필요한 정보를 구할 수 있다. 이들 메서드는 다음과 같다.  

+ String getName() : 이름을 구한다.
+ Object getCredentials() : 인증 대상 주체를 증명하는 값을 구한다. 비밀번호 등이 이에 해당한다.
+ Object getPrincipal() : 주체를 표현하는 객체를 구한다.
+ Object getDetails() : 주체에 대한 상세 정보를 구한다. 접속한 IP 등의 정보를 저장하는 용도로 사용할 수 있다.  
+ boolean isAuthenticated() : 인증 여부를 리턴한다.
+ void setAuthenticated(boolean authenticated) : 인증 여부를 설정한다.
+ Collection&#60;? extends GrantedAuthority&#62; getAuthorities() : 주체가 가진 권한 목록을 구한다. GrantedAuthority가 권한을 의미한다.  

Authentication 타입은 다음의 두 가지 목적으로 사용된다.  

+ AuthenticationManager에 인증을 요청할 때 피룡한 정보를 담는 목적
+ 현재 접속한 사용자에 대한 정보를 표현하기 위한 목적  

먼저, 스프링 시큐리티는 인증을 위한 목적으로 Authentication 객체를 사용한다. 스프링 시큐리티는 AuthenticationManager를 사용해서 인증을 처리하는데, 이 AuthenticationManager가 입력으로 받는 값의 타입이 Authentication이다.  

두 번째로 현재 사용자에 대한 정보를 표현하기 위해 Authentication 객체를 사용한다. 스프링 시큐리티는 SecurityContext에 보관된 Authentication 객체를 가져와, 이 Authentication 객체가 지정한 자원(URL 경로)에 접근할 수 있는지 검사한다. 따라서, 스프링 시큐리티 프레임워크를 잘 사용하려면 알맞은 Authentication 객체를 생성해주어야 한다.  

예를 들어, 스프링 시큐리티는 UsernamePasswordAuthenticationToken 클래스를 제공하고 있는데, 이 클래스는 사용자 아이디와 암호를 이용해서 인증을 처리할 때 사용되는 Authentication 구현체다. 또한, AnonymousAuthenticationToken 클래스를 제공하고 있는데, 이 클래스는 아직 인증을 거치지 않은 사용자를 표현하기 위한 구현체로 사용된다. 스프링 시큐리티가 이미 다양한 상황에 맞게 사용할 수 있는 Authentication 구현 클래스를 제공하고 있지만, 필요에 따라 직접 알맞은 Authentication 구현체를 구현해야 할 때도 있다.  

##### 3.1.2. GrantedAuthority 인터페이스
<br/>
org.springframework.security.core.GrantedAuthority 인터페이스는 권한을 표현할 때 사용된다. Authentication 인터페이스의 getAuthorities() 메서드는 사용자가 가진 권한 목록을 리턴한다고 했는데, 이때 GrantedAuthority를 사용했다. 이 인터페이스는 다음과 같이 정의되어 있다.  

```java
public interface GrantedAuthority extends Serializable {
	String getAuthority();
}
```

GrantedAuthority의 getAuthority() 메서드의 리턴 타입은 String인데, 이는 스프링 시큐리티가 모든 권한을 문자열로 표현한다는 것을 의미한다. 예를 들어, 접근 권한을 설정할 때 사용한 코드를 보면, 아래 코드처럼 "USER&#95;MANAGER"와 같은 문자열을 사용해서 표현했다.  

```xml
<sec:intercept-url pattern="/admin/usermanager/**" access="hasAuthority('USER_MANAGER')" />
```

org.springframework.security.core.authority.SimpleGrantedAuthority 클래스는 GrantedAuthority 타입의 객체를 직접 생성해야 할 때 사용할 수 있는 클래스로서, 이 클래스는 생성자를 이용해서 String 타입의 권한 값을 전달받는다.  

```java
GrantedAuthority authority = new SimpleGrantedAuthority("USER_MANAGER");
```

#### 3.2. 보안 필터 체인
<br/>
스프링을 이용한 웹 보안 처리에서 핵심 요소 중의 하나를 꼽으라면 보안 필터 체인을 들 수 있다. 로그인 폼을 보여준다거나, 접근 권한이 없는 경우 403 상태 코드를 응답하거나, 특정 경로로 요청이 들어올 때 로그아웃을 하는 등 보안과 관련된 작업을 처리하는 것이 바로 보안 필터 체인이다.  

web.xml에 설정한 DelegatingFilterProxy는 스프링 시큐리티가 생성하는 FilterChainProxy에 필터 처리를 위임하는데, 이 FilterChainProxy는 다시 여러 필터 체인 형식으로 갖고 있는 SecurityFilterChain에 처리를 위임한다.  

만약 현재 접속한 사용자가 실제 자원에 접근할 권한이 없다면 보안 필터 체인은 사용자가 실제 자원에 접근하는 것을 차단하고, 403과 같은 응답을 전송하게 된다. 보안 필터 체인을 어떻게 구성했냐에 따라 에러 화면을 보여주거나 로그인 폼을 보여주기도 한다. 즉, 보안 필터 체인은 현재 사용자가 실제 자원에 접근 가능한지 여부를 따져서, 접근 권한이 있는 사용자만 통과시켜 주고 권한이 없는 사용자는 차단하는 기능을 제공한다.  

보안 필터 체인이라는 이름에서 알 수 있듯이, 여러 필터가 모여서 하나의 보안 필터 체인을 형성하게 된다. 스프링 시큐리티는 보안 필터 체인에서 사용되는 기본적인 필터를 이미 제공하고 있다.  

보안 필터 체인이 갖고 있는 필터 목록에서 위에 위치한 필터가 먼저 적용되고 아래에 위치한 필터가 나중에 적용된다. 웹 어플리케이션을 위한 스프링 시큐리티의 설정은 대부분 이 필터들을 위한 설정이다. 예를 들어, 아래 코드를 이용하면 LogoutFilter, FilterSecurityInterceptor, DefaultLoginPageGeneratingFilter 그리고 UsernamePasswordAuthenticationFilter를 생성한다.  

```xml
<sec:http use-expression="true">
	<sec:intercept-url pattern="/**" access="permitAll"/>
	<sec:form-login />
	<sec:logout />
</sec:http>
```

별도 설정을 하지 않아도 몇몇 필터는 기본으로 사용된다. 예를 들어, SecurityContextPersistenceFilter, AnonymousAuthenticationFilter, ExceptionTranslationFilter 등의 필터는 별도로 설정하지 않아도 기본으로 포함된다.  

각 필터는 자신만의 고유 역할을 갖고 있다. 예를 들어, FilterSecurityInterceptor는 사용자가 자원에 접근할 수 있는지 여부를 검사하며, LogoutFilter는 로그아웃 경로로 요청이 오면 로그아웃 처리를 한다. 스프링 시큐리티가 제공하는 주요 필터의 역할은 다음과 같다. 스프링 시큐리티가 제공하는 주요 필터의 역할은 다음과 같다.  

+ ChannelProcessingFilter  
HTTP 요청일 경우 같은 경로를 갖는 HTTPS 경로로 리다이렉트시키고, 이후 필터를 진행하지 않는다.
+ SecurityContextPersistenceFilter  
SecurityContext 객체를 SecurityContextHolder에 저장하고, 요청 처리가 끝나면 제거한다.
+ CsrfFilter  
CSRF 공격을 막기 위한 처리를 한다.
+ LogoutFilter  
지정한 경로의 요청이 오면 로그아웃을 처리하고 지정한 페이지로 이동한다. 이후 필터를 진행하지 않는다.  
+ UsernamePasswordAuthenticationFilter  
지정한 경로로 POST 방식 요청이 오면, 기본 제공하는 로그인 폼을 출력하고 이후 필터를 진행하지 않는다.  
+ RememberMeAuthenticationFilter  
시스템 간의 연동을 위한 메시징 프레임워크를 제공한다.  
+ AnonymousAuthenticationFilter  
현재 사용자가 인증 전일 경우, 임의 사용자에 해당하는 Authentication 객체를 SecurityContext에 설정한다. 생성된 Authentication 객체는 이름이 "anonymousUser"이고, "ROLE&#95;ANONYMOUS" 권한을 가지며, 인증되지 않은 상태 값을 갖는다.  
+ SessionManagementFilter  
세션 타임아웃, 동시 접근 제어, 세션 고정 공격 등을 처리한다.  
+ ExceptionTranslationFilter  
FilterSecurityInterceptor에서 발생한 익셉션을 웹에 맞는 응답으로 변환한다. 예를 들어. 403 상태 코드를 응답하거나 로그인 페이지로 이동하는 등의 작업을 수행한다.  
+ FilterSecurityInterceptor  
현재 주체가 지정한 자원에 접근할 수 있는지 여부를 검사한다. 권한이 있으면 보안 필터를 통과시켜 자원에 접근할 수 있게 하고, 권한이 없으면 익셉션을 발생시킨다.  

보안 필터 체인에서 각 필터는 올바른 순서대로 위치해야 한다. 예를 들어, 스프링 시큐리티의 기능이 올바르게 동작하려면 SecurityContext와 Authentication이 올바르게 생성되어 있어야 하기 때문에, SecurityContext를 생성해주는 SecurityContextPersistenceFilter를 다른 필터보다 먼저 적용해야 한다. 또한, FilterSecurityInterceptor 보다 LogoutFilter가 먼저 적용되어야 로그아웃 경로로 요청이 들어올 때 LogoutFilter가 먼저 로그아웃 처리를 해서 FilterSecurityInterceptor가 실행되지 않을 것이다.  

실제 스프링 시큐리티는 다음에 표시한 순서대로 필터를 적용한다.  

+ ChannelProcessingFilter
+ SecurityContextPersistenceFilter
+ LogoutFilter
+ X509AuthenticationFilter
+ AbstractPreAuthenticatedProcessingFilter
+ CasAuthenticationFilter
+ UsernamePasswordAuthenticationFilter
+ OpenIDAuthenticationFilter
+ DefaultLoginPageGeneratingFilter
+ ConcurrentSessionFilter
+ DigestAuthenticationFilter
+ BasicAuthenticationFilter
+ RequestCacheAwareFilter
+ SecurityContextHolderAwareRequestFilter
+ JaasApiIntegrationFilter
+ RememberMeAuthenticationFilter
+ AnonymousAuthenticationFilter
+ SessionManagementFilter
+ ExceptionTranslationFilter
+ FilterSecurityInterceptor
+ SwitchUserFilter  

직접 구현한 필터를 보안 필터 체인에 추가해야 할 경우 필터가 추가될 위치를 지정해야 하는데, 이때 위 필터 이름을 사용하게 된다.  

#### 3.3. AuthenticationManager의 인증 처리
<br/>
스프링 시큐리티는 인증이 필요할 때 org.springframework.security.authentication.AuthenticationManager를 이용한다. 이 인터페이스는 다음과 같이 단순하게 정의되어 있다.  

```java
public interface AuthenticationManager {
	Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

authenticate() 메서드는 인증하는데 필요한 정보를 담은 Authentication 객체를 입력으로 전달받는다. 인증에 성공하면 인증 정보를 담은 Authentication 객체를 리턴하고, 그렇지 않을 경우 익셉션을 발생한다.  

일반적으로 사용자 이름(아이디)과 암호를 사용해서 사용자가 누구인지 인증한다. 실제 스프링 시큐리티가 기본으로 제공하는 인증 관련 기능도 아이디와 암호를 사용해서 인증을 처리하고 있다. 이런 이유로 스프링 시큐리티의 많은 클래스가 앞서 언급한 것처럼 UsernamePasswordAuthenticationToken을 Authentication의 기본 구현체로 사용하고 있다.  

스프링이 제공하는 AuthenticationManager 인터페이스의 구현 클래스로 ProviderManager 클래스를 제공하고 있는데, 이 클래스는 인증 처리를 AuthenticationProvider에게 위임한다.  

ProviderManager는 한 개 이상의 AuthenticationProvider를 가질 수 있으며, 다음곽 같은 방식으로 동작한다.  

(1) 등록된 AuthenticationProvider에 대해 차례대로 다음 과정을 실행한다.
&nbsp;&nbsp;&nbsp;&nbsp;A. authenticate() 메서드를 실행해서 인증 처리를 요청한다.  
&nbsp;&nbsp;&nbsp;&nbsp;B. authenticate()가 Authentication 객체를 리턴하면, 해당 객체를 리턴한다.  
(2) 어떤 AuthenticationProvider도 인증에 성공하지 못할 경우, 익셉션을 발생한다.  

스프링 시큐리티의 인증 부분을 커스터마이징해야 할 경우, AuthenticationManager의 커스텀 구현을 제공할 수도 있지만 보통은 ProviderManager를 그대로 사용하고 AuthenticationProvider을 구현하는 방법을 선택한다. 또한, 스프링 시큐리티는 AuthenticationProvider의 몇 가지 기본 구현체를 제공하고 있기 때문에, 특수한 상황이 아니면 기본 구현체로도 충분한다. 스프링 시큐리티가 제공하는 AuthenticationProvider의 주요 구현체에는 다음과 같다.  
+ DaoAuthenticationProvider : DAO를 이용해서 사용자 정보를 읽어와 인증을 처리한다.
+ LdapAuthenticationProvider : LDAP 서버나 액티브 디렉토리를 이용해서 인증을 처리한다.
+ OpenIDAuthenticationProvider : 오픈ID를 이용한 인증을 처리한다.  

앞에서 다음과 같이 사용자 정보를 설정했는데, 이 설정은 DaoAuthenticationProvider를 생성한다. DaoAuthenticationProvider는 내부적으로 UserDetailsService를 이용해서 사용자 정보를 읽어오는데, 아래 설정은 메모리를 이용해서 사용자 정보를 제공하는 InMemoryUserDetailsManager를 사용하도록 설정한다.  
	
```xml
<sec:authentication-manager>
	<sec:authentication-provider>
		<sec:user-service>
			<sec:user name="bkchoi" password="1234" authorities="ROLE_USER"/>
			<sec:user name="manager" password="qwer" authorities="ROLE_MANAGER"/>
			<sec:user name="admin" password="admin" authorities="ROLE_ADMIN.ROLE_USER" />
		<sec:user-service>
	</sec:authentication-provider>
</sec:authentication-manager>
```

실제 환경에서는 메모리가 아닌 DB를 사용해서 인증 정보를 읽오올 것이다.  

#### 3.4. FilterSecurityInterceptor와 AccessDecisionManager의 인가 처리
<br/>
권한 검사를 위한 설정 코드를 다시 보자.  

```xml
<sec:http use-expressions="true">
	<sec:intercept-url pattern="/admin/**" access="hasAuthority('ROLE_ADMIN)"/>
	<sec:intercept-url pattern="/manager/**" access="hasRole('ROLE_MANAGER')" />
	<sec:intercept-url pattern="/member/**" access="isAuthenticated()" />
	<sec:intercept-url pattern="/**" access="permitAll"/>
	<sec:form-login />
	<sec:logout />
</sec:http>
```

&#60;intercept-url&#62; 태그는 지정한 경로 패턴별로 접근 권한을 지정해주는데. 스프링 시큐리티는 이 설정을 이용해서 다음의 세 구성 요소를 설정한다.  

+ FilterSecurityInterceptor
+ FilterInvocationSecurityMetadataSource
+ AccessDecisionManager  

앞서 보안 필터 체인에서 FilterSecurityInterceptor는 체인의 가장 마지막에 위치했었다. 체인의 앞쪽에 위치한 SecurityContextPersistenceFilter나 AnonymousAuthenticationFilter 등의 필터가 SecurityContextHolder에 SecurityContext를 설정하면, 마지막에 위치한 FilterSecurityInterceptor는 SecurityContext에 보관된 Authentication가 요청 경로(보안 대상)에 접근할 수 있는 지 여부를 검사하게 된다.  

FilterSecurityInterceptor가 접근 가능 여부를 검사하는 과정은 다소 복잡한데, 이 과정에서 앞서 언급한 세 개의 구성 요소가 사용된다. 이 과정을 간략하게 정리하면 다음과 같이 표현할 수 있다.  

(1) 보안 필터 체인을 거쳐 Authentication 객체를 SecurityContext에 저장하게 되고, FilterSecurityInterceptor가 실행된다.  
(2) FilterSecurityInterceptor는 요청 경로(FilterInvocation)에 대한 보안 설정(ConfigAttribute) 정보를 FilterInvocationSecurityMetadataSource에 요청한다.  
(3) FilterInvocationSecurityMetadataSource는 보안 설정 목록을 리턴한다.  
(4) FilterSecurityInterceptor는 AccessDecisionManager의 decide() 메서드를 호출해서 Authentication이 요청 경로(FilterInvocation)에 대해 보안 설정(ConfigAttribute)을 충족하는지 검사한다.  

AccessDecisionManager는 현재 사용자(Authentication)가 지정한 자원에 대한 접근 권한이 없다면 AccessDeniedException을 발생시킨다. AccessDeniedException이 발생하면 FilterSecurityInterceptor는 익셉션을 재발생해서 필터 체인의 앞쪽에 위치한 필터에서 익셉션을 처리하도록 한다.  

AccessDecisionManager가 익셉션을 발생하지 않으면 권한이 있는 것으로 간주하고 FilterSecurityInterceptor는 사용자가 요청한 경로를 실행한다.  

##### 3.4.1. AccessDecisionManager와 AccessDecisionVoter
<br/>
AccessDecisionManager 인터페이스는 다음과 같이 정의되어 있다.  

```java
public interface AccessDecisionManager {
	void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes) throws AccessDeniedException, InsufficientAuthenticationException;

	boolean supports(ConfigAttribute attribute);
	boolean supports(Class<?> clazz);
}
```

decide() 메서드는 현재 사용자를 표현하는 authentication 객체가 보안 대상 객체인 object에 대해 configAttribute에 지정한 권한을 갖고 있는지 검사한다. 만약 권한을 갖고 있지 않으면 AccessDeniedException을 발생시키고, 권한을 갖고 있으면 익셉션을 발생하지 않고 리턴한다.  

실제 스프링이 기본으로 사용하는 AccessDecisionManager 구현 클래스는 AffirmativeBased 클래스인데, 이 클래스는 접근 권한을 가졌는지의 여부를 직접 판단하지 않고 AccessDeicisionVoter 타입의 객체에 한 번 더 위임한다.  

AccessDecisionVoter 인터페이스는 다음과 같이 정의되어 있다. 이 인터페이스의 vote() 메서드는 authentication이 보안 대상 object에 대해 attribute 권한을 갖고 있으면 ACCESS&#95;GRANTED를 리턴하고 권한이 없으면 ACCESS&#95;DENIED를 리턴한다. AccessDecisionVoter가 권한을 가졌는지 여부를 판단할 수 없으면(예, 지원하지 않는 ConfigAttribute 타입이나 보안 대상 객체 타입), ACCESS&#95;ABSTAIN을 리턴한다.  

```java
public interface AccessDecisionVoter<S> {
	int ACCESS_GRANTED = 1;
	int ACCESS_ABSTAIN = 0;
	int ACCESS_DENIED = -1;

	boolean supports(ConfigAttribute attribute);
	boolean supports(Class<?> clazz);
	int vote(Authentication authentication, S object, Collection<ConfigAttribute> attribute);
}
```

AffirmativeBased 클래스의 decide() 메서드는 등록된 AccessDecisionVoter 객체들의 vote() 메서드를 차례대로 호출하고 결과에 따라 다음과 같이 동작한다.  

+ 등록된 AccessDecisionVoter 객체의 vote() 메서드 중 하나라도 ACCESS&#95;GRANTED를 리턴하면, 권한이 있는 것으로 간주해서 정상 리턴한다.
+ 한 개의 AccessDecisionVoter 객체도 ACCESS&#95;GRANTED를 리턴하지 않으면 권한이 없는 것으로 판단해 AccessDeniedException을 발생한다.  

앞서 설정을 보면 다음과 같이 &#60;http&#62; 태그의 use-expression 속성의 값을 true로 지정했는데, 이 경우 AccessDecisionVoter의 구현체로 WebExpressionVoter 클래스를 사용한다.  

위 코드에 표시한 "hasAuthority('ROLE&#95;ADMIN')"이나 "hasRole()", "authenticated" 등은 스프링 시큐리티가 제공하는 스프링 표현식(SpEL)인데, WebExpressionVoter는 이 SpEL로 설정한 권한을 가졌는지 검사한다.  

스프링 시큐리티가 제공하는 AccessDecisionManager 구현 클래스에는 AffirmativeBased 클래스를 포함해 다음의 세 클래스가 존재한다.  

+ AffirmativeBased  
한 개의 AccessDecisionVoter라도 ACCESS&#95;GRANTED를 리턴하면 접근 허용으로 간주한다. 기본으로 사용된다.
+ ConsensusBased  
각 AccessDecisionVoter가 리턴한 ACCESS&#95;GRANTED의 개수가 ACCESS&#95;DENIED 보다 크거나 같으면 접근 허용으로 간주한다. (같을 때 접근 금지로 처리할지 여부를 설정 가능한다.)
+ UnanimousBased  
한 개의 AccessDecisionVoter라도 ACCESS&#95;DENIED를 리턴하면, 접근 금지로 결정한다.  

### 4. 웹 요청 인가 설정 표현식
<br/>
접근 제어를 위한 설정 코드 부분을 다시 보자.  

```xml
<sec:http use-expressions="true">
	<sec:intercept-url pattern="/admin/**" access="hasAuthority('ROLE_ADMIN)"/>
	<sec:intercept-url pattern="/manager/**" access="hasRole('ROLE_MANAGER')" />
	<sec:intercept-url pattern="/member/**" access="isAuthenticated()" />
	<sec:intercept-url pattern="/**" access="permitAll"/>
	<sec:form-login />
	<sec:logout />
</sec:http>
```

&#60;intercept-url&#62; 태그의 access 속성은 스프링 시큐리티가 제공하는 스프링 표현식을 사용해서 접근 권한을 설정하고 있다. 스프링 시큐리티는 위 코드에 보여준 것 말고도 접근제어를 위해 사용할 수 있는 다양한 표현식을 제공하고 있는데, 주요 표현식은 다음과 같다.  

+ hasRole('권한'), hasAuthority('권한')  
해당 권한을 가졌는지 검사한다.
+ hasAnyRole('권한1, 권한2'), hasAnyAuthrity('권한1, 권한2')  
지정한 권한 중 하나라도 가졌는지 검사한다. 각 권한은 콤마로 구분한다.
+ permitAll  
모두 허용한다.
+ denyAll  
모두 거부한다.
+ isAnonymous()  
임의 사용자인지 검사한다.
+ isAuthenticated()  
인증된 사용자인지 검사한다. 기억된 사용자도 인증 사용자로 처리한다.
+ isRememberMe()  
기억된 사용자인지 검사한다.
+ isFullyAuthenticated()  
완전한 인증을 거친 사용자인지 검사한다. 기억된 사용자인 경우 완전한 인증을 거치지 않은 것으로 판단하며, 실제 로그인 과정을 거쳐야 완전한 인증 사용자로 처리한다.
+ hasIpAddress('IP표현')  
클라이언트가 지정한 IP인지 검사한다. 특정 IP 뿐만 아니라 192.168.1.0/24와 같은 CIDR로 범위를 지정할 수 있다.  

### 5. 상황별 스프링 시큐리티 설정
<br/>
스프링 시큐리티를 실제 프로젝트에 적용하려면 상황에 맞게 교체하거나 설정해야 하는 부분이 있다.  

#### 5.1. 일부 경로 스프링 시큐리티 적용 안하기
<br/>
앞서 스프링 시큐리티 구조의 '보안 필터 체인'에서 설명했듯이 스프링 시큐리티 웹 모듈은 필터를 이용해서 접근을 제어한다. 그런데, 모든 경로에 '보안 필터 체인'을 적용한 필요는 없을 것이다. 예를 들어, CSS나 JS, 또는 이미지 등의 경로는 (많은 경우) 접근 제어 대상이 아니기 때문에, 보안 필터 체인을 적용하지 않아도 된다. 이렇게 보안 필터 체인을 적용할 필요가 없는 대상이 있다면, 다음과 같이 &#60;http&#62; 태그를 설정해주면 된다.  

```xml
<sec:http pattern="/css/**" security="none" />
<sec:http pattern="/js/**" security="none" />
```

첫 두 개의 &#60;http&#62; 태그는 security 속성의 값을 "none"으로 설정했다. 이 경우 pattern 속성으로 지정한 경로에 대해서 스프링 시큐리티의 필터 체인을 적용하지 않는다. 이렇게 되면 FilterSecurityInterceptor가 적용되지 않으므로 "/css/"로 시작하는 경로나 "/js/"로 시작하는 경로에 대해 보안 검사를 하지 않게 된다.  

단, 보안 필터 체인을 적용하지 않는다는 것은 SecurityContextPersistenceFilter도 적용하지 않는다는 것을 뜻한다. SecurityContextPersistenceFilter는 SecurityContext와 Authentication 객체를 사용할 수 있게 설정해주는 기능을 제공하기 때문에, 보안 필터 체인을 적용하지 않는 경로는 SecurityContextHolder로부터 SecurityContext를 구할 수 없게 된다.  

#### 5.2. DB를 이용한 인증 처리
<br/>
앞서 예제는 메모리에 사용자 정보를 설정해서 인증을 수행하는데, 실제로는 메모리가 아닌 외부 저장소에 있는 사용자 DB를 이용해서 인증을 처리할 것이다. 스프링 시큐리티가 기본으로 제공하는 기능을 사용하면 매우 간단한 설정으로 DB를 이용해서 사용자 인증 처리를 할 수 있다.  

##### 5.2.1. 사용자 및 권한 매핑 DB 테이블 생성
<br/>
먼저 할 일은 사용자 정보와 매핑 정보를 담을 테이블을 생성하는 것이다.  

##### 5.2.2. 스프링 시큐리티 설정
<br/>
생성한 테이블을 이용해서 인증을 수행하도록 설정하는 것은 매우 간단하다. &#60;jdbc-user-service&#62; 태그를 추가해주기만 하면 된다.  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:sec="http://www.springframework.org/schema/security"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
					http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">

	<sec:authentication-manager alias="authenticationManager">
		<sec:authentication-provider>
			<sec:jdbc-user-service data-source-ref="dataSource"/>
		</sec:authentication-provider>
	</sec:authentication-manager>
	
	<sec:http use-expressions="true">
		<sec:intercept-url pattern="/admin/usermeanager/**" access="hasAuthority('USER_MANAGER')" />
		<sec:intercept-url pattern="/member/**" access="hasAuthority('USER')"/>
		<sec:intercept-url pattern="/**" access="permitAll"/>
		<sec:form-login />
		<sec:logout/>
	</sec:http>
</beans>
```

&#60;authentication-provider&#62; 태그에 &#60;jdbc-user-service&#62; 자식 태그를 추가하면, DB를 이용해서 인증에 필요한 사용자 정보를 조회한다. &#60;jdbc-user-service&#62; 태그의 data-source-ref 속성은 사용자 정보 DB에 연결할 때 사용할 DataSource를 지정한다. 더 할 것은 없다. 이걸로 설정이 끝났다.  

스프링 시큐리티에서 사용할 DataSource는 스프링 설정 파일에 정의한다.  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
		<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="jdbcUrl" value="jdbc:mysql://localhost/ssuserdb?characterEncoding=utf8" />
		<property name="user" value="spring4" />
		<property name="password" value="spring4" />
	</bean>
</beans>
```

##### 5.2.3. 데이터 준비 및 스프링 MVC 설정
<br/>
스프링 MVC는 다음과 같이 설정한다.  

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
					http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

	<mvc:annotation-driven/>
	
	<mvc:default-servlet-handler/>
	
	<mvc:view-controller path="/index" view-name="index" />
	<mvc:view-controller path="/admin/usermanager/main" view-name="usermanagerMain" />
	<mvc:view-controller path="/member/main" view-name="memberMain" />
	
	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/view/" />
		<property name="suffix" value=".jsp" />
	</bean>
</beans>
```

스프링 MVC의 DispatcherServlet 및 스프링 시큐리티 적용을 위한 web.xml 설정은 다음과 같이 작성한다.  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="spring4-chap16" version="3.0">
  <display-name>spring4-chap16-s1</display-name>
  
  <context-param>
  	<param-name>contextConfigLocation</param-name>
  	<param-value>
  		classpath:/spring-security-s1.xml
  		classpath:/spring-application.xml
  	</param-value>
  </context-param>
  
  <listener>
  	<listener-class>
  		org.springframework.web.context.ContextLoaderListener
  	</listener-class>
  </listener>
  
  <servlet>
  	<servlet-name>dispatcher</servlet-name>
  	<servlet-class>
  		org.springframework.web.servlet.DispatcherServlet
  	</servlet-class>
  	<init-param>
  		<param-name>contextConfigLocation</param-name>
  		<param-value>
  			classpath:/spring-mvc-s1.xml
  		</param-value>
  	</init-param>
  	<load-on-startup>1</load-on-startup>
  </servlet>
  
  <servlet-mapping>
  	<servlet-name>dispatcher</servlet-name>
  	<url-pattern>/</url-pattern>
  </servlet-mapping>
  
  <filter>
  	<filter-name>springSecurityFilterChain</filter-name>
  	<filter-class>
  		org.springframework.web.filter.DelegatingFilterProxy
  	</filter-class>
  </filter>
  <filter-mapping>
  	<filter-name>springSecurityFilterChain</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
  
  <filter>
  	<filter-name>encodingFilter</filter-name>
  	<filter-class>
  		org.springframework.web.filter.CharacterEncodingFilter
  	</filter-class>
  	<init-param>
  		<param-name>encoding</param-name>
  		<param-value>UTF-8</param-value>
  	</init-param>
  </filter>
  <filter-mapping>
  	<filter-name>encodingFilter</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
```

##### 5.2.4. 사용자 권한에 따라 다른 내용을 보여주는 JSP 코드 예제
<br/>
##### 5.2.5. DB를 이용한 인증 처리 확인
<br/>
#### 5.3. DB를 이용한 인증 처리 구조
<br/>
앞서 스프링 시큐리티의 구조에서 설명했던 것처럼 스프링의 AuthenticationManager 구현 클래스인 ProviderManager는 AuthenticationProvider에 인증 처리를 위임한다. &#60;jdbc-user-service&#62; 태그를 설정하면 AuthenticationProvider의 구현 클래스로 DaoAuthenticationProvider를 사용한다.  

DaoAuthenticationProvider는 사용자 정보를 읽어올 때 UserDetailService 타입의 객체를 사용한다. DaoAuthenticationProvider의 authenticate() 메서드는 다음과 같은 과정을 거쳐 인증을 처리한다.  

(1) UserDetailsService의 loadUserByUsername() 메서드로 사용자 이름(아이디)에 해당하는 UserDetails 객체를 구한다.  
&nbsp;&nbps;&nbsp;&nbsp;A. UserDetailsService는 사용자 이름에 해당하는 UserDetails 객체가 존재하지 않으면 UsernameNotFoundException을 발생시킨다.  
(2) 입력한 암호가 UserDetails 객체의 getPassword()로 구한 암호와 일치하는지 비교한다.  
&nbps;&nbps;&nbsp;&nbsp;A. 두 암호가 일치하지 않으면 BadCredentialsException을 발생시킨다.  
(3) 두 암호가 일치하면, UserDetails 객체로부터 Authentication 객체를 생성해서 리턴한다.  

인증 처리 과정에서 알 수 있겠지만, 실제 DB에서 사용자 정보를 읽어오는 것은 UserDetailsService가 책임지며, &#60;jdbc-user-service&#62; 태그를 이용하면 DB에서 사용자 정보를 읽어오는 JdbcUserDetailsManager 클래스를 UserDetailsService의 구현체로 사용한다.  

JdbcUserDetailsManager 클래스가 JdbcDaoImpl 클래스를 상속받는 것을 알 수 있다. JdbcDaoImpl 클래스와 JdbcUserDetailManager 클래스는 JdbcTemplate을 이용해서 DB 연동을 처리하기 때문에 DB 연결을 제공하는 DataSource를 필요로 한다. &#60;jdbc-user-service&#62; 태그의 data-source-ref 속성에서 지정한 DataSource가 바로 이 용도로 사용된다.  

##### 5.3.1. UserDetailsService와 UserDetails
<br/>
org.springframework.security.core.userdetails.UserDetailsService 인터페이스는 다음과 같이 한 개의 메서드만 정의하고 있다.  

```java
public interface UserDetailsService {
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

loadUserByUsername() 메서드는 username에 해당하는 사용자가 존재하면, 해당 사용자의 정보를 담은 UserDetails 객체를 리턴한다. UserDetails 인터페이스는 사용자 인증을 처리하는데 필요한 정보를 제공하기 위한 목적으로 사용되며, 사용자 이름, 암호, 사용 가능 여부 등을 제공하는 메서드를 정의하고 있다.  

&#60;jdbc-user-service&#62; 태그를 설정하면 사용되는 JdbcUserDetailsManager의 경우 사용자 이름, 암호, 사용 가능 여부(enabled), 권한 목록 정보만 제공할 수 있다.  

##### 5.3.2. 사용자 정보 조회를 위한 쿼리 변경
<br/>
정의한 테이블로부터 사용자 정보를 읽어올 때 사용할 쿼리는 (JdbcUserDetailsManager의 부모 클래스인 JdbcDaoImpl 클래스에 정의되어 있다. 다음은 스프링 시큐리티 소스 코드를 살펴보면 사용자 정보를 읽어오는 쿼리가 미리 정의된 것을 알 수 있다.  

테이블 명명 규칙 등의 이유로 실제 생성한 테이블과 컬럼 이름이 위 쿼리와 다를 수 있는데, 이런 경우에도 &#60;jdbc-user-service&#62; 태그가 제공하는 속성을 이용해서 쿼리를 변경할 수 있다. 쿼리를 변경할 때 사용되는 속성은 다음과 같다.  

<table>
	<thead>
		<tr>
			<td>속성</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>users-by-username-query</td>
			<td>
				사용자 이름으로 UserDetails를 찾을 때 사용되는 쿼리를 입력한다. 쿼리는 다음 규칙을 따라야 한다.<br/><br/>
				<li>select로 선택된 결과 중 세 개 컬럼이 차례대로 사용자 이름, 암호, 사용 가능 여부로 사용된다.</li>
				<li>where 절에 1개의 인덱스 파라미터(물음표)를 포함해야 하며, 이 파라미터에 사용자 이름이 할당된다.</li>
			</td>
		</tr>
		<tr>
			<td>authorities-by-username-query</td>
			<td>
				사용자 이름을 권한 목록을 찾을 때 사용되는 쿼리를 입력한다. 쿼리는 다음 규칙을 따라야 한다.  
				<li>select로 선택된 결과 중 두 번째 컬럼이 권한 값으로 사용된다.</li>
				<li>where 절에 1개의 인덱스 파라미터(물음표)를 포함해야 하며, 이 파라미터에 사용자 이름이 할당된다.</li>
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 5.3.3. UserDetailsManager를 이용한 사용자 관리: 사용자 추가
<br/>
&#60;jdbc-user-service&#62; 태그를 사용하면 JdbcUserDetailsManager 클래스를 구현 클래스로 사용한다고 했는데, 이 JdbcUserDetailsManager 클래스는 UserDetailsManager 인터페이스를 상속받고 있다. UserDetailsManager 인터페이스는 createUser()를 포함해 사용자 정보를 변경할 수 있는 기능을 정의하고 있다. 스프링 시큐리티는 JdbcUserDetailsManager 객체를 빈으로 등록하기 때문에, 이 객체를 이용하면 JdbcUserDetailManager 객체를 이용해서 사용자 테이블에 대한 CRUD 연동을 처리할 수 있게 된다.  

&#60;jdbc-user-service&#62; 태그를 설정할 때 id 속성을 사용하면 JdbcUserDetailsManager 객체를 빈으로 등록할 때 사용할 식별값을 지정할 수 있다.  

```xml
<sec:authentication-manager alias="authenticationManager">
	<sec:authentication-provider>
		<sec:jdbc-user-service data-source-ref="dataSource" id="빈식별자"/>
	</sec:authentication-provider>
</sec:authentication-manager>

```

JdbcUserDetailsManager 객체를 사용하려면 생성자나 설정 메서드를 이용해서 또는 자동 주입을 이용해서 UserDetailsManager 객체나 JdbcUserDetailsManager 객체를 전달받으면 된다. 다음은 서비스 클래스에서 회원 가입을 처리하기 위해 UserDetailsManager를 사용하는 코드의 작성 예이다.  

```java
public class UserJoinService {
	private UserDetailsManager userDetailsManager;

	public void setUserDetailsManager(UserDetailsManager userDetailsManager) {
		this.userDetailsManager = userDetailsManager;
	}

	@Transactional
	public void join(NewUser newUser) {
		UserDetails user = new User(newUser.getName(), newUser.getPassword(), Arrays.asList(new SimpleGrantedAuthority("USER")));

		try {
			userDetailsManager.createUser(user);
		} catch (DuplicatedKeyException ex) {
			throw new DuplicatedUsernameException(String.format("Username [%s] is already exists", newUser.getName()), ex);
		}
	}
}
```

JdbcUserDetailsManager의 createUser() 메서드는 추가할 사용자를 UserDetails 타입으로 전달받는다. 스프링 시큐리티는 UserDetails 인터페이스를 구현한 User 클래스를 제공하고 있으며, User 클래스는 '이름, 암호, 권한 목록'을 생성자로 입력받는다.  

JdbcUserDetailsManager의 createUser() 메서드는 UserDetails 객체의 데이터를 DB 테이블에 삽입할 쿼리를 이미 정의하고 있다.  

아쉽게도 &#60;jdbc-user-service&#62; 태그는 정의된 쿼리를 변경하기 위한 설정을 지원하지 않는다. 따라서, 테이블 이름이나 컬럼 이름등을 변경하고 싶다면 커스텀 UserDetailsService를 등록하는 방법을 사용해야 한다.  

JdbcUserDetailsManager는 create() 뿐만 아니라 update(), changePassword() 등의 기능을 제공하고 있는데, 각 메서드별로 사용하는 SQL 쿼리 및 동작방식은 다음 표와 같다.  

<table>
	<thead>
		<tr>
			<td>메서드</td>
			<td>사용 쿼리</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>createUser(UserDetails)</td>
			<td>
				insert into users (username, password, enabled) values (?, ?, ?)<br/><br/>
				insert into authorities (username, authority) values (?, ?)
			</td>
			<td>사용자 정보를 DB에 삽입한다.</td>
		</tr>
		<tr>
			<td>updateUser(UserDetails)</td>
			<td>
				update users set password = ?, enabled = ? where username = ?<br/><br/>
				delete from authorities where username = ?<br/><br/>
				insert into authorities (username, authorities) values (?, ?)
			</td>
			<td>사용자 정보를 수정한다. 권한 정보를 수정할 때에는 기존 권한 매핑을 모두 제거하고 입력받은 권한 정보를 삽입한다.</td>
		</tr>
		<tr>
			<td>deleteUser(String)</td>
			<td>
				delete from authorities where username = ?<br/><br/>
				delete from users where username = ?
			</td>
			<td>사용자의 권한 매핑 및 사용자 정보 자체를 삭제한다.</td>
		</tr>
		<tr>
			<td>chanagePassword(String oldPassword, String newPassword)</td>
			<td>update users set password = ? where username = ?</td>
			<td>현재 인증된 사용자와 비밀번호를 변경한다. oldPassword가 현재 암호와 일치하지 않으면 익셉션을 발생한다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>

&#42; createUser() 메서드와 updateUser() 메서드는 UserDetails.getAutorities()가 null을 리턴하면 익셉션을 발생시킨다.  

당연한 얘기지만, 사용자 정보를 생성하기 위해 꼭 UserDetailsManager의 createUser() 메서드를 사용해야 하는 것은 아니다. DB 테이터를 관리하는 코드는 직접 만들고, 인증 처리 부분만 스프링 시큐리티가 제공하는 기능을 사용해도 부방하다. 인증 처리만 JdbcUserDetailsManager를 사용할 경우 &#60;jdbc-user-service&#62; 태그가 제공하는 쿼리 설정 속성을 이용해서 쿼리만 알맞게 변경해주면 끝난다.  

##### 5.3.4. 커스텀 UserDetailsService 구현 사용하기
<br/>
스프링 시큐리티가 제공하는 JDBC 연동 인증 구현을 사용하면 매우 빠르게 DB를 이용한 인증 처리를 할 수 있지만, 지정한 형식의 스키마를 가진 테이블을 사용해야 한다는 한계가 있다. 또한, 스프링 시큐리티가 제공하는 기본 기능으로 처리할 수 없는 경우도 있다.  

자신의 환경에 맞게 인증을 처리해야 한다면, 다음의 세 방법 중 하나를 이용해서 커스텀 인증 기능을 구현하고 스프링 시큐리티 프레임워크에 적용해야 한다.  

+ AuthenticationManger 인터페이스를 직접 구현
+ AuthenticationProvider 인퍼페이스를 직접 구현
+ UserDetailsService 인터페이스를 구현  

세 방법 중에서 스프링 시큐리티가 제공하는 기능을 최대한 활용할 수 있는 것은 UserDetailsService 인퍼페이스를 구현하는 것이다. 앞서 보여준 것처럼 DaoAuthenticationProvider의 authenticate() 메서드는 UserDetailsService로부터 사용자 정보(UserDetails)를 읽어와 인증을 수행하는데, 이 과정은 다음과 같다.  

+ UserDetailsService가 UserNotFoundException을 발생하면, 그대로 재발생한다.  
+ 사용자가 입력한 암호와 UserDetails의 암호가 일치하는지 여부를 검사한다. 일치하지 않으면 익셉션을 발생시킨다. (암호가 일치하는지 여부를 확인하기 위해 PasswordEncoder라는 것을 사용한다.)
+ 암호가 일치할 경우 UserDetails로부터 Authentication 객체를 생성해서 리턴한다.  

위 과정을 모두 DaoAuthenticationProvider가 해주기 때문에, UserDetailService 인터페이스를 상속받은 클래스는 loadUserByUsername() 메서드만 올바르게 구현해주면 된다. 다음은 loadUserByUsername() 메서드를 구현할 때 지켜야 할 규칙이다.  

+ username에 해당하는 데이터가 존재하면 UserDetails 타입의 객체를 리턴한다.
+ username에 해당하는 데이터가 존재하지만, 해당 사용자가 어떤 권한(Granted Authority)도 갖고 있지 않을 경우 UserNotFoundException을 발생시킨다.
+ username에 해당하는 데이터가 존재하지 않으면, UserNotFoundException을 발생시킨다.  

UserDetailService 인터페이스의 구현 클래스는 다음과 같은 방식으로 구현한다.  

```java
public class CustomUserDetailsService implements UserDetailsService {

	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		UserInfo userInfo = findUserInfo(username);
		if (userInfo == null)
			throw new UsernameNotFoundException(username);

		List<UserPermission> perms = loadPermission(userInfo.getId());
		List<GrantedAuthority> authorities = new ArrayList<>();
		for (UserPermission perm : perms) {
			authorities.add(new SimpleGrantedAuthority(perm.getName()));
		}
		return new User(username, userInfo.getPassword(). authorities);
	}

	private UserInfo findUserInfo(String username) {
		// 어플리케이션에서 사용하는 사용자 정보 객체를 리턴
	}

	private List<UserPermission> loadPermission(String username) {
		// 어플리케이션에서 사용하는 사용자의 권한 정보 목록을 리턴
	}

	...
}
```

이 코드에서 UserInfo 클래스와 UserPermission 클래스는 직접 구현한 클래스로 어플리케이션에서 사용자와 권한 관리 기능을 구현할 때 사용하는 클래스라고 가정한다. 커스텀 UserDetailsService 클래스는 username에 해당하는 사용자가 있는지 검사하고, 사용자가 존재하면 어플리케이션에서 사용하는 타입으로 권한 정보를 읽어와 스프링 시큐리티가 권한을 표현하는데 사용하는 GrantedAuthority로 변환한다. 그리고, 최종적으로 UserDetails 인터페이스의 구현체 중 하나인 User 객체를 생성해서 리턴한다.  

org.sprignframework.security.core.userdetails.User 클래스는 다음의 생성자를 제공하므로, 이들 중 알맞은 생성자를 사용하면 된다. 첫 번째 생성자를 사용하면, enabled, accountNonExpired, credentailsNonExpired, accountNonLocked의 값으로 true를 사용한다.  

+ User(String username, String password, Collection&#60;? extends GrantedAuthority&#62; authorities)
+ User(String username, String password, boolean enabled, boolean accountNonExpired, boolean credentialsNonExpired, boolean accountNonLocked, Collection&#60;? extends GrantedAuthority&#62; authorities)  

커스텀 UserDetailsService를 구현했다면, &#60;authentication-provider&#62; 태그의 user-service-ref 속성 값으로 커스텀 UserDetailsService 빈 객체를 전달해서 스프링 시큐리티가 커스텀 구현을 사용하도록 설정해주면 된다. 예를 들어, 아래 코드처럼 설정하면, UserDetailsService로 CustomUserDetailsService 클래스를 사용한다.  

```xml
<bean id="빈식별자" class="커스텀 UserDetailsService를 구현한 완전한클래스명"/>

<sec:authentication-manager alias="authenticationManager">
	<sec:authentication-provider user-serivice-ref="빈식별자">
	</sec:authentication-provider>
</sec:authentication-manageer>
```

##### 5.3.5. 커스텀 AuthenticationProvider 구현하기
<br/>
사용자 정보 조회뿐만 아니라 암호 비교까지 직접 구현해야 한다면, AuthenticationProvider 인터페이스를 상속받아 구현해주어야 한다. 예를 들어, 사용자 아이디/암호가 일치하는지 여부는 LDAP을 이용해서 처리하고 각 사용자가 가진 권한은 DB에서 읽어와야 한다면, UserDetailsService 만으로는 인증 흐름을 제어할 수 없기 때문에, AuthenticationProvider를 직접 구현해야 한다.  

org.springframework.security.authentication.AuthenticationProvider 인터페이스는 다음의 두 메서드를 정의하고 있다.  

```java
public interface AuthenticationProvider {
	Authentication authenticate(Authentication authentication) throws AuthenticationException;
	boolean supports(Class<?> authentication);
}
```

supports() 메서드는 해당 타입의 Authentication 객체를 이용해서 인증 처리를 할 수 있는지 여부를 리턴한다.  

authenticate() 메서드를 구현하는 규칙은 다음과 같다.

+ 파라미터로 전달받은 authentication 객체에 대해 인증 처리를 지원하지 않는다면 null을 리턴한다.
+ authentication 객체를 이용한 인증에 실패했다면 AuthenticationException을 발생시킨다.
+ 인증에 성공하면, 인증 정보를 담은 Authentication 객체를 리턴한다.  

다음은 AuthenticationProvider의 구현 방식을 보여주고 있다.  

```java
public class CustomAuthenticationProvider implements AuthenticationProvider {
	@Override
	public boolean supports(Class<?> authentication) {
		return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
	}

	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		// 1. authentication을 지원하는 타입으로 변환한다.
		UsernamePasswordAuthenticationToken authToken = (UsernamePasswordAuthenticationToken) authentication;
		// 2. 사용자 정보를 조회한다. 존재하지 않으면 익셉션을 발생
		UserInfo userInfo = findUserById(authToken.getName());
		if (userInfo == null)
			throws new UsernameNotFoundException(authToken.getName());
		// 3. 조회한 사용자와 라라미터로 받은 authentication의 암호를 비교한다.
		// 암호가 일치하지 않으면 익셉션을 발생한다.
		if (!matchPassword(userInfo.getPassword(), authToken.getCredentials()))
			throw new BadCredentialsException("not matching username or password");

		// 4. 사용자가 가진 권한 목록을 구한다.
		List<GrantedAuthority> authorities = getAuthorities(userInfo.getId());
		// 5. 인증된 사용자에 대한 Authentication 객체를 생성해서 리턴한다.
		return new UsernamePasswordAuthenticationToken(new UserInfo(userInfo.getId(), userInfo.getName(), null), null, authorities);
	}

	private UserInfo findUserById(String id) {
		... // id에 해당하는 정보를 읽어와 리턴하는 코드
	}

	private boolean matchPassword(String password, Object credentials) {
		return password.equals(credentials);
	}

	private List<GrantedAuthority> getAuthorities(String id) {
		... // id가 갖고 있는 권한 정보 로딩 및 생성하는 코드
	}

	...
}
```

autheticate() 메서드는 이증에 성공할 경우 UsernamePasswordAuthenticationToken 객체를 생성해서 리턴한다. Authentication 인터페이스를 구현한 클래스를 직접 만들어도 되지만, 이 코드처럼 스프링 시큐리티가 제공하고 있는 구현 클래스를 사용해도 무방하다.  

UsernamePasswordAuthenticationToken 클래스 생성자는 다음곽 같다. 이 생성자를 사용하면 isAuthtenticated() 메서드가 true를 리턴한다. 즉, 인증된 상태의 Authentication 객체가 된다.  

+ UsernamePasswordAuthenticationToekn(Objejct principal, Object credentials, Collection&#60;? extends GrantedAuthority&#62; authorities)  

첫 번째 파라미터는 인증 대상 주체를 표현하는 객체를 사용한다. 이 인증 대상 객체로 UserDetails 타입이나 Principal 타입을 보통 전달하는데, 앞서 코드에서는 UserInfo 객체를 그대로 전달했다. 첫 번째 파라미터로 전달한 객체가 getPrincipal() 메서드의 리턴 값으로 사용된다.  

두 번째 파라미터는 인증 증명에 사용되는 크리덴셜(credential)인데, 보통 이 값은 메모리에 남지 않도록 null을 준다. (또는 객체 생성 후 eraseCredentials() 메서드를 호출해서 제거한다.  

세 번째 파라미터는 인증된 사용자가 갖는 권한 목록이다. getAuthorities()의 리턴 값으로 사용된다.  

커스텀 AuthenticationProvider 클래스를 만들었다면, &#60;authentication-provider&#62; 태그의 ref 속성 값에 커스텀 AuthenticationProvider 빈 객체를 전달해서 스프링 시큐리티가 커스텀 구현을 사용하도록 설정해주면 된다. 예를 들어, 아래 코드처럼 설정하면, AuthenticationProvider로 CustomAuthenticationProvider 클래스를 사용한다.  

```xml
<bean id="빈식별자" class="CustomAuthenticationProvider를 구현한 완전한클래스명"/>

<sec:authentication-manager alias="authenticationManager">
	<sec:authentication-provider ref="빈식별자" />
</seic:authentication-manager>
```

#### 5.4. 비밀번호 암호화 처리
<br/>
지금까지 살펴본 예제의 문제점은 비밀번호가 평문이라는 점이다. DB 테이블에 비밀번호가 평문으로 들어가 있으면, 보안 사고가 발생했을 때 사용자의 비밀번호가 바로 노출된다. 사고 발생시 문제를 최소화하려면 비밀번호를 암호화해서 쉽게 유추할 수 없게 만들어야 한다.  

앞서 DaoAuthenticationProvider는 UserDetailsService로부터 사용자 정보를 읽어와 인증을 처리한다고 했는데, DaoAuthenticationProvider 클래스는 암호화 처리 기능을 지원하고 있다. DaoAuthenticationProvider 클래스는 사용자가 입력한 암호와 UserDetailsService에서 구한 UserDetails의 암호가 일치하는 여부를 판단하기 위해 PasswordEncoder를 사용한다.  

별도 설정을 하지 않을 겨우 DaoAuthenticationProvider는 PasswordEncoder의 구현 클래스로 PlaintextPasswordEncoder를 사용하는데, 이 클래스는 암호화를 하지 않고 단수 문자열 비교를 통해 비밀번호 일치 여부를 비교한다.  

만약 비밀번호를 암호화하고 이를 기준으로 암호를 비교하고 싶다면, 암호화 기능을 제공하는 PasswordEncoder를 스프링 빈으로 등록하고, 이를 사용하도록 설정하면 된다. 다음은 설정 예를 보여주고 있다.  

```xml
<!-- 스프링 시큐리티가 기본 제공하는 PasswordEncoder 구현 -->
<bean id="passwordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" />

<sec:authentication-manager alias="authenticationManager">
	<sec:authentication-provider>
		<sec:jdbc-user-service data-source-ref="dataSource" />
		<!-- 암호를 비교할 때 사용할 PasswordEncoder 지정 -->
		<sec:password-encoder ref="passwordEncoder" />
	</sec:authentication-provider>
</sec:authentication-manager>
```

DaoAuthenticationProvider는 &#60;password-encoder&#62; 태그로 지정한 PasswordEncoder를 이용해서 사용자가 입력한 비밀번호와 DB에 보관된 암호화된 비밀번호가 일치하는지 여부를 판단한다.  

여기서 유의할 점은 DB에 넣을 비밀번호를 암호화 할 때에도 같은 PasswordEncoder를 사용해야 한다는 점이다.  

DB에ㅐ 데이터를 넣을 때 사용하는 PasswordEncoder와 인증 과정에서 비밀번호를 비교할 때 사용하는 PasswordEncoder과 다르면, 정상적인 비교가 되지 않으므로 같은 PasswordEncoder를 사용해야 한다.  

##### 5.4.1. PasswordEncoder 인터페이스와 구현 클래스
<br/>
org.springframework.security.crpto.password.PasswordEncoder 인터페이스는 다음의 두 메서드를 정의하고 있다.  

```java
public interface PasswordEncoder {
	String encode(CharSequence rawPassword);
	boolean matches(CharSequence rawPassword, String encodedPassword);
}
```

encode() 메서드는 rawPassword를 암호화한 결과를 리턴한다. 평문을 암호화해서 DB에 넣어야 할 때 encode() 메서드를 사용하면 된다.  

matches() 메서드는 rawPassword와 암호화된 물자열인 encodedPassword와 일치하는지 여부를 검사한다. 일치하면 true를 리턴하고, 일치하지 않으면 false를 리턴한다.  

스프링 시큐리티가 제공하는 PasswordEncoder 구현 클래스로는 다음의 두 가지가 있다.  

+ org,springframework.security.crypto.bcrypt.BCryptPasswordEncoder : BCrypt를 이용해서 암호화된다.
+ org.springframework.security.crypto.password.StandardPasswordEncoder : SHA-256을 이용해서 암호화한다.  

진행중인 프로젝트에서 사용중인 암호화 알고리즘이 없다면, 두 PasswordEncoder 중 하나를 사용하면 된다.  

#### 5.5. 로그인/로그아웃 UI/에러 응답 설정 변경
<br/>
##### 5.5.1. 로그인 폼과 인증 요청 경로 변경
<br/>
스프링 시큐리티가 제공하는 로그인 폼이 있지만, 이 폼을 그래도 사용하고 싶지는 않을 것이다. 로그인 폼과 로그인을 처리하는 URL도 바꾸고 싶을 것이다. 이를 위한 설정은 매우 간단하다. &#60;form-login&#62; 설정 태그의 속성을 변경하고 로그인 폼 화면을 만들어주면 된다. 참고로, 이 속성들에 지정한 경로 값은 웹 어플리케이션의 컨텍스트 경로를 기준으로 한다.  


<table>
	<thead>
		<tr>
			<td>속성</td>
			<td>설명</td>
			<td>기본값</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>login-page</td>
			<td>인증 전에 접근한 경로에 접근 권항이 없을 경우, 인증 처리를 위해 이동할 로그인 폼 경로를 지정한다.</td>
			<td>/spring&#95;security&#95;login</td>
		</tr>
		<tr>
			<td>login-processing-url</td>
			<td>로그인 요청을 위한 경로를 지정한다. 이 경로로 POST 요청이 들어오면 인증을 처리한다.</td>
			<td>/j&#95;spring&#95;security&#95;check</td>
		</tr>
		<tr>
			<td>username-parameter</td>
			<td>로그인 요청에서 사용자 이름(로그인ID)에 해당하는 요청 파파미터의 이름을 지정한다</td>
			<td>j&#95;username</td>
		</tr>
		<tr>
			<td>password-parameter</td>
			<td>로그인 요청에서 사용자 비밀번호에 해당하는 요청 파라미터의 이름을 지정한다.</td>
			<td>j&#95;password</td>
		</tr>
		<tr>
			<td>default-target-url</td>
			<td>로그인 성공시 이동할 기본 경로를 지정한다. 만약 인증 전에 접근한 경로에 권한이 없어서 로그인 화면으로 이동했다면 로그인 성공 후 원래 요청한 경로로 이동한다.</td>
			<td>/</td>
		</tr>
		<tr>
			<td>authentication-failure-url</td>
			<Td>인증 실패시 이동할 경로를 지정한다</td>
			<td>/spring&#95;security&#95;login?login&#95;error</td>
		</tr>
	</tbody>
</table>
<br/><br/>

다음은 &#60;form-login&#62; 태그의 속성을 이용해서 로그인 폼 경로를 비롯한 관련 속성을 설정한 예제 코드이다.  

```xml
<sec:http use-expression="true" entry-point-ref="빈식별자1">
	<sec:intercept-url pattern="/user/loginform" access="permitAll" />
	<sec:intercept-url pattern="/member/**" access="isAuthenticated()" />
	...
	<sec:form-login login-page="/user/loginform" login-processing-url="/user/login" username-parameter="userid" password-parameter="password" default-target-url="/index" authentication-failure-url="/user/loginform?error=true" authentication-success-hanlder-ref="빈식별자2" authentication-failure-handler-ref="빈식별자3"/>
	<sec:logout />
</sec:http>

<bean id="빈식별자1" class="AuthenticationEntryPoint를 구현한 완전한클래스명"/>
<bean id="빈식별자2" class="AuthenticationSuccessHandler를 구현한 완전한 클래스명"/>
<bean id="빈식별자3" class="AuthenticationFailureHandler를 구현한 완전한 클래스명"/>
```

위 설정을 사용하면 아직 인증을 하지 않은 상태에서 인증을 필요로 하는 경로인 /member/main에 접근하면 로그인 폼을 보여주기 위해 /user/loginform 경로로 리다이렉트한다. 따라서, /user/loginform 경로로 접근시 로그인 폼을 보여줘야 한다. 로그인 폼을 보여주는 경로는 누구나 접근할 수 있어야 한므로 &#60;intercept-url&#62; 태그를 이용해서 /user/loginform 경로의 권한을 permitAll로 설정한다.  

스프링 MVC는 login-page 속성에 지정한 경로로 요청이 들어오면 로그인 폼을 보여주는 뷰를 사용하도록 설정한다. 별도 컨트롤러를 구현할 필요가 없다면 뷰 컨트롤러를 설정해주면 될 것이다.  

로그인 폼을 보여주는 뷰 코드는 login-processing-url 속성에 지정한 경로를 &#60;form&#62; 택그의 action 경로로 사용하고, username-parameter 속성과 password-parameter 속성에서 지정한 이름을 각각 아이디와 비밀번호를 파라미터 이름으로 사용한다.  

##### 5.5.2. 로그인 처리 관련 필터의 동작 방식
<br/>
아직 인증을 거치지 않은 사용자가 특정 권한을 필요로 하는 경로에 접근하면, 로그인 폼을 보여주는 경로로 이동하는데, 이 과정은 어떻게 처리되는 것일까? 이 과정에는 네 개의 구성 요소가 관여하며, 이들은 다음과 같이 동작한다.  

인증되지 않은 사용자가 특정 권한이 필요한 경로에 접근하면 FilterSecurityInterceptor는 Authentication을 발생시킨다. 이 익셉션이 발생하면 보안 필터 체인에서 FilterSecurityInterceptor 이전에 위치한 ExceptionTranslationFilter가 해당 익셉션을 받는다.  

ExceptionTranslationFilter는 아직 사용자가 인증 전이면 4번 과정처럼 요청 정보를 RequestCache에 보관하고, 5번 과정처럼 등록된 AuthenticationEntryPoint의 commence() 메서드를 호출해서 인증 처리 시작을 요청한다.  

AuthenticationEntryPoint 클래스는 LoginUrlAuthenticationEntryPoint인데, 이 클래스는 로그인 포그인 폼으로 지정한 경로로 리다이렉트 함으로써 사용자가 인증을 할 수 있도록 한다. 이때 이 로그인 폼으로 사용할 경로가 &#60;form-login&#62; 태그의 login-page 속성으로 지정한 값이다.  

로그인 폼에서 &#60;form-login&#62; 태그의 login-processing-url 속성에 지정한 경로로 로그인 요청을 전송하면 다음과 같은 과정을 거쳐 인증을 처리한다.  

3번 과정에서 인증에 성공하면 4번~8번 과정을 실행하고, 인증에 실패하면 9~10번 과정을 실행한다.  

(1) UsernamePasswordAuthenticationFilter는 &#60;form-login&#62; 태그의 login-processing-url 속성에 지정한 경로로 요청이 들어오면 인증 처리를 시작한다. UsernamePasswordAuthenticationFilter는 &#60;form-login&#62; 태그의 username-parameter 속성과 password-parameter 속성으로 지정한 파라미터로부터 인증에 사용할 사용자 이름과 비밀번호 값을 가져온다.  
(2) AuthenticationManager 클래스의 authenticate 메서드로 인증 요청을 수행한다.  
(3) AuthenticationManager는 인증에 성공하면 Authentication 객체를 리턴한고 싶패하면 익셉션을 발생한다.
(4) 3번 과정에서 리턴받은 Authentication 객체를 SecurityContext에 설정한다.  
(5) AuthenticationSuccessHandler 객체에 인증 성공 후처리를 요청한다. SavedRequestAwareAuthenticationSuccessHandler를 기본 구현으로 사용한다.  
(6) RequestCache 객체에서 요청 정보를 구한다.  
(7) RequestCache 객체는 보관된 요청 정보가 있으면 리턴하고, 없으면 null을 리턴한다. 인증되지 않은 사용자가 인증이 필요한 자원에 접근하면, 로그인 후 돌아갈 페이지 정보가 RequestCache에 보관된다.  
(8) RequestCache 객체가 요청 정보를 리턴하면, 해당 요청 정보의 경로로 리다이렉트한다. 이 정보가 null이면, &#60;form-login&#62; 태그의 default-target-url 속성에 지정한 경로로 리다이렉트 처리를 요청한다.  
(9) AuthenticationFailureHandler에 인증 실패 후처리(onAuthenticationFailure 메서드)를 요청한다. 기본 구현체로는 SimpleUrlAuthenticationFailureHandler 클래스를 사용한다.  
(10) &#60;form-login&#62; 태그의 authentication-failure-url 속성에 지정한 경로로 리다이렉트 처리를 요청한다.  

인증에 성공하거나 실패했을 때 기본 사용되는 AuthenticationSuccessHandler 객체나 AuthenticationFailureHandler 객체 대신 다른 구현을 사용하고 싶다면, &#60;form-login&#62; 태그의 authentication-success-handler-ref 속성과 authentication-failure-handler-ref 속성을 사용해서 사용할 빈 객체를 지정한다. 이 두 속성을 사용하면 기본 구현을 사용하지 않기 때문에 default-target-url 속성이나 authentication-failure-url이 적용되지 않는다.  

또한, &#60;http&#62; 태그의 entry-point-ref 속성을 사용하면 AuthenticationEntryPoint로 사용할 빈 객체를 직접 지정하기 때문에 &#60;form-login&#62; 태그의 login-page 속성이 적용되지 않는다.  

##### 5.5.3. 로그아웃 URL 및 로그아웃 후 이동 경로
<br/>
스프링 시큐리티 설정에서 &#60;logout&#62; 태그를 사용하면, /j&#95;spring&#95;security&#95;logout 경로 요청에 해해 로그아웃 처리를 한다. 로그아웃 처리는 LogoutFilter를 통해서 이루어지는데, 로그아웃 경로를 변경하고 싶다면 다음과 같이 &#60;logout&#62; 태그의 logout-url 속성을 사용하면 된다.  

```xml
<sec:logout logout-url="/user/logout"/>
```

LogoutFilter는 logout-url 속성에 지정한 경로로 요청이 들어오면 로그아웃 처리를 수행하고, logout-success-handler-url 속성에 지정한 경로로 리다이렉트한다. 이 속성을 지정하지 않으면 기본 값은 "/"로 컨텍스트 경로 내에서의 루트로 리다이렉트한다.  

&#60;logout&#62; 태그의 주요 속성은 다음과 같다.  

<table>
	<thead>
		<tr>
			<td>속성</td>
			<td>설명</td>
			<td>기본값</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>logout-url</td>
			<td>로그아웃 요청 경로를 지정한다.</td>
			<td>/j&#95;spring&#95;security&#95;logout</td>
		</tr>
		<tr>
			<td>invalidate-session</td>
			<td>기존 세션을 제거할지 여부를 지정한다.</td>
			<td></td>
		</tr>
		<tr>
			<td>delete-cookies</td>
			<td>로그아웃 시, 삭제할 쿠키 목록을 저장한다. 각 쿠키 이름을 콤마로 구분한다.</td>
			<td></td>
		</tr>
		<tr>
			<td>logout-success-url</td>
			<td>로그아웃 이후, 리다이렉트할 경로를 지정한다.</td>
			<td></td>
		</tr>
		<tr>
			<td>success-handler-url</td>
			<td>로그아웃 성공 후, 이동 처리를 하는 LogoutSuccessHandler 빈을 직접 지정한다.</td>
			<td></td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 5.5.4. 권한 없음 응답 화면 설정
<br/>
인증된 사용자가 권한이 엇는 경로에 접근할 경우, 스프링 시큐리티는 403 상태 코드를 웹 브라우저에 응답한다. 만약 403 응답 코드가 아니라 별도의 안내 페이지를 작성하고 싶다면, 다음과 같이 &#60;access-denied-handler&#62; 태그의 error-page 속성을 사용해서 에러 페이지 경로를 지정해주면 된다.  

```xml
<sec:http use-expresssions="true">
	<sec:access-denied-handler error-page="/security/accessDenied"/>
	...
</sec:http>
```

AccessDeniedHandler의 기본 구현체인 AccessDeniedHandlerImpl 클래스는 error-page를 지정한 경우 해당 경로로 포워드를 한다. 리다이렉트가 아니라 포워드이므로 웹 브라우저의 주소는 그대로 유지된다.  

AccessDeniedHandler로 사용할 빈을 직접 지정하고 싶다면 ref 속성을 사용해서 지정하면 된다.  

#### 5.6. HTTPS 및 포트 매핑 설정하기
<br/>
로그인 폼의 데이터를 전송할 때에는 데이터의 안전한 전송을 위해 HTTPS 프로토콜을 사용하게 되는데, 스프링 시큐리티는 경로별로 HTTPS 프로토콜을 사용하도록 강제할 수 있다. &#60;intercept-url&#62; 태그의 requires-channel 속성을 사용하면 해당 경로에 접근할 때 어떤 프로토콜로 접근 가능한지를 제한할 수 있다. 예를 들어, 아래 코드는 /user/loginform 경로나 /user/login 경로는 HTTPS 프로토콜로만 접근할 수 있도록 제한하고 있다.  

```xml
<sec:http use-expressions="true">
	<sec:port-mappings>
		<sec:port-mapping https="8443"/>
	</sec:port-mapping>
	<sec:intercept-url pattern="/user/loginform" access="permitAll" requires-channel="https"/>
	<sec:intercept-url pattern="/user/login" access="permitAll" requires-channel="https"/>
	...
</sec:http>
```

HTTPS로 제한한 경로에 HTTP로 연결하면, 프로토콜만 HTTPS로 바꿔 같은 경로로 리다이렉트시킨다.  

requires-channel 속성이 가질 수 있는 값은 "http", "https", "any"이다.  

리다이렉트 할 때 사용할 HTTP와 HTTPS의 포트가 다를 경우 &#60;port-mapping&#62; 태그를 이용해서 포트 번호를 지정할 수 있다. 다음 설정의 경우 HTTPS로 리다이렉트할 때 8443 포트를 사용해서 리다이렉트 경로를 생성한다. HTTP 포트를 지정하고 싶으면 http 속성을 사용하면 된다.  

#### 5.7. 세션 대신 쿠키 사용하기
<br/>
대규모 트래픽이 발생하는 서비스는 HttpSession 대신 쿠키를 이용해서 인증 상태를 유지하거나 별도 외부 서버에 세션 정보를 보관하기도 한다. 이런 서비스는 수십에서 수백대에 이르는 웹 서버를 사용하고 사용하고 사용자가 어떤 서버에 접속할지 알 수 없기 때문에 로그인 성공 후 이동할 경로를 HttpSession에 보관하는 방법을 사용할 수 없다. 대신, 요청 파라미터에 이동할 경로 정보를 담는 방법을 주로 사용한다.  

스프링 시큐리티는 기본적으로 사용자 인증 정보를 HttpSession에 보관하고 로그인 성공시 이동할 경로를 보관하기 위해서도 HttpSession을 사용한다. 따라서, 인증 정보를 쿠키나 외부 저장소에 보관한다거나 로그인 성공 후 이동 경로를 파라미터로 전달하고 싶다면 스프링 시큐리티의 기본 기능 대신 커스텀 구현을 일부 사용해야 한다.  

##### 5.7.1. 로그인 성공 후 이동 경로를 파라미터로 지정하기
<br/>
앞서 '로그인 처리 관련 필터의 동작 방식' 절에서 두 내용을 살펴봤었다.  

+ 아직 인증 전인 사용자가 인증이 필요한 경로에 접근할 경우 ExceptionTranslationFilter는 HttpSessionRequestCache에 로그인 성공시 이동할 경로를 저장하고, AuthenticationEntryPoint를 이용해서 로그인 폼으로 리다이렉트 처리  
+ 로그인 폼에서 로그인을 시도할 경우, 인증에 성공하면, SavedRequestAwareAuthenticationSuccessHandler가 HttpSessionRequestCache에서 이동할 경로를 구해서 리다이렉트 처리. 인증에 실패하면, SimpleUrlAuthenticationFailureHandler가 실패시 경로로 리다이렉트 처리  

로그인에 성공할 때 이동할 경로를 HttpSession이 아닌 요청 파라미터로 전달하고 싶다면 위 부분을 알맞게 커스터마이징하면 된다. 커스텀 구현을 만드는 다양한 방법이 존재할 수 있는데, 다음과 같이 구현할 것이다.  

+ 커스텀 AuthenticationEntryPoint 구현 : 로그인 폼 경로로 리다이렉트할 때, 현재 요청 경로를 return 파라미터로 붙여서 보내는 커스텀 구현
+ 커스텀 AuthenticationSuccessHandler 구현 : 로그인 성공시, returl 파라미터의 값이 존재하면 해당 경로로 리다이렉트하는 커스텀 구현
+ 커스텀 AuthenticationFailureHandler 구현 : 로그인 실패시, 로그인 경로로 다시 포워딩한다.  
+ 로그인 폼에서 returl 파라미터를 전송하도록 폼 구성
+ NullRequestCache 사용 : 아무 기능도 하지 않는 RequestCache 사용  

먼저, 로그인 폼으로 리다이렉트 할 때, 현재 경로를 return 파라미터로 붙여서 보내주는 커스텀 AuthenticationEntryPoint 클래스는 다음과 같이 구현할 수 있다.  

```java
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {
	
	private String loginFormPath;

	@Override
	public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
		String redirectUrl = UrlUtils.buildFullRequestUrl(request);
		String encoded = response.encodeRedirectURL(redirectUrl);
		response.sendRedirect(request.getContextPath() + loginFormPath + "?returl=" + encoded);
	}

	public void setLoginFormPath(String loginPage) {
		this.loginFormPath = loginPage;
	}

}
```

UrlUtils 클래스는 스프링 시큐리티가 제공하는 클래스로, 이 클래스를 이용해서 현재 경로의 URL을 구성한다. 현재 요청 경로의 URL을 구한 뒤에는 로그인 폼 경로로 리다이렉트하는데, 리다이렉트할 경로를 returl 파라미터 값으로 지정한다.  

인증 성공 후처리를 할 커스텀 AuthenticationSuccessHandler는 returl 파라미터로 전달받은 경로롤 이동하도록 다음과 같이 구현한다.  

```java
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
	
	@Override
	public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
		String returnUrl = request.getParameter("returl");
		if (retUrl == null || retUrl.isEmpty()) {
			response.sendRedirect(request.getContextPath());
			return;
		}
		response.sendRedirect(retUrl);
	}

}
```

retUrl 값이 없을 경우에는 컨텍스트 루트로 이동하도록 구현한다.  

인증 실패 후처리를 하는 커스텀 AuthenticationFailureHandler는 원래 로그인 폼으로 다시 돌아가면 된다. 이때 리다이렉트 대신 포워드를 사용하면 returl 파라미터 값을 주소 뒤에 붙여주지 않아도 전달받은 returl 파라미터의 값을 사용할 수 있다. 커스텀 구현은 다음과 같다.  

```java
public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {

	private String loginFormPath;

	@Ovrride
	public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
		request.getRequestDispatcher(loginFormPath).forward(request, response);
	}

	public void setLoginFormPath(String loginFormPath) {
		this.loginFormPath = loginFormPath;
	}

}
```

스프링 시큐리티가 커스텀 구현을 사용하도록 설정해주면 된다. 다음은 설정 예를 보여주고 있다.  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:sec="http://www.springframework.org/schema/security"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
					http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">

	<bean id="빈식별자1" class="AuthenticationEntryPoint를 구현한 완전한클래스명"/>

	<bean id="빈식별자2" class="AuthenticationSuccessHandler를 구현한 완전한 클래스명">
		<property name="loginFormPath" value="/user/loginform" />
	</bean>

	<bean id="빈식별자3" class="AuthenticationFailureHandler를 구현한 완전한 클래스명">
		<property name="loginFormPath" value="/user/loginform" />
	</bean>

	<bean id="빈식별자4" class="SecurityContextRepository를 구현한 완전한 클래스명" />

	<bean id="nullRequestCache" class="org.springframework.security.web.savedrequest.NullRequestCache" />

	<sec:http use-expressions="true" entry-point-ref="customAuthenticationEntryPoint" security-context-repository-ref="빈식별자4">
		<sec:request-cache ref="nullRequestCache" />
		<sec:intercept-url pattern="/member/**" access="isAuthenticated()" />
		<sec:intercept-url pattern="/**" access="permitAll" />

		<sec:form-login authentication-success-handler-ref="customSuccessHandler" authentication-failure-handler-ref="customFailureHandler" username-parameter="userid" password-parameter="password" login-processing-url="user/login" />

		<sec:logout logout-url="/user/logout" />
	</sec:http>

	<sec:authentication-manager>
		...
	</sec:authentication-manager>
</beans>
```

&#60;request-cache&#62;의 ref 속성으로 NullRequestCache 빈 객체를 설정하면 인증이 필요해서 로그인 폼으로 이동할 때 세션에 현재 요청 경로 정보를 보관하지 않는다.  

##### 5.7.2. 인증 상태를 쿠키에 보관하기
<br/>
대형 포털은 서비스별로 작게는 수십에서 많게는 수백 대에 이르는 웹 서버를 사용하는데, 이 경우 인증 상태를 보관하기 위해 HttpSession을 사용하긴 힘들기 때문에, 대신 쿠키를 이용해서 인증 상태를 공유하는 방법을 주로 사용한다.  

쿠키를 이용해서 인증 상태를 유지하려면 다음의 작업을 처리해야 한다.  

+ 로그인 성공 시점에는 AuthenticationSuccessHandler에서 인증 상태 유지를 위한 쿠키를 생성
+ SecurityContextPersistenceFilter가 사용하는 SecurityContextRespostiory 커스텀 구현(loadContext() 메서드가 인증 쿠키로부터 Authentication 객체 생성, saveContext() 메서드는 아무 동작도 하지 않음)
+ 로그아웃 성공시 쿠키 삭제  

보안 필터 체인의 앞쪽에 위치한 SecurityContextPersistenceFilter는 SecurityContextRepository를 이용해서 SecurityContext를 가져오거나 저장한다. 기본 구현 클래스로 사용되는 HttpSessionSecurityContextRepository는 HttpSession을 이용해서 SecurityContext를 보관하기 때문에, SecurityContextRepository의 커스텀 구현을 이용해서 HttpSession 대신 쿠키에서 인증 정보를 읽어오도록 하면 된다.  

먼저 로그인 성공시 쿠키를 생성하도록 AuthenticationSuccessHandler를 구현한다. 앞서 만들었었던 코드에 생성하는 기능만 추가하면 된다. 구현 코드는 다음과 같다.  

```java
public class CustomBasedCookieAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

	@Override
	public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
		addAuthCookie(response, authentication);
		String retUrl = request.getParameter("retUrl");
		if (retUrl == null || retUrl.isEmpty()) {
			response.sendRedirect(request.getContextPath() + "/");
			return;
		}
		response.sendRedirect(retUrl);
	}

	private void addAuthCookie(HttpServletResponse response, Authentication authentication) {
		UserDetails user = (UserDetails) authentication.getPrincipal();
		String cookieValue = user.getUsername();
		if (authentication.getAuthorities() != null) {
			for (GrantedAuthority auth : authentication.getAuthorities())
				cookieValue += "," + auth.getAuthority();
		}
		try {
			// 실제로는 쿠키값을 암호화해서 저장한다.
			Cookie cookie = new Cookie("AUTH", URLEncoder.encode(cookieValue, "utf-8"));
			cookie.setPath("/");
			response.addCookie(cookie);
		} catch (UnsupportedEncodingException e) {
			throw new RuntimeException(e);
		}
	}

}
```

실제 운영 환경에서는 암호화해서 쿠키 변조가 발생하지 않도록 해야 할 것이다.  

다음으로 구현할 코드는 쿠키에서 Authentication 정보를 읽어오는 SecurityContextRepository를 구현하는 것이다. 이 코드는 인증 성공 시점에 생성한 쿠키 값을 이용해서 Authentication 객체를 생성하면 된다. 구현 코드는 다음과 같다.  

```java
public class CustomSecurityContextRepository implements SecurityContextRepository {

	@Override
	public SecurityContext loadContext(HttpRequestResponseHolder reqResHolder) {
		SecurityContext ctx = SecurityContextHolder.createEmptyContext();
		String authValue = getAuthCookieValue(reqResHolder.getRequest());
		if (authValue != null) {
			String[] values = authValue.split(",");
			String username = values[0];
			List<GrantedAuthority> authorities = new ArrayList<>();
			for (int i = 1; i < values.length; i++) {
				authorities.add(new SimpleGrantedAuthority(values[i]));
			}
			User user = new User(userName, "", authorities);
			Authentication authentication = new UsernamePasswordAuthenticationToken(user, "", authorities);
			ctx.setAuthentication(authentication);
		}
		return ctx;
	}

	private String getAuthCookieValue(HttpServletRequest request) {
		Cookie[] cookies = request.getCookies();
		if (cookies == null)
			return null;
		for (Cookie cookie : cookies) {
			if (cookie.getName().equals("AUTH")) {
				try {
					return URLDecoder.decode(cookie.getValue(), "utf-8");
				} catch (UnsupportedEncodingException e) {
					return null;
				}
			}
		}
		return null;
	}

	@Override
	public void saveContext(SecurityContext, HttpServletRequest request, HttpServletResponse response) {
	}

	@Override
	public boolean containsContext(HttpServletRequest request) {
		return false;
	}

}
```

CustomSecurityContextRepository 클래스의 loadContext() 메서드는 인증 성공시 생성한 쿠키카 존재하면, 해당 쿠키로부터 Authentication 객체를 생성한다. 이때, Authentication의 principal 객체로는 User(즉, UserDetails 타입) 객체를 사용했다. Authentication 객체를 생성한 뒤에는 SecurityContext 객체에 Authentication을 보관한다.  

마지막으로 로그아웃 성공시 쿠키를 삭제하기 위해 LogoutSuccessHandler의 커스텀 구현을 만든다. 구현 코드는 다음과 같다.  

```java
public class CustomLogoutSuccessHandler implements LogoutSuccessHolder {

	@Override
	public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
		Cookie cookie = new Cookie("AUTH", "");
		cookie.setPath("/");
		cookie.setMaxAge(0);
		response.addCookie(cookie);
		response.sendRedirect(request.getContextPath() + "/");
	}
}
```

onLogoutSuccess()는 LogoutFilter가 로그아웃 처리 후 호출하는데, CustomLogoutSuccessHandler는 이 메서드에서 쿠키 삭제 처리를 하고 컨텍스트 루트 경로로 리다이렉트하도록 구현했다.  

&#60;logout&#62; 태그의 delete-cookie 속성을 사용해도 쿠키를 삭제할 수 있는데, 이 속성에 지정한 쿠키는 컨텍스트 경로를 기준으로 한다. 예를 들어, 컨텍스트 경로가 "/spring-chap16"인 경우 경로가 "/"로 만들어진 쿠키는 삭제되지 않는다. 이런 이유로 LogoutSuccessHandler의 커스텀 구현을 만들었다.  

&#60;http&#62; 태그의 security-context-repository-ref 속성은 SecurityContextRepository로 사용할 빈을 전달받는다. SecurityContextPersistenceFilter는 이 빈을 이용해서 구한 SecurityContext 객체를 SecurityContextHolder.setContext()에 설정한다. 따라서, 스프링 시큐리티는 커스텀 SecurityContextRepository에서 생성한 SecurityContext의 정보를 사용해서 권한 접근 검사 등을 수행하게 된다.  

&#60;logout&#62; 태그는 success-handler-ref 속성을 사용해서 LogoutSuccessHandler로 커스텀 구현을 사용하도록 설정한다.  

### 6. JSP 태그 라이브러리
<br/>
스프링 시큐리티는 JSP에서 사용할 수 있는 커스텀 태그를 제공하고 있다. 이들 태그를 사용하면 사용자가 가진 권한에 따라 메뉴를 보여주거나 현재 접속한 사용자의 정보를 보여줄 수 있다.  

JSP에서 스프링 시큐리티가 제공하는 커스텀 태그를 사용하려면 JSP 코드에 다음과 같이 taglib 디렉티브를 추가한다.  

```xml
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
```

태그 라이브러리가 제공하는 커스텀 태그에서 가장 많이 사용되는 태그는 &#60;sec:authorize&#62; 태그다. &#60;sec:authorize&#62; 커스텀 태그는 access 속성에 지정한 권한 값에 따라 태그의 몸체 내용을 보여준다. 예를 들어, 인증된 경우에만 로그아웃 링크를 보여주고 싶다면, 다음과 같이 &#60;sec:authorize&#62; 태그를 사용하면 된다.  

access 속성에는 hasRole(), hasAuthority(), isAuthenticated() 등의 표현식이 올 수 있다.  

특정 경로에 접근 가진 경우만 몸체 내용을 처리하고 싶다면 url 속성을 사용하면 된다.  

&#60;sec:authentication&#62; 태그는 SecurityContext에 보관된 Authentication 객체의 값을 출력할 때 사용된다. 예를 들어, Authentication 객체의 getPrincipal() 메서드가 리턴하는 객체 타입이 UserDetails라면, 다음의 코드를 이용해서 ISerDetails의 값을 출력할 수 있다.  

```jsp
<sec:authentication preperty="principal.username" />
```
