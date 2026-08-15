---
date: 2025-12-13
tags:
  - kanban
  - project
---
settings에서 중요해보이는거 몇 가지만 정리

## INSTALLED_APPS
Django 프로젝트에서 활성화할 애플리케이션 목록

```python
INSTALLED_APPS = [
    "django.contrib.admin", # 관리자 인터페이스
    "django.contrib.auth", # 인증 시스템
    "django.contrib.contenttypes", # 콘텐츠 타입 프레임워크
    "django.contrib.sessions", # 세션 관리
    "django.contrib.messages", # 메시지 프레임워크
    "django.contrib.staticfiles", # 정적 파일 관리

    # 3rd party apps
    "django_apscheduler", # 스케줄링
    "rest_framework", # REST_API
    "corsheaders", # CORS 처리

    # apps
    'accounts', # 사용자 계정 관리
    'projectname.apps.ProjectnameConfig', # projectname 메인앱
]
```

## MIDDELWARE
미들웨어는 요청/응답 처리 과정에서 실행되는 컴포넌트 
순서가 중요

### 실행 순서
- 요청: 위에서 아래로 실행
- 응답: 아래에서 위로 실행

```python
MIDDLEWARE = [
    "corsheaders.middleware.CorsMiddleware", # CORS 처리
    "django.middleware.security.SecurityMiddleware", # 보안 헤더 설정
    "django.contrib.sessions.middleware.SessionMiddleware", # 세션 관리
    "django.middleware.locale.LocaleMiddleware", # 다국어 처리
    "django.middleware.common.CommonMiddleware", # 공통 기능(URL 정규화 등)
    "django.middleware.csrf.CsrfViewMiddleware", # CSRF 보호
    "django.contrib.auth.middleware.AuthenticationMiddleware", # 인증 정보 제공
    "accounts.middleware.JWTAuthenticationMiddleware", # JWT 인증 처리(커스텀)
    "django.contrib.messages.middleware.MessageMiddleware", # 메시지 프레임워크
    "django.middleware.clickjacking.XFrameOptionsMiddleware", # 클릭재킹 방지
]
```

## TEMPLATES
템플릿 엔진과 템플릿 관련 설정

```python
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [os.path.join(BASE_DIR, 'templates')],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
                "accounts.context_processors.user_menu_permissions",
            ],
        },
    },
]
```

### 주요 설정:
- BACKEND: 템플릿 엔진 지정 (Django 템플릿 사용)
- DIRS: 프로젝트 레벨 템플릿 디렉토리 (templates/)
- APP_DIRS: True이면 각 앱의 templates/도 검색
- OPTIONS.context_processors: 템플릿에 자동으로 제공되는 컨텍스트 변수
	- debug: 디버그 정보
	- request: 현재 요청 객체
	- auth: 인증 관련 변수 (예: user)
	- messages: 메시지 프레임워크 변수
	- user_menu_permissions: 커스텀 메뉴 권한 변수

## WSGI_APPLICATION
wsgi 서버가 django application을 로드하는 진입점 지정

```python
WSGI_APPLICATION = "config.wsgi.application"
```

## JWT_AUTH
JWT 쿠키 보안 설정

```python
# JWT Cookie configuration
JWT_AUTH_COOKIE = 'access_token'
JWT_AUTH_REFRESH_COOKIE = 'refresh_token'
JWT_AUTH_SECURE = not DEBUG  # HTTPS only in production
JWT_AUTH_HTTPONLY = True  # XSS protection
JWT_AUTH_SAMESITE = 'Lax'  # CSRF protection
```

- JWT_AUTH_HTTPONLY: True
	- 브라우저가 cookie를 HTTP 요청에만 포함, JavaScript(documents.cookie)로 접근 불가
	- XSS 공격으로 Cookie 탈취되는것 방지
- JWT_AUTH_SAMESITE:
	- 'Strict': 같은 사이트에서만 전송 (가장 엄격)
	- 'Lax': 같은 사이트 + 일부 안전한 크로스 사이트 요청(GET 등) 허용
	- 'None': 모든 크로스 사이트 요청 허용 (HTTPS 필수)

