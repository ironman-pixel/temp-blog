---
date: 2026-03-17
tags:
  - cs
  - oop
  - architecture
---
# Java 다형성 (Polymorphism)

> [!abstract] 한 줄 정의
> **하나의 타입(참조)으로 여러 형태의 객체를 다룰 수 있는 능력**

## 핵심 전제 조건

다형성이 성립하려면 반드시 **상속(또는 인터페이스 구현)** 관계가 있어야 한다.

```
부모 타입 변수 = 자식 타입 인스턴스;   // ✅ 업캐스팅
자식 타입 변수 = 부모 타입 변수;       // ❌ 컴파일 에러 (명시적 다운캐스팅 필요)
```

---

## 종류

### 1. 컴파일 타임 다형성 — 오버로딩 (Overloading)

같은 이름의 메서드를 **매개변수 시그니처**만 다르게 정의.

```java
class Calculator {
    int add(int a, int b)          { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c)   { return a + b + c; }
}
```

> [!tip] 반환 타입만 다른 경우는 오버로딩이 **아니다** (컴파일 에러)

---

### 2. 런타임 다형성 — 오버라이딩 (Overriding)

부모 메서드를 자식이 **재정의**하고, 실행 시점에 실제 객체 타입의 메서드가 호출된다.

```java
class Animal {
    void sound() { System.out.println("..."); }
}

class Dog extends Animal {
    @Override
    void sound() { System.out.println("멍멍"); }
}

class Cat extends Animal {
    @Override
    void sound() { System.out.println("야옹"); }
}

// 다형성 활용
Animal a = new Dog();
a.sound(); // "멍멍" — 런타임에 Dog의 메서드 호출
```

> [!info] 동적 바인딩 (Dynamic Binding)
> JVM은 참조 변수의 타입이 아닌 **실제 객체의 타입**을 기준으로 메서드를 결정한다.

---

## 캐스팅 (Casting)

### 업캐스팅 (Upcasting) — 자동

```java
Animal a = new Dog(); // 자동 변환, 명시 불필요
```

- 자식 → 부모 방향
- **안전** (항상 성공)
- 자식 고유 멤버 접근 ❌

### 다운캐스팅 (Downcasting) — 명시적

```java
Animal a = new Dog();
Dog d = (Dog) a;      // 명시적 캐스팅 필요
d.fetch();            // 자식 고유 메서드 접근 가능
```

> [!warning] ClassCastException 주의
> 실제 객체가 `Dog`가 아닌데 `Dog`로 캐스팅하면 런타임 에러 발생

```java
Animal a = new Cat();
Dog d = (Dog) a; // ❌ ClassCastException!
```

### instanceof — 안전한 캐스팅

```java
if (a instanceof Dog d) { // Java 16+ 패턴 매칭
    d.fetch();
}

// Java 16 미만
if (a instanceof Dog) {
    Dog d = (Dog) a;
    d.fetch();
}
```

---

## 인터페이스와 다형성

인터페이스도 부모 타입처럼 활용할 수 있어 다형성의 핵심 도구다.

```java
interface Flyable {
    void fly();
}

class Bird implements Flyable {
    public void fly() { System.out.println("새처럼 날다"); }
}

class Drone implements Flyable {
    public void fly() { System.out.println("드론처럼 날다"); }
}

Flyable f1 = new Bird();
Flyable f2 = new Drone();
f1.fly(); // "새처럼 날다"
f2.fly(); // "드론처럼 날다"
```

---

## 실전 활용 패턴

### 배열/컬렉션으로 묶어서 처리

```java
List<Animal> animals = List.of(new Dog(), new Cat(), new Dog());

for (Animal a : animals) {
    a.sound(); // 각 객체의 실제 타입 메서드가 호출됨
}
```

### 메서드 매개변수 타입에 활용

```java
void makeSound(Animal a) {
    a.sound(); // Dog든 Cat이든 다 받을 수 있음
}
```

---

## 오버로딩 vs 오버라이딩 비교

| 구분          | 오버로딩     | 오버라이딩         |
| ----------- | -------- | ------------- |
| 바인딩         | 컴파일 타임   | 런타임           |
| 관계          | 같은 클래스 내 | 상속 관계         |
| 시그니처        | 달라야 함    | 동일해야 함        |
| 반환 타입       | 달라도 됨    | 동일 (공변 반환 허용) |
| `@Override` | 불필요      | 권장            |

---

## 다형성의 장점

> [!success] 왜 쓰는가?
> 1. **유연성** — 새 타입 추가 시 기존 코드 변경 최소화
> 2. **확장성** — `List<Animal>`에 새 동물 클래스를 추가해도 루프 코드 불변
> 3. **결합도 감소** — 구체 타입이 아닌 추상 타입에 의존 (DIP)
