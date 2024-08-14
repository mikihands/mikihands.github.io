---
layout: post
title:  "Linux 시스템의 파일소유권과 권한 #2"
date:   2024-07-24 01:00:00 +0900
categories: Linux
tags: Linux 명령어 User UserGroup
---


## 리눅스 시스템에서 USER 와  USER GROUP

### 사용자와 사용자 그룹의 개념

**사용자 (User):**

- 사용자란 시스템에 접근할 수 있는 개별 계정을 의미합니다. 각 사용자는 **고유한 사용자 ID (UID)**를 가지고 있습니다.
- 사용자는 시스템 리소스에 접근하거나 프로그램을 실행할 수 있습니다.
- 사용자 계정은 개인 사용자, 서비스 계정 또는 시스템 계정일 수 있습니다.

**사용자 그룹 (User Group):**

- 사용자 그룹은 여러 사용자 계정을 묶어 관리하는 단위입니다. 각 그룹은 고유한 그룹 ID (GID)를 가지고 있습니다.
- 그룹을 사용하면 여러 사용자에게 동일한 권한을 쉽게 부여할 수 있습니다.
- 파일 및 디렉토리의 접근 권한을 그룹 단위로 설정할 수 있습니다.

### 사용자와 그룹의 활용

1. **보안 및 권한 관리:**
   - 특정 사용자에게만 특정 파일이나 디렉토리에 접근 권한을 줄 수 있습니다.
   - 그룹을 사용하면 동일한 권한을 여러 사용자에게 효율적으로 부여할 수 있습니다.
2. **협업 환경:**
   - 여러 사용자가 동일한 프로젝트나 리소스에 접근해야 할 때, 해당 사용자들을 그룹에 추가하면 쉽게 권한을 관리할 수 있습니다.
3. **시스템 관리:**
   - 서비스 계정 및 시스템 계정을 분리하여 시스템 리소스 및 서비스를 안전하게 관리할 수 있습니다.

### 관련 bash 명령어

1. **사용자 관련 명령어:**

   - 사용자 추가: `sudo adduser <username>`

   - 사용자 삭제: `sudo deluser <username>`

   - 사용자 정보 수정: 

     ```bash
     sudo usermod [옵션] <username>
     ```

     - 예: 사용자를 그룹에 추가: `sudo usermod -aG <groupname> <username>`

   - 사용자 정보 확인: `id <username>`

2. **그룹 관련 명령어:**

   - 그룹 추가: `sudo addgroup <groupname>`
   - 그룹 삭제: `sudo delgroup <groupname>`
   - 그룹에 사용자 추가: `sudo usermod -aG <groupname> <username>`
   - 그룹에 사용자 제거: `sudo deluser <username> <groupname>`

3. **파일 및 디렉토리 권한 관련 명령어:**

   - 소유자 변경: 

     ```bash
     sudo chown <owner>:<group> <file/directory>
     ```

     - 예: `sudo chown jessekim:jessekim /path/to/file`

   - 권한 변경: 

     ```bash
     chmod [옵션] <file/directory>
     ```

     - 예: 읽기, 쓰기, 실행 권한 부여: `chmod 770 /path/to/directory`

   - 소유자 및 권한 확인: `ls -l <file/directory>`

     
4. **기타 유용한 명령어:**

   - 현재 로그인된 사용자 확인: `whoami`

   - 시스템에 존재하는 모든 사용자 확인: `cat /etc/passwd`

   - 시스템에 존재하는 모든 그룹 확인: `cat /etc/group`

     - `www-data:x:33:jessekim` 의미 : 

       1. **`www-data`**: 그룹의 이름입니다. 이 경우 `www-data` 그룹입니다. 일반적으로 웹 서버 소프트웨어(Apache, Nginx 등)가 사용하는 그룹입니다.
       2. **`x`**: 패스워드 필드입니다. 예전에는 그룹 패스워드가 저장되었지만, 현재는 대부분의 시스템에서 이 기능을 사용하지 않으며, `x`로 표시됩니다. 실제 패스워드는 `/etc/gshadow` 파일에 저장될 수 있습니다.
       3. **`33`**: 그룹 ID(GID)입니다. 이 그룹에 할당된 고유 숫자 ID입니다. 시스템에서는 이 숫자를 사용하여 그룹을 식별합니다.
       4. **`jessekim80`**: 그룹에 속한 사용자입니다. 이 경우 `jessekim` 사용자가 `www-data` 그룹에 속해 있음을 의미합니다. 콤마(,)로 구분하여 여러 사용자가 나열될 수 있습니다.

       정리하면, `www-data:x:33:jessekim`의 의미는 다음과 같습니다:

       - `www-data`라는 이름의 그룹이 있습니다.
       - 이 그룹의 GID는 `33`입니다.
       - `jessekim` 사용자가 이 그룹에 속해 있습니다.

### 사용 예시

1. **사용자 추가 및 그룹 설정:**

   ```bash
   sudo adduser jessekim
   sudo addgroup mk_project
   sudo usermod -aG mk_project jessekim80
   ```

2. **파일 및 디렉토리 권한 설정:**

   ```bash
   sudo chown jessekim:mk_project /mnt/disks/newdisk/mk_project/django/media
   sudo chmod 770 /mnt/disks/newdisk/mk_project/django/media
   ```

### 결론

사용자와 사용자 그룹을 나누는 것은 시스템 보안과 권한 관리를 효율적으로 하기 위해 필수적입니다. 이를 통해 시스템 자원을 보호하고, 여러 사용자가 협업하는 환경에서 효과적으로 권한을 부여할 수 있습니다. 위의 명령어들을 사용하여 우분투 시스템에서 사용자와 그룹을 효율적으로 관리할 수 있습니다.

### 추가 유용한 명령어

`grep`과 함께 사용하면 꽤 유용하다. 보통 Linux 시스템에 사용자인 내가 아니더라도 각 종 패키지 및 시스템에서 생성한  user 와 group 이 많기 때문에 원하는 정보를 한눈에 찾기 힘들다. 이때 `grep`은 정말 큰 도움이 된다. 

```bash
grep <찾고싶은유저명> /etc/group
grep <찾고싶은유저명> /etc/passwd
```
