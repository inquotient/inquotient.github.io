---
title:  입력 스트림
categories:
- Java Network Programming
feature_text: |
  ## 입력 스트림
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

자바가 제공하는 기본 입력 클래스는 java.io.InputStream이다.  

+ public abstract class InputStream  

이 클래스는 바이트 데이터를 읽는데 다음과 같은 기본 메소드를 제공한다.  

+ public abstract int read() throws IOException
+ public int read(byte[] input) throws IOException
+ public int read(byte[] input, int offset, int length) throws IOException
+ public long skip(long n) throws IOException
+ public int available() throws IOException
+ public void close() throws IOException  

InputStream의 서브클래스는 특정 매체로부터 데이터를 읽기 위해 이 메소드를 사용한다. 예를 들어, FileInputStream은 파일로부터 데이터를 읽는데 사용하며, TelnetInputStream은 네트워크 연결로부터 데이터를 읽는데 사용한다. 그리고 ByteArrayInputStream은 바이트 배열로부터 데이터를 읽는데 이 메소드를 사용한다. 스트림을 통해 읽고 있는 대상에 관계 없이 주로 이 6개의 메소드를 사용하여 데이터를 읽을 수 있다. 가끔은 읽고 있는 스트림의 종류가 정확히 무엇인지 알지 못하는 상황에서 이 메소드를 사용하기도 한다. 예를 들어, TelnetInputStream 클래스의 경우 sun.net 패키지 안에 숨겨져 있고 문서화조차 되어 있지 않다. TelnetInputStream의 인스턴스는 java.net.URL의 openStream() 메서드를 포함한 java.net 패키지의 다양한 메소드에 의해 반환된다. 하지만 이러한 메소드는 TelnetInputStream이 아닌 InputStream을 반환하도록 선언되어 있고, 구체적인 서브클래스에 대한 인식 없이 동일한 방법으로 위 6개의 메소드를 사용하여 데이터를 읽을 수 있다. 이것은 바로 다형성이 동작하기 때문이다. 서브클래스의 인스턴스는 슈퍼클래스의 인스턴스처럼 투명하게 사용될 수 있으며, 서브클래스에 대한 별다른 지식 없이 사용할 수 있다.  

InputStream의 기본 메소드는 인자가 없는 read() 메소드다. 이 메소드는 입력 스트림의 매체로부터 단일 바이트를 읽고, 0에서 255 사이의 값을 정수 타입으로 반환한다. 그리고 스트림의 끝에 도달할 경우 -1을 반환한다. read() 메소드는 1바이트도 읽을 것이 없는 경우 프로그램의 실행을 중단하고 기다리며, 이러한 특성 때문에 종종 입출력은 매우 느리고 성능에도 많은 영향을 준다. 속도에 민감한 작업을 처리해야 하는 상황이라면 입출력을 별도의 스레드로 분리하는 것이 좋다.  

서브클래스마다 특정 매체를 처리하기 위해 read() 메소드를 수정할 필요가 있다. 그래서 read() 메소드는 추상 메소드로 선언되어 있다. 예를 들어, ByteArrayInputStream의 read() 메소드는 순수 자바 코드를 사용하여 배열로부터 바이트를 복사하도록 구현되고, TelnetInputStream의 read() 메소드는 호스트 플랫폼의 네트워크 인터페이스로부터 데이터를 읽기 위해 네이티브 라이브러리를 사용하여 구현된다.  

다음 코드는 InputStream in에서 10바이트를 읽고, 바이트 배열 input에 저장한다. 그리고 스트림의 끝에 도달하면 루프는 바로 종료된다.  

```java
byte[] input = new byte[10];
for (int i = 0; i < input.length; i++) {
  int b = in.read();
  if (b == -1) break;
  input[i] = (byte) b;
}
```

read() 메소드는 비록 바이트 값만 읽을 수 있지만 int 타입을 반환한다. 그리고 read() 메소드의 반환값을 바이트 배열에 저장하기 전에 byte 타입으로 캐스팅한다. int 타입을 바이트 타입으로 캐스팅하면 0에서 255까지의 부호 없는 바이트가 아닌 -128에서 127까지 부호 있는 바이트 값이 생성되며, 부호 없는 바이트 값이 필요한 경우 다음과 같이 변환할 수 있다.  

```java
int i = b >= 0 ? b: 256 + b;
```

한 번에 1바이트씩 읽는 것은 한 번에 1바이트씩 쓰는 것과 마찬가지로 매우 비효율적이다. 그 결과, 지정된 스트림으로부터 읽은 다수의 데이터를 배열로 채워 반환하는 read(byte[] input)과 read(byte[] input, int offset, int length) 두 개의 오버로드된 메소드가 추가로 존재한다. 첫 번째 메소드는 배열 input 크기만큼 읽기를 시도하며, 두 번째 메소드는 배열의 offset 위치부터 length 길이만큼 읽기를 시도한다.  

이 두 메소드는 배열의 크기만큼 읽어 반환을 시도(attempt)하지만 항상ㅇ 성공하는 것은 아니다. 흔히 발생하는 경우는 아니지만, 예를 들어 작성한 프로그램이 원격의 서버로부터 데이터를 읽고 있는 동안 ISP에 위치한 스위치 장비의 장애 때문에 네트워크가 단절될 수도 있다. 이러한 상황에서 읽기는 실패하며 IOException이 발생한다. 좀 더 일반적인 상황으로는 요청한 데이터의 전체가 아닌 일부만 읽을 수 있는 실패도 성공도 아닌 애매한 상황이 발생하기도 한다. 예를 들어, 네트워크 연결로부터 1,024바이트만큼 읽기를 시도했지만, 실제로는 512바이트만 서버로 도착했고 나머지 512바이트는 전송 중인 상태가 발생할 수 있다. 전송 중인 나머지 512바이트도 결국 도착할 테지만, 지금 이 순간에는 읽을 수 없다. 이러한 경우 멀티바이트 read() 메소드는 실제로 읽은 바이트 수를 반환한다. 다음 코드를 살펴보자.  

```java
byte[] input = new byte[1024];
int bytesRead = in.read(input);
```

이 코드는 InputStream in에서 1,024바이트만큼 읽어 input 배열에 저장하려고 한다. 그러나 만약 512바이트만 이용 가능한 상황이라면, read 함수는 512바이트만 읽고 반환한다. 실제 읽고자 하는 바이트의 크기가 보장되어야 하는 바이트의 크기가 보장되어야 하는 상황이라면, 배열이 가득 찰 때까지 반복해서 시도하는 루프 안에 read 메소드를 위치시켜 이러한 문제를 해결할  수있다. 예를 들면 다음 코드와 같다.  

```java
int bytesRead = 0;
int bytesToRead = 1024;
byte[] input = new byte[bytesToRead];
while (bytesRead < bytesToRead) {
  bytesRead += in.read(input, bytesRead, bytesToRead - bytesRead);
}
```

이 방법은 네트워크 스트림에서 읽을 때 자주 사용된다. 물론 파일을 읽을 때도 사용될 수 있다. 그러나 네트워크를 통한 이동이 CPU보다 훨씬 느리고 종종 데이터가 모두 도작하기도 전에 네트워크 버퍼를 비우는 프로그램으로 인해 네트워크 스트림에서 좀 더 유용하게 사용된다. 이 두 메소드는 임시적으로 비어 있는 열린 네트워크 버퍼에 대해 읽기를 시도할 경우 일반적으로 0을 반환하며, 읽을 데이터는 없지만 스트림은 여전히 열려 있는 상태를 나타낸다. 이러한 동작 방식을 같은 상황에서 스레드의 실행을 중지시키는 단일 바이트 read()보다 종종 유용하게 사용된다.  

이 세 가지 모든 read() 메소드는 스르림의 끝에 ㄷ달할 경우 -1을 반환한다. 아직 읽지 않은 데이터가 남아 있는 상태에서 스트림의 종료될 경우 멀티바이트 read() 메소드는 버퍼를 비울 때까지 데이터를 모두 읽어 반환한다. 그리고 다시 read() 함수를 호출하면 -1을 반환한다. -1이 반환될 경우 배열에 어떠한 값도 반환되지 않으며, 배열의 값은 호출 전 상태로 남아 있다. 위의 예제 코드는 1,024바이트 이하의 데이터가 전송되고 스트림의 종료되는 상황을 고려하지 않고 있다. read() 메소드의 반환값을 bytesRead에 더하기 전에 검사하여 이러한 상황을 처리할 수 있다. 다음 코드를 보도록 하자.  

```java
int bytesRead = 0;
int bytesToRead = 1024;
byte[] input = new byte[bytesToRead];
while (bytesRead < bytesToRead) {
  int result = in.read(input, bytesRead, bytesToRead - bytesRead);
  if (result == -1) break; // 스트림의 끝
  bytesRead += result;
}
```

필요한 데이터를 모두 읽을 수 있을 때까지 실행이 대기되는 상황을 원하지 않을 경우, 대기 없이 즉시 읽을 수 있는 바이트 수를 반환하는 available() 메소드를 사용하여 읽을 바이트 수를 결장할 수 있다. 이 메소드는 읽을 수 있는 최소 바이트 수를 반환한다. 사실 좀 더 많은 바이트를 읽을 수도 있겠지만, 최소한 available() 메소드가 제안하는 만큼은 읽을 수 있음을 의미한다. 다음 예제를 보자.  

```java
int bytesAvailable = in.available();
byte[] input = new byte[bytesAvailable];
int bytesRead = in.read(input, 0, bytesAvailable);
// 대기 없이 프로그램의 싫랭을 계속한다
```

이 예제에서 bytesRead의 값이 bytesAvailable의 값과 정확히 같다고 기대할 수 있지만 항상 그런 것은 아니다. 스트림의 끝에 이를 경우 available() 메소드는 0을 반환한다. 그러나 일반적으로 read(byte[] input, int offset, int length) 메소드는 스트림의 끝에 이를 경우 -1을 반환하고 읽을 데이터가 없는 경우 0을 반환한다.  

종종 일부 데이터를 읽지 않고 건너뛰어야 할 경우가 있다. 이때는 skip() 메소드를 사용할 수 있다. 이 메소드는 파일로부터 읽을 때보다 네트워크 연결에 사용할 때 다소 유용하지 못하다. 네트워크 연결은 순차적이며 일반적으로 상당히 느리다. 그러므로 데이터를 건너뛴다고 해서 성능에 크게 영향을 주지 않는다. 파일을 읽을 때 사용하면 좀더 유용하긴 하지만, 파일은 네트워크와 달리 임의 위치 접근이 가능하기 때문에 이 경우에도 건너뛰는 방식보다는 파일 포인터의 위치를 재지정하는 편이 낫다.  

출력 스트림과 마찬가지로, 입력 스트림을 다 쓴 후에는 스트림 자신의 close() 메소드를 호출하여 닫아야 한다. close() 메소드는 파일 핸들 또는 포트 같은 스트림과 관련된 리소스를 해제한다. 입력 스트림을 닫은 후에 추가적인 읽기 시도가 있는 경우 IOException이 발생한다. 그러나 몇몇 종류의 스트림은 닫은 후에도 해당 스트림의 객체에 대해서 일부 작업이 허용된다. 예를 들어, java.security.DigestInputStream의 경우 일반적으로 데이터를 읽고 스트림을 종료한 후에야 메시지 다이제스트(message digest)를 얻을 수 있다.
