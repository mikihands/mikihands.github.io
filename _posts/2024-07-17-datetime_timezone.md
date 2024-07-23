---
layout: post
title:  "Django timezone VS python datetime 의 차이"
date:   2024-07-17 18:32:00 +0900
categories: Django
tags: Django python
---

## 1. 시간대 변환 

사용자에게 시간을 표시할 때는 다음과 같이 변환할 수 있습니다:

```python
from django.utils.timezone import localtime

user_time = localtime(subscription.next_billing_date, pytz.timezone('Asia/Seoul'))
```



특정국가에 제한되어 서비스를 제공하는 것이 아니라면 Django나 Celery 서버 모두 UTC로 설정하고, 필요한 경우 특정국가의 사용자에게만 localtime함수를 이용하여 로컬시간을 표시해주는 것이 좋다. 



왜냐하면? db나 paypal, stripe 와 같은 결제 도구역시 utc로 시간을 기록하거나 전송해주기 때문에 로직이 복잡해지면 시간대가 상이하여 오류가 발생할 수 있고 디버깅이 힘들어진다. 



## 2. `datetime` 모듈과 Django의 `timezone` 유틸리티의 차이점



`datetime` 모듈과 Django의 `timezone` 유틸리티는 둘 다 날짜와 시간을 다루기 위한 도구라는 측면에서 비슷해 보이지만 목적과 기능에서 차이가 있습니다.



### `datetime` 모듈

Python의 내장 모듈로, 기본적인 날짜와 시간 조작을 제공합니다.

- **주요 클래스**: `datetime.datetime`, `datetime.date`, `datetime.time`, `datetime.timedelta`

- **용도**: 단순한 날짜와 시간 계산, 현재 시간 가져오기, 날짜와 시간 형식 지정 및 변환 등

- 예시

  ```python
  import datetime
  now = datetime.datetime.now()  # 현재 로컬 시간
  utc_now = datetime.datetime.utcnow()  # 현재 UTC 시간
  ```



### Django의 `timezone` 유틸리티

Django 프레임워크의 일부로, 시간대 인식 날짜 및 시간을 다루기 위한 도구입니다.

- **주요 함수**: `now()`, `localtime()`, `make_aware()`, `make_naive()`

- **용도**: 시간대를 인식한 시간 관리, 시간대 변환, 시간대를 포함한 현재 시간 가져오기

- 예시

  ```python
  from django.utils import timezone
  now = timezone.now()  # 현재 시간 (설정된 시간대에 따라)
  local_time = timezone.localtime(now)  # 로컬 시간대로 변환
  ```



### `datetime` 과 `timezone`의 차이점 및 구분

1. **시간대 인식 여부**:
   - `datetime` 모듈은 기본적으로 시간대 정보를 포함하지 않습니다.
   - `timezone` 유틸리티는 시간대 인식을 지원하며, 시간대 변환을 쉽게 할 수 있습니다.
2. **Django 설정과의 통합**:
   - `timezone`은 Django 프로젝트의 `TIME_ZONE` 설정을 따릅니다.
   - `datetime` 모듈은 독립적으로 동작합니다.
3. **일관된 시간 관리**:
   - `timezone`을 사용하면 Django 프로젝트 전반에서 일관된 시간대를 유지할 수 있습니다.
   - `datetime` 모듈을 사용할 때는 시간대 관리에 주의를 기울여야 합니다.

### 요약

- **`datetime`**: 기본적인 날짜와 시간 조작.
- **`timezone`**: 시간대 인식 및 변환, Django 설정과 통합된 시간 관리.

Django 프로젝트 내에서 일관된 시간 관리를 위해 `timezone`을 사용하는 것이 일반적입니다.



## 3. `모듈`과 `유틸리티`



눈썰미가 있는 분 혹은 꼼꼼하신 분들은 윗 글에서 datetime은 `모듈`이라는 용어를 사용하여 지칭하고, `timezone`은 `유틸리티`라는 용어로 지칭하고 있다는 것을 눈치채셨을 것 같습니다. 이미 그 이유를 알고 계신 분들은 너무 당연하다고 생각해서 위화감을 느끼지 못하셨을 수도 있겠지요. 



왜 이 두가지를 `모듈`이나 `유틸리티`라는 다른 용어로 지칭할까요? 



`datetime`과 `timezone`을 각각 모듈과 유틸리티로 구분한 이유는 이 둘의 **성격과 역할이 다르기 때문**입니다.



### `datetime` 모듈

- **모듈**: Python 표준 라이브러리에 포함된 하나의 독립적인 모듈입니다. 모듈은 여러 함수와 클래스 등을 포함하여 특정 기능을 수행할 수 있도록 구성된 코드의 집합입니다.

- **역할**: 날짜와 시간 관련 기본적인 기능을 제공하는 모듈로, 다양한 클래스 (`datetime.datetime`, `datetime.date`, `datetime.time`, `datetime.timedelta`)를 통해 날짜와 시간을 조작할 수 있습니다.

- **독립성**: `datetime` 모듈은 Django와는 독립적으로 동작하며, Python의 다른 프로젝트에서도 사용할 수 있는 범용 모듈입니다.

  

### Django의 `timezone` 유틸리티

- **유틸리티**: Django 프레임워크의 일부로 포함된 도구입니다. 유틸리티는 특정 목적을 달성하기 위한 함수와 클래스의 모음으로, 특정 작업을 쉽게 수행할 수 있도록 돕는 보조 도구입니다.
- **역할**: 시간대 인식 시간 관리, 시간대 변환 등의 기능을 제공하며, Django 프로젝트의 설정 (`TIME_ZONE`)과 밀접하게 연동되어 작동합니다.
- **통합성**: `timezone` 유틸리티는 Django의 일부분으로, Django 프로젝트 내에서 일관된 시간 관리를 위해 사용됩니다. 이는 Django의 다른 구성 요소들과 긴밀히 통합되어 작동합니다.
- 

### 소결

- **`datetime` 모듈**: Python 표준 라이브러리에 포함된 독립적인 모듈로, 날짜와 시간 조작을 위한 기본 기능을 제공합니다.
- **Django `timezone` 유틸리티**: Django 프레임워크의 일부로, 시간대 인식 시간 관리와 시간대 변환 기능을 제공하며, Django 설정과 통합된 보조 도구입니다.

따라서, 글에서는 `datetime`을 모듈로, `timezone`을 유틸리티로 구분하여 용어를 사용함으로써 이들의 성격과 역할의 차이를 명확히 하고 있습니다.

