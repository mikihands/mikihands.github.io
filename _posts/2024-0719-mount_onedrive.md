---
layout: post
title:  "Linux에 Onedrive 마운트 하기"
date:   2024-07-19 01:08:00 +0900
categories: Linux
tags: Linux Onedrive 마운트드라이브
---



# Linux에 Onedrive 마운트 하기



## 1. 설치

Ubuntu에서 Microsoft OneDrive를 시스템에 마운트하는 방법은 몇 가지가 있습니다. 그 중 하나는 `rclone`을 사용하는 방법입니다. `rclone`은 다양한 클라우드 저장소를 로컬 파일 시스템에 마운트할 수 있게 해주는 도구입니다. 아래에 rclone을 사용하여 OneDrive를 마운트하는 방법을 단계별로 설명하겠습니다.

1. **rclone 설치**

```bash
sudo apt update
sudo apt install rclone
```

1. **rclone 구성**

```sh
rclone config
```

`rclone config` 명령어를 실행하면 다음과 같은 메뉴가 나타납니다:

```
plaintextCopy codeNo remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q>
```

여기서 `n`을 입력하여 새로운 원격 저장소를 설정합니다. 그런 다음 아래의 프롬프트에 따라 설정을 진행합니다:

- **name> onedrive** 
- **client_id>** : 그냥 Enter 로 default
- **client_secret>** Enter 로 default
- Edit advanced config? (y/n) > No
- Use auto config? y
- Choose a number from below, or type in an existing value
  1 / OneDrive Personal or Business
- Onedrive로 브라우저가 열리며 인증하고 로그인하고 Success가 나올때까지 진행
- OAuth2.0 인증이 옴 : 인증 결과는 ~/.config/rclone/rclone.conf 파일에 저장됨



## 2. Mount 하기



### `systemd` 서비스로 자동 마운트 설정

다음은 `systemd` 서비스를 사용하여 시스템 부팅 시 OneDrive를 자동으로 마운트하는 방법입니다.

1. **서비스 파일 생성**: `/etc/systemd/system/onedrive.service` 파일을 생성하고 편집합니다.

   ```bash
   sudo nano /etc/systemd/system/onedrive.service
   ```

2. **서비스 파일 내용**: 아래 내용을 서비스 파일에 복사하여 붙여넣습니다. `your_username`을 실제 사용자 이름으로 바꾸세요.

   ```
   plaintextCopy code[Unit]
   Description=OneDrive Rclone Mount
   After=network-online.target
   
   [Service]
   Type=notify
   ExecStart=/usr/bin/rclone mount onedrive: /home/your_username/onedrive --vfs-cache-mode full --vfs-cache-max-age 24h
   ExecStop=/bin/fusermount -u /home/your_username/onedrive
   Restart=always
   User=your_username
   Group=your_username
   
   [Install]
   WantedBy=default.target
   ```

3. **서비스 파일 저장 및 종료**: 파일을 저장하고 나옵니다. (Nano 편집기에서 저장은 `Ctrl+O`, 종료는 `Ctrl+X`)

4. **`systemd` 데몬 재로드**: 새로 생성한 서비스를 인식시키기 위해 `systemd` 데몬을 재로드합니다.

   ```sh
   sudo systemctl daemon-reload
   ```

5. **서비스 활성화 및 시작**: 서비스를 활성화하고 시작합니다.

   ```bash
   sudo systemctl enable onedrive
   sudo systemctl start onedrive
   ```

### 서비스 상태 확인

서비스가 제대로 실행되고 있는지 확인하려면 다음 명령어를 사용합니다:

```sh
sudo systemctl status onedrive
```

이제 시스템을 재부팅하면 `systemd`가 자동으로 OneDrive를 마운트합니다.



### 수동으로 필요할 때만 Mount 하기 설정

`rclone mount remote:path /path/to/mountpoint [flags]` 명령어로 필요할 때만 수동 마운트를 할 수 있습니다. 터미널에 마운트가 활성화된 상태에서 다른 터미널을 열어서 onedrive의 디렉토리르 사용하거나, Ubuntu GUI 버젼인경우 Nautilus로 사용할수도 있습니다. 



 `[flags]`는 `rclone` 마운트 명령어의 다양한 옵션을 설정하는 플래그입니다. 이러한 플래그는 마운트 동작을 세밀하게 제어할 수 있게 해줍니다. 주요 플래그와 그 역할을 설명드리겠습니다.

1. **`--vfs-cache-mode`**
   - 캐시 모드를 설정합니다.
   - 예시: `--vfs-cache-mode writes` (쓰기 캐시를 사용)
2. **`--config`**
   - rclone 설정 파일의 경로를 지정합니다.
   - 예시: `--config /path/to/rclone.conf`
3. **`--allow-other`**
   - 다른 사용자가 마운트된 파일 시스템에 접근할 수 있도록 허용합니다.
   - 예시: `--allow-other`
4. **`--read-only`**
   - 마운트된 파일 시스템을 읽기 전용으로 만듭니다.
   - 예시: `--read-only`
5. **`--buffer-size`**
   - 파일 전송 시 사용되는 버퍼 크기를 설정합니다.
   - 예시: `--buffer-size 32M`
6. **`--dir-cache-time`**
   - 디렉토리 목록을 캐시하는 시간을 설정합니다.
   - 예시: `--dir-cache-time 5m`
7. **`--log-file`**
   - 로그 파일의 경로를 지정합니다.
   - 예시: `--log-file /path/to/logfile.log`
8. **`--log-level`**
   - 로그 레벨을 설정합니다 (DEBUG, INFO, NOTICE, ERROR).
   - 예시: `--log-level INFO`
9. **`--umask`**
   - 생성된 파일의 기본 퍼미션 마스크를 설정합니다.
   - 예시: `--umask 022`
10. **`--uid`와 `--gid`**
    - 마운트된 파일 시스템의 파일과 디렉토리에 적용될 사용자와 그룹 ID를 설정합니다.
    - 예시: `--uid 1000 --gid 1000`

예를 들어, OneDrive를 마운트할 때 쓰기 캐시 모드를 사용하고, 다른 사용자도 접근 가능하게 설정하고, 로그 파일을 지정하려면 다음과 같이 명령어를 작성할 수 있습니다:

```sh
rclone mount onedrive: ~/OneDrive --vfs-cache-mode writes --allow-other --log-file /var/log/rclone.log --log-level INFO
```

이와 같이 필요한 플래그를 추가하여 원하는 동작을 설정할 수 있습니다. `rclone mount`의 전체 플래그 목록은 `--help`를 통해 전체 플래그의 종류와 기능을 확인할 수 있습니다. 

`rclone`에서 `--vfs-cache-mode` 옵션을 사용하여 캐시를 활성화하면, 캐시는 시스템을 종료하거나 마운트를 해제한 후에도 유지됩니다. 이는 다시 마운트할 때 캐시된 데이터를 재사용할 수 있도록 하기 위함입니다. 그러나 디스크 공간을 확보하기 위해 캐시를 수동으로 삭제할 수 있습니다.

### 캐시 파일 위치

기본적으로 `rclone`의 VFS 캐시는 사용자 홈 디렉토리 아래의 `.cache/rclone` 디렉토리에 저장됩니다.

### 캐시 파일 삭제 방법

1. **수동 삭제**: 터미널을 열고 다음 명령어를 실행하여 캐시 디렉토리를 삭제할 수 있습니다.

   ```sh
   rm -rf ~/.cache/rclone
   ```

2. **자동 삭제 설정**: `rclone`을 마운트할 때 `--vfs-cache-max-age` 플래그를 사용하여 캐시 파일의 최대 생존 시간을 설정할 수 있습니다. 예를 들어, 캐시 파일을 1시간 후에 자동으로 삭제하려면 다음과 같이 설정할 수 있습니다.

   ```bash
   rclone mount onedrive: ~/onedrive --vfs-cache-mode writes --vfs-cache-max-age 1h
   ```

### 전체 명령 예시

다음은 OneDrive를 마운트하고 캐시를 1시간 후에 자동으로 삭제하도록 설정하는 예시입니다.

```bash
rclone mount onedrive: ~/onedrive --vfs-cache-mode writes --vfs-cache-max-age 1h
```

이와 같이 설정하면 캐시가 자동으로 삭제되어 디스크 공간을 확보할 수 있습니다. 시스템을 종료할 때 캐시가 자동으로 삭제되기를 원한다면, `rclone`을 마운트 해제하고 캐시를 삭제하는 스크립트를 만들고 이를 종료 시 실행되도록 설정할 수 있습니다.

### 예시 스크립트

```bash
#!/bin/bash
# Unmount rclone
fusermount -u ~/onedrive
# Delete rclone cache
rm -rf ~/.cache/rclone
```

이 스크립트를 저장하고 실행 권한을 부여한 후, 시스템 종료 시 자동으로 실행되도록 설정할 수 있습니다.

이 방법으로 시스템 종료 시 캐시가 삭제되어 디스크 공간이 다시 확보될 것입니다.



캐시를 사용하지 않는 경우에는 몇 가지 불편함과 성능 저하가 있을 수 있습니다. 캐시가 꺼진 상태에서의 주요 차이점과 그 영향은 다음과 같습니다.

### 주요 차이점 및 영향

1. **파일 접근 속도 저하**:
   - 캐시를 사용하지 않으면 매번 파일을 열거나 저장할 때마다 원격 서버와 통신해야 하므로 파일 접근 속도가 느려질 수 있습니다. 특히, 파일 크기가 크거나 네트워크 연결이 느린 경우에는 더욱 그렇습니다.
2. **연속적인 파일 읽기/쓰기 성능 저하**:
   - 파일을 여러 번 읽거나 쓰는 경우, 캐시가 없는 상태에서는 반복적인 네트워크 요청이 발생하여 성능이 떨어집니다. 캐시는 이러한 반복 요청을 줄여 성능을 향상시킵니다.
3. **파일 시스템 일관성 문제**:
   - 캐시가 없으면 파일 시스템 일관성을 유지하기가 더 어려울 수 있습니다. 원격 서버와의 통신이 끊기거나 지연될 경우, 파일의 일관성이 깨질 수 있습니다.
4. **오프라인 접근 불가**:
   - 캐시는 로컬에 파일을 저장하여 네트워크 연결이 없을 때도 파일에 접근할 수 있게 합니다. 캐시가 없으면 네트워크 연결이 없을 때 파일에 접근할 수 없습니다.
5. **네트워크 사용량 증가**:
   - 캐시가 없으면 모든 파일 읽기/쓰기 작업이 네트워크를 통해 원격 서버와 직접적으로 이루어지므로 네트워크 사용량이 증가합니다. 이는 데이터 요금이나 네트워크 성능에 영향을 미칠 수 있습니다.

### 사용 예시

캐시를 사용하지 않고 `rclone`을 마운트하는 명령어는 다음과 같습니다:

```bash
rclone mount onedrive: ~/onedrive
```

이 상태에서 파일을 접근하거나 수정하는 작업을 하면 매번 원격 서버와의 통신이 필요하게 됩니다. 캐시를 사용하지 않더라도 기본적인 파일 작업은 가능하지만, 위에서 언급한 불편함과 성능 저하를 경험할 수 있습니다.

### 캐시 없이 사용하는 것이 적절한 경우

- 원격 저장소의 파일을 자주 변경하지 않고 읽기 전용으로 사용한다면 캐시 없이도 큰 문제가 없을 수 있습니다.
- 네트워크 연결이 안정적이고 빠르다면 캐시가 없더라도 성능 저하를 크게 느끼지 않을 수 있습니다.

결론적으로, 캐시 없이 사용할 수는 있지만, 성능과 사용 편의성 면에서 캐시를 사용하는 것이 훨씬 유리합니다. 캐시 사용이 필수적이지 않지만, 보다 원활한 사용을 위해 캐시 옵션을 고려하는 것이 좋습니다.




### `--vfs-cache-mode` 옵션 설명

 `--vfs-cache-mode full` 옵션을 사용하면 읽기와 쓰기 모두에 대해 캐시를 사용하게 됩니다. 이 모드는 `rclone`의 VFS(Virtual File System) 캐시 기능을 최대한 활용하여 파일 시스템의 성능을 최적화합니다.

`--vfs-cache-mode` 옵션에는 여러 가지 수준이 있으며, 각 수준은 캐시의 사용 범위를 결정합니다:

1. **`off`**: 캐시를 사용하지 않습니다.
2. **`minimal`**: 최소한의 캐시를 사용합니다. 예를 들어, 쓰기 작업을 위해 열린 파일은 캐시되지 않습니다.
3. **`writes`**: 쓰기 작업에 대해서만 캐시를 사용합니다. 파일을 작성하거나 수정할 때 로컬에 캐시한 후 한 번에 업로드합니다.
4. **`full`**: 읽기와 쓰기 모두에 대해 캐시를 사용합니다. 이는 원격 저장소와의 모든 파일 작업을 로컬 캐시를 통해 수행하여 성능을 최적화합니다.

### `full` 모드의 동작 방식

- **읽기 캐시**: 파일을 처음 열 때 로컬에 다운로드하여 캐시합니다. 이후에 동일한 파일을 다시 열면 로컬 캐시에서 빠르게 접근할 수 있습니다.
- **쓰기 캐시**: 파일을 수정하거나 새로 작성할 때 먼저 로컬에 저장한 후, 백그라운드에서 원격 저장소로 업로드합니다. 이는 특히 대규모 파일 작업이나 빈번한 파일 수정 시 유용합니다.

### 명령어 예시

다음은 `full` 모드를 사용하여 OneDrive를 마운트하는 예시입니다:

```bash
rclone mount onedrive: ~/onedrive --vfs-cache-mode full
```

이 명령어를 사용하면 읽기와 쓰기 작업 모두에서 로컬 캐시가 사용되므로, 네트워크 상태와 무관하게 파일 접근 속도가 빨라지고 성능이 향상됩니다.

### 주의 사항

- **디스크 공간 사용**: `full` 모드를 사용하면 로컬 디스크 공간을 더 많이 사용하게 됩니다. 따라서 충분한 디스크 공간이 있는지 확인해야 합니다.

- **캐시 관리**: 캐시된 파일이 오래된 경우 자동으로 삭제되도록 `--vfs-cache-max-age` 옵션을 사용할 수 있습니다. 예를 들어, 캐시 파일을 24시간 후에 자동으로 삭제하려면 다음과 같이 설정합니다:

  ```bash
  rclone mount onedrive: ~/onedrive --vfs-cache-mode full --vfs-cache-max-age 24h
  ```

### 결론

`--vfs-cache-mode full` 옵션을 사용하면 읽기와 쓰기 모두에서 캐시를 활용하여 성능을 최적화할 수 있습니다. 네트워크 상태나 파일 접근 속도가 중요한 경우 이 옵션을 사용하는 것이 좋습니다.