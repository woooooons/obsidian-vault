---
title: ERPnext
date: 2025-12-03
tags:
  - it
  - 개발
  - 프로젝트
  - ERP
---
ERPNext는 **오픈소스 ERP 시스템**으로, 회계·인사·재고·CRM·프로젝트·제조 등 기업 전반의 워크플로우를 통합 관리하는 풀스택 ERP 플랫폼이다.  
프레임워크는 **Frappe Framework(파이썬 + MariaDB + Redis + Node.js)** 기반이며, 소스 전체가 열려 있어 **커스텀 코드, 커스텀 앱, 커스텀 도큐타입**을 만들 수 있다.  
또한 서버를 직접 운영하면 **데이터 100% 로컬 보관**, **네트워크 내부망 운영**, **코드 커스텀**, **API 통합**이 모두 가능하다.

참고: 공식 GitHub  
[https://github.com/frappe/frappe](https://github.com/frappe/frappe)  
[https://github.com/frappe/erpnext](https://github.com/frappe/erpnext)

---

# 1. ERPNext 셀프호스팅 방식 전체 구조

### 셀프호스팅 방법은 크게 두 가지다:

## **A. Docker 기반 설치 (가장 쉬움, Dev·Prod 둘 다 가능)**

- 도커로 구성된 공식 스택 존재
    
- 구성 요소는 다음과 같이 자동 띄워짐
    
    - Frappe/ERPNext Backend (Python + Gunicorn)
        
    - Frontend (Node.js)
        
    - MariaDB
        
    - Redis
        
    - Workers (long, short, scheduler)
        
    - NGINX Reverse Proxy
        
- 커스텀 앱 설치도 간단함 (`bench get-app`, `bench install-app`)
    

## **B. Bench 기반 수동 설치 (고급 사용자용)**

- 시스템 직접 설치
    
- 높은 커스텀 자유도
    
- 개발 환경에 적합
    

대부분은 **Docker 기반**을 선택한다.

---

# 2. Docker 기반 셀프호스팅 절차 (가장 간단한 실전 가이드)

아래 절차는 **git clone 후 docker-compose.yml 직접 실행**하는 방식이다.

---

## ■ 1단계: docker + docker compose 설치

Ubuntu 기준

```bash
apt update
apt install docker.io docker-compose -y
```

---

## ■ 2단계: ERPNext 도커 스택 다운로드

공식 Docker repo:  
[https://github.com/frappe/frappe_docker](https://github.com/frappe/frappe_docker)

실제 클론:

```bash
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```

---

## ■ 3단계: 환경 파일 생성

기본 템플릿 사용:

```bash
cp env-example .env
```

`.env` 항목 예시:

```
ERPNEXT_VERSION=v15
MARIADB_HOST=mariadb
SITE_NAME=erp.mydomain.local
```

---

## ■ 4단계: docker-compose.yml 선택

`compose.yaml` 혹은 `compose.prod.yaml` 둘 중 선택한다.

개발용(로컬):

```
compose.yaml
```

프로덕션:

```
compose.prod.yaml
```

---

## ■ 5단계: 컨테이너 시작

```bash
docker compose up -d
```

---

## ■ 6단계: 새 사이트 생성

```bash
docker compose exec backend bench new-site erp.yourdomain.local
```

---

## ■ 7단계: ERPNext 앱 설치

```bash
docker compose exec backend bench get-app erpnext --branch version-15
docker compose exec backend bench install-app erpnext
```

---

## ■ 8단계: 브라우저 접속

```
http://localhost:8080
```

기본 포트는 8080이지만 compose에 따라 다를 수 있다.

---

# 3. 커스텀 코드/커스텀 앱 사용 방법

ERPNext는 **"App" 단위**로 커스텀 기능을 추가한다.  
프로젝트를 직접 수정하지 않고, 별도의 앱을 만들어 확장하는 방식이다.

---

## ■ 커스텀 앱 만들기

```bash
docker compose exec backend bench new-app my_custom_app
docker compose exec backend bench install-app my_custom_app
```

이제 `/apps/my_custom_app` 안에서

- 새로운 DocType
    
- API 엔드포인트
    
- REST API
    
- 배치 작업
    
- UI 확장
    

전부 가능하다.

---

# 4. 로컬 데이터 저장: 데이터 위치

Docker 기반일 때 기본 볼륨 경로:

- MariaDB 데이터:
    

```
./volumes/db
```

- ERPNext 사이트 파일:
    

```
./volumes/sites
```

**이 두 디렉토리만 백업하면 서버 복구 가능하다.**

---

# 5. 운영 시 고려사항

### 1) 백업 자동화

ERPNext는 자체 백업 명령 존재:

```bash
docker compose exec backend bench backup
```

### 2) SSL 적용

NGINX + Certbot 또는 Cloudflare로 쉽게 가능  
프로덕션 compose 파일에 예제가 포함돼 있음

### 3) 업데이트

```bash
docker compose pull
docker compose up -d
docker compose exec backend bench migrate
```

---

# 6. 참조 자료

- ERPNext 공식 문서  
    [https://docs.erpnext.com/](https://docs.erpnext.com/)
    
- Frappe Docker 공식  
    [https://github.com/frappe/frappe_docker](https://github.com/frappe/frappe_docker)
    
- Frappe Framework  
    [https://frappeframework.com/docs](https://frappeframework.com/docs)
    

---

