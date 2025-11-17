# 3. IP 기반 위협 인텔리전스 대시보드

## 📌 목적
의심 IP 기반 국가/도시, 공격 대상 엔드포인트, 로그인 실패 패턴 등을 분석하여 위험도를 판단.

---

## 📊 주요 패널

### 1) 의심 IP 상위 20
- SPL:
index=main sourcetype=access_combi...P, fails AS "실패 횟수", total AS "전체 요청 수", fail_rate AS "실패율(%)"

---

### 2) 국가/도시 포함 의심 IP Top20
- SPL:
index=main sourcetype=access_combined
| stats count AS total...ity total fails fail_rate
| sort -fail_rate -fails
| head 20

---

### 3) 국가별 로그인 실패 현황
- SPL:
index=main sourcetype=access_combined (uri_path="/login.php" ...d=lat longfield=lon
| eval fail_rate=round(fails/total*100,2)

---

### 4) 시간대별 로그인 실패 분석
- SPL:
index=main sourcetype=access_combined (uri_path="/login.php" ...ails count as total
| eval fail_rate=round(fails/total*100,2)

---

### 5) 반복 로그인 실패 후 성공 IP 탐지
- SPL:
index=main sourcetype=access_combined (uri_path="/login.php")...lientip, user
| where fails>=3 AND success>=1
| sort - fails

---

### 6) 공격 대상 엔드포인트 Top20
- SPL:
index=main sourcetype=access_combined (uri_path="/login.php" ...)
| stats count AS hits by uri_path
| sort - hits
| head 20

---

### 7) 공격 발생 국가 Top10
- SPL:
index=main
| where isnotnull(clientip)
| iplocation clientip
| stats count by Country
| sort -count
| head 10

---

### 8) 의심 IP 로그인 실패 과다 탐지
- SPL:
index=main ("login" OR "auth") ("fail" OR "invalid" OR "denie...s>100, "⚠️ 의심 로그인 다수 탐지", "✅ 최근 1시간 내 이상 없음")
| table status

---

## 📝 분석 포인트
- 특정 국가/도시 집중 실패  
- 실패 → 성공 패턴의 알려진 공격 방식 가능성  
