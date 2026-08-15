---
date: 2025-09-03
tags:
  - package-manager
  - til
  - dev-tools
---
# ❓ Information
* pyproject.toml 파일 사용법

---

# 🔰 Content ->  

## Usage

FastAPI, Django, Flask 등 모든 파이썬 웹 프레임워크 프로젝트에서 의존성 관리 도구 설정을 위해 사용됨

## File Structure
### project

프로젝트 이름, 버전, 설명 등 기본적인 정보를 명시
```toml
[project]
name = "ai-planner-backend"
version = "1.0.0"
description = "This is backend for the AI Planner."
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
]
```


### dependency-groups

requirements.txt 에서 모든 의존성을 관리하는것이 아니라 효율적임
프로덕션 환경에 불필요한 패키지가 설치되는 것을 방지
```toml
[dependency-groups]
dev = [
    "pre-commit>=4.3.0",
    "ruff>=0.12.11",
]
```

아래 명령으로 개발 의존성 설치 가능
```bash
$ pip install '.[dev]'
```


### tool.{}

다음은 ruff의 예시
```toml
[tool.ruff]
line-length = 80
exclude = [
    ".git",
	".venv",
    "__pycache__",
    "tests/*"
]
lint.ignore = ["F401", "E402", "E501", "F403", "F405", "F811"]
lint.select = ["I", "E", "F", "W", "C90"]
```

- line-length: 줄 길이 제한 설정
- exclude: 제외할 폴더나 파일
- lint.ignore: 특정 에러 코드 무시
	- F403: from module import * 사용 (star import)
	- F405: star import로 인해 정의되지 않을 수 있는 이름들
	- F811: 사용하지 않는 변수의 재정의
- lint.select: 검사할 규칙 선택
    - "I": 임포트 관련 규칙 검사
    - "E": 코드 스타일 검사, 주로 **PEP 8 스타일 문제**와 관련된 규칙 검사
    - "F": 일반적인 **Python 코드 오류**
    - "W": **잠재적인 경고**를 생성할 수 있는 요소
    - "C90": 코드의 **복잡도**와 관련된 문제 검사



