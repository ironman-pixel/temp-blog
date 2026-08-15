---
date: 2025-11-14
tags:
  - cli
  - terminal
  - til
  - dev-tools
  - windows
---
# ❓ Information
* Windows 에서 Scoop으로  Oh-My-Posh설치

---

# 🔰 Content ->  

Scoop 으로 oh-my-posh 설치
```bash
$ scoop install oh-my-posh
```

PowerShell 프로필 `$PROFILE` 설정
```bash
$ notepad $PROFILE
# 아래 내용 입력
# oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\powerlevel10k_lean_my.omp.json" | Invoke-Expression
```

만약 `notepad $PROFILE` 경로 없음 오류 발생 시 
```bash
# 폴더가 없으면 먼저 생성
New-Item -Type Directory -Path (Split-Path $PROFILE) -Force

# 빈 파일 생성
New-Item -Type File -Path $PROFILE -Force

# 다시 시도
$ notepad $PROFILE
# 아래 내용 입력
# oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\powerlevel10k_lean_my.omp.json" | Invoke-Expression
```

`POSH_THEMES_PATH` 경로로 이동하여 커스텀 테마 작성
```bash
$ cd $env:POSH_THEMES_PATH
$ cp .\powerlevel10k_lean.omp.json powerlevel10k_lean_my.omp.json
$ notepad .\powerlevel10k_lean_my.omp.json
```

powerlevel10k_lean 을 조금 수정한 커스텀 테마
```json
{
  "$schema": "https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/schema.json",
  "version": 3,
  "blocks": [
    {
      "type": "prompt",
      "alignment": "left",
      "newline": false,
      "segments": [
        {
          "type": "path",
          "style": "plain",
          "foreground": "#77E4F7",
          "properties": {
            "style": "full"
          },
          "template": "{{ .Path }} "
        },
        {
          "type": "python",          
          "style": "plain",
          "foreground": "#00FF88",
          "template": "{{ if .Venv }}({{ .Venv }}) {{ end }}"
        }
      ]
    },
    {
      "type": "prompt",
      "alignment": "right",
      "newline": false,
      "segments": [
        {
          "type": "time",
          "style": "plain",
          "foreground": "#00C5C7",
          "properties": {
            "time_format": "15:04:05"
          },
          "template": " {{ .CurrentDate | date .Format }} "
        },
        {
          "type": "executiontime",
          "style": "plain",
          "foreground": "#FFD700",
          "properties": {
            "threshold": 0
          },
          "template": "{{ .FormattedMs }} "
        }
      ]
    },
    {
      "type": "prompt",
      "alignment": "left",
      "newline": true,
      "segments": [
        {
          "type": "text",
          "style": "plain",
          "foreground": "#43D426",
          "template": "❯ "
        }
      ]
    }
  ]
}
```

---

VSCode 에서 동작하도록 하려면 VSCode 에 font 설정을 해줘야함
`Debug > Console: Font Family` 여기에 `'MesloLGS NF'` 라고 넣어주면 됨

---

GitBash 에서도 세팅 필요함
Scoop 으로 git 을 설치했다면:
```bash
$ scoop prefix git
C:\Users\toplo\scoop\apps\git\current
```
`C:\Users\<사용자>\scoop\apps\git\current\bin\bash.exe` 
이게 GitBash 경로임

아마 `~/.bash_profile` 파일도 없을것임
```bash
# ~/.bash_profile
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```
이렇게 하나 만들어주면
로그인 쉘에서도 `~/.bashrc` 내용이 실행됨

Git Bash는 기본적으로 비로그인 셸로 실행되므로
사실 Git Bash에서 .bashrc만 있어도 대부분 문제 없음
다만, Windows Terminal 또는 다른 터미널에서 로그인 셸로 Git Bash를 열 경우를 대비해 위 `~/.bash_profile`파일 추가 추천

이제 `~/.bashrc` 파일 만들면 됨
```bash
eval "$(oh-my-posh --init --shell bash --config ~/powerlevel10k_lean_my.omp.json)"
```
json 파일이 wsl에 있어서 그런가 잘 안되길래 `/c/Users/<사용자 이름>` 아래에 복붙한다음 사용함... 좋은 방법은 아닌듯
