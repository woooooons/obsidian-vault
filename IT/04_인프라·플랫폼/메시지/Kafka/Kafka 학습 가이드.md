---
title: Kafka 학습 가이드
date: 2025-12-07
tags:
  - it
  - 인프라
  - 메시지
  - kafka
  - 학습
---
## 1. Kafka란?

**Apache Kafka**는 LinkedIn에서 개발한 분산 스트리밍 플랫폼. 실시간 데이터 파이프라인과 스트리밍 애플리케이션을 구축하는 데 사용.

### 핵심 개념

**Producer (생산자)**: 메시지를 생성하여 Kafka에 전송하는 애플리케이션

**Consumer (소비자)**: Kafka로부터 메시지를 읽어가는 애플리케이션

**Topic (토픽)**: 메시지가 저장되는 카테고리/채널. 데이터베이스의 테이블과 유사

**Broker (브로커)**: Kafka 서버. 메시지를 저장하고 관리

**Partition (파티션)**: 토픽을 분할한 단위. 병렬 처리와 확장성을 제공

**Consumer Group**: 여러 Consumer가 협력하여 메시지를 분산 처리

**Offset**: 각 메시지의 고유한 위치 번호

### Kafka의 장점

- **높은 처리량**: 초당 수백만 건의 메시지 처리
- **확장성**: 클러스터에 브로커 추가로 쉽게 확장
- **내구성**: 디스크에 메시지를 저장하여 데이터 손실 방지
- **분산 처리**: 여러 서버에 데이터를 분산하여 고가용성 제공

### 주요 사용 사례

- 로그 수집 및 모니터링
- 실시간 데이터 처리 (주식 시세, IoT 센서 데이터)
- 마이크로서비스 간 이벤트 기반 통신
- 데이터 파이프라인 구축

## 2. 실습: Docker로 Kafka 환경 구축**실행 방법**:

```bash
# Docker Compose로 Kafka 실행
docker-compose up -d

# 실행 확인
docker ps
```

## 3. Python으로 Producer/Consumer 구현**실행 순서**:

```bash
# 1. kafka-python 설치
pip install kafka-python

# 2. Consumer 실행 (먼저 실행해서 대기)
python consumer.py

# 3. 다른 터미널에서 Producer 실행
python producer.py
```

## 4. Node.js 예제 (선택사항)

```js
// npm install kafkajs

const { Kafka } = require('kafkajs');

const kafka = new Kafka({
  clientId: 'my-app',
  brokers: ['localhost:9092']
});

// Producer 예제
async function runProducer() {
  const producer = kafka.producer();
  await producer.connect();
  
  for (let i = 0; i < 5; i++) {
    await producer.send({
      topic: 'test-topic',
      messages: [
        { 
          value: JSON.stringify({ 
            number: i, 
            message: `Hello from Node.js ${i}` 
          }) 
        }
      ]
    });
    console.log(`Sent message ${i}`);
  }
  
  await producer.disconnect();
}

// Consumer 예제
async function runConsumer() {
  const consumer = kafka.consumer({ groupId: 'node-group' });
  await consumer.connect();
  await consumer.subscribe({ topic: 'test-topic', fromBeginning: true });
  
  await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
      console.log({
        topic,
        partition,
        offset: message.offset,
        value: message.value.toString()
      });
    }
  });
}

// 실행
// runProducer();
// runConsumer();
```
## 5. 주요 CLI 명령어

```bash
# Kafka 컨테이너 접속
docker exec -it kafka bash

# 토픽 생성
kafka-topics --create --topic my-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1

# 토픽 목록 확인
kafka-topics --list --bootstrap-server localhost:9092

# 토픽 상세 정보
kafka-topics --describe --topic test-topic --bootstrap-server localhost:9092

# 콘솔에서 메시지 생성
kafka-console-producer --topic test-topic --bootstrap-server localhost:9092

# 콘솔에서 메시지 소비
kafka-console-consumer --topic test-topic --from-beginning --bootstrap-server localhost:9092
```

## 6. 참고 자료

**공식 문서**:

- Apache Kafka 공식 문서: https://kafka.apache.org/documentation/
- Kafka 빠른 시작 가이드: https://kafka.apache.org/quickstart

**한글 자료**:

- Kafka 한국 사용자 모임: https://kafka.apache.org/
- 우아한형제들 기술 블로그 Kafka 관련 글들

**영상 강의**:

- Confluent의 Kafka 101 시리즈 (YouTube)
- 인프런 - "Apache Kafka 애플리케이션 프로그래밍" 강의

**책**:

- "카프카, 데이터 플랫폼의 최강자" (고승범 저)
- "Learning Apache Kafka" (Nishant Garg)

**실습 환경**:

- Confluent Cloud (무료 체험): https://confluent.cloud/
- Docker Compose로 로컬 환경 구축 (위에서 제공)