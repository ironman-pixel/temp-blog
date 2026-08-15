---
date: 2025-12-03
tags:
  - til
  - dev-tools
---
# ❓ Information
* CRLF, LF 차이

---

# 🔰 Content ->  

### Carriage Return(CR) (프로그래밍 언어로 `\r`)

말 그대로 캐리지를 리턴한다는 뜻이다. 타자기를 사용하는 모습을 보면 움직이던 부분을 다시 앞으로 이동시키는 모습을 볼 수 있다. 이 행동을 Carriage Return이라고 한다. 이때 커서가 해당 라인의 제일 앞쪽으로 이동하게 된다. 우리가 글을 쓰다가 `Home` 버튼을 누르게 되는 상황과 비슷하다.

### Line Feed(LF) (프로그래밍 언어로 `\n`)

커서는 그대로 있으면서 종이만 올린다. 즉, 줄을 바꾼다는 뜻이다. 우리가 키보드로 아래쪽 방향키를 누를때의 느낌과 비슷하다고 볼 수 있다.

우리가 흔히 줄바꿈을 할때 많이 쓰는 엔터키는 캐리지 리턴과 라인 피드가 합쳐진 것이다. (CRLF) 아랫줄로 내려가서 제일 앞쪽으로 이동하는 것. 이게 엔터이다.

CRLF는 `\r\n` 로 처리되기 때문에 줄바꿈을 할때마다 4바이트를 사용하고 LF는 `\n`로 처리되어 2바이트를 사용한다. 줄바꿈을 할때마다 2배정도 차이가 나기 때문에 보통 LF를 권하고 있다.


### Windows 에서 format때문에 파일 실행이 안되거나 할 때 해결방법

높은 활률로 이런 글로벌 설정이 되어있을것이다
```bash
git config --global core.autocrlf true
```

위 설정은 git clone 받을 당시에 파일들을 CRLF로 변환한다
즉, 저장소 안에 LF로 들어가 있어도 clone 하면서 CRLF로 바뀐다는것이다

**문제는 쉘 스크립트나 SSH private key 등의 파일은 CRLF가 섞이면 정상 실행 되지 않는다**

임시 방편으로 문제를 일으키는 파일만 CRLF -> LF 으로 바꾸는 명령은 다음과 같다
```bash
# 파일명은 알아서 수정
dos2unix SRC/ssh/id_rsa
```

vim에서 바꿀수도 있다 
```bash
:set ff=unix
```

하지만 다음과 같이 그냥 글로벌 설정을 바꿔놓는게 좋다
```bash
git config --global core.autocrlf input
```