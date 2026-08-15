---
date: 2025-09-24
tags:
  - kanban
  - project
---
## 1. OS `crontab` 직접 설정

### 장점
- **가장 표준적이고 안정적**
- **추가 라이브러리 불필요**: Django 앱 코드와 별개로 동작 -> 애플리케이션 장애와 무관하게 스케줄링 유지
- **유연성**: `docker exec`, `docker-compose run`, `python manage.py ...` 등 원하는 커맨드를 그대로 등록 가능

### 단점
- **애플리케이션 레벨과 분리**되어 관리해야 함 -> 인프라 쪽에서 설정 필요
- 컨테이너 환경이라면, `crontab`을 호스트에 둘지 컨테이너 내부에 둘지 설계가 필요


## 2. `django-crontab` 라이브러리 사용

### 장점
- **Django 설정**에 포함되므로, 배포 시 코드와 함께 스케줄링 정의가능
- `python manage.py crontab add/show/remove` 같은 명령어로 관리 -> 운영 편의성 있음

### 단점
- **실제로 내부적으로는 OS cron을 등록**하는 방식이다. 즉, `django-cron`은 cron 자체를 대체하는게 아니라 "cron 엔트리를 자동으로 생성/제거 하는 래퍼(Wrapper)"
- 따라서 **리눅스/유닉스 계열에서만 동작** 하는게 맞음. (Windows 개발 환경에서는 실행 불가)
- 컨테이너 환경에서는 ephermeral 특성 때문에 `crontab add`로 등록한 스케줄이 컨테이너 재시작 시 날아갈 수 있음. (-> entrypoint에서 매번 등록해주는 식으로 우회 가능)


## 3. Docker 환경 고려
- 운영 환경이 Docker라면 두 가지 접근 가능:
	1. 호스트 **OS의 crontab** 사용 -> `docker exec`로 컨테이너 안에서 명령 실행. (운영 환경에서 안정적)
	2. **별도 cron 전용 컨테이너**를 띄워서 주기적으로 `django manage.py`를 실행하게 함.(k8s cronjob 같은 패턴과 유사)
	3. `django-crontab`은 컨테이너 재시작/스케일링 문제로 잘 안쓰이는 편.


## ✅ 결론
- 배포 환경이 Docker라면 **OS cron** 또는 **별도 cron 컨테이너**를 쓰는 게 더 안정적이고 권장되는 방법이다.
- `django-crontab`은 개발/테스트나 단일 서버 환경에서는 편할 수 있지만, Docker 기반 프로덕션에는 적합하지 않음.
- "django-crontab은 리눅스에서만 동작한다" -> 사실이다.
  Windows 개발 환경에서는 직접 실행이 불가능 하고, 배포용으로도 실무에서는 잘 안쓰임.


## 👉 내 상황(개발: Windows, 배포: Linux Docker)이라면:
- 운영은 OS cron에 두는게 맞음.
- 개발 중에는 `python manage.py <commnad>`를 직접 실행해서 테스트 하고, 운영 배포시 `cron`을 Docker 외부(호스트나 별도 컨테이너)에서 관리하는걸 추천.