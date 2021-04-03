---
title: 스프링 테스트 지원
categories:
- Spring
feature_text: |
  ## 스프링 테스트 지원
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

소프트웨어를 개발하는데 있어서 테스트는 필수 요소 중 하나로 자리잡고 있다. 테스트를 작성하는 것은 개발 효율을 높이는 데 도움을 주며, 자동화 된 테스트를 통해 생산성 향상을 꾀할 수 있다. 스프링은 JUnit 4를 위한 지원 클래스를 통해서 스프링 기반의 통합 테스트 코드를 작성할 수 있도록 하고 있으며, 스프링 MVC 테스트를 위한 지원 클래스도 제공하고 있다.  

### 1. 메이븐 의존 설정
<br/>
스프링이 제공하는 테스트 지원 기능을 사용하려면 spring-test 모듈 및 JUnit 관련 모듈을 의존에 추가해주어야 한다.  

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-test</artifactId>
	<version>4.0.4.RELEASE</version>
</dependency>

<dependency>
	<groupId>junit</gruopId>
	<artifactId>junit</artifactId>
	<version>4.11</version>
	<scope>test</scope>
</dependency>

<dependency>
	<groupId>org.hamcrest</groupId<
	<artifactId>hamcrest-library</artifactId>
	<version>1.3</version>
	<scope>test</scope>
</dependency>
```

테스트 관련 모듈 중에서 spring-test와 junit 모듈은 필수로 필요하며, 테스트 코드에서 사용할 다양한 비교 기능을 사용하고 싶다면 hamcrest-library 의존도 추가한다.  

### 2. JUnit4의 스프링 테스트 통합 테스트
<br/>
스프링 컨테이너를 생성하고, 컨테이너가 생성한 빈이 제대로 동작하는지 테스트하고 싶다고 하자. 스프링의 테스트 지원 기능을 사용하지 않는다면 다음과 같은 코드를 사용해야 할 것이다.  

```java
public class CalculatorBeanTest {

	private Calculator calculator;

	@Before
	public void setup() {
		GenericXmlApplicationContext ctx = new GenericXmlApplicationContext("classpath:/springconf.xml");
		calculaotr = ctx.getBean(Calculator.class);
	}

	@Test
	public void sum() {
		assertThat(calculator.sum(1, 2), equalTo(3L));
	}
}
```

위 코드는 GenericXmlApplicationContext 클래스를 이용해서 직접 컨테이너를 생성하고 있다. 컨테이너로부터 특정 빈 객체를 구하고, 이 빈 객체를 이용해서 테스트 메서드를 작성했다.  

스프링의 테스트 지원 기능을 사용하면 위 코드를 다음과 같이 작성할 수 있다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:/springconf.xml")
public class UseXmlConfTest {

	@Autowired
	priavate Calculator calculator;

	@Test
	public void sum() {
		assertThat(calculaotr.sum(1, 2), equalTo(3L));
	}
}
```

앞서 컨테이너를 직접 생성했던 테스트 코드와 비교해보자. 위 코드는 컨테이너 객체를 생성하고 컨테이너로부터 테스트 대상 빈 객체를 구하는 코드가 없다. 대신, &#64;ContextConfiguration 애노테이션을 이용해서 컨테이너를 생성할 때 사용할 스프링 설정 파일을 지정하고, &#64;Autowired 애노테이션을 이용해서 테스트 대상 빈 객체를 지정했다.  

&#64;ContextConfiguration 애노테이션을 보면 이 테스트 클래스가 스프링 컨테이너의 빈을 이용해서 테스트 한다는 것과 컨테이너 생성을 위해 어떤 설정을 사용하는지 알 수 있다. 또한, &#64;Autowired나 &#64;Resource 등 자동 주입을 위한 애노테이션이 적용된 필드를 통해 테스트 대상이 되는 빈 객체가 무엇인지 쉽게 유추할 수 있게 된다.  

#### 2.1. 스프링 테스트 지원 기능 기본 사용
<br/>
스프리의 테스트 지원 기능을 이용해서 컨테이너를 생성하고 빈 객체를 구하려면 다음과 같이 테스트 코드를 작성해야 한다.  

+ SpringJUnit4ClassRunner를 테스트 실행기로 지정
+ &#64;ContextConfiguration 애노테이션으로 설정 정보 지정
+ 자동 주입 애노테이션을 이용해서 테스트에서 사용할 빈 객체 필드로 보관  

사용할 XML 파일이 두 개 이상이면 다음과 같이 배열로 설정 파일 목록을 전달한다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:/spring-datasource.xml", "spring-app"})
public class UserXmlConfTest {
```

location 속성을 이용해서 XML 설정 파일의 경로를 지정할 수도 있다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:/spring-datasource.xml", "spring-app"})
public class UserXmlConfTest {
```

&#64;Configuration 애노테이션을 이용한 자바 코드를 설정으로 사용한다면, classes 속성 값으로 설정 클래스 목록을 전달하면 된다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConf.class)
public class UseJavaConfTest {
```

SpringJUnit4ClassRunner 클래스는 JUnit의 Runner를 구현한 클래스로서, 이 클래스는 &#64; &#64;ContextConfiguration에 지정한 설정 정보를 이용해서 스프링 컨테이너를 생성한다. 그리고, &#64;Autowired 등 자동 주입 기능을 이용해서 스프링 컨테이너를 생성한다. 그리고, &#64;Autowired 등 자동 주입 기능을 이용해서 컨테이너가 생성한 빈 객체를 테스트 클래스의 필드에 할당해준다. 따라서, 테스트 코드에서 사용할 스프링 빈 객체는 자동 주입 애노테이션을 이용해서 필드로 선언하면 된다.  

스프링 MVC를 위한 설정을 이용해서 테스트 코드를 작성하려면, &#64;WebAppConfiguration 애노테이션을 추가로 적용한다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:/springconf.xml", "classpath:/spring-mvc.xml"})
@WebAppConfiguration
public class UseXmlConfTest {
```

&#64;WebAppConfiguration 애노테이션을 사용하면, 웹을 위한 WebApplicationContext 타입의 컨테이너를 생성한다. 이때, WebApplicationContext가 사용하는 웹 어플리케이션의 기준 경로는 "src/main/webapp"로, 웹 어플리케이션 경로를 기준으로 자원을 구하는 코드는 src/main/webapp 디렉토리를 기준으로 자원을 찾는다. 만약 웹 어플리케이션 디렉토리 경로를 바꾸고 싶다면 &#64;WebAppConfiguration 애노테이션 경로 값을 지정하면 된다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:/springconf.xml", "classpath:/spring-mvc.xml"})
@WebAppConfiguration("src/webapp")
public class UseXmlConfTest {
```

&#64;ContextConfiguration 애노테이션에 어떤 값도 지정하지 않으면, 테스트 클래스와 같은 패키지에 위치한 "테스트클래스-context.xml" 파일을 설정 파일로 사용한다.  
&#64;ContextCofiguration 애노테이션에 값을 지정하지 않고 클래스 내부에 중첩 클래스로 &#64;Configuration 적용 클래스가 존재하면, 해당 클래스를 설정정으로 사용한다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
// 중첩 클래스인 Config를 설정 클래스로 사용
@ContextConfiguration
public class UseInnerConfClassTest {
	@Resource(name = "calculaotr3")
	private Calculator calculator;

	@Test
	public void sum() {
		assertThat(calculaotr.sum(1, 2), equalTo(3L));
	}

	@Configuration
	public static class Config {
		@Bean
		public Calculator calculator3() {
			return new Calculator();
		}
	}
}
```

#### 2.2. 설정 재사용
<br/>
스프링 테스트 지원 기능을 이용해서 테스트 코드를 작성하다 보면, 동일한 설정을 사용하는 테스트 클래스가 생기곤 한다.  

동일한 설정 코드가 중복해서 출현하는데, 스프링은 이런 테스트 코드 설정의 중복을 없앨 수 있는 두 가지 방법을 제공하고 있다. 첫 번째 방법은 스프링 4 버전부터 지원하기 시작한 메타 애노테이션을 사용하는 것이다.  

메타 애노테이션은 애노테이션 설정을 묶어 놓은 또 다른 애노테이션이다. 예를 들어, 다음은 앞서 스프링 테스트 설정 애노테이션 설정을 묶어 놓은 메타 애노테이션을 보여주고 있다.  

```java
@ContextConfiguration({"classpath:/springconf.xml", "classpath:/spring-mvc.xml"})
@WebAppConfiguration
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface SpringTestConfig {
}
```

&#64;SpringTestConfig 애노테이션은 앞서 중복해서 존재했던 스프링 테스트 설정을 적용하고 있다. &#64;SpringTestConfig 애노테이션을 테스트 코드에 적용하면, SpringJUnit4ClassRunner는 이 애노테이션에 적용되어 있는 애노테이션들을 스프링 테스트 클래스에 적용한다. 따라서, 앞서 중복이 발생했던 테스트 코드에 스프링 테스트 관련 애노테이션 대신 &#64;SpringTestConfig 애노테이션을 적용함으로써, 테스트 클래스들의 설정 중복을 제거할 수 있게 된다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringTestConfig // 메타 애노테이션 사용
public class UseMetaAnnotationTest {
```

스프링 설정 중복을 제거하는 또 다른 방법은 상속을 사용하는 것이다. 이 방법을 사용하려면 먼저 여러 테스트 클래스에 중복해서 출현하는 설정을 담은 추상 클래스를 만든다. 다음 코드는 작성 예를 보여주고 있다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:/springconf.xml", "classpath:/spring-mvc.xml"})
@WebAppConfiguration
public abstract class AbstractCommonConfTest {
}
```

테스트를 실행할 클래스는 동일한 테스트 설정 코드를 중복해서 작성할 필요 없이, 이 클래스를 상속받아 구현하면 된다.  

```java
public class ReuseParentConfTest extends AbastractCommonConfTest {

	@Autowired
	private ApplicationContext context;

	@Test
	public void beanExists() {
		assertThat(context.containBean("calculator"), equalTo(true));
	}
}
```

만약 상속받은 설정 정보 외에 설정 정보를 추가하고 싶다면, 다음과 같이 &#64;ContextConfiguration 애노테이션에 추가할 정보를 지정하면 된다.  

```java
@ContextConfiguration("classpath:/springconf2.xml")
public class ReuseParentConfTest extends AbstractCommonConfTest {

	@Autowired
	private ApplicationContext context;

	@Resource(name = "systemLogger")
	private SystemLogger logger;

	@Test
	public void logger() throws UnsupportedEncodingException {
		logger.log("test");
		assertThat(out.toString("utf-8").trim(), equalTo("test"));
	}
}
```

AbstractCommonConfTest가 사용하는 설정이 {"classpath:/springconf.xml", "classpath:/spring-mvc.xml"}이었으므로, 위 코드가 스프링 컨테이너를 생성할 때 사용한 설정은 {"classpath:/springconf.xml", "classpath:/spring-mvc.xml", "classpath:/springconf2.xml"}이 된다.  

상위 클래스의 설정 목록(&#64;ContextConfiguration 애노테이션의 value/locations 속성이나 classes 속성)을 사용하지 않으려면 다음과 같이 inhritLocations 속성을 false로 지정한다.  

```java
// 상위 클래스에서 지정한 설정 목록은 사용하지 않음
// 즉, 설정 파일로 "springconf2.xml"만 사용
@ContextConfiguration(value = "classpath:/springconf2.xml", inheritLocations=false)
public class OverrideParentConfTest extends AbstractCommonConfTest {
```

#### 2.3. 프로필 선택
<br/>
테스트에서 사용하는 설정이 여러 프로필 설정을 포함하고 있다면, 다음과 같이 &#64;ActiveProfile 애노테이션을 이용해서 테스트 코드에서 사용할 프로필을 선택할 수 있다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath:embeded-db.xml", "classpath:datasource.xml"})
@ActiveProfiles({"dev", "local"})
public class ChangePasswordTest {
	...
}
```

&#64;ActiveProfiles 애노테이션도 &#64;ContextConfiguration 애노테이션과 마찬가지로 상위 클래스의 설정을 상속받게 되며, 하위 클래스에서 사용할 프로필을 추가할 수 있다.  

상위 클래스에 지정한 &#64;ActiveProfiles의 설정을 사용하고 싶지 않다면 inheritProfiles 속성을 false로 지정한다.  

#### 2.4. 컨텍스트 리로딩 처리
<br/>
JUnit은 각 테스트 메서드를 실행할 때마다 테스트 클래스의 객체를 생성한다. 예를 들어, 다음과 코드를 보자.  

```java
public class SomeTest {
	@Test public void testA() {...}
	@Test public void testB() {...}
	@Test public void testC() {...}
}
```

JUnit은 testA(), testB(), testC() 메서드를 실행할 때마다 매번 SomeTest 클래스의 객체를 생성해서 실행한다. 그렇다면 다음 테스트의 경우 스프링 컨텍스트는 몇 개 생성될까?  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:/springconf.xml")
public class UseXmlConfTest {
	@Autowired
	private Calculator calculator;

	@Test
	public void sum() {
		assertThat(calculator.sum(1, 2), equalTo(3L));
	}

	@Test
	public void sum1() {
		assertThat(calculator.sum(3, 4), equalTo(7L));
	}

	@Test
	public void sum2() {
		assertThat(calculator.sum(5, 6), equalTo(11L));
	}
}
```

sum(), sum1(). sum2() 메서드를 실행할 때마다 UseXmlconfTest 객체를 생성하므로 스프링 컨텍스트도 3개 생성될 거라 생각하기 쉽지만, 실제로 생성되는 스프링 컨텍스트의 개수는 한 개다. 통합 테스트를 진행하다 보면 스프링 컨텍스트를 초기화하는 시간이 길어지기 때문에, 테스트 메서드마다 스프링 컨텍스트를 생성하면 그 만큼 테스트 실행 시간이 증가하게 된다. 이런 이유로 SpringJUnit4ClassRunner는 스프링 컨텍스트를 한 번만 생성하고 각 테스트 메서드마다 스프링 컨텍스트를 재사용한다.  

테스트 메서드 실행 후, 컨텍스트를 다시 로딩하도록 만들려면, &#64;DirtiesContext 애노테이션을 사용한다. 예를 들어, 아래 코드에서 makeContextDirty() 테스트 메서드에 &#64;DirtiesContext 애노테이션을 적용했는데, 이 경우 makeContextDirty() 테스트 메서드 실행 후 다른 테스트 메서들 실행하기 전에 스프링 컨텍스트를 리로딩해서 초기 상태로 되돌린다. 따라서 makeContextDirty() 테스트 메서드 이후에 실행되는 테스트 메서드는 초기화된 컨텍스트를 이용하게 된다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration
public class SomeTest {
	...

	@DirtyContext // 메서드 실행 후, 다음 테스트 실행 전에 컨텍스트 초기화
	@Test
	public void makeContextDirty() {
		...
	}

	...

	@Test // 초기화된 컨텍스트를 이용해서 실행
	public void runWithNonDirtyContext() {
		...
	}
}
```

특정 테스트 메서드가 아닌 각 테스트 메서드를 실행할 때마다 초기화를 하고 싶다면, 다음과 같이 클래스에 &#64;DirtiesContext 애노테이션을 적용하면 된다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:/spring-conf.xml")
@DirtiesContext(classMode = ClassMode.AFTER_EACH_TEST_METHOD)
public class UseXmlConfTest {
	...
}
```

위 코드에서 classMode 속성의 값으로 AFTER&#95;EACH&#95;TEST&#95;METHOD를 지정했는데, 이 경우 각 테스트 메서드 실행 후 다음 테스트 메서드를 실행하기 전에 컨텍스트를 초기화한다.  

&#64;DirtiesContext 애노테이션의 classMode 속성 기본 값은 AFTER&#95;CLASS다. 이는 테스트 클래스의 모든 테스트 메서드 실행을 마친 후에 다음 테스트 클래스를 실행하기 전에 컨텍스트를 리로딩한다는 것을 의미한다. 그럼, 여기서 왜 클래스의 테스트 메서드를 다 실행한 뒤에 컨텍스트를 리로딩하는지 궁금할 것이다. 그 이유는 스프링 컨텍스트를 서로 다른 테스트 클래스를 실행할 때에도 공유하기 때문이다. 예를 들어, 다음 코드를 보자.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConf.class)
public class ConfTest1 {
	...
}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConf.class)
public class ConfTest2 {
	...
}
```

여기서 ConfTest1과 ConfTest2는 같은 설정을 사용하고 있다. 이때 이 두 테스트 클래스를 한 프로세스에서 실행하는데 ConfTest1를 먼저 테스트하고 그 다음 ConfTest2를 테스트한다고 해보자. 이 경우 ConfTest1을 테스트할 때 생성된 스프링 컨텍스트가 ConfTest2를 테스트할 때 그대로 사용된다. 즉, 스프링 컨텍스트가 메모리에 캐시되어 같은 설정을 사용하는 테스트 클래스 간에 공유되는 것이다. 이런 이유로 한 테스트를 완료하고 같은 설정을 사용하는 다른 테스트를 시작하기 전에 컨테이너를 초기화하고 싶다면, 클래스에 &#64;DirtiesContext 애노테이션을 적용해주어야 한다.  

#### 2.5. 테스트 코드의 트랜잭션 처리 과정
<br/>
DB 연동을 포함한 통합 테스트틀 실행하면, 테스트 결과로 DB의 데이터 값이 변경될 수 있는데, 테스트 실행 후 DB 상태가 변경되면, 다음에 다시 테스트를 실행할 때 테스트가 실패할 수 있다. 예를 들어, 다음 코드를 보자. 이 코드는 특정 DAO에 대한 테스트를 실행한다.  

```java
public class MessageDaoIntTest {
	@Autowired
	private MessageDao messageDao;

	@Test
	public void counts() {
		assertThat(messageDao.counts(), equalTo(22));
	}

	@Test
	public void insert() {
		Message message = new Message();
		message,setName("bkchoi");
		message.setMessage("message");
		message.setCreationTime(new Date());
		int newMessageId = messageDao.insert(message);
		assertThat(newMessageId, greaterThan(0));
	}

}
```

먼저 작성한 테스트 메서드는 counts()였고, 그 시점에 행의 개수가 22개였기 때문에, 테스트를 통과시키기 위해 숫자 22를 사용해서 테스트를 검증했다. 그리고, 그 다음에 insert() 테스트 메서드를 작성해서 insert() 테스트 메서드를 작성해서 insert가 제대로 동작하는지 확인했다. 여기서 문제는 insert() 메서드를 실행하고 나면 행의 개수가 바뀌기 때문에 더 이상 counts() 테스트 메서드가 성공하지 않는다는 점이다. 그렇다고 counts() 테스트를 통과시키기 위해 매번 equalTo()의 값을 22, 23, 24 등으로 바꿀 수도 없는 노릇이다.  

이럴 땐 스프링 테스트의 트랜잭션 롤백 기능을 사용하면 도움이 된다. 스프링 테스트 클래스에 &#64;TransactionConfiguration 애노테이션과 &#64;Transactional 애노테이션을 함께 적용하면 테스트 메서드를 트랜잭션 범위에서 실행한 뒤 트랜잭션을 롤백한다. 다음은 이 두 애노테이션을 함께 적용한 코드의 작성 예를 보여주고 있다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:/applicationContext.xml")
@TransactionConfiguration
@Transactional
public class MessageDaoIntTest {

	@Autowired
	private MessageDao messageDao;

	@Test
	public void counts() {
		assertThat(messageDao.counts(), equalTo(22));
	}

	@Test
	public void insert() {
		Message message = new Message();
		message.setName("bkchoi");
		message.setMessage("message");
		message.setCreationTime(new Date());
		int newMessageId = messageDao.insert(message);
		assertThat(newMessageId, greaterThat(0));
	}

}
```

이 테스트 코드를 실행하면 counts() 테스트 메서드와 insert() 테스트 메서드는 각각 트랜잭션 범위 내에서 실행되며, 각 테스트 메서드 실행이 끝나면 스프링 테스트는 트랜잭션을 롤백한다. 즉, insert() 테스트 메서드에서 DB 테이블에 한 개 행을 추가하더라도 롤백되기 때문에 결과적으로 행이 추가되지 않는다. 따라서, 매번 위 테스트를 실행해도 테이블의 행 개수가 변하지 않기 때문에, counts() 테스트 메서드는 실패하지 않는다.  

&#64;TrasactionConfiguration 애노테이션은 PlatformTransactionManager를 이용해서 트랜잭션을 처리한다. 따라서 최소한 한 개의 PlatformTransactionManager가 스프링 설정에 존재해야 한다. 스프링 설정에 두 개 이상의 PlatformTransactionManager가 존재할 경우, 이름이 "transactionManager"인 빈을 트랜잭션 관리자로 사용한다. 만약 트랜잭션 관리자가 두 개 이상인데 이름이 "transactionManager"인 빈이 존재하지 않는다면 다음처럼 transactionManager 속성을 이용해서 트랜잭션 관리자로 사용할 빈 이름을 지정해야 한다.  

```java
RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:/applicationContext.xml")
@TransactionConfiguration(transactionManager="txManager")
@Transactional
public class MessageDaoIntTest {
```

&#64;TransactionConfiguration 애노테이션을 사용하면 기본적으로 테스트 메서드 실행 후 트랜잭션을 롤백한다. 만약 전체 테스트 메서드가 아닌 특정 테스트 메서드에 대해서만 트랜잭션을 롤백하고 싶다면 다음과 같이 defaultRollback 속성을 false로 지정하고, 트랜잭션을 롤백하고 싶은 테스트 메서드에 &#64;Rollback(true)를 적용하면 된다.  

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:/applicationContext.xml")
@TransactionConfiguration(defaultRollback=false)
@Transactional
public class MessageDaoIntTest {
	...
	@Rollback(true)
	@Test
	public void insert() {
		Message message = new Message();
		message.setName("bkchoi");
		message.setMessage("message");
		message.setCreationTime(new Date());
		int newMessageId = messageDao.insert(message);
		assertThat(newMessageId, greaterThat(0));
	}
}
```

&#64;Rollback 애노테이션의 값을 false로 지정하면 트랜잭션을 롤백하지 않고 커밋한다.  

전체 테스트 메서드가 아닌 특정 테스트 메서드만 트랜잭션 범위 내에서 실행하고 싶다면, &#64;Transactional 애노테이션을 해당 메서드에만 적용하면 된다.  

참고로, JUnit의 &#64;Before 및 &#64;After가 적용된 메서드도 한 트랜잭션 범위 내에서 실행된다.  

테스트를 실행할 때 트랜잭션을 롤백하는 방법말고도 DBUnit과 같은 테스트 도구를 이용해서 DB 상태를 동일 상태로 유지하는 방법도 있다. DBUnit과 스프링 통합 테스트를 연동해서 사용할 수 있게 해주는 Spring Test DBUnit을 사용하면 스프링 테스트 호나경에서 좀 더 쉽게 DBUnit을 사용할 수 있다.  

### 3. 스프링 MVC 테스트
<br/>
구현한 컨트롤러가 지정한 경로로 제대로 리다이렉트시키는지, 또는 JSON 응답을 올바르게 응답하는지 확인해야 한다면, 스프링 테스트가 제공하는 MockMvc를 이용해서 보다 빠르게 테스트를 수행할 수 있다. 컨트롤러의 실행 결과를 확인하기 위해 톰캣을 실행하고 웹 브라우저를 이용해서 눈으로 응답 내용을 검증할 수 있지만, MockMvc를 이용해서 테스트 코드를 작성하면 보다 빠르고 효과적으로 응답 결과를 검증할 수 있다.  

#### 3.1. 스프링 테스트의 MockMvc 사용하기
<br/>
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:/spring-mvc.xml")
@WebAppConfiguration
public class HellControllerTest {

	@Autowired
	private WebApplicationContext ctx;

	private MockMvc mockMvc;

	@Before
	public void setUp() {
		mockMvc = MockMvcBuilders.webAppContextSetup(ctx).build();
	}

	@Test void testHello() throws Exception {
		mockMvc.perform(get("/hello").param("name", "bkchoi")).andExpect(status().isOk()).andExpect(model().attributeExists("greeting"));
	}
}
```

perform() 메서드는 파라미터로 전달한 RequestBuilder를 이용해서 HttpServletRequest를 생성하고 DispatcherServlet을 실행한다. 이 과정에서 테스트하려는 컨트롤러나 스프링 MVC 설정을 테스트 할 수 있게 된다. 예를 들어, &#64;RequestMapping 같이 "/hello"인 컨트롤러의 메서드를 실행하게 된다.  

andExpect() 메서드는 컨트롤러가 생성한 응답을 확인한다. 코드를 보면 메서드 이름을 통해 무엇을 검사하는지 쉽게 유추할 수 있다.  

#### 3.2. MockMvc 생성 방법
<br/>
스프링 MVC를 테스트하려면 가장 먼저 해야 할 작업이 org,spirngframework.test.web.servlet.MockMvc 객체를 생성하는 것이다. 이 객체는 다음의 두 가지 방식으로 생성할 수 있다.  

+ WebApplicationContext를 이용해서 생성
+ 테스트 하려는 컨트롤러를 이용해서 생성  

첫 번째 방법은 &#64;WebApplicationContext를 이용해서 생성하는 것이다.  

MockMvcBuilders.webAppContextSetup() 메서드는 전달받은 WebApplicationContext를 이용해서 MockMvc를 생성하는데, 이 경우 완전한 스프링 MVC 환경을 이용해서 테스트를 실행할 수 있다. ViewResolver를 비롯해 스프링 MVC 환경에 필요한 설정은 &#64;ContextConfiguration 애노테이션으로 지정한 설정에 포함된다.  

MockMvc를 생성하는 두 번째 방법은 컨트롤러를 이용해서 MockMvc를 생성하는 것이다.  

```java
public class HellocControllerTest2 {

	private MockMvc mockMvc;

	@Before
	public void setUp() {
		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
		viewResolver.setPrefix("/WEB-INF/view/");
		viewResolver.setSuffix(".jsp");

		mockMvc = MockMvcBuilders.standaloneSetup(new HelloController()).setViewResolvers(viewResolver).build();
	}
}
```

MockMvcBuilders.standaloneSetup() 메서드는 컨트롤러 객체를 전달받아 MockMvc 객체를 생성하며, 테스트 메서드는 전달받은 컨트롤러 객체만을 테스트 대상으로 사용할 수 있다.  

위 코드는 setViewResolvers() 메서드를 이용해서 DispathcerServlet이 사용할 ViewResolver를 직접 설정했는데, 그 이유는 요청 경로인 "/hello"와 HelloController가 리턴하는 뷰 이름인 "hello"가 같기 때문이다. ViewResolver를 지정하지 않을 경우 기본 사용되는 ViewResolver는 컨트롤러가 리턴하는 뷰 이름을 응답 결과를 보여줄 경로로 사용한다. 이 경우 뷰 이름이 "hello"면 요청 경로였던 "/hello"와 같아지기 때문에, MockMvc는 perform()을 수행하는 과정에서 익셉션을 발생한다.  

##### 3.2.1. 필터 설정하기
<br/>
스프링 시큐리티처럼 DispatcherServlet을 실행하기 전에 특정 필터를 적용해야 한다면, MockMvcBuilders의 addFilter() 메서드를 사용하면 된다. 아래 코드는 addFilter() 메서드의 사용 예를 보여주고 있다.  

```java
@Before
public void init() {
	DelegatingFilterProxy securityFilter = new DelegatingFilterProxy();
	securityFilter.setTartgetBeanName("springSecurityFilterChain");
	securityFilter.setServletContext(context.getServletContext());
	mockMvc = MockMvcBuilders.webAppContextSetup(context).addFilter(securityFilter, "/*").build();
}
```

addFilter() 메서드는 첫 번째 파라미터를 적용할 필터 객체를, 두 번째 파라미터는 필터 매핑을 설정한다. 두 번째 파라미터는 가변 아니자로서 한 개 이상의 URL 패턴을 지정할 수 있다.  

#### 3.3. 요청 구성
<br/>
MockMvc 객체를 생성했다면 그 다음은 perform() 메서드를 이용해서 요청을 DispatcherServlet에 전송하는 것이다. perform() 메서드는 RequestBuilder 타입의 인자를 받는데, 이 객체를 직접 생성하기 보다는 스프링 테스트가 제공하는 메서드를 이용해서 RequestBuilder 객체를 생성한다.  

org.springframework.test.web.servlet.request.MockMvcRequestBuilders 클래스는 RequestBuilder를 생성하는데 필요한 다양한 정적 메서드를 제공하고 있다. 이 클래스의 주요 정적 메서드는 다음과 같다. 각 메서드는 GET, POST, PUT, DELETE 요청 방식에 해당한다. (이외에는 동일한 구성의 options(), patch() 메서드를 제공하고 있다.)  

+ MockHttpServletRequestBuilder get(String urlTemplate, Object... urlVariables)
+ MockHttpServletRequestBuilder get(URI uri)
+ MockHttpServletRequestBuilder post(String urlTemplate, Object... urlVariables)
+ MockHttpServletRequestBuilder post(URI uri)
+ MockHttpServletRequestBuilder put(String urlTemplate, Object... urlVariables)
+ MockHttpServletRequestBuilder delete(String urlTemplate, Object... urlVariables)
+ MockHttpServletRequestBuilder delete(URI uri)  

위 정적 메서드를 사용할 때에는 아래 코드처럼 정적 임포트를 코드 가독성을 높인다.  
```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:/spring-mvc.xml")
@WebAppConfiguration
public class HelloControllerTest {
	...// MockMvc 객체 생성

	@Test
	public void testHello() throws Exception {
		mockMvc.perform("/hello").param("name", "bkchoi")).andExpect(status().isOk()).andExpect(model().attributeExists("greeting"));
	}
}
```

get(). post() 등의 메서드는 MockHttpServletRequestBuilder 객체를 리턴하는데, 이 객체는 요청 파라미터 구성, 헤더 설정, 쿠키 설정 등을 할 수 있는 메서드를 제공하고 있다. 예를 들어, 위 코드의 param() 메서드는 이름이 name이고 값이 bkchoi인 요청 파라미터를 설정했다. param() 메서드를 포함해 요청 정보를 생성하기 위해 MockHttpServletRequestBuilder 클래스가 제공하는 메서드 목록은 다음과 같다.  

+ param(String name, String... values)  
파라미터 이름이 name이고 값이 values인 요청 파라미터를 추가한다.
+ cookie(Cookie... cookies)  
요청으로 보낼 쿠키를 지정한다.
+ contentType(MediaType mediaType)
요청 컨텐트 타입을 지정한다.
+ accept(MediaType... mediaTypes)  
Accept 헤더의 값을 지정한다.
+ accept(String... mediaTypes)  
Accept 헤더의 값을 지정한다.  
+ locale(Locale locale)  
요청 로케일을 지정한다.
+ header(String name, Object... values)  
이름이 name이고 값이 values인 요청 헤더를 추가한다.
+ headers(HttpHeaders httpHeaders)  
HttpHeaders를 이용해서 요청 헤더를 추가한다.
+ content(byte[] content)  
몸체 내용을 지정한다.
+ content(String content)  
몸체 내용을 지정한다. UTF-8을 이용해서 byte 배열로 변환한다.
+ contextPath(String contextPath)  
컨텍스트 경로를 지정한다.
+ sessionAttr(String name, Object value)  
세션의 속성과 값을 설정한다.  

위의 모든 메서드는 MockHttpServletRequestBuilder를 다시 리턴한다. 따라서, 다음 코드처럼 메서드 체이닝 방식으로 요청 정보를 구성할 수 있다.  

```java
@Test
public void testHelloJson() throws Exception {
	mockMvc.perform(post("/hello.json").contextPath("/spring-chap18-m").contentType(MediaType.APPLICATION_JSON).content("{\"name\": \"inquotient\"}")).andExpect(status().isOk()).andExpect(jsonPath("$.greeting", equalTo("안녕하세요. 최범균")));
}
```

contentType() 메서드나 accept() 메서드에 파라미터로 사용되는 org.springframework.http.MediaType 클래스는 컨텐트 타입을 위한 몇 가지 상수를 정의하고 있다. 위 코드에서는 JSON을 위한 상수를 사용했는데, 주요 상수 값은 다음과 같다.  

+ APPLICATION&#95;FORM&#95;URLENCODED (application/x-www-form-urlencoded)
+ MULTIPART&#95;FORM&#95;DATA (multipart/form-data)
+ APPLICATION&#95;OCTET&#95;STREAM (application/octet-stream)
+ APPLICATION&#95;JSON (application/json)
+ APPLICATION&#95;XML (application/xml)
+ TEXT&#95;XML (text/xml)
+ TEXT&#95;HTML (text/html)
+ TEXT&#95;PLAIN (text/plain)  

각 상수 이름 뒤에 '&#95;VALUE'가 붙은 상수는 괄호 안에 표기한 String 타입의 컨텐트 타입 값을 갖는다.  

#### 3.4. 응답 검증
<br/>
perform() 메서드를 이용해서 요청을 전송하면, 그 결과 org,springframework.test.web.servlet.ResultAction을 리턴한다. ResultAction은 결과를 검증할 수 있는 andExpect(ResultMatcher matcher) 메서드를 제공하고 있다. andExpect()가 요구하는 ResultMatcher는 MockMvcResultMatchers에 정의된 정적 메서드를 이용해서 생성할 수 있다. 앞서 클래스들과 마찬가지로 코드 가독성을 높이기 위해 정적 임포트를 이용한다.  

status(), jsonPath() 등을 비롯해 MockMvcResultMatcher가 제공하는 다양한 정적 메서드가 존재한다.  

##### 3.4.1. 상태 코드 검증
<br/>
응답 결과를 검증하기 위해 사용되는 정적 메서드는 status()이다. status() 메서드는 StatusResultMatchers 객체를 리턴하며, 이 클래스에 정의된 메서드를 이용해서 응답 상태 코드를 검사할 수 있다.  

응답 상태 코드 확인을 위해 StatusResultMatchers 클래스가 제공하는 메서드는 다음과 같다.  

+ isOk(), isCreated(), isAccepted()  
각각 차례대로 200, 201, 202 응답 상태 코드인지 확인한다.  
+ isMovedPermanently(), isFound(), isNotModified()  
각각 차례대로 301, 302, 304 응답 상태 코드인지 확인한다.
+ isBadRequest(), isUnauthorized(). isForbidden(), isNotFound(), isMethodNotAllowed(), isNotAcceptable(), isUnsupportedMediaType()  
각각 차례대로 400, 401, 403, 404, 405, 406, 415 응답 상태 코드인지 확인한다.  
+ isInternalServerError()  
500 응답 상태 코드인지 확인한다.
+ is2xxSuccessful(), is3xxRedirection(), is4xxClientError(), is5xxServerError()  
각각 응답 코드가 2xx 범위, 3xx 범위, 4xx 범위, 5xx 범위인지 확인한다.  
+ is(int status)  
응답 상태 코드가 status인지 확인한다.  

##### 3.4.2. 뷰/리다이렉트 이름 검증
<br/>
뷰 이름을 검증할 때는 다음과 같이 MockMvcResultMatchers.view() 메서드를 이용한다.  

```java
@Test
public void testHello() throws Exception {
	mockMvc.perform(get("/hello").param("name", "bkchoi")).andExpect(status().isOk()).andExpect(view().name("hello")).andExpect(model().attributeExists("greeting"));
}
```

name() 메서드는 컨트롤러 결과를 보여줄 뷰 이름이 파라미터로 지정한 이름과 같은지 검사한다.  

요청 처리 결과가 리다이렉트 응답이라면 다음과 같이 redirectUrl() 메서드를 이용해서 검사할 수 있다.  

```java
mockMvc.perform(get("/main")).andExpect(status().isMovedTemporalily()).andExpect(redirectUrl("/main/home"));
```

redirectUrlPattern() 메서드를 이용하면 Ant 경로 패런을 이용해서 리다이렉트 경로를 검사할 수 있다.  

##### 3.4.3. 모델 확인
<br/>
컨트롤러에서 생성한 모델을 검사하고 싶다면, MockMvcResultMatchers.model() 메서드를 이용한다.  

model() 메서드는 ModelResultMatchers 객체를 리턴하는데, 이 클래스는 모델 값을 검증하기 위해 다음의 메서드를 제공한다.  

+ attribute(String name, Object value)  
모델 name 속성의 값이 value인지 검사한다.
+ attribute(String name, Matcher&#60;T&#62; matcher)  
모델 name 속성의 값이 Hamcrest의 Matcher에 매칭되는지 검사한다.
+ attributeExists(String... names)  
지정한 이름의 모델 속성이 존재하는지 검사한다.
+ attributeDoseExists(String... names)  
지정한 이름의 모델 속성이 존재하지 않는지 검사한다.
+ attributeErrorCount(String name, int expectedCount)  
지정한 속성에 대해 에러 개수가 지정한 숫자와 같은지 검사한다.
+ attributeHasErros(final String... names)  
지정한 속성이 에러가 없는지 검사한다.
+ attributeHasFieldErrors(String name, String... fieldNames)  
지정한 속성의 특정 필드가 에러를 가졌는지 검사한다.
+ errorCount(int expectedCount)  
전체 에러 개수가 지정한 숫자와 같은지 검사한다.
+ hasErrors()  
에러를 가졌는지 검사한다.
+ hasNotErros()  
에러가 없는지 검사한다.  

여러 메서드를 사용해야 하면 다음과 같이 andExpect() 메서드를 여러 번 호출해야 한다.  

```java
mockMvc.perform(get("/hello").param("name", "bkchoi")).andExpect(model().attributeExists("greeting")).andExpect(model().hasNoErrors());
```

##### 3.4.4. 헤더 검증
<br/>
응답 헤더를 검사하고 싶다면, MockMvcResultMatchers.header() 메서드를 사용한다.  

```java
mockMvc.perform(get("/hello").param("name", "bkchoi")).andExpect(header().doesNotExist("UAC"));
```

header() 메서드는 HeaderResultMatchers 객체를 리턴하며, 이 클래스는 헤더 검사를 위해 다음의 메서드를 제공한다.  

+ doesNotExists(String name)  
지정한 이름을 가진 헤더가 없는지 검사한다.
+ string(String name, String value)  
지정한 이름을 가진 헤더의 값이 value인지 검사한다.
+ string(String name, Matcher&#60;? super String&#62; matcher)  
지정한 이름을 가진 헤더의 값이 Hamcrest Matcher에 매칭되는지 확인한다.
+ longValue(String name, long value)  
지정한 이름을 가진 헤더의 값이 long 타입의 value인지 검사한다.  

##### 3.4.5. 쿠키 검증
<br/>
응답 결과로 생성되는 쿠키를 확인하고 싶다면 MockMvcResultMatchers.cookie() 메서드를 사용한다.  

```java
mockMvc.perform(post("/hello.json").contentType(MediaType,APPLICATION_JSON).content("{\"name\": \"inquotient\"}")).andExpect(status().isOk()).andExpect(cookie().doesNotExists("UAC"));
```

cookie() 메서드는 CookieResultMatchers 객체를 리턴하며, 이 클래스는 쿠키 관련 검사를 위해 다음의 메서드를 제공한다.  

<table>
	<thead>
		<tr>
			<td>메서드</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>doesNotExist(String name)</td>
			<td>해당 이름을 갖는 쿠키가 응답에 포함되어 있지 않은지 검사한다.</td>
		</tr>
		<tr>
			<td>exists(String name)</td>
			<td>해당 이름을 갖는 쿠키가 응답에 포함되어 있는지 검사한다.</td>
		</tr>
		<tr>
			<td>
				value(String name, String expectedValue)<br/>
				value(String name, Matcher&#60;? super String&#62; matcher)
			</td>
			<td>이름이 name인 쿠키의 값이 지정한 값과 일치하는지 검사한다.</td>
		</tr>
		<tr>
			<td>
				maxAge(String name, int maxAge)<br/>
				maxAge(String name, Matcher&#60;? super Integer&#62; matcher)
			</td>
			<td>이름이 name인 쿠키의 유효 시간이 지정한 값과 일치하는지 검사한다.</td>
		</tr>
		<tr>
			<td>
				path(String name, String path)<br/>
				path(String name, Matcher&#60;? super String&#62; matcher)
			</td>
			<td>이름이 name인 쿠키의 경로 값이 지정한 값과 일치하는지 검사한다.</td>
		</tr>
		<tr>
			<td>
				domain(String name, String domain)<br/>
				domain(String name, Matcher&#60;? super String &#62; matcher)
			</td>
			<td>이름이 name인 쿠키의 도메인 값이 지정한 값과 일치하는지 검사한다.</td>
		</tr>
		<tr>
			<td>secure(String name, boolean secure)</td>
			<td>이름이 name인 쿠키가 보안 프로토콜로 전송되는지 여부를 검사한다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 3.4.6. JSON 응답 검증
<br/>
컨트롤러가 JSON 응답을 리턴한다면, MockMvcResultMatchers.jsonPath() 메서드를 사용한다. 이 메서드를 사용하려면 먼저 JSON 응답에서 경로 값을 추출할 때 사용하는 json-path 모듈을 의존에 추가해야 한다.  

```xml
<dependency>
	<groupId>com.jayway.jsonpath</groupId>
	<artifactId>json-path</artifactId>
	<version>0.9.0</version>
	<scope>test</scope>
</dependency>
```

MockMvcResultMatchers.jsonPath() 메서드를 사용해서 JSON 응답을 검사하는 코드에는 다음과 같다.  

```java
mockMvc.perform(get("/books.json")).andExpect(jsonPath("$.books[2].title", equalTo("제목3")) );
```

위 코드는 응답이 다음과 같은 구조일 때 세 번째 title의 값을 검사한다.  

```javascript
{"books": {
	{"title": "제목1", "price": 1000},
	{"title": "제목2", "price": 2000},
	{"title": "제목3", "price": 3000}
}}
```

jsonPath() 메서드의 두 번째 인자는 JSON 경로에 해당하는 값을 비교할 때 사용할 Matcher를 지정한다. org.hamcrest.Matchers.equalTo() 메서드처럼 Hamcrest가 제공하는 Matcher를 사용해서 값을 비교할 수 있다.  

jsonPath()의 JSON 경로를 자바 문자열 포맷을 이용해서 설정할 수도 있다. 예를 들어, 다음 코드는 경로를 자바 문자열 포맷을 이용해서 값을 검증하는 몇 가지 예를 보여주고 있다.  

```java
mockMvc.perform(get("/books.json")).andExpect(jsonPath("$.books").value(hasSize(3))).andExpect(jsonPath("$.books[0].title").value("제목1")).andExpect(jsonPath("$.books[%d].price", 0).value(equalTo(1000))).andExpect(jsonPath("$.books[1]%s", "price").value(2000))
```

위 코드에서 첫 번째와 두 번째 jsonPath() 메서드는 문자열 포맷에서 인자 값이 없는 문자열 포맷을 사용하고 있으며, 세 번째와 네 번째는 각각 %d와 %s를 포함한 문자열 포맷을 사용하고 있다. 문자열 포맷을 사용하는 jsonPath() 메서드는 JsonPathResultMatchers 객체를 리턴한다. 위 코드에서 주로 사용한 value() 메서드를 포함해서 JSON 경로의 값을 검증하기 위해 사용되는 JsonPathResultMatchers 클래스의 메서드는 다음과 같다.  

<table>
	<thead>
		<tr>
			<td>메서드</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				value(final Matcher&#60;T&#62; matcher)<br/>
				value(final Object expectedValue)
			</td>
			<td>JSON 경로가 지정한 값과 일치하는지 검사한다.</td>
		</tr>
		<tr>
			<td>exists()</td>
			<td>JSON 경로가 존재하는지 검사한다.</td>
		</tr>
		<tr>
			<td>doesNotExist()</td>
			<td>JSON 경로가 존재하지 않는지 검사한다.</td>
		</tr>
		<tr>
			<td>doesNotExist()</td>
			<td>JSON 경로가 존재하지 않는지 검사한다.</td>
		</tr>
		<tr>
			<td>isArray()</td>
			<td>JSON 경로가 배열인지 검사한다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>

jsonPath() 메서드가 사용하는 JSON 경로 표현식은 http://goessner,net/articles/JsonPath에 정의되어 있는 표현식을 따른다.  

##### 3.4.7. XML 응답 검증
<br/>
MockMvcResultMatchers.xpath() 메서드를 이용하면 XPath 경로를 이용해서 XML 응답을 검사할 수 있다.  

```java
mockMvc.perform(get("/books.xml")).andExpect(xpath("/book-list/book[3]/title").string("제목3)).andExpect(xpath("/book-list/book[3]/%s", "price").number(3000.0));
```

위 코드의 두 번째 xpath() 메서드처럼 문자열 포맷을 사용할 수 있다.  

xpath() 메서드는 XpathResultMatchers 객체를 리턴하며, 이 클래스는 값 검증을 위해 다음과 같은 메서드를 제공하고 있다.  

<table>
	<thead>
		<tr>
			<td>메서드</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>exists()</td>
			<td>XPath에 해당하는 값이 존재하는지 검사한다,</td>
		</tr>
		<tr>
			<td>doesNotExist()</td>
			<td>XPath에 해당하는 값이 존재하지 않는지 검사한다,</td>
		</tr>
		<tr>
			<td>
				nodeCount(int expectedCount)<br/>
				nodeCount(Matcher&#60;Integer&#62; matcher)
			</td>
			<td>XPath에 해당하는 노드 개수가 일치하는지 검사한다,</td>
		</tr>
		<tr>
			<td>
				string(String expectedValue)<br/>
				string(Mathcer&#60;? super String&#62; matcher)
			</td>
			<td>XPath에 해당하는 값이 지정한 값과 일치하는지 검사한다.</td>
		</tr>
		<tr>
			<td>
				number(Double expectedValue)<br/>
				number(Matcher&#60;? super Double&#62; matcher)
			</td>
			<td>XPath에 해당하는 숫자 값이 지정한 값과 일치하는지 검사한다.</td>
		</tr>
		<tr>
			<td>booleanValue(Boolean value)</td>
			<td>XPath에 해당하는 boolean 값이 지정한 값과 일치하는지 검사한다.</td>
		</tr>
		<tr>
			<td>node(Matcher&#60;? super Node&#62; matcher)</td>
			<td>XPath에 해당하는 노드 값이 지정한 Matcher와 일치하는지 검사한다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 3.4.8. 요청/응답 내용 출력
<br/>
MockMvc를 이용해서 테스트를 진행하다 보면 실제로 생성된 요청과 응답이 어떻게 구성됐는지 궁금할 때가 있다. 이런 경우에는 다음의 코드를 사용해서 MockMvc가 생성하는 요청과 응답 내용을 출력할 수 있다.  

```java
import static org,springframework.test.web.servlet.result.MockMvcResultHandlers.*;

mockMvc.perform(get("/books.xml")).andDo(status().isOk()).andExpect(xpath("/book-list/book[3]/title").string("제목3")).andExpect(xpath("/book-list/book[3]/%s", "price").number(3000.0));
```

perform() 메서드가 리턴하는 ResultActions 클래스는 andDo(ResultHandler handler) 메서드를 제공하고 있는데, MockMvcResultHandlers.print() 메서드는 이 ResultHandler의 구현 클래스인 ConsolePrintingResultHandler 객체를 리턴한다. ConsolePrintingResultHandler 클래스는 요청 내용과 응답 결과 내용을 콘솔에 출력한다.  

따라서, 콘솔에 출력된 내용을 통해 테스트 코드나 컨트롤러 코드가 무엇이 문제인지 파악하는데 도움을 얻을 수 있다.
