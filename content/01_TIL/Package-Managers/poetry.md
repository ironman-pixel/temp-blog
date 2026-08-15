---
date: 2025-09-10
tags:
  - kanban
  - project
---
## Install
패키징 생태계를 더욱 예측 가능하고 간편하게 다룰 수 있는 가상환경 제공

```bash
$ pip install poetry
```

## Activate

가상환경 생성, 경로를 알려줌
```bash
$ poetry env activate
Creating virtualenv fastapi-test-bvaH2DfL-py3.9 in C:\Users\ybshi\AppData\Local\pypoetry\Cache\virtualenvs
'C:\Users\ybshi\AppData\Local\pypoetry\Cache\virtualenvs\fastapi-test-bvaH2DfL-py3.9\Scripts\activate'
```

아래 명령에서 가상환경 실행 가능
```bash
source C:/Users/ybshi/AppData/Local/pypoetry/Cache/virtualenvs/fastapi-test-bvaH2DfL-py3.9/Scripts/activate
```

## Package Manage

```bash
# 실행 시 poetry.lock 파일이 생성됨
$ poetry install
```

```bash
$ poetry add fastapi[standard]
```

## Runserver

```bash
$ uvicorn main:app --reload  # --port 8000
```

