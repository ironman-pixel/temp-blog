---
date: 2025-05-29
tags:
  - til
  - dev-tools
  - git
---
# how to use git from quartz

```
git stash         # 변경사항 임시 저장
git fetch upstream
git switch main
git rebase upstream/main
git switch my-feature
git rebase main


git push --force-with-lease origin my-feature

```

upstream에서 최신 상황을 pull 받으면서
내 전용 브랜치에만 push 하는 상황은 처음이라
[gpt한테 물어봄](https://chatgpt.com/c/683863af-5fc0-8001-bfe9-6f4746ad6023)

---

## fork from quartz repo

사실 `fork` 한건 아니지만 `upstream` 을 설정 함으로써 거의 동일한 환경이 된것 같다.

### GitHub Fork와의 차이점

|항목|GitHub Fork|수동 설정 (`remote add`, `set-url`)|
|---|---|---|
|**웹 UI 통합**|GitHub에서 Fork 버튼으로 자동 생성됨|직접 로컬에서 설정 필요|
|**PR 연결성**|원본 리포에 Pull Request 보내기 쉬움|직접 upstream을 설정하고, 수동으로 PR|
|**리포 구조**|자동으로 `forked from` 정보 표시됨|GitHub에서 표시되지 않음|
|**초기 세팅**|GitHub가 알아서 원본 연결까지 함|remote 수동 설정 필요|

---

## upstream 을 rebase 하면서 사용

### 시나리오 정리

1. `my-feature` 브랜치는 이미 원격에 푸시된 상태였음:
```
A -- B -- C       (v4)
		  \
		   D -- E      (my-feature) ← 이미 push됨

```

2. 그 후 upstream이 업데이트됨 → v4도 최신화됨 → `my-feature`에 rebase:
```
A -- B -- C -- F -- G       (v4)
                       \
                        D' -- E'    (my-feature, 로컬에서 rebase 됨)

```
> 이 시점에서 **`my-feature`의 커밋 D′, E′는 기존 D, E와 해시가 다름.**

3. 그리고 **새 커밋 F′를 추가**
```
A -- B -- C -- F -- G       (v4)
                       \
                        D' -- E' -- F'    (my-feature)

```


### 그럼 왜 force push가 필요한가?

`git push`는 기본적으로 **fast-forward push**만 허용함. 즉, 현재 `origin/my-feature`의 커밋들이 **로컬 브랜치의 조상이어야** 함.

하지만 현재 상황은?
- `origin/my-feature`: D -- E
- `local my-feature`: D′ -- E′ -- F′

**문제:**  
D′ ≠ D, E′ ≠ E → 완전히 다른 커밋 체인  
→ Git은 "이건 이전 커밋을 덮는 거네?" 라고 판단함

그래서 에러:
```
! [rejected]        my-feature -> my-feature (non-fast-forward)
```

### 해결 방법
```
git push --force-with-lease origin my-feature
```
- 이전 푸시와 비교해서 **다른 사람이 그 브랜치를 수정하지 않았을 때만 강제로 덮어씌움**
- 협업 시 가장 안전한 force push 방식

---

### 핵심 요약

| 상황                         | force push 필요 여부                 |
| -------------------------- | -------------------------------- |
| 기존 커밋 그대로이고, 새로운 커밋만 추가    | ❌ (fast-forward 가능)              |
| 기존 커밋이 rebase로 변경됨 (해시 변경) | ✅                                |
| rebase 이후 새로운 커밋 추가됨       | ✅                                |
| 브랜치를 merge한 경우             | ❌ (보통은 fast-forward 또는 merge 커밋) |

---

### 팁: push 전에 항상 히스토리 확인하고 싶다면?

`git log --oneline --graph --all`

→ 어떤 커밋이 rebase됐고, 어디서 분기됐는지 한눈에 보여줌