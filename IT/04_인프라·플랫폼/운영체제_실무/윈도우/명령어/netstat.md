---
title: netstat
date: 2026-01-08
tags:
  - it
  - 인프라
  - 운영체제_실무
  - 윈도우
  - 명령어
  - netstat
---

## 1. `netstat`란

`netstat`(Network Statistics)는 **운영체제 레벨에서 현재 네트워크 연결 상태, 포트 사용 현황, 라우팅 통계**를 확인하는 명령줄 도구.  
Windows, Linux, Unix 계열 모두에 존재하며, **트러블슈팅·보안 점검·서버 운영**에서 가장 기본이 되는 명령어.

Windows에서는 **TCP/IP 스택 내부 상태를 직접 출력**하기 때문에, 애플리케이션 로그보다 더 낮은 계층의 정보를 제공.

> 핵심 역할

- 어떤 포트가 열려 있는가
    
- 어떤 IP와 통신 중인가
    
- 어떤 프로세스가 해당 포트를 점유하고 있는가
    

---

## 2. 기본 사용법

```bat
netstat
```

기본 실행 시 다음 정보를 출력한다.

|항목|의미|
|---|---|
|Proto|프로토콜 (TCP / UDP)|
|Local Address|로컬 IP:포트|
|Foreign Address|원격 IP:포트|
|State|연결 상태 (TCP만)|

예시:

```
TCP    127.0.0.1:8080    127.0.0.1:51532    ESTABLISHED
```

---

## 3. 핵심 옵션 정리 (Windows 기준)

### 3.1 `-a` : 모든 연결 + LISTEN 포트 표시

```bat
netstat -a
```

- 현재 연결 중인 포트
    
- **리스닝 중인 서버 포트까지 전부 출력**
    

서버 점검 시 가장 많이 사용된다.

---

### 3.2 `-n` : 숫자 형태로 출력 (DNS 해석 제거)

```bat
netstat -n
```

- IP와 포트를 **숫자로 그대로 표시**
    
- DNS 역조회 제거 → **속도 향상**
    

실무에서는 거의 항상 `-n`을 붙인다.

---

### 3.3 `-o` : 프로세스 ID(PID) 표시

```bat
netstat -ano
```

- 포트를 점유한 **프로세스의 PID 출력**
    
- 보안·장애 분석의 핵심 옵션
    

출력 예:

```
TCP    0.0.0.0:3306    0.0.0.0:0    LISTENING    8424
```

---

### 3.4 PID → 프로세스 이름 확인

```bat
tasklist | findstr 8424
```

또는

```bat
tasklist /FI "PID eq 8424"
```

→ **어떤 프로그램이 해당 포트를 사용 중인지 식별 가능**

---

### 3.5 `-b` : 실행 파일 이름 표시 (관리자 권한 필요)

```bat
netstat -ab
```

- PID 대신 **실제 실행 파일(.exe) 경로 표시**
    
- 느리고 권한 요구 높음
    
- 보안 분석 시 유용
    

---

### 3.6 `-p` : 프로토콜 지정

```bat
netstat -an -p tcp
netstat -an -p udp
```

---

### 3.7 `-r` : 라우팅 테이블 확인

```bat
netstat -r
```

내부적으로 `route print`와 동일한 결과를 제공한다.

---

### 3.8 `-s` : 프로토콜 통계

```bat
netstat -s
```

- TCP 재전송
    
- 패킷 오류
    
- 세션 통계
    

→ **네트워크 품질 분석용**

---

## 4. TCP 상태(State) 해석이 핵심이다

|상태|의미|
|---|---|
|LISTENING|서버가 포트 열고 대기 중|
|ESTABLISHED|정상 연결|
|TIME_WAIT|종료 후 잔여 세션 정리 중|
|CLOSE_WAIT|상대는 종료, 로컬이 아직 종료 안 함|
|SYN_SENT|연결 시도 중|
|SYN_RECEIVED|연결 요청 수신|

### 실무 포인트

- `TIME_WAIT` 과다 → 포트 고갈 위험
    
- `CLOSE_WAIT` 다수 → **애플리케이션 버그 가능성**
    

---

## 5. 실전 활용 시나리오

### 5.1 특정 포트 점유 프로세스 찾기

```bat
netstat -ano | findstr :8080
```

→ PID 확인 → `tasklist`로 프로세스 확인

---
### 5.2 특정 ip 프로세스 찾기

```bat
- ```
    netstat -ano | findstr "192.168.20.100"
    ```
```

→ PID 확인 → `tasklist`로 프로세스 확인

---

### 5.3 외부와 통신 중인 의심 연결 탐지

```bat
netstat -ano | findstr ESTABLISHED
```

- 낯선 해외 IP
    
- 비표준 포트 사용  
    → 악성코드 의심 가능
    

---

### 5.4 서버에서 열려 있으면 안 되는 포트 점검

```bat
netstat -an | findstr LISTENING
```

- 21, 23, 3389, 3306 등 점검 대상
    

---

## 6. 운영·보안 관점에서의 `netstat`

### 장점

- OS 기본 내장
    
- 설치 불필요
    
- 즉각적인 상태 확인
    

### 한계

- 과거 연결 이력 미보존
    
- 패킷 내용 확인 불가
    
- 대규모 트래픽 분석에는 부적합
    

> 보완 도구

- Wireshark (패킷 레벨)
    
- TCPView (GUI)
    
- Resource Monitor
    

---

## 7. `netstat`를 쓰는 이유

Windows PowerShell에는 `Get-NetTCPConnection` 같은 고급 명령이 존재하지만,

- SSH 원격
    
- 장애 상황
    
- 최소 환경(Server Core)
    

에서는 **`netstat -ano`가 가장 빠르고 확실한 선택**이다.

---

## 8. 요약

- `netstat`는 **네트워크 상태의 진실**을 보여준다
    
- Windows 서버·개발·보안 모두 필수
    
- 실무 기본 조합은 다음 한 줄이다
    

```bat
netstat -ano
```

---

## 참고 자료 (공식 문서)

- Microsoft Docs – netstat  
    [https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netstat](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netstat)
    
- TCP 상태 설명 (RFC 793)  
    [https://datatracker.ietf.org/doc/html/rfc793](https://datatracker.ietf.org/doc/html/rfc793)
    
