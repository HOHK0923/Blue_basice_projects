
# 방화벽 및 포트 설정 매뉴얼

## 1. firewalld 서비스 개요
서버는 `firewalld`를 통해 인바운드·아웃바운드 포트를 관리합니다.  
현재 활성화된 존은 **public**이며, 웹 서비스 및 SSH 접속을 허용하도록 설정되어 있습니다.

### 서비스 활성 상태
```bash
sudo firewall-cmd --list-all
````

출력 예시:

```
public
  services: http https ssh mdns dhcpv6-client
  ports: 8000/tcp
  rich rules:
        rule family="ipv4" source address="210.106.232.151" reject
        rule family="ipv4" source address="210.106.232.188" reject
        rule family="ipv4" source address="104.244.72.115" reject
```

---

## 2. 주요 포트 개방 정보

| 포트   | 프로토콜 | 용도                | 서비스            |
| ---- | ---- | ----------------- | -------------- |
| 80   | TCP  | HTTP              | Apache (httpd) |
| 443  | TCP  | HTTPS (예정)        | Apache (httpd) |
| 8000 | TCP  | 내부 웹 UI           | Splunk         |
| 8089 | TCP  | Splunk Management | Splunkd        |
| 22   | TCP  | SSH 접속            | ec2-user 원격 접근 |

---

## 3. 설정 명령어 모음

### 서비스 추가

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh
```

### 포트 추가

```bash
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload
```

### 특정 IP 차단

```bash
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="210.106.232.151" reject'
sudo firewall-cmd --reload
```

---

## 4. 현재 리스닝 상태 확인

```bash
sudo ss -tulpn | grep -E '(:80|:443|:8000|:8080|:8089)'
```

출력 예시:

```
tcp LISTEN 0 511 *:80 *:* users:(("httpd",pid=33016,fd=4))
tcp LISTEN 0 128 0.0.0.0:8000 0.0.0.0:* users:(("splunkd",pid=2394,fd=189))
tcp LISTEN 0 128 0.0.0.0:8089 0.0.0.0:* users:(("splunkd",pid=2394,fd=4))
```

---

## 5. 관리 권장 사항

* 변경 후 반드시 `sudo firewall-cmd --reload` 수행
* `firewalld` 상태 확인: `sudo systemctl status firewalld`
* 불필요한 포트는 즉시 제거 (`--remove-port` 옵션 사용)
* 외부 접근 제한 시 CIDR 기반 접근제어 병행 (`--add-rich-rule`)

```


