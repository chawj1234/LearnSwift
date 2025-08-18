1. `throws` : 함수의 에러 발생 가능성 알리는 키워드
	1. **직접 처리하기**: 함수 내에서 `do-catch`를 사용해 에러을 안전하게 처리합니다.
	2. **책임 전가하기**: `try`만 사용해서 에러를 자기를 호출한 다른 함수에게 그대로 던져버립니다.
2. `do-catch`
	1. `do`: 에러가 발생할 수 있는 '위험한 코드'를 실행하는 공간
	2. `catch`: `do`에서 날아온 에러를 처리하는 공간
3.  [[try]]
	1. `throws`가 붙은 위험한 함수를 `do`에서 호출할 때 사용하는 실행 버튼

```swift
// 주스 만들기 기계에 문제가 생길 수 있다는 것을 나타내는 에러 종류 정의
enum JuiceMachineError: Error {
    case outOfOranges // 오렌지가 없음
    case machineIsBroken // 기계 고장
}

// 'throws'를 붙여서 이 함수가 에러를 던질 수 있음을 알립니다.
func makeJuice(oranges: Int) throws -> String {
    if oranges < 1 {
        // 에러 상황! "오렌지가 부족하다"는 에러를 던집니다(throw).
        throw JuiceMachineError.outOfOranges
    }
    // 성공하면 주스를 반환합니다.
    return "\(oranges)개의 오렌지로 만든 주스 완성!"
}

// 'do' 라는 안전 구역 안에서
do {
    // 'try' 버튼을 눌러서 'throws' 함수를 실행!
    // 1. 성공 시나리오: 오렌지가 충분할 때
    let myJuice = try makeJuice(oranges: 5)
    print("주문 성공: \(myJuice)") // "주문 성공: 5개의 오렌지로 만든 주스 완성!" 출력
    
    // 2. 실패 시나리오: 오렌지가 없을 때
    let anotherJuice = try makeJuice(oranges: 0) // 여기서 에러가 'throw' 됩니다!
    print("이 메시지는 출력되지 않아요.") // 에러가 발생하면 그 즉시 catch 블록으로 점프합니다.

// 'catch'가 에러를 잡아서 처리합니다.
} catch {
    // 위에서 던져진 JuiceMachineError.outOfOranges 에러가 'error' 상수에 담깁니다.
    print("사장님! 문제 발생: \(error)") // "사장님! 문제 발생: outOfOranges" 출력
}
```

에러 다루는 방법
1. `catch { ... }`: 암시적 `error`상수 사용하기
2. `catch let 이름 { ... }`: 직접 상수 이름 정하기
3. `catch enum.case { ... }`: 특정 에러 케이스 enum으로 골라서 처리하기