---
title: 확장포인트와 PropertyEditor/ConversionService
categories:
- Spring
feature_text: |
  ## 확장포인트와 PropertyEditor/ConversionService
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

### 1. 스프링 확장 포인트

스프링은 다음의 두 가지 방법을 이용해서 기능을 확장하는 방법을 제공하고 있다.
+ BeanFactoryPostProcessor를 이용한 빈 설정 정보 변경
+ BeanPostProcessor를 이용한 빈 객체 변경

#### 1.1. BeanFactoryPostProcessor를 이용한 빈 설정 정보 변경
<br/>
PropertySourcesPlaceholderConfigurer 클래스는 BeanFactoryPostProcessor 인터페이스를 구현하고 있는데, 스프링은 빈 객체를 실제로 생성하기 전에 설정 메타 정보를 변경하기 위한 용도로 BeanFactoryPostProcessor 인터페이스를 사용한다.

BeanFactoryPostProcessor 인터페이스를 구현한 클래스는 postProcessBeanFactory() 메서드에 전달되는 ConfigurableListableBeanFactory를 이용해서 설정 정보를 읽어와 변경하거나 새로운 설정 정보를 추가할 수 있다.

```java
public class 클래스명 implements BeanFactoryPostProcessor {

	...

	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		...
	}

	...
}
```

ConfigurableListableBeanFactory는 설정 정보를 구할 수 있는 두 개의 메서드를 제공하고 있다.
+ String[] getBeanDefinitionNames()  
설정된 모든 빈의 이름을 구한다.
+ BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException  
지정한 이름을 갖는 빈의 설정 정보를 구한다.

스프링이 BeanFactoryPostProcessor 구현 클래스를 사용하도록 만들려면, BeanFactoryPostProcessor 구현 클래스를 스프링 빈으로 등록해주어야 한다.

BeanFactoryPostProcessor는 빈의 설정 정보를 변경하는 방법을 사용하는데, 이런 이유로 &#64;Configuration 애노테이션을 이용해서 생성한 빈 객체에는 적용되지 않는다.

##### 1.1.1. BeanDefinition의 주요 메서드
<br/>
BeanDefinition 인터페이스는 빈 설정 정보를 구하거나 수정할 때 필요한 메서드를 정의하고 있다.
+ String getBeanClassName()  
생성할 빈의 클래스 이름을 구한다.
+ setBeanClassName(String beanClassName)  
생성할 빈의 클래스 이름을 지정한다.
+ String getFactoryMethodName()  
팩토리 메서드 이름을 구한다.
+ setFactoryMethodName(String factoryMethodName)  
팩토리 메서드 이름을 지정한다.
+ ConstructorArgumentValues getConstructorArgumentValues()  
생성자 인자 값 설정 정보를 구한다.
+ MutablePropertyValues getPropertyValues()  
프로퍼티 설정 정보를 구한다.
+ boolean isSingleton()  
싱글톤 범위를 갖는지 여부를 확인한다.
+ boolean isPrototype()  
프로토타입 범위를 갖는지 여부를 확인한다.
+ String getScope()  
빈의 범위를 문자열로 구한다.
+ setScope(String scope)  
빈의 범위를 문자열로 설정한다.

생성자 인자 설정과 프로퍼티 설정을 변경하려면 다음의 두 클래스를 이용하면 된다.
+ org.springframework.factory.config.ConstructorArgumentValues
+ org.springframework.MutablePropertyValues

#### 1.2. BeanPostProcessor를 이용한 빈 객체 변경
<br/>
스프링은 빈 객체를 초기화하는 과정에서 BeanPostProcessor의 두 메서드를 호출하고, 이 메서드가 리턴한 객체를 빈 객체로 사용한다. 따라서, 실제 사용할 빈 객체를 교체하고 싶다면 메서드로 전달받은 원래 빈 객체(bean 파라미터)가 아닌 새로운 객체를 생성해서 리턴하면 된다.

#### 1.3. Ordered 인터페이스/&#64;Order 애노테이션 적용 순서 지정
<br/>
두 개 이상의 BeanPostFactory가 존재할 경우, org.springframework.core.Ordered 인터페이스를 이용해서 순서를 정할 수 있도록 해야 한다.

스프링은 BeanPostProcessor가 여러 개 존재할 때, 다음의 방법을 이용해서 적용순서를 정한다.

+ BeanPostProcessor 구현 클래스가 Ordered 인터페이스를 구현한 경우, getOrder() 메서드를 이용해서 적용 순서 값을 구한다.
+ getOrder()로 구한 순서 값이 작은 BeanPostProcessor를 먼저 적용한다.
+ Ordered 인터페이스를 구현하지 않은 BeanPostProcessor는 나중에 적용한다.

```java
public class 클래스명 implements BeanPostProcessor, Ordered {
	
	private int order;

	@Override
	public int getOrder() {
		return order;
	}

	public void setOrder(int order) {
		this.order = order;
	}

	...

}
```

```java
@Order(자연수)
public class 클래스명 implements BeanPostProcessor {
	...
}
```

### 2. PropertyEditor와 ConversionService
#### 2.1. PropertyEditor를 이용한 타입 변환
<br/>
자바빈 규약은 문자열을 특정 타입의 프로퍼티로 변환할 때 java.beans.PropertyEditor 인터페이스를 사용한다. 자바는 기본 데이터 타입을 위한 PropertyEditor를 이미 제공하고 있다. sun.beans.editors 패키지에는 BooleanEditor, LongEditor, EnumEditor 등 기본 타입을 위한 PropertyEditor를 제공하고 있으며, 이들을 통해서 문자열을 boolean, long, enum 등의 타입으로 변환할 수 있다.

자바에 기본으로 포함된 PropertyEditor 구현체는 기본적인 타입만 지원하기 때문에, 스프링은 설정 등의 편의를 위해 PropertyEditor를 추가로 제공하고 있다.

##### 2.1.1. 스프링이 제공하는 주요 PropertyEditor
<br/>
스프링은 URL을 비롯해 주요 타입에 대한 PropertyEditor를 제공하고 있으며 이들 목록은 다음과 같다. 모든 클래스는 org,springframework.beans.propertyeditors 패키지에 위치하며, '기본'이라고 표시한 것은 별도 설정 없이 사용되는 것들이다.

+ ByteArrayPropertyEditor  
String.getBytes()를 이용해서 문자열을 byte 배열로 변환(기본)
+ CharArrayPropertyEditor  
String.toCharArray()를 이용해서 문자열을 char 배열로 변환
+ CharsetEditor  
문자열을 Charset으로 변환(기본)
+ ClassEditor  
문자열을 Class 타입으로 변환(기본)  
+ CurrencyEditor  
문자열을 java.util.Currency로 변환
+ CustomBooleanEditor(기본)  
문자열을 Boolean 타입으로 변환
+ CustomDateEditor  
DateFormat을 이용해서 문자열을 java.util.Date로 변환
+ CustomNumberEditor(기본)  
Long, Double, BigDecimal 등 숫자 타입을 위한 프로퍼티 에디터
+ FileEditor(기본)  
문자열을 java.io.File로 변환
+ LocaleEditor(기본)  
문자열을 Locale로 변환
+ PatternEditor(기본)  
정규 표현식 문자열을 Pattern으로 변환
+ PropertiesEditor(기본)  
문자열을 Properties로 변환
+ URLEditor(기본)  
문자열을 URL로 변환

CustomDateEditor나 PatternEditor는 기본으로 사용되지 않는다. 따라서, XML 설정에서 Date 타입의 프로퍼티를 설정하면 익셉션을 발생시킨다. 기본으로 사용되지 않는 PropertyEditor를 사용하려면, PropertyEditor를 추가로 등록해주어야 한다.

##### 2.1.2. 커스텀 PropertyEditor 구현하기
<br/>
직접 구현한 클래스처럼 PropertyEditor가 존재하지 않는 경우 PropertyEditor를 직접 구현해주어야 한다.
```java
public class 클래스명 extends PropertyEditorSupport {

	@Override
	public void setAsText(String text) throws IllegalArgumentException {
		
		...

		setValue(...);

		...

	}
}
```

##### 2.1.3. PropertyEditor 추가 : 같은 패키지에 PropertyEditor 위치시키기
<br/>
PropertyEditor를 구현했다면, 실제로 PropertyEditor를 사용하도록 PropertyEditor를 등록해주어야 한다. 이를 위한 가장 쉬운 방법은 다음과 같은 규칙에 따라 PropertyEditor를 작성하는 것이다. (이 규칙은 자비빈 규약에 명시된 것이다.)

+ 변환 대상 타입과 동일한 패키지에 '타입Editor' 이름으로 PropertyEditor를 구현

##### 2.1.4. PropertyEditor 추가 : CustomEditorConfigurer 사용하기
<br/>
특정 타입을 위해 구현한 PropertyEditor가 다른 패키지에 위치하거나 이름이 '타입Editor'가 아니라면, CustomEditorConfigurer를 이용해서 설정해주어야 한다. CustomEditorConfigurer는 BeanFactoryPostProcessor로서 스프링 빈을 초기화하기 전에 필요한 PropertyEditor를 등록할 수 있도록 해준다.

```xml
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
	<property name="customEditors">
		<map>
			<entry key="완전한 클래스명" value="PropertyEditorSupport를 상속한 완전한 클래스명"/>
		</map>
	</property>
</bean>
```

##### 2.1.5. PropertyEditor 추가 : PropertyEditorRegistrar 사용하기
<br/>
CustomEditorConfigurer의 customEditors 프로퍼티를 이용해서 설정할 때 단점은 PropertyEditor에 매개변수를 설정할 수 없다는 점이다. 특정 패턴에 맞게 입력한 문자열을 Date 타입으로 변환하고 싶은 경우 패턴을 지정할 수 있어야 하는데, 앞서 설정 방식은 PropertyEditor의 클래스 이름을 사용하므로 이런 매개변수를 지정할 수 없다. 이렇게 PropertyEditor에 매개 변수를 지정하고 싶을 때 사용할 수 있는 것이 PropertyEditorRegistrar이다.

특정 타입을 위한 PropertyEditor 객체를 직접 생성해서 설정하고 싶다면 다음과 같은 방법으로 PropertyEditorRegistrar를 사용하면 된다.
+ PropertyEditorRegistrar 인터페이스를 상속받은 클래스에서 PropertyEditor를 직접 생성하고 등록한다.
+ 1번 과정에서 생성한 클래스를 빈으로 등록하고, CustomEditorConfigurer에 propertyEditorRegistrars 프로퍼티로 등록한다.

먼저 할 작업은 PropertyEditorRegistrar 인터페이스를 상속받아 구현하는 것이다. 상속 받은 클래스는 registerCustomEditors() 메서드에서 원하는 PropertyEditor를 생성해서 등록하면 된다.

#### 2.2. ConversionService를 이용한 타입 변환
<br/>
스프링 3 버전부터 ConversionService가 추가되었다. ConversionService는 좀 더 범용적인 타입 변환을 처리하기 위한 인터페이스다. 앞서 살펴본 PropertyEditor는 자바빈 규약에 따라 문자열과 타입 간의 변환을 처리해주는 방식인데 반해, ConversionService는 타입과 타입 간의 변환을 처리하기 위한 기능을 정의하고 있다.

ConversionService를 스프링 빈으로 등록하면, 스프링은 PropertyEditor 대신 ConversionService를 이용해서 타입 변환을 처리한다.

스프링은 이미 ConversionService 인터페이스를 구현한 클래스를 제공하고 있으므로, 이 구현체를 사용하면 된다.

##### 2.2.1. ConversionServiceFactoryBean를 이용한 ConversionService 등록
<br/>
DefaultConversionService 클래스는 스프링이 제공하는 ConversionService 구현이다. 이 클래스를 ConversionService로 사용하려면 ConversionServiceFactoryBean 클래스를 빈으로 등록하면 된다. ConversionServiceFactoryBean는 팩토리로서 DefaultConversionService를 빈 객체로 생성해준다.
```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
</bean>
```

ConversionServiceFactoryBean 클래스를 설정할 때 주의할 점은 빈 객체의 이름을 "conversionService"로 등록해야 한다는 점이다. 그렇지 않을 경우 PropertyEditor를 사용한다.

위와 같이 "conversionService" 빈 객체를 등록하면, 스프링은 빈 객체를 설정할 때 conversionService의 convert() 메서드를 이용해서 String 데이터를 int 타입으로 변환한다.

DefaultConversionService 클래스는 타입 변환을 직접 하지 않고 내부에 등록된 GenericConverter에 위임한다. DefaultConversionService는 다수의 컨버터를 갖고 있으며, convert() 메서드를 실행하면 다음과 같은 방식으로 타입을 변환한다.

+ 등록된 GenericConverter들 중에서 소스 객체의 타입을 대상 타입으로 변환해주는 Generic Converter를 찾는다.
+ GenericConverter가 존재할 경우, 해당 GenericConverter를 이용해서 타입 변환을 수행한다.
+ 존재하지 않을 경우, 익셉션을 발생한다.

PropertyEditor와 마찬가지로 DefaultConversionService는 이미 기본적인 타입 변환을 위한 GenericConverter를 등록하고 있다. 따라서, DefaultConversionService만 등록하면 추가 설정 없이 기본 데이터 타입이나, Properties, URL 타입 등에 대한 변환을 처리할 수 있다.

##### 2.2.2. GenericConverter를 이용한 커스텀 변환 구현
<br/>
getConvertibleTypes() 메서드는 변환 가능한 타입 쌍(ConvertiblePair)의 집합을 리턴한다. convert() 메서드는 TypeDescriptor 정보를 이용해서 source 객체를 대상 타입으로 변환해서 리턴한다.

```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
	<property name="converters">
		<set>
			<bean class="GenericConverter를 구현한 완전한 클래스명" />
		</set>
	</property>
</bean>
```

##### 2.2.3. Converter를 이용한 커스텀 변환 구현
<br/>
여러 타입에 대해 타입 변환 기능을 제공할 경우 GenericConverter 인터페이스를 구현하는 것이 적합하다. 그런데, 많은 경우 원본 타입과 대상 타입이 하나이다. 그런데, 많은 경우 원본 타입과 대상 타입이 하나이다.

##### 2.2.4. FormattingConversionServiceFactoryBean을 이용한 ConversionService 등록
<br/>
스프링은 DefaultConversionService 외에 DefaultFormattingConversionService를 추가로 제공하고 있다. DefaultFormattingConversionService 클래스는 Converter/GenericConverter뿐만 아니라 Formatter를 이용해서 타입 변환을 수행하는 ConversionService 구현체이다. DefaultFormattingConversionService를 ConversionService로 사용하려면 다음과 같이 FormattingConversionServiceFactoryBean 클래스를 빈으로 등록하면 된다. 이 팩토리 클래스는 DefaultFormattingConversionService 빈 객체를 생성해준다.

```xml
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
</bean>
```

DefaultFormattingConversionService은 앞서 살펴본 DefaultConversionService가 기본으로 등록하는 Converter/GenericConverter 외에 날짜/시간 변환을 위한 Formatter와 Converter를 추가로 등록한다. 이 중에는 &#64;DateTimeFormat 애노테이션과 &#64;NumberFormat을 이용해서 타입 변환을 처리할 수 있는 Formatter도 포함되어 있다.

```java
public class 클래스명 {
	private Date 변수명;

	@DateTimeFormat(pattern="yyyy-MM-dd HH:mm:ss")
	public void set변수명(Date date) {
		this.변수명 = date;
	}

	...

}
```

##### 2.2.5. Formatter를 이용한 커스텀 변환 구현
<br/>
DefaultFormattingConversionService는 GenericConverter/Converter와 함께 Formatter/Parser/Printer 타입을 이용한 타입 변환을 지원한다. 

```java
public interface Printer<T> {
	String print(T object, Locale loacle);
}

public interface Parser<T> {
	T parse(String text, Locale locale) throws ParseException;
}

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```

위 타입 정의를 보면 알 수 있겠지만, Printer 인터페이스는 객체를 문자열로 변환해주고 Parser 인터페이스는 문자열을 지정한 타입으로 변환해준다. print() 메서드와 parse() 메서드는 Locale을 파라미터로 전달받는데, 이는 로케일에 따라 변환 결과를 만들어낼 수 있다는 것을 뜻한다.

Formatter 인터페이스는 Printer와 Parser 인터페이스를 상속받은 인터페이스로, 두 기능을 함께 제공할 클래스는 Formatter 인터페이스를 상속받아 구현하면 된다.

```java
public class 클래스명 implements Formatter<포맷팅할클래스> {

	@Override
	public String print(포맷팅할클래스, Locale locale) {

		...

		return 출력할문자열;
	}

	@Override
	public 포맷팅할클래스 parse(String text, Locale locale) throws ParseException {
		
		...

		return new 포맷팅할클래스(파라미터);
	}
}
```
