# Apache ModSecurity 로그 요약 및 분석

## 1. 엔진 개요
- **모듈:** ModSecurity (Apache용 WAF)
- **룰셋:** OWASP Core Rule Set (CRS) 4.2.0  
- **경로:** `/etc/httpd/modsecurity.d/activated_rules/`
- **로그 파일:** `/var/log/httpd/error_log`

---

## 2. 탐지된 주요 이벤트
| ID | 유형 | 심각도 | 설명 |
|----|------|---------|------|
| 920350 | Warning | Low | Host 헤더가 IP 주소 형태로 감지됨 |
| 930130 | Critical | High | `.env` 파일 접근 시도 (LFI 탐지) |
| 941160 | Critical | High | XSS Injection (HTML 속성 내 스크립트 감지) |
| 949110 | Critical | High | Inbound Anomaly Score 초과로 요청 차단 |

---

## 3. 실제 로그 요약

### 🔸 LFI (Local File Inclusion) 차단
```

[Mon Nov 24 10:46:46] ModSecurity: Access denied with code 403
[id "930130"] [msg "Restricted File Access Attempt"]
[data "Matched Data: .env found within REQUEST_FILENAME: /.env"]

```
→ 외부에서 `.env` 파일 접근 시도 차단됨 (403 응답)

---

### 🔸 XSS (Cross Site Scripting) 탐지
```

[id "941160"] [msg "NoScript XSS InjectionChecker: HTML Injection"]
[data "Matched Data: onmouseover= found within ARGS:full_name"]

```
→ `onmouseover` / `style` 속성 기반 공격 탐지 및 차단

---

### 🔸 Host 헤더 기반 경고
```

[id "920350"] [msg "Host header is a numeric IP address"]

```
→ 도메인 대신 IP(15.164.94.241)로 접근 시 경고 발생

---

### 🔸 종합 평가
```

[id "949110"] [msg "Inbound Anomaly Score Exceeded (Total Score: 8)"]

````
→ 누적 위험 점수 5 이상 시 자동 차단 (403 반환)

---

## 4. 운영 개선 포인트
- **도메인 기반 접근 유도:** `ServerName` 설정으로 IP 직접 접근 방지  
- **정상 트래픽 화이트리스트:** 팀 내부 테스트 IP 추가 허용  
- **XSS/LFI 룰 튜닝:** 특정 파라미터(`full_name`, `profile.php`) 예외처리 고려  
- **로그 로테이션:** `/var/log/httpd/error_log` → `/etc/logrotate.d/httpd`에 추가

---

## 5. 참고 명령어
```bash
sudo tail -f /var/log/httpd/error_log
sudo grep "ModSecurity" /var/log/httpd/error_log | less
sudo systemctl restart httpd
````

---

## 6. 요약

* WAF가 실시간으로 XSS, LFI, IP 기반 접근을 차단 중
* OWASP CRS 4.2.0 규칙에 따라 정상 작동 확인
* 공격 트래픽이 반복적으로 감지되므로, 방화벽 차단 정책(firewall-cmd)과 병행 필요

```
