---
layout: post
title:  "Docker 이미지 빌드시 수정내용이 적용되지 않는 현상 해결"
date:   2024-07-13 23:16:00 +0900
categories: Docker
tags: Docker Deployment
---

### Docker 이미지 빌드시 변경사항이 반영되지 않는 현상의 원인과 해결 방법

Docker 이미지를 빌드할 때 변경사항이 반영되지 않는 문제는 많은 개발자들이 직면하는 흔한 문제입니다. 이는 Docker가 빌드 속도를 최적화하기 위해 캐싱 메커니즘을 사용하기 때문입니다. 이 글에서는 Docker 이미지 빌드시 변경사항이 반영되지 않는 원인을 분석하고, 이를 해결할 수 있는 세 가지 방법을 제시합니다.

#### 원인

Docker는 빌드 프로세스를 최적화하기 위해 각 명령어 단계의 결과를 캐시합니다. 따라서 Dockerfile의 내용이 변경되지 않으면, Docker는 이전에 캐시된 단계를 재사용하여 빌드 시간을 단축합니다. 이로 인해 코드가 변경되었음에도 불구하고 새로운 이미지에 반영되지 않는 문제가 발생할 수 있습니다.

### 해결 방법

#### 1. 이미지 태그 변경 및 컴포즈 파일 업데이트

이미지 빌드 시마다 새로운 태그를 사용하고, Docker Compose 파일을 업데이트하여 새로운 이미지를 배포하는 방법입니다.

**예시:**

1. **새로운 이미지 태그로 빌드:**

   ```bash
   sudo docker build -t myapp:1.1 .
   sudo docker push myapp:1.1
   ```

2. **Docker Compose 파일 업데이트:**

   ```yaml
   version: '3.9'
   
   services:
     web:
       image: myapp:1.1
       ports:
         - "8000:8000"
   ```

3. **스택 업데이트:**

   ```bash
   sudo docker stack deploy -c docker-compose.yml myapp
   ```

이 방법은 매번 새로운 태그를 사용하여 변경사항이 항상 반영되도록 합니다. 그러나 태그를 관리하는 데 번거로움이 있을 수 있습니다.

#### 2. Dockerfile에 의미 없는 `echo` 명령 추가

Dockerfile에 의미 없는 `RUN echo` 명령어를 추가하여 캐시를 무효화하는 방법입니다. 이 방법은 파일 시스템에 변화를 일으켜 Docker가 캐시를 사용하지 않고 이미지를 새로 빌드하게 합니다.

**예제:**

```dockerfile
FROM python:3.9

# Install dependencies
COPY requirements.txt /app/requirements.txt
RUN pip install --no-cache-dir -r /app/requirements.txt

# Add a meaningless RUN command to bust the cache
RUN echo "hello!! have a good day!!"

# Copy application code
COPY . /app

# Set working directory
WORKDIR /app

CMD ["python", "manage.py", "runserver"]
```

이 방법은 간단하지만, Dockerfile이 지저분해질 수 있습니다.

#### 3. `ARG CACHEBUST`를 사용한 캐시 무효화

Dockerfile에 `ARG CACHEBUST`를 정의하고, 빌드할 때마다 해당 값을 변경하여 캐시를 무효화하는 방법입니다. 두 가지 방식이 있습니다: Dockerfile에서 값을 **직접 올리거나**, 빌드 **명령어에서 동적으로** 값을 설정하는 방법입니다.

**예제:**

1. **Dockerfile에 `ARG CACHEBUST` 추가:**

```dockerfile
FROM python:3.9

ARG CACHEBUST=1

# Install dependencies
COPY requirements.txt /app/requirements.txt
RUN pip install --no-cache-dir -r /app/requirements.txt

# Copy application code
COPY . /app

# Set working directory
WORKDIR /app

CMD ["python", "manage.py", "runserver"]
```

1. **빌드 명령어에서 동적으로 값 설정:**

```bash
sudo docker build --build-arg CACHEBUST=$(date +%s) -t myapp:latest .
```

1. **혹은 Dockerfile에서 숫자 값 수동 증가:**

```dockerfile
ARG CACHEBUST=2
```

빌드 명령어:

```bash
sudo docker build -t myapp:latest .
```

이 방법은 명확하고, Dockerfile을 깔끔하게 유지하면서 캐시를 무효화할 수 있습니다.

### 결론

Docker 이미지를 빌드할 때 변경사항이 반영되지 않는 문제를 해결하기 위해 여러 가지 방법을 사용할 수 있습니다. 첫째, 이미지 태그를 변경하여 새로운 이미지를 빌드하고 배포하는 방법. 둘째, Dockerfile에 의미 없는 `RUN echo` 명령을 추가하여 캐시를 무효화하는 방법. 셋째, `ARG CACHEBUST`를 사용하여 캐시를 무효화하는 방법입니다. 이 중에서 저는 첫째 방법과 Dockerfile에서 `CACHEBUST` 아규먼트를 사용하여 업데이트할 때 직접 숫자를 올리는 방법 두가지를 병행하는 것을 선호합니다. 이 방법은 명확하고 관리하기 쉬우며, Dockerfile을 깔끔하게 유지할 수 있습니다. (물론 하나만 해도 효과는 확실합니다!)

여러분도 자신의 환경과 취향에 맞는 방법을 선택하여 사용하시길 바랍니다. Docker 이미지 빌드 문제를 해결하고, 효율적으로 애플리케이션을 배포하세요!



#### 참고 : $(date +%s) 란?

`$(date +%s)`는 쉘 명령어로, 현재 시간을 초 단위로 나타내는 유닉스 타임스탬프를 반환합니다. 이 명령어는 현재 시간을 1970년 1월 1일 00:00:00 UTC부터 경과된 초 수로 표현합니다.

#### 유닉스 타임스탬프의 예시

예를 들어, 2024년 7월 24일 12:00:00의 유닉스 타임스탬프는 `1721841600`입니다. `$(date +%s)`를 사용하면 매번 실행할 때마다 현재 시각을 초 단위로 반환하여, 유일한 값(즉, 매번 다른 값)을 생성합니다.

이 명령어를 Docker 빌드 명령어의 `--build-arg` 옵션과 함께 사용하면, 매번 새로운 값이 생성되어 캐시를 무효화할 수 있습니다.

터미널에서 직접 실행해보면 다음과 같습니다:

```bash
date +%s
1627137600
```
