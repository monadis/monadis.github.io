# Observer patterns

*sometimes called Broadcaster/Listener, Publish/Subscribe or Notifications*

[http://cocoawithlove.com/2008/06/five-approaches-to-listening-observing.html](http://cocoawithlove.com/2008/06/five-approaches-to-listening-observing.html)

두 모듈간 관계를 추상화 하기에 좋은 강력한 도구. 즉, 어떤 모듈을 단위별로 독립적으로 다루기 쉽게 유지하고자 한다면 옵저버 패턴이 유용하다. 수신자가 누군지 신경쓰지 않고 명령만 전달하는 형식으로 클래스를 구현함으로써 송신자와 수신자간의 결합이 줄어든다. 



이 패턴을 구현하는 방법들



## Delegate를 이용한 관찰

setDelegate:메소드 및 프로토콜을 이용하여 구현.

- 장점: 세부정보를 전달 가능. 프로토콜을 이용해서 타입정보를 세팅가능함으로써 특별한 도큐먼트가 필요없다.  
- 단점: 해당클래스가 딜리게이트를 지원해야 함. 프레임워크를 이용할 경우, 오직 하나의 수신자(딜리게이트)만 연결가능.




## Property Observer 를 이용한 관찰.

willSet, didSet를 말한다. 

구현이 매우 편리하지만 수신자가 자기 자신이라는 한계가 있다. 이 한계는 앞의 delegate와 연계하여 극복할수 있다.

```swift
protocol CanObserveA {
    func doSomethingForA()
}
class A {
    var a : String = "a" {
        didSet {
            // print("a(기존값: \(oldValue))  에 다음 값이 세팅됨 : \(a)")
            observer?.doSomethingForA()
        }
    }
    var observer : CanObserveA?
}

class B : CanObserveA {
    let a = A()

    func doSomethingForA() {
        print("do something")
    }
}

let a = A()
let b = B()
a.observer = b
a.a = "b"
```




## Notification Center

언제 사용하나?

- 라이브러리에서 노티피케이션 센터 API를 제공한 경우
- 관찰대상이 프로퍼티의 변화 자체가 아니라 어떤 이벤트일때
- KVO로 구현하기엔 너무 노티피케이션의 발신과 수신이 적을 때.
- 퍼포먼스가 그리 문제되지 않을때 (노티피케이션센터는 동기화문제 때문에 퍼포먼스가 나쁘다)



종류:

- NSNotificationCenter를 이용하는 방법: 동기적 알림
- NSNotificationQueue를 이용하는 방법: 비동기적 알림. 해당 스레드에 run loop가 존재해야만 실행가능.
  - 옵저버들이 idle상태일때 노티피케이션을 받게된다. 
  - 큐 안의 알림은 priority에 따라 전달순서 정해짐
  - 유사 알림들을 병합 가능
  - 아직 전달되지 않은 알림을 큐에서 삭제 가능
- NSDistributedNotificationCenter를 이용하는 방법: 
  - 어플리케이션간 통신을 위한 것. 다른프로세스에 알림. 
  - 비동기적. 
  - 프로퍼티리스트값만 포함가능.



사용법:

```objective-c
// 메시지 보내기
[[NSNotificationCenter defaultCenter] postNotificationName:@"myNotificationName" object:송신자];

// 메시지 받기위해 등록하기
[[NSNotificationCenter defaultCenter] addObserver:수신자 selector:@selector(receivingMethodOnListener:) name:@"myNotificationName" object:송신자(nil로 놔도 상관없음)];
```



- 장점: 송신자와 수신자 양쪽이 서로 몰라도 된다. 알림을 받을 메소드를 지정할 수 있다. 노티피케이션 이름을 자유롭게 정할수 있다.
- 단점: KVO와 마찬가지로 지울때 등록해지해야 함.




> 한꺼번에 많은 옵저버를 등록하거나 해지하지 마라. 차라리 중개역할을 하는 옵저버를 하나 만들어 이것이 다른 객체들과 송수신하게 하는것이 퍼포먼스에 좋다. 




## Context Notifications

NSManagedObject의 프로퍼티를 관찰할 경우, NSManagedObjectContextObjectsDidChangeNotification를 수신할 수 있다. 

위의 노티피케이션 센터와 비슷하지만 송신측이 약간 다르다. 송신측 코드를 작성할 필요가 없다.

```objective-c
[[NSNotificationCenter defaultCenter] addObserver:listenerObejct selector:@selector(receivingMethodOnListener:) name:NSManagedObjectContextObjectsDidChangeNotification object:observedManagedObjectContext];
```

또한, receivingMethodOnListener: 에서는 ,  notification의 userInfo NSInsertedObjectsKey, NSUpdatedObjectsKey 및 NSDeletedObjectsKey 등의 키값을 받아 변경된 객체들의 집합을 얻을 수 있다.

- 장점: 컨택스트상의 변화를 트래킹하기 편리하다.
- 단점: 코어데이터에서만 사용가능. 제한적인 객체 정보 이상의 자세한 정보는 얻을수 없다.





## Target - Action

이 방식의 유연성은 Responder Chain을 이용할때 나온다. 커맨드를 받으면 리스폰더체인을 따라가면서 알맞은 객체가 응답하도록 할수 있다.





## 수동 BroadCaster/Listener (NSObject only)

송신자(브로드캐스터)는 수신자 목록을 컬렉션 객체를 프로퍼티로 가져야 한다.

이 때, Array 혹은 Set형태로 목록을 갖는다면 다음의 코드로 송신하면 된다.

[listenersCollection makeObjectsPerformSelector:@selector(methodSupportedByEveryListener)];

Dictionary를 이용, 이벤트식별자 키값으로하여 옵저버를들 분류하기도 한다.

이방식의 장단점

- 장점: 송신자가 수신자를 완전히 제어/관리할수 있다.
- 단점: 수동으로 수신자 제거, 추가를 해야하므로 불편. 이벤트가 여러 종류면 더욱 관리가 어려워짐.



## KVO (NSObject only)

참고: <a href="./KVC KVO.md">KVO</a>

Notification Center보다 성능이 좋지만 잡아내기 힘든 에러를 발생시킬수 있으므로 가능하면 쓰지 않는 것이 좋다. (권장하지 않음) 

NSObject를 상속한 객체가 아니면 사용이 불가하다는 한계가 있다. 

표준 프로퍼티를 구현하면 자동으로 KVO를 만족하므로 별도의 송신자측 코딩이 필요없다.

다음과 같이 addObserver메소드만 호출하면 되므로 구현이 간단. (setter 내부에 willChangeValueForKey: didChangeValueForKey:를  직접 코딩하지 않아도 된다. 단, 값을 할당할때 setter를 거치는지 유의하자. _object=value; 이런식으로 직접 내부변수에 할당하면 KVO가 작동하지 않는다.)

[송신자 addObserver:수신자 forKeyPath:@"myValue" options:NSKeyValueChangeNewKey context:nil];

위 예의 경우, setMyValue라는 이름의 setter에 새 값이 입력될때 마다 수신자에게 observeValueForKeyPath:ofObject:change:context: 라는 메시지가 전달됨.

observationInfo를 이용하면 관찰자가 누군지 알수 있다. 개발용도라기 보다는 디버깅에 편리하다.

- 장점: 모두 자동이다. 간단하다. keypath도 관찰할 수 있다.
- 단점: 송신자는 수신자가 누군지 알수없다. naming convention을 따라야 한다. 수신자는 반드시 옵저버 등록을 해지 후에 삭제되어야 한다. 

