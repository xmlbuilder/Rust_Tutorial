# Reference
 **참조(reference)**와 &, &&, &&& 같은 표현들에 대해 깊이 있게 정리.  
 Rust는 메모리 안전성과 성능을 동시에 잡기 위해 참조 개념을 아주 정교하게 다루고 있어요.
 

## 🧠 Reference란?
Rust에서 &T는 값 T에 대한 불변 참조를 의미합니다.
참조는 실제 값을 복사하지 않고, 그 값이 저장된 메모리 주소를 가리키는 포인터예요.
```rust
let my_number = 9;
let reference = &my_number;
```

- reference는 my_number의 참조
- *reference는 참조를 역참조(dereference) 해서 실제 값을 가져옴

## 🔁 &, &&, &&&의 의미
Rust에서는 &를 여러 번 중첩해서 사용할 수 있어요. 각각은 참조의 참조를 의미합니다.
| 표현   | 의미         | 타입 예시 |
|--------|--------------|-----------|
| `&T`   | 값 `T`에 대한 참조       | `&i32`     |
| `&&T`  | 참조에 대한 참조 (`&(&T)`) | `&&i32`    |
| `&&&T` | 참조의 참조의 참조 (`&(&(&T))`) | `&&&i32`   |

이게 왜 가능하냐면, Rust는 자동 역참조(deref coercion) 덕분에 이런 중첩된 참조도 자연스럽게 처리해줘요.

## ✅ 예제 분석 ①: 기본 참조
```rust
fn main(){
    let my_number = 9;
    let reference = &my_number;

    println!("{}", &my_number == reference); // true
    println!("{}", my_number == *reference); // true
    println!("{}", *reference);              // 9
}
```

- &my_number == reference: 두 참조가 같은 주소를 가리키므로 true
- *reference: 역참조해서 실제 값 9를 가져옴
- my_number == *reference: 값 비교 → true

## ✅ 예제 분석 ②: 구조체와 . 연산자
```rust
struct Item {
    number: u8,
}

impl Item {
    fn compare_number(&self, other_number: u8){
        println!("Are {} and {} equal? {}", self.number, other_number, other_number == self.number);
    }
}


fn main(){
    let item = Item { number: 8 };
    let reference_item = &item;
    let reference_item_two = &&item;

    item.compare_number(8);              // 직접 호출
    reference_item.compare_number(8);    // 참조로 호출
    reference_item_two.compare_number(8);// 참조의 참조로 호출
}
```


## 🔍 핵심 포인트: . 연산자와 자동 역참조
Rust에서는 . 연산자를 사용할 때 자동으로 역참조를 해줍니다.
그래서 &&item.compare_number(8)처럼 참조가 여러 겹이어도 문제 없이 메서드 호출이 가능해요.
즉, reference_item_two.compare_number(8)는 내부적으로 다음과 같이 처리됩니다:
(*(*reference_item_two)).compare_number(8);

Rust가 알아서 *를 적용해주기 때문에 개발자가 직접 *를 쓸 필요가 없어요. 이게 바로 deref coercion!


## 📌 요약: 참조와 자동 역참조
| 표현                | 설명 |
|---------------------|------|
| `&T`                | 값 `T`에 대한 참조 |
| `&&T`               | 참조에 대한 참조 (`&(&T)`) |
| `*reference`        | 참조를 역참조하여 실제 값 가져오기 |
| `reference.method()`| 자동 역참조로 메서드 호출 가능 |
| `.`                 | 자동으로 `*`를 적용해줌 (deref coercion) |


---

Rust에서 deref coercion은 초보자들이 헷갈리기 쉬운 개념

## 🧠 Deref Coercion이란?
Deref Coercion은 Rust가 **자동으로 * 역참조(dereference)**를 해주는 기능이에요.
즉, 어떤 타입이 Deref 트레잇을 구현하고 있다면, Rust는 그 타입을 자동으로 참조 해제해서 우리가 원하는 타입으로 바꿔줘요.
예를 들어:
```rust
fn greet(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let my_string = String::from("JungHwan");
    greet(&my_string); // &String → &str로 자동 변환됨!
}
```


왜 되는 걸까?
- String은 Deref<Target = str>을 구현하고 있어요.
- 그래서 &String은 자동으로 &str로 변환돼요.
- 우리는 *를 직접 쓰지 않아도 되고, Rust가 알아서 해줘요.


## 🔧 어떤 타입들이 Deref Coercion을 지원하나요?

| 원래 타입   | 자동 변환 대상 타입 |
|-------------|---------------------|
| `&String`   | `&str`              |
| `&Vec<T>`   | `&[T]`              |
| `Box<T>`    | `&T`                |
| `Rc<T>`     | `&T`                |


이 덕분에 스마트 포인터나 복잡한 참조 구조도 마치 일반 참조처럼 사용할 수 있어요.

## 📌 핵심 요약
| 개념   | 설명 |
|--------|------|
| `Deref` | `*` 연산자를 통해 참조를 역참조할 수 있게 해주는 트레잇 |
|  Deref Coercion  | `*`는 참조를 해제하여 실제 값을 가져오는 연산자 |
|  장점   | 코드가 간결해지고 참조 타입 간의 호환성이 좋아짐    |

---



