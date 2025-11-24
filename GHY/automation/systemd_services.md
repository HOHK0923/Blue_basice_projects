
# systemd 서비스 운영 매뉴얼

## 1) 개요
Apache(httpd)와 MariaDB를 포함한 주요 데몬을 **자동 시작/자동 복구**하도록 관리하는 가이드입니다.  
대상 OS: Amazon Linux 2023 (systemd 기반)

---

## 2) 기본 명령어
```bash
# 상태/로그
systemctl status httpd --no-pager
journalctl -u httpd -n 100 --no-pager

# 시작/중지/재시작
sudo systemctl start httpd
sudo systemctl restart httpd
sudo systemctl stop httpd

# 부팅 시 자동 시작
sudo systemctl enable httpd
sudo systemctl disable httpd

# 데몬 리로드(유닛 변경 후)
sudo systemctl daemon-reload
````

---

## 3) 자동 재시작(실패 시) 설정

서비스가 비정상 종료되면 자동으로 복구되도록 **drop-in override**를 추가합니다.

### httpd

```bash
sudo systemctl edit httpd
```

편집기에 아래 내용 입력/저장(`/etc/systemd/system/httpd.service.d/override.conf`):

```
[Service]
Restart=on-failure
RestartSec=5s
StartLimitIntervalSec=300
StartLimitBurst=5
```

적용:

```bash
sudo systemctl daemon-reload
sudo systemctl restart httpd
```

### mariadb

```bash
sudo systemctl edit mariadb
```

```
[Service]
Restart=on-failure
RestartSec=5s
StartLimitIntervalSec=300
StartLimitBurst=5
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart mariadb
```

> 참고: 네트워크 준비 후 시작 보장

```
[Unit]
Wants=network-online.target
After=network-online.target
```

---

## 4) 헬스체크 + 자동 복구(타이머)

HTTP 200이 아닐 때 재시작하는 간단한 헬스체크를 **타이머**로 주기 실행합니다.

### (1) 헬스 스크립트

```bash
sudo tee /usr/local/bin/web-healthcheck.sh >/dev/null <<'EOF'
#!/usr/bin/env bash
URL="http://127.0.0.1/health.php"
if curl -fsS --max-time 5 "$URL" >/dev/null; then
  exit 0
else
  systemctl restart httpd
  exit 1
fi
EOF
sudo chmod +x /usr/local/bin/web-healthcheck.sh
```

### (2) 유닛/타이머

`/etc/systemd/system/web-healthcheck.service`

```
[Unit]
Description=Web health check (restart httpd on failure)
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/web-healthcheck.sh
```

`/etc/systemd/system/web-healthcheck.timer`

```
[Unit]
Description=Run web health check every 5 minutes

[Timer]
OnBootSec=2m
OnUnitActiveSec=5m
Persistent=true

[Install]
WantedBy=timers.target
```

적용/실행:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now web-healthcheck.timer
systemctl list-timers | grep web-healthcheck
```

---

## 5) journald 로그 운영

영구 저장 및 정리 정책:

```bash
# 영구 저장
sudo sed -i 's/^#\?Storage=.*/Storage=persistent/' /etc/systemd/journald.conf
sudo systemctl restart systemd-journald

# 용량 관리(예: 200MB 유지)
sudo journalctl --vacuum-size=200M
```

서비스별 조회:

```bash
journalctl -u httpd -S today --no-pager
journalctl -u mariadb -S "2025-11-24 00:00" -U "2025-11-24 23:59"
```

---

## 6) 부팅/종료 시퀀스 튜닝(선택)

DB → 웹 순서 보장을 원하면 httpd 유닛에 추가:

```
[Unit]
After=mariadb.service
Requires=mariadb.service
```

---

## 7) 트러블슈팅 체크리스트

* `systemctl status <svc>` 에서 **Active/Loaded/Drop-In** 확인
* `journalctl -xeu <svc>` 로 최근 에러 추적
* 설정 변경 후 `daemon-reload` 누락 여부 확인
* 포트 충돌 시 `ss -tulpn | grep :80` 점검
* 반복 크래시 시 `RestartSec` 증가, `StartLimitBurst`/`IntervalSec` 조정

---

## 8) 요약

* 실패 자동 재시작: `Restart=on-failure` + `RestartSec=5s`
* 주기 헬스체크: **oneshot 서비스 + 타이머**
* 로그: journald 영구화 및 용량 관리
* 네트워크·의존성: `After=network-online.target`, `Requires=mariadb.service`

```
```
