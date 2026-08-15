---
date: 2025-09-24
tags:
  - kanban
  - project
---
## AS IS
Django DRF + React axios zustand 프로젝트에서 내가 사용한 Django 응답 구조는 다음과 같다

```python
Response(
	dict(
		success=False,
		message="실패...",
		errors=serializer.errors,
	),
	status=status.HTTP_400_BAD_REQUEST,
)
```

- pros:
	- `success`, `message`, `errors`가 한눈에 보여서 React 쪽에서 처리하기 편함
- cons:
	- DRF의 기본 pagination, exception, handler, status 코드 일관성과 약간 충돌
	- pagination이 자동으로 적용되지 않음
	- DRF의 다른 기능(`ListAPIView`, `ModleViewSet`등)과 통일되지 않음

즉 **"일관된 DRF 방식" 과 다르게 수동으로 Response를 포장했기 때문에 장단점이 섞여있음**


## TO BE

RESTful DRF 방식

**성공응답**:
```python
# ViewSet / GenericAPIview
return Response(serializer.data) # status 기본 200
```

**Validation Error**: DRF가 자동으로 처리
```json
{
	"username": ["This field is required."]
}
```

**Pagination**:
```python
class UserListView(generics.ListAPIView):
	queryset = User.objects.all()
	serializer_class = UserSerializer
	pagination_class = PageNumberPagination
```
-> 자동으로
```json
{
	"count": 123,
	"next": "http://.../page=2",
	"previous": null,
	"results": [...]
}
```

**특징**:
- 별도의 `success`나 `message` 키 없이 **HTTP 상태 코드**만으로 성공/실패 판단 가능
- pagination, filtering, ordering 등이 기본 기능으로 자동 적용
- React 쪽에서 `res.status`와 `res.data`로 쉽게 처리 가능


---


### DRF View & Pagination

```python
# views.py
from rest_framework import generics
from rest_framework.pagination import PageNumberPagination
from .models import User
from .serializers import UserSerializer

class StandardResultsSetPagination(PageNumberPagination):
	page_size = 10
	page_size_query_param = 'page_size'
	max_page_size = 100
	
class UserListView(generics.ListAPIView):
	queryset = user.objects.all()
	serializer_class = UserSerializer
	pagination_class = StandardResultsSetPagination
```

- 이 구조대로 하면 DRF가 자동으로 `count`, `next`, `previous`, `results` 형태의 pagination JSON을 만들어줌
- Validation Error도 serializer에서 자동으로 처리됨


### Axios 설정 (interceptor)

```ts
// apiClient.ts
import axios from 'axios';

const api = axios.create({
	baseURL: "/api",
	withCredentials: true,
});

api.interceptors.response.use(
	(response) => response, // 성공 응답은 그대로
	(error) => {
		// 에러 공통 처리
		const data = error.response?.data;
		const status = error.response?.status;
		
		// 예: 401 Unauthorized 처리
		if (status === 401) {
			console.log("로그인 필요");
		}
		
		return Promise.reject(error);
	}
);

export default api;
```


### Zustand Store

```ts
// useUserStore.ts
import { create } from "zustand";
import api from "./apiClient";

interface Pagination {
	count: number;
	next: string | null;
	previous: string | null;
}

interface UserState {
	users: any[];
	pagination: Pagination | null;
	loading: boolean;
	error: any | null;
	fetchUsers: (page?: number) => Promise<void>;
}

export const useUserStore = create<UserState>((set) => ({
	users: [],
	pagination: null,
	loading: false,
	error: null,
	
	fetchUsers: async (page = 1) => {
		set({ loading true, error: null });
		try {
			cosnt res = await api.get(`/users/?page=${page}`);
			set({
				users: res.data.results,
				pagination: {
					count: res.data.count,
					next: res.data.next,
					previous: res.data.previous,
				},
				loading: false,
			});
		} catch (err: any) {
			set({ error: err.response?.data, loading: false});
		}
	},
}));
```

- `loading`상태 -> 호출 중이면 `true`
- `users` -> 현재 페이지 데이터
- `pagination` -> `count`, `next`, `previous`
- `error` -> validation 에러나 서버 에러 그대로


### React 컴포넌트 사용 예시

```tsx
import { useEffect } from "react";
import { useUserStore } from "./useUserStore";

export default function UserList() {
	const { users, pagination, loading, error, fetchUsers } = useUserStore();
	
	useEffect(() => {
		fetchUsers();
	}, []);
	
	if (loading) return <p>Loading...</p>;
	if (error) return <p style={{ color: "red"}}>{JSON.stringify(error)}</p>;
	
	return (
		<div>
			<ul>
				{users.map((user) => (
					<li key={user.id}>{user.username}</li>
				))}
			</ul>
			
			<div>
				{pagination?.previous && (
					<button onClick={() => fetchUsers(new URL(pagination.previous).searchParams.get("page"))}>
						이전
					</button>
				)}
				{pagination?.next && (
					<button onClick={() => fetchUsers(new URL(pagination.next).searchParams.get("page"))}>
						다음
					</button>
				)}
			</div>
		</div>
	)
}
```

- `pagination.next` / `pagination.previous` 를 이용해 페이지 이동 가능
- validation error나 서버 에러도 `error`에서 그대로 표시
- loading 상태에 따라 로딩 UI 표시

### 이 구조의 장점:
1. **DRF 기본 RESTful** 응답 그대로 활용 -> DRF 기능 최대 활용
2. **React + Zustand**에서 상태 관리 깔끔하게 통합
3. **loading / error / pagination**까지 한눈에 관리 가능
4. 필요하면 **Axios 인터셉터**로 인증/공통 에러 처리 확장 가능

