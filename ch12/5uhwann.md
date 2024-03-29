# Ch12. 동시 성능 기법

### JMM의 이해
- 자바1.0 버전부터 공식적으로 공개된 메모리 모델 -> 자바5와 함께 배
- 두 가지 방식으로 접근
  1. 강한 메모리 모델: 전체 코어가 항상 같은 값을 바라본다.
  2. 약한 메모리 모델: 코어마다 다른 값을 바라볼 수 있고, 그 시점을 제어하는 특별한 캐시 규칙이 존재한다.
- JMM은 아주 약한 메모리 모델
- 어플리케이션 보호를 위한 JMM의 기본 개념
  - Happens-Before
    - 한 이벤트는 무저건 다른 이벤트보다 먼저 발생한다.
  - Synchronized-With
    - 이벤트가 객체 뷰를 메인 메모리와 동기화시킨다.
  - As-If-Serial
    - 실행 스레드 밖에서는 명령어가 순차 실행되는 것처럼 보인다.
  - Release-Before-Acquire
    - 한 스레드에 걸린 락을 다른 스레드가 그 락을 획득하기 전에 해제한다.
- 동기화를 통한 락킹은 가변 상태를 공유하는 가장 중요한 기법이다.
- 자바에서 스레드는 객체 상태 정보를 스스로 들고 다니며, 스레드가 변경한 내용을 메인 메모리로 곧장 반영되고 같은 데이터를 액세스하느 다른 스레드가 다시 읽는 구조이다.

## 동시성 라이브러리 구축
- java.util.concurrent 패키지: 멀티 스레들 어플리케이션을 자바로 더 쉽게 개발할 수 있게 설계된 ㄹ아이브러리
- 핵심 요소
  - 락, 세마포어
  - 아토믹스
  - 블로킹 큐
  - 래치
  - 실행자(executor)
  
### Unsafe
- Unsafe로 할 수 있는 일
  - 객체는 할당하지만 생성하지 않는다.
  - 원메모리에 엑세스하고 포인터 수준의 연산을 수행한다.
  - 프로세서별 하드웨어 특성을 이용한다.
- 구현 가능한 고수준 프레임워크 기능 
  - 신속한 (역)직렬화
  - 스레드 안전한 네이티브 메모리 엑세스
  - 아토믹 메모리 연산
  - 효율적인 객체/메모리 레이아웃
  - 커스텀 메모리 펜스
  - 네이티브 코드와의 신속한 상호작용
  - JNI에 관한 다중 운영체제
  - 배열 원소에 violate하게 엑세스
- 자바 17부터 `java.lang.invoke.MethodHandles.Lookup::defineHiddenClass` 및 `java.lang.invoke.MethodHandles.Lookup::defineHiddenClassWithClassData`로 API 이동

### 락과 스핀락
- 스핀락: 블로킹된 스레드를 CPU에 활성 상태로 놔두고 아무 일도 시키지 않은 채 락을 손에 넣을 때까지 CPU를 태워가며 계속 재시도하는 기법
  - 완전 상호 배타적 락보다 가볍게 사용 가능

## 동시 라이브러리 정리
### java.util.concurrent 락
- lock(): 기존 방식대로 락을 획득하고 사용할 수 있을 때까지 블로킹
- newCondition(): 락 주위에 조건을 설정해 좀 더 유연하게 락을 활용, 락 내부에 관심사 분리
- tryLock(): 락을 획득하료 시도
- unLock(): 락을 해제, lock()에 대응되는 후속 호출

### 읽기/쓰기 락
- ReentrantReadWriteLock 클래스의 ReadLock, WriteLock을 활용하면 여러 스레드가 읽기 작업을 하는 도중에도 다른 읽기 스레드를 블로킹하지 않게 할 수 있다.

## 최신 자바 동시성
### 스트림과 병렬 스트림
- Collection 인터페이스의 parallelStream()을 이용해 병렬려 데이터를 작업 후 그 결과를 재조합 할 수 있는 스트림을 생성할 수 있다.
- 스트림은 원래 불변이기 때문에 병렬 실행을 수행하더라도 상태 변경으로 생기는 문제를 예방할 수 있다.
- 태스크를 찢어 여러 스레드에 분배하고 그 결과를 다시 취합하는 일은 피할 수 없기 때문에 컬렉션이 작을수록 직렬 연산이 병렬 연산보다 훨씬 빠르다.

## 정리
- 싱글 스레드 어플리케이션을 동시성 기반의 설계 방식을 전환할 때는 다음을 고려
  - 순서대로 죽 처리하는 성능을 정확히 측정할 수 있어야 한다.
  - 변경을 적용한 다음 진짜 성능이 향상되었는지 확인한다.
  - 성능 테스트는 재실행하기 쉬어야 한다.
- 다음의 유혹을 피한다.
  - 병렬 스트림을 곳곳에 가져다 쓴다.
  - 수동을 락킹하는 복잡한 자료 구조를 생성한다.
  - java.uti.concurrent에 이미 있는 구조를 다시 만든다.
- 다음의 목표를 설정한다.
  - 동시 컬렉션을 이용해 스레드 핫 성능을 높인다.
  - 하부 자료 구조를 최대한 활용할 수 있는 형태로 엑세스를 설계한다.
  - 어플리케이션 전반에 걸쳐 락킹을 줄인다.
  - 가급적 스레드를 직접 처리하지 않도록/비동기를 적절히 추상화 한다.
