# Ch10. JIT 컴파일의 세계로

## JIT Watch
- 오픈소스 자바FX 툴
- 어플리케이션 실행 중 핫스팟이 실제로 바이트코드에 무슨 일을 했는지 이해할 수 있는 정보 제공
- 실행 중인 자바 어플리케이션이 생성한 핫스팟 컴파일 상세 로그를 파싱/분석하여 결과를 자바FX GUI형태로 보여준다
- `-XX:+UnlockDiagnosticVMOptions -XX:+TraceClassLoading -XX:+LogCompliation`

### 기본적인 JITWatch
- 실행시킨 프로그램의 로그를 적재하는 것 뿐만 아니라, JIT 작동을 시험해볼 수 있는 샌드박스 환경 제공
- JITWatch의 3단뷰는 소스코드가 바이트코드, 어셈블리 양쪽으로 어떻게 컴파일 되었는지 알 수 있는 뷰를 제공한다.

## JIT 컴파일 개요
- **핫스팟JIT 컴파일의 컴파일러 최적화 기법**
  1. 인라이닝
  2. 루프 펼치기
  3. 탈출 분석
  4. 락 생략/확장
  5. 단일형 디스패치
  6. 인트린직

## 인라이닝
- 제일 먼저 적용하는 최적화, 관문 최적화라고도 함
- 인라이닝은 호출된 메서드의 콘텐츠를 호출한 지점에 복사하는 것
- 다음과 같은 오버헤드 제거
  - 전달할 매개변수 세팅
  - 호출할 메서드를 정확하게 룩업
  - 새 호출 프레임에 맞는 런타임 자료구조 생성
  - 새 메서드로 제어권 이송
  - 호출부에 결과 반환(결괏값이 있는 경우)
- 핫스팟이 메서드 인라이닝을 결정하는 항목
  - 인라이닝할 메서드의 바이트 크기
  - 현채 호출 체인에서 인라이닝할 메서드의 깊이
  - 메서드를 컴파일한 버전이 코드 캐시에서 차지하는 공간

## 루프 펼치기
- 루프 내부의 메서드가 호출을 전부 인라이닝할 경우, 컴파일러는 루프 순회시 마다 비용이 얼마나 드는지, 반복 실행되는 코드는 크기가 얼마나 되는지 분명히 할 수 있음 -> 매 순회때마다 루프 처음으로 돌아가는 횟수를 줄이기 위해 루프를 펼친다.
- 루프 펼치기 여부 기준
  - 루프 카운터 변수 유형(대부분 객체 아닌 int, long 사용)
  - 루프 보폭(한번 순회 시마다 루프 카운터 값이 얼마나 바뀌는가)
  - 루프 내부의 탈출 지점 개수(return break)
- 루프 펼치기는 핫스팟 버전별로 로직이 상이하고 아키텍처마다 많이 다르다.
- 핫스팟은 루프를 돌며 배열에 엑세스할 떄 루프를 세 구역으로 나누어 배열 경계 검사를 실행
  - 사전 루프
  - 메인 루프
  - 사후 루프
- 핫스팟은 카운터가 int, short, char일 경우 루프 최적화 진행

## 탈출 분석
- 어떤 메서드가 내부에서 수행한 작업을 그 메서드 경계 밖에서도 볼 수 있는지, 혹은 사이드 이펙트를 유발하지 않는지 범위 기반 분석을 통해 판별하는 것
- 반드시 인라이닝을 수행한 이후 시도
- 탈출 분석의 목표는 힙 할당을 막을 수 있는지 추론하는 것 -> 힙 할당을 막을 수 있다면, 객체는 스택에 자동 할당되고 GC 압박을 덜 수 있음

### 락과 탈출 분석
- 핫스팟은 탈출 분석 및 관련 기법을 통해 락 성능 역시 최적화한다.(단, 해당 최적화는 synchronized를 사용한 인스린직 락에만 해당, java.util.concurrent 패키지의 락에는 적용X)
- 락 최적화 핵심
  - 비탈출 객체에 있는 락은 제거(**락 생략**)
  - 같은 락을 공유한, 락이 걸린 연속된 영역은 병합(**락 확장**)
  - 락을 해제하지 않고 같은 락을 반복 획득한 블록을 찾아냄(**중첩 락**)

### 탈출 분석의 한계
- 힙이 아니더라도 다른 어딘가에 할당을 해야 하지만 CPU 레지스터나 스택 공간은 상대적으로 희소한 리소스이다.
- 원소 개수가 64개 이상인 배열은 핫스팟에서 탈출 분석의 혜택을 볼 수 없다.

## 단형성 디스패치
- 어떤 객체에 있는 메서드를 호출할 때, 그 메서드를 최초로 호출한 객체의 런타임 타입을 알아내면 그 이후의 모든 호출도 동일한 타입일 가능성이 크다는 가정
- 해당 가정에 따라 호출부의 메서드 호출을 최적

## 인트린직
- JIT 서브시스템이 동적 생성하기 전 JVM이 이미 알고 있는, 고도로 튜닝된 네이티브 메서드 구현체를 가리키는 용어
- 주로 OS, CPU 아키텍처의 특정 기능을 응요하는 등 성능이 필수적인 코어 메서드에 쓰인다.

## 온-스택 치환
- 컴파일을 일으킬 정도로 호출 빈도가 높지는 않지만 메서드 내부에 핫 루프가 포함된 경우 온-스택 치환을 이용해 최적화 한다.(ex. main()메서드)

## 세이브포인트 복습
- JVM에 세이브포인트가 걸리는 조건 
  - GC STW
  - 메서드 역최적화
  - 힙 덤프 생성
  - 바이어스 락을 취소
  - 클래스 재정의
  - 루프 백 브랜치 지점
  - 메서드 반환 지점
