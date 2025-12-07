---
title: git lab, n8n, teams 연동
date: 2025-12-05
tags:
  - it
  - 가이드
---
## 사전 준비

1. **n8n 설치**
    
    - 로컬: Node.js 또는 Docker
        
    - 서버: Docker 권장
        
2. **GitLab 프로젝트**
    
3. **Microsoft Teams 채널**
    
4. **Teams Incoming Webhook 권한**
    

---

## STEP 1: Microsoft Teams Webhook 생성

### 1-1. Teams 채널 이동

- Teams → 알림 받을 채널 선택
    
- 오른쪽 상단 `⋯` → **커넥터(Connectors)**
    

### 1-2. Incoming Webhook 추가

1. **Incoming Webhook** 검색 → 추가
    
2. 이름: `GitLab Bot`
    
3. 생성 후 **Webhook URL 복사**
    

예시:

```
https://outlook.office.com/webhook2/xxxxxxxx
```

이 URL은 **n8n에서만 사용.**

---

## STEP 2: n8n Webhook 노드 생성 (GitLab 수신)

### 2-1. 새 워크플로우 생성

- n8n 접속 → **New Workflow**
    

### 2-2. Webhook 노드 추가

| 항목          | 값           |
| ----------- | ----------- |
| HTTP Method | POST        |
| Path        | `gitlab`    |
| Respond     | Immediately |

생성되는 URL 예시:

```
http://localhost:5678/webhook-test/gitlab
```

또는 운영 서버:

```
https://n8n.example.com/webhook/gitlab
```

---

## STEP 3: GitLab Webhook 설정

GitLab 프로젝트 → **Settings → Webhooks**

| 항목                      | 값               |
| ----------------------- | --------------- |
| URL                     | n8n Webhook URL |
| Push events             | ✅               |
| Merge request events    | ✅               |


저장 후:

- **Test → Push events** 클릭
![[Pasted image 20251205104749.png]]
    
- n8n Webhook 노드에 JSON이 들어오면 성공
    

---

## STEP 4: Set 노드 추가 (Teams용 메시지 변환)

Webhook 노드 뒤에 **Set 노드** 추가
![[Pasted image 20251205104901.png]]

### 4-1. Set 노드 기본 설정

| 항목   | 값              |
| ---- | -------------- |
| Mode | Manual Mapping |
![[Pasted image 20251205104943.png]]
### 4-2. Field 추가

|Key|Type|Value|
|---|---|---|
|text|String|아래 내용 복붙|

```text
[GitLab 알림]  

이벤트: {{ $json.body.object_kind }} 
프로젝트: {{ $json.body.project.name }} 
사용자: {{ $json.body.user.username }}
브랜치: {{ $json.body.project.default_branch }}
URL: {{ $json.body.project.http_url }}
```


---

## STEP 5: HTTP Request 노드 추가 (Teams로 전송)

Set 노드 뒤에 **HTTP Request 노드** 추가

### 5-1. 기본 설정

| 항목                | 값                            |
| ----------------- | ---------------------------- |
| Method            | POST                         |
| URL               | ✅ Teams Incoming Webhook URL |
| Send Body         | true                         |
| Body Content Type | JSON                         |
| Specify Body      | Using Fields Below           |

---

### 5-2. Body Parameters 설정 (가장 중요)

|Name|Value|
|---|---|
|text|`{{$json.text}}`|

⚠️ 주의 사항:

- Name은 반드시 `text`
    
- Value는 반드시 `{{$json.text}}`
    
- 따옴표 `" "` 사용 금지
    

---

## STEP 6: 전체 워크플로우 구조

```
[Webhook (GitLab)]
        ↓
[Set (text 생성)]
        ↓
[HTTP Request (Teams 전송)]
```


---

## STEP 7: 동작 테스트

1. GitLab → Webhooks → **Test → Push events**
    
2. n8n:
    
    - Webhook ✅
        
    - Set ✅
        
    - HTTP Request ✅
        
3. Teams 채널에 메시지 도착 확인 ✅
    

---

## 정상 알림 예시 (Teams)

```
[GitLab 알림]

이벤트: push
프로젝트: api-server
사용자: jiwoon
브랜치: main
URL: https://gitlab.com/team/api-server
```

```
[GitLab 알림]

이벤트: merge_request
프로젝트: frontend
사용자: minsoo
브랜치: feature/login
```

---

## 문제 해결

### Webhook은 뜨는데 Teams가 안 올 때

- HTTP Request Body에 `text`가 없는 경우
    
- `{{$json.text}}` 대신 문자열로 입력한 경우
    

---

### HTTP 400 발생 시

원인 100%:

- `text` 필드 없음
    
- JSON 구조 잘못됨
    
- Teams Webhook URL 오타
    

---

### 메시지가 비어 있을 때

- Set 노드에서 `text`를 만들지 않았거나
    
- Set 노드가 HTTP Request와 연결되지 않음
    

---

## 확장 구성 (선택)

### main 브랜치만 알림

- IF 노드 추가
    
- 조건:
    

```text
{{$json.ref}} contains "main"
```

---

### Merge Request만 알림

```text
{{$json.object_kind}} = "merge_request"
```

---

### 카드형 메시지 (버튼 포함)

HTTP Request Body를 MessageCard JSON으로 변경하면 가능

---

## 최종 요약

- GitLab → Teams **직결은 불안정**
    
- 핵심은:
    
    - Set 노드에서 `text` 생성
        
    - HTTP Request Body에 `text: {{$json.text}}`


## 참고 링크

- GitLab Webhook 공식 문서  
https://docs.gitlab.com/ee/user/project/integrations/webhooks.html  

- n8n Webhook 노드  
https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/  

- n8n HTTP Request 노드  
https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httpRequest/  

- Microsoft Teams Incoming Webhook  
https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook  

- Microsoft Teams MessageCard 포맷  
https://learn.microsoft.com/en-us/outlook/actionable-messages/message-card-reference  

- n8n Docker 설치  
https://docs.n8n.io/hosting/installation/docker/  

- GitLab Webhook 이벤트 JSON 샘플  
https://docs.gitlab.com/ee/user/project/integrations/webhook_events.html  

- Teams 카드 미리보기 도구  
https://adaptivecards.io/designer/


