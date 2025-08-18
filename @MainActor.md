## 메인 스레드(작업 단위)
사용자 입력 처리, 화면 업데이트 등 **애플리케이션의 사용자 인터페이스와 관련된 모든 작업을 전담**한다.
메인 스레드가 특정 작업을 너무 오랫동안 처리하여 다른 UI 관련 작업을 제때 처리하지 못하게 되는 상황(앱이 멈춘 것처럼 보임, UI 깨짐, 응답성 저하)을 반드시 피해야한다.
이를 피하기 위해서 네트워크 요청 등 시간이 오래 걸리는 작업은 백그라운드 스레드에서 수행하고 UI 업데이트가 필요할 때만 메인 스레드로 돌아와야 한다. (MainActor 도입 이유)

## 과거의 방식: DispatchQueue.main.async
Swift 동시성 모델 도입 전 백그라운드 작업이 끝난 후 메인 스레드로 돌아오기 위해DispatchQueue.main.async를 사용했다.
```swift
func fetchImage(for url: URL, completion: @escaping (Result<UIImage, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, _, error in
        //... 오류 처리...
        guard let data = data, let image = UIImage(data: data) else {
            // 실패 시 메인 스레드에서 완료 클로저 호출
            DispatchQueue.main.async {
                completion(.failure(ImageFetchingError.decodingFailed))
            }
            return
        }
        // 성공 시 메인 스레드에서 완료 클로저 호출
        DispatchQueue.main.async {
            completion(.success(image))
        }
    }.resume()
}
```
#### DispatchQueue.main.async 단점
1. 취약성: 개발자가 수동으로 DispatchQueue.main.async 사용을 기억해야한다.
2. 가독성 저하: DispatchQueue.main.async 호출이 반복되어 지저분해진다.
3. 컴파일러 안정성 부재: 컴파일할때 메인 스레드로 돌아가게 하는 코드가 에러인지(백그라운드에서 UI 조작등) 안전한지 미리 알 수 없다.

 \***컴파일 타임:** 사람이 작성한 코드->컴퓨터가 이해하는 기계어 코드로 번역되는 단계 (컴파일러가 번역가)
	 - 문법 오류 검사
	 - 타입 검사
	 - 최적화: 실행 속도를 높이거나 메모리 사용량 감소를 위해 코드를 미리 개선한다. ?????????
 \***런타임:** 컴파일된 프로그램을 실행하고 사용자와 상호작용하는 모든 시점
	 - 실제 연산 수행
	 - 외부 자원 접근
	 - 사용자 입력 처리


## MainActor: 개념 및 전역 액터 인스턴스
Swift Concurrency 프레임워크가 제공하는, 전역적으로 유일한 **액터**이며 메인 스레드에서만 모든 작업을 수행하도록 강제하여 UI 안전성을 보장한다.

\***액터**: **데이터 경합**을 컴파일 타임에 방지함, 자신의 데이터에 대한 모든 접근(await 사용함)을 직렬화함으로 이루어진다.
**\*데이터 경합**: 하나의 공유된 데이터에 여러 스레드가 접근하여 읽고 쓰려고 할 때 발생한다.

## @MainActor: 속성 및 컴파일러 지시자
Swift 언어에서 제공하는 속성(attribute)이다. 
컴파일러에게 특정 함수, 클로저등이 `MainActor`에 의해 격리되어야 한다고 알려주는 지시자이다.
@MainActor를 사용하면 컴파일러가 해당 코드 블럭이 **메인 스레드로 전환, 실행**되어야 함을 컴파일 타임에 인지하고 강제한다.
#### **주의점**: 
메인 스레드로의 전환 보장은 **비동기 컨텍스트에서 호출될 때만 유효**하다. 따라서 동기 함수에 붙은 @MainActor는 "나는 메인 액터에 있을 것으로 기대한다"는 선언에 가깝다!

### 둘의 관계성
`MainActor`는 개념적 실체이다 실제 실행자이며, 여기가 바로 메인 스레드 공간이야 라고 선언하는 존재
`@MainActor`는 그 개념적 실체를 코드에 적용하는 방법이다. 이 코드는 메인 스레드 공간에서만 실행하라고 지시하는 표식이다.


## @MainActor
#### 전역 격리
@MainActor를 선언함으로 해당 클래스와 구조체의 모든 메서드, 프로퍼티가 **MainActor에 격리**된다. 클래스의 경우 서브클래스 또한 @MainActor를 선언해주어야 한다.

#### 격리 탈출: nonisolated
특정 메서드, 프로퍼티를 액터 격리에서 명시적으로 제외하는 키워드
1. 성능 최적화: 불필요한 MainActor로의 전환 비용 방지
2. 프로토콜 준수: 동기적인 프로퍼티나 메서드를 @MainActor 클래스에 요구할 때 nonisolated 사용
액터 격리에서 제외된 메서드, 프로퍼티에서 격리된 값들에 접근할 수 없다.

#### 명시적 일회성 실행: MainActor.run
일회성 DispatchQueue.main.async 블록을 대체하는 방법

#### @MainActor VS MainActor.run
함수의 전체 논리적 연산이 메인 스레드에 속해야 한다면 **@MainActor**
백그라운드 작업 아주 일부분의 UI를 건드려야 한다면 **MainActor.run**


## @MainActor 인스턴스 생성
클래스가 @MainActor로 지정되면 클래스의 초기화 메서드(init) 또한 메인 액터에 격리된다.
메인 액터에 있지 않은 다른 비격리 클래스의 init(동기)에서 격리된 클래스의 init을 호출하면 오류가 발생한다.
```swift
// ViewModel은 메인 액터에 격리됨
@MainActor
final class MyViewModel {
    init() { /*... */ }
}

// AppCoordinator는 비격리 상태
final class AppCoordinator {
    let viewModel: MyViewModel

    init() {
        // 오류 발생! 동기적인 비격리 컨텍스트에서
        // 메인 액터에 격리된 init을 호출할 수 없음
        self.viewModel = MyViewModel()
    }
}
```
##### **이를 해결하기 위해서 @MainActor 클래스의 init을 명시적으로 nonisolated로 표시한다.**
```swift
@MainActor
final class MyViewModel {
    // nonisolated로 표시하여 격리에서 제외
    nonisolated init() {
        //...
    }
}
```
**이렇게 하면 init 메서드만 메인 액터 격리에서 제외되어, 어떤 컨텍스트에서든 오류 없이 동기적으로 호출할 수 있게 된다.**

### nonisolated init
하나의 오류를 해결했지만 이제 nonisolated init 내부에서 새로운 문제에 직면합니다.
1. nonisolated init 내부에서는 메인 액터에 격리된 프로퍼티를 변경, 접근할 수 없다.
2. nonisolated init으로 데이터를 전달 받아 저장 프로퍼티에 할당할 수는 있지만 이 데이터가 Sendable이어야만 한다, Sendable은 액터 경계를 안전하게 넘나들수 있음을 나타낸다. Sendable이 아니라면 컴파일러는 데이터의 접근을 막습니다.

이러한 문제들의 근본적인 이유는 '흐름 민감 격리(Flow-Sensitive Isolation)'라는 개념에 있다. 컴파일러는 self의 상태를 추적합니다. 액터에 격리된 프로퍼티에 값을 할당하는 순간, 컴파일러는 해당 인스턴스가 이제 그 액터의 '소유'가 되었다고 간주하며, 비격리 컨텍스트에서의 추가적인 접근을 금지한다.

결국 "Catch-22"와 같은 딜레마에 빠집니다. @MainActor 타입을 메인 스레드가 아닌 곳에서 생성하려면 nonisolated init이 필요하지만, nonisolated init 안에서는 해당 타입이 관리해야 할 @MainActor 격리 상태를 설정할 수 없습니다. 이 딜레마 때문에 nonisolated init만으로는 부족하며, 고급 패턴들이 필수적입니다.
1. 지연 초기화 패턴: lazy var로 생성 시점 늦추기
2. 비동기 팩토리 패턴: async/await로 인스턴스 구축하기



### 핵심 원칙 요약

- UI 업데이트는 항상 메인 액터에서 수행해야 한다.
- 수동 디스패칭(DispatchQueue)보다 컴파일러가 강제하는 안전성(@MainActor)을 선호해야 한다.
- 합리적인 최소 단위를 격리해야 합니다. 길고 동기적인 비-UI 작업을 메인 액터에 두지 마라
- init은 특별한 경우임을 이해하고, 컨텍스트에 맞는 올바른 초기화 패턴을 사용해야 한다.
- SwiftUI에서는 @StateObject가 @MainActor ViewModel을 소유하고 생성하는 주요 도구이다.

