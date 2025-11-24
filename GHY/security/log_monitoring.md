# 보안 로그 모니터링 및 분석

## 1. 개요
서버 내 주요 로그 파일(`/var/log/secure`, `/var/log/httpd/`, `/var/log/fail2ban.log`)을 주기적으로 점검하여  
비정상 로그인 시도, 웹 공격 탐지, 방화벽 차단 이벤트 등을 모니터링합니다.

---

## 2. 주요 로그 파일 경로
| 로그 종류 | 파일 경로 | 내용 |
|------------|------------|------|
| SSH 접속 로그 | `/var/log/secure` | 로그인, 인증 실패, root 접근 시도 |
| Apache 접근 로그 | `/var/log/httpd/access_log` | 웹 요청 정보 |
| Apache 에러 로그 | `/var/log/httpd/error_log` | 서버 오류, ModSecurity 탐지 결과 |
| Fail2Ban 로그 | `/var/log/fail2ban.log` | 차단/해제 이벤트 기록 |

---

## 3. 실시간 로그 확인
```bash
# SSH 접근 로그 실시간 확인
sudo tail -f /var/log/secure

# Apache 로그 실시간 확인
sudo tail -f /var/log/httpd/access_log
sudo tail -f /var/log/httpd/error_log

# Fail2Ban 로그 실시간 확인
sudo tail -f /var/log/fail2ban.log
````

---

## 4. 실패한 SSH 로그인 시도 검색

```bash
sudo grep "Failed password" /var/log/secure | tail -n 20
```

성공/실패 요약:

```bash
sudo grep "Accepted password" /var/log/secure | wc -l
sudo grep "Failed password" /var/log/secure | wc -l
```

---

## 5. 특정 IP의 시도 내역 분석

```bash
sudo grep "203.0.113.45" /var/log/secure
sudo grep "203.0.113.45" /var/log/httpd/access_log
```

---

## 6. ModSecurity 탐지 로그 분석

```bash
sudo grep "\[security2:error\]" /var/log/httpd/error_log | tail -n 30
```

예시 탐지:

```
ModSecurity: Access denied with code 403 (Inbound Anomaly Score Exceeded)
```

이 메시지가 지속적으로 반복될 경우 웹 공격(예: SQLi, XSS, LFI) 시도일 가능성이 높습니다.

---

## 7. 로그 압축 및 보존

```bash
sudo journalctl --vacuum-time=30d
sudo logrotate -f /etc/logrotate.conf
```

* `/etc/logrotate.d/httpd` 설정 확인
* 기본 보존 기간: 4주 (필요 시 조정 가능)

---

## 8. 로그 통합 수집 (Splunk Forwarder 미설치 시)

현재 `/opt/splunkforwarder` 미존재 확인됨 → 추후 수동 설치 가능.

```bash
# 설치 예시 (Amazon Linux 2023 기준)
wget -O splunkforwarder.tgz https://download.splunk.com/products/universalforwarder/releases/9.1.2/linux/splunkforwarder-9.1.2-linux-2.6-x86_64.tgz
tar -xvf splunkforwarder.tgz -C /opt
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```

---

## 9. 요약

| 항목             | 상태            |
| -------------- | ------------- |
| SSH 로그         | ✅ 감시 중        |
| Apache 로그      | ✅ 활성화         |
| ModSecurity 탐지 | ✅ 정상 작동       |
| Fail2Ban 로그    | ✅ 차단 기록 확인 가능 |
| 로그 압축          | ✅ 30일 단위 순환   |

---

## 10. 추천 후속 작업

* Splunk 또는 ELK Stack 연동으로 로그 통합 시각화
* 이상 징후 자동 알림 (Webhook, 이메일, Slack 등)
* 정기 점검 스크립트(`cron` 활용) 작성

```
