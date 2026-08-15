---
date: 2025-09-29
tags:
  - book
  - react
  - design-patterns
  - frontend
  - general
---
콜백 함수의 참조를 메모이제이션 하고 최적화 할 때 사용
콜백 함수를 자식 컴포넌트에게 전달하거나 다른 훅의 의존성 목록으로 지정할 때 유용

```ts
const memoizedCallback = useCallback(callback, dependencies);
```

- `callback`:
  메모이제이션 대상이 되는 함수, 인라인 함수 또는 참조(변수)
- `dependencies`:
  메모이제이션 하려는 콜백함수의 의존성 배열

예시: 
에디터 컴포넌트에서 사용자가 글자를 입력할 때마다 본문에 업데이트가 생김.
이는 리렌더링 발생.
리렌더링할 때마다 새로운 함수가 생성되어 성능 저하

```ts
const ArticleEditor = ({ id }: { id: string })  => {
	const submitChange = useCallback(
		async (summary: string) => {
			try {
				await fetch(`/api/articles/${id}`, {
					method: "POST",
					body: JSON.stringify({ id, summary }),
					headers: {
						"Content-Type": "application/json",
					},
				});
			} catch (error) {
				// handling errors
			}
		},
		[id]
	);
	return (
		<div>
			<ArticleForm onSubmit={submitChange} />
		</div>
	)
}
```
