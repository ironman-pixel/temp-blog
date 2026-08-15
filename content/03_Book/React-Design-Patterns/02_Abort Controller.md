---
date: 2025-09-29
tags:
  - book
  - react
  - design-patterns
  - frontend
  - general
---
## `setTimeout`으로 컴포넌트 마운트 해제시 타이머 삭제하는 예시

```ts
const Timer = () = {
	useEffect(() => {
		const timerId = setTimeout(() => {
			console.log("time is up")
		}, 1000);
		return () => {
			clearTimeout(timerId)
		};
	}, [])
	retrun <div>Hello timer</div>;
}
```

## 예시와 같은 정리 함수로 메모리 누수 방지

```ts
useEffect(() => {
	const controller = new AbortController();
	const signal = controller.signal;
	const fetchArticleDetail = async (id: string) => {
		fetch(`/api/articles/${id}`, { signal })
			.then((res) => res.json())
			.then((data) => setArticle(data));
	};
	
	fetchArticleDetail(id);
	
	return () -> {
		controller.abort();
	};
}, [id]);
```

컴포넌트 마운트 시 AbortController인스턴스 생성하고 signal 참조
signal은 fetch 함수에 전달되고 컨트롤러 요청과 연결

만약 네트워크 요청 완료 이전에 컴포넌트 언마운트 시 abort 메서드로 수행중인 네트워크 요청 취소하는 정리 함수 호출

언마운트된 컴포넌트의 상태를 업데이트 하는 등의 잠재적인 이슈 방지
