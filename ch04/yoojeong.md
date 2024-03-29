
> 성능 테스트는 소프트웨어와 인프라의 속도를 테스트한다. 메모리 사용량, 네트워크 대역폭 및 처리량, 응답 시간, CPU 사용률 등의 기준을 두고 측정할 수 있다. 시스템 내에 통신 과정의 병목을 찾아내기 위해서 진행 한다.

### 성능 테스트 유형

![image](https://github.com/rachel5004/24-optimizing-java-2/assets/75432228/4b9ab298-56f5-40d4-83c0-3fb11f25c309)


|TEST|DESCRIPTION|방법|
|:--:|:--|:--|
|지연 테스트(Latency Test)| 종단 트랜잭션에 걸리는 시간 측정. 평균값은 사실 큰 의미가 없고 가장 느린 응답을 확인해야함 ||
|처리율 테스트(Throughput Test)|지연 분포가 갑자기 변하는 시점(한계점,변곡점)이 사실상 최대 처리율||
|부하 테스트(Load Test)| 일정 시간 동안 부하를 가해 서버가 처리할 수 있는 최대 TPS와 응답시간을 측정 |순차적으로 증가하는 부하를 제한 시간동안 부여. 한시간 이내|
|내구 테스트(Endurance Test)|메모리릭, 캐시오염, 메모리단편화 등 시간이 지나고 드러나는 문제점 파악|특정한 부하를 오랜 시간동안 부여. 5시간 이상|
|스트레스 테스트(Stress Test)| 시스템의 한계점 파악|임계값 이상의 부하를 제한 시간동안 부여|
|스파이크 테스트(Spike Test)| 순간적으로 사용자 수가 증가했다 감소할 때 동작 테스트(AutoScale의 정상처리 등..)|일반적인 트래픽을 크게 초과하는 부하를 급격하게 부여, 감소|

![image](https://github.com/rachel5004/24-optimizing-java-2/assets/75432228/5ab7e850-b029-4014-a2e7-a26c9326b154)



- 용량계획 테스트(Capacity Planning Test) : 리소스를 업그레이드하면 어느정도 성능이 늘어나는가?
- 저하 테스트(Degradation Test) : 시스템이 부분적으로 실패할 경우 어떤 일이 벌어지나?(잘 복원되나?)


#### 주요 측정 항목

`응답`
- Average ResponseTime
- Max ResponseTime
- Error rate

`처리량`
- Concurrent users: 일반적으로 테스트의 virtual user
- Throughput: 주로 TPS 를 사용하고, 대상 리소스별로 지칭하는 용어가 별도 존재한다.
  - CPU : MIPS, MFLOPS
  - Network : BPS, pps
  - Server : tpmC
  - C/S, TP-Monitor, Mainframe : TPS
  - Storage : IOPS


### 기본 베스트 프랙티스

- 테스트 환경은 운영 환경과 비슷하게 구축되어야 한다. (인프라 스펙, 데이터셋 등)
- 성능 비기능 요건(최적화하려는 핵심 지표)을 구성원들과 잘 협의해야 한다.
- 소프트웨어개발주기(SDLC) 의 일부에 포함 시켜 상시로 성능 회귀 테스트를 진행한다.
- 자바에서 특정해 발생하는 이슈에 주의한다.

#### Java의 특정 이슈

메모리 영역 동적 튜닝 등 JVM 특유의 다이나믹한 자기관리 때문에 복잡도가 올라갈 수 있다.
(ex. JIT 컴파일의 대상이 되는 메서드)

- JIT 컴파일 할정도로 자주 실행되는 메서드가 아니거나
- 메서드가 너무 크고 복잡해서 도저히 컴파일 분석을 할 수 없는 경우

JIT 컴파일 대상이 되지 않을 수 있는데, JVM 이 어플리케이션의 중요 메서드를 JIT 컴파일 타깃으로 만드는 것이 중요

9장에서 상세하게 나옴

### 결론

직접 테스트 해보고 병목 지점을 파악해라
