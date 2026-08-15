---
date: 2026-01-02
tags:
  - til
  - dev-tools
  - git
---
# ❓ Information
* .gitignore에 추가하지 않고 특정 파일의 변경사항 무시하는 방법

---
# ❗ Relevant data
## 📦 Information Resources
[blog: .gitignore에 추가하지 않고 변경사항 무시하는 방법](https://blog.joe-brothers.com/ignore-git-changes-without-modifying-gitignore/_)

# 🔰 Content ->  

## 아직 git에서 트래킹되지 않는 파일인 경우
`.git/info/exclude` 에 무시할 파일을 추가한다

```shell
mkdir -p .git/info
echo {무시할_파일} > .git/info/exclude
```


## 이미 git에서 트래킹하는 파일인 경우
이미 기존에 커밋되어서 트래킹되는 파일인데, 로컬 변경사항만 무시하고 싶은 경우엔 아래 명령어를 사용한다

```shell
git update-index --assume-unchanged {무시할_파일}
```

만약 나중에 다시 git에 변경사항을 반영하고 싶다면, 아래 명령어로 되돌리면 된다

```shell
git update-index --no-assume-unchanged {다시_반영할_파일}
```

무시된 파일 목록 확인하는 명령

```shell
git ls-files -v | grep "^S"
```