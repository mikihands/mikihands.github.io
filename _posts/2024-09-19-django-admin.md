---
layout: post
title:  "`Django Admin 기능정리 "
date:   2024-09-19 22:15:00 +0900
categories: Django
tags: Django Django-Admin
---

### Django Admin 기능 정리

Django admin은 웹 애플리케이션을 위한 강력한 관리 도구로, 데이터베이스의 모델을 손쉽게 관리할 수 있는 기능을 제공한다. 기본적으로 Django는 모델을 정의하고 admin에 등록하는 것만으로도 다양한 CRUD(Create, Read, Update, Delete) 작업을 할 수 있게 해주며, 관리 화면을 다양한 방식으로 커스터마이징할 수 있다.

#### 1. **Admin 계정 사용법**

Django admin을 사용하기 위해서는 먼저 superuser 계정을 생성해야 한다. 다음 명령어를 통해 superuser를 생성할 수 있다:

```bash
python manage.py createsuperuser
```

이 명령어를 실행한 후, 사용자명, 이메일, 비밀번호를 설정하면 superuser가 생성된다. 생성된 계정으로 `/admin` 경로에서 로그인할 수 있으며, 로그인 후 등록된 모든 모델에 접근할 수 있다.

#### 2. **Admin에 모델 등록**

모델을 Django admin에 등록하는 방법은 매우 간단하다. `admin.py` 파일에 모델을 불러온 후 `admin.site.register()`를 사용해 등록하면 된다. 예를 들어, `CustomUser` 모델을 등록하는 코드는 다음과 같다:

```python
from django.contrib import admin
from .models import CustomUser

admin.site.register(CustomUser)
```

이렇게 하면 `CustomUser` 모델에 대한 기본 CRUD 기능이 admin에 자동으로 생성된다.

#### 3. **Admin 커스터마이징 방법**

기본 admin 기능 외에도 다양한 커스터마이징이 가능하다. 이를 통해 모델의 데이터를 더 효율적으로 관리할 수 있다.

- **목록 표시 필드 설정**: 기본적으로 모델의 `__str__()` 메서드에서 반환하는 값만 목록에 표시되지만, `list_display`를 사용하면 목록에 추가적인 필드를 표시할 수 있다.

  ```python
  class CustomUserAdmin(admin.ModelAdmin):
      list_display = ['username', 'email', 'date_joined', 'is_active']
  
  admin.site.register(CustomUser, CustomUserAdmin)
  ```

- **검색 기능 추가**: `search_fields`를 사용하면 모델의 특정 필드를 기준으로 검색할 수 있다.

  ```python
  class CustomUserAdmin(admin.ModelAdmin):
      search_fields = ['username', 'email']
  ```

- **필터 기능 추가**: `list_filter`를 통해 모델 데이터를 필터링할 수 있다.

  ```python
  class CustomUserAdmin(admin.ModelAdmin):
      list_filter = ['is_active', 'date_joined']
  ```

- **폼 커스터마이징**: `ModelForm`을 사용하여 admin에 표시되는 폼을 커스터마이징할 수 있다. 이를 통해 데이터 입력 시 더 복잡한 검증 로직을 추가하거나 UI를 조정할 수 있다.

  ```python
  from django import forms
  from .models import CustomUser
  
  class CustomUserForm(forms.ModelForm):
      class Meta:
          model = CustomUser
          fields = '__all__'
      
      def clean_username(self):
          data = self.cleaned_data['username']
          if "@" not in data:
              raise forms.ValidationError("올바른 이메일 형식이 아닙니다.")
          return data
  
  class CustomUserAdmin(admin.ModelAdmin):
      form = CustomUserForm
  
  admin.site.register(CustomUser, CustomUserAdmin)
  ```

#### 4. **Django Admin의 장점**

- **자동화된 CRUD 기능**: 모델을 정의하고 등록하기만 하면 자동으로 생성되는 CRUD 기능 덕분에 데이터 관리에 드는 노력이 크게 줄어든다.
- **권한 관리**: Django의 사용자 권한 시스템과 연동되어, 관리자나 일반 사용자에게 각기 다른 권한을 부여할 수 있다. 예를 들어, 특정 사용자 그룹만 특정 모델을 관리하도록 설정할 수 있다.
- **확장성**: `ModelAdmin`을 통해 필요한 기능을 쉽게 확장할 수 있으며, 커스텀 액션을 정의하여 데이터를 대량으로 처리하거나, 특정 조건에 맞는 데이터를 한 번에 수정하는 작업도 가능하다.
- **검색 및 필터 기능**: 간단한 설정으로 강력한 검색과 필터 기능을 제공하여 대량의 데이터를 효율적으로 관리할 수 있다.
- **모바일 최적화**: Django admin은 기본적으로 반응형 디자인을 지원하므로, 모바일 기기에서도 쉽게 접근하고 사용할 수 있다.

#### 5. **결론**

Django admin은 기본적인 데이터 관리 도구로는 물론, 복잡한 비즈니스 로직을 처리할 수 있는 강력한 도구로 활용할 수 있다. 기본 제공되는 기능뿐만 아니라, 다양한 커스터마이징 옵션을 통해 각 프로젝트에 맞는 관리 인터페이스를 구축할 수 있다. 특히, 데이터 관리와 권한 제어, 확장성 면에서 매우 유용하다.

이러한 이유로 Django admin을 적절히 커스터마이징하고 활용하면, 별도의 복잡한 DB 브라우저 없이도 프로젝트 전반의 데이터 관리와 운영을 효율적으로 할 수 있다.