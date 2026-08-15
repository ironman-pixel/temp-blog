---
date: 2025-08-20
tags:
  - package-manager
  - til
  - dev-tools
  - python
---
# ❓ Information
* how to control python version from mac

---

# 🔰 Content ->  

1. Homebrew 에서 원하는 python version 이 있는지 확인한다.
`brew list | grep python`

2. Homebrew 에서 원하는 python 버전을 설치한다.
`brew install python@3.10`

3. 설치한 경로를 확인한다.
`brew --prefix python@3.10`

4. 만약  venv 가상환경을 만들거라면 다음과 같이 사용 가능
`{path}/bin/python3.10 -m venv venv-py310`

path 는 실행 파일이 아니라 심볼릭 링크 디렉터리 이기 때문에 
`/bin/python3.10` **실행기** 를 붙어줘야함