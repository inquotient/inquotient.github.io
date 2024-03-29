---
title:  자바 성능 튜닝이야기
categories:
- Java Performance
feature_text: |
  ## 자바 성능 튜닝이야기
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 16. JVM은 도대체 어떻게 구동될까?  
<br/>

#### 16.1. HotSpot VM은 어떻게 구성되어 있을까?  
<br/>

HotSpot VM은 자바 1.3 버전부터 기본 VM으로 사용되어 왔다.  

HotSpot VM은 세 가지 주요 컴포넌트로 되어 있다.  

+ VM(Virtual Machine) 런타임
+ JIT(Just In Time) 컴파일러
+ 메모리 관리자  

HotSpot VM은 높은 성능과 확장성을 제공한다. JIT 컴파일러는 자바 애플리케이션이 수행되는 상황을 보고 최적화를 수행한다.  

HotSpot VM 런타임은 GC 방식과 JIT 컴파일러를 끼워 맞춰 사용할 수 있다. 이를 위해서 VM 런타임은 JIT 컴파일러용 API와 가비지 컬렉터용 API를 제공한다. 그리고, JVM을 시작하는 런처와 스레드 관리, JNI 등도 VM 런타임에서 제공한다.  

#### 16.2. JIT Optimizer라는 게 도대체 뭘까?  
<br/>

JIT는 애플리케이션에서 각각의 메서드를 컴파일할 만큼 시간적 여유가 많지 않다. 그러므로, 모든 코드는 초기에 인터프리터에 의해서 시작되고, 해당 코드가 충분히 많이 사용될 경우에 컴파일할 대상이 된다. HotSpot VM에서 이 작업은 각 메서드에 있는 카운터를 통해서 통제되며, 메서드에는 두 개의 카운터가 존재한다.  

+ 수행 카운터(invocation counter): 메서드를 시작할 때마다 증가
+ 백에지카운터(backedge counter): 높은 바이트 코드 인덱스에서 낮은 인덱스로 컨트롤 흐름이 변경될 때마다 증가  

여기서 백에지 카운터는 메서드가 루프가 존재하는지를 확인할 때 사용되며, 수행 카운터 보다 컴파일 우선순위가 높다.  

이 카운터들이 인터프리터에 의해서 증가될 때마다, 그 값들이 한계치에 도달했는지를 확인하고, 도달했을 경우 인터프리터는 컴파일을 요청한다. 여기서 수행 카운터에서 사용하는 한계치는 CompilerThreshold이며, 백에지 카운터에서 사용하는 한계치는 다음의 공식으로 계산한다.  

CompilerThreshold &#42; OnStackReplacePercentage / 100  

이 두 개의 값들은 JVM이 시작할 때 지정 가능하며 다음과 같이 시작 옵션에 지정할 수 있다.  

+ XX:CompileThreshold=35000
+ XX:OnStackReplacePercentage=80  

즉, 이렇게 지정하면, 메서드가 35000번 호출되었을 때 JIT에서 컴파일을 하며, 백에지 카운터가 35000 &#42; 80 / 28000이 되었을 때 컴파일한다.  

컴파일이 요청되면 컴파일 대상 목록의 큐에 쌓이고, 하나 이상의 컴파일러 스레드가 이 큐를 모니터링한다. 만약 컴파일러 스레드가 바쁘지 않을 때는 큐에서 대상을 빼내서 컴파일을 시작한다. 보통 인터프리터는 컴파일이 종료되기를 기다리지 않는 대신, 수행 카운터를 리셋하고 인터프리터에서 메서드 수행을 계속한다. 컴파일이 종료되면, 컴파일된 코드와 메서드가 연결되어 그 이후부터는 메서드가 호출되면 컴파일된 코드를 사용하게 된다. 만약 인터프리터에서 컴파일이 종료될 때까지 기다리도록 하려면, JVM 시작시 -Xbatch나 -XX:BackgroundCompilation 옵션을 지정하여 컴파일을 기다리도록 할 수도 있다.  

HotSpot VM은 OSR(On Stack Replacement)이라는 특별한 컴파일도 수행한다. 이 OSR은 인터프리터에서 수행한 코드 중 오랫동안 루프가 지속되는 경우에 사용된다. 만약 해당 코드의 컴파일이 완료된 상태에서 최적화되지 않은 코드가 수행되고 있는 것을 발견한 경우에 인터프리터에 계속 머무르지 않은 코드가 수행되고 있는 것을 발견한 경우에 인터프리터에 계속 머무르지 않고 컴파일된 코드로 변경한다. 이 작업은 인터프리터에서 시작된 오랫동안 지속되는 루프가 다시는 불리지 않을 경우엔 도움이 되지 않지만, 루프가 끝나지 않고 지속적으로 수행되고 있을 경우에는 큰 도움이 된다.  

HotSpot VM은 JVM이 시작될 때 플랫폼과 시스템 설정을 평가하여 자동으로 garbage collector를 선정하고, 자바 힙 크기와 JIT 컴파일러를 선택하는 것이다. 이 기능을 통해서 애플리케이션의 활동과 객체 할당 비율에 따라서 garbage collector가 동적으로 자바 힙 크기를 조절하며, New와 Eden과 Survivor, Old 영역의 비율을 자동적으로 조절하는 것을 의미한다. 이 기능은 -XX:+UserParallelGC와 -XX:+UserParallelOldGC에서만 적용되며, 이 기능을 제거하려면 -XX:-UseAdaptiveSizePolicy라는 옵션을 적용하여 끌 수가 있다.  

#### 16.3. JRockit의 JIT 컴파일 및 최적화 절차  
<br/>

+ JRockit runs JIT compilation  
자바 애플리케이션을 실행하면 기본적으로는 JIT 컴파일을 거친 후 실행이 된다. 이 단계를 거친 후 메서드가 수행되면, 그 다음부터는 컴파일된 코드를 호출하기 때문에 처리 성능이 빨라진다.  

애플리케이션이 시작하는 동안 몇천 개의 새로운 메서드가 수행되며 이로 인해 다른 JVM보다 JRockit JVM이 더 느릴 수도 있다. 그리고, 이 작업으로 인해 JIT가 메서드를 수행하고 컴파일하는 작업은 오버헤드가 되지만, JIT가 없으면 JVM은 계속 느린 상태로 지속될 것이다. 다시 말해서, JIT를 사용하면 시작할 때의 성능은 느리겠지만, 지속적으로 수행할 때는 더 빠른 처리가 가능하다. 따라서, 모든 메서드를 컴파일하고 최적화하는 작업은 JVM 시작 시간을 느리게 만들기 때문에 시작할 때는 모든 메서드를 최적화하지는 않는다.  

+ JRockit monitors threads  
JRockit에는 'sampler thread'라는 스레드가 존재하며 주기적으로 애플리케이션의 스레드를 점검한다. 이 스레드는 어떤 스레드가 동작 중인지 여부와 수행 내역을 관리한다. 이 정보들을 통해서 어떤 메서드가 많이 사용되는지를 확인하여 최적화 대상을 찾는다.  

+ JRockit JVM Runs Optimization  
sampler thread가 식별한 대상을 최적화한다. 이 작업은 백그라운드에서 진행되며 수행중인 애플리케이션에 영향을 주지는 않는다.  

#### 16.4. IBM JVM의 JIT 컴파일 및 최적화 절차  
<br/>

+ 인라이닝(inlining)  
메서드가 단순할 때 적용되는 방식이며, 호출된 메서드가 단순할 경우 그 내용이 호출한 메서드의 코드에 포함해 버린다. 이렇게 될 경우 자주 호출하는 메서드의 성능이 향상된다는 장점이 있다.  

+ 지역 최적화(Local optimization)  
작은 단위의 코드를 분석하고 개선하는 작업을 수행한다.  

+ 조건 구문 최적화(Control flow optimizations)  
메서드 내의 조건 구문을 최적화하고, 효율성을 위해서 코드의 수행 경로를 변경한다.  

+ 글로벌 최적화(Global optimization)  
메서드 전체를 최적화하는 방식이다. 매우 비싼 방식이며, 컴파일 시간이 많이 소요된다는 단점이 있지만, 성능 개선이 많이 될 수 있다는 장점이 있다.  

+ 네이티브 코드 최적화(Native code generation)  
이 방식은 플랫폼 아키텍처에 의존적이다. 다시 말해서 아키텍처에 따라서 최적화를 다르게 처리한다.  

컴파일된 코드는 '코드 캐시(Code cache)'라고 하는 JVM 프로세스 영역에 저장된다. 결과적으로 JVM 프로세스는 JVM 수행 파일과 컴파일된 JIT 코드의 집합으로 구분된다.  

#### 16.5. JVM이 시작할 때의 절차는 이렇다.  
<br/>

(1) java 명령어 줄에 있는 옵션 파싱:  
일부 명령은 자바 실행 프로그램에서 적절한 JIT 컴파일러를 선택하는 등의 작업을 하기 위해서 사용하고, 다른 명령들은 HotSpot VM에 전달된다.  

(2) 자바 힙 크기 할당 및 JIT 컴파일러 타입 지정(이 옵션들이 명령줄에 지정되지 않았을 경우):  
메모리 크기나 JIT 컴파일러 종류가 명시적으로 지정되지 않은 경우에 자바 실행 프로그램이 시스템의 상황에 맞게 선정한다. 이 과정은 좀 복잡한 단계(HotSpot VM Adapter Tuning)를 거치니 일단 넘어가자.  

(3) CLASSPATH와 LD&#95;LIBRARY&#95;PATH 같은 환경 변수를 지정한다.  

(4) 자바의 Main 클래스가 지정되지 않았으면, Jar 파일의 manifest 파일에서 Main 클래스를 확인한다.  

(5) JVI 표준 API인 JNI&#95;CreateJavaVM를 사용하여 새로 생성한 non-primordial이라는 스레드에서 HotSpot VM을 생성한다.  

(6) HotSpot VM이 생성되고 초기화되면, Main 클래스가 로딩된 런처에서는 main() 메서드의 속성 정보를 읽는다.  

(7) CallStaticVoidMethod는 네이티브 인터페이스를 불러 HotSpot VM에 있는 main() 메서드가 수행된다. 이때 자바 실행 시 Main 클래스 위에 있는 값들이 전달된다.  

추가로 JNI&#95;CreateJavaVM 단계에 대해서 더 알아보자. 이 단계에서는 다음의 절차를 거친다.  

(1) JNI&#95;CreateJavaVM는 동시에 두개의 스레드에서 호출할 수 없고, 오직 하나의 HotSpot VM 인스턴스가 프로세스 내에서 생성될 수 있도록 보장된다. HotSpot VM이 정적인 데이터 구조를 생성하기 때문에 다시 초기화는 불가능하기 때문에, 오직 하나의 HotSpot VM이 프로세스에서 생성될 수 있다.  

(2) JNI 버전이 호환성이 있는지 점검하고, GC 로깅을 위한 준비도 완료된다.  

(3) OS 모듈들이 초기화된다. 예를 들면 랜덤 번호 생성기, PID 할당 등이 여기에 속한다.  

(4) 커맨드 라인 변수와 속성들이 JNI&#95;CreateJavaVM 변수에 전달되고, 나중에 사용하기 위해서 파싱한 후 보관한다.  

(5) 표준 자바 시스템 속성(properties)이 초기화된다.  

(6) 동기화, 메모리, safepoint 페이지와 같은 모듈들이 초기화된다.  

(7) libzip, libhpi, libjava, libthread와 같은 라이브러리들이 로드된다.  

(8) 시그널 처리기가 초기화 및 설정된다.  

(9) 스레드 라이브러리가 초기화된다.  

(10) 출력(output) 스트림 로거가 초기화된다.  

(11) JVM을 모니터링하기 위한 에이전트 라이브러리가 설정되어 있으면 초기화 및 시작된다.  

(12) 스레드 처리를 위해서 필요한 스레드 상태와 스레드 로컬 저장소가 초기화된다.  

(13) HotSpot VM의 '글로벌 데이터'들이 초기화된다. 글로벌 데이터에는 이벤트 로그(event log), OS 동기화, 성능 통계 메모리(perfMemory), 메모리 할당자(chunkPool)들이 있다.  

(14) HotSpot VM에서 스레드를 생성할 수 있는 상태가 된다. main 스레드가 생성되고, 현재 OS 스레드에서 붙는다. 그러나 아직 스레드 목록에 추가되지는 않는다.  

(15) 자바 레벨의 동기화가 초기화 및 활성화된다.  

(16) 부트 클래스로더, 코드 캐시, 인터프리터, JIT 컴파일러, JNI, 시스템 directory, '글로벌 데이터' 구조의 집합인 universe 등이 초기화된다.  

(17) 스레드 목록에 자바 main 스레드가 추가되고, universe의 상태를 점검한다. HotSpot VM의 중요한 기능을 하는 HotSpot VMThread가 생성된다. 이 시점에 HotSpot VM의 현재 상태를 JVMTI에 전달한다.  

(18) java.lang 패키지에 있는 String, System, Thread, ThreadGroup, Class 클래스와 java.lang의 하위 패키지에 있는 Method, Finalizer 클래스 등이 로딩되고 초기화된다.  

(19) HotSpot VM의 시그널 핸들러 스레드가 시작되고, JIT 컴파일러가 초기화되며, HotSpot의 컴파일 브로커 스레드가 시작된다. 그리고, HotSpot VM과 관련된 각종 스레드들이 시작한다. 이때부터 HotSpot VM의 전체 기능이 동작한다.  

(20) JNIEnv가 시작되며, HotSpot VM을 시작한 호출자에게 새로운 JNI 요청을 처리할 상황이 되었다고 전달해 준다.  

이렇게 복잡한 JNI&#95;CreateJavaVM 시작 단계를 거치고, 나머지 단계들을 거치면 JVM이 시작된다.  


#### 16.6. JVM이 종료될 때의 절차는 이렇다  
<br/>

그러면 JVM이 종료될 때는 어떤 절차를 거칠까? 만약 정상적으로 JVM을 종료시킬 때는 다음의 절차를 거치지만, OS의 kill -9와 같은 명령으로 JVM을 종료시키면 이 절차를 다르지 않는다는 것을 명심하기 바란다.  

만약 JVM이 시작할 때 오류가 있어 시작을 중지할 때나, JVM에 심각한 에러가 있어서 중지할 필요가 있을 때는 DestroyJavaVM이라는 메서드를 HotSpot 런처에서 호출한다.  

HotSpot VM의 종료는 다음의 DestroyJavaVM 메서드의 종료 절차를 따른다.  

(1) HotSpot VM이 작동중인 상황에서는 단 하나의 데몬이 아닌 스레드(nondaemon thread)가 수행될 때까지 대기한다.  

(2) java.lang 패키지에 있는 Shutdown 클래스의 shutdown() 메서드가 수행된다. 이 메서드가 수행되면 자바 레벨의 shutdown hook이 수행되고, finalization-on-exit이라는 값이 true일 경우에 자바 객체 finalizer를 수행한다.  

(3) HotSpot VM 레벨의 shutdown hook을 수행함으로써 HotSpot VM의 종료를 준비한다. 이 작업은 JVM&#95;OnExit() 메서드를 통해서 지정된다. 그리고, HotSpot VM의 profiler, stat sampler, watcher, garbage collector 스레드를 종료시킨다.  

(4) HotSpot의 JavaThread::exit() 메서드를 호출하여 JNI 처리 블록을 해제한다. 그리고, guard pages, 스레드 목록에 있는 스레드들을 삭제한다. 이 순간부터는 HotSpot VM에서는 자바 코드를 실행하지 못한다.  

(5) HotSpot VM 스레드를 종료한다. 이 작업을 수행하면 HotSpot VM에 남아 있는 HotSpot VM 스레드들을 safepoint로 옮기고, JIT 컴파일러 스레드들을 중지시킨다.  

(6) JNI, HotSpot VM, JVMTI barrier에 있는 추적(tracing) 기능을 종료시킨다.  

(7) 네이티브 스레드에서 수행하고 있는 스레드들을 위해서 HotSpot의 "vm exited" 값을 설정한다.  

(8) 현재 스레드를 삭제한다.  

(9) 입출력 스트림을 삭제하고, PerfMemory 리소스 연결을 해제한다.  

(10) JVM 종료를 호출한 호출자로 복귀한다.  

이 절차를 거처 JVM이 종료된다.  


#### 16.7. 클래스 로딩 절차도 알고 싶어요?  
<br/>

클래스 로딩 절차도 간단히 살펴보자.  

(1) 주어진 클래스의 이름으로 클래스 패스에 있는 바이너리로 된 자바 클래스를 찾는다.  

(2) 자바 클래스를 정의한다.  

(3) 해당 클래스를 나타내는 java.lang 패키지의 Class 클래스의 객체를 생성한다.  

(4) 링크 작업이 수행된다. 이 단계에서 static 필드를 생성 및 초기화하고, 메서드 테이블을 할당한다.  

(5) 클래스의 초기화가 진행되며, 클래스의 static 블록과 필드가 가장 먼저 초기화 된다. 당연한 이야기지만, 해당 클래스가 초기화되기 전에 부모 클래스의 초기화가 먼저 이루어진다.  


이렇게 나열하니 단계가 복잡해 보이지만, loading → linking → initializing로 기억하면 된다.  

클래스가 로딩될 때 다음과 같은 에러가 발생될 수 있다. 참고로, 일반적으로 이 에러들은 자주 발생하지 않는다.  

+ NoClassDefFoundError: 만약 클래스 파일을 찾지 못할 경우
+ ClassFormatError: 클래스 파일의 포맷이 잘못된 경우
+ UnsupportedClassVersionError: 상위 버전의 JDK에서 컴파일한 클래스를 하위 버전의 JDK에서 실행하려고 하는 경우
+ ClassCircularityError: 부모 클래스를 로딩하는 데 문제가 있는 경우 (자바는 클래스를 로딩하기 전에 부모 클래스들을 로딩해야 한다.)
+ IncompatibleClassChangeError: 부모가 클래스인데 implements를 하는 경우나 부모가 인터페이스인데 extends하는 경우
+ VerifyError: 클래스 파일의 semantic, 상수 풀, 타입 등의 문제가 있을 경우  

그런데 클래스 로더가 클래스를 찾고 로딩할 대 다른 클래스 로더에 클래스를 로딩해 달라고 하는 경우가 있다. 이를 'class loader delegation'이라고 부른다. 클래스 로더는 계층적으로 구성되어 있다. 기본 클래스 로더는 '시스템 클래스 로더'라고 불리며 main 메서드가 있는 클래스와 클래스 패스에 있는 클래스들이 이 클래스 로더에 속한다. 그 하위에 있는 애플리케이션 클래스 로더는 자바 SE의 기본 라이브러리에 있는 것이 될 수도 있고, 개발자가 임의로 만든 것일 수도 있다.  


##### 16.7.1. 부트스트랩(Bootstrap) 클래스 로더  
<br/>

HotSpot VM은 부트스트랩 클래스 로더를 구현한다. 부트스트랩 클래스 로더는 HotSpot VM의 BOOTCLASSPATH에서 클래스들을 로드한다. 예를 들면, Java SE(Standard Edition) 클래스 라이브러리들을 포함하는 rt.jar가 여기에 속한다.  

보다 빠른 JVM의 시작을 위해서 클라이언트 HotSpot VM은 '클래스 데이터 공유'라고 불리는 기능을 통해서 클래스를 미리 로딩할 수도 있으며, 이 기능은 기본적으로 켜져 있다. 이 기능은 -Xshare:on이라는 옵션을 사용해서 명시적으로 켤 수 있으며, -Xshare:off 옵션을 사용해서 기능을 끈다. 그런데, 서버 HotSpot VM은 '클래스 데이터 공유' 기능을 제공하지 않고, 또 클라이언트 HotSpot VM도 시리얼 GC를 사용하지 않을 경우에는 이 기능을 제공하지 않는다.  

##### 16.7.2. HotSpot의 클래스 메타데이터(Metadata)  
<br/>

HotSpot VM 내에서 클래스를 로딩하면 클래스에 대한 instanceKlass와 arrayKlass라는 내부적인 형식을 VM의 Perm 영역에 생성한다. instanceKlass는 클래스의 정보를 포함하는 java.lang.Class 클래스의 인스턴스를 말한다. HotSpot VM은 내부 데이터 구조인 klassOop라는 것을 사용하여 내부적으로 instanceKlass에 접근한다. 여기서 Oop라는 것은 ordinary object pointer의 약자다. 즉, klassOop는 클래스를 나타내는 ordinary object pointer를 의미한다.  

##### 16.7.3. 내부 클래스 로딩 데이터의 관리  
<br/>

HotSpot VM은 클래스 로딩을 추적하기 위해서 다음의 3개의 해시 테이블을 관리한다.  

+ SystemDictionary  
로드된 클래스를 포함하며, 클래스 이름 및 클래스 로더를 키를 갖고 그 값으로 klassOop를 갖고 있다. SystemDictionary는 클래스 이름과 초기화한 정보, 클래스 이름과 정의한 정보도 포함한다. 이 정보들은 safepoint에서만 제거한다.  

+ PlaceholderTable  
현재 로딩된 클래스들에 대한 정보를 관리한다. 이 테이블은 ClassCircularityError를 체크할 때 사용하며, 다중 스레드에서 클래스를 로딩하는 클래스 로더에서도 사용된다.  

+ LoaderConstraintTable  
타입 체크시의 제약 사항을 추정하는 용도로 사용된다.  

##### 16.7.4. 예외는 JVM에서 어떻게 처리될까?  
<br/>

마지막으로 예외가 발생했을 때 JVM에서 어떻게 처리되는지 알아보자.  

JVM은 자바 언어의 제약을 어겼을 때 예외(exception)라는 시그널(signal)로 처리한다. HotSpot VM 인터프리터, JIT 컴파일러 및 다른 HotSpot VM 컴포넌트는 예외 처리와 모두 관련되어 있다. 일반적인 예외 처리 경우는 아래 두 가지 경우다.  

+ 예외를 발생한 메서드에서 잡을 경우
+ 호출한 메서드에 의해서 잡힐 경우  

후자의 경우에는 보다 복잡하며, 스택을 뒤져서 적당한 핸들러를 찾는 작업을 필요로 한다.  

예외는, 던져진 바이트 코드에 의해서 초기화될 수 있으며, VM 내부 호출의 결과로 넘어올 수도 있고, JNI 호출로부터 넘어올 수도 있고, 자바 호출로부터 넘어올 수도 있다.  

여기서 가장 마지막 경우는 단순히 앞의 세 가지 경우의 마지막 단계에 속할 뿐이다.  

VM의 예외가 던져졌다는 것을 알아차렸을 때, 해당 예외를 처리하는 가장 가까운 핸들러를 찾기 위해서 HotSpot VM 런타임 시스템이 수행된다. 이 때, 핸들러를 찾기 위해서는 다음의 3개의 정보가 사용된다.  

+ 현재 메서드
+ 현재 바이트 코드
+ 예외 객체  

만약 현재 메서드에서 핸들러를 찾지 못했을 때는 현재 수행되는 스택 프레임을 통해서 이전 프레임을 찾는 작업을 수행한다. 적당한 핸들러를 찾으면, HotSpot VM 수행 상태가 변경되며, HotSpot VM은 핸들러로 이동하고 자바 코드 수행은 계속된다.  

### 17. 도대체 GC는 언제 발생할까?  
<br/>

#### 17.1. 자바의 Runtime data area는 이렇게 구성된다  
<br/>

+ 클래스 로더 서브 시스템  
클래스나 인터페이스를 JVM으로 로딩하는 기능을 수행  

+ 실행 엔진  
로딩된 클래스의 메서드들에 포함되어 있는 모든 인스트럭션 정보를 실행  

+ Heap 메모리  
클래스 인스턴스, 배열이 이 메모리에 쌓인다. 이 메모리는 '공유(shared) 메모리'라고도 불리우며 여러 스레드에서 공유하는 데이터들이 저장되는 메모리  

+ 메서드 영역  
메서드 영역은 모든 JVM 스레드에서 공유한다.  

+ 런타임 상수 풀  
자바의 클래스 파일에는 constant&#95;pool이라는 정보가 포함되어 있다. 이 constant&#95;pool에 대한 정보를 실행 시에 참조하기 위한 영역이다. 실제 상수 값도 여기에 포함될 수 있지만, 실행 시에 변하게 되는 필드 참조 정보도 포함된다.  

+ JVM 스택  
스레드가 시작할 대 JVM의 스택이 생성된다. 이 스택에는 메서드가 호출되는 정보인 프레임(frame)이 저장된다. 그리고, 지역 변수와 임시 결과, 메서드 수행과 리턴에 관련된 정보들도 포함한다.  

+ 네이티브 메서드 스택  
자바 코드가 아닌 다른 언어로 된(보통은 C로 된) 코드들이 실행하게 될 때의 스택 정보를 관리한다.  

+ PC 레지스터  
자바의 스레드들은 각자의 pc(Program Counter) 레지스터를 갖는다. 네이티브한 코드를 제외한 모든 자바 코드들이 수행될 때 JVM의 인스트럭션 주소를 pc 레지스터에 보관한다.  

스택의 크기는 고정하거나 가변적일 수 있다. 만약 연산을 하다가 JVM의 스택 크기의 최대치를 넘어섰을 경우에는 StackOverflowError가 발생한다. 그리고, 가변적일 경우 스택의 크기를 늘이려고 할 때 메모리가 부족하거나, 스레드를 생성할 때 메모리가 부족한 경우에는 OutOfMemoryError가 발생한다.  

여기서 Heap 영역과 메서드 영역은 JVM이 시작될 때 생성된다.  

#### 17.2. GC의 원리  
<br/>

GC 작업을 하는 가비지 콜렉터(Garbage Collector)는 다음의 역할을 한다.  

+ 메모리 할당
+ 사용 중인 메모리 인식
+ 사용하지 않는 메모리 인식  

사용하지 않는 메모리를 인식하는 작업을 수행하지 않으면, 할당한 메모리 영역이 꽉 차서 JVM에 행(Hang)이 걸리거나, 더 많은 메모리를 할당하려는 현상이 발생할 것이다. 만약 JVM의 최대 메모리 크기를 지정해서 전부 사용한 다음, GC를 해도 더 이상 사용 가능한 메모리 영역이 없는데 계속 메모리를 할당하려고 하면 OutOfMemoryError가 발생하여 JVM가 다운될 수도 있다.  

자바의 힙 영역은 크게 Young, Old, Perm 세 영역으로 나뉜다. 이 중 Perm(Permanent)영역은 거의 사용이 되지 않는 영역으로 클래스와 메서드 정보와 같이 자바 언어 레벨에서 사용하는 영역이 아니다. 게다가 JDK 8부터는 이 영역이 사라진다. Young 영역은 다시 Eden 영역 및 두 개의 Survivor 영역으로 나뉘므로 우리가 고려해야 할 자바의 메모리 영역은 총 4개의 영역으로 나뉜다고 볼 수 있다.  

Perm 영역에는 클래스와 메서드 정보외에도 intern된 String 정보도 포함하고 있다. String 클래스에는 intern() 이라는 메서드가 존재한다. 이 메서드를 호출하면 해당 문자열의 값을 바탕으로 한 단순 비교가 가능하다. 즉, 참조 자료형은 equals() 메서드로 비교를 해야 하지만, intern() 메서드가 호출한 문자열들은 == 비교가 가능하다. 따라서, 값 비교 성능은 빨리지지만, 문자열 정보들이 Perm 영역에 들어가서 때문에 Perm 영역의 GC가 발생하는 원인이 되기도 한다. 물론 이 현상은 JDK 8부터는 발생하지 않을 것이다.  

일단 메모리에 객체가 생성되면, Eden 영역에 객체가 지정된다.  

Eden 영역에 데이터가 꽉 차면, 이 영역에 있던 객체가 어디론가 옮겨지거나 삭제되어야 한다. 이 때 옮겨 가는 위치가 Survivor 영역이다. 두 개의 Survivor 영역 사이에 우선 순위가 있는 것은 아니다. 이 두 개의 영역 중 한 영역은 반드시 비어 있어야 한다. 그 비어 있는 영역에 있던 객체 중 GC 후에 살아 남아 있는 객체들이 이동한다.  

이와 같이 Eden 영역에 있던 객체는 Survivor 영역의 둘 중 하나에 할당된다. 할당된 Survivor 영역이 차면, GC가 되면서 Eden 영역에 있는 객체와 꽉 찬 Survivor 영역에 있는 객체가 비어 있는 Survivor 영역으로 이동한다. 이러한 작업을 반복하면서, Survivor 1과 2를 왔다 갔다 하던 객체들은 Old 영역으로 이동한다.  

그리고, 객체의 크기가 아주 큰 경우에 Young 영역에서 Old 영역으로 넘어가는 객체 중 Survivor 영역을 거치지 않고 바로 Old 영역으로 이동하는 객체가 있을 수 있다.  

#### 17.3. GC의 종류  
<br/>

+ 마이너GC: Young 영역에서 발생하는 GC
+ 메이저GC: Old 영역이나 Perm 영역에서 발생하는 GC  

이 두 가지 GC가 어떻게 상호 작용하느냐에 따라서 GC 방식에 차이가 나며, 성능에도 영향을 준다.  

GC가 발생하거나 객체가 각 영역에서 다른 영역으로 이동할 때 애플리케이션의 병목이 발생하면서 성능에 영향을 주게 된다. 그래서 HotSpot JVM에서는 스레드 로컬 할당 버퍼(TLABs: Thread-Local Allocation Buffer)라는 것을 사용한다. 이를 통하여 각 스레드별 메모리 버퍼를 사용하면 다른 스레드에 영향을 주지 않는 메모리 할당 작업이 가능해진다.  

#### 17.4. 5가지 GC 방식  
<br/>

JDK 7이상에서 지원하는 GC 방식에는 다섯 가지가 있다.  

+ Serial Collector
+ Parallel Collector
+ Parallel Compacting Collector
+ Concurrent Mark-Sweep (CMS) Collector
+ Garbage First Collector(G1 콜렉터)  

##### 17.4.1. 시리얼 콜렉터  
<br/>

Young 영역와 Old 영역이 시리얼하게(연속적으로) 처리되며 하나의 CPU를 사용한다. Sun에서는 이 처리를 수행할 때를 Stop-the-world라고 표현한다. 다시 말하면, 콜렉션이 수행될 때 애플리케이션 수행이 정지된다.  

(1) 일단 살아 있는 객체들은 Eden 영역에 있다  
(2) Eden 영역이 꽉 차게 되면 To Survivor 영역(비어 있는 영역)으로 살아 있는 객체가 이동한다. 이때 Survivor 영역에 들어가기에 너무 큰 객체는 바로 Old 영역으로 이동한다. 그리고 From Survivor 영역에 있는 살아 있는 객체는 To Survivor 영역으로 이동한다.  
(3) To Survivor 영역이 꽉 찼을 경우, Eden 영역이나 From Survivor 영역에 남아 있는 객체들은 Old 영역으로 이동한다.  

이후에 Old 영역이나 Perm 영역에 있는 객체들은 Mark-sweep-compact 콜렉션 알고리즘을 따른다. 이 알고리즘에 대해서 간단하게 말하면, 쓰이지 않는 객체를 표시해서 삭제하고 한 곳으로 모으는 알고리즘이다. Mark-sweep-compact 콜렉션 알고리즘은 다음과 같이 수행된다.  

(1) Old 영역으로 이동된 객체들 중 살아 있는 객체를 식별한다(표시 단계).  
(2) Old 영역의 객체들을 훑는 작업을 수행하여 쓰레기 객체를 식별한다(스왑 단계).  
(3) 필요 없는 객체들을 지우고 살아 있는 객체들을 한 곳으로 모은다(컬렉션 단계).  

이렇게 작동하는 시리얼 콜렉터는 일반적으로 클라이언트 종류의 장비에서 많이 사용된다. 다시 말하면, 대기 시간이 크게 문제되지 않는 시스템에서 사용된다는 의미이다. 시리얼 콜렉터를 명시적으로 지정하려면 자바 명령 옵션에 -XX:+UseSerialGC를 지정하면 된다.  

##### 17.4.2. 병렬 콜렉터  
<br/>

이 방식은 스루풋 콜렉터(throughput collector)로도 알려진 방식이다. 이 방식의 목표는 다른 CPU가 대기 상태로 남아 있는 것을 최소화하는 것이다. 시리얼 콜렉터와 달리 Young 영역에서의 콜렉션을 병렬(parallel)로 처리한다. 많은 CPU를 사용하기 때문에 GC의 부하를 줄이고 애플리케이션의 처리량을 증가시킬 수 있다.  

Old 영역의 GC는 시리얼 콜렉터와 마찬가지로 Mark-sweep-compact 콜렉션 알고리즘을 사용한다. 이 방법으로 GC를 하도록 명시적으로 지정하려면 -XX:+UseParallelGC 옵션을 자바 병렬 옵션에 추가하면 된다.  

##### 17.4.3. 병렬 콜랙팅 콜렉터  
<br/>

이 방식은 JDK 5.0 업데이트 6부터 사용 가능하다. 병렬 콜렉터와 다른 점은 Old 영역 GC에서 새로운 알고리즘을 사용한다는 것이다. 그러므로 Young 영역에 대한 GC는 병렬 콜렉터와 동일하지만, Old 영역의 GC는 다음의 3단계를 거친다.  

+ 표시 단계: 살아 있는 객체를 식별하여 표시해 놓는 단계.
+ 종합 단계: 이전에 GC를 수행하여 컬렉션된 영역에 살아 있는 객체의 위치를 조사하는 단계.
+ 컬렉션 단계: 컬렉션을 수행하는 단계. 수행 이후에는 컬렉션된 영역과 비어있는 영역으로 나뉜다.  

병렬 콜렉터와 동일하게 이 방식도 여러 CPU를 사용하는 서버에 적합하다. GC를 사용하는 스레드 개수는 -XX:ParallelThreads=n 옵션으로 조정할 수 있다. 이 방식을 사용하려면 -XX:+UseParallelOldGC 옵션을 자바 명령 옵션에 추가하면 된다.  

##### 17.4.4. CMS 콜렉터  
<br/>

이 방식은 로우 레이턴시 콜렉터(low-latency collector)로도 알려져 있으며, 힙 메모리 영역의 크기가 클 때 적합하다. Young 영역에 대한 GC는 병렬 콜렉터와 동일하다.  

Old 영역의 GC는 다음 단계를 거친다.  

+ 초기 표시 단계: 매우 짧은 대기 시간으로 살아 있는 객체를 찾는 단계.
+ 컨커런트 표시 단계: 서버 수행과 동시에 살아 있는 객체에 표시를 해 놓는 단계.
+ 재표시(remark) 단계: 컨커런트 표시 단계에서 표시하는 동안 변경된 객체에 대해서 다시 표시하는 단계.
+ 컨커런트 스윕 단계: 표시되어 있는 쓰레기를 정리하는 단계.  

CMS는 컬렉션 단계를 거치지 않기 때문에 왼쪽으로 메모리를 몰아 놓는 작업을 수행하지 않는다. 그래서 GC 이후에는 빈 공간이 발생하므로, -XX:CMSInitiatingOccupancyFraction=n 옵션을 사용하여 Old 영역의 %를 n값에 지정한다. 여기서 n값의 기본값은 68이다.  

CMS 콜렉터 방식은 2개 이상의 프로세서를 사용하는 서버에 적당하다. 가장 적당한 대상으로는 웹 서버가 있다. -XX:+UseConcMarkSweepGC 옵션으로 이 GC 방식을 지정할 수 있다.  

CMS 콜렉터는 추가적인 옵션으로 점진적 방식을 지원한다. 이 방식은 Young 영역의 GC를 더 잘게 쪼개어 서버의 대기 시간을 줄일 수 있다. CPU가 많지 않고 시스템의 대기 시간의 짧아야 할 때 사용하면 좋다. 점진적인 GC를 수행하려면 -XX:+CMSIncrementalMode 옵션을 지정하면 된다. JVM에 따라서는 -Xincgc라는 옵션을 지정해도 같은 의미가 된다. 하지만 이 옵션을 지정할 경우 예기치 못한 성능상 처리가 발생할 수 있으므로, 충분한 테스트를 한 후에 운영 서버에 적용해야 한다.  

##### 17.4.5. G1 콜렉터  
<br/>

G1 콜렉터는 바둑판 모양으로 각 바둑판의 사각형을 region이라고 하는데, Young 영역이나 Old 영역이라는 단어와 구분하기 위해서 한국말로 '구역'이라고 하자(이 구역의 기본 크기는 1MB이며 최대 32MB까지 지정 가능하다.). 각 구역의 크기는 모두 동일하다. 여기서 구역의 개수는 약 2000개 정도라고 한다.  

이 바둑판 모양의 구역이 각각 Eden, Survivor, Old 영역의 역할을 변경해 가면서 하고, Humongous라는 영역도 포함된다.  

G1이 Young GC를 어떻게 하는지 살펴보면 다음과 같다.  

(1) 몇 개의 구역을 선정하여 Young 영역으로 지정한다.  
(2) 이 Linear하지 않은 구역에 객체가 생성되면서 데이터가 쌓인다.  
(3) Young 영역으로 할당된 구역에 데이터가 꽉 차면, GC를 수행한다.  
(4) GC를 수행하면서 살아있는 객체들만 Survivor 구역으로 이동시킨다.  

이렇게 살아 남은 객체들이 이동된 구역은 새로운 Survivor 영역이 된다. 그 다음에 Young GC가 발생하면 Survivor 영역에 계속 쌓는다. 그러면서, 몇 번의 aging 작업을 통해서는(Survivor 영역에 있는 객체가 몇 번의 Young GC 후에도 살아 있으면), Old 영역으로 승격된다.  

G1의 Old 영역 GC는 CMS GC의 방식과 비슷하며 아래 여섯 단계로 나뉜다. 여기서 STW라고 표시된 단계에서는 모두 Stop the world가 발생한다.  

+ 초기 표시 (Initial Mark) 단계 (STW): Old 영역에 있는 객체에서 Survivor 영역의 객체를 참조하고 있는 객체들을 표시한다.
+ 기본 구역 스캔(Root region scanning) 단계: Old 영역 참조를 위해서 Survivor 영역을 훑는다. 참고로 이 작업은 Young GC가 발생하기 전에 수행된다.
+ 컨커런트 표시 단계: 전체 힙 영역에 살아있는 객체를 찾는다. 만약 이때 Young GC가 발생하면 잠시 멈춘다.
+ 재 표시(Remark) 단계 (STW): 힙에 있는 살아있는 객체들의 표시 작업을 완료한다. 이 때 snapshot-at-the-beginning (SATB)라는 알고리즘을 사용하며, 이는 CMS GC에서 사용하는 방식보다 빠르다.
+ 청소(Cleaning) 단계 (STW): 살아있는 객체와 비어 있는 구역을 식별하고, 필요 없는 객체들을 지운다. 그리고 나서 비어 있는 구역을 초기화한다.
+ 복사 단계 (STW): 살아있는 객체들을 비어 있는 구역으로 모은다.  

### 18. GC가 어떻게 수행되고 있는지 보고 싶다.  
<br/>

#### 18.1. 자바 인스턴스 확인을 위한 jps  
<br/>

jps는 해당 머신에서 운영 중인 JVM의 목록을 보여준다. JDK와 bin 디렉터리에 있다. jps의 사용법은 매우 간단하다. 커맨드 프롬프트나 유닉스의 터미널에서 다음과 같은 옵션으로 수행하면 된다.  

```
jps [-q] [-mlvV] [-Joption] [<hostid>]
```

jps 명령의 각 옵션은 다음과 같다.  

+ -q: 클래스나 JAR 파일명, 인수 등을 생략하고 내용을 나타낸다(단지 프로세스 id만 나타난다).
+ -m: main 메서드에 지정한 인수들을 나타낸다.
+ -l: 애플리케이션의 main 클래스나 애플리케이션 JAR 파일의 전체 경로 이름을 나타낸다.
+ -v: JVM에 전달된 자바 옵션 목록을 나타낸다.
+ -V: JVM의 플래그 파일을 통해 전달된 인수를 나타낸다.
+ -Joption: 자바 옵션을 이 옵션 뒤에 지정할 수 있다.  

플래그 파일이란 .hostpotrc의 확장자를 가지거나 자바 옵션에 -XX:Flags=&lt;file name&gt;로 명시한 파일이다. 이 파일을 통해서 JVM의 옵션을 지정할 수 있다.  

#### 18.2. GC 상황을 확인하는 jstat  
<br/>

jstat는 GC가 수행되는 정보를 확인하기 위한 명령어이다. jstat를 사용하면 유닉스 장비에서 vmstat나 netstat와 같이 라인 단위로 결과를 보여 준다.  

```
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```
&#45;&lt;option&gt; 부분을 제외한 jstat 명령의 옵션을 살펴보자.  

+ -t: 수행 시간을 표시한다.
+ -h:lines: 각 열의 설명을 지정된 라인 주기로 표시한다.
+ interval: 로그를 남기는 시간의 차이(밀리초 단위임)를 의미한다.
+ count: 로그 남기는 횟수를 의미한다.  

&lt;option&gt;의 종류는 다음과 같으며, &lt;option&gt;에 따라서 나타나는 결과의 내용이 많이 달라진다.  

+ class: 클래스 로더에 대한 통계
+ compiler: 핫스팟 JIT 컴파일러에 대한 통계
+ gc: GC 힙 영역에 대한 통계
+ gccapacity: 각 영역의 허용치와 연관된 영역에 대한 통계
+ gccause: GC의 요약 정보와, 마지막 GC와 현재 GC에 대한 통계
+ gcnew: 각 영역에 대한 통계
+ gcnewcapacity: Young 영역과 관련된 영역에 대한 통계
+ gcold: Old와 Perm 영역에 대한 통계
+ gcoldcapacity: Old 영역의 크기에 대한 통계
+ gcpermcapacity: Perm 영역의 크기에 대한 통계
+ printcompilation: 핫 스팟 컴파일 메서드에 대한 통계  

#### 18.4. 원격으로 JVM 상황을 모니터링하기 위한 jstatd  
<br/>

```
jstatd [-nr] [-p port] [-n rminame]
```

+ nr: RMI registry가 존재하지 않을 경우 새로운 RMI 레지스트리를 jstatd 프로세스 내에서 시작하지 않는 것을 정의하기 위한 옵션이다.
+ p: RMI 레지스트리를 식별하기 위한 포트 번호.
+ n: RMI 객체의 이름을 지정한다. 기본 이름은 JStatRemoteHost이다.  

#### 18.5. verbosegc 옵션을 이용하여 gc 로그 남기기  
<br/>

```
java -verbosegc <기타 다른 옵션들> 자바 애플리케이션 이름
```

##### 18.5.1. PrintGCTimeStamps 옵션  
<br/>

이렇게 옵션으 주고 수행하면 언제 GC가 발생되었는지 알 수 없다. 이 경우를 대비해서 verbosegc 옵션과 함께 사용할 수 있는 -XX:+PrintGCTimeStamps 옵션이 있다.  

##### 18.5.2. PrintHeapAtGC 옵션  
<br/>

또 다른 옵션으로 -XX:+PrintHeapAtGC라는 것이 있다. 이 옵션을 지정하면 GC에 대한 더 많은 정보를 볼 수 있지만, 너무 많은 내용을 보여주기 때문에 분석하기가 그리 쉽지는 않다. 그러나 툴로 분석할 대는 아주 상세한 결과도 확인할 수 있다.  

##### 18.5.3. PrintGCDetails  
<br/>

더 간결하고 보기 쉬운 옵션이다.  

### 19. GC 튜닝을 항상 할 필요는 없다.  
<br/>

#### 19.1. GC 튜닝을 꼭 해야 할까?  
<br/>

결론부터 이야기하면 Java 기반의 모든 서비스에서 GC 튜닝을 진행할 필요는 없다. GC 튜닝이 필요 없다는 이야기는 운영 중인 Java 기반 시스템의 옵션에 기본적으로 다음과 같은 것들은 추가되어 있을 때의 경우다.  

+ -Xms 옵션과 -Xmx 옵션으로 메모리 크기를 지정했다.
+ -server 옵션이 포함되어 있다.  

그리고 시스템의 로그에는 타임아아수 관련 로그가 남아있지 않아야 한다. 여기서 타임아웃은 다음과 같은 것들을 말한다.  

+ DB 작업과 관련된 타임아웃
+ 다른 서버와의 통신시 타임아웃  

타임아웃 로그가 존재하고 있다는 것은 그 시스템을 사용하는 사용자 중 대다수나 일부는 정상적인 응답을 받지 못했다는 말이다. 그리고, 대부분 서로 다른 서버간에 통신 문제나 원격 서버의 성능이 느려서 타임아웃이 발생할 수도 있지만, 그 이유가 GC 때문일 수도 있다.  

지금까지 이야기한 것을 정리하자면, JVM의 메모리 크기도 지정하지 않았고, Timeout이 지속적으로 발생하고 있다면 시스템에서 GC 튜닝을 하는 것이 좋다. 그렇지 않다면, GC 튜닝할 시간에 다른 작업을 하는 것이 더 낫다.  

그런데 한 가지 꼭 명심해야 하는 점이 있다. GC 튜닝은 가장 마지막에 하는 작업이라는 것이다.  

##### 19.1.1. Old 영역으로 넘어가는 객체의 수 최소화하기  
<br/>

Oracle JVM에서 제공하는 모든 GC는 Generational GC이다. 즉, Eden 영역에서 객체가 처음 만들어지고, Survivor 영역을 오가다가, 끝까지 남아 있는 객체는 Old 영역으로 이동한다(G1의 경우 약간 상이하게 동작한다.). 간혹 Eden 영역에서 만들어지다가 크기가 커져서 Old 영역으로 바로 넘어가는 객체도 있긴 하다. Old 영역의 GC는 New 영역의 GC에 비하여 상대적으로 시간이 오래 소요되기 때문에 Old 영역으로 넘어가는 객체의 수를 줄인다는 말을 잘못 이해하면 객체를 마음대로 New 영역에만 남길 수 있다고 생각할 수 있지만, 그렇게 할 수는 없다. 하지만 New 영역의 크기를 잘 조절함으로써 큰 효과를 볼 수는 있다.  

##### 19.1.2. Full GC 시간 줄이기  
<br/>

Full GC의 수행 시간은 상대적으로 Young GC에 비하여 길다. 그래서 Full GC 실행에 시간이 오래 소요되면(1초 이상) 연계된 여러 부분에서 타임아웃이 발생할 수 있다. 그렇다고 Full GC 실행 시간을 줄이기 위해서 Old 영역의 크기를 줄이면 OutOfMemoryError가 발생하거나 Full GC 횟수가 늘어난다. 반대로 Old 영역의 크기를 늘리면 Full GC 횟수는 줄어들지만 실행 시간이 늘어난다. Old 영역의 크기를 적절하게 잘 설정해야 한다.  

#### 19.2. GC의 성능을 결정하는 옵션들  
<br/>

<table>
    <thead>
        <tr>
            <th>구분</th>
            <th>옵션</th>
            <th>설명</th>
        </tr>
    </thead>
    <tboyd>
        <tr>
            <td rowspan="2">힙(heap) 영역 크기</td>
            <td>-Xms</td>
            <td>JVM 시작 시 힙 영역 크기</td>
        </tr>
        <tr>
            <td>-Xmx</td>
            <td>최대 힙 영역 크기</td>
        </tr>
        <tr>
            <td rowspan="3">New 영역의 크기</td>
            <td>-XX:NewRatio</td>
            <td>New 영역과 Old 영역의 이름</td>
        </tr>
        <tr>
            <td>-XX:NewSize</td>
            <td>New 영역의 크기</td>
        </tr>
        <tr>
            <td>-XX:SurvivorRatio</td>
            <td>Eden 영역과 Survivor 영역의 비율</td>
        </tr>
    </tbody>
</table>
<br/><br/>

Perm 영역의 크기는 OutOfMemoryError가 발생하고, 그 문제의 원인이 Perm 영역의 크기 때문일 때에만 -XX:PermSize 옵션과 -XX:MaxPermSize 옵션으로 지정해도 큰 문제는 없다.  

GC의 성능에 많은 영향을 주는 또 다른 옵션은 GC 방식이다.  

<table>
    <thead>
        <tr>
            <th>구분</th>
            <th>옵샨</th>
            <th>비고</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Serial GC</td>
            <td>-XX:+UseSerialGC</td>
            <td></td>
        </tr>
        <tr>
            <td>Parallel GC</td>
            <td>
                -XX:+UseParallelGC<br/>>
                -XX:ParallelGCThreads=value
            </td>
            <td></td>
        </tr>
        <tr>
            <td>Parallel Compacting GC</td>
            <td>-XX:+UseParallelOldGC</td>
            <td></td>
        </tr>
        <tr>
            <td>CMS GC</td>
            <td>
                -XX:+UseConcMarkSweepGC<br/>
                -XX:+UseParNewGC<br/>
                -XX:+CMSParallelRemarkEnabled<br/>
                -XX:CMSInitiatingOccupancyFraction=value<br/>
                -XX:+UseCMSInitiatingOccupancyOnly
            </td>
            <td></td>
        </tr>
        <tr>
            <td>G1</td>
            <td>
                -XX:+UnlockExperimentalVMOptions<br/>
                -XX:+UseG1GC
            </td>
            <td>JDK 6에서는 두 옵션을 반드시 같이 사용해야 함</td>
        </tr>
    </tbody>
</table>
<br/><br/>

G1 GC를 제외하고는, 각 GC 방식의 첫 번째 줄에 있는 옵션을 지정하면 GC 방식이 변경된다. GC 방식 중에서 특별히 신경 쓸 필요가 없는 방식은 Serial GC다. Serial GC는 클라이언트 장비에 최적화되어 있기 때문이다.  

#### 19.3. GC 튜닝의 절차  
<br/>

(1) GC 상황 모니터링  
(2) 모니터링 결과 분석 후 GC 튜닝 여부 결정  
(3) GC 방식/메모리 크기 지정  
(4) 결과 분석  
(5) 결과가 만족스러울 경우 전체 서버에 반영 및 종료  

