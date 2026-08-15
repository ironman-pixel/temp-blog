---
date: 2025-09-24
tags:
  - kanban
  - project
---
## DRF 내장 페이지네이션 클래스 구성하기

### 1. PageNumberPagination(가장 많이 사용)
데이터를 **일정 크기의 페이지로 나누고**
클라이언트가 **특정 페이지를 요청할 수 있게** 함
- URL에 `?page=2` 처럼 쿼리 파라미터로 페이지 이동
- 기본 page_size = 10, settings에서 조정 가능
- 예:
```python
class UserListView(generics.ListAPIView):
	queryset = User.objects.all()
	serializer_class = UserSerializer
	pagination_class = PageNumberPagination
```
- 결과: `{count, next, previous, results}`

### 2. LimitOffsetPagination
클라이언트가 반환할 항목 수 와 데이터 컬렉션 내에서 
시작 지점을 지정
- `?limit=20&offset=40`형태
- 무한 스크롤/로드 모어에 유용
### 3. CursorPagination
데이터 세트에 대해 더 효율적인 
커서 기반의 페이지네이션 시스템 제공
- `id` 기반 커서 방식, 정렬 기반으로 안정적인 무한 스크롤 가능
- 대규모 데이터셋에 적합

_페이지네이션 클래스 사용을 위해 DRF 설정의 DEFAULT_PAGINATION_CLASS 에 추가하고 페이지 크기 지정 필요_
```python
REST_FRAMEWORK = {
	'DEFAULT_PAGINATION_CALSS': 'rest_framework.pagination.PageNumberPagination',
	'PAGE_SIZE': 10,
	...
}
```


### 보안 민감 필터 사용시 처리 방식
_관행적으로는 POST body에 filter, URL query에 pagination을 섰어서 쓰는 경우가 많다._
```swift
POST /api/users/search/?page=2&page_size=20
{
	"department": "financs",
	"active": true
}
```
**이유는**:
- URL Query Params로 page/page_size를 쓰면 → **PageNumberPagination 그대로 사용 가능**
- Pagination도 body 안에 넣고 싶다면 → **커스텀 페이지네이터 작성 필요**

**추천 설계**:
1. **민감 필터 값**은 body에 담아서 POST 요청으로 전달
    - `/api/users/search/`
    - body = `{filters...}`
2. **페이징 정보**는 URL query로 두고 PageNumberPagination 그대로 활용
    - `/api/users/search/?page=2&page_size=20`
3. **응답 구조**는 DRF 표준 유지
```json
{
  "count": 123,
  "next": "http://.../search/?page=3&page_size=20",
  "previous": "http://.../search/?page=1&page_size=20",
  "results": [...]
}
```


---
# 사용자 정의 페이지네이션 체계 구현

`rest_framework.pagination.BasePagination` 를 상속하고 `.paginate_queryset(self, queryset, request, view)`와 `.get_paginated_response(self, data)` 메서드 구현이 필요

_DRF 의 `Pagination` 을 서브클래싱 하여 직접 페이지네이션 클래스를 생성할 수 있음_
```python
from rest_framework.pagination import PageNumberPagination

class CustomPagination(PageNumberPagination):
	page_size = 20
	page_size_query_param = 'page_size'
	max_page_size = 100
```
사용자 정의 페이지네이션 클래스 `CustomPagination`
클라이언트가 `page_size` 쿼리 파라미터를 제공하여 페이지 크기를 변경할 수 있음
`max_page_size` 속성은 최대 페이지 크기를 100으로 제한


## DRF의 내장 필터링과 정렬 기능 사용

API 요청에서 ordering 쿼리 파라미터를 사용하여 모델의 어떤 필드로든 정렬 가능
name, price 필드가 있는 Product 모델에서는 `GET /products?ordering=price` 와 같은 요청으로 price를 오름차순 정렬 가능

```python
from rest_framework import filters

class MyModelViewSet(viewsets.ModelViewsSet):
	queryset = Mymodel.objects.all()
	serializer_class = MyModelSerializer
	filter_backends = [filters.SearchFilter, filters.OrderingFilter]
	search_fields = ['name', 'description']
	ordering_fields = ['name', 'created_at']
```

`SearchFilter` -> 지정된 필드를 기준으로 검색을 수행
`OrderingFilter` -> 지정된 필드를 기준으로 정렬 수행


## 사용자 정의 필터링과 정렬 기능 추가

`rest_framework.filters.BaseFilterBackend` 를 상속하고 `.filter_queryset(self, request, queryset, view)` 메서드를 구현하여 사용자 정의 필터 백엔드를 만들 수 있다

```python
from rest_framework import filters

class CustomFilterBackend(filters.BaseFilterBackend):
	def filter_queryset(self, request, queryset, view):
		return queryset
```


# 예제 코드

## 1️⃣ 페이지네이션 클래스 구성

`settings.py` 파일에 설정 추가
```python
REST_FRAMEWORK = {
	'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination', 
	'PAGE_SIZE': 10, 
	... 
}
```


## 2️⃣ 뷰셋에서 페이지네이션 활성화

`view.py` 파일에 `pagination_class` 속성 설정하여 
뷰셋에 페이지네이션 클래스 추가
```python
from rest_framework.pagination import PageNumberPagination

class BookViewSet(viewsets.ModelViewSet):
	...
	pagination_class = PageNumberPagination
	...
```
`pagination_class`를 `PageNumberPagination`으로 설정함으로써 `BookViewSet`에서 페이지네이션을 활성화함
`pagination_class`는 해당 뷰셋에만 페이지네이션을 적용하거나 특정 뷰나 API 엔드포인트에만 페이지네이션을 적용할 수도 있음


## 3️⃣ 필터링과 정렬 활성화

`views.py` 에서 `filter_backends` 속성 설정하여
필터링 및 정렬 기능을 뷰셋에 추가
```python
from rest_framework.filters import SearchFilter, OrderingFilter

class BookViewSet(viewsets.ModelViewSet):
	...
	filter_backends = [SearchFilter, OrderingFilter]
	search_fields = ['title', 'author']
	ordering_fields = ['publication_date', 'price']
	...
```


## 4️⃣ 사용자 정의 필터링 및 정렬 구현

`views.py` 에서 
사용자 정의 필터링 및 정렬 로직 구현

```python
from rest_framework import viewsets
from rest_framework.filters import SearchFilter, OrderingFilter
from .models import Book
from .serializers import BookSerializer

class BookViewSet(viewsets.ModelViewSet):
	queryset = Book.objects.all()
	serializer_class = BookSerializer
	pagination_class = PageNumberPagination
	filter_backends = [SearchFilter, OrderingFilter]
	search_fields = ['title', 'author']
	ordering_fields = ['publication_date', 'price']
	
	def get_queryset(self):
		queryset = Book.objects.all()
		
		# 요청에서 필터링 매개변수 가져오기
		title = self.request.query_params.get('title', None)
		author = self.request.query_params.get('author', None)
		
		# 필터링 적용
		if title:
			queryset = queryset.filter(title__icontains=title)
		if author:
			queryset = queryset.filter(author__icontains=author)
		
		# 정렬 적용
		ordering = self.request.query_params.get('ordering', None)
		if ordering in self.ordering_fileds:
			queryset = queryset.order_by(ordering)
		
		return queryset
```

