---
date: 2025-12-13
tags:
  - kanban
  - project
---
## Django의 기본 User 모델을 사용하지 않는 경우

다른 Field 구조를 갖기 위해  `AbstractBaseUser`를 상속할 수 있다
이 경우 사용자 생성 로직을 커스텀 매니저에서 정의해야 한다

## Django 인증 시스템에서 기대하는것
아래 메서드 들이 없으면 오류가 발생할 수 있다

- `create_user()`: 일반 사용자 생성
- `create_superuser()`: 관리자 생성

## CustomUserManager의 역할
