---
date: 2025-09-01
tags:
  - package-manager
  - til
  - dev-tools
---
# ❓ Information
* corepack 와 nvm을 같이 사용하면 골치아파진다

---
# ❗ Relevant data
## 🎯 What Is The Objective
Windows + nvm 환경에서 Corepack 동작


# 🔰 Content ->  
## 현재 상황
1. `npm ls -g` 로 확인해 보니 이미 corepack 가 설치된 상태였다.
2. `nvm list` 로 현재 사용중인 node 버전을 확인한다음
3. `corepack enable` 을 실행하니 
4. `C:\Users\ybshi\AppData\Roaming\nvm\v22.12.0\` 경로에 pnpm과 yarn이 자동으로 설치되었다.

## Corepack (용도: 실행파일 관리)
- Corepack: **PackageManager 실행 파일을 다운로드 및 관리**
- 일반적으로 `AppData\Local\Corepack\store-v1` 에 packageManager를 설치 하지만
**nvm 환경에서는 각 Node 버전 폴더 내에 설치**

## PackageManager (용도: 패키지 설치/캐시 관리)

### pnpm
- store 와 프로젝트 node_modules 를 통해 관리
- store 는 **pnpm 이 공유 캐시용으로 사용하는 전역 저장소**
- 실제 프로젝트에서 `pnpm install` 하면:
	1. 필요한 패키지가 **store 에서 가져와짐**
	2. 프로젝트 **node_modules 에 symlink 생성**

### pnpm store 경로 확인
```bash
pnpm store path
```

