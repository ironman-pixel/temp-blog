---
date: 2026-03-07
tags:
  - til
  - ai
  - llm
  - gemini
---
## 핵심 명령어 3가지

### 1. `/clear` - 대화 초기화
- 정확한 답변을 위해 새로운 주제로 전환할 때 사용
- 컨텍스트 윈도우를 소모하기 때문에 이를 관리할 필요가 있음

### 2. `/compress` - 토큰 최적화
- 대화 내용을 압축하여 토큰 수를 줄이는 기능
- 주요 내용은 유지하면서 효율성 개선

### 3. `/memory` - 장기 기억 저장
- `/memory add`로 규칙 저장
- `/memory show`로 확인
- 중요한 정책을 반복적으로 입력할 필요 제거

```
/memory add you do not allow to run `kubectl create` and `kubectl run` without specific namespace
```
장기 기억의 실제 내용은 `~/.gemini/GEMINI.md` 파일에 기록됨

---

## GEMINI.md 파일 활용

글로벌 설정 파일(`~/.gemini/GEMINI.md`)에 저장되는 장기 기억으로, 다음 정보를 포함:
- 기술 스택 및 버전
- 프로젝트 구조
- 코드 스타일 가이드
- 금지 사항 명시

---

## settings.json 설정

### excludeTools 옵션
- 위험한 명령어를 완전히 차단 (예: `kubectl delete`)
- "blocked by configuration" 에러로 실행 방지

### contextFileName 옵션
- GEMINI.md, ARCHITECTURE.md, AGENTS.md 등 다중 문서 활용
- 프로젝트별 지역 설정 가능

---

## .env 파일 관리

- API 키를 셸 설정 대신 `.env`에 저장
- 현재 디렉토리 → 프로젝트 루트 → 홈 디렉토리 순서로 검색
- 라운드 로빈 방식으로 여러 키 관리 가능

---

## Agent Skills (에이전트 스킬)

### 개요
Agent Skills는 Gemini CLI를 전문적인 기술과 작업 특화 리소스로 확장하는 기능입니다. "Agent Skills 개방형 표준"을 기반으로 하며, 각 스킬은 지침과 자산을 패키징한 독립적 디렉토리입니다.

### 핵심 이점
- **공유 가능한 전문성**: 팀의 PR 검토 프로세스 같은 복잡한 워크플로우를 폴더로 패키징 가능
- **반복 가능한 워크플로우**: 다단계 작업을 일관되게 수행하도록 절차적 프레임워크 제공
- **리소스 번들링**: 스크립트, 템플릿, 예제 데이터를 지침과 함께 포함
- **점진적 공개**: 초기에는 스킬 메타데이터만 로드하고 상세 지침은 활성화 시에만 로드하여 효율성 제공

### 스킬 발견 계층 (우선순위 순)
1. **워크스페이스 스킬**: `.gemini/skills/` 또는 `.agents/skills/`
2. **사용자 스킬**: `~/.gemini/skills/` 또는 `~/.agents/skills/`
3. **확장 프로그램 스킬**: 설치된 확장 프로그램 내 스킬

### 스킬 관련 주요 명령어
| 명령어 | 설명 |
|--------|------|
| `/skills list` | 발견된 모든 스킬 목록 표시 |
| `/skills disable <name>` | 특정 스킬 비활성화 |
| `/skills enable <name>` | 특정 스킬 활성화 |
| `gemini skills install` | 스킬 설치 |
| `gemini skills link` | 로컬 디렉토리에서 스킬 연결 |

### 스킬 활용 팁
- 자주 사용하는 작업 흐름은 스킬로 패키징하여 팀원들과 공유 가능
- 로컬 워크스페이스에 스킬을 배치하면 프로젝트별 커스텀 도구 생성 가능
- 확장 프로그램을 통해 커뮤니티 스킬 설치하고 활용할 수 있음
