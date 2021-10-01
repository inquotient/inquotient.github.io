---
title:  필터 스트림
categories:
- Java Network Programming
feature_text: |
  ## 필터 스트림
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

InputStream과 OutputStream은 매우 원시적인 클래스다. 두 클래스는 단일 바이트나 그룹 바이트를 읽고 쓰는데, 이것이 두 클래스가 하는 일의 전부다. 읽고 쓰는 바이트의 의미가 정수인지 IEEE 754 부동소수점인지 아니면 유니코드 텍스트인지를 결정하는 것은 전적으로 프로그래머와 코드에 달려 있다. 그러나 몇몇 데이터 타입의 경우 그 형태가 매우 일반적이기 때문에 클래스 라이브러리로 구현해 놓으면 유용하게 사용되는 경우가 있다. 예를 들어, 네트워크 프로토콜로 전송되는 대부분의 정수는 32비트 빅엔디안을 사용하고, 웹을 통해 전달되는 대부분의 텍스트는 7비트 아스키나, 8비트 Latin-1 또는 멀티바이트 UTF-8로 되어 있다. 그리고 FTP에 의해 전달되는 많은 파일들은 ZIP 형식으로 저장되어 있다. 자바는 raw 바이트를 이러한 데이터 형식으로 변환하기 위해 저수준 스트림에 연결할 수 있는 많은 필터 클래스를 제공한다.  

필터는 필터 스트림, reader, writer로 나뉜다. 필터 스트림은 기본적으로 raw 바이트를 대상으로 동작한다. 예를 들어, 데이터를 압축하거나 이진수로 변환하는 일들이 포함된다. reader와 writer는 UTF-8과 ISO 8859-1 같은 형식으로 인코딩된 특수한 텍스트를 처리하는데 사용된다.  

필터의 연결 구조에서 각 링크는 이전의 필터나 스트림으로부터 데이터를 잔달받고 다음 연결된 링크로 전달한다.  

모든 필터 출력 스트림은 java.io.OutputStream 같은 write(). close(), flush() 메소드를 제공한다. 또한 모든 필터 입력 스트림은 java.io.InputStream 같은 read(), close(), available() 메소드를 제공한다. BufferedInputStream과 BufferedOutputStream 같은 몇몇 스트림의 경우, 위의 메소드 이외에 추가적인 메소드를 제공하지 않는다. 필터링은 순수하게 스트림 내부에서 이뤼지며 public으로 선언된 어떠한 인터페이스도 제공하지 않는다. 그러나 대부분의 경우에 필터 스트림은 부가적인 목적으로 public 메소드를 추가하여 제공한다. 때로 PushbackInputStream의 unread() 같은 메소드가 일반적인 read(), write() 메소드와 함께 사용될 목적으로 추가된다. 그리고 경우에 따라, 추가된 메소드가 기존의 인터페이스를 대체하기도 한다. 예를 들어, PrintStream의 경우 write() 메소드보다는 상대적으로 print()나 println() 메소드가 더 자주 쓰인다.
