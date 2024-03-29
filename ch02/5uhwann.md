# Ch2. JVM 이야기

## 인터프리팅과 클래스로딩
- JVM은 스택 기반 해석 머신
  - 인터프리터의 기본 로직은 마지막에 실행된 명령어를 순서대로 처리하는 **while 루프 안의 switch문**
- 자바 어플리케이션의 진입점인 `main()` 메서드에 프로그램의 제어권을 넘기기 위해서는 클래스 로딩이 필요하다.
  - **자바 클래스 로딩 매커니즘**
    1. 부트스트랩 클래스가 자바 런타임 코어 클래스 로드(java.lang, java.util, java.io ...)
    2. 확장 클래스 로더 생성
    3. 어플리케이션 클래스 로더 생성
  - 한 시스템에서 클래스는 풀 클래스명(패키지명 포함)과 자신을 로드한 클래스로더, 두 가지 정보로 식별됨

## 바이트 코드 실행
- javac(자바 컴파일러)의 자바 소스 코드 컴파일. 소스코드 -> .class 파일
  - **클래스 파일 구조**
    1. Magic Number
    2. Version
    3. Constant Pool
    4. Access Plug
    5. this Class
    6. SuperClass
    7. Interface
    8. field
    9. method
    10. attribute

## 핫스팟 입문
### JIT 컴파일
- 자바 프로그램이 성능을 최대로 내기 위해 바이트 코드를 네이티브 코드로 컴파일 기법
- 어플리케이션을 모니터링 하면서 가장 자주 실행되는 코드 파트를 발견해 JIT 컴파일 수행
- 컴파일러가 해석 단계에서 수집한 추적 정보를 바탕으로 최적화를 결정
- 자바 소스코드와 실제로 JIT컴파일 후 실행되는 코드는 원본 코드와 전혀 다른 모습

## JVM 메모리 관리
- Garbage Collection을 이용해 힙 메모리를 자동관리
- **자바 성능 최적화의 중심 주제**

## JVM 모니터링과 툴링
- JMX
  - JVM 기반 어플리케이션 제어, 모니터링 툴
- 자바 에이전트
  - 자바 언어로 작성된 툴 컴포넌트
  - java.lang.instrument 인터페이스를 통해 메서드 바이트코드 조작
- VisualVM
  - 자바 어플리케이션 시각화 툴
