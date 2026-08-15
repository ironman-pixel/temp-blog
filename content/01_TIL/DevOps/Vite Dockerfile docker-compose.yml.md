---
date: 2025-08-25
tags:
  - til
  - frontend
  - react
  - vite
---
# ❓ Information
* vite 프로젝트를 docker 로 배포하는 방법

---

# Dockerfile (멀티 스테이지 빌드)
```dockerfile
# 1단계: build
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

## 1단계 
1. `package*.json`만 먼저 복사 -> `npm ci` 실행 -> `node_modules` 생성.
2. 이후에 `COPY . .` 로 전체 소스 복사
이렇게 하면 **Docker layer cache** 를 활용할 수 있다.
- `package.json` 안바뀌면 `npm ci` 단계는 캐시됨 -> build 속도 빨라짐.
- 만약 처음부터 `COPY . .` 해버리면, 코드 조금만 바뀌어도 매번 dependency 설치를 새로 해야 되서 비효율적.

## 2단계
### nginx:alpine vs nginx:latest
- `nginx:apine` -> 경량화된 버전 (약 20MB)
- `nginx:latest` -> 표준 버전 (약 130MB 이상)
  👉운영 환경에서는 불필요한 패키지가 빠진 **alpine** 버전을 많이 씀. (보안/배포 속도/스토리지 절약)

### `/usr/share/nginx/html` 이유
- nginx 기본 설정에서 **정적 파일 root 경로**가 `/usr/share/nginx/html` 로 잡혀 있음
- 우리가 `dist` 빌드 결과를 여기에 복사하면 nginx 가 자동으로 index.html + 정적 파일을 서비스함
  👉즉, nginx 디폴트 동작을 그대로 활용하는것

### `/etc/nginx/conf.d/default.conf` 이유
- nginx 기본 이미지는 `/etc/nginx/conf.d/default.conf` 라는 파일을 기본 설정으로 둠.
- 우리가 정의한 `nginx.conf` 를 이 위치에 덮어씌우면 **커스텀 설정** 이 적용됨.
- `/etc/nginx/nginx.conf` 도 있지만, 보통 메인 프레임워크 파일이고 실제 vhost(사이트 설정)는 `conf.d` 폴더에서 불러옴
  👉따라서 프로젝트에서 만든 `nginx.conf` -> `default.conf` 로 복사하는게 일반적인 패턴.

---
# docker-compose.yml
```yml
version: "3.8"

networks:
  projectname-licence-network:
    driver: bridge

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
      - db
```

## `networks: external: true` 
- `external: true` -> 이미 존재하는 네트워크를 사용하겠다는 의미
- `driver: bridge` -> 새로운 네트워크를 docker-compose 실행할 때 만들겠다는 의미.
	  이미 `projectname-license-network` 라는 네트워크를 만들었고, FE/BE/DB를 같은 네트워크에 붙이려는거라면 `external: true` 를 써야함

## `build: contsxt: .`
- `context: .` -> **Docker build context** 를 현재 디렉토리 (`.`) 로 설정.
- 이 경로 안에 있는 모든 파일이 build 과정에서 COPY 가능
- `dockerfile: Dockerfile` 로 특정 Dockerfile 지정
즉,
```yml
build: 
	context: .
	dockerfile: Dockerfile
```
= 현재 폴더에 있는 Dockerfile 로 빌드

## depnedes_on 은 service 이름 기준


# nginx.conf
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

## `try_files` 지시어가 하는 역할
- `location / {...}`  
  -> 모든 요청 경로 (`/` 로 시작하는 것 전부) 에 대해 적용 하겠다는 의미.
- `try_files $uri /index.html;`
  -> 요청이 들어왔을 때 다음 순서대로 확인:
	1. `$uri` -> 클라이언트가 요청한 실제 파일 경로가 존재하는지 확인 (예: `/style.css`, `/main.js`)
	2. 없으면 `/index.html` 로 fallback

### 왜 이렇게 쓰는가? (SPA 라우팅 문제 해결)
Vite + React 같은 SPA 는 라우팅을 클라이언트 측에서 처리함.
예시:
- 사용자가 `/dashboard` 주소로 들어옴.
- 서버 입장에서는 `/dashboard/index.html` 같은 파일이 없음 -> 원래라면 **404 NotFound**.
- 하지만 SPA 는 `index.html` 하나로 시작해서 React Route 같은 클라이언트 라우터가  `/dashboard`를 해석해야함
  👉 그래서 `try_fielx $uri /index.html;` 을 두면,
- 정적 파일(css/js) 요청은 그대로 처리
- 없는 경로 (`/dashboard`, `/settings` 등)는 `/index.html` 을 반환 -> React Router 가 이어받아 처리

### 만약 이걸 안쓰면?
- SPA 라우팅에서 직접 UTL 입력 시 전부 404 발생
- 오직 `/` (루트)에서만 정상적으로 앱이 열림

### 정리
`try_filex $uri /index.html;` 은 **SPA 배포시 필수 설정** 으로 ,
정적 파일은 정상적으로 제공하고, 나머지 모든 경로는 `index.html`로 fallback 시켜서 클라이언트 라우팅이 동적하게 만드는 역할임.