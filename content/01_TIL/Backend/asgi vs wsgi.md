---
date: 2025-06-20
tags:
  - til
  - backend
  - django
  - fastapi
  - python
---
# ❓ Information
* WSGI / ASGI 내용 정리

---

# 🔰 Content ->  
## 1️⃣ WSGI / ASGI는 "인터페이스 명세"

- 둘 다 **웹 서버 ↔ 웹 애플리케이션 사이의 통신 방법을 표준화**한 **프로토콜**이자 **인터페이스 규약**
- Python이 만들어낸 명세이기 때문에, **Python 웹 프레임워크는 반드시 이 중 하나를 따라야만** 웹서버가 실행할 수 있음

| 구분              | 설명                                                            |
| --------------- | ------------------------------------------------------------- |
| **WSGI** (2003) | 동기 기반. Flask, Django (3.0 이전) 등 전통 프레임워크용                     |
| **ASGI** (2018) | 비동기 + 동기 둘 다 지원. FastAPI, Starlette, Django(3.0+) 등 현대 프레임워크용 |

예시:

```python
# WSGI: 함수형 인터페이스
def app(environ, start_response):
    ...

# ASGI: 비동기 지원, 이벤트 기반
async def app(scope, receive, send):
    ...
```

---

## 2️⃣ Python은 시스템 수준 웹 서버(ex. nginx, apache)를 **직접 다룰 수 없음**

- Python 코드 자체로는 HTTP 소켓을 수신하고 처리하는 일을 하기에 너무 느리고, 구조적으로 복잡해.
- 그래서 Python 앱은 **명세(WSGI/ASGI)를 따르고**, 이 명세를 해석해서 실제 HTTP 통신을 처리해주는 **"서버"가 따로 필요**함.

---

## 3️⃣ Uvicorn은 **ASGI 명세를 지원하는 고성능 웹 서버**

- FastAPI 같은 앱은 내부적으로 `async def app(scope, receive, send)`처럼 생긴 ASGI 앱을 정의함
- Uvicorn은 이걸 불러서 HTTP 요청을 처리하고, FastAPI에게 전달해주는 역할을 함

즉:

```
[브라우저 요청] → uvicorn (ASGI 서버) → FastAPI 앱 실행 → 응답
```

---

## 4️⃣ 그럼 언어마다 웹 서버가 다른 이유는?

### ▶ 언어마다 **실행 모델과 생태계가 다르기 때문**

- Python: 느리고 싱글스레드 중심 → WSGI/ASGI 필요
- Node.js: 자체가 이벤트 루프 기반 서버임 → Express도 결국 `http.createServer()` 사용
- Go: net/http 내장 서버 있음 → goroutine으로 동시성 처리 가능
- Java: 톰캣, Jetty 같은 서블릿 컨테이너 → JVM 기반 스펙(Tomcat은 일종의 WSGI 같은 역할)

| 언어      | 웹 인터페이스/스펙        | 실행 서버                        |
| ------- | ----------------- | ---------------------------- |
| Python  | WSGI / ASGI       | gunicorn, uvicorn, hypercorn |
| Node.js | 자체 이벤트 루프         | Express, Koa, Fastify 등      |
| Java    | 서블릿, JAX-RS       | Tomcat, Jetty                |
| Go      | net/http, fiber 등 | 내장 서버 또는 HTTP 프레임워크          |
| Ruby    | Rack (WSGI 느낌)    | Puma, Unicorn                |

---

## 5️⃣ 결론 – 왜 Uvicorn이 필요한가?

- FastAPI 앱은 **ASGI 명세를 따르는 오브젝트**일 뿐이야.
- 이걸 실제 HTTP 요청으로 실행시켜줄 **ASGI-compatible 서버**가 없으면 **웹 서비스 불가**.
- Uvicorn은 이 역할을 수행하는 **초고속 비동기 이벤트 기반 서버**야.
- 그래서 FastAPI를 "돌리려면" 반드시 Uvicorn, Daphne, Hypercorn 같은 **ASGI 서버**가 필요.

---

### FastAPI를 **그냥 gunicorn으로 배포하는 건 "바보 같은 짓" 맞음.**

하지만...

### **gunicorn + uvicorn worker로 배포하는 건 똑똑한 선택**

이유는...

---

## FastAPI는 **ASGI** 앱이다

- FastAPI는 `async def` 기반 비동기 처리를 기본으로 설계된 **ASGI 앱**이다.
- 따라서 **WSGI 서버(gunicorn) 단독**으로는 실행 불가능.
- gunicorn은 **WSGI-only** 서버기 때문에 **ASGI 앱을 이해하지 못함**.

---

## gunicorn으로 FastAPI 실행은 언제 말이 되는가

```bash
복사편집
gunicorn app.main:app -k uvicorn.workers.UvicornWorker -w 4
```

- 여기서 핵심은 `k uvicorn.workers.UvicornWorker`
    - gunicorn이 **워커 관리 (프로세스, 로깅 등)** 만 담당
    - uvicorn이 **실제 ASGI 실행** 담당

### 이건 "ASGI 서버 + WSGI 프로세스 관리자" 조합이므로:

- 멀티코어 환경에서 성능 최적화 가능
- 로드밸런싱, graceful reload, worker 재시작 등 gunicorn의 강력한 기능 사용 가능

---

## 그래서 최종 정리:

|방법|설명|적절성|
|---|---|---|
|`gunicorn app.main:app`|❌ 동작 안 함 (FastAPI는 ASGI)|❌|
|`uvicorn app.main:app`|✅ 단독 실행, 비동기 I/O 최적화|✅ (개발/소규모 배포용)|
|`gunicorn -k uvicorn.workers.UvicornWorker`|✅ gunicorn + uvicorn 결합|✅ (프로덕션용 Best Practice)|

---

## 더 깊게 들어가면

- gunicorn은 안정적인 멀티프로세스 관리에 강하고,
- uvicorn은 초고속 비동기 이벤트 루프(uvloop, httptools) 처리에 강함
- 이 둘을 결합하면:
    - **고성능 + 안정성 + 확장성**을 동시에 챙길 수 있음

---

**결론 요약:**

- ✅ **FastAPI + uvicorn** = 깔끔한 기본 구성
- ✅ **FastAPI + gunicorn (uvicorn worker)** = 실전 배포에 최적
- ❌ **FastAPI + gunicorn (WSGI만)** = 완전 잘못된 구성

---
