---
date: 2025-06-03
tags:
  - til
  - dev-tools
  - git
---
# verbose SSH로 push 시도

```
GIT_SSH_COMMAND="ssh -v" git push -u origin main
```

이렇게 하면 실제 어떤 키를 시도하는지, 어떤 인증 실패가 일어나는지 로그로 나옴

