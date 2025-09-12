# 🧠 Copy vs Clone 트레잇 요약
| 트레잇   | 목적                          | 동작 방식               | 특징 및 제약사항                          |
|----------|-------------------------------|--------------------------|-------------------------------------------|
| `Copy`   | 값의 **얕은 복사** (shallow copy) | 자동 복사 (`let y = x`) | 오버라이딩 불가, 모든 필드가 `Copy`여야 함 |
| `Clone`  | 값의 **깊은 복사** (deep copy)   | 명시적 호출 (`x.clone()`) | 오버라이딩 가능, 자원 복사에 적합           |



## ✅ Copy 트레잇
- 스택에 저장되는 값에 적합 (예: i32, bool, f64)
- Drop 트레잇이 구현된 타입은 Copy 불가
- #[derive(Copy, Clone)]로 자동 구현 가능
- 모든 필드가 Copy를 구현해야 전체 타입에 Copy 적용 가능
### 예시
```rust
#[derive(Debug, Copy, Clone)]
struct MyStruct {
    x: i32,
    y: i32,
}

fn main() {
    let a = MyStruct { x: 1, y: 2 };
    let b = a; // 자동 복사
    println!("{:?}", a); // 가능
    println!("{:?}", b);
}
```



## ✅ Clone 트레잇
- 힙에 저장된 자원을 복사할 때 사용 (예: String, Vec<T>)
- Copy보다 유연하며, 오버라이딩 가능
- #[derive(Clone)] 또는 impl Clone으로 구현
- 명시적으로 .clone() 호출 필요
### 예시
```rust
#[derive(Debug)]
struct MyStruct {
    x: i32,
    y: i32,
}

impl Clone for MyStruct {
    fn clone(&self) -> Self {
        println!("clone is called!");
        MyStruct { x: self.x, y: self.y }
    }
}

fn main() {
    let a = MyStruct { x: 1, y: 2 };
    let b = a.clone(); // 명시적 복사
    println!("{:?}", a);
    println!("{:?}", b);
}
```


## 🧪 Copy vs Clone: 어떤 걸 써야 할까?
| 상황 또는 목적                         | 추천 트레잇 |
|--------------------------------------|-------------|
| 단순한 스칼라 값만 포함된 타입         | `Copy`      |
| 힙 자원을 포함한 타입 (예: String, Vec) | `Clone`     |
| 복사 동작을 커스터마이징하고 싶을 때    | `Clone`     |
| 자동 복사로 성능 최적화가 필요한 경우   | `Copy`      |



## 📌 참고: Copy 가능한 타입 예시
- i32, u64, bool, f64
- (i32, i32) 같은 Copy 타입으로만 구성된 튜플
- char, usize, &T (immutable reference)
❌ String, Vec<T>, Box<T> 등은 Copy 불가 → Clone 필요


# Copy / Clone 성능 
Rust의 Copy와 Clone 트레잇을 중심으로 성능, 메모리 모델, 라이프타임과의 관계까지 확장한 정리.  
단순한 트레잇 설명을 넘어서, Rust의 안전성과 효율성 철학을 이해하는 데 핵심이 되는 내용.

## ⚡ 1. 성능 비교: Copy vs Clone
| 항목           | Copy                          | Clone                             |
|----------------|-------------------------------|-----------------------------------|
| 복사 방식      | `let b = a`                   | `let b = a.clone()`               |
| 복사 비용      | 매우 저렴 (스택 메모리 복사)   | 상대적으로 비쌈 (힙 메모리 복사 포함 가능) |
| 호출 방식      | 자동                           | 명시적                            |
| 오버라이딩 가능 | ❌ 불가능                       | ✅ 가능                            |
| 사용 대상      | 스칼라 값, Copy 가능한 구조체  | 힙 자원 포함 타입, 커스텀 복사 필요 시 |


## ✅ 예시: 성능 차이 측정
```rust
#[derive(Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}

#[derive(Clone)]
struct BigData {
    values: Vec<i32>,
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = p1; // 빠름

    let b1 = BigData { values: vec![1; 1000000] };
    let b2 = b1.clone(); // 느림 (힙 복사)
}
```

- Point는 스택에 저장되므로 복사 비용이 거의 없음
- BigData는 힙에 저장된 Vec을 복사하므로 성능에 영향

## 🧠 2. 메모리 모델과 Copy/Clone의 관계
Rust의 메모리 모델은 **소유권(ownership)**과 **이동(move)**을 중심으로 설계되어 있습니다.

### ✅ Copy가 가능한 타입
- 스택에 저장되는 프리미티브 타입: i32, bool, char, f64, usize
- Copy 가능한 타입으로만 구성된 튜플: (i32, i32)
- &T (불변 참조)

### ❌ Copy가 불가능한 타입
- 힙에 저장되는 타입: String, Vec<T>, Box<T>
- Drop 트레잇을 구현한 타입
- 복사 시 자원 소유권을 고려해야 하는 타입

### ✅ Clone은 힙 자원을 복사할 수 있음
- Vec, String, HashMap 등은 Clone을 통해 복사 가능
- Clone은 Drop과 함께 사용 가능

## ⏳ 3. 라이프타임과의 관계
Copy와 Clone은 라이프타임 자체를 변경하지는 않지만,
값의 소유권과 참조 방식에 따라 라이프타임 설계에 영향을 줍니다.

### ✅ Copy와 라이프타임
```rust
fn use_copy(x: i32) {
    let y = x; // x는 여전히 유효
}
```

- i32는 Copy → 소유권 이동 없음 → 라이프타임 유지
### ❌ Clone과 라이프타임
```rust
fn use_clone(s: String) {
    let s2 = s.clone(); // s와 s2는 별개의 힙 자원
}
```

- String은 Clone → 힙 자원 복사 → 라이프타임 분리 가능
## ✅ 참조와 라이프타임
```rust
fn use_ref<'a>(s: &'a String) {
    println!("{}", s);
}
```


- 참조를 사용할 경우 라이프타임 'a를 명시해야 안전성 보장

## 🧪 실전 팁 요약
| 상황 또는 목적                         | 추천 방식       | 설명                                      |
|--------------------------------------|----------------|-------------------------------------------|
| 스택 기반 값 복사                    | `Copy`         | 빠르고 자동으로 복사됨                     |
| 힙 자원 포함 타입 복사               | `Clone`        | 명시적으로 `.clone()` 호출 필요             |
| 복사 동작을 커스터마이징하고 싶을 때 | `Clone + impl` | `Clone`을 직접 구현하여 로깅 등 추가 가능     |
| 라이프타임 명시 없이 값 전달         | `Copy`         | 소유권 이동 없이 값 전달 가능               |
| 성능 최적화가 중요한 경우            | `Copy`         | 메모리 복사 비용이 낮아 빠른 처리 가능       |


---

