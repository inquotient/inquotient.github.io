---
title: Environment, 프로퍼티, 프로필, 메시지
categories:
- Spring
feature_text: |
  ## Environment, 프로퍼티, 프로필, 메시지
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

### 1. Environment 소개
스프링의 Envirionment는 다음의 두 가지 기능을 제공함.
+ 프로퍼티 통합 관리
+ 프로필을 이용해서 선택적으로 설정 정보를 사용할 수 있는 방법 제공  

Environment는 시스템 환경 변수, JVM 시스템 프로퍼티, 프로퍼티 파일 등의 프로퍼티를 PropertySource라는 것으로 통합 관리한다. 따라서 설정 파일이나 클래스 수정없이 시스템 프로퍼티나 프로퍼티 파일 등을 이용해서 설정 정보의 일부를 변경할 수 있다.


#### 1.1. Environment 구하기
<br/>
```java
ConfigurableApplicationContext context = new AnnotationConfigApplicationContext();
ConfigurableEnvironment environment = context.getEnvironment();
environment.setActiveProfiles("dev");
```

### 2. Environment와 PropertySource
#### Environment의 프로퍼티 읽는 과정
(1) ConfigurableEnvironment에 프로퍼티 값 요청  
(2) MutablePropertySources 프로퍼티 값 요청  
(3) org.springframework.core.env.MutablePropertySources에 두 개 이상의 PropertySource가 등록되어 있을 경우, 프로퍼티 값을 구할 때까지 등록된 순서에 따라 차례 값 요청  

#### 자바의 시스템 프로퍼티 설정방법
(1) java -D 이름=값...  
(2) System.setProperty("이름", "값");

#### 2.1. Environment에서 프로퍼티 읽기

##### 2.1.1. Environment가 제공하는 프로퍼티 관련 주요 메서드
+ boolean containsProperty(String key)  
지정한 key에 해당하는 프로퍼티가 존재하는지 확인함.
+ String getProperty(String key)  
지정한 key에 해당하는 프로퍼티 값을 구하고 존재하지 않으면 null을 리턴함,
+ String getProperty(String key, String defaultValue)  
지정한 key에 해당하는 프로퍼티 값을 구하고 존재하지 않으면 defaultValue를 리턴함.
+ String getRequiredProperty(String key) throws IllegalStateException  
지정한 key에 해당하는 프로퍼티 값을 구하고 존재하지 않으면 익셉션을 발생시킴.
+ &lt;T&gt; T getProperty(String key, Class&lt;T&gt; targetType)  
지정한 key에 해당하는 프로퍼티의 값을 targetType으로 변환해서 구하고 존재하지 않을 경우 null을 리턴함.
+ &lt;T&gt; T getProperty(String key, Class&lt;T&gt; targetType, T defaultValue)  
지정한 key에 해당하는 프로퍼티의 값을 targetType으로 변환해서 구한다. 존재하지 않을 경우 defaultValue를 리턴함.
+ &lt;T&gt; T getRequiredProperty(String key, Class&lt;T&gt; targetType) throws IllegalStateException  
지정한 key에 해당하는 프로퍼티의 값을 targetType으로 변환해서 구하고 존재하지 않을 경우 익셉션을 발생시킴.

#### 2.2. Environment에 새로운 PropertySource 추가하기
<br/>
```java
ConfigurableEnvironment env = context.getEnvironment();
MutablePropertySources propertySources = env.getPropertySources();
propertySources.addLast(new ResourcePropertySource("classpath:/경로명/properties파일"));
System 변수명 = env.getProperty("프로퍼티키");
```

MutablePropertySources는 새로운 PropertySource를 추가해주는 메서드를 제공하고 있는데, 위 코드에서는 addLast() 메서드를 사용했다. addLast() 메서드를 사용하면, 파라미터로 전달한 PropertySource를 마지막 PropertySource로 등록한다. 즉, 프로퍼티 탐색 과정에서 우선순위가 제일 낮다. 반대로 addFirst() 메서드를 사용하면 첫 번째 PropertySource가 되어 우선순위가 제일 높아진다.  

org.springframework.core.io.support.ResourcePropertySource 클래스는 자바 프로퍼티 파일로부터 값을 읽어오는 PropertySource 구현 클래스이며, 이 외에도 스프링은 자바의 Properties 객체로부터 프로퍼티 값을 읽어오는 PropertiesPropertySource, 디렉토리 서버에서 프로퍼티 값을 읽어오는 JndiPropertySource 등을 제공하고 있다.

```java
@Configuration
@PropertySource("classpath:/경로명/프로퍼티파일명1")
// ignoreResourceNotFound 속성이 true로 지정되면 자원이 없는 경우 익셉션을 발생시키지 않고 무시함.
// 자바 8의 @Repeatable을 적용하고 있으므로, 자바 8을 사용한다면 @PropertySource 애노테이션을 여러 번 사용할 수 있음
@PropertySource(value={"classpath:/경로명/프로퍼티파일명1", "classpath:/경로명/프로퍼티파일명2"},
		ignoreResourceNotFound=true
)
@PropertySources({
	@PropertySource("classpath:/경로명/프로퍼티파일명3")
	@PropertySource(value="classpath:/경로명/프로퍼티파일명4", ignoreResourceNotFound=true)
})
public class 빈설정클래스명 {
	...
}
```

### 3. Environment를 스프링 빈에서 사용하기
#### 3.1. org.springframework.context.EnvironmentAware 인터페이스를 구현

```java
public class 클래스명 implements EnvironmentAware {

	...

	@Override
	public void setEnvironment(Environment environment) {
		...
	}

	...

}
```
#### 3.2. &#64;Autowired 애노테이션을 Environment 필드에 적용
```java
public class 클래스명 {

	...

	@Autowired
	private Environment environment;

	...

}
```

### 4. 프로퍼티 파일을 이용한 프로퍼티 설정

```xml
<context:property-placeholder location="classpath:/경로명/프로퍼티파일명1" />
<context:property-placeholder location="classpath:/경로명/*.properties" />
<bean id="빈식별자" class="완전한 클래스명">
	<property name="프로퍼티명1" value="${프로퍼티키1}"/>
</bean>
```

#### 4.1. &lt;context:property-placeholder&gt; 태그가 제공하는 주요 속성  
+ file-encoding  
파일을 읽어올 때 사용할 인코딩을 지정한다. 이 값이 없으면, 자바 프로퍼티 파일 인코딩을 다른다. (JDK에서 제공하는 native2ascii 도구를 이용해서 생성 가능한 인코딩)
+ ignore-resource-not-found  
이 속성의 값이 true면, location 속성에 지정한 자원이 존재하지 않아도 익셉션을 발생시키지 않는다. false면, 자원이 존재하지 않을 때 익셉션을 발생시킨다. 기본값은 false이다.
+ ignore-unresolvable  
이 값이 true면, 플레이스홀더에 일치하는 프로퍼티가 없어도 익셉션을 발생시키지 않는다. false면 플레이스홀더와 일치하는 프로퍼티가 없을 경우 익셉션을 발생시킨다. 기본값은 false이다.

#### 4.1.1. &lt;context:property-placeholder&gt; 태그를 사용할 때 주의할 점    
전체 설정에서 이 태그를 두 번 이상 사용할 경우, 첫 번째로 사용한 태그의 값이 우선순위를 갖는다. 따라서 다음과 같이 별도의 XML 파일에서 &lt;context:property-placeholder&gt; 태그를 설정하고 다른 XML 파일에서 플레이스홀더를 사용하도록 구성하는 것이 좋다.
```xml
<context:property-placeholder location="classpath:/경로명/프로퍼티파일명1", "classpath:/경로명/*.properties"/>
```
#### 4.1.2. PropertySourcePlaceholderConfigurer의 동작 방식  
&lt;context:property-placeholder&gt; 태그는 내부적으로 PropertySourcesPlaceholderConfigurer 객체를 빈으로 등록한다. PropertySourcesPlaceholderConfigurer는 다른 빈 객체를 생성하기 전에 먼저 생성되어, 다른 빈들의 설정 정보에 있는 플레이스홀더의 값을 프로퍼티의 값으로 변경한다. PropertySourcesPlaceholderConfigurer 클래스는 BeanFactoryPostProcessor 인터페이스를 구현하고 있는데, 스프링은 BeanFactoryPostProcessor 인터페이스를 구현한 객체를 특수한 방식으로 사용한다.

스프링은 설정 정보를 읽은 뒤에, BeanFactoryPostProcessor를 구현한 클래스가 있으면, 그 빈 객체를 먼저 생성한다. 그런 뒤에 다른 빈의 메타 정보를 BeanFactoryPostProcessor를 구현한 빈 객체에 전달해서 메타 정보를 변경할 수 있도록 한다. PropertySourcesPlaceholderConfigurer의 경우, 전달하는 메타 정보에 플레이스홀더가 포함되어 있으면, 플레이스홀더를 일치하는 프로퍼티 값으로 치환하는 방식으로 메타 정보를 변경하게 된다.

### 4.2. Configuration 애노테이션을 이용하는 자바 설정에서의 프로퍼티 사용  
```java
@Configuration
public class 빈설정클래스명 {
	@Value("${프로퍼티키1}")
	private String 변수명1;

	...


	@Bean
	public static PropertySourcesPlaceholderConfigurer properties() {
		PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
		configurer.setLocation(new ClassPathResource("프로퍼티키1"));
		return configurer;
	}

	@Bean(initMethod = "init메소드명1")
	public 클래스명 메소드명() {
		...
	}

	...

}
```

```java
@Configuration
@PropertySources(@PropertySource("classpath:/경로명/프로퍼티파일명1"))
public class 빈설정클래스명 {
	@Bean
	public static PropertySourcesPlaceholderConfigurer properties() {
		PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
		return configurer;
	}
	
	@Value("${프로퍼티키1}")
	private String 변수명1;

	...

}
```
#### 4.2.1. PropertySourcesPlaceholderConfigurer 클래스를 직접 사용할 때의 메서드  
+ setLocation(Resource location)  
location을 프로퍼티 파일로 사용한다.
+ setLocation(Resource[] locations)  
locations를 프로퍼티 파일로 사용한다.
+ setFileEncoding(String encoding)  
프로퍼티 파일을 읽어올 때 사용할 인코딩을 지정한다. 지정하지 않을 경우 자바 프로퍼티 파일 인코딩을 따른다.
+ setIgnoreResourceNotFound(boolean b)  
true를 전달하면, 자원을 발견할 수 없어도 익셉션을 발생하지 않고 무시한다.
+ setIgnoreUnresolvablePlaceholders(boolean b)  
true를 전달하면, 플레이스홀더에 해당하는 프로퍼티를 발견할 수 없어도 익셉션을 발생하지 않고 무시한다.

#### 4.2.2. Resource 인터페이스  
+ org.springframework.core.io.ClassPathResource  
클래스패스에 위치한 자원으로부터 데이터를 읽음.
+ org.springframework.core.io.FileSystemResource  
파일 시스템에 위치한 자원으로부터 데이터를 읽음,

&#64;Configuration 애노테이션 기반의 자바 설정에서 PropertySourcesPlaceholderConfigurer 클래스의 setLocation()이나 setLocations() 메서드를 직접 호출할 경우에 이들 Resource 구현 클래스를 주로 사용한다.

#### 4.2.3. &#64;Value 애노테이션  
스프링에서 프로퍼티 값을 설정할 때 사용할 수 있는 애노테이션이다. 그런데, &#64;Configuration 애노테이션을 사용한 설정 클래스 역시 스프링에서는 빈 객체로 생성된다. 따라서, &#64;Configuration 애노테이션 사용 클래스에서 &#64;Value가 붙은 필드는 스프링에서 빈의 프로퍼티로 인식되며, &#64;Value가 플레이스홀더를 가질 경우 PropertySourcesPlaceholderConfigurer의 치환 대상이 된다.

### 5. 프로필을 이용한 설정  
#### 5.1. XML 설정에서 프로필 사용하기  
<br/>
```xml
<!--xml 설정파일1-->
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
				http://www.springframework.org/schema/beans/spring-beans.xsd"
	profile="프로필식별자1">

	...

</beans>
```

```xml
<!--xml 설정파일2-->
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
				http://www.springframework.org/schema/beans/spring-beans.xsd"
	profile="프로필식별자2">

	...

</beans>
```

```xml
<!-- 여러 프로필을 한 설정파일에 작성 -->
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
				http://www.springframework.org/schema/beans/spring-beans.xsd">

	...

	<beans profile="프로필식별자1">
	
		...
	
	</beans>

	...

	<beans profile="프로필식별자2">

		...
	
	</beans>

	...

</beans>
```

```java
GenericXmlApplicationContext context = new GenericXmlApplicationContext();
context.getEnvironment().setActiveProfiles("프로필식별자1");
context.load("classpath:/경로명1/xml 설정파일1", "classpath:/경로명/xml 설정파일2");
context.refresh();
```

```console
java -D spring.profiles.active=프로필식별자1,프로필식별자2 자바클래스명
```

#### 5.2. 자바 &#64;Configuration 설정에서 프로필 사용하기
<br/>
```java
@Configuration
@Profile("프로필식별자1")
public class 빈설정클래스1 {

	@Bean
	public 주입받을클래스명 메소드명() {
	
		...

		return 주입받을객체 생성;
	}
}
```

```java
@Configuration
public class 빈설정클래스2 {
	
	...

	@Configuration
	@Profile("프로필식별자2")
	public static class 빈설정클래스3 {
		
		...
	
		@Bean
		public 주입받을클래스명 메소드명() {
	
			...

			return 주입받을객체 생성;
		}
	}
	
	@Configuration
	@Profile("프로필식별자3")
	public static class 빈설정클래스4 {
		
		...
	
		@Bean
		public 주입받을클래스명 메소드명() {
	
			...

			return 주입받을객체 생성;
		}
	}
	
	...

}
```

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
context.getEnvironment().setActiveProfiles("프로필식별자1");
context.register(빈설정클래스1.class, 빈설정클래스2.class);
context.refresh();
```

#### 5.3. 다수 프로필 설정
<br/>
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd"
	profile="프로필식별자1, 프로필식별자2">
```
```java
@Configuration
@Profile("프로필식별자1, 프로필식별자2")
public class 빈설정클래스 {

	...

}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd"
	profile="!프로필식별자1">

<!-- 프로빌식별자1 프로필이 활성화되지 않을 겨우 사용된다는 의미 -->
```

### 6. MessageSource를 이용한 메시지 국제화 처리  
```xml
<bean id="basenames" class="org.springframework.context.support.ResourceBundleMessageSource">
	<property name="프로퍼티키1">
		<value>프로퍼티값1</value>
	</property>
</bean>
```

#### 6.1. 프로퍼티 파일과 MessageSource  
+ String getMessage(String code, Object[] args, String defaultMessage, Locale locale)
+ String getMessage(String code, Object[] args, Locale locale)

#### 6.2. ResourceBundleMessageSource를 이용한 설정  
ResourceBundleMessageSource 클래스는 MessageSource 인터페이스의 구현 클래스로서 java.util.ResourceBundle을 이용해서 메시지를 읽어온다. basename 프로퍼티의 값은 메시지를 로딩할 때 사용할 ResourceBundle의 베이스 이름을 의미한다. 베이스 이름은 패키지를 포함한 완전한 이름이어야 한다. 따라서 프로퍼티키의 포로퍼티의 값이 패키지명.파일명 일 경우, 패키지에 위치한 프로퍼티 파일(정확하게는 파일명.properties나 파일명&#95;언어.properties)로부터 메시지를 가져온다. 

한 개 이상의 프로퍼티 파일로부터 메시지를 로딩하고 싶다면, 프로퍼티에 &lt;list&gt; 태그를 이용하여 베이스 이름 목록을 전달하면 된다.

ResourceBundle은 프로퍼티 파일의 이름을 이용하여 언어 및 지역에 따른 메시지를 로딩한다.
+ 파일명.properties : 기본 메시지. 시스템의 언어 및 지역에 맞는 프로퍼티 파일이 존재하지 않을 경우에 사용된다.
+ 파일명&#95;en.properties : 영어 메시지
+ 파일명&#95;ko.properties : 한글 메시지
+ 파일명&#95;en&#95;UK.properties : 영국을 위한 영어 메시지

```xml
<!-- 자바 5 버전 이하의 한글 프로퍼티파일 -->
hello = \uc548\ub155\ud558\uc138\uc694!
<!-- 자바 6 버전 이상의 한글 프로퍼티파일 -->
hello = 안녕하세요!
```

자바 5 버전까지 ResourceBundle이 사용하는 파일은 위와 같이 유니코드를 이용해서 입력해주어야 했다.

하지만, 자바 6 버전부터는 캐릭터 인코딩을 지정할 수 있는 방법이 추가되었다. 메시지 파일이 UTF-8로 작성되었을 경우 다음과 같이 defaultEncoding 프로퍼티의 값을 "UTF-8"로 지정해주면 UTF-8로 작성된 메시지 파일을 올바르게 읽어온다.

```xml
<bean id="basenames" class="org.springframework.context.support.ResourceBundleMessageSource">
	<property name="프로퍼티키">
		...
	</property>
	<property name="defaultEncoding" value="UTF-8"/>
</bean>
```

#### 6.3. ReloadableResourceBundleMessageSource를 이용한 설정  
ResourceBundleMessageSource가 자바의 리소스 번들을 이용해서 메시지를 읽어오는데, 이 때 메시지 파일을 클래스패스 외에 다른 곳에는 위치시킬 수 없는 불편함이 있다. 또한, 한 번 메시지 파일을 읽어오면 메시지 파일이 변경되더라도 변경된 내용이 반영되지 않는다.

스프링은 이런 두 가지 단점을 보완하기 위해 ReloadableResourceBundleMessageSource를 제공하고 있다. ReloadableResourceBundleMessageSource는 다음과 같은 특징이 있다.

+ 클래스패스뿐만 아니라 특정 디렉토리에 위치한 메시지 파일을 사용할 수 있다.
+ 클래스패스를 사용하지 않을 경우, 파일 내용이 변경되었을 때, 변경된 내용이 반영된다.

```xml
<bean id="빈식별자" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
	<property name="프로퍼티명">
		<list>
			<value>file:/.../경로명/</value>
			<value>classpath:.../경로명</value>
		</list>
		<property name="defaultEncoding" value="UTF-8"/>
		<property name="cacheSeconds" value="10"/>
	</property>
</bean>
```

cacheSeconds 프로퍼티는 메시지 파일 정보를 캐싱할 시간을 초 단위로 기록한다. 이 값이 0인 경우에는 메시지를 요청할 때마다 매번 파일 변경 여부를 확인하기 때문에 성능이 심각하게 나빠질 수 있으므로 실제 제품 환경에서는 cacheSeconds의 값으로 0을 사용해선 안 된다.

basenames 프로퍼티에서 값을 지정할 때 주의할 점은 클래스패스 자원의 경우는 리로딩 기능이 적용되지 않는다는 점이다. 따라서, 리로드 대상 메시지는 클래스패스가 아닌 파일 자원을 이용해야 한다.

#### 6.4. 빈 객체에서 메시지 이용하기  
빈 객체에서 스프링이 제공하는 MessageSource를 사용하려면 다음의 두 가지 방법 중 한 가지를 사용하면 된다.
+ ApplicationContextAware 인터페이스를 구현한 뒤, setApplicationContext() 메서드를 통해 전달받은 ApplicationContext의 getMessage() 메서드를 이용하여 메시지 사용
+ MessageSourceAware 인터페이스를 구현한 뒤, setMessageSource() 메서드를 통해 전달받은 MessageSource의 getMessage() 메서드를 이용하여 메시지 사용

```java
public class 클래스명 implements MessageSourceAware {
	private MessageSource messageSource;
	public void setMessageSource(MessageSource messageSource) {
		this.messageSource = messageSource;
	}

	...

}
```
