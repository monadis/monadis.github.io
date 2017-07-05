



## Swift Tips

#### 숫자를 Optional로 감싸는 방법

```swift
let d = Optional(1)
```

Optional Type의 String을 출력하면

`Optional("this is a string.")`

이런식으로 나오는데 이를 막으려면 아래와 같이 하면 된다.
```swift
print(str ?? "nil")   // nil에 큰따옴표가 있음을 유의하자.
```

#### 1,000 을 1_000으로 표현 가능하다.

Objective-C에서 nil은 "존재하지 않는 객체로의 포인터"를 나타내지만 Swift의 nil은 "값이 없음"을 나타냅니다. 즉, 전자는 포인터이지만, 후자는 포인터가 아닙니다. 전자는 객체형에만 사용할 수 있지만, 후자는 optional한 모든 자료형에 사용할 수 있습니다. 그리고 optional 변수(또는 상수)를 선언할 때 초기값을 지정해 주지 않으면 자동으로 nil이 할당됩니다.

Objective-C에서는 UINT32_MAX와 같은 상수를 통해서 최대/최소값을 알아낼 수 있지만, Swift에서는 min, max 속성을 사용해서 자료형의 값의 범위를 확인할 수 있습니다.
`Int.min Int.max`

정수형을 사용할 때는 대부분 UInt16, Int32 등과 같은 자료형 대신 Int(또는 UInt)를 사용하게 될 것입니다. Int는 C나 Objective-C의 int와 마찬가지로 CPU의 WORD와 동일한 크기를 가지는 자료형입니다. 즉, 32비트에서는 Int32가 되고, 64비트에서는 Int64가 되는 것입니다. Int를 사용하는 것은 명시적으로 크기를 지정하는 자료형을 사용하는 것에 비해 일관성과 호환성 측면에서 유리하기 때문에, Swift 문서에서도 Int를 사용할 것을 추천하고 있습니다.

Swift의 Float는 Objective-C의 float에 해당되는 자료형이며 4바이트의 크기를 가집니다. double에 해당되는 자료형은 Double이며 8바이트 크기를 가집니다.

특정 클래스에서만 사용하는 상수는 private를 붙이자.
연관된 상수들은 enum으로 만들자.
 여러개의 프로토콜을 하나의 클래스에서 구현하지말자. 각각을 extension으로 분리시키자.
3회 반복하는 루프문은 아래와 같이하자.
```swift
for _ in 1...3 {
  println("Hello three times")
}
```

단, 역순은 3...1 이 아니라 (1...3).reverse() 로 해야 함.


몇몇 함수는 아래처럼 인자를 명확히 드러내는게 낫다.
```swift
let bounds = CGRect(x: 40, y: 20, width: 120, height: 80)
let centerPoint = CGPoint(x: 96, y: 42)
가능하면 네이티브 타입을 사용하라. Objective-c 타입은 거기에만 있는 함수가 필요할때만 사용.
let width: NSNumber = 120.0                          // NSNumber
let widthString: NSString = width.stringValue        // NSString
```
보다는 아래를 사용하자.
```swift
let width = 120.0                                    // Double
let widthString = (width as NSNumber).stringValue    // String 
```
산술연산은 overflow 금지. 오버플로 지원 연산자는 따로 도입 `(&-,&+,&/,&*,&%)`
```swift
var a = Int.max		// 9223372036854775807
a = a + 1      // 에러
a = a &+ 1    // 에러안남.  -9223372036854775808
```





#### AnyObject vs. Any : 가능하면 Any를 쓰자.
obj-c에서의 id에 해당하는것이 AnyObject?이다. 어떤 타입이든 넣을수 있는 배열을 만들려면 Any 혹은 AnyObject키워드를 써서 선언한다.

Any는 AnyObject의 스위프트 버전이라고 할수 있다. Any 키워드를 사용하면 참조타입의 객체는 저장을 할수 없지만 Int, Double, Float, String, Array, Dictionary 과 같은 구조체부터 tuple과 function type까지도 저장이 가능하다.
AnyObject는 Objective-C와의 호환성을 위해 사용된다. 여기에 구조체를 저장하는것도 가능하지만 자동으로 NSNumber, NSString, NSArray, NSDictionary등의으로 변환되어 저장된다. 따라서 호환성이 필요없다면 Any를 사용하는것이 성능상 이득이다.  

