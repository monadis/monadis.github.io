## Swift Syntax


[Apple Swift Guide Site](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/GuidedTour.html#//apple_ref/doc/uid/TP40014097-CH2-ID1)

#### [String and Characters](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/StringsAndCharacters.html#//apple_ref/doc/uid/TP40014097-CH7-ID295)

#### [Upcasting and Downcasting](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TypeCasting.html)
> 업캐스팅 및 성공이 보장된 형변환은 as로 하고, 다운캐스팅은  as!  또는 as?로 한다. 

#### [if let 문법](https://gist.github.com/monadis/f8e3b9d3bb7fd16161e10666aa205fec)


#### Typealias
Swift에서는 typedef 대신 typealias 키워드를 사용합니다. 이미 존재하는 타입을 이용해서 새로운 타입을 만들수 있다. typealias를 이용하자.
```swift
typealias AudioSample = UInt16        // UInt16 보다 더 직관적이고 버그도 방지된다.
```
typealias를 이용해서 tuple타입에 이름을 정해줄수 있다. 
```swift
typealias Point = (Int, Int)
let origin: Point = (0, 0)
```


#### 타입의 이해 
왜 부울타입에는 true, false를 입력할수 있을까? 왜 조건문에 부울타입이 들어갈수 있을까? 사용자 정의 타입을 만들려면 어떻게 해야 할까? 이같은 의문의 답을 알고자 한다면 아래의 링크를 보자.
https://developer.apple.com/swift/blog/?id=8

### 함수 

```swift
func aMethod(str1:String? = nil, _ str2:String? = nil) { }
```
 스위프트에서는 디폴트 값이 있는 인수는 생략이 가능. 따라서 위 메소드는 아래 세 가지 방법으로 호출 가능.
```swift
aMethod("abc", "def")      // str1 = "abc", str2 = "def"
aMethod("abc")             // str1 = "abc, str2 = nil
aMethod()                  // str1 = nil, str2 = nil
```

 첫번째 파라미터에서 외부파라미터를 생략하지않고자 한다면 둘 다 동일하게 써주어야 한다.
```swift
func aMehtod(str str:String) { }
```
함수의 파라미터는 기본적으로 let형이다. 즉, 변형불가다. 만약 변형하고 자 한다면 인수명 앞에 var를 붙여야 한다.
```swift
func aMehtod(var str:String) { }
```



#### 함수 타입
두개의 정수를 파라미터로 받고 하나의 정수를 리턴하는 함수의 함수타입은 (Int, Int) -> Int
파라미터가 없고 리턴값도 없는 함수의 함수타입은 () -> ()  

함수타입 덕분에 함수 자체를 프로퍼티화 할수 있다.
```swift
func addTwoInts(a: Int, b: Int) -> Int {
    return a + b
}
var mathFunction: (Int, Int) -> Int = addTwoInts
```
위는 addTwoInts라는 함수를 mathFunction이라는 함수타입 변수에 할당한것.


#### C언어 API와의 호환
스위프트는 C언어 API와도 긴밀하게 호환이 된다. 
immutable array는 const pointer로 바로 argument로 전달하면 되지만
mutable array의 경우 `&`연산자를 써서 전달하면 된다.
또한 `const char *`의 경우 String과 호환이 된다.
하지만 이러한 변환은 일반적은 다른 스위프트 코드에 비해 unsafe하므로 주의가 필요하다.
https://developer.apple.com/swift/blog/?id=6

#### 가변 파라미터
가변파라미터로 받아진 변수들은 해당 변수타입의 배열로 취급된다.
함수 하나당 가변파라미터는 최대 한개만 가능하며 가장 마지막 파라미터가 되어야 한다.(디폴트값이 있는 파라미터보다도 뒤쪽에 위치) 이는 파라미터들간에 혼동을 방지하기 위함이다.

#### repeat while 문
do-while문의 do 가 error handling에 쓰이게 되었으므로 대신에 repeat이라는 키워드를 사용한다.
```swift
var a :Int = 1
repeat {
	a++
} while (a < 10)
```

[Guard 와 Defer 문법](nshipster.com/guard-and-defer/) 
(delay가 아니다. defer!!)

#### guard
https://gist.github.com/monadis/524c74a3a52c2811e3a47f4482004ff3

가드문의 else블록에 들어가는 코드는 return, continue, break와 같이 해당 스코프를 빠져나가는 구문이있어야 한다. fatalError()와 같은 @noreturn함수도 가능하다.
꼭 함수의 첫부분에만 쓰이는건 아니다. 아래와 같이 루프문 내에서도 쓸수 있다.
```swift
for imageName in imageNamesList {
    guard let image = UIImage(named: imageName) 
        else { continue }
    // do something with image
}
```

#### defer
guard문의 등장으로 인해 early return 방식의 코딩이 대세가 되었다. 하지만 이 방식의 단점은 그 끝처리에 있다. 이 방식에서는 return포인트가 여러군데 발생하기 때문에 리턴되기 직전에 처리해야 하는 작업이 있을경우 그 동일한 작업을 모든 리턴포인트에서 해줘야 하는 것이다. defer는 마치 completion block처럼 코드블록을 리턴직후로 밀어서 처리할수 있게 해주므로 이문제를 해결할수 있다.
**여러개의 defer 블록을 쓴다면 각 블록들은 코드상 순서의 역순으로 실행된다는점을 유의하자. 또한 코드  가독성을 해칠수 있으니 남용하지말자. **



### AssociatedType & Generics


#### NSInvocation 및 NSMethodSignature 의 대안
Swift에서는 NSMethodSignature 나 NSInvocation 같은 클래스를 사용하지 않는데 대신에 연관타입을 이용한다.

<<Request.playground>>

아래의 코드 구조는 외워두는게 좋다. 
https://developer.apple.com/swift/blog/?id=19

https://gist.github.com/monadis/36dc47b68746815f9be95bcc22c94554

#### 연관타입과 제네릭스의 차이
http://stackoverflow.com/questions/26554987/why-dont-associated-types-for-protocols-use-generic-type-syntax-in-swift
연관타입이 훨씬 유연하다. Java 의 Scala언어로부터 차용됨. 
예들들어 ```Array<String, Int, Generator<String>>``` 는 ``` Array<String, Int, Generator<String>>``` 끼리만 호환이 되지만 연관타입을 이용하면 내부적으로 타입이 달라도 호환이 가능.



#### Reflection, Introspection

Reflection comes from the idea of "self-examination, self-modification, and self-replication", reflecting on one's self for the purpose of change.
리플렉션은 코드가 그 자체를 인식하여 스스로를 수정하는 방식이다. 자기인식, 자기변형, 자기복제를 가능케하는 자기진화적 코드 패턴이다. 인트로스펙션은 리플렉션의 한가지 요소인 self-examination 을 의미한다. 보통은 introspection과 객체지향의 다형성을 이용하면 자기변형과 복제도 가능하여 리플렉션을 구현할수있다. 하지만 그 자유도에는 한계가 있다.
클래스 이름만 전달해도 그 이름으로 클래스의 인스턴스를 생성할수 있도록 하고자 한다면 어떨까? 하지만 문자열은 오타의 가능성이 존재하므로 불안전하다. 그래서 나온것이 Meta type이다.

####Meta Type

SomeSubClass.self는 SomeSubClass의 메타타입을 가진 객체를 의미한다. 메타타입은 타입 자체를 변수에 저장할수 있다. 즉, 이덕분에 동적 타입의 객체를 생성하는것도 가능해진다. 외부에서 객체를 받아서 그 객체와 동일한 타입의 객체를 생성해서 리턴해줄수도 있다.
 이때, 메타타입을 전달하려면 메타타입 자체도 변수에 넣어야하는데 그렇다면 그 변수의 타입은 무엇일까? SomeSubClass.Type이다.
클래스에서만 메타타입객체를 얻어낼수 있는건 아니다. 인스턴스에서도 someInstance.dynamicType 의 문법으로 메타타입 객체를 얻어낼수 있다.
```swift
		class SomeBaseClass {
		    class func printClassName() {
		        print("SomeBaseClass")
		    }
		}
		class SomeSubClass: SomeBaseClass {
		    override class func printClassName() {
		        print("SomeSubClass")
		    }
		}
		let someInstance: SomeBaseClass = SomeSubClass()
		// The compile-time type of someInstance is SomeBaseClass,
		// and the runtime type of someInstance is SomeBaseClass
		someInstance.dynamicType.printClassName()
		// prints “SomeSubClass"


		// 어떤 인스턴스의 runtime 타입이 compile-time의 타입과 동일한지 알려면 
		//  === 혹은 ==! 연산자를 사용하면된다.
		if someInstance.dynamicType === someInstance.self {
		    print("The dynamic type of someInstance is SomeBaseCass")
		} else {
		    print("The dynamic type of someInstance isnt SomeBaseClass")
		}
		// prints "The dynamic type of someInstance isn't SomeBaseClass”

		// required init 생성자 메소드를 이용하면 메타타입을 통한 객체생성도 가능하다.
		
		class AnotherSubClass: SomeBaseClass {
		    let string: String 
		    required init(string: String) {
		        self.string = string
		    }
		    override class func printClassName() {
		        print("AnotherSubClass")
		    }
		}
		let metatype: AnotherSubClass.Type = AnotherSubClass.self
		let anotherInstance = metatype.init(string: "some string")
```
#### dump()
dump uses reflection recursively to print out an instance’s children, their children, and so on

```swift
dump(CGRect.zeroRect)
// ▿ (0.0, 0.0, 0.0, 0.0)
//   ▿ origin: (0.0, 0.0)
//     - x: 0.0
//     - y: 0.0
//   ▿ size: (0.0, 0.0)
//     - width: 0.0
//     - height: 0.0
```


#### Mirror Type
https://gist.github.com/monadis/e5a3b26c5b5310b30c581be5d7ba3135?ts=4
위와같은 dump()함수가 다양한 객체를 받아 유사하게 작동할수 있는것는 미러타입 때문이다.	

#### ~~@noescape~~ 
~~해당 함수의 Lifetime보다 더 오래 살 수 없는 클로져블록.~~ 
~~기존의 클로져는 함수 자체보다 더 오래 살아남아서 의도치않은 작업을 하기도 하였음. 단순히 코드의 일부를 외부에서 받고자 클로저를 사용하는 경우에는 이런 부작용을 사전에 차단하기위한 장치가 필요함.~~ 
~~noescape클로져는 async디스패치로 전달이 불가하며 변수로 저장도 불가함. 내부에 self키워드를 쓸필요가 없어짐.~~

~~http://stackoverflow.com/a/28428521/2047287~~

#### @escaping
해당 함수의 Lifetime보다 더 오래 살 수 있는 클로져블록. 



#### Selector

`#selector(FetchTasklistsOP.didFinishWithTicket(_:receivedData:error:))`

#### Lazy Loading
계산 시간이 오래 걸리는 경우 사용하거나, 인스턴스 생성 시점에는 계산에 필요한 정보를 아직 다 얻을 수 없는 경우 사용한다.
늦은 초기화를 위해 **지역변수 선언부 앞에 lazy를 사용.** (글로벌 상수 및 변수를 초기화하는 initializer는 lazy initialize되며, dispatch_once가 디폴트로 적용되어 atomic이다.  즉 lazy 키워드를 생략한다.)

이때 **변수는 항상 var 타입**이어야 함. 왜냐하면 let타입은 초기화 전에 값이 정해지는 상수이기 때문이다. lazy변수는 이 변수에 접근하여 사용하기 전까지는 변수의 초기화가 이뤄지지 않는다.

https://developer.apple.com/swift/blog/?id=7
 >어떤 구조(function, class등)에도 속하지 않은 코드를 top-level code라고 한다.
 >playground에서는 어떻게 이런 코드가 실행될까? 기본적으로 swift는 order-independent하다. 즉, import문이 파일 끝부분에 있어도 해당 코드를 실행하는데 아무 문제가 없다. 반면에 Playground는 order-dependent하다. swift가 order-independent하다면  프로그램의 진입점은 어디가 되어야 할지가 문제가된다. OSX프로그래밍에서 main.swift 파일은 플레이그라운드와 같이 order-dependent하다.  이곳에 탑레벨코드가 존재하고 여기가 프로그램의 진입점이 된다.그러나 iOS프로젝트에서는 이 파일을 볼수가 없는데 대신 @UIApplicationMain 지시자가 존재하는 임의의 swift파일이 진입점이 되도록 자동으로 main.swift파일을 구생성 및 구성해준다.
 >
 >var someGlobal = foo()
 >
 >그렇다면 위같은 글로벌 변수는 언제 초기화가 될까?  
 >단일파일로된 프로그램이라면 대답은 간단하다. 탑다운으로 실행시키면 되므로. 하지만 파일이 많다면?
 >언어에 따라 세 가지 방식의 초기화 방식이 있다.

>1. C언어: 단순한 상수형 초기화만 용납함. 싱글톤이나 딕셔너리 같은 복잡한 구조는 초기화가 안되니 너무 제한적이라 별로.
>2. C++ : static constructor 라는 것을 도입. 좀 복잡한 초기화도 가능해짐. 앱이 로드할때 각 파일들에 존재하는 글로벌 변수들이 먼저 초기화됨. 하지만 이 방식은 대형 앱일수록 로딩시간이 길어짐. 파일이 많아질수록 각 파일에 존재하는 글로벌 변수가 초기화되는 순서가 달라지는 경우의 수가 많아져서 예측이 힘듬.
>3. 자바: lazy initializer. 최초로 사용할때 비로소 초기화됨. 

>swift는 3번 방식을 채택하였다. 늦은 초기화.



#### Lazy Caching
http://stackoverflow.com/a/40847994/2047287
```swift
lazy var someVar : Int! = {  }()
```
위처럼 타입 뒤에 `!`마크를 붙이고 nil을 대입한 후 재사용할때 재초기화가 이뤄진다.
마치 ObjC의 늦은초기화 프로퍼티처럼 쓸수있는것이다.
만약 재초기화를 원하지 않고 nil을 넣을수 있게 하고자 한다면 `?`를 사용하면 된다.

#### Lazy Sequence

배열은 함수와 매핑해서 사용하면 각 멤버들에게 함수를 적용시킬수 있다. 근데 모든 멤버에 즉각 함수를 적용시키는것은 불필요한 계산을 야기할수 있다. 현재 접근하고자 하는 멤버들에게만 계산하는것이 좋을것이다. 그래서 lazy 함수를 사용한다.

```swift
func increment(x: Int) -> Int {
  print("Computing next value of " + \(x))
  return x+1
}

let array = Array(0..<1000)
let incArray = array.lazy.map(increment)  // lazy를 없애고 실행하여 비교해보자. 
print("Result:")
print(incArray[0], incArray[4])

```

아래처럼 연속으로 사용해도 lazy의 효과는 유지된다.

```swift
let doubleArray = array.lazy.map(increment).map(double)
```



#### static methods / static properties

“static” methods and properties are now allowed in classes (as an alias for “class final”).
즉, static은 오버라이딩이 불가능하다는 점을 제외하고는 class와 같은 의미다.
Objective-C에서 썼던 static method variable은 Swift에서는 존재하지 않는다. 대신 메소드 외부(클래스 내부)에 선언하면 된다.

### Singleton
In Swift, you can simply use a static type property, which is guaranteed to be lazily initialized only once, even when accessed across multiple threads simultaneously:
```swift
final class Singleton {
	static let shared = Singleton()
	private init() {} //This prevents others from using the default '()' initializer for this class.
}
```
If you need to perform additional setup beyond initialization, you can assign the result of the invocation of a closure to the global constant:
```swift
final class Singleton {
    static let shared:Singleton  = {
        let instance = Singleton()
        // setup code
        return instance
    }()
    private init() {} //This prevents others from using the default '()' initializer for this class.
}
```
https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/AdoptingCocoaDesignPatterns.html

> [구조체를 쓰지 않는 이유: 클래스 대신 구조체를 이용하여 싱글톤을 구현하면 값을 바꿀수 없게 된다.](http://stackoverflow.com/a/36788519/2047287) 



### Closure

매개변수 이름을 사용하지 않고, 인자로 넘어오는 순서대로 ```$0, $1, ...```를 사용하면 ```in```과 그 앞의 매개변수 목록도 없앨 수 있다.

```swift
let reversed = ["a", "b"].sort({ $0 > $1 })
```

이렇게 두 인자를 받아서 연산자(>)에 넘기는 경우, > 자체가 저 클로저의 타입 ``` (String, String) -> Bool ```과 타입이 같다. 이런 경우 연산자를 바로 넘겨도 된다.

```swift
let reversed2 = ["a", "b"].sort(>)
```



### Property

기본적으로 프로퍼티는 아래와 같은 형태다.
```swift
private var _document: UIDocument? 
var document: UIDocument? {
	get {        
		return _document
    }
    set {
	// 1
        _document = newValue
	//2
    }
}
```
하지만 이러한 코딩은 매우 귀찮으므로  축약을 하게된다.
위의 코드는 다음과같이 줄일수 있다.
```swift
// stored property
var document: UIDocument?
```
1 혹은 2번 위치에 코드를 넣고 싶다면 willSet이나 didSet을 쓰면 된다. 
willSet에서는 newValue가 디폴트 파라미터인 반면,
didSet에서는 oldValue가 디폴트 파라미터라는점을 주의하자. 
이때, willSet, didSet을 프로퍼티 옵저버라고 한다.
```swift
// stored property
var document: UIDocument? {
	didSet {
    	useDocument()
	}
}
```
ivar가 필요없는 computed 변수는 ivar를 지우고  `computed property`라고 한다.  오버라이드가 아닌 computed property는 `willSet, didSet`이 필요없다. 그냥 get set 블록 내에 코드를 직접 넣으면 되기 때문이다.
```swift
   var perimeter: Double {
         get {
             return 3.0 * sideLength
         }
         set {
             sideLength = newValue / 3.0
         }
    }
```
여기서 `newValue`는 새값을 받는 인자의 묵시적인 이름이다. 만약 이 이름이 싫다면 
`set(명시적 이름) { … }` 
형태로 작성해주면 된다.

lazy loading은 아래와 같이 구현할수도 있지만 
```swift
private var _value : Int?
var value: Int {
	get {
		if _value == nil {
			_value = 초기값
		}
		return _value!
	}
}
```
lazy 키워드로 축약이 가능하다.
```swift
lazy var value:Int = { return 초기값 }() 
```

위와 같은 축약된 패턴들로 구현이 불가능한 경우에는 맨 위의 원본 코드를 사용하면된다.
예를들어 lazy loading은 초기화 코드가 단 한번만 실행된다.(objective-c에서는 nil이 될때마다 실행) 따라서 Objective-C의 방식을 원할경우 직접구현해줘야 한다.

```swift
class Foo {
	var propertyWillChange : (Int) -> Void = {
		print("The storedProperty will change to \($0)")
	}
	var propertyDidChange : (Int, Int) -> Void = {
		print("The value of storedProperty has changed from \($0) to \($1)")
	}
	// stored property에는 디폴트값을 설정할수 있지만 computed property는 불가능
	var storedProperty : Int = 0 {        
		willSet { propertyWillChange(newValue)}
        didSet { propertyDidChange(oldValue, self.storedProperty) }
	}
    // 오버라이드가 아닌 computed property는 willSet, didSet이 필요없다. 그냥 get set 블록 내에 코드를 직접 넣으면 되기 때문이다.
	var computedProperty : Int {
		get {
			return 1
		}        
		set {
			// set에 괄호가 없으면 newValue가 디폴트 파라미터가 된다.
			doSomething(newValue)
		}    
	}
	func doSomething(i: Int) {
      
	}
}
let a = Foo()
a.storedProperty = 3
```

#### property override
super를 활용한다.  setter를 구현했다면 getter역시 구현해야 한다.
```swift
override var 속도: Double  {
  get {
    return super.속도
  }
  set {
   super.속도 = min(newValue, 40.0)
  }
}
```


#### dynamic modifier
```swift
class C {
	dynamic var someText: String = "" {
		didSet {
			print("changed to \(someText)")
		}
	}
}
```
Apply this modifier to any member of a class that can be represented by Objective-C. When you mark a member declaration with the dynamic modifier, access to that member is always dynamically dispatched using the Objective-C runtime. Access to that member is never inlined or devirtualized by the compiler.
Because declarations marked with the dynamic modifier are dispatched using the Objective-C runtime, they’re implicitly marked with the objc attribute.



#### Property Observers

참고 : <a href="./Observer Patterns.md">Observer Patterns</a>

- lazy 가 아닌 어떤 stored property에라도 옵저버를 달아놓을수 있다.
- 상속받은 프로퍼티에도 오버라이딩을 통해 옵저버를 달 수 있다.
- computed property에는 옵저버를 달 필요가 없다. 왜냐하면 그냥 세터 내부에서 처리하면 되므로.
- 처음 초기화시에는 willSet과 didSet이 작동하지 않는다. 외부에서 값을 바꾸려할때만 작동. 마찬가지로 옵저버 내부에서 값을 변경해도 옵저버가 호출되지 않는다.
- newValue, oldValue라는 디폴트 변수를 이용할수 있다.
- 기존의값과 새값이 똑같다 하더라도 willSet, didSet이 호출된다.
- 전역변수와 지역변수에도 옵저버 적용 가능하다.
- 서브클래스에서 세터와 옵저버를 동시에 오버라이드 할 수 없다.







#### Closure를 이용한 초기화

우측의 () 괄호를 주목하자. 이 때문에 해당 클로져는 바로 실행되어 리턴값이 someProperty에 저장된다. 만약 괄호가 없다면 해당 프로퍼티에는 클로져 자체가 들어가게된다. 

혼동하기 쉬운 세가지 문법을 비교해보자.

```swift
var someProperty2 = { return 2 }()	// 클로져를 이용한 값초기화
var someProperty1:Int { return 2 }  // read-only computed property
var someProperty3 = { return 2 }	// 클로져 자체를 저장하는 프로퍼티
```

 

### Type Property

 swift에서는 타입이 공용으로 소유하는 프로퍼티를 타입프로퍼티라고 부른다.   static 키워드를 사용한다. 

### Type Method

타입이 공동으로 소유하는 메소드. static 혹은 class 키워드를 붙인다. 후자의 경우 서브클래싱이 가능하다.

 타입메소드 내에서 또다른 타입메소드에 접근할때는 닷연산자가 필요없이 바로 호출이 가능하다.

타입메서드 내부에서 쓰이는 self는 그 타입 자신을 가리킨다.



###module

 코드 배포 단위. 타겟, 프레임웍, 앱번들, 어플리케이션 등이 모두 모듈로 볼수 있다. 하나의 모듈은 다른 모듈에 import될수 있다. 모듈은 객체들의 네임스페이스 역할을 한다. 따라서 표준프레임워크 내의 클래스와 동일한 이름을 사용해도 이름 충돌이 발생하지 않는다. 따라서 클래스 prefix도 더이상 필요없다. 동일한 클래스명을 구분해야 하는경우 앞에 모듈명을 붙여주면 된다. (Swift.Array 또는 MyProject.Array 와 같이)

###Access level

각각의 접근 한정자에 대해 간략히 알아보자(가장 개방된 것부터 가장 제한적인 순서로).

**open**

가장 개방된 접근 한정자로써 소속 모듈 또는 소속 모듈을 import하는 모든 모듈에서 class와 class 멤버에 접근할 수 있으며 open class를 상속 받아 sub class를 생성하거나 메서드를 override 할 수 있다. 간단히 이야기 하자면 다른 언어에서의 public과 유사하다.

**public**

open 과 동일한 접근을 허용하지만 sub class 생성과 override에 제한이 있다. 소속 모듈 내에서는 sub class 생성과 sub class 내에서의 override가 허용된다. 이 제한은 프레임워크(모듈)을 제작하는 경우 유용하다. 프레임워크 내에서는 자유롭게 상속 받지만 외부에서는 상속을 받을 수 없기 때문에 확장을 제한할 수 있다.

**internal**

접근 한정자가 지정되지 않은 경우 기본적으로 사용되는 접근 수준이다. 소속 모듈의 모든 소스 파일에서 사용할 수 있지만 모듈 외부에서는 접근 할 수 없다.

**fileprivate**

소속 소스 파일 내에서만 접근이 가능하다.

**private**

현재 소스를 둘러싸는 선언으로 내에서만 접근 가능하다.

Swift 3에서는 위에서 열거한 5가지 레벨로 접근을 제한하도록 되어 있다. 단, Objective-C 클래스와 메소드는 이제 **open** 상태로 가져온다.

------

여기까지만 보면 다른 언어들에 비해 너무 복잡하다는 생각이 들 수 있다. 하지만 어플리케이션은 하나의 모듈과 동일하게 생각하면 되므로 어플리케이션 내에서 작성된 코드는 기본적으로 모든 어플리케이션 소스 내에서 접근 가능하다. 따라서 프레임워크(모듈)를 제작하는 것이 아니라면 **fileprivate** 과 **private** 를 이용해서 접근을 제한하는 경우만 고려하면 된다.

[objective-c에 존재하던 protected는 사라졌다.](https://developer.apple.com/swift/blog/?id=11)

구성요소의 접근레벨은 그것들이 모여 이루는 전체의 접근레벨보다 크거나 같아야 한다. 예를들어 파라미터의 접근레벨이 internal인 함수는 public이 될수 없다. private인 클래스 타입의 변수는 internal이 될수 없다. tuple도 마찬가지의 룰이 적용됨.

결론: 일반적인 개발에서는 **private**만 제대로 사용하면된다.



**Setter에게 Getter 보다 낮은 접근 제한자를 지정하여 쓰기 접근을 제한 할 수 있다.**

```swift
private(set) var someFlag = false
fileprivate(set) var someFlag = false
```



**@testable**

단위 테스트 대상은 자체 모듈이므로 기본적으로 **internal** 인 Application 모듈의 모든 메서드 또는 변수에 접근할 수 없다. 이런 경우는 `@testable` 속성을 사용해서 import 한다.

```swift
import XCTest
@testable import MyDataSource

class MyDataSourceTests: XCTestCase {
  // func testSomething() {...}
}
```




### Loop



#### label

라벨 붙은 문장: 문장 앞에 라벨을 붙이고, continue, break에서 라벨을 지정해 그리로 이동 가능

라벨은 중첩된 중괄호 내부에서 흐름을 관리할때 유용하다.  

아래의 break는 switch문과 while문의 중괄호를 한번에 빠져나가야 하므로 라벨이 필요하다.

```swift
let finalSquare = 25
var board = Int
board[03] = +08; board[06] = +11; board[09] = +09; board[10] = +02
board[14] = -10; board[19] = -11; board[22] = -02; board[24] = -08
var square = 0
var diceRoll = 0
print(board)
// 아래 gameLoop: 부분이 라벨임
gameLoop: while square != finalSquare {
  if ++diceRoll == 7 { diceRoll = 1 }
  switch square + diceRoll {
  case finalSquare:
    // diceRoll을 더하니 마지막 칸으로 이동함. 끝!
    break gameLoop
  case let newSquare where newSquare > finalSquare:
    // diceRoll을 더하니 마지막 칸보다 더 뒤로 가버림. 다시던지기
    continue
  default:

    // 정상 이동. 보드 업데이트.
    print("current square : (square)   diceRoll : (diceRoll)")
    square += diceRoll
    square += board[square]
  }
}
print("Game over!")
```

#### fallthrough

switch case 문의 흐름제어는 break, continue 외에도 fallthrough라는 것이 있다.

fallthrough: case에서 다음 case로 계속 실행을 이어나가고 싶을 때 이를 명시함.

(주의: fallthrough로 제어가 넘어갈때는 다음번 case문을 체크하지 않고 그 안으로 실행이 넘어간다. 즉, C switch/case에서의 동작과 같다)





### 생성자(Initializer)

**디폴트 생성자 **

구조체 혹은 base class에만 해당. 상속을 받은 클래스의 경우는 디폴트 생성자가 생기지 않음.

모든 멤버 프로퍼티가 초기값이 제공되어있는(혹은 optional인) 클래스 또는 구조체가 아무런 initializer가 없다면 스위프트는 자동으로 디폴트 생성자를 마련해준다.

**멤버와이즈 생성자 **

구조체에만 적용됨. 각 멤버변수들을 초기화시킬수 있는 생성자가 자동으로 마련됨.

 

커스텀 생성자가 존재하는 클래스나 구조체에는 디폴트 생성자는 자동생성이 되지 않는다. 커스텀 생성자를 이용해서 생성해야 하는 객체인데 실수로 디폴트 생성자를 사용하면 안되기 때문이다.

만약 커스텀 생성자와 디폴트 생성자를 함께 쓰고싶으면 extension을 사용하자. 커스텀 생성자는 extension내에 구현하자. 

 

**생성자 체인 규칙 **

클래스를 여러가지 용도로 재사용하기 위해서는 생성자가 다양해질수밖에 없고, 생성자들이 서로를 호출함으로써 복잡하게 얽히게되는데 이로인해 각종 오류가 유발될수 있다. 이를 방지하기 위해서는 생성자를 다루는 룰이 필요하다.

생성자는 designated initializer(DI)와 convenience initializer(CI)로 나뉜다. DI는 이 클래스 객체를 생성할때 꼭 거쳐야 하는 통로다.CI는 DI를 쓸때의 복잡한 생성과정을 단축시키기 위하여 마련된 편의성 생성자다. 앞에 convenience키워드가 붙지 않은 것은 전부 DI로 보면 된다.

1. DI는 바로위의 superclass의 designated생성자를 호출한다. (물론 base class인 경우는 예외.)
2. CI는 같은 클래스 내의 다른 생성자를 호출한다.
3. CI는 결과적으로는 DI를 호출하게 되어있다.

![생성자 체인 규칙](./images/생성자 체인 규칙.jpg)

>  위의 도식은 위임관계를 말하는것이지 상속(override) 관계가 아니다. 착각하지말자. subclass의 CI가 superclass의 DI를 override해도 상관이 없다.

위의 그림과 같이 DI는 항상 위로 위임하고, CI는 항상 옆으로 위임한다. 다시말하면 서브클래스의 DI에서는 항상 super.init(…)이 호출된다. CI에서는 항상 self.init(…)이 호출된다. 

init내에서는 먼저 프로퍼티를 초기화한 후, 다른 메소드를 사용해야 한다. 그 이유는 먼저 초기화를 마쳐야만 self가 사용가능해지고 self.를 통해 다른 메소드에 접근할수 있기 때문이다. super.init을 호출하는 것 역시 프로퍼티 초기화가 된 이후에 해야 한다. 왜냐하면 super.init내에서 다른 메소드를 호출하는 경우, 서브클래스가 오버라이드 한 버전의 메소드를 호출하도록 하기 위함이다. 초기화가 되지 않은 상태에서는 서브클래스가 생성되지 않으며 따라서 오버라이드도 안되기 때문이다. 

>  WWDC intermediate swift 동영상 initialization부분 참고

컴파일러는 init에 대해 다음의 안전체크를 한다.

- designated생성자는 superclass의 생성자에게로 딜리게이트 하기 전에 모든 그 클래스의 프로퍼티를 초기화해야 한다.
- 상속받은 프로퍼티를 커스터마이징 하기전에 superclass의 생성자에게 먼저 딜리게이트가 이뤄져야 한다. 커스터마이징 이후에 딜리게이트가 이뤄지게되면 커스터마이징한 값들이 딜리게이트 과정중에 덮어씌워지기 때문이다.
- convenience생성자는 우선 다른 생성자에게 딜리게이트를 진행후 자신의 작업을 실행해야 한다. 마찬가지로 덮어씌워짐을 방지하기 위해.
- 생성자는 첫번째 생성단계가 끝날때까지는 다른 인스턴스 메소드를 실행하지 않으며, 프로퍼티값을 읽지 않으며, self를 참조하지도 않아야 한다. 

생성자는 크게 세 부분으로 이뤄진다고 생각하면 이해가 쉽다. 

1. 고유값 초기화 : 서브클래스에만 있는 프로퍼티를 초기화함.
2. 딜리게이트 : 수퍼클래스의 프로퍼티를 초기화하도록 위임함.
3. 상속값 커스터마이징 : 상속받은 프로퍼티의 값을 수정.

딜리게이트는 다시 그 내부에서 값초기화, 딜리게이트, 커스터마이징이 순서대로 진행되므로 이를 재귀적으로 반복하면  결국은 값초기화, 커스터마이징으로 순서가 결정된다. 

이는 생성자의 2단계 룰을 완벽하게 충족시킨다.



```swift
class Vehicle {
  var numberOfWheels = 0
  var description : String { // computed property
    return "(numberOfWheels) wheels"
  }
}

class Bicycle: Vehicle {
  // hasBasket 이 초기값이 없음에도 Optional이 아닌 이유는 init()에서 value initialize부분이 있기 때문이다.
  var hasBasket : Bool
  override init() {
    hasBasket = true  // value initialize
    super.init()  // delegate
    numberOfWheels = 2  // inherited property customize
  }
}
```



**생성자 상속 규칙 **

수퍼클래스의 생성자는 기본적으로 상속되지 않도록 되어있다. 수퍼클래스의 생성자가 서브클래스에서는 쓸모없는경우가 많기 때문이다. 하지만 그렇지않은 경우도 물론 있으므로 아래의 조건 하에서만 자동으로 상속을 해준다. 

- 서브클래스에 DI가 없을경우 수퍼클래스의 모든 DI를 자동상속시킨다.
- 서브클래스가 수퍼클래스의 모든 DI를 상속받았거나 구현했을경우 수퍼클래스의 모든 CI 역시 자동상속됨(단, 서브클래스에 구현되지 않은 것만) 
- 서브클래스의 프로퍼티에 전부 디폴트값이 있고, 생성자가 하나도 없다면 자동으로 DI, CI가 모두 상속된다

>  따라서 자동상속이 되지 않은 경우, 수퍼클래스의 CI와 동일한 생성자를 서브클래스에 정의해도 override 키워드는 사용할필요가 없다. 또, requred생성자일 경우에도 마찬가지로 override가 필요없다. 



**required 생성자 **

- 위의 자동상속 조건을 충족시키지 못했지만 일부 CI생성자는 상속받고싶은경우가 있을것이다. 그때는 required를 이용하자. 
- 반드시 서브클래스가 구현해야 하는 DI생성자 역시 required를 쓰면된다.

수퍼클래스의 DI를 오버라이딩할때는 항상 override 키워드를 쓴다. 즉, 서브클래스의 override키워드를 보고 수퍼클래스의 DI가 오버라이딩 되고 있다는것을 알수 있다.  (수퍼클래스의 CI는 서브클래스에서 오버라이딩이 불가하다.)

required 생성자를 오버라이드 할때는 override 키워드를 쓰지 않는다.


**failable initializer ** 

이전까지는 어떤 객체를 생성하다가 실패할 경우 이를 통지하려면 별도의 팩토리 메소드가 필요했다.

예를 들어 enum 형에는 fromRaw 메소드가 팩토리 메소드 역할을 했다. 하지만 이는 failable initializer로 대체되었다. 

```swift
enum Color : Int {
	case Red = 0, Green = 1, Blue = 2
	// implicitly synthesized
    var rawValue: Int { /* returns raw value for current case */ }
		// implicitly synthesized
    init?(rawValue: Int) {
      switch rawValue { 
      	case 0: self = .Red
        case 1: self = .Green
        case 2: self = .Blue
        default: return nil
      }
   }
}
```



### deinit

deinit 에서는 객체를 해제할때 해야 하는 일을 한다. 객체의 메모리 해제에 관련해서는 ARC가 대부분 해주지만 다른 작업들(파일을 닫는 작업 등)은 직접 해야 한다. 꼭 메모리해제 관련일만 해야 하는건 아니다. 

superclass의 deinit은 서브클래스의 deinit이 끝나는 시점에 자동으로 실행되므로 직접 호출할 필요없다.

  




### [Error Handling](https://gist.github.com/monadis/56051d57f5fe8c7faa7641930be9658a)





### Fast Enumerator

for-in 루프에 호환이 되는 NSFastEnumerator객체 구현

여기서 EnumerableClass는 Apple이 제공하는 샘플 클래스임.

```swift
struct MyGenerator: GeneratorType {
     let enumerator: NSEnumerator
     mutating func next() -> AnyObject? {
          return enumerator.nextObject()
     }
}

class MyEnumerableClass : EnumerableClass, SequenceType {
	func generate() -> MyGenerator {
          let mg = MyGenerator(enumerator: self.objectEnumerator())
          return mg
     }
}

let myEnumerable = MyEnumerableClass(capacity: 50)
for item in myEnumerable {
     print("(item)")
}

// 물론 아래와 같이 하는게 훨씬 편하다.
let enumerable = EnumerableClass(capacity: 50)
enumerable.enumerateObjectsUsingBlock { (obj, idx, stop) -> Void in
     print("(obj)")
}
```



###[Operator 및 연산자 오버로딩](https://gist.github.com/monadis/58161905dc3bd644512d593dbe456c0e)



## ⚠️유의사항

- 타입명이 맘에 안들면 typealias를 사용하자. 타입명이 직관적이지 않거나 쓸데없이 길거나 혼동이 오는 경우는 매우 많기 때문이다. 

