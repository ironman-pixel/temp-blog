---
date: 2025-09-22
tags:
  - til
  - devops
  - infra
  - crontab
---
# ❓ Information
* Cron의 기본적인 사용 방법과 실용적인 예제 소개

---
# ❗ Relevant data
## 📦 Information Resources
[naverblog: Cron 사용 가이드](https://m.blog.naver.com/yak-bang/223373744270)


# 🔰 Content ->  
리눅스에서 특정 작업을 자동화 하여 반복적으로 실행하고자 할 때 사용.

## Cron 이란
시간 기반의 작업 스케줄러

crontab 파일에 정의됨.
```bash
분 시 일 월 요일 명령어
```

- 분: 0 - 59
- 시: 0 - 23
- 일: 1 - 31
- 월: 1 - 12
- 요일: 0 - 6 (일요일=0)
- 명령어: 실행할 스크립트나 명령어


## Cron 작업 설정하기

crontab 파일을 편집모드로 연다.
```bash
crontab -e
```

### Cron 예제
- 매일 자정에 백업 스크립트 실행하기
```bash
0 0 * * * /home/user/backup.sh
```
- 매주 월요일 오전 9시에 이메일 보내기 스크립트 실행하기
```bash
0 9 * * 1 /home/user/send_email.sh
```


## Cron 작업 관리
- Cron 작업 목록 보기: crontab -l
- Cron 작업 삭제하기: crontab -r
- 특정 Cron 작업 수정하기: crontab -e를 사용하여 편집


## Cron 작업에 로그 남기기
작업이 예상대로 실행되고 있는지 확인하고 문제를 진단하는데 매우 유용함.
기본적인 방법은 작업의 출력을 파일로 리다이렉션 하는것임.

### 1. 기본 로그 남기기
매일 자정에 `/home/user/backup.sh` 스크립트를 실행하고 그 결과를 `/home/user/backup.log` 파일에 남기려면 다음과 같이 crontab에 작성함:
```bash
0 0 * * * /home/user/backup.sh > /home/user/backup.log 2>&1
```
이 명령어는 `backup.sh` 스크립트의 표준 출력(standard output, **stdout**)과 표준 에러 (standard error, **stderr**) 모두를 `backup.log` 파일로 리다이렉션 한다.
`2>&1` 은 표준 에러 출력을 표준 출력과 같은 곳으로 리다이렉션 하라는 의미이다.

### 2. 날짜별 로그 파일 생성
로그 파일이 너무 커지는 것을 방지하거나 날짜별로 로그를 구분하고 싶다면, 로그 파일 이름에 날짜를 포함시킬 수 있다. 
```bash
0 0 * * * /home/user/backup.sh > /home/user/backup_$(date +\%Y-\%m-\%d).log 2>&1
```
이 경우 매일 새로운 로그 파일이 생성됨.
`date +\%Y-\%m-\%d`는 현재 날짜를 년-월-일 형식으로 출력하는 명령어이고, crontab 파일 내에서 `%` 문자는 특별한 의미를 가지므로 앞에 `\`를 붙어 이스케이프 처리해야 한다.

