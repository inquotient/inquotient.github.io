---
title:  PrintStream 클래스
categories:
- Java Network Programming
feature_text: |
  ## PrintStream 클래스
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

PrintStream 클래스는 System.out으로 사용되기 때문에 대부분의 프로그래머가 처음 접하게 되는 필터 출력 스트림이다. 그러나 다음 두 개의 생성자를 사용하여 다른 출력 스트림을 프린트 스트림(print stream)에 연결할 수 있다.  

+ public PrintStream(OutputStream out)
+ public PrintStream(OutputStream out, boolean autoFlush)  

기본적으로 프린트 스트림은 명시적으로 플러시해야 한다. 그러나 autoFlush 인자를 true로 설정하면, 바이트 배열이나 라인피드(linefeed)가 쓰이거나 println() 메소드가 호출될 때 자동으로 플러시가 실행된다.  

그리고 PrintStream 클래스는 write(), flush(), close() 메소드뿐만 아니라 9개의 오버로드된 print() 메소드와 10개의 오버로드된 println() 메소드를 포함하고 있다.  

+ public void print(boolean b)
+ public void print(char c)
+ public void print(int i)
+ public void print(long l)
+ public void print(float f)
+ public void print(double d)
+ public void print(char[] text)
+ public void print(String s)
+ public void print(Object o)
+ public void println()
+ public void println(boolean b)
+ public void println(char c)
+ public void println(int i)
+ public void println(long l)
+ public void println(float f)
+ public void println(double d)
+ public void println(char[] text)
+ public void println(String s)
+ public void println(Object o)  

각 print() 메소드는 인자로 주어진 값을 분자열로 변경하고, 이 문자열을 기본 인코딩을 사용하여 내장된 출력 스트림에 쓴다. println() 메소드도 비슷하게 동작하지만 출력할 줄의 마지막에 플랫폼 의존적인 줄 구분자(line separator)를 추가한다. 맥 OS X를 포함한 유닉스의 경우 줄 구분자로 라인피드(&#92;n)가 사용되고, 맥 OS 9에서는 캐리지리턴(carriage return)(&#92;r)이, 윈도우의 경우 캐리지리턴(&#92;r)/라인피드(&#92;n) 쌍이 사용된다. 따라서 네트워크 프로그래머라면 PrintStream의 사용을 피하는게 좋다.  

println() 메소드 출력의 첫 번째 문제는 플랫폼 의존적이라는 것이다. 코드가 실행되는 시스템에 따라 라인피드, 캐리지리턴, 라인피드/캐리지리턴 때문에 종종 줄이 깨지기도 한다. 콘솔(console)에 출력하는 경우 큰 문제를 일으키지 않지만 엄격한 프로토콜이 요구되는 네트워크 클라이언트나 서버에서는 큰 재앙이 발생할 수 있다. HTTP와 Gnutella 같은 대부분의 네트워크 프로토콜이 캐리지리턴/라인피드 쌍으로 종료되길 요구한다. println()을 사용하면 윈도우에서 동작하는 프로그램을 쉽게 작성할 수 있지만 유닉스나 맥에서는 제대로 동작하지 않는다. 이미 널리 사용되는 많은 서버나 클라이언트는 올바르지 않은 줄 구분자에 충분히 대응할 수 있지만 가끔 예외가 발생한다.  

PrintStream의 두 번째 문제는 프로그램이 실행 중인 플랫폼의 인코딩을 기본 인코딩으로 가정한다는 것이다. 그러나 이 기본 인코딩은 서버나 클라이언트가 예상하던 것이 아닐 수 있다. 예를 들어, XML 파일을 수신 중인 웹 브라우저는 서버가 특별한 인코딩 타입을 지정하지 않으면 UTF-8이나 UTF-16으로 인코딩되어 있다고 가정한다. 그러나 PrintStream을 사용하는 웹 서버의 경우 클라이언트가 이해할 수 있는 인코딩 타입에 대한 고려 없이 서버의 지역화 설정에 따라 한국의 경우 KSC5601, 미국의 경우 CP1252, 일본의 경우 SJIS로 인코딩하여 정송하는 상황이 발생할 수 있다. PrintStream은 기본 인코딩 타입을 변경할 수 있는 어떠한 방법도 제공하지 않는다. 이 문제는 이와 관련된 PrintWriter 클래스를 대신 사용하면 해결된다. 하지만 PrintStream의 문제는 여전히 남아 있다.  

세 번째 문제점은 PrintStream이 모든 예외를 무시한다는 것이다. 이러한 PrintStream의 특성은 이를 HelloWorld 같은 교과서용 예제 프로그램에나 적합하게 만든다. 예외 처리를 처음 배우는 학생들에게 PrintStream을 사용하여 콘솔 출력을 쉽게 가르칠 수 있기 때문이다. 그러나 네트워크 연결의 경우 콘솔 출력보다 다소 안정적이지 못하다. 네트워크 연결은 네트워크의 과부하 및 원격 시스템 오류 같은 많은 이유로 종종 실패한다. 네트워크 프로그램은 데이터 전송이 방해 받는 예상치 못한 상황에 대응할 수 있도록 준비해야 한다. 예외 처리를 통해서 이러한 일에 대응할 수 있지만, PrintStream은 내장된 출력 스트림이 발생시키는 모든 예외를 붙잡는다. PrintStream의 다섯 가지 표준 OutputStream 메소드의 선언에는 thrwos IOException이 선언되어 있지 않다는 것응 알 수 있다.  

+ public abstract void write(int b)
+ public void write(byte[] data)
+ public void write(byte[] data, int offset, int length)
+ public void flush()
+ public void close()  

대신 PrintStream은 에러 플래그(error flag) 방식에 의존하고 있으며, 하위 스트림에서 예외가 발생하면 이 내부 에러 플래그를 설정한다. 프로그래머는 checkError() 메소드를 호출하여 플래그의 값을 확인할 수 있다.  

+ public boolean checkError()  

PrintStream에서 발생하는 모든 에러를 검사하기 위해 PrintStream의 메소드가 호출될 때마다 명시적으로 checkError()를 호출하여 검사해야 한다. 게다가 에러가 발생한 이후에는 추가 에러를 탐지하기 위해 플래그를 초기화할 방법이 없고, 에러에 대한 추가적인 정보를 얻을 수도 없다. 요약하면, 신뢰할 수 없는 네트워크 연결에 대해 PrintStream이 제공하는 에러 알림 방식은 상당히 부적절하다.
