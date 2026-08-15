---
date: 2025-12-13
tags:
  - course
  - bash
  - shell-script
  - linux
---
## tar
```bash
# create
tar -cvzf 압축할파일명 압축할디렉토리/파일

# extract
tar xvzf 압축파일명
```

## mariabackup
```bash
mariabackup \
--backup \
--no-lock \
--target-dir=백업파일을 저장할 디렉토리
# 아래는 생략 가능
--host=백업할호스트ip \
--port=3306 \
--user=유저명 \
--password=비번 \
```

백업하는 동안 업데이트된 내용, 트랜잭션들이 `ib_logfile0` 에 쌓인다
이 파일도 백업파일에 포함해야 한다

```bash
mariabackup
--prepare \
--target-dir=위에서 지정한 디렉토리
# 아래는 생략 가능
--host=백업할호스트ip \
--port=3306 \
--user=유저명 \
--password=비번 \
```

복구하는방법
```bash
mariabackup \
--move-back \
--user=유저명
--password=비번 \
--target-dir=백업한 디렉토리 \
--data-dir=복구할 디렉토리
```


### 백업 정책
tar볼로 묶어서 보관
db 덤프는 mariadb 전용 백업 툴 mariabckup 사용

하루 한번 트래픽이 가장 적은 시간 대 (새벽 3시 ~ 5시)

스토리지의 BACKUP 디렉토리
`/mnt/BACKUP/서버의호스트이름`