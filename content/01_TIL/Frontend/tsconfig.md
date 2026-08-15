---
date: 2025-06-28
tags:
  - til
  - frontend
  - vite
  - typescript
---
# tsconfig.json 외에 tsconfig.app.json, tsconfig.node.json 파일이 생성되는 이유

> [tistory: tsconfig, tsconfig.appjson, tsconfig.node.json](https://ramincoding.tistory.com/entry/Vite-tsconfigjson-%EC%99%B8%EC%97%90-tsconfigappjson-tsconfignodejson-%ED%8C%8C%EC%9D%BC%EC%9D%B4-%EC%83%9D%EC%84%B1%EB%90%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)

## 각 파일의 역할
- `tsconfig.json`
	- 공통적인 설정을 정의
	- 빌드설정, 전역옵션, 참조파일 목록 등
- `tsconfig.app.json`
	- 브라우저와 관련된 옵션
	- JSX설정, 라이브러리, DOM 관련 옵션
- `tsconfig.node.json`
	- Node와 관련된 옵션
