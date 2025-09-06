# String/&str
String과 &str에 대한 핵심 개념, 메모리 관리, 메서드 사용, 참조 규칙까지 모두 포함.

## 🧵 1. String vs &str 기본 개념
| 항목               | `String`                                      | `&str` (문자열 슬라이스)                     |
|--------------------|-----------------------------------------------|---------------------------------------------|
| 소유권             | 있음                                           | 없음 (불변 참조자)                           |
| 저장 위치          | 힙 (heap)                                      | 힙 또는 바이너리 내부                        |
| 크기               | 고정 (24 bytes)                                | 가변 (문자열 길이에 따라 달라짐)            |
| 가변성             | `mut`로 수정 가능                              | 불변                                         |
| 생성 방법          | `String::from("text")`, `"text".to_string()`   | `"text"`, `&s[start..end]`                   |
| 주요 용도          | 동적 문자열 조작                               | 읽기 전용 참조                               |



## ⚙️ 2. 메모리 관리와 소유권 이동
- String은 힙에 메모리를 요청하고, 스코프가 끝나면 자동으로 drop() 호출됨
- 소유권이 이동되면 원래 변수는 더 이상 사용할 수 없음
```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1; // s1 → s2로 소유권 이동
    // println!("{}", s1); // ❌ 에러: s1은 더 이상 유효하지 않음
}
```
## 메모리 상태
### s1
![string_s1.png](/image/string_s1.png)
### s2 = s1
![string_s1=s2.png](/image/string_s1=s2.png)
### s1
![string_s1_drop.png](/image/string_s1_drop.png)


## 🛠️ 3. 문자열 생성 및 조작
### 생성
```rust
let s = "text".to_string();
let s = String::from("text");
let s: String = "text".into(); // 타입 명시 필요
```

### 추가
```rust
let mut s = String::from("foo");
s.push_str("bar"); // "foobar"
```

## 연결
```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1은 이동됨
```


## 🚫 4. 인덱싱 불가
- Rust는 UTF-8 기반이므로 문자열 인덱싱은 허용되지 않음
```rust
let s = String::from("hello");
let h = s[0]; // ❌ 컴파일 에러
```


## 📏 5. 크기 확인
```rust
println!("{}", std::mem::size_of::<String>()); // 항상 24 bytes
println!("{}", std::mem::size_of_val("서태지")); // 9 bytes
```

- String: 크기 고정
- &str: 크기 가변, size_of_val()로 확인

### 🧠 6. 슬라이스와 참조 규칙
슬라이스 반환 시 주의
```rust
fn return_str() -> &str {
    let country = String::from("Austria");
    &country // ❌ country는 함수 종료 시 drop됨
}
```

- 함수 내에서 생성된 String의 참조를 반환하면 댕글링 참조자 발생 → 컴파일 에러

## 🔄 7. 참조자와 가변성
가변 참조자 사용
```rust
let mut num = 8;
let num_ref = &mut num;
*num_ref += 10;
```

## 중첩 참조
```rust
let second_number = 800;
let triple_reference = &&&second_number;
println!("{}", second_number == ***triple_reference); // true
```


## 불변 + 가변 참조 충돌
```rust
let mut number = 10;
let number_ref = &number;
let number_change = &mut number; // ❌ 에러: 불변 참조 중에 가변 참조 불가
```

## 순서 바꾸면 OK
```rust
let mut number = 10;
let number_change = &mut number;
*number_change += 10;
let number_ref = &number;
println!("{}", number_ref); // OK
```


✅ 전체 요약 (Markdown 표)
| 분류         | 개념/기능                     | 설명 및 예시                                                  |
|--------------|-------------------------------|---------------------------------------------------------------|
| 문자열 타입   | `String` vs `&str`             | 힙에 저장 vs 슬라이스 참조, 소유권 여부 차이                  |
| 메모리 관리   | drop                           | 스코프 종료 시 자동 메모리 반납                               |
| 생성 방법     | `from`, `to_string`, `into`    | 다양한 방식으로 `String` 생성 가능                            |
| 문자열 조작   | `push_str`, `+`                | 문자열 추가 및 연결                                           |
| 인덱싱        | ❌ 불가                         | UTF-8 기반으로 바이트 단위 인덱싱 위험                        |
| 크기 확인     | `size_of`, `size_of_val`       | `String`: 고정, `&str`: 가변                                  |
| 슬라이스 반환 | 댕글링 참조자 방지             | 함수 내 생성된 `String` 참조 반환 금지                        |
| 참조 규칙     | 불변 vs 가변 참조              | 동시에 혼용 불가, 순서로 해결 가능                            |
| 중첩 참조     | `&&&T`                         | 중첩 참조도 가능, `***r`로 접근                                |


---


