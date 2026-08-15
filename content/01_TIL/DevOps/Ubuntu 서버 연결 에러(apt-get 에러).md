---
date: 2025-06-05
tags:
  - til
  - devops
  - infra
  - linux
---
1. WSL을 종료
`wsl --shutdown`

2. Ubuntu 안에서 `/etc/resolv.conf` 수정
`echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf`

3. WSL 재실행
