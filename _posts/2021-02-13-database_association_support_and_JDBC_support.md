---
title: 데이터베이스 연동 지원과 JDBC 지원
categories:
- Spring
feature_text: |
  ## 데이터베이스 연동 지원과 JDBC 지원
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	td { border: 1px solid #444444; }
</style>

상당수 웹 어플리케이션은 데이터베이스 연동을 피요로 한다. JDBC API를 이용해서 데이터베이스에 접근할 수 있으며, 하이버네이트나 JPA와 같은 ORM 프레임워크를 이용해서 데이터베이스를 연동할 수도 있다. 스프링은 JDBC를 비롯하여 ORM 프레임워크를 이용해서 데이터베이스를 연동할 수도 있다. 스프링은 JDBC를 비롯하여 ORM 프레임워크를 직접적으로 지원하고 있기 때문에, 약간의 노력만 들이면 JDBC 뿐만 아니라 다른 ORM 프레임워크를 스프링과 쉽게 연동할 수 있다.  

### 1. 스프링의 데이터베이스 연동 지원
<br/>
스프링은 JDBC를 이용한 DAO 클래스를 구현할 수 있도록 다양한 기능을 지원하고 있는데, 그 내용은 다음과 같다.  

+ 템플릿 클래스를 통한 데이터 접근 지원
+ 의미 있는 익셉션 타입
+ 트랜잭션 처리  

이 세 가지는 어떤 연동 방식을 사용해도 동일하게 적용된다.  

#### 1.1. 데이터베이스 연동을 위한 템플릿 클래스
<br/>
데이터에 접근하는 코드는 거의 동일한 코드 구성을 갖는다. 이런 반복되는 구조의 코드는 개발자를 귀찮게 만들 뿐만 아니라 코드 누락 등의 실수를 유발할 수 있다. 템플릿 메서드 패턴과 전략 패턴을 함께사용하면 이런 구조적인 중복을 줄일 수 있는데, 스프링은 이미 이 두 가지 패턴이 적용된 JDBC 템플릿 클래스를 제공하고 있다.  

예를 들어, JDBC를 위한 JdbcTemplate 클래스를 제공하고 있으며, 이 클래스를 사용하면 try-catch-finally 블록 및 커넥션 관리를 위한 중복되는 코드를 줄이거나 없앨 수 있다.  

#### 1.2. 스프링의 익셉션 지원
<br/>
JDBC 프로그래밍을 할 때 아쉬운 부분 중의 하나는 데이터베이스 처리 과정에서 발생하는 에러는 항상 SQLException이라는 점이다. 예를 들어, SQLException을 catch하는 코드는 Connection을 구하는 과정에서 익셉션이 발생했는지 Statement를 생성하는 과정에서 익셉션이 발생했는지, 아니면 SQL 쿼리를 실행하는 과정에서 익셉션이 발생했는지 바로 알 수 없다.  

왜 익셉션이 발생했는지 확인하려면 SQLException의 실제 타입이 뭔지 확인해야 하고, 에러 코드를 확인해야 한다. 하지만, 익셉션이 발생하는 원인을 찾기 위한 코드를 작성하는 일은 꽤 성가신 일이다.  

스프링은 데이터베이스 처리 과정에서 발생한 익셉션이 왜 발생했는지를 구체적으로 확인할 수 있도록, 데이터베이스 처리와 관련된 익셉션 클래스를 제공하고 있다. 예를 들어, OptimisticLockingFailureException이나 DataRetrieveFailureException과 같이 실패(익셉션 발생) 원인을 보다 구체적으로 설명해주는 익셉션 클래스를 제공하고 있다.  

JdbcTemplate 클래스는 DB 연동 과정에서 SQLException이 발생하면 스프링이 제공하는 익셉션 클래스 중 알맞는 익셉션 클래스로 변환해서 발생시킨다. 예를 들어, 올바르지 않은 SQL 쿼리를 실행하는 경우 JdbcTemplate은 BadSqlGrammerException을 발생시킨다.  

스프링이 제공하는 데이터베이스 관련 익셉션 클래스들은 모두 DataAccessException 클래스를 상속받고 있는데, DataAccessException은 RuntimeException이다. 따라서, 필요한 경우에만 try-catch 블록을 이용해서 익셉션을 처리하면 된다.  

JdbcTemplate 뿐만 아니라 JPA, 하이버네이트, MyBatis를 위한 지원 기능은 내부적으로 발생하는 익셉션 클래스를 알맞게 변환한 익셉션을 발생시킨다. 따라서, 스프링이 제공하는 기능을 사용하면 데이터베이스 연동 기술에 상관없이 동일한 익셉션 타입을 이용해서 에러를 처리할 수 있게 된다.  

### 2. DataSource 설정
<br/>
데이터베이스 연동을 하려면 데이터베이스에 연결해야 한다. 자바에서 데이터베이스에 연결하는 방법에는 몇 가지가 있는데, 그중에서 스프링은 DataSource 방식을 사용하고 있다. 앞서 잠시 언급했던 JdbcTemplate이나 JPA/하이버네이트/MyBatis 지원 기능을 사용할 경우, 데이터베이스 연결을 위해 DataSource를 설정해주어야 한다.  

스프링은 다음과 같이 세 가지 방식의 DataSource 설정을 지원하고 있다.  

+ 커넥션 풀을 이용한 DataSource 설정
+ JNDI를 이용한 DataSource 설정
+ DriverManager를 이용한 DataSource 설정 (테스트 목적)  

#### 2.1. 커넥션 풀을 이용한 DataSource 설정
<br/>
스프링은 커넥션 풀 구현 클래스를 직접 제공하고 있진 않다. 대신 c3p0와 같은 커넥션 풀 라이브러리를 이용해서 커넥션 풀을 지원하는 DataSource를 설정할 수 있다. 이를 위해 먼저 해야 할 일은 메이븐 의존에 c3p0 라이브러리를 추가하는 것이다.  

```xml
<!-- pom.xml 파일의 의존 설정 부분 -->
<dependencies>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-jdbc</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<!-- c3p0 라이브러리 의존 -->
	<dependency>
		<groupId>com.mchange</groupId>
		<artifactId>c3p0</artifactId>
		<version>0.9.2.1</version>
	</dependency>
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>5.1.30</version>
	</dependency>
</dependencies>
```

위 의존 설정을 보면 c3p0 외에 JDBC 드라이버 의존도 함께 추가한 것을 알 수 있다. MySQL과 달리 오라클이나 MS SQL과 같은 상용 DBMS의 JDBC 드라이버는 라이선스 문제로 메이븐 중앙 리파지터리에 존재하지 않는 경우가 있다. 이런 경우 메이븐 명령어를 이용해서 직접 로컬 리파지터리에 등록해야 한다.  

메이븐 의존을 설정했다면, c3p0가 제공하는 ComboPoolDataSource 클래스를 이용해서 DataSource를 설정할 수 있다. 다음 설정 코드 예를 보여주고 있다.  

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
	<property name="driverClass" value="com.mysql.jdbc.Driver" />
	<property name="jdbcUrl" value="jdbc:mysql://localhost/guestbook" />
	<property name="user" value="spring4" />
	<property name="password" value="spring4" />
</bean>
```

ComboPooledDataSource 클래스는 커넥션 풀을 관리하기 다양한 프로퍼티를 제공하는데, 이들 프로퍼티는 다음과 같다.  

+ acquireIncrement  
풀에 커넥션이 없을 때 증가시킬 커넥션의 개수, 기본값은 3이다.
+ initialPoolSize  
초기의 커넥션 풀의 크기, 기본값은 3이다.
+ maxPoolSize  
커넥션 풀의 최대 크기, 기본값은 15이다.
+ minPoolSize  
커넥션 풀의 최소 크기, 기본값은 3이다.
+ maxConnectionAge  
커넥션의 유효 시간, 단위는 초, 지정한 시간이 지나면 자동으로 풀에서 제거된다. 값이 0일 경우 제거되지 않는다. 기본값은 0이다.
+ maxIdleTime  
지정한 시간 동안 사용되지 않는 커넥션을 제거한다. 단위는 초이다. 값이 0일 경우 제거하지 않는다. 기본값은 0이다.
+ checkoutTimeout  
풀에서 커넥션을 가져올 때 대기 시간. 1/1000초. 0은 무한히 기다리는 것을 의미한다. 지정한 시간 동안 풀에서 커넥션을 가져오지 못할 경우 SQLException을 발생시킨다. 기본값은 0이다.
+ automaticTestTable  
값이 존재할 경우 지정한 이름의 테이블을 생성한 뒤, 해당 테이블을 이용해서 커넥션이 유효한 지의 여부를 검사한다. 기본값은 null이다. 이 값을 제공하면 preferredTestQuery는 무시된다.
+ idleConnectionTestPeriod  
풀 속에 있는 커넥션의 테스트 주기. 단위는 초이며, 0인 경우 검사하지 않는다. 기본값은 0이다.
+ preferredTestQuery  
커넥션을 테스트 할 때 사용할 쿼리. 기본값은 null이다.
+ testConnectionOnCheckIn  
true인 경우 커넥션을 풀에 반환할 때 커넥션이 유효한지의 여부를 비동기로 검사한다. 기본값은 false이다.
+ testConnectionOnCheckout  
true인 경우 커넥션을 풀에서 가져올 때 유효한지의 여부를 검사한다. 기본값은 false이다. 추가적인 검사로 인한 성능 저하가 발생할 수 있기 때문에, 이 값을 true로 하기 보다는 idleConnectionTestPeriod를 이용해서 주기적으로 검사하는 방법이 더 나은 선택이다.  

#### 2.2. JNDI를 이용한 DataSource 설정
<br/>
웹로직이나 JBoss와 같은 JEE 어플리케이션 서버를 사용할 경우, JNDI을 이용해서 DataSource를 설정할 때가 많다. 톰캣이나 제티(Jetty) 등의 웹 콘테이너를 사용하는 경우에도 JNDI로부터 DataSource를 구하도록 설정 가능하다.  

JNDI로부터 DataSource를 가져오고 싶다면, 다음과 같이 &#60;jee:jndi-lookup&#62; 태그를 이용해서 JNDI에 등록된 객체의 이름을 명시하면 된다.  

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:jee="http://www.springframework.org/schema/jee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd
			http://www.springframework.org/schema/jee
			http://www.springframework.org/schema/jee/spring-jee.xsd">

	<jee:jndi-lookup id="dataSource" jndi-name="jdbc/guestbook" resource-ref="true" />

	<bean id="jdbcMessageDao" class="net.madvirus.spring4.chap11.guest.jdbc.JdbcMessageDao">
		<property name="dataSource" ref="dataSource" />
	</bean>
	
</beans>
```

&#60;jee:jndi-lookup&#62; 태그를 사용하기 위해서는 jee 네임스페이스 및 관련 XML 스키마를 등록해야 한다.  

&#60;jee:jndi-lookup&#62; 태그의 jndi-name 속성은 JNDI에서 객체를 검색할 때 사용할 이름을 입력한다. resource-ref 속성의 값이 true일 경우 검색할 이름 앞에 "java:comp/env"가 붙는다. 따라서, 위 설정은 "java:comp/env/jdbc/guestbook"을 사용해서 JNDI에서 객체를 검색하게 된다.  

&#60;jee:jndi-lookup&#62; 태그를 사용하지 않고 다음과 같이 JndiObjectFactoryBean 클래스를 이용해서 JNDI로부터 DataSource를 구하도록 설정할 수 있다.  

```xml
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
	<property name="jndiName" value="jdbc/guestbook" />
	<property name="resourceRef" value="true" />
</bean>
```

&#60;jee:jndi-lookup&#62; 태그는 내부적으로 JndiObjectFactoryBean 클래스를 사용한다.

#### 2.3. DriverManager를 이용한 DataSource 설정
<br/>
로컬 테스트 목적으로 DataSource가 필요한 경우, DriverManager를 이용해서 커넥션을 제공하는 DriverManagerDataSource 클래스를 사용할 수 있다. 설정 방법은 다음과 같다.  

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	<property name="driverClassName" value="com.mysql.jdbc.Driver" />
	<property name="url" value="jdbc:mysql://localhost/test?characterEncoding=euckr" />
	<property name="username" value="root" />
	<property name="password" value="root" />
</bean>
```

DriverManagerDataSource는 커넥션 풀이 아니기 때문에, 실제 운영 환경에서 사용할 경우 성능에 심각한 문제가 발생할 수 있다. 따라서, 테스트 목적으로만 사용할 것을 권한다.

### 3. 스프링 JDBC 지원
<br/>
JDBC를 이용해서 프로그래밍을 할 때 성가신 작업 중의 하나는 동일한 형태의 try-catch-finally 블록을 사용해야 한다는 점이다.  

메서드의 절반 정도가 Connection을 구하고, 익셉션 처리를 하고, Connection 등의 자원을 반환하는 코드이다.  

Connection을 구하고, try-catch-finally로 자원을 관리하는 등의 중복된 코드를 매번 입력하는 것은 중복된 작업이다. 스프링은 이런 중복된 코드를 제거할 수 있도록 해주는 템플릿 클래스를 제공하고 있다. 이 중 JDBC와 관련된 템플릿 클래스는 다음과 같다.  

+ JdbcTemplate : 기본적인 JDBC 템플릿 클래스로서 JDBC를 이용해서 테이터에 대한 접근을 제공한다.
+ NamedParameterJdbcTemplate : PreparedStatement에서 인덱스 기반의 파라미터가 아닌 이름을 가진 파라미터를 사용할 수 있도록 지원하는 템플릿 클래스
+ SimpleJdbcInsert : 데이터 삽입을 위한 인터페이스를 제공해주는 클래스.
+ SimpleJdbcCall : 프로시저 호출을 위한 인터페이스를 제공해주는 클래스.  

#### 3.1. JdbcTemplate 클래스를 이용한 JDBC 프로그래밍
<br/>
org.springframework.jdbc.core.JdbcTemplate 클래스는 SQL 실행을 위한 메서드를 제공하고 있다. 이들 메서드를 사용하면 데이터 조회, 삽입, 수정, 삭제를 위한 SQL 쿼리를 실행할 수 있다. JdbcTemplate 클래스를 사용하려면 다음과 같이 JdbcTemplate 객체를 생성할 때 DataSource를 전달해주면 된다.  

```java
public class 클래스명 {

	...

	private JdbcTemplate jdbcTemplate;

	...

	public 생성자(DataSource dataSource) {
		jdbcTemplate = new JdbcTemplate(dataSource);
	}

	...

	public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}

	...


}
```

위 클래스에 대한 스프링 설정 파일은 다음과 같을 것이다.

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destory-method="close">
	<property name="driverClass" value="com.mysql.jdbc.Driver" />
	<property name="jdbcUrl" value="jdbc:mysql://localhost/guestbook?characterEncoding=utf8" />
	<property name="user" value="spring4" />
	<property name="password" value="spring4" />
</bean>

<bean id="클래스명" class="완전한클래스명">
	<constructor-arg ref="dataSource"/>
</bean>
```

setJdbcTemplate() 메서드를 통해 JdbcTemplate 클래스를 전달받도록 구현할 수도 있다.  

클래스에서 JdbcTemplate을 프로퍼티나 생성자에서 전달받을 경우, 스프링 설정 파일에서는 다음과 같이 JdbcTemplate을 빈 객체로 생성해주면 된다.  

```xml
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
	<property name="dataSource" ref="dataSource" />
</bean>

<bean id="빈식별자" class="완전한클래스명">
	<property name="jdbcTemplate" ref="jdbcTemplate" />
</bean>
```

객체가 내부적으로 JdbcTemplate 객체를 생성하거나 JdbcTemplate 객체를 젅달받았다면, JdbcTemplate 객체를 이용해서 SQL을 실행할 수 있다.  

##### 3.1.1. 조회를 위한 메서드 : query()
<br/>
쿼리 실행 결과를 객체 목록으로 가져올 때에는 PreparedStatement용 SQL 쿼리와 RowMapper를 이용하는 query() 메서드를 이용하면 된다. JdbcTemplate 클래스는 다음과 같은 query() 메서드를 제공하고 있다.   

+ List&#60;T&#62; query(String sql, RowMapper&#60;T&#62; rowMapper)
+ List&#60;T&#62; query(String sql, Object[] args, RowMapper&#60;T&#62; rowMapper)
+ List&#60;T&#62; query(String sql, Object[] args, int[] argTypes, RowMapper&#60;T&#62; rowMapper)  

위 메서드에서 각 파라미터는 다음과 같다.  

+ sql : SQL 쿼리. 위치 기반 파라미터(물음표)를 이용하는 PreparedStatement 용 쿼리를 사용할 수 있다.
+ RowManager : 조회 결과 ResultSet으로부터 데이터를 읽어와 객체를 생성해주는 매퍼.
+ args, argTypes : args는 PerparedStatement를 실행할 때 사용할 파라미터 바인딩 값 목록을, argTypes는 파라미터를 바인딩 할 때 사용할 SQL 타입 목록이다. argTypes에 사용되는 값은 java.sql.Types 클래스에 정의된 값을 사용한다.  

RowMapper는 ResultSet에서 값을 가져와 원하는 타입으로 매핑할 때 사용되며, 다음과 같이 정의되어 있다.

```java
public interface RowMapper<T> {
	T mapRow(ResultSet rs, int rowNum) throws SQLException;
}
```

RowMapper의 mapRow() 메서드는 ResultSet에서 읽어온 값을 이용해서 원하는 타입의 객체를 생성한 뒤 리턴한다. rowNum은 행번호를 의미하며 0부터 시작한다.  

아래 코드는 query() 메서드의 사용 예이다.

```java
public List<Message> select(int start, int size) {
	List<Message> messages = jdbcTemplate.query(
		"select * from guestmessage order by id desc limit ?, ?",
		new Object[] { start, size },
		new RowMapper<Message>() {
			@Override
			public Message mapRow(ResultSet rs, int rowNum) throws SQLException {
				Message m = new Message();
				m.setId(rs.getInt("id"));
				m.setName(rs.getString("name"));
				m.setMessage(rs.getString("message"));
				m.setCreationTime(rs.getTimestamp("creationTime"));
				return m;
			}
		});
	return messages;
}
```

query() 메서드의 두 번째 파라미터는 {start, size}인데, 이 배열의 값은 각각 차례대로 쿼리의 물음표 위치에 설정된다.  

query() 메서드에 RowMapper의 구현 객체를 전달할 때에는 위와 같이 임의 클래스를 주로 사용한다. 하지만, 여러 메서드에서 공통으로 사용되는 코드가 있다면, 다음과 같이 RowMapper 구현 클래스를 별도로 구현해서 코드 중복을 제거할 수 있다.  

```java
public class MessageRowMapper implements RowMapper<Message> {

	@Override
	public Message mapRow(ResultSet rs, int rowNum) throws SQLException {
		...
	}
}
```

지금까지 예로 든 코드는 모두 PreparedStatement의 바인딩 파라미터 값을 지정하기 위해 Object 객체 배열을 사용했다.  

객체 배열을 사용하는 대신 PreparedStatement를 이용해서 파라미터 값을 직접 지정하고 싶다면 org.springframework.jdbc.core.PreparedStatementSetter를 파리미터로 갖는 query() 메서드를 사용하면 된다.  

+ List&#60;T&#62; query(String sql, PreparedStatementSetter setter, RowMapper&#60;T&#62; rowMapper)  

PreparedStatementSetter 인터페이스는 다음과 같이 PreparedStatement를 파라미터로 전달받는 setValues() 메서드를 정의하고 있다.  

```java
public interface PreparedStatementSetter {
	void setValues(PreparedStatement ps) throws SQLException;
}
```

PreparedStatementSetter 구현 객체는 setValues() 메서드에서 PreparedStatement의 바인딩 파라미터 값을 지정하면 된다. 아래 코드는 PreparedStatementSetter를 이용한 query() 메서드의 사용 예이다.  

```java
public List<Message> select(final int start, final int size) {
	List<Message> messages = jdbcTemplate.query(
		"select * from guestmessage order by id desc limit ?, ?",
		new PreparedStatementSetter() {
			@Override
			public void setValues(PreparedStatement ps) throws SQLException {
				ps.setInt(1, start);
				ps.setInt(2, size);
			}
		},
		messageRowMapper);
	return messages;
}
```

SQL 쿼리를 파리미터로 사용하는 방법 대신 직접 PreparedStatement 객체를 생성하고 바인딩 파라미터를 지정해주고 싶다면 org.springframework.jdbc.core.PreparedStatementCreator 타입을 파라미터로 갖는 다음 query() 메서드를 사용하면 된다.  

+ List&#60;T&#62; query(PreparedStatementCreator psc, RowMapper&#60;T&#62; rowMapper)  

PreparedStatementCreator 인터페이스는 다음과 같이 Connection을 파라미터로 전달받고 PreparedStatement를 리턴하는 메서드를 정의하고 있다.  

```java
public interface PreparedStatementCreator {
	PreparedStatement createPreparedStatement(Connection con) throws SQLException;
}
```

PreparedStatementCreator 구현 객체는 createPreparedStatement() 메서드에서 파라미터로 전달받은 Connection을 이용해서 PreparedStatement 객체를 생성하고 바인딩 파라미터 값을 설정한 뒤, 생성한 PreparedStatement 객체를 리턴해주면 된다. 다음은 PreparedStatementCreator 인터페이스를 사용한 query() 메서드의 사용 예이다.  

```java
public List<Message> select(final int start, final int size) {
	List<Message> messages = jdbcTemplate.query(
		new PreparedStatementCreator() {
			@Override
			public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
				PreparedStatement pstmt = con.prepareStatement("select * from guestmessage order by id desc limit ?, ?");
				pstmt.setInt(1, start);
				pstmt.setInt(2, size);
				return pstmt;
			}
		},
		messageRowMapper);
	return messages;
}
```

##### 3.1.2. 조회를 위한 메서드 : queryForList()
<br/>
쿼리 실행 결과로 읽어온 칼럼 개수가 한 개라면 다음의 queryForList() 메서드를 이용해서 데이터를 조회할 수 있다.  

+ List&#60;T&#62; queryForList(String sql, Class&#60;T&#62; elementType)
+ List&#60;T&#62; queryForList(String sql, Object[] args, Class&#60;T&#62; elementType)
+ List&#60;T&#62; queryForList(String sql, Object[] args, int[] argTypes, Class&#60;T&#62; elementType)  

queryForList() 메서드에서 elementType 파라미터는 조회할 데이터 타입을 지정할 때 사용한다. 예를 들어, 조회할 칼럼 타입이 String이라면, 다음과 같이 queryForList() 메서드를 사용하면 된다.  

##### 3.1.3. 조회를 위한 메서드 : queryForObject()
<br/>
쿼리 실행 결과로 구하는 행의 개수가 정확히 한 개라면, queryForObject() 메서드를 사용해서 쿼리 실행 결과를 가져올 수 있다. 사용 가능한 queryForObject() 메서드는 다음과 같다.  

+ T queryForObject(String sql, RowMapper&#60;T&#62; rowMapper)
+ T queryForObject(String sql, Object[] args, RowMapper&#60;T&#62; rowMapper)
+ T queryForObject(String sql, Object[] args, int[] argTypes, RowMapper&#60;T&#62; rowMapper)
+ T queryForObject(String sql, Class&#60;T&#62; requiredType)
+ T queryForObject(String sql, Object[] args, Class&#60;T&#62; requiredType)
+ T queryForObject(String sql, Object[] args, int[] argTypes, Class&#60;T&#62; requiredType)
+ T queryForObject(String sql, RowMapper&#60;T&#62;, Object ... args)  

queryForObject() 메서드에 전달되는 각각의 파라미터는 query() 메서드와 동일하며, 차이점이 있다면 List 대신 한 개의 객체를 리턴한다는 점이다. 주의할 점은, 쿼리 실행 결과로 구한 행 개수가 정확하게 한 개가 아니면-즉 행의 개수가 0이거나 2개 이상인 경우-IncorrectResultSizeDataAccessException 익셉션을 발생시킨다.  

queryForObject() 메서드에서 쿼리 실행 결과로 한 개 이상의 칼럼을 조회하는 경우에는 RowMapper를 사용해서 데이터럴 가져오면 되고, 한 개 칼럼만 조회하는 경우에는 Class를 인자로 받는 queryForObject() 메서드를 사용하면 된다. Class를 인자로 잔달받는 경우, 조회하는 칼럼의 개수가 두 개 이상이면 익셉션을 발생시킨다.  

자바 8 버전의 사용한다면 RowMapper 타입의 임의 객체를 사용하는 대신 람다식을 사용할 수 있다.  

##### 3.1.4. 삽입/수정/삭제를 위한 메서드 : update()
<br/>
INSERT, UPDATE, DELETE 쿼리를 실행할 때에는 update() 메서드를 사용한다. update() 메서드도 query() 메서드와 마찬가지로 인덱스 파라미터를 위한 값을 전달받는 메서드와 그렇지 않은 메서드로 구분된다.  

+ int update(String sql)
+ int update(String sql, Object... args)
+ int update(String sql, Object[] args, int[] argTypes)
+ int update(String sql, PreparedStatementSetter pss)
+ int update(PreparedStatementCreator psc)  

update() 메서드는 쿼리 실행 결과로 변경된 행의 개수를 리턴한다.  

지금까지의 JdbcTemplate의 사용 예제 코드를 보면 try-catch-finally 블록뿐만 아니라 Connection을 구하기 위한 코드가 전혀 포함되지 않을 것을 알 수 있다. 이렇게 템플릿 클래스를 사용하면 성가시고 중복되는 코드를 작성하지 않아도 되며, 코드의 가독성을 향상시킬 수 있다.  

아래 코드는 update() 메서드의 사용 예이다.  

```java
public int delete(int id) {
	return jdbcTemplate.update("delete from guestmessage where id = ?", id);
}
```
##### 3.1.5. KeyHolder를 이용한 자동 생성 키 구하기
<br/>
MySQL의 auto&#95;increment 칼럼과 같이 데이터를 삽입할 때 값이 자동으로 생성되는 키 칼럼이 있다. insert 쿼리를 실행할 때 이렇게 자동 생성되는 키 값을 구하고 싶다면, org.springframework.jdbc.core.PreparedStatementCreator 인터페이스를 파라미터로 갖는 update() 메서드를 이용하면, 자동 생성되는 키 값을 구할 수 있다. 다음 코드는 사용 예이다.  

```java
@Override
public int insert(final Message message) {
	KeyHolder keyHolder = new GeneratedKeyHolder();
	jdbcTemplate.update(new PreparedStatementCreator() {
		@Override
		public PreparedStatement createPreparedStatement(Connection conn) throws SQLException {
			PreparedStatement pstmt = conn.prepareStatement("insert into guestmessage (name, message, creationTime) values (?,?,?)", new String[] {"id"});
			pstmt.setString(1, message.getName());
			pstmt.setString(2, message.getMessage());
			pstmt.setTimestamp(3, new Timestamp(message.getCreationTime().getTime()));
			return pstmt;
		}
	}, keyHolder);
	Number idNum = keyHolder.getKey();
	return idNum.intValues();
}
```

위 코드에서 JdbcTemplate.update() 메서드의 첫 번째 파라미터는 PreparedStatementCreator 객체를 사용한다. 이 객체의 createPreparedStatement() 메서드는 파라미터로 전달받은 Connection 객체의 prepareStatement() 메서드를 이용해서 PreparedStatement 객체를 생성하는데, 이때 preparedStatement() 메서드의 두 번째 파라미터로 자동 생성되는 키 칼럼을 지정한다.  

JdbcTemplate.update() 메서드의 두 번째 파라미터는 읽어온 키 값을 보관할 KeyHolder 객체를 전달한다. 생성된 키 값을 구할 때에는 org,springframework.jdbc.support.KeyHolder 인터페이스에 정의된 getKey() 메서드를 사용한다. 이 메서드는 java.lang.Number 타입을 리턴하므로 Number의 intValue()나 longValue() 메서드를 이용해서 생성된 키 값을 구할 수 있다.  

PreparedStatementCreator 인터페이스는 한 개의 메서드만 갖고 있으므로, 자바 8의 람다식을 이용해서 표현할 수 있다.  

당연한 얘기지만 스프링이 제공하는 인터페이스 중에서 한 개 메서드만 정의하고 있는 모든 인터페이스는 자바 8에서 람다식으로 표현할 수 있다.  


##### 3.1.6. ConnectionCallback을 이용한 Connection 사용
<br/>
Connection 객체에 직접 접근해야 한다면, execute() 메서드를 사용하면 된다. execute() 메서드는 파라미터로 전달받은 ConnectionCallback 인터페이스 구현 객체의 doInConnection() 메서드를 호출하는데, 이때 Connection을 doInConnection()에 인자로 전달한다. ConnectionCallback 인터페이스는 다음과 같이 정의되어 있다.  

```java
public interface ConnectionCallback<T> {
	T doInConnection(Connection con) throws SQLException, DataAccessException;
}
```

ConnectionCallback을 파라미터로 갖는 JdbcTemplate의 execute() 메서드는 파라미터로 전달받은 ConnectionCallback 객체의 doInConnection() 메서드가 리턴한 객체를 리턴한다.  

아래 코드는 ConnectionCallback 인터페이스의 사용 예를 보여주고 있다.  

```java
public int count() {
	return jdbcTemplate.execute(new ConnectionCallback<Integer>() {
		@Override
		public Integer doInConnection(Connection con) throws SQLException, DateAccessException {
			Statement stmt = null;
			ResultSet rs = null;
			try {
				stmt = con.createStatement();
				rs = stmt.executeQuery("select count(*) from GUESTBOOK_MESSAGE");
				rs.next();
				return rs,getInt(1);
			} finally {
				JdbcUtils.closeResultSet(rs);
				JdbcUtils.closeStatement(stmt);
			}
		}
	}
}
```

ConnectionCallback 구현 객체의 doInConnection() 메서드는 파라미터로 전달받은 Connection을 이용해서 알맞은 작업을 수행하면 된다. 커넥션 생성과 종료는 JdbcTemplate이 처리하므로 doInConnection() 메서드에서는 Connection을 종료할 필요가 없다.  

#### 3.2. NamedParameterJdbcTemplate 클래스를 이용한 JDBC 프로그래밍
<br/>
org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate 클래스는 JdbcTemplate 클래스와 동일한 기능을 제공하는데, 차이점이 있다면 인덱스 기반의 파라미터가 아니라 이름 기반의 파라미터를 설정할 수 있도록 해준다는 점이다. 예를 들어, 인덱스 기반의 파라미터인 물음표를 사용하지 않고 다음과 같이 이름 기반의 파라미터를 쿼리에서 사용할 수 있도록 지원한다.  

```java
select * from guestmessage order by id desc limit :start, :size;
```

NamedParameterJdbcTemplate 클래스는 다음과 같이 생성자를 이용해서 DataSource를 전달받는다.  

```java
public class 클래스명

	...

	private NamedParameterJdbcTemplate template;

	...

	public 생성자(DataSource dataSource) {
		this.template = new NamedParameterJdbcTemplate(dataSource);
	}

	...

}
```

NamedParameterJdbcTemplate 클래스는 JdbcTemplate 클래스와 동일한 이름의 메서드를 제공한다. 차이점이 있다면 인덱스 기반의 파라미터가 아니라 이름 기반의 파라미터 값을 설정하기 위해 Map이나 SqlParameterSource을 전달받는다는 것이다.  

##### 3.2.1. Map을 이용한 파라미터 값 설정 메서드
<br/>
Map 기반의 메서드는 Object 배열이 아닌 Map을 이용해서 이름을 가진 파라미터 값을 설정한다. 아래 코드는 Map 기반 메서드와 이름 기반의 파라미터를 갖는 SQL 쿼리 사용 예를 보여주고 있다.  

```java
@Override
public List<Message> select(int start, int size) {
	Map<String, Object> params = new HahsMap<>();
	params.put("start", start);
	params.put("size", size);
	List<Message> messages = template.query("select * from guestmessage order by id desc limit :start, :size", params, new MessageRowMapper());
	return messages;
}
```

위 코드에서 보듯이 SQL 쿼리는 인덱스 기반 파라미터 대신에 이름 기반의 파라미터를 사용하고 있으며, Map에서 동일한 이름을 갖는 키의 값이 파라미터의 값으로 설정된다.  

NamedParameterJdbcTemplate 클래스가 제공하는 Map 기반 메서드는 다음곽 같다.  

+ List&#60;T&#62; query(String sql, Map&#60;String, ?&#62; paramMap, RowMapper&#60;T&#62; rowMapper)
+ List&#60;T&#62; queryForList(String sql, Map&#60;String, ?&#62; paramMap, Class&#60;T&#62; elementType)
+ T queryForObject(String sql, Map&#60;String, ?&#62; paramMap, RowMapper&#60;T&#62; rowMapper)
+ T queryForObject(String sql, Map&#60;String, ?&#62; paramMap, Class&#60;T&#62; requiredType)
+ int update(String sql, Map&#60;String, ?&#62; paramMap)
+ int update(String sql, Map&#60;String, ?&#62; paramMap, KeyHolder keyHolder)  

이름 기반의 파라미터를 갖지 않은 쿼리를 실행하는 경우에는 아무 값도 갖지 않는 Map 객체를 사용하면 된다.  

```java
@Override
public int count() {
	return template.queryForObject("select count(*) from guestmessage", Collections.<String, Object> emptyMap(), Integer.class);
}
```

##### 3.2.2. SqlParameterSource를 이용한 파라미터 값 설정 메서드
<br/>
Map 대신에 org.springframework.jdbc.core.namedparam.SqlParameterSource 인터페이스를 이용해서 파라미터 값을 설정할 수도 있다. 다음 메서드는 SqlParameterSource를 인자로 전달받는 메서드의 목록이다.  

+ List&#60;T&#62; query(String sql, SqlParameterSource paramSource, RowMapper&#60;T&#62; rowMapper)
+ List&#60;T&#62; queryForObject(String sql, SqlParameterSource paramSource, Class&#60;T&#62; elementType)
+ T queryForObject(String sql, SqlParameterSource paramSource, RowMapper&#60;T&#62; rowMapper)
+ T queryForObject(String sql, SqlParameterSource paramSource, Class&#60;T&#62; requiredType)
+ int update(String sql, SqlParameterSource paramSource)
+ int update(String sql, SqlParameterSource paramSource, KeyHolder keyHolder)  

SqlParameterSource는 인터페이스이기 때문에 실제로 사용할 때에네는 SqlParameterSource 인터페이스를 구현한 클래스를 사용해서 파라미터 값을 전달해주어야 한다. 스프링은 다음과 같은 두 개의 SqlParameterSource 구현 클래스를 제공하고 있다.  

+ org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource
+ org.springframework.jdbc.core.namedparam.MapSqlParameterSource  

BeanPropertySqlParameterSource 클래스는 동일한 이름을 갖는 자바 객체의 프로퍼티 값을 이용해서 파라미터 값을 설정한다. 아래 코드는 BeanPropertySqlParameterSource 클래스의 사용 예를 보여주고 있다.  

```java
@Override
public int insert(Message message) {
	SqlParameterSource paramSource = new BeanPropertySqlParameterSource(message);
	template.update("insert into guestmessage (name, message, creationTime)" + "values (:name, :message, :creationTime)", paramSource, keyHolder);
	Number idNum = keyHolder.getKey();
	return idNum.intValue();
}
```

위 코드에서 쿼리에 포함된 name, message, createTime 파라미터는 각각 message 객체의 name 프로퍼티, message 프로퍼티, 그리고 createTime 프로퍼티 값을 이용해서 설정된다.  

MapSqlParameterSource 클래스는 Map과 비슷하게 &#60;이름, 값&#62; 쌍을 이용해서 파라미터의 값을 설정한다. MapSqlParameterSource 객체를 생성한 뒤, addValue() 메서드를 이용해서 파라미터 이름과 값을 설정해주면 된다. 아래 코드는 MapSqlParameterSource 클래스를 사용해서 파라미터 값을 설정하는 예를 보여주고 있다.  

```java
public int delete(int id) {
	MqpSqlParameterSource paramSource = new MapSqlParameterSource();
	paramSource.addValue("id", id);
	return template.update("delete from geustmessage where id = :id", paramSource);
}
```

##### 3.2.3. JdbcTemplate의 메서드 사용하기
<br/>
NamedParameterJdbcTemplate은 내부적으로 JdbcTemplate을 사용하고 있고, getJdbcOperations() 메서드를 이용하면 사용중인 JdbcTemplate 객체를 구할 수 있다. 따라서, JdbcTemplate의 메서드를 사용하고 싶다면, JdbcTemplate 객체를 생성할 필요 없이 getJdbcOperations() 메서드가 리턴한 JdbcTemplate 객체를 사용하면 된다.  

```java
public class 클래스명 {

	...

	private NamedParameterJdbcTemplate template;

	...

	public 생성자(DateSource dataSource) {
		this.template = new NamedParameterJdbcTemplate(dataSource);
	}

	...

	@Override
	public int counts() {
		return template.getJdbcOperations().queryForObject("select count(*) from guestmessage", Integer.class);
	}

	...

}
```

NamedParameterJdbcTemplate의 getJdbcOperations() 메서드가 리턴하는 실제 타입은 org,springframework.jdbc.core.JdbcOperations 인터페이스다. JdbcTemplate은 이 인터페이스를 구현하고 있으며, 앞서 JdbcTemplate 클래스를 설명할 때 언급한 메서드들은 모두 JdbcOperations 인터페이스에 정의되어 있는 메서드들이다.  

#### 3.3. SimpleJdbcInsert 클래스를 이용한 데이터 삽입
<br/>
SimpleJdbcInsert 클래스는 쿼리를 사용하지 않고 데이터를 삽입할 수 있도록 해주는 클래스이다. SimpleJdbcInsert 클래스를 이용하는 가장 간단한 방법은 다음과 같다.  

```java
public class 클래스명 {

	...

	private SimpleJdbcInsert simpleInsert;

	...


	public 생성자(DataSource dataSource) {
		...
		simpleInsert = new SimpleJdbcInsert(dataSource);
		simpleInsert.withTabelName("guestmessage");
	}

	...

	@Override
	public int insert(Message message) {
		Map<String, Object> values = new HashMap<>();
		values.put("NAME", message.getName());
		values.put("message", message.getMessage());
		values.put("creationTime", new Timestamp(message.getCreationTime().getTime()));
		int insertedCount = simpleInsert.execute(values);
		...
	}

	...

}
```

위 코드에서 SimpleJdbcInsert 클래스의 withTabelName() 메서드는 데이터를 삽입할 테이블의 이름을 지정한다. execute(Map&#60;String, Object&#62;) 메서드는 Map의 키를 칼럼명으로 사용하고, 값을 칼럼에 삽입할 데이터로 사용하는 SQL 쿼리를 실행한다. 즉, 위 코드가 실행하는 SQL 쿼리는 다음과 같다.  

```sql
INSERT INTO guestmessage (id, name, message, creationTime) VALUES (?, ?, ?, ?)

```

위 쿼리를 생성할 때 사용할 칼럼 목록은 지정한 테이블의 메타정보에서 읽어온다. 만약 DB가 제공하는 메타정보가 아닌 직접 사용할 칼럼 이름을 지정하고 싶다면, 아래 코드처럼 usingColumns() 메서드를 사용하면 된다.  

```java
public 생성자(DateSource dataSource) {
	...
	simpleInsert = new SimpleJdbcInsert(dataSource);
	simpleInsert.withTableName("geustmessage");
	simpleInsert.usingColumns("name", "message", "creationTime");
}
```

usingColumns() 메서드를 사용하면, 지정한 칼럼에 대해서만 값을 삽입하기 때문에, usingColumns() 메서드에서 지정하지 않은 칼럼에 대해서는 값이 삽입되지 않는다.  

SimpleJdbcInsert 클래스가 제공하는 설정 메서드는 메서드 체이닝(method chaining)을 지원한다.  

#### 3.3.1. execute() 메서드를 이용한 데이터 삽입
<br/>
SimpleJdbcInsert 클래스를 이용해서 데이터를 삽입할 때에는 execute() 메서드를 사용하면 된다. SimpleJdbcInsert 클래스는 다음과 같은 execute() 메서드를 제공하고 있으며, execute() 메서드의 리턴 값은 쿼리 실행 결과로 영향을 받은 행의 개수이다.  

+ int execute(Map&#60;String, Object&#62; args)
+ int execute(SqlParameterSource parameterSource)  

Map을 전달하는 경우 대소문자를 구분하지 않고 칼럼명과 Map의 키 값이 일치하는 지 여부를 검사한다. SimpleJdbcInsert.usingColumns() 메서드에서 지정한 칼럼명과 Map의 키는 소문자로 변환해서 일치할 경우 매칭된다.  

SqlParameterSource를 사용해서 execute() 메서드를 실행하는 경우, 다음 규칙에 따라서 칼럼명의 일치 여부를 검사한다.  

+ 지정한 칼럼명과 동일한 이름을 갖는 파라미터 값이 설정되어 있는지 검사한다.
+ '&#95;'이 포함된 경우 '&#95;'를 제외한 나머지 문자열과 일치하는 파라미터 값이 설정되어 있는지 검사한다.  

다음 코드는 SqlParameterSource를 사용한 코드의 예이다.  

```java
public int insert(Message message) {
	BeanPropertySqlParameterSource paramSource = new BeanPropertySqlParameterSource(message);
	int insertedCount = simpleInsert.execute(paramSource);
	...
}
```

실제로 Map, SqlParameterSource와 칼럼명 사이의 매핑 처리는 TableMetaDataContext 클래스를 통해서 처리된다.  

#### 3.3.2. executeAndReturnKey() 메서드를 이용한 데이터 삽입 및 자동 생성 키 조회
<br/>
MySQL의 auto&#95;increment 칼럼은 데이터가 삽입될 때마다 자동으로 증가된 값이 생성되는 칼럼이다. 이렇게 데이터 삽입시 자동으로 생성되는 키 칼럼을 구하고 싶을 때에는 executeAndReturnKey() 메서드를 사용하면 된다. executeAndReturnKey() 메서드는 다음과 같이 네 개가 존재한다.  

+ Number executeAndReturnKey(Map&#60;String, Object&#62; args)
+ Number executeAndReturnKey(SqlParameterSource paramSource)
+ KeyHolder executeAndReturnKey(Map&#60;String, Object&#62; args)
+ KeyHolder executeAndReturnKey(SqlParameterSource paramSource)  

executeAndReturnKey() 메서드를 사용하려면 usingGenerationKeyColumns() 메서드를 이용해서 자동 생성되는 키 칼럼을 지정해주어야 한다.  

```java
insertMessage.withTableName("GUESTBOOK_MESSAGE")
	.usingGeneratedKeyColumns("MESSAGE_ID")
	.usingColumns("GUEST_NAME", "MESSAGE", "REGISTRY_DATE");
```

usingGerneratedKeyColumns() 메서드로 자동 생성 키 칼럼을 지정했다면, executeAndReturnKey() 메서드를 이용해서 생성되는 키 값을 구할 수 있다.  

```java
public 생성자(DataSource dataSource) {
	simpleInsert = new SimpleJdbcInsert(dataSource);
	simpleInsert.withTableName("guestmessage")
		.usingColumns("name", "message", "creationTime")
		.setGeneratedKeyName("id");
}

@Override
public int insert(Message message) {
	BeanPropertySqlParameterSource paramSource = new BeanPropertySqlParameterSource(message);
	Number genKey = simpleInsert.executeAndReturnKey(paramSource);
	return genKey.intValue();
}
```

Number가 아니라 KeyHolder를 리턴하는 executeAndReturnKeyHolder() 메서드를 사용하면 KeyHolder로부터 키 값을 구하면 된다.  

```java
KeyHolder keyHolder = simpleInsert.executeAndReturnKeyHolder(paramSource);
return keyHolder.getKey().intValue();
```
