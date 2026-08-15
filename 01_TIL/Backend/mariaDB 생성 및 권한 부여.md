---
date: 2025-09-10
tags:
  - til
  - backend
  - mariadb
---
# ❓ Information
* mariaDB 생성 및 권한 부여

---
# ❗ Relevant data
## 📦 Information Resources
[velog: mariaDB 생성 및 권한 부여](https://velog.io/@rara_kim/Database-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-%EC%83%9D%EC%84%B1-%EB%B0%8F-%EA%B6%8C%ED%95%9C%EB%B6%80%EC%97%AC)

# 🔰 Content ->  

```bash
mysql -u root -p   # 특정 포트를 지정하려면 -p(포트번호) 추가
```

```sql
SHOW DATABASES;

CREATE DATABASE 데이터베이스명;

CREATE USER '아이디'@'%' IDENTIFIED BY '비밀번호';
CREATE USER '아이디'@'localhost' IDENTIFIED BY '비밀번호';

GRANT ALL PRIVILEGES ON 데이터베이스명.* TO '아이디'@'%' IDENTIFIED BY '비밀번호';
GRANT ALL PRIVILEGES ON 데이터베이스명.* TO '아이디'@'localhost' IDENTIFIED BY '비밀번호';

FLUSH PRIVILEGES;
```

``` sql
DROP USER '아이디'@'%' IDENTIFIED BY '비밀번호';
DROP DATABASE 데이터베이스명;
```

