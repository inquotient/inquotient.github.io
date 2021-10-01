---
title:  데이터 스트림
categories:
- Java Network Programming
feature_text: |
  ## 데이터 스트림
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

DataInputStream과 DataOutputStream 클래스는 자바의 기본 데이터 타입과 바이너리 포맷의 문자열을 일고 쓰기 위한 메소드를 제공한다. 바이너리 형식은 두 개의 다른 자바 프로그램이 네트워크 연결이나 데이터 파일, 파이프, 또는 다른 매개체를 통해 데이터를 교환하기 위한 목적으로 주로 사용된다. 데이터 출력 스트림이 쓴 데이터는 데이터 입력 스트림이 읽을 수 있다. 그러나 그 바이너리 형식이 인터넷 프로토콜에서 바이너리 숫자를 교환하기 위해 사용하는 형식 중 하나와 같은 경우도 있다. 예를 들어, 타임(time) 프로토콜은 자바의 int 데이터 타입과 같은 32비트 빅 엔디안 정수를 사용한다. 부하 제어 네트워크 요소 서비스(controlled-load network element service)는 자바의 float 데이터 타입과 같은 32비트 IEEE 754 부동소수점수를 사용한다. (자바와 대부분의 네트워크 포로토콜은 유닉스 개발자에 의해 설계되었고, 결과적으로 둘 모두 대부분의 유닉스 시스템에서 일반적으로 사용되는 형식을 사용하는 경향이 있다.) 그러나 모든 네트워크 프로토콜이 이와 같이 바바의 기본 타입과 같지는 않으므로 사용할 프로토콜을 자세히 살펴봐야 한다. 예를 들어, 네트워크 다임 프로토콜(NTP, Network Time Protocol)은 64비트의 부호 없는 고정 수수형 숫자를 사용하여 시간을 표현한다. 64비트의 앞쪽 32비트는 정수 부분이고 뒤쪽 32비트는 소수 부분이다. 비록 이 타입이 NTP에서 꼭 필요하고, 다루기도 어렵지는 않지만 어떠한 일반적인 프로그래밍 언어에서도 기본 타입으로 제공하지 않는다.  

DataOutputStream 클래스는 특정 자바 데이터 타입을 출력하는 데 사용하는 다음과 같은 11가지 메소드를 제공한다.  

+ public final void writeBoolean(boolean b) throws IOException
+ public final void writeByte(int b) throws IOException
+ public final void writeShort(int s) throws IOException
+ public final void writeChar(int c) throws IOException
+ public final void writeInt(int i) throws IOException
+ public final void writeLong(long l) throws IOException
+ public final void writeFloat(float f) throws IOException
+ public final void writeDouble(double d) throws IOException
+ public final void writeChars(String s) throws IOException
+ public final void writeBytes(String s) throws IOException
+ public final void writeUTF(String s) throws IOException  

모든 데이터는 빅 엔디안 형식으로 써지고, 정수는 가능한 한 최소의 바이트만을 사용하여 2의 보수로 써진다. 따라서 byte 타입은 1바이트, short 타입은 2바이트, int 타입은 4바이트, long 타입은 8바이트로 써지며, float과 double 타입은 각각 4바이트와 8바이트의 IEEE 754 형태로 써진다. 또한 불리언(boolean)은 false인 경우 0, true인 경우 1의 값을 가지는 단일 바이트로 써지고, char 타입은 두 개의 부호 없는 바이트로 써진다.  

마지막 메소드 세 개에는 약간의 트릭이 숨어 있다. writeChars() 메소드는 단순히 문자열 인자에서 각각의 문자를 2바이트 빅 엔디안 유니코드 문자로 변경하여 쓰는 일을 반복한다. writeBytes() 메소드도 문자열 타입 인자를 처리하지만 각 문자의 최하위 바이트(LSB)만 쓴다. 따라서 Latin-1 문자 집합에 속하지 않는 문자를 포함한 문자열의 경우에 일부 정보가 손실된다. 이 메소드는 아스키 인코딩을 사용하는 네트워크 포로토콜에서 유용하게 사용될 수 있지만 대부분의 경우에 손실이 발생할 수 있으므로 사용을 피하는 것이 좋다.  

writeChars() 메소드나 writeBytes 메소드는 출력 스트림에 문자열의 길이를 인코딩하지 않는다. 그 결과, 문자 자체와 문자열의 일부를 구성하는 문자를 구별할 수 없다. writeUTF() 메소드는 문자열의 길이를 포함한다. writeUTF() 메소드는 문자열을 유니코드 UTF-8 변형(수정된 UTF-8 버전)으로 인코딩한다. 이 UTF-8 변형은 자바로 작성되지 않은 대부분의 소프트웨어와 미묘하게 호환되지 않기 때문에 DataInputStream을 사용하여 문자열을 읽는 다른 자바 프로그램과 데이터를 교환할 때만 사용해야 한다. 다른 소프트웨어와 UTF-8 텍스트를 교환해야 할 경우, 적절한 인코딩 타입과 함께 InputStreamReader를 사용해야 한다.  

물론, DataOutputStream은 이처럼 이진수와 문자열을 쓰기 위한 메소드 외에도 보통의 OutputStream 클래스가 제공하는 write(), flush(), close() 역시 제공한다.  

DataInputStream은 DataOutputStream에 대응되는 클래스다. DataOutputStream이 쓰는 모든 형식은 DataInputStream이 읽을 수 있다. 게다가 DataInputStream은 완전한 바이트 배열과 텍스트의 한 줄을 읽기 위한 메소드뿐만 아니라 보통의 read(), available(), skip(), close() 메소드를 제공한다.  

DataInputStream에는 바이너리 데이터를 읽기 위한 9개의 메소드를 제공하며, DataOutputStream이 제공하는 11개의 메소드에 대응한다. (정확히 말하면, writeBytes()와 writeChars()에 대응하는 메소드는 없다. 대신 한 번에 한 바이트나 한 문자를 읽어서 처리할 수 있다.)

+ public final boolean readBoolean() throws IOException
+ public final byte readByte() throws IOException
+ public final char readChar() throws IOException
+ public final short readShort() throws IOException
+ public final int readInt() throws IOException
+ public final long readLong() throws IOException
+ public final float readFloat() throws IOException
+ public final double readDouble() throws IOException
+ public final String readUTF() throws IOException  

또한 DataInputStream은 부호 없는 byte 타입과 부호 없는 short 타입을 읽고, 이에 대응하는 int 타입으로 반환하는 두 개의 메소드를 제공한다. 자바는 이러한 데이터 타입을 지원하지 않지만, 종종 C로 장성된 프로그램에 의해 만들어진 바이너리 데이터를 읽을 때 필요하다.  

+ public final String readUnsignedByte() throws IOException
+ public final String readUnsignedShort() throws IOException  

또한, DataInputStream은 요청된 바이트 수만틈 읽을 때까지 내장된 입력 스트림에서 배열로 반복해서 데이터를 읽는 두 개의 readFully() 메소드를 제공한다. 충분한 데이터를 읽지 못할 경우 IOException이 발생한다. 이 메소드는 정확히 얼마만큼의 데이터를 읽을 수 있는지 미리 알고 있을 때 특히 유용하다. 예를 들어, HTTP 요청의 응답을 처리할 때 HTTP 헤더의 Content-length 필드를 읽어서 본문의 길이를 미리 알 수 있으므로 readFully() 메소드를 유용하게 사용할 수 있다.  

+ public final int read(byte[] input) throws IOException
+ public final int read(byte[] input, int offset, int length) throws IOException
+ public final int readFully(byte[] input) throws IOException
+ public final int readFully(byte[] input, int offset, int length) throws IOException  

마지막으로, DataInputStream은 텍스트로부터 한 줄을 읽는 데 널리 사용되는 readLine() 메소드를 제공하며, 이 메소드는 줄 구분자로 나뉜 한 줄을 읽고서 문자열로 반환된다.  

+ public final String readLine() throws IOException  

그러나 readLine() 메소드는 (중요도가 떨어져) 곧 지원이 중단될 예정이며, 자체적인 버그도 존재한다. 그래서 어떤 상황에서도 사용해서는 안 된다. 지원 중단에 가장 큰 영향을 미친 것은 readLine() 메소드가 주로 아스키가 아닌 문자를 바이트로 올바르게 변환하지 못했기 때문이다. 지금은 이 일을 BufferedReader 클래스의 readLine() 메소드가 대신하고 있다. 그러나 이 두 메소드는 단일 캐리지리턴으로 끝나는 줄을 인식하지 못하는 공통된 문제가 있다. 대신 readLine() 메소드는 단일 라인피드 또는 캐리지리턴/라인피드 쌍으로 된 줄 구분자만을 인식한다. readLine() 메소드는 스트림에서 캐리지리턴이 발견되면 다음 문자가 라인피드인지 검사하기 위해 대기한다. 다음 문자가 라인피드라면, 캐리지리턴과 라인피드를 날려 버리고 이 줄을 문자열로 반환한다. 반대로 다음 준자가 라인피드가 아니라면, 캐리지리턴을 날려 버리고 이 줄을 문자열로 반환한다. 그리고 라인피드를 찾기 위해 읽은 문자는 다음 행의 시작에 포함시킨다. 그러나 캐리지리턴이 스트림의 마지막 문자인 경우, readLine() 메소드는 오지도 않을 다음 문자를 무한히 대기한다.  

파일을 읽을 때는 파일의 끝이 아닌 이상 항상 다음 문자를 읽을 수 있고, 더 이상 읽을 수 없는 스트림의 끝에서는 -1이 반환되기 때문에 이 문제가 명확하게 노출되지 않는다. 그러나 FTP나 최근 HTTP 모델에서 사용하는 지속형 연결 방식에서 서버와 클라이언트는 요청의 마지막 문자를 보낸 다음, 연결을 종료하지 않고 응답을 대기하기 때문에 이 문제에 쉽게 노출된다. 운이 좋다면, 몇 분이 걸리긴 하겠지만 타임 아웃에 의해 연결이 끊어지고 IOException이 발생할 것이다. 거기에 스트림의 마지막 라인이 손실될 수도 있다. 만일 운이 나쁘다면, 프로그램은 무한 대기 상태가 된다.  
