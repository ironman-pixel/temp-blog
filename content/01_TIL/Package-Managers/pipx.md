---
date: 2025-12-08
tags:
  - package-manager
  - til
  - dev-tools
  - python
---
# ❓ Information
* pipx는 TIL 설치시 유용하다

---

# 🔰 Content ->  

## pip와 관계
pip와 밀접한 관련이 있으며, 실제로 pip를 사용하지만, 
주로 명령줄에서 직접 애플리케이션으로 실행할 수 있는 Python 패키지를 설치하고 관리하는데 중점을 둔다

## pip와 차이점
pip는 환경 격리 없이 라이브러리와 애플리케이션 모두를 위한 범용 패키지 설치 도구 
- 프로젝트 안에서 독립적인 가상환경을 만들고 그 안에 패키지를 설치하기 때문에 아직 사용됨

pipx는 각 애플리케이션과 관련 패키지에 대한 격리된 환경을 만듦
- thefuck
- black
- ruff
- httpie
- cookiecutter

## 주요 기능
install 명령을 사용하여 격리된 환경에 설치된 패키지("apps")의 CLI entrypoints를 노출한다