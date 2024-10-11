---
layout: post
title: "PotsgreSQL 주요명령어"
date: 2024-10-11 10:08:00 +09:00
categories: Database
tags: Linux Database Postgresql Commands
---

# PostgreSQL 주요 명령어 정리

PostgreSQL을 자주 사용하는 게 아니라 가끔 필요할 때마다 설정하다 보니 명령어들을 자꾸 잊어버리게 된다. 그래서 미래의 내가 또 기억이 안 나면 참고할 수 있도록 주요 명령어들을 정리해 둔다.

## 1. PostgreSQL 접속

PostgreSQL 명령어를 실행하려면 먼저 데이터베이스에 접속해야 한다.

```
sudo -u postgres psql
```

`postgres`는 기본 관리 사용자 계정이다. 이 명령어로 접속한 후 SQL 명령어를 실행할 수 있다.

## 2. 데이터베이스 생성

새로운 데이터베이스를 생성하려면:

```
CREATE DATABASE database_name;
```

`database_name`을 원하는 이름으로 변경해서 사용한다.

## 3. 사용자 생성

데이터베이스 접근 권한을 부여하기 위한 사용자 생성 방법:

```
CREATE USER user_name WITH PASSWORD 'password';
```

`user_name`과 `'password'`는 원하는 사용자명과 비밀번호로 설정한다.

## 4. 데이터베이스에 사용자 권한 부여

생성한 사용자에게 데이터베이스 접근 권한을 부여하려면:

```
GRANT ALL PRIVILEGES ON DATABASE database_name TO user_name;
```

특정 권한만 부여하려면 `ALL PRIVILEGES` 대신 `SELECT`, `INSERT`, `UPDATE`와 같은 권한을 명시할 수 있다.

## 5. 데이터베이스 목록 확인

현재 서버에 있는 데이터베이스 목록을 확인하려면:

```
\l
```

## 6. 사용자 목록 확인

PostgreSQL 사용자 목록을 확인하려면:

```
\du
```

## 7. 특정 데이터베이스에 연결

다른 데이터베이스로 전환하려면:

```
\c database_name
```

`database_name`을 연결하고자 하는 데이터베이스 이름으로 변경한다.

## 8. 테이블 목록 보기

현재 연결된 데이터베이스의 테이블 목록을 확인하려면:

```
\dt
```

## 9. 데이터베이스 삭제

필요 없는 데이터베이스를 삭제하려면:

```
DROP DATABASE database_name;
```

주의: 데이터베이스를 삭제하면 그 안의 데이터도 모두 사라진다.

## 10. 사용자 삭제

사용자를 삭제하려면:

```
DROP USER user_name;
```

## 11. 특정 테이블에 대한 권한 부여

사용자에게 특정 테이블에 대한 권한을 부여하려면:

```
GRANT SELECT, INSERT, UPDATE, DELETE ON table_name TO user_name;
```

`table_name`은 권한을 설정할 테이블 이름이다.

## 12. 슈퍼유저 권한 부여

특정 사용자에게 슈퍼유저 권한을 부여하려면:

```
ALTER USER user_name WITH SUPERUSER;
```

## 13. 사용자 비밀번호 변경

사용자의 비밀번호를 변경하려면:

```
ALTER USER user_name WITH PASSWORD 'new_password';
```

## 14. PostgreSQL 서비스 제어

PostgreSQL 서비스를 제어하는 명령어:

```
# 시작
sudo systemctl start postgresql

# 중지
sudo systemctl stop postgresql

# 재시작
sudo systemctl restart postgresql
```

## 15. 데이터베이스 백업 및 복원

- **백업**: `pg_dump` 명령어를 사용하여 데이터베이스를 백업할 수 있다.

  ```
  pg_dump -U user_name -W -F t database_name > backup_file.tar
  ```

- **복원**: `pg_restore` 명령어를 사용하여 백업 파일을 복원할 수 있다.

  ```
  pg_restore -U user_name -W -d database_name backup_file.tar
  ```

이 주요 명령어들을 기억해두면 PostgreSQL 설정과 관리가 훨씬 수월해질 것이다. 필요할 때마다 이 목록을 참고해서 빠르게 작업을 처리하자.