# MariaDB 초기 설정 및 운영 가이드

## 1. 기본 정보
- **DB 엔진:** MariaDB 10.5.29  
- **상태:** active (running)  
- **자동 시작:** enabled  

```bash
sudo systemctl status mariadb
sudo systemctl is-enabled mariadb
````

---

## 2. 데이터베이스 목록

```bash
sudo mysql -e "SHOW DATABASES;"
```

출력:

```
information_schema
mysql
performance_schema
tdata
vulnerable_sns
```

---

## 3. 사용자 계정 목록

```bash
sudo mysql -e "SELECT user, host FROM mysql.user;"
```

| 사용자         | 호스트       | 설명          |
| ----------- | --------- | ----------- |
| root        | localhost | 관리자 계정      |
| firstteam   | localhost | 프로젝트 운영용 계정 |
| webuser     | localhost | 웹서버 연결용     |
| teamlead_db | localhost | 팀 리더 관리용    |
| common_user | localhost | 공용 접근용      |
| mariadb.sys | localhost | 시스템 계정      |
| mysql       | localhost | 내부용 계정      |

---

## 4. 보안 설정

초기 설치 후 다음 명령으로 기본 보안 설정을 강화합니다.

```bash
sudo mysql_secure_installation
```

* root 비밀번호 설정
* 익명 사용자 제거
* 원격 root 접속 차단
* 테스트 DB 삭제
* 권한 테이블 재로드

---

## 5. DB 자동 백업 권장 구조

```bash
/home/ec2-user/backup/full_backup.sh
```

예시 스크립트:

```bash
#!/bin/bash
DATE=$(date +%F)
BACKUP_DIR="/home/ec2-user/backup"
mysqldump -u root --all-databases > "$BACKUP_DIR/db_backup_$DATE.sql"
find "$BACKUP_DIR" -type f -mtime +14 -delete
```

크론 등록:

```
0 15 * * 1,4 /home/ec2-user/backup/full_backup.sh
```

---

## 6. 추가 권장 설정

* `/etc/my.cnf.d/` 내부에 `charset=utf8mb4` 지정
* 사용자별 최소 권한 원칙 적용 (`GRANT SELECT, INSERT, UPDATE` 등 제한)
* 데이터 무결성 점검 주기적 수행 (`CHECK TABLE`, `ANALYZE TABLE`)

```
