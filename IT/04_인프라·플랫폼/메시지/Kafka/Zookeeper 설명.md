---
title: Zookeeper 설명
date: 2025-12-07
tags:
  - it
  - 인프라
  - 메시지
  - kafka
---
## Zookeeper란?

**Apache Zookeeper**는 분산 시스템을 위한 코디네이션 서비스. 분산 환경에서 설정 관리, 네이밍, 동기화, 그룹 서비스 등을 제공.

### Kafka에서 Zookeeper의 역할

Kafka는 전통적으로 Zookeeper에 의존해 옴. 주요 역할은 다음과 같다:

**1. 클러스터 메타데이터 관리**

- 브로커 목록 및 상태 관리
- 토픽 설정 정보 저장
- 파티션 리더 정보 관리

**2. 브로커 상태 모니터링**

- 각 브로커의 생존 여부 체크
- 브로커 장애 감지 및 알림

**3. 리더 선출**

- 파티션의 리더 브로커 선출
- 컨트롤러 브로커(클러스터 관리자) 선출

**4. 설정 변경 알림**

- 토픽 생성/삭제 시 브로커들에게 알림
- 설정 변경 사항 전파

### 간단한 비유

Zookeeper는 오케스트라의 **지휘자** 같은 역할:

- 각 연주자(브로커)가 누군지 알고 있음
- 누가 주선율(리더)을 맡을지 결정
- 악보(설정)를 관리하고 변경사항을 전달
- 연주자가 빠지면 다른 연주자에게 역할 재배치

## 중요: Kafka의 변화 (KRaft)

**Kafka 3.3.1 버전부터** Zookeeper 없이도 Kafka를 운영할 수 있는 **KRaft(Kafka Raft)** 모드가 도입.

### KRaft의 장점

- **단순화된 아키텍처**: Zookeeper 없이 Kafka만으로 운영
- **빠른 시작**: 관리할 컴포넌트가 줄어듦
- **향상된 확장성**: 수만 개의 파티션도 효율적으로 관리
- **빠른 복구**: 메타데이터 복구 시간 단축

### 현재 상황

- **Kafka 3.5.0+**: KRaft가 production-ready로 선언됨
- **Kafka 4.0 (예정)**: Zookeeper 지원 완전 제거 예정
- **권장사항**: 새로운 프로젝트는 KRaft 모드 사용 추천

## Zookeeper 없이 Kafka 실행하기 (KRaft)**실행 방법**:

```bash
# KRaft 모드로 Kafka 실행
docker-compose -f docker-compose-kraft.yml up -d

# 확인 - Zookeeper 없이 Kafka만 실행됨
docker ps
```

## Zookeeper vs KRaft 비교

|항목|Zookeeper 모드|KRaft 모드|
|---|---|---|
|컴포넌트 수|Kafka + Zookeeper|Kafka만|
|복잡도|높음|낮음|
|메타데이터 저장|Zookeeper|Kafka 내부|
|파티션 제한|~200,000개|수백만 개|
|복구 시간|느림|빠름|
|운영 난이도|어려움|쉬움|
|상태|레거시|최신 (권장)|

## 어떤 것을 사용해야 할까?

**새 프로젝트**: KRaft 모드 사용 (Zookeeper 불필요)

- 더 간단하고 현대적인 아키텍처
- Kafka 4.0부터는 Zookeeper 지원 중단 예정

**기존 프로젝트**:

- Kafka 3.5 이상이면 KRaft로 마이그레이션 고려
- 그 이하 버전이면 Zookeeper 유지

**학습 목적**: KRaft 모드부터 시작하는 것을 추천

- 미래 지향적
- 설정과 관리가 더 단순

## Zookeeper vs Kubernetes 비교

### 핵심 차이점: 완전히 다른 목적

둘 다 분산 시스템 관련 기술이지만 역할이 완전히 다름.

### 간단 비교표

| 구분        | Zookeeper          | Kubernetes        |
| --------- | ------------------ | ----------------- |
| **주요 목적** | 분산 코디네이션 서비스       | 컨테이너 오케스트레이션      |
| **관리 대상** | 설정 정보, 메타데이터       | 컨테이너화된 애플리케이션     |
| **규모**    | 소규모, 저지연 최적화       | 대규모 클러스터 관리       |
| **사용 사례** | 리더 선출, 분산 락, 설정 관리 | 앱 배포, 스케일링, 로드밸런싱 |

#### 비유로 이해하기

**Zookeeper**: 회사의 **총무팀**

- 누가 팀장인지 결정 (리더 선출)
- 회의실 예약 관리 (분산 락)
- 회사 규정 관리 (설정 정보)
- 직원 출근부 관리 (상태 모니터링)

**Kubernetes**: 회사의 **IT 운영팀**

- 서버 배포 및 관리
- 애플리케이션 확장/축소
- 트래픽 라우팅
- 장애 복구 및 자동화

### 실제 사용 예시

#### Zookeeper가 필요한 경우

```
분산 시스템에서:
- Kafka: 브로커 간 메타데이터 공유
- HBase: 마스터 서버 선출
- Hadoop: 네임노드 HA 구성
- 분산 락이 필요한 마이크로서비스
```

#### Kubernetes가 필요한 경우

```
컨테이너 애플리케이션 관리:
- Docker 컨테이너 배포 및 스케일링
- 마이크로서비스 아키텍처 운영
- 로드밸런싱 및 서비스 디스커버리
- CI/CD 파이프라인 자동화
```

### 함께 사용하는 경우

Kubernetes는 컨테이너 오케스트레이션에 중점을 두고, Zookeeper는 분산 시스템을 위한 코디네이션 서비스를 제공. 실제로 많은 조직에서 둘을 함께 사용:

**예시**: Kafka를 Kubernetes에서 운영

- Kubernetes가 Kafka 컨테이너를 배포/관리
- Zookeeper가 Kafka 브로커 간 코디네이션 담당

## 참고 자료

### 공식 문서

- **Zookeeper 공식**: https://zookeeper.apache.org/doc/current/
- **Kubernetes 공식**: https://kubernetes.io/docs/home/
- **Kubernetes에서 Zookeeper 실행하기**: https://kubernetes.io/docs/tutorials/stateful-application/zookeeper/

### 비교 및 아키텍처 자료

- **StackShare 비교**: https://stackshare.io/stackups/kubernetes-vs-zookeeper
- **Kafka Architecture 가이드**: https://kafka.apache.org/documentation/#design
- **Confluent Kafka 문서**: https://docs.confluent.io/

### 한글 학습 자료

- **카카오 기술 블로그 - Zookeeper**: https://tech.kakao.com/
- **우아한형제들 - Kafka & Kubernetes**: https://techblog.woowahan.com/
- **네이버 D2 - 분산 시스템**: https://d2.naver.com/

### 실습 가이드

- **Kubernetes + Zookeeper 실습**: https://kubernetes.io/docs/tutorials/stateful-application/zookeeper/
- **Confluent Platform Quickstart**: https://docs.confluent.io/platform/current/quickstart/index.html

### YouTube 강의

- "Apache Zookeeper Explained" - ByteByteGo
- "Kubernetes Tutorial for Beginners" - TechWorld with Nana
- "Kafka + Kubernetes" - Confluent

### 책 추천

- **Zookeeper**: "ZooKeeper: Distributed Process Coordination" (O'Reilly)
- **Kubernetes**: "쿠버네티스 인 액션" (에이콘)
- **통합**: "클라우드 네이티브 데브옵스" (한빛미디어)
