---
date: 2025-06-17
tags:
  - til
  - frontend
  - react
---
# ❓ Information
![[static/Pasted image 20250617093523.png|400]]
* (Abstract Syntax Tree)
* 각 언어로 프로그래밍 된 소스 코드는 추상적으로 바뀌게 되고 코드의 구조는 위와 같이 노드로 바뀌게 된다.
* 이러한 노드들이 모여 만들어진 자료 구조가 AST다.

---
# ❗ Relevant data
## 🎯 What Is The Objective
AST가 무엇이고 어떻게 쓰이는가?
## 📦 Information Resources
> [velog:추상 구문 트리 AST란](https://velog.io/@hbsps/%EC%B6%94%EC%83%81-%EA%B5%AC%EB%AC%B8-%ED%8A%B8%EB%A6%ACAST%EB%9E%80)


# 🔰 Content ->  
## 1️⃣ Intro 
* eslint가 소스 코드에 AST를 활용하는 이유
* babel이 AST를 활용하는 이유
## 2️⃣ Overview 
### eslint 가 소스 코드에서 각 문자열이 어떤 역할을 하는지 확인하는 방법
AST는 각 구문에 대해 해당 구문의 타입, 위치, 식별자 등을 토큰화 해서 트리로 나타낸것이다.
즉, 각 코드가 어떤 역할을 하는지 명확해진다.
AST를 활용하면 소스코드를 토큰으로 나눈 뒤 추상화된 트리를 만들게 된다.

### 바벨이 트랜스파일링 하는 방법
react를 열심히 JSX로 만들어도 브라우저는 이해하지 못한다.
결국 JSX를 트랜스파일링 해야 하는데 정규식 등의 방법을 활용한다면 문제가 발생한다.
이때, **AST를 활용**:
	JSX 코드를 AST로 만들고 각 노드가 어떤 역할을 하는지 파악 가능
이를 **브라우저에서 실행할 수 있는 JS 로 변환**: 
	브라우저에서 화면을 볼 수 있게 됨
위 방식을 응용한다면:
	ES6 문법의 JS를 AST로 변환한 뒤, 각 노드의 타입을 확인하여 적절한 버전의 JS 문법으로 교체한다면 소스 코드의 틀은 유지한 채 문법의 버전만 바꿀 수 있을것

### 결론
AST는 컴파일러에서 사용하는 자료 구조중 하나다.
AST를 응용한다면 소스 코드와 관련된 다양한 것을 할 수 있을것이다.