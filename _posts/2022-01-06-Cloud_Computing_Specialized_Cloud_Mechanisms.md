---
title: 전문화된 클라우드 메커니즘
categories:
- Cloud Computing
feature_text: |
  ## 8. 전문화된 클라우드 메커니즘
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 8.1. 자동 확장 리스너
<br/>
자동 확장 리스너 메커니즘은 동적 확장 용도를 위해 클라우드 서비스 소비자와 클라우드 서비스간의 통신을 모니터하고 추적하는 서비스 에이전트다. 자동 확장 리스너는 자동으로 작업 부하 상태 정보를 추적할 수 있는 곳으로부터 주로 방화벽 근처인 클라우드 내에 배치된다. 작업 부하는 클라우드 소비자가 발생시킨 요청의 양이나 요청의 특정 유형에 의해 작동되는 백엔드 처리 요구를 통해 결정될 수 있다.  

자동 확장 리스너는 다음의 작업 부하 변동 조건에 따른 응답의 다른 유형을 제공할 수 있다.  

+ 클라우드 소비자에 의해 미리 정의된 변수에 근거해 IT 자원을 자동 확장하거나 축소하는 것(보통 자동 확장으로 불린다)
+ 작업 부하가 현재 한계점을 초과하거나 할당된 자원 아래로 떨어질 때 클라우드 소비자의 자동 알림으로, 클라우드 소비자가 현재 IT 자원 할당을 조절하기 위해 선택하는 방법  

클라우드 제공자 업체에 따라 자동 확장 리스너로 동작하는 서비스 에이전트의 이름이 다르다.  

<img src="/assets/images/Cloud_Computing/Fig08-01.jpg" width="100%" height="100%"/>
<br/>

### 8.2. 부하 분산
<br/>
수평적 확장의 일반적인 접근은 단일 IT 자원이 제공할 수 있는 것을 넘는 성능과 용량을 증가하기 위해 두 개나 그 이상의 IT 자원에 걸리는 작업 부하의 밸런스를 조절하는 것이다. 부하 분산 메커니즘은 이 전제를 기반으로 한 런타임 에이전트다.  

작업 알고리즘의 단순 분배를 넘어, 부하 분산은 다음을 포함하는 특화된 런타임 작업 부하 분산 기능의 영역을 수행할 수 있다.  

+ 비대칭 분배  
더 많은 작업 부하가 더 높은 처리량과 함께 IT 자원에 발행된다.  
+ 작업 부하 우선순위  
작업 부하는 우선순위 단계에 따라 계획하고, 대기하며, 폐기되고, 분배된다.  
+ 컨텐츠 인식 분산  
요청은 요청 컨텐츠에 명시된 대로 다른 IT 자원에 분배된다.  

<img src="/assets/images/Cloud_Computing/Fig08-05.jpg" width="100%" height="100%"/>
<br/>

로드 밸런서는 성능과 QoS 규칙의 집합과, IT 자원 사용을 최적화하고, 과부하를 피하며, 처리량을 최대화하는 목적과 함께한 변수로 프로그램되거나 설정된다.  

로드 밸런서 메커니즘은 다음으로 존재할 수 있다.  

+ 다중 단계 네트워크 스위치
+ 전용 하드웨어 기기
+ 전용 소프트웨어 기반 시스템(일반적인 서버 운영체제)
+ 서비스 에이전트(보통 클라우드 관리 소프트웨어에 의해 통제됨)  

로드 밸런서는 전형적으로 작업 부하를 발생시키는 IT 자원과 처리 중인 작업 부하를 수행하는 IT 자원 사이의 통신 경로에 위치한다. 이 메커니즘은 클라우드 서비스 소비자로부터 숨겨지는 투명한 에이전트나 작업 부하를 수행하는 IT 자원을 추상화하는 프록시 요소로서 고안될 수 있다.  

### 8.3. SLA 모니터
<br/>
SLA 모니터 메커니즘은 SLA에 발행된 계약상의 QoS 요구를 만족하는 것을 확신하기 위해 클라우드 서비스의 런타임 성능을 명확하게 관찰하곤 한다. SLA 모니터로 수집된 데이터는 SLA 보고 측량을 집계하기 위해 SLA 관리 시스템에 의해 처리된다. 시스템은 SLA 모니터가 클라우드 서비스가 '다운'으로 보고할 때와 같은 예외 조건이 발생할 때 사전 대책을 마련해 수리하거나 클라우드 서비스를 시스템 대체 작동할 수 있다.  

<img src="/assets/images/Cloud_Computing/Fig08-07.jpg" width="100%" height="100%"/>
<br/>

### 8.4. 사용량당 과금 모니터
<br/>
사용량당 과금 모니터 메커니즘은 미리 정의된 가격 변수와 일치하는 클라우드 기반 IT 자원 사용을 측정하고, 요금 계산과 과금 목적을 위해 사용 로그를 발생시킨다.  

전형적인 모니터링 변수는 다음과 같다.  

+ 요청/응답 메시지 수량
+ 전송된 데이터 볼륨
+ 대역폭 소비  

사용량당 과금 모니터로 수집된 데이터는 지불 요금을 계산하는 과금 관리 시스템에 의해 처리된다.  

<img src="/assets/images/Cloud_Computing/Fig08-12.jpg" width="100%" height="100%"/>
<br/>

클라우드 소비자는 클라우드 서비스의 새 인스턴스의 생성을 요청한다(1). IT 자원은 초기화되고 사용량당 과금 모니터는 자원 소프트웨어로부터 '시작' 이벤트 알림을 받는다(2). 사용량당 과금 모니터는 로그 데이터베이스에 값 타임스탬프를 저장한다(3). 클라우드 소비자는 나중에 클라우드 서비스 인스턴스가 정지되는 것을 요청한다(4). 사용량당 과금 모니터는 자원 소프트웨어로부터 '정지' 이벤트 알림을 받고(5), 로그 데이터베이스에 값 타임스탬프를 저장한다(6).  

<img src="/assets/images/Cloud_Computing/Fig08-13.jpg" width="100%" height="100%"/>
<br/>

클라우드 서비스 소비자는 클라우드 서비스에 요청 메시지를 보낸다(1). 사용량당 과금 모니터는 메시지를 가로채고(2), 이것을 클라우드 서비스에 전송하며(3a), 해당 모니터링 측정과 일치하는 사용 정보를 저장한다(3b). 클라우드 서비스는 요청된 서비스를 제공하기 위해 클라우드 서비스 소비자에게 응답 메시지를 되돌려준다(4).  

### 8.5. 감사 모니터
<br/>
감사 모니터 메커니즘은 규제력을 지니고 계약상의 의무 지원으로(혹은 명시되어) 네트워크와 IT 자원을 위해 감사 추적 데이터를 모으곤 한다.  

<img src="/assets/images/Cloud_Computing/Fig08-15.jpg" width="100%" height="100%"/>
<br/>

클라우드 서비스 소비자는 보안 자격과 함께 로그인 요청 메시지를 보내서 클라우드 서비스에 접근을 요청한다(1). 감사 모니터는 메시지를 가로채고(2) 이것을 인증 서비스에 전달한다(3). 인증 서비스는 보안 자격을 처리한다. 응답 메시지는 클라우드 서비스 소비자를 위해 발생하고, 로그인 시도의 원인이 된다. 감사 모니터는 응답 메시지를 가로채고 전체 모인 로그인 이벤트 세부 사항을 기관의 감사 정책 요구마다 로그 데이터베이스에 저장한다(5). 접근이 승인되고 응답은 클라우드 서비스 소비자에게 되돌려진다(6).  

### 8.6. 대체 작동 시스템
<br/>
대체 작동 시스템 메커니즘은 중복 구현을 제공하기 위해 수립된 클러스터링 기술을 사용해서 IT 자원의 안정성과 가용성을 증가시키곤 한다. 대체 작동 시스템은 현재 사용 중인 IT 자원이 사용할 수 있지 않게 될 때마다 여유이거나 대기 중인 IT 자원 인스턴스로 자동 변환하게 설정된다.  

대체 작동 시스템은 다중 애플리케이션에 실패의 단일 포인트를 소개할 수 있는 임무 중요 프로그램과 재사용 가능한 서비스에 일반적으로 사용된다. 대체 작동 시스템은 각 지역이 같은 IT 자원의 한 개나 그 이상의 여유 구현을 제공하기 위해 하나의 지리학적 지역보다 많이 퍼질 수 있다.  

자원 복제 메커니즘은 때때로 오류나 비 가용성 조건을 감지하기 위해 활발히 모니터하는 여유 IT 자원 인스턴스를 제공하기 위해 대체 작동 시스템을 사용한다.  

대체 작동 시스템은 두가지 기본 설정이 있다.  

#### 8.6.1. 능동-능동
<br/>
능동-능동(Active-Active) 설정에서 IT 자원의 여유 구현은 활발히 작업 부하를 동시다발적으로 제공한다. 능동 인스턴스 사이의 부하 분산은 필수다. 실패가 감지됐을 때, 실패한 인스턴스는 부하 분산 스케줄러에서 제거된다. 실패가 감지될 때 어느쪽이든 IT 자원이 운용상 남는 것이 처리를 넘겨 받는다.  

<img src="/assets/images/Cloud_Computing/Fig08-17.jpg" width="100%" height="100%"/>
<br/>

<img src="/assets/images/Cloud_Computing/Fig08-18.jpg" width="100%" height="100%"/>
<br/>

<img src="/assets/images/Cloud_Computing/Fig08-19.jpg" width="100%" height="100%"/>
<br/>

#### 8.6.2. 능동-수동
<br/>
능동-수동(Active-Passive) 구성에서 대기나 활동하지 않는 구성은 사용할 수 없게 된 IT 자원으로부터 처리를 넘겨받기 위해 활성화되고, 상응하는 작업 부하는 운용을 넘겨받을 인스턴스에 전가된다.  

일부 대체 작동 시스템은 실패 조건을 감지하고 작업 부하 분배에 실패한 IT 자원 인스턴스를 배제하는 특화된 로드 밸런서에 의존하는 IT 자원을 활성화하기 위해 작업 부하를 전가하게 고안된다. 대체 작동 시스템의 이러한 유형은 실행 사태 관리를 요구하지 않고 상태가 없는 처리 능력을 제공하는 IT 자원에 적합하다. 전형적으로 클러스터링과 가상화 기술에 기반을 둔 기술 아키텍처에서 복제와 대기 IT 자원 구현은 그들의 상태와 실행 문맥을 공유하는 것이 요구된다. 실패한 IT 자원에서 수행됐던 복잡한 작업은 복제 구조 중 하나에서 운용을 유지할 수 있다.  

<img src="/assets/images/Cloud_Computing/Fig08-20.jpg" width="100%" height="100%"/>
<br/>

<img src="/assets/images/Cloud_Computing/Fig08-21.jpg" width="100%" height="100%"/>
<br/>

<img src="/assets/images/Cloud_Computing/Fig08-22.jpg" width="100%" height="100%"/>
<br/>

### 8.7. 하이퍼바이저
<br/>
하이퍼바이저 메커니즘은 물리 서버의 가상 서버 인스턴스를 주로 발생시키곤 하는 가상화 인프라의 근본 부분이다. 하이퍼바이저는 일반적으로 하나의 물리 서버에 제한되고, 그래서 해당 서버의 가상 이미지를 유일하게 생성할 수 있다. 유사하게 하이퍼바이저만이 가상 서버가 동일 선상의 물리 서버에 있는 자원 풀을 발생시키도록 할 수 있다. 하이퍼바이저는 가상 서버의 용량을 증가시키거나 중단시키는 것과 같은 가상 서버 관리 특성을 제한한다. VIM은 물리 서버 위의 여러 하이퍼바이저를 관리하기 위해 특성 범위를 제공한다.  

<img src="/assets/images/Cloud_Computing/Fig08-27.jpg" width="100%" height="100%"/>
<br/>

하이퍼바이저 소프트웨어는 베어메탈 서버에 온 프레미스될 수 있고, 프로세서 전원과 메모리, I/O와 같은 하드웨어 자원의 사용을 통제하고 공유, 계획하기 위한 특성을 제공한다. 이것은 전용 자원으로서 각 가상 서버의 운영체제에 나타날 수 있다.  

### 8.8. 자원 클러스터
<br/>
지역적으로 퍼진 클라우드 기반 IT 자원은 할당과 사용을 향상하기 위해 논리적인 그룹으로 묶을 수 있다. 자원 클러스터 메커니즘은 그들이 단일 IT 자원으로서 수행될 수 있도록 다중 IT 자원 인스턴스를 묶곤 한다. 이것은 혼합된 컴퓨팅 능력과 부하 분산, 클러스터된 IT 자원의 가용성을 증가시킨다.  

자원 클러스터 아키텍처는 작업량 분배와 업무 스케줄링, 데이터 공유, 시스템 동기화에 대해 통신하기 위한 IT 자원 인스턴스 사이 고속의 전용 네트워크 연결이나 클러스터 노드에 의존한다. 모든 클러스터 노드에서 분배된 미들웨어로서 수행 중인 클러스터 관리 플랫폼은 보통 이 활동에 대해 책임이 있다. 이 플랫폼은 분배된 IT 자원이 하나의 IT 자원으로서 나타나게 하고 또한 클러스터 내의 IT 자원을 수행하는 조정 기능을 구현한다.  

일반 자원 클러스터 유형은 다음을 포함한다.  

+ 서버 클러스터  
물리나 가상 서버는 성능과 가용성을 증가시키기 위해 클러스터된다. 다른 물리 서버에서 수행 중인 하이퍼바이저는 클러스터된 가상 서버를 수립하기 위해 가상 서버 실행 상태(메모리 페이지나 프로세서 레지스터 상태와 같은)를 공유하기 위해 설정될 수 있다. 이런 속성 내에 공유 스토리지에 접근할 수 있도록 물리 서버를 일반적으로 요구하는 가상 서버는 하나에서 그 밖의 곳으로 실시간 이관할 수 있다. 이 처리에서 가상화 플랫폼은 하나의 물리 서버에 주어진 가상 서버의 실행을 중단하고 다른 물리 서버에서 재개한다. 처리는 가상 서버 운영체제에 투명하고, 과부하의 물리 서버에서 수행 중인 가상 서버를 적합한 능력을 갖춘 다른 물리 서버에 라이브 이동해서 확장성을 증가시킬 수 있다.  

+ 데이터베이스 클러스터  
데이터 가용성을 향상하기 위해 고안된 고가용성 자원 클러스터는 클러스터 내에 사용된 다른 스토리지 장치에 저장되는 데이터의 일관성을 유지하는 동기화 특성을 가진다. 중복 능력은 보통 동기화 조건을 유지하기 위해 투입된 능동-능동이나 능동-수동 대체 작동 시스템에 기반을 둔다.  

+ 큰 데이터 모음 클러스터  
데이터 분할과 분배는 목적 데이터 모음이 절충된 데이터 무결성이나 컴퓨팅 정확성 없이 효율적으로 분할될 수 있도록 구현된다. 각 클러스터 노드는 다른 클러스터 유형만큼 많은 노드와 통신하지 않고 작업량을 처리한다.  

많은 자원 클러스터는 자원 클러스터 아키텍처 내에 일관성을 유지하고 설계를 단순화하기 위해서 거의 동일한 컴퓨팅 능력과 특성을 가질 수 있도록 클러스터 노드를 요구한다. 고가용성 클러스터 아키텍처의 클러스터 노드는 일반 스토리지 IT 자원에 접근하고 공유할 필요가 있다. 이것은 스토리지 장치에 접근하기 위함과 IT 자원 통합을 수행하기 위한 노드 사이 통신의 두 단계를 요구할 수 있다. 일부 자원 클러스터는 네트워크 단계를 유일하게 요구할 수 있는 더 많은 느슨하게 묶인 자원과 고안된다.  

<img src="/assets/images/Cloud_Computing/Fig08-31.jpg" width="100%" height="100%"/>
<br/>

부하 분산과 자원 복제는 클러스터 가능해진 하이퍼바이저를 통해 구현된다. 전용 스토리지 지역 네트워크는 일반 클라우드 스토리지 장치를 공유할 수 있는 클러스터된 스토리지와 클러스터 된 서버를 연결하곤 한다. 이것은 스토리지 클러스터에 독립적으로 옮겨지는 스토리지 복제 처리를 단순화 한다.  

<img src="/assets/images/Cloud_Computing/Fig08-32.jpg" width="100%" height="100%"/>
<br/>

느슨하게 묶인 서버 클러스터는 로드 밸런서를 포함한다. 공유 스토리지는 없다. 자원 복제는 클러스터 소프트웨어에 의해 네트워크를 통해 클라우드 스토리지 장치를 복제하곤 한다.  

자원 클러스터의 두 가지 기본 유형이 있다.  

+ 로드밸런스된 클러스터  
이 자원 클러스터는 IT 자원 관리의 중앙화를 보존하는 동안 IT 자원량을 증가시키기 위해 클러스터 노드 사이의 작업 부하를 분산한다. 이것은 보통 클러스터 관리 플랫폼 내에 내장되거나 분리된 IT 자원으로서 설정되는 로드 밸런서 메커니즘을 구현한다.  

+ HA 클러스터  
고가용성 클러스터는 다중 노드 실패의 이벤트에 시스템 가용성을 유지하고 클러스터된 IT 자원 대부분이나 모두의 중복 구현을 한다. 이것은 실패 조건을 모니터하고 실패된 노드로부터 작업 부하를 자동으로 돌리는 대체 작동 시스템 메커니즘을 구현한다.  

클러스터된 IT 자원의 프로비저닝은 동등한 컴퓨팅 능력을 갖추는 단독의 IT 자원의 프로비저닝보다 상당히 비쌀 수 있다.  

### 8.9. 다중 장치 중개자
<br/>
각 클라우드 서비스는 제공된 하드웨어 장치나 동신 요청으로 차별화된 클라우드 서비스 소비자의 점주에 의해 접근되는 것이 필요할 것이다. 클라우드 서비스와 분리된 클라우드 서비스 소비자간의 비호환성을 극복하기 위해, 타당성을 확인하는 것은 런타임에 교환되는 정보를 변형시키기 위해(혹은 고치기 위해) 필요하다.  

다중 장치 중개자 메커니즘은 클라우드 서비스가 클라우드 서비스 소비자 프로그램과 장치의 넓은 범주에 접근 가능하게 하기 위해서 런타임 데이터 변형을 가능하게 하곤 한다.  

<img src="/assets/images/Cloud_Computing/Fig08-35.jpg" width="100%" height="100%"/>
<br/>

다중 장치 중개자는 클라우드 서비스와 다른 유형의 클라우드 서비스 소비자 장치간의 데이터 교환을 변형시키기 위해 필요한 타당성의 확인을 포함한다. 이 시나리오는 다중 장치 중개자가 자신의 API와 함께 클라우드 서비스로서 묘사된다. 이 메커니즘은 또한 필요한 변형을 수행하기 위해 런타임에 메시지를 가로채는 서비스 에이전트로서 구현될 수 있다.  

다중 장치 중개자는 일반적으로 다음과 같은 게이트웨이나 무형의 게이트웨이 구성요소로서 존재한다.  

+ XML 게이트웨이  
XML 데이터를 전송하고 인증한다.
+ 클라우드 스토리지 게이트웨이  
클라우드 스토리지 프로토콜을 변형시키고 데이터 전송과 스토리지를 가능하게 하기 위해 스토리지 장치를 암호화한다.
+ 모바일 장치 게이트웨이  
클라우드 서비스와 호환되는 프로토콜 내 모바일 장치에 의해 사용되는 통신 프로토콜을 변형시킨다.  

변형 로직이 생성될 수 있는 단계는 다음을 포함한다.  

+ 전송 프로토콜
+ 메시징 포로토콜
+ 스토리지 장치 프로토콜
+ 데이터 스키마/데이터 모델  

예를 들어 다중 장치 중개자는 모바일 장치로 클라우드 서비스에 접근하는 클라우드 서비스 소비자를 위해 전송과 메시징 프로토콜을 고치는 타당성을 포함할 수도 있다.  

### 8.10. 상태 관리 데이터베이스
<br/>
상태 관리 데이터베이스는 소프트웨어 프로그램을 위해 일시적으로 상태 데이터를 지속하기 위해 사용되는 스토리지 장치다. 메모리에 상태 데이터를 캐싱하는 것의 대체로서, 소프트웨어 프로그램은 소비하는 런타임 메모리의 양을 줄이기 위해 데이터베이스에 상태 데이터를 없앨 수 있다. 이렇게 해서 소프트웨어 프로그램과 둘러싸인 인프라는 더 확장 가능하다. 일반적으로 상태 관리 데이터베이스는 길게 수행되는 런타임 활동에 특별히 포함되는 클라우드 서비스로 사용된다.  

<img src="/assets/images/Cloud_Computing/Fig08-37.jpg" width="100%" height="100%"/>
<br/>

<img src="/assets/images/Cloud_Computing/Fig08-38.jpg" width="100%" height="100%"/>
<br/>