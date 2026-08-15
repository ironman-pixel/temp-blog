---
date: 2025-08-29
tags:
  - kanban
  - project
---
## Dockerfile

```Dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt /app

RUN pip install --upgrade pip && \ 
	pip install -r requirements.txt
	
COPY . /app

# The enviroment variable ensures that the python output is set straight
# to the terminal with out buffering it first
ENV PYTHONBUFFERED=1

CMD ["uvicorn", "config.asgi:application", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```


### RUN 
- `pip install --upgrade pip`
  기본 이미지에 들어있는 pip 버전이 낮으면 최신 패키지 설치에서 에러가 나는 경우가 있어서 관례적으로 넣음

### ENV 
- `PYTHONUNBUFFERED=1`
  Python이 stdout/stderr을 버퍼링하지 않고 바로 출력하게 만드는 설정
- 보통 `ENV` 같은 런타임 관련 설정은 Dockerfile의 **아래쪽**에 두는 게 관례 (읽는 사람이 "최종 실행환경 설정은 여기구나" 하고 보기 좋으라고)

### CMD 
#### uvicorn
Django DRF를 ASGI 서버로 실행하기 위한 툴.  
(`python manage.py runserver`는 개발용 서버라 운영에서는 권장 X)

#### config.asgi:application
Django 프로젝트의 ASGI 엔트리포인트 (보통 `config/asgi.py` 안에 `application` 변수가 정의되어 있음)
→ DRF라 해도 Django 기반이니까 WSGI 대신 ASGI 서버를 띄우는 거지.

#### --workers 4
프로세스 개수 (병렬 요청 처리)
- 운영 환경에서는 적절한 워커 수(코어 수 기반)를 잡아줘야 함.
- 개발 환경이라면 굳이 필요 없음.

#### 실행 방식: 문자열 배열 vs 평문

- **exec form** (문자열 배열) **권장**
```
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```
Docker가 **PID 1 프로세스**를 바로 실행함 -> 쉘을 거치지 않음

- **shell form** (평문)
```
CMD python manage.py runserver 0.0.0.0:8000
```
내부적으로는 `/bin/sh -c "python manage.py runserver 0.0.0.0:8000"` 로 실행됨