---
layout: post
title: "Django 로깅 설정 및 활용가이드"
date: 2024-10-18 09:27:00 +09:00
categories: Django
tags: Django Python Logging
---

# Django 로깅 설정 및 활용 가이드

Django에서 로깅을 설정할 때에는 로그의 **레벨**을 이해하는 것이 중요하다. 로그 레벨에는 `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` 다섯 가지가 있다. `DEBUG`는 가장 상세한 정보를 기록하며 개발 단계에서 주로 사용되고, `INFO`는 일반적인 동작을, `WARNING`은 주의가 필요한 상황을, `ERROR`는 오류가 발생했을 때, `CRITICAL`은 매우 심각한 오류를 기록한다. 참고로, `DEBUG` 레벨을 사용하면 그보다 낮은 모든 레벨(`INFO`, `WARNING`, `ERROR`, `CRITICAL`)의 로그가 모두 기록된다.

### 1. Django에서 기본적인 로깅 설정하기
Django는 기본적으로 Python의 로깅 라이브러리를 사용하여 로깅을 설정할 수 있다. 기본적인 로깅 설정은 `settings.py` 파일에서 할 수 있다. 다음은 간단한 로깅 설정의 예시이다.

```python
# settings.py
import os

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '[%(asctime)s] %(levelname)s [%(name)s:%(lineno)s] %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(message)s'
        },
    },
    'handlers': {
        'file': {
            'level': 'DEBUG',
            'class': 'logging.FileHandler',
            'filename': os.path.join(BASE_DIR, 'debug.log'),
            'formatter': 'verbose'
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
}
```

이 설정을 통해 Django 애플리케이션의 모든 DEBUG 수준 이상의 로그가 `debug.log` 파일에 기록된다. `formatter`를 사용하여 로그 메시지의 형식을 정의할 수 있다. 위 설정에서는 `verbose` 형식을 사용하여 로그에 타임스탬프, 로그 레벨, 로그 발생 위치 등을 포함하고 있다.

### 2. 앱별 로거 설정하기
Django에서는 각 앱별로 로거를 설정할 수 있다. 예를 들어 `app_1`이라는 앱이 있다고 가정하고, 이 앱에 대해 DEBUG 레벨과 ERROR 레벨을 로깅하되, ERROR 레벨은 별도의 파일(`error.log`)에 저장하는 설정 예시는 다음과 같다.

```python
# settings.py
import os

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '[%(asctime)s] %(levelname)s [%(name)s:%(lineno)s] %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(message)s'
        },
    },
    'handlers': {
        'file': {
            'level': 'DEBUG',
            'class': 'logging.FileHandler',
            'filename': os.path.join(BASE_DIR, 'debug.log'),
            'formatter': 'verbose'
        },
        'error_file': {
            'level': 'ERROR',
            'class': 'logging.FileHandler',
            'filename': os.path.join(BASE_DIR, 'error.log'),
            'formatter': 'verbose'
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'DEBUG',
            'propagate': True,
        },
        'app_1': {
            'handlers': ['file', 'error_file'],
            'level': 'DEBUG',
            'propagate': False,
        },
    },
}
```

위 설정에서는 `app_1`의 DEBUG 레벨 로그는 `debug.log` 파일에 기록되며, ERROR 레벨 로그는 `error.log` 파일에 별도로 기록된다. 이를 통해 각 로그 레벨에 따른 분류와 관리가 용이해진다.

### 3. 파일 용량 무한 증가 방지하기 (롤링 파일 핸들러 사용)
로그 파일이 너무 커지는 것을 방지하기 위해 **Rotating File Handler**를 사용할 수 있다. 이 핸들러는 파일 크기가 설정된 용량에 도달하면 자동으로 새 로그 파일을 생성하고, 오래된 파일은 백업으로 유지하는 방식으로 작동한다.

다음은 `RotatingFileHandler`를 사용하는 예시이다.

```python
from logging.handlers import RotatingFileHandler

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '[%(asctime)s] %(levelname)s [%(name)s:%(lineno)s] %(message)s'
        },
    },
    'handlers': {
        'rotating_file': {
            'level': 'DEBUG',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(BASE_DIR, 'debug.log'),
            'maxBytes': 1024 * 1024 * 5,  # 5MB
            'backupCount': 3,
            'formatter': 'verbose'
        },
    },
    'loggers': {
        'django': {
            'handlers': ['rotating_file'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
}
```

여기서 `maxBytes`는 로그 파일이 최대 몇 바이트까지 커질 수 있는지를 설정하며, `backupCount`는 백업으로 유지할 로그 파일의 개수를 의미한다. 위 설정에서는 로그 파일이 5MB를 초과하면 새로운 파일로 롤링되며, 최대 3개의 백업 파일을 유지한다. 즉, 로그 파일이 `debug.log`, `debug.log.1`, `debug.log.2`, `debug.log.3`으로 나뉘어 관리되며 더 오래된 로그는 삭제된다.

### 4. 로깅 설정의 유연성
위에서 사용한 `RotatingFileHandler` 외에도 **Timed Rotating File Handler**를 사용할 수 있다. 이 핸들러는 파일 크기와 상관없이 일정 시간이 지나면 로그 파일을 롤링한다. 예를 들어, 매일 자정에 로그 파일을 새로 생성하도록 설정할 수 있다.

```python
from logging.handlers import TimedRotatingFileHandler

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '[%(asctime)s] %(levelname)s [%(name)s:%(lineno)s] %(message)s'
        },
    },
    'handlers': {
        'timed_rotating_file': {
            'level': 'DEBUG',
            'class': 'logging.handlers.TimedRotatingFileHandler',
            'filename': os.path.join(BASE_DIR, 'debug.log'),
            'when': 'midnight',
            'interval': 1,
            'backupCount': 7,
            'formatter': 'verbose'
        },
    },
    'loggers': {
        'django': {
            'handlers': ['timed_rotating_file'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
}
```

위 설정에서는 `when`과 `interval`을 통해 매일 자정마다 로그 파일을 롤링하고, 최대 7개의 백업 파일을 유지하도록 설정하였다. 이를 통해 특정 주기마다 로그 파일을 새롭게 유지할 수 있어 관리가 용이하다.

### 5. 마무리
Django의 로깅 설정은 애플리케이션을 모니터링하고 문제를 추적하는 데 필수적이다. 특히 로그 파일의 크기를 제한하고 롤링을 통해 효율적으로 관리함으로써 서버의 안정성을 유지하는 것이 중요하다. 위 예시들을 참고하여 프로젝트에 맞는 로깅 설정을 적용해 보자.

이제 파일로 로그를 저장하고, 용량 문제를 예방하기 위해 롤링 설정을 적용하는 방법을 이해했으리라 생각한다. 이런 설정은 서버 운영 중 발생할 수 있는 문제를 추적하고 해결하는 데 큰 도움이 될 것이다.

