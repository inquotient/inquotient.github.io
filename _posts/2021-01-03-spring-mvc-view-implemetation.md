---
title: 스프링 MVC - 뷰 구현
categories:
- Spring
feature_text: |
  ## 스프링 MVC : 뷰 구현
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	td { border: 1px solid #444444; }
</style>
컨트롤러는 최종적으로 결과를 출력할 뷰와 뷰에 전달할 모델을 담고 있는 ModelAndView 객체를 리턴한다. DispatcherServlet은 ViewResolver를 사용하여 결과를 출력할 View 객체를 구하고, 구한 View 객체를 이용해서 내용을 생성한다.  

컨트롤러 구현과 쌍을 이루는 것이 뷰 구현이다.  

### 1. ViewResolver 설정
<br/>
스프링 컨트롤러는 뷰에 의존적이지 않다. 컨토롤러는 결과를 생성할 뷰 이름만 지정할 뿐이다.  

컨트롤러가 지정한 뷰 이름으로부터 응답 결과 화면을 생성하는 View 객체를 구할 때 사용되는 것이 ViewResolver이다. 스프링은 몇 가지 ViewResolver 구현 클래스를 제공하고 있는데, 주요 ViewResolver 구현 클래스는 다음과 같다.  

<table>
	<thead>
		<tr>
			<td>ViewResolver 구현 클래스</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>InternalResourceViewResolver</td>
			<td>뷰 이름으로부터 JSP나 Tiles 연동을 위한 View 객체를 리턴한다.</td>
		</tr>
		<tr>
			<td>VeolcityViewResolver</td>
			<td>뷰 이름으로부터 Velocity 연동을 위한 View 객체를 리턴한다.</td>
		</tr>
		<tr>
			<td>VelocityLayoutViewResolver</td>
			<td>VelocityViewResolver와 동일한 기능을 제공하며, 추가로 Velocity의 레이아웃 기능을 제공한다.</td>
		</tr>
		<tr>
			<td>BeanNameViewResolver</td>
			<td>뷰 이름과 동일한 이름을 갖는 빈 객체를 View 객체로 사용한다.</td>
		</tr>
	</tbody>
</table>
<br/>

#### 1.1. ViewResolver 인터페이스
<br/>
ViewResolver는 뷰 이름과 지역화를 위한 Locale을 파라미터로 전달받으며, 매핑되는 View 객체를 리턴한다. 만약, 매핑되는 View 객체가 존재하지 않으면 null을 리턴한다.  
#### 1.2. View 객체
<br/>
ViewResolver가 리턴하는 뷰 객체는 응답 결과를 생성하는 역활을 한다. 모든 뷰 클래스는 View 인터페이스를 구현하고 있다. View 인터페이스는 다음과 같은 메서드를 정의하고 있다.
+ String getContentType()
+ void render(Map&#60;String, ?&#62; model, HttpServletRequest request, HttpServletResponse response) throws Exception  

getContentType() 메서드는 "text/html"과 같은 응답 결과의 컨텐트 타입을 리턴한다.  
render() 메서드는 실제로 응답 결과를 생성한다. render() 메서드의 첫 번째 파라미터인 model에는 컨트롤러가 생성한 모델 데이터가 전달된다. 각각의 View 객체는 이 모델 데이터로부터 응답 결과를 생성하는 데 필요한 정보를 구한다.  

#### 1.3. InternalResourceViewResolver 설정
<br/>
InternalResourceViewResolver 클래스는 InternalResourceView 타입의 뷰 객체를 리턴하는데, 이 뷰는 JSP나 HTML 파일과 같이 웹 어플리케이션의 내부 자원을 이용해서 응답 결과를 생성한다. JSTL이 존재할 경우 InternalResourceView의 하위 타입은 JstlView 객체를 리턴한다. (JstlView 클래스는 스프링의 지역화 관련 설정이 JSTL 커스텀 태그에 적용되도록 해준다.)  

InternalResourceViewResolver 클래스의 설정 방법은 다음과 같다.  
```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/view/" />
	<property name="suffix" value=".jsp" />
</bean>
```
InternalResourceViewResolver는 컨트롤러가 지정한 뷰 이름으로부터 실제로 사용될 뷰를 선택하며, 이때 컨트롤러가 지정한 뷰 이름 앞뒤로 prefix 프로퍼티와 suffix 프로퍼티를 추가한 값이 실제로 사용될 자원의 경로가 된다.  

예를 들어, 컨트롤러에서 다음과 같이 뷰 이름을 "hello"로 지정했다고 하자.  
```java
ModelAndView mav = new ModelAndView("hello");
return mav;
```
이 경우 InternalResourceViewResolver가 사용하는 자원의 경로는 다음과 같이 결정된다.  

```
/WEB-INF/view/hello.jsp
```

#### 1.4. BeanNameViewResolver 설정
<br/>
BeanNameViewResolver 클래스는 뷰 이름과 동일한 이름을 갖는 빈을 뷰로 사용한다. BeanNameViewResolver는 주로 커스텀 View 클래스를 뷰로 사용해야 할 때 이용된다.
```java
@Controller
public class 클래스명 {

	@RequestMapping("경로명")
	public ModelAndView 메소드명(HttpServletRequest request, HttpServletResponse response) {
		File downloadFile = getFile(request);
		return new ModelAndView("download", "downloadFile", downloadFile);
	}

	...

}
```
위 결과를 보여줄 View 클래스가 DownloadView 클래스라고 하자. 이 경우 DownloadView 클래스를 "download" 이름으로 빈에 등록하고 ViewResolver로 BeanNameViewResolver 클래스를 사용하면 된다.

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.BeanNameViewResolver" />

<bean id="download" class="...DownloadView" />
```
위 코드와 같이 설정하면 BeanNameViewResolver는 이름이 "download"인 DownloadView 객체를 뷰로 검색하며, DispatcherServlet은 DownloadView를 이용해서 DownloadController의 처리 결과를 출력하게 된다.  

#### 1.5. 다수의 ViewResolver 설정하기
<br/>
하나의 DispatcherServlet은 두 개 이상의 ViewResolver를 가질 수 있다. 다수의 ViewResolver 중에서 어느 ViewResolver를 먼저 사용하맂 여부는 순서값을 이용해서 결정하며, 적용 순서 값은 다음의 두 가지 중 한 방법으로 설정할 수 있다.  

+ ViewResolver 구현 클래스가 org.springframework.core.Oredered 인터페이스를 구현했다면, order 프로퍼티에 우선순위 값을 설정
+ ViewResolver 구현 클래스에 &#64;Order 애노테이션이 있다면, &#64;Order 애노테이션의 값을 우선순위 값으로 사용  

우선순위 값이 작을수록 우선순위가 높으며, 우선순위를 지정하지 않을 경우 가장 낮은 우선순위에 해당하는 Integer.MAX&#95;VALUE를 우선순위 값으로 갖는다. ViewResolver 구현 클래스가 org.springframework.core.Ordered 인터페이스를 구현하지 않았거나 &#64;Order 애노테이션으로 순서를 정하지 않은 경우에도 가장 낮은 우선순위를 갖게 된다.  

참고로, 스프링이 제공하는 모든 ViewResolver는 Ordered 인터페이스를 상속받고 있기 때문에 order 프로퍼티를 이용해서 우선순위를 지정할 수 있다.  

DispatcherServlet은 우선순위가 높은(즉, order 값이 작은) ViewResolver에게 뷰 이름에 해당하는 View 객체를 요청한다. 만약, 우선순위가 높은 ViewResolver가 null을 리턴하면, 그 다음 우선순위를 갖는 ViewResolver에 View를 요청한다. 최종적으로 ViewResolver를 통해서 View 객체를 구하면, 해당 View 객체를 이용하여 응답 결과를 생성한다.  

아래 코드는 order 프로퍼티를 이용해서 우선순위를 지정한 설정의 예를 보여주고 있다.  

```xml
<bean class="org.springframework.web.servlet.view.BeanNameViewResolver" p:order="0" />

<bean class="org.sprinframework.web.servlet.view.InternalResourceViewResolver" p:prefix="/WEB-INF/viewjsp/" p:suffix=".jsp" p:order="1" />
```
우선순위를 결정할 때 주의할 점은 InternalResourceViewResolver는 마지막 우선순위를 갖도록 지정해야 한다는 점이다. 그 이유는 InternalResourceViewResolver는 항상 뷰 이름에 매핑되는 View 객체를 리턴하기 때문이다. InternalResourceViewResolver는 null을 리턴하지 않기 때문에, InternalResourceViewResolver의 우선순위가 높을 경우 우선순위가 낮은 ViewResolver는 사용되지 않게 된다.  

### 2. HTML 특수 문자 처리 방식 설정
<br/>
스프링은 JSP나 몇몇 템플릿 엔진을 위해 메시지나 커맨드 객체의 값을 출력할 수 있는 기능을 제공하고 있다. JSP를 뷰 기술로 사용할 경우 다음의 커스텀 태그를 이용해서 메시지를 출력할 수 있다.
```html
<title><spring:message code="login.form.title"/></title>
```
만약 &#60;spring:message&#62; 커스텀 태그가 출력하는 값이 '&#60;입력폼&#62;'이라고 하자. 이때, '&#60;'와 '&#62;'는 HTML에서 특수 문자이기 때문에 '&#38;lt;'나 '&#38;gt;'와 같은 엔티티레퍼런스로 변환해주어야 원하는 형태로 화면에 표시가 된다.  

이런 특수 문자를 어떻게 처리할 지의 여부는 defaultHtmlEscape 컨텍스트 파라미터를 통해서 지정할 수 있다. 아래 코드는 설정 예이다.  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app ...>

	<context-param>
		<param-name>defaultHtmlEscape</param-name>
		<param-value>false</param-value>
	</context-param>

	...

</web-app>
```
defaultHtmlEscape 컨텍스트 파라미터의 값을 true로 지정하면 스프링이 제공하는 커스텀 태그나 Velocity 매크로는 HTML '&#38;'나 '&#60;'와 같은 특수 문자를 엔티티 레퍼런스로 치환한다. defaultHtmlEscape 컨텍스트 파라미터의 값이 false면, 특수 문자를 그대로 출력한다.  

참고로, defaultHtmlEscape 컨텍스트 파라미터의 기본 값은 true이다.

### 3. JSP를 이용한 뷰 구현
<br/>
JSP를 뷰로 사용하려면 앞서 살펴봤듯이 InternalResourceViewResolver를 사용한다. 아래 코드처럼 suffix 프로퍼티의 값을 .jsp로 지정함으로써 논리적 뷰 이름을 특정 JSP에 매핑할 수 있다.

```
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/view/" />
	<property name="suffix" value=".jsp" />
</bean>
```
위 설정을 /WEB-INF 디렉터리의 하위 디렉터리에 JSP 파일을 위치시키고 있다. 이렇게 /WEB-INF 디렉터리에 JSP를 위치시키는 이유는 클라이언트가 뷰를 위한 JSP에 직접 접근하는 것을 막기 위함이다. /WEB-INF 디렉터리는 특수한 디렉터리로서 웹 컨테이너는 클라이언트가 /WEB-INF 경로에 직접 접근하는 것을 제한하고 있다.  

스프링은 JSP에서 사용할 수 있는 커스텀 태그를 제공하고 있다.  

#### 3.1. 메시지 출력을 위한 설정
<br/>
스프링이 제공하는 커스텀 태그를 이용해서 메시지를 출력하려면 먼저 MessageSource를 등록해야 한다.  
```xml
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
	<property name="basenames">
		<list>
			<value>message.label</value>
			<value>message.error</value>
		</list>
	</property>
	<property name="defaultEncoding" value="UTF-8" />
</bean>
```

#### 3.2. 메시지 출력을 위한 &#60;spring:message&#62; 커스텀 태그
<br/>
스프링은 MessageSource로부터 메시지를 가져와 출력해주는 &#60;spring:message&#62; 커스텀 태그를 제공하고 있다. &#60;spring:message&#62; 커스텀 태그는 다음과 같이 code 속성을 이용하여 익어 올 메시지의 코드를 지정한다.  

```jsp
<%@ page contentType="text/html; charset=utf-8" %>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<!DOCTYPE html>
<html>
	<head><title><spring:message code="login.form.title"/></title></head>
	<body>
		<form:form commandName="loginCommand">
			<form:errors element"div" />
			<p>
				<label for="email"><spring:message code="email" /></head>
				<input type="text" name="email" id="email" value="${loginCommand.email}">
				<form:errors path="email"/>
			</p>
			<p>
				<label for="password"><spring:message code="password" /></label>
				<input type="password" name="password" id="password">
				<form:errors path="password"/>
			</p>
			<input type="submit" value="<spring:message code"login.form.login" />">
		</form:form>

		<ul>
			<li><spring:message code="login.form.help" /></li>
		</ul>
	</body	
</html>
```
위 코드에서 사용되는 메시지를 읽어올 때 사용되는 리소스 파일은 다음과 같이 각 코드 값에 해당하는 메시지를 설정하고 있을 것이다.

```
email=이메일
password=암호

login.form.title=로그인 폼
login.form.login=로그인
login.form.help=이메일/암호로 yuna@yuna.com/yuna 입력 테스트
```

메시지 리소스 파일은 {숫자} 형식을 이용하여 변하는 부분을 명시할 수 있다. 다음은 메시지 프로퍼티 파일에 {숫자} 형식을 사용한 예를 보여주고 있다.  

```
greeting={0} 회원님, {1}
```

{0}과 {1}은 플레이스홀더(place holder)로서 &#60;spring:message&#62; 커스텀 태그는 arguments 속성을 통해 플레이스홀더에 들어갈 값을 설정할 수 있다. 이때, 각 값은 콤마를 이용하여 구분한다. 아래 코드는 arguments 속성의 사용 예를 보여 주고 있다.  
```
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
...
<spring:message code="greeting" arguments="${me}, ${greeting}" />
```
&#60;spring:message&#62; 태그는 주어진 코드에 해당하는 메시지가 존재하지 않으면 익셉션을 발생시킨다. 코드에 해당하는 메시지가 없을 때 익셉션을 발생시키는 대신 지정한 메시지를 출력하고 싶다면 text 속성에 기본 메시지를 입력하면 된다.  
```
<spring:message code="no_code" text="코드가 없습니다"/>
```

출력되는 메시지에 '&#60;'나 '&#38;'와 같이 HTML에서 특수하게 처리되는 문자가 포함되어 있을 경우, 알맞게 처리해주어야 할 것이다. 이런 경우에는 htmlEscape 속성의 값을 true로 지정해주면 된다. 그러면, '&#60;', '&#38;'와 같은 특수한 문자는 '&#38;lt;', '&#38;amp;'와 같이 변경되어 출력된다. htmlEscape 속성값을 지정하지 않으면, defaultHtmlEscape 컨텍스트 파라미터에서 지정한 값을 사용한다.  

자바 스크립트에서 &#60;spring:message&#62; 태그가 생성한 문자열을 변수 값으로 사용하고 싶다면 javaScriptEscape 속성의 값을 true로 지정한다. javaScriptEscape 속성 값이 true인 경우, 작은 따옴표나 큰 따옴표와 같은 문자를 \' 또는 \"와 같은 자바 스크립트 특수 문자로 치환해준다.  
```jsp
<script type="text/javascript">
	var value = '<spring:message code="title" javascriptEscape="true" />'
</script>
...
<input type="submit" value="<spring:message code="login.form.submit" htmlEscape="false" />
```

&#60;spring:message&#62; 태그가 생성한 메시지를 출력하지 않고 request나 session과 같은 기본 객체의 속성에 저장할 수도 있다. 아래는 사용 예이다.
```jsp
<spring:message code="login.form.password" var="label" scope="request"/>
${label}: <input type=.../>
```

var 속성은 &#60;spring:message&#62; 태그가 생성한 메시지를 저장할 변수 이름을 지정한다. 이 변수 이름은 JSP의 page, request, session 기본 객체 등에 값을 저장할 때 사용되는 속성 이름으로 사용된다. scope 속성은 메시지를 저장할 범위를 지정한다. 지정 가능한 범위는 page, request, session, application이며, 기본 값은 page이다.  

#### 3.3. 스프링이 제공하는 폼 관련 커스텀 태그
<br/>
스프링의 장점 중 하나는 입력 폼 값을 커맨드 객체에 저장하는 기능을 제공한다는 것이다. 또한 반대로 커맨드 객체의 값을 입력 폼에 출력해주는 JSP 커스텀 태그를 제공하고 있어, 좀 더 쉽게 폼 관련 태그를 생성할 수 있도록 도와 준다. 이들 태그를 사용하려면 다음과 같이 커스텀 태그를 설정해주어야 한다.

```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
```
스프링은 &#60;form:form&#62; 태그부터 &#60;form:textarea&#62; 태그에 이르기까지 커맨드 객체를 손쉽게 연동할 수 있도록 도와주는 커스텀 태그를 제공하고 있다.

##### 3.3.1. &#60;form&#62; 태그를 위한 커스텀 태그 : &#60;form:form&#62;
<br/>
&#60;form:form&#62; 커스텀 태그는 &#60;form&#62; 태그를 생성할 때 사용된다. &#60;form:form:&#62; 커스텀 태그를 사용하는 가장 간단한 방법은 다음과 같다.
```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
...
<form:form>
	<form:errors />
	<p>
		<label for="email"><spring:message code="email" /></label>:
		<input type="text" name="email" id="email" value="${loginCommand.email}">
		<form:errors path="email"/>
	</p>
	...
	<input type="submit" value="<spring:message code="login.form.login" />">
</form:form>
```
&#60;form:form&#62; 태그의 method 속성과 action 속성을 지정하지 않으면 method 속성의 값은 "post"로 설정되고 action 속성의 값은 현재 요청 URL의 값이 설정된다.  

생성된 &#60;form&#62; 태그의 id 속성은 입력 폼의 값을 저장하는 커맨드 객체의 이름이 할당된다. 만약, 커맨드 객체의 이름이 기본값인 "command"가 아니라면 다음과 같이 commandName 속성에 커맨드 객체의 이름을 명시해주어야 한다.  

&#60;form:form&#62; 커스텀태그는 &#60;form&#62; 태그와 관련하여 다음의 속성들을 추가적으로 제공하고 있다.  
+ action : 폼 데이터를 정송할 URL을 입력(HTML &#60;form&#62; 태그 속성)
+ enctype : 전송될 데이터의 인코딩 타입. HTML &#60;form&#62; 태그 속성과 동일
+ method : 전송 방식. HTML &#60;form&#62; 태그 속성과 동일  

&#60;form:form&#62; 태그의 몸체에는 &#60;input&#62; 태그나 &#60;select&#62; 태그와 같이 입력 폼을 출력하는 데 필요한 HTML 태그를 입력할 수 있다. 이때, 입력한 값이 잘못 되어 다시 값을 입력해야 하는 경우에는 다음과 같이 커맨드 객체의 값을 사용해서 이전에 입력한 값을 출력할 수 있을 것이다.  

이렇게 커맨드 객체의 값을 다시 입력 폼에 출력해주어야 하는 경우에는 &#60;form:input&#62;이나 &#60;form:checkbox&#62;와 같이 스프링이 제공하는 커스텀 태그를 사용하면 좀 더 쉽게 커맨드 객체의 값을 입력 폼에 설정할 수 있다. 스프링은 입력 폼을 위한 다양한 커스텀 태그를 제공하고 있다.  

##### 3.3.2. &#60;input&#62; 태그를 위한 커스텀 태그 : &#60;form:input&#62;, &#60;form:password&#62;, &#60;form:hidden&#62;
<br/>
스프링은 &#60;input&#62; 태그를 위해 다음과 같은 커스텀 태그를 제공하고 있다.  
<table>
	<thead>
		<tr>
			<td>커스텀 태그</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>&#60;form:input</td>
			<td>text 타입의 &#60;input&#62; 태그</td>
		</tr>
		<tr>
			<td>&#60;form:password</td>
			<td>password 타입의 &#60;input&#62; 태그</td>
		</tr>
		<tr>
			<td>&#60;form:hidden</td>
			<td>hidden 타입의 &#60;input&#62; 태그</td>
		</tr>
	</tbody>
</table>

&#60;form:input&#62; 커스텀 태그는 다음과 같이 path 속성을 사용해서 바인딩 될 커맨드 객체의 프로퍼티를 지정한다.  
```jsp
<form:form commandName="memberInfo">
<p>
	<form:label path="userId">회원ID</form:label>
	<form:input path="userId" />
	<form:errors path="userId" />
</p>
```
위 코드가 생성하는 HTML &#60;input&#62; 태그는 아래와 같다. 이때 id 속성과 name 속성의 값은 프로퍼티의 이름으로 설정하고, value 속성에는 &#60;form:input&#62; 커스텀 태그의 path 속성에서 지정한 커맨드 객체의 프로퍼티 값이 출력된다.  

```html
<form id="memberInfo" action="경로명" method="post">
<p>
	<label for="email">이메일</label>:
	<input id="email" name="email" type="text" value=""/>

</p>
```
&#60;form:password&#62; 커스텀 태그는 password 타입의 &#60;input&#62; 태그를 생성하고, &#60;form:hidden&#62; 커스텀 태그는 hidden 타입의 &#60;input&#62; 태그를 생성한다. 두 태그 모두 path 속성을 사용하여 바인딩 할 커맨드 객체의 프로퍼티를 지정한다.  

##### 3.3.3. &#60;select&#62; 태그를 위한 커스텀 태그 : &#60;form:select&#62;, &#60;form:options&#62;, &#60;form:option&#62;
<br/>
&#60;select&#62; 태그와 관련된 커스텀 태그는 다음과 같이 세 가지가 존재한다.  

<table>
	<thead>
		<tr>
			<td>커스텀 태그</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>&#60;form:select&#62;</td>
			<td>&#60;select&#62;태그를 생성한다. &#60;option&#62; 태그를 생성하는 데 필요한 콜렉션을 전달받을 수도 있다.</td>
		</tr>
		<tr>
			<td>&#60;form:options&#62;</td>
			<td>지정한 콜렉션 객체를 이용하여 &#60;option&#62; 태그를 생성한다.</td>
		</tr>
		<tr>
			<td>&#60;form:option&#62;</td>
			<td>한 개의 &#60;option&#62; 태그를 생성한다.</td>
		</tr>
	</tbody>
</table>
<br/>
&#60;select&#62; 태그는 선택 옵션을 제공할 때 주로 사용된다. 예를 들어, &#60;select&#62; 태그를 이용해서 작업 선택을 위한 옵션을 제공한다고 하자. 이런 옵션 정보는 컨트롤러에서 생성해서 뷰에 전달하는 경우가 많다. 보통 &#64;ModelAttribute 애노테이션을 이용해서 &#60;select&#62; 태그에서 사용될 옵션 목록을 전달한다.  
```java
@ModelAttribute("loginTypes")
protected List<String> referenceData() throws Exception {
	List<String> loginTypes = new ArrayList<String>();
	loginTypes.add("일반회원");
	loginTypes.add("기업회원");
	loginTypes.add("헤드헌터회원");
	return loginTypes;
}
```
이 경우 &#60;form:select&#62; 커스텀 태그를 사용하면 뷰에 전달한 객체를 이용하여 간단하게 &#60;select&#62;와 &#60;option&#62; 태그를 생성할 수 있다. 다음 코드는 &#60;form:select&#62; 커스텀 태그를 이용하여 &#60;select&#62; 태그를 생성하는 예를 보여주고 있다. 이때 path 속성은 바인딩 될 커맨드 객체의 프로퍼티 이름을 입력하며, items 속성에는 &#60;option&#62; 태그를 생성할 때 사용될 콜렉션 객체를 지정한다.  
```jsp
<form:form commandName="login">
	<form:errors />
	<p>
		<label for="loginTypes"><spring:message code="login.form.type" /></label>
		<form:select path="loginType" items="${loginTypes}" />
	</p>
	...
</form:form>
```
위의 &#60;form:select&#62; 커스텀 태그는 다음과 같은 HTML 태그를 생성한다. (실제로는 한 줄로 생성되는데, 가독성을 위해 형식을 일부 변경했다.)  

```html
<select id="loginType" name="loginType">
	<option value="일반회원">일반회원</option>
	<option value="기업회원">기업회원</option>
	<option value="헤드헌터회원">헤드헌터회원</option>
</select>
```
생성된 코드를 보면 콜렉션 객체의 값을 갖고서 &#60;option&#62; 태그의 value 속성과 텍스트를 설정한 것을 알 수 있다.  

&#60;form:options&#62; 태그를 사용해도 동일한 작업을 수행할 수 있다. &#60;form:options&#62; 커스텀 태그를 사용할 경우, 다음과 같이 &#60;form:select&#62; 커스텀 태그에 &#60;form:options&#62; 커스텁 태그를 중첩하며, &#60;form:options&#62; 커스텀 태그에서 items 속성을 설정한다.  
```jsp
<form:select path="loginType">
	<option value="">--- 선택하세요 ---</option>
	<form:options items="${loginTypes}"/>
</form:select>
```
&#60;form:options&#62; 커스텀 태그는 주로 위 코드의 &#60;options&#62; 태그처럼 콜렉션에 포함되지 않은 값을 갖는 &#60;option&#62; 태그를 함께 추가할 때 사용된다.  

&#60;form:option&#62; 커스텀 태그는 &#60;option&#62; 태그를 직접 지정할 때 사용된다. 다음 코드는 &#60;form:option&#62; 커스텀 태그의 사용 예이다.  

```jsp
<form:select path="lgoinType">
	<form:option value="일반회원" />
	<form:option value="기업회원">기업</form:option>
	<form:option value="헤드헌터회원 label="헤드헌터" />
</form:select>
```
&#60;form:option&#62; 커스텀 태그의 value 속성은 &#60;option&#62; 태그의 value 속성 값을 지정한다. &#60;form:option&#62; 커스텀 태그의 몸체 내용을 입력하지 않으면 value 속성에 지정한 값이 텍스트로 사용되고, 몸체 내용을 입력하면 몸체 내용이 텍스트로 사용된다. label 속성을 사용할 경우에는 label 속성에 명시한 값이 텍스트로 사용된다. 위 코드가 생성한 HTML 코드는 다음과 같다.  

```html
<select id="loginType" name="loginType">
	<option value="일반회원">일반회원</option>
	<option value="기업회원">기업</option>
	<option value="헤드헌터회원">헤드헌터</option>
</select>
```
&#60;option&#62; 태그를 생성하는 데 사용되는 콜렉션 객체가 String이 아닐 수도 있다. 예를 들어, &#60;option&#62;을 생성하는 데 사용되는 콜렉션에 다음과 같은 Code 클래스의 객체가 저장된다고 하자.  

```java
public class Code {

	private String code;
	private String label;
	... // get & set 메서드
}
```
이 경우, Code 객체의 code 프로퍼티와 label 프로퍼티를 각각 &#60;option&#62; 태그의 value 속성과 텍스트로 사용하고 싶을 것이다. 이렇게 콜렉션에 저장된 객체 자체가 아닌 객체의 특정 프로퍼티를 사용해야 하는 경우에는 다음과 같이 itemValue 속성과 itemLabel 속성을 사용해서 &#60;option&#62; 태그를 생성하는 데 사용될 객체의 프로퍼티를 지정할 수 있다.  

```jsp
<form:select path="jobCode">
	<option value="">--- 선택하세요 ---</option>
	<form:options items="${jobCodes}" itmeLabel="label" itemValue="code" />
</form:select>
```
위 코드는 jobCodes 콜렉션에 저장된 객체를 이용해서 &#60;option&#62; 태그를 생성한다. 이때 객체의 code 프로퍼티 값을 &#60;option&#62; 태그의 value 속성 값으로 사용하고, 객체의 label 프로퍼티의 값을 &#60;option&#62; 태그의 텍스트로 사용한다. &#60;form:select&#62; 커스텀 태그도 &#60;form:options&#62; 커스텀 태그와 마찬가지로 itemLabel 속성과 itemValue 속성을 사용할 수 있다.  

스프링이 제공하는 &#60;form:select&#62;, &#60;form:options&#62;, &#60;form:option&#62; 커스텀 태그의 장점은 커맨드 객체의 프로퍼티 값과 일치하는 값을 갖는 &#60;option&#62;을 자동으로 선택해준다는 점이다. 예를 들어, 커맨드 객체의 loginType 프로퍼티의 값이 "기업회원"인 경우 다음과 같이 일치하는 &#60;option&#62; 태그에 selected 속성이 추가된다.  

```html
<select id="loginType" name="loginType">
	<option value="일반회원">일반회원</option>
	<option value="기업회원" selected="selected">기업회원</option>
	<option value="헤드헌터회원">헤드헌터회원</option>
</select>
```

##### 3.3.4. checkbox 타입 &#60;input&#62; 태그를 위한 커스텀 태그 : &#60;form:checkboxes&#62;, &#60;form:checkbox&#62;
<br/>
한 개 이상의 값을 커맨드 객체의 특정 프로퍼티에 저장하고 싶은 경우, 배열이나 List와 같은 타입을 사용해서 값을 저장한다.  
```java
public class MemberRegistRequest {

	private String[] favoriteOs;

	public String[] getFavoriteOs() {
		return favoriteOs;
	}

	public void setFavoriteOs(String[] favoriteOs) {
		this.favoriteOs = favoriteOs;
	}

	...

}
```
HTML 입력 폼에서는 다음과 같이 checkbox 타입의 &#60;input&#62; 태그를 이용해서 한 개 이상의 값을 선택할 수 있도록 한다.  
```html
<input type="checkbox" name="favoriteOs" value="윈도우XP">윈도우XP</input>
<input type="checkbox" name="favoriteOs" value="윈도우7">윈도우7</input>
```
스프링은 checkbox 타입의 &#60;input&#62; 태그와 관련하여 다음과 같은 커스텀 태그를 제공하고 있다.  

<table>
	<thead>
		<tr>
			<td>커스텀 태그</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>&#60;form:checkboxes&#62;</td>
			<td>커맨드 객체의 특정 프로퍼티와 관련된 checkbox 타입의 &#60;input&#62; 태그 목록을 생성한다.</td>
		</tr>
		<tr>
			<td>&#60;form:checkbox&#62;</td>
			<td>커맨드 객체의 특정 프로퍼티와 관련된 한 개의 checkbox 타입 &#60;input&#62; 태그를 생성한다.</td>
		</tr>
	</tbody>
</table>
&#60;form:checkboxes&#62; 커스텀 태그는 items 속성을 이용하여 값으로 사용할 콜렉션을 전달받고, path 속성을 이용하여 값을 바인딩 할 커맨드 객체의 프로퍼티를 지정한다. 아래 코드는 &#60;form:checkboxes&#62; 커스텀 태그의 사용 예이다.  

```jsp
<p>
	<form:label path="favoriteOs">선호 OS</form:label>
	<form:checkboxes items="${favoriteOsNames}" path="favoriteOs"/>
	<form:errors path="favoriteOs" />
<p>
```
favoriteOsNames 모델의 값이 {"윈도우XP", "윈도우7"}일 경우, 위 코드의 &#60;form:checkboxes&#62; 커스텀 태그는 다음과 같은 HTML 코드를 생성한다. (아래 코드는 실제로 공백 없이 모두 한 줄로 생성된다.)  

```html
<span>
	<input id="favoriteOs1" name="favoriteOs" type="checkbox" value="윈도우XP"/>
	<label for="favoriteOs1">윈도우XP</label>
</span>
<span>
	<input id="favoriteOs2" name="favoriteOs" type="checkbox" value="윈도우7"/>
	<label for="favoriteOs2">윈도우7</label>
</span>
<input type="hidden" name="_favoriteOs" value="on"/>
```
각 &#60;input&#62; 태그의 value 속성에 사용된 값이 체크박스를 위한 텍스트로 사용되고 있다. 앞서 &#60;option&#62; 태그에서와 마찬가지로 콜렉션에 저장된 객체가 String이 아니라면, itemValue 속성과 itemLabel 속성을 이용해서 값과 텍스트를 설정할 때 사용될 객체의 프로퍼티를 설정할 수 있다.  

```jsp
<p>
	<form:label path="favoriteOs">선호 OS</form:label>
	<form:checkboxes items="${favoritesOsCodes}" path="favoriteOs" itemValue="code" itemLabel="label" />
	<form:errors path="favoriteOs" />
</p>
```
&#60;form:checkbox&#62; 커스텀 태그는 &#60;form:option&#62; 커스텀 태그와 같이 한 개의 checkbox 타입의 &#60;input&#62; 태그를 생성할 때 사용된다. &#60;form:checkbox&#62; 커스텀 태그는 value 속성과 label 속성을 사용해서 값과 텍스트를 설정한다.  

```jsp
<form:checkbox path="favoriteOs" value="WIN2000" label="윈도우즈2000" />
<form:checkbox path="favoriteOs" value="WINXP" label="윈도우XP" />
```
&#60;form:checkbox&#62; 커스텀 태그는 바인딩되는 값의 타입에 따라서 처리 방식이 달라진다. 첫 번째로 바인딩되는 값의 타입이 boolean 기본 데이터 타입이나 Boolean 래퍼 타입이라고 하자.

```java
public class MemberRegistRequest {

	private boolean allowNoti;

	public boolean isAllowNoti() {
		return allowNoti;
	}

	public void setAllowNoti(boolean allowNoti) {
		this.allowNoti = allowNoti;
	}

	...

}
```
이 경우 &#60;form:checkbox&#62;는 바인딩되는 값이 true인 경우 "checked" 속성을 설정하며, false인 경우 "checked" 속성을 설정하지 않는다. 또한 생성되는 &#60;input&#62; 태그의 value 속성의 값은 "true"가 된다. 아래 코드는 사용 예이다.

```jsp
<form:checkbox path="allowNoti" label="이메일을 수신합니다."/>
```

allowNoti의 값이 false와 true인 경우 각각 생성되는 HTML 코드는 다음과 같다. (실제로는 두 경우 모두 한 줄로 생성된다.)

```html
<!-- allowNoti가 false인 경우 -->
<input id="allowNoti" name="allowNoti" type="checkbox" value="true"/>
<label for="allowNoti1">이메일을 수신합니다.</label>
<input type="hidden" name="_allowNoti" value="on"/>

<!-- allowNoti가 true인 경우 -->
<input id="allowNoti" name="allowNoti" type="checkbox" value="true" checked="checked"/>
<label for="allowNoti1">이메일을 수신합니다.</label>
<input type="hidden" name="_allowNoti" value="on"/>
```
&#60;form:checkbox&#62; 태그의 두 번째 처리 방식은 바인딩되는 값의 타입이 배열이나 Collection인 경우이다. 값의 타입이 배열이나 Collection인 경우, 해당 콜렉션에 값이 포함되어 있는 경우에 "checked" 속성을 설정한다. 예를 들어, 아래와 같은 배열 타입의 프로퍼티가 있다고 해보자.

```java
public class MemberRegistRequest {

	private String[] favoriteOs;

	public String[] getFavoriteOs() {
		return favoriteOs;
	}

	public void setFavoriteOs(String[] favoriteOs) {
		this.favoriteOs = favoriteOs;
	}

	...
}
```
&#60;form:checkbox&#62; 커스텀 태그를 사용하면 다음과 같이 favoriteOs 프로퍼티에 대한 폼을 처리할 수 있다.

```jsp
<form:checkbox path="favoriteOs" value="윈도우XP" label="윈도우XP" />
<form:checkbox path="favoriteOs" value="윈도우7" label="윈도우7" />
<form:checkbox path="favoriteOs" value="윈도우8" label="윈도우8" />
```
&#60;form:checkbox&#62; 태그의 세 번째 처리 방식은 임의 타입의 프로퍼티와 바인딩되는 경우이다. 이 경우, &#60;form:checkbox&#62; 태그는 value 속성의 값과 프로퍼티의 값이 일치하지 않는 경우 "checked" 속성의 값을 설정한다.

##### 3.3.5. radio 타입 &#60;input&#62; 태그를 우히ㅏㄴ 커스텀 태그 : &#60;form:radiobuttons&#62; &#60;from:radiobutton&#62;
<br/>
여러 가지 옵션 중에서 한 가지를 선택해야 하는 경추, radio 타입의 &#60;input&#62; 태그를 사용한다. 스프링은 radio 타입의 &#60;input&#62; 태그와 관련하여 다음과 같은 커스텀 태그를 제공하고 있다.  

+ &#60;form:radiobuttons&#62;  
커맨드 객체의 특정 프로퍼티와 관련된 radio. 타입의 &#60;input&#62; 태그 목록을 생성한다.
+ &#60;form:radiobutton&#62;  
커맨드 객체의 특정 프로퍼티와 관련된 한 개의 radio 타입 &#60;input&#62; 태그를 생성한다.  

&#60;form:radiobuttons&#62; 커스텀 태그는 타음과 같이 items 속성을 이용하여 값으로 사용할 콜렉션을 전달받고, path 속성을 이용하여 값을 바인딩 할 커맨드 객체의 프로퍼티를 지정한다.  

```jsp
<p>
	<form:label path="tool">주로 사용하는 개발툴</form:label>
	<form:radiobuttons items="${tools}" path="tool"/>
</p>
```
<br/>
&#60;form:radiobuttons&#62; 커스텀 태그는 다음과 같은 HTML 태그를 생성한다.
```html
<span><input id="tool1" name="tool" type="radio" value="Eclipse"/><label form="tool1">Eclipse</span>
<span><input id="tool2" name="tool" type="radio" value="IntelliJ"/><label form="tool2">IntelliJ</span>
<span><input id="tool3" name="tool" type="radio" value="NetBeans"/><label form="tool3">NetBeans</span>
```
<br/>
&#60;form:radiobutton&#62; 커스텀 태그는 1개의 radio 타입 &#60;input&#62; 태그를 생성할 때 사용되며, value 속성과 label 속성을 이용하여 값과 텍스트를 설정한다. 사용방법은 &#60;form:checkbox&#62; 태그와 동일하다.  

##### 3.3.6. &#60;textarea&#62; 태그를 위한 커스텀 태그 : &#60;form:textarea&#62;
<br/>
게시글 내용과 같이 여러 줄을 입력 받아야 하는 경우 &#60;textarea&#62; 태그를 사용한다. 스프링은 &#60;form:textarea&#62; 커스텀 태그를 제공하고 있으며, 이 태그를 이용하면 커맨드 객체와 관련된 &#60;textarea&#62; 태그를 생성할 수 있다.
```jsp
<p>
	<form:label path="etc">기타</form:label>
	<form:textarea path="etc" cols="20" rows="3"/>
</p>
```
<br/>
```html
<p>
	<label for="etc">기타</label>
	<textarea id="etc" name="etc" rows="3" cols="20"></textarea>
</p>
```

##### 3.3.7. CSS 및 HTML 태그와 관련된 공통 속성
<br/>
&#60;form:input&#62;, &#60;form:select&#62; 등 입력 폼과 관련해서 제공하는 스프링 커스텀 태그는 HTML의 CSS 및 이벤트 관련 속성을 제공하고 있다. 먼저 CSS와 관련된 속성은 다음과 같다.  
+ cssClass : HTML의 class 속성 값
+ cssErrorClass : 폼 검증 에러가 발생했을 때 사용할 HTML의 class 속성 값
+ cssStyle : HTML의 style 속성 값  
<br/>
HTML 태그가 사용하는 속성 중 다음의 속성들도 사용 가능하다.  
+ id, title, dir
+ disabled, tabindex
+ onfocus, onblur, onchange
+ onclick, ondbclick
+ onkeydown, onkeypress, onkeyup
+ onmousedown, onmousemove, onmouseup
+ onmouseout, onmoustover
또한, 각 커스텀 태그는 htmlEscape 속성을 사용해서 커맨드 객체의 값에 포함된 HTML 특수 문자를 엔티티 레퍼런스로 변환할 지의 여부를 결정할 수 있다. htmlEscape 속성을 지정하지 않을 경우 defaultHtmlEscape 컨텍스트 파라미터에서 설정한 값을 사용한다.  

#### 3.4. 값 포맷팅 처리
<br/>
&#60;form:input&#62;을 포함한 스프링의 폼 관련 커스텀 태그는 스프링 MVC를 위해 등록한 PropertyEditor나 ConversionService를 이용해서 값을 변환한다. 예를 들어, 컨트롤러 클래스에서 다음과 같이 Date 타입을 위한 PropertyEditor로 CustomDateEditor를 등록했다고 하자.

```java
@Controller
@RequestMapping("경로명1")
public class 클래스명 {

	...

	@RequestMapping(method = RequestMethod.POST)
	public String regist(@ModelAttribute("memberInfo") MemberRegistRequest memRegReq, BindingResult bindingResult) {
		new MemberRegistValidator().validate(memRegReq, bindingResult);
		if (bindingResult.hasError()) {
			return MEMBER_REGISTRATION_FORM;
		}
		memberService.registNewMember(memRegReq);
		return "member/registered";
	}

	...

	@Initbinder
	protected void initBinder(WebDataBinder binder) {
		CustomDateEditor dateEditor = new CustomDateEditor(new SimpleDateFormat("yyyyMMdd"), true);
		binder.registerCustomEditor(Date.class, dateEditor);
	}

	...

}
```
<br/>
WebDataBinder에 등록한 PropertyEditor는 요청 파라미터의 값을 Date 타입으로 변환하는 데 사요될 뿐만 아니라, 스프링 커스텀 태그에서 커맨드 객체의 Date 타입 프로퍼티 값을 문자열로 변환하는 데 사용된다. memberInfo 커맨드 객체의 birthday 프로퍼티가 Date 타입이라고 할 경우, 아래 &#60;form:input&#62; 태그는 CustomDateFormat을 이용해서 birthday 프로퍼티를 String 타입으로 변환해서 출력한다.  

```jsp
<form:form commandName="memberInfo">
...
	<p>
		<label for="birthday">생일</label>: 형식: YYYYMMDD, 예: 20140101
		<form:input path="birthday">
		<form:errors path="birthday" /><br/>
	</p>
</form:form>
```
<br/>
&#60;mvc:annotation-driven&#62;이나 &#64;EnableWebMvc를 이용해서 설정하면, 스프링 MVC는 ConversionService로 DefaultFormattingConversionService를 사용한다. 따라서, (DefaultFormattingConversionService이 지원하는) &#64;DateTimeFormat 애노테이션을 사용하면, 별도의 PropertyEditor를 등록하지 않아도 &#64;DateTimeFormat에서 지어한 형식으로 값을 출력할 수 있다.  

##### 3.4.1. 커스,텀 포맷터 등록하기
<br/>
WebDateBinder/&#64;InitBinder를 이용해서 타입 변환을 위한 PropertyEditor를 등록하는 방법은 단일 컨트롤러에만 적용된다. 경우에 따라 스프링 MVC의 뷰로 사용되는 모든 JSP에 대해 동일한 변환 방식을 적용하고 싶을 때가 있을 것이다. 이런 경우 CoversionService를 직접 생성해서 Formatter를 등록하는 방법을 사용한다.  

ConversionService를 설정할 때에는 스프링 MVC가 기본으로 사용하는 FormattingConversionServiceFactoryBean를 사용하면 된다. 다음은 설정 예이다.  
```xml
<mvc:annotation-driven conversion-service="formattingConversionService" />

<bean id="formattingConversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
	<property name="formatters">
		<set>
			<bean class="Formatter를 구현한 완전한 클래스명" />
		</set>
	</property>
</bean>
```

#### 3.5. 스프링이 제공하는 에러 관련 커스텀 태그
<br/>
Validator를 이용해서 커맨드 객체의 값을 검사하여 Errors나 BindingResult를 이용해서 에러 정보를 추가한 경우, &#60;form:errors&#62; 커스텀 태그를 이용해서 에러 메시지를 출력할 수 있다. &#60;form:errors&#62; 커스텀 태그는 path 속성을 이용해서 커맨드 객체의 특정 프로퍼티와 관련된 에러 메시지를 출력할 수 있다.  
```jsp
<form:form commandName="memberInfo">
	<p>
		<label for="email">이메일</label>
		<form:input path="email" />
		<form:errors path="email" />
	</p>

	...

</form:form>
```
위 코드의 경우 "email" 포로퍼티와 관련된 모든 에러 메시지를 출력한다.  

&#60;form:errors&#62; 커스텀 태그는 지정한 프로퍼티와 관련된 한 개 이상의 에러 메시지를 출력하게 된다. 각 에러 메시지를 생성할 때 다음과 같은 두 개의 속성이 사용된다.  
+ element : 각 에러 메시지를 출력할 때 사용될 HTML 태그. 기본 값은 span이다.
+ delimiter : 각 에러 메시지를 구분할 때 사용될 HTML 태그. 기본 값은 &#60;br/&#62;이다.  

#### 3.6. &#60;spring:htmlEscape&#62; 커스텀 태그와 htmlEscape 속성
<br/>
defaultHtmlEscape 컨텍스트 파라미터를 사용해서 웹 어플리케이션 전반에 걸쳐서 HTML의 특수 문자를 엔티티 레퍼런스로 치환할 지의 여부를 경정하는데, 만약, 각 JSP 페이지별로 특수 문자 치환 여부를 설정해주고 싶다면 다음과 같이 &#60;spring:htmlEscape&#62; 커스텀 태그를 사용하면 된다.  

```jsp
<%-- JSP 페이지의 앞 부분에서 설정 -->
<spring:htmlEscape defaultHtmlEscape="true"/>
...
<spring:message .../>
<form:input .../>
```
&#60;spring:htmlEscape&#62; 커스텀 태그를 설정하면, 이후로 실행되는 &#60;spring:message&#62; 커스텀 태그나 &#60;form:iput&#62; 커스텀 태그와 같이 스프링이 제공하는 커스텀 태그는 &#60;spring:htmlEscape&#62; 커스텀 태그의 defaultHtmlEscape 속성에서 지정한 값을 기본값으로 사용한다.  

#### 3.7. &#60;form:form&#62;의 RESTful 지원
<br/>
스프링 MVC는 HTTP의 GET, POST, PUT, DELETE 방식을 지원하고 있으며, 컨트롤러 메서드에서 어떤 HTTP 방식을 지원할지 선택할 수 있다.  

그런데, 웹 브라우저는 GET 방식과 POST 방식만을 지원하고 있어서 DELETE 방식이나 PUT 방식의 요청을 전송할 수 없기 때문에, 웹 브라우저를 이용할 경우 GET 방식과 POST 방식으로만 처리해야 했다.  

스프링은 PUT과 DELETE 방식을 이용해서 컨트롤러를 구현하면서, 웹 브라우저에서도 그대로 해당 컨트롤러를 사용할 수 있도록 지원하고 있다. 이 기능을 이용하려면 다음의 두 가지 작업만 해주면 된다.  

+ web.xml 파일에 HiddenHttpMethodFilter 적용
+ &#60;form:form&#62; 태그의 method 속성에 put 또는 delete 이용  

먼저, web.xml 파일에 HiddenHttpMethodFilter를 설정한다. HiddenHttpMethodFilter의 필터 매핑 대상으로는 아래 코드와 같이 DispatcherServlet을 지정한다.  
```xml
<web-app>

	...

	<filter>
		<filter-name>httpMethodFilter</filter-name>
		<filter-class>
			org.springframework.web.filter.HiddenHttpMethodFilter
		</filter-class>
	</filter>

	<filter-mapping>
		<filter-name>httpMethodFilter</filter-name>
		<servlet-name>dispatcher</servlet-name>
	</filter-mapping>

	...

</web-app>
```

이제 &#60;form:form&#62; 태그의 method 속성 값으로 delete 또는 put을 지정한다.  

```jsp
<form:form method="delete">
...
</form:form>
```

&#60;form:form&#62; 태그의 method 값이 delete나 put인 경우 &#60;form:form&#62; 태그는 다음과 같이 hidden 타입의 &#60;input&#62; 태그를 추가로 생성한다.  
```html
<form id="article" action="경로명" method="post">
	<input type="hidden" name="_method" value="delete"/>
	...
	<input type="submit" value="삭제">
</form>
```
HiddenHttpMethodFilter는 요청 파라미터에 &#95;method 파라미터가 존재할 경우, &#95;method 파라미터의 값을 요청 방식으로 사용하도록 스프링 MVC의 관련 정보를 설정한다. Dispatcher은 이 정보를 이용해서 컨트롤러의 알맞은 메서드를 찾기 때문에, 웹 브라우저를 이용하더라도 RESTful 방식으로 구현된 컨트롤러를 사용할 수 있게 된다.  

### 4. HTML 이외의 뷰 구현
<br/>
#### 4.1. 파일 다운로드 구현을 위한 커스텀 View
<br/>
파일 다운로드를 구현하는 경우, 컨트롤러 클래스는 다운로드 받을 파일과 관련된 정보를 생성해서 뷰에 전달할 것이다.  

```java
@Controller
public class DownloadController implements ApplicationContextAware {

	private WebApplicationContext context = null;

	@RequestMapping("/file/{fileId}")
	public ModelAndView download(@PathVariable String fileId, HttpServletResponse response) throws IOException {
		File downloadFile = getFile(fileId);
		if (download == null) {
			response.sendError(HttpServletResponse.SC_NOT_FOUND);
			return null;
		}
		return new ModelAndView("download", "downloadFile", downloadFile);
	}

	private File getFile(String fileId) {
		String baseDir = context.getServletContext().getRealPath("/WEB-INF/files");
		if (fileId.equals("1"))
			return new File(baseDir, "객체지향JCO14회.zip");
		return null;
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		this.context = (WebApplicationContext) applicationContext;
	}

}
```

JSP는 파일 다운로드와 같은 기능을 구현하기 보다는 HTML과 같은 결과를 보여주기에 적합한 뷰 구현 기술이다. 보통, HTML 응답이 아닌 경우에는 그에 알맞은 전용 뷰 클래스를 구현한다. 또한, BeanNameViewResolver를 이용해서 커스텀 뷰 클래스를 사용할 수 있도록 알맞게 설정해주어야 한다. 아래 코드는 설정 예이다.  

```xml
<bean class="org.springframework.web.servlet.view.BeanNameViewResolver" />

<bean id="download" class="뷰구현완전한클래스명" />
```
파일 다운로드를 구현하려면 컨텐트 타입을 "application/octet-stream"과 같이 다운로드를 위한 타입으로 설정해주어야 하며, 다운로드 받는 파일 이름을 알맞게 설정하려면 Content-Disposition 헤더의 값을 알맞게 설정해주어야 한다.  

```java
public class DownloadView extends AbstractView {

	public DownloadView() {
		setContentType("application/download; charset=utf-8");
	}

	@Override
	protected void renderMergedOutputModel(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
		File file = (File) model.get("downloadFile");

		response.setContentType(getContentType());
		response.setContentLength((int) file.length());

		String userAgent = request.getHeader("User-Agent");
		boolean ie = userAgent.indexOf("MSIE") > -1;
		String fileName = null;
		if (ie) {
			fileName = URLEncoder.encode(file.getName(), "utf-8");
		} else {
			fileName = new String(file.getName().getByte("utf-8"), "iso-8859-1");
		}
		response.setHeader("Content-Disposition", "attachment; fileName=\"" + fileName + "\";");
		response.setHeader("Content-Transfer-Encoding", "binary");
		OutputStream out = response.getOutputStream();
		FileInputStream fis = null;
		try {
			fis = new FileInputStream(file);
			FileCopyUtils.copy(fis, out);
		} fianlly {
			if (fis != null)
				try {
					fis.close();
				} catch (IOException ex) {
				}
		}
		out.flush();
	}
}
```


#### 4.2. AbstractExcelView 클래스를 이용한 엑셀 다운로드 구현
<br/>
스프링은 엑셀 형식으로 뷰 데이터를 생성할 수 있도록 다음의 두 View 클래스를 제공하고 있다.  
+ AbstractExcelView : POI API를 이용하여 엑셀 응답을 생성한다.
+ AbstractJExcelView : JExcel API를 이용하여 엑셀 응답을 생성한다.  

POI API를 이용하여 엑셀 응답 결과를 생성하는 AbstractExcelView 클래스의 사용 방법을 살펴볼 것이다. POI를 이용하려면 관련 의존 설정을 메이븐 pom,xml 파일에 추가해주어야 한다. 다음은 추가 예이다.  

```xml
...

<dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi</artifactId>
	<version>3.9</version>
</dependency>

...
```
AbstractExcelView 클래스는 엑셀 응답 결과를 생성하기 위한 기본 기능을 제공하고 있으며, 이 클래스를 상속받은 뒤 다음의 메서드만 알맞게 재덩의하면 된다.  

HSSFWorkbook은 POI API가 제공하는 엑셀 관련 클래스다. 하위 클래스는 이 클래스를 상속받아 엑셀 문서를 생성하면 된다.  

```java
public class 클래스명 extends AbstractExcelView {

	@SuppressWarning("unchecked")
	@Override
	protected void buildExcelDocument(Map<String, Object> model, HSSFWorkbook workbook, HttpServletRequest request, HttpServletResponse response) throws Exception {
		response.setHeader("Content-Disposition", "attachment; filename=\"pagerank.xls\";");

		HSSFSheet sheet = createFirstSheet(workbook);
		createColumnLabel(sheet);

		List<PageRank> pageRanks = (List<PageRank>) model.get("pageRankList");
		int rowNum = 1;
		for (PageRank rank : pageRanks) {
			createOageRankRow(sheet, rank, rowNum++);
		}
	}

	private HSSFSheet createFirstSheet(HSSFWrokbook workbook) {
		HSSFSheet sheet = workbook.createSheet();
		workbook.setSheetName(0, "페이지 순위");
		sheet.setColumnWidth(1, 256 * 20);
		return sheet;
	}

	private void createColumnLabel(HSSFSheet sheet) {
		HSSFRow firstRow = sheet.createRow(0);
		HSSFCell cell = firstRow.createCell(0);
		cell.setCellValue("순위");

		cell = firstRow.createCell(1);
		cell.setCellValue("페이지");
	}

	private void createPageRankRow(HSSFSheet sheet, PageRank rank, int rowNum) {
		HSSFRow row = sheet.createRow(rowNum);
		HSSFCell cell = row.createCell(0);
		cell.setCellValue(rank.getRank());

		cell = row.createCell(1);
		cell.setCellValue(rank.getPage());

	}

}
```

PageRankView를 만들었으므로, 컨트롤러에서 사용할 수 있도록 다음과 같이 "pageRank"라는 이름으로 PageRankView를 빈으로 설정하자.  

```xml
<bean id="pageRank" class="net.madvirus.spring4.chap08.stat.PageRankView">
</bean>

<bean class="org.springframework.web.servlet.view.BeanNameViewResolver">
	<property name="order" value="1"/>
</bean>
```
컨트롤러 클래스는 다음과 같이 PageRankView에서 필요로 하는 데이터를 모델에 담고 사용할 뷰 이름으로 "pageRank"를 리턴함으로써, PageRankView를 뷰로 사용할 수 있게 된다.  

```java
@Controller
public class PageRankStatController {

	@RequestMapping("/pagestat/rank")
        public String pageRank(Model model) {
                List<PageRank> pageRanks = ...
                model.addAttribute("pageRankList", pageRanks);
                return "pageRank";
        }
}
```

#### 4.3. AbstractPdfView 클래스를 이용한 PDF 다운로드 구현
<br/>
스프링은 iText API를 이용해서 PDF를 생성할 수 있는 AbstractPdfView 클래스를 제공하고 있다. iText를 사용하려면 다음과 같은 의존 설정을 메이븐 pom.xml에 추가해야 한다. (참고로, 아래 설정에서 &#60;exclutions&#62; 부분은 iText를 사용하기 위해 추가로 필요한 모듈 중에서 사용하지 않을 모듈을 지정한 것으로서 암호화 관련된 기능을 제외시켰다.)  

```xml
<dependency>
	<groupId>com.lowagie</groupId>
	<artifactId>itext</artifactId>
	<version>2.1.7</version>
	<exclusions>
		<exclusion>
			<groupId>bouncycastle</groupId>
			<artifactId>bcmail-jdk14</artifactId>
		</exclusion>
		<exclussion>
			<groupId>bouncycastle</groupId>
			<artifactId>bcprov-jdk14</artifactId>
		</exclusion>
		<exclusion>
			<groupId>bouncycastle</groupId>
			<artifactId>bctsp-jdk14</artifactId>
		</exclusion>
	</exclusions/>
</dependency>
```
AbstractPdfView 클래스를 상속받은 클래스는 다음의 메서드를 알맞게 재정의해서 PDF를 생성하면 된다.  

com.lowagie.text.Document 클래스는 iText가 제공하는 클래스로서, Document 객체에 PDF 문서를 생성하는 데 필요한 데이터를 추가함으로써 PDF 문서를 생성할 수 있다.  

```java
public class PageReportView extends AbstractPdfView {
	private String fontPath = "c:\\windows\\Fonts\\malgun.ttf";

	@SuppressWarnings("unchecked")
	@Override
	protected void buildPdfDocument(Map<String, Object> model, Document document, PdfWriter writer, HttpServletRequest request, HttpServletResponse response) throws Exception {
		List<PageRank> pageRanks = (List<PageRank>) model.get("pageRankList");
		Table table = new Table(2, pageRanks.size() + 1);
		table.setPadding(5);

		BaseFont bfKorean = BaseFont.createFont(fontPath, BaseFont.IDENTITY_H, BaseFont.EMBEDDED);

		Font font = new Font(bfKorean);
		Cell cell = new Cell(new Paragraph("순위", font));
		cell.setHeader(true);
		table.addCell(cell);
		cell = new Cell(new Paragraph("페이지", font));
		table.addCell(cell);
		table.endHeaders();

		for (PageRank rank : pageRanks) {
			table.addCell(Integer.toString(rank.getRank()));
			table.addCell(rank.getPage());
		}
		document.add(table);
	}

	public void setFontPath(String fontPath) {
		this.fontPath = fontPath;
	}

}
```

### 5. Locale 처리
<br/>
스프링이 제공하는 &#60;spring:message&#62; 커스텀 태그는 웹 요청과 관련된 언어 정보를 이용해서 알맞은 언어의 메시지를 출력한다.  
실제로, 스프링 MVC는 LocaleResolver를 이용해서 웹 요청과 관련된 Locale을 추출하고, 이 Locale 객체를 이용해서 알맞은 언어의 메시지를 선택하게 된다.  

#### 5.1. LocaleResolver 인터페이스

```java
public interface LocaleResolver {
	Locale resolveLocale(HttpServletRequest request);
	void setLocale(HttpServletRequest, HttpServletResponse response, Locale locale);
}
```

resolveLocale() 메서드는 요청과 관련된 Locale을 리턴한다. DispatcherServlet은 등록되어 있는 LocaleResolver의 resolveLocale() 메서드를 호출해서 웹 요청을 처리할 때 사용할 Locale을 구한다.  

setLocale() 메서드는 Locale을 변경할 때 사용된다. 예를 들어, 쿠기나 HttpSession에 Locale 정보를 저장할 때에 이 메서드가 사용된다.

#### 5.2. LocaleResolver의 종류
<br/>
스프링이 기본적으로 제공하는 LocaleResolver 구현 클래스는 모두 org.springframework.web.servlet.i18n 패키지에 위치한다.  

+ AcceptHeaderLocaleResolver  
웹 브라우저가 전송한 Accept-Language 헤더로부터 Locale을 선택한다. setLocale() 메서드를 지원하지 않는다.
+ CookieLocaleResolver  
쿠키를 이용해서 Locale 정보를 구한다. setLocale() 메서드는 쿠키에 Locale 정보를 저장한다.
+ SessionLocaleResolver  
세션으로부터 Locale 정보를 구한다. setLocale() 메서드는 세션에 Locale 정보를 저장한다.
+ FixedLocaleResolver  
웹 요청에 상관없이 특정한 Locale로 설정한다. setLocale() 메서드를 지원하지 않는다.  

LocaleResolver를 직접 등록할 경우 빈의 이름을 "localeResolver"로 등록해주어야 한다.  

##### 5.2.1. AcceptHeaderLocaleResolver
<br/>
LocaleResolver를 별도로 설정하지 않을 경우 AcceptHeaderLocaleResolver를 기본 LocaleResolver로 사용한다. AcceptHeaderLocaleResolver는 Accept-Language 헤더로부터 Locale 정보를 추출한다.  

헤더로부터 Locale 정보를 추출하기 때문에, setLocale() 메서드를 이용해서 Locale 설정을 변경할 수 없다.  

##### 5.2.2. CookieLocaleResolver
<br/>
CookieLocalResolver는 쿠키를 이용해서 Locale 정보를 저장한다. setLocale() 메서드를 호출하면 Locale 정보를 담은 쿠키를 생성하고, resolveLocale() 메서드는 쿠키로부터 Locale 정보를 가져와 Locale을 설정한다. 만약 Locale 정보를 담은 쿠키가 존재하지 않을 경우, defaultLocale 프로퍼티의 값을 Locale로 사용한다. defaultLocale 프로퍼티의 값이 null인 경우에는 Accept-Language 헤더로부터 Locale 정보를 추출한다.  

CookieLocaleResolver는 쿠키와 관련해서 별도 설정을 필요로 하지 않지만, 생성할 쿠키 이름, 도메인, 경로 등의 설정을 직접하고 싶다면 다음과 같은 프로퍼티를 알맞게 설정해주면 된다.  

+ cookieName : 사용할 쿠키 이름
+ cookieDomain : 쿠키 도메인
+ cookiePath : 쿠키 경로. 기본값은 "/"이다.
+ cookieMaxAge : 쿠키 유효 시간
+ cookieSecure : 보안 쿠키 여부. 기본값은 false이다.  

##### 5.2.3. SessionLocaleResolver
<br/>
SessionLocaleResolver는 HttpSession에 Locale 정보를 저장한다. setLocale() 메서드를 호출하면 Locale 정보를 세션에 저장하고, resolveLocale() 메서드는 세션으로부터 Locale을 가져와 웹 요청의 Locale을 설정한다. 만약 Locale 정보가 세션에 존재하지 않으면, defaultLocale 프로퍼티의 값을 Locale로 사용한다. defaultLocale 프로퍼티의 값이 null인 경우에는 Accept-Language 헤더로부터 Locale 정보를 추출한다.

##### 5.2.4. FixedLocaleResolver
<br/>
FixedLocaleResolver는 웹 요청에 상관없이 defaultLocale 프로퍼티로 설정한 값을 웹 요청을 위한 Locale로 사용한다. FixedLocaleResolver는 setLocale() 메서드를 지원하지 않는다. setLocale() 메서드를 호출할 경우 UnsupportedOperationException 예외를 발생시킨다.

#### 5.3. LocaleResolver 등록
<br/>
DispatcherServlet은 이름이 "localeResolver"인 빈을 LocaleResolver로 사용한다. 따라서, 원하는 LocalResolver를 적용하고 싶다면, 빈 이름을 잘못 입력하지 않도록 주의해야 한다.  

#### 5.4. LocaleResolver를 이용한 Locale 변경
<br/>
LocaleResolver를 빈으로 등록했다면, LocaleResolver를 이용해서 Locale을 변경할 수 있게 된다. 예를 들어, 다음과 같이 LocaleResolver를 설정했다고 하자.
```xml
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver" />
```

이 경우, 컨트롤러 클래스는 다음과 같이 LocaleResolver의 setLocale() 메서드를 호출해서 클라이언트의 웹 요청을 위한 Locale을 변경할 수 있다.  

```java
@Controller
public class LocaleChangeController {

	@Inject
	private LocaleResolver localeResolver;

	@RequestMapping("경로명")
	public String change(@RequestParam("lang") String language, HttpServletRequest request, HttpServletResponse response) {
		Locale locale = new Locale(language);
		// LocaleResolver localResolver = RequestContextUtils.getLocaleResolver(request);
		localeResolver.setLocale(request, response, locale);
		return "redirect:/index.jsp";
	}

}
```

LocaleResolver를 이용해서 Locale을 변경하면, 이후 요청에 대해서는 지정한 Locale을 이용해서 메시지 등을 로딩하게 된다.  

RequestContextUtils 클래스는 웹 요청과 관련된 LocaleResolver를 구할 수 있는 메서드를 제공하고 있다.  

Locale 변경을 지원하지 않는 LocaleResolver의 setLocale() 메서드를 실행하면 익셉션이 발생한다. 따라서, Locale 변경 기능을 사용할 때에는 사용하는 LocaleResolver가 Locale 변경을 지원하는지 확인해야 한다.  

#### 5.5. LocaleChangeInterceptor를 이용한 Locale 변경
<br/>
Locale을 변경하기 위해 별도의 컨트롤러 클래스를 개발한다는 것은 다소 성가신 일이다. 이 경우, 스프링이 제공하는 LocaleChangeInterceptor 클래스를 사용하면 웹 요청 파리미터를 이용해서 손쉽게 Locale을 변경할 수 있다.  

LocaleChangeInterceptor 클래스는 HandlerInterceptor로서 다음곽 같이 설정한다. 아래 설정에서 paramName 프로퍼티는 Locale 언어를 변경할 때 사용될 요청 파라미터의 이름을 지정한다.  

```xml
<mvc:interceptors>
	<bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
		<property name="paramName" value="language" />
	</bean>
</mvc:interceptors>
```
위 코드에서는 paramName 프로퍼티의 값으로 language를 설정했는데, 이 경우 language 요청 파라미터를 사용해서 Locale을 변경할 수 있다.  

LocaleChanageInterceptor는 paramName 프로퍼티로 설정한 요청 파라미터가 존재할 경우, 파라미터의 값을 이용해서 Locale을 생성한 뒤 LocaleResolver를 이용해서 Locale을 변경한다. 이후, 요청에서는 변경된 Locale이 적용된다.
