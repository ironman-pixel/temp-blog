---
date: 2025-09-02
tags:
  - til
  - backend
  - django
  - fastapi
  - pydantic
  - python
---
# ❓ Information
* Pydantic의 데이터 검증

---
# ❗ Relevant data
## 🎯 What Is The Objective
Pydatic이 무엇인지 알아본다
## 📦 Information Resources
[tistory: Pydantic이란](https://lsjsj92.tistory.com/650)

# 🔰 Content ->  

## Install
```bash
pip install pydantic
```

## FastAPI에서 Pydantic활용 데이터 검증
input, output 형태 검증

```python
from fastapi import FastAPI
from pydantic import BaseModel
from pydantic import Field

app = FastAPI()

class DataInput(BaseModel):
	name: str

class PredictOutput(BaseModel):
	prob: float
	prediction: int

@app.post(“/pydantic”, response_model=PredictOutput)
def pydantic_post(data_request: DataInput):
	return {“prob”: 0.1, “prediction”: 0}
```

`data_request` 는 `DataInput` class 형태로 받고, 
`response_model`은 `PredictionOutput`형태로 지정했다.


## DRF Serializer 와 비교

### Pydantic
1. 속도
	- **데이터 검증이 빠름**
	- **대규모 데이터나 API응답 처리**에서 DRF 보다 효율적임
2. 타입 힌트와 통합
	- **자동 타입 변환, 유효성 검사** 지원
	- FastAPI 연결로 **자동 문서화, 검증**이 쉬움
3. 경량화
	- Django 전체를 쓰지 않고 독립적으로 **데이터모델 + 검증** 구현 가능
4. 직관적인 직렬화/역직렬화
	- 딕셔너리 -> 객체 -> 딕셔너리 순환이 간단
	- `dict()`, `json()` 같은 메서드로 변환 가능

### DRF Serializer
1.  Django ORM과 통합
	- 모델, 쿼리셋, 관계형 필드, Nested serializer 까지 자연스럽게 처리 가능
	- DB기반 CRUD API작성에 최적화됨
2. 권한, validation, save() 기능
	- 검증뿐 아니라 **create/update 시DB 저장 로직**까지 포함 가능
	- Pydantic은 순수 데이터 검증이 주 목적이어서 DB 저장은 별도 구현 필요
3. 강력한 커스터마이징
	- 필드별 검증, 오류 메시지, 중첩serializer, ListField, ChoiceFiled등 다양한 옵션 제공
	- Django의 Form/ModelForm과 친숙하게 연동됨
