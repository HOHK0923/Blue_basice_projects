
# healthmash.net 접속 불가 문제 분석 및 해결 로그

## 1) 문제 요약
- 증상: 외부에서 `http://healthmash.net` 접속 시 `ERR_CONNECTION_TIMED_OUT`
- 내부 점검 결과:
  - DNS: 정상 (A 레코드 → `15.164.94.241`)
  - 인스턴스: Apache 작동, 내부 `curl -I http://localhost` 결과 정상(HTTP 302 Found)
  - 보안 그룹: 22, 80, 443, 8000 포트 모두 허용
  - 서브넷: 퍼블릭 (IGW 연결 확인)
  - 문제 원인: OS 방화벽(`firewalld`)에서 80/443 포트 미허용

---

## 2) 점검 과정 요약

### (1) 내부 웹서버 상태 확인
```bash
sudo systemctl status httpd
curl -I http://localhost
````

출력:

```
HTTP/1.1 302 Found
Server: Apache/2.4.65 (Amazon Linux)
Location: login.php
```

→ Apache 작동 및 PHP 세션 정상

---

### (2) 보안 그룹 확인

* Inbound: 22, 80, 443, 8000 (모두 0.0.0.0/0 허용)
* Outbound: All traffic 허용
  ✅ 정상

---

### (3) 서브넷 라우팅 확인

* Route Table:

  ```
  172.31.0.0/16 → local
  0.0.0.0/0     → igw-0077e17a3a4277aa1
  ```

  ✅ IGW 연결 → 퍼블릭 서브넷 정상

---

### (4) OS 방화벽 점검

```bash
sudo firewall-cmd --list-ports
```

출력:

```
8000/tcp
```

→ 80, 443 미허용 ❌

---

## 3) 해결 조치

### (1) 방화벽 포트 추가

```bash
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --add-port=443/tcp --permanent
sudo systemctl restart firewalld
sudo firewall-cmd --list-ports
```

출력:

```
80/tcp 443/tcp 8000/tcp
```

✅ 포트 정상 추가 완료

---

### (2) 접속 확인

```bash
curl -I http://15.164.94.241
```

결과:

```
HTTP/1.1 302 Found
Location: login.php
```

→ Apache 응답 정상

DNS 전파 후 외부에서도 `http://healthmash.net` 접속 정상 확인.

---

## 4) 선택적 구성 (비용 절감 방침 반영)

* Elastic IP(EIP)는 사용하지 않음 (비용 절약 목적)
* IPv4가 재할당될 때마다 수동으로 Route 53 A레코드 갱신
* A레코드 TTL을 300초(5분)로 설정하여 빠른 갱신 반영

---

## 5) 결론

| 항목        | 결과                   |
| --------- | -------------------- |
| DNS 구성    | 정상                   |
| 서브넷 라우팅   | IGW 연결 정상            |
| 보안 그룹     | 22, 80, 443, 8000 허용 |
| Apache 상태 | 정상 (302 Found)       |
| OS 방화벽    | 🔥 수정 완료 (80/443 허용) |
| 접속 상태     | ✅ 정상화 완료             |

---

## 6) 참고 명령어 모음

```bash
# 실시간 포트 상태 확인
sudo ss -tulpn | grep httpd

# DNS 테스트
nslookup healthmash.net
dig A healthmash.net +short

# 웹 포트 접근 확인 (외부 PC)
telnet healthmash.net 80
```

```
