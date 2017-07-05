# Core Data

코어데이터는 Storage와 Object Graph와 UI를 연동시키기 위한 프레임워크다.

데이터베이스를 쓰려는 경우가 아니더라도 객체를 파일에 저장하고 복원하는 작업을 한다면 코어데이터를 사용하는게 좋다.

물론 코어데이터의 진가는 다수의 대용량 데이터를 다룰때 발휘된다. 객체를 메모리에 캐싱하고  릴리즈하고, 객체를 검색하고 수정하는 등의 작업들을 해야 한다면 코어데이터를 쓰는게 좋다. prefetching과 faulting등의 기술이 기본으로 들어가므로 유저가 별도로 구현할 필요없다. 또한 멀티스레딩에서의 충돌관리도 해준다. 객체를 versioning하고 merging policy에 따라 데이터를 병합할수 있다.

**보통의 데이터베이스들과는 다르게 코어데이터는 완전한 인메모리(in-memory) 형태로 사용이 가능하다.** 

일반적으로 사용자들이 영속성 특성을 대부분 사용하기 때문에 코어데이터가 “객체 영속성(object persistence)” 프레임워크라고 불리지만, 실제로는 어떠한 형태의 영속성도 지니지 않은 in-memory형태로 사용하는 것이 가능한 것이다. (역자주: 코어데이터는 명시적으로 저장 명령을 내릴때까지 디스크에 저장하지 않는다.)
즉, 코어데이터를 사용할때 “영속성”이 의무적인 기능이 아니란 것을 알아두는 것이 중요하다.

**코어데이터는 멀티스레드가 지원되지 않는다.** 

SQLite 또한 싱글 스레드만 지원하지만, 다른 많은 데이터베이스들은 멀티스레드/멀티유저를 지원한다.
일반적으로 스레딩을 위한 락(lock)을 사용하지 않을경우 프레임워크는 훨씬 빠르고 심플하게 구현될 수 있으며 싱글유저 환경(데스크탑 또는 아이폰)에서 발생할 수 있는 일반적인 시나리오들을 충분히 실현 가능하기때문에 코어데이터는 멀티스레드 기능이 빠져있다. 

**코어데이터는 데이터들을 메모리에 로딩하는 과정 없이는 작업이 불가능하다.**

디스크에서 직접 값을 수정하는 DB와는 달리 코어데이터는 메모리상의 객체를 수정하는것만이 가능하기 때문에 이러한 온디스크(on-disk) 방식의 사용이 불가능하다. 심지어 코어데이터는 객체를 삭제 할 때도, 일단 인스턴스화 시켜서 메모리에 로드를 먼저 해야 삭제가 가능하다. 객체에 추가적으로 오버라이드된 동작들이 로드되고 실행되기 위해서, 그리고 다른 객체와의 연결정보를 최신으로 유지하기 위해서는 이러한 메모리 로드작업이 필수적일 수밖에 없다. 

따라서 코어데이터에서 많은 수의 객체들(수만개 혹은 그 이상)을 수정할 경우 다음과 같은 방법들을 사용하여 메모리 사용량(memory footprint)을 최소한으로 유지할 필요가있다.

- `refreshObject:mergeChanges:`메서드를 이용하여 주기적으로 변하지 않은 객체들을 해제
- `NSFetchRequest`사용시 `setIncludesPropertyValues:NO` 설정을 이용하여 객체의 전체데이터를 불러오는 것을 피하기
- 전체 컨텍스트를 저장한 후, 로드되어 있던 모든 객체들을 릴리즈

하지만 이런 방법을 써도 대량의 데이터를 한꺼번에 업데이트 하려면 불편하다. 메모리에 순차적으로 올려놓으면서 수정하고 다시 save하는 과정을 반복해야 한다. 느릴수밖에 없다. 따라서 Batch Update API가 제공되는데 이를 사용하면 직접 Persistence Store에 접근해서 수정할수 있다.  `NSBatchUpdateRequest`





|           **데이터베이스(Database)**           |           **코어데이터(Core Data)**           |
| :--------------------------------------: | :--------------------------------------: |
|            주된 기능은 데이터 저장/불러오기            | 주된 기능은 객체 그래프 관리(하지만 디스크로의 읽기/쓰기 또한 중요한 기능들 중 하나) |
| 디스크에 저장된 데이터에 대해 작업 (필요에 따라 처리할 데이터만 최소한으로 메모리에 로드) | 메모리상에 로드된 객체에 대해 작업 (디스크로부터 lazy loading이 가능하긴 함) |
|            멍청한(dumb) 데이터를 저장             | 스스로 관리되는 완전한 객체를 다루며 서브클래싱을 통해 커스터마이즈된 동작 정의 가능 |
|  Transactional, Thread safe, multi-user  | Non-transactional, single threaded, single user |
|       메모리에 로딩할 필요없이 테이블 삭제 및 편집 가능       |           무조건 메모리에 로드해야 작업 가능            |
|         디스크에 영속적으로 저장 (에러에 강인함)          |              별도의 저장 프로세스 필요              |
|          수백만 레코드를 생성하려면 오랜시간 소요          | 메모리상에 객체 생성은 매우 빠르게 가능(디스크에 저장 하는 작업은 동일하게 오래 걸림) |
|      “unique” key 와 같은 데이터 제약기능 제공       |  데이터 제약기능은 프로그램 내부의 비지니스 로직에서 따로 구현해야 함  |





##  Managed Object Model

### migration

모델을 수정할때마다 persistent store파일을 삭제하는것은 피곤한 일이다. 마이그레이션을 하면 그럴필요없이 간단하게 새 모델로 업데이트할수 있다.





## Persistence Store

디폴트로 제공되는 SQLite는 타입에 제한이 있다. 
MySQL을 사용하고싶을수도 있다. 때론 객체를 XML파일에 기록하고 싶을때도 있다.  클라우드상의 DB와 로컬디바이스 상의 DB를 똑같이 맞춰야 동기화가 편할수도 있다. 이런 경우 다양한 Persistence Store를 사용하게 된다.

[링크](https://developer.apple.com/library/content/documentation/DataManagement/Conceptual/IncrementalStorePG/Introduction/Introduction.html)에 보면 다양한 종류의 Persistence Store가 소개되어 있다. Atomic Persistence Store(APS)는 파일의 모든 데이터를 한번에 읽어들이는 반면  Incremental PS는 선별적으로 읽어들인다. 전자는 가벼운 데이터에, 후자는 무거운 데이터 관리에 유용하다. APS는 파일의 종류에 제약이 덜하다. HTML파일이나 심지어는 컴마로 연결된 텍스트파일의 문자열도 인식하여 데이터로 사용할수 있게 해준다.

Atomic:

- In-Memory Store (`NSInMemoryStoreType`)
- XML Store (`NSXMLStoreType`)     - iOS에서는 사용불가.
- Binary Store (`NSBinaryStoreType`)

Incremental:

- SQLite Store (`NSSQLiteStoreType`)



> Web Service를 CoreData에 Sync하려는 시도가 있다. RestKit, RestfulCoreData 등이 그것이다. 하지만 이들은 Rest만 지원하며, 서버측이 Cocoa API여야 한다. 그래서 등장한 것이 NSIncrementalStore다. (이를 이용한  AFIncrementalStore도 나왔으나 개발자가 Swift용 라이브러리인 Alamofire 개발하느라 현재는 개발중단된 상태.)



## Managed Objects







#### transient property

```swift
var fullnameChanged = false

extension Employee {


var firstname: String? {
  get {
    willAccessValueForKey("firstname")
    let firstname = primitiveValueForKey("firstname") as? String
    didAccessValueForKey("firstname")
    return firstname
  }

  set {
    willChangeValueForKey("firstname")
    setPrimitiveValue(newValue, forKey: "firstname")
    didChangeValueForKey("firstname")
    fullnameChanged = true
  }
}

	@NSManaged var primitiveFirstname: String?
	@NSManaged var lastname: String?
	var fullname: String? {
		get {
			if fullnameChanged {
				let firstname = primitiveValueForKey("firstname") as? String
				let lastname = primitiveValueForKey("lastname") as? String
				let newFullname = firstname! + " " + lastname!
				setPrimitiveValue(newFullname, forKey: "fullname")
				fullnameChanged = false
				return newFullname
			} else {
				willAccessValueForKey("fullname")
				let fullname = primitiveValueForKey("fullname") as? String
				didAccessValueForKey("fullname")
				return fullname
			}
		}
	}
	@NSManaged var date: NSDate?

}
```



#### KVO

NSOperation은 isExecuting등의 상태변화를 표현하기 위해 willChangeValueForKey...등의 메소드들을 이용한다.

```objective-c
// NSOperation

- (void) start {
    Log(@"Starting %@",self);
    [self willChangeValueForKey: @"isExecuting"];
    _isExecuting = YES;
    [self didChangeValueForKey: @"isExecuting"];
}

- (void) finish {
    [self willChangeValueForKey: @"isExecuting"];
    [self willChangeValueForKey: @"isFinished"];
    _isExecuting = NO;
    _isFinished = YES;
    [self didChangeValueForKey: @"isFinished"];
    [self didChangeValueForKey: @"isExecuting"];
}
```







#### custom accessor



`func [will|did]AccessValue(forKey key: String?)`는 폴트를 자동으로 fire하게 해준다. 따라서 이 메소드를 뺀다면 폴트객체가 제대로 원본 데이터를 보여주지 않을것이다.

`func [will|did]ChangeValue(forKey key: String?)`는 KVO를 위한 메소드다.



### 대량의 MO 다루기

objectID를 활용하자. object graph를 통째로 로드할 필요가 없기 때문에 메모리 효율상 이득임. objectIDsForRelationshipNamed 같은 메소드를 사용하면 objectID로 결과를 얻어낼수 있다.

##### Example

```swift
let task = Task()
let moc = mainContext
let childrenIDs = task.objectIDs(forRelationshipNamed: "children")
let request = NSFetchRequest<Task>(entityName: "Task")
request.fetchBatchSize = 100
let pred = NSPredicate(format: "self IN %@", childrenIDs)
request.predicate = pred
let batchResults : [Task] = try! moc.fetch(request)

for result in batchResults {
    // work with 100 rows at a time
}
```





## MOC functions



#### MOC.reset()

discard changes기능이라고 보면 된다. MOC상의 객체들이 메모리에서 제거된다. fetched objects는 더이상 사용할수 없으므로 refetch해야 한다.

> refresh류의 메소드는 reset과는 달리 객체 참조를 잃어버리지 않는다. 



#### MOC.refreshAllObjects()

MOC상의 모든 객체를 리프레쉬. 단, unsaved changes를 보존함. 참조가 유지되므로 refetch할 필요 없음.
retain cycle을 끊어주고 싶을때도 유용.



#### MOC.refresh(_ object: NSManagedObject, mergeChanges flag: Bool)

MOC상의 특정 객체에 대한 refresh를 할수 있다. 이때 discard changes할것인지 여부는 flag를 조정하면 된다.

flag가 true일 경우엔 persistence store에서 일단 저장된 값을 복원 후, changes를 재적용하는 방식으로 merge한다. 그렇기때문에 merge conflict가 발생하지 않는다.



#### NSBatchDeleteRequest

수천개 이상의 단위로 MO를 지워야할때 Context를 통한 기존의 방식으로 지우면 굉장히 느리다. 따라서 PS에 직접 접근해서 지우는 방식이다.  



## Multi-Threading

- MO는 멀티스레드에서 동시접근 하게되면 데드락에 걸릴수 있다. (faulting메카니즘 때문.)
- 이 때문에 멀티스레딩을 하려면 복수의 MOC를 만들어야 하고 ManagedObjectID를 이용해 refetch를 해야 하는 불편함이 있음.
- context가 MainQueueConcurrencyType일 경우 굳이 실행코드를 perform블록으로 감쌀필요 없다. (해도 상관은 없다.)
- parent / child context 를 이용한 방식.  [하지만 이 방식은 단점이 존재.](http://floriankugler.com/2013/04/02/the-concurrent-core-data-stack/) child는 parent라는 중간단계를 통해 persistent store와 연결되어있기때문에 결국 child가 PS로부터 대량의 import를 하게되면 parent 즉, 메인쓰레드에 부하를 주게됨. 대량의 save과정도 마찬가지.
- [main / background context 방식](http://floriankugler.com/2013/04/29/concurrent-core-data-stack-performance-shootout/) : NSManagedObjectContextDidSaveNotification를 이용한 merge를 함. AERecord에도 사용됨. (⚠️이 방식을 쓸 경우 백그라운드MOC는 데이터 수정후 꼭 save를 해줘야 함. 그래야 메인MOC가 업데이트되고 UI에 반영됨.)
- NSFetchedResultsControllerDelegate는 반드시 메인스레드에서 동작해야함. 

#### AERecord 사용시 유의점

- context save을 잘 이용해야 함. save를 통해 main context - background context간의 동기화가 이뤄짐.

  ```swift
      // MARK: - Sync
      
      @objc func contextDidSave(_ notification: Notification) {
          guard
              let context = notification.object as? NSManagedObjectContext,
              let contextToRefresh = context == mainContext ? backgroundContext : mainContext
          else { return }
          
          mergeChanges(from: notification, in: contextToRefresh)
      }
  ```

  위의 메소드를 보면 알겠지만 backgroundContext를 mainContext와 동일하게 만들려면  mainContext를 save하면 됨.

  ​

  ​


## Parent / Child Context

[멀티 쓰레딩을 위한 구조라고 착각하기 쉽지만 사실은 멀티 뷰컨트롤러를 위한 것이다.](http://stackoverflow.com/a/36154062/2047287) 



## NSPredicate Examples





## XCode GUI

### unique constraints

엔티티의 특정 어트리뷰트에 유일속성을 줄수 있다. 중복된 엔티티를 걸러낼수 있게된다. 단, save시에 적용된다는점 때문에 FRC를 쓸때는 별로 유용하지 않다.

http://stackoverflow.com/a/32814593/2047287





## ⚠️주의사항

1. primitive getter를 사용하지 말자. fault객체가 제대로 fetch되지 않는다.
2. 각 MO클래스에 `func observeValue(...)` 메소드를 만들고 로그를 찍자.  어떤 KVO가 백그라운드에서 작동하는지를 캐치할수 있다. 불필효한 것을 제거할수 있다.

```swift
override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
	Log("\(type(of:self)) '\(self.title.truncateTail(5))'의 '\(keyPath!)'프로퍼티에 KVO가 발동 : \n <변경사항>\n 기존 값: \(change![NSKeyValueChangeKey.oldKey] ?? "null") \n 새 값 : \(change![NSKeyValueChangeKey.newKey] ?? "null") \n")
	if keyPath == "isCompleted" && context == &(Task.taskModificationContext) {
		guard let newValue = change?[NSKeyValueChangeKey.newKey] as? Bool else { fatalError() }
		// ...
	} else {
		assertionFailure("Unexpected key: \(keyPath)")
		super.observeValue(forKeyPath: keyPath, of: object, change: change, context: context)
	}
}
```



## 참고

- http://petersteinberger.com/blog/2012/dont-call-willchangevalueforkey/
- http://www.letmecompile.com/코어데이터core-data와-데이터베이스의-차이/
- ​

