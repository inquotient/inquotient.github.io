---
title:  필터 연결하기
categories:
- Java Network Programming
feature_text: |
  ## 필터 연결하기
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

```java
FileInputStream fin = new FileInputStream("data.txt");
BufferedInputStream bin = new BufferedInputStream(fin);
```

```java
InputStream in = new FileInputStream("data.txt");
in = new BufferedInputStream(in);
```

```java
DataOutputStream in = new DataOutputStream(new BufferedOutputStream(new FileOutputStream("data.txt")));
```

필터 체인에 있는 여러 필터의 메소드를 사용해야 할 때까 있다. 예를 들어, 유니코드 텍스트 파일을 읽고 있다면 파일의 인코딩 종류가 빅 엔디안 UCS-2인지 리틀 엔디안 UCS-2인지, 아니면 UTF-8인지 판단하기 위해 파일의 처음 3바이트에 있는 바이트 순서 표식(BOM, Byte Order Mark)을 읽고 나서 해당하는 인코딩의 Reader 필터를 선택한다. 또는 웹 서버에 접속 중이라면 콘텐츠 인코딩(Content-encoding) 정보를 찾기 위해 서버가 보낸 헤더를 읽고, 웹 서버가 보낸 본문을 읽기 위해 필요한 reader를 고르는 데 헤더에서 읽은 콘텐츠 인코딩 정보를 사용할 수 있다. 또는 DataOutputStream을 사용하여 부동소주점수를 네트워크로 전송해야 할 경우가 있다. 이때 DataOutputStream에 연결된 DigestOutputStream에서 MessageDigest를 가져올 수 있다. 이와 같은 경우에는 내장된 스트림의 레퍼런스를 사용하고 저장할 필요가 있다. 그러나 이외의 상황에서는 필터 체인의 마지막 필터 외에 무언가를 쓰거나 읽는 상황이 발생하지 않을 것이다.
