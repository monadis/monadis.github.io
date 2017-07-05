## KVC

아래 링크에 쉽게 설명되어 있다.
http://blog.dbinsight.net/2

KVC는 method calling 대신 message sending을 함으로써 두 객체간 런타임의 연결관계를 느슨하게 하기 위한 수단이다.
swift에서는 옵셔널이 이 기능을 대신한다.

## KVO

아래의 메소드들은 KVO 구현을 위한 메소드들이다. 뒤의 두개는 컬렉션용이다.

willChangeValueForKey() , didChangeValueForKey() , willChange(_:valuesAtIndexes:forKey:) didChange(_:valuesAtIndexes:forKey:)

하지만 오버라이드 하지말도록 규정되어있다.  willSet, didSet 내에 코딩하면 동일한 효과를 볼수 있기때문에 굳이 필요없다. 이를 사용하기 힘든 경우라면 observeValueForKeyPath() 메소드를 사용하자.

#### 알아두기
- 특정 프로퍼티의 변화를 감지한다는 것은 그 프로퍼티 자체가 변하는것을 감지하는 것이다. 따라서 그 프로퍼티 내부의 하부 프로퍼티가 변한다고 해도 옵저버는 감지하지 않는다.
- KVO를 쓰려면 Objective-C 클래스의 서브클래스 이거나 `dynamic` 키워드 사용한 Swift클래스 여야 한다.
- willSet, didSet도 비슷한 기능을 하긴 하지만 자신의 프로퍼티만 관찰할수 있는 한계가 있다.


#### 옵저버의 등록

먼저 컨텍스트를 만든다. 그 이유는 동일한 프로퍼티에 다른 이유로 접근하는 경우가 있을수 있기 때문에 구분시키기 위함이다.
```
static void *PersonAccountBalanceContext = &PersonAccountBalanceContext;
[account addObserver:self
              forKeyPath:@"balance"
                 options:(NSKeyValueObservingOptionNew |
                          NSKeyValueObservingOptionOld)
                 context:PersonAccountBalanceContext]; 
```

위 메소드는 관찰객체, 관찰대상객체, 컨텍스트 중 어느것도 strong retain을 하지않는다. 따라서 사용자 측에서 해당 객체들을 retain해야 한다.
이 말은 해당 객체들이 release되기 전에 반드시 옵저버를 해지시켜줘야 한다는 것을 의미한다. 그렇지 않으면 에러가 날 것이다. 보통은 init이나 viewDidLoad메소드 등에서 register하고 dealloc메소드에서 unregister한다. 예외처리 하는 곳에서 unregister하는 경우도 있다.
```
[account removeObserver:self
            forKeyPath:@"balance"
              context:PersonAccountBalanceContext];

```
콜백 메소드도 만들어야 한다.
```
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary *)change
                       context:(void *)context {

    if (context == PersonAccountBalanceContext) {
        // Do something with the balance…

    } else if (context == PersonAccountInterestRateContext) {
        // Do something with the interest rate…

    } else {
        // Any unrecognized context must belong to super
        [super observeValueForKeyPath:keyPath
                             ofObject:object
                               change:change
                               context:context];
    }
  }
```

#### KVO를 발동시키는 방법

KVO 는 KVC 에 기반한다. 따라서 메시지 보내는 방식으로 프로퍼티에 접근해야만 KVO가 작동한다.
아래는 KVO가 작동케 하는 메시지 송신의 예다.
```
// Call the accessor method.
[account setName:@"Savings"];

// Use setValue:forKey:.
[account setValue:@"Savings" forKey:@"name"];

// Use a key path, where 'account' is a kvc-compliant property of 'document'.
[document setValue:@"Savings" forKeyPath:@"account.name"];

// Use mutableArrayValueForKey: to retrieve a relationship proxy object.
Transaction *newTransaction = <#Create a new transaction for the account#>;
NSMutableArray *transactions = [account mutableArrayValueForKey:@"transactions"];
[transactions addObject:newTransaction];
```

#### 수동 KVO의 구현

다음처럼 프로퍼티를 직접 구현할수 있다. (값이 변할때만 작동하도록 해도 된다.)
```
- (void)setBalance:(double)theBalance {
    if (theBalance != _balance) {
        [self willChangeValueForKey:@"balance"];
        _balance = theBalance;
        [self didChangeValueForKey:@"balance"];
    }
  }
```
여러 값이 동시에 변한다면 아래와 같이 nesting구조로 작성하자.
```
- (void)setBalance:(double)theBalance {
    [self willChangeValueForKey:@"balance"];
    [self willChangeValueForKey:@"itemChanged"];
    _balance = theBalance;
    _itemChanged = _itemChanged+1;
    [self didChangeValueForKey:@"itemChanged"];
    [self didChangeValueForKey:@"balance"];
  }
```
자동 KVO는 
```
self.setValue("egf", forKey: "lastname")
```
이런 식으로 setValue 메소드를 이용해야만 작동한다. 하지만 수동 KVO는 직접  
```
self.lastname = "egf" 
```
라고 해도 작동한다.
**만약 수동 KVO가 적용된 프로퍼티인데도 setValue를 사용하여 값을 변경하면 노티피케이션이 두번 발생하는것을 확인할수 있다.**
수동 노티피케이션과 자동 노티피케이션이 둘 다 동작했기 때문이다. 자동 노티피케이션을 끄는 방법은 아래에서 설명하겠다.

to-many 릴레이션십에서는 불필요한 노티피케이션이 많이 발생할수 있다. 변경을 그룹단위로 통지해도 될것이다. 따라서 수동으로 구현해주는게 좋다.

ordered to-many 릴레이션십에 KVO를 적용하고자 한다면 다음과 같이 구현한다.
```
- (void)removeTransactionsAtIndexes:(NSIndexSet *)indexes {
    [self willChange:NSKeyValueChangeRemoval
        valuesAtIndexes:indexes forKey:@"transactions"];

    // Remove the transaction objects at the specified indexes.

    [self didChange:NSKeyValueChangeRemoval
        valuesAtIndexes:indexes forKey:@"transactions"];
  }
```

수동 KVO를 구현했으니 자동은 꺼도 된다. (수동 KVO와 자동 KVO는 mutual exclusive하지 않다. 즉, 둘 다 써도 된다. 하지만 일반적으로는 하나만 쓰는게 좋을것이다.) to-many릴레이션십일 경우 특히 이 처리가 중요하다. (단, NSManagedObject에서는 이 함수가 약간 다르게 동작하므로 주의해야 한다. http://stackoverflow.com/q/3728247/2047287 코어데이터는 NSManagedObject의 서브클래스를 만들때 willChangeValueForKey 등을 사용한 프로퍼티 코드를 자동으로 작성하여 제공하므로 사실상 수동KVO를 사용한다.)

```
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)theKey {

    BOOL automatic = NO;
    if ([theKey isEqualToString:@"balance"]) {
        automatic = NO;
    }
    else {
        automatic = [super automaticallyNotifiesObserversForKey:theKey];
    }
    return automatic;
  }
```
위의 코드는 아래처럼 간단하게 써도 된다. 
```swift
class func automaticallyNotifiesObserversOfBalance() -> Bool {
	return false
}
```

## 프로퍼티간 의존성 맺기

의존성 맺는것은 주로 to One relationship 일때 사용한다. 
예를 들어 아래와 같은 프로퍼티가 있다고 하자.

```swift
	var fullname : String {
		return firstname + lastname
	}
```
이 경우 firstName 혹은 lastName이 변경시 자동으로 fullName도 변경되었다는 통지가 발생해야 할 것이다.

이는 다음과 같이 의존성을 등록할 수 있다.
```
+ (NSSet *)keyPathsForValuesAffectingValueForKey:(NSString *)key {

    NSSet *keyPaths = [super keyPathsForValuesAffectingValueForKey:key];

    if ([key isEqualToString:@"fullName"]) {
        NSArray *affectingKeys = @[@"lastName", @"firstName"];
        keyPaths = [keyPaths setByAddingObjectsFromArray:affectingKeys];
    }
    return keyPaths;
  }
```
위의 코드를 줄여서 다음과 같이 써도 된다.
```swift
class func keyPathsForValuesAffectingFullname() -> NSSet {
	return ["firstname","lastname"]
}
```

fullname에 대해 addObserver를 해놓은 뒤, firstname 혹은 lastname을 수정하면  observeValueForKeyPath이 fullname이 변경된것처럼 실행된다. 이때 setValue를 이용해 수정을 해야 자동KVO가 작동한다는것을 명심하자. 물론 수동KVO를 써도 되는데 이때는 firstname혹은 lastname의 setter 내부에서 willChangeValueForKey/didChangeValueForKey 가 호출되어야 할 것이다. 이 두 프로퍼티에 대해서 addObserver를 할 필요는 없다. 

#### to Many 릴레이션십에서의 의존성 맺기는 다음을 참고하자.

https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVODependentKeys.html#//apple_ref/doc/uid/20002179-BAJEAIEE




