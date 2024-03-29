# Ch6. 가비지 수집 기초

## 마크 앤 스위프
1. 할당 리스트를 순회하면서 마크비트를 제거
2. GC 루트부터 살아 있는 객체 탐색
3. 찾은 객체에 대한 마크 비트 세팅
4. 할당 리스트를 순회하면서 마크 비트가 세팅되지 않은 객체 탐색
    - 힙에서 메모리 회수 후 프리 리스트(메모리에 할당되지 않은 영역을 링크드 리스트로 연결)에 되돌린다.
    - 할당 리스트에서 객체를 삭제

### GC 용어
- **STW(Stop The World)**
  - GC가 가비지 수집을 진행하는 동안 모든 어플리케이션 스레드가 중단

- **동시**
  - GC 스레드는 어플리케이션 스레드와 동시(병행) 실행될 수 있다.
  - 사실 상 100% 동시 실행을 보장하는 GC 알고리즘은 존재X

- **병렬**
  - 여러 스레드를 동원하여 가비지 수집 진행

- **정확**
  - 정확한 가비지 수집을 한번에 진행할 수 있게 힙 상태에 대한 충분한 타입 정보를 가지고 있음

- **보수**
  - 보수적인 스킴은 정확한 스킴의 정보가 없어 리소스를 낭비하는 일이 잦고 근본적을 타입 체계를 무시하기 때문에 비효율적

- **이동**
  - 이동 수집기에서 객체는 메모리를 여기저기 오갈 수 있다. -> 객체 주소 고정X

- **압착**
  - 할당된 메모리(살아남은 객체들)은 GC 사이클 마지막에 연속된 단일 영역으로 배열됨

- **방출**
  - 수집 사이클 마지막에 할당된 여역을 완전히 비우고 살아남은 객체는 모두다른 메모리 영역으로 이동

## 핫스팟 런타임 개요

### GC 루트 및 아레나
- GC 루트는 메모리 고정점으로 메모리 풀 외부에서 내부를 가리키는 포인터.
- 핫스팟 GC는 **아레나**라는 메모리 영역에서 작동
- **핫스팟은 자바 힙을 관리할 때 시스템 콜을 하지 않는다.**

## 할당과 수명 
- 자바 어플리케이션에서 가비지 수집이 일어나는 주된 원인
  1. 할당률
  2. 객체 수명

- 할당률
  - 일정 기간 새로 생성된 객체가 사용한 메모리량
  - JVM이 할당률을 직접 기록하진 않지만, 비교적 쉽게 측정 가능(툴 이용 - 센섬)
- 객체 수명
  - 대부분 측정하기 어려움
  - 할당률보다 더 핵심적인 요인

### 약한 세대별 가설
- 약한 세대별 가설: JVM 및 유사 소프트웨어 시스템에선 거의 대부분의 객체는 아주 짧은 시간만 살아 있지만, 나머지 객체는 기대 수명이 훨씬 길다.
  -> 가비지를 수집하는 힙은, 단명 객체는 쉽고 빠르게 수집해야 하며 장수 객체와 단명 객체를 완전히 떼어놓는 게 좋다.
- 핫스팟 메커니즘
  - 객체마다 세대 카운트(객체가 가비지 수집에서 살아남은 횟수)
  - 큰 객체를 제외한 나머지 객체는 에덴 공간에 생성. 여기서 살아남은 객체는 다른 곳으로 이동
  - 충분히 오래 살아남은 객체들은 별도의 메모리 영역(올드 or 테뉴어드 generation)에 보관
- **자바17까지도 G1GC가 default이기 때문에 G1GC에 대한 추가 학습 필요**

## 할당의 역할
- GC 사이클은 하나 이상의 힙 메모리 공간이 가득 차 더 이상 객체를 생성할 공간이 없을 때 일어난다. -> GC는 불확정적으로, 불규칙하게 발생하는 것이 가장 중요한 특징
- 가비지 수집 로그는 기존의 시계열 해석 방법으로는 처리하기 어렵다.
- 할당률이 높을수록 GC는 더욱 자주 발생하고, 할당률이 지나치게 높을 경우 객체는 어쩔 수 없이 테뉴어드로 바로 승격된다. 이러한 현상을 조기 승격이라한다. -> **가비지 수집에서 가장 중요한 간접 효과이자 튜닝 활동의 출발점**
