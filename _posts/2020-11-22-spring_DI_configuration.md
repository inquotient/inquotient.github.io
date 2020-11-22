---
title: 스프링 DI 설정
categories:
- Spring
feature_text: |
  ## 스프링 DI 설정
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

## 3. 스프링 DI 설정

### 3.1. XML을 이용한 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xmlns:context="http://www.springframework.org/schema/context
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			    http://www.springframework.org/schema/beans/spring-beans.xsd
			    http://www.springframework.org/schema/context
			    http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- import 태그를 통한 설정 파일 조합 -->
	<import resource="classpath:/경로명/파일명" />

	<bean id="빈식별자1" class="생성할 객체의 완전한 클래스 이름">
		<property name="프로퍼티이름1">
			<value>프로퍼티값1</value>
		</property>
		<property name="프로퍼티이름2" value="프로퍼티값2"/>
		<property name="프로퍼티이름3" ref="다른 빈 식별자"/>
		<property name="프로퍼티이름4">
			<list>
				<ref bean="다른 빈 식별자"/>
				<ref bean="다른 빈 식별자"/>
			</list>
		</property>
		<property name="프로퍼티이름5">
			<map>
				<entry>
					<key>
						<value>키값1</value>
					</key>
					<ref-bean="다른 빈 식별자"/>
				</entry>
			</map>
		</property>
		<property name="프로퍼티이름6">
			<map>
				<entry key="키" value="기본데이터 래퍼 타입값"/>
			</map>
		</property>
		<property name="프로퍼티이름7">
			<set>
				<value>값</value>
			</set>
		</property>
		<property name="프로퍼티이름8">
			<props>
				<prop key="키">값</prop>
			</props>
		</property>
		<property name="프로퍼티이름9">
			<value>
				키1 = 값1
				키2 = 값2
			</value>
		</property>
	</bean>
	
	<bean id="빈식별자2" class="생성할 객체의 완전한 클래스 이름">
		<constructor-arg><value>인자값</value></constructor-arg>
		<constructor-arg><ref bean="다른 빈 식별자"/></constructor-arg>
		<constructor-arg value="인자값"/>
	</bean>
	<bean id="빈식별자3" class="생성한 객체의 완전한 클래스 이름" c:파라미터명="값"/>
	<bean id="빈식별자4" class="생성한 객체의 완전한 클래스 이름" c:파라미터명-ref="생성할 객체의 완전한 클래스 이름"/>
	<bean id="빈식별자5" class="생성한 객체의 완전한 클래스 이름" c:_인덱스="값"/>
	<bean id="빈식별자6" class="생성한 객체의 완전한 클래스 이름" c:_인덱스-ref="생성할 객체의 완전한 클래스 이름"/>

	<!-- XML 빈설정과 Java 빈 설정을 같이 사용하는 경우 start -->
	<!-- Java에서 @Configuration 애노테이션이 설정된 클래스를 bean 태그를 이용해서 스프링 빈으로 등록함 -->
	<!-- Javad에서 @Inject, @Autowired, @Resource, @PostConstruct, @Qualifier 등의 DI 애노테이션을 이용할 때도 필요함 -->
	<context:annotation-config/>

	<bean class="등록할 빈설정의 완전한 클래스 이름"/> 
	<!-- XML 빈설정과 Java 빈 설정을 같이 사용하는 경우 end -->

	<bean id="빈식별자6" class="완전한 팩토리 클래스 이름" factory-method="팩토리메서드명"/>
	<bean id="빈식별자7" class="완전한 팩토리 클래스 이름" factory-method="파라미터가 필요한 팩토리메서드명">
		<constructor-arg>
			<!-- property 태그 설정과 동일 -->
		</constructor-arg>
	</bean>
	
	<bean id="빈식별자8" class="완전한 클래스 이름">
		<qualifier value="한정자명"/>
	</bean>

	<!-- 지정한 패키지에 @Component 애노테이션이 적용된 클래스를 검색하여 빈으로 등록함 -->
	<!-- include-filter는 컴포넌트 스캔에 포함시키는 설정 -->
	<!-- exclude-filter는 컴포넌트 스캔에서 제외시키는 설정 -->
	<context:component-scan base-package="패키지명"/>
	<context:component-scan base-package="패키지명">
		<context:include-filter type="annotation" expression="검색할 패키지에서 설정한 완전한 애노테이션 클래스명"/>
		<context:exclude-filter type="annotation" expression="검색할 패키지에서 설정한 완전한 애노테이션 클래스명"/>

		<context:include-filter type="assignable" expression="검색할 패키지에서 완전한 클래스명"/>
		<context:exclude-filter type="assignable" expression="검색할 패키지에서 완전한 클래스명"/>

		<context:include-filter type="regex" expression="정규표현식"/>
		<context:exclude-filter type="regex" expression="정규표현식"/>

		<context:include-filter type="aspectj" expression="AspectJ 표현식"/>
		<context:exclude-filter type="aspectj" expression="AspectJ 표현식"/>
	</context:component-scan>
</beans>
```

```java
public class 팩토리 클래스명 {
	public static 주입받을 클래스명 팩토리메서드명() {
		return 주입받을 객체 생성;
	}
}

```
```java

// 애스터리크(*) 사용 가능
GenericXmlApplicationContext ctx1 = new GenericXmlApplicationContext("classpath:XML설정파일명1");
GenericXmlApplicationContext ctx2 = new GenericXmlApplicationContext("classpath:XML설정파일명2", "classpath:XML설정파일명2");
GenericXmlApplicationContext ctx3 = new GenericXmlApplicationContext("file:XML설정파일명3");
GenericXmlApplicationContext ctx4 = new GenericXmlApplicationContext("file:/상대경로/XML설정파일명4", "file:/상대경로/XML설정파일명5");
클래스 객체명 = ctx1.getBean("빈식별자", 해당하는 빈식별자의 클래스명.class);

```

```java
@Configuration
// @Import(빈설정클래스명1.class) 하나의 설정클래스만 설정하는 경우
@Import({빈설정클래스명1.class, 빈설정클래스명2.class})
// Java 빈설정에서 xml 빈설정을 같이 쓰는 경우
// @ImportResource("classpath:/경로명/파일명.xml") 하나의 설정파일만 설정하는 경우
@ImportResource({"classpath:/경로명/파일명1.xml", "classpath:/경로명/파일명2.xml"})
// @ComponentScan(basePackages = "패키지명") Java 빈설정에서 컴포넌트 스캔을 설정
// FilterType.ANNOTATION : 특정 애노테이션이 적용된 클래스를 필터링 대상으로 하고 value 속성에서 Class 목록을 값으로 갖음
// FilterType.ASSIGNALBLE_TYPE : 지정한 타입에 할당 가능한 클래스를 필터링 대상으로 하고 value 속성에서 Class 목록을 값으로 갖음
// FilterType.REGEX : 이름이 정규표현식에 매칭되는 클래스를 필터링 대상으로 하고 pattern 속성에서 정규표현식 목록을 값으로 갖음
// FilterType.ASPECTJ : 이름이 AspectJ 표현식에 매칭되는 클래스를 대상으로 하고 pattern 속성에서 AspectJ 표현식 목록을 값으로 갖음
@ComponentScan(basePackages = "패키지명", 
		includeFilters = {@Filter(type = FilterType.필드명, value="완전한 클래스명" pattern="완전한 클래스명 또는 표현식"}, 
		excludeFilters = {@Filter(type = FilterType.필드명, value="완전한 클래스명" pattern="완전한 클래스명 또는 표현식"})
public class 빈설정클래스명 {
	@Bean
	public 주입받을 클래스명 빈식별자() {
		
		...

		return 주입받을 객체 생성;
	}
}
```

```java
AnnotationConfigApplicationContext ctx1 = new AnnotationConfigApplicationContext(빈설정클래스명.class);
AnnotationConfigApplicationContext ctx2 = new AnnotationConfigApplicationContext(빈설정클래스명1.class, 빈설정클래스명2.class);
AnnotationConfigApplicationContext ctx3 = new AnnotationConfigApplicationContext(빈설정클래스가 속한 완전한 패키지명);
AnnotationConfigApplicationContext ctx4 = new AnnotationConfigApplicationContext(빈설정클래스가 속한 완전한 패키지명1, 빈설정클래스가 속한 완전한 패키지명2);

클래스명 객체명 = ctx1.getBean("빈식별자", 주입받을 클래스명.class);

```



&#64;Autowired가 설정되어 있으면 객체명을 xml 설정에서 별도 설정이 없어도 스프링이 동일한 타입을 갖는 빈을 프로퍼티의 값으로 사용한다.


```java
public class Autowired가 설정된 프로퍼티를 가진 클래스명 {
	private 주입받을클래스명 객체명1;
	private 주입받을클래스명 객체명2;
	private 주입받을클래스명 객체명3;
	private 주입받을클래스명 객체명4;
	private 주입받을클래스명 객체명5;
	private 주입받을클래스명 객체명6;

	@AutoWired
	public void set객체명1(파라미터) {
		this.객체명1 = 파라미터를 이용하여 객체 할당;
	}
	
	//필수로 받지 않아도 되는 설정
	//@Autowired 설정으로 빈 스캔을 해서 동일한 타입의 빈이 존재하지 않으면 최소 한 개의 주입받을 클래스가 필요하다고 예외를 발생시키는데 
	//required = false를 하면 예외를 출력하지 않고 주입받을 빈이 존재하지 않으면 주입받을 객체의 레퍼런스 null을 가리키고 있음
	//기본값은 true
	@AutoWired(required = false)
	public void set객체명2(파라미터) {
		this.객체명2 = 파라미터를 이용하여 객체 할당;
	}

	//@Autowird 설정은 동일한 클래스가 두 개 이상 빈으로 등록되어 있으면 어떤 빈 객체를 주입해야 하는지 알 수 없다고 예외를 발생시키는데
	//@Qualifier를 사용하면 빈식별자로 매칭하므로 예외를 발생시키지 않음
	//이때 @Qualifier의 빈식별 파라미터를 한정자라고 함
	@Qulifier("한정자명")
	@AutoWired
	public void set객체명2(파라미터) {
		this.객체명2 = 파라미터를 이용하여 객체 할당;
	}
	
	// 빈식별자 기준으로 주입받을 객체 선택
	// 해당하는 빈식별자로 등록된 빈이 존재하지 않으면 익셉션 발생
	@Resource(name = "빈식별자")
	public void set객체명3(파라미터) {
		this.객체명3 = 파라미터를 이용하여 객체 할당;
	}

	//@Resource에 name 프로퍼티 설정이 없으면 주입받는 필드명이나 프로퍼티명의 이름으로 된 빈식별자를 주입하거나 set객체명4()로 주입한다.
	//일치하는 빈이 없거나 두 개 이상 존재하면 익셉션을 발생시킨다.
	@Resource
	public void set객체명4(파라미터) {
		this.객체명4 = 파라미터를 이용하여 객체 할당;
	}
	
	//@Autowired와 동일
	@Inject
	public void set객체명5(파라미터) {
		this.객체명5 = 파라미터를 이용하여 객체 할당;
	}

	@Named("빈식별자")
	public void set객체명6(파라미터) {
		this.객체명6 = 파라미터를 이용하여 객체 할당;
	}
}
```

```xml
<bean id="빈식별자7" class="Autowired가 설정된 프로퍼티를 가진 완전한 클래스명">
	<!-- 객체명1에 대한 설정이 없음 -->
	<!-- 빈식별자8이 객체명1에 대한 주입받을 클래스인 유일한 빈이라면 스프링은 자동으로 이를 주입함 -->
</bean>

<bean id="빈식별자8" class="주입받을 완전한 클래스명" ...>
	...
</bean>
```

컴포넌트 스캔을 이용한 빈 자동 등록

```java

//@Component 클래스명의 첫 글자를 소문자로 한 빈식별자로 빈으로 등록함
//다음과 같이 빈식별자를 설정할 수 있음
@Component("빈식별자")
public class 클래스명 {
	
	...

}
```

&#64;Component 애노테이션은 용도 별로 의미를 부여하는 하위 타입을 가지고 있음.

+ org.springframework.stereotype.Component : 스프링 빈 임을 의미함.
+ org.springframework.stereotype.Service : DDD(도메인 주도 설계)에서의 서비스를 의미함.
+ org.springframework.stereotype.Repository : DDD(도메인 주도 설계)에서의 리파지터리를 의미함.
+ org.springframework.stereotype.Controller : 웹 MVC의 컨트롤러를 의미함.

```xml
<context:component-scan base-package="패키지명">
	<context:include-filter type="type속성
</context:component-scan>
```
#### BeanFactory 인터페이스 메서드 정의
+ &lt;T&gt; T getBean(String name, Class&lt;T&gt; requiredType)
이름이 name이고, 타입이 requiredType인 빈을 구한다. 일치하는 빈이 존재하지 않을 경우 NoSuchBeanDefinitionException이 발생한다.
+ &lt;T&gt; T getBean(Class&lt;T&gt; requiredType)
타입이 requiredType인 빈을 구한다. 일치하는 타입을 가진 빈이 존재하지 않을 겨우 NoSuchBeanDefinitionException이 발생하고, 같은 타입의 빈이 두 개 이상일 겨우 NoUniqueBeanDefinitionException이 발생한다.
+ boolean containsBean(String name)
지정한 이름을 가진 빈이 존재할 경우 true를 리턴한다.
+ boolean isTypeMatch(String name, Class&lt;T&gt; targetType)
지정한 이름을 가진 빈의 타입이 targetType인 경우 true를 리턴한다. 해당 이름을 가진 빈이 존재하지 않을 경우 NoSuchBeanDefinitionException이 발생한다.
+ Class&lt;?&gt; getType(String name)
이름이 name인 빈의 타입을 구한다. 해당 이름을 가진 빈이 존재하지 않을 경우 NoSuchBeanDefinintionException이 발생한다.

#### ListableBeanFactory 인터페이스 메서드 정의
+ int getBeanDefinitionCount()
전체 빈의 개수를 리턴한다.
+ String&#91;&#93; getBeanDefinitionNames()
전체 빈의 이름 목록을 배열로 구한다.
+ String&#91;&#93; getBeanNamesForType(Class&lt;T&gt; type)
지정한 타입을 가진 빈의 이름 목록을 배열로 구한다.
+ &lt;T&gt; Map&lt;String, T&gt; getBeansOfType(Class&lt;T&gt; type)
지정한 타입을 가진 빈 객체를 맵으로 구한다. 맵의 키는 빈의 이름이고, 맵의 값이 빈 객체이다.
+ String&#91;&#93; getBeansNamesForAnnotation(Class&lt;? extends Annotation&gt; annotationType)
클래스가 지정한 애노테이션을 가진 빈의 이름 목록을 배열로 구한다.
+ Map&lt;String, Object&gt; getBeansWithAnnotation(Class&lt;? extends Annotation&gt; annotationType)
지정한 애노테이션을 가진 빈 객체를 맵으로 구한다. 맵의 키는 빈의 이름이고, 맵의 값이 빈 객체이다.

#### 스프링 컨테이너의 생성과 종료
1. 컨테이너 생성
2. 빈 메타 정보(XML이나 자바 기반 설정)를 이용해서 빈 객체 생성
3. 컨테이너 사용
4. 컨테이너 종료 (빈 객체 제거)

+ 다음과 같이 컨테이너를 먼저 생성하고,  그 다음에 메타 정보를 제공할 수 있음
```java
// 1. 컨테이너 생성
GenericXmlApplicationContext ctx = new GenericXmlApplicationContext();
// 2. 메타 정보 제공 및
ctx.load(xml 설정 파일명);
// 2. 빈 객체 생성(읽어온 메타 정보로 빈 객체 재생성)
ctx.refresh();
// 4. 컨테이너 종료
ctx.close();
// java를 실행 중인 콘솔에서 Ctrl+C 키를 눌러서 자바 프로세스를 강제 종료할 수 있는데 이 경우에는 close() 메서드를 호출하는 부분의 코드가 실행되지 않을 수 있으므로 다음 메서드 사용
ctx.registerShutDownHook();
```
