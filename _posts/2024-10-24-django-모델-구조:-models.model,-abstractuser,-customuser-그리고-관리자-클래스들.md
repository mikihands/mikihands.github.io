---
layout: post
title: "Django 모델 구조: models.Model, AbstractUser, CustomUser 그리고 관리자 클래스들"
date: 2024-10-24 09:56:00 +09:00
categories: Django
tags: Django Python Model
---

Django의 모델 구조를 제대로 파악해보고 정리해본다. 지금까지는 그냥 문서나 튜토리얼을 따라가면서 "원래 이렇게 사용하는 거구나"라고 생각하고 넘어갔던 부분인데, 이번 기회에 더 깊이 파고들어 보니 꽤 흥미로운 관계들이 있다는 걸 알게 됐다.

## models.Model의 역할
먼저 Django에서 가장 기본적인 모델 클래스는 **models.Model**이다. 모든 Django 모델은 models.Model을 상속받아서 정의된다. 이 클래스는 내가 정의한 모델을 데이터베이스 테이블로 매핑해주는 역할을 한다. 각 모델 클래스는 데이터베이스의 테이블이고, 그 안에 있는 필드들은 테이블의 열(columns)이 되는 것이다.

여기서 중요한 점은 models.Model을 상속받음으로써 기본적으로 사용할 수 있는 여러 가지 유용한 메서드들이 제공된다는 것이다. 예를 들어, 다음과 같은 메서드들이 있다:

- `save()`: 데이터베이스에 객체를 저장하는 메서드.
- `delete()`: 객체를 삭제하는 메서드.
- `objects.create()`: 새로운 객체를 만들고 저장하는 메서드.
- `objects.filter()`: 특정 조건을 만족하는 객체들을 조회하는 메서드.

그러니까, 내가 어떤 모델을 만들 때 models.Model을 상속받는 것만으로도 Django가 제공하는 모든 ORM 기능을 사용할 수 있게 된다.

## AbstractUser와 CustomUser의 관계
이제 사용자 모델로 넘어가 보자. Django에는 기본적으로 제공되는 **auth.User**라는 내장 사용자 모델이 있다. 이걸 기본적으로 사용할 수 있지만, 대부분의 경우 우리는 사용자 모델을 커스터마이징하고 싶어 한다. 예를 들어, 전화번호나 주소 같은 추가 필드를 넣고 싶은 경우다.

이때 사용하는 것이 바로 **AbstractUser**다. 이 클래스는 Django가 기본적으로 제공하는 사용자 모델(즉, auth.User)을 확장할 수 있도록 만들어진 추상 클래스다. 그런데 재미있는 점은 AbstractUser도 models.Model을 상속받고 있다는 것이다. 따라서, 내가 AbstractUser를 상속받아서 커스텀 사용자 모델을 만들면, 이미 models.Model의 기능을 모두 가지고 있는 것이 된다.

예를 들어, 이렇게 CustomUser를 만들 수 있다:

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    phone_number = models.CharField(max_length=15, blank=True)
    address = models.CharField(max_length=255, blank=True)
```

이 CustomUser 모델은 AbstractUser에서 확장된 것이니까, 사용자 인증과 관련된 모든 기능을 그대로 사용할 수 있을 뿐만 아니라, 내가 원하는 필드도 추가할 수 있다. 그리고 여전히 Django의 ORM 기능들을 자유롭게 사용할 수 있다. `save()`나 `delete()` 같은 메서드들이 자동으로 포함되는 것도 이 덕분이다.

## models.Manager와 AbstractUserManager의 역할
Django 모델에서 **models.Manager**는 데이터베이스에 대한 쿼리를 처리하는 인터페이스 역할을 한다. 내가 모델에 대한 쿼리를 보낼 때, `objects`라는 기본 매니저를 통해 데이터를 조회하거나 조작하게 된다. 이 매니저는 `filter()`나 `get()`, `create()` 같은 메서드를 제공해서 내가 쉽게 데이터를 다룰 수 있도록 해준다.

여기서 CustomUser와 같은 사용자 모델은 사용자 생성과 관련된 로직이 필요한데, 이때 등장하는 것이 **AbstractUserManager**이다. AbstractUserManager는 models.Manager를 상속받아서 사용자 생성에 특화된 메서드들을 제공한다. 예를 들어, `create_user()`나 `create_superuser()` 같은 메서드를 통해 일반 사용자나 관리자 계정을 쉽게 만들 수 있게 해준다.

```python
from django.contrib.auth.models import AbstractUser, BaseUserManager

class CustomUserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        # 사용자 생성 로직
        pass

    def create_superuser(self, email, password=None, **extra_fields):
        # 슈퍼유저 생성 로직
        pass
```

이렇게 사용자 생성에 필요한 매니저 로직을 CustomUserManager로 커스터마이징할 수 있다. 중요한 것은, 이 매니저도 결국은 **models.Manager**에서 확장된 것이기 때문에 데이터베이스 쿼리를 처리하는 기본적인 기능을 모두 가지고 있다는 점이다.

## 정리: 모델 클래스들 간의 관계
지금까지 정리해보니, Django의 모델 클래스들 간에는 다음과 같은 관계가 있다:

- **models.Model**은 모든 모델 클래스의 기반이 된다. 모든 Django 모델은 이를 상속받아 데이터베이스와 매핑되며, ORM 기능을 사용할 수 있다.
- **AbstractUser**는 models.Model을 상속받은 추상 클래스이며, 이를 상속받아 CustomUser 모델을 만들면 models.Model의 기능을 포함한 사용자 모델을 만들 수 있다.
- **AbstractUserManager**는 models.Manager를 상속받아 사용자 생성과 관련된 매니저 기능을 제공하며, 이를 통해 사용자 모델의 생성 로직을 커스터마이징할 수 있다.

결론적으로, 내가 사용자 모델을 확장해서 커스텀 필드를 추가할 때는 **AbstractUser**와 **models.Model**이 이미 잘 설계된 상속 구조를 통해 필요한 기능을 모두 포함하고 있으니, 그 구조를 이해하고 활용하면 훨씬 더 효율적으로 Django 모델을 설계할 수 있다.

이제 Django 모델 구조를 깊이 이해했으니, 앞으로 더 자유롭게 Django 프로젝트를 설계하고 확장할 수 있을 것 같다!