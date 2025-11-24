# healthmash.net 도메인 운영 가이드

## 1) 개요
- **도메인:** healthmash.net  
- **소유/이관:** 팀장 보유 → (2025-11) 본인 AWS 계정으로 이관  
- **운영 목적:** LAMP 기반 웹서비스 및 보안 모니터링 데모
- **대상 인스턴스:** EC2 `t3.medium` (Amazon Linux 2023)  
- **공인 IP:** `15.164.94.241`

---

## 2) DNS 구성 (현재 권장값)
Route 53 호스티드 존(또는 외부 레지스트라 DNS)에 아래 레코드 생성:

| Type | Name | Value | TTL | 비고 |
|-----|------|-------|-----|------|
| A   | `@` (healthmash.net) | `15.164.94.241` | 300 | 루트 도메인 |
| CNAME | `www` | `healthmash.net` | 300 | www → 루트 |
| TXT | `_amazonses` | (선택) | 300 | SES 사용 시 |
| TXT | `@` | `v=spf1 include:amazonses.com ~all` | 300 | 메일 발송 시 권장 |
| CNAME | `_acme-challenge` | (발급 시 값) | 60 | DNS-01 인증서 발급용 |

> 레지스트라가 외부라면 **네임서버(NS)** 를 Route 53의 NS로 교체해야 합니다.

---

## 3) 이관 메모
- 원 소유자: 팀장  
- 이관 방식: **네임서버 이전**(권장) 또는 **도메인 트랜스퍼**  
- 완료 체크리스트  
  - [ ] 레지스트라 WHOIS에 소유자/관리자 이메일 갱신  
  - [ ] 네임서버 Route 53 NS로 변경 반영  
  - [ ] DNS 전파 확인 (`dig NS healthmash.net +short`)

---

## 4) 서버(Apache) 가상호스트 예시
```apache
# /etc/httpd/conf.d/healthmash.conf
<VirtualHost *:80>
    ServerName healthmash.net
    ServerAlias www.healthmash.net
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog  /var/log/httpd/healthmash_error.log
    CustomLog /var/log/httpd/healthmash_access.log combined
</VirtualHost>
````

적용:

```bash
sudo systemctl reload httpd
```

---

## 5) SSL/TLS(권장)

### 옵션 A) Let’s Encrypt (Certbot, Apache)

```bash
sudo dnf install -y certbot python3-certbot-apache
sudo certbot --apache -d healthmash.net -d www.healthmash.net
# 자동 갱신 확인
sudo systemctl status certbot-renew.timer
```

### 옵션 B) AWS ACM(+ALB/CloudFront 사용 시)

* ACM에서 인증서 발급 → DNS 검증(CNAME) → 로드밸런서/CloudFront에 연결

---

## 6) 점검/검증

```bash
# DNS
dig A healthmash.net +short
dig CNAME www.healthmash.net +short

# HTTP SNI/호스트 헤더 체크
curl -I http://healthmash.net
curl -I http://www.healthmash.net

# 방화벽/포트
sudo firewall-cmd --list-all | grep -E 'http|https'
sudo ss -tulpn | grep -E '(:80|:443)'
```

---

## 7) 운영 정책

* IP 변경 시: A 레코드만 교체 → 5분(TTL=300) 내 외부 반영
* 유지보수 창: 평일 09:00~18:00 내 작업, 종료 전 서비스 헬스체크
* 공격 대응:

  * ModSecurity 차단 이벤트 증가 시 `security/modsecurity_logs.md` 참조
  * 필요 시 `firewall_rules.md`의 rich-rule로 IP 임시 차단

---

## 8) 위험/백업

* **단일 EC2 직접 연결** 구조라 ISP/인스턴스 장애 시 서비스 중단 가능

  * (대안) ALB + Auto Scaling / CloudFront 캐시
* 백업: `automation/backup_script.md` 정책 준수(일일/7일 보존)

---

## 9) 변경 이력(요약)

* 2025-11: 도메인 운영 주체를 팀장 → AWS 계정(본인)으로 이관
* 2025-11: A 레코드 `15.164.94.241` 설정, www CNAME 정리

```
```
