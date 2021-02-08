---
title: 스프링 MVC 기타 설정
categories:
- Spring
feature_text: |
  ## 스프링 MVC : 기타 설정
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	td { border: 1px solid #444444; }
</style>

### 1. 서블릿 3 기반 설정
<br/>
서블릿 3 버전부터 web.xml 파일을 사용하는 대신 자바 코드를 이용해서 서블릿/필터를 등록할 수 있게 되었다. 스프링도 이에 따라 web.xml이 아닌 자바 코드를 이용해서 DispatcherServlet을 설정할 수 있는 방법을 제공하고 있다. 자바 코드로 DispatcherServlet을 설정하는 방법은 간단한다. 스프링이 제공하는 org.springframework.web.WebApplicationInitializer 인터페이스는 다음과 같이 한 개의 메서드만을 정의하고 있다.  

```java
public interface WebApplicationInitializer {
	void onStartUp(ServletContext servletContext) throws ServletException;
}
```

WebApplicationInitializer 인터페이스를 상속받은 클래스는 onStartup() 메서드에서 DispatcherServlet을 직접 생성해서 ServletContext에 등록해주면 된다.  

```java
public class 서블릿설정클래스명 implements WebApplicationInitializer {

	@Override
	public void onStartup(ServletContext servletContext) throws ServletException {
		XmlWebApplicationContext servletAppContext = new XmlWebApplicationContext();
		servletAppContext.setConfigLocation("/WEB-INF/dispatcher.xml");

		DispatcherServlet dispatcherServlet = new DispatcherServlet(servletAppContext);
		ServletRegistration.Dynamic registration = servletContext.addServlet("dispatcher", dispatcherServlet);
		registartion.setLoadOnStartup(1);
		registration.addMapping("*.do");
	}
}
```

필터와 &#64;Configuration을 이용한 설정, 루트 컨텍스트 설정 등을 모두 할 수 있긴 하지만, 그러기엔 코딩해주어야 할 내용이 많다. 그래서 스프링은 WebApplicationInitializer 인터페이스를 상속받아 필요한 기능을 미리 구현한 추상 클래스를 제공하고 있다.  

+ WebApplicationInitializer ← AbstractContextLoaderInitializer ← AbstractDispatcherServletInitializer ← AbstractAnnotationConfigDispatcherServletInitializer  

스프링 설정으로 XML을 사용한다면 AbstractDispatcherServletInitializer 클래스를 상속받아 필요한 메서드만 제정의하면 된다.  

```java
public class 서블릿설정클래스명 extends AbstractDispatcherServletInitializer {
	// abstract 메서드 재정의
	@Override
	protected WebApplicationContext createServletApplicationContext() {
		XmlWebApplicationContext servletAppContext = new XmlWebApplicationContext();
		servletAppContext.setConfigLocation("/WEB-INF/dispatcher.xml");
		return servletAppContext;
	}

	// 상위 클래스는 "dispatcher" 리턴
	@Override
	protected String getServeltName() {
		return "dispatcher2"; // 기본 값은 "dispatcher"
	}

	// abstract 메서드 재정의
	@Override
	protected String getServletMapping() {
		return new String[] { "*.do" };
	}

	// 상위 클래스는 true 리턴
	@Override
	protected boolean isAsyncSupported() {
		return super.isAsyncSupported();
	}

	// abstract 메서드 재정의
	@Override
	protected WebApplicationContext createRootApplicationContext() {
		XmlWebApplicationContext rootAppContext = new XmlWebApplicationContext();
		rootAppContext.setConfigLocation("/WEB-INF/root.xml");
		return rootAppContext;
	}

	// 상위 클래스는 기본으로 null 리턴
	@Override
	protected Filter[] getServletFilters() {
		CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
		encodingFilter.setEncoding("utf-8");
		Filter[] filters = new Filter[] { encodingFilter };
		return filters;
	}

}
```

+ createServletApplicationContext()  
DispatcherServlet이 사용할 WebApplicationContext 객체를 생성한다.
+ getServletName()
DispatcherServlet의 서블릿 이름을 리턴한다. 이 메서드를 재정의하지 않을 경우 "dispatcher"를 이름으로 사용한다.
+ getServletMappings()  
생성할 DispatcherServlet이 매핑될 경로를 리턴한다.
+ isAsyncSupported()  
DispatcherServlet이 비동기를 지원하는지 여부를 리턴한다. 재정의하지 않을 경우 기본 값은 true이다.  

앞서 WebApplicationInitializer 인터페이스를 상속받은 경우와 달리 DispatcherServelt 객체를 직접 생성할 필요는 없다. DispatcherSevlet 객체는 상윜 클래스인 AbstractDispatcherServletInitailizer에서 생성하며, 생성 과정에서 위에서 언급한 네 가지의 메서드를 이용해 DispatcherServlet을 설정한다.  

나머지 두 가지 메서드는 다음과 같다.

+ createRootApplicationContext() : 루트 컨텍스트를 생성한다. 만약 루트 컨텍스트가 필요 없다면 단순히 null을 리턴하면 된다.
+ getServletFilters() : DipatcherServlet에 적용할 서블릿 필터 객체를 리턴한다.  

XML 설정이 아니라 &#64;Configuration 기반 자바 설정을 사용하고 싶다면, AbstractAnnotationConfigDispatcherServletInitializer 클래스를 상속받아 좀 더 간결하게 설정할 수 있다.  


```java
public class 서블릿설정클래스명 extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class<?>[] { RootConfig.class };
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class<?>[] { WebConfig.class };
	}

	// 상위 클래스는 "dispatcher" 리턴
	@Override
	protected String getServletName() {
		return "dispatcher3"; // 기본 값은 "dispatcher"
	}

	// abstract 메서드 재정의
	@Override
	protected String[] getServletMapping() {
		return new String[] {"*.do" };
	}

	// 상위 클래스는 true 리턴
	@Override
	protected boolean isAsyncSupported() {
		return super.isAsyncSupported();
	}

	// 상위 클래스는 기본으로 null 리턴
	@Override
	protected Filter[] getServletFilters() {
		CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
		encodingFilter.setEncoding("utf-8");
		Filter[] filters = new Filter[] { encodingFilter };
		return filters;
	}

}
```

앞서 XmlWebApplicationContext 객체를 직접 생성했던 것과 달리 getRootConfigClasses() 메서드와 getServletConfigClasses() 메서드에서 설정으로 사용할 클래스 목록을 리턴하는 것으로 끝난다. 이 두 메서드는 각각 루트 컨텍스트와 DispatcherServlet의 컨텍스트를 생성할 때 사용할 스프링 설정 클래스의 목록을 리턴한다. 실제 AnnotationConfigWebApplicationContext 객체를 생성하는 것은 상위 클래스인 AbstractAnnotationConfigDispatcherServletInitializer에서 이루어진다.  

web.xml을 사용하는 것과 자바 코드 기반의 서블릿 설정을 사용하는 것 중 무엇이 더 좋은지는 말하기 어려운 주제다. 자바 코드 기반의 서블릿 설정을 사용하면 IDE의 코드 자동 완성 기능의 도움을 얻을 수 있어 좋다. 하지만, 급하게 운영중인 서버의 설정을 변경해서 재시작해야 하는 경우에는 web.xml 파일을 사용할 때 더 민첩하게 반응할 수 있다. 두 가지 방식 중에서 무엇을 사용할지 여부는 명확한 근거보다는 선호에 의해 결정되는 경우가 더 많다. 그러니 두 방식 중 무엇이 좋을지 고민하지 말고 익숙한 방식이나 원하는 방식을 사용하면 된다.
