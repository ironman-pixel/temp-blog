---
date: 2025-11-04
tags:
  - til
  - frontend
  - react
  - vite
  - vitest
---
# ❓ Information
* Vite 기반의 빠르고 가벼운 유닛테스트 프레임워크
* Jest와 유사한 API를 제공하면서도 Vite의 빠른 HMR을 활용

---
# ❗ Relevant data
## 📦 Information Resources
[heropydev: React 테스트 자동화 w. Vitest & Testing Library & MSW & Cypress & Playwright](https://www.heropy.dev/p/Bgimsk)

# 🔰 Content ->  

**jsdom**: 
- 브라우저 환경을 Node.js에서 시뮬레이션
- 브라우저 없이 DOM API로 테스트
**@testing-library/react**: 
- React 컴포넌트를 테스트하기위한 유틸리티 함수 모음
**@testing-library/dom**: 
- DOM 노드를 테스트하기위한 유틸리티 함수 모음
**@testing-library/user-event**: 
- 사용자 이벤트를 시뮬레이션하는 유틸리티
**@testing-library/jest-dom**:
- Jest의 expect 함수를 확장하여 DOM 관련 Matchers를 제공
- `toBeInTheDocument()`, `toHaveTextContent()` 등의 DOM 특화 검증 가능
**@types/jest**:
- Vitest(Jest)의 전역 함수를 import 없이 사용할 때, 타입 오류를 방지하기 위한 타입 정의
**@vitejs/plugin-react**:
- Next.js 프로젝트에서 Vite 기반의 Vitest에서 JSX(TSX) 문법을 이해하고 React 컴포넌트를 처리할 수 있도록 함
**vite-tsconfig-paths**:
-  Next.js 프로젝트에서 TS의 `tsconfig.json` 구성에 정의된 경로 별칭(Path Aliases)를 인식할 수 있도록 함

