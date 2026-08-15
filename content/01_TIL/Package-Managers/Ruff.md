---
date: 2025-09-03
tags:
  - package-manager
  - til
  - dev-tools
  - python
---
# ❓ Information
* Ruff (Python Linter & Formatter)

---

# 🔰 Content ->  

## Install 
```bash
pip install ruff
```

## 프로젝트에 적합하게 Ruff 설정 구성

_나는 pyproject.toml 을 사용함_
```bash
[tool.ruff]
line-length = 80
exclude = [
	".git",
	"__pycache__",
	"tests/*"
]
lint.ignore = ["F401", "E402", "E501"]
lint.select = ["I", "E", "F", "W", "C90"]
```

## python 코드 품질 분석
```bash
$ ruff check .

$ ruff check . --fix
```
