
## 구조

DB: postgresql 
BE: django(DRF)
FE: react

## 배포 개요

git clone 이후 각 project 경로에서 해당 script 실행

1. **Create Docker Network
	`docker network create projectname-network`
	- Usage: DB, BE, FE Container간 통신
2. **DB Deploy**
	`docker-compose up -d`
	- Detail
		1. Create Table
		2. Insert data (csv)
3. **BE Deploy
	`docker-compose up -d`
	- Detail
		1. requirements 설치
		2. project 복사
		3. uvicorn server 실행
4. **FE Deploy**
	`docker-compose up -d`
	- Detail
		1. project 빌드
		2. nginx server 실행


## 1️⃣ Docker Network 구성
### Purpose
소스 수정이 있는 것만 재배포 가능하도록 개별 배포할 것임. 
따라서 Container 간 통신을 위한 network 필요.

### Command
```bash
$ docker network create projectname-network

$ docker network ls
NETWORK ID     NAME                         DRIVER    SCOPE
3b0a21b49731   projectname-network         bridge    local
```


## 2️⃣ DB Container 구성

### Structure
```bash
├── csv/                   # Data
├── projectname_init.sql          # DDL
├── insert_data.sql        # DML
├── docker-compose.yml
```

### docker-compose
1. Base Image 로 postgres 설정
2. volume 설정
3. `docker-entrypoint-initdb.d` 경로에 `.sql`, `.csv` 파일 복사

> Postgres 공식 이미지의 초기화 메커니즘:
> - `postgres:15` 이미지는 `docker-entrypoint.sh`라는 엔트리포인트 스크립트를 가지고 있음.
> - 초기화 시점에 `docker-entrypoint.sh`는 `/docker-entrypoint-initdb.d/` 경로에 있는 모든 `*.sql`, `*.sql.gz`, `*.sh` 파일들을 찾아 실행함.

```yml
networks:
  projectname-network:
    external: true

services:
  db:
    image: postgres:15
    container_name: postgres_db
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: qwer1234
      POSTGRES_DB: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./projectname_init.sql:/docker-entrypoint-initdb.d/projectname_init.sql
      - ./insert_dummy_data.sql:/docker-entrypoint-initdb.d/insert_dummy_data.sql
      - ./csv:/csv
    ports:
      - '5432:5432'
    networks:
      - projectname-network

volumes:
  postgres_data:
```

#### csv Insert 방식
1. 임시 테이블 생성 (모든 컬럼 TEXT)
2. 임시 테이블에 CSV 데이터 로드
3. 데이터 변환하여 최종 테이블에 삽입
```sql
CREATE TEMP TABLE temp_patient_vital (
    identifiy_id TEXT,
    register_id TEXT,
    ...
);

COPY temp_patient_vital
FROM PROGRAM 'for f in /csv/horizon_0/data_S-*.csv; do tail -n +2 "$f"; done'
WITH (FORMAT csv);

INSERT INTO public.patient_vital 
(patient_id, register_id, ...)
SELECT 
    identifiy_id,
    register_id,
    ...
FROM temp_patient_vital;
```


## 3️⃣ BE Container 구성

### .env.prod
1. `DATABASE_URL` 의 host 에 DB Container 의 service name 지정
2. `ALLOWED_HOSTS` 에 BE Container 의 service name 추가
```env
# Development Environment
DEBUG=True
SECRET_KEY=your-secreat-key

# Database
DATABASE_URL=postgres://projectname:qwer1234@db:5432/projectname

# Allowed Hosts
ALLOWED_HOSTS=backend,0.0.0.0
```

### Dockerfile
1. requirements 설치
2. project 복사
3. uvicorn server 실행
```Dockerfile
FROM python:3.10-slim

WORKDIR /app

# Copy requirements file
COPY requirements.txt /app

# Install Python dependencies
RUN pip install --upgrade pip && \
    pip install -r requirements.txt

# Copy application code
COPY . .

# The environment variable ensures that the python output is set straight
# to the terminal without buffering it first
ENV PYTHONUNBUFFERED=1

CMD ["uvicorn", "config.asgi:application", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

### docker-compose
1. Dockerfile 빌드
2. `.env.prod` 파일 사용
```yml
version: '3.8'

networks:
  projectname-network:
    external: true

services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: backend
    env_file:
      - .env.prod
    ports:
      - '8000:8000'
    networks:
      - projectname-network
    restart: unless-stopped
```


## 4️⃣ FE Container 구성

### Nginx
1. 루트 디렉토리 경로 지정
2. SPA routing 설정
3. BE proxy 설정 (BE Container 의 service name 을 host 로 지정)
```conf
server {
    listen 80;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://backend:8000/;
    }
}
```

### Dockerfile
1. package  설치
2. project 빌드
3. nginx custom conf 복사
```dockerfile
# 1단계: build (package-lock 기준)
FROM node:22 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build 

# 2단계: serve
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

### docker-compose
1. Dockerfile 빌드
```yml
version: "3.8"

networks:
  projectname-network:
    external: true

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: frontend
    ports:
      - "3000:80"

    networks:
      - projectname-network
```
