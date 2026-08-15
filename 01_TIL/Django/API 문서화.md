---
date: 2025-09-10
tags:
  - kanban
  - project
---
자동으로 API 문서를 생성하는 Django 앱
serializers, viewsets을 기반으로 문서를 생성함

**OpenAPI 사양**(이전에는 Swagger로 알려진)을 사용하여 구조화된 형식으로 API 문서를 생성함


## DRF Docs
### Install

```bash
pip install drf-yasg
```

`settings.py`
```python
INSTALLED_APPS = [
	...
	'drf_yasg',
	...
]
```

`urls.py`
```python
...
from django.urls import re_path
from rest_framework import permissions
from drf_yasg.views import get_schema_view
from def_yasg import openapi

...

schema_view = get_schema_view(
	openapi.Info(
		title="Snippets API",
		default_version='v1',
		description="Test description",
		terms_of_service="https://www.google.com/policies/terms/",
		contact=openapi.Contact(email="contact@snippets.local"),
		license=openapi.License(name="BSD License"),
	),
	public=True,
	permission_classes=(permissions.AllowAny,),
)

urlpatterns = [
	path('swagger<format>/', schema_view.without_ui(cache_timeout=0), name='schema-json'),
	path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui')
	path('redoc/', schema_view.with_ui('redoc', cache_timeout=0), name='schema-redoc'),
	...
]
```


## DRF 뷰와 엔드포인트에 대한 테스트 작성 실행

테스트를 작성하려면 일반적으로 앱 디렉터리에 `test.py`  파일 생성
`django.test.TestCast` 를 상속하는 새 클래스를 정의 하고 이 클래스 내에서 실행할 각 테스트에 대한 메서드 정의

```python
from django.test import TestCase
from rest_framework import status
from rest_framework.test import APIClient
from .models import Book
from .serializers import BookSerializer

class UserListViewTest(TestCase):
	def setUp(self):
		self.client = APIClient()
		self.book_data = {
			'title': 'Test Book',
			'author': 'Jhon Doe',
			'publication_date': '2023-01-01',
			'price': '9.99',
		}
		self.book = Book.obejcts.create(**self.book_data)
	
	def test_get_all_books(self):
		response = self.client.get('/myapp/books/')
		books = Book.objects.all()
		serializer = BookSerializer(books, many=True)
		self.assertEqual(response.status_code, stauts.HTTP_200_OK)
		self.assertEqual(response.data['results'], serializer.data)
	
	def test_get_single_book(self):
		response = self.client.get('/myapp/books/')
		books = Book.objects.all()
		serializer = BookSerializer(books, many=True)
		self.assertEqual(response.status_code, stauts.HTTP_200_OK)
		self.assertEqual(response.data['results'], serailizer.data)  # Check 'results' key
	
	def test_create_book(self):
		response = self.client.post('/myapp/create/', data=self.book_data, format='json')
		self.assertEqual(response.status_code, status.HTTP_201_CREATED)
		self.assertEqual(Book.objects.count(), 2)
	
	def test_update_book(self):
		updated_data = {
			'title': 'Updated Book',
			'author': 'Jane Smith',
			'publication_date': '2023-02-01',
			'price': '19.99',  # Updated to a string value
		}
		response = self.client.put(f'/myapp/books/{self.book.id}/', data=updated_data, format='json')
		self.assertEqual(response.status_code, status.HTTP_200_OK)
		updated_book = Book.objects.get(id=self.book.id)
		self.assertEqual(updated_book.title, updated_data['title'])
		self.assertEqual(updated_book.author, updated_data['author'])
		self.assertEqual(updated_book.publication_date.strftime('%Y-%M-%D'), updated_data['publication_date'])
		self.assertEqual(str(updated_book.price), updated_data['price'])  # Convert Decimal to string
	
	def test_delete_book(self):
		response = self.client.delete(f'/myapp/books/{self.book.id}/')
		self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
		self.assertEqual(Book.objects.count(), 0)
```


테스트 작성 후에는 Django의 테스트 관리 명령을 사용하여 테스트 실행 가능
```bash
python manage.py test
```

