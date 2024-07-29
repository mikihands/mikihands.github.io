---
layout: post
title:  "Let's Encrypt SSL 인증서 수동 갱신 방법"
date:   2024-07-23 17:42:00 +0900
categories: Linux
tags: Linux SSL
---


## Let's Encrypt SSL 인증서 수동 갱신 방법

### 소개

SSL 인증서는 웹사이트의 보안을 위해 필수적입니다. Let's Encrypt는 무료로 SSL 인증서를 제공하며, 이 글에서는 Let's Encrypt SSL 인증서를 수동으로 갱신하는 방법을 자세히 설명합니다. 이 가이드는 도메인을 Squarespace에서 구매한 경우를 기준으로 합니다.

### 요구 사항

- SSH 접근 권한이 있는 서버
- Certbot 설치
- Squarespace 계정

### 1단계: Certbot 설치

서버에 Certbot이 설치되어 있어야 합니다. 설치되지 않은 경우 다음 명령어를 사용하여 설치합니다.

**Ubuntu/Debian:**

```bash
sudo apt-get update
sudo apt-get install certbot
```

### 2단계: Certbot 명령어 실행

Certbot을 수동 모드로 실행하여 DNS-01 챌린지를 수행합니다. 이 명령어를 실행하면 Certbot이 필요한 정보를 제공합니다.

```
bash
Copy code
sudo certbot certonly --manual --preferred-challenges dns -d mail.yourdomain.com
```

명령어를 실행하면 다음과 같은 출력이 나타납니다.

```plaintext
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Renewing an existing certificate for mail.yourdomain.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name:

_acme-challenge.mail.yourdomain.com.

with the following value:

KQYI-value_for_yourdomain_encrypte_ssl

Before continuing, verify the TXT record has been deployed. Depending on the DNS
provider, this may take some time, from a few seconds to multiple minutes. You can
check if it has finished deploying with aid of online tools, such as the Google
Admin Toolbox: https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.mail.yourdomain.com.
Look for one or more bolded line(s) below the line ';ANSWER'. It should show the
value(s) you've just added.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

### 3단계: Squarespace에서 DNS 설정 수정

Squarespace 계정에 로그인하여 DNS 설정을 수정합니다.

1. **Squarespace 로그인**: 웹 브라우저를 열고 Squarespace에 로그인합니다.

2. **도메인 설정으로 이동**: 상단 메뉴에서 "Settings" (설정)를 클릭한 후, "Domains" (도메인)을 클릭합니다.

3. **DNS 설정 열기**: 갱신할 도메인을 선택하고 "DNS Settings" (DNS 설정)를 클릭합니다.

4. TXT 레코드 추가 또는 수정

   : 기존 

   ```
   _acme-challenge.mail
   ```

    레코드가 있는 경우 값을 업데이트하고, 없는 경우 새로 추가합니다.

   - **Type**: `TXT`
   - **Name**: `_acme-challenge.mail`
   - **Value**: Certbot이 제공한 값 (`KQYI-value_for_yourdomain_encrypte_ssl`)





변경사항을 저장하고 DNS 레코드가 전파될 때까지 기다립니다. 이는 최대 몇 분에서 몇 시간까지 걸릴 수 있습니다.

### 4단계: DNS 레코드 전파 확인

DNS 레코드가 올바르게 전파되었는지 확인합니다. 이를 위해 [Google Admin Toolbox](https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.mail.yourdomain.com)와 같은 도구를 사용할 수 있습니다.

```plaintext
; <<>> DiG 9.10.6 <<>> TXT _acme-challenge.mail.yourdomain.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64089
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;_acme-challenge.mail.yourdomain.com. IN    TXT

;; ANSWER SECTION:
_acme-challenge.mail.yourdomain.com. 299 IN TXT    "KQYI-value_for_yourdomain_encrypte_ssl"

;; Query time: 23 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Jul 29 12:34:56 PDT 2024
;; MSG SIZE  rcvd: 132
```

### 5단계: Certbot에서 Enter 키 누르기

DNS 레코드가 올바르게 전파되었음을 확인한 후, 터미널로 돌아가서 Enter 키를 눌러 Certbot이 계속 진행할 수 있도록 합니다.

```plaintext
Press Enter to Continue
```

Certbot이 DNS 레코드를 확인하고 인증서를 갱신합니다. 성공 메시지가 나타나면 갱신이 완료된 것입니다.

### 6단계: Postfix 및 Dovecot 서비스 재시작

Postfix와 Dovecot이 갱신된 인증서를 사용하도록 서비스를 재시작합니다.

```bash
sudo systemctl restart postfix
sudo systemctl restart dovecot
```

### 결론

이제 Let's Encrypt SSL 인증서를 수동으로 갱신하는 방법을 배웠습니다. 이 과정을 통해 도메인 보안을 유지할 수 있습니다. 수동 갱신이 번거로울 수 있지만, 필요할 때마다 이 가이드를 따라 쉽게 갱신할 수 있습니다.

### 자동 갱신에 대해

자동 갱신을 설정하려면 DNS 제공자의 API를 사용해야 합니다. Squarespace는 API를 제공하지 않으므로 수동 갱신을 계속해야 합니다. 갱신 알림을 받아 적절한 시기에 갱신을 수행하는 것이 중요합니다.