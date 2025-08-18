프로토콜은 "특정 작업이나 기능에 적합한 메서드, 프로퍼티, 그리고 다른 요구사항들의 청사진을 정의"한다.
이는 특정 타입이 반드시 이행해야 할 최소한의 자격 요건을 명시하는 일종의 '계약'으로 비유할 수 있다.. 어떤 타입이 이 계약의 **모든 조항**을 충족시킬 때, 그 타입은 해당 프로토콜을 '준수한다'고 표현한다.
전통적인 객체 지향 프로그래밍이 가진 고질적인 문제들에 대한 해답을 제시하며, 코드 유연성, 재사용성을 극대화하는 새로운 길을 열었습니다.

# 핵심 문법
프로토콜은 protocol 키워드를 사용하여 정의합니다. 특정 타입이 프로토콜을 채택하기 위해서는 타입 이름 위에 ':'을 붙이고 채택할 프로토콜의 이름을 명시합니다. 여러 프로토콜을 동시에 채택할 경우, 쉼표(,)로 구분하여 나열할 수 있습니다.
만일 클래스가 특정 슈퍼클래스를 상속받으면서 프로토콜을 채택한다면, 슈퍼 클래스의 이름을 가장 먼저 명시하고 그 뒤에 프로토콜 목록을 기술해야 합니다.
```swift
// 단일 프로토콜 정의  
protocol SomeProtocol {  
    // 요구사항 정의  
}  
  
// 다중 프로토콜을 채택하는 구조체  
struct SomeStructure: FirstProtocol, AnotherProtocol {  
    // 구조체 구현  
}  
  
// 슈퍼클래스를 상속하고 프로토콜을 채택하는 클래스  
class SomeClass: SuperClass, FirstProtocol, AnotherProtocol {  
    // 클래스 구현  
}
```

# 프로퍼티 요구사항
프로토콜은 자신을 채택한 타입이 특정 이름, 타입을 가진 프로퍼티를 제공하도록 강제할 수 있다. 그러나 프로퍼티가 값을 직접 저장하는 '저장 프로퍼티'인지 아니면 계산해서 값을 제공하는 '계산 프로퍼티'인지를 명시하지 않는다.
즉, 프로퍼티의 이름, 타입, 읽기/쓰기 기능 여부만을 제공하고 어떻게 프로퍼티를 구현할 것인지는 타입에게 맡긴다. 
프로퍼티 요구사항은 항상 var 키워드로 선언되어야 하며, 읽기만 가능한 프로퍼티는 {get}으로, 읽고 쓰기가 모두 가능한 프로퍼티는 {get set}으로 명시한다.
```swift
protocol NameCard {  
    // 읽기 전용 프로퍼티 요구사항  
    var id: String { get }  
     
    // 읽기 및 쓰기 가능 프로퍼티 요구사항  
    var Name: String { get set }  
}
```

# 메서드 요구사항
프로토콜은 특정 **인스턴스 메서드**나 **타입 메서드**의 구현을 요구할 수 있다.. 프로토콜 내에서는 메서드의 이름, 매개변수, 반환 타입만을 정의하며, 실제 구현부인 중괄호`{}`와 코드는 포함하지 않는다..
- **인스턴스 메서드 및 타입 메서드**: 타입 메서드는 `static`키워드를 메서드 선언 앞에 붙인다.
- **Mutating 메서드**: 구조체나 열거형과 같은 값 타입의 인스턴스 메서드가 자신의 인스턴스를 수정해야 할 경우, 프로토콜 정의에서 해당 메서드 앞에 `mutating`키워드를 명시해야 합니다. 이는 해당 메서드가 인스턴스 내부의 상태를 변경할 수 있음을 나타내는 중요한 표식입니다.
```swift
protocol Taggable {
    // 인스턴스 프로퍼티
    var tags: { get }
    
    // 인스턴스의 상태를 변경하는 mutating 메서드
    mutating func addTag(_ tag: String)
    
    // 타입 메서드
    static func maxTags() -> Int
}
```

\***인스턴스 메서드**
**정의**: 타입의 개별 인스턴스에 속하는 메서드이다. 즉, 이 메서드를 사용하려면 먼저 해당 타입의 인스턴스를 만들어야 한다.
**특징**: 
1. **인스턴스에 종속**: 인스턴스의 속성에 접근하거나 그 값을 변경할 수 있다.
2. **호출 방법**: 타입의 인스턴스를 생성한 후, `.`을 사용하여 해당 인스턴스를 호출한다. 
3. **`mutating`**(값 타입 한정 키워드): 구조체나 열거형(값 타입)의 인스턴스 메서드가 자신의 인스턴스 내부에 있는 프로퍼티를 변경하려면, 메서드 선언 앞에 `mutating`키워드를 붙여야 합니다. class(참조 타입)dms **`mutating`** 키워드가 필요 없습니다.
**예시**
```swift
struct Dog {
	var name: String // 인스턴스 프로퍼티
	
	// 인스턴스 메서드: Dog 인스턴스의 이름을 출력
	func bark(){
		print("\(name)가 짖습니다.")
	}
	
	// mutating 인스턴스 메서드: Dog 구조체(값 타입)의 프로퍼티를 변경
	mutating func changeName(to newName: String){
		self.name = newName
		print("\(name)로 이름이 변경되었습니다.")
	}
}

var coco = Dog(name: "코코") // Dog 인스턴스 생성
coco.bark() // 출력: 코코가 짖습니다. (coco 인스턴스의 bark 메서드 호출)
coco.changeName(to: "샤넬") // 출력: 샤넬로 이름이 변경되었습니다. (coco 인스턴스의 changeName 메서드 호출)
```

**\*타입 메서드**
**정의:** 타입의 개별 인스턴스가 아닌, 타입 자체에 속하는 메서드이다. 이 메서드는 인스턴스를 만들지 않고도 호출할 수 있다.
**특징:**
1. **타입에 종속:** 인스턴스 프로퍼티에 직접 접근할 수 없다. 대신 타입 프로퍼티( `static`또는 `class` 프로퍼티)에 접근하거나 변경할 수 있다.
2. **호출 방법**: `.`과 타입의 이름을 통해 호출한다.
3. **`static`또는`class` 키워드**:
	- **`static`**: 구조체, 열거형, 클래스에서 모두 사용 가능하며, 해당 메서드가 타입 자체에 묶여 있음을 나타낸다. 클래스에서 **`static`** 메서드는 서브 클래스에서 재정의(override)할 수 없다..
	- **`class`**: class에서만 사용 가능하며, 이 메서드가 서브 클래스에서 재정의 될 수 있음을 나타낸다. 주로 overriding을 허용해야 할 때 사용한다. (최종적으로 재정의를 막으려면 `final class func`를 사용한다.)
**예시:**
```swift
class MathUtil{
	static let PI: Double = 3.14 // 타입 프로퍼티 (모든 MathUtil 인스턴스가 공유)
	
	// static 타입 메서드: MathUtil 타입에 묶여 있으며 재정의 불가, 인스턴스 없이 호출 가능
	static func calculateCircleArea(radius: Double) -> Double{
		return PI * radius * radius
	}
	
	// class 타입 메서드 (class에서만 사용): 서브 클래스에서 override 가능
	class func describe() {
		print("이것은 수학 관련 유틸리티 클래스입니다.")
	}
}

// MathUtil 인스턴스를 만들 필요 없이 바로 호출
let area = MathUtil.calculateCircleArea(radius: 5.0) // MathUtil 타입의 static 메서드(calculateCircleArea) 호출

print("원의 면적: \(area)") // 출력: 원의 면적: 78.5

MathUtil.describe() // MathUtil 타입의 class 메서드 호출

// 클래스 상속 예시
class AdvancedMathUtil: MathUtil{
    override class func describe(){ // class 메서드는 override 가능
        print("이것은 고급 수학 유틸리티 클래스입니다.")
    }
}

AdvancedMathUtil.describe() // AdvancedMathUtil 타입의 재정의된 class 메서드 호출
```

# 이니셜자이저 요구사항
프로토콜은 특정 매개변수를 갖는 이니셜라이저의 구현을 강제할 수 있다. 
이니셜자이저 요구사항은 일반적인 이니셜자이저 선언과 동일한 방식으로 작성되지만, 구현부는 포함하지 않는다. 또한 실패 가능한 이니셜자이저(init?)도 요구할 수 있다.
```swift
protocol Serializable {
    // 실패 가능한 이니셜라이저 요구사항
    init?(data:)
}
```
class 타입이 프로토콜의 이니셜라이저 요구사항을 구현할 때는 특별한 규칙이 적용된다.
해당 이니셜라이저는 반드시 `required`수정자로 표시해야 한다. 이는 해당 클래스의 모든 서브 클래스 또한 이 이니셜라이저 요구사항을 반드시 구현하도록 강제하여, 프로토콜의 계약이 상속 계층 전체에 걸쳐 유지되도록 보장하는 안전장치이다.
이를 통해 상속 계층의 어떤 지점에서 인스턴스를 생성하더라도 프로토콜의 요구사항이 충족됨을 보증한다.

# struct, class, enum별 프로토콜 채택방법

아래 예제는 각 타입이 자신의 특성(값 타입, 참조 타입, 상속 가능성 등)에 맞춰 프로토콜의 요구사항을 어떻게 충족하는지를 명확히 보여줍니다.
```swift
// 1. 프로토콜 정의
enum LogLevel {
    case info, warning, error
}

protocol Loggable {
    var logID: String { get }
    var logLevel: LogLevel { get set }
    
    mutating func updateLogLevel(to newLevel: LogLevel)
    static var logPrefix: String { get }
    
    init(id: String)
}

// 2. Struct의 프로토콜 준수
struct User: Loggable {
    let logID: String
    var logLevel: LogLevel =.info
    
    // 값 타입이므로 인스턴스 수정을 위해 'mutating' 키워드 사용
    mutating func updateLogLevel(to newLevel: LogLevel) {
        self.logLevel = newLevel
    }
    
    static var logPrefix: String {
        return "[User]"
    }
    
    // 구조체는 멤버와이즈 이니셜라이저를 제공하지만, 프로토콜 요구사항을 충족하기 위해 명시적으로 구현
    init(id: String) {
        self.logID = id
    }
}

// 3. Class의 프로토콜 준수
class ServerConnection: Loggable {
    let logID: String
    var logLevel: LogLevel =.info
    
    // 참조 타입이므로 'mutating' 불필요
    func updateLogLevel(to newLevel: LogLevel) {
        self.logLevel = newLevel
    }
    
    static var logPrefix: String {
        return ""
    }
    
    // 클래스는 상속될 수 있으므로 'required' 키워드 필수
    required init(id: String) {
        self.logID = id
    }
}

// 4. Enum의 프로토콜 준수
enum SystemEvent: Loggable {
    case boot(id: String) // enum의 연관 값
    case shutdown(id: String) // enum의 연관 값
    
    var logID: String {
        switch self {
        case.boot(let id),.shutdown(let id):
            return id
        }
    }
    
    // logLevel은 각 case에 직접 저장할 수 없으므로, 별도의 저장소가 필요하거나 계산 프로퍼티로 처리해야 함
    // 여기서는 단순화를 위해 모든 case가 동일한 logLevel을 공유한다고 가정
    private static var _logLevel: LogLevel =.info
    var logLevel: LogLevel {
        get { return SystemEvent._logLevel }
        set { SystemEvent._logLevel = newValue }
    }
    
    // 열거형은 값 타입이므로 'mutating' 키워드 사용
    mutating func updateLogLevel(to newLevel: LogLevel) {
        self.logLevel = newLevel
    }
    
    static var logPrefix: String {
        return ""
    }
    
    // 이니셜라이저는 특정 case를 생성하도록 구현
    init(id: String) {
        self =.boot(id: id)
    }
}
```

# OOP vs POP
### OOP 접근 방식 (상속)
불필요한 상속을 해결하기 위해서 메서드를 override하여 빈 구현을 넣거나 오류가 발생하는 것은 부자연스러운 설계이다.
```swift
class Bird {
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    func fly() {
        // 모든 새가 날지는 않으므로, 이 구현은 문제가 될 수 있다.
        print("\(name)이(가) 하늘을 납니다.")
    }
    
    func swim() {
        // 모든 새가 수영하지는 않는다.
        print("\(name)이(가) 수영을 합니다.")
    }
}

class Eagle: Bird {
    // fly()는 적합하지만, swim()은 불필요하게 상속받는다.
    override func swim() {
        print("독수리는 수영을 할 수 없습니다.")
    }
}

class Penguin: Bird {
    // swim()은 적합하지만, fly()는 불필요하게 상속받는다.
    override func fly() {
        print("펭귄은 날 수 없습니다.")
    }
}

let eagle = Eagle(name: "독수리")
eagle.fly()   // "독수리이(가) 하늘을 납니다."
eagle.swim()  // "독수리는 수영을 할 수 없습니다."

let penguin = Penguin(name: "펭귄")
penguin.fly()   // "펭귄은 날 수 없습니다."
penguin.swim()  // "펭귄이(가) 수영을 합니다."
```

### POP 접근 방식 (조합)
필요한 protocol을 채택하므로 불필요한 기능을 상속받는 문제가 없고 여러 프로토콜을 조합하는 것도 매우 자연스럽다. 이는 유연하고 모듈화된 설계를 보여준다.
```swift
protocol Flyable {
    func fly()
}

protocol Swimmable {
    func swim()
}

// 각 타입은 필요한 능력(프로토콜)만 '조합'하여 채택한다.
struct Eagle: Flyable {
    var name: String
    
    func fly() {
        print("\(name)이(가) 하늘을 힘차게 납니다.")
    }
}

struct Penguin: Swimmable {
    var name: String
    
    func swim() {
        print("\(name)이(가) 물 속을 빠르게 헤엄칩니다.")
    }
}

// 두 가지 능력을 모두 가진 타입도 쉽게 만들 수 있다.
struct Duck: Flyable, Swimmable {
    var name: String
    
    func fly() {
        print("\(name)이(가) 하늘을 납니다.")
    }
    
    func swim() {
        print("\(name)이(가) 물 위를 떠다닙니다.")
    }
}

let eaglePOP = Eagle(name: "독수리")
eaglePOP.fly() // "독수리이(가) 하늘을 힘차게 납니다."

let penguinPOP = Penguin(name: "펭귄")
penguinPOP.swim() // "펭귄이(가) 물 속을 빠르게 헤엄칩니다."
```

# 고급 프로토콜 기법
Swift 프로토콜은 기본적인 요구사항 정의를 넘어, 프로토콜 간의 관계를 설정하고 코드 재사용성을 극대화하는 강력한 고급 기능(**프로토콜 상속, 조합, 확장**)들을 제공한다.

## 프로토콜 상속
프로토콜도 다른 프로토콜로부터 요구사항을 물려받을 수 있다. 이를 통해 요구사항의 계층 구조를 만들 수 있다. (부모-자식 개념) 자식 프로토콜을 준수하는 타입은 부모와 자식 프로토콜의 모든 요구사항을 구현해야 한다.
```swift
// 기본 미디어 프로토콜
protocol Media {
    var title: String { get }
}

// Media를 상속받는 재생 가능 프로토콜
protocol Playable: Media {
    func play()
}

// Playable을 준수하는 Song 구조체
struct Song: Playable {
    var title: String
    
    func play() {
        print("\(title)을(를) 재생합니다.")
    }
}

let mySong = Song(title: "Hype Boy")
print(mySong.title) // "Hype Boy"
mySong.play()       // "Hype Boy을(를) 재생합니다."
```

## 프로토콜 조합
때로는 새로운 프로토콜을 상속 관계로 정의하지 않고, 여러 프로토콜의 요구사항을 임시로 결합하여 사용하고 싶을 때가 있다. Swift는 `&`연산지를 사용하여 여러 프로토콜을 하나의 임시 타입으로 조합하는 기능을 제공한다.
```swift
// Playable과 Stoppable 프로토콜을 모두 준수하는 타입만 받는 함수
func activate(media: Playable & Stoppable) {
    print("\(media.title) 미디어를 활성화합니다.")
    media.play()
    //... 미디어 재생 중 작업...
    media.stop()
}

// 두 프로토콜을 모두 준수하는 Movie 클래스
class Movie: Playable, Stoppable {
    var title: String
    
    init(title: String) {
        self.title = title
    }
    
    func play() {
        print("영화 \(title) 재생 시작.")
    }
    
    func stop() {
        print("영화 \(title) 재생 중지.")
    }
}

let inception = Movie(title: "인셉션")
activate(media: inception)


프로토콜 조합은 typealias를 사용하여 더 읽기 쉬운 이름으로 만들 수도 있다.

typealias ControllableMedia = Playable & Stoppable

func deactivate(media: ControllableMedia) {
    //...
}
```

## 프로토콜 확장
extension 키워드를 사용하여 기존 프로토콜에 새로운 기능을 추가하거나, 프로토콜이 요구하는 메서트 및 계산 프로퍼티에 대한 기본 구현을 제공할 수 있다.
프로토콜에 기본 구현을 제공하면, 해당 프로토콜을 준수하는 모든 타입은 별도의 구현 없이도 그 기능을 즉시 사용할 수 있다. 필요하다면 override하여 맞춤형 동작을 구현하는 것도 가능하다.
```swift
protocol Describable {
    var description: String { get }
    func describe()
}

// Describable 프로토콜에 기본 구현 제공
extension Describable {
    func describe() {
        print("--- 설명 ---")
        print(description)
        print("------------")
    }
}

struct Book: Describable {
    var title: String
    var author: String
    
    // description 프로퍼티는 반드시 구현해야 함 (요구사항)
    var description: String {
        return "'\(title)' by \(author)"
    }
    
    // describe() 메서드는 기본 구현을 사용하므로 구현할 필요 없음
}

class Product: Describable {
    var name: String
    var price: Double
    
    init(name: String, price: Double) {
        self.name = name
        self.price = price
    }
    
    // description 프로퍼티 구현
    var description: String {
        return "\(name) - $\(price)"
    }
    
    // describe() 메서드를 재정의하여 커스텀 동작 추가
    func describe() {
        print("===== 상품 정보 =====")
        print(description)
        print("===================")
    }
}

let swiftBook = Book(title: "The Swift Programming Language", author: "Apple")
swiftBook.describe()

let macbook = Product(name: "MacBook Pro", price: 2499.0)
macbook.describe()
```

# associatedtype을 통한 제네릭 프로토콜
프로토콜은 때때로 자신을 준수하는 타입을 알지 못한 채 타입에 대한 요구사항을 일반화해야 할 필요가 있다. 이때 사용되는 것이 바로 associatedtype이다.
#### associatedtype소개
associatedtype은 프로토콜 정의 내에서 사용될 타입에 대한 플레이스 홀더(임시로 사용하는 기호)역할을 한다.
### associatedtype vs. 제네릭 (\<T>)
이 둘은 유사한 목적을 갖지만 사용되는 주체와 방식에서 차이가 있다.
#### 1. associatedtype: 
   프로토콜에 associatedtype item이 있는 경우, item의 구체적인 타입은 프로토콜을 준수하는 타입이 자신의 구현을 통해 결정한다.
```swift
// associatedtype을 가진 프로토콜: '재생 목록'을 가진 모든 플레이어의 계약
protocol MusicPlayer {
    associatedtype Music // <- 여기에 associatedtype: 이 플레이어는 어떤 'Music' 타입을 다룬다.
    mutating func add(_ music: Music) // Music 타입의 음악을 추가
    func playNext() // 다음 음악 재생 (Music 타입)
}

// 1. 'Song' 타입의 음악만 재생할 수 있는 플레이어 구현
struct SimpleSongPlayer: MusicPlayer {
    private var playlist: [String] = [] // String 타입의 음악 목록
    
    mutating func add(_ music: String) { // Music이 String으로 결정
        playlist.append(music)
        print("재생 목록에 '\(music)'을(를) 추가했습니다.")
    }
    
    func playNext() {
        if !playlist.isEmpty {
            let nextSong = playlist.removeFirst()
            print("'\(nextSong)'을(를) 재생합니다.")
        } else {
            print("재생 목록이 비어 있습니다.")
        }
    }
}

// 2. 'Podcast Episode' 타입의 음악만 재생할 수 있는 플레이어 구현
struct SimplePodcastPlayer: MusicPlayer {
    // typealias Music = String // Swift가 아래 add 메서드의 파라미터 타입으로 Music이 String임을 추론
    private var episodeList: [String] = [] // String 타입의 에피소드 목록
    
    mutating func add(_ music: String) { // Music이 String으로 결정되었으므로 여기도 String
        episodeList.append(music)
        print("에피소드 목록에 '\(music)'을(를) 추가했습니다.")
    }
    
    func playNext() {
        if !episodeList.isEmpty {
            let nextEpisode = episodeList.removeFirst()
            print("'\(nextEpisode)'을(를) 재생합니다.")
        } else {
            print("에피소드 목록이 비어 있습니다.")
        }
    }
}

let songPlayer = SimpleSongPlayer()
songPlayer.add("좋은 노래") // 재생 목록에 '좋은 노래'을(를) 추가했습니다.
songPlayer.playNext() // '좋은 노래'을(를) 재생합니다.

let podcastPlayer = SimplePodcastPlayer()
podcastPlayer.add("재밌는 팟캐스트") // 에피소드 목록에 '재밌는 팟캐스트'을(를) 추가했습니다.
podcastPlayer.playNext() // '재밌는 팟캐스트'을(를) 재생합니다.

// 오류 발생: 노래 플레이어에 팟캐스트 에피소드를 넣을 수 없습니다. (컴파일 에러)
// songPlayer.add(podcastPlayer.episodeList.first!)
비록 둘 다 `MusicPlayer` 프로토콜을 준수하고 `associatedtype Music`을 `String`으로 사용하기로 했지만, 서로 다른 구체적인 타입(구조체)의 인스턴스여서 호환되지 않는다.
```

#### 2. 제네릭 (\<T>): 
   제네릭을 사용하는 구체적인 타입이나 함수를 호출하거나 인스턴스화하는 쪽에서 구체적인 타입을 결정한다.
```swift
// 제네릭 구조체: 어떤 색깔 펜이든 담을 수 있는 상자
struct ColorPenBox<PenColor> { // <PenColor>가 제네릭 타입 플레이스홀더
    var color: PenColor // 상자 안에 'PenColor' 타입의 색깔 하나
    
    init(color: PenColor) {
        self.color = color
    }
    
    func describe() {
        print("이 상자에는 \(color) 색 펜이 들어있습니다.")
    }
}

// 1. 빨간 펜 상자 생성 (PenColor가 String 타입으로 결정됨)
let redPenBox = ColorPenBox(color: "빨강") // Swift가 String으로 추론
redPenBox.describe() // 출력: 이 상자에는 빨강 색 펜이 들어있습니다.

// 2. 파란 펜 상자 생성 (PenColor가 String 타입으로 결정됨)
let bluePenBox = ColorPenBox(color: "파랑") // Swift가 String으로 추론
bluePenBox.describe() // 출력: 이 상자에는 파랑 색 펜이 들어있습니다.

// 3. 숫자 펜 상자 생성 (PenColor가 Int 타입으로 결정됨 - 이상하지만 예시를 위해)
let numberPenBox = ColorPenBox(color: 10)
numberPenBox.describe() // 출력: 이 상자에는 10 색 펜이 들어있습니다.
```

https://docs.google.com/document/d/1_x1lArkXzqvBFQgWz2m6MxIp3F08nqW5lksVKqvZIHo/edit?tab=t.0