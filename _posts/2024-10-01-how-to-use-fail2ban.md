---
layout: post
title:  "Fail2ban을 이용한 postfix 메일서버 보안강화"
date:   2024-10-01 19:00:00 +0900
categories: Linux
tags: Linux Postfix Security
published : false
---

### Fail2Ban을 이용한 Postfix 메일 서버 보안 강화

**2024-10-01**

오늘은 **Postfix 메일 서버**의 보안을 강화하기 위해 **Fail2Ban**을 설치하고 설정하는 작업을 했다. 서버 로그를 살펴보니 매일같이 수많은 공격이 들어오고 있었고, 이를 자동으로 차단하기 위해 Fail2Ban을 활용하기로 결정했다.

#### 1. **Fail2Ban 설치**

먼저 Fail2Ban을 설치했다. Fail2Ban은 로그 파일을 감시하고, 특정 패턴에 맞는 공격을 감지해 IP를 차단하는 도구다.

```bash
sudo apt-get install fail2ban
```

#### 2. **VM 방화벽 연동 스크립트 작성**

기본적으로 Fail2Ban은 **로컬 방화벽(ufw나 iptables)**을 사용하지만, 나는 VM 자체의 외부 방화벽을 사용 중이라 Fail2Ban이 감지한 IP를 **GCP 방화벽 규칙(block_ip)**에 추가하는 스크립트를 작성했다.

**`/etc/fail2ban/action.d/gcloud-ban.conf`** 파일을 생성하고 아래 내용을 작성했다:

```bash
[Definition]
actionban = /usr/bin/gcloud compute firewall-rules update block-ip --add-source-ranges %(banip)s
actionunban = /usr/bin/gcloud compute firewall-rules update block-ip --remove-source-ranges %(banip)s
```

- **`%(banip)s`**: Fail2Ban이 감지한 차단 대상 IP가 자동으로 전달되는 변수.
- **`actionban`**: 감지된 IP를 GCP 방화벽 규칙에 추가.
- **`actionunban`**: 차단 해제 시 해당 IP를 GCP 방화벽에서 제거.

#### 3. **Fail2Ban 설정 파일 수정**

이제 Fail2Ban이 Postfix와 Dovecot 로그 파일을 감시하고, 감지된 IP를 GCP 방화벽에 추가하도록 설정을 진행했다.

**`/etc/fail2ban/jail.local`** 파일을 수정해 아래와 같이 Postfix와 Dovecot에 대한 설정을 추가했다:

```ini
[postfix]
enabled  = true
port     = smtp,ssmtp
filter   = postfix
logpath  = /var/log/mail.log
maxretry = 3
action   = gcloud-ban

[dovecot]
enabled  = true
port     = pop3,pop3s,imap,imaps
filter   = dovecot
logpath  = /var/log/mail.log
maxretry = 3
action   = gcloud-ban
```

#### 4. **필터 파일 확인**

Fail2Ban은 로그 파일에서 **정규식 패턴**을 이용해 공격 시도를 감지한다. 예를 들어, Postfix의 **SASL 로그인 실패**와 같은 패턴을 감지하도록 필터가 설정되어 있다.

**`/etc/fail2ban/filter.d/postfix.conf`** 파일에서 감지할 패턴이 어떻게 정의되어 있는지 확인해보았다.

```bash
cat /etc/fail2ban/filter.d/postfix.conf
```

여기서 **`mdre-auth`** 정규식을 통해 **SASL 인증 실패**가 감지되면 IP가 차단되도록 설정되어 있었다.

#### 5. **Fail2Ban 서비스 재시작**

모든 설정을 마친 후 Fail2Ban 서비스를 재시작했다.

```bash
sudo systemctl restart fail2ban
```

#### 6. **작동 확인 방법**

1. **Fail2Ban 로그 파일 확인**: Fail2Ban이 잘 작동하는지 확인하려면 **로그 파일**을 모니터링할 수 있다.

   ```bash
   sudo tail -f /var/log/fail2ban.log
   ```

   여기서 Fail2Ban이 감지한 IP가 차단되는지 확인할 수 있다.

2. **GCP 방화벽 규칙 확인**: Fail2Ban이 감지한 IP가 **GCP 방화벽 규칙(block-ip)**에 추가되었는지 확인하기 위해 GCP CLI를 사용했다.

   ```bash
   gcloud compute firewall-rules describe block-ip
   ```

   이 명령어로 차단된 IP 목록을 확인할 수 있다.

#### 7. **필터 패턴 추가**

비정상적인 연결 시도나, 명령어 없이 연결이 끊어지는 패턴도 차단하기 위해 **필터 패턴을 추가**했다. 아래는 비정상적인 연결 끊김 패턴을 감지하는 정규식이다.

**`/etc/fail2ban/filter.d/postfix.conf`**에 추가:

```ini
failregex = postfix/smtpd.*: lost connection after UNKNOWN from \S+\[\S+\]
```

이 패턴을 통해 **연결이 끊어진 비정상적인 연결 시도**도 차단할 수 있게 되었다.

------

### 마무리

이번 작업을 통해 **메일 서버의 보안**을 한층 더 강화할 수 있었다. 특히 매일 들어오는 공격을 **Fail2Ban이 자동으로 감지하고 GCP 방화벽에 차단**하는 구조로 설정한 덕분에, 불필요한 접근 시도를 자동으로 차단할 수 있게 되었다.

이제 서버 로그를 주기적으로 모니터링하면서 추가적인 공격 패턴이 발견되면 **필터를 추가**해 대응할 계획이다. Fail2Ban은 앞으로도 중요한 보안 도구로 자리 잡을 것 같다.