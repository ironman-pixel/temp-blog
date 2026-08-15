---
date: 2025-11-17
tags:
  - cli
  - terminal
  - til
  - dev-tools
---
# ❓ Information
* 설치 방식의 철학 차이

---

# 🔰 Content ->  

## 장점
1. 최신 버전 확보
	- brew/apt 는 패키지 관리자가 최신 버전을 바로 제공하지 않을 수 있음
	- curl 스크립트는 공식 GitHub 배포판을 바로 내려받기 때문에 최신 기능을 즉시 쓸 수 있음
2. OS 독립성
	- WSL, Ubuntu, macOS 등 OS 가 달라도 같은 설치 스크립트로 설치 가능
	- brew는 Lunux용으로 설치 가능하지만 일부 패키지는 macOS 용 중심
3. 간단한 초기화
	- 스크립트가 자동으로 PATH 설정까지 넣어주고 초기화 코드를 작성해줌
	- brew 설치 이후에도 `.zshrc` 에 초기화 코드가 필요함

## 단점
1. 관리 어려움
	- 설치한 패키지가 OS 패키지 DB에 기록되지 않음 -> `apt list`나 `brew list`로 안나옴
	- 삭제 시 스크립트가 제공하는 uninstall 명령어를 써야 하거나 수동으로 디렉토리 제거
2. 의존성 관리 어려움
	- brew는 의존성 패키지까지 관리하지만, 스크립트 설치는 독립 설치
