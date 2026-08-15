---
date: 2025-06-28
tags:
  - kanban
  - project
---
>[vindevs: how to use django field choices with code examples](https://vindevs.com/blog/how-to-use-django-field-choices-with-code-examples-p60/)

## 상황에 따른 choices 사용 구분
- Django 모델의 `choices`에 쓸 거면 → `models.TextChoices` ✅
- 내부 로직에서만 쓸 상수면 → `enum.Enum` 또는 `StrEnum` ✅
- Python 3.11 이상이라면 `StrEnum`도 고려 가능 (문자열 비교 안전성 증가)

## Model 의 Choices 에서 사용되는 예시

- Model 정의 시에 특정 Field가 정해진 옵션들만 가질 수 있는 경우 (job_status)
  `choices` 에 해당 옵션들을 넣게 된다.
```
from django.db import models
from common.choices import JobStatus

class Job(models.Model):
	file = models.ForeignKey('file.File', null=False, on_delete=modles.CASCADE, verbose_name='파일 ID', related_name='job2file')
	...
	job_status = models.CharField(choices=JobStaus.choices, max_length=4, default=JobStatus.Initial, verbose_name='작업 상태')
```

- common 앱 내부에 choices.py 파일을 만들어서 다음과 같이 `model.TextChoices`를 추가 할 수 있다.
  _`choices.py`로 구조분리 함으로써 가독성 높이기_
```
from django.db import models

class JobStatus(modles.TextChoices):
	Initial = 'JS01', '초기'
	Complete = 'JS02', '완료'
	Reject = 'JS03', '반려'
```

- 이걸 다음과 같이 바로 필드에 쓸 수 있음:
```
status = modles.CharField(
	max_length=4,
	choices=JobStatus.choices,
	default=JobStatus.Initial,
)
```

## 결론
- `CharField`나 `TextField`에서 `choices=` 옵션에 연결할 값이면 **무조건 `models.TextChoices`**.

