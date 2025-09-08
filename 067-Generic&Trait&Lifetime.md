# Generic/Trait/Lifetime
## 🧠 핵심 개념 간단 설명
| 개념       | 핵심 역할                                      |
|------------|------------------------------------------------|
| Generics   | 여러 타입(`T`, `U`)에 대해 공통 코드를 작성할 수 있게 함 |
| Trait      | 다양한 타입 중 특정 기능을 구현한 타입만 사용하도록 제약 |
| Lifetime   | 참조값의 유효 기간을 명시하여 메모리 안전성을 보장     |


## 📚 학습 흐름 요약
### ① Generics
- 함수, 구조체, 열거형에 T, U 등 타입 매개변수 사용
- 제약 없는 일반 타입부터 시작 → Trait bound로 확장
- 예: fn add<T: Add>(a: T, b: T) -> T
### ② Trait
- trait 정의 → impl로 구현
- dyn Trait vs impl Trait 차이
- Trait object, default method, supertrait 등 고급 개념도 있음
- 예: trait Drawable { fn draw(&self); }
### ③ Lifetime
- 'a, 'static 같은 lifetime 명시
- 함수 간 참조 전달 시 충돌 방지
- 구조체에 lifetime 붙이기, 함수 시그니처에 명시하기
- 예: fn longest<'a>(x: &'a str, y: &'a str) -> &'a str

## 🧭 추천 학습 순서
- Generics 기본 문법 → 구조체/함수에 적용
- Trait 정의 및 구현 → Trait bound와 함께 사용
- Trait + Generics 조합 → 실전 함수 설계
- Lifetime 기본 개념 → 함수에서 참조값 다루기
- Lifetime + 구조체 → 복잡한 참조 관계 이해
- 실전 예제: API 설계, Iterator, 클로저 등에서 활용
