좋아요 JungHwan님, 주신 코드와 이미지 속 내용, 그리고 에러 메시지까지 모두 종합해서
**Rust 클로저의 환경 캡처(Closure Environment Capture)**에 대한 문서를 정리해 드릴게요.

## 📚 Rust 클로저의 환경 캡처 정리
### 1. 클로저(Closure)란?
- 익명 함수로, 변수에 저장하거나 다른 함수에 인자로 전달 가능
- 환경 캡처 기능을 통해, 정의된 스코프의 변수를 참조·변경·소유 가능
- 함수와 달리 외부 변수 접근이 가능하다는 점이 핵심 차이점


#### 사용법
```rust
fn main() {
    let multiplier = 5;
    let func = |x: i32| -> i32 { x * multiplier };
    for i in 1..=5 {
        println!("{}", func(i));
    }
    println!("{}", multiplier); // 👍

}
```

#### 출력 결과
```
5
10
15
20
25
5
```

### 2. 환경 캡처 방식
| 캡처 방식 | 키워드 | 예시 코드 |
|-----------|--------|-----------|
| `&T` (불변 참조) | 없음   | `let c = || println!("{}", x);` |
| `&mut T` (가변 참조) | 없음   | `let mut c = || x += 1;` |
| `T` (소유권 이동) | `move` | `let c = move || println!("{}", x);` |


### 3. 함수와 클로저의 차이
- 클로저: 외부 변수를 캡처 가능
- 일반 함수: 외부 변수를 직접 캡처 불가
#### ✅ 클로저는 가능
```rust
fn main() {
    let x = 4;
    let equal_to_x = |z| z == x;
    let y = 4;
    assert!(equal_to_x(y));
}
```

#### ❌ 함수는 불가능 (컴파일 에러)
```rust
fn main() {
    let x = 4;
    fn equal_to_x(z: i32) -> bool { z == x } // error[E0434]
    let y = 4;
    assert!(equal_to_x(y));
}
```


### 4. move 키워드와 소유권 이동
- move를 사용하면 변수의 소유권을 클로저로 이동
- 주로 스레드로 데이터 이동 시 사용
- 이동된 변수는 원래 스코프에서 더 이상 사용 불가
```rust
fn main() {
    let x = vec![1, 2, 3];
    let equal_to_x = move |z| z == x;
    // println!("{:?}", x); // ❌ 소유권이 이동되어 사용 불가
    let y = vec![1, 2, 3];
    assert!(equal_to_x(y));
}
```


### 5. 사용 가능 예시 (소유권 이동 없이)
```rust
fn main() {
    let x = vec![1, 2, 3];
    let equal_to_x = |z| z == x; // 참조로 캡처
    println!("{:?}", x); // ✅ 여전히 사용 가능
    let y = vec![1, 2, 3];
    assert!(equal_to_x(y));
}
```


### 6. 클로저 캡처 시 주의할 점
- 메모리 오버헤드: 캡처한 변수를 저장하기 위해 추가 메모리 사용
- 타입 고정: 클로저는 최초 호출 시 타입이 고정됨
- 소유권 규칙: move 사용 시 원래 스코프에서 변수 사용 불가
- 멀티스레드: move와 Send/Sync 트레잇 조합 필요

### 7. 관련 에러 사례
| 에러 코드 | 키워드 | 예시 코드 |
|-----------|--------|-----------|
| E0434     | 없음   | `fn equal_to_x(z: i32) -> bool { z == x }` |
| E0382     | move   | `println!("{:?}", x)` |


### 8. 핵심 요약
- 클로저는 함수보다 강력한 이유: 환경 캡처 가능
- 캡처 방식: &T(불변), &mut T(가변), T(소유권 이동)
- move 키워드: 소유권을 클로저로 이동, 주로 스레드 작업에 활용
- 함수는 환경 캡처 불가 → 오버헤드 없음
- 캡처 방식에 따라 소유권과 빌림 규칙이 달라짐

--- 

## Fn / FnMut / FnOnce

앞서 정리한 환경 캡처 방식을 기반으로, Rust의 Fn / FnMut / FnOnce 트레잇과 매핑한 표.
| 트레잇      | 캡처 방식        | 키워드 | 특징 |
|-------------|-----------------|--------|------|
| `Fn`        | `&T` (불변 참조) | 없음   | 외부 변수를 읽기 전용으로 캡처, 여러 번 호출 가능 |
| `FnMut`     | `&mut T` (가변 참조) | 없음   | 외부 변수를 수정 가능하게 캡처, 호출 시 가변 대여 필요 |
| `FnOnce`    | `T` (소유권 이동) | `move` | 외부 변수의 소유권을 가져와 캡처, 한 번만 호출 가능 (소유권 소비) |



## 📌 이해 포인트
- Fn → 읽기 전용 캡처, 가장 제한이 적고 재호출 가능
- FnMut → 가변 참조 캡처, 호출 시마다 가변 대여 필요
- FnOnce → 소유권 이동 캡처, 한 번만 호출 가능 (값을 소비하기 때문)


fn main() {
    println!("=== Fn (불변 참조 캡처) ===");
    fn_example();

    println!("\n=== FnMut (가변 참조 캡처) ===");
    fnmut_example();

    println!("\n=== FnOnce (소유권 이동 캡처) ===");
    fnonce_example();
}

### Fn: 불변 참조 캡처
```rust
fn fn_example() {
    let multiplier = 5;
    let func = |x: i32| x * multiplier; // 불변 참조로 multiplier 캡처
    println!("func(2) = {}", func(2)); // 10
    println!("func(3) = {}", func(3)); // 15
    println!("multiplier 여전히 사용 가능: {}", multiplier); // 5
}
```

### FnMut: 가변 참조 캡처
```rust
fn fnmut_example() {
    let mut multiplier = 5;
    let mut func = |x: i32| {
        multiplier += 1; // 가변 참조로 multiplier 캡처
        x * multiplier
    };
    println!("func(2) = {}", func(2)); // multiplier = 6 → 12
    println!("func(3) = {}", func(3)); // multiplier = 7 → 21
    println!("multiplier 최종 값: {}", multiplier); // 7
}
```

### FnOnce: 소유권 이동 캡처
```rust
fn fnonce_example() {
    let numbers = vec![1, 2, 3];
    let consume = move |x: i32| {
        println!("Captured vec: {:?}", numbers);
        x * 2
    };
    println!("consume(5) = {}", consume(5)); // 10
    // println!("{:?}", numbers); // ❌ 소유권 이동으로 사용 불가
}
```

---
