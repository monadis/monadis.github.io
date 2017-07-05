https://gist.github.com/monadis/c25211ff6f49aaff9e337883e5852820


## 설치법

Podfile

```shell
use_frameworks!

target "타겟명" do

    pod 'Operations'

end

workspace 'MyWorkspace'
```

## Operation 프레임워크의 개념

Operation은 뭐에 쓰나? 데이터 계산, 파싱, 복구, 뷰컨트롤러 프레젠테이션 등등.. 
사실 모든 작업을 할수 있지만 대개는 특정 조건일때 내용이 달라질 수 있는 분기형 작업, 연결형 작업 등을 할 때 쓴다.

오퍼레이션은 서브클래싱을 해서 만들면 된다.하지만 일반적으로 자주 쓰이는 것들은 해당 킷에서 제공하고 있으니 갖다쓰자.

자세한 사용법은 여기에
https://github.com/ProcedureKit/ProcedureKit

### Scheduling

아래의 링크에 각 오퍼레이션 상태 (pending, ready, executing, finished, canceled)에 대한 설명이 나와있다.

https://operations.readme.io/docs/scheduling


### 서브클래싱 방법

```swift
import Operations

class MyFirstOperation: Operation {

    override func execute() {
        guard !cancelled else { return }
        
        // 나의 코드
        
        finish()
    }

}
```

## Observer

`(WillExecute/DidFinish/WillCancel/DidCancel/ProducedOperation)Observer` 등이 있다.
~~StartedObserver, CancelledObserver, ProducedObserver and FinishedObserver 등이 있다.~~ 
started 와 finished 옵저버는 execute()가 실행되기 전후로 실행되는 콜백메소드를 소유한다.
옵저버의 갯수는 제한이 없다.
```swift
operation.addObserver(WillExecuteObserver { op in 
    print("Lets go!")
})
```

#### 그밖의 옵저버
BackgroundObserver : 앱이 백그라운드로 가도 작업을 계속해야 할때 사용.
TimeoutObserver : 일정 시간이 지나도 오퍼레이션이 종료되지 않으면 Cancel해버림.
NetworkObserver : `network activity indicator`를 표시하기 위한 옵저버.  (네트워크가 사용가능한 상태인지를 알아내는 `ReachableOperation`와 혼동하지 말자.)

자세한 내용은 아래에
https://operations.readme.io/docs/observers

## Condition 

> Condition 실패로 인해 발생한 에러는 해당 오퍼레이션이 끝나는 WillFinish옵저버에서 처리가능하다. 의존관계로 연결된 다음 오퍼레이션들은 자동으로 Cancel상태가 된다.  따라서 의존성으로 연결된 연쇄적인 작업들 중에 발생한 컨디션 에러를 실타래의 맨 마지막에 처리하고자 한다면 GroupOperation를 이용해야 한다.

Condition자체도 Operation의 서브클래스임. 따라서 컨디션이 다른 컨디션에 dependency를 가질수도 있고 오퍼레이션에 대해서도 마찬가지.
컨디션이 의존하는 오퍼레이션은 자동으로 큐에 넣어진다. 아래의 예문으로 확인할수 있다.
```swift
let i = 7
let op1 = BlockOperation{print("op1")}
let op2 = BlockOperation{print("op2")}
let cond = BlockCondition{ i > 4 }
cond.addDependency(op1)
op2.addCondition(cond)
queue.addOperations(op2)
// 실행 결과 
// op1
// op2
```

가장 기본적인 컨디션은 아래와 같다.

#### BlockCondition
```swift
operation.addCondition(BlockCondition { 
    // operation will only be executed if this is true
    return trueOrFalse
})
```
각 컨디션은  .Satisfied 또는 .Failed(ErrorType) 를 리턴함

#### MutuallyExclusive

AlertOperation은 기본적으로 MutuallyExclusive 컨디션이 적용되어있다.
따라서 연속적으로 두개 이상의 AlertOperation을 큐에 넣으면 첫번째의 Alert창이 사라질때 비로서 두번째 Alert창이 뜬다. 즉, MutuallyExclusive는 동일한 종류의 오퍼레이션 실행을 늦추는 역할을 한다. 자세히 말하면, 동일한 종류의 오퍼레이션이 큐에 있으면 그에 대해 dependency를 자동으로 부여함으로써 둘이 동시에 실행되는 것을 막는 것이다.

```swift
let alert = AlertOperation(presentAlertFrom: self)
alert.addActionWithTitle("alert")
let alert2 = AlertOperation(presentAlertFrom: self)
alert2.addActionWithTitle("alert2")
queue.addOperations(alert, alert2)
```

#### UserConfirmationCondition

UserConfirmationCondition은 AlertOperation을 의존하고있다. 즉 내가 직접 AlertOperation을 큐에 넣지 않더라도 이 컨디션에 의해 자동으로 큐에 넣어지고 실행된다.

```swift
let userConfirm = UserConfirmationCondition(title: "Hello", action: "Action", presentConfirmationFrom: self)
let op1 = BlockOperation{printLine("op1")}
op1.addCondition(userConfirm)
queue.addOperations(op1)
```
#### SilentCondition
위의 UserConfirmationCondition 예에서 addCondition부분을 다음과 같이 바꿔보자.
```swift
op1.addCondition(SilentCondition(userConfirm))
```
이러면 의존관계에 있는 AlertOperation이 작동하지 않는다. 
즉, SilentCondition은 의존관계를 끊어주어 해당 컨디션만 실행되게 한다.

#### NegatedCondition
Not 연산을 함. 아래의 예에서는 i<=3 인 Condition으로 바꿔줌.
```swift
let condition = BlockCondition{ self.i > 3 }
alert.addCondition(NegatedCondition(condition))
```
#### ReachabilityCondition

#### AuthorizedFor
아래의 Authorization 참고

## Authorization

컨디션을 이용하여 특정 기능에 대한 Authorization을 요청하기

```swift
class ReminderOperation: Operation {
   override init() {
       super.init()
       name = "Reminder Operation"
       addCondition(AuthorizedFor(Capability.Calendar(.Reminder)))
   }
   
   override func execute() {
       // do something with EventKit here
       finish()
   }
}
```
**Capabilities**

`.Calendar, .Cloud, .Health, .Location, .Passbook, .Photos`

(기능에 대한 요청없이) 해당 기능이 사용가능한지 체크하기
```swift
func updateForStatus(enabled: Bool, status: EKAuthorizationStatus) {
    // etc - note that the status is specific to EventKit
}

func checkReminderAuthorization() {
    let capability = Capability.Calendar(.Reminder) 
    let check = GetAuthorizationStatus(capability, completion: updateForStatus)
    queue.addOperation(check)
}
```

컨디션을 이용하지 않고 Authorize오퍼레이션을 이용하여 명시적으로 authorization을 요청하는 방법도 있다. AuthorizedFor컨디션 자체가 Authorize오퍼레이션을 사용하므로 사실상 동일한 작업이다.

```swift
func requestReminderAuthorization() {
    let capability = Capability.Calendar(.Reminder) 
    let request = Authorize(capability, completion: updateForStatus)
    queue.addOperation(request)
}
```
Condition으로 Authorization을 할때의 단점은 UX적으로 바람직하지 않은 타이밍에 Authorization을 묻는 창이 뜰 수 있다는것이다. 이를 수동으로 억제시키고자 할때 `SilentCondition`을 활용하면 된다.
```swift
func doBonusReminderStuff() {
    let doReminderStuff = ReminderOperation() // NSOperation
    let operation = ComposedOperation(doReminderStuff) // Operation
    let capability = Capability.Calendar(.Reminder)
    operation.addCondition(SilentCondition(AuthorizedFor(capability)))
    queue.addOperation(operation)
}
```
해당 Capability가 작동하지 않는 상황에 대처하기 위해 `NegatedCondition`을 활용할수 있다.
```swift
func fallbackGracefullyWithoutReminders() {
    let fallback = FallbackOperation() // assume we have this `Operation` subclass
    let capability = Capability.Calendar(.Reminder)
    let condition = NegatedCondition(SilentCondition(AuthorizedFor(capability)))
    fallback.addCondition(condition)
    queue.addOperation(fallback)
}
```

## Logging

 `.Verbose, .Notice, .Info, .Warning, .Fatal` 의 다섯가지가 있음.
보통은 .Notice로 놓는다.
스케줄링과 관련한 디버깅을 할때는 .Verbose를 사용하자. 

```swift
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {

    LogManager.severity = .Notice
    return true
}
```

로그를 받고싶지 않다면
```swift
LogManager.enabled = false
```
https://operations.readme.io/docs/logging


## Procedure의 종류

#### Network Procedure

```swift
func testNetworkProcedure() {
	let url : URL! = "http://www.dragonballgt.com/images/content-keyart-home.jpg"
	let request : URLRequest! = URLRequest(url: url)
	let session = URLSession(configuration: .default)
	let download = NetworkDataProcedure(session: session, request: request) { result in
		let data : Data = result.value!.payload!
		let image = UIImage(data: data)
		ULog(image: image)
	}
	operationQueue.add(operation: download)
}
```

#### ComposedOperation

NSOperation의 서브클래스를 Operation으로 바꿔주는 wrapper

#### GatedOperation
ComposedOperation와 유사하지만 블록을 이용해서 컨디션을 추가할수 있다.
```swift
let operation = GatedOperation(anotherOperation) { return trueOrFalse() }
queue.addOperation(operation)
```

#### DelayOperation
```swift
let delayInterval = DelayOperation(interval: 1)
let delayDate = DelayOperation(date: dateInTheFuture)
```

#### GroupOperation
블록오퍼레이션 다음으로 가장 자주 사용되는 오퍼레이션
오퍼레이션은 큰 작업을 작은 작업들로 분할하는 장점이 있지만 다른한편으로는 파편화해서 정리가 안되는 단점도 존재한다. 따라서 이를 보완하기 위해 여러 오퍼레이션을 그룹화하여 다루는 것이 그룹오퍼레이션이다. 

그룹오퍼레이션 만들기
```swift
class LoginOperation: GroupOperation {

    class PersistLoginInfo: Operation { /* etc */ }
    class LoginSessionTask: NSURLSessionTask { /* etc */ }
    
    init(credentials: Credentials /* etc, inject known dependencies */) {

        // Create the child operations
        let persist = PersistLoginInfo(credentials)
        let login = URLSessionTaskOperation(task: LoginSessionTask(credentials))
        
        // Do any configuration or setup
        persist.addDependency(login)
        
        // Call the super initializer with the operations
        super.init(operations: [persist, login])
        
        // Configure any properties, such as name.
        name = "Login"
        
        // Add observers, conditions etc to the group
        addObserver(NetworkObserver())
        addCondition(MutualExclusive<LoginOperation>())
    }
}
```

사용법
```swift
let group = GroupOperation(operations: opOne, opTwo, opThree)
group.addObserver(FinishedObserver { op in
    // the group has finished, meaning all operations have finsihed.
})
queue.addOperation(group)
```
그룹 오퍼레이션 실행도중에 다른 오퍼레이션을 추가하는것도 가능하다.
```swift
group.addOperation(opFour)
group.addOperations([opFive, opSix])
```

child operation을 동적으로 추가하는 그룹오퍼레이션 만드는 방법은 아래의 링크 참고.
https://operations.readme.io/docs/groups

#### UIOperation

뷰컨트롤러를 presenting할때 사용하는 오퍼레이션.
mutual exclusivity를 보장한다.
Observer를 이용해 콜백 구현가능하다. 컨디션을 만족하지 못하여 presenting을 실패한 경우 셀을 deselect하는 코드 등은 이를 통해 구현할 수 있다.
> 굳이 별도의 클래스를 만들지 않고도 콜백 작업을 할수 있다

```swift
let vc = UIViewController()
vc.view.backgroundColor = UIColor.redColor()

let ui = UIOperation(
	controller: vc,
	displayControllerFrom: .Present(self)
)
ui.addObserver(WillExecuteObserver { op in print("Lets go!")})
ui.addObserver(DidFinishObserver { op in print("Did Finish!")})
queue.addOperation(ui)
```
https://operations.readme.io/docs/ui-operation

#### BlockOperation

UIOperation은 블록오퍼레이션과 Segue의 조합으로도 구현 가능하다.
```swift

let operation = BlockOperation {
  if !isRunningFetchOperation {
    self.performSegueWithIdentifier("showTasks", sender: nil)
  }
}

operation.addCondition(MutuallyExclusive<UIViewController>())

let blockObserver = BlockObserver { _, errors in
  /*
  If the operation errored (ex: a condition failed) then the segue
  isn't going to happen. We shouldn't leave the row selected.
  */
  if !errors.isEmpty {
    dispatch_async(dispatch_get_main_queue()) {
      tableView.deselectRowAtIndexPath(indexPath, animated: true)
    }
  }
}

operation.addObserver(blockObserver)
operationQueue.addOperation(operation)
```
https://operations.readme.io/docs/block-operation


#### 기타 오퍼레이션들
CloudKit Operation
Network Operation (URLSessionTaskOperation)
Reachable Operation : 상대방 호스트가 제대로 작동하는지 확인
UserLocationOperation : 사용자의 위치를 알아냄
AlertOperation

## Injecting Results

오퍼레이션들간의 결과값 전달은 일반적인 함수와는 달리 동적으로 타입이 정해져야 하고 decoupling을 유지하는게 좋다.
그러기 위해서 프로토콜을 이용한다.

#### 자동의 결과값 전달 (결과 프로퍼티가 한개일 경우)
전달하는 쪽은 `ResultOperationType`을 구현하고, 받는쪽은 `AutomaticInjectionOperationType`을 구현하면 된다.

```swift
// 네트워크에서 데이터를 받는 오퍼레이션
class DataRetrieval: Operation, ResultOperationType {
    private(set) var result: NSData?
    
    override func execute() {
        fetchDataFromNetwork { data in
            result = data
                finish()
        }
    }
}

// 데이터를 처리하는 오퍼레이션
// 전달받은 데이터는 requirement에 저장되며 이를 처리하여 result에 저장. 
class DataProcessing: Operation, AutomaticInjectionOperationType, ResultOperationType {
    var requirement: NSData?
    private(set) var result: NSData?
    
    override func execute() {
        processData(requirement) { processed in
            result = processed
            finish()
        }
    }
}
```
**전달되는 변수의 타입은 Optional로 하는 것이 좋다.**

사용은 아래와 같이 한다.
```swift
let retrieval = DataRetrieval()
let processing = DataProcessing()
processing.injectResultFromDependency(retrieval)
queue.addOperations(retrieval, processing)
```

#### 수동의 결과값 전달 (결과 프로퍼티가 다수일 경우)
```swift
let retrieval = DataRetrieval()
let processing = DataProcessing()
processing.injectResultFromDependency(retrieval) { op, dep, errors in 
    // Here we have access to both operations, plus any errors
    // from the dependency.
}
```



## ⚠️ 주의 사항

- OperationQueue에 addOperation(...) 이라는 메소드가 있는데 이를 사용하면 안되고 ProcedureKit의 add(operations:)메소드를 사용해야 한다.