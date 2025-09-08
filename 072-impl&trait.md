# impl & trait
Rust에서 impl과 trait는 객체 지향적인 기능을 제네릭과 함께 제공하는 핵심 요소입니다.  
많은 사람들이 trait에 집중하면서도 impl의 역할을 간과하는 경우가 많음. 

## 🧱 impl이란?
impl은 구조체나 열거형에 대해 메서드를 구현하는 블록입니다.  
객체 지향 언어에서 클래스의 메서드를 정의하는 것과 유사하며, Rust에서는 impl을 통해 구조체에 기능을 부여합니다.
### ✅ 기본 구조
```rust
struct Calculator {
    value: i32,
}

impl Calculator {
    fn new() -> Calculator {
        Calculator { value: 0 }
    }

    fn add(&mut self, num: i32) {
        self.value += num;
    }

    fn sub(&mut self, num: i32) {
        self.value -= num;
    }
}
```

- Calculator 구조체에 new, add, sub 메서드를 정의
- self는 해당 인스턴스를 가리키며, &mut self는 가변 참조
### 🧩 제네릭 구조체에 대한 impl
```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

- impl<T>는 Point<T>의 타입 파라미터 T를 받아 메서드를 정의
- x()는 x 필드의 참조를 반환

### ⚠️ `impl` 사용 시 주의사항

| 항목             | 설명                                                                 |
|------------------|----------------------------------------------------------------------|
| `x, y, T`            | 제네릭 구조체의 필드 타입이 동일해야 함 (`x`, `y`가 모두 `T`)         |
| `self` 종류      | `self`, `&self`, `&mut self`에 따라 메서드 호출 방식과 가변성 결정     |
| `impl<T>`        | 제네릭 타입에 대해 `impl`을 정의할 때 타입 파라미터 명시 필요         |
| `impl Trait`     | 함수 인자나 반환값에 트레이트 기반 타입을 사용할 때 `impl Trait` 사용 |

## 🧠 trait란?
trait는 공통된 동작을 정의하는 인터페이스입니다.  
여러 타입이 동일한 동작을 구현하도록 강제할 수 있으며, **다형성(polymorphism)** 을 Rust에서 구현하는 핵심 도구입니다.
### ✅ 기본 구조
```rust
trait Greet {
    fn greeting(&self) -> String;
}
```

- Greet는 greeting()이라는 메서드를 요구
- impl Greet for Pet처럼 특정 타입에 대해 구현 가능

### 🧩 다양한 타입에 대한 구현
```rust
enum Pet {
    Dog,
    Cat,
    Tiger,
}

impl Greet for Pet {
    fn greeting(&self) -> String {
        match self {
            Pet::Dog => "Dog".into(),
            Pet::Cat => "Cat".into(),
            Pet::Tiger => "Tiger".into(),
        }
    }
}

struct Person {
    name: String,
    active: bool,
}

impl Greet for Person {
    fn greeting(&self) -> String {
        "Hello".into()
    }
}
```

### ✅ trait 기반 함수 호출
```rust
fn meet(one: &impl Greet, another: &impl Greet) {
    println!("one = {}", one.greeting());
    println!("another = {}", another.greeting());
}
```

→ impl Greet를 통해 타입이 아닌 기능을 기준으로 인자를 받음

### ⚠️ `trait` 사용 시 주의사항

| 항목                  | 설명                                                                 |
|-----------------------|----------------------------------------------------------------------|
| `trait` 정의          | 공통된 동작을 명세하며, 여러 타입이 이를 구현하도록 강제함            |
| `fn meet2<T: Greet>(...)` | 제네릭을 사용해 같은 타입에 대해 `trait`를 구현한 인자만 받도록 제한     |
| `T: Greet + Debug + Display` | 다중 트레이트 바운드를 통해 타입에 복합적인 제약을 걸 수 있음             |
| `where` 절            | 복잡한 바운드를 가독성 좋게 표현하는 방식 (`where T: Trait1 + Trait2`) |
| `trait` 내 디폴트 메서드 | `say_hello()`처럼 기본 구현을 제공하여 코드 재사용성과 일관성 향상         |


### 🧩 디폴트 메서드 예시
```rust
trait Greet {
    fn name(&self) -> &str;
    fn say_hello(&self) {
        println!("Hello {}!", self.name());
    }
}
```

→ name()만 구현하면 say_hello()는 자동으로 동작


---

### 주의 사항 (트레이트 메서드에서 인스턴스 데이터 접근 오류)
```rust
trait Greet {
    fn say_hello() {}
}

impl Greet for Student {}

struct Student {
    name: String,
    age: i32,
    alive: bool,
    major: String,
}
```
- trait Greet는 say_hello()라는 메서드를 정의했지만, 구현 내용은 비어 있음.
- impl Greet for Student를 통해 Student 구조체가 Greet 트레이트를 구현한다고 선언.
- 하지만 say_hello()를 오버라이드하지 않았기 때문에, 기본 구현(비어 있음)이 그대로 사용됨.

### 📌 핵심 요약
- 트레이트 메서드에서 self.name처럼 구조체의 필드에 직접 접근하려 하면 컴파일 오류가 발생합니다.
- 트레이트는 구조체의 내부 필드를 알 수 없기 때문에, 필드 접근은 추상화된 메서드를 통해 간접적으로 해야 합니다.
#### 🧩 잘못된 코드 예시
```rust
trait Greet {
    fn say_hello(&self) {
        println!("Hello, {}", self.name);
    }
}
```

- self.name은 Greet 트레이트에 정의되어 있지 않기 때문에 컴파일 오류 발생.
- 트레이트는 name이라는 필드가 존재한다고 보장할 수 없음.
#### 🧩 올바른 코드 예시
```rust
trait Greet {
    fn say_hello(&self) {
        println!("Hello, Rustacean!");
    }
}
```

- 구조체의 필드에 접근하지 않고, 고정된 문자열을 출력하므로 문제 없음.

#### ✅ 해결 방법
트레이트에서 필드 접근이 필요하다면, 필드를 반환하는 메서드를 정의하고 이를 통해 간접적으로 접근해야 합니다:
```rust
trait Greet {
    fn name(&self) -> &str;
    fn say_hello(&self) {
        println!("Hello, {}!", self.name());
    }
}
```

```rust
// 트레이트 구현
impl Greet for Student {
    fn name(&self) -> &str {
        &self.name
    }
}

// 실행 예제
fn main() {
    let s = Student {
        name: String::from("JungHwan"),
        age: 25,
        alive: true,
        major: String::from("Computer Science"),
    };
    s.say_hello(); // 출력: Hello, JungHwan!
}

```

→ 이렇게 하면 name()을 각 구조체에서 구현하고, say_hello()는 안전하게 필드 값을 사용할 수 있어요.


### 🎯 `impl` vs `trait` 정리

| 항목     | 설명                                                                 |
|----------|----------------------------------------------------------------------|
| `impl`   | 구조체나 열거형에 대해 실제 기능(메서드)을 구현하는 블록               |
|          | 객체에 동작을 부여하며, 생성자나 내부 로직을 정의할 수 있음            |
|          | 제네릭 타입에 대해 `impl<T>`로 구현 가능                              |
| `trait`  | 공통된 동작을 명세하는 인터페이스 역할                                 |
|          | 여러 타입이 동일한 동작을 구현하도록 강제함                            |
|          | 다형성을 구현하며, `impl Trait`, `dyn Trait`, `where` 절 등과 함께 사용 |

---
