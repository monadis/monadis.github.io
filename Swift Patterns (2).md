# Swift Patterns (2)





## Enum

다른 언어에서와의 enum과는 달리 그 자체로 클래스의 특성을 지님. 현재 값에 대한 부가정보를 제공할수있는 프로퍼티를 소유할수 있으며, 값정보를 표현하는 메소드와 값의 초기화메소드 등을 포함하기도 한다. 기능을 entend할수 있으며 프로토콜을 상속할수도 있다.

특별히 명시하지 않으면 디폴트값이 자동으로 0,1,2… 로 주어지는 다른 언어와는 달리 swift에서는 임의의 값이 주어진다.

enum형으로 만든 타입명은 대문자로 시작해야 하며, 단수형으로 이름짓자.

case를 이용해서 멤버들을 그룹화할수 있다.

```swift
enum CompassPoint {
  case North
  case South
  case East
  case West 
}

var directionToHead = CompassPoint.West
directionToHead = .East

// 한줄로 표현가능.

enum Planet: Int {
  case Mercury = 1, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}

let earthsOrder = Planet.Mercury.rawValue
// earthsOrder is 3

// rawValue를 이용해 enum값을 생성할수 있다.
let possiblePlanet = Planet(rawValue: 7)
// print(directionToHead)를 하면 해당 enum의 내용을 출력해볼수 있다.
```



3상태 스위치를 만들수 있음.

```swift
    enum TriStateSwitch {
        case Off, Low, High

        mutating func next() {
            switch self {
            case Off:
                self = Low
            case Low:
                self = High
            case High:
                self = Off
            }
        }
    }
    var ovenLight = TriStateSwitch.Low
    ovenLight.next() // .High로 변함
    ovenLight.next() // .Off로 변함
```



**Raw Value**

앞에서 Planet은 Int타입이였는데 다음과 같이 문자로도 가능하다.

```swift
// 열거형 뒤에 :로 타입을 지정하고, 각 case 식별자마다 기본값(raw 값)을 지정 가능. 로우값으로 지정가능한 타입은 문자열, 문자, 정수 또는 부동소수 타입 뿐임. 연관값은 case 식별자마다 다 타입을 다르게 부여할 수 있고, 어떤 타입이건 가능하지만, 로우값은 그렇지 않음에 유의할 것.
enum ASCIIControlCharacter: Character {
  case Tab = "\t"
  case LineFeed = "\n"
  case CarriageReturn = "\r"
}

// raw값으로부터 case를 생성하기.
let ascii = ASCIIControlCharacter(rawValue: "\r") // ASCIIControlCharacter? 타입
```



**Associated Value**

enum의 case에 값을 연결할수 있음(연관값, associated value라 부름)

```swift
enum Barcode {
  case UPCA(Int, Int, Int)
  case QRCode(String)
}
var productCode = Barcode.UPCA(8, 85909_51226,3)

// switch로 매치시 case 이름을 사용해 각 성분을 분리해 낼 수 있음
switch productCode {
  case .UPCA(let numberSystem, let identifier, let check):
    print("UPC-A with value of (numberSystem), (identifier), (check).")
  case .QRCode(let productCode):
    print("QR code with value of (productCode).")
}

// 연관값의 enum은 튜플의 확장판에 가깝다. 어떤 연관된 데이터들을 묶고 각 데이터들에 이름을 붙이고 패턴을 분류할수 있게 해준다.

// 각 아이템에 라벨을 붙일수도 있다.  
  case UPCA(a:Int, b:Int, c:Int)
```



**enum을 이용한 함수의 조직화**

데이터뿐만 아니라 함수도 enum을 이용하여 그룹화 할수 있다. 아래 계산기의 예를 보자.

```swift
enum Operator {
  case ADD((Int,Int)->Int)
  case SUB((Int,Int)->Int)
  case MUL((Int,Int)->Int)
  case DIV((Int,Int)->Int)
}
// recursive enum을 구현하려면 indirect를 써줘야 한다.
enum Expr {
  indirect case BINOP(Operator, Expr, Expr)
  case NUMBER(Int)
}
let ADDOP = Operator.ADD(+)
let SUBOP = Operator.SUB(-)
let MULOP = Operator.MUL(*)
let DIVOP = Operator.DIV(/)
func calc(e:Expr) -> Int {
  switch(e) {
  case .BINOP(let op, let e1, let e2):
    switch(op) {
    case .ADD(let f):
      return f(calc(e1), calc(e2))
    case .SUB(let f):
      return f(calc(e1), calc(e2))
    case .MUL(let f):
      return f(calc(e1), calc(e2))
    case .DIV(let f):
      return f(calc(e1), calc(e2))
    }
  case .NUMBER(let v):
    return v
  }
}

// (1*3) + (6/2)
let testExp = Expr.BINOP(ADDOP,
  Expr.BINOP(MULOP,.NUMBER(1),.NUMBER(3)),
  Expr.BINOP(DIVOP,.NUMBER(6),.NUMBER(2))
)
print("(1 * 3) + (6 / 2) = (calc(testExp))")
```

함수 내부구현중 일부만 다른 함수 여러개가 있는경우는 nested function부분을 참고하자.

**Enum 형을 AnyObject타입으로 넘겨줘야 할때는 rawValue로 넘겨줘야 한다.**

```swift
let predicate = MPMediaPropertyPredicate(value:MPMediaType.Music.rawValue, forProperty: MPMediaItemPropertyMediaType!, comparisonType:MPMediaPredicateComparison.EqualTo)
```



**값 순환**

```swift
enum TestEnum: String {
    case Test1, Test2
    mutating func flip() {
        switch self {
            case .Test1:
                self = .Test2
            case .Test2:
                self = .Test1
        }
    }
}
```



[**String을 Enum으로 대체하여 Compile-time safety를 얻자.**](https://gist.github.com/monadis/9a5234ed5093be0dc837)





## Tuple

**타입으로써의 튜플 **

Tuple을 이용하면 복수의 리턴값을 가질수 있다

```swift
func getGasPrices() -> (Double, Double) { return (2.3, 4.2) }
```

외부 파라미터를 이용하면 튜플에 min, max라는 멤버명을 할당할수 있고 닷연산자를 이용하여 접근이 가능하다.

```swift
func minMax(array: [Int]) -> (min: Int, max: Int) {
     var currentMin = array[0]
     var currentMax = array[0]
     for value in array[1..<array.count] {
          if value < currentMin {
               currentMin = value
          } else if value > currentMax {
               currentMax = value
          }
     }
     return (currentMin, currentMax)
}

let bounds = minMax([8, -6, 2, 109, 3, 71])

print("min is \(bounds.min) and max is \(bounds.max)") // min, max값은 bounds.0 과 bounds.1 로도 접근 가능하다.
```

튜플도 (Int, Int)? 와 같은 표기법을 통해 optional로 만들수 있다. 또한 이는 (Int?, Int?)와는 다르다는 것을 유의하자. 후자의 경우 각 멤버가 nil이 될수 있는것이고 전자는 튜플 자체가 nil이 될수 있는것이다.



**익명 구조체로써의 튜플 **

```swift
struct User {
  let name: String
  let age: Int
}

// vs.
let user = (name: "Carl", age: 40)
```



튜플은 익명의 구조체 역할을 한다. 
코딩을 하다보면 서로 연관이 있는 변수들을 여러개 선언하게 되는데 이들을 임시로 묶어서 다루면 가독성이 좋아진다. 

다차원 배열 대신 튜플을 사용할수도 있으며, 옵저버 패턴에서 Listener를 구조체 대신 튜플의 형태로 저장할수도 있다.

만약 scope 범위를 넘어 사용된다면 튜플보다는 구조체 (또는 클래스)가 낫다.

튜플은 타입으로만 사용할수 있는게 아니다. 아래 몇가지 사례를 살펴보자.

**패턴 매칭 도구로써의 튜플**

```swift
let age = 23
let job: String? = "Operator"
let payload: AnyObject = NSDictionary()

// 23살이며 직업이 있고 payload가 NSDictionary타입으로 되어있는 케이스인지 식별하는 코드
switch (age, job, payload) {
  case (let age, _?, _ as NSDictionary) where age < 30:
  print(age)
  default: ()
}
```



switch - case 문을 튜플과 조합해서 사용하면 if - else 구문보다 가독성이 좋다. 자주 사용하자.

**다수의 변수값 대입 도구로써의 튜플**

```swift
// a is "test", b is 12, c is 3, and 9.45 is ignored
var (a, _, (b, c)) = ("test", 9.45, (12, 3))

// 여러개의 함수를 한줄에 호출하여 결과값 얻기
let (a, b, c) = (a(), b(), c())

// Swap 함수 없이 변수값 스왑하기
var a = 5
var b = 4
(b, a) = (a, b)
```



배열을 대신해 튜플을 사용하는것도 가능하다.

```swift
// 짧은 배열 대신 사용하기. 
var monthValues: [Int]
var monthValues: (Int, Int, Int, Int, Int, Int, Int, Int, Int, Int, Int, Int)

// 짧은 다차원 배열 대신 사용하기
var matrix = ((1, 0, 0), (0, 1, 0), (0, 0, 1))
```

배열은 길이에 제한을 줄수 없지만 튜플은 가능하다. 배열을 쓰는경우 런타임에 guard구문을 통해 배열의 크기가 12개를 넘겼는지 여부를 체크해야 한다.  반면, 튜플을 쓰면 컴파일타임에 이 제한규정을 지켰는지 체크가 되므로 디버깅이 쉬워진다.  하지만 루프를 돌리거나 매핑을 할수 없는 등의 단점도 있다.

튜플은 컬렉션이 아니다. 따라서 Type-safe하게 멤버를 Looping하거나 Mapping할 수가 없다. 하지만 우회방법으로 Mirror를 통해 reflection하는 방법이 존재한다. [링크](appventure.me/2015/07/19/tuples-swift-advanced-usage-best-practices/) 참조.



하나의 함수에 1개 밖에 쓸수없는 가변인자의 한계를 튜플로 극복하자.

```swift
func batchUpdate(updates: (String, Int)...) -> Bool {
    self.db.begin()
    for (key, value) in updates {
    self.db.set(key, value)
    }
    self.db.end()
}

// We're imagining a weird database
batchUpdate(("tk1", 5), ("tk7", 9), ("tk21", 44), ("tk88", 12))
```

typealias와 사용해보자.

```swift
typealias Example = (num:Int, age:Int, name:String)

// 이렇게만든 타입을 이용하여 캐스팅하면 순서가 뒤바뀐 튜플도 올바른순서로 reordering이 가능하다.
let e = (name:"choi", age:19, num:89) as Example

// 배열도 마찬가지로 자동 캐스팅된다.
let array:[Example] = [(name:”choi”, age:19, num:89)] 

// Function Header는 튜플과 호환된다.
func foo(a: Int, _ b: Int, _ name: String) -> Int {
    return a
}

let arguments = (4, 3, "hello")
foo(arguments) // returns 4

// 단, 레이블을 일치시켜야 한다.
func foo2(a a: Int, b: Int, name: String) -> Int {
    return a
}

let arguments = (4, 3, "hello")
foo2(arguments) // fails to work
let arguments2 = (a: 4, b: 3, name: "hello")
foo2(arguments2) // works! (4)
```



## Function Factory Method

코딩을 하다보면 유사한 함수를 여러개 만들어야 하는 경우가 생긴다. 이럴때 Nested Function을 이용하여 해결이 가능하다. 클로져는 구현의 일부를 caller에게 받는 플러그인 형태로 주로 사용되는 반면 nested function은 구현을 숨기고 interface만 외부에 전달하는 형태로 사용된다. 공통되는 코드나 변수들을 갖고있는 함수가 여러개라면 nested 함수로 만들면 코드가 줄어든다. 로컬변수를 제한없이 사용할수 있다는 scope적인 이점이 있다.

```swift
enum Test { case C, B }

func nested(_ foo:Test) -> (String) -> String {
    let a = "aaa"
    let f = "fff"
    let d = "ddd"

    func B(_ b:String) -> String {
        return b + a + "bbb" + f + d
    }

    func C(_ c:String) -> String {
        return c + a + "ccc" + d + f
    }

    switch(foo) {
    case .C : return C
    case .B : return B
    }
}

print(nested(.C)("AAA"))
print(nested(.B)("BBB"))
```



## Curried Function

Curried Function도 사실 Function Factory Method와 다를바 없다. 

```swift
// 아래 두 함수는 동일하다. 

// 1 : nested function 방식의 표현.
func addTwoIntsCurried(_ a: Int) -> (Int) -> Int {
    func addTheOtherInt(b: Int) -> Int {
        return a + b
    }
    return addTheOtherInt
}

// 2 : curried function 방식의 표현.
func addTwoIntsCurried(_ a: Int) -> (Int) -> Int {
    return { b in
        return a + b
    }
}

let addThreeTo = addTwoIntsCurried(3)
let result = addThreeTo(4)  // result = 7
(1...5).map(addThreeTo) // result = [4, 5, 6, 7, 8]
```

위에서 보듯, **curried function을 이용하면 함수에 map을 적용할수 있다.**



코딩을 하다보면 인자가 많은 함수를 정의하게 되는 경우가 많다. 그러면 가독성이 떨어져서 함수의 이름만으로 그 용도를 설명하기 어려워진다. 일반적으로 개발자들은 그 함수를 감싸는 파사드 함수를 정의하여 인자를 줄이곤 한다. 커리함수는 그 작업을 보다 간단하게 해준다.   커리함수는 위와 같이 인자를 줄임과 동시에 함수의 명칭을 우리가 이해하기 쉽게 재정의해줌으로써 '인자가 많아질수록 가독성이 떨어지는'  함수의 단점을 보완할 수 있다. 

>  “This can enable classes to be selectively ignorant about certain aspects of how a function should be used, and reduce coupling between collaborating objects.”



**모나드를 만드는 데도 유용하다.**

```swift
let poleA = landLeft(2, landLeft(1, landRight(2, landRight(2, landLeft(3, (0,0))))))
```

위와같이 함수가 중첩되어 가독성이 악화되는 코드는 모나드를 이용하면 아래와 같이 직렬화 할수 있다. 

```swift
let poleA = (0, 0) >>= landLeft(3) >>= landRight(2) >>= landRight(2) >>= landLeft(1) >>= landLeft(2)
```

https://gist.github.com/monadis/12f50b36fae32a065794





## [Pattern Matching](https://gist.github.com/monadis/e6c7e660b786283e8534de28d5a38343)

## [Type Matching 예문](pastebin.com/MBX1YxZA)





## 대량의 검색결과 처리 패턴

검색결과를 반환하는 클래스를 작성한다고 하자. 검색결과가 적을 경우는 하나의 메소드로 구현이 가능하지만 많을 경우 메모리제한 때문에 이를 한꺼번에 리턴하는건 불가능해진다.

따라서 이런 경우는 검색결과를 바로 반환하기 보다는 검색결과를 묶음단위로 반환해주는 Generator를 만들어주는 방식이 좋다.  이 제네레이터는 iterator와 유사한 패턴으로 hasMoreItems()와 getNextBatch() 라는메소드를 가지고 있다. 

이같은 패턴은 관심사항을 분리시켜서 검색기능과 대용량처리기능이 분할되는 것이다.

얼핏 생각할때는 상속으로 기능을 확장해주는게 좋지 않은가 생각하기 쉽다. 하지만 상속을 사용하면 다른 목적을 가진 메소드들의 인터페이스가 섞여서 혼동을 가져올 우려가 있다.