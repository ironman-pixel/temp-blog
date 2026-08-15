---
date: 2025-09-09
tags:
  - til
  - devops
  - infra
  - docker
---
# ❓ Information
* docker 명령어 모음

---

# ❗ Relevant data
## 📦 Information Resources
> [Docker 설명 블로그](https://www.lainyzine.com/ko/article/how-to-name-a-docker-image/)
> 
> [Docker 입문 튜토리얼 블로그](https://www.lainyzine.com/ko/article/docker-tutorial/)

# 🔰 Content ->  e로 image 만들기

1. Dockerfile 생성
```
FROM ubuntu:latest
RUN apt update
RUN apt install -y git
```

2. image 생성
```
$ docker build -t ubuntu:git .
```

`-t` 는 이미지 이름을 지정하는 옵션

# Docker image 이름 변경

1. image 이름 추가
```
$ docker tag ubuntu:git tools:git
```

2. image 목록 확인
```
$ docker images | grep git
tools     git    bb7aff0f9054    5 minutes ago    284MB 
ubuntu    git    bb7aff0f9054    5 minutes ago    284MB
```

3. 기존 이미지 삭제
```
$ docker rmi ubuntu:git
Untagged: ubuntu:git
```

4. 확인
```
$ docker images | grep git
tools     git    bb7aff0f9054    5 minutes ago    284MB 
```

# Docker Hub에 image push

1. docker login
```
$ docker login
```

2. image 이름 형식 갖추기 (갖추고 있으면 스킵)
```
$ docker tag tools:git roxanne0808/tools:git
```

3. docker hub 에 push
```
$ docker push roxanne0808/tools:git
The push refers to repository [docker.io/roxanne0808/tools]
7d81155787e6: Pushed
aa9581a8697c: Pushed
e4da4fde4d34: Pushed
d9d352c11bbd: Pushed
git: digest: sha256:65ea5d71c13b7e9eb7ae90b95f290191e129410906cae3f18cc29d8caadcaf1e size: 855
```

4. 이제 누구나 pull 받을 수 있음
```
$ docker pull roxanne0808/tools:git
```


---

# Docker 명령어 형식

```
docker <SUBCOMMAND> (<OPTIONS>)
```

## ps -a 에서 필터링

```
$ docker ps -a --filter volume=mysql_data
```

## run 명령어 형식

```
docker run (<OPTIONS>) <IMAGENAME> (<COMMAND>)
```

### 백그라운드 실행
```
$ docker run -d --name nginx-8080 -p 8080:80 nignx
```

### 퍼블리시

1. html 파일을 서빙하는 nginx 예시를 들겠다.
   *마지막에 `:ro`는, 컨테이너에서 이 디렉터리에서 쓰기를 할 수 없도록 읽기 전용으로 마운트하라는 의미*
```
$ cd wherever

$ vi index.html
  <html> 
  <h1>Hello World</h1> 
  </html>

$ docker run -d --name nginx-static -p 8080:80 -v $(pwd):/usr/share/nginx/html:ro nginx
```

### 셸을 실행하거나 명령어 출력을 바로 보고 싶을 때

1. `-it` 옵션 필요
```
$ docker run -it debian:12 bash
```

### Container 종료 시 Container 삭제

1. `--rm` 옵션 필요
```
$ docker run -it --rm debian:12 bash
```

### Volume 추가

1. `-v` 옵션 필요
```
# Windows
$ docker run -it --rm -v ${pwd}:/data linuxserver/ffmpeg:latest

# macOS / Linux
$ docker run -it --rm -v $(pwd):/data linuxserver/ffmpeg:latest
```

```
[호스트의 디렉터리]:[컨테이커 내부의 디렉터리]
```

### 포터블 앱으로 사용

1. ffmpeg를 예시로 들겠다. 
   `--rm` 으로 명령어 실행 이후 컨테이너 삭제
   `-v`로 볼륨 연결
   *해당 이미지는 Entrypoint를 제공해서 명령어 맨 앞에 ffmpeg를 붙이면 안됨*
```
$ cd wherever

$ wget https://www.lainyzine.com/ko/images/docker-logo.png

$ docker run -it --rm \ 
-v $(pwd):/data \ 
linuxserver/ffmpeg:latest \ 
-loop 1 -i /data/docker-logo.png -c:v libx264 -t 15 -pix_fmt yuv420p -vf scale=320:240 /data/docker-logo.mp4
```

### DB server 로 사용

1. MySQL 서버를 예시로 들겠다.
   *`-e`는 환경변수 설정하는거다.*
   *root 유저를 pw: 1234로 생성하는 명령이다.*
   *생성 전에 사용 가능한 port인지 확인하고 쓸것. 왜냐면 사용중인 port에 매핑 했는데 오류가 안나는 경험을 함...*
```
$ docker run -d --name mysql \
	-p 3306:3306 \
	-v mysql_data:/var/lib/mysql \
	-e MYSQL_ROOT_PASSWORD=1234 \
	mysql:9.3
```

## Container 중지/삭제 명령어

```
$ docker stop <CONTAINER> -- 종료
$ docker container kill <CONTAINER> -- 강제종료

$ docker rm <CONTAINER> -- 삭제
$ docker rm -f <CONTAINER> -- 종료 및 삭제

$ docker container prune -- 중지된 모든 container 삭제
```

## Container Log 명령어

### 전체 log 한번만 출력
```
$ docker logs <CONTAINER>
```

### 최근 로그 10줄 출력하고 대기
```
$ docker logs -n 10 -f <CONTAINER>
```

### 같은 방식으로 docker compose 파일이 있는 디렉터리에서 로그 확인 가능
```
$ docker compose logs
```

## docker-compose 

### docker-compose.yml 예시

```
services:
  mysql:
    image: mysql:8.0
    volumes:
      - mysql-data:/var/lib/mysql
    environment:
      MYSQL_RANDOM_ROOT_PASSWORD: true
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: awesome-wordpress-password

  wordpress:
    depends_on:
      - mysql
    image: wordpress:latest
    ports:
      - "3000:80"
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: awesome-wordpress-password

volumes:
  mysql-data:
```

### compose 파일 실행/종료

컨테이너를 실행/종료 한다고 보면 됨
```
$ docker compose up -d

$ docker compose down
```

코드를 수정한 경우 `--build`를 붙여야 변경 사항이 반영됨

- `docker-compose up -d` → 이미지 그대로, 실행만.
- `docker-compose up --build -d` → 이미지 빌드 후 실행.
- `docker-compose build --no-cache` → 특별히 캐시 무시할 필요 있을 때만 사용 (requirements나 base image 꼬였을 때).


이렇게 실행하면 컨테이너 이름이 이렇게 생길거임
> `wordpress-mysql-1`
> `wordpress-wordpress-1`

```
<디렉터리 이름>-<서비스이름>-<자동으로 붙는 인덱스>
```

### compose 명령 옵션

```bash
# Detached mode
$ docker-compose up -d

# Build images
# 소스 수정 이후 container 다시 올릴때 사용
$ docker-compose up --build -d

# stop and recreate all containers
$ docker-compose up --force-recreate -d
```

```bash
# Build images without cache 
# 패키지 버전을 최신화 하는게 필요하거나 빌드 오류가 난 경우 사용
$ docker-compose build --no-cache
```

## Network 명령어

### inspect

```bash
# 네트워크 조회
$ docker network ls

# 네트워크 생성
$ docker network create my-network

# 네트워크 상세 정보 확인
$ docker network inspect my-network
```

### connect/disconnect

```bash
$ docker network connect {network name} {container name}

$ docker network disconnect {network name} {container name}

# connection test
$ docker exec {container name} ping {container name}
```

### prune

```bash
# 미사용 네트워크 청소
$ docker network prune
```


## Volume 명령어

### create
```bash
$ docker volume create {volume name}
```

### ls
```bash
$ docker volume ls
```

### inspect
```bash
# Mountpoint 항목을 보면 해당 볼륨이 컴퓨터의 어느 경로에 생성되었는지 확인 가능
$ docker volume inspect {volume name}
[
    {
        "CreatedAt": "2020-05-09T17:03:46Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/our-vol/_data",
        "Name": "our-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

### inspect
```bash
$ docker inspect {volume name}
```

### rm
```bash
$ docker volume rm {volume name}
```

### prune
```bash
$ docker volume prune
```

### 컨테이너 상에서 볼륨을 영구적으로 사용할 수 있는 방법

#### 1. 호스트 볼륨 공유
호스트 디렉토리를 컨테이너 내에 마운트
```bash
# MYSQL 컨테이너 실행
$ docker run -d --name wordpressdb_hostvolume \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=workpress \
-v /home/wordpress_db:/var/lib/mysql \
mysql:5.7

# 워드프레스 웹 서버 컨테이너 생성
$ docker run -d \
-e WORDPRESS_DB_PASSWORD=password \
--name wordpress_hostvolume \
--link wordpressdb_hostvolume:mysql \
-p 80 \
wordpress
```

**`-v 옵션`으로 `/home/wordpress_db:/var/lib/mysql`로 설정** 
**호스트의 `/home/wordpress_db` 디렉터리와 컨테이너의 `var/lib/mysql` 디렉터리를 공유한다는 뜻**

#### 2. 볼륨 컨테이너
_내 생각에는 거의 안쓸거 같아서 패스_

#### 3. 도커 볼륨
```bash
# create volume
$ docker volume create {volume name}

# mount volume
$ docker run -d \
--name test-mysql \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=test-mysql \
-p 3306:3306 \
-v {volume name}:/var/lib/mysql \
mysql:5.7
```


---

# Docker image 의 태그 규칙

docker-library 공식 문서를 보면 몇몇 이미지에서 동일한 **tag 규칙**을 따르며 배포 함
- name:version
- name:version-slim
- name:version-alpine
- name:version-windowsservercore

## name:version
가장 기본이 되는 이미지.
**배포 뿐 아니라  base 이미지로 사용되기 위해서 설계**됨.

### 뒤에 buster, stretch 가 붙는 경우
**Debian** 계열 이미지를 베이스로 배포 되었음. 
이미지가 어떤 릴리즈를 기반으로 배포 되었는지 알 수 있는 척도.
이 안에 몇가지 패키지를 추가로 설치하고 싶을 때 배포 시 **발생하는 에러를 최소화 하기 위해서는 명시**한것임.

## name:version-slim
파이썬을 실행하는데 필요한 최소한의 패키지만 설치됨.
순수하게 파이썬만 실행할 수 있는 이미지.
따라서 배포 환경에 용량 제한이 심하거나 **순수하게 파이썬만 실행하는 환경이 아니라면 사용하지 않는것이 좋다**.

## name:version-alpine
**alpine linux** 를 기반으로 제작된 이미지.
alpine 은 OS 용량 자체가 매우 작기로 유명함.
주의할점: alpine 은 glibc 가 아니라 musl libc 를 사용하기 때문에 **가끔 C 라이브러리 의존성에 이슈가 있을수 있다는 단점**이 있다.

## name:version-windowsservercore
**windows server** 이미지를 기반으로 배포 되었다.


