---
title: 스프링의 트랜잭션 관리
categories:
- Spring
feature_text: |
  ## 스프링의 트랜잭션 관리
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

NoSQL과 빅데이터가 급부상하면서 RDBMS가 제공하는 엄격한 트랜잭션보다는 대용량 데이터 처리를 위한 느슨한 방식의 무결성 처리 기법을 적용하는 곳이 증가하고 있다. 하지만, 그럼에도 불구하고 전통적인 RDBMS가 제공하는 트랜잭션은 중요하다. 예를 들어, 결제와 동시에 좌석을 할당하는 시스템을 생각해보자. 만약 이 시스템에서 결제는 이뤄진 상태에서 좌석 할당에는 실패하는 상황이 발생하면 당연히 결제에 성공하면 안 된다. 이런 경우 결제와 좌석 할당은 모두 실패로 처리되어야 한다.  

트랜잭션은 이렇게 성공적으로 처리되거나 또는 하나라도 실패하면 완전히 실패 처리를 해야 하는 경우에 사용된다. 스프링은 데이터베이스 연동분만 아니라 트랜잭션 관리 기능을 지원하고 있기 때문에, 스프링을 사용하면 간단한 설정만으로 단일 자원과 다중 자원에 대한 트랜잭션을 처리할 수 있다.  

### 1. 트랜잭션이란
<br/>
인터넷 서점 사이트에서 도서를 구매할 경우 다음과 같은 순서로 작업이 진행될 것이다.  

(1) 결제를 수행한다.  
(2) 결제 내역을 저장한다.  
(3) 구매 내역을 저장한다.  

위의 과정은 도서 구매 시 반드시 모두 성공적으로 이루어져야 한다. 하나라도 실패한 경우 반드시 모든 과정이 취소되어야 한다. 예를 들어, 결제 내역 저장까지는 성공했는데, 구매 내역을 저장하는 과정이 실패했다고 하자. 이때, 전 과정이 취소되지 않는다면 구매자는 결제만 하고 구매는 하지 않은 것처럼 될 것이다.  

트랜잭션은 여러 과정을 하나의 행위로 묶을 때 사용된다. 트랜잭션은 트랜잭션 범위 내에 있는 처리 과정 중 한 가지라도 실패할 경우 전체 과정을 취소시킴으로써 데이터의 무결성을 보장한다.  

#### 1.1. ACID
<br/>
트랜잭션을 설명할 때 보통 네 가지 특징인 ACID를 이용한다. ACID는 다음과 같다.  

+ 원자성(Atomicity)  
트랜잭션은 한 개 이상의 동작을 논리적으로 한 개의 작업 단위(unit of work)로 묶는다. 원자성은 트랜잭션 범위에 있는 모든 동작이 모두 실행되거나 또는 모두 실행이 취됨을 보장한다. 모든 동작이 성공적으로 실행되면 트랜잭션은 성공한다. 만약 하나라도 실패하면 트랜잭션은 실패하고 모든 과정을 롤백한다.  

+ 일관성(Consistency)  
트랜잭션이 종료되면, 시스템은 비즈니스에서 기대하는 상태가 된다. 예를 들어, 서적 구매 트랜잭션이 성공적으로 실행되면 결제 내역, 구매 내역, 잔고 정보가 비즈니스에 맞게 저장되고 변경된다.  

+ 고립성(isolation)  
트랜잭션은 다른 트랜잭션과 독립적으로 실행되어야 하며, 서로 다른 트랜잭션이 동일한 데이터에 동시에 접근할 경우 알맞게 동시 접근을 제어해야 한다. (동시 접근 제어는 설정한 격리 레벨에 따라 달라진다.)  

+ 지속성(Durability)  
트랜잭션이 완료되면, 그 결과는 지속적으로 유지되어야 한다. 현재의 어플리케이션이 변경되거나 없어지더라도 데이터는 유지된다. 일반적으로 물리적인 저장소를 통해서 트랜잭션 결과가 저장된다.  

예를 들어, 도서 구매의 경우 결제 처리, 구매 내역 처리 등이 하나의 작업으로 처리되어야 한다. 만약, 이 중 하나라도 실패하면 전체 구매 과정이 취소된다. 트랜잭션이 성공적으로 실행되면 결제 정보와 구매 내역 정보가 반드시 시스템에 기록되어 이후 사용할 수 있어야 한다. 동일한 서적에 대한 구매 트랜잭션이 동시에 한 개 이상 실행될 경우, 재고 정보가 알맞게 처리되어야 하므로 각 트랜잭션이 알맞은 순서로 실행되어야 한다. 시스템이 일시적으로 정지되더라도 구매 정보는 사라져서는 안 되며, 다시 시스템이 구동될 때 사용 가능해야 한다.  

### 2. 스프링의 트랜잭션 지원
<br/>
스프링은 코드 기반의 트랜잭션 처리(Programmatic Transaction) 뿐만 아니라 선언적 트랜잭션(Declarative Transaction)을 지원하고 있다. 따라서, 개발자가 직접적으로 트랜잭션의 범위를 코드 수준에서 정의하고 싶은 경우에는 스프링이 제공하는 트랜잭션 템플릿 클래스를 이용해서 손쉽게 트랜잭션 범위를 지정할 수 있다. 또한, 설정 파일이나 애노테이션을 이용해서 트랜잭션의 범위 및 규칙을 정의할 수 있기 때문에 트랜잭션을 매우 쉽게 관리할 수 있다.  

스프링은 데이터베이스 연동 기술에 상관없이 동일한 방식으로 트랜잭션을 처리할 수 있도록 하고 있다. 예를 들어, 하이버네이트를 사용하거나 JDBC를 사용하거나 JPA를 사용하거나 또는 JTA를 이용해서 트랜잭션을 처리하는 지의 여부에 상관없이 스프링은 동일한 코드를 이용해서 트랜잭션을 관리할 수 있도록 지원한다.  

#### 2.1. 스프링의 PlatformTransactionManager 설정
<br/>
스프링은 PlatformTransactionManager 인터페이스를 이용해서 트랜잭션 처리를 추상화하였고, 데이터베이스 연동 기술에 따라 알맞은 PlatformTransactionManager 구현 클래스를 제공하고 있다.  

각각의 트랜잭션 관리자 구현 클래스는 관련된 데이터베이스 기술에 따라 알맞은 트랜잭션 처리를 수행한다. 예를 들어, HibernateTransactionManager의 경우 내부적으로 하이버네이트의 Transation을 이용해서 트랜잭션을 처리한다. 즉, 트랜잭션 관련 처리를 HibernateTransactionManager에 요청하면, HibernateTransactionManager는 그 요청을 하이버네이트의 Transaction에 전파함으로써 트랜잭션을 처리하게 된다.  

실제 스프링의 트랜잭션 기능을 사용할 때에는 PlatformTransactionManager를 직접 사용하진 않는다. 대신, TransactionTemplate 클래스나 선언적 방식을 이용해서 트랜잭션을 처리하게 된다.  

스프링을 이용해서 트랜잭션을 처리하려면 사용하는 DB 연동 구현 기술에 알맞은 트랜잭션 관리자를 등록해주어야 한다.  

#### 2.2. JDBC 기반 트랜잭션 관리자 설정\
<br/>
JDBC나 MyBatis와 같이 JDBC를 이용해서 데이터베이스 연동을 처리하는 경우, 다음과 같이 DataSourceTransationManager를 트랜잭션 관리자로 사용한다.  

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
	<property name="driverClass" value="com.mysql.jdbc.Driver" />
	<property name="jdbcUrl" value="jdbc:mysql://localhost/guestbook?characterEncoding=utf-8" />
	<property name="user" value="spring4" />
	<property name="password" value="spring4" />
</bean>

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>
```

DataSource로부터 Connection을 가져온 뒤, Connection의 commit(), rollback() 등의 메서드를 사용해서 트랜잭션을 관리한다.  

#### 2.3. JPA 트랜잭션 관리자 설정
<br/>
JPA를 사용할 경우에는 JpaTransactionManager를 트랜잭션 관리자로 사용하면 된다. 아래 코드는 설정 예이다.  

```xml
<bean id="entityManagerFactory" class="org.sprinfrmaework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	...
</bean>

<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
	<property name="entityManagerFactory" ref="entityManagerFactory" />
</bean>
```

JpaTransactionManager는 entityManagerFactory 프로퍼티를 통해 전달받은 EntityManagerFactory를 이용해서 트랜잭션을 관리한다.  

#### 2.4. 하이버네이트 트랜잭션 관리자 설정
<br/>
하이버네이트를 사용하는 경우에는 HibernateTransactionManager를 트랜잭션 관리자로 사용한다. 아래 코드는 설정 예이다.  

```xml
<bean id="transactionManager" class="org.sprinframework.orm.hibernate3.HibernateTransactionManager">
	<property name="sessionFactory" ref="sessionFactory" />
</bean>

<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
	<property name="mappingResources" >
		...
	</property>
	<property name="hibernateProperties">
		...
	</property>
</bean>
```

HibernateTransactionManager는 sessionFactory 프로퍼티를 통해서 전달받은 하이버네이트 Session으로부터 하이버네이트 Transaction을 생성한 뒤, Transaction을 이용해서 트랜잭션을 관리한다.  

#### 2.5. JTA 트랜잭션 관리자 설정
<br/>
다중 자원에 접근하는 경우 JTA(Java Trnasaction API)를 이용해서 트랜잭션을 처리하게 되는데, 이 경우에는 JtaTransactionManager를 사용한다. JtaTransactionManager는 아래 코드와 같이 transactionManagerName 프로퍼티를 이용해서 JTATransactionManager를 구할 수 있는 JNDI 이름을 설정한다.  

```xml
<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager">
	<property name="transactionManagerName" value="java:comp/TranasactionManager" />
</bean>
```

컨테이너가 제공하는 TransactionManager를 사용하지 않고, TransactionEssentials과 같은 오픈 소스 라이브러리를 이용해서 로컬 JTA를 사용할 경우에는, userTransaction 프로퍼티를 이용해서 UserTransaction을 직접 설정할 수도 있다. 다음은 TransactionEssentials라는 JTA 구현체를 이용할 때 JtaTransactionManager를 설정하는 예를 보여주고 있다.  

```xml
<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager" depends-on="userTransactionService">
	<property name="transactionManager" ref="atomikosTransactionManager" />
	<property name="userTransaction" ref="atomikosUserTransaction" />
</bean>

<bean id="userTransactionService" class="com.atomikos.icatch.config.UserTransactionServiceImp" init-method="init" destroy-method="shutdownForce">
	...
</bean>

<bean id="atomikosTransactionManager" class="com.atomikos.icatch.jta.UserTransactionManager" init-method="init" destory-method="close" depends-on="userTransactionService">
	<property name="forceShutdown" value="false" />
</bean>

<bean id="atomikosUserTransaction" class="com.atomikos.icatch.jta.UserTransactionImp" depends-on="userTransactionService">
	<property name="transactionTimeout" value="300" />
</bean>
```

JtaTransactionManager는 내부적으로 javax.transaction.UserTransaction의 commit(), rollback() 등을 이용해서 트랜잭션을 처리한다.  

#### 2.6. 트랜잭션 전파와 격리 레벨
<br/>
현재 진행중인 트랜잭션이 있는 상태에서 새로운 트랜잭션을 시작하고 싶다면 어떻게 할까? JDBC를 이용한다면, 아마도 다음과 같이 새로운 커넥션을 가져와 트랜잭션을 시작할 것이다.  

```java
Connection conn = getConnection();
conn.setAutoCommit(false);

...

Connection connNew = getConnection();
connNew.setAutoCommit(false); // 새로운 커넥션으로 트랜잭션 시작

...

connNew.comiit(); // 새로운 트랜잭션 종료
connNew.close();

...

conn.commit();
conn.close();
```

스프링은 새로운 트랜잭션을 새로 생성하는 것뿐만 아니라 기존 트랜잭션을 그대로 사용하거나 기존에 트랜잭션이 진행중인 상태에서 현재 코드를 실행하는 등의 트랜잭션 전파와 관련된 부분을 지정할 수 있도록 지원하고 있다. 스프링이 제공하는 트랜잭션 전파 지원은 다음과 같다. 실제 설정에서 사용하는 값은 사용할 트랜잭션 지원 방식에 따라 조금씩 다르다. 예를 들어 TransactionTemplate에서는 PROPAGATION&#95;REQUIRED를 값으로 사용하는 반면에 &#60;tx:advice&#62; 태그에서는 REQUIRED를 사용한다.  

+ REQUIRED  
메서드를 수행하는 데 트랜잭션이 필요하다는 것을 의미한다. 현재 진행 중인 트랜잭션이 존재하면, 해당 트랜잭션을 사용한다. 존재하지 않는다면 새로운 트랜잭션을 생성한다.  
+ MANDATORY  
메서드를 수행하는 데 트랜잭션이 필요하다는 것을 의미한다. 하지만, REQUIRED와 달리, 진행 중인 트랜잭션이 존재하지 않을 경우 익셉션을 발생시킨다.  
+ REQUIRES&#95;NEW  
항상 새로운 트랜잭션을 시작한다. 기존 트랜잭션이 존재하면 기존 트랜잭션을 일시 중지하고 새로운 트랜잭션을 시작한다. 새로 시작된 트랜잭션이 종료된 뒤에 기존 트랜잭션이 계속된다.
+ SUPPORTS  
메서드가 트랜잭션을 필요로 하지는 않지만, 기존 트랜잭션이 존재할 경우 트랜잭션을 사용한다는 것을 의미한다. 진행 중인 트랜잭션이 존재하지 않더라도 메서드는 정상적으로 동작한다.  
+ NOT&#95;SUPPORTED  
메서드가 트랜잭션을 필요로 하지 않음을 의미한다. SUPPORTS와 달리 진행 중인 트랜잭션이 존재할 경우 메서드가 실행되는 동안 트랜잭션은 일시 중지되며, 메서드 실행이 종료된 후에 트랜잭션을 계속 진행한다.  
+ NEVER  
메서드가 트랜잭션을 필요로 하지 않으며, 만약 진행 중인 트랜잭션이 존재하면 익셉션을 발생시킨다.  
+ NESTED  
기존 트랜잭션이 존재하면, 기존 트랜잭션에 중첩된 트랜잭션에서 메서드를 실행한다. 기존 트랜잭션이 존재하지 않으면 REQUIRED와 동일하게 동작한다. 이 기능은 JDBC 3.0 드라이버를 사용할 때에만 적용된다. (JTA Provider가 이 기능을 지원할 경우에도 사용 가능하다.)  

스프링에서 설정 가능한 트랜잭션 격리 레벨은 다음과 같다.  

+ DEFAULT  
기본 설정을 사용한다.  
+ READ&#95;UNCOMMITTED  
다른 트랜잭션에서 커밋하지 않은 데이터를 읽을 수 있다.  
+ READ&#95;COMMITTED  
다른 트랜잭션에 의해 커밋된 데이터를 읽을 수 있다.  
+ REPEATABLE&#95;READ  
처음에 읽어 온 데이터와 두 번째 읽어 온 데이터가 동일한 값을 갖는다.  
+ SERIALIZABLE  
동일한 데이터에 대해서 동시에 두 개 이상의 트랜잭션이 수행될 수 없다.  

### 3. TransactionTemplate을 이용한 트랜잭션
<br/>
#### 3.1. TransactionTemplate과 TransactionCallback으로 트랜잭션 처리하기\
<br/>
직접 트랜잭션을 처리하려면 org.sprinframework.transaction.support.TransactionTemplate 클래스를 이용한다. TransactionTemplate 클래스는 트랜잭션과 관련된 작업(트랜잭션 시작, 커밋, 롤백 등)을 처리해주는 템플릿 클래스이다.  

TransactionTemplate을 사용하려면 먼저 TransactionTemplate을 빈으로 설정해주어야 한다. TransactionTemplate을 설정할 때에는 다음과 같이 transactionManager 프로퍼티에 스프링의 PlatformTransactionManager 빈을 지정하면 된다.  

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>

<bean id="transactionTemplate" class="org.springframework.taransaction.support.TransactionTemplate">
	<property name="transactionManager" ref="transactionManager" />
</bean>
```

TransactionTemplate 빈 객체를 설정했다면, 이 객체를 이용해서 DB 트랜잭션을 처리할 수 있다. 아래 코드는 TransactionTemplate 클래스의 사용 예를 보여주고 있다.  

TransactionTemplate의 execute() 메서드는 TransactionCallback 타입의 객체를 파라미터로 전달받는다. execute() 메서드는 내부적으로 PlatformTransactionManager를 이용해서 트랜잭션을 시작한 뒤에 파라미터로 전달받은 TransactionCallback의 doInTransaction() 메서드를 실행한다. doInTransaction() 메서드가 정상적으로 실행되면, TransactionTemplate의 execute() 메서드는 트랜잭션을 커밋하고 결과를 리턴한다.  

org,springframework.transaction.support.TransactionCallback 인터페이스는 다음과 같이 정의되어 있다.  

```java
public interface TransactionCallback<T> {
	T doInTransaction(TransactionStatus status);
}
```

일반적으로 TransactionTemplate 클래스의 execute() 메서드를 사용할 때에는 TransactionCallback 인터페이스를 구현한 클래스를 작성하기 보다는 앞서 살펴봤던 코드와 같이 임의 클래스를 사용하거나 자바 8의 람다식을 이용해서 TransactionCallback 객체를 전달한다.  

doInTransaction() 메서드는 throws를 통해서 발생시킬 수 있는 익셉션을 설정하고 있지 않기 때문에, doInTransaction() 메서드는 내부에서는 RuntimeException이나 Error 타입의 익셉션만 발생시킬 수 있다. 만약, doInTransaction 내부에서 반드시 catch로 처리해주어야 하는 checked 익셉션을 발생시킨다면, doInTransaction() 메서드 내부에서 try-catch 블록을 사용해서 익셉션을 처리한 뒤 롤백 여부를 설정해주어야 한다.  

TransactionStatus의 setRollbackOnly() 메서드는 트랜잭션을 롤백시킨다.  

#### 3.2. TransactionTemplate의 트랜잭션 설정
<br/>
TransactionTemplate은 기본적으로 다음의 트랜잭션 속성을 갖는다.

+ 트랜잭션 전파 속성 : 트랜잭션 필요 (REQUIRED)
+ 트랜잭션 격리 레벨 : 데이터베이스의 기본 (DEFAULT)
+ 트랜잭션 타임아웃 : 없음
+ 읽기 전용 아님  

만약 트랜잭션 속성을 다르게 지겅하고 싶다면, TransactionTemplate을 설정할 때 관련 속성을 변경해주면 된다. 위의 네 가지 속성과 관련된 프로퍼티 및 사용 가능한 값은 다음과 같다.

<table>
	<thead>
		<tr>
			<td>프로퍼티</td>
			<td>값</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>propagationBehaviorName(propagationBehavior)</td>
			<td>
				트랜잭션 전파 범위를 설정한다. 다음과 같은 값을 사용할 수 있다. (괄호 안의 숫자는 propagationBehavior 프로퍼티를 설정할 때 사용 가능한 값이다.)<br/><br/>
				<li>PROPAGATION&#95;REQUIRED(0)</li>
				<li>PROPAGATION&#95;SUPPORTS(1)</li>
				<li>PROPAGATION&#95;MANDATORY(2)</li>
				<li>PROPAGATION&#95;REQUIRES&#95;NEW(3)</li>
				<li>PROPAGATION&#95;NOT&#95;SUPPORTED(4)</li>
				<li>PROPAGATION&#95;NEVER(5)</li>
				<li>PROPAGATION&#95;NESTED(6)</li>
				<br/><br/>
				기본 값은 PROPAGATION&#95;REQUIRED이다.
			</td>
		</tr>
		<tr>
			<td>isolationLevelName(isolationLevel)</td>
			<td>
				트랜잭션 격리 레벨을 지정한다. (괄호 안의 값은 isolationLevel 프로퍼티를 설정할 때 사용 가능한 값이다.)<br/><br/>
				<li>ISOLATION&#95;DEFAULT(-1)</li>
				<li>ISOLATION&#95;READ&#95;UNCOMMITTED(1)</li>
				<li>ISOLATION&#95;READ&#95;COMMITED(2)</li>
				<li>ISOLATION&#95;REPEATABLE&#95;READ(4)</li>
				<li>ISOLATION&#95;SERIALIZABLE(8)</li>
				<br/><br/>
				기본 값은 ISOLATION&#95;DEFAULT이다.
			</td>
		</tr>
		<tr>
			<td>timeout</td>
			<td>초 단위로 타임아웃 값을 지정한다. 기본값은 -1로서 타임아웃이 없다.</td>
		</tr>
		<tr>
			<td>readOnly</td>
			<td>true를 지정하면 읽기 전용 트랜잭션이며, false로 지정하면 읽기/쓰기 트랜잭션으로 설정한다. 기본 값은 false이다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>
기존에 트랜잭션이 존재했는지 여부에 상관없이 트랜잭션을 새롭게 시작하고, 격리 레벨을 REPEATABLE&#95;READ로 지정하고 싶다면 다음과 같이 TransactionTemplate을 설정할 수 있을 것이다.  

```xml
<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
	<property name="transactionManager" ref="transactionManager" />
	<property name="propagationBehaviorName" ref="PROPAGATION_REQUIRES_NEW" />
	<property name="isolationLevelName" ref="ISOLATION_REPEATABLE_READ" />
</bean>
```

### 4. 트랜잭션과 DataSource
<br/>
TransactionTemplate 클래스의 execute() 메서드에서 각 DAO의 메서드가 한 트랜잭션으로 실행될 수 있을까? (JTA를 사용하지 않는 이상) JDBC의 경우 동일한 Connection을 사용해야 한 트랜잭션으로 묶을 수 있기 때문이다.  

여기서 비밀은 JdbcTemplate에 있다. JdbcTemplate의 쿼리 실행 메서드는 내부적으로 다음의 코드를 이용해서 Connection을 구한다.  

```java
// JdbcTemplate이 내부적으로 Connection을 구할 때 사용하는 코드
Connection con = DataSourceUtils.getConnection(getDataSource());
```

위 코드에서 DataSourceUtils.getConnection() 메서드는 현재 코드가 트랜잭션 범위에서 실행되고 있으면, 트랜잭션과 엮어 있는 Connection을 리턴한다. 트랜잭션 범위에 있지 않을 경우 새로운 Connection을 리턴한다. SimpleJdbcInsert을 포함한 다른 템플릿 클래스도 내부적으로 JdbcTemplate을 사용하기 때문에 결과적으로 현재 트랜잭션이 진행 중일 경우 같은 Connection을 이용하게 된다.  

그렇다면 DataSourceUtils.getConnection() 메서드는 트랜잭션 범위 내에 있는지를 어떻게 알 수 있을까? 이는 DataSourceTransactionManager가 처리한다. DataSourceTransactionManager는 트랜잭션을 시작할 때 DataSourceUtils에 트랜잭션이 시작되었음을 알리며, 이후 DataSourceUtils.getConnection() 메서드는 DataSourceTransactionManager가 시작한 트랜잭션과 연결된 Connection 객체를 리턴한다.  

이런 동작 방식 때문에, DateSource를 직접 사용해서 Connection 객체를 가져와 사용할 경우 그 코드는 스프링이 제공하는 트랜잭션 범위 내에서 실행되지 않는다.  

만약 DataSource를 직접 사용하면서 스프링의 트랜잭션 지원 기능을 사용하고 싶다면, Connection 객체를 구할 때 다음과 같이 DataSourceUtils 클래스의 getConnection() 메서드를  사용해야 한다.  

### 5. 선언적 트랜잭션 처리
<br/>
선언적 트랜잭션(Declarative Transaction)은 TransactionTemplate과 달리 트랜잭션 처리를 코드에서 직접 수행하지 않고, 설정 파일이나 애노테이션을 이용해서 트랜잭션의 범위, 롤백 규칙 등을 정의하게 된다. 선언적 트랜잭션은 다음과 같은 두 가지 방식으로 정의할 수 있다.  

+ &#60;tx:advice&#62; 태그를 이용한 트랜잭션 처리
+ &#64;Transactional 애노테이션을 이용한 트랜잭션 설정  

#### 5.1. tx 네임스페이스를 이용한 트랜잭션 설정
<br/>
&#60;tx:advice&#62; 태그를 이용해서 트랜잭션 속성을 정의하기 위해서는 먼저 tx 네임스페이스를 추가해주어야 한다. tx 네임스페이스를 &#60;beans&#62; 태그에 추가한 뒤, &#60;tx:advice&#62; 태그, &#60;tx:attribute&#62; 태그, 그리고 &#60;tx:method&#62; 태그를 이용해서 트랜잭션 속성을 정의할 수 있다.  

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd
			http://www.springframework.org/schema/aop
			http://www.springframework.org/schema/aop/spring-aop.xsd
			http://www.springframework.org/schema/tx
			http://www.springframework.rog/schema/spring-tx.xsd">

	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager" p:dataSource-ref="dataSource" />

	<tx:advice id="txAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="order" propagation="REQUIRED" />
			<tx:method name="get*" read-only="true" />
		</tx:attributes>
	</tx:advice>

	...

</beans>
```

먼저, &#60;tx:advice&#62; 태그는 트랜잭션을 적용할 때 사용될 Advisor를 생성한다. id 속성은 생성될 트랜잭션 Advisor의 식별 값을 입력하며, transaction-manager 속성에는 스프링의 PlatformTransactionManager 빈을 설정한다.  

&#60;tx:method&#62; 태그는 &#60;tx:attributes&#62; 태그의 자식 태그로 설정하며, 트랜잭션을 적용할 메서드 및 트랜잭션 속성을 설정한다. &#60;tx:method&#62;의 속성은 다음과 같다.  

+ name  
트랜잭션이 적용될 메서드 이름을 명시한다. '&#42;'을 사용한 설정이 가능하다. 예를 들어, "get&#42;"으로 설정할 경우 이름이 get으로 시작하는 메서드를 의미한다.  
+ propagation  
트랜잭션 전파 규칙을 설정한다. REQUIRED(기본값), MANDATORY, REQUIRES&#95;NEW, SUPPORTS, NOT&#95;SUPPORTED, NEVER, NESTED를 값으로 갖는다.  
+ isolation  
트랜잭션 격리 레벨을 설정한다. DEFAULT, READ&#95;UNCOMMITTED, READ&#95;COMMITTED, REPEATABLE&#95;READ, SERIALIZABLE을 값으로 갖는다.  
+ read-only  
읽기 전용 여부를 설정한다.  
+ no-rollback-for  
트랜잭션을 롤백하지 않을 익셉션 타입을 지정한다.  
+ rollback-for  
트랜잭션을 롤백할 익셉션 타입을 지정한다.  
+ timeout  
트랜잭션의 타임 아웃 시간을 초 단위로 지정한다.  

&#60;tx:advice&#62; 태그는 Advisor만 생성하는 것이지 실제로 트랜잭션을 적용하는 것은 아니다. 실제로 트랜잭션을 적용하는 것은 AOP를 통해서 이루어진다. 아래 코드는 설정 예이다.  

```xml
<tx:advice id="txAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<tx:method name="order" propagation="REQUIRED" />
	</tx:attribuates>
</tx:advice>

<aop:config>
	<aop:pointcut id="servicePublicMethod" expression="execution(public * net.madvirus.spring4..*Service.*(..))" />
	<aop:advisor advice-ref="txAdvice" pointcut-ref="servicePublicMethod" />
</aop-config>
```

위 코드는 &#60;aop:config&#62; 태그를 이용해서 net.madvirus.spring4 패키지의 하위 패키지에 있는 &#42;Service의 public 메서드에 &#60;tx:advice&#62;로 설정한 트랜잭션 Advisor를 적용하도록 설정하고 있다. &#60;tx:method&#62; 태그에서 이름이 "order"인 메서드에 대해 트랜잭션을 적용하도록 설정했으므로, 실제로는 &#42;Service의 public 메서드 중에서 order 메서드를 호출할 때에 트랜잭션이 적용된다.  

##### 5.1.1. &#60;tx:method&#62; 태그의 rollback-for 속성과 no-rollback-for 속성을 통한 롤백 처리
<br/>
스프링 트랜잭션은 기본적으로 RuntimeException 및 Error에 대해서만 롤백 처리를 수행한다. 따라서, Throwable이나 Exception 타입의 익셉션이 발생하더라도 롤백되지 않고, 익셉션이 발생하기 전까지의 작업이 커밋된다.  

익셉션 발생시 트랜잭션의 롤백 규칙을 좀 더 정교하게 정의하고 싶다면 rollback-for 속성과 no-rollback-for 속성을 사용하면 된다. rollback-for 속성은 익셉션 발생시 롤백 작업을 수행할 익셉션 타입을 설정하며, no-rollback-for 속성은 익셉션이 발생하더라도 롤백하지 않을 익셉션 타입을 설정한다.  

명시할 익셉션 타입이 한 개 이상인 경우 각각의 익셉션은 콤마로 구분한다. 익셉션 클래스는 완전한 이름을 입력하거나 또는 패키지 이름을 제외한 클래스 이름만 입력해도 된다.  

#### 5.2. 애노테이션 기반 트랜잭션 설정
<br/>
&#64;Transactional 애노테이션을 사용해서 트랜잭션을 설정할 수도 있다. &#64;Transactional 애노테이션은 다음 코드와 같이 메서드나 클래스에 적용되며 관련 트랜잭션 속성을 설정한다.  

```java
public class 클래스명 {

	...

	@Transactional
	public Object 메서드명() {
		...
	}

	...

}
```

&#64;Transactional 애노테이션은 propagation 속성을 비롯하여 다음에 표시한 속성을 이용해서 트랜잭션 속성을 정의한다.  

+ propagation  
트랜잭션 전파 규칙을 설정한다. org.springframework.transaction.annotation.Propagation 열거형 타입에 같이 정의되어 있다. 기본값은 Propagation.REQUIRED이다.  
+ isolation  
트랜잭션 격리 레벨을 설정한다. org.springframework.transaction.annotation.Isolation 열거형 타입에 같이 정의되어 있다.  
+ readOnly  
읽기 전용 여부를 설정한다. boolean 값을 설정하며, 기본값은 false이다.  
+ rollbackFor  
트랜잭션을 롤백할 익셉션 타입을 설정한다. 예, rollbackFor=(Exception.class)  
+ noRollbackFor  
트랜잭션을 롤백할 익셉션 타입을 설정한다. 예, noRollbackFor=(ItemNotFoundException.class)  
+ timeout  
트랜잭션의 타임아웃 시간을 초 단위로 설정한다.  

&#64;Transactional 애노테이션이 적용된 스프링 빈에 트랜잭션을 실제로 적용하려면 다음과 같이 &#60;tx:annotation-driven&#62; 태그를 설정해주어야 한다.  

```xml
<tx:annoatation-driven transaction-manager="transactionManager"/>
```

&#60;tx:annotation-driven&#62; 태그가 제공하는 속성은 다음과 같다.  

<table>
	<thead>
		<tr>
			<td>속성</td>
			<td>설명</td>
			<td>기본값</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>transaction-manager</td>
			<td>사용할 PlatformTransactionManager 빈의 이름</td>
			<td>transactionMaanger</td>
		</tr>
		<tr>
			<td>proxy-target-class</td>
			<td>클래스에 대해서 프록시를 생성할 지의 여부, true일 경우 CGLIB를 이용해서 프록시를 생성하며, false인 경우 자바 다이나믹 프록시를 이용해서 프록시를 생성한다.</td>
			<td>false</td>
		</tr>
		<tr>
			<td>order</td>
			<td>Advice 적용 순서</td>
			<td>int의 최대값(가장 낮은 순위)</td>
		</tr>
	</tbody>
</table>
<br/><br/>
자바 코드 설정을 사용한다면 &#64;EnableTransactionManagement 클래스를 이용해서 &#64;Transactional 애노테이션을 적용할 수 있다.  

&#64;EnableTransactionManagement 애노테이션과 &#60;tx:annotation-driven&#62; 태그와의 차이점이 있다면, &#60;tx:annotation-driven&#62; 태그는 사용할 PlatformTransactionManager 빈의 이름을 직접 지정하는 반면에 &#64;EnableTransactionManagement 애노테이션은 PlatformTransactionManagement 타입의 빈을 PlatformTransactionManager로 사용한다는 점이다.  

만약 &#64;EnableTrnasactionManagement 애노테이션을 이용할 때, 사용할 PlatformTransactionManager를 직접 지정하고 싶다면 TransactionManagementConfigurer 인터페이스를 상속받은 자바 설정 코드를 구현해 주어야한다. 다음은 이 인터페이스의 사용 예이다.  

```java
@Configuration
@EnableTransactionManagement
public class 빈설정클래스명 implements TransactionManagementConfigurer {

	...

	@Override
	@Bean
	public PlatformTransactionManager annotationDrivenTransactionManager() {
		DataSourceTransactionManager txMgr = new DataSourceTransactionManager();
		txMgr.setDataSource(dataSource());
	}

	@Bean(destoryMethod = "close")
	public DataSource dataSource() {
		ComboPooledDataSource ds = new ComboPooledDataSource();
		try {
			ds.setDriverClass("com.mysql.jdbc.Driver");
		} catch (PropertyVetoException e) {
			throw new RuntimeException(e);
		}

		...

		return ds;
	}

	...

}
```

위 코드에서 annotationDrivenTransactionManager()는 TransactionManagerConfigurer 인터페이스에 정의되어 있는 메서드로서, TransactionManagerConfigurer 타입을 가진 &#64;Configuration 설정 클래스의 annotationDrivenTransactionManager() 메서드가 생성한 PlatformTransactionManager 빈을 트랜잭션 관리자로 사용한다.  

&#64;EnableTransactionManagement 애노테이션은 다음의 속성을 갖는다.  

+ proxyTargetClass  
클래스를 이용해서 프록시를 생성할지 여부를 지정한다. 기본 값은 false로서 인터페이스를 이용해서 포륵시를 생성한다.  
+ order  
AOP 적용 순서를 지정한다. 기본 값은 가장 낮은 우선순위에 해당하는 int의 최대값이다.  

##### 5.2.1. 트랜잭션 관리자 지정하기
<br/>
한 개의 어플리케이션에서 두 개 이상의 DB를 사용할 때가 있다. 예를 들어, 회원 관리 기능과 주문 관리 기능을 하나의 웹 어플리케이션에서 제공하는데, 회원 DB와 주문 DB가 분리되어 있다고 해보자. 만약 두 개의 DB에 동시에 변경을 가할 일이 없다면 글로벌 트랜잭션을 사용할 필요가 없으며, 이 경우 DB별로 트랜잭션 관리자를 따로 지정하는 것이 성능면에서 유리하다. 예를 들면 다음과 같이 DataSource별로 PlatformTransactionManager를 지정할 수 있을 것이다.  

```xml
<bean id="memTxMgr" class="org,springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="memDataSource" />
</bean>

<bean id="orderTxMgr" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="orderDataSource" />
</bean>

<tx:annotation-driven transaction-manager="memTxMgr" />
```

위 코드의 경우 &#60;tx:annotation-driven&#62; 태그의 transaction-manager 속성 값으로 "memTxMgr" 빈을 지정했다. 따라서, &#64;Transactional 애노테이션을 만나면, memTxMgr이 관리하는 트랜잭션 범위 내에서 코드를 실행하게 된다. 여기서 문제는 orderTxMgr이 관리하는 트랜잭션 범위 내에서 동작해야 하는 코드에서 &#64;Transactional을 사용하면, orderTxMgr이 아닌 memTxMgr이 관리하는 트랜잭션 범위 내에서 동작하게 된다는 것이다.  

이처험 PlatformTrnasactionManagement를 두 개 이상 정의한 상태에서 코드에 따라 &#64;Transactional 애노테이션이 속할 트랜잭션 범위를 다르게 설정하려면, &#64;Transactional 애노테이션의 value 속성 값으로 사용할 PlatformTransactionManagement 빈의 이름을 지정하면 된다.  

```java
@Transactional(value="orderTxMgr")
public Object 메서드명() {
	...
}
```

빈 이름 대신 한정자(qualifier)를 이용해도 된다.  

```xml
<bean id="orederTxMgr" class="org,springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
	<qualifier value="orderTx" />
</bean>
```

위 코드처럼 한정자 값을 "orderTx"로 지정했다면, &#64;Transactional 애노테이션에서 "orderTx"를 값으로 사용해서 orderTxMgr을 트랜잭션 관리자로 사용할 수 있다.  

#### 5.3. 트랜잭션과 프록시
<br/>
선언적 트랜잭션은 스프링의 AOP를 이용하고 있다. &#64;Transactional이나 tx 네임스페이스를 이용하면, 트랜잭션을 처리하기 위해 빈 객체에 대한 프록시 객체를 생성한다. 이 프록시 객체는 PlatformTransactionManager를 이용해서 트랜잭션을 시작한 뒤에, 실제 객체의 메서드를 호출하고, 그 다음에 PlatformTransactionManager를 이용해서 트랜잭션을 커밋한다.  

선언적 트랜잭션은 AOP를 사용하므로, 하나의 객체에 대해 두 개 이상의 프록시 객체가 생성될 수 있다. (예, 트랜잭션을 위한 프록시와 객발자가 직접 구현한 AOP 프록시) 이 때, 프록시가 실행되는 순서가 중요하다면, 원하는 순서로 프록시가 적용되도록 명시적으로 AOP 순서를 지정해야 한다.  

### 6. TransactionsEssentials를 이용한 분산 트랜잭션
<br/>
두 개 이상의 자원에 동시에 접근하는 데 트랜잭션이 필요한 경우가 있다. 예를 들어, 결재 정보를 저장하는 데이터베이스와 구매 내역을 저장하는 데이터베이스가 다르다고 하자. 이 경우 두 데이터베이스에 접근하기 위한 DataSource는 서로 다르지만, 두 데이터베이스에 접근하는 코드는 단일 트랜잭션으로 처리되어야 한다.  

자바에서 분산 트랜잭션을 처리하기 위해서는 분산 트랜잭션 서비스를 제공해주는 트랜잭션 관리자가 필요하다. WebLogic이나 JBoss 같은 컨테이너는 자체적으로 분산 트랜잭션을 지원하고 있지만, 톰캣과 같은 서블릿 컨테이너는 분산 트랜잭션을 지원하고 있지 않다.  

단위 테스트를 수행할 때 컨테이너 없이 분산 트랜잭션을 테스트해야 한다거나 톰캣과 같이 분산 트랜잭션 서비스를 지원하지 않는 컨테이너에서 분산 트랜잭션을 구현해야 한다면, TransactionEssentials나 Bitronix와 같은 트랜잭션 매니저를 이용하면 된다.  

최근의 추세는 두 개 이상의 자원을 하나의 트랜잭션으로 묶어서 처리하는 방식보다는 중간에 메시징 시스템을 두고 비동기로 데이터를 동기화하는 방식을 채택하고 있다. 비동기 방식을 선택하는 이유 중의 하나는 성능 때문이다. 하지만, 성능보다 트랜잭션 보장이 더 중요한 영역이 존재하므로 두 개 이상의 자원에 대해 엄격하게 트랜잭션을 적용해야 한다면 글로벌 트랜잭션을 사용해야 한다.  

#### 6.1. TransactionsEssentials 메이븐 설정
<br/>
TransactionsEssentials은 Atomikos에서 개발한 ExtremeTransactions의 오픈 소스 버전으로서 메이븐 리파지터리에 등록되어 있다. 따라서, 다음과 같이 pom.xml 파일에 관련 의존 설정을 추가해주기만 하면 TransactionEssentials을 이용한 분산 트랜잭션을 사용할 수 있다. (참고로, 아래 코드는 3.9.3 버전을 기준으로 작성된 것으로서 버전에 따라 차이점이 존재할 수 있다.)  

```xml
<dependencies>
	...
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-tx</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<!-- TransactionsEssentials를 RDBMS에 사용하기 위한 의존 모듈 설정 -->
	<dependency>
		<groupId>com.atomikos</groupId>
		<artifactId>transactions-jdbc</artifactId>
		<version>3.9.3</version>
	</dependency>
	
	<!-- 스프링4 버전은 JTA 1.1을 사용한다 -->
	<dependency>
		<groupId>javax.transaction</groupId>
		<artifactId>jta</artifactId>
		<version>1.1</version>
	</dependency>
</dependencies>	
```

#### 6.2. TransactionsEssentials와 스프링 연동
<br/>
TransactionEssentials에 대한 의존 설정을 완료했다면, 스프링 설정에 다음과 같은 정보를 추가한다.  

+ TransactionsEssentials를 이용한 JtaTransactionManager 설정
+ TransactionsEssentials가 제공하는 클래스를 이용한 XADataSource 설정
+ DAO 등 스프링 빈에서 XADataSource를 사용하도록 설정  

##### 6.2.1. TransactionsEssentials를 이용한 JtaTransactionManager 설정
<br/>
TransactionsEssentials을 이용하는 JtaTransactionManager를 설정하려면 아래와 같은 설정을 추가해주어야 한다.  

```xml
<!-- Transaction Essentials를 이용한 JtaTransactionManager 설정 -->
<bean id="userTransactionService" class="com.atomikos.icatch.config.UserTransactionServiceImpl" init-method="init" destroy-method="shutdownForce">
	<constructor-arg>
		<!-- 여기에 Atomikos 프로퍼티 위치 -->
		<props>
			<prop key="com.atomikos.icatch.service">
				com.atomikos.icatch.standalone.UserTransactionServiceFactory
			</prop>
		</props>
	</constructor-arg>
</bean>

<bean id="atomikosTransactionManager" class="com.atomikos.icatch.jta.UserTransactionManager" init-method="init" destroy-method="close">
	<property name="startupTransactionService" value="false"/>
</bean>

<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager">
	<property name="transactionManager">
		<ref bean="atomikosTransactionManager" />
	</property>
	<property name="userTransaction">\
		<ref bean="atomikosUserTransaction" />
	</property>
</bean>
```

##### 6.2.2. TransactionsEssentials를 이용한 XADataSource 설정
<br/>
JtaTransactionManager 설정이 완료되었다면, 그 다음으로 할 작업은 TransactionsEssentials가 제공하는 클래스를 이용해서 XADataSource를 생성하는 것이다. TransactionsEssentials는 다음의 두 가지 XADataSource를 제공하고 있다.  

+ com.atomikos.jdbc.AtomikosDataSourceBean  
XA를 지원하는 JDBC 드라이버를 위한 DataSource 설정.  
+ com.atomikos.jdbc.nonxa.AtomikosNonXADataSource  
XA를 지원하지 않는 JDBC 드라이버를 위한 DataSource 설정. 이 클래스는 XA에 호환되지 않기 때문에 트랜잭션의 원자성(atomic)을 보장할 수 없다.  

XA를 지원하는 JDBC 드라이버를 갖고 있다면, AtomikosDataSourceBean를 이용해서 DataSource를 설정해주면 된다. 아래 코드는 설정 예를 보여주고 있다.

```xml
<!-- XA를 위한 DataSource 설정 -->
<bean id="shopDataSource" class="com.atomikos.jdbc.AtomikosDataSourceBean">
	<property name="uniqueResourceName" value="shopXaDs" />
	<property name="xaDataSourceClassName" value="com.mysql.jdbc.jdbc2.optional.MysqlXADataSource" />
	<property name="xaProperties">
		<props>
			<prop key="user">spring4</prop>
			<prop key="password">spring4</prop>
			<prop key="url">jdbc:mysql://localhost/shop?characterEncoding=utf8</prop>
			<prop key="pinGlobalTxToPhysicalConnection">true</prop>
		</props>
	</property>
</bean>
```

AtomikosDataSourceBean 클래스를 사용할 때 설정할 프로퍼티는 다음과 같다.  

+ uniqueResourceName : DataSource를 식별하는 고유 자원 이름
+ xaDataSourceClassName : XADataSource 구현 클래스의 완전한 이름
+ xaProperties : XADataSource를 설정할 때 필요한 &#60;이름, 값&#62; 쌍  

JDBC 드라이버가 XA를 지원하지 않는 경우에는 AtomikosNonXADataSourceBean 클래스를 사용해서 DataSource를 설정하면 된다. 아래 코드는 설정 예이다.  

```xml
<bean id="shopDataSource" class="com.atomikos.jdbc.nonxa.AtomikosNonXADataSourceBean" init-method="init" destroy-method="close">
	<property name="uniqueResourceName" value="shopXaDs" />
	<property name="user" value="spring4" />
	<property name="password value="spring4" />
	<property name="url" value="jdbc:mysql://localhost/shop?characterEncoding=utf8" />
	<property name="driverClassName" value="com.mysql.jdbc.Driver" />
	<property name="poolSize" value="10" />
</bean>
```

AtomikosNonXADataSourceBean 클래스의 주요 프로퍼티는 다음과 같다.  

+ user : DB에 접근할 때 사용할 사용자 계정
+ password : DB에 접근할 때 사용할 암호
+ url : JDBC URL
+ driverClassName : JDBC 드라이버 클래스 이름  

AtomikosNonXADataSourceBean 클래스의 경우 완전한 XA를 지원하지 않기 때문에, 원자성(atomic)을 100% 보장하지 못한다는 점에 유의해야 한다.  

##### 6.2.3. TransactionsEssentials가 제공한 XADataSource를 이용
<br/>
TransactionsEssentials가 제공하는 클래스를 이용해서 XADataSource를 설정했다면, 남은 작업은 하나의 트랜잭션 범위에서 두 개 이상의 DataSource를 사용하도록 설정하는 것뿐이다.  

##### 6.2.4. 커넥션 풀 관련 프로퍼티
<br/>
AtomikosDataSourceBean 클래스와 AtomikosNonXaDataSourceBean 클래스의 커넥션 풀 관련 프로퍼티는 다음과 같다.

<table>
	<thead>
		<tr>
			<td>프로퍼티</td>
			<td>설명</td>
			<td>기본값</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>maxPoolSize</td>
			<td>최대 풀 개수</td>
			<td>1</td>
		</tr>
		<tr>
			<td>minPoolsize</td>
			<td>최소 풀 개수</td>
			<td>1</td>
		</tr>
		<tr>
			<td>poolSize</td>
			<td>최대/최소 풀 개수를 한번에 설정한다. maxPoolSize를 설정하지 않은 경우 필요하다.</td>
			<td>없음</td>
		</tr>
		<tr>
			<td>borrowConnectionTimeout</td>
			<td>풀에서 커넥션을 가져오기까지 대기 시간. 시간 단위는 초이다.</td>
			<td>30</td>
		</tr>
		<tr>
			<td>maintenanceInterval</td>
			<td>커넥션 풀의 검사 주기를 초 단위로 설정한다.</td>
			<td>60</td>
		</tr>
		<tr>
			<td>maxIdleTime</td>
			<td>사용되지 않는 커넥션의 최대 유효 시간</td>
			<td>60</td>
		</tr>
		<tr>
			<td>testQuery</td>
			<td>사용된 커넥션을 풀에 되돌리기 전에 커넥션을 검사할 때 사용할 쿼리. 이 쿼리는 JTA 트랜잭션 범위에서 실행되지 않는다.</td>
			<td>null</td>
		</tr>
	</tbody>
</table>
