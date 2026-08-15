---
date: 2025-06-21
tags:
  - kanban
  - project
---
# Init

## 실행 코드
```
# 프로젝트 생성 (Vite + React + TypeScript)
npm create vite@latest pojectname --template react-ts
cd pojectname

# 패키지 설치 (핵심 라이브러리)
npm install
npm install react-router-dom zustand axios 

npm install @radix-ui/react-dialog @radix-ui/react-dropdown-menu @radix-ui/react-popover @radix-ui/react-slot @radix-ui/react-tabs

# Tailwind CSS v4 설치 (postcss, autoprefixer 불필요)
npm install -D tailwindcss @tailwindcss/vite

# (선택) Storybook 설치
npx storybook init

# 개발 서버 실행
npm run dev
```

## 아직은 굳이
```
# (선택)
npm install framer-motion

# (선택) keycloak 설치 여기선 필요 없긴 하지
npm install react-keycloak-web 
```

## 직접 npm lib 만드는법
> https://medium.com/@gadallah.hatem/how-to-create-your-own-react-icon-library-from-scratch-817fedb2e1f3

## 추가로 넣은거
```
npm install -D vite-plugin-svgr

npm install -D @types/node
```

## 저장시 정렬되는거 설정 해야겠다.
```
npm install -D prettier prettier-plugin-tailwindcss eslint-plugin-prettier
```

## husky 라는거 확인해볼까.
[velog: husky](https://velog.io/@gyur1kim/2025%EB%85%84-%EA%B8%B0%EC%A4%80-Next.js-15-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%85%8B%ED%8C%85%ED%95%98%EA%B8%B0-prettier-eslint-husky#husky)

## Tailwind css에서 cn 함수 사용을 위한 추가 설치
```
npm install --save clsx
npm install tailwind-merge
```
