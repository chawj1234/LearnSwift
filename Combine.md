# UIKit에서 Combine 왜 필요한거야?
시간의 흐름에 따라 발생하는 이벤트를 다뤄야 하는 경우가 많다.
예를 들어
1. 사용자가 버튼 클릭
2. 네트워크에서 데이터 다운로드
3. 검색창에 글자 입력
4. 타이머가 1초마다 신호 보내는 것
위와 같은 이벤트들은 언제, 몇 번이나 발생할지 예측하기 어렵다. 
**Combine**은 위와 같이 **비동기**적으로 발생하는 **데이터의 흐름**을 아주 우아하고 통일된 방식으로 **처리**하기 위해 만든 Framework이다!

#### Publisher: 데이터를 생성하고 내보내는 주체
- `Just`: 단 하나의 값만 내보내고 바로 '완료'되는 가장 간단한 Publisher
- `Future`: 단 한 번의 비동기 작업의 결과를 받아와 성공 또는 실패를 전달하는 Publisher
- `PassthroughSubject`: 외부에서 원하는 타이밍에 값을 주입해서 내보낼 수 있는 Publisher
#### Subscriber: 데이터가 필요해서 구독을 신청한 객체, 데이터를 처리한다.
- `sink`: 값이 들어올 때마다, 그리고 완료/에러 신호가 올 때마다 실행할 클로저를 제공하는 일반적인 Subscriber
- `assign`: 받은 값을 특정 객체의 프로퍼티에 바로 할당해주는 Subscriber
#### Operator: 데이터가 생성되면 가공, 필터링, 결합한다.
- `map`: 들어온 값을 다른 형태로 변환
- `filter`: 특정 조건에 맞는 값만 통과
- `debounce`: 사용자의 타이핑이 끝나면 마지막 값만 내보낸다.(검색 기능)
- `combineLatest`: 여러 Publisher들을 결합해서, 그중 하나라도 새로운 값을 내보내면 모든 Publisher의 최신 값들을 묶어서 전달한다.

핵심은 **구독**이라는 행위이다. Subscriber가 Publisher를 구독해야 비로소 데이터가 흐르기 시작한다.

#### 간단한 코드 예시
```swift
import Combine

// 구독을 관리할 보관함을 미리 만들어 둡니다. (아래에서 자세히 설명!)
var cancellables = Set<AnyCancellable>()

// 1. Publisher: [45, 73, 55, 92, 60] 점수들을 하나씩 순서대로 방출할 게시자입니다.
let scorePublisher = [45, 73, 55, 92, 60].publisher

print("--- 합격자 점수 발표 ---")

// 2. 이 모든 것을 연결하여 구독을 시작합니다.
scorePublisher
    // 3. Operator (filter): 60점 이상인 점수만 아래로 통과시킵니다.
    // 45, 55는 여기서 탈락합니다.
    .filter { score in
        return score >= 60
    }
    // 4. Operator (map): 통과된 숫자 점수를 "숫자 + 점" 형태의 문자열로 변신시킵니다.
    // (예: 73 -> "73점")
    .map { score in
        return "\(score)점"
    }
    // 5. Subscriber (sink): 최종적으로 가공된 값을 받아서 출력합니다.
    .sink { finalScore in
        print(finalScore)
    }
    // 6. 생성된 구독권을 보관함에 저장하여 구독이 취소되지 않도록 합니다.
    .store(in: &cancellables)


--- 합격자 점수 발표 ---
73점
92점
60점


```

#### Cancellable 구독 관리
`sink`를 사용해 구독을 시작하면, 그 결과로 **"구독권"** 이 하나 생깁니다. 이 구독권의 이름이 바로 `AnyCancellable`이다. `AnyCancellable`을 보관하지 않으면 구독이 취소된다.
해결 방법
1. **보관함 만들기**: `AnyCancellable`들을 담아둘 `Set` 타입 변수를 만든다. 보통 `Cancellable`로 명명
2. **보관하기**: 구독 코드 마지막에 `.store(in: &candellables)`를 붙여준다. 

##### 이렇게 서랍장에 구독권을 잘 보관해두면, 우리가 이 구독에 여전히 관심이 있다는 뜻이 되므로 데이터가 계속 잘 흘러들어온다. 그리고 나중에 이 서랍장(객체)이 사라질 때, 서랍장 안에 있던 모든 구독권들도 함께 자동으로 폐기(구독 취소)되어 메모리 누수 없이 깔끔하게 정리된다.

---

## 초보자를 위한 SwiftUI 기반 Combine 학습 로드맵 🗺️

이 로드맵은 Combine의 문법 하나하나를 따로 외우는 방식이 아닙니다. SwiftUI 앱을 만들면서 **'왜 Combine이 필요한지'** 체감하고, **'문제를 해결하는 도구'**로서 Combine을 익히는 데 초점을 맞춥니다.

### 🚩 1단계: SwiftUI의 심장, `ObservableObject`와 `@Published` 이해하기

가장 먼저 Combine 코드를 직접 짜보는 대신, SwiftUI에 내장된 Combine의 '결과물'을 경험하며 시작합니다. 데이터가 바뀌면 UI가 '알아서' 바뀌는 마법을 체험하는 단계입니다.

- **학습 목표**:
    - 데이터를 담는 클래스가 `ObservableObject` 프로토콜을 따르는 이유를 이해합니다.
    - 프로퍼티 앞에 `@Published`를 붙이면 어떤 일이 일어나는지 이해합니다.
    - View에서 `@StateObject` 또는 `@ObservedObject`를 사용해 ViewModel의 데이터를 구독하고 화면에 표시합니다.
        
- **핵심 개념**: `ObservableObject`, `@Published`, `@StateObject`

- **실습 예제**:
    - `CounterViewModel`을 만들고 `@Published var count = 0` 프로퍼티를 선언합니다.
    - SwiftUI View에 `Button`과 `Text`를 만듭니다.
    - 버튼을 누르면 ViewModel의 `count`를 증가시키는 메소드를 호출하고, `Text`에 `count`를 표시합니다.
    - **결과**: `.sink`나 `.store` 없이도, 버튼을 누르면 화면의 숫자가 자동으로 바뀌는 것을 확인하며 SwiftUI의 반응형 원리를 이해합니다.

### 🚩 2단계: 첫 Combine 파이프라인 만들기 (`$`, `map`, `assign`)

이제 SwiftUI가 자동으로 해주던 연결을 우리 손으로 직접 만들어 봅니다. `@Published` 프로퍼티 앞에 `$`를 붙여 Publisher로 변신시키는 것이 첫걸음입니다.

- **학습 목표**:
    - `$`의 의미: `@Published` 프로퍼티를 Publisher로 변환하는 방법을 배웁니다.
    - `map` 연산자: 들어온 데이터를 다른 형태로 가공하는 방법을 익힙니다.
    - `assign(to:)` 구독자: 가공된 최종 데이터를 다른 `@Published` 프로퍼티에 할당하는 방법을 배웁니다.
        
- **핵심 개념**: `$` 접두사, `map`, `assign(to:)`
    
- **실습 예제**:
    - 1단계의 `CounterViewModel`에 `@Published var countMessage = ""` 프로퍼티를 추가합니다.
    - `$count` Publisher에 `map` 연산자를 연결하여, 숫자가 들어올 때마다 `"현재 카운트: \(숫자)"` 형태의 문자열로 변환합니다.
    - 변환된 문자열 스트림을 `assign(to: &$countMessage)`를 이용해 `countMessage` 프로퍼티에 할당합니다.
    - View의 `Text`가 `countMessage`를 표시하도록 변경합니다.
    - **결과**: `count`가 바뀔 때마다 `countMessage`가 자동으로 업데이트되고, 화면도 함께 바뀌는 데이터 흐름을 직접 설계합니다.
        

### 🚩 3단계: 실시간 UI 이벤트 제어하기 (`debounce`, `filter`)

사용자의 연속적인 입력을 효과적으로 처리하는 방법을 배웁니다. Combine의 강력함을 가장 크게 체감할 수 있는 '실시간 검색' 기능을 구현하는 단계입니다.

- **학습 목표**:
    
    - `debounce` 연산자: 사용자의 빠른 입력을 기다렸다가 마지막 값만 처리하는 방법을 배웁니다.
        
    - `filter` 연산자: 불필요한 값(예: 빈 문자열)을 데이터 흐름에서 걸러내는 방법을 익힙니다.
        
    - `removeDuplicates` 연산자: 중복된 값을 제거하여 불필요한 작업을 막는 방법을 배웁니다.
        
- **핵심 개념**: `debounce`, `filter`, `removeDuplicates`
    
- **실습 예제**:
    
    - `SearchViewModel`을 만들고 `@Published var searchText = ""`와 `@Published var searchResults: [String] = []`를 선언합니다.
        
    - View에 `TextField`를 만들고 `searchText`와 바인딩합니다.
        
    - `$searchText` Publisher에 `debounce`, `filter`, `removeDuplicates`를 연결하여 API 요청에 적합한 검색어만 추출합니다.
        
    - (지금은 실제 네트워크 대신) 가공된 검색어로 미리 만들어둔 배열에서 검색 결과를 필터링하여 `searchResults`에 할당합니다.
        
    - View의 `List`는 `searchResults`를 표시합니다.
        
    - **결과**: 타이핑을 멈췄을 때만 검색 결과가 부드럽게 갱신되는, 실제 앱과 같은 사용자 경험을 구현합니다.
        

### 🚩 4단계: 비동기 작업의 꽃, 네트워킹 (`URLSession`, `decode`, `catch`)

드디어 Combine을 사용해 서버 API와 통신합니다. 로딩, 성공, 실패와 같은 다양한 네트워크 상태를 깔끔하게 처리하는 방법을 배우는 단계입니다.

- **학습 목표**:
    - `URLSession.dataTaskPublisher`: 네트워킹을 위한 기본 Publisher 사용법을 익힙니다.
    - `decode` 연산자: 서버에서 받은 JSON 데이터를 Swift 모델 객체로 자동 변환합니다.
    - `catch` 연산자: 네트워크 에러 등 파이프라인에서 발생하는 오류를 잡아서 처리하는 방법을 배웁니다.
    - `receive(on:)`: UI 업데이트를 위해 메인 스레드로 작업을 전환하는 이유와 방법을 이해합니다.
    
- **핵심 개념**: `URLSession.dataTaskPublisher`, `decode`, `catch`, `receive(on:)`
    
- **실습 예제**:
    - `PostViewModel`을 만들고 `@Published var posts: [Post] = []`, `@Published var isLoading = false` 등을 선언합니다. (혹은 상태를 나타내는 enum)
    - `URLSession` Publisher를 사용해 API로부터 게시물 목록(JSON)을 가져옵니다.
    - `decode`로 JSON을 `[Post]` 배열로 변환합니다.
    - `catch`를 이용해 네트워크 오류가 발생했을 때의 처리(예: 에러 메시지 표시)를 구현합니다.
    - `receive(on:)`으로 최종 결과를 메인 스레드에서 받아 `posts` 프로퍼티에 할당합니다.
    - **결과**: 로딩 인디케이터를 보여주고, 통신이 성공하면 목록을, 실패하면 에러 메시지를 보여주는 완전한 비동기 데이터 로딩 흐름을 완성합니다.
        

### 🚩 5단계: 여러 데이터 흐름 묶기 (`CombineLatest`)

여러 개의 데이터 소스를 조합하여 하나의 결과를 만들어내는 고급 기술을 배웁니다. 회원가입 폼의 유효성 검사가 대표적인 예입니다.

- **학습 목표**:
    
    - `CombineLatest` Publisher: 여러 Publisher들의 최신 값을 결합하는 방법을 배웁니다.
        
- **핵심 개념**: `Publishers.CombineLatest`, `Publishers.CombineLatest3` 등
    
- **실습 예제**:
    
    - 회원가입 폼 ViewModel에 `@Published var username`, `@Published var password`, `@Published var passwordConfirm` 프로퍼티를 만듭니다.
        
    - `@Published var isSignUpButtonEnabled = false` 프로퍼티를 추가합니다.
        
    - `CombineLatest3`를 사용해 세 개의 텍스트 필드 Publisher를 묶습니다.
        
    - `map` 연산자 안에서 세 값의 유효성(예: 아이디 길이, 비밀번호 일치 여부)을 검사하여 `Bool` 값을 반환합니다.
        
    - 결과 `Bool` 값을 `isSignUpButtonEnabled`에 할당합니다.
        
    - View의 `Button`은 `isSignUpButtonEnabled` 값에 따라 활성/비활성 상태가 결정됩니다.
        
    - **결과**: 모든 조건이 만족될 때만 회원가입 버튼이 활성화되는, 복잡한 로직을 선언적이고 깔끔한 Combine 코드로 구현합니다.