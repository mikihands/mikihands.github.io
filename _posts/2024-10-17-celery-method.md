---
layout: post
title: "Celery 주요 메서드와 관리 방법 정리"
date: 2024-10-17 22:28:00 +09:00
categories: Django
tags: Django Python Celery
---

### Celery 주요 메서드와 관리 방법 정리

이번에 Celery를 사용하면서 공부했던 주요 메서드와 관리 방법들에 대해 정리해보려고 한다. 이 내용들을 내 개발일지에 기록해두면 나중에 Celery와 관련된 작업을 다시 하게 될 때 참조하기 좋을 것 같다.

#### 1. `.apply_async()`와 `.delay()` 메서드

- **`.apply_async()`**

  - Celery에서 작업을 비동기로 예약할 때 가장 유연하게 사용할 수 있는 메서드이다.
  - **정교한 제어**: 특정 시점에 작업을 예약(`eta`), 지연(`countdown`), 특정 큐 지정(`queue`) 등의 기능을 사용할 수 있다.
  - 작업을 더 세부적으로 제어하고 싶은 경우 적합하다.
  - `AsyncResult` 객체의 주요 필드와 메서드:
    - **`id`**: 작업의 고유 식별자. 작업을 추적하거나 취소할 때 사용된다.
      ```python
      print(result.id)  # 작업의 고유 ID 출력
      ```
    - **`status`**: 작업의 현재 상태. `PENDING`, `STARTED`, `SUCCESS`, `FAILURE` 등의 값을 가질 수 있다.
      ```python
      print(result.status)  # 작업 상태 확인
      ```
    - **`get(timeout=None)`**: 작업의 결과를 반환한다. 타임아웃을 설정할 수 있으며, 작업이 완료될 때까지 대기한다.
      ```python
      result_value = result.get(timeout=10)  # 작업 결과 가져오기 (타임아웃 설정 가능)
      ```
    - **`ready()`**: 작업이 완료되었는지 여부를 반환한다.
      ```python
      print(result.ready())  # 작업 완료 여부 확인
      ```
    - **`successful()`**: 작업이 성공적으로 완료되었는지 여부를 반환한다.
      ```python
      print(result.successful())  # 작업 성공 여부 확인
      ```
    - **`failed()`**: 작업이 실패했는지 여부를 반환한다.
      ```python
      print(result.failed())  # 작업 실패 여부 확인
      ```
    - **`revoke(terminate=False, signal=None)`**: 예약된 작업을 취소할 수 있다. `terminate=True`를 설정하면 워커에서 실행 중인 작업도 종료된다.
      ```python
      result.revoke(terminate=True)  # 작업 취소
      ```

  ```python
    result = my_task.apply_async(countdown=600)  # 10분 후에 작업 실행
    print(result.status)  # 작업 상태 확인
    result_value = result.get(timeout=10)  # 작업 결과 가져오기 (타임아웃 설정 가능)
  ```

  - 예시:
    ```python
    my_task.apply_async(countdown=600)  # 10분 후에 작업 실행
    my_task.apply_async(eta=datetime.now() + timedelta(minutes=5))  # 특정 시간에 작업 실행
    my_task.apply_async(queue='high_priority')  # 특정 큐에 작업 보내기
    ```

- **`.delay()`**

  - `.apply_async()`의 간단한 버전으로, 기본적인 작업 실행을 위한 메서드이다.
  - **간단하고 직관적**: 추가적인 옵션 없이 작업을 비동기로 곧바로 실행한다.
  - `AsyncResult` 객체의 주요 필드와 메서드는 `.apply_async()`에서 설명한 것과 동일하게 사용할 수 있다.

  ```python
    result = my_task.delay(10, 20)  # 작업에 필요한 인자를 전달하여 비동기로 작업 실행
    print(result.status)  # 작업 상태 확인
    result_value = result.get(timeout=10)  # 작업 결과 가져오기
  ```

- **비교**:

  - `.apply_async()`는 정교한 예약이 가능하며, 특정한 작업 제어가 필요할 때 사용한다.
  - `.delay()`는 단순히 작업을 비동기로 호출하고 싶을 때 간편하게 사용한다.

#### 2. 기타 주요 메서드

- **`.retry()`**
  - 작업이 실패했을 때 자동으로 재시도할 수 있도록 하는 메서드이다. 예를 들어 네트워크 오류 등으로 인해 작업이 실패했을 때 일정 시간 후 재시도할 수 있다.
  ```python
  try:
      # 작업 수행 코드
  except Exception as exc:
      raise self.retry(exc=exc, countdown=60)  # 60초 후 재시도
  ```

- **`.get()`**
  - 작업의 결과를 동기적으로 가져오는 메서드로, 작업이 완료될 때까지 대기한다. 타임아웃을 설정하여 작업 완료를 기다리는 시간을 제한할 수 있다.
  ```python
  result_value = result.get(timeout=10)  # 10초 동안 작업 결과 기다리기
  ```

- **`.add_periodic_task()`**
  - Celery Beat와 함께 사용하여 주기적인 작업을 등록할 수 있는 메서드이다. 예를 들어 매일 특정 시간에 작업을 실행하고 싶을 때 유용하다.
  ```python
  from celery.schedules import crontab

  app.add_periodic_task(
      crontab(hour=7, minute=30, day_of_week=1),
      my_task.s(),
  )  # 매주 월요일 오전 7시 30분에 실행
  ```

#### 3. 예약된 작업 관리의 어려움과 대안

- `.apply_async()`와 `.delay()` 모두 예약된 작업을 Celery 워커에서 실행하도록 해준다. 하지만 예약된 작업들을 관리하는 것은 어려울 수 있다.
- 특히, **Flower**와 같은 도구를 사용하면 작업의 상태를 실시간으로 모니터링하고 관리할 수 있지만, **서버 재가동 시 예약된 작업 정보가 사라지는 문제**가 있다.
  - **서버 업데이트**나 **재시작**이 필요할 때, 이전의 예약된 작업들이 Flower에서 더 이상 보이지 않아서 추적이 어려워진다.

#### 3. 대안적인 작업 관리 방법

- **데이터베이스에 작업 정보 저장**

  - Django 모델을 사용하여 Celery 작업의 정보를 데이터베이스에 저장하고 관리할 수 있다.
  - 이 방식으로 작업의 상태를 직접 관리하고, 서버 재가동 후에도 Django Admin에서 작업을 추적할 수 있다.

  ```python
  class TaskLog(models.Model):
      task_id = models.CharField(max_length=255)
      task_name = models.CharField(max_length=100)
      status = models.CharField(max_length=50, default='PENDING')
      created_at = models.DateTimeField(auto_now_add=True)
      completed_at = models.DateTimeField(null=True, blank=True)
  ```

  - Celery 작업이 실행될 때마다 해당 모델을 업데이트하여 작업 상태를 관리한다.

- **Django Signals와 Celery 연동**

  - Django Signals를 사용하여 Celery 작업의 상태를 기록할 수 있다. 작업 실행 전후로 Signal을 통해 작업의 상태를 데이터베이스에 기록하여 관리할 수 있다.

  ```python
  @signals.task_success.connect
  def task_success_handler(sender=None, result=None, task_id=None, **kwargs):
      TaskLog.objects.filter(task_id=task_id).update(status='SUCCESS')
  ```

- \*\*`django-celery-results`\*\***와 ********`django-celery-beat`******** 사용**

  - `django-celery-results`를 통해 작업 결과를 Django 데이터베이스에 저장하고, Django Admin에서 관리할 수 있게 한다.
  - `django-celery-beat`를 사용하면 주기적인 작업을 Django Admin에서 직접 설정하고 관리할 수 있다.

#### 마무리

Celery 작업을 관리하고 추적하는 것은 개발 효율성을 높이는 중요한 요소다. `.apply_async()`와 `.delay()`를 적절히 사용하고, Django와의 통합을 통해 예약된 작업을 데이터베이스에서 관리하면 Flower의 한계를 극복할 수 있다. 이번에 배운 내용들을 잘 정리해두면 앞으로 Celery 작업 관리에서 유용하게 사용할 수 있을 것 같다.

