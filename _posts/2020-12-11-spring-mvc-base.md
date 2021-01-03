---
title: 스프링 MVC - 기본기
categories:
- Spring
feature_text: |
  ## 스프링 MVC : 기본기
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	td { border: 1px solid #444444;}
</style>
### 1. 스프링 MVC 일단 해보기
<br/><br/>
### 2. 기본 흐름과 주요 컴포넌트
<br/>
스프링 MVC는 여러 구성 요소가 맞물려 동작하기 때문에, 스프링 MVC를 이요해서 웹 어플리케이션을 개발하려면 적어도 스프링 MVC가 어떤 식으로 동작하는지 이해하고 있어야 한다. 이에 대한 이해가 부족하면 문제를 해결하는 좋은 방법을 찾을 수 없게 된다.

#### 2.1. 스프링 MVC의 주요 구성 요소
+ DispatcherServlet  
클라이언트의 요청을 전달받는다. 컨트롤러에게 클라이언트의 요청을 전달하고, 컨트롤러가 리턴한 결과값을 View에 전달하여 알맞은 응답을 생성하도록 한다.  
+ HandlerMapping  
클라이언트의 요청 URL을 어떤 컨트롤러가 처리할지를 결정한다.  
+ HandlerAdapter  
DispatcherServlet의 처리 요청을 변환해서 컨트롤러에게 전달하고, 컨트롤러의 응답 결과를 DispatcherServlet이 요구하는 형식으로 변환한다. 웹 브라우저 캐시 등의 설정도 담당한다.  
+ 컨트롤러(Controller)  
클라이언트의 요청을 처리한 뒤, 결과를 리턴한다. 응답 결과에서 보여줄 데이터를 모델에 담아 전달한다.  
+ ModelAndView  
컨트롤러의 처리 결과를 보여줄 뷰를 결정한다.  
+ 뷰(View)  
컨트롤러의 처리 결과 화면을 생성한다. JSP나 Velocity 템플릿 파일 등을 이용해서 클라이언트에 응답 결과를 전송한다.

### 3. 스프링 MVC 설정 기초
<br/>
스프링 MVC를 이용해서 웹 어플리케이션을 개발할 때 가장 먼저 해야 할 일은 스프링 MVC 설정을 작성하는 것이다. 기본 설정을 완료하면 본격적인 스프링 MVC를 이용한 웹 개발에 진입하게 된다.  

스프링 MVC를 사용하기 위한 기본 설정 과정은 다음과 같다.  

(1) web.xml에 DispatcherServlet 설정  
(2) web.xml에 캐릭터 인코딩 처리를 위한 필터 설정  
(3) 스프링 MVC 설정  
	A. HandlerMapping, HandlerAdapter 설정  
	B. ViewResolver 설정  

#### 3.1. DispatcherServlet 서블릿 설정
<br/>
DispatcherServlet은 스프링 MVC 프레임워크의 중심이 되는 서블릿 클래스이다. 앞서 처리 흐름에서 보았듯이 웹 브라우저의 요청을 DispatcherServlet이 받게 되며, DispatcherServlet은 관련 컴포넌트를 이용해서 웹 브라우저의 요청을 처리한 뒤 결과를 정송하게 된다.  

DispatcherServlet은 내부적으로 스프링 컨테이너를 생성한다. 별도의 초기화 파라미터없이 DispatcherServlet을 설정하면, 웹 어플리케이션의 /WEB-INF/ 디렉터리에 위치한 [서블릿이름]-servlet.xml 파일을 스프링 설정 파일로 사용한다.


한 개 이상의 설정 파일을 사용해야 하거나 또는 이름이 [이름]-servlet.xml 형식이 아닌 파일을 사용해야 한다면, 다음과 같이 contextConfigLocation 초기화 파라미터로 파일 목을 지정하면 된다.
```xml
<servlet>
	<servlet-name>dispatcher</servlet-name>
	<servlet-class>
		org.springframework.web.servlet.DispatcherServlet
	</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
			/WEB-INF/main.xml
			/WEB-INF/bbs.xml
			classpath:/common.xml
		</param-value>
	</init-param>
</servlet>
```

contextConfigLocation 초기화 파라미터는 설정 파일의 목록을 값으로 갖는데, 이때 각 설정 파일은 콤마(","), 공백(" "), 줄 바꿈(\n), 세미콜런(";")을 이용하여 구분한다. 각 설정 파일의 경로는 웹 어플리케이션 루트 디렉터리를 기준으로 하며, "file:"이나 "classpath:" 접두어를 이용해서 로컬 파일이나 클래스패스에 위치한 파일을 사용할 수 있다.  

XML 설정 파일이 아닌 &#64;Configuration 클래스를 이용해서 설정 정보를 작성했다면, 다음과 같이 contextClass를 추가 설정해주어야 한다.

```xml
<servlet>
	<servlet-name>dispatcher</servlet-name>
	<servlet-class>
		org.springframework.web.servlet.DispatcherServlet
	</servlet-class>
	<init-param>
		<param-name>contextClass</param-name>
		<param-value>
			org.springframework.web.context.support.AnnotationConfigWebApplicationContext
		</param-value>
	</init-param>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
			완전한 스프링설정 클래스명
		</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>dispatcherConfig</servlet-name>
	<url-pattern>/config/*</url-pattern>
</servlet-mapping>
```

contextClass 초기화 파라미터는 DispatcherServlet이 스프링 컨테이너를 생성할 때 사용할 구현 클래스를 지정한다. 이 값을 지정하지 않으면 XmlWebApplicationContext를 사용하는데, 이 클래스는 XML 설정 파일을 사용한다. 따라서, &#64;Configuration 기반의 자바 설정을 이용하는 경우 AnnotationConfigWebApplicationContext 클래스를 사용하도록 contextClass 초기화 파라미터의 값을 지정해주어야 한다.  

contextConfigLocation 초기화 파라미터의 값은 XML 설정 파일의 경로 대신 &#64;Configuration 자바 클래스의 완전한 이름을 지정한다. 두 개 이상인 경우 콤마(","), 공백(" "), 탭(\t), 줄 바꿈(\n), 세미콜론(";")을 이용해서 구분한다.

##### 3.1.1. 캐릭터 인코딩 필터 설정
<br/>
서블릿/JSP 프로그래밍에서 웹 브라우저가 전송한 요청 파라미터를 올바르게 처리하려면, 알맞은 캐릭터 인코딩을 지정해주어야 한다. 스프링은 요청 파라미터의 개릭터 인코딩을 지정할 수 있는 서블릿 필터(CharacterEncodingFilter)를 제공하고 있으므로, 요청 파라미터의 캐릭터 인코딩 처리를 위해 별도의 코드를 작성할 필요가 없다. 다음은 이 필터의 설정 방법이다.

```xml
<!-- web.xml -->
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
```

#### 3.2. 스프링 MVC 설정 기초
<br/>
스프링 MVC를 설정하려면 최소한 다으므이 구성 요소를 빈 객체로 등록해주어야 한다.
+ HandlerMapping 구현 객체
+ HandlerAdapter 구현 객체
+ ViewResolver 구현 객체  

이 중에서 HandlerMapping과 HandlerAdapter는 다음과 같이 &#60;mvc:annotation-driven&#62; 태그를 이용하면 설정이 끝난다. 따라서, ViewResolver만 추가로 설정해주면 된다.

&#60;mvc:annotation-driven&#62; 태그는  다음의 두 클래스를 빈으로 등록해준다.
+ RequestMappingHandlerMapping
+ RequestMappingHandlerAdapter  

이 두 클래스는 &#64;Controller 애노테이션이 적용되 클래스를 컨트롤러로 사용할 수 있도록 해준다. 이 두 객체 외에 &#60;mvc:annotation-driven&#62; 태그는 JSON이나 XML 등 요청/응답 처리를 위해 필요한 변환 모듈이나 데이터 바인딩 처리를 위한 ConversionService 등을 빈으로 등록해준다.  

위 설정에서 InternalResourceViewResolver는 JSP를 이용해서 뷰를 생성할 때 사용되는 ViewResolver 구현체이다. ViewResolver를 지정할 때 주의할 점은 ViewResolver의 이름이 "viewResolver"여야 한다는 점이다. prefix와 suffix는 컨트롤러의 처리 결과를 보여줄 JSP의 경로를 생설할 때 사용된다.  

&#64;Configuration 자바 설정을 사용한다면, &#64;EnableWebMvc 애노테이션을 사용하면 된다. &#64;EnableWebMvc 애노테이션을 사용하면 &#60;mvc:annotation-driven&#62;과 동일하게 스프링 MVC를 설정하는데 필요한 빈을 자동으로 등록해준다.

```java
@Configuration
@EnableWebMvc
public class 빈설정클래스 {
	...
}
```

#### 3.3. 서블릿 매핑에 따른 컨트롤러 경로 매핑과 디폴트 서블릿 설정
<br/>
RequestMappingHandlerMapping는 요청 URL과 매핑되는 컨트롤러를 찾을 때 다음의 방법을 사용한다.
+ 서블릿 매핑이 "/경로/&#42;" 형식이면, "/경로" 이후 부분을 사용해서 컨트롤러 검색
+ 아니면, 컨텍스트 경로를 제외한 나머지 경로를 사용해서 컨트롤러 검색

위 규칙에 따르면 같은 URL이라도 서블릿 매핑 설정을 어떻게 했느냐에 따라 컨트롤러 매핑 경로가 바뀐다.  

DispatcherServlet에 대한 매핑 URL 패턴을 어떻게 설정했는냐에 따라 컨트롤러를 찾을 때 사용되는 경로가 달라지는데, 보통은 이런 동작을 원하지 않는다. mvc 네임스페이스를 사용하지 않는다면 RequestMappingHandlerMapping의 alwaysUserFullPath 프로퍼티의 값을 true로 지정해서 이 문제를 해소할 수 있다.

```xml
<!-- <mvc:annotation-driven>을 사용하지 않는다면 -->

<bean id="handlerMapping" class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
	<property name="alwaysUserFullPath" value="true" />
</bean>

<bean id="handlerAdapter" class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter" />
```

위와 같이 설정하면, 서블릿 매핑 URL 패턴에 상관없이 컨텍스트 경로를 제외한 나머지 전체 경로를 이용해서 컨트롤러를 찾게 된다. 하지만, RequestMappingHandlerMapping을 직접 등록하는 방식보다는 &#60;mvc:annotation-driven&#62;을 사용하는 것이 설정을 쉽게 할 수 있기 때문에, &#60;mvc:annotation-driven&#62; 등장 이후로는 &#60;mvc:annotation-driven&#62; 설정 방식을 많이 사용하고 있다. 그런데, 아쉽게도 &#60;mvc:annotation-driven&#62; 태그는 alwaysUserFullPath와 같은 설정 방법을 따로 제공하고 있지 않다.(이는 &#64;EnableWebMvc 애노테이션도 동일하다.)  

다음의 경우는 어떻게 해야 할까?  

+ URL에 do와 같은 확장자를 사용하지 않으면서 컨트롤러 매핑 경로로 젠처ㅔ 경로를 사용하고 싶음  

이런 경우에는 다음의 설정 방법을 사용하면 된다.
+ 서블릿 매핑 설정에서 URL 패턴을 "/"로 지정
+ 스프링 MVC 설정에 디폴트 서블릿 핸들러를 설정

그리고, 스프링 MVC 설정에서는 다음과 같이 &#60;mvc:default-servlet-handler /&#62; 태그를 추가하면 된다.  

서블릿 매핑 설정에서 매핑 URL 패턴을 &#60;url-pattern&#62;/&#60;url-pattern&#62;으로 설정하면 jsp 요청을 제외한 나머지 모든 요청을 DispatcherServlet이 받게 된다.  

&#60;mvc:default-servlet-handler /&#62; 설정을 추가하면, 디폴트 서블릿 핸들러가 빈으로 등록되며, 스프링 MVC는 다음과 같이 동작한다.
<br/>
(1) 요청 URL에 매핑되는 컨트롤러를 검색한다.  
	A. 존재할 경우, 컨트롤러를 이용해서 클라이언트 요청을 처리한다.  
<br/>
(2) 디폴트 서블릿 핸들러가 등록되어 있지 않다면, 404 응답 에러를 전송한다.  
(3) 디폴트 서블릿 핸들러가 등록되어 있으면, 디폴트 서블릿 핸들러에 요청을 전달한다.
	A. 디폴트 서블릿 핸들러는 WAS의 디폴트 서블릿에 요청을 전달한다.  

각 WAS는 서블릿 매핑에 존재하지 않는 요청을 처리하기 위한 디폴트 서블릿을 제공하고 있다. 예를 들어, JSP에 대한 요청을 처리하는 것이 바로 디폴트 서블릿이다. 그런데, 앞서 DispatcherServlet의 매핑 URL 패턴을 "/"로 지정하면 JSP를 제외한 모든 요청이 DispatcherServlet으로 가기 때문에, WAS가 제공하는 디폴트 서블릿이 &#42;.html이나 &#42;.css와 같은 요청을 처리할 수 없게 된다. 디폴트 서블릿 핸들러는 바로 이디폴트 서블릿에 요청을 전달해주는 핸들러로서, 요청 URL에 매핑되는 컨트롤러가 존재하지 않을 때 404 응답 대신, 디폴트 서블릿이 해당 요청 URL을 처리하도록 만든다. 따라서, .css와 같이 컨트롤러에 매핑되어 있지 않은 URL 요청은 최종적으로 3.A 과정을 통해 디폴트 서블릿에 전달되어 처리된다.  

&#64;EnableWebMvc 애노테이션을 사용할 경우 아래 코드처럼 디폴트 서블릿 핸들러를 등록할 수 있다.  

```java
@Configuration
@EnableWebMvc
public class 빈설정클래스 extends WebMvcConfigurerAdapter {
	
	...

	@Override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable();
	}

	...

}
```

WebMvcConfigureAdapter 클래스는 @EnableWebMvc 애노테이션을 이용해서 MVC를 설정할 때 사용되는 클래스로서, 스프링 MVC와 관련된 설정을 추가할 때 사용된다. 위 코드의 경우 WebMvcConfigurerAdapter에 정의된 메서드 중에서 configureDefaultServletHandling() 메서드를 재정의했는데, 이 메서드의 파라미터로 전달받은 DefaultServletHandlerConfigurer의 enable() 메서드를 호출함으로써 디폴트 서블릿 핸들러가 활성화된다.

### 4. 컨트롤러 구현
<br/>
#### 4.1. &#64;Controller/&#64;RequestMapping/Model을 이용한 컨트롤러 구현
<br/>
컨트롤러를 구현하는 과정은 다음과 같다.  

(1) &#64;Controller 애노테이션을 클래스에 적용한다.  
(2) &#64;RequestMapping 애노테이션을 이용해서 처리할 요청 경로를 지정한다.  
(3) 웹 브라우저의 요청을 처리할 메서드를 구현하고, 뷰 이름을 리턴한다.  

```java
@Controller
public class 클래스명 {

	...

	@RequestMapping("경로명1")
	public String 메소드명1() { // 웹 요청을 처리할 메소드

		...

		return "뷰의절대경로"; // 뷰 이름 리턴
	}

	...

	@RequestMapping("경로명2")
	public String 메소드명2(Model model) {
	
		...

		model.addAttribute("모델데이터명", 값);

		return "뷰의절대경로";
	}

	...

	@RequestMapping("경로명3")
	public String 메소드명3(ModelMap model) {
	
		...

		model.addAttribute("모델데이터명", 값);

		return "뷰의절대경로";
	}

	...

}
```

&#64;RequestMapping이 적용된 메서드는 클라이언트의 요청 결과를 보여줄 뷰 이름을 리턴한다. String 타입 외에 다른 타입의 객체를 리턴할 수도 있다.  

대부분의 &#64;RequestMapping 메서드는 뷰 이름을 리턴하기 전에 다음의 두 가지 작업을 수행한다.
+ 클라이언트의 요청을 처리한다.
+ 처리 결과 데이터를 뷰에 전달한다.  

예를 들어 이벤트 목록을 조회하는 요청을 받은 경우, 사용자 화면에 이벤트 목을 보여줘야 한다. 이를 위해서 컨트롤러는 이벤트 목록 데이터를 조회해서, 뷰가 이벤트 목록을 출력할 수 있도록 List를 뷰에 전달해야 한다.  

뷰에서 결과를 보여줄 때 필요한 데이터를 전달하기 위해 사용하는 것이 바로 모델이다. 컨트롤러는 뷰에서 필요로 하는 데이터를 모델에 담아서 전달하고, 뷰는 결과를 생성하는데 필요한 데이터를 모델에서 가져와 사용하게 된다.  

컨트롤러는 다양한 방식으로 모델에 데이터를 담을 수 있는데, 가장 간단한 방법은 Model을 파라미터로 추가하고, Model 파라미터에 데이터를 추가하는 것이다.  

Model 클래스는 addAttribute() 메서드를 제공하고 있다. 이 메서드는 첫 번째 파라미터로 모델 데이터의 이름을, 두 번째 파라미터로 값을 입력받는다. 이렇게 입력받은 모델 데이터는 뷰 코드에서 이름을 통해 접근할 수 있다.  

Model에 모델 데이터를 추가하는데 사용되는 메서드는 다음과 같다.  
+ Model addAttribute(String attrName, Object attrValue)  
이름이 attrName이고 값이 attrValue인 모델 속성을 추가한다.  
+ Model addAllAttribute(Map&60;String, ?&#62; attributes)  
맵의 &#60;키, 값&#62; 쌍들을 모델 속성 이름과 값으로 추가한다.  
+ boolean containsAttribute(String attrName)  
이름이 attrName인 모델 속성을 포함할 경우 true를 리턴한다.    

Model의 추가 메서드는 Model 자신을 리턴하기 때문에, 다음과 같이 메서드 체이닝 방식으로 작성해서 불필요한 코드 입력을 줄일 수 있다.

```java
model.addAttribute("모델데이터명1", 값1).addAttribute("모델데이터명2", 값2);
```

Model 타입 대신이 ModelMap 타입을 사용해도 동일한 결과를 얻을 수 있다.  

ModelMap 클래스는 Model 인터페이스와 동일한 메서드를 제공하고 있기 때문에, addAttribute() 등의 메서드를 그대로 사용할 수 있다.  

##### 4.1.1. ModelAndView를 사용한 모델/뷰 처리
<br/>
ModelAndView는 모델 설정과 뷰 이름을 합쳐 놓은 것이다. &#64;Controller를 이용한 컨트롤러 구현이 대세로 자리 잡기 전에 컨트롤러 클래스 메서드의 리턴 타입은 ModelAndView였다. &#64;Controller를 이용하는 경우에도 ModelAndView를 사용할 수 있다.

```java
public class 클래스명 {
	
	...

	@RequestMapping("경로명")
	public ModelAndView 메소드명() {
		
		...

		ModelAndView modelView = new ModelAndView();
		modelView.setViewName("뷰의절대경로");
		modelView.addObject("모델데이터명", 값);
		return modelView;
	}

	...

}
```

앞서 Model을 사용하는 경우와 비교해보면 다음의 차이점이 있다.
+ Model을 사용하는 경우 뷰 이름을 리턴하는데, ModelAndView를 사용하는 경우는 setViewName()을 이용해서 뷰 이름을 지정한다.
+ Model은 addAttribute() 메서드를 사용하는데, ModelAndView를 사용하는 경우는 addObject() 메서드를 사용한다.

#### 4.2. &#64;RequestMapping을 이용한 요청 매핑
<br/>
컨트롤러를 구현한다는 것은 클라이언트의 요청을 처리할 메서드를 구현한다는 것을 뜻한다. 클라이언트는 URL로 요청을 전송하는데, 요청 URL을 어떤 메서드가 처리할지 여부를 결정하는 것이 &#64;RequestMapping 애노테이션이다.  

##### 4.2.1. &#64;RequestMapping을 이용한 경로 지정
<br/>
&#64;RequestMapping 애노테이션의 값으로 경로를 지정한다. 스프링 MVC는 웹 브라우저의 요청이 들어오면 &#64;RequestMapping의 값을 이용해서 요청을 처리할 컨트롤러 메서드를 결정한다.

&#64;RequestMapping 애노테이션은 여러 속성을 제공하고 있는데, 이들 속성들을 사용해야 할 경우 value 속성을 사용해서 경로를 지정할 수 있다.

```java
@Controller
public class 클래스명 {

	...

	@RequestMapping(value="겅로명1", method=RequestMethod.POST)
	public String 메소드명1(...) {

		...

	}

	...

	@RequestMapping({"겅로명1", "경로명2"})
	public String 메소드명2(...) {

		...

	}

	...

}
```

여러 경로를 한 메서드에서 처리하고 싶다면, 배열로 경로 목록을 지정하면 된다.

##### 4.2.2. 클래스와 메서드에 &#64;RequestMapping 적용하기
<br/>
```java
@Controller
@RequestMapping("경료명1")
public class 클래스명 {

	...

	@RequestMapping("경로명2")
	public String 메소드명(Model model) {
		...
	}

	...

}
```

위와 같이 컨트롤러와 메서드에 &#64;RequestMapping 애노테이션을 모두 사용하면, 클래스에 적용한 값과 메서드에 적용한 값을 조합해서 매핑될 경로를 결정한다.  

클래스에 &#64;RequestMapping 애노테이션을 적용하는 것은 컨트롤러 클래스가 특정 경로를 기준으로 그 하위 경로만을 처리한다는 것을 의미한다. 즉, 같은 경로를 공유하는 경우, 클래스에 &#64;RequestMapping 애노테이션을 적용함으로써 코드에서 직관적으로 공통 경로를 확인할 수 있다.

##### 4.2.3. HTTP 전송 방식 지정
<br/>
웹 어플리케이션을 개발할 때, 로그인 데이터를 전송하거나 게시글 데이터를 전송해야할 경우 &#60;form&#62; 태그에서 method 속성값을 "post"로 지정하는 것이 일반적이다.  

POST는 HTTP의 전송 방식(method) 중 하나인데, &#64;RequestMapping 애노테이션은 method 속성을 이용해서 메서드에서 처리할 전송 방식을 지정할 수 있다.  

```java
@Controller
public class 클래스명 {
	
	...

	@RequestMapping(value="경로명1", method=RequestMethod.GET)
	public String 메소드명1(...) {
		...

		return "경로명1";
	}

	...

	@RequestMapping(value="경로명2", method=RequestMethod.POST)
	public String 메소드명2(...) {
		...

		return "경로명2";
	}

	...

}
```

```java
@Controller
@RequestMapping("경로명1")
public class 클래스명 {

	@RequestMapping(method=RequestMethod.GET)
	public String 메소드명1() {
		...
	}

	...

	@RequestMapping(method=RequestMethod.POST)
	public String 메소드명2() {
		...
	}
}
```

URL 요청이 들어오더라도 HTTP 전송 방식에 따라서 처리를 수행하는 메서드가 달라지게 된다.  

org.springframework.web.bing.annotation.RequestMethod는 열거 타입으로 다음의 값을 정의하고 있다.
+ GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE  

웹 브라우저는 GET과 POST 방식만을 지원하기 때문에, 웹 브라우저에서 실행하는 자바 스크립트에서 웹 서버에 PUT이나 DELETE 방식의 요청을 보낼 수 없다. REST API를 만들 때, PUT, DELETE 전송 방식을 이용해서 구현해야 하는 경우 웹 브라우저의 전송 방식 제한은 문제가 된다. 이런 문제를 해소하기 위해 스프링은 HiddenHttpMethodFilter를 제공하고 있는데, 이 필터를 사용하면 컨트롤러 메서드가 PUT, DELETE 등의 전송방식을 처리하도록 구현하면서, 동시에 같은 메서드를 이용해서 Ajax 요청을 처리할 수 있게 된다.  

##### 4.2.4. &#64;PathVariable을 이용한 경로 변수
<br/>
게시글의 내용을 보여주는 URL을 보면 http://host/readArticles?id=10와 같이 요청 파라미터를 이용해서 읽어올 게시글을 지정하는 경우가 많다. 하지만, 아래와 같이 요청 파라미터를 사용하지 않고 URL 자체를 이용해서 게시글 링크를 표현하는 경우도 많다.
+ http://host/articles/10, http://host/articles/20
+ http://host/members/bkchoi/orders
+ http://host/members/madvirus/orders/30  

RESTful API의 증가와 함께 이런 방식을 URL 구성이 증가하고 있는데, &#64;PathVariable 애노테이션을 사용하면 위와 같은 URL을 쉽게 처리할 수 있다.

```java
@Controller
public class 클래스명 {

	@RequestMapping("경로명1/{변수명1}/{변수명2}/{변수명3:정규표현식}")
	public String 메소드명(@PathVariable String 변수명1, @PathVariable("변수명2") Long 변수명4, @PathVariable String 변수명3, ...) {
		...
	}

	...

}
```

경로 변수는 한 개 이상 사용할 수 있다.  

스프링은 경로 변수의 값을 파라미터의 타입에 맞게 변환해준다. 경로 변수를 파라미터 타입으로 변환할 수 없을 경우, 스프링 MVC는 웹 브라우저에 400 에러 코드를 응답 결과로 전송한다.  

경로 변수에 정규 표현식을 사용할 수도 있다. 경로 변수 이름 뒤에 콜론과 정규 표현식을 함께 사용해서 정규표현식에 매칭되는 경로만 매핑되도록 제한할 수 있다.  

정규 표현식에 매칭되지 않는 경로를 입력하면 스프링은 404 에러를 응답한다.  

정규 표현식을 사용해서 경로 변수의 값을 제한할 수 있는 것이 유용해 보일 수 있지만, 실제 정규 표현식이 사용되는 부분은 업무 영역의 규칙인 경우가 많다. 예를 들어, 업로드한 파일의 ID 생성은 컨트롤러가 아닌 파일을 관리하는 도메인 영역의 코드에서 하게 된다. 즉, 서비스와 같은 도메인 영역의 코드에서 파일 ID 생성 규칙을 관리하게 된다. 따라서, 컨트롤러의 경로 변수에 정규 표현식을 사용하는 것보다는, 서비스와 같은 도메인 영역의 코드에서 잘못 들어온 파일 ID를 검사하는 것이 코드의 유지보수성을 높이는 데 도움이 된다.  

##### 4.2.5. Ant 패턴을 이용한 경로 매핑
<br/>
&#64;RequestMapping 애노테이션의 값으로 Ant 패턴을 사용할 수도 있다. Ant 패턴은 &#42;, &#42;&#42;, ?의 세 가지 특수 문자를 이용해서 경로를 표현하는 패턴으로, 세 가지 문자는 다음의 의미를 갖는다.  
+ &#42; - 0개 또는 그 이상의 글자
+ ? - 1개 글자
+ &#42;&#42; - 0개 또는 그 이상의 디렉토리 경로  

이들 문자를 사용한 경로 표현 예는 다음과 같다.
+ &#64;RequestMapping("/member/?&#42;.info")  
/member/로 시하고 확장자가 .info로 끝나는 모든 경로  
+ &#64;RequestMapping("/faq/f?00.fq")  
/faq/f로 시작하고, 1글자가 사이에 위치하고 00.fq로 끝나는 모든 경로  
+ &#64;RequestMapping("/folders/&#42;&#42;/files")
/folders/로 시작하고, 중간에 0개 이상의 중간 경로가 존재하고 /files로 끝나는 모든 경로. 예를 들어, /folders/files, folders/1/2/3/files 등이 매핑된다.  

Ant 패턴을 유용하게 사용할 수 있는 경우는 트리 형식으로 표현된 경로를 정의하구 싶을 때이다.  

Ant 패턴을 사용할 경우, 실제 경로 값을 구하려면 요청 URI로부터 패턴 부분을 분리하는 작업을 해야 한다.  

org.springframework.util.AntPathMatcher 클래스를 사용하면 요청 URI에서 패턴 부분을 조금 더 쉽게 찾아올 수 있다.

##### 4.2.6. 처리 가능한 요청 컨텐트 타입/응답 가능한 컨텐트 타입 한정
<br/>
서비스 또는 클라이언트/서버 간 통신 방식으로 REST API가 자리를 잡으면서 HTTP의 데이터로 JSON이나 XML을 전송하는 경우가 증가했다. 예를 들어, 웹 브라우저에서 폼을 전송할 때 사용하는 application/x-www-form-urlencoded 컨텐트 타입이 아니라 application/json이나 application/xml과 같은 컨텐트 타입을 이용해서 서버와 데이터를 주고 받는 경우가 증가하고 있다.  

&#64;RequestMapping은 컨트롤러 메서드에서 처리 가능한 요청 컨텐트 타입과 응답 컨텐트 타입을 제한하는 방법을 제공하고 있으며, 이를사용하면 같은 URL이라 하더라도 콘텐트 타입에 따라 다른 응답을 보여주도록 처리할 수 있다.  

요청 컨텐트 타입을 제한하고 싶다면, comsumes 속성을 사용하면 된다.

```java
@Controller
public class 클래스명 {

	...

	@RequestMapping(value="경로명1", method=RequestMethod.POST, consumes="application/json")
	public ModelAndView 메소드명(@RequestBody 클래스명 변수명) {
		...
	}

	...

}
```

반대로 응답 결과로 JSON을 요구하는 요청을 처리하고 싶다면, 즉 Accept 요청 헤더에 application/json이 포함된 경우만 처리하고 싶다면, produces 속성을 사용하면 된다.  

```java
@Controller
public class 클래스명 {

	...

	@RequestMapping(value="경로명1", method=RequestMethod.GET, produces="application/json")
	@ResponseBody
	public 클래스명 메소드명(...) {
		...
	}

	...

}
```

consumes 속성과 produces 속성은 요청/응답이 XML이나 JSON인 경우에 이를 명시적으로 지정하고 제한하기 위해 사용된다.

#### 4.3. HTTP 요청 파라미터와 폼 데이터 처리
<br/>
웹 브라우저는 웹 서버에 HTTP 요청 파라미터를 통해서 데이터를 전송한다. 스프링 MVC에서 요청 파라미터 값을 구하는 방법은 크게 다음의 세 가지 정도로 구분할 수 있다.
+ HttpServletRequest의 getParameter() 메서드를 이용해서 구하기
+ &#64;RequestParam 애노테이션을 이용해서 구하기
+ 커맨드 객체를 이용해서 구하기  

##### 4.3.1. HttpServletRequest를 이용한 요청 파라미터 구하기
<br/>
요청 파라미터를 처리하는 첫 번째 방법은 HttpServletRequest를 이용하는 것이다. &#64;RequestMapping 메서드에 HttpServletRequest 타입의 인자를 추가한 뒤, getParameter() 메서드를 이용해서 HTTP 요청 파라미터를 구하면ㅌ 된다.

```java
@Controller
public class 클래스명 {

	...

	@RequestMapping("경로명")
	public String 메소드명(HttpServletRequest request, ...) throws IOException {
	
		...

		String 변수명 = request.getParameter("요청파라미터명");

		...

		return "경로명";
	}
}
```
##### 4.3.2. &#64;RequestParam 애노테이션을 이용한 요청 파라미터 구하기
<br/>
&#64;RequestParam 애노테이션을 사용하면 메서드의 파라미터를 이용해서 HTTP 요청 파라미터를 받을 수 있다.

```java
@Controller
public class 클래스명 {

	...

	@RequestMapping("경로명1")
	public String 메소드명(@RequestParam("요청파라미터명") 타입명 변수명, ...) {
	
		...

		return "경로명1";
	}

	...

	@RequestMapping("경로명2")
	public String 메소드명(@RequestParam(value = "요청파라미터명", required=false) 타입명 변수명, ...) {
	
		...

		return "경로명2";
	}

	...

	@RequestMapping("경로명3")
	public String 메소드명(@RequestParam(value = "요청파라미터명", defaultValue="") 타입명 변수명, ...) {
	
		...

		return "경로명3";
	}

	...

}
```

요청 파라미터의 값이 없거나 해당 타입으로 변환할 수 없을 경우, 스프링은 컨트롤러의 메서드를 실행하지 않고 바로 400 에러 코드를 클라이언트에 응답한다.  

요청 파라미터가 필수가 아닐 수도 있다. 이렇게 요청 파라미터가 필수가 아니라면 &#64;RequestParam 애노테이션의 required 속성 값을 false로 지정해주면 된다.  

이렇게 하면 요청 파라미터가 존재하지 않아도 스프링 MVC는 400 에러를 응답하지 않고 파라미터의 값으로 null을 리턴한다.  

defaultValue 속성을 사용하면, 요청 파라미터가 존재하지 않을 때 null 대신 다른 값을 사용하도록 설정할 수도 있다.  

defaultValue를 이용하면 null 검사를 생략할 수 있기 때문에, 페이징 처리에서의 페이지 번호를 위한 요청 파라미터를 처리할 때 유용하게 사용할 수 있다.  

##### 4.3.3. 커맨드 객체를 이용한 폼 전송 처리하기
<br/>
회원 가입을 받기 위해 다음과 같은 HTML 폼을 사용한다고 하자.
```html
<form method="post">
이메일: <input type="text" name="email" /> <br/>
이름: <input type="text" name="name" /> <br/>
암호: <input type="password" name="password" /> <br/>
암호 확인: <input type="password" name="confirmPassword" /> <br/>
<input type="submit" value="가입" />
```

이 폼을 이용해서 전송한 요청 파라미터를 사용하기위해 다음 코드처럼 요청 파라미터를 개별적으로 처리할 수 있을 것이다.  

```java
@Controller
public class 클래스명 {

	...

	@RequestMapping(value="경로명", method=RequestMethod.POST)
	public String 메소드명(@RequestParam("email") String email,
				@RequestParam("name") String name,
				@RequestParam("password") String password,
				@RequestParam("confirmPassword") String confirmPassword) {
		MemberRegistRequest memRegReq = new MemberRegistRequest();
		memRegReq.setEmail(email);
		memRegReq.setName(name);
		memReqReq.setPassword(confirmPassword);

		...

	}

	...

}
```

그런데, HTTP 요청 파라미터를 모아서 하나의 객체에 담아야 한다면 어떻게 될가? 이 경우 HTTP 요청 파라미터를 MemberRegistRequest 객체에 복사하려면, 번거러운 코드를 작성해야 한다.  

스프링 MVC는 이런 불편함을 줄여주는 방법을 제공하고 있다. 방법은 매우 쉽다. 다음과 같이 요청 파라미터를 지정해주기만 하면 된다.

```java
@Controller
@RequestMapping("/member/regist")
public class 클래스명 {

	...

	@RequestMapping(method = RequestMethod.POST)
	public String regist(MemberRegistRequest memRegReq) {
		
		...

		return "member/registered";
	}
}
```

스프링 MVC는 &#64;RequestMapping이 적용된 메서드의 파라미터에 객체를 추가하면, 그 객체의 set 메서드를 호출해서 요청 파라미터 값을 전달한다. 이때 요청 파라미터의 값을 객체의 프로퍼티로 복사할 때에는 같은 이름을 갖는 요청 파라미터와 프로퍼티를 매핑한다. 예를 들어, memRegReq.setEmail() 메서드를 이용해서 email 요청 파라미터의 값을 전달받으며, memRegReq.setPassword() 메서드를 이용해서 password 요청 파라미터의 값을 전달받는다.  

여기서 MemberRegistRequest 객체와 같이 HTTP 요청 파라미터 값을 전달받을 때 사용되는 객체를 커맨드 객체라고 부른다. 커맨드 객체로 사용될 클래스에 제한은 없으며, 자바빈 규약에 맞춰 알맞은 set 메서드만 제공하면 된다.  

커맨드 객체는 뷰에 전달할 모델에 자동으로 포함된다. 위 코드의 경우 memRegReq 커맨드 객체는 "memberRegistRequest"라는 이름의 모델 이름으로 뷰에 전달된다. (즉, 단순 클래스 이름의 첫 글자를 소문자로 변환한 이름을 사용한다.) 따라서, 뷰 코드에서는 "memberRegistRequest"라는 이름을 사용해서 컨맨드 객체의 값을 사용할 수 있다.  

```java
public class MemberRegistRequest {
	private String email;
	private String name;
	private String password;
	private String confirmPassword;
	private boolean allowNoti;
	private int[] favoriteIds;
	private List favoriteIdList;
	private Address address;
	
	public String getEmail() {
		return email;
	}
	
	public void setEmail(String email) {
		this.email = email;
	}
	
	public String getName() {
		return name;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	
	public String getPassword() {
		return password;
	}
	
	public void setPassword(String password) {
		this.password = password;
	}
	
	public String getConfirmPassword() {
		return confirmPassword;
	}
	
	public void setConfirmPassword(String confirmPassword) {
		this.confirmPassword = confirmPassword;
	}
	
	public boolean isAllowNoti() {
		return allowNoti;
	}
	
	public void setAllowNoti(boolean allowNoti) {
		this.allowNoti = allowNoti;
	}
	
	public int[] getFavoriteIds() {
		return favoriteIds;
	}
	
	public void setFavoriteIds(int[] favoriteIds) {
		this.favoriteIds = favoriteIds;
	}
	
	public List getFavoriteIdList() {
		return favoriteIdList;
	}
	
	public void setFavoriteIdList(List favoriteIdList) {
		this.favoriteIdList = favoriteIdList;
	}
	
	public Address getAddress() {
		return address;
	}
	
	public void setAddress(Address address) {
		this.address = address;
	}
}
```

```html
	<form action="/spring4-chap07/member/regist.do" method="post">
		이메일: <input type="text" name="email"/> <br/>
		이름: <input type="text" name="name"/> <br/>
		암호: <input type="password" name="password" /> <br/>
		암호 확인: <input type="password" name="confirmPassword" /> <br/>
		<label>
			<input type="checkbox" name="allowNoti" value="true" />
			이메일을 수신합니다. <br/>
		</label>
		<label>
			favoriteIds <br/>
			<input type="checkbox" name="favoriteIds" value="1" /> 1 <br/>
			<input type="checkbox" name="favoriteIds" value="2" /> 2 <br/>
			<input type="checkbox" name="favoriteIds" value="3" /> 3 <br/>
		</label>
		<label>
			favoriteIdList <br/>
			<input type="checkbox" name="favoriteIdList" value="1" /> 1 <br/>
			<input type="checkbox" name="favoriteIdList" value="2" /> 2 <br/>
			<input type="checkbox" name="favoriteIdList" value="3" /> 3 <br/>
		</label>
		<label>주소</label>
		주소1
		<input type="text" name="address.address1" /> <br/>
		주소2
		<input type="text" name="address.address2" /> <br/>
		우편번호
		<input type="text" name="address.zipcode" /> <br/>
		<input type="submit" value="가입" />
	</form>
```
&#64;RequestParam과 비슷하게 HTTP 요청 파라미터의 값을 커맨드 객체의 프로퍼티 타입에 맞게 변환해준다.  

같은 이름의 요청 파라미터가 두 개 이상 존재할 경우, 배열을 이용해서 요청 파라미터 목록을 전달받으면 된다.  

배열 대신에 Collection이나 List와 같은 콜렉션 타입을 사용해도 된다.  

다수의 요청 파라미터 값을 한 개의 객체에 담을 수 있기 때문에, 폼 전송이나 요청 파라미터 개수가 많은 경우 커맨드 객체를 사용하면 컨트롤러 코드를 간결하게 유지할 수 있다. 이런 장점 외에도 스프링이 제공하는 객체 검증, 타입 변환 등을 이용하면, 번거로운 코드를 줄일 수 있다.  

##### 4.3.4. 커맨드 객체의 중첩 객체 프로퍼티 지원
<br/>
스프링은 커맨드 객체의 중첩 프로퍼티를 지원한다. 

```java
public class Address {
	
	private String address1;
	private String address2;
	private String zipcode;
	
	public String getAddress1() {
		return address1;
	}
	
	public void setAddress1(String address1) {
		this.address1 = address1;
	}
	
	public String getAddress2() {
		return address2;
	}
	
	public void setAddress2(String address2) {
		this.address2 = address2;
	}
	
	public String getZipcode() {
		return zipcode;
	}
	
	public void setZipcode(String zipcode) {
		this.zipcode = zipcode;
	}
}
```

이 코드에서 address 프로퍼티는 Address 타입의 객체이다. Address 클래스는 다음과 같이 주소를 표현하는데 사용되는 프로퍼티를 정의하고 있다.  

컨트롤러가 MemberRegistRequest 객체를 커맨드 객체로 사용하면서, address 프로퍼티에도 값을 채우고 싶다면 파라미터를 중첩해서 사용하면 된다.  

위 HTML 폼에서 이름이 "address.address1", "address.address2", "address.zipcode"인 &#60;input&#62; 태그가 있는데, 이들 입력 값은 커맨드 객체의 "address" 프로퍼티 객체의 "address1", "address2", "zipcode" 프로퍼티 값으로 설정된다. 따라서, MemberRegistRequest 타입을 커맨드 객체로 사용하면, 중첩 프로퍼티에 알맞은 요청 파라미터 값이 할당된다.

##### 4.3.5. 커맨드 객체의 배열/리스트 타입 프로퍼티 처리
<br/>
```html
<input type="hidden" name="perms[0].id" value="bkchoi1">
<input type="checkbox name="perms[0].canRead" value="true" checked>

...

<input type="hidden name="perms[1].id" value="bkchoi2">
<input type="checkbox" name="perms[1].canRead" value="false" checked>

<input type="hidden name="perms[3].id" value="madvirus">
<input type="checkbox" name="perms[3].canRead" value="false" checked>

```
위 HTML 코드에서 파라미터 이름은 "프로퍼티이름[인덱스].프로퍼티이름"의 구조를 갖고 있다.  

List 타입이나 배열 타입 프로퍼티의 경우, 위 매핑처럼 '프로퍼티이름[인덱스]'의 형식으로 해당 인덱스의 값에 접근할 수 있다. 또한, 중첩 프로퍼티를 사용해서 프로퍼티를 설정할 수 있다. 인덱스 기반 프로퍼티와 중첩 프로퍼티를 이용하면, 컨트롤러는 커맨드 객체를 사용해서 간단하게 요청 파라미터를 입력받을 수 있게 된다.  

만약 인덱스 번호가 중간에 비면 어떻게 될까?  

위 HTML 코드는 인덱스가 1인 값이 존재하지 않는데, 이 경우 커맨드 객체의 perms 필드는 다음과 같은 값을 갖는 객체 목록을 갖게 된다.  
+ perms.get(0): (id: "bkchoi1", canRead: "true")
+ perms.get(1): (id: "bkchoi2", canRead: "false")
+ perms.get(2): (id: null, canRead: null)
+ perms.get(3): (id: "madvirus", canRead: "false")  

위 목록을 보면 perms의 2번 인덱스 값이 null이 아니라는 점에 주목하자. 스프링은 빈 인덱스 번호에 객체를 생성하고, 중첩 프로퍼티의 각 값에 null을 할당하고 있다. 따라서, 위와 같이 데이터가 없는 인덱스 항목이 존재할 수 있다면 알맞게 처리해야 한다.  

인덱스 기반 프로퍼티의 타입으로 int나 String과 같은 단순 데이터 타입을 사용해도 된다.  

##### 4.3.6. GET과 POST에서 동일 커맨드 객체 사용하기
<br/>

#### 4.4. &#64;ModelAttribute를 이용한 모델 데이터 처리
<br/>
##### 4.4.1. &#64;ModelAttribute를 이용한 커맨드 객체 이름 지정
<br/>
커맨드 객체로 사용될 파라미터에 &#64;ModelAttribute를 적용하면 커맨드 객체의 모델 이름을 변경할 수 있다.  

```java
@Controller
@RequestMapping("/member/regist")
public class RegistrationController {

	...

	@RequestMapping(mehtod = RequestMethod.POST)
	public String regist(@ModelAttribute("memberInfo") MemberRegistRequest memRegReq) {
	
		...

		return "member/registered";
	}

	...

}
```

위 코드에서 regist() 메서드를 보면 memRegReq 파라미터에 &#64;ModelAttribute("memberInfo") 애노테이션을 적용했는데, 이 경우 memRegReq  커맨드 객체는 모델에 "memberInfo"라는 이름으로 저장된다. 따라서, 뷰 코드에서는 커맨드 객체의 클래스 이름이 아닌 "memberInfo"를 이용해서 커맨드 객체에 접근하게 된다.

```jsp
<!-- JSP 코드에서 @ModelAttribute로 지정한 모델 이름 사용 -->
${memberInfo.name}님의 회원 가입을 완료했습니다.
```

##### 4.4.2. &#64;ModelAttribute 애노테이션을 이용한 공통 모델 처리
<br/>
```java
@Controller
@RequestMapping("/event")
public class EventController {

	@ModelAttribute("recEventList")
	public List<Event> recommend() {
		return eventService.getRecommendedEventList();
	}

	@RequestMapping("/list1")
	public String list(Model model) {
		List<Event> eventList = eventService.getOpenedEventList();
		model.addAttribute("eventList", eventList);
		return "event/list";
	}

	@RequestMapping("/detail")
	public String list(HttpServletRequest request, Model model) throws IOException {
		...
		return "event/detail";
	}

	@RequestMapping("/list2")
	public String list(@ModelAttribute("recEventList") List<Event> recEventList, Model model) {
		...
	}

	...

}
```

위 코드에서 recommend() 메서드에 &#64;ModelAttribute 애노테이션을 붙였는데, 스프링 MVC는 recommend() 메서드의 리턴 결과를 "recEventList" 모델 속성으로 추가한다. 따라서 요청 경로가 "/event/list"나 "/event/detail"인지에 상관없이 뷰 코드에서는 "recEventList" 이름을 이용해서 recommend() 메서드가 리턴한 객체에 접근할 수 있다.  

&#64;ModelAttribute 애노테이션이 적용된 메서드의 객체를 &#64;RequestMapping 애노테이션이 적용된 메서드에서 접근할 수도 있다. 다음과 같이 같은 값을 갖는 &#64;ModelAttribute 애노테이션을 사용하면 된다. 아래 코드에서 list() 메서드의 첫 번째 파라미터로 전달되는 객체는 recommend() 메서드가 리턴한 객체가 된다.  

#### 4.5. &#64;CookieValue와 &#64;RequestHeader를 이용한 쿠키 및 요청 헤더 구하기
<br/>
HttpServletRequest를 사용하는 대신 &#64;RequestParam 애노테이션을 사용해서 요청 파라미터 값을 구했던 것처럼 쿠키 값과 헤더를 구하기 위해 &#64;CookieValue 애노테이션과 &#64;RequestHeader 애노테이션을 사용할 수 있다. 다음은 두 애노테이션의 사용 예이다.
```java
@Controller
public class SimpleHeaderController {

	...

	@RequestMapping("/header/simple1")
	public String simple1(@RequestHeader("Accept") String acceptType, @CookieValue("auth") Cookie cookie, Model model) {
		...
	}

	...

	@RequestMapping("/header/simple2")
	public String simple2(@RequestHeader(value="Accept", defaultValue="text/html") String acceptType, @CookieValue(value = "auth", required = false) Integer authValue, Model model) {
		...
	}

	...

}
```

위 코드는 이름이 "Accept"인 요청 헤더의 값과 이름이 "auth"인 쿠키를 각각 acceptType 파라미터와 cookie 파라미터에 전달한다. 지정한 이름의 헤더나 쿠키가 존재하지 않을 경우 스프링은 400 에러를 응답한다.  

두 애노테이션 모두 required 속성과 defaultValue 속성을 이용해서 필수 여부와 기본값을 지정할 수 있다. 위 코드는 "Accept" 헤더가 존재하지 않을 경우 기본값으로 "text/html"을 사용하고 "auth" 쿠키가 존재하지 않으면 cookie 파라미터에 null을 전달하도록 설정한 코드의 작성 예이다.  

javax.servlet.http.Cookie 타입 대신에 쿠키 값에 해당하는 타입을 바로 사용해도 된다. 예를 들어, 쿠키 값이 정수 타입이라면 다음과 같이 Integer나 int 등의 타입을 사용하면 알맞게 타입 변환을 한다.

#### 4.6. 리다이렉트 처리
<br/>
컨트롤러에서 클라이언트의 요청을 처리한 후에 다른 페이지로 리다이렉트하고 싶다면 뷰 이름 앞에 "redirec:" 접두어를 붙이면 된다. 아래 코드는 작성 예이다.  

```java
@RequestMapping("/header/createauth")
public String createAuth(HttpServletResponse response) {
	...
	return "redirect:/main";
}

@RequestMapping(value="/files/{fileId}", method=RequestMethod.POST)
public String updateFile(@PathVariable String fileId, ..,) {
	...
	return "redirect:/files/{fileId}";
}
```

"redirect:"에 경로 변수를 사용할 수도 있다.

### 5. 커맨드 객체 값 검증과 에러 메시지
<br/>
웹 개발에서 입력 폼의 값이 올바른지 검증하는 것은 매우 중요한 작업이다. 요청 파라미터 값을 확인하지 않고 그대로 사용하게 되면, 데이터베이스 등에 잘못된 데이터가 들어갈 가능성이 높아지게 된다. 요청 파라미터 값을 검사할 때에는 크게 두 가지 방법을 이용한다.
+ 웹 브라우저 : 자바 스크립트를 이용해서 데이터를 웹 서버에 전송하기 전에 미리 검사한다.
+ 웹 서버 : 전달받은 요청 파라미터의 값을 검사한다. 파라미터 값이 올바르지 않을 경우 에러 코드를 응답하거나 재입력을 위한 폼 화면을 웹 브라우저에 전송한다.  

스프링은 이 두 영역 중에서 서버 측의 요청 파라미터 검사 기능을 지원하고 있다.  

웹 브라우저에서 자바 스크립트를 이용해서 폼 값을 검사하는 것은 사용자에게 빠르게 검사 결과를 보여줄 수 있기 때문에 사용자 입장에서 중요하다. 하지만, 악의적으로 잘못된 데이터를 서버에 전송하거나 클라이언트에서 완전히 값을 검증하지 못할 수도 있기 때문에, 서버에서도 반드시 값을 검증해야 한다.

#### 5.1. Validator와 Erros/BindingResult를 이용한 객체 검증
<br/>
org.springframework.validation.Validator 인터페이스를 사용하면, 스프링이 제공하는 객체 검증 및 에러 메시지 지원 등의 기능을 사용할 수 있다. 컨트롤러에서 커맨드 객체의 값을 검증할 때 특히 Validator를 유용하게 사용할 수 있다.  

Validaotr 인터페이스는 두 개의 메서드를 정의하고 있다.  

supports() 메서드는 Validator가 해당 타입의 객체를 지원하는지 여부를 리턴한다. 실제 값을 검증하는 코드는 validate() 메서드에 위치한다. validate() 메서드의 target 파라미터는 값을 검증할 객체이며, errors 파라미터는 값이 올바르지 않을 경우 그 내용을 저장하기 위해 사용된다.

```java
public class MemberRegistValidator implements Validator {

	@Override
	public boolean supports(Class<?> clazz) {
		return MemberRegistRequest.class.isAssignableFrom(clazz);
	}

	@Override
	public void validate(Object target, Errors errors) {
		MemberRegistRequest regReq = (MemberRegistRequest) target;
		if (regReq.getEmail() == null || regReq.getEmail().trim().isEmpty())
			errors.rejectValue("email", "required");

		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "password", "required");
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "confirmPassword", "required");
		if (regReq.hasPassword()) {
			if (regReq.getPassword().length() < 5)
				errors.rejectValue("password", "shortPassword");
			else if (!regReq.isSamePasswordConfirmPassword())
				errors.rejectValue("confirmPassword", "notSame");
		}
		Address address = regReq.getAddress();
		if (address == null) {
			errors.rejectValue("address", "required");
		} else {
			errors.pushNestedPath("address");
			try {
				ValidationUtils.rejectIfEmptyOrWhitespace(errors, "address1", "required");
				ValidationUtils.rejectIfEmptyOrWhitespace(errors, "address2", "required");
			} finally {
				errors.popNestedPath();
			}
		}
	}
}
```

MemberRegistValidator 클래스는 MemberRegistRequest 타입의 객체를 지원하도록 supports() 메서드를 구현했다.  

실제로 객체에 대한 검증 작업은 validate() 메서드에서 수행한다.

```java
@Controller
@RequestMapping("/member/regist")
public class RegistrationController {

	...

	@RequestMapping(method = RequestMethod.POST)
	public String regist(@ModelAttribute("memberInfo") MemberRegistRequest memRegReq, BindingResult bindingResult) {
		new MemberRegistValidator().validate(memRegReq, bindingResult);
		if (bindingResult.hasErrors()) {
			return MEMBER_REGISTRATION_FORM;
		}
		memberService.registNewMember(memRegReq);
		return "member/registered";
	}

	...

}
```

위 코드에서 regist() 메서드는 커맨드 객체인 memRegReq 파라미터와 에러 정보를 보관할 bindingResult 파라미터를 갖고 있다. BindingResult는 Errors 인터페이스를 상속받은 타입으로 컨트롤러를 구현할 때에는 두 타입 중 하나를 사용하면 된다.  

커맨드 객체를 검증하려면 MemberRegistValidator 객체를 생성하고 validate() 메서드를 호출하면 된다. validate() 메서드를실행한 후에 값에 오류가 존재하면 bindingResult.hasErrors() 메서드는 true를 리턴한다. 따라서, bindingResult.hasErrors()가 true를 리턴하면 다시 입력 폼을 보여주거나 에러 화면을 보여주는 등의 입력 오류 처리를 하면 된다.  

앞서 Validator 구현 코드에서 Errors.rejectValue() 메서드를 이용해서 잘못된 프로퍼티를 등록했는데, 이렇게 reject 메서드를 이용하면 데이터에 오류가 있다고 등록하게 된다. 따라서, Validator에서 Errors의 reject 메서드를 한 번 이상 호출하면, Errors.hasErrors() 메서드는 true를 리턴하게 된다.  

커맨드 객체의 에러 정보를 담기 위해 사용되는 Errors 파라미터나 BindingResult 파라미터는 반드시 커맨드 객체 파라미터 바로 뒤에 위치해야 한다.  

만약 Errors나 BindingResult가 커맨드 객체 바로 뒤에 위치하지 않으면 스프링 MVC는 익셉션을 발생시킨다.  

#### 5.2. Errors와 BindingResult 인터페이스의 주요 메서드
<br/>
Validator는 객체 데이터에 오류가 있을 경우 Errors 객체에 오류를 등록한다. org.springframework.validation.Errors 인터페이스는 오류를 등록하기 위한 다양한 메서드를 제공하고 있으며, 주요 메서드는 다음과 같다.  
+ reject(String errorCode)  
전체 객체에 대한 글로벌 에러 코드를 추가한다.
+ reject(String errorCode, String defaultMessage)  
전체 객체에 대한 글로벌 에러 코드를 추가한다. 에러 코드에 대한 메시지가 존재하지 않을 경우 defaultMessage를 사용한다.
+ reject(String errorCode, Object[] errorArgs, String defaultMessage)  
전체 객체에 대한 글로벌 에러 코드를 추가한다. 메시지 인자로 errorArgs를 전달한다. 에러 코드에 대한 메시지가 존재하지 않을 경우 defaultMessage를 사용한다.
+ rejectValue(String field, String errorCode)  
필드에 대한 에러 크도를 추가한다.
+ rejectValue(String field, String errorCode, String defaultMessage)  
필드에 대한 에러 코드를 추가한다. 에러 코드에 대한 메시지가 존재하지 않을 경우 defaultMessage를 사용한다.
+ rejectValue(String field, String errorCode, Object[] errorArgs, String defaultMessage)  
필드에 대한 에러 코드를 추가한다. 메시지 인자로 errorArgs를 전달한다. 에러 코드에 대한 메시지가 존재하지 않을 경우 defaultMessage를 사용한다.  

reject() 메서드는 객체 자체의 에러 정보를 추가할 때 사용된다.  

rejectValue() 메서드는 객체의 개별 프로퍼티(필드)에 대한 에러 정보를 추가할 때 사용된다.  

글로벌 에러 정보나 특정 필드에 대한 에러 정보는 두 번 이상 추가될 수 있다.  

Errors 인터페이스는 에러가 발생했는지의 여부를 확인할 수 있도록 다음과 같은 메서드를 제공한다.  

+ boolean hasErrors()  
에러가 존재하는 경우 true를 리턴한다.
+ int getErrorCount()  
에러 개수를 리턴한다.
+ boolean hasGlobalErrors()  
reject() 메서드를 이용해서 추가된 글로벌 에러가 존재할 경우 true를 리턴한다.
+ int getGlobalErrorCount()  
reject() 메서드를 이용해서 추가된 글로벌 에러 개수를 리턴한다.
+ boolean hasFieldErrors()  
rejectValue() 메서드를 이용해서 추가된 에러가 존재할 경우 true를 리턴한다.
+ int getFieldErrorCount()  
rejectValue() 메서드를 이용해서 추가된 에러 개수를 리턴한다.  
+ boolean hasFieldErrors(String field)  
rejectValue() 메서드를 이용해서 추가한 특정 필드의 에러가 존재할 경우 true를 리턴한다.  
+ int getFieldErrorCount(String field)  
rejectValue() 메서드를 이용해서 추가한 특정 필드의 에러 개수를 리턴한다.  

org.springframework.validation.BindingResult 인터페이스는 Errors 인터페이스를 상속받았으며, 에러 메시지를 구하거나 검증 대상 객체를 구하는 등의 추가 기능을 정의하고 있다. BindingResult 인터페이스에 정의된 메서드 중 주요 메서드는 다음과 같다.
+ Object getTarget()  
검사 대상이 되는 객체를 구한다. 컨트롤러의 경우 커맨드 객체가 된다.  
+ String[] resolveMessageCodes(String errorCode)  
에러 코드를 메시지 코드로 변환한다.
+ String[] resolveMessageCodes(String errorCode, String field)  
에러 코드를 field에 해당하는 메시지 코드로 변환한다.  

##### 5.2.1. ValidationUtils 클래스를 이용한 값 검증
<br/>
ValidationUtils.rejectIfEmpty() 메서드와 값이 null이거나 길이가 0인 경우 에러 코드를 추가하며, rejectIfEmptyOrWhitesapce() 메서드는 값이 null이거나 길이가 0이거나 또는 값이 공백 문자로 구성되어 있는 경우 에러 코드를 추가한다.
+ rejectIfEmpty(Errors errors, String field, String errorCode)
+ rejectIfEmpty(Errors errors, String field, String errorCode, String defaultMessage)
+ rejectIfEmpty(Errors errors, String field, String errorCode, Object[] errorArgs)
+ rejectIfEmpty(Errors errors, String field, String errorCode, Object[] errorArgs, String defaultMessage)
+ rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode)
+ rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode, String defaultMessage)
+ rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode, Object[] errorArgs)
+ rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode, Object[] errorArgs, String defaultMessage)  

메서드 파라미터에서 errorArgs는 에러 메시지에 삽입될 인자 목록이고, defaultMessage는 에러 코드에 해당하는 에러 메시지가 존재하지 않을 때 사용할 기본 메시지이다.

#### 5.3. 에러 코드와 메시지
<br/>
Validator를 이용해서 오류를 확인하면, 오류 내용을 화면에 보여주고 싶을 것이다. 오류 내용을 알려줄 때 사용할 수 있는 것이 에러 코드다. 스프링 MVC는 에러 코드로부터 메시지를 가져오는 방법을 제공하고 있으며, 이 메시지를 응답 결과에 보여주는 방법 또한 제공하고 있다. 검증 과정에서 추가된 에러 메시지를 사용하려면 다음과 같은 코드를 작성해야 한다.  
+ 메시지를 읽어올 때 사용할 MessageSource를 스프링 설정에 등록한다.
+ MessageSource에서 메시지를 가져올 때 사용할 프로퍼티 파일을 작성한다.
+ JSP와 같은 뷰 코드에서 스프링이 제공하는 태그를 이용해서 에러 메시지를 출력한다.  

에러 메시지를 출력하려면 먼저 MessageSource를 등록해야 한다. 다음은 등록 예이다.

```java
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
	<property name="basename">
		<list>
			<value>message.error</value>
		</list>
	</property>
	<property name="defaultEncoding" value="UTF-8" />
</bean>
```

위 설정의 경우 message 패키지에 위치한 error.properties 파일 (또는 error&#95;언어.properties 파일)로부터 메시지를 읽어오도록 설정했다. error.properties 파일은 다음과 같이 에러 내용을 보여줄 때 사용할 메시지를 담고 있을 것이다.

```
# 메시지 파일 (error.properties)
required=필수 항목입니다.
minlength=최소 {1} 글자 이상 입력해야 합니다.
maxlength=최대 {1} 글자까지만 입력해야 합니다.
unsafe.password=암호는 알파벳과 숫자를 포함해야 합니다.
InvalidOrPassword=아이디와 암호가 일치하지 않습니다.
```

위 메시지 파일에서 minlength, required, unsafe.password 등은 메시지 코드가 된다.  

스프링 MVC는 에러 코드로부터 생성된 메시지 코드를 사용해서 에러 메시지를 출력하게 된다.  

글로벌 에러 코드인 경우 다음의 우선순위에 따라 메시지 코드를 생성한다.  

(1) 에러코드 + "." + 커맨드객체이름
(2) 에러코드  

예를 들어, Validator에서 글로벌 에러 코드를 아래와 같이 등록했다고 하자.  
```java
@RequestMapping(method = RequestMethod.POST)
public String login(LoginCommand loginCommand, Errors errors) {

	...

	try {
		authenticator.authenticate(loginCommand.getEmail(), loginCommand.getPassword());
		return "redirect:/index.jsp";
	} catch (AuthenticationException ex) {
		errors.reject("invalidOrPassword");
		return LOGIN_FORM;
	}
}
```

위 코드에서 사용한 글로벌 에러 코드는 "invalidOrPassword"이고, 커맨드 객체의 모델 이름은 "loginCommand"다. 따라서, 메시지를 읽어올 때에는 다음의 두 메시지 코드를 사용한다.  

(1) invalidOrPassword.loginCommand  
(2) invalidOrPassword  

에러 메시지를 읽어올 때, 먼저 메시지 코드가 invalidOrPassword.loginCommand인 메시지를 찾는다. 존재하면 그 메시지를 사용하고, 존재하지 않으면 InvalidOrPassword를 사용한다. 따라서, 메시지 포로퍼티 파일에 다음과 같이 invalidOrPassword 메시지만 정의되어 있으면, 에러 메시지를 출력할 때 invalidOrPassword 메시지의 내용을 사용한다.  

Errors.rejectValue()를 이용해서 생성한 에러 코드는 다음의 우선순위를 사용해서 메시지 코드를 생성한다.

(1) 에러코드 + "." + 커맨드객체이름 + "." + 필드명  
(2) 에러코드 + "." + 필드명  
(3) 에러코드 + "." + 필드타입
(4) 에러코드  

필드가 List나 목록인 경우 다음의 순서를 사용해서 메시지 코드를 생성한다.  

(1) 에러코드 + "." + 커맨드객체이름 + "." + 필드명[인덱스].중첩필드명  
(2) 에러코드 + "." + 커맨드객체이름 + "." + 필드명.중첩필드명  
(3) 에러코드 + "." + 필드명[인덱스].중첩필드명  
(4) 에러코드 + "." + 필드명.중첩필드명  
(5) 에러코드 + "." + 중첩필드명  
(6) 에러코드 + "." + 필드타입
(7) 에러코드  

에러 메시지 프로퍼티 파일이 아래와 같고, email과 password 프로퍼티에 값이 없어서 둘 다 에러 코드로 "required"를 등록했다고 하자.

```
required=필수 항목입니다.
required.email=이메일을 입력하세요.
```
이 경우, 메시지 코드 우선순위에 따라 "email" 프로퍼티의 에러 메시지는 "required.email" 메시지를 이용하고 "password" 프로퍼티의 에러 메시지는 "required" 메시지를 이용하게 된다.  

##### 5.3.1. 에러 메시지 출력
<br/>
스프링 MVC는 JSP에서 에러 메시지를 출력할 수 있도록 커스텀 태그를 제공하고 있다. JSP에서 에러 메시지를 출력하는 가장 쉬운 방법은 스프링이 제공하는 &#60;form:errors&#62; 태그를 함께 사용하는 것이다.  

```html
<%@ page contentType="text/html; charset=utf-8" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<!DOCTYPE html>
<html>
	<head><title>로그인</title></head>
	<body>
		<form:form commandName="loginCommand">
			<form:errors element="div" />
			<label for="email">이메일</label>:
			<input type="text" name="email" id="email" value="${loginCommand.email}">
			<form:errors path="email"/> <br>

			<label for="password">암호</label>
			<input type="password" name="password" id="password">
			<form:errors path="password"/> <br>
			<br/>

			<input type="submit" value="로그인">
		</form:form>
	</body>
</html>
```
스프링의 &#60;form:form&#62; 커스텀 태그는 HTML의 &#60;form&#62; 태그 대신 사용할 수 있는데, 이 태그를 사용하면 스프링의 폼 관련 태그에서 커맨드 객체 정보를 사용할 수 있다. commandName 속성 값으로 "loginCommand"를 지정했는데, 이는 &#60;form:태그&#62;에서 이름이 "loginCommand"인커맨드객체를 사용한다는 것을 뜻한다.  

&#60;form:errors&#62; 태그는 &#60;form:form&#62; 태그에서 지정한 커맨드 객체의 에러 코드를 이용해서 에러 메시지를 출력한다. 예를 등러, 글로벌 에러 코드에 "invalidOrPassword"가 존재한다고해보자. 이 경우 &#60;form:errors&#62; 태그는 "invalidOrPassword.loginCommand" 메시지를 읽어와 출력한다. 만약 이 메시지가 존재하지 않으면 "invalidOrPassword" 메시지를 읽어온다.  

커맨드 객체의 "email" 프로퍼티와 관련된 에러 메시지를 출력한다. 만약 "email" 프로퍼티와 관련된 에러 코드가 존재하면, 해당 에러 코드를 이용해서 알맞은 메시지를 출력한다.  

&#60;form:form&#62; 태그를 사용하지 않고 HTML의 &#60;form&#62; 태그를 사용하고 싶다면 어떻게 해야 할까? 이런 경우에는 &#60;spring:hasBindErrors&#62; 태그와 &#60;form:errors&#62; 태그를 사용하면 된다.  

```html
<%@ page contentType="text/html; charset=utf-8" %>
<@# taglib prefix="spring" uri="http://www.springframework.org/tags" %>
<@% taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<!DOCTYPE html>
<html>
	<spring
	<head><title>로그인</title></head>
	<body>
		<spring:hasBindErrors name="memberInfo" />
		<form method="post">
			<label for="email">이메일</label>:
			<input type="text" name="email" id="email" value="${loginCommand.email}">
			<form:errors path="memberInfo.email"/> <br>

			<label for="password">암호</label>
			<input type="password" name="password" id="password">
			<form:errors path="memberInfo.password"/> <br>
			<br/>

			<input type="submit" value="로그인">
		</form:form>
	</body>
</html>
```
위 코드에서 &#60;spring:hasBindErrors&#62; 태그의 name 속성으로 커맨드 객체 이름을 입력하면, &#60;form:errors&#62; 태그를 이용해서 커맨드 객체에 대한 에러 메시지를 출력할 수 있다. 앞서 &#60;form:form&#62; 태그를 사용할 때와 차이점은 &#60;form:errors&#62; 태그의 path 속성에 커맨드 객체 이름을 함께 사용해야 한다는 것이다.

#### 5.4. &#64;Valid 애노페이션과 &#64;InitBinder 애노테이션을 이용한 검증 실행
<br/>
JSR 303 표준에 정의된 &#64;Valid 애노테이션을 이용하면 커맨드 객체를 검사하는 코드를 직접 호출하지 않고 스프링 프레임워크가 호출하도록 설정할 수 있다. JSR 303은 Bean Validation API로서 이 표준에 정의된 &#64;Valid 애노테이션을 사용해서 커맨드 객체의 유효성을 검증한다는 것을 표시한다.

&#64;Valid 애노테이션을 사용하려면 JSR 303 API를 클래스패스에 추가해 주어야 한다. 메이븐을 사용할 경우 다음의 의존 설정을 추가해주면 된다.  

```xml
<dependency>
	<groupId>javax.validation</groupId>
	<artifactId>validation-api</artifactId>
	<version>1.0.0.GA</version>
</dependency>
```
스프링 MVC는 JSR 303의 &#64;Valid 애노테이션과 스프링 프레임워크의 &#64;InitBinder 애노테이션을 이용해서 스프링프레임워크가 유효성 검사 코드를 실행하도록 할 수 있다.  

```java
@Controller
public class 클래스명 {
	
	...

	@RequestMapping(...)
	public String 메소드명(@Valid 클래스명 변수명) {
		
		...
	
	}

	...

	@InitBinder
	protected void 메소드명(WebDataBinder binder) {
		
		...

		binder.setValidator(new LoginCommandValidator());

		...

	}

	...

}
```

스프링은 &#64;InitBinder 애노테이션이 적용된 메서드를 이용해서 폼과 커맨드 객체 사이의 매핑을 처리해주는 WebDataBinder를 초기화할 수 있도록 하고 있다. WebDataBinder.setValidator()를 통해 커맨드 객체의 유효성 여부를 검사할 때 사용할 Validator를 설정하게 된다. 스프링 MVC는 &#64;RequestMapping 애노테이션이 설정된 메서드를 실행하기 전에 설정한 Validator 객체를 이용해서 &#64;Valid 애노테이션이 붙은 커맨드 객체를 검증한다.  

#### 5.4. 글로벌 Validator와 컨트롤러 Validator
<br/>
글로벌 Validator를 사용하면 한 개의 Validator를 이용해서 모든 커맨드 객체를 검증할 수 있다. 글로벌 Validator를 적용하는 방법은 간단하다. 다음과 같이 &#60;mvc:annotation-driven&#62; 태그의 validator 속성에 글로벌 Validator로 사용할 빈을 등록해주면 된다.

```xml
<mvc:annotation-driven validator="빈식별자" />
<bean id="빈식별자" class="Validator 구현 클래스명" />
```
이 경우 해당 Validator 구현 클래스를 이용해서 &#64;Valid 애노테이션이 붙은 커맨드 객체를 검사하게 된다.  

글로벌 Validator가 커맨드 객체에 대한 검증을 지원하지 않으면(즉, 글로벌 Validator의 supports() 메서드가 false를 리턴하면), 글로벌 Validator는 커맨드 객체를 검증하지 않는다.  

글로벌 Validator를 사용하지 않고 다른 Validator를 사용하려면 &#64;InitBinder가 적용된 메서드에서 WebDataBinder의 setValidator() 메서드를 이용하면 된다.  

글로벌 Validator를 사용하면서 컨트롤러를 위한 Validator를 추가로 사용하고 싶다면 setValidator() 대신 addValidator() 메서드를 사용해서 Validator를 등록해주면 된다.

```java
@InitBinder
protected void 메소드명(WebDataBinder binder) {
	// 글로벌 Validator와 함께 사용
	binder.addValidator(new Validator구현클래스명());
}
```

&#64;EnableWebMvc 애노테이션을 사용했다면, 다음과 같이 WebMvcConfigurerAdapter를 상속받은 &#64;Configuration 클래스에서 글로벌 Validaotr 객체를 생성하도록 getValidator() 메서드를 재정의한다.  

```java
@Configuration
@EnableWebMvc
public class 빈설정클래스명 extends WebMvcConfigurerAdapter {

	...

	@Override
	public Validator getValidator() {
		return new Validator구현클래스명();
	}

	...

}
```

&#60;mvc:annotation-driven&#62; 태그의 validator 속성을 지정하지 않거나 getValidator() 메서드를 재정의하지 않으면 JSR 303 애노테이션을 지원하는 Validator를 글로벌 Validator로 등록한다.  

#### 5.6. &#64;Valid 애노테이션 및 JSR 303 애노테이션을 이용한 값 검증 처리
<br/>
&#64;Valid 애노테이션은 Bean Validation API(JSR 303)에 정의되어 있는데, 이 API에는 &#64;Valid 애노테이션 뿐만 아니라 &#64;NotNull, &#64;Digits, &#64;Size 등의 애노테이션을 제공하고 있다. 이들 애노테이션을 사용하면 Validator 클래스 작성 없이 애노테이션만으로 커맨드 객체의 값 검증을 처리할 수 있다.  

JSR 303이 제공하는 애노테이션을 이용해서 커맨드 객체의 값을 검증하는 방법은 다음과 같다.  
+ 커맨드 객체에 &#64;NotNull, &#64;Digits 등의 애노테이션을 이용해서 검증 규칙을 설정한다.
+ LocalValidatorFactoryBean 클래스를 이용해서 JSR 303 프로바이더를 스프링의 Validator로 등록한다.
+ 컨틀롤러가 두 번째에서 생성한 빈을 Validator로 사용하도록 설정한다.  

가장 먼저 할 작업은 JSR 303 API와 JSR 303 프로바이더를 설치하는 것이다. 예를 들어, Hibernate Validator를 프로바이더 구현체로 사용하고 싶다면, Hibernate Validator 관련 jar 파일을 클래스패스에 등록해주면 된다. 다음은 메이븐을 사용하는 경우 의존 설정 예이다.  

```xml
<dependency>
	<groupId>javax.validation</groupId>
	<artifactId>validation-api</artifactId>
	<version>1.0.0.GA</version>
</dependency>
<dependency>
	<groupId>org.hibernate</groupId>
	<artifactId>hibernate-validator</artifactId>
	<version>4.3.1.Final</version>
</dependency>
```
스프링 JSR 303 프로바이더 구현체로 Hibernate Validator를 사용할 경우, &#64;Range나 &#64;NotEmpty와 같은 추가적인 애노테이션을 사용할 수 있다.  

JSR 303 애노테이션을 적용했으면, 그 다음으로 할 작업은 커맨드 객체의 값을 검증할 때 JSR 303 프로바이더를 사용하도록 설정하는 것이다. JSR 303 프로바이더를 Validator로 사용하려면 LocalValidatorFactoryBean 클래스를 빈으로 등록하고, 이 빈을 Validator로 사용하면 된다.  

&#60;mvc:annotation-driven&#62; 태그를 사용하면 기본으로 LocalValidatorFactoryBean이 생성한 JSR 303 Validator를 글로벌 Validator로 등록해준다. 따라서 다음과 같이 &#60;mvc:annotation-driven&#62; 태그를 설정하면 JSR 303 애노테이션을 사용하는 커맨드 객체에 대한 값 검증을 할 수 있다.  

남은 작업은 JSR 303 애노테이션을 사용하는 커맨드 객체 앞에 &#64;Valid 애노테이션을 붙여서, JSR 303 지원 Validator가 값을 검증하도록 만들면 된다.  

JSR 303의 기본 에러 메시지가 아니라 스프링이 제공하는 메시지를 이용하고 싶다면, MessageSource가 사용하는 프로퍼티 파일에 다음 규칙을 따르는 메시지 코드를 등록해주어야 한다.  

(1) 애노테이션이름.커맨드객체모델명.프로퍼티명  
(2) 애노테이션이름.프로퍼티명  
(3) 애노테이션  

따라서, 메시지 프로퍼티 파일에 위 규칙에 맞게 에러메시지를 입력해주면, JSR 303의 기본 에러메시지 대신 원하는 에러 메시지를 출력할 수 있다.  

##### 5.6.1. JSR 303의 주요 애노테이션
<br/>
JSR 303에서 제공하는 값 규칙 설정과 관련된 주요 애노테이션은 다음과 같다.  

<table style="border: 1px solid #444444; border-collapse;">
	<thead>
		<tr>
			<td>애노테이션</td>
			<td>주요 속성 (괄호는 기본 값)</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>&#64;NotNull</td>
			<td></td>
			<td>값이 null이면 안 된다.</td>
		</tr>
		<tr>
			<td>&#64;Size</td>
			<td>min: 최소 크기, int(0)<br/>max: 최대 크기, int(int의 최대 값)</td>
			<td>값의 크기가 min부터 max 사이에 있는지 검사한다.<br/>String인 경우 문자열의 길이를 검사한다.<br/>콜렉션인 경우 구성 요소의 개수를 검사한다.<br/>배열인 경우 배열의 길이를 검사한다.<br/>값이 null인 경우 유효한 것으로 판단한다.</td>
		</tr>
		<tr>
			<td>&#64;Min<br/>&#64;Max</td>
			<td>value: 최소 또는 최대 값, long</td>
			<td>숫자의 값이 지정한 값 이상(&#64;Min) 또는 이하(&#64;Max)여야 한다. BigDecimal, BigInteger, 정수 타입(int, long 등)  및 래퍼타입에 적용된다. (double과 float은 지원하지 않는다.)<br/>값이 null인 경우 유효한 것으로 판단한다.</td>
		</tr>
		<tr>
			<td>&#64;DecimalMin<br/>&#64;DecimalMax</td>
			<td>value: 최소 또는 최대 값, String(BigDecimal 형식으로 값 표현)</td>
			<td>숫자의 값이 지정한 값 이상(&#64;Min)또는이하(&#64;Max)인지 검사한다. BigDecimal, BigInteger, String, 정수 타입(int, long 등) 및 래퍼타입에 적용된다. (double과 float은 지원하지 않는다.)<br/>값이 null인 경우 유효한 것으로 판단한다.</td>
		</tr>
		<tr>
			<td>&#64;Digits</td>
			<td>integer: 정수부분 숫자 길이, int<br/>fraction: 소수부분 숫자 길이, int</td>
			<td>숫자의 정수 부분과 소수점부분의 길이가 범위에 있는지 검사한다. BigDecimal, BigInteger, String, 정수 타입(int, long 등) 및 래퍼타입에 적용된다.<br/>값이 null인 경우 유효한 것으로 판단한다.</td>
		</tr>
		<tr>
			<td>&#64;Pattern</td>
			<td>regexp: 정규표현식, String</td>
			<td>문자열이 지정한 패턴에 일치하는지 검사한다.<br/>값이 null인 경우 유효한 것으로 판단한다.</td>
		</tr>
	</tbody>
</table>
<br/>
위의 표를 보면 &#64;NotNull을 제외한 나머지 애노테이션은 검사 대상 값이 null인 경우 유효한 것으로 판단하는 것을 알 수 있다. 따라서, 필수 입력 값을 검사할 때에는 다음과 같이 &#64;NotNull과 &#64;Size를 함께 사용해주어야 한다.
```java
@NotNull
@Size(min=1)
private String title;
```
무심코 &#64;NotNull만 사용하면, title의 값이 빈 문자열일 경우 값 검사 과정에서 통과된다.  

JSR 303에 정의된 애노테이션의 이런 단점을 보완하기 위해 Hibernate Validator 모듈은 추가로 다음의 애노테이션을 제공하고 있다. (org.hibernate.validator.constrains 패키지에 정의되어 있다.)  
<table>
	<thead>
		<tr>
			<td>&#64;NotEmpty</td>
			<td></td>
			<td>String인 경우 빈 문자열이 아니여야 하고, 콜렉션이나 배열인 경우 크기가 1 이상이어야 한다.</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>&#64;NotBlank</td>
			<td></td>
			<td>&#64;NotEmpty와 동일하다. &#64;NotEmpty와 차이점은 String의 경우, 뒤의 공백문자를 무시한다는 점이다.</td>
		</tr>
		<tr>
			<td>&#64;Length</td>
			<td>min: 최소 길이, int(0)<br/>max: 최대 길이, int(int의 최대 값)</td>
			<td>문자열의 길이가 min과 max 사이에 있는지 검사한다.</td>
		</tr>
		<tr>
			<td>&#64;Range</td>
			<td>min: 최소 값, long(0)<br/>max: 최대 값, long(long의 최대 값)</td>
			<td>숫자 값이 min과 max 사이에 있는지 검사한다. 값의 타입이 String인 경우 숫자로 변환한 결과를 이용해서 검사한다.</td>
		</tr>
		<tr>
			<td>&#64;Email</td>
			<td></td>
			<td>값이 올바른 이메일 주소인지 검사한다.</td>
		</tr>
		<tr>
			<td>&#64;URL</td>
			<td></td>
			<td>값이 올바른 URL인지 검사한다.</td>
		</tr>
	</tbody>
</table>
<br/>

### 6. 요청 파라미터의 값 변환 처리
<br/>
#### 6.1. WebDataBinder/&#64;InitBinder와 PropertyEditor를 이용한 타입 변환
<br/>
앞서 커맨드 객체를 검증할 때 WebDataBinder를 이용해서 Validator를 등록하는 방법을 설명했었는데, WebDataBinder는 커맨드객체의 값 검증 뿐만 아니라 웹 요청 파라미터로부터 커맨드 객체를 생성할 때에도 사용된다. WebDataBinder는 커맨드객체를 생성하는 과정에서 String 타입의 요청 파라미터를 커맨드 객체 프로퍼티의 타입으로 변환한다. 이때 WebDataBinder는 PropertyEditor와 ConversionService를 사용한다.  

같은 타입이라고 하더라도 컨트롤러 클래스마다 다른 변환 규칙을 사용해야 할 때가 있는데, 이런 경우에는 컨트롤러마다 개별적으로 PropertyEditor를 등록하는 방법을 사용하면 된다. WebDataBinder는 PropertyEditor를 등록할 수 있는 registerCustomEditor() 메서드를 제공하고 있으며, &#64;InitBinder 메서드에서 이 메서드를 이용해서 필요한 PropertyEditor를 등록하면 된다.  

예를 들어, 스프링은 문자열을 java.util.Date 타입으로 변환해주는 CustomDateEdiotr를 제공하고 있는데, CustomDateEditor를 사용하면 특정 형식을 갖는 요청 파라미터를 Date 타입으로 변환할 수 있다. 다음 코드를 보자.  

```java
@Controller
@RequestMapping(...)
public class 클래스명 {

	...

	@InitBinder
	protected void 메소드명(WebDataBinder binder) {
		// 커맨드 객체의 Date 타입 프로퍼티에
		// 파라미터 값을 할당할 때 사용
		CustomDateEditor dateEditor = new CustomDateEditor(new SimpleDateFormat("yyyyMMdd"), true);
		binder.registerCustomEditor(Date.class, dateEditor);

		CustomDateEditor dateEditor1 = new CustomDateEditor(new SimpleDateFormat("yyyyMMdd"), true);
		binder.registerCustomEditor(Date.class, "파라미터명1", dateEditor1);

		CustomDateEditor dateEditor2 = new CustomDateEditor(new SimpleDateFormat("HH:mm"), true);
		binder.registerCustomEditor(Date.class, "파라미터명2", dateEditor2);
	}

	...

}
```
CustomDateEditor는 문자열을 Date로 변환할 때 사용할 SimpleDateFormat 객체를 생성자를 통해서 전달받는다. CustomDateEditor 객체를 생성했다면 WebDataBinder의 registerCustomEditor() 메서드를 이용해서 에디터를 등록하면 된다. 위 코드에서 이 메서드를 호출할 때 첫 번째 파라미터로 Date.class를 주었는데, 이는 Date 타입의 프로퍼티의 값을 변환할 때 dateEditor를 사용한다는 것을 의미한다.  

만약 프로퍼티마다 다른 커스텀 에디터를 사용하고 싶다면 다음과 같이 프로퍼티 이름을 지정해주면 된다.  

위 코드를 봅면 CustomDateEditor 생성자의 두 번째 파라미터로 true를 전달했다. 이 값이 true면 요청 파라미터의 값이 null이거나 빈 문자열("")일 때 변환 처리를 하지 않고 null을 값으로 할당한다. 이 값이 false면 타입 변환 과정에서 에러가 발생하고, 에러 코드로 "typeMismatch"가 추가된다.  

#### 6.2. WebDataBinder와 ConversionService
<br/>
&#60;mvc:annotation-driven&#62; 태그는 정말 많은 설정을 대신 해준다. &#60;mvc:annotation-driven&#62;이 설정하는 빈 정보를 직접 설정하면 거의 80줄에 육박하는 XML 설정을 작성하게 된다. 이렇게 해주는 많은 설정 중에는 다음의 내요잉 포함되어 있다.  
```xml
<!-- mvc:annotation-driven 태그가 생성하는 빈 객체의 일부 설정 -->
<!-- 이름이 conversionService가 아니므로 설정 정보를 변환할 때는 사용되지 않음 -->
<bean id="formattingConversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
</bean>

<bean id="configurableWebBindingInitializer" class="org.springframework.web.bind.support.ConfigurableWebBindingInitailizer">
	<property name="conversionService" ref="formattingConversionService" />
	
	...

</bean>

<bean id="requestMappingHandlerAdapter" class="org.springframework.web.servlet.mvc.method.RequestMappingHandlerAdapter">

	...

	<property name="webBindingInitializer" ref="configurableWebBindingInitializer" />
</bean>
```
위 코드에서 ConfigurableWebBindingInitializer는 WebDataBinder를 초기화해주는 기능을 제공한다. 클라이언트의 요청을 컨트롤러 객체에 전달해주는 RequestMappingHandlerAdapter는 이 WebBindingInitializer를 이용해서 컨트롤러에 전달할 WebDataBinder 객체를 생성하고 초기화한다.  

그런데, 위 설정을 보면 ConfigurableWebBindingInitializer를 설정할 때 FormattingConversionServiceFactoryBean이 생성한 ConversionService를 설정하고 있는 것을 알 수 있다. ConfigurableWebBindingInitializer는 WebDataBinder를 생성할 때 이 ConversionService 객체를 전달한다. 즉, WebDataBinder는 PropertyEditor 뿐만 아니라 ConversionService를 이용해서 요청 파라미터를 알맞은 타입으로 변환한다.  

&#60;mvc:annotation-driven&#62; 태그(또는 &#64;EnableWebMvc)를 사용하면 DefaultFormattingConversionService가 WebDataBinder의 ConversionService로 사용되는데, 이는 &#64;DateTimeFormat 애노테이션과 &#64;NumberFormat 애노테이션을 이용해서 요청 파리머터를 날짜/시간 타입이나 숫자 타입으로 변환할 수 있다는 것을 의미한다. (DefaultFormattingConversionService는 기본적으로 &#64;DateTimeFormat과 &#64;NumberFormat을 위한 Formatter를 포함하고 있다.)  

##### 6.2.1.&#64;DateTimeFormat 애노테이션을 이용한 날짜/시간 변환
<br/>
&#64;DateTimeFormat 애노테이션은 특정 형식을 갖는 요청 파라미터를 java.util.Date 타입이나 java.time.LocalDate 타입과 같이 날짜/시간 타입으로 변환할 때 사용된다. 예를 들어, 생일을 입력받기 위해 다음과 같은 &#60;input&#62; 태그를 사용했다고 하자.  

```html
<label for="birthday">생일</label>: 형식: YYYYMMDD, 예: 20140101
<input type="text" name="birthday" id="birthday" />
<form:errors path="memberInfo.birthday"/> <br/>
```

```java
public class MemberRegistRequest {

	...

	private Date birthday;

	public Date getBirthday() {
		return birthday;
	}
	public void setBirthday(Date birthday) {
		this.birthday = birthday;
	}
}
```

```java
@RequestMapping(method = RequestMethod.POST)
public String regist(@ModelAttribute("memberInfo") MemberRegistRequest memRegReq, BindingResult bindingResult) {
	new MemberRegistValidator().validate(memRegReq, bindingResult);

	...
}
```
여기서 문제는 "20140101" 형식으로 전송되는 요청 파라미터를 Date 타입으로 변환해주어야 한다는 점이다. 앞서 살펴본 CustomDateEditor를 사용해도 되지만, @DateTimeFormat 애노테이션을 사용하면 좀 더 간단하게 처리할 수 있다. 다음과 같이 날짜/시간 타입 필드에 @DateTimeFormat 애노테이션을 사용해주면 끝난다.  

```java
public class MemberRegistRequest {

	...

	@DateTimeFormat(pattern="yyyyMMdd")
	private Date birthday;

	...

}
```
스프링은 요청 파라미터의 값을 커맨드 객체의 프로퍼티로 복사하는 과정에서 시간 타입을 갖는 프로퍼티에 &#64;DateTimeFormat 애노테이션이 붙어 있으면, 애노테이션에 설정한 형식을 이용해서 요청 파라미터 값을 변환한다. 위 코드의 경우 &#64;DateTimeFormat의 pattern 값으로 "yyyyMMdd"를 사용했으므로 요청 파리미터가 "20140101"일 경우, 2014년 1월 1일에 해당하는 Date 객체가 birthday 필드에 할당된다.  

&#64;DateTimeFormat에서 사용할 패턴을 "yyyyMMdd"로 지정했는데, 요청 파라미터의 값이 "2014-01-01"과 같이 다른 형식인 경우 해당 필드의 에러 코드로 "typeMismatch"가 추가된다. 따라서, 잘못된 형식일 때알맞은 메시지를 출력하고 싶다면, 메시지 프로퍼티파일에 "typeMismatch" 메시지를 추가해주면 된다.  

&#60;mvc:annotation-driven&#62;이나 &#64;EnableWebMvc를 이용한 스프링 설정 방법을 설명하고 있는데, 이를 사용하지 않을 경우 &#64;DateTimeFormat이 기본으로 처리되지 않는다. MVC 스키마나 애노테이션 설정을 사용하지 않고 직접 RequestMapping 등의 빈 객체를 등록해서 설정을 했다면 &#64;DateTimeFormat을 사용하기 위해 추가 설정을 해주어야 한다.  

만약 Errors나 BindingResult 타입의 파라미터가 없다면, 스프링은 타입 변환 실패시 400 응답 에러를 발생시킨다. 예를 들어, 아래 코드처럼 커맨드 객체 다음에 Errors 타입의 파라미터가 없다고 해보자.  
```java
// Errors/BindingResult 타입 파라미터가 없을 경우,
// 잘못된 형식의 요청 파라미터가 전송되면 400 응답 에러를 발생시킨다.
@RequestMapping(method = RequestMethod.POST)
public String regist(@ModelAttribute("memberInfo") MemberRegistRequest regReq) {
	...
}
```
이 상태에서 &#64;DateTimeFormat(pattern="yyyyMMdd")가 붙은 필드에 해당하는 요청 파라미터의 값으로 "1234-5-6"과 같이 잘못된 형식의 값이 오면, regist() 메서드를 실행하지 않고 400 에러 코드를 웹 브라우저에 응답한다.  

##### 6.2.2. &#64;DateTimeFormat 애노테이션의 속성과 설정 방법
<br/>
&#64;DateTimeFormat 애노테이션은 날짜/시간 형식을 지정하기 위해 세 개의 속성을 사용한다. 세 개의 속성은 함께 사용할 수 없으며 필요에따라 한 개의 속성만 사용해야 한다. 다음은 각 속성에서 사용할 수 있는 값을 정리한 것이다.  

<table>
	<thead>
		<tr>
			<td>속성</td>
			<td>표</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>style</td>
			<td>S, M, L, F를 이용해서 날짜/시간을 표현한다. 날짜와 시간이 각각 한글자를 사용하며, 날짜나 시간을 생략할 경우 '-'를 사용한다. "SS", "S-", "-F"과 같은 값을 갖는다. 각 문자는 다음을 의미한다.<br/> 
+ S : 짧은 표현
+ M : 중간 표현
+ L : 긴 표현
+ F : 완전한 표현</td>
		</tr>
		<tr>
			<td>iso</td>
			<td>&#64;DateTimeFormat의 중첩 타입인 ISO 열거 타입에 정의된 값을 사용한다.<br/>
+ ISO.DATE : yyyy-MM-dd 형식 (2014-01-01)
+ ISO.TIME : HH:mm:ss.SSSZ 형식 (13:40:50.113+80:30)
+ ISO.DATE&#95;TIME : yyyy-MM-dd'T'HH:mm:ss.SSSZ 형식</td>
		</tr>
		<tr>
			<td>pattern</td>
			<td>java.text.DateFormat에 정의된 패턴을 사용한다.</td>
		</tr>
	</tbody>
</table>
<br/>
##### 6.2.3. &#64;NumberFormat 애노테이션을 이용한 숫자 변환
<br/>
&#64;NumberFormat 애노테이션은 특정 형식을 갖는 문자열을 숫자 타입으로 변환할 때 사용된다. (org.springframework.format.annotation 패키지에 속해 있다.) 사용 방법은 &#64;DateTimeFormat 애노테이션과 비슷하며, &#64;NumberFormat 애노테이션의 속성은 다음과 같다. 두 속성 중 한 속성만 사용해야 한다.  

<table>
	<thead>
		<tr>
			<td>속성</td>
			<td>값</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>pattern</td>
			<td>java.text.NumberFormat에 따른 숫자 형식을 입력한다.</td>
		</tr>
		<tr>
			<td>style</td>
			<td>&#64;NumberFormat의 중첩 타입인 Style 열거 타입에 정의된 값을 사용한다.<br/>
+ Style.NUMBER : 현재 로케일을 위한 숫자 형식
+ Style.CURRENCY : 현재 로케일을 위한 통화 형식
+ Style.PERCENT : 현재 로케일을 위한 백분율 형</td>
		</tr>
	</tbody>
</table>


#### 6.3. 글로벌 변환기 등록하기
<br/>
WebDataBinder/&#64;InitBinder를 이용하는 방법은 단일 컨트롤러에만 적용된다. 경우에 따라 전체 컨트롤러에 동일한 변환 방식을 적용하고 싶을 때가 있는데, 이 때에는 ConversionService를 직접 생성해서 등록하는 방법을 사용한다. ConversionService를 설정할 때에는 스프링MVC가 기본으로 사용하는 FormattingConversionServiceFactoryBean를 사용하면 된다. 다음은 설정 예이다.  
```xml
<mvc:annotation-driven conversion-service="빈식별자" />

<bean id="빈식별자" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
	<property name="formatters">
		<set>
			<bean class="Formatter구현완전한클래스명" />
		</set>
	</property>
</bean>
```
&#64;EnableWebMvc 애노테이션을 사용한다면, 다음과 같이 WebMvcConfigurerAdapter 클래스의 addFormatters() 메서드를 재정의해서 registry에 Formatter를 등록하면 된다.  
```java
@Configuration
@EnableWebMvc
public class 빈설정클래스명 extends WebMvcConfigurerAdapter {

	...

	@Override
	public void addFormatters(FormatterRegistry registry) {
		registry.addFormatter(new Formatter구현클래스명());
	}

	...

}
```

### 7. HTTP 세션 사용하기
<br/>
#### 7.1. HttpSession을 직접 사용하기
<br/>
HttpSession을 사용하는 가장 손쉬운 방법은 HttpSession을 컨트롤러 메서드의 파라미터로 지정하는 것이다.  

```java
@Controller
@RequestMapping(...)
public class 클래스명 {

	...

	@RequestMapping(value = "경로명1", method = RequestMethod.POST)
	public String 메소드명(..., HttpSession session) {
	
		...

	}

	...

	@RequestMapping(value = "경로명2", method = RequestMethod.POST)
	public String 메소드명(..., HttpServletRequest request) {
	
		...

		HttpSession session = request.getSession();

		...

	}
}
```
HttpSession 타입의 파라미터가 존재하면 스프링 MVC는 HttpSession을 생성해서 파라미터 값으로 전달한다. 기존에 세션이 존재하면 그 세션을 전달하고, 세션이 존재하지 않았으면 새로운 세션을 생성해서 전달한다.  

만약 상황에 따라 세션을 생성하고 싶다면, HttpSession을 파라미터로 받으면 안 된다. HttpServletRequest를 받아서 상황에 따라 세션을 직접 생성해주어야 한다.  

#### 7.2. &#64;SessionAttributes 애노테이션을 이용한 모델과 세션 연동
<br/>
스프링은 임시 용도로 사용될 데이터를 세션에 보관할 때 사용할 수 있는 &#64;SessionAttributes 애노테이션을 제공하고 있다. 이 애노테이션을 이요해서 각 화면 간 모델 데이터를 공유하는 방법은 다음과 같다.
+ 클래스에 &#64;SessionAttributes를 적용하고, 세션으로 공유할 객체의 모델 이름을 지정한다.
+ 컨트롤러 메서드에서 객체를 모델에 추가한다.
+ 공유한 모델의 사용이 끝나면 SessionStatus를 시용해서 세션에서 객체를 제거한다.  

세션을 이용해서 특정 객체를 공유하려면, 객체를 세션에 보관할 때 사용할 속성 이름이 필요하다. &#64;SessionAttributes 애노테이션을 사용해서 이 이름을 지정하게 된다. 
```java
@Controller
@SessionAttributes("세션명")
public class 클래스명 {
	
	...

	@ModelAttribute("세션명")
	public 모델클래스명 메소드명1() {
		return new 모델클래스명();
	}

	...

	@RequestMapping("경로명1")
	public String 메소드2(Model model) {
		model.addAttribute("세션명", new 모델클래스명());

		...
	}

	...

	@RequestMapping(value = "경로명2", method = RequestMethod.POST)
	public String 메소드명3(@ModelAttribute("세션명") 모델클래스명 변수명, BindingResult result, SessionStatus sessionStatus) {
		...

		sessionStatus.setComplete();

		...

	}

	...

}
```
&#64;SessionAttributes가 제대로 동작하려면 모델에 같은 이름을 갖는 객체를 추가해야 한다. 보통 첫 번째 단계를 처리하는 컨트롤러 메서드에서 모델에 객체를 추가한다.

위 코드처럼 &#64;ModelAttribute가 적용된 메서드에서 모델 객체를 생성하면, &#64;RequestMapping 메서드에서 Model에 객체를 추가할 필요가 없다. &#64;ModelAttribute가 적용된 메서드가 실행될 때 매번 객체가 생성되기 때문에 데이터 공유를 위해 세션에 객체를 보관하는데 문제가 있을 거라 생각할 수도 있을 것이다. 하지만, 스프링은 &#64;ModelAttribute가 적용된 메서드를 실행하기 전에 세션에 동일 이름을 갖는 객체가 존재하면 그 객체를 모델 객체로 사용한다.  

세션을 이용한 객체 공유가 끝나면 SessionStatus의 setComplete() 메서드를 호출하면 된다.  

SessionStatus.setComplete() 메서드를 실행하면 세션에서 객체를 제거한다. 단, SessionStatus.setComplete()는 세션에서 객체를 제거할 뿐 모델에서 제거하지는 않는다. 따라서, 뷰 코데에서는 모델의 값을 사용할 수 있다.  

임시 데이터를 보관하기 위해 세션을 사용한다는 것은 사용자마다 세션이 생성된다는 것을 의미한다. 즉, 접속자가 늘어날수록 세션 객체 개수가 증가하고 이에 따라 세션 객체가 차지하는 메모리 비중이 높아진다. 예를 들어, 포털 사이트처럼 동시에 접속하는 사용자수가 많은 곳에서 세션을 사용하면 세션 객체로 인해 메모리 부족 현상이 발생할 수 있다. 따라서 트래픽이 높을 것으로 예상되는 곳에서 세션을 사용할 때에는 메모리가 부족하지 않을 지 따져봐야 하며, 메모리가 부족할 것으로 예상되면 세션 대신에 데이터베이스나 외부의 캐시 서버 등에 임시 데이터를 보관할 것을 고려해야 한다.  

### 8. 익셉션 처리
<br/>
스프링은 에러 처리와 관련된 몇 가지 방법을 제공하고 있다.  
+ &#64;ExceptionHandler 애노테이션을 이용한 익셉션 처리
+ &#64;ControllerAdvice 애노테이션을 이용한 공통 익셉션 처리
+ &#64;ResponseStatus 애노테이션을 이용한 익셉션 처리  
#### 8.1. &#64;ExceptionHandler를 이용한 익셉션 처리
<br/>
컨트롤러의 &#64;RequestMapping 메서드를 실행하는 과정에서 익셉션이 발생할 때 직접 그 익셉션 처리를 하고 싶다면, &#64;ExceptionHandler 애노테이션을 사용하면 된다. 다음 코드처럼 익셉션을 처리할 메서드에 &#64;ExceptionHandler 애노테이션을 적용하고, 처리할 익셉션 타입을 &#64;ExceptionHandler 애노테이션의 값으로 지정하면 된다.  
```java
@Controller
public class 클래스명 {

	...

	@ExceptionHandler(Exception클래스명1.class)
	public String 메소드명1() {

		...

		return "에러뷰 경로명";
	}

	...

	@ExceptionHandler(Exception클래스명2.class)
	public String 메소드명2(HttpServletResponse response) {
		response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);

		...

		return "에러뷰 경로명";
	}

	...

	@ExceptionHandler(Exception클래스명3.class)
	public String 메소드명3(Exception클래스명 exception) {

		...// exception을 알맞게 사용

		return "에러뷰 경로명";
	}
}
```
익셉션 타입을 지정하면, 해당 타입을 포함해 하위 타입까지도 처리할 수 있다. 만약 &#64;ExceptionHandler의 값으로 RuntimeException.class를 설정하면, RuntimeException 뿐만 아니라 하위 타입인 ArithmeticException까지 처리하게 된다.  

&#64;ExceptionHandler 메서드를 통해서 익셉션을 처리하면 기본적으로 응답 코드가 정상 처리를 뜻하는 200이 된다. 만약 500과 같은 다른 응답 코드를 전송하고 싶다면, &#64;ExceptionHandler 메서드에서 HttpServletResponse를 파라미터로 추가하고 setStatusCode() 메서드를 이용해서 알맞은 응답 코드를 지정하면 된다.  

&#64;ExceptionHandler 메서드는 HttpServletResponse 뿐만 아니라 HttpServletRequest, HttpSession, Model 등 응답 결과를 생성하는데 필요한 타입을 파라미터로 전달받을 수 있다. 또한, ModelAndView를 리턴하는 것도 가능하다.  

스프링 MVC는 &#64;ExceptionHandler 메서드로 처리한 익셉션 객체를 JSP 뷰 코드에서 사용할 수 있도록 해준다. 따라서,  JSP 코드에서 다음과 같이 exception 기본 객체를 이용해서 익셉션 객체에 접근할 수 있다.  
```jsp
<%@ page contentType="text/html; charset=utf-8" %>
<%@ page isErrorPage="true" %>
<!DOCTYPE html>
<html>
	<head><title>에러 발생</title></head>
	<body>
		작업 처리 도중 문제가 발생했습니다.
		<%= exception %>
	</body>
</html>
```
위 코드는 &#64;ExceptionHandler의 값으로 익셉션 타입을 지정하지 않았는데, 이 경우 파라미터에 있는 익셉션 타입을 이용한다.

스프링 MVC는 컨트롤러에서 익셉션이 발생하면 HandlerExceptionResolver에 처리를 위임한다. HandlerExceptionResolver에는 여러 종류가 존재하는데, MVC 설정(&#60;mvc:annotation-driven&#62; 태그나 &#64;EnableWebMvc 애노테이션)을 사용할 경우 내부적으로 ExceptionHandlerExceptionResolver를 등록한다. 이 클래스는 &#64;ExceptionHanlder 애노테이션이 적용된 메서드를 이용해서 익셉션을 처리하는 기능을 제공하고 있다.  

MVC 설정을 사용할 경우 다음의 순서대로 HandlerExceptionResolver를 사용하게 된다.  

(1) ExceptionHandlerExceptionResolver : 발생한 익셉션과 매칭되는 &#64;ExceptionHandler 메서드를 이용해서 익셉션을 처리한다.  
(2) DefaultHandlerExceptionResolver : 스프링이 발생시키는 익셉션에 대한 처리. 예를 들어, 요청 URL에 매핑되는 컨트롤러가 없을 경우 NoHandlerExceptionResolver는 404 에러 코드를 응답으로 전송한다.  
(3) ResponseStatusExceptionResolver : 익셉션 타입에 &#64;ResponseStatus 애노테이션이 적용되어 있을 경우, &#64;ResponseStatus 애노테이션의 값을 이용해서 응답 코드를 전송한다.  

DispatcherServlet은 익셉션이 발생하면, 가장 먼저 ExceptionHandlerExceptionResolver에 익셉션 처리를 요청한다. 만약 익셉션에 매핑되는 &#64;ExceptionHandler 메서드가 존재하지 않으면, ExceptionHandlerExceptionResolver는 익셉션을 처리하지 않는다. 익셉션이 처리되지 않으면 그 다음 차례인 DefaultHandlerExceptionResolver를 사용하고, 여기서도 익셉션을 처리하지 않으면 마지막으로 ResponseStatusExceptionResolver를 사용한다.  

마지막 차례인 ResponseStatusExceptionResolver에서도 익셉션을 처리하지 않으면, 컨테이너가 익셉션을 처리하도록 만든다.  

#### 8.2. &#64;ControllerAdvice를 이용한 공통 익셉션 처리
<br/>
컨트롤러 클래스에 &#64;ExceptionHandler 애노테이션을 적용하면 해당 컨트롤러에서 발생한 익셉션만을 처리한다. 그런데, 다수의 컨트롤러에서 동일 타입의 익셉션을 발생시킬 수 있고, 이때 익셉션 처리 코드가 동일하다면 어떻게 해야 할까? 각 컨트롤러 클래스마다 익셉션 처리 메서드를 구현하는 것은 불필요한 코드 중복을 발생시킨다.  

이렇게 여러 컨트롤러에서 동일한 익셉션을 발생시킬 경우, &#64;ControllerAdvice 애노테이션을 이용해서 익셉션 처리 메서드 중복을 없앨 수 있다. 다음은 &#64;ControllerAdvice 애노테이션의 사용 예이다.  
```java
@ControllerAdvice("패키지명")
public class 클래스명 {

	...

	@ExceptionHandler(Exception클래스명1.class)
	public String 메소드명() {

		...

		return "에러뷰 경로명";
	}

	...

}
```
&#64;ControllerAdvice 애노테이션이 적용된 클래스는 지정한 벙위의 컨트롤러에서 공통으로 사용될 설정을 지정할 수 있다.  

&#64;ControllerAdvice 적용 클래스가 동작하려면 해당 클래스를 스프링에 빈으로 등록해주어야 한다.  

```xml
<bean class="@ControllerAdvice가적용된완전한클래스명" />
```
&#64;ControllerAdvice 클래스에 있는 &#64;ExceptionHandler 메서드와 컨트롤러 클래스에 있는 &#64;ExceptionHandler 메서드 중 우선순위를 갖는 것은 컨트롤러 클래스에 적용된 &#64;ExceptionHandler 클래스이다. 즉, 컨트롤러의 메서드를 실행하는 과정에서 익셉션이 발생하면 다음의 순서로 익셉션을 처리할 &#64;ExceptionHandler 메서드를 찾는다.  

(1) 같은 컨트롤러에 위치한 &#64;ExceptionHandler 메서드 중 해당 익셉션을 처리할 수 있는 메서드를 검색  
(2) 같은 클래스에 위치한 메서드가 익셉션을 처리할 수 없을 경우, &#64;ControllerAdvice 클래스에 위치한 &#64;ExceptionHandler 메서드를 검색  

&#64;ControllerAdvice 애노테이션은 공통 설정을 적용할 컨트롤러 대상을 지정하기 위해 몇 가지 속성을 제공하는데, 이 속성은 다음과 같다.  
<table>
	<thead>
		<tr>
			<td>속성</td>
			<td>타입</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>value<br/>basePackages</td>
			<td>String[]</td>
			<td>공통 설정을 적용할 컨트롤러들이 속하는 기준 패키지</td>
		</tr>
		<tr>
			<td>annotations</td>
			<td>Class&#60;? extends Annotation&#62;[]</td>
			<td>특정 애노테이션이 적용된 컨트롤러 대상</td>
		</tr>
		<tr>
			<td>assignableTypes</td>
			<td>Class&#60;?&#62;[]</td>
			<td>특정 타입 또는 그 하위 타입인 컨트롤러 대상</td>
		</tr>
	</tbody>
</table>
<br/>

#### 8.3. &#64;ResponseStatus를 이용한 익셉션의 응답 코드 설정
<br/>
&#64;ResponseStatus 애노테이션은 익셉션 자체에 에러 응답 코드를 설정하고 싶을 때 사용한다. 예를 들어, 아래 코드를 보면 fileInfo가 null일 때 NoFileInfoException을 발생시키고 있다.  
```java
@ResponseMapping(value="/files/{fileId:[a-zA-Z]\\d{3}}", method=RequestMethod.GET)
public String fileInfo(@PathVariable STring fileId) throws NoFileInfoException {
	FileInfo fileInfo = getFileInfo(fileId);
	if (fileInfo == null) {
		throws new NoFileInfoException();
	}

	...

}
```
NoFileInfoException의 경우는 FileInfo가 존재하지 않기 때문에 발생한 것이다. 따라서, 에러 코드로 서버 내부에러를 뜻하는 500코드 보다 존재하지 않음을 뜻하는 404 코드를 응답으로 전송하면 더 의미에 맞을 것이다. 이럴 때 &#64;ResponseStatus 애노테이션을 사용하면 익셉션 처리를 따로 하지 않고 응답 코드를 변경할 수 있다. &#64;ResponseStatus 애노테이션은 다음과 같이 익셉션 클래스에 적용한다.  
```java
@ResponsStatus(HttpStatus.NOT_FOUND)
public class 클래스명 extends Exception {
	...
}
```
&#64;ResponseStatus의 값은 HttpStatus를 사용한다. HttpStatus 열거 타입은 여러 응답 코드를 우히나 열거 값을 정의하고 있다. 위 코드의 경우 404에 해당하는 NOT&#95;FOUND를 값으로 주었다.  

org.springframework.http.HttpStatus 열거 타입에서 주로 사용되는 값은 다음과 같다.
+ OK(200, "OK")
+ MOVED&#95;PERMANENTLY(301, "Moved Permanently")
+ NOT&#95;MODIFIED(304, "Not Modified")
+ TEMPORARY&#95;REDIRECT(307, "Temporary Redirect")
+ BAD&#95;REQUEST(400, "Bad Request")
+ UNAUTHORIZED(401, "Unauthorized")
+ PAYMENT&#95;REQUIRED(402, "Payment Required")
+ FORBIDDEN(403, "Forbidden")
+ NOT&#95;FOUND(404, "Not Found")
+ METHOD&#95;NOT&#95;ALLOWED(405, "Method Not Allowed")
+ NOT&#95;ACCEPTABLE(406, "Not Acceptable")
+ UNSUPPORTED&#95;MEDIA&#95;TYPE(415, "Unsupported Media Type")
+ TOO&#95;MANY&#95;REQUEST(429, "Too Many Request")
+ INTERNAL&#95;SERVER&#95;ERROR(500, "Internal Server Error")
+ NOT&#95;IMPLEMENTED(501, "Not Implemented")
+ SERVICE&#95;UNAVAILABLE(503, "Service Unavailable")  

##### 8.4.1. 서비스/도메인/영속성 영역의 익셉션 코드에서는 &#64;ResponseStatus를 사용하지 말 것
<br/>
컨트롤러 영역의 익셉션이 아닌 서비스/도메인/영속성 영역의 익셉션에 &#64;ResponseStatus 애노테이션을 적용하지 않도록 하자. &#64;ResponseStatus 애노테이션은 그 자체가 HTTP 요청/응답 영역인 UI 처리의 의미를 내포하고 있기 때문에, 서비스/도메인/영속성 영역의 코드에 &#64;ResponseStatus 애노테이션을 사용하면 UI 영역에 의존하는 결과를 만든다. 이 경우 UI를 HTTP에서 소켓을 직접 이용하는 방식으로 변경하면 서비스/도메인/영속성 코드도 함께 영향을 받을 가능성이 높아진다. 즉, UI의 변경이 핵심 영역(서비스/도메인 등)의 코드에 영향을 주는 것이다. 이런 상화잉 발생하지 않도록, &#64;ResponseStatus 애노테이션은 컨트롤러 영역에서 발생키는 익셉션 코드에만 사용해야 한다.  

### 9. 컨트롤러 메서드의 파라미터 타입과 리턴 타입
<br/>
&#64;RequestMapping 애노테이션이 적용된 메서드에서 사용할 수 있는 파라미터 타입은 다음과 같다.  
<table>
	<thead>
		<tr>
			<td>파라미터</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>HttpServletRequest, HttpServletResponse</td>
			<td>요청/응답 처리를 위한 서블릿 API</td>
		</tr>
		<tr>
			<td>HttpSession</td>
			<td>Http 세션을 위한 서블릿 API</td>
		</tr>
		<tr>
			<td>org.springframework.ui.Model, org.springframework.ui.ModelMap, java.util.Map</td>
			<td>뷰에 데이터를 전달하기 위한 모델</td>
		</tr>
		<tr>
			<td>&#64;RequestParam</td>
			<td>Http 요청 파라미터 값</td>
		</tr>
		<tr>
			<td>&#64;RequestHeader, &#64;CookieValue</td>
			<td>요청 헤더와 쿠키 값</td>
		</tr>
		<tr>
			<td>&#64;PathVariable</td>
			<td>경로 변수</td>
		</tr>
		<tr>
			<td>커맨드 객체</td>
			<td>요청 데이터를 저장할 객체</td>
		</tr>
		<tr>
			<td>Errors, BindingResult</td>
			<td>검증 결과를 보관할 객체. 커맨드 객체 바로 뒤에 위치해야 함</td>
		</tr>
		<tr>
			<td>&#64;RequestBody (파라미터에 적용)</td>
			<td>요청 몸체를 객체로 변환. 요청 몸체의 JSON이나 XML을 알맞게 객체로 변환</td>
		</tr>
		<tr>
			<td>Writer, OutputStream</td>
			<td>응답 결과를 직접 쓸 때 사용할 출력 스트림</td>
		</tr>
	</tbody>
</table>
<br/>
&#64;RequestMapping 애노테이션이 적용된 메서드에서 사용할 수 있는 주요 리턴 타입은 다음과 같다.
<table>
	<thead>
		<tr>
			<td>리턴 타입</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>String</td>
			<td>뷰 이름</td>
		</tr>
		<tr>
			<td>void</td>
			<td>컨트롤러에서 응답을 직접 생성</td>
		</tr>
		<tr>
			<td>ModelAndView</td>
			<td>모델과 뷰 정보를 함께 리턴</td>
		</tr>
		<tr>
			<td>객체</td>
			<td>메서드에 &#64;ResponseBody가 적용된 경우, 리턴 객체를 JSON이나 XML과 같은 알맞은 응답으로 변환</td>
		</tr>
	</tbody>
</table>
<br/>
리턴 타입이 void인 경우 컨트롤러에서 직접 응답을 생허나는 것을 뜻한다. 예를 들어, 컨틀로러 메서드는 다음과 같이 HttpServletResponse를 이용해서 직접 응답 결과를 생성하는 경우 리턴 타입을 void로 지정해주면 된다.  
```java
@RequestMapping("경로명")
public void 메서드명(HttpServletResponse response) throws IOException {
	response.setContentType("text/plain");
	response.setCharacterEncoding("utf-8");
	PrintWriter writer = response.getWriter();
	writer.write("안녕하세요");
	writer.flush();
}
```

### 10. 스프링 MVC 설정
<br/>
#### 10.1. WebMvcConfigurer를 이용한 커스텀 설정
<br/>
MVC 네임스페이스를 사용할 때와 달리 &#64;EnableWebMvc 애노테이션을 이용하는 경우에는 다음의 인터페이스를 상속받은 &#64;Configuration 클래스를 구현해야 할 때가 있다.  
+ org.springframework.web.servlet.config.annotation.WebMvcConfigurer 인터페이스  
이 인테페이스는 MVC 네임스페이스를 이용한 설정과 동일한 설정을 하는 데 필요한 메서드를 정의하고 있다. 예를 들어, &#60;mvc:view-controller&#62; 태그는 뷰 컨트롤러를 설정할 때 사용되는데 이와 동일한 설정을 위해 WebMvcConfigurer 인터페이스의 addViewControllers() 메서드를 이용한다.  
WebMvcConfigurer 인터페이스는 10개가 넘는 메서드를 정의하고 있는데, 이들 메서드를 모두 구현하는 경우는 드물다. 대신, WebMvcConfigurer 인터페이스를 구현하고 있는 org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter 클래스를 상속받아 필요한 메서드만 구현하는 것이 일반적이다.  

WebMvcConfigurer 인터페이스나 WebMvcConfigurerAdapter 클래스를 상속받아 구현한 클래스는 &#64;Configuration 애노테이션을 적용해서, 설정 정보로 사용해야 한다.  

```java
@Configuration
@EnableWebMvc
public class 클래스명 extends WebMvcConfigurerAdapter {

	...

	@Override
	public void addViewControllers(ViewControllerRegistry registry) {
		registry.addViewController("/index").setViewName("index");
	}

	...

}
```
위 코드처럼 &#64;EnableWebMvc 애노테이션을 적용한 클래스가 WebMvcConfigurerAdapter 클래스를 상속받도록 구현하면 MVC 관련 설정이 한 클래스에 모이므로, 이 방법이 코드 관리 측면에서 좋다고 본다.  

#### 10.2. 뷰 전용 컨트롤러 설정하기
<br/>
```xml
<benas ...>
	<mvc:annotation-driven />
	<mvc:default-servlet-hanlder />
	<mvc:view-controller path="/index" view-name="index" />
	...
```
위 설정을 사용하면 별도의 컨트롤러 코드를 작성하지 않고 "/index" 경로에 대해 index 뷰를 사용하게 된다. 여기서 path 속성의 값은 컨텍스트 경로를 제외한 나머지 경로에 해당된다.  

&#64;EnableWebMvc를 사용한다면, WebMvcConfigurerAdapter를 상속받고 addViewControllers() 메서드에서 경로와 뷰 이름을 매핑해주면 된다. 

#### 10.3. 디폴트 서블릿 설정과 동작 방식
<br/>
web.xml 파일에서 DispatcherServlet에 대한 경로 맹핑을 '/'로 했다고 하자.
```xml
<servlet>
	<servlet-name>dispatcher3</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	...
</servlet>

<servlet-mapping>
	<servlet-name>dispatcher3</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>
```
이 경우, CSS나 JS, HTML, JSP 등에 대한 요청이 DispatcherServlet으로 전달된다. 하지만, 이들 자원 경로에 매핑된 컨트롤러가 존재하지 않는 이상 DispatcherServlet은 404 응답 에러를 발생시킨다. 실제로 css, js, html, jsp 등의 요청은 WAS가 기본으로 제공하는 디폴트 서블릿이 처리하게 되어 있기 때문에, 이들 자원에 대한 요청이 들어오면 디폴트 서블릿이 처리하도록 해야 한다. (디폴트 서블릿이 있는데 JSP나 CSS와 같은 경로를 처리하는 컨트롤러를 따로 개발하는 것은 불필요한 작업이다.)  

MVC 네임스페이스나 &#64;EnableWebMvc 애노테이션을 이용한 쉬운 설정을 사용할 때 DispatcherServlet에 대한 매핑 경로로 "/"를 주는 경우는 매우 흔하지 때문에, 스프링 MVC는 디폴트 서블릿 핸들러라는 특별한 핸들러 구현을 제공하고 있다. 이 디폴트 서블릿 핸들러는 css, js, jsp 등에 대한 요청이 들어오면 그 요청을 디폴트 서블릿에 다시 전달하는 핸들러이다. 따라서, 디폴트 서블릿 핸들러를 사용하면 DispatcherServlet의 매핑 경로로 "/"를 지정하면서 CSS, JS, JSP 등의 처리는 디폴트 서블릿이 처리하도록 할 수 있다.  

디폴트 서블릿 핸들러를 설정하는 방법은 아래와 같다.  
```xml
<beans ...>
	<mvc:annotation-driven />
	<mvc:default-servlet-handler />
</beans>
```
```java
@Configuration
@EnableWebMvc
public class 빈설정클래스명 extends WebMvcConfigurerAdapter {

	...
	
	@Override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable();
	}

	...

}
```
디폴트 서블릿 핸들러를 등록하면, DispatcherServlet은 요청이 들어올 때 다음의 과정을 거쳐서 요청을 처리하게 된다.  
(1) 요청 경로와 일치하는 컨트롤러를 찾는다.  
(2) 컨트롤러가 존재하지 않으면, 디폴트 서블릿 핸들러에 전달한다.  
(3) 디폴트 서블릿 핸드러는 WAS의 디폴트 서블릿에 처리를 위임한다.  
(4) 디폴트 서블릿의 처리 결과를 응답으로 전송한다.  

디폴트 서블릿의 이름은 WAS마다 다른데, 디폴토 서블릿 핸들러 설정에서 디폴트 서블릿의 이름을 저지어하고 싶다면 다음과 같은 코드를 사용하면 된다.  

```xml
<!-- XML -->
<mvc:default-servlet-handler default-servlet-name="default" />
```
```java
// 자바 설정
@Configuration
@EnableWebMvc
public class 빈설정클래스명 extends WebMvcConfigurerAdapter {

	...

	@Override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable("default");
	}

	...

}
```
디폴트 서블릿 이름을 지정하지 않으면, 각 WAS별로 다음과 같은 디폴트 서블릿 이름을 사용한다.
+ 톰갯, Jetty, JBoss : default
+ 웹로직 : FileServlet
+ 구글 앱엔진 : &#95;ah&#95;default
+ 웹스피어 : SimpleFileServlet  

#### 10.4. 정적 자원 설정하기
<br/>
CSS, JS, 이미지 등의 자원은 거의 변화지 않기 때문에, 웹 브라우저에 캐시를 하면 네트워크 사용량, 서버 사용량, 웹 브라우저의 반응 속도 등을 개선할 수 있다. 이런 정적 자원은 보통 별도 웹 서버에서 제공하기 때문에 웹 서버의 캐시 옵션 설정을 통해 웹 브라우저 캐시를 활성화시킬 수  있다. 하지만, 스프링 MVC를 이용하는 웹 어플리케이션에 정적 자원 파일이 함께 포함되어 있다면 웹 서버 설정을 사용하지 않고 &#60;mvc:resources&#62; 설정을 이용해서 웹 브라우저 캐시를 사용하도록 지정할 수 있다. 다음은 &#60;mvc:resources&#62; 태그의 사용 예를 정리한 것이다.  
```xml
<mvc:resources mapping="/images/**" location="/img/,/WEB-INF/resources/" cache-period="60"/>
```
&#60;mvc:resources&#62; 태그의 각 속성은 다음과 같다.
+ mapping : 요청 경로 패턴을 설정한다. 컨텍스트 경로를 제외한 나머지 부분의 경로와 매핑된다.
+ location : 웹 어플리케이션 내에서 요청 경로 패턴에 해당하는 자원의 위치를 지정한다. 위치가 여러 곳일 경우 각 위치를 콤마로 구분한다.
+ cache-period : 웹 브라우저에 캐시 시간 관련 응답 헤더를 전송한다. 초 단위로 캐시 시간을 지정하며, 이 값이 0일 경우 웹 브라우저가 캐시하지 않도록 한다. 이 값을 설정하지 않으면 캐시 관련된 응답 헤더를 전송하지 않는다.  

&#64;EnableWebMvc 설정을 사용한다면, 다음과 같이 WebMvcConfigurerAdapter 클래스를 상속받은 뒤 addResourceHandlers() 메서드를 재정의하면 된다.
```java
@Configuration
@EnableWebMvc
public class 빈설정클래스명 extends WebMvcConfigurerAdapter {

	...

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/images/**").addResourceLocations("/img/", "/WEB-INF/resources/").setCachePeriod(60);
	}

	...

}
```
### 11. HandlerInterceptor를 이용한 인터셉터 구현
<br/>
요청 경로마다 접근 제어를 다르게 해야 한다거나 사용자가 특정 URL을 요청할 때마다 접근 내역을 기록하고 싶다면 어떻게 해야 할까? 이런기능은 특정 컨트롤러에 종속되기보다는 여러 컨트롤러에 공통으로 적용되는 기능들이다. 이런 기능을 각 컨트롤러에서 개별적으로 구현하면 중복 코드가 발생하므로, 코드 중복 없이 컨트롤러에 적용하는 방법이 필요하다.  

스프링은 이미 AOP를 제공하고 있지만, AOP는 너무 법용적인 방법이다. 스프링 MVC는 여러 컨트롤러에 공통으로 적용되는 기능을 구현하는 방법인 HandlerInterceptor를 제공하고 있으며, 이를 사용하면 스프링 MVC에 맞게 공통 기능을 다수의 URL에 적용할 수 있게 된다.  

#### 11.1. HandlerInterceptor 인터페이스 구현하기
<br/>
org.springframework.web.servlet.HandlerInterceptor 인터페이스를 사용하면, 다음의 세 가지 시점에 대해 공통 기능을 넣을 수 있다.  
+ 컨트롤러(핸들러) 실행 전
+ 컨트롤러(핸들러) 실행 후, 아직 뷰를 실행하기 전
+ 뷰를 실행한 이후  

세 시점을 처리하기 위해 HandlerInterceptor 인터페이스는 다음과 같이 세 개의 메서드를 정의하고 있다.
+ boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception
+ void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception
+ void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception  

preHandle() 메서드를 컨트롤러/핸들러 객체를 실행하기 전에 필요한 기능을 구현할 때 사용된다. handler 파라미터는 웹 요청을 처리할 컨트롤러/핸들러 객체이다. 이 메서드를 사용하면 접근 권한이 없는 경우 컨트롤러를 실행하지 않는다거나, 컨트롤러를 실행하기 전에 컨트롤러에서 필요로 하는 정보를 생성하는 등의 작업이 가능하다. preHandle() 메서드의 리턴 타입은 boolean인데, preHandle() 메서드가 false를 리턴하면 컨트롤러(또는 다음 HandlerInterceptor)를 실행하지 않는다.  

postHandle() 메서드는 컨트롤러/핸들러가 정상적으로 실행된 이후에 추가 기능을 구현할 때 사용된다. 만약 컨트롤러가 익셉션을 발생하면 postHandle() 메서드는 실행되지 않는다.  

afterCompletion() 메서드는 클라이언트에 뷰를 전송한 뒤에 실행된다. 만약 컨트롤러를 실행하는 과정에서 익셉션이 발생하면, 이 메서드의 네 번째 파라미터로 전달된다. 익셉션이 발생하지 않았다면 네 번째 파라미터는 null이 된다. 따라서, 컨트롤러 실행 이후에 예기치 않게 발생한 익셉션을 로그로 남긴다거나 실행 시간을 기록하는 등의 후처리를 하기에 적합한 메서드이다.  

org.springframework.web.servlet.handler.HandlerInterceptorAdapter 클래스는 HandlerInterceptor 인터페이스를 구현하고 있는데 각 메서드는 아무 기능도 수행하지 않는다. 따라서, HandlerInterceptor 인터페이스의 메서드를 모두 구현할 필요가 없다면, HandlerInterceptorAdapter 클래스를 상속받은 뒤 필요한 메서드만 재정의하면 된다.  

#### 11.2. HandlerInterceptor 설정하기
<br/>
HandlerInterceptor를 구현했다면, 다음으로 할 작업은 웹 요청을 처리할 때 HandlerInterceptor가 적용되도록 설정에 추가하는 것이다. 추가하는 방법은 다음과 같이 간단하다.  
```xml
<mvc:interceptors>
	<bean id="빈식별자" class="HandlerInterceptor를구현한완전한클래스명" />
</mvc:interceptors>
```
&#60;mvc:interceptors&#62; 태그는 HandlerInterceptor 설정과 경로 설정을 함께 설정할 때 사용된다. 위 설정의 경우 &#60;mvc:interceptors&#62; 태그 내부에 정의한 빈 객체를 핸들러 인터셉터로 사용하고, DispatcherServlet이 처리하는 요청에 대해 핸들러 인터셉터를 적용하게 된다.  

자바 기반의 설정을 사용한다면 다음과 같이 WebMvcConfigurer의 addInterceptors() 메서드를 사용하면 된다.  
```java
@Configuration
@EnableWebMvc
public class 빈설정클래스명 extends WebMvcConfigurerAdapter {

	...

	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(new HandlerInterceptor를구현한클래스명());
	}

	...

}
```
앞서 설정은 DispatcherServlet이 처리하는 모든 요청에 대해 핸들러 인터셉터를 적용하는데, 만약 특정 요청 경로에 대해서만 핸들러 인터셉터를 적용하고 싶다면 다음과 같이 중첩된 &#60;mvc:interceptors&#62; 태그(태그 이름 뒤에 s가 없다)를 사용한다.  

```xml
<mvc:interceptors>
	<mvc:interceptor>
		<mvc:mapping path="/event/**"/>
		<mvc:mapping path="/folders/**"/>
		<bean class="HandlerInterceptor를구현한완전한클래스명" />
	</mvc:interceptor>
</mvc:interceptors>
```
위 설정에서 &#60;mvc:mapping&#62;은 핸들러 인터셉터를 적용할 요청 경로 패턴을 지정한다.(이 경로는 컨텍스트 경로를 제외한 나머지 경로와 매핑된다.) &#60;mvc:mapping&#62; 태그로 지정한 경로 패턴에 적용될 핸들러 인터셉터는 &#60;bean&#62; 태그를 이용해서 지정한다.  

자바 설정을 사용한다면 다음과 같이 addPathPatterns() 메서드를이용해서 경로 패턴 목록을 지정해주면 된다. addPathPatterns() 메서드의 파라미터는 가변 인자이므로 한 개 이상 목록을 지정해주면 된다.  

```java
@Configuration
@EnableWebMvc
public class 빈설정클래스 extends WebMvcConfigurerAdapter {

	...

	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(new HandlerInterceptor를구현한클래스명()).addPathPatterns("/event/**", "/folders/**");
	}

	...

}
```

#### 11.3. HandlerInterceptor의 실행 순서
<br/>
한 요청 경로에 대해 두 개 이상의 핸들러 인터셉터를 적용할 수도 있다.

```xml
<mvc:interceptors>
	<mvc:interceptor>
		<mvc:mapping path="/event/**"/>
		<mvc:mapping path="/folders/**"/>
		<bean class="HandlerInterceptor를구현한완전한클래스명1" />
	</mvc:interceptor>
	<bean class="HandlerInterceptor를구현한완전한클래스명2" />
	<mvc:interceptor>
		<mvc:mapping path="/event/**"/>
		<mvc:mapping path="/folders/**"/>
		<bean class="HandlerInterceptor를구현한완전한클래스명3" />
	</mvc:interceptor>
</mvc:interceptors>
```
preHandle() 메서드는 지정한 순서대로 실행되고, postHandle() 메서드와 afterComplete() 메서드는 지정한 역순으로 실행된다.  

특정 경로 패턴에 대해 핸들러 인터셉터를 적용하고 싶지 않다면 &#60;mvc:exclude-mapping path=""/&#62; 태그 또는 excludePathPatterns() 메서드를 이용해서 제외할 경로 패턴을 지정한다.  

```xml
<mvc:interceptors>
	<mvc:interceptor>
		<mvc:mapping path="/event/**"/>
		<mvc:mapping path="/folders/**"/>
		<mvc:exclude-mapping path="/acl/modify"/>
		<bean class="HandlerInterceptor를구현한완전한클래스명" />
	</mvc:interceptor>
</mvc:interceptors>
```
```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
	registry.addInterceptor(new HandlerInterceptor를구현한클래스명()).addPathPatterns("/acl/**", "/header/**", "/newevent/**").excludePathPatterns("/acl/modify");
}
```
### 12. WebApplicationContext 계층
<br/>
DispatcherServlet은 그 자체가 서블릿이기 대문에 한 개 이상의 DispatcherServlet을 설정하는 것이 가능하다.  

일반적인 웹 어플리케이션에서 컨트롤러는 클라이언트의 요청을 비즈니즈 로직을 구현한 서비스 레이어를 이용하여 처리하는 것이 보통이다. 또한, 서비스 레이어는 영속성 레이어를 사용해서 데이터 접근을 처리하곤 한다.  

서로 다른 DispatcherServlet이 공통 빈을 필요로 하는 경우, ContextLoaderListener를 사용하여 공통으로 사용될 빈을 설정할 수 있게 된다. 다음과 같이 ContextLoaderListener를 ServletListener로 등록하고 contextConfigLocation 컨텍스트 파라미터를 이용하여 공통으로 사용될 빈 정보를 담고 있는 설정 파일 목록을 지정하면 된다.  

```xml
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>/WEB-INF/service.xml, /WEB-INF/persistence.xml</param-value>
</context-param>

<listener>
	<listener-class>
		org.springframework.web.context.ContextLoaderListener
	</listener-class>
</listener>

<servlet>
	<servlet-name>front</servlet-name>
	<servlet-class>
		org.springframework.web.servlet.DispatcherServlet
	</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/front.xml</param-value>
	</init-param>
</servlet>
<servlet>
	<servlet-name>rest</servlet-name>
	<servlet-class>
		org.springframework.web.servlet.DispatcherServlet
	</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/rest.xml</param-value>
	</init-param>
</servlet>
```
실제로 ContextLoaderListener와 DispatcherServlet은 각각 WebApplicationContext 객체를 생성한다.  

ContextLoaderListener가 생성하는 WebApplicationContext는 웹 어플리케이션에서 루트 컨텍스트가 되며, DispatcherServlet이 생성하는 WebApplicationContext는 루트 컨텍스르를 부모로 사용하는 자식 컨텍스트가 된다. 이때 자식은 root가 제공하는 빈을 사용할 수 있기 때문에 각각의 DispatcherServlet이 공통으로 필요로 하는 빈을 ContextLoaderListener를 이용하여 설정하는 것이다. 

ContextLoaderListener는 contextConfigLoaction 컨텍스트 파라미터를 명시하지 않으면 /WEB-INF/applicationContext.xml을 설정 파일로 사용한다. 또한, 클래스패스에 위치한 파일로부터 설정 정보를 읽어 오고 싶다면 &#60;param-value&#62; 태그의 값으로 'classpath:' 접두어를사용하여 설정 파일을 명시하면 된다.  

### 13. DelegatingFilterProxy를 이용한 서블릿 필터 등록
<br/>
스프링 MVC를 이용해서웹 어플리케이션을 개발하다 보면 서블릿 필터에서 스프링 컨테이너에 등록된 빈을 사용해야 하는 경우가 있다. WebApplicationContextUtils 클래스를 사용해도 되지만, 서블릿 필터자체를 스프링 컨테이너에빈으로 등록해서 스프링이 제공하는 DI를 통해 다른 빈을 사용하는 방법을 더 선호한다.  

서블릿 필터를 스프링 빈으로 등록할 때 사용되는 클래스가 바로 DelegatingFilterProxy이다. DelegatingFilterProxy 클래스는 스프링 컨테이너에 빈으로 등록된 서블릿 필터에 필터 처리를 위임한다. DelegatingFilterProxy를 사용하기 위해서는 다음과 같이 web.xml 파일에 DelegatingFilterProxy를 서블릿 필터로 등록해주면 된다.  

```xml
<filter>
	<filter-name>profileFilter</filter-name>
	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	<init-param>
		<param-name>targetBeanName</param-name>
		<param-value>webProfileBean</param-value>
	</init-param>
	<init-param>
		<param-name>contextAttribute</param-name>
		<param-value>
			org.springframework.web.servlet.FrameworkServlet.CONTEXT.dispatcher
		</param-value>
	</init-param>
	<init-param>
		<param-name>targetFilterLifecycle</param-name>
		<param-value>true</param-value>
	</init-param>
</filter>
```
위 코드의 경우 profileFilter는 그 요청을 targetBeanName 초기화 파라미터로 지정한 빈에 위임한다. targetBeanName 초기화 파라미터를 지정하지 않으면, &#60;filter-name&#62;에 지정한 이름을 빈 이름으로 사용한다.  

DelegatingFilterProxy가 사용될 빈 객체는 다음의 두 가지 중 한 군데에 등록될 것이다.  
+ DispatcherServlet이 생성한 WebApplicationContext
+ ContextLoaderListener가 생성한 루트 WebApplicationContext  
이 중, DispatcherServlet이 생성한 스프링 컨테이너에 필터로 사용할 빈 객체가 존재한다면, contextAttribute 초기화 파라미터를 이용해서 DispatcherServlet이 컨테이너를 보관할 때 사용하는 속성 이름을 지정해야 한다. 속성 이름은 "org.springframework.web.servlet.FrameworkServlet.CONTEXT.[서블릿이름]"의 형식을 갖는다.  

contextAttribute 초기화 파라미터를 지정하지 않으면 루트 WebApplicationContext에 등록된 빈을 사용한다.  

DelegatingFilterProxy는 기본적으로 Filter.init() 메서드와 Filter.destroy() 메서드에 대한 호출은 위임하지 않는데, 그 이유는 스프링 컨테이너가 라이프 사이클을 관리하기 때문이다. 만약 Filter의 init() 메서드와 destroy() 메서드에 대해서도 위임을 하고 싶다면 targetFilterLifecycle 초기화 파라미터의 값을 true로 지정해주면 된다.  

### 14. 핸들러, HandlerMapping, HandlerAdapter
<br/>
DispatcherServlet은 웹 요청을 실제로 처리하는 객체의 타입을 &#64;Controller 애노테이션을 구현한 클래스로 제한하지 않는다. 실제로 거의 모든 종류의객체로 웹 요청을 처리할 수 있다. 그래서 웹 요청을 처리하는 객체를 좀 더 범용적인 의미로 '핸들러(Handler)'라고 부른다.  

DispatcherServlet은 핸들러 객체의실제 타입이 무엇인지는 상관하지 않는다. 단지, 웹 요청 처리 결과로 ModelAndView만 리턴하면 DispatcherServlet이 올바르게 동작한다. 그런데, 모든 핸들러 객체가 ModelAndView 객체를 리턴하는 것은 아니다. String을 리턴하는 경우도 있다. 따라서, 누군가는 String 타입을 ModelAndView로 변경해주어야 하는데, 이때 사용하는 것이 바로 HandlerAdapter이다.  

HandlerAdapter는 핸들러의 실행 결과를 DispatcherServlet이 요구하는 ModelAndView로 변환해준다. 따라서, 어떤 종류의 핸들러 객체가 사용되더라도 알맞은 HandlerAdapter만 있으면 스프링MVC 프레임워크에 기반해서 웹 요청을 처리할 수 있게 된다.  

#### 14.1. HandlerMapping의 우선순위
<br/>
MVC 설정을 이용하면 최소 두 개 이상의 HandlerMapping이 등록된다. 각 HandlerMapping은 우선순위를 갖고 있으며, 요청이 들어왔을 때 DispatcherServlet은 우선순위에 따라 HandlerMapping에 요청을 처리할 핸들러 객체를 의뢰한다.  

우선순위가 높은 HandlerMapping이 요청을 처리할 핸들러 객체를 리턴하면, 그 핸들러 객체를 이용한다. 만약 HandlerMapping이 null을 리턴하면, 그 다음 우선순위를 갖는 HandlerMapping을 이용한다. 이렇게 HandlerMapping이 null을 리턴하지 않을 때까지 이 과정을 반복하고 마지막 HandlerMapping까지 null을 리턴하면, DispatcherServlet은 404 에러 코드를 응답한다.  

HandlerMapping이 요청을 처리할 핸들러 객체를 리턴하면, 핸들러 객체를 처리할 HandlerAdapter를 찾은 뒤에 HandlerAdapter에 핸들러 객체 실행을 위임한다.  

#### 14.2. MVC 설정에서의 HandlerMapping과 HandlerAdapter
<br/>
&#60;mvc:annotation-driven&#62; 설정이나 &#64;EnableWebMvc 애노테이션을 사용하면, 다음과 같은 HandlerMapping과 HandlerAdapter를 등록한다.  
<table>
	<thead>
		<tr>
			<td>빈 클래스</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>RequestMappingHandlerMapping</td>
			<td>&#64;Controller 적용 빈 객체를 핸들러로 사용하는 HandlerMapping 구현. 적용 우선순위 높음.</td>
		</tr>
		<tr>
			<td>SimpleUrlHandlerMapping</td>
			<td>&#60;mvc:default-servlet-handler&#62;, &#60;mvc:view-controller&#62; 또는 &#60;mvc:resources&#62; 태그를 사용할 때 등록되는 HandlerMapping 구현. URL과 핸들러 객체를 매핑함. 적용 우선순위 낮음.</td>
		</tr>
		<tr>
			<td>RequestMappingHandlerAdapter</td>
			<td>&#64;Controller 적용 빈 객체에 대한 어댑터.</td>
		</tr>
		<tr>
			<td>HttpRequestHandlerAdapter</td>
			<td>HttpRequestHandler 타입의 객체에 대한 어댑터.</td>
		</tr>
		<tr>
			<td>SimpleControllerHandlerAdapter</td>
			<td>Controller 인터페이스를 구현한 객체에 대한 어댑터.</td>
		</tr>
	</tbody>
</table>
<br/>
&#64;Controller 애노테이션 기반의 컨트롤러는 주로 개발자가 구현할 코드이다. RequestMappingHandlerMapping/Adapter를 이용해서 &#64;Controller 기반 컨트롤러 객체를 핸들러로 사용하게 된다.  

HttpRequestHandler 인터페이스는 주로 스프링이 기본으로 제공하는 핸들러 클래스가 구현하고 있다. 예를 들어, 디폴트 서블릿 핸들러 설정을 위해 다음의 태그를 사용했다.  
```xml
<mvc:default-servlet-handler />
```
위 설정을 하면, 아래 설정과 같이 HttpRequestHandler 인터페이스를 구현한 DefaultServletHttpRequestHandler 클래스와 SimpleUrlHandlerMapping 클래스를 빈으로 등록한다.
```xml
<bean id="defaultServletHttpRequestHandler" class="org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler">
</bean>

<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	<property name="urlMap">
		<map>
			<entry key="/**" value-ref="defaultServletHttpRequestHandler" />
		</map>
	</property>
</bean>
```
&#60;mvc:annotation-driven&#62; 태그가 등록하는 RequestMappingHandlerMapping의 우선순위가 &#60;mvc:default-servlet-handler&#62; 태그가 등록하는 SimpleUrlHandlerMapping의 우선순위보다 높다. 따라서, 특정 요청이 들어올 경우 RequestMappingHandlerMapping을 먼저 확인하고, 그 다음에 SimpleUrlHandlerMapping을 확인한다.  

&#60;mvc:default-servlet-handler&#62; 설정이 등록하는 SimpleUrlHandlerMapping은 우선순위가 낮기 때문에, &#64;Controller 클래스에 매핑되어 있지 않거나 &#60;mvc:view-controller&#62;, &#60;mvc:resources&#62; 등에 메핑되어 있지 않은 요청 경로는 최종적으로 defaultServletHttpRequestHandler가 처리하게 된다. (위 설정에서 "/&#42;&#42;" 요청 경로를 defaultServletHttpRequestHandler에 매핑하고 있다.)
