---
date: 2025-09-12
tags:
  - til
  - backend
  - django
  - celery
  - python
---
# ❓ Information
* Django에서 Celery 사용 방법

---

# 🔰 Content ->  

## Install 
```bash
pip install celery redis
```

`myProject/config/celery.py`
```python
import os
from celery import Celery

# 1. Django settings 모듈 지정
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")

# 2. Celery 앱 생성
app = Celery("config")

# 3. Django settings 에서 CELERY_ 접두사로된 설정 읽기
app.config_from_object("django.conf:settings", namespace="CELERY")

# 4. Django 앱별 task.py 자동 발견
app.autodiscover_tasks()

```

`settings.py`
```python
CELERY_BROKER_URL = "reids://localhost:6379/0"
```

## Task 정의

```python
from celery import shared_task
from django.db import transaction

from .models import JobImage


@shared_task
def assign_job(member_id):
    # 트랜잭션 시작
    with transaction.atomic():
        # 아직 할당되지 않은 작업 하나 가져오기 + 잠금
        job = (
            JobImage.objects.select_for_update(skip_locked=True)
            .filter(member__isnull=True)
            .order_by("created_at")
            .first()
        )
        if job:
            # 해당 row 에 member_id 할당
            job.member = member_id
            job.save()
            return job.id
        return None
```


## Task 호출

아래 코드의 작업은 짧고 즉시 끝나는 트랜잭션이다:
- DB에서 아직 할당되지 않은 `JobImage`를 하나 잡아서 `select_for_update(skip_locked=True)`
- 현재 `member_id`에 할당하고
- 그 PK(`job_id`)를 바로 리턴한다.

이럴때는 Celery가 사실은 필요 없다.


```python
# views.py
from rest_framework import status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response

from .serializers import JobImageSerializer
from .tasks import assign_job


@api_view(["POST"])
@permission_classes([IsAuthenticated])
def get_job(request):
    """작업 할당"""
    member_id = request.user.id

	try:
        # Celery 비동기 호출
        task = assign_job.delay(member_id)

        if task.ready():
            job_id = task.result

            if job_id:
                serializer = JobImageSerializer(job_id)
                return Response(status=status.HTTP_200_OK)
            else:
                return Response(status=status.HTTP_204_NO_CONTENT)

    except Exception as e:
        print(f"작업 할당 실패: {e}")
        return Response(status=status.HTTP_400_BAD_REQUEST)

```

