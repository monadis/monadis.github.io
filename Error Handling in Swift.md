# Error Handling in Swift

https://andybargh.com/error-handling-in-swift/#Recoverable_Error_Conditions

에러 핸들링이 복잡한 이유: 에러는 에러가 발생한 코드 위치가 아니라 사용자 측에서 핸들링 해야 되는 경우가 많다.

에러 핸들링의 Golden Rule : 에러가 발생할 가능성이 조금이라도 있다면 이를 명확히 해라. 무시하여 묻어놓고 에러가 발생하지 않기를 기도하지마라.   

## 에러의 종류

-   내부 에러 : 코드 논리 구조에 문제가 있어 발생하는 에러. 로직 에러라고도 한다. assert문 등을 이용해서 잡아낸다. 
-   외부 에러: 현재 스코프의 코드는 이상이 없는데 외부조건이나 입력값의 잘못으로 인해 발생하는 에러다. 외부에서 처리해야 하므로 throw error 혹은 리턴값을 통해 에러가 났음을 알려준다.


  #### Logical Error Conditions

객체가 있어야 할 곳에 nil이라거나, 배열의 out of range 인덱스에 접근하는 등 논리적인 오류가 발생한 경우. 결과적으로 앱 크래시가 일어남. 

  #### Simple Error Conditions

```swift
  let value : Int? = Int("Hello") // nil
```
  에러 발생시 그 이유가 명백한 에러. Optional을 이용하여 에러를 처리하면 됨.

  #### Recoverable Error Conditions
  몇몇 에러들은 대응 코드를 실행하거나 사용자에게 해결방법을 알려줌으로써 해결이 가능하다. 

## 복구 가능한 에러를 다루는 법

```swift
func contentsOf(file filename: String) -> String? {
    // ...
}
```
위의 메소드는 파일을 읽는데 에러발생시 nil을 반환함. 하지만 파일 읽기 에러가 나는 원인은 다양함. nil만 반환해서는 무슨 에러가 발생했는지 알기 힘듬.

이를 해결하는 두 가지 방법이 있음.
1. Result type을 이용하는 방법
2. Error throw를 이용하는 방법





#### Error Throw를 이용한 방법

다 아는 방법이므로 자세한 내용은 생략...

```swift
func contentsOf(file filename: String) throws -> String {
    //...
}
```

##### rethrow

현 스코프에 에러의 원인이 있을리 없을 경우 그 처리를 상위 스코프로 다시 throw해준다. 

```swift
func log(description: () throws -> String) rethrows -> () {
    print(try description())
}
```

##### try? 와 try!

try-catch 구문을 쓸 필요가 없는 경우가 있다.

guard 구문을 이용해 에러가 발생할 사전 조건을 미리 검사하고 차단한 경우이다.

그래서 실제 에러가 발생하는 조건이 단 하나만 남았다면 `try?`를 사용해도 될 것이다.

` let result = try? throwingFunction()`

혹은 에러가 발생할 조건을 모두 차단했다면 `try!`를 써도 될 것이다.





#### Result Type을 이용한 방법
 에러 목록을 enum에 작성한다. 
```swift
enum FileError : ErrorProtocol {
	case notFound
	case permissionDenied
	case unknownFormat
}
```
  Result Type을 작성한다.
```swift
enum ResultType<T, E> {
	case failure(E)
	case success(T)
}
```

메소드는 아래와 같이 고치면 된다.
```swift
func contentsOf(file filename: String) -> ResultType<String, FileError> {
    //…
    return Result.failure(.notFound)
}
```
에러 처리는 다음과 같이 한다.
```swift
let filename = "source.txt"
let result = contentsOf(file: filename)

switch result {
case let .success(content):
    print(content)
case let .failure(error):
    switch error {
    case .notFound:
        print("Unable to find file \(filename).")
    case .permissionDenied:
        print("You do not have permission to access the file \(filename).")
    case .unknownFormat:
        print("Unable to open file \(filename) - incompatible format.")
    }
}
```

* Result Type을 이용한 방법은 비동기 함수에서 특히 유용하다. 이것을 error throw방식으로 구현하려면 다소 복잡해진다.(해당 방법은 맨 위의 링크에 자세히 설명되어있다.)










