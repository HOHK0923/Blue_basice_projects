# 자동 백업 스크립트 운영 매뉴얼

## 1) 개요
LAMP 서버의 핵심 데이터(웹 디렉토리, DB 덤프)를 매일 백업하도록 설정하는 가이드입니다.  
주기적 백업을 통해 서비스 장애 또는 공격 발생 시 신속한 복구를 목표로 합니다.

---

## 2) 백업 대상
| 항목 | 경로 | 설명 |
|------|------|------|
| 웹 서비스 파일 | `/var/www/html/` | 웹 콘텐츠 및 설정 |
| MariaDB 데이터 | `/var/lib/mysql/` | 데이터베이스 데이터 파일 |
| DB 덤프 | `/home/ec2-user/db_dumps/` | 논리적 백업 (mysqldump) |
| 로그 백업 | `/var/log/httpd/`, `/var/log/secure` | 주요 로그 |

---

## 3) 백업 스크립트 생성
```bash
sudo mkdir -p /home/ec2-user/backup
sudo vi /home/ec2-user/backup/full_backup.sh
````

내용:

```bash
#!/usr/bin/env bash
# === full_backup.sh ===
# 날짜 기반 백업
DATE=$(date +"%Y%m%d_%H%M")
BACKUP_DIR="/home/ec2-user/backup/${DATE}"
DB_DUMP_DIR="/home/ec2-user/db_dumps"
LOG_FILE="/home/ec2-user/backup/backup.log"

mkdir -p "$BACKUP_DIR"
mkdir -p "$DB_DUMP_DIR"

echo "[${DATE}] 백업 시작" >> "$LOG_FILE"

# 1) DB 덤프
mysqldump -u root -p'비밀번호' --all-databases > "${DB_DUMP_DIR}/alldb_${DATE}.sql" 2>>"$LOG_FILE"

# 2) 웹 디렉토리 압축
tar -czf "${BACKUP_DIR}/web_backup_${DATE}.tar.gz" /var/www/html 2>>"$LOG_FILE"

# 3) 로그 백업
tar -czf "${BACKUP_DIR}/logs_${DATE}.tar.gz" /var/log/httpd /var/log/secure 2>>"$LOG_FILE"

# 4) 오래된 백업 삭제 (7일 이상)
find /home/ec2-user/backup/ -type d -mtime +7 -exec rm -rf {} \; 2>>"$LOG_FILE"

echo "[${DATE}] 백업 완료" >> "$LOG_FILE"
```

권한 설정:

```bash
sudo chmod +x /home/ec2-user/backup/full_backup.sh
```

---

## 4) 실행 테스트

```bash
bash /home/ec2-user/backup/full_backup.sh
ls /home/ec2-user/backup/
```

---

## 5) 자동화 (cron)

```bash
sudo crontab -e
```

추가:

```
0 3 * * * /home/ec2-user/backup/full_backup.sh >/dev/null 2>&1
```

→ 매일 새벽 3시에 자동 백업 실행

---

## 6) 백업 검증

* 압축 파일 크기 확인
* tar 복원 테스트:

```bash
tar -tzf /home/ec2-user/backup/<날짜>/web_backup_<날짜>.tar.gz | head
```

* DB 복원 테스트:

```bash
mysql -u root -p < /home/ec2-user/db_dumps/alldb_<날짜>.sql
```

---

## 7) 원격 백업 (선택)

S3 연동 예시:

```bash
aws s3 cp /home/ec2-user/backup/ s3://team-backup-bucket/ --recursive
```

---

## 8) 요약

| 항목      | 상태                                     |
| ------- | -------------------------------------- |
| 백업 스크립트 | ✅ /home/ec2-user/backup/full_backup.sh |
| 주기 설정   | ✅ 매일 03:00                             |
| 보존 정책   | ✅ 7일                                   |
| 로그 기록   | ✅ backup.log                           |
| 원격 저장   | ⚙️ 선택 (S3 연동 시)                        |

```
```
