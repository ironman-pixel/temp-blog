---
date: 2025-09-01
tags:
  - til
  - frontend
  - vite
---
# ❓ Information
* 

---
# ❗ Relevant data
## 📦 Information Resources
[tistory: Vite 프로젝트 SWC 도입](https://kangs-develop.tistory.com/25)


# 🔰 Content ->  

## SWC (Speedy Web Compiler) 
- Rust 로 작성된 웹 애플리케이션 컴파일러

### 역할
- 최신 사양의 JS 코드를 브라우저 호환성을 위해 이전 사양의 코드로 트랜스파일
- TS 코드를 JS 코드로 컴파일

### 신뢰도
- Next.js 12 버전부터 공식 템플릿에 도입이 됐을 정도로 신뢰도 높음

### 기대효과
- SWC 공식문서에서 Babel 보다 싱글 스레드인 경우 20배가 더 빠르고, 4코어의 경우 70배가 더 빠르다고 나옴

### 빌드를 수행할 때 tsc(TS 내장 컴파일러) 가 타입 체크를 하는 이유
SWC는 코드를 트랜스파일만 하고 타입체크는 수행하지 않는다고 함.

따라서, 빌드시 타입 체크는 tsc 에게 위임하고,
빌드 수행은 SWC를 활용해 트렌스파일을 수행

