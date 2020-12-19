---
title: 스프링 MVC : 기본기
categories:
- Spring
feature_text: |
  ## 스프링 MVC : 기본기
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
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

contextClass 초기화 파라미터는 DispatcherServlet이 스프링 컨테이너를 생성할 때 사용할 구현 클래스를 지정한다. 이 값을 지정하지 않으면 XmlWebApplicationContext를 사용하는데, 이 클래스는 XML 설정 파일을 사용한다. 따라서 ,&#64;Configuration 기반의 자바 설정을 이용하는 경우 AnnotationConfigWebApplicationContext 클래스를 사용하도록 contextClass 초기화 파라미터의 값을 지정해주어야 한다.  

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
(1) 요청 URL에 매핑되는 컨트롤러를 검색한다.  
	A. 존재할 경우, 컨트롤러를 이용해서 클라이언트 요청을 처리한다.  
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

```javapublic class MemberRegistRequest {
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
