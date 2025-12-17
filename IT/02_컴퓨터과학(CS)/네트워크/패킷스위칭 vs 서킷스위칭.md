---
title: 패킷스위칭 vs 서킷스위칭
date: 2025-12-11
tags:
  - it
  - CS
  - 네트워크
  - 스위칭
---

## 네트워크 코어 개요

**네트워크 코어**는 라우터들로 구성된 중앙 네트워크 영역으로, 데이터를 송신자에서 수신자까지 전달하는 역할.

데이터 전송 방식은 크게 두 가지로 나뉨:

- **패킷 스위칭 (Packet Switching)**: 데이터를 작은 패킷으로 나누어 전송
- **서킷 스위칭 (Circuit Switching)**: 전용 회선을 설정하여 전송

---

## 패킷 스위칭 (Packet Switching)

### 기본 원리

패킷 스위칭은 데이터를 작은 단위인 **패킷**으로 나누어 전송하는 방식. 각 패킷은 독립적으로 라우팅되어 목적지에 도착.

### 패킷을 쪼개는 이유

1. **에러 복구 비용 절감**
    
    - 큰 데이터 덩어리에서 에러 발생 시 전체를 재전송해야 함
    - 작은 패킷 단위로 나누면 에러가 발생한 패킷만 재전송
2. **공평한 자원 사용**
    
    - 큰 패킷이 전송 중이면 다른 데이터는 오래 대기해야 함
    - 작은 패킷으로 나누면 여러 사용자가 공평하게 네트워크 사용 가능
3. **표준 규격 준수**
    
    - 이더넷 표준: 최대 1,500바이트 (MTU: Maximum Transmission Unit)
    - IPv4: 라우터에서 필요시 패킷 분할 (Fragmentation) 가능
    - IPv6: 패킷 분할 기능 제거 (성능 향상을 위해 송신 측에서 처리)

### 라우터의 역할

#### 1. 포워딩 (Forwarding)

- **정의**: 라우터 내부의 로컬 작업
- **기능**: 들어온 패킷을 적절한 출력 링크로 전송
- **동작**: 라우팅 테이블을 참조하여 패킷의 목적지 IP 주소를 확인하고 다음 홉(hop) 결정

#### 2. 라우팅 (Routing)

- **정의**: 네트워크 전체의 글로벌 작업
- **기능**: 최적의 경로를 찾아 라우팅 테이블 생성
- **동작**: 라우터들이 서로 정보를 교환하여 네트워크 토폴로지 파악

**IP 주소 구조**:

- 32비트 주소 (IPv4 기준)
- 네트워크 부분: 목적지 네트워크 식별
- 호스트 부분: 네트워크 내 특정 호스트 식별

### 패킷 스위칭의 지연 (Delay)

패킷이 네트워크를 통과하면서 발생하는 지연:

1. **처리 지연 (Processing Delay)**
    - 패킷 헤더 검사, 라우팅 테이블 조회 시간
2. **큐잉 지연 (Queueing Delay)**
    - 출력 링크가 사용 중일 때 대기하는 시간
3. **전송 지연 (Transmission Delay)**
    - 패킷을 링크에 올리는 시간 = L(패킷 크기) / R(링크 대역폭)
4. **전파 지연 (Propagation Delay)**
    - 신호가 물리적 매체를 통과하는 시간

### Delay 원인과 대응 방법

#### 1. 처리 지연 (Processing Delay) 대응 전략

패킷 헤더 검사, 포워딩 결정, ACL/NAT/QoS 처리 시간

**핵심 원인**
- CPU 성능 부족
    
- 소프트웨어 기반 패킷 처리
    
- 과도한 기능 활성화 (ACL, DPI, IPS 등)

**대응 방법**

1. **고성능 라우터/스위치 사용**
    
    - ASIC / NPU 기반 장비 선택
        
    - 소프트웨어 라우터(PF, iptables)는 한계 명확
        
2. **Fast Path / Hardware Offloading 활성화**
    
    - Linux: XDP, DPDK
        
    - 네트워크 장비: CEF, Cut-through switching
        
3. **불필요한 패킷 검사 제거**
    
    - L7 검사, 과도한 로깅 비활성화
        
    - ACL 최소화, 룰 정렬 최적화
        
4. **패킷 크기 일관성 유지**
    
    - MTU 미스매치 → fragment 처리 → 처리 지연 증가


---

#### 2. 큐잉 지연 (Queueing Delay) 대응 전략

출력 링크가 바쁠 때 패킷이 줄 서서 기다리는 시간

**핵심 원인**
- 링크 대역폭 부족
    
- 트래픽 버스트
    
- QoS 미설정

**대응 방법**
1. **링크 대역폭 증설**
    
    - 가장 단순하고 가장 확실
        
    - 1G → 10G 업그레이드가 모든 QoS보다 효과적일 때도 많음
        
2. **QoS (Quality of Service) 적용**
    
    - 우선순위 큐 (Voice, Control Plane)
        
    - WFQ, CBWFQ
        
3. **트래픽 셰이핑 / 폴리싱**
    
    - 송신 측에서 미리 속도 제한
        
    - 버스트를 평탄화하여 큐 폭주 방지
        
4. **ECMP / 로드 밸런싱**
    
    - 한 링크로 몰리는 트래픽 분산

---

#### 3. 전송 지연 (Transmission Delay) 대응 전략

L / R — 패킷을 링크에 “밀어 넣는” 시간

**핵심 원인**
- 링크 속도(R) 낮음
    
- 패킷 크기(L) 큼

**대응 방법**
1. **링크 대역폭 증가**
    
    - R 증가 → 전송 지연 직접 감소
        
    - 유일하게 공식에 정면으로 작용
        
2. **패킷 크기 최적화**
    
    - 불필요하게 큰 패킷 제거
        
    - 애플리케이션 레벨에서 chunk size 조절
        
3. **압축 사용**
    
    - WAN 구간에서 효과적
        
    - CPU 자원과 트레이드오프
        
4. **병렬 전송**
    
    - HTTP/2, HTTP/3, Multiplexing
        
    - 단일 큰 패킷보다 여러 스트림

#### 4. 전파 지연 (Propagation Delay) 대응 전략

거리 / 신호 전파 속도  
물리 법칙 영역

**핵심 원인**
- 물리적 거리
    
- 매체 종류 (광섬유, 구리, 위성)

**대응 방법**
1. **거리 단축**
    
    - 서버를 사용자 가까이 배치
        
    - Region, AZ 분산
        
2. **CDN 사용**
    
    - 콘텐츠를 엣지에 미리 배치
        
    - 전파 지연을 구조적으로 제거
        
3. **위성 → 지상망 전환**
    
    - GEO 위성은 지연 자체가 수백 ms
        
    - LEO(Starlink)는 개선되었지만 한계 존재
        
4. **프로토콜 최적화**
    
    - TCP handshake 최소화
        
    - QUIC, 0-RTT

#### 한 장 요약 (현업식 정리)

|지연 종류|본질|실전 대응|
|---|---|---|
|처리 지연|CPU 문제|ASIC, 오프로딩|
|큐잉 지연|혼잡 문제|대역폭, QoS|
|전송 지연|속도 문제|링크 업그레이드|
|전파 지연|거리 문제|서버 위치|

#### 참고 근거 (고전이지만 여전히 기준)

- Kurose & Ross, _Computer Networking: A Top-Down Approach_  
    [https://gaia.cs.umass.edu/kurose_ross/](https://gaia.cs.umass.edu/kurose_ross/)
    
- Cisco QoS Design Guide  
    [https://www.cisco.com/c/en/us/solutions/enterprise/design-zone-quality-of-service/index.html](https://www.cisco.com/c/en/us/solutions/enterprise/design-zone-quality-of-service/index.html)
    
- Cloudflare – Network Latency Explained  
    [https://www.cloudflare.com/learning/performance/glossary/latency/](https://www.cloudflare.com/learning/performance/glossary/latency/)

### 큐잉과 패킷 손실

**큐잉 발생 조건**:

- 입력 속도 > 출력 속도
- 패킷이 버퍼에 쌓임

**패킷 손실 (Packet Loss)**:

- 버퍼가 가득 차면 새로 도착한 패킷 드롭
- 손실된 패킷은 상위 계층에서 재전송 처리

### 장점

- 유연한 자원 활용
- 많은 사용자 동시 수용 가능
- 간헐적 데이터 전송에 효율적

### 단점

- 큐잉 지연 발생 가능
- 패킷 손실 가능성
- 서비스 품질(QoS) 보장 어려움

---

## 서킷 스위칭 (Circuit Switching)

### 기본 원리

서킷 스위칭은 통신 시작 전에 **전용 회선**을 설정하고, 통신이 끝날 때까지 독점적으로 사용하는 방식.

전통적인 전화망에서 사용되는 방식으로, 연결이 설정되면 고정된 대역폭을 보장받음.

### 자원 할당 방식

#### 1. FDM (Frequency Division Multiplexing)

- **주파수 분할 다중화**
- 전체 주파수 대역을 여러 개의 작은 대역으로 분할
- 각 연결에 특정 주파수 대역 할당
- 동시에 여러 통신 가능
- 예: FM 라디오 방송국들이 서로 다른 주파수 사용

#### 2. TDM (Time Division Multiplexing)

- **시간 분할 다중화**
- 전체 대역폭을 모두 사용하되, 시간을 분할
- 각 연결에 특정 시간 슬롯 할당
- 실제 전화망에서 주로 사용
- 시간 슬롯이 매우 짧아 사용자는 독점 사용하는 것처럼 느낌

### 동작 과정

1. **연결 설정 (Setup)**: 송신자와 수신자 간 경로 설정
2. **데이터 전송 (Data Transfer)**: 설정된 경로로 데이터 전송
3. **연결 해제 (Teardown)**: 통신 종료 후 자원 반환

### 장점

- 일정한 대역폭 보장
- 지연 시간 예측 가능
- 연속적인 데이터 전송에 효율적
- 품질 보장 (QoS)

### 단점

- 자원 낭비 가능 (사용하지 않아도 할당됨)
- 연결 설정에 시간 소요
- 제한된 동시 연결 수
- 비용이 높음

---

## 두 방식의 비교

### 자원 활용도 비교 예제

**가정 조건**:

- 링크 대역폭: 1 Gbps
- 각 사용자 최대 전송률: 100 Mbps
- 사용자가 활성 상태일 확률: 10%

#### 서킷 스위칭의 경우

- 각 사용자에게 100 Mbps 전용 할당 필요
- 최대 수용 인원: **10명** (1,000 Mbps ÷ 100 Mbps)
- 실제 평균 사용률: 10% (자원 낭비 90%)

#### 패킷 스위칭의 경우

- 통계적 다중화 활용
- 동시에 11명 이상 활성화될 확률을 계산

**확률 계산**:

P(k명 활성) = C(n, k) × (0.1)^k × (0.9)^(n-k)

P(과부하) = Σ P(k) for k = 11 to n

**목표 설정**:

- 허용 가능한 과부하 확률: 0.0004 (0.04%)
- 이 조건에서 최대 수용 인원: **약 35명**

#### 결론

패킷 스위칭이 **3.5배 더 많은 사용자** 수용 가능

### 종합 비교표

|특성|패킷 스위칭|서킷 스위칭|
|---|---|---|
|자원 할당|동적 (필요시)|정적 (미리 예약)|
|대역폭|공유|독점|
|연결 설정|불필요|필요|
|지연|가변적|고정적|
|자원 효율|높음|낮음|
|QoS 보장|어려움|쉬움|
|적합한 트래픽|간헐적, 버스티|연속적, 실시간|
|대표 사례|인터넷|전화망|

---

## 실제 적용 사례

### 패킷 스위칭을 개선하는 기술

패킷 스위칭은 기본적으로 확률적 방식이지만, 다양한 기술로 서킷 수준의 서비스 품질을 제공하려 노력합니다.

#### 1. 큐 스케줄링 (Queue Scheduling)

**FIFO (First In First Out)**:

- 가장 기본적인 방식
- 들어온 순서대로 처리

**우선순위 큐 (Priority Queue)**:

- 중요한 패킷(음성, 영상)을 먼저 처리
- 작은 패킷을 우선 전송하여 평균 대기 시간 감소

**공정 큐잉 (Fair Queueing)**:

- 각 흐름(flow)에 공평하게 대역폭 할당
- 특정 사용자가 독점하는 것 방지

#### 2. 능동적 큐 관리 (Active Queue Management)

**RED (Random Early Detection)**:

- 버퍼가 가득 차기 전에 일부 패킷을 선제적으로 드롭
- TCP가 전송 속도를 줄이도록 유도
- 전체 네트워크 혼잡 완화

**최적의 드롭 전략**:

- 이론: 큐의 맨 앞 패킷 드롭이 가장 효율적
- 이유: 대기 중인 모든 패킷의 대기 시간 감소
- 현실: 공평성 문제로 실제 구현은 다양

#### 3. QoS 메커니즘

**트래픽 쉐이핑 (Traffic Shaping)**:

- 전송 속도를 제어하여 버스트 완화

**리소스 예약**:

- RSVP (Resource Reservation Protocol)
- 중요한 트래픽을 위해 일정 자원 예약

**DiffServ (Differentiated Services)**:

- 패킷에 우선순위 표시 (DSCP 필드)
- 라우터가 우선순위에 따라 차별적 처리

### 언제 어떤 방식을 선택할까?

#### 패킷 스위칭이 적합한 경우

- 웹 브라우징, 이메일, 파일 전송
- 간헐적으로 데이터를 주고받는 응용
- 비용 효율이 중요한 경우
- 많은 사용자를 수용해야 하는 경우

#### 서킷 스위칭이 적합한 경우

- 음성 통화 (전통적인 전화)
- 연속적인 데이터 스트림
- 지연과 지터가 치명적인 응용
- 대역폭 보장이 필수적인 경우

#### 하이브리드 접근

현대 네트워크는 두 방식의 장점을 결합:

- **MPLS (Multiprotocol Label Switching)**: 패킷 스위칭 기반에 서킷처럼 경로 설정
- **SD-WAN**: 소프트웨어로 네트워크 경로와 QoS 동적 제어
- **5G 네트워크 슬라이싱**: 하나의 물리 네트워크를 여러 가상 네트워크로 분할

---
## 참고 자료

**패킷 스위칭 및 서킷 스위칭 기본**

- [GeeksforGeeks - Circuit Switching vs Packet Switching](https://www.geeksforgeeks.org/computer-networks/difference-between-circuit-switching-and-packet-switching/)
- [Wikipedia - Packet Switching](https://en.wikipedia.org/wiki/Packet_switching)
- [Wikipedia - Circuit Switching](https://en.wikipedia.org/wiki/Circuit_switching)
- [Britannica - Packet-Switched Network](https://www.britannica.com/technology/packet-switched-network)

**네트워크 코어 및 라우팅**

- [Network Core Overview](http://www2.ic.uff.br/~michael/kr1999/1-introduction/1_040-network_core.htm)
- [NinjaOne - Circuit vs Packet Switching Overview](https://www.ninjaone.com/blog/circuit-switching-vs-packet-switching/)

**FDM 및 TDM 멀티플렉싱**

- [GeeksforGeeks - FDM vs TDM](https://www.geeksforgeeks.org/computer-networks/difference-between-tdm-and-fdm/)
- [Wikipedia - Frequency Division Multiplexing](https://en.wikipedia.org/wiki/Frequency-division_multiplexing)
- [Wikipedia - Time Division Multiplexing](https://en.wikipedia.org/wiki/Time-division_multiplexing)
- [Testbook - TDM and FDM Explained](https://testbook.com/electrical-engineering/time-division-multiplexing-and-frequency-division-multiplexing)

**현대 네트워크 기술**

- [Cisco - SD-WAN vs MPLS](https://www.cisco.com/site/us/en/learn/topics/networking/what-is-the-difference-between-sd-wan-and-mpls.html)
- [Fortinet - SD-WAN vs MPLS Comparison](https://www.fortinet.com/resources/cyberglossary/sd-wan-vs-mpls)
- [Cloudflare - SD-WAN vs MPLS Overview](https://www.cloudflare.com/learning/network-layer/sd-wan-vs-mpls/)
- [Palo Alto Networks - MPLS vs SD-WAN](https://www.paloaltonetworks.com/cyberpedia/sd-wan-vs-mpls)
