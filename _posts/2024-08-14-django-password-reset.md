---
layout: post
title:  "Django의 auth_views에서 제공하는 PasswordReset 기능: 사용 방법과 커스터마이징"
date:   2024-08-18 21:07:00 +0900
categories: Django
tags: Django Python
---

## Django의 auth_views에서 제공하는 PasswordReset 기능: 사용 방법과 커스터마이징

Django는 강력하고 보안성이 높은 사용자 인증 시스템을 내장하고 있습니다. 그중에서도 비밀번호 재설정(Password Reset) 기능은 매우 유용하고, 실제로 많은 웹 애플리케이션에서 필수적인 기능입니다. 이 글에서는 Django의 auth_views에서 제공하는 일련의 PasswordReset 관련 메서드들이 무엇인지, 언제 사용하는지, 그리고 이를 어떻게 커스터마이징할 수 있는지에 대해 설명하겠습니다.

### 1. Django의 Password Reset 기능이란?
비밀번호 재설정 기능은 사용자가 비밀번호를 잊었을 때 이메일을 통해 비밀번호를 재설정할 수 있도록 도와주는 기능입니다. Django는 이를 위해 몇 가지 내장된 뷰를 제공합니다:

- PasswordResetView: 사용자가 이메일을 입력하여 비밀번호 재설정 요청을 할 수 있는 페이지를 렌더링합니다.
- PasswordResetDoneView: 비밀번호 재설정 이메일이 성공적으로 발송된 후 보여주는 페이지입니다.
- PasswordResetConfirmView: 사용자가 이메일로 받은 링크를 클릭하면, 새로운 비밀번호를 설정할 수 있는 페이지를 렌더링합니다.
- PasswordResetCompleteView: 비밀번호가 성공적으로 재설정된 후 보여주는 페이지입니다.

이 모든 뷰는 Django의 auth_views 모듈에 내장되어 있어, 복잡한 구현 없이 간단하게 비밀번호 재설정 기능을 사용할 수 있습니다.

### 2. 언제 사용하고, 얼마나 편리한가?
Django의 비밀번호 재설정 기능은 사용자 관리가 필요한 대부분의 웹 애플리케이션에서 필수적입니다. 예를 들어, 사용자가 자신의 계정에 접근할 수 없을 때, 비밀번호 재설정 링크를 통해 간편하게 비밀번호를 다시 설정할 수 있습니다.

이 기능은 특히 다음과 같은 점에서 편리합니다:

- **내장 템플릿**: Django는 기본적으로 필요한 모든 템플릿을 제공하므로, 최소한의 설정만으로도 기능을 활성화할 수 있습니다.
- **보안성**: Django는 사용자와 토큰을 기반으로 비밀번호 재설정 링크를 생성합니다. 이 링크는 단 한 번만 사용할 수 있고, 일정 시간이 지나면 만료되므로 보안성이 뛰어납니다.
- **유연성**: 각 뷰는 커스터마이징이 가능하며, 프로젝트의 요구에 맞게 템플릿이나 동작을 쉽게 수정할 수 있습니다.

### 3. 보안성은 어떠한가?
Django는 보안에 대해 매우 신경을 씁니다. 비밀번호 재설정 기능 역시 강력한 보안 메커니즘을 갖추고 있습니다:

- **토큰 기반 인증**: 비밀번호 재설정 링크는 토큰을 사용하여 사용자 계정을 보호합니다. 이 토큰은 한 번 사용되면 즉시 무효화되며, 유효 기간이 지나면 만료됩니다.
- **암호화된 사용자 ID**: 비밀번호 재설정 과정에서 사용자의 ID는 Base64로 인코딩되어 안전하게 전송됩니다.
- **CSRF 보호**: Django는 모든 폼 요청에 대해 CSRF 토큰을 사용하여 중간자 공격을 방지합니다.

이러한 기능들은 기본적으로 Django의 비밀번호 재설정 시스템에 내장되어 있으며, 추가적인 보안 설정을 통해 더 강화할 수 있습니다.

### 4. 커스터마이징: 네임스페이스 문제 해결하기

개발 중에는 프로젝트 구조상 URL 네임스페이스를 사용하는 경우가 많습니다. **Django의 내장 템플릿이나 뷰는 기본적으로 네임스페이스를 고려하지 않기 때문에, 네임스페이스가 있는 프로젝트에서는 NoReverseMatch 오류가 발생할 수 있습니다.**

이 문제를 해결하기 위해서는 다음과 같은 방법으로 커스터마이징할 수 있습니다:

#### 4.1. 템플릿의 커스터마이징

기본적으로 Django에서 제공하는 템플릿이 있으나, 사용자 경험을 높이고 제공하는 서비스 어플리케이션과의 통일된 디자인을 위해서 템플릿을 커스터마이징 하는 것을 추천합니다. 이 때 경로가 중요한데, 
`templates/your_app/registration/` 디렉토리 안에 작성해줘야 합니다. 

작성해줘야 하는 템플릿은 다음과 같다. 

    1. 사용자의 이메일을 입력받는 페이지 
    2. 사용자에게 발송될 이메일양식.html
    3. 사용자가 이메일의 임시링크를 타고 연결되는 페이지(password재설정 페이지)
    4. password reset이 성공적으로 완료되었을 때 렌더되는 페이지

이렇게 4개의 html을 작성해주면 된다. 

#### 4.2. URL patterns 의 커스터마이징

**예시** : 앱에  `name_dpace1`라는 name space를 설정한 경우, 각각의 내장함수에 위에서 제작한 템플릿으로 제대로 리디렉션 되도록 설정을 해줄 수 있습니다. 이때 reverse_lazy는 사용합니다. reverse_lazy는 Django의 URL을 지연(reverse) 로딩하여 줍니다. PasswordResetView가 처음 로드될 때 바로 URL을 생성하지 않고, 실제로 URL이 필요할 때(POST 요청 후)에 생성합니다. 이 방식은 URL 패턴이 로드되기 전에 참조해야 할 경우 유용합니다.

특히, 내장함수를 호출할 때 옵션을 걸어줘야하는 함수는 `PasswordResetView` 함수와 `PasswordResetConfirmView` 함수이므로 이 둘에 대해 다음과 같이 리디렉션될 템플릿 페이지 경로를 잘 작성을 해줘야 합니다. 

```python
from django.urls import path
from django.contrib.auth import views as auth_views
from django.urls import reverse_lazy

app_name = 'name_space1'

urlpatterns = [
    # 기타 여러 url patters 들 
    path('password_reset/', auth_views.PasswordResetView.as_view(template_name='your_app/registration/password_reset_form.html', 
                                                                 email_template_name='your_app/registration/password_reset_email.html',
                                                                 success_url=reverse_lazy('name_space1:password_reset_done'),), 
                                                                 name='password_reset'),
    path('password_reset/done/', auth_views.PasswordResetDoneView.as_view(template_name='your_app/registration/password_reset_done.html'), name='password_reset_done'),
    path('reset/<uidb64>/<token>/', auth_views.PasswordResetConfirmView.as_view(template_name='your_app/registration/password_reset_confirm.html',
                                                                                success_url=reverse_lazy('name_space1:password_reset_complete'),), 
                                                                                name='password_reset_confirm'),
    path('reset/done/', auth_views.PasswordResetCompleteView.as_view(template_name='your_app/registration/password_reset_complete.html'), name='password_reset_complete'),
    
    # 이하생략
] 
```

### 포스팅을 마치며:

Django는 정말 매력적인 프레임워크이지만 한국내에서는 그리 사용하는 개발자가 없는 듯 합니다.
하나하나 문제가 생길 때마다 물어볼 곳도 마땅치 않고 오로지 이해가 안되도 수십번을 Django Doc를 검색하여 읽고 또 읽고 또 읽고, 
gpt에게 물어보고, 책을 찾아보고 어렵게 어렵게 한걸음씩 배우고 있습니다. 

이번 네임스페이스로 오류가 났을 때도 처음에는 상당히 난감했습니다. 내장함수를 찾아서 직접 경로를 수정해줘야 하나 생각할 정도 였습니다. 

하지만 언제나처럼 Django는 천재 개발자들이 이런 상황을 예측해서 준비해 놓은 장치가 있고, 그 방법을 혼자 찾아내서 도달하는 것은 늘 어렵고 도전적인 일입니다만,
해결해내고 나면 언제나 기분이 좋고 성공했을 때의 희열을 느낍니다. 이런 맛에 코딩을 하는 것 같습니다. 

그리고 언제나 이렇게 오픈소스로 훌륭한 프레임워크를 제공해주는 천재들에게 감사하는 마음과 경의로운 마음을 가지게 됩니다. 


