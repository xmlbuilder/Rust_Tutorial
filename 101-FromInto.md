## From/Into
Rust의 From, Into, TryFrom, TryInto 트레이트는 타입 간 변환을 안전하고 일관되게 처리할 수 있도록 도와주는 핵심 기능.

## 🧠 핵심 개념 요약
| 트레이트       | 반환 또는 변환 구조         |
|----------------|-----------------------------|
| `From<T>`      | `T → Self`                   |
| `Into<T>`      | `Self → T`                   |
| `TryFrom<T>`   | `Result<Self, E>`            |
| `TryInto<T>`   | `Result<T, E>`               |

✅ From을 구현하면 Into는 자동으로 제공됩니다.
✅ TryFrom을 구현하면 TryInto도 자동으로 제공됩니다.

### 📦 예제: From 트레이트
```rust
impl From<Book> for u32 {
    fn from(book: Book) -> u32 {
        book.isbn.parse().unwrap_or(0)
    }
}
```
- Book → u32로 변환
- u32::from(book) 또는 book.into()로 사용 가능

#### 전체 코드
```rust
use std::vec::Vec;
#[derive(Debug)]
struct Book{
    title: String,
    author: String,
    published: u32,
    isbn: String,
}
impl From<Book> for u32 {
    fn from(book: Book) -> u32 {
        book.isbn.parse().unwrap_or(0)
    }
}
fn main(){
    let the_book = Book{
        title: String::from("The Rust Programming Language"),
        author: String::from("Steve Klabnik and Carol Nichols"),
        published: 20230228,
        isbn:   String::from("718503105")
    };
    let isbn2 = u32::from(the_book);
    println!("The book is {isbn2}");
}

```

### 🧪 예제: TryFrom 트레이트
```rust
impl TryFrom<Book> for u32 {
    type Error = u32;
    fn try_from(book: Book) -> Result<u32, Self::Error> {
        book.isbn.parse::<u32>().map_err(|_| 0)
    }
}
```

- 실패 가능성이 있는 변환
- u32::try_from(book) 또는 book.try_into()로 사용 가능

#### 실제 코드
```rust
use std::vec::Vec;
#[derive(Debug)]
struct Book{
    title: String,
    author: String,
    published: u32,
    isbn: String,
}
impl TryFrom<Book> for u32 {
    type Error = u32;
    fn try_from(book: Book) -> Result<u32, Self::Error> {
        match book.isbn.parse::<u32>() {
            Ok(n) => Ok(n),
            Err(_) => Err(0)
        }
    }
}

fn main(){
    let the_book = Book{
        title: String::from("The Rust Programming Language"),
        author: String::from("Steve Klabnik and Carol Nichols"),
        published: 20230228,
        isbn:   String::from("718503105")
    };
    //let isbn = the_book.into();
    let isbn2 = u32::try_from(the_book).expect("Error");
    println!("the book isbn {}", isbn2);
}
```


### 🔍 차이점 요약
| 트레이트     | 실패 가능성 | 반환 타입         | 사용 예시                         |
|--------------|--------------|-------------------|-----------------------------------|
| `From`       | ❌ 없음       | `Self`            | `T::from(x)`, `x.into()`          |
| `TryFrom`    | ✅ 있음       | `Result<Self, E>` | `T::try_from(x)`, `x.try_into()`  |


## 🎯 언제 사용하면 좋을까?
| 상황 또는 목적               | 추천 트레이트         |
|-----------------------------|------------------------|
| 변환이 항상 성공하는 경우     | `From` / `Into`        |
| 변환이 실패할 수 있는 경우     | `TryFrom` / `TryInto`  |
| 명확한 변환 방향 정의         | `From`                 |
| 안전한 파싱 또는 유효성 검사  | `TryFrom`              |

---

# FromStr, AsRef, IntoIterator

FromStr, AsRef, IntoIterator는 Rust에서 데이터 변환과 반복 처리를 유연하게 만들어주는 표준 트레이트들. \
From, Into, TryFrom과 함께 사용하면 모듈 간 데이터 흐름을 깔끔하게 설계.

## 🧠 1. FromStr – 문자열을 타입으로 변환
- str → T 변환을 정의하는 트레이트
- 실패 가능성이 있으므로 Result<T, E> 반환
```rust
use std::str::FromStr;
let num = i32::from_str("42").unwrap(); // 또는 "42".parse::<i32>().unwrap();
```

### ✅ 커스텀 타입에 구현 예시
```rust
use std::str::FromStr;

struct Color(u8, u8, u8);

impl FromStr for Color {
    type Err = String;
    fn from_str(s: &str) -> Result<Self, Self::Err> {
        let parts: Vec<&str> = s.split(',').collect();
        if parts.len() != 3 {
            return Err("Invalid format".into());
        }
        let r = parts[0].parse().map_err(|_| "Invalid red")?;
        let g = parts[1].parse().map_err(|_| "Invalid green")?;
        let b = parts[2].parse().map_err(|_| "Invalid blue")?;
        Ok(Color(r, g, b))
    }
}
```


### 🧠 2. AsRef<T> – 참조로 변환
- &Self → &T 변환을 정의
- 소유권을 이동하지 않고 참조만 제공함
- 다양한 타입을 받아들이는 API에 유용
```rust
fn print_path<P: AsRef<std::path::Path>>(path: P) {
    println!("Path: {:?}", path.as_ref());
}

print_path("my_file.txt"); // &str → &Path
print_path(std::path::PathBuf::from("my_file.txt"));
```


#### ✅ 실전 활용
- String, &str, PathBuf, &Path 등 다양한 타입을 하나의 함수에서 처리 가능
- AsRef<str>을 사용하면 String, &str 모두 받아들일 수 있음

### 🧠 3. IntoIterator – 반복 가능한 타입
- for 루프에서 사용되는 핵심 트레이트
- Vec<T>, 배열, HashMap, Option, Result 등 대부분의 컬렉션이 구현
```rust
let v = vec![1, 2, 3];
for x in v.into_iter() {
    println!("{}", x);
}
```

### ✅ 함수 인자에 활용
```rust
fn print_all<I>(items: I)
where
    I: IntoIterator<Item = String>,
{
    for item in items {
        println!("{}", item);
    }
}

print_all(vec!["a".to_string(), "b".to_string()]);
```

- IntoIterator<Item = impl AsRef<str>>로 더 유연하게 받을 수도 있음

## 📦 트레이트 요약

| 트레이트       | 변환 방향 또는 역할                  | 반환 타입 또는 특징            |
|----------------|--------------------------------------|-------------------------------|
| `FromStr`      | `&str` → `T`                         | `Result<T, E>`                |
| `AsRef<T>`     | `&Self` → `&T`                       | 참조 기반 변환, 소유권 유지   |
| `IntoIterator` | `Self` → `Iterator<Item = T>`        | 반복 가능, `for` 루프에 사용  |

---

