---
title:  IP 멀티캐스트
categories:
- Java Network Programming
feature_text: |
  ## IP 멀티캐스트
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

### 1. 멀티캐스팅
<br/>
#### 1.1. 멀티캐스트 주소와 그룹
<br/>
멀티캐스트 주소는 멀티캐스트 그룹이라고 불리는 호스트 그룹의 공유 주소이다. 먼저 멀티캐스트 주소를 살펴보겠다. IPv4 멀티캐스트 주소는 CIDR 그룹 224.0.0.0/4 안의 IP 주소들이다. 즉, 224.0.0.0부터 239.255.255.255 범위의 IP 주소들이다. 이 범위의 모든 주소는 처음 네 개의 비트가 이진수 1110으로 시작한다. IPv6 멀티캐스트 주소는 CIDR 그룹 ff00::/8 안의 IP 주소들이다(즉, 0xFF 바이트 또는 이진수로 11111111으로 시작하는 모든 주소가 포함된다). IP 주소처럼 멀티캐스트 주소도 호스트네임을 갖는다. 예를 들어, 멀티캐스트 주소 224.0.1.1에는 (분산 네트워크 타임 프로토콜 서비스의 주소) 이름 ntp.mcast.net이 할당되어 있다.  

멀티캐스트 그룹이란 하나의 멀티캐스트 주소를 공유하는 인터넷 호스트들의 집합이다. 멀티캐스트 주소로 전송된 임의의 데이터는 그룹의 모든 멤버에게 전달된다. 멀티캐스트 그룹에게 가입할 권한은 누구에게나 있다. 호스트는 아무 때나 그룹에 가입하고 탈퇴할 수 있다. 그룹은 영원히 지속될 수 있고 일시적일 수도 있다. 영원히 지속되는 그룹은 그 그룹에 속한 멤버가 있든 없든 간에 항상 일정한 주소를 핟당한다. 그러나 대부분의 멀티캐스트 그룹은 일시적인 것이고 그룹에 속한 멤버가 있을 때만 존재한다. 새로운 멀티캐스트 그룹을 생성하기 위해 해야 하는 일은 225.0.0.0부터 238.255.255.255까지의 주소 중에서 임의의 주소를 선택하여 그 주소에 대한 InetAddress 객체를 생성하고 해당 주소로 데이터 전송을 시작하는 것이다.  

영원히 지속되는 멀티캐스트 그룹의 주소는 IANA(Internet Assigned Numbers Authority)에 요청하여 받을 수 있다. 지금까지는 수백여 개의 주소가 할당되어 있다. 링크로컬 멀티캐스트 주소는 224.0.0(즉, 224.0.0.0부터 224.0.0.25까지)으로 시작하며, 라우팅 프로토콜과 게이트웨이 탐지 및 그룹 멤버 리포팅 같은 저수준 동작을 위한 용도로 예약되어 있다. 예를 들어, all-systems.mcast.net 이름의 주소 224.0.0.1은 로컬 서브넷에 있는 모든 시스템을 포함하는 멀티캐스트 그룹이다. 멀티캐스트 라우터는 이 범위에 있는 목적지로는 데이터그램을 전달하지 않는다.  

<table>
  <thead>
    <tr>
      <td>도메인 네임</td>
      <td>IP 주소</td>
      <td>목적</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>BASE-ADDRESS.MCAST.NET</td>
      <td>224.0.0.0</td>
      <td>예약된 베이스 주소. 어떤 멀티캐스트 그룹에도 할당되지 않는다.</td>
    </tr>
    <tr>
      <td>ALL-SYSTEMS.MCAST.NET</td>
      <td>224.0.0.1</td>
      <td>로컬 서브넷의 모든 시스템</td>
    </tr>
    <tr>
      <td>ALL-ROUTERS.MCAST.NET</td>
      <td>224.0.0.2</td>
      <td>로컬 서브넷의 모든 라우터</td>
    </tr>
    <tr>
      <td>DVMRP.MCAST.NET</td>
      <td>224.0.0.4</td>
      <td>해당 서브넷에 연결된 DVMRP(Distance Vector Multicast Routing Protocol) 라우터</td>
    </tr>
    <tr>
      <td>MOBILE-AGENTS.MCAST.NET</td>
      <td>224.0.0.11</td>
      <td>로컬 서브넷의 모바일 에이전트</td>
    </tr>
    <tr>
      <td>DHCP-AGENTS.MCAST.NET</td>
      <td>224.0.0.12</td>
      <td>이 멀티캐스트 그룹은 클라이언트를 로컬 서브넷에 있는 DHCP 서버나 전달 에이전트(relay agent)에 넣을 수 있도록 한다.</td>
    </tr>
    <tr>
      <td>RSVP-ENCAPSULATION.MCAST.NET</td>
      <td>224.0.0.14</td>
      <td>서브넷에 RSVP 캡슐화. RSVP(Resource reSerVation setup Protocol)는 데이터를 전손하기 전에 인터넷 대역폭을 미리 예약한다.</td>
    </tr>
    <tr>
      <td rowspan="12">VRRP.MCAST.NET</td>
      <td>224.0.0.18</td>
      <td>VRRP(Virtual Router Redundancy Protocol)라우터</td>
    </tr>
    <tr>
      <td>224.0.0.35</td>
      <td>DXCluster는 외부 아마추어 방송국을 알리기 위해 사용된다.</td>
    </tr>
    <tr>
      <td>224.0.0.36</td>
      <td>DVD 플레이어. 텔레비전과 유사 장치들 사이에서 암호화하는 DTCP(Digital Transmission Content Protection), DRM(digital restrictions management) 기술</td>
    </tr>
    <tr>
      <td>224.0.0.37-224.0.0.68</td>
      <td>자동 주소 할당(zeroconf addressing)</td>
    </tr>
    <tr>
      <td>224.0.0.106</td>
      <td>멀티캐스트 라우터 탐색</td>
    </tr>
    <tr>
      <td>224.0.0.112</td>
      <td>MMA(Multipath Management Agent) 장치 탐색</td>
    </tr>
    <tr>
      <td>224.0.0.113</td>
      <td>퀄컴의 올조인(AllJoyn)</td>
    </tr>
    <tr>
      <td>224.0.0.114</td>
      <td>RFID 간의 reader 프로토콜</td>
    </tr>
    <tr>
      <td>224.0.0.251</td>
      <td>멀티캐스트 DNS에서 사용하며, 멀티캐스트 주소에 대해 스스로 호스트 네임을 할당하고 검색한다.</td>
    </tr>
    <tr>
      <td>224.0.0.252</td>
      <td>멀티캐스트 DNS(mDNS)의 가장 널리 사용되는 방식인 LLMNR(Link-local Multicast Name Resolution)에서 사용하는 주소이며, 로컬 네트워크 내에서 스스로 도메인 네임을 할당하고 검색할 수 있도록 한다.</td>
    </tr>
    <tr>
      <td>224.0.0.253</td>
      <td>Teredo는 IPv4를 통해 IPv6로 터널링을 위해 사용되며, 같은 IPv4 서브넷에 위치하는 Teredo 클라이언트는 이 멀티캐스트 주소로 응답해야 한다.</td>
    </tr>
    <tr>
      <td>224.0.0.254</td>
      <td>실험용</td>
    </tr>
  </tbody>
</table>
<br/><br/>

영구적으로 할당된 멀티캐스트 주소들은 224.1 또는 224.2로 시작하는 로컬 서브넷을 넘어 확장된다. 다음 표는 이러한 영구 주소들 중 일부를 나열한다. 수십 개 수천 개까ㅓ지 다양한 범위를 가진 주소 블록들이 특별한 목적을 위해 예약되어 있다. 전체 목록은 iana.org에서 확인할 수 있으며, 이 목록에 있는 것 중 상당수는 더 이상 존재하지 않는 서비스, 프로토콜 그리고 회사들이므로 주의해서 봐야 한다. 남아 있는 2억 4,800만 개의 멀티캐스트 주소들은 일시적인 목적으로 누구나 사용할 수 있다. 멀티캐스트 라우터는 다른 두 시스템이 동시에 같은 주소를 사용하지 않도록 관리해야 한다.  

<table>
  <thead>
    <tr>
      <td>도메인 네임</td>
      <td>IP 주소</td>
      <td>목적</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>NTP.MCAST.NET</td>
      <td>224.0.1.1</td>
      <td>네트워크 타임 프로토콜</td>
    </tr>
    <tr>
      <td>NSS.MCAST.NET</td>
      <td>224.0.1.6</td>
      <td>네임 서비스 서버</td>
    </tr>
    <tr>
      <td>AUDIONEWS.MCAST.NET</td>
      <td>224.0.1.7</td>
      <td>오디오 뉴스 멀티캐스트</td>
    </tr>
    <tr>
      <td>MTP.MCAST.NET</td>
      <td>224.0.1.9</td>
      <td>멀티캐스트 전송 프로토콜</td>
    </tr>
    <tr>
      <td>IETF-1-LOW-AUDIO.MCAST.NET</td>
      <td>224.0.1.10</td>
      <td>IETF의 저음질 오디오 채널1</td>
    </tr>
    <tr>
      <td>IETF-1-AUDIO.MCAST.NET</td>
      <td>224.0.1.11</td>
      <td>IETF의 고음질 오디오 채널1</td>
    </tr>
    <tr>
      <td>IETF-1-VIDEO.MCAST.NET</td>
      <td>224.0.1.12</td>
      <td>IETF의 비디오 채널1</td>
    </tr>
    <tr>
      <td>IETF-2-LOW-AUDIO.MCAST.NET</td>
      <td>224.0.1.10</td>
      <td>IETF의 저음질 오디오 채널2</td>
    </tr>
    <tr>
      <td>IETF-1-AUDIO.MCAST.NET</td>
      <td>224.0.1.11</td>
      <td>IETF의 고음질 오디오 채널2</td>
    </tr>
    <tr>
      <td>IETF-1-VIDEO.MCAST.NET</td>
      <td>224.0.1.12</td>
      <td>IETF의 비디오 채널2</td>
    </tr>
    <tr>
      <td>MLOADD.MCAST.NET</td>
      <td>224.0.1.19</td>
      <td>MLOADD는 몇 초 동안 여러 개의 네트워크 인터페이스의 트래픽 부하를 측정한다.</td>
    </tr>
    <tr>
      <td>EXPERIMENT.MCAST.NET</td>
      <tr>
        <td>224.0.1.20</td>
        <td>실험용</td>
      </tr>
      <tr>
        <td>224.0.23.178</td>
        <td>JDP(Java Discovery Protocol). 네트워크에서 관리할 수 있는 JVM을 찾는 프로토콜</td>
      </tr>
    </tr>
    <tr>
      <td>MICROSOFT.MCAST.NET</td>
      <td>224.0.1.24</td>
      <td>WINS(Windows Internet Name Service) 서버가 서로를 찾기 위해 사용된다.</td>
    </tr>
    <tr>
      <td>MTRACE.MCAST.NET</td>
      <td>224.0.1.32</td>
      <td>traceroute의 멀티캐스트 버전</td>
    </tr>
    <tr>
      <td>JINI-ANNOUNCEMENT.MCAST.NET</td>
      <td>224.0.1.84</td>
      <td>JINI 공지</td>
    </tr>
    <tr>
      <td>JINI-REQUEST.MCAST.NET</td>
      <td>224.0.1.85</td>
      <td>JINI 요청</td>
    </tr>
    <tr>
      <td></td>
      <td>224.0.1.143</td>
      <td>EMWIN(Emergency Managers Weather Information Network) 미국에서 날씨 정보를 배포하기 위해 사용</td>
    </tr>
    <tr>
      <td></td>
      <td>224.2.0.0-224.2.255.255</td>
      <td>MBONE 주소는 멀티미디어 회의를 위해 예약되어 있다. 즉, 여러 사람이 함께 오디오와 비디오, 화이트보드, 웹 브라우저를 공유한다.</td>
    </tr>
    <tr>
      <td></td>
      <td>224.2.2.2</td>
      <td>이 주소의 9875번 포트는 현재 이용 가능한 MBONE 프로그래밍을 브로드캐스팅하는 데 사용된다. X윈도우 유틸리티인 sdr이나 윈도우/유닉스의 멀티키트(multikit) 프로그램으로 이것을 살펴볼 수 있다.</td>
    </tr>
    <tr>
      <td></td>
      <td>239.0.0.0-239.255.255.255</td>
      <td>TTL을 이요하여 멀티캐스트 패킷을 전송 범위를 제한하는 것이 아니라, 특정 범위의 주소가 일정한 지역이나 라우터 그룹을 벗어나지 않도록 제한한다. 이것을 관리영역이라 부른다. 예를 들어, UPnP(Universal Plug and Play) 장치는 네트워크에 가입하기 위해 HTTPU(UDP 기반의 HTTP) 메시지를 멀티캣트 주소 239.255.255.250의 포트 1900으로 보낸다. 이런 기법을 사용하면 TTL 값에 의존하지 않고 그룹을 미리 만들 수 있다.</td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 1.2. 클라이언트와 서버
<br/>
호스트가 멀티캐스트 그룹으로 데이터를 전송하려고 한다면, 호스트는 데이터를 멀티캐스트 데이터그램에 넣는다. 이 방법은 멀티캐스트 그룹 주소로 설정된 UDP 데이터그램 그 이상은 아니다. 멀티캐스트 데이터는 신뢰할 수 없는 UDP를 통해 전송되며, 연결 기반 TCP를 통해 데이터를 전송하는 것보다 세 배 정도 빠르다. (TCP를 통한 멀티캐스트는 불가능하다. TCP는 호스트가 패킷을 수신하면 ACK를 보내도록 한다. 멀티캐스트에서 ACK를 처리하는 것은 거의 재난에 가깝다.) 데이터 손실을 허용하지 않는 멀티캐스트 애플리케이션을 작성한다면 전송되는 도중에 데이터가 손상되었는지를 확인하고 손실된 데이터를 어떻게 처리할 것인지를 결정하는 것은 프로그래머의 몫이다. 예를 들어, 분산 캐시 시스템을 구성하고 있다면, 수신하지 못한 파일은 그냥 두고 캐시에서 가져오도록 결정할 수 있다.  

앞서 말했듯이 애플리케이션 프로그래머의 관점에서 멀티캐스팅과 보통의 UDP 소켓의 가장 큰 차이점은 TTL 값을 고려해야 한다는 것이다. 이 값은 1에서 255까지의 값을 가지는 IP 헤더에 있는 단일 바이트이다. TTL은 해당 패킷이 버려지기 전까지 지나갈 수 있는 대략적인 라우터의 개수를 의미한다. 각각의 패킷은 라우터를 통과할 때마다 TTL 필드가 1씩 감소한다. 일부 라우터의 경우 2 이상의 값을 감소시키기도 한다. TTL 값이 0이 되면 해당 패킷은 버려진다. TTL 필드는 모든 패킷은 언젠가는 결국 버려진다는 것을 보장함으로써 패킷이 루프를 도는 것을 방지하기 위해 설계되었다. TTL은 잘못 설정된 라우터가 서로에게 무한히 패킷을 주고받는 상황을 예방한다. IP 멀티캐스팅에서 TTL은 멀티캐스트 패킷을 지리적으로 제한하기 위해서 TTL 값을 사용한다. 예를 들어, 일반적으로 TTL 값을 사용한다. 예를 들어, 일반적으로 TTL 값 16은 해당 패킷을 로컬 영역으로 제한하며 TTL 값 127은 패킷을 전 세계로 보낸다. 물론 두 수의 중간값도 사용할 수 있다. 그러나 TTL 값을 지리적인 거리로 정확하게 환산하는 방법은 없다. 일반적으로 사이트가 멀리 떨어질수록 중간에 거쳐야 하는 라우터의 개수가 많아지기 때문에 TTL 값이 작은 패킷은 TTL 값이 큰 패킷만큼 멀리 전달하지 못한다. 다음은 TTL 값의 지리적인 도달 범위의 대략적인 예측을 제공한다. 224.0.0.0부터 224.0.0.255까지의 멀티캐스트 그룹으로 전송되는 패킷은 사용된 TTL 값에 상관없이 절대 로컬 서브넷을 벗어나지 못한다.  

+ 로컬 호스트 : 0
+ 로컬 서브넷 : 1
+ 여러 개의 LAN에 연결된 로컬 캠퍼스 : 16
+ 동일 국가에 있는 대역폭이 높은 사이트. 보통 이런 사이트는 백본(backbone) 가까이에 있다. : 32
+ 동일 국가의 모든 사이트 : 48
+ 동일 대륙에 있는 모든 사이트 : 64
+ 전 세계에 있는 대역폭이 높은 사이트 : 128
+ 전 세계의 모든 사이트 : 255  

일단 데이터가 데이터그램에 채워지면 송신 호스트는 데이터그램을 인터넷으로 내보낸다. 이것은 마치 일반 UDP 데이터를 송신하는 것과 같다. 좀 더 자세히 살펴보면, 송신 호스트는 멀티캐스트 데이터그램을 로컬 네트워크로 보낸다. 이 패킷은 같은 서브넷에 있는 멀티캐스트 그룹의 모든 멤버들에게 즉시 전달된다. 만약 패킷의 TTL 값이 1보다 크면 로컬 네트워크의 멀티캐스트 라우터는 해당 멀티캐스트 그룹의 멤버를 가지고 있는 다른 네트워크로 패킷을 전달한다. 패킷이 최종 목적지에 도착하면 외부 네트워크에 있는 멀티캐서 라우터는 멀티캐스터 그룹에 가입한 각 호스트에게 패킷을 전송한다. 패킷이 최종 목적지에 도착하면 외부 네트워크에 있는 멀티캐스터 라우터는 멀티캐스터 그룹에 가입한 각 호스트에게 패킷을 전송한다. 필요한 경우, 멀티캐스트 라우터는 또한 현재 라우터와 최종 목적지 사이의 경로에 있는 다음 라우터에게 패킷을 재전송한다.  

데이터가 멀티캐스트 그룹에 가입한 호스트에 도착하면, 호스트는 패킷의 목적지 주소가 호스트의 주소와 맞지 않더라도, 다른 UDP 데이터그램을 수신하는 것처럼 데이터를 받는다. 호스트는 패킷에 표시된 멀티캐스트 그룹에 가입했다는 것을 알고 있기 때문에 주소가 달라도 데이터그램이 제대로 도착하였음을 알 수 있다. 이는 마치 자신의 이름이 '거주지'는 아니지만, '거주자'라고 찍힌 우편물을 그 주택의 거주자가 수신하는 것과 같다. 수신 호스트는 적당한 포트에서 대기하고 있다가 데이터그램이 도착하면 처리할 수 있도록 준비해야 한다.  

#### 1.3. 라우터와 라우팅
<br/>
멀티캐스트 소켓은 인터넷으로 하나의 데이터스트림을 클라이언트와 라우터와 보낸다. 그러면 라우터는 데이터를 복사하여 각 클라이언트로 보낸다. 멀티캐스트 소켓을 사용하지 않는다면 서버는 똑같은 데이터를 네 번씩이나 라우터로 보내야 하고 라우터는 데이터를 각 클라이언트로 보낼 것이다. 스트림 하나를 사용하여 데이터를 여러 개의 클라이언트로 전송하면 인터넷 백본(backbone)의 대역폭을 절약할 수 있다.  

물론 실제 경로에는 중복된 라우터가 다단계 계층으로 구성되어 있으므로 훨씬 복잡하다. 하지만 멀티캐스트 소켓의 목적은 간단하다. 네트워크가 아무리 복잡해도 동일한 데이터를 절대로 한 번 이상 보내지 않는 것이다. 다행히도 프로그래머는 라우팅에 관련된 것들은 신경 쓰지 않아도 된다. 그냥 MulticastSocket을 만들고 소켓을 멀티캐스트 그룹에 가입시킨다. 그리고 전송하려는 DatagramPacket에 멀티캐스트의 주소를 채워 넣는다. 나머지 일은 라우터와 MulticastSocket이 알아서 처리한다.  

멀티캐스팅에서는 라우터가 멀티캐스트를 지원해야 한다. 멀티캐스트 라우터는 일반적인 인터넷 라우터나 워크스테이션을 IP 멀티캐스트가 가능하도록 확장하여 재구성한다. 개인고객에게 인터넷 서비스를 제공하는 ISP에서는 일부러 라우터가 멀티캐스팅을 하지 못하도록 설정한다.  

멀티캐스트 데이터를 로컬 서브넷을 넘어 주고받으려면 멀티캐스트 라우터가 필요하다. 네트워크 관리자에게 문의하여 라우터가 멀티캐스팅을 지원하는가를 확인해야 한다. 또는 all-routers.mcast.net으로 ping을 실행해 본다. 임의의 라우터가 응답을 한다면 멀티캐스트 라우터가 연결되어 있다는 것을 의미한다.  

하지만 그렇다고 해서 인터넷의 모든 멀티캐스팅이 가능한 호스트와 데이터를 주고받을 수 있는 것은 아니다. 여러분이 보낸 패킷이 어떤 호스트로 전송되려면 두 호스트 사이에 멀티캐스팅을 지원하는 라우터가 있어야 한다. 그러나 어떤 사이트에서는 모든 라우터가 이해할 수 있도록 멀티캐스트 데이터를 유니캐스트의 UDP 데이터그램으로 터널링하는 소프트웨어를 사용하여 연결되어 있을 수도 있다.  

### 2. 멀티캐스트 소켓 다루기
<br/>
자바에서는 java.net.MulticastSocket 클래스를 사용하여 데이터를 멀티캐스트하며, 이 클래스는 java.net.DatagramSocket의 서브클래스이다.  

+ public class MulticastSocket extends DatagramSocket implements Closeable, AutoCloseable  

이미 예상할 수 있듯이 MulticastSocket의 동작은 DatagramSocket과 매우 유사하다. MulticastSocket을 이용하여 주고받으려는 데이터를 DatagramPacket 객체에 넣는다. 따라서 기본적인 내용은 다시 반복해서 설명하지 않는다.  

원격 사이트로부터 멀티캐스트 데이터를 수신하기 위해 MulticastSocket() 생성자로 MulticastSocket을 만든다. 다른 종류의 소켓과 마찬가지로 대기할 포트를 알아야 한다. 다음 코드는 포트 2300에 대기하는 MulticastSocket을 연다.  

```java
MulticastSocket ms = new MulticastSocket(2300);
```

다음으로, MulticastSocket의 joinGroup() 메소드를 사용하여 멀티캐스트 그룹에 가입한다.  

```java
InetAddress group = InetAddress.getByName("224.2.2.2");
ms.joinGroup(group);
```

로컬 호스트와 서버 사이에 있는 라우터들에게 브로드캐스트 패킷 전송의 시작을 알린다. 그리고 또한 로컬 호스트에게는 해당 멀티캐스트 그룹으로 전송되는 IP 패킷을 여러분에게 전달해 줘야 함을 알린다.  

일단 멀티캐스트 그룹에 가입하면 DatagramSocket을 다루던 방식으로 UDP 데이터를 수신한다. 즉, 데이터를 저장하기 위해 버퍼로 사용할 바이트 배열로 DatagramPacket을 생성한다. 그리고 DatagramSocket 클래스에서 상속한 receive() 메소드를 호출하여 데이터를 수신하는 루프로 들어간다.  

```java
byte[] buffer = new byte[8192];
DatagramPacket dp = new DatagramPacket(buffer, buffer.length);
ms.receive(dp);
```

더 이상 데이터 수신을 원하지 않는 경우 소켓의 leaveGroup() 메소드를 호출하여 멀티캐스트 그룹에서 떠날 수 있다. 그러고 난 다음 DatagramSocket에서 상속된 close() 메소드를 호출하여 소켓을 닫는다.  

```java
ms.leaveGroup(group);
ms.close();
```

멀티캐스트 주소로 데이터를 전송하는 방법은 유니캐스트 주소로 UDP 데이터를 전송하는 것과 크게 다르지 않다. 멀티캐스트 그룹에 가입하지 않아도 데이터를 전송할 수 있다. DatagramPacket 객체를 생성하고 전송할 데이터와 멀티캐스트 그룹의 주소로 패킷을 채운다. 그리고 멀티캐스트 소켓의 send() 메소드에 전달한다.  

```java
InetAddress ia = InetAddress.getByName("experiment.mcast.net");
byte[] data = "Here's some multicast data\r\n".getBytes("UTF-8");
int port = 4000;
DatagramPacket dp = new DatagramPacket(data, data.length, ia, port);
MulticastSocket ms = new MulticastSocket();
ms.sned(dp);
```

멀티캐스트 소켓은 보안에 허술하므로 주의해야 한다. 따라서 SecurityManager의 제어하에 실행되는 신뢰되지 않은 코드는 멀티캐스트 소켓과 관련된 어떠한 작업도 할 수 없다. 원격에서 로드된 코드는 일반적으로 자신이 다운로드된 호스트에 대해서만 데이터그램을 보내거나 받을 수 있다. 그러나 멀티캐스트 소켓은 이런 제약 사항에 대해 패킷에 어떤 표시도 할 수 없다. 일단 데이터를 멀티캐스트 소켓으로 전송하면 어느 호스트가 그 데이터를 받아야 하는지 혹은 받아서는 안 되는지를 관리하기가 매우 힘들고 신뢰성도 없다. 따라서 원격 코드를 실행하는 환경에서는 모든 멀티캐스팅을 허용하지 않는 보수적인 접근 방법을 취한다.  

#### 2.1. 생성자
<br/>
생성자는 간단하다. 여러분이 직접 대기할 포트를 선택하거나 자바가 임의의 포트를 할당해 준다.  

+ public MulticastSocket() throws SocketException
+ public MulticastSocket(int port) throws SocketException
+ public MulticastSocket(SocketAddress bindAddress) throws IOException  

예를 들어:  

```java
MulticastSocket ms1 = new MulticastSocket();
MulticastSocket ms2 = new MulticastSocket(4000);
SocketAddress address = new InetSocketAddress("192.168.254.32", 4000);
MulticastSocket ms3 = new MulticastSocket(address);
```

이 세 개의 생성자는 모두 소켓을 생성할 수 없는 경우 SocketException 예외를 발생시킨다. 포트를 바인드할 적절한 권한이 없거나 바인드할 포트가 이미 사용 중인 경우 소켓을 생성할 수 없다. 운영체제가 관여하는 한 멀티캐스트 소켓은 데이터그램 소켓과 동일하므로, MulticastSocket은 DatagramSocket이 이미 사용 중인 포트를 사용할 수 없으며 반대도 마찬가지다.  

생성자에 인자로 널(null)을 전달하면 바인드되지 않은 소켓이 생성되며, 나중에 bind() 메소드를 사용하여 연결할 수 있다. 이 생성자는 소켓을 바인드하기 전에 설정해야 하는 소켓 옵션을 설정할 때 유용하게 사용된다. 다음 코드는 SO&#95;REUSEADDR 옵션을 끈 멀티캐스트 소켓을 생성한다. (이 옵션은 보통 멀티캐스트 소켓에 대해 기본적으로 켜 있다.)  

```java
MulticastSocket ms = new MulticastSocket(null);
ms.setReuseAddress(false);
SocketAddress address = new InetSocketAddress(4000);
ms.bind(address);
```

#### 2.2. 멀티캐스트 그룹과 통신하기
<br/>
일단 MulticastSocket이 생성되면, 다음과 같은 네 가지 동작을 수행할 수 있다.  

(1) 멀티캐스트 그룹에 가입하기  
(2) 그룹의 멤버들에게 데이터 보내기  
(3) 그룹에서 데이터 받기  
(4) 멀티캐스트 그룹 탈퇴하기  

MulticastSocket 클래스는 1번과 4번에 사용될 메소드를 제공한다. 데이터를 보내거나 받기 위한 새로운 메소드는 필요하지 않으며, 슈퍼클래스인 DatagramSocket이 제공하는 send()와 receive() 메소드를 사용하면 충분하다. 그룹에서 데이터를 수신하기 전에 먼저 가입을 해야 한다는 것만 제외하면, 위 동작들은 순서에 상관없이 수행할 수 있다. 그룹에 가입하지 않아도 데이터를 전송할 수 있으며 전송과 수신을 자유롭게 섞어서 쓸 수 있다.  

##### 2.2.1. 그룹에 가입하기
<br/>
그룹에 가입하기 위해 해당 멀티캐스트 그룹에 대한 InetAddress나 SocketAddress를 joinGroup() 메소드에 전달한다.  

+ public void joinGroup(InetAddress address) throws IOException
+ public void joinGroup(SocketAddress address, NetworkInterface interface) throws IOExeption  

멀티캐스트 그룹에 가입하고 나면, 유니캐스트 데이터그램을 수신하는 방법과 동일한 방법으로 데이터그램을 수신한다. 즉, 버퍼로 DatagramPacket을 생성하고 이것을 소켓의 receive() 메소드로 전달한다. 예를 들어:  

```java
try {
  MulticastSocket ms = new MulticastSocket(4000);
  InetAddress ia = InetAddress.getByName("224.2.2.2");
  ms.joinGroup(ia);
  byte[] buffer = new byte[8192];
  while (true) {
    DatagramPacket dp = new DatagramPacket(buffer, buffer.length);
    ms.receive(dp);
    String s = new String(dp.getData(), "8859_1");
    System.out.println(s);
  }
} catch (IOException ex) {
  System.err.println(ex);
}
```

가입하려는 주소가 멀티캐스트 주소가 아닌 경우(224.0.0.0부터 239.255.255.255 사이의 주소가 아닌 경우), joinGroup() 메소드는 IOExeption 예외를 발생시킨다.  

하나의 MulticastSocket은 다수의 멀티캐스트 그룹에 가입할 수 있다. 멀티캐스트 그룹의 멤버십 정보는 멀티캐스트 소켓 객체가 아닌 멀티캐스트 라우터에 저장된다. 이 경우에 해당 패킷이 어떤 주소로 전달되어야 하는지 결정하기 위해 수신된 데이터그램 안에 저장된 주소를 사용한다.  

한 장비의 다수의 멀티캐스트 소켓과 심지어 하나의 자바 프로그램 안에 있는 다수의 멀티캐스트 소켓은 모두 동일한 그룹에 가입할 수 있다. 그러한 경우 로컬 호스트에 도착한 해당 그룹의 데이터를 모든 소켓이 수신하게 된다.  

joinGroup() 메소드의 두 번재 생성자에 인터페이스를 전달하면, 명시된 로컬 네트워크 인터페이스만 멀티캐스트 그룹에 가입된다. 예를 들어, 다음 코드는 eth0라는 이름의 네트워크 인터페이스에서 IP 주소 224.2.2.2의 그룹에 가입을 시도한다. 인자로 전달된 인터페이스가 존재하지 않는 경우 사용 가능한 모든 네트워크 인터페이스에서 그룹에 가입한다.  

```java
MulticastSocket ms = new MulticastSocket();
SocketAddress group = new InetSocketAddress("224.2.2.2", 40);
NetworkInterface ni = NetworkInterface.getByName("eth0");
if (ni != null) {
  ms.joinGroup(group. ni);
} else {
  ms.joinGroup(group);
}
```

대기할 네트워크 인터페이스를 명시하는 추가적인 인자를 제공하는 것 말고는 이 메소드의 동작은 단일 인자를 사용하는 joinGroup() 메소드와 같다. 예를 들어, 첫 번째 인자로 멀티캐스트 그룹이 아닌 SocketAddress 객체를 전달하면 IOExeption 예외가 발생한다.  

##### 2.2.2. 그룹 탈퇴와 연결 종료하기
<br/>
특정 멀티캐스트 그룹으로부터 더 이상 데이터그램 수신을 원하지 않을 경우 leaveGroup() 메소드를 호출한다. 이때 특정 인터페이스를 선택할 수 있으며, 생략할 경우 모든 인터페이스에 대해 탈퇴한다.  

+ public void leaveGroup(InetAddress address) throws IOException
+ public void leaveGroup(SocketAddress multicastAddress, NetworkInterface interface) throws IOException  

이 메소드는 로컬 멀티캐스트 라우터에게 더 이상 데이터그램을 보내지 말라는 신호를 보낸다. 탈퇴하려는 주소가 멀티캐스트 주소가 아닌 경우(즉, 224.0.0.0부터 239.255.255.255까지) IOException 예외를 발생시킨다. 그러나 가입되지 않은 멀티캐스트 그룹에 탈퇴를 시도해도 예외는 발생하지 않는다.  

MulticastSocket 클래스가 제공하는 거의 모든 메소드는 IOExeption 예외를 발생시키므로 보통 try 블록 안에서 사용한다. 자바 7에서 DatagramSocket 클래스는 AutoCloseable 인터페이스를 구현하고 있으므로 try-with-resources 구문을 사용할 수 있다.  

```java
try (MulticastSocket socket = new MulticastSocket()) {
  // 서버에 연결
} catch (IOException ex) {
  ex.printStackTrace();
}
```

자바 6 이전 버전에서는 소켓이 사용한 리소스를 해제하기 위해 finally 블록에서 명시적으로 소켓을 종료할 수 있다.  

```java
MulticastSocket socket = null;
try {
  socket = new MulticastSocket();
  // 서버에 연결
} catch (IOException ex) {
  ex.printStackTrace();
} finally {
  if (socket != null) {
    try {
      socket.close();
    } catch (IOException ex) {
      // 무시한다.
    }
  }
}
```

##### 2.2.3. 멀티캐스트 데이터 전송하기
<br/>
MulticastSocket으로 데이터를 전송하는 것은 DatagramSocket으로 데이터를 보내는 것과 비슷하다. 데이터를 DatagramPacket에 채우고 DatagramSocket에서 상속한 send() 메소드를 사용하여 패킷을 전송한다.  

데이터는 패킷이 가리키는 멀티캐스트 그룹에 속한 모든 호스트로 전송된다. 예를 들어:  

```java
try {
  InetAddress ia = InetAddress.getByName("experiment.mcast.net");
  byte[] data = "Here's some multicast data\r\n".getBytes();
  int port = 4000;
  DatagramPacket dp = new DatagramPacket(data, data.length, ia, port);
  MulticastSocket ms = new MulticastSocket();
  ms.send(dp);
} catch (IOException ex) {
  System.err.println(ex);
}
```

기본적으로 멀티캐스트 소켓은 TTL 값으로 1을 사용한다. (즉, 멀티캐스트 패킷은 로컬 서브넷을 벗어날 수 없다.) 그러나 생성자의 첫 번째 인자로 0에서 255 사이의 값을 전달하여 개별 패킷에 대해 TTL 값을 변경할 수 있다.  

setTimeToLive() 메소드는 DatagramSocket 클래스에서 상속된 send(DatagramPacket dp) 메소드를 사용하여 소켓으로부터 전송되는 패킷에 사용될 기본 TTL 값을 설정한다. [Multicast 클래스는 데이터그램마다 TTL 값을 설정하는 send(DatagramPacket dp, byte ttl) 메소드를 제공한다.] getTimeToLive() 메소드는 MulticastSocket의 기본 TTL 값을 반환한다.  

+ public void setTimeToLive(int ttl) throws IOException
+ public int getTimeToLive() throws IOException  

예를 들어, 다음 코드는 TTL 값으로 64를 설정한다.  

```java
try {
  InetAddress ia = InetAddress.getByName("experiment.mcast.net");
  byte[] data = "Here's some multicast data\r\n".getBytes();
  int port = 4000;
  DatagramPacket dp = new DatagramPacket(data, data.length, ia, port);
  MulticastSocket ms = new MulticastSocket();
  ms.setTimeToLive(64);
  ms.send(dp);
} catch (IOException ex) {
  System.err.println(ex);
}
```

##### 2.2.4. 루프백(loopback) 주소
<br/>
호스트가 자신이 보낸 멀티캐스트 패킷을 다시 수신할지 여부는 플랫폼마다 다르다. 즉, setLoopback() 메소드에 true를 전달하면 자신이 보낸 패킷을 받지 않겠다는 것을 의미하며, false를 전달하면 자신이 보낸 패킷을 받겠다는 것을 의미한다.  

+ public void setLoopbackMode(boolean disable) throws SocketException
+ public boolean getLoopbackMode() throws SocketException  

그러나 플랫폼에 따라 설정할 대로 동작하지 않을 수도 있다. 루프백 모드를 모든 시스템이 지원하는 것은 아니기 때문에 패킷을 보내기도 하고 받기도 한다면 루프백 모드를 확인하다는 것이 중요하다. getLoopbackMode() 메소드는 해당 패킷이 루프백 모드가 아닌 경우 true를 반환하고, 루프백 모드인 경우 false를 반환한다. (메소드의 반환값이 반대로 된 것 같지만, 실제로 이렇게 동작한다.)  

시스템이 패킷 루프백을 지원하지만 자신이 보낸 패킷의 수신을 원치 않을 경우 해당 패킷이 수신될 때 인식해서 버려야 한다. 시스템이 패킷 루프백을 지원하지 않지만 지산이 보낸 패킷을 수신하고자 할 경우 자신이 보낸 패킷을 복사하여 저장해 두었다가 패킷을 보낼 때마다 직접 내부 데이터 구조에 넣어 주어야 한다. setLoopback() 메소드를 통해 자신이 원하는 대로 루프백을 제어할 수 있지만, 플랫폼에 따라서 지원하지 않을 수도 있으므로 이 함수를 무턱대고 믿을 수는 없다.  

##### 2.2.5. 네트워크 인터페이스
<br/>
여러 개의 네트워크 인터페이스를 갖고 있는 멀티홈 호스트에서는 setInterface()와 setNetworkInterface() 메소드를 사용하여 멀티캐스트를 보내거나 받는 데 사용할 네트워크 인터페이스를 선택할 수 있다.  

+ public void setInterface(InetAddress address) throws SocketException
+ public InetAddress getInterface() throws SocketException
+ public void setNetworkInterface(NetworkInterface interface) throws SocketException
+ public NetworkInterface getNetworkInterface() throws SocketException  

set 메소드는 인자로 전달된 주소가 로컬 장비의 네트워크 인터페이스가 아닌 경우 SocketException 예외를 발생시킨다. 유니캐스트의 Socket이나 DatagramSocket에서는 네트워크 인터페이스를 생성자에서 설정하는데, 왜 MulticastSocket에서는 따로 메소드를 사용하는지는 명확하지 않다. 안전을 위해 MulticastSocket을 생성한 즉시 인터페이스를 설정해야 하며 그 이후로 변경하지 않도록 해야 한다. 다음은 setInterface()의 사용 예다.  

```java
try {
  InetAddress ia = InetAddress.getByName("www.ibiblio.org");
  MulticastSocket ms = new MulticastSocket(2048);
  ms.setInterface(ia);
  // 데이터를 보내고 받는다
} catch (UnknownHostException ue) {
  System.err.println(ue);
} catch (SocketException se) {
  System.err.println(se);
}
```

setNetworkInterface() 메소드는 setInterface()와 같은 목적으로 사용된다. 즉, 멀티캐스트 데이터를 송수신하는 데 사용할 네트워크 인터페이스를 선택한다. 하지만 네트워크 인터페이스에 바인드될 IP 주소(InetAddress 객체)가 아니라, eth0(NetworkInterface)같이 네트워크 인터페이스의 로컬 이름에 기반을 둔다. setNetworkInterface() 메소드는 NetworkInterface 인자가 로컬 장비에 없는 인터페이스일 경우 SocketException을 발생시킨다.  

getNetworkInterface() 메소드는 해당 MulticastSocket이 데이터를 대기하고 있는 네트워크 인터페이스를 나타내는 NetworkInterface 객체를 반환한다. 이 메소드는 생성자나 setNetworkInterface() 메소드를 통해 명시적으로 네트워크 인터페이스가 설정되지 않은 경우, 주소가 "0.0.0.0"이고 인덱스가 -1인 대체 객체를 반환한다. 예를 들어, 다음 코드는 소켓에 의해 사용되는 네트워크 인터페이스를 출력한다.  

```java
NetworkInterface intf = ms.getNetworkInterface();
System.out.println(intf.getName());
```
