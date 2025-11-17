# 1. 접속 및 인증 모니터링 대시보드

## 📌 목적
로그인 요청/실패/의심 IP 등을 통해 인증 기반 공격 탐지 및 이상 징후 파악.

---

## 📊 주요 패널

### 1) 로그인 요청 수
- SPL:
index=main sourcetype=access_combined uri_path="/login.php" | stats count AS "로그인 요청 수"

---

### 2) 시간대별 로그인 요청 추이
- SPL:
index=main sourcetype=access_combined uri_path="/login.php"
| timechart span=1h count as "로그인 요청 수"

---

### 3) 시간대별 로그인 실패율(%)
- SPL:
index=main sourcetype=access_combined uri_path="/login.php"
..." = round(100*fail/(fail+success), 2)
| fields _time "실패율(%)"

---

### 4) 상위 로그인 실패 IP (최근 7일)
- SPL:
index=main sourcetype=access_combined uri_path="/login.php" s...- fails
| head 10
| rename clientip as IP, fails as "실패 횟수"

---

## 📝 분석 포인트
- 로그인 실패 IP가 특정 시간대 집중 여부
- 반복 실패 → 성공 로그인 패턴 존재 여부
