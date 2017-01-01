## Capture



아래의 코드는 35를 출력한다. 그 이유는 기본적으로 클로져는 참조를 캡쳐하기 때문이다.

```swift
var i = 25
var cl = {print(i)}
i = 35
cl()
```

반면에,  아래처럼 바꾸면 값을 캡쳐하므로 25가 출력된다.

```swift
var i = 25
var cl = {[i] in print(i)}
i = 35
cl()
```



클래스 인스턴스의 경우도 마찬가지다. 아래의 코드는 "UpdatedValue"를 출력한다.

```swift
class MyClass {
	var value = "InitialValue"
}

var instance = MyClass() // original assignment
var cl2 = {print(instance.value)}
instance = MyClass() // updated assignment
instance.value = "UpdatedValue"
cl2()
```

코드를 `var cl2 = {[instance] in print(instance.value)}` 로 수정하면

"InitailValue"를 출력한다.



## Strong Reference Cycle

발견하는 방법

http://stackoverflow.com/a/32262299





## unowned, weak

