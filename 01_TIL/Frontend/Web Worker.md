---
date: 2025-07-16
tags:
  - til
  - frontend
  - vite
---
# ❓ Information
* pdfjs-dist 를 코드내에서 동적으로 로드 하기 위한 Vite 설정이 필요했다. 관련 내용 이해를 위해 Web Worker 개념에 대해 알아보았다.

---


# 🔰 Content ->  

## 1️⃣ Worker 스크립트란 무엇인가? (Web Workers)
* JS 는 기본적으로 Single Thread 에서 동작함
	* 한번에 하나의 작업만 처리 가능
* Web Worker 는 이 문제점을 해결하기 위해 등장함
* Web Worker:
	* 역할:
	  현재 실행중인 Main Thread와 별개의 Background Thread를 생성하여 무거운 작업을 그곳에서 처리하도록 위임
	* 동작 방식:
		1. **Main Thread** 가 `new Worker('worker.js')` 와 같은 코드로 worker를 생성하고, `worker.js` 라는 별도의 스크립트 파일을 로드
		2. **Main Thread**는 `postMessage()` 를 통해 worker 에게 작업을 요청하고 데이터를 전달
		3. worker Script는 **Background**에서 전달받은 작업을 수행. 이 시간동안 **Main Thread**는 자유롭게 다른 일을 할 수 있으므로 UI는 멈추지 않음
		4. 작업이 끝나면 worker는 다시 `postMessage()`를 통해 결과를 Main Thread에게 돌려줌
- 결론적으로 Worker는 무거운 작업을 Background로 보내 UI 가 멈추는 현상을 방지하는 기술이다.
## 2️⃣ 왜 pdfjs-dist는 동적으로 로드 되어야 하는가?
* pdf 렌더링은 매우 무거운 작업이다
* pdfjs-dist는 가장 무거운 핵심 작업을 `pdf.worker.js` 라는 자체 worker script에게 맡김
* 동적 로딩의 필요성:
	* 성능 최적화:
		1. 사람들이 PDF를 보려고 할때 pdf.js 라이브러리가 로드됨
		2. pdf.js는 자신의 일을 도와줄 `pdf.worker.js` 파일을 동적으로 로드하여 worker 생성
			- worker script가 처음부터 Main 코드에 포함되어있으면 pdf를 보지 않을때도 무거운 worker code를 다운해야 하므로 초기 로딩 속도가 느려짐
	- Vite와의 관계:
		- Vite는 기본적으로 코드에 명시된 import 문을 따라 파일을 분석
		- pdf.js 라이브러리 내부에서 new Worker('pdf.worker.js') 와 같이 문자열 형태로 worker를 로드하는 코드는 Vite 가 정적으로 분석하기 어려움
- 요약 및 정리:
	- `Worker Script`: 
		- UI 멈춤 현상을 막기 위해 JS 연산을 Background Thread에서 처리 하도록 하는 기술
	- `pdfjs-dist` 동작:
		1. PDF 렌더링이라는 무거운 작업을 자체 Worker Sctipt ('pdf.worker.js')를 사용해 Background에서 처리함
		2. 이 worker script 는 필요할 때 동적으로 로드됨
	- `vite.config.ts` 설정의 이유:
		- Vite가 pdfjs-dist 내부에서 동적으로 로드되는 pdf.worker.js 의 존재를 놓치지 않도록, `optimizeDeps: { include: ["pdfjs-dist"] }` 설정을 통해 pdfjs-dist 와 관련된 파일들을 미리 스캔하고 준비해 둘것을 명시적으로 알리기 위함

