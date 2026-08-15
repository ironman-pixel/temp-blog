---
date: 2025-08-18
tags:
  - til
  - backend
  - postgresql
  - python
---
# ❓ Information
* 의존성 설치중 psycopg2 문제 해결

---
# ❗ Relevant data
## 📦 Information Resources
[tistory:psycopg2](https://ks1171-park.tistory.com/142)

# 🔰 Content ->   
## psycopg2
- C 언어로 작성된 PostgreSQL 언어 바인딩의 Python 구현
- Python 환경에 직접 컴파일 해야 하기 때문에 **컴파일러와 관련된 의존성**이 있다
- 설치하려면 해당 운영체제에 컴파일러 및 PostgreSQL 개발 파일이 설치되어 있어야 한다
- 설치를 위해 일반적으로 C 컴파일러와 PostgreSQL 개발 파일을 수동으로 설치 해야한다

## psycopg2-binary
- psycopg2의 이진 패키지로 C 컴파일 과정을 거치지 않고 바로 사용할 수 있는 **사전 빌드된 바이너리 파일**을 제공한다
- 때문에 psycopg2를 설치할 때 컴파일러 및 PostgreSQL개발 파일에 대한 의존성을 없애주고 설치 과정이 간단함

## GPT 왈
production 환경에서는 보통 PostgreSQL 클라이언트가 설치 되어있으니 `psycopg2`를 사용하고,
dev 환경에서는 그냥 `psycopg2-binary`를 사용해도 무관