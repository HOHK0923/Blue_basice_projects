# 🚨 HTTP 플러드(과다 요청) 의심 경고

특정 IP에서 짧은 시간 동안 비정상적으로 많은 HTTP 요청이 발생할 경우  
**HTTP 플러드(DoS 형태) 의심 트래픽**으로 탐지하는 Alert입니다.

---

## 1. Alert 조건 (SPL)

```spl
index=main source="/var/log/httpd/access_log"
| bin _time span=1m
| stats count AS req_count values(uri) AS pages BY clientip _time
| where req_count >= 10
```

### ✔ 설명
- Apache access_log 기준
- 1분 단위로 요청을 집계
- 동일 clientip 기준으로 요청 수(req_count) 계산
- 1분 동안 요청 수가 10건 이상인 경우 Alert 발생

---

## 2. Discord Webhook (Better Webhook App)

webhook.json 파일 내용을 Splunk Alert → Actions → Webhook Payload에 그대로 입력하면 됩니다.

---

## 3. 주요 필드

| 필드명 | 의미 |
|-------|------|
| `clientip` | 요청 발생 IP |
| `req_count` | 해당 시간대 요청 수 |
| `pages` | 요청된 URI 목록 |
| `_time` | 이벤트 시간 |

