---
title: taskkill
date: 2026-01-08
tags:
  - it
  - 인프라
  - 운영체제_실무
  - 윈도우
  - 명령어
  - taskkill
---
## 1. `taskkill`이란

`taskkill`은 **Windows에서 실행 중인 프로세스를 명령줄로 종료**하는 도구.
GUI 작업 관리자(Task Manager)와 동일한 작업을 수행하지만, 다음 상황에서 훨씬 강력.

- 원격 서버 접속 (RDP, SSH)
    
- 서비스가 응답하지 않을 때
    
- 자동화 스크립트
    
- 포트 점유 문제 해결
    

> 핵심 포인트

- PID 또는 프로세스 이름 기준 종료
    
- 강제 종료 가능
    
- 조건 필터링 가능
    

---

## 2. 기본 사용법

### 2.1 PID로 종료 (가장 정확)

```bat
taskkill /PID 8424
```

- 단일 프로세스 종료
    
- 정상 종료 시도 (Graceful)
    

---

### 2.2 프로세스 이름으로 종료

```bat
taskkill /IM node.exe
```

- 같은 이름의 프로세스 **여러 개가 동시에 종료될 수 있음**
    
- 서버 환경에서는 주의 필요
    

---

## 3. 필수 옵션 정리

### 3.1 `/F` : 강제 종료

```bat
taskkill /PID 8424 /F
```

- 프로세스 응답 여부 무시
    
- `SIGKILL`에 가까운 동작
    
- **데이터 유실 가능성 존재**
    

실무에서 가장 많이 사용되는 옵션이다.

---

### 3.2 `/T` : 자식 프로세스까지 종료

```bat
taskkill /PID 8424 /T
```

- 해당 PID의 **하위 프로세스 트리 전체 종료**
    
- Node.js, Java, Python 서버에서 매우 중요
    

```bat
taskkill /PID 8424 /F /T
```

→ 서버 프로세스 완전 종료 표준 조합

---

### 3.3 `/IM` + `/F`

```bat
taskkill /IM java.exe /F
```

- 동일 이름의 모든 Java 프로세스 종료
    
- 운영 서버에서는 **위험도가 높음**
    

---

## 4. netstat과의 실전 연계

### 4.1 포트 점유 프로세스 강제 종료 흐름

```bat
netstat -ano | findstr :8080
```

출력:

```
TCP    0.0.0.0:8080    0.0.0.0:0    LISTENING    8424
```

```bat
taskkill /PID 8424 /F
```

→ **포트 즉시 해제**

---

### 4.2 좀비 서버(포트만 살아있는 경우) 처리

- 서버 재시작 실패
    
- 포트 바인딩 오류 발생
    
- 서비스는 죽었는데 프로세스는 살아있음
    

이 경우 `/F` 없이는 종료되지 않는 경우가 많다.

---

## 5. 조건 필터링 (`/FI`) 고급 사용

### 5.1 메모리 사용량 기준 종료

```bat
taskkill /FI "MEMUSAGE gt 500000" /F
```

- 메모리 500MB 초과 프로세스 종료
    
- 장애 대응 자동화에 사용
    

---

### 5.2 특정 사용자 프로세스만 종료

```bat
taskkill /FI "USERNAME eq testuser" /F
```

---

### 5.3 상태 기준 종료

```bat
taskkill /FI "STATUS eq NOT RESPONDING" /F
```

GUI 작업 관리자에서 “응답 없음”과 동일 기준

---

## 6. `taskkill` vs 작업 관리자(Task Manager)

|구분|taskkill|작업 관리자|
|---|---|---|
|원격 서버|가능|불편|
|자동화|가능|불가|
|정확성|PID 기준|실수 가능|
|대량 처리|가능|불가|

---

## 7. 운영·보안 관점 주의사항

### 7.1 무조건 `/F`는 위험하다

- 로그 미기록
    
- DB 트랜잭션 손상 가능
    
- 파일 락 잔존 가능성
    

**원칙**

1. `/F` 없이 시도
    
2. 실패 시 `/F`
    
3. 서버 프로세스는 `/T` 포함
    

---

### 7.2 시스템 프로세스 종료 금지

절대 종료하면 안 되는 예:

- `csrss.exe`
    
- `wininit.exe`
    
- `services.exe`
    
- `lsass.exe`
    

→ 시스템 즉시 종료 또는 블루스크린

---

## 8. PowerShell 환경에서의 위치

PowerShell에는 `Stop-Process`가 존재하지만,

```powershell
Stop-Process -Id 8424 -Force
```

- **cmd 기반 서버**
    
- **윈도우 서버 최소 설치(Server Core)**
    
- **운영 자동화 배치**
    

에서는 여전히 `taskkill`이 표준이다.

---

## 9. 요약 정리

- `taskkill`은 **문제 프로세스 정리의 최종 수단**
    
- `netstat -ano → taskkill /PID /F /T`는 실무 필수 콤보
    
- 강력한 만큼 신중하게 사용해야 한다
    

### 실무 표준 명령어

```bat
taskkill /PID <PID> /F /T
```

---

## 참고 자료

- Microsoft Docs – taskkill  
    [https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/taskkill](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/taskkill)
    
- Windows Process Management 개요  
    [https://learn.microsoft.com/en-us/windows/win32/procthread/processes-and-threads](https://learn.microsoft.com/en-us/windows/win32/procthread/processes-and-threads)
    



