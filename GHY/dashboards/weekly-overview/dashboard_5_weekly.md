# 5. 요일별 보안 이벤트 요약 대시보드

## 📌 목적
요일/시간대별 공격 패턴을 분석하여 반복 공격 가능성 파악.

---

## 📊 주요 패널

### 1) 요일별 이벤트/실패 요약
- SPL:
index=main sourcetype=access_combin...fail_rate = round(fails*100/events, 2)
| sort w
| fields - w

---

### 2) 요일×시간대 로그인 실패 히트맵
- SPL:
index=main sourcetype=access_combined (status>=400 OR uri_pat...weekday h
| sort w
| fields - w
| xyseries weekday h fails

---

## 📝 분석 포인트
- 특정 요일/시간 집중 공격  
