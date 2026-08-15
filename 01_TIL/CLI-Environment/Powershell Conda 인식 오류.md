---
date: 2025-06-05
tags:
  - cli
  - terminal
  - til
  - dev-tools
  - python
  - conda
---
# ❓ Information
* Powershell Conda 인식 오류

---
# ❗ Relevant data
## 📦 Information Resources
[gpt 대화 내용](https://chatgpt.com/c/683d34dc-51e8-8001-a3af-0e81b1472801)

# 🔰 Content ->  

## 문제 상황
PowerShell에서 conda 명령이 실행되지 않음

## 원인 파악

### conda가 어떤 경로에서 실행되고 있는지 확인함

```
Get-Command conda
```

결과는 

```
CommandType     Name                           Version    Source
-----------     ----                           -------    ------
Alias           conda -> Invoke-Conda          0.0        Conda

```

> `Get-Command conda` 결과를 보면 `conda`가 **Alias**로 `Invoke-Conda`에 연결되어 있다. 
> 즉, `conda` 명령어 자체가 PowerShell 함수 또는 커맨드렛 형태로 재정의되어 있다는 뜻
> 
> alias(`Invoke-Conda`)를 제거하고 기본 `conda` 명령어를 사용하는 게 가능하고, 심지어 권장되는 방법 이라고함? 


## 문제 해결
### conda 에서 `Invoke-Conda` alias를 제거함

```
Remove-Item Alias:conda
Get-Command conda
```

결과는

```
CommandType     Name            Version    Source
-----------     ----            -------    ------
Application     conda.exe       0.0.0.0    C:\ProgramData\Anaconda3\...
```

그리고 conda 명령이 잘 동작 한다.
