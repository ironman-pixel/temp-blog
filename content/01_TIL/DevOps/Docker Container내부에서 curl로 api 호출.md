---
date: 2025-09-25
tags:
  - til
  - backend
  - django
---
# ❓ Information
* Linux 서버 위에 Docker Network에서 Django Container, Nginx React Container가 돌고 있다. 
  POST 요청 /accounts/login 엔드포인트에서 email, password를 받아서 인증을 처리하는데 setCookie 로 refresh token을 등록하고, access token은 response 에 담아서 넘기고 있다. 
  access token 하나를 알고 싶어서 curl 호출을 하려고 한다.

---

# 🔰 Content ->  
## 1️⃣ Install

Container에 접속
```bash
docker exec -it {container명} /bin/bash
```

curl 설치
```bash
apt update && apt install -y curl
```
## 2️⃣ Command 

curl로 endpoint 호출
```bash
curl -X POST http://{service명}:8000/accounts/login/   -H "Content-Type: application/json"   -d '{"email":"...", "password":"..."}'
```

