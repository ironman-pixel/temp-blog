---
date: 2026-08-15
tags:
  - devops
  - infra
  - server
  - dev-environment
  - linux
---
ubuntu server에서 dev server 세팅 과정


## 1. 새로운 dir에 git clone, Dockerize
운영 중인 컨테이너와 겹치지 않도록 
docker image name, docker container name, service name 을 다르게 수정하여 배포

## 2. nginx settings 에서 새로운 서버 블록 생성
```nginx
server {
    listen 80;
    server_name 172.7.0.57 domainname.kr;

    client_max_body_size 500M;

    location /static/ {
        alias /app/folder/projectname_aws/static/;
        autoindex on;
    }

    location /proj2/static/ {
        alias /app/folder/projectname_aws/static/;
        autoindex on;
    }


    location /media/ {
        alias /media/proj1_media/;
        autoindex on;
    }

    location /dashboard/ {
        proxy_pass http://127.0.0.1:3001/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        #proxy_http_version 1.1;
        #proxy_set_header Upgrade $http_upgrade;
        #proxy_set_header Connection "upgrade";
        # try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }



    location /proj2/ {
        proxy_pass http://127.0.0.1:8005/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }


    location / {
        proxy_pass http://127.0.0.1:8006/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    real_ip_header X-Forwarded-For;
    set_real_ip_from 127.0.0.1;        # Trust requests from localhost (Gunicorn)
    set_real_ip_from 172.17.0.0/16;    # Trust Docker's internal network
    set_real_ip_from 192.168.0.0/16;
    set_real_ip_from 172.7.0.0/16;
    real_ip_recursive on;
}





server {
    listen 8002;
    server_name _;

    client_max_body_size 500M;

    location /static/ {
        alias /app/folder/test/projectname_aws/staticfiles/;
        autoindex off;
    }

    location /media/ {
        alias /media/proj2_media/;
    }

    location / {
        proxy_pass http://127.0.0.1:8008/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```


## 3. 리눅스 방화벽 확인

이 명령으로 현재 서버 외부에서 접근이 허용된 포트를 확인한다
```
sudo ufw status
```

만약 없을 경우 아래와 같이 추가가 가능하지만
```
sudo ufw allow 8007/tcp
sudo ufw reload
```
서버 설정을 변경하는건 위험할수 있으니 열려있는 포트를 사용해서 nginx를 포팅 해준다


## 4. docker 설정을 git이 되돌리지 못하게 막는다

git 이 이 파일을 건드리지 말아야 할 파일로 취급
```
git update-index --skip-worktree path/to/file
```

취소 하는 방법
```
git update-index --skip-worktree path/to/file
```

