# LAMP 구축 및 기본 설정 매뉴얼

## 1. 시스템 개요
- **AMI:** Amazon Linux 2023 (x86_64) — `first-team-project-seoul-copy`
- **Instance Type:** t3.medium
- **Public IP:** 15.164.94.241
- **Private IP:** 172.31.40.109
- **Key Pair:** seoul-key.pem
- **Default User:** ec2-user

## 2. 주요 구성요소 버전
| 구성요소 | 버전 |
|-----------|--------|
| Apache | 2.4.65 |
| PHP | 8.1.33 |
| MariaDB | 10.5.29 |

## 3. 서비스 상태
- httpd: active (running), enabled
- mariadb: active (running), enabled

## 4. SSH 접속 예시
```bash
ssh -i seoul-key.pem ec2-user@15.164.94.241
