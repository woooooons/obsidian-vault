---
title: Windows 환경에서 Node.js 다중 버전 관리 (nvm-windows) 가이드
date: 2025-12-24
tags:
  - it
  - 가이드
---
## 1. 기존 Node.js 완전 삭제 (충돌 방지)

### 현재 설치 여부 확인

```text
node v24.11.1
npm 11.6.2
```

전역 Node.js가 설치된 상태이므로 nvm 사용 전 제거가 필요.

### 삭제 절차

1. **제어판 → 프로그램 → Node.js 제거**
    
2. **잔존 파일 수동 삭제**
    
    ```
    C:\Program Files\nodejs
    C:\Users\%USERNAME%\AppData\Roaming\npm
    C:\Users\%USERNAME%\AppData\Roaming\npm-cache
    ```
    
3. **환경 변수(PATH) 정리**
    
    - `C:\Program Files\nodejs`
        
    - npm 관련 경로
        
4. **터미널 재시작 후 확인**
    
    ```bash
    node -v
    npm -v
    ```
    
    인식되면 삭제가 완료되지 않은 상태.
    

---

## 2. nvm-windows 설치

### 설치

- [https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases)
    
- `nvm-setup.exe` 다운로드 후 설치
    

### 설치 확인

```powershell
nvm -v
```

출력 예:

```text
1.2.2
```

---

## 3. Node.js 여러 버전 설치 및 전환

### 버전 설치

```bash
nvm install 18.20.4
nvm install 20.15.1
```

### 버전 전환

```bash
nvm use 18.20.4
node -v

nvm use 20.15.1
node -v
```

### 설치된 버전 목록 확인

```bash
nvm list
```

---

## 4. 프로젝트별 Node.js 버전 관리 (Windows 기준)

### `.nvmrc` 파일

프로젝트 루트에 `.nvmrc` 파일을 생성하고 **버전만 작성**.

```
16.16.0
```

### 중요한 제한 사항

- nvm-windows는 `.nvmrc`를 **자동으로 읽지 않음**
    
- `nvm use` 단독 실행은 동작하지 않는다
    
- 디렉토리 이동 시 자동 전환 기능은 지원되지 않음
    

### 올바른 사용 방법

#### 방법 1. 명시적 버전 지정

```bash
nvm use 16.16.0
```

#### 방법 2. PowerShell에서 `.nvmrc`를 직접 읽기

```powershell
nvm use (Get-Content .nvmrc)
```

---

## 5. Windows nvm-windows에서 지원되지 않는 기능

- `nvm use` (버전 미지정)
    
- `.nvmrc` 자동 로드
    
- 디렉토리 이동 시 자동 전환
    
- `nvm alias default`
    

위 기능들은 Linux/macOS용 nvm(nvm-sh)에만 존재한다.
