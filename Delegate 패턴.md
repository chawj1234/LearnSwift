델리게이트 패턴 왜 사용하는 걸까?
한 객체가 너무 많은 일을 하려고 하면 코드가 복잡해지고 수정하기 어렵기 때문이다.

이를 위해 3명이 필요하다
1. Delegator(일을 시키는 객체)
2. Protocol (업무 규칙서)
3. Delegate (일을 처리하는 객체)
#### Delegator
실제 처리 방법은 모른다. 그저 '때가 되면' 대리인을 호출한다.
#### Protocol
Delegate이 되려면 '이런 기능이 있어야 한다'는 설계도
**Delegator와 Delegate 사이의 의사소통 창구**이자 **개발자와 코드 사이의 소통 창구**
#### Delegate
Protocol을 따르고, 기능을 처리한다.

### 델리게이트 패턴 구현 4단계

1. Protocol 정의하기
- 어떤 기능을 해야하는지 명시한다.
```swift
//"보고서 대리인이 되려면, 반드시 '보고서 생성' 기능을 구현해야 합니다." 
protocol ReportDelegate { 
	func createReport() -> String 
	// 보고서를 만들어 문자열로 반환하는 기능 
}
```

2. Delegator 만들기
- Delegator class를 만들고 Delegate 자리를 만든다.
```swift
class ReportBot { // 보고가 필요한 로봇 (Delegator) 
	// ReportDelegate 자격증을 가진 어떤 Delegate가 앉을 수 있는 자리 
	weak var delegate: ReportDelegate? 
	
	// 특정 조건이 되면 보고서를 요청하는 함수 
	func generatePerformanceReport() { 
		print("🤖 로봇: 성능 분석이 완료되었습니다. 보고서 생성을 요청합니다...") 
		
		// 만약 대리인이 지정되어 있다면, 일을 시킨다! 
		if let report = delegate?.createReport() { 
			print("--- 보고서 내용 ---") 
			print(report) 
			print("------------------") 
		} else { 
			print("⚠️ 로봇: 보고서를 생성할 대리인이 지정되지 않았습니다.") 
		} 
	} 
}
```

3. Delegate 만들기
- Protocol을 준수하는 Delegate을 만든다.
```swift
// "제가 바로 ReportDelegate 자격증을 가진 데이터 분석가입니다." 
class DataAnalyst: ReportDelegate { 

	// 규칙서에 약속된 createReport 기능을 실제로 구현 
	func createReport() -> String { 
	
		// 복잡한 데이터 분석 과정을 거쳤다고 가정... 
		return "📈 2분기 매출이 20% 상승했으며, 주 고객층은 30대입니다." 
	} 
}
```

4. Delegator와 Delegate을 연결한다.
- 독립적으로 만들어진 Delegator와 Delegate을 연결해준다.
```swift
// 1. 로봇(Delegator)과 분석가(Delegate)를 각각 선언 
let bot = ReportBot() 
let analyst = DataAnalyst() 

// 2. 로봇의 대리인으로 분석가를 공식 임명
bot.delegate = analyst 

// 3. 로봇에게 일을 시킴
bot.generatePerformanceReport()
```

\> 실행 결과
```
🤖 로봇: 성능 분석이 완료되었습니다. 보고서 생성을 요청합니다... 
--- 보고서 내용 --- 
📈 2분기 매출이 20% 상승했으며, 주 고객층은 30대입니다.
------------------
```

### 델리게이트 패턴의 장점
1. 분리(Decoupling): 일을 시키는 코드와 실제 처리하는 코드가 완전히 분리된다.
2. 재사용성: Delegator의 코드를 한 줄도 바꾸지 않고 다른 Delegate를 임명할 수 있다.
```swift
class FinanceTeam: ReportDelegate {
    func createReport() -> String {
        return "💰 2분기 지출은 5% 감소했으며, 예산이 10% 남았습니다."
    }
}
//Delegate 선언
let financeExpert = FinanceTeam()
//임명
bot.delegate = financeExpert // 전문가만 바꿔 끼우기!
//로봇에게 일을 시킴
bot.generatePerformanceReport() // 결과가 달라짐!
```
3. 확장성: 새로운 기능 추가시 새로운 Delegate class만 만들면 된다.
### 주의점: 메모리 누수
Delegator와 Delegate 사이에 강한 참조가 생길 수 있다. 이를 방지하기 위해 Delegate을 선언할 때 거의 항상 `weak` 키워드를 사용한다.
> `weak var delegate: ReportDelegate?`

**delegate 변수 앞에는 `weak`을 붙이는 것이 규칙**
