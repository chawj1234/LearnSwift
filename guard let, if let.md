옵셔널의 내부 구현
```swift
public enum Optional<Wrapped> {
    case none // 값이 없는 경우 (nil)
    case some(Wrapped) // 값이 있는 경우, Wrapped 타입의 값을 가짐
}
```

옵셔널은 값을 담는 '상자'나 '래퍼(wrapper)'와 같기 때문에, 상자 안에 있는 실제 값을 접근하려면 '언래핑(Unwrapping)'이라는 과정을 거쳐야 한다.
## 언래핑 방법
- **강제 언래핑(Forced Unwrapping, !):** 옵셔널 변수 뒤에 느낌표(!)를 붙여 값을 강제로 추출한다.
- **옵셔널 바인딩(Optional Binding, if let & guard let):** 옵셔널에 값이 있는지 안전하게 확인하고 값이 있다면 임시 상수나 변수에 그 값을 할당하여 사용하는 방식이다.
- **nil 병합 연산자(Nil-coalescing Operator, ??):** 옵셔널이 nil일 경우 사용할 기본값을 지정하는 간결한 방법이다.
- **옵셔널 체이닝 (Optional Chaining, ?.):** 옵셔널 변수의 프로퍼티나 메서드에 안전하게 접근할 때 사용합니다. 체인의 어느 한 부분이라도 nil이면, 전체 표현식은 nil을 반환하고 추가적인 연산을 중단하여 런타임 오류를 방지한다.
이러한 도구들 중에서 if let과 guard let은 코드의 흐름을 제어하며 옵셔널을 다루는 가장 중요한 구조이다.

### 옵셔널 바인딩이란
1. **옵셔널 값의 존재 여부 확인**: 옵셔널 변수에 실제 값이 들어있는지(`.some(Wrapped)`) 아니면 값이 없는지(`.none`, 즉 `nil`)를 확인한다.
2. **값 추출 및 임시 상수/변수 할당**: 만약 옵셔널에 값이 있다면, 그 값을 옵셔널 '상자'에서 꺼내어(언래핑) 새로 정의하는 임시 상수나 변수에 할당한다.
3. **스코프 내에서 안전하게 값 사용**: 이렇게 추출된 값은 코드 블럭 내에서 non-optional 타입으로 안전하게 사용할 수 있습니다.

# if let
if let 구문은 **옵셔널 값**이 **존재할 때** 특정 로직을 수행하고, **존재하지 않을 때** **다른 로직을 수행**하거나 아무것도 하지 않는 **'조건부 분기'** 를 위해 설계되었다.
```swift
var optionalVariable: String? = "wonjun"
if let constantName = optionalVariable {
    // optionalVariable에 값이 있는 경우 실행될 코드
    // constantName은 여기서 언래핑된 값으로 사용 가능
} else {
    // optionalVariable이 nil인 경우 실행될 코드 (else 블록은 선택 사항)
}
```
### 위 코드의 작동 방식
1. optionalVariable에 값이 있는지 확인한다.
2. **값이 있다면**, 그 값을 언래핑하여 새로운 상수 constantName에 할당하고 if 블록 {} 내부의 코드를 실행한다.
3. optionalVariable이 **nil**이면 if 블록은 건너뛰고, else 블록이 있다면 해당 코드를 실행합니다. else 블록은 필수가 아니며, 생략 가능하다. (생략된다면 아무것도 실행되지 않음)

### if let의 제한된 범위

if let을 통해 생성된 상수나 변수는 오직 **해당 If 블록의 중괄호 {} 안에서만 유효**하다. 블록 밖에서는 더 이상 존재하지 않으며 접근할 수 없다.

```swift
var optionalNumber: Int? = 100

if let number = optionalNumber {
    print("The number is \(number)") // 여기서 number는 Int 타입의 100
} else {
    print("Optional has no value")
}

print(number) // 컴파일 오류: Cannot find 'number' in scope
```


### 다중 옵셔널 바인딩
**쉼표(,)** 를 사용하여 하나의 if let 구문에서 여러 개의 옵셔널을 동시에 바인딩할 수 있습니다. 이 경우, 나열된 모든 옵셔널이 값을 가지고 있을 때(**nil이 아닐때**)만 if 블록이 실행된다.

```swift
var optionalName: String? = "wonjun"
var optionalAge: Int? = 24

if let name = optionalName, let age = optionalAge{
	// optionalName과 optionalAge 모두 값이 있을 때만 실행
    print("\(name) is \(age) years old.")
}
```

### 불리언 조건 결합
**쉼표(,)** 뒤에 Boolean 조건을 추가하여 바인딩에 추가적인 제약을 걸 수 있다.
```swift
var optionalAge: Int? = 22

// 쉼표를 사용한 불리언 조건 추가 (현대적인 방식)
if let age = optionalAge, age >= 20 {
    // age 값이 존재하고, 그 값이 20 이상일 때 실행
    print("You are an adult aged \(age).")
}
```

### **단축 구문**
Swift 5.7부터 옵셔널 변수와 바인딩될 상수의 이름이 같을 때 코드를 더 간결하게 작성할 수 있는 단축 구문이 도입 되었다.
```swift
let name: String? = "wonjun"

// Swift 5.7 이전
if let name = name {
    print("Hello, \(name)!")
}

// Swift 5.7 이상
if let name { // 'if let name = name'과 동일
    print("Hello, \(name)!")
}
```

---
# guard let
guard let 구문은 if let과 목적 자체가 다르다. guard let은 조건들이 충족되지 않으면 **else를 통해 현재 코드 블록의 실행을 즉시 중단**시키는, 조기 종료(early exit)를 위한 도구이다.
**else 블록이 필수**이며, 이 else 블록은 반드시 return, break, continue, throw와 같은 제어 전송문을 포함하여 현재 스코프를 벗어나야 합니다. 이는 컴파일러에 의해 강제된다.
\**컴파일러에 의해 강제된다*: 컴파일 단계에서 오류가 발생하여 프로그램이 실행되지 않음

```swift
func process(user: User?) {
    guard let validUser = user else {
        print("Error: User object is nil.")
	    //이 블록은 반드시 현재 스코프를 빠져나가야 함 (return, break, continue, throw 등)
	    return // 여기서는 return 사용
    }
    // 이 지점부터 validUser는 nil이 아님이 보장됨
    //... 유효한 사용자에 대한 로직 계속 진행
}
```
만약 guard의 조건이 true이면(즉, 옵셔널에 값이 있으면), else 블록은 실행되지 않고 프로그램의 흐름은 guard 구문 바로 다음 줄로 계속 이어집니다.

### guard let과 if let의 차이점
둘의 극명한 차이점은 상수, 변수의 스코프이다.
1. guard let을 통해 바인딩 된 상수,변수는 guard let 구문 스코프 바깥 전체에서 계속 사용 가능하다.
2. if let을 통해 바인딩 된 상수,변수는 if let 구문 내에서만 사용 가능하다.

guard의 목적은 전제 조건을 '검증'한 후, 검증한 값을 가지고 '로직을 계속 진행'하는 것이다. 따라서 로직이 값에 접근하기 위해서 guard let에서 검증한 값을 **계속 유지**해한다.


### guard let과 if let 비교 분석

| 기능                | if let                              | guard let                                     |
| ----------------- | ----------------------------------- | --------------------------------------------- |
| 주요 목적             | 조건부 실행, 로직 분기                       | 전제 조건 검증, 요구사항 충족 확인                          |
| 바인딩된 변수의 스코프      | if 블록 내부로 제한됨                       | guard 구문을 포함하는 전체 스코프로 확장됨                    |
| else 절            | 선택 사항                               | 필수 사항                                         |
| 제어 흐름 <br>요구사항    | 없음. 블록 실행 후 흐름이 계속됨                 | else 블록은 반드시 현재 스코프를 종료해야 함 (return, throw 등) |
| 코드 구조에 <br>미치는 영향 | 중첩된 코드("Pyramid of Doom")를 유발할 수 있음 | 실패 사례를 조기에 처리하여 평탄하고 선형적인 코드를 촉진함             |
| 사용 사례             | "이 값이 존재하면, 특별한 작업을 수행하라."          | "이 값은 반드시 존재해야만 나머지 작업이 의미 있다."               |


https://docs.google.com/document/d/1KxmxQTP7JiObJUpgeflryjfW96Tk8gDQOs4KGx8FtGo/edit?tab=t.0




