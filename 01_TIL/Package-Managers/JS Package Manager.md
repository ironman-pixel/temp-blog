---
date: 2025-06-17
tags:
  - til
  - frontend
  - javascript
---
# ❓ Information
* 패키지 매니저 선택에 필요한 정보

---
# ❗ Relevant data
## 📦 Information Resources
> [toss.tech: package-manager](https://toss.tech/article/lightning-talks-package-manager)


# 🔰 Content ->  
## 1️⃣ Intro 
* npm, pnpm, PnP 비교
## 2️⃣ Overview 
### Resolution 단계
1. 라이브러리를 정확한 버전으로 고정
2. 설치한 라이브러리가 사용하는 다른 라이브러리(의존성) 확인
3. 의존성의 버전 고정

### Fetch 단계
 - 결정된 버전의 파일을 다운로드

### Link 단계
- Resolution/Fetch 된 라이브러리를 소스 코드에서 사용할 수 있는 환경을 제공하는 과정

#### npm Linker
`package.json`에서 명시하는 모든 의존성을 `node_modules` 디렉토리 밑에 하나씩 씀

Cons:
- 패키지를 찾기 위해 `node_modules`를 계속 타고 올라가며 파일을 읽어야함.
- 디렉토리 크기가 너무 커짐.
  *그래서 **호이스팅(Hoisting)** 방법을 사용하기도 하는데, 최적화가 완전히 되는것도 아니고 불안정 하기도 해서 좋은 방법은 아님.*

#### pnpm Linker
퍼포먼스가 향상된(performant) npm.

기존 `node_modules`디렉토리를 그대로 사용함.
의존성이 디스크에 하나만 설치 되고, Hard link 방식으로 바로 접근하는 방식.
`node_modules`를 쓸 때도 파일을 하나하나 쓸 필요가 없어지고, 속도도 빠름.

#### PnP Linker
Yarn에서 `node_modules` 없이 의존성을 처리하는 방법

어떤 파일에서 무엇을 import 하는가 라는 관점.
JS 객체로 처리.

`.pnp.cjs` 파일을 이용해 `JavaScriptMap`으로 관리함.
`Yarn`을 실행하는 순간에 일어나는 일:
	`Node.js`프로세스가 `PnP Map`을 메모리에 전부 로드하고
	`import`와 `require`문에서 Map을 참조함.
	Node.js의 `--require` 옵션과 `--loader` 옵션을 사용해서  Map을 로딩시킴.
	*`import`와 `require`의 동작을 바꾸는 Node.js의 API를 사용해서 동작을 바꿔서 참고해 사용하도록 한것.*

Pros.
- `yarn.lock` 기반으로 `.pnp.cjs` 파일만 만들어서 쓰면 끝나기 때문에 설치 속도가 빠름.
- 메모리에 파일이 로드되고 나면, Map 연산만 하기 때문에 `import`나 `require` 하는 속도도 빠름. 

Cons
- Node.js 프로세스가 뜨는 속도가 느림.
- `node_modules` 디렉토리와 호환성이 낮음.
