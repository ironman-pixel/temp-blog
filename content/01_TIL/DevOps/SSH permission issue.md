---
date: 2025-12-04
tags:
  - til
  - devops
  - infra
  - ssh
---
# ❓ Information
* ssh 권한 문제 해결

---

# 🔰 Content ->  

## Issue
ssh 접속 시도 중에 다음 에러를 만났다
```bash
$ ssh cent3
Bad owner or permissions on /root/.ssh/config
```

## Troubleshooting
SSH는 보안상 `.ssh/config` 파일에 대해 다음 조건을 요구함
1. 권한: 소유자만 읽을 수  있어야 함 (`600`)
2. 소유자: 현재 사용자(root)가 소유해야 함

```bash
-rw-r--r-- 1 1000 1000  579 Dec  4 14:46 config
```

위와 같이 뜬다면 문제가 있는 것이다

아래 명령으로 권한 조정이 필요하다

```bash
chown root:root /root/.ssh/*
# /root/.ssh 디렉토리 자체 권한도 700이 보안상 권장됨
chmod 700 /root/.ssh/
# id_rsa는 600이 필수
chmod 600 /root/.ssh/id_rsa
# authorized_keys는 600 또는 644
chmod 644 /root/.ssh/authorized_keys
```

