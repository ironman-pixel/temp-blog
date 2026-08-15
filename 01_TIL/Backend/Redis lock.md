---
date: 2025-09-16
tags:
  - til
  - backend
  - redis
  - python
---
# ❓ Information
* Redis 에서 사용자 단위로 접근을 제어 하는 방법

---

# 🔰 Content ->  

```python
lock_key = f'lock:get_job_annotator:{user_id}'
if not redis_client.set(lock_key, "locked", nx=True, ex=5):
	return JsonResponse({'error': 'Resource is locked'}, status=423)
```

1. `lock_key` 생성
	- Redis 에 저장될 락(lock) 키 이름을 문자열로 만드는것

2. Redis `set` 명령어 (옵션들)
	- Redis 에서 분산 락 (distributed lock) 기능처럼 사용하는 패턴

	- 옵션 (해당 user_id 에 대한 락이 없다면 5초동안 락을 걸어라)
		- `nx=True` 
		  키가 존재하지 않을때만 저장 (Not eXists)
		- `ex=5`
		  5초 동안만 유지되는 TTL(time to live) 설정
		- "locked"
		  저장되는 값. 값 자체는 중요하지 않고 "락 걸렸음" 표시용

3. 락 실패 처리
	- `set` 이 실패 했다면 -> 이미 `lock_key` 가 존재한다는 뜻
	- 즉 사용자의 중복 요청이 들어온 상황
	- 이 경우 바로 응답
		- `423 locked` -> HTTP 상태 코드
		- 메시지: `"Resource is locked"`

