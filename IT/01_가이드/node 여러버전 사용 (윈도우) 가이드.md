---
title: node 여러버전 사용 (윈도우) 가이드
date: 2025-12-24
tags:
  - it
  - 가이드
---
## 1. 기존 Node 완전 삭제 (충돌 방지)

확인 
```
C:\Users\user>npm -v
11.6.2

C:\Users\user>node -v
v24.11.1
```

1. 프로그램 제거

제어판 → 프로그램 → Node.js 제거

2. 잔존 파일 삭제
아래 경로가 남아 있으면 수동 삭제
```
C:\Program Files\nodejs
C:\Users\%USERNAME%\AppData\Roaming\npm
C:\Users\%USERNAME%\AppData\Roaming\npm-cache
```

3. 환경 변수 정리

PATH에 남아 있는 항목 제거

C:\Program Files\nodejs

npm 관련 경로

4. 터미널 재시작 후 확인

```
node -v
npm -v
```


→ 인식되면 아직 덜 지운 상태.

## 2. nvm 설치

1. 프로그램 설치
https://github.com/coreybutler/nvm-windows/releases
여기에 가서 nvm-setup.exe 다운로드 받고 설치

2. 프로그램 설치 확인
```
PS C:\Users\user> nvm -v
1.2.2
```

## 3. Node 여러 버전 설치 & 전환

1. 설치  
nvm install 18.20.4  
nvm install 20.15.1

2. 전환  
nvm use 18.20.4  
node -v

nvm use 20.15.1  
node -v


3. 현재 설치된 버전 목록 확인  
nvm list

## 4. 프로젝트별 Node.js 버전 관리

1. `.nvmrc` 사용

   * 파일에 **버전만 작성**

     ```
     16.16.0
     ```

3. 버전 전환

   * **자동 전환 X**
   * 프로젝트 디렉토리에서 **반드시 수동 실행**

     ```bash
     nvm use
     ```