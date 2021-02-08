---
title: 스프링 MVC XML/JSON, 파일 업로드, 웹소켓
categories:
- Spring
feature_text: |
  ## 스프링 MVC : XML/JSON, 파일 업로드, 웹소켓
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	td { border: 1px solid #444444; }
</style>
### 1. XML/JSON 변환 처리
<br/>
서비스나 데이터를 HTTp 기반 API 형태로 제공하는 곳이 많다. 예를 들어, 트위터는 타임라인이나 멘션 목록, 팔로워 목록 등을 제공하는 HTTP 기반의 API를 제공하고 있고, 네이버 또한 단축 URL을 제공하는 HTTP 기반의 API를 제공하고 있다. 이들 API들의 특징 중 하나는 응답으로 XML이나 JSON 형식을 사용한다는 점이다.  

스프링 MVC를 사용할 때, XML/JSON 응답 생성을 위한 뷰 클래스를 사용하거나 HttpServeltResponse를 이용해서 직접 원하는 응답을 생성할 수 있을 것이다. 하지만, 스프링 MVC는 좀 더 쉽게 XML과 JSON 형식을 처리하는 방법을 제공하고 있는데, 그것은 바로 &#64;RequestBody 애노테이션과 &#64;ResponseBody 애노테이션을 사용하는 것이다.  

#### 1.1. &#64;RequestBody/&#64;ResponseBody와 HttpMessageConverter
<br/>
웹 브라우저와 웹 서버 간에 데이터를 주고 받을 때 사용되는 HTTP 프로토콜은 헤더와 몸체로 구성되어 있다.  

요청 몸체에는 웹 브라이저에 전송할 데이터가 담기는데, 이 데이터의 형식은 Content-Type 헤더 값으로 지정한다. 응답도 동일하게 헤더와 몸체로 구성되며, Content-Type 헤더를 이용해서 몸체의 데이터 형식을 지정한다.  

스프링이 제공하는 &#64;RequestBody 애노테이션과 &#64;ResponseBody 애노테이션은 각각 요청 몸체와 응답 몸체와 관련되어 있다. 먼저 &#64;RequestBody 애노테시녕은 요청 몸체를 자바 객체로 변환할 때 사용된다. 예를 들어, 요청 파라미터 문자열을 String 자바 객체로 변환하거나, JSON 형식의 요청 몸체를 ㄹ자바 객체로 변환할 때 &#64;RequestBody 애노테이션을 사용한다. 비슷하게 &#64;ResponseBody 애노테이션은 자바 객체를 응답 몸체로 변환하기 위해 사용된다. 보통 자바 객체를 JSON 형식이나 XML 형식의 문자열로 변환할 때 &#64;ResponseBody를 사용한다.  

#### 1.2. HttpMessageConverter를 이용한 변환 처리
<br/>
스프링 MVC는 &#64;RequestBody 애노테이션이나 &#64;ResponseBody 애노테이션이 있으면, HttpMessageConverter를 이용해서 자바 객체와 Http 요청/응답 몸체 사이의 변환을 처리한다. 스프링은 다양한 타입을 위한 HttpMessageConverter 구현체를 제공하고 있다. 예를 들어, String 타입의 리턴 객체를 응답 몸체로 변환할 때에는 StringHttpMeesageConverter를 사용한다.  
&#60;mvc:annotation-driven&#62; 태그나 &#64;EnableWebMvc 애노테이션을 사용하면, StringHttpMessageConverter를 포함해 다수의 HttpMessageConverter 구현 클래스를 등록한다. 이들 HttpMessageConverter는 다음과 같다.  

+ StringHttpMessageConverter  
요청 몸체를 문자열로 변환하거나 문자열을 응답 몸체로 변환한다.
+ Jaxb2RootElementHttpMesssageConverter  
XML 요청 몸체를 자바 객체로 변환하거나 자바 객체를 XML 응답 몸체로 변환한다. 요청 컨텐트 타입으로 text/html, application/xml, application/&#42;+xml을 지원한다. 생성하는 응답 컨텐트 타입은 application/xml이다.
+ MappingJackson2HttpMessageConverter  
JSON 요청 몸체를 자바 객체로 변환하거나 자바 객체를 JSON 응답 몸체로 변환한다. Jackson2가 존재하는 경우에 사용된다. 지원하는 요청 컨텐트 타입은 application/json, application/&#42;+json이다.
+ MappingJacksonHttpMessageConverter  
JSON 요청 몸체를 자바 객체로 변환하거나 자바 객체를 JSON 응답 몸체로 변환한다. Jackson이 존재하는 경우에 사용된다. 지원하는 요청 컨텐트 타입은 application/json, application/&#42;+json이다.
+ ByteArrayHttpMessageConverter  
요청 몸체를 byte 배열로 변환하거나 byte 배열을 응답 몸체로 변환한다.
+ ResourceHttpMessageConverter  
요청 몸체를 스프링의 Resource로 변환하거나 Resource를 응답 몸체로 변환한다.
+ SourceHttpMessageConverter  
XML 요청 몸체를 XML Source로 변환하거나 XML Source를 XML 응답으로 변환한다.
+ AllEncompassingFormHttpMessageConverter  
폼 전송 형식의 요청 몸체를 MultiValueMap으로 변환하거나, MultiValueMap을 응답 몸체로 변환할 때 사용된다. 지원하는 요청 컨텐트 타입은 application/x-www-form-urlencoded, multipart/form-data이다. multipart/form-data 형식의 요청 몸체의 각 부분을 변환할 때에는 이 위의 HttpMessageConverter를 사용한다.

#### 1.3. JAXB2를 이용한 XML 처리
<br/>
JAXB2 API는 자바 객체와 XML 사이의 변호나을 처리해주는 API다. Jaxb2RootElementHttpMessageConverter는 JAXB2 API를 이용해서 자바 객체를 XML 응답으로 변환하거나 XML 요청 몸체를 자바 객체로 변환한다.  

JAXB2 API는 자바 6 이후 버전에 기본으로 포함되어 있으므로 메이븐 의존에 따로 추가할 필요가 없다.  

Jaxb2RootElementHttpMessageConverter는 다음의 변환 처리를 지원한다.
+ XML → &#64;XmlRootElement 객체 또는 &#64;XmlType 객체로 읽기
+ &#64;XmlRootElement 적용 객체 → XML로 쓰기  

따라서 XML 요청 몸체를 &#64;RequestBody 애노테이션을 이용해서 JAXB2 기반의 자바 객체로 변환하거나 JAXB2 기반 객체를 &#64;ResponseBody 애노테이션을 이용해서 XML 응답으로 변환하려면, Jaxb2RootElementHttpMessageConverter를 사용하면 된다. 앞서 MVC 설정을 사용하면 Jaxb2RootElementHttpMesssageConverter는 기본으로 등록되므로, 추가로 설정할 필요는 없다.  

&#64;XmlRootElement 애노테이션의 name 프로퍼티를 지정시 해당 이름으로 루트 태그를 가진 XML이 생성된다.

```java
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "", propOrder = {"id", "message", "creationTime"})
public class GuestMessage {
	
	private Integer id;
	private String message;
	@XmlJavaTypeAdapter(value=JAXB2DateConversionAdapter.class)
	private Date creationTime;
	
	public GuestMessage() {
	}

	public GuestMessage(Integer id, String message, Date creationTime) {
		super();
		this.id = id;
		this.message = message;
		this.creationTime = creationTime;
	}

	public Integer getId() {
		return id;
	}

	public String getMessage() {
		return message;
	}

	public Date getCreationTime() {
		return creationTime;
	}
}
```
<br/>
```java
public class JAXB2DateConversionAdapter extends XmlAdapter<String, Date>{

	private SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
	
	@Override
	public String marshal(Date date) throws Exception {
		// TODO Auto-generated method stub
		return simpleDateFormat.format(date);
	}

	@Override
	public Date unmarshal(String date) throws Exception {
		// TODO Auto-generated method stub
		return simpleDateFormat.parse(date);
	}
	
}
```

Date 타입의 경우 XmlAdapter를 상속한 클래스를 통해 String marshal(Date date), Date unmarshal(String date) 메서드를 오버라이드 하여 구현한 후 해당 Xml 해당하는 클래스의 필드에 @XmlJavaTypeAdapter(value=XmlAdpater구현클래스명.class) 애노테이션을 추가하면 정상적으로 변환한다.
nested exception is com.sun.xml.internal.bind.v2.runtime.illegalannotationsexception: 2 counts of illegalannotationexceptions

xml을 자바 객체로 변환할 때 맵핑되는 타겟 필드 혹은 메서드가 두 개일 때 발생하는 오류이다. 해당하는 필드나 메서드를 지워주면 해결된다.

웹 브라우저 별로 XML이나 JSON 형식의 데이터를 서버에 전송하기 위한 플러그인이 존재한다. 예를 들어, 크롬의 경우 Advanced Rest Client 확장 프로그램을 사용하면 자바스크립트 코드를 작성할 필요 없이 서버에 XML이나 JSON 형식의 데이터를 전송할 수 있다.

#### 1.4. Jackson2를 이용한 JSON 처리
<br/>
Jackson2는 자바 객체를 JSON으로 변환하거나 JSON을 자바 객체로 변환할 때 주로 사용하는 라이브러리로, 스프링 MVC의 MappingJackson2HttpMessageConverter는 Jackson2를 이용해서 자바 객체와 JSON 간의 변환을 처리한다.  

Jackson2를 사용하려면 먼저 메이븐 의존 설정에 Jackson2 의존을 추가해주어야 한다.  

```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.3.3</version>
</dependency>
```
<br/>
Jackson2 의존을 추가하면 필요한 설정은 끝이다. 자바 객체를 JSON 응답으로 변환하는 것은 매우 간단하다. 컨트롤러 메서드의 리턴 타입으로 자바 객체를 리턴해주기만 하면 된다.  

#### 1.5. 커스텀 HttpMessageConverter 등록하기
<br/>
스프링은 이미 널리 사용되는 요청/응답 형식을 위한 HttpMessageConverter를 제공하고 있기 때문에, 직접 HttpMessageConverter를 구현해야 하는 경우는 극히 드물다. 만약 직접 구현한 HttpMessageConverter를 추가하고 싶다면 다음과 같이 &#60;mvc:message-converters&#62; 태그에 사용할 HttpMessageConverter 빈을 등록해주면 된다.  

```xml
<mvc:annotation-driven>
	<mvc:message-converters register-defaults="false">
		<bean class="커스텀HttpMessageConverter의완전클래스명" />
	</mvc:message-converters>
</mvc:annotation-driven>
```
<br/>
위 설정을 사용하면 스프링 MVC는 CustomMessageConverter를 등록하고, 기본으로 사용하는 HttpMessageConverter를 등록한다.  

만약 기본으로 추가되는 HttpMessageConverter를 등록하고 싶지 않다면, 다음과 같이 register-defaults 속성 값을  false로 지정하면 된다.  

&#64;EnableWebMvc 애노테이션을 이용하는 경우, 다음과 같이 WebMvcConfigurerAdapter 클래스를 상속받아 configureMessageConverters() 메서드를 재정의하면 된다.  

```java
@Configuration
@EnableWebMvc
public class SampleConfig extends WebMvcConfigurerAdapter {

	@Override
	public void configureMessageConverters(List<HttpMessageConverters<?> converters) {
		converters.add(new CustomMessageConverter());
	}
}
```
<br/>
위와 같이 커스텀 HttpMessageConverter를 등록할 때 주의할 점은, &#60;mvc:annotation-driven&#62; 태그를 사용할 때와 달리 위와 같이 커스텀 HttpMessageConverter를 등록하면 기본으로 등록했던 HttpMessageConverter를 등록하지 않는다는 점이다. 따라서, 위와 같이 커스텀 HttpMessageConverter를 등록하고, 더불어 XML이나 JSON 변환 처리를 하고 싶다면 나머지 HttpMessageConverter도 함께 등록해주어야 한다.

### 2. 파일 업로드
<br/>
#### 2.1. MultipartResolver 설정
<br/>
멀티파티 지원 기능을 사용하려면 먼저 MultipartResolver를 스프링 설정 파일에 등록해주어야 한다. MultipartResolver는 멀티파트 형식으로 데이터가 전송된 경우, 해당 데이터를 스프링 MVC에서 사용할 수 있도록 변환해주는 역할을 한다. 예를 들어, &#64;RequestParam 애노테이션을 이용해서 멀티파트로 전송된 파라미터 값과 파일 데이터를 사용할 수 있도록 해준다.  

스프링이 기본으로 제공하는 MultipartResolver는 다음의 두 개가 있다.
+ org.springframework.web.multipart.commons.CommonsMultipartResolver  
Commons FileUpload API를 이용해서 멀티파트 데이터를 처리한다.
+ org.springframework.web.multipart.support.StandardServletMultipartResolver  
서블릿 3.0의 Part를 이용해서 멀티파트 데이터를 처리한다.  

위 두 MultipartResolver 구현체 중 하나를 스프링 빈으로 등록해주면 된다. 이 때 주의할 점은 스프링 빈의 이름은 "multipartResolver"이어야 한다는 점이다. DispatcherServlet은 이름이 "multipartResolver"인 빈을 사용하기 때문에, 다른 이름을 사용할 경우 MultipartResolver로 사용되지 않는다.  

##### 2.1.1. Commons FileUpload를 이용하기 위한 설정
<br/>
CommonsMultipartResolver는 Commons FileUpload API를 이용해서 Multipart를 처리해준다. CommonsMultipartResolver를 MultipartResolver로 사용하려면 다음과 같이 등록하면 된다.  

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
```
<br/>
CommonsMultipartResolver 클래스는 업로드와 관련해서 다음과 같은 프로퍼티를 제공하고 있다.
<br/>
<table>
	<thead>
		<tr>
			<td>프로퍼티</td>
			<td>타입</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>maxUploadSize</td>
			<td>long</td>
			<td>최대 업로드 가능한 바이트 크기. -1은 제한이 없음을 의미한다. 기본 값은 -1이다.</td>
		</tr>
		<tr>
			<td>maxInMemorySize</td>
			<td>int</td>
			<td>디스크에 임시 파일을 생성하기 전에 메모리에 보관할 수 있는 최대 바이트 크기. 기본 값은 10240 바이트이다.</td>
		</tr>
		<tr>
			<td>defaultEncoding</td>
			<td>String</td>
			<td>요청을 파싱할 때 사용할 캐릭터 인코딩. 지정하지 않을 경우, HttpServletRequest.setCharacterEncoding() 메서드로 지정한 캐릭터 셋이 사용된다. 아무 값도 없을 경우 ISO-8859-1을 사용한다.</td>
		</tr>
	</tbody>
</table>
<br/>
CommonsMultipartResolver는 Commons FileUpload API를 사용하기 때문에, Commons FileUpload 라이브러리를 클래스패스에 추가해주어야 한다. 메이븐을 사용할 경우 다음과 같은 의존을 추가해주면 된다.  

```xml
<dependency>
	<groupId>commons-fileupload</groupId>
	<artifactId>commons-fileupload</groupId>
	<version>1.3</version>
</dependency>
```

##### 2.1.2. 서블릿 3의 파일 업로드 기능 사용을 위한 설정
<br/>
서블릿 3의 파일 업로드 기능을 사용하려면 다음의 설정이 필요하다.
+ DispatcherServlet이 서블릿 3의 Multipart를 처리하도록 설정
+ StandardServletMultipartResolver 클래스를 MultipartResolver로 설정  
먼저, 서블릿 3의 파일 업로드 기능을 사용하려면 &#60;multipart-config&#62; 태그를 이용해서 DispatcherServlet이 멀티파트를 처리할 수 있도록 설정해주어야 한다. 다음은 설정 예이다.

```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
		http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">

	<servlet>
		<servlet-name>dispatcher</servlet-name>
		<servlet-class>
			org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>
				/WEB-INF/mvc-quick-start.xml
			</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
		<multipart-config>
			<location>C:\temp</location>
			<max-file-size>1</max-file-size>
			<max-request-size>-1</max-request-size>
			<file-size-threshold>1024</file-size-threshold>
		</multipart-config>
	</servlet>
```
&#60;multipart-config&#62;의 설정 관련 태그를 다음에 설명했다. 각 태그는 필수가 아니며, 필요한 태그만 선택해서 사용하면 된다.  
+ &#60;location&#62;  
업로드 한 파일이 임시로 저장될 위치를 지정한다.
+ &#60;max-file-size&#62;  
업로드 가능한 파일의 최대 크기를 바이트 단위로 지정한다. -1은 제한이 없음을 의미하며 기본값은 -1이다.
+ &#60;file-size-threshold&#62;  
업로드 한 파일 크기가 이 태그의 값보다 크면 &#60;location&#62;에서 지정한 디렉토리에 임시로 파일을 생성한다. 업로드 파일 크기가 이 태그의 값 이하면 메모리에 파일 데이터를 보관한다. 단위는 바이트이며, 기본 값은 0이다.
+ &#60;max-request-size&#62;  
전체 Multipart 요청 데이터의 최대 제한 크기를 바이트 단위로 지정한다. -1은 제한이 없음을 의미하며, 기본 값은 -1이다.  

DispatcherServlet이 서블릿 3의 파일 업로드 기능을 사용할 수 있도록 &#60;multipart-config&#62;를 설정했다면, 스프링 설정에서 StandardServletMultipartResolver를 빈으로 등록해주면 된다.  

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver"/>
```
<br/>
파일 최대 크기나 임시 저장 디렉토리 등은 &#60;multipart-config&#62; 태그를 이용해서 설정하기 때문에, StandardServletMultipartResolver는 별도 프로퍼티 설정을 제공하지 않는다.  

서블릿 3 버전부터 자바 클래스를 이용해서 서블릿을 설정하는 방법이 추가됐는데, DispatcherServlet도 마찬가지로 web.xml이 아닌 자바 코드를 이용해서 설정할 수 있게 되었다. 즉, 자바 코드를 이용해서 서블릿 3의 파일 업로드 관련 부분을 설정할 수 있다.  

StandardServletMultipartResolver는 서블릿 3 버전 이상만 지원하므로, 톰캣 6 버전과 같이 서블릿 2.5 또는 그 이하를 지원하는 웹 컨테이너에서는 올바르게 동작하지 않는다.

#### 2.2. 업로드한 파일 접근하기
<br/>
##### 2.2.1. MultipartFile 인터페이스 사용
<br/>
org,springframework.web.multipart.MultipartFile 인터페이스는 업로드 한 파일 정보 및 파일 데이터를 표현하기 위한 용도로 사용되며,  이 인터페이스를 이용해서 업로드한 파일 데이터를 읽을 수 있다. MultipartFile 인터페이스가 제공하는 주요 인터페이스는 다음과 같다.  
+ String getName()  
파라미터 이름을 구한다.
+ String getOriginalFilename()  
업로드한 파일의 이름을 구한다.  
+ boolean isEmpty()  
업로드한 파일이 존재하지 않는 경우 true를 리턴한다.
+ long getSize()  
업로드한 파일의 크기를 구한다.  
+ byte[] getBytes() throws IOException  
업로드한 파일 데이터를 구한다.
+ InputStream getInputStream() throws IOException  
업로드한 파일 데이터를 읽어오는 InputStream을 구한다. InputStream의 사용이 끝나면 알맞게 종료해주어야 한다.
+ void trasferTo(File dest) throws IOException  
업로드한 파일 데이터를 지정한 파일에 저장한다.  

업로드한 파일 데이터를 구하는 가장 단순한 방법은 MultipartFile.getBytes() 메서드를 이용하는 것이다. 바이트 배열을 구한 뒤에 파일이나 DB 등에 저장하면 된다.  

MultipartFile.transferTo() 메서드를 사용하면 좀 더 간결하게 업로드한 파일을 지정한 파일에 저장할 수 있다.  

##### 2.2.2. &#64;RequestParam 애노테이션을 이용한 업로드 파일 설정
<br/>
업로드한 파일을 전달받는 첫 번째 방법은 &#64;RequestParam 애노테이션이 적용된 MultipartFile 타입의 파라미터를 사용하는 것이다.  

MultipartFile 인터페이스는 스프링에서 업로드한 파일을 표현할 때 사용되는 인터페이스로서, MultipartFile 인터페이스를 이용해서 업로드한 파일의 이름, 실제 데이터, 파일 크기 등을 구할 수 있다.

##### 2.2.3. MultipartHttpServletRequest를 이용한 업로드 파일 접근
<br/>
업로드한 파일을 전달받는 두 번째 방법은 MultipartHttpServletRequest 인터페이스를 사용하는 것이다.  

MultipartHttpServletRequest 인터페이스는 스프링이 제공하는 인터페이스로서, 멀티파트 요청이 들어올 때 내부적으로 원본 HttpServletRequest 대신 사용되는 인터페이스이다. MultipartHttpServlet 인터페이스는 실제로는 어떤 메서드도 선언하고 있지 않으며, HttpServletRequest 인터페이스와 MultipartRequest 인터페이스를 상속받고 있다.  

MultipartHttpServletRequest 인터페이스는 javax.servlet.HttpServletRequest 인터페이스를 상속받기 때문에 웹 요청 정보를 구하기 위한 getParameter()나 getHeader()와 같은 메서드를 사용할 수 있으며, 추가로 MultipartRequest 인터페이스가 제공하는 멀티파트 관련 메서드를 사용할 수 있다.  

MultipartRequest 인터페이스가 제공하는 업로드 파일 관련 주요 메서드는 다음과 같다.
+ Iterator&#60;String&#62; getFileNames()  
업로드 된 파일들의 파라미터 이름 목록을 제공하는 Iterator를 구한다.  
+ MultipartFile getFile(String name)  
파라미터 이름이 name인 업로드 파일 정보를 구한다.
+ List&#60;MultipartFile&#62; getFile(String name)  
파라미터 이름이 name인 업로드 파일 정보 목록을 구한다.
+ Map&#60;String, MultipartFile&#62; getFileMap()  
파라미터 이름을 키로 파라미터에 해당하는 파일 정보를 값으로 하는 Map을 구한다.

##### 2.2.4. 커맨드 객체를 통한 업로드 파일 접근
<br/>

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
```

bean 설정 파일에 위와 같이 설정한다.  

```xml
<dependency>
	<groupId>commons-fileupload</groupId>
	<artifactId>commons-fileupload</groupId>
	<version>1.3</version>
</dependency>
```

메이븐 의존설정파일에 위와 같이 설정한다.  

커맨드 객체를 이용해도 업로드한 파일을 전달받을 수 있다. 단지 커맨드 클래스에 파라미터와 동일한 이름의 MultipartFile 타입 프로퍼티를 추가해주기만 하면 된다.

##### 2.2.5. 서블릿 3의 Part 사용하기
<br/>
서블릿 3의 파일 업로드 기능을 사용했다면, 서블릿 3에 추가된 javax.servlet.http.Part 타입을 이용해서 업로드한 파일을 처리할 수 있다.  

### 3. 웹소켓 서버 구현 지원
<br/>
HTML5의 주요 API 중 하나인 웹소켓(WebSocket)은 HTTP 프로토콜을 기반으로 웹브라우저와 웹 서버 간 양방향 통신을 지원하기 위한 표준이다. 웹소켓을 사용하면 마치 소켓을 사용하는 것처럼 클라이언트와 서버가 메시지를 자유롭게 주고 받을 수 있다. 이런 이유로 실시간 알림, 채팅, 웹 기반의 실시간 협업 도구와 같이 클라이언트와 서버 간에 메시지를 빈번하게 주고 받는 웹 어플리케이션을 개발할 때 웹소켓을 적용하는 사례가 증가하고 있다.  

자바의 웹소켓 표준인 JSR-356 표준에 맞춰 웹소켓 서버 기능을 구현할 경우 스프링의 DispatcherServlet의 연동이나 스프링 빈 객체를 사용하기가 매우 번거롭다. 스프링은 이런 번거로움을 줄여주기 위한 기반 클래스를 제공하고 있으며, 이를 통해 컨트롤러를 구현하는 것과 비슷한 방식으로 서버를 구현할 수 있게 된다.  

#### 3.1. 메이븐 의존 설정
<br/>
스프링 4의 웹소켓 기능을 사용하려면, 메이븐 의존에 웹소켓 모듈 설정을 추가한다. 또한, 스프링 4의 웹소켓 모듈은 서블릿 3의 웹소켓 기능에 의존하고 있으므로, 서블릿 3을 지원하는 컨테이너에서만 이 기능을 사용할 수 있다.  

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-websocket</artifactId>
	<version>4.0.4.RELEASE</version>
</dependency>
```

#### 3.2. WebSocketHandler를 이용한 웹소켓 서버 구현
<br/>
스프링 웹소켓 기능은 스프링 MVC를 지원하기 때문에, 스프링 MVC 환경에서 간단한 설정만으로 웹소켓 서버 프로그램을 구현할 수 있다. 스프링 웹소켓을 이용해서 웹소켓 서버를 구현하려면 다음의 두 가지만 하면 된다.
+ WebSocketHandler 인터페이스를 구현한다.
+ &#60;websocket:handlers&#62; &#64;EnableWebSocket 애노테이션을 이용해서 앞서 구현한 WebSocketHandler 구현 객체를 웹소켓 엔드포인트로 등록한다.  

먼저 할 작업은 WebSocketHandler 인터페이스를 구현하는 것이다. 사실 스프링 웹소켓을 이용해서 웹소켓 서버를 구현할 때 개발자가 직접 구현하는 분분은 이것뿐이다. 스프링 웹소켓 모듈은 웹소켓 클라이언트가 연결되거나 데이터를 보내거나 연결을 끊는 경우 WebSocketHandler에 관련 데이터를 전달한다. 예를 들어, 웹소켓 클라이언트가 특정 엔드포인트로 연결하면, 웹소켓 모듈은 엔드포인트에 매핑된 WebSocketHandler의 afterConnectionEstablished() 메서드를 호출한다. 비슷하게 웹소켓 클라이언트가 데이터를 전송하면 WebSocketHnadler의 handleMessage()를 호출해서 클라이언트가 전송한 데이터를 전달한다. 실제 스프링 웹소켓 모듈은 복잡한 과정을 거쳐서 WebSocketHandler의 메서드를 실행하지만, 개발자 입장에서는 WebSocketHandler만 알맞게 구현하면 웹소켓 서버를 만들 수 있다.  

웹소켓 서버를 구현할 때 WebSocketHandler 인터페이스를 직접 상속받기 보다는 기본 구현을 일부 제공하고 있는 AbstractWebSocketHandler나 TextWebSocketHandler 클래스를 상속받아 구현하게 된다. TextWebSocketHandler 클래스는 텍스트 데이터를 주고 받을 때 상속받아 사용할 수 있는 기반 클래스이다.  

afaterConnectionEstablished()와 afterConnectionClosed()는 각각 웹소켓 클라이언트와 연결되거나 연결이 종료될 때 호출된다. 두 메서드의 첫 번째 파라미터인 WebSocketSession은 클라이언트와의 세션을 관리하는 객체이다.  

handleTextMessage() 메서드는 웹소켓 클라이언트가 텍스트 메시지를 전송할 때 호출되는 메서드이다. TextMessage 타입의 두 번째 파라미터인 message는 클라이언트가 전송한 텍스트 데이터를 담고 있으며, getPayload() 메서드를 이용해서 텍스트 데이터를 구할 수 있다.  

WebSocketSession의 sendMessage() 메서드는 웹소켓 클라이언트에게 데이터를 전송한다.  

웹소켓 서버를 위한 WebSocketHandler 구현 클래스를 만들었다면, 스프링 설정을 이용해서 웹소켓 기능을 활성화하면 된다, XML을 사용하는 경우 다음과 같이 websocket 네임스페이스를 이용해서 웹소켓 서버를 설정하면 된다.  

```xml
<beans xmlns="http://www,springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:websocket="http://www.springframework.org/schema/websocket"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
				http://www.springframework.org/schema/mvc
				http://www.springframework.org/schema/mvc/spring-mvc.xsd
				http://www.springframework.org/schema/websocket
				http://www.springframework.org/schema/websocket/spring-websocket.xsd">

	<websocket:handlers>
		<websocket:mapping handler="빈식별자" path="웹소켓 클라이언트가 사용할 엔드포인트 경로명" />
	</websocket:handlers>

	<bean id="빈식별자" class="WebSocketHandler를구현한완전환클래스명" />

	<mvc:default-servlet-handler />

	...

</beans>
```

&#60;websocket:mapping&#62;은 웹소켓 클라이언트가 연결할 때 사용할 엔드포인트(path 속성)와 WebSocketHandler 객체를 연결해준다. 위 설정의 경우, 웹소켓 클라이언트가 path에 설정된 경로로 접속하면 WebSocketHandler를 구현한 클래스의 빈을 이용해서 처리한다고 설정하고 있다. 
해당 path에 확장자 또는 경로가 servlet-mapping에 부합하는지 확인하여야 한다.  

&#60;websocket:handlers&#62;는 내부적으로 스프링 MVC의 SimpleUrlHandlerMapping을 포함해 몇 개의 빈을 등록해준다. 따라서, 이를 사용하려면 web.xml에 DispatcherServlet 설정을 추가해주어야 한다.  

##### 3.2.1. WebSocketHandler 인터페이스, WebSocketMessage 클래스와 관련 클래스
<br/>
org.springframework.web.socket.WebSocketHandler 인터페이스에 정의된 메서드는 다음과 같다.
+ void afterConnectionEstablished(WebSocketSession session) throws IOException  
웹소켓 클라이언트가 연결되면 호출된다.
+ void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception  
웹소켓 클라이언트가 데이터를 전송하면 호출된다. message는 클라이언트가 전송한 데이터를 담고 있다.
+ void handleTransportError(WebSocketSession session, Throwable exception) throws Exception  
웹소켓 클라이언트와의 연결에 문제가 발생하면 호출된다.
+ void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception  
웹소켓 클라이언트가 연결을 직접 끊거나 서버에서 타임아웃이 발생해서 연결을 끊을 때 호출된다.
+ boolean supportPartiaMessages()  
큰 데이터를 나눠서 받을 수 있는지 여부를 지정한다. 이 값이 true고 웹소켓 컨테이너(톰캣이나 Jetty 등)가 부분 메시지를 지원할 경우, 데이터가 크거나 미리 데이터의 크기를 알 수 없을 때 handleMessage()를 여러 번 호출해서 데이터를 부분적으로 전달한다.  

WebSocketHandler 인터페이스를 상속받아 모든 메서드를 구현하기 보다는 AbstractWebSocketHandler 클래스를 상속받아 필요한 메서드만 구현하기 보다는 AbstractWebSocketHandler 클래스를 상속받아 필요한 메서드만 구현하는 것이 보통이다.  

AbstractWebSocketHandler 클래스의 handleMessage() 메서드는 WebSocketMessage의 타입에 따라 다음의 세 메서드 중 하나를 호출한다. (모두 protected void 이며, throws Exception을 생략하였다.)
+ handleTextMessage(WebSocketSession session, TextMessage message)
+ handleBinaryMessage(WebSocketSession session, BinaryMessage message)
+ handlePongMessage(WebSocketSession session, PongMessage message)  

AbstractWebSocketHandler를 상속받은 하위 클래스는 처리하려는 메시지 타입에 따라 알맞은 메서드를 재정의하면 된다.  

TextWebSocketHandler 클래스는 handleBinaryMessage() 메서드가 익셉션을 발생하도록 재정의하고 있으며, 이를 통해 텍스트 메시지만을 처리하도록 제한하고 있다. BinaryWebSocketHandler 클래스는 유사한 방법으로 바이너리 메시지만 처리하도록 제한하고 있다.  

스프링 웹소켓은 주고 받는 데이터를 담기 위해 org.springframework.web.socket.WebSocketMessage 인터페이스를 사용한다. WebSocketMessage 인터페이스는 다음과 같이 정의되어 있다.  

```java
public interface WebSocketMessage<T> {
	T getPayload();
	boolean isLast();
}
```

WebSocketMessage의 하위 타입은 다음과 같이 두 개가 존재한다.
+ TextMessage : 텍스트를 담는 메시지로 String 타입 데이터를 담는다.
+ BinaryMessage : 바이트를 담는 메시지로 java.nio.ByteBuffer 타입 데이터를 담는다.  

BinaryMessage는 ByteBuffer 또는 byte 배열을 이용해서 객체를 생성할 수 있다.  

WebSocketMessage의 하위 타입에는 PintMessage와 PongMessage도 존재한다. 이 두 클래스는 서버와 클라이언트 간 연결을 유지하거나 확인하기 위해 사용되는 메시지이다.  

org.springframework.web.socket.WebSocketSession 인터페이스는 웹소켓 클라이언트의 세션을 표현한다. WebSocketSession 인터페이스는 sendMessage() 메서드와 같이 웹소켓 클라이언트와 통신을 하는데 필요한 기능을 제공하고 있으며, 주요 메서드는 다음과 같다.  

+ String getId() : 세션ID를 리턴한다.
+ URI getUri() : 엔드포인트 경로를 리턴한다.
+ InetSocketAddress getLocalAddress() : 로컬 서버 주소를 리턴한다.
+ InetSocketAddress getRemoteAddress() : 클라이언트 주소를 리턴한다.
+ boolean isOpen() : 소켓이 열려 있는지 여부를 리턴한다.
+ sendMessage(WebSocketMessage<?> message) throws IOException : 메시지를 전송한다.
+ void close() throws IOException : 소켓을 종료한다.

##### 3.2.2. 웹소켓 적용시 고려사항
<br/>
웹소켓 서비를 지원하는 컨테이너에 따라 서버의 동작 방식이 일부 다를 수 있다. 예를 들어, Tomcat 7.0.52 버전은 웹소켓 클라이언트가 연결되면, 직접 연결을 종료하기 전까지 연결을 유지한다. 반면에 Jetty 9 버전의 경우 Ping/Pong 메시지를 이용해서 클라이언트가 연결을 유지하고 있는지 확인한다. 따라서, Jetty는 연결이 정상이 아닌 것으로 간주하고 마지막 메시지 송수신 이후 일정 시간(기본 300초)이 지나면 연결을 끊는다. 문제는 웹 브라우저마다 Ping/Pong을 지원할지 여부가 다르고, Ping/Pong을 지원하더라도 스프링에서 제대로 처리하지 못하는 경우가 있다는 점이다. 따라서, 이를 고려해서 웹컨테이너를 선택하고 클라이언트 코드는 연결이 끊기면 재연결을 하는 등의 처리가 필요하다.  

웹소켓을 사용할 때 또 하나 고려할 점은 웹소켓 서버와 클라이언트 사이에 위치한 다양한 네트워크 장비들(프록시, L4 등)이 아직 웹소켓 프로토콜을 완벽하게 지원하는 것은 아니라는 점이다. 예를 들어 프록시는 길게 연결된 HTTP 연결을 끊거나 중간에 데이터를 일부 버퍼링할 수도 있는데, 이는 연결된 상태에서 즉각적인 응답을 주고 받아야 하는 웹소켓 프로그래밍에서 문제가 된다.

#### 3.3. SockJS 지원
<br/>
웹소켓이 웹 브라우저에 적용되기 이전에 클라이언트와 서버 간에 데이터를 주고 받기 위한 다양한 기법이 존재했는데, 이런 기법들 역시 브라우저 종류와 버전마다 다르게 적용해야 했다. 예를 들어, iframe을 사용하기도 하고 Long-Polling을 사용하기도 했다. 또한 웹소켓을 지원하면 직접 웹소켓을 쓰기도 한다. 문제는 각 방식에 따라 자바 스크립트 코드와 서버 코드를 작성해야 했는데, 이런 불편함을 해소하기 위해 만들어진 것이 바로 SockJS이다.

스프링 웹소켓 모듈을 사용하면 SockJS 서버를 만들 수 있다. 톰캣 6 버전처럼 웹소켓을 지원하지 않는 서버에서 양방향 통신을 처리하고 싶을 때에는 스프링의 SockJS 서버 지원을 이용해서 SockJS 클라이언트와의 연결을 처리할 수 있게 된다.
##### 3.3.1. SockJS란?
<br/>
SockJS는 이런 다양한 우회 기법들을 추상화해서 웹소켓과 유사한 API로 웹 서버와 웹 브라우저가 통신할 수 있도록 해준다. 실제로 SockJS가 제공하는 클라이언트 API를 이용해서 작성한 코드는 웹소켓 API를 이용해서 작성한 코드와 거의 동일하다.  

SockJS는 다양한 환경을 위한 서버 모듈과 클라이언트 모듈을 제공하고 있다. 예를 들어, sockjs-clients는 웹 브라우저를 위한 자바 스크립트 모듈로서 브라우저 환경에 상관없이 SockJS API를 이용해서 서버와 클라이언트가 양방향 통신을 할 수 있도록 만들어준다. 실제 SockJS가 내부적으로 웹소켓, Long-Polling 등을 사용할지 여부는 몰라도 된다.  

SockJS 클라이언트와 통신할 수 있는 SockJS 서버는 웹소켓, Long-Polling 등 SockJS 클라이언트가 사용하는 다양한 방식을 지원한다. 현재 SockJS를 지원하는 서버로는 Node.js, 파이썬, 자바 Vert.x 서버 등이 있으며, 이들 서브를 사용하면 단일 서버 API를 이용해서 SockJS 클라이언트와 양방향 통신하는 서버를 만들 수 있다.
