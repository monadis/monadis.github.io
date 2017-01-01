## Lazy Cached Computed Property



Objective-C에서는 다음과 같은 코드를 패턴으로 사용했다.

```objective-c
- (Whatever *)instance {
    if (_ivar == nil) {
        _ivar = [[Whatever alloc] init];
    }
    return _ivar;
}
```

이런 패턴은 lazy loading으로 알려져 있다.

하지만 이 패턴은 단순히 늦은 초기화를 위한 패턴이 아니다. computed property로 사용할수 있으며 또한 결과값을 캐싱한후에 나중에 재초기화도 가능하다. lazy, cache, compute 의 특성을 모두 갖고있는 것이다.

이를 LCC pattern이라고 하자.

Swift에서는 이를 lazy property로 구현하고 있다.물론 간편하지만 Objective-C의 것과는 달리 lazy loading의 기능만을 구현하고있다는 점이 에러다. 결과값을 다시 계산하는것이 불가능하다.

이는 computed property와도 다르다. 캐싱이 되기 때문이다. 굳이 표현하자면 backing store가 있는 computed property인 것이다.

물론 Swift에도 동일한 코드를 만들수 있다.

```swift
var instance : Whatever! {
  if _ivar == nil {
    _ivar = Whatever()
  }
  return _ivar
}
```

하지만 이러한 패턴은 상당히 자주 쓰이므로 많이 사용하면 코드가 ugly해진다.



그래서 아래와 같은 Wrapper를 사용하면 이러한 코드를 단순화시킬수 있다.

```swift

struct Cached<T> {
	private var _cache:T!
	var cache: T! {
		mutating get {
			if _cache == nil {
				_cache = constructor()
			}
			return _cache
		}
		set {
			_cache = newValue
		}
	}
	private let constructor: () -> T
	init(_ constructor: () -> T) {
		self.constructor = constructor
	}

	mutating func clear() { _cache = nil }
}

class Test {
	var a = "A" {
		didSet {
			if oldValue != a {
				cachedString.clear()
			}
		}
	}
	var b = "B"
	var cachedString : Cached<String>!

	init () {
		cachedString = Cached { self.a + self.b }
	}
}

var t = Test()
t.cachedString.cache
t.a = "C"
t.cachedString.cache

```



## 참고

http://swiftrien.blogspot.kr/2015/02/lazy-and-cache.html

