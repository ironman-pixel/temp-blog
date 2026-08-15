---
date: 2026-08-15
tags:
  - backend
  - authentication
  - email
  - django
  - python
---
```mermaid
sequenceDiagram
    actor User as 사용자
    participant SignupForm as 회원가입 폼
    participant Django as Django 서버
    participant DB as 데이터베이스
    participant NHN as NHN Cloud Email
    participant EmailLink as 이메일 링크

    User->>SignupForm: 회원정보 입력
    SignupForm->>Django: POST /accounts/signup/
    Django->>Django: 입력값 검증
    Django->>DB: INSERT USER<br/>(EMAIL_VERIFIED='N'<br/>APPROVAL_STATUS='PENDING')
    Django->>Django: UUID 토큰 생성
    Django->>NHN: POST 인증 이메일 발송
    NHN-->>User: 이메일 수신
    Django-->>SignupForm: "이메일 인증 링크 확인"
    
    User->>EmailLink: 인증 링크 클릭
    EmailLink->>Django: GET /accounts/verify-email/{token}/
    Django->>DB: SELECT USER WHERE token=?
    Django->>Django: 토큰 유효성 검증
    Django->>DB: UPDATE EMAIL_VERIFIED='Y'
    Django-->>User: "이메일 인증 완료<br/>관리자 승인 대기"
    
    User->>Django: 로그인 시도
    Django->>DB: SELECT USER
    Django->>Django: EMAIL_VERIFIED 확인
    Django->>Django: APPROVAL_STATUS 확인
    Django-->>User: 승인 대기 메시지
```



![[static/email-verification.svg]]


```
accounts/
├── email_utils.py          # 이메일 발송
│   ├── generate_verification_code()
│   ├── get_verification_code_template()
│   └── send_verification_code_email()
│
├── views.py                # API 엔드포인트
│   ├── send_email_verification_code()  # POST 코드 발송
│   ├── verify_email_code()             # POST 코드 확인
│   └── signup_view()                   # 세션에서 인증 확인
│
└── urls.py
    ├── /send-verification-code/
    └── /verify-code/

templates/pages/
└── signup.html             # 인증 UI + JavaScript

config/
└── settings.py             # NHN Cloud Email API 설정
```