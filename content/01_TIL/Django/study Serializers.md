---
date: 2025-06-28
tags:
  - kanban
  - project
---
# DRF(Django Rest Framework)


# Serializer

## 직렬화 기능
>python data를 JSON 타입의 데이터로 변환:
>	queryset, model instance 등을 JSON, XML 등의 컨텐트 타입으로 쉽게 변환 가능한 python datatype으로 변환시킨다.

## 작성

1. Django model에 대한 serializer를 작성하기 위해서는,
   DRF 의 `serializers.Serializer` 또는 `serializers.ModelSerializer`를 상속 받은 새로운 클래스를 정의해야함.
   _Meta 클래스 내부에서 직렬화에 포함시키고자 하는 모델과 필드를 지정함._
```
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
	class Meta:
		model: Book
		fields = ['title', 'auther', 'publication_date', 'price']
```

2. 특정 serializer class 안에서 다른 app의 serializer 를 사용하여 `related_name`으로 변수를 정의하는 경우는,
   중첩된 객체 구조로 응답을 구성하기 위해서이다.
```
from rest_framework import serializers
from book.models import Book
from job.serializers import JobSerializer

class BookSerializer(serializers.ModelSerializer):
	job2book = JobSerializer(many=True, read_only=True)

	class Meta:
		model: Book
		fields = "__all__"
```
_위와 같이 정의한 경우 아래와 같은 계층 구조의 응답이 만들어진다._
```
{
	"title": "story_one",
	"job2book": [
		{
			"id": 1,
			"job_status": "Complete",
			...
		},
		...
	],
	...
}
```

## 사용

1. serializer를 사용하기 위해, 해당 serializer의 instance를 생성하고 `model instance`나 `queryset`을 데이터 속성에 전달하면 됨.
   _아래는 pk=1 인 Book의 instance를 가져와서 BookSerializer class의 instance를 생성하고, 직렬화된 데이터를 출력함._
```
book = Book.objects.get(pk=1)
serializer = BookSerializer(book)
print(serializer.data)
```


# serializers.CharField

문자열 필드 정의 도구.
models.Charfield와는 다른 개념으로 요청/응답 처리용.

### 사용목적
- 클라이언트가 보내는 요청(body)의 특정 필드를 받기 위해 선언
- 자동으로 유효성 검사까지 처리해줌

### 예시로 다시 보면
```
class RegisterSerializer(serializers.ModelSerializer):
	password = serializers.CharField(
		write_only=True,
		required=True,
		validators=[validate_password]
	)
	password2 = serializers.charField(write_only=True, required=True)
```

여기서 `password`, `password2`는 모델에는 없는 필드인데, 클라이언트로부터 받기 위해 serializer 에서만 임시로 선언한것이다.

### 실전 흐름 정리
```
password = serializers.CharField(...)
```
1. 클라이언트가 JSON 으로 `password` 보내면
2. DRF가 이걸 `.CharField()` 에 넣어 자동 검증(필수 여부, 길이 , validator 등)
3. `.validator()` 메서드에서 추가 비교 (`password == password2`)
4. `.create()`에서 실제 모델 객체에 저장

