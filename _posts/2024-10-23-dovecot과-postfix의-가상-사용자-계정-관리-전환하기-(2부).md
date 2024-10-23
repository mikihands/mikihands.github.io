---
layout: post
title: "Dovecot과 Postfix의 가상 사용자 계정 관리 전환하기 (2부)"
date: 2024-10-23 11:05:00 +09:00
categories: Linux
tags: Mail-server, Postfix, Dovecot
---

### Dovecot과 Postfix의 가상 사용자 계정 관리 전환하기 (2부)

---

앞서 1부에서는 **Dovecot과 Postfix의 역할** 및 **이메일 계정 관리 방식**에 대해 설명했다. 이번 글에서는 시스템 사용자 계정 기반에서 **가상 사용자 기반**으로 전환하는 방법을 단계별로 설명한다. 가상 사용자 계정 관리를 통해 더 많은 사용자와 유연한 메일 서버 관리를 할 수 있다.

---

### 1. PostgreSQL 데이터베이스 설정

가상 사용자 정보를 저장할 데이터베이스로 **PostgreSQL**을 사용할 수 있다. 먼저, PostgreSQL을 설치하고, 가상 사용자 데이터를 관리할 데이터베이스와 테이블을 설정해야 한다.

#### PostgreSQL 설치

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

#### 가상 사용자용 데이터베이스 및 테이블 생성

```sql
CREATE DATABASE mailserver;
CREATE USER mailuser WITH PASSWORD 'yourpassword';
GRANT ALL PRIVILEGES ON DATABASE mailserver TO mailuser;

\c mailserver;

CREATE TABLE virtual_domains (
  id SERIAL PRIMARY KEY,
  domain VARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE virtual_users (
  id SERIAL PRIMARY KEY,
  domain_id INTEGER REFERENCES virtual_domains(id) ON DELETE CASCADE,
  email VARCHAR(255) NOT NULL UNIQUE,
  password VARCHAR(255) NOT NULL
);

CREATE TABLE virtual_aliases (
  id SERIAL PRIMARY KEY,
  source VARCHAR(255) NOT NULL,
  destination VARCHAR(255) NOT NULL
);
```

위 명령어를 통해 가상 도메인, 사용자, 이메일 별칭을 저장할 테이블을 생성한다.

---

### 2. Postfix 설정 변경

Postfix가 가상 사용자 데이터를 데이터베이스에서 확인하고 메일을 처리하도록 설정을 변경해야 한다. 이를 위해 **Postfix의 설정 파일**을 수정하고, 데이터베이스와의 연동을 위한 추가 설정을 해야 한다.

#### `/etc/postfix/main.cf` 파일 수정

```bash
virtual_mailbox_domains = proxy:pgsql:/etc/postfix/pgsql-virtual-domains.cf
virtual_mailbox_maps = proxy:pgsql:/etc/postfix/pgsql-virtual-users.cf
virtual_alias_maps = proxy:pgsql:/etc/postfix/pgsql-virtual-aliases.cf
```

#### `pgsql` 설정 파일 생성

다음은 PostgreSQL과 연동할 설정 파일들이다.

- **pgsql-virtual-domains.cf**

  ```bash
  user = mailuser
  password = yourpassword
  hosts = 127.0.0.1
  dbname = mailserver
  query = SELECT domain FROM virtual_domains WHERE domain='%s';
  ```

- **pgsql-virtual-users.cf**

  ```bash
  user = mailuser
  password = yourpassword
  hosts = 127.0.0.1
  dbname = mailserver
  query = SELECT email FROM virtual_users WHERE email='%s';
  ```

- **pgsql-virtual-aliases.cf**

  ```bash
  user = mailuser
  password = yourpassword
  hosts = 127.0.0.1
  dbname = mailserver
  query = SELECT destination FROM virtual_aliases WHERE source='%s';
  ```

Postfix는 위 설정 파일들을 참조하여 가상 사용자 정보를 가져온다.

---

### 3. Dovecot 설정 변경

Dovecot이 가상 사용자 정보를 데이터베이스에서 가져오도록 설정을 변경해야 한다.

#### `/etc/dovecot/dovecot-sql.conf.ext` 파일 수정

```bash
driver = pgsql
connect = host=127.0.0.1 dbname=mailserver user=mailuser password=yourpassword

user_query = SELECT '/var/mail/%d/%n' as home, 'maildir:/var/mail/%d/%n' as mail, 5000 AS uid, 5000 AS gid FROM virtual_users WHERE email='%u';

password_query = SELECT email as user, password FROM virtual_users WHERE email='%u';
```

#### `/etc/dovecot/conf.d/10-auth.conf` 파일 수정

```bash
disable_plaintext_auth = yes
auth_mechanisms = plain login

passdb {
  driver = sql
  args = /etc/dovecot/dovecot-sql.conf.ext
}

userdb {
  driver = sql
  args = /etc/dovecot/dovecot-sql.conf.ext
}
```

#### Dovecot의 메일 저장소 설정

Dovecot의 `mail_location` 설정을 가상 사용자에 맞게 수정한다.

```bash
mail_location = maildir:/var/mail/vmail/%d/%n
```

---

### 4. 기존 메일 데이터 백업 및 마이그레이션

현재 시스템 사용자 계정에 저장된 이메일 데이터를 새로운 가상 사용자 메일 디렉토리로 옮기는 절차가 필요하다. 이를 위해 `rsync` 명령어를 사용하여 데이터를 백업하고 마이그레이션할 수 있다.

```bash
rsync -av /home/olduser/Maildir/ /var/mail/vmail/example.com/virtualuser/
```

이 명령어를 통해 기존 데이터를 손실 없이 옮길 수 있다.

---

### 5. Postfix 및 Dovecot 재시작

설정 변경이 완료되면 Postfix와 Dovecot을 재시작하여 변경 사항을 적용한다.

```bash
sudo systemctl restart postfix
sudo systemctl restart dovecot
```

---

### 6. 테스트 및 검증

이제 가상 사용자 계정 관리로 전환되었는지 확인하기 위해 이메일 송수신 테스트를 진행한다. Postfix와 Dovecot의 로그를 확인하여 설정이 올바르게 적용되었는지 검토하고, 실제로 이메일이 정상적으로 전송 및 수신되는지 확인해야 한다.
