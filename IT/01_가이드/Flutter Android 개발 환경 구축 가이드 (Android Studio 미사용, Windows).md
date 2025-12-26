---
title: 플러터 + 안드로이드 sdk
date: 2025-12-26
tags:
  - it
  - 가이드
---
## 사전 준비

- OS: Windows 11
    
- Flutter: stable 채널
    
- Android SDK: Command-line tools 사용

---

## 2. Flutter SDK 설치

### 2.1 Flutter SDK 다운로드

공식 사이트에서 Windows용 Flutter SDK ZIP 다운로드  
[https://docs.flutter.dev/get-started/install/windows](https://docs.flutter.dev/get-started/install/windows)

### 2.2 Flutter SDK 압축 해제

예시 경로:

```
C:\flutter
```

### 2.3 Flutter 실행 경로 추가 (환경변수)

시스템 환경변수 `PATH`에 추가:

```
C:\flutter\bin
```

확인:

```
flutter --version
```

---

## 3. Android SDK (Command-line Tools) 설치

### 3.1 Android SDK Command-line Tools 다운로드

공식 다운로드:  
[https://developer.android.com/studio#command-tools](https://developer.android.com/studio#command-tools)

파일 예:

```
commandlinetools-win-xxxx_latest.zip
```

---

## 4. Android SDK 디렉터리 구성

### 4.1 SDK 루트 디렉터리 생성

권장 경로:

```
C:\Android\Sdk
```

---

### 4.2 Command-line Tools 압축 해제 및 이동

압축 해제 후 폴더 구조를 아래와 같이 맞춘다.

```
C:\Android\Sdk\
 └─ cmdline-tools\
     └─ latest\
         ├─ bin
         ├─ lib
```

※ `cmdline-tools/latest` 구조는 필수 조건 (latest없으면 폴더 생성 후 집어넣기)

---

## 5. Android SDK 구성요소 설치

### 5.1 sdkmanager 실행 위치

```
C:\Android\Sdk\cmdline-tools\latest\bin
```

PowerShell 기준 실행 시 `.\` 사용

---

### 5.2 라이선스 동의

```
.\sdkmanager --licenses
```

모두 `y`로 동의

---

### 5.3 필수 SDK 패키지 설치

```
.\sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"
```

Flutter 요구사항에 따라 추가 설치:

```
.\sdkmanager "platforms;android-36" "build-tools;28.0.3"
```

---

### 5.4 최종 Android SDK 디렉터리 구조

```
C:\Android\Sdk\
 ├─ platform-tools
 ├─ platforms
 │   ├─ android-34
 │   └─ android-36
 ├─ build-tools
 │   ├─ 28.0.3
 │   └─ 34.0.0
 └─ cmdline-tools
     └─ latest
```

---

## 6. Flutter에 Android SDK 경로 등록

### 6.1 Flutter 설정 명령

```
flutter config --android-sdk C:\Android\Sdk
```

확인:

```
flutter config
```

---

## 7. 환경변수 설정

### 7.1 시스템 환경변수

```
ANDROID_SDK_ROOT = C:\Android\Sdk
ANDROID_HOME     = C:\Android\Sdk
```

### 7.2 PATH 추가

```
C:\Android\Sdk\platform-tools
C:\Android\Sdk\cmdline-tools\latest\bin
```

설정 후 PowerShell 재시작

---

## 8. 설치 검증

```
flutter doctor
```

정상 결과:

```
[✓] Android toolchain - develop for Android devices
• No issues found!
```

