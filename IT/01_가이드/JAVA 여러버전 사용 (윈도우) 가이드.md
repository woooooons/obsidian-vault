---
title: JAVA 여러버전 사용 (윈도우) 가이드
date: 2025-12-07
tags:
  - it
  - 가이드
---
## 사전 준비
![[Pasted image 20251207153227.png]]  

- jdk 다운 (여러버전)

## STEP 1: 환경 변수 설정

![[Pasted image 20251207153338.png]]  
1. 시스템 환경 변수에 JAVA_HOME 변수 설정 값은 jdk 경로인데 bin 부모 폴더
![[Pasted image 20251207153412.png]]  
2. 시스템 환경 변수 path에 %JAVA_HOME%\bin 추가

## STEP 2: 스크립트 작성
![[Pasted image 20251207153450.png]]  

1. 위의 scripts 폴더에 bat 파일 버전 별로 생성

![[Pasted image 20251207153618.png]]  
```sh
@echo off
set JAVA_HOME=JDK경로
set Path=%JAVA_HOME%\bin;%Path%
echo Java 버전 activated.
```
2. 스크립트 작성
![[Pasted image 20251207153703.png]]
3. 터미널에서 실행

## 참고 링크
https://velog.io/@heyhighbyee/JDK-%EC%97%AC%EB%9F%AC-%EB%B2%84%EC%A0%84-%EC%84%A4%EC%B9%98%ED%95%98%EC%97%AC-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0