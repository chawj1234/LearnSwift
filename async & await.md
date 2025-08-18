async/await 기본
	•	async 함수 작성과 호출 방법
	•	Task 생성과 사용법
	•	completion handler → async/await 변환 연습

병렬 처리
	•	async let으로 병렬 실행
	•	순차 vs 병렬 성능 비교 실습
	•	실제 API 호출 예제로 체감하기

TaskGroup 기본
	•	withTaskGroup 사용법
	•	동적 개수의 작업 병렬 처리
	•	에러 처리와 결과 수집

Actor와 @MainActor
	•	Actor 타입 이해와 활용
	•	@MainActor로 UI 업데이트 안전하게 처리
	•	Data race 방지 실습


---

# 기본 개념 정립
## 비동기
**작업의 결과를 기다리지 않고 다음 작업으로 넘어가는 것을 의미한다.**
네트워크 요청과 같이 시간이 오래 걸리는 작업을 시작한 후, 그 작업이 완료되기를 기다리지 않고 즉시 반환하여 프로그램의 다른 부분이 계속 실행될 수 있도록 한다. 작업의 결과는 나중에 콜백이나 반환값을 통해 전달된다.

Swift에서 `async`로 선언된 함수는 실행 도중 스스로를 **일시 중단**하고 자신이 사용하던 쓰레드를 시스템에 양보한다. 이를 통해 시스템 자원을 효율적으로 사용하게 한다. 

## 동시성
**독립적으로 실행되는 여러 작업을 구성하고 관리하는 것을 의미한다.** 즉, 한 번에 여러 가지 일을 처리하는 개념
Swift에서는 '동시성'이라는 용어를 비동기 코드와 병렬 코드를 포괄하는 상위 개념으로 사용합니다. Swift의 동시성 시스템은 개발자가 직접 스레드를 관리하는 대신, `Task`와 같은 추상화된 작업 단위를 통해 여러 작업을 관리하며, 시스템이 이를 효율적으로 스케줄링합니다.

## 병렬성
**여러 작업이 물리적으로 동시에 실행되는 것**을 의미한다. 이를 위해서 반드시 여러 개의 CPU 코어가 필요합니다.
Swift의 런타임(Swift 언어로 작성된 프로그램이 실제로 실행될 때 필요한 모든 지원 기능을 제공하는 시스템)은 멀티코어 하드웨어 환경에서 **시스템 스케줄러**가 자원 상황에 따라 동시성 작업을 병렬적으로 실행할 수 있습니다.
이를 통해 개발자의 사고 방식은 '스레드 관리'에서 '작업 구조화'로 전환되며, 이는 복잡성을 줄이고 잠재적 오류를 방지하는 고수준의 추상화를 제공합니다.

# async/await 문법
**스레드**: 컴퓨터에서 작업을 실행하는 **작업의 흐름이자 최소 단위**
## 핵심 키워드
- async: 함수, 메서드, 클로저가 비동기적임을 나타내는 키워드. 즉, 실행을 일시 중단할 수 있음을 나타낸다.
- await: 잠재적 일시 중단 지점을 명시하는 키워드. await 키워드를 사용해서 호출된 async 함수의 결과를 기다릴때까지 스레드를 다른 작업에 양보한다. 
	즉, Swift 런타임에게 "이 결과를 기다려야 하니, 나의 현재 상태를 잠시 저장해두고 이 스레드는 다른 급한 작업에 사용하세요"라고 알리는 것과 같습니다.
## async throws를 통한 통합된 오류 처리
async 함수는 throws 키워드를 함께 사용하여 오류를 발생시킬 수 있다. 이를 통해 비동기 작업에서 발생한 오류가 호출 스택에 따라 전파됩니다. 
try await을 사용하여 async throws 함수를 호출하고, 이를 do-catch 블록으로 감싸면 모든 단계에서 발생하는 오류를 한 곳에서 일관되게 처리할 수 있습니다.
```swift
Task {  
    do {  
        let finalURL = try await processFirstFriendImage(for: someUserID)  
        print("Successfully processed and uploaded image: \(finalURL)")  
    } catch MyError.noFriends {  
        print("User has no friends to process.")  
    } catch {  
        print("An unexpected error occurred: \(error)")  
    }  
}
```

# Task
Task{...}를 통해 비동기 작업임을 나타내는 기본 단위, 스코프의 생명주기에 얽매이지 않으며 부모 스코프로부터 취소 신호가 자동으로 전파되지 않는다.
특히 **Task.detached{...}**는 1. 동기적인 컨텍스트에서 비동기 작업을 시작할때, 2. 생명주기 독립성이 필요할때 사용한다.
둘의 차이점은 
1. 컨텍스트(스코프, 우선순위, Actor 격리)의 상속 여부
2. 독립성의 정도와 책임

Task는 생성된 현재 컨텍스트를 상속 받고 이때문에 독립성의 정도가 상대적으로 약하다. 그러나Task.detached는 부모의 어떤 컨텍스트도 상속받지 않기 때문에 독립성이 강하다.

## async let을 이용한 병렬 처리
`async let`은 **정적이고, 실행 시점에 이미 개수가 정해진** 독립적인 비동기 작업들을 병렬로 실행할 때 사용한다.
`async let`으로 변수를 선언하면 변수에 할당된 비동기 함수는 즉시 자식 Task로 분기하여 실행을 시작하고 부모 Task는 자신의 코드를 계속 실행하다가 async let으로 선언된 변수의 결과값이 필요한 시점에서 await을 사용하여 결과값을 기다립니다. 
```swift
func loadUserProfileAndPosts(for userID: UserID) async throws -> (Profile, [Post]) {  
    async let profile = fetchProfile(id: userID) // 즉시 병렬 실행 시작  
    async let posts = fetchPosts(for: userID)   // 즉시 병렬 실행 시작  
     
    // 부모 Task는 다른 작업을 계속 수행할 수 있음  
     
    // 결과가 필요한 시점에서 await를 통해 대기  
    let userProfile = try await profile  
    let userPosts = try await posts  
     
    return (userProfile, userPosts)  
}
```

## TaskGroup을 이용한 동적 병렬 처리
`TaskGroup`은 **동적이고, 실행 시점에야 개수가 정해지는** 여러 비동기 작업을 병렬로 처리할 때 사용됩니다.
1. withTaskGroup 또는 오류 처리가 포함된 withThrowingTaskGroup을 사용하여 그룹을 생성
2. group.addTask {... }를 통해 동적으로 자식 Task를 추가
3. 모든 자식 Task의 결과는 for await 루프를 통해 비동적으로 수집
```swift
func downloadImages(from urls:) async throws -> [UIImage] {  
    var images: [UIImage] =  
    try await withThrowingTaskGroup(of: UIImage.self) { group in  
        for url in urls {  
            group.addTask {  
                // 각 다운로드 작업은 병렬로 실행됨  
                return try await downloadImage(from: url)  
            }  
        }  
         
        // 완료되는 순서대로 결과를 수집  
        for try await image in group {  
            images.append(image)  
        }  
    }  
    return images  
}
```

## 작업 취소 방식
Swift의 동시성 시스템에서 작업(Task)을 취소하는 방식은 선점형이 아닌 협력형입니다. 이는 매우 중요한 개념이다.
1. **협력적 취소의 원리:**
   task.cancel() 메서드를 호출하는 것은 해당 작업 내부에 isCancelled라는 플래그를 true로 설정하는 '취소 신호'를 보내는 것일 뿐, 현재 실행 중인 작업을 즉시 강제로 중단시키지 않는다. 따라서 주기적으로 이 isCancelled 플래그를 확인하고, 취소 요청이 있을 경우 스스로 작업을 안전하게 정리한 후 종료해야 합니다.
2. **취소 상태 확인 방법:**
	1. **try Task.checkCancellation():** 작업이 취소되었다면 CancellationError를 던져 즉시 실행을 중단한다.
	2. **if Task.isCancelled: Bool** 값을 확인하여 작업이 취소되었는지 여부를 확인한다. 이를 통해 작업 일부만 중다, 부분적인 결과 반환과 같은 더 유연한 제어가 가능하다.
3. **구조화된 동시성과 취소의 자동 전파**
	1. `async let`이나 `TaskGroup`과 같은 구조화된 동시성 모델의 가장 큰 장점 중 하나는 취소의 자동 전파이다. 즉, 부모 Task가 취소되면 속한 자식 Task들도 자동으로 취소 신호를 받는다.

| 기능       | await (순차적)             | async let (정적 병렬)         | TaskGroup (동적 병렬)                  |
| -------- | ----------------------- | ------------------------- | ---------------------------------- |
| 주요 사용 사례 | 한 작업의 결과가 다음 작업에 필요할 때  | 정해진 개수의 독립적인 작업을 병렬 실행할 때 | 동적인 개수의 독립적인 작업을 병렬 실행할 때          |
| 실행 흐름    | 선형적, 한 번에 하나의 작업만 일시 중단 | 동시 시작, 단일 조인 포인트에서 결과 대기  | 동시 시작, 완료되는 순서대로 결과 수집             |
| Task 구조  | 새로운 자식 Task를 생성하지 않음    | 구조화된 자식 Task 생성           | 구조화된 자식 Task 그룹 생성                 |
| 취소       | 호출 스택을 따라 오류처럼 전파       | 스코프 종료 시 자식 Task 자동 취소    | 스코프 종료 또는 그룹 취소 시 모든 자식 Task 자동 취소 |
https://yudonlee.tistory.com/39
https://1000ji.tistory.com/4
https://docs.google.com/document/d/1UwgtwxRLtbpUrVFb64XweXKzDYpP2SAWtqZdriI5AkI/edit?tab=t.0