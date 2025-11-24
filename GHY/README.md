
## ☁️ 서버 구성 요약
- **플랫폼:** Amazon Linux 2023 (t3.medium)  
- **스택:** Apache / PHP / MariaDB (LAMP)  
- **로그 수집:** Splunk Universal Forwarder  
- **운영 정책:** Root 계정 09:00–18:00, IPv4 일일 공지  
- **도메인:** `healthmash.net` (Route 53 관리)

---

## 🔒 주요 보안 항목
- SSH 접근제어 및 포트 고정 (`security/ssh_security.md`)  
- Fail2Ban 및 FirewallD 정책 적용  
- 로그 실시간 감시 (`security/log_monitoring.md`)  
- Splunk 연동 경보 및 웹훅 알림 자동화  

---

## ⚙️ 운영 자동화
- `systemd_services.md` — 주요 서비스 자동시작 관리  
- `backup_script.md` — DB·로그 주기 백업  
- `cron_backup.md` — 크론 스케줄 관리  

---

## 🌐 도메인 관리
- Route 53 A 레코드 → `15.164.94.241`  
- 초기 접속 실패 원인: firewalld 80/443 미허용 → 수정 완료  
- 세부 기록: `domain/healthmash_debug.md`

---

## 🚨 사고 대응 (Incident)
- **2025-11-12 SSH 포트 변경 사고**
  - 팀원이 22→2222 포트 변경 후 접속 불가
  - SSM Session Manager로 진입해 복구
  - 문서: `incidents/2025-11-12-ssh-port-misconfig.md`

---

## 📊 Splunk 구성 요약
- **대시보드:** 로그인 감시, 트래픽 이상, 공격 탐지 등 5종  
- **경보(Alert):** HTTP Flood, Webshell, 비정상 URI 다양성 탐지  

---

**작성자:** 권호영 (Blue Team)  
**업데이트:** 2025-11-24  
**환경:** AWS EC2 · Amazon Linux 2023 · Splunk 10.x
