
# Fail2Ban 설정 및 동작 확인 매뉴얼

## 1. 개요
SSH 및 웹 서비스에 대한 무차별 대입 공격(Brute Force Attack)을 방어하기 위해  
Fail2Ban을 설치하고 구성하는 절차입니다.  
로그 기반 자동 차단을 통해 비정상 접속 시도를 실시간으로 탐지 및 차단합니다.

---

## 2. 설치 및 서비스 등록
```bash
sudo dnf install fail2ban -y
sudo systemctl enable --now fail2ban
sudo systemctl status fail2ban --no-pager
````

결과 예시:

```
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/usr/lib/systemd/system/fail2ban.service; enabled)
     Active: active (running)
```

---

## 3. 주요 설정 파일

기본 설정 파일은 `/etc/fail2ban/jail.conf`이며, 수정 전 복사본 생성:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vi /etc/fail2ban/jail.local
```

---

## 4. SSH 보호 설정

`[sshd]` 섹션을 아래와 같이 수정합니다.

```
[sshd]
enabled = true
port    = ssh
logpath = /var/log/secure
maxretry = 5
bantime = 3600
findtime = 600
```

의미:

| 옵션         | 설명           |
| ---------- | ------------ |
| `enabled`  | 감시 활성화 여부    |
| `maxretry` | 연속 실패 허용 횟수  |
| `bantime`  | 차단 시간(초)     |
| `findtime` | 실패 감지 기간(초)  |
| `logpath`  | 감시할 로그 파일 경로 |

---

## 5. 서비스 재시작 및 상태 확인

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

결과 예시:

```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 2
|  |- Total failed:     8
|  `- File list:        /var/log/secure
`- Actions
   |- Currently banned: 1
   `- Banned IP list:   203.0.113.45
```

---

## 6. 차단 해제

```bash
sudo fail2ban-client set sshd unbanip <IP_ADDRESS>
```

---

## 7. 로그 위치

Fail2Ban 자체 로그:

```bash
sudo tail -f /var/log/fail2ban.log
```

---

## 8. Apache 보호 (선택)

웹 공격 시도(IP 반복 요청 등)에 대해 `apache-auth` 규칙도 활성화 가능:

```
[apache-auth]
enabled = true
port    = http,https
logpath = /var/log/httpd/*access_log
maxretry = 10
bantime = 3600
```

---

## 9. 요약

| 항목     | 상태                       |
| ------ | ------------------------ |
| SSH 보호 | ✅ 활성화                    |
| 로그 감시  | `/var/log/secure`        |
| 차단 기준  | 5회 실패 시 1시간 차단           |
| 서비스    | `fail2ban.service` 자동 기동 |

---

## 10. 참고 명령어

```bash
sudo fail2ban-client ping
sudo fail2ban-client reload
sudo fail2ban-client set sshd banaction iptables-multiport
```

```
```
