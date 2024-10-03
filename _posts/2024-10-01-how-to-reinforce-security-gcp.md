GCP 서버에서의 브루트포스 공격 및 해킹 시도를 방어하기 위해 여러 가지 조치를 취할 수 있습니다. 서버 보안은 다층적인 접근이 필요하며, 아래와 같은 방법들을 통해 보안을 강화할 수 있습니다.

### 1. **GCP 방화벽 규칙 강화**

#### 1.1 SSH 포트제한

- **SSH 포트 제한**: 기본적으로 SSH 포트(22)를 사용 중이라면 이를 비표준 포트로 변경하거나, 방화벽에서 SSH 접근을 허용하는 IP 범위를 제한하세요. 고정 IP가 없다면, DDNS와 같은 동적 DNS 서비스를 사용하여 특정 도메인에 대해서만 접근을 허용할 수 있습니다.

- CP의 방화벽 규칙에서는 **직접적으로 도메인(DNS 이름)을 입력하여 접근을 허용하는 방식**을 지원하지 않습니다. 방화벽 규칙은 오직 IP 주소나 IP 범위를 기반으로 작동하며, 도메인 이름을 입력하는 옵션이 없습니다. 그러나 동적 IP 환경에서 DDNS를 활용하여 접근을 허용하려면 **다음과 같은 대안적인 방법**을 고려할 수 있습니다.

##### 1.1.1. 동적 DNS 스크립트를 사용한 방화벽 자동 업데이트

동적 IP를 사용하는 경우, DDNS 서비스를 통해 제공받는 도메인의 IP 주소가 변경될 때마다 해당 IP를 GCP 방화벽 규칙에 자동으로 반영하는 스크립트를 작성할 수 있습니다. 아래는 기본적인 아이디어입니다.

1. **현재 DDNS 도메인의 IP 주소 확인**: `dig` 명령어 등을 사용하여 DDNS 도메인의 IP를 확인합니다.

   ```bash
   dig +short blog.mikihands.com
   ```

2. **GCP 방화벽 규칙 업데이트**: GCP CLI 또는 API를 통해 해당 IP를 방화벽 규칙에 반영하는 스크립트를 작성합니다.

   - 방화벽 규칙 업데이트 명령어:

   ```bash
   gcloud compute firewall-rules update <FIREWALL_RULE_NAME> \
   --allow tcp:22 \
   --source-ranges=<CURRENT_IP>
   ```

3. **주기적인 IP 변경 감지**: 위 스크립트를 크론 작업(cron job)으로 설정하여 정기적으로 DDNS 도메인의 IP 변화를 감지하고, 변경 시 자동으로 GCP 방화벽을 업데이트하도록 합니다.

예시 스크립트:

```bash
#!/bin/bash
DDNS_DOMAIN="blog.mikihands.com"
FIREWALL_RULE_NAME="allow-ssh-from-ddns"

# 현재 DDNS 도메인의 IP 확인
CURRENT_IP=$(dig +short $DDNS_DOMAIN)

# GCP 방화벽 규칙에 업데이트
if [ -n "$CURRENT_IP" ]; then
  gcloud compute firewall-rules update $FIREWALL_RULE_NAME \
  --allow tcp:22 \
  --source-ranges=$CURRENT_IP
fi
```

이 스크립트를 주기적으로 실행하여 IP가 변경되면 GCP 방화벽 규칙을 자동으로 업데이트할 수 있습니다.


#### 1.2 IP 화이트리스트
서버 접근을 허용할 IP를 최소화하세요. 0.0.0.0/0으로 설정된 방화벽 규칙을 특정 신뢰 IP나 지역으로 한정하는 것이 좋습니다. 특히 포트 5432(PostgreSQL 접근)도 IP 범위를 제한하는 것이 중요합니다.

### 2. **SSH 보안 강화**

- SSH 키 인증 사용

  : 패스워드 인증을 비활성화하고 SSH 키 인증을 사용하여 보안을 강화하세요.

  ```bash
  # /etc/ssh/sshd_config 수정
  PasswordAuthentication no
  ```

- Fail2Ban 설치

  : SSH 로그인 시도 실패를 감지하여 특정 IP를 자동으로 차단하는 Fail2Ban과 같은 도구를 사용할 수 있습니다.

  ```bash
  sudo apt install fail2ban
  ```

  기본적으로 SSH 로그인 시도 횟수가 일정 수치를 넘으면 해당 IP를 일정 시간 동안 차단하게 설정할 수 있습니다.

### 3. **2단계 인증(2FA) 도입**

- SSH 접근에 Google Authenticator와 같은 2단계 인증을 추가하면 보안이 한층 강화됩니다.

  ```bash
  sudo apt install libpam-google-authenticator
  ```

  이와 함께 Google Authenticator를 서버와 연동하여 SSH 로그인 시 OTP를 요구할 수 있습니다.

### 4. **포트 스캔 탐지 및 차단**

- 포트 스캔 방지

  : 포트 스캔 탐지를 위해 

  ```
  psad
  ```

  와 같은 포트 스캐닝 감지 도구를 사용해, 탐지된 공격 IP를 자동 차단할 수 있습니다.

  ```bash
  sudo apt install psad
  ```

### 5. **자동화된 모니터링 및 알림 시스템 설정**

- **ClamAV**와 같은 도구를 사용하여 서버의 파일 시스템을 주기적으로 검사하고 의심스러운 파일을 감지하세요.
- **로그 모니터링**: GCP의 Stackdriver 또는 `logwatch`를 사용하여 의심스러운 로그 이벤트를 실시간으로 모니터링하고, 알림을 받을 수 있도록 설정하세요.

### 6. **Failover 및 백업 시스템 구축**

- 해킹이 발생해도 빠르게 복구할 수 있도록 주기적인 백업을 설정하세요. GCP의 스냅샷 기능을 이용하여 시스템을 복구할 수 있는 상태로 유지하는 것이 좋습니다.

### 7. **웹 애플리케이션 방화벽(WAF) 도입**

- Django나 Nginx 서버 앞에 WAF(Web Application Firewall)를 추가하면 SQL 인젝션, XSS와 같은 웹 애플리케이션 공격을 막을 수 있습니다. GCP의 Cloud Armor를 고려해 볼 수 있습니다.

### 8. **GCP Identity and Access Management (IAM) 설정 강화**

- 필요 최소한의 권한만 부여하여 IAM 정책을 설정하세요. 특정 서비스 계정이나 사용자에게 필요한 권한만 부여하고 불필요한 권한을 제거하세요.

이 방법들을 단계적으로 적용해 나가면 GCP 서버의 보안이 크게 강화될 것입니다.