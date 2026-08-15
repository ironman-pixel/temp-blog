---
date: 2025-06-17
tags:
  - til
  - frontend
---
# ❓ Information
* 나무를 흔들어서 떨어뜨리듯, 코드를 빌드할 때도 실제로 쓰지 않는 코드들을 제외한다는뜻
* import 하여 가져오는 모듈 안에서 사용하는 코드만 빌드하기 위함

---
# ❗ Relevant data
## 📦 Information Resources
> [tistory: tree shaking 설명](https://helloinyong.tistory.com/305)


# 🔰 Content ->  
## 1️⃣ Intro 
* Tree Shaking 적용 방법
## 2️⃣ Overview 
### 1. Babelrc 파일 설정
*Babel은: 
	자바스크립트 문법이 브라우저에서 호환이 가능하도록 ES5 문법으로 변환하는 라이브러리이다. 이 작업을 poliful이라 부르고, 이는 현재 웹 개발에 있어서 필수 요소 중 하나라고 봐도 무방하다*
그러나 이 Babel의 역할은 Tree Shaking 작업을 하는데 있어서 걸림돌이 되는 요소이다.
Babel은 import를 require 구문으로 변환을 시키는데, require는 export 하는 모듈을 가져오게 된다.

이를 방지하기 위해 아래처럼 설정 가능하다.
```
{
	"presets" : [
		[
			"@barbel/preset-env",
			{
				"modules": false
			}
		]
	]
}
```
>`babel-preset-env`에 `modules`를 `false`로 하면, import, export의 구문을 ES5의 문법으로 변환시키지 않는다.

### 2. 모듈 내 Side-Effect 발생 여부 확인
*Side-Effect란: 
	현재 모듈 외에 다른 코드에 영향을 끼치는 요소가 있으면, Side-Effect 가 있다고 할 수 있다.*
```
let animals = ['dog', 'cat']

const addAnimals = (name) => {
	animals.push(name);
}
```
위 코드가 Side-Effect 가 발생하고 있는 예시이다.
실제로 addAnimals() 라는 함수가 쓰이지 않아 다른 코드에 영향을 주지 않는다고 해도, addAnimals() 함수 바깥의 변수를 변경하는 작업으로 인해 Side-Effect를 일으킨다고 판단하여 Shaking을 하지 못하게 된다.
**Side-Effect를 일으키지 않는 모듈**은:
	바깥의 변수의 값을 변경하지 않고, 모듈 내 코드로만 봤을 때 인풋 파라미터 값에 의해 아웃풋 결과값을 예측할 수 있도록 되어있어야 Tree-Shaking하기에 안전한 모듈이다.
	
우선 `package.json`에 아래처럼 설정한다.
```
{
	"name": "webpack-tree-shaking-example",
	"version": "1.0.0",
	"sideEffects": false
}
```

혹은 특정 파일만 Side-Effect를 발생하지 않는 모듈이라고 따로 선언할 수 있다.
```
{
	"name": "webpack-tree-shaking-example",
	"version": "1.0.0",
	"sideEffects": [
		"./src/utils/utils.js"
	]
}
```

### 3. 필요한 모듈만 Import 해서 가져오기
```
import { module1, module2 } from '../utilFile';
```

대부분 이 정도면 최신 webpack 환경에서 Tree Shaking 조건을 갖추었기 때문에 사용하지 않는 불필요한 코드를 빌드하지 않도록 Shaking 할 수 있다.
