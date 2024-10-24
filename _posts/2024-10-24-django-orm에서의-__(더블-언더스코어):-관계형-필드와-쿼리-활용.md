---
layout: post
title: "Django ORM에서의 __(더블 언더스코어): 관계형 필드와 쿼리 활용"
date: 2024-10-24 10:09:00 +09:00
categories: Django
tags: Django ORM
---

오늘은 Django ORM에서 매우 자주 사용되는 __(더블 언더스코어)에 대해 좀 더 깊이 파헤쳐 보려고 한다. 사실 이 기호는 처음에는 낯설고 복잡해 보였는데, 그 의미와 역할을 제대로 이해하니까 정말 강력하고 유용한 도구라는 걸 알게 됐다. 특히 모델 간의 관계를 처리하고 데이터베이스에서 다양한 조건으로 데이터를 필터링할 때 필수적인 역할을 한다는 걸 깨달았다.

## __(더블 언더스코어)란?
Django에서 __(더블 언더스코어)는 모델 간의 관계 필드를 통해 데이터베이스 쿼리를 작성할 때 사용된다. 간단히 말해, 다른 테이블과 연결된 필드에 접근하거나, 특정 조건을 가진 데이터를 필터링하는 데 사용된다.

예를 들어, 두 개 이상의 모델이 ForeignKey나 OneToOneField 등의 관계로 연결되어 있을 때, 그 관계를 통해 원하는 필드에 접근할 수 있다. 여기서 __는 일종의 "연결 고리" 역할을 하며, 서로 다른 모델의 필드들을 체인처럼 연결해서 접근하는 방식이다.

## 예시: 모델 간의 관계에서의 __ 사용
간단한 예시를 들어보자. 만약 Profile 모델이 User 모델과 연결되어 있다고 하자. 그리고 그 사용자 모델에는 username이라는 필드가 있다. 우리는 Profile 모델에서 User의 username을 기반으로 특정 프로필들을 필터링할 수 있다.

### 모델 정의:
```python
from django.db import models
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    pass

class Profile(models.Model):
    user = models.OneToOneField(CustomUser, on_delete=models.CASCADE)
    bio = models.TextField()
```

위 예시에서 Profile 모델은 CustomUser와 1:1 관계를 맺고 있다. 그럼 이제 __를 사용하여 Profile에서 CustomUser의 필드를 접근하는 방법을 보자.

### __를 사용한 쿼리:
```python
# CustomUser의 username을 기반으로 Profile 객체 필터링
profiles = Profile.objects.filter(user__username='jessekim80')
```

여기서 user__username이 핵심이다. user는 Profile 모델의 CustomUser와 연결된 필드이고, 그 필드의 **username**에 접근하려면 __로 연결하면 된다. 이렇게 하면 Profile 모델에서 CustomUser의 username 필드를 기준으로 데이터베이스에서 조건에 맞는 데이터를 가져올 수 있다.

## __의 다양한 활용
__는 단순히 필드 접근뿐만 아니라, 조건을 부여하는 데도 자주 사용된다. Django는 쿼리셋에서 다양한 조건을 설정할 수 있는 여러 필터링 기능을 제공하는데, 그 대부분이 __를 통해 구현된다. 자주 사용되는 조건들은 다음과 같다:

- `exact`: 정확히 일치하는 값을 찾는다. (기본값이므로 생략 가능)
- `icontains`: 대소문자 구분 없이 포함된 값을 찾는다.
- `gt`, `lt`, `gte`, `lte`: 크거나 작은 값을 찾는 조건 (greater than, less than).
- `startswith`, `endswith`: 특정 문자열로 시작하거나 끝나는 값을 찾는다.

### 예시: 다양한 필터 조건
```python
# CustomUser의 이메일이 'gmail.com'을 포함하는 모든 프로필 조회
profiles_with_gmail = Profile.objects.filter(user__email__icontains='gmail.com')

# CustomUser의 ID가 10보다 큰 프로필 조회
profiles_with_id_gt_10 = Profile.objects.filter(user__id__gt=10)

# CustomUser의 username이 'jesse'로 시작하는 프로필 조회
profiles_starting_with_jesse = Profile.objects.filter(user__username__startswith='jesse')
```

이처럼, __ 뒤에 다양한 조건을 붙여 쿼리를 생성할 수 있고, 이렇게 필터링된 데이터를 쉽게 조회할 수 있다.

## __를 통한 다중 관계 필드 접근
Django에서 가장 강력한 기능 중 하나는 여러 개의 모델이 연결된 경우에도 __를 통해 다중 관계를 따라가며 데이터를 조회할 수 있다는 점이다. 예를 들어, Order라는 모델이 Profile과 연결되어 있고, 다시 Profile이 CustomUser와 연결되어 있는 상황을 생각해보자.

### 다중 관계 모델 예시:
```python
class Order(models.Model):
    profile = models.ForeignKey(Profile, on_delete=models.CASCADE)
    order_date = models.DateTimeField()
    amount = models.DecimalField(max_digits=10, decimal_places=2)
```

이제 우리는 Order에서 CustomUser의 username을 기준으로 데이터를 조회하고 싶다고 하자.

### 다중 관계 필터링:
```python
# Order에서 CustomUser의 username이 'jessekim80'인 주문 조회
orders_for_user = Order.objects.filter(profile__user__username='jessekim80')
```

여기서 profile__user__username으로 여러 단계의 관계를 타고 CustomUser의 필드에 접근할 수 있다. Order → Profile → CustomUser로 이어지는 관계를 __로 연결하여 간단하게 필터링을 구현한 것이다.

## 요약
오늘의 핵심 개념인 **__**는 Django ORM에서 매우 중요한 역할을 한다. 여러 모델이 연결된 관계형 데이터베이스에서 필드를 통해 쿼리하고 데이터를 필터링할 때, 이 __는 일종의 "연결 고리" 역할을 한다. 이를 통해 우리는 다중 관계의 모델 간 데이터를 매우 직관적이고 쉽게 조회할 수 있다.

- __는 모델 간 관계를 따라 필드를 참조할 때 사용된다.
- 다양한 쿼리 조건을 설정해 데이터를 필터링할 수 있다.
- 다중 관계의 필드를 연결해 복잡한 쿼리도 간단하게 처리할 수 있다.

이제 이 __의 역할과 사용법을 이해했으니, Django ORM에서 더욱 강력한 쿼리를 작성할 수 있을 것이다. 데이터베이스의 관계형 구조를 생각하며 이 기호를 적절히 활용하면, 효율적인 데이터 처리와 더불어 복잡한 관계를 간단하게 다룰 수 있게 될 것이다.