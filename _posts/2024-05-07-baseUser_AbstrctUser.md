---
layout: post
title:  "Django BaseUser 와 AbstractBaseUser"
date:   2024-05-07 20:47:00 +0900
categories: Django
tags: Django Usermodel
---

Dango 프로젝트에서 User Model을 만들 때 사용하게 되는, BaseUserMangager와 AbstactBaseUser의 개념을 알아봅시다. 

# 핵심 : BaseUserManager 는 사용자 생성 및 관리 로직을 담당하고,AbstractBaseUser 는 위에서 생성된 사용자 모델의 로직을 근간으로 필드를 만들고, 그에 대한 속성을 정의하는 역할을 합니다.

## 각각의 역할

**BaseUserManager** 는 어떻게 사용자를 생성하고 관리할지를 정의하는 클래스입니다.
사용자 생성, 인증, 암호 해시, 권한 부여 등의 로직을 담당합니다.
Django 기본 제공 AbstractBaseUserManager 클래스를 상속받아 커스텀 로직을 구현할 수 있습니다.
**AbstractBaseUser** 는 사용자 모델의 기본적인 뼈대를 제공하는 추상 클래스입니다.
사용자의 필수 필드 (예: 이메일, 이름, 비밀번호 등)를 정의합니다.
USERNAME_FIELD, REQUIRED_FIELDS 등의 속성을 통해 사용자 모델의 기본 동작을 설정합니다.
직접 사용하지 않고, 커스텀 사용자 모델을 만들 때 상속받아 사용합니다.

## 코드 예시에서:

CustomUserManager 는 BaseUserManager 를 상속받아 다음과 같은 기능을 구현합니다.

> - create_user: 일반 사용자 생성
> - create_superuser: 관리자 사용자 생성
> - normalize_email: 이메일 주소 정규화
> - set_password: 비밀번호 해시 설정   

CustomUser 는 AbstractBaseUser 를 상속받아 다음과 같은 필드와 속성을 정의합니다.

> - email: 사용자의 이메일 주소 (필수 필드)
> - first_name: 사용자의 이름 (필수 필드)
> - last_name: 사용자의 성 (필수 필드)
> - is_staff: 사용자가 관리자인지 여부
> - is_active: 사용자 계정이 활성화되었는지 여부
> - plan_type: 사용자의 요금제 유형 (추가 필드)
> - api_key: 사용자의 암호화된 API 키 (추가 필드)
> - usage_data: PAYG 사용자의 데이터 사용량 (추가 필드)

따라서, BaseUserManager 는 사용자 생성 및 관리 로직을 담당하고, AbstractBaseUser 는 위에서 생성된 사용자 모델의 필드 및 속성을 정의하는 역할을 한다고 볼 수 있습니다.

## 추가적으로:

AbstractBaseUser 는 django.contrib.auth.models 모듈에서 제공되는 추상 클래스이며, 이 클래스를 상속받아 커스텀 사용자 모델을 만들 때 기본적인 필드와 속성을 쉽게 추가할 수 있습니다.
BaseUserManager 는 AbstractBaseUser 와 함께 사용하며, 사용자 생성/관리 로직을 구현하는 데 사용됩니다.
필요한 경우 사용자 생성 시 추가적인 검증이나 데이터 처리를 수행할 수 있습니다.

## 추가2 : set_password를 통해 해싱된 password의 필드정의에 대하여

AbstractBaseUser를 상속받은 CustomUser 모델을 사용할 때, AbstractBaseUser에 이미 password 필드가 내장되어 있습니다. Django의 AbstractBaseUser 클래스는 사용자 인증을 처리할 수 있도록 설계되었으며, 기본적으로 password 필드를 포함하고 있습니다. 따라서 password 필드를 CustomUser 모델에 별도로 정의할 필요가 없습니다.

set_password 메서드는 이 내장된 password 필드에 대해 작동하며, 제공된 비밀번호를 해싱하여 저장합니다. 이 메서드는 AbstractBaseUser에서 제공되므로 CustomUser에서 별도로 구현할 필요가 없습니다. 사용자 생성 과정에서 set_password를 호출하면, 사용자의 평문 비밀번호는 안전하게 해싱되어 데이터베이스에 저장됩니다.

**결론적으로, CustomUser 모델에서 password 필드를 명시적으로 추가할 필요는 없습니다.**

