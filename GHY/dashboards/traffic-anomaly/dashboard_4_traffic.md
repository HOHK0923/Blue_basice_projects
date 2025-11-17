# 4. 트래픽 이상 징후 감시 대시보드

## 📌 목적
트래픽 급증, 특정 엔드포인트 집중 요청, 비정상 User-Agent 등을 기반으로 이상 징후 탐지.

---

## 📊 주요 패널

### 1) 전체 트래픽 이벤트 수 (1H)
- SPL:
index=main sourcetype=access_combined earliest=-1h latest=now | stats count AS "이벤트 수 (1시간)"

---

### 2) 총 전송 바이트 (24H)
- SPL:
index=main sourcetype=access_combined earliest=-24h@h latest=...(isnum(bytes),bytes,0) | stats sum(bytestonum) AS "총 바이트(24h)"

---

### 3) 시간대별 트래픽 추이
- SPL:
index=main sourcetype=access_combined earliest=-24h@h latest=now | timechart span=1h count AS events sum(bytes) AS bytes

---

### 4) 상위 요청 엔드포인트 Top20
- SPL:
index=main sourcetype=access_combined earliest=-24h@h latest=now | stats count AS hits BY uri_path | sort -hits | head 20

---

### 5) 상위 소스 IP Top10
- SPL:
index=main sourcetype=access_combined earliest=-24h@h latest=...w | stats count AS events BY clientip | sort -events | head 10

---

### 6) 상위 User-Agent
- SPL:
index=main sourcetype=access_combined earliest=-24h@h latest=...now | stats count AS hits BY user_agent | sort -hits | head 10

---

### 7) 트래픽 발생 국가 Top10
- SPL:
index=main sourcetype=access_combined earliest=-7d@d latest=n...ip | stats count AS events BY Country | sort -events | head 10

---

### 8) z-score 기반 이상 트래픽 탐지
- SPL:
index=main sourcetype=access_combined earliest=-24h@h latest=..."anomaly", "normal")
| table _time cnt mean sd zscore anomaly

---

### 9) 최근 이상 지점 (z > 3)
- SPL:
index=main sourcetype=access_combined earliest=-24h@h latest=...ean)/sd,0)
| where zscore>3
| stats count AS "이상발생 시점(24h)"

---

## 📝 분석 포인트
- 특정 엔드포인트 집중 공격  
- 비정상 User-Agent 비율 증가  
