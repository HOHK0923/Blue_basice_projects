# SSH 접근 제어 및 보안 설정

## 1. 개요
AWS EC2 인스턴스(amazon-linux 2023) 환경에서 SSH 접근을 안전하게 관리하기 위한 보안 설정 지침입니다.  
비밀번호 로그인 차단, 포트 변경, 허용 IP 제한, 공개키 인증 강화 등을 포함합니다.

---

## 2. SSH 설정 확인
```bash
sudo vi /etc/ssh/sshd_config
````

중요 설정 항목:

```
Port 22                   # 필요 시 다른 포트로 변경 (예: 2222)
PermitRootLogin no        # root 직접 로그인 금지
PasswordAuthentication no # 비밀번호 로그인 비활성화
PubkeyAuthentication yes  # 공개키 인증만 허용
AllowUsers ec2-user       # 특정 사용자만 허용
```

적용 후 SSH 서비스 재시작:

```bash
sudo systemctl restart sshd
```

---

## 3. 보안 그룹 및 방화벽 설정

AWS 보안 그룹에서 SSH(22/tcp)는 **본인 IP만 허용**하도록 제한:

```
Type: SSH
Protocol: TCP
Port Range: 22
Source: My IP (예: 125.xxx.xxx.xxx/32)
```

서버 내부에서도 확인:

```bash
sudo firewall-cmd --list-all
```

---

## 4. SSH 키 관리

* 키 생성 (로컬 PC):

  ```bash
  ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
  ```
* 공개키 업로드:

  ```bash
  ssh-copy-id -i ~/.ssh/id_rsa.pub ec2-user@<EC2_IP>
  ```
* 권한 확인:

  ```bash
  chmod 700 ~/.ssh
  chmod 600 ~/.ssh/authorized_keys
  ```

---

## 5. Fail2Ban 설치 (Brute Force 방어)

```bash
sudo dnf install fail2ban -y
sudo systemctl enable --now fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vi /etc/fail2ban/jail.local
```

`sshd` 설정 섹션 수정:

```
[sshd]
enabled = true
port    = ssh
logpath = /var/log/secure
maxretry = 5
bantime = 3600
```

적용:

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

---

## 6. 접속 로그 모니터링

로그 파일:

```bash
sudo tail -f /var/log/secure
sudo grep "Failed password" /var/log/secure
sudo lastlog | head
```

---

## 7. 요약

| 항목        | 상태     |
| --------- | ------ |
| root 로그인  | ❌ 차단   |
| 비밀번호 로그인  | ❌ 비활성화 |
| 공개키 인증    | ✅ 허용   |
| SSH 접근 IP | ✅ 제한   |
| Fail2Ban  | ✅ 동작 중 |

---

## 8. 권장 추가 조치

* 포트포워딩 또는 VPN 기반 접근
* MFA SSH (예: `google-authenticator`)
* 자동 로그 분석 스크립트 연결 (예: Splunk SSH 실패 로그 수집)

```