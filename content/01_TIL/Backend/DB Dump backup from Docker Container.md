# ❓ Information
* DB 마이그레이션 관리가 안되는 프로젝트에 투입되어 다음 안건을 제기했다

---

# 🔰 Content ->  

```bash
docker exec projectname_db   mariadb-dump -u root -p'비밀번호' projectname_db   > projectname_db_backup.sql
```

---

I got some idea about DB maintaining 

since we dont have any migration tools 
adding Dev DB changes to Prod DB could be tricky

why dont we keep the migration history manually on the project folder?
kinda like this

example file structure
```sql
projectname_aws/
  db_migrations/
    001_create_user.sql
    002_add_email_index.sql
    003_alter_order_status.sql
```
example sql file
```sql
-- 002_add_email_index.sql
ALTER TABLE users
ADD INDEX idx_users_email (email);
```

so on the prod server side we can apply the changes like this
```bash
docker exec projectname_db   mariadb-dump -u root -p'비밀번호' projectname_db   > /app/folder/projectname_aws/db_migrations/003_alter_order_status.sql
```

the only problem is dbeaver GUI customizations can only be used for testing
and the actual changes must be done in SQL




---


## DB 변경 이력 관리 방식
DB 스키마 변경 내용을 직접 SQL로 작성하여 migration 적용

### 개요
1. 개발 DB 의 변경사항을 migration 파일로 저장
2. git commit, git push
3. 운영 서버에서 git pull
4. 운영 DB 백업
5. 운영 서버에서 migration 파일 실행

### 1. 개발 DB 의 변경사항을 migration 파일로 저장
#### migration 파일 저장 위치:   
```sql
projectname_aws/
  db_migrations/
    001_create_user.sql
    002_add_email_index.sql
    003_alter_order_status.sql
```
- 파일명: `순번_변경 내용.sql`

#### migration 작성 규칙:   
- SQL로 작성
- 변경 사항은 반드시 파일로 남기고 Git Commit
- Dev / Prod 모두 같은 SQL 파일 사용

### 2. git commit, git push
### 3. 운영 서버에서 git pull
### 4. 운영 DB 백업
마이그레이션 적용 전, **반드시 DB 전체를 백업한다**   
파일명: `날짜_before_순번.sql`
```bash
docker exec projectname_db \
  mariadb-dump -u root -p'비밀번호' projectname_db \
  > /app/folder/projectname_aws_db/db_backups/2024xxxx_before_003.sql
```

### 5. 운영 서버에서 migration 파일 실행
마이그레이션 적용 순서는 파일 번호 기준
```bash
docker exec -i projectname_db \
  mariadb -u root -p'비밀번호' projectname_db \
  < /app/folder/projectname_aws/db_migrations/003_alter_order_status.sql
```

### 기타 사항
#### GUI 도구 사용 원칙 (DBeaver 등)
Dev DB에서 테스트용으로만 사용   
실제 스키마 수정은 반드시 SQL 파일로 작성하여 실행   
Prod DB에 GUI로 직접 수정 금지

#### 백업 기반 롤백
본 프로젝트의 DB 롤백은 자동화되어 있지 않으며,   
사전 백업 복원으로 수행
```bash
docker exec -i projectname_db \
  mariadb -u root -p'비밀번호' projectname_db \
  < /app/folder/projectname_aws_db/db_backups/2024xxxx_before_003.sql
```
