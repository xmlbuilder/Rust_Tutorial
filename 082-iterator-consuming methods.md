# iterator-consuming methods
 Rust의 **이터레이터 소비 함수(iterator-consuming methods)**에 대해 깊이 있게 정리. 
 이터레이터가 어떻게 데이터를 처리하고, 어떤 방식으로 결과를 반환하는지 명확하게 설명.

## 🧠 이터레이터 소비 함수란?
이터레이터 소비 함수는 이터레이터를 순회하면서 값을 소비하고, 결과를 반환하는 함수입니다.
즉, 이터레이터를 끝까지 사용하고 나면 더 이상 사용할 수 없게 되는 함수들이에요.

### 🔍 대표적인 Rust 이터레이터 소비 함수들

| 함수 이름     | 설명                                               | 반환 타입                    |
|--------------|----------------------------------------------------|------------------------------|
| `.sum()`     | 모든 요소를 더해서 합계를 반환                     | `i32`, `f64` 등 숫자 타입    |
| `.max()`     | 가장 큰 요소를 반환                                 | `Option<&T>`                |
| `.min()`     | 가장 작은 요소를 반환                               | `Option<&T>`                |
| `.count()`   | 요소의 개수를 반환                                  | `usize`                     |
| `.collect()` | 이터레이터 결과를 컬렉션으로 수집                   | `Vec<T>`, `HashMap<K, V>` 등 |
| `.fold()`    | 초기값부터 누적 계산 수행                           | `T`                         |
| `.find()`    | 조건을 만족하는 첫 번째 요소를 반환                 | `Option<&T>`                |
| `.any()`     | 하나라도 조건을 만족하는 요소가 있는지 확인         | `bool`                      |
| `.all()`     | 모든 요소가 조건을 만족하는지 확인                  | `bool`                      |



## 🧪 예제 1: .sum(), .max(), .min() 사용
```rust
fn main() {
    let num = [1, 2, 3];

    let sum = num.iter().sum::<i32>();
    let max = num.iter().max().unwrap_or(&-1);
    let min = num.iter().min().unwrap_or(&-1);

    println!("sum is {sum}, max: {max}, min: {min}");
}
```

### 🔍 설명
- .iter()는 [i32; 3] 배열을 참조로 반복.
- .sum()은 모든 요소를 더해 i32로 반환.
- .max()와 .min()은 가장 큰/작은 값을 Option<&i32>로 반환.
- unwrap_or(&-1)로 None일 경우 기본값 처리.

## 🧪 예제 2: .collect()로 벡터 만들기
```rust
fn main() {
    let nums = vec![1, 2, 3];
    let maps: Vec<i32> = nums.iter().map(|x| x * 2).collect();
    println!("{:?}", maps); // [2, 4, 6]
}
```

### 🔍 설명
- .map()은 각 요소에 연산 적용 (x * 2)
- .collect()는 결과를 Vec<i32>로 수집

## 🧪 예제 3: .filter()로 조건에 맞는 요소 추출
```rust
fn main() {
    let nums = vec![1, 2, 3];
    let filters: Vec<i32> = nums.into_iter().filter(|x| x % 2 == 0).collect();
    println!("{:?}", filters); // [2]
}
```

### 🔍 설명
- .into_iter()는 소유권을 이동시켜 값 자체를 반복
- .filter()는 짝수만 추출
- .collect()로 벡터로 수집

## 🧪 예제 4: .entry()와 .iter()로 빈도수 계산
```rust
use std::collections::HashMap;
fn main() {
    let fruits = vec!["apple", "banana", "apple", "banana", "orange", "pear", "orange"];
    let mut counts: HashMap<&str, i32> = HashMap::new();

    for fruit in fruits.iter() {
        let count = counts.entry(fruit).or_insert(0);
        *count += 1;
    }

    println!("counts: {:?}", counts);
}
```

### 🔍 설명
- .iter()로 각 과일을 참조
- .entry()로 해시맵에 키 삽입 또는 참조
- .or_insert(0)로 기본값 설정 후 증가

##🧪 예제 5: .map()과 .filter()를 함께 사용
.map()은 참조 기반 반복, .filter()는 소유권 이동이 필요해 .into_iter() 사용
```rust
fn main() {
    let nums: Vec<i32> = vec![1, 2, 3, 4];
    let maps = nums.iter().map(|x| x + 1).collect::<Vec<i32>>();
    println!("{:?}", maps); // [2, 3, 4, 5]

    let filters: Vec<i32> = nums.into_iter().filter(|x| x % 2 == 1).collect();
    println!("{:?}", filters); // [1, 3]
}
```


## 🧪 예제 6: .cloned()으로 참조를 값으로 복사
.cloned()는 참조된 값을 복사해서 소유권을 획득
```rust
fn main() {
    let nums: Vec<i32> = vec![1, 2, 3];

    let maps: Vec<i32> = nums.iter().map(|x| x + 1).collect();
    println!("{:?}", maps); // [2, 3, 4]

    let filters: Vec<i32> = nums.iter().filter(|x| **x % 2 == 1).cloned().collect();
    println!("{:?}", filters); // [1, 3]
}
```


### ✅ 정리: 소비 함수의 특징
- 소비 함수는 이터레이터를 끝까지 사용하고 결과를 반환함
- `.sum()`, `.max()`, `.min()`은 숫자 계산에 유용
- `.collect()`는 벡터, 해시맵 등으로 결과 수집
- `.filter()`는 조건에 맞는 요소만 추출
- `.map()`은 요소 변환에 사용
- `.cloned()`는 참조를 값으로 복사


이터레이터 소비 함수는 Rust의 함수형 프로그래밍 스타일을 구현하는 데 핵심이에요

---
