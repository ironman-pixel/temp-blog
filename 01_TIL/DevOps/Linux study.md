---
date: 2025-07-13
tags:
  - til
  - devops
  - infra
  - docker
  - linux
---
# ❓ Information
* learn Linux through youtube

---
# ❗ Relevant data
## 📦 Information Resources
[youtube: linux 강좌](https://www.youtube.com/watch?v=dmsZ-0iQrgc&list=PLq8wAnVUcTFU9zLWK-dHWrvTJ0PF8Y0Sf&index=2)



# 🔰 Content ->  

## 리눅스 운영 + 실습 환경 만들기

### 1. 파일 다루기 + 로그 확인

| 내용                                                                             | 목표                       |
| ------------------------------------------------------------------------------ | ------------------------ |
| `cd`, `ls`, `cp`, `mv`, `rm`, `cat`, `grep`, `find`, `less`, `tail -f`, `vi` 등 | 파일/디렉터리 구조 이해, 로그 분석의 기초 |
| `journalctl`, `/var/log/syslog`, `/var/log/messages`                           | 시스템 로그 이해                |

사용중인 포트 확인하기
`sudo netstat -tuln | grep :8000`


---

### 2. 프로세스 & 네트워크 기본

|내용|명령어|
|---|---|
|프로세스 찾기, 죽이기|`ps`, `top`, `htop`, `kill`, `killall`, `pkill`|
|포트 확인|`netstat -tuln`, `ss -lntp`, `lsof -i :포트`|
|파일 찾기|`find`, `locate`|

---

## 서비스 구성 및 자동화

### 3. 기본 서버 소프트웨어 설치 & 설정

| 소프트웨어           | 실습 목표                            |
| --------------- | -------------------------------- |
| JDK, Tomcat     | 설치, 환경변수 등록, `WAR` 배포 실습         |
| MySQL / MariaDB | 설치, DB 생성, 포트 열기, CLI 접속, 테이블 생성 |
| Apache (httpd)  | 설치, 리버스 프록시, 기본 HTML 서비스 제공      |

→ 이 부분에서 APM 환경이 완성됨:  
**JDK + Tomcat + DB + Apache** → 웹 애플리케이션 WAR 배포 → 접속 확인

---

### 4. 서비스 관리 + 부팅 자동실행

|내용|명령어|
|---|---|
|서비스 실행/중지|`systemctl start|
|부팅 시 자동 실행 등록|`systemctl enable`|
|부팅 후 확인|`systemctl is-enabled`|

---

### 5. 배포 자동화 / 백업 / 일정 작업

| 기능       | 도구                                      |
| -------- | --------------------------------------- |
| 배포 스크립트  | bash/zsh script (`scp`, `rsync`, `ssh`) |
| 배치 작업    | `crontab -e` 로 등록                       |
| Git/SVN  | 설치 및 버전관리 테스트                           |
| FTP/SFTP | `vsftpd`, `sftp`, FileZilla 연동          |
| PuTTY    | Windows에서 SSH 접속 실습                     |

---

## 보안 + 서버 튜닝

### 6. 사용자, 그룹, 권한

|내용|명령어|
|---|---|
|사용자/그룹 추가|`adduser`, `usermod`, `groupadd`|
|권한 변경|`chmod`, `chown`, `umask`|
|디스크 마운트 권한|`mount`, `/etc/fstab`, `lsblk`, `df`, `du`|

---

### 7. 방화벽, 보안, 해킹 방어

|항목|설명|
|---|---|
|포트 제어|`ufw`, `firewalld`, `iptables`|
|비밀번호 정책|`/etc/login.defs`, `passwd`, `chage`|
|SSH 보안 강화|`PermitRootLogin no`, 포트 변경|
|침입 탐지|`fail2ban`, `logwatch`, `rkhunter`|
|웹 방화벽|`mod_security` (Apache 모듈) 등|

---

### 8. 도메인 및 이메일

| 항목        | 실습 예시                               |
| --------- | ----------------------------------- |
| 도메인 연결    | `/etc/hosts`, DNS 설정                |
| 메일 서버 세팅  | `Postfix`, `Dovecot` + SMTP/IMAP 설정 |
| 외부 발신 테스트 | Gmail SMTP relay 등                  |
