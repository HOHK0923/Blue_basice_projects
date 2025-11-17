# ⚠️ 비정상적인 URI 다양성 증가 탐지 (Abnormal URI Diversity)

웹쉘(`file.php`) 호출 이후 짧은 시간 안에  
**URI 종류가 급격히 많아지거나**,  
**전체 요청 수가 급증하는 패턴**을 탐지하는 Alert입니다.

웹쉘이 이미 심어진 뒤, 공격자가 내부를 이것저것 건드리며 탐색/추가 공격을 할 때  
다양한 경로로 요청이 튀어나오는 특징을 이용한 룰입니다.

---

## 1. Alert 조건 (SPL)

```spl
index=main source="/var/log/httpd/access_log"
| eval is_webshell = if(method="GET" AND like(uri, "%file.php%"), 1, 0)
| bin _time span=1m
| stats
    max(is_webshell) AS used_webshell
    dc(uri) AS unique_uri
    count AS total_req
    values(uri) AS pages
  BY clientip _time
| where used_webshell = 1 AND (unique_uri >= 3 OR total_req >= 40)
