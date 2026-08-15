---
date: 2025-06-23
tags:
  - kanban
  - project
---
# Generics

## Generic View

### 동작 순서

```
class RegisterView(generics.CreateAPIView):
	queryset = Member.objects.all()
	permission_classes = (AllowAny,)
	serializer_class = RegisterSerializer
```

### RegisterSerializer 자동 호출 구조
1. 클라이언트가 `POST /member/register/` 로 요청을 보냄
2. `CreateAPIView`는 내부적으로 `.post()` -> `.create()` -> `serializer.is_valid()` -> `serializer.save()` 흐름을 자동 실행함
3. 이때 다음 순서로 `RegisterSerializer`가 호출됨:

### 호출 흐름 상세
1. `serializer = RegisterSerializer(data=request.data)`
   -> 요청 데이터(JSON)를 serializer에 전달
2. `serializer.is_valid`
   -> 내부적으로 `validate()` 호출됨
```
def validate(self, attrs):
	if attrs['password'] != attrs['password2']:
		raise serializers.ValidationError({"password":"Password fields didn't match"})
	return attrs
```
3. `serializer.save()`
   -> 내부적으로 `create(validated_data)` 호출됨
```
def create(self, validated_data):
	member = Member.objects.create(
		username=validated_data['username'],
		email=validated_data['email'],
	)
	member.set_password(validated_data['password'])
	member.save()
	return member
```

### 결론
`RegisterSerializer` 안의 `validate()` 와 `create()`는 직접 호출하지 않아도, `CreateAPIView`의 기본 흐름 안에서 자동 실행됨. 
이것이 바로 DRF의 핵심적인 **"자동처리"** 철합임.

## genetics.CreateAPIView

새로운 리소스를 생성하는데 특화된 APIView

### 하는 일
- HTTP POST 요청만 처리함
- `.create()` 메서드를 자동 호출함 (내부적으로 serializer를 통해 객체 생성)
- 예: 사용자 등록, 글 작성, 댓글 생성 등 "추가(create)" 동작에 딱 맞음

### 내부적으로 하는 일 
```
def post(self, request, *args, **kwargs):
	return self.create(request, *args, **kwargs)
```

### 정리: DRF: CreateAPIView 기본구조
```
from rest_framework import generics
from myapp.models import User
from myapp.serializers import UserSerializer

class UserCreateView(generics.CreateAPIView):
	queryset = User.objects.all()
	serializer_class = UserSerializer
```

