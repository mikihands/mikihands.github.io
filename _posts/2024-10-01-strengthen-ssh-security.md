### GCP VM SSH 보안 강화 기록

**2024-10-01**

오늘은 GCP VM에 대한 **SSH 보안 강화** 작업을 했다. 해킹 공격이 끊이지 않는 상황에서 단순히 RSA 키 인증만 사용하는 것에 대한 불안감이 있었고, 조금 더 체계적으로 접근하기로 결정했다. 특히 외부에서 작업할 때 보안이 걱정되기도 하고, 그동안 귀찮아서 신경을 못 썼던 부분인데 이번 기회에 확실히 개선해 두었다.

#### 1. **PasswordAuthentication 비활성화**

먼저 `sshd_config` 파일에서 **`PasswordAuthentication`**을 `no`로 설정했다. 패스워드 인증은 보안상 위험 요소가 많으니, **RSA 키** 인증만을 사용하도록 완전히 차단했다. 이 조치 하나만으로도 로그인 시도의 공격을 크게 줄일 수 있다.

#### 2. **집에 있는 Raspi를 통한 보안 강화**

집에서 상시 동작 중인 **라즈베리 파이(Raspi)**를 점프 서버로 사용하기로 했다. GCP VM의 방화벽 규칙을 설정해 **Raspi의 WAN IP**만 SSH 포트(22번)에 대한 접근을 허용했다.

Raspi는 고정된 IP가 아니어서 매번 수동으로 관리하기는 번거로웠지만, 사실 IP가 자주 바뀌지는 않으니 IP가 변경될 때는 GCP 콘솔에서 방화벽 규칙을 수동으로 업데이트하기로 했다. 자동화 스크립트를 작성해보기도 했지만, GCP CLI 설정 등에서 시간이 좀 걸려서 당분간은 수동으로 관리하는 게 더 편할 것 같다.

#### 3. **SSH 점프 서버 설정**

외부에서 작업할 때는 매번 Raspi로 점프해서 GCP VM에 접근해야 하는데, 이 과정도 단순화할 방법이 없을까 고민하다가, **SSH config 파일**을 활용하는 방법을 찾았다. `-J` 옵션을 수동으로 입력하지 않고, **ProxyJump** 설정을 추가해서 Raspi를 경유하도록 설정했다.

SSH config 파일을 이렇게 수정했다:

```bash
Host mikihands
    HostName mikihands.com
    User jessekim80
    IdentityFile ~/.ssh/id_rsa
    ProxyJump raspi  # Raspi를 점프 호스트로 설정

Host raspi
    HostName blog.mikihands.com
    User jesse
    Port 2222
    IdentityFile ~/.ssh/id_rsa
```

이제 `ssh mikihands`라고만 입력하면 **자동으로 Raspi를 경유해서 GCP VM에 접속**할 수 있다. 더 이상 매번 `-J` 옵션을 쓸 필요가 없으니, 외부에서도 쉽게 GCP VM에 안전하게 접근할 수 있게 되었다.

#### 4. **PostgreSQL 포트 포워딩도 함께 설정**

외부에서 DB에 접근할 때도 SSH config 파일을 통해 포트 포워딩을 자동화했다. 이제 **로컬에서 5433 포트로** 접근하면 **GCP VM의 PostgreSQL 5432 포트로 연결**된다.

```bash
Host postgres
    HostName mikihands.com
    User jessekim80
    IdentityFile ~/.ssh/id_rsa
    LocalForward 5433 localhost:5432
    ProxyJump raspi  # Raspi를 경유해서 PostgreSQL에 접근
```

#### 5. **느낀 점**

오늘 작업을 마치고 나니, 보안이 한층 더 강화된 느낌이라 마음이 든든하다. 특히 외부에서 작업할 때 보안에 대한 걱정 없이 **Raspi를 경유해서 안전하게 GCP VM에 접근**할 수 있게 된 점이 가장 큰 성과다. 앞으로도 IP가 변경될 때만 주기적으로 GCP 콘솔에서 방화벽 규칙을 업데이트하는 수고를 감수하면, 아주 큰 문제 없이 서버를 안전하게 운영할 수 있을 것 같다.