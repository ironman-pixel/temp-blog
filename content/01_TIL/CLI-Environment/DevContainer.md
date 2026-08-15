---
date: 2025-10-30
tags:
  - cli
  - terminal
  - til
  - dev-tools
  - devcontainer
---
# ❓ Information
* 개발에 필요한 모든 도구와 설정을 컨테이너로 패키징한 개발환경

---

# 🔰 Content ->  
## 장점

개발에 필요한 모든 설정이 코드로 정의되어 있어, 버전 관리 시스템을 통해 팀원들과 공유할 수 있다

팀원들은 운영체제에 상관없이 컨테이너 내부에서 동일한 환경이 제공된다

## 사용법

VSCode Extention: Dev Containers 설치

`.devcontainer` 내부의 `devcontainer.json`

```json
{
  "name": "Python 3.13",
  "image": "mcr.microsoft.com/devcontainers/python:3.13"
}
```

설정 후 VSCode 의 명령 팔레트 안에서 다음 실행 (Docker 가 실행 중이어야함)

![[static/Pasted image 20251030133842.png | 400]]

그 다음 IDE 가 해당 환경 안에서 열림

_아직 완전한 단계는 아니라고 함?_
