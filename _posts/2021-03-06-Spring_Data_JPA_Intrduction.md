---
title: 스프링 데이터 JPA 소개
categories:
- Spring
feature_text: |
  ## 스프링 데이터 JPA 소개
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>
DB 연동을 위해 사용되는 코드는 중복된 코드를 갖는 경우가 많다. 예를 들어, JPA를 이용해서 DAO를 구현할 때, 아래의 두 클래스는 사용하는 엔티티 클래스의 이름만 다를 뿐 나머지 구현은 완전히 동일하다.  

```java
public class JpaEmployeeRepository implements EmpolyeeRepository {
	@PersistenceContext
	private EntityManager entityManager;

	@Override
	public Employee findById(Long Id) {
		return entityManager.find(Employee.class, Id);
	}
}

public class JpaTeamRepository implements TeamRepository {
	@PersistenceContext
	private EntityManager entityManager;

	@Override
	public Team findById(Long Id) {
		return entityManager.find(Team.class, Id);
	}
}
```

스프링 데이터(Spring Data) 모듈은 이렇게 단순 반복되는 코드의 양을 줄이는 것을 목표로 하고 있다. 스프링 데이터 모듈은 구현 클래스를 만들 필요 없이 정해진 규칙에 따라 인터페이스를만들기만 하면, 런타임에 알맞은 구현 객체를 생성해주는 기능을 제공한다.(MyBatis의 매퍼 자동 생성 기능과 유사하다,) 예를 들어, 다음과 같은 인터페이스가 존재하면, 스프링 데이터 모듈은 이 인터페이스를 이용해서 JPA에 맞게 구현한 스프링 빈 객체를 런타임에 생성해준다.  

```java
public interface EmployeeRepository extends Repository<Employee, Long> {
	public Employee findOne(Long id);
}
```

흔히 보일러플레이트(boiler-plate)라고 불리는 반복되는 구조의 코르를 작성하지 않아도 되기 때문에, 스프링 데이터 모듈을 사용하면 단순 DB 연동 코드 작성 시간을 줄이고 핵심 로직을 구현하는데 더 집중할 수 있게 된다.  

스프링 데이터는 JPA, MongoDB, Neo4j, Redis, JDBC 확장 등 다양한 백엔드 연동 기술에 맞는 모듈을 제공하고 있다.  

### 리파지터리(Repository)?
<br/>
도메인 주도 설계(Domain Driven Design: DDD) 방식으로 어플리케이션을 개발할 때 사용되는 주요 모델로 엔티티, 서비스, 리파지터리 등이 존재하며, 이 중에서 리파지터리는 엔티티를 보관하는 목적으로 사용된다. 새로운 엔티티 객체를 생성하면 리파지터리에 보관하며, 리파지터리로부터 필요한 엔티티 객체를 검색하게 된다. 즉, 리파지터리는 엔티티를 보관하는 역할을 표현하는 용어로 사용된다. 일반적으로 엔티티 객체의 지속적인 보관을 위해 저장소로 데이터베이스를 사용하는데, DDD 리파지터리를 구현할 때 객체와 데이터베이스 간의 매필을 위해 ORM을 사용하곤 한다.  

스프링 데이터는 데이터를 관리하는 인터페이스의 이름으로 DAO가 아닌 리파지터리란 용어를 선택했는데, 이는 DDD의 리파지터리와 잘 들어맞는다. 실제로 DDD 방식으로 어플리케이션을 구현할 때 스프링 데이터 JPA를 이용해서 DDD의 리파지터리를 구현하는 경우가 많다.  

### 1. 스프링 데이터 JPA 시작하기
<br/>
#### 1.1. 스프링 데이터 JPA 의존 추가
<br/>
먼저 스프링 데이터 JPA를 사용하기 위해 spring-data-jpa 모듈 의존을 추가한다.

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.data</groupId>
		<artifactId>spring-data-jpa</artifactId>
		<version>1.6.0.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.hibernate</groupId>
		<artifactId>hibernate-entitymanager</artifactId>
		<version>4.3.4.Final</version>
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
		<groupId>org.springframework</groupId>
		<artifactId>spring-context</artifactId>
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

스프링 데이터 JPA는 JPA를 위한 모듈이므로 JPA 프로바이더도 함께 의존 추가를 해주어야 한다.  

#### 1.2. DB 테이블 및 초기 데이터 생성  
<br/>
#### 1.3. ORM에서 사용할 도메인 클래스
<br/>
#### 1.4. 리파지터리 인터페이스 작성하기
<br/>
스프링 데이터 JPA를 사용할 때의 핵심은 바로 인터페이스를 알맞게 작성하는 것이다. 스프링 데이터 JPA를 이용한다는 것은 스프링 데이터가 제공하는 Repository 인터페이스를 상속받은 인터페이스를 만든다는 것을 뜻한다.  

스프링 데이터 JPA 모듈은 Repository 인터페이스를 상속받은 인터페이스를 검색해서 리파지터리 구현 객체를 생성한다. Repository 인터페이스는 제네릭 타입으로서 첫 번째 타입 파라미터는 리파지터리가 다룰 엔티티 타입을 지정하고, 두 번째 타입 파라미터에는 엔티티의 식별값 타입을 지정한다. 여기서 엔티티 타입은 JPA의 &#64;Entity 애노테이션이 적용된 클래스에 해당하며, 식별값 타입은 &#64;Id 애노테이션이 적용된 프로퍼티의 타입이다.  

리파지터리 인터페이스는 스프링 데이터가 정한 규칙에 따라 메서드를 정의한다. 두 개의 메서드를 정의학고 있는데 다음과 같은 의미를 갖는다.  

+ save() : 엔티티를 저장하는 메서드이다. 파라미터로는 저장할 엔티티 객체를 전달한다. (EntityManager의 persist() 또는 merge() 메서드를 이용해서 엔티티를 저장한다.)
+ findOne() : 특정 식별값을 갖는 엔티티를 구한다. (EntityManager의 find() 메서드를 이용해서 엔티티를 구한다.)

#### 1.5. 스프링 설정하기
<br/>
리파지터리 인터페이스만 작성하면 나머지는 스프링 데이터 JPA가 알아서 처리한다. 스프링 데이터 모듈은 Repository 인터페이스를 상속 받은 인터페이스를 검색한 뒤에, 그 인터페이스에 정의된 메서드 목록을 찾는다. 그리고, 런타임에 그 메서드를 알맞게 구현한 리파지터리 객체를 생성해서 스프링 빈으로 등록한다.  

스프링 데이터 JPA 모듈이 리파지터리 인터페이스를 찾아서 알맞은 스프링 빈 객체를 생성하도록 하려면 다음처럼 &#60;jpa:repositories&#62; 태그를 추가해주면 된다.  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:jpa="http://www.springframework.org/schema/data/jpa"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx.xsd
		http://www.springframework.org/schema/data/jpa
		http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">


	<context:annotation-config />
	
	<tx:annotation-driven />
	
	<jpa:repositories base-package="net.madvirus.spring4.chap14.domain">
	</jpa:repositories>
	
	<bean id="updateEmployeeService" class="net.madvirus.spring4.chap14.application.UpdateEmployeeServiceImpl">
	</bean>

	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
		<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="jdbcUrl" value="jdbc:mysql://localhost/hrdb?characterEncoding=utf8" />
		<property name="user" value="spring4" />
		<property name="password" value="spring4" />
	</bean>
	
	<bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />
	
	<bean id="jpaVendorAdapter" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
		<property name="database" value="MYSQL" />
	</bean>
	
	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
		<property name="packagesToScan">
			<list>
				<value>net.madvirus.spring4.chap14.domain</value>
			</list>
		</property>
	</bean>
	
	<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
		<property name="entityManagerFactory" ref="entityManagerFactory" />
	</bean>
</beans>
```

jpa 네임스페이스에 대한 스키마를 설정하고, &#60;jpa:repositories&#62; 태그를 이용해서 리파지터리 인터페이스를 검색할 패키지를 지정해주기만 하면 된다. 따라서 그 하위 패키지에서 Repository 인터페이스를 상속받은 인터페이스를 찾아서 리파리터리 구현 빈 객체를 생성한다.  
리파지터리를 사용할 빈은 &#64;Autowired나 &#64;Resource와 같은 애노테이션을 이용해서 의존 자동 주입을 사용하면 된다. 이런 이유로 &#60;context:annotation-config&#62; 태그를 사용했다.  

#### 1.6. 예제 실행
<br/>
### 2. 리파지터리 인터페이스 메서드 작성 규칙
<br/>
#### 2.1. Repository 인터페이스
<br/>
스프링 데이터 JPA 모듈이 리파지터리로 사용할 인터페이스는 org.springframework.data.repository.Repository 인터페이스를 상속받아야 한다. 이 인터페이스는 다음과 같이 정의되어 있다.  

```java
public interfact Repository<T, ID extends Serializable> {
}
```

Repository 인터페이스의 타입 파리미터 T는 엔티티의 타입을 의미하며, 타입 파라미터 ID는 식별값 타입을 의미한다. 식별값으로 사용될 타입은 Serializable 인터페이스를 구현하고 있어야 한다.  

스프링 데이터 JPA가 생성하는 리파지터리 객체는 기본적으로 다음과 같이 &#64;Transactional 애노테이션을 적용한다.  

+ 조회 메서드: &#64;Transactional(readOnly = true)
+ 변경 메서드: &#64;Transactional  

즉, 리파지터리의 각 메서드는 스프링의 트랜잭션 범위 내에서 실행되며, 조회 메서드만 실행할 경우 읽기 전용 트랜잭션으로 실행한다.  

#### 2.2. 조회 메서드 규칙
<br/>
스프링 데이터 JPA 모듈을 사용할 때 가장 많이 참조하게 될 규칙이 바로 조회 메서드 작성 규칙이다. 먼저, 특정 식별값을 갖는 엔티티를 검색하는 메서드는 다음과 같이 findOne() 메서드를 사용한다.  

+ T findOne(ID primaryKey)  

T는 엔티티 타입이고, ID는 식별값 타입이다. 식별값에 해당하는 엔티티가 존재하면 해당 엔티티 객체를 리턴하고, 존재하지 않으면 null을 리턴한다.  

모든 엔티티 목록을 구하고 싶을 때에는 findAll() 메서드를 사용한다. 리턴 타입은 다음과 같이 List나 Interable 중 하나를 사용하면 된다.  

+ List&#60;T&#62; findAll()
+ Iterable&#60;T&#62; findAll()  

특정 프로퍼티 값을 이용해서 검색하고 싶다면 findBy프로퍼티이름() 형식의 메서드를 사용한다.  

findBy로 시작하는 메서드의 경우에도 리턴 타입으로 List나 Iterable을 사용할 수 있다.  

두 개 이상의 프로퍼티를 검색 조건으로 사용하고 싶다면 다음과 같이 And나 Or를 이용하면 된다.  

And나 Or 외에 크기 비교나 Null 여부, Like 쿼리를 위한 메서드 키워드를 제공하고 있다. 스프링 데이터 JPA 레퍼런스 문서에 따르면 다음 표와 같이 사용 가능한 키워드가 존재한다. 표에서 JPQL의 ?1이나 ?2는 메서드에 전달된 첫 번째 파라미터와 두 번째 파라미터 값을 의미한다.  

<table>
	<thead>
		<tr>
			<td>키워드</td>
			<td>메서드 예시</td>
			<td>JPQR 변환</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>And</td>
			<td>findByLastnameAndFirstname</td>
			<td>where x.lastnmae = ?1 and x.firstname = ?2</td>
		</tr>
		<tr>
			<td>Or</td>
			<td>findByFirstOrLastname</td>
			<td>where x.firstname = ?1 and x.lastname = ?2</td>
		</tr>
		<tr>
			<td>Is, Equals</td>
			<td>
				findByName<br/>
				findByNameIs</br/>
				findByNameEquals
			</td>
			<td>where x.name = ?1</td>
		</tr>
		<tr>
			<td>Between</td>
			<td>findByStartDateBetween</td>
			<td>where x.startDate between ?1 and ?2</td>
		</tr>
		<tr>
			<td>LessThan</td>
			<td>findByAgeLessThan</td>
			<td>where x.age &#60; ?1</td>
		</tr>
		<tr>
			<td>LessThanEqual</td>
			<td>findByAgeLessThanEqual</td>
			<td>where x.age &#60;= ?1</td>
		</tr>
		<tr>
			<td>GreaterThan</td>
			<td>findByAgeGreaterThan</td>
			<td>where x.age &#62; ?1</td>
		</tr>
		<tr>
			<td>GreaterThanEqual</td>
			<td>findByAgeGreaterThanEqual</td>
			<td>where x.age &#62;= ?1</td>
		</tr>
		<tr>
			<td>After</td>
			<td>findByStartDateAfter</td>
			<td>where x.startDate &#62; ?1</td>
		</tr>
		<tr>
			<td>Before</td>
			<td>findByStartDateBefore</td>
			<td>where x.startDate &#60; ?1</td>
		</tr>
		<tr>
			<td>isNull</td>
			<td>findByAgeIsNull</td>
			<td>where x.age is null</td>
		</tr>
		<tr>
			<td>
				IsNotNull<br/>
				NotNull
			</td>
			<td>
				findByAgeIsNotNull<br/>
				findByAgeNotNull
			</td>
			<td>where x.age not null</td>
		</tr>
		<tr>
			<td>Like</td>
			<td>findByNameLike</td>
			<td>where x.name like ?1</td>
		</tr>
		<tr>
			<td>NotLike</td>
			<td>findByNameNotLike</td>
			<td>where x.name like ?1</td>
		</tr>
		<tr>
			<td>StartingWith</td>
			<td>findByNameStartingWith</td>
			<td>
				where x.name like ?1<br/>
				(파라미터 값 뒤에 %가 붙음)
			</td>
		</tr>
		<tr>
			<td>EndingWith</td>
			<td>findByNameEndingWith</td>
			<td>
				where x.name like ?1<br/>
				(파라미터 값 앞에 %가 붙음)
			</td>
		</tr>
		<tr>
			<td>Containing</td>
			<td>findByNameContaining</td>
			<td>
				where x.name like ?1<br/>
				(파라미터 값 양 쪽에 %가 붙음)
			</td>
		</tr>
		<tr>
			<td>Not</td>
			<td>findByNameNot</td>
			<td>where x.name &#60;&#62; ?1</td>
		</tr>
		<tr>
			<td>In</td>
			<td>findByAgeIn(Collection&#60;Age&#62; ages)</td>
			<td>where x.age in ?1</td>
		</tr>
		<tr>
			<td>NotIn</td>
			<td>findByAgeNotIn(Collection&#60;Age&#62; ages)</td>
			<td>where x.age not in ?1</td>
		</tr>
		<tr>
			<td>Ture</td>
			<td>findByActiveTrue</td>
			<td>where x.active = true</td>
		</tr>
		<tr>
			<td>False</td>
			<td>findByActiveFalse</td>
			<td>where x.active = false</td>
		</tr>
		<tr>
			<td>IgnoreCase</td>
			<td>findByNameIgnoreCase</td>
			<td>
				where x.name like ?1<br/>
				(파라미터 값 양 쪽에 %가 붙음)
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

Like 키워드의 경우 '%'가 붙지 않는다.  

만약 Like를 사용하면서 전달한 파라미터 값을 포함한 경우를 찾고 싶다면, 다음과 같이 직접 '%'를 파라미터 값에 사용해야 한다.  

```java
Iterable<Employee> emps = empRepository.findByNameLike("%범%");
```

쿼리 결과로 조회되는 객체가 1개 또는 0개인 경우도 있다. 이렇게 조회 결과가 1개나 0개인 뭐리 메서드는 리턴 타입으로 Iterable 대신 엔티티 타입을 사용해도 된다.  

쿼리 메서드의 리턴 타입으로 엔티티 타입을 사용할 때 주의할 점은 만약 조회 결과 엔티티 개수가 두 개 이상이면 익셉션이 발생한다는 점이다.  

##### 2.2.1. 개수 조회 메서드
<br/>
전체 개수를 구하는 메서드를 작성하고 싶다면, 다음과 같이 메서드 이름을 count()로 지정하면 된다.

```java
public interface EmployeeRepository extends Repository<Employee, Long> {
	long count();
	...
}
```

count() 메서드를 실행하면 전체 엔티티의 개수-즉, 테이블의 전체 행 개수-를 리턴한다.  

long 타입 대신에 inrt, Long, Integer, Number 타입을 사용할 수도 있다.  

특정 조건을 충족하는 엔티티의 개수를 구하고 싶다면 다음과 같이 countBy로 시작하는 메서드를 추가하면 된다.  

```java
public long countByTeamId(Long teamId);
```

##### 2.2.2. 쿼리 메서드의 중첩 프로퍼티 접근
<br/>
중첩 프로퍼티를 쿼리 메서드에서 사용하려면 단순히 중첩 프로퍼티 이름을 연속해서 붙여주면 된다.  

```java
public Interable<Employee> findByTeamName(String teamName);
```

만약 다음과 같이 두 개의 프로퍼티가 존재한다면 어떻게 될까?

```java
@Entity
@Table(name = "EMPLOYEE")
public class Employee {
	@Column(name = "TEAMNAME")
	private String teamName;
	@ManyToOne
	private Team team;
	...
}
```

이 경우 리파지터리 인터페이스의 findByTeamName(String name) 메서드는 teamName 프로퍼티 값을 비교하게 된다. 만약 teamName 프로퍼티가 아니라 team 프로퍼티의 중첩된 프로퍼티인 name 값을 이용해서 비교하고 싶다면, 다음과 같이 밑줄을 이용해서 중첩 프로퍼티임을 구체적으로 지정해주어야 한다.  

```java
public List<Employee> findByTeam_Name(String teamName);
```

#### 2.3. 정렬과 페이징 처리
<br/>
정렬을 처리하는 방법은 크게 두 가지가 있다. 하나는 메서드 이름에 OrderBy를 이용해서 지정하는 것이다. 다음은 OrderBy 키워드의 몇 가지 사용 예를 보여주고 있다.  

```java
public List<Employee> findByStartingWithOrderByNameAsc(String name);
public List<Employee> findByTeamIdOrderByDesc(Long teamId);
public List<Employee> findByBirthYearOrderByTeamNameAscNameAsc(int year);
```

주의할 점은 OrderBy 키워드를 사용하려면 반드시 findBy와 함께 사용해야 한다는 점이다. findAllOrderByNameDesc()와 같은 메서드 이름을 사용할 경우 스프링 데이터 JPA 모듈이 리파지터리 인터페이스의 메서드 이름을 분석하는 과정에서 익셉션을 발생시킨다. findAll() 메서드에 정렬을 적용하고 싶다면 뒤에서 설명할 Sort나 Pageable을 파라미터로 전달하는 방법을 사용해야 한다.  

정렬하는 두 번째 방법은 Sort나 Pageable을 사용하는 것이다. OrderBy 키워드는 정렬 기준이 고정된 반면에, Sort나 Pageable을 사용하면 런타임에 정렬 순서를 지정할 수 있다. 따라서, 런타임에 정렬 순서를 지정하려면 Sort나 Pageable 타입의 파라미터를 사용해야 한다.  

org.springframework.data.domain.Sort 클래스는 쿼리 메서드에서 정렬 순서를 지정할 때 사용된다. 쿼리 메서드에서 정렬 순서를 지공하고 싶다면, 다음과 같이 쿼리 메서드의 마지막 파라미터로 Sort 타입 파라미터를 사용할 수 있다.  

```java
public interface EmployeeRepository extends Repository<Employee, Long> {
	public List<Employee> findAll(Sort sort);
	public List<Employee> findByTeam(Team team, Sort sort);
}
```

Sort 객체를 생성할 때에는 다음과 같이 org.springframework.data.domain.Sort.Order 클래스를 이용할 수 있다. 아래 코드에서 Direction 열거 타입과 Order 클래스는 Sort 클래스에 중첩 정의된 클래스이다.  

```java
// team.id 프로퍼티와 name 프로퍼티를 오름차순으로 정렬
// 즉, from Employee e order by e.team.id asc, e.name asc JPQL과 동일한 결과
Sort sort = new Sort(
	new Order(Direction.DESC, "team_id"),
	new Order(Direction.ASC, "name") );

List<Employee> emps = employeeRepository.findAll(sort);
```

만약 모든 프로퍼티를 오름차순으로 정렬할 목적으로 Order 객체를 생성한다면, 다음과 같이 Direction을 생략하고 파라미터 이름만 이용해서 Order 객체를 생성할 수 있다.  

```java
// team.id 프로퍼티와 name 프로퍼티를 오름차순으로 정렬
Sort sort = new Sort(new Order("team.id"), new Order("name"));
```

지정한 프로퍼티를 모두 오름차순으로 정렬한다면 Order 객체를 생성하는 대신 다음과 같이 Sort 객체를 생성할 때 오름차순으로 정렬할 프로퍼티 목록을 전달해도 된다.  

```java
Sort sort = new Sort(Direction.DESC, "team_id", "birthYear");
```

Sort 대신에 org.springframework.data.domain.Pageable 인터페이스를 사용할 수도 있다. 쿼리 메서드의 마지막 파라미터로 Pageable 타입을 사용하면, 조회 범위와 정렬 순서를 함께 지정할 수 있다. 다음은 Pageable 타입의 파라미터를 마지막 인자로 갖는 메서드의 작성 예를 보여주고 있다.  

```java
public interface EmployeeRepository extends Repository<Employee, Long> {
	...
	public List<Employee> findByBirthYearLessThan(int birthYear, Pageable pageable);
}
```

Pageable 타입의 객체를 생성할 때에는 org.springframework.data.domain.PageRequest 클래스를 사용한다. (직접 Pageable 인터페이스를 구현한 클래스를 만들 수도 있지만, PageRequest 클래스가 이미 필요한 내용을 구현하고 있다.) PageRequest 클래스의 생성자는 다음과 같다.  

+ PageRequest(int page, int size)
+ PageRequest(int page, int size, Sort sort)
+ PageRequest(int page, int size, Direction direction, String... properties)  

page 파라미터와 size 파라미터는 페이징과 관련된 값이다. page 파라미터는 0 기반의 페이지 번호를 지정하며, size는 한 페이지에 읽어올 개수를 지정한다.  

정렬을 지정하고 싶다면 Sort 파라미터를 전달받는 생성자를 사용하고, 정렬에 사용할 프로퍼티가 모두 동일한 정렬 방향을 갖는다면 Direction과 프로퍼티 목록을 전달받는 생성자를 사용한다.  

```java
Pageable pageable = new PageRequest(3, 10, new Sort("birthYear"));
List<Employee> emps = empRepo.findByBirthYearLastThan(2000, pageable);
```

Pageable 타입 파라미터를 갖는 쿼리 메서드는 리턴 타입으로 org.springframework.data.domain.Page&#60;T&#62; 타입을 사용할 수 있다. 앞서 살펴본 쿼리 메서드가 리턴 타입으로 엔티티 타입이나 Iterable 타입(또는 그 하위 타입인 List 등의 타입)을 사용했는데, 이들 타입은 값 목록만을 표현한다는 특징이 있다. 반면에 Page 타입은 값 목록뿐만 아니라 전체 페이지 개수, 전체 데이터 개수, 이전/다음 페이지를 가졌는지 여부 등의 정보도 함께 제공한다.  

다음 코드는 리턴 타입으로 Page를 사용하는 쿼리 메서드의 작성 예를 보여주고 있다.  

```java
public Page<Employee> findByTeam(Team team, Pageable pageable);
```

Page 인터페이스는 다음과 같은 메서드를 제공하고 있드며, 이들 메서드를 이용해서 페이징 처리 및 조회된 결과 데이터를 사용할 수 있다.  

+ int getTotalPages()
전체 페이지 개수를 구한다.  
+ long getTotalElements()  
전체 엘리먼트(엔티티) 개수를 구한다.  
+ int getNumber()  
현재 페이지 번호를 구한다.  
+ int getNumberOfElements()  
리턴된 엘리먼트의 개수를 구한다.  
+ int getSize()  
페이지의 기준 크기를 구한다.  
+ boolean hasContent()  
조회 결과가 존재하는지 여부를 구한다.  
+ List&#60;T&#62; getContent()  
조회된 엘리먼트 목록을 구한다.  
+ boolean isFirst()  
첫 번째 페이지인지 여부를 구한다.  
+ boolean isLast()  
마지막 페이지인지 여부를 구한다.  
+ boolean hasNext()  
다음 페이지가 존재하는지 여부를 구한다.  
+ boolean hasPrevious()  
이전 페이지가 존재하는지 여부를 구한다.  

만약 쿼리 메서드 이름에 OrderBy를 사용하고 파라미터 타입으로 Sort나 Pageable을 사용하면 어떻게 될까? 예를 들어, 다음의 쿼리 메서드를 생각해보자.  

```java
public Iterable<Employee> findByTeamIdOrderByNameDesc(Long teamId, Sort sort);
```

이 쿼리 메서드는 OrderBy를 이용해서 name 역순으로 정렬이 되도록 했고, 메서드의 마지막 파라미터로 Sort를 전달받고 있다. 이 메서드를 이용해서 다음과 같은 조회 코드를 실행할 수 있을 것이다.  

```java
Sort sort = new Sort("birthYear");
Iterable<Employee> emps = empRepository.findByTeamIdOrderByNameDesc(1L, sort);
```

위 코드를 실행하면 실제로 다음과 같은 order by 절을 이용해서 정렬 순서를 정하게 된다.
```
select * from Employee e where e.team.id = 1L order by name desc, birthYear asc;
```
이 쿼리를 보면 OrderBy 키워드로 지정한 정렬 순서가 먼저 적용되고 그 다음에 Sort나 Pageable 파라미터로 잔달받은 정렬 순서가 적요된 것을 알 수 있다.  

##### 2.3.1. Pageable과 SQL 쿼리
<br/>
Pageable을 쿼리 메서드의 파라미터로 사용할 때 리턴 타입이 Iterable이나 List와 같은 단순 목록이나 아니면 Page이냐에 따라 실행되는 쿼리가 달라진다. 먼저 두 가지 경우에 모두 지정한 범위의 데이터를 읽어오기 위한 쿼리를 실행한다. 예를 들어, 다음과 같은 코드를 실행했다고 하자.  

```java
Pageable pageable = new PageRequest(1, 4, new Sort("birthYear"));
Team team = ...; // id가 1L인 Team 객체
List<Employee> emps = employeeRepository.findByTeamId(1L, pageable);
Page<Employee> pageEmp = employeeRepository.findByTeam(team, pageable);
```

이때, 리턴 타입이 List인 findByTeamId() 메서드와 리턴 타입이 Page인 findByTeam() 메서드는 둘 다 아래의 쿼리를 실행한다. 참고로, MySQL에서 실행했기 때문에 쿼리를 보면 페이징 처리를 위해 limit 키워드가 사용된 것을 알 수 있다.  

```sql
select
	E.EPLOYEE_ID, E.HOME_ADDR1, E.HOME_ADDR2, E.HOME_ZIPCODE, E.BIRTH_YEAR, E.EMPLOYEE_NUM, E.JOINED_DATE, E.NAME, E.TEAM_ID
from EMPLOYEE E left outer join TEAM T on E.TEAM_ID=T.TEAM_ID
where T.TEAM_ID=?
order by E.BIRTH_YEAR asc limit ?, ?
```

그리고, 리턴 타입이 Page인 경우는 페이징과 관련된 정보를 생성하기 위해 다음의 쿼리를 먼저 실행한다.  

```sql
select count(E.EMPLOYEE_ID)
from EMPLOYEE E left outer join TEAM T on E.TEAM_ID=T.TEAM_ID
where T.TEAM_ID=?
```

따라서, 전체 개수나 전체 페이지 개수 등의 정보가 필요 없다면 리턴 타입으로 Page를 사용하지 않아야 불필요하게 count 쿼리를 실행하지 않는다.  

#### 2.4. 저장 메서드 규칙
<br/>
엔티티를 DB에 저장하려면 다음곽 같은 save() 메서드를 사용하면 된다.  

```java
public interface EmployeeRepository extends Repository<Employee, Long> {
	Employee save(Employee entity);
	...
}
```

save() 메서드는 저장하거나 또는 수정한다. 다음의 경우 save() 메서드는 EntityManager.persist() 메서드를 이용해서 엔티티를 DB에 저장하고, 파라미터로 전달받은 entity 객체를 리턴한다.  

+ 버전 프로퍼티가 없는 경우 : 파라미터로 전달한 엔티티 객체의 ID에 해당하는 프로퍼티 값이 null임
+ 버전 프로퍼티가 있는 경우 : 버전 프로퍼티의 값이 null임  

다음의 경우에는 EntityManager.merge() 메서드를 이용해서 기존 데이터를 변경하고, merge() 메서드의 결과를 리턴한다.  

+ 버전 프로퍼티가 없는 경우 : 파라미터로 전달한 엔티티 객체의 ID에 해당하는 프로퍼티 값이 null이 아님
+ 버전 프로퍼티가 있는 경우 : 버전 프로퍼티의 값이 null이 아님  

즉, 리파지터리 인터페이스의 save() 메서드는 실제로는 saveOrUpdate()와 같은 기능을 제공한다.  

만약 엔티티 클래스가 org.springframework.data.domain.Persistable 인터페이스를 구현하고 있다면, 이 인터페이스에 정의된 isNew() 메서드를 이용해서 새로운 객체인지 여부를 판단한다.  

#### 2.5. 삭제 메서드 규칙
<br/>
식별값을 파라미터로 전달받는 delete() 메서드는 식별값에 해당하는 데이터를 DB에서 삭제한다. 만약 id에 해당하는 데이터가 존재하지 않으면 EmptyResultDataAccessException을 발생시킨다.  

한 개 엔티티를 전달받는 delete() 메서드와 Iterable로 목록을 전달받는 delete() 메서드는 파라미터로 전달받은 엔티티를 DB에서 삭제한다.  

deleteAll() 메서드는 모든 데이터를 삭제한다. deleteAll() 메서드를 사용하면 내부적으로 delete(findAll())을 실행한다. 즉, "delete from EMPLOYEE"가 아닌 "delete from EMPLOYEE where EMPLOYEE&#95;ID = ?" 쿼리를 엔티티 개수만큼 실행하므로, 조금이라도 빠른 실행 속도를 원한다면 deleteAll() 메서드를 사용하는 대신 쿼리를 직접 실행하는 방법을 사용해야 한다.  

### 3. &#64;Query를 이용한 JPQL/네이티브 쿼리 사용
<br/>
org.springframework.data.jpa.repository.Query 애노테이션을 사용하면 조회 메서드에서 실행할 쿼리를 직접 지정할 수 있다. 다음은 &#64;Query 애노테이션을 이용한 조회 메서드의 작성 예를 보여주고 있다.  

```java
@Query("from Employee e where e.employeeNumber = ?1 or e.name like %?2%")
public Employee findByEmployeeNumberLike(String empNum, String name);

@Query("from Employee e where e.birthYear < :year order by e.birthYear")
public List<Employee> findEmployeeBornBefore(@Param("year") int year);
```

&#64;Query 애노테이션은 실행할 JPQL을 값으로 갖는다. 첫 번째 &#64;Query 애노테이션 쿼리를 보면 위치 기반 쿼리 파라미터인 ?1과 ?2가 있는데, 여기서 ?1과 ?2는 각각 첫 번째 파라미터 empName과 두 번째 파라미터 name 값을 사용한다. ?2를 보면 앞 뒤로 %가 있는데, 이는 like 검색을 위한 것이다.  

두 번째 &#64;Query의 JPQL은 ":year"를 포함하고 있다. ":이름"은 이름 기반의 네임드 파라미터로서, 네임드 파라미터의 이름과 동일한 값을 갖는 &#64;Param 애노테이션이 적용된 메서드 파라미터 값이 사용된다. 위 코드의 경우 :year 위치에 year 파라미터 값이 사용된다.  

&#64;Query 애노테이션에서 사용하는 쿼리에서 order by를 사용해서 정렬 순서를 지정할 수 있다. 또한, 다음 코드처럼 조회 메서드의 파라미터로 Sort와 Pageable을 사용해서 정렬 순서와 페이징 처리를 할 수도 있다.  

```java
@Query("from Employee e where e.birthYear < :year")
public List<Employee> findEmployeeBornBefore(@Param("year") int year, Sort sort);

@Query("from Employee e where e.birthYear < :year order by e.birthYear")
public Page<Employee> findEmployeeBornBefore(@Param("year") int year, Pageable pageable);
```

첫 번째 메서드처럼 &#64;Query의 쿼리에 order by 절이 없으면 Sort 타입 파라미터나 Pageable에 지정한 Sort를 이용해서 order by 부분을 생성한다. 반면에 두 번째 메서드처럼 &#64;Query의 쿼리에 order by가 존재하고 메서드 파라미터에 Sort가 포함되어 있다면, 쿼리의 order by 절 뒤에 Sort로 지정한 정렬 순서를 추가한다.  

자바 7 버전까지는 네임드 파라미터를 사용하려면 &#64;Param 애노테이션을 이용해서 메서드 파라미터와 매핑될 네임드 파라미터의 이름을 지정해 주어야했다. 하지만, 자바 8 버전부터는 새롭게 추가된 메서드 파라미터 이름 발견 기능을 통해 네임드 파라미터와 동일한 이름을 갖는 데서드 파라미터를 사용할 수 있다.  

#### 3.1. 수정 쿼리 실행하기
<br/>
수정 쿼리를 사용할 겨우 &#64;Modifying 애노테이션을 함께 사용해야 한다.  

```java
public interface TeamRepository extends Repository<Team, Long> {
	
	@Modifying
	@Query("update Team set t.name ?2 where t.id = ?1")
	public int updateName(Long id, String newName);

	...
}
```

수정 쿼리 메서드는 쿼리 실행 결과로 수정된 행의 개수를 리턴한다.  

수정 쿼리 메서드를 실행할 때 주의할 점이 있다. 일단 다음 코드를 보자.  

```java
@Transactional
@Override
public void updateName(Long teamId, String newName) {
	Team team = teamRepository.findOne(teamId);
	if (team == null)
		throws new TeamNotFoundException("No Team for Id[" + teamId + "]");

	// 변경 전: team.getName()은 변경 전 값

	int updated = teamRepository.updateName(teamId, newName);
	// 쿼리 실행 후: team.getName()은 여전히 이전 값

}
```

위 코드는 findOne() 메서드로 엔티티 객체를 구한다. 엔티티가 존재하면 updateName()을 이용해서 수정 쿼리를 실행한다. updateName() 쿼리를 실행하면 DB 데이터가 변경되지만, 앞서 읽어온 엔티티의 데이터는 변경되지 않는다. 이는 JPA의 영속성 컨텍스트에 변경 내역이 반영되지 않았기 때문에 발생하는 증상이다.  

만약 수정 쿼리를 실행한 후에 이미 로딩한 엔티티 객체에 변경 내역을 반영하고 싶다면, 다음과 같이 &#64;Modifying 애노테이션의 clearAutomatically 속성 값을 true로 지정해주면 된다.  

```java
@Modifying(clearAutomatically=true)
@Query("update Team t set t.name = ?2 where t.id = ?1")
public int updateName(Long id, String newName);
```

#### 3.2. 네이티브 쿼리 사용하기
<br/>
네이티브 쿼리를 실행하고 싶다면, &#64;Query 애노테이션의 값으로 네이티브 쿼리를 입력하고 nativeQuery 속성의 값을 true로 지정하면 된다.  

```java
@Query(value = "select * from TEAM where NAME like ?1%", nativeQuery = true)
List<Team> findByNameLike(String name);
```

네이티브 쿼리를 사용할 때 주의할 점은 메서드에 Sort나 Pageable을 파라미터로 추가해도 원하는 결과를 얻을 수 없다는 점이다. 네이티브 쿼리를 사용하면 정렬이나 페이징 처리와 관련된 쿼리를 각 DBMS에 맞게 생성하는 것이 어렵기 때문에, Sort나 Pageable이 제대로 반영되지 않는다.  

### 4. Specification을 이용한 검색 조건 표현
<br/>
상황에 따라 다양한 조건을 조합해서 검색 조건을 생성해야 할 때가 있다. 예를 들어, 다음과 같이 조건 조합에 따라 검색한다고 해보자.  

+ 검색어가 있다면 : 검색어와 같은 name을 갖거나 같은 employeeNumber를 갖는 Employee를 검색한다.
+ 검색 조건에 팀ID가 있다면 : 해당 팀에 속하는 Employee를 검색한다.
+ 검색어와 팀ID가 없다면 : 최근 한 달 내에 입사한 Employee를 검색한다.  

위 조건의 경우 검색어와 팀ID의 존재 여부에 따라 다음과 같은 쿼리를 실행하게 된다.  

+ from Employee e where (e.name = 검색어 or e.employeeNumber = 검색어)
+ from Employee e where e.team.id = 팀ID
+ from Employee e where (e.name = 검색어 or e.employeeNumber = 검색어) and e.team.id = 팀ID
+ from Employee e where e.joinedDate &#62; '한 달 전 날짜 값'  

검색 조건이 더 다양하다면 사용해야 할 쿼리도 더 많아지게 되고, 이는 결과적으로 리파지터리의 쿼리 메서드를 증가시키는 상황을 만들게 된다. 이런 문제를 해소하기 위해 다음 코드처럼 JPA의 Criteria API를 사용할 수 있다.  

```java
public class JpaEmployeeListService implements EmployeeListService {

	@PersistenceUnit
	private EntityManagerFactory entityManagerFactory;
	private EmployeeRepository employeeRepository;

	@Transactional
	@Override
	public List<Employee> getEmployee(String keyword, Long teamId) {
		CriteriaBuilder cb = entityManagerFactory.getCriteriaBuilder();

		CriteriaQuery<Employee> query = cb.createQuery(Employee.class);
		Root<Employee> employee = query.from(Employee.class);
		query.select(employee);

		if (hasValue(keyword) || hasValue(teamId)) {
			if (hasValue(keyword) && !hasValue(teamId)) {
				query.where(cb.or(cb.equal(employee.get("name"), keyword),
					cb.equal(employee.get("employeeNumber"), keyword)));
			} else if (!hasValue(keyword) && hasValue(teamId)) {
				query.where(cb.equal(employee.get("team").get("id"), teamId));
			} else {
				query.where(cb.and(cb.or(cb.equal(employee.get("name"), keyword),
							cb.equal(employee.get("employeeNumber"), keyword)),
						cb.equal(employee.get("team").get("id"), teamId)));
			}
		} else {
			Calendar cal = Calendar.getInstance();
			cal.add(Calendar.DATE, -30);
			query.where(cb.greaterThan(employee.<Date> get("joinedDate"), cal.getTime()));
		}
		// 실제로 스프링 데이터 JPA는 CriteriaQuery 타입 파라미터를 지원하지 않음
		return employeeRepository.findAll(query);
	}
	...
}
```

위 코드처럼 CriteriaBuilder를 이용해서 조건에 맞는 검색 조건을 생성하는 CriteriaQuery 객체를 생성할 수 있지만, 위 코드는 다음과 같은 단점을 갖고 있다.  

+ EmployeeListService는 DB에 대한 직접 접근이 필요 없음에도 불구하고 Criteria API를 사용하기 위해 DB와 관련된 EntityManagerFactory를 필드로 참조해야 한다.
+ 스프링 데이터 JPA는 리파지터리 메서드의 파라미터로 CriteriaQuery 타입을 지원하지 않는다.  

스프링 데이터 JPA는 조회 메서드에서 CriteriaQuery를 직접 지원하지 않는 대신, 좀 더 표현력이 좋은 Specification 타입을 도입했다. Specification 타입을 사용하면, Criteria API와 같은 검색 조건 조합을 만들 수 있으면서도 검색 조건을 생성하는 코드에서 EntityManagerFactory, CriteriaBuilder 등 JPA 관련 코드를 직접 사용하지 않아도 되는 장점이 있다.  

Specification을 이용해서 검색 조건을 지정하려면 다음과 같은 작업을 하면 된다.  

+ Specification을 입력 받도록 Repository 인터페이스를 정의하기
+ 검색 조건을 모아 놓은 클래스 만들기
+ 검색 조건을 조합한 Specification 인스턴스를 이용해서 검색하기  

#### 4.1. 리파지터리 인터페이스에 Specification 타입 파라미터 추가하기
<br/>
리파지터리 인터페이스에 Specification 타입의 파라미터를 추가하는 것은 어렵지 않다. 다음과 같이 org.springframework.data.jpa.domain.Specification&#60;엔티티타입&#62; 타입의 파라미터를 추가해주기만 하면 된다.  

```java
public interface EmployeeRepository extends Repository<Employee, Long> {
	public List<Employee> findAll(Specification<Employee> spec);
	...
}
```

다른 조회 메서드와 마찬가지로 Specification 타입 뒤에 정렬이나 페이징 처리를 위한 Sort나 Pageable 파라미터를 추가할 수 있으며, Pageable 파라미터를 가질 경우 타입으로 Page를 사용할 수 있다.  

#### 4.2. Specification을 생성해주는 클래스 만들기
<br/>
앞서 추가한 메서드를 사용하려면, 알맞은 Specification 객체를 생성해주면 된다. Specification은 검색 조건을 표현하는 인터페이스로서 다음과 같이 정의되어 있다.  

```java
public interface Specification<T> {
	Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);
}
```

Specification 구현 클래스는 toProdicate() 메서드에서 검색 조건에 해당하는 Predicate 객체를 생성해주어야 한다. 예를 들어, Employee 엔티티의 name이 특정 값과 같은지 확인하는 조건을 나타내는 Specification 객체는 다음과 같이 생성할 수 있다.  

```java
final String name = ...;
Specification<Employee> spec = new Specification<Employee>() {
	@Override
	public Predicate toPredicate(Root<Employee> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
		return cb.equal(root.get("name"), name);
	}
};
List<Employee> empList = employeeRepository.findAll(sepc);
```

모든 Specification 객체를 위 코드처럼 임의 객체를 이용해서 생성할 수 있지만, 그것보다는 엔티티별로 알맞은 Specification 객체를 생성해주는 클래스를 만들어서 사용하는 것이 코드 가독성과 관리면에서 좋다. 예를 들어, 다음과 같이 Employee 엔티티 타입을 위한 검색 조건을 생성해주는 클래스를 만들 수 있을 것이다.  

```java
public class EmployeeSpec {

	public static Specification<Employee> nameEq(final String name) {
		return new Specification<Employee>() {
			@Override
			public Predicate toPredicate(Root<Employee> root, CriteraiQuery<?> query, CriteriaBuilder cb) {
				return cb.equal(root.get("name"), name);
			}
		};
	}

	public static Specification<Employee> employeeNumberEq(final String num) {
		return new Specification<Employee>() {
			@Override
			public Predicate toPredicate(Root<Employee> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
				return cb.equal(root.get("employeeNumber"), num);
			}
		};
	}
}
```
##### 4.2.1. JPA 메타 모델 클래스
<br/>
앞서 코드를 보면, 조건을 생성할 때 다음곽 같이 엔티티의 프로퍼티 이름을 문자열로 지정하고 있다.  

그런데 프로퍼티 이름을 문자열로 입력하면 오타와 같은 실수를 하기 쉽다. 이런 단순 실수를 줄이기 위한 방법이 있는데, 그것은 바로 JPA이 메타 모델 클래스를 사용하는 것이다. 메타 모델 클래스는 다음과 같이 생겼다.  

```java
@StaticMetamodel(Employee.class)
public class Employee_ {
	public static volatile SingularAttribute<Employee, Long> id;
	public static volatile SingularAttribute<Employee, String> employeeNumber;
	public static volatile SingularAttribute<Employee, String> name;
	public static volatile SingularAttribute<Employee, Address> address;
	public static volatile SingularAttribute<Employee, Integer> birthYear;
	public static volatile SingularAttribute<Employee, Team> team;
	public static volatile SingularAttribute<Employee, Date> joinedDate;

}
```

메타 모델 클래스의 이름은 모델 클래스 이름 뒤에 밑줄('&#95;')을 붙인 것을 사용한다. 위 코드는 Employee 클래스에 대한 메타 모델 클래스가 된다. 메타 모델 클래스는 정적 필드를 이용해서 실제 모델 클래스가 된다. 메타 모델 클래스는 정적 필드를 이용해서 실제 모델 클래스에 대한 프로퍼티 정보를 기술한다. 예를 들어, 위 코드에서 id 정적 필드는 같은 이름을 갖는 Employee의 id 프로퍼티에 대한 정보를 기술한다.   

이렇게 모델에 대한 메타 모델 클래스를 작성하면, 검색 조건을 생성할 때 문자열 대신 메타 모델 클래스를 이용해서 프로퍼티를 지정할 수 있다.  

```java
public static Specification<Employee> nameEq(final String name) {
	return new Specification<Employee>() {
		@Override
		public Predicate toPredicate(Root<Employee> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
			return cb.equal(root.get(Employee_.name), name);
		}
	};
}
```

위 코드처럼 메타 모델 클래스를 사용하면, 오타를 사전에 알 수 있고(컴파일 에러) 이클리스와 같은 개발환경에서는 코드 자동 완성 기능을 이용해서 빠르게 코드를 완성할 수 있게 된다.  

메타 모델 클래스의 코드를 직접 장성할 수 있지만, 모델 코드로부터 자동 생성하는 기능이 있기 때문에 그 기능을 사용하면 된다.  

##### 4.2.1. JPA 메타 모델 클래스
<br/>
주요 검색 조건별로 Specification 객체를 생성해주는 클래스를 만들면, 다음과 같이 간결한 코드를 이용해서 EmployeeRepository에 전달할 Sepcification에 전달할 Specifiaction 객체를 생성할 수 있다.  

```java
List<Employee> empList = employeeRepostory.findAll(EmployeeSpec.nameEq(name));
```

복합적인 검색 조건을 사용해야 한다면, org.springframework.data.jpa.domain.Specification 클래스를 이용해서 각 Specification을 AND와 OR로 조합할 수 있다. 다음은 Specification을 이용해서 두 Specification을 AND로 조합하는 코드의 예를 보여주고 있다.  

```java
// spec1과 spec2가 있다고 가정
Specification<Employee> specs = Specifications.where(spec1);
Specification<Employee> andSpecs = specs.and(spec2); // (spec1 and spec2) 조건 생성
List<Employee> empList = employeeRepository.findAll(andSpecs);
```

Specifications.where() 메서드는 Specification을 파라미터로 전달받고 검색 조건을 조합할 수 있는 Specifications 객체를 리턴한다. Specifications 클래스의 and() 메서드는 검색 조건을 AND로 조합한 새로운 Specifications 객체를 리턴한다. 예를 들어, 위 코드의 경우 specs.and(spec2) 코드는 spec1과 spec2를 AND 조합한 Specifications를 생성한다. Specifications 클래스는 Specification 인터페이스를 상속받고 있기 때문에, 위 코드처럼 Specifications를 검색 조건으로 전달할 수 있다. OR 조합을 하고 싶다면 and() 메서드 대신에 or() 메서드를 사용하면 된다.  

and() 메서드와 or() 메서드의 파라미터는 가변인자이므로 다음과 같이 2개 이상의 조건을 조합할 있다.  

```java
Specifications<Employee> specs = Specifications.where(spec1);
Specifications<Employee> andSpecs = specs.and(spec2, spec3);
```

정적 메서드인 where() 메서드와 조합을 위한 and()/or() 메서드는 모두 Specifications 객체를 리턴하므로, 다음과 같이 메서드를 연결하면 좀 더 간결하게 조건을 표현할 수 있다.  

```java
Specifications<Employee> specs = Specifications.where(spec1).or(spec2, spec3);
```

검색 조건을 모아 놓은 EmployeeSpec 클래스와 Specifications의 조합 기능을 이용하면 다음과 같이 위 검색 조건을 위한 조회 기능을 구현할 수 있을 것이다.  

```java
public class SpecEmployeeListService implements EmployeeListService {
	private EmployeeRepository employeeRepository;

	@Transactional
	@OVerride
	public List<Employee> getEmployee(String keyword, Long teamId) {
		if (hasValue(keyword) || hasValue(teamId)) {
			if (hasValue(keyword) && !hasValue(teamId)) {
				return employeeRepository.findAll(where(nameEq(keyword)).or(employeeNumberEq(keyword)));
			} else if (!hasValue(keyword) && hasValue(teamId)) {
				return employeeRepository.findAll(spec1.and(teamIdEq(teamId)));
			}
		} else {
			Calendar cal = Calendar.getInstance();
			cal.add(Calendar.DATE, -30);
			return employeeRepository.findAll(joinedDateGt(cal.getTime()));
		}
	}

	private boolean hasValue(Object value) {
		return value != null;
	}

	...

}
```

위 코드를 보면 EmployeeSpec의 정적 멤버와 Specifications.where 메서드를 정적 임포트했다. 이렇게 하면 EmployeeSpec.nameEq() 대신에 nameEq() 메서드를 그리고 Specifications.where() 대신에 where() 메서드를 바로 사용할 수 있으므로, 검색 조건 생성 코드의 가독성이 향상된다. 또한, 위 코드를 보면 joinedDateGt()나 nameEq()처럼 Criteria API를 직접 사용하는 경우와 비교해서 검색 조건의 의미를 더 잘 드러내는 것을 알 수 있다.  

### 5. 기본 제공 인터페이스
<br/>
인터페이스만 작성하면 런타임에 리파지터리 구현 객체를 만들어주기 때문에, 실제 작성할 코드 양이 상당히 줄어들게 된다. 그런데, 이 인터페이스를 작성하다 보면 역시나 save(), findOne()과 같은 메서드를 여러 리파지터리에서 반복해서 작성하게 된다.  

스프링 데이터 JPA는 이런 기본 메서드를 각 리파지터리마다 작성해야 하는 번거로움을 없애기 위해, 자주 사용되는 기본 메서드를 정의한 인터페이스를 이미 제공하고 있다.  

리파지터리로 사용할 인터페이스는 필요한 메서드를 정의한 인터페이스를 상속받기만 하면 된다. 예를 들어, CrudRepository 인터페이스는 findOne()과 save()를 비롯해 CRUD에 필요한 기본적인 메서드를 포함하고 있기 때문에, 기본 CRUD 기능 외에 자신만의 추가적인 조회 메서드가 필요하다면 Repository 인터페이스 대신 CrudRepository 인터페이스를 상속받은 리파지터리를 만들면 된다.  

```java
public interface EmployeeRepository extends CrudRepository<Employee, Id> {
	// findOne(), save() 등의 메서드는 이미 CrudRepository에 정의
	public List<Employee> findByJoinedDateBetween(Date from, Date to);
}
```

만약 인터페이스에 정의된 모든 메서드가 필요하다면 JpaRepository와 JpaSpecificationExecutor 인터페이스를 상속받으면 된다.  

JpaRepository 인터페이스와 JpaSpecificationExecutor 인터페이스를 상속받으면, 기본적인 모든 메서드를 상속받기 때문에 추가할 메서드는 몇 개 되지 않는다. 이런 이유로 코딩의 편리함을 위해 이 두 인터페이스를 모두 상속받고 시작할 때도 있다. 반면에, 일부 개발자는 필요한 메서드만 추가하는 것을 선호하는데, 이런 성향의 개발자는 이들 인터페이스를 상속받아 필요하지도 않은 메서드를 포함시키기 보다는 필요한 메서드만 직접 추가한다.  

#### 5.1. CrudRepository 인터페이스의 메서드
<br/>
org.springframework.data.repository.CrudRepository 인터페이스가 제공하는 메서드는 다음과 같다.

```java
@NoRepositoryBean
public interface CrudRepository<T, ID extends Serializable> extends Repository<T, ID> {
	<S extends T> S save(S entity);
	<S extends T> Iterable<S> save(Iterable<S> entities);
	T findOne(ID id);
	boolean exists(ID id);
	Iterable<T> findAll();
	Iterable<T> findAll(Iterable<ID> ids);
	long count();
	void delete(ID id);
	void delete(T entity);
	void delete(Iterable<? extends T> entities);
	void deleteAll();
}
```

#### 5.2. PagingAndSortingRepository 인터페이스의 메서드
<br/>
org.springframework.data.repository.PagingAndSortingRepository 인터페이스가 정의한 메서드는 다음과 같다. 이름에서 알 수 있듯이 정렬과 페이징에 대한 메서드가 추가되어 있다.  

```java
public interface PagingAndSortingRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {
	Iterable<T> findAll(Sort sort);
	Page<T> findAll(Pageable pageable);
}
```

#### 5.3. JpaRepository 인터페이스의 메서드
<br/>
org.springframework.data.jpa.repository.JpaRepository 인터페이스가 제공하는 메서드는 다음과 같으며, JPA 영속성 컨텍스트를 플러시하는 기능과 리턴 타입이 List인 조회 메서드 등이 포함되어 있다.  

```java
public interface JpaRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID> {
	List<T> findAll();
	List<T> findAll(Sort sort);
	List<T> findAll(Iterable<ID> ids);
	<S extends T> List<S> save(Iterable<S> entities);
	void flush();
	<S extends T> S saveAndFlush(S entity);
	void deleteInBatch(Iterable<T> entities);
	void deleteAllInBatch();
	T getOne(ID id);
}
```

메서드 목록에서 flush() 메서드는 아직 DB에 반영되지 않은 내용을 DB에 반영시킨다. saveAndFlush() 메서드는 save()를 수행한 뒤에 flush()를 한다.  

deleteInBatch() 메서드는 파라미터로 전달받은 엔티티 목록의 식별값을 이용해서 다음의 쿼리를 실행한다. (JPA Query의 executeUpdate() 메서드를 이용해서 실행한다.)

```sql
delete from 엔티티타입 x where x = ?1 or x = ?2 or ...
```

deleteAllInBatch() 메서드는 다음 쿼리를 이용해서 전체 삭제 처리를 한다.  

```sql
delete from 엔티티타입 x
```

getOne() 메서드는 EntityManager의 getReference() 메서드를 이용해서 엔티티에 대한 레퍼런스를 구한다. 참고로 EntityManager.getReference()가 리턴한 레퍼런스 객체는 실제 엔티티 객체가 아닌 프록시 객체다. 이 프록시 객체는 최초로 데이터에 접근할 때 DB에서 데이터를 읽어오는데 만약 DB에 식별값에 해당하는 데이터가 존재하지 않으면 EntityNotFoundException을 발생시킨다.(스프링은 EntityNotFoundException을 다시 스프링에 맞는 ObjectRetrievalFailureException으로 변환해서 발생한다.)  

#### 5.4. JpaSpecificationExecutor 인터페이스의 메서드
<br/>
org.springframework.data.jpa.repository.JpaSpecificationExecutor 인터페이스는 Specification 타입을 파라미터로 갖는 메서드를 정의하고 있다. 정의된 메서드는 다음과 같다.  

```java
public interface JpaSpecificationExecutor<T> {
	T findOne(Specification<T> spec);
	List<T> findAll(Specification<T> spec);
	Page<T> findAll(Specification<T> spec, Pageable pageable);
	List<T> findAll(Specification<T> spec, Sort sort);
	long count(Specification<T> spec);
}
```

#### 5.5. 공통 인터페이스 만들기
<br/>
많은 엔티티 클래스가 PK에 해당하는 식별값 프로퍼티 외에 추가로 고유한 값을 갖는 name 프로퍼티를 갖는다고 해보자. 이 경우 각 엔티티에 해당하는 리파지터리 인터페이스는 다음과 같이 findByName(String name) 메서드를 정의할 것이다.  

```java
// findByName 메서드 중복 입력 발생
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
	Employee findByName(String name);
	...
}

public interface TeamRepository extends JpaRepository<Team, Long> {
	Team findByName(String name);
	...
}
```

이렇게 중복해서 출현하는 메서드가 있다면, 이 중복 메서드를 정의한 인터페이스를 정의하는 방법으로 메서드 중복 입력을 줄일 수 있다. 예를 들어, 다음과 같이 findByName() 메서드를 정의한 인터페이스를 만들 수 있다.  

```java
public interface NameFindableRepository<T> {
	T findByName(String name);
}
```

이제 findByName() 메서드가 필요한 리파지터리 인터페이스는 이 메서드를 직접 추가하는 대신 NameFindableRepository를 상속받으면 된다.  

### 6. 커스텀 구현 추가하기
<br/>
다음과 같은 Option 클래스가 있다고 해보자.

```java
public class Option<T> {

	private T value;

	public Option(T value) {
		this.value = value;
	}

	public boolean hasValue() {
		return value != null;
	}

	public T get() {
		if (value == null) throw new IllegalStateException("no value");
		return value;
	}
}
```

Option 타입을 리턴 값으로 사용하면 다음과 같이 null 체크 대신에 hasValue() 메서드를 이용해서 값이 존재하는지 여부를 확인하도록 바꿀 수 있다.  

```java
Option<Employee> empOp = employeeRepo.getOptionEmployee(someId);
if (empOp.hasValue()) {
	Employee emp = empOp.getValue();
	...
}
```

null을 리턴하는 대신 Option을 리턴하면 값이 존재하지 않을 수도 있다는 것을 명시적으로 표현할 수 있기 때문에, null 검사를 안 해서 NullPointerException이 발생하는 실수를 줄일 수 있다.  

그런데, 스프링 데이터 JPA는 리턴 타입으로 Option 타입을 지원하지 않기 때문에, 다음과 같은 메서드를 추가할 수 없다.  

```java
public interface EmployeeRepository extends Repository<Employee, Long> {
	// 리파지터리 인터페이스를 분석하는 과정에서 익셉션 발생!
	public Option<Employee> getOptionEmployee(Long id);
}
```

이렇게 스프링이 지원하는 규칙에서 벗어난 메서드를 추가하고 싶다면, 직접 커스텀 구현을 만들어야 한다. 커스텀 구현을 추가하는 방법은 두 가지가 있다. 첫 번째는 단일 리파지터리를 위한 커스텀 구현 추가 방법이고 다른 하나는 모든 리파지터리를 위한 커스텀 구현 추가 방법이다.  

#### 6.1. 단일 리파지터리를 위한 구현 클래스 등록
<br/>
단일 리파지터리를 위한 커스텀 메서드 구현을 등록하려면 다음과 같이 하면 된다.  

(1) 커스텀 메서드를 정의한 인터페이스를 정의한다.  
(2) 리파지터리로 사용할 인터페이스가 (1)에서 만든 커스텀 인터페이스를 상속받도록 한다. 
(3) (1)에서 정의한 커스텀 인터페이스를 구현한 커스텀 구현 클래스를 작성한다.  
(4) (선택) (3)에서 구현한 커스텀 클래스를 스프링 빈으로 등록한다.  
&nbsp;&nbsp;&nbsp;&nbsp;A. 리파지터리 인터페이스 이름 뒤에 지정한 접미사(Impl)을 붙인 이름을 커스텀 클래스의 이름으로 사용했다면 스프링 빈으로 등록하지 않아도 된다.  

먼저 할 일은 커스텀 메서드를 정의한 인터페이스를 작성하는 것이다. 다음은 커스텀 인터페이스 작성 예이다.  

```java
public interface EmployeeCustomRepository {
	public Option<Employee> getOptionEmployee(Long id);
}
```

커스텀 인터페이스를 작성했다면, 리파지터리로 사용할 인터페이스가 커스텀 인터페이스를 상속하도록 한다.  

```java
public interface EmployeeRepository extends EmployeeCustomRepository, Repository<Employee, Long> {
	...
}
```

절반은 끝났다. 이제 나머지 절반은 커스텀 인터페이스를 구현한 클래스를 작성하는 것이다. 앞서 만든 커스텀 인터페이스에 대한 구현 클래스를 다음 코드처럼 구현해보았다.  

```java
public class EmployeeRepositoryImpl implements EmployeeCustomRepository {
	@PersistenceContext
	private EntityManager entityManager;

	@Override
	public Option<Employee> getOptionEmployee(Long id) {
		Employee emp = entityManager.find(Employee.class, id);
		return Option.value(emp);
	}
}
```

커스텀 구현 클래스를 스프링 빈으로 등록하면 준비는 끝난다. 스프링 빈으로 등록하는 방법에는 두 가지가 있다. 먼저 구현 클래스 이름이 다음 형식을 따르면 스프링 데이터 JPA가 자동으로 검색해서 빈으로 등록한다.  

+ 인터페이스 이름 + 접미사(Impl)  

예를 들어, 앞서 리파지터리 인터페이스 이름은 'EmployeeRepository'인데, 이 경우 스프링은 이름이 EmployeeRepositoryImpl인 클래스를 찾아서 커스텀 구현체로 사용한다. 단, 이 클래스는 스프링 데이터 JPA가 스캔하는 패키지에 위치해야 한다.  

접미사를 붙이는 방법을 사용하면 커스텀 구현체가 스캔을 통해서 등록된다. 따라서, 커스텀 구현 클래스에서 다른 빈 객체를 내부에서 사용해야 한다면, &#64;Autowired나 &#64;Resource 등의 애노테이션을 이용해서 의존 자동 주입을 해야 한다.  

기본 접미사는 Impl인데, 접미사를 바꾸고 싶다면 다음과 같은 설정을 사용한다.  

```xml
<jpa:repositories base-package="net.madvirus.spring4.chap14.domain" repository-impl-postfix="CustomImpl">
</jpa:repositories>
```

```java
@EnableJpaRepositories(basePackages = "net.madvirus.spring4.chap14.domain", repositoryImplementationPostfix = "CustomImpl")
public class JPA설정클래스명 ... {
	...
}
```

남은 건 리파지터리의 커스텀 메서드를 사용하는 것뿐이다. 스프링 데이터 JPA는 리파지터리의 커스텀 메서드를 호출하면, 앞서 구현한 커스텀 구현 클래스를 이용해서 기능을 제공하게 된다.  

만약 정해진 접미사를 사용하지 않았거나 복잡한 설정 때문에 자동 스캔을 사용할 수 없다면, 직접 커스텀 구현 객체를 스프링 빈으로 등록해도 된다. 이때 빈의 식별 값은 '리파지터리 타입 이름 + 접미사'의 형식을 가져야 한다.  

#### 6.2. 전체 리파지터리를 위한 메서드 구현 등록
<br/>
개별 리파지터리가 아닌 전체 리파지터리에 커스텀 기능을 추가할 수도 있다. 이를 하기에 앞서 실제로 스프링 데이터 JPA가 사용하는 구현 클래스를 잠깐 살펴보자. 스프링 데이터 JPA는 리파지터리 인터페이스에 정의된 save(), findOne() 등의 구현을 제공하는 SimpleJpaRepository 클래스를 제공하고 있다. 이 클래스는 JpaRepository 인터페이스와 JpaSpecificationExecutor 인터페이스를 상속받고 있으며, 상위 인터페이스에 정의된 모든 메서드의 구현을 제공하고 있다.  

스프링 데이터 JPA는 런타임에 리파지터리 구현 객체를 생성할 때, SimpleJpaRepository 객체의 메서드를 구현으로 사용한다. 예를 들어, EmployeeRepository 인터페이스에 정의된 findOne() 메서드를 실행하면 실제로 SimpleJpaRepository 객체의 findOne() 메서드가 실행된다. 즉, 런타임에 리파지터리에서 사용할 구현체로 SimpleJpaRepository 클래스를 사용한다.  

런타임에 사용할 리파지터리 구현체를 생성해주는 것이 바로 JpaRepositoryFactory 클래스이고, 이 클래스의 객체를 생성할 때 사용하는 것이 JpaRepositoryFactoryBean 클래스다. 따라서, 전체 리파지터리에서 사용할 수 있는 커스텀 기능을 추가하려면 커스텀 리파지터리 인터페이스와 각 구현 클래스를 추가해주면 된다.  

가장 먼저 기능을 정의한 인터페이스를 작성한다. 이 인터페이스는 JpaRepository 인터페이스를 상속받은 뒤, 추가할 메서드를 정의한다. 다음은 작성 예이다.  

```java
public interface CustomRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
	// 모든 리파지터리 대상으로 추가 가능한 메서드 정의
	public Option<T> getOption(ID id);
}
```

다음으로 추가할 기능의 구현을 제공할 클래스를 작성한다. 이 클래스는 SimpleJpaRepository 클래스를 상속받고, 앞서 정의한 인터페이스를 구현한다. 다음은 CustomRepository 인터페이스를 위한 구현 클래스의 작성 예이다.  

```java
public class CustomJpaRepository<T, ID extends Serializable> extends SimpleJpaRepository<T, ID> implements CustomRepository<T, ID> {
	
	// 기능 구현에 필요한 EntityManager 필드에 보관
	private EntityManager entityManager;

	public CustomJpaRepository(Class<T> domainClass, EntityManager em) {
		super(domainClass, em);
		this.entityManager = em;
	}

	public CustomJpaRepository(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
		super(entityInformation, entityManager);
		this.entityManager = entityManager;
	}

	// 커스텀 기능 구현
	@Override
	public Option<T> getOption(ID id) {
		return Option.value(entityManager.find(getDomainClass(), id));
	}
}
```

커스텀 기능 구현 클래스를 만들었으니, 이제 이 구현 클래스를 이용해서 객체를 생성하는 팩토리를 만들 차례다. 이 팩토리 클래스는 JpaRepositoryFactory 클래스를 상속받은 뒤 getTargetRepository() 메서드와 getRepositoryBaseClass()를 재정의하면 된다. 구현 예는 다음과 같다.  

```java
public class CustomRepositoryFactory extends JpaRepositoryFactory {

	private EntityManager entityManager;

	public CustomRepositoryFactory(EntityManager entityManager) {
		super(entityManager);
		this.entityManager = entityManager;
	}

	@SuppressWarnings({ "rawtypes", "unchecked" })
	@Override
	protected Object getTargetRepository(ReposiotryMetadata metadata) {
		return new CustomJpaRepository(metadata.getDomainType(), entityManager);
	}

	@Override
	protected Class<?> getRepositoryBaseClass(RepositoryMetadata metadata) {
		return CustomJpaRepository.class;
	}
}
```

위 코드에서 두 메서드는 앞서 구현한 CustomJpaRepository를 사용해서 재정의했다.  

이제 마지막으로 팩토리 생성을 위한 FactoryBean 클래스를 작성할 차례이다. JpaRepositoryFactoryBean 클래스를 상속받은 뒤 앞서 작성한 팩토리 객체를 리턴하도록 createRepositoryFactory() 메서드를 재정의해주면 된다.  

```java
public class CustomRepositoryFactoryBean <T extends Repository<S, ID>, S, ID extends Serializable> extends JpaRepositoryFactoryBean<T, S, ID> {

	@Override
	protected RepositoryFactorySupport createRepositoryFactory(EntityManager entityManager) {
		return new CustomRepositoryFactory(entityManager);
	}

}
```

이제 스프링 데이터 JPA 모듈이 커스텀 구현을 사용하도록 만들 차례다. 스프링이 커스텀 구현 클래스를 사용하도록 하려면, 다음과 같이 JPA 설정에서 앞서 작성한 CustomRepositoryFactoryBean 클래스를 팩토리 클래스로 사용하도록 지정하면 된다.  

```xml
<jpa:repositories base-package="net.madvirus.spring4.chap14.domain" factory-class="net.madvirus.spring4.chap14.common.CustomRepositoryFactoryBean" />
```

```java
@EnableJpaRepositories(basePackages = "net.madvirus.spring4.chap14.domain", repositoryFactoryBeanClass = CustomRepositoryFactoryBean.class)
public class JPA설정클래스명 {
	...
}
```

모든 설정이 끝났다. 남은 건 리파지터리 인터페이스에서 커스텀 메서드를 사용하는 것이다.
