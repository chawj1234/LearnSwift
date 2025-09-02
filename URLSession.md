#### URLSesseion: 네트워킹 작업 관리자
네트워크 데이터 전송 작업을 생성하고 관리하는 객체, 앱과 서버 간의 통신 세션을 나타낸다.
- `URLSession.shared`: 간단한 데이터 요청에는 `shared`인스턴스를 사용하면 된다.
#### URL, URLRequest: 요청 명세서
- `URL`: 요청을 보낼 서버의 주소를 나타내는 구조체
- `URLRequest`: `URL`보다 더 구체적인 요청 명세서
	- HTTP Method: 데이터 요청 방식 (`GET`: 데이터 조회, `POST`: 데이터 생성 등)
	- Headers: 요청에 대한 부가 정보
	- Body: 서버로 전송할 데이터
#### URLSession.shared.dataTask: 실제 작업 실행
`dataTask`는 `URLSession`이 생성하는 여러 종류의 작업(Task) 중 하나로, 
서버로부터 데이터를 받아 한번에 메모리에 `Data`객체 형태로 가져오는 작업
- `dataTask(with:completionHandler:)`: 가장 일반적으로 사용되는 `dataTask` 생성 메소드이다. 이 메소드는 비동기적으로 동작한다.
	- `with`: 앞서 생성한 `URLRequest` 객체를 전달받아, 어떤 요청을 보낼지 결정합니다.
	- `completionHandler`: 네트워크 작업이 완료된 후 실행될 코드 블록(클로저)입니다. 이 클로저는 세 개의 파라미터를 가진다.
	    1. `Data?`: **요청이 성공**했을 경우, **서버로부터 받은 원시 데이터**(raw data). 0과 1로 이루어진 바이너리 데이터 덩어리이며, 아직 앱에서 바로 사용할 수 있는 형태가 아니다
	    2. `URLResponse?`: HTTP 헤더나 상태 코드와 같이, **서버의 응답**에 대한 메타데이터를 담고 있다. 보통 `HTTPURLResponse` 타입으로 캐스팅하여 상태 코드(`statusCode`)를 확인한다. (예: `200`은 성공, `404`는 찾을 수 없음).
	    3. `Error?`: 인터넷 연결 문제나 서버 주소 오류 등 네트워크 단에서 **요청이 실패**했을 경우, **실패 원인에 대한 정보**를 담고 있는 객체이다.
        
- `.resume()`: `dataTask`는 생성 직후에는 '일시정지' 상태. 반드시 이 메소드를 호출해야 실제 네트워크 요청이 시작된다.

##### `URLSession의 다른 Task 종류`
dataTask 외에 다음과 같은 Task들을 제공한다.
- `URLSessionUploadTask(업로드 작업)`: 서버로 대용량 데이터를 백그라운드에서 업로드하는데 최적화된 작업
- `URLSessionDownloadTask(다운로드 작업)`: 서버에서 대용량 파일을 다운로드하는 데 특화된 작업
- `URLSessionWebSocketTask(웹소켓 작업)`: 채팅 앱처럼 실시간으로 데이터를 주고 받는 양방향 통신 작업

#### Codable: 데이터 구조 정의서 (Decodable+Encodable)
struct, class와 같은 타입이 외부 표현(예: JSON)과 자신을 서로 변환할 수 있음을 나타내는 프로토콜
- `Decodable`: 외부 데이터(JSON)를 Swift 타입의 인스턴스로 변환(Decoding) 할 수 있는 기능을 제공
- `Encodable`: Swift 타입의 인스턴스를 외부 데이터(JSON)로 변환(Encoding) 할 수 있는 기능을 제공
서버로부터 받은 JSON 데이터의 키(Key)와 Swift `struct`의 프로퍼티 이름이 일치하면, `Codable`채택만으로 자동 변환이 가능해진다!

```swift
// 서버에서 아래와 같은 JSON을 받는다고 가정
{
    "id": 1,
    "title": "My First Post",
    "userId": 10
}


// 위 JSON에 대응하는 Codable을 준수하는 Swift 구조체
struct Post: Codable {
    let id: Int
    let title: String
    let userId: Int
}
```

#### JSONDecoder와 decoder.decode: 데이터 변환기
- `JSONDecoder`: JSON 형식의 `Data`를 `Codable`을 준수하는 swift 타입으로 변환(Decoding)하는 작업을 전문적으로 수행하는 객체
- `decoder.decode(_:from:)`: `JSONDecoder`가 가진 핵심 메소드
	- **첫 번째 파라미터**(_ type: T.Type): 어떤 swift 타입으로 변환할지 타입 자체를 전달한다. ex) `Post` 배열로 변환하고 싶다면 `[Post].self`를 전달한다. `.self`를 통해 인스턴스가 아닌 타입 자체임을 명시한다.
	- **두 번째 파라미터**(from data: Data): `dataTask`의 `completionHandler`를 통해 받은 원시 `Data` 객체를 전달한다.
	- **반환 및 오류**: 디코딩에 성공하면 지정한 타입의 인스턴스를 반환한다. 만약 JSON 데이터 구조가 swift 타입과 일치하지 않으면 error를 throw 한다. 따라서 이 메소드는 보통 `do-catch` 구문 안에서 호출된다.

### URLSession 과정 요약
1. `URLSession`을 사용하여 `URLRequest`를 기반으로 `dataTask`를 생성하고 실행한다.
2. 작업 완료 후 `complettionHandler`에서 서버로부터 `Data`를 받습니다.
3. `JSONDecoder`를 생성한다.
4. `decoder.decode` 메소드를 호출하여 `Data`를 `Codable`을 준수하는 swift 모델 객체로 변환한다.
5. 변환된 객체를 앱의 로직에 활용한다.

```swift
import Foundation

// 1. 📝 JSON 데이터와 일치하는 Swift 모델을 정의합니다. (Codable 준수)
// 서버에서 받을 데이터의 청사진입니다.
struct Todo: Codable {
    let id: Int
    let title: String
    let completed: Bool
}

// 2. ➡️ 네트워크 요청을 수행하는 함수를 만듭니다.
func fetchTodo() {
    // 3. 요청할 URL 주소를 정의합니다.
    let urlString = "https://jsonplaceholder.typicode.com/todos/1"
    guard let url = URL(string: urlString) else {
        print("Error: 유효하지 않은 URL입니다.")
        return
    }
    
    // 4. dataTask를 생성합니다. (아직 시작은 안 함)
    // 네트워크 작업이 끝나면 completionHandler 클로저가 실행됩니다(비동기로 다른 일 하다가 수행).
    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        
        // --- 여기서부터는 네트워크 작업이 완료된 후 실행되는 영역 ---
        
        // 5. 🚦 응답 결과를 확인하고 처리하는 '안전 장치' 코드
        
        // 5-1. 네트워크 전송 자체에 에러가 있었는지 확인합니다.
        if let error = error {
            print("Error: 네트워크 요청 실패 - \(error.localizedDescription)")
            return
        }
        
        // 5-2. 서버 응답이 유효한지, 상태 코드가 성공 범위(200-299)인지 확인합니다.
        guard let httpResponse = response as? HTTPURLResponse,(200...299).contains(httpResponse.statusCode) else {
            print("Error: 유효하지 않은 서버 응답입니다.")
            return
        }
        
        // 5-3. 받아온 데이터가 존재하는지 확인합니다.
        guard let data = data else {
            print("Error: 데이터를 받지 못했습니다.")
            return
        }
        
        // 6. ✅ 모든 확인이 끝나면, 데이터를 디코딩합니다.
        do {
            let decoder = JSONDecoder()
            let todo = try decoder.decode(Todo.self, from: data)
            
            // 성공적으로 변환된 객체를 사용합니다.
            // UI 업데이트는 반드시 메인 스레드에서 수행해야 합니다.
            DispatchQueue.main.async {
                print("--- 성공 ---")
                print("ID: \(todo.id)")
                print("Title: \(todo.title)")
                print("Completed: \(todo.completed)")
            }
            
        } catch {
            // 🚨 디코딩 과정에서 에러가 발생한 경우
            print("Error: JSON 디코딩 실패 - \(error.localizedDescription)")
        }
    }
    
    // 7. 🚀 준비된 작업을 시작합니다!
    task.resume()
}

// 함수를 호출하여 네트워크 요청을 시작합니다.
fetchTodo()

--- 예상 실행 결과 ---
--- 성공 ---
ID: 1
Title: delectus aut autem
Completed: false
```

### 핵심 로직 설명
1. 네트워크 자체에 문제가 있었나? (`error` 확인)
	- 인터넷 연결이 끊겼거나 주소를 잘못 입력하는 등 통신 자체가 불가능했는지 확인
2. 통신은 됐는데, 서버가 거절했나? (`response` 확인)
	- 통신은 성공했지만 서버가 페이지 없음(404), 내부 오류(500) 같은 응답을 주지 않았는지 상태 코드 확인
3. 데이터는 잘 도착했다. (`data` 확인)
	- 통신 성공, 서버 응답도 정상이지만 데이터가 비어있지 않은지 최종 확인
4. 데이터가 잘 변환되었나? (`do-catch`로 디코딩)
	- 위 관문을 통과한 안전한 `data`를 `Todo` 객체로 변환한다. JSON 형식과 `Todo` 구조체가 맞지 않으면 에러가 발생하기에 `do-catch`로 처리한다.














