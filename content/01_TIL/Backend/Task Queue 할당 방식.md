---
date: 2025-09-11
tags:
  - til
  - backend
  - django
---
# ❓ Information
* redis+batch vs celery+redis

---

# 🔰 Content ->  

여러 세션이 동시에 DB Table에 접근하여 update 를 시도할 경우 
DB lock 을 걸거나, 원자적으로 row 를 관리 할 수 있어야 한다.

예를 들어 `user_id IS NULL` 조건으로 `limit 1` 을 조회하여
`user_id` 를 업데이트 하려는 상황이 있다.


## 1️⃣ Redis에 row PK를 넣고 `spop` 으로 가져오기

### 흐름
1. 주기적으로 DB에서 `user_id IS NULL` 인 row 의 PK를 Redis `SMEMBERS` 혹은 `SADD` 로 채움.
2. Django에서 Redis `SPOP`으로 하나씩 꺼내서 DB에 `user_id` 할당
3. `SPOP`은 **원자적**이므로 동시에 여러 프로세스가 접근해도 하나씩만 가져감

### 장점
- **단순함**: 스크립트 한두 개로 바로 구현 가능
- **동시성 안전**: `SPOP` 자체가 원자적이므로 multiple worker가 동시에 꺼내도 겹치지 않음
- DB lock 필요 없음 -> DB 부하 적음
- 이미 가져올 row를 Redis에 미리 넣어둬서 worker 속도가 빠름

### 단점
- **DB <-> Redis 동기화 문제**: 
	- 주기적으로 Redis에 batch로 채워주므로, DB에 새 row 가 생기면 바로 반영 안됨
	- worker가 job을 다 소모하면 Redis가 비고, DB 갱신까지 기다려야함
- **추가 스크립트 필요**:
	- Redis에 job을 채우는 cron/배치 스크립트 필요
- **추적 어려움**:
	- 어떤 row 가 현재 처리 중인지 Redis에서만 존재 -> 장애 시 확인 어렵다


## 2️⃣ Celery + Redis + DB 방식 `(select_for_update(skip_locked))`

### 흐름
1. Celery worker가 task 호출
2. DB에서 `user_id IS NULL` row를 `select_for_update(skip_locked)` 로 가져옴
3. DB row에 바로 `user_id` 할당
4. Redis는 단순 브로커 역할 -> Task Queue 관리만 함

### 장점
- **실시간 처리**: DB row를 바로 가져오기 때문에 새로 생긴 row도 즉시 처리 가능
- **단순 배포/관리**: 별도 batch 스크립트 없이 task 호출만으로 처리
- **안전한 동시성 처리**: skip_locked 덕분에 여러 worker 동시에 실행해도 충돌 없음
- **장애 복구 용이**: task 실패 시 재시도 로직 쉽게 구현 가능(Celery `retry`)
- **확장성**: worker를 늘려도 자동 분산 처리 가능

### 단점
- DB에 트랜잭션/lock을 걸기 때문에 **DB 부하**가 batch + Redis 방식보다 조금 있음
- MariaDB 트랜잭션/row lock을 잘 이해해야 함


## 3️⃣ 비교 요약
| 항목     | Redis SPOP 방식     | Celery + Redis + DB 방식        |
| ------ | ----------------- | ----------------------------- |
| 구현 난이도 | 쉬움                | 조금 더 복잡 (Celery 설정 필요)        |
| 동시성 안전 | O (SPOP 원자성)      | O (skip_locked + transaction) |
| 실시간 처리 | 미묘함 (batch 주기 문제) | 즉시 처리 가능                      |
| DB 부하  | 적음                | 조금 있음 (트랜잭션+lock)             |
| 장애 복구  | 어려움               | Celery retry 가능               |
| 확장성    | 제한적               | 용이 (worker 늘리면 분산 처리)         |
| 유지보수   | 배치 스크립트 필요        | Celery task 중심, 코드 관리 간단      |

### 🔍 결론

- **작업량 적고, DB row가 자주 안 생기거나 단순 배치로 충분한 경우** → Redis SPOP 방식도 충분함.    
- **동시성 높은 환경, 실시간 처리 필요, 장애 복구/재시도 중요** → Celery + Redis + DB 방식이 더 안정적임.

즉, **규모/복잡도/실시간 요구에 따라 선택**하면 됨.


