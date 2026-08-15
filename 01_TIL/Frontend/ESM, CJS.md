---
date: 2025-06-17
tags:
  - til
  - frontend
  - javascript
---
# ❓ Information
* ESM(ECMAScript Modules), CJS(CommonJS)는 JS **모듈 시스템** 이다.
* *모듈: 
  프로그램을 구성하는 시스템을 기능 단위로 독립적인 부분으로 분리한 것*
* **모듈 시스템**: 
  모듈을 정의하고, 가져오고, 관리하는 규칙과 방법을 제공

---
# ❗ Relevant data
## 📦 Information Resources
> [velog: 디자인 시스템을 NPM 배포하는 방법](https://velog.io/@doeunnkimm_/%EC%9E%91%EA%B3%A0-%EC%86%8C%EC%A4%91%ED%95%9C-%EB%94%94%EC%9E%90%EC%9D%B8-%EC%8B%9C%EC%8A%A4%ED%85%9C%EC%9D%84-NPM-%EB%B0%B0%ED%8F%AC%ED%95%B4%EB%B3%B4%EC%9E%90)


# 🔰 Content ->  
It's like a conversation, you need to explain the information however you must only speak about subjects you understand and like. 
## 1️⃣ Intro 
* EMS, CJS 개요
## 2️⃣ Overview 
### EMS 
- `import`와 `export` 키워드 사용
- 비동기 로딩: 모듈을 비동기적으로 로드 가능
- 정적 분석: 코드가 실행되기 전에 의존성 분석 가능 -> Tree-Shaking 쉽게 가능
- 정적인 구조로 모듈끼리 의존하도록 강제 -> 불러온 값 수정 X
- *Node.js 12부터 추가됨. 이미 많은 프로젝트에서 CJS를 사용하고 있을것임.*

### CJS
- `require()`와 `module.exports` 사용
- 동기 로딩: 모듈을 동기적으로 로드
- require/module.exports를 동적으로 하는것에 제약 X -> 불러온 모듈 수정 가능
	- 런타임에 모듈이 동적으로 로드되고, 수정될 수 있음:
	  코드가 실행되기 전에는 어떤 모듈이 어떤 의존성을 가질지 예측하기 어려움
	- 특정 조건에 따라 모듈을 다르게 로드하거나 수정하는 경우:
	  빌드 도구는 이러한 동적 관계를 분석 X

### package.json에서 type, main, module 필드
- `type`: "module" | "commonjs"
	- 모듈의 유형을 지정. 기본적으로 어떤 모듈 시스템으로 처리 될지를 결정
	- 디폴트는 "commonjs"
- `main`
	- 패키지를 불러올 때 기본적으로 사용할 진입점 파일을 지정
- `module`
	- `main` 필드와 유사항 목적으로 사용되는 필드
	- ESM 환경에서 패키지를 사용할 때 진입되는 경로

### 왜 여태 생각없이 사용했을까?
최신 스캐폴딩 도구를 사용하면 `package.json`에서 `type` 필드를 `module` 로 설정해주기 때문.
`type` 필드 값에 따라 `.js`가 어떻게 처리될 지가 결정됨.
- "module"이면, ESM으로 처리
- "commonjs"이면, CJS로 처리
아니면, 직접적으로 확장자명을 입력하여 처리되도록 하는 방법도 있음.
- `.mjs`: ESM 파일로 인식
- `.cjs`: CJS 파일로 인식
그래서 JS 파일이 CJS인지 ESM인지 확인하려면 파일 확장자, `package.json`의 `type` 필드, 사용된 모듈 구문을 확인하면 됨.
