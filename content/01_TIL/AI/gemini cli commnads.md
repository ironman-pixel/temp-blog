---
date: 2025-07-19
tags:
  - til
  - ai
  - llm
  - gemini
---
# ❓ Information
* gemini-cli commands

---
# ❗ Relevant data
## 📦 Information Resources
[notion: gemini-cli](https://www.0x00.kr/ai/gemini/gemini-cli-install-and-simple-usage)

# 🔰 Content ->  

## 내장된 명령어 목록

- 명령어 목록
	- `/help`: 모든 명령어랑 간단한 설명 보여줌.

- 인증 및 계정 관리
	- `/auth`: 구글 계정 인증 관리함.
		- `login`: 로그인함.
		- `logout`: 로그아웃함.
		- `status`: 지금 로그인 상태 확인해줌.

- 메모리 및 설정 관리
	- `/memory`: Gemini 장기 기억(사용자 정보) 관리함.
		- `list`: 저장된 기억 다 보여줌.
		- `clear`: 기억 다 지워줌.
	- `/config`: CLI 설정 관리함.
		- `list`: 설정 키랑 값 다 표시해줌.
		- `get` : 특정 설정 값 확인해줌.
		- `set` : 특정 설정 값 바꿔줌.
		- `reset`: 모든 설정 기본값으로 돌려놓음.

- 유틸리티 및 기타 기능
	- `/bug`: 버그 신고 시작함.
	- `/feedback`: 피드백 제출함.
	- `/clear`: 터미널 화면 지워줌.
	- `/history`: 이 세션에서 쓴 명령어 기록 보여줌.
	- `/retry`: 마지막으로 실패한 명령어 다시 시도함.
	- `/exit` 또는 `/quit`: Gemini-CLI 세션 끝냄.
	- `/theme`: CLI 색깔 테마 바꿔줌.
	- `/workspace`: 작업 디렉토리 정보 보여주거나 바꿔줌.
	- `/mode`: 도구 사용 확인 모드(자동/수동) 전환함.
	- `/log`: CLI 로그 확인해줌.
	- `/share`: 이 세션 공유 링크 만들어줌.
	- `/undo`: 마지막 작업 취소함.
	- `/redo`: 취소했던 작업 다시 실행함.