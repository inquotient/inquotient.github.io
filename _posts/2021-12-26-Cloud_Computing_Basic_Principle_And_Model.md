---
title: 기본 개념과 모델
categories:
- Cloud Computing
feature_text: |
  ## 4. 기본 개념과 모델
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 4.2. 클라우드의 특징
<br/>
확장 가능하며 측정된 IT 자원을 효율적으로 원격에서 제공하기 위해서는 IT 환경에 몇 가지 특징이 필요하다. IT 환경이 효율적인 클라우드가 되기 위해 이러한 특징들이 필요하다.  

다음의 6가지 특징은 클라우드 환경에서 주요하게 쓰이는 특징들이다.  

+ 온디맨드식의 사용
+ 어디에서나 가능한 접근
+ 멀티테넌시(와 자원 풀링)
+ 탄력성
+ 사용량 측정
+ 복원력  

#### 4.2.1. 온디맨드식의 사용
<br/>
클라우드 소비자는 자유롭게 IT 자원을 자체 공급하는 클라우드 기반 IT 자원에 일방적으로 접근할 수 있다. 한 번 설정하면 더 이상의 클라우드 소비자나 클라우드 제공자의 개입 없이 자체 공급되는 IT 자원의 사용이 자동화된다. 그 결과 온디맨드식의 사용 환경이 가능하다. '온디맨드식의 셀프 서비스 사용'으로 알려진 이 특징은 주류의 클라우드에서 확립된 서비스 기반의 특징과 사용 유도적 특징을 가능하게 된다.  

#### 4.2.2. 언제 어디서나 가능한 접근
<br/>
어디에서나 가능한 접근이라는 특징은 클라우드 서비스가 넓은 영역에서 접근 가능한 능력을 의미한다. 클라우드 서비스에 접근 가능하도록 하기 위해서는 장비와 전송 프로토콜, 인터페이스, 보안 기술의 지원이 필요하다. 이런 수준의 접근을 가능하게 하기 위해 클라우드 서비스 아키텍처를 여러 클라우드 서비스 소비자의 요구사항에 맞춰 적용해야 한다.  

#### 4.2.3. 멀티테넌시(와 자원 풀링)
<br/>
여러 소비자에게 프로그램 인스턴스를 제공하여 각 소비자가 독립적으로 사용할 수 있게 하는 소프트웨어 프로그램의 특을 멀티테넌시라 한다. 클라우드 제공자는 가상화 기술을 필요로 하는 멀티테넌시 모델을 이용하여 어러 클라우드 서비스 소비자가 사용할 수 있게 IT 자원을 풀링한다. 멀티테넌시 기술을 바탕으로 IT 자원은 클라우드 서비스 소비자의 요구에 따라 동적으로 할당, 재할당될 수 있다. 클라우드 제공자는 여러 클라우드 소비자에게 서비스하는 대규모 IT 자원을 풀링한다. 물리적 IT 자원과 가상의 IT 자원은 클라우드 소비자의 요구에 따라 동적으로 할당, 재할당되는데 일반적으로 통계적 다중화 방식을 바탕으로 한 실행을 따른다. 자원 풀링은 대개 멀티테넌시 기술을 기반으로 달성되며 멀티테넌시 특징에 의해 포함된다.  

#### 4.2.4. 탄력성
<br/>
탄력성은 클라우드가 런타임 상황에서 필요한 만큼 또는 클라우드 소비자나 클라우드 사용자가 미리 정해놓은 만큼 투명하게 IT 자원을 확장하는 자동화된 능력이다. 탄력성은 주요 투자 절감 및 비례 비용 이익과 밀접하게 관련이 있기에 클라우드 컴퓨팅 도입 시 핵심 평가 요소가 된다. 광범위한 IT 자원을 소유하고 있는 클라우드 제공자는 방대한 범위의 탄력성을 제공한다.  

<img src="/assets/images/Cloud_Computing/Fig04-08.jpg" width="100%" height="100%"/>
<img src="/assets/images/Cloud_Computing/Fig04-09.jpg" width="100%" height="100%"/>
<br/>

#### 4.2.5. 사용량 측정
<br/>
사용량 측정의 특징은 주로 클라우드 소비자가 사용한 IT 자원의 사용량을 기록하는 클라우드 플램폼의 능력을 말한다.  

무엇을 측정했는지를 기준으로, 즉 실제 사용한 IT 자원에 대해서나 IT 자원에 접속한 시간에 대해서 클라우드 제공자는 클라우드 소비자에게 청구한다. 이런 맥락에서 사용량 측정은 온디맨드 특성과 밀접한 관련이 있다.  

사용량 측정은 청구를 목적으로 한 통계에만 국한되지 않는다. 사용량 측정은 일반적인 IT 자원 모니터링, 관련된 사용 보고(클라우드 제공자와 클라우드 소비자 모두에게)에 사용된다. 따라서 사용량 측정은 (이후 클라우드 배포 모델 부분에 나올 프라이빗 클라우드 배포 모델에 적용되는) 사용에 대해 청구하지 않는 클라우드에도 관련이 있다.  

#### 4.2.6. 복원력
<br/>
복원이 가능한 컴퓨팅은 물리적인 위치를 넘어 IT 자원을 중복 구현하여 배포한 시스템 대체 작동의 형태다. IT 자원은 사전 설정에 따라 한쪽의 용량이 부족해지면 자동으로 다른 중복 구현된 쪽으로 처리를 넘긴다. 클라우드 컴퓨팅 내에서 복원력은 같은 클라우드(그러나 물리적으로 다른 곳에 있는)나 여러 클라우드를 아우르는 범위 안에 있는 중복 IT 자원을 나타낸다. 클라우드 소비자는 클라우드 기반 IT 자원의 복원력을 활용하여 신뢰성과 가용성을 둘 다 높일 수 있다.  

<img src="/assets/images/Cloud_Computing/Fig04-10.jpg" width="100%" height="100%"/>
<br/>

### 4.3. 클라우드 전달 모델
<br/>
클라우드 전달 모델은 클라우드 제공자가 제공하는 특정한, 미리 준비된 IT 자원의 조합이다. 가장 널리 확립되고 공식화 된 세가지 일반적인 클라우드 전달 모델은 다음과 같다.  

+ IaaS(Infrastructure-as-a-Service)
+ PaaS(Platfor-as-a-Service)
+ SaaS(Software-as-a-Service)  

위의 세 가지 기본적인 클라우드 전달 모델의 변형들이 많이 등장했으며, 각각은 IT 자원의 개별적인 조합으로 구성된다.  

+ 서비스형 스토리지(Storage-as-a-Service)
+ 서비스형 데이터베이스(Database-as-a-Service)
+ 서비스형 보안(Security-as-a-Service)
+ 서비스형 통신(Communication-as-a-Service)
+ 서비스형 통합(Integration-as-a-Service)
+ 서비스형 테스팅(Testing-as-a-Service)
+ 서비스형 프로세스(Process-as-a-Service)  

클라우드 전달 모델이 클라우드 서비스 제공의 여러 형태로 분류되기 때문에 클라우드 전달 모델은 클라우드 서비스 전달 모델이라고도 불린다.  

#### 4.3.1. IaaS
<br/>
IaaS(Infrastructure-as-a-Service) 전달 모델은 클라우드 서비스 기반 인터페이스와 툴을 이용해 접근하고 관리하는 인프라 중심의 IT 자원으로 구성된 필요 시설이 자체적으로 구비된 IT 환경을 말한다. 이러한 환경은 하드웨어, 네트워크, 접속, 운영체제, '그대로의' IT 자원들을 포한한다. 전통적인 호스팅이나 아웃소싱 환경과는 달리 IaaS는 IT 자원을 가상화하고 패키지화하여 번들로 제공해서 런타임 선행 확장과 인프라를 원하는 대로 만드는 것을 간단하게 할 수 있다. IaaS 환경의 일반적인 목표는 클라우드 소비자에게 클라우드를 설정하고 활용하는 높은 수준의 제어 및 책임을 제공하는 것이다. IaaS가 제공하는 IT 자원은 일반적으로 미리 설정되어 있지 않으며 직접적인 관리의 책임이 클라우드 소비자에게 있다. 따라서 이 모델은 클라우드 소비자가 만들고 싶은 클라우드 기반의 환경에 높은 수준의 제어를 필요로 하는 소비자들이 사용한다.  

클라우드 제공자들은 때로 클라우드 환경을 확장하기 위해 나머지 클라우드 제공자로부터 IaaS를 제공받는 계약을 맺을 것이다. 다른 클라우드 제공자들이 제공하는 IaaS 상품이 제공하는 IT 자원의 유형과 브랜드는 매우 다양할 수 있다. IaaS 환경을 바탕으로 이용 가능한 IT 자원은 대개 초기화된 가상 인스턴스 형태로 제공된다. 전형적인 IaaS 환경의 중심에 있는 주요한 IT 자원은 바로 가상 서버다. 가상 서버는 프로세서 용량, 메모리, 스토리지 공간과 같은 하드웨어 요구사항에 맞춰 임대된다.  

<img src="/assets/images/Cloud_Computing/Fig04-11.jpg" width="100%" height="100%"/>
<br/>

#### 4.3.2. PaaS
<br/>
PaaS(Platfor-as-a-Service) 전달 모델은 이미 배포되고 설정된 IT 자원으로 구성된, 이미 정의된 '사용할 준비가 되어 있는' 환경을 말한다. 특히 PaaS는 주문 제작 애플리케이션의 전달 생명주기를 지원하기 위해 이미 패키지화되어 있는 제품과 도구로 구성된 이미 만들어진 환경을 사용한다(그리고 환경에 의해 주로 정의된다).  

클라우드 소비자가 PaaS 환경을 사용하거나 투자하는 공통적인 이유는 다음과 같다.  

+ 클라우드 소비자는 확장성과 경제적 이유 때문에 온 프레미스 환경을 클라우드로 확장하고 싶어한다.
+ 클라우드 소비자는 온 프레미스 환경을 완전히 대체할 사용할 준비가 되어 있는 환경을 사용한다.
+ 클라우드 소비자는 클라우드 제공자가 되어 또 다른 외부의 클라우드 소비자가 사용할 수 있는 클라우드 서비스를 배포하기를 원한다.  

기존에 만들어진 플랫폼에서 작업해서 클라우드 소비자는 IaaS 모델을 통해 제공되는 IT 자원의 인프라를 설정하고 유지하는 관리적 책임을 면제받는다. 반면, 클라우드 소비자에게는 플랫폼을 제공하는 기반 IT 자원에 대한 낮은 수준의 제어가 부여된다.  

PaaS 제품은 다른 개발 스택과 함께 이용 가능하다.  

<img src="/assets/images/Cloud_Computing/Fig04-12.jpg" width="100%" height="100%"/>
<br/>

#### 4.3.3. SaaS
<br/>
공유되는 클라우드 서비스 형태이며 '제품'이나 일반적인 유틸리티로 이용 가능한 소프트웨어 프로그램이 SaaS(Software-as-a-Service) 제공의 전형적인 모습이다. SaaS 전달 모델은 재사용 가능한 클라우드 서비스를 다양한 클라우드 소비자가 널리(때로는 상업적으로) 이용하게 한다. 전체 시장은 여러 목적과 조건으로 임대되고 사용되는 SaaS 제품 위주로 형성된다.  

클라우드 소비자는 SaaS 구현에 매우 제한적인 관리 제어 권한을 갖는다. 대부분 클라우드 제공자에게 주어지며 법적으로 클라우드 서비스 소유자 역할을 하는 주체가 소유한다. 예를 들어 PaaS 환경에서 작업하면서 클라우드 소비자의 역할을 하는 조직은 SaaS를 제공해서 같은 환경에서 배포하기로 한 클라우드 서비스를 구축할 수 있다. 한 조직이 제공하는 SaaS 기반의 클라우드 서비스를 클라우드 소비자인 다른 조직이 이용 가능하므로 효과적으로 클라우드 제공자의 역할을 하게 된다.  

<img src="/assets/images/Cloud_Computing/Fig04-13.jpg" width="100%" height="100%"/>
<br/>

#### 4.3.4. 클라우드 전달 모델 비교
<br/>
<table>
  <thead>
    <tr>
      <td>클라우드 전달 모델</td>
      <td>클라우드 소비자의 제어 수준</td>
      <td>클라우드 소비자가 사용할 수 있는 기능</td>
      <td>클라우드 소비자 활동</td>
      <td>클라우드 제공자 활동</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>SaaS</td>
      <td>사용 및 사용 관련 설정</td>
      <td>프론트 엔드의 사용자 인터페이스에 접근</td>
      <td>클라우드 서비스의 사용 및 설정</td>
      <td>
        클라우드 서비스의 구현, 관리, 유지보수<br/>
        클라우드 소비자의 사용 모니터링
      </td>
    </tr>
    <tr>
      <td>PaaS</td>
      <td>제한적인 관리</td>
      <td>플랫폼 사용과 연관된 IT 자원에 대한 중간 수준의 관리자의 권한 제어</td>
      <td>클라우드 서비스와 클라우드 기반 솔루션 개발, 테스트, 배포, 관리</td>
      <td>
        플랫폼 사전 설정 및 기반 인프라, 미들웨어, IT 자원 제공<br/>
        클라우드 소비자의 사용 모니터링
      </td>
    </tr>
    <tr>
      <td>IaaS</td>
      <td>모든 관리</td>
      <td>가상 인프라와 관련된 IT 자원에 대한 완전한 접근, 가능한 기반 물리적 IT 자원까지 접근</td>
      <td>기반 인프라의 초기 설정 및 필요한 소프트웨어의 설치, 관리, 모니터링</td>
      <td>
        물리적 프로세싱, 스토리지, 네트워킹, 호스팅의 제공 및 관리<br/>
        클라우드 소비자의 사용 모니터링
      </td>
    </tr>
  </tbody>
</table>
<br/><br/>

#### 4.3.5. 클라우드 전달 모델 조합
<br/>
모델들의 조합 적용을 좀 더 자세히 살펴볼 기회를 감안하여 세 가지 기본적인 클라우드 전달 모델은 자연적인 프로비저닝 계층으로 구성된다. 다음 부분에서는 두 가지 일반적인 조합과 관련된 고려사항을 중점적으로 다룬다.  

##### 4.3.5.1. IaaS &#43; PaaS
<br/>
PaaS 환경은 물리와 가상 서버, IaaS 환경에서 제공되는 다른 IT 자원들과 맞먹는 기반 인프라 위에 구성된다.  

클라우드 제공자는 대개 클라우드 소비자가 PaaS 환경을 이용할 수 있도록 클라우드에서 IaaS 환경을 제공할 필용가 없다. PaaS 환경을 제공하는 클라우드 제공자가 또 다른 클라우드 제공자의 IaaS 환경을 임차했다고 하자.  

이러한 배치의 동기는 경제적인 요인의 영향 때문이거나 첫 번째 클라우드 제공자가 다른 클라우드 소비자를 지원하는 데 용량이 초과할 것 같기 때문이다. 또는 아마도 특정 클라우드 소비자가 데이터가 물리적으로 특정 지역(첫 번째 클라우드 제공자의 클라우드가 있는 곳이 아닌)에 보관되어야 한다는 법적 요건을 제기했을 수 있다.  

<img src="/assets/images/Cloud_Computing/Fig04-14.jpg" width="100%" height="100%"/>
<img src="/assets/images/Cloud_Computing/Fig04-15.jpg" width="100%" height="100%"/>
<br/>

##### 4.3.5.2. IaaS &#43; PaaS &#43; SaaS
<br/>
세 가지 모든 클라우드 전달 모델이 결합되어 IT 자원 계층을 수립할 수도 있다. 예를 들어 클라우드 소비자 조직은 상용 제품으로 사용될 수 있는 클라우드 소비자의 SaaS 클라우드 서비스를 개발하고 배포하기 위해 선행하는 계층의 아키턱처 위에 더하여 PaaS 환경이 제공하는 이미 만들어진 환경을 사용할 수 있다.

<img src="/assets/images/Cloud_Computing/Fig04-16.jpg" width="100%" height="100%"/>
<br/>

### 4.4. 클라우드 배포 모델
<br/>
클라우드 배포 모델은 클라우드 환경의 특정한 형태를 말하는 것으, 주로 소유권과 규모, 접근방법에 의해 분류된다.  

네 가지 일반적인 클라우드 배포 모델은 다음과 같다.  

+ 퍼블릭 클라우드
+ 커뮤니티 클라우드
+ 프라이빗 클라우드
+ 하이브리드 클라우드

다음 부분에서 각각에 대해 설명한다.  

#### 4.4.1. 퍼블릭 클라우드
<br/>
퍼블릭 클라우드는 제3자인 클라우드 제공자가 소유하고 공개적으로 접근 가능한 클라우드 환경이다. 퍼블릭 클라우드 내에 있는 IT 자원은 대개 앞서 기술된 클라우드 전달 모델을 통해 제공되며 클라우드 소비자에게 유상으로 제공되거나 다른 수익원(광고와 같은)을 통해 상용화된다.  

클라우드 제공자는 퍼블릭 클라우드와 IT 자원을 창조하고 지속적으로 관리해야 할 책임이 있다.  

<img src="/assets/images/Cloud_Computing/Fig04-17.jpg" width="100%" height="100%"/>
<br/>

#### 4.4.2. 커뮤니티 클라우드
<br/>
커뮤니티 클라우드는 특정 커뮤니티의 클라우드 소비자에게만 접근이 허용된다는 점을 제외하고는 퍼블릭 클라우드와 비슷하다. 커뮤니티 클라우드는 커뮤니티 멤버나 제한적인 접근을 갖는 퍼블릭 클라우드를 제공하는 제3자 클라우드 제공자에게 동시에 소유될 수 있다. 커뮤니티의 구성원인 클라우드 소비자는 커뮤니티 클라우드를 정의하고 발전시키는 책임을 공유한다.  

커뮤니티의 회원이 모든 클라우드의 IT 자원에 대한 접근이나 제어를 보장해야 할 필요는 없다. 커뮤니티 바깥의 주체는 커뮤니티가 허가하지 않는 한 접근 권한을 가질 수 없다.  

<img src="/assets/images/Cloud_Computing/Fig04-18.jpg" width="100%" height="100%"/>
<br/>

#### 4.4.3. 프라이빗 클라우드
<br/>
프라이빗 클라우드는 하나의 조직이 소유한다. 프라이빗 클라우드는 조직이 조직의 다른 파트나 지역, 부서에 의해 IT 자원에 대한 접근을 집중화하기 위해 클라우드 컴퓨팅 기술을 사용하는 것을 가능하게 한다. 프라이빗 클라우드가 제어된 환경에 존재할 때 위험과 고려사항에서 언급한 문제점들은 적용되지 않는다.  

프라이빗 클라우드의 사용은 조직적인 신뢰 경계가 정의되고 적용되는 방법을 변화시킨다. 프라이빗 클라우드 환경의 실제 관리는 내부 또는 외주 인력에 의해 수행된다.  

프라이빗 클라우드를 사용하는 한 조직은 기술적으로 클라우드 소비자가 됨과 동시에 클라우드 제공자가 된다. 이러한 역할은 다음과 같이 구분된다.  

+ 분리된 조직적 부서는 대개 클라우드를 제공하는 역할을 한다(즉 클라우드 제공자의 역할을 한다).
+ 프라이빗 클라우드에 접근을 요청하는 부서는 클라우드 소비자의 역할을 한다.  

프라이빗 클라우드에서는 '온 프레미스'와 '클라우드 기반' 용어를 정확하게 사용하는 것이 매우 중요하다. 프라이빗 클라우드가 물리적으로 조직 내부 온 프레미스되어 있더라도 IT 자원이 클라우드 소비자에게 원격에서 제공되는 한 '클라우드 기반'으로 여겨진다. 클라우드 소비자의 역할을 하는 부서가 프라이빗 클라우드 외부에서 호스팅하는 IT 자원은 프라이빗 클라우드 기반 IT 자원과 비교하면 '온 프레미스'로 간주된다.

<img src="/assets/images/Cloud_Computing/Fig04-19.jpg" width="100%" height="100%"/>
<br/>

#### 4.4.4. 하이브리드 클라우드
<br/>
하이브리드 클라우드는 두 가지 이상의 클라우드 배포 모델로 구성된 클라우드 환경이다. 예를 들어 클라우드 소비자는 민감한 데이터를 처리하는 클라우드 서비스는 프라이빗 클라우드에, 덜 민감한 클라우드 서비스는 퍼블릭 클라우드에 배포할 것이다. 이러한 조합의 결과가 바로 하이브리드 배포 모델이다.  

하이브리드 배포 아키텍처는 클라우드 환경간의 잠재적 차이와 프라이빗 클라우드 제공자 조직과 퍼블릭 클라우드 제공자 사이의 관리 책임 분리 등의 이유로 창조하고 관리하기에 복잡하고 고려사항이 많을 수 있다.  

<img src="/assets/images/Cloud_Computing/Fig04-20.jpg" width="100%" height="100%"/>
<br/>

#### 4.4.5. 기타 클라우드 배포 모델
<br/>
기본적인 클라우드 배포 모델의 다양한 변형이 있다. 예는 다음과 같다.  

+ 가상 프라이빗 클라우드  
'전용 클라우드' 또는 '호스티드 클라우드'로 알려진 모델로 퍼블릭 클라우드 제공자에 의해 호스팅되고 관리되는 필요 시설이 다 구비된 클라우드 환경이다.  

+ 상호 클라우드  
두 개 이상의 상호 연결된 클라우드로 구성된 아키텍처를 기반으로 하는 모델이다.