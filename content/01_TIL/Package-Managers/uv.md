---
date: 2025-12-13
tags:
  - kanban
  - project
---
## Create Project

```bash
# 새로운 uv 프로젝트를 초기화
uv init uv-demo
cd uv-demo
```

```bash
# .venv 폴더는 아직 보이지 않을 수 있는데, 의존성을 추가하면 자동으로 생성됨
uv-demo/  
├── .python-version  # 파이썬 버전 고정
├── .gitignore  
├── pyproject.toml  # 의존성 및 프로젝트 메타데이터 정의
├── hello.py  
└── README.md
```

## Install Dependency

```txt
# requirements.txt
kuzu==0.7.1  
lancedb==0.17.0  
llama-index==0.12.8  
llama-index-llms-openai==0.3.12  
llama-index-embeddings-openai==0.3.1  
llama-index-graph-stores-kuzu==0.6.0  
llama-index-vector-stores-lancedb==0.3.0  
numpy==2.2.1  
polars==1.18.0  
pyarrow==18.1.0  
python-dotenv==1.0.1  
  
# 사전 준비: 캐시 정리  
`uv clean cache`  
`pip cache purge`  
`poetry cache clear - all .`
```

### 1) pip 사용 시 
```bash
python -m venv .venv  # 로컬 가상환경 생성
source .venv/bin/activate  # 가상환경 활성화
time pip install -r requirements.txt
```

- 설치 시간: 약 18.3초
- 주의 사항: 가상환경을 활성화하는 작업을 잊으면 **시스템 전역**에 설치해버릴 위험이 있음

###  2) poetry 사용 시 
```bash
rm -rf .venv  # 기존 가상환경 제거
poetry init  # poetry 설정 초기화
poetry shell  # poetry가 만든 가상환경 진입
time poetry install  # 의존성 설치
```

- 설치 시간: 약 6.3초
- poetry는 `poetry.lock`을 생성하고, 이를 기반으로 의존성을 세심하게 관리함
- 사용 시 `poetry shell`에 들어가야 하므로, 약간 귀찮을 수 있음

### 3) uv 사용 시
```bash
uv init  # pyproject.toml 생성 (필요 시)
time uv add -r requirements.txt  # 의존성 설치
```

- 설치 시간: 약 2.3초
- .venv 폴더가 자동으로 생성되고, 가상환경을 별도로 활성화할 필요가 없음
- 이후 `uv run` 명령어로 Python 파일을 실행 할 때, 자동으로 .venv 환경이 적용됨


## 인터렉티브 개발과 uv

### 인터렉티브 환경 (예: VSCode, Cursor IDE)
- 프로젝트 초기 단계나 실험 단계에서, IDE/에디터 내장 **대화형 실행(Shift+Enter 등)** 을 자주 활용함
- `ipykernel`을 이용하면 Jupyter notebook처럼 코드 셸 단위로 빠르게 실행, 테스트할 수 있지만, 가상환경이 매번 달라지면 불편함

### uv의 장점
- 동일한 .venv 가상 환경을 IDE가 바로 인식하도록 해줌
- 예를 들어, `uv add -- dev ipykernel` 명령을 통해 **개발용 의존성(dev dependency group)** 으로 ipykernel을 설치해두면, IDE가 이를 자동으로 감지해 가상환경 커널을 연결함

```bash
uv add -r requirements.txt  # 주요 의존성 설치
uv add --dev ipykernel  # 개발용(인터렉티브) 의존성 추가
```

- 이로써 **주피터 노트북처럼 별도의 커널을 생성/관리** 할 필요가 현저히 줄어듦
- 팀원들이 다른 에디터를 쓰더라도, uv sync만 하면 동일한 .venv를 재현할 수 있음

### Jupyter 노트북 vs IDE 내장 커널
- 기존에는 Jupyter Lab이나 노트북에서 `python -m ipykernel install ...` 등 추가 작업을 해야 했지만, **uv는 프로젝트 폴더 하나만 공유**하면, 그 안의 . venv와 pyproject.toml, uv.lock 으로 곧바로 동일한 가상환경을 복원할 수 있음.


## 명령줄 실행

1. uv run
	- Python 스크립트를 CLI에서 실행할 때도, uv가 알아서 가상환경을 적용해줌
	```python
	# hello.py 예시
	import polars as pl
	
	df = pl.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6]})
	print("Hello from uv-demo!")
	
	# 명령줄에서 
	$ uv run hello.py
	# 출력: Hello from uv-demo!
	```
	1. Python이 설치되어 있는지 확인 및 설치(필요 시)
	2. .venv 가상환경 생성 및 활성화
	3. pyproject.toml 기반 의존성 설치
	4. 코드 실행 ...

**이 모든 과정을 uv run  한 줄로 해결함**

2. uv sync
	- CI/CD 등 자동화 환경에서 `uv sync` 만으로 프로젝트에 필요한 Python 버전과 .venv가 자동으로 맞춰짐

3. 의존성 잠금파일 (uv.lock)
	- poetry.lock 처럼 `uv.lock` 파일을 생성해 의존성 버전을 고정함
	- 빠른 빌드와 재현성을 동시에 보장함


## 명령어 소개

### uv init
- 새로운 uv 프로젝트 초기화, pyproject.toml 생성

### uv add <패키지명>
- 특정 패키지 추가
- `uv add -r requirements.txt`로 `requirements.txt`전체 추가 가능

### uv run <파일명.py>
- pyproject.toml과 uv.lock 파일을 기준으로 가상환경 재생성 및 동기화

### uv run <파일명.py>
- 가상환경을 자동 적용하여 파이썬 스크립트 실행

### uv tool install --python 3.11 <패키지명>
- 설치할 도구가 특정 Python 버전을 필요로 하는 경우


### uvx <툴> (`uv tool run` 명령어의 축약형)
- ruff, mypy 등 서드파트 명령을 자동으로 설치/실행 해줌
- 예) `uvx ruff check .`


## uv로 무엇을 대체할 수 있나?
- **pip**: 패키지 설치
- **pyenv**: 파이썬 버전 관리
- **poetry**: 의존성/빌드 관리
-  **venv**: 가상환경 생성
- **pipenv**: 환경+의존성 관리
- 대규모 팀 환경일수록, **도구 하나로 끝낸다** 라는 데서 오는 이점이 큼
- 프로젝트 빌드/실행/배포 파이프라인에서 uv를 중심에 두면, **Rust/Go처럼 깔끔한 개발자 경험**을 누릴 수 있다는 것이 핵심 포인트임


## uv 설치 경로 구조 이해
- `C:\Users\toplo\AppData\Roaming\uv\python\cpython-3.13.9-windows-x86_64-none\python.exe`
  → uv가 관리하는 **원본 Python 인터프리터 위치**  
  → 직접 PATH에 넣어도 되지만, 버전별로 디렉토리가 길고 바뀔 수 있음

- `C:\Users\toplo\.local\bin\python3.13.exe`
  → uv가 **실행용 래퍼(executable shim)** 를 만들어 놓은 위치  
  → Python 실행 시 이 파일을 호출하면 uv가 내부 Python 인터프리터를 자동으로 연결  
  → PATH에 넣으면 **uv가 관리하는 Python을 바로 호출 가능**

## uvx
원하는 Python 버전을 `--python` 플래그를 사용하여 지정

- **Python 3.11**이 설치된 환경에서 `flask`를 설치하고 `python` 인터프리터를 실행합니다.
```bash
$ uvx --from flask --python 3.11 python
```

- **Python 3.10** 환경에서 `pandas`가 설치된 상태로 `ipython`을 실행합니다.
```bash
$ uvx --from pandas --python 3.10 ipython
```

Python 인터프리터의 **절대 경로** 또는 **상대 경로**로 지정할 수도 있음
- `uvx --from flask /usr/bin/python3.12`
- `uvx --from black ~/venvs/py39/bin/python`
