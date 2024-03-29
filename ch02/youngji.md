> 자바 개발자가 플랫폼을 저수준에서 다 알필요 없도록 설계되어 있지만, 성능에 관심이 있다면 기본적인 JVM을 이해해야 한다.
> 

> JVM 기술 스택의 구조와 JVM이 자바 코드를 실행하는 법을 학습한다.
> 

---

## 1. 인터프리팅과 클래스로딩

JVM은 스택 기반의 해석 머신이다.

물리적인 하드웨어인 레지스터는 없지만 일부 결과를 실행 스택에 보관하며, 스택의 맨위에 쌓인 값들을 가져와 계산한다. 

JVM **인터프리터(해석기)**의 기본 로직은, 평가 스택을 이용해 중산값들을 담아두고 가장 마지막에 실행된 명령어와 독립적으로 프로그램을 구성하는 옵코드를 하나씩 순서대로 처리하는 ‘while 루프 안의 switch문’이다. (실제품급 자바 인터프리터는 이보다 훨씬 복잡하지만, 지금은 while 루프 문안에 switch문 정도의 이미지를 떠올리면 이해하기 쉽다.)

```bash
java HelloWorld
```

명령어를 입력하여 자바 어플리케이션을 실행해보자

1. OS는 가상 머신 프로세스(자바 바이너리)를 구동한다.
2. 자바 가상 환경이 구성되고 스택 머신이 초기화된 다음, 사용자가 작성한 HelloWorld 클래스 파일이 실행된다.

애플리케이션의 진입점은 HelloWorld.class에 있는 main() 메소드이다.

제어권을 이 클래스로 넘기려면 가상 머신이 실행되기 전에 이 클래스를 로드해야한다. 이때 자바 **클래스로딩 메커니즘**이 관여한다.

자바 프로세스가 새로 초기화되면 사슬처럼 줄지어 연결된 클래스 로더가 차례로 동작하게 된다. 

1. 제일 먼저 **부트스트랩 클래스**가 동작하며 자바 런타임 코어 클래스를 로드한다.
    
    부트스트랩 클래스로더의 주임무는 다른 클래스로더가 나머지 시스템에 필요한 클래스를 로드할 수 있게 최소한의 필수 클래스(Object, Class, Classloader 등..)만 로드하는 것이다. 
    
    → 런타임 코어 클래스는 자바8 이전까지는 rt.jar 파일에서 가져오지만, 자바9 이후부터는 런타임이 모듈화되고 클래스로딩 개념 자체가 많이 달라졌다.
    
    → 자바에서 Classloader는 런타임과 타입체계 내부에 있는 하나의 객체에 불과하다. 따라서 클래스 초기 세트가 존재하게 할 방법이 없다. (그걸 해주는 것이 부크스트랩 클래스) 그렇지 않으면 클래스로더를 정의하는 과정에서 순환오류 문제가 발생할 것이다.
    
2. 그 다음, **확장 클래스로더**가 생긴다.
    
    확장 클래스로더는 부트스트랩 클래스로더를 자기 부모로 설정하고 필요할때 클래스로딩 작업을 부모에게 넘긴다.
    
3. 끝으로 확장 클래스로더의 자식인 **애플리케이션 클래스로더**가 생성되고 지정된 클래스패스에 위치한 **유저 클래스**를 로드한다.

자바는 프로그램 실행 중 처음 보는 새 클래스를 **티펜던시(의존체)**에 로드한다. 클래스를 찾지 못한 클래스로더는 기본적으로 자신의 부모 클래스로더에게 대신 룩업을 넘긴다. 부모로 거슬러 올라가 결국 부트스트랩도 룩업을 하지 못하면 ClassNotFoundException 예외가 발생한다. 

때문에 빌더 프로세스 수립 시 운영 환경과 동일한 클래스패스로 컴파일하는 것이 좋다.

한 시스템에서 클래스는 패키지명을 포함한 풀 클래스명과 자신을 로드한 클래스로더, 두가지 정보로 식별된다. (똑같은 클래스를 상이한 클래스로더가 두 번 로드할 가능성이 있으니 주의한다.)

---

## 2. 바이트코드 실행

자바 소스 코드가 실행되는 첫 단계는 자바 컴파일러 javac를 이용해 컴파일하는 것으로, 자바 소스 코드를 바이트코드로 가득 찬 .class 파일로 바꾸는 것이다.

(native code는 OS상에서 직접 컴파일하여 바로 기계로 실행 가능한 반면 인터프리터가 반드시 있어야 실행되는 코드를 managed code라고 한다.)

javac는 컴파일하는 동안 최적화는 거의 하지 않기 때문에 그 결과로 생성된 바이트코드는 쉽게 해독할 수 있다.

(javap 같은 표준 역어셈블리 툴로 열어보면 원래 자바 코드도 어렵지 않게 알아볼 수 있다.)

→ 바이트코드는 특정 컴퓨터 아키텍쳐에 특정하지 않은, 중간 표현형이다. 컴퓨터 아키텍쳐에 종속적이지 않아 이식성이 좋아 개발을 마친(컴파일된) 소프트웨어는 JVM 지원 플랫폼 어디든 실행이 가능하다.

컴파일러가 생성한 클래스파일은 아주 명확하게 잘 정의된 구조를 갖춘다.

| entry | 설명 |
| --- | --- |
| 매직넘버 | 0XCAFEBABE : 해당 파일이 클래스 파일임을 나타내는 4바이트 16진수로 시작한다. |
| 클래스 파일 포맷 버전 | 그 다음 4바이트는 클래스 파일을 컴파일할때 필요한 메이저/마이너 버전 숫자이다. 대상 JVM 버전이 컴파일한 JVM 버전보다 낮을 경우 호환되지 않아 런타임에 UnsupportedClassVersionError 예외가 발생한다. |
| 상수 풀 | 상수값. 클래스명, 인터페이스명, 필드명 등등
JVM은 코드를 실행할 때 런타임에 배치된 메모리 대신, 해당 상수 풀 테이블을 찾아보고 필요한 값을 참조한다. |
| 액세스 플래그 | 클래스에 적용한 수정자를 결정한다.
ACC_PUBLIC(0x0001), ACC_FiNAL(0x0010), ACC_SUPER(0x0020), ACC_INTERFACE(0x0200), ACC_ABSTRACT(0x0400), ACC_SYNTHETIC(0x1000), ACC_ANNOTATION(0x2000), ACC_ ENUM(0x4000) |
| this 클래스 |  |
| 슈퍼클래스 |  |
| 인터페이스 | 위 세가지 엔트리는 클래스에 포함된 타입 계층을 나타내며, 각각 상수 풀을 가르키는 인덱스로 표시된다. |
| 필드 |  |
| 메소드 | 필드와 메소드는 시그니처 비슷한 구조를 정의하고 수정자도 포함한다. |
| 속상 | 더 복잡하고 크기가 고정되어 있지 않은 구조를 나타내는 데 쓰인다.  |

---

## 3. 핫스팟 입문

성능 관점에서 자바에 가장 큰 변화를 가져왔던것은 핫스팟 가상 머신이다.

핫스팟을 처음 선보인 이후로 C/C++ 같은 언어에 필적할 만한 성능을 자랑하며 진화를 거듭했다.

언어 및 플랫폼 설계 과정은 ‘기계어에 가까운 언어’와 개발자의 생산성에 무개를 둔 ‘일을 대행하는 언어’ 사이에서의 갈등을 겪게 된다.

제로-오버헤드 원칙은 이론적으로는 그럴싸하지만 아래와 같은 문제가 있다.

→ 개발자가 아주 세세한 저수준까지 알아야하는 학습 부담이 있다.

→ 특정 플랫폼에서만 사용가능한 기계어로 컴파일 된다. (`**사진(AOT) 컴파일**`)

자바는 이러한 제로-오버헤드 추상화 철학에 한번도 동조한 적이 없으며, 오히려 핫스팟은 프로그램의 런타임 동작을 분석하고 성능에 유리한 방향으로 최적화를 적용하는 가상 머신이다. 

핫스팟 VM의 목표는 개발자가 억지로 VM 틀에 맞게 프로그램을 욱여넣는 대신, 자연스럽게 자바 코드를  작성하고 바람직하게 설계원리를 따르도록 한다.

- JIT 컴파일
    
    프로그램이 성능을 최대로 내려면 네이티브 기능을 활용하여 CPU에서 직접 프로그램을 실행시켜야한다. 이를 위해 핫스팟은 프로그램을 인터프리티 바이트코드에서 네이티브 코드로 컴파일한다. 
    
    → `**적시(JIT) 컴파일**`
    
    JIT 컴파일을 수행하는 과정에서 컴파일러가 해석단계에서 미리 정보를 수집하여 추적 정보를 근거로 최적화를 결정한다는 것이 가장 큰 장점이다.
    

---

## 4. JVM 메모리 관리

개발자가 직접 메모리를 관리하면 좀 더 확정적인 성능을 낼 수 있고 리소스 수명을 관리할 수 있다는 장점이 있지만, 그만큼 반드시 개발자가 메모리를 정확하게 계산해야 한다는 막중한 책임이 수반된다.

자바는 **가비지수집**을 이용해 힙 메모리를 자동 관리하는 방식으로 이러한 문제를 해결한다. 

가비지 수집이란 JVM이 더 많은 메모리를 할당해야 할때 불필요한 메모리를 회수하거나 재사용하는 불확정적 프로세스이다.

CG의 작업은 결코 간단하지 않고 자바 역사를 통틀어 온갖 알고리즘이 개발/응용되었다. 일단 CG가 실행되면 그동안 다른 애플리케이션은 모두 중단되고 하던일을 멈춰야한다. 이 중단 대기 시간은 대개 아주 짧지만, 부화가 늘수록 이 시간도 무시할 수는 없다. ****

---

## 5. 스레딩과 자바 메모리 모델(JMM)

자바는 1.0부터 멀티스레드 프로그래밍을 기본 지원했다.

주류 JVM 구현체에서 자바 어플리케이션 스레드는 각각 하나의 전용 OS 스레드에 대응된다.

공유 스레드 풀을 이용해 전체 자바 어플리케이션 스레드를 실행하는 방안(그린 스레드)도 있지만, 복잡하고 만족할 만한 수준의 성능이 나오지 않아 보통 주류를 따른다.

자바의 멀티스레드 방식 기본 설계 원칙

1. 자바 프로세스의 모든 스레드는 가비지가 수집되는 하나의 공동 힙을가진다.
2. 한스레드가 생성한 객체는 그 객체를 참조하는 다른 스레드가 액세스 할 수 있다.
3. 기본적으로 객체는 변경 가능하다. (명시적 final 키워드를 표시하지 않는 이상)

JMM은  서로 다른 실행 스레드가 객체 안에 변경되는 값을 어떻게 바라보는지 기술한 공식 메모리 모델이다. 

두개의 스레드가 하나의 객체를 참조할때 하나의 스레드가 해당 값을 바꾸면 어떻게 될까? → 상호 배타적 락은 코드가 동시에 실행되는 도중 객체가 손상는 현상을 막을 수 있지만, 실제 어플리케이션에 사용하려면 상당히 복잡해질 수 있다. 

---

## 6. JVM 구현체 종류

오라클이 제작한 핫스팟 이외에도 여러 자바 구현체가 있다. 

OpenJDK, Oracle, Zulu,..

---

## 7. JVM 모니터링 툴링

JVM은 실행 중인 어플리케이션을 인스트루먼데이션, 모니터링, 관측하는 다양한 기술을 제공한다.

- 자바 관리 확장 (JMX)
- 자바 에이전트
- JVM 툴 인터페이스 (JVMTI)
- 서비서빌리티 에이전트 (SA)

javac, java 등 잘 알려진 바이너리뿐 아니라 JDK에는 유용한 가외 툴이 많다.

- VisualVM
    
    : 넷빈즈 플랫폼 기반의 시각화 툴
