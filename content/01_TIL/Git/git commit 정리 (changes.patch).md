---
date: 2025-10-15
tags:
  - til
  - dev-tools
  - git
---
# ❓ Information
* git 에서 유용한 changes.patch 기능 발견

---

# 🔰 Content ->  

## 문제 상황

a, b(나), c 사용자가 같은 프로젝트에서 git 작업중.

로컬 main branch 깃 로그 상황:
```
a: commit1
b: commit2
c: commit3
b: commit4
a: commit5
c: commit6
a: commit7
b: commit8
```

원격 main branch 상황:
```
a: commit1
c: commit3
a: commit5
c: commit6
a: commit7
```

rebase해서 아래와 같이 수정하려고 함:
```
a: commit1
c: commit3
a: commit5
c: commit6
a: commit7
b: commit8
```

아직 푸시하지 않은 로컬 커밋을 삭제(rebase), 이 상태로 강제 푸시(--force) 하면 충돌 안남

---
## 해결 방안:

### rebase + force push

일단 git branch 확인

```
git rebase -i HEAD~8
```

에디터에서 지우고 싶은 커밋을 지운다 (물론 원격에 없는것만)
``` 
pick <hash1> commit1
pick <hash2> commit2   <----지워
pick <hash3> commit3
pick <hash4> commit4   <----지워
pick <hash5> commit5
pick <hash6> commit6
pick <hash7> commit7
pick <hash8> commit8
```

git log --oneline 해서 로컬에 로그가 잘 정리 되었나 확인

git push --force (히스토리를 변경 했기에 강제 푸시로 해야함)(내 히스토리로 원격을 덮어쓰는 것이기 때문)

충돌이 안나는 이유: 푸시하지 않은 로컬 커밋만 수정하거나 삭제했기 때문

---
### 패치를 활용

```
git switch main
git diff origin/main > changes.patch
git reset --hard origin/main
git apply changes.patch
git add .
git commit -m "b: 모든 작업을 하나의 커밋으로 정리"
git push origin main
```