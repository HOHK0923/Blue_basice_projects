# 🚨 웹쉘 업로드 & 실행 징후 탐지 (Webshell Upload & Execution Detection)

Apache access_log 기반으로 **웹쉘 업로드 시도(POST)** 및  
**웹쉘 실행(file.php 호출)** 패턴을 탐지합니다.

공격자는 다음 단계를 거쳐 웹쉘 공격을 수행합니다:

1) `POST /new_post.php` — 파일 업로드 시도  
2) `GET /file.php` — 업로드된 웹쉘 실행  

이 두 동작이 짧은 시간(30초) 안에 발생할 경우 고위험 이벤트로 판단합니다.

---

## 1. Alert 조건 (SPL)

```spl
index=main source="/var/log/httpd/access_log"
| eval is_upload   = if(method="POST" AND uri="/new_post.php", 1, 0)
| eval is_webshell = if(method="GET"  AND uri="/file.php",      1, 0)
| bin _time span=30s
| stats
    sum(is_upload)   AS upload_hits
    sum(is_webshell) AS webshell_hits
    values(uri)      AS pages
    count            AS req_count
  BY clientip _time
| where upload_hits >= 2 or webshell_hits >= 1
```

---

## ✔ 조건 설명

| 항목 | 의미 |
|------|------|
| `is_upload` | 웹쉘 업로드 POST 요청 |
| `is_webshell` | 웹쉘 실행 요청(file.php) |
| `upload_hits ≥ 2` | 업로드 시도가 30초 내 2회 이상 |
| `webshell_hits ≥ 1` | 웹쉘 실행 1회 이상 발생 시 즉시 경고 |

---

## 2. Discord Webhook Payload (Better Webhook App)

> webhook.json 내용을 그대로 Splunk Alert → Webhook Payload에
