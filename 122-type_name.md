# type_name
Rust의 std::any::type_name::<T>()는 컴파일 타임에 타입 정보를 문자열로 반환하는 함수로,
디버깅이나 로깅, 제네릭 타입 추적에 매우 유용한 도구입니다.


## 🧠 std::any::type_name::<T>()란?
- 정의: fn type_name<T>() -> &'static str
- 역할: 제네릭 타입 T의 정적 타입 이름을 문자열로 반환
- 사용 목적: 디버깅, 로깅, 타입 추적, 제네릭 함수 내부에서 타입 확인
```rust
use std::any::type_name;

fn print_type_of<T>(_: &T) {
    println!("{}", type_name::<T>());
}
```

### 🔍 `type_name::<T>()` 특징 요약

| 항목                  | 설명 또는 예시 출력값                        |
|-----------------------|----------------------------------------------|
| 반환 타입             | `&'static str`                               |
| 기본 타입             | `"i32"`, `"f64"`, `"bool"` 등                |
| 제네릭 타입           | `"std::vec::Vec<i32>"`                       |
| 함수 이름             | `"print_type_of::<i32>"`                     |
| 클로저 표현           | `"main::{{closure}}"`                        |
| 함수 포인터           | `"playground::main"`                         |
| 문자열 리터럴         | `"&str"`                                     |
| 라이프타임 정보        | `'a`, `'static` 등은 **출력에 포함되지 않음** |

## 🧪 예제 분석
```rust
fn main() {
    let s = "Hello";                  // &str
    let i = 42;                       // i32
    print_type_of(&s);               // prints "&str"
    print_type_of(&i);               // prints "i32"
    print_type_of(&main);            // prints "playground::main"
    print_type_of(&print_type_of::<i32>); // prints "playground::print_type_of<i32>"
    print_type_of(&{ || "Hi!" });    // prints "playground::main::{{closure}}"
    print_type_of(&32.90);           // prints "f64"
    print_type_of(&vec![1, 2, 4]);   // prints "std::vec::Vec<i32>"
}
```

### 🔎 해석
- &s: &str → 문자열 slice
- &i: i32 → 정수 타입
- &main: 함수 포인터 → 네임스페이스 포함
- &print_type_of::<i32>: 제네릭 함수 → 타입 인자까지 명시
- &{ || "Hi!" }: 클로저 → 익명 함수로 표현됨
- &vec![1, 2, 4]: Vec<i32> → 제네릭 컨테이너 타입

### ⚠️ 주의사항
- type_name::<T>()은 디버깅용이지, 타입 비교나 로직 분기에는 사용하지 마세요
- 타입 이름은 컴파일러 버전이나 최적화 수준에 따라 달라질 수 있음
- 라이프타임, 트레잇 바운드, const generics 등은 출력에 포함되지 않음

### 🎯 실전 활용 예시
```rust
fn log_type<T>(value: &T) {
    println!("Type: {}", std::any::type_name::<T>());
}
```

- 제네릭 함수에서 타입 추적
- 디버깅 시 구조체나 enum의 실제 타입 확인
- 매크로 내부에서 타입 기반 로깅

---
