# 2. 공격 시도 탐지 대시보드

## 📌 목적
최근 24시간 내 공격 이벤트, 의심 IP, 불법 요청을 추적하여 보안 위협을 조기 발견.

---

## 📊 주요 패널

### 1) 공격 시도 이벤트 수 (24H)
- SPL:
index=main sourcetype=access_combined status>=400
| stats count AS "공격 시도 수 (최근 24시간)"

---

### 2) 공격 IP Top10
- SPL:
index=main sourcetype=access_combined status>=400
| where is...y clientip
| sort - 공격시도수
| head 10
| rename clientip AS IP

---

### 3) HTTP 상태코드 분포
- SPL:
index=main sourcetype=access_combined
| stats count by status
| sort - count

---

### 4) 공격 시도 추이
- SPL:
index=main sourcetype=access_combined status>=400
| timechart span=1h count as 공격시도수

---

### 5) 의심 URI Top10
- SPL:
index=main sourcetype=access_combined status>=400
| where is...t - hits
| head 10
| rename uri_path AS "URI", hits AS "요청수"

---

### 6) 최근 의심 로그인 시도 Top20
- SPL:
index=main sourcetype=access_combined uri_path="/login.php"
...(clientip) as IP by uri_path, _time
| sort - 실패횟수
| head 20

---

## 📝 분석 포인트
- 다량 400/403/404 발생 IP  
- 자주 접근하는 위험 URI 패턴  
