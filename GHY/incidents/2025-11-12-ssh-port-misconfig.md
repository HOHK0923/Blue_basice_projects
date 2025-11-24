````markdown
# Incident Postmortem — SSH Port Misconfiguration

- **ID:** INC-2025-11-12-SSH-PORT  
- **서비스:** EC2 (Amazon Linux 2023, t3.medium)  
- **도메인/서비스:** healthmash.net  
- **영향 범위:** 운영 EC2 SSH 원격접속 불가  
- **발생 일시:** 2025-11-12 (KST)  
- **해결 일시:** 2025-11-12 (KST)  
- **감지 방법:** 팀원 보고 + 직접 접속 실패  
- **담당:** 블루팀 (본인)

---

## 1) 요약
팀원이 테스트 목적으로 **SSH 포트(22 → 2222) 변경**을 시도하면서 접속이 끊김.  
**SSM Session Manager**로 인스턴스에 진입해 설정을 복구하고,  
**보안 그룹/방화벽/sshd**를 정정하여 정상화.

---

## 2) 타임라인 (KST)
- **11-12 14:xx** 팀원 테스트로 SSH 포트 2222 생성/변경, 이후 접속 불가 발생  
- **11-12 15:xx** 세션 매니저(SSM)로 인스턴스 접속 성공  
- **11-12 15:xx** `sshd_config` 원복, 보안 그룹/방화벽 점검  
- **11-12 16:xx** SSH(22) 정상 접속 확인, 사고 종료

---

## 3) 영향
- 관리 접속(운영/장애 대응) 지연  
- 잠재적 장애 확대 위험(무중단 배포/복구 불가 상태)

---

## 4) 원인
- 운영 서버에서 **무검증 포트 변경** 수행  
- 변경 시 **보안 그룹/OS 방화벽/sshd** 간 일관성 검증 부재  
- 변경 절차와 롤백 가이드 미정의

---

## 5) 복구 절차 (실행 내역)

### 5.1 SSM을 통한 진입
```bash
# AWS Console → Session Manager로 인스턴스 접속
````

### 5.2 SSH 데몬 설정 원복

```bash
sudo vi /etc/ssh/sshd_config
# Port 22 로 설정
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers ec2-user
sudo systemctl restart sshd
sudo systemctl status sshd --no-pager
```

### 5.3 보안 그룹 / NACL 확인

* SG Inbound: **TCP 22**(관리 IP), 웹서비스 관련 포트 정책 확인
* NACL: 22 및 에페메럴(1024–65535) 정상 허용 검증

### 5.4 OS 방화벽 확인

```bash
sudo firewall-cmd --list-ports
# 필요 시 추가
sudo firewall-cmd --permanent --add-port=22/tcp
sudo systemctl restart firewalld
```

### 5.5 접속 확인

```bash
ssh -i <key.pem> ec2-user@<PUBLIC_IP>
```

---

## 6) 증빙 (채팅 로그/명령 결과)

* 파일: `docs/chatlogs/ChatGPT-AWS-CLI-접속-문제-해결.csv`
  (원본: `/mnt/data/ChatGPT-AWS CLI 접속 문제 해결.csv`)

---

## 7) 재발 방지 대책

* **운영 변경 승인제:** 포트·방화벽·핵심 서비스 변경은 사전 리뷰/승인
* **롤백 가이드 추가:** `security/ssh_security.md`에 변경→복구 절차 추가
* **자동 점검 스크립트:** `systemd_services.md`에 22포트 리스닝 검사 추가
* **테스트 정책 분리:** 운영 서버 테스트 금지 (별도 스테이징 환경 사용)
* **감사 로깅:** 변경 이력(명령/커밋)을 `docs/change_log.md`에 기록

---

## 8) 상태

| 항목       | 상태                                |
| -------- | --------------------------------- |
| sshd(22) | ✅ 정상                              |
| 보안 그룹    | ✅ 정상                              |
| OS 방화벽   | ✅ 22/tcp 허용                       |
| 세션 매니저   | ✅ 운영용 활성화                         |
| 운영 문서 갱신 | ✅ 완료 (`security/ssh_security.md`) |

---

## 9) 교훈

* 네트워크/보안 구성 변경은 **SG, NACL, OS FW, sshd 설정**이 일관돼야 함.
* **SSM(Session Manager)** 은 비상 복구용으로 매우 유용하므로 항상 활성 유지.

```
```
