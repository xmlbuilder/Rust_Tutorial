# Generic Trait
Rust에서 제네릭 타입에 특성(트레이트) 조건을 부여하는 방법은 매우 중요한 개념.  
이걸 제대로 이해하면 타입 안정성과 코드 재사용성을 동시에 잡을 수 있음.

## 🧩 도입 배경: 왜 제네릭에 트레이트 조건이 필요한가?
Rust의 제네릭은 다양한 타입을 받아들이는 유연한 코드를 만들 수 있게 해줍니다.  
하지만 모든 타입이 모든 연산을 지원하는 건 아니기 때문에, 제네릭 타입에 제약 조건을 걸어야 할 때가 있습니다.
예를 들어, x + y를 수행하려면 해당 타입이 + 연산을 지원해야 합니다.  
이걸 Rust에서는 **트레이트(Trait)**로 표현합니다.

## 📛 제네릭의 한계와 트레이트 조건의 필요성
####  ❌ 문제 코드
```rust
fn add<T>(x: T, y: T) -> T {
    x + y
}
```

#### ⚠️ 컴파일 오류
error[E0369]: cannot add `T` to `T`
= help: consider restricting type parameter `T`


#### 💡 해결 방법
```rust
use std::ops::Add;

fn add<T: Add<Output = T>>(x: T, y: T) -> T {
    x + y
}
```

→ T 타입이 Add 트레이트를 구현하고, 결과 타입이 T여야 함을 명시

#### 전체 코드
```rust
🧪 문제 코드 (제네릭 타입에 트레이트 조건 없이 사용)
fn add<T>(x: T, y: T) -> T {
    x + y
}

fn main() {
    let x = add(1, 2);
    let y = add(1.0, 2.0);
    let z = add("1", "2");
    println!("x = {:?}, y = {:?}, z = {:?}", x, y, z);
}
```


##### ⚠️ 컴파일 오류 메시지
```
error[E0369]: cannot add `T` to `T`
 --> src/main.rs:2:5
  |
2 |     x + y
  |     ^ no implementation for `T + T`
  |
  = help: consider restricting type parameter `T`
  = note: required by `add`

```


## 🧪 실전 예제: Copy와 Clone 트레이트
```rust
fn copy(item: impl Copy) {
    print!("{} ", item);
}

fn clone(item: impl Clone) {
    print!("{} ", item);
}
```

### ⚠️ 오류 발생
```rust
let string = String::from("hello world");
copy(string); // ❌ 오류 발생
```

#### 🔍 오류 메시지 요약
- impl Copy 타입이 Display를 구현하지 않아서 {} 포맷팅 불가
- 해결 방법: 트레이트 바운드 추가
```rust
fn copy(item: impl Copy + std::fmt::Display) {
    print!("{} ", item);
}
```


## 📌 트레이트 바운드(Trait Bound)란?
트레이트 바운드는 제네릭 타입이 특정 트레이트를 구현하고 있어야 함을 명시하는 방법입니다.
### 기본 문법
```rust
fn print<T: Display>(item: T) {
    println!("{}", item);
}
```

### impl Trait 문법
```rust
fn print(item: impl Display) {
    println!("{}", item);
}
```

### where 절 사용
```rust
fn print<T>(item: T)
where T: Display
{
    println!("{}", item);
}
```


## ⚠️ 제네릭 사용 시 주의사항

| 항목             | 설명                                                                 |
|------------------|----------------------------------------------------------------------|
| 출력 포맷        | `{}`는 `Display` 트레이트 필요, `{:?}`는 `Debug` 트레이트 필요         |
| Copy vs Clone    | `Copy`는 값 복사, `Clone`은 메모리 복제. `String`은 `Clone`만 가능     |
| 트레이트 바운드  | 여러 트레이트는 `+`로 연결하거나 `where` 절로 가독성 있게 표현 가능     |



## ✅ 정리: 제네릭 타입에 트레이트 조건 부여
```rust
use std::fmt::Display;

fn show<T: Display>(item: T) {
    println!("{}", item);
}
```

또는
```rust
fn show(item: impl Display) {
    println!("{}", item);
}
```


→ 타입 안정성과 기능 보장을 동시에 확보!

---
