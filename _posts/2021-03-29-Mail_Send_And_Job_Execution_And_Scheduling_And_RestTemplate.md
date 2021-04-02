---
title: 메일 발송, 작업 실행과 스케줄링, RestTemplate
categories:
- Spring
feature_text: |
  ## 메일 발송, 작업 실행과 스케줄링, RestTemplate
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 1. 메일 발송
<br/>
#### 1.1. MailSender와 JavaMailSender를 이용한 메일 발송
<br/>
스프링은 메일 발송 기능을 위한 MailSender 인터페이스를 제공하고 있으며, 다음과 같이 정의되어 있다.  

```java
public interface MailSender {
	void send(SimpleMailMessage simpleMessage) throws MailException;
	void send(SimpleMailMessage[] simpleMessage) throws MailException;
}
```

MailSender 인터페이스는 SimpleMailMessage를 전달받아 메일을 발송하는 기능을 정의하고 있다. SimpleMailMessage는 메일 제목과 단순 텍스트 내용으로 구성된 메일을 발송할 때에 사용된다.  

MailSender 인터페이스를 상속받은 JavaMailSender는 Java Mail API의 MimeMessage를 이용해서 메일을 발송하는 기능을 추가적으로 정의하고 있다. JavaMailSender 인터페이스는 다음과 같이 정의되어 있다.  

```java
public interface JavaMailSender extends MailSender {
	MimeMessage createMimeMessage();
	MimeMessage createMimeMessage(InputStream contentStream) throws MailException;
	void send(MimeMessage mimeMessage) throws MailException;
	void send(MimeMessage[] mimeMessage) throws MailException;
	void send(MimeMessagePreparator mimeMessagePreparator) throws MailException;
	void send(MimeMessagePreparator[] mimeMessagePreparators) throws MailException;
}
```

MailSender 인터페이스와 JavaMailSender 인터페이스의 메서드들이 발생하는 MailException은 RuntimeException이므로, 익셉션 처리가 필요한 경우에만 catch 블록에서 처리해주면 된다.  

스프링은 JavaMailSender 인터페이스의 구현체로 JavaMailSenderImpl 클래스를 제공하고 있으므로, 이 클래스를 이용해서 빈 설정을 하게 된다.  

##### 1.1.1. JavaMailSender 빈 설정
<br/>
JavaMailSender를 이용하려면 먼저 메이븐 설정에 다음의 의존을 추가해야 한다.  

```xml
<dependencies>
	<!-- 메일 발송 지원 기능 포함 -->
	<dependency>
		<groupId>org,springframework</groupId>
		<artifactId>spring=context-support</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
	<!-- Java Mail API -->
	<dependency>
		<groupId>javax.mail</groupId>
		<artifactId>mail</artifactId>
		<version>1.4.7</version>
	</dependency>
</dependencies>
```

JavaMailSenderImpl 클래스는 Java Mail API를 이용해서 메일을 발송하며 기본적으로 SMTP 프로토콜을 사용한다. SMTP 서버를 이용해서 메일을 발송하므로 SMTP 서버 주소와 포트 번호를 필요로 한다. 이 두 정보는 각각 host 프로퍼티와 port 프로퍼티를 이용해서 입력 받는다.  

```xml
<bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
	<property name="host" value="mail.host.com" />
	<property name="port" value="25" />
	<property name="defaultEncoding" value="utf-8" />
	<property name="username" value="system" />
	<property name="password" value="syspass" />
	<property name="defaultEncoding" value="utf-8" />
</bean>

<bean id="someNotifier" class="...">
	<property name="mailSender" ref="mailSender" />
</bean>
```

port 프로퍼티의 기본 값은 25이므로, 포트 번호가 25가 아닌 경우에만 port 속성을 설정하면 된다. defaultEncoding 포로퍼티는 발송될 메일의 기본 인코딩을 설정한다. JavaMailSenderImpl은 내부적으로 Java Mail API의 MimeMessage를 이용하기 때문에, 인코딩을 지정하지 않은 SimpleMailMessage를 이용할 경우에 defaultEncoding 프로퍼티의 속성 값을 알맞게 입력해주는 것이 좋다.  

만약, SMTP 서버에서 인증을 필요로 한다면 username 프로퍼티와 password 포로퍼티를 이용해서 인증에 사용되는 정보를 입력한다.  

##### 1.1.2. SimpleMailMessage를 이용한 메일 발송
<br/>
단순히 텍스트로만 구성된 메일 메시지를 생성할 때에는 SimpleMailMessage를 이용한다. 메일 내용을 구성하는데 사용되는 SimpleMailMessage의 메서드는 다음과 같다.  

+ setFrom(String from)  
발신자 설정
+ setReplyTo(String replyTo)  
응답 주소 설정
+ setTo(String to)  
수신자 설정
+ setTo(String[] to)  
수신자 목록 설정
+ setCc(String cc)  
참조자 설정
+ setCc(String[] cc)  
참조자 목록 설정
+ setBcc(String bcc)  
숨은 참조자 설정
+ setBcc(String[] bcc)  
숨은 참조자 목록 설정
+ setSentDate(Date sentDate)  
메일 발송일 설정
+ setSubject(String subject)  
메일 제목(주제) 설정
+ setText(String text)  
메일 내용 설정  

아래 코드는 SimpleMailMessage를 이용해서 메일 메시지를 생성한 뒤 MailSender를 이용해서 메일을 발송하는 예제 코드이다.  

```java
public class SimpleRegistrationNotifier impleements RegistrationNotifier {

	private MailSender mailSender;

	public void setMailSender(MailSender mailSender) {
		this.mailSender = mailSender;
	}

	@Override
	public void sendMail(Member member) {
		SimpleMailMessage message = new SimpleMailMessage();
		message.setSubject("[Simple] 회원 가입 안내");
		message.setFrom("no-reply@madvirus.net");
		message.setText("회원 가입을 환영입니다.");
		message.setTo(memeber.getEmail());
		try {
			mailSender.send(message);
		} catch (MailException ex) {
			ex.printStackTrace(); // 알맞게 익셉션 처리
		}

	}
}
```

##### 1.1.3. SimpleMailMessage의 재사용
<br/>
SimpleMailMessage 클래스는 다른 SimpleMailMessage 클래스로부터 내용을 복사해서 사용하는 기능을 제공하고 있다. 아래 코드와 같이 생성자를 통해 SimpleMailMessage 객체를 전달하면, 설정 정보를 모두 복사한다.  

```java
SimpleMailMessage anotherMailMessage = new SimpleMailMessage();
anotherMailMessage.setFrom(...);
...
SimpleMailMessage message = new SimpleMailMessage(anotherMailMessage);
```

이 기능을 이용하면 SimpleMailMessage를 스프링 설정 파일에서 설정하고 필요로 하는 빈에서 재사용할 수 있게 된다. 아래 코드는 설정 예를 보여 주고 있다.  

```xml
<bean id="templateMailMessage" class="org.springframework.mail.SimpleMailMessage">
	<property name="from" value="no-reply@madvirus.net" />
	<property name="subject value="[템플릿]회원 가입 안내" />
	<property name="text" value="[템플릿]회원 가입을 환영합니다." />
</bean>

<bean id="simpleNotifier2" class="net.madvrius.spring4.chap17.mail.SimpleRegistrationNotifier2">
	<property name="mailSender" ref="mailSender" />
	<property name="templateMailMessage ref="templateMailMessage" />
</bean>
```

templateMailMessage 빈을 전달받은 클래스에서는 templateMailMessage 객체를 이용해서 SimpleMailMessage 객체를 생성해주고 추가적으로 필요한 정보만 설정하면 되므로 메일 메시지를 생성하는 코드가 좀 더 간단해진다. 다음 코드는 재사용의 예를 보여주고 있다.  

```java
public class SimpleRegistrationNotifier2 implements RegistrationNotifier {
	private MailSender mailSender;
	private SimpleMailMessage templateMailMessage;

	public void setMailSender(MailSender mailSender) {
		this.mailSender = mailSender;
	}

	public void setTemplateMailMessage(SimpleMailMessage templateMailMessage) {
		this.templateMailMessage = templateMailMessage;
	}

	@Override
	public void sendMail(Mmember member) {
		SimpleMailMessage message = new SimpleMailMessage(templateMailMessage);
		message.setTo(member.getEmail()); // 나머지 값은 템플릿의 값을 그대로 사용
		try {
			mailSender.send(message);
		} catch (MailException ex) {
			ex.printStackTrace(); // 실제는 알맞게 익셉션 처리
		}

	}
}
```

##### 1.1.4. Java Mail API의 MimeMessage를 이용한 메시지 생성  
<br/>
SimpleMailMessage는 단순히 텍스트 기반의 메시지를 발송하는 데에는 적합하지만, 메일 내용이 HTML로 구성되어 있다던가, 파일을 첨부해야 하는 경우에는 사용할 수 없다. 이런 기능이 필요할 경우에는 Java Mail API가 제공하는 MimeMessage를 직접 이용해서 메일을 발송해주어야 한다.  

JavaMailSender 인터페이스는 MimeMessage 객체를 생성해주는 createMimeMessage() 메서드를 제공하고 있으며, 이 메서드가 리턴한 MimeMessage 객체를 이용해서 메시지를 구성한 뒤 메일을 발송하면 된다. 다음은 MimeMessage를 이용해서 메일 메시지를 구성한 뒤 JavaMailSender를 이용해서 메일을 발송하는 예제 코드이다.  

```java
public class MimeRegistrationNotifier implements RegistrationNotifier {
	private JavaMailSender mailSender;

	public void setMailSender(JavaMailSender mailSender) {
		this.mailSender = mailSender;
	}

	@Override
	public void sendMail(Member member) {
		MimeMessage message = mailSender.createMimeMessage();
		try {
			message.setSubject("[Mime] 회원 가입 안내", "utf-8");
			String htmlContext = "<strong>안녕하세요</strong>, 반갑습니다.";
			message.setText(htmlContent, "utf-8", "html");
			message.setFrom(new InternetAddress("no-reply@madvirus.net"));
			message.addRecipient(RecipientType.TO, new InternetAddress(member.getEmail()) );
		} catch (MessagingException ex) {
			// 실제로 익셉션 발생 내지 로그 남김
			ex.printStackTrace();
			return;
		}
		try {
			mailSender.send(message);
		} catch (MailException e) {
			// 실제로 익셉션 발생 내지 로그 남김
			e.printStackTrace();
		}
	}

}
```

#### 1.2. MimeMessageHelper를 이용한 파일 첨부
<br/>
Java Mail API의 MimeMessage를 이용하면 파일을 첨부할 수 있긴 하지만, 처리해주어야 할 코드가 많기 때문에 꽤 성가신데, 스프링이 제공하는 MimeMessageHelper 클래스를 사용하면 파일 첨부를 비교적 간단한 코드로 처리할 수 있다.  

MimeMessageHelper 클래스는 MimeMessage 클래스를 직접 사용하지 않고 MimeMessageHelper 클래스가 제공하는 기능을 이용해서 메일 메시지를 생성할 수 있도록 돕는 클래스이다. MimeMessageHelper 클래스는 setSubject(), setTo(), setFrom(), setText() 등의 메서드를 제공하고 있어 메일 내용을 구성할 수 있으며, addAttachment() 메서드를 사용해서 파일을 첨부할 수 있다.  

##### 1.2.1. MimeMessageHelper를 이용한 첨부 파일 추가
<br/>
다음 코드는 MimeMessageHelper 클래스를 이용하여 파일 첨부된 메일을 발송하는 방법을 보여 주고 있다.  

```java
public class MimeAttachmentNotifier implements RegistrationNotifier {

	private JavaMailSender mailSender;

	public void setMailSender(JavaMailSender mailSender) {
		this.mailSender = mailSender;
	}

	@Override
	public void sendMail(Member member) {
		MimeMessage message = mailSender.createMimeMessage();
		try {
			MimeMessageHelper messageHelper = new MimeMessageHelper(message, "utf-8");
			messageHelper.setSubject("회원 가입 안내 [Attachment]");
			String htmlContent = "<strong>안녕하세요</strong>, 반갑습니다.";
			messageHelper.setText(htmlCotent, true);
			messageHelper.setFrom("no-reply@madvirus.net", "운영자");
			messageHelper.setTo(new InternetAddress(member.getEmail(), "utf-8"));

			DataSource dataSource = new FileDataSource("안내문.docx");
			messageHelper.addAttachment(MimeUtility.encodeText("안내문.docx", "utf-8", "8"), dataSource);
			mailSender.send(message);
		} catch (Exception e) {
			e.printStackTrace(); // 실제로는 알맞게 익셉션 처리
		}

	}

}
```

MimeMessageHelper 객체를 생성할 때에는 메일 메시지를 구성할 때 사용되는 MimeMessage 객체를 전달한다. MimeMessageHelper 생성자의 두 번째 파라미터는 MultiPart 여부를 설정하며 true로 지정해야 첨부 파일을 추가할 수 있다.  

```java
MimeMessage message = mailSender.createMimeMessage();
MimeMessageHelper messageHelper = new MimeMessageHelper(message, true /* MultiPart 여부 */, "utf-8");
```

파일 첨부를 하기 위해서는 먼저 파일 정보를 제공할 javax.activation.DataSource를 생성해주어야 한다. Activation API는 파일을 데이터로 사용하는 FileDataSource 클래스를 제공하고 있으므로 이 클래스를 사용하면 파일을 메일에 첨부할 수 있다.  

FileDataSource를 만들었다면 MimeMessageHelper.addAttachment() 메서드를 이용해서 파일을 첨부한다.  

```java
DataSource dataSource = new FileDataSource("c:\\안내문.doc");
messageHelper.addAttachment(MimeUtility.encodeText("안내문.doc", "utf-8", "B"), dataSource);
```

MimeUtility 클래스는 Java Mail API에서 제공하는 클래스로서 ASCII 이외의 문자로 구성된 문장을 알맞게 인코딩할 때 사용된다.  

MimeMessageHelper가 생성한 메일 메시지는 생성자를 통해 전달받은 MimeMessage 객체에 전달되므로, MimeMessageHelper로 메일 내용을 구성한 후에는 JavaMailSender의 send() 메서드에 MimeMessage 객체를 전달해서 메일을 발송하면 된다.  

##### 1.2.2. MimeMessageHelper를 이용한 인라인 자원 추가
<br/>
MimeMessageHelper.addInline() 메서드를 사용하면 메일 내용에 이미지나 파일 등을 삽입할 수 있다. 다음 코드는 사용 예이다.  

```java
public class MimeInlineNotifier implements RegistrationNotifier {
	private JavaMailSender mailSender;

	public void setMailSender(JavaMailSender mailSender) {
		this.mailSender = mailSender;
	}

	@Ovrride
	public void sendMail(Member member) {
		MimeMessage message = mailSender.createMimeMessage();
		try{
			MimeMessageHelper messageHelper = new MimeMessageHelper(message, true, "utf-8");
			messageHelper.setSubject("[Inline] 회원 가입 안내");
			String htmlContent = "<strong>안녕하세요</strong>, 반갑습니다." + "<img src='cid:signature' />";
			messageHelper.setText(htmlContent, true);
			messageHelper.setFrom("no-reply@madvirus.net", "운영자");
			messageHelper.setTo(new InternetAddress(member.getEmail(), "utf-8"));

			messageHelper.addInline("signature", new FileDataSource("sign.jpg"));
			mailSender.send(message);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}
```

### 2. 작업 실행과 스케줄링
<br/>
스프링은 작업 실행 및 작업 스케줄링을 위한 인터페이스와 구현 클래스를 제공하고 있으며, 이를 통해 간단한 설정만으로 스케줄링, 비동기 처리 등을 할 수 있게 해주고 있다.  

작업 실행 기능을 사용하려면 다음과 같이 spring-context-support 모듈을 의존에 추가해주면 된다.  

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context-support</artifactId>
		<version>4.0.4.RELEASE</version>
	</dependency>
</dependencies>
```

#### 2.1. &#60;task:executor&#62;를 이용한 작업 실행
<br/>
작업 실행과 관련된 핵심 인터페이스는 TaskExecutor이다. TaskExecutor 인터페이스 및 하위 인터페이스는 작업 실행과 관련된 인터페이스를 제공하고 있다.  

+ &#60;task:executor&#62; 태그를 이용한 TaskExecutor 빈 설정
+ TaskExecutor 빈 객체에 Runnable 구현 객체를 전달해서 작업 실행  

##### 2.1.1. &#60;task:executor&#62;를 이용한 TaskExecutor 빈 설정
<br/>
&#60;task:executor&#62; 태그는 TaskExecutor 빈을 생성할 때 사용되는 태그로서, 다음과 같이 사용된다.  

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:task="http://www.springframework.org/schema/task"
	xmlns:xsi:"http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd
			http://www.springframework.org/schema/task
			http://www.springframework.org/schema/task/spring-task.xsd">

	<task:executor id="executor" keep-alive="10" pool-size="10-20" queue-capacity="10" rejection-policy="ABORT" />
</beans>
```

&#60;task:executor&#62; 태그를 사용하기 위해서는 task 네임스페이스를 설정해주어야 하며, id 속성을 이용해서 빈 식별 값을 설정한다. &#60;task:executor&#62; 태그의 주요 속성은 다음과 같다.  

<table>
	<thead>
		<tr>
			<td>속성</td>
			<td>설명</td>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>id</td>
			<td>생성할 빈의 식별 값을 지정한다.</td>
		</tr>
		<tr>
			<td>pool-size</td>
			<td>쓰레드 풀의 개수를 지정한다. '최소크기-최대크기'으로 지정하거나 '개수'를 지정한다. 예를 들어, 5-20으로 지정하거나 15와 같이 지정할 수 있다. '개수' 형식으로 지정할 경우, 최소 크기와 최대 개수가 동일한 값을 갖는다.<br/>기본 값은 최소 크기 1이고, 최대 크기는 Integer.MAX&#95;VALUE이다.</td>
		</tr>
		<tr>
			<td>queue-capacity</td>
			<td>풀의 쓰레드가 모두 작업을 실행중인 경우 큐에서 대기할 수 있는 작업의 개수를 지정한다. 기본값은 Integer.MAX&#95;VALUE이다.</td>
		</tr>
		<tr>
			<td>keep-alive</td>
			<td>풀에 있는 쓰레드의 최대 유휴 시간을 지정한다. 이 시간 동안 새로운 작업을 실행하지 않은 쓰레드는 풀에서 제거된다. 단위는 초이다.</td>
		</tr>
		<tr>
			<td>rejection-policy</td>
			<td>
				큐가 다 차서 더 이상 작업을 받을 수 없을 때 작업을 어떻게 처리할지를 결정한다.<br/><br/>
				<li>ABORT : 작업을 거부하고 익셉션을 발생시킨다.</li>
				<li>CALLER&#95;RUNS : 호출한 쓰레드를 이용해서 실행한다.</li>
				<li>DISCARD : 작업을 실행하지 않고 무시한다.</li>
				<li>DISCARD&#95;OLDEST : 큐의 헤드에서 하나를 제거하고 작업을 추가한다.</li>
				<br/>
				기본 값은 ABORT이다.
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

&#60;task:executor&#62; 태그를 사용하면 내부적으로 java.util.concurrent.ThreadPoolExecutor를 사용해서 작업을 실행하게 되는데, ThreadPoolExecutor는 다음과 같은 규칙을 이용해서 풀의 크기를 관리한다.  

+ 풀에 최소 크기보다 작은 개수의 쓰레드가 존재할 경우, 쓰레드를 새롭게 생성한다.  
+ 풀에 최소 크기와 같거나 많은 개수의 쓰레드가 존재하고 큐에 여분이 남아 있는 경우, 작업을 큐에 저장한다.  
+ 작업을 큐에 보관할 수 없을 경우, 풀에 최대 크기보다 작은 개수의 쓰레드가 존재할 경우 쓰레드를 새롭게 생성한다. 그렇지 않을 경우 작업을 거부한다.  

위 규칙에 따르면 큐의 개수가 Integer.MAX&#95;VALUE인 경우, 큐에 객체를 저장하는 과정에서 메모리 부족 현상이 발생할 수도 있다. 또한, 쓰레드의 최대 개수가 Integer.MAX&#95;VALUE인 경우 불필요하게 많은 쓰레드가 생성되어 오히려 전반적인 처리 속도가 저하될 수도 있다. 따라서, 풀의 최소/최대 크기와 큐의 개수는 처리할 작업의 상태에 따라 알맞은 개수를 지정해주어야 한다.  

##### 2.1.2. TaskExecutor와 AsyncTaskExecutor 인터페이스를 이용한 작업 실행
<br/>
&#60;task:executor&#62; 태그를 이용해서 TaskExecutor 빈을 설정했다면, 해당 빈을 이용해서 작업을 실행할 수 있다. &#60;task:executor&#62; 태그는 org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor 객체를 생성한다.  

ThreadPoolExecutor 클래스가 구현하고 있는 TaskExecutor 인터페이스와 AsyncTaskExecutor 인터페이스는 각각 작업을 실행하는데 필요한 메서드를 정의하고 있으며, 각 메서드는 다음과 같다.  

+ void execute(Runnable task) : task를 실행한다.
+ Future&#60;?&#62; submit(Runnable task) : task를 실행한다. Future를 통해 작업이 완료될 때 처리 결과를 확인할 수 있다.
+ Future&#60;?&#62; submit(Callable&#60;T&#62; task) : task를 실행한다. Future를 통해 작업이 완료된 이후 처리 결과 및 리턴 값을 확인할 수 있다.
+ ListenableFuture&#60;?&#62; submitListenable(Runnable task) : task를 실행한다. ListenableFuture에 ListenableFutureCallback을 등록해서 작업이 완료될 때 콜백으로 결과를 받을 수 있다.
+ ListenableFuture&#60;T&#62; submitListenable(Callable&#60;T&#62; task) : task를 실행한다. ListenableFuture에 ListenableFutureCallback을 등록해서 작업이 완료될 때 콜백으로 결과를 받을 수 있다.  

ThreadPoolTaskExecutor 클래스의 execute() 메서드와 submit() 메서드는 비동기로 작업을 실행한다. 따라서, execute() 메서드나 submit() 메서드를 실행하면 작업 실행 여부에 상관없이 메서드가 즉시 리턴된다.  

아래 코드는 &#60;task:executor&#62; 태그를 이용해서 생성한 ThreadPoolTaskExecutor 클래스를 사용해서 작업을 실행하는 코드의 작성 예이다.  

```java
public class Processor {
	private ThreadPoolTaskExecutor taskExecutor;
	public void setTaskExecutor(ThreadPoolTaskExecutor taskExecutor) {
		this.taskExecutor = taskExecutor;
	}

	public void process(final Work work) {
		Future<?> future = taskExecutor.submit(nwe Runnable() {
			@Override
			public void run() {
				work.doWork();
			}
		});
		try {
			future.get(); // 작업이 끝날 때까지 대기
		} catch (InterruptedException e) {
			// 익셉션 처리
		} catch (ExecutionException e) {
			// 익셉션 처리
		}
		return;
	}
}
```

위 코드의 Processor 클래스를 초기화 해주는 스프링 설정은 다음과 같을 것이다.  

```xml
<task:executor id="executor" keep-alive="10" pool-size="10-20" queue-capacity="10" rejection-policy="ABORT" />

<bean id="processor" class="...">
	<property name="taskExecutor" ref="executor" />
</bean>
```

submitListenable() 메서드를 사용하면 org.springframework.util.concurrent.ListenableFuture 타입 객체를 리턴하는데, 이 타입 및 ListenableFutureCallback 인터페이스는 다음과 같이 정의되어 있다.  

```java
public interface ListenableFuture<T> extends Future<T> {
	void addCallback(ListenableFutureCallback<? super T> callback);
}

public interface ListenableCallback<T> {
	void onSuccess(T result);
	void onFailure(Throwable t);
}
```

ListenableFuture의 addCallback() 메서드는 콜백으로 사용할 콜백 객체를 전달받으며, 작업 실행이 성공하면 콜백 객체의 onSuccess() 메서드를, 익셉션이 발생하면 onFailure() 메서드를 실행해서 결과를 콜백 객체에 전달한다.  

#### 2.2. &#60;task:scheduler&#62;를 이용한 스케줄러 사용
<br/>
org.springframework.scheduling.TaskScheduler 인터페이스는 지정된 시간 또는 반복적으로 작업을 실행하기 위한 메서드를 제공하고 있다. 이 인터페이스를 구현한 빈 객체를 생성할 때 &#60;task:scheduler&#62; 태그를 이용한다.  

##### 2.2.1. &#60;task:scheduler&#62;를 이용한 스케줄러 생성
<br/>
&#60;task:scheduler&#62; 태그를 이용해서 스케줄러 빈을 생성하는 방법은 매우 간단하다. 다음과 같이 생성할 빈의 식별값과 쓰레드 풀의 개수만 입력해주면 된다.  

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:task="http://www.springframework.org/schema/task"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/spring-beans.xsd
			http://www.springframework.org/schema/task
			http://www.springframework.org/schema/task/spring-task.xsd"

	<task:scheduler id="scheduler" pool-size="4" />
```

pool-size를 지정하지 않을 경우 쓰레드 풀의 기본값은 1이다. 풀에 생성된 쓰레드를 이용해서 작업을 스케줄링 하기 때문에, 스케줄링 될 작업 개수에 다라 알맞게 풀의 크기를 지정해주면 된다.  

&#60;task:scheduler&#62; 태그가 생성하는 빈 객체는 ThreadPoolTaskScheduler 타입이다.  

##### 2.2.2. TaskScheduler 인터페이스를 이용한 작업 스케줄링
<br/>
&#60;task:scheduler&#62; 태그를 이용해서 TaskScheduler 빈을 생성했다면, TaskScheduler 인터페이스가 제공하는 메서드를 이용해서 작업 실행을 스케줄링 할 수 있다. TaskScheduler 인터페이스가 제공하는 메서드는 다음과 같다. 참고로, 모든 메서드의 리턴 타입은 SchedulerFuture이며, period 파라미터와 delay 파라미터의 단위는 1/1000초이다.  

+ schedule(Runnable task, Trigger trigger)  
Trigger가 지정한 시간에 작업을 실행한다.  
+ schedule(Runnable task, Date startTime)  
startTime에 작업을 한 번 실행한다.  
+ scheduleAtFixedRate(Runnable task, Date startTime, long period)  
startTime부터 period 시간마다 작업을 실행한다.  
+ scheduleAtFixedRate(Runnable task, long period)  
가능한 빨리 작업을 실행하고, 이후 period 시간마다 작업을 실행한다.  
+ scheduleWithFixedDelay(Runnable task, Date startTime, long delay)  
startTime부터 작업을 delay 시간 간격으로 작업을 실행한다.  
+ scheduleWithFixedDelay(Runnable task, long delay)  
가능한 빨리 작업을 실행하고, 이후 delay 시간 간격으로 작업을 실행한다.  

&#60;task:scheduler&#62; 태그를 이용하면 ThreadPoolTaskScheduler 타입의 빈 객체가 생성되는데 이 클래스는 TaskScheduler 인터페이스 뿐만 아니라 SchedulingTaskExecutor 인터페이스와 AsyncListenableTaskExecutor 인터페이스도 함께 구현하고 있다. 따라서, 스케줄링을 위한 메서드와 작업 실행을 위한 메서드도 실행할 수 있다.  

##### 2.2.3. CronTrigger를 이용한 스케줄링 설정
<br/>
schedule(Runnable task, Trigger trigger) 메서드의 두 번째 파라미터의 타입은 org.springframework.scheduling.Trigger 인터페이스인데, Trigger 인터페이스는 작업의 다음 실행 시간을 결정해주는 역할을 제공한다.  

스프링은 Trigger 인터페이스 구현 클래스로서 다음의 두 가지를 기본으로 제공하고 있다.  

+ org.springframework.support.CronTrigger
+ org.springframework.support.PeriodTrigger  

이름에서 유추할 수 있듯이 각 구현은 cron 방식의 스케줄링과 주기적인 실행을 위한 스케줄링을 이용할 때 사용된다.  

CronTrigger는 cron 표현식을 이용해서 작업 실행 시간을 제공한다. 아래 코드는 CronTrigger를 이용해서 작업 실행을 스케줄링하는 코드의 예를 보여주고 있다.  

```java
CronTrigger trigger = new CronTrigger("0 30 0 * * *");
scheduler.scheduler(logCollector, trigger);
```

위 코드는 매일 0시 30분에 logCollector를 실행하도록 설정한다.  

cron 표현식은 다음과 같이 구성된다.  

+ 초 분 시 일 월 요일  

각 시간 단위 값은 공백으로 구분되며, 다음의 표현식을 통해서 시간 간격, 주기 등을 표현할 수 있다.  

+ 특정 값 : 해당 시간을 정확하게 지정. 예) "0", "10", "20"
+ 값1-값2 : 값1부터 값2 사이를 표현. 예) "0-10"
+ 값1, 값2, 값3 : 콤마로 구분하여 특정 값 목록을 지정. 예) "0, 15, 30"
+ 범위/숫자 : 범위에 속한 값 중 숫자 만큼 간격으로 값 목록을 지정. 예를 들어, "0-23/2"는 0부터 23까지 2간격으로 값을 설정한다. 즉, 0, 2, 4, 6, ~, 20, 22의 값을 표현한다. &#42;을 사용해서 &#42;/2와 같이 표현할 수도 있다.  

허용되는 값의 범위는 다음과 같다.  

+ 0 0 &#42; &#42; &#42; &#42; : 매일 매시 정각
+ &#42;/10 &#42; &#42; &#42; &#42; &#42; : 0, 10, 20, 30, 40, 50초
+ 0 0 8-10 &#42; &#42; &#42; : 매일 8시, 9시, 10시 정각
+ 0 0/30 8-10 &#42; &#42; &#42; : 매일 8시, 8시 30분, 9시, 9시 30분, 10시
+ 0 0 9-18 &#42; &#42; 1-5 : 매주 월요일부터 금요일의 9시부터 오후 6시까지 매시  

#### 2.3. &#60;task:scheduled-tasks&#62;를 이용한 작업 스케줄링
<br/>
&#60;task:scheduled-tasks&#62; 태그를 이용하면 스케줄러를 사용해서 지정한 시간에 작업을 실행할 수 있다. &#60;task:scheduled-tasks&#62; 태그의 설정 방법은 다음과 같다.  

```xml
<task:scheduler id="scheduler" pool-size="5" />

<task:scheduled-tasks scheduler="scheduler">
	<task:scheduled ref="logCollector" method="collect" cron="0 30 0 0 0 0" />
</task:scheduled-tasks>

<bean id="logCollector" class="..." />
```

&#60;task-scheduled-tasks&#62; 태그의 scheduler 속성은 작업을 실행할 스케줄러 빈을 설정한다.  
&#60;task:scheduled-tasks&#62; 태그는 한 개 이상의 &#60;task:scheduled&#62; 태그를 가질 수 있다. &#60;task:scheduled&#62; 태그는 스케줄러를 통해서 실행될 작업을 설정하며, ref 속성과 method 속성을 이용해서 실행할 빈 객체와 메서드를 지정한다.  

&#60;task:scheduled&#62; 태그는 작업을 언제 실행할 지의 여부를 지정하기 위해 다음의 세 속성 중 한 가지를 사용한다.  

+ cron : cron 표현식을 이용해서 실행 시간을 표현한다.
+ fixed-delay : 지정된 시간 간격으로 작업을 실행한다. 단위는 1/1000초이다.
+ fixed-rate : 지정한 시간 주기로 작업을 실행한다. 단위는 1/1000초이다.
+ initial-delay : 지정한 시간 이후에 실행을 시작한다. fixed-delay나 fixed-rate를 사용할 때 적용되며, 단위는 1/1000초이다.  

위 속성을 여러 개 지정할 경우 적용 우선 순위는 cron, fixed-delay, fixed-rate 순이다.  

#### 2.4. 애노테이션을 이용한 작업 실행 설정
<br/>
&#60;task:scheduled-tasks&#62; 태그를 사용하지 않고 애노테이션을 사용해서 특정 빈 객체의 메서드를 주기적으로 실행하거나 비동기로 실행할 수 있다.  

&#64;Scheduled 애노테이션과 &#64;Async 애노테이션을 사용하려면 먼저 &#60;task:annotation-driven&#62; 태그를 이용해서 작업을 실행할 때 사용할 TaskExecutor와 TaskScheduler를 지정해야 한다.  

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:task="http://www.springframework.org/schema/task"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd
			http://www.springframework.org/schema/task
			http://www.springframework.org/schema/spring-task.xsd">

	<task:annotation-driven executor="executor" scheduler="scheduler" />

	<task:scheduler id="scheduler" pool-size="10" />
	<task:executor id="executor" keep-alive="5" pool-size="5-10" queue-capacity="10" rejection-policy="ABORT" />

	...
</beans>
```

##### 2.4.1. &#64;Scehduled 애노테이션을 이용한 작업 실행 설정
<br/>
&#64;Scheduled 애노테이션은 빈 객체의 특정 메서드를 스케줄러를 이용해서 실행할 때 사용된다. 다음 코드는 &#64;Scheduled 애노테이션의 적용 예이다.  

```java
public class LogProccessor {
	
	@Scheduled(fixedRate = 10000)
	public void handle() {
		...
	}
}
```

위 코드에서 &#64;Scheduled는 fixedRate의 값으로 10000를 주었는데, 이는 LogProccessor의 process() 메서드를 10초 주기로 실행한다는 것을 의미한다. &#60;task:annotation-driven&#62; 태그와 함께 빈 객체로 등록해주면, 지정한 시간 주기로 process() 메서드가 실행된다.  

```xml
<benas xmlns="http://www.springframework.org/schema/beans"
	xmlns:task="http://www.springframework.org/schema/task"
	...>
	<task:annotation-driven executor="executor" scheduler="scheduler" />
	<bean class="..." />

	...
</beans>
```

&#64;Scheduled 애노테이션은 리턴 타입이 void이고 파라미터를 갖지 않는 메서드에 적용되며, 스케줄링 설정을 위해 사용할 수 있는 속성은 다음과 같다.  

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
			<td>cron</td>
			<td>String</td>
			<td>cron 표현식을 설정한다.</td>
		</tr>
		<tr>
			<td>zone</td>
			<td>String</td>
			<td>cron 표현식의 시간을 구할 때 사용할 시간대를 지정한다. 지정하지 않으면 기본 시간대를 사용한다.</td>
		</tr>
		<tr>
			<td>fixedRate</td>
			<td>String</td>
			<td>지정한 시간 주기로 실행한다. 단위는 1/1000이다.</td>
		</tr>
		<tr>
			<td>fixedDelay</td>
			<td>String</td>
			<td>지정한 시간 간격으로 실행한다. 단위는 1/1000초이다.</td>
		</tr>
		<tr>
			<td>initialDelay</td>
			<td>String</td>
			<td>지정한 시간 이후에 실행을 시작한다. fixedRate 속성이나 fixedDelay 속성을 사용할 때 적용되며, 단위는 1/1000초이다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 2.4.2. &#64;Async 애노테이션을 이용한 비동기 실행
<br/>
&#64;Async 애노테이션은 지정한 메서드를 비동기 실행으로 변환해준다. 다음은 &#64;Async 애노테이션의 적용 예를 보여주고 있다.  

```java
public class MessageSender {

	@Async
	public void send(String message) {
		...// 비동기로 실행
	}
}
```


&#64;Scheduled 애노테이션과 마찬가지로 &#64;Async 애노테이션도 &#60;task:annotation-driven&#62; 태그와 함께 사용되어야 한다.  

```java
<task:annotation-driven executor="executor" scheduler="scheduler" />
<bean id="messageSender" class="..." />
```

빈으로 등록한 객체의 &#64;Async 애노테이션 적용 메서드를 호출하면 바로 리턴되며, 실제 메서드는 비동기로 실행된다.  

```java
MessageSender sneder = ctx.getBean("messageSender", MessageSender.class);
sender.send("로그");
// send 메서드 실행 시 바로 리턴 됨
// 실제 MessageSender.send() 메서드는 비동기로 실행
```

&#64;Async 애노테이션이 적용된 메서드는 파라미터를 가질 수 있으며, 리턴 타입으로 void나 Future 타입을 가질 수 있다. 따라서, 비동기로 실행된 메서드의 결과가 필요하다면 Future를 리턴 타입으로 사용하면 된다.  

```java
public class MessageSender {

	@Async
	public Future<String> send(String message) {
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("메시지 발송: " + message);
		return new AsyncResult<String>(message);
	}
}
```

AsyncResult 클래스는 스프링이 제공하는 Future 인터페이스 구현 클래스로서, Future 타입을 리턴해야 하는 경우 AsyncResult 클래스를 이용한다.  

다음 코드는 Future 타입을 리턴하는 &#64;Async 메서드이 사용 예를 보여주고 있다.  

```java
Future<String> future = messageSender.send("로그");
...
String result = future.get();
```

##### 2.4.3. &#60;task:annotation-driven&#62;의 프록시 생성 방식 설정
<br/>
&#60;task:annotation-driven&#62; 태그는 &#64;Async 애노테이션이 적용된 빈 객체를 비동기로 처리하기 위해 프록시 객체를 생성하는데, 이 프록시 객체를 클래스를 기준으로 생성하고 싶다면 다음과 같이 proxy-target-class 속성 값을 true로 지정하면 된다.  

```xml
<task:annotation-driven executor="executor" scheduler="scheduler" proxy-target-class="true"/>
```

#### 2.5. &#64;EnableScheduling 애노테이션을 이용한 스케줄러 실행
<br/>
자바 기반 설정을 사용할 경우, &#64;EnableScheduling 애노테이션을 이용하면 &#64;Scheduled 애노테이션이 적용된 빈 객체를 스케줄링해서 실행한다.  

```java
@Configuration
@EnableScheduling
public class TaskConfig {
	@Bean
	public LogProccesor logProcessor() {
		return new LogProccessor(); // @Schedulued 애노테이션 적용 클래스
	}
}
```

위 코드는 스케줄러를 설정하지 않았는데, 이 경우 단일 쓰레드를 사용하는 Executor를 통해 작업을 실행한다. 단일 쓰레드를 사용하는 스케줄러 대신 TaskScheduler나 ScheduledExecutorService를 스케줄러로 사용하고 싶다면, 이 두 타입을 지원하는 클래스를 빈으로 직접 등록해주면 된다. 

```java
@Configration
@EnableScheduling
public class TaskConfig {

	@Override
	public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
		taskRegistrar.addCronTask(new Runnable() {
			@Override
			public void run() {
				logCollector().collect();
			}
		}, "*/5 * * * * *");
	}

	@Bean
	public LogProccesor logProccessor() {
		return new LogProccessor();
	}

	@Bean
	public ThreadPoolTaskScheduler taskScheduler() {
		ThreadPoolTaskScehduler scheduler = new ThreadPoolTaskScheduler();
		scheduler.setPoolSize(4);
		scheduler.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
		return scheduler;
	}
}
```

작업 스케줄링을 등록하고 싶다면, 위와 같이 SchedulingConfigurer 인터페이스를 상속받은 &#64;Configuration 설정 클래스를 작성하면 된다.  

SchedulingConfigurer 인터페이스를 상속받은 클래스는 configureTasks() 메서드에서 스케줄러로 실행할 작업을 등록할 수 있다. configureTask() 메서드에서 스케줄러로 실행할 작업을 등록할 수 있다. configureTask() 메서드의 ScheduledTaskRegistrar 타입 파라미터는 스케줄링할 작업을 등록할 때 사용되며, 작업 등록을 위해 다음의 메서드를 제공하고 있다. 참고로 period, delay 파라미터의 시간 단위는 1/1000초이다.  

+ addTriggerTask(Runnable task, Trigger trigger)
+ addTriggerTask(TriggerTask task)
+ addCronTask(Runnable task, String expression)
+ addCronTask(CronTask task)
+ addFixedRateTask(Runnable task, long period)
+ addFiexdRateTask(IntervalTask task)  
+ addFixedDelayTask(Runnable task, long delay)
+ addFixedDelayTask(IntervalTask task)  

메서드 이름만 보면 어떤 주기로 실행될 작업을 등록하는지 이해가 될 것이다. TriggerTask, CronTask, IntervalTask 클래스는 작업 등록할 때 필요한 정보를 한 객체에 담아 제공할 때 사용되는 클래스로서, 각각 다음의 생성자를 제공하고 있다. 참고로, 이들 클래스는 모두 org.springframework.scheduling.config 패키지에 포함되어 있다.  

+ TriggerTask(Runnable runnable, Trigger trigger)
+ CronTask(Runnable runnable, String expression)
+ CronTask(Runnable runnable, CronTrigger cronTrigger)
+ IntervalTask(Runnable runnable, long interval, long initialDelay)
+ IntervalTask(Runnable runnable, long interval)

#### 2.6. &#64;EnableAsync 애노테이션을 이용한 &#64;Async 비동기 실행
<br/>
&#64;Configuration 설정 클래스에 &#64;EnableAsync 애노테이션을 사용하면 &#64;Async가 붙은 메서드를 비동기로 처리해준다.  

```java
@Configuration
@EnableAsync
public class TaskConfig {
	@Bean
	public MessageSender messageSender() {
		return new MessageSender(); // @Async 적용 메서드
	}
}
```

만약 비동기로 실행할 때 사용할 실행기를 지정하고 싶다면, 다음과 같이 &#64;Configuration 클래스에서 AsyncConfigurer 인터페이스를 상속받아 getAsync Executor() 메서드를 재정의해주면 된다.  

```java
@Configuration
@EnableAsync
public class TaskConfig implements AsyncConfigurer {

	@Bean
	public ThreadPoolTaskScheduler taskScheduler() {
		ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
		scheduler.setPoolSize(4);
		scheduler.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
		return scheduler;
	}

	@Bean
	public MessageSender messageSender() {
		return new MessageSender();
	}

	@Override
	public Executor getAsyncExecutor() {
		return taskScheduler(); // 비동기로 실행할 때 사용할 Executor 리턴
	}
}
```

### 3. RestTemplate을 이용한 HTTP 클라이언트 구현
<br/>
HTTP 기반의 오픈 API가 증가하고, 내부 서비스 간에도 HTTP를 기반으로 통신하는 경우가 증가하고 있다. 이런 이유로 HTTP에 기반한 클라이언트 코드를 작성해야 할 때가 많은데, 스프링은 HTTP 클라이언트를 구현하는데 필요한 기능을 구현한 RestTemplate 클래스를 제공하고 있다.  

#### 3.1. RestTemplate의 기본 사용법
<br/>
org.springframework.web.client.RestTemplate 클래스의 사용방법은 매우 간단하다. 다음과 같이 RestTemplate 객체를 생성하고 필요한 방식으로 접근하는 메서드를 사용해서 결과를 받기만 하면 된다.  

```java
RestTemplate restTemplate = new RestTemplate();
String body = restTemplate.getForObject("http://www.daum.net", String.class);
```

getForObject() 메서드는 HTTP GET 방식으로 http://www.daum.net에 연결해서 응답 결과를 String 타입으로 구한다. String 타입 외에 HttpMessageConverter를 사용하면 JSON이나 XML 응답을 바로 자바 객체로 변환할 수 있다.  

getForObject()에서 get은 HTTP GET 방식(method)을 의미하는 것으로, postForObject(), put(), delete(), headForHeaders() 등 각 HTTP 방식별로 메서드가 존재한다.  

RESTful 방식의 API는 URL이 "http://host/stores/스토어ID/items/아이템ID"와 같이 자원을 중심으로 경로를 구성하는데, RestTemplate은 이런 구조의 URL을 좀 더 쉽게 구성할 수 있도록 경로 변수를 사용할 수 있는 메서드를 함께 제공하고 있다.  

```java
String response = restTemplate.getForObject("http://localhost:8080/spring-chap17-s/stores/{storeId}", String.class, "1"};
```

위 코드는 경로 변수 storeId의 값으로 "1"이 사용되어 실제 사용하는 URL은 "http://localhost:8080/spring-chap17-s/stores/1"이 된다. 경로 변수를 두 개 이상 사용할 수 있으며, 이 경우 각 경로 변수의 위치에 해당하는 인자가 경로 변수의 값으로 사용된다. (위 메서드의 마지막 인자는 가변인자다.)

```java
String response = restTemplate.getForObject("http://localhost:8080/spring4-chap17-s/stores/{storeId}/items/{itemId}", String.class, "1", "I100");
```

다음과 같이 가변인자 대신 Map을 사용해서 경로 변수의 값을 지정할 수도 있다.  

```java
Map<String, Object> pathVariableMap = new HashMap<>();
pathVariableMap.put("storeId", 1L);

Store store = restTemplate.getForObject("http://localhost:8080/spring4-chap17-s/stores/{storeId}", Store.class, pathVariableMap);
```

RestTemplate 객체를 매번 생성해서 사용할 수 있지만, 그것보다는 다음과 같이 필드로 설정해서 사용한다.  

##### 3.1.1. 서버 에러 응답 처리
<br/>
RestTemplate은 서버와 통신하는 과정에서 문제가 발생하면 org.springframework.web.client.RestClientException을 발생시킨다. 상황별로 발생하는 RestClientException의 하위 타입은 다음과 같다.  

+ HttpStatusCodeException : 응답 코드가 에러에 해당할 경우 발생한다.
+ HttpClientErrorException : 응답 코드가 4XX일 때 발생한다.
+ HttpServerErrorException : 응답 코드가 5XX일 때 발생한다.
+ ResourceAccessException : 네트워크 연결에 문제가 있을 경우 발생한다.
+ UnknownHttpStatusCodeException : 알 수 없는 상태 코드일 때 발생한다.  

서버에서 에러 응답이 온 경우를 처리하고 싶다면 다음과 같이 익셉션을 잡아야 한다.  

```java
try {
	String response = restTemplate.getForObject("http://localhost:8080/spring4-chap17-s/stores/{storeId}/items/{itemId}", String.class, "1", "I100");
	...
} catch (HttpStatusCodeException e) {
	if (e.getStatusCode() == HttpStatus.NOT_FOUND) {
		...알맞은 익셉션 처리
	}
}
```

HttpStatusCodeException 및 그 하위 타입 두 개는 상태 코드를 구할 수 있도록 다음의 메서드를 제공하고 있다.  

+ HttpStatus getStatusCode() : org.springframework.http.HttpStatus 열거 타입을 리턴한다.  

UnknownHttpStatusCodeException 클래스가 제공하는 메서드는 다음과 같다.  

+ int getRawStatusException : 서버에서 전송한 응답 코드를 구한다.  

다음 메서드는 ResourceAccessException을 제외한 나머지 익셉션 타입이 동일하게 제공한다.  

+ String getStatusText() : HTTP 상태 문자값을 리턴한다.
+ HttpHeaders getResponseHeaders() : 응답 헤더를 org.springframework.http.HttpHeaders로 리턴한다.
+ String getResponsebodyAsString() : 응답 몸체를 문자열로 구한다.

#### 3.2. RestTemplate의 주요 메서드 : GET, POST, PUT, DELETE 처리
<br/>
RestTemplate 클래스는 각 HTTP 방식(method)별로 메서드를 제공하고 있는데, 이 중 GET 방식과 관련된 메서드는 다음과 같다.  

+ T getForObject(String url, Class&#60;T&#62; responseType, Object... urlVariables)
+ T getForObject(String url, Class&#60;T&#62; responseType, Map&#60;String, ?&#62; urlVariables)
+ T getForObject(URI url, Class&#60;T&#62; responseType)
+ ResponseEntity&#60;T&#62; getForEntity(String url, Class&#60;T&#62; responseType, Object... urlVariables)
+ ResponseEntity&#60;T&#62; getForEntity(String url, Class&#60;T&#62; responseType, Map&#60;String, ?&#62; urlVariables)
+ ResponseEntity&#60;T&#62; getForEntity(URI url, Class&#60;T&#62; responseType)  

각 파라미터는 다음과 같다.  

+ String 타입 url파라미터 : URL을 지정한다. 경로 변수를 포함될 수 있다.
+ URI 타입 url 파라미터 : URI를 이용해서 연결할 URL을 지정한다.  
+ urlVariables : 경로 변수에 사용될 값을 지정한다.
+ responseType : 응답 결과를 변환할 타입  

getForObject() 메서드는 응답 결과를 responseType으로 지정한 타입으로 바로 변환해서 리턴하며, getForEntity() 메서드는 org.springframework.http.ResponseEntity 타입으로 리턴한다. 상태 코드나 헤더 등의 값에 접근해야 할 경우 ResponseEntity를 사용할 수 있다. ResponseEntity가 제공하는 주요 메서드는 다음과 같다.  

+ HttpStatus getStatusCode() : 응답 상태 코드를 구한다.
+ HttpHeaders getHeaders() : 헤더 값을 구한다.
+ T getBody() : 몸체를 구한다.  

다음은 객체와 ResponseEntity를 리턴받는 두 경우의 사용 예를 보여주고 있다.  

```java
Store store1 = restTemplate.getForObject("http://localhost:8080/spring-chap17-s/stroes/{storeId}", Store.class, "1");

ResponseEntity<Store> response = restTemplate.getForEntity("http://localhost:8080/spring-chap17-s/stores/{storeId}", Store.class, "2");
Store store2 = response.getBody();
```

POST 방식을 위한 메서든 다음과 같다.  

+ T postForObject(String url, Object request, Class&#60;T&#62; responseType, Object... urlVariables)
+ T postForObject(String url, Object request, Class&#60;T&#62; responseType, Map&#60;String, ?&#62; urlVariables)
+ T postForObject(URI url, Object request, Class&#60;T&#62; responseType)
+ ResponseTypeEntity&#60;T&#62; postForEntity(String url, Object request, Class&#60;T&#62; responseType, Object... uriVariables)
+ ResponseTypeEntity&#60;T&#62; postForEntity(String url, Object request, Class&#60;T&#62; responseType, Map&#60;String, ?&#62; uriVariables)
+ ResponseEntity&#60;T&#62; postForEntity(URI url, Object request, Class&#60;T&#62; responseType)
+ URI postForLocation(String url, Object request, Object... urlVariables)
+ URI postForLocation(String url, Object request, Map&#60;String, ?&#62; urlVariables)
+ URI postForLocation(URI url, Object request)  

request 파라미터는 요청 몸체로 전송되며, 나머지 파라미터는 GET의 경우와 동일하다. postForObject()와 postForEntity()는 응답의 몸체 내용을 구할 때 사용되고, URI를 리턴하는 postForLocation() 메서드는 응답 결과로 응답의 Location 헤더 값을 구할 때 사용된다.  

RESTful 방식 API에서 POST는 새로운 데이터를 추가하는 목적으로 사용되는데, 이때 서버는 응답으로 새로 생성된 데이터에 접근할 수 있는 URL을 'Location' 응답 헤더에 담아 주는 경우가 많다. 이를 위해 RestTemplate은 postForLocation() 메서드를 추가로 제공하고 있다.  

PUT 방식을 위한 메서드는 다음과 같다. 이들 메서드는 리턴 타입이 모두 void다.  

+ void put(String url, Object request, Object... urlVariables)
+ void put(String url, Object request, Map&#60;String, ?&#62; urlVariables)
+ void put(URI url, Object request)  

DELETE 방식을 위한 메서드는 다음과 같다. 메서드 리턴 타입은 void다.  

+ void delete(String url, Object... urlVariables)
+ void delete(String url, Map&#60;String, ?&#62; urlVariables)
+ void delete(URI url)  

#### 3.3. HttpMessageConverter를 이용한 타입 변환
<br/>
GET이나 POST를 위한 메서드는 몸체 내용을 특정 타입의 객체로 변환해서 리턴한다. 비슷하게 POST나 PUT을 위한 메서드는 메서드에 전달한 객체를 요청 몸체로 알맞게 변환한다. 예를 들어, getForObject() 메서드는 JSON 형식의 응답 몸체 내용을 Store 타입의 객체로 변환해서 리턴하며, postForLocation() 메서드는 store3 객체를 JSON 형식의 요청 몸체로 변환해서 전송한다.  

RestTEmplate은 자바 객체와 요청/응답 몸체 사이의 변환 처리를 위해 HttpMessageConverter를 사용한다. RestTemplate이 기본으로 사용하는 HttpMessageConverter는 다음과 같다.  

+ ByteArrayHttpMessageConverter
+ StringHttpMessageConverter
+ ResourceHttpMessageConverter
+ SourceHttpMessageConverter&#60;Source&#62;
+ AllEncompassingFormHttpMessageConverter
+ AtomFeedHttpMessageConverter (Rome 라이브러리 존재시)
+ RssChannelHttpMessageConverter (Rome 라이브러리 존재시)
+ Jaxb2RootElementHttpMessageConverter
+ MappingJackson2HttpMessageConverter (Jackson2 라이브러리 존재시)  

MappingJackson2HttpMessageConverter와 Jaxb2RootElementHttpMessageConverter가 기본으로 사용되기 때문에, JSON이나 XML 형식의 응답을 자바 객체로 받거나 반대의 경우를 쉽게 처리할 수 있다.  

만약 RestTemplate이 사용하는 MessageConverter 구현체 목록을 변경하고 싶다면, 다음의 메서드를 이용해서 교체하면 된다.  

+ setMessageConverters(List&#60;HttpMessageConverter&#60;?&#62;&#62; messageConverters)  

#### 3.4. exchange() 메서드를 이용한 요청 헤더 설정
<br/>
요청 헤더를 직접 설정하고 싶다면 다음의 exchange() 메서드를 사용한다.  

+ ResponseEntity&#60;T&#62; exchange(String url, HttpMethod method, HttpEntity&#60;?&#62; requestEntity, Class&#60;?&#62; responseType, Object... uriVariables)
+ ResponseEntity&#60;T&#62; exchange(String url, HttpMehtod method, HttpEntity&#60;?&#62; requestEntity, Class&#60;?&#62; responseType, Map&#60;String, ?&#62; uriVariables)
+ ResponseEntity&#60;T&#62; exchange(URI url, HttpMethod method, HttpEntity&#60;?&#62; requestEntity, Class&#60;T&#62; responseType)  

org.springframework.http.HttpMethod 열거 타입은 전송 방식을 정한다. 이 열거 타입에는 GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE의 값이 정의되어 있다.  

org.springframework.http.HttpEntity는 요청 헤더와 요청 몸체를 설정할 때 사용되는 타입으로 주된 사용방법은 다음과 같다.  

```java
// getForEntity의 기능을 exchange를 이용해서 구현한 코드
HttpHeaders headers = new HttpHeaders();
headers.add("AUTHKEY", "mykey");
headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON);
HttpEntity<Void> requestEntity = new HttpEntity<>((Void) null, headers);

ResponseEntity<Item> itemResponse = restTemplate.exchange("http://localhost:8080/spring-chap17-s/stores/{storeId}/itmes/{itemId}", HttpMethod.GET, requestEntity, Item.class, "1", "I100");
Item item = itemResponse.getBody();
System.out.println(item);

// postForLocation의 기능을 exchange를 이용해서 구현한 코드
HttpEntity<Store> requestEntity2 = new HttpEntity<>(new Store("새로운 가게 2"), headers);

ResponseEntity<Void> postResponse = restTemplate.exchange("http://localhost:8080/spring-chap17-s/stores", HttpMethod.POST, rqeustEntity2, Void.class);
URI newStoreUri = postResponse.getHeaders().getLocation();
System.out.println(newStoreUri);
```

#### 3.5. URIBuilder를 이용한 URI 생성
<br/>
RestTemplate 클래스는 java.net.URI 타입을 이용해서 연결할 URL을 지정하는 메서드를 함께 제공하고 있다. URI 객체를 직접 생성할 수도 있지만, 스프링 제공하는 UriComponentsBuilder 클래스를 이용하면 경로 변수를 사용하는 URL을 포함한 URI 객체를 생성할 수 있다. 아래 코드는 작성 예이다.  

```java
UriComponentsBuilder uriCompBuilder = UriComponentsBuilder.newInstance();
UriComponents uriComp = uriCompBuilders.scheme("http").host("localhost").port(8080).path("/상점/{storeId}/items/{itmeId}).build();
uriComp = uriComp.expand("1", "품001").encode();
URI uri = uriComp.toUri();
```

UriComponentsBuilder 클래스는 scheme(), host(), port(), path() 메서드를 이용해서 URI를 구성할 수 있다. port()나 path() 등을 지정하지 않으면 생성되는 URI에 포함되지 않는다. path()로 지정하는 값은 위 코드에서 보듯 경로 변수를 사용할 수 있다.  

UriComponentsBuilder의 build() 메서드를 호출하면 UriComponents 객체를 생성한다. UriComponents의 expand() 메서드는 경로 변수의 값을 설정할 때 사용된다. 위 코드처럼 가변 인자로 경로 변수의 각 값을 설정해도 되고, Map을 이용해서 설정해도 된다.  

UriComponents의 encode() 메서드는 URI 각 구성 요소를 UTF-8 캐릭터셋을 이용해서 인코딩한다.  

encode() 메서드를 사용하지 않을 때 생성되는 URI 값은 원본을 그대로 포함한다.  

UriComponentsBuilder와 UriComponents는 모두 메서드 체이닝 방식을 사용할 수 있기 때문에, 다음과 같이 연속된 메서드 호출로 URI를 생성할 수도 있다.  

```java
URI uri = UriComponentsBuilder.newInstance().scheme("http").host("localhost").port(8080).path("/상점/{storeId}/itmes/{itemId}").build().expand("1", "품001").encode().toUri();
```

#### 3.6. AsyncRestTemplate을 이용한 비동기 응답 처리
<br/>
org.springframework.web.client.AsyncRestTemplate 클래스는 RestTemplate과 동일한 메서드를 제공하고 있다. 차이점이 있다면, 결과 객체를 바로 받는 대신 ListableFuture를 리턴 타입으로 갖는다는 점이다. 다음 코드는 AsyncRestTemplate의 사용 예이다.  

```java
AsyncRestTemplate asyncTemplate = new AsyncRestTemplate();
ListableFuture<ResponseEntity<String>> future = asyncTemplate.getForEntity("http://www.daum.net", String.class);

future.addCallback(new ListableFutureCallback<ResponseEntity<String>>() {
	@Override
	public void onSuccess(ResponseEntity<String> s) {
		String content = s.getBody();
		System.out.println(content.substring(0, 100));
	}
	@Override
	public void onFailure(Throwable t) {
		// 익셉션 처리
	}
});
```

비동기로 결과를 처리하기 위해 AsyncRestTemplate의 메서드는 ListableFuture 타입을 리턴하고, 결과 타입으로는 ResponseEntity를 사용한다.  

ListableFuture.addCallback()에 ListableFutureCallback 타입의 객체를 전달해서 처리 결과를 받을 수 있다. 성공적으로 처리되면 콜백 객체의 onSuccess() 메서드가 호출되고, 익셉션이 발생하면 onFailure() 메서드가 호출된다. onSuccess() 메서드는 ResponseEntity를 이용해서 필요한 값을 구할 수 있으며, onFailure() 메서드는 파라미터로 전달된 Throwable 객체를 이용해서 알맞은 익셉션 처리를 하면 된다.
