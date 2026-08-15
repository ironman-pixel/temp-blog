---
date: 2025-09-04
tags:
  - til
  - ai
  - llm
  - ollama
---
# ❓ Information
* Ollama 사용법

---
# ❗ Relevant data
## 📦 Information Resources
[modulabs: ollama-langchain](https://modulabs.co.kr/blog/ollama-langchain)
[tistory: Ollama 성능 최적화 및 유지보수]


# 🔰 Content ->  

## Ollama 

PC에서 대규모 언어 모델(LLM)을 직접 실행할 수 있도록 도와주는 오픈소스 플랫폼
다양한 오픈소스 LLM 모델의 설치, 실행, 커스터마이즈 기능을 간편 제공

장점: 데이터 프라이버시, 비용 효율성, 오프라인 접근성

## Install

Ollama 홈페이지([https://ollama.com/](https://ollama.com/)) 에서 운영체제에 맞는 버전 download

### Model download
```bash
$ ollama pull <model name>

$ ollama list

$ ollama run <model name>
```

### Example
```bash
from langchain_core.prompts import ChatPromptTemplate
from langchain_ollama.llms import OllamaLLM

# Ollama 모델 초기화
model = OllamaLLM(model="llama3.2")

# 프롬프트 템플릿 정의
template = 
	"""
		You are an English vocabulary tutor. 
		When given a word, explain its meaning in simple terms, and provide an example sentence. 
		If the word has multiple meanings, explain each with examples. 
		Here's the input: 
		
		Word: {word} 
		
		Output format: 
		1. Definition: [Brief explanation in simple English] 
		2. Example Sentence: [A sentence using the word in context]
	"""

# LangChain 체인 구성
prompt = ChatPromptTemplate.from_template(template)
chain = prompt | model

def get_definition(word):
	return chain.invoke({"word": word})

if __name__ == '__main__':
	while True:
		word = input('enter word: ')
		if word == '/bye':
			break
		print(get_definition(word))
		print()
```


---

# 🔰 Ollama 기반 AI 성능 최적화 및 유지보수

## 성능 최적화가 중요한 이유

모델이 크면 **응답속도가 느려지고 CPU/GPU 사용량 증가**
특히 Ollama 같은 로컬 LLM을 실행할 경우,
최적화 없이 운영하면 **메모리 과부하 및 성능 저하 발생**

### ✅ AI 성능 저하 원인과 해결 방법

| 문제원인         | 해결방법                       |
| ------------ | -------------------------- |
| 모델 응답 속도 느림  | 경량 모델 사용 (mistral, llama3) |
| 메모리 사용량 과다   | GPU 가속 활용, 캐싱 적용           |
| API 요청 병목 발생 | 비동기 FastAPI 적용, 요청 큐 관리    |

_목표: 응답속도 개선, 리소스 사용 최적화_

## Ollama AI 모델 최적화 방법

### 1️⃣ 가벼운 모델 사용 (Mistral, Llama3)

#### 🔹 모델별 성능 비교

| 모델          | 크기    | 응답 속도 | 특징              |
| ----------- | ----- | ----- | --------------- |
| **Mistral** | 7B    | 빠름    | 가벼운 범용 AI 모델    |
| **Llama3**  | 13B   | 중간    | 성능과 속도 균형       |
| **GPT-4**   | 100B+ | 느림    | 고성능 (외부 API 필요) |

### 2️⃣ Ollama의 GPU 가속 활용
_GPU 사용 가능 여부를 확인하고, GPU 가속이 가능한 환경이면 활성화_

GPU를 활용하면 **AI 모델의 응답 속도를 최대 3~5배 향상** 가능
#### 🔹 GPU 가속 활성화 (NVIDIA GPU 사용 가능 시)
```bash
OLLAMA_USE_CUDA=1 ollama run mistral
```

#### 🔹 Docker에서도 GPU 가속 적용 가능
```bash
docker run --gpus all -p 8000:8000 ai-agent
```


### 3️⃣ AI 응답 캐싱 적용 (Redis 활용)

같은 질문을 여러번 받는 경우,
AI가 매번 연산하는 대신 캐시에서 빠르게 응답하도록 설정 가능
#### 🔹 Redis 설치 및 실행
```bash
docker run -d --name redis -p 6379:6379 redis
```

#### 🔹 Python 코드 (FastAPI + Redis 캐싱 적용)
```bash
import redis
import hashlib
from fastapi import FastAPI

app = FastAPI()
cache = redis.Redis(host='localhost', port=6379, db=0)
  
def cache_response(question, response): 
	key = hashlib.sha256(question.encode()).hexdigest() 
	cache.setex(key, 3600, response) # 1시간 동안 캐싱

def get_cached_response(question): 
	key = hashlib.sha256(question.encode()).hexdigest() 
	return cache.get(key)

@app.get("/ask/") 
async def ask_ai(question: str): 
	cached_response = get_cached_response(question) 
	if cached_response: 
		return {"response": cached_response.decode("utf-8")} 
	response = "AI가 처리한 결과" # CrewAI 실행 결과 (예제) 
	cache_response(question, response) 
	return {"response": response}
```

#### 🔹 AI 응답 속도 최적화 방법
- Redis를 활용하여 **같은 질문에 대한 응답을 캐싱**
- 반복적인 연산을 줄여 **API 응답 속도 향상**
- 사용자 경험 개선 (즉각적인 응답 제공)

### 4️⃣ AI API 서버 최적화 (FastAPI 비동기 처리)

#### 🔹 1. FastAPI 비동기 (Async) 적용
FastAPI는 비동기 기능을 활용하면 
**AI API 요청을 더욱 빠르게 처리** 가능

-  **비동기 요청의 장점**
	- **동시에 여러개의 AI 요청 처리 가능**
	- **API 응답 속도 향상** (사용자 대기 시간 감소)
	- **FastAPI와의 완벽한 호환성**

#### 🔹 2. AI 요청 큐 관리 (Celery 활용)
여러 사용자가 동시에 AI API를 요청하면 **서버 과부하 발생 가능**
Celery를 활용하여 **비동기 요청 큐**를 설정하면 더욱 효율적인 처리 가능

_요청을 큐에 저장하고, 비동기적으로 실행하여 서버 부하 감소_

- **Celery 설치 및 실행**
```bash
pip install celery redis
celery -A tasks worker --loglevel=info
```

- **비동기 요청 큐 적용 예제**
```python
from celery import Celery

app = Celery("tasks", broker="redis://localhost:6379/0")

@app.task
def process_ai_request(question):
	return "AI 처리 완료"  # CrueAI 실행 결과
```

### 5️⃣ AI 시스템 유지보수 전략

#### 🔹 1. 로그 관리 및 모니터링
AI 서비스가 정상적으로 동작하는지 확인하기 위해
로그를 저장하고, 모니터링 시스템을 구축

_log를 저장하여 서비스 장애 발생 시 분석 가능_
_Prometheus 등 모니터링 도구를 활용하여 AI 시스템 상태 체크_


- **AI 응답 로그 저장 (logging 활용)**
```python
import logging

logging.basicConfig(filename="ai_logs.log", level=logging.INFO)

def log_request(question, response):
	logging.info(f"질문: {question} | 응답: {response}")
```

- **AI 서비스 모니터링 (Prometheus 활용)**
```bash
docker run -d --name prometheus -p 9090:9090 prom/prometheus
```

#### 🔹 2. AI 모델 업데이트 및 지속적인 개선
AI 모델을 최신 상태로 유지하려면 주기적으로 업데이트 해야 함

_새로운 AI 모델을 적용하고, 지속적인 학습을 통해 AI 성능 개선_

- **최신 모델 다운로드 및 적용**
```bash
ollama pull mistral
ollama pull llama3
```

- **FastAPI 서버 재시작하여 모델 적용**

```bash
docker restart ai-agent
```

### 6️⃣ AI 성능 최적화 및 유지보수 정리

 - 경량 모델(Mistral, Llama3) 사용으로 속도 최적화 가능
 - GPU 가속 활용으로 AI 응답 속도 3~5배 향상 가능
 - Redis 캐싱으로 동일 질문에 대한 AI 처리 속도 향상
 - 비동기 FastAPI & Celery를 활용한 AI 요청 최적화
 - 로그 관리 및 모니터링으로 AI 유지보수 체계 구축

