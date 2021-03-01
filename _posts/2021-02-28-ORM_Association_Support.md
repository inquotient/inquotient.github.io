---
title: ORM 연동 지원
categories:
- Spring
feature_text: |
  ## ORM 연동 지원
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

DB 연동을 할 때 JDBC API를 직접 사용하는 경우는 드물다. 보통은 MyBatis와 같은 SQL 매퍼나 하이버네이트, JPA와 같은 ORM 프레임워크를 사용한다. 이들 프레임워크는 각자 자신만의 트랜잭션 처리 방법과 DB 연결 설정 방법을 정의하고 있다. 스프링 기반 어플리케이션에서 이들 프레임워크를 이용하면서 스프링이 제공하는 트랜잭션 처리나 DataSource 연동을 사용할 수 없다면 스프링의 장점(선언적 트랜잭션 지원 등)이 반감될 것이다. 다행히 스프링은 이들 DB 연동 기술을 사용하면서 동시에 스프링이 제공하는 DB 지원 기능(트랜잭션, DataSource 설정)을 적용할 수 있도록 하고 있다.  

### 1. &#64;Repository 애노테이션을 이용한 익셉션 변환 처리
<br/>
JdbcTemplate 클래스는 내부적으로 JDBC API를 이용해서 데이터베이스 연동을 처리하는데, 이 과정에서 SQLException이 발생하면 SQLException을 스프링의 DataAccessException으로 변환해서 발생시킨다.  

이와 비슷하게 org.springframework.stereotype.Repository 애노테이션을 이용하면 하이버네이트와 JPA API가 발생시키는 익셉션을 스프링의 DataAccessException으로 변환할 수 있다. 예를 들어, 하이버네이트 API를 사용하는 과정에서 하이버네이트의 ObjectNotFoundException이 발생하면 동일한 의미를 갖는 스프링의 ObjectRetrievalFailureException으로 변환해서 발생시킨다.  

&#64;Repository 애노테이션이 적용된 클래스의 메서드가 발생시킨 익셉션을 스프링의 DataAccessException으로 변환하려면, PersistenceExceptionTranslatePostProcessor를 빈 객체로 등록해주어야 한다.  

```xml
<bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor">
	<property name="proxyTargetClass" value="true" />
</bean>

<bean id="itemRepository" class="net.madvirus.spring4.chap13.store.persistence.HibernateItemRepository">
	<property name="sessionFactory" ref="sessionFactory" />
</bean>
```

PersistenceExceptionTranslationPostProcessor 클래스는 &#64;Repository 애노테이션이 적용된 빈 객체의 프록시 객체를 생성한다. PersistenceExceptionTrnalationPostProcessor는 기본적으로 인터페이스를 이용해서 프록시 객체를 생성하므로, 클래스 기반의 프록시를 생성하길 원한다면 다음과 같이 proxyTargetClass 속성의 값을 ture로 설정해주어야 한다.  

#### 1.1. PersistenceExceptionTranslationPostProcessor의 동작 방식
<br/>
PersistenceExceptionTranslationPostProcessor는 &#64;Repository가 적용된 빈 객체의 프록시를 생성한다고 했다. 예를 들어, 다음과 같은 설정이 있고, HibernateItemRepository 클래스에 &#64;Repository 애노테이션이 적용되어 있다고 해보자.

이 경우, PersistenceExceptionTranslationPostProcessor에 의해 HibernateItemRepository에 대한 프록시 객체가 생성된다. 이때, 생성된 프록시 객체는 원본 대상 객체에서 익셉션이 발생하면 직접 익셉션 변환 처리를 하지 않고, PersistenceExceptionTranslator 구현 객체에 변환을 위임한다.  

그러면, PersistenceExceptionTranslator는 어떻게 설정할까? 스프링은 등록된 빈 객체 중에서 PersistenceExceptionTranslator 타입을 검색해서 사용한다. 이는, PersistenceExceptionTranslator 타입의 빈을 추가로 등록해주어야 한다는 것을 뜻한다.  

하지만 실제로 PersistenceExceptionTranslator 타입의 빈을 추가로 설정할 필요는 없다. 스프링이 제공하는 연동 모듈은 자체적으로 PersistenceExceptionTranslator을 구현하고 있다. 예를 들어, 하이버네이트 연동에 사용되는 LocalSessionFactoryBean는 이미 PersistenceExceptionTranslator 인터페이스를 구현하고 있기 때문에, 다음과 같이 스프링 연동 설정을 하는 과정에서 하이버네이트 용 PersistenceExceptionTranslator 구현이 등록된다.  

JPA와 MyBatis 연동에 사용되는 클래스들 역시 PersistenceExceptionTranslator를 구현하고 있으므로, 추가로 PersistenceExceptionTranslator 타입의 빈을 등록할 필요가 없다.  

### 2. 하이버네이트 연동 지원
<br/>
스프링 4 버전은 하이버네이트 3.6 또는 그 이후 버전을 지원한다.  

#### 2,1, 하이버네이트 4 버전 연동 설정
<br/>
하이버네이트 4 버전을 스프링에서 사용하려면 다음과 같은 설정을 해주면 된다.  

+ 의존에 하이버네이트 4 모듈 추가
+ 스프링 설정  
+ LocalSessionFactoryBean으로 SessionFactory 설정
+ HibernateTransactionManager로 트랜잭션 관리자 설정
+ DAO에서 SessionFactory 사용  

##### 2.1.1. 하이버네이트 4 의존 추가
<br/>
먼저 해야 할 작업은 하이버네이트 4 모듈을 클래스패스에 추가하는 것이다. 메이븐 설정에서는 pom.xml 파일에 다음과 같은 의존을 추가해주면 된다. 다음은 하이버네이트 4.2.12 버전을 기준으로 메이븐의 의존을 설정한 예이다.  

```xml
<dependencies>
	<dependency>
		<groupId>org.hibernate</groupId>
		<artifactId>hibernate-core</artifactId>
		<version>4.2.12.Final</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-orm</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-tx</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.aspectj</groupId>
		<artifactId>aspectjweaver</artifactId>
		<version>1.7.4</version>
	</dependency>
	...
</dependencies>
```

##### 2.1.2. LocalSessionFactoryBean과 트랜잭션 관리자 설정
<br/>
다음으로 할 작업은 스프링 설정 작업이다. 하이버네이트만 사용할 경우 hibernate.cfg.xml 파일을 이용해서 DB 연결 정보, 트랜잭션 관련 정보, 매핑 목록 등을 설정하는데, 스프링을 사용할 경우 LocalSessionFactoryBean을 이용해서 이들 정보를 설정한다. 따라서, 별도의 하이버네이트 설정 파일을 작성하지는 않는다.  

하이버네이트를 스프링과 연동할 때 사용되는 빈은 다음과 같다.  

+ org.springframework.orm.hibernate4.LocalSessionFactoryBean  
하이버네이트의 SessionFactory를 생성하기 위한 팩토리 빈.  
+ org.springframework.orm.hibernate4.HibernateTransactionManager  
스프링의 트랜잭션 관리 기능과 하이버네이트의 트랜잭션을 연동해주는 트랜잭션 관리자.  

설정 방법은 다음과 같다.  

```xml
<bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />

<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
	<property name="annotatedClasses">
		<list>
			<value>net.madvirus.spring4.chap13.store.domain.Item</value>
			<value>net.madvirus.spring4.chap13.store.domain.PaymentInfo</value>
			<value>net.madvirus.spring4.chap13.store.domain.Purchase</value>
		</list>
	</property>
	<property name="hibernateProperties">
		<value>
			hibernate.dialect=rog.hibernate.dialect.MySQL5InnoDBDialect
		</value>
	</property>
</bean>

<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
	<property name="sessionFactory" ref="sessionFactory" />
</bean>
```

&#64;Configuration을 이용한 설정을 사용할 경우, 다음과 같이 LocalSessionFactoryBean과 HibernateTransactionManager를 설정한다.  

```java
@Configuration
@EnableTransactionManagement
public class 빈설정클래스명 {
	
	@Bean(destroyMethod = "close")
	public DataSource dataSource() {
		ComboPooledDataSource ds = new ComboPooledDataSource();
		...
		return ds;
	}

	@Bean
	public PersistenceExceptionTranslationPostProcessor persistenceExceptionTranslationPostProcessor() {
		return new PersistenceExceptionTranslationPostProcessor();
	}

	@Bean
	public PlatformTransactionManager transactionManager() {
		HibernateTransactionManager txMgr = new HibernateTransactionManager();
		txMgr.setSessionFactory(sessionFactoryBean().getObject());
		return txMgr;
	}

	@Bean
	public LocalSessionFactoryBean sessionFactoryBean() {
		LocalSessionFactoryBean sessionFactoryBean = new LocalSessionFactoryBean();
		sessionFactoryBean.setDataSource(dataSource());
		sessionFactoryBean.setMappingResources("hibernate/Item.hbm.xml", "hibernate/Order.hbm.xml");
		Properties prop = new Properties();
		prop.setProperty("hibernate.dialect", "org.hibernate.dialect.MySQL5InnoDBDialect");
		sessionFactoryBean.setHibernateProperties(prop);
		return sessionFactoryBean;
	}

	...

}
```

##### 2.1.3. DAO에서 SessionFactory 사용하기
<br/>
SessionFactory를 설정했다면, 그 다음으로 할 일은 SessionFactory를 필요로 하는 빈에서 SessionFactory를 사용하는 것뿐이다.  

앞서 설정한 LocalSessionFactory를 DI로 전달해주면 된다.  

LocalSessionFactoryBean 클래스는 하이버네이트의 컨텍스트 세션을 지원하기 때문에, SessionFactory.getCurrentSession() 메서드를 이용해서 SessionFactory를 구하면 된다. 이렇게 구한 SessionFactory는 스프링이 제공하는 트랜잭션 관리 기능과 연동된다. 따라서, 하이버네이트의 트랜잭션 관련 코드를 사용할 필요 없이, 스프링이 제공하는 트랜잭션 관리 기능(선언적 트랜잭션이나 TransactionTemplate)을 그대로 사용하면서 하이버네이트 코드를 스프링이 관리하는 트랜잭션 범위 내에서 실행할 수 있다.  

#### 2.2. 하이버네이트 3 버전 연동 설정
<br/>
하이버네이트 4 버전 설정은 하이버네이트 4 버전 설정과 다르지 않다. 차이점이 있다면, 패키지 이름이 org.springframework.orm.hibernate4 대신 org.springframework.orm.hibernate3을 사용한다는 것과 애노테이션 기반 매핑 설정을 사용할 경우 AnnotationSessionFactoryBean 클래스를 이용해서 설정한다는 점이다.  

##### 2.2.1. 하이버네이트 3 의존 추가
<br/>
하이버네이트 3 버전을 사용하려면 관련 의존 모듈을 추가해주어야 한다. 다음은 메이븐 설정 예시 코드이다.  

```xml
<dependencies>
	<!-- 하이버네이트 3 관련 의존 모듈 -->
	<dependency>
		<groupId>org.hibernate</groupId>
		<artifactId>hibernate-core</artifactId>
		<version>3.6.10.Final</version>
	</dependency>

	<dependency>
		<groupId>javassist</groupId>
		<artifactId>javassist</artifactId>
		<version>3.12.0.GA</version>
	</dependency>

	<!-- 스프링 관련 의존 설정 -->
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-orm</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-tx</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.aspectj</groupId>
		<artifactId>aspectjweaver</artifactId>
		<version>1.7.4</version>
	</dependency>

	...
</dependencies>
```

##### 2.2.2. LocalSessionFactoryBean과 트랜잭션 관리자 설정
<br/>
다음으로 할 작업은 스프링 설정 작업이다. 여기서는 다음 두 빈을 설정한다.  

+ org.springframework.hibernate3.LocalSessionFactoryBean  
하이버네이트의 Session Factory를 생성하기 위한 팩토리 빈.  
+ org.springframework.orm.hibernate3.HibernateTransactionManager  
스프링의 트랜잭션 관리 기능과 하이버네이트의 트랜잭션을 연동해주는 트랜잭션 관리자.  

설정 방법은 다음과 같다.  

```xml
<bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />

<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
	<property name="annotationClasses">
		<list>
			<value>net.madvirus.spring4.chap13.store.domain.Item</value>
			<value>net.madvirus.spring4.chap13.store.domain.PaymentInfo</value>
			<value>net.madvirus.spring4.chap13.store.domain.Purchase</value>
		</list>
	</property>
	<property name="hibernateProperties">
		<value>
			hibernate.dialect=rog.hibernate.dialect.MySQL5InnoDBDialect
		</value>
	</property>
</bean>

<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
	<property name="sessionFactory" ref="sessionFactory" />
</bean>
```
LocalSessionFactoryBean과 HibernateTransactionManager의 설정은 앞서 설명했던 하이버네이트 4 버전과 동일하므로 설명을 생략한다.  

XML 설정 파일 대신 &#64;Entity 등의 애노테이션을 이용해서 매핑 정보를 자바 코드에 설정했다면, LocalSessionFactoryBean 대신 AnnotationSessionFactoryBean 클래스를 이용해야 한다. 다음은 설정 예이다.  

```xml
<bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />

<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
	<property name="annotatedClasses">
		<list>
			<value>net.madvirus.spring4.chap13.store.domain.Item</value>
			<value>net.madvirus.spring4.chap13.store.domain.PaymentInfo</value>
			<value>net.madvirus.spring4.chap13.store.domain.Purchase</value>
		</list>
	</property>
	<property name="hibernateProperties">
		<value>
			hibernate.dialect=rog.hibernate.dialect.MySQL5InnoDBDialect
		</value>
	</property>
</bean>

<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
	<property name="sessionFactory" ref="sessionFactory" />
</bean>
```

&#64;Configuration 애노테이션을 이용한 스프링 설정을 사용할 겨우, 설정 자바 코드 작성 방법은 하이버네이트 4 버전과 동일하다.  

##### 2.2.3. DAO에서 SessionFactory 사용하기
<br/>
스프링 3 버전에 맞게 SessionFactory를 설정했다면, 그 다음으로 할 일은 SessionFactory를 필요로 하는 빈에서 SessionFactory를 사용하는 것이다. 이 과정은 하이버네이트 4 버전을 사용할 때와 동일하다.  

#### 2.3. LocalSessionFactoryBean의 주요 프로퍼티
<br/>
하이버네이트 3과 4 버전을 위한 LocalSessionFactoryBean의 주요 설정 프로퍼티는 다음과 같다. 프로퍼티 중 (&#42;) 표시한 것은 하이버네이트 3 버전 기준으로 AnnotationSessionFactoryBean에서 제공한다.  

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
			<td>dataSource</td>
			<td>DataSource</td>
			<td>DB 연결에 사용할 DataSource를 지정한다.</td>
		</tr>
		<tr>
			<td>mappingResources</td>
			<td>String[]</td>
			<td>클래스패스에 취치한 매핑 경로 목록을 지정한다.</td>
		</tr>
		<tr>
			<td>mappingLocations</td>
			<td>Resources[]</td>
			<td>스프링 Resource 타입으로 매핑 파일 경로 목록을 지정한다.</td>
		</tr>
		<tr>
			<td>mappingDirectoryLocations</td>
			<td>Resources[]</td>
			<td>지정한 디렉토리 및 하위 디렉토리에 위치한 모든 &#42;.hbm.xml 파일을 매핑 파일로 사용한다.</td>
		</tr>
		<tr>
			<td>entityInterceptor</td>
			<td>Interceptor</td>
			<td>하이버네이트의 엔티티 Interceptor를 설정한다. (org.hibernate.Interceptor 구현 객체)</td>
		</tr>
		<tr>
			<td>hibernateProperties</td>
			<td>Properties</td>
			<td>하이버네이트 설정 프로퍼티를 지정한다.</td>
		</tr>
		<tr>
			<td>annotatedClasses (&#42;)</td>
			<td>Class[]</td>
			<td>애노테이션 매핑 정보를 담은 클래스 목록을 지정한다.</td>
		</tr>
		<tr>
			<td>packagesToScan (&#42;)</td>
			<td>String[]</td>
			<td>애노테이션 매핑 정보를 담은 클래스를 검색할 패키지 목록을 지정한다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>

#### 2.4. 하이버네이트와 JTA 트랜잭션 사용 설정
<br/>
JTA를 이용해서 트랜잭션을 관리해야 할 경우, JtaTransactionManager를 사용해야 한다. JtaTransactionManager가 관리하는 트랜잭션을 하이버네이트의 SessionFactory에 적용하려면, LocalSessionFactoryBean의 jtaTransactionManager 프로퍼티에 JtaTransactionManager 객체를 전달해주어야 한다. 다음은 JtaTransactionManager를 사용할 때의 하이버네이트 설정 예를 보여주고 있다.  

```xml
<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager">
	...
</bean>

<!-- XA를 위한 DataSource 설정 -->
<bean id="shopDataSource" class="com.atomikos.jdbc.AtomikosDataSourceBean">
	...
</bean>

<bean id="payDataSource" class="com.atomikos.jdbc.AtomikosDataSourceBean">
	...
</bean>

<!-- 각 DataSource를 위한 SessionFactory 설정 -->
<bean id="shopSessionFactory" class="rog.springframework.orm.hibernate4.LocalSessionFactoryBean">
	<property name="dataSource" ref="shopDataSource" />
	...
	<property name="jtaTransactionManager" ref="transactionManager" />
</bean>

<bean id="paySessionFactory" class="org,springframework.orm.hibernate4.LocalSessionFactoryBean">
	<property name="dataSource" ref="payDataSource" />
	...
	<property name="jtaTransactionManager" ref="transactionManager" />
</bean>

...
```

위 설정에서 중요한 점은 두 개의 SessionFactory를 생성할 때 서로 다른 DataSource를 사용하도록 했고, 두 SessionFactory를 JTA 트랜잭션으로 묶기 위해 jtaTransactionManager 프로퍼티에 JtaTransactionManager 빈 객체를 전달했다는 점이다. 이렇게 함으로써 스프링이 제공하는 트랜잭션 관리 범위에서 두 개의 서로 다른 하이버네이트 세션을 실행할 수 있게 된다.  

### 3. JPA 연동 지원(하이버네이트 4 기준)
<br/>
스프링은 하이버네이트와 비슷한 방식으로 JPA(Java Persistence API) 연동을 지원한다. JPA는 오라클에서 정의한 자바 ORM 표준으로 스프링 4 버전은 JPA 2.1/2.0을 지원하고 있다. JPA 표준을 지원하는 프로바이더로는 하이버네이트, EclipseLink, OpenJPA 등이 존재하는데, 이 중에서 가장 널리 사용되는 JPA 프로바이더로 하이버네이트를 들 수 있다.  

JPA는 지연 로딩, 객체 간 연관 처리 등을 위해 클래스 로딩 과정에서 클래스를 변경한다. JPA 프로바이더마다 클래스를 변환하는 방식이 다른데, 이를 위해 별도의 JVM 에이전트를 사용하기도 한다. 스프링의 경우 LoadTimeWeaver를 이용해서 실행 환경에 따라 알맞게 클래스 변환 처리를 할 수 있도록 하고 있다. 하이버네이트는 별도의 클래스 변환 과정이 필요 없기 때문에, 다른 JPA 프로바이더로 하이버네이트를 선호한다.  

스프링에 JPA를 설정하는 과정은 다음과 같다.  

+ 의존에 JPA 프로바이더 모듈 추가
+ 스프링 설정
+ LocalContainerEntityManagerFactoryBean으로 EntityManagerFactory 설정
+ JpaTransactionManager로 트랜잭션 관리자 설정
+ DAO에서 EntityManagerFactory나 EntityManager 사용  

#### 3.1. JPA 프로바이더 모듈 의존 설정
<br/>
먼저 할 일은 JPA 프로바이더 모듈을 메이븐 의존 설정에 추가하는 것이다. 다음과 같이 하이버네이트 의존을 추가해주면 된다.  

```xml
<dependencies>
	<dependency>
		<groupId>org.hibernate</groupId>
		<artifactId>hibernate-entitymanager</artifactId>
		<version>4.3.4.Final</version>
	</dependency>
	...
</dependencies>
```

hibernate-entitymanager 모듈은 하이버네이트의 JPA 프로바이더 모듈이다. 위 의존 설정에서는 하이버네이트 4.3 버전을 사용했는데, 하이버네이트 4.3 버전은 JPA 2.1 버전을 지원한다. JPA 2.0 버전을 사용하려면 하이버네이트 4.2나 3.6 버전을 사용하면 된다.  

위 메이븐 의존 설정을 보면 하이버네이트 4.3 버전 모듈만 설정하고 JPA 2.1 스펙 의존을 설정하지 않았는데, 그 이유는 하이버네이트 4.3 모듈을 의존에 추가하면, 하이버네이트가 사용하는 JPA 2.1 스펙 모듈이 자동으로 포함되기 때문이다.  

#### 3.2. LocalContainerEntityManagerFactoryBean과 트랜잭션 관리자 설정
<br/>
DB 연결과 트랜잭션 관리는 스프링이 처리하기 때문에 JPA의 설정 파일인 persistence.xml에는 매핑 관련 정보와 JPA 프로바이더에 특화된 프로퍼티만 설정하면 된다. 클래스패스의 META-INF/persistence.xml 파일이 다음과 같다고 해보자.  

```xml
<?xml version="1.0" encoding="UTF-8" />
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence_2_1.xsd"
	version="2.1">
	<persistence-unit name="store">
		<class>net.madvirus.spring4.chap13.store.domain.Item</class>
		<class>net.madvirus.spring4.chap13.store.domain.PaymentInfo</class>
		<class>net.madvirus.spring4.chap13.store.domain.PurchaseOreder</class>
		<exclude-unlisted-classes>false</exclude-unlisted-classes>
	</persistence-unit>
</persistence>
```

위 persistence.xml을 위한 LocalContainerEntityManagerFactoryBean 쿨래스를 설정할 때에는 다음과 같이 영속성 단위 이름인 "store"를 persistenceUnitName 프로퍼티 값으로 지정하면 된다. dataSource 프로퍼티는 DB 연결에 사용할 DataSource를 전달한다. 트랜잭션 관리자로는 JpaTransactionManager를 사용한다.  

```xml
<bean id="jpaVendorAdapter" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
	<property name="dataSource" value="MYSQL" />
</bean>

<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	<property name="persistenceUnitName" value="store" />
	<property name="dataSource" value="dataSource" />
	<property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
</bean>

<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
	<property name="entityManagerFactory" ref="entityManagerFactory" />
</bean>
```

LocalContainerEntityManagerFactoryBean 설정에서 눈여겨볼 부분은 jpaVendorAdapter 프로퍼티다. 이 프로퍼티는 JPA 프로바이더에 알맞은 설정을 제공하기 위한 어댑터 클래스로 DB, SQL 출력 여부 등을 설정한다. 위 코드에서는 하이버네이트를 위한 어댑터 HibernateJpaVendorAdapter 클래스를 사용했다. (이외에 OpenJpaVendorAdapter 클래스와 EclipseLinkJpaVendorAdapter를 제공하고 있다.)벤더 어댑터를 통해 설정할 수 있는 프로퍼티는 다음과 같다.  

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
			<td>database</td>
			<td>org.springframework.jpa.vendor.Database</td>
			<td>DB 종류를 지정한다.</td>
		</tr>
		<tr>
			<td>databasePlatform</td>
			<td>String</td>
			<td>프로바이더에 맞는 DB 관련 값을 지정한다. 하이버네이트의 경우 "hibernate.dialect" 설정 프로퍼티 값으로 사용할 클래스 이름을 지정한다.</td>
		</tr>
		<tr>
			<td>generateDdl</td>
			<td>boolean</td>
			<td>EntityManagerFactory를 초기화할 때 관련된 테이블을 생성할지 여부를 지정한다. 기본값은 false이다.</td>
		</tr>
		<tr>
			<td>showSql</td>
			<td>boolean</td>
			<td>실행하는 쿼리를 로그로 기록할지 여부를 지정한다. 기본값은 false이다.</td>
		</tr>
	</tbody>
</table>

JPA 프로바이더가 올바르게 동작하려면 DBMS를 알맞게 지정해주어야 한다. 따라서, database 프로퍼티와 databasePlatform 프로퍼티 중 하나는 반드시 설정해주어야 한다. 그렇지 않을 경우 정상적으로 동작하지 않을 수 있다.

스프링 4.0.4. 버전 기준으로 Database 열거 타입이 정의하고 있는 DBMS 목록은 다음과 같다.  

+ DEFAULT
+ DB2
+ DERBY
+ H2
+ HSQL
+ INFORMIX
+ MYSQL
+ ORACLE
+ POSTGRESQL
+ SQL&#95;SERVER
+ SYBASE  

JpaTransactionManager는 스프링 트랜잭션과 JPA 트랜잭션을 연동해준다. entityManagerFactory 프로퍼티를 이용해서 트랜잭션을 연동할 EntityManagerFactory 빈을 지정한다.  

&#64;Configuration을 이용한 LocalContainerEntityManagerFactoryBean 설정 예는 다음 코드와 같다.  

```java
@Configuration
@EnableTrnasactionManagement
public class 빈설정클래스명 {

	@Bean(destoryMethod = "close"(
	public DataSource dataSource() {
		ComboPooledDataSource ds = new ComboPooledDataSource();
		...
		return ds;
	}

	@Bean
	public PersistenceExceptionTrnaslationPostProcessor persistenceExceptionTranslationPostProcessor() {
		return new PersistenceExceptionTranslationPostProcessor();
	}

	@Bean
	public LocalContainerEntityManagerFactoryBean emf() {
		LocalContainerEntityManagerFactoryBean emfBean = new LocalContainerEntityManagerFactoryBean();
		emfBean.setDataSource(dataSource());
		emfBean.setPersistenceUnitName("store");
		HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
		emfBean.setJpaVendorAdapter(vendorAdapter);
		return emfBean;
	}

	@Bean
	public PlatformTransactionManager transactionManager(EntityManagerFactory emFactory) {
		JpaTrnasactionManager txMgr = new JpaTransactionManager();
		txMgr.setEntityManagerFactory(emFactory);
		return txMgr;
	}

	...

}
```

#### 3.3. EntityManagerFactory와 EntityManager 사용하기
<br/>
JPA를 이용해서 DB 연동 코드를 작성하려면, EntityManagerFactory에서 EntityManager를 구해야 한다. 가장 간단한 방법은 EntityManagerFactory를 DI를 통해서 전달받는 것이다. 예를 들어, 다음 코드처럼 설정 메서드를 이용해서 EntityManagerFactory를 전달받도록 구현한 코드가 있다고 하자.  

```java
public class JpaItemRepository implements ItemRepository {

	private EntityManagerFactory entityManagerFactory;

	public void setEntityManagerFactory(EntityManager emf) {
		this.entityManagerFactory = emf;
	}

	@Override
	public Item findById(Integer itemId) {
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		entityManager.joinTranslation();
		return entityManager.find(Item.class, itemId);
	}

}
```

스프링 설정에서는 설정 프로퍼티에 EntityManagerFactory 빈을 전달해주면 끝이다.  

```xml
<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	...
</bean>


<bean id="itemRepository" class="net.madvirus.spring4.chap13.store.persistence.JpaItemRepository">
	<property name="entityManagerFactory" ref="entityManagerFactory" />
</bean>
```

EntityManagerFactory를 전달받는 또 다른 방법은 javax.persistence.PersistenceUnit 애노테이션을 이용하는 것이다. 스프링은 &#64;PersistenceUnit 애노테이션을 지원하고 있으며, &#64;PersistenceUnit 애노테이션이 적용된 대상에 등록된 EntityManagerFactory 빈을 할당한다.  

```java
public class JpaPaymentInfoRepository implements PaymentInfoRepository {

	@PersistenceUnit
	private EntityManagerFactory entityManagerFactory;

	@Override
	public void save(PaymentInfo paymentInfo) {
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		entityManager.joinTransaction();
		entityManager.persist(paymentInfo);
	}

}
```

&#64;PersistenceUnit 애노테이션을 적용하려면 PersistenceAnnotationBeanPostProcessor 클래스를 빈 객체로 등록해주어야 한다. 또는, PersistenceAnnotationBeanPostProcessor 대신에 &#60;context:annotation-config&#62; 태그를 사용해도 된다. AnnotationConfigApplicationContext와 같은 애노테이션 기반 스프링 컨테이너를 사용하면 기본으로 등록된다.  


```xml
<bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor" />

<!-- 또는 아래 태그 사용
<context:annotation-config />
-->
```

JPA를 이용한 크도를 보면 EntityManagerFactory로부터 생성한 EntityManager를 이용해서 DB 연동 코드를 작성한다. EntityManagerFactory로부터 매번 EntityManager를 가져올 수 있지만, 현재 진행중인 트랜잭션 범위에서 EntityManager가 동작하게 하려면 아래 코드처럼 EntityManager.jointTransaction() 메서드를 실행해야 한다.  

```java
@Repostory
public class JpaPaymentInfoDao implements PaymentInfoDao {

	private EntityManagerFactory emf;

	@Override
	public void insert(PaymentInfo paymentInfo) {
		EntityManager entityManager = emf.createEntityManager();
		entityManager.joinTransaction(); // 현재 진행중인 트랜잭션에 참여
		entityManager.persist(paymentInfo);
		...
	}

	...

}
```

EntityManager.joinTransaction() 메서드를 호출하면 현재 진행중인 트랜잭션에 EntityManager를 연동할 수 있지만, 그것보다는 아래 코드와 같이 &#64;PersistenceContext 애노테이션을 이용하면 트랜잭션에 이미 연동된 EntityManager를 사용할 수 있다.  

```java
public class JpaPurchaseOrderRepository implements PurchaseOrderRepository {

	@PersistenceContext
	private EntityManager entityManager;

	@Override
	public void save(PurchaseOrder order) {
		entityManager.persist(order);
	}

}
```

스프링은 &#64;PersistenceContext 애노테이션이 적용된 대상에 프록시 객체를 할당한다. 이 프록시 객체는 내부적으로 현재 진행중인 트랜잭션과 연동된 EntityManager에 모든 요청을 위임한다. 예를 들어, 위 코드에서 entityManager.persist(order) 코드를 실행하면 실제로는 스프링이 제공한 프록시 객체의 persist() 메서드가 호출되며, 프록시의  persist() 메서드는 현재 진행중인 트랜잭션과 연동된 EntityManager의 persist() 메서드를 호출한다.  

&#64;PersistenceContext 애노테이션이 올바르게 동작하려면 &#64;PersistenceUnit 애노테이션과 마찬가지로 PersistenceAnnotationBeanPostProcessor 클래스나 &#60;context:annotation-config&#62; 태그를 설정 파일에 등록해주어야 한다. AnnotationConfigApplicationContext와 같은 애노테이션 기반 스프링 컨테이너를 사용하면 기본으로 등록된다.  

#### 3.4. LocalContainerEntityManagerFactoryBean 클래스의 기타 설정
<br/>
##### 3.4.1. 영속성 설정 XML 파일 경로 사용과 영속 단위 이름
<br/>
LocalContainerEntityManagerFactoryBean 클래스는 JPA 설정을 위한 다양한 프로퍼티를 제공하고 있다. 스프링은 기본적으로 classpath:/META-INF/persistence.xml 경로에서 JPA 영속 단위 정보를 읽어오는데, 이 경로가 아닌 다른 경로에 위치한 파일을 사용하고 싶다면 persistenceXmlLocation 프로퍼티를 이용하면 된다.  

```xml
<bean id="entityManagerFactory" class="org,springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	<property name="persistenceXmlLocation" value="classpath:/META-INF/otherpath.xml" />
	<property name="persistenceUnitName" value="store-other" />
	...
</bean>
```

persistenceUnitName 프로퍼티는 영속성 설정 XML 파일에 있는 영속성 단위 이름을 지정한다. persistenceUnitName 프로퍼티를 지정하지 않을 경우, 지정한 영속성 설정 XML 파일에 명시된 영속성 단위 이름을 사용한다.  

##### 3.4.2. JPA 매핑 클래스 스캔
<br/>
persistence.xml 파일을 사용하는 대신 클래스 스캔 기능을 이용해서 매핑 클래스 정보를 읽어올 수 있다. 이를 위해 해야 할 작업은 packagesToScan 프로퍼티에 클래스를 검색할 패키지 목록을 지정하는 것이다.  

```xml
<bean id="entityManagerFactory" class="org,springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	<property name="packagesToScan">
		<list>
			<value>net.madvirus.spring4.chap13.store.domain</value>
		</list>
	</property>
	<property name="dataSource" ref="dataSource" />
	<property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
	<property name="jpaProperties">
		<value>
			hibernate.format_sql = true
		</value>
	</property>
</bean>
```

위 설정을 사용하면 지정한 패키지 및 그 하위 패키지에 위치한 JPA 매핑 클래스를 검색해서 JPA 매핑 정보로 사용하게 된다. 따라서, 매핑 클래스 목록을 지정하기 위해 별도의 persistence.xml 파일을 작성하지 않아도 된다.  

##### 3.4.3. jpaPropertyMap과 jpaProperties를 이용한  프로바이더 설정 추가
<br/>
JpaVendorAdapter 클래스를 사용하면 JPA 프로바이더들이 공통으로 제공하는 프로퍼티를 설정할 수 있는데, 이외에 특정 JPA 프로바이더를 위한 설정을 추가하고 싶다면 jpaPropertyMap 프로퍼티나 jpaProperties 프로퍼티를 이용하면 된다.  


#### 3.5. JPA와 JTA 연동(하이버네이트 4.3 기준)
<br/>
JPA와 JTA 트랜잭션을 연동하는 것은 간단한 듯 하면서도 설정에 어려움이 따른다. 각 JPA 프로바이더와 사용하는 JTA 구현에 따라 연동 설정 방법이 조금씩 다르기 때문이다.  
하이버네이트 4.3 기반 JPA 프로바이더가 JTA 트랜잭션과 연동되도록 설정하려면 다음과 같이 LocalContainerEntityManagerFactoryBean의 jpaProperties 프로퍼티 또는 jpaPropertyMap 프로퍼티를 이용해서 하이버네이트의 hibernate.transaction.jta.platform 설정 속성을 JTA 플랫폼에 맞게 설정해주어야 한다.  

```xml
<!-- JTA 관련 설정 생략 -->

<!-- XaDataSource 관련 설정 생략 -->
<bean id="shopDataSource" class="com.atomikos.jdbc.AtomikosDataSourceBean">
	<property name="uniqueResourceName" value="shopXaDs" />
	...
</bean>

<!-- JPA EntityManagerFactory 설정 -->
<bean id="shopEntityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManageFactoryBean">
	<property name="persistenceXmlLocation" value="/META-INF/conf-4-jta1.xml" />
	<property name="persistenceUnitName" value="shop" />
	<property name="jtaDataSource" ref="shopDataSource" />
	<property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
	<property name="jpaProperties">
		<props>
			<prop> key="hibernate.transaction.jta.platform">
				net.madvirus.spring4.chap13.atomikos.AtomikosJtaPlatform
			</prop>
		</props>
	</property>
</bean>

<bean id="jpaVendorAdapter" class="org,springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
	<property name="dataSource" value="MYSQL" />
	<property name="showSql" value="true" />
</bean>
```

하이버네이트의 hibernate.transaction.jta.platform 설정 속성은 하이버네이트 4 버전에서 사용되는 설정으로 하이버네이트가 JTA 트랜잭션 관리자를 찾을 때 사용할 JtaPlatform 구현체를 지정한다. 하이버네이트는 각 컨테이너마다 알맞은 JtaPlatform 구현 클래스를 제공하고 있는데, TrnasactionsEssentials의 경우 하이버네이트에 기본으로 포함되어 있지 않다. TransactionsEssentials은 자제적으로 하이버네이트 연동에 필요한 구현 클래스를 제공하고 있는데, TransactionsEssentials 3.9 버전까지는 하이버네이트 4 지원을 위한 클래스가 제공되고 있지 않다. 이런 이유로 하이버네이트 4의 JtaPlatform을 TransactionsEssentials 3.9 버전에 맞게 구현한 클래스를 사용해야 한다. 소스 코드는 다음과 같다.  

```java
public class AtomikosJtaPlatform extends AbstractJtaPlatform {
	private static final long serialVersionUID = 1L;

	private UserTransactionManager utm;

	public AtomikosJtaPlatform() {
		utm = new UserTransactionManager();
	}

	@Override
	protected TransactionManager locateTransactionManager() {
		return utm;
	}

	@Override
	protected UserTransaction locationUserTransaction() {
		return utm;
	}
}
```

개발 중인 TransactionsEssentials 4 버전에서는 하이버네이트 4와 연동에 필요한 모든 클래스를 제공할 예정이다.  

하이버네이트 4.3은 각 플랫폼에 맞는 JtaPlatform 구현체를 겢공하고 있는데, 주요 항목은 다음과 같다.  

+ org.hibernate.engine.transaction.jta.platform.internal.WeblogicJtaPlatform
+ org.hibernate.engine.transaction.jta.platform.internal.WebSphereJtaPlatform
+ org.hibernate.engine.transaction.jta.platform.internal.JBossAppServerJtaPlatform
+ org.hibernate.engine.transaction.jta.platform.internal.ResinJtaPlatform
+ org.hibernate.engine.transaction.jta.platform.internal.JOTMJtaPlatform
+ org.hibernate.engine.transaction.jta.platform.internal.BitronixJtaPlatform

### 4, MyBatis 연동 지원
<br/>
스프링 4 버전에서는 MyBatis/iBATIS와의 연동 기능이 포함되어 있지 않다. 대신 MyBatis가 직접 스프링과 MyBatis를 연동하기 위한 모듈을 제공하고 있으므로 이 모듈을 사용하면 스프링이 제공하는 DataSource 및 트랜잭션 관리 기능을 MyBatis에 적요할 수 있다.  

MyBatis를 스프링과 연동하는 방법은 다음과 같다.  

+ MyBatis-Spring 모듈 추가
+ SqlSessionFactoryBean을 이용해서 SqlSessionFactory 설정
+ 트랜잭션 설정
+ MyBatis를 이용한 DAO 구현
+ SqlSession 이용 구현
+ 매퍼 동적 생성 이용 구현

#### 4.1. MyBatis Spring 모듈 추가
<br/>
가장 먼저 할 작업은 아래 코드처럼 MyBatis 및 MyBatis-Spring 모듈을 추가하는 것이다.  

```xml
<dependencies>
	<dependency>
		<groupId>org,springframework</groupId>
		<artifactId>spring-context</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-jdbc</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org,mybatis</groupId>
		<artifactId>mybatis-spring</artifactId>
		<version>1.2.2</version>
	</dependency>
	<dependency>
		<groupId>org,mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.2.3</version>
	</dependency>
	...
</dependencies>
```

mybatis-spring 모듈은 스프링의 DataSource 및 트랜잭션 관리 기능을 MyBatis와 연동하는데 필요한 기능을 제공하고 있다. 이 중 핵심 클래스는 SqlSessionFactoryBean과 SqlSessionTemplate이다.  

#### 4.2. SqlSessionFactoryBean과 트랜잭션 관리자 설정
<br/>
스프링의 DB 관련 기능과 MyBatis를 연동하려면, mybatis-spring 모듈이 제공하는 SqlSessionFactoryBean을 이용해서 mybatis의 SqlSessionFactory를 생성해야 한다.  

```xml
<bean id="sqlSessionFactory" class="org,mybatis.spring.SqlSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
	<property name="mapperLocations">
		<list>
			<value>classpath:/mybatis/itemdao.xml</value>
			<value>classpath:/mybatis/purchaseorderdao</value>
		</list>
	</property>
	<property name="typeAliases">
		<list>
			<value>net.madvirus.spring4.chap13.store.model.PurchaseOrder</value>
		</list>
	</property>
</bean>
```

dataSource 프로퍼티는 DB 연결을 구할 때 사용할 DataSource를 설정한다.  

mapperLocations 프로퍼티는 매핑 쿼리를 담고 있는 파일의 목록을 지정한다. 예를 들어, 책의 예제에서 사용하는 purchaseorderdao.xml 파일은 다음과 같이 작성했다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper PUBLIC "~//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="net.madvirus.spring4.chap13.store.dao.PurchaseOrderDao">

	<insert id="save" parameterType="PurchaseOrder" useGeneratedKeys="true" keyProperty="id">
		insert PURCHASE_ORDER (ITEM_ID, PAYMENT_INFO_ID, ADDRESS)
		values (#{itemId}, #{paymentInfoId}, #{address})
	</insert>

</mapper>
```

typeAlias 프로퍼티는 매퍼 XML 파일에서 완전한 클래스 이름 대신 별칭을 사용할 클래스 목록을 지정할 때 사용한다. 앞서 SqlSessionFactoryBean 클래스 설정에서는 typeAlias 프로퍼티 값으로 PurchaseOrder 클래스를 설정했는데, 이 클래스는 다음과 같이 MyBatis의 &#64;Alias 애노테이션을 이용해서 매퍼 XML 파일에서 사용할 타입 별칭을 지정한다. (위 코드에서 &#60;insert&#62; 태그의 parameterType 속성 값으로 PurchaseOrder를 사용했는데, 이는 아래 코드의 &#64;Alias 애노테이션으로 지정한 값이다.)

```java
@Alias("PurchaseOrder")
public class PurchaseOrder {
	...
}
```

MyBatis를 위한 트랜잭션 관리자로는 DataSourceTransactionManager를 사용하면 된다.  

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource,DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>
```

다음 코드는 &#64;Configuration 자바 설정을 이용할 때 SqlSessionFactory의 설정 예이다.  

```java
@Configuration
@EnableTransactionManagement
public class 빈설정클래스명 {

	@Bean
	public PersistenceExceptionTrnaslactionPostProcessor postProcessor() {
		return new PersistenceExceptionTranslationPostProcessor();
	}

	@Bean(destroyMethod = "close")
	public DataSource dataSource() {
		...
	}

	@Bean
	public PlatformTransactionManager transactionManager() {
		DataSourceTransactionManager tm = new DataSourceTransactionManager();
		tm.setDataSource(dataSource());
		return tm;
	}

	@Bean
	public SqlSessionFactoryBean sqlSessionFactory() throws Exception {
		SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
		factoryBean.setDataSource(dataSource());
		Resource[] mapperLocations = new Resource[2];
		mapperLocations[0] = new ClassPathResource("/mybatis/itemdao.xml");
		mapperLocations[1] = new ClassPathResource("/mybatis/purchaseorderdao.xml");
		factoryBean.setMapperLocations(mapperLocations);
		factoryBean.setTypeAliases(new Class<?>[] { PurchaseOrder.class });
		return factoryBean;
	}
}
```

#### 4.3. MyBatis를 이용한 DAO 구현
<br/>
SqlSessionFactoryBean을 이용해서 SqlSessionFactory를 설정하면, MyBatis를 이용해서 DAO를 구현할 수 있다. MyBatis를 이용한 DAO 구현 방법에는 크게 다음의 두 종류로 분류할 수 있다.  

+ SqlSessionTemplate을 이용한 DAO 구현
+ 자동 매퍼 생성 기능을 이용한 DAO 구현  

##### 4.3.1. SqlSessionTemplate을 이용한 DAO 구현
<br/>
mybatis-spring 모듈은 MyBatis의 SqlSession 기능과 스프링의 DB 지원 기능을 연동해주는 SqlSessionTemplate 클래스를 제공하고 있다. SqlSessionTemplate 클래스는 SqlSession을 위한 스프링 연동 부분을 구현하고 있으며(실제로는 SqlSessionTemplate이 내부적으로 사용하는 SqlSession 프록시 객체가 스프링 연동 처리), DAO는 SqlSessionTemplate을 이용해서 알맞게 DAO를 구현하면 된다.  

다음 코드는 SqlSessionTemplate의 설정 예를 보여주고 있다. SqlSessionTemplate 클래스의 생성자에 sqlSessionFactory를 전달해주면 된다.  

```xml
<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
	<constructor-arg ref="sqlSessionFactory" />
</bean>

<bean id="itemDao" class="net.madvirus.spring4.chap13.store.dao.MyBatisItemDao">
	<property name="sqlSessionTemplate" ref="sqlSessionTemplate" />
</bean>
```

DAO 클래스는 생성자나 프로퍼티를 통해서 SqlSessionTemplate을 전달받아 DB 기능을 구현하면 된다. SqlSessionTemplate은 MyBatis의 SqlSession 인터페이스를 상속받고 있기 때문에, 실제로는 SqlSession에 정의된 메서드를 그대로 사용해서 DAO를 구현하게 된다. 다음 코드는 SqlSessionTemplate을 이용한 DAO의 구현 예를 보여주고 있다.  

```java
@Repository
public class MyBatisItemDao implements ItemDao {

	private SqlSessionTemplate sqlSessionTemplate;

	public void setSqlSessionTemplate(SqlSessionTemplate sqlSession) {
		this.sqlSessionTemplate = sqlSession;
	}

	@Override
	public Item findById(Integer itemId) {
		Item item = (Item) sqlSessionTemplate.selectOne("net.madvirus.spring4.chap13.store.dao.ItemDao.findById", ItemId);
		return item;
	}

}
```

SqlSessionTemplate은 SqlSession 인터페이스를 상속받고 있기 때문에, 다음과 같이 SqlSessionTemplate 대신 SqlSession 타입을 이용해도 된다.  

```java
@Repository
public class MyBatisItemDao implements ItemDao {
	private SqlSession sqlSession;

	public void setSqlSession(SqlSession sqlSession) {
		this.sqlSession = sqlSession;
	}

	@Override
	public Item findById(Integer itemId) {
		Item item = (Item) sqlSession.selectOne("net.madvirus.spring4.chap13.store.dao.ItemDao.findById", itemId);
		return item;
	}

}
```

##### 4.3.2. SqlSessionDaoSupport 클래스를 이용한 DAO 구현
<br/>
SqlSessionDaoSupport 클래스를 상속받아 DAO를 구현할 수도 있다. SqlSessionDaoSupport 클래스는 스프링과 연동된 SqlSession을 제공하는 getSqlSession() 메서드를 포함하고 있으며, 하위 클래스는 이 메서드를 이용해서 SqlSession에 접근할 수 있다. 다음은 SqlSessionDaoSupport 클래스를 상속받은 클래스의 구현 예를 보여주고 있다. 다음은 SqlSessionDaoSupport 클래스를 상속받은 클래스의 구현 예를 보여주고 있다.  

```java
public class MyBatisItemDao2 extends SqlSessionDaoSupport implements ItemDao {

	@Override
	public Item findById(Integer itemId) {
		Item item = (Item) getSqlSession().selectOne("net.madvirus.spring4.chap13.store.dao.ItemDao.findById", itemId);
		return item;
	}

}
```

SqlSessonDaoSupport 클래스가 정상 동작하려면 아래 코드처럼 SqlSessionFactory나 SqlSessionTemplate을 설정해주어야 한다.  

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
	...
</bean>

<bean id="sqlSessionTemplate" class="org.mybatis.spring.sqlSessionTemplate">
	<constructor-arg ref="sqlSessionFactory" />
</bean>

<!-- SqlSessionFactory를 설정 -->
<bean id="itemDao2" class="net.madvirus.spring4.chap13.store.dao.MyBatisItemDao2">
	<property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>

<!-- 또는 SqlSessionTemplate을 설정 -->
<bean id="itemDao2" class="net.madvirus.spring4.chap13.store.dao.MyBatisItemDao2">
	<property name="sqlSessionTemplate" ref="sqlSessionTemplate" />
</bean>
```

##### 4.3.3. 자동 매퍼 생성 기능을 이용한 DAO 구현
<br/>
MyBatis를 이용해서 DAO 구현할 때, 대부분의 코드는 단순히 SqlSession의 메서드를 호출하는 것으로 끝이 난다. 예를 들어, 앞서 봤던 MyBatisItemDao 클래스의 findById() 메서드의 경우 selectOne() 메서드를 호출하는 코드만 실행하고 끝이 난다.  

이런 단순 코드 작업을 줄여주기 위해 MyBatis는 인터페이스를 이용해서 런타임에 매퍼 객체를 생성하는 기능을 제공하고 있다. 스프링에서 개별 매퍼 인터페이스로부터 DAO 빈 객체를 생성하려면 MapperFactoryBean 클래스를 사용하면 된다.  

```xml
<bean id="purchaseOrderDao" class="org.mybatis.spring.mapper.MapperFactoryBean">
	<property name="mapperInterface" value="net.madvirus.spring4.chap13.store.dao.PurchaseOrderDao" />
	<property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>

<bean id="placeOrderService" class="net.madvirus.spring4.chap13,store.service.PlaceOrderServiceImpl">
	<!-- MapperFactoryBean이 생성한 빈 객체 사용 -->
	<property name="purchaseOrderDao" ref="purchaseOrderDao" />
	...
</bean>
```

MapperFactoryBean의 sqlSessionFactory 프로퍼티는 SqlSessionFactory 빈을 설정하고, MapperFactoryBean의 mapperInterface 프로퍼티는 매퍼로 사용할 인터페이스 타입을 지정한다. MapperFactoryBean의 mapperInterface 프로퍼티는 매퍼로 사용할 인터페이스 타입을 지정한다. mapperInterface 프로퍼티로 지정한 타입이 MapperFactoryBean가 생성하는 빈 객체의 타입이 된다. 즉, 위 설정의 경우 MapperFactoryBean이 생성하는 빈 객체의 타입은 PurchaseOrderDao가 된다.  

자동으로 생성된 빈 객체는 인터페이스의 패키지 이름을 포함한 완전한 이름과 메서드 이름을 이용해서 매퍼 XML 파일에서 사용할 쿼리를 찾는다. 예를 들어, 이 예제에서 사용한 PurchaseOrderDao 인터페이스과 다음과 같다고 하자.  

```xml
public interface PurchaseOrderDao {
	void save(PurchaseOrder order);
}
```

MapperFactoryBean이 생성한 PurchaseOrderDao 빈 객체의 save() 메서드를 호출하면, 다음을 기준으로 실행할 쿼리를 찾는다.  

+ 네임스페이스 : PurchaseOrderDao의 완전한 타입 이름(예 : net,madvirus.spring4.chap13.store.dao.PurchaseOrderDao)
+ 실행할 쿼리 식별 값 : 실행할 메서드 이름(예 : save)  

예제에서 사용한 매퍼 설정 XML 파일은 다음과 같은데, PurchaseOrderDao.save()를 실행하면 &#60;insert&#62; 태그로 설정한 쿼리가 실행된다.  

```xml
<mapper namespace="net.madvirus.spring4.chap13.store.dao.PurchaseOrderDao">

	<insert id="save" parameterType="PurchaseOrder" useGeneratedKeys="true" keyProperty="id">
		insert PURCHASE_ORDER (ITEM_ID, PAYMENT_INFO_ID, ADDRESS)
		values (#{itemId}, #{paymentInfoId}, #{address})
	</insert>
</mapper>
```

MyBatis는 XML 매퍼 설정 파일을 작성하는 대신, 애노테이션을 이용해서 매퍼 인터페이스의 메서드에 직접 쿼리를 설정할 수도 있다. 다음은 &#64;Insert 애노테이션의 사용 예이다.  

```java
@Repository
public interface PaymentInfoDao {
	@Insert("insert into PAYMENT_INFO (PRICE) values (#{price})")
	@Options(keyProperty = "id", useGeneratedKeys = true)
	void save(PaymentInfo paymentInfo);
}
```

애노테이션을 이용해서 실행할 쿼리를 설정한 매퍼 인터페이스도 MapperFactoryBean을 통해서 빈 객체를 생성할 수 있다.  

&#64;Configuration 자바 설정에서 자동 매퍼 생성 기능을 사용하려면 다음과 같이 두 가지 방식 중 한 가지를 사용하면 된다.  

+ SqlSessionTemplate,getMapper() 사용
+ MapperFactoryBean 사용  

먼저 SqlSessionTemplate의 getMapper()를 사용하는 방법을 알아보자. SqlSessionFactory를 생성할 때 매퍼 인터페이스의 매칭되는 매퍼 설정을 추가했다면(아래 코드에서 setMapperLocation() 설정 부분), SqlSessionTemplate이 제공하는 getMapper() 메서드를 이용해서 간단하게 매퍼 객체를 생성할 수 있다. 다음은 설정 예제 코드이다.  

```java
@Bean
public SqlSessionFactory sqlSessionFactory() throws Exception {
	SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
	Resource[] mapperLocations = new Resources[2];
	mapperLocations[0] = new ClassPathResource("/mybatis/itemdao.xml");
	// purchaseorderdao.xml 매퍼 파일이 PurchaseOrderDao 인터페이스와 매칭
	mapperLocations[1] = new ClassPathResource("/mybatis/purchaseorderdao.xml");
	factoryBean.setMapperLocations(mapperLocations);
	...
	return factoryBean.getObject();
}

@Bean
public SqlSessionTemplate sqlSessionTemplate() throws Exception {
	return new SqlSessionTemplate(sqlSessionTemplateFactory());
}

// 첫 번째 방법: SqlSessionTemplate의 getMapper() 사용
@Bean
public PurchaseOrderDao purchaseOrderDao() throws Exception {
	// 인터페이스 작성 만으로 매퍼 객체 생성
	return sqlSessionTemplate().getMapper(PurchaseOrderDao.class);
}

// 두 번째 방법 : MapperFactoryBean 사용
@Bean
public PaymentInfoDao paymentInfoDao() throws Exception {
	MapperFactoryBean<PaymentInfoDao> factoryBean = new MapperFactoryBean<>();
	factoryBean.setMapperInterface(PaymentInfoDao.class);
	factoryBean.setSqlSessionFactory(sqlSessionFactory());
	factoryBean.afterPropertiesSet();
	return factoryBean.getObject();
}
```

SqlSessionFactory를 생성할 때, 인터페이스와 매칭되는 매퍼 설정을 등록하지 않는다면, XML 설정과 동일하게 MapperFactoryBean 클래스를 이용하면 된다.  

##### 4.3.4. 스캔을 이용한 매퍼 검색
<br/>
MapperFactoryBean이 단일 매퍼 인테페이스를 빈으로 생성할 때 사용된다면, 스캔은 다수의 매퍼 인터페이스를 검색해서 자동으로 빈으로 등록할 때 사용된다. 스프링의 컴포넌트 스캔 기능과 동일한데 차이라면 검색된 인터페이스가 매퍼 인터페이스로 사용된다는 점이다.  

XML 설정에서 매퍼 스캔 기능을 사용하려면 다음과 같이 &#60;mybatis:scan&#62; 태그를 사용하면 된다.  

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
	xsi:schemaLocation="...
			http://mybatis.org/schema/mybatis-spring
			http://mybatis.org/schema/mybatis-spring.xsd">
	
	<tx:annotation-driven />
	...dataSource/transactionManager 설정 생략

	<mybatis:scan base-packages="net.madvirus.spring4.chap13.store.dao" />

	<bean id="sqlSessionFactory" class="org,mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		...
	</bean>

	<bean id="placeOrderService" class="net.madvirus.spring4.chap13.store.service.PlaceOrderServiceImpl">
	</bean>
</beans>
```

&#60;mybatis:scan&#62; 태그는 base-package 속성으로 설정한 패키지 및 그 하위 패키지의 모든 인터페이스를 검색해서 매퍼로 등록한다. 생성된 매퍼 빈이 필요한 클래스는 스프링의 자동 연결 기능(&#64;Autowired나 &#64;Resource와 같은 애노테이션 등)을 이용해서 빈을 사용하면 된다.  

&#60;mybatis:scan&#62;이 검색할 대상 인터페이스를 제한하고 싶다면 다음의 속성 중 annotation 속성이나 mybatis-interface 솟성을 사용하면 된다.  

+ annotation
특정 애노테이션을 지정한 인터페이스만 검색한다.  
+ marker-interface  
지정한 인테페이스를 상속한 인테페이스만 검색한다.  
+ factory-ref  
SqlSessionFactory 빈의 이름을 지정한다.  
+ template-ref  
SqlSessionTemplate 빈의 이름을 지정한다.  
+ base-package  
인터페이스를 검색할 패키지를 지정한다.  

factory-ref와 template-ref를 둘 다 설정할 경우, factory-ref의 설정이 무시된다. 둘 다 설정하지 않으면 타입 기반 자동 설정을 이용해서 존재하는 SqlSessionFactory 빈과 SqlSessionTemplate 빈 중 하나를 사용한다. 두 속성을 모두 설정하지 않은 상태에서 SqlSessionFactory 타입 빈이나 SqlSessionTemplate 타입 빈이 두 개 이상 존재하면 익셉션이 발생하므로, 가능하면 둘 중 하나를 명시적으로 설정해서 익셉션이 발생할 가능성을 없애는 것이 좋다.  

자바 코드 설정을 사용한다면 &#64;MapperScan 애노테이션을 사용하면 된다.  

```java
@Configuration
@EnableTransactionManagement
@MapperScan("net.madvirus.spring4.chap13.store.dao")
public class 빈설정클래스명 {
	...
}
```

&#64;MapperScan 애노테이션의 속성은 다음과 같다.  

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
			<td>basePackages</td>
			<td>String[]</td>
			<td>스캔 대상 패키지를 지정한다.</td>
		</tr>
		<tr>
			<td>annotationClass</td>
			<td>Class</td>
			<td>특정 애노테이션을 지정한 인테페이스만 검색한다.</td>
		</tr>
		<tr>
			<td>markerInterface</td>
			<td>Class</td>
			<td>지정한 인터페이스를 상속한 인터페이스만 검색한다.</td>
		</tr>
		<tr>
			<td>sqlSessionFactoryRef</td>
			<td>String</td>
			<td>SqlSessionFactory 빈의 이름을 지정한다.</td>
		</tr>
		<tr>
			<td>sqlSessionTemplateRef</td>
			<td>String</td>
			<td>SqlSessionTemplate 빈의 이름을 지정한다.</td>
		</tr>
	</tbody>
</table>

#### 4.4. SqlSessionFactoryBean 기타 설정
<br/>
SqlSessionFactoryBean의 주요 프로퍼티는 다음곽 같다.  

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
			<td>mapperLocations</td>
			<td>Resources[]</td>
			<td>매퍼 XML 파일 위치를 지정한다.</td>
		</tr>
		<tr>
			<td>typeAlias</td>
			<td>Class[]</td>
			<td>&#64;Alias 애노테이션을 가진 클래스 목록을 지정한다.</td>
		</tr>
		<tr>
			<td>typeAliasesPackage</td>
			<td>String</td>
			<td>타입 별칭 정보를 담고 있는 타입을 검색할 패키지를 지정한다.</td>
		</tr>
		<tr>
			<td>typeHandlers</td>
			<td>TypeHandler[]</td>
			<td>타입 핸들러 목록을 지정한다.</td>
		</tr>
		<tr>
			<td>typeHandlersPackage</td>
			<td>String</td>
			<td>타입 핸들러를 검색할 패키지를 지정한다.</td>
		</tr>
		<tr>
			<td>configurationProperties</td>
			<td>Properties</td>
			<td>SqlSession을 위한 프로퍼티를 설정한다.</td>
		</tr>
		<tr>
			<td>databaseIdProvider</td>
			<td>DatabaseIdProvider</td>
			<td>DatabaseIdProvider를 지정한다.</td>
		</tr>
		<tr>
			<td>plugins</td>
			<td>Interceptor[]</td>
			<td>Interceptor를 지정한다.</td>
		</tr>
	</tbody>
</table>

##### 4.4.1. MyBatis JTA 트랜잭션 연동
<br/>
MyBatis를 JTA 트랜잭션과 연동하려면 SqlSessionFactoryBean의 transactionFactory 프로퍼티에 ManagedTransactionFactory 빈 객체를 설정하면 된다.  

```xml
<bean id="sqlSessionFactory" class="org,mybatis.spring.SqlSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
	<property name="transactionFactory">
		<bean class="org.apache.ibatis.transaction.managed.ManagedTransactionFactory"/>
	</property>
	<property name="mapperLocations">
		<list>
			<value>classpath:/mybatis/itemdao.xml</value>
			<value>classpath:/mybatis/purchaseorderdao.xml</value>
		</list>
	</property>
	<property name="typeAliasesPackage" value="net.madvirus.spring4.chap13.store.model" />
</bean>
```
