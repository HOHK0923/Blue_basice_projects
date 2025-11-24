# 백업 및 스케줄 설정 매뉴얼

## 1. 크론탭 등록 확인
```bash
crontab -l
````

출력:

```
0 15 * * 1,4 /home/ec2-user/backup/full_backup.sh
```

* 매주 월요일, 목요일 오후 3시 실행
* 백업 스크립트 경로: `/home/ec2-user/backup/full_backup.sh`

---

## 2. 백업 스크립트 예시

```bash
#!/bin/bash
DATE=$(date +%F)
BACKUP_DIR="/home/ec2-user/backup"
LOG_FILE="/var/log/backup.log"

echo "[$DATE] Starting backup..." >> $LOG_FILE
mysqldump -u root --all-databases > "$BACKUP_DIR/db_backup_$DATE.sql"
tar -czf "$BACKUP_DIR/www_backup_$DATE.tar.gz" /var/www/html

# 2주 이상 된 백업 자동 삭제
find "$BACKUP_DIR" -type f -mtime +14 -delete

echo "[$DATE] Backup completed." >> $LOG_FILE
```

* DB와 웹 루트를 동시에 백업
* `/var/log/backup.log`에 실행 이력 기록
* 자동 정리로 디스크 사용량 관리

---

## 3. 권장 디렉토리 구조

```bash
/home/ec2-user/backup/
 ├── full_backup.sh
 ├── db_backup_YYYY-MM-DD.sql
 ├── www_backup_YYYY-MM-DD.tar.gz
 └── logs/
```

---

## 4. 점검 명령어

```bash
sudo systemctl status crond
sudo tail -n 50 /var/log/backup.log
df -h /home
```

---

## 5. 개선 포인트

* 백업 파일 암호화 (예: `gpg --symmetric`)
* 외부 스토리지 전송 (AWS S3 CLI 활용)
* Slack 또는 이메일 알림 추가 가능
* 백업 완료 후 해시 검증 (`md5sum`)으로 무결성 체크

---

## 6. 요약

* 주 2회 자동 백업 스크립트 정상 작동
* 백업 로그 및 자동 삭제 기능 포함
* 추후 AWS S3 업로드 자동화 고려 가능

```
