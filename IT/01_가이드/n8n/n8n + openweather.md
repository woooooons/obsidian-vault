---
title: n8n + openweather
date: 2026-01-12
tags:
  - it
  - 가이드
---
## 사전 준비

1. **n8n(1.120.4)**
    
2. **discode API KEY**
    
3. **openweathermap API KEY**
    

---

## STEP 1: node 생성

### 1-1 Schedule 노드 생성
![[Pasted image 20260112120437.png]]  

### 1-2 HTTP 노드 생성

**날씨**
https://api.openweathermap.org/data/3.0/onecall?lat=&lon=&exclude=current,minutely,hourly,alerts&units=metric&lang=kr&appid=
경도, 위도, api 채우기
![[Pasted image 20260112120841.png]]  

**먼지**
https://api.openweathermap.org/data/2.5/air_pollution/history?lat=&lon=&appid=
경도, 위도, api 채우기

쿼리 파라미터에 end : {{ Math.floor(Date.now() / 1000) }}, start : {{ Math.floor(Date.now() / 1000) - 7200 }}
![[Pasted image 20260112121006.png]]  

### 1-3 merge 노드 생성
![[Pasted image 20260112121032.png]]  


### 1-4 js 노드 생성
![[Pasted image 20260112121104.png]]
```js
const data = $input.item.json;

// ===== 날씨 =====
if (!data.daily || !Array.isArray(data.daily)) {
  throw new Error("날씨 daily 데이터가 없습니다.");
}

// ===== 미세먼지 =====
if (!data.list || data.list.length < 2) {
  throw new Error("미세먼지 2시간 데이터가 부족합니다.");
}

const today = data.daily[0];

// 최근 2시간 미세먼지
const air1 = data.list[data.list.length - 1]; // 가장 최근
const air2 = data.list[data.list.length - 2]; // 1시간 전

// 날짜
const date = new Date(today.dt * 1000)
  .toLocaleDateString('ko-KR', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit'
  })
  .replace(/\. /g, '.')
  .replace('.', '. ');

// ===== 날씨 정보 =====
const temp = Math.round(today.temp.day);
const feelsLike = Math.round(today.feels_like.day);
const tempMin = Math.round(today.temp.min);
const tempMax = Math.round(today.temp.max);
const description = today.weather[0].description;
const humidity = today.humidity;
const windSpeed = today.wind_speed;

// ===== 미세먼지 수치 =====
const pm10_1 = air1.components.pm10;
const pm10_2 = air2.components.pm10;

const pm25_1 = air1.components.pm2_5;
const pm25_2 = air2.components.pm2_5;

// ===== 미세먼지 경보 판정 =====
let dustLevel = "정상";
let dustMessage = "대기질이 양호합니다.";

if (pm10_1 >= 150 && pm10_2 >= 150) {
  dustLevel = "경보";
  dustMessage = "미세먼지 경보: 외출을 자제하세요.";
} else if (pm10_1 >= 75 && pm10_2 >= 75) {
  dustLevel = "주의보";
  dustMessage = "미세먼지 주의보: 장시간 실외활동을 피하세요.";
}

// ===== 바람 =====
let windMessage;
if (windSpeed >= 5.5) {
  windMessage = `바람이 강합니다 (${windSpeed}m/s)`;
} else if (windSpeed >= 3.4) {
  windMessage = `바람이 약간 있습니다 (${windSpeed}m/s)`;
} else {
  windMessage = `바람이 잔잔합니다 (${windSpeed}m/s)`;
}

// ===== Discord 메시지 =====
const message =
`📌 오늘의 날씨 (${date})

🌤️ 날씨: ${description}
🌡️ 기온: ${temp}°C (체감 ${feelsLike}°C)
⬆️ 최고 / ⬇️ 최저: ${tempMax}°C / ${tempMin}°C
💧 습도: ${humidity}%
🌬️ ${windMessage}

🌫️ 미세먼지 상태: ${dustLevel}
• PM10: ${pm10_2} → ${pm10_1} ㎍/m³
• PM2.5: ${pm25_2} → ${pm25_1} ㎍/m³
⚠️ ${dustMessage}`;

return [
  {
    json: { message }
  }
];

```

### 1-5 discode 노드 생성
![[Pasted image 20260112121255.png]]  
크리덴셜 추가 후 선택해야함  
![[Pasted image 20260112121212.png]]  

## STEP 2: 워크 플로우 실행
![[Pasted image 20260112121409.png]]  
