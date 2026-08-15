---
date: 2025-06-17
tags:
  - til
  - frontend
  - vite
  - webpack
---
# ❓ Information
* 

---
# ❗ Relevant data
## 📦 Information Resources
> [vercel-blog: Webpack vs Vite](https://duck-blog.vercel.app/blog/web/why-webpack-is-slow-and-why-new-bundlers-are-fast)


# 🔰 Content ->  
## 1️⃣ Intro 
* Webpack 과 Vite 비교
* A preview of what's to come 
## 2️⃣ Overview 
### Webpack이 느린 이유
#### 복잡한 설정 및 구성
Webpack의 강력한 기능:
	복잡한 설정 파일로 유연성을 제공하지만, 초기 설정 및 유지 보수가 어려움.
	특히 대규모 프로젝트에서는 설정 파일이 매우 복잡해져 빌드 시간이 길어짐.

#### 큰 규모의 프로젝트에서의 한계
- `webpack`: 큰 규모의 프로젝트에서 빌드 속도가 느려질 수 있으며, 개발 환경과 프로덕션 환경 모드에서 전체 번들링을 수행함
- `Vite`: 개발 환경에서는 ES 모듈을 직접 로드하여 빠른 속도를 제공하며, 프로덕션 빌드에서는 `Rollup`을 사용하여 최적화된 번들을 생성함
- `Rollup`: ES 모듈에 최적화된 설계와 효율적인 Tree-Shaking, 간단한 Plugin 시스템을 통해 빠른 빌드 속도를 제공
이러한 차이점들 때문에, 대규모 프로젝트에서는 Vite가 개발 속도와 빌드 최적화 측면에서 더 유리할 수 있다.

#### 파일 크키 및 로딩 시간 문제
**Webpack**
- **전체 번들링**:  
  모듈을 하나의 번들 파일로 묶는 방식을 기본으로 함. 
  이는 로딩 시점에 모든 코드를 한번에 가져와야 하기 때문에, 번들 파일이 커질수록 초기 로딩 시간이 길어짐.
- **Code-Splitting**: 
  번들을 여러개의 청크로 나누어 로딩 시간을 최적화 할 수 있음. 
  그러나 설정이 복잡할 수 있으며, 재대로 구현하지 않으면 성능 최적화 효과가 제한적일 수 있음.
- **Tree-Shaking**: 
  사용되지 않는 코드를 제거. 
  하지만 완벽하지 않을 수 있으며, 의존성 관리가 복잡할 수 있음.

**Vite**
- **ES 모듈 기반 개발**: 
  개발 환경에서 ES 모듈을 직접 로드하여 필요한 모듈만 가져옴.
  초기 로딩 시간 단축.
- **프로적션 빌드**:
  Rollup 을 사용하여 최적화된 번들 생성.
  Rollup은 ES 모듈에 최적화 되어있으며, 효율적인 Tree-Shaking과 Code-Splitting을 통해 파일 크기를 최소화.
- **Lazy-Loading**:
  지연 로딩을 통해 필요한 시점에만 모듈을 로드 할 수 있도록 지원.
  초기 로딩 시간 단축.

#### Plugin 및 Loader 사용의 복잡성
**Webpack**:
- **다양한 Plugin 및 Loader**:
  매우 풍부한 Plugin과 Loader 생태계를 가짐.
  이를 통해 파일 형식을 변환하고, 코드 압축, Tree-Shaking, Code-Splitting, 환경변수 설정 등 다양한 기능을 구현할 수 있음.
- **복잡한 설정**:
  각 Plugin과 Loader는 특정 설정을 요구하며, 여러 플러그인과 로더를 함께 사용할 때 설정 충돌이 발생할 수 있음.
- **의존성 관리**:
  복잡한 프로젝트에서는 Plugin과 Loader간의 의존성을 관리하는것이 어렵고, 설정 파일이 복잡해질 수 있음.

**Vite**:
- **간단하고 직관적인 Plugin 시스템**:
  Rollup 의 Plugin 인터페이스를 기반으로 하며, Rollup Plugin을 그대로 사용할 수 있음.
  Vite 전용 Plugin도 쉽게 작성 가능.
- **빠른 설정**:
  Plugin과 Loader 설정이 간단하고 직관적.
  플러그인 간의 충돌이 적고, 설정 파일이 간결하여 관리하기 쉬움.
- **모듈 방식**:
  기본적으로  ES 모듈을 사용하여 모듈간의 의존성을 효율적으로 관리함.

### Vite가 빠른 이유

#### TL;DR
ESBuild와 Pollup의 장점을 합쳤다.

#### ESBuild
이전까지 모든 번들러는 JS 기반이었다. 하지만 JS는 성능상 한계가 있다.
ESBuild는 Go 로 제작되어 10배 빠르다.

#### Vite 특징
- native ES modules 기반의 강력한 개발서버
- ESBuild로 파일들을 통합하고 Rollup을 통해 번들링:
	- 개발 중에는 **ESBuild로 빠르게 모듈 변환/통합**만 함.
	- 빌드시에는 **Rollup을 사용해 최종 번들링**을 함.
- 모든 CommonJS 및 UMD 파일을 ESM으로 불러올 수 있도록 변환함
- 별도 설정 없이 다양한 리소스 import 가능
- CSS 빌드 최적화(Direct Import 구문에 대해 Preload 하도록 함으로써 네트워크 비용 줄임)

### 주요 차이점
#### 설정 복잡성
- **Webpack**: 
  다양한 기능을 제공하지만, 설정이 복잡하고 시간이 많이 소요될 수 있음.
  플러그인 및 로더 간의 의존성 및 충돌 문제를 해결해야 할 때가 많음.
- **Vite**: 
  설정이 간단하고 직관적. 
  플러그인 사용이 간편하며, Rollup의 플러그인 생태계를 그대로 활용 가능.

#### 생태계
- **Webpack**: 
  풍부한 플러그인 및 로더 생태계를 가지고 있으며, 다양한 기능을 구현 가능
- **Vite**: 
  Rollup 플러그인과 Vite 전용 플러그인을 모두 사용할 수 있어, 플러그인 선택의 폭이 넓음.

#### 사용 편의성
- **Webpack**: 
  설정이 복잡할 수 있지만, 강력한 커스터마이징이 가능.
- **Vite**: 
  설정이 간단하여 빠르게 플러그인과 로더를 추가하고 사용할 수 있음.

### 결론
대규모 프로젝트에서는 Vite가 개발 속도와 빌드 최적화 측면에서 더 유리할 수 있으며, 
Webpack은 다양한 기능과 강력한 커스터마이징이 필요한 경우에 적합