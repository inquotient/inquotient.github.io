---
title: Nginx Http Server
categories:
- Nginx
feature_text: |
  ## Nginx Http Server
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
	.align-center { text-align: center; }
</style>


### 2. 기본 엔진엑스 구성
<br/>

#### 2.1. 구성 파일 구문
<br/>

##### 2.1.2. 구조와 포함
<br/>

include 지시어는 이름 그대로 지정된 파일을 인클루드시킨다. 지시어가 있는 곳에 포함될 파일의 내용이 삽입된다.  

<table>
	<thead>
		<tr>
			<th>표준 파일명</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>nginx.conf</td>
			<td>웹 서버의 기본 구성</td>
		</tr>
		<tr>
			<td>mime.types</td>
			<td>파일 확장자와 연관된 MIME 타입의 목록</td>
		</tr>
		<tr>
			<td>fastcgi&#95;params</td>
			<td>Fast CGI 관련 구성</td>
		</tr>
		<tr>
			<td>proxy.conf</td>
			<td>프록시 관련 구성</td>
		</tr>
		<tr>
			<td>sites.conf</td>
			<td>엔진엑스로 제공되는 웹 사이트(가상 호스트라고도 부름) 구성, 도메인 단위로 분리하기를 권장한다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 2.1.3. 지시어 블록
<br/>

서버 전반에 영향을 미치는 지시어는 반드시 어떤 블록에도 속하지 않은 구성의 최상위에 위치해야 한다. 이러한 구성 파일의 최상위 수준을 main 블록이라고 한다.  

대부분의 경우에 블록은 규약에 따라 서로 중첩될 수 있다.  

server 블록은 서버 기기에서 돌아가는 웹 사이트, 즉 가상 호스트를 구성한다.  

server 블록에는 지정된 경로와 일치할 때만 설정이 적용되게 하는 location 블록이 들어갈 수도 있다.  

#### 2.2. 기반 모듈의 지시어
<br/>

##### 2.2.3. 핵심 모듈 지시어
<br/>

<table>
	<thead>
		<tr>
			<th>지시어 이름과 맥락</th>
			<th>구문과 설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>daemon</td>
			<td>
				가능한 값: on이나 off<br/>
				구문:<br/>
				daemon on;<br/>
				기본값: on<br/>
				데몬 모드를 켜거나 끈다. 끄면 프로그램이 백그라운드에서 시작하지 않는다. 셸에서 실행하면 전면(foreground)에서 실행된다. 언제 어디서 엔진엑스가 충돌이 나는지 알고 싶을 때 디버깅에 유용하다.
			</td>
		</tr>
		<tr>
			<td>debug&#95;points</td>
			<td>
				가능한 값: on이나 off<br/>
				구문:<br/>
				debug&#95;points stop;<br/>
				기본값: None<br/>
				엔진엑스 내부의 디버그 지점을 활성화시킨다. stop은 붙어 있는 디버거가 디버그 지점에 다다랐을 때 프로그램을 중지시키며, abort는 디버그 지점에서 프로그램을 중단시키고 코어 덤프 파일을 생성하게 한다. 이 기능을 비활성화하려면 지시어를 사용하지 않는다.
			</td>
		</tr>
		<tr>
			<td>env</td>
			<td>
				구문:<br/>
				env MY&#95;VARIABLE;<br/>
				env MY&#95;VARIABLE=my&#95;value;<br/>
				환경 변수를 정의하거나 덮어씌운다.
			</td>
		</tr>
		<tr>
			<td>env</td>
			<td>
				구문:<br/>
				env MY&#95;VARIABLE;<br/>
				env MY&#95;VARIABLE=my&#95;value;<br/>
				환경 변수를 정의하거나 덮어씌운다.
			</td>
		</tr>
		<tr>
			<td>
				error&#95;log<br/>
				맥락: main, http, mail, stream, server, location
			</td>
			<td>
				구문:<br/>
				error&#95;log /file/path level;<br/>
				기본값: logs/error.log error<br/>
				level은 가장 상세한 로그를 남기는 debug부터 info, notice, warn, error, crit, alert, 그리고 가장 치명적인 오류만 보고하는 emerg 중 하나로 지정할 수 있다. 오류 로그를 끄고 싶으면 로그 출력을 /dev/null로 지정하면 된다. 구성 파일의 최상위에 다음과 같이 지정해보라.<br/>
				error&#95;log /dev/null crit;<br/>
				파일 경로 대신 다른 것을 지정할 수도 있다. 표준 오류 출력 장치로 보내고 싶으면 stderr, 시스템 로그로 보내고 싶으면 syslog, 메모리로 보내고 싶으면 memory로 지정할 수 있다.
			</td>
		</tr>
		<tr>
			<td>load&#95;module</td>
			<td>
				구문: 파일 경로<br/>
				load&#95;module modules/ngx&#95;mail&#95;module.so;<br/>
				기본값: 없음<br/>
				동적 모듈을 읽어 활성화한다. 상호 배제를 위한 락 파일을 지정한다. 컴파일할 때 켜지 않는 한 기본적으로는 꺼져있다. 락은 대부분의 운영체제에서 원자적 연산으로 락을 구현하기 때문에 이 지시어는 무시된다.
			</td>
		</tr>
		<tr>
			<td>lock&#95;file</td>
			<td>
				구문: 파일 경로<br/>
				lock&#95;file logs/nginx.lock;<br/>
				기본값: 컴파일할 때 결정된 값<br/>
				상호 배제를 위한 락 파일을 지정한다. 컴파일할 때 켜지 않는 한 기본적으로는 꺼져있다. 락은 대부분의 운영체제에서 원자적 연산으로 락을 구현하기 때문에 이 지시어는 무시된다.
			</td>
		</tr>
		<tr>
			<td>master&#95;process</td>
			<td>
				가능한 값: on이나 off<br/>
				master&#95;process on;<br/>
				기본값: on<br/>
				기본적으로 활성화돼 있으며, 엔진엑스가 시작할 때 주 프로세스와 작업자 프로세스로 나뉘어 다중 프로세스로 동작한다. 비활성화시키면 엔진엑스는 단 하나의 프로세스로 동작하는데, 이때는 테스트 용도로만 사용해야 한다. 주 프로세스가 비활성화돼 클라이언트가 서버에 접속할 수 없기 때문이다.
			</td>
		</tr>
		<tr>
			<td>prce&#95;jit</td>
			<td>
				가능한 값: on이나 off<br/>
				prce&#95;jit on;<br/>
				펄 호환 정규식(이하 PCRE) 8.20 이상의 정규식을 실행 시점(Just-In-Time 이하 JIT)에 컴파일하는 것을 켜고 끈다. 켰을 때 처리 속도가 비약적으로 향상될 수 있다. 이 기능이 동작하려면 PCRE 라이브러리가 --enable-jit 옵션으로 빌드된 것이어야 하고, 엔진엑스도 --with-pcre-jit 옵션으로 빌드된 것이어야 한다.
			</td>
		</tr>
		<tr>
			<td>pid</td>
			<td>
				구문: 파일 경로<br/>
				pid&#95;logs/nginx.pid;<br/>
				기본값: 컴파일할 때 결정된 값<br/>
				엔진엑스 데몬의 pid 파일 경로다. 기본값은 컴파일할 때 결정된다. pid 파일은 운영체제에 종속적인 엔진엑스 init 스크립트에서 사용될 수 있기 때문에 이 지시어는 반드시 올바르게 설정돼야 한다.
			</td>
		</tr>
		<tr>
			<td>ssl&#95;engine</td>
			<td>
				구문: 문자열<br/>
				ssl&#95;engine enginename;<br/>
				기본값: 없음<br/>
				여기서 enginename은 시스템에서 사용 가능한 하드웨어 SSL 가속 명령어의 이름이다. 사용 가능한 하드웨어 SSL 가속 명령어를 확인하고 싶으면 다음 명령어를 셸에서 실행해보라.<br/>
				openssl engine -t
			</td>
		</tr>
		<tr>
			<td>thread&#95;pool</td>
			<td>
				구문:<br/>
				thread&#95;pool name threads=number [max&#95;queue=65536];<br/>
				기본값:<br/>
				thread&#95;pool default threads=32 max&#95;queue=65536;<br/>
				대용량 파일을 비동기로 처리하기 위한 aio 지시어와 함께 사용될 수 있는 스레드 풀 참조를 정의한다.
			</td>
		</tr>
		<tr>
			<td>timer&#95;resolution</td>
			<td>
				구문: 숫자(시간)<br/>
				timer&#95;resolution 100ms;<br/>
				기본값: 없음<br/>
				내부 시계를 동기화하는 gettimeofday() 시스템 호출 간격을 조절한다. 이 값이 지정되지 않으면 커널 이벤트 알림(kernel event notification)때마다 시계가 고쳐진다.
			</td>
		</tr>
		<tr>
			<td>user</td>
			<td>
				구문:<br/>
				user username groupname;<br/>
				user username;<br/>
				기본값: 컴파일할 때 결정됨.<br/>
				구성 지시어로도 정의되지 않으면 엔진엑스 주 프로세스의 사용자와 그 롤이 사용된다. 엔진엑스 작업자 프로세스를 시작시키는 사용자 계정과 그룹을 지정할 수 있다. 그룹이 필수는 아니다. 보안상의 이유로 제한된 권한의 사용자와 그룹을 지정해야 하는데, 예를 들면 엔진엑스 전용의 사용자와 그룹을 만들고 서비스되는 파일에 적절한 권한을 적응하는 것이다.
			</td>
		</tr>
		<tr>
			<td>worker&#95;cpu&#95;affinity</td>
			<td>
				구문:<br/>
				worker&#95;cpu&#95;affinity CPU&#95;마스크<br/>
				worker&#95;cpu&#95;affinity auto [CPU&#95;마스크];<br/>
				기본값: 없음<br/>
				이 지시어는 worker&#95;process와 함께 사용돼 동작하며, 작업자 프로세스와 CPU 코어 간의 선호도(affinity)를 부여한다.<br/>
				CPU&#95;마스크에는 작업자 프로세스 개수만큼의 이진수가 있고 각 이진수의 자릿수는 CPU 코어의 수와 같다. 엔진엑스가 세 개의 작업자 프로세스를 갖도록 설정했다면 세 벌의 이진수가 있을 것이고, 여기에 듀얼 코어 CPU라면 각 이진수는 자릿수가 둘일 것이다.<br/>
				worker&#95;cpu&#95;affinity 01 01 10;<br/>
				첫 번째 이진수 01은 첫 번째 작업자 프로세스가 두 번째 코어를 선호한다는 것을 나타낸다.<br/>
				두 번째 이진수 01은 두 번째 작업자 프로세스가 두 번째 코어를 선호한다는 것을 나타낸다.<br/>
				세 번째 이진수 10은 세 번째 작업자 프로세스가 첫 번째 코어를 선호한다는 것을 나타낸다.<br/>
				선호도는 멀티코어 CPU일 때만 권장되며, 하이퍼스레딩(hyper-threading)이나 유사한 기술에는 권장되지 않는다.<br/>
				auto 값이 지정되면 작업자 프로세스가 자동으로 가용한 CPU에 결합된다.
			</td>
		</tr>
		<tr>
			<td>worker&#95;priority</td>
			<td>
				구문: 숫자<br/>
				worker&#95;priority 0;<br/>
				기본값: 0<br/>
				작업자 프로세스의 우선순위를 최고 -20에서 최소 19까지의 범위에서 지정한다. 기본값은 0이다. 참고로 커널 프로세스는 우선순위 -5에서 실행되기 때문에 -5 이하의 우선순위는 설정하지 않는 것이 좋다.
			</td>
		</tr>
		<tr>
			<td>worker&#95;processes</td>
			<td>
				구문: 숫자나 auto<br/>
				worker&#95;processes 4;<br/>
				기본값: 1<br/>
				작업자 프로세스의 수를 지정한다. 엔진엑스는 요청 처리를 다수의 프로세스로 나눠준다. 기본값은 1이지만 CPU가 듀얼 코어 이상이라면 더 큰 값으로 지정하는 것이 좋다. 게다가 느린 I/O 수행으로 프로세스가 차단(block)되더라도 들어오는 요청을 다른 프로세스에 위임할 수 있다. 대신 엔진엑스가 이 지시어의 값을 적절히 선택하도록 auto를 사용할 수도 있다. 기본적으로 auto는 시스템에서 감지된 CPU 코어의 수가 된다.
			</td>
		</tr>
		<tr>
			<td>worker&#95;shutdown&#95;timeout</td>
			<td>
				구문: 시간<br/>
				worker&#95;shutdown&#95;timeout 10s;<br/>
				기본값: 없음<br/>
				안전하게 종료(graceful shutdown)할 때 진행 중인 작업자 프로세스가 끝내도록 대기할 시한을 정한다. 시한이 지나면 모든 연결을 강제로 끊고 프로세스를 종료한다.
			</td>
		</tr>
		<tr>
			<td>worker&#95;rlimit&#95;core</td>
			<td>
				구문: 숫자(크기)<br/>
				worker&#95;rlimit&#95;core 100m;<br/>
				기본값: 없음<br/>
				작업자 프로세스당 코어 파일의 크기를 지정한다.
			</td>
		</tr>
		<tr>
			<td>worker&#95;rlimit&#95;nofile</td>
			<td>
				구문: 숫자<br/>
				worker&#95;rlimit&#95;nofile 10000;<br/>
				기본값: 없음<br/>
				작업자 프로세스가 동시에 사용할 수 있는 파일의 개수를 지정한다.
			</td>
		</tr>
		<tr>
			<td>working&#95;directory</td>
			<td>
				구문: 디렉터리 경로<br/>
				working&#95;directory /usr/local/nginx;<br/>
				기본값: 컴파일할 때 결정되는 기준 경로(prefix)<br/>
				작업 디렉터리(working directory)는 작업자 프로세스가 코어 파일의 위치를 지정하는 용도로만 사용된다. user 지시어로 지정되는 작업자 프로세스의 사용자 계정은 이 디렉터리에 코어 파일을 쓰려면 반드시 쓰기 권한이 있어야 한다.
			</td>
		</tr>
		<tr>
			<td>worker&#95;aio&#95;requests</td>
			<td>
				구문: 숫자<br/>
				worker&#95;aio&#95;requests 10000;<br/>
				aio에 epoll 연결 처리 방식을 사용 중이라면 이 지시어는 작업자 프로세스 하나의 최대 비동기 I/O 작업수를 설정한다.
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 2.2.4. 이벤트 모듈
<br/>

<table>
	<thead>
		<tr>
			<th>지시어 이름</th>
			<th>구문과 설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>accept&#95;mutex</td>
			<td>
				가능한 값: on, off<br/>
				accept&#95;mutex on;<br/>
				기본값: off<br/>
				소켓 연결을 열고자 mutex(상호 배제)를 사용할지 여부를 결정한다.
			</td>
		</tr>
		<tr>
			<td>accept&#95;mutex&#95;delay</td>
			<td>
				구문: 숫자(시각)<br/>
				accept&#95;mutex&#95;delay 500ms;<br/>
				기본값: 500밀리초<br/>
				자원을 다시 얻으려고 시도하기 전에 작업자 프로세스가 기다리는 시간을 지정한다. accept&#95;mutex 지시어가 off이면 사용되지 않는다.
			</td>
		</tr>
		<tr>
			<td>debug&#95;connection</td>
			<td>
				구문: IP 주소나 CIDR 블록<br/>
				debug&#95;connection 172.63.155.21;<br/>
				debug&#95;connection 172.63.155.0/24;<br/>
				기본값: 없음<br/>
				이 IP 주소나 주소 블록과 일치하는 요청에 대해 상세 로그를 남긴다. 이 디버깅용 정보는 error&#95;log 지시어에서 지정한 파일에 디버그 레벨로 기록된다. 이 기능을 사용하려면 반드시 엔진엑스를 --debug 스위치와 함께 컴파일해야 한다.
			</td>
		</tr>
		<tr>
			<td>multi&#95;accept</td>
			<td>
				구문: on이나 off<br/>
				multi&#95;accept off;<br/>
				기본값: off<br/>
				엔진엑스가 수신 큐에서 들어와 있는 연결을 한 번에 다 받게 할지 여부를 결정한다.
			</td>
		</tr>
		<tr>
			<td>use</td>
			<td>
				가능한 값: /dev/poll, epoll, eventport, kqueue, rtsig, select<br/>
				use kqueue;<br/>
				기본값: 컴파일할 때 결정<br/>
				어떤 이벤트 모델을 사용할지 선택한다. 컴파일할 때 활성화한 모델 중에서 선택할 수 있다. 엔진엑스가 자동으로 가장 적합한 모델을 선택하기 때문에 사용자가 이 값을 바꿀 필요가 없다. 사용할 수 있는 모델은 다음과 같다.
				ㆍselect: 기본적이고 표준적인 모델이다. OS가 더 좋은 모델을 지원하지 않을 때 사용된다. 윈도우 환경에서만 사용할 수 있다. 높은 부하를 감당하는 서버에서는 권장되지 않는다.<br/>
				ㆍpoll: select보다는 우선적으로 선택되지만 모든 시스템에서 사용할 수 있는 것은 아니다.<br/>
				ㆍkqueue: FreeBSD 4.1 이상, OpenBSD 2.9 이상, NetBSD 2.0 이상, 맥 OS X 이상에서 사용할 수 있는 효율적인 방식이다.<br/>
				ㆍepoll: 리눅스 2.6 이상에서 효율적이다.
				ㆍrtsig: 리눅스 2.2.19부터 사용할 수 있는 실시간 신호 방식인데, 큐의 수용량이 최대 1,024까지만 설정 가능하기 때문에 트래픽이 많은 환경에서는 적합하지 않다.<br/>
				ㆍ/dev/poll: 솔라리스(Solaris) 7 11/99 이상, HP/UX 11.22 이상, IRIX 6.5.15 이상, Tru64 유닉스 5.1A 이상의 운영체제에서 효율적인 방식이다.<br/>
				ㆍeventport: 보안 패치가 필요하긴 하지만, 솔라리스 10에서 효율적인 방식이다.
			</td>
		</tr>
		<tr>
			<td>worker&#95;connections</td>
			<td>
				구문: 숫자<br/>
				worker&#95;connections 1024;<br/>
				기본값: 없음<br/>
				작업자 프로세스가 동시에 처리할 수 있는 연결의 수를 지정한다.
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 2.2.6. 필수 조정
<br/>

+ user root root;  
이 지시어는 작업자 프로세스가 root 사용자로 시작하도록 root 사용자로 시작하도록 지정하는데, 파일 시스템의 전체 권한을 엔진엑스에 부여하기 때문에 보안상 위험하다. 시스템에 새로운 사용자 계정을 만들고 그 계정을 사용하게 해야 한다. 권장 값(www-data 사용자와 그룹 계정을 부여하게 함): user www-data www-data;  

+ workder&#95;processes 1;  
이 설정은 작업자 프로세스 하나만 시작하게 하는데, 모든 요청이 하나의 실행 경로로 처리되며, CPU 코어 하나로 실행됨을 의미한다. 이 값은 늘리는 것이 좋은데, CPU 코어당 최소 하나의 프로세스를 갖도록 설정한다. 또는 auto로 설정해서 엔진엑스가 최적의 값을 결정하게 하는 것도 좋다.  
권장 값: workder&#95;processes auto;  

+ worker&#95;priority 0;  
기본적으로 작업자 프로세스는 일반적인 프로세스 우선순위로 시작된다. 시스템이 다른 작업을 동시에 수행하는 경우에 엔진엑스 작업자 프로세스에 높은 우선순위를 부여할 수 있다. 프로세스 우선순위는 낮은 값이 높은 우선순위를 의미하기 때문에 이 경우 숫자를 낮춰야 한다. 값의 범위는 최고 우선순위인 -20부터 최저 우선순위인 19까지 가능하다. 하지만 커널 프로세스가 -5의 우선순위를 갖기 때문에 더 낮은 수를 주면 안 된다. 필요한 상황에 따라 전혀 달라지므로 별도의 권장 값은 없다.  

+ log&#95;not&#95;found on;  
이 지시어는 엔진엑스가 404 errors 로그를 남길 것인지 아닌지 지정한다. 물론 이 로그가 누락된 자원에 대한 유용한 정보를 제공하긴 하지만, 일명 파비콘(fabicon)에 웹 브라우저가 접근하거나 검색 로봇이 robots.txt에 접근하는 수많은 로그가 쌓이게 된다.  

+ worker&#95;connections 1024;  
이 설정은 작업자 프로세스의 수와 함께 서버가 동시에 수용할 수 있는 연결 수를 결정한다. 예를 들어 각각 1024개의 연결을 수용하는 작업자 프로세스가 4개라면 서버는 동시 연결을 최대 4096개까지 처리하게 된다. 이 설정은 보유한 하드웨어에 맞게 조장해야 한다. 더 많은 RAM과 고성능의 CPU를 가진 서버일수록 더 많은 동시 연결을 수용할 수 있다. 대용량 트래픽을 서비스하는 괴물 같은 서버라면 이 설정 값을 올리고 싶을 것이다.  

#### 2.3. 서버 테스트
<br/>

##### 2.3.2. 성능 테스트
<br/>

서버의 성능을 측정하는 세 가지 도구가 있다. 각 도구 모두 웹 서버의 부하 테스트에 특화돼 있고 태생적으로 서로 다른 접근 방식을 갖고 있다.  

+ httperf: HP에서 개발되고 상대적으로 잘 알려진 오픈소스 도구며, 리눅스만 지원한다.
+ Autobench: 펄(Perl)로 httpref와 for를 래핑한 도구이며, 테스트 동작 방식을 개선하고 상세한 보고서를 생성하고자 만들어졌다.
+ OpenWebLoad: 윈도우와 리눅스 모두를 지원하는 경량화된 오픈소스 부하테스트 도구다.  

###### 2.3.2.1. Httperf
<br/>

Httperf는 공식 웹 사이트 http://www.hpl.hp.com/research/linux/httperf/에서 다운로드할 수 있는 간단한 명령행 도구다. 각자 사용하는 OS의 기본 패키지 저장소에서도 구할 수 있을 것이다. 소스코드는 tar.gz로 압축돼 있으며, 일반적인 방법인 ./configure make, make install로 컴파일해야 한다.  

일단 설치하고 나면 다음 명령을 실행할 수 있다.  

```sh
httperf --server 192.168.1.10 --port 80 --uri /index.html --rate 300 --num-conn 30000 --num-call 1 --timeout 5
```

이 명령에서 숫자는 원하는 대로 바꿀 수 있다.  

+ --server: 테스트하려는 웹 사이트 호스트 이름
+ --uri: 다운로드할 파일의 경로
+ --rate: 전송할 초당 요청 횟수
+ --num-call: 연결마다 전송될 요청 횟수
+ --timeout: 요청이 실패했다고 판단할 시한  

###### 2.3.2.2. 오토벤치
<br/>

오토벤치(Autobench)는 httperf를 더욱 효과적으로 사용하게 해주는 펄(Perl) 스크립트다. 이 도구는 서버가 포화 상태가 될 때까지 요청 비율을 자동으로 높이면서 테스트를 계속 실행한다. 오토벤치의 인상적인 기능 하나는 .tsv 보고서를 생성하는 기능인데, 이 파일을 다양한 애플리케이션에서 열어 그래프를 생성할 수 있다.  

여러 호스트를 한 번에 테스팅할 수 있지만, 최대한 단순하게 단일 호스트만 사용하겠다. 실행할 명령은 httperf의 예와 비슷하다.  

```sh
autobench --single_port --host1 192.168.1.10 --uri1 /index.html --quiet --low_rate 20 --high_rate 200 --rate_step 20 --num_call 10 --num_conn 5000 --timeout 5 --file results.tsv
```

구성할 수 있는 스위치는 다음과 같다.  

+ --host1: 테스트하려는 웹 사이트 호스트 이름
+ --uri1: 다운로드할 파일 경로
+ --quiet: httperf 정보를 화면에 출력하지 않도록 차단
+ --low&#95;rate: 테스트 시작 시 초당 연결 횟수
+ --high&#95;rate: 테스트 종료 시 초당 연결 횟수
+ --rate&#95;step: 각 테스트마다 연결 횟수 증가율
+ --num&#95;call: 연결마다 전송될 요청 횟수
+ --num&#95;conn: 총 연결 횟수
+ --timeout: 요청이 실패했다고 판단할 시한
+ --file: 지정한 (.tsv) 파일로 보고서 출력  

테스트가 종료되면 마이크로소프트 엑셀 같은 애플리케이션에서 읽어 들일 수 있는 .tsv 파일을 얻게 된다.  

###### 2.3.2.3. 오픈웹로드
<br/>

오픈웹로드(OpenWebLoad)는 무료 오픈소스 애플리케이션이다. 리눅스와 윈도우 플랫폼 모두 지원하며 21세기 초반, 웹 1.0 시대에 개발됐다. 오픈웹로드는 다른 접근법을 사용하는데, 서버에 요청 부하를 주고 얼마나 올바로 처리하는지 보는 대신 단순히 다양한 연결 횟수를 사용해 최대한 많은 요청을 보내고 나서 매초 리포트를 보여준다.  

오픈웹로드는 공신 웹 사이트인 http://openwebload.sourceforge.net에서 다운로드할 수 있다. .tar.gz 압축 파일을 푼 다음에 ./configure, make, make install 명령을 차례로 실행하자.  

사용법은 앞의 두 도구에 비해 단순하다.  

```sh
openload example.com/index.html 10
```

##### 2.3.3. 무중단 엔진엑스 업그레이드
<br/>

엔진엑스는 중단 없이 실행 파일을 바꿔치기 할 수 있는 메커니즘을 보유하고 있으므로 다음 과정만 조심해서 따른다면 요청 손실 0%를 보장한다.  

1. 기존 엔진엑스 실행 파일(기본은 /usr/local/nginx/sbin/nginx)을 새것으로 교체한다.  
2. ps x | grep nginx | grep master 같은 명령이나 pid 파일의 값을 통해 엔진엑스 주 프로세스의 PID를 알아낸다.  
3. kill -USR2 1234 같은 명령으로 USR2(12) 신호를 주 프로세스에 보낸다. 여기서 1234는 2단계에서 찾은 pid로 바꾼다. 이를 통해 기존 .pid 파일명이 바뀌고 새 실행 파일이 실행됨으로써 업그레이드가 시작될 것이다.  
4. kill -WINCH 1234 명령으로 기존 주 프로세스에 WINCH(28) 신호를 보낸다. 여기서 1234는 2단계에서 찾은 pid로 바꾼다. 이를 통해 기존 작업자 프로세스들이 작업이 끝난 순서대로 점차 종료된다.  
5. 모든 기존 작업자 프로세스가 종료됐는지 확인한 후에 kill -QUIT 1234 명령으로 기존 주 프로세스에 QUIT 신호를 보낸다. 여기서 1234는 2단계에서 찾은 pid로 바꾼다.  

### 3. HTTP 구성
<br/>

#### 3.1. HTTP 핵심 모듈
<br/>

##### 3.1.1. 구조 블록
<br/>

+ http  
이 블록은 구성 파일의 최상위에 삽입된다. 엔진엑스의 HTTP와 관련된 모듈 전부의 지시어와 블록은 http 블록에만 정의할 수 있다. 특별히 그럴만한 이뉴는 없지만, 이 블록은 여러 번 추가될 수 있는데, 이런 경우 뒤에 오는 블록의 지시어 값이 선행되는 블록의 지시어 값을 재지정하게 된다.  

+ server  
이 블록으로는 웹 사이트 하나를 선언할 수 있다. 다시 말해 엔진엑스가 특정 웹 사이트(하나 이상의 호스트 이름, 예를 들어 www.mywebsite.com같은 이름으로 식별됨)를 인식하고 그 구성을 얻는 블록이다. 이 블록은 http 블록 안에서만 사용할 수 있다.  

+ location  
웹 사이트의 특정 위치에만 적용되는 설정을 정의하는 데 쓰는 블록이다. 이 블록은 server 블록 안이나 다른 location 블록 안에 중첩해서 사용할 수 있다.  

http 블록 수준에서 설정을 하면 앞으로 포함될 server와 location 블록에도 이 설정이 유지된다.  

#### 3.2. 모듈 지시어
<br/>

##### 3.2.1. 소켓과 호스트 구성
<br/>

###### 3.2.1.1. listen
<br/>

웹 사이트를 제공하는 소켓을 여는 데 사용되는 IP 주소나 포트, 또는 두 가지 모두를 지정한다. 웹 사이트는 보통 HTTP 기본값은 80 포트나 HTTPS 기본값인 443 포트에서 제공된다.  

이 지시어는 유닉스 소켓도 지원한다.  

```
listen unix:/tmp/nginx.sock;
```

+ 구문: listen [주소][:포트] [추가 옵션];
+ 추가옵션 
  + default&#95;server  
  해당 server 블록을 지정된 IP주소와 포트로 들어온 모든 요청의 기본 웹 사이트로 지정
  + ssl  
  웹 사이트가 SSL을 통해 제공되도록 지정
  + http2  
  http&#95;v2 모듈이 있을 경우 HTTP/2 프로토콜을 지원하도록 활성화
  + proxy&#95;protocol  
  포트로 접속된 모든 네트워크 연결에 프록시 프로토콜을 활성화
  + 다른 옵션은 bind와 listen 시스템 호출과 관련됨  
  + setfib=숫자
  + fastopen=숫자
  + backlog=숫자
  + rcvbuf=크기
  + sndbuf=크기
  + accept&#95;filter=필터
  + deferred
  + bind
  + ipv6only=on|off
  + reuseport
  + so&#95;keepalive=on|off|[keepidle];[keepintvl]:[keepcnt]  
  

###### 3.2.1.2. server&#95;name
<br/>

server 블록에 하나 이상의 호스트 이름을 할당하는 지시어다. 엔진엑스는 HTTP 요청을 받을 때 요청의 Host 헤더를 server 블록 모두와 비교한다. 이 호스트 이름과 맞는 첫 번째 server 블록이 선택된다.  

+ 맥락: server
+ 대안  
아무런 server 블록도 요청의 호스트와 맞지 않다면 엔진엑스는 listen 지시어의 매개변수와 맞는 server 블록을 선택한다. 예를 들어 listen &#42;:80은 80 포트로 들어오는 모든 요청을 잡는 데 listen 지시어에 default 옵션이 활성화된 첫 블록에 우선권이 주어진다.  

이 지시어는 정규식은 물론 와일드카드도 사용할 수 있음을 명심하자. 정규식을 쓸 때 호스트 이름은 ~ 문자로 시작해야 한다.  

+ 예  
```
server_name www.website.com;
server_name www.website.com website.com;
server_name *.website.com;
server_name .website.com;
server_name *.website.*;
server_name ^(www)\.example\.com$;
```

이 지시어 값에 빈 문자열을 사용해 Host 헤더 없이 들어오는 모든 요청을 받게 할 수도 있다. 다만 적어도 하나의 정규 호스트 이름(또는 더미 호스트 이름인 "&#95;" 문자) 이 앞에 있어야 한다.  

###### 3.2.1.3. server&#95;name&#95;in&#95;redirect
<br/>

이 지시어는 내부적인 경로 재설정의 경우에 적용된다. on으로 설정하면 엔진엑스는 server&#95;name 지시어에 지정된 첫 호스트 이름을 사용한다. off로 설정하면 엔진엑스는 HTTP 요청의 Host 헤더 값을 사용한다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.1.4. server&#95;names&#95;hash&#95;max&#95;size
<br/>

엔진엑스는 요청을 처리하는 속도를 향상시키고자 해시 테이블을 여러 데이터 컬렉션에 사용한다. 이 지시어는 서버 이름 해시 테이블의 최대 크기를 정의한다. 기본값은 대부분 구성에 잘 맞는다. 이 값을 변경할 필요가 있다면 엔진엑스가 기동할 때나 구성을 다시 읽을 때 자동으로 알려줄 것이다.  

+ 맥락: http
+ 구문: 숫자 값
+ 기본값: 512  

###### 3.2.1.5. server&#95;names&#95;hash&#95;bucket&#95;size
<br/>

서버 이름 해시 테이블의 최대 버킷 크기를 설정한다. 엔진엑스가 알려줄 때에만 이 값을 바꿔야 한다.  

+ 맥락: http
+ 구문: 숫자 값
+ 기본값: 32(또는 64, 또는 128, 사용되는 프로세서 캐시 사양에 따름)  

###### 3.2.1.6. port&#95;in&#95;redirect
<br/>

경로 재설정 상황에 이 지시어는 엔진엑스가 포트 번호를 재설정되는 URL에 추가할지 여부를 지정한다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: on  

###### 3.2.1.7. tcp&#95;nodelay
<br/>

연결 유지(Keep-alive) 상황에서 TCP&#95;NODELAY 소켓 옵션을 활성화하거나 비활성화한다. 소켓 프로그래밍에 대한 리눅스 문서를 인용하면 다음과 같다.  

> "TCP&#95;NODELAY"는 내글 버퍼링 알고리즘(Nagle buffering algorithm)을 비활성화하는 특수 목적에 쓰인다. 즉각적인 반응 없이 낮은 빈도의 정보를 주기적으로 보내는 데이터 전송이 적기에 이뤄져야 하는 애플리케이션에서만 사용해야 한다. 마우스 이동이 전형적인 예다."  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: on  

###### 3.2.1.8. tcp&#95;nopush
<br/>

TCP&#95;NOPUSH(FreeBSD)나 TCP&#95;CORK(리눅스) 소켓 옵션을 활성화 또는 비활성화한다. 이 옵션은 sendfile 지시어가 활성화돼야 적용된다. tcp&#95;nopush가 on이면 엔진엑스는 모든 HTTP 응답 헤더를 단일 TCP 패킷에 전송하려고 시도한다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.1.9. sendfile
<br/>

이 지시어가 활성화되면 엔진엑스는 sendfile 커널 호출을 사용해서 파일을 전송한다. 비활성화되면 엔진엑스는 스스로 파일을 전송한다. 전송될 파일의 (NFS 같은) 물리적인 위치에 따라 이 옵션은 서버 성능에 영향을 미친다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.1.10. sendfile&#95;max&#95;chunk
<br/>

이 지시어는 sendfile 호출마다 사용될 데이터 최대 크기를 정의한다.  

+ 맥락: http, server
+ 구문: 숫자 값(크기)
+ 기본값: 0  

###### 3.2.1.11. send&#95;lowat
<br/>

FreeBSD에서 TCP 소켓에 SO&#95;SNDLOWAT 플래그를 사용하게 하는 옵션이다. 이 값은 출력 동작용 버퍼의 최소 바이트 수를 정의한다.  

+ 맥락: http, server
+ 구문: 숫자 값(크기)
+ 기본값: 0  

###### 3.2.1.12. reset&#95;timedout&#95;connection
<br/>

클라이언트 연결이 시한 만료가 될 때 연결의 상태에 따라 이 연결과 연관된 정보가 메모리에 남게 된다. 이 지시어를 활성화하면 시한이 만료된 후에 이 연결과 연관된 모든 정보가 삭제된다. 1.15.2부터는 "444 응답 없이 연결 닫음" 코드를 반환한다.  

+ 맥략: http, server, location
+ 구문: on이나 off
+ 기본값: off  

##### 3.2.2. 경로와 문서
<br/>

###### 3.2.2.1. root
<br/>

이 지시어는 방문자에게 제공하고자 하는 파일을 담고 있는 최상위 문서 위치를 정의한다.  

+ 맥락: http, server, location, if, 변수 사용 가능
+ 구문: 디렉터리 경로
+ 기본값: html  

```
root /home/website.com/public_html;
```

###### 3.2.2.2. alias
<br/>

alias는 location 블록 안에서만 쓸 수 있다. 엔진엑스가 특정 요청에서 별도 경로의 문서를 읽도록 할당한다. 다음 구성 예를 살펴보자.  

```
http {
	server {
		server_name localhost;
		root /var/www/website.com/html;
		location /admin/ {
			alias /var/www/locked;
		}
	}
}
```

http://localhost/ 요청이 들어오면 /var/www/website.com/html/ 폴더의 파일이 제공된다. 하지만 엔진엑스가 http://localhost/admin/ 요청을 받아 파일을 읽을 때 사용되는 경로는 /var/www/locked/다. 그렇다고 최상위 문서 위치를 나타내는 root 지시어의 값이 변경되는 것은 아니다. 이 작업 절차는 동적 스크립트의 눈에 보이지 않는다.  

+ 맥락: location, 변수 사용 가능
+ 구문: 디렉터리(끝이 / 문자로 끝나야 함)나 파일 경로  

###### 3.2.2.3. error&#95;page
<br/>

HTTP 응답 코드에 맞춰 URI를 조작하거나 이 코드를 다른 코드로 대체한다.  

+ 맥락: http, server, location, if, 변수 사용 가능
+ 구문: error&#95;page code1 [code2...] [=대체 코드] [&#64;block | URI]

```
error_page 404 /not_found.html;
error_page 500 501 503 504 /server_error.html;
error_page 403 http://website.com/;
error_page 404 @notfound; # 지정한 location 블록으로 이동
error_page 404 =200 /index.html; # 404 오류의 경우, 응답 코드를 200 OK로 바꾸고 index.html로 경로를 돌림
```

###### 3.2.2.4. if&#95;modified&#95;since
<br/>

엔진엑스가 If-Modified-Since HTTP 헤더를 처리하는 방법을 정의한다. 이 헤더는 검색 엔진 스파이더(구글 웹 크롤링 봇 같은 것)가 주로 사용한다. 이 로봇은 마지막으로 받은 파일의 날짜와 시간을 알려준다. 그 시간 이후 요청된 파일이 수정된 적이 없다면 서버는 본문 없이 단순히 "304 변경되지 않음" 응답 코드만 반환한다.  

이 지시어에 다음과 같은 세 가지 값을 지정할 수 있다.  

+ off  
If-Modified-Since 헤더를 무시한다.

+ exact  
실제 요청된 파일의 수정 날짜와 HTTP 헤더에 지정된 날짜와 시간이 정확히 일치하면 "304 변경되지 않음" 응답 코드를 반환한다. 파일 수정 날짜가 이루거나 늦으면 이 파일은 정상적("200 OK" 응답)으로 제공된다.

+ before  
HTTP 헤더에 지정된 날짜와 시간이 요청된 파일의 수정된 날짜보다 이르거나 같으면 "304 변경되지 않음" 응답 코드를 반환한다.  

+ 맥락: http, server, location
+ 구문: if&#95;modified&#95;since off | exact | before
+ 기본값: exact  

###### 3.2.2.5. index
<br/>

요청에 아무런 파일명도 지정되지 않았을 때 (색인 페이지 또는 인덱스 페이지라고도 하는) 엔진엑스가 기본으로 제공할 페이지를 정의한다. 이 지시어에 여러 파일명을 지정하면 발견되는 첫 파일이 제공된다. 지정된 파일 중 아무것도 찾지 못했고 autoindex 지시어가 활성화돼 있다면 (HTTP 자동 색인 모듈 참고), 엔진엑스는 자동으로 파일 색인을 생성하려 할 것이다. 활성화되지 않았다면 "403 접근 금지" 응답 오류가 반환될 것이다. 원한다면 (/page.html 같이 최상위 문서 위치에 기반을 둔) 절대 파일 경로를 지정할 수도 있지만 이 지시어의 마지막 인자여야 한다.  

+ 맥락: http, server, location, 변수 사용 가능
+ 구문: index file1 [file2...] [절대&#95;파일&#95;경로];
+ 기본값: index.html  

```
index index.php index.html index.htm;
index index.php index2.php /catchall.php;
```

###### 3.2.2.6. recursive&#95;error&#95;pages
<br/>

가끔 error&#95;page 지시어로 제공되는 오류 페이지 자체가 오류를 일으켜서 다시 (재귀적으로) error&#95;page로 들어가는 경우가 있다. 이 지시어는 반복적인 오류 페이지 진입을 허용하거나 막는다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.2.7. try&#95;files
<br/>

첫 번째부터 마지막에서 두 번째까지 인자로 지정된 파일을 제공하려고 시도한다. 이 파일이 모두 존재하지 않는다면 마지막 인자에 지정된 이름의 location 블록으로 이동하거나 지정된 URI를 제공한다.  

+ 맥락: server, location, 변수 사용 가능
+ 구문: 하나 이상의 파일 경로를 나열하고 마지막에 location 블록 이름이나 URI  
+ 예:  

```
location / {
	try_files $uri $uri.html $uri.php $uri.xml @proxy;
}
# 다음은 "명명된 location 블록"이다.
location @proxy {
	proxy_pass 127.0.0.1:8080;
}
```

이 예에서 엔진엑스는 보통 때와 같이 파일을 제공하려고 한다. 요청된 URI가 존재하는 어떤 파일에도 해당하지 않는다면 엔진엑스는 .html을 URI 뒤에 추가하고 다시 파일을 제공하려고 해본다. 또 다시 실패하면 엔진엑스는 .php를, 그다움에는 .xml을 차례로 시도한다. 결국 모든 시도가 실패하면 다른 location 블록인 proxy가 이 요청을 처리한다.  

요청에 해당하는 이름의 디렉터리가 존재하는지 확인하고자 $uri/를 인자 목록에 지정할 수도 있다.  

##### 3.2.3. 클라이언트 요청
<br/>

이제는 엔진엑스가 클라이언트 요청을 처리하는 방법을 정리한다. 무엇보다 연결유지(Keep-alive) 메커니즘의 동작 방식을 구성하고 클라이언트 요청을 파일에 로그로 남기게 할 수 있다.  

###### 3.2.3.1. keepalive&#95;requests
<br/>

한 연결을 닫지 않고 유지하면서 제공할 최대 요청 횟수를 지정한다.  

+ 맥락: http, server, location
+ 구문: 숫자 값
+ 기본값: 100  

###### 3.2.3.2. keepalive&#95;timeout
<br/>

서버가 유지되는 연결을 끊기 전에 몇 초를 기다릴지 정의하는 지시어다. 옵션인 두 번째 매개변수는 Keep-Alive: timeout= HTTP 응답 헤더의 값으로 전달된다. 이 시간이 지난 후에는 클라이언트가 스스로 연결을 끊게 하려는 의도다. 어떤 브라우저는 이 설정을 무시하니 유의하자. 예를 들어 인터넷 익스플로러는 60초가 지나면 자동으로 연결을 닫는다.  

+ 맥락: http, server, location  
+ 구문: keepalive&#95;timeout 시간1 [시간2];
+ 기본값: 75  

```
keepalive_timeout 75;
keepalive_timeout 75 60;
```

###### 3.2.3.3. keepalive&#95;disable
<br/>

이 옵션으로 연결 유지 기능을 비활성화할 브라우저를 선택할 수 있다.  

+ 맥락: http, server, location
+ 구문: keepalive&#95;disable browser1 browser2;
+ 기본값: msie6  

###### 3.2.3.4. send&#95;timeout
<br/>

지정된 시간이 지난 후에는 엔진엑스가 비활성 상태의 연결을 닫는다. 클라이언트가 데이터 전송을 중단하는 순간부터 연결은 비활성 상태가 된다.  

+ 맥락: http, server, location
+ 구문: 시간 값(초 단위)
+ 기본값: 60  

###### 3.2.3.5. client&#95;body&#95;in&#95;file&#95;only
<br/>

이 지시어가 활성화되면 들어오는 HTTP 요청의 본문 데이터가 디스크에 실제 파일로 지정된다. 클라이언트의 본문 데이터는 클라이언트 HTTP 요청 전체 데이터에서 헤더 부분을 뺀 나머지다(다시 말해 POST 요청의 내용 부분이다). 파일은 일반 텍스트 문서로 저장된다.  

이 지시어에는 다음과 같은 세 가지 값을 인자로 쓸 수 있다.  

+ off: 데이터를 파일에 저장하지 않는다.
+ clean: 요청된 본문 데이터를 파일에 저장한 후 요청이 처리되고 나면 이 파일을 삭제한다.
+ on: 요청 본문 데이터를 파일에 저장하지만 요청이 처리되고 난 후에도 제거하지 않는다(디버깅 목적으로만 사용하기를 권고).  

+ 구문: client&#95;body&#95;in&#95;file&#95;only on | clean | off
+ 기본값: off  

###### 3.2.3.6. client&#95;body&#95;in&#95;single&#95;buffer
<br/>

이 지시어는 엔진엑스가 요청 본문 데이터를 메모리에 있는 단일 버퍼에 저장할지 여부를 정의한다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.3.7. client&#95;body&#95;buffer&#95;size
<br/>

이 client&#95;body&#95;buffer&#95;size 지시어는 클라이언트 요청의 본문 데이터를 보관할 버퍼의 크기를 지정한다. 요청의 크기가 너무 크면 본문이나 그 일부가 디스크에 저장된다. client&#95;body&#95;in&#95;file&#95;only 지시어가 활성화되면 요청 본문은 크기에 상관없이 (버퍼에 들어가든 말든) 언제나 디스크의 파일로 저장됨을 명심하자.  

+ 구문: 크기 값
+ 기본값: 8k나 16k(2 메모리 페이지) 컴퓨터 아키텍처에 따라 다름  

###### 3.2.3.8. client&#95;body&#95;temp&#95;path
<br/>

클라이언트 요청 본문 파일이 저장될 디렉터리 경로를 정의할 수 있는 옵션이다. 부가 옵션을 지정하면 3단계까지 폴더 계층 구조로 파일을 분할할 수 있다.  

+ 맥락: http, server, location
+ 구문: client&#95;body&#95;temp&#95;path [1단계] [2단계] [3단계]
+ 기본값: client&#95;body&#95;temp  

```
client&#95;body&#95;temp&#95;path /tmp/nginx_rbf;
client&#95;body&#95;temp&#95;path 2; # 엔진엑스가 요청 본문 파일을 보관할 두 자릿수 폴더를 생성함
client&#95;body&#95;temp&#95;path 1 2 4; # 엔진엑스가 세 단계의 폴더를 생성함
										 # (첫 단계: 한 자리, 둘째 단계: 두 자리, 셋째 단계: 네 자리)
```

###### 3.2.3.9. client&#95;body&#95;timeout
<br/>

클라이언트 요청 본문 데이터를 읽는 동안 적용될 비활성 시한을 정의하는 지시어다. 연결은 클라이언트가 데이터 전송을 중단하는 순간 비활성화한다. 기한이 차면 엔진엑스는 "408 요청 시한 만료" HTTP 오류를 반환한다.  

+ 맥락: http, server, location
+ 구문: 시간 값(초 단위)
+ 기본값: 60  

###### 3.2.3.10. client&#95;header&#95;buffer&#95;size
<br/>

엔진엑스가 요청 헤더에 할당할 버퍼의 크기를 정의할 수 있는 지시어다. 보통 1k로 충분하다. 하지만 때로는 헤더가 대규모의 쿠기 데이터나 무척 긴 요청 URI를 포함한다. 그런 경우가 생기면 엔진엑스는 하나 이상의 대규모 버퍼를 할당하는데, 이 버퍼의 크기는 large&#95;client&#95;header&#95;buffers 지시어로 정의된다.  

+ 구문: 크기 값
+ 기본값: 1k  

###### 3.2.3.11. client&#95;header&#95;timeout
<br/>

이 지시어는 클라이언트 요청 헤더를 읽는 동안 적용될 비활성 시한을 정의한다. 연결은 클라이언트가 데이터 전송을 중단하는 순간 비활성화한다. 기한이 차면 엔진엑스는 "408 요청 시한 만료" HTTP 오류를 반환한다.  

+ 맥락: http, server, location
+ 구문: 시간 값(초 단위)  
+ 기본값: 60  

###### 3.2.3.12. client&#95;max&#95;body&#95;size
<br/>

클라이언트 요청 본문 데이터의 최대 크기다. 이 크기가 넘으면 엔진엑스는 "413 요청 내용 용량 초과" HTTP 오류를 반환한다. 이 설정은 사용자가 HTTP로 서버에 파일을 올리도록 할 때 특히 중요하다.  

+ 맥락: http, server, location
+ 구문: 크기 값
+ 기본값: 1m  

###### 3.2.3.13. large&#95;client&#95;header&#95;buffers
<br/>

client&#95;header&#95;buffer&#95;size로 정한 기본 버퍼가 부족할 경우 클라이언트 요청을 저장하는 데 사용될 대용량 버퍼의 개수와 크기를 정한다. 헤더의 줄 하나하나는 버퍼 하나에 들어가야 한다. 요청된 URI가 버퍼 하나의 크기보다 크면 엔진엑스는 "414 길이 초과 요청 URI" 오류를 반환한다. 다른 헤더 한 줄이 버퍼 크기를 넘으면 엔진엑스는 "400 잘못된 요청" 오류를 반환한다.  

+ 맥락: http, server, location
+ 구문: large&#95;client&#95;header&#95;buffers 개수 크기 값
+ 기본값: 4&#42;8k  

###### 3.2.3.14. lingering&#95;time
<br/>

이 지시어는 본문이 있는 클라이언트 요청에 적용된다. 전송되는 데이터가 max&#95;client&#95;body&#95;size 크기를 넘자마자 엔진엑스는 바로 "413 요청 내용 용량 초과" 응답을 보낸다. 하지만 대부분 브라우저는 이 알림을 무시하고 계속 데이터를 전송한다. 이 지시어는 엔진엑스가 오류를 전송하고 연결을 닫기까지 기다릴 시간을 지정한다.  

+ 맥락: http, server, location
+ 구문: 시간 값
+ 기본값: 30초  

###### 3.2.3.15. lingering&#95;timeout
<br/>

이 지시어는 엔진엑스가 클라이언트 연결을 닫기 전에 읽는 두 작업 사이에서 기다려야 할 시간을 정의한다.  

+ 맥락: http, server, location
+ 구문: 시간 값
+ 기본값: 5초  

###### 3.2.3.16. lingering&#95;close
<br/>

이 lingering&#95;close 지시어는 엔진엑스가 클라이언트 연결을 닫는 방식을 제어한다. 모든 요청 데이터가 수신되는 즉시 연결을 닫으려면 이 지시어를 off로 설정하자. 기본값인 on은 엔진엑스가 필요하다면 추가 데이터를 기다리면서 처리하게 허용한다. always로 설정하면 엔진엑스는 연결을 닫기 전에 얼마간 기다린다. 기다릴 시간은 lingering&#95;timeout 지시어로 정의한다.  

+ 맥락: http, server, location
+ 구문: on, off, always
+ 기본값: on  

###### 3.2.3.17. ignore&#95;invalid&#95;headers
<br/>

이 지시어가 비활성화되면 엔진엑스는 구문이 잘못된 요청 헤더가 들어올 경우 "400 잘못된 요청" HTTP 오류를 반환한다.  

+ 맥락: http, server
+ 구문: on이나 off
+ 기본값: on  

###### 3.2.3.18. chunked&#95;transfer&#95;encoding
<br/>

이 지시어는 HTTP 1.1 요청의 분할 전송 인코딩(chunked transfer encoding)을 활성화하거나 비활성화한다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: on  

###### 3.2.3.19. max&#95;ranges
<br/>

이 지시어는 클라이언트가 파일의 일부 내용을 요청할 때 허용할 바이트 수의 범위를 정의한다. 값을 지정하지 않으면 기본으로 제한을 두지 않게 된다. 0으로 설정하면 바이트로 범위를 요청받는 기능이 비활성화된다.  

+ 맥락: http, server, location
+ 구문: 크기 값  

##### 3.2.4. MIME 타입
<br/>

엔진엑스는 MIME 타입을 구성하는 데 유용한 두 가지 지시어 types와 default&#95;type을 제공한다. 이 둘은 문서의 기본 MIME 타입을 정의한다. 이 지시어는 응답에 포함돼 보내질 Content-Type HTTP 헤더에 영향을 미친다. 계속 읽어보자.  

###### 3.2.4.1. types
<br/>

이 지시어는 MIME 타입과 파일 확장자의 상관관계를 맺는데 쓰인다. 사실 이 지시어는 특별한 구문을 가진 블록이다.  

```
types {
	MIME타입1 확장자1;
	MIME타입2 확장자2 [확장자3...];
	[...]
}
```

엔진엑스는 어떤 파일을 제공할 때 파일 확장자를 확인해서 MIME 타입을 결정한다. 그리고는 그 MIME 타입을 응답에서 Content-Type HTTP 헤더의 값으로 보낸다. 이 헤더는 브라우저가 파일을 처리할 방법에 영향을 미칠 것이다. 예를 들어 요청된 파일의 MIME 타입이 application/pdf라면 브라우저는 이 파일을 다운로드만 하지 않고 MIME 타입에 연결된 플러그인을 사용해서 표시하려고 할 것이다.  

엔진엑스는 MIME 타입 기본 세트를 mime.types라는 별도 파일에 갖고 있는데, include 지시어로 병합된다.  

```
include mime.types;
```

이 파일이 중요 파일 확장자를 대부분 포함하고 있으므로 수정할 필요는 없을 것이다. 엔진엑스가 나열된 타입에서 제공하려는 파일의 확장자를 찾을 수 있다면 default&#95;type 지시어로 정의한 기본 타입이 사용된다.  

types 블록을 다시 선언해서 타입 목록을 재정의할 수 있다. 예를 들어 한 폴더의 모든 파일을 강제로 표시되는 대신 다운로드되게 할 수도 있다.  

```
http {
	include mime.types;
	[...]
	location /downloads/ {
		# 모든 MIME 타입 제거
		types { }
		default_type application/octet-stream;
	} [...]
}
```

어떤 브라우저는 파일명이 .html이나 .txt 같이 이미 알고 있는 확장자로 끝날 경우 MIME 타입을 무시하고 여전히 파일을 표시하려고 할 것이다.  

방문자의 브라우저가 파일을 처리하는 방법을 더 확실하고 결정적인 방식으로 제어하려면 add&#95;header 지시어를 통해 Content-Disposition HTTP 헤더를 사용해야만 한다.  

mime.types 파일이 병합되지 않았을 때 기본값은 다음과 같다.  

```
types {
	text/html html;
	image/gif gif;
	image/jpeg jpg;
}
```

###### 3.2.4.2. default&#95;type
<br/>

기본 MIME 타입을 정의한다. 엔진엑스가 파일을 제공할 때 Content-Type HTTP 응답 헤더 값으로 적절한 MIME 타입이 반환되게 하려면 파일 확장자로 types 블록에 선언된 타입 중에서 맞는 타입이 있는지 찾는다. 이 확장자가 알고 있는 MIME 타입 어느 것과도 일치하지 않는다면 default&#95;type 지시어의 값이 사용된다.  

+ 구문: MIME 타입
+ 기본값: text/plain  

###### 3.2.4.3. types&#95;hash&#95;max&#95;size
<br/>

MIME 타입 해시 테이블의 최대 크기를 정의한다.  

+ 맥락: http, server, location
+ 구문: 숫자 값
+ 기본값: 4k나 8k  

###### 3.2.4.4. types&#95;hash&#95;bucket&#95;size
<br/>

MIME 타입 해시 테이블의 버킷 크기를 설정하는 지시어다. 이 값은 엔진엑스가 알려줄 때에만 바꿔야 한다.  

+ 맥락: http, server, location
+ 구문: 숫자 값
+ 기본값: 64(CPU 캐시 라인 크기)  

##### 3.2.5. 제한과 제약
<br/>

###### 3.2.5.1. limit&#95;except
<br/>

이 지시어는 명시적으로 허용하는 것을 제외하고는 모든 HTTP 메서드를 막게 해준다. location 블록 안에서 POST 요청을 보내지 못하도록 클라이언트를 막는 등 일부 HTTP 메서드를 사용하지 못하도록 제약해야 할 때가 있다.  

```
location /admin/ {
	limit_except GET {
		allow 192.168.1.0/24;
		deny all;
	}
}
```

이 예제는 /admin/이란 location 블록에 제약을 가해 모든 방문자는 GET 메서드만 사용할 수 있다. allow 지시어(HTTP 접속 모듈에서 자세히 설명함)에 지정된 대로 내부 IP 주소를 사용하는 방문자는 이 제약의 영향을 받지 않는다. 방문자가 금지된 메서드를 사용한다면 엔진엑스는 "403 접근 금지" HTTP 오류를 반환할 것이다. GET 메서드는 HEAD 메서드를 포함하므로 GET을 허용하면 GET과 HEAD 모두 허용하게 된다.  

이 지시어의 구문은 다음과 같다.  

```
limit_except 메서드1 [메서드2...] {
	allow | deny | auth_basic | auth_basic_user_file | proxy_pass | perl;
}
```

###### 3.2.5.2. limit&#95;rate
<br/>

이 지시어로 개별 클라이언트 연결의 전송률을 제한할 수 있다. 1.17.0부터 변수를 사용할 수 있다.  

전송률은 초당 바이트로 표현된다.  

```
limit_rate 500k;
```

이 설정으로 연결 전송률은 초당 500킬로바이트로 제한될 것이다. 클라이언트가 연결 두 개를 열면 이 클라이언트에게 2 × 500 킬러바이트까지 전송량이 허용된다.  

+ 맥락: http, server, location, if
+ 구문: 크기 값
+ 기본값: 무제한  

###### 3.2.5.3. limit&#95;rate&#95;after
<br/>

limit&#95;rate 지시어가 효력을 발휘하기 전에 전송되는 데이터의 양을 정의한다. 1.17.0부터 변수를 사용할 수 있다.  

```
limit_rate 10m;
```

엔진엑스는 처음 10메가바이트를 최고 속력으로 보낸다. 이 크기를 넘으면 전송률이 limit&#95;rate 지시어에 지정한 값으로 제한된다. limit&#95;rate 지시어와 비슷하게 이 설정은 연결 하나에 적용된다.  

+ 구문: 크기 값
+ 기본값: 없음  

###### 3.2.5.4. satisfy
<br/>

satisfy 지시어는 클라이언트가 모든 접근 조건을 만족해야 하는지, 아니면 하나만 해당하면 충족되는지 정의한다.  

이 예에서 클라이언트가 자원에 접근하려면 다음 두 가지 조건을 통과해야 한다.  

+ (HTTP 접근 모듈의) allow와 deny 지시어를 통해 내부 IP 주소를 갖는 클라이언트만 허용한다. 다른 클라이언트는 접근이 거부된다.
+ (HTTP 인증 모듈의) auth&#95;basic과 auth&#95;basic&#95;file 지시어를 통해 유효한 사용자 이름과 비밀번호를 입력한 클라이언트만 허용한다.  

satisfy all이라면 클라이언트는 자원에 접근할 궎나을 얻고자 두 조건 모두를 만족시켜야 한다. satisfy all이라면 클라이언트는 자원에 접근할 권한을 얻고자 두 조건 모두를 만족시켜야 한다. satisfy any라면 클라이너트가 두 조건 중 하나만 만족해도 접근이 승인된다.  

+ 맥락: location
+ 구문: satisfy any|all
+ 기본값: all  

###### 3.2.5.5. internal
<br/>

이 지시어는 location 블록을 내부용으로 지정한다. 다시 말해 지정된 자원은 외부 요청이 접근하지 못한다.  

```
server {
	[...]
	server_name .website.com;
	location /admin/ {
		internal;
	}
}
```

이 구성에서 클라이언트는 http://website.com/admin/ 페이지를 열어볼 수 없다. 이런 요청은 "404 찾을 수 없음" 오류를 접하게 될 것이다. 이 자원에 접근할 유일한 방법은 내부 경로 재설정(internal redirect)뿐이다(내부 경로 재설정에 대한 정보는 '재작성 모듈
' 절을 확인하자).  

##### 3.2.6. 파일 처리와 캐시
<br/>

###### 3.2.6.1. disable&#95;symlinks
<br/>

이 지시어로 엔진엑스가 심볼릭 링크를 웹으로 제공해야 할 때 이를 다루는 방법을 제어한다. 기본적으로 (지시어 값 off) 심볼릭 링크가 허용되며, 엔진엑스는 링크가 가리키는 파일을 찾는다. 다음 값 중 하나를 사용해서 특정 조건에서 심볼릭 링크가 가리키는 파일을 따라가며 찾지 않도록 할 수 있다.  

+ on: 요청 URI의 특정 부분이 심볼릭 링크라면 이 접근은 거부되고 엔진엑스는 "403 HTTP" 오류 페이지를 반환한다.
+ if&#95;not&#95;onwer: on과 비슷하지만 링크와 링크가 가리키는 대상의 소유자가 서로 다를 때 접근이 거부된다.
+ 옵션인 from= 매개변수를 지정하면 URL의 특정 부분은 심볼릭 링크 여부를 확인하지 않는다. 예를 들어 disable&#95;symlinks on from=$document&#95;root라고 설정하면 엔진엑스는 $document&#95;root 폴더 이전까지는 심볼릭 링크를 정상적으로 따른다. 그 이후로는 심볼릭 링크가 URI 일부로 발견되면 요청된 파일은 접근이 거부된다.  

###### 3.2.6.2. directio
<br/>

이 지시어가 활성화되면 지정된 값보다 크기가 큰 파일은 다이렉트 I/O로 읽혀진다. 다이렉트 I/O는 엔진엑스가 중간의 캐시 처리 과정 없이 저장 장치에서 메모리로 직접 데이터를 읽을 수 있게 해준다.  

+ 맥락: http, server, location
+ 구문: 크기 값이나 off
+ 기본값: off  

###### 3.2.6.3. directio&#95;alignment
<br/>

이 지시어는 directio를 사용할 때 바이트 정렬 크기를 설정한다. 리눅스에서 XFS를 사용한다면 이 값을 4k로 설정하라.  

+ 맥락: http, server, location
+ 구문: 크기 값
+ 기본값: 512  

###### 3.2.6.4. open&#95;file&#95;cache
<br/>

이 지시어로 열린 파일의 정보를 캐시에 보관하게 할 수 있다. 이 캐시는 실제 파일 내용을 저장하지는 않고 다음의 정보만 갖고 있다.  

+ 파일 서술자(파일 크기, 수정 시간 등)
+ 파일과 디렉터리의 존재 여부
+ 접근 거부, 파일 없음 등의 파일 오류, open&#95;file&#95;cache&#95;errors 지시어로 비활성화할 수 있다.  

이 지시어에 쓸 수 있는 인자는 두 가지다.  

+ max=X, X는 캐시가 저장할 항목의 수다. 항목이 지정한 값만큼 많아지면 오래된 항목은 새 항목을 위한 공간을 마련하고자 제거된다.
+ inactive=Y, Y는 캐시 항목이 저장될 시간을 초로 나타낸 값이다. 기본적으로 엔진엑스는 캐시 항목을 지우기 전에 60초간 기다린다. 캐시 항목이 사용될 때마다 이 시간은 초기화된다. 캐시 항목이 open&#95;file&#95;cache&#95;min&#95;uses에 정의된 값보다 자주 사용되면 엔진엑스가 공간이 부족해서 오래된 항목을 지우기로 결정하지 않는 한 캐시 항목은 지워지지 않는다.  

+ 맥락: http, server, location
+ 구문: open&#95;file&#95;cache max=X [inactive=Y] 또는 off
+ 기본값: off  
+ 예:
```
open_file_cache max=5000 inactive=100;
```

###### 3.2.6.5. open&#95;file&#95;cache&#95;errors
<br/>

이 지시어는 open&#95;file&#95;cache 지시어로 파일 오류를 캐시에 보관할지 여부를 정한다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.6.6. open&#95;file&#95;cache&#95;min&#95;uses
<br/>

기본적으로 open&#95;file&#95;cache의 항목은 일정 시간 이상(기본으로 60초) 사용되지 않으면 삭제된다. 하지만 일정 정도 이상 사용된 경우에는 엔진엑스가 캐시 항목을 지우지 못하게 막을 수 있다. 이 지시어는 삭제되지 않게 막으려면 한 항목이 몇 번 사용돼야 하는지 정의한다.  

```
open_file_cache_min_uses 3;
```

캐시 항목이 3회 이상 캐시에서 읽혔다면 영구적으로 활성 상태가 되며, 엔진엑스가 공간을 확보하고자 오래된 항목을 제거하기로 결정하지 않는 한 제거되지 않는다.  

+ 맥락: http, server, location
+ 구문: 숫자 값
+ 기본값: 1  

###### 3.2.6.7. open&#95;file&#95;cache&#95;valid
<br/>

열린 파일 캐시 메커니즘은 중요하지만, 캐시된 정보는 금방 쓸모없어진다. 특히 빠르게 움직이는 파일 시스템에서는 그렇다. 이런 이유로 정보는 짧은 시간이 지난 후에 다시 검증돼야 한다. 이 지시어는 엔진엑스가 얼마 후에 캐시 항목을 다시 검증할지 정의한다.  

+ 맥락: http, server, location
+ 구문: 시간 값(초 단위)
+ 기본값: 60  

###### 3.2.6.8. read&#95;ahead
<br/>

read&#95;ahead 지시어는 파일에서 미리 읽어둘 바이트 수를 정의한다. 리눅스 기반 운영체제에서 이 지시어를 0 이상 값으로 설정하면 미리 읽는 기능이 활성화되지만 지정한 실제 숫자는 아무런 영향을 미치지 못한다. 0으로 설정하면 미리 읽는 기능이 비활성화된다.  

+ 맥락: http, server, location
+ 구문: 크기 값
+ 기본값: 0  

##### 3.2.7. 기타 지시어
<br/>

###### 3.2.7.1. log&#95;not&#95;found
<br/>

이 지시어는 "404 찾을 수 없음" HTTP 오류를 로그에 남길지 말지 결정한다. favicon.ico나 robots.txt를 찾다가 못 찾는 404 오류로 로그가 가득하다면 이 기능을 끄고 싶을 것이다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: on  

###### 3.2.7.2. log&#95;subrequest
<br/>

내부 경로 재설정(internal redirect)이나 SSI 요청에 의해 발생된 2차 요청을 요구에 남길지 결정한다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.7.3. merge&#95;slashes
<br/>

이 지시어를 활성화하면 URI의 연속되는 슬래시 문자를 하나로 합치게 된다. 결과적으로 다음과 같은 상황에 특히 유용하다.  

```
server {
	[...]
	server_name website.com;
	location /documents/ {
		type { }
		default_type text/plain;
	}
}
```

기본적으로 클라이언트가 http://website.com/documents/에 접근한다고 하자(URI 중간에 있는 '//'를 주의하자), 엔진엑스는 "404 찾을 수 없음" HTTP 오류를 반환할 것이다. 이 지시어를 활성화하면 두 슬래시 문자는 하나로 합해지고 이 location 블록의 패턴과 일치하게 된다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.7.4. msie&#95;padding
<br/>

이 지시어는 마이크로소프트 인터넷 익스플로러(MSIE)와 구글 크롬 브라우저에서 동작한다. (오류 코드 400이상) 오류 페이지의 경우 응답 본문의 길이가 512 미만이면 이 브라우저는 자체 오류 페이지를 표시하는데, 종종 서버가 보내주는 정보가 풍부한 페이지가 무용지물이 된다. 이 옵션을 활성화하면 상태 코드가 400이나 그 이하인 응답의 본문이 512바이트가 될 때까지 다른 데이터로 채워진다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.7.5. msie&#95;refresh
<br/>

또 다른 MSIE 전용 지시어로, HTTP 응답 코드가 "301 영구 이동"과 "302 임시 이동"인 경우에 영향을 미친다. 활성화되면 엔진엑스는 응답 본문에 refresh 메타태그(&lt;meta http-equiv="Refresh"...&gt;)를 MSIE 브라우저가 동작하는 클라이언트에 보내서 요청한 자원의 새로운 위치로 브라우저가 이동하게 만든다.  

+ 맥락: http, server, location
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.7.6. resolver
<br/>

이 지시어는 엔진엑스가 호스트 이름으로 IP 주소를 찾거나 그 반대 작업을 할 때 사용할 DNS 서버를 지정한다. DNS 질의 결과는 DNS 서버가 제공하는 TTL을 반영하거나 valid 인자에 지정된 시간 값에 따라 일정 시간 캐시가 된다.  

+ 맥락: http, server, location
+ 구문: 하나 이상의 IPv4나 IPv6 주소, valid=시간 값, ipv6=on|off
+ 기본값: 없음(시스템 기본값)  

```
resolver 127.0.0.1; # 자체 DNS 사용
resolver 8.8.8.8 8.8.4.4 valid=1h; # 구글 DNS를 사용하고 결과를 1시간 동안 캐시
```

###### 3.2.7.7. resolver&#95;timeout
<br/>

호스트 이름 IP 변환 요청의 제한시간이다.  

+ 맥락: http, server, location
+ 구문: 시간 값(초 단위)
+ 기본값: 30  

###### 3.2.7.8. server&#95;tokens
<br/>

이 지시어로 엔진엑스가 실행되는 버전 정보를 클라이언트에게 알릴지 여부를 정의할 수 있다. 엔진엑스가 자신의 버전 번호를 알리는 상황은 세 가지가 있다.  

+ HTTP 응답의 서버 헤더 안(nginx/1.8.0처럼), server&#95;tokens를 off를 설정했다면 서버 헤더에는 엔진엑스를 쓴다는 사실만 남을 것이다.
+ 오류 페이지, 엔진엑스는 하단에 버전 번호를 표시한다. server&#95;token을 off로 설정하면 오류 페이지 하단에는 엔진엑스만 표시된다.
+ build로 설정하면 컴파일 시 --build 스위치에 지정한 값이 노출된다.  

구형 엔진엑스를 사용하고 업그레이드를 할 계획이 없다면 보안상 버전 번호를 숨기는 것이 좋다.  

+ 맥락: http, server, location
+ 구문: on|off|build
+ 기본값: on  

###### 3.2.7.9. underscores&#95;in&#95;headers
<br/>

사용자 정의 HTTP 헤더 이름에 밑줄 부호(&#95;)를 허용할지 여부를 지정하는 지시어다. 지시어가 on으로 설정되면 다음 예의 헤더는 엔진엑스에 유효하다.  

```
test_header: value
```

+ 맥락: http, server
+ 구문: on이나 off
+ 기본값: off  

###### 3.2.7.10. variables&#95;hash&#95;max&#95;size
<br/>

이 지시어는 변수 해시 테이블의 최대 크기를 정한다. 서버 구성에서 사용되는 변수의 총합이 1024개 이상이라면 이 값을 높여야 한다.  

+ 맥락: http
+ 구문: 숫자 값
+ 기본값: 1024  

###### 3.2.7.11. variables&#95;hash&#95;bucket&#95;size
<br/>

이 지시어로 변수 해시 테이블의 버킷 크기를 설정할 수 있는 지시어다.  

+ 맥락: http
+ 구문: 숫자 값
+ 기본값: 64(또는 32 또는 128. 프로세서 캐시 규격에 따라 다름)  

###### 3.2.7.12. post&#95;action
<br/>

이 post&#95;action 지시어는 요청 처리가 완료된 후에 엔진엑스가 호출하는 URI를 정의한다.  

+ 맥락: http, server, location, if
+ 구문: URI나 location 블록 이름
+ 예:
```
location /payment/ {
	post_action /scripts/done.php;
}
```

#### 3.3. 모듈 변수
<br/>

HTTP 핵심 모듈은 매우 많은 변수를 갖고 있어서 지시어의 값으로 사용할 수 있다. 소수의 지시어만 변수를 값 정의에 사용할 수 있도록 허용하니 주의하자. 변수를 허용하지 않는 지시어의 값에 변수를 사용하면 아무런 오류 메시지도 표시되지 않는다. 대신 변수 이름이 그대로 문자로 쓰일 뿐이다.  

변수에는 서로 다른 세 가지 종류가 있다. 첫 번째 유형은 클라이언트 요청의 헤더로 전송되는 값을 나타낸다. 두 번째 유형은 클라이언트에 보내지는 응답헤더에 대응된다. 마지막으로 세 번째 유형은 엔진엑스가 생성하는 변수로 구성된다.  

##### 3.3.1. 요청 헤더
<br/>

엔진엑스는 클라이언트 요청 헤더에 변수의 형태로 접근하게 해준다. 이를 나중에 구성에 사용할 수 있다.  

<table>
	<thead>
		<tr>
			<th>변수</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>$http&#95;host</td>
			<td>Host HTTP 헤더의 값, 클라이언트가 접근하기 원하는 호스트 이름을 나타내는 문자열이다.</td>
		</tr>
		<tr>
			<td>$http&#95;user&#95;agent</td>
			<td>User-Agent HTTP 헤더의 값, 클라이언트의 웹 브라우저를 나타내는 문자열이다.</td>
		</tr>
		<tr>
			<td>$http&#95;referer</td>
			<td>Referer HTTP 헤더의 값. 클라이언트가 마지막으로 방문했던 이전 페이지의 URL을 나타내는 문자열이다.</td>
		</tr>
		<tr>
			<td>$htt&#95;via</td>
			<td>Via HTTP 헤더의 값. 클라이언트가 사용했을 프록시에 대한 정보다.</td>
		</tr>
		<tr>
			<td>$http&#95;x&#95;forwarded&#95;for</td>
			<td>X-Forwarded-For HTTP 헤더의 값. 클라이언트가 프록시를 거쳐 접근할 경우 실제 클라이언트의 IP 주소다.</td>
		</tr>
		<tr>
			<td>$http&#95;cookie</td>
			<td>Cookie HTTP 헤더의 값. 클라이언트가 전송한 쿠키 데이터다.</td>
		</tr>
		<tr>
			<td>$http&#95;...</td>
			<td>그 외의 클라이언트가 보낸 헤더는 $http&#95; 접두사 뒤에 헤더 이름을 소문자로 바꾸고 '-' 문자는 '&#95;' 문자로 대치한 문자열을 붙여 만든 변수 이름으로 얻을 수 있다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 3.3.2. 응답 헤더
<br/>

비슷한 방식으로, 클라이언트에 보내지는 응답의 HTTP 헤더에 접근할 수 있다. 이 변수는 응답이 전송된 후에만 값을 갖기 때문에 항상 사용 가능한 건 아니다. 예를 들어 로그에 메시지를 기록할 때나 사용할 수 있다.  

<table>
	<thead>
		<tr>
			<th>변수</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>$sent&#95;http&#95;content&#95;type</td>
			<td>Content-Type HTTP 헤더의 값. 전송된 자원의 MIME 타입을 나타낸다.</td>
		</tr>
		<tr>
			<td>$sent&#95;http&#95;content&#95;length</td>
			<td>Content-length HTTP 헤더의 값. 응답 본문의 길이를 클라이언트에 알린다.</td>
		</tr>
		<tr>
			<td>$sent&#95;http&#95;location</td>
			<td>Location HTTP 헤더의 값. 요구된 자원의 위치가 원래 요청에 지정된 위치와 다를 때 그 위치를 나타낸다.</td>
		</tr>
		<tr>
			<td>sent&#95;http&#95;last&#95;modified</td>
			<td>Last-Modified HTTP 헤더의 값. 요청된 자원의 수정된 날짜에 해당한다.</td>
		</tr>
		<tr>
			<td>$sent&#95;http&#95;connection</td>
			<td>Connection HTTP 헤더의 값. 연결이 유지될 것인지 닫힐 것인지 정의한다.</td>
		</tr>
		<tr>
			<td>$sent&#95;http&#95;keep&#95;alive</td>
			<td>Keep-Alive HTTP 헤더의 값. 연결이 유지되는 시간을 정의한다.</td>
		</tr>
		<tr>
			<td>$sent&#95;http&#95;transfer&#95;encoding</td>
			<td>Transfer-Encoding HTTP 헤더의 값. (compress, gzip 같은) 응답 본문 인코딩 방법에 대한 정보를 제공한다.</td> 
		</tr>
		<tr>
			<td>$sent&#95;http&#95;cache&#95;control</td>
			<td>Cache-Control HTTP 헤더의 값. 클라이언트 브라우저가 자원을 캐시해야 할지 여부를 알려준다.</td>
		</tr>
		<tr>
			<td>$sent&#95;http&#95;...</td>
			<td>그 외의 클라이언트에 보내지는 헤더는 $sent&#95;http&#95; 접두사 뒤에 헤더 이름을 소문자로 바꾸고 '-' 문자는 '&#95;' 문자로 대치한 문자열을 붙여 만든 변수 이름으로 얻을 수 있다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 3.3.3. 엔진엑스 생성
<br/>

HTTP 헤더 외에도 엔진엑스는 요청. 요청이 처리된 방식과 앞으로 처리돼야 할 방식, 현재 구성에서 사용되는 설정에 관한 많은 변수를 제공한다.  

<table>
	<thead>
		<tr>
			<th>변수</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>$arg&#95;XXX</td>
			<td>GET 메서드의 매개변수를 나타내는 질의 문자열(query string)에 접근한다.</td>
		</tr>
		<tr>
			<td>$args</td>
			<td>모든 인자 값이 하나로 합쳐진 형태의 질의 문자열(query string)이다.</td>
		</tr>
		<tr>
			<td>$binary&#95;remote&#95;addr</td>
			<td>4바이트 이진 데이터 형태의 클라이언트의 IP 주소다.</td>
		</tr>
		<tr>
			<td>$body&#95;bytes&#95;sent</td>
			<td>응답의 본문으로 보내진 바이트 수다(응답 헤더는 포함되지 않음).</td>
		</tr>
		<tr>
			<td>$bytes&#95;sent</td>
			<td>클라이언트에 보내진 바이트 수다.</td>
		</tr>
		<tr>
			<td>$connection</td>
			<td>연결을 식별하기 위한 일련번호다.</td>
		</tr>
		<tr>
			<td>$connection&#95;rquests</td>
			<td>현재 사용되는 연결로 지금까지 처리된 요청 수다.</td>
		</tr>
		<tr>
			<td>$connection&#95;length</td>
			<td>Content-Length HTTP 헤더와 동일하다.</td>
		</tr>
		<tr>
			<td>$content&#95;type</td>
			<td>Content-Type HTTP 헤더와 동일하다.</td>
		</tr>
		<tr>
			<td>$cookie&#95;XXX</td>
			<td>쿠키 값. XXX는 활용하고 싶은 쿠키 매개견수 이름이다.</td>
		</tr>
		<tr>
			<td>$document&#95;root</td>
			<td>현 요청의 root나 alias 지시어의 값이다.</td>
		</tr>
		<tr>
			<td>document&#95;uri</td>
			<td>현 요청의 URI다. 내부 경로 재설정 처리 결과에 따라 원래 받은 요청의 URI와 다를 수 있다. $uri 변수와 동일하다.</td>
		</tr>
		<tr>
			<td>$host</td>
			<td>요청의 Host HTTP 헤더와 동일한 변수다. 엔진엑스는 요청에 Host 헤더가 없는 상황에 자체 값을 이 변수로 제공한다.</td>
		</tr>
		<tr>
			<td>$hostname</td>
			<td>서버 컴퓨터의 시스템 호스트명이다.</td>
		</tr>
		<tr>
			<td>$https</td>
			<td>HTTPS 연결일 때 on, 아니면 빈값이다.</td>
		</tr>
		<tr>
			<td>is&#95;args</td>
			<td>$args 변수가 정의됐으면 $is&#95;args는 값이 ?이고, $args가 비었으면 $is&#95;args 역시 빈값이다. 이 변수는 index.php$is&#95;args$args와 같이 옵션인 질의 문자열로 URI를 만드는 데 사용할 수 있다. 요청의 질의 문자열 인자가 있다면 $is&#95;args는 ? 값이므로 이 URI는 유효하게 된다.</td>
		</tr>
		<tr>
			<td>limit&#95;rate</td>
			<td>
				limit&#95;rate 지시어로 정한 전송률 제한 값이다. (재설정 모듈) set 지시어로 이 변수를 수정할 수 있다.<br/>
				set $limit&#95;rate 128k;
			</td>
		</tr>
		<tr>
			<td>$msec</td>
			<td>현재 시간(초 + 밀리초)이다.</td>
		</tr>
		<tr>
			<td>nginx&#95;version</td>
			<td>실행 중인 엔진엑스의 버전 번호다.</td>
		</tr>
		<tr>
			<td>$pid</td>
			<td>엔진엑스 프로세스의 ID다.</td>
		</tr>
		<tr>
			<td>$pipe</td>
			<td>현 요청이 한 연결의 여러 요청이 응답 대기 없이 연달아 들어오는 파이프라인(Piplelining)된 요청이라면 "p" 값으로, "."으로 설정된다.</td>
		</tr>
		<tr>
			<td>$proxy&#95;protocol&#95;addr</td>
			<td>listen 지시어의 proxy&#95;protocol 매개변수가 활성화되면 이 변수는 클라이언트 주소 값을 갖게 된다.</td>
		</tr>
		<tr>
			<td>$proxy&#95;protocol&#95;port</td>
			<td>listen 지시어의 proxy&#95;protocol 매개변수가 활성화되면 이 변수는 클라이언트 포트 값을 갖게 된다.</td>
		</tr>
		<tr>
			<td>$query&#95;string</td>
			<td>$args와 동일하다.</td>
		</tr>
		<tr>
			<td>$remote&#95;addr</td>
			<td>클라이언트의 IP 주소다.</td>
		</tr>
		<tr>
			<td>$remote&#95;port</td>
			<td>클라이언트 소켓의 포트다.</td>
		</tr>
		<tr>
			<td>$remote&#95;user</td>
			<td>인증을 가졌다면 클라이언트 사용자명을 반환한다.</td>
		</tr>
		<tr>
			<td>$realpath&#95;root</td>
			<td>현 요청의 최상위 문서 경로를 심볼릭 링크가 실제 경로로 변환된 상태로 반환한다.</td>
		</tr>
		<tr>
			<td>$request&#95;body&#95;file</td>
			<td>요청 본문이 저장됐다면 (client&#95;body&#95;in&#95;file&#95;only 지시어 참고) 이 변수는 임시 파일의 경로를 나타낸다.</td>
		</tr>
		<tr>
			<td>$request&#95;completion</td>
			<td>요청이 완료되면 'OK', 아니면 빈 문자열이다.</td>
		</tr>
		<tr>
			<td>$request&#95;filename</td>
			<td>현재 요청에서 제공된 전체 파일명이다.</td>
		</tr>
		<tr>
			<td>$request&#95;id</td>
			<td>임의로 생성된 16바이트 길이의 16진수 요청 고유 ID다.</td>
		</tr>
		<tr>
			<td>$request&#95;method</td>
			<td>GET이나 POST 같은 요청에 사용된 HTTP 메서드다.</td>
		</tr>
		<tr>
			<td>$request&#95;time</td>
			<td>클라이언트에서 첫 바이트를 읽은 이후로 경과된 시간(초 + 밀리초)이다.</td>
		</tr>
		<tr>
			<td>$request&#95;uri</td>
			<td>요청의 원래 URI에 해당한다. $document&#95;uri/$uri와 달리 처리 과정에도 수정되지 않고 유지된다.</td>
		</tr>
		<tr>
			<td>$scheme</td>
			<td>요청에 따라 http나 https를 반환한다.</td>
		</tr>
		<tr>
			<td>$server&#95;addr</td>
			<td>서버의 IP 주소다. 이 변수를 사용할 때마다 시스템 호출이 일어나므로 대용량 설정 상황에서 전체 성능에 영향을 미칠 수도 있으니 주의해서 사용해야 한다.</td>
		</tr>
		<tr>
			<td>$server&#95;name</td>
			<td>요청을 처리하는 중에 사용된 $server&#95;name 지시어의 값이다.</td>
		</tr>
		<tr>
			<td>$server&#95;port</td>
			<td>요청 데이터를 받은 서버 소켓 포트다.</td>
		</tr>
		<tr>
			<td>$server&#95;protocol</td>
			<td>프로토콜과 버전이다. 보통은 HTTP/1.0이나 HTTP/1.1이다.</td>
		</tr>
		<tr>
			<td>$status</td>
			<td>응답 상태 코드다.</td>
		</tr>
		<tr>
			<td>
				$tcpinfo&#95;rtt,<br/>
				$tcpinfo&#95;rttvar<br/>
				$tcpinfo&#95;snd&#95;cwnd,<br/>
				$tcpinfo&#95;rcv&#95;space
			</td>
			<td>운영체제가 TCP&#95;INFO 소켓 옵션을 지원하면 각 변수는 현재 클라이언트 TCP 연결의 정보 값을 갖게 된다.</td>
		</tr>
		<tr>
			<td>
				$time&#95;iso8601,<br/>
				$time&#95;local
			</td>
			<td>각 ISO 8601과 지역 형식으로 제공되는 access&#95;log 지시어에 쓰일 현재 시간이다.</td>
		</tr>
		<tr>
			<td>$uri</td>
			<td>$document&#95;uri와 동일하다.</td>
		</tr>
	</tbody>
</table>
<br/><br/>

#### 3.4. location 블록
<br/>

##### 3.4.1. 위치 조정 부호
<br/>

엔진엑스는 location 블록을 정의하면서 요청된 문서의 URI와 비교할 패턴을 지정할 수 있게 해준다.  

```
server {
	server_name website.com;
	location /admin/ {
		# http://website.com/admin/에 적용될 구성만 여기에 지정함
	}
}
```

단순히 폴더 이름 대신 복잡한 패턴을 사용할 수 있다. location 블록의 구문은 다음과 같다.  

```
location [=|~|~*|^~|@] 패턴 { ... }
```

처음 부분의 생략 가능한 인자는 위치 조정 부호라고 부르는 기호로, 지정된 패턴을 엔진엑스에 적용하는 방법을 정할 뿐 아니라 패턴의 성격(단순 문자열이나 정규식)도 정의한다. 아래는 여러 조정 부호와 그 동작 방식의 명세다.  

###### 3.4.1.1. = 조정 부호
<br/>

요청된 문서 URI는 반드시 지정된 패턴과 정확히 일치해야 한다. 여기에 쓰이는 패턴은 단순한 문자열이어야 한다. 정규식은 사용할 수 없다.  

```
server {
	server_name website.com;
	location = /abcd {
		[...]
	}
}
```

이 location 블록의 구성은 다음과 같이 처리된다.  

+ http://website.com/abcd에 적용됨(정확히 일치)
+ http://website.com/ABCD에 적용될 수 있음(운영체제가 대소문자를 구분하는 파일 시스템을 사용할 때에만 대소문자를 구분)
+ http://website.com/abcd?param1&param2에 적용됨(질의 문자열 인자에 상관없음)
+ http://website.com/abcd에 적용되지 않음(뒤에 붙은 '/' 기호)
+ http://website.com/abcd에 적용되지 않음(지정된 패턴 뒤에 글자가 더 있음)  

###### 3.4.1.2. 조정 부호 생략
<br/>

요청된 문서 URI가 지정된 패턴으로 시작해야 한다. 정규식은 사용할 수 없다.  

```
server {
	server_name website.com;
	location /abcd {
		[...]
	}
}
```

이 location 블록의 구성은 다음과 같이 처리된다.  

+ http://website.com/abcd에 적용됨(정확히 일치)
+ http://website.com/ABCD에 적용될 수 있음(운영체제가 대소문자를 구분하는 파일 시스템을 사용할 때에만 대소문자를 구분)
+ http://website.com/abcd?param1&param2에 적용됨(질의 문자열 인자에 상관없음)
+ http://website.com/abcd에 적용되지 않음(뒤에 붙은 '/' 기호)
+ http://website.com/abcd에 적용되지 않음(지정된 패턴 뒤에 글자가 더 있음)  

###### 3.4.1.3. ~ 조정 부호
<br/>

요청된 문서 URI가 지정된 정규식에 일치하는지 비교하면서 대소문자를 구분한다.   

```
server {
	server_name website.com;
	location ~ ^/abcd$ {
		[...]
	}
}
```

이 예제에 사용된 정규식 ^/abcd$에 따르면 패턴은 반드시 /로 시작(^)해야 하며, abc로 이어지고 d로 끝나야($) 한다. 결국 이 location 블록의 구성은 다음과 같이 처리된다.  

+ http://website.com/abcd에 적용됨(정확히 일치)
+ http://website.com/ABCD에 적용되지 않음(대소문자 구분)
+ http://website.com/abcd?param1&param2에 적용됨(질의 문자열 인자에 상관없음)
+ 지정된 정규식에 따라 http://website.com/abcd/에 적용되지 않음(뒤에 붙은 '/' 기호)
+ 지정된 정규식에 따라 http://website.com/abcde에 적용되지 않음(뒤에 글자가 더 있음)  

마이크로소프트 윈도우 같은 운영체제에서는 ~ 과 ~&#42; 모두 대소문자를 구분하지 않는다. 운영체제가 대소문자를 구분하지 않는 파일 시스템을 쓰기 때문이다.  

###### 3.4.1.4. ~&#42; 조정 부호
<br/>

요청된 URI가 지정된 정규식에 일치하는지 비교하면서 대소문자를 구분하지 않는다.  

```
server {
	server_name website.com;
	location ~* ^/abcd$ {
		[...]
	}
}
```

예에 사용된 정규식은 이전 예와 비슷하게 사용된다. 결국 이 location 블록의 구성은 다음과 같이 처리된다.  

+ http://website.com/abcd에 적용됨(정확히 일치)
+ http://website.com/ABCD에 적용됨(대소문자를 구분하지 않음)
+ http://website.com/abcd?param1&param2에 적용됨(질의 문자열 인자에 상관없음)
+ 지정된 정규식에 따라 http://website.com/abcd/에 적용되지 않음(뒤에 붙은 슬래시 '/')
+ 지정된 정규식에 따라 http://website.com/abcde에 적용되지 않음(뒤에 글자가 더 있음)  

###### 3.4.1.5. ^~ 조정 부호
<br/>

조정 부호가 생략된 경우와 비슷하게 동작한다. 위치 URI가 지정된 패턴으로 시작해야 한다. 패턴이 일치하면 엔진엑스는 다른 패턴을 찾지 않는 것이 차이점이다.  

###### 3.4.1.6. @ 조정 부호
<br/>

이름이 지정된 location 블록을 정의한다. 외부 클라이언트는 이 블록에 직접 접근할 수 없고 try&#95;files나 error&#95;page 같은 다른 지시어에 의해 생성된 내부 요청만 가능하다.  

##### 3.4.2. 탐색 순서와 우선순위
<br/>

서로 다른 패턴으로 다수의 location 블록을 정의할 수 있으므로 엔진엑스가 요청을 받았을 때 일어나는 일을 이해할 필요가 있다. 엔진엑스는 이 요청이 들어오면 해당 URI에 가장 부합하는 location 블록을 찾는다.  

```
server {
	server_name website.com;
	location /files/ {
		# "/files/"로 시작하는 모든 요청에 적용됨
		# 예, /files/doc.txt, /files/, /files/temp/
	}
	location = /files/ {
		# 정확히 "/files/"인 요청에 적용됨
		# /files/doc.txt에 적용되지 않음
	}
}
```

클라이언트가 http://website.com/files/doc.txt를 방문하면 첫 location 블록이 적용된다. 하지만 클라이언트가 http://website.com/files/에 방문하면 (첫 번째 블록이 일치함에도) 두 번째 블록이 적용된다. 두 번째 블록이 정확히 일치하므로 첫 블록보다 우선순위가 높기 때문이다.  

/files/ 블록을 =/files/ 블록 앞에 뒀지만 구성 파일에 블록을 나열한 순서는 상관이 없다. 엔진엑스는 특정 순서로 일치하는 패턴을 탐색한다.  

+ = 조정 부호가 사용된 location 블록: 요청된 URI와 지정된 문자열이 정확히 일치하면 엔진엑스는 이 location 블록을 후보로 기억한다.
+ 조정 부호가 생략된 location 블록: 요청된 URI가 지정된 문자열로 시작하면 엔진엑스는 이 location 블록을 유지한다.
+ ^~ 조정 부호가 사용된 location 블록: 요청된 URI가 지정된 문자열로 시작하면 엔진엑스는 이 location 블록을 유지한다.
+ ~ 또는 ~&#42; 조정 부호가 사용된 location 블록: 요청된 URI와 정규식이 일치하면 엔진엑스는 이 location 블록을 유지한다.
+ 조정 부호가 생략된 location 블록: 요청된 URI가 지정된 문자열로 시작하면 엔진엑스는 이 location 블록을 유지한다.  

### 4. 모듈 구성
<br/>

#### 4.1. 재작성 모듈
<br/>

재작성 모듈을 URL을 재작성하는 것이 목적이다. 이 메커니즘을 사용하면 여러 매개변수가 줄줄이 따라오는 보기 흉한 URL을 제거할 수 있다. 예를 들어 http://example.com/article.php?id=1234&comment=32 같은 URL은 특히 일반 방문자가 보기에 정보도 빈약하고 의미도 없다. 웹 사이트를 가리키는 링크는 유용한 정보를 포함해서 방문자가 방문하려는 페이지의 속성을 잘 드러내게 될 것이다. 예를 들어 보면 http://website.com/article-1234-32-US-economy-strengthens.html과 같을 것이다. 이런 방식은 방문자의 흥미를 끌 뿐 아니라 검색엔진의 관심도 받게 된다. URL 재작성은 검색 엔진 최적화(SEO, Search Engine Optimization)의 핵심 요소다.  

이 메커니즘을 뒷받침하는 원칙은 단순하다. 클라이언트 요청이 수신된 후 파일을 제공하기 전에 요청 URI를 재작성하는 것이다. 재작성하고 나면 재작성된 URI를 location 블록과 비교하면서 해당 요청에 적용할 서버 구성을 찾는다.  

##### 4.1.1. 정규식 복습
<br/>

URL 재작성은 rewrite 지시어로 처리되는데, 이 지시어는 정규식 패턴과 대체 URI를 인자로 받는다.  

###### 4.1.1.1. 목적
<br/>

###### 4.1.1.2. PCRE 구문
<br/>

엔진엑스가 도입한 정규식 구문은 펄 호환 정규식(PCRE, Perl Compatible Regular Expression) 라이브러리가 기원이다. 이 라이브러리는 이 모듈을 비활성화하지 않는 한 엔진엑스를 직접 컴파일하기 전에 미리 준비해야 하는 구성 요소다. PCRE는 가장 많이 사용되는 형태의 정규식이다.  

###### 4.1.1.3. 수량 표식
<br/>

###### 4.1.1.4. 캡처
<br/>

###### 4.1.1.5. 내부 요청
<br/>

엔진엑스에서는 외부 요청과 내부 요청을 구분한다. 외부 요청은 클라이언트에서 직접 온 요청이다. 외부 요청의 URI는 일치하는 location 블록에 대응된다.  

```
server {
	server_name website.com;
	location = /document.html {
		deny all; # 예제 지시어
	}
}
```

http://website.com/document.html로 들어오는 클라이언트 지시어는 이 location 블록에 직접 연결된다.  

이와 반대로 내부 요청은 엔진엑스에서 특수한 지시어에 의해 발생한다. 기본 엔진엑스 모듈에서 제공되는 지시어에는 내부 요청을 발생시키는 지시어가 여럿인데, error&#95;page, index, rewrite, try&#95;files 외에 첨가 모듈(Addition module)의 add&#95;before&#95;body, add&#95;after&#95;body, SSI 명령 등이 있다.  

내부 요청은 다음 두 가지 유형으로 나뉜다.  

+ 내부 경로 재설정  
엔진엑스는 클라이언트 요청을 내부에서 경로를 변경해 처리한다. 원래 URI가 바뀌기 때문에 이 요청은 다른 location 블록에 부합하는 것으로 판단되도록 다른 설정이 적용된다. 내부 요청이 쓰이는 가장 일반적인 경우는 rewrite 지시어가 쓰일 때다. 이 지시어는 요청 URI를 재작성한다.  

+ 부가 요청  
원래 요청을 보완할 내용을 생성하고자 내부에서 추가로 새로 만들어지는 요청이 있다. 단순한 예로 첨가 모듈을 들 수 있다. add&#95;after&#95;body 지시어는 원래 URI 후에 특정 URI가 처리되게 할 수 있다. 이렇게 생성된 내부 요청의 결과는 원래 요청의 본문 뒤에 첨가된다. SSI 모듈 역시 include SSI 명령으로 내용을 삽입하는 데 부가 요청을 사용한다.  

###### 4.1.1.6. error&#95;page 지시어
<br/>

엔진엑스 HTTP 핵심 모듈의 모듈 지시어에서 자세히 설명했던 error&#95;page는 특정 오류 코드가 발생했을 때 서버의 행위를 정의한다. 가장 단순한 형태는 오류 코드에 URI를 적용하는 것이다.  

```
server {
	server_name website.com;
	error_page 403 /errors/forbidden.html;
	error_page 404 /errors/not_found.html;
}
```

클라이언트가 (서버에 없는 문서나 파일을 읽어 404 오류가 발생한다거나 하는) 오류 중 하나를 일으키는 URI에 접근하려고 할 때 엔진엑스는 오류 코드에 연관된 페이지를 제공하게 된다. 사실 클라이언트에게 오류 페이지를 전송할 뿐 아니라 사실 새 URI에 따라 새로운 요청을 일으킨다.  

결과적으로 다음 예제와 같이 다른 구성으로 대체된다.  

```
server {
	server_name website.com;
	root /var/www/vhosts/website.com/httpdocs/;
	error_page 404 /errors/404.html;
	location /errors/ {
		alias /var/www/common/errors/;
		internal;
	}
}
```

클라이언트가 존재하지 않는 문서를 읽으려할 때 404 오류를 수신한다. 예에서는 error&#95;page 지시어를 사용해 404 오류가 발생하면 내부적으로 /errors/404.html로 경로가 재설정되도록 지정했다. 결국 엔진엑스는 /errors/404.html 새 요청을 생성한다. 이 URI는 location 블록 /errors/에 대응하게 되고 해당 구성이 적용된다.  

경로 변경과 URL 재작성 작업을 할 때는 로그가 특히 유용하다. error&#95;log 지시어를 debug로 설정했을 때에만 내부 경로 변경에 대한 정보가 로그에 보일 것이다. 필요하면 언제든 rewrite&#95;log on;으로 지정해서 notice 수준으로 경로 재작성 정보를 로그에 남길 수도 있다.  

location 블록에서 사용된 internal 지시어 덕에 클라이언트가 /errors/ 디렉터리에 접근하지 못하도록 차단된다는 점에 주의하자. 이 location 블록은 내부 경로 재설정을 통해서만 접근이 가능하다.  

이 메커니즘은 (색인 모듈 항목에서 자세히 다루는) index 지시어에서도 동일하게 적용된다. 클라이언트의 요청에 파일 경로가 명시되지 않았다면 엔진엑스는 내부 경로를 재설정해서 특정 색인 페이지를 제공하려고 할 것이다.  

###### 4.1.1.7. 재작성
<br/>

error&#95;page 지시어는 사실 재작성 모듈의 일부가 아니지만, 이 모듈의 기능을 자세히 알면 엔진엑스가 클라이언트 요청을 처리하는 방식을 명확히 알 수 있다.  

error&#95;page 지시어가 다른 위치로 경로를 재설정하는 방식과 비슷하게 rewrite 지시어로 URI를 재작성하면 내부 경로 재설정이 일어난다.  

```
server {
	server_name website.com;
	root /var/www/vhosts/website.com/httpdocs/;
	location /storage/ {
		internal;
		alias /var/www/storage/;
	}
	location /documents/ {
		rewrite ^/documents/(.*)$ /storage/$1;
	}
}
```

http://website.com/documents/file.txt로 들어오는 클라이언트 요청은 처음에 두 번째 location 블록(/documents/)에 일치한다. 하지만 이 블록에는 rewrite 지시어가 있어 이 URI를 /documents/file.txt에서 /storage/file.txt로 변환한다. 이렇게 URI를 변환하면 요청 처리 과정을 처음부터 다시 시작하며, 새 URI를 처리할 location 블록을 새로 찾는다. 이 경우 첫 번째 location 블록(/storage/)이 변경된 URI(/storage/file.txt)에 해당한다.  

다시 한 번 디버그 로그에서 해당 부분을 찾아보면 이 메커니즘을 자세히 알 수 있다.  

###### 4.1.1.8. 무한 루프
<br/>

구문과 지시어가 워낙 다양해서 헷갈리기 쉬운데 안타깝게도 사람뿐 아니라 엔직엔스도 헛갈리게 만들 수 있다. 재작성 규칙이 과하면 내부 경로 재설정이 무한히 반복하는 루프에 빠지는 경우가 그 예다.  

```
server {
	server_name website.com;
	location /documents/ {
		rewrite ^(.*)$ /documents/$1;
	}
}
```

문제없다고 생각하지만 이 구성은 사실 내부 경로 재설정을 일으켜서 /documents/로 시작하는 모든 경로를 /documents//documents/로 시작하게 바꾼다. 게다가 location 패턴이 내부 경로 재설정 후 다시 평가되므로, /documents// documents/로 시작되는 경로는 다시 /documents//documents//documents/로 시작하도록 바뀐다.  

다음은 디버그 로그에서 해당 부분을 발췌한 내용이다.  

이런 일이 무한하게 일어날까 궁금하겠지만 그렇지 않다. 이 순환 주기는 10회로 제한돼 있다. 내부 경로 재설정은 10회까지만 일어날 수 있다. 이 한계를 넘으면 엔진엑스는 "500 내부 서버" 오류를 발생시킬 것이다.  

###### 4.1.1.9. SSI
<br/>

SSI(Server Side Include) 모듈도 부가 요청의 근원 중 하나다. SSI는 PHP나 다른 전처리기와 비슷한 방식으로 서버가 클라이언트에 응답을 보내기 전에 문서를 분석하도록 하려는 것이 목적이다.  

(예를 들어) 일반 HTML 파일 안에 엔진엑스가 실행할 명령에 해당하는 태그를 삽입할 수 있게 된다.  

```html
<html>
	<head>
		<!-- include file="header.html" -->
	</head>
	<body>
		<!-- include file="body.html" -->
	</body>
</html>
```

엔진엑스는 이 두 명령을 처리한다. 이 경우 엔진엑스는 header.html과 body.html을 읽어서 해당 내용을 원래 문서에 삽입하고 클라이언트에 전송한다.  

여러 명령을 사용할 수 있는데, 자세한 내용은 SSI 모듈 항목에서 다룬다. 지금 당장은 include 명령이 궁금할 텐데, 이 명령은 다른 파일에 어떤 파일을 포함시킬 때 사용한다.  

```
<!--# include virtual="/footer.php?id=123" -->
```

지정된 파일을 단순히 고정된 위치에서 찾아 읽지 않는다. 부가 요청 전체가 엔진엑스에 의해 처리되며, 이 요청의 본문이 include 태그 자리에 대신 삽입된다.  

###### 4.1.1.10. 조건부 구조
<br/>

재작성 모듈에서 제공하는 새로운 지시어와 블록 중에는 if 조건부 구조도 있다.  

```
server {
	if ($request_method = POST) {
		[...]
	}
}
```

if 조건부 구조로 어떤 구성을 특정 조건에서만 적용하게 할 수 있다. 이 조건이 참이면 구성이 적용되고 그렇지 않으면 적용되지 않는다.  

+ None 없음  
지정된 변수나 데이터가 빈 문자열이거나 0이 아니라면 조건은 참이다.  
```
if ($string) {
	[...]
}
```

+ =, !=  
=에 앞선 인자와 뒤따르는 인자가 같으면 이 조건은 참이다. 다음 예는 "request&#95;method가 POST와 같다면 다음 구성을 적용하라"고 읽을 수 있다.  
```
if ($request_method = POST) {
	[...]
}
```
!=는 반대로 동작해서 아래는 "요청 메서드가 GET이 아니면 다음 구성을 적용하라"이다.  
```
if ($request_method != GET) {
	[...]
}
```

+ ~, ~, !~, !~  
~ 기호에 선행하는 인자가 그다음에 오는 정규식 패턴과 일치하면 이 조건은 참이다.  
```
if ($request_filename ~ "\.txt$") {
	[...]
}
```
~는 대소문자를 구분하지만 ~&#42;는 구분하지 않는다. ! 기호는 정규식 평가 결과를 반대로 만든다.  
```
if ($request_filename !~* "\.php$") {
	[...]
}
```
정규식으로 비교 값의 일부를 캡처할 수도 있다.  
```
if ($uri ~ "^/search/(.*)$") {
	set $query $1;
	rewrite ^ http://google.com/search?q=$query;
}
```

+ -f, !-f  
지정된 파일이 있는지 확인한다.  
```
if (-f $request_filename) {
	[...] # 파일이 존재하는 경우
}
```
! -f는 파일이 없는지 확인하는 데 쓰인다.  
```
if (!-f $request_filename) {
	[...] # 파일이 존재하지 않는 경우
}
```

+ -d, !-d  
-f 연산자와 비슷하게 디렉터리가 있는지 확인하는 데 쓰인다.  

+ -e, !-e  
-f 연산자와 비슷하게 파일, 디렉터리, 심볼릭 링크가 있는지 확인하는 데 쓰인다.  

+ -x, !-x  
-f 연산자와 비슷하게 파일이 있고 실행 가능한지 확인하는 데 쓰인다.  

버전 1.16.1을 기준으로 else나 else if와 유사한 명령은 없다. 하지만 제어 흐름을 제어할 수 있는 다른 지시어가 있다.  

어쩌면 if 블록 대신 location 블록을 사용해서 얻는 이득이 뭔지 궁긍할 것이다. 진짜로 다음 두 예는 결과가 동일하다.  

```
if ($uri ~ /search/) {
	[...]
}

location ~ /search/ {
	[...]
}
```

정확히 말하면 주요 차이점은 각 블록에서 사용할 수 있는 지시어에 있다. 어떤 지시어는 if 블록에서 사용할 수 있지만 어떤 지시어는 쓰지 못한다. 반대로 location 블록에는 지금까지 지시어 목록에서 봤던 것처럼 거의 모든 지시어를 사용할 수 있다. 보통 if 블록 안에는 재작성 모듈의 지시어만 넣는 것이 가장 좋다. 애초에 다른 지시어는 염두에 두고 만들지 않았다.  

###### 4.1.1.11. 지시어
<br/>

재작성 모듈은 URI을 재작성하는 것 이상의 지시어를 제공한다. 다음 표에서는 이러한 지시어와 해당 지시어를 사용할 수 있는 맥락을 설명한다.  

+ rewrite  
이전에 다룬 것처럼 rewrite 지시어는 현재 요청의 URI를 재작성할 수 있게 한다. 해당 요청은 재작성된 URI에 따라 처리 방식이 바뀐다.  

정규식은 URI가 대체되려면 일치돼야 할 패턴이다. 플래그는 다음 값 중 하나를 취한다.  
	+ 맥락: server, location, if  
	+ 구문: rewrite 정규식 대체 [플래그];
	+ last  
	현재 재작성 규칙이 적용될 규칙 중 마지막이어야 한다. 이 규칙이 적용되고 나면 엔진엑스는 재작성 작업을 끝내고 새 URI를 처리하기 시작하고 대응될 location 블록을 찾는다. 그 이상의 재작성 규칙은 무시된다.  
	+ break  
	현재 재작성 규칙이 적용되기는 하지만 엔진엑스가 수정된 URI로 새 요청을 처리하지 않는다(일치하는 location 블록을 찾지 않는다). 이후의 모든 rewrite 지시어는 무시된다.  
	+ redirect  
	"302 임시 이동" HTTP 응답이 반환된다. 대체된 URI는 location 헤더의 값에 사용된다.  
	+ permanent  
	"301 영구 이동" HTTP 응답이 반환된다. 대체된 URI는 location 헤더의 값에 사용된다.  
	+ http://로 시작하는 대체 URI를 지정하면 엔진엑스는 자동으로 redirect 플래그를 사용한다.
	+ 이 지시어로 처리되는 요청 URI는 호스트 이름과 프로토콜이 빠진 상대 URI이나 주의하자. http://website.com/documents/page.html 같은 요청의 경우 요청 URI는 /documents/page.html이다.
	+ 디코딩 여부  
	http://website.com/my%20page.html 같은 요청에 해당하는 URI는 /my page.html일 것이다. 인코딩된 URI의 %20은 공백 문자를 나타낸다.  
	+ 인자 포함 여부  
	http://website.com/page.php?id=1&p=2 같은 요청의 경우 URI는 /page.php가 된다. 이 URI를 재작성할 때 엔진엑스가 알아서 해주기 때문에 대체 URI에 인자를 포함하도록 신경 쓸 필요가 없다. 재작성된 URI 뒤에 엔진엑스가 인자를 추가하게 하기 싫다면 rewrite ^/ search/(.*)$/search.php?q=$1처럼 대체 URI 끝에 ? 문자를 추가해야 한다.  
	+ 예  
	```
	rewrite ^/search/(.*)$ /search.php?q=$1;
	rewrite ^/search/(.*)$ /search.php?q=$1?;
	rewrite ^ http://website.com;
	rewrite ^ http://website.com permanent;
	```

+ break  
break 지시어는 더 이상의 rewrite 지시어를 방지하는 데 사용된다. 이 지점 이후에는 URI 고정돼 변경할 수 없다.  
	+ 예  
	```
	if (-f $uri) {
		break; # 파일이 있으면 재작성 방지
	}
	if ($uri ~ ^/search/(.*)$) {
		set $query $1;
		rewrite ^ /search.php?q=$query?;
	}
	```
이 예는 /search/ 키워드 같은 검색 질의를 /search.php?q=키워드로 재작성한다. 하지만 요청된 파일이 존재한다면 (/search/index.html 같이) break 명령 때문에 엔진엑스가 이 URI를 재작성하지 못한다.  

+ return  
요청을 처리하는 것을 중단하고 특정 HTTP 상태 코드나 지정한 문자를 반환한다.  
코드는 상태 코드 204, 400~406, 408, 410, 411, 413, 416, 500~504 중 하나다. 여기에 더해 엔진엑스 고유 코드인 444를 추가적인 응답 헤더나 본문 없이 "HTTP 200 OK" 상태 코드를 반환하게 하는 데 사용할 수 있다.  
코드 대신 응답 본문으로 사용자에게 반환될 문자를 직접 지정할 수 있다. 요청이 특정 location 블록에 잘 들어갔는지 확인할 때 간편하게 사용할 수 있다.  
	+ 맥락: server, location, if
	+ 구문: return 코드 | 문자;
	+ 예  
	```
	if ($uri ~ ^/admin/) {
		return 403;
		# 엔진엑스가 이미 요청을 처리했으므로
		# 이 아래 명령은 실행되지 않음
		rewrite ^ http://website.com;
	}
	```

+ set  
변수를 초기화하거나 다시 정의한다. 어떤 변수는 재정의할 수 없으니 주의하자. 예를 들어 $url의 값은 변경할 수 없게 막혀 있다.  
	+ 맥락: server, location, if
	+ 구문: set $변수이름 값;
	+ 예  
	```
	set $var1 "some text";
	if ($var1 ~ ^(.*) (.*)$) {
		set $var2 $1$2; # 이어붙임
		rewrite ^ http://website.com/$var2;
	}
	```

+ uninitialized&#95;variable&#95;warn  
on으로 설정하면 아직 초기화되지 않은 변수를 구성에서 사용할 때 엔진엑스가 로그 메시지를 발행한다.  
	+ 맥락: http, server, location, if
	+ 구문: on이나 off
	```
	uninitialized_variable_warn on;
	```

+ rewrite&#95;log  
on으로 설정하면 재작성 엔진이 수행하는 모든 동작에 엔진엑스가 notice 오류 수준에서 로그 메시지를 발행한다(error&#95;log 지시어 참고)  
	+ 맥락: http, server, location, if
	+ 구문: on이나 off
	+ 기본값: off
	```
	rewrite_log off;
	```

##### 4.1.2. 일반 재작성 규칙
<br/>

#### 4.2. SSI 모듈
<br/>

##### 4.2.1. 모듈 지시어와 변수
<br/>

엔진엑스가 제공하는 파일의 실제 내용에 지시어를 넣는다면 한 가지 해결할 문제가 대두된다. SSI 명령을 처리하려면 엔진엑스가 어떤 파일을 분석해야 할까? (.gif, .jpg, .png 등의) 이미지나 다른 미디어 파일 같은 이진 파일을 분석하는 것은 자원을 낭비할 뿐이다. 이런 파일에는 SSI 명령이 들어있지 않을 것이기 때문이다. SSI 모듈에서 제공하는 지시어를 사용해서 엔진엑스를 올바르게 설정해야 한다.  

+ ssi  
SSI 명령을 처리하고자 파일을 분석한다. 엔진엑스는 ssi&#95;types 지시어로 선택된 MIME 타입과 일치하는 파일만 분석한다.  
	+ 맥락: http, server, location, if
	+ 구문: on이나 off
	+ 기본값: off
	```
	ssl on;
	```

+ ssi&#95;types  
SSI 분석에 적합한 파일의 MIME 타입을 정한다. text/html은 언제나 포함된다.  
	+ 맥락: http, server, location
	+ 구문: ssi&#95;types type1 [type2] [type3...]; ssi&#95;types &#42;
	+ 기본값: text/html
	```
	ssi_types text/plain;
	```

+ ssi&#95;silent&#95;errors  
일부 SSI 명령은 오류를 일으킨다. 이런 경우 엔진엑스는 지시어를 분석하는 중 오류가 발생한 명령 위치에 메시지를 출력한다. 이 옵션을 활성화하면 엔진엑스가 오류가 나도 메시지를 출력하지 않고 조용히 지나간다.  
	+ 구문: on이나 off
	+ 기본값: off
	```
	ssi&#95;silent&#95;errors off;
	```

+ ssi&#95;value&#95;length  
SSI 명령에는 값을 받는 인자가 있다(예를 들어 <!--# include file="value" -->), 이 지시어는 엔진엑스가 허용할 최대 길이를 정의한다.  
	+ 맥락: http, server, location
	+ 구문: 숫자 값
	+ 기본값: 256(문자)
	```
	ssi_value_length 256;
	```

+ ssi&#95;min&#95;file&#95;chunk
버퍼의 크기가 ssi&#95;min&#95;file&#95;chunk보다 크다면 데이터를 파일에 저장하고 sendfile 커널 호출을 사용해서 전송한다. 아니면 데이터를 메모리에서 직접 전송한다.  
	+ 맥락: http, server, location
	+ 구문: 숫자 값(크기)
	+ 기본값: 1,024  

+ ssi&#95;last&#95;modified  
on으로 설정하면 엔진엑스는 캐시될 가능성을 높이고자 SSI를 처리하는 동안 원래 응답의 Last-modified 헤더를 보존한다. 기본으로는 응답에 포함된 동적 생성 요소가 수정 날짜가 다르기 때문에 Last-modified 필드를 제거한다.  
	+ 맥락: http, server, location
	+ 구문: on이나 off
	+ 기본값: off  

SSI 엔진의 자원 사용과 관련해 우려할만한 점을 간단히 언급하자면 location이나 server 블록에서 SSI 모듈을 활성화하면 모든 (클라이언트 브라우저에 표시될 거의 모든 페이지만) text/html 파일을 분석하게 된다. 엔진엑스 SSI 모듈이 효율적으로 최적화되긴 했지만 필요 없는 파일은 분석하지 않게 하고 싶을 것이다.  

먼저 SSI 명령이 포함된 모든 페이지는 확장자가 .shtml(서버 HTML)이어야 한다. 그리고 location 블록의 구성에서 SSI 엔진을 특정 조건에서만 활성화되게 한다. 제공될 파일이 .shtml로 끝나야만 한다.  

```
server {
	server_name website.com;
	location ~* \.shtml$ {
		ssi on;
	}
}
```

이렇게 하면 엔진엑스에 보내지는 모든 HTTP 요청은 부가적인 정규식 패턴 확인 과정을 거쳐야 한다. 반면에 정적인 HTML 파일이나 (.php 같은) 다른 인터프리터에서 처리하는 파일은 불필요하게 분석하지 않는다.  

마지막으로, SSI 모듈로 다음 두 가지 변수가 생긴다.  

+ $date&#95;local: 시스템의 시간대에 따른 현재 시간을 반환한다.
+ $date&#95;gmt: 서버 시간대에 상관없이 GMT 기준의 현재 시간을 반환한다.  

##### 4.2.2. SSI 명령
<br/>

일단 웹 페이지에서 동작할 SSI 엔진이 활성화되면 첫 동적 HTML 페이지를 작성할 준비가 된 것이다. 다시 말하지만 원리는 단순하다. 일반 HTML 코드를 사용해서 웹 사이트의 페이지를 설계하라. 이 페이지 내부에 SSI 명령을 삽입하게 될 것이다.  

SSI 명령은 특정 구문을 따르지만, 처음에는 일반 HTML의 주석처럼 보인다. 주석처럼 보이게 함으로써 뜻하지 않게 SSI 분석 기능이 비활성화되더라도 이 SSI 명령은 클라이언트 브라우저에 표시되지 않고 실제 HTML 주석처럼 소스코드에서만 볼 수 있다. 전체 구문은 다음과 같다.  

```
<!--# 명령 매개변수1="값1" 매개변수2="값2" ... -->
```

###### 4.2.2.1. 외부 파일 포함
<br/>

SSI 모듈의 핵심 명령은 분명 include 명령이다. 이 명령은 두 가지 다른 방식으로 사용할 수 있다.  

첫째, 단순히 다른 파일을 포함시킬 수 있다.  

```
<!--# include file="header.html" -->
```

이 명령은 엔진엑스가 처리할 부가 요청을 생성한다. 생성된 응답의 본문은 include 명령 자리에 대신 삽입된다.  

둘째, include virtual 명령을 사용한다.  

```
<!--# include virtual="/sources/header.php?id=123" -->
```

이 명령도 서버에 부가 요청을 보낸다. 엔진엑스가 지정한 파일을 가져오는 방법이 다르다(include file을 사용할 때는 wait 매개변수가 자동으로 활성화된다). 사실 include 명령 태그에 넣을 수 있는 매개변수는 두 가지다. 기본적으로 모든 SSI 요청은 병렬로 동시에 발생된다. 이 때문에 서버 부하가 심할 때에는 속도가 늦어지거나 제한시간 내에 처리되지 않을 수 있다. 대신 wait="yes" 매개변수를 사용해서 엔진엑스가 다른 설정 파일을 포함하기 전에 이 요청이 완료되기를 대기하도록 할 수 있다.  

```
<!--# include virtual="/header.php" wait="yes" -->
```

include 명령의 결과가 빈값이거나 (404, 500 등의) 오류가 발생한다면 엔진엑스는 해당 오류 페이지의 &lt;html&gt;[...]404 Not Found&lt;/body&gt;&lt;/html&gt; 같은 HTML 코드를 그대로 삽입한다. 이 메시지는 include 명령을 삽입한 곳과 정확히 동일한 자리에 표시될 것이다. 이런 동작 방식을 바꾸고 싶다면 블록을 하나 만드는 방법이 있다. include 명령에 이 블록을 연결하면 오류가 발생하는 경우 include 명령 태그의 자리에 블록의 내용이 보일 것이다.  

###### 4.2.2.2. 변수 사용
<br/>

엔진엑스 SSI 모듈은 변수를 활용할 기회도 제공한다. echo 명령으로 변수를 화면에 표시할 수 있다. 다시 말해 변수의 값을 최종 HTML 소스코드에 삽입한다.  

```
<!--# echo var="변수_이름" -->
```

이 명령에는 다음 세 가지 매개변수를 사용할 수 있다.  

+ var  
표시하기 원하는 변수의 이름, 예를 들어 REMOTE&#95;ADDR 변수는 클라이언트의 IP 주소를 표시한다.

+ default  
변수가 빈값일 때 대신 표시될 문자열이다. 이 매개변수를 지정하지 않으면 (none)이 출력된다.  

+ encoding  
문자열 인코딩 방법이다. 아무런 인코딩도 하지 않을 때는 none, 공백 문자를 %20이 된다거나 하는 식으로 문자열을 URL처럼 인코딩할 때는 url, & 문자를 &로 표현하는 식의 HTML 엔티티에는 entity를 지정할 수 있다.  

set 명령을 사용해서 원하는 새 변수를 정의할 수도 있다.  

```
<!--# set var="새_변수_이름" value="변수의 값" -->
```

value 매개변수에 지정된 값은 SSI 엔진이 해석을 하기 때문에 기존 변수를 사용할 수 있다.  

```
<!--# echo var="MY_VARIABLE" -->
<!--# echo var="MY_VARIABLE" value="hello" -->
<!--# echo var="MY_VARIABLE" -->
<!--# echo var="MY_VARIABLE" value="$MY_VARIABLE there" -->
<!--# echo var="MY_VARIABLE" -->
```

###### 4.2.2.4. 조건부 구조  
<br/>

이제 설명하는 명령으로 조건에 따라 문구를 삽입하거나 다른 지시어를 사용할 수 있다. 조건부 구조의 구문은 다음과 같다.  

```
<!--# if expr="expression1" -->
[...]
<!--# elif expr="expression2" -->
[...]
<!--# else -->
[...]
<!--# endif -->
```

조건식은 다음과 같은 세 가지 다른 방식으로 형성된다.  

+ 변수 검사  
&lt;!--# if expr="$variable" --&gt;. 재작성 모듈의 if 블록과 비슷하게 변수가 빈값이 아니면 조건은 참이다.  

+ 두 문자열 비교  
&lt;!--# if expr="$variable = hello" --&gt;. 첫 문자열이 두 번째 문자열과 같으면 조건은 참이다. 첫 문자열과 두 번째 문자열이 달라야 참인 반대 조건을 만들려면 = 대신 !=를 사용하자.  

+ 정규식 패턴과 일치  
&lt;!--# if expr="$variable = /pattern/" --&gt;. 패턴은 두 / 문자 사이에 둬야 하며, 그렇지 않으면 일반 문자열로 취급된다(예를 들어 &lt;!--# if expr="$MY&#95;VARIABLE = /^/documents//" --&gt;). 문자열 비교 방식과 비슷하게 != 연산자를 부정 조건에 사용한다. 문자열 캡처는 지원하지 않는다.  

조건 블록 안에는 정규 HTML 코드나 if를 제외한 추가 SSI 지시어를 넣을 수 있다. if 블록은 중첩이 허용되지 않는다.  

###### 4.2.2.5. config 명령
<br/>

마지막은 엔진엑스가 제공하는 SSI 명령 중 가장 쓸 일이 적은 config 명령이다. 이 명령으로 두 가지 단순한 매개변수를 구성할 수 있다.  

먼저 SSI 엔진이 잘못된 태그나 유효하지 않은 식을 발견했을 때 표시할 메시지다. 기본적으로 엔진엑스는 [an error occurred while processing the directive]라고 표시한다. 다른 문구를 표시하고 싶다면 다음과 같이 입력한다.  

```
<!--# config errmsg="뭔가 끔직한 일이 일어났음" -->
```

여기에 timefmt 매개변수를 사용해서 $date&#95;local과 $date&#95;gmt 변수로 얻는 날짜의 형식도 바꿀 수 있다.  

```
<!--# config timefmt="%A, %d-%b-%Y %H:%M:%S %Z -->
```

이 매개변수에 지정하는 문자열은 strftime C 함수의 형식 문자열과 같다. 이 인자에 사용할 수 있는 형식 문자열의 더 자세한 정보는 C 언어의 strftime 함수 문서를 참조하자(http://www.opengroup.org/onlinepubs.009695399/functions/strftime.html).  

#### 4.3. 부가 모듈
<br/>

##### 4.3.1. 웹 사이트 접근 제어와 로그
<br/>

###### 4.3.1.1. 색인 모듈
<br/>

색인(Index) 모듈은 index라는 단순한 지시어를 제공한다. 이 지시어는 클라이언트 요청에 파일명이 명시돼 있지 않으면 엔진엑스가 기본으로 제공할 페이지를 정의하는데 사용할 수 있다. 하나 이상의 파일명을 지정할 수 있는데, 이 파일 중 가장 먼저 발견되는 파일이 제공된다. 지정된 파일 중 아무 파일도 찾지 못하면 엔진엑스는 자동으로 파일 색인을 생성해서 제공하거나 "403 접근 금지" 오류를 반환한다.  

선택적으로 (/page.html처럼) 파일명을 절대 경로로 넣을 수 있지만 이 지시어의 마지막 인자여야 한다.  

+ 구문: index 파일1 [파일2...] [절대&#95;경로&#95;파일]
+ 기본값: index index.html  

```
index index.php index.html index.htm;
index index.php index2.php /catchall.php;
```

index 지시어는 http, server, location 블록에서만 유효하다.  

###### 4.3.1.2. 자동 색인 모듈
<br/>

엔진엑스는 요청된 디렉터리의 색인 페이지를 제공할 수 없을 때 기본으로 "403 접근 금지" HTTP 오류 페이지를 반환하도록 동작한다. 다음 지시어는 요청된 디렉터리에 있는 파일의 목록을 자동으로 생성하게 한다.  

+ autoindex  
색인 페이지가 없을 때 디렉터리 파일 목록을 자동으로 생성하는 기능을 끄거나 켠다.  
	+ 맥락: http, server, location
	+ 구문: on이나 off  

+ autoindex&#95;exact&#95;size  
on으로 설정하면 이 지시어는 목록에서 파일 크기를 바이트로 표시한다. off로 설정하면 KB, MB, GB 같은 다른 단위가 사용된다.  
	+ 맥락: http, server, location
	+ 구문: on이나 off
	+ 기본값: on  

+ autoindex&#95;localtime  
기본적으로 이 지시어는 off로 설정해서 목록에 보이는 파일의 날짜와 시간을 GMT 시간으로 표시하게 한다. 이 값을 on으로 설정하면 시간대가 고려되지 않은 서버의 지역 시간이 사용된다.  
	+ 맥락: http, server, location
	+ 구문: on이나 off
	+ 기본값: off  

+ autoindex&#95;format  
엔진엑스는 다양한 형태로 디렉터리 색인을 제공한다.  
지시어 값을 jsonp로 설정하면 엔진엑스는 JSONP 콜백으로 callback 질의 인자 값을 삽입한다. 작성하는 스크립트가 다음 URI처럼 callback 매개변수를 사용해 호출해야 한다.  
/folder/?callback=콜백&#95;함수&#95;이름  
	+ 맥락: http, server, location
	+ 구문: autoindex&#95;format&#95;html | xml | json | jsonp;

###### 4.3.1.3. 무작위 색인 모듈
<br/>

이 모듈은 random&#95;index 지시어 하나를 활성화한다. 이 지시어는 location 블록에서 사용할 수 있으며, 지정된 디렉터리의 파일 중에서 무작위(Random)로 선택해서 색인페이지를 만들어 반환한다.  

이 모듈은 엔진엑스를 컴파일할 때 기본적으로 포함되지 않는다.  

+ 구문: on이나 off  

###### 4.3.1.4. 로그 모듈
<br/>

이 모듈은 엔진엑스가 접근 로그(access log)를 다루는 동작 방식을 제어한다. 로그 모듈은 시스템 관리자에게는 핵심 모듈이다. 웹 애플리케이션이 어떤 식으로 실행되는지 분석할 수 있기 때문이다.  

+ access&#95;log  
이 매개변수는 접근 로그 파일 경로를 정한다. 템플릿 이름을 지정해 접근 로그 항목의 형식을 정할 수도 있으며, 접근 로그를 비활성화할 수도 있다.  
몇 가지 지시어 구문 관련 유의 사항은 다음과 같다.  
	+ 맥락: http, server, location, (location 내) if, limit&#95;except
	+ 구문: access&#95;log 경로 [format [buffer=크기]] | off;
	+ access&#95;log off는 현 수준에서 접근 로그를 비활성화하는 데 쓴다.
	+ 형식 인자는 앞으로 설명할 log&#95;format 지시어로 선언한 템플릿에 대응한다.
	+ 형식 인자가 지정되지 않으면 기본 형식인 combined가 사용된다.
	+ 파일 경로에 변수를 사용할 수 있다.  

+ log&#95;format  
접근 로그의 항목에 포함될 내용을 서술하는 식으로 access&#95;log 지시어에서 활용할 템플릿을 정의한다.  
기본 템플릿의 명칭은 combined이며 다음 예에 해당한다.  
escape 매개변수는 문자 이스케이핑을 json이나 default 중 하나를 설정하는 데 사용된다. 사용하는 로그 형식에 맞춰 값을 지정하자.  
	+ 맥락: http, server, location
	+ 구문: log&#95;format 템플릿&#95;이름 [escape=default|json|none] 형식;
	```
	log_format combined '$remote_addr - $remote_usr [$time_local] '"$request" $status $body_bytes_sent '"$http_referer" "$http_user_agent"';
	# 다른 예
	log_format simple '$remote_addr $request';
	```

+ open&#95;log&#95;file&#95;cache  
로그 파일 서술자(log file descriptor)용 캐시를 구성한다. 추가 정보는 HTTP 핵심 모듈의 open&#95;file&#95;cache 지시어를 참고하자.  
인자는 open&#95;file&#95;cache나 다른 관련 지시어와 비슷하다. 차이라면 이 지시어는 접근 로그 파일에만 적용된다는 점이다.  
	+ 맥락: http, server, location
	+ 구문: open&#95;log&#95;file&#95;cache max=숫자 [inactive=시간] [min&#95;uses=숫자] [valid=시간] | off;  

로그 모듈에서 로그를 남길 때 다음과 같은 몇 가지 새로운 변수를 사용할 수 있다.  

+ $connection: 연결 번호
+ $pipe: 파이프라인으로 처리되는 요청일 경우 "p" 값이 되는 변수
+ $time&#95;local: 지역 시간(로그를 남기는 시점)
+ $msec: 1/1000초 단위로 표현된 지역 시간(로그를 남기는 시점)
+ $request&#95;time: 요청을 처리한 총 시간, 1/1000초 단위
+ $status: 응답 상태 코드
+ $bytes&#95;sent: 클라이언트에 전송된 총 바이트 수
+ $body&#95;bytes&#95;sent: 클라이언트에 전송된 응답에서 헤더를 제외한 데이터 자체의 총 바이트 수  
+ $apache&#95;bytes&#95;sent: $body&#95;bytes와 비슷하며 아파치 mod&#95;log&#95;config의 매개변수 %B에 해당
+ $request&#95;length: 요청으로 수신된 데이터 자체의 크기  

##### 4.3.2. 제한과 제약
<br/>

###### 4.3.2.1. auth&#95;basic 모듈
<br/>

auth&#95;basic 모듈은 기본 인증 기능을 제공한다. 이 모듈의 두 지시어를 사용해서 웹 사이트나 서버의 특정 위치를 사용자 이름과 비밀번호로 인증한 사용자만 사용하게 제한할 수 있다.  

```
location /admin/ {
	auth_basic "Admin control panel"; # 변수 사용 가능
	auth_basic_user_file access/password_file;
}
```

첫 번째 지시어 auth&#95;basic에는 off나 메시지 문구를 지정할 수 있다. 이 문구는 흔히 인증 시도(authentication challenge)나 인증 영역(authentication realm)이라고 부른다. 이 메시지는 클라이언트가 인증이 필요한 자원에 접근하려고 할 때 웹 브라우저의 사용자번호/비밀번호 입력 창에 표시된다.  

두 번째 지시어 auth&#95;basic&#95;user&#95;file은 비밀번호가 저장된 파일의 경로를 정하는데, 이 경로는 구성 파일의 디렉터리에 대한 상대 경로다. 비밀번호 파일은 각 줄마다 사용자&#95;이름:[{해싱&#95;방법}]비밀번호[:주석]의 구문을 따른다. 각 항목의 의미는 다음과 같다.  

+ 사용자&#95;이름: 평문으로 된 사용자 이름
+ {해싱&#95;방법}: 선택 항목이며 비밀번호 해싱 방법(scheme)을 나타낸다. 현재 다음 세 가지를 지원한다.
	+ {PLAIN}은 평문 비밀번호
	+ {SHA}은 SHA-1 해싱
	+ {SSHA}은 SHA-1 해싱에 솔트(salt)를 더해서 보안 강도를 더 높인 방법
+ 비밀번호: 비밀번호
+ 주석: 임의의 목적으로 적는 평문 설명  

해싱 방법을 지정하지 않으면 비밀번호는 crypt(3) 함수로 암호화해야 한다. 아파치 패키지에 포함된 htpasswd 명령행 유틸리티를 사용해서 crypt로 암호화할 수 있다.  

단지 htpasswd 도구 때문에 아파치를 시스템에 설치하는 열정을 보이고 싶지 않다면 온라인에서 사용할 수 있는 도구가 많으니 이를 사용할 수도 있다. 자주 쓰는 검색 엔진을 실행하고 "online htpasswd"라고 입력해보라.  

###### 4.3.2.2. 접근 모듈
<br/>

allow와 deny라는 두 가지 중요한 지시어가 접근(Access) 모듈을 통해 제공된다. 이 지시어로 특정 IP 주소나 IP 주소 범위에서 자원에 접근하도록 허가하거나 거절할 수 있다.  

두 지시어 모두 구문이 allow IP | CIDR | unix: | all과 같은 식이다. IP는 IP주소, CIDR은 IP 주소 범위(CIDR 표현 방식), unix:는 유닉스 도메인 소켓, all은 지시어가 모든 클라이언트에 적용됨을 지정한다.  

```
location {
	allow 127.0.0.1; # 로컬 IP 주소를 허용
	allow unix:; # 유닉스 도메인 소켓을 허용
	deny all; # 모든 IP 주소 차단
}
```

규칙들이 위에서 아래로 처리된다는 점에 주의하자. 첫 명령이 deny all이면 그 뒤에 따르는 모든 allow 예외 조건이 아무런 효력도 발휘하지 못한다. 그 역도 마찬가지인데, allow all로 시작하면 그 후에 적힌 모든 deny 지시어는 이미 모든 IP 주소를 허용했기 때문에 무효가 된다.  

###### 4.3.2.3. 접속 제한 모듈
<br/>

이 모듈의 처리 구조는 정규식 모듈보다 좀 더 복잡하다. 이 모듈은 특정 구역(zone)별로 서버 최대 동시 접속 수를 정하게 해준다.  

첫 단계는 limit&#95;conn&#95;zone 지시어로 구역을 정의하는 것이다.  

+ 지시어 구문: limit&#95;conn&#95;zone $변수 zone=이름:크기;
+ $변수: 한 고객을 다른 고객과 구분하는 데 사용될 변수로 주로 클라이언트의 IP 주소를 이진 형식으로 갖고 있어서 아스키 형식보다 효율이 좋은 $binary&#95;remote&#95;addr을 사용
+ 이름: 지정하는 구역에 부여할 임의의 이름
+ 크기: 세션 상태를 보관하고자 할당할 저장 공간의 최대 크기  

다음은 클라이언트 IP 주소로 구역을 정의한 예다.  

```
limit_conn_zone $binary_remote_addr zone=myzone:10m;
```

이제 구역을 정의했으니 limit&#95;conn으로 접속 수를 제한한다.  

```
limit_conn 구역_이름 접속_제한;
```

이전 예에 적용해보면 다음과 같다.  

```
location /downloads/ {
	limit_conn myzone 1;
}
```

결국 같은 $binary&#95;remote&#95;addr을 공유하는 요청은 동시에 한 접속만 허용하는 제한의 대상이 된다. 제한 수준에 도달하면 부가되는 동시 접속은 "503 서비스 사용 불가" HTTP 응답을 받게 된다. 이 응답 코드는 limit&#95;conn&#95;status 지시어에 지정한 다른 코드로 대체할 수 있다. 설정한 한계에 제한된 클라이언트 응답을 로그에 남기고 싶다면 limit&#95;conn&#95;log&#95;level 지시어로 로그 수준(info|notice|warn|error)을 지정한다.  

###### 4.3.2.4. 요청 제한 모듈
<br/>

비슷한 방식으로 요청 제한 모듈은 정의된 구역에 요청 수를 제한할 수 있게 해준다.  

구역은 limit&#95;req&#95;zone 지시어로 정의한다. 이 지시어의 구문은 limit&#95;conn&#95;zone 지시어와 다르다.  

```
limit_req_zone $변수 zone=이름:최대_메모리_크기 rate=비율;
```

이 지시어의 매개변수는 뒤에 붙은 초당 또는 분당 요청 횟수를 나타내는 rate를 제외하고는 동일하다. 이 매개변수는 특정 구역에서 클라이언트에 적용될 시간당 요청 횟수를 정한다. 구역을 위치에 적용하는 데에는 limit&#95;req 지시어를 쓴다.  

```
limit_req zone=이름 burst=허용_초과_요청 횟수 [nodelay|delay=숫자];
```

burst 매개변수는 최대 허용 초과 요청 횟수를 정한다. 구역에 정의된 시간당 요청 횟수 한도를 초과해서 요청이 수신되면 한도를 충족시키고자 요청을 지연하게 된다. 어느 정도까지는 burst 매개변수의 값만큼만 동시에 받아들여질 것이다. burst 값을 넘는 요청은 엔진엑스가 "503 서비스 사용 불가" 오류 응답을 반환한다. 이 응답 코드는 limit&#95;req&#95;status 지시어에 다른 코드를 지정해서 변경할 수 있다.  

지연을 원하지 않으면 nodelay 옵션을 지정하고, 일정 수 이후로 지연시키고 싶으면 한도를 delay에 지정한다.  

```
limit_req_zone $binary_remote_addr zone=myzone:10m rate=2r/s;
[...]
location /downloads/ {
	limit_req zone=myzone burst=10;
	limit_req_status 404; # 한도를 초과하면 404 오류를 반환
}
```

지정한 한도의 영향을 받은 클라이언트 요청을 로그에 남기고 싶다면 limit&#95;req&#95;log&#95;level 지시어를 사용해서 로그 수준(info|notice|warn|error)을 지정한다.  

###### 4.3.2.5. auth&#95;request 모듈
<br/>

auth&#95;request 모듈은 최근 엔진엑스에 도입됐고 부가 요청의 결과에 따라 자원 접근을 허락할지 거부할지 결정할 수 있게 해준다. 엔진엑스는 auth&#95;request 지시어에 지정한 URI를 호출해서 이런 부가 요청이 2XX 응답 코드(정확히는 HTTP/200 OK)를 반환하면 접근을 허용한다. 부가 요청이 401이나 403 상태 코드를 반환하면 접근이 거부되고 엔진엑스는 해당 응답 코드를 클라이언트에 전달한다. 뒷단에서 다른 응답 코드가 반환되면 엔진엑스는 이를 오류로 간주하고 자원 접근을 불허한다.  

```
location /downloads/ {
	# 스크립트가 200 상태 코드를 반환하면
	# 자료를 다운로드하도록 허용한다.
	auth_request /authorization.php;
}
```

이 모듈은 auth&#95;request&#95;set이라는 두 번째 지시어를 제공함으로써 부가 요청이 수행된 후에 변수 값을 설정할 수 있게 해준다. 부가 요청에서 기인하는 $upstream&#95;http&#95;server나 다른 서버 응답의 HTTP 헤더 값을 $upstream&#95;http&#95;&#42; 형태의 변수로 삽입할 수 있다.  

##### 4.3.3. 콘텐트와 인코딩
<br/>

다음의 각 모듈은 클라이언트에 전송되는 콘텐트에 영향을 주는 기능을 제공한다. 응답이 인코딩되는 방식을 수정하거나, 헤더에 영향을 미치거나, 응답을 완전히 새로 생성하기도 한다.  

###### 4.3.3.1. 빈 GIF
<br/>

이 모듈의 목적은 1 × 1 크기의 투명한 GIF 이미지를 메모리에서 전송하는 지시어를 제공하는 것이다. 이런 파일은 웹 디자이너가 웹 사이트를 조작할 때 종종 사용한다. 이 지시어로 저장 공간에서 실제 GIF 파일을 읽어 처리하는 대신 메모리에서 직접 빈 GIF를 얻을 수 있다.  

이 기능을 활용하려면 empty&#95;gif 지시어를 원하는 location 블록에 삽입하기만 하면 된다.  

```
location = /empty.gif {
	empty_gif;
}
```

###### 4.3.3.2. FLV와 MP4
<br/>

FLV와 MP4는 별도 모듈로 플래시(FLV)나 MP4 비디오 파일을 제공할 때 유용하게 쓰일 수 있는 단순한 기능을 활성화한다. 이 모듈은 요청의 start라는 특별한 인자를 분석하는데, 이 인자는 클라이언트가 다운로드하거나 스트리밍하기 원하는 부분의 시작 위치를 나타낸다. 따라서 이 비디오 파일은 video.flv?start=XXX 같은 URI로 접근해야 한다. 이 매개변수는 JWPlayer 같은 유명한 비디오 재생기가 자동으로 보내준다.  

이 기능을 활용하려면 flv나 mp4 지시어를 선택된 location 블록에 삽입하기만 하면 된다.  

```
location ~* \.flv {
	flv;
}

location ~* \.mp4 {
	mp4;
}
```

엔진엑스가 비디오 파일에서 요청된 위치를 찾지 못한 경우에는 주의해야 한다. "500 내부 서버" 오류 HTTP 응답이 요청 결과로 반환될 것이다. JWPlayer는 종종 이 오류를 잘못 해석해서 비디오를 찾을 수 없다는 문구를 표시한다.  

###### 4.3.3.3. HTTP 헤더
<br/>

이 모듈이 소개하는 두 지시어는 클라이언트에 보내는 응답의 헤더에 영향을 미친다.  

먼저 add&#95;header 이름 값 [always]는 응답 헤더에 새로운 줄을 추가할 수 있게 해준다. 추가되는 헤더의 구문은 이름: 값 같은 식이다. 이 헤더는 응답 코드가 200, 201, 204, 301, 302, 304일 때만 추가된다. 값 인자에는 변수를 사용할 수 있다. 지시어 값의 끝에 always를 명기하면 새 헤더가 응답 코드와 상관없이 언제나 추가된다.  

add&#95;trailer는 add&#95;header와 거의 같지만 헤더가 아닌 응답 뒤에 필드를 더한다. 여기에 더해 expires 지시어는 클라이언트에 전송될 Expires와 Cache-Control HTTP 헤더의 값을 제어할 수 있게 해준다. 이 지시어도 나열한 코드의 응답에 영향을 미친다. 값은 다음 중 하나만 받는다.  

+ off: 두 헤더 모두 그대로 둔다.
+ 시간 값: 파일의 만기일이 현재 시간에 지정한 시간을 더해서 지정된다. 예를 들어 expires 24h는 만기일을 지금 시간에 24시간을 더한 시간으로 반환할 것이다.
+ epoch: 파일의 만기일이 1970년 1월 1일로 설정된다. Cache-Control 헤더는 no-cache로 된다.
+ max: 파일의 만기일이 2037년 12월 31일이 된다. Cache-Control 헤더는 10년으로 된다.  

###### 4.3.3.4. 첨가 모듈
<br/>

이 첨가(Addition) 모듈은 단순한 지시어를 통해 HTTP 응답의 본문 앞이나 뒤에 내용을 추가할 수 있게 해준다.  

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

두 가지 주요 지시어는 다음과 같다.  

```
add_before_body file_uri;
add_after_body file_uri;
```

앞에서 언급했듯이 엔진엑스는 지정된 URI의 내용을 얻고자 부가 요청을 발생시킨다. location 블록의 패턴이 충분히 구체적이지 않을 경우 내용이 추가될 파일의 유형을 정의할 수도 있다. 기본 파일 유형은 text/html이다.  

```
addition_types mime_type1 [mime_type2...];
addition_types *;
```

###### 4.3.3.5. 대체 모듈
<br/>

앞의 첨가 모듈과 같은 선상에서 이 대체(Substitution) 모듈은 응답 본문에서 직접 문자를 찾아 바꿀 수 있게 해준다. 한 블록에서 여러 번 사용할 수 있다.  

```
sub_filter 찾을_문자 바꿀_문자;
```

찾을 문자에 변수를 쓸 수 있다.  

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

다양한 상황에 대응하게 다음과 같은 두 가지 추가 지시어가 제공된다.  

+ sub&#95;filter&#95;once(on이나 off, 기본값 on): 문자를 한 번만 교체하고 첫 번째 문자를 발견한 후에는 더 이상 찾지 않는다.  

+ sub&#95;filter&#95;types(기본값 text/html): 문자를 교체할 대상이 될 MIME 타입을 추가한다.  

###### 4.3.3.6. GZip 필터
<br/>

이 모듈은 클라이언트에 전송하기 전에 Gzip 알고리즘으로 응답의 본문을 압축할 수 있게 한다. GZip 압축을 하려면 gzip 지시어를 http, server, location 블록에 사용하면 된다. if 수준에도 사용할 수는 있지만 권장하지 않는다. 다음은 추가로 이 필터 옵션을 구성하는 데 도움이 되는 지시어들이다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				gzip&#95;buffers<br/>
				맥락: http, server, location
			</td>
			<td>
				압축된 응답을 저장하는 데 사용될 버퍼의 개수와 크기를 정의한다.<br/>
				구문: gzip&#95;buffers 버퍼 수 크기;
				기본값: gzip&#95;buffers 4 4k(OS에 따라 8k)
			</td>
		</tr>
		<tr>
			<td>
				gzip&#95;comp&#95;level<br/>
				맥락: http, server, location
			</td>
			<td>
				GZip 알고리즘의 압축 정도를 정한다. 압축 정도로 지정할 값의 범위는 1(압축률은 낮지만 빠름)에서 9(압축이 많이 되지만 느림)까지다.<br/>
				구문: gzip&#95;comp&#95;level 숫자;
				기본값: 1
			</td>
		</tr>
		<tr>
			<td>
				gzip&#95;disable<br/>
				맥락: http, server, location
			</td>
			<td>
				요청의 User-Agent HTTP 헤더가 지정된 정규식과 일치할 경우 GZip 압축을 비활성화한다.<br/>
				구문: gzip&#95;disable 정규식<br/>
				기본값: 없음
			</td>
		</tr>
		<tr>
			<td>
				gzip&#95;http&#95;version<br/>
				맥락: http, server, location
			</td>
			<td>
				특정 프로토콜 버전(1.0 또는 1.1)에서 Gzip 압축을 활성화한다.<br/>
				구문: gzip&#95;http&#95;version HTTP&#95;버전;<br/>
				기본값: 1.1
			</td>
		</tr>
		<tr>
			<td>
				gzip&#95;min&#95;length<br/>
				맥락: http, server, location
			</td>
			<td>
				응답 본문의 길이가 지정된 값보다 작으면 압축을 하지 않는다.<br/>
				구문: gzip&#95;min&#95;length 크기;<br/>
				기본값: 0
			</td>
		</tr>
		<tr>
			<td>
				gzip&#95;proxied<br/>
				맥락: http, server, location
			</td>
			<td>
				프록시에서 받은 응답의 본문을 Gzip으로 압축할지 여부를 결정한다. 역프록시(reverse proxy) 메커니즘은 나중에 설명한다. 이 지시어에 다음 매개변수를 사용할 수 있으며, 일부는 조합할 수도 있다.<br/>
				+ off/any: 모든 요청을 압축할지 여부를 결정<br/>
				+ expired: Expires 헤더가 캐싱을 하지 않게 돼 있다면 압축을 활성화<br/>
				+ no-cache/no-store/private: Cache-Control 헤더가 no-cache나 no-store, 또는 private으로 설정했으면 압축을 활성화<br/>
				+ no&#95;last&#95;modified: Last-Modified 헤더가 없는 경우에 압축을 활성화<br/>
				+ no&#95;etag: ETag 헤더가 없을 경우에 압축을 활성화<br/>
				+ auth: Authorization 헤더가 있으면 압축을 활성화<br/>
			</td>
		</tr>
		<tr>
			<td>
				gzip&#95;types<br/>
				맥락: http, server, location
			</td>
			<td>
				기본 MIME 타입인 text/html 외의 다른 타입에 압축을 활성화한다.<br/>
				구문:<br/>
				gzip&#95;types&#95;mime&#95;타입1 [mime&#95;타입2...];<br/>
				gzip&#95;types &#42;;<br/>
				기본값: text/html(비활성화할 수 없음)
			</td>
		</tr>
		<tr>
			<td>
				gzip&#95;vary<br/>
				맥락: http, server, location
			</td>
			<td>
				Vary: Accept-Encoding HTTP 헤더를 응답에 추가한다.<br/>
				구문: on이나 off<br/>
				기본값: off
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

###### 4.3.3.7. 정적 Gzip
<br/>

이 모듈은 Gzip 필터 메커니즘에 간단한 기능을 추가한다. gzip&#95;static 지시어가 활성화되면 엔진엑스는 요청된 문서를 제공하기 전에 자동으로 그에 상응하는 .gz 파일을 찾는다. 이를 통해 엔진엑스는 요청이 올 때마다 문서를 압축하는 대신 미리 압축된 파일을 전송할 수 있게 된다. always 인자가 지정해서 엔진엑스에게 클라이언트가 gzip 인코딩을 받을 수 있든 말든 상관없이 gzip 버전을 제공하게 할 수 있다.  

이 모듈은 엔진엑스가 컴파일할 때 자동으로 포함되지 않는다.  

클라이언트가 /디렉터리/페이지.html를 요청하면 엔진엑스는 /디렉터리/페이지.html.gz 파일이 있는지 확인한다. .gz 파일을 찾으면 클라이언트에 전송한다. 엔진엑스는 요청된 파일을 전송한 후라도 직접 .gz 파일을 생성하지는 않는다.  

###### 4.3.3.8. gzip 압축 해제 필터
<br/>

Gunzip 필터 모듈을 사용하면 뒷단의 서버에서 gzip으로 압축된 응답을 전송받아도 클라이언트에게는 압축되지 않은 그대로 데이터를 제공할 수 있다. 예를 들어 (마이크로소프트 인터넷 익스플로러 6 같이) 클라이언트 브라우저가 gzip으로 압축된 파일을 처리하지 못할 때는 이 모듈을 사용할 location 블록에 gunzip on;을 삽입하기만 하면 된다. gunzip&#95;buffers 버퍼 수 크기로 버퍼의 수와 크기를 지정할 수도 있다. 여기서 버퍼 수는 할당할 버퍼의 개수이고, 크기 인자는 할당될 각 버퍼의 크기다.  

###### 4.3.3.9. 문자 세트 필터
<br/>

문자 세트 필터 모듈로는 응답 본문의 문자 세트를 더 정확히 제어할 수 있다. (Content-Type: text/html; charset=utf-8처럼) Content-Type HTTP 헤더의 charset 인자 값을 지정할 수 있을 뿐 아니라 엔진엑스가 자동으로 특정 인코딩 방법으로 다시 인코딩을 할 수도 있다.  

<table>
	<thead>
		<tr>
			<td>
				charset<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 응답의 Content-Type 헤더에 특정 인코딩을 추가한다. 지정된 인코딩과 source&#95;charset의 값이 다를 경우 엔진엑스는 문서를 다시 인코딩한다.<br/>
				구문: charset 인코딩 | off;<br/>
				기본값: off<br/>
				예: charset utf-8;
			</td>
		</tr>
		<tr>
			<td>
				source&#95;charset<br/>
				맥락: http, server, location, if
			</td>
			<td>
				응답의 초기 인코딩을 정한다. 이 값이 charset 지시어에 지정된 값과 다르다면 엔진엑스는 문서를 다시 인코딩한다.<br/>
				구문: source&#95;charset encoding;
			</td>
		</tr>
		<tr>
			<td>
				override&#95;charset<br/>
				맥락: http, server, location, if
			</td>
			<td>
				엔진엑스가 프록시나 FastCGI 게이트웨이에서 응답을 받았을 때 이 지시어는 문자 인코딩을 확인해서 재지정해야 할지 여부를 정한다.<br/>
				구문: on이나 off<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				charset&#95;types<br/>
				맥락: http, server, location
			</td>
			<td>
				재인코딩 대상이 되는 문서의 MIME 타입을 정한다.<br/>
				구문: charset&#95;types mime&#95;타입1 [mime&#95;타입2...]; charset&#95;types &#42;;9<br/>
				기본값: text/html, text/xml, text/plain, text/vnd.wap.wml, application/x-javascript, application/rss+xml
			</td>
		</tr>
		<tr>
			<td>
				charset&#95;types<br/>
				맥락: http, server, location
			</td>
			<td>
				재인코딩 대상이 되는 문서의 MIME 타입을 정한다.<br/>
				구문: charset&#95;types mime&#95;타입1 [mime&#95;타입2...]; charset&#95;types &#42; ;9<br/>
				기본값: text/html, text/xml, text/plain, text/vnd.wap.xml, application/x-javascript, application/rss+xml
			</td>
		</tr>
		<tr>
			<td>
				charset&#95;map<br/>
				맥락: http
			</td>
			<td>
				문자 재인코딩 테이블을 정의할 수 있다. 테이블의 각 행은 교환될 두 16진수 코드로 구성돼 있다. 기본 엔진엑스 구성 폴더를 보면 koi8-r 문자 세트용 재인코딩 테이블(koi-win과 koi-utf)을 볼 수 있다.<br/>
				구문: charset&#95;map 원천&#95;인코딩&#95;대상&#95;인코딩 { ... }
			</td>
		</tr>
	</thead>
</table>
<br/><br/>

###### 4.3.3.10. 맴캐시디
<br/>

맴캐시디(Memcached)는 소켓으로 접속할 수 있는 데몬 애플리케이션이다. 이 애플리케이션의 주목적은 이름에서 알 수 있듯 효과적인 분산 키-값 메모리 캐싱 시스템을 제공하는 것이다. 엔진엑스 멤캐시디 모듈은 멤캐시디 데몬에 접속하도록 구성할 수 있는 지시어를 제공한다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				memcached&#95;pass<br/>
				맥락: location, if
			</td>
			<td>
				멤캐시디 데몬의 호스트 이름과 포트를 정의한다.<br/>
				구문: memcached&#95;pass 호스트&#95;이름:포트;<br/>
				예: memcached&#95;pass localhost:11211;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;bind<br/>
				맥락: http, server, location
			</td>
			<td>
				엔진엑스가 멤캐시디 서버에 접속할 때 강제로 특정 로컬 IP 주소를 사용하게 만든다. 서버에 여러 네트워크 인터페이스 카드가 서로 다른 네트워크에 연결돼 있다면 편리하게 쓸 수 있다.<br/>
				off 값으로 지정하면 상위 수준에서 상속된 구성을 취소한다. transparent 옵션을 지정하면 기기의 IP가 아닌, 예를 들어 클라이언트의 IP가 아닌, 예를 들어 클라이언트의 IP로 지정할 수 있다.<br/>
				구문: memcached&#95;bind IP주소 [transparent]|off;<br/>
				예: memcached&#95;bind 192.168.1.2;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;connect&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				1/1000초 단위로 연결 시한을 정한다.<br/>
				기본값: 60,000<br/>
				예: memcached&#95;connect&#95;timeout 5000;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;send&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				1/1000초 단위로 데이터 전송 작업 시한을 정한다.<br/>
				기본값: 60,000<br/>
				예: memcached&#95;send&#95;timeout 5,000;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;read&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				1/1000초 단위로 데이터 수신 작업 시한을 정한다.<br/>
				기본값: 60,000<br/>
				예: memcached&#95;read&#95;timeout 5,000;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;socket&#95;keepalive<br/>
				맥락: http, server, location
			</td>
			<td>
				맴캐시디 연결이 유지되도록 TCP 소켓을 구성한다. 기본적으로는 운영체제의 소켓 설정이 그대로 반영된다. 이 지시어의 값이 on이면 SO&#95;KEEPALIVE 소켓 옵션이 사용하는 소켓에 활성화된다.<br/>
				구문: memcached&#95;socket&#95;keepalive on | off;<br/>
				기본값: off<br/>
				예: memcached&#95;socket&#95;keepalive on;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;buffer&#95;size<br/>
				맥락: http, server, location
			</td>
			<td>
				데이터를 송수신하는 데 사용할 버퍼의 크기를 바이트 단위로 정한다.<br/>
				기본값: 페이지 크기<br/>
				예: memcached&#95;buffer&#95;size 8k;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;next&#95;upstream<br/>
				맥락: http, server, location
			</td>
			<td>
				memcached&#95;pass 지시어가 업스트림 블록에 연결돼 있다면(업스트림 모듈 항목 참조) 이 지시어는 다음 업스트림 서버로 무시하고 넘어갈 조건을 정의한다.<br/>
				구문: memcached&#95;next&#95;upstream error | timeout | invalid&#95;response | not&#95;found | off<br/>
				기본값: error timeout<br/>
				예: memcached&#95;next&#95;upstream off;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;next&#95;upstream&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				요청을 다음 서버로 넘길 시간을 제약한다. 0 값은 이 제약을 없앤다.<br/>
				구문: memcached&#95;next&#95;upstream&#95;timeout 시간;<br/>
				기본값: 0<br/>
				예: memcached&#95;next&#95;upstream&#95;timeout 3s;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;next&#95;upstream&#95;tries<br/>
				맥락: http, server, location
			</td>
			<td>
				요청을 다음 서버로 넘기는 최대 시도 횟수를 정한다. 0 값은 이 제약을 없앤다.<br/>
				구문: memcached&#95;next&#95;upstream&#95;tries 숫자;<br/>
				기본값: 0<br/>
				예: memcached&#95;next&#95;upstream&#95;tries 3;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;gzip&#95;flag<br/>
				맥락: http, server, location
			</td>
			<td>
				맴캐시디 서버 응답에 특정 플래그가 있는지 확인한다. 해당 플래그가 있다면 엔진엑스는 Content-encoding 헤더를 gzip으로 설정해서 gzip으로 압축한 내용임을 알 수 있게 한다.<br/>
				구문: memcached&#95;gzip&#95;flag 플래그&#95;숫자;<br/>
				기본값: (없음)<br/>
				예: memcached&#95;gzip&#95;flag 1;
			</td>
		</tr>
		<tr>
			<td>
				memcached&#95;force&#95;ranges<br/>
				맥락: http, server, location
			</td>
			<td>
				멤캐시디 서버 캐시 여부와 상관 응답의 바이트 범위(byte-range)의 지원을 활성화한다.<br/>
				구문: memcached&#95;force&#95;ranges on | off;<br/>
				기본값: off<br/>
				예: memcached&#95;force&#95;ranges off;
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

덧붙이자면 캐시에 데이터를 넣거나 뺄 때 사용할 요소의 키를 정하는 $memcached&#95;key 변수를 정의해야 한다. 예를 들어 set $memcached&#95;key $uri나 set $memcached&#95;key $uri?$args 같은 구성을 사용할 수 있다.  

엔진엑스 멤캐시디 모듈은 캐시의 데이터를 얻어올 수 있을 뿐임을 알아두자. 이 모듈은 요청의 결과를 저장하지는 못한다. 데이터를 캐시에 저장하는 일은 서버의 스크립트에서 처리해야 한다. 서버 스크립트와 엔진엑스 구성이 정확히 같은 키 명명 규칙을 사용하도록 확인해야 한다. 예를 들어 요청이 들어오면 캐시에서 데이터를 먼저 얻으려고 시도하고, 요청된 URI가 캐시에 없을 때 요청을 프록시에 전달하는 식으로 멤캐시디를 사용할 수 있다.  

```
server {
	server_name example.com;
	[...]
	location / {
		set $memcached_key $uri;
		memcached_pass 127.0.0.1:11211;
		error_page 404 @notcached;
	}
	location @notcached {
		internal;
		# 파일을 찾지 못하면 프록시에 요청을 전달한다.
		proxy_pass 127.0.0.1:8080;
	}
}
```

###### 4.3.3.11. 이미지 필터
<br/>

이 모듈은 GD 그래픽 라이브러리(흔히 gdlib라고 부름)를 통해 이미지 처리 기능을 제공한다. Webp를 지원하게 컴파일된 gdlib를 사용해야 Webp 형식으로 변환할 수 있다.  

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

다음 표의 지시어들은 location ~&#42; \.(png|jpg|webp|gif)$ { ... } 같이 이미지 파일만 고르는 location 블록에서 사용해야 한다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				image&#95;filter<br/>
				맥락: location
			</td>
			<td>
				이미지를 클라이언트에 전송하기 전에 변형을 가한다. 가용한 다섯 가지 옵션이 있다.<br/>
				+ test: 요청된 문서가 이미지 파일인지 확인하고, 확인하지 못했을 경우 "415 지원하지 않는 미디어" HTTP 요류를 반환
				+ size: 이미지의 크기나 종류 같은 정보를 나타내는 단순한 JSON 응답을 구성함(예: {"img": {"width":50, "height":50, "type":"png"}}), 파일이 유효하지 않다면 {}만 반환됨
				+ resize 폭 높이: 지정한 치수대로 이미지의 크기를 변형
				+ crop 폭 높이: 지정한 차수대로 이미지의 일부를 잘라냄
				+ rotate 90 | 180 | 270: 지정한 각도에 따라 이미지를 회전  
				예: image&#95;filter resize 200 100;
			</td>
		</tr>
		<tr>
			<td>
				image&#95;filter&#95;buffer<br/>
				맥락: http, server, location
			</td>
			<td>
				처리할 최대 이미지 파일 크기를 정한다.<br/>
				기본값: image&#95;filter&#95;buffer 1m;
			</td>
		</tr>
		<tr>
			<td>
				image&#95;filter&#95;jpeg&#95;quality<br/>
				맥락: http, server, location
			</td>
			<td>
				생성될 JPEG 이미지의 품질을 정한다. 1에서 100까지의 수치를 사용할 수 있다. 권장하는 최대 수치는 95다. 변수를 사용할 수 있다.<br/>
				기본값: image&#95;filter&#95;jpeg&#95;quality 75;
			</td>
		</tr>
		<tr>
			<td>
				image&#95;filter&#95;webp&#95;quality<br/>
				맥락: http, server, location
			</td>
			<td>
				생성될 Webp 이미지의 품질을 정한다. 1에서 100까지의 수치를 사용할 수 있다. 변수를 사용할 수 있다.<br/>
				기본값: image&#95;filter&#95;jpeg&#95;quality 75;
			</td>
		</tr>
		<tr>
			<td>
				image&#95;filter&#95;transparency<br/>
				맥락: http, server, location
			</td>
			<td>
				기본적으로 PNG와 GIF 이미지의 투명도는 이미지 필터 모듈을 사용해서 처리되는 동안 유지된다. 이 지시어를 off로 설정하면 기존 투명도는 무시되지만 이미지의 품질은 향상될 것이다.<br/>
				구문: on이나 off<br/>
				기본값: on
			</td>
		</tr>
		<tr>
			<td>
				image&#95;filter&#95;sharpen<br/>
				맥락: http, server, location
			</td>
			<td>
				지정한 백분율에 따라 이미지를 더 선명하게 만든다.<br/>
				구문: 숫자(100 이상)<br/>
				기본값: 0
			</td>
		</tr>
		<tr>
			<td>
				image&#95;filter&#95;interlace<br/>
				맥락: http, server, location
			</td>
			<td>
				생성되는 이미지를 비월 주사 방식(interlacing)으로 이미지가 생성되게 한다. 이미지가 JPG 파일이면 이미지는 점진적 JPG(progressive JPEG) 형식으로 생성된다.<br/>
				구문: on이나 off<br/>
				기본값: off
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

JPG 이미지일 경우 메타데이터가 전체 파일 크기의 5% 이상을 차지하면 엔진엑스는 자동으로 (EXIF 같은) 메타데이터를 제거한다는 점을 알아두자.  

###### 4.3.3.12. XSLT
<br/>

엔진엑스 XSLT 모듈은 XML 파일이나 뒷단 서버(프록시, FastCGI 등)에서 받은 응답을 클라이언트에 제공하기 전에 XSLT로 변환하게 해준다.  

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				xml&#95;entities<br/>
				맥락: http, server, location
			</td>
			<td>
				요소 정의가 담긴 DTD 파일을 지정한다.<br/>
				구문: 파일 경로<br/>
				예: xml&#95;entities xml/entities.dtd;
			</td>
		</tr>
		<tr>
			<td>
				xslt&#95;stylesheet<br/>
				맥락: location
			</td>
			<td>
				XSLT 템플릿 파일 경로를 매개변수와 함께 지정한다. 변수를 매개변수에 사용할 수 있다.<br/>
				구문: xslt&#95;stylesheet template [param1] [param2...];
				예: xslt&#95;stylesheet xml/sch.xslt param=value;
			</td>
		</tr>
		<tr>
			<td>
				xslt&#95;types<br/>
				맥락: http, server, location
			</td>
			<td>
				text/xml 외에 XSLT 변환을 추가로 적용할 MIME 타입을 정의한다.<br/>
				구문: MIME 타입<br/>
				예:<br/>
				xslt&#95;types text/xml text/plain;<br/>
				xslt&#95;types &#42;;
			</td>
		</tr>
		<tr>
			<td>
				xslt&#95;paramxslt&#95;string&#95;param<br/>
				맥락: http, server, location
			</td>
			<td>
				이 두 지시어로 XSLT 스타일시트에 사용할 매개변수를 정할 수 있다. 둘의 차이는 지정된 값을 해석하는 방법에 있다. xslt&#95;param은 값에 포함된 XPath 식을 처리하는 데 쓰이고 xslt&#95;string&#95;param은 평문 문자열에 쓰인다.<br/>
				구문: xslt&#95;param 키 값;
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 4.3.4. 방문자 정보
<br/>

###### 4.3.4.1. 브라우저 모듈
<br/>

브라우저 모듈은 클라이언트 요청의 User-Agent HTTP 헤더를 분색해서 나중에 엔진엑스 구성에 사용할 수 있는 변수의 값을 얻는다. 다음 세 변수가 만들어진다.  

+ $modern&#95;browser  
클라이언트 브라우저가 최신 웹 브라우저인 것으로 밝혀지면 이 변수에는 modern&#95;browser&#95;value 지시어로 정한 값이 담긴다.  

+ $ancient&#95;browser  
브라우저가 구식 웹 브라우저인 것으로 밝혀지면 이 변수에는 ancient&#95;browser&#95;value 지시어로 정한 값이 담긴다.  

+ $msie  
클라이언트가 마이크로소프트 IE 브라우저면 이 변수에는 1이 담긴다.  

웹 브라우저를 식별해서 최신뿐 아니라 구식 버전까지 지원하도록 엔진엑스를 도우려면 ancient&#95;browser와 modern&#95;browser 지시어를 여러 번 사용해야 한다.  

```
modern_browser opera 10.0;
```

이 예에서 User-Agent HTTP 헤더에 Opera 10.0 문자가 포함돼 있으면 클라이언트는 최신으로 취급된다.  

###### 4.3.4.2. 맵 모듈
<br/>

브라우저 모듈처럼 맵(Map) 모듈은 한 변수의 값에 따라 그에 대응하는 값을 새 변수에 할당한다.  

```
map $uri $variable {
	/page.html 0;
	/contact.html 1;
	/index.html 2;
	default 0;
}
rewrite ^ /index.php?page=$variable;
```

map 지시어는 http 블록 안에만 쓸 수 있다는 점을 주의하자. 이 예에 따르면 $variable은 세 가지 다른 값을 갖는다. $url 값이 /page.html이면 $variable은 0이 된다. $uri이 /contact.html이면 $variable은 1이다. $uri이 /index.html이면 $variable은 2다. 그 외의 경우(default), $variable은 0이 된다. 마지막 명령은 URL은 새로 생성된 변수의 값에 따라 재작성한다. default 외에도 이 지시어에서 쓸 수 있는 특별한 키워드로 hostnames와 volatile이 있다. hostnames는 &#42;.domain.com과 같이 호스트 이름에 와일드카드를 사용해서 일치하는지 따진다. volatile은 변수가 캐시되지 않도록 한다.  

엔진엑스가 메모리에서 이런 메커니즘을 관리하는 방법을 조작할 수 있는 다음과 같은 지시어 두 가지가 더 있다.  

+ map&#95;hash&#95;max&#95;size: 맵을 보관하는 해시 테이블의 최대 크기
+ map&#95;hash&#95;bucket&#95;size: 맵에 쓸 수 있는 각 항목의 최대 크기  

~(대소문자 구분) 또는 ~&#42;(대소문자 구분하지 않음)으로 시작하는 패턴 형태로 정규식도 사용할 수 있다.  

```
map $http_referer $ref {
	~google "Google";
	~* yahoo "Yahoo";
	\~bing "Bing"; # ~ 문자 앞에 \이 있어 정규식이 아님
	default $http_referer; # 변수 사용
}
```

###### 4.3.4.3. 지리 정보 모듈
<br/>

이 모듈의 목적은 클라이언트 데이터(이 경우엔 IP 주소)에 기반을 두고 변수 값을 결정한다는 면에서 map 지시어와 매우 비슷한 기능을 제공하는 것이다.  

```
geo $variable {
	default unknown;
	127.0.0.1 local;
	123.12.3.0/24 uk;
	92.43.0.0/16 fr;
}
```

적절한 지리상 위치를 탐지하려면 GeoIP 모듈을 사용해야 한다. 이 블록에는 이 모듈 고유의 여러 가지 지시어를 사용할 수 있다.  

+ delete  
특정 하위 네트워크를 일치 여부 판단에서 제외한다.

+ default  
사용자의 IP 주소가 지정된 IP 범위 중 어느 것에도 부합되지 않을 때 $variable에 부여할 기본값이다.

+ include  
외부 파일의 내용을 포함한다.

+ proxy  
신뢰할 수 있는 주소의 서브넷을 정의한다. IP 주소가 신뢰할 수 있으면 소켓의 IP 주소 대신 X-Forwarded-For 헤더를 IP 주소로 사용한다.

+ proxy&#95;recursive  
이 지시어가 사용되면 클라이언트의 IP 주소가 신뢰할 수 없을 때 X-Forwarded-For 헤더의 값을 찾는다.

+ ranges  
geo 블록의 첫 줄에 이 지시어를 넣으면 CIDR 마스크 대신 IP 주소 범위를 지정할 수 있게 된다. 127.0.0.1-127.0.0.255 LOCAL; 같은 구문을 허용한다.  

###### 4.3.4.4. GeoIP 모듈
<br/>

이름에서 알 수 있는 것처럼 앞의 모듈과 비슷한 면도 있다. 이 모듈은 옵션으로 맥스마인드(http://www.maxmind.com)의 GeoIP 이진 데이터베이스를 사용해서 방문객에 대한 정확한 지리 정보를 제공한다. 맥스마인드 웹 사이트에서 이 데이터베이스를 다운로드한 후 엔진엑스 디렉터리에 저장해야 한다.  

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

이제 해야 할 일은 다음 지시어 중 하나를 데이터베이스 경로를 지정하는 것이다.  

```
geoip_country country.dat; # 국가 정보 DB
geoip_city city.dat;	   # 도시 정보 DB
geoip_org geoiporg.dat;    # ISP/기관 DB
```

첫 지시어는 몇 가지 변수를 사용할 수 있게 한다. $geoip&#95;country&#95;code는 두 문자 국가 코드, $geoip&#95;country&#95;code3는 세 문자 국가 코드, $geoip&#95;country&#95;name은 전체 국가명이다. 두 번째 지시어는 같은 변수를 포함하면서 $geoip&#95;region, $geoip&#95;city, $geoip&#95;postal&#95;code, $geoip&#95;city&#95;continent&#95;code, $geoip&#95;latitude, $geoip&#95;longitude, $geoip&#95;dma&#95;code, $geoip&#95;area&#95;code, $geoip&#95;region&#95;name 등 추가 정보를 제공한다. 세 번째 지시어는 특정 IP를 소유한 조직이나 ISP의 정보를 갖는 $geoip&#95;org 변수를 제공한다.  

변수가 UTF-8로 인코딩돼야 한다면 utf8 키워드를 geoip&#95; 지시어의 끝에 추가하기만 하면 된다.  

###### 4.3.4.5. UserID 필터
<br/>

이 모듈은 쿠키를 생성해서 클라이언트에 ID를 할당한다. 이 ID는 엔진엑스 구성에서 $uid&#95;got와 $uid&#95;set 변수를 통해 얻을 수 있다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				userid<br/>
				맥락: http, server, location
			</td>
			<td>
				쿠키를 발급하고 로그에 남기는 기능을 켜거나 끈다. 이 지시어에는 네 가지 값을 사용할 수 있다.<br/>
				+ on: v2 쿠키를 발급하고 로그에 남김
				+ v1: v1 쿠키를 발급하고 로그에 남김
				+ log: 쿠키 데이터를 보내지 않고 수신되는 쿠키만 로그에 남김
				+ off: 쿠키 데이터를 보내지 않음<br/>
				기본값: userid off;
			</td>
		</tr>
		<tr>
			<td>
				userid&#95;service<br/>
				맥락: http, server, location
			</td>
			<td>
				쿠키를 발급하는 서버의 IP 주소를 정한다.<br/>
				구문: userid&#95;service ip;
				기본값: 서버의 IP 주소
			</td>
		</tr>
		<tr>
			<td>
				userid&#95;name<br/>
				맥락: http, server, location
			</td>
			<td>
				쿠키에 할당할 이름을 정한다.<br/>
				구문: userid&#95;name 이름;<br/>
				기본값: uid
			</td>
		</tr>
		<tr>
			<td>
				userid&#95;domain<br/>
				맥락: http, server, location
			</td>
			<td>
				쿠키에 할당할 도메인을 정한다.<br/>
				구문: userid&#95;domain domain;<br/>
				기본값: 없음(쿠키의 도메인 부분이 전송되지 않음)
			</td>
		</tr>
		<tr>
			<td>
				user&#95;path<br/>
				맥락: http, server, location
			</td>
			<td>
				쿠키의 경로 부분을 정한다.<br/>
				구문: userid&#95;path 경로;<br/>
				기본값: /
			</td>
		</tr>
		<tr>
			<td>
				userid&#95;expires<br/>
				맥락: http, server, location
			</td>
			<td>
				쿠키의 만기일을 정한다.<br/>
				구문: userid&#95;expires 날짜 | max;<br/>
				기본값: 만기일 없음
			</td>
		</tr>
		<tr>
			<td>
				userid&#95;p3p<br/>
				맥락: http, server, location
			</td>
			<td>
				쿠키와 함께 전송될 P3P 헤더의 값을 할당한다.<br/>
				구문: userid&#95;p3p 문자열 | none;<br/>
				기본값: none
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

###### 4.3.4.6. 참조 모듈
<br/>

참조(Referer) 모듈의 지시어는 단순한 valid&#95;referers 하나뿐이다. 이 지시어의 목적은 클라이언트 요청에서 Referer HTTP 헤더를 확인하고, 값에 따라 접근을 거절할 수도 있다. 참조 사이트가 유효하지 않다고 판단되면 $invalid&#95;referer 변수가 1이 된다. 유효한 참조 사이트 목록에 다음 세 종류의 값을 사용할 수 있다.  

+ none: 참조 사이트 정보가 없으면 유효하다고 취급
+ blocked: (XXXXX 같이) 마스킹된 참조 사이트도 유효하다고 취급
+ 서버 이름: 특정 서버 이름은 유효하다고 취급  

다음은 $invalid&#95;referer 변수를 정한다. 예를 들어 참조 사이트가 유효하지 않다고 판단되면 오류 코드를 반환한다.  

```
valid_referers none blocked *.website.com *.google.com;
if ($invalid_referer) {
	return 403;
}
```

Referer HTTP 헤더를 조작하는 것은 매우 간단한 일이다. 따라서 클라이언트 요청의 참조 사이트를 확인하는 것은 보안 목적으로 쓰여서는 안 된다.  

이 모듈에는 두 가지 지시어가 더 제공된다. referer&#95;hash&#95;bucket&#95;size는 헤시 테이블의 버킷 크기를 정하고 referer&#95;hash&#95;max&#95;size는 해시 테이블의 최대 유효 참조 사이트 길이다.  

###### 4.3.4.7. Real IP 모듈
<br/>

이 모듈은 단순 기능 하나를 제공한다. 프록시를 통해 방문한 클라이언트의 IP 주소와 포트 번호를 X-Real-IP HTTP 헤더에 지정된 값으로 교체하거나, 엔진엑스가 뒷단 서버로 사용될 경우에 적절한 헤더에서 IP 주소를 취득한다(이 모듈은 아파치의 mod&#95;rpaf와 같은 효과를 준다.). 이 기능을 활성화하려면 활용할 HTTP 헤더(X-Real-IP나 X-Forwarded-For)를 정의하는 real&#95;ip&#95;header 지시어를 삽입해야 한다. 그다음엔 신뢰하는 IP 주소, 다시 말해 이 헤더를 사용할 클라이언트를 정의해야 한다. 이 작업은 IP 주소와 CIDR 주소 범위를 모두 인자로 받는 set&#95;real&#95;ip&#95;from 지시어로 할 수 있다.  

```
real_ip_header X-Forwarded-For;
set_real_ip_from 192.168.0.0/16;
set_real_ip_from 127.0.0.1;
set_real_ip_from unix; # UNIX 도메인 소켓은 모두 신뢰
```

원래 클라이언트 주소와 포트 번호는 $realip&#95;remote&#95;addr과 $realip&#95;remote&#95;port 변수에 보관된다.  

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

###### 4.3.5. 클라이언트 분리 모듈
<br/>

클라이언트 분리 모듈은 방문객 전체를 특정 비율에 따라 하위 그룹으로 나눠 자원을 효율적으로 활용할 수 있게 해준다. 방문객을 여러 그룹으로 분산시킬 때 엔진엑스는 (방문객의 IP 주소, 쿠키 데이터, 질의 인자 등) 제공되는 값을 해시해서 어느 그룹에 방문객을 배정할지 결정한다. 다음은 방문객을 IP 주소에 따라 세 그룹으로 나누는 구성 예다. 방문객이 처음 50%에 해당하면 $variable 값은 group1이 된다.  

```
split_clients "$remote_addr" $variable {
	50% "group1";
	30% "group2";
	20% "group3";
}
location ~ \.php$ {
	set $args "${query_string}&group=${variable}";
}
```

##### 4.3.6. SSL과 보안
<br/>

###### 4.3.6.1. SSL 모듈
<br/>

SSL 모듈은 HTTPS를 지원한다. 정확히 말하면 SSL/TLS 보안 계층 위로 HTTP를 전송한다. 이 모듈에는 아래 지시어로 정의하는 인증서, 인증 키 등 다양한 매개변수를 통해 안전한 웹 사이트를 제공하는 여러 옵션이 있다.  

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				ssl<br/>
				맥락: http, server
			</td>
			<td>
				구형 버전에서만 사용할 수 있다. 엔진엑스 1.15.0 이후에는 listen의 ssl 인자를 대신 사용하자.<br/>
				지정한 서버에 HTTPS를 활성화한다. 이 지시어는 좀 더 일반적으로 listen 443 ssl이나 listen port ssl과 동등한 것이다.<br/>
				구문: on이나 off<br/>
				기본값: ssl off;
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;certificate<br/>
				맥락: http, server
			</td>
			<td>
				PEM 인증서의 경로를 정한다. 다른 유형의 인증서를 지정하도록 여러 번 사용할 수 있다. 변수를 사용할 수 있다.<br/>
				구문: 파일 경로
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;certificate&#95;key<br/>
				맥락: http, server
			</td>
			<td>
				PEM 비밀 키 파일 경로를 정한다. 다른 유형의 인증서를 지정하게 여러 번 사용할 수 있다. 변수를 사용할 수 있다.<br/>
				구문: 파일 경로
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;client&#95;certificate<br/>
				맥락: http, server
			</td>
			<td>
				클라이언트 PEM 인증서의 경로를 정한다.<br/>
				구문: 파일 경로
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;crl<br/>
				맥락: http, server
			</td>
			<td>
				엔진엑스에게 인증서의 폐기 상태를 확인할 수 있는 인증서 폐기 목록(CRL, Certificate Revocation List) 파일을 읽도록 지시한다.
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;dhparam<br/>
				맥락: http, server
			</td>
			<td>
				디피 헬만(Diffie-Hellman) 매개변수 파일의 경로를 정한다.<br/>
				구문: 파일 경로
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;protocol<br/>
				맥락: http, server
			</td>
			<td>
				사용할 프로토콜을 지정한다.<br/>
				구문: ssl&#95;protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2] [TLSv1.3]<br/>
				기본값: ssl&#95;protocols TLSv1 TLSv2 TLSv1.1 TLSv1.2;
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;ciphers<br/>
				맥락: http, server
			</td>
			<td>
				사용할 암호화 방식을 지정한다. 사용할 수 있는 암호화 방식은 셸에서 openssl ciphers 명령을 실행시켜 얻을 수 있다.<br/>
				구문: ssl&#95;ciphers cipher1[:cipher2...];<br/>
				기본값: ssl&#95;ciphers ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;prefer&#95;server&#95;ciphers<br/>
				맥락: http, server
			</td>
			<td>
				서버의 암호화 방식을 클라이언트의 암호화 방식보다 선호할지 여부를 지정한다.<br/>
				구문: on이나 off<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;verify&#95;client<br/>
				맥락: http, server
			</td>
			<td>
				클라이언트에서 보내온 인증서를 검증해서 그 결과를 $ssl&#95;client&#95;verify 변수에 저장하게 한다. optional&#95;no&#95;ca 인자를 주면 인증서가 유효한지 검증은 하지만 신뢰할 수 있는 인증기관의 서명이 필요하진 않다.<br/>
				구문: on | off | optional | optional&#95;no&#95;ca<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;verify&#95;depth<br/>
				맥락: http, server
			</td>
			<td>
				클라이언트 인증 사슬(certificate chain)의 검증 깊이를 지정한다.<br/>
				구문: 숫자 값<br/>
				기본값: 1
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;session&#95;cache<br/>
				맥락: http, server
			</td>
			<td>
				SSL 세션의 캐시를 구성한다.<br/>
				구문: off, none, builtin:크기, shared:이름:크기<br/>
				기본값: off(SSL 세션 비활성화)
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;session&#95;timeout<br/>
				맥락: http, server
			</td>
			<td>
				SSL 세션이 활성화됐을 경우 이 지시어는 세션 데이터의 제한시간을 정한다.<br/>
				구문: 시간 값<br/>
				기본값: 5m
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;password&#95;phrase<br>
				맥락: http, server
			</td>
			<td>
				비밀 키의 비밀번호를 담고 있는 파일을 지정한다. 각 비밀번호는 별도 줄에 지정된다. 인증서 키를 읽을 때 첫 비밀번호부터 하나씩 시도하게 된다.<br/>
				구문: 파일명<br/>
				기본값: 없음1
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;buffer&#95;size<br/>
				맥락: http, server
			</td>
			<td>
				SSL을 통해 요청을 제공할 때 사용할 버퍼 크기를 지정한다.<br/>
				구문: 크기 값<br/>
				기본값: 16k
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;session&#95;tickets<br/>
				맥락: http, server
			</td>
			<td>
				TLS 세션 티켓을 활성화한다. 이 티켓을 통해 클라이언트가 다시 협상하는 과정 없이 신속히 재접속할 수 있게 된다.<br/>
				구문: on이나 off<br/>
				기본값: on
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;session&#95;ticket&#95;key<br/>
				맥락: http, server
			</td>
			<td>
				TLS 세션 티켓을 암/복호화할 때 사용되는 키 파일의 경로를 설정한다. 기본적으로 임의의 값이 생성된다.<br/>
				구문: 파일명<br/>
				기본값: 없음
			</td>
		</tr>
		<tr>
			<td>
				ssl&#95;trusted&#95;certificate<br/>
				맥락: http, server
			</td>
			<td>
				클라이언트 인증서의 신뢰성을 확인할 뿐 아니라 OSCP 응답에 스테이플링(stapling)하는 데도 사용할 신뢰할 수 있는 인증서 파일(PEM 형식)의 경로를 설정한다.<br/>
				구문: 파일명<br/>
				기본값: 없음
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

이 모듈에서는 다음 변수들도 사용할 수 있다.  

+ $ssl&#95;cipher  
현재 요청에 사용된 암호 방식

+ $ssl&#95;client&#95;serial  
클라이언트 인증서의 일련번호

+ $ssl&#95;client&#95;s&#95;dn와 $ssl&#95;client&#95;i&#95;dn  
클라이언트 인증서의 주체(subject)와 발행자 DN 값(RFC 2253 형식)

+ $ssl&#95;client&#95;s&#95;dn&#95;legacy와 $ssl&#95;lcient&#95;i&#95;dn&#95;legacy  
클라이언트 인증서의 주체와 발행자 DN 값(기존 형식)

+ $ssl&#95;protocol  
현재 요청에 사용된 프로토콜

+ $ssl&#95;client&#95;cert, $ssl&#95;client&#95;raw&#95;cert, ssl&#95;client&#95;escaped&#95;cert  
각각 클라이언트 인증서 데이터와 가공되지 않은 원천 인증서 데이터와 PEM 형태로 인코딩된 인증서 데이터

+ $ssl&#95;client&#95;verify  
클라이언트 인증서가 성공적으로 검증되면 SUCCESS 값을, 실패할 때 "FAILED:certificate has expired" 같이 실패한 이유를 갖음

+ $ssl&#95;client&#95;v&#95;start  
클라이언트 인증서 시작일

+ $ssl&#95;client&#95;v&#95;end  
클라이언트 인증서 만료일

+ $ssl&#95;client&#95;v&#95;remain  
클라이언트 인증서 만료일까지 남은 일 수

+ $ssl&#95;session&#95;id  
SSL 세션의 ID  

###### 4.3.6.2. SSL 인증서 설정
<br/>

SSL 모듈이 많은 기능을 제공하지만 대부분의 경우 보안 웹 사이트를 설정하는 데 두어 가지 지시어만 실제로 유용하게 쓰인다. 이 지침은 웹 사이트용 SSL 인증서를 사용하도록 엔진엑스를 구성하는 데 도움이 될 것이다. 여기세어 대상 웹 사이트는 secure.website.com이라고 정하겠다. 설정을 하기 전에 다음이 준비돼 있는지 확인하자.  

+ openssl genrsa -out secure.website.com.key 1024 명령으로 생성한 .key 파일(1024 이외의 암호화 수준도 가능)
+ openssl req -new -key secure.website.com.key -out secure.website.com.csr 명령으로 생성한 .csr 파일
+ 인증기관에서 발급받은 경우 웹 사이트 인증서 파일, 예를 들어 secure.website.com.crt(주의: 인증기관에서 인증서를 얻으려면 .csr 파일을 제공해야 함)
+ 인증기관에서 발급받은 인증기관 인증서 파일(예를 들어 http://www.GoDaddy.com에서 인증서를 구입했다면 gd&#95;bundle.crt)  

첫 단계에서 다음 명령으로 웹 사이트 인증서와 인증기관 인증서를 하나로 합친다.  

```
cat secure.website.com.crt gd_bundle.crt > combined.crt
```

이제 엔진엑스를 보안 콘텐트를 제공하도록 구성할 준비가 됐다.  

```
secure {
	listen 443;
	server_name secure.website.com;
	ssl on;
	ssl_certificate /path/to/combined.crt;
	ssl_certificate_key /path/to/secure.website.com.key;
}
```

###### 4.3.6.3. SSL 스테이플링
<br/>

온라인 인증 상태 프로토콜(OCSP, Online Certificate Status Protocol) 스테이플링으로도 불리우는 SSL 스테이플링(SSL Stapling)은 인증기관에 접속할 필요 없이 클라이언트가 SSL/TLS 서버와 접속하거나 세션을 재개하기 쉽게 해주는 기술로, SSL 혐상 시간을 단축한다. 보통 OCSP 트랜잭션은 클라이언트가 일반적으로 인증기관에 접속해서 서버의 인증서 폐기 상태를 확인하기 때문에 인증기관 서버에 엄청난 부하를 줄 수 있다. 이런 문제를 위해 설계된 해법이 스테이플링이다. OCSP 기록은 서버가 인증기관에서 정기적으로 취득한다. 그리고 클라이언트와 교환하려고 스테이플된다. 인증기관과 통신을 줄이고자 OCSP 기록이 서버에 최대 48시간까지 캐시된다.  

SSL 스테이플링을 사용하면 방문객과 서버 간의 통신 속도가 올라간다. 엔진엑스에 이를 적용하는 건 상대적으로 단순하다. 실제 필요한 작업은 다음과 같은 세 지시어를 sever 블록에 추가하고 (루트와 중간 인증서 모두를 갖고 있는) 신뢰하는 인증서 사슬 파일을 인증기관에서 얻는 것뿐이다.  

+ ssl&#95;stapling on: server 블록 내에 스테이플링 활성화
+ ssl&#95;stapling&#95;verify on: OCSP 응답 검증 활성화
+ ssl&#95;trusted&#95;certificate 파일 경로: 신뢰할 전체 인증서 파일 경로 지정(확장자는 .pem이어야 함)  

이 모듈의 동작을 변경하는 데 선택적으로 사용할 수 있는 지시어가 두 개 있다.  

+ ssl&#95;stapling&#95;file 파일명: 캐시된 OCSP 기록의 경로, 인증서 파일에 지정된 OCSP 서버 응답의 기록을 대체한다.
+ ssl&#95;stapling&#95;responder url: url은 인증기관의 OCSP 서버 URL이며, 인증서 파일에 지정된 URL 값을 대체한다.  

OCSP 서버에 접속하는 데 문제가 있다면 엔진엑스 구성에 유효한 DNS 서버가 (resolver 지시어를 사용해서) 지정됐는지 확인하자.  

##### 4.3.7. HTTP/2
<br/>

엔진엑스는 1.9.5부터 그동안 지원하던 SPDY 프로토콜을 제거하고 대신 HTTP/2를 지원한다. HTTP/2 모듈은 기본으로 포함되지 않는다. listen 지시어의 마지막에 http2 키워드를 서버에 추가해서 HTTP/2를 활성화할 수 있다.  

```
server {
	listen 443 ssl http2;
	[...]
}
```

HTTP/2의 속성상 SSL 계층 위에서만 동작한다. 다음과 같은 지시어들이 이 모듈에서 제공된다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				http2&#95;body&#95;preread&#95;size<br/>
				맥락: http, server
			</td>
			<td>
				각 요청의 본문을 처리하기 전에 보관할 버퍼의 크기를 지정한다.<br/>
				구문: 크기<br/>
				기본값: http2&#95;body&#95;preread&#95;size 64k;
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;chunk&#95;size<br/>
				맥락: http, server, location
			</td>
			<td>
				응답 본문을 분할할 HTTP/2 분할의 크기를 지정한다.<br/>
				구문: 크기<br/>
				기본값: http2&#95;chunk&#95;size 8k
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;idle&#95;timeout<br/>
				맥락: http, server
			</td>
			<td>
				연결이 닫힌 후 비활성 시간의 한도를 지정한다.<br/>
				구문: 시간<br/>
				기본값: http2&#95;idle&#95;timeout 3m;
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;max&#95;concurrent&#95;pushes<br/>
				맥락: http, server
			</td>
			<td>
				한 연결에서 동시 푸시 요청의 최대 수를 제한한다.<br/>
				구문: 숫자<br/>
				기본값: http2&#95;max&#95;concurrent&#95;pushes 10;
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;max&#95;concurrent&#95;streams<br/>
				맥락: http, server
			</td>
			<td>
				한 연결에서 동시 HTTP/2 스트림의 최대 수를 지정한다.<br/>
				구문: 숫자<br/>
				기본값: http2&#95;max&#95;concurrent&#95;streams 128;
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;max&#95;header&#95;size<br/>
				맥락: http, server
			</td>
			<td>
				HPACK 압축을 해제한 후 전체 요청 헤더 목록의 최대 크기를 지정한다.<br/>
				구문: 크기<br/>
				기본값: http2&#95;max&#95;filed&#95;size 16k;
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;max&#95;header&#95;size<br/>
				맥락: http, server
			</td>
			<td>
				HPACK으로 압축된 헤더 필드의 최대 크기를 지정한다.<br/>
				구문: 크기<br/>
				기본값: http2&#95;max&#95;field&#95;size 4k;
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;max&#95;requests<br/>
				맥락: http, server
			</td>
			<td>
				HTTP/2 연결 하나로 제공되는 최대의 요청 수를 지정한다.<br/>
				구문: 숫자<br/>
				기본값: http2&#95;max&#95;requests 1000;
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;push<br/>
				맥락: http, server, location
			</td>
			<td>
				요청에 대한 응답을 클라이언트에 보내면서 지정된 url로 선제적(pre-emptively) 요청을 발행한다.<br/>
				uri에는 변수를 포함시킬 수 있고 http2&#95;push 지시어를 한 블록에서 여러 번 사용할 수 있다.<br/>
				구문: 숫자 값<br/>
				기본값: http2&#95;push off;
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;push&#95;preload<br/>
				맥락: http, server, location
			</td>
			<td>
				응답 link 헤더에 지정된 preload 유형의 링크를 푸시 요청으로 자동 변환한다.<br/>
				구문: on|off;<br/>
				기본값: http2&#95;push&#95;preload off;
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;recv&#95;buffer&#95;size<br/>
				맥락: http
			</td>
			<td>
				작업자 프로세스당 입력 버퍼의 크기를 지정한다.<br/>
				구문: 크기<br/>
				기본값: http2&#95;recv&#95;buffer&#95;size off;
			</td>
		</tr>
		<tr>
			<td>
				http2&#95;recv&#95;timeout<br/>
				맥락: http, server
			</td>
			<td>
				연결이 닫힌 후에 클라이언트의 추가 데이터를 기다릴 최대의 대기 시한을 지정한다.<br/>
				구문: 시간 값<br/>
				기본값: http2&#95;recv&#95;timeout 30s;
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

위 지시어와 함께 엔진엑스 HTTP/2 모듈은 $http2 변수도 제공한다. 이 변수에는 합의된 프로토콜 ID 값을 갖는다.  

+ h2: TLS 기반의 HTTP/2
+ h2c: TCP 평문 기반의 HTTP/2
+ 빈문자열: 기타  

###### 4.3.7.1. 보안 링크
<br/>

SSL 모듈과는 아무런 상관업시 보안 링크(Secure link)는 사용자가 자원에 접근하기 던에 URL에 특정 해시 값이 있는지 확인함으로써 기본적으로 보호한다.  

```
location /downloads/ {
	secure_link_md5 "secret";
	secure_link $arg_hash,$arg_expires;
	if ($secure_link = "") {
		return 403;
	}
}
```

이 구성으로 /downloads/ 폴더의 문서는 URL의 질의 문자열에 hash=XXX 매개변수가 있어야 한다(예제의 $arg&#95;hash에 주의), 여기서 XXX는 secure&#95;link&#95;md5 지시어로 정의한 비밀코드의 MD5 해시 값이다. secure&#95;link 지시어의 두 번째 인자는 만기일을 정하는 유닉스 타임스탬프다. URI에 적절한 해시 값이 없거나 만기일을 넘었을 경우 $secure&#95;link 변수는 빈값이 된다. 그 외에는 1이 된다.  

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

##### 4.3.8. 기타 잡다한 모듈
<br/>

###### 4.3.8.1. 현황 모듈
<br/>

현환(stup status) 모듈은 활성 연결 횟수, 처리된 총 요청 횟수 등 서버의 현재 상태에 대한 정보를 제공하도록 설계됐다. 이 모듈을 사용하려면 location 블록에 stub&#95;status 지시어를 추가한다. 이 location 블록에 해당하는 요청은 상태 페이지를 얻게 된다.  

```
location = /nginx_status {
	stub_status on;
	allow 127.0.0.1; # 정보를 외부에 노출하지 않음
	deny all;
}
```

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

모니터릭스(Monitorix)처럼 현황 페이지를 정기적으로 호출하고 통계를 분석하는 서버 모니터링 솔루션이 많으니 관심을 갖자.  

###### 4.3.8.2. 품질 저감 모듈
<br/>

HTTP 품질 저감(Degradation) 모듈은 여유 메모리가 낮아지면 오류 페이지를 반환하도록 서버를 구성해준다. 낮다고 판단할 메모리양을 정의해주면 그에 따라 동작한다. 기준을 정한 후에는 품질 저검을 확인할 위치를 지정해주면 된다.  

###### 4.3.8.3. gperftools
<br/>

이 모듈은 엔진엑스 작업자 프로세스를 위해 구글 성능 도룩 프로파일링 메커니즘과 인터페이스한다. 이 도구는 실행 코드의 성능을 분석해서 보고서를 생성해준다. 프로젝트 공식 웹 사이트(https://github.com/gperftools/gperftools)에서 더 자세한 정보를 찾을 수 있다.  

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

이 기능을 활성화하려면 google&#95;perftools&#95;profiles 지시어를 사용해서 리포트 파일을 생성할 경로를 지정해야 한다.  

```
google_perftools_profiles logs/profiles;
```

###### 4.3.8.4. WebDAV
<br/>

WebDAV는 잘 알려진 HTTP 프로토콜의 확장한 프로토콜이다. HTTP가 방문객이 웹 사이트에서 자원을 다운로드하게(다시 말해 데이터를 읽게) 설계됐다면 WebDAV는 파일이나 폴더를 생성하거나 파일을 이동하고 복사하는 등의 쓰기 작업을 추가함으로써 웹 서버의 기능을 확장한다. 엔진엑스 WebDAV 모듈은 WebDAV 프로토콜의 작은 일부를 구현한다.  

이 모듈은 엔진엑스가 컴파일될 때 자동으로 포함되지 않는다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				dev&#95;methods<br/>
				맥락: http, server, location
			</td>
		</tr>
		<tr>
			<td>
				활성화할 DAV 메서드를 선택한다.<br/>
				구문: dev&#95;methods [off | [PUT] [DELETE] [MKCOL] [COPY] [MOVE]];<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				dev&#95;across<br/>
				맥락: http, server, location
			</td>
			<td>
				현 수준의 접근 권한을 정한다.<br/>
				구문: dev&#95;access [user:r|w|rw] [group:r|w|rw] [all:r|w|rw];<br/>
				기본값: dev&#95;access user:rw;
			</td>
		</tr>
		<tr>
			<td>
				create&#95;full&#95;put&#95;path<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 클라이언트 요청이 존재하지 않는 디렉터리 안에 파일을 생성할 때 어떤 행동을 할지 정한다. on으로 설정하면 디렉터리 경로가 생성된다. off로 설정하면 파일이 생성되지 않는다.<br/>
				구문: on이나 off<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				min&#95;delete&#95;depth<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 DELETE 명령을 처리할 때 삭제할 파일이나 디렉터리 URI의 최소 길이를 정한다.<br/>
				구문: 숫자 값<br/>
				기본값: 0
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 4.3.9. 서드파티 모듈
<br/>

엔진엑스 커뮤니티는 지난 수년간 더 성장했고 많은 첨가 모듈이 서드파티 개발자에 의해 작성됐다. 이런 모듈은 위키 공식 웹 사이트인 http://wiki.nginx.org/nginx3rdPartyModules에서 다운로드할 수 있다. 지금 다운로드할 수 있는 모듈은 광범위한 새로운 기능을 제공하는데, 다음과 같은 것들이 있다.  

+ 화려한 인덱스(Fancy Indexes) 모듈은 엔진엑스의 자동 디렉터리 목록 생성 기능을 향상시킨다.
+ 추가 헤더(Headers More) 모듈은 HTTP 헤더의 유연성을 향상시킨다.
+ 웹 서버의 다양한 부분과 관련된 더 많은 기능이 있다.  

서드파티 모듈은 엔진엑스에 통합하려면 다음의 단순한 세 단계를 따르면 된다.  

+ 모듈과 관련된 .tar.gz 압축 파일을 다운로드한다.
+ tar xzf 모듈&#95;이름.tar.gz 명령으로 압축 파일을 푼다.
+ 엔진엑스를 컴파일할 때 다음과 같이 옵션을 추가한다.  

```
./configure --add-module=/모듈/소스코드/경로 [...]
```

애플리케이션을 컴파일해서 설치하고 나면 일반 엔진엑스 모듈과 같은 방식으로 모듈의 지시어와 변수를 사용할 수 있다.  

자신만의 엔진엑스 모듈을 만드는 데 관심이 있다면 에반 밀러(Evan Miller)가 공개한 훌륭한 지침서 "에밀리의 엔진엑스 모듈 개발 가이드"가 있다. 이 전체 가이드는 그의 개인 웹 사이트(http://www.evanmiller.org/nginx-module-guide.html)에서 볼 수 있다.  

### 5. 엔진엑스와 PHP/파이썬 통합
<br/>

#### 5.1. FastCGI 소개
<br/>

##### 5.1.1. CGI 구조 이해
<br/>

클라이언트가 동적인 페이지 방문하려 할 때 웹 서버는 요청을 받아 외부의 애플리케이션에 전달한다. 애플리케이션은 스크립트를 독립적으로 처리해서 생성된 결과를 웹 서버에 반환하며, 이 결과는 클라이언트에 응답으로 되돌려진다.  

1990년대 초, 웹 서버와 애플리케이션이 연동되도록 공통 게이트웨이 인터페이스(CGI, Common Gateway Interface) 규약이 개발됐다.  

##### 5.1.2. CGI
<br/>

인터넷 협회(ISOC, Internet Society)가 설계한 RFC 3875(CGI v1.1)는 다음과 같이 시작한다.  

> CGI 덕에 HTTP 서버와 CGI 스크립트가 함께 클라이언트의 요청을 처리할 수 있다. <중략> 서버는 네트워크 연결을 관리하고, 데이터를 보내고, 전송 처리하고, 클라이언트 요청과 연관된 네트워크 문제를 관리할 책임을 진다.  

CGI는 웹 서버(엔진엑스)와 게이트웨이 애플리케이션(PHP, 파이썬 등) 간에 정보를 교환할 방법을 기술한 규약이다. 사실 웹 서버가 게이트웨이 애플리케이션(PHP, 파이썬 등)으로 전달해야 할 요청을 받으면 웹 서버는 애플리케이션에 맞는 적절한 명령(예를 들어 /usr/bin/php)으로 실행한다. (브라우저와 요청 정보 같은) 클라이언트 요청 세부 내용은 명령행 인자나 환경 변수로 전달되며, POST나 PUT 요청에 포함된 실제 데이터는 표준 입력으로 보내진다. 실행된 애플리케이션은 처리된 문서의 내용을 표준 출력에 출력하는데, 이를 웹 서버가 획득한다.  

이 기술이 처음에는 단순하고 충분히 효과적인 것처럼 보였지만 중요한 결점이 몇 가지 있다.  

+ 요청마다 새로운 프로세스가 생성된다. 메모리와 다른 상태 정보가 요청과 요청 사이에 상실된다.
+ 프로세스를 기동하면서 시스템의 자원이 소비된다. 동시에 다량의 요청이 발생하면 그만큼 프로세스가 생성되면서 서버가 순식간에 아수라장이 된다.
+ 웹 서버가 애플리케이션을 서로 다른 컴퓨터에 분산하는 아키텍처를 설계하기 불가능하진 않겠지만 어려울 것이다.  

##### 5.1.3. FastCGI
<br/>

앞서 설명한 CGI의 문제 때문에 CGI는 부하가 심해지곤 하는 서버에는 상대적으로 비효율적이었다. 1990년대 중반, 해결책을 찾으려고 노력하던 중 오픈 마켓(Open Market)은 CGI를 진화시킨 고속 공통 게이트웨이 인터페이스(FastCGI, Fast Common Gateway Interface)를 개발했다. FastCGI는 지난 15년이 넘는 기간 동안 주요 표준이 됐고, 이제는 마이크로소프트 IIS 등의 상용 서버 소프트웨어를 비롯한 대부분의 웹 서버가 FastCGI 기능을 제공한다.  

목적은 여전히 같지만, FastCGI는 다음과 같은 원리를 수립해 CGI에 비해 뚜렷한 개선점을 제공한다.  

+ 요청마다 새 프로세스를 생성하는 대신 FastCGI는 영구적으로 지속되는 프로세스가 여러 요청을 처리하게 한다.
+ 웹 서버와 게이트웨이 애플리케이션은 TCP나 POSIX 로컬 IPC 소켓 같은 소켓을 사용해서 통신한다. 결국 웹 서버와 뒷단의 프로세스를 네트워크상의 다른 두 컴퓨터를 분산 배치할 수 있다.
+ 웹 서버는 단일 네트워크 연결로 게이트웨이에 클라이언트의 요청을 전달하고 응답을 받는다. 게다가 이어지는 요청을 부가 생성하지 않고도 이어서 보낼 수 있다. 다만 엔진엑스와 아파치(Apache)를 포함한 대부분의 웹 서버에서 구현된 FastCGI는 (적어도 완전한) 멀티플렉싱을 지원하지 않는다.  
+ FastCGI가 소켓 기반의 프로토콜인 만큼 FastCGI는 어떤 플랫폼에서든 어떤 언어를 사용해서도 구현할 수 있다.  

FastCGI를 사용한 아키텍처를 설계하는 일은 사실 상상하는 만큼 복잡하지는 않다. 웹 서버와 뒷단에서 실행하는 애플리케이션이 있는 것은 동일하다. 남은 난제는 두 구성 요소를 서로 연결하는 일뿐이다. 이런 구조에서 첫 단계로 엔진엑스가 FastCGI 애플리케이션과 소통하도록 설정해야 한다. 엔진엑스와 FastCGI가 호환되게 하는 기능은 FastCGI 모듈이 담당한다. 이 모듈은 여러 엔진엑스 빌드에 기본으로 포함돼 있으며, 다양한 소프트웨어 저장소에서 다운로드해 설치한 엔진엑스 빌드도 마찬가지다.  

##### 5.1.4. uWSGI와 SCGI
<br/>

+ uWSGI 모듈은 엔진엑스와 애플리케이션이 uwsgi 프로토콜로 통신하게 해준다. uwsgi 프로토콜은 웹 서버 게이트웨이 인터페이스(WSGI, Web Server Gateway Interface)에서 파생됐다. (고유의 구현이 아닌) 가장 많이 사용되는 uwsgi 프로토콜 서버 구현은 조금은 뻔한 이름의 uWSGI 서버다. 최신 문서는 http://uwsgi-docs.readthedocs.org에서 찾을 수 있다. uWSGI 프로젝트가 주로 파이썬 애플리케이션용으로 설계됐음을 고려하면 이 모듈은 파이썬 전문가에게 유용할 것이다.  

+ SCGI는 단순 일반 게이트웨이 인터페이스(Simple Common Gateway Interface)의 약자로 CGI 프로토콜의 변종이며 FastCGI와 유사하다. FastCGI보다 최신 프로토콜로 규약이 처음 공개된 해가 2006년이다. SCGI는 좀 더 쉽고 이름과 같이 단순하게 구현할 수 있도록 설계됐다. SCGI 인터페이스와 모듈은 아파치, IIS, 자바, 체로키(Cherokee)등 다양한 소프트웨어 프로젝트에서 발견할 수 있다.  

엔진엑스가 FastCGI, uWSGI, SCGI 프로토콜을 다루는 방법에는 큰 차이가 없다. 프로토콜마다 비슷한 이름의 지시어를 가진 모듈을 각각 갖고 있다.  

게다가 엔진엑스 개발 팀은 세 모듈 모두를 지금까지 동시에 유지 보수해 왔다. 새로운 지시어나 지시어 개선 사항은 언제나 세 모듈 모두에 적용된다. 따라서 엔진엑스와 FastCGI 프로토콜 구현을 문서화하더라도 uWSGI와 SCGI에도 적용된다.  

##### 5.1.5. 주요 지시어
<br/>

FastCGI, uWSGI, SCGI 모듈은 기본 엔진엑스 빌드에 포함돼 있다. 컴파일하면서 따로 활성화할 필요가 없다. 아래 표에 나열된 지시어로 엔진엑스가 FastCGI/uWSGI/SCGI 애플리케이션에 요청을 전달하는 방식을 구성할 수 있다. fastcgi&#95;params, uwsgi&#95;params, scgi&#95;params 파일은 대부분의 경우의 유용한 지시 값을 정의해 놓은 파일로, 엔진엑스 구성 폴더에서 찾을 수 있다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				fastcgi&#95;pass<br/>
				맥락: location, if
			</td>
			<td>
				위치에 따라 요청이 FastCGI로 전달되도록 지정하는 지시어다.<br/>
				+ TCP 소켓용 구문: fastcgi&#95;pass 호스트&#95;이름:포트;
				+ 유닉스 도메인 소켓용 구문: fastcgi&#95;pass unix:/도메인/소켓/경로;
				+ 업스트림(upstream) 블록 지정(자세한 설명은 다음 절 참고): fastcgi&#95;pass 블록명;<br/>
				예:<br/>
				```
				fastcgi_pass localhost:9000;
				fastcgi_pass 127.0.0.1:9000;
				fastcgi_pass unix:/tmp/fastcgi.socket;
				# 업스트림 블록 사용
				upstream fastcgi {
					server 127.0.0.1:9000;
					server 127.0.0.1:9001;
				}
				location ~* \.php$ {
					fastcgi_pass fastcgi;
				}
				```
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;param<br/>
				맥락: http, server, location
			</td>
			<td>
				요청이 FastCGI로 전달되도록 구성하는 지시어다. 모든 FastCGI 요청에 SCRIPT&#95;FILENAME과 QUERY&#95;STRING 두 매개변수가 필수다.<br/>
				예:<br/>
				fastcgi&#95;param SCRIPT&#95;FILENAME /home/website.com/www$fastcgi&#95;script&#95;name;<br/>
				fastcgi&#95;param QUERY&#95;STRING $query&#95;string;<br/>
				POST 요청에는 REQUEST&#95;METHOD, CONTENT&#95;LENGTH 매개변수가 추가로 필요하다.<br/>
				fastcgi&#95;param REQUEST&#95;METHOD $request&#95;method;<br/>
				fastcgi&#95;param CONTENT&#95;TYPE $content&#95;type;<br/>
				fastcgi&#95;param CONTENT&#95;LENGTH $content&#95;length;<br/>
				엔진엑스 환경 구성 폴더에 있는 fastcgi&#95;params 파일에는 FastCGI 구성마다 지정해야 하는 모든 필수 매개변수(SCRIPT&#95;FILENAME 제외)가 이미 정의돼 있다.<br/>
				매개변수 이름이 HTTP&#95;로 시작한다면 클라이언트 요청에 이미 있을 HTTP 헤더 값을 덮어 쓰게 된다.<br/>
				if&#95;not&#95;empty 키워드는 선택적으로 지정할 수 있는데, 지정한 값이 없을 때에만 매개변수를 전송하도록 엔진엑스에게 시킨다.<br/>
				구문: fastcgi&#95;param 매개변수이름 매개변수값 [if&#95;not&#95;empty];
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;bind<br/>
				맥락: http, server, location
			</td>
			<td>
				소켓을 기기의 IP 주소에 결속시키는 지시어다. FastCGI 통신에 사용하기 원하는 네트워크 인터페이스를 지정할 수 있다.<br/>
				off 값으로 지정하면 상위 수준에서 상속한 구성을 취소한다.<br/>
				transparent 옵션을 지정하면 기기의 IP가 아닌, 예를 들어 클라이언트의 IP로 지정할 수 있다.<br/>
				구문: fastcgi&#95;bind IP&#95;주소 [transparent]|off;<br/>
				예: fastcgi&#95;bind $remote&#95;addr transparent;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;pass&#95;header<br/>
				맥락: http, server, location
			</td>
			<td>
				부가적으로 FastCGI 서버로 전달돼야 할 헤더를 지정하는 지시어다.<br/>
				구문: fastcgi&#95;pass&#95;header 헤더&#95;이름;<br/>
				예: fastcgi&#95;pass&#95;header Authorization;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;hide&#95;header<br/>
				맥락: http, server, location
			</td>
			<td>
				FastCGI에 숨겨야 하는 헤더(엔진엑스가 전달하지 않는 헤더)를 지정하는 지시어다.<br/>
				구문: fastcgi&#95;hide&#95;header 헤더&#95;이름;<br/>
				예: fastcgi&#95;hide&#95;header X-Forwarded-For;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;index<br/>
				맥락: http, server, location
			</td>
			<td>
				FastCGI 서버는 자동 디렉터리 색인을 제공하지 않는다. 요청된 URI가 /로 끝난다면 엔진엑스가 fastcgi&#95;index 값을 덧붙인다.<br/>
				구문: fastcgi&#95;index 파일명;<br/>
				예: fastcgi&#95;index index.php;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;ignore&#95;client&#95;abort<br/>
				문맥: http, server, location
			</td>
			<td>
				클라이언트가 웹 서버에 보낸 요청을 취소하면 어떻게 처리할지 정의할 수 있는 지시어다. 지시어가 커져 있으면 엔진엑스는 요청 취소를 무시하고 요청 처리를 마무리한다. 지시어가 꺼져 있으면 요청 취소를 무시하지 않고 요청을 취소한다. 요청 처리를 중단하고 FastCGI 서버와의 모든 관련 통신을 취소한다.<br/>
				구문: on이나 off<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;intercept&#95;errros<br/>
				맥락: http, server, location
			</td>
			<td>
				게이트웨이가 반환한 오류를 엔진엑스가 처리할지, 아니면 클라이언트에 직접 오류 페이지를 반환할지 지정하는 지시어다(참고: 오류 처리는 error&#95;page 지시어로 수행됨).<br/>
				구문: on이나 off<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;read&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				FastCGI 애플리케이션의 응답 제한시간을 정의하는 지시어다. 엔진엑스가 지정한 시간 안에 응답을 받지 못하면 "504 게이트웨이 시간 초과" HTTP 오류가 반환된다.<br/>
				구문: 시간 값(초 단위)<br/>
				기본값: 60
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;connect&#95;timeout<br/>
				문맥: http, server, location
			</td>
			<td>
				백엔드 서버 연결 시간제한을 정의하는 지시어다. 이 시간제한은 읽기/전송 시간제한과 다르다. 엔진엑스가 백엔드 서버와 이미 연결됐으면 fastcgi&#95;connect&#95;timeout은 적용되지 않는다.<br/>
				구문: 시간 값(초 단위)<br/>
				기본값: 60
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;send&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				백엔드 서버에 데이터를 전송하는 시간을 제한한다. 이 제한은 전체 응답의 지연 시간이 아닌 두 쓰기 동작 사이에만 적용된다.<br/>
				구문: 시간 값(초 단위)<br/>
				기본값: 60
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;split&#95;path&#95;info<br/>
				맥락: location
			</td>
			<td>
				http://website.com/page.php/param1/param2/와 같은 형태의 URL에 특히 유용한 지시어다. 이 지시어는 경로 정보를 지정한 정규식에 따라 분할한다.<br/>
				fastcgi&#95;split&#95;path&#95;info &#94;(.+\.php)(.&#42;)$;<br/>
				이 지시어는 두 변수에 영향을 미친다.<br/>
				+ $fastcgi&#95;script&#95;name: 실행된 실제 스크립트의 파일명(예에서는 page.php)<br/>
				+ $fastcgi&#95;path&#95;info: 스크립트명 뒤에 오는 URL의 일부(예에서는 /param1/param2/)<br/>
				두 변수는 추후 매개변수 정의에 사용될 수 있다.<br/>
				fastcgi&#95;param SCRIPT&#95;FILENAME /home/website.com/www$fastcgi&#95;script&#95;name;<br/>
				fastcgi&#95;param PATH&#95;INFO $fastcgi&#95;path&#95;info;<br/>
				구문: 정규식
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;store<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 FastCGI 애플리케이션에서 온 응답을 저장 장치에 파일 형태로 저장하는 단순한 캐시 저장 기능을 활성화한다. 동일한 URI로 다시 요청을 받으면 FastCGI 애플리케이션으로 요청을 전달하는 대신 캐시 저장소에 저장된 문서를 직접 제공한다.<br/>
				이 지시어는 캐시 저장 기능을 켜거나 끈다.<br/>
				구문: on이나 off
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;store&#95;access<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 캐시 저장 시 생성되는 파일에 적용될 접근 권한을 정의한다.<br/>
				구문: fastcgi&#95;store&#95;access [user:r|w|rw] [group:r|w|rw] [all:r|w|rw];<br/>
				기본값: fastcgi&#95;store&#95;access user:rw;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;temp&#95;path<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 임시 파일과 캐시 저장 파일의 경로를 지정한다.<br/>
				구문: 파일 경로<br/>
				예: fastcgi&#95;temp&#95;path /tmp/nginx&#95;fastcgi;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;max&#95;temp&#95;file&#95;size<br/>
				맥락: http, server, location
			</td>
			<td>
				FastCGI 요청에 임시 파일을 사용하지 않게 하려면 이 지시어에 0을 설정하고 아니면 최대 파일 크기를 지정한다.<br/>
				기본값: 1GB<br/>
				구문: 파일 크기<br/>
				예: fastcgi&#95;max&#95;temp&#95;file&#95;size 5m;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;temp&#95;file&#95;write&#95;size<br/>
				맥락: http, server, location
			</td>
			<td>
				임시 파일을 저장 장치에 기록할 때 사용할 버퍼의 크기를 지정하는 지시어다.<br/>
				구문: 크기 값<br/>
				기본값: 8k 또는 16k
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;send&#95;lowat<br/>
				맥락: http, server, location
			</td>
			<td>
				TCP 소켓용 SO&#95;SNDLOWAT 플래그를 사용하도록 허용하는 옵션으로, FreeBSD 전용이다. 이 값은 출력 작업용 버퍼의 최소 바이트 수를 정의한다.<br/>
				구문: 숫자 값(크기)<br/>
				기본값: 0
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;pass&#95;request&#95;body<br/>
				fastcgi&#95;pass&#95;request&#95;headers<br/>
				맥락: http, server, location
			</td>
			<td>
				요청 본문과 부가 요청 헤더가 뒷단의 서버에 전달될지 여부를 정하는 지시어다.<br/>
				구문: on이나 off<br/>
				기본값: on
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;ignore&#95;headers<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단 서버가 보낸 응답에서 다음에 나열한 헤더를 엔진엑스가 처리하지 않게 막는 지시어다.<br/>
				+ X-Accel-Redirect
				+ X-Accel-Expires
				+ Expires
				+ Cache-Control
				+ X-Accel-Limit-Rate
				+ X-Accel-Buffering
				+ X-Accel-Charset  <br/>
				구문: fastcgi&#95;ignore&#95;headers header1 [header2...];
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;next&#95;upstream<br/>
				맥락: http, server, location
			</td>
			<td>
				fastcgi&#95;pass가 업스트림 블록과 연결돼 있을 때 이 지시어는 요청을 버리고 해당 블록의 다음 업스트림 서버로 다시 보내야 하는 경우를 정한다. 이 지시어에 다음 값의 조합을 사용할 수 있다.<br/>
				+ error: 서버와 통신하거나 통신을 시도하는 중에 오류가 발생함
				+ timeout: 데이터 전송이나 연결 시도 중에 지정된 시간을 넘음
				+ invalid&#95;header: 뒷단의 서버가 빈 응답이나 잘못된 응답을 반환함
				+ http&#95;500, http&#95;502, http&#95;503, http&#95;504, http&#95;404, http&#95;429: 각 HTTP 오류가 발생했을 경우 엔진엑스는 다음 업스트림으로 전환한다.
				+ non&#95;idempotent: 보통 요청이 뒷단 서버로 전송된 후에는 POST, LOCK, PATCH 같은 비멱등 메서드일 때 다음 서버로 넘기지 않는다. 이 옵션은 재전송하게 만든다.
				+ off: 다음 업스트림 서버로 넘기는 기능을 사용하지 못하게 함<br/>
				예:<br/>
				fastcgi&#95;next&#95;upstream error timeout http&#95;504;<br/>
				fastcgi&#95;next&#95;upstream timeout invalid&#95;header;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;next&#95;upstream&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				fastcgi&#95;next&#95;upstream과 연관돼 사용될 시간제한 값을 정한다. 지시어를 0으로 설정하면 시간을 제한하지 않는다.<br/>
				구문: 시간 값(초 단위)
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;next&#95;upstream&#95;tries<br/>
				맥락: http, server, location
			</td>
			<td>
				오류 메시지를 반환하기 전에 시도할 최대 업스트림 서버 수를 정의한다. fastcgi&#95;next&#95;upstream과 함께 사용된다.<br/>
				구문: 숫자 값(초 단위)
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;catch&#95;stderr<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 stderr로 보내질 오류 메시지 중 일부를 가로채서 엔진엑스 오류 로그에 저장하게 된다.<br/>
				구문: fastcgi&#95;catch&#95;stderr filter;<br/>
				예: fastcgi&#95;catch&#95;stderr "PHP Fatal error:";
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;keep&#95;conn<br/>
				맥락: http, server, location
			</td>
			<td>
				on으로 설정하면 엔진엑스 FastCGI 서버와의 연결을 보존해 부하를 줄인다.<br/>
				구문: on이나 off(기본값: off)<br/>
				주의: uWSGI와 SCGI 모듈에는 상응하는 지시어가 없다.
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;socket&#95;keepalive<br/>
				맥락: http, server, location
			</td>
			<td>
				FastCGI 연결이 유지되도록 TCP 소켓을 구성한다. 기본적으로는 운영체제의 소켓 설정이 그대로 반영된다. 이 지시어의 값이 on이면 SO&#95;KEEPALIVE 소켓 옵션이 사용하는 소켓에 활성화된다.<br/>
				구문: fastcgi&#95;socket&#95;keepalive on | off;<br/>
				기본값: off<br/>
				예: fastcgi&#95;socket&#95;keepalive on;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;force&#95;ranges<br/>
				맥락: http, server, location
			</td>
			<td>
				on으로 설정하면 엔진엑스는 FastCGI 백엔드에서 오는 응답의 범위를 바이트 단위로 지정하는 기능을 활성화한다.<br/>
				구문: on이나 off(기본값: off)
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;limit&#95;rate<br/>
				맥락: http, server, location
			</td>
			<td>
				엔진엑스가 FastCGI 백엔드에서 오는 응답을 다운로드하는 속도를 제한한다.<br/>
				구문: 숫자 값(초당 바이트)
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache<br/>
				맥락: http, server, location
			</td>
			<td>
				캐시 구역(cache zone)을 정의하는 지시어다. 구역에 부여된 식별 문자는 나중에 재사용된다.<br/>
				구문: fastcgi&#95;cache 식별&#95;문자;<br/>
				예: fastcgi&#95;cache&#95;cache1;
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 5.1.6. FastCGI 캐싱과 버퍼링
<br/>

엔진엑스를 FastCGI 애플리케이션과 동작하도록 올바르게 구성했다면 캐시 시스템을 설정해 전반적인 서버 성능을 향상시키는 데 도움이 되는 다음 지시어를 선택적으로 사용할 수 있다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				fastcgi&#95;cache<br/>
				맥락: http, server, location
			</td>
			<td>
				캐시 영역을 정의하는 지시어다. 영역에 주어지는 ID는 이후 지시어에서  재사용된다.<br/>
				구문: fastcgi&#95;cache 영역&#95;이름;<br/>
				예: fastcgi&#95;cache cache1;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache&#95;key<br/>
				맥락: http, server, location
			</td>
			<td>
				캐시 키를 정의하는 지시어로, 이 키로 캐시에 등록하는 항목을 다른 항목과 구별 짓는다. 캐시 키를 $uri로 설정하면 결국 모든 유사한 $uri을 가진 모든 요청은 동일한 캐시 항목에 대응하게 된다. 이는 대부분의 동적 웹 사이트에는 충분하지 않은데, 질의 문자열(query string) 인자도 캐시 키에 포함해야만 /index.php와 /index.php?page=contact가 동일 캐시 항목을 가리키지 않게 된다.<br/>
				구문: fastcgi&#95;cache&#95;key 키;<br/>
				예: fastcgi&#95;cache "$scheme$host$request&#95;uri $cookie&#95;user";
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache&#95;methods<br/>
				맥락: http, server, location
			</td>
			<td>
				캐시할 HTTP 메서드를 정의하는 지시어다. GET과 HEAD는 기본으로 포함되며 비활성화할 수 있다. 예를 들어 POST 요청을 캐시에 포함시킬 수 있다.<br/>
				구문: fastcgi&#95;cache&#95;methods 메서드;<br/>
				예: fastcgi&#95;cache&#95;methods POST;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache&#95;min&#95;uses<br/>
				맥락: http, server, location
			</td>
			<td>
				요청이 캐싱 후보가 되기 전가지의 최소 적중수를 정의하는 지시어다. 기본적으로 요청의 응답은 한 번 적중된 후에 캐시가 된다(동일 캐시 키를 가진 이후 요청은 캐시된 응답을 받게 된다).<br/>
				구문: 숫자 값<br/>
				예: fastcgi&#95;cache&#95;min&#95;uses 1;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache&#95;path<br/>
				맥락: http
			</td>
			<td>
				캐시 파일이 저장될 디렉터리를 가리키는 지시어다.<br/>
				구문: fastcgi&#95;cache&#95;path path [levels=숫자들] keys&#95;zone=이름:크기 [inactive=시간] [max&#95;size=크기] [loader&#95;files=숫자] [loader&#95;sleep=시간] [loader&#95;threshold=시간];<br/>
				부가 매개변수는 다음과 같다.<br/>
				+ levels: 하위 디렉터리의 깊이를 나타냄(1:2는 두 단계까지 하위 디렉터리가 만들어질 것을 나타냄)
				+ keys&#95;zone: fastcgi&#95;cache 지시어로 선언된 영역을 선택하고 메모리에서 차지하는 용량을 나타냄
				+ inactive: 캐시된 응답이 일정 시간 동안 사용되지 않으면 캐시에서 제거될(기본값: 10분)
				+ max&#95;size: 전체 캐시의 최대 용량을 정의
				+ manager&#95;files, manager&#95;sleep, manager&#95;threshold: 사용한 지 오래된 캐시를 삭제하는 캐시 관리자를 구성한다. 한 주기 안에 삭제할 파일의 수(manager&#95;files, 기본값: 100파일), 삭제 주기 사이의 중단 시간(manager&#95;sleep, 기본 50ms), 삭제 주기의 최대 작업 시간(manager&#95;threshold, 기본값: 200ms)
				+ leader&#95;files, loaded&#95;sleep, loader&#95;threshold: 파일 시스템에 보관된 캐시를 읽는 캐시 로더(cache loader)를 구성한다. 한 주기 안에 처리할 파일의 수(loader&#95;files, 기본값: 100파일), 읽기 주기 사이의 중단 시간(loader&#95;sleep, 기본값: 50ms), 읽는 주기의 최대 작업 시간(loader&#95;threshold, 기본값: 200ms)<br/>
				예:<br/>
				fastcgi&#95;cache&#95;path /tmp/nginx&#95;cache levels=1:2 zone=zone1:10m inactive=10m max&#95;size=200M;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache&#95;use&#95;stale<br/>
				맥락: http, server, location
			</td>
			<td>
				(게이트웨이와 관련된) 특정 상황에서 엔진엑스가 만기된 캐시 데이터라도 제공할지 여부를 정의하는 지시어다. fastcgi&#95;cache&#95;use&#95;stale timeout을 사용하고 게이트웨이에서 시간 내에 응답이 오지 않으면 엔진엑스는 캐시된 데이터를 응답으로 제공한다.<br/>
				Cache-Control 헤더에 stale-while-revalidate 속성이 있으면 업데이트 중에 만기된 캐시 응답이 사용된다. Cache-Control 헤더에 stale-if-error 속성이 있으면 오류가 발생했을 때 만기된 캐시 응답이 사용된다.<br/>
				구문: fastcgi&#95;cache&#95;use&#95;stale [updating] [error] [timeout] [invalid&#95;header] [http&#95;500] [http&#95;502] [http&#95;503] [http&#95;504] [http&#95;404] [http&#95;429] [off];<br/>
				예: fastcgi&#95;cache&#95;use&#95;stale error timeout;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache&#95;background&#95;update<br/>
				맥락: http, server, location
			</td>
			<td>
				on으로 설정하면 만기된 캐시 데이터가 제공되는 동안 배후에서 부가요청을 보내 만기된 캐시 항목을 갱신한다. fastcgi&#95;cache&#95;use&#95;stale에 updating이 설정돼 있어야 한다.<br/>
				구문: fastcgi&#95;cache&#95;background&#95;update on|off;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache&#95;valid<br/>
				맥락: http, server, location
			</td>
			<td>
				응답 코드마다 별도의 캐시 보관 시간을 지정할 수 있는 지시어다. "404" 오류 코드의 응답은 1분 동안 캐시하면서 "200 OK" 응답은 10분 이상 보관해야 할 때가 있다. 이 지시어는 다음과 같이 여러 번 사용할 수 있다.<br/>
				fastcgi&#95;cache&#95;valid 404 1m;<br/>
				fastcgi&#95;cache&#95;valid 500 502 504 5m;<br/>
				fastcgi&#95;cache&#95;valid 200 10;<br/>
				구문: fastcgi&#95;cache&#95;valid code1 [code2...] 시간;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;no&#95;cache<br/>
				맥락: http, server, location
			</td>
			<td>
				요청이 특정 조건에 부합할 때 캐시를 적용하지 않아야 할 때가 있다. 이 지시어는 변수를 여러 개 나열할 수 있다. 이 변수 중 하나라도 값을 갖고 있을 경우(빈 문자열이나 0이 아님) 이 요청은 캐시에 저장되지 않는다.<br/>
				구문: fastcgi&#95;no&#95;cache $변수1 [$변수2] [...]ㅓ;<br/>
				예: fastcgi&#95;no&#95;cache $args&#95;nocaching;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache&#95;bypass<br/>
				맥락: http, server, location
			</td>
			<td>
				캐시에 저장된 응답이 있을 때 응답을 캐시에서 읽어야 할지 알려준다는 것을 제외하면 (요청 결과를 저장할지 판단하는) fastcgi&#95;no&#95;cache와 비슷하게 동작하는 지시어다.<br/>
				구문: fastcgi&#95;cache&#95;bypass $변수1 [$변수2] [...];<br/>
				예: fastcgi&#95;cache&#95;bypass $cookie&#95;bypass&#95; cache;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache&#95;lock, fastcgi&#95;cache&#95;lock&#95;timeout, fastcgi&#95;cache&#95;lock&#95;age<br/>
				맥락: http, server, location
			</td>
			<td>
				on으로 설정하면 fastcgi&#95;cache&#95;lock은 fastcgi&#95;cache&#95;lock&#95;age에 지정한 기간만큼 캐시를 갱신하길 기다리면서 추가 요청을 뒷단으로 보내지 않는다(fastcgi&#95;cache&#95;lock&#95;timeout은 fastcgi&#95;cache&#95;age와 같지만 요청을 캐시하지 않는다).<br/>
				예:<br/>
				fastcgi&#95;cache&#95;lock on;<br/>
				fastcgi&#95;cache&#95;lock timeout 10s;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;cache&#95;revalidate<br/>
				맥락: http, server, location
			</td>
			<td>
				활성화되면 If-modified-since와 If-none-match 헤더로 지시될 때 엔진엑스가 만기된 캐시 항목을 다시 검증한다.<br/>
				구문: fastcgi&#95;cache&#95;revalidate on off;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;buffering, fastcgi&#95;request&#95;buffering<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단의 FastCGI에서 보낸 응답(fastcgi&#95;request&#95;buffers의 경우는 클라이언트의 응답)을 버퍼링할지 말지 결정한다. 비활성화되면 엔진엑스는 응답을 클라이언트에 동기 방식으로 전달한다. 활성화되면 뒷단에서 모든 내용이 다 전송될 때까지 응답을 버퍼에 저장하다가 한 번에 클라이언트에 보낸다.<br/>
				구문: fastcgi&#95;buffering on | off;<br/>
				기본값: on
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;buffers<br/>
				맥락: http, server, location
			</td>
			<td>
				FastCGI 애플리케이션에서 오는 응답 데이터를 읽는 데 사용될 버퍼의 용량과 크기를 설정하는 지시어다.<br/>
				구문: fastcgi&#95;buffers 용량 크기;<br/>
				기본값: 8버퍼, 버퍼당 4k 또는 8k(플랫폼에 따라 다름)<br/>
				예: fastcgi&#95;buffers 8 4k;
			</td>
		</tr>
		<tr>
			<td>
				fastcgi&#95;buffer&#95;size<br/>
				맥락: http, server, location
			</td>
			<td>
				FastCGI 애플리케이션에서 오는 응답의 시작을 읽는 데 사용될 버퍼의 크기를 설정하는 지시어다. 기본값은 이전 지시어(fastcgi&#95;buffers)로 정의한 버퍼 하나의 크기다.<br/>
				구문: fastcgi&#95;buffer&#95;size 값<br/>
				예: fastcgi&#95;buffer&#95;size 4k;
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

다음은 전체 엔진엑스 FastCGI 캐시 구성 예로, 앞의 표에서 설명한 캐시 관련 지시어 대부분을 사용했다.  

```
fastcgi_cache phpcache;
fastcgi_cache_key "$scheme$host$request_uri"; # $request_uri는 요청의 인자를 포함(예: /page.php?arg=value)
fastcgi_cache_min_uses 2; # 캐시에 두 번 적중된 후 캐시된 응답을 제공
fastcgi_cache_path /tmp/cache levels=1:2 keys_zone=phpcache:10m inactive=30m max_size=500M;
fastcgi_cache_use_stale updating timeout;
fastcgi_cache_valid 404 1m;
fastcgi_cache_valid 500 502 504 5m;
```

위의 각 지시어는 어느 가상 호스트 구성에도 유효하기 때문에 (fastcgi&#95;cache 같은) 별도 파일에 저장했다가 적절한 위치에 포함시키고 싶을 것이다.  

```
http {
	......
	server {
		server_name website.com;
		location ~* \.php$ {
			fastcgi_pass 127.0.0.1:9000;
			fastcgi_param SCRIPT_FILENAME /home/website.com/www$fastcgi_script_name;
			fastcgi_param PATH_INFO $fastcgi_script_name;
			include fastcgi_params;
		}
		include fastcgi_cache;
	}
}
```

#### 5.2. 엔진엑스와 PHP
<br/>

FastCGI를 사용해 PHP를 동작하게 구성하려 한다. SCGI나 uWSGI 같은 대안도 있는데 왜 FastCGI를 사용해야 할지 궁금할 것인데, 답은 PHP 버전 5.3.3에 있다. 이 버전부터 시작해 모든 PHP는 FastCGI 프로세스 관리자가 통합돼 FastCGI 프로토콜을 구현하는 애플리케이션을 쉽게 연결할 수 있다. PHP를 빌드할 때 --enable-fpm 옵션으로 구성했다면 아무런 추가 작업도 필요 없다. 지금 설치한 PHP 설정에 필요한 요소가 포함됐는지 확실하지 않더라도 걱정하지 않아도 된다. 직접 빌드하는 대신 저장소 대부분에 등록돼 있는 php-fpm이나 php7-fpm 패키지를 사용하는 방법도 있다.  

##### 5.2.1. 아키텍처
<br/>

설정 과정을 시작하기 전에 PHP가 엔진엑스와 연동되는 방식을 이해하는 것이 중요하다. FastCGI는 소켓을 이용해 동작하는 통신 프로토콜이라고 정의했었는데, 이는 통신의 주체인 클라이언트와 서버가 있음을 뜻한다. 클라이언트는 당연히 엔진엑스다. 문제는 서버인데, 단지 'PHP'라고 답하기엔 사실 좀 복잡하다.  

기본적으로 PHP는 FastCGI 프로토콜을 지원한다. PHP는 스크립트를 처리하며 엔진엑스와 소켓으로 연동할 수 있다. 하지만 우리는 프로세스 관리 전반을 개선하는 부가 요소를 사용하려고 한다. PHP-FPM으로도 알려진 FastCGI 프로세스 관리자가 그것이다.  

PHP-FPM은 FastCGI 지원 수준을 완전히 새로운 경지에 이르게 한다. PHP-FPM의 다양한 특징을 자세히 알아보자.  

##### 5.2.2. PHP-FPM
<br/>

이 프로세스 관리자는 이름이 말하듯 PHP 프로세스를 관리하는 프로세스를 관리하는 스크립트다. PHP-FPM은 엔진엑스에서 수신되는 지시를 기다렸다가 구성된 환경에서 요청된 PHP 스크립트를 실행한다. 실제로 PHP-FPM은 다음과 같은 가능성을 제공한다.  

+ PHP의 자동 데몬 프로세스화(백그라운드 프로세스로 전환)
+ chroot로 격리된 환경에서 스크립트를 수행
+ 로그 개선, IP 주소 제한, 풀(pool) 분리 등  

##### 5.2.3. PHP와 PHP-FPM 설정
<br/>

지금 PHP 5.3.3 이전 버전을 사용한다면 이 각 단계를 따라 PHP를 빌드해야 한다.  

###### 5.2.3.1. 다운로드와 압축 해제
<br/>

압축된 tar 파일을 다음 명령으로 다운로드한다.  

```sh
wget https://www.php.net/distributions/php-7.3.13.tar.gz
```

다운로드가 끝났으면 tar 명령으로 PHP 파일을 압축 해제한다.  

```sh
tar xzf php-7.3.13.tar.gz
```

###### 5.2.3.2. 요구 사항
<br/>

PHP를 PHP-FPM과 빌드하는 데 반드시 필요한 두 가지는 libevent와 libxml 개발 라이브러리다. 이 라이브러리가 시스템에 설치되지 않았다면 사용하는 시스템의 패키지 관리자로 설치해야 한다.  

레드햇 기반의 시스템이나 Yum을 패키지 관리자로 사용하는 다른 시스템에서는 다음 명령을 사용한다.  

```sh
yum install libevent-devel libxml2-devel
```

우분투(Ubuntu), 데비안(Debian)이나 다른 apt-get 또는 aptitude를 사용하는 시스템에서 입력할 명령은 다음과 같다.  

```sh
apt-get install libxml2-dev libevent-dev`
```

###### 5.2.3.3. PHP 컴파일
<br/>

필요한 외부 라이브러리를 모두 설치하면 PHP 빌드 작업을 시작할 수 있다. 이전에 설치했던 다른 애플리케이션이나 라이브러리와 비슷하게 configure, make, make install라는 세 가지 명령이 기본적으로 필요하다. 이 작업으로 새 PHP가 설치될 것이다. 이미 PHP가 패키지로 시스템에 설치돼 있어도 새 PHP가 이미 설치된 PHP를 덮어 쓰지 않게 할 수 있다. configure 명령에 --prefix 옵션으로 설치 위치를 지정하면 다른 위치에 설치된다. 새 PHP가 설치될 위치는 make install 명령이 실행되는 동안에 표시된다.  

첫 단계인 구성(configure)이 가장 중요하다. PHP가 PHP-FPM 관련 기능을 포함하도록 옵션을 활성화해야 하기 때문이다. configure 명령에 전달할 수 있는 구성 인자는 엄청나게 다양하다. 어떤 인자는 데이터베이스 연동, 정규식, 파일 압축 기능, 웹 서버 통합 같은 중요한 기능을 활성화하는 데 필요하다. 다음 명령을 실행하면 모든 구성 옵션 목록이 표시된다.  

```sh
./configure --help
```

최소한의 명령이 사용될 수도 있지만, 다량의 기능이 누락될 것임을 명심하자. 다른 기능 요소가 포함되길 원한다면 부가적인 라이브러리가 필요할 것이다. 어떤 경우라도 --enable-fpm 스위치는 포함해야 한다.  

```sh
./configure --enable-fpm [...]
```

다음 단계는 애플리케이션을 빌드하는 동시에 설치하는 것이다. 설치에는 root 권한이 필요하다.  

```sh
make
sudo make install
```

이 작업은 시스템 사양에 따라 시간이 좀 걸릴 수 있다. 빌드 과정에 제공되는 정보 중 중요한 것을 기록해두자. 컴파일된 이진 실행 파일과 구성 파일의 위치를 지정하지 않았다면 끝날 때 표시될 것이다.  

###### 5.2.3.4. 설치 후 구성
<br/>

새로 설치된 PHP를 구성하는 것으로 시작하자. 예를 들어 기존 PHP의 php.ini를 새 PHP에 복사하는 방법도 있다.  

```sh
sudo cp php.ini-production /usr/local/etc/php.ini
```

엔진엑스가 스크르비트 파일과 요청 정보를 PHP에 전달하는 방식 때문에 cgi.fix&#95;pathinfo=1 구성 옵션을 사용하면 보안 침입이 발생할 수 있다. php.ini 파일에서 이 옵션을 0으로 설정하기를 강력히 권고한다. 이 보안 문제와 관련해 더 자세한 정보를 원하면 다음 글을 참고하자.  

http://cnedelcu.blogspot.com/2010/05/nginx-php-via-fastcgi-important.html  

다음 단계는 PHP-FPM 구성이다. 먼저 /usr/local/etc/에 php-fpm.conf 파일을 생성해서 편집한다. 이 디렉터리에 저장돼 있는 php-fpm.conf.default를 복사하거나 이름을 바꿔서 만들어도 된다. /usr/local/etc/php-fpm.d에도 풀 설정을 해야 하는데, 이미 이 디렉터리에 있는 www.conf.default 파일을 복사하거나 참조해서 .conf로 끝나는 파일을 만들어두면 자동으로 읽혀진다.  

여기에서 PHP-FPM 구성의 모든 내용을 자세히 다룰 수는 없다(구성 파일 안에 다량의 설명이 포함돼 있다). 하지만 잊어서는 안 되는 중요한 구성 지시어가 있다.  

+ 작업자 프로세스와 유닉스 소켓이 사용할 사용자와 그룹 수정
+ PHP-FPM이 통신에 사용할 주소와 포트
+ 동시에 처리할 요청 횟수
+ PHP-FPM에 접속을 허용할 IP 주소  

###### 5.2.3.5. 실행과 제어
<br/>

PHP-FPM 구성 파일을 수정한 후에 다음 명령으로 PHP-FPM을 실행할 수 있다(파일 경로는 빌드 구성에 따라 전혀 다를 수 있음).  

```sh
/usr/local/sbin/php-fpm -c /usr/local/etc/php.ini --pid /usr/local/var/run/php-fpm.pid --fpm-config=/usr/local/etc/php-fpm.conf -D
```

이 명령은 몇 가지 중요한 인자를 포함하고 있다.  

+ -c /usr/local/etc/php.ini: PHP 구성 파일의 경로 지정
+ --pid /usr/local/var/run/php-fpm.pid: 초기화 스크립트에서 프로세스를 제어하는 데 유용한 PID 파일의 경로 지정
+ --fpm-config=/usr/local/etc/php-fpm.conf: PHP-FPM이 특정 구성 파일을 사용하도록 강제 지정
+ -D: PHP-FPM을 데몬화(배경에서 PHP-FPM이 동작하게 됨)  

다른 명령행 인자는 php-fpm -h로 출력할 수 있다.  

kill이나 killall 명령으로 PHP-FPM을 종료시킬 수 있다. 또는 이 프로세스를 실행하거나 중단시키는 데 초기화 스크립트를 사용할 수도 있다. 이 스크립트는 설치한 PHP에 포함돼 있다.  

##### 5.2.4. 엔진엑스 구성
<br/>

PHP-FPM을 올바로 구성하고 시작하는 데 성공했다면 엔진엑스 구성 파일을 손봐서 이 둘이 연결되게 할 차례다. 다음 server 설정 블록은 단순하고 유효한 템플릿으로, 웹 사이트 구성의 기본 바탕으로 사용할 수 있다.  

```
server {
	server_name .website.com; # 서버 이름, www 인정
	listen 80; # 80 포트에 접속 가능
	root /home/website/www; # 최상위 문서 경로
	index index.php; # 기본 요청 파일명: index.php

	location ~* \.php$ { # 요청된 파일이 .php로 끝날 때
		# 앞에서 구성한 FastCGI 주소와 포트 지정
		fastcgi_pass 127.0.0.1:9000;
		# PHP-FPM에 전달될 문서 경로
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		fastcgi_param PATH_INFO $fastcgi_script_name;
		# 다른 FastCGI 관련 구성 설정을 읽어서 포함
		include fastcgi_params;
	}
}
```

구성 파일을 저장한 후에 엔진엑스가 이 구성을 다시 읽도록 다음 명령 중 하나를 사용한다.  

```sh
/usr/local/nginx/sbin/nginx -s reload
```

또는  

```sh
service nginx reload
```

웹 사이트의 루트 디렉터리에 간단한 스크립트를 만들어서 PHP가 올바로 실행되는지 확인하자.  

```sh
echo "<?php phpinfo(); ?>" > /home/website/www/index.php
```

사용하는 어느 웹 브라우저든 실행해서 http://localhost/(또는 웹 사이트 URL)를 읽는다. 다음 화면과 비슷한 PHP 서버 정보 페이지를 보게 될 것이다.  

파일과 디렉터리 접근 권한이 올바로 설정돼 있지 않으면 이따금 "403 접근금지" HTTP 오류가 발생한다. 이런 경우엔 php.fpm.conf 파일의 사용자와 그룹을 정확히 지정하자. 그러면 디렉터리와 파일을 PHP가 읽을 수 있다.  

#### 5.3. 파이썬과 엔진엑스
<br/>

##### 5.3.1. 장고
<br/>

##### 5.3.2. 파이썬과 장고 설정
<br/>

###### 5.3.2.1. 파이썬
<br/>

파이썬은 사용 중인 패키지 관리자 저장소에서 구할 수 있다. 레드햇 기반의 시스템이나 yum을 패키지 관리자로 사용하는 다른 시스템에서 파이썬을 설치하려면 다음 명령을 사용한다.  

```sh
yum install python3 python3-devel
```

우분투, 데비안이나 apt 또는 apititude를 사용하는 다른 시스템에서는 다음 명령을 사용한다.  

```sh
apt-get install python3 python3-dev
```

패키지 관리자가 필요한 다른 라이브러리를 알아서 설치할 것이다.  

###### 5.3.2.2. 장고
<br/>

장고(Django)를 설치할 때는 다른 방식을 사용한다(이 단계를 완전히 무시하고 통상의 저장소에서 설치할 수도 있다). 장고 프레임워크를 PHP를 사용해서 다운로드하겠다. PIP는 파이썬 패키지를 단순하게 설치하게 만들어 주는 도구다. 따라서 첫 단계는 PIP 설치다. 레드햇 기반의 시스템이나 Yum을 패키지 관리자로 사용하는 다른 시스템에서는 다음 명령을 사용한다.  

```sh
yum install python3-pip
```

우분투, 데비안이나 apt 또는 apititude를 사용하는 다른 시스템은 다음 명령을 사용한다.  

```sh
apt-get install python3-pip
```

각 패키지 관리자는 스스로 필요한 다른 패키지를 설치한다. PIP가 일단 설치되면 다음 명령을 실행해 장기 지원 안정 버전인 장고 2.2.x를 다운로드해 설치한다.  

```sh
pip3 install django~=2.2.0
```

끝으로 엔진엑스와 파이썬을 uWSGI로 연동하게 하는 데 필요한 마지막 요소가 하나 있다. uwsgi 프로토콜을 지원하는 미들웨어 uWSGI를 사용하겠다.  

```sh
pip3 install uwsgi
```

###### 5.3.2.3. uWSGI 시작
<br/>

장고 프레임워크로 웹 사이트를 구축하는 초기 작업은 아주 단순해서 다음 명령을 실행하기만 하면 된다.  

```sh
django-admin startproject mysite
```

작업이 끝나면 기본 프로젝트 템플릿에 딸려오는 manage.py라는 파이썬 스크립트가 보일 것이다. manage.py가 저장돼 있는 새로 생성된 웹 사이트 디렉터리로 이동해 다음 명령을 실행한다.  

```sh
uwsgi --socket :9000 -p -5 --module mysite.wsgi
```

여기서 --socket 옵션은 통신에 사용할 포트 번호를 지정하는 옵션이다. 엔진엑스와 uWSGI는 9000번 포트로 uwsgi 프로토콜을 사용해 데이터를 주고받는다. -p는 작업자 프로세서의 개수다. 위 명령은 가장 간단한 형태의 uWSGI 실행 방법에 속한다. 자세한 내용은 uWSGI 참조 문서(https://uwsgi-docs.readthedocs.io)를 참고하자.  

모든 것이 정확히 구성됐다면, 그리고 필요한 외부 아리브러리가 올바로 설치됐다면 각종 프로세스 정보가 화면에 표시될 것이다. 이제 uWSGI 서버가 실행돼 연결을 기다린다. ps 명령으로 애플리케이션이 실행되는지 확인할 수 있다(예를 들어 ps aux | grep uwsgi 실행), 실행되는 프로세스가 전혀 없다면 위 실행 명령어에서 포트를 살짝 다른 것으로 바꿔보자. 이제 해야 할 작업은 엔진엑스 구성 파일에 가상 호스트를 설정하는 것뿐이다.  

##### 5.3.3. 엔진엑스 구성
<br/>

엔진엑스 구성은 PHP와 비슷하다.  

```
server {
	server_name .website.com;
	listen 80;

	# 파이썬 프로젝트 공개 파일의 경로를 아래에 삽입
	root /home/website/www;

	location / {
		uwsgi_pass uwsgi://127.0.0.1:9000;
		include uwsgi_params;
	}

	location /static/ {
		root home/website/mysite/static/;
	}
}
```

### 6. 아파치와 엔진엑스 연동
<br/>

#### 6.1. 역프록시로 엔진엑스 활용
<br/>

역프록시 메커니즘을 오랫동안 사용할 최종적인 해법으로 생각해서는 안 된다는 점을 염두에 두자. 이 해법은 다음과 같은 문제 상황에서 입시 아키텍처로 이용해야 한다.  

+ 엔진엑스로 이전하기 어려울만큼 복잡한 구성의 아파치가 이미 설치돼 있거나 엔진엑스로 완전히 전환할 여유가 없을 때
+ 구버전 페러럴즈 플래스키(Parallels Flask)나 씨패널(cPanel) 또는 자동으로 아파치 구성 파일을 생성하는 다른 시스템 관리 패널을 사용해서 시스템을 운영하고 있을 때
+ 프로젝트나 아키텍처에서 요구되는 기능이 아파치에서는 가능하지만 엔진엑스로는 안 될 때  

다른 경우에는 엔진엑스로 완전히 전환하는 것이 더 낫다.  

##### 6.1.1. 문제 이해
<br/>

이 역프록시 메커니즘은 주로 아파치의 전체적인 서비스 속도라는 한 가지 문제를 다룬다. 아파치가 (수신되는 요청마다) 메모리에 적재해 놓는 다량의 모듈과 다른 구성 요소 때문에 동시에 대규모로 요청이 유입되면 서버 성능이 급격히 떨어질 것이다. 어떤 사람은 아파치가 최적화와 처리 속도를 희생하면서 기능에 초점을 맞춘다고 말할 것이다. 실제로 이 때문에 과도한 메모리와 CPU 부하가 발생한다. 반대로 엔진엑스는 다량의 요청을 처리하면서도 (아파치에 비해 RAM와 CPU를 적게 사용해서) 가볍고 안정적인 것으로 판명됐다.  

어떻게 이럴 수 있었을까? 이 질문에 답하기 전에 웹 서버가 제공하는 콘텐츠의 유형을 분석해보면 흥미로울 것이다. 무수한 사람이 매일 정기적으로 읽어 들이는 http://www.yahoo.com을 살펴보자. 야후가 전적으로 월드와이드웹을 대변하지는 않지만, 우리 분석은 다수의 웹 사이트에 유효할 것이고 야후 홈 페이지는 우리가 대변하는 문제를 완벽히 드러내는 사례다.  

일반 사용자가 http://www.yahoo.com에 방문하면 웹 브라우저는 사실 매우 많은 양의 데이터를 다운로드해야 한다.  

브라우저가 다운로드해야 하는 데이터양이 상대적으로 낮다.  

정작 문제가 되는 것은 서버가 다뤄야 하는 요청 횟수다. 모든 첫 방문자와 캐시하지 않고 이 페이지를 읽는 모든 브라우저를 위해 웹 서버는 최소한 41개의 요청을 처리하게 된다. 다행히 대부분은 파일을 캐시해 놓고 쓰지만, 웹 서버의 내용이 자주 바뀐다는 점을 명심하자. 웹 서버의 내용이 최신 정보를 제공하는 뉴스일 수도 있고 일상적인 웹 사이트 변경 때문에 내용이 바뀔 수도 있다.  

웹 서버가 HTTP 요청 41개를 1초 안에 처리할 수 있을까? 초당 1,000페이지 뷰일 때 요청 41,000건이 발생하는데, 이를 처리할 수 있을까? 410,000건은 처리할 수 있을까? 그럴 수 있다면 이미 이런 부하를 처리할 수 있는 인프라를 갖고 있는 것이다. 어느 쪽이든 엔진엑스로 처리하면 더 낫다. 이미 본 것처럼 요청 41개 중 40개는 이미지, CSS, 자바스크립트 코드 파일 같은 정적 콘텐츠다. 엔진엑스가 이런 파일을 처리하는 속도를 생각해봤을 때 엔진엑스가 정적 파일을 제공하고 아파치는 동적인 내용을 다루게 하는 아키텍처를 설계할 수 있다.  

##### 6.1.2. 역프록시 메커니즘
<br/>

엔진엑스는 외부 셰계와 직접 연결돼 소통하게 되는 반면 아파치는 그 뒤에서 엔진엑스와 데이터를 교환하는 서버 역할만 하게 된다.  

이제 요청을 처리하고자 운영되는 웹 서버가 둘이다.  

+ 엔진엑스는 앞단 서버(다시 말해 역프록시) 역할을 맡으면서 바깥 세계에서 들어오는 모든 요청을 받는다. 엔진엑스는 들어노는 요청을 선별해서 직접 정적 파일을 클라이언트에 제공하거나 동적 요청을 아파치로 전달한다.
+ 아파치는 뒷단 서버 역할을 맡고, 오직 엔진엑스하고만 통신한다. 앞단 서버와 같은 컴퓨터에서 운영될 수도 있는데, 이때 80 포트는 엔진엑스를 위해 남겨두고 사용할 수신 포트를 수정해야 한다. 또는 여러 컴퓨터에 여러 뒷단 서버를 운영해서 부하를 분산할 수도 있다.  

서로 소통하면서 상호작용하도록 어느 프로세스도 FastCGI를 사용하지 않을 것이다. 대신 이름이 뜻하는 대로 엔진엑스는 단순한 프록시 서버로 행동한다. 클라이언트에게는 HTTP 서버처럼 행동하며 HTTP 요청을 받고 뒷단의 서버에는 HTTP 클라이언트처럼 행동해서 받은 요청을 전달한다. 따라서 새로운 프로토콜이나 소프트웨어는 필요 없다. 이 메커니즘은 엔진엑스의 프록시 모듈로 처리된다.  

##### 6.1.3. 역프록시의 장단점
<br/>

서비스 속드를 높이는 것이 엔진엑스를 앞단 서버로 쓰고 아파치에게 단순한 뒷단 역할을 맡기는 구조의 주요 목적이다. 이미 언급했듯이 클라이언트에게서 들어오는 대다수의 요청은 정적 파일이고, 정적 파일은 엔진엑스가 훨씬 빨리 처리한다. 클라이언트와 서버 양쪽에서 전체 성능이 눈에 띄게 향상된다.  

여담이지만 아파치는 과거에 무척 많은 보안 문제를 겪었고, 이 때문에 새 버전을 출시했다. 웹 서버에서 보안 문제를 완전히 제거하려면 시스템을 최신 상태로 유지했어야 했다. 인기 있는 웹 서버일수록 버그와 보안 문제가 빨리 드러나서 고쳐질 것이라는 말도 일리는 있다. 하지만 엔진엑스의 안정 버전을 늘 안전한 편이었고, 심각한 보안 문제로 새 버전이 출시되는 일은 드물었다.  

결국에는 이런 방법을 받아들이면서 아파치 구성을 거의 손 볼 일이 없어서 설정이 매우 쉽다는 것을 알게 된다. 단순히 포트를 바꾸기만 하면 끝이며, 이조차 엔진엑스와 아파치를 다른 서버에 설치하기만 하면 필요 없다. 서버가 설정 그대로 동작하기 때문에 이미 아파치를 PHP, 파이썬 같은 서버 전처리기와 연동되도록 구성하는 데 여러 시간 고생했다면 이런 특성은 매우 유용하다.  

반면 동적 콘텐츠의 요청은 여전히 아파치에 넘기기 때문에 엔진엑스와 FastCGI를 조합한 경우보다 대부분 느리게 동작한다. 앞에서 언급했듯이 엔진엑스로 완전히 전환하고 아파치를 떼어내는 것이 최적의 해법일 것이다.  

게다가 엔진엑스가 앞단에 설치된 이상 엔진엑스는 사용자가 보내는 요청을 가공없이 그대로 받는다. 이는 요청 URI가 원형대로 들어온다는 뜻으로, 정적과 동적 콘텐츠를 구분할 수 없어서 엔진엑스가 혼란을 겪게 된다. 이 문제를 해결할 방법은 두 가지다. 아파치의 재작성 규칙(rewrite rule)을 엔진엑스에 이식할 수도 있고, 결과가 404 오류인 모든 요청을 뒷단의 아파치로 재전송할 수 있다.  

이 문제를 예시로 설명하고자 /articles/43515-us-economy-strengthens.html 같은 요청 URI이 들어왔는데, 시스템의 어느 파일과도 맞지 않아 보인다고 하자. 이는 URI을 재작성해서 /article.php?id=43515와 유사한 요청으로 바꿔야 한다는 뜻이다. 첫 번째 해결방법은 엔진엑스 구성에 재작성 규칙을 직접 포함시키는 것이다. 또 다른 방법은 파일이 존재하는지 확인해보고 없다면 요청을 아파치로 재전송하는 것이다.  

마지막으로 중요한 문제가 하나 더 있다. 이 문제는 다시 다룰 것인데, 페러럴즈 플레스크(Parallels Flask)이나 cPanel 같은 제어 패널 소프트웨어와 문제가 좀 있을 수 있다. 이 패널은 아파치 구성에 가상 호스트를 추가하거나 이메일 계정을 생성하거나 DNS 데몬을 구성하거나 하는 다양한 귀찮은 작업을 일부 자동화해주기 때문에 서버 관리자에게 매우 유용하다.  

+ 이런 제어 패널은 변경 사항을 웹 서버 구성에 적용하며, 변경 내용에 기반을 두고 자동으로 유효한 서버 구성 파일을 생성한다. 불행히 이 제어 패널들은 많은 경우 아파치와만 호환돼 엔진엑스 구성 파일은 생성하지 않는다. 따라서 제어 패널로 변경을 해도 엔진엑스에는 아무런 영향이 없다.
+ 아파치를 엔진엑스로 완전히 대체하든, 역프록시 메커니즘으로 가든, 엔진엑스는 결국 80 포트(HTTPS로는 443 포트)로 보통 운영된다. 구성 파일을 생성하는 제어 패널 소프트웨어는 이런 사실을 모를 뿐 아니라 고집스러워서 구성 파일을 생성하면서 아파치 포트를 80으로 되돌려 놓는데, 이 때문에 엔진엑스와 충돌을 일으키게 된다.  

#### 6.2. 엔진엑스 프록시 모듈
<br/>

엔진엑스 기본 빌드에는 클라이언트에서 들어오는 HTTP 요청을 뒷단 서버로 전송할 수 있는 프록시 모듈이 포함돼 있다. 이 모듈의 다양한 측면을 구성해보겠다.  

+ 뒷단 서버의 기본 주소와 포트 정보
+ 캐시, 버퍼링, 임시 파일 옵션
+ 한계치, 시간 제약, 오류 처리
+ 기타 옵션  

##### 6.2.1. 주요 지시어
<br/>

첫 번째 지시어는 뒷단 서버의 위치나 전달할 정보 및 전달 방법 등을 지정하는 기본 구성에 쓰인다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				proxy&#95;pass<br/>
				맥락: location, if
			</td>
			<td>
				위치를 알려서 요청이 뒷단 서버로 전달되도록 지정한다.<br/>
				일반 HTTP 전달은 구문이 proxy&#95;pass http://호스트명:포트다.<br/>
				유닉스 도메인 소켓은 구문이 proxy&#95;pass http:// unix:/path/to/file.socket;이다.<br/>
				upstream 블록을 참조할 수도 있다. proxy&#95;pass http://myblock; http:// 대신 https://를 사용해서 안전하게 통신하게 할 수 있다. URI의 나머지 부분이나 변수도 사용할 수 있다.<br/>
				예:<br/>
				```
				proxy_pass http://localhost:8080;
				proxy_pass http://127.0.0.1:8080;
				proxy_pass http://unix:/tmp/nginx.sock;
				proxy_pass https://192.168.0.1;
				proxy_pass http://localhost:8080/uri/;
				proxy_pass http://unix:/tmp/nginx.sock:/uri/;
				proxy_pass http://$server_name:8080;
				# Using an upstream
				upstream backend
				{
					server 127.0.0.1:8080;
					server 127.0.0.1:8081;
				}
				location ~* \.php$
				{
					proxy_pass http://backend;
				}
				```
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;method<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단 서버로 전달될 요청의 HTTP 메서드를 재정의한다. 예를 들어 POST로 지정하면 뒷단으로 전달되는 모든 요청은 POST 요청이 된다. 변수를 사용할 수 있다.<br/>
				구문: proxy&#95;method method;<br/>
				예: proxy&#95;method POST;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;hide&#95;header<br/>
				맥락: http, server, location
			</td>
			<td>
				기본적으로 엔진엑스는 뒷단 서버에서 받아 클라이언트에게 돌려 줄 응답을 준비하기 때문에 Date, Server, X-Pad, X-Accel-&#42; 같은 몇 가지 헤더를 무시한다. 이 지시어로 클라이언트로 전달하지 않고 숨길 헤더를 추가로 지정할 수 있다. 이 지시어를 한 번에 헤더 이름 하나씩 여러 번 추가해도 된다.<br/>
				구문: proxy&#95;hide&#95;header header&#95;name;<br/>
				예: proxy&#95;hide&#95;header Cache-Control;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;pass&#95;header<br/>
				맥락: http, server, location
			</td>
			<td>
				proxy&#95;hide&#95;header와 반대로 이 지시어는 무시되는 헤더를 강제로 클라이언트에 전달되게 한다.<br/>
				구문: proxy&#95;pass&#95;header headername;<br/>
				예: proxy&#95;pass&#95;header Date;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;pass&#95;request&#95;bodyproxy&#95;pass&#95;request&#95;headers<br/>
				맥락: http, server, location
			</td>
			<td>
				요청 본문과 부가 요청 헤더가 뒷단 서버로 전달될지 여부를 정한다.<br/>
				구문: on이나 off<br/>
				기본값: on
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;redirect<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단 서버에서 유발된 재설정의 Location HTTP 헤더 속 URL을 재작성한다.<br/>
				구문: off, default, 또는 선택된 URL<br/>
				off: proxy&#95;pass 지시어의 값을 호스트 이름으로 사용하고 현재 경로의 문서를 추가한다. 구성 파일이 순차적으로 해석되기 때문에 proxy&#95;redirect 지시어는 반드시 proxy&#95;pass 지시어 다음에 들어가야 한다.<br/>
				URL: URL의 일부를 다른 값으로 대체한다. 게다가 재작성된 URL에 변수를 사용해도 된다.<br/>
				예:<br/>
				proxy&#95;redirect off;<br/>
				proxy&#95;redirect default;<br/>
				proxy&#95;redirect http://localhost:8080/ http://example.com/;<br/>
				proxy&#95;redirect http://localhost:8080/wiki/ /w/;<br/>
				proxy&#95;redirect http://localhost:8080/ http://$host/;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;next&#95;upstream<br/>
				맥락: http, server, location
			</td>
			<td>
				proxy&#95;pass가 upstream 블록과 연결됐을 때 이 지시어는 요청을 버리고 블록의 다음 업스트림 서버로 재전송하는 경우를 정의한다. 이 지시어는 다음 값을 조합을 허용한다.<br/>
				+ error: 서버와 통신 중이거나 통신을 시도하는 중에 오류가 발생함
				+ timeout: 전송하거나 연결 시도 중에 제한시간을 넘음
				+ invalid&#95;header: 뒷단의 서버가 빈 결과나 유효하지 않은 응답을 반환함
				+ http&#95;500, http&#95;502, http&#95;503, http&#95;403, http&#95;404, http&#95;429: HTTP 오류가 발생한 경우 엔진엑스는 다음 업스트림 서버로 전환한다.
				+ non&#95;idempotent: 보통 요청이 뒷단 서버로 전송된 후에는 POST, LOCK, PATCH 같은 비멱등 메서드일 때 다음 서버로 넘기지 않는다. 이 옵션은 재전송하게 만든다.
				+ off: 다음 업스트림 서버 사용 금지<br/>
				예:<br/>
				proxy&#95;next&#95;upstream error timeout http&#95;504;<br/>
				proxy&#95;next&#95;upstream timeout invalid&#95;header;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;next&#95;upstream&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				proxy&#95;next&#95;upstream과 결합해서 사용될 제한시간을 정한다.<br/>
				구문: 시간 값(초 단위)
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;next&#95;upstream&#95;tries<br/>
				맥락: http, server, location
			</td>
			<td>
				오류 메시지를 반환하기 전에 시도할 최대 업스트림 서버 수를 정한다. proxy&#95;next&#95;upstream과 결합해서 사용된다.<br/>
				구문: 숫자 값(기본값: 0)
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 6.2.2. 캐시, 버퍼링, 임시 파일
<br/>

이상적으로는 가능한 한 뒷단 서버로 전달되는 요청의 수를 줄여야 한다. 다음 지시어는 캐시 시스템을 구축할 때는 물론 버퍼링 제어 옵션과 엔진엑스가 임시 파일을 다루는 방법을 제어하는 데 도움이 된다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				proxy&#95;buffer&#95;size<br/>
				맥락: http, server, location
			</td>
			<td>
				둣단 서버의 응답 시작 부분을 읽는 데 사용할 버퍼의 크기를 설정한다. 이 버퍼는 보통 단순한 헤더 데이터를 담는다.<br/>
				기본값은 앞서 주어진 지시어(proxy&#95;bufers)에서 정한 대로 버퍼 하나의 크기에 해당한다.<br/>
				구문: 숫자 값(크기)<br/>
				예: proxy&#95;buffer&#95;size 4k;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;buffering, proxy&#95;reqeust&#95;buffering<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단 서버에서 오는 응답(proxy&#95;request&#95;buffering의 경우는 클라이언트 요청)을 버퍼에 담응맂 여부를 정의한다. on으로 설정하면 엔진엑스는 버퍼가 제공하는 메모리 공간을 사용해서 응답 데이터를 메모리에 저장할 것이다. 버퍼가 가득차면 응답 데이터는 임시 파일로 저장될 것이다. 이 지시어가 off로 설정되면 응답은 그대로 클라이언트에게 전달된다.<br/>
				구문: on이나 off<br/>
				기본값: on
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;buffers<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단 서버에서 응답 데이터를 읽는 데 사용될 버퍼의 개수와 크기를 설정한다.<br/>
				구문: proxy&#95;buffers 개수 크기;<br/>
				기본값: 버퍼 8개, 플랫폼에 따라 4k 또는 8k<br/>
				예: proxy&#95;buffers 8 4k;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;busy&#95;buffers&#95;size<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단에서 받은 데이터가 버퍼에 쌓이다가 지정한 값을 넘으면 그 버퍼는 비워지고 데이터는 클라이언트에 보내진다.<br/>
				구문: 숫자 값(크기)<br/>
				기본값: 2 &#42; proxy&#42;buffer&#42;size
			</td>
		</tr>
		<tr>
			<td>
				proxy&#42;cache<br/>
				맥락: http, server, location
			</td>
			<td>
				캐시 구역(cache zone)을 정의한다. 구역에 주어지는 ID는 향후 지시어에서 재사용된다.<br/>
				구문: proxy&#95;cache 영역&#95;이름;<br/>
				예: proxy&#95;cache cache1;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;cache&#95;key<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 캐시 키를 정의한다. 이 키는 캐시에 등록되는 데이터를 다른 데이터와 구분되게 한다. 캐시 키가 $url로 설정되면 결과적으로 이 $url로 들어오는 모든 요청은 한 캐시 데이터로 동작한다. 하지만 이런 방식은 대부분 동적 웹 사이트에서 충분하지 않다. 캐시 키에 질의 문자열 인자도 포함해야만 /index.php와 /index.php?page=contact가 같은 캐시 데이터를 가리키지 않게 된다.<br/>
				구문: proxy&#95;cache&#95;key key;<br/>
				예: proxy&#95;cache&#95;key "$scheme$host$request&#95; uri $cookie&#95;user";
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;cache&#95;path<br/>
				맥락: http
			</td>
			<td>
				캐시 파일을 저장할 디렉터리와 함께 다른 매개변수를 지정한다.<br/>
				구문: proxy&#95;cache&#95;path 경로 [use&#95;temp&#95;path=on|off] [levels=숫자들] keys&#95;zone=이름:크기 [inactive=시간] [max&#95;size=크기] [manager&#95;files=숫자] [manager&#95;sleep=시간] [manager&#95;threshold=시간] [loader&#95;files=숫자] [loader&#95;sleep=숫자] [loader&#95;threshold=숫자]<br/>
				부가 매개변수는 다음과 같다.<br/>
				+ use&#95;temp&#95;path: proxy&#95;temp&#95;path 지시어로 정한 경로를 사용하고 싶다면 이 플래그를 on으로 설정한다.
				+ levels: 하위 디렉터리의 깊이를 표현한다(보통 1:2로 충분함).
				+ keys&#95;zone: proxy&#95;cache 지시어로 앞서 선언된 구역을 사용하게 하고 메모리상에 점유할 크기를 표현한다.
				+ inactive: 캐시된 응답이 지정된 시간 범위 안에 사용되지 않으면 캐시에서 삭제된다.
				+ max&#95;size: 전체 캐시의 최대 크기를 정한다.
				+ manager&#95;files, manager&#95;sleep, manager&#95;threshold: 사용한 지 오래된 캐시를 삭제하는 캐시 관리자를 구성한다. 한 주기 안에 삭제할 파일의 수(manager&#95;files, 기본값: 100 파일), 삭제 주기 사이의 중단 시간(nanager&#95;sleep, 기본 50ms), 삭제 주기의 최대 작업 시간(manager&#95;threshold, 기본값: 200ms)
				+ loader&#95;files, loader&#95;sleep, loader&#95;threshold: 파일 시스템에 보관된 캐시를 읽는 캐시 로더(cache loader)를 구성한다. 한 주기 안에 처리할 파일의 수(loader&#95;files, 기본값: 100 파일), 읽는 주기 사이의 중단 시간(loader&#95;sleep, 기본 50ms), 읽는 주기의 최대 작업 시간(loader&#95;threshold, 기본값: 200ms)<br/>
				예:<br/>
				proxy&#95;cache&#95;path /tmp/nginx&#95;cache levels=1:2 keys&#95;zone=zone1:10m inactive=10m max&#95;size=200M;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;cache&#95;methods<br/>
				맥락: http, server, location
			</td>
			<td>
				캐시에 맞는 HTTP 메서드를 정한다. GET와 HEAD는 기본적으로 포함이며 제외할 수 있다.<br/>
				구문: proxy&#95;cache&#95;methods 메서드;<br/>
				예: proxy&#95;cache&#95;methods OPTIONS;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;cache&#95;convert&#95;haed<br/>
				맥락: http, server, location
			</td>
			<td>
				on이면 캐시되도록 HEAD 메서드를 GET으로 바꾼다. off일 경우 cache&#95;key가 $request&#95;method를 포함해야 한다.<br/>
				구문: proxy&#95;cache&#95;convert&#95;head on|off;<br/>
				예: proxy&#95;cache&#95;convert&#95;head on;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;cache&#95;min&#95;uses<br/>
				맥락: http, server, location
			</td>
			<td>
				요청이 캐시되기에 알맞다고 판단되기 전에 적중될 최소 수치를 정한다. 기본적으로 요청의 응답은 한 번 적중된 후에 캐시가 된다.(이어서 같은 캐시 키로 들어오는 요청은 캐시된 응답을 받게 된다).<br/>
				구문: 숫자 값<br/>
				예: proxy&#95;cache&#95;min&#95;uses 1;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;cache&#95;valid<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 서로 응답 코드 유형별로 캐시되는 시간을 바꿀 수 있도록 허용한다. "404" 오류 코드와 관련된 응답은 1분간 캐시되고, 반대로 "200 OK" 응답은 10분이나 그 이상 캐시되게 할 수 있다. 이 지시어는 한 번 이상 사용할 수 있다.<br/>
				proxy&#95;cache&#95;valid 404 1m;<br/>
				proxy&#95;cache&#95;valid 500 502 504 5m;<br/>
				proxy&#95;cache&#95;valid 200 10;<br/>
				구문: proxy&#95;cache&#95;valid 코드1 [코드2...] 시간;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;cache&#95;use&#95;stale<br/>
				맥락: http, server, location
			</td>
			<td>
				엔진엑스가 (게이트웨이와 관련된) 특정 상황에 인가된 캐시 데이터를 사용할지 여부를 정한다. proxy&#95;cache&#95;use&#95;stale timeout을 사용했는데 게이트웨이가 그 시간을 넘으면 엔진엑스는 캐시된 데이터를 제공한다.<br/>
				Cache-Control 헤더에 stale-while-revalidate 속성이 있으면 업데이트 중에 인가된 캐시 응답이 사용된다. Cache-Control 헤더에 stale-if-error 속성이 있으면 오류가 발생했을 때 만기된 캐시 응답이 사용된다.<br/>
				구문: proxy&#95;cache&#95;use&#95;stale [updating] [error] [timeout] [invalid&#95;header] [http&#95;500] [http&#95;502] [http&#95;503] [http&#95;504] [http&#95;403] [http&#95;404] [http&#95;429] [off];<br/>
				예: proxy&#95;cache&#95;use&#95;stale error timeout updating;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;cache&#95;background&#95;update<br/>
				맥락: http, server, location
			</td>
			<td>
				on으로 설정하면 만기된 캐시 데이터가 제공되는 동안 배후에서 부가 요청을 보내서 만기된 캐시 항목을 갱신한다. proxy&#95;cache&#95;use&#95;stale에 updating이 설정돼 있어야 한다.<br/>
				구문: proxy&#95;cache&#95;background&#95;update on|off;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;max&#95;temp&#95;file&#95;size<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어를 0으로 설정하면 프록시 전달에 적합한 요청에 임시 파일을 사용하지 않게 된다. 임시 파일을 쓰고 싶다면 최대 파일 크기를 설정한다.<br/>
				구문: 크기 값<br/>
				기본값: 1GB<br/>
				예: proxy&#95;max&#95;temp&#95;file&#95;size 5m;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;temp&#95;file&#95;wrtie&#95;size<br/>
				구문: http, server, location
			</td>
			<td>
				저장 장치에 임시 파일을 저장할 때 쓰기 작업용 퍼버의 크기를 설정한다.<br/>
				구문: 크기 값<br/>
				기본값: 2 &#42; proxy&#95;buffer&#95;size
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;temp&#95;path<br/>
				맥락: http, server, location
			</td>
			<td>
				임시 파일과 캐시 저장 파일의 경로를 설정한다.<br/>
				구문: proxy&#95;temp&#95;path 경로 [수준1 [수준2...]]<br/>
				예:<br/>
				proxy&#95;temp&#95;path /tmp/nginx&#95;proxy;<br/>
				proxy&#95;temp&#95;path /tmp/cache 1 2;
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 6.2.3. 한계치, 시간 제약, 오류
<br/>

다음에 나열되는 지시어는 시간 제약과 관련된 행동은 물론 뒷단 서버와 통신하는 상황과 관련된 다양한 한계치를 정하는 데 도움이 된다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				proxy&#95;connect&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단 서버에 연결하는 제한시간을 정한다. 이 값은 읽기/쓰기 제한시간과는 다르다. 엔진엑스가 뒷단 서버에 이미 연결이 돼 있다면 proxy&#95;connect&#95;timeout은 적용이 안 된다.<br/>
				구문: 시간 값(초 단위)<br/>
				예: proxy&#95;connect&#95;timeout 15;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;read&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단 서버에서 데이터를 읽는 제한시간을 정한다. 이 시간 제약은 전체 응답 지연이 아닌 읽은 두 작업 사이에 적용된다.<br/>
				구문: 시간 값(초 단위)<br/>
				기본값: 60<br/>
				예: proxy&#95;read&#95;timeout 60;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;send&#95;timeout<br/>
				맥락: http, server, location
			</td>
			<td>
				이 시간 제약은 데이터를 뒷단 서버에 데이터를 보내는 용도다. 이 시간 제약은 전체 응답 지연이 아닌 쓰는 두 동작 사이에 적용된다.<br/>
				구문: 시간 값(초 단위)<br/>
				기본값: 60<br/>
				예: proxy&#95;send&#95;timeout 60;
			</td>>
		</tr>
		<tr>
			<td>
				proxy&#95;ignore&#95;client&#95;abort<br/>
				맥락: http, server, location
			</td>
			<td>
				on으로 설정하면 클라이언트가 요청을 취소했더라도 엔진엑스는 프록시 요청을 계속 처리한다. 반면에 off인 경우에는 엔진엑스도 뒷단 서버로 보내는 요청을 취소한다.<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;intercept&#95;errors<br/>
				맥락: http, server, location
			</td>
			<td>
				기본적으로 엔진엑스는 뒷단 서버에서 보내는 모든 (HTTP 상태 코드 400 이상의) 오류 페이지를 그대로 클라이언트에 반환한다. 이 지시어를 on으로 설정하면 오류 코드를 해석해서 error&#95;page 지시어에 지정된 값과 일치하는지 비교할 수 있다.<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;send&#95;lowat<br/>
				맥락: http, server, location
			</td>
			<td>
				BSD 기반 운영체제라면 TCP 소켓의 SO&#95;SNDLOWAT 플래그를 쓰게 할 수 있는 옵션이다. 이 값은 출력 작업용 버퍼의 최소 바이트 수를 정한다.<br/>
				구문: 숫자 값(크기)<br/>
				기본값: 0
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;limit&#95;rate<br/>
				맥락: http, server, location
			</td>
			<td>
				엔진엑스가 뒷단 프록시에서 응답을 다운로드하는 속도를 제한한다.<br/>
				구문: 숫자 값(초당 바이트)
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 6.2.4. SSL 관련 지시어
<br/>

SSL로 뒷단 서버와 운영하려면 다음 지시어가 유용할 것이다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				proxy&#95;ssl&#95;certificate<br/>
				맥락: http, server, location
			</td>
			<td>
				SSL 서버의 인증서를 담고 있는 PEM 파일의 경로를 설정한다.<br/>
				구문: 파일 경로<br/>
				기본값: 없음
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;certificate&#95;key<br/>
				맥락: http, server, location
			</td>
			<td>
				SSL 서버의 인정에 쓰는 (PEM 형식의) 보안 키 파일의 경로를 설정한다.<br/>
				구문: 파일 경로<br/>
				기본값: 없음
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;ciphers<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단 서버와 SSL 통신을 하는 용도의 암호 알고리즘을 설정한다. 아래 셸 명령을 실행시키면 사용할 수 있는 서버의 암호 알고리즘 목록을 얻을 수 있다.<br/>
				```sh
				openssl ciphers
				```
				구문: cipher 이름<br/>
				기본값: DEFAULT
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;crl<br/>
				맥락: http, server, location
			</td>
			<td>
				PEM 형식의 인증서 폐기 목록(CRL, Certificate Revocation List) 파일 경로를 설정한다. CRL은 엔진엑스가 뒷단 서버 SSL 인증서의 폐기 상태를 검증하는 데 쓰인다.<br/>
				구문: 파일 경로<br/>
				기본값: -
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;name<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 엔진엑스가 뒷단 서버 SSL 인증서의 폐기 상태를 검증할 때 사용할 서버 이름을 직접 지정한다.<br/>
				구문: 문자열<br/>
				기본값: $proxy&#95;host 값과 동일
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;password&#95;file<br/>
				맥락: http, server, location
			</td>
			<td>
				인증 키를 읽을 때 하나씩 적용해 볼 비밀번호를 (한 줄에 하나씩) 보관하는 파일의 경로를 설정한다.<br/>
				구문: 파일 경로<br/>
				기본값: -
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;server&#95;name<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어를 on으로 설정하면(기본값은 off) 서버 이름이 서버명 표시(SNI, Server Name Indication) 프로토콜에 따라 뒷단 서버에 알려진다.<br/>
				구문: on이나 off<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;session&#95;reuse<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 엔진엑스에게 뒷단 서버와 통신할 때 기존 SSL 세션을 재사용하도록(따라서 부하를 줄이도록) 지시한다. 공식 문서에서는 서버 로그에 SSL3&#95;GET&#95;FINISHED:digest check failed 오류가 보이기 시작하면 이 기능을 비활성하도록 권한다.<br/>
				구문: on이나 off<br/>
				기본값: on
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;protocols<br/>
				맥락: http, server, location
			</td>
			<td>
				SSL 뒷단 서버와 통신할 때 사용할 프로토콜을 설정한다.<br/>
				구문: proxy&#95;ssl&#95;protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2];<br/>
				기본값: TLSv1 TLSv1.1 TLSv1.2
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;trusted<br/>
				맥락: http, server, location
			</td>
			<td>
				신뢰할 수 있는 인증기관 (PEM 형식) 인증서의 경로를 설정한다.<br/>
				구문: 파일 경로<br/>
				기본값: -
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;verify<br/>
				맥락: http, server, location
			</td>
			<td>
				on으로 설정하면 엔진엑스는 SSL 뒷단 서버의 인증서를 검증한다.<br/>
				구문: on이나 off<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ssl&#95;verify&#95;depth<br/>
				맥락: http, server, location
			</td>
			<td>
				proxy&#95;ssl&#95;verify 지시어가 on으로 설정되면 이 지시어는 인증서 체인의 검증 길이를 설정한다.<br/>
				구문: 숫자 값<br/>
				기본값: 1
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 6.2.5. 기타 지시어
<br/>

분류되지 않은 프록시 모듈의 나머지 지시어는 다음과 같다.  

<table>
	<thead>
		<tr>
			<th>지시어</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				proxy&#95;headers&#95;hash&#95;max&#95;size<br/>
				맥락: http, server, location
			</td>
			<td>
				프록시 헤더의 해시 테이블 최대 크기를 설정한다.<br/>
				구문: 숫자 값<br/>
				기본값: 512
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;headers&#95;hash&#95;bucket&#95;size<br/>
				맥락: http, server, location
			</td>
			<td>
				프록시 헤더의 해시 테이블용 버킷 크기를 설정한다.<br/>
				구문: 숫자 값<br/>
				기본값: 64
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;force&#95;ranges<br/>
				맥락: http, server, location
			</td>
			<td>
				on으로 설정하면 엔진엑스는 뒷단 서버에서 오는 응답의 바이트 범위(byte-range) 지원 기능을 활성화한다.<br/>
				구문: on이나 off<br/>
				기본값: off
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;ignore&#95;headers<br/>
				맥락: http, server, location
			</td>
			<td>
				엔진엑스가 뒷단 서버 응답에서 X-Accel-Redirect, X-Accel-Expires, Expires, Cache-Control 등 네 가지 헤더를 처리하지 못하게 막는다.<br/>
				구문: proxy&#95;ignore&#95;headers header1 [header2...];<br/>
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;set&#95;body<br/>
				맥락: http, server, location
			</td>
			<td>
				디비깅 목적에 쓸 정적인 요청 본문을 설정할 수 있다. 이 지시어의 값으로 변수가 사용될 것이다.<br/>
				구문: 문자열 값(임의의 값)<br/>
				기본값: proxy&#95;set&#95;body test;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;set&#95;header<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단 서버로 전송될 헤더 값을 다시 정의할 수 있게 해주는 지시어다.<br/>
				구문: proxy&#95;set&#95;header 헤더 값;<br/>
				기본값: proxy&#95;set&#95;header Host $host;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;store<br/>
				맥락: http, server, location
			</td>
			<td>
				뒷단 서버 응답이 파일로 저장돼야 할지 여부를 지정한다. 저장된 응답은 다른 요청에 재사용될 수 있다.<br/>
				가능한 값: on, off 또는 최상위 문서 위치(또는 별명)에 대한 상태 경로. 이 지시어를 on으로 설정하고 proxy&#95;temp&#95;path 지시어를 정의해도 된다.
				예: <br/>
				proxy&#95;store on;<br/>
				proxy&#95;temp&#95;path /temp/store;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;store&#95;access<br/>
				맥락: http, server, location
			</td>
			<td>
				이 지시어는 응답을 저장한 파일의 파일 접근 권한을 정한다.<br/>
				구문: proxy&#95;store&#95;access [user:[r|w|rw]] [group:[r|w|rw]] [all:[r|w|rw]];<br/>
				예: proxy&#95;store&#95;access user:rw group:rw all:r;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;http&#95;version<br/>
				맥락: http, server, location
			</td>
			<td>
				프록시 뒷단과 통신하는 데 쓰일 HTTP 버전을 설정한다. HTTP 1.0이 기본값이지만 연결을 유지해서 재사용하려면 이 지시어를 1.1로 설정해야 한다.<br/>
				구문: proxy&#95;http&#95;version 1.0 | 1.1;<br/>
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;cookie&#95;domain, proxy&#95;cookie&#95;path<br/>
				맥락: http, server, location
			</td>
			<td>
				쿠키의 도메인이나 경로 속성을 실시간으로 조작하게 한다. 대소문자를 따지지 않는다.<br/>
				구문:<br/>
				proxy&#95;cookie&#95;domain off | 도메인 대체&#95;문자열;<br/>
				proxy&#95;cookie&#95;path off | 도메인 경로;
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;socket&#95;keepalive<br/>
				맥락: http, server, location
			</td>
			<td>
				프록시 서버와의 연결이 유지되도록 TCP 소켓을 구성한다. 기본적으로는 운영체제의 소켓 설정이 그대로 반영된다. 이 지시어의 값이 on이면 SO&#95;KEEPALIVE 소켓 옵션이 사용하는 소켓에 활성화된다.<br/>
				구문: proxy&#95;socket&#95;keepalive on | off;<br/>
				기본값: off<br/>
				예: proxy&#95;socket&#95;keepalive on;
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

##### 6.2.6. 변수
<br/>

프록시 모듈은 다양한 위치, 예를 들어 proxy&#95;set&#95;header 지시어나 log&#95;format 같은 로그 관련 지시어에 삽입할 수 있는 여러 변수를 제공한다. 사용 가능한 변수는 다음과 같다.  

+ $proxy&#95;host: 현 요청에 사용되는 뒷단 서버의 호스트명을 가진다.
+ $proxy&#95;port: 현 요청에 사용되는 뒷단 서버의 포트 번호를 가진다.
+ $proxy&#95;add&#95;x&#95;forwarded&#95;for: 이 변수는 X-Forwarded-For 요청 헤더 값과 클라이언트의 원격 주소를 가진다. 두 값은 콤마(,)로 구분된다. X-Forwarded-For 요청 헤더가 없다면 이 변수는 클라이언트 원격 주소만 가진다.
+ $proxy&#95;internal&#95;body&#95;length: (proxy&#95;set&#95;body 지시어로 설정이 된) 요청 본문의 길이다. 설정이 안 됏으면 0이다.  

#### 6.3. 아파치와 엔진엑스 구성
<br/>

##### 6.3.1. 아파치 재구성
<br/>

###### 6.3.1.1. 구성 개요
<br/>

###### 6.3.1.2. 포트 번호 재설정
<br/>

(수작업으로 구축했는지, 씨패널(cPanel)이나 플레스크(Plesk) 같은 서버 패널 관리 도구로 자동으로 구성했는지) 웹 서버가 구성된 방식에 따라 다수의 구성 파일을 수정해야 할 수도 있다. 주요 구성 파일은 주로 /etc/httpd/conf/나 /etc/apache2/에 저장되지만 구성 파일이 구조화된 방식에 따라 더 많을 수도 있다. 일부 서버 패널 관리 도구는 가상 호스트마다 별도 구성 파일을 생성한다.  

아파치 구성 파일에서 교체해야 할 구성 요소는 다음과 같은 세 가지다.  

+ Listen 지시어는 기본으로 80 포트에서 수신하도록 설정된다. 이 포트 번호를 8080 같은 다른 번호로 교체해야 한다. 이 지시어는 보통 주 구성 파일에 있지만 어떤 설정에서는 ports.conf 같은 별도 파일에 들어 있다.
+ 다음 구성 지시어는 주 구성 파일에 반드시 있어야 한다. 다음 코드에서 A.B.C.D는 서버 통신에 사용되는 주 네트워크 인터페이스의 IP 주소다.  

```
NameVirtualHost A.B.C.D:8080
```

+ 변경하기로 선택한 포트는 모든 가상 호스트 구성 부분에 반영돼야 한다.  

가상 호스트 구성 부분은 다음과 같은 식으로 바뀌어야 한다.  

+ 변경 전:  
```
<VirtualHost A.B.C.D:80>
	ServerName	example.com
	ServerAlias	www.example.com
	[...]
</VirtualHost>
```

+ 변경 후:  
```
<VirtualHost A.B.C.D:8080>
	ServerName	example.com:8080
	ServerAlias	www.example.com
	[...]
</VirtualHost>
```

이 예에서 A.B.C.D는 가상 호스트의 IP 주소이고 example.com은 가상 호스트의 이름이다.  

###### 6.3.1.3. 로컬 요청 한정 수신
<br/>

아파치가 로컬 요청만 받고 외부에서 들어오는 접근은 거부하게 막는 방법은 다양하다. 하지만 가장 먼저 이렇게 하는 이유를 알아야 한다. 클라이언트와 아파치 사이에 추가 계층이 자리 잡은 이상 엔진엑스는 보안 관점에서 어느 정도 안정성을 제공한다. 방문자는 이제 아파치에 직접 접속하지 않기 때문에 아무도 앞으로 밝혀질지 모를 취약점을 공격할 수 없어 잠재적인 위험이 감소한다. 흔히 말하는 최소 권한 원칙(principle of least privilege)이 적용돼야 한다.  

첫 번째 방법은 주 구성 파일에서 수신 네트워크 인터페이스를 바꾸는 것으로 이뤄진다. 아파치의 Listen 지시어로 포트뿐 아니라 IP 주소도 지정할 수 있다. 하지만 기본적으로 아무 IP 주소도 지정하지 않고 결국 통신이 모든 인터페이스를 통해 일어난다. 해야 할 일이라고는 Listen 8080 지시어를 Listen 127.0.0.1:8080으로 교체하는 것뿐이다. 이렇게 하면 아파치는 로컬 IP 주소로만 수신하게 된다. 같은 서버에서 아파치를 운영하지 않을 경우에는 엔진엑스를 운영하는 서버와 통신할 수 있는 네트워크 인터페이스의 주소를 지정해야 한다.  

다른 대안은 가상 호스트마다 다음과 같이 제한을 추가하는 것이다.  

```
<VirtualHost A.B.C.D:8080>
	ServerName	example.com:8080
	ServerAlias	www.example.com
	[...]
	order deny,allow
	allow from 127.0.0.1
	allow from 192.168.0.1
	deny all
</VirtualHost>
```

allow와 deny 아파치 지시어를 사용하면 특정 가상 호스트에 접근하는 IP 주소를 제한할 수 있다. 이 방법으로 세밀한 구성이 가능하므로, 웹 사이트의 일부를 엔진엑스로 충분히 제공하지 못할 경우에 유용하다.  

수정이 완료되면 service httpd reload나 /etc/init.d/httpd reload 명령으로 서버가 새 구성 정보를 다시 읽어서 적용되게 해야 한다.  

##### 6.3.2. 엔진엑스 구성
<br/>

###### 6.3.2.1. 프록시 옵션 활성화
<br/>

첫 단계에는 location 블록에서 요청을 프록시로 처리하게 활성화한다. proxy&#95;pass 지시어는 http나 server 수준에서는 쓰지 못하기 때문에 뒷단으로 전달하고 싶은 location 블록마다 매번 추가해줘야 한다. 보통 break 구문을 포함하는 location 블록과 일치하지 않는 한 모든 요청을 망라하는 location / &#40; 블록에 두는 것으로 충분하다.  

다음은 동일한 서버에서 정적인 뒷단 서버를 하나 사용하는 단순한 예다.  

```
server {
	server_name .example.com;
	root /home/example.com/www;
	[...]
	location / {
		proxy_pass http://127.0.0.1:8080;
	}
}
```

다음은 upstream 블록을 사용해 다수의 서버를 지정하는 예다.  

```
upstream apache`{
	server 192.168.0.1:80;
	server 192.168.0.2:80;
	server 192.168.0.3:80 weight=2;
	server 192.168.0.4:80 backup;
}

server {
	server_name .example.com;
	root /home/example.com/www;
	[...]
	location / {
		proxy_pass http://apache;
	}
}
```

지금까지 구성한 결과로 모든 요청이 프록시에 의해 뒷단 서버로 전달된다. 이제 내용을 두 가지로 구분하려고 한다.  

+ 동적 파일: 파일이 클라이언트로 보내지기 전에 처리돼야 한다. PHP, 펄(perl), 루비(Ruby) 스크립트는 아파치가 제공할 것이다.
+ 정적 파일: 다른 내용은 모두 추가적인 처리가 필요하지 않은 이미지, CSS 파일, 정적 HTML 파일, 기타 미디어 파일로 직접 엔진엑스가 제공할 것이다.  

따라서 두 서버에 나눠 제공되도록 정적인 내용에서 동적인 부분을 분리할 방법이 필요하다.  

###### 6.3.2.2. 내용 분리
<br/>

단순하게 location 블록 두 개를 사용해서 하나는 동적 파일 확장자에 대응되게 하고 다른 하나는 기타 파일을 모두 다루게 하면 계획한 내용을 분리할 수 있다. 이 예는 .php 파일의 요청을 프록시에 전달한다.  

```
server {
	server_name .example.com;
	root /home/example.com/www;
	[...]
	location / {
		proxy_pass http://127.0.0.1:8080;
	}
}
[...]
location ~* \.php.$ {
	# URI가 .php로 끝나는 모든 요청을 프록시로 처리
	# (PHP, PHP3, PHP4, PHP5 등 포함)
	proxy_pass http://127.0.0.1:8080;
}
location / {
	# 정적 파일을 위한 옵션은 이곳에 설정함
	# 예) 캐시 제어, 별칭 등록 등
	expires 30d;
}
```

이 방법은 단순하지만 URL 재작성을 사용하는 웹 사이트에서는 문제가 일어날 것이다. 웹 2.0 웹 사이트 대부분은 http://example.com/articles/us-economy-strengthens/ 같이 파일 확장자를 숨기는 링크를 사용한다. 어떤 경우는 .html 같은 가짜 확장자를 추가하기도 한다. http://example.com/us-economy-strengthens.html처럼 말이다.  

이 상황에 직면하면 다음 방법 중 하나로 해결할 수 있다.  

+ 좀 더 깔끔한 방법으로, 아파치 재작성 규칙(rewrite rule)을 엔진엑스에 맞게 변환하는 것이다. 아파치의 재작성 규칙은 보통 웹 사이트의 최상위 디렉터리에 .htaccess란 이름의 파일로 저장돼 있다. 이렇게 하면 엔진엑스가 요청이 가리키는 파일의 실제 확장자를 알 수 있어 이 요청을 아파치에 올바로 전달하게 된다.
+ 아파치 재작성 규칙을 엔진엑스로 이전하기 싫다면 try&#95;files 지시어를 사용해서 요청된 URI을 제공하려고 시도해보게 한 후에 엔진엑스에서 URI에 해당하는 파일을 찾을 수 없을 때 아파치에 재전송하게 할 수 있다. 또는 단순히 error&#95;page 지시어를 통해 아파치가 "404" 응답을 처리하게 할 수 있다.  

가장 적절한 방법은 try&#95;files을 사용하는 방법이다. 요청된 URI를 직접, 또는 /문자를 뒤에 붙여 대응하는 폴더로 만들어 제공하려고 시도할 것이다. 모두 실패하면 이 요청을 아파치에 전달한다.  

```
server {
	server_name .example.com;
	root /home/example.com/www;
	[...]
	location / {
		# 요청된 파일을 제공하려고 시도하고, 안 되면 아파치에 전달
		try_files $uri $uri/ @proxy;
		# 정적 파일 관련 구성은 이곳에 추가
		expires 30d;
		[...]
	}
	location @proxy {
		# 요청을 아파치에 전달
		proxy_pass http://127.0.0.1:8080;
	}
}
```

다음은 error&#95;page 지시어를 사용한 방식을 구현한 예다.  

```
server {
	server_name .example.com;
	root /home/example.com/www;
	[...]
	location / {
		# 정적 파일 제공
		expires 30d;
		[...]
		# 404 오류가 발생하면 요청을 @proxy location 블록으로 전달
		error_page 404 @proxy;
	}
	location @proxy {
		# 요청을 아파치에 전달
		proxy_pass http://127.0.0.1:8080;
	}
}
```

대안으로 재작성(Rewrite) 모듈의 if 지시어를 사용한다.  

```
server {
	server_name .example.com;
	root /home/example.com/www;
	[...]
	location / {
		# 요청된 파일 확장가 .php로 끝나면
		# 요청을 아파치에 전달
		if ($request_filename ~* \.php.$) {
			break; # 재작성 규칙 추가 적용 중단
			proxy_pass http://127.0.0.1:8080;
		}

		# 요청된 파일이 없으면 요청을 아파치에 전달
		if (!-f $request_filename) {
			break; # 재작성 규칙 추가 적용 중단
			proxy_pass http://127.0.0.1:8080;
		}

		# 정적 파일은 여기에서 제공
		expires 30d;
	}
}
```

어떤 방법을 써도 뒷단에 보내는 요청의 수는 같기 때문에 성능상 차이는 많이 나지 않는다. 성능을 최적화하겠다면 아파치 재작성 규칙을 엔진엑스로 이전해야 한다.  

##### 6.3.3. 고급 구성
<br/>

지금까지는 프록시 모듈이 제공하는 지시어를 하나만 사용했다. 프록시 모듈에는 설계를 최적화하는 데 사용할 수 있는 기능이 더 많다. 다음 표는 개별적으로 검증이 필요하지만 역프록시 구성 대부분에 유효한 일부 설정의 목록이다. 이 설정은 여러 번 사용될 수 있기 때문에 location 블록에서 불러서 쓸 수 있도록 별도 구성 파일로 분리해두는 것도 좋다.  

엔진엑스 구성 지시어를 넣을 proxy.conf 텍스트 파일을 만드는 것으로 시작하자. 다음 표에서 설명할 지시어를 이 파일에 넣자. 그리고 뒷단 서버나 업스트림 블록으로 요청을 전달하는 if 블록 위치마다 다음 줄을 proxy&#95;pass 지시어 다음에 추가하자.  

```
include proxy.conf;
```

일부 설정의 권장 값은 다음과 같다.  

<table>
	<thead>
		<tr>
			<th>설정</th>
			<th>설명</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				proxy&#95;set&#95;header Host $host;
			</td>
			<td>
				뒷단 서버로 전달되는 요청의 Host HTTP 헤더는 기본값으로 구성 파일에 지정한 프록시의 호스트명이다. 이 설정은 엔진엑스가 클라이언트 요청의 원래 Host 값을 대신 사용하도록 설정한다.
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;set&#95;header X-Real-IP $remote&#95;addr;
			</td>
			<td>
				뒷단 서버가 엔진엑스에서 오는 요청을 수신하는 이상. 통신하는 IP 주소가 클라이언트의 것이 아니다. 이 설정을 사용해서 클라이언트의 실제 IP 주소를 X-Real-IP라는 새로운 헤더에 담아 전달하자.
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;set&#95;header X-Forwarded-For $proxy&#95;add&#95;x&#95;forwarded&#95;for;
			</td>
			<td>
				X-Real-IP와 비슷하지만, 클라이언트가 이미 스스로 프록시를 사용하고 있다면 클라이언트의 실제 IP 주소는 X-Forwarded-For라는 요청 헤더에 들어 있을 것이다. 통신에 사용하는 소켓과 (프록시 뒤에 있는) 클라이언트의 원래 IP 주소 모두를 뒷단 서버에 확실히 전달하는 데 $proxy&#95;add&#95;x&#95;forwarded&#95;for를 사용하자.
			</td>
		</tr>
		<tr>
			<td>
				client&#95;max&#95;body&#95;size 10m;
			</td>
			<td>
				요청 본문의 최대 크기를 10메가바이트로 제한하자. 사실 이 설정은 참고일 뿐이고 뒷단 서버와 같은 수준으로 이 값을 조정하자. 아니면 엔진엑스가 정상적으로 받아 처리한 요청이 뒷단에 성공적으로 전달되지 못할 것이다.
			</td>
		</tr>
		<tr>
			<td>
				client&#95;body&#95;buffer&#95;size 128k;
			</td>
			<td>
				요청 본문을 보관할 메모리 버퍼의 최소 크기를 정한다. 이 크기를 넘으면 데이터는 임시 파일에 저장된다. client&#95;max&#95;body&#95;size처럼 이 값도 방문자가 보낼 것으로 예상되는 요청의 크기에 맞게 조정하자.
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;connect&#95;timeout 1S;
			</td>
			<td>
				내부 네트워크상에 있는 뒷단 서버로 작업한다고 하면 이 값을 적절한 수준으로 낮추자. 예에서는 15초라고 했지만 평균 부하에 따라 달라진다. 이 지시어의 최댓값은 75초다.
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;send&#95;timeout 1S;
			</td>
			<td>
				(뒷단 서버와 통신하는 동안 두 쓰기 작업 사이의 제한시간만) 쓰기 작업용 제한시간을 정한다.
			</td>
		</tr>
		<tr>
			<td>
				proxy&#95;read&#95;timeout 1S;
			</td>
			<td>
				쓰기 작업용이란 점만 빼면 이전 지시어와 비슷하다.
			</td>
		</tr>
	</tbody>
</table>
<br/><br/>

다른 많은 지시어도 여기에 구성할 수 있지만 대부분 구성에는 기본값이 적절하다.  

#### 6.4. 역프록시 아키텍처 개선
<br/>

##### 6.4.1. 올바른 IP 주소 전달
<br/>

요즘에는 상당한 웹 사이트가 방문자의 IP 주소를 다양한 이유로 사용한다.  

+ 블로그나 게시판에 댓글을 다는 방문자의 IP 주소 저장
+ 지리적 위치 기반 광고나 다른 서비스
+ 특정 IP 주소 범위를 대상으로 서비스 제한  

따라서 이런 웹 사이트에서는 웹 서버가 방문자의 IP 주소를 정확히 얻을 수 있게 만드는 게 중요하다.  

##### 6.4.2. SSL 문제와 해법
<br/>

##### 6.4.3. 서버 제어 패널 문제
<br/>