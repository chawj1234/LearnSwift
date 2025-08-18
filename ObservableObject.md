@State가 단일 뷰의 간단한 상태를 관리하는 데 최적화되어 있다면, 더 복잡하고 공유된 상태는 어떻게 다루어야 할까? 이를 위해 우리는 참조 타입 기반 데이터 모델로 눈을 돌려야 한다.
## ObservableObject 소개
ObservableObject는 [[Combine]]프레임워크에서 제공하는 프로토콜로, **class** 타입만이 채택할 수 있다. **프로토콜을 채택한 class의 인스턴스는 SwiftUI 뷰에 의해(@ObservedObject,@StateObject) 관찰 될 수 있다.** 
이는 **SwiftUI**의 데이터 흐름에 **ViewModel**을 **통합**하는 주된 방법이다. 
ObservableObject는 struct 뷰와 class 데이터 모델(ViewModel) 사이를 연결하는 다리 역할을 한다.

## @Published: 프로퍼티 변경 자동 알림
ObservableObject를 준수하는 class 내부에서만 사용되는 프로퍼티 래퍼이다. @Published가 붙은 프로퍼티의 값이 변경되기 직전 시점을 감지해 objectWillChange 퍼블리셔를 자동으로 트리거 한다.

### objectViewChange 퍼블리셔: 뷰 업데이트 엔진
class가 ObservableObject 프로토콜을 준수하면, 컴파일러는 objectWillChange라는 이름의 퍼블리셔를 자동으로 합성해준다. objectWillChange는 객체의 @Published 프로퍼티(SwiftUI에서 ObservableObject 프로토) 중 하나가 변경되기 직전에 신호를 방출한다.
변경되기 직전에 신호를 방출하는 것은 SwiftUI가 '변경 전' 상태와 캡처할 수 있게 되면서 '변경 후' 상태를 비교해 효율적인 UI 업데이트 계획을 수립할 수 있기 때문이다.

## ViewModel 구축
ObservableObject와 @Published를 함께 사용하여 상태와 상태를 조작하는 로직(비즈니스 로직)을 하나의 객체로 캡슐화하는 ViewModel을 만들 수 있다.

```swift
import Foundation
import Combine // ObservableObject와 @Published를 사용하기 위해 필요

// 1. ObservableObject 프로토콜을 채택하는 클래스(ViewModel)를 정의한다.
class UserProfileViewModel: ObservableObject {
    
    // 2. 뷰에 의해 관찰되어야 하는 프로퍼티에 @Published를 붙인다.
    @Published var username: String = "Anonymous"
    @Published var score: Int = 0
    @Published var isLoggedIn: Bool = false
    
    // 3. 데이터(상태)를 변경하는 비즈니스 로직을 메서드로 캡슐화한다.
    func login(name: String) {
        // 실제 앱에서는 네트워크 요청, 데이터 유효성 검사 등이 포함될 수 있다.
        self.username = name
        self.isLoggedIn = true
    }
    
    func logout() {
        self.username = "Anonymous"
        self.isLoggedIn = false
        self.score = 0
    }
    
    func incrementScore() {
        guard isLoggedIn else { return }
        self.score += 1
    }
}
```

# 소유권, 생명주기, 그리고 @StateObject
`observableObject`를 사용하는 과정에서 개발자들이 가장 흔하게 겪는 혼란과 버그의 원인은 바로 소유권과 생명주기의 문제이다. SwiftUI는 ObservableObject를 관찰하기 위해서 @ObservedObject와 @StateObject 이 두가지 프로프티 래퍼를 사용한다. 이 둘은 비슷해 보이지만 결정적인 차이가 있다.

## @ObservedObject
뷰 외부에서 생성되고 소유된 ObservableObject를 관찰하기 위해 사용한다. 즉, **생명주기를 관리하지 않고** 외부에서 전달받은 객체를 관찰할 뿐이다
이 지점은 심각한 버그의 원인이 될 수 있다. 만약 뷰 내부에서 ObservedObject를 사용하여 객체를 직접 초기화하면, (`@ObservedObject var viewModel = MyViewModel()`) 뷰의 상태 변경등으로 인해 뷰가 다시 그려지면 MyViewModel 인스턴스로 함께 초기화된다. 이로 인해 이전 인스턴스는 더 이상 어떤 변수도 참조하지 않음으로 새로 생성된 MyViewModel 인스턴스는 초기값을 가지게 된다.
```swift
다음 코드는 부모 뷰의 상태 변경이 자식 뷰의 @ObservedObject 상태를 어떻게 파괴하는지 보여준다.
class MyViewModel: ObservableObject {
    @Published var count = 0
    init() { print("MyViewModel Initialized!") }
}

struct CounterView: View {
    // @ObservedObject로 인해 뷰가 다시 그려질 때마다 viewModel이 새로 생성됨
    @ObservedObject var viewModel = MyViewModel()

    var body: some View {
        VStack {
            Text("Count: \(viewModel.count)")
            Button("Increment") { viewModel.count += 1 }
        }
    }
}

struct ParentView: View {
    @State private var text = ""

    var body: some View {
        VStack {
            TextField("Type here to re-render", text: $text)
            Divider()
            CounterView() // text가 변경될 때마다 CounterView도 다시 생성됨
        }
       .padding()
    }
}

이 예제에서 ParentView의 TextField에 글자를 입력하면 text 상태가 변경되어 ParentView의 body가 다시 그려집니다. 이때 CounterView()도 새로 생성되면서 MyViewModel이 다시 초기화되고, 콘솔에는 "MyViewModel Initialized!"가 계속 출력되며 카운터 값은 항상 0으로 리셋된다.

```

## @StateObject
뷰가 ObservableObject의 인스턴스를 직접 생성하고 소유해야 할 때 사용된다. 
@StateObject는 SwiftUI에게 "이 객체는 이 뷰의 생명주기와 함께 가야 하니 단 한 번만 생성하고 뷰가 살아있는 동안 계속 유지해달라"고 지시한다. 이로써 ViewModel은 뷰 내부에서 안전하게 생성하고 상태를 유지할 수 있게 된다.

## 황금률: @StateObject로 생성하고, @ObservedObject로 관찰하라.
- 객체를 생성하는 뷰(객체의 소유자)는 반드시 @StateObject를 사용해야 한다.
- 생성된 객체를 전달받아 사용하는 모든 자식 뷰(객체의 관찰자)는 반드시 @ObservedObject를 사용해야한다.
이러한 관계는 @State와 @Binding의 관계와 개념적으로 동일합니다. @State가 값 타입의 소유권을 선언하고 @Binding이 그에 대한 빌린 접근을 제공하는 것처럼, @StateObject는 참조 타입의 소유권을 선언하고 @ObservedObject는 그에 대한 빌린 관찰을 제공합니다. 이처럼 네 가지 도구를 완전히 별개로 보기보다는, 값 타입과 참조 타입이라는 Swift의 두 타입 카테고리에 맞춰 동일한 소유권 문제를 해결하는 두 쌍의 도구로 이해하면 정신적 모델을 크게 단순화할 수 있습니다.

## 상태 관리 도구 선택을 위한 의사결정 트리
1. 데이터의 종류는 무엇인가?
	- 단순 값 타입인가?
		- 예->2번으로 이동
	- 복잡한 참조 타입인가?
		- 예->3번으로 이동
2. (값 타입의 경우) 데이터의 소유자는 누구인가?
	- 이 뷰가 데이터를 소유하고 직접 관리하는가
		- 예->@State 사용
	- 다른 뷰(주로 부모 뷰)가 데이터를 소유하고 이 뷰는 그 데이터를 읽고 수정할 필요만 있는가?
		- 예->@Binding 사용
3. (참조 타입의 경우) 객체의 소유자는 누구인가?
	- 이 뷰가 객체를 생성하고 그 생명주기를 책임지는가?
		- 예->@StateObject 사용
	- 이 뷰는 단지 전달 받아 관찰하기만 하는가?
		- 예->@ObservedObject 또는 @EnvironmentObject
```
@ObservedObject: "이 뷰가 외부에서 전달받거나 (우연히) 직접 생성한 객체를 관찰할 거야. 이 객체의 생명주기 책임은 내게 없어."
@StateObject: "이 뷰가 이 객체를 처음 생성하고, 이 뷰의 생명주기 동안 이 객체를 안정적으로 소유하며 관리할 거야."
```

## MVVM 아키텍처에서의 상태 관리
### Model
애플리케이션의 핵심 데이터와 비즈니스 로직을 나타낸다. 주로 struct로 정의하여 데이터 불변성, 예측 가능성을 확보하지만, 필요에 따라 class로 정의할 수도 있다. SwiftUI 프레임워크에 대한 의존성이 없어야한다.
### View
UI의 구조와 레이아웃, 외관을 선언하는 SwiftUI View struct이다. View는 가능한 한 가볍고 로직이 없어야 한다. 사용자 상호작용이 발생하면 그 이벤트를 ViewModel에 전달하고, ViewModel의 상태 변화에 따라 UI를 업데이트 한다. View는 @StateObject를 사용하여 ViewModel 인스턴스를 생성하고 소유하며 이를 통해 View의 상태와 로직을 완벽하게 분리한다.
### ViewModel
ObservableObject 프로토콜을 준수하는 class이다. View에 필요한 **상태**를 **@Published 프로퍼티**로 노출하고, View의 **비즈니스 로직**을 **메서드**로 구현한다. 

## 흔히 빠지는 함정
1. @ObservedObject로 객체 초기화
2. 뷰에 복잡한 로직 포함
3. 참조 타입에 @State 사용 (참조 타입의 내부 변경을 관찰하려면 반드시 ObservableObject와 @Published를 사용)
4. @EnvironmentObject의 남용: @EnvironmentObject는 앱 전역적인 데이터를 편리하게 공유할 수 있는 강력한 도구이지만, 과도하게 사용하면 데이터의 흐름을 추적하기 어렵게 만드는 숨겨진 의존성을 생성 따라서 **꼭 필요한 전역 상태(예: 인증 정보, 테마 설정)에만 제한적으로 사용**

# Observation 프레임워크 소개

Ios 17, macOS 14부터 @Observable 매크로라는 새로운 관찰 시스템을 도입했다.
더 이상 ObservableObject 프로토콜을 채택하거나, 변경을 알리고 싶은 프로퍼티에 @Published를 붙일 필요가 없다. class의 모든 저장 프로퍼티는 기본적으로 관찰 가능하게 된다.
```swift
// 이전 방식 (ObservableObject)
import Combine

class OldViewModel: ObservableObject {
    @Published var name = ""
}


// 새로운 방식 (@Observable)
import Observation

@Observable
class NewViewModel {
    var name = ""
}
```

@observable 매크로는 두 가지 장점을 제공한다.
- 간결한 구문
- 세분화된 업데이트: ObservableObject 시스템에서는 객체의 @Published 프로퍼티 중 하나라도 변경되면, 해당 객체를 관찰하는 모든 뷰가 다시 렌더링되었다. 심지어 뷰가 변경된 프로퍼티를 전혀 사용하지 않아도 다시 렌더링 되었다. 이는 성능 저하를 유발한다. 반면, **@Observable을 사용하면 실제로 읽는 프로퍼티가 변경될 때만 다시 렌더링 된다.**
