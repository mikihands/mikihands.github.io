---
layout: post
title:  "도커허브를 이용하지 않고 도커이미지 전송하기"
date:   2024-08-21 11:13:00 +0900
categories: Docker
tags: Docker SSH
---



### 도커허브를 이용하지 않고 도커이미지 전송하기

도커(Docker)는 애플리케이션을 컨테이너로 묶어 배포하고 관리할 수 있게 해주는 강력한 도구이다. 보통 도커 이미지를 공유하거나 배포할 때 도커 허브(Docker Hub)를 많이 사용하지만, 때로는 도커 허브를 사용하지 않고 직접 이미지를 전송해야 할 때가 있다. 이 글에서는 SSH를 통해 도커 이미지를 직접 VM(Virtual Machine)으로 전송하고 배포하는 방법을 소개한다.

#### 1. 도커 이미지 저장하기

먼저 로컬 머신에서 도커 이미지를 파일로 저장해야 한다. 이때 `docker save` 명령어를 사용한다. 예를 들어, `myapp:latest`라는 이름의 이미지를 `myapp.tar`라는 파일로 저장하려면 다음과 같이 한다.

```bash
docker save -o myapp.tar myapp:latest
```

이 명령어는 `myapp:latest` 이미지를 `myapp.tar`라는 파일로 변환하여 저장한다. 이 파일은 나중에 다른 시스템에서 이미지를 복원하는 데 사용된다.

#### 2. SSH를 이용한 이미지 전송

이미지를 저장한 후, 이를 SSH를 통해 원격 VM으로 전송할 수 있다. `scp` 명령어를 사용하면 SSH 연결을 통해 파일을 안전하게 전송할 수 있다. 다음은 `myapp.tar` 파일을 VM으로 전송하는 예시이다.

```bash
scp myapp.tar username@vm_ip:/path/to/destination/
```

여기서 `username`은 VM에 접속할 때 사용하는 사용자 이름이고, `vm_ip`는 VM의 IP 주소이다. `/path/to/destination/`은 파일을 저장할 디렉토리 경로이다. 예를 들어, 유저 이름이 `user`이고, VM의 IP 주소가 `192.168.1.100`, 파일을 `/home/user/`에 저장하고자 한다면 다음과 같이 입력한다.

```bash
scp myapp.tar user@192.168.1.100:/home/user/
```

이렇게 하면 `myapp.tar` 파일이 로컬에서 VM으로 복사된다.

#### 3. VM에서 도커 이미지 로드하기

VM에 접속한 후, 전송된 tar 파일을 도커 이미지로 로드해야 한다. 이때 `docker load` 명령어를 사용한다.

```bash
docker load -i /path/to/myapp.tar
```

앞서 `/home/user/`에 파일을 전송했다면 다음과 같이 명령어를 입력한다.

```bash
docker load -i /home/user/myapp.tar
```

이 명령어는 `myapp.tar` 파일을 도커 이미지로 변환하여 로드한다. 이제 이 이미지를 기반으로 컨테이너를 생성할 수 있다.

#### 4. 도커 컨테이너 실행하기

이미지를 성공적으로 로드한 후, 도커 컨테이너를 실행할 수 있다. `docker run` 명령어를 사용하여 로드된 이미지를 기반으로 컨테이너를 생성하고 실행한다.

```bash
docker run -d --name myapp_container -p 8000:8000 myapp:latest
```

이 명령어는 `myapp:latest` 이미지를 사용하여 `myapp_container`라는 이름의 컨테이너를 생성하고 백그라운드에서 실행한다. 또한, 호스트의 8000번 포트를 컨테이너의 8000번 포트와 연결한다.

#### 5. 추가 팁: 압축 사용하기

tar 파일이 클 경우 압축을 사용하면 전송 시간을 줄일 수 있다. 이미지를 저장하면서 바로 압축하는 방법은 다음과 같다.

```bash
docker save myapp:latest | gzip > myapp.tar.gz
```

이제 `myapp.tar.gz` 파일을 `scp` 명령어를 사용하여 전송하고, VM에서 압축을 푼 후 이미지를 로드한다.

```bash
gzip -d myapp.tar.gz
docker load -i myapp.tar
```

#### 결론

이 글에서는 도커 허브를 사용하지 않고 SSH를 통해 도커 이미지를 VM으로 직접 전송하는 방법을 살펴보았다. 이 방법은 네트워크가 제한적이거나, 내부 네트워크에서만 이미지를 배포해야 하는 상황에서 유용하다. 이제 이 방법을 통해 도커 이미지를 안전하게 전송하고 배포할 수 있을 것이다.