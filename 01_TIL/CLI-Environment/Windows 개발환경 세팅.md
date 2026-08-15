---
date: 2025-11-17
tags:
  - til
  - dev-tools
  - windows
---
# ❓ Information
* Windows 10 환경 C드라이브 포맷한다음 환경세팅한 과정 기록
* 2025.11.14에 세팅하고 2025.11.17에 정리하는거라... 아마 빠진 내용이 있을수도 있다는점

---

# 🔰 Content ->  

## Windows Terminal 설치
MS Store 에서 설치

이후로 모든 명령은 PowerShell에서 진행

---
# 1. WSL 환경 세팅

## wsl2 설치
각 단계마다 PC 재시작 말고 완전종료&전원On 으로 껐다 켜주기
### 1. PC 환경 설정
```bash
# "제어판"→"프로그램"→"프로그램 및 기능"→"windows 기능 켜기/끄기"
# Linux용 Windows 하위 시스템 on
$ dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all
# 가상머신 플랫폼 on
$ dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all
```

### 2. Ubuntu 설치
MS Store 에서 `Ubuntu 24.04.2 LTS` 설치

윈도우 탐색기에서 Ubuntu 실행하고 Enter username 나올때까지 대기
username, password 세팅

### 3. WSL2 설치
PowerShell
```bash
$ wsl --install
$ wsl --set-default-version 2

$ wsl -v -l
WSL 버전: ...
...
```

```bash
$ wsl --status
```
## Docker Desktop 설치
MS Store 에서 설치

## WSL 터꾸
중간중간 터미널을 재시작 해야 적용됨
### 1. zsh 설치
```bash
$ sudo apt update
$ sudo apt install zsh -y

# 기본 쉘을 zsh로 변경
$ chsh -s $(which zsh)
# 터미널 재시작
```

### 2. oh-my-zsh 설치
```bash
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

$ echo $ZSH_CUSTOM
/home/rox/.oh-my-zsh/custom
```

만약 여기서 잘 안되면 다음 원인들이 있다
- WSL이 재시작될 때마다 `/etc/resolv.conf`를 Windows에서 동적으로 생성
- DNS 서버가 간헐적으로 응답하지 않거나, VPN/프록시 환경 때문에 일시적으로 실패
- ping은 ICMP라 성공하지만, git/curl은 HTTPS라 DNS를 정확히 확인해야 함

해결방법: `/etc/resolv.conf` 고정
1. `/etc/wsl.conf` 생성/편집:
```bash
$ sudo nano /etc/wsl.conf
```

내용 추가:
```ini
# 이건 원래 있을것임
# [boot]
# systemd=true

[network]
generateResolvConf = false
```

2. 기존 resolv.conf 제거:
```bash
$ sudo rm /etc/resolv.conf
```

3. 새 resolv.conf 생성:
```bash
$ sudo nano /etc/resolv.conf
```

내용:
```nginx
nameserver 8.8.8.8
nameserver 8.8.4.4
```

4. WSL 재시작
wsl 말고 다른 쉘에서 실행
```bash
$ wsl --shutdoen
$ wsl
```


### 3. Powerlevel10k 설치
```bash
$ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

### 4. ~/.zshrc 수정
IDE가 wsl에 연결 되려면 plugin(wsl) 이 설치되어 있어야함
그냥 nano로 하는게 빠름
```bash
$ nano ~/.zshrc

ZSH_THEME="powerlevel10k/powerlevel10k"
```

### 5. 폰트 설치
[https://www.nerdfonts.com/](https://www.nerdfonts.com/) 에서 “MesloLGS NF” 다운로드
다운로드 한거 더블클릭해서 실행하면 자동으로 pc에 깔림
설치 후, Windows Terminal → Settings → Ubuntu 프로필 → Font → **MesloLGS NF** 로 변경

### 6. p10k configure
```bash
$ p10k configure
```

### 7. plugin 추가
git clone 받고
```bash
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

설정파일에 등록을 한다
```bash
$ vi ~/.zshrc

# plugins = ( 
# 			git 
# 			zsh-autosuggestions 
# 			zsh-syntax-highlighting 
# 			)
```



## WSL 환경에 이것저것 설치
[[curl 설치 사용 이유]]
### 1. NVM
```bash
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
```

적용:
```bash
$ source ~/.zshrc
```

버전 설치 및 기본 지정:
```bash
$ nvm install --lts
$ nvm use --lts
$ nvm alias default lts/*
$ node -v
```

### 2. Python
도구 비교 (사실 현업에서 asdf 많이 쓴다고 하던데... 일단 나는 uv로)

| 도구    | 장점                     | 단점                    | 추천 상황                         |
| ----- | ---------------------- | --------------------- | ----------------------------- |
| pyenv | 오래된 안정성, virtualenv 호환 | 빌드 느림, PATH 꼬임        | 기존 프로젝트, 여러 버전 관리 필요          |
| uv    | 빠른 설치, 최신 Python, 간단   | 커뮤니티 작음, WSL 호환 체크 필요 | Python 전용 최신 환경, 빠른 설치 원하는 경우 |
| asdf  | 다중 언어 통합               | Python 별도 plugin 필요   | Node+Python+Java 통합 관리        |

uv 설치:
```bash
# 설치 스크립트
$ curl -fsSL https://raw.githubusercontent.com/python-build/uv/main/install.sh | bash

# 초기화
$ echo 'export PATH="$HOME/.uv/bin:$PATH"' >> ~/.zshrc
```

### 3. `~/.zshrc` 내용 확인
```bash
# =====================
# NVM (Node)
# =====================
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

# =====================
# UV (Python)
# =====================
export PATH="$HOME/.local/bin:$PATH"
```

### 4. pip3 설치
```bash
# 실행 결과 pip 관련 모듈이 아예 빠진 상태라고 뜸
$ python3 -m ensurepip --version
/usr/bin/python3: No module named ensurepip

# 설치하고
$ sudo apt install python3-pip

# 결과 확인
$ which pip3
/usr/bin/pip3
$ pip3 --version
pip 24.0 from /usr/lib/python3/dist-packages/pip (python 3.12)
```

### 5. 명령어 설치
```bash
$ sudo apt update

$ sudo apt install pipx

$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
info: downloading installer
$ source $HOME/.cargo/env

$ sudo apt install htop

$ sudo apt install bat
$ ln -s /usr/bin/batcat ~/.local/bin/bat

$ sudo apt install eza
# ~/.zshrc
# alias ls="eza"
# alias lt="eza -aT -L1 --color=always --group-directories-first --icons"
# alias lt2="eza -aT -L2 --color=always --group-directories-first --icons"
# alias lt3="eza -aT -L3 --color=always --group-directories-first --icons"

$ sudo apt install ranger
# ~/.zshrc
# export EDITOR=vim

$ sudo apt install most
# ~/.zshrc
# export PAGER="most"
# export GROFF_NO_SGR=1

$ sudo apt install fzf
# ~/.zshrc
# alias fzfp="fzf --preview 'bat --color=always {}' --preview-window=right:60%"

# just for fun
$ pipx install smassh

$ cargo install theattyr

$ cargo install tgt
$ sudo apt install libc++1
```

### 6. thefuck 설치
python 3.12 이상에서는 정상적으로 작동 하지 않기 때문에 pipx 로 설치를 했다
```bash
# 일단 uv로 python 3.11 설치
$ uv python install 3.11

$ pipx install thefuck --python $(uv python find 3.11)

# ~/.zshrc 
# eval $(thefuck --alias)
```

생각보다 오래 걸리는 명령이라 좀 실망 (한 3-5초?)
그리고 오류 메시지가 없었는 경우에는 fuck 도 찾질 못함

---

# 2. Windows Native 환경 세팅
## Scoop 설치
```bash
# PowerShell 실행 정책 설정 (관리자 권한 필요)
$ Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

# Scoop 설치 (사용자 권한)
$ iwr -useb get.scoop.sh | iex

# 설치 확인
$ scoop --version
```

bucket 추가
```bash
$ scoop install git
$ scoop bucket add main
$ scoop bucket add extras
$ scoop bucket add java
```

패키지 추가
```bash
$ scoop install aria2
$ scoop install 7zip
$ scoop install curl
$ scoop install fd
$ scoop install fzf
$ scoop install gcc
$ scoop install lazygit
$ scoop install neovim
$ scoop install ripgrep
$ scoop install unzip
$ scoop install wezterm
```

Scoop 으로 설치한 라이브러리들은 주기적으로 업데이트 필요
```bash
$ scoop update

# 하나만 update 하려면 다음과 같이 가능
$ scoop update pnpm
```

설치한건 이렇게 확인 가능
```bash
$ scoop list
$ scoop bucket list
```

## 개발 언어/도구 설치
### NVM
```bash
$ scoop install nvm
$ nvm install lts
$ nvm use lts
$ nvm list

$ scoop install pnpm
```

### Python
```bash
$ scoop install uv
$ uv --version

$ uv python install 3.10 3.11 3.12
```

windows 사용자 변수 $PATH 에 설정해야 쉘에서 python 사용이 가능하다
uv 로 설치한 파이썬 경로는 아래와 같이 확인 가능하다
```
$ uv python list 
...
cpython-3.13.9-windows-x86_64-none                   C:\Users\toplo\AppData\Roaming\uv\python\cpython-3.13.9-windows-x86_64-none\python.exe

cpython-3.13.9-windows-x86_64-none                   C:\Users\toplo\.local\bin\python3.13.exe

cpython-3.12.12-windows-x86_64-none                  C:\Users\toplo\AppData\Roaming\uv\python\cpython-3.12.12-windows-x86_64-none\python.exe

cpython-3.12.12-windows-x86_64-none                  C:\Users\toplo\.local\bin\python3.12.exe
...
```

위에서 확인한 `C:\Users\toplo\.local\bin` 을 PATH 에 추가해준다
> - uv는 `.local\bin`에 **버전 독립 실행 래퍼**를 만들고, 원본 인터프리터는 Roaming 폴더에 있음
> - PATH에는 **`.local\bin`** 만 추가하면 uv가 관리하는 Python을 바로 사용 가능
> - 원본 인터프리터 경로는 직접 추가할 필요 없음, uv가 내부적으로 연결해 줌

Shell 을 새로 켜서 확인해보면 아래와 같이 확인이 가능하다
```bash
$ python3.13 --version
Python 3.13.9
$ python3.12 --version
Python 3.12.12
$ python3.11 --version
Python 3.11.14
```

즉 `python --version` 은 결과가 나오지 않는다
우리가 특정 버전을 경로로 지정한게 아니기 때문에
대신 버전을 뒤에 붙이면 사용 할 수 있는것이다

프로젝트 환경에서는 다음과 같이 uv 로 가상환경을 생성하여 사용 할 수 있다
```bash
$ cd your-project-dir
$ uv python pin 3.12
$ uv venv .venv
$ .venv\Scripts\activate
$ uv pip install -r requirements.txt
```
[[UV setup]]

### JAVA
https://github.com/ScoopInstaller/Java/tree/master/bucket 
여기서 install  가능한 jdk들 확인 가능
```bash
# 이런 식으로 검색도 가능
# scoop search temurin

$ scoop install termin8-jdk

# 터미널 재시작
$ scoop prefix termin8-jdk
$ echo $JAVA_HOME
$ java -version
```


## Redis 설치
이건 scoop 으로 하지 않고 **공식 installer 를 사용한다**
이유는 **Windows 서비스 등록을 하기 위함** (pc 시작시 server 실행하는 기능)
[tistory: Windows10 환경에 Redis 설치하기](https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-Window10-%ED%99%98%EA%B2%BD%EC%97%90-Redis-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)
이걸 참고했던걸로 기억함
헷갈렸던건 여기서 `redis.windows-service.conf` 파일을 수정하면 됨

Redis 서버 실행 확인
```bash
# 이미 실행 중인지 확인
$ Get-Process redis-server
```

공식 Installer 에서 설치 할 때 PATH 추가 안했으면 환경변수 설정
`C:\Program Files\Redis` 아마 여기 있을것임

Redis 접속 테스트
```bash
$ redis-cli ping
pong
```

## Oh My Posh 설치
[[Windows Scoop Oh-My-Posh]]


## Miniconda 설치
같은 Miniconda 기반인데 어떤 설치파일이느냐에 따라 처음부터 깔리는 패키지가 다르다고 한다
Miniconda Installer, Distribution Installer 중 고민 했는데 
env create 당시에 기본으로 깔리는 패키지의 차이
(Python + conda | Python + conda + 여러 데이터 과학/ML 기본 패키지)
라길래 더 가벼운 Miniconda를 선택


## scoop으로 nvim 설치
Lazyvim을 GitBash 에서 너무 쓰고싶어서 어찌저찌 알아보고 하나 골랐는데 그게 scoop 이다 (퍼플렉시티 만세)

[dev.to: configuring-lazyvim-on-windows](https://dev.to/gitaroktato/configuring-lazyvim-and-python-on-windows-nke)
위 블로그 왈 Lazyvim의 정상적인 사용을 위해 아래 것들은 다 설치가 되어 있어야 한다고 함 (내가 위에서 다 설치 하긴 했음)
```bash
$ scoop install extras/wezterm
$ scoop install main/git main/neovim main/curl main/gcc main/fzf main/fd main/ripgrep main/unzip extras/lazygit
$ scoop install main/nvm
```

nvim 이 잘 실행되나 확인만 하고 닫으면 됨
```bash
$ nvim
```

LazyVim 설정 레포 클론
```bash
# mac의 경우
# $ git clone https://github.com/LazyVim/starter ~/.config/nvim

# windows 에서는 여기
$ git clone https://github.com/LazyVim/starter ~/AppData/Local/nvim
```

다시 nvim 실행하고 다 다운로드 되도록 기다림
```bash
$ nvim
```

## 명령어 설치
```bash
$ scoop install bat

# just for fun
$ scoop install typioca
```

## uv 로 thefuck 설치
```bash
$ uv python install 3.11

$ uv tool install --python 3.11 thefuck

# ~/.bashrc
# eval "$(thefuck --alias)"
```

uv로 install 한 경우 gitbash 시작할때마다 경고문구가 뜰건데 무시해도 됨
경고가 싫으면 pipx로 설치를 해보는거 추천
## scoop으로 firefox 설치
어차피 영상 pip view 재생 말고는 쓸일이 없어서... scoop으로 깔끔하게 설치
