try로 호출시 에러처리는 
1. `do-catch`구문
2. `throws` 키워드가 있는 `throwing function`
이 두가지를 통해서 합니다.

## do-catch
do절에서 실행된 try에서 발생한 에러를 catch에서 처리하는 방법
실행 방법
1. try에서 에러가 발생한다. 
2. 에러가 발생한다면 이후 코드는 실행하지 않고 바로 catch로 이동해서 해당하는 에러를 찾는다.

## throwing function
throws라는 키워드를 붙이면 에러 전달이 가능한 throwing function입니다.
throwing function는 발생한 에러를 호출된 범위로 최상단 함수까지 전달됩니다.
**throws 키워드가 없는** 함수일 경우 함수 내부에서 발생한 에러는 반드시 함수 내부에서 do-catch를 사용해 처리가 되어야 합니다.