---
date: 2025-09-09
tags:
  - kanban
  - project
---
## Resource 
[wikidocs: Djnago REST Framework 에서의 인증과 권한부여](https://wikidocs.net/197565)

## 세밀한 접근 제어를 위해 DRF 권한 클래스 사용

- `IsAuthenticated` 인증된 사용자만 접근 허용
- `IsAdminUser` 관리자 사용자만 접근 허용
- `IsAuthenticatedOrReadOnly` 인증되지 않은 사용자에게는 읽기 전용 접근 허용, 인증된 사용자에게는 전체 접근 허용

이러한 권한 클래스를 사용하려면 DRF 설정의 `PERMISSION_CLASSES`에 추가 해야 함
```python
REST_FRAMEWORK = {
	'DEFALUT_PERMISSION_CLASSES': [
		'rest_framework.permissions.IsAuthenticated',
	],
	# 기타 DRF 설정
}
```

**view 수준에서 `.permission_classes` 속성을 사용하여 권한 지정 가능**


## 토큰 기반 인증을 구현하는 단계별 가이드

### 1단계: 필요한 Package 설치
```bash
pip install djangorestframework-simplejwt
```

### 2단계: Django settings 구성
```python
INSTALLED_APPS = [
	...
	'rest_framework_simplejwt',
	...
]

# 인증 백엔드 구성
REST_FRAMEWORK = {
	'DEFAULT_AUTHENTICATION_CLASSES': [
		'rest_framework_simplejwt.authentication.JWTAuthentication',
	],
}
```

### 3단계: JWT 토큰 생성
```python
from rest_framework_simplejwt.views import (
	TokenObtainPairView,
	TokenRefreshView,
)

urlpatterns = [
	...
	path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
	path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
	...
]
```

### 4단계: 사용자 인증을 위한 View 생성
```python
from rest_framework import generics
from rest_framework.response import Response
from rest_framework_simplejwt.tokens import RefreshToken
from .serializers import UserSerializer

class UserRegistrationView(generics.CreateAPIView):
	serializer_class = UserSerializer
	
	def post(self, request, *args, **kwargs):
		serializer = self.get_serializer(data=request.data)
		serializer.is_valid(raise_exception=True)
		user = serializer.validate_data
		
		refresh = RefreshToken.for_user(user)
		response = {
			'refresh': str(refresh).
			'access': str(refresh.access_token)
		}
		return Response(response)
```

### 5단계: 사용자 Serializer 생성
```python
from rest_framework import serializers
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
	password = serializers.CharField(write_only=True)
	
	def create(self, validated_data):
		user = User.objects.create(
			username=validated_data['username'],
			email=validated_data['email']
		)
		user.set_password(validated_data['password'])
		user.save()
		return user
		
	class Meta:
		model = User
		fields = ['username', 'email', 'password']
```

### 6단계: API 엔드포인트 보호
API 엔드포인트를 보호하기 위해 DRF의 `@api_view` 데코레이터와 `permission_classes` 속성을 사용할 수 있다
```python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def protected_view(request):
	# 보호된 뷰 로직을 여기 작성하세요
	return Response({'message': 'This is a protected View.'})
```

### 7단계: 엔드포인트 테스트
- 새 사용자 등록: 
  `POST` 요청을 `/api/register/`에 보내며, 요청 본문에 `username`, `email`, `password`를 포함 시킴
- Token 얻기: 
  `POST` 요청을 `/api/token/`에 보내며, 요청 본문에 `username`과 `password`를 포함 시킴
- 만료된 Token 재발급: 
  `POST` 요청을 `/api/token/refresh/`에 만료된 토큰을 요청 본문에 포함시켜 보냄
- 보호된 View 접근: 
  요청의 Authorization 헤더에 토큰을 포함시키면 됨. 예: `Authorization: Bearer <token>`